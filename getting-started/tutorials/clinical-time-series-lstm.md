# Clinical Time Series LSTM

In this tutorial, we will learn how to apply a long-short term memory \(LSTM\) neural network to a medical time series problem. The data used comes from 4000 intensive care unit \(ICU\) patients and the goal is to predict the mortality of patients using 6 general descriptor features, such as age, gender, and weight along with 37 sequential features, such as cholesterol level, temperature, pH, and glucose level. Each patient has multiple measurements of the sequential features, with patients having a different amount of measurements taken. Furthermore, the time between measurements also differ among patients as well.

A LSTM is well suited for this type of problem due to the sequential nature of the data. In addition, LSTM networks avoid vanishing and exploding gradients and are able to effectively capture long term dependencies due to its cell state, a feature not present in typical recurrent networks.

## Imports

```scala
import org.datavec.api.records.reader.SequenceRecordReader;
import org.datavec.api.records.reader.impl.csv.CSVSequenceRecordReader;
import org.datavec.api.split.NumberedFileInputSplit;
import org.deeplearning4j.datasets.datavec.SequenceRecordReaderDataSetIterator;
import org.deeplearning4j.eval.ROC;
import org.deeplearning4j.nn.api.OptimizationAlgorithm;
import org.deeplearning4j.nn.conf.ComputationGraphConfiguration;
import org.deeplearning4j.nn.conf.NeuralNetConfiguration;
import org.deeplearning4j.nn.conf.Updater;
import org.deeplearning4j.nn.conf.layers.GravesLSTM;
import org.deeplearning4j.nn.conf.layers.RnnOutputLayer;
import org.deeplearning4j.nn.graph.ComputationGraph;
import org.deeplearning4j.optimize.listeners.ScoreIterationListener;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.deeplearning4j.nn.weights.WeightInit;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.dataset.api.DataSet;
import org.nd4j.linalg.lossfunctions.LossFunctions;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;
import org.nd4j.linalg.learning.config.Adam

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import org.apache.commons.io.FileUtils;
import org.apache.commons.io.FilenameUtils;
import java.io.IOException;
import java.util.HashMap;
import java.util.Arrays;
import java.net.URL;
import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.BufferedOutputStream;
import java.io.FileOutputStream;
import java.lang.Byte;

import org.apache.commons.compress.archivers.tar.TarArchiveEntry;
import org.apache.commons.compress.archivers.tar.TarArchiveInputStream;
import org.apache.commons.compress.compressors.gzip.GzipCompressorInputStream;
```

Now that we have imported everything needed to run this tutorial, we will start with obtaining the data and then converting the data into a format a neural network can understand.

## Data Source

The data is contained in a compressed tar.gz file. We will have to download the data from the url below and then extract csv files containing the ICU data. Each patient will have a separate csv file for the features and labels. The features will be contained in a directory called sequence and the labels will be contained in a directory called mortality. The features are contained in a single csv file with the columns representing the features and the rows representing different time steps. The labels are contained in a single csv file which contains a value of 0 indicating death and a value of 1 indicating survival.

```scala
val DATA_URL = "https://dl4jdata.blob.core.windows.net/training/physionet2012/physionet2012.tar.gz"
val DATA_PATH = FilenameUtils.concat(System.getProperty("java.io.tmpdir"), "dl4j_physionet/")
```

### Download Data

To download the data, we will create a temporary directory that will store the data files, extract the tar.gz file from the url, and place it in the specified directory.

```scala
val directory = new File(DATA_PATH)
directory.mkdir() // create new directory at specified path

val archizePath = DATA_PATH + "physionet2012.tar.gz" // set path for tar.gz file
val archiveFile = new File(archizePath) // create tar.gz file
val extractedPath = DATA_PATH + "physionet2012" 
val extractedFile = new File(extractedPath)

FileUtils.copyURLToFile(new URL(DATA_URL), archiveFile) // copy data from URL to file
```

Next, we must extract the data from the tar.gz file, recreate directories within the tar.gz file into our temporary directory, and copy the files into our temporary directory.

```scala
var fileCount = 0
var dirCount = 0
val BUFFER_SIZE = 4096

val tais = new TarArchiveInputStream(new GzipCompressorInputStream( new BufferedInputStream( new FileInputStream(archizePath))))

var entry = tais.getNextEntry().asInstanceOf[TarArchiveEntry]

while(entry != null){
    if (entry.isDirectory()) {
        new File(DATA_PATH + entry.getName()).mkdirs()
        dirCount = dirCount + 1
        fileCount = 0
    }
    else {
        
        val data = new Array[scala.Byte](4 * BUFFER_SIZE)

        val fos = new FileOutputStream(DATA_PATH + entry.getName());
        val dest = new BufferedOutputStream(fos, BUFFER_SIZE);
        var count = tais.read(data, 0, BUFFER_SIZE)
        
        while (count != -1) {
            dest.write(data, 0, count)
            count = tais.read(data, 0, BUFFER_SIZE)
        }
        
        dest.close()
        fileCount = fileCount + 1
    }
    if(fileCount % 1000 == 0){
        print(".")
    }
    
    entry = tais.getNextEntry().asInstanceOf[TarArchiveEntry]
}
```

### DataSetIterators

Our next goal is to convert the raw data \(csv files\) into a DataSetIterator, which can then be fed into a neural network for training. Our training data will have 3200 examples which will be represented by a single DataSetIterator, and the testing data will have 800 examples which will be represented by a separate DataSet Iterator.

```scala
val NB_TRAIN_EXAMPLES = 3200 // number of training examples
val NB_TEST_EXAMPLES = 800 // number of testing examples
```

In order to obtain DataSetIterators, we must first initialize CSVSequenceRecordReaders, which will parse the raw data into record-like format. We will first set the directories for the features and labels and initialize the CSVSequenceRecordReaders.

Next, we can initialize the SequenceRecordReaderDataSetIterator using the previously created CSVSequenceRecordReaders. We will use an alignment mode of ALIGN\_END. This alignment mode is needed due to the fact that the number of time steps differs between different patients. Because the mortality label is always at the end of the sequence, we need all the sequences aligned so that the time step with the mortality label is the last time step for all patients. For a more in depth explanation of alignment modes, see [Recurrent Networks in DL4J](../../models/recurrent.md#masking-one-to-many-many-to-one-and-sequence-classification).

```scala
val path = FilenameUtils.concat(DATA_PATH, "physionet2012/") // set parent directory

val featureBaseDir = FilenameUtils.concat(path, "sequence") // set feature directory
val mortalityBaseDir = FilenameUtils.concat(path, "mortality") // set label directory

// Load training data

val trainFeatures = new CSVSequenceRecordReader(1, ",")
trainFeatures.initialize( new NumberedFileInputSplit(featureBaseDir + "/%d.csv", 0, NB_TRAIN_EXAMPLES - 1))

val trainLabels = new CSVSequenceRecordReader()
trainLabels.initialize(new NumberedFileInputSplit(mortalityBaseDir + "/%d.csv", 0, NB_TRAIN_EXAMPLES - 1))

val trainData = new SequenceRecordReaderDataSetIterator(trainFeatures, trainLabels,
              32, 2, false, SequenceRecordReaderDataSetIterator.AlignmentMode.ALIGN_END)
              
              
// Load testing data
val testFeatures = new CSVSequenceRecordReader(1, ",");
testFeatures.initialize(new NumberedFileInputSplit(featureBaseDir + "/%d.csv", NB_TRAIN_EXAMPLES, NB_TRAIN_EXAMPLES + NB_TEST_EXAMPLES - 1));
       
val testLabels = new CSVSequenceRecordReader();
testLabels.initialize(new NumberedFileInputSplit(mortalityBaseDir + "/%d.csv", NB_TRAIN_EXAMPLES, NB_TRAIN_EXAMPLES  + NB_TEST_EXAMPLES - 1));

val testData = new SequenceRecordReaderDataSetIterator(testFeatures, testLabels,
                32, 2, false,SequenceRecordReaderDataSetIterator.AlignmentMode.ALIGN_END);
```

## Neural Network Configuration

Now we can finally configure and then initialize the neural network for this problem. We will be using the ComputationGraph class of DL4J.

```scala
// Set neural network parameters
val NB_INPUTS = 86
val NB_EPOCHS = 10
val RANDOM_SEED = 1234
val LEARNING_RATE = 0.005
val BATCH_SIZE = 32
val LSTM_LAYER_SIZE = 200
val NUM_LABEL_CLASSES = 2
```

```scala
val conf = new NeuralNetConfiguration.Builder()
        .seed(RANDOM_SEED)
        .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
        .updater(new Adam(LEARNING_RATE))
        .weightInit(WeightInit.XAVIER)
        .dropOut(0.25)
        .graphBuilder()
        .addInputs("trainFeatures")
        .setOutputs("predictMortality")
        .addLayer("L1", new GravesLSTM.Builder()
                .nIn(NB_INPUTS)
                .nOut(LSTM_LAYER_SIZE)
                .forgetGateBiasInit(1)
                .activation(Activation.TANH)
                .build(),
                "trainFeatures")
        .addLayer("predictMortality", new RnnOutputLayer.Builder(LossFunctions.LossFunction.XENT)
                .activation(Activation.SOFTMAX)
                .nIn(LSTM_LAYER_SIZE).nOut(NUM_LABEL_CLASSES).build(),"L1")
        .build()
        
val model = new ComputationGraph(conf)
```

## Training

To train the neural network, we simply call the fit method of the ComputationGraph on the trainData DataSetIterator and also pass how many epochs it should train for.

```scala
model.fit(trainData, 2)
```

## Model Evaluation

Finally, we can evaluate the model with the testing split using the AUC \(area under the curve metric \) using a ROC curve. A randomly guessing model will have an AUC close to 0.50, while a perfect model will achieve an AUC of 1.00

```scala
val roc = new ROC(100);

while (testData.hasNext()) {
    val batch = testData.next();
    val output = model.output(batch.getFeatures());
    roc.evalTimeSeries(batch.getLabels(), output(0));
}

println("FINAL TEST AUC: " + roc.calculateAUC());
```

We see that this model achieves an AUC on the test set of 0.69!


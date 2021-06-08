# Using Multiple GPUs

Training neural network models can be a computationally expensive task. In order to speed up the training process, you can choose to train your models in parallel with multiple GPU’s if they are installed on your machine. With deeplearning4j \(DL4J\), this isn’t a difficult thing to do. In this tutorial we will use the MNIST dataset \(dataset of handwritten images\) to train a feed forward neural network in parallel with multiple GPUs.

**Note:** This also works if you can't fully load your CPU. In that case you just stay with the CPU specific backend.

## Prerequisite

* You must have multiple CUDA compatible GPUs, ideally of the same speed
* You must setup your project to use the CUDA Backend, for help see [Backends](../../config/backends/)

## Imports

```scala
import org.deeplearning4j.datasets.iterator.impl.MnistDataSetIterator;
import org.deeplearning4j.eval.Evaluation;
import org.deeplearning4j.nn.api.OptimizationAlgorithm;
import org.deeplearning4j.nn.conf.MultiLayerConfiguration;
import org.deeplearning4j.nn.conf.NeuralNetConfiguration;
import org.deeplearning4j.nn.conf.Updater;
import org.deeplearning4j.nn.conf.inputs.InputType;
import org.deeplearning4j.nn.conf.layers.ConvolutionLayer;
import org.deeplearning4j.nn.conf.layers.DenseLayer;
import org.deeplearning4j.nn.conf.layers.OutputLayer;
import org.deeplearning4j.nn.conf.layers.SubsamplingLayer;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.deeplearning4j.nn.weights.WeightInit;
import org.deeplearning4j.optimize.listeners.ScoreIterationListener;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.api.buffer.DataBuffer;
import org.nd4j.linalg.api.buffer.util.DataTypeUtil;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.dataset.DataSet;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;
import org.nd4j.linalg.lossfunctions.LossFunctions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.deeplearning4j.parallelism.ParallelWrapper;
```

## Data Set

To obtain the data, we use built-in DataSetIterators for the MNIST with a random seed of 12345. These DataSetIterators can be used to directly feed the data into a neural network.

```scala
val batchSize = 128
val mnistTrain = new MnistDataSetIterator(batchSize,true,12345)
val mnistTest = new MnistDataSetIterator(batchSize,false,12345)
```

## Model Configuration

Next, we set up the neural network configuration using a convolutional configuration and initialize the model.

```scala
val nChannels = 1
val outputNum = 10
val seed = 123

val conf = new NeuralNetConfiguration.Builder()
            .seed(seed)
            .weightInit(WeightInit.XAVIER)
            .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
            .updater(Updater.NESTEROVS)
            .list()
            .layer(0, new ConvolutionLayer.Builder(5, 5)
                //nIn and nOut specify depth. nIn here is the nChannels and nOut is the number of filters to be applied
                .nIn(nChannels)
                .stride(1, 1)
                .nOut(20)
                .activation(Activation.IDENTITY)
                .build())
            .layer(1, new SubsamplingLayer.Builder(SubsamplingLayer.PoolingType.MAX)
                .kernelSize(2,2)
                .stride(2,2)
                .build())
            .layer(2, new ConvolutionLayer.Builder(5, 5)
                //Note that nIn need not be specified in later layers
                .stride(1, 1)
                .nOut(50)
                .activation(Activation.IDENTITY)
                .build())
            .layer(3, new SubsamplingLayer.Builder(SubsamplingLayer.PoolingType.MAX)
                .kernelSize(2,2)
                .stride(2,2)
                .build())
            .layer(4, new DenseLayer.Builder().activation(Activation.RELU)
                .nOut(500).build())
            .layer(5, new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
                .nOut(outputNum)
                .activation(Activation.SOFTMAX)
                .build())
            .setInputType(InputType.convolutionalFlat(28,28,1)) //See note below
            .build()

val model = new MultiLayerNetwork(conf)
model.init()
```

## Parallel Wrapper

Next we need to configure the parallel training with the ParallelWrapper class using the MultiLayerNetwork as the input. The ParallelWrapper will take care of load balancing between different GPUs.

The notion is that the model will be duplicated within the ParallelWrapper. The prespecified number of workers \(in this case 2\) will then train its own model using its data. After a specified number of iterations \(in this case 3\), all models will be averaged and workers will receive duplicate models. The training process will then continue in this way until the model is fully trained.

```scala
val wrapper = new ParallelWrapper.Builder(model)
            .prefetchBuffer(24)
            .workers(2)
            .averagingFrequency(3)
            .reportScoreAfterAveraging(true)
            .build()
```

To train the model, the fit method of the ParallelWrapper is used directly on the DataSetIterator. Because the ParallelWrapper class handles all the training details behind the scenes, it is very simple to parallelize this process using dl4j.

```scala
wrapper.fit(mnistTrain)
```


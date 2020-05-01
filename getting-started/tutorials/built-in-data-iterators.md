# Built-in Data Iterators

Toy datasets are essential for testing hypotheses and getting started with any neural network training process. Deeplearning4j comes with built-in dataset iterators for common datasets, including but not limited to:

* MNIST
* Iris
* TinyImageNet \(subset of ImageNet\)
* CIFAR-10
* Labelled Faces in the Wild
* Curve Fragment Ground-Truth Dataset

These datasets are also used as a baseline for testing other machine learning algorithms. Please remember to use these datasets correctly within the terms of their license \(for example, you must obtain special permission to use ImageNet in a commercial project\).

## What are we going to learn in this tutorial?

Building on what we know about `MultiLayerNetwork` and `ComputationGraph`, we will instantiate a couple data iterators to feed a toy dataset into a neural network for training. This tutorial is focused on training a classifier \(you can also train networks for regression, or use them for unsupervised training via an autoencoder\), and you will also learn how to interpret the output in the console.

## Imports

```scala
import org.nd4j.linalg.activations.Activation
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator
import org.deeplearning4j.datasets.iterator.impl.MnistDataSetIterator
import org.nd4j.evaluation.classification.Evaluation
import org.deeplearning4j.nn.conf.MultiLayerConfiguration
import org.deeplearning4j.nn.conf.NeuralNetConfiguration
import org.nd4j.linalg.learning.config.Nesterovs
import org.deeplearning4j.nn.conf.layers.DenseLayer
import org.deeplearning4j.nn.conf.layers.OutputLayer
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork
import org.deeplearning4j.nn.weights.WeightInit
import org.deeplearning4j.optimize.listeners.ScoreIterationListener
import org.nd4j.linalg.api.ndarray.INDArray
import org.nd4j.linalg.dataset.DataSet
import org.nd4j.linalg.lossfunctions.LossFunctions.LossFunction
import org.slf4j.Logger
import org.slf4j.LoggerFactory
```

## The MNIST classifier network

A `MultiLayerNetwork` can classify MNIST digits. If you are not familiar with MNIST, it is a dataset originally assembled for recognizing hand-written numerals. You can read more about MNIST [here](https://en.wikipedia.org/wiki/MNIST_database).

Once you have imported what you need, set up a basic MultiLayerNetwork like below.

```scala
//number of rows and columns in the input pictures
val numRows = 28
val numColumns = 28
val outputNum = 10 // number of output classes
val batchSize = 128 // batch size for each epoch
val rngSeed = 123 // random number seed for reproducibility
val numEpochs = 15 // number of epochs to perform

val conf: MultiLayerConfiguration = new NeuralNetConfiguration.Builder()
    //include a random seed for reproducibility
    .seed(rngSeed) 
    //specify the learning rate and the rate of change of the learning rate.
    .updater(new Nesterovs(0.006, 0.9))
    .l2(1e-4)
    .list()
    //create the first, input layer with xavier initialization
    .layer(0, new DenseLayer.Builder() 
            .nIn(numRows * numColumns)
            .nOut(1000)
            .activation(Activation.RELU)
            .weightInit(WeightInit.XAVIER)
            .build())
    //create hidden layer
    .layer(1, new OutputLayer.Builder(LossFunction.NEGATIVELOGLIKELIHOOD) 
            .nIn(1000)
            .nOut(outputNum)
            .activation(Activation.SOFTMAX)
            .weightInit(WeightInit.XAVIER)
            .build())
    .build()

val model = new MultiLayerNetwork(conf)
model.init()
//print the score with every 10 iteration
model.setListeners(new ScoreIterationListener(10))
```

### Using the MNIST iterator

The MNIST iterator, like most of Deeplearning4j’s built-in iterators, extends the `DataSetIterator` class. This API allows for simple instantiation of datasets and automatic downloading of data in the background. The MNIST data iterator API specifically allows you to specify whether you are using the training or testing dataset, so instantiate two different iterators to evaluate your network.

```scala
//Get the DataSetIterators:
val mnistTrain = new MnistDataSetIterator(batchSize, true, rngSeed)
val mnistTest = new MnistDataSetIterator(batchSize, false, rngSeed)
```

### Performing basic training

Now that the network configuration is set up and instantiated along with our MNIST test/train iterators, training takes just a few lines of code. The fun begins.

Earlier we attached a `ScoreIterationListener` to the model by using the `setListeners()` method. Depending on the browser you are using to run this notebook, you can open the debugger/inspector to view listener output. This output is redirected to the console since the internals of Deeplearning4j use SLF4J for logging, and the output is being redirected by Zeppelin. This is a good thing since it can reduce clutter in notebooks.

As a well-tuned model continues to train, its error score will decrease with each iteration. This error or loss score will eventually converge to a value close to zero. Note that more complex networks and problems may never yield an optimal score. This is where you need to become the expert and continue to tune and change your model’s configuration.

```scala
// the simplest way to do multiple epochs is to pass them to `fit`
model.fit(mnistTrain, numEpochs)

/* try below if you want to check the current number of epoch
for (i <- 1 to numEpochs) {
    println("Epoch " + i + " / " + numEpochs)
    model.fit(mnistTrain)
}
*/
```

### Evaluating the model

“Overfitting” is a common problem in deep learning where your model doesn’t generalize well to the problem you are trying to solve. This can happen when you have run the algorithm for too many epochs over a training dataset, when you haven’t used a regularization technique like [Dropout](https://en.wikipedia.org/wiki/Dropout_%28neural_networks%29), or the training dataset isn’t big enough and doesn’t encapsulate all of the features that are descriptive of your classes in the real world.

Deeplearning4j comes with built-in tools for model evaluation. The simplest is to pass a testing iterator to `eval()` and retrieve an `Evaluation` object. Many more, including ROC plotting and regression evaluation, are available in the `org.nd4j.evaluation.classification` package.

```scala
val evaluation = model.evaluate[Evaluation](mnistTest)

// print the basic statistics about the trained classifier
println("Accuracy: "+evaluation.accuracy())
println("Precision: "+evaluation.precision())
println("Recall: "+evaluation.recall())

// in more complex scenarios, a confusion matrix is quite helpful
println(evaluation.confusionToString())
```


# Logistic Regression

With deep learning, we can compose a deep neural network to suit the input data and its features. The goal is to train the network on the data to make predictions, and those predictions are tied to the outcomes that you care about; i.e. is this transaction fraudulent or not, or which object is contained in the photo? There are different techniques to configure a neural network, and all of them build a relational hierarchy between the inputs and outputs.

In this tutorial, we are going to configure the simplest neural network and that is logistic regression model network.

Regression is a process that helps show the relations between the independent variables \(inputs\) and the dependent variables \(outputs\). Logistic regression is one in which the dependent variable is categorical rather than continuous - meaning that it can predict only a limited number of classes or categories, like a switch you flip on or off. For example, it can predict that an image contains a cat or a dog, or it can classify input in ten buckets with the integers 0 through 9.

A simple logistic regression calculates `x*w + b = y`. Where `x` is an instance of input data, `w` is the weight or coefficient that transforms that input, `b` is the bias and `y` is the output, or prediction about the data. The biological terms show how this artificial neuron loosely maps to a neuron in the human brain. The most important point is how data flows through and is transformed by this structure.



![  Source:  http://cs231n.github.io/neural-networks-1/](https://i.pinimg.com/736x/61/fe/81/61fe81589ab491d1d3ba612b3bdf5b51--convolutional-neural-network-neuron-model.jpg)

## What will we learn in this tutorial?

We’re going to configure the simplest network, with just one input layer and one output layer, to show how logistic regression works.

## Imports

```scala
import org.deeplearning4j.nn.conf.graph.MergeVertex
import org.deeplearning4j.nn.conf.layers.{DenseLayer, GravesLSTM, OutputLayer, RnnOutputLayer}
import org.deeplearning4j.nn.conf.{ComputationGraphConfiguration, MultiLayerConfiguration, NeuralNetConfiguration}
import org.deeplearning4j.nn.graph.ComputationGraph
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork
import org.deeplearning4j.nn.weights.WeightInit
import org.nd4j.linalg.activations.Activation
import org.nd4j.linalg.learning.config.Nesterovs
import org.nd4j.linalg.lossfunctions.LossFunctions
```

## Configuring logistic regression layers

We are going to first build the layers and then feed these layers into the network configuration.

```scala
//Building the output layer
val outputLayer : OutputLayer = new OutputLayer.Builder()
    .nIn(784) //The number of inputs feed from the input layer
    .nOut(10) //The number of output values the output layer is supposed to take
    .weightInit(WeightInit.XAVIER) //The algorithm to use for weights initialization
    .activation(Activation.SOFTMAX) //Softmax activate converts the output layer into a probability distribution
    .build() //Building our output layer
```

```scala
//Since this is a simple network with a stack of layers we're going to configure a MultiLayerNetwork
val logisticRegressionConf : MultiLayerConfiguration = new NeuralNetConfiguration.Builder()
    //High Level Configuration
    .seed(123)
    .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
    .updater(new Nesterovs(0.1, 0.9)) 
    //For configuring MultiLayerNetwork we call the list method
    .list() 
    .layer(0, outputLayer) //    <----- output layer fed here
    .build() //Building Configuration
```

### Why we didn’t build an input layer

You may be wondering why didn’t we write any code for building our input layer. The input layer is only a set of inputs values fed into the network. It doesn’t perform a calculation. It’s just an input sequence \(raw or pre-processed data\) coming into the network, data to be trained on or to be evaluated. Later, we are going to work with data iterators, which feed input to a network in a specific pattern, and which can be thought of as an input layer of the network.


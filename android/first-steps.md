---
title: Android for Deep Learning
short_title: Android Overview
description: Using Deep Learning and Neural Networks in Android Applications
category: Mobile
weight: 0
---

# Tutorial: First Steps

Generally speaking, training a neural network is a task best suited for powerful computers with multiple GPUs. But what if you want to do it on your humble Android phone or tablet? Well, it’s definitely possible. Considering an average Android device’s specifications, however, it will most likely be quite slow. If that’s not a problem for you, keep reading.

## Setup

For detailed setup instructions see the [Deeplearning4J Android Setup](setup.md) section.

## Starting an Asynchronous Task

Training a neural network is CPU-intensive, which is why you wouldn’t want to do it in your application’s UI thread. I’m not too sure if DL4J trains its networks asynchronously by default. Just to be safe, I’ll spawn a separate thread.

```java
private void startBackgroundThread() {
    Thread thread;
    thread = new Thread(() -> {
        createAndUseNetwork();
        });
    thread.start();
}
```

Because the method createAndUseNetwork\(\) doesn’t exist yet, create it.

```java
private void createAndUseNetwork() {
}
```

## Creating a Neural Network

DL4J has a very intuitive API. Let us now use it to create a simple multi-layer perceptron with hidden layers. It will take two input values, and spit out one output value. To create the layers, we’ll use the DenseLayer and OutputLayer classes. Accordingly, add the following code to the createAndUseNetwork\(\) method you created in the previous step:

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
                .seed(seed)
                .weightInit(WeightInit.XAVIER)
                .updater(Updater.ADAM)
                .list()
                .layer(new DenseLayer.Builder()
                        .nIn(2)
                        .nOut(3)
                        .activation(Activation.RELU)
                        .build())
                .layer(new DenseLayer.Builder()
                        .nOut(2)
                        .activation(Activation.RELU)
                        .build())        
                .layer(new OutputLayer.Builder(LossFunction.NEGATIVELOGLIKELIHOOD)
                        .weightInit(WeightInit.XAVIER)
                        .activation(Activation.SIGMOID)
                        .nOut(1).build())
                .build();


MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
```

## Creating Training Data

To create our training data, we’ll be using the INDArray class, which is provided by ND4J. Here’s what our training data will look like:

```text
INPUTS      EXPECTED OUTPUTS
------      ----------------
0,0         0
0,1         1
1,0         1
1,1         0
```

As you might have guessed, our neural network will behave like an XOR gate. The training data has four samples, and you must mention it in your code.

```java
final int NUM_SAMPLES = 4;
```

And now, create two INDArray objects for the inputs and expected outputs, and initialize them with zeroes.

```java
INDArray trainingInputs = Nd4j.zeros(NUM_SAMPLES, inputLayer.getNIn());
INDArray trainingOutputs = Nd4j.zeros(NUM_SAMPLES, outputLayer.getNOut());
```

Note that the number of columns in the inputs array is equal to the number of neurons in the input layer. Similarly, the number of columns in the outputs array is equal to the number of neurons in the output layer.

Filling those arrays with the training data is easy. Just use the putScalar\(\) method:

```java
// If 0,0 show 0
trainingInputs.putScalar(new int[]{0, 0}, 0);
trainingInputs.putScalar(new int[]{0, 1}, 0);
trainingOutputs.putScalar(new int[]{0, 0}, 0);
// If 0,1 show 1
trainingInputs.putScalar(new int[]{1, 0}, 0);
trainingInputs.putScalar(new int[]{1, 1}, 1);
trainingOutputs.putScalar(new int[]{1, 0}, 1);
// If 1,0 show 1
trainingInputs.putScalar(new int[]{2, 0}, 1);
trainingInputs.putScalar(new int[]{2, 1}, 0);
trainingOutputs.putScalar(new int[]{2, 0}, 1);
// If 1,1 show 0
trainingInputs.putScalar(new int[]{3, 0}, 1);
trainingInputs.putScalar(new int[]{3, 1}, 1);
trainingOutputs.putScalar(new int[]{3, 0}, 0);
```

We won’t be using the INDArray objects directly. Instead, we’ll convert them into a DataSet.

```java
DataSet myData = new DataSet(trainingInputs, trainingOutputs);
```

At this point, we can start the training by calling the `fit()` method of the neural network and passing the data set to it. The `for` loop controls the iterations of the data set through the network. It is set to 1000 iterations in this example.

```java
for(int l=0; l<=1000; l++) {
    myNetwork.fit(myData);
}
```

And that’s all there is to it. Your neural network is ready to be used.

## Conclusion

In this tutorial, you saw how easy it is to create and train a neural network using the Deeplearning4J library in an Android Studio project. I’d like to warn you, however, that training a neural network on a low-powered, battery operated device might not always be a good idea.

A second example DL4J Android Application which includes a user interface can be found [here](linear-classifier.md). This example trains a neural network on the device using Anderson’s iris data set for iris flower type classification. The application includes user input for the measurements and returns the probability that these measurements belong to one of three iris types \(_Iris serosa, Iris versicolor,_ and _Iris virginica_\).

The limitations of processing power and battery life on mobile devices make training robust, multi-layer networks unfeasible. As an alternative to training a network on the device, the neural network being used by your application can be trained on the desktop, saved via ModelSerializer, and then loaded as a pre-trained model in the application. A third example DL4J Android Application can be found [here](image-classification.md) which loads a pre-trained Mnist network and uses it to classify user drawn numbers.


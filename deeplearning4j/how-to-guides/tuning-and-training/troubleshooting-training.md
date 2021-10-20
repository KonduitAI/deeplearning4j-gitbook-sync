---
description: Understanding common errors like NaNs and tuning hyperparameters.
---

# Troubleshooting Training

## Troubleshooting Neural Net Training

Neural networks can be difficult to tune. If the network hyperparameters are poorly chosen, the network may learn slowly, or perhaps not at all. This page aims to provide some baseline steps you should take when tuning your network.

Many of these tips have already been discussed in the academic literature. Our purpose is to consolidate them in one site and express them as clearly as possible.

## Contents

* [Data Normalization](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#data-normalization)
* [Weight Initialization](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#weight-initialization)
* [Epochs and Iterations](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#number-of-epochs-and-number-of-iterations)
* [Learning Rate](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#learning-rate)
* [Activation Function](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#activation-function)
* [Loss Function](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#loss-function)
* [Regularization](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#regularization)
* [Minibatch Size](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#minibatch-size)
* [Updater and Optimization Algorithm](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#updater-and-optimization-algorithm)
* [Gradient Normalization](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#gradient-normalization)
* [Recurrent Neural Networks](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#recurrent-neural-networks-truncated-backpropagation-through-time)
* [Deep Belief Network](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#visible-hidden-unit)
* [NaN, Not a Number issues](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md#nan-not-a-number-errors)

## [Data Normalization](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md)

What's distribution of your data? Are you scaling it properly? As a general rule:

* For continuous values: you want these to be in the range of -1 to 1, 0 to 1 or ditributed normally with mean 0 and standard deviation 1. This does not have to be exact, but ensuring your inputs are approximately in this range can help during training. Scale down large inputs, and scale up small inputs.
* For discrete classes (and, for classification problems for the output), generally use a one-hot representation. That is, if you have 3 classes, then your data will be represeted as \[1,0,0], \[0,1,0] or \[0,0,1] for each of the 3 classes respectively.

Note that it's very important to use the exact same normalization method for both the training data and testing data.

## [Weight Initialization](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md)

Deeplearning4j supports several different kinds of weight initializations with the weightInit parameter. These are set using the .weightInit(WeightInit) method in your configuration.

You need to make sure your weights are neither too big nor too small. Xavier weight initialization is usually a good choice for this. For networks with rectified linear (relu) or leaky relu activations, RELU weight initialization is a sensible choice.

## [Number of Epochs and Number of Iterations](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md)

An epoch is defined as a full pass of the data set.

Too few epochs don't give your network enough time to learn good parameters; too many and you might overfit the training data. One way to choose the number of epochs is to use early stopping. [Early stopping](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/early-stopping.md) can also help to prevent the neural network from overfitting (i.e., can help the net generalize better to unseen data).

## [Learning Rate](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md)

The learning rate is one of, if not the most important hyperparameter. If this is too large or too small, your network may learn very poorly, very slowly, or not at all. Typical values for the learning rate are in the range of 0.1 to 1e-6, though the optimal learning rate is usually data (and network architecture) specific. Some simple advice is to start by trying three different learning rates – 1e-1, 1e-3, and 1e-6 – to get a rough idea of what it should be, before further tuning this. Ideally, they run models with different learning rates simultaneously to save time.

The usual approach to selecting an appropriate learning rate is to use [DL4J's visualization interface](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/visualization.md) to visualize the progress of training. You want to pay attention to both the loss over time, and the ratio of update magnitudes to parameter magnitudes (a ratio of approximately 1:1000 is a good place to start). For more information on tuning the learning rate, see [this link](http://cs231n.github.io/neural-networks-3/#baby).

For training neural networks in a distributed manner, you may need a different (frequently higher) learning rate compared to training the same network on a single machine.

### Policies and Scheduling

You can optionally define a learning rate policy for your neural network. A policy will change the learning rate over time, achieving better results since the learning rate can "slow down" to find closer local minima for convergence. A common policy used is scheduling. See the [LeNet example](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/convolution/LenetMnistExample.java) for a learning rate schedule used in practice.

Note that if you're using multiple GPUs, this will affect your scheduling. For example, if you have 2x GPUs, then you will need to divide the iterations in your schedule by 2, since the throughput of your training process will be double, and the learning rate schedule is only applicable to the local GPU.

## [Activation Function](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md)

There are two aspects to be aware of, with regard to the choice of activation function.

First, the activation function of the hidden (non-output) layers. As a general rule, 'relu' or 'leakyrelu' activations are good choices for this. Some other activation functions (tanh, sigmoid, etc) are more prone to vanishing gradient problems, which can make learning much harder in deep neural networks. However, for LSTM layers, the tanh activation function is still commonly used.

Second, regarding the activation function for the output layer: this is usually application specific. For classification problems, you generally want to use the softmax activation function, combined with the negative log likelihood / MCXENT (multi-class cross entropy). The softmax activation function gives you a probability distribution over classes (i.e., outputs sum to 1.0). For regression problems, the "identity" activation function is frequently a good choice, in conjunction with the MSE (mean squared error) loss function.

## [Loss Function](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md)

Loss functions for each neural network layer can either be used in pretraining, to learn better weights, or in classification (on the output layer) for achieving some result. (In the example above, classification happens in the override section.)

Your net's purpose will determine the loss function you use. For pretraining, choose reconstruction entropy. For classification, use multiclass cross entropy.

## [Regularization](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md)

Regularization methods can help to avoid overfitting during training. Overfitting occurs when the network predicts the training set very well, but makes poor predictions on data the network has never seen. One way to think about overfitting is that the network memorizes the training data (instead of learning the general relationships in it).

Common types of regularization include:

* l1 and l2 regularization penalizes large network weights, and avoids weights becoming too large. Some level of l2 regularization is commonly used in practice. However, note that if the l1 or l2 regularization coefficients are too high, they may over-penalize the network, and stop it from learning. Common values for l2 regularization are 1e-3 to 1e-6.
* [Dropout](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/dropout/Dropout.java), is a frequently used regularization method can be very effective. Dropout is most commoly used with a dropout rate of 0.5.
* Dropconnect (conceptually similar to dropout, but used much less frequently)
* Restricting the total number of network size (i.e., limit the number of layers and size of each layer)
* [Early stopping](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/early-stopping.md)

To use l1/l2/dropout regularization, use .regularization(true) followed by .l1(x), .l2(y), .dropout(z) respectively. Note that z in dropout(z) is the probability of retaining an activation.

## [Minibatch Size](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md)

A minibatch refers to the number of examples used at a time, when computing gradients and parameter updates. In practice (for all but the smallest data sets), it is standard to break your data set up into a number of minibatches.

The ideal minibatch size will vary. For example, a minibatch size of 10 is frequently too small for GPUs, but can work on CPUs. A minibatch size of 1 will allow a network to train, but will not reap the benefits of parallelism. 32 may be a sensible starting point to try, with minibatches in the range of 16-128 (sometimes smaller or larger, depending on the application and type of network) being common.

## [Updater and Optimization Algorithm](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md)

In DL4J, the term 'updater' refers to training mechanisms such as momentum, RMSProp, adagrad, and others. Using one of these methods can result in much faster network training companed to 'vanilla' stochastic gradient descent. You can set the updater using the .updater(Updater) configuration option.

The optimization algorithm is how updates are made, given the gradient. The simplest (and most commonly used) method is stochastic gradient descent (SGD), however DL4J also provides SGD with line search, conjugate gradient and LBFGS optimization algorithms. These latter algorithms are more powerful compared to SGD, but considerably more costly per parameter update due to a line search component, and aren't used as much in practice. Note that you can in principle combine any updater with any optimization algorithm.

A good default choice in most cases is to use the stochastic gradient descent optimization algorithm combined with one of the momentum/rmsprop/adagrad updaters, with momentum frequently being used in practice. Note that for momentum, the updater is called NESTEROVS (a reference to the Nesterovs variant of momentum), and the momentum rate can be set by the .momentum(double) option.

## [Gradient Normalization](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md)

When training a neural network, it can sometimes be helpful to apply gradient normalization, to avoid the gradients being too large (the so-called exploding gradient problem, common in recurrent neural networks) or too small. This can be applied using the .gradientNormalization(GradientNormalization) and .gradientNormalizationThreshould(double) methods. For an example of gradient normalization see, [GradientNormalization.java](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/GradientNormalization.java). The test code for that example is [here](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-core/src/test/java/org/deeplearning4j/nn/updater/TestGradientNormalization.java).

## [Recurrent Neural Networks: Truncated Backpropagation through Time](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md)

When training recurrent networks with long time series, it is generally advisable to use truncated backpropagation through time. With 'standard' backpropagation through time (the default in DL4J) the cost per parameter update can become prohibative. For more details, see [this page](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/models/recurrent.md#truncated-back-propagation-through-time).

## [NaN, Not a Number Errors](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/how-to-guides/tuning-and-training/troubleshooting-training.md)

Q. Why is my Neural Network throwing nan values?

A. Backpropagation involves the multiplication of very small gradients, due to limited precision when representing real numbers values very close to zero can not be represented. The term for this issue is Arithmetic Underflow. If your Neural Network is throwing nan's then the solution is to retune your network to avoid the very small gradients. This is more likely an issue with deeper Neural Networks.

You can try using double data type but it's usually recommended to retune the net first.

Following the basic tuning tips and monitoring the results is the way to ensure NAN doesn't show up anymore.

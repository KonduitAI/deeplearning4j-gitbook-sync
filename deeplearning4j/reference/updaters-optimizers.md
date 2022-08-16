---
description: Special algorithms for gradient descent.
---

# Updaters/Optimizers

## What are updaters?

The main difference among the updaters is how they treat the learning rate. Stochastic Gradient Descent, the most common learning algorithm in deep learning, relies on `Theta` (the weights in hidden layers) and `alpha` (the learning rate). Different updaters help optimize the learning rate until the neural network converges on its most performant state.

## Usage

To use the updaters, pass a new class to the `updater()` method in either a `ComputationGraph` or `MultiLayerNetwork`.

```java
ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .updater(new Adam(0.01))
    // add your layers and hyperparameters below
    .build();
```

## Available updaters

### NadamUpdater

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/NadamUpdater.java)

The Nadam updater. [https://arxiv.org/pdf/1609.04747.pdf](https://arxiv.org/pdf/1609.04747.pdf)

**applyUpdater**

```
public void applyUpdater(INDArray gradient, int iteration, int epoch)
```

Calculate the update based on the given gradient

* param gradient the gradient to get the update for
* param iteration
* return the gradient

### NesterovsUpdater

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/NesterovsUpdater.java)

Nesterov’s momentum. Keep track of the previous layer’s gradient and use it as a way of updating the gradient.

**applyUpdater**

```
public void applyUpdater(INDArray gradient, int iteration, int epoch)
```

Get the nesterov update

* param gradient the gradient to get the update for
* param iteration
* return

### RmsPropUpdater

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/RmsPropUpdater.java)

RMS Prop updates:

[http://www.cs.toronto.edu/\~tijmen/csc321/slides/lecture\_slides\_lec6.pdf](http://www.cs.toronto.edu/\~tijmen/csc321/slides/lecture\_slides\_lec6.pdf) [http://cs231n.github.io/neural-networks-3/#ada](http://cs231n.github.io/neural-networks-3/#ada)

### AdaGradUpdater

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/AdaGradUpdater.java)

Vectorized Learning Rate used per Connection Weight

Adapted from: [http://xcorr.net/2014/01/23/adagrad-eliminating-learning-rates-in-stochastic-gradient-descent](http://xcorr.net/2014/01/23/adagrad-eliminating-learning-rates-in-stochastic-gradient-descent) See also [http://cs231n.github.io/neural-networks-3/#ada](http://cs231n.github.io/neural-networks-3/#ada)

**applyUpdater**

```
public void applyUpdater(INDArray gradient, int iteration, int epoch)
```

Gets feature specific learning rates Adagrad keeps a history of gradients being passed in. Note that each gradient passed in becomes adapted over time, hence the opName adagrad

* param gradient the gradient to get learning rates for
* param iteration

### AdaMaxUpdater

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/AdaMaxUpdater.java)

The AdaMax updater, a variant of Adam. [http://arxiv.org/abs/1412.6980](http://arxiv.org/abs/1412.6980)

**applyUpdater**

```
public void applyUpdater(INDArray gradient, int iteration, int epoch)
```

Calculate the update based on the given gradient

* param gradient the gradient to get the update for
* param iteration
* return the gradient

### NoOpUpdater

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/NoOpUpdater.java)

NoOp updater: gradient updater that makes no changes to the gradient

### AdamUpdater

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/AdamUpdater.java)

The Adam updater. [http://arxiv.org/abs/1412.6980](http://arxiv.org/abs/1412.6980)

**applyUpdater**

```
public void applyUpdater(INDArray gradient, int iteration, int epoch)
```

Calculate the update based on the given gradient

* param gradient the gradient to get the update for
* param iteration
* return the gradient

### AdaDeltaUpdater

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/AdaDeltaUpdater.java)

[http://www.matthewzeiler.com/pubs/googleTR2012/googleTR2012.pdf](http://www.matthewzeiler.com/pubs/googleTR2012/googleTR2012.pdf) [https://arxiv.org/pdf/1212.5701v1.pdf](https://arxiv.org/pdf/1212.5701v1.pdf)

Ada delta updater. More robust adagrad that keeps track of a moving window average of the gradient rather than the every decaying learning rates of adagrad

**applyUpdater**

```
public void applyUpdater(INDArray gradient, int iteration, int epoch)
```

Get the updated gradient for the given gradient and also update the state of ada delta.

* param gradient the gradient to get the updated gradient for
* param iteration
* return the update gradient

### SgdUpdater

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/SgdUpdater.java)

SGD updater applies a learning rate only

### GradientUpdater

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/GradientUpdater.java)

Gradient modifications: Calculates an update and tracks related information for gradient changes over time for handling updates.

### AMSGradUpdater

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/AMSGradUpdater.java)

The AMSGrad updater\
Reference: On the Convergence of Adam and Beyond - [https://openreview.net/forum?id=ryQu7f-RZ](https://openreview.net/forum?id=ryQu7f-RZ)

---
description: Special algorithms for gradient descent.
---

# Activations

## What are activations?

At a simple level, activation functions help decide whether a neuron should be activated. This helps determine whether the information that the neuron is receiving is relevant for the input. The activation function is a non-linear transformation that happens over an input signal, and the transformed output is sent to the next neuron.

## Usage

The recommended method to use activations is to add an activation layer in your neural network, and configure your desired activation:

```java
GraphBuilder graphBuilder = new NeuralNetConfiguration.Builder()
    // add hyperparameters and other layers
    .addLayer("softmax", new ActivationLayer(Activation.SOFTMAX), "previous_input")
    // add more layers and output
    .build();
```

## Available activations

### ActivationRectifiedTanh

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationRectifiedTanh.java)

Rectified tanh

Essentially max(0, tanh(x))

Underlying implementation is in native code

## ActivationELU

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationELU.java)

f(x) = alpha (exp(x) - 1.0); x < 0 = x ; x>= 0

alpha defaults to 1, if not specified

## ActivationReLU

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationReLU.java)

f(x) = max(0, x)

## ActivationRationalTanh

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationRationalTanh.java)

Rational tanh approximation From [https://arxiv.org/pdf/1508.01292v3](https://arxiv.org/pdf/1508.01292v3)

f(x) = 1.7159 tanh(2x/3) where tanh is approximated as follows, tanh(y) \~ sgn(y) { 1 - 1/(1+|y|+y^2+1.41645y^4)}

Underlying implementation is in native code

## ActivationThresholdedReLU

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationThresholdedReLU.java)

Thresholded RELU

f(x) = x for x > theta, f(x) = 0 otherwise. theta defaults to 1.0

## ActivationReLU6

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationReLU6.java)

f(x) = min(max(input, cutoff), 6)

## ActivationHardTanH

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationHardTanH.java)

```java
          ⎧  1, if x >  1
 f(x) =   ⎨ -1, if x < -1
          ⎩  x, otherwise
```

## ActivationSigmoid

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationSigmoid.java)

f(x) = 1 / (1 + exp(-x))

## ActivationGELU

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationGELU.java)

GELU activation function - Gaussian Error Linear Units

## ActivationPReLU

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationPReLU.java)

/ Parametrized Rectified Linear Unit (PReLU)

f(x) = alpha x for x < 0, f(x) = x for x >= 0

alpha has the same shape as x and is a learned parameter.

## ActivationIdentity

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationIdentity.java)

f(x) = x

## ActivationSoftSign

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationSoftSign.java)

| f\_i(x) = x\_i / (1+ | x\_i | ) |
| -------------------- | ---- | - |

## ActivationHardSigmoid

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationHardSigmoid.java)

f(x) = min(1, max(0, 0.2x + 0.5))

## ActivationSoftmax

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationSoftmax.java)

f\_i(x) = exp(x\_i - shift) / sum\_j exp(x\_j - shift) where shift = max\_i(x\_i)

## ActivationCube

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationCube.java)

f(x) = x^3

## ActivationRReLU

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationRReLU.java)

f(x) = max(0,x) + alpha min(0, x)

alpha is drawn from uniform(l,u) during training and is set to l+u/2 during test l and u default to 1/8 and 1/3 respectively

[Empirical Evaluation of Rectified Activations in Convolutional Network](https://arxiv.org/abs/1505.00853)

## ActivationTanH

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationTanH.java)

f(x) = (exp(x) - exp(-x)) / (exp(x) + exp(-x))

## ActivationSELU

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationSELU.java)

[https://arxiv.org/pdf/1706.02515.pdf](https://arxiv.org/pdf/1706.02515.pdf)

## ActivationLReLU

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationLReLU.java)

Leaky RELU f(x) = max(0, x) + alpha min(0, x) alpha defaults to 0.01

## ActivationSwish

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationSwish.java)

f(x) = x sigmoid(x)

## ActivationSoftPlus

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationSoftPlus.java)

f(x) = log(1+e^x)

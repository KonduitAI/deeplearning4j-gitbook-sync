---
title: "Activations"
description: "Activation functions in ND4J — the Activation enum, IActivation interface, mathematical definitions, and usage in layers"
---

# Activation Functions

Activation functions introduce non-linearity into neural networks. Without them, stacking multiple layers would be equivalent to a single linear transformation, regardless of depth. ND4J provides all activation functions through a common `Activation` enum and the `IActivation` interface.

## Using Activations

### In Layer Configuration

The most common way to use activations is through the `Activation` enum when configuring layers:

```java
import org.nd4j.linalg.activations.Activation;

new DenseLayer.Builder()
    .nIn(784).nOut(256)
    .activation(Activation.RELU)
    .build()
```

### As a Standalone Layer

You can use `ActivationLayer` when you want the activation separate from the linear transformation:

```java
import org.deeplearning4j.nn.conf.layers.ActivationLayer;

.addLayer("relu", new ActivationLayer(Activation.RELU), "dense1")
```

### Directly on INDArrays

Apply activations to raw tensors using the `Transforms` class:

```java
import org.nd4j.linalg.ops.transforms.Transforms;

INDArray x = Nd4j.create(new double[]{-2, -1, 0, 1, 2});

INDArray relu = Transforms.relu(x, false);       // copy
// [0, 0, 0, 1, 2]

INDArray sigmoid = Transforms.sigmoid(x, false);  // copy
// [0.1192, 0.2689, 0.5, 0.7311, 0.8808]

INDArray tanh = Transforms.tanh(x, false);        // copy
// [-0.9640, -0.7616, 0, 0.7616, 0.9640]
```

The second argument controls in-place behavior: `true` modifies `x` directly, `false` returns a copy.

### Via the IActivation Interface

For programmatic access, get the `IActivation` instance from the enum:

```java
import org.nd4j.linalg.activations.IActivation;

IActivation reluFn = Activation.RELU.getActivationFunction();
INDArray activated = reluFn.getActivation(input.dup(), true);
```

## Available Activations

All activations are in the `org.nd4j.linalg.activations.Activation` enum. Implementations are in `org.nd4j.linalg.activations.impl`.

### ReLU Family

| Activation | Enum Value | Formula | Notes |
|-----------|-----------|---------|-------|
| ReLU | `RELU` | f(x) = max(0, x) | Default choice for hidden layers. Fast, effective, but can suffer from "dying ReLU" |
| Leaky ReLU | `LEAKYRELU` | f(x) = max(alpha * x, x), alpha=0.01 | Avoids dying ReLU by allowing small negative gradients |
| RReLU | `RRELU` | f(x) = max(alpha * x, x), alpha ~ U(l,u) | Randomized leaky ReLU. l=1/8, u=1/3 by default. Uses (l+u)/2 at test time |
| ReLU6 | `RELU6` | f(x) = min(max(0, x), 6) | Capped ReLU for mobile/quantized networks |
| Thresholded ReLU | `THRESHOLDEDRELU` | f(x) = x if x > theta, 0 otherwise. theta=1.0 | Sparse activations |
| PReLU | Via `PReLULayer` | f(x) = max(alpha * x, x), alpha learned | Parametric ReLU — alpha is a trainable parameter |
| ELU | `ELU` | f(x) = x if x >= 0, alpha*(exp(x)-1) if x < 0. alpha=1.0 | Smooth alternative to ReLU with negative values |
| SELU | `SELU` | f(x) = lambda * (x if x >= 0, alpha*(exp(x)-1) if x < 0) | Self-normalizing. Use with `WeightInit.LECUN_NORMAL` and `AlphaDropout` |

### Smooth Activations

| Activation | Enum Value | Formula | Notes |
|-----------|-----------|---------|-------|
| GELU | `GELU` | f(x) = x * Phi(x), where Phi is the Gaussian CDF | Used in Transformers (BERT, GPT). Smooth approximation of ReLU |
| Mish | `MISH` | f(x) = x * tanh(softplus(x)) | Self-regularized, smooth. Good general-purpose alternative to ReLU |
| Swish | `SWISH` | f(x) = x * sigmoid(x) | Smooth, non-monotonic. Discovered via neural architecture search |
| Softplus | `SOFTPLUS` | f(x) = log(1 + exp(x)) | Smooth approximation of ReLU |

### Sigmoid Family

| Activation | Enum Value | Formula | Notes |
|-----------|-----------|---------|-------|
| Sigmoid | `SIGMOID` | f(x) = 1 / (1 + exp(-x)) | Output range (0,1). Use for binary classification output |
| Hard Sigmoid | `HARDSIGMOID` | f(x) = min(1, max(0, 0.2x + 0.5)) | Fast piecewise linear approximation of sigmoid |

### Tanh Family

| Activation | Enum Value | Formula | Notes |
|-----------|-----------|---------|-------|
| Tanh | `TANH` | f(x) = (exp(x) - exp(-x)) / (exp(x) + exp(-x)) | Output range (-1,1). Alternative to ReLU for hidden layers, especially RNNs |
| Hard Tanh | `HARDTANH` | f(x) = -1 if x < -1, 1 if x > 1, x otherwise | Fast piecewise linear approximation of tanh |
| Rectified Tanh | `RECTIFIEDTANH` | f(x) = max(0, tanh(x)) | Combination of ReLU and tanh |
| Rational Tanh | `RATIONALTANH` | f(x) = 1.7159 * tanh(2x/3) with rational approximation | Fast approximation from LeCun 1998 |

### Output Activations

| Activation | Enum Value | Formula | Notes |
|-----------|-----------|---------|-------|
| Softmax | `SOFTMAX` | f_i(x) = exp(x_i - max) / sum_j(exp(x_j - max)) | Multi-class classification output. Outputs sum to 1. Pair with `LossMCXENT` |
| Identity | `IDENTITY` | f(x) = x | Regression output (linear). Pair with `LossMSE` |

### Other Activations

| Activation | Enum Value | Formula | Notes |
|-----------|-----------|---------|-------|
| Softsign | `SOFTSIGN` | f(x) = x / (1 + \|x\|) | Alternative to tanh — converges polynomially instead of exponentially |
| Cube | `CUBE` | f(x) = x^3 | Rarely used in practice |

## Recommended Pairings

Choosing the right activation depends on the layer type and task:

### Hidden Layers

| Network Type | Activation | Weight Init | Why |
|-------------|-----------|-------------|-----|
| Feed-forward, CNN | `RELU` | `WeightInit.RELU` | Fast convergence, avoids vanishing gradient |
| RNN (LSTM, GRU) | `TANH` | `WeightInit.XAVIER` | Bounded output prevents exploding activations in recurrence |
| Self-normalizing nets | `SELU` | `WeightInit.LECUN_NORMAL` | Maintains mean=0, variance=1 through layers |
| Transformer blocks | `GELU` | `WeightInit.XAVIER` | Smooth, works well with attention mechanisms |

### Output Layers

| Task | Activation | Loss Function | Why |
|------|-----------|---------------|-----|
| Multi-class classification | `SOFTMAX` | `LossMCXENT` | Outputs are valid probabilities summing to 1 |
| Binary classification | `SIGMOID` | `LossBinaryXENT` | Output is probability in (0,1) |
| Multi-label classification | `SIGMOID` | `LossBinaryXENT` | Each output is independent binary decision |
| Regression | `IDENTITY` | `LossMSE` or `LossMAE` | Linear output, no bounds |
| Bounded regression [0,1] | `SIGMOID` | `LossMSE` | Output constrained to (0,1) |

## Custom Activations

Implement the `IActivation` interface at `org.nd4j.linalg.activations.IActivation`:

```java
import org.nd4j.linalg.activations.BaseActivationFunction;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.common.primitives.Pair;

public class CustomActivation extends BaseActivationFunction {

    @Override
    public INDArray getActivation(INDArray in, boolean training) {
        // Modify 'in' in-place and return it
        // Example: f(x) = x * sigmoid(x)  (Swish)
        INDArray sigmoid = Transforms.sigmoid(in.dup(), false);
        in.muli(sigmoid);
        return in;
    }

    @Override
    public Pair<INDArray, INDArray> backprop(INDArray in, INDArray epsilon) {
        // Compute activation gradient and multiply by upstream gradient
        // Return: Pair(gradient, null)
        // The second element is null for activations without learnable parameters
        INDArray gradient = computeGradient(in);  // your derivative
        gradient.muli(epsilon);
        return new Pair<>(gradient, null);
    }
}
```

Use a custom activation in a layer:

```java
new DenseLayer.Builder()
    .nIn(256).nOut(128)
    .activation(new CustomActivation())
    .build()
```

## Activation Function Comparison

For quick reference, key properties of each activation:

| Activation | Range | Monotonic | Smooth | Zero-Centered | Computational Cost |
|-----------|-------|-----------|--------|---------------|-------------------|
| ReLU | [0, inf) | Yes | No | No | Very Low |
| Leaky ReLU | (-inf, inf) | Yes | No | Yes | Very Low |
| ELU | (-alpha, inf) | Yes | Yes | ~Yes | Medium |
| SELU | (-lambda*alpha, inf) | Yes | Yes | ~Yes | Medium |
| GELU | ~(-0.17, inf) | No | Yes | No | High |
| Mish | ~(-0.31, inf) | No | Yes | No | High |
| Swish | ~(-0.28, inf) | No | Yes | No | Medium |
| Sigmoid | (0, 1) | Yes | Yes | No | Medium |
| Tanh | (-1, 1) | Yes | Yes | Yes | Medium |
| Softmax | (0, 1) per class | N/A | Yes | No | Medium |
| Identity | (-inf, inf) | Yes | Yes | Yes | None |

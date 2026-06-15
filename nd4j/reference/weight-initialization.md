---
title: "Weight Initialization"
description: "Weight initialization strategies in ND4J — WeightInit enum, IWeightInit interface, and choosing the right initializer"
---

# Weight Initialization

Weight initialization determines the starting values for a layer's parameters (weights and biases) before training begins. Proper initialization is critical — poor choices lead to vanishing gradients (values shrink to zero through layers), exploding gradients (values grow unbounded), or slow convergence.

## Usage

### In Layer Configuration

Set weight initialization using the `WeightInit` enum:

```java
import org.nd4j.weightinit.WeightInit;

new DenseLayer.Builder()
    .nIn(784).nOut(256)
    .weightInit(WeightInit.RELU)
    .activation(Activation.RELU)
    .build()
```

### As a Global Default

Set in the network configuration builder (applies to all layers unless overridden):

```java
new NeuralNetConfiguration.Builder()
    .weightInit(WeightInit.XAVIER)     // default for all layers
    .list()
    .layer(new DenseLayer.Builder()
        .nIn(784).nOut(256)
        .weightInit(WeightInit.RELU)   // override for this layer
        .build())
    .build();
```

### With a Custom Distribution

Use `WeightInit.DISTRIBUTION` with a specific distribution:

```java
new DenseLayer.Builder()
    .nIn(784).nOut(256)
    .weightInit(new NormalDistribution(0, 0.01))
    .build()
```

## The WeightInit Enum

All initializers are in `org.nd4j.weightinit.WeightInit`.

### Xavier Family (Glorot)

Designed for sigmoid and tanh activations. Keeps the variance of activations roughly constant across layers.

| Initializer | Enum Value | Distribution | Notes |
|------------|-----------|-------------|-------|
| Xavier (Glorot) Normal | `XAVIER` | N(0, 2/(fanIn + fanOut)) | Default. Best for tanh/sigmoid |
| Xavier Uniform | `XAVIER_UNIFORM` | U(-sqrt(6/(fanIn+fanOut)), sqrt(6/(fanIn+fanOut))) | Uniform variant of Xavier |

**fanIn** = number of input connections, **fanOut** = number of output connections.

### He Family (Kaiming)

Designed for ReLU and its variants. Accounts for the fact that ReLU zeros out half the distribution.

| Initializer | Enum Value | Distribution | Notes |
|------------|-----------|-------------|-------|
| He Normal | `RELU` | N(0, 2/fanIn) | Best for ReLU, Leaky ReLU, ELU |
| He Uniform | `RELU_UNIFORM` | U(-sqrt(6/fanIn), sqrt(6/fanIn)) | Uniform variant for ReLU |

### LeCun Family

Designed for SELU activations in self-normalizing networks.

| Initializer | Enum Value | Distribution | Notes |
|------------|-----------|-------------|-------|
| LeCun Normal | `LECUN_NORMAL` | N(0, 1/fanIn) | Use with SELU activation |
| LeCun Uniform | `LECUN_UNIFORM` | U(-sqrt(3/fanIn), sqrt(3/fanIn)) | Uniform variant for SELU |

### Variance Scaling

General-purpose initializers that let you choose the fan mode:

| Initializer | Enum Value | Distribution |
|------------|-----------|-------------|
| Fan-In Normal | `VAR_SCALING_NORMAL_FAN_IN` | N(0, 1/fanIn) |
| Fan-Out Normal | `VAR_SCALING_NORMAL_FAN_OUT` | N(0, 1/fanOut) |
| Fan-Avg Normal | `VAR_SCALING_NORMAL_FAN_AVG` | N(0, 2/(fanIn + fanOut)) |
| Fan-In Uniform | `VAR_SCALING_UNIFORM_FAN_IN` | U(-sqrt(3/fanIn), sqrt(3/fanIn)) |
| Fan-Out Uniform | `VAR_SCALING_UNIFORM_FAN_OUT` | U(-sqrt(3/fanOut), sqrt(3/fanOut)) |
| Fan-Avg Uniform | `VAR_SCALING_UNIFORM_FAN_AVG` | U(-sqrt(6/(fanIn+fanOut)), sqrt(6/(fanIn+fanOut))) |

### Simple Initializers

| Initializer | Enum Value | Description |
|------------|-----------|-------------|
| Normal | `NORMAL` | N(0, 1) — standard normal. Rarely used alone; variance is too high for most networks |
| Uniform | `UNIFORM` | U(-1, 1) — uniform between -1 and 1. Rarely used alone |
| Zero | `ZERO` | All zeros. Use for biases if desired |
| Ones | `ONES` | All ones |
| Constant | `CONSTANT` | All set to a user-specified value |
| Identity | `IDENTITY` | Identity matrix. Only works for square layers (nIn == nOut) |

### Special Initializers

| Initializer | Enum Value | Description |
|------------|-----------|-------------|
| Distribution | `DISTRIBUTION` | Use a custom distribution (NormalDistribution, UniformDistribution, etc.) |
| Supplied | `SUPPLIED` | Use a user-provided INDArray directly |

## Choosing the Right Initializer

### Rules of Thumb

| Activation Function | Recommended WeightInit | Why |
|--------------------|----------------------|-----|
| ReLU, Leaky ReLU, ELU | `RELU` (He) | Compensates for ReLU zeroing half the distribution |
| Tanh, Sigmoid | `XAVIER` (Glorot) | Balances variance for symmetric activations |
| SELU | `LECUN_NORMAL` | Required for the self-normalizing property |
| GELU, Mish, Swish | `XAVIER` or `RELU` | Both work; `XAVIER` is slightly more common |
| Softmax (output) | `XAVIER` | Standard choice for classification output |
| Identity (regression) | `XAVIER` | Standard choice for regression output |

### What Happens with Bad Initialization

**Too small (e.g., `ZERO` for weights):**
- All neurons compute the same output ("symmetry problem")
- Gradients are identical for all weights — network cannot learn different features
- Loss does not decrease

**Too large (e.g., `NORMAL` without scaling):**
- Activations saturate (sigmoid/tanh outputs are all near -1/+1)
- Gradients vanish due to saturation
- With ReLU: exploding activations, NaN loss

**Just right (e.g., `XAVIER` or `RELU`):**
- Activations have roughly constant variance across layers
- Gradients flow well through the network
- Training converges efficiently

## Using a Custom Distribution

For fine-grained control, pass a distribution object:

```java
import org.nd4j.linalg.api.rng.distribution.impl.*;

// Normal distribution with specific mean and std
new DenseLayer.Builder()
    .nIn(784).nOut(256)
    .weightInit(new NormalDistribution(0, 0.02))
    .build()

// Uniform distribution with specific bounds
new DenseLayer.Builder()
    .nIn(784).nOut(256)
    .weightInit(new UniformDistribution(-0.05, 0.05))
    .build()

// Truncated normal (values clipped to 2 standard deviations)
new DenseLayer.Builder()
    .nIn(784).nOut(256)
    .weightInit(new TruncatedNormalDistribution(0, 0.02))
    .build()
```

## Supplying Custom Weight Arrays

Use `WeightInit.SUPPLIED` when you need exact control over initial weights — for example, when loading pretrained weights outside of the standard model serialization:

```java
// Not commonly needed — usually use ModelSerializer for pretrained weights
INDArray customWeights = Nd4j.rand(DataType.FLOAT, 784, 256).mul(0.01);

// Set via the model after initialization
model.init();
model.setParam("0_W", customWeights);
```

## Custom IWeightInit

For reusable custom initialization logic, implement `IWeightInit`:

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.weightinit.BaseWeightInitScheme;

public class OrthogonalInit extends BaseWeightInitScheme {

    private final double gain;

    public OrthogonalInit(char order, double gain) {
        super(order);
        this.gain = gain;
    }

    @Override
    public INDArray doCreate(long[] shape, INDArray paramsView) {
        // Generate random matrix
        INDArray random = Nd4j.randn(DataType.FLOAT, shape);

        // Compute SVD for orthogonal initialization
        // (simplified — full implementation would use nd4j linear algebra)
        long rows = shape[0];
        long cols = shape.length > 1 ? shape[1] : 1;

        if (rows < cols) {
            random = random.transpose();
        }
        // ... SVD decomposition and orthogonalization ...

        random.muli(gain);

        if (paramsView != null && paramsView.length() == random.length()) {
            paramsView.assign(random.reshape(paramsView.shape()));
            return paramsView;
        }
        return random;
    }
}
```

Use it in a layer builder:

```java
new DenseLayer.Builder()
    .nIn(256).nOut(256)
    .weightInit(new OrthogonalInit('c', 1.0))
    .build()
```

## Bias Initialization

By default, biases are initialized to zero. This is generally appropriate — the weights provide asymmetry, and zero biases don't create any initial preference.

For specific cases where non-zero bias initialization helps:

```java
// After model.init(), set bias values directly
model.init();
INDArray bias = Nd4j.ones(256).mul(0.1);
model.setParam("0_b", bias);
```

A common pattern for ReLU networks is to initialize biases to a small positive value (0.01 or 0.1) to ensure all ReLU units are active initially. However, the benefit is marginal with proper weight initialization.

## Initialization and Reproducibility

Weight initialization depends on the random number generator. For reproducible initialization, set a seed:

```java
new NeuralNetConfiguration.Builder()
    .seed(42)                        // ensures reproducible initialization
    .weightInit(WeightInit.XAVIER)
    .list()
    // ...
    .build();
```

The seed ensures that calling `model.init()` produces identical weight values every time, which is essential for debugging and comparing experiments.

## Quick Reference

| Scenario | WeightInit | Activation |
|----------|-----------|-----------|
| General-purpose CNN | `RELU` | `RELU` |
| RNN / LSTM | `XAVIER` | `TANH` |
| Self-normalizing network | `LECUN_NORMAL` | `SELU` |
| Transformer layers | `XAVIER` | `GELU` |
| Output (classification) | `XAVIER` | `SOFTMAX` |
| Output (regression) | `XAVIER` | `IDENTITY` |
| Pretrained / frozen layer | N/A (loaded) | varies |
| Fine-tuned from pretrained | Original init (loaded) | varies |

---
title: Arbiter Layer Spaces
description: Layer parameter spaces in Arbiter — configuration of per-layer hyperparameter search spaces for DL4J neural networks.
---

## Overview

Layer spaces are the building blocks of an Arbiter `MultiLayerSpace` or `ComputationGraphSpace`. Each layer space mirrors the corresponding DL4J layer builder, but instead of fixed values, each configurable parameter accepts either a concrete value or a `ParameterSpace<T>` that defines a range to sample from.

All layer spaces extend `LayerSpace<T>` from the `arbiter-deeplearning4j` module. They share a common set of inherited parameters and add layer-specific ones.

---

## Common Parameters (All Layers)

The following parameters are available on every layer space via `BaseLayerSpace`:

| Parameter | Type | Description |
|---|---|---|
| `nIn` | `ParameterSpace<Integer>` or `int` | Input size |
| `nOut` | `ParameterSpace<Integer>` or `int` | Output size |
| `activation` | `ParameterSpace<Activation>` or `Activation` | Activation function |
| `weightInit` | `ParameterSpace<WeightInit>` or `WeightInit` | Weight initialization strategy |
| `l1` | `ParameterSpace<Double>` or `double` | L1 regularization coefficient |
| `l2` | `ParameterSpace<Double>` or `double` | L2 regularization coefficient |
| `dropOut` | `ParameterSpace<Double>` or `double` | Dropout rate |
| `updater` | `ParameterSpace<IUpdater>` or `IUpdater` | Parameter updater |

---

## Dense and Feed-Forward Layers

### DenseLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/DenseLayerSpace.java)

Hyperparameter search space for fully connected (multi-layer perceptron) layers.

```java
DenseLayerSpace denseSpace = new DenseLayerSpace.Builder()
    .nIn(784)
    .nOut(new IntegerParameterSpace(64, 512))
    .activation(new DiscreteParameterSpace<>(Activation.RELU, Activation.TANH))
    .l2(new ContinuousParameterSpace(1e-5, 1e-3))
    .dropOut(new ContinuousParameterSpace(0.0, 0.5))
    .build();
```

### OutputLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/OutputLayerSpace.java)

Hyperparameter search space for output layers (classification and regression).

```java
OutputLayerSpace outputSpace = new OutputLayerSpace.Builder()
    .nOut(10)
    .activation(Activation.SOFTMAX)
    .lossFunction(LossFunctions.LossFunction.MCXENT)
    .build();
```

Additional parameters:
- `lossFunction` — `ParameterSpace<ILossFunction>` or `LossFunctions.LossFunction`
- `iLossFunction` — `ParameterSpace<ILossFunction>` for custom loss function instances

Example with a variable loss function:

```java
ILossFunction[] losses = {
    new LossMCXENT(Nd4j.create(new double[]{1.0, 2.0})),
    new LossMCXENT(Nd4j.create(new double[]{1.0, 5.0}))
};

OutputLayerSpace outputSpace = new OutputLayerSpace.Builder()
    .nOut(2)
    .activation(Activation.SOFTMAX)
    .iLossFunction(new DiscreteParameterSpace<>(losses))
    .build();
```

### EmbeddingLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/EmbeddingLayerSpace.java)

Hyperparameter search space for embedding layers (integer-index input to dense vector).

```java
EmbeddingLayerSpace embeddingSpace = new EmbeddingLayerSpace.Builder()
    .nIn(vocabSize)
    .nOut(new IntegerParameterSpace(32, 256))
    .build();
```

### ActivationLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/ActivationLayerSpace.java)

Hyperparameter space for standalone activation layers (no learnable parameters).

```java
ActivationLayerSpace activationSpace = new ActivationLayerSpace.Builder()
    .activation(new DiscreteParameterSpace<>(Activation.RELU, Activation.ELU, Activation.LEAKYRELU))
    .build();
```

---

## Convolutional Layers

### ConvolutionLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/ConvolutionLayerSpace.java)

Hyperparameter search space for 2D convolutional layers.

| Parameter | Type | Description |
|---|---|---|
| `nOut` | `ParameterSpace<Integer>` | Number of output feature maps (filters) |
| `kernelSize` | `ParameterSpace<int[]>` or `int[]` | Kernel height and width |
| `stride` | `ParameterSpace<int[]>` or `int[]` | Stride height and width |
| `padding` | `ParameterSpace<int[]>` or `int[]` | Padding height and width |
| `cudnnAlgoMode` | `ParameterSpace<ConvolutionLayer.AlgoMode>` | cuDNN algorithm selection |

```java
ConvolutionLayerSpace convSpace = new ConvolutionLayerSpace.Builder()
    .nOut(new IntegerParameterSpace(32, 128))
    .kernelSize(new DiscreteParameterSpace<>(
        new int[]{3, 3},
        new int[]{5, 5}
    ))
    .activation(Activation.RELU)
    .build();
```

### SubsamplingLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/SubsamplingLayerSpace.java)

Hyperparameter search space for pooling layers.

| Parameter | Type | Description |
|---|---|---|
| `poolingType` | `ParameterSpace<PoolingType>` | MAX, AVG, SUM, or PNORM pooling |
| `kernelSize` | `ParameterSpace<int[]>` | Pooling kernel size |
| `stride` | `ParameterSpace<int[]>` | Pooling stride |

```java
SubsamplingLayerSpace poolSpace = new SubsamplingLayerSpace.Builder()
    .poolingType(new DiscreteParameterSpace<>(PoolingType.MAX, PoolingType.AVG))
    .kernelSize(2, 2)
    .stride(2, 2)
    .build();
```

### GlobalPoolingLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/GlobalPoolingLayerSpace.java)

Hyperparameter space for global pooling layers (reduces spatial dimensions to a single vector).

```java
GlobalPoolingLayerSpace globalPoolSpace = new GlobalPoolingLayerSpace.Builder()
    .poolingType(new DiscreteParameterSpace<>(PoolingType.MAX, PoolingType.AVG))
    .build();
```

---

## Recurrent Layers

### LSTMLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/LSTMLayerSpace.java)

Hyperparameter search space for LSTM recurrent layers.

| Parameter | Type | Description |
|---|---|---|
| `nOut` | `ParameterSpace<Integer>` | LSTM hidden state size |
| `activation` | `ParameterSpace<Activation>` | Activation for gates |
| `gateActivationFn` | `ParameterSpace<IActivation>` | Gate activation function |
| `forgetGateBiasInit` | `ParameterSpace<Double>` | Forget gate bias initialization value |

```java
LSTMLayerSpace lstmSpace = new LSTMLayerSpace.Builder()
    .nOut(new IntegerParameterSpace(64, 256))
    .activation(Activation.TANH)
    .dropOut(new ContinuousParameterSpace(0.0, 0.3))
    .build();
```

### GravesLSTMLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/GravesLSTMLayerSpace.java)

Hyperparameter space for the Graves variant of LSTM. Same parameters as `LSTMLayerSpace`. Use `LSTMLayerSpace` for new code; `GravesLSTMLayerSpace` is retained for backward compatibility.

### GravesBidirectionalLSTMLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/GravesBidirectionalLSTMLayerSpace.java)

Hyperparameter space for bidirectional Graves LSTM layers.

```java
GravesBidirectionalLSTMLayerSpace biLstmSpace = new GravesBidirectionalLSTMLayerSpace.Builder()
    .nOut(new IntegerParameterSpace(64, 256))
    .activation(Activation.TANH)
    .build();
```

### RnnOutputLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/RnnOutputLayerSpace.java)

Hyperparameter space for `RnnOutputLayer`, the time-distributed output layer for recurrent networks.

```java
RnnOutputLayerSpace rnnOutSpace = new RnnOutputLayerSpace.Builder()
    .nOut(numClasses)
    .activation(Activation.SOFTMAX)
    .lossFunction(LossFunctions.LossFunction.MCXENT)
    .build();
```

### Bidirectional

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/Bidirectional.java)

A wrapper that converts any recurrent layer space into a bidirectional counterpart:

```java
LSTMLayerSpace underlying = new LSTMLayerSpace.Builder()
    .nOut(new IntegerParameterSpace(64, 256))
    .activation(Activation.TANH)
    .build();

Bidirectional biDirectionalSpace = new Bidirectional(underlying);
```

---

## Normalization and Regularization Layers

### BatchNormalizationSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/BatchNormalizationSpace.java)

Hyperparameter search space for batch normalization layers.

| Parameter | Type | Description |
|---|---|---|
| `decay` | `ParameterSpace<Double>` | Running mean/variance decay factor |
| `eps` | `ParameterSpace<Double>` | Numerical stability epsilon |
| `isMinibatch` | `ParameterSpace<Boolean>` | Whether to use minibatch statistics |
| `lockGammaBeta` | `ParameterSpace<Boolean>` | Lock gamma and beta parameters |

```java
BatchNormalizationSpace bnSpace = new BatchNormalizationSpace.Builder()
    .decay(new ContinuousParameterSpace(0.9, 0.999))
    .eps(1e-5)
    .build();
```

---

## Autoencoder Layers

### AutoEncoderLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/AutoEncoderLayerSpace.java)

Layer space for restricted Boltzmann machine / denoising autoencoder layers.

```java
AutoEncoderLayerSpace aeSpace = new AutoEncoderLayerSpace.Builder()
    .nOut(new IntegerParameterSpace(64, 256))
    .activation(Activation.RELU)
    .build();
```

### VariationalAutoencoderLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/VariationalAutoencoderLayerSpace.java)

Layer space for variational autoencoder layers.

```java
VariationalAutoencoderLayerSpace vaeSpace = new VariationalAutoencoderLayerSpace.Builder()
    .nOut(new IntegerParameterSpace(32, 128))
    .encoderLayerSizes(new IntegerParameterSpace(128, 512))
    .decoderLayerSizes(new IntegerParameterSpace(128, 512))
    .build();
```

---

## Specialized Layers

### OCNNLayerSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-deeplearning4j/src/main/java/org/deeplearning4j/arbiter/layers/OCNNLayerSpace.java)

Layer space for One-Class Neural Network layers (unsupervised anomaly detection).

| Parameter | Type | Description |
|---|---|---|
| `hiddenLayerSize` | `ParameterSpace<Integer>` | Size of hidden representation |
| `nu` | `ParameterSpace<Double>` | Fraction of outliers parameter |
| `activationFunctionHidden` | `ParameterSpace<IActivation>` | Activation for hidden layer |

```java
OCNNLayerSpace ocnnSpace = new OCNNLayerSpace.Builder()
    .hiddenLayerSize(new IntegerParameterSpace(32, 128))
    .nu(new ContinuousParameterSpace(0.01, 0.5))
    .build();
```

---

## Full Example: MultiLayerSpace with Mixed Layer Types

```java
MultiLayerSpace mls = new MultiLayerSpace.Builder()
    .seed(42)
    .updater(new AdamSpace(new ContinuousParameterSpace(1e-4, 3e-3)))
    .l2(new ContinuousParameterSpace(1e-6, 1e-3))

    // Variable-width hidden layer
    .addLayer(new DenseLayerSpace.Builder()
        .nIn(200)
        .nOut(new IntegerParameterSpace(64, 256))
        .activation(new DiscreteParameterSpace<>(Activation.RELU, Activation.LEAKYRELU))
        .dropOut(new ContinuousParameterSpace(0.0, 0.5))
        .build())

    // Optional batch normalization (present in some candidates)
    .addLayer(new BatchNormalizationSpace.Builder().build(),
        new DiscreteParameterSpace<>(0, 1))  // 0 or 1 BN layers

    // Output
    .addLayer(new OutputLayerSpace.Builder()
        .nOut(5)
        .activation(Activation.SOFTMAX)
        .lossFunction(LossFunctions.LossFunction.MCXENT)
        .build())
    .numEpochs(20)
    .build();
```

---

## Related Pages

- [Arbiter Overview](./overview.md) — optimization configuration and runner
- [Parameter Spaces](./parameter-spaces.md) — primitive and composite parameter space types

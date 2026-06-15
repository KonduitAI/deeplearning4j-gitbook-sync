---
title: "Layers Reference"
description: "Complete reference for all layer types in Deeplearning4j — Dense, Activation, Dropout, Embedding, BatchNormalization, and more"
---

## Overview

Layers are the building blocks of both `MultiLayerNetwork` and `ComputationGraph`. Each layer has a builder class that follows the same pattern:

```java
new LayerType.Builder()
    .nIn(inputSize)
    .nOut(outputSize)
    .activation(Activation.RELU)
    // ...other options...
    .build()
```

Layers inherit common options (weight init, updater, regularization, dropout) from the global `NeuralNetConfiguration.Builder` configuration, and can override them individually.

---

## Common Builder Options (All Layers)

| Method | Description |
|--------|-------------|
| `.nIn(int)` | Number of input units / channels |
| `.nOut(int)` | Number of output units / channels |
| `.activation(Activation)` | Activation function |
| `.weightInit(WeightInit)` | Weight initialization scheme |
| `.updater(IUpdater)` | Per-layer optimizer override |
| `.l1(double)` / `.l2(double)` | Per-layer regularization |
| `.dropOut(double)` | Retain probability for dropout applied to this layer's input |
| `.hasBias(boolean)` | Whether to include a bias parameter (default: true) |
| `.dist(Distribution)` | Weight distribution (used with `WeightInit.DISTRIBUTION`) |

---

## DenseLayer

**Class:** `org.deeplearning4j.nn.conf.layers.DenseLayer`
**Source:** [DenseLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/DenseLayer.java)

A standard fully connected feedforward layer. Computes `output = activation(W * input + b)`.

### Builder Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `nIn` | int | required | Number of input features |
| `nOut` | int | required | Number of output units |
| `activation` | Activation | RELU | Activation function |
| `hasBias` | boolean | true | Include bias vector |
| `hasLayerNorm` | boolean | false | Apply layer normalization after the linear transform |

### Example

```java
new DenseLayer.Builder()
    .nIn(256)
    .nOut(128)
    .activation(Activation.RELU)
    .weightInit(WeightInit.XAVIER)
    .hasBias(true)
    .build()
```

### With Layer Normalization

```java
new DenseLayer.Builder()
    .nIn(256)
    .nOut(128)
    .activation(Activation.RELU)
    .hasLayerNorm(true)   // applies LayerNorm before activation
    .build()
```

---

## OutputLayer

**Class:** `org.deeplearning4j.nn.conf.layers.OutputLayer`
**Source:** [OutputLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/OutputLayer.java)

An output layer that contains a fully connected linear transform followed by an activation and a loss function. This is the final layer for training in `MultiLayerNetwork`. `OutputLayer` has learnable parameters (weights + bias), which means it can project from a different `nIn` to `nOut`.

### Builder Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `lossFunction` | LossFunction | Required. E.g., `NEGATIVELOGLIKELIHOOD`, `MSE`, `MCXENT`, `XENT` |
| `nIn` | int | Input size |
| `nOut` | int | Number of output units (classes for classification, outputs for regression) |
| `activation` | Activation | `SOFTMAX` for multi-class, `SIGMOID` for binary, `IDENTITY` for regression |

### Classification Example

```java
new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
    .nIn(128)
    .nOut(10)
    .activation(Activation.SOFTMAX)
    .build()
```

### Regression Example

```java
new OutputLayer.Builder(LossFunctions.LossFunction.MSE)
    .nIn(64)
    .nOut(1)
    .activation(Activation.IDENTITY)
    .build()
```

### Common Loss Functions

| LossFunction | Use Case |
|--------------|----------|
| `NEGATIVELOGLIKELIHOOD` | Multi-class classification with SOFTMAX |
| `MCXENT` | Multi-class cross-entropy (equivalent to NLL + SOFTMAX) |
| `XENT` | Binary cross-entropy with SIGMOID |
| `MSE` | Mean squared error for regression |
| `MAE` | Mean absolute error for regression |
| `HINGE` | SVM-style hinge loss |
| `COSINE` | Cosine proximity loss |

---

## LossLayer

**Class:** `org.deeplearning4j.nn.conf.layers.LossLayer`
**Source:** [LossLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LossLayer.java)

A parameter-free output layer that applies a loss function to its inputs without any linear transform. Unlike `OutputLayer`, `LossLayer` has no weights — it simply wraps whatever activation comes in with a loss function. Output size equals input size.

Use `LossLayer` when you have already projected to the correct output dimension in the previous layer and only need a loss function.

### Example

```java
// Previous layer outputs 10 units with softmax already applied
new LossLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
    .activation(Activation.SOFTMAX)
    .build()
```

---

## ActivationLayer

**Class:** `org.deeplearning4j.nn.conf.layers.ActivationLayer`
**Source:** [ActivationLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/ActivationLayer.java)

Applies an activation function as a standalone layer with no learned parameters. Useful in `ComputationGraph` when you need to apply an activation after a residual addition, or when building custom architectures where activation needs to be a named vertex.

### Example

```java
new ActivationLayer.Builder()
    .activation(Activation.RELU)
    .build()

// Or with an IActivation instance for parameterized activations
new ActivationLayer.Builder()
    .activation(new ActivationPReLU())
    .build()
```

---

## DropoutLayer

**Class:** `org.deeplearning4j.nn.conf.layers.DropoutLayer`
**Source:** [DropoutLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/DropoutLayer.java)

Applies dropout as a standalone layer. At training time, activations are randomly zeroed with probability `(1 - retainProbability)`. At test time, activations pass through unchanged.

This differs from the `.dropOut()` option on other layers in that it is an explicit layer in the graph (with a named vertex in `ComputationGraph`), rather than dropout applied implicitly to the previous layer's output.

### Builder Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| Constructor `double` | double | Retain probability (e.g., `0.5` means 50% chance of keeping each unit) |

### Example

```java
// As a standalone layer between two dense layers
.layer(new DenseLayer.Builder().nIn(256).nOut(256).activation(Activation.RELU).build())
.layer(new DropoutLayer.Builder(0.5).build())
.layer(new DenseLayer.Builder().nIn(256).nOut(128).activation(Activation.RELU).build())
```

---

## BatchNormalization

**Class:** `org.deeplearning4j.nn.conf.layers.BatchNormalization`
**Source:** [BatchNormalization.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/BatchNormalization.java)

Normalizes layer inputs to zero mean and unit variance per minibatch during training, then applies a learned scale (`gamma`) and shift (`beta`). At inference time, running mean/variance statistics accumulated during training are used.

### Builder Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `nIn` | int | auto | Number of input channels/features |
| `nOut` | int | auto | Must equal `nIn` |
| `decay` | double | 0.9 | Momentum for running statistics update |
| `eps` | double | 1e-5 | Small constant for numerical stability |
| `isMinibatch` | boolean | true | Use minibatch statistics during training |
| `lockGammaBeta` | boolean | false | If true, gamma=1 and beta=0 are fixed (not learned) |
| `cudnnAllowFallback` | boolean | true | Fall back to non-CuDNN if GPU error occurs |

### Example — After a Dense Layer

```java
.layer(new DenseLayer.Builder().nIn(256).nOut(128).activation(Activation.IDENTITY).build())
.layer(new BatchNormalization.Builder().nIn(128).nOut(128).build())
.layer(new ActivationLayer.Builder().activation(Activation.RELU).build())
```

### Example — After a Convolutional Layer

BatchNormalization normalizes across all spatial positions per channel when used after convolutional layers:

```java
.layer(new ConvolutionLayer.Builder(3, 3).nIn(64).nOut(128)
    .activation(Activation.IDENTITY).build())
.layer(new BatchNormalization.Builder().build())
.layer(new ActivationLayer.Builder().activation(Activation.RELU).build())
```

When `setInputType()` is used, `nIn`/`nOut` for `BatchNormalization` can be inferred automatically.

---

## EmbeddingLayer

**Class:** `org.deeplearning4j.nn.conf.layers.EmbeddingLayer`
**Source:** [EmbeddingLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/EmbeddingLayer.java)

Maps integer indices to dense embedding vectors. Mathematically equivalent to a `DenseLayer` with a one-hot input, but far more efficient for large vocabularies because it performs a direct row lookup rather than a full matrix multiply.

**Restrictions:**
- Can only be the first layer of a network.
- Input shape: `[minibatch, 1]` — a single integer index per example.
- Output shape: `[minibatch, embeddingSize]`.

### Builder Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `nIn` | int | required | Vocabulary size (number of distinct tokens) |
| `nOut` | int | required | Embedding dimension |
| `hasBias` | boolean | false | Include per-embedding bias |
| `activation` | Activation | IDENTITY | Activation applied after lookup |
| `weightInit(INDArray)` | INDArray | — | Initialize from a pre-trained embedding matrix `[vocabSize, embeddingSize]` |
| `weightInit(EmbeddingInitializer)` | EmbeddingInitializer | — | Initialize from a Word2Vec model or similar |

### Example — Basic

```java
new EmbeddingLayer.Builder()
    .nIn(10000)    // vocabulary of 10,000 tokens
    .nOut(128)     // 128-dimensional embeddings
    .build()
```

### Example — Pre-trained Embeddings

```java
INDArray pretrainedVectors = /* shape [vocabSize, 300] loaded from Word2Vec */;

new EmbeddingLayer.Builder()
    .nIn(vocabSize)
    .nOut(300)
    .weightInit(pretrainedVectors)
    .build()
```

---

## EmbeddingSequenceLayer

**Class:** `org.deeplearning4j.nn.conf.layers.EmbeddingSequenceLayer`
**Source:** [EmbeddingSequenceLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/EmbeddingSequenceLayer.java)

Sequence-aware version of `EmbeddingLayer`. Accepts a sequence of integer indices per example and outputs a sequence of embedding vectors.

- Input shape: `[minibatch, inputLength]` or `[minibatch, 1, inputLength]`.
- Output shape: `[minibatch, nOut, inputLength]` — a 3D time-series tensor ready for RNN or CNN-1D layers.

**Restrictions:** Can only be the first layer of a network.

### Builder Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `nIn` | int | required | Vocabulary size |
| `nOut` | int | required | Embedding dimension |
| `inputLength` | int | required | Sequence length |
| `inferInputLength` | boolean | false | Infer sequence length from input at runtime |
| `hasBias` | boolean | false | Include bias |
| `weightInit(INDArray)` | INDArray | — | Pre-trained embedding matrix |

### Example

```java
new EmbeddingSequenceLayer.Builder()
    .nIn(5000)          // vocabulary
    .nOut(64)           // embedding dim
    .inputLength(100)   // sequence length
    .build()
```

Use this in conjunction with LSTM or Conv1D layers for text classification:

```java
.layer(new EmbeddingSequenceLayer.Builder().nIn(5000).nOut(64).inputLength(100).build())
.layer(new LSTM.Builder().nIn(64).nOut(128).activation(Activation.TANH).build())
.layer(new RnnOutputLayer.Builder(LossFunctions.LossFunction.MCXENT)
    .nIn(128).nOut(numClasses).activation(Activation.SOFTMAX).build())
```

---

## GlobalPoolingLayer

**Class:** `org.deeplearning4j.nn.conf.layers.GlobalPoolingLayer`
**Source:** [GlobalPoolingLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/GlobalPoolingLayer.java)

Reduces spatial or temporal dimensions to a single value per channel/feature via pooling. Works with 2D (feedforward), 3D (time series/RNN), 4D (CNN), and 5D (CNN3D) inputs.

Default behaviour (collapseDimensions=true):
- 3D time series `[mb, features, T]` -> 2D `[mb, features]`
- 4D CNN `[mb, C, H, W]` -> 2D `[mb, C]`
- 5D CNN3D `[mb, C, D, H, W]` -> 2D `[mb, C]`

Supports masking for variable-length sequences.

### Builder Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `poolingType` | PoolingType | AVG | `MAX`, `AVG`, `SUM`, `PNORM` |
| `collapseDimensions` | boolean | true | Collapse spatial/temporal dims to 1 |
| `pnorm` | int | 2 | P value, only for `PNORM` pooling |
| `poolingDimensions` | int[] | auto | Override which dimensions to pool over |

### Example — Global Average Pooling after CNN

```java
// Replaces Flatten + Dense with a parameter-free global pool
.layer(new ConvolutionLayer.Builder(3, 3).nIn(64).nOut(128).activation(Activation.RELU).build())
.layer(new GlobalPoolingLayer.Builder(PoolingType.AVG).build())
.layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
    .nIn(128).nOut(numClasses).activation(Activation.SOFTMAX).build())
```

### Example — Global Max Pooling for sequence classification

```java
// After EmbeddingSequenceLayer + Conv1D: pool across time
.layer(new GlobalPoolingLayer.Builder(PoolingType.MAX).build())
```

---

## LocalResponseNormalization

**Class:** `org.deeplearning4j.nn.conf.layers.LocalResponseNormalization`
**Source:** [LocalResponseNormalization.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LocalResponseNormalization.java)

Implements the local response normalization described in the AlexNet paper. Normalizes over `n` adjacent feature maps. Largely superseded by Batch Normalization in modern architectures but included for legacy compatibility.

### Builder Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `k` | 2.0 | Additive constant |
| `n` | 5.0 | Number of adjacent kernel maps |
| `alpha` | 1e-4 | Scaling constant |
| `beta` | 0.75 | Exponent |

---

## ElementWiseMultiplicationLayer

**Class:** `org.deeplearning4j.nn.conf.layers.misc.ElementWiseMultiplicationLayer`
**Source:** [ElementWiseMultiplicationLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/misc/ElementWiseMultiplicationLayer.java)

Computes `output = activation(input . w + b)` where `.` is element-wise multiplication and `w` is a learnable weight vector of length `nOut`. Input and output sizes are the same.

Useful for gating mechanisms and attention-like weighting.

---

## RepeatVector

**Class:** `org.deeplearning4j.nn.conf.layers.misc.RepeatVector`
**Source:** [RepeatVector.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/misc/RepeatVector.java)

Repeats a 2D input `[mb, length]` a specified number of times to produce a 3D output `[mb, n, length]`. Commonly used in sequence-to-sequence encoder-decoder architectures to broadcast the encoder's context vector across all decoder time steps.

### Builder Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `repetitionFactor(int)` | int | Number of times to repeat (n) |

### Example

```java
// Encoder produces [mb, 128]; repeat for 10 decoder time steps -> [mb, 10, 128]
new RepeatVector.Builder()
    .repetitionFactor(10)
    .nIn(128)
    .nOut(128)
    .build()
```

---

## MaskLayer

**Class:** `org.deeplearning4j.nn.conf.layers.util.MaskLayer`

Applies the mask array to both forward pass activations and backward pass gradients. Works with 2D, 3D, and 4D inputs. Use when you need to apply masking logic at a specific point in the graph rather than relying on the implicit masking propagated by `DataSet.featuresMaskArray`.

---

## MaskZeroLayer

**Class:** `org.deeplearning4j.nn.conf.layers.util.MaskZeroLayer`

Wraps a recurrent layer and masks time steps where the input activation equals the specified masking value (default: `0.0`). Input shape: `[batch, inputSize, timesteps]`. Useful for variable-length sequence handling without explicit mask arrays.

---

## LocallyConnected1D / LocallyConnected2D

Locally connected layers are like convolutions except that each spatial position has its own independent filter weights (no weight sharing). They are more parameter-heavy than convolutions but more flexible.

### LocallyConnected1D

```java
new LocallyConnected1D.Builder()
    .nIn(32).nOut(64)
    .kernelSize(3).stride(1).padding(1)
    .activation(Activation.RELU)
    .setInputSize(100)  // sequence length of the input
    .build()
```

### LocallyConnected2D

```java
new LocallyConnected2D.Builder()
    .nIn(3).nOut(32)
    .kernelSize(3, 3).stride(1, 1).padding(1, 1)
    .activation(Activation.RELU)
    .setInputSize(28, 28)  // spatial dimensions of input
    .build()
```

---

## Full Configuration Example (MLP Classifier, M2.1)

```java
import org.deeplearning4j.nn.conf.MultiLayerConfiguration;
import org.deeplearning4j.nn.conf.NeuralNetConfiguration;
import org.deeplearning4j.nn.conf.layers.*;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.deeplearning4j.nn.weights.WeightInit;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.learning.config.Adam;
import org.nd4j.linalg.lossfunctions.LossFunctions;

MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(0.001))
    .weightInit(WeightInit.XAVIER)
    .l2(1e-5)
    .list()
    .layer(new DenseLayer.Builder()
        .nIn(784).nOut(512)
        .activation(Activation.RELU)
        .build())
    .layer(new BatchNormalization.Builder().nIn(512).nOut(512).build())
    .layer(new DropoutLayer.Builder(0.5).build())
    .layer(new DenseLayer.Builder()
        .nIn(512).nOut(256)
        .activation(Activation.RELU)
        .build())
    .layer(new BatchNormalization.Builder().nIn(256).nOut(256).build())
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nIn(256).nOut(10)
        .activation(Activation.SOFTMAX)
        .build())
    .build();

MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
System.out.println(model.summary());
```

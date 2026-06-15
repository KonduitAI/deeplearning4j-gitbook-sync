---
title: "Neural Network Fundamentals"
description: "Layers, activation functions, loss functions, weight initialization, and regularization in Deeplearning4j"
---

# Neural Network Fundamentals

This page covers the building blocks used to construct neural networks in the DL4J ecosystem. These components — layers, activations, loss functions, weight initialization, and regularization — are shared between `MultiLayerNetwork`, `ComputationGraph`, and `SameDiff`.

## Layers

A layer is the fundamental unit of a neural network. Each layer takes an `INDArray` as input, applies a transformation (often a linear operation followed by a non-linear activation), and produces an `INDArray` as output. Trainable layers have parameters (weights and biases) that are also `INDArray`s, updated during training.

In DL4J, layers are configured using builder objects in `org.deeplearning4j.nn.conf.layers`:

```java
new DenseLayer.Builder()
    .nIn(784)                        // number of inputs
    .nOut(256)                       // number of outputs
    .activation(Activation.RELU)     // activation function
    .weightInit(WeightInit.XAVIER)   // weight initialization
    .build()
```

### Layer Types Overview

**Feed-Forward Layers:**

| Layer | Class | Use Case |
|-------|-------|----------|
| Dense (fully connected) | `DenseLayer` | General-purpose hidden layer |
| Output | `OutputLayer` | Final layer with loss function |
| Loss | `LossLayer` | Applies loss without parameters |
| Dropout | `DropoutLayer` | Regularization via random zeroing |
| Activation | `ActivationLayer` | Applies activation without parameters |
| Embedding | `EmbeddingLayer` | Lookup table for discrete inputs (NLP) |
| Embedding Sequence | `EmbeddingSequenceLayer` | Embedding for sequence inputs |

**Convolutional Layers:**

| Layer | Class | Use Case |
|-------|-------|----------|
| 1D Convolution | `Convolution1DLayer` | Temporal/sequence convolution |
| 2D Convolution | `ConvolutionLayer` | Image feature extraction |
| 3D Convolution | `Convolution3D` | Video / volumetric data |
| 2D Deconvolution | `Deconvolution2D` | Upsampling / transpose convolution |
| 3D Deconvolution | `Deconvolution3D` | Volumetric upsampling |
| Depthwise Conv2D | `DepthwiseConvolution2D` | Efficient mobile convolutions |
| Separable Conv2D | `SeparableConvolution2D` | Depthwise + pointwise |
| Subsampling (Pooling) | `SubsamplingLayer` | Max/average pooling (2D) |
| 1D Subsampling | `Subsampling1DLayer` | Pooling for 1D data |
| 3D Subsampling | `Subsampling3DLayer` | Pooling for 3D data |
| Global Pooling | `GlobalPoolingLayer` | Pool across entire spatial dimension |
| Upsampling 1D/2D/3D | `Upsampling1D`, `Upsampling2D`, `Upsampling3D` | Nearest-neighbor upsampling |
| Zero Padding | `ZeroPaddingLayer`, `ZeroPadding1DLayer`, `ZeroPadding3DLayer` | Padding input |
| Cropping | `Cropping1D`, `Cropping2D`, `Cropping3D` | Crop spatial dimensions |
| Batch Normalization | `BatchNormalization` | Normalize activations per mini-batch |
| Local Response Norm | `LocalResponseNormalization` | Cross-channel normalization |
| Locally Connected | `LocallyConnected1D`, `LocallyConnected2D` | Unshared-weight convolution |
| Space to Batch | `SpaceToBatchLayer` | Spatial rearrangement |
| Space to Depth | `SpaceToDepthLayer` | Trade spatial for depth |

**Recurrent Layers:**

| Layer | Class | Use Case |
|-------|-------|----------|
| LSTM | `LSTM` | Long short-term memory (default RNN choice) |
| GravesLSTM | `GravesLSTM` | Original Graves LSTM variant |
| Simple RNN | `SimpleRnn` | Basic recurrent layer |
| Bidirectional | `Bidirectional` | Wrapper — runs any RNN in both directions |
| Last Time Step | `LastTimeStep` | Extracts final time step output |
| Time Distributed | `TimeDistributed` | Applies a layer independently to each time step |
| RNN Output | `RnnOutputLayer` | Output layer for sequence-to-sequence |
| RNN Loss | `RnnLossLayer` | Loss layer for sequence outputs |
| Self-Attention | `SelfAttentionLayer` | Multi-head self-attention |
| Learned Self-Attention | `LearnedSelfAttentionLayer` | Attention with learned queries |
| Recurrent Attention | `RecurrentAttentionLayer` | Attention over RNN outputs |

**Generative / Specialized:**

| Layer | Class | Use Case |
|-------|-------|----------|
| Autoencoder | `AutoEncoder` | Unsupervised feature learning |
| Variational Autoencoder | `VariationalAutoencoder` | Generative model with latent space |
| Capsule Layer | `CapsuleLayer` | Capsule networks (dynamic routing) |
| Primary Capsules | `PrimaryCapsules` | Initial capsule layer |
| YOLO2 Output | `Yolo2OutputLayer` | Object detection output |
| CNN Loss | `CnnLossLayer` | Per-pixel loss for segmentation |
| Center Loss Output | `CenterLossOutputLayer` | Face recognition / metric learning |
| OCNN Output | `OCNNOutputLayer` | One-class neural network (anomaly detection) |

**SameDiff-Backed Custom Layers:**

| Layer | Class | Use Case |
|-------|-------|----------|
| SameDiff Layer | `SameDiffLayer` | Custom layer with autodiff |
| SameDiff Output | `SameDiffOutputLayer` | Custom output layer with autodiff |
| SameDiff Lambda | `SameDiffLambdaLayer` | Stateless transform via SameDiff |
| SameDiff Vertex | `SameDiffVertex` | Custom graph vertex with autodiff |

## Activation Functions

Activation functions introduce non-linearity. Without them, stacking layers would be equivalent to a single linear transformation.

The `Activation` enum is at `org.nd4j.linalg.activations.Activation`. Use it in layer builders:

```java
.activation(Activation.RELU)
```

### Available Activations in M2.1

| Activation | Enum Value | Typical Use |
|------------|-----------|-------------|
| ReLU | `RELU` | Default for hidden layers |
| Sigmoid | `SIGMOID` | Binary classification output |
| Softmax | `SOFTMAX` | Multi-class classification output |
| Tanh | `TANH` | Hidden layers (alternative to ReLU) |
| Identity | `IDENTITY` | Regression output (linear) |
| Leaky ReLU | `LEAKYRELU` | Avoids dying ReLU problem |
| ELU | `ELU` | Smooth alternative to ReLU |
| SELU | `SELU` | Self-normalizing networks |
| GELU | `GELU` | Transformer networks |
| Mish | `MISH` | Smooth, self-regularized |
| Swish | `SWISH` | Smooth alternative to ReLU |
| ReLU6 | `RELU6` | Capped ReLU for mobile nets |
| Hard Sigmoid | `HARDSIGMOID` | Fast approximation of sigmoid |
| Hard Tanh | `HARDTANH` | Fast approximation of tanh |
| Softplus | `SOFTPLUS` | Smooth ReLU approximation |
| Softsign | `SOFTSIGN` | Alternative to tanh |
| RReLU | `RRELU` | Randomized leaky ReLU |
| Thresholded ReLU | `THRESHOLDEDRELU` | ReLU with custom threshold |
| Cube | `CUBE` | x^3 activation |
| Rational Tanh | `RATIONALTANH` | Fast tanh approximation |
| Rectified Tanh | `RECTIFIEDTANH` | max(0, tanh(x)) |
| PReLU | Via `PReLULayer` | Learned leaky ReLU slope |

For custom activations, implement the `IActivation` interface at `org.nd4j.linalg.activations.IActivation`.

### Common Pairings

- **Hidden layers (feed-forward, CNN):** `RELU` with `WeightInit.RELU`
- **Hidden layers (RNN):** `TANH` with `WeightInit.XAVIER`
- **Multi-class output:** `SOFTMAX` with `LossMCXENT`
- **Binary output:** `SIGMOID` with `LossBinaryXENT`
- **Regression output:** `IDENTITY` with `LossMSE`

## Loss Functions

Loss functions measure how far the network's predictions are from the true labels. In M2.1, loss functions are instances of `ILossFunction` (at `org.nd4j.linalg.lossfunctions.ILossFunction`).

Use them in output layers:

```java
// Using ILossFunction instance (preferred in M2.1)
new OutputLayer.Builder(new LossMCXENT())
    .activation(Activation.SOFTMAX)
    .nIn(256).nOut(10)
    .build()

// Using the convenience enum (still works)
new OutputLayer.Builder(LossFunctions.LossFunction.MSE)
    .activation(Activation.IDENTITY)
    .nIn(256).nOut(1)
    .build()
```

### Available Loss Functions

| Loss | Class | Use Case |
|------|-------|----------|
| Multi-class cross entropy | `LossMCXENT` | Multi-class classification (with softmax) |
| Binary cross entropy | `LossBinaryXENT` | Binary / multi-label classification |
| Negative log likelihood | `LossNegativeLogLikelihood` | Similar to MCXENT |
| Sparse MCXENT | `LossSparseMCXENT` | MCXENT with integer labels (not one-hot) |
| Mean Squared Error | `LossMSE` | Regression |
| Mean Absolute Error | `LossMAE` | Regression (robust to outliers) |
| Mean Squared Log Error | `LossMSLE` | Regression on log scale |
| Mean Absolute Percentage Error | `LossMAPE` | Percentage-based regression |
| L1 Loss | `LossL1` | Sparse regression |
| L2 Loss | `LossL2` | Standard regression |
| Hinge Loss | `LossHinge` | SVM-style classification |
| Squared Hinge | `LossSquaredHinge` | Smooth hinge loss |
| Poisson | `LossPoisson` | Count data regression |
| KL Divergence | `LossKLD` | Distribution matching |
| Cosine Proximity | `LossCosineProximity` | Similarity learning |
| Wasserstein | `LossWasserstein` | GAN training |
| F-Measure | `LossFMeasure` | Optimize F1 score directly |
| Multi-Label | `LossMultiLabel` | Multi-label classification |
| Mixture Density | `LossMixtureDensity` | Mixture density networks |

## Weight Initialization

Weight initialization determines the starting values for a layer's parameters. Poor initialization can lead to vanishing or exploding gradients.

The `WeightInit` enum is at `org.nd4j.weightinit.WeightInit`:

```java
.weightInit(WeightInit.XAVIER)
```

### Available Initializers

| Initializer | Enum Value | When to Use |
|-------------|-----------|-------------|
| Xavier (Glorot) | `XAVIER` | Default for sigmoid/tanh activations |
| Xavier Uniform | `XAVIER_UNIFORM` | Uniform variant of Xavier |
| ReLU (He) | `RELU` | ReLU and variants |
| ReLU Uniform | `RELU_UNIFORM` | Uniform variant for ReLU |
| LeCun Normal | `LECUN_NORMAL` | SELU activations |
| LeCun Uniform | `LECUN_UNIFORM` | Uniform variant for SELU |
| Variance Scaling (Fan In) | `VAR_SCALING_NORMAL_FAN_IN` | General purpose |
| Variance Scaling (Fan Out) | `VAR_SCALING_NORMAL_FAN_OUT` | General purpose |
| Variance Scaling (Fan Avg) | `VAR_SCALING_NORMAL_FAN_AVG` | General purpose |
| Normal | `NORMAL` | N(0, 1) — rarely used alone |
| Uniform | `UNIFORM` | U(-1, 1) — rarely used alone |
| Zero | `ZERO` | All zeros (use for biases) |
| Ones | `ONES` | All ones |
| Identity | `IDENTITY` | Identity matrix (square layers only) |
| Constant | `CONSTANT` | User-specified constant value |
| Supplied | `SUPPLIED` | User-provided INDArray |

**Rules of thumb:**
- `XAVIER` for tanh/sigmoid networks
- `RELU` for ReLU/Leaky ReLU/ELU networks
- `LECUN_NORMAL` for SELU networks

## Regularization

Regularization prevents overfitting by constraining the model's parameters.

### L1 and L2 Regularization

Applied via the `NeuralNetConfiguration.Builder`:

```java
new NeuralNetConfiguration.Builder()
    .l2(1e-4)         // L2 regularization on all layers
    .l1(1e-5)         // L1 regularization on all layers
    // ...
```

L2 penalizes large weights (encourages small, distributed weights). L1 encourages sparsity (drives some weights to zero). L2 is more common.

`WeightDecay` is an alternative to L2 that decouples regularization from the learning rate:

```java
.l2(0)  // disable L2
.weightDecay(1e-4, true)  // use weight decay instead
```

### Dropout

Randomly zeros a fraction of activations during training:

```java
// In the configuration builder (applies to all layers):
.dropOut(0.5)  // 50% dropout rate

// Or as a specific layer:
new DropoutLayer.Builder(0.5).build()
```

Variants available in `org.deeplearning4j.nn.conf.dropout`:

| Dropout Type | Class | Description |
|-------------|-------|-------------|
| Standard | `Dropout` | Randomly zero with probability p |
| Gaussian | `GaussianDropout` | Multiply by N(1, rate) |
| Gaussian Noise | `GaussianNoise` | Add N(0, stddev) noise |
| Alpha Dropout | `AlphaDropout` | For SELU networks |
| Spatial Dropout | `SpatialDropout` | Drops entire feature maps (for CNNs) |

### Weight Noise

Adds noise to weights during training:

```java
.weightNoise(new WeightNoise(new NormalDistribution(0, 0.01)))
```

`DropConnect` is a variant that randomly zeros weight values (not activations):

```java
.weightNoise(new DropConnect(0.5))
```

### Parameter Constraints

Constrain parameter values after each update:

```java
new DenseLayer.Builder()
    .constrainWeights(new MaxNormConstraint(2.0, 1))    // max L2 norm of 2.0
    .constrainBias(new NonNegativeConstraint())           // biases >= 0
    .build()
```

Available constraints: `MaxNormConstraint`, `MinMaxNormConstraint`, `NonNegativeConstraint`, `UnitNormConstraint`.

### Gradient Normalization

Prevents exploding gradients:

```java
new NeuralNetConfiguration.Builder()
    .gradientNormalization(GradientNormalization.ClipElementWiseAbsoluteValue)
    .gradientNormalizationThreshold(1.0)
    // ...
```

Options: `None`, `RenormalizeL2PerLayer`, `RenormalizeL2PerParamType`, `ClipElementWiseAbsoluteValue`, `ClipL2PerLayer`, `ClipL2PerParamType`.

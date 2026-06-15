---
title: "Convolutional Layers"
description: "CNN layers in Deeplearning4j — Conv1D/2D/3D, pooling, deconvolution, depthwise, separable, and upsampling layers"
---

## Overview

Deeplearning4j provides a complete set of convolutional layers covering 1D (sequences), 2D (images), and 3D (video/volumetric) convolutions. All CNN layers share common builder patterns and integrate with `setInputType()` for automatic shape inference and pre-processor insertion.

---

## ConvolutionMode

`ConvolutionMode` controls how output spatial dimensions are computed relative to input size. Set it with `.convolutionMode(ConvolutionMode.X)` on any convolutional layer.

| Mode | Description |
|------|-------------|
| `Truncated` (default) | Output may be smaller than input if padding doesn't divide evenly; no truncation warning |
| `Same` | Output has the same spatial size as the input (appropriate zero-padding is added automatically) |
| `Full` | Output is larger than input; full convolution with padding |
| `Causal` | Causal (1D only): no future information leaks into past time steps |

```java
import org.deeplearning4j.nn.conf.ConvolutionMode;

new ConvolutionLayer.Builder(3, 3)
    .convolutionMode(ConvolutionMode.Same)
    .nIn(64).nOut(128)
    .build()
```

---

## ConvolutionLayer (Conv2D)

**Class:** `org.deeplearning4j.nn.conf.layers.ConvolutionLayer`
**Source:** [ConvolutionLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/ConvolutionLayer.java)

Standard 2D convolution layer. Input shape: `[minibatch, channels, height, width]`.

### Builder Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| constructor `(int, int)` | int, int | required | Kernel height and width |
| `nIn` | int | auto | Number of input channels |
| `nOut` | int | required | Number of output filters |
| `stride(int, int)` | int, int | 1, 1 | Stride in height and width |
| `padding(int, int)` | int, int | 0, 0 | Zero-padding in height and width |
| `dilation(int, int)` | int, int | 1, 1 | Dilation (atrous convolution) |
| `activation` | Activation | RELU | Activation function |
| `hasBias` | boolean | true | Include bias |
| `convolutionMode` | ConvolutionMode | Truncated | How to handle border effects |
| `cudnnAlgoMode` | AlgoMode | PREFER_FASTEST | CuDNN algorithm selection |

### Example — Basic Conv2D

```java
new ConvolutionLayer.Builder(3, 3)
    .nIn(3)
    .nOut(64)
    .stride(1, 1)
    .padding(1, 1)
    .activation(Activation.RELU)
    .build()
```

### Example — Dilated (Atrous) Convolution

```java
new ConvolutionLayer.Builder(3, 3)
    .nIn(64).nOut(64)
    .dilation(2, 2)          // receptive field = 5x5 without extra parameters
    .convolutionMode(ConvolutionMode.Same)
    .activation(Activation.RELU)
    .build()
```

### Full Network with setInputType

```java
int channels = 3, height = 32, width = 32, numClasses = 10;

MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .weightInit(WeightInit.XAVIER)
    .list()
    .layer(new ConvolutionLayer.Builder(3, 3)
        .nIn(channels).nOut(32)
        .stride(1, 1).padding(1, 1)
        .activation(Activation.RELU).build())
    .layer(new SubsamplingLayer.Builder(PoolingType.MAX)
        .kernelSize(2, 2).stride(2, 2).build())
    .layer(new ConvolutionLayer.Builder(3, 3)
        .nOut(64).stride(1, 1).padding(1, 1)
        .activation(Activation.RELU).build())
    .layer(new SubsamplingLayer.Builder(PoolingType.MAX)
        .kernelSize(2, 2).stride(2, 2).build())
    .layer(new DenseLayer.Builder().nOut(256).activation(Activation.RELU).build())
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nOut(numClasses).activation(Activation.SOFTMAX).build())
    // Automatically infers nIn for every layer and inserts CnnToFeedForwardPreProcessor
    .setInputType(InputType.convolutional(height, width, channels))
    .build();
```

---

## Convolution1DLayer (Conv1D)

**Class:** `org.deeplearning4j.nn.conf.layers.Convolution1D` (alias: `Convolution1DLayer`)
**Source:** [Convolution1D.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Convolution1D.java)

1D convolution for sequence data. Input shape: `[minibatch, channels, sequenceLength]`.

### Builder Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| constructor `(int)` | int | required | Kernel size (length) |
| `nIn` | int | auto | Input channels (embedding dim) |
| `nOut` | int | required | Number of filters |
| `stride(int)` | int | 1 | Stride along sequence |
| `padding(int)` | int | 0 | Zero-padding on each side |
| `dilation(int)` | int | 1 | Dilation along sequence |
| `convolutionMode` | ConvolutionMode | Truncated | Border handling |

### Example — Text Classification

```java
.layer(new EmbeddingSequenceLayer.Builder()
    .nIn(vocabSize).nOut(128).inputLength(maxLen).build())
.layer(new Convolution1D.Builder(3)      // trigram-style kernel
    .nOut(128).activation(Activation.RELU)
    .convolutionMode(ConvolutionMode.Same).build())
.layer(new GlobalPoolingLayer.Builder(PoolingType.MAX).build())
.layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
    .nIn(128).nOut(numClasses).activation(Activation.SOFTMAX).build())
```

---

## Convolution3D

**Class:** `org.deeplearning4j.nn.conf.layers.Convolution3D`
**Source:** [Convolution3D.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Convolution3D.java)

3D convolution for volumetric data (e.g., video, medical imaging). Supports two data formats.

### Builder Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `kernelSize(int...)` | int[3] | required | Kernel in (depth, height, width) |
| `stride(int...)` | int[3] | 1,1,1 | Stride in (depth, height, width) |
| `padding(int...)` | int[3] | 0,0,0 | Padding in (depth, height, width) |
| `dilation(int...)` | int[3] | 1,1,1 | Dilation in (depth, height, width) |
| `dataFormat` | DataFormat | NCDHW | `NCDHW` (channels first) or `NDHWC` (channels last) |
| `hasBias` | boolean | true | Include bias |

### Example

```java
new Convolution3D.Builder()
    .kernelSize(3, 3, 3)
    .stride(1, 1, 1)
    .padding(1, 1, 1)
    .nIn(16).nOut(32)
    .dataFormat(Convolution3D.DataFormat.NCDHW)
    .activation(Activation.RELU)
    .build()
```

---

## SubsamplingLayer (Pooling2D)

**Class:** `org.deeplearning4j.nn.conf.layers.SubsamplingLayer`
**Source:** [SubsamplingLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/SubsamplingLayer.java)

Standard 2D pooling layer (max, average, sum, or p-norm). Input shape: `[minibatch, channels, height, width]`. No learnable parameters.

### Builder Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| constructor `PoolingType` | PoolingType | required | `MAX`, `AVG`, `SUM`, `PNORM`, `NONE` |
| `kernelSize(int, int)` | int, int | required | Pooling window height, width |
| `stride(int, int)` | int, int | same as kernelSize | Stride |
| `padding(int, int)` | int, int | 0, 0 | Padding |
| `convolutionMode` | ConvolutionMode | Truncated | Border handling |
| `pnorm` | int | 2 | P-norm exponent (only for PNORM) |

### Example

```java
// Max pooling 2x2 with stride 2 (halves spatial dimensions)
new SubsamplingLayer.Builder(SubsamplingLayer.PoolingType.MAX)
    .kernelSize(2, 2)
    .stride(2, 2)
    .build()

// Average pooling 3x3 same-padded
new SubsamplingLayer.Builder(SubsamplingLayer.PoolingType.AVG)
    .kernelSize(3, 3)
    .stride(1, 1)
    .convolutionMode(ConvolutionMode.Same)
    .build()
```

---

## Subsampling1DLayer (Pooling1D)

**Class:** `org.deeplearning4j.nn.conf.layers.Subsampling1DLayer`

1D temporal pooling. Input shape: `[minibatch, channels, sequenceLength]`. Supports `MAX`, `AVG`, `SUM`, `PNORM`.

```java
new Subsampling1DLayer.Builder(SubsamplingLayer.PoolingType.MAX)
    .kernelSize(2)
    .stride(2)
    .build()
```

---

## GlobalPoolingLayer

See [GlobalPoolingLayer in the Layers Reference](layers.md#globalpoolinglayer) for full documentation. In the CNN context:

```java
// Replace flatten + dense with global average pooling
.layer(new GlobalPoolingLayer.Builder(PoolingType.AVG).build())
.layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
    .nIn(numFilters).nOut(numClasses).activation(Activation.SOFTMAX).build())
```

---

## Deconvolution2D (Transposed Convolution)

**Class:** `org.deeplearning4j.nn.conf.layers.Deconvolution2D`
**Source:** [Deconvolution2D.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Deconvolution2D.java)

Transposed convolution (also called fractionally strided convolution or deconvolution). Swaps the forward and backward passes of a regular `ConvolutionLayer`, producing outputs that are typically larger than the input. Used in decoders, segmentation networks, and generative models.

Reference: Zeiler et al., "Deconvolutional Networks" (CVPR 2010).

### Builder Parameters

Same as `ConvolutionLayer` — `kernelSize`, `stride`, `padding`, `nIn`, `nOut`, `activation`, `convolutionMode`, `hasBias`.

### Example — Upsampling Decoder Block

```java
// Encoder output: [mb, 256, 4, 4]
// After Deconvolution2D with kernel 4, stride 2: [mb, 128, 8, 8]
new Deconvolution2D.Builder(4, 4)
    .nIn(256).nOut(128)
    .stride(2, 2)
    .activation(Activation.RELU)
    .build()
```

---

## DepthwiseConvolution2D

**Class:** `org.deeplearning4j.nn.conf.layers.DepthwiseConvolution2D`
**Source:** [DepthwiseConvolution2D.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/DepthwiseConvolution2D.java)

Applies a separate convolutional filter to each input channel independently (no cross-channel mixing). Used as the first step in MobileNet-style depthwise separable convolutions.

### Builder Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `nIn` | int | Number of input channels |
| `depthMultiplier` | int | Number of output channels per input channel (nOut = nIn * depthMultiplier) |
| `kernelSize(int, int)` | int, int | Kernel size |
| `stride(int, int)` | int, int | Stride |
| `convolutionMode` | ConvolutionMode | Border handling |

### Example

```java
// 64 input channels, depthMultiplier=1 -> 64 output channels
new DepthwiseConvolution2D.Builder()
    .nIn(64)
    .depthMultiplier(1)
    .kernelSize(3, 3)
    .stride(1, 1)
    .convolutionMode(ConvolutionMode.Same)
    .activation(Activation.RELU)
    .build()
```

---

## SeparableConvolution2D

**Class:** `org.deeplearning4j.nn.conf.layers.SeparableConvolution2D`
**Source:** [SeparableConvolution2D.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/SeparableConvolution2D.java)

Combines a depthwise convolution with a 1x1 pointwise convolution for cross-channel mixing. Approximates a standard convolution with far fewer parameters — the MobileNet building block.

### Builder Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `nIn` | int | Input channels |
| `nOut` | int | Output channels (after pointwise conv) |
| `kernelSize(int, int)` | int, int | Depthwise kernel size |
| `depthMultiplier` | int | Intermediate channels = nIn * depthMultiplier |
| `stride(int, int)` | int, int | Stride for depthwise conv |
| `convolutionMode` | ConvolutionMode | Border handling |

### Example

```java
// MobileNet-style block
new SeparableConvolution2D.Builder(3, 3)
    .nIn(64).nOut(128)
    .depthMultiplier(1)
    .convolutionMode(ConvolutionMode.Same)
    .activation(Activation.RELU)
    .build()
```

---

## Upsampling2D

**Class:** `org.deeplearning4j.nn.conf.layers.Upsampling2D`
**Source:** [Upsampling2D.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Upsampling2D.java)

Nearest-neighbor upsampling — repeats each value `size` times in both the height and width dimensions. No learnable parameters. Used in U-Net decoders and image segmentation.

Input `[mb, C, H, W]` with `size=2` -> output `[mb, C, 2H, 2W]`.

### Builder Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `size(int)` | int | Upsampling factor (applied to both H and W) |
| `size(int[])` | int[2] | Separate height and width factors |

### Example

```java
new Upsampling2D.Builder()
    .size(2)    // double spatial resolution
    .build()

// Non-uniform upsampling
new Upsampling2D.Builder()
    .size(new int[]{2, 3})   // 2x height, 3x width
    .build()
```

---

## Upsampling1D

**Class:** `org.deeplearning4j.nn.conf.layers.Upsampling1D`

Repeats each value along the sequence dimension. Input `[mb, C, L]` -> output `[mb, C, size*L]`.

```java
new Upsampling1D.Builder().size(2).build()
```

---

## Upsampling3D

**Class:** `org.deeplearning4j.nn.conf.layers.Upsampling3D`

Repeats values in depth, height, and width. Input `[mb, C, D, H, W]` -> output `[mb, C, size*D, size*H, size*W]`.

```java
new Upsampling3D.Builder().size(2).build()          // uniform
new Upsampling3D.Builder().size(new int[]{2,2,2}).build()
```

---

## ZeroPaddingLayer (2D)

**Class:** `org.deeplearning4j.nn.conf.layers.ZeroPaddingLayer`
**Source:** [ZeroPaddingLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/ZeroPaddingLayer.java)

Adds zero-padding explicitly around 2D spatial inputs. Useful when you need precise control over padding at a specific point in a `ComputationGraph`, separate from a convolution layer's built-in padding.

```java
// Pad 2 rows top, 2 rows bottom, 2 cols left, 2 cols right
new ZeroPaddingLayer.Builder(2, 2, 2, 2).build()

// Symmetric padding
new ZeroPaddingLayer.Builder(1, 1).build()   // 1 top/bottom, 1 left/right
```

---

## ZeroPadding1DLayer

Adds zero-padding to the start and end of 1D sequences.

```java
new ZeroPadding1DLayer.Builder(2, 2).build()   // 2 steps left, 2 steps right
```

---

## ZeroPadding3DLayer

Adds zero-padding to volumetric data. Accepts a 6-element array: `[padLeftD, padRightD, padLeftH, padRightH, padLeftW, padRightW]`.

```java
new ZeroPadding3DLayer.Builder(1, 1, 1, 1, 1, 1).build()
```

---

## Cropping Layers

### Cropping1D

Removes `n` time steps from the start and/or end of sequences.

```java
new Cropping1D.Builder(2, 2).build()   // remove 2 steps from each end
```

### Cropping2D

Removes rows/columns from 2D spatial inputs.

```java
new Cropping2D.Builder(1, 1, 1, 1).build()   // top, bottom, left, right
```

### Cropping3D

```java
new Cropping3D.Builder(1, 1, 1).build()       // depth, height, width (symmetric)
new Cropping3D.Builder(1, 1, 2, 2, 0, 0).build()  // per-side for D, H, W
```

---

## SpaceToDepthLayer

**Class:** `org.deeplearning4j.nn.conf.layers.SpaceToDepthLayer`

Rearranges spatial blocks into the channel (depth) dimension. Downsamples spatial size while increasing channel count without any learned parameters. Used in certain detection architectures (YOLO passthrough layer).

Input `[mb, C, H, W]` with block size `b` -> output `[mb, C * b * b, H/b, W/b]`.

```java
new SpaceToDepthLayer.Builder()
    .blocks(2)    // 2x2 spatial blocks -> 4x channel expansion
    .dataFormat(SpaceToDepthLayer.DataFormat.NCHW)
    .build()
```

---

## Yolo2OutputLayer

For object detection with the YOLOv2 architecture:

```java
// boundingBoxPriors: shape [numBoxes, 2] with (w, h) as fraction of grid cell
INDArray priors = Nd4j.create(new double[][] {{1.3221, 1.73145}, {3.19275, 4.00944}});

new Yolo2OutputLayer.Builder()
    .boundingBoxPriors(priors)
    .build()
```

See the [DL4J object detection examples](https://github.com/eclipse/deeplearning4j-examples) for a full YOLO pipeline.

---

## Complete CNN Architecture Example

A simple VGG-style classifier for CIFAR-10:

```java
int h = 32, w = 32, c = 3, numClasses = 10;

MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .weightInit(WeightInit.XAVIER)
    .l2(1e-4)
    .list()
    // Block 1
    .layer(new ConvolutionLayer.Builder(3, 3).nIn(c).nOut(64)
        .padding(1,1).activation(Activation.RELU).build())
    .layer(new ConvolutionLayer.Builder(3, 3).nOut(64)
        .padding(1,1).activation(Activation.RELU).build())
    .layer(new SubsamplingLayer.Builder(PoolingType.MAX)
        .kernelSize(2,2).stride(2,2).build())
    // Block 2
    .layer(new ConvolutionLayer.Builder(3, 3).nOut(128)
        .padding(1,1).activation(Activation.RELU).build())
    .layer(new ConvolutionLayer.Builder(3, 3).nOut(128)
        .padding(1,1).activation(Activation.RELU).build())
    .layer(new SubsamplingLayer.Builder(PoolingType.MAX)
        .kernelSize(2,2).stride(2,2).build())
    // Classifier head
    .layer(new DenseLayer.Builder().nOut(512).activation(Activation.RELU).build())
    .layer(new DropoutLayer.Builder(0.5).build())
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nOut(numClasses).activation(Activation.SOFTMAX).build())
    .setInputType(InputType.convolutional(h, w, c))
    .build();
```

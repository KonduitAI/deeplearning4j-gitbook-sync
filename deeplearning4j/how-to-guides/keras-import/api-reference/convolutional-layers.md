---
title: Keras Convolutional Layer Import
description: DL4J equivalents and API reference for Keras convolutional layers — Conv1D, Conv2D, Conv3D, SeparableConv2D, transposed convolutions, cropping, upsampling, and zero-padding.
---

## Keras Convolutional Layer Import

Convolutional layer support is implemented in the [layers/convolutional](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional) package.

### Support Summary

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| Conv1D | Convolution1DLayer | Yes |
| Conv2D | ConvolutionLayer | Yes |
| Conv3D | ConvolutionLayer (3D) | Yes |
| AtrousConvolution1D | Convolution1DLayer (with dilation) | Yes |
| AtrousConvolution2D | ConvolutionLayer (with dilation) | Yes |
| SeparableConv1D | — | No |
| SeparableConv2D | SeparableConvolution2DLayer | Yes |
| DepthwiseConv2D | DepthwiseConvolution2DLayer | Yes |
| Conv2DTranspose | Deconvolution2DLayer | Yes |
| Conv3DTranspose | — | No |
| Cropping1D | Cropping1DLayer | Yes |
| Cropping2D | Cropping2DLayer | Yes |
| Cropping3D | Cropping3DLayer | Yes |
| UpSampling1D | Upsampling1DLayer | Yes |
| UpSampling2D | Upsampling2DLayer | Yes |
| UpSampling3D | Upsampling3DLayer | Yes |
| ZeroPadding1D | ZeroPadding1DLayer | Yes |
| ZeroPadding2D | ZeroPaddingLayer | Yes |
| ZeroPadding3D | ZeroPadding3DLayer | Yes |

---

## KerasConvolution2D

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasConvolution2D.java)

Imports a Keras Conv2D layer as a DL4J `ConvolutionLayer`.

#### Constructor

```java
public KerasConvolution2D(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

Pass-through constructor. `kerasVersion` is the major Keras version (1 or 2).

#### getConvolution2DLayer

```java
public ConvolutionLayer getConvolution2DLayer()
```

Returns the DL4J `ConvolutionLayer`.

#### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

---

## KerasConvolution1D

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasConvolution1D.java)

Imports a Keras Conv1D layer as a DL4J `Convolution1DLayer`.

#### Constructor

```java
public KerasConvolution1D(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

#### getConvolution1DLayer

```java
public Convolution1DLayer getConvolution1DLayer()
```

#### getInputPreprocessor

```java
public InputPreProcessor getInputPreprocessor(InputType... inputType)
        throws InvalidKerasConfigurationException
```

#### setWeights

```java
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

---

## KerasConvolution3D

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasConvolution3D.java)

Imports a Keras Conv3D layer as a DL4J 3D `ConvolutionLayer`.

---

## KerasAtrousConvolution1D / KerasAtrousConvolution2D

[source 1D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasAtrousConvolution1D.java) |
[source 2D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasAtrousConvolution2D.java)

Imports Keras 1.x AtrousConvolution (dilated convolution) layers. In Keras 2, dilation is specified via the `dilation_rate` argument of standard Conv layers; the importer handles both representations.

```java
public KerasAtrousConvolution2D(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

---

## KerasSeparableConvolution2D

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasSeparableConvolution2D.java)

Imports a Keras SeparableConv2D layer as a DL4J `SeparableConvolution2DLayer`. Depthwise and pointwise weight tensors are loaded separately from the HDF5 file.

```java
public KerasSeparableConvolution2D(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

---

## KerasDeconvolution2D (Conv2DTranspose)

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasDeconvolution2D.java)

Imports a Keras `Conv2DTranspose` layer as a DL4J `Deconvolution2DLayer`.

```java
public KerasDeconvolution2D(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

---

## Cropping Layers

### KerasCropping1D

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasCropping1D.java)

```java
public KerasCropping1D(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

### KerasCropping2D

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasCropping2D.java)

```java
public KerasCropping2D(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException

public Cropping2D getCropping2DLayer()
```

### KerasCropping3D

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasCropping3D.java)

---

## Upsampling Layers

### KerasUpsampling1D

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasUpsampling1D.java)

```java
public KerasUpsampling1D(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException

public Upsampling1D getUpsampling1DLayer()
```

### KerasUpsampling2D

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasUpsampling2D.java)

```java
public Upsampling2D getUpsampling2DLayer()
```

### KerasUpsampling3D

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasUpsampling3D.java)

```java
public Upsampling3D getUpsampling3DLayer()
```

---

## Zero Padding Layers

### KerasZeroPadding1D

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasZeroPadding1D.java)

```java
public KerasZeroPadding1D(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

### KerasZeroPadding2D

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasZeroPadding2D.java)

```java
public ZeroPaddingLayer getZeroPadding2DLayer()
```

### KerasZeroPadding3D

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasZeroPadding3D.java)

```java
public ZeroPadding3DLayer getZeroPadding3DLayer()
```

---

## Notes

- **Data format**: Keras defaults to channels-last (`NHWC`). DL4J uses channels-first (`NCHW`) internally. The importer transposes weight tensors during loading so you do not need to manually convert weights.
- **SeparableConv1D**: not supported. Use `Conv1D` with `groups` if available in your Keras version, or restructure the model.
- **Conv3DTranspose**: no DL4J equivalent at this time.
- **Dilation**: both the legacy `AtrousConvolution` names (Keras 1) and the `dilation_rate` argument (Keras 2) on standard Conv layers are handled transparently.

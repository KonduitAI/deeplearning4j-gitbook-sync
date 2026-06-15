---
title: Keras Locally Connected Layer Import
description: DL4J equivalents and API reference for Keras locally connected layers — LocallyConnected1D and LocallyConnected2D.
---

## Keras Locally Connected Layer Import

Locally connected layers are like convolutional layers, but do not share weights across spatial locations. Each spatial position has its own distinct set of filters. Support is implemented in the [layers/local](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/local) package.

### Support Summary

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| LocallyConnected1D | LocallyConnected1D | Yes |
| LocallyConnected2D | LocallyConnected2D | Yes |

---

## KerasLocallyConnected1D

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/local/KerasLocallyConnected1D.java)

Imports a Keras `LocallyConnected1D` layer as a DL4J `LocallyConnected1D` layer.

### Constructor

```java
public KerasLocallyConnected1D(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

Pass-through constructor. `kerasVersion` is the major Keras version (1 or 2).

### getLocallyConnected1DLayer

```java
public LocallyConnected1D getLocallyConnected1DLayer()
```

Returns the DL4J `LocallyConnected1D` layer. Configuration includes:

- `filters` → number of output filters (nOut)
- `kernel_size` → filter size
- `strides` → stride
- `padding` → `"valid"` only (same padding is not supported for locally connected)
- `activation` → per-position activation function

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Returns the output `InputType` based on the filter count, kernel size, and stride.

**Parameters:**
- `inputType` — array of input `InputType` objects

### setWeights

```java
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

Loads the locally connected weight tensor and bias from the HDF5 file. The weight tensor for `LocallyConnected1D` has shape `[output_length, kernel_size * input_channels, filters]`; the importer handles the reshape from Keras's storage format.

**Parameters:**
- `weights` — map from parameter name to `INDArray`

---

## KerasLocallyConnected2D

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/local/KerasLocallyConnected2D.java)

Imports a Keras `LocallyConnected2D` layer as a DL4J `LocallyConnected2D` layer.

### Constructor

```java
public KerasLocallyConnected2D(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

### getLocallyConnected2DLayer

```java
public LocallyConnected2D getLocallyConnected2DLayer()
```

Returns the DL4J `LocallyConnected2D` layer. Configuration includes:

- `filters` → number of output filters
- `kernel_size` → 2D filter size (height, width)
- `strides` → 2D stride
- `padding` → `"valid"` only
- `activation` → activation function

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

### setWeights

```java
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

Loads the 2D locally connected weight tensor and bias. The weight tensor has shape `[output_rows, output_cols, kernel_rows * kernel_cols * input_channels, filters]`.

---

## Notes

- Locally connected layers have significantly more parameters than equivalent convolutional layers because weights are not shared spatially.
- Only `padding="valid"` is supported. `padding="same"` will throw `UnsupportedKerasConfigurationException`.
- These layers are relatively uncommon in modern architectures; they were more prevalent in early face recognition models (e.g., DeepFace). For most use cases, standard convolutional layers with sufficient depth are preferred.

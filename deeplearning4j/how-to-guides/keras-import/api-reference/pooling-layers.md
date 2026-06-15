---
title: Keras Pooling Layer Import
description: DL4J equivalents and API reference for Keras pooling layers — MaxPooling, AveragePooling, and GlobalPooling in 1D, 2D, and 3D variants.
---

## Keras Pooling Layer Import

Pooling layer support is implemented in the [layers/pooling](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling) package. All standard Keras pooling layers are supported.

### Support Summary

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| MaxPooling1D | Subsampling1DLayer (MAX) | Yes |
| MaxPooling2D | SubsamplingLayer (MAX) | Yes |
| MaxPooling3D | Subsampling3DLayer (MAX) | Yes |
| AveragePooling1D | Subsampling1DLayer (AVG) | Yes |
| AveragePooling2D | SubsamplingLayer (AVG) | Yes |
| AveragePooling3D | Subsampling3DLayer (AVG) | Yes |
| GlobalMaxPooling1D | GlobalPoolingLayer (MAX) | Yes |
| GlobalMaxPooling2D | GlobalPoolingLayer (MAX) | Yes |
| GlobalMaxPooling3D | GlobalPoolingLayer (MAX) | Yes |
| GlobalAveragePooling1D | GlobalPoolingLayer (AVG) | Yes |
| GlobalAveragePooling2D | GlobalPoolingLayer (AVG) | Yes |
| GlobalAveragePooling3D | GlobalPoolingLayer (AVG) | Yes |

---

## KerasPooling1D

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPooling1D.java)

Imports a Keras 1D pooling layer (`MaxPooling1D` or `AveragePooling1D`) as a DL4J `Subsampling1DLayer`.

### Constructor

```java
public KerasPooling1D(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

**Parameters:**
- `layerConfig` — dictionary containing the Keras layer configuration

### getSubsampling1DLayer

```java
public Subsampling1DLayer getSubsampling1DLayer()
```

Returns the DL4J `Subsampling1DLayer`. The pooling type (MAX or AVG) is inferred from the Keras class name.

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

---

## KerasPooling2D

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPooling2D.java)

Imports a Keras 2D pooling layer (`MaxPooling2D` or `AveragePooling2D`) as a DL4J `SubsamplingLayer`.

### Constructor

```java
public KerasPooling2D(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

### getSubsampling2DLayer

```java
public SubsamplingLayer getSubsampling2DLayer()
```

Returns the DL4J `SubsamplingLayer`.

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

---

## KerasPooling3D

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPooling3D.java)

Imports a Keras 3D pooling layer (`MaxPooling3D` or `AveragePooling3D`) as a DL4J `Subsampling3DLayer`.

### Constructor

```java
public KerasPooling3D(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

### getSubsampling3DLayer

```java
public Subsampling3DLayer getSubsampling3DLayer()
```

---

## KerasGlobalPooling

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasGlobalPooling.java)

Imports any Keras global pooling layer (GlobalMaxPooling1D/2D/3D, GlobalAveragePooling1D/2D/3D) as a DL4J `GlobalPoolingLayer`.

### Constructor

```java
public KerasGlobalPooling(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

### getGlobalPoolingLayer

```java
public GlobalPoolingLayer getGlobalPoolingLayer()
```

Returns the DL4J `GlobalPoolingLayer`. Pooling type and dimensionality are inferred from the Keras class name.

### getInputPreprocessor

```java
public InputPreProcessor getInputPreprocessor(InputType... inputType)
        throws InvalidKerasConfigurationException
```

Returns the appropriate `InputPreProcessor` when the global pooling layer follows a layer with a different output format (e.g., following a convolutional layer).

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

---

## KerasPoolingUtils

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPoolingUtils.java)

Utility class for pooling layer import.

### mapPoolingType

```java
public static PoolingType mapPoolingType(String className, KerasLayerConfiguration conf)
        throws UnsupportedKerasConfigurationException
```

Maps a Keras pooling class name (e.g., `"MaxPooling2D"`) to the corresponding DL4J `PoolingType` enum value (`PoolingType.MAX` or `PoolingType.AVG`).

**Parameters:**
- `className` — the Keras layer class name string
- `conf` — the `KerasLayerConfiguration` instance

**Returns:** DL4J `PoolingType`

---

## Notes

- All pooling layers use the `pool_size`, `strides`, and `padding` parameters from the Keras configuration.
- The `padding` value `"same"` and `"valid"` are both supported and translated to the equivalent DL4J padding modes.
- Global pooling layers collapse the spatial dimensions entirely; DL4J's `GlobalPoolingLayer` handles all three dimensionalities via the same class, parameterized by `PoolingDimensions`.

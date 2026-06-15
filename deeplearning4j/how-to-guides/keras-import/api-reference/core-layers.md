---
title: Keras Core Layer Import
description: DL4J equivalents and API reference for Keras core layers — Dense, Flatten, Dropout, Reshape, Merge, Permute, and more.
---

## Keras Core Layer Import

Core layers are implemented in the [layers/core](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core) package of `deeplearning4j-modelimport`.

### Support Summary

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| Dense | DenseLayer | Yes |
| Activation | ActivationLayer | Yes |
| Dropout | DropoutLayer | Yes |
| Flatten | CnnToFeedForwardPreProcessor / RnnToFeedForwardPreProcessor | Yes |
| Reshape | ReshapePreProcessor | Yes |
| Merge | MergeVertex | Yes |
| Permute | PermutePreProcessor | Yes |
| RepeatVector | RepeatVector | Yes |
| Lambda | SameDiffLambda | Yes |
| ActivityRegularization | — | No |
| Masking | MaskZeroLayer | Yes |
| SpatialDropout1D | DropoutLayer (spatial 1D) | Yes |
| SpatialDropout2D | DropoutLayer (spatial 2D) | Yes |
| SpatialDropout3D | DropoutLayer (spatial 3D) | Yes |

---

## KerasPermute

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasPermute.java)

Imports a Permute layer from Keras as a DL4J input preprocessor.

#### Constructor

```java
public KerasPermute(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

**Parameters:**
- `layerConfig` — dictionary containing Keras layer configuration

#### isInputPreProcessor

```java
public boolean isInputPreProcessor()
```

Returns `true`; Permute is implemented as an input preprocessor in DL4J.

#### getInputPreprocessor

```java
public InputPreProcessor getInputPreprocessor(InputType... inputType)
        throws InvalidKerasConfigurationException
```

Returns the DL4J `InputPreProcessor` corresponding to the Permute configuration.

#### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Returns the output `InputType` after permutation.

---

## KerasFlatten

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasFlatten.java)

Imports a Keras Flatten layer as a DL4J `CnnToFeedForwardInputPreProcessor` or `RnnToFeedForwardPreProcessor`, depending on the preceding layer type.

#### Constructor

```java
public KerasFlatten(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

#### isInputPreProcessor

```java
public boolean isInputPreProcessor()
```

Returns `true`; Flatten is implemented as an input preprocessor.

#### getInputPreprocessor

```java
public InputPreProcessor getInputPreprocessor(InputType... inputType)
        throws InvalidKerasConfigurationException
```

Selects the correct DL4J preprocessor based on `inputType`.

#### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

---

## KerasReshape

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasReshape.java)

Imports a Keras Reshape layer as a DL4J input preprocessor.

#### Constructor

```java
public KerasReshape(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

#### isInputPreProcessor / getInputPreprocessor / getOutputType

Same pattern as KerasFlatten above.

---

## KerasMerge

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasMerge.java)

Imports a Keras Merge layer as a DL4J graph vertex. Supports `add`, `multiply`, `subtract`, `average`, `maximum`, and `concatenate` modes. The `dot` mode is not supported.

#### Constructor

```java
public KerasMerge(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

#### getOutputType

```java
public InputType getOutputType(InputType... inputType)
```

---

## KerasDense

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasDense.java)

Imports a Keras Dense layer as a DL4J `DenseLayer`.

#### Constructor

```java
public KerasDense(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

#### getDenseLayer

```java
public DenseLayer getDenseLayer()
```

Returns the DL4J `DenseLayer`.

#### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

#### getNumParams

```java
public int getNumParams()
```

Returns the number of trainable parameters (weights + bias).

#### setWeights

```java
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

---

## KerasDropout

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasDropout.java)

Imports a Keras Dropout layer as a DL4J `DropoutLayer`.

#### Constructor

```java
public KerasDropout(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

#### getDropoutLayer

```java
public DropoutLayer getDropoutLayer()
```

#### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

---

## KerasMasking

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasMasking.java)

Imports a Keras Masking layer as a DL4J `MaskZeroLayer`.

---

## KerasSpatialDropout

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasSpatialDropout.java)

Imports Keras SpatialDropout1D, SpatialDropout2D, and SpatialDropout3D layers as DL4J spatial dropout variants.

---

## KerasLambda

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasLambda.java)

Imports a Keras Lambda layer. The Lambda layer is mapped to a SameDiff-based lambda function in DL4J. The function body is serialized separately.

---

## KerasRepeatVector

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasRepeatVector.java)

Imports a Keras RepeatVector layer, which repeats the input a fixed number of times along a new axis.

---

## Notes

- `ActivityRegularization` has no direct DL4J equivalent and will throw `UnsupportedKerasConfigurationException` at import time. Apply regularization directly in the layer configuration instead.
- The Permute, Flatten, and Reshape layers are all implemented as `InputPreProcessor` instances in DL4J rather than as trainable layers.

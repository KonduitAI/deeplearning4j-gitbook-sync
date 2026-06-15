---
title: Keras Normalization Layer Import
description: DL4J equivalents and API reference for Keras normalization layers — BatchNormalization.
---

## Keras Normalization Layer Import

Normalization layer support is implemented in the [layers/normalization](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/normalization) package.

### Support Summary

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| BatchNormalization | BatchNormalization | Yes |

---

## KerasBatchNormalization

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/normalization/KerasBatchNormalization.java)

Imports a Keras `BatchNormalization` layer as a DL4J `BatchNormalization` layer. Both training-mode (with learned gamma/beta) and inference-mode (with running mean/variance) parameters are loaded from the HDF5 file.

### Constructor

```java
public KerasBatchNormalization(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

Pass-through constructor. `kerasVersion` is the major Keras version (1 or 2).

### getBatchNormalizationLayer

```java
public BatchNormalization getBatchNormalizationLayer()
```

Returns the configured DL4J `BatchNormalization` layer. Configuration includes:

- `epsilon` — small constant added to the variance for numerical stability (default `1e-3` in Keras)
- `momentum` — momentum for updating running statistics (called `decay` in Keras 1, `momentum` in Keras 2)
- `center` — whether a bias (beta) term is used
- `scale` — whether a scale (gamma) term is used

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Returns the output `InputType`. For `BatchNormalization`, the output shape is the same as the input shape.

**Parameters:**
- `inputType` — array of input `InputType` objects

**Returns:** output `InputType`

### getNumParams

```java
public int getNumParams()
```

Returns the number of trainable parameters (4: gamma, beta, running mean, running variance).

### setWeights

```java
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

Loads the four weight tensors from the weight map:

| DL4J parameter name | Keras HDF5 tensor |
|---|---|
| `gamma` | scale parameter |
| `beta` | center parameter |
| `mean` | running mean |
| `var` | running variance |

**Parameters:**
- `weights` — map from parameter name to `INDArray`

---

## Keras BatchNormalization Parameters

The following Keras parameters are mapped during import:

| Keras parameter | DL4J configuration method |
|---|---|
| `axis` | determines which dimension is normalized |
| `momentum` | `decay()` in DL4J builder |
| `epsilon` | `eps()` in DL4J builder |
| `center` | whether beta is included |
| `scale` | whether gamma is included |

**Note on `axis`**: Keras BatchNormalization normalizes along the feature axis. In Keras with channels-last (`NHWC`), this is axis 3. DL4J uses channels-first (`NCHW`), where the feature axis is 1. The importer translates the axis value automatically.

---

## Example

```python
# Python: define and save a model with BatchNormalization
from keras.models import Sequential
from keras.layers import Dense, BatchNormalization

model = Sequential()
model.add(Dense(128, input_dim=64))
model.add(BatchNormalization())
model.add(Dense(10, activation='softmax'))
model.save('bn_model.h5')
```

```java
// Java: import and run inference
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights("bn_model.h5");
INDArray input = Nd4j.randn(32, 64);
INDArray output = model.output(input);
```

---

## Notes

- At inference time, DL4J uses the running mean and variance (not batch statistics), which matches Keras inference behavior.
- `LayerNormalization` (Keras) has no direct equivalent in DL4J core and is not currently supported by the importer.

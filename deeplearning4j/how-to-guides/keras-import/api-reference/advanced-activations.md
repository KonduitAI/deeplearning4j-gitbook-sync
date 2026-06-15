---
title: Keras Advanced Activation Layer Import
description: DL4J equivalents and API reference for Keras advanced activation layers — LeakyReLU, PReLU, ELU, and ThresholdedReLU.
---

## Keras Advanced Activation Layer Import

Advanced activation layers in Keras are parameterized activation functions implemented as full layers (with optional learnable parameters). Support is in the [layers/advanced/activations](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/advanced/activations) package.

### Support Summary

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| LeakyReLU | ActivationLayer (LeakyReLU) | Yes |
| PReLU | PReLULayer | Yes |
| ELU | ActivationLayer (ELU) | Yes |
| ThresholdedReLU | ActivationLayer (ThresholdedReLU) | Yes |

---

## KerasLeakyReLU

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/advanced/activations/KerasLeakyReLU.java)

Imports a Keras `LeakyReLU` layer as a DL4J `ActivationLayer` with a `LeakyReLU` activation function. The slope of the negative part (`alpha`) is read from the Keras configuration.

### Constructor

```java
public KerasLeakyReLU(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

**Parameters:**
- `layerConfig` — dictionary containing the Keras layer configuration

### getActivationLayer

```java
public ActivationLayer getActivationLayer()
```

Returns the DL4J `ActivationLayer` configured with a `LeakyReLU` activation.

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Returns the same `InputType` as the input (activations are shape-preserving).

---

## KerasPReLU

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/advanced/activations/KerasPReLU.java)

Imports a Keras `PReLU` layer as a DL4J `PReLULayer`. PReLU has per-channel (or per-element) learnable slope parameters; these are loaded from the HDF5 weights.

### Constructor

```java
public KerasPReLU(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

### getPReLULayer

```java
public PReLULayer getPReLULayer()
```

Returns the DL4J `PReLULayer` with learned slope weights.

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

### setWeights

```java
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

Loads the PReLU slope parameters from the weight map.

**Parameters:**
- `weights` — map from parameter name to `INDArray` containing the slope tensor

---

## KerasThresholdedReLU

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/advanced/activations/KerasThresholdedReLU.java)

Imports a Keras `ThresholdedReLU` layer as a DL4J `ActivationLayer` with a thresholded ReLU activation. The threshold value is read from the Keras configuration.

`ThresholdedReLU(x) = x if x > theta, else 0`

### Constructor

```java
public KerasThresholdedReLU(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

### getActivationLayer

```java
public ActivationLayer getActivationLayer()
```

Returns the DL4J `ActivationLayer` configured with the threshold value.

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

---

## ELU

ELU (Exponential Linear Unit) is supported as an activation function in standard layers (e.g., `Dense(activation='elu')`) and also as an explicit `ELU` layer. The importer handles both forms. When used as a standalone layer, it is mapped to a DL4J `ActivationLayer` with `ActivationELU`.

---

## Example

```python
# Python: model with LeakyReLU and PReLU layers
from keras.models import Sequential
from keras.layers import Dense, LeakyReLU, PReLU

model = Sequential()
model.add(Dense(64, input_dim=100))
model.add(LeakyReLU(alpha=0.1))
model.add(Dense(64))
model.add(PReLU())
model.add(Dense(10, activation='softmax'))
model.save('advanced_act_model.h5')
```

```java
// Java: import
MultiLayerNetwork model = KerasModelImport
    .importKerasSequentialModelAndWeights("advanced_act_model.h5");
```

---

## Notes

- Unlike Keras, where advanced activations are standalone layers, DL4J maps them to `ActivationLayer` (for fixed-parameter activations) or `PReLULayer` (for learned parameters). The layer count in `model.summary()` may therefore differ between DL4J and Keras.
- `Softmax` used as a standalone `Activation` layer rather than a parameter is also supported and maps to `ActivationLayer(ActivationSoftmax)`.

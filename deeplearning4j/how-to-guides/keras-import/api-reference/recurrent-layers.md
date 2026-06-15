---
title: Keras Recurrent Layer Import
description: DL4J equivalents and API reference for Keras recurrent layers — SimpleRNN, LSTM, and associated utilities.
---

## Keras Recurrent Layer Import

Recurrent layer support is implemented in the [layers/recurrent](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/recurrent) package.

### Support Summary

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| SimpleRNN | SimpleRnn | Yes |
| GRU | — | No |
| LSTM | LSTM | Yes |
| ConvLSTM2D | — | No |

---

## KerasSimpleRnn

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/recurrent/KerasSimpleRnn.java)

Imports a Keras `SimpleRNN` layer as a DL4J `SimpleRnn` layer.

### Constructor

```java
public KerasSimpleRnn(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

Pass-through constructor. `kerasVersion` is the major Keras version (1 or 2).

### getSimpleRnnLayer

```java
public Layer getSimpleRnnLayer()
```

Returns the configured DL4J `SimpleRnn` layer.

**Parameters (from Keras config):**
- `layerConfig` — dictionary containing Keras layer configuration

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Returns the output `InputType` for this RNN layer.

**Parameters:**
- `inputType` — array of input `InputType` objects

**Returns:** output `InputType`

### getNumParams

```java
public int getNumParams()
```

Returns the number of trainable parameters. For a SimpleRNN with hidden size `h` and input size `i`, this is `h*(h+i) + h` (recurrent weights + input weights + bias).

### getInputPreprocessor

```java
public InputPreProcessor getInputPreprocessor(InputType... inputType)
        throws InvalidKerasConfigurationException
```

Returns the appropriate DL4J `InputPreProcessor` if a format conversion is needed before the RNN layer (e.g., feed-forward to RNN).

### getUnroll

```java
public boolean getUnroll()
```

Returns whether this SimpleRNN layer should be unrolled for truncated BPTT. Corresponds to the Keras `unroll` parameter.

### setWeights

```java
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

Loads weights into the layer. The weight map uses DL4J parameter names derived from the Keras HDF5 structure.

---

## KerasLSTM

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/recurrent/KerasLSTM.java)

Imports a Keras `LSTM` layer as a DL4J `LSTM` layer. Both Keras 1 and Keras 2 LSTM configurations are supported.

### Constructor

```java
public KerasLSTM(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

### getLSTMLayer

```java
public Layer getLSTMLayer()
```

Returns the configured DL4J `LSTM` layer.

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

### getNumParams

```java
public int getNumParams()
```

Returns the number of trainable parameters (12 weight matrices for 4 gates × input + recurrent + bias terms).

### getInputPreprocessor

```java
public InputPreProcessor getInputPreprocessor(InputType... inputType)
        throws InvalidKerasConfigurationException
```

### setWeights

```java
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

Loads LSTM gate weight matrices (kernel, recurrent kernel) and biases from the weight map.

### getUnroll

```java
public boolean getUnroll()
```

Returns whether the LSTM should be unrolled for truncated BPTT.

### getGateActivationFromConfig

```java
public IActivation getGateActivationFromConfig(Map<String, Object> layerConfig)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Reads the LSTM gate (recurrent) activation function from the Keras layer configuration. Corresponds to the Keras `recurrent_activation` parameter (default: `hard_sigmoid`).

**Parameters:**
- `layerConfig` — dictionary containing the Keras layer configuration

**Returns:** DL4J `IActivation` for the gate activation

### getForgetBiasInitFromConfig

```java
public double getForgetBiasInitFromConfig(Map<String, Object> layerConfig, boolean train)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Reads the forget gate bias initialization value from the Keras config. Keras 2 uses `unit_forget_bias=True` by default, which initializes the forget gate bias to 1.

**Parameters:**
- `layerConfig` — dictionary containing the Keras layer configuration
- `train` — whether this is the training phase

**Returns:** forget gate bias initialization value

---

## KerasRnnUtils

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/recurrent/KerasRnnUtils.java)

Shared utilities for RNN layer import.

### getUnrollRecurrentLayer

```java
public static boolean getUnrollRecurrentLayer(KerasLayerConfiguration conf,
        Map<String, Object> layerConfig) throws InvalidKerasConfigurationException
```

Reads the `unroll` parameter from the Keras layer configuration. Unrolling converts the RNN to a fixed-length sequence of operations; this can improve speed on short sequences but uses more memory.

### getRecurrentDropout

```java
public static double getRecurrentDropout(KerasLayerConfiguration conf,
        Map<String, Object> layerConfig)
        throws UnsupportedKerasConfigurationException, InvalidKerasConfigurationException
```

Reads the recurrent dropout rate from the Keras layer configuration. **Note:** non-zero recurrent dropout rates are currently not supported in DL4J. If a model was trained with recurrent dropout, the dropout will be silently ignored at import time (inference behavior is unaffected).

---

## Input Shape Convention

DL4J RNN layers expect input in `[batch, features, timesteps]` format (channels-first time axis). Keras RNNs use `[batch, timesteps, features]`. The importer automatically inserts the necessary `RnnToFeedForwardPreProcessor` or `FeedForwardToRnnPreProcessor` at layer boundaries.

When constructing input arrays manually for inference, use `[batch, features, timesteps]`:

```java
// 8 samples, 10 features, 50 timesteps
INDArray input = Nd4j.rand(8, 10, 50);
INDArray output = model.output(input);
```

---

## Unsupported Layers

- **GRU**: the Keras GRU gate formulation differs from DL4J's. Models using GRU must be redesigned with LSTM or SimpleRNN before import.
- **ConvLSTM2D**: no DL4J equivalent. Models using ConvLSTM2D cannot currently be imported.

---

## See Also

- [Wrapper Layers (Bidirectional)](layers-wrappers.md)
- [Supported Features](supported-features.md)

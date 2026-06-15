---
title: Keras Wrapper Layer Import
description: DL4J equivalents and API reference for Keras wrapper layers — Bidirectional and TimeDistributed.
---

## Keras Wrapper Layer Import

Wrapper layers in Keras apply a transformation around an inner layer. Support is implemented in the [layers/wrappers](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/wrappers) package.

### Support Summary

| Keras Wrapper | DL4J Equivalent | Supported |
|---|---|---|
| Bidirectional | Bidirectional | Yes |
| TimeDistributed | — | No |

---

## KerasBidirectional

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/wrappers/KerasBidirectional.java)

Builds a DL4J `Bidirectional` layer from a Keras `Bidirectional` layer wrapper. The importer reads the wrapped RNN layer (LSTM or SimpleRNN) and constructs a DL4J `Bidirectional` wrapper around it.

### Constructor

```java
public KerasBidirectional(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

Pass-through constructor. `kerasVersion` is the major Keras version (1 or 2).

### getUnderlyingRecurrentLayer

```java
public Layer getUnderlyingRecurrentLayer()
```

Returns the underlying DL4J recurrent layer (e.g., `LSTM` or `SimpleRnn`) that was wrapped by the Keras `Bidirectional` wrapper.

**Throws:** `InvalidKerasConfigurationException` if the inner layer configuration is invalid; `UnsupportedKerasConfigurationException` if the inner layer type is not supported.

### getBidirectionalLayer

```java
public Bidirectional getBidirectionalLayer()
```

Returns the DL4J `Bidirectional` layer, which wraps the underlying recurrent layer and runs it in both forward and backward directions. The merge mode (sum, multiply, concat, average) is read from the Keras configuration.

**Merge mode mapping:**

| Keras merge_mode | DL4J Bidirectional.Mode |
|---|---|
| `"sum"` | ADD |
| `"mul"` | MUL |
| `"concat"` | CONCAT |
| `"ave"` | AVERAGE |

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Returns the output `InputType`. For `concat` merge mode, the output size is doubled relative to the inner RNN's output size.

**Parameters:**
- `inputType` — array of input `InputType` objects

### getNumParams

```java
public int getNumParams()
```

Returns the total number of trainable parameters (twice the underlying layer's count, once for each direction).

### getInputPreprocessor

```java
public InputPreProcessor getInputPreprocessor(InputType... inputType)
        throws InvalidKerasConfigurationException
```

Returns the appropriate `InputPreProcessor` for format conversion before the bidirectional layer.

### setWeights

```java
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

Loads weights for both the forward and backward RNN layers from the weight map. Keras stores bidirectional weights with `forward_` and `backward_` prefixes; the importer splits these into the appropriate DL4J parameter names.

**Parameters:**
- `weights` — map from parameter name to `INDArray`

---

## Example

```python
# Python: Bidirectional LSTM
from keras.models import Sequential
from keras.layers import Embedding, Bidirectional, LSTM, Dense

model = Sequential()
model.add(Embedding(10000, 128, input_length=50))
model.add(Bidirectional(LSTM(64)))
model.add(Dense(1, activation='sigmoid'))
model.compile(optimizer='adam', loss='binary_crossentropy')
model.save('bidirectional_lstm.h5')
```

```java
// Java: import
import org.deeplearning4j.nn.modelimport.keras.KerasModelImport;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

MultiLayerNetwork model = KerasModelImport
    .importKerasSequentialModelAndWeights("bidirectional_lstm.h5");

// Input: [batch, sequence_length] integer token ids
INDArray input = Nd4j.create(new float[]{1, 5, 3, 0, 0}, new int[]{1, 5});
INDArray output = model.output(input);
// output shape: [1, 1] (single sigmoid output)
```

---

## TimeDistributed

`TimeDistributed` is not currently supported. This wrapper applies an inner layer (e.g., Dense) independently to each timestep of a sequence.

**Workarounds:**
- For dense operations on sequences, consider replacing `TimeDistributed(Dense(n))` with a 1D convolutional layer with kernel size 1, which applies the same transformation at each timestep.
- For feature extraction followed by an RNN, restructure the architecture to use a convolutional encoder before the recurrent layers.

---

## Notes

- The `Bidirectional` wrapper works with both `LSTM` and `SimpleRNN` inner layers. Wrapping a `GRU` is not supported because `GRU` itself is unsupported.
- For `Bidirectional` with `return_sequences=True` on the inner LSTM (which is needed when stacking bidirectional layers), both the forward and backward sequences are merged and passed to the next layer. Verify the expected shape from the Keras model summary before constructing manual inputs.

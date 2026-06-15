---
title: Keras Embedding Layer Import
description: DL4J equivalents and API reference for Keras embedding layers — Embedding mapped to EmbeddingSequenceLayer.
---

## Keras Embedding Layer Import

Embedding layer support is implemented in the [layers/embeddings](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/embeddings) package.

### Support Summary

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| Embedding | EmbeddingSequenceLayer | Yes |

---

## KerasEmbedding

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/embeddings/KerasEmbedding.java)

Imports a Keras `Embedding` layer as a DL4J `EmbeddingSequenceLayer`. The embedding matrix (shape `[vocab_size, embedding_dim]`) is loaded from the HDF5 file.

### Constructor (no-arg, for unit tests)

```java
public KerasEmbedding() throws UnsupportedKerasConfigurationException
```

Pass-through constructor used in unit tests.

### getEmbeddingLayer

```java
public EmbeddingSequenceLayer getEmbeddingLayer()
```

Returns the configured DL4J `EmbeddingSequenceLayer`. Configuration parameters derived from Keras:

- `input_dim` → vocabulary size
- `output_dim` → embedding dimension
- `input_length` → sequence length (optional)
- `embeddings_initializer` → weight initializer
- `embeddings_regularizer` → L1/L2 regularization on the embedding matrix
- `mask_zero` → whether to mask the zero index (affects masking in downstream RNN layers)

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Returns an `InputType.recurrent(embeddingDim)` reflecting the sequence output of the embedding layer.

**Parameters:**
- `inputType` — array of input `InputType` objects

**Returns:** output `InputType`

### getNumParams

```java
public int getNumParams()
```

Returns 1 (the embedding matrix is a single parameter tensor).

### setWeights

```java
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

Loads the embedding matrix from the weight map. In the HDF5 file, Keras stores this as `embeddings_0` or a similar key depending on the version.

**Parameters:**
- `weights` — map from parameter name to `INDArray` containing the embedding matrix

---

## Keras Embedding Parameters

| Keras parameter | DL4J equivalent |
|---|---|
| `input_dim` | `nIn()` (vocabulary size) |
| `output_dim` | `nOut()` (embedding dimension) |
| `input_length` | sequence length |
| `embeddings_initializer` | weight initializer |
| `embeddings_regularizer` | weight regularization |
| `mask_zero` | masking configuration |
| `trainable` | weight update flag |

---

## Example

```python
# Python: define and save a model with Embedding
from keras.models import Sequential
from keras.layers import Embedding, LSTM, Dense

model = Sequential()
model.add(Embedding(input_dim=10000, output_dim=128, input_length=50))
model.add(LSTM(64))
model.add(Dense(2, activation='softmax'))
model.compile(optimizer='adam', loss='binary_crossentropy')
model.save('embedding_lstm.h5')
```

```java
// Java: import and run inference
import org.deeplearning4j.nn.modelimport.keras.KerasModelImport;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

MultiLayerNetwork model = KerasModelImport
    .importKerasSequentialModelAndWeights("embedding_lstm.h5");

// Input: integer token indices, shape [batch, sequence_length]
// DL4J EmbeddingSequenceLayer expects float representation of integer indices
INDArray input = Nd4j.create(new float[]{1, 5, 3, 42, 0}, new int[]{1, 5});
INDArray output = model.output(input);
```

---

## Notes

- DL4J's `EmbeddingSequenceLayer` handles sequence inputs natively and is preferred over `EmbeddingLayer` for Keras Embedding import because it preserves the temporal dimension for downstream RNN layers.
- Sparse embedding updates (Keras `embeddings_constraint`) are read but DL4J uses dense gradient updates for the embedding matrix.
- If `mask_zero=True` was used in Keras, downstream LSTM/SimpleRNN layers will apply masking. Verify that the DL4J model handles masking as expected for your data.

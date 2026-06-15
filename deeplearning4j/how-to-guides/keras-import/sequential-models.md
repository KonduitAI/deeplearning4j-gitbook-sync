---
title: "Sequential Model Import"
description: "Importing Keras Sequential models as MultiLayerNetwork"
---

## Importing Keras Sequential Models

Keras `Sequential` models are linear stacks of layers with a single input and a single output. They map directly to DL4J's `MultiLayerNetwork`.

---

## Define a Sequential Model in Keras

```python
from keras.models import Sequential
from keras.layers import Dense

model = Sequential()
model.add(Dense(units=64, activation='relu', input_dim=100))
model.add(Dense(units=10, activation='softmax'))
model.compile(loss='categorical_crossentropy', optimizer='sgd', metrics=['accuracy'])
```

---

## Saving the Model

Keras provides several serialization options, each corresponding to a different import method in DL4J:

```python
# Option 1: Save full model — architecture, weights, and training config
model.save('full_model.h5')

# Option 2: Save architecture as JSON
model_json = model.to_json()
with open("model_config.json", "w") as f:
    f.write(model_json)

# Option 3: Save weights only
model.save_weights('model_weights.h5')
```

If you intend to continue training the model in DL4J after import, use `model.save(...)` so that the training configuration (optimizer settings, loss function) is preserved. The other options omit training configuration.

---

## Loading the Model in Java

### Load Full Model (Recommended)

```java
import org.deeplearning4j.nn.modelimport.keras.KerasModelImport;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.nd4j.linalg.io.ClassPathResource;

String fullModel = new ClassPathResource("full_model.h5").getFile().getPath();
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(fullModel);
```

If the Keras model was not compiled (no training configuration in the HDF5 file), pass `false` for `enforceTrainingConfig`:

```java
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(fullModel, false);
```

### Load from Separate Config and Weights Files

```java
String modelJson    = new ClassPathResource("model_config.json").getFile().getPath();
String modelWeights = new ClassPathResource("model_weights.h5").getFile().getPath();

MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(modelJson, modelWeights);
```

### Load Configuration Only

```java
String modelJson = new ClassPathResource("model_config.json").getFile().getPath();
MultiLayerConfiguration config = KerasModelImport.importKerasSequentialConfiguration(modelJson);

MultiLayerNetwork model = new MultiLayerNetwork(config);
model.init();
```

---

## Running Inference

After import, inference follows standard DL4J conventions:

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

// Batch of 256 samples, each with 100 features
INDArray input = Nd4j.create(256, 100);
INDArray output = model.output(input);

// output shape: [256, 10]
```

---

## Training After Import

If the model was imported with training configuration, you can continue training directly:

```java
// Create dummy training data
INDArray features = Nd4j.rand(1000, 100);
INDArray labels   = Nd4j.zeros(1000, 10);
// ... populate labels ...

org.nd4j.linalg.dataset.DataSet ds = new org.nd4j.linalg.dataset.DataSet(features, labels);
model.fit(ds);
```

For larger datasets, use a `DataSetIterator`:

```java
org.nd4j.linalg.dataset.api.iterator.DataSetIterator iterator = /* your iterator */;
model.fit(iterator);
```

---

## KerasSequentialModel API Reference

The `KerasSequentialModel` class underlies `KerasModelImport` for Sequential models.

---

### KerasSequentialModel

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/KerasSequentialModel.java)

Builds a `MultiLayerNetwork` from a Keras Sequential model configuration.

#### getMultiLayerConfiguration

```java
public MultiLayerConfiguration getMultiLayerConfiguration()
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Returns the `MultiLayerConfiguration` from the parsed Keras Sequential model configuration.

---

#### getMultiLayerNetwork

```java
public MultiLayerNetwork getMultiLayerNetwork()
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Builds and returns a `MultiLayerNetwork` from this Keras Sequential model configuration, with weights loaded.

---

#### getMultiLayerNetwork (with weight control)

```java
public MultiLayerNetwork getMultiLayerNetwork(boolean importWeights)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Builds and returns a `MultiLayerNetwork`. Pass `importWeights=false` to get a randomly-initialized network with the correct architecture.

**Parameters:**
- `importWeights` — whether to import weights from the HDF5 source

---

## Example: CNN for Image Classification

**Python**

```python
from keras.models import Sequential
from keras.layers import Conv2D, MaxPooling2D, Flatten, Dense, Dropout

model = Sequential()
model.add(Conv2D(32, (3, 3), activation='relu', input_shape=(28, 28, 1)))
model.add(MaxPooling2D((2, 2)))
model.add(Conv2D(64, (3, 3), activation='relu'))
model.add(MaxPooling2D((2, 2)))
model.add(Flatten())
model.add(Dense(128, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(10, activation='softmax'))

model.compile(optimizer='adam', loss='categorical_crossentropy', metrics=['accuracy'])
model.save('cnn_classifier.h5')
```

**Java**

```java
String modelPath = new ClassPathResource("cnn_classifier.h5").getFile().getPath();
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(modelPath);

System.out.println(model.summary());

// Input: batch of 4 grayscale 28x28 images
// DL4J uses NCHW format: [batch, channels, height, width]
INDArray input = Nd4j.rand(4, 1, 28, 28);
INDArray output = model.output(input);

// output shape: [4, 10]
System.out.println("Predictions shape: " + java.util.Arrays.toString(output.shape()));
```

---

## Example: LSTM for Sequence Classification

**Python**

```python
from keras.models import Sequential
from keras.layers import LSTM, Dense

model = Sequential()
model.add(LSTM(64, input_shape=(50, 10)))  # 50 timesteps, 10 features
model.add(Dense(5, activation='softmax'))

model.compile(optimizer='adam', loss='categorical_crossentropy', metrics=['accuracy'])
model.save('lstm_classifier.h5')
```

**Java**

```java
String modelPath = new ClassPathResource("lstm_classifier.h5").getFile().getPath();
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(modelPath);

// DL4J RNN input: [batch, features, timesteps]
INDArray input = Nd4j.rand(8, 10, 50);
INDArray output = model.output(input);

// output shape: [8, 5]
```

---

## Troubleshooting

**Model loads as ComputationGraph by mistake**: ensure you call `importKerasSequentialModelAndWeights` (not `importKerasModelAndWeights`) for Sequential models.

**Input shape mismatch**: DL4J uses NCHW for images (channels first) while Keras defaults to NHWC (channels last). The importer handles this transpose automatically for Conv2D and pooling layers. However, if you build `INDArray` inputs manually, verify the expected shape from `model.summary()`.

**LSTM input ordering**: DL4J RNN layers expect `[batch, features, timesteps]`, which is the transpose of Keras's `[batch, timesteps, features]`. The importer adds the necessary `RnnToFeedForwardPreProcessor` and `FeedForwardToRnnPreProcessor` where needed, but verify the input array ordering when constructing inputs manually.

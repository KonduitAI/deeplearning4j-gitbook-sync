---
title: "Functional Model Import"
description: "Importing Keras Functional API models as ComputationGraph"
---

## Importing Keras Functional API Models

Keras Functional API models (`keras.models.Model`) are directed acyclic graphs with arbitrary topology: multiple inputs, multiple outputs, shared layers, and residual or skip connections are all supported. In DL4J, these map to `ComputationGraph`.

---

## Define a Functional Model in Keras

```python
from keras.models import Model
from keras.layers import Dense, Input

inputs = Input(shape=(100,))
x = Dense(64, activation='relu')(inputs)
predictions = Dense(10, activation='softmax')(x)

model = Model(inputs=inputs, outputs=predictions)
model.compile(loss='categorical_crossentropy', optimizer='sgd', metrics=['accuracy'])
```

---

## Saving the Model

Keras provides several serialization options. Each is supported for import:

```python
# Option 1: Save everything in a single HDF5 file (recommended)
model.save('full_model.h5')

# Option 2: Save architecture as JSON
model_json = model.to_json()
with open("model_config.json", "w") as f:
    f.write(model_json)

# Option 3: Save weights only
model.save_weights('model_weights.h5')
```

If you want to continue training in DL4J after import, use `model.save(...)` (Option 1). It preserves the training configuration (optimizer, loss, metrics). The other options do not include training configuration.

---

## Loading the Model in Java

### Load Full Model

This is the recommended path when the model was saved with `model.save()`:

```java
String fullModel = new ClassPathResource("full_model.h5").getFile().getPath();
ComputationGraph model = KerasModelImport.importKerasModelAndWeights(fullModel);
```

If the Keras model was not compiled before saving, skip training configuration enforcement:

```java
ComputationGraph model = KerasModelImport.importKerasModelAndWeights(fullModel, false);
```

### Load from Separate Config and Weights

When architecture and weights were saved separately:

```java
String modelJson    = new ClassPathResource("model_config.json").getFile().getPath();
String modelWeights = new ClassPathResource("model_weights.h5").getFile().getPath();

ComputationGraph model = KerasModelImport.importKerasModelAndWeights(modelJson, modelWeights);
```

### Load Configuration Only

To load just the graph topology without weights:

```java
String modelJson = new ClassPathResource("model_config.json").getFile().getPath();
ComputationGraphConfiguration config = KerasModelImport.importKerasModelConfiguration(modelJson);

ComputationGraph model = new ComputationGraph(config);
model.init();
```

---

## Running Inference

After import, inference on a `ComputationGraph` follows DL4J conventions. `output()` accepts one or more `INDArray` inputs and returns an array of outputs (one per output node):

```java
INDArray input = Nd4j.create(256, 100);
INDArray[] outputs = model.output(input);

// For a single-output model
INDArray predictions = outputs[0];
```

For models with multiple inputs:

```java
INDArray input1 = Nd4j.rand(32, 100);
INDArray input2 = Nd4j.rand(32, 50);
INDArray[] outputs = model.output(input1, input2);
```

---

## KerasModel API Reference

The `KerasModel` class underlies `KerasModelImport` when importing Functional API models. Use it directly when you need more control:

---

### KerasModel

[source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/KerasModel.java)

Builds a `ComputationGraph` from a Keras Functional API model configuration.

#### Constructor

```java
public KerasModel(KerasModelBuilder modelBuilder)
        throws UnsupportedKerasConfigurationException, IOException, InvalidKerasConfigurationException
```

Recommended builder-pattern constructor. Use `KerasModelBuilder` to configure the import (model file, training config enforcement, etc.) before building.

**Parameters:**
- `modelBuilder` — a configured `KerasModelBuilder` instance

**Throws:**
- `IOException` — IO exception
- `InvalidKerasConfigurationException` — invalid Keras configuration
- `UnsupportedKerasConfigurationException` — unsupported Keras configuration

---

#### getComputationGraphConfiguration

```java
public ComputationGraphConfiguration getComputationGraphConfiguration()
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Returns the `ComputationGraphConfiguration` from the parsed Keras model configuration. Training-related settings that are not supported (e.g., unknown regularizers) are ignored or throw exceptions depending on the `enforceTrainingConfig` flag.

---

#### getComputationGraph

```java
public ComputationGraph getComputationGraph()
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Builds and returns a `ComputationGraph` from this model configuration, with weights loaded.

---

#### getComputationGraph (with weight control)

```java
public ComputationGraph getComputationGraph(boolean importWeights)
        throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Builds and returns a `ComputationGraph`. Pass `importWeights=false` to obtain a randomly-initialized graph with the correct architecture.

**Parameters:**
- `importWeights` — whether to load weights from the HDF5 source

---

## Example: ResNet-style Skip Connections

The Functional API allows residual connections that cannot be expressed in a `Sequential` model. DL4J's `ComputationGraph` handles these natively.

**Python**

```python
from keras.models import Model
from keras.layers import Dense, Input, Add

inputs = Input(shape=(256,))
x = Dense(256, activation='relu')(inputs)
residual = Add()([x, inputs])  # skip connection
output = Dense(10, activation='softmax')(residual)

model = Model(inputs=inputs, outputs=output)
model.save('resnet_like.h5')
```

**Java**

```java
String modelPath = new ClassPathResource("resnet_like.h5").getFile().getPath();
ComputationGraph model = KerasModelImport.importKerasModelAndWeights(modelPath, false);

INDArray input = Nd4j.rand(4, 256);
INDArray[] outputs = model.output(input);
System.out.println(outputs[0]);
```

---

## Example: Multi-Input Model

```python
from keras.models import Model
from keras.layers import Dense, Input, Concatenate

input_a = Input(shape=(100,), name='input_a')
input_b = Input(shape=(50,),  name='input_b')

merged = Concatenate()([input_a, input_b])
output = Dense(10, activation='softmax')(merged)

model = Model(inputs=[input_a, input_b], outputs=output)
model.save('multi_input.h5')
```

**Java**

```java
String modelPath = new ClassPathResource("multi_input.h5").getFile().getPath();
ComputationGraph model = KerasModelImport.importKerasModelAndWeights(modelPath, false);

INDArray inputA = Nd4j.rand(4, 100);
INDArray inputB = Nd4j.rand(4, 50);
INDArray[] outputs = model.output(inputA, inputB);
```

---

## Troubleshooting

**Wrong number of inputs at inference time**: ensure that the order of `INDArray` arguments to `model.output(...)` matches the order of inputs in the Keras model's `inputs` list.

**ComputationGraph vs MultiLayerNetwork**: functional models always load as `ComputationGraph`. Passing a Functional API model path to `importKerasSequentialModelAndWeights` will fail. Use `importKerasModelAndWeights` instead.

**Merge layer axis**: the `Dot` merge layer is not supported. All other standard Keras merge layers (Add, Multiply, Subtract, Average, Maximum, Concatenate) are supported.

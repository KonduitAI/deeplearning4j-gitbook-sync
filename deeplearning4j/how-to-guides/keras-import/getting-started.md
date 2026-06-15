---
title: "Keras Import Getting Started"
description: "Step-by-step guide to importing Keras models — saving in Python, loading in Java, and running inference"
---

## Getting Started with Keras Model Import

This guide walks through the complete workflow: defining and saving a Keras model in Python, adding the Maven dependency, loading the model in Java, and running inference. A complete working example is provided at the end.

---

## Prerequisites

- Java 8 or later
- Maven 3.x
- A trained Keras model (Keras 1.x or 2.x with any backend: TensorFlow, Theano, CNTK)

---

## Step 1: Save Your Keras Model in Python

Keras models must be serialized before they can be imported. There are three serialization modes, each suitable for different scenarios.

### Option A: Save the Full Model (Recommended)

This saves the model architecture, weights, and training configuration together in a single HDF5 file:

```python
model.save('my_model.h5')
```

This is the recommended approach. When you load this file in DL4J, you get a fully configured model including weights. If the model was compiled in Keras, training configuration (optimizer, loss, metrics) is also preserved.

### Option B: Save Architecture and Weights Separately

Save the JSON configuration and weights independently:

```python
# Save architecture
model_json = model.to_json()
with open("model_config.json", "w") as f:
    f.write(model_json)

# Save weights
model.save_weights('model_weights.h5')
```

Use this when you want to version architecture and weights separately. Note that training configuration is not included.

### Option C: Architecture Only (No Weights)

```python
model_json = model.to_json()
with open("model_config.json", "w") as f:
    f.write(model_json)
```

Use this to inspect or configure the model before initializing weights independently.

---

## Step 2: Add the Maven Dependency

Add the following dependency to your `pom.xml`. The version should match the rest of your DL4J dependencies:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-modelimport</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

You also need ND4J with a backend. For CPU:

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native-platform</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

For GPU (CUDA):

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.6-platform</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

---

## Step 3: Load a Sequential Model as MultiLayerNetwork

A Keras `Sequential` model maps to a DL4J `MultiLayerNetwork`.

### Loading from a Full Model File

```java
import org.deeplearning4j.nn.modelimport.keras.KerasModelImport;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.deeplearning4j.util.ModelSerializer;
import org.nd4j.linalg.io.ClassPathResource;

String modelPath = new ClassPathResource("my_model.h5").getFile().getPath();

// Load full model (architecture + weights + training config)
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(modelPath);
```

If the Keras model was not compiled (no training configuration), use:

```java
// false = do not enforce training config
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(modelPath, false);
```

### Loading from Separate Config and Weights Files

```java
String configPath  = new ClassPathResource("model_config.json").getFile().getPath();
String weightsPath = new ClassPathResource("model_weights.h5").getFile().getPath();

MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(configPath, weightsPath);
```

### Loading Configuration Only (No Weights)

```java
String configPath = new ClassPathResource("model_config.json").getFile().getPath();
MultiLayerConfiguration config = KerasModelImport.importKerasSequentialConfiguration(configPath);
MultiLayerNetwork model = new MultiLayerNetwork(config);
model.init();
```

---

## Step 4: Load a Functional API Model as ComputationGraph

A Keras `Model` (Functional API) maps to a DL4J `ComputationGraph`, which supports arbitrary graph topologies.

### Loading from a Full Model File

```java
import org.deeplearning4j.nn.graph.ComputationGraph;

String modelPath = new ClassPathResource("full_model.h5").getFile().getPath();

ComputationGraph model = KerasModelImport.importKerasModelAndWeights(modelPath);
```

Without training configuration enforcement:

```java
ComputationGraph model = KerasModelImport.importKerasModelAndWeights(modelPath, false);
```

### Loading from Separate Files

```java
String configPath  = new ClassPathResource("model_config.json").getFile().getPath();
String weightsPath = new ClassPathResource("model_weights.h5").getFile().getPath();

ComputationGraph model = KerasModelImport.importKerasModelAndWeights(configPath, weightsPath);
```

---

## Step 5: Run Inference

After import, the model is a standard DL4J network and inference works identically to a natively-built DL4J model.

### MultiLayerNetwork Inference

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

// Create a batch of 8 input vectors, each of length 100
INDArray input = Nd4j.rand(8, 100);

// Run forward pass
INDArray output = model.output(input);

System.out.println("Output shape: " + Arrays.toString(output.shape()));
// Output shape: [8, 10]  (8 samples, 10 output classes)
```

### ComputationGraph Inference

```java
INDArray input = Nd4j.rand(8, 100);

// output() returns an array of outputs, one per output node
INDArray[] outputs = model.output(input);

System.out.println("Output shape: " + Arrays.toString(outputs[0].shape()));
```

---

## Step 6: Save the Imported Model

After import, save the model in DL4J's native format to avoid re-importing from Keras on subsequent loads:

```java
File saveLocation = new File("model.zip");
ModelSerializer.writeModel(model, saveLocation, true); // true = save updater state

// Restore later:
MultiLayerNetwork restored = ModelSerializer.restoreMultiLayerNetwork(saveLocation);
```

---

## Complete Example: Sequential MLP

The following is a self-contained example from Python training to Java inference.

**Python**

```python
from keras.models import Sequential
from keras.layers import Dense, Dropout
import numpy as np

# Build a simple classifier
model = Sequential()
model.add(Dense(128, activation='relu', input_dim=784))
model.add(Dropout(0.2))
model.add(Dense(64, activation='relu'))
model.add(Dropout(0.2))
model.add(Dense(10, activation='softmax'))

model.compile(
    optimizer='adam',
    loss='categorical_crossentropy',
    metrics=['accuracy']
)

# Train on dummy data
x_train = np.random.rand(1000, 784)
y_train = np.eye(10)[np.random.randint(0, 10, 1000)]
model.fit(x_train, y_train, epochs=3, batch_size=32)

# Save
model.save('mlp_classifier.h5')
print("Model saved.")
```

**Java**

```java
import org.deeplearning4j.nn.modelimport.keras.KerasModelImport;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.io.ClassPathResource;

import java.io.File;

public class KerasImportExample {

    public static void main(String[] args) throws Exception {
        // Path to the saved Keras model
        String modelPath = new ClassPathResource("mlp_classifier.h5").getFile().getPath();

        // Import the model
        MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(modelPath);

        System.out.println("Model imported successfully.");
        System.out.println(model.summary());

        // Run inference on a batch of 4 samples
        INDArray input = Nd4j.rand(4, 784);
        INDArray output = model.output(input);

        System.out.println("Predictions:");
        System.out.println(output);

        // Save in DL4J format for future use
        File savedModel = new File("mlp_classifier_dl4j.zip");
        org.deeplearning4j.util.ModelSerializer.writeModel(model, savedModel, true);
        System.out.println("Model saved to: " + savedModel.getAbsolutePath());
    }
}
```

---

## Common Issues

### Training configuration not found

**Symptom**: `InvalidKerasConfigurationException: Could not find training configuration`

**Fix**: The model was not compiled in Keras before saving, so no training config exists in the HDF5 file. Pass `false` as the `enforceTrainingConfig` parameter:

```java
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(path, false);
```

### Unsupported layer

**Symptom**: `UnsupportedKerasConfigurationException`

**Fix**: The model uses a layer not yet supported. Check [Supported Features](./supported-features). Consider replacing the unsupported layer with a supported equivalent, or implement a custom layer mapper.

### ClassPathResource not found

**Symptom**: `IOException: Could not find resource`

**Fix**: Ensure the `.h5` file is placed in `src/main/resources/` of your Maven project, or provide an absolute file path directly.

---

## Next Steps

- [Sequential Model](./sequential-model) — detailed API for Sequential model import
- [Functional Model](./functional-model) — detailed API for Functional model import
- [API Reference](./model-import-api) — all `KerasModelImport` method signatures
- [Supported Features](./supported-features) — complete layer, activation, optimizer, and loss support matrix

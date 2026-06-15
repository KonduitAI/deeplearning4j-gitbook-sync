---
title: "Model Persistence"
description: "Saving and loading neural networks — ModelSerializer, saving normalizers, and model format details"
---

# Model Persistence

Eclipse Deeplearning4j uses `ModelSerializer` (`org.deeplearning4j.util.ModelSerializer`) as the primary utility for saving and restoring neural networks. Models are saved as `.zip` archives containing the network configuration, trained parameters, and optionally the optimizer state (updater) and data normalizers.

---

## Saving Models

### MultiLayerNetwork

```java
import org.deeplearning4j.util.ModelSerializer;

File modelFile = new File("/tmp/my-model.zip");

// Save with updater state (recommended if you plan to continue training)
ModelSerializer.writeModel(model, modelFile, true);

// Save without updater state (smaller file, inference-only)
ModelSerializer.writeModel(model, modelFile, false);
```

### ComputationGraph

The same `writeModel` overloads work for `ComputationGraph`:

```java
ComputationGraph graph = /* trained graph */;
ModelSerializer.writeModel(graph, new File("/tmp/my-graph.zip"), true);
```

### Saving to an OutputStream

Useful when writing to cloud storage, HTTP responses, or non-file destinations:

```java
try (OutputStream os = new FileOutputStream("/tmp/model.zip")) {
    ModelSerializer.writeModel(model, os, true);
}
```

### Saving with a Normalizer

If you normalise your input data, save the normalizer alongside the model so it can be restored together:

```java
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerStandardize;

NormalizerStandardize normalizer = new NormalizerStandardize();
normalizer.fit(trainIter);

// Write model + normalizer in one file
ModelSerializer.writeModel(model, modelFile, true, normalizer);
```

The normalizer is stored as a serialised entry inside the same `.zip` file.

---

## Loading Models

### MultiLayerNetwork

```java
import org.deeplearning4j.util.ModelSerializer;

// Load from file, with updater state
MultiLayerNetwork model = ModelSerializer.restoreMultiLayerNetwork(modelFile);

// Load without updater state (faster, less memory)
MultiLayerNetwork model = ModelSerializer.restoreMultiLayerNetwork(modelFile, false);

// Load from path string
MultiLayerNetwork model = ModelSerializer.restoreMultiLayerNetwork("/tmp/my-model.zip");

// Load from InputStream
try (InputStream is = new FileInputStream(modelFile)) {
    MultiLayerNetwork model = ModelSerializer.restoreMultiLayerNetwork(is, true);
}
```

### ComputationGraph

```java
ComputationGraph graph = ModelSerializer.restoreComputationGraph(modelFile);
ComputationGraph graph = ModelSerializer.restoreComputationGraph(modelFile, false);
ComputationGraph graph = ModelSerializer.restoreComputationGraph("/tmp/my-graph.zip");

try (InputStream is = new FileInputStream(modelFile)) {
    ComputationGraph graph = ModelSerializer.restoreComputationGraph(is, true);
}
```

### Restoring a Normalizer

```java
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerStandardize;

// Restore model and normalizer together
Pair<MultiLayerNetwork, NormalizerStandardize> restored =
    ModelSerializer.restoreMultiLayerNetworkAndNormalizer(modelFile, true);

MultiLayerNetwork model      = restored.getFirst();
NormalizerStandardize norm   = restored.getSecond();

// Apply normalizer to new inference data
norm.transform(inputFeatures);
INDArray output = model.output(inputFeatures);
```

For `ComputationGraph`:

```java
Pair<ComputationGraph, NormalizerStandardize> restored =
    ModelSerializer.restoreComputationGraphAndNormalizer(modelFile, true);
```

---

## The .zip File Format

A model file saved by `ModelSerializer` is a standard `.zip` archive. You can inspect its contents with any unzip tool. The entries are:

| Entry name | Contents |
|---|---|
| `configuration.json` | JSON serialisation of the network configuration (`MultiLayerConfiguration` or `ComputationGraphConfiguration`). Includes layer types, hyperparameters, activation functions, etc. |
| `coefficients.bin` | Flat binary array of all trainable parameters (weights and biases) in the order they appear in the network. Uses ND4J's binary format. |
| `updaterState.bin` | Serialised optimizer state (momentum buffers, Adam m/v estimates, etc.). Only present when `saveUpdater=true`. |
| `normalizer.bin` | Serialised `DataNormalization` object. Only present when a normalizer is saved. |

Because the format is a zip file, you can add arbitrary objects to an existing model file using the object serialization API:

```java
// Attach a custom metadata object to an existing model file
ModelSerializer.addObjectToFile(modelFile, "myMetadata", mySerializableObject);

// Retrieve it later
MyMetadata meta = ModelSerializer.getObjectFromFile(modelFile, "myMetadata");
```

---

## Saving for Inference Only

When deploying a model for inference you do not need the updater state. Excluding it reduces file size — sometimes substantially for large models with stateful updaters like Adam.

```java
// false = don't save updater state
ModelSerializer.writeModel(model, inferenceFile, false);
```

When restoring for inference, also pass `false` to skip loading updater state:

```java
MultiLayerNetwork model = ModelSerializer.restoreMultiLayerNetwork(inferenceFile, false);
```

---

## RNG Seed After Restore

If your model uses stochastic regularisation (Dropout, DropConnect, etc.), the random number generator (RNG) state is not saved. To guarantee reproducible results across sessions, set the RNG seed immediately after restoring:

```java
import org.nd4j.linalg.factory.Nd4j;

Nd4j.getRandom().setSeed(12345);
MultiLayerNetwork model = ModelSerializer.restoreMultiLayerNetwork(modelFile);
```

---

## Appending a Normalizer to an Existing File

If you saved a model without a normalizer and want to add one later:

```java
import org.deeplearning4j.util.ModelSerializer;

NormalizerStandardize normalizer = /* fitted normalizer */;
ModelSerializer.addNormalizerToModel(modelFile, normalizer);
```

---

## API Reference

| Method | Description |
|---|---|
| `writeModel(Model, File, boolean)` | Save model to file. `boolean` = save updater state. |
| `writeModel(Model, File, boolean, DataNormalization)` | Save model and normalizer to file. |
| `writeModel(Model, String, boolean)` | Save model to path string. |
| `writeModel(Model, OutputStream, boolean)` | Save model to output stream. |
| `writeModel(Model, OutputStream, boolean, DataNormalization)` | Save model and normalizer to output stream. |
| `restoreMultiLayerNetwork(File)` | Load `MultiLayerNetwork` from file (with updater). |
| `restoreMultiLayerNetwork(File, boolean)` | Load `MultiLayerNetwork`; `boolean` = load updater. |
| `restoreMultiLayerNetwork(String)` | Load from path string. |
| `restoreMultiLayerNetwork(InputStream, boolean)` | Load from stream. |
| `restoreMultiLayerNetworkAndNormalizer(File, boolean)` | Load model and normalizer as a `Pair`. |
| `restoreComputationGraph(File)` | Load `ComputationGraph` from file. |
| `restoreComputationGraph(File, boolean)` | Load with or without updater. |
| `restoreComputationGraph(String)` | Load from path string. |
| `restoreComputationGraph(InputStream, boolean)` | Load from stream. |
| `restoreComputationGraphAndNormalizer(File, boolean)` | Load graph and normalizer as a `Pair`. |
| `addNormalizerToModel(File, Normalizer)` | Append a normalizer to an already-saved model file. |
| `addObjectToFile(File, String, Object)` | Attach a serializable object under a named key. |
| `getObjectFromFile(File, String)` | Retrieve a named object from a model file. |

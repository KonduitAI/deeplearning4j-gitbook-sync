---
title: "Keras Import API Reference"
description: "KerasModelImport API — all import methods and configuration options"
---

## Keras Model Import API Reference

`KerasModelImport` is the main entry point for importing Keras models into DL4J. It reads stored Keras configurations and weights from:

- A single HDF5 archive storing model architecture, training configuration, and weights (produced by `model.save()`)
- A JSON file containing the model configuration combined with a separate HDF5 file containing weights (produced by `model.to_json()` and `model.save_weights()`)

[Source](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/KerasModelImport.java)

---

## Importing Functional API (ComputationGraph) Models

These methods import Keras Functional API models (`keras.models.Model`) as DL4J `ComputationGraph`.

---

### importKerasModelAndWeights (InputStream)

```java
public static ComputationGraph importKerasModelAndWeights(
        InputStream modelHdf5Stream,
        boolean enforceTrainingConfig)
        throws IOException, UnsupportedKerasConfigurationException, InvalidKerasConfigurationException
```

Load a Keras Functional API model saved using `model.save(...)` from an `InputStream`.

**Parameters:**
- `modelHdf5Stream` — `InputStream` containing the HDF5 archive
- `enforceTrainingConfig` — whether to enforce training configuration options; set to `false` if the model was not compiled

**Returns:** `ComputationGraph`

---

```java
public static ComputationGraph importKerasModelAndWeights(InputStream modelHdf5Stream)
        throws IOException, UnsupportedKerasConfigurationException, InvalidKerasConfigurationException
```

Load a Keras Functional API model from an `InputStream` with training configuration enforced by default.

**Parameters:**
- `modelHdf5Stream` — `InputStream` containing the HDF5 archive

**Returns:** `ComputationGraph`

---

### importKerasModelAndWeights (file path)

```java
public static ComputationGraph importKerasModelAndWeights(
        String modelHdf5Filename,
        int[] inputShape,
        boolean enforceTrainingConfig)
        throws IOException, UnsupportedKerasConfigurationException, InvalidKerasConfigurationException
```

Load a Keras Functional API model saved using `model.save(...)` from a file path. Use this overload for models that do not include input shape information (e.g., `include_top=False` models).

**Parameters:**
- `modelHdf5Filename` — path to the HDF5 archive
- `inputShape` — optional input shape; pass `null` if the model already contains shape information
- `enforceTrainingConfig` — whether to enforce training configuration options

**Returns:** `ComputationGraph`

---

```java
public static ComputationGraph importKerasModelAndWeights(
        String modelHdf5Filename,
        boolean enforceTrainingConfig)
        throws IOException, UnsupportedKerasConfigurationException, InvalidKerasConfigurationException
```

**Parameters:**
- `modelHdf5Filename` — path to the HDF5 archive
- `enforceTrainingConfig` — whether to enforce training configuration options

**Returns:** `ComputationGraph`

---

```java
public static ComputationGraph importKerasModelAndWeights(String modelHdf5Filename)
        throws IOException, UnsupportedKerasConfigurationException, InvalidKerasConfigurationException
```

Load a Keras Functional API model from a file path with training configuration enforced by default.

**Parameters:**
- `modelHdf5Filename` — path to the HDF5 archive

**Returns:** `ComputationGraph`

---

### importKerasModelAndWeights (JSON config + weights)

```java
public static ComputationGraph importKerasModelAndWeights(
        String modelJsonFilename,
        String weightsHdf5Filename,
        boolean enforceTrainingConfig)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Load a Keras Functional API model for which the configuration and weights were saved separately using `model.to_json()` and `model.save_weights(...)`.

**Parameters:**
- `modelJsonFilename` — path to the JSON file storing the model configuration
- `weightsHdf5Filename` — path to the HDF5 archive storing model weights
- `enforceTrainingConfig` — whether to enforce training configuration options

**Returns:** `ComputationGraph`

---

```java
public static ComputationGraph importKerasModelAndWeights(
        String modelJsonFilename,
        String weightsHdf5Filename)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

**Parameters:**
- `modelJsonFilename` — path to the JSON file
- `weightsHdf5Filename` — path to the HDF5 weights file

**Returns:** `ComputationGraph`

---

### importKerasModelConfiguration

```java
public static ComputationGraphConfiguration importKerasModelConfiguration(
        String modelJsonFilename,
        boolean enforceTrainingConfig)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Load only the model configuration (no weights) for a Keras Functional API model.

**Parameters:**
- `modelJsonFilename` — path to the JSON file
- `enforceTrainingConfig` — whether to enforce training configuration options

**Returns:** `ComputationGraphConfiguration`

---

```java
public static ComputationGraphConfiguration importKerasModelConfiguration(String modelJsonFilename)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

**Parameters:**
- `modelJsonFilename` — path to the JSON file

**Returns:** `ComputationGraphConfiguration`

---

## Importing Sequential (MultiLayerNetwork) Models

These methods import Keras `Sequential` models as DL4J `MultiLayerNetwork`.

---

### importKerasSequentialModelAndWeights (InputStream)

```java
public static MultiLayerNetwork importKerasSequentialModelAndWeights(
        InputStream modelHdf5Stream,
        boolean enforceTrainingConfig)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Load a Keras Sequential model saved using `model.save(...)` from an `InputStream`.

**Parameters:**
- `modelHdf5Stream` — `InputStream` containing the HDF5 archive
- `enforceTrainingConfig` — whether to enforce training configuration options

**Returns:** `MultiLayerNetwork`

---

```java
public static MultiLayerNetwork importKerasSequentialModelAndWeights(InputStream modelHdf5Stream)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

**Parameters:**
- `modelHdf5Stream` — `InputStream` containing the HDF5 archive

**Returns:** `MultiLayerNetwork`

---

### importKerasSequentialModelAndWeights (file path)

```java
public static MultiLayerNetwork importKerasSequentialModelAndWeights(
        String modelHdf5Filename,
        int[] inputShape,
        boolean enforceTrainingConfig)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Load a Keras Sequential model from a file path. Supply `inputShape` for models that omit input shape in their configuration.

**Parameters:**
- `modelHdf5Filename` — path to the HDF5 archive
- `inputShape` — optional input shape array; pass `null` if not needed
- `enforceTrainingConfig` — whether to enforce training configuration options

**Returns:** `MultiLayerNetwork`

---

```java
public static MultiLayerNetwork importKerasSequentialModelAndWeights(
        String modelHdf5Filename,
        boolean enforceTrainingConfig)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

**Parameters:**
- `modelHdf5Filename` — path to the HDF5 archive
- `enforceTrainingConfig` — whether to enforce training configuration options

**Returns:** `MultiLayerNetwork`

---

```java
public static MultiLayerNetwork importKerasSequentialModelAndWeights(String modelHdf5Filename)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

**Parameters:**
- `modelHdf5Filename` — path to the HDF5 archive

**Returns:** `MultiLayerNetwork`

---

### importKerasSequentialModelAndWeights (JSON config + weights)

```java
public static MultiLayerNetwork importKerasSequentialModelAndWeights(
        String modelJsonFilename,
        String weightsHdf5Filename,
        boolean enforceTrainingConfig)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Load a Keras Sequential model for which the configuration and weights were saved separately.

**Parameters:**
- `modelJsonFilename` — path to the JSON configuration file
- `weightsHdf5Filename` — path to the HDF5 weights file
- `enforceTrainingConfig` — whether to enforce training configuration options

**Returns:** `MultiLayerNetwork`

---

```java
public static MultiLayerNetwork importKerasSequentialModelAndWeights(
        String modelJsonFilename,
        String weightsHdf5Filename)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

**Parameters:**
- `modelJsonFilename` — path to the JSON configuration file
- `weightsHdf5Filename` — path to the HDF5 weights file

**Returns:** `MultiLayerNetwork`

---

### importKerasSequentialConfiguration

```java
public static MultiLayerConfiguration importKerasSequentialConfiguration(
        String modelJsonFilename,
        boolean enforceTrainingConfig)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Load only the model configuration for a Keras Sequential model. No weights are loaded.

**Parameters:**
- `modelJsonFilename` — path to the JSON configuration file
- `enforceTrainingConfig` — whether to enforce training configuration options

**Returns:** `MultiLayerConfiguration`

---

```java
public static MultiLayerConfiguration importKerasSequentialConfiguration(String modelJsonFilename)
        throws IOException, InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

**Parameters:**
- `modelJsonFilename` — path to the JSON configuration file

**Returns:** `MultiLayerConfiguration`

---

## Exception Types

| Exception | Meaning |
|---|---|
| `InvalidKerasConfigurationException` | The Keras configuration contains elements that are recognized but cannot be parsed or mapped |
| `UnsupportedKerasConfigurationException` | The Keras configuration contains features that are not supported by the importer |
| `IOException` | File not found or unreadable |

---

## enforceTrainingConfig Flag

The `enforceTrainingConfig` parameter controls how training-related configuration in the HDF5 is handled:

- `true` (default): an exception is thrown if any training configuration element cannot be parsed or mapped. Use this when you intend to continue training the model in DL4J.
- `false`: training configuration parsing errors are logged as warnings but do not stop import. Use this for inference-only workflows or when the Keras model was not compiled before saving.

---

## Method Selection Guide

| Situation | Method to use |
|---|---|
| Sequential model, single .h5 file | `importKerasSequentialModelAndWeights(String)` |
| Sequential model, config + weights separate | `importKerasSequentialModelAndWeights(String, String)` |
| Sequential model, config only | `importKerasSequentialConfiguration(String)` |
| Sequential model, from InputStream | `importKerasSequentialModelAndWeights(InputStream)` |
| Functional model, single .h5 file | `importKerasModelAndWeights(String)` |
| Functional model, config + weights separate | `importKerasModelAndWeights(String, String)` |
| Functional model, config only | `importKerasModelConfiguration(String)` |
| Functional model, from InputStream | `importKerasModelAndWeights(InputStream)` |
| Model missing input shape | Use the overload with `int[] inputShape` parameter |
| Model not compiled in Keras | Pass `enforceTrainingConfig=false` |

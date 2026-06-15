---
title: Keras Backend Configuration
description: DL4J Keras model import is backend-agnostic — models trained with TensorFlow, Theano, or CNTK backends can all be imported into Eclipse Deeplearning4j 1.0.0-M2.1.
---

## Keras Backend Support

DL4J Keras model import is **backend agnostic**. Regardless of which backend was used to
train the Keras model — TensorFlow, Theano, or CNTK — the model can be imported into DL4J.
The importer reads the serialised HDF5 file and interprets the layer configurations and
weights without relying on the original backend being present.

### Supported Backends

| Keras backend | Import supported | Notes                                               |
|---------------|-----------------|-----------------------------------------------------|
| TensorFlow    | Yes             | The most common backend; widest test coverage       |
| Theano        | Yes             | Supported; some weight axis orderings differ        |
| CNTK          | Yes             | Supported; limited production usage                 |

### Theano vs. TensorFlow Weight Ordering

The primary difference between backends relates to **weight axis ordering** for
convolutional layers:

- **TensorFlow** (channels-last, `NHWC`): filter shape is `(height, width, in_channels, out_channels)`
- **Theano** (channels-first, `NCHW`): filter shape is `(out_channels, in_channels, height, width)`

DL4J automatically detects the backend from the `backend` field stored in the Keras model
config and transposes convolutional weights as needed during import. You do not need to
configure this manually.

### Detecting the Backend from an HDF5 File

The backend is stored in the Keras model config JSON inside the HDF5 file:

```python
import h5py, json

with h5py.File("model.h5", "r") as f:
    config = json.loads(f.attrs["model_config"])
    backend = config.get("backend", "tensorflow")
    print("Backend:", backend)
```

### Keras Version Compatibility

DL4J model import supports both Keras 1 and Keras 2 model formats. The major version is
detected automatically from the HDF5 file metadata.

| Keras major version | Supported | Notes                                        |
|---------------------|-----------|----------------------------------------------|
| Keras 1.x           | Yes       | Legacy format; field names differ slightly   |
| Keras 2.x           | Yes       | Current recommended format                   |
| tf.keras (TF 2.x)   | Yes       | Saved with `model.save()` in HDF5 format     |

### Backend Configuration in DL4J

DL4J itself uses ND4J as its numerical backend, which can be accelerated with either
CPU (OpenBLAS/MKL) or GPU (CUDA) via its own backend system. This is independent of
the Keras training backend.

To configure ND4J backends, set the appropriate dependency in your `pom.xml`:

```xml
<!-- CPU backend -->
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-native-platform</artifactId>
  <version>1.0.0-M2.1</version>
</dependency>

<!-- GPU (CUDA) backend -->
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-cuda-11.6-platform</artifactId>
  <version>1.0.0-M2.1</version>
</dependency>
```

### Saving a Keras Model for Import

Always save Keras models in the HDF5 format using `model.save()` to ensure both the
architecture and weights are available:

```python
# Save the full model (architecture + weights + optimizer config)
model.save("model.h5")

# For inference-only import, weights-only files also work:
model.save_weights("weights.h5")
# Architecture can be saved separately:
with open("architecture.json", "w") as f:
    f.write(model.to_json())
```

### Import Examples

```java
import org.deeplearning4j.nn.modelimport.keras.KerasModelImport;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.deeplearning4j.nn.graph.ComputationGraph;

// Import a Sequential model (full model with weights)
MultiLayerNetwork seqModel = KerasModelImport
    .importKerasSequentialModelAndWeights("sequential_model.h5");

// Import a Functional API model (full model with weights)
ComputationGraph funcModel = KerasModelImport
    .importKerasFunctionalModelAndWeights("functional_model.h5");

// Import architecture only, then load weights separately
MultiLayerNetwork modelNoWeights = KerasModelImport
    .importKerasSequentialModelAndWeights("model.h5", false);
```

### Notes

- Models saved in the TensorFlow SavedModel format (a directory rather than a single `.h5`
  file) are not supported by the Keras import module. Use DL4J's TensorFlow import instead.
- Models saved with `tf.saved_model.save()` or `model.save('dir/')` in TF2 use the
  SavedModel format by default. Pass `save_format='h5'` to force HDF5 output.
- The `keras.json` backend configuration file on the training machine does not need to be
  present on the DL4J inference machine.

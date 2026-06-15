---
title: "Keras Import Overview"
description: "Importing Keras models into Deeplearning4j — supported features, limitations, and getting started"
---

## Keras Model Import

[Keras model import](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras)
provides routines for importing neural network models originally configured and trained
using [Keras](https://keras.io/), a popular Python deep learning library.

Once you have imported your model into DL4J, the full production stack is available. DL4J supports import of all Keras model types, most layers, and practically all utility functionality. See [Supported Features](./supported-features) for a complete list.

---

## Quick Start

The following example creates a simple MLP in Keras, saves it to HDF5, and loads it as a DL4J `MultiLayerNetwork` in three lines of Java.

**Python (save the model)**

```python
from keras.models import Sequential
from keras.layers import Dense

model = Sequential()
model.add(Dense(units=64, activation='relu', input_dim=100))
model.add(Dense(units=10, activation='softmax'))
model.compile(loss='categorical_crossentropy', optimizer='sgd', metrics=['accuracy'])

model.save('simple_mlp.h5')
```

**Java (load and run the model)**

```java
String simpleMlp = new ClassPathResource("simple_mlp.h5").getFile().getPath();
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(simpleMlp);

INDArray input = Nd4j.create(256, 100);
INDArray output = model.output(input);
```

`KerasModelImport` is the main entry point. It handles all Keras-to-DL4J concept mapping internally; you provide the model file and get back a fully initialized DL4J network.

---

## Maven Dependency

Add the following to your `pom.xml`:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-modelimport</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

Replace `${dl4j.version}` with the version matching the rest of your DL4J dependencies.

---

## Model Types

Keras has two model construction APIs, and DL4J maps each to an equivalent:

| Keras model type | Keras class | DL4J equivalent |
|---|---|---|
| Sequential | `keras.models.Sequential` | `MultiLayerNetwork` |
| Functional API | `keras.models.Model` | `ComputationGraph` |

- **Sequential models** are linear stacks of layers with exactly one input and one output. They map cleanly to `MultiLayerNetwork`.
- **Functional API models** can have arbitrary graph topologies: multiple inputs, multiple outputs, shared layers, and skip connections. They map to `ComputationGraph`.

---

## Saving Formats

Keras can serialize a model in several ways. Each is supported for import:

| What is saved | Python call | DL4J import method |
|---|---|---|
| Full model (config + weights + training config) | `model.save('model.h5')` | `importKerasSequentialModelAndWeights(path)` or `importKerasModelAndWeights(path)` |
| Config only (JSON) | `model.to_json()` | `importKerasSequentialConfiguration(jsonPath)` or `importKerasModelConfiguration(jsonPath)` |
| Weights only | `model.save_weights('weights.h5')` | Pass both JSON and weights paths to the two-argument import methods |

If you only save the configuration without a training configuration (i.e., the model was not compiled in Keras), set `enforceTrainingConfig=false` when calling the import method. Otherwise an exception is thrown when DL4J tries to read a training config that does not exist.

---

## Popular Supported Models

A growing set of well-known architectures has been validated end-to-end. These include:

- ResNet50
- SqueezeNet
- MobileNet
- Inception
- Xception
- UNET
- Deep convolutional GANs
- Wasserstein GANs

The full end-to-end test list is maintained in [KerasModelEndToEndTest.java](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/test/java/org/deeplearning4j/nn/modelimport/keras/e2e/KerasModelEndToEndTest.java).

---

## Supported Features Summary

| Category | Coverage |
|---|---|
| Core layers | Dense, Dropout, Flatten, Reshape, Merge, Permute, RepeatVector, Lambda, Masking, SpatialDropout1D/2D/3D |
| Convolutional layers | Conv1D, Conv2D, Conv3D, SeparableConv2D, Conv2DTranspose, Cropping1/2/3D, UpSampling1/2/3D, ZeroPadding1/2/3D, AtrousConv1D/2D |
| Pooling layers | MaxPooling1/2/3D, AveragePooling1/2/3D, GlobalMaxPooling1/2/3D, GlobalAveragePooling1/2/3D |
| Recurrent layers | SimpleRNN, LSTM (GRU and ConvLSTM2D not supported) |
| Embedding layers | Embedding |
| Normalization layers | BatchNormalization |
| Advanced activations | LeakyReLU, PReLU, ELU, ThresholdedReLU |
| Noise layers | GaussianNoise, GaussianDropout, AlphaDropout |
| Wrappers | Bidirectional (TimeDistributed not supported) |
| Losses | 13 of 14 standard Keras losses (logcosh excluded) |
| Activations | softmax, elu, selu, softplus, softsign, relu, tanh, sigmoid, hard_sigmoid, linear |
| Initializers | All 15 standard Keras initializers |
| Regularizers | l1, l2, l1_l2 |
| Constraints | max_norm, non_neg, unit_norm, min_max_norm |
| Optimizers | SGD, RMSprop, Adagrad, Adadelta, Adam, Adamax, Nadam (TFOptimizer excluded) |

---

## After Import: Saving the DL4J Model

Once your Keras model is imported, save it using DL4J's `ModelSerializer` for subsequent loads without the overhead of re-importing from Keras:

```java
File locationToSave = new File("my-model.zip");
boolean saveUpdater = true;
ModelSerializer.writeModel(model, locationToSave, saveUpdater);

// Later:
MultiLayerNetwork restored = ModelSerializer.restoreMultiLayerNetwork(locationToSave);
```

---

## Troubleshooting

**`IncompatibleKerasConfigurationException`**: the model uses a feature not supported by the importer. Check [Supported Features](./supported-features) to confirm the layer or configuration in question.

**`UnsupportedKerasConfigurationException`**: a recognized feature has parameters that DL4J cannot map. Common causes include unknown regularizer types or unsupported layer parameter combinations.

**Weights not loaded correctly**: verify that you are using a full model save (not weights-only) or that you are providing both the JSON config and weights paths to the split-file import method.

For additional help, visit the [DL4J community forums](https://community.konduit.ai) or open an issue on [GitHub](https://github.com/eclipse/deeplearning4j/issues).

---

## Why Keras Model Import?

Keras is a user-friendly deep learning library written in Python. Its intuitive API makes rapid prototyping easy. However, Python-trained models often need to be deployed in Java or JVM-based production environments.

Keras model import bridges this gap. Data scientists write and train models in Python; the import module brings them into the DL4J production stack without requiring Python at runtime. The imported model can be:

- Used for inference directly
- Fine-tuned with additional DL4J training
- Integrated with DL4J's distributed training (Spark)
- Deployed via Konduit Serving

---

## Next Steps

- [Getting Started](./getting-started) — step-by-step import walkthrough with full code examples
- [Sequential Model](./sequential-model) — importing `Sequential` models as `MultiLayerNetwork`
- [Functional Model](./functional-model) — importing Functional API models as `ComputationGraph`
- [API Reference](./model-import-api) — complete `KerasModelImport` method signatures
- [Supported Features](./supported-features) — full support matrix

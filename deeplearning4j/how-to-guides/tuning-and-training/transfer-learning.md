---
title: "Transfer Learning"
description: "Transfer learning in Deeplearning4j — TransferLearning.Builder, FineTuneConfiguration, freezing layers, and modifying pretrained networks"
---

## Overview

Transfer learning reuses a pretrained model as a starting point for a new task, rather than training from scratch. DL4J's transfer learning API lets you:

- Freeze layers (hold parameters constant during training)
- Modify the number of outputs of an existing layer (`nOutReplace`)
- Remove layers from an existing model
- Add new layers
- Override the learning configuration (learning rate, updater, regularization) for unfrozen layers

The API supports both `MultiLayerNetwork` and `ComputationGraph` via `TransferLearning.Builder` and `TransferLearning.GraphBuilder` respectively.

---

## Core Classes

| Class | Description |
|-------|-------------|
| `TransferLearning.Builder` | Modifies a `MultiLayerNetwork` |
| `TransferLearning.GraphBuilder` | Modifies a `ComputationGraph` |
| `FineTuneConfiguration` | Learning hyperparameters applied to all unfrozen layers |
| `TransferLearningHelper` | Pre-computes and caches activations at the freeze boundary to speed up training |

---

## FineTuneConfiguration

`FineTuneConfiguration` specifies the training hyperparameters that will be applied to all unfrozen (trainable) layers. Values set here override what was in the original model's configuration.

```java
import org.deeplearning4j.nn.transferlearning.FineTuneConfiguration;
import org.nd4j.linalg.learning.config.Adam;
import org.nd4j.linalg.learning.config.Nesterovs;

FineTuneConfiguration fineTuneConf = new FineTuneConfiguration.Builder()
    .updater(new Adam(1e-4))      // use Adam with a small learning rate for fine-tuning
    .seed(42)
    .l2(1e-5)
    .build();
```

Key options:

| Method | Description |
|--------|-------------|
| `.updater(IUpdater)` | Optimizer for unfrozen layers (e.g., `new Adam(lr)`, `new Nesterovs(lr, momentum)`) |
| `.seed(long)` | Random seed |
| `.l1(double)` / `.l2(double)` | Regularization for unfrozen layers |
| `.activation(Activation)` | Override activation for all unfrozen layers |
| `.dropOut(double)` | Dropout retain probability |

**Note:** Newly added layers can specify their own learning rate or updater in their layer builder, which takes priority over `FineTuneConfiguration`.

---

## TransferLearning.Builder (for MultiLayerNetwork)

### Basic Usage

```java
import org.deeplearning4j.nn.transferlearning.TransferLearning;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;

// Load or build a pretrained MultiLayerNetwork
MultiLayerNetwork pretrainedNet = ModelSerializer.restoreMultiLayerNetwork(new File("pretrained.zip"));

// Build a new model with frozen layers and a replaced output layer
MultiLayerNetwork transferModel = new TransferLearning.Builder(pretrainedNet)
    .fineTuneConfiguration(fineTuneConf)
    .setFeatureExtractor(3)          // freeze layers 0–3 (inclusive)
    .removeOutputLayer()             // remove the last layer
    .addLayer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nIn(256).nOut(5)            // new task: 5 classes instead of 1000
        .activation(Activation.SOFTMAX)
        .weightInit(WeightInit.XAVIER)
        .build())
    .build();
```

`TransferLearning.Builder` returns a **new** `MultiLayerNetwork`. The original `pretrainedNet` is not modified.

### Key Methods

| Method | Description |
|--------|-------------|
| `fineTuneConfiguration(FineTuneConfiguration)` | Set learning hyperparameters for unfrozen layers |
| `setFeatureExtractor(int layerNum)` | Freeze all layers from 0 up to and including `layerNum` |
| `nOutReplace(int layerNum, int nOut, WeightInit scheme)` | Change nOut of a layer and reinitialize affected weights |
| `nInReplace(int layerNum, int nIn, WeightInit scheme)` | Change nIn of a layer |
| `removeOutputLayer()` | Remove the last layer (convenience for replacing the output head) |
| `removeLayersFromOutput(int n)` | Remove the last `n` layers |
| `addLayer(Layer layer)` | Append a layer (call multiple times to add a stack) |
| `setInputPreProcessor(int layer, InputPreProcessor)` | Manually add a pre-processor |

---

## TransferLearning.GraphBuilder (for ComputationGraph)

```java
import org.deeplearning4j.nn.transferlearning.TransferLearning;
import org.deeplearning4j.nn.graph.ComputationGraph;

ComputationGraph pretrainedNet = ModelSerializer.restoreComputationGraph(new File("vgg16.zip"));

ComputationGraph transferModel = new TransferLearning.GraphBuilder(pretrainedNet)
    .fineTuneConfiguration(fineTuneConf)
    .setFeatureExtractor("fc2")                  // freeze up to and including layer named "fc2"
    .removeVertexKeepConnections("predictions")   // remove output vertex, keep its connections
    .addLayer("predictions",
        new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
            .nIn(4096).nOut(5)
            .activation(Activation.SOFTMAX)
            .weightInit(WeightInit.XAVIER)
            .build(),
        "fc2")                                   // connect new layer to "fc2"
    .build();
```

### Key Methods

| Method | Description |
|--------|-------------|
| `fineTuneConfiguration(FineTuneConfiguration)` | Learning config for unfrozen vertices |
| `setFeatureExtractor(String vertexName)` | Freeze all vertices up to and including `vertexName` |
| `nOutReplace(String vertexName, int nOut, WeightInit scheme)` | Change nOut of a named vertex |
| `removeVertexKeepConnections(String vertexName)` | Remove vertex, preserve its connections in the graph |
| `removeVertexAndConnections(String vertexName)` | Remove vertex and all its connections |
| `addLayer(String name, Layer layer, String... inputs)` | Add a new layer vertex |
| `addVertex(String name, GraphVertex vertex, String... inputs)` | Add a non-layer vertex |
| `setOutputs(String...)` | Declare new output vertices |

---

## Common Transfer Learning Patterns

### Pattern 1: Replace the Classification Head Only

The most common pattern: keep the feature extractor frozen, replace only the final output layer.

```java
// Assuming VGG-style network with layers: conv_block1 ... fc1, fc2, predictions
ComputationGraph model = new TransferLearning.GraphBuilder(pretrainedNet)
    .fineTuneConfiguration(new FineTuneConfiguration.Builder()
        .updater(new Nesterovs(5e-5, 0.9))
        .seed(42)
        .build())
    .setFeatureExtractor("fc2")                    // freeze everything up to fc2
    .removeVertexKeepConnections("predictions")     // remove old 1000-class output
    .addLayer("predictions",
        new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
            .nIn(4096).nOut(numClasses)
            .activation(Activation.SOFTMAX)
            .weightInit(WeightInit.XAVIER)
            .build(),
        "fc2")
    .build();
```

After a small number of training iterations (e.g., 30-100), this pattern typically achieves high accuracy because the frozen feature extractor already captures rich visual representations.

### Pattern 2: Modify an Intermediate Layer Width and Add New Layers

```java
ComputationGraph model = new TransferLearning.GraphBuilder(pretrainedNet)
    .fineTuneConfiguration(fineTuneConf)
    .setFeatureExtractor("block5_pool")            // freeze through conv blocks
    .nOutReplace("fc2", 1024, WeightInit.XAVIER)   // resize fc2 from 4096 to 1024
    .removeVertexAndConnections("predictions")
    .addLayer("fc3",
        new DenseLayer.Builder()
            .nIn(1024).nOut(256)
            .activation(Activation.RELU)
            .build(),
        "fc2")
    .addLayer("predictions",
        new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
            .nIn(256).nOut(numClasses)
            .activation(Activation.SOFTMAX)
            .build(),
        "fc3")
    .setOutputs("predictions")
    .build();
```

**Important:** `nOutReplace` automatically adjusts the `nIn` of all downstream layers that receive input from the modified layer. You do not need to update them manually.

### Pattern 3: Progressive Unfreezing

Start with more layers frozen, then unfreeze progressively:

```java
// Phase 1: Freeze up to block5_pool, train only the head
ComputationGraph phase1 = new TransferLearning.GraphBuilder(pretrainedNet)
    .fineTuneConfiguration(fineTuneConf)
    .setFeatureExtractor("block5_pool")
    .removeVertexKeepConnections("predictions")
    .addLayer("predictions", newOutputLayer, "fc2")
    .build();

// Train phase1 for some epochs...
phase1.fit(trainIter, 5);

// Phase 2: Unfreeze block5 layers and continue training with a lower LR
ComputationGraph phase2 = new TransferLearning.GraphBuilder(phase1)
    .fineTuneConfiguration(new FineTuneConfiguration.Builder()
        .updater(new Adam(1e-5))  // very small LR to avoid destroying pretrained features
        .build())
    .setFeatureExtractor("block4_pool")   // now freeze only up to block4
    .build();

phase2.fit(trainIter, 5);
```

### Pattern 4: MultiLayerNetwork Transfer Learning

```java
MultiLayerNetwork pretrainedMln = ModelSerializer.restoreMultiLayerNetwork(new File("pretrained.zip"));

MultiLayerNetwork transferMln = new TransferLearning.Builder(pretrainedMln)
    .fineTuneConfiguration(fineTuneConf)
    .setFeatureExtractor(5)               // freeze layers 0–5
    .removeLayersFromOutput(1)            // remove last 1 layer (the output layer)
    .addLayer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nIn(512).nOut(numClasses)
        .activation(Activation.SOFTMAX)
        .weightInit(WeightInit.XAVIER)
        .build())
    .build();
```

---

## TransferLearningHelper

`TransferLearningHelper` speeds up transfer learning when you have a large frozen section. Instead of running the full forward pass through frozen layers at every training step, it pre-computes and caches ("featurizes") the activations at the freeze boundary.

This can dramatically reduce training time when the frozen section is computationally expensive (e.g., deep VGG convolutional blocks).

### Featurizing a Dataset

```java
import org.deeplearning4j.nn.transferlearning.TransferLearningHelper;

// Freeze pretrainedNet at "fc2" and create the helper
TransferLearningHelper helper = new TransferLearningHelper(pretrainedNet, "fc2");

// Featurize the training data (runs forward pass through frozen layers once)
List<DataSet> featurizedData = new ArrayList<>();
while (trainIter.hasNext()) {
    DataSet featurized = helper.featurize(trainIter.next());
    featurizedData.add(featurized);
    // Optionally: save to disk to avoid re-running on each training run
}
```

### Training on Featurized Data

```java
// Build transfer model (head only) — helper already froze pretrainedNet in-place
TransferLearningHelper headHelper = new TransferLearningHelper(vgg16Transfer);

for (int epoch = 0; epoch < numEpochs; epoch++) {
    for (DataSet batch : featurizedData) {
        headHelper.fitFeaturized(batch);
    }
}
```

The helper modifies the model **in place**. The parameters of the unfrozen portion of `pretrainedNet` are updated with each call to `fitFeaturized`.

### Helper Methods

| Method | Description |
|--------|-------------|
| `featurize(DataSet)` | Returns a `DataSet` where inputs are frozen-layer activations |
| `featurize(MultiDataSet)` | Multi-input/output version |
| `fitFeaturized(DataSetIterator)` | Train the unfrozen head on featurized data |
| `fitFeaturized(MultiDataSetIterator)` | Multi-dataset version |
| `outputFromFeaturized(INDArray)` | Inference from featurized (post-freeze-boundary) input |
| `unfrozenMLN()` | Returns the unfrozen portion as a standalone `MultiLayerNetwork` |
| `unfrozenGraph()` | Returns the unfrozen portion as a standalone `ComputationGraph` |

---

## Using Zoo Models as Starting Points

DL4J's model zoo provides pretrained weights for common architectures:

```java
import org.deeplearning4j.zoo.ZooModel;
import org.deeplearning4j.zoo.model.VGG16;
import org.deeplearning4j.zoo.PretrainedType;

// Download and initialize VGG16 with ImageNet weights
ZooModel zooModel = VGG16.builder().build();
ComputationGraph pretrainedVGG16 = (ComputationGraph) zooModel.initPretrained(PretrainedType.IMAGENET);

// Now apply transfer learning
ComputationGraph myModel = new TransferLearning.GraphBuilder(pretrainedVGG16)
    .fineTuneConfiguration(new FineTuneConfiguration.Builder()
        .updater(new Adam(5e-5))
        .seed(42)
        .build())
    .setFeatureExtractor("fc2")
    .removeVertexKeepConnections("predictions")
    .addLayer("predictions",
        new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
            .nIn(4096).nOut(numClasses)
            .activation(Activation.SOFTMAX)
            .build(),
        "fc2")
    .build();

System.out.println(myModel.summary());
```

---

## Important Notes

### Frozen Layers Are Not Saved

When you serialize a model with frozen layers using `ModelSerializer`, the frozen state is **not** preserved. When you reload the model, no layers will be frozen. To continue training with frozen layers after reloading, you must re-apply the `TransferLearning` API or `TransferLearningHelper`.

```java
// After reloading, re-freeze as needed:
MultiLayerNetwork reloaded = ModelSerializer.restoreMultiLayerNetwork(modelFile);
MultiLayerNetwork frozenAgain = new TransferLearning.Builder(reloaded)
    .fineTuneConfiguration(fineTuneConf)
    .setFeatureExtractor(layerNumToFreeze)
    .build();
```

### TransferLearning Returns a New Model

`TransferLearning.Builder.build()` always returns a **new** model instance. The original pretrained model is not modified. Keep memory constraints in mind when working with large models.

### Changing nOut Cascades to nIn of Downstream Layers

When you call `nOutReplace("fc2", 1024, WeightInit.XAVIER)`, DL4J automatically updates the `nIn` of every layer that directly receives input from `fc2`. You do not need to manually specify `nIn` on those layers. You can optionally specify separate weight init schemes for the modified layer and its downstream consumers:

```java
.nOutReplace("fc2", 1024, WeightInit.XAVIER, WeightInit.XAVIER)
//           layer   nOut  scheme-for-fc2   scheme-for-next-layers
```

### FineTuneConfiguration Selectively Updates

`FineTuneConfiguration` only updates the learning parameters of unfrozen layers. Frozen layers retain their original configuration. Newly added layers inherit `FineTuneConfiguration` unless they specify their own updater/LR in their layer builder.

---

## Saving the Transfer-Learned Model

```java
import org.deeplearning4j.util.ModelSerializer;

// Save including updater state (for continued training)
ModelSerializer.writeModel(myModel, new File("myTransferModel.zip"), true);

// Load
ComputationGraph loaded = ModelSerializer.restoreComputationGraph(new File("myTransferModel.zip"));
```

---
title: "ComputationGraph"
description: "The ComputationGraph API — multi-input, multi-output, skip connections, and arbitrary DAG topologies"
---

## Overview

`ComputationGraph` is DL4J's general-purpose network class. Unlike `MultiLayerNetwork`, which connects layers in a fixed chain, `ComputationGraph` supports arbitrary directed acyclic graph (DAG) topologies:

- Multiple network inputs
- Multiple network outputs (mixed classification and regression)
- Skip connections and residual connections
- Branching and merging of activation streams
- Siamese, multi-task, and encoder-decoder architectures

Everything `MultiLayerNetwork` can do, `ComputationGraph` can also do — though configuration is slightly more verbose.

---

## When to Use ComputationGraph vs. MultiLayerNetwork

| Requirement | MultiLayerNetwork | ComputationGraph |
|-------------|:-----------------:|:----------------:|
| Single input, single output | Yes | Yes |
| Multiple inputs | No | Yes |
| Multiple outputs | No | Yes |
| Skip / residual connections | No | Yes |
| Complex loss combinations | No | Yes |
| Siamese / shared-weight sub-networks | No | Yes |
| Simpler configuration | Yes (preferred) | More verbose |

---

## Graph Building API

Configuration starts the same way as `MultiLayerNetwork` — with `NeuralNetConfiguration.Builder` — but instead of calling `.list()` you call `.graphBuilder()`.

```java
import org.deeplearning4j.nn.conf.ComputationGraphConfiguration;
import org.deeplearning4j.nn.conf.NeuralNetConfiguration;
import org.deeplearning4j.nn.graph.ComputationGraph;
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.learning.config.Adam;

ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .weightInit(WeightInit.XAVIER)
    .graphBuilder()
    // --- define inputs, layers, vertices, outputs here ---
    .addInputs("input")
    .addLayer("dense1",
        new DenseLayer.Builder().nIn(784).nOut(256).activation(Activation.RELU).build(),
        "input")
    .addLayer("out",
        new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
            .nIn(256).nOut(10).activation(Activation.SOFTMAX).build(),
        "dense1")
    .setOutputs("out")
    .build();

ComputationGraph model = new ComputationGraph(conf);
model.init();
```

### addInputs(String...)

Declares one or more named network inputs. The order determines which position in the `INDArray[]` passed to `fit()` and `output()` corresponds to which input.

```java
.addInputs("imageInput", "metadataInput")
```

### addLayer(String, Layer, String...)

Adds a layer vertex to the graph.

- First argument: unique name for this layer.
- Second argument: layer configuration.
- Remaining arguments: names of this layer's inputs (other layers or declared inputs).

```java
.addLayer("conv1",
    new ConvolutionLayer.Builder(3, 3).nIn(3).nOut(32).activation(Activation.RELU).build(),
    "imageInput")
```

### addVertex(String, GraphVertex, String...)

Adds a non-layer vertex (merge, element-wise op, subset, etc.):

```java
.addVertex("merge", new MergeVertex(), "branch1", "branch2")
.addVertex("add",   new ElementWiseVertex(ElementWiseVertex.Op.Add), "L1", "shortcut")
```

### setOutputs(String...)

Declares which vertices produce the network's outputs. The order determines the position of output arrays in `output()` return values and in `MultiDataSet` label arrays.

```java
.setOutputs("classOutput", "regressionOutput")
```

### setInputTypes(InputType...)

Enables automatic `nIn` inference and automatic insertion of pre-processors between mismatched layer types (e.g., CNN -> Dense):

```java
.setInputTypes(InputType.convolutional(32, 32, 3), InputType.feedForward(16))
```

---

## Types of Graph Vertices

### LayerVertex

Standard neural network layer. Added via `addLayer()`. Supports all layer types available in `MultiLayerNetwork`.

### InputVertex

Created automatically when you call `addInputs()`. One InputVertex per named input.

### MergeVertex

Concatenates activations from two or more inputs along the feature dimension. Use this to combine branches.

```java
// L1 outputs 64 features, L2 outputs 64 features -> merge outputs 128 features
.addVertex("merged", new MergeVertex(), "L1", "L2")
```

For CNN activations, merging happens along the channel dimension. For RNN activations, along the feature dimension.

### ElementWiseVertex

Applies an element-wise operation to inputs of identical shape:

```java
// Residual / skip connection: add input directly to layer output
.addVertex("residual", new ElementWiseVertex(ElementWiseVertex.Op.Add), "blockOut", "shortcut")
```

Supported ops: `Add`, `Subtract`, `Product`, `Average`, `Max`.

### SubsetVertex

Extracts a range of features from a vertex's output:

```java
// Take features 0..63 from a 128-feature layer
.addVertex("first64", new SubsetVertex(0, 63), "sharedLayer")
```

### PreProcessorVertex

Applies an `InputPreProcessor` as a standalone graph node (without attaching it to a layer):

```java
.addVertex("reshape", new PreprocessorVertex(new CnnToFeedForwardPreProcessor(7, 7, 512)), "convOut")
```

---

## Example 1: Recurrent Network with Skip Connection

```java
ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .graphBuilder()
    .addInputs("input")
    // LSTM layer reading from input
    .addLayer("lstm1",
        new LSTM.Builder().nIn(32).nOut(64).activation(Activation.TANH).build(),
        "input")
    // Output layer receives both the LSTM output AND the raw input (skip connection)
    .addLayer("out",
        new RnnOutputLayer.Builder(LossFunctions.LossFunction.MCXENT)
            .nIn(64 + 32).nOut(10).activation(Activation.SOFTMAX).build(),
        "lstm1", "input")   // <- two inputs: lstm output + raw input
    .setOutputs("out")
    .build();

ComputationGraph model = new ComputationGraph(conf);
model.init();
```

The `nIn` on the output layer is `64 + 32 = 96` because `MergeVertex`-style concatenation happens implicitly when a layer lists multiple inputs and no explicit vertex is added.

---

## Example 2: Multiple Inputs with a Merge Vertex

Two separate input streams (e.g., image features and tabular features) are processed independently and then merged.

```java
ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .graphBuilder()
    .addInputs("imgFeatures", "tabFeatures")
    // Process image branch
    .addLayer("imgDense",
        new DenseLayer.Builder().nIn(512).nOut(128).activation(Activation.RELU).build(),
        "imgFeatures")
    // Process tabular branch
    .addLayer("tabDense",
        new DenseLayer.Builder().nIn(30).nOut(32).activation(Activation.RELU).build(),
        "tabFeatures")
    // Merge both branches: 128 + 32 = 160 features
    .addVertex("merge", new MergeVertex(), "imgDense", "tabDense")
    // Final classification head
    .addLayer("out",
        new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
            .nIn(160).nOut(5).activation(Activation.SOFTMAX).build(),
        "merge")
    .setOutputs("out")
    .build();
```

---

## Example 3: Multi-Task Learning

One shared trunk feeds two independent output heads — one for classification and one for regression.

```java
ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .graphBuilder()
    .addInputs("input")
    // Shared representation layers
    .addLayer("shared1",
        new DenseLayer.Builder().nIn(256).nOut(128).activation(Activation.RELU).build(),
        "input")
    .addLayer("shared2",
        new DenseLayer.Builder().nIn(128).nOut(64).activation(Activation.RELU).build(),
        "shared1")
    // Classification head
    .addLayer("classHead",
        new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
            .nIn(64).nOut(10).activation(Activation.SOFTMAX).build(),
        "shared2")
    // Regression head
    .addLayer("regHead",
        new OutputLayer.Builder(LossFunctions.LossFunction.MSE)
            .nIn(64).nOut(1).activation(Activation.IDENTITY).build(),
        "shared2")
    .setOutputs("classHead", "regHead")
    .build();

ComputationGraph model = new ComputationGraph(conf);
model.init();
```

Training with `MultiDataSet` (required for multiple outputs):

```java
// MultiDataSet: inputs[] and labels[] arrays match .addInputs() / .setOutputs() order
model.fit(multiDataSetIterator);
```

---

## Example 4: Residual Block

A residual (skip) connection adds the layer input directly to the layer output via `ElementWiseVertex`:

```java
ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .graphBuilder()
    .addInputs("input")
    .addLayer("conv1",
        new ConvolutionLayer.Builder(3, 3).nIn(64).nOut(64)
            .padding(1, 1).activation(Activation.RELU).build(),
        "input")
    .addLayer("conv2",
        new ConvolutionLayer.Builder(3, 3).nIn(64).nOut(64)
            .padding(1, 1).activation(Activation.IDENTITY).build(),
        "conv1")
    // Add conv2 output + original input (residual connection)
    .addVertex("residual",
        new ElementWiseVertex(ElementWiseVertex.Op.Add),
        "conv2", "input")
    .addLayer("relu",
        new ActivationLayer.Builder().activation(Activation.RELU).build(),
        "residual")
    .addLayer("out",
        new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
            .nIn(64 * 7 * 7).nOut(10).activation(Activation.SOFTMAX).build(),
        "relu")
    .setOutputs("out")
    .setInputTypes(InputType.convolutional(28, 28, 64))
    .build();
```

---

## Example 5: Siamese Network

A Siamese network uses two identical subnetworks (shared weights) to compare two inputs. In DL4J this is done with `ComputationGraph` by routing both inputs through the same named layer — but note that DL4J does not natively support weight sharing between separate named layers. The typical approach is to use two separate layer definitions with identical configurations, or to compute the representations separately in preprocessing and use a single-input graph for the comparison head.

For a simplified Siamese distance-based network where two feature vectors are compared:

```java
ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .graphBuilder()
    .addInputs("inputA", "inputB")
    // Two encoder branches (same architecture, separate params)
    .addLayer("encA",
        new DenseLayer.Builder().nIn(128).nOut(64).activation(Activation.RELU).build(),
        "inputA")
    .addLayer("encB",
        new DenseLayer.Builder().nIn(128).nOut(64).activation(Activation.RELU).build(),
        "inputB")
    // Merge and compare
    .addVertex("merged", new MergeVertex(), "encA", "encB")
    .addLayer("out",
        new OutputLayer.Builder(LossFunctions.LossFunction.XENT)
            .nIn(128).nOut(1).activation(Activation.SIGMOID).build(),
        "merged")
    .setOutputs("out")
    .build();
```

---

## Training Data

### DataSet / DataSetIterator

Use when the graph has a single input and single output. Same as `MultiLayerNetwork`.

```java
model.fit(dataSetIterator);
```

### MultiDataSet / MultiDataSetIterator

Required for multiple inputs or multiple outputs.

```java
// Manual MultiDataSet construction
INDArray[] inputs = new INDArray[]{ inputA, inputB };
INDArray[] labels = new INDArray[]{ classLabels, regLabels };
MultiDataSet mds = new org.nd4j.linalg.dataset.MultiDataSet(inputs, labels);
model.fit(mds);
```

### RecordReaderMultiDataSetIterator

```java
int batchSize = 32;
MultiDataSetIterator iter = new RecordReaderMultiDataSetIterator.Builder(batchSize)
    .addReader("features", featureReader)
    .addReader("labels",   labelReader)
    .addInput("features", 0, 9)          // columns 0–9 as input 0
    .addOutput("labels", 0, 4)           // columns 0–4 as output 0
    .addOutputOneHot("labels", 5, 10)    // column 5 -> one-hot, 10 classes
    .build();
```

---

## Inference

```java
// Single input, single output
INDArray out = model.outputSingle(input);

// Multiple inputs
INDArray[] outputs = model.output(inputA, inputB);

// Multiple inputs via array
INDArray[] outs = model.output(false, inputs);  // false = test mode
```

---

## Evaluation

```java
// Classification (single output)
Evaluation eval = model.evaluate(testIter);
System.out.println(eval.stats());

// Multi-output: evaluate a specific output by index
Evaluation eval0 = model.evaluate(testIter, Collections.singletonList("classHead"));
```

---

## Saving and Loading

```java
import org.deeplearning4j.util.ModelSerializer;

// Save
ModelSerializer.writeModel(model, new File("cgModel.zip"), true);

// Load
ComputationGraph loaded = ModelSerializer.restoreComputationGraph(new File("cgModel.zip"));
```

---

## Key API Reference

| Method | Description |
|--------|-------------|
| `graphBuilder()` | Returns a `ComputationGraphConfiguration.GraphBuilder` |
| `addInputs(String...)` | Declare network input names |
| `addLayer(String, Layer, String...)` | Add a layer vertex with named inputs |
| `addVertex(String, GraphVertex, String...)` | Add a non-layer vertex |
| `setOutputs(String...)` | Declare which vertices are outputs |
| `setInputTypes(InputType...)` | Enable automatic shape inference |
| `init()` | Initialize parameters |
| `fit(DataSetIterator)` | Train (single input/output) |
| `fit(MultiDataSetIterator)` | Train (multiple inputs/outputs) |
| `output(INDArray...)` | Run forward pass, return output arrays |
| `outputSingle(INDArray)` | Convenience method: single input/output |
| `evaluate(DataSetIterator)` | Returns `Evaluation` for single-output graph |
| `summary()` | Print graph topology and parameter counts |

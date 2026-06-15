---
title: "Graph Vertices"
description: "Vertex types for ComputationGraph — Merge, ElementWise, Subset, Stack, Reshape, and custom vertices"
---

# Graph Vertices

In Eclipse Deeplearning4j a **vertex** is a node in a `ComputationGraph` that can accept multiple inputs and produce one or more outputs. Vertices enable complex topologies that `MultiLayerNetwork` cannot express: multi-input merging, inception modules, siamese networks, highway layers, and more.

Vertices are added to a `ComputationGraphConfiguration` using `addVertex(String name, GraphVertex vertex, String... inputs)`.

---

## MergeVertex

Concatenates two or more input activations along the feature dimension (axis 1 for 2D, axis 1 for 4D CNN feature maps). The output size equals the sum of the input sizes.

**Common use:** Combining outputs from parallel branches (e.g., inception modules).

```java
import org.deeplearning4j.nn.graph.vertex.impl.MergeVertex;

ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .graphBuilder()
    .addInputs("input")
    // Branch 1: 1x1 convolution
    .addLayer("branch1", new ConvolutionLayer.Builder(1, 1)
        .nIn(64).nOut(32).build(), "input")
    // Branch 2: 3x3 convolution
    .addLayer("branch2", new ConvolutionLayer.Builder(3, 3)
        .nIn(64).nOut(32).stride(1,1).convolutionMode(ConvolutionMode.Same).build(), "input")
    // Concatenate both branches: output has 64 channels
    .addVertex("merged", new MergeVertex(), "branch1", "branch2")
    .addLayer("output", new CnnLossLayer.Builder().build(), "merged")
    .setOutputs("output")
    .build();
```

---

## ElementWiseVertex

Applies an element-wise operation across two or more inputs of the same shape. All inputs must have identical dimensions.

### Operations

| Operation constant | Behaviour |
|---|---|
| `ElementWiseVertex.Op.Add` | Element-wise sum of all inputs |
| `ElementWiseVertex.Op.Subtract` | Element-wise difference (input0 - input1) |
| `ElementWiseVertex.Op.Product` | Element-wise product (Hadamard) |
| `ElementWiseVertex.Op.Average` | Element-wise mean of all inputs |
| `ElementWiseVertex.Op.Max` | Element-wise maximum across all inputs |

**Common use:** Residual connections (Add), gating mechanisms (Product).

```java
import org.deeplearning4j.nn.graph.vertex.impl.ElementWiseVertex;

// Residual / skip connection: add input to transformed output
conf.addVertex("residual",
    new ElementWiseVertex(ElementWiseVertex.Op.Add),
    "inputLayer", "transformLayer");

// Attention gate: element-wise product of gate and values
conf.addVertex("gated",
    new ElementWiseVertex(ElementWiseVertex.Op.Product),
    "gate", "values");
```

---

## SubsetVertex

Selects a contiguous range of columns (features) from a 2D input (shape `[batch, features]`). Useful for splitting the output of a layer into separate streams.

```java
import org.deeplearning4j.nn.graph.vertex.impl.SubsetVertex;

// Split a 256-unit dense layer output into two 128-unit streams
conf.addLayer("dense", new DenseLayer.Builder().nIn(128).nOut(256).build(), "input");
conf.addVertex("stream1",
    new SubsetVertex(0, 127),   // columns 0 to 127 inclusive
    "dense");
conf.addVertex("stream2",
    new SubsetVertex(128, 255), // columns 128 to 255 inclusive
    "dense");
```

---

## StackVertex and UnstackVertex

These vertices work as a pair to allow shared-weight processing across multiple inputs.

### StackVertex

Stacks multiple inputs along dimension 0 (the batch dimension), producing a single output with a larger batch size. This enables a single shared layer to process multiple inputs without duplicating weights.

**Common use:** Siamese networks, triplet embedding where the same encoder processes anchor, positive, and negative inputs.

```java
import org.deeplearning4j.nn.graph.vertex.impl.StackVertex;
import org.deeplearning4j.nn.graph.vertex.impl.UnstackVertex;

ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .graphBuilder()
    .addInputs("anchor", "positive", "negative")
    // Stack all three inputs into one larger batch
    .addVertex("stacked", new StackVertex(), "anchor", "positive", "negative")
    // Shared encoder processes all three in one forward pass
    .addLayer("encoder", new DenseLayer.Builder().nIn(128).nOut(64).build(), "stacked")
    // Unstack back into three separate streams (stackSize = 3)
    .addVertex("anchorOut",   new UnstackVertex(0, 3), "encoder")
    .addVertex("positiveOut", new UnstackVertex(1, 3), "encoder")
    .addVertex("negativeOut", new UnstackVertex(2, 3), "encoder")
    // ... compute triplet loss
    .setOutputs("anchorOut", "positiveOut", "negativeOut")
    .build();
```

### UnstackVertex

Reverses a `StackVertex` by extracting a single slice from dimension 0. Parameters:

- `index` — which example to extract (0-based).
- `stackSize` — the total number of examples stacked (used to compute the step/stride).

```java
// Extract the second of three stacked inputs
new UnstackVertex(1, 3)
```

---

## ReshapeVertex

Reshapes the activation tensor to a new shape, enabling transitions between 2D (fully connected) and 4D (convolutional) representations within a `ComputationGraph`.

```java
import org.deeplearning4j.nn.graph.vertex.impl.ReshapeVertex;

// Flatten a [batch, channels, h, w] CNN output to [batch, features] for a dense layer
conf.addVertex("flatten",
    new ReshapeVertex('c', new int[]{-1, channels * h * w}),
    "convLayer");

// Or reshape a flat vector back to a 3D spatial tensor for a decoder
conf.addVertex("reshape",
    new ReshapeVertex('c', new int[]{-1, 64, 7, 7}),
    "denseBottleneck");
```

The first argument is the array ordering (`'c'` for C order, `'f'` for Fortran order). Use `-1` for the batch dimension (it is inferred automatically). `ReshapeVertex` validates that the reshaping is compatible during both forward and backward passes.

---

## L2NormalizeVertex

Performs L2 normalisation on its single input, so that each example's feature vector lies on the unit hypersphere. The output has the same shape as the input.

**Common use:** Metric learning, face verification, embedding spaces where cosine distance is used.

```java
import org.deeplearning4j.nn.graph.vertex.impl.L2NormalizeVertex;

conf.addVertex("l2norm",
    new L2NormalizeVertex(new int[]{1}, 1e-12),  // normalise along axis 1, epsilon 1e-12
    "embeddingLayer");
```

---

## L2Vertex

Computes the L2 (Euclidean) distance between exactly two inputs of the same shape. The output is a scalar (or batch of scalars).

**Common use:** Triplet loss networks — compute distance between anchor-positive pair and anchor-negative pair, then feed both scalars into a loss layer.

```java
import org.deeplearning4j.nn.graph.vertex.impl.L2Vertex;

conf.addVertex("distPos",
    new L2Vertex(),        // default epsilon
    "anchorEmb", "positiveEmb");
conf.addVertex("distNeg",
    new L2Vertex(),
    "anchorEmb", "negativeEmb");
// Feed distPos and distNeg into a LossLayer for triplet loss
```

---

## ScaleVertex

Multiplies the activations of a single input by a scalar constant. Gradients are scaled by the same factor during backpropagation.

**Common use:** Scaling residual branch outputs (e.g., multiplying by 0.1 in very deep networks to stabilise variance), or implementing highway networks.

```java
import org.deeplearning4j.nn.graph.vertex.impl.ScaleVertex;

// Scale activations by 0.1 to reduce variance
conf.addVertex("scaled", new ScaleVertex(0.1), "residualBranch");
```

---

## ShiftVertex

Adds a scalar constant to all activations of a single input element-wise.

**Common use:** Adding a bias offset after a layer, or computing `(1 - sigmoid(x))` in a highway network:

```java
import org.deeplearning4j.nn.graph.vertex.impl.ShiftVertex;

// Step 1: sigmoid gate
conf.addLayer("gate", new DenseLayer.Builder().activation(Activation.SIGMOID)
    .nIn(n).nOut(n).build(), "input");
// Step 2: compute (1 - gate) using Scale(-1) then Shift(+1)
conf.addVertex("negGate",   new ScaleVertex(-1.0), "gate");
conf.addVertex("oneMinusG", new ShiftVertex(1.0),  "negGate");
// Step 3: gate * transform(input) + (1 - gate) * input
conf.addLayer("transform", new DenseLayer.Builder().activation(Activation.TANH)
    .nIn(n).nOut(n).build(), "input");
conf.addVertex("gatedTransform",
    new ElementWiseVertex(ElementWiseVertex.Op.Product), "gate", "transform");
conf.addVertex("passthrough",
    new ElementWiseVertex(ElementWiseVertex.Op.Product), "oneMinusG", "input");
conf.addVertex("highway",
    new ElementWiseVertex(ElementWiseVertex.Op.Add), "gatedTransform", "passthrough");
```

---

## PreprocessorVertex

Wraps an `InputPreProcessor` as a `ComputationGraph` vertex. This allows inserting preprocessing steps (e.g., `CnnToFeedForwardPreProcessor`, `FeedForwardToCnnPreProcessor`) between layers in a graph where the automatic preprocessor insertion does not apply.

```java
import org.deeplearning4j.nn.graph.vertex.impl.PreprocessorVertex;
import org.deeplearning4j.nn.conf.preprocessor.CnnToFeedForwardPreProcessor;

conf.addVertex("cnnToFF",
    new PreprocessorVertex(new CnnToFeedForwardPreProcessor(height, width, channels)),
    "convLayer");
```

---

## ReverseTimeSeriesVertex

Reverses the time axis of a sequence input. Useful for building bidirectional RNN variants manually, where one branch processes the sequence forward and another processes it backwards.

Masked time steps (padding) are handled correctly: only the present (mask = 1) time steps are reversed in place; padding (mask = 0) remains at the end of the reversed sequence.

```java
import org.deeplearning4j.nn.graph.vertex.impl.rnn.ReverseTimeSeriesVertex;

conf.addVertex("reversed", new ReverseTimeSeriesVertex("inputMask"), "rnnInput");
conf.addLayer("backwardRnn",
    new LSTM.Builder().nIn(inputSize).nOut(hiddenSize).build(), "reversed");
```

---

## PoolHelperVertex

A specialised vertex for removing the first row and column from a 4D CNN activation tensor. Originally designed to aid importing Caffe's GoogLeNet architecture where the pooling layer produces an output that is one pixel larger than expected.

```java
import org.deeplearning4j.nn.graph.vertex.impl.PoolHelperVertex;

conf.addVertex("poolHelper", new PoolHelperVertex(), "poolLayer");
```

---

## Custom Vertices

Implement `org.deeplearning4j.nn.graph.vertex.GraphVertex` (or extend `org.deeplearning4j.nn.graph.vertex.BaseGraphVertex`) to create a custom vertex.

Key methods to override:

```java
public class MyVertex extends BaseGraphVertex {

    public MyVertex(ComputationGraph graph, String name, int vertexIndex,
                    MemoryWorkspace workspace) {
        super(graph, name, vertexIndex, workspace);
    }

    @Override
    public boolean hasLayer() { return false; }

    @Override
    public boolean isOutputVertex() { return false; }

    @Override
    public Layer getLayer() { return null; }

    @Override
    public INDArray doForward(boolean training, LayerWorkspaceMgr workspaceMgr) {
        // inputs available via: this.inputs[0], this.inputs[1], ...
        INDArray input0 = inputs[0];
        // ... compute output ...
        return output;
    }

    @Override
    public Pair<Gradient, INDArray[]> doBackward(boolean tbptt, LayerWorkspaceMgr workspaceMgr) {
        // epsilon is the gradient from the next layer
        INDArray epsilon = this.epsilon;
        // ... compute gradients ...
        return new Pair<>(null, new INDArray[]{ gradient0 });
    }
}
```

Register the vertex using the standard `addVertex` API with an instance of your class.

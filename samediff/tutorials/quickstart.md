---
title: "SameDiff Overview"
description: "Automatic differentiation framework in ND4J — define-and-run computation graphs, comparison with MultiLayerNetwork and ComputationGraph"
---

# SameDiff Overview

SameDiff is the automatic differentiation (autograd) framework built into ND4J. It lets you define mathematical computation graphs in Java, execute them against real data, and compute gradients automatically — without writing any backpropagation code by hand.

## What SameDiff Is

At its core, SameDiff represents a computation as a directed acyclic graph (DAG) where:

- **Nodes** are variables (`SDVariable` instances) holding arrays of numbers.
- **Edges** are operations that consume one or more input variables and produce an output variable.

When you write code like:

```java
SameDiff sd = SameDiff.create();
SDVariable x = sd.placeHolder("x", DataType.FLOAT, -1, 784);
SDVariable w = sd.var("w", DataType.FLOAT, 784, 10);
SDVariable b = sd.var("b", DataType.FLOAT, 10);
SDVariable logits = x.mmul(w).add(b);
SDVariable output = sd.nn.softmax("output", logits);
```

you are **defining** the graph, not executing it. No numeric computation happens yet. The graph is a blueprint that SameDiff stores internally. Execution happens separately when you call `output()`, `exec()`, or `fit()`.

This approach is called **define-and-run** (as opposed to the eager evaluation model where each line immediately computes a result).

## Automatic Gradient Computation

The major payoff of building a computation graph is that SameDiff can traverse it in reverse to compute gradients with respect to any variable automatically. When training, SameDiff:

1. Runs the forward pass (evaluates all nodes in topological order).
2. Computes the scalar loss value.
3. Runs the backward pass (applies the chain rule through each op in reverse order).
4. Updates trainable `VARIABLE`-type parameters using the configured optimizer.

You never implement `backward()` methods. The gradients for every built-in operation are pre-registered in the framework.

## Key Classes

| Class | Role |
|---|---|
| `SameDiff` | The graph container. Holds all variables, ops, and training configuration. Create one with `SameDiff.create()`. |
| `SDVariable` | A node in the graph. Wraps an `INDArray` (when values are available) and knows its position in the graph. |
| `TrainingConfig` | Bundles the optimizer, loss variable name, data-type mappings, and listener list for a training run. |
| `History` | Returned by `fit()`; records loss and metric values epoch by epoch. |
| `InferenceSession` | Low-level execution engine; usually used indirectly via `sd.output()`. |

## When to Use SameDiff vs MultiLayerNetwork / ComputationGraph

DL4J provides three ways to build neural networks. Choose based on your needs:

### MultiLayerNetwork

Use when your network is a simple sequential stack of layers. It is the easiest API:

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .list()
    .layer(new DenseLayer.Builder().nIn(784).nOut(256).activation(Activation.RELU).build())
    .layer(new OutputLayer.Builder().nIn(256).nOut(10).activation(Activation.SOFTMAX).build())
    .build();
MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
```

**Best for:** standard feedforward networks, CNNs with a single input/output, beginners.

### ComputationGraph

Use when your network has multiple inputs, multiple outputs, skip connections, or branching paths (e.g. encoder-decoder, Siamese networks). Still configuration-driven but more flexible than `MultiLayerNetwork`.

**Best for:** complex topologies that can still be described with DL4J's built-in layer types.

### SameDiff

Use when you need:

- **Custom operations or loss functions** that have no counterpart in the DL4J layer catalogue.
- **Research and experimentation** where you want full symbolic control over every operation.
- **Fine-grained weight sharing** or unusual parameter tying.
- **Importing and fine-tuning TensorFlow/ONNX models** — the model import pipeline internally produces SameDiff graphs.

SameDiff is more verbose than the DL4J layer APIs but gives you complete flexibility over every computation in your model.

## Building a Simple Neural Net in SameDiff

Here is a complete minimal example of a one-hidden-layer network for MNIST classification, from graph definition through to a training loop.

### Step 1: Define the graph

```java
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.autodiff.samediff.SDVariable;
import org.nd4j.autodiff.samediff.TrainingConfig;
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.learning.config.Adam;
import org.nd4j.weightinit.impl.XavierInitScheme;

SameDiff sd = SameDiff.create();

// Placeholders receive data at runtime
SDVariable input  = sd.placeHolder("input",  DataType.FLOAT, -1, 784);
SDVariable labels = sd.placeHolder("labels", DataType.FLOAT, -1, 10);

// Trainable parameters — Xavier-initialised
SDVariable w1 = sd.var("w1", new XavierInitScheme('c', 784, 256), DataType.FLOAT, 784, 256);
SDVariable b1 = sd.var("b1", DataType.FLOAT, 256);

SDVariable w2 = sd.var("w2", new XavierInitScheme('c', 256, 10), DataType.FLOAT, 256, 10);
SDVariable b2 = sd.var("b2", DataType.FLOAT, 10);

// Forward pass
SDVariable hidden  = sd.nn.relu("hidden",  input.mmul(w1).add(b1), 0);
SDVariable logits  = hidden.mmul(w2).add(b2);
SDVariable softmax = sd.nn.softmax("softmax", logits);

// Loss — cross-entropy averaged over the minibatch
SDVariable loss = sd.loss.softmaxCrossEntropy("loss", labels, logits, null);
```

### Step 2: Configure training

```java
TrainingConfig config = TrainingConfig.builder()
    .updater(new Adam(1e-3))
    .dataSetFeatureMapping("input")        // DataSet feature -> placeholder name
    .dataSetLabelMapping("labels")         // DataSet label   -> placeholder name
    .build();

sd.setTrainingConfig(config);
```

### Step 3: Train

```java
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;

DataSetIterator trainIter = /* your iterator */ null;
int numEpochs = 10;

sd.fit(trainIter, numEpochs);
```

### Step 4: Run inference

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import java.util.Map;

INDArray testInput = /* your test batch */ null;
Map<String, INDArray> results = sd.output(
    Map.of("input", testInput),
    "softmax"
);
INDArray predictions = results.get("softmax");
```

## How the Graph Executes

When `sd.output()` is called, SameDiff internally uses an `InferenceSession` that:

1. Resolves which nodes need to be computed in order to produce the requested output variables.
2. Determines a valid topological execution order.
3. Evaluates each op in that order, passing intermediate results through the graph.
4. Returns the values of the requested output nodes.

Only the ops necessary to compute the requested outputs are evaluated — unreachable subgraphs are skipped.

## Graph Inspection

SameDiff provides several utilities for inspecting the graph you have built:

```java
// Print a summary of all variables and their types
sd.summary();

// List all variable names
List<String> varNames = sd.variableNames();

// Get a variable by name
SDVariable v = sd.getVariable("hidden");

// View the output shape of a variable (without executing)
long[] shape = sd.getShapeForVarName("hidden");
```

## Thread Safety and Multiple Graphs

Each `SameDiff` instance is a self-contained graph. You can have multiple `SameDiff` instances in the same JVM, but variables from one instance cannot be mixed with variables from another. All `SDVariable` objects carry a reference back to their owning `SameDiff`.

`SameDiff` instances are not thread-safe for concurrent mutation. For inference in a multi-threaded server environment, either synchronise access or keep a pool of separate `SameDiff` instances loaded from the same saved file.

## Next Steps

- [Variables](./variables) — learn about `SDVariable` types (`VARIABLE`, `CONSTANT`, `PLACEHOLDER`, `ARRAY`), data types, and type conversion.
- [Operations](./operations) — explore the op namespaces: `sd.math`, `sd.nn`, `sd.cnn`, `sd.rnn`, `sd.loss`, `sd.random`.
- [Training](./training) — configure `TrainingConfig`, run `fit()`, and track progress with `History`.
- [Execution and Inference](./execution) — understand `sd.output()`, placeholder binding, and batch inference.
- [Serialization](./serialization) — save and load graphs with `sd.save()` and `SameDiff.load()`.

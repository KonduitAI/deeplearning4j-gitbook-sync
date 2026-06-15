---
title: "SameDiff Import Overview"
description: "Importing TensorFlow and ONNX models into SameDiff — architecture, supported ops, and usage"
---

## SameDiff Import Overview

The SameDiff import subsystem converts models from external frameworks (TensorFlow, ONNX) into SameDiff computation graphs. Once imported, models are represented as native SameDiff graphs and inherit all SameDiff capabilities: autograd, custom training loops, op inspection, graph manipulation, and export.

---

## Architecture

The import framework is implemented in Kotlin and lives in the `nd4j-samediff-import` family of modules. Its design is structured around three key abstractions:

### FrameworkImporter

`FrameworkImporter` is the top-level interface for importing a model from a given framework. Each supported framework has a concrete implementation:

- `TensorflowFrameworkImporter` — reads TF protobuf or SavedModel formats
- `OnnxFrameworkImporter` — reads ONNX `.onnx` files

### ImportGraph

`ImportGraph` performs the core translation work: it walks the framework-specific graph representation, resolves op mappings, and emits a SameDiff graph. It delegates to a registry of op mappers for individual operation translation.

### Op Mapping Registry

Each framework integration registers a set of op mappers. A mapper takes a framework op (e.g., a TF `MatMul` node) and produces the corresponding ND4J/SameDiff op. The mapping registry can be extended with custom mappers for ops not covered by default.

---

## Supported Formats

| Format | Extension | Import path |
|---|---|---|
| TensorFlow frozen graph | `.pb` (protobuf) | `TFGraphMapper` |
| TensorFlow SavedModel | directory with `saved_model.pb` | `TFGraphMapper` |
| ONNX | `.onnx` | `OnnxGraphMapper` |

---

## Maven Dependencies

### TensorFlow Import

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-samediff-import-tensorflow</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

### ONNX Import

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-samediff-import-onnx</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

---

## General Import Workflow

Regardless of the source framework, the import workflow follows the same pattern:

1. Load the model file into a framework-specific protobuf/binary representation
2. Call the appropriate `FrameworkImporter` or graph mapper
3. The importer walks the graph, resolves op mappings, and emits a `SameDiff` graph
4. Map input placeholders to input names
5. Execute the `SameDiff` graph with input data

```java
// Pseudocode illustrating the general pattern
SameDiff sd = FrameworkImporter.importGraph(modelFile);
Map<String, INDArray> inputs = new HashMap<>();
inputs.put("input_placeholder_name", inputData);
Map<String, INDArray> outputs = sd.output(inputs, "output_node_name");
```

Specific code for TF and ONNX is covered in the respective pages.

---

## Op Coverage

### TensorFlow Ops

The TF import registry covers the majority of ops used by standard TF models. Broadly supported categories include:

- Arithmetic: Add, Sub, Mul, Div, MatMul, BatchMatMul, etc.
- Neural network ops: Conv2D, DepthwiseConv2D, BiasAdd, Relu, Sigmoid, Softmax, etc.
- Pooling: MaxPool, AvgPool, MaxPool3D, AvgPool3D
- Normalization: FusedBatchNorm, FusedBatchNormV2, FusedBatchNormV3
- Recurrent: LSTM, LSTMBlockCell, GRU
- Shape manipulation: Reshape, Transpose, Concat, Stack, Unstack, Split, Slice
- Control flow: Switch, Merge, Enter, Exit, NextIteration (TF 1.x style)
- Reduction: ReduceSum, ReduceMean, ReduceMax, ReduceMin, ReduceAll, ReduceAny
- Embedding and lookup: GatherV2, ResourceGather
- Image ops: ResizeBilinear, ResizeNearestNeighbor, CropAndResize

Unsupported ops result in an `OpNotYetImplementedException` at import time. The error message includes the op name; opening a GitHub issue or contributing a mapper is encouraged.

### ONNX Ops

The ONNX op set coverage follows the ONNX standard opset versions. Broadly supported:

- Arithmetic: Add, Sub, Mul, Div, Gemm, MatMul
- Activations: Relu, Sigmoid, Tanh, Softmax, Elu, Selu, LeakyRelu
- Convolution: Conv, ConvTranspose
- Pooling: MaxPool, AveragePool, GlobalMaxPool, GlobalAveragePool
- Normalization: BatchNormalization, InstanceNormalization
- Recurrent: LSTM, GRU, RNN
- Shape: Reshape, Transpose, Concat, Flatten, Squeeze, Unsqueeze
- Reduction: ReduceSum, ReduceMean, ReduceMax
- Other: Gather, Slice, Cast, Identity, Dropout (inference mode)

---

## How Op Mapping Works

Each op mapper implements the `OpMapper` interface. A mapper receives:

- The framework op's configuration (attributes, inputs, outputs)
- The in-progress SameDiff graph

It produces:

- One or more SameDiff ops added to the graph

Mappers handle attribute translation (e.g., TF `data_format="NHWC"` to ND4J channel-last mode), input reordering, and output naming.

The registry is consulted by name: for each op type encountered in the source graph, the registry looks up the registered mapper. Ops with no registered mapper result in an error unless a passthrough is configured.

---

## SameDiff Graph Capabilities After Import

Once a model is imported as a `SameDiff` graph, you can:

- **Run inference**: call `sd.output(inputs, outputNames)` with input data
- **Inspect the graph**: call `sd.summary()` or iterate over `sd.ops()` 
- **Modify the graph**: add ops, change inputs, replace subgraphs
- **Compute gradients**: call `sd.calculateGradients()` for autograd
- **Fine-tune**: set up a training configuration on the imported graph
- **Export**: serialize the SameDiff graph for later loading

```java
// List all ops in the imported graph
for (SameDiffOp op : sd.ops().values()) {
    System.out.println(op.getName() + " : " + op.getOp().getClass().getSimpleName());
}
```

---

## Limitations

- Custom TF ops (registered via `tf.RegisterGradient` or C++ extension) cannot be imported unless a custom mapper is implemented
- TF 2.x eager mode models must be converted to a frozen graph or SavedModel before import
- ONNX models using very recent opset versions may contain ops not yet mapped; check the op registry for your opset version
- Dynamic control flow in TF models (while loops, if branches) is partially supported; complex dynamic shapes may require manual graph preprocessing

---

## Further Reading

- [TensorFlow Import](./tensorflow) — importing `.pb` and SavedModel files
- [ONNX Import](./onnx) — importing `.onnx` files
- [SameDiff documentation](../../samediff/) — SameDiff API and training

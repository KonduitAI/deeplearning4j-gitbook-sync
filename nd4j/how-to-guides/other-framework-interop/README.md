---
title: "Model Import Overview"
description: "All model import paths in DL4J — Keras to DL4J, TF/ONNX to SameDiff, and direct inference runtimes"
---

## Model Import Overview

Eclipse Deeplearning4j provides multiple paths for importing pre-trained models from other frameworks. Whether your model was trained in Keras, TensorFlow, or exported to ONNX, there is a supported import or inference path that brings it into the JVM ecosystem for production deployment.

This page maps out all available import paths, provides a decision tree to help you pick the right one, and gives a comparison across available approaches.

---

## Import Paths at a Glance

DL4J supports the following import and inference strategies:

| Strategy | Input Format | Output | Use Case |
|---|---|---|---|
| Keras model import | Keras H5 (.h5) | MultiLayerNetwork / ComputationGraph | Import and run Keras models in DL4J |
| SameDiff TF import | TF frozen .pb / SavedModel | SameDiff graph | Import TF graphs for inference or further training |
| SameDiff ONNX import | ONNX .onnx | SameDiff graph | Import ONNX models via SameDiff |
| ONNX Runtime | ONNX .onnx | OrtSession inference | Direct ONNX inference without conversion |
| TF Direct (JavaCPP) | TF frozen .pb | TF Session inference | Run TF graphs via JavaCPP TF bindings |
| TensorFlow Lite | .tflite | TFLite interpreter | Edge/mobile inference with TFLite 2.8 |
| Apache TVM | Compiled TVM module | TVM runtime inference | Compiler-optimized inference via TVM 0.8 |

---

## Decision Tree: Which Import Path to Use?

Work through the questions below to identify the right approach for your situation.

### Step 1: What is your model source?

**Keras model (.h5)?**
Go to [Keras Import](#keras-model-import).

**TensorFlow frozen graph (.pb) or SavedModel?**
Continue to Step 2.

**ONNX model (.onnx)?**
Continue to Step 3.

**Already compiled TVM module?**
Go to [Apache TVM](#apache-tvm-integration).

**TensorFlow Lite model (.tflite)?**
Go to [TensorFlow Lite](#tensorflow-lite).

---

### Step 2: TensorFlow models

Do you need to **further train** the model inside DL4J/SameDiff, or access intermediate activations?

- Yes: use [SameDiff TF import](./samediff-import/tensorflow). SameDiff converts the frozen graph into a SameDiff computation graph that supports autograd and further training.
- No (inference only): consider [TF Direct inference](./tensorflow/overview) via `nd4j-tensorflow` (JavaCPP bindings). This runs the graph natively without conversion overhead.

---

### Step 3: ONNX models

Do you need a **Java-native graph representation** (e.g., to inspect ops, modify the graph, or train)?

- Yes: use [SameDiff ONNX import](./samediff-import/onnx).
- No (pure inference): use [ONNX Runtime](./onnx-runtime/overview) via `nd4j-onnxruntime`. This delegates execution to the ONNX Runtime 1.10 C++ library for optimal throughput without any conversion.

---

## Import Path Details

### Keras Model Import

The `deeplearning4j-modelimport` module reads Keras H5 files (produced by `model.save()`) and maps:

- `Sequential` models to `MultiLayerNetwork`
- Functional API models to `ComputationGraph`

Weights, layer configurations, and (optionally) training configurations are all preserved. After import, the model participates fully in the DL4J ecosystem for inference, transfer learning, or continued training.

**Supported Keras versions:** 1.x and 2.x (TensorFlow, Theano, and CNTK backends)

**Relevant pages:**
- [Keras Import Overview](./keras/overview)
- [Getting Started](./keras/getting-started)
- [Sequential Model](./keras/sequential-model)
- [Functional Model](./keras/functional-model)
- [Supported Features](./keras/supported-features)
- [API Reference](./keras/model-import-api)

---

### SameDiff TF/ONNX Import

The `nd4j-samediff-import` subsystem (written in Kotlin) provides a `FrameworkImporter` abstraction and concrete `ImportGraph` implementations for TensorFlow and ONNX. After import, the model lives as a `SameDiff` graph and inherits all SameDiff capabilities: autograd, custom training loops, op inspection, and export.

**Relevant pages:**
- [SameDiff Import Overview](./samediff-import/overview)
- [TensorFlow Import](./samediff-import/tensorflow)
- [ONNX Import](./samediff-import/onnx)

---

### ONNX Runtime

The `nd4j-onnxruntime` module wraps ONNX Runtime 1.10. Models run directly in the ORT C++ runtime; there is no conversion to a SameDiff graph. This is the fastest path for ONNX inference when you do not need graph mutation.

**Relevant pages:**
- [ONNX Runtime Overview](./onnx-runtime/overview)

---

### TensorFlow Direct Inference

The `nd4j-tensorflow` module uses JavaCPP TF bindings to execute TensorFlow frozen graphs natively. No conversion occurs. Suitable when you have a stable frozen graph and want minimal overhead inference.

**Relevant pages:**
- [TF Direct Inference Overview](./tensorflow/overview)

---

### TensorFlow Lite

The `nd4j-tensorflow-lite` module integrates TFLite 2.8 for embedded and edge inference. Accepts `.tflite` flatbuffer models. Ideal for Android and constrained-resource deployments.

**Relevant pages:**
- [TensorFlow Lite](./tensorflow/tensorflow-lite)

---

### Apache TVM Integration

The `nd4j-tvm` module integrates Apache TVM 0.8. TVM is a machine learning compiler that produces optimized native binaries tuned for a target hardware backend. You compile your model with TVM's Python toolchain first, then load the resulting module in Java.

**Relevant pages:**
- [Apache TVM Overview](./tvm/overview)

---

## Comparison Table

| Feature | Keras Import | SameDiff TF | SameDiff ONNX | ORT | TF Direct | TFLite | TVM |
|---|---|---|---|---|---|---|---|
| Java-native graph | Yes (DL4J) | Yes (SameDiff) | Yes (SameDiff) | No | No | No | No |
| Further training | Yes | Yes | Partial | No | No | No | No |
| No conversion overhead | No | No | No | Yes | Yes | Yes | Yes |
| Op coverage | Keras ops | TF op set | ONNX op set | ONNX op set | TF op set | TFLite op set | Wide |
| Edge/mobile | No | No | No | Limited | No | Yes | Yes |
| Compiler optimization | No | No | No | Limited | No | No | Yes |
| Maven module | deeplearning4j-modelimport | nd4j-samediff-import-tensorflow | nd4j-samediff-import-onnx | nd4j-onnxruntime | nd4j-tensorflow | nd4j-tensorflow-lite | nd4j-tvm |

---

## Maven Coordinates

All modules share the same version. Replace `${dl4j.version}` with your project version (e.g., `1.0.0-M2.1`).

```xml
<!-- Keras model import -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-modelimport</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- SameDiff TF import -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-samediff-import-tensorflow</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- SameDiff ONNX import -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-samediff-import-onnx</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- ONNX Runtime direct inference -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-onnxruntime</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- TF direct inference -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-tensorflow</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- TFLite -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-tensorflow-lite</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- Apache TVM -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-tvm</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

---

## Troubleshooting

**`IncompatibleKerasConfigurationException`**: the Keras model uses a layer or feature not yet supported by the importer. Check [supported features](./keras/supported-features) and file a GitHub issue if needed.

**`UnsupportedKerasConfigurationException`**: a specific parameter value or combination is not handled. Usually workable by adjusting the Keras model before export (e.g., removing unsupported regularizers).

**Op not found during SameDiff import**: the TF or ONNX graph contains an op with no registered SameDiff mapping. Check the op coverage tables in the SameDiff import pages and consider filing an issue or contributing a mapping.

**ORT session creation fails**: verify that the `nd4j-onnxruntime` native binaries match your OS/architecture. ORT ships CPU binaries by default; GPU requires additional configuration.

---

## Further Reading

- [DL4J examples repository](https://github.com/eclipse/deeplearning4j-examples)
- [Model import source code](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport)
- [SameDiff import source code](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff/transform)

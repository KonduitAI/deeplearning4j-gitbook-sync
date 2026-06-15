---
title: "ONNX Runtime"
description: "Direct ONNX model inference via ONNX Runtime 1.10 — no conversion to SameDiff required"
---

## ONNX Runtime

The `nd4j-onnxruntime` module provides direct ONNX model inference via ONNX Runtime (ORT) 1.10. Unlike [SameDiff ONNX import](../samediff-import/onnx), there is no conversion of the model into a SameDiff graph. The ONNX model runs natively inside the ORT C++ runtime, which is accessed from Java via JavaCPP bindings.

This is the fastest path for ONNX inference when you do not need to inspect or modify the graph.

---

## When to Use ONNX Runtime vs SameDiff Import

| Requirement | Use ORT | Use SameDiff Import |
|---|---|---|
| Pure inference, minimum latency | Yes | No |
| Inspect or modify the graph in Java | No | Yes |
| Further training in Java | No | Yes |
| Op coverage: maximum ONNX compatibility | Yes (ORT is the reference) | Partial |
| Custom Java op integration | No | Yes |
| GPU execution | Yes (CUDA EP) | Via ND4J CUDA backend |

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-onnxruntime</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

The module bundles native ORT binaries for Linux x86_64, Windows x86_64, and macOS x86_64. ARM platforms require building from source.

---

## Basic Usage

```java
import org.nd4j.onnxruntime.runner.OnnxRuntimeRunner;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

import java.util.LinkedHashMap;
import java.util.Map;

// Load the ONNX model
OnnxRuntimeRunner runner = OnnxRuntimeRunner.builder()
        .modelUri("path/to/model.onnx")
        .build();

// Prepare input data
INDArray input = Nd4j.rand(1, 3, 224, 224);  // NCHW image

// Build input map — keys must match ONNX model input names
Map<String, INDArray> inputs = new LinkedHashMap<>();
inputs.put("input", input);

// Run inference
Map<String, INDArray> outputs = runner.exec(inputs);

// Retrieve output
INDArray predictions = outputs.get("output");
System.out.println("Output shape: " + java.util.Arrays.toString(predictions.shape()));

// Close the runner when done
runner.close();
```

---

## Finding Input and Output Names

Use Python to inspect the ONNX model for input and output tensor names before running in Java:

```python
import onnx

model = onnx.load("model.onnx")
print("Inputs:")
for inp in model.graph.input:
    print(f"  {inp.name}")

print("Outputs:")
for out in model.graph.output:
    print(f"  {out.name}")
```

Alternatively, use ONNX Runtime Python API:

```python
import onnxruntime as ort

sess = ort.InferenceSession("model.onnx")
for inp in sess.get_inputs():
    print(f"Input: {inp.name}, shape: {inp.shape}, dtype: {inp.type}")
for out in sess.get_outputs():
    print(f"Output: {out.name}, shape: {out.shape}")
```

---

## OnnxRuntimeRunner Builder Options

```java
OnnxRuntimeRunner runner = OnnxRuntimeRunner.builder()
        .modelUri("path/to/model.onnx")     // path to .onnx file (required)
        .build();
```

The runner manages the ORT session lifecycle. Close it when done to release native resources:

```java
try (OnnxRuntimeRunner runner = OnnxRuntimeRunner.builder()
        .modelUri("model.onnx")
        .build()) {

    Map<String, INDArray> outputs = runner.exec(inputs);
    // use outputs...
}
```

---

## Batched Inference

ONNX Runtime supports batched inference when the model has a dynamic batch dimension:

```python
# Python export with dynamic batch
torch.onnx.export(
    model, dummy, "model.onnx",
    dynamic_axes={"input": {0: "batch_size"}, "output": {0: "batch_size"}}
)
```

```java
// Java: batch of 8
INDArray batchInput = Nd4j.rand(8, 3, 224, 224);
inputs.put("input", batchInput);
Map<String, INDArray> outputs = runner.exec(inputs);
// outputs["output"] shape: [8, num_classes]
```

---

## Data Types

ONNX Runtime accepts various tensor data types. ND4J `INDArray` types are mapped:

| ND4J DataType | ONNX Tensor Type |
|---|---|
| FLOAT | FLOAT (float32) |
| DOUBLE | DOUBLE (float64) |
| INT32 | INT32 |
| INT64 | INT64 |
| BOOL | BOOL |
| FLOAT16 | FLOAT16 |

Ensure your `INDArray` has the correct data type before passing to the runner. Cast if necessary:

```java
INDArray floatInput = Nd4j.rand(1, 3, 224, 224).castTo(org.nd4j.linalg.api.buffer.DataType.FLOAT);
```

---

## Complete Example: Image Classification

```java
import org.nd4j.onnxruntime.runner.OnnxRuntimeRunner;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

import java.util.LinkedHashMap;
import java.util.Map;

public class OrtInferenceExample {

    public static void main(String[] args) throws Exception {
        String modelPath = "resnet50.onnx";

        try (OnnxRuntimeRunner runner = OnnxRuntimeRunner.builder()
                .modelUri(modelPath)
                .build()) {

            // Simulate a preprocessed image [batch=1, channels=3, H=224, W=224]
            INDArray image = Nd4j.rand(new int[]{1, 3, 224, 224})
                    .castTo(org.nd4j.linalg.api.buffer.DataType.FLOAT);

            Map<String, INDArray> inputs = new LinkedHashMap<>();
            inputs.put("input", image);

            long start = System.currentTimeMillis();
            Map<String, INDArray> outputs = runner.exec(inputs);
            long elapsed = System.currentTimeMillis() - start;

            INDArray logits = outputs.get("output");
            int topClass = logits.argMax(1).getInt(0);

            System.out.printf("Top class: %d  (inference time: %d ms)%n", topClass, elapsed);
        }
    }
}
```

---

## Troubleshooting

**Native library not found**: verify that the `nd4j-onnxruntime` artifact was downloaded correctly and that the platform classifier matches your OS/architecture. On Linux, ensure `libonnxruntime.so` is accessible.

**Wrong input name**: the input name passed to the runner must exactly match the ONNX model's input tensor name. Use the Python inspection snippet to verify.

**Data type mismatch**: ORT is strict about tensor data types. If the model expects `FLOAT` and you provide `DOUBLE`, an error is thrown. Cast the array to the expected type before inference.

**Session already closed**: `OnnxRuntimeRunner` is `AutoCloseable`. After calling `close()` or exiting a try-with-resources block, the session is released and cannot be reused.

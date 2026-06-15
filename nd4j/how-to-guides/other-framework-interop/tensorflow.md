---
title: "TensorFlow Direct Inference"
description: "Running TensorFlow frozen graphs directly via JavaCPP TF bindings"
---

## TensorFlow Direct Inference

The `nd4j-tensorflow` module runs TensorFlow frozen graphs natively using JavaCPP TF bindings. There is no conversion to a SameDiff graph; the TF runtime executes the graph directly. This is appropriate when you have a stable frozen graph and want low-overhead inference without the SameDiff import layer.

For models that need graph inspection or further training in Java, use [SameDiff TF import](../samediff-import/tensorflow) instead.

---

## When to Use TF Direct vs SameDiff Import

| Requirement | TF Direct | SameDiff TF Import |
|---|---|---|
| Pure inference, stable frozen graph | Yes | Yes |
| Graph inspection in Java | No | Yes |
| Further training in Java | No | Yes |
| Dependency on TF native libraries | Yes (JavaCPP) | No |
| Access TF-specific ops not in SameDiff | Yes | No |

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-tensorflow</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

The module bundles TensorFlow native libraries via JavaCPP presets for Linux x86_64, Windows x86_64, and macOS.

---

## Preparing the Model

TF direct inference requires a frozen graph (a protobuf where all `Variable` ops have been converted to constants). See [TensorFlow Import — Preparing Your Model](../samediff-import/tensorflow#preparing-your-tensorflow-model) for Python export instructions.

---

## Running Inference

Use `TensorFlowRunner` to load and execute a frozen graph:

```java
import org.nd4j.tensorflow.conversion.graphrunner.GraphRunner;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

import java.util.Arrays;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

// Specify input names and output names
List<String> inputNames  = Arrays.asList("input_1");
List<String> outputNames = Arrays.asList("output/Softmax");

// Build the runner
GraphRunner runner = GraphRunner.builder()
        .graphPath("frozen_model.pb")
        .inputNames(inputNames)
        .outputNames(outputNames)
        .build();

// Prepare inputs
INDArray input = Nd4j.rand(1, 224, 224, 3).castTo(org.nd4j.linalg.api.buffer.DataType.FLOAT);
Map<String, INDArray> inputs = new LinkedHashMap<>();
inputs.put("input_1", input);

// Run
Map<String, INDArray> outputs = runner.run(inputs);
INDArray predictions = outputs.get("output/Softmax");
System.out.println("Shape: " + Arrays.toString(predictions.shape()));

runner.close();
```

---

## GraphRunner Builder Options

```java
GraphRunner runner = GraphRunner.builder()
        .graphPath("frozen_model.pb")      // path to frozen .pb file
        .inputNames(inputNames)             // list of placeholder node names
        .outputNames(outputNames)           // list of output node names to fetch
        .build();
```

`GraphRunner` is `AutoCloseable`:

```java
try (GraphRunner runner = GraphRunner.builder()
        .graphPath("frozen_model.pb")
        .inputNames(inputNames)
        .outputNames(outputNames)
        .build()) {

    Map<String, INDArray> result = runner.run(inputs);
}
```

---

## Data Format Notes

TensorFlow defaults to NHWC (channels-last) data format for images: `[batch, height, width, channels]`. This differs from ND4J's default NCHW. Construct your input `INDArray` in NHWC order for TF models that were not explicitly configured for NCHW.

```java
// NHWC: batch=1, height=224, width=224, channels=3
INDArray nhwcInput = Nd4j.rand(1, 224, 224, 3);
```

If your model uses `data_format='channels_first'` (NCHW), adjust accordingly:

```java
// NCHW: batch=1, channels=3, height=224, width=224
INDArray nchwInput = Nd4j.rand(1, 3, 224, 224);
```

---

## Getting Node Names

To find placeholder and output node names in a frozen graph:

```python
import tensorflow as tf

graph_def = tf.compat.v1.GraphDef()
with open('frozen_model.pb', 'rb') as f:
    graph_def.ParseFromString(f.read())

# Print all node names and types
for node in graph_def.node:
    print(f"{node.op:30s}  {node.name}")
```

---

## Complete Example

```java
import org.nd4j.tensorflow.conversion.graphrunner.GraphRunner;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

import java.util.Arrays;
import java.util.LinkedHashMap;
import java.util.Map;

public class TfDirectInferenceExample {

    public static void main(String[] args) throws Exception {
        String modelPath   = "frozen_model.pb";
        String inputName   = "input_1";
        String outputName  = "predictions/Softmax";

        try (GraphRunner runner = GraphRunner.builder()
                .graphPath(modelPath)
                .inputNames(Arrays.asList(inputName))
                .outputNames(Arrays.asList(outputName))
                .build()) {

            // NHWC image: batch=1, H=224, W=224, C=3
            INDArray image = Nd4j.rand(1, 224, 224, 3)
                    .castTo(org.nd4j.linalg.api.buffer.DataType.FLOAT);

            Map<String, INDArray> inputs = new LinkedHashMap<>();
            inputs.put(inputName, image);

            Map<String, INDArray> outputs = runner.run(inputs);
            INDArray softmax = outputs.get(outputName);

            int topClass = softmax.argMax(1).getInt(0);
            System.out.println("Predicted class: " + topClass);
        }
    }
}
```

---

## Troubleshooting

**Native library load failure**: the TF JavaCPP preset must match your platform. Verify that the artifact downloaded native libraries for your OS. On Linux, `LD_LIBRARY_PATH` may need to include the directory containing `libtensorflow.so`.

**Node name not found**: node names in TF frozen graphs are case-sensitive and include the full path (e.g., `conv1/Relu` not just `Relu`). Use the Python node listing snippet to find exact names.

**Variable op in graph**: `GraphRunner` requires a fully frozen graph. Variables that were not converted to constants cause runtime errors. Re-run the Python freezing step.

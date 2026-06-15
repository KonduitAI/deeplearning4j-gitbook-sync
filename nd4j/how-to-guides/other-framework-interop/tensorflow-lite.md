---
title: "TensorFlow Lite"
description: "TensorFlow Lite 2.8 inference for mobile and edge deployment"
---

## TensorFlow Lite

The `nd4j-tensorflow-lite` module integrates TensorFlow Lite (TFLite) 2.8 for inference on mobile and edge devices. It accepts `.tflite` flatbuffer models and runs them via the TFLite C++ interpreter, accessed from Java through JavaCPP bindings.

TFLite models are optimized for constrained environments: they are smaller, faster, and use less memory than full TF models. Android deployment and ARM-based edge hardware are primary use cases.

---

## When to Use TFLite

| Scenario | Recommended |
|---|---|
| Android deployment | Yes |
| Edge device (Raspberry Pi, Jetson Nano) | Yes |
| Quantized model (INT8, FLOAT16) | Yes |
| Server-side inference with large models | No — use ORT or SameDiff |
| Model inspection or training | No — use SameDiff |

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-tensorflow-lite</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

The module includes TFLite 2.8 native libraries for Linux x86_64 and ARM64, and Android ARM64/ARM32.

---

## Converting a Model to TFLite

### From Keras / TF SavedModel

```python
import tensorflow as tf

# From a saved Keras model
converter = tf.lite.TFLiteConverter.from_saved_model('my_saved_model')
tflite_model = converter.convert()
with open('model.tflite', 'wb') as f:
    f.write(tflite_model)
```

### With INT8 Quantization

```python
import tensorflow as tf
import numpy as np

def representative_dataset():
    for _ in range(100):
        yield [np.random.rand(1, 224, 224, 3).astype(np.float32)]

converter = tf.lite.TFLiteConverter.from_saved_model('my_saved_model')
converter.optimizations = [tf.lite.Optimize.DEFAULT]
converter.representative_dataset = representative_dataset
converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
converter.inference_input_type  = tf.int8
converter.inference_output_type = tf.int8

tflite_model = converter.convert()
with open('model_quant.tflite', 'wb') as f:
    f.write(tflite_model)
```

### From a Frozen Graph

```python
import tensorflow as tf

converter = tf.compat.v1.lite.TFLiteConverter.from_frozen_graph(
    'frozen_model.pb',
    input_arrays=['input_1'],
    output_arrays=['output/Softmax']
)
tflite_model = converter.convert()
with open('model.tflite', 'wb') as f:
    f.write(tflite_model)
```

---

## Loading and Running a TFLite Model

Use `TFLiteRunner` to load a `.tflite` model and run inference:

```java
import org.nd4j.tensorflow.conversion.TFLiteRunner;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

import java.util.LinkedHashMap;
import java.util.Map;

try (TFLiteRunner runner = TFLiteRunner.builder()
        .modelUri("model.tflite")
        .build()) {

    // Prepare input
    // TFLite models commonly expect NHWC: [batch, height, width, channels]
    INDArray input = Nd4j.rand(1, 224, 224, 3)
            .castTo(org.nd4j.linalg.api.buffer.DataType.FLOAT);

    Map<String, INDArray> inputs = new LinkedHashMap<>();
    inputs.put("input", input);  // key = tensor name or index

    Map<String, INDArray> outputs = runner.exec(inputs);
    INDArray predictions = outputs.get("output");

    int topClass = predictions.argMax(1).getInt(0);
    System.out.println("Top class: " + topClass);
}
```

---

## Finding Tensor Names

TFLite models expose tensor metadata. Inspect in Python before import:

```python
import tensorflow as tf

interpreter = tf.lite.Interpreter(model_path="model.tflite")
interpreter.allocate_tensors()

print("Input details:")
for d in interpreter.get_input_details():
    print(f"  name={d['name']}, shape={d['shape']}, dtype={d['dtype']}")

print("Output details:")
for d in interpreter.get_output_details():
    print(f"  name={d['name']}, shape={d['shape']}, dtype={d['dtype']}")
```

---

## Quantized Models

TFLite supports INT8 quantized models for faster inference on hardware with integer accelerators. When using a quantized model, ensure your input data is quantized appropriately:

```java
// For INT8 models: scale to [-128, 127]
INDArray floatInput = Nd4j.rand(1, 224, 224, 3);
// Apply quantization: input_int8 = float_input / scale + zero_point
// scale and zero_point come from interpreter.get_input_details()[0]['quantization']
INDArray int8Input = floatInput.mul(255).sub(128)
        .castTo(org.nd4j.linalg.api.buffer.DataType.INT8);

inputs.put("input", int8Input);
```

For FLOAT16 quantized models, no special handling is needed — the TFLite interpreter dequantizes internally.

---

## Complete Example: MobileNet V2 on Android

This example targets Android deployment. The Maven dependency is replaced with the Android-specific AAR in a Gradle build file, but the Java API is identical.

```java
// Android (same Java API)
import org.nd4j.tensorflow.conversion.TFLiteRunner;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

import java.io.*;
import java.util.*;

public class MobileNetActivity {

    private TFLiteRunner runner;

    public void init(Context context) throws Exception {
        // Load model from Android assets
        InputStream modelStream = context.getAssets().open("mobilenet_v2.tflite");
        byte[] modelBytes = readBytes(modelStream);

        runner = TFLiteRunner.builder()
                .modelBytes(modelBytes)
                .build();
    }

    public int classify(float[] pixelData) {
        // pixelData: 224*224*3 floats, normalized to [0,1]
        INDArray input = Nd4j.create(pixelData, new int[]{1, 224, 224, 3});

        Map<String, INDArray> inputs = new LinkedHashMap<>();
        inputs.put("input", input);

        Map<String, INDArray> outputs = runner.exec(inputs);
        INDArray probs = outputs.get("MobilenetV2/Predictions/Reshape_1");

        return probs.argMax(1).getInt(0);
    }

    public void close() {
        if (runner != null) runner.close();
    }

    private byte[] readBytes(InputStream is) throws IOException {
        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        byte[] chunk = new byte[4096];
        int n;
        while ((n = is.read(chunk)) != -1) {
            buffer.write(chunk, 0, n);
        }
        return buffer.toByteArray();
    }
}
```

---

## Troubleshooting

**Unsupported op**: TFLite has a restricted op set. If conversion fails or inference throws an error about an unsupported op, use `converter.target_spec.supported_ops` to include `TFLITE_BUILTINS` and `SELECT_TF_OPS` for a broader op set during Python conversion.

**Model not found**: on Android, place `.tflite` files in `src/main/assets/`. Access them via `context.getAssets().open("model.tflite")`.

**Wrong data type**: quantized INT8 models require INT8 inputs. Float models require FLOAT32. Mismatches cause runtime errors.

**Inference speed**: if inference is slow on a device with a Neural Processing Unit (NPU), ensure the TFLite delegate for that hardware is enabled (e.g., NNAPI delegate on Android, GPU delegate). The nd4j-tensorflow-lite module uses the CPU delegate by default.

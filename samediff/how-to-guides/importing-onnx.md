---
title: "ONNX Import"
description: "Importing ONNX models into SameDiff"
---

## ONNX Import

The `nd4j-samediff-import-onnx` module imports ONNX (Open Neural Network Exchange) models into SameDiff. After import, the model is a native SameDiff graph.

For pure inference without graph conversion, consider [ONNX Runtime](../onnx-runtime/overview) instead.

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-samediff-import-onnx</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

---

## Exporting an ONNX Model

Most frameworks can export to ONNX. Common export patterns:

**From PyTorch:**

```python
import torch
import torch.onnx

model = MyModel()
model.eval()
dummy_input = torch.randn(1, 3, 224, 224)

torch.onnx.export(
    model,
    dummy_input,
    "model.onnx",
    opset_version=11,
    input_names=["input"],
    output_names=["output"]
)
```

**From Keras/TensorFlow:**

```python
import tf2onnx
import tensorflow as tf

model = tf.saved_model.load("my_saved_model")
spec = (tf.TensorSpec((None, 3, 224, 224), tf.float32, name="input"),)
output_path = "model.onnx"

model_proto, _ = tf2onnx.convert.from_function(
    model.signatures["serving_default"],
    input_signature=spec,
    opset=11,
    output_path=output_path
)
```

**From scikit-learn via sklearn-onnx:**

```python
from skl2onnx import convert_sklearn
from skl2onnx.common.data_types import FloatTensorType

initial_type = [("float_input", FloatTensorType([None, 4]))]
onx = convert_sklearn(clf, initial_types=initial_type)
with open("model.onnx", "wb") as f:
    f.write(onx.SerializeToString())
```

---

## Importing an ONNX Model

Use `OnnxGraphMapper` to import an ONNX file:

```java
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.imports.graphmapper.onnx.OnnxGraphMapper;

import java.io.File;

File onnxFile = new File("model.onnx");
SameDiff sd = OnnxGraphMapper.importGraph(onnxFile);
```

From an `InputStream`:

```java
import java.io.FileInputStream;
import java.io.InputStream;

try (InputStream is = new FileInputStream("model.onnx")) {
    SameDiff sd = OnnxGraphMapper.importGraph(is);
}
```

---

## Running Inference

After import, provide values for each ONNX input by name:

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

import java.util.HashMap;
import java.util.Map;

// Input matching the ONNX model's expected shape
INDArray inputData = Nd4j.rand(1, 3, 224, 224);  // NCHW for a typical image model

Map<String, INDArray> inputs = new HashMap<>();
inputs.put("input", inputData);

// Run inference; specify output node names from the ONNX model
Map<String, INDArray> result = sd.output(inputs, "output");

INDArray predictions = result.get("output");
System.out.println("Output shape: " + java.util.Arrays.toString(predictions.shape()));
```

---

## Inspecting the Graph

To find input and output names in an ONNX model before or after import:

```python
# Python: inspect ONNX model
import onnx

model = onnx.load("model.onnx")
print("Inputs:")
for inp in model.graph.input:
    print(f"  {inp.name}: {[d.dim_value for d in inp.type.tensor_type.shape.dim]}")

print("Outputs:")
for out in model.graph.output:
    print(f"  {out.name}: {[d.dim_value for d in out.type.tensor_type.shape.dim]}")
```

In Java, after import:

```java
// List all variables; placeholders are the ONNX inputs
for (SDVariable v : sd.variables()) {
    if (v.isPlaceHolder()) {
        System.out.println("Input: " + v.name());
    }
}
```

---

## Supported ONNX Ops

The following ONNX operators are mapped to SameDiff:

**Arithmetic:**
- Add, Sub, Mul, Div, Pow, Mod, Abs, Neg, Exp, Log, Sqrt, Ceil, Floor, Round

**Linear algebra:**
- Gemm, MatMul, BatchMatMul

**Activation functions:**
- Relu, Sigmoid, Tanh, Softmax, LogSoftmax, Elu, Selu, LeakyRelu, PRelu, HardSigmoid, HardSwish, Mish, Swish

**Convolution:**
- Conv, ConvTranspose, DepthwiseConv

**Pooling:**
- MaxPool, AveragePool, GlobalMaxPool, GlobalAveragePool, LpPool

**Normalization:**
- BatchNormalization, InstanceNormalization, LayerNormalization, GroupNormalization

**Recurrent:**
- LSTM, GRU, RNN

**Shape manipulation:**
- Reshape, Transpose, Concat, Flatten, Squeeze, Unsqueeze, Expand, Tile, Pad

**Indexing and slicing:**
- Gather, GatherElements, GatherND, Slice, ScatterElements, ScatterND, NonZero

**Reduction:**
- ReduceSum, ReduceMean, ReduceMax, ReduceMin, ReduceProd, ReduceL1, ReduceL2

**Type:**
- Cast, Identity, Constant, ConstantOfShape

**Comparison:**
- Equal, Greater, GreaterOrEqual, Less, LessOrEqual, Not, And, Or, Where

**Other:**
- Dropout (inference mode — passthrough), Shape, Size, Range, OneHot, EyeLike

---

## Supported ONNX Opsets

The op mapper is tested against ONNX opsets 9 through 13. Most ops in these opsets are covered. Newer opset versions may introduce ops not yet mapped; use `onnx.version_converter.convert_version()` in Python to downgrade to opset 11 if needed.

---

## Complete Example: ResNet-50 Inference

```python
# Python: export ResNet-50 to ONNX
import torch
import torchvision.models as models

model = models.resnet50(pretrained=True)
model.eval()

dummy = torch.randn(1, 3, 224, 224)
torch.onnx.export(
    model, dummy, "resnet50.onnx",
    opset_version=11,
    input_names=["input"],
    output_names=["output"],
    dynamic_axes={"input": {0: "batch"}, "output": {0: "batch"}}
)
```

```java
// Java: import and run
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.imports.graphmapper.onnx.OnnxGraphMapper;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

import java.io.File;
import java.util.*;

public class ResNet50Example {
    public static void main(String[] args) throws Exception {
        SameDiff sd = OnnxGraphMapper.importGraph(new File("resnet50.onnx"));

        // ResNet expects NCHW: [batch, 3, 224, 224]
        // ImageNet normalization would be applied here in a real pipeline
        INDArray image = Nd4j.rand(1, 3, 224, 224);

        Map<String, INDArray> inputs = new HashMap<>();
        inputs.put("input", image);

        Map<String, INDArray> result = sd.output(inputs, "output");
        INDArray logits = result.get("output");

        int topClass = logits.argMax(1).getInt(0);
        System.out.println("Predicted class: " + topClass);
    }
}
```

---

## Troubleshooting

**Op not supported**: the ONNX model contains an operator with no registered mapper. Note the op name from the error and check whether it appears in the supported list above. File a GitHub issue for coverage gaps.

**Input name not found**: use the Python ONNX inspection snippet to find the exact input names. Names are case-sensitive.

**Shape mismatch**: ONNX models (especially those exported from PyTorch) typically use NCHW format `[batch, channels, height, width]`. Ensure your input `INDArray` is constructed in this layout.

**Dynamic shapes**: ONNX models with `dim_value=0` (fully dynamic dimensions) may need concrete shapes provided at inference time. The importer handles most common patterns, but unusual dynamic shape expressions may require manual intervention.

**Opset too new**: if the model uses an opset version higher than 13, convert it to opset 11 in Python using `onnx.version_converter.convert_version(model, 11)` before importing.

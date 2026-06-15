---
title: "TensorFlow Import"
description: "Importing TensorFlow frozen graphs and SavedModels into SameDiff"
---

## TensorFlow Import

The `nd4j-samediff-import-tensorflow` module imports TensorFlow frozen graphs (`.pb` protobuf files) and TensorFlow SavedModels into SameDiff. After import, the model lives as a native SameDiff graph.

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-samediff-import-tensorflow</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

---

## Preparing Your TensorFlow Model

### Exporting a Frozen Graph from TF 1.x

```python
import tensorflow as tf
from tensorflow.python.framework.graph_util import convert_variables_to_constants

# Assuming you have a trained session
with tf.Session() as sess:
    # ... restore or train your model ...

    # Freeze: convert variables to constants
    frozen_graph_def = convert_variables_to_constants(
        sess,
        sess.graph_def,
        output_node_names=['output/Softmax']  # your output node name(s)
    )

    # Write to disk
    with tf.io.gfile.GFile('frozen_model.pb', 'wb') as f:
        f.write(frozen_graph_def.SerializeToString())
```

### Exporting a Frozen Graph from TF 2.x

```python
import tensorflow as tf

# Load a saved model or define your model
model = tf.saved_model.load('my_saved_model')

# Convert to concrete function
concrete_func = model.signatures['serving_default']

# Use tf2onnx or tf.lite.TFLiteConverter to export, OR
# use tf.compat.v1.graph_util to freeze

# Alternatively, save as SavedModel directory for direct import
model.save('my_saved_model_dir')
```

### Getting Node Names

To identify placeholder (input) and output node names in a frozen graph:

```python
import tensorflow as tf

graph_def = tf.compat.v1.GraphDef()
with open('frozen_model.pb', 'rb') as f:
    graph_def.ParseFromString(f.read())

for node in graph_def.node:
    print(node.op, node.name)
```

---

## Importing a Frozen Graph

Use `TFGraphMapper` to import a TF frozen protobuf file:

```java
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.imports.graphmapper.tf.TFGraphMapper;

import java.io.File;

// Import a frozen graph
File frozenGraph = new File("frozen_model.pb");
SameDiff sd = TFGraphMapper.importGraph(frozenGraph);
```

You can also import from an `InputStream`:

```java
import java.io.FileInputStream;
import java.io.InputStream;

try (InputStream is = new FileInputStream("frozen_model.pb")) {
    SameDiff sd = TFGraphMapper.importGraph(is);
}
```

---

## Placeholder Mapping

TF frozen graphs have placeholder nodes for inputs. After import, you must provide values for each placeholder at inference time. Use `sd.output()` with a map from placeholder name to `INDArray`:

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

import java.util.HashMap;
import java.util.Map;

// Create input matching the placeholder's expected shape
INDArray inputData = Nd4j.rand(1, 224, 224, 3);  // NHWC for an image model

Map<String, INDArray> placeholders = new HashMap<>();
placeholders.put("input_1", inputData);  // use your actual placeholder name

// Run inference; specify which output nodes to evaluate
Map<String, INDArray> result = sd.output(placeholders, "output/Softmax");

INDArray predictions = result.get("output/Softmax");
System.out.println("Predictions shape: " + java.util.Arrays.toString(predictions.shape()));
```

---

## Multiple Inputs and Outputs

For models with multiple inputs:

```java
Map<String, INDArray> inputs = new HashMap<>();
inputs.put("input_text",   Nd4j.create(new float[]{1, 5, 3, 0, 0}, new int[]{1, 5}));
inputs.put("input_length", Nd4j.create(new float[]{3}, new int[]{1}));

Map<String, INDArray> results = sd.output(inputs, "output_logits", "output_probs");
INDArray logits = results.get("output_logits");
INDArray probs  = results.get("output_probs");
```

---

## Inspecting the Imported Graph

After import, you can inspect the graph structure:

```java
// Print summary of all variables and ops
System.out.println(sd.summary());

// List all placeholder (input) variable names
for (SDVariable v : sd.variables()) {
    if (v.isPlaceHolder()) {
        System.out.println("Placeholder: " + v.name() + " shape: " + v.getShape());
    }
}

// List op names
for (String opName : sd.ops().keySet()) {
    System.out.println("Op: " + opName);
}
```

---

## Supported TF Ops

The following categories of TensorFlow ops are mapped:

**Arithmetic and math:**
- Add, AddV2, Sub, Mul, Div, FloorDiv, FloorMod, Pow, Exp, Log, Sqrt, Square, Abs, Neg, Sign, Ceil, Floor, Round
- MatMul, BatchMatMul, BatchMatMulV2
- Minimum, Maximum, BiasAdd

**Neural network:**
- Conv2D, Conv2DBackpropInput, DepthwiseConv2dNative, Conv3D
- Relu, Relu6, Elu, Selu, LeakyRelu, Sigmoid, Tanh, Softmax, LogSoftmax
- MaxPool, AvgPool, MaxPool3D, AvgPool3D, MaxPoolV2
- FusedBatchNorm, FusedBatchNormV2, FusedBatchNormV3

**Shape and indexing:**
- Reshape, Transpose, Concat, ConcatV2, Stack, Unstack, Pack, Unpack
- Split, SplitV, Slice, StridedSlice, GatherV2, GatherNd, ScatterNd
- Squeeze, ExpandDims, Tile, Pad, PadV2, MirrorPad

**Reduction:**
- Sum (ReduceSum), Mean (ReduceMean), Max (ReduceMax), Min (ReduceMin)
- ReduceAll, ReduceAny, ReduceProd

**Type and cast:**
- Cast, Identity, Snapshot

**Control flow (TF 1.x):**
- Switch, Merge, Enter, Exit, NextIteration

**Recurrent:**
- LSTMBlockCell, BlockLSTM, GRUBlockCell

**Image:**
- ResizeBilinear, ResizeNearestNeighbor, CropAndResize, DecodeJpeg (decode only)

**Lookup and embedding:**
- ResourceGather, GatherV2

---

## Complete Example: MobileNet Inference

```python
# Python: export a MobileNet frozen graph
import tensorflow as tf
import numpy as np

# Load MobileNet from Keras (requires TF 1.x-compatible freeze)
from keras.applications import MobileNet
from keras.backend import get_session
from tensorflow.python.framework.graph_util import convert_variables_to_constants

model = MobileNet(weights='imagenet')
sess = get_session()

frozen = convert_variables_to_constants(
    sess,
    sess.graph_def,
    output_node_names=['act_softmax/Softmax']
)
with open('mobilenet_frozen.pb', 'wb') as f:
    f.write(frozen.SerializeToString())
```

```java
// Java: load and run inference
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.imports.graphmapper.tf.TFGraphMapper;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

import java.io.File;
import java.util.*;

public class MobileNetInference {
    public static void main(String[] args) throws Exception {
        SameDiff sd = TFGraphMapper.importGraph(new File("mobilenet_frozen.pb"));

        // MobileNet expects [batch, height, width, channels] in [0, 1] range
        INDArray image = Nd4j.rand(1, 224, 224, 3);

        Map<String, INDArray> inputs = new HashMap<>();
        inputs.put("input_1", image);

        Map<String, INDArray> results = sd.output(inputs, "act_softmax/Softmax");
        INDArray predictions = results.get("act_softmax/Softmax");

        // Find top-1 class
        int topClass = predictions.argMax(1).getInt(0);
        System.out.println("Top class index: " + topClass);
    }
}
```

---

## Troubleshooting

**Op not found**: a TF op in the frozen graph has no registered SameDiff mapper. Check the op name in the error message, verify the op list above, and open a GitHub issue if it should be supported.

**Placeholder name unknown**: run the Python snippet above to print all node names before import. Use the exact string that appears after the `Placeholder` op type.

**Shape mismatch at inference**: TF models trained with NHWC data format (TensorFlow default) expect inputs as `[batch, height, width, channels]`. Construct your `INDArray` accordingly.

**Variables not frozen**: if the frozen graph still contains `Variable` ops, the freeze step was incomplete. Ensure `convert_variables_to_constants` ran successfully and all variable names were captured.

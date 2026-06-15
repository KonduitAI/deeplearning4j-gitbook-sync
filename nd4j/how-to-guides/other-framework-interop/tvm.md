---
title: "Apache TVM"
description: "Apache TVM 0.8 integration for optimized model inference"
---

## Apache TVM

The `nd4j-tvm` module integrates Apache TVM 0.8 for compiler-optimized model inference. TVM is a machine learning compiler that takes a model from a framework (PyTorch, TensorFlow, ONNX, MXNet, etc.), applies hardware-specific optimization passes (operator fusion, layout optimization, auto-tuning), and compiles it to a native binary for a target hardware backend.

The resulting compiled module is loaded in Java via the TVM runtime and `nd4j-tvm` bindings.

---

## What TVM Provides

TVM operates in two phases:

1. **Compile-time (Python)**: load a model, apply optimization passes, auto-tune for target hardware, and export a compiled module (`.so` shared library + metadata JSON)
2. **Runtime (Java)**: load the compiled module, provide input tensors, run the optimized kernel

The key benefit is that TVM can produce kernels significantly faster than the original runtime, especially on CPUs with specific instruction sets (AVX-512, ARM NEON) and on GPUs and NPUs.

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-tvm</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

The module includes the TVM 0.8 runtime library. The compilation step (Python) must be done separately.

---

## Step 1: Compile Your Model with TVM (Python)

TVM model compilation is done in Python using the `tvm` Python package.

### Install TVM

```bash
pip install apache-tvm==0.8.0
```

### Compile an ONNX Model

```python
import tvm
from tvm import relay
import tvm.relay.testing
import onnx
import numpy as np

# Load ONNX model
onnx_model = onnx.load("resnet50.onnx")

# Define input shape
input_name  = "input"
input_shape = (1, 3, 224, 224)
shape_dict  = {input_name: input_shape}

# Import into TVM Relay IR
mod, params = relay.frontend.from_onnx(onnx_model, shape_dict)

# Compile for CPU (x86 with AVX2)
target = tvm.target.Target("llvm -mcpu=core-avx2")
with tvm.transform.PassContext(opt_level=3):
    lib = relay.build(mod, target=target, params=params)

# Export the compiled module
lib.export_library("resnet50_tvm.so")

# Export graph JSON and params (needed at runtime)
import json
with open("resnet50_graph.json", "w") as f:
    json.dump(lib.get_graph_json(), f)
with open("resnet50_params.bin", "wb") as f:
    f.write(relay.save_param_dict(lib.get_params()))
```

### Compile a PyTorch Model

```python
import torch
import torchvision
import tvm
from tvm import relay

model = torchvision.models.resnet50(pretrained=True)
model.eval()

# Trace the model
input_shape = (1, 3, 224, 224)
scripted    = torch.jit.trace(model, torch.zeros(*input_shape))

# Import into TVM Relay
mod, params = relay.frontend.from_pytorch(scripted, [("input", input_shape)])

# Compile
target = tvm.target.Target("llvm -mcpu=core-avx2")
with tvm.transform.PassContext(opt_level=3):
    lib = relay.build(mod, target=target, params=params)

lib.export_library("resnet50_tvm.so")
```

### Auto-Tuning for Better Performance

Auto-tuning searches for the best kernel configurations for your hardware. This can take minutes to hours but typically improves inference speed significantly:

```python
import tvm
from tvm import relay, autotvm

# After importing the model into mod, params:
target = tvm.target.Target("llvm -mcpu=core-avx2")

# Define tuning tasks
tasks = autotvm.task.extract_from_program(mod["main"], target=target, params=params)

# Tune
tuner_log = "tuning_log.json"
for i, task in enumerate(tasks):
    tuner = autotvm.tuner.XGBTuner(task)
    tuner.tune(
        n_trial=200,
        measure_option=autotvm.measure_option(
            builder=autotvm.LocalBuilder(),
            runner=autotvm.LocalRunner(repeat=3, number=10, timeout=10)
        ),
        callbacks=[autotvm.callback.log_to_file(tuner_log)]
    )

# Compile with tuning records
with autotvm.apply_history_best(tuner_log):
    with tvm.transform.PassContext(opt_level=3):
        lib = relay.build(mod, target=target, params=params)

lib.export_library("resnet50_tvm_tuned.so")
```

---

## Step 2: Load and Run the Compiled Module in Java

```java
import org.nd4j.tvm.runner.TvmRunner;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

import java.util.LinkedHashMap;
import java.util.Map;

try (TvmRunner runner = TvmRunner.builder()
        .modelUri("resnet50_tvm.so")
        .build()) {

    // Input: [batch=1, channels=3, H=224, W=224]
    INDArray input = Nd4j.rand(1, 3, 224, 224)
            .castTo(org.nd4j.linalg.api.buffer.DataType.FLOAT);

    Map<String, INDArray> inputs = new LinkedHashMap<>();
    inputs.put("input", input);

    Map<String, INDArray> outputs = runner.exec(inputs);

    INDArray predictions = outputs.get("output");
    int topClass = predictions.argMax(1).getInt(0);
    System.out.println("Predicted class: " + topClass);
}
```

---

## TvmRunner Builder Options

```java
TvmRunner runner = TvmRunner.builder()
        .modelUri("path/to/compiled_model.so")  // compiled .so file (required)
        .build();
```

`TvmRunner` is `AutoCloseable`. Use try-with-resources to ensure native resources are released:

```java
try (TvmRunner runner = TvmRunner.builder().modelUri("model.so").build()) {
    // use runner
}
```

---

## Supported Target Backends

TVM can compile for many hardware targets. The target string is specified at compile time (Python):

| Hardware | TVM Target String |
|---|---|
| x86 CPU (generic) | `"llvm"` |
| x86 with AVX2 | `"llvm -mcpu=core-avx2"` |
| x86 with AVX-512 | `"llvm -mcpu=skylake-avx512"` |
| ARM CPU | `"llvm -device=arm_cpu -target=arm-linux-gnueabihf"` |
| ARM64 | `"llvm -device=arm_cpu -target=aarch64-linux-gnu"` |
| CUDA GPU | `"cuda"` |
| Vulkan | `"vulkan"` |
| Metal (macOS) | `"metal"` |
| WebAssembly | `"wasm"` |

Compile-time target selection is independent of the Java runtime module; the compiled `.so` is target-specific.

---

## Troubleshooting

**`UnsatisfiedLinkError` loading the .so**: ensure the compiled `.so` was built for the same target CPU/OS as where it is being loaded. Cross-compiled binaries (e.g., compiled on x86, loaded on ARM) will fail.

**Input name mismatch**: the input name passed to `runner.exec()` must match the name used during TVM compilation. If the name was `"input_1"` in the ONNX model, use `"input_1"` in the Java map.

**Performance not as expected**: if auto-tuning was not performed, performance may be similar to unoptimized execution. Run auto-tuning for the target hardware to see TVM's full benefit.

**TVM version mismatch**: the `nd4j-tvm` module bundles TVM 0.8 runtime libraries. A compiled module from a different TVM version may not load. Compile with TVM 0.8 to match.

---
title: Overview
short_title: Overview
description: Onnx interop Key features and brief samples.
category: Onnx
weight: 1
---

# Onnx

Nd4j allows execution of models via onnx runtime using nd4j's INDArray as a data structure. Leveraging the nd4j-onnxruntime interop is fairly simple. An onnx model can be loaded and executed as follows:

```java
        File f = new File ("path/to/your/model.onnx");
        INDArray x = Nd4j.scalar(1.0f).reshape(1,1);
        INDArray y = Nd4j.scalar(1.0f).reshape(1,1);
        OnnxRuntimeRunner onnxRuntimeRunner = OnnxRuntimeRunner.builder()
                .modelUri(f.getAbsolutePath())
                .build();
        Map<String,INDArray> inputs = new LinkedHashMap<>();
        inputs.put("x",x);
        inputs.put("y",y);
        Map<String, INDArray> exec = onnxRuntimeRunner.exec(inputs);
        INDArray z = exec.get("z");
```

This outputs a result with a map of output names to the ndarray result from onnx.

In maven, add the following dependency:

```markup
<dependency>
   <groupId>org.nd4j</groupId>
   <artifactId>nd4j-onnxruntime</groupId>
   <version>${nd4j.version}</version>
</dependency>
```

Note, this depends on javacpp's onnxruntime bindings. This means onnx runtime's native binaries are managed by javacpp. Javacpp will bundle all native binaries for all platforms by default unless you specify a platform. You can do this by specifying a -Dplatform=your-platform-of-choice. You may find more [here](https://github.com/bytedeco/javacpp-presets/wiki/Reducing-the-Number-of-Dependencies) in the javacpp docs.


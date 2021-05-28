---
title: Overview
short_title: Overview
description: TVM Key features and brief samples.
category: TVM
weight: 1
---


# TVM java interop

Nd4j allows execution of models via javacpp's tvm bindings using nd4j's INDArray as a data structure.
Leveraging the nd4j-tvm interop is fairly simple.


A TVM model can be loaded and executed as follows:

```java
        File f = new File ("path/to/your/model.pb");
        INDArray x = Nd4j.scalar(1.0f).reshape(1,1);
        TvmRunner tvmRunner = TvmRunner.builder()
                .modelUri(f.getAbsolutePath())
                .build();
        Map<String,INDArray> inputs = new LinkedHashMap<>();
        inputs.put("x",x);
        Map<String, INDArray> exec = tvmRunner.exec(inputs);
        INDArray z = exec.get("0");
```


This outputs a result with a map of output names to the ndarray result from tvm.



In maven, add the following dependency:
```xml
<dependency>
   <groupId>org.nd4j</groupId>
   <artifactId>nd4j-tvm</groupId>
   <version>${nd4j.version}</version>
</dependency>
```


Note, this depends on javacpp's tvm bindings. This means tvm's native binaries are managed by javacpp.
You can do this by specifying a -Dplatform=your-platform-of-choice. You may find more [here](https://github.com/bytedeco/javacpp-presets/wiki/Reducing-the-Number-of-Dependencies) in the javacpp docs.


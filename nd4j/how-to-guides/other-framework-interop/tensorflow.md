---
title: Overview
short_title: Overview
description: Tensorflow interop Key features and brief samples.
category: Tensorflow
weight: 1
---

# Tensorflow

Nd4j allows execution of models via javacpp's tensorflow bindings using nd4j's INDArray as a data structure. Leveraging the nd4j-tensorflow interop is fairly simple.

Note, this is based on tensorflow 1.x. TF java 2 will be coming at a later time.

A tensorflow model can be loaded and executed as follows:

```java
        File f = new File ("path/to/your/model.pb");
        INDArray x = Nd4j.scalar(1.0f).reshape(1,1);
        INDArray y = Nd4j.scalar(1.0f).reshape(1,1);
        List<String> inputNames = Arrays.asList("x","y");
        List<String> outputNames = Arrays.asList("z");
        GraphRunner tensorflowRunner = GraphRunner.builder()
                .inputNames(inputNames)
                .graphPath(f)
                .outputNames(outputNames).
                build();
        Map<String,INDArray> inputs = new LinkedHashMap<>();
        inputs.put("x",x);
        inputs.put("y",y);
        Map<String, INDArray> exec = tensorflowRunner.exec(inputs);
        INDArray z = exec.get("z");
```

This outputs a result with a map of output names to the ndarray result from tensorflow.

In maven, add the following dependency:

```markup
<dependency>
   <groupId>org.nd4j</groupId>
   <artifactId>nd4j-tensorflow</groupId>
   <version>${nd4j.version}</version>
</dependency>
```

Note, this depends on javacpp's tensorflow bindings. This means tensorflow's native binaries are managed by javacpp. Javacpp will bundle all native binaries for all platforms by default unless you specify a platform. You can do this by specifying a -Dplatform=your-platform-of-choice. You may find more [here](https://github.com/bytedeco/javacpp-presets/wiki/Reducing-the-Number-of-Dependencies) in the javacpp docs.


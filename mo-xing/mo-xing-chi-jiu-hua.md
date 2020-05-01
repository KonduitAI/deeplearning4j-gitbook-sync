---
description: 神经网络的存储与加载。
---

# 模型持久化

## 神经网络的存储与加载

MultiLayerNetwork 与 ComputationGraph 都有保存和加载方法。

可以使用以下命令保存/加载MultiLayerNetwork：

```java
MultiLayerNetwork net = ...
net.save(new File("...");

MultiLayerNetwork net2 = MultiLayerNetwork.load(new File("..."), true);
```

类似地，可以使用以下命令保存/加载ComputationGraph：

```java
ComputationGraph net = ...
net.save(new File("..."));

ComputationGraph net2 = ComputationGraph.load(new File("..."), true);
```

在内部这些方法使用`ModelSerializer`类，它是一个处理加载和保存模型的类。通过链接显示的示例中保存模型有两种方法。第一个例子保存了一个正常的多层网络，第二个例子保存了一个计算图。

下面是一个[基本示例](https://github.com/deeplearning4j/dl4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/modelsaving)，其中包含使用ModelSerializer类保存计算图的代码，以及使用ModelSerializer保存使用MultiLayer配置构建的神经网络的示例。

### RNG 种子

如果你的模型使用概率（即，DropOut/DropConnect），那么单独保存它并在恢复模型之后应用它可能是有意义的；即：

```java
 Nd4j.getRandom().setSeed(12345);
 ModelSerializer.restoreMultiLayerNetwork(modelFile);
```

这将保证会话/JVM之间相等的结果。


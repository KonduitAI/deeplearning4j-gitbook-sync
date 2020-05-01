---
description: 为自定义层扩展DL4J功能。
---

# 自定义层

## 编写自定义层

有两个组件可添加自定义层:

1. 添加层配置类: 扩展 org.deeplearning4j.nn.conf.layers.Layer
2. 添加层实现类: 实现 org.deeplearning4j.nn.api.Layer

配置层\(以上\(1\)\)类处理设置。这是你在构建多层网络或计算图时所使用的方法。你可以在这里添加自定义设置，并在你的图层中使用这些设置。

实现层\(以上\(2\)\)类具有参数，并处理网络前向传播、反向传播等。它是从org.deeplearning4j.nn.conf.layers.Layer.instantiate\(…\)方法创建的。换句话说：instanceiate方法是我们从配置到实现的方式；MultiLayerNetwork或ComputationGraph在初始化的时候调用。

其中的一个例子是CustomLayer（配置类）和CustomLayerImpl（实现类）。这两类都对它们的方法有广泛的注释。

你将注意到，在DL4J中有两个DenseLayer 类 、两个GravesLSTM类等：原因在于一个用于配置，一个用于实现。我们没有遵循这个“同名”模式，希望避免混淆。

## 测试自定义层

一旦添加了自定义层，就需要运行一些测试来确保它是正确的。

这些测试至少应包括以下内容：

1. 测试以确保JSON配置（到/从JSON）正常工作，这对于你的自定义层与模型序列化（保存）和Spark训练都起作用的网络来说是必要的。
2. 梯度检查，以确保执行是正确的。

## 示例

我们提供了一个完整的自定义层示例。在我们的 [示例仓库 ](https://github.com/deeplearning4j/dl4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/customlayers) 中。


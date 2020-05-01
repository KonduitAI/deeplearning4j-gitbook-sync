---
description: 为开箱即用应用程序预先构建的模型架构和权重。
---

# 动物园用法

## 关于DL4J模型动物园

DL4J具有可直接从DL4J访问和实例化的本地模型动物园。模型动物园还包括用于不同数据集的预训练权重，这些数据集是自动下载的，并使用校验总和机制检查完整性。

如果你想使用新的模型动物园，你需要添加它作为依赖项。Maven POM将添加以下内容：

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-zoo</artifactId>
    <version>1.0.0-beta6</version>
</dependency>
```

## 入门

一旦你成功地将动物园依赖项添加到项目中，就可以开始导入和使用模型。每一个模型扩展了`ZooModel`抽象类，并使用了`InstantiableModel`接口。这些类提供了帮助你初始化空的、新的网络或预先训练的网络的方法。

### 初始化新配置

你可以使用`.init()`方法立即从动物园实例化一个模型。例如，如果要实例化一个新的未经训练的AlexNet网络，可以使用以下代码：

```java
import org.deeplearning4j.zoo.model.AlexNet
import org.deeplearning4j.zoo.*;

...

int numberOfClassesInYourData = 1000;
int randomSeed = 123;
int iterations = 1; // 这几乎总是1。

ZooModel zooModel = new AlexNet(numberOfClassesInYourData, randomSeed, iterations);
Model net = zooModel.init();
```

如果希望调优参数或更改优化算法，可以获得对底层网络配置的引用：

```java
ZooModel zooModel = new AlexNet(numberOfClassesInYourData, randomSeed, iterations);
MultiLayerConfiguration net = zooModel.conf();
```

### 初始化预先训练的权值

一些模型具有可用的预训练权重，并且少量模型跨不同的数据集进行预训练。PretrainedType是概述不同权重类型的枚举器，包括IMAGENET、MNIST、CIFAR10和VGGFACE。

例如，你可以初始化VGG-16模型，使用ImageNet的权重，例如：

```java
import org.deeplearning4j.zoo.model.VGG16;
import org.deeplearning4j.zoo.*;

...

ZooModel zooModel = new VGG16();
Model net = zooModel.initPretrained(PretrainedType.IMAGENET);
```

并用VGGFace训练的权值初始化另一个VGG16模型：

```java
ZooModel zooModel = new VGG16();
Model net = zooModel.initPretrained(PretrainedType.VGGFACE);
```

如果不确定模型是否包含预训练权重，可以使用返回布尔值的.`pretrainedAvailable`\(\)方法。简单地将一个`PretrainedType`枚举传递给这个方法，如果权重可用，它将返回true。

注意，对于卷积模型，输入形状信息遵循NCHW约定。因此，如果模型的输入形状默认值是 `new int[]{3, 224, 224}，`则意味着该模型具有3个通道和高度/宽度为224。

## 动物园里有什么？

模型动物园带有在深入学习社区著名的图像识别配置。动物园还包括一个用于文本生成的LSTM，以及一个用于一般图像识别的简单CNN。

你可以使用这个[deeplearning4j-zoo](https://github.com/deeplearning4j/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model)找到一个完整的模型列表。

这包括ImageNet模型，如VGG-16, ResNet-50, AlexNet, Inception-ResNet-v1, LeNet等。

* [AlexNet](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/AlexNet.java)
* [Darknet19](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/Darknet19.java)
* [FaceNetNN4Small2](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/FaceNetNN4Small2.java)
* [InceptionResNetV1](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/InceptionResNetV1.java)
* [LeNet](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/LeNet.java)
* [ResNet50](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/ResNet50.java)
* [SimpleCNN](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/SimpleCNN.java)
* [TextGenerationLSTM](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/TextGenerationLSTM.java)
* [TinyYOLO](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/TinyYOLO.java)
* [VGG16](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/VGG16.java)
* [VGG19](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/VGG19.java)

## 高级用法

如果你想在不同的用例中使用这些模型，动物园还有几个附加的特征。

### 改变输入

除了将某些配置信息传递给动物园模型的构造函数之外，还可以使用.setInputShape\(\)更改其输入形状。注意：这只适用于新的配置，并且不会影响预先训练的模型：

```java
ZooModel zooModel = new ResNet50(10, 123, 1);
zooModel.setInputShape(new int[]{3,28,28});
```

### 迁移学习

预先训练的模型对于迁移学习来说是完美的！你可以在[这里](https://deeplearning4j.org/docs/latest/deeplearning4j-nn-transfer-learning)阅读更多关于迁移学习的内容。

### 工作间

初始化方法通常有一个名为“`workspaceMode`”的附加参数。对于大多数用户来说，你不需要使用它；但是，如果你有一台具有“健壮”规范的大型机器，则可以为具有数百万参数的模型传递`WorkspaceMode.SINGLE` ，如VGG-19。要了解更多关于工作间的内容，请参阅[本节](https://deeplearning4j.org/docs/latest/deeplearning4j-config-workspaces)。


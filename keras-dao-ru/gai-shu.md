---
description: 模型导入概述
---

# 概述

## DL4J: Keras模型导入

[Keras模型导入](https://github.com/deeplearning4j/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras)为导入最初用[Keras](https://keras.io/)配置和训练的神经网络模型提供了例程，Keras是一个流行的Python深度学习库。

一旦你的模型导入到DL4J，我们的整个生产栈是由你来处理的。我们支持导入所有的Keras模型类型、大多数层和几乎所有的实用功能。请在[这里](zhi-chi-gong-neng/)查看支持的Keras特性的完整列表。

## 入门：在60秒内导入一个Keras模型

要导入Keras模型，首先需要创建和[序列化](https://keras.io/getting-started/faq/#how-can-i-save-a-keras-model)这样的模型。这里有一个你可以使用的简单例子。该模型是一个简单的MLP，它采用长度为100的小批量向量，具有两个密集层，并预测总共10个类别。在定义模型之后，我们将其序列化为HDF5格式。

```python
from keras.models import Sequential
from keras.layers import Dense

model = Sequential()
model.add(Dense(units=64, activation='relu', input_dim=100))
model.add(Dense(units=10, activation='softmax'))
model.compile(loss='categorical_crossentropy',optimizer='sgd', metrics=['accuracy'])

model.save('simple_mlp.h5')
```

如果将这个模型文件（`simple_mlp.h5`）放到项目的资源文件夹的根目录中，则可以将Keras模型加载为DL4J 的 MultiLayerNetwork，如下所示

```java
String simpleMlp = new ClassPathResource("simple_mlp.h5").getFile().getPath();
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(simpleMlp);
```

就是这样！KerasModelImport是模型导入的主要入口点，类负责在内部将Keras映射到DL4J概念。作为用户，你只需要提供你的模型文件，请参阅我们的[入门指南](ru-men.md)，以了解将Keras模型加载到DL4J中的更多细节和选项。

现在可以使用导入的模型进行推断（这里使用简单的数据来简化）

```java
INDArray input = Nd4j.create(256, 100);
INDArray output = model.output(input);
```

以下是你如何在DL4J中为你导入的模型做训练：

```java
model.fit(input, output);
```

在我们的[DL4J示例](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/modelimport/keras/basic/SimpleSequentialMlpImport.java)中可以找到刚才所示的完整示例。

## 项目设置

要在现有项目中使用Keras模型导入，只需要将下列依赖项添加到pom.xml中。

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-modelimport</artifactId>
    <version>1.0.0-beta</version> // 此版本应与其他DL4J项目依赖项匹配。
</dependency>
```

如果首先需要开始一个项目，请考虑克隆[DL4J示例](https://github.com/deeplearning4j/dl4j-examples)，并按照仓库中的说明来构建项目。

## 后端

DL4J Keras 模型导入与后端无关。 不管你选择哪一个后端 \(TensorFlow, Theano, CNTK\), 你的模号可以导入DL4J。

## 流行的模型与应用

我们支持为越来越多的应用程序导入，在[这里](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/test/java/org/deeplearning4j/nn/modelimport/keras/e2e/KerasModelEndToEndTest.java)查看当前所覆盖的模型的完整列表。这些应用包括

* Deep convolutional and Wasserstein GANs
* UNET
* ResNet50
* SqueezeNet
* MobileNet
* Inception
* Xception

## 故障排除与支持

`IncompatibleKerasConfigurationException`信息说明你正在尝试导入一个当前不被DL4J支持的Keras模型（要么因为模型导入不覆盖它，要么DL4J不实现该层或特征）。

一旦导入了模型，我们就推荐我们自己的“`ModelSerializer`”类来进一步保存和重新加载模型。

你可以通过访问[DL4J gitter](https://gitter.im/deeplearning4j/deeplearning4j)频道进一步咨询。你可能会考虑[通过Github来提交一个特性请求](https://github.com/deeplearning4j/deeplearning4j/issues)，这样这个缺失的功能可以放在DL4J开发路线图上，或者甚至向我们发送一个带有必要更改的pull请求！

## 为什么需要Keras模型导入?

Keras是用Python编写的一个流行的、用户友好的深度学习库。Keras的直观API使得Python轻松地定义和运行你的深度学习模型。Keras允许你选择它在哪个底层库上运行，但是为每个这样的后端提供了统一的API。目前，Keras支持Tensorflow、CNTK和Theano后端，但是Skymind也在为Keras开发ND4J后端。

一个公司的生产系统和它的数据科学家的实验设置之间经常有差距。Keras模型导入允许数据科学家用Python编写他们的模型，但是仍然与生产栈无缝集成。

Keras模型导入主要针对在Python中熟悉用Keras编写模型的用户。通过模型导入，你可以通过允许用户将模型导入DL4J生态圈以进行进一步的训练或评估，从而将Python模型带到生产中。

当项目的试验阶段完成并且需要将模型交付生产时，你应该使用这个模块。Skymind商业支持Keras在企业中的实现。


---
description: 导入functional模型
---

# Functional模型

## 导入Keras函数模型入门

假设你使用Keras的函数API开始定义一个简单的MLP：

```java
from keras.models import Model
from keras.layers import Dense, Input

inputs = Input(shape=(100,))
x = Dense(64, activation='relu')(inputs)
predictions = Dense(10, activation='softmax')(x)
model = Model(inputs=inputs, outputs=predictions)
model.compile(loss='categorical_crossentropy',optimizer='sgd', metrics=['accuracy'])
```

在Keras，有几种保存模型的方法。你可以将整个模型（模型定义、权重和训练配置）存储为HDF5文件，仅存储模型配置（作为JSON或YAML文件）或仅存储权重（作为HDF5文件）。以下是你如何做每一件事：

```java
model.save('full_model.h5')  # save everything in HDF5 format

model_json = model.to_json()  # save just the config. replace with "to_yaml" for YAML serialization
with open("model_config.json", "w") as f:
    f.write(model_json)

model.save_weights('model_weights.h5') # save just the weights.
```

如果你决定保存完整的模型，那么你将能够访问模型的训练配置，否则你将不访问。因此，如果你想在导入之后在DL4J中进一步训练模型，请记住这一点，并使用model.save\(...\)来持久化你的模型。

## 载加你的Keras模型

让我们从推荐的方法开始，将完整模型加载回DL4J（我们假设它在类路径上）：

```java
String fullModel = new ClassPathResource("full_model.h5").getFile().getPath();
ComputationGraph model = KerasModelImport.importKerasModelAndWeights(fullModel);
```

万一你没有编译你的Keras模型，它就不会有一个训练配置。在这种情况下，你需要显式地告诉模型导入忽略训练配置，方法是将enforceTrainingConfig标志设置为false，如下所示：

```java
ComputationGraph model = KerasModelImport.importKerasModelAndWeights(fullModel, false);
```

若要仅从JSON加载模型配置，请按如下使用`KerasModelImport`

```java
String modelJson = new ClassPathResource("model_config.json").getFile().getPath();
ComputationGraphConfiguration modelConfig = KerasModelImport.importKerasModelConfiguration(modelJson)
```

如果另外你还想加载模型权重与配置，那么以下是你要做的：

```java
String modelWeights = new ClassPathResource("model_weights.h5").getFile().getPath();
MultiLayerNetwork network = KerasModelImport.importKerasModelAndWeights(modelJson, modelWeights)
```

在后面两种情况下，将不读取训练配置。

#### KerasModel

[\[源码\]](https://github.com/deeplearning4j/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras//KerasModel.java)

从Keras（函数API）模型或序列模型配置构建计算图。

KerasModel

```java
public KerasModel(KerasModelBuilder modelBuilder)
            throws UnsupportedKerasConfigurationException, IOException, InvalidKerasConfigurationException
```

（建议）（函数API）模型的构建器模式构造器。

* 参数 modelBuilder 构建器对象
* 抛出 IOException IO 异常
* 抛出 InvalidKerasConfigurationException 无效的 Keras 配置
* 抛出 UnsupportedKerasConfigurationException 不支持的 Keras 配置

getComputationGraphConfiguration

```java
public ComputationGraphConfiguration getComputationGraphConfiguration()
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

（不推荐）来自模型配置（JSON或YAML）、训练配置（JSON）、权重和“训练模式”布尔指示符的（函数 API）模型的构造器。当内置在训练模式时，某些不支持的配置（例如，未知的正则化器）将抛出异常。当强制TrainingConfig= false时，这些将生成警告，但将被忽略。

* 参数 modelJson 模型配置JSON  字符串
* 参数 modelYaml 模型配置 YAML 字符串
* 参数 enforceTrainingConfig 是否实施训练相关配置
* 抛出 IOException IO 异常
* 抛出 InvalidKerasConfigurationException 无效的 Keras 配置
* 抛出 UnsupportedKerasConfigurationException 不支持的 Keras 配置

getComputationGraph

```java
public ComputationGraph getComputationGraph()
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

从这个Keras模型配置构建计算图并导入权重。

* 返回 ComputationGraph

getComputationGraph

```java
public ComputationGraph getComputationGraph(boolean importWeights)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

从这个Keras模型配置构建计算图并（可选的）导入权重。

* 参数 importWeights 是否导入权重
* 返回 ComputationGraph


---
description: 导入functional模型。
---

# Sequential模型

### 导入Keras序列模型入门

假设你使用Keras开始定义一个简单的MLP：

```java
from keras.models import Sequential
from keras.layers import Dense

model = Sequential()
model.add(Dense(units=64, activation='relu', input_dim=100))
model.add(Dense(units=10, activation='softmax'))
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

### 载加你的Keras模型

让我们从推荐的方法开始，将完整模型加载回DL4J（我们假设它在类路径上）：

```java
String fullModel = new ClassPathResource("full_model.h5").getFile().getPath();
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(fullModel);
```

万一你没有编译你的Keras模型，它就不会有一个训练配置。在这种情况下，你需要显式地告诉模型导入忽略训练配置，方法是将enforceTrainingConfig标志设置为false，如下所示：

```java
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights(fullModel, false);
```

若要仅从JSON加载模型配置，请按如下使用`KerasModelImport`

```java
String modelJson = new ClassPathResource("model_config.json").getFile().getPath();
MultiLayerNetworkConfiguration modelConfig = KerasModelImport.importKerasSequentialConfiguration(modelJson)
```

如果另外你还想加载模型权重与配置，那么以下是你要做的：

```java
String modelWeights = new ClassPathResource("model_weights.h5").getFile().getPath();
MultiLayerNetwork network = KerasModelImport.importKerasSequentialModelAndWeights(modelJson, modelWeights)
```

在后面两种情况下，将不读取训练配置。


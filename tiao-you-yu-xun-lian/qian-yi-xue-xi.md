# 迁移学习

### DL4J迁移学习API <a id="dl4js-transfer-learning-api"></a>

DL4J迁移学习API使用户能够：

* 修改现有模型的结构
* 调优现有模型的学习配置。
* 在训练期间保持指定层参数为常量（恒定不变），也称为“冻结”。

将某些层冻结在网络上，并且训练实际上与对输入的转换版本的训练相同，转换版本是冻结层边界处的中间输出。这是从输入数据中“特征提取”的过程，在本文档中称为“特征化”。

### 迁移学习帮手 <a id="the-transfer-learning-helper"></a>

在大型的、预训练的网络上，对输入数据进行“特征化”的前向传递可能很耗时。DL4J还提供了一个具有以下功能的TransferLearningHelper类。

* 对输入数据集进行特征化保存以备将来使用
* 用特征化数据集拟合冻结层模型
* 给定一个特征化输入，从冻结层的模型输出。

当运行多个epoch时，用户将节省计算时间，因为对冻结层/顶点的昂贵传递将只需进行一次。

### 给我看代码 <a id="show-me-the-code"></a>

本例将使用VGG16对属于五种花卉的图像进行分类。数据集将自动从http://download.tensorflow.org/example\_images/flower\_photos.tgz下载

### I. 导入动物园模型

在0.9.0（0.8-1快照）下，DL4J有一个新的本地模型动物园。阅读有关[deeplearning4j-zoo](https://deeplearning4j.org/model-zoo) 模块的更多信息来了解如何使用预训练模型。这里，我们加载用ImageNet上训练的权重初始化的预训练VGG-16模型：

```java
ZooModel zooModel = new VGG16();
ComputationGraph pretrainedNet = (ComputationGraph) zooModel.initPretrained(PretrainedType.IMAGENET);
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### II. 设置一个调优配置

```java
FineTuneConfiguration fineTuneConf = new FineTuneConfiguration.Builder()
            .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
            .updater(new Nesterovs(5e-5))
            .seed(seed)
            .build();
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### III. 基于VGG16构建新模型

#### A.只修改最后一层，保持其他冻结

VGG16的最后一层对ImageNet.中的1000个类进行软最大回归。我们修改最后一层，给出五个类的预测，保持其他层的冻结。

```java
ComputationGraph vgg16Transfer = new TransferLearning.GraphBuilder(pretrainedNet)
    .fineTuneConfiguration(fineTuneConf)
              .setFeatureExtractor("fc2")
              .removeVertexKeepConnections("predictions") 
              .addLayer("predictions", 
        new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
                        .nIn(4096).nOut(numClasses)
                        .weightInit(WeightInit.XAVIER)
                        .activation(Activation.SOFTMAX).build(), "fc2")
              .build();
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

在仅仅30次迭代（在这种情况下是曝光450幅图像）之后，该模型在测试数据集上达到了&gt;75%的准确率。考虑到从头训练图像分类器的复杂性，这相当显著。

#### B.连接新层到瓶颈（block5\_pool）

在这里，我们保持除了最后三个密连层之外的所有密连层冻结，并将新的密连层附加到其上。请注意，本文的主要目的是演示API的使用，其次可能提供更好结果。

```java
ComputationGraph vgg16Transfer = new TransferLearning.GraphBuilder(pretrainedNet)
              .fineTuneConfiguration(fineTuneConf)
              .setFeatureExtractor("block5_pool")
              .nOutReplace("fc2",1024, WeightInit.XAVIER)
              .removeVertexAndConnections("predictions") 
              .addLayer("fc3",new DenseLayer.Builder()
         .activation(Activation.RELU)
         .nIn(1024).nOut(256).build(),"fc2") 
              .addLayer("newpredictions",new OutputLayer
        .Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
                                .activation(Activation.SOFTMAX)
                                .nIn(256).nOut(numClasses).build(),"fc3") 
            .setOutputs("newpredictions") 
            .build();
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

#### C. 从先前保存的模型中调优图层

假设我们已经从\(B\)中保存了模型，现在想要允许“block\_5”层进行训练。

```java
ComputationGraph vgg16FineTune = new TransferLearning.GraphBuilder(vgg16Transfer)
              .fineTuneConfiguration(fineTuneConf)
              .setFeatureExtractor(“block4_pool”)
              .build();
```

### IV. 保存“特征化”数据集并与之进行训练。

我们使用迁移学习助手API。注意，这冻结了传入的模型的层。  
以下是如何在指定层“fc2”获得数据集的特征版本。

```java
TransferLearningHelper transferLearningHelper = 
    new TransferLearningHelper(pretrainedNet, "fc2");
while(trainIter.hasNext()) {
        DataSet currentFeaturized = transferLearningHelper.featurize(trainIter.next());
        saveToDisk(currentFeaturized,trainDataSaved,true);
  trainDataSaved++;
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

下面是如何与特征数据集拟合的方法。vgg16Transfer 是第三部分（a）中设置的一种模型。

```java
TransferLearningHelper transferLearningHelper = 
    new TransferLearningHelper(vgg16Transfer);
while (trainIter.hasNext()) {
       transferLearningHelper.fitFeaturized(trainIter.next());
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 注意 <a id="notes"></a>

* 迁移学习构建器返回一个DL4J模型的新实例。

请记住，这是第二个模型，使原件保持原状。对于大型的预训练网络，要考虑内存需求，并相应地调整JVM堆空间。

* 受过训练的模型助手从Keras导入模型而不强制执行训练配置。

因此，最后一层（如打印摘要时所见）是密连层，而不是具有损失函数的输出层。因此，为了修改输出层的nOut，我们删除了层顶点，保持了它的连接，并添加回具有相同名称、不同nOut、合适的损失函数等的新输出层。

* 在层/顶点处更改nOut将修改它的层/顶点的nIn。

当改变nOut时，用户可以指定层的权重初始化方案或分布，以及它的层的单独的权重初始化方案或分布。

* 当将模型写入磁盘时，不保存冻结层配置。

换句话说，在序列化和读回时具有冻结层的模型将不具有任何冻结层。为了持续训练以保持特定层常量，用户需要通过迁移学习助手或迁移学习API。在DL4J模型中有两种方法来“冻结”层。

* On a copy: 使用迁学习API，它将返回一个包含相关冻结层的新模型
* In place: 使用迁移学习帮助API将冻结层应用于给定模型。
* 调优配置将有选择地更新学习参数。

例如，如果指定了学习速率，则该学习速率将应用于模型中的所有未冻结/可训练层。然而，新添加的层可以通过在层构建器中指定它们自己的学习速率来覆盖此学习速率。

### 实用类 <a id="utilities"></a>


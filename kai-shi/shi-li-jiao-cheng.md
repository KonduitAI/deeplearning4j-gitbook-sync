---
description: 简要介绍DL4J中的可用示例。
---

# 示例教程

### DL4J示例调研

DL4J的Github仓库有很多示例可以涵盖它的功能。快速入门向你展示了如何设置Intellij并克隆仓库。本页提供这些例子中的一些概述。

### DataVec 示例

大多数示例都使用DataVec，这是一个通过归一化，标准化，搜索和替换，列洗牌和向量化 预处理和清洗数据 的工具包。为神经网络读取原始数据并将其转换为DataSet对象通常是训练该网络的第一步。如果你不熟悉DataVec，这里有一个描述和一些有用的例子的链接。

#### IrisAnalysis.java

本例采用同名花卉品种的标准Iris数据集，其相关测量值是萼片长度、萼片宽度、花瓣长度和花瓣宽度。它从相对较小的数据集构建一个Spark RDD，并对其进行分析。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/datavec-examples/src/main/java/org/datavec/transform/analysis/IrisAnalysis.java)

#### BasicDataVecExample.java

加载数据到一个Spark RDD的示例。所有 DataVec 转换操作都使用Spark RDD。这里我们使用DataVec来过滤数据，应用时间转换和删除列。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/datavec-examples/src/main/java/org/datavec/transform/basic/BasicDataVecExample.java)

#### PrintSchemasAtEachStep.java

这个例子显示了打印概要工具，这些工具对于可视化和确保转换的代码行为符合预期是有用的。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/datavec-examples/src/main/java/org/datavec/transform/debugging/PrintSchemasAtEachStep.java)

#### JoinExample.java

在传递传递数据集到一个神经网络之前，你可能需要连接数据集。你可以用DataVec实现，这个例子告诉你如何实现。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/datavec-examples/src/main/java/org/datavec/transform/join/JoinExample.java)

#### LogDataExample.java

这是使用DataVec解析日志数据的示例。明显的使用场景是是网络安全和客户关系管理。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/datavec-examples/src/main/java/org/datavec/transform/logdata/LogDataExample.java)

#### MnistImagePipelineExample.java

这个例子是从下面的视频中演示的，它展示了ParentPathLabelGenerator和ImagePreProcessing缩放器。

{% embed url="https://youtu.be/GLC8CIoHDnI" %}

[显示代码](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/dataexamples/MnistImagePipelineExample.java)

#### PreprocessNormalizerExample.java

此示例演示了DataVec中可用的预处理特征。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/dataexamples/PreprocessNormalizerExample.java)

#### CSVExampleEvaluationMetaData.java

DataMeta数据跟踪——即查看每个示例的数据来自何处——在跟踪导致错误和其他问题的格式错误的数据时非常有用。此示例演示RecordMetaData类中的功能。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/dataexamples/CSVExampleEvaluationMetaData.java)

### DeepLearning4J 示例

为了构建一个神经网络，你将使用多层网络或计算图。这两种选项都使用Builder接口工作。下面描述了几个例子中的亮点。

#### MNIST dataset of handwritten digits

MNIST是深度学习的入门。简单，直接，重心在于图像识别，神经网络做得很好的任务。

#### MLPMnistSingleLayerExample.java

这是一个用于识别数字的单层感知器。请注意，这会从包含数据集的二进制包中提取图像，这是数据提取的一种特殊情况。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/feedforward/mnist/MLPMnistSingleLayerExample.java)

#### MLPMnistTwoLayerExample.java

MNIST的两层感知器，展示对于一个给定的数据集有不止一个有用的网络。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/feedforward/mnist/MLPMnistTwoLayerExample.java)

#### Feedforward Examples

通过前馈神经网络的数据流， 通过隐藏层从输入到输出的单向传递。

这些网络可用于各种各样的任务，这取决于它们被配置。除了对MNIST数据进行图像分类之外，这个目录还有演示回归、分类和异常检测的示例。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/feedforward)

#### Convolutional Neural Networks

卷积神经网络主要用于图像识别，虽然它们也适用于声音和文本。

#### AnimalsClassification.java

此示例可以使用LeNet或AlexNet运行。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/convolution/AnimalsClassification.java)

### 保存和加载模型

在大量训练数据上训练网络需要时间。幸运的是，你可以保存一个经过训练的模型并加载模型以供以后的训练或推断。

#### SaveLoadComputationGraph.java

演示了保存和加载用计算图构建的神经网络。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/modelsaving/SaveLoadComputationGraph.java)

#### SaveLoadMultiLayerNetwork.java

演示如何保存和加载用多层网络构建的神经网络。

#### 保存／加载一个已训练的模型并传入一个新的输入

我们的视频系列显示了包括保存和加载模型以及推理的代码。

[我们的YouTube栏目](https://www.youtube.com/channel/UCa-HKBJwkfzs4AgZtdUuBXQ)

### 定制损失函数和层

你需要添加一个不可用的或是没有预构建的损失函数吗？查看这些示例。

#### CustomLossExample.java

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/lossfunctions/CustomLossExample.java)

#### CustomLossL1L2.java

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/lossfunctions/CustomLossL1L2.java)

#### Custom Layer

你需要添加一个在DL4J中没有的特征的层吗？这个例子展示了如何开始。

#### CustomLayerExample.java

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/customlayers/CustomLayerExample.java)

### 自然语言处理

我们也有自己的自然语言处理神经网络

#### GloVe

用于词表示的全局向量对于检测词之间的关系是有用的。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/glove/GloVeExample.java)

#### Paragraph Vectors

单词的矢量化表示。[这里](https://cs.stanford.edu/~quocle/paragraph_vector.pdf)有描述。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/paragraphvectors/ParagraphVectorsClassifierExample.java)

#### Sequence Vectors

表示句子的一种方式是单词序列。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/sequencevectors/SequenceVectorsTextExample.java)

#### Word2Vec

[这里](../yu-yan-chu-li/word2vec.md)有描述

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/word2vec/Word2VecRawTextExample.java)

### 数据可视化

T-分布随机相邻嵌入（T-SNE）对于数据可视化是有用的。我们在NLP部分包括一个例子，因为词相似性可视化是一种常用的用法。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/tsne/TSNEStandardExample.java)

### 循环神经网络

循环神经网络可用于处理时间序列数据或其他顺序馈送的数据，如视频。循环神经网络的示例文件夹有以下内容：

#### BasicRNNExample.java

学习字符串的循环神经网络。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/basic/BasicRNNExample.java)

### LSTMCharModellingExample.java

将莎士比亚的全部作品作为字符序列，并训练神经网络逐字创作“莎士比亚”。

[显示代码](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/character/LSTMCharModellingExample.java)



#### AdditionRNN.java

这个例子训练一个神经网络做加法。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/seq2seq/AdditionRNN.java)

#### RegressionMathFunctions.java

这个例子训练一个神经网络来执行各种数学运算。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/feedforward/regression/RegressionMathFunctions.java)

#### UCISequenceClassificationExample.java

一个公开的六类时间序列数据集，循环，向上等，例如，一个学习分类时间序列的RNN示例。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/seqclassification/UCISequenceClassificationExample.java)

#### VideoClassificationExample.java

自动驾驶车辆如何区分行人、停车标志和绿灯？在一组视频上训练卷积和循环层的复杂神经网络。实况车载视频传递给训练好的网络，并且基于来自神经网络的目标检测的决策确定车辆动作。

这个例子类似，但简化了。它结合卷积、最大池、密连（前馈）和循环（LSTM）层来对视频中的帧进行分类。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/video/VideoClassificationExample.java)

#### SentimentExampleIterator.java

该情感分析实例使用词向量和循环神经网络将情感分类为正或负。

[显示代码](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/word2vecsentiment/Word2VecSentimentRNN.java)

### 在Spark上分布式训练

DL4J支持使用Spark集群进行网络训练。这里是例子。

#### MnistMLPExample.java

这是一个多层感知机对手写数字的MNIST数据集进行训练的例子。

[显示代码](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-spark-examples/dl4j-spark/src/main/java/org/deeplearning4j/legacyExamples/mlp/MnistMLPExample.java)

#### SparkLSTMCharacterExample.java

在Spark上的一个LSTM循环网络

[显示代码](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-spark-examples/dl4j-spark/src/main/java/org/deeplearning4j/legacyExamples/rnn/SparkLSTMCharacterExample.java)

### ND4J 示例

ND4J是张量处理库。它可以被认为是JVM版的Numpy。神经网络通过处理和更新多维数值数组来工作。在典型的神经网络应用程序中，您使用DataVec提取并转换数据为数字。使用的类是RecordReader。一旦需要将数据传递到神经网络，通常使用RecordReaderDataSetIterator。RecordReaderDataSetIterator返回DataSet对象。DataSet由输入特征和标签的NDArray组成。

学习算法和损失函数作为ND4J操作被执行。

#### 基本的ND4J示例

这是一个带有创建和操作NDArrays示例的目录

[显示代码](https://github.com/deeplearning4j/dl4j-examples/tree/master/nd4j-examples/src/main/java/org/nd4j/examples)

### 强化学习示例

深度学习算法已经学会了使用强化学习来玩《Space Invaders》和《Doom》。DL4J/RL4J强化学习的例子在这里可用：

[显示代码](https://github.com/deeplearning4j/dl4j-examples/tree/master/rl4j-examples)




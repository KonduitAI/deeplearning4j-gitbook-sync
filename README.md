---
title: Core Concepts in Deeplearning4j
short_title: Core Concepts
description: >-
  本页将向您全面介绍如何运行DL4J示例，启动您自己的项目。建议您加入我们的QQ交流群。您可以在QQ上请求帮助、提出反馈，不过也请您在遇到问题时先参考本指南中已列出的疑难解答。如果您是初次接触深度学习，我们准备了一份初学者学习计划，包括课程、阅读材料和其他资源的链接。
category: Get Started
weight: 1
---

# 核心概念

{% hint style="info" %}
由于项目的进展速度超出了我们翻译文档的速度，因此请同时参考[英文文档](https://deeplearning4j.konduit.ai/)。
{% endhint %}

## 概述

每一个机器学习工作流至少由两部分组成。第一部份是加载你的数据并准备它用于学习。我们将此部分称为ETL（提取、转换、加载）过程。DataVec是我们为让构建数据管道更容易而构建的库。第二部分是实际的学习系统本身。这是DL4J的算法核心。

所有的深层学习都是基于向量和张量的，DL4J依赖于一个叫做ND4J的张量库。它为我们提供了处理n维数组（也叫做张量）的能力。由于其不同的后端，它甚至使我们能够同时使用CPU和GPU。

## 为学习和预测准备数据

与其他机器学习或深度学习框架不同，DL4J将加载数据和训练算法的任务视为单独的过程。你不只是把模型指向在磁盘上保存的数据，而是使用DataVec加载数据。这为你提供了更大的灵活性，并保留了简单数据加载的便利性。

在算法开始学习之前，你必须准备好数据，即使你已经有了一个经过训练的模型。准备数据意味着加载数据并将其置于正确的形状和值范围（例如，归一化、零均值和单位方差）。从头开始构建这些过程是容易出错的，因此尽可能使用DataVec。

Deeplearning4j可以处理许多不同的数据类型，比如图像、CSV、ARFF、纯文本，并且通过Apache Camel集成，可以处理几乎任何其它可以想到的数据类型。

要使用DataVec，需要连同RecordReaderDataSetIterator一起使用一个RecordReader接口的实现。

一旦你有了一个DataSetIterator, 它是一个描述顺序访问数据的模式，你可以使用它得到适合训练神经网络模型的格式的数据。

## 归一化数据

当它们被馈送的数据被归一化时，神经网络工作得最好，数据被限制在1到1之间。这样做有几个原因。一个是使用梯度下降训练网络，并且它们的激活函数通常在-1和1之间的某个范围。即使使用不会很快饱和的激活函数，将你的值限制到这个范围以提高性能仍然是很好的实践。

在DL4J中归一化数据相当简单，取决于你想要怎样归一化你的数据，并为你的DataSetIterator设置相关的DataNormalization作为预处理器。

ImagePreProcessingScaler显然是图像数据的不错的选择。如果你在输入数据的所有维度上具有统一的范围，那么NormalizerMinMaxScaler是一个不错的选择，并且NormalizerStandardize是你在其他情况下通常使用的工具。

如果你需要其他类型的归一化，你也可以自由地实现DataNormalization接口。

如果你使用NormalizerStandardize，请注意这是一个取决于从数据中提取的统计信息的一个归一化器，所以你必须同模型一起保存这些统计信息，以便你可以在恢复模型时恢复它们。

## DataSets, INDArrays 和 Mini-Batches

顾名思义，DataSetIterator会返回DataSet对象。DataSet对象是数据的特征和标签的容器。但它们并不局限于一次只持有一个实例。数据集可以包含需要的多个实例。

它通过在几个INDArray实例中保存这些值：一个用于实例的特性，一个用于标签，以及另外两个用于屏蔽，如果你正在使用时间序列数据（参见使用RNN/Masking，了解更多信息）。

INDArray是ND4J中使用的n维数组或张量之一。在特征的情况下，它是实例大小数x特征数量的矩阵。即使只有一个实例，它也会有这种形状。

为什么它不同时包含所有的数据实例？

这是深入学习的另一个重要概念：迷你批处理。为了产生准确的结果，经常需要大量真实世界的训练数据。通常，这是比在可用内存中拟合的数据更多，所以有时将其存储在单个数据集中是不可能的。但是，即使有足够的数据存储，还有一个重要的原因不立即使用所有的数据。那就是使用小批量，你可以在一次训练中获得更多的更新。

那么，为什么要在数据集中有一个以上的例子呢？由于模型是使用梯度下降训练，它需要一个良好的梯度学习如何最小化误差。一次只使用一个示例将创建只考虑当前示例产生的错误的梯度。这会使学习行为不稳定，减慢学习，甚至可能导致不可用的结果。

一个小批量应该足够大，以提供真实世界（或者至少是你的数据）的代表性样本。这意味着它应该始终包含你想要预测的所有类，并且这些类的计数应该以与总体数据中的类分布大致相同。

## 构建一个神经网络模型

DL4J为数据科学家和开发人员提供了工具，用于在一个高层使用的概念上构建一个深度神经网络，例如 layer。它使用一个构建器模式来声明性地构建神经网络，正如您在这个（简化的）示例中可以看到的：

```java
MultiLayerConfiguration conf = 
    new NeuralNetConfiguration.Builder()
        .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
        .updater(new Nesterovs(learningRate, 0.9))
        .list(
            new DenseLayer.Builder().nIn(numInputs).nOut(numHiddenNodes).activation("relu").build(),
            new OutputLayer.Builder(LossFunction.NEGATIVELOGLIKELIHOOD).activation("softmax").nIn(numHiddenNodes).nOut(numOutputs).build()
        ).backprop(true).build();
```

如果你熟悉其他的深度学习框架，你会注意到这有点像Keras。

与其他框架不同，DL4J从更新器算法中分离优化算法。这允许灵活性，当你寻求一个优化器与更新器来为你的数据和问题最好的工作。

除了上面示例中看到的DenseLayer和OutputLayer之外，还有其他几种层类型，如GravesLSTM、卷积层、RBM、EmbeddingLayer等。使用这些层，你不仅可以定义简单的神经网络，还可以定义递归和卷积网络。

## 训练一个模型

在你配置你的神经网络之后，你必须训练你的模型。最简单的情况是在模型上简单的调用.fit\(\)方法，并用你的DataSetIterator作为配置参数。这将在你所有的数据上一次性训练你的模型。在整个数据集上传递一次叫做一次训练。DL4J 有多个不同的方法来多次传递数据。

最简单的方法，是重置你的DataSetIterator并在fit的调用上循环你想要的次数。这种方法可以训练你的模型一直到你认为你的训练有良好的拟合。

然而还有一种方法是使用[EarlyStoppingTrainer](https://deeplearning4j.org/api/1.0.0-beta2/org/deeplearning4j/earlystopping/trainer/EarlyStoppingTrainer.html)。只要你喜欢，你可以配置这个训练器用于你想要的训练次数。它将在每个训练之后评估你的网络性能（或者你 所配置的任何阶段），并保存性能最好的版本供以后使用。

还要注意的是，DL4J不仅支持多层次网络的训练，而且还支持更灵活的计算图

## 评估模型性能

当你训练你的模型时，你会想测试它的性能。对于该测试，你将需要一个专用的数据集，该数据集将不用于训练，而是仅用于评估模型。这些数据应该与你想用模型进行预测的真实世界数据有相同的分布。不能简单地将训练数据用于评估的原因是因为机器学习方法容易过拟合（擅长对训练集进行预测，但在较大的数据集上表现不佳）。

评价类用于评价。略有不同的方法适用于评估一个正常的前馈网络或循环网络。有关使用它的更多细节，请查看相应的[示例](https://github.com/deeplearning4j/dl4j-examples)。

### 神经网络模型故障排查

建立神经网络来解决问题是一个经验过程。也就是说，它需要反复试验。因此，你必须尝试不同的设置和体系结构，以便找到性能良好的神经网络配置。

DL4J提供了一个监听器工具，帮助你直观地监视网络的性能。你可以为你的模型设置监听器，在每个小批量处理之后调用。DL4J最常用的监听器之一是[ScoreIterationListener](https://deeplearning4j.org/api/1.0.0-beta2/org/deeplearning4j/optimize/listeners/ScoreIterationListener.html)。检查更多的[Listeners](https://deeplearning4j.org/docs/latest/deeplearning4j-nn-listeners)。

虽然ScoreIterationListener将简单的打印你的网络的当前错误分数，HistogramIterationListener将启动一个网页界面来提供你一组可以用来微调网络配置的不同信息。查看可视化，网络学习监控与调试来知道如何解释这些数据

查看神经网络模型故障排查获取如何改进结果的更多信息。




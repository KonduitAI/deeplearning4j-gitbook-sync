---
description: 在特定条件下终止训练。
---

# 早停

## 什么是早停?

在训练神经网络时，需要对使用的设置（超参数）作出许多决策，以便获得良好的性能。一旦这样的超参数是训练epochs的数目：也就是说，数据集（epochs）的完整传递应该有几次？如果我们使用太少的epochs，我们可能欠拟合（即，不能从训练数据中学习我们能学的所有）；如果我们使用太多的epochs，我们可能过拟合（即，在训练数据中拟合“噪声”，而不是信号）。  
早期停止尝试删除手动设置该值的需要。它也可以被认为是一种正则化方法（如L1/L2权重衰减和丢弃），因为它可以阻止网络过拟合。

早停的思想相对简单：

* 将数据分割成训练集和测试集
* 在每个epoch的末尾（或每N个epoch）：
  * 评估测试集上的网络性能
  * 如果网络性能超过以前的最佳模型：在当前epoch保存网络的副本
* 作为最终模型，具有最佳测试集性能的模型

下面用图形显示：

![](../.gitbook/assets/image%20%282%29.png)

最好的模型是在垂直虚线时保存的模型，即在测试集上具有最高准确率的模型。  
使用DL4J的早停功能需要你提供一些配置选项：

* 得分计算器，如多层网络的DataSetLossCalculator\([JavaDoc](https://deeplearning4j.org/api/latest/org/deeplearning4j/earlystopping/scorecalc/DataSetLossCalculator.html), [Source Code](https://github.com/deeplearning4j/deeplearning4j/blob/c152293ef8d1094c281f5375ded61ff5f8eb6587/deeplearning4j-core/src/main/java/org/deeplearning4j/earlystopping/scorecalc/DataSetLossCalculator.java)\)或计算图的_DataSetLossCalculatorCG_ \([JavaDoc](https://deeplearning4j.org/api/latest/org/deeplearning4j/earlystopping/scorecalc/DataSetLossCalculatorCG.html), [Source Code](https://github.com/deeplearning4j/deeplearning4j/blob/c152293ef8d1094c281f5375ded61ff5f8eb6587/deeplearning4j-core/src/main/java/org/deeplearning4j/earlystopping/scorecalc/DataSetLossCalculatorCG.java)\)。用于在每个epoch进行计算（例如：测试集上的损失函数值或测试集上的准确率）
* 我们想要计算分数函数的频率（默认值：每个epoch）
* 一个或多个终止条件，它告诉训练过程何时停止。停止条件有两类：
  * Epoch终止条件:每N个epoch评估
  * 迭代终止条件: 每个小批量评估一次 
* 一个模型保存器，它定义了如何保存模型。

例如，在epoch终止条件最大为30epoch、最大为20分钟的训练时间的情况下，计算每个epoch的得分，并将中间结果保存到磁盘：

```java

MultiLayerConfiguration myNetworkConfiguration = ...;
DataSetIterator myTrainData = ...;
DataSetIterator myTestData = ...;

EarlyStoppingConfiguration esConf = new EarlyStoppingConfiguration.Builder()
		.epochTerminationConditions(new MaxEpochsTerminationCondition(30))
		.iterationTerminationConditions(new MaxTimeIterationTerminationCondition(20, TimeUnit.MINUTES))
		.scoreCalculator(new DataSetLossCalculator(myTestData, true))
        .evaluateEveryNEpochs(1)
		.modelSaver(new LocalFileModelSaver(directory))
		.build();

EarlyStoppingTrainer trainer = new EarlyStoppingTrainer(esConf,myNetworkConfiguration,myTrainData);

//执行早停训练
EarlyStoppingResult result = trainer.fit();

//打印结果
System.out.println("Termination reason: " + result.getTerminationReason());
System.out.println("Termination details: " + result.getTerminationDetails());
System.out.println("Total epochs: " + result.getTotalEpochs());
System.out.println("Best epoch number: " + result.getBestModelEpoch());
System.out.println("Score at best epoch: " + result.getBestModelScore());

//得到最佳模型:
MultiLayerNetwork bestModel = result.getBestModel();

```

你还可以实现自己的迭代和epoch终止条件。

## 早停并行包装器

上面描述的早停实现将仅用于单个设备。然而，`EarlyStoppingParallelTrainer`提供与早期停止类似的功能，并允许你为多个CPU或GPU进行优化。`EarlyStoppingParallelTrainer`将你的模型包装在`ParallelWrapper`类中，并执行本地化的分布式训练。

请注意，`EarlyStoppingParallelTrainer`并不支持作为其单个设备使用时所具有的所有功能。它不是UI兼容的，可能无法与复杂的迭代监听器一起工作。这是由于模型是在后台分发和复制的机置引起的。

## API


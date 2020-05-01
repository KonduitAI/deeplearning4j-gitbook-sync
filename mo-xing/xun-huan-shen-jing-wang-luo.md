---
description: 循环神经网络在DL4J中的实现。
---

# 循环神经网络

### DL4J中的循环神经网络

本文概述了在DL4J中如何使用循环神经网络的具体训练特征和实用性。本文假定对循环神经网络及其使用有一定了解，而不是对循环神经网络的介绍，并且假定你对它们的使用和术语有一些熟悉。

**内容**

* [基础：数据和网络配置](xun-huan-shen-jing-wang-luo.md#ji-chu-shu-ju-he-wang-luo-pei-zhi)
* [RNN训练特征](xun-huan-shen-jing-wang-luo.md#rnn-xun-lian-te-zheng)
  * [通过时间截断的反向传播](xun-huan-shen-jing-wang-luo.md#tong-guo-shi-jian-jie-duan-de-fan-xiang-chuan-bo)
  * [掩码：一对多、多对一和序列分类](xun-huan-shen-jing-wang-luo.md#yan-ma-yi-dui-duo-duo-dui-yi-he-xu-lie-fen-lei)
    * [训练后的掩码与序列分类](xun-huan-shen-jing-wang-luo.md#xun-lian-hou-de-yan-ma-yu-xu-lie-fen-lei)
  * [RNN层与其它层结合](xun-huan-shen-jing-wang-luo.md#rnn-ceng-yu-qi-ta-ceng-jie-he)
* [测试时间：一步一步的预测](xun-huan-shen-jing-wang-luo.md#ce-shi-shi-jian-yi-bu-yi-bu-de-yu-ce)
* [导入时间序列数据](xun-huan-shen-jing-wang-luo.md#dao-ru-shi-jian-xu-lie-shu-ju)
* [示例](xun-huan-shen-jing-wang-luo.md#shi-li-1-zai-dan-du-wen-jian-zhong-tong-yi-chang-du-shu-ru-he-biao-qian-de-shi-jian-xu-lie)

## [基础：数据和网络配置](xun-huan-shen-jing-wang-luo.md)

DL4J 目前支持以下类型的循环神经网络

* GravesLSTM \(长短期记忆\)
* BidirectionalGravesLSTM（双向格拉夫长短期记忆）
* BaseRecurrent

每种网络的Java文档都是可用的, [GravesLSTM](https://deeplearning4j.org/api/1.0.0-beta2/org/deeplearning4j/nn/conf/layers/GravesLSTM.html),[BidirectionalGravesLSTM](https://deeplearning4j.org/api/1.0.0-beta2/org/deeplearning4j/nn/conf/layers/GravesBidirectionalLSTM.html), [BaseRecurrent](https://deeplearning4j.org/api/latest/org/deeplearning4j/nn/conf/layers/BaseRecurrentLayer.html)

用于RNN的数据

暂时考虑一个标准的前馈网络（DL4J中的多层感知机或“密连层”）。这些网络期望输入和输出数据是二维的：即，具有“形状”的数据\[numExamples,inputSize\]。这意味着进入前馈网络的数据具有“numExamples”行/示例，其中每行由“inputSize”列组成。单个示例将具有形状\[1,inputSize\]，但是在实践中，为了计算和优化效率，我们通常使用多个示例。类似地，标准前馈网络的输出数据也是二维的，具有形状\[numExamples,outputSize\]。

相反，RNN的数据是时间序列。因此，他们有3个维度：一个额外的时间维度。输入数据因此具有形状\[numExamples,inputSize,timeSeriesLength\]，输出数据具有形状\[numExamples,outputSize,timeSeriesLength\]。这意味着，我们的INDArray中的数据被布置成如此 使得位置（i，j，k）的值是 小批量中第i个示例的第k个时间步骤的第j个值。该数据布局如下所示。

当使用类CSVSequenceRecordReader导入时间序列数据时，数据文件中的每一行表示一个时间步骤，用第一行（或头行后第一行（如果存在）中的最早观察到的时间序列 及csv的最后一行中的最新观察来表示。每个特征时间序列是CSV文件的单独列。例如，如果你在时间序列中有五个特征，每个特征具有120个观察值，以及大小为53的训练和测试集，那么将有106个输入csv文件（53个输入，53个标签）。53个输入CSV文件将分别有五列和120行。标签CSV文件将有一列（标签）和一行。

![Data: Feed Forward vs. RNN](http://upload-images.jianshu.io/upload_images/14495907-0d958a8df992072c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

RnnOutputLayer （循环神经网络输出层）

循环神经网络输出层是用作具有许多循环神经网络系统（用于回归和分类任务）的最后层的一种层。循环神经网络输出层处理诸如评分计算、给定损失函数时的错误计算（预测与实际）等问题。在功能上，它与“标准”OutputLayer类（与前馈网络一起使用）非常相似；但是它同时输出（并且期望作为标签/目标）三维时间序列数据集。

循环神经网络输出层的配置遵循与其他层相同的设计：例如，将多层网络中的第三层设置为循环神经网络输出层以进行分类：

```java
.layer(2, new RnnOutputLayer.Builder(LossFunction.MCXENT).activation(Activation.SOFTMAX)
.weightInit(WeightInit.XAVIER).nIn(prevLayerSize).nOut(nOut).build())
```

在实践中使用循环神经网络输出层可以在示例中看到，链接到本文末尾。

## [RNN 训练特征](xun-huan-shen-jing-wang-luo.md)

### [通过时间截断的反向传播](xun-huan-shen-jing-wang-luo.md)

训练神经网络（包括RNNs）会在计算上非常苛刻。对于循环神经网络，当我们处理长序列时尤其如此，即具有许多时间步长的训练数据。

为了降低循环神经网络中每个参数更新的计算复杂度，提出了截断反向传播时间算法（BPTT）。总而言之，对于给定的计算能力，它允许我们更快地训练网络（通过执行更频繁的参数更新）。建议在输入序列长的时候使用截断的BPTT（通常超过几百个时间步长）。

考虑当训练具有长度为12个时间步长的时间序列的循环神经网络时会发生什么。这里，我们需要进行12步的正向传递，计算误差（基于预测的与实际的），并且进行12步的后向传递：

![Standard Backprop Training](http://upload-images.jianshu.io/upload_images/14495907-57ee585a634debeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ​对于12个时间步长，在上面的图像中，这不是问题。然而，考虑到输入时间序列是10000个或更多的时间步长。在这种情况下，对于每个参数更新的每个正向和向后传递，通过时间的标准反向传播将需要10000个时间步骤。这计算要求当然是非常大的。

在实践中，截断的BPTT将前向和后向传播分成一组较小的前向/后向传播操作。这些向前/向后传播片断的长度是由用户设置的参数。例如，如果我们使用长度为4个时间步长的截断BPTT，学习看起来如下：

![Truncated BPTT](http://upload-images.jianshu.io/upload_images/14495907-198f4001a7b255ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ​请注意，截断BPTT和标准BPTT的总体复杂度大致相同——在前向/反向传播中，它们时间步数量都相同。然而，使用这种方法，我们得到3个参数更新，而不是一个近似相同的工作量。然而，成本并不完全相同，每个参数更新都有少量的开销。

截断BPTT的缺点是在截断的BPTT中学习的依赖的长度可以短于完整的BPTT。这是很容易看到的：考虑上面的图像，TBPTT长度为4。假设在时间步骤10，网络需要存储来自时间步骤0的一些信息，以便做出准确的预测。在标准的BPTT中，这是可以的：梯度可以从时间10到时间0沿着展开的网络一路向后流动。在截断的BPTT中，这是有问题的：时间步10的梯度没有返回到足够远的地方，导致存储所需信息的所需参数更新。这种折衷通常是值得的，并且（只要适当地设置截断BPTT长度），截断BPTT在实践中工作得很好。

在DL4J中使用截断的BPTT非常简单：只需将下列代码添加到网络配置中（最后，在网络配置的最后.build\(\)之前）

```java
.backpropType(BackpropType.TruncatedBPTT)
.tBPTTLength(100)
```

上面的代码片段将导致任意网络训练（即，对MultiLayerNetwork.fit\(\) 方法的调用）使用长度为100步的片段的截断BPTT。

一些值得注意的事情:

* 默认情况下（如果不手动指定反向传播类型），DL4J将使用BackpropType.Standard（即，全BPTT）。
* tBPTTLength配置参数设置截断的BPTT传递的长度。通常，这是在50到200个时间步长的某个地方，不过取决于应用程序和数据。
* 截断的BPTT长度通常是总时间序列长度（即，200对序列长度1000）的一部分，但是当使用TBPTT（例如，具有两个序列的小批—一个长度为100和另一个长度为1000——以及TBPTT长度为200 -将正确工作）时，在同一个小批次中时可变长度时间序列是可以的。 

### [掩码：一对多、多对一和序列分类](xun-huan-shen-jing-wang-luo.md)

基于填充和掩码的思想DL4J支持RNN的一些相关的训练特征。填充和掩码允许我们支持训练情况，包括一对多、多对一，还支持可变长度时间序列（在同一小批量中）。

假设我们想训练一个具有不会在每一个时间步长发生输入或输出的循环神经网络。这个例子（对于一个例子）在下面的图像中显示。DL4J支持所有这些情况的网络训练：

![RNN Training Types](http://upload-images.jianshu.io/upload_images/14495907-8a95eeb94f999b6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ​没有掩码和填充，我们仅限于多对多的情况（上面，左边）：即，（a）所有示例都具有相同的长度，（b）示例在所有时间步骤都有输入和输出。

填充背后的思想很简单。考虑在相同的小批量中两个长度分别为50和100时间步的时间序列。训练数据是矩形阵列；因此，我们填充（即，向其添加零）较短的时间序列（对于输入和输出），使得输入和输出都具有相同的长度（在本示例中：100个时间步骤）。

当然，如果这是我们全部所做的，它会在训练过程中产生问题。因此，除了填充，我们使用掩码机制。掩码背后的思想很简单：我们有两个额外的数组，记录输入或输出是否实际出现在给定时间步长和示例中，或者输入/输出是否只是填充。

回想一下，对于RNN，我们的小批量处理数据具有3维，分别具有输入和输出的形状\[miniBatchSize、inputSize、timeSeriesLength\]和\[miniBatchSize、outputSize、timeSeriesLength\]。然后填充数组是二维的，输入和输出都具有形状\[miniBatchSize，timeSeriesLength\]，每个时间序列和示例的值是0 \(‘absent’\) or 1 \(‘present’\)。用于输入和输出的掩码数组被存储在单独的数组中。

对于单个示例，输入和输出掩码数组如下所示：

![RNN Training Types](http://upload-images.jianshu.io/upload_images/14495907-979fa2b4f6eb5214.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ​对于“不需要掩码”的情况，我们可以等效地使用所有1的掩码数组，这将给出与完全没有掩码数组相同的结果。还要注意，在学习RNN时可以使用零、一或两个掩码数组——例如，多对一的情况可以只针对输出使用掩码数组。

在实践中：这些填充数组通常在数据导入阶段创建（例如，由SequenceRecordReaderDatasetIterator（稍后讨论）创建），并且包含在DataSet对象中。如果DataSet包含掩码数组，多层网络拟合将在训练期间自动使用它们。如果它们不存在，则不使用掩码功能。

带有掩码的评估与评分

当进行评分和评估时（也就是说，当评估RNN分类器的精度）时，掩码数组也是重要的。例如，考虑多对一的情况：每个示例只有一个输出，并且任何评估都应该考虑这一点。

使用（输出）掩码数组的评估可以在评估时使用，通过把它传递到以下方法：

```java
Evaluation.evalTimeSeries(INDArray labels, INDArray predicted, INDArray outputMask)
```

其中，标签是实际输出\(3d时间序列\)，预测的是网络预测\(3d时间序列，与标签的形状相同\)，并且outputMask是用于输出的2d掩码数组。注意，评估不需要输入掩码数组。

得分计算也将利用掩码数组，通过MultiLayerNetwork.score\(DataSet\) 方法。同样，如果DataSet包含输出掩码数组，那么在计算网络的得分（损失函数-均方误差、负对数似然等）时将自动使用它。

#### [训练后的掩码与序列分类](xun-huan-shen-jing-wang-luo.md)

序列分类是掩码的一种常用方法。其思想是，尽管我们有一个序列（时间序列）作为输入，但我们只希望为整个序列提供一个单一的标签（而不是在序列中的每个时间步提供一个标签）。

然而，RNN通过设计输出序列，输入序列的长度相同。对于序列分类，掩码允许我们在最后的时间步用这个单一标签训练网络，我们本质上告诉网络除了最后的时间步之外实际上没有任何标签数据。

现在，假设我们已经训练了我们的网络，并且希望从时间序列输出数组获得最后的预测时间步。我们该怎么做呢？

为了得到最后一个时间步，有两个案例需要注意。首先，当只有一个示例时，实际上不需要使用掩码数组：我们只需要获得输出数组中的最后一个时间步：

```java
    INDArray timeSeriesFeatures = ...;
    INDArray timeSeriesOutput = myNetwork.output(timeSeriesFeatures);
    int timeSeriesLength = timeSeriesOutput.size(2);        //Size of time dimension
    INDArray lastTimeStepProbabilities = timeSeriesOutput.get(NDArrayIndex.point(0), NDArrayIndex.all(), NDArrayIndex.point(timeSeriesLength-1));
```

假设分类（不过，回归的过程相同），上面的最后一行给出了最后一个时间步的概率，即序列分类的类概率。

稍微复杂一点的情况是，在一个小批（特征数组）中有多个示例，其中每个示例的长度不同。（如果所有长度相同：我们可以使用与上面相同的过程）。

在这个“可变长度”的情况下，我们需要分别为每个示例获取最后一个时间步。如果数据流管道中的每个示例都有时间序列长度，那么就变得简单了：我们只是迭代示例，用该示例的长度替换上面代码中的`timeSeriesLength`。

如果我们没有直接的时间序列的长度，我们需要从掩码数组中提取它们。

如果我们有一个标签掩码数组（它是一个one-hot向量，像每个时间序列\[0，0，01,1,0\]）：

```java
    INDArray labelsMaskArray = ...;
    INDArray lastTimeStepIndices = Nd4j.argMax(labelMaskArray,1);
```

或者，如果我们只有特征掩码：一个快速和粗爆的方法就是使用这个：

```java
    INDArray featuresMaskArray = ...;
    int longestTimeSeries = featuresMaskArray.size(1);
    INDArray linspace = Nd4j.linspace(1,longestTimeSeries,longestTimeSeries);
    INDArray temp = featuresMaskArray.mulColumnVector(linspace);
    INDArray lastTimeStepIndices = Nd4j.argMax(temp,1);
```

要理解这里正在发生的事情，请注意，最初我们有一个特征掩码，如\[1,1,1,1,0\]，我们希望从中获得最后一个非零元素。我们映射\[1,1,1,1,0\] 到 \[1，2，3，4，0\]，然后得到最大的元素（这是最后一个时间步长）。

在这两种情况下，我们都可以做到以下几点：

```java
    int numExamples = timeSeriesFeatures.size(0);
    for( int i=0; i<numExamples; i++ ){
        int thisTimeSeriesLastIndex = lastTimeStepIndices.getInt(i);
        INDArray thisExampleProbabilities = timeSeriesOutput.get(NDArrayIndex.point(i), NDArrayIndex.all(), NDArrayIndex.point(thisTimeSeriesLastIndex));
    }
```

### [RNN层与其它层结合](xun-huan-shen-jing-wang-luo.md)

DL4J中的RNN层可以与其他层类型相结合。例如，可以在同一网络中组合DenseLayer（密连层）和LSTM（长短记录单元）层，或者组合用于视频的卷积\(CNN\)层和LSTM层。

当然，DenseLayer（密连层）和卷积层不处理时间序列数据——他们期望不同类型的输入。为了解决这个问题，我们需要使用层预处理器功能：例如，CnnToRnnPreProcessor和FeedForwardToRnnPreprocessor类。请参见[这里](https://github.com/deeplearning4j/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/preprocessor)所有预处理器。幸运的是，在大多数情况下，DL4J配置系统将根据需要自动添加这些预处理器。然而，可以手动添加预处理器（覆盖每个层自动添加的预处理器）。

例如，为了手动添在1层和2层之间添加预处理器，请将下列内容添加到网络配置中：`.inputPreProcessor(2, new RnnToFeedForwardPreProcessor())`

### [测试时间：一步一步的预测](xun-huan-shen-jing-wang-luo.md)

与其他类型的神经网络一样，可以使用MultiLayerNetwork.output\(\)和`MultiLayerNetwork.feedForward()` 方法生成对RNNs的预测。这些方法在许多情况下是有用的；然而，这些方法的局限性在于，我们只能对时间序列从零开始每次生成预测。

例如，考虑我们希望在实时系统中生成预测的情况，其中这些预测基于大量的历史。在这种情况下，使用输出/前馈方法是不切实际的，因为它们在每次调用整个数据历史时进行完全的正向传递。如果我们希望在每个时间步长对单个时间步长进行预测，那么这些方法可能既\(a\)非常耗性能，又\(b\)浪费，因为它们一遍又一遍地进行相同的计算。

对于这些情况，多层网络提供了四种方法：

* `rnnTimeStep(INDArray)`
* `rnnClearPreviousState()`
* `rnnGetPreviousState(int layer)`
* `rnnSetPreviousState(int layer, Map<String,INDArray> state)`

rnnTimeStep\(\)方法被设计成允许有效地执行前向传递（预测），一次执行一个或多个步骤。与输出/前馈方法不同，rnnTimeStep方法在被调用时跟踪RNN层的内部状态。重要的是要注意，rnnTimeStep和输出/feedForward方法的输出应该是相同的（对于每个时间步），无论我们是一次全部做出这些预测（输出/feedForward），还是每次生成一个或多个步骤（rnnTimeStep）。因此，唯一的区别应该是计算成本。

总之，MultiLayerNetwork.rnnTimeStep\(\)方法做了两件事：

1. 使用先前存储状态（如果有的话）生成输出/预测（前向传递）
2. 更新存储的状态，存储最后一个时间步的激活（准备下次rnnTimeStep被调用时使用）

例如，假设我们想使用RNN来预测天气，提前一小时（基于前面100小时的天气作为输入）。如果我们要使用输出方法，在每一小时，我们需要输入整整100个小时的数据，以预测101个小时的天气。然后，为了预测102小时的天气，我们需要输入100小时（或101小时）的数据，103小时等等。

或者，我们可以使用rnnTimeStep方法。当然，如果我们要在进行第一次预测之前利用全部100个小时的历史，我们仍然需要进行全面的向前传递：

![RNN Time Step](http://upload-images.jianshu.io/upload_images/14495907-059c30dbe2ef0ecf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们第一次调用rnnTimeStep时，两种方法之间唯一的实际区别是存储了上一个时间步的激活/状态——这用橙色表示。但是，下次我们使用rnnTimeStep方法时，这个存储状态将被用于做出下一个预测：

![RNN Time Step](http://upload-images.jianshu.io/upload_images/14495907-06f4f11b030be018.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ​这里有许多重要的区别：

1. 在第二个图片中（rnnTimeStep的第二次调用），输入数据由单个时间步组成，而不是由数据的完整历史组成
2. 前向传播是一个单一的时间步（与数百个或更多）相比。
3. rnnTimeStep方法返回后，内部状态将自动更新。因此，可以以与时间102相同的方式进行时间103的预测。等等。

但是，如果希望开始对新的（完全独立的）时间序列进行预测，则必须（而且很重要）使用`MultiLayerNetwork.rnnClearPreviousState()`方法手动清除存储的状态。这将重置网络中所有循环层的内部状态。

如果需要存储或设置用于预测的RNN的内部状态，则可以针对每一层分别使用rnnGetPreviousState和rnnSetPreviousState方法。例如，在序列化（网络保存/加载）期间，这可能是有用的，因为rnnTimeStep方法中的内部网络状态在缺省情况下没有被保存，并且必须单独保存和加载。注意，这些GET/SET状态方法返回并接受一个由激活类型作为键的映射。例如，在LSTM模型中，有必要存储输出激活和存储单元状态。

其他一些注意事项：

* 我们可以同时为多个独立的示例/预测使用rnnTimeStep方法。在上面的天气示例中，例如，我们希望使用相同的神经网络对多个地点进行预测。这与训练和前向传递/输出方法相同：多行（输入数据中的维度0）用于多个示例。
* 如果没有设置历史/存储状态（即，最初或调用rnnClearPreviousState之后），则使用默认初始化（零）。这是与训练过程相同的方法。
* rnnTimeStep可以同时用于任意数量的时间步-不只是一个时间步。然而，重要的是要注意：
  * 对于单个时间步预测：数据是二维的，形状是 \[numExamples，nIn\]；在这种情况下，输出也是二维的，形状是\[numExamples，nOut\]。
  * 对于多个时间步预测：数据是三维的，具有形状\[numExamples,nIn,numTimeSteps\]；输出将具有形状\[numExamples,nOut,numTimeSteps\]。同样，最后的时间步激活和以前一样被存储。
* 在rnnTimeStep的调用之间不可能改变示例的数量（换句话说，如果rnnTimeStep的第一次使用是针对例如3个示例，则所有后续的调用必须具有3个示例）。在重置内部状态之后（使用rnnClearPreviousState\(\)\)，任何数量的示例都可以用于rnnTimeStep的下一次调用。
* rnnTimeStep方法不改变参数，只在训练完成后才使用网络。
* rnnTimeStep方法与包含单个和堆叠/多个RNN层的网络以及结合其他层类型（例如卷积层或密连层）的网络一起工作。
* RnnOutputLayer 层类型不具有任何内部状态，因为它没有任何循环连接。

### [导入时间序列数据](xun-huan-shen-jing-wang-luo.md)

RNN的数据导入是复杂的，因为我们有多种不同类型的数据可用于RNN：一对多、多对一、可变长度时间序列等。本节将描述当前实现的DL4J的数据导入机制。

这里描述的方法利用SequenceRecordReaderDataSetIterator类，以及来自DataVec的CSVSequenceRecordReader类。此方法目前允许你从文件中加载被（制表符、逗号等）分隔的数据，其中每个时间序列位于单独的文件中。该方法还支持：

* 可变长度时间序列输入
* 一对多和多对一数据加载（输入和标签在不同文件中）
* 用于分类的从索引到one-hot表示（即“2”到\[0,0,1,0\]）的标签转换
* 跳过数据文件开始时的固定/指定行数（即注释或头行）

注意在所有情况下，数据文件中的每一行代表一个时间步。

（除了下面的例子，你可能会发现这些[单元测试](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-core/src/test/java/org/deeplearning4j/datasets/datavec/RecordReaderDataSetiteratorTest.java)是有用处的。）

#### [示例1：在单独文件中同一长度、输入和标签的时间序列](xun-huan-shen-jing-wang-luo.md)

假设在我们的训练数据中有10个时间序列，由20个文件表示：每个时间序列有10个文件用于输入，而输出/标签有10个文件。现在，假设这20个文件都包含相同数量的时间步（即，相同的行数）。

为了使用[SequenceRecordReaderDataSetIterator](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-datavec-iterators/src/main/java/org/deeplearning4j/datasets/datavec/SequenceRecordReaderDataSetIterator.java)和[CSVSequenceRecordReader](https://github.com/deeplearning4j/deeplearning4j/blob/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/csv/CSVSequenceRecordReader.java)方法，我们首先创建两个CSVSequenceRecordReader对象，一个用于输入，一个用于标签：

```java
SequenceRecordReader featureReader = new CSVSequenceRecordReader(1, ",");
SequenceRecordReader labelReader = new CSVSequenceRecordReader(1, ",");
```

这个特定的构造函数需要跳过的行数（这里跳过的1行）和分隔符（这里使用逗号字符）。

第二，我们需要初始化这两个读取器，告诉他们从哪里获取数据。我们使用一个InputSplit对象来实现这一点。假设我们的时间序列被编号，文件名为“myInput\_0.csv”、“myInput\_1.csv”、“myLabels\_0.csv”等等。一种方法是使用[NumberedFileInputSplit](https://github.com/deeplearning4j/deeplearning4j/blob/master/datavec/datavec-api/src/main/java/org/datavec/api/split/NumberedFileInputSplit.java)：

```java
featureReader.initialize(new NumberedFileInputSplit("/path/to/data/myInput_%d.csv", 0, 9));
labelReader.initialize(new NumberedFileInputSplit(/path/to/data/myLabels_%d.csv", 0, 9));
```

在这个特定的方法中，“%d”被替换为相应的数字，并且使用数字0到9（包括 ）。

最后，我们可以创建我们的SequenceRecordReaderdataSetIterator：

```java
DataSetIterator iter = new SequenceRecordReaderDataSetIterator(featureReader, labelReader, miniBatchSize, numPossibleLabels, regression);
```

这个DataSetIterator可以被传送到MultiLayerNetwork.fit\(\) 方法来训练网络。

miniBatchSize参数指定每个小批量中的实例数量（时间序列）。例如，对于总共10个文件，miniBatchSize为5将给我们两个数据集，每个数据集包含2个小批（DataSet对象），每个小批中包含5个时间序列。

注意：

* 对于分类问题: numPossibleLabels是你的数据集中类别的数量 。 使用  regression = false。
  * 标签数据：每行一个值，作为一个分类索引
  * 标签数据将自动转换为one-hot表示。
* 对于回归问题: numPossibleLabels不被使用\(设为任何值\) 并使用 regression = true。
  * 输入和标签中的值可以是任意的（不像分类：可以有任意数量的输出）。
  * 当regression = true 不进行标签的处理。

#### [示例2：在同一文件中同一长度、输入和标签的时间序列](xun-huan-shen-jing-wang-luo.md)

根据上一个示例，假设我们的输入数据和标签不是一个单独的文件，而是在同一个文件中。但是，每个时间序列仍然在一个单独的文件中。

从DL4J 0.4-rc3.8开始，这种方法对输出具有单列的限制（分类索引或单个实值回归输出）

在这种情况下，我们创建并初始化单个读取器。同样，我们跳过一个标题行，并将格式指定为逗号分隔，并假设我们的数据文件名为“myData\_0.csv”, …, “myData\_9.csv”:

```java
SequenceRecordReader reader = new CSVSequenceRecordReader(1, ",");
reader.initialize(new NumberedFileInputSplit("/path/to/data/myData_%d.csv", 0, 9));
DataSetIterator iterClassification = new SequenceRecordReaderDataSetIterator(reader, miniBatchSize, numPossibleLabels, labelIndex, false);
```

`miniBatchSize`和`numPossibleLabels`与前面的示例相同。这里，`labelIndex`指定标签在哪个列。例如，如果标签在第五列中，则使用labelIndex= 4（即，列被索引为0到numColumns-1）。

对于单一输出值的回归，我们使用：

```java
DataSetIterator iterRegression = new SequenceRecordReaderDataSetIterator(reader, miniBatchSize, -1, labelIndex, true);
```

同样，numPossibleLabels参数不用于回归。

#### [示例3：不同长度的时间序列（多对多）](xun-huan-shen-jing-wang-luo.md)

根据前两个示例，假设对于每个示例，输入和标签具有相同的长度，但是这些长度在时间序列之间不同。

我们可以使用相同的方法（CSVSequenceRecordReader和SequenceRecordReaderDataSetIterator），不过使用不同的构造函数：

```java
DataSetIterator variableLengthIter = new SequenceRecordReaderDataSetIterator(featureReader, labelReader, miniBatchSize, numPossibleLabels, regression, SequenceRecordReaderDataSetIterator.AlignmentMode.ALIGN_END);
```

此处的参数与前面的示例相同，除了AlignmentMode.ALIGN\_END。这种对齐模式输入告诉SequenceRecordReaderDataSetIterator需要两件事：

1. 时间序列可以具有不同的长度。
2. 为每个示例把输入和标签对齐，以使它们的最后值出现在同一时间步。

注意，如果特征和标签总是具有相同的长度（如示例3中的假设），那么两个对齐模式（AlignmentMode.ALIGN\_END 和 AlignmentMode.ALIGN\_START）将给出相同的输出。对齐模式选项将在下一节中解释。

还要注意：在数据数组中，可变长度时间序列总是在时间0处开始：如果要求，将在时间序列结束之后添加填充。

与上面的示例1和2不同，上面的variableLengthIter实例生成的DataSet对象还将包括输入和掩码数组，如本文前面所述。

例4：多对一和一对多数据

我们还可以使用示例3中的AlignmentMode功能来实现多对一RNN序列分类器。在这里，让我们假设：

* 输入和标签在单独的被分隔的文件中。
* 标签文件包含一个单行（时间步）（用于分类的类索引，或用于回归的一个或多个数字）。
* 示例之间的输入长度可以（可选地）不同。

实际上，与示例3相同的方法可以做到这一点：

```java
DataSetIterator variableLengthIter = new SequenceRecordReaderDataSetIterator(featureReader, labelReader, miniBatchSize, numPossibleLabels, regression, SequenceRecordReaderDataSetIterator.AlignmentMode.ALIGN_END);
```

对准模式相对简单。它们指定是否填充较短时间序列的开始或结束。下面的图表展示了它如何与掩码数组（如本文前面所讨论的）一起工作：

![Sequence Alignment](http://upload-images.jianshu.io/upload_images/14495907-21a643f775073463.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ​一对多个案例（类似于上面的最后一个案例，但只有一个输入）是通过使用AlignmentMode.ALIGN\_START实现。

注意，在包含不同长度的时间序列的训练数据的情况下，标签和输入将针对每个示例单独对齐，然后按要求填充更短的时间序列：

![Sequence Alignment](http://upload-images.jianshu.io/upload_images/14495907-4a8a57efedf03acc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 可用的层


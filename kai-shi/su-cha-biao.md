---
description: 提供Eclipse Deeplearning4j里通用的功能与代码片段
---

# 速查表

## 快速检索

DL4J（和相关项目）有很多功能。此篇的目标是总结这个功能，以便用户知道存在什么功能，以及在哪里可以找到更多信息。

内容

* [层](su-cha-biao.md#ceng)
  * [前馈层](su-cha-biao.md#qian-kui-ceng)
  * [输出层](su-cha-biao.md#shu-chu-ceng)
  * [卷积层](su-cha-biao.md#juan-ji-ceng)
  * [循环层](su-cha-biao.md#xun-huan-ceng)
  * [无监督层](su-cha-biao.md#wu-jian-du-ceng)
  * [其它层](su-cha-biao.md#qi-ta-ceng)
  * [图顶点](su-cha-biao.md#tu-ding-dian)
  * [输入预处理器](su-cha-biao.md#shu-ru-yu-chu-li-qi)
* [迭代/训练监听器](su-cha-biao.md#die-dai-xun-lian-jian-ting-qi)
* [评估](su-cha-biao.md#ping-gu)
* [网络保存和加载](su-cha-biao.md#wang-luo-de-bao-cun-yu-jia-zai)
* [网络配置](su-cha-biao.md#wang-luo-pei-zhi)
  * [激活函数](su-cha-biao.md#ji-huo-han-shu)
  * [权重初始化](su-cha-biao.md#quan-zhong-chu-shi-hua)
  * [更新器 \(优化器\)](su-cha-biao.md#geng-xin-qi-you-hua-qi)
  * [学习率调度](su-cha-biao.md#xue-xi-tiao-du)
  * [正则化](su-cha-biao.md#zheng-ze-hua)
    * [ L1/L2 正则化](su-cha-biao.md#l-1-l2-zheng-ze-hua)
    * [Dropout\(丢弃\)](su-cha-biao.md#dropout-diu-qi)
    * [权重噪声](su-cha-biao.md#quan-zhong-zao-sheng)
    * [约束](su-cha-biao.md#yue-shu)
* [数据类](su-cha-biao.md#shu-ju-lei)
  * [迭代器](su-cha-biao.md#die-dai-qi)
    * [迭代器 - 内置 \(DL4J-提供数据\)](su-cha-biao.md#nei-zhi-die-dai-qi-dl-4-j-ti-gong-shu-ju)
    * [迭代器 -用户提供数据](su-cha-biao.md#die-dai-qi-yong-hu-ti-gong-de-shu-ju)
    * [迭代器 - 适配器和实用迭代器](su-cha-biao.md#die-dai-qi-kuo-pei-qi-yu-shi-yong-die-dai-qi)
  * [读取原始数据: DataVec 记录读取器](../datavec/gai-shu.md#shu-ju-xiang-liang-yi-ge-xiang-liang-hua-de-etl-chou-qu-zhuan-huan-he-jia-zai-ku)
  * [数据归一化](su-cha-biao.md#shu-ju-gui-yi-hua)
  * [Spark 网络训练数据类](../fen-bu-shi-shen-du-xue-xi/spark-shu-ju-guan-dao-zhi-nan.md)
* [迁移学习](su-cha-biao.md#qian-yi-xue-xi)
* [已训练的模型库 - Model Zoo](su-cha-biao.md#yi-xun-lian-de-mo-xing-ku-model-zoo)
* [Keras 导入](../keras-dao-ru/gai-shu.md#dl-4-j-keras-mo-xing-dao-ru)
* [分布式训练 \(Spark\)](../fen-bu-shi-shen-du-xue-xi/jie-shao-yu-ru-men.md)
* [超参数优化\(Arbiter\)](../arbiter/gai-shu.md#chao-can-shu-you-hua)

### [层](su-cha-biao.md)

### [前馈层](su-cha-biao.md)

DenseLayer - \([源码](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/feedforward/dense/DenseLayer.java)\) - 简单/标准全连接层

EmbeddingLayer - \([源码](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/feedforward/embedding/EmbeddingLayer.java)\) - 以正整数索引作为输入，输出向量。只作为模型中的第一层使用。数学上等效于（当启用偏置）DenseLayer，使用OneHot输入，但更高效。

#### [输出层](su-cha-biao.md)

输出层:通常用作网络中的最后一层。这里会设置损失函数。

OutputLayer - \([源码](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/OutputLayer.java)\) - 在MLPs/CNNs中的标准分类/回归输出层。有一个内置的全连接的DenseLayer。 2d 输入/输出 \(即, 每个示例中的行向量\)。

LossLayer - \([源码](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LossLayer.java)\) - 没有参数的输出层 - 只有损失函数和激活函数。2d 输入/输出 \(即, 每个示例中的行向量\)。与Outputlayer 不同，它有nIn = nOut的限制。

RnnOutputLayer - \([源码](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/RnnOutputLayer.java)\) - 循环神经网络的输出层。三维（时间序列）的输入和输出。内置有时间分布全连接层。

RnnLossLayer - \([源码](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/RnnLossLayer.java)\) - 无参版本的RnnOutputLayer。 三维（时间序列）的输入和输出。

CnnLossLayer - \([源码](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/CnnLossLayer.java)\) - 与CNNs一起使用，其中必须在输出的每个空间位置进行预测（例如：分割或去噪）。没有参数，四维输入/输出与形状\[小批量，深度，高度，宽度\]。当使用softmax时，这是在每个空间位置的深度应用。

Yolo2OutputLayer - \([源码](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/objdetect/Yolo2OutputLayer.java)\) - 用于目标检测的YOLO 2模型实现

CenterLossOutputLayer - \([源码](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/CenterLossOutputLayer.java)\) - OutputLayer的一个版本，也试图最小化示例激活的类内距离，即，“如果示例x在Y类中，则确保嵌入\(x\)接近于所有示例y在Y中的平均值\(嵌入\(y\)\)。

#### [卷积层](su-cha-biao.md)

* **ConvolutionLayer** / Convolution2D - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/ConvolutionLayer.java)\) - 标准的二维卷积神经网络层。输入和输出有4个维度形状分别为  \[minibatch,depthIn,heightIn,widthIn\] 和\[minibatch,depthOut,heightOut,widthOut\]。
* **Convolution1DLayer** / Convolution1D - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Convolution1DLayer.java)\) - 标准的一维卷积神经网络层。
* **Deconvolution2DLayer** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/convolution/Deconvolution2DLayer.java)\) - 也称为转置或分数阶卷积。可以认为是“反向”卷积层；输出通常大于输入，同时保持空间连接结构。
* **SeparableConvolution2DLayer** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/convolution/SeparableConvolution2DLayer.java)\) - 深度可分离卷积层。
* **SubsamplingLayer** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/convolution/subsampling/SubsamplingLayer.java)\) -为CNNs实现的标准二维空间池化，最大、平均和p范数池可用。
* **Subsampling1DLayer** - \([Source](https://deeplearning4j.org/docs/latest/deeplearning4j-cheat-sheet)\)
* **Upsampling2D** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/convolution/upsampling/Upsampling2D.java)\) - 通过重复行／列 来升级CNN激活 。
* **Upsampling1D** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/convolution/upsampling/Upsampling1D.java)\) - 一维版本的上采样层
* **Cropping2D** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/convolutional/Cropping2D.java)\) - 二维卷积神经网络的裁剪层。
* **ZeroPaddingLayer** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/convolution/ZeroPaddingLayer.java)\) -非常简单的层，将指定数量的零填充添加到四维输入激活的边缘。
* **ZeroPadding1DLayer** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/convolution/ZeroPadding1DLayer.java)\) - 一维版本的ZeroPaddingLayer
* **SpaceToDepth** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/SpaceToDepthLayer.java)\) - 给定块大小，这个操作采用四维数组，并把数据从空间维度移动到通道。
* **SpaceToBatch** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/SpaceToBatchLayer.java)\) - 根据指定的“块”，将张量从2个空间维度转换为批量维度。

#### [循环层](su-cha-biao.md)

* **LSTM** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LSTM.java)\) - 没有窥视孔连接的LSTM RNN。 支持 CuDNN。
* **GravesLSTM** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/GravesLSTM.java)\) - 具有窥视孔连接的LSTM RNN。不支持CuDNN（因此对于GPU，LSTM应该优先使用）。
* **GravesBidirectionalLSTM** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/GravesBidirectionalLSTM.java)\) - 具有窥视连接的双向LSTM实现。等效于双向（ADD，GravesLSTM）。由于增加了双向包装（以下），已被主干弃用。
* **Bidirectional** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/recurrent/Bidirectional.java)\) - 一个“包装”层-将任何标准的单向RNN转换成双向RNN（双倍数量的参数-前向/后向网络具有独立的参数）。前向/后向网络的激活可以是增加的、乘法的、平均的或级联的。
* **SimpleRnn** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/recurrent/SimpleRnn.java)\) - 一个标准的“普通”RNN层。在长时间系列依赖的情况下，通常在实际中不生效。更推荐使用LSTM。 
* **LastTimeStep** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/recurrent/LastTimeStep.java)\) -  一个“包装器”层提取出它封装的（非双向）RNN层的最后一个时间步长。三维输入的形状\[minibatch, size, timeSeriesLength\]，二维输出与形状\[minibatch, size\]。

#### [无监督层](su-cha-biao.md)

* **VariationalAutoencoder** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/variational/VariationalAutoencoder.java)\) 一种用于编码器和解码器的MLP/稠密层的变分自编码器实现。支持多种不同类型的[重构分布](https://github.com/deeplearning4j/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/variational)。
* **AutoEncoder** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/AutoEncoder.java)\) - 标准降噪自动编码器层

#### [其它层](su-cha-biao.md)

* **GlobalPoolingLayer** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/GlobalPoolingLayer.java)\) - 实现基于时间的池化（对于RNs/时间序列-输入大小\[minibatch，size，timeSeriesLength\]、out\[minibatch，size\]）和全局空间池化（对于CNN-输入大小\[minibatch，.，h，w\]、out\[minibatch，.\]）。可用的池模式：和，平均，最大和p-范数。
* **ActivationLayer** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/ActivationLayer.java)\) - 将激活函数（仅）应用于输入激活。请注意，大多数DL4J层具有作为配置选项内置的激活函数。
* **DropoutLayer** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/DropoutLayer.java)\) -  实现丢弃的单独的层。注意大多数 DL4J层有一个内置的丢弃配置选项。
* **BatchNormalization** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/BatchNormalization.java)\) - 二维（前馈），三维（时间系列）或4维（卷积神经网络）激少的批量归一化。对于时间系列，参数是跨时间共享的；对于卷积神经网络，参数是跨空间位置（不是深度）共享的。
* **LocalResponseNormalization** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LocalResponseNormalization.java)\) - 卷积神经网络的本地响应归一化层。在现代的卷积神经网络架构中不经常用。
* **FrozenLayer** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/misc/FrozenLayer.java)\) - 通常不会被用户直接使用-被作为迁移学习的一部份添加，用于冻结层在将来的训练中不再改变的参数。

#### [图顶点](su-cha-biao.md)

图顶点: 与 ComputationGraph 一起使用。和层类似，顶点通常没有任何参数，并可以支持多个输入。

* **ElementWiseVertex** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/ElementWiseVertex.java)\) - 对输入进行元素操作-加法、减法、乘积、平均值、最大值
* **L2NormalizeVertex** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/L2NormalizeVertex.java)\) - 通过对每个示例除以L2范数来归一化输入激活。即，out＜-out／L2范数（out）
* **L2Vertex** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/L2Vertex.java)\) - 为每个示例分别计算两个输入阵列之间的L2距离。对于每个输入值，输出是一个单一值。
* **MergeVertex** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/L2Vertex.java)\) - 将输入激活沿维度1合并，以生成更大的输出数组。对于CNNs，它实现沿深度/通道维度的合并。
* **PreprocessorVertex** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/PreprocessorVertex.java)\) - 包括一个输入预处理器的简单的图顶点
* **ReshapeVertex** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/ReshapeVertex.java)\) - 执行任意激活阵列整形。下一节中的预处理器通常是首选的。
* **ScaleVertex** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/ScaleVertex.java)\) - 实现输入的简单乘法缩放，即OUT =标量\*输入。
* **ShiftVertex** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/ShiftVertex.java)\) - 在输入上实现简单的标量元素添加（即，out＝输入+标量）。
* **StackVertex** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/StackVertex.java)\) - 用于按小批量的维度堆叠所有输入。类似于MergeVertex，但沿维度0（小批量）而不是维度1（输出/通道）
* **SubsetVertex** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/SubsetVertex.java)\) - 用于获得沿维度1的输入激活的连续子集。例如，可以使用两个SubsetVertex实例来将激活从输入数组分割为两个单独的激活。本质上与MergeVertex是相反的。
* **UnstackVertex** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/UnstackVertex.java)\) - 与SubsetVertex类似，但沿维度0（小批量）而不是维度1（输出/通道）。与StackVertex相反。

### [输入预处理器](su-cha-biao.md)

输入预处理器是一个简单的类/接口，它对一个层的输入进行操作。也就是说，预处理器连接到一个层上，并在输入到输出之前对输入执行一些操作。预处理器还处理反向传播——即，预处理操作一般是可求导的。

请注意，在许多情况下（例如XtoYPreProcessor类），用户不需要（也不应该）手动添加这些，而只能使用.setInputType\(InputType.feedForward\(10\)\)来代替，这会根据需要推断和添加预处理器。

* **CnnToFeedForwardPreProcessor** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/preprocessor/CnnToFeedForwardPreProcessor.java)\) - 对一个卷积层\(ConvolutionLayer, SubsamplingLayer, etc\) 到 DenseLayer/OutputLayer的转换做必要的激活修正处理。
* **CnnToRnnPreProcessor** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/preprocessor/CnnToRnnPreProcessor.java)\) - 对一个卷积神经网络层到循环神经网络层的转换做必要的激活修正处理。
* **ComposableInputPreProcessor** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/preprocessor/ComposableInputPreProcessor.java)\) - 一个简单的类，允许多个预处理器链接在单个层上。
* **FeedForwardToCnnPreProcessor** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/preprocessor/FeedForwardToCnnPreProcessor.java)\) - 对一个行向量到一个卷积网络层的转换做激活修正处理。注意这种转换或预处理仅在激活为真实的卷积神经网络激活，但已被扁平化为一个行向量。
* **FeedForwardToRnnPreProcessor** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/preprocessor/FeedForwardToRnnPreProcessor.java)\) - 处理从（时间分布）前馈层到RNN层的转换。
* **RnnToCnnPreProcessor** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/preprocessor/RnnToCnnPreProcessor.java)\) - 处理从具有形状\[minibatch，._._.，timeSeriesLength\]格式的CNN激活序列到时间分布\[numExam.\*timeSeriesLength，numChannels，inputWidth，inputHeight\]格式的转换。
* **RnnToFeedForwardPreProcessor** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/preprocessor/RnnToFeedForwardPreProcessor.java)\) - 处理从时间序列激活\(.\[minibatch,size,timeSeriesLength\]\)到时分布前馈\(.\[minibatch\*tsLength,size\]\)激活的转换。

### [迭代/训练监听器](su-cha-biao.md)

迭代监听器：可以附加到模型，并在训练期间调用，在每次迭代之后（即，在每次参数更新之后）。训练监听器：迭代监听器的扩展。在训练的不同阶段调用许多附加方法。即在向前传递、梯度计算之后，在每次训练开始或结束。

没有（迭代/训练）在训练之外（即在输出或前馈方法中）被调用。

* **ScoreIterationListener** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/ScoreIterationListener.java), Javadoc\) - 记录每隔n次训练迭代的损失函数评分。
* **PerformanceListener** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/PerformanceListener.java), Javadoc\) -记录每N次训练迭代的性能（每秒钟多少示例，每秒钟多少微批次，ETL时间）并可以选择评分
* **EvaluativeListener** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/EvaluativeListener.java), Javadoc\) -  在一个测试集上评估每N次迭代或训练的网络性能。也有一个回调系统，来保存评估结果。
* **CheckpointListener** - \([Source](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/listeners/checkpoint/CheckpointListener.java), Javadoc\) - 周期性的保存网络检查点-基于训练，迭代或时间（或这三个中的组合）
* **StatsListener** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-ui-parent/deeplearning4j-ui-model/src/main/java/org/deeplearning4j/ui/stats/StatsListener.java)\) - DL4J的基于网页的神经网络训练用户界面的主要监听器。查看[可视化页面](../tiao-you-yu-xun-lian/ke-shi-hua.md)来获取详情。
* **CollectScoresIterationListener** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/CollectScoresIterationListener.java), Javadoc\) -  与ScoreIterationListener类似，但在一个本地的列表中保存评分（用于之后获取），而不是记录评分
* **TimeIterationListener** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/TimeIterationListener.java), Javadoc\) - 试图，基于当前的速度和指定的迭代次数估算训练完成之前的时间。

### [评估](su-cha-biao.md)

链接: [主要的评估页](../tiao-you-yu-xun-lian/ping-gu.md)

DL4J具有用于评估网络性能的多个类，与测试集相对应。不同的评估类适合于不同类型的网络。

* **Evaluation** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/Evaluation.java)\) - 用于多类分类器的评估（假设标准one-hot标签，以及N类上的软最大概率分布用于预测）。计算一些度量-正确率，精确率，召回，F1，Fβ，马休斯相关系数，混淆矩阵。可选地计算前N正确率、自定义二分类决策阈值和成本数组（对于非二分类情况）。通常用于软最大 + 麦克森特/负对数似然网络。
* **EvaluationBinary** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/EvaluationBinary.java)\) -评估类的多标签二分类版本。假设每个网络输出是独立的/独立的二分类，概率0到1与所有其他输出无关。通常用于sigmoid+二值交叉熵网络。
* **EvaluationCalibration** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/EvaluationCalibration.java)\) - 用于评价二分类或多类分类器的校准。产生可靠性图、残差图和概率直方图。使用EvaluationTools.exportevaluationCalibrationToHtmlFile 方法导出图表到HTML
* **ROC** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/ROC.java)\) - 仅用于单输出二分类器。即，具有nOut\(1\) + sigmoid, 或 nOut\(2\) + softmax。支持2种模式：阈值（近似）或精确（默认）。计算ROC曲线下面积，精确召回曲线下面积。使用EvaluationTools绘制ROC和P-R曲线到HTML。
* **ROCBinary** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/ROCBinary.java)\) - 一个用于多标签二分类网络（即 sigmoid + 二值交叉熵）的ROC版本，它的每个网络的输出假被为一个独立的二分类变量。
* **ROCMultiClass** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/ROCMultiClass.java)\) - 一个用于多类（非二分类）网络的ROC版本。 \(即, softmax + 麦克森特/负对数似然 网络\)。由于ROC度量仅定义为二分类，因此将多类输出视为一组“一对所有”的二分类问题。
* **RegressionEvaluation** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/RegressionEvaluation.java)\) - 一个用于回归模型的评估类（包括多输出回归模型）。报告每个输出的度量，例如均方误差（MSE）、平均绝对误差等。

### [网络的保存与加载](su-cha-biao.md)

可以使用ModelSerializer类，特别是writeModel、restoreMultiLayerNetwork和restoreComputationGraph方法来保存多层网络和计算图。

对于当前主干（但不是0.9.1） MultiLayerNetwork.save\(File\)方法 和 MultiLayerNetwork.load\(File\) 方法已被添加。这些在内部使用ModelSerializer。计算图也增加了类似的保存/加载方法。

示例: [加载与保存网络](https://github.com/deeplearning4j/dl4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/modelsaving)

网络可以在保存和加载之后进一步训练：但是，请确保加载“更新器”（即，更新器的历史状态，如momentum）。如果不需要进一步的训练，则更新器状态可以被忽略以节省磁盘空间和内存。

大多数归一化器\(实现ND4J Normalizer接口\)也可以使用addNormalizerToModel方法添加到模型中。

注意，DL4J中用于模型的格式是.zip：可以使用支持zip格式的程序打开/提取这些文件。

### [网络配置](su-cha-biao.md)

本节列出了DL4J支持的各种配置选项。

#### [激活函数](su-cha-biao.md)

激活函数可以用两种方式之一定义：\(a\)通过向配置传递激活枚举值，例如，.activation\(Activation.TANH\)\(b\)通过传递IActivation实例，例如，.activation\(new ActivationSigmoid\(\)\)。

注意，DL4J支持自定义激活函数，它可以通过扩展[BaseActivationFunction](https://github.com/deeplearning4j/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl)来定义。

支持的激活函数列表:

* **CUBE** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationCube.java)\) - `f(x) = x^3`
* **ELU** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationELU.java)\) - 指数线性单位\([参考文献](https://arxiv.org/abs/1511.07289)\)
* **HARDSIGMOID** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationHardSigmoid.java)\) - 标准sigmoid激活函数的分段线性化. `f(x) = min(1, max(0, 0.2*x + 0.5))`
* **HARDTANH** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationHardTanH.java)\) - 标准 tanh 激活函数的分段线性化.
* **IDENTITY** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationIdentity.java)\) - 一个“无运算”激活函数: `f(x) = x`
* **LEAKYRELU** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationLReLU.java)\) - 漏校正线性单元. `f(x) = max(0, x) + alpha * min(0, x)` 默认的 `alpha=0.01` .
* **RATIONALTANH** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationRationalTanh.java)\) - `tanh(y) ~ sgn(y) * { 1 - 1/(1+|y|+y^2+1.41645*y^4)}` 近似于 `f(x) = 1.7159 * tanh(2x/3)`, 但执行起来更快. \([参考文献](https://arxiv.org/abs/1508.01292)\)
* **RELU** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationReLU.java)\) - 标准校正线性单元: `f(x) = x` if `x>0` 或 `f(x) = 0` 
* **RRELU** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationRReLU.java)\) - 随机校正线性单位。在测试期间有确定性. \([参考文献](https://arxiv.org/abs/1505.00853)\)
* **SIGMOID** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationSigmoid.java)\) - 标准的 sigmoid 激活函数, `f(x) = 1 / (1 + exp(-x))`
* **SOFTMAX** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationSoftmax.java)\) - 标准的 softmax 激活函数
* **SOFTPLUS** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationSoftPlus.java)\) - `f(x) = log(1+e^x)` - 形状类似于RELU 激活函数的平滑版本
* **SOFTSIGN** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationSoftSign.java)\) - `f(x) = x / (1+|x|)` - 形状类似于标准的 tanh 激活函数 \(计算更快\).
* **TANH** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationTanH.java)\) - 标准的 tanh \(双曲正切\) 激活函数
* **RECTIFIEDTANH** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationRectifiedTanh.java)\) - `f(x) = max(0, tanh(x))`
* **SELU** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationSELU.java)\) - 比例指数线性单位，与自归一化神经网络一起使用
* **SWISH** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/impl/ActivationSwish.java)\) - Swish 激活函数, `f(x) = x * sigmoid(x)`\([参考文献](https://arxiv.org/abs/1710.05941)\)

#### [权重初始化](su-cha-biao.md)

权值初始化指的是一个新网络的初始参数应该被设置的方法。

权重初始化通常使用WeightInit枚举来定义。

自定义权重初始化可以使用 `.weightInit(WeightInit.DISTRIBUTION).dist(new NormalDistribution(0, 1))` 例如. 对于主干 \(非 0.9.1 版本\) `.weightInit(new NormalDistribution(0, 1))` 也是可用的, 这相当于以前的方法。

可用的权重初始化。并不是所有的版本都在0.9.1版本中可用：

* **DISTRIBUTION**: Sample weights from a provided distribution 从给定的分布获取权重样例 \(specified 通过  `dist` 配置方法来指定）
* **ZERO**: 生成权重为零
* **ONES**: 所有权重设为1
* **SIGMOID\_UNIFORM**: sigmoid激活函数的一个XAVIER\_UNIFORM版本。 U\(-r,r\) with r=4\*sqrt\(6/\(fanIn + fanOut\)\)
* **NORMAL**:  均值为0，标准差为 1/sqrt\(fanIn\)的 正态/高斯分布。这是Klambauer等人提出的初始化，2017、“自归一化神经网络”论文。相当于 DL4J’的  XAVIER\_FAN\_IN 和  LECUN\_NORMAL \(即. Keras 的  “lecun\_normal”\)
* **LECUN\_UNIFORM**: U\[-a,a\] 与 a=3/sqrt\(fanIn\)保持统一
* **UNIFORM**: U\[-a,a\] 与  a=1/sqrt\(fanIn\)保持统一。 Glorot和BeNIO 2010的“常用启发式”
* **XAVIER**: As per [Glorot and Bengio 2010](http://proceedings.mlr.press/v9/glorot10a/glorot10a.pdf): 均值 0, 方差为 2.0/\(fanIn + fanOut\)的高斯分布
* **XAVIER\_UNIFORM**: As per [Glorot and Bengio 2010](http://proceedings.mlr.press/v9/glorot10a/glorot10a.pdf): 分布 U\(-s,s\) 与  s = sqrt\(6/\(fanIn + fanOut\)\)保持统一
* **XAVIER\_FAN\_IN**: 类似于Xavier, 除了 1/fanIn -&gt; Caffe 原来用过这个.
* **RELU**: [He et al. \(2015\), “深入研究整流器”](https://arxiv.org/abs/1502.01852). 方差为2.0/nIn的正态分布
* **RELU\_UNIFORM**: [He et al. \(2015\), “深入研究整流器”](https://arxiv.org/abs/1502.01852). 分布 U\(-s,s\) 与 s = sqrt\(6/fanIn\)保持统一
* **IDENTITY**: 权重被设置为单位矩阵。注：只能与平方权重矩阵一起使用。
* **VAR\_SCALING\_NORMAL\_FAN\_IN**: 均值0，方差为1.0/\(fanIn\)的高斯分布
* **VAR\_SCALING\_NORMAL\_FAN\_OUT**: 均值0，方差为1.0/\(fanOut\)的高斯分布
* **VAR\_SCALING\_NORMAL\_FAN\_AVG**: 均值0，方差为1.0/\(\(fanIn + fanOut\)/2\)的高斯分布
* **VAR\_SCALING\_UNIFORM\_FAN\_IN**: U\[-a,a\] 与 a=3.0/\(fanIn\)保持统一
* **VAR\_SCALING\_UNIFORM\_FAN\_OUT**: U\[-a,a\] 与 a=3.0/\(fanOut\)保持统一
* **VAR\_SCALING\_UNIFORM\_FAN\_AVG**:U\[-a,a\] 与 a=3.0/\(\(fanIn + fanOut\)/2\)保持统一

#### [更新器 \(优化器\)](su-cha-biao.md)

DL4J中的“更新器”是一个需要原始梯度并将其修改为更新的类。然后将这些更新应用于网络参数。这篇[CS231n 课程笔记](https://cs231n.github.io/neural-networks-3/#ada)对这些更新器有很好的解释。

DL4J支持的更新器:

* **AdaDelta** - \([Source](https://github.com/deeplearning4j/nd4j/blob/e92cae4626e2c9cf27d416136f1895b747a32cee/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/config/AdaDelta.java)\) - [Reference](https://arxiv.org/abs/1212.5701)
* **AdaGrad** - \([Source](https://github.com/deeplearning4j/nd4j/blob/e92cae4626e2c9cf27d416136f1895b747a32cee/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/config/AdaGrad.java)\) - [Reference](http://jmlr.org/papers/v12/duchi11a.html)
* **AdaMax** - \([Source](https://github.com/deeplearning4j/nd4j/blob/e92cae4626e2c9cf27d416136f1895b747a32cee/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/config/AdaMax.java)\) - Adam 更新器的一个变体 - [参考文献](https://arxiv.org/abs/1412.6980)
* **Adam** - \([Source](https://github.com/deeplearning4j/nd4j/blob/e92cae4626e2c9cf27d416136f1895b747a32cee/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/config/Adam.java)\)
* **Nadam** - \([Source](https://github.com/deeplearning4j/nd4j/blob/e92cae4626e2c9cf27d416136f1895b747a32cee/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/config/Nadam.java)\) - Adam 更新器的一个变体，使用牛顿动量更新规则 - [参考文献](https://arxiv.org/abs/1609.04747)
* **Nesterovs** - \([Source](https://github.com/deeplearning4j/nd4j/blob/e92cae4626e2c9cf27d416136f1895b747a32cee/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/config/Nesterovs.java)\) - 牛顿动量更新器
* **NoOp** - \([Source](https://github.com/deeplearning4j/nd4j/blob/e92cae4626e2c9cf27d416136f1895b747a32cee/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/config/NoOp.java)\) - 一个“无操作”更新程序。也就是说，梯度不会被这个更新器修改。数学等价于学习率为1的SGD更新器
* **RmsProp** - \([Source](https://github.com/deeplearning4j/nd4j/blob/e92cae4626e2c9cf27d416136f1895b747a32cee/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/config/RmsProp.java)\) - [参考文献 ](http://www.cs.toronto.edu/~tijmen/csc321/slides/lecture_slides_lec6.pdf)
* **Sgd** - \([Source](https://github.com/deeplearning4j/nd4j/blob/e92cae4626e2c9cf27d416136f1895b747a32cee/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/config/Sgd.java)\) - 标准随机梯度下降更新器。此更新器仅适用学习速率。

#### [学习调度](su-cha-biao.md)

支持学习速率的所有更新器也支持学习速率调度（牛顿动量更新器也支持动量调度）。学习速率调度可以根据迭代次数或已逝去的训练数来指定。Dropout（见下文）也可以利用这里列出的调度表。

配置用法，例如: `.updater(new Adam(new ExponentialSchedule(ScheduleType.ITERATION, 0.1, 0.99 )))` 你可以在你创建的调度对象上通过调用`ISchedule.valueAt(int iteration, int epoch)` 来制图／监视将在任意点使用的学习率。

可用的调度:

* **ExponentialSchedule** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/schedule/ExponentialSchedule.java)\) - 实现 `value(i) = initialValue * gamma^i`
* **InverseSchedule** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/schedule/InverseSchedule.java)\) - 实现 `value(i) = initialValue * (1 + gamma * i)^(-power)`
* **MapSchedule** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/schedule/MapSchedule.java)\) - 基于用户提供的映射的学习率调度。注意所提供的映射必须有一个用于迭代／训练 0次 的值。有一个构建器类来方便的定义一个调度。
* **PolySchedule** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/schedule/PolySchedule.java)\) - 实现 `value(i) = initialValue * (1 + i/maxIter)^(-power)`
* **SigmoidSchedule** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/schedule/SigmoidSchedule.java)\) - 实现 `value(i) = initialValue * 1.0 / (1 + exp(-gamma * (iter - stepSize)))`
* **StepSchedule** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/schedule/StepSchedule.java)\) - 实现 `value(i) = initialValue * gamma^( floor(iter/step) )`

请注意，自定义调度可以通过实现ISchedule接口来创建。

#### [正则化](su-cha-biao.md)

#### [L1/L2 正则化](su-cha-biao.md)

L1和L2正则化可以容易地通过配置：.l1\(0.1\).l2\(0.2\)添加到网络中。注意， .regularization\(true\) 必须在0.9.1上启用（这个选项在0.9.1发布后被删除）。 L1和L2正则化仅适用于权重参数。也就是说，.l1 和 .l2 不会影响偏置参数-这些可以使用.l1Bias\(0.1\).l2Bias\(0.2\)实现被正则化。

#### [Dropout（丢弃）](su-cha-biao.md)

所有的丢弃类型公在训练时应用。它们不在测试时应用。

* **Dropout** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/dropout/Dropout.java)\) - 每个输入激活X被独立地设置为（0，与概率1-p）或（x/p与概率p）。
* **GaussianDropout** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/dropout/GaussianDropout.java)\) - 这是一个输入激活上的乘法高斯噪声（均值1\)。每个输入激活X独立地设置为：x \* y,     y ~ N\(1, stdev = sqrt\(\(1-rate\)/rate\)\)
* **GaussianNoise** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/dropout/GaussianNoise.java)\) - 将加法，平均零高斯噪声应用于输入-即 x = x + N\(0,stddev\)
* **AlphaDropout** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/dropout/AlphaDropout.java)\) -  AlphaDropout是一个丢弃技术，由[Klaumbauer et al. 2017 - 自归一化神经网络](https://arxiv.org/abs/1706.02515)提出。设计为自归一化神经网络\(SELU 激活函数, NORMAL 权重初始化\)。试图让丢弃后激活的均值和方差与AlphaDropout被应用之前相同。

注意（从当前主干开始，但不是0.9.1），丢弃参数也可以根据学习率调度部分中提到的任何调度类来指定。

#### [权重噪声](su-cha-biao.md)

根据丢弃，丢弃连接/权重噪声只适用于训练时间。

* **DropConnect** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/weightnoise/DropConnect.java)\) -  [参考文献](https://cs.nyu.edu/~wanli/dropc/dropc.pdf)  DropConnect与dropout类似，但应用于网络参数（而不是输入激活）
* **WeightNoise** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/weightnoise/WeightNoise.java)\) - 在训练时把指定分布噪声应用于权重。支持加法和乘法模式。-当 加法时，噪声应当均值为0，当乘法时，噪声均值应当为1。

#### [约束](su-cha-biao.md)

约束是在每次迭代结束时（在参数更新发生之后）放置在模型的参数上的确定性限制。它们可以被认为是正则化的一种类型。

* **MaxNormConstraint** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/constraint/MaxNormConstraint.java)\) - 将每个单元的输入权重的最大L2范数约束为小于或等于指定值。如果L2范数超过指定值，则权重将被缩减以满足约束。
* **MinMaxNormConstraint** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/constraint/MinMaxNormConstraint.java)\) -将每个单元的输入权重的最小和最大L2范数约束在指定值之间。如果需要的话，权重将被放大/缩小。
* **NonNegativeConstraint** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/constraint/NonNegativeConstraint.java)\) - 约束所有参数为非负。负参数将被替换为0。
* **UnitNormConstraint** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/b841c0f549194dbdf88b42836df662d9b8ea8c6d/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/constraint/UnitNormConstraint.java)\) -将每个单元的输入权重的L2范数约束为1。

### [数据类](su-cha-biao.md)

#### [迭代器](su-cha-biao.md)

DataSetIterator是DL4J用于对小批量数据进行迭代的抽象，用于训练。DataSetIterator返回DataSet对象，这些对象是小批量，并支持最多1个输入和1个输出数组（INDArray）。 MultiDataSetIterator类似于DataSetIterator，但是返回MultiDataSet对象，该对象可以具有网络所需的多个输入和多个输出数组。

#### [内置迭代器 \(DL4J-提供数据\)](su-cha-biao.md)

这些迭代器按需要下载它们的数据。它们返回的实际数据集不是可定制的。

* **MnistDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-datasets/src/main/java/org/deeplearning4j/datasets/iterator/impl/MnistDataSetIterator.java)\) - 著名的MNIST数字数据集的DataSetIterator。默认情况下，返回行向量（1x784），其值被归一化为0至1范围。使用.setInputType\(InputType.convolutionalFlat\(\)\)来与CNN一起使用。
* **EmnistDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-datasets/src/main/java/org/deeplearning4j/datasets/iterator/impl/EmnistDataSetIterator.java)\) - 类似于MNIST数字数据集，但有更多的例子，也有字母。包括多个不同的分割（仅字母，数字，字母+数字等）。因此，可以使用与MNIST相同的1x784格式（除了用于某些分割的不同数量的标签之外）作为MnistDataSetIterator的置换置换。 [参考文献 1](https://www.nist.gov/itl/iad/image-group/emnist-dataset), [参考文献2](https://arxiv.org/abs/1702.05373)
* **IrisDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-datasets/src/main/java/org/deeplearning4j/datasets/iterator/impl/IrisDataSetIterator.java)\) -一个众所周知的鸢尾花数据集的迭代器。4个特征，3个输出类。
* **CifarDataSetIterator** - \([Source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-datasets/src/main/java/org/deeplearning4j/datasets/iterator/impl/Cifar10DataSetIterator.java)\) - CIOFAR图像数据集的迭代器。10类，在DL4J中CNNs的4D特征/激活格式：\[minibatch,channels,height,width\] = \[minibatch,3,32,32\]。特征不是归一化的，而是在0到255的范围内。
* **LFWDataSetIterator** - \([Source](https://deeplearning4j.org/docs/latest/deeplearning4j-cheat-sheet)\)
* **TinyImageNetDataSetIterator** \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-datasets/src/main/java/org/deeplearning4j/datasets/iterator/impl/TinyImageNetDataSetIterator.java)\) - 标准IMANET数据集的子集；200个类，每个类500个图像
* **UciSequenceDataSetIterator** \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-datasets/src/main/java/org/deeplearning4j/datasets/iterator/impl/UciSequenceDataSetIterator.java)\) - UCI 综合控制时间序列数据集

#### [迭代器-用户提供的数据](su-cha-biao.md)

此子章节的迭代器与用户提供的数据一起使用。

* **RecordReaderDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-datavec-iterators/src/main/java/org/deeplearning4j/datasets/datavec/RecordReaderDataSetIterator.java)\) - 采用DataVec记录读取器（如CsvRecordReader或ImageRecordReader）并处理到数据集的转换、批处理、屏蔽等的迭代器。DL4J中最常用的迭代器之一。只处理非序列数据，作为输入（即，RecordReader，非SequenceeRecordReader）。
* **RecordReaderMultiDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-datavec-iterators/src/main/java/org/deeplearning4j/datasets/datavec/RecordReaderMultiDataSetIterator.java)\) - RecordReaderDataSetIterator 的MultiDataSet版本, 支持多个读取器。具有用于创建更复杂的数据管道的构建器模式（例如，读取器输出到不同输入/输出阵列的不同子集、转换到一个热点等等）。处理序列和非序列数据作为输入。
* **SequenceRecordReaderDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-datavec-iterators/src/main/java/org/deeplearning4j/datasets/datavec/SequenceRecordReaderDataSetIterator.java)\) - RecordReaderDataSetIterator 的sequence \(SequenceRecordReader\)  版本。 用户最好结合RecordReaderMultiDataSetIterator使用。
* **DoublesDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/DoublesDataSetIterator.java)\)
* **FloatsDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/FloatsDataSetIterator.java)\)
* **INDArrayDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/INDArrayDataSetIterator.java)\)

#### [迭代器 - 适配器与实用迭代器](su-cha-biao.md)

* **MultiDataSetIteratorAdapter** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/datasets/iterator/impl/MultiDataSetIteratorAdapter.java)\) - 包装一个 DataSetIterator来转换为一个MultiDataSetIterator
* **SingletonMultiDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/impl/SingletonMultiDataSetIterator.java)\) - 包装一个MultiDataSet 转换为一个 MultiDataSetIterator 并返回一个 MultiDataSet \(即, 包装的MultiDataSet是不可分割的\)
* **AsyncDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/AsyncDataSetIterator.java)\) - 在适当的情况下由多层网络和计算图自动使用。实现数据集的异步预获取以提高性能。
* **AsyncMultiDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/AsyncMultiDataSetIterator.java)\) - 在适当的情况下由计算图自动使用。实现多数据集的异步预获取以提高性能。
* **AsyncShieldDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/AsyncShieldDataSetIterator.java)\) - 通常只用于调试。使用AsyncDataSetIterator来停止多层网络和计算图。
* **AsyncShieldMultiDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/AsyncShieldMultiDataSetIterator.java)\) -  AsyncShieldDataSetIterator 的 MultiDataSetIterator 版本。
* **EarlyTerminationDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/EarlyTerminationDataSetIterator.java)\) - 包装另一个DataSetIterator，确保在重置之间仅返回指定（最大）数量的小批量（DataSet）对象。可以用来“剪短”一个迭代器，只返回前N个数据集。
* **EarlyTerminationMultiDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/EarlyTerminationMultiDataSetIterator.java)\) - EarlyTerminationDataSetIterator的MultiDataSetIterator版本 
* **ExistingDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/ExistingDataSetIterator.java)\) - 转换一个 `Iterator<DataSet>` 或 `Iterable<DataSet>` 为 一个 DataSetIterator。 不拆分基础数据集对象
* **FileDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/file/FileDataSetIterator.java)\) - 一个迭代器，用于迭代以前用 DataSet.save\(File\)保存的DataSet文件。支持随机化、过滤、不同的输出批量大小与保存的数据集批量大小等。
* **FileMultiDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/file/FileMultiDataSetIterator.java)\) - FileDataSetIterator的MultiDataSet版本。
* **IteratorDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/IteratorDataSetIterator.java)\) - 转换一个 `Iterator<DataSet>` 为一个 DataSetIterator. 与ExistingDataSetIterator不同，底层DataSet对象可以是拆分/组合的——即，对于输出，小批量大小可能与输入迭代器不同。
* **IteratorMultiDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/IteratorMultiDataSetIterator.java)\) - IteratorDataSetIterator 的`Iterator<MultiDataSet>版本`
* **MultiDataSetWrapperIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/MultiDataSetWrapperIterator.java)\) - 转换一个MultiDataSetIterator 为一个 DataSetIterator。 注意，如果特征和标签数组的数量等于1，才是可能的。
* **MultipleEpochsIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/MultipleEpochsIterator.java)\) - 当训练时，将基础迭代器的多次训练视为单个训练。
* **WorkspaceShieldDataSetIterator** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/WorkspacesShieldDataSetIterator.java)\) - 通常只用于调试，而通常不由用户使用。分离/迁移来自底层DataSetIterator的数据集。

### [数据归一化](su-cha-biao.md)

ND4J提供了用于执行数据归一化的多个类。这些实现为数据集预处理器。归一化的基本模式：

1. 创建你的 \(非归一化\) DataSetIterator 或 MultiDataSetIterator: `DataSetIterator myTrainData = ...`
2. 创建你想使用的归一化器: `NormalizerMinMaxScaler normalizer = new NormalizerMinMaxScaler();`
3. 拟合归一化器: `normalizer.fit(myTrainData)`
4. 在迭代器上设置归一化器／预处理器 : `myTrainData.setPreProcessor(normalizer);` 最终结果：来自DataSetIterator的数据现在将被归一化。

通常你应该只在训练数据上拟合，并且与仅在训练数据上拟合的相同的/单一的归一化器一起执行 `trainData.setPreProcessor(normalizer)` 和 `testData.setPreProcessor(normalizer)`

注意，在适当的情况下（NormalizerStandard.，NormalizerMinMaxScaler），诸如平均值/标准偏差/最小值/最小值的统计数据，跨时间（对于时间序列）和跨图像x/y位置（但是对于图像数据不是深度/通道）共享。

数据归一化示例: [链接](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/dataexamples/PreprocessNormalizerExample.java)

**可用的归一化器: DataSet / DataSetIterator**

* **ImagePreProcessingScaler** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/dataset/api/preprocessor/ImagePreProcessingScaler.java)\) - 应用最小最大缩放到图像激活。默认设置将0到255输入到0-1输出（但是是可配置的）。注意，与这里的其他归一化器不同，该归一化器不依赖于从数据收集的统计数据\(均值/最小值/最大值 等\)，因此normalizer.fit\(trainData\)步骤是不必要的\(是非操作性的\)。
* **NormalizerStandardize** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/dataset/api/preprocessor/NormalizerStandardize.java)\) - 独立地将每个特征值（和可选的标签值）归一化为0平均值和1的标准差。
* **NormalizerMinMaxScaler** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/dataset/api/preprocessor/NormalizerMinMaxScaler.java)\) - 独立归一化每个特征值（以及可选的标签值），使其位于最小值和最大值之间（默认情况下在 0和1之间）
* **VGG16ImagePreProcessor** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/dataset/api/preprocessor/VGG16ImagePreProcessor.java)\) - 这是一个专门用于VG16的预处理器。在训练集上计算，减去每个像素RGB的平均值，如在[链接](https://arxiv.org/pdf/1409.1556.pdf)中所报告的。 

**可用的归一化器: MultiDataSet / MultiDataSetIterator**

* **ImageMultiPreProcessingScaler** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/dataset/api/preprocessor/ImageMultiPreProcessingScaler.java)\) - ImagePreProcessingScaler的MultiDataSet/MultiDataSetIterator版本
* **MultiNormalizerStandardize** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/dataset/api/preprocessor/MultiNormalizerStandardize.java)\) - NormalizerStandardize的MultiDataSet/MultiDataSetIterator版本
* **MultiNormalizerMinMaxScaler** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/dataset/api/preprocessor/MultiNormalizerMinMaxScaler.java)\) - NormalizerMinMaxScaler的 MultiDataSet/MultiDataSetIterator 版本
* **MultiNormalizerHybrid** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/dataset/api/preprocessor/MultiNormalizerHybrid.java)\) - 一个 MultiDataSet归一化器，可以为 不同的 输入/特征 和输出/标签 数组组合不同的归一化类型（标准化，最小／最大化） 。

### [迁移学习](su-cha-biao.md)

DL4j具有用于执行迁移学习的类/实用程序——即，采用现有网络，并修改一些层（可选地冻结其他层，以便它们的参数不改变）。例如，可以在ImageNet上训练图像分类器，然后应用于新的/不同的数据集。多层网络和计算图都可以与迁移学习一起使用——通常从模型动物园的预训练模型开始（参见下一节），虽然可以单独使用任何多层网络/计算图。

链接: [迁移学习示例](https://github.com/deeplearning4j/dl4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/transferlearning/vgg16)

迁移学习的主要类别是[TransferLearning](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/transferlearning/TransferLearning.java)。该类具有可用于添加/删除层、冻结层等的构建器模式。[FineTuneConfiguration](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/transferlearning/FineTuneConfiguration.java)可用于指定非冻结层的学习速率和其他设置。

### [已训练的模型库 - Model Zoo](su-cha-biao.md)

DL4J提供了一个“model zoo”——一组预训练模型，可以下载和使用（例如，用于图像分类），或者经常用于迁移学习。

链接: [Deeplearning4j Model Zoo](../mo-xing/dong-wu-yuan-yong-fa.md)

DL4J 的 model zoo中可用的模型有:

* **AlexNet** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/AlexNet.java)\)
* **Darknet19** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/Darknet19.java)\)
* **FaceNetNN4Small2** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/FaceNetNN4Small2.java)\)
* **InceptionResNetV1** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/InceptionResNetV1.java)\)
* **LeNet** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/LeNet.java)\)
* **ResNet50** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/ResNet50.java)\)
* **SimpleCNN** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/SimpleCNN.java)\)
* **TextGenerationLSTM** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/TextGenerationLSTM.java)\)
* **TinyYOLO** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/TinyYOLO.java)\)
* **VGG16** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/VGG16.java)\)
* **VGG19** - \([Source](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/VGG19.java)\)

_\*注_: Keras 已训练好的模型 \(不是 DL4J 提供\) 或许也可以导入, 使用 DL4J的 Keras 模型导入功能。

## [速查表代码片段](su-cha-biao.md)

Eclipse DL4J库提供了很多功能，我们将这个速查表放在一起，以帮助用户组装神经网络并更快地使用张量。

### [神经网络](su-cha-biao.md)

用于多层网络和计算图的通用参数和层的配置代码。完整的API见[MultiLayerNetwork](https://deeplearning4j.org/api/1.0.0-beta2/org/deeplearning4j/nn/multilayer/MultiLayerNetwork.html)和[ComputationGraph](https://deeplearning4j.org/api/1.0.0-beta2/org/deeplearning4j/nn/graph/ComputationGraph.html)。

**序列网络**

大多数网络配置可以使用多层网络类，如果它们是序列的和简单的。

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(1234)
    // 如下的参数会被复制到网络中的每一层
    // 对于像 dropOut() 或 activation()这样的参数你应该每一层都设置
    // 只指定你需要的参数
    .updater(new AdaGrad())
    .activation(Activation.RELU)
    .dropOut(0.8)
    .l1(0.001)
    .l2(1e-4)
    .weightInit(WeightInit.XAVIER)
    .weightInit(Distribution.TruncatedNormalDistribution)
    .cudnnAlgoMode(ConvolutionLayer.AlgoMode.PREFER_FASTEST)
    .gradientNormalization(GradientNormalization.RenormalizeL2PerLayer)
    .gradientNormalizationThreshold(1e-3)
    .list()
    // 网络中的层，按顺序添加
    // 每层设置的参数覆盖上面设置的参数
    .layer(new DenseLayer.Builder().nIn(numInputs).nOut(numHiddenNodes)
            .weightInit(WeightInit.XAVIER)
            .build())
    .layer(new ActivationLayer(Activation.RELU))
    .layer(new ConvolutionLayer.Builder(1,1)
            .nIn(1024)
            .nOut(2048)
            .stride(1,1)
            .convolutionMode(ConvolutionMode.Same)
            .weightInit(WeightInit.XAVIER)
            .activation(Activation.IDENTITY)
            .build())
    .layer(new GravesLSTM.Builder()
            .activation(Activation.TANH)
            .nIn(inputNum)
            .nOut(100)
            .build())
    .layer(new OutputLayer.Builder(LossFunction.NEGATIVELOGLIKELIHOOD)
            .weightInit(WeightInit.XAVIER)
            .activation(Activation.SOFTMAX)
            .nIn(numHiddenNodes).nOut(numOutputs).build())
    .pretrain(false).backprop(true)
    .build();

MultiLayerNetwork neuralNetwork = new MultiLayerNetwork(conf);
```

**复杂网络**

具有复杂图和“分支”的网络需要使用计算图。

```text
ComputationGraphConfiguration.GraphBuilder graph = new NeuralNetConfiguration.Builder()
    .seed(seed)
   // 如下的参数会被复制到网络中的每一层
    // 对于像 dropOut() 或 activation()这样的参数你应该每一层都设置
    // 只指定你需要的参数  
    .activation(Activation.IDENTITY)
    .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
    .updater(updater)
    .weightInit(WeightInit.RELU)
    .l2(5e-5)
    .miniBatch(true)
    .cacheMode(cacheMode)
    .trainingWorkspaceMode(workspaceMode)
    .inferenceWorkspaceMode(workspaceMode)
    .cudnnAlgoMode(cudnnAlgoMode)
    .convolutionMode(ConvolutionMode.Same)
    .graphBuilder()
    // 网络中的层，按顺序添加
    // 每层设置的参数覆盖上面设置的参数
    // 注意你必须为每一层命名并手动指定它的输入
    .addInputs("input1")
    .addLayer("stem-cnn1", new ConvolutionLayer.Builder(new int[] {7, 7}, new int[] {2, 2}, new int[] {3, 3})
        .nIn(inputShape[0])
        .nOut(64)
        .cudnnAlgoMode(ConvolutionLayer.AlgoMode.NO_WORKSPACE)
        .build(),"input1")
    .addLayer("stem-batch1", new BatchNormalization.Builder(false)
        .nIn(64)
        .nOut(64)
        .build(), "stem-cnn1")
    .addLayer("stem-activation1", new ActivationLayer.Builder()
        .activation(Activation.RELU)
        .build(), "stem-batch1")
    .addLayer("lossLayer", new CenterLossOutputLayer.Builder()
        .lossFunction(LossFunctions.LossFunction.SQUARED_LOSS)
        .activation(Activation.SOFTMAX).nOut(numClasses).lambda(1e-4).alpha(0.9)
        .gradientNormalization(GradientNormalization.RenormalizeL2PerLayer).build(),
        "stem-activation1")
    .setOutputs("lossLayer")
    .setInputTypes(InputType.convolutional(224, 224, 3))
    .backprop(true).pretrain(false).build();

ComputationGraph neuralNetwork = new ComputationGraph(graph);
```

### [训练](su-cha-biao.md)

下面的代码片段创建一个基本的管道，从磁盘加载图像，应用随机变换，并将它们拟合到神经网络。它还设置了UI实例，以便你可以可视化进度，并使用早期停止来提前终止训练。你可以为许多不同的用例修改此管道。

```java
ParentPathLabelGenerator labelMaker = new ParentPathLabelGenerator();
File mainPath = new File(System.getProperty("user.dir"), "dl4j-examples/src/main/resources/animals/");
FileSplit fileSplit = new FileSplit(mainPath, NativeImageLoader.ALLOWED_FORMATS, rng);
int numExamples = Math.toIntExact(fileSplit.length());
// 在仅在你的根目录是干净的：只有标签子目录的时候才会起作用。
int numLabels = fileSplit.getRootDir().listFiles(File::isDirectory).length; 
BalancedPathFilter pathFilter = new BalancedPathFilter(rng, labelMaker, numExamples, numLabels, maxPathsPerLabel);

InputSplit[] inputSplit = fileSplit.sample(pathFilter, splitTrainTest, 1 - splitTrainTest);
InputSplit trainData = inputSplit[0];
InputSplit testData = inputSplit[1];

boolean shuffle = false;
ImageTransform flipTransform1 = new FlipImageTransform(rng);
ImageTransform flipTransform2 = new FlipImageTransform(new Random(123));
ImageTransform warpTransform = new WarpImageTransform(rng, 42);
List<Pair<ImageTransform,Double>> pipeline = Arrays.asList(
    new Pair<>(flipTransform1,0.9),
    new Pair<>(flipTransform2,0.8),
    new Pair<>(warpTransform,0.5));

ImageTransform transform = new PipelineImageTransform(pipeline,shuffle);
DataNormalization scaler = new ImagePreProcessingScaler(0, 1);

// 训练数据集
ImageRecordReader recordReaderTrain = new ImageRecordReader(height, width, channels, labelMaker);
recordReader.initialize(trainData, null);
DataSetIterator trainingIterator = new RecordReaderDataSetIterator(recordReaderTrain, batchSize, 1, numLabels);

//测试数据集
ImageRecordReader recordReaderTest = new ImageRecordReader(height, width, channels, labelMaker);
recordReader.initialize(testData, null);
DataSetIterator testingIterator = new RecordReaderDataSetIterator(recordReaderTest, batchSize, 1, numLabels);

//早停配置，模型保存器，还有训练器
EarlyStoppingModelSaver saver = new LocalFileModelSaver(System.getProperty("user.dir"));
EarlyStoppingConfiguration esConf = new EarlyStoppingConfiguration.Builder()
     //最大50轮
    .epochTerminationConditions(new MaxEpochsTerminationCondition(50)) 
    .evaluateEveryNEpochs(1)
    //最多20分钟
    .iterationTerminationConditions(new MaxTimeIterationTerminationCondition(20, TimeUnit.MINUTES)) 
     //计算测试集得分
    .scoreCalculator(new DataSetLossCalculator(testingIterator, true))    
    .modelSaver(saver)
    .build();

EarlyStoppingTrainer trainer = new EarlyStoppingTrainer(esConf, neuralNetwork, trainingIterator);

// 开始训练
trainer.fit();
```

### [复杂的转换](su-cha-biao.md)

DataVec附带了一个便利的转换进程类，允许更复杂的数据冲突和数据转换。它与2D和序列数据集都能很好地工作。

```java
Schema schema = new Schema.Builder()
    .addColumnsDouble("Sepal length", "Sepal width", "Petal length", "Petal width")
    .addColumnCategorical("Species", "Iris-setosa", "Iris-versicolor", "Iris-virginica")
    .build();

TransformProcess tp = new TransformProcess.Builder(schema)
    .categoricalToInteger("Species")
    .build();

// 在spark上进行转换
JavaRDD<List<Writable>> processedData = SparkTransformExecutor.execute(parsedInputData, tp);
```

在创建更复杂的转换之前，我们建议先查看一下 [DataVec examples](https://github.com/deeplearning4j/dl4j-examples/tree/master/datavec-examples/src/main/java/org/datavec/transform)。

### [评估](su-cha-biao.md)

MultiLayerNetwork和ComputationGraph都带有内置的eval\(\)方法，允许你传递数据集迭代器并返回评估结果。

```java
// 返回具有准确度、精确度、召回和其他类别的统计信息
Evaluation eval = neuralNetwork.eval(testIterator);
System.out.println(eval.accuracy());
System.out.println(eval.precision());
System.out.println(eval.recall());

// 在多分类数据集上用于曲线下面积的ROC（非二分类）
ROCMultiClass roc = neuralNetwork.doEvaluation(testIterator, new ROCMultiClass());
System.out.println(roc.calculateAverageAuc());
System.out.println(roc.calculateAverageAucPR());
```

对于高级评估，下面的代码片段可以被适用于训练管道。这是当内置的neuralNetwork.eval\(\)方法输出混乱的结果或你需要检查原始数据时需要使用。 ​

```java
//在测试集上评估模型
Evaluation eval = new Evaluation(numClasses);
INDArray output = neuralNetwork.output(testData.getFeatures());
eval.eval(testData.getLabels(), output, testMetaData); //Note we are passing in the test set metadata here

//从评估对象上获取一个预测错误列表
//这样的预测误差只有在调用之后才可用。
iterator.setCollectMetaData(true)
List<Prediction> predictionErrors = eval.getPredictionErrors();
System.out.println("\n\n+++++ Prediction Errors +++++");
for(Prediction p : predictionErrors){
    System.out.println("Predicted class: " + p.getPredictedClass() + ", Actual class: " + p.getActualClass()
        + "\t" + p.getRecordMetaData(RecordMetaData.class).getLocation());
}

//我们也可以加载原始数据:
List<Record> predictionErrorRawData = recordReader.loadFromMetaData(predictionErrorMetaData);
for(int i=0; i<predictionErrors.size(); i++ ){
    Prediction p = predictionErrors.get(i);
    RecordMetaData meta = p.getRecordMetaData(RecordMetaData.class);
    INDArray features = predictionErrorExamples.getFeatures().getRow(i);
    INDArray labels = predictionErrorExamples.getLabels().getRow(i);
    List<Writable> rawData = predictionErrorRawData.get(i).getRecord();

    INDArray networkPrediction = model.output(features);

    System.out.println(meta.getLocation() + ": "
        + "\tRaw Data: " + rawData
        + "\tNormalized: " + features
        + "\tLabels: " + labels
        + "\tPredictions: " + networkPrediction);
}

//一些有用的评估方法:
//预测: 实际类 1,预测为类 2
List<Prediction> list1 = eval.getPredictions(1,2);
//预测类2的所有预测                  
List<Prediction> list2 = eval.getPredictionByPredictedClass(2);
//对实际类2的所有预测     
List<Prediction> list3 = eval.getPredictionsByActualClass(2);       
```



\`\`


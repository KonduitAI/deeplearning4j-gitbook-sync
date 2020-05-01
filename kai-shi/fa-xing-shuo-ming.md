---
description: Eclipse Deeplearning4j的每个版本都有新的变化。
---

# 发行说明

**内容**

* [Version 1.0.0-beta5](fa-xing-shuo-ming.md#1-0-0beta-5-fa-hang-shuo-ming)
  * [Deeplearning4j 与 DL4J Keras 导入](fa-xing-shuo-ming.md#deeplearning-4-j)
  * [ND4J 与 SameDiff](fa-xing-shuo-ming.md#nd-4-jsamediff-gong-neng-he-zeng-qiang)
  * [DataVec](fa-xing-shuo-ming.md#datavec)
  * [RL4J](fa-xing-shuo-ming.md#rl-4-j-gong-neng-he-zeng-qiang)
  * [Arbiter](fa-xing-shuo-ming.md#arbiter)
  * [ND4S](fa-xing-shuo-ming.md#nd-4-s-gong-neng-he-zeng-qiang)
* [Version 1.0.0-beta4](fa-xing-shuo-ming.md#1-0-0beta-4-ban-ben-fa-hang-shuo-ming)
  * [Deeplearning4j 与 DL4J Keras 导入](fa-xing-shuo-ming.md#deeplearning-4-j-1)
  * [ND4J 与 SameDiff](fa-xing-shuo-ming.md#nd-4-j-yu-samediff-1)
  * [DataVec](fa-xing-shuo-ming.md#datavec-1)
  * [Arbiter](fa-xing-shuo-ming.md#arbiter-1)
* [Version 1.0.0-beta3](fa-xing-shuo-ming.md#1-0-0beta-3-ban-ben-fa-hang-shuo-ming)
  * [Deeplearning4j](fa-xing-shuo-ming.md#deeplearning-4-j-2)
  * [Deeplearning4j Keras 导入](fa-xing-shuo-ming.md#deeplearing-4-j-keras-dao-ru)
  * [ND4J](fa-xing-shuo-ming.md#nd-4-j)
  * [DataVec](fa-xing-shuo-ming.md#datavec-2)
  * [Arbiter](fa-xing-shuo-ming.md#arbiter-2)
  * [RL4J](fa-xing-shuo-ming.md#rl-4-j-1)
  * [ScalNet](fa-xing-shuo-ming.md#scalnet)
  * [ND4S](fa-xing-shuo-ming.md#nd-4-s-1)
* [Version 1.0.0-beta2](fa-xing-shuo-ming.md#1-0-0beta-2-ban-ben-fa-hang-shuo-ming)
  * [Deeplearning4j](fa-xing-shuo-ming.md#deeplearning-4-j-3)
  * [Deeplearning4j Keras 导入](fa-xing-shuo-ming.md#deeplearing-4-j-keras-dao-ru-1)
  * [ND4J](fa-xing-shuo-ming.md#nd-4-j-1)
  * [DataVec](fa-xing-shuo-ming.md#datavec-3)
  * [Arbiter](fa-xing-shuo-ming.md#arbiter-3)
  * [RL4J](fa-xing-shuo-ming.md#rl-4-j-2)
  * [ScalNet](fa-xing-shuo-ming.md#scalnet-1)
  * [ND4S](fa-xing-shuo-ming.md#nd-4-s-2)
* [Version 1.0.0-beta](fa-xing-shuo-ming.md#1-0-0beta-ban-ben-fa-hang-shuo-ming)
  * [Deeplearning4j](fa-xing-shuo-ming.md#deeplearning-4-j-4)
  * [Deeplearning4j Keras 导入](fa-xing-shuo-ming.md#deeplearing-4-j-keras-dao-ru-2)
  * [ND4J](fa-xing-shuo-ming.md#nd-4-j-2)
  * [DataVec](fa-xing-shuo-ming.md#datavec-4)
  * [Arbiter](fa-xing-shuo-ming.md#arbiter-4)
  * [RL4J](fa-xing-shuo-ming.md#rl-4-j-3)
  * [ScalNet](fa-xing-shuo-ming.md#scalnet-2)
  * [ND4S](fa-xing-shuo-ming.md#nd-4-s-3)
* [Version 1.0.0-alpha](fa-xing-shuo-ming.md#1-0-0alpha-fa-hang-shuo-ming)
  * [Deeplearning4j](fa-xing-shuo-ming.md#deeplearning-4-j-5)
  * [Deeplearning4j Keras 导入](fa-xing-shuo-ming.md#deeplearing-4-j-keras-dao-ru-3)
  * [ND4J](fa-xing-shuo-ming.md#nd-4-j-3)
  * [DataVec](fa-xing-shuo-ming.md#datavec-5)
  * [Arbiter](fa-xing-shuo-ming.md#arbiter-5)
  * [RL4J](fa-xing-shuo-ming.md#rl-4-j-4)
  * [ScalNet](fa-xing-shuo-ming.md#scalnet-3)
  * [ND4S](fa-xing-shuo-ming.md#nd-4-s-3)
* [Version 0.9.1](fa-xing-shuo-ming.md#0-9-1-ban-ben-fa-hang-shuo-ming)
* [Version 0.9.0](fa-xing-shuo-ming.md#0-9-0-ban-ben-fa-hang-shuo-ming)
* [Version 0.8.0](fa-xing-shuo-ming.md#0-8-0-ban-ben-fa-hang-shuo-ming)
* [Version 0.7.2](fa-xing-shuo-ming.md#0-7-2-ban-ben-fa-hang-shuo-ming)
* [Version 0.6.0](fa-xing-shuo-ming.md#0-6-0-ban-ben-fa-hang-shuo-ming)
* [Version 0.5.0](fa-xing-shuo-ming.md#0-5-0-ban-ben-fa-hang-shuo-ming)
* [Version 0.4.0](fa-xing-shuo-ming.md#0-4-0-ban-ben-fa-hang-shuo-ming)

## [1.0.0-beta5发行说明](fa-xing-shuo-ming.md)

### 1.0.0-beta5 发行亮点

* 添加了模型服务器-使用JSON或（可选）二进制序列化对SameDiff和DL4J模型进行远程推理
  * 服务器: 查看 [JsonModelServer](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-remote/deeplearning4j-json-server/src/main/java/org/deeplearning4j/remote/JsonModelServer.java)
  * 客户端:  查看 [JsonRemoteInference](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-remote/nd4j-json-client/src/main/java/org/nd4j/remote/clients/JsonRemoteInference.java)
  * 测试/例子: 查看 [链接](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-remote/deeplearning4j-json-server/src/test/java/org/deeplearning4j/remote/JsonModelServerTest.java)  [链接](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-remote/deeplearning4j-json-server/src/test/java/org/deeplearning4j/remote/BinaryModelServerTest.java)
* 增加了Scala 2.12支持，删除了Scala2.10支持。具有Scala依赖关系的模块现在随Scala 2.11和2.12版本一起发布。
* Apache Spark 1.x 支持删除   \(现在只支持 Spark 2.x \). Note注意: Spark 删除了版本后缀 用于升级: `1.0.0-beta4_spark2 -> 1.0.0-beta5`
* 添加FastText 支持到 deeplearning4j-nlp
* CUDA支持所有ND4J/SameDiff操作
  * 在1.0.0-beta4中，有些操作仅限于CPU。现在，所有的操作都有完整的CUDA支持
* 在ND4J（和DL4J/SameDiff）中添加了对新数据类型的支持：BFLOAT16、UINT16、UINT32、UINT64
* ND4J: 隐式广播支持添加到 INDArray \(已经出现在SameDiff - 例如形状 `[3,1]+[3,2]=[3,2]`\)
* CUDA 9.2, 10.0 与  10.1-Update2 仍然支持
  * 注意: 对于 CUDA 10.1, CUDA 10.1 update 2是推荐的。 CUDA 10.1 与  10.1 Update 1仍将运行，但在某些系统上的多线程代码中可能会遇到罕见的内部cuBLAS问题
* 依赖项升级: Jackson \(2.5.1 to 2.9.9/2.9.9.3\), Commons Compress \(1.16.1 to 1.18\), Play Framework \(2.4.8 to 2.7.3\), Guava: \(20.0 to 28.0-jre, 并遮挡以避免依赖关系冲突\)
* CUDA: 现在，除了设备（GPU）缓冲区之外，主机（RAM）缓冲区只在需要时分配（以前：主机缓冲区总是被分配的）

### [Deeplearning4J](fa-xing-shuo-ming.md)

#### Deeplearning4J: 功能和增强

* 添加了FastText-推理和训练，包括OOV（词汇表外）支持（[链接](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nlp-parent/deeplearning4j-nlp/src/main/java/org/deeplearning4j/models/fasttext/FastText.java)）
* Scala 2.12 支持添加, Scala 2.10支持删除 \([链接](https://github.com/SkymindIO/deeplearning4j/pull/199)\)
* 添加模型服务器\(DL4J 与  SameDiff 模型, JSON 与二进制通信\) - [JsonModelServer](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-remote/deeplearning4j-json-server/src/main/java/org/deeplearning4j/remote/JsonModelServer.java), [JsonRemoteInference](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-remote/nd4j-json-client/src/main/java/org/nd4j/remote/clients/JsonRemoteInference.java), [链接](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-remote/deeplearning4j-json-server/src/test/java/org/deeplearning4j/remote/JsonModelServerTest.java)，[链接](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-remote/deeplearning4j-json-server/src/test/java/org/deeplearning4j/remote/BinaryModelServerTest.java) 
* 添加了保存的模型格式验证实用程序 - DL4JModelValidator, DL4JKerasModelValidator \([链接](https://github.com/eclipse/deeplearning4j/pull/7701)\)
* 添加LabelLastTimeStepPreProcessor \([链接](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/dataset/api/preprocessor/LabelLastTimeStepPreProcessor.java)\)
* BertIterator: 添加了将令牌前置到输出的选项（例如某些模型所期望的\[cls\]）（[链接](https://github.com/SkymindIO/deeplearning4j/pull/39)）
* 在多层网络和计算图中添加跟踪级日志记录有助于调试某些问题（[链接](https://github.com/SkymindIO/deeplearning4j/pull/79)）
* Upsampling3D: 添加 NDHWC 支持 \([链接](https://github.com/eclipse/deeplearning4j/issues/8016)\)
* MergeVertex 现在支持广播 \([链接](https://github.com/eclipse/deeplearning4j/issues/6488)\)
* LSTM 与 Dropout现在将依赖于内置实现，如果从cuDNN中遇到异常 \(与 Subsampling/ConvolutionLayer一样 \) \([链接](https://github.com/SkymindIO/deeplearning4j/pull/152)\)
* 改进的JavaDoc和清理WordVectorSerializer的API \([链接](https://github.com/SkymindIO/deeplearning4j/pull/221)，[链接](https://github.com/eclipse/deeplearning4j/issues/8137)\)

#### Deeplearning4J: Bugs修复和优化

* 更新的deeplearning4j-ui界面主题\([Link](https://github.com/eclipse/deeplearning4j/pull/7683)\)
* 修复了MergeVertex和CNN3D激活的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7715)\)
* 修复Yolo2OutputLayer builder/configuration 方法名中排版错误  \([Link](https://github.com/eclipse/deeplearning4j/pull/7728)\)
* 改进计算图构建器输入类型验证\([Link](https://github.com/eclipse/deeplearning4j/pull/7752)\)
* 移除dl4j-spark-ml模块，直到它可以被适当的维护 \([Link](https://github.com/eclipse/deeplearning4j/pull/7764)\)
* 修复了BertWordPieceTokenizerFactory和错误字符编码的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7678)\)
* 修复了LearnedSelfAttentionLayer和变量 minibatch 大小的问题 \([Link](https://github.com/eclipse/deeplearning4j/pull/7780), [Link](https://github.com/eclipse/deeplearning4j/issues/7777)\)
* 修复了从环境变量设置SharedTrainingMaster控制器地址时出现的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7786)\)
* 修复了在某些情况下SameDiffOutputLayer初始化的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7785)\)
* 默认情况下，https用于数据和zoo模型下载 \([Link](https://github.com/eclipse/deeplearning4j/issues/7779), [Link](https://github.com/eclipse/deeplearning4j/pull/7793)\)
* 修复了UI-WebJars依赖项检查每个构建上的更新的问题\([Link](https://github.com/eclipse/deeplearning4j/issues/7730), [Link](https://github.com/eclipse/deeplearning4j/pull/7793)\)
* 修复了上采样层内存报告可能产生OOM异常的问题 \([Link](https://github.com/eclipse/deeplearning4j/pull/8015)\)
* 改进了RecordReaderDataSetIterator的UX/验证 \([Link](https://github.com/eclipse/deeplearning4j/pull/8042)\)
* 修正了EmbeddingSequenceLayer不检查掩码数组数据类型的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7866)\)
* 使用非rank-2（shape\[1，numParams\]）数组初始化网络时的验证改进 \([Link](https://github.com/eclipse/deeplearning4j/issues/7890)\)
* 修正了BertIterator的数据类型问题 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/17)\)
* 修正了Word2Vec模型向后兼容（beta3和早期的模型现在可以再次加载）[Link](https://github.com/SkymindIO/deeplearning4j/pull/18)
* 修复了某些Keras导入模型可能会失败的问题`Could not read abnormally long HDF5 attribute` \([Link](https://github.com/eclipse/deeplearning4j/issues/7919)\)
* 为RnnOutputLayer特征/标签数组长度添加了验证 \([Link](https://github.com/eclipse/deeplearning4j/issues/7925)\)
* 修复了SameDiffOutputLayer不支持变量minibatch大小的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7844)\)
* 修复了DL4J SameDiff 层掩码支持 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/67)\)
* DL4J UI: 修复了可视化保存/存储的数据时选项卡切换不起作用的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7954), [Link](https://github.com/SkymindIO/deeplearning4j/pull/243)\)
* DL4J UI: 修复了一个罕见的UI线程问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/8017)\)
* 修复了JSON格式更改时的Keras导入问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7992)\)
* 修复了Keras导入问题，其中更新程序学习速率调度可能被错误导入\([Link](https://github.com/SkymindIO/deeplearning4j/pull/84)\)
* 修复了CnnSentenceDataSetIterator 的问题，该问题是使用 `UnknownWordHandling.UseUnknownVector` 时产生的\([Link](https://github.com/eclipse/deeplearning4j/issues/8121), [Link](https://github.com/eclipse/deeplearning4j/issues/8120)\)
* 对DL4J SameDiff层的修复和优化 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/156)\)
* 如果在工作区关闭期间发生第二个异常，多层网络/计算图现在将记录原始异常，而不是将其吞入（推理/拟合操作try/finally块） \([Link](https://github.com/SkymindIO/deeplearning4j/pull/161)\)
* 更新依赖: Jackson \(2.5.1 to 2.9.9/2.9.9.3\), Commons Compress \(1.16.1 to 1.18\), Play Framework \(2.4.8 to 2.7.3\), Guava: \(20.0 to 28.0-jre, 并遮挡以避免依赖关系冲突\) \([Link](https://github.com/SkymindIO/deeplearning4j/pull/199)\)
* 现在可以为DL4J UI配置日志框架（由于Play framework依赖项升级） \([Link](https://github.com/eclipse/deeplearning4j/issues/3808)\)
* MnistDataFetcher产生的垃圾量减少（影响MNIST和EMNIST DataSetIterators） \([Link](https://github.com/SkymindIO/deeplearning4j/pull/200)\)
* 激活函数反向传播已经为许多激活函数进行了优化 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/211), [Link](https://github.com/SkymindIO/deeplearning4j/pull/207)\)

#### Deeplearning4j: 迁移指南, 1.0.0-beta4 到 1.0.0-beta5

* DL4J AsyncDataSetIterator 和 AsyncMultiDataSetIterator移到了 ND4J, 使用 `org.nd4j.linalg.dataset.Async(Multi)DataSetIterator` 替换原来的
* 无法再加载从1.0.0-alpha及以前版本中保存的具有自定义图层的模型。解决方法：加载1.0.0-beta4，然后重新保存模型（[链接](https://github.com/SkymindIO/deeplearning4j/pull/199)）。没有自定义层的模型仍然可以加载回0.5.0
* Apache Spark 1.x 支持被删除\( 现在只有 Spark 2.x 被支持\). 注意：Spark版本后缀已删除：对于升级，请按如下方式更改版本: `1.0.0-beta4_spark2 -> 1.0.0-beta5`
* Scala 2.10 被删除, Scala 2.12 被添加  \(对于具有Scala依赖项的模块\)

#### Deeplearning4j: 1.0.0-beta5 已知的问题 

* dl4j-spark\_2.11和\_2.12依赖项错误地拉入datavec-spark\_2.11/2.12版本1.0.0-SNAPSHOT。解决方法：根据[此处](https://github.com/eclipse/deeplearning4j-examples/pull/901/files)或[此处](https://gist.github.com/AlexDBlack/5721cb6fbd6cd98b6702cd49e733dacf)使用依赖关系管理的控件版本
* 由于添加了同步，某些层（如LSTM）在不使用cuDNN时在CUDA的1.0.0-beta5上可能比1.0.0-beta4上运行得慢。此同步将在1.0.0-beta5之后的下一个版本中删除
* CUDA 10.1：在运行cuda10.1 update 1（可能还有10.1）时，在某些系统上的多线程代码中可能会遇到罕见的内部cuBLAS问题。建议使用CUDA 10.1 update2。

### [ND4J 与 SameDiff](fa-xing-shuo-ming.md)

#### ND4J/SameDiff: 功能和增强

* 添加新数据类型: BFLOAT16, UINT16, UINT32, UINT64 \([Link](https://github.com/eclipse/deeplearning4j/pull/7723)\)
* CUDA支持所有没有CUDA实现的操作 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/26), [Link](https://github.com/SkymindIO/deeplearning4j/pull/57), [Link](https://github.com/SkymindIO/deeplearning4j/pull/63), [Link](https://github.com/SkymindIO/deeplearning4j/pull/69), [Link](https://github.com/SkymindIO/deeplearning4j/pull/95)\)
* 添加了模型服务器（DL4J和SameDiff模型、JSON和二进制通信）- [JsonModelServer](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-remote/deeplearning4j-json-server/src/main/java/org/deeplearning4j/remote/JsonModelServer.java), [JsonRemoteInference](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-remote/nd4j-json-client/src/main/java/org/nd4j/remote/clients/JsonRemoteInference.java), [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-remote/deeplearning4j-json-server/src/test/java/org/deeplearning4j/remote/JsonModelServerTest.java), [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-remote/deeplearning4j-json-server/src/test/java/org/deeplearning4j/remote/BinaryModelServerTest.java)
* 添加了对形状为零的空数组的支持，以与TensorFlow导入兼容 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/10)\)
* CUDA：现在，除了设备（GPU）缓冲区外，主机（RAM）缓冲区只在需要时分配（以前：总是分配主机缓冲区）
* 改进的SameDiff训练API-添加了“在线”测试集评估、返回带有损失曲线的历史对象等 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/99)\)
* 添加了保存的模型格式验证实用程序 - Nd4jValidator, Nd4jCommonValidator \([Link](https://github.com/eclipse/deeplearning4j/pull/7701)\)
* 添加 SameDiff ScoreListener \(等同于 DL4J ScoreIterationListener/PerformanceListener\) \([Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/listeners/impl/ScoreListener.java), [Link](https://github.com/eclipse/deeplearning4j/pull/7774)\)
* 添加SameDiff.convertDataTypes 方法,用于变量dtype转换 \([Link](https://github.com/eclipse/deeplearning4j/blob/a76a44e1988fa785f38fd99df17e7396a7746b53/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff/SameDiff.java#L3799-L3812)\)
* 添加crop 与 resize 操作 \([Link](https://github.com/eclipse/deeplearning4j/pull/7741)\)
* DL4J AsyncDataSetIterator 与 AsyncMultiDataSetIterator 迁移到  ND4J [Link](https://github.com/eclipse/deeplearning4j/pull/7774)
* 添加 basic/MVP SameDiff UI listener \([Link](https://github.com/eclipse/deeplearning4j/pull/7787)\)
* 添加 SameDiff CheckpointListener \([Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/listeners/checkpoint/CheckpointListener.java), [Link](https://github.com/eclipse/deeplearning4j/pull/7807)\)
* 添加 SameDiff名称作用域  \([Link](https://github.com/eclipse/deeplearning4j/blob/a76a44e1988fa785f38fd99df17e7396a7746b53/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff/SameDiff.java#L544-L579)\)
* SameDiff: 更新器 状态和训练配置现在写入FlatBuffers格式 \([Link](https://github.com/eclipse/deeplearning4j/pull/7825)\)
* 添加了可从Java调用的c++基准测试套件
* `Nd4j.getExecutioner().runLightBenchmarkSuit()` 与   `Nd4j.getExecutioner().runFullBenchmarkSuit()` \([Link](https://github.com/SkymindIO/deeplearning4j/pull/3)\)
* 添加了带InputStream/OutputStream参数的SameDiff.save/load方法 \([Link](https://github.com/eclipse/deeplearning4j/issues/7856), [Link](https://github.com/SkymindIO/deeplearning4j/pull/6)\)
* 为评估实例的添加轴配置 \(Evaluation, RegressionEvaluation, ROC, etc - getAxis 与 setAxis 方法\) 来允许不同的数据格式 \(NCHW vs. 用于CNNs的NHWC, 例如 \) \([Link](https://github.com/SkymindIO/deeplearning4j/pull/6)\)
* SameDiff: 添加了将常量转换为占位符的支持, 通过  SDVariable.convertToConstant\(\) 方法 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/11)\)
* SameDiff:  为 SameDiff添加了GradCheckUtil.checkActivationGradients方法来检查激活梯度（不仅仅是在现有的梯度检查方法中的参数梯度） \([Link](https://github.com/SkymindIO/deeplearning4j/pull/19)\)
* 添加CheckNumerics 操作 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/23/commits/8399ef0e11d7a23a3636516fab7ee6fdc7d8f313)\)
* 添加 FakeQuantWithMinMaxArgs 与 FakeQuantWithMinMaxVars 操作  \([Link](https://github.com/SkymindIO/deeplearning4j/pull/24)\)
*  添加了带“keep dimensions”选项的INDArray缩减方法-例如，`INDArray.mean(boloean, int... dimension)` \([Link](https://github.com/SkymindIO/deeplearning4j/pull/33)\)
* 添加 Nd4j SystemInfo 类  - SystemInfo.getSystemInfo, .writeSystemInfo\(File\) 帮助调试问题 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/34), [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/systeminfo/SystemInfo.java)\)
* 添加了INDArray.toString\(NDArrayStrings options\)、toStringFull\(\)和toString方法重载，以便更轻松地控制数组打印\([Link](https://github.com/SkymindIO/deeplearning4j/pull/36)\)
* 添加  HashCode 操作, INDArray.hashCode\(\) \([Link](https://github.com/SkymindIO/deeplearning4j/pull/50)\)
* SameDiff: 为 loops/conditional 操作添加 whileLoop，ifCond方法 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/52)\)
* 清理了一些不常用的Nd4j方法 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/89), [Link](https://github.com/SkymindIO/deeplearning4j/pull/101), [Link](https://github.com/SkymindIO/deeplearning4j/pull/102), [Link](https://github.com/SkymindIO/deeplearning4j/pull/231)\)
* 增加了位整数运算：左/右位移位、左/右循环位移位、位汉明距离 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/115), [Link](https://github.com/SkymindIO/deeplearning4j/pull/118), [Link](https://github.com/SkymindIO/deeplearning4j/pull/126), [Link](https://github.com/SkymindIO/deeplearning4j/pull/192), [Link](https://github.com/SkymindIO/deeplearning4j/pull/195)\)
* deeplearning4j-nlp: 重命名 AggregatingSentencePreProcessor 为 sentencePreProcessor method \([Link](https://github.com/eclipse/deeplearning4j/issues/8122)\)
* 升级 Protobuf 版本 - 3.5.1 到 3.8.0 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/162)\)
* 为 libnd4j 本地操作切换c=style错误处理\([Link](https://github.com/SkymindIO/deeplearning4j/pull/169)\)
* 重命名  FlatBuffers 枚举 `org.nd4j.graph.DataType` 为 `org.nd4j.graph.DType` 避免用户在使用Nd4j方法时导入不正确的类型 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/228), [Link](https://github.com/eclipse/deeplearning4j/issues/8159)\)
* 为位操作添加 SameDiff.bitwise 命名空间 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/232), [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff/ops/SDBitwise.java)\)

#### ND4J/SameDiff: Bug 修复与优化

* 更新到JavaCPP/JavaCV 1.5.1-1 \([Link](https://github.com/eclipse/deeplearning4j/pull/8004)\)
* SameDiff:  现在只有在需要计算请求的变量时，才必须提供占位符\([Link](https://github.com/eclipse/deeplearning4j/pull/7814)\)
* SameDiff: 修正了重复变量名验证的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7705)\)
* SameDiff: 修复了标量的SDVariable.getArr问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7546)\)
* 向DeviceLocalNDArray添加了延迟模式（需要时才复制到设备） \([Link](https://github.com/SkymindIO/deeplearning4j/pull/149)\)
* ND4J: 修复了以numpy.npy格式写入0d（标量）NDArrays的问题\([Link](https://github.com/eclipse/deeplearning4j/pull/7712)\)
* 修复了一些固定情况下Pad操作的问题 \([Link](https://github.com/eclipse/deeplearning4j/pull/7713)\)
* 修复了strided\_slice操作的一些问题 \([Link](https://github.com/eclipse/deeplearning4j/pull/7719), [Link](https://github.com/eclipse/deeplearning4j/pull/7823), [Link](https://github.com/SkymindIO/deeplearning4j/pull/14)\)
* SameDiff:  修复了某些使用ND4J默认数据类型的操作的DataType推理问题\([Link](https://github.com/eclipse/deeplearning4j/pull/7699)\)
* INDArray.castTo\(DataType\) 当数组已经是正确的类型时，现在是no-op \([Link](https://github.com/eclipse/deeplearning4j/issues/7754)\)
* SameDiff:  修复了训练混合精度网络的问题\([Link](https://github.com/eclipse/deeplearning4j/issues/6992)\)
* 修复了Evaluation类错误地报告二进制情况下macro-averaged精度的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7802)\)
* 从 SameDiff TrainingConfig \(不再需要\)移除 trainableParams config/field \([Link](https://github.com/eclipse/deeplearning4j/pull/7807)\)
* ND4J javadoc的改进与清理 \([Link](https://github.com/eclipse/deeplearning4j/pull/7997), [Link](https://github.com/SkymindIO/deeplearning4j/pull/110), [Link](https://github.com/SkymindIO/deeplearning4j/pull/111), [Link](https://github.com/SkymindIO/deeplearning4j/pull/112)\)
* 修正了CUDA上Cholesky Lapack操作的问题\([Link](https://github.com/eclipse/deeplearning4j/pull/8188), [Link](https://github.com/eclipse/deeplearning4j/issues/8167)\)
* 修复当\[1,N\] 与 \[N,1\]时INDArray.isMatrix\(\) 不被认为是矩阵的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7839)\)
* 为 4D arrays 修复RegressionEvaluation \(CNNs / segmentation\) \([Link](https://github.com/SkymindIO/deeplearning4j/pull/6), [Link](https://github.com/eclipse/deeplearning4j/issues/7859)\)
* 修复INDArray.median\(int... dimension\) 的问题\([Link](https://github.com/eclipse/deeplearning4j/issues/7848)\)
* 修复当执行收集操作backprop 可能发生无指针异常\([Link](https://github.com/eclipse/deeplearning4j/issues/7852)\)
* 修复 LogSumExp操作Java/C++映射的问题  \([Link](https://github.com/eclipse/deeplearning4j/issues/7850)\)
* 在读取Numpy.npy文件时添加了头验证，以确保文件有效 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/46)\)
* 修复了在CUDA上读取Numpy.npy文件时可能出现的问题\([Link](https://github.com/SkymindIO/deeplearning4j/pull/64)\)
* 修复了读取Numpy.npy布尔文件时出现的问题\([Link](https://github.com/SkymindIO/deeplearning4j/pull/91)\)
* TensorFlow导入的各种问题修复 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/55)\)
* 修复了少数Nd4j.create方法不创建与java原始类型对应的数组的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/8057)\)
* 一些Nd4j.create方法的形状验证改进 \([Link](https://github.com/eclipse/deeplearning4j/issues/7551)\)
* 清理了未维护的Nd4j.createSparse方法 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/92)\)
* 修复了带有CC 3.0的CUDA GPU的CUDA问题 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/105)\)
* 修复了c++代码中一些可能的整数溢出\([Link](https://github.com/SkymindIO/deeplearning4j/pull/108)\)
* 删除了不推荐的方法：Nd4j.trueScalar和Nd4j.trueVector \([Link](https://github.com/SkymindIO/deeplearning4j/pull/129), [Link](https://github.com/SkymindIO/deeplearning4j/pull/145)\)
* 修正了一些jvm可能由于SameDiff依赖关系（现在已删除）而警告“非法反射访问”的问题\([Link](https://github.com/eclipse/deeplearning4j/issues/8123)\)
* SDVariable现在不再扩展DifferentialFunction\([Link](https://github.com/SkymindIO/deeplearning4j/pull/150)\)
* 将数个操作calculateOutputShape的实例从Java迁移到C++ \([Link](https://github.com/SkymindIO/deeplearning4j/pull/151)\)
* 修复了maxpool2d\_bp在存在NaN值时引发异常的问题\([Link](https://github.com/SkymindIO/deeplearning4j/pull/160)\)
* 修复了空形状（带零）连接的问题 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/167)\)
* 删除 INDArray.javaTensorAlongDimension \([Link](https://github.com/SkymindIO/deeplearning4j/pull/170)\)
* LayerNorm 操作现在支持axis参数, NCHW 格式数据 \([Link](https://github.com/eclipse/deeplearning4j/issues/8008)\)
* libnd4j:由于cuBLAS的限制，cuBLAS hgemm（FP16 gemm）仅对计算能力大于等于5.3的设备调用\([Link](https://github.com/SkymindIO/deeplearning4j/pull/181)\)
* Nd4j.readNumpy 优化 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/183)\)
* 在c语言的ELU和lrelu-bp操作中添加了可配置的alpha参数++ \([Link](https://github.com/SkymindIO/deeplearning4j/pull/213)\)
* 清理 SameDiff SDCNN/SDRNN \(SameDiff.cnn, .rnn\) API/方法 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/230), [Link](https://github.com/SkymindIO/deeplearning4j/pull/238)\)

#### ND4J: 迁移指南, 1.0.0-beta4 到  1.0.0-beta5

* OldAddOp, OldSubOp, 等被删除 : 用  AddOp, SubOp, 等操作
* Nd4j.trueScalar 与  trueVector 被删除 ; 使用  Nd4j.scalar 和 Nd4j.createFromArray 方法
* INDArray.javaTensorAlongDimension 被删除 ; 用 INDArray.tensorAlongDimension 代替它
* INDArray.lengthLong\(\) 被删除 ; 使用 INDArray.length\(\) 代替它 

#### ND4J: 1.0.0-beta5 已知的问题 

* nd4j-native 在一些  OSX 系统上会出现这样的错误 `Symbol not found: ___emutls_get_address` - 查看[此链接](https://github.com/eclipse/deeplearning4j/issues/8217) 
* SBT1.3.0可能失败，Illegal character in path；SBT1.2.8可以。这是SBT问题，不是ND4J问题。有关详细信息，请参阅此[链接](https://github.com/sbt/sbt/issues/5046)

### [DataVec](fa-xing-shuo-ming.md)

#### DataVec: 功能和增强

* ImageRecordReader: 添加了对16位TIFF的支持 \([Link](https://github.com/eclipse/deeplearning4j/pull/7731)\)
* 添加 SequenceTrimToLengthTransform \([Link](https://github.com/SkymindIO/deeplearning4j/pull/61)\)

#### DataVec: Bug修复和优化

* 修复了AnalyzeSpark和字符列的问题\([Link](https://github.com/eclipse/deeplearning4j/issues/7680)\)
* 修复了NumberedFileInputScheme中URL方案检测的问题 \([Link](https://github.com/eclipse/deeplearning4j/pull/7742)\)
* 修正了RandomPathFilter有偏差的问题 \([Link](https://github.com/eclipse/deeplearning4j/pull/7769), [Link](https://github.com/eclipse/deeplearning4j/issues/7738)\)

### [RL4J](fa-xing-shuo-ming.md)

#### RL4J: 功能和增强

* API清理和重构\([Link](https://github.com/eclipse/deeplearning4j/pull/7958), [Link](https://github.com/eclipse/deeplearning4j/pull/8034), [Link](https://github.com/eclipse/deeplearning4j/pull/8050), [Link](https://github.com/eclipse/deeplearning4j/pull/8065)\)

#### RL4J: Bug修复和优化

*  修复了HistoryProcessor的压缩问题 \([Link](https://github.com/eclipse/deeplearning4j/pull/7819)\)

### [Arbiter](fa-xing-shuo-ming.md)

#### Bug修复和优化

* 更新EvaluationScoreFunction以使用ND4J评估类度量 \([Link](https://github.com/eclipse/deeplearning4j/issues/7804)\)
* 修复在GridSearchCandidateGenerator不正确的搜索大小 \([Link](https://github.com/eclipse/deeplearning4j/issues/8082)\)

#### Arbiter: 已知问题

* Jackson版本的升级需要更改通用对象序列化的执行方式；存储在1.0.0-beta4或更早格式中的Arbiter JSON数据在1.0.0-beta5中可能不可读取。（[链接](https://github.com/SkymindIO/deeplearning4j/pull/237)）

### [ND4S](fa-xing-shuo-ming.md)

#### ND4S功能和增强

* 根据ND4J向ND4S添加了完整的数据类型支持 \([Link](https://github.com/SkymindIO/deeplearning4j/pull/51)\)
* 为 SameDiff \(implicits, operator overloads\) 添加语法糖\([Link](https://github.com/SkymindIO/deeplearning4j/pull/113)\)

## [1.0.0-beta4版本发行说明 ](fa-xing-shuo-ming.md)

###  1.0.0-beta4发行亮点

主要亮点：对ND4J和DL4J的完全多数据类型支持。在过去的版本中，ND4J中的所有N维数组都被限制为单个数据类型（float或double），全局设置。现在，所有数据类型的数组都可以同时使用。支持以下数据类型：

* DOUBLE: 双精度浮点，64位 \(8 字节\)
* FLOAT: 单精度浮点，32位 \(4 字节\)
* HALF: 半精度浮点，16位（2字节）, "FP16"
* LONG: 长有符号整数，64位（8字节）
* INT: 有符号整数，32位（4字节）
* SHORT: 有符号短整数，16位（2字节）
* UBYTE: 无符号字节，8位（1字节），0到255
* BYTE: 有符号字节，8位（1字节），-128至127
* BOOL: 布尔类型，（0/1，真/假）。使用ubyte存储以便于操作并行化
* UTF8: 字符串数组类型，UTF8格式

_ND4J_ 行为变化_:_

* 从Java原生数组创建INDArray时，INDArray数据类型将由原生数组类型确定（除非指定了数据类型）
  * 例如：Nd4j.createFromArray\(double\[\]\)-&gt;double数据类型INDArray
  * 类似地，Nd4j.scalar\(1\)、Nd4j.scalar\(1L\)、Nd4j.scalar\(1.0\)和Nd4j.scalar\(1.0f\)将分别生成INT、LONG、DOUBLE和FLOAT类型的标量INDArray
* 某些操作要求操作数具有匹配的数据类型
  * 例如，如果x和y是不同的数据类型，则可能需要强制转换: x.add\(y.castTo\(x.dataType\(\)\)\)
* 有些操作有数据类型限制：例如，不支持UTF8数组上的sum，BOOL数组上的variance也不受支持。对于布尔数组（如sum）上的某些操作，首先强制转换为整数或浮点类型可能是有意义的。

_DL4J_ 行为变化_:_

* MultiLayerNetwork/ComputationGraph 不再以任何方式依赖于ND4J全局数据类型。
  * 网络的数据类型（用于参数和激活的数据类型）可以在构建期间设置， 使用  `NeuralNetConfigutation.Builder().dataType(DataType)`方法
  * 网络可以从一种类型转换为另一种类型（双精度到浮点，浮点到半精度等）使用  `MultiLayerNetwork/ComputationGraph.convertDataType(DataType)` 方法

主要新方法：

* Nd4j.create\(\), zeros\(\), ones\(\), linspace\(\), 等带有DataType参数的方法
* INDArray.castTo\(DataType\) 方法 - 将INDArrays从一种数据类型转换为另一种数据类型
* 新 Nd4j.createFromArray\(...\) 方法

**ND4J/DL4J: CUDA - 10.1 支持添加, CUDA 9.0 支持删除**  

1.0.0-beta4支持的CUDA版本: CUDA 9.2, 10.0, 10.1.

**ND4J: Mac/OSX CUDA 支持被删除**

不再提供Mac（OSX）CUDA二进制文件。Linux（x86\_64，ppc64le）和Windows（x86\_64）CUDA支持仍然存在。OSX CPU支持（x86\_64）仍然可用。

**DL4J/ND4J: MKL-DNN 支持被添加** 在CPU/本机后端上运行时，默认情况下，DL4J（和ND4J conv2d等操作）现在支持MKL-DNN。MKL-DNN支持针对以下层类型实现：

* ConvolutionLayer 与 Convolution1DLayer \(与  Conv2D/Conv2DDerivative ND4J 操作\)
* SubsamplingLayer 与 Subsampling1DLayer \(与 MaxPooling2D/AvgPooling2D/Pooling2DDerivative ND4J 操作 \)
* BatchNormalization 层 \(与 BatchNorm ND4J op\)
* LocalResponseNormalization 层 \(与 LocalResponseNormalization ND4J 操作 \)
* Convolution3D 层 \(与 Conv3D/Conv3DDerivative ND4J 操作\)

MKL-DNN对其他层类型（如LSTM）的支持将在将来的版本中添加。

MKL-DNN 可以使用以下代码全局禁用（ND4J和DL4J）`Nd4jCpu.Environment.getInstance().setUseMKLDNN(false);`

MKL-DNN 可以全局禁用特定操作通过设置 `ND4J_MKL_FALLBACK`  环境变量为要禁用MKL-DNN支持的操作的名称。例如：`ND4J_MKL_FALLBACK=conv2d,conv2d_bp`

**ND4J: 内存管理更改提高了性能**

以前的ND4J版本使用定期垃圾收集（GC）来释放未在内存工作区中分配的内存。（请注意，默认情况下，DL4J几乎对所有操作都使用工作区，因此在训练 DL4J网络时，经常会禁用定期GC）。但是，对垃圾收集的依赖导致性能开销随着JVM堆中对象的数量而增加。

在1.0.0-beta4中，周期性垃圾收集在默认情况下是禁用的；相反，只有在需要从工作区外分配的数组中回收内存时，才会调用GC。 

要重新启用定期GC（根据beta3中的默认设置），并将GC频率设置为每5秒（5000ms），可以使用：

```text
Nd4j.getMemoryManager().togglePeriodicGc(true);
Nd4j.getMemoryManager().setAutoGcWindow(5000);
```

**ND4J: 改进了Rank 0/1数组支持**

在以前的ND4J版本中，在获取行/列、使用INDArray.get（NDArrayIndex…）获取子数组或从Java数组/标量创建数组时，标量和向量有时是rank 2而不是rank 0/1。现在，这些rank 0/1情况的行为应该更加一致。注意：为了保持getRow和getColumn的旧行为（即分别返回shape\[1，x\]和\[x，1\]的rank2数组），可以使用getRow\(long，boolean\)和getColumn\(long，boolean\)方法。

**DL4J: Attention 层被添加**

* [AttentionVertex](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/AttentionVertex.java)
* [LearnedSelfAttentionLayer](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LearnedSelfAttentionLayer.java)
* [RecurrentAttentionLayer](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/RecurrentAttentionLayer.java)
* [SelfAttentionLayer](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/SelfAttentionLayer.java)

### [Deeplearning4J](fa-xing-shuo-ming.md)

#### Deeplearning4J: 功能和增强

* 增加了对Conv/Pool/BatchNorm/LRN层的MKL-DNN支持。使用nd4j-native后端时，将自动使用MKL-DNN。 \([Link](https://github.com/eclipse/deeplearning4j/pull/7151), [Link](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/mkldnn)\)
* L1/L2正则化现在变成了一个类；增加了权重衰减，更好地控制何时/如何应用它。有关L2和权重衰减之间的差异的详细信息，请参见[本页](https://www.fast.ai/2018/07/02/adam-weight-decay/)。一般来说，权值衰减应优先于L2正则化。（[链接](https://github.com/eclipse/deeplearning4j/pull/7097)，[链接](https://github.com/eclipse/deeplearning4j/issues/7079)）
* 添加点积attention 层 : [AttentionVertex](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/AttentionVertex.java), [LearnedSelfAttentionLayer](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LearnedSelfAttentionLayer.java), [RecurrentAttentionLayer](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/RecurrentAttentionLayer.java) 与 [SelfAttentionLayer](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/SelfAttentionLayer.java)
* NeuralNetConfiguration.Builder\([Link](https://github.com/eclipse/deeplearning4j/issues/7532)\)上的dataType\(dataType\)方法为新网络设置新模型的参数/激活数据类型
* MultiLayerNetwork/ComputationGraph 可以在参数和激活的（浮点）数据类型FP16/32/64之间使用`MultiLayerNetwork/ComputationGraph.convertDataType(DataType)` 方法 \([Link](https://github.com/eclipse/deeplearning4j/pull/7531), [Link](https://github.com/eclipse/deeplearning4j/issues/7520)\)
* EmbeddingLayer 与 EmbeddingSequenceLayer builders 现在有  `.weightInit(INDArray)` 和  `.weightInit(Word2Vec)` 方法用于从预训练的词向量（[链接](https://github.com/eclipse/deeplearning4j/pull/7173)）初始化参数
* PerformanceListener 现在可以配置为报告垃圾收集信息（数量/持续时间）[链接](https://github.com/eclipse/deeplearning4j/issues/6717)
* 评估类现在将检查预测输出中的NaN，并抛出异常，而不是将argMax\(NaNs\)视为具有值0（[Link](https://github.com/eclipse/deeplearning4j/issues/6748)）
* 为ParallelInference添加了ModelAdapter，以方便使用和YOLO等用例（通过避免分离（不在工作区内）数组提高性能）
* 增加了GELU激活函数（[链接](https://github.com/eclipse/deeplearning4j/pull/7426)）
* 添加BertIterator（BERT训练的MultiDataSetIterator-有监督和无监督）[链接](https://github.com/eclipse/deeplearning4j/pull/7430)
* 向多层网络/计算图添加验证，当尝试对分类器执行回归评估时抛出异常，反之亦然（[Link](https://github.com/eclipse/deeplearning4j/issues/6735)，[Link](https://github.com/eclipse/deeplearning4j/pull/6774)）
* 添加`ComputationGraph.output(List<String> layers, boolean train, INDArray[] features, INDArray[] featureMasks)` 方法仅获取特定层/顶点集的激活（无多余计算）（[链接](https://github.com/eclipse/deeplearning4j/issues/6736)）
* 网络的权重初始化现在实现为类（不仅仅是枚举），因此现在可以通过IWeightInit接口（Link）进行扩展；也就是说，现在支持自定义权重初始化（[Link](https://github.com/eclipse/deeplearning4j/pull/6820)，[Link](https://github.com/eclipse/deeplearning4j/issues/6813)）
* 增加了胶囊网络层（在下一版本发布之前没有GPU加速）- [CapsuleLayer](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/CapsuleLayer.java), [CapsuleStrengthLayer](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/CapsuleStrengthLayer.java) 与 [PrimaryCapsules](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/PrimaryCapsules.java) \([Link](https://github.com/eclipse/deeplearning4j/pull/7391)\)
* 添加  `Cifar10DataSetIterator` 来替代 `CifarDataSetIterator` \([Link](https://github.com/eclipse/deeplearning4j/pull/6875), [Link](https://github.com/eclipse/deeplearning4j/issues/6834#issuecomment-446095723)\)
* Keras导入：现在支持从InputStream导入模型 \([Link](https://github.com/eclipse/deeplearning4j/issues/5594), [Link](https://github.com/eclipse/deeplearning4j/pull/6980)\)
* Layer/NeuralNetConfiguration builder现在也有getter/setter方法，以便更好地支持Kotlin \([Link](https://github.com/eclipse/deeplearning4j/pull/6990)\)
* UI的大多数JavaScript依赖项和字体已经迁移到WebJars \([Link](https://github.com/eclipse/deeplearning4j/pull/7046)\)
* CheckpointListener 现在有静态的availableCheckpoints\(File\), loadCheckpointMLN\(File, int\) 与 lostLastCheckpointMLN\(File\) 等方法 \([Link](https://github.com/eclipse/deeplearning4j/issues/7032)\)
* MultiLayerNetwork/ComputationGraph  现在，在某些不兼容的RNN配置中验证并抛出异常，如通过时间的截断反向传播与LastTimeStepLayer/Vertex 相结合\([Link](https://github.com/eclipse/deeplearning4j/issues/6991)\)
* 添加 BERT WordPiece 分词器 \([Link](https://github.com/eclipse/deeplearning4j/pull/7141)\)
* Deeplearning4j UI现在支持多用户/多会话 - 用 `UIServer.getInstance(boolean multiSession, Function<String,StatsStorage>)` 在多会话模式下启动UI \([Link](https://github.com/eclipse/deeplearning4j/pull/7185)\)
* Layer/NeuralNetworkConfiguration builder 方法验证标准化和改进\([Link](https://github.com/eclipse/deeplearning4j/pull/7218)\)
* WordVectorSerializer 现在支持读取和导出文本格式向量，通过 WordVectorSerializer.writeLookupTable 与 readLookupTable方法 \([Link](https://github.com/eclipse/deeplearning4j/pull/7219)\]
* 更新JavaCPP, JavaCPP presets, 与  JavaCV 到 1.5版本  \([Link](https://github.com/eclipse/deeplearning4j/pull/7296)\)
* 增加了EvaluationBinary报警失败率计算\([Link](https://github.com/eclipse/deeplearning4j/pull/7320)\)
* ComputationGraph GraphBuilder有一个 appendLayer 方法可用于添加层连接到上次添加的层/顶点 \([Link](https://github.com/eclipse/deeplearning4j/pull/7403)\)
* 添加 Wasserstein损失函数 \([Link](https://github.com/eclipse/deeplearning4j/pull/7406)\)
* Keras导入：改进了lambda层导入的错误/异常 \([Link](https://github.com/eclipse/deeplearning4j/pull/7535)\)
* Apache Lucene/Solr从7.5.0升级到7.7.1 \([Link](https://github.com/eclipse/deeplearning4j/pull/7539)\)
* KMeans聚类策略现在是可配置的 \([Link](https://github.com/eclipse/deeplearning4j/pull/7471)\)

#### Deeplearning4J: Bug修复和优化

* DL4J Spark 训练：修复共享集群（多个同时训练作业）-现在随机生成的Aeron流ID（[链接](https://github.com/eclipse/deeplearning4j/pull/6673)）
* 如果抛出内存不足异常（[Link](https://github.com/eclipse/deeplearning4j/issues/6691)），cuDNN帮助程序将不再试图依靠内置层实现
* 批量归一化全局方差重新参数化，以避免在分布式训练期间出现下溢和零/负方差（[Link](https://github.com/eclipse/deeplearning4j/issues/6750)）
* 修复了在使用dropout的迁移学习时在层之间错误共享dropout实例的错误\([Link](https://github.com/eclipse/deeplearning4j/issues/6756),[Link](https://github.com/eclipse/deeplearning4j/pull/6758)\)
* 修正了tensorAlongDimension可能导致边界情况的数组顺序不正确，从而导致LSTMs（Link）中出现异常的问题（[Link](https://github.com/eclipse/deeplearning4j/issues/6770)）
* 修复了ComputationGraph.getParam\(String\)的边界情况问题，其中层名称包含下划线（[Link](https://github.com/eclipse/deeplearning4j/issues/6734)）
* 修复了在CUDA上使用ParallelInference的边界情况，其中（很少）在线程（[链接](https://github.com/eclipse/deeplearning4j/issues/6730)、[链接](https://github.com/eclipse/deeplearning4j/pull/6774)）之间传输数组之前，输入数组操作（如归一化）可能无法完全完成
* 修复了当示例总数不是批处理大小（[Link](https://github.com/eclipse/deeplearning4j/issues/6786)，[Link](https://github.com/eclipse/deeplearning4j/pull/6810)）的倍数时使用KFoldIterator的边界情况
* 修复了DL4J UI在Java 9/10/11（[Link](https://github.com/eclipse/deeplearning4j/pull/6819)，[Link](https://github.com/eclipse/deeplearning4j/issues/5804)）上抛出NoClassDefFoundError的问题
* Keras导入：为权重初始化添加别名（[链接](https://github.com/eclipse/deeplearning4j/pull/6849)）
* 修复了在克隆网络配置时无法正确克隆dropout实例的问题（[链接](https://github.com/eclipse/deeplearning4j/issues/6841)）
* 修复了具有单个输入的ElementwiseVertex的工作区问题（[链接](https://github.com/eclipse/deeplearning4j/issues/6811)）
* 修复了用户界面中分离StatsStorage可能会尝试两次删除存储的问题，从而导致异常（[链接](https://github.com/eclipse/deeplearning4j/issues/6859)）
* 修复了当小批量中的所有标签都是同一类时，LossMultiLabel将生成nan的问题。现在返回0梯度。（[链接](https://github.com/eclipse/deeplearning4j/pull/6893)，[链接](https://github.com/eclipse/deeplearning4j/issues/6880)）
* 修复了从保存的格式（[链接](https://github.com/eclipse/deeplearning4j/issues/6911)）还原网络时DepthwiseConv2D权重可能形状错误的问题
* 修复了BaseDatasetIterator.next\(\)在设置预处理器（[链接](https://github.com/eclipse/deeplearning4j/issues/6941)）时不应用预处理器的问题
* 改进了CenterLossOutputLayer（[链接](https://github.com/eclipse/deeplearning4j/issues/6812)）的默认配置
* 修复了UNet非预训练配置（[Link](https://github.com/eclipse/deeplearning4j/issues/6955)）的问题
* 修复了Word2Vec VocabConstructor在某些情况下可能死锁的问题（[链接](https://github.com/eclipse/deeplearning4j/pull/7048)）
* SkipGram和CBOW（在Word2Vec中使用）被设置为本地操作以获得更好的性能（[Link](https://github.com/eclipse/deeplearning4j/pull/7059)）
* 修复了在使用InMemoryStatsListener（[链接](https://github.com/eclipse/deeplearning4j/issues/7064)）时保留对分离的StatsListener实例的引用可能导致内存问题的问题
* 优化：工作区已添加到SequenceVectors和Word2Vec（[链接](https://github.com/eclipse/deeplearning4j/pull/7087)）
* 为RecordReaderDataSetIterator改进校验 \([Link](https://github.com/eclipse/deeplearning4j/issues/7140)\)
* 改进的WordVectors实现中的未知词处理\([Link](https://github.com/eclipse/deeplearning4j/pull/7154)\)
* Yolo2OutputLayer: 添加了对错误标签形状的验证。（[链接](https://github.com/eclipse/deeplearning4j/issues/7152)）
* LastTimeStepLayer 当输入掩码全部为0时，现在将引发异常（没有数据-没有最后一个时间步）（[链接](https://github.com/eclipse/deeplearning4j/issues/7115)）
* 修复了MultiLayerNetwork/ComputationGraph.setLearningRate方法在某些罕见情况下可能导致更新器状态无效的问题（[Link](https://github.com/eclipse/deeplearning4j/issues/6809)）
* 修正了Conv1D 层在MultiLayerNetwork.summary\(\)中计算输出长度的问题（[Link](https://github.com/eclipse/deeplearning4j/issues/7104)）
* 异步迭代器现在用于EarlyStoppingTrained中，以提高数据加载性能（[Link](https://github.com/eclipse/deeplearning4j/issues/7190)）
* EmbeddingLayer 与 EmbeddingSequenceLayer在CUDA上的性能已经被提升 \([Link](https://github.com/eclipse/deeplearning4j/issues/7180)\)
* 删除了过时的/遗留的scala工具库\([Link](https://github.com/eclipse/deeplearning4j/pull/7220), [Link](https://github.com/eclipse/deeplearning4j/issues/7216)\)
* 修复L2NormalizeVertex equals/hashcode方法中的问题（[链接](https://github.com/eclipse/deeplearning4j/issues/7225)）
* 修复了ConvolutionalListener中的工作区问题（[Link](https://github.com/eclipse/deeplearning4j/issues/7217)）
* 修复了 EvaluationBinary falsePositiveRate 计算 \([Link](https://github.com/eclipse/deeplearning4j/pull/7313)\)
* 为MultiLayerNetwork.output（DataSetIterator）方法添加了验证和有用的异常\([Link](https://github.com/eclipse/deeplearning4j/issues/7352)\)
* 修复了在尚未调用init\(\)的情况下ComputationGraph.summary\(\)将引发NullPointerException的小问题 \([Link](https://github.com/eclipse/deeplearning4j/pull/7353)\)
* 修正了一个计算图问题，在训练过程中，输入到一个单层/顶点重复多次可能会失败 \([Link](https://github.com/eclipse/deeplearning4j/pull/7420)\)
* 改进KMeans实现的性能 \([Link](https://github.com/eclipse/deeplearning4j/pull/7427)\)
* 修复了“wrapper”层（如FrozenLayer）中RNNs的rnnGetPreviousState问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7437)\)
* Keras导入：修复了导入某些Keras分词器时出现的单词顺序问题（[链接](https://github.com/eclipse/deeplearning4j/issues/7448)）
* Keras导入：修复了KerasTokenizer类中可能存在UnsupportedOperationException异常的问题\([Link](https://github.com/eclipse/deeplearning4j/issues/7073)\)
* Keras导入：修复了一个导入问题，模型结合了嵌入、变形和卷积层 \([Link](https://github.com/eclipse/deeplearning4j/issues/7013)\)
* Keras导入：修复了一些RNN模型的输入类型推理的导入问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/6995)\)
* 修复在  LocallyConnected1D/2D 层中的一些填充问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7541)\)

### [ND4J 与 SameDiff](fa-xing-shuo-ming.md)

#### ND4J/SameDiff: 功能和增强

* 删除了对处理工作区外（分离的）INDArrays内存管理的定期垃圾收集调用的依赖 \([Link](https://github.com/eclipse/deeplearning4j/pull/7506)\)
* 添加了INDArray.close\(\)方法，允许用户立即手动释放堆外内存 \([Link](https://github.com/eclipse/deeplearning4j/pull/6883)\)
* SameDiff: 添加了TensorFlowImportValidator工具以确定是否可以将TensorFlow图导入SameDiff。报告使用的操作以及SameDiff中是否支持这些操作 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/imports/tensorflow/TensorFlowImportValidator.java)\)
* 添加了Nd4j.createFromNpzFile方法以加载Numpy npz文件 \([Link](https://github.com/eclipse/deeplearning4j/pull/6837)\)
* 添加了对将BERT模型导入SameDiff的支持 \([Link](https://github.com/eclipse/deeplearning4j/pull/7245), [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-tests/src/test/java/org/nd4j/imports/TFGraphs/BERTGraphTest.java)\)
* 添加SameDiff GraphTransformUtil用于迁移学习和其他图变更 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff/transform/GraphTransformUtil.java), [Link](https://github.com/eclipse/deeplearning4j/issues/7199), [Link](https://github.com/eclipse/deeplearning4j/pull/7245)\)
* Evaluation, RegressionEvaluation等现在支持4D（CNN分段）数据格式；还添加了Evaluation.setAxis\(int\)方法，以支持如CNN1的channels-last/NHWC和CNN1D/RNNs的NWC的其他通道数据格式。默认为轴1（匹配DL4J CNN和RNN数据格式） \([Link](https://github.com/eclipse/deeplearning4j/issues/7250), [Link](https://github.com/eclipse/deeplearning4j/pull/7293)\)
* Added basic \("technology preview"\) of SameDiff UI. Should be considered early WIP with breaking API changes expected in future releases. Supports plotting of SameDiff graphs as well as various metrics \(line charts, histograms, etc\)添加了SameDiff UI的基本（“技术预览”）。应被视为早期WIP，并在未来版本中打破预期的API更改。支持绘制SameDiff图以及各种度量（折线图、直方图等）
  * 当前嵌入在DL4J UI中 - 调用 `UIServer.getInstance()` 然后访问 `localhost:9000/samediff` 。
  * 更多详情请查看 [1](https://github.com/eclipse/deeplearning4j/pull/7057), [2](https://github.com/eclipse/deeplearning4j/pull/7062), [3](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-ui-parent/deeplearning4j-play/src/test/java/org/deeplearning4j/ui/play/TestSameDiffUI.java)
* 添加 DotProductAttention 与 MultiHeadDotProductAttention 操作 \([Link](https://github.com/eclipse/deeplearning4j/blob/3ea2876dddeba62e999034388408225aeb34085b/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff/ops/SDNN.java#L789-L925)\)
* 添加 Nd4j.exec\(Op\) 与 Nd4j.exec\(CustomOp\) 方便的方法 \([Link](https://github.com/eclipse/deeplearning4j/issues/7024)\)
* ND4J/SameDiff -新操作被添加:
  * [NonMaxSuppression](https://github.com/eclipse/deeplearning4j/pull/6685), [LogMatrixDeterminant](https://github.com/eclipse/deeplearning4j/pull/6689), [NthElement](https://github.com/eclipse/deeplearning4j/pull/6699), [TruncateMod](https://github.com/eclipse/deeplearning4j/pull/6699)
  * [Cholesky Decomposition](https://github.com/eclipse/deeplearning4j/pull/6703), [Image resize nearest neighbor](https://github.com/eclipse/deeplearning4j/pull/6705), [crop\_and\_resize](https://github.com/eclipse/deeplearning4j/pull/6711)
  * [fake\_quant\_with\_min\_max\_vars](https://github.com/eclipse/deeplearning4j/pull/6711), [reduce\_logsumexp](https://github.com/eclipse/deeplearning4j/pull/6711), [pow \(broadcastable\)](https://github.com/eclipse/deeplearning4j/pull/6944)\), [linspace \(dynamic args\)](https://github.com/eclipse/deeplearning4j/issues/6723)
  * [ExtractImagePatches](https://github.com/eclipse/deeplearning4j/issues/6668), [GELU](https://github.com/eclipse/deeplearning4j/pull/7132), [LSTMBlockCell, LSTMBLock, GRUCell](https://github.com/eclipse/deeplearning4j/pull/7208)
  * [Standardize and LayerNorm ops](https://github.com/eclipse/deeplearning4j/pull/7251)
* SameDiff TensorFlow 导入
  * TF Assertions导入被添加 \([Link](https://github.com/eclipse/deeplearning4j/pull/6710)\)
  * Support/fixes 用于控制依赖 \([Link](https://github.com/eclipse/deeplearning4j/pull/6967)\)
  * Support/fixes 用于TensorArray 和相关操作 \([Link](https://github.com/eclipse/deeplearning4j/issues/6972), [Link](https://github.com/eclipse/deeplearning4j/pull/6976), [Link](https://github.com/eclipse/deeplearning4j/pull/6996)\)
* nd4j-common - tar/tar.gz 支持被添加; Zip文件列且与单文件抽取被添加  \([Link](https://github.com/eclipse/deeplearning4j/issues/6686), [Link](https://github.com/eclipse/deeplearning4j/pull/6729)\)
* SameDiff: 缩减操作现在支持轴参数的“动态”（非常量）输入。 \([Link](https://github.com/eclipse/deeplearning4j/pull/6906)\)
* ROCBinary 现在有.getROC\(int outputNum\) 方法 \([Link](https://github.com/eclipse/deeplearning4j/issues/7074)\)
* SameDiff: L1/L2 正则化被添加\([Link](https://github.com/eclipse/deeplearning4j/issues/7076), [Link](https://github.com/eclipse/deeplearning4j/pull/7128)\)
* SameDiff: 添加 SDVariable.convertToVariable\(\) 与  convertToConstant\(\) - 以改变 SDVariable 类型 \([Link](https://github.com/eclipse/deeplearning4j/pull/7162)\)
* 为空数组上的缩减添加了检查和有用的异常\([Link](https://github.com/eclipse/deeplearning4j/issues/7143)\)
* SameDiff“op creator”方法（SameDiff.tanh\(\)、SameDiff.conv2d\(…\)等）已移动到子类-通过SameDiff.math\(\)/random\(\)/nn\(\)/cnn\(\)/rnn\(\)/loss\(\)方法或SameDiff.math/random/nn/cnn/rnn/loss字段访问creator \([Link](https://github.com/eclipse/deeplearning4j/pull/7174)\)
* SameDiff TensorFlow 导入: 对于用户定义函数等情况，现在可以重写导入 \([Link](https://github.com/eclipse/deeplearning4j/pull/7184), [Link](https://github.com/eclipse/deeplearning4j/issues/7178)\)
* Libnd4j \(c++\) 增加基准测试框架 \([Link](https://github.com/eclipse/deeplearning4j/pull/7241)\)
* 添加了OpExecutioner.inspectArray\(INDArray\)方法以获取用于分析/调试的摘要统计信息（[链接](https://github.com/eclipse/deeplearning4j/pull/7258)）
* 添加 `INDArray.reshape(char order, boolean enforceView, long... newShape)` 来变形数组 ，如果变形无法执行并抛出异常\(而不是返回一个拷贝\) \([Link](https://github.com/eclipse/deeplearning4j/issues/7292), [Link](https://github.com/eclipse/deeplearning4j/commit/8d2bfbb8cca1597ef93dedd62ac0f0625cccc4ee)\)
* 为Kotlin添加了SDVariable方法重载（plus, minus, times, 等） \([Link](https://github.com/eclipse/deeplearning4j/issues/7367)\)
* 增加了dot, reshape, permute的SDVariable便利方法 \([Link](https://github.com/eclipse/deeplearning4j/pull/7371)\)
* 添加SameDiff SDIndex.point\(long, boolean keepDim\)方法（将输出数组中的点索引保持为大小1轴） \([Link](https://github.com/eclipse/deeplearning4j/pull/7392)\)
* 添加了SameDiff ProtoBufToFlatBufConversion命令行工具，用于将TensorFlow冻结模型（protobuf）转换为SameDiff FlatBuffers \([Link](https://github.com/eclipse/deeplearning4j/pull/7460)\)
* 改进了SameDiff操作的数据类型验证 \([Link](https://github.com/eclipse/deeplearning4j/issues/6861)\)

#### ND4J/SameDiff: API 改变 \(迁移指南\): 1.0.0-beta3 到 1.0.0-beta4

* ND4J数据类型-重大更改，请参阅本节顶部的突出显示
* nd4j-base64 模块 \(在 beta3中弃用\) 已经被移除 . Nd4jBase64 类已被移到 nd4j-api \([Link](https://github.com/eclipse/deeplearning4j/pull/6672)\)
* 在指定执行沿维度操作（例如，缩减）的参数时，现在在操作构造函数中指定了缩减轴，而不是在OpExecutioner调用中单独指定。 \([Link](https://github.com/eclipse/deeplearning4j/pull/6902)\)
* 移除旧的基于Java循环的BooleanIndexing方法。应该使用等效的本地操作。 \([Link](https://github.com/eclipse/deeplearning4j/issues/7007)\)
* 移除 Nd4j.ENFORCE\_NUMERICAL\_STABILITY, Nd4j.copyOnOps, 等 \([Link](https://github.com/eclipse/deeplearning4j/issues/6884)\)
* SameDiff“op creator”方法（SameDiff.tanh\(\)、SameDiff.conv2d\(…\)等）已移动到子类-通过SameDiff.math\(\)/random\(\)/nn\(\)/cnn\(\)/rnn\(\)/loss\(\)方法或SameDiff.math/random/nn/cnn/rnn/loss字段访问creator \([Link](https://github.com/eclipse/deeplearning4j/pull/7174)\)
* Nd4j.emptyLike\(INDArray\) 已被移除。使用 Nd4j.like\(INDArray\) 代替 \([Link](https://github.com/eclipse/deeplearning4j/issues/7252)\)
* org.nd4jutil.StringUtils 被移除;  建议使用Apache commons lang3 StringUtils 代替 \([Link](https://github.com/eclipse/deeplearning4j/pull/7487)\)
* ND4J Jackson RowVector\(De\)Serializer由于数据类型更改，已弃用 ; NDArrayText\(De\)Serializer 应该用来代替它 \([Link](https://github.com/eclipse/deeplearning4j/issues/7561), [Link](https://github.com/eclipse/deeplearning4j/pull/7577)\)
* nd4j-instrumentation 模块由于缺乏使用/维护而被移除 \([Link](https://github.com/eclipse/deeplearning4j/pull/7627)\)

#### ND4J/SameDiff: Bug修复与优化

* 修复了带有InvertMatrix.invert\(\)和\[1,1\]形状矩阵的错误 \([Link](https://github.com/eclipse/deeplearning4j/issues/6728)\)
* 修复了长度为1的状态数组的更新器实例的边界情况缺陷 \([Link](https://github.com/eclipse/deeplearning4j/issues/6671)\)
* 修复了带有空文档的FileDocumentIterator的边界情况缺陷 \([Link](https://github.com/eclipse/deeplearning4j/issues/6712)\)
* SameDiff: 多项修复和增强
  * [1](https://github.com/eclipse/deeplearning4j/issues/6674), [2](https://github.com/eclipse/deeplearning4j/pull/6816), [3](https://github.com/eclipse/deeplearning4j/issues/7001), [4](https://github.com/eclipse/deeplearning4j/pull/7099)
  * 改进的损失函数功能 \([Link](https://github.com/eclipse/deeplearning4j/pull/6844), [Link](https://github.com/eclipse/deeplearning4j/pull/7020), [Link](https://github.com/eclipse/deeplearning4j/pull/7022), [Link](https://github.com/eclipse/deeplearning4j/pull/7042)\)
  * 改进了占位符丢失/拼写错误的错误 \([Link](https://github.com/eclipse/deeplearning4j/issues/5299)\)
  * 修复在循环中的边界缺陷\([Link](https://github.com/eclipse/deeplearning4j/pull/7033), [Link](https://github.com/eclipse/deeplearning4j/pull/7035)\)
* 修复了1d矩阵上Nd4j.vstack返回1d输出而不是二维叠加输出的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/6985)\)
* Conv2D 操作可以根据需要直接从输入数组推断内核大小 \([Link](https://github.com/eclipse/deeplearning4j/pull/7098), [Link](https://github.com/eclipse/deeplearning4j/issues/7008)\)
* 修复了Numpy格式导出的问题 - `Nd4j.toNpyByteArray(INDArray)` \([Link](https://github.com/eclipse/deeplearning4j/issues/7466)\)
* 当SameDiff在外部工作区中使用时的修复 \([Link](https://github.com/eclipse/deeplearning4j/pull/7124)\)
* 修复了一个问题，即空NDarray可能被报告为具有标量形状信息，长度为1 \([Link](https://github.com/eclipse/deeplearning4j/pull/7163)\)
* 优化: libnd4j \(c++\) 操作的索引将在需要和可能的情况下使用uint进行更快的偏移计算。 \([Link](https://github.com/eclipse/deeplearning4j/pull/7164)\)
* 优化: libnd4j 为了加快某些操作的执行速度，循环性能得到了改进 \([Link](https://github.com/eclipse/deeplearning4j/pull/7176), [Link](https://github.com/eclipse/deeplearning4j/pull/7187), [Link](https://github.com/eclipse/deeplearning4j/pull/7200)\)
* 局部响应归一化操作优化 \([Link](https://github.com/eclipse/deeplearning4j/pull/7236), [Link](https://github.com/eclipse/deeplearning4j/pull/7244)\)
* 修复了INDArray.repeat在某些视图数组上的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7277)\)
* 在视图阵列上执行某些操作的性能得到改进 \([Link](https://github.com/eclipse/deeplearning4j/pull/7295)\)
* 提高广播操作性能 \([Link](https://github.com/eclipse/deeplearning4j/pull/7303), [Link](https://github.com/eclipse/deeplearning4j/pull/7334), [Link](https://github.com/eclipse/deeplearning4j/pull/7396)\)
* 改进了non-EWS降维操作的性能 \([Link](https://github.com/eclipse/deeplearning4j/pull/7304)\)
* 改进了IndexReduce 操作 \([Link](https://github.com/eclipse/deeplearning4j/pull/7308)\) 与小缩减的性能 \([Link](https://github.com/eclipse/deeplearning4j/pull/7348)\)
* 改进了one\_hot 操作 \([Link](https://github.com/eclipse/deeplearning4j/pull/7339)\), tanh 操作的性能 \([Link](https://github.com/eclipse/deeplearning4j/pull/7379)\)
* 改进转换操作的性能 \([Link](https://github.com/eclipse/deeplearning4j/pull/7359)\)
* 优化: 空数组只创建一次并缓存（因为它们是不可变的） \([Link](https://github.com/eclipse/deeplearning4j/issues/7168)\)
* 利用张量沿维并行化提高运算性能 \([Link](https://github.com/eclipse/deeplearning4j/pull/7347), [Link](https://github.com/eclipse/deeplearning4j/pull/7557)\)
* 改进“reduce 3”缩减操作的性能 \([Link](https://github.com/eclipse/deeplearning4j/pull/7464)\)
* 改进了在多线程环境中对CUDA上下文的处理 \([Link](https://github.com/eclipse/deeplearning4j/pull/7434)\)
* 修复了Evaluation.reset\(\)无法正确清除字符串类标签的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7435)\)
* SameDiff: 改进的梯度计算性能/效率；现在不再为非浮点变量定义“梯度”，也不再为不需要计算损失或参数梯度的变量定义“梯度” \([Link](https://github.com/eclipse/deeplearning4j/pull/7452)\)
* IEvaluation实例的行为现在不再依赖于全局（默认）数据类型设置 \([Link](https://github.com/eclipse/deeplearning4j/issues/7550)\)
* INDArray.get\(point\(x\), y\) 或 .get\(y, point\(x\)\) 现在，当对rank 2数组执行时，返回rank 1数组\([Link](https://github.com/eclipse/deeplearning4j/issues/7092)\)
* 删除了对SameDiff的Guava的依赖，修复了Java 11/12以及早期版本的Guava在类路径上的潜在问题。 \([Link](https://github.com/eclipse/deeplearning4j/pull/7575), [Link](https://github.com/eclipse/deeplearning4j/issues/7170)\)
* 为更好的性能和可靠性改写ND4J索引\(INDArray.get\)实现\([Link](https://github.com/eclipse/deeplearning4j/pull/7577)\)
* 本地响应归一化反向传播操作的修复 \([Link](https://github.com/eclipse/deeplearning4j/pull/7597)\)

#### ND4J: 已知问题

* 大多数CustomOperation操作（如SameDiff中使用的操作）在下一版本之前都是CPU。未能及时完成1.0.0-beta4版本的GPU支持。
* 一些使用Intel Skylake CPU的用户报告说，当OMP\_NUM\_THREADS设置为8或更高时，MKL-DNN卷积2d反向传播操作（DL4J卷积层反向传播，ND4J“conv2d\_bp”操作）出现死锁。调查表明，这可能是MKL-DNN的问题，而不是DL4J/ND4J的问题。见[7637](https://github.com/eclipse/deeplearning4j/issues/7637)号问题。解决方法：通过ND4J\_MKL\_FALLBACK（见前面）禁用用于conv2d\_bp操作的MKL-DNN，或对Skylake cpu全局禁用MKL-DNN。

### [DataVec](fa-xing-shuo-ming.md)

#### DataVec: 功能与增强

* 添加了PythonTransform（预处理的任意python代码执行） \([Link](https://github.com/eclipse/deeplearning4j/pull/7188), [Link](https://github.com/eclipse/deeplearning4j/blob/master/datavec/datavec-python/src/main/java/org/datavec/python/PythonTransform.java)\)
* 添加FirstDigit（本福德定律）变换\([Link](https://github.com/eclipse/deeplearning4j/issues/7260), [Link](https://github.com/eclipse/deeplearning4j/blob/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/categorical/FirstDigitTransform.java)\)
* StringToTimeTransform 现在支持设置区域 \([Link](https://github.com/eclipse/deeplearning4j/pull/6901), [Link](https://github.com/eclipse/deeplearning4j/issues/6825)\)
* 添加了StreamInputSplit，用于创建本地数据管道，其中数据远程存储在HDFS或S3等存储设备上\([Link](https://github.com/eclipse/deeplearning4j/blob/master/datavec/datavec-api/src/main/java/org/datavec/api/split/StreamInputSplit.java), [Link](https://github.com/eclipse/deeplearning4j/issues/7351)\)
* LineRecordReader （和子类型）现在可以选择定义字符集 \([Link](https://github.com/eclipse/deeplearning4j/issues/7407)\)
* 添加 TokenizerBagOfWordsTermSequenceIndexTransform \(TFIDF 转换\), GazeteerTransform \(单词表示的二进制向量\) 与 MultiNlpTransform 转换; 添加 BagOfWordsTransform 接口 \([Link](https://github.com/eclipse/deeplearning4j/pull/7542)\)

#### DataVec: 优化与缺陷修复

* 修复了 ImageLoader.scalingIfNeeded 的问题\([Link](https://github.com/eclipse/deeplearning4j/pull/7159)\)

### [Arbiter](fa-xing-shuo-ming.md)

#### Arbiter: 增强

* Arbiter 现在支持遗传算法搜索 \([Link](https://github.com/eclipse/deeplearning4j/pull/7081)\)

#### Arbiter: 修复

* 修复了Arbiter中使用的提前停止将导致序列化异常的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/7029)\)

## [1.0.0-beta3版本发行说明](fa-xing-shuo-ming.md)

### 1.0.0-beta3 发行亮点

* ND4J/Deeplearning4j: 增加了对CUDA 10.0的支持。放弃了对CUDA 8.0的支持。（1.0.0-beta3版本支持CUDA 9.0、9.2和10.0）
* SameDiff现在支持从DataSetIterator和MultiDataSetIterator进行训练和评估。 评估类已经被迁移到ND4J.
* DL4J Spark 训练（梯度共享）现在是完全容错的，并且对阈值自适应（可能更稳健的收敛）有改进。现在可以在master/worker上轻松地独立配置端口。

### [Deeplearning4J](fa-xing-shuo-ming.md)

#### Deeplearning4J: 新功能

* Added 添加OutputAdapter 接口和 `MultiLayerNetwork/ComputationGraph.output` 方法，使用OutputAdapter重载 \(GC以免分配需要通过GC清理的堆外内存\) [Link](https://github.com/eclipse/deeplearning4j/pull/6229), [Link,](https://github.com/eclipse/deeplearning4j/blob/37e053ad901ad364b6fe6658517b8d2e5e7d8894/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/adapters/ArgmaxAdapter.java) [Link](https://github.com/eclipse/deeplearning4j/blob/6bef4d587da9471e885a1616eb3f13239d91face/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/multilayer/MultiLayerNetwork.java#L2300-L2316)
* 使用用户指定的工作区添加了ComputationGraph/MultiLayerNetwork rnnTimeStep重载。 [Link](https://github.com/eclipse/deeplearning4j/pull/6295)
* 添加 Cnn3DLossLayer [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Cnn3DLossLayer.java)
* ParallelInference:实例现在可以实时更新模型（无需重新初始化） [Link](https://github.com/eclipse/deeplearning4j/pull/6190)
* ParallelInferenc: 添加 ParallelInference 的原地模式 [Link](https://github.com/eclipse/deeplearning4j/pull/6229)
* 增加了对不兼容损失/激活函数组合的验证\(如softmax+nOut=1, 或 sigmoid+mcxent\)。可以使用outputValidation\(false\)禁用验证 [Link](https://github.com/eclipse/deeplearning4j/issues/6280)
* Spark 训练: 为梯度共享实现增加了完全容错（健壮的故障恢复） [Link](https://github.com/eclipse/deeplearning4j/pull/6115) [Link](https://github.com/eclipse/deeplearning4j/pull/6455)
* Spark 训练现在支持更灵活地配置端口（不同的worker有不同的配置），使用 PortSupplier [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-parameter-server-parent/nd4j-parameter-server-node/src/main/java/org/nd4j/parameterserver/distributed/v2/transport/PortSupplier.java) [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-parameter-server-parent/nd4j-parameter-server-node/src/main/java/org/nd4j/parameterserver/distributed/v2/transport/impl/EnvironmentVarPortSupplier.java)
* Spark 训练: 改进了梯度共享阈值自适应算法；使自定义阈值设置成为可能，并且使默认值对初始阈值配置更加健壮，在某些情况下提高了收敛速度。 [Link](https://github.com/eclipse/deeplearning4j/pull/6631)
* Spark 训练: 为大消息实现分块消息传递以减少内存需求（以及缓冲区长度不足问题） [Link](https://github.com/eclipse/deeplearning4j/pull/6115)
* Spark 训练: 添加 MeshBuildMode 配置用于提高了大型集群的可扩展性  [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-parameter-server-parent/nd4j-parameter-server-node/src/main/java/org/nd4j/parameterserver/distributed/v2/enums/MeshBuildMode.java)
* Spark 网络数据管道: 为“小文件”（图像等）分布式训练用例添加了FileBatch、FileBatchRecordReader等 [Link](https://github.com/eclipse/deeplearning4j/pull/6601)
* 为容错/调试目的添加了FailureTestingListener [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/FailureTestingListener.java)
* 将Apache Lucene/Solr升级到7.5.0版（从7.4.0版） [Link](https://github.com/eclipse/deeplearning4j/pull/6485)
* 添加系统属性 \(`org.deeplearning4j.tempdir` 与 `org.nd4j.tempdir`\) 允许重写ND4J和DL4J使用的临时目录 [Link](https://github.com/eclipse/deeplearning4j/issues/6362) [Link](https://github.com/eclipse/deeplearning4j/commit/21dc50fb069f4584df8560340c56f1be2bf2430e)
*  MultiLayerNetwork/ComputationGraph.clearLayerStates 方法修改为  public \(以前是 protected\) [Link](https://github.com/eclipse/deeplearning4j/commit/dc192d29257736995f7878f32576f206ef13eac0)
* `AbstactLayer.layerConf()` 方法现在是 public [Link](https://github.com/eclipse/deeplearning4j/pull/6553)
* ParallelWrapper模块现在不再有artifact id的Scala版本后缀; 新 artifact id 是 `deeplearning4j-parallel-wrapper` [Link](https://github.com/eclipse/deeplearning4j/pull/6560)
* 在Yolo2OutputLayer中改进了无效输入/标签的验证和错误消息 [Link](https://github.com/eclipse/deeplearning4j/issues/6584)
* Spark 训练: 添加 SharedTrainingMaster.Builder.workerTogglePeriodicGC 和  .workerPeriodicGCFrequency 在worker上轻松配置ND4J垃圾回收配置。在worker上将默认GC设置为5秒 [Link](https://github.com/eclipse/deeplearning4j/pull/6604)
* Spark 训练: 添加了阈值编码调试模式（在训练 期间记录每个worker的当前阈值和编码统计信息）。使用 `SharedTrainingConfiguration.builder.encodingDebugMode(true)`启用。注意这个操作有计算开销。 [Link](https://github.com/eclipse/deeplearning4j/pull/6622)

#### Deeplearning4J: Bug 修复与优化 

*  修复了L1/L2和更新器（Adam、Nesterov等）在用小批量划分梯度以获得平均梯度之前被应用的问题。保持旧的行为，使用`NeuralNetConfiguration.Builder.legacyBatchScaledL2(true)` [Link](https://github.com/eclipse/deeplearning4j/blob/87167e91c616584a296abe637d408a8efd9e05b7/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/NeuralNetConfiguration.java#L1034-L1045).
  * 请注意，对于某些更新器（如Adam）来说，学习率可能需要降低，以解释与早期版本相比的这一变化。其他一些更新器（如SGD、NoOp等）应该不受影响。
  * 请注意，保存在1.0.0-beta2或更早版本中的反序列化（加载）配置/网络将默认为旧行为，以实现向后兼容性。所有新网络（在1.0.0-beta3中创建）将默认为新行为。
* 修复了EarlyStoppingScoreCalculator不能正确处理“最大分数”的情况而不是最小化的问题。 [Link](https://github.com/eclipse/deeplearning4j/issues/6237)
* 修复VGG16ImagePreProcessor通道偏移值的顺序（BGR与RGB） [Link](https://github.com/eclipse/deeplearning4j/pull/6254)
* 基于权值噪声的变分自编码缺陷修复 [Link](https://github.com/eclipse/deeplearning4j/pull/6289)
* 修复BaseDataSetIterator 不遵循 'maximum examples' 配置的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6283)
* 优化: 工作区现在用于计算图/多层网络评估方法（避免在计算期间分配必须由垃圾收集器清理的堆外内存） [Link](https://github.com/eclipse/deeplearning4j/pull/6295)
* 修复了一个问题，即混洗与MnistDataSetIterator的子集组合在一起时，在重置之间不会保持相同的子集 [Link](https://github.com/eclipse/deeplearning4j/issues/6299)
* 修复StackVertex.getOutputType的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6128)
* 修复CNN到/从RNN预处理器处理掩码数组的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6316)
* 修复了模型动物园中VGG16非预训练配置的问题 [Link](https://github.com/eclipse/deeplearning4j/pull/6348)
* 修复 了TransferLearning nOutReplace中一行中多个层被修改的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6343)
* 修复了CuDNN工作区在标准fit调用之外执行反向传播的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6358)
* 修复了在计算图中的输出层上过早清除丢弃掩码的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6326)
* RecordReaderMultiDataSetIterator 现在支持5D数组  \(用于 3D CNNs\) [Link](https://github.com/eclipse/deeplearning4j/issues/6366)
* 修复了TBPTT结合掩码和不同数量输入输出数组的多输入输出计算图中的缺陷 [Link](https://github.com/eclipse/deeplearning4j/pull/6375)
* 改进了批归一化层的输入验证/异常 [Link](https://github.com/eclipse/deeplearning4j/issues/6403)
* 修复了TransferLearning GraphBuilder nOutReplace与子采样层结合时的错误 [Link](https://github.com/eclipse/deeplearning4j/issues/6389)
* SimpleRnnParamInitializer 现在正确地遵循偏差初始化配置 [Link](https://github.com/eclipse/deeplearning4j/issues/6431)
* 修复SqueezeNet动物园模型非预处理配置 [Link](https://github.com/eclipse/deeplearning4j/issues/6500)
* 修复Xception动物园模型非预训练配置 [Link](https://github.com/eclipse/deeplearning4j/issues/6501)
* 修正了多输出计算图的一些评估签名问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6497)
* 改进的大型网络多层网络/计算图汇总格式[Link](https://github.com/eclipse/deeplearning4j/issues/6502)
* 修正了一个问题，如果一个层中的所有参数的梯度都恰好为0.0，那么梯度归一化可能导致NaNs [Link](https://github.com/eclipse/deeplearning4j/issues/6539#issuecomment-427726265)
* 修复了MultiLayerNetwork/ComputationGraph.setLearningRate可能引发SGD和NoOp更新器异常的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6520)
* 修复了StackVertex加掩码在某些罕见情况下的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6490)
* 修复了1.0.0-alpha之前格式的冻结层的JSON反序列化问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6552)
* 修复了GraphBuilder.removeVertex在某些有限情况下可能失败的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6565)
* 修复 CacheableExtractableDataSetFetcher中的缺陷 [Link](https://github.com/eclipse/deeplearning4j/pull/6602)
* DL4J Spark 训练:修复了多GPU训练+评估的线程/设备关联性问题 [Link](https://github.com/eclipse/deeplearning4j/pull/6614)
* DL4J Spark 训练: 使所有Aeron线程后台程序线程在所有其他线程完成时阻止Aeron停止JVM关闭 [Link](https://github.com/eclipse/deeplearning4j/pull/6614)
* 为批归一化层添加了cudnnAllowFallback配置（如果CuDNN意外失败，则回滚到内置实现） [Link](https://github.com/eclipse/deeplearning4j/pull/6614)
* 修正了Spark训练中多worker（多GPU）节点的一些罕见并发问题[Link](https://github.com/eclipse/deeplearning4j/pull/6618) [Link](https://github.com/eclipse/deeplearning4j/pull/6636)
* 修复了BatchNormalization层的问题，该问题阻止在每个worker上正确同步平均值/方差估计值以进行GradientSharing训练，从而导致收敛问题[Link](https://github.com/eclipse/deeplearning4j/pull/6626)
* 在ArchiveUtils中添加了检测ZipSlip CVE尝试的检查 [Link](https://github.com/eclipse/deeplearning4j/pull/6630)
* DL4J Spark 训练与评估:方法现在使用Spark上下文中的Hadoop配置来确保运行时集配置在Spark函数中可用，这些函数直接从远程存储（HDFS等）读取 [Link](https://github.com/eclipse/deeplearning4j/pull/6633)
* MultiLayerNetwork 与 ComputationGraph 现在支持超过 Integer.MAX\_VALUE 个参数 [Link](https://github.com/eclipse/deeplearning4j/issues/6611) [Link](https://github.com/eclipse/deeplearning4j/pull/6634)
* 为Nd4j.readTxt添加了数据验证-现在对无效输入抛出异常，而不是返回不正确的值 [Link](https://github.com/eclipse/deeplearning4j/issues/6632)
* 修复了KNN实现中使用无效距离函数（返回小于0的“距离”）时可能出现死锁的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6639)
* 在加载Keras导入模型时添加了同步，以避免用于加载的底层HDFS库中出现线程安全问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6649)
* 修复了具有大预取值的Async\(Multi\)DataSetIterator的罕见问题 [Link](https://github.com/eclipse/deeplearning4j/pull/6662)

#### Deeplearning4J: API 修改 \(迁移指南\): 1.0.0-beta2 到 1.0.0-beta3

* DL4J中的IEvaluation类已被弃用并移到ND4J，以便它们可用于SameDiff训练。功能和api不变
* MultiLayerConfiguration/ComputationGraphConfiguration `pretrain(boolean)` 与 `backprop(boolean)` 已被弃用并不再使用。使用 fit 和  pretrain/pretrainLayer 方法来替代 。 [Link](https://github.com/eclipse/deeplearning4j/pull/6296)
* ParallelWrapper 模块现在不再有artifact id的Scala版本后缀; 新 artifact id 为 `deeplearning4j-parallel-wrapper` 应用用来替代 [Link](https://github.com/eclipse/deeplearning4j/pull/6560)
* deeplearning4j-nlp-korean 模块现在有Scala版本后缀，由于scala依赖项; 新artifact ID 是 `deeplearning4j-nlp-korean_2.10` 与 `deeplearning4j-nlp-korean_2.11` [Link](https://github.com/eclipse/deeplearning4j/issues/6306)

#### Deeplearning4J: 已知问题: 1.0.0-beta3

* 在一个物理节点上同时运行多个Spark训练作业（即，来自一个或多个Spark作业的多个jvm）可能会导致网络通信问题。解决方法是在VoidConfiguration中手动设置唯一的流ID。对不同的作业使用唯一（或随机）整数值。 [Link](https://github.com/eclipse/deeplearning4j/blob/b05c95b05404b722f908daf601ba290907d9c81e/nd4j/nd4j-parameter-server-parent/nd4j-parameter-server-node/src/main/java/org/nd4j/parameterserver/distributed/conf/VoidConfiguration.java#L48-L52)

### [Deeplearing4J: Keras 导入](fa-xing-shuo-ming.md)

* 修复了由于Keras 2.2.3+的Keras JSON格式更改而导致的导入问题 [Link](https://github.com/eclipse/deeplearning4j/pull/6590)
* 为时间序列预处理添加了Keras导入 [Link](https://github.com/eclipse/deeplearning4j/pull/6127)
* Elephas [Link](https://github.com/eclipse/deeplearning4j/pull/6197)
* 修复了在嵌入层之后使用导入模型变形的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6175)
* 增加了对keras掩码层的支持 [Link](https://github.com/eclipse/deeplearning4j/pull/6250)
* 修复了某些层/预处理器（如Permute）的JSON反序列化问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6489)
* 修正了Keras导入Nadam配置的问题 [Link](https://github.com/eclipse/deeplearning4j/pull/6646)

### [ND4J](fa-xing-shuo-ming.md)

#### ND4J: 新功能

* 添加了SameDiff训练和评估：SameDiff实例现在可以直接使用DataSetIterator和MultiDataSetIterator进行训练，并使用IEvaluation实例（已从ND4J移动到DL4J）进行评估 [Link](https://github.com/eclipse/deeplearning4j/pull/6599)
* 添加了GraphServer实现：带有Java API的SameDiff（和Tensorflow，通过TF导入）的c++推理服务器 [Link](https://github.com/eclipse/deeplearning4j/pull/6273)
* SameDiff实例现在可以从序列化的FlatBuffers格式（SameDiff.asFlatFile 和 fromFlatFile）加载 [Link](https://github.com/eclipse/deeplearning4j/pull/6484) [Link](https://github.com/eclipse/deeplearning4j/issues/5759)
* 为某些操作（Conv2d等）添加了MKL-DNN支持 [Link](https://github.com/eclipse/deeplearning4j/pull/6204)
* 将ND4J（和DataVec）升级到Arrow 0.11.0 Link，它还修复了 [Link](https://github.com/eclipse/deeplearning4j/issues/6372)
* Added Nd4j.where 操作方法 \(和  numpy.where同样的语义\) [Link](https://github.com/eclipse/deeplearning4j/pull/6242)
* Added Nd4j.stack 操作方法 \(combine arrays 结合数组 +把数组的rank加到1 \) [Link](https://github.com/eclipse/deeplearning4j/blob/663224cc901e99553ff775fb1ebdde479b5648fd/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/factory/Nd4j.java#L5165-L5178)
* Libnd4j 新操作:
  * Matrix band part [Link](https://github.com/eclipse/deeplearning4j/pull/6251)
  * 分散 ND, ND-add, ND-sub 和 ND-更新操作 [Link](https://github.com/eclipse/deeplearning4j/pull/6272)
  * 基于logits的稀疏softmax交叉熵损失函数 [Link](https://github.com/eclipse/deeplearning4j/pull/6307)
  * 直方图固定宽度 [Link](https://github.com/eclipse/deeplearning4j/pull/6325)
  * broadcast\_to 操作 [Link](https://github.com/eclipse/deeplearning4j/pull/6354)
  * deconv3d 操作被添加 [Link](https://github.com/eclipse/deeplearning4j/pull/6387)
  * 添加了未排序的段操作 [Link](https://github.com/eclipse/deeplearning4j/pull/6391)
  * Segment\_X  反向传播被添加 [Link](https://github.com/eclipse/deeplearning4j/pull/6402)
  * batchnorm\_new 操作被添加，支持多轴平均/方差 [Link](https://github.com/eclipse/deeplearning4j/pull/6443)
  * GRU 单元反向传播被添加 [Link](https://github.com/eclipse/deeplearning4j/pull/6588)
* Nd4j Preconditions类现在有格式化INDArray参数的方法[Link](https://github.com/eclipse/deeplearning4j/issues/6451), [Link](https://github.com/eclipse/deeplearning4j/pull/6470)
* SameDiff损失函数：清除加正向传播实现 [Link](https://github.com/eclipse/deeplearning4j/pull/6534)
* CudaGridExecutioner 现在警告异常堆栈跟踪可能会延迟，以避免在异步执行操作期间调试异常时出现混淆 [Link](https://github.com/eclipse/deeplearning4j/issues/6493)
* JavaCPP 与 JavaCPP-presets 已更新到1.4.3版本 [Link](https://github.com/eclipse/deeplearning4j/pull/6587)
* 改进SDVariable 类的 Javadoc [Link](https://github.com/eclipse/deeplearning4j/pull/6661)

#### ND4J: Bug修复与优化

* android端修复: 删除RawIndexer的使用 [Link](https://github.com/eclipse/deeplearning4j/pull/6205)
* Libnd4j 定制操作: 卷积操作权重布局现在不依赖于输入格式\(NCHW/NHWC\) - 现在通常 `[kH, kW, inChannels, outChannels]` 用于 2d CNNs, `[kH, kW, kD, inChannels, outChannels]` 用于 3d CNNs. [Link](https://github.com/eclipse/deeplearning4j/pull/6412), [Link](https://github.com/eclipse/deeplearning4j/issues/6393)
* Libnd4j 本地操作修复:
  * 点积操作反向传播 [Link](https://github.com/eclipse/deeplearning4j/pull/6109), 决定条件 [Link](https://github.com/eclipse/deeplearning4j/pull/6110)
  * 一些成对转换自定义操作实现的广播情况的反向传播修复 [Link](https://github.com/eclipse/deeplearning4j/issues/6037)
  * 修复了带有rank1输入的反向自定义操作 [Link](https://github.com/eclipse/deeplearning4j/issues/6142)
  * ATan2 操作现在是可广播的 [Link](https://github.com/eclipse/deeplearning4j/pull/6157)
  * 布尔自定义操作广播修复/添加 [Link](https://github.com/eclipse/deeplearning4j/issues/6158)
  * 散点操作边界情况修复 [Link](https://github.com/eclipse/deeplearning4j/pull/6167)
  * ArgMin shape函数修复 [Link](https://github.com/eclipse/deeplearning4j/pull/6176), 负轴修复 [Link](https://github.com/eclipse/deeplearning4j/pull/6209)
  * Unique 操作修复 [Link](https://github.com/eclipse/deeplearning4j/pull/6178)
  * Pad 操作修复 [Link](https://github.com/eclipse/deeplearning4j/pull/6191)
  * shape函数操作修复 [Link](https://github.com/eclipse/deeplearning4j/pull/6238)
  * SVD rank 1边界情况修复 [Link](https://github.com/eclipse/deeplearning4j/pull/6239)
  * Range 操作  [Link](https://github.com/eclipse/deeplearning4j/pull/6291)
  * Split 与 space\_to\_batch 修复 [Link](https://github.com/eclipse/deeplearning4j/pull/6318)
  * 广播动态形状 [Link](https://github.com/eclipse/deeplearning4j/pull/6365)
  * embedding\_lookup操作现在支持多输入数组 [Link](https://github.com/eclipse/deeplearning4j/issues/6311)
  * 矩阵行列式操作边界情况（rank 0结果）形状修复 [Link](https://github.com/eclipse/deeplearning4j/issues/6441)
* SameDiff TensorFlow 导入: 多个操作的修复 [Link](https://github.com/eclipse/deeplearning4j/pull/6145), [Link](https://github.com/eclipse/deeplearning4j/pull/6196), [Link](https://github.com/eclipse/deeplearning4j/pull/6236), [Link](https://github.com/eclipse/deeplearning4j/pull/6373)
* SameDiff: 改进的多输出情况下的错误处理 [Link](https://github.com/eclipse/deeplearning4j/pull/6216)
* 修复了INDArray.permute无法正确抛出无效长度情况异常的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6159)
* 修复了INDArray.get/put和SpecifiedIndex的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6341), [Link](https://github.com/eclipse/deeplearning4j/issues/6327)
* DataSet.merge小修改 -现在签名可以接受任何DataSet的子类型  [Link](https://github.com/eclipse/deeplearning4j/pull/6424)
* INDArray.transposei 操作不是原地操作 [Link](https://github.com/eclipse/deeplearning4j/issues/6401)
* 修复 INDArray.mmul 与 MMulTranspose的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6378)
* 为ND4J创建方法添加了额外的顺序 验证\(create, rand, 等\) [Link](https://github.com/eclipse/deeplearning4j/issues/6442)
* 修复从堆字节缓冲区反序列化时ND4J二进制反序列化\(BinarySerde\) 的问题  [Link](https://github.com/eclipse/deeplearning4j/pull/6461)
* Fixed issue with Nd4j-common ClassPathResource path resolution in some IDEs修复了某些ide中Nd4j-common ClassPathResource路径解析的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6483)
* 修复了rank 1数组上的INDArray.get\(interval\)将返回rank 2数组的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6347)
* 修正了Nd4j.gemm/mmuli对视图的验证问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6521) [Link](https://github.com/eclipse/deeplearning4j/issues/6543)
* INDArray.assign\(INDArray\) 不再允许指定不同的形状数组（标量/矢量情况除外） [Link](https://github.com/eclipse/deeplearning4j/issues/6545)
* NDarrayStrings \(and INDArray.toString\(\)\) 现在格式化数字时总是使用US语言环境[Link](https://github.com/eclipse/deeplearning4j/pull/6537)
* 修正了V100 GPU特有的GaussianDistribution问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6518)
* 修复了V100 GPU特有的位图压缩/编码问题 [Link](https://github.com/eclipse/deeplearning4j/pull/6638)
* Transforms.softmax 现在在不支持的形状上抛出错误，而不是简单地不应用操作 [Link](https://github.com/eclipse/deeplearning4j/issues/6512)
* VersionCheck 功能: 处理在早期版本的Android上没有SimpleFileVisitor的情况 [Link](https://github.com/eclipse/deeplearning4j/issues/6609)
* SameDiff 卷积层配置 \(Conv2dConfig/Conv3dConfig/Pooling3dConfig 等\)已将参数名对齐 [Link](https://github.com/eclipse/deeplearning4j/issues/5577)

#### ND4J: API 变更 \(迁移指南\): 1.0.0-beta2 到 1.0.0-beta3

* CUDA 8.0支持已被删除。CUDA 9.0、9.2和10.0支持在1.0.0-beta3中提供
* nd4j-base64模块内容已被弃用；从现在起使用nd4j-api中的等效类 [Link](https://github.com/eclipse/deeplearning4j/pull/6599)
* nd4j jackson模块中的某些类已被弃用；从现在起使用nd4j-api中的等效类 [Link](https://github.com/eclipse/deeplearning4j/pull/6599)

#### ND4J: 已知问题: 1.0.0-beta3

* Android 可能需要手动排除掉模块 nd4j-base64（现在已弃用）。这是由于`org.nd4j.serde.base64.Nd4jBase64` 类在 nd4j-api 和  nd4j-base64 模块中都出现了。两个版本都有相同的内容。使用 `exclude group: 'org.nd4j', module: 'nd4j-base64'` 来排除。

### [DataVec](fa-xing-shuo-ming.md)

#### DataVec: 新功能

* 为org.opencv.core.Mat和字符串文件名添加了NativeImageLoader方法重载 [Link](https://github.com/eclipse/deeplearning4j/pull/6459)

#### DataVec: Bug 修复与优化

* 修复JDBCRecordReader对空值的处理 [Link](https://github.com/eclipse/deeplearning4j/pull/6113)
* 改进了针对无效输入的ObjectDetectionRecordReader的错误/验证（如果图像对象中心位于图像边界之外） [Link](https://github.com/eclipse/deeplearning4j/issues/6101)
* 修复了使用在早期版本的Android上不可用的方法进行FileSplit的问题[Link](https://github.com/eclipse/deeplearning4j/issues/6457)
* 为Spark函数中需要Hadoop配置的情况添加了SerializableHadoopConfiguration和BroadcastHadoopConfigHolder [Link](https://github.com/eclipse/deeplearning4j/blob/master/datavec/datavec-spark/src/main/java/org/datavec/spark/util/SerializableHadoopConfig.java) [Link](https://github.com/eclipse/deeplearning4j/blob/master/datavec/datavec-spark/src/main/java/org/datavec/spark/util/BroadcastHadoopConfigHolder.java)
* 修复了JDBCRecordReader处理实数值列结果类型的问题 [Link](https://github.com/eclipse/deeplearning4j/pull/6617)
* 添加了CSVRecordReader/LineRecordReader在未初始化的情况下使用的验证和有用异常 [Link](https://github.com/eclipse/deeplearning4j/commit/fdffabd38bc8e5f2498a144576864f7dc5c33fa8)

### [Arbiter](fa-xing-shuo-ming.md)

#### Arbiter: 修复

* 修正了一些与dropout层有关的问题 [Link](https://github.com/eclipse/deeplearning4j/pull/6265)

### [RL4J](fa-xing-shuo-ming.md)

* stepCounter, epochCounter 和 historyProcessor 现在可以被设置  [Link](https://github.com/eclipse/deeplearning4j/pull/5972)
* 现在ACPolicy加载随机种子已加载 [Link](https://github.com/eclipse/deeplearning4j/issues/5543)

### [ScalNet](fa-xing-shuo-ming.md)

### [ND4S](fa-xing-shuo-ming.md)

* 添加了org.nd4j.linalg.primitives.Pair/Triple和Scala元组之间的转换 [Link](https://github.com/eclipse/deeplearning4j/pull/6323)

## [1.0.0-beta2版本发行说明](fa-xing-shuo-ming.md)

### 1.0.0-beta2 发行亮点

* ND4J/Deeplearning4j: 增加了对CUDA9.2的支持。放弃对CUDA 9.1的支持。（1.0.0-beta2版本支持CUDA 8.0、9.0和9.2）
* Deeplearning4j: 带训练支持的新SameDiff层 - [Link](https://github.com/eclipse/deeplearning4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/samediff) [Link](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/samediff)
* Deeplearning4j 资源（数据集、预训练模型）存储目录现在可以通过 `DL4JResources.setBaseDirectory` 方法或 `org.deeplearning4j.resources.directory` 系统属性
* ND4J: 现在所有索引都是用long而不是int来完成的，以使数组的维数和长度大于整数（大约 21亿）。
* ND4J: 现在将使用Intel MKL-DNN作为默认/捆绑的BLAS实现（替换以前默认的 OpenBLAS）
* Deeplearning4j: 添加了内存溢出（OOM）崩溃转储报告功能。在训练 /推理OOM时提供内存使用和配置转储（以帮助调试和调整内存配置）。
* Deeplearning4j -新层：局部连接1d [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LocallyConnected1D.java), 局部连接2D [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LocallyConnected2D.java)

### [Deeplearning4J](fa-xing-shuo-ming.md)

#### Deeplearning4J: 新功能

*  添加了新的SameDiff层（自动区分-仅限单个类，需要前向传播定义）到DL4J，并提供了完整的训练支持-SameDiffLayer、SameDiffVertex、SameDiffOutputLayer、SameDiffLambdaLayer、SameDiffLambdaVertex-请注意，这些层目前仅限CPU执行 [Link](https://github.com/eclipse/deeplearning4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/samediff) [Link](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/samediff) [Link](https://github.com/eclipse/deeplearning4j/pull/5730)
* 资源（数据集、预训练模型）存储目录现在可以通过 `DL4JResources.setBaseDirectory` 方法或 `org.deeplearning4j.resources.directory` 系统属性。请注意，还可以为下载设置不同的基本位置（用于DL4J资源的本地镜像） [Link](https://github.com/eclipse/deeplearning4j/pull/5315)
*  添加了内存溢出（OOM）崩溃转储报告功能。如果训练/推理OOM，则提供内存使用和配置的转储。对于MultiLayerNetwork/ComputationGraph.memoryInfo方法，也提供了相同的信息（没有崩溃）。可以使用系统属性禁用（或输出目录设置） - [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/util/CrashReportingUtil.java)
* 添加了复合的\[Multi\]DataSetPreProcessor，以便在单个迭代器中应用多个\[Multi\]DataSetPreProcessor [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/dataset/api/preprocessor/CompositeDataSetPreProcessor.java)
* 多输出网络的附加计算图评估方法: `evaluate(DataSetIterator, Map<Integer,IEvaluation[]>)` 与  `evaluate(MultiDataSetIterator, Map<Integer,IEvaluation[]>)` [Link](https://github.com/eclipse/deeplearning4j/pull/5623)
* 添加了JointMultiDataSetIterator-用于从多个DataSetIterator创建多个MultiDataSetIterator的实用迭代器 [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/JointMultiDataSetIterator.java)
* GraphVertices现在可以直接使用可训练参数（不只是用可训练参数封闭层） [Link](https://github.com/eclipse/deeplearning4j/pull/5730)
* 添加 MultiLayerNetwork/ComputationGraph getLearningRate 方法 [Link](https://github.com/eclipse/deeplearning4j/pull/5766)
* 添加 RandomDataSetIterator 与 RandomMultiDataSetIterator \(主要用于测试/调试\) [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/RandomDataSetIterator.java) [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/RandomMultiDataSetIterator.java)
* 增加了周期性的“1周期”学习率调度等 - [Link](https://github.com/eclipse/deeplearning4j/pull/5844)
* Spark训练的RDD重新分区更具可配置性\(adds Repartitioner 接口\) [Link](https://github.com/eclipse/deeplearning4j/pull/5858)
* 添加 ComputationGraph.getIterationCount\(\) 与 .getEpochCount\(\) 为了与 MultiLayerNetwork保持一致  [Link](https://github.com/eclipse/deeplearning4j/pull/5880)
* 添加了局部连接的1d层 [Link](https://github.com/eclipse/deeplearning4j/pull/5891) [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LocallyConnected1D.java)
* Spark "data loader" API \(主要用于Spark\) [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-common/src/main/java/org/nd4j/api/loader/Loader.java) [Link](https://github.com/eclipse/deeplearning4j/search?q=DataLoader+in%3Apath&unscoped_q=DataLoader+in%3Apath) [Link](https://github.com/eclipse/deeplearning4j/blob/528443d6b83352b8cc07ce891a603da2540e58d8/deeplearning4j/deeplearning4j-scaleout/spark/dl4j-spark/src/main/java/org/deeplearning4j/spark/impl/graph/SparkComputationGraph.java#L241-L255)
* Spark 评估: 添加了允许指定评估worker数量（小于Spark线程数）的评估方法重载 [Link](https://github.com/eclipse/deeplearning4j/pull/5904)
* CnnSentenceDataSetIterator 现在有一个Format参数，支持RNNs和1D CNNs的输出数据 [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nlp-parent/deeplearning4j-nlp/src/main/java/org/deeplearning4j/iterator/CnnSentenceDataSetIterator.java#L68-L76)
* 添加 `ComputationGraph/MultiLayerNetwork.pretrain((Multi)DataSetIterator, int epochs)` 方法重载 [Link](https://github.com/eclipse/deeplearning4j/pull/5947)
* 多层网络和计算图现在有`output`方法重载，网络输出可以放在用户指定的工作区中，而不是分离  [链接](https://github.com/eclipse/deeplearning4j/issues/5932) [链接](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/graph/ComputationGraph.java#L1632-L1647)。这可以用来避免在释放本机内存之前创建需要垃圾收集的INDarray。
* EmbeddingSequenceLayer现在除了支持 `[minibatch,seqLength]` 格式数据外还支持 `[minibatch,1,seqLength]` 格式序列数据 [Link](https://github.com/eclipse/deeplearning4j/issues/5960)
* CuDNN 批归一化实现现在将用于rank 2输入，而不仅仅是rank 4输入 [Link](https://github.com/eclipse/deeplearning4j/pull/6011)
* DL4J的环境变量和系统属性已集中到DL4JResources和DL4JEnvironmentVars类中，并有适当的描述 [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-common/src/main/java/org/deeplearning4j/config/DL4JEnvironmentVars.java) [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-common/src/main/java/org/deeplearning4j/config/DL4JSystemProperties.java)
* MultiLayerNetwork 与 ComputationGraph output/feedForward/fit 方法现在是线程安全的，通过同步来实现。注意，由于性能的原因，不建议并发使用（相反：使用并行推理）；但是现在同步的方法应该避免由于并发修改而导致的模糊错误 [Link](https://github.com/eclipse/deeplearning4j/pull/6018)
* BarnesHutTSNE 现在在距离度量未定义的情况下抛出一个有用的异常（例如，所有零加上余弦相似性） [Link](https://github.com/eclipse/deeplearning4j/pull/6094)

#### Deeplearning4J: Bug 修复与优化

* 如果监听器已存在，则ComputationGraph.addListeners无法正常工作 [Link](https://github.com/eclipse/deeplearning4j/pull/5281), [Link](https://github.com/eclipse/deeplearning4j/issues/5251)
* TinyImageNetDataSetIterator 未验证/正确使用输入形状配置 [Link](https://github.com/eclipse/deeplearning4j/pull/5281), [Link](https://github.com/eclipse/deeplearning4j/issues/5234)
* BatchNormalization 层现在正确地断言，如果需要，将设置nOut（而不是以后出现不友好的形状错误） [Link](https://github.com/eclipse/deeplearning4j/pull/5302)
* 修正了OutputLayer不能正确初始化参数约束的问题 [Link](https://github.com/eclipse/deeplearning4j/pull/5306)
* 修正了Nesterov更新器在执行CUDA时只使用CPU操作的性能问题 [Link](https://github.com/eclipse/deeplearning4j/pull/5331)
* 删除了DL4J优化器的TerminationCondition-在实践中未使用，并且有小的开销 [Link](https://github.com/eclipse/deeplearning4j/pull/5340)
* 修复了启用工作区时EvaluativeListener可能遇到工作区验证异常的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/5351)
* 修复了计算图未正确调用TrainingListener.onEpochStart/onEpochEnd的问题  [Link](https://github.com/eclipse/deeplearning4j/pull/5414)
* 修复了TensorFlowCnnToFeedForwardPreProcessor工作区问题 [Link](https://github.com/eclipse/deeplearning4j/pull/5465)
* CuDNN批量归一化的性能优化 [Link](https://github.com/eclipse/deeplearning4j/pull/5483)
* 性能优化：在安全的情况下，Dropout将会被原地应用，避免复制。 [Link](https://github.com/eclipse/deeplearning4j/pull/5489)
* 增加了Dropout的CuDNN实现 [Link](https://github.com/eclipse/deeplearning4j/pull/5501)
* 减少CuDNN的内存使用：CuDNN工作内存现在在网络中的层之间共享和重用 [Link](https://github.com/eclipse/deeplearning4j/pull/5539)
* CuDNN批处理归一化实现将失败，数据类型为FP16 [Link](https://github.com/eclipse/deeplearning4j/pull/5554)
* 修复双向LSTM可能错误地使用导致异常的工作区问题 [Link](https://github.com/eclipse/deeplearning4j/issues/5472)
* 修复早停，分数最大化（准确率，F1等）没有正确触发终止条件的问题 [Link](https://github.com/eclipse/deeplearning4j/pull/5565)
* 修复了在ComputationGraph.computeGradientAndScore\(\)中标签掩码计数器可能错误递增的问题 [Link](https://github.com/eclipse/deeplearning4j/pull/5595)
* ComputationGraph 在训练期间没有设置lastEtlTime 字段  [Link](https://github.com/eclipse/deeplearning4j/issues/5614)
* 修复了启用工作区时自动编码器层的问题[Link](https://github.com/eclipse/deeplearning4j/pull/5663)
* 修复了EmbeddingSequenceLayer使用掩码数组的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/5778)
* Lombok现在在任何地方都提供了作用域，在使用DL4J时不在用户类路径上 [Link](https://github.com/eclipse/deeplearning4j/pull/5785)
* 修复了WordVectorSerializer.readParagraphVectors\(File\)初始化标签源的问题[Link](https://github.com/eclipse/deeplearning4j/issues/5806)
* Spark 训练（梯度共享）现在可以正确地处理训练期间遇到的空分区边界情况 [Link](https://github.com/eclipse/deeplearning4j/pull/5829)
* 对于Spark梯度共享训练，错误传播得更好/更一致 [Link](https://github.com/eclipse/deeplearning4j/pull/5879)
* 修复了1D CNN层的问题，该层具有遮罩阵列且stride&gt;1（遮罩未正确缩小） [Link](https://github.com/eclipse/deeplearning4j/pull/5880)
* DL4J批归一化实现仅在训练期间（CuDNN未受影响），在推理期间未正确添加epsilon值 [Link](https://github.com/eclipse/deeplearning4j/issues/5836)
* 当最大非填充值小于0时，具有最大池和ConvolutionMode.SAME的CUDNN子采样层可能把填充值（0）作为边界值的最大值。 [Link](https://github.com/eclipse/deeplearning4j/issues/5836)
* Spark梯度共享训练现在可以将监听器正确地传递给worker [Link](https://github.com/eclipse/deeplearning4j/pull/5947)
* 修复了用户界面和FileStatsStorage的罕见（非终端）并发修改问题 [Link](https://github.com/eclipse/deeplearning4j/issues/5519)
* CuDNN 卷积层现在支持dilation&gt;2（以前：使用DL4J conv层实现作为回调） [Link](https://github.com/eclipse/deeplearning4j/issues/4866)
* Yolo2OutputLayer 现在实现 computeScoreForExamples\(\) [Link](https://github.com/eclipse/deeplearning4j/issues/5056)
* SequenceRecordReeaderDataSetIterator 现在正确处理“没有标签”的情况[Link](https://github.com/eclipse/deeplearning4j/issues/5966)
* 修复了BarnesHutTSNE可能遇到工作区验证异常的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/5977)
* 在某些情况下，EMNIST迭代器在重置后可能会产生不正确的数据[Link](https://github.com/eclipse/deeplearning4j/pull/6061)

#### Deeplearning4J: API 变更 \(迁移指南\): 1.0.0-beta 到 1.0.0-beta2

* 由于缺少CuDNN支持，Graves LSTM已被弃用，取而代之的是LSTM，但其精度与实际情况类似。改用LSTM类。
* deeplearning4j-modelexport-solr: 现在使用Lucene/Solr版本7.4.0（以前是7.3.0） [Link](https://github.com/eclipse/deeplearning4j/pull/5744)
* CNN2d层的掩码数组必须是可广播的4d格式: `[minibatch,depth or 1, height or 1, width or 1]` - 以前是二维的形状 `[minibatch,height]` 或 `[minibatch,width]`。 这可以防止以后的情况（池化层）中的模糊性，并允许更复杂的掩码场景（例如在同一个小批量中为不同的图像大小进行掩码 ）。 [Link](https://github.com/eclipse/deeplearning4j/pull/5942)
* 一些旧的/不推荐使用的模型和层方法已被删除。（validateInput\(\), initParams\(\)）。因此，可能需要更新某些自定义层 [Link](https://github.com/eclipse/deeplearning4j/pull/5954)

#### Deelpearning4J: 1.0.0-beta2 已知问题

*  Windows用户无法加载SvhnLabelProvider（在HouseNumberDetection示例中使用）中使用的HDF5文件。Linux/Mac用户不受影响。windows用户的解决方法是添加sonatype快照依赖项`org.bytedeco.javacpp-presets:hdf5-platform:jar:1.10.2-1.4.3-SNAPSHOT` [Link](https://github.com/eclipse/deeplearning4j/issues/6017)

### [Deeplearing4J: Keras 导入 ](fa-xing-shuo-ming.md)

* Keras 模型导入现在导入每个Keras应用程序
* 支持GlobalPooling3D图层导入
* 支持RepeatVector层导入
* 支持LocallyConnected1D和LocallyConnected2D层
* 现在可以通过注册自定义SameDiff层导入Keras Lambda层
* 现在支持所有Keras优化器
* 现在可以导入所有高级激活功能。
* 许多小错误已经被修复，包括对所有的BatchNormalization配置进行适当的权重设置，改进SeparableConvolution2D的形状，以及完全支持双向层。

### [ND4J](fa-xing-shuo-ming.md)

#### ND4J: 新功能

* ND4J: all indexing is now done with longs instead of ints to allow for arrays with dimensions and lengths greater than Integer.MAX\_VALUE \(approx. 2.1 billion\)现在所有索引都是用long而不是int来完成的，以使数组的维数和长度大于整数（大约为 21亿）。
* 添加了使用Nd4j.writesNumpy\(INDArray,File\)编写Numpy.npy格式的功能，并使用Nd4j.convertToNumpy\(INDArray\)将INDArray转换为内存中严格的Numpy格式[Link](https://github.com/eclipse/deeplearning4j/pull/5973)
* ND4j-common ClassPathResource: 添加 ClassPathResource.copyDirectory\(File\) [Link](https://github.com/eclipse/deeplearning4j/issues/5298)
* SameDiff: 现有大量新操作和已存在操作的反向传播实现
* 添加  Nd4j.randomBernoulli/Binomial/Exponential 便捷方法 [Link](https://github.com/eclipse/deeplearning4j/blob/b887d2f0601fc6562a5a278e822690d2c338aaad/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/factory/Nd4j.java#L3150-L3223)
* 添加了禁用/禁止ND4J初始化日志记录的方法 通过 `org.nd4j.log.initialization` 系统属性  [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-common/src/main/java/org/nd4j/config/ND4JSystemProperties.java#L27-L33)
* SameDiff 类 - 大多数op/constructor方法现在都有完整/有用的javadoc [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff/SameDiff.java)
* 现在可以全局禁用工作区，忽略工作区配置。这主要用于调试；使用 `Nd4j.getWorkspaceManager().setDebugMode(DebugMode.DISABLED)` 或 `Nd4j.getWorkspaceManager().setDebugMode(DebugMode.SPILL_EVERYTHING);` 来启用它。 [Link](https://github.com/eclipse/deeplearning4j/issues/5980) \[Link\]
* 为环境变量处理添加了EnvironmentalAction API [Link](https://github.com/eclipse/deeplearning4j/pull/6003)
* ND4J 环境变量和系统属性集中在ND4jEnvironmentVars 与 ND4jSystemProperties 类  [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-common/src/main/java/org/nd4j/config/ND4JEnvironmentVars.java) 与 [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-common/src/main/java/org/nd4j/config/ND4JSystemProperties.java)

#### ND4J: Bug 修复与优化

* SameDiff: 执行和单个操作的大量错误修复
* 修复了使用真标量的INDArray.toDoubleArray\(\)的问题（rank 0数组）[Link](https://github.com/eclipse/deeplearning4j/issues/5362)
* 修复了DataSet.sample\(\)不适用于rank 3+功能的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/5477)
* IActivation 现在实现对激活和梯度验证/强制执行相同的形状[Link](https://github.com/eclipse/deeplearning4j/issues/5357)
* 修正了向量为1d的muliColumnVector问题 [Link](https://github.com/eclipse/deeplearning4j/issues/5530)
* ImagePreProcessingScaler现在支持通过NormalizerSerializerStrategy和ModelSerializer进行序列化 [Link](https://github.com/eclipse/deeplearning4j/pull/5694)
* DL4J Spark梯度共享分布式训练实现中阈值编码的性能优化 [Link](https://github.com/eclipse/deeplearning4j/pull/5767)
* SameDiff: DL4J Spark梯度共享分布式训练实现中阈值编码的性能优化 [Link](https://github.com/eclipse/deeplearning4j/issues/5934)
* DataSet.save\(\) 与 MultiDataSet.save\(\) 方法现在出现时保存示例元数据 [Link](https://github.com/eclipse/deeplearning4j/issues/4557)
* 修正了当数据集不等分为没有余数的包时KFoldIterator的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/5974)
* 修复了当资源位于带有空格的路径上时版本检查功能无法加载资源的问题 [Link](https://github.com/eclipse/deeplearning4j/issues/6056)

#### ND4J: 已知问题

#### ND4J: API 变更 \(迁移指南\): 1.0.0-beta 到 1.0.0-beta2

* CUDA 9.1支持已被移除。提供CUDA 8.0、9.0和9.2支持
* 由于长索引的变化，在某些地方应该使用 long/long\[\]代替int/int\[\]。\(如 INDArray.size\(int\), INDArray.shape\(\)\)
* 简化DataSetIterator API: totalExamples\(\), cursor\(\) 与 numExamples\(\) - 这些在大多数DataSetIterator实现中都是不受支持的，并且在实践中没有用于训练。自定义迭代器也应该删除这些方法 [Link](https://github.com/eclipse/deeplearning4j/pull/5560)
* 已删除长期不推荐使用的getFeatureMatrix\(\)。改用DataSet.getFeatures\(\)。 [Link](https://github.com/eclipse/deeplearning4j/pull/6006)
* 已删除未使用且未正确测试/维护的实用程序类BigDecimalMath。如果需要，用户应该为这个功能找到一个候选库。
* 未正确维护的复数支持类（IComplexNumber，IComplexNDArray）已完全删除 [Link](https://github.com/eclipse/deeplearning4j/pull/6031)

### [DataVec](fa-xing-shuo-ming.md)

#### DataVec:新功能

* 添加了AnalyzeLocal类以镜像AnalyzeSpark的功能（但没有Spark依赖项） [Link](https://github.com/eclipse/deeplearning4j/blob/master/datavec/datavec-local/src/main/java/org/datavec/local/transforms/AnalyzeLocal.java)
* 添加了JacksonLineSequenceRecordReader:RecordReader用于多示例JSON/XML，其中文件中的每一行都是一个独立的示例[Link](https://github.com/eclipse/deeplearning4j/blob/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/jackson/JacksonLineSequenceRecordReader.java)
* 添加 `RecordConvert.toRecord(Schema, List<Object>)` [Link](https://github.com/eclipse/deeplearning4j/pull/5849)
* 添加缺失的  FloatColumnCondition [Link](https://github.com/eclipse/deeplearning4j/pull/5933)
* 为“CSV中的每一行是一个序列，序列是单值/单变量”添加了CSVLineSequenceRecordReader [Link](https://github.com/eclipse/deeplearning4j/blob/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/csv/CSVLineSequenceRecordReader.java)
* 为“单个CSV中的多个多值序列”数据添加了CSVMultiSequenceRecordReader [Link](https://github.com/eclipse/deeplearning4j/blob/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/csv/CSVMultiSequenceRecordReader.java) 

#### DataVec: 优化与Bug修复

* 修复了Android上NativeImageLoader的问题 [Link](https://github.com/eclipse/deeplearning4j/pull/5468)
* 已修复ExcelRecordReader的问题 [Link](https://github.com/eclipse/deeplearning4j/pull/5758)
* 修复了 `CSVRecordReader.next(int)` 参数错误的问题，可能导致生成不必要的大列表 [Link](https://github.com/eclipse/deeplearning4j/pull/5963)

#### DataVec: API 变更 \(迁移指南\): 1.0.0-beta 到 1.0.0-beta2

### [Arbiter](fa-xing-shuo-ming.md)

#### Arbiter: 新功能

* 添加了DataSource接口。与旧的DataProvider不同，这不需要JSON序列化（只需要一个无参数构造函数） [Link](https://github.com/eclipse/deeplearning4j/pull/5952)
* 增加了许多增强和缺少的配置选项（约束、扩展等）[Link](https://github.com/eclipse/deeplearning4j/issues/6062) [Link](https://github.com/eclipse/deeplearning4j/pull/6089)

#### Arbiter: 修复

* DataProvider已被弃用。改用DataSource。

### [RL4J](fa-xing-shuo-ming.md)

* stepCounter, epochCounter 与 historyProcessor现在可以被设置 [Link](https://github.com/eclipse/deeplearning4j/pull/5972)
* 现在为加载ACPolicy加载随机种子 [Link](https://github.com/eclipse/deeplearning4j/issues/5543)

### [ScalNet](fa-xing-shuo-ming.md)

### [ND4S](fa-xing-shuo-ming.md)

## [1.0.0-beta版本发行说明](fa-xing-shuo-ming.md)

### 1.0.0-beta 发行亮点

* DL4J的性能和内存优化

### [Deeplearning4J](fa-xing-shuo-ming.md)

#### Deeplearning4J: 新功能

* 新层或增强层:
  * 添加 Cropping1D 层 [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/convolutional/Cropping1D.java)
  * 添加 Convolution3D, Cropping3D, UpSampling3D, ZeroPadding3D, Subsampling3D 层 \(都有Keras导入支持\): [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Convolution3D.java) [Link](https://github.com/eclipse/deeplearning4j/pull/5026)
  * 添加 EmbeddingSequenceLayer \(时间序列的EmbeddingLayer\) [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/EmbeddingSequenceLayer.java)
  * 添加 OCNNOutputLayer \(一分类神经网络\) - [这篇论文](https://arxiv.org/pdf/1802.06360.pdf)的实现 - [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/ocnn/OCNNOutputLayer.java)
  * 添加 FrozenLayerWithBackprop 层 [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/misc/FrozenLayerWithBackprop.java)
  * 添加 DepthwiseConvolution2D 层 [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/DepthwiseConvolution2D.java)
* 添加 ComputationGraph.output\(DataSetIterator\) 方法 [Link](https://github.com/eclipse/deeplearning4j/issues/4965)
* 添加 MultiLayerNetwork/ComputationGraph.layerInputSize 方法 [Link](https://github.com/eclipse/deeplearning4j/issues/4670) [Link](https://github.com/eclipse/deeplearning4j/pull/5018)
* 添加 SparkComputationGraph.feedForwardWithKey 具有特征掩码支持的重载 [Link](https://github.com/eclipse/deeplearning4j/issues/4984)
* 添加 MultiLayerNetwork.calculateGradients 方法（便于获取参数和输入梯度，例如一些模型可解释性方法） [Link](https://github.com/eclipse/deeplearning4j/pull/5018) [Link](https://github.com/eclipse/deeplearning4j/issues/2866)
* 添加了从配置获取每个层的输入/激活类型的支持：`ComputationGraphConfiguration.getLayerActivationTypes(InputType...)`, `ComputationGraphConfiguration.GraphBuilder.getLayerActivationTypes()`, `NeuralNetConfiguration.ListBuilder.getLayerActivationTypes()`, `MultiLayerConfiguration.getLayerActivationTypes(InputType)` methods [Link](https://github.com/eclipse/deeplearning4j/pull/5031)
* Evaluation.stats\(\) 现在以更易于读取的矩阵格式而不是列表格式打印混淆矩阵 [Link](https://github.com/eclipse/deeplearning4j/issues/5096)
*  添加ModelSerializer.addObjectToFile, .getObjectFromFile and .listObjectsInFile 用于将任意Java对象存储在与保存的网络相同的文件中 [Link](https://github.com/eclipse/deeplearning4j/issues/4957)
* 添加了SpatialDropout支持（具有Keras导入支持） [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/dropout/SpatialDropout.java)
* 添加`MultiLayerNetwork/ComputationGraph.fit((Multi)DataSetIterator, int numEpochs)` 重载 [Link](https://github.com/eclipse/deeplearning4j/pull/5118)
* 增加了性能（硬件）监听器: `SystemInfoPrintListener` 与 `SystemInfoFilePrintListener` [Link](https://github.com/eclipse/deeplearning4j/pull/5151)

#### Deeplearning4J: Bug修复与优化

* 通过优化工作区的内部使用来优化性能和内存 [Link](https://github.com/eclipse/deeplearning4j/pull/4900)
* 反射库已完全从DL4J中删除，不再需要自定义层序列化/反序列化 [Link](https://github.com/eclipse/deeplearning4j/pull/4950), [Link](https://github.com/eclipse/deeplearning4j/pull/4956)
  * 修复了Android上自定义和一些Keras导入层的问题
* RecordReaderMultiDataSetIterator 将不再尝试将未使用的列转换为数值 [Link](https://github.com/eclipse/deeplearning4j/pull/4945)
* 新增动物园模型:
  * \(将要做 \)
* Android编译修复（删除重复类、对齐版本、删除一些依赖项） [Link](https://github.com/eclipse/deeplearning4j/pull/4955) [Link](https://github.com/eclipse/deeplearning4j/pull/5074) [Link](https://github.com/eclipse/deeplearning4j/issues/5087)
* 修正了RecordReaderMulitDataSetIterator，其中某些构造函数的输出可能不正确 [Link](https://github.com/eclipse/deeplearning4j/issues/4969)
* 在返向传播期间，冻结层之前的非冻结层将不再被跳过（对于GANs和类似的架构很有用） [Link](https://github.com/eclipse/deeplearning4j/pull/5009) [Link](https://github.com/eclipse/deeplearning4j/issues/4964)
* 修复了计算图拓扑排序在所有平台上可能不一致的问题；有时可能中断在PC上训练并部署在Android上的计算图（具有多个有效拓扑排序） [Link](https://github.com/eclipse/deeplearning4j/pull/5050)
* 修复了CuDNN批归一化的问题，使用 `1-decay` 替代 `decay` [Link](https://github.com/eclipse/deeplearning4j/pull/5076)
* deeplearning4j-cuda 如果在将nd4j-native后端设置为更高优先级的类路径上出现异常，则不再引发异常 [Link](https://github.com/eclipse/deeplearning4j/issues/5000)
* 为CifarDataSetIterator添加了RNG控件 [Link](https://github.com/eclipse/deeplearning4j/issues/5067)
* WordVectorSerializer完成后立即删除临时文件 [Link](https://github.com/eclipse/deeplearning4j/pull/5166)

#### Deeplearning4J: API 更新\(迁移指南\): 1.0.0-alpha 到 1.0.0-beta

* WorkspaceMode.SINGLE 与 SEPARATE 已经被弃用; 使用 WorkspaceMode.ENABLED 代替
* 内部层API更改：自定义层将需要更新为新的层API-请参阅内置层或自定义层示例
* 由于JSON格式更改，需要先注册1.0.0-beta JSON（ModelSerializer）之前格式的自定义层等，然后才能对其进行反序列化。保存在1.0.0-beta或更高版本中的内置层和模型不需要这样做. 使用  `NeuralNetConfiguration.registerLegacyCustomClassesForJSON(Class)` 来实现这个目的
* IterationListener已被弃用，取而代之的是TrainingListener。对于现有自定义侦听器, 从 `implements TrainingListener` 切换到 `extends BaseTrainingListener` [Link](https://github.com/eclipse/deeplearning4j/pull/5014)
* ExistingDataSetIterator 已弃用；使用 `fit(DataSetIterator, int numEpochs)` method instead方法代替

#### Deelpearning4J: 1.0.0-beta 已知问题

* ComputationGraph TrainingListener onEpochStart 与 onEpochEnd 未正确调用方法
* DL4J Zoo Model FaceNetNN4Small2 未正确调用方法
* 早停计分计算器的值THAR应最大化（准确性，F1等）不正常工作（值被最小化，而不是最大化）。解决方法：重写`ScoreCalculator.calculateScore(...)` 并返回 `1.0 - super.calculateScore(...)`.

### [Deeplearing4J: Keras 导入](fa-xing-shuo-ming.md)

#### Deeplearning4J: Keras 导入 - API 变更 \(迁移指南\): 1.0.0-alpha 到 1.0.0-beta

### [ND4J](fa-xing-shuo-ming.md)

#### ND4J: 新功能

#### ND4J:已知问题

* 并非所有的操作梯度都实现了自动微分
* 在1.0.0-beta版本中添加的绝大多数新操作尚未使用GPU。

#### ND4J: API 变更 \(迁移指南\): 1.0.0-alpha 到 1.0.0-beta

### [DataVec](fa-xing-shuo-ming.md)

#### DataVec: 新功能

* ImageRecordReader 现在记录推理的标签类的数量（以减少用户在错误配置时丢失问题的风险） [Link](https://github.com/deeplearning4j/DataVec/issues/569)
* 为多个列添加了AnalyzeSpark.getUnique重载[Link](https://github.com/deeplearning4j/DataVec/issues/71)
* 增加了性能/计时模块 [Link](https://github.com/deeplearning4j/DataVec/pull/580)

#### DataVec: 优化与Bug 修复 

* 通过缓冲区重用减少ImageRecordReader垃圾生成 [Link](https://github.com/deeplearning4j/DataVec/pull/573)
* Android编译修复（对齐版本，删除了一些依赖项） [Link](https://github.com/deeplearning4j/DataVec/pull/567) [Link](https://github.com/deeplearning4j/DataVec/pull/575)
* 删除了DataVec中使用的反射库 [Link](https://github.com/deeplearning4j/DataVec/pull/570)
* TransformProcessRecordReader批处理支持修复 [Link](https://github.com/deeplearning4j/DataVec/issues/561)
* 带过滤器操作的TransformProcessRecordReader修复 [Link](https://github.com/deeplearning4j/DataVec/issues/552)
* 修复了ImageRecordReader/ParentPathLabelGenerator错误过滤包含 `.` character\(s\) 的目录[Link](https://github.com/deeplearning4j/DataVec/issues/273)
* ShowImageTransform现在延迟初始化帧以避免出现空白窗口 [Link](https://github.com/deeplearning4j/DataVec/pull/579)

#### DataVec: API 变更 \(迁移指南\): 1.0.0-alpha 到 1.0.0-beta

* DataVec ClassPathResource 已被弃用； 使用 nd4j-common 版本替代 [Link](https://github.com/deeplearning4j/DataVec/issues/521)

### [Arbiter](fa-xing-shuo-ming.md)

#### Arbiter: 新功能

* 为OCNN（一分类神经网络）添加LayerSpace

#### Arbiter: 修复

* 修复了可能导致在UI中不正确呈现第一个模型结果的时间戳问题 [Link](https://github.com/deeplearning4j/Arbiter/pull/163)
* 在遇到终止条件时返回之前,执行现在等待最后一个模型完成[Link](https://github.com/deeplearning4j/Arbiter/pull/162)
* 根据DL4J等：反射库的使用已完全从Arbiter中删除 [Link](https://github.com/deeplearning4j/Arbiter/pull/154)
* 由于Android编译的问题，不再使用Eclipse集合库 [Link](https://github.com/deeplearning4j/Arbiter/pull/156)
* 对已完成模型的改进清理以减少对训练的最大内存需求[Link](https://github.com/deeplearning4j/Arbiter/pull/160)

### [RL4J](fa-xing-shuo-ming.md)

### [ScalNet](fa-xing-shuo-ming.md)

### [ND4S](fa-xing-shuo-ming.md)

## [1.0.0-alpha发行说明](fa-xing-shuo-ming.md)

### 1.0.0-alpha发行亮点

* ND4J:添加了SameDiff-Java自动微分库（alpha版本）和Tensorflow导入（技术预览）以及数百个新操作
* ND4J: 增加了CUDA 9.0和9.1支持（使用cuDNN），放弃了对CUDA 7.5的支持，继续支持CUDA 8.0
* ND4J: 本机二进制文件（Maven Central上的nd4j-native）现在支持AVX/AVX2/AVX-512（Windows/Linux）
* DL4J: 大量新层和API改进
* DL4J: Keras 2.0 导入支持

### [Deeplearning4J](fa-xing-shuo-ming.md)

#### Deeplearning4J: 新功能

* Layers \(新的和增强的\)
  * 添加 Yolo2OutputLayer CNN 层用于目标检测 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/objdetect/Yolo2OutputLayer.java)\). 同样查看 DataVec的 [ObjectDetectionRecordReader](https://github.com/deeplearning4j/DataVec/blob/master/datavec-data/datavec-data-image/src/main/java/org/datavec/image/recordreader/objdetect/ObjectDetectionRecordReader.java)
  * 添加通过 `hasBias(boolean)` 实现对“无偏置”层的支持

    config \(DenseLayer, EmbeddingLayer, OutputLayer, RnnOutputLayer, CenterLossOutputLayer, ConvolutionLayer, Convolution1DLayer\). EmbeddingLayer 现在默认为无偏置 \([Link](https://github.com/eclipse/deeplearning4j/pull/3882)\)

  * 增加了对空洞卷积（也称为“膨胀”卷积）的支持- ConvolutionLayer, SubsamplingLayer, 和 1D 版本 。 \([Link](https://github.com/eclipse/deeplearning4j/pull/3922)\)
  * 添加了Upsampling2D 层, Upsampling1D 层 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Upsampling2D.java), [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Upsampling1D.java)\)
  * ElementWiseVertex 现在（另外） 支持 `Average` 和 `Max` 模式，除Add/Subtract/Product 外\([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/ElementWiseVertex.java)\)
  * 添加 SeparableConvolution2D 层 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/SeparableConvolution2D.java)\)
  * 添加 Deconvolution2D 层 \(也叫转置卷积，分步卷积层\) \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Deconvolution2D.java)\)
  * 添加ReverseTimeSeriesVertex \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/graph/rnn/ReverseTimeSeriesVertex.java)\)
  * 添加 RnnLossLayer - 无参数版本的RnnOutputLayer，或相当于RNN的LossLayer \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/RnnLossLayer.java)\)
  * 添加了CnnLossLayer-没有参数CNN输出层，用于分割、去噪等用例。 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/CnnLossLayer.java)\)
  * Added Bidirectional layer wrapper \(converts any uni-directional RNN to a bidirectional RNN\) 添加了双向层包装器（将任何单向RNN转换为双向RNN）\([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/recurrent/Bidirectional.java)\)
  * 添加SimpleRnn层\(也叫 "香草" RNN 层\) \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/recurrent/SimpleRnn.java)\)
  *  添加了LastTimeStep包装层（包装一个RNN层以获取上一个时间步，如果存在则考虑掩码）\([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/recurrent/LastTimeStep.java)\)
  * 添加了MaskLayer实用程序层，当存在一个掩码数组时，该实用程序层仅在正向传递时将激活归零 \([Link](https://github.com/eclipse/deeplearning4j/pull/4647)\)
  * 添加了alpha版本（还不稳定）SameDiff层对DL4J的支持（注意：向前传递，目前仅限于CPU）\([Link](https://github.com/eclipse/deeplearning4j/pull/4675)\)
  * 添加了SpaceToDepth和SpaceToBatch层 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/convolution/SpaceToDepth.java), [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/convolution/SpaceToBatch.java)\)
  * 添加Cropping2D 层 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/convolutional/Cropping2D.java)\)
* 添加了参数约束API（LayerConstraint接口）, 和  MaxNormConstraint, MinMaxNormConstraint, NonNegativeConstraint, UnitNormConstraint 实现 \([Link](https://github.com/eclipse/deeplearning4j/pull/3957)\)
* 学习率调度的显著重构 \([Link](https://github.com/eclipse/deeplearning4j/pull/3985)\)
  * 添加了ISchedule接口；添加了Exponential、Inverse、Map、Poly、Sigmoid和Step调度实现\([Link](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/schedule)\)
  * Added support for both iteration-based and epoch-based schedules via ISchedule. Also added support for custom \(user defined\) schedules通过ISchedule增加了对基于迭代和基于轮数的计划的支持。还添加了对自定义（用户定义）计划的支持
  * 在更新程序上配置学习率调度, 通过  `.updater(IUpdater)` 方法
* 添加了dropout API（IDropout-以前的dropout是可用的，但不是类）；添加了Dropout、AlphaDropout（用于自归一化网络）、GaussianDropout（乘法）、GaussianNoise（加法）。添加~~了~~对自定义dropout类型的支持 \([Link](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/dropout)\)
* 通过ISchedule接口增加了对辍学调度的支持 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/dropout/Dropout.java#L64)\)
* 增加了权重/参数噪声API（IWeightNoise接口）；增加了DropConnect和WeightNoise（加法/乘法高斯噪声）实现（[Link](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/weightnoise)）；DropConnect和dropout现在可以同时使用
* 添加层配置别名 `.units(int)` 相当于 `.nOut(int)` \([Link](https://github.com/eclipse/deeplearning4j/pull/3900)\)
* 添加 ComputationGraphConfiguration GraphBuilder `.layer(String, Layer, String...)` 的别名 `.addLayer(String, Layer, String...)`
* 不再需要层索引 MultiLayerConfiguration ListBuilder \(i.e., `.list().layer(<layer>)` 现在可以用于配置\) \([Link](https://github.com/eclipse/deeplearning4j/issues/3888)\)
* 添加 `MultiLayerNetwork.summary(InputType)` 和 `ComputationGraph.summary(InputType...)` 方法（显示层和激活大小信息） \([Link](https://github.com/eclipse/deeplearning4j/pull/3983)\)
* MultiLayerNetwork, ComputationGraph 分层可训练层现在可以跟踪轮数量 \([Link](https://github.com/eclipse/deeplearning4j/pull/3957)\)
* 添加deeplearning4j-ui-standalone 模块: uber-jar 方便用户界面服务器的启动 \(使用: `java -jar deeplearning4j-ui-standalone-1.0.0-alpha.jar -p 9124 -r true -f c:/UIStorage.bin`\)
* 权重初始化:
  * 添加 `.weightInit(Distribution)` 方便/重载（之前：必需`.weightInit(WeightInit.DISTRIBUTION).dist(Distribution)`\) \([Link](https://github.com/eclipse/deeplearning4j/commit/45cbb6efc2ad015397b4fdf5eac9d1e9dc70ac9c)\)
  * WeightInit.NORMAL \(用于自归一化神经网络\) \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/weights/WeightInit.java)\)
  * Ones, Identity 权重初始化 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/weights/WeightInit.java)\)
  * 添加新分布\(LogNormalDistribution, TruncatedNormalDistribution, OrthogonalDistribution, ConstantDistribution\) 可用于权重初始化 \([Link](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/distribution)\)
  * RNNs: 增加了将循环网络权重的权重初始化分别指定为“输入”权重的功能\([Link](https://github.com/eclipse/deeplearning4j/pull/4579)\)
* 添加的层别名: Convolution2D \(ConvolutionLayer\), Pooling1D \(Subsampling1DLayer\), Pooling2D \(SubsamplingLayer\) \([Link](https://github.com/eclipse/deeplearning4j/pull/4026)\)
* 增加了Spark IteratorUtils-包装了一个用于Spark网络训练的RecordReaderMultiDataSetIterator \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-scaleout/spark/dl4j-spark/src/main/java/org/deeplearning4j/spark/datavec/iterator/IteratorUtils.java)\)
* CuDNN支持层（卷积层等）现在警告用户如果使用没有CuDNN的CUDA \([Link](https://github.com/eclipse/deeplearning4j/pull/4039)\)
* 二进制交叉熵（LossBinaryXENT）现在实现了裁剪（默认情况下为1e-5到（1-1e-5））以避免数值下溢/NaNs \([Link](https://github.com/deeplearning4j/nd4j/pull/2121)\)
* SequenceRecordReaderDataSetIterator 现在支持多标签回归\([Link](https://github.com/eclipse/deeplearning4j/pull/4080)\)
* TransferLearning FineTuneConfiguration 现在有了设置训练/推理工作区模式的方法 \([Link](https://github.com/eclipse/deeplearning4j/pull/4090)\)
* IterationListener iterationDone 方法现在同时报告当前迭代和epoch计数；删除了不必要的调用/被调用方法 \([Link](https://github.com/eclipse/deeplearning4j/pull/4091)\)
* 添加 MultiLayerNetwork.layerSize\(int\), ComputationGraph.layerSize\(int\)/layerSize\(String\) 轻松确定层的大小 \([Link](https://github.com/eclipse/deeplearning4j/pull/4582)\)
* 添加 MultiLayerNetwork.toComputationGraph\(\) 方法 \([Link](https://github.com/eclipse/deeplearning4j/blob/988630b1c8cde8da6414ca80d146097c902993fb/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/multilayer/MultiLayerNetwork.java#L3391-L3398)\)
* 添加 NetworkUtils 易于更改已初始化网络的学习速率的方便方法 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/util/NetworkUtils.java)\)
* 添加 MultiLayerNetwork.save\(File\)/.load\(File\) 和 ComputationGraph.save\(File\)/.load\(File\) 方便方法 \([Link](https://github.com/eclipse/deeplearning4j/pull/4401)\)
* 添加 CheckpointListener 在训练期间定期保存模型副本（每N个iter/epoch，每T个时间单位） \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/CheckpointListener.java)\)
* 添加带掩码数组的ComputationGraph输出方法重载 \([Link](https://github.com/eclipse/deeplearning4j/pull/4553)\)
* 一种新的LossMultiLabel损失函数 \([Link](https://github.com/deeplearning4j/nd4j/pull/2724)\)
* 新增动物园模型：
  * Darknet19 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/Darknet19.java)\)
  * TinyYOLO \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/TinyYOLO.java)\)
* 新的迭代器和迭代器改进：
  * 添加FileDataSetIterator, FileMultiDataSetIterator 用于灵活地迭代保存的（多）数据集对象的目录 \([Link](https://github.com/eclipse/deeplearning4j/pull/4618)\)
  * UCISequenceDataSetIterator \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-datasets/src/main/java/org/deeplearning4j/datasets/iterator/impl/UciSequenceDataSetIterator.java)\)
  * RecordReaderDataSetIterator 现在有了方便的builder模式，改进了javadoc \([Link](https://github.com/eclipse/deeplearning4j/pull/4424)\)
  * 添加 DataSetIteratorSplitter, MultiDataSetIteratorSplitter \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/DataSetIteratorSplitter.java), [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-utility-iterators/src/main/java/org/deeplearning4j/datasets/iterator/MultiDataSetIteratorSplitter.java)\)
* 增加了用于早停的额外评分功能（ROC指标、全套评估/回归指标等） \([Link](https://github.com/eclipse/deeplearning4j/pull/4630)\)
* 为多层网络和计算图添加额外的ROC和ROCMultiClass评估重载 \([Link](https://github.com/eclipse/deeplearning4j/pull/4642)\)
* 阐明Evaluation.stats\(\)输出以引用“预测”而不是“示例”（前者更适合于RNN） \([Link](https://github.com/eclipse/deeplearning4j/issues/4674)\)
* EarlyStoppingConfiguration 现在支持 `Supplier<ScoreCalculator>` 用于非序列化分数计算器 \([Link](https://github.com/eclipse/deeplearning4j/pull/4694)\)
* 改进的通过错误方法加载模型时的ModelSerializer异常 \(例如 尝试通过restoreMultiLayerNetwork加载计算图\) \([Link](https://github.com/eclipse/deeplearning4j/issues/4487)\)
* 添加了SparkDataValidation实用程序方法来验证HDFS或本地上保存的DataSet和MultiDataSet \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-scaleout/spark/dl4j-spark/src/main/java/org/deeplearning4j/spark/util/data/SparkDataValidation.java)\)
* ModelSerializer: 添加 restoreMultiLayerNetworkAndNormalizer and restoreComputationGraphAndNormalizer 方法\([Link](https://github.com/eclipse/deeplearning4j/pull/4827)\)
* ParallelInference 现在有了支持输入掩码数组的输出重载 \([Link](https://github.com/eclipse/deeplearning4j/pull/4836)\)

#### Deeplearning4J: Bug 修复与优化

* Lombok不再作为传递依赖项包含 \([Link](https://github.com/eclipse/deeplearning4j/pull/4847)\)
* ComputationGraph 现在可以有一个顶点作为输出（不仅仅是层） \([Link](https://github.com/eclipse/deeplearning4j/issues/3858), [Link](https://github.com/eclipse/deeplearning4j/pull/3865)\)
* 具有大量历史记录的J7FileStatsStorage的性能改进 \([Link](https://github.com/eclipse/deeplearning4j/pull/3907)\)
* 修复变分自动编码器层的用户界面层大小 \([Link](https://github.com/eclipse/deeplearning4j/issues/3905)\)
* 修复以避免HDF5库崩溃 \([Link](https://github.com/eclipse/deeplearning4j/pull/4069), [Link](https://github.com/eclipse/deeplearning4j/pull/4870)\)
* UI Play servers 切换到生产（PROD）模式\([Link](https://github.com/eclipse/deeplearning4j/pull/4074)\)
* 与以上相关：用户现在可以设置 `play.crypto.secret` 系统属性手动设置播放应用程序密钥；默认情况下是随机生成的 \([Link](https://github.com/eclipse/deeplearning4j/pull/4289)\).
* SequenceRecordReaderDataSetIterator 会应用预处理器两次 \([Link](https://github.com/eclipse/deeplearning4j/issues/4103)\)
* Evaluation 在Spark上使用时，无参构造函数可能导致NaN评估度量
* CollectScoresIterationListener可能以无休止地重复 \([Link](https://github.com/eclipse/deeplearning4j/pull/4208)\)
* Async\(Multi\)DataSetIterator 调用 reset\(\) 在某些情况下，底层迭代器可能会导致问题\([Link](https://github.com/eclipse/deeplearning4j/pull/4239)\)
* 在某些情况下，L2正则化可能（错误地）应用于冻结层 \([Link](https://github.com/eclipse/deeplearning4j/pull/4260)\)
* NearestNeighboursServer的日志修复 \([Link](https://github.com/eclipse/deeplearning4j/pull/4272)\)
* BaseStatsListener内存优化 \([Link](https://github.com/eclipse/deeplearning4j/pull/4292)\)
* 用于从流加载Keras模型的ModelGuesser修复（以前可能会失败）\([Link](https://github.com/eclipse/deeplearning4j/issues/4309)\)
* 多层网络和计算图中工作区的各种问题修复 \([Link](https://github.com/eclipse/deeplearning4j/pull/4320), [Link](https://github.com/eclipse/deeplearning4j/issues/4291), [Link](https://github.com/eclipse/deeplearning4j/issues/4337), [Link](https://github.com/eclipse/deeplearning4j/pull/4349), [Link](https://github.com/eclipse/deeplearning4j/pull/4541), [Link](https://github.com/eclipse/deeplearning4j/pull/4671)\)
* 修复DuplicateTimeSeriesVertex中的错误条件 \([Link](https://github.com/eclipse/deeplearning4j/issues/4199)\)
* 修复某些有效计算图网络上的getMemoryReport异常 \([Link](https://github.com/eclipse/deeplearning4j/issues/4223)\)
* RecordReaderDataSetIterator 当与预处理器一起使用时，在某些情况下可能会导致异常 \([Link](https://github.com/eclipse/deeplearning4j/issues/4214)\)
* CnnToFeedForwardPreProcessor 只要输入数组长度与预期长度匹配，就可以悄悄地对无效输入变形 \([Link](https://github.com/eclipse/deeplearning4j/issues/4200)\)
* ModelSerializer如果JVM崩溃，将不会删除临时文件；现在不再需要时立即删除 \([Link](https://github.com/eclipse/deeplearning4j/issues/3855)\)
* RecordReaderMultiDataSetIterator 在某些情况下，当设置为“ALIGN\_END”模式时，不能添加掩码数组 \([Link](https://github.com/eclipse/deeplearning4j/issues/4238)\)
* ConvolutionIterationListener 以前在冻结所有卷积层时产生的IndexOutOfBoundsException\([Link](https://github.com/eclipse/deeplearning4j/issues/4313)\)
* PrecisionRecallCurve.getPointAtRecall  当多个点具有相同的召回率时，可以以正确但次优的精度返回一个点\([Link](https://github.com/eclipse/deeplearning4j/pull/4327)\)
* 如果dropout存在于现有层上，在迁移学习上设置dropout\(0\)，FineTuneConfiguration不会删除dropout。 \([Link](https://github.com/eclipse/deeplearning4j/issues/4368)\)
* 在某些罕见的情况下，Spark评估可能导致NullPointerException \([Link](https://github.com/eclipse/deeplearning4j/issues/3970)\)
* ComputationGraph: 配置验证中不总是检测到断开连接的顶点\([Link](https://github.com/eclipse/deeplearning4j/issues/2714)\)
* 激活层并不总是继承全局激活函数配置 \([Link](https://github.com/eclipse/deeplearning4j/issues/4094)\)
* RNN评估内存优化：当为训练配置TBPTT时，也使用TBPTT样式的拆分进行评估（结果相同，内存较少） \([Link](https://github.com/eclipse/deeplearning4j/pull/4405), [Link](https://github.com/eclipse/deeplearning4j/issues/3482)\)
* PerformanceListener 现在可序列化 \([Link](https://github.com/eclipse/deeplearning4j/pull/4423)\)
* ScoreIterationListener 与 PerformanceListener 现在报告模型迭代，而不是“自监听器创建以来的迭代” \([Link](https://github.com/eclipse/deeplearning4j/pull/4444)\)
* 合并ROC实例后，不能更新ROC类中的精度/回调曲线缓存值 \([Link](https://github.com/eclipse/deeplearning4j/issues/4442)\)  
* 在评估大量实例之后，ROC合并可能会产生IllegalStateException\([Link](https://github.com/eclipse/deeplearning4j/issues/4459)\)
* 向EmbeddingLayer添加了对无效输入索引的检查\([Link](https://github.com/eclipse/deeplearning4j/pull/4585)\)
* 修复了从JSON加载遗留（0.9.0以前版本）模型配置时可能出现的NPE问题 \([Link](https://github.com/eclipse/deeplearning4j/pull/4593)\)
* 修复了EvaluationCalibration HTML导出图表呈现的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/4589)\)
* 修复了与Spark训练一起使用时使用J7FileStatsStorage可能不正确呈现UI/StatsStorage图表的问题\([Link](https://github.com/eclipse/deeplearning4j/issues/4615)\)
* MnistDataSetIterator 不会总是可靠地检测并自动修复/重新下载损坏的下载数据 \([Link](https://github.com/eclipse/deeplearning4j/issues/4588)\)
* MnistDataSetIterator / EmnistDataSetIterator: 主机URL更改后更新的下载位置\([Link](https://github.com/eclipse/deeplearning4j/pull/4632), [Link](https://github.com/eclipse/deeplearning4j/pull/4637)\)
* 修复线程中断的传播 \([Link](https://github.com/eclipse/deeplearning4j/pull/4644)\)
* MultiLayerNetwork/ComputationGraph 如果网络不包含参数，则在初始化期间将不再引抛出ND4JIllegalStateException \([Link](https://github.com/eclipse/deeplearning4j/issues/4635), [Link](https://github.com/eclipse/deeplearning4j/pull/4664)\)
* 修复TSNE将数据发布到UI以进行可视化 \([Link](https://github.com/eclipse/deeplearning4j/pull/4667)\)
* PerformanceListener现在对无效的频率参数抛出一个有用的异常（在构造函数中），而不是运行时ArithmeticException \([Link](https://github.com/eclipse/deeplearning4j/pull/4679)\)
* RecordReader\(Multi\)DataSetIterator 现在，Writable值为非数值时抛出更有用的异常\([Link](https://github.com/eclipse/deeplearning4j/issues/4484)\)
* UI: 修复了从uber JARs读取国际化数据.txt文件时非英语语言可能出现的字符编码问题\([Link](https://github.com/eclipse/deeplearning4j/issues/4512)\)
* UI: 修复了加载I18N数据时试图错误解析非DL4J UI资源的问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/4497)\)
* 各种线程修复 \([Link](https://github.com/eclipse/deeplearning4j/pull/4794)\)
* Evaluation: 现在无参方法（f1\(\)，precion\(\)等）为二分类返回单个类值，而不是宏平均值；请澄清stats\(\)方法和javadoc中的值 \([Link](https://github.com/eclipse/deeplearning4j/pull/4802)\)
* 早停训练：未正确调用TrainingListener opEpochStart/End（等）方法 \([Link](https://github.com/eclipse/deeplearning4j/issues/4798)\)
* 修复了不总是将dropout应用于RNN层输入的问题\([Link](https://github.com/eclipse/deeplearning4j/pull/4823)\)
* ModelSerializer:改进了从无效/空/关闭流中读取数据时的验证/异常 \([Link](https://github.com/eclipse/deeplearning4j/pull/4827)\)
* ParallelInference 修复:
  * 修复使用批量模式时的可变大小输入（可变长度时间序列，可变大小CNN输入） \([Link](https://github.com/eclipse/deeplearning4j/pull/4836)\)
  * 修复了输出方法期间的底层模型异常现在正确地传播回用户\([Link](https://github.com/eclipse/deeplearning4j/pull/4836)\)
  * 修复了对“pre-batched”输入的支持（即，小批量大小大于1的输入） \([Link](https://github.com/eclipse/deeplearning4j/pull/4836)\)
* 基于就地随机操作的网络权值初始化内存优化 \([Link](https://github.com/eclipse/deeplearning4j/pull/4837)\)
* 对具有SAME模式填充的CuDNN的修复 \([Link](https://github.com/eclipse/deeplearning4j/pull/4864), [Link](https://github.com/eclipse/deeplearning4j/pull/4871)\)
* 修正了VariationalAutoencoder builder解码器层大小验证\([Link](https://github.com/eclipse/deeplearning4j/pull/4874)\)
* 改进的Kmeans吞吐量[link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nearestneighbors-parent/nearestneighbor-core/src/main/java/org/deeplearning4j/clustering/algorithm/BaseClusteringAlgorithm.java)
* 将RPForest添加到邻近值算法 [link](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nearestneighbors-parent/nearestneighbor-core/src/main/java/org/deeplearning4j/clustering/randomprojection)

#### Deeplearning4J: API 变更 \(迁移指南\): 0.9.1 到 1.0.0-alpha

* 对于多层网络和计算图，缺省训练工作空间模式已从NONE切换到SEPARATE。 \([Link](../pei-zhi/nei-cun-gong-zuo-jian.md)\)
* 行为变更: `fit(DataSetIterator)`类似的方法不再进行分层预训练，然后进行反向传播。用于预训练，使用 `pretrain(DataSetIterator)` 和 `pretrain(MultiDataSetIterator)` 方法 \([Link](https://github.com/eclipse/deeplearning4j/pull/4279)\)
* 以前不推荐的更新器配置方法 \(`.learningRate(double)`, `.momentum(double)` etc\) 全被删除
  * 配置学习率: 使用  `.updater(new Adam(lr))` 代替`.updater(Updater.ADAM).learningRate(lr)`
  * 配置偏置学习率: 使用 `.biasUpdater(IUpdater)` 方法
  * 配置学习率调度: 使用 use `.updater(new Adam(ISchedule))` 
* 通过枚举更新器配置\(i.e., `.updater(Updater)`\) 已弃用；使用 `.updater(IUpdater)`  
* `.regularization(boolean)` 已删除配置；功能现在总是等同于`.regularization(true)`
* `.useDropConnect(boolean)` 移除；使用 `.weightNoise(new DropConnect(double))` 代替
* `.iterations(int)` 方法已被删除（很少使用，用户感到困惑）
* 多个实用程序类 \(在 `org.deeplearning4j.util`\)  已弃用和/或移至 nd4j-common. 在nd4j common中使用相同的类名`org.nd4j.util` 代替。
* DL4J中的DataSetIterators已从deeplearning4j nn模块移动到新的deeplearning4j-datasets, deeplearning4j-datavec-iterators 和 deeplearning4j-utility-iterators。包/导入保持不变；deeplearning4j-core将它们作为可传递的依赖项引入，因此在大多数情况下不需要用户更改。 \([Link](https://github.com/eclipse/deeplearning4j/pull/4855)\)
* 以前已弃用 `.activation(String)` 已被移除；使用 `.activation(Activation)` 或 `.activation(IActivation)` 代替
* 层API更改：自定义层可能需要实现`applyConstraints(int iteration, int epoch)` 方法
*  参数初始值设定项API更改：自定义参数初始值设定项可能需要实现`isWeightParam(String)` 和 `isBiasParam(String)` 方法
* RBM （受限制的玻尔兹曼机器）层已完全移除。考虑使用VariationalAutoencoder作为替换 \([Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/variational/VariationalAutoencoder.java)\)
* GravesBidirectionalLSTM 已被弃用；使用 `new Bidirectional(Bidirectional.Mode.ADD, new GravesLSTM.Builder()....build()))` 代替
* 以前不推荐的WordVectorSerializer方法现在已被删除 \([Link](https://github.com/eclipse/deeplearning4j/issues/4359)\)
* 删除 deeplearning4j-ui-remote-iterationlisteners 模块并废弃 RemoteConvolutionalIterationListener \([Link](https://github.com/eclipse/deeplearning4j/pull/4772)\)

#### Deeplearning4J: 1.0.0-alpha 已知问题

* 与0.9.1（配置了工作区）相比，CUDA上某些网络类型的性能可能会降低。这将在下一个版本中解决
* 在对CUDA的FP16支持下发现了一些问题 \([Link](https://github.com/eclipse/deeplearning4j/issues/4897)\)

### [Deeplearing4J: Keras 导入](fa-xing-shuo-ming.md) 

* Keras 2支持，保持Keras 1的向后兼容性
* Keras 2和1导入使用完全相同的API，由DL4J推断
* Keras单元测试覆盖率增加了10倍，更多的真实集成测试
* 用于导入和检查层权重的单元测试
* Leaky ReLU, ELU, SELU 支持模型导入
* 所有Keras层都可以用可选的偏置项导入
* 删除旧的deeplearning4j keras模块，删除旧的“模型”API
* 所有Keras初始化 \(Lecun normal, Lecun uniform, ones, zeros, Orthogonal, VarianceScaling, Constant\) 被支持
* DL4J和Keras模型导入支持的1D卷积和池化 
* Keras模型导入支持的一维和二维空洞卷积层
* 支持1D 零填充层
* DL4J和模型导入完全支持Keras约束模块
* DL4J和Keras模型导入中的1D和2D层上采样（包括测试中的GAN示例）
* Keras模型导入支持的大多数合并模式，支持Keras 2合并层API
* DL4J与Keras模型导入支持可分离卷积二维层
* DL4J和Keras模型导入支持二维反卷积
* 导入时完全支持Keras噪声层（Alpha dropout、Gaussian dropout和noise）
* 在Keras模型导入中支持SimpleRNN层
* 支持双向层wrapper keras模型导入
* 在DL4J中添加LastTimestepVertex以支持Keras RNN层的return\_sequences=False。
* DL4J支持循环权重初始化和Keras导入集成。
* SpaceToBatch和BatchToSpace层在DL4J中提供更好的YOLO支持，加上端到端的YOLO Keras导入测试。
* DL4J和Keras模型导入中的Cropping2D支持

#### Deeplearning4J: Keras 导入 - API 变更  \(迁移指南\): 0.9.1 到 1.0.0-alpha

* 在0.9.1中已弃用 `Model` 和 `ModelConfiguration` 已被永久删除。使用 [KerasModelImport](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/KerasModelImport.java) 代替, 它现在是Keras模型导入的唯一入口点。

#### Deeplearning4J: Keras 导入 - 已知问题

* Embedding layer: 在DL4J中，嵌入层的输出默认为2D，除非指定了预处理器。在Keras中，输出总是3D的，但是根据指定的参数可以解释为2D。这通常会导致在导入嵌入层时出现困难。许多情况已得到处理，问题已得到解决，但仍有不一致之处。
* Batchnormalization layer: DL4J的批归一化层比Keras的版本要严格得多（一种好的方式）。例如，DL4J仅允许对4D卷积输入的空间维度进行归一化，而在Keras中，任何轴都可以用于归一化。根据维度顺序（NCHW对比NHWC）和Keras用户使用的特定配置，这可能会导致预期的\(!\) 以及意外的导入错误。
* 在DL4J中导入用于训练 目的的Keras模型的支持仍然非常有限\(enforceTrainingConfig == true\)，将在下一版本中正确处理。
* Keras Merge layers:Keras functional API似乎很好用，但是在Sequential模型中使用时会出现问题。
* Reshape layers: 在导入时可能有点不可靠。DL4J很少需要在标准输入预处理器之外显式地重塑输入。在路缘石中，经常使用重塑层。在边缘情况下，映射这两种范式可能很困难。

### [ND4J](fa-xing-shuo-ming.md)

#### ND4J:新功能

* 新增数百项新操作
* 具有自动微分功能的新DifferentialFunctionapi（参见SameDiff一节） [Link](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff)
* 新增tensorflow导入技术预览（支持1.4.0及以上版本）
* 增加了Apache Arrow序列化，支持新的tensor API [Link](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-serde/nd4j-arrow)
* 增加对AVX/AVX2和AVX-512指令集的支持，适用于Windows/Linux的nd4j-native [Link](http://repo1.maven.org/maven2/org/nd4j/nd4j-native/1.0.0-alpha/)
* Nvidia CUDA 8/9.0/9.1 现在被支持
* 引入了工作区改进以确保安全：默认情况下启用SCOPE\_PANIC profiling模式
* INDArray serde的FlatBuffers支持
* 添加了对自动广播操作的支持
* libnd4j, underlying c++ library, 功能增强，现在提供: NDArray 类, Graph 类, 并可以用作独立库或可执行文件。
* 卷积相关操作现在除了支持NCHW数据格式外，还支持NHWC。
* 累积操作现在可以选择保持缩小的维度。

#### ND4J: 已知问题

* 并非所有的操作梯度都实现了自动微分
* 在1.0.0-alpha中添加的绝大多数新操作尚未使用GPU。

#### ND4J: API 变更 \(迁移指南\): 0.9.1 到  1.0.0-alpha

### [ND4J - SameDiff](fa-xing-shuo-ming.md)

* 初始技术预览 [Link](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff)
* IF和WHILE原生支持控制流

用于ND4J的[SameDiff](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff)自动微分引擎的Alpha发布。

#### 功能

* 有两种可用的执行模式：Java驱动的执行和序列化图的本机执行。
* SameDiff图可以使用FlatBuffers序列化
* 从SameDiff操作生成并运行计算图。
* 图可以对输入数据运行正向传播，并计算反向传播的梯度。
* 已经支持许多高级层，如密集连层、卷积（1D-3D）反卷积、可分离卷积、池和上采样、批处理归一化、局部响应归一化、LSTM和GRU。
* 总共有大约350个SameDiff操作可用，包括许多用于构建复杂图的基本操作。
* 支持[TensorFlow](https://github.com/deeplearning4j/nd4j/tree/d4a15e394ef81592237677aee932eb734d64f5a7/nd4j-backends/nd4j-tests/src/test/java/org/nd4j/imports)和ONNX图的基本导入以进行推理。
* [TFOpTests](https://github.com/kgnandu/TFOpTests) 是用于为TensorFlow导入创建测试资源的专用项目。

已知问题和限制

* 在1.0.0-alpha中添加的绝大多数新操作尚未使用GPU。
* 虽然许多在实践中广泛使用的基本操作和高级层都得到了支持，但是操作的覆盖范围仍然有限。目标是实现与TensorFlow的特征等价，并完全支持TF图的导入。
* 一些现有的操作没有实现反向传播。 \(在SameDiff中叫作 `doDiff` \).

### [DataVec](fa-xing-shuo-ming.md)

#### DataVec: 新功能

* 添加了ObjectDetectionRecordReader-用于DL4J的Yolo2OutputLayer（[Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-data/datavec-data-image/src/main/java/org/datavec/image/recordreader/objdetect/ObjectDetectionRecordReader.java)）（还支持图像转换：[Link](https://github.com/deeplearning4j/DataVec/pull/432)）
* 添加了ImageObjectLabelProvider、VocLabelProvider和SvhnLabelProvider（Streetview门牌号），用于ObjectionDetectionRecordReader \([Link](https://github.com/deeplearning4j/DataVec/pull/411), [Link](https://github.com/deeplearning4j/DataVec/pull/449)\)
* 为单机执行添加了LocalTransformExecutor（没有Spark依赖项） \([Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-local/src/main/java/org/datavec/local/transforms/LocalTransformExecutor.java)\)
* 添加了ArrowRecordReader（用于读取ApacheArrow格式的数据） \([Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-arrow/src/main/java/org/datavec/arrow/recordreader/ArrowRecordReader.java)\)
* 添加了用于在RecordReader和RecordWriter之间转换的RecordMapper类\([Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-api/src/main/java/org/datavec/api/records/mapper/RecordMapper.java)\)
* RecordWriter和InputSplit API已经被改进；更灵活和支持跨所有writer的分区。 \([Link](https://github.com/deeplearning4j/DataVec/pull/528), [Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-api/src/main/java/org/datavec/api/split/partition/Partitioner.java), [Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-api/src/main/java/org/datavec/api/split/partition/NumberOfRecordsPartitioner.java)\)
* 添加了ArrowWritableRecordBatch和NDArrayRecordBatch以实现高效的批存储 \(`List<List<Writable>>`\) \([Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-arrow/src/main/java/org/datavec/arrow/recordreader/ArrowWritableRecordBatch.java), [Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-api/src/main/java/org/datavec/api/writable/batch/NDArrayRecordBatch.java)\)
* 添加了BoxImageTransform-一种不改变纵横比而裁剪或填充的ImageTransform \([Link](https://github.com/deeplearning4j/DataVec/pull/453)\)
* TransformProcess 现在有 `executeToSequence(List<Writable))`, `executeSequenceToSingle(List<List<Writable>>)` 与 `executeToSequenceBatch(List<List<Writable>>)` 方法 \([Link](https://github.com/deeplearning4j/DataVec/pull/392), [Link](https://github.com/deeplearning4j/DataVec/pull/395)\)
* 添加 CSVVariableSlidingWindowRecordReader \([Link](https://github.com/deeplearning4j/DataVec/pull/398)\)
* ImageRecordReader: 支持标签的回归用例（以前：仅分类）\([Link](https://github.com/deeplearning4j/DataVec/pull/423)\)
* ImageRecordReader: 支持多类多标签图像分类\(通过 PathMultiLabelGenerator 接口\) \([Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-api/src/main/java/org/datavec/api/io/labels/PathMultiLabelGenerator.java), [Link](https://github.com/deeplearning4j/DataVec/pull/520)\)
* DataAnalysis/AnalyzeSpark 现在包括分位数（通过t-digest）\([Link](https://github.com/deeplearning4j/DataVec/pull/436)\)
* 添加AndroidNativeImageLoader.asBitmap\(\), Java2DNativeImageLoader.asBufferedImage\(\) \([Link](https://github.com/deeplearning4j/DataVec/pull/448)\)
* 添加新的RecordReader/SequenceRecordReader实现：
  * datavec-excel 模块与ExcelRecordReader \([Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-excel/src/main/java/org/datavec/poi/excel/ExcelRecordReader.java)\)
  * JacksonLineRecordReader \([Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-api/src/main/java/org/datavec/api/records/reader/impl/jackson/JacksonLineRecordReader.java)\)
  * ConcatenatingRecordReader \([Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-api/src/main/java/org/datavec/api/records/reader/impl/ConcatenatingRecordReader.java)\)
* 添加新转换：
  * TextToTermIndexSequenceTransform \([Link](https://github.com/deeplearning4j/DataVec/pull/408)\)
  * ConditionalReplaceValueTransformWithDefault \([Link](https://github.com/deeplearning4j/DataVec/pull/409)\)
  * GeographicMidpointReduction \([Link](https://github.com/deeplearning4j/DataVec/pull/446)\)
* StringToTimeTransform 如果没有提供格式，将尝试猜测时间格式 \([Link](https://github.com/deeplearning4j/DataVec/pull/498)\)
* 改进了Android上NativeImageLoader的性能 \([Link](https://github.com/deeplearning4j/DataVec/pull/507)\)
* 添加 BytesWritable \(Writable for byte\[\] data\) \([Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-api/src/main/java/org/datavec/api/writable/BytesWritable.java)\)
* 添加 TranformProcess.inferCategories 从RecordReader自动推断类别的方法 \([Link](https://github.com/deeplearning4j/DataVec/blob/1d6ed6f290b7ee78d66ef8599bdeb18d37c47876/datavec-api/src/main/java/org/datavec/api/transform/TransformProcess.java#L490-L568)\)

#### DataVec: 修复

* Lombok不再作为传递依赖项包含 \([Link](https://github.com/deeplearning4j/DataVec/pull/538)\)
* MapFileRecordReader 和 MapFileSequenceRecordReader可以处理多部分映射文件的空分区/拆分 \([Link](https://github.com/deeplearning4j/DataVec/pull/393)\)
* CSVRecordReader 现在可以使用Java序列化正确序列化 \([Link](https://github.com/deeplearning4j/DataVec/pull/474)\) 和 Kryo 序列化 \([Link](https://github.com/deeplearning4j/DataVec/pull/476)\)
* Writables: 相等语义已更改：例如现在 DoubleWritable\(1.0\) 等于于 IntWritable\(1\) \([Link](https://github.com/deeplearning4j/DataVec/pull/410)\)
* NumberedFileInputSplit 现在支持前导零 \([Link](https://github.com/deeplearning4j/DataVec/pull/420)\)
* CSVSparkTransformServer 与  ImageSparkTransformServer 播放服务器改为生产模式\([Link](https://github.com/deeplearning4j/DataVec/pull/430)\)
* 修复FloatMetaData的JSON子类型信息 \([Link](https://github.com/deeplearning4j/DataVec/pull/452)\)
*  JacksonRecordReader, RegexSequenceRecordReadert序列化修复 \([Link](https://github.com/deeplearning4j/DataVec/pull/463)\)
* 添加RecordReader.resetSupported\(\) 方法 \([Link](https://github.com/deeplearning4j/DataVec/pull/469)\)
* SVMLightRecordReader 现在实现 nextRecord\(\) 方法 \([Link](https://github.com/deeplearning4j/DataVec/pull/485)\)
* 使用条件时自定义缩减的修复 \([Link](https://github.com/deeplearning4j/DataVec/pull/487)\)
* SequenceLengthAnalysis 现在是可序列化的 \([Link](https://github.com/deeplearning4j/DataVec/pull/495)\) 以及对序列化/反序列化JSON的支持 \([Link](https://github.com/deeplearning4j/DataVec/pull/504)\)
* FFT功能的修复 \([Link](https://github.com/deeplearning4j/DataVec/pull/505), [Link](https://github.com/deeplearning4j/DataVec/pull/508)\)
* 不再使用后端口java.util.functions；改为使用ND4J函数API \([Link](https://github.com/deeplearning4j/DataVec/pull/506)\)
* 修正时间列的转换数据质量分析 \([Link](https://github.com/deeplearning4j/DataVec/pull/510)\)

#### DataVec: API 变更 \(迁移指南\): 0.9.1 迁移到 1.0.0-alpha

* 许多util类 \(主要在 `org.datavec.api.util` 包\) 已弃用或删除；在nd4j-common模块中使用等效名称的util类 \([Link](https://github.com/deeplearning4j/DataVec/pull/514)\)
* RecordReader.next\(int\) 方法现在返回 `List<List<Writable>>` 用于批量, 不是 `List<Writable>`。同样查看 [NDArrayRecordBatch](https://github.com/deeplearning4j/DataVec/blob/master/datavec-api/src/main/java/org/datavec/api/writable/batch/NDArrayRecordBatch.java)
* RecordWriter 和 SequenceRecordWriter APIs 已使用多个新方法更新

### [Arbiter](fa-xing-shuo-ming.md)

#### Arbiter: 新功能

* 已添加工作区支持 \([Link](https://github.com/deeplearning4j/Arbiter/issues/110), [Link](https://github.com/deeplearning4j/Arbiter/pull/113)\)
* 添加了新图层空间: LSTM, CenterLoss, Deconvolution2D, LossLayer, 双向层包装器 \([Link](https://github.com/deeplearning4j/Arbiter/pull/113), [Link](https://github.com/deeplearning4j/Arbiter/pull/132)\)
* 根据DL4J API更改: 更新器配置选项（learning rate, momentum, epsilon, rho等）已移到参数空间。 引入更新空间（AdamSpace、AdaGradSpace等） \([Link](https://github.com/deeplearning4j/Arbiter/pull/103)\)
* 根据DL4J API的变化：Dropout配置现在是通过 `ParameterSpace<IDropout>`, DropoutSpace 引入 \([Link](https://github.com/deeplearning4j/Arbiter/pull/103)\)
* RBM 层空间被删除 \([Link](https://github.com/deeplearning4j/Arbiter/pull/113)\)
* ComputationGraphSpace: 添加 layer/vertex 带有预处理器重载的方法 \([Link](https://github.com/deeplearning4j/Arbiter/issues/109)\)
* 添加了直接使用DL4J层指定“固定”层的支持（而不是使用layerspace，即使对于没有超参数的层也是如此） \([Link](https://github.com/deeplearning4j/Arbiter/pull/128)\)
* 添加 LogUniformDistribution \([Link](https://github.com/deeplearning4j/Arbiter/blob/master/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/distribution/LogUniformDistribution.java)\)
* 分数函数的改进；增加了ROC分数函数 \([Link](https://github.com/deeplearning4j/Arbiter/pull/128)\)
* 增加学习率调度支持 \([Link](https://github.com/deeplearning4j/Arbiter/pull/131)\)
* 为 `ParameterSpace<Double>` 和 `ParameterSpace<Integer>` 添加数学操作 \([Link](https://github.com/deeplearning4j/Arbiter/pull/142)\)

#### Arbiter: 修复

* Fix parallel job execution \(when using multiple execution threads\)修复并行作业执行（使用多个执行线程时） \([Link](https://github.com/deeplearning4j/Arbiter/issues/135), [Link](https://github.com/deeplearning4j/Arbiter/pull/137)\)
* 改进了失败任务执行的日志记录 \([Link](https://github.com/deeplearning4j/Arbiter/pull/121)\)
* 修复UI JSON序列化 \([Link](https://github.com/deeplearning4j/Arbiter/pull/128)\)
* 修复在CUDA和多个执行线程上运行时的线程问题 \([Link](https://github.com/deeplearning4j/Arbiter/pull/138), [Link](https://github.com/eclipse/deeplearning4j/issues/4659), [Link](https://github.com/deeplearning4j/Arbiter/pull/140)\)
* 将保存的模型文件重命名为model.bin \([Link](https://github.com/deeplearning4j/Arbiter/pull/144)\)
* 修复使用非线程安全候选/参数空间线程问题 \([Link](https://github.com/deeplearning4j/Arbiter/pull/147)\)
* Lombok不再作为传递依赖项包含 \([Link](https://github.com/deeplearning4j/Arbiter/pull/148)\)

#### Arbiter: API 变更 \(迁移指南\): 0.9.1 到 1.0.0-alpha

* 根据DL4J更新器API的更改：旧的更新器配置（learningRate、momentum等）方法已被删除。使用 `.updater(IUpdater)` 或  `.updater(ParameterSpace<IUpdater>)` 方法代替

### [RL4J](fa-xing-shuo-ming.md)

* 为A3C增加对LSTM层的支持
* 修复A3C，使其使用新的 `ActorCriticLoss` 正确使用随机性
* 修复`QLearning`失败的情况（非平面输入、不完整序列化、不正确的归一化）
* 用异步算法修`HistoryProcessor`的逻辑和图像预处理失败
* 整理并更正统计数据的输出，还允许使用`IterationListener`
* 修复妨碍CUDA高效执行的问题
* 使用`NeuralNet.getNeuralNetworks()`, `Policy.getNeuralNet()`和方便的`Policy`构造函数提供对更多内部结构的访问
* Add MDPs for ALE \(Arcade Learning Environment\) and MALMO to support Atari games and Minecraft为ALE（街机学习环境）和MALMO添加mdp以支持Atari游戏和Minecraft
* 为Doom更新MDP以允许使用最新版本的VizDoom

### [ScalNet](fa-xing-shuo-ming.md)

* First release of [ScalNet Scala API](https://github.com/deeplearning4j/scalnet), which closely resembles Keras' API.
* 可以用sbt和maven构建。
* 支持与DL4J的`MultiLayerNetwork`相对应的Keras启发的 [Sequential](https://github.com/deeplearning4j/ScalNet/blob/master/src/main/scala/org/deeplearning4j/scalnet/models/Sequential.scala)模型和与`ComputationGraph`相对应的 [Model](https://github.com/deeplearning4j/ScalNet/blob/master/src/main/scala/org/deeplearning4j/scalnet/models/Model.scala)
* 项目结构与DL4J模型导入模块和Keras紧密结合。
* 支持以下层: Convolution2D, Dense, EmbeddingLayer, AvgPooling2D, MaxPooling2D, GravesLSTM, LSTM, 双向层包装器, Flatten, Reshape。此外, DL4J OutputLayers 被支持。

### [ND4S](fa-xing-shuo-ming.md)

* Scala 2.12 支持

## [0.9.1版本发行说明](fa-xing-shuo-ming.md)

**Deeplearning4J**

* 修复了0.9.0中版本依赖项不正确的问题
* 添加 EmnistDataSetIterator [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-data/deeplearning4j-datasets/src/main/java/org/deeplearning4j/datasets/iterator/impl/EmnistDataSetIterator.java#L32)
* 使用softmax对LossMCXENT / LossNegativeLogLikelihood的数值稳定性改进（应使用非常大的激活减少NaNs）

**ND4J**

* 为ND4J, DL4J, RL4J, Arbiter, DataVec添加运行时版本检查 [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/versioncheck/VersionCheck.java)

**已知问题**

* Deeplearning4j：使用Evaluation类无参构造函数（即new Evaluation\(\)）可能导致accuracy/stats报告为0.0。其他评估类构造函数和ComputationGraph/MultiLayerNetwork.evaluate\(DataSetIterator\)方法按预期工作。
  * 这也会影响Spark（分布式）评估：解决方法是`sparkNet.evaluate(testData);` 和 `sparkNet.doEvaluation(testData, 64, new Evaluation(10))[0];`, 其中10是要使用的类数，64是要使用的评估小批量大小。
* SequenceRecordReaderDataSetIterator对每个数据集应用两次预处理器（例如归一化）（可能的解决方法：使用RecordReaderMultiDataSetIterator + MultiDataSetWrapperIterator）
* 迁移学习：计算图可能错误地将l1/l2正则化（在FinetuneConfiguration中定义）应用于冻结层。解决方法：在FineTuneConfiguration上设置0.0 l1/l2，在新的/非冻结层上直接设置所需的l1/l2。请注意，使用迁移学习的多层网络似乎不受影响。

## [0.9.0版本发行说明](fa-xing-shuo-ming.md)

**Deeplearning4J**

* 添加了工作区功能（更快的训练性能+更少的内存） [Link](https://deeplearning4j.org/workspaces)
* 为Spark网络训练添加了SharedTrainingMaster（提高了性能） [Link 1](../fen-bu-shi-shen-du-xue-xi/jie-shao-yu-ru-men.md), [Link 2](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-spark-examples/dl4j-spark/src/main/java/org/deeplearning4j/legacyExamples/mlp/MnistMLPDistributedExample.java)
* ParallelInference被添加-使用内部批处理和队列请求的服务器推理包装器  [Link](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/inference/ParallelInferenceExample.java)
* 除了现有的参数平均模式之外，ParallelWrapper现在还可以与梯度共享一起工作 [Link](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-cuda-specific-examples/src/main/java/org/deeplearning4j/examples/multigpu/GradientsSharingLenetMnistExample.java)
* VPTree 性能显著提高
* 增加了CacheMode网络配置选项-以牺牲额外的内存使用为代价提高了CNN和LSTM的性能 [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/CacheMode.java)
* 添加LSTM层，用CUDNN支持[链接](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LSTM.java)（注意到现有的GraveLSTM实现不支持CUDNN）
* 带有预训练ImageNet、MNIST和VGG-Face权重的新型本地模型动物园 [Link](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo)
* 卷积性能改进，包括激活缓存
* 现在支持自定义/用户定义的更新器  [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/learning/config/IUpdater.java)
* 评估改进
  * EvaluationBinary, ROCBinary 类被添加 : 二进制多类网络的评估\(sigmoid + xent 输出层 \) [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/EvaluationBinary.java)
  * Evaluation和其他工具现在有G-Measure和Matthews相关系数支持；还有对Evaluation类度量的宏+微观平均支持 [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/EvaluationAveraging.java)
  * ComputationGraph 与 SparkComputationGraph 增加评估便利方法 \(evaluateROC, etc\)
  * ROC和ROCMultiClass支持精确计算（以前：使用阈值计算） [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/ROC.java#L78-L81)
  * ROC类现在支持precision recall曲线计算下的区域；在指定的阈值处获取precision/recall/confusion矩阵（通过PrecisionRecallCurve类）[Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/curves/PrecisionRecallCurve.java#L102-L193)
  * RegressionEvaluation、ROCBinary等现在支持每个输出掩码（除了每个示例/每个时间步掩码）
  * 增加评估校准（残差图、可靠性图、概率直方图） [Link 1](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/EvaluationCalibration.java) [Link 2](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-core/src/main/java/org/deeplearning4j/evaluation/EvaluationTools.java#L180-L188)
  * Evaluation 和 EvaluationBinary: 现在支持自定义分类阈值或成本数组 [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/Evaluation.java#L150-L170)
* 优化：更新器，偏置计算
* 增加了网络内存估计功能。可以从配置中估计内存需求，而无需实例化网络 [Link 1](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/MultiLayerConfiguration.java#L310-L317) [Link 2](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/ComputationGraphConfiguration.java#L451-L458)
* 新损失函数:
  * Mixture density 损失函数 [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/lossfunctions/impl/LossMixtureDensity.java)
  * F-Measure 损失函数 [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/lossfunctions/impl/LossFMeasure.java)

**ND4J**

* 添加了工作区功能 [Link](https://github.com/eclipse/deeplearning4j-examples/blob/master/nd4j-examples/src/main/java/org/nd4j/examples/Nd4jEx15_Workspaces.java)
* 添加了原生并行排序
* 新操作被添加: SELU/SELUDerivative, TAD-based comparisons, percentile/median, Reverse, Tan/TanDerivative, SinH, CosH, Entropy, ShannonEntropy, LogEntropy, AbsoluteMin/AbsoluteMax/AbsoluteSum, Atan2
* 添加了新的距离函数: CosineDistance, HammingDistance, JaccardDistance

**DataVec**

* 添加 MapFileRecordReader 和 MapFileSequenceRecordReader  [Link 1](https://github.com/deeplearning4j/DataVec/blob/master/datavec-hadoop/src/main/java/org/datavec/hadoop/records/reader/mapfile/MapFileRecordReader.java) [Link 2](https://github.com/deeplearning4j/DataVec/blob/master/datavec-hadoop/src/main/java/org/datavec/hadoop/records/reader/mapfile/MapFileSequenceRecordReader.java)
* Spark：将`JavaRDD<List<Writable>>`和`JavaRDD<List<List<Writable>>`数据保存和加载到Hadoop MapFile和SequenceFile格式的实用程序 [Link](https://github.com/deeplearning4j/DataVec/blob/master/datavec-spark/src/main/java/org/datavec/spark/storage/SparkStorageUtils.java)
* TransformProcess和Transforms现在支持NDArrayWritables和NDArrayWritable列
* 多个新转换类

**Arbiter**

* Arbiter UI: [Link](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/arbiter/BasicHyperparameterOptimizationExample.java)
  * UI现在使用Play 框架，与DL4J UI（取代Dropwizard后端）集成。已修复依赖关系问题/冲突版本。
  * 支持DL4J StatsStorage和StatsStorageRouter机制（FileStatsStorage、通过RemoveUIStatsStorageRouter实现的远程用户界面）
  * 常规UI改进（附加信息、格式修复）

#### 0.8.0 -&gt; 0.9.0 迁称说明

**Deeplearning4j**

* 更新器配置方法，如 .momentum\(double\) 和 .epsilon\(double\) 已弃用。替换: 使用 `.updater(new Nesterovs(0.9))` 和 `.updater(Adam.builder().beta1(0.9).beta2(0.999).build())` 等来配置

**DataVec**

* CsvRecordReader构造函数：现在使用字符作为分隔符，而不是字符串（即，'，'而不是“，”）

**Arbiter**

* Arbiter UI现在是一个单独的模块，带有Scala版本后缀： `arbiter-ui_2.10` and `arbiter-ui_2.11`

## [0.8.0版本发行说明](fa-xing-shuo-ming.md)

* 添加迁移学习API [Link](https://github.com/eclipse/deeplearning4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/transferlearning/vgg16)
* Spark 2.0支持（DL4J和DataVec；请参阅下面的迁移说明）
* 新层
  * 全局池化（也称为“随时间池化”；可用于RNN和CNN） [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/pooling/GlobalPoolingLayer.java)
  * 中心损失输出层 [Link](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/centerloss/CenterLossLenetMnistExample.java)
  * 一维卷积和子采样层 [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/convolution/Convolution1DLayer.java) [Link2](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/convolution/subsampling/Subsampling1DLayer.java)
  * ZeroPaddingLayer [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/layers/convolution/ZeroPaddingLayer.java)
* 新 ComputationGraph 顶点
  * L2距离顶点
  * L2归一化顶点
* 现在大多数损失函数都支持每输出掩码（对于每输出掩码，使用与标签数组大小/形状相等的掩码数组；以前的掩码功能是针对RNN的每个示例）
* L1和L2正则化现在可以针对偏置配置（通过l1Bias和l2Bias配置选项）
* 评估改进：
  * DL4J现在有一个IEvaluation类（Evaluation、RegressionEvaluation等都实现了。也允许对Spark进行自定义评估） [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/IEvaluation.java)
  * 添加了多个类（一对所有） ROC: ROCMultiClass [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/ROCMultiClass.java)
  * 对于MultiLayerNetwork和SparkDl4jMultiLayer：添加了evaluateRegression、evaluateROC、evaluateROCMultiClass便利方法
  * 为ROC图表添加了HTML导出功能 [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-core/src/main/java/org/deeplearning4j/evaluation/EvaluationTools.java#L93)
  * TSNE重新添加到新UI
  * 训练用户界面：现在可以在没有internet连接的情况下使用（不再依赖外部托管的字体）
  * UI: “无数据”条件下错误处理的改进
* Epsilon配置现在用于Adam和RMSProp更新器
* 修复双向LSTM+可变长度时间序列（使用掩码）
* 添加了CnnSentenceDataSetIterator（用于“CNN句子分类”结构） [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nlp-parent/deeplearning4j-nlp/src/main/java/org/deeplearning4j/iterator/CnnSentenceDataSetIterator.java) [Link2](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/convolution/sentenceclassification/CnnSentenceClassificationExample.java)
* Spark+Kryo：现在测试序列化，如果配置错误则抛出异常（而不是记录可以忽略的错误）
* MultiLayerNetwork 现在，如果未指定名称，则添加默认图层名
* DataVec:
  * JSON/YAML支持数据分析、自定义转换等
  * 对ImageRecordReader进行了重构，以减少垃圾收集负载（从而使大型训练集提高性能）
  * 更快的质量分析。
* Arbiter: 添加新的层类型以匹配DL4J
  * Word2Vec/ParagraphVectors分词与训练的性能改进。
* 段落向量的批量推理
* Nd4j 改进
  * 可用于ND4j的新本地操作: firstIndex, lastIndex, remainder, fmod, or, and, xor.
  * OpProfiler NAN\_PANIC & INF\_PANIC 现在还要检查BLAS调用的结果。
  * Nd4.getMemoryManager\(\) 现在提供了调整GC行为的方法。
* Spark引入了Word2Vec/ParagraphVectors参数服务器的Alpha版本。请注意：目前还不推荐用于生产。
* CNN推理的性能改进

#### 0.7.2 -&gt; 0.8.0 迁移说明

* Spark版本控制方案：随着Spark 2支持的增加，Deeplearning4j和DataVec Spark模块的版本已经改变
  * 对于 Spark 1: 用 `<version>0.8.0_spark_1</version>`
  * 对于 Spark 2: 用  `<version>0.8.0_spark_2</version>`
  * 另请注意：支持Spark 2的模块仅支持Scala 2.11。Spark 1模块同时支持Scala 2.10和2.11

#### 0.8.0 已知问题（发布时）

* UI/CUDA/Linux 问题: [Link](https://github.com/eclipse/deeplearning4j/issues/3026)
* JVM退出时的脏关闭可能是CUDA的后端引起的: [Link](https://github.com/eclipse/deeplearning4j/issues/3028)
* RBM实现问题 [Link](https://github.com/eclipse/deeplearning4j/issues/3049)
* Keras 1D卷积层和池化层还不能导入。将在即将发布的版本中得到支持。
* 无法导入Keras v2模型配置。将在即将发布的版本中得到支持。

## [0.7.2版本发行说明](fa-xing-shuo-ming.md)

* 增加变分自动编码器 [Link](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/unsupervised/variational/VariationalAutoEncoderExample.java)
* 激活函数重构
  * 激活函数现在是一个接口 [Link](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/activations/IActivation.java)
  * 现在通过枚举进行配置，而不是通过 String \(查看例子 - [Link](https://github.com/eclipse/deeplearning4j-examples)\)
  * 现在支持自定义激活函数 [Link](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/activationfunctions/CustomActivationExample.java)
  * 新增激活函数: hard sigmoid, randomized leaky rectified linear units \(RReLU\)
* Keras模型导入的多个修复/改进
* 为CNN添加了P-norm池化（作为子采样层配置的一部分）
* 迭代计数持久化：在模型配置中正确存储/持久化 + Spark网络训练的学习率调度修正
* LSTM:现在可以配置门激活函数（以前：硬编码为sigmoid）
* UI:
  * 增加中文翻译
  * 修复UI+预训练层
  * 添加了与Java 7兼容的stats集合兼容性 [Link](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/userInterface/UIStorageExample_Java7.java)
  * 前端处理NAN的改进
  * 添加  UIServer.stop\(\) 方法
  * 修复分数与迭代移动平均线（带子采样）
* 用基于Spring Boot的应用程序解决了Jaxb/Jackson问题
* RecordReaderDataSetIterator现在支持对标签使用NDArrayWritable（设置 regression==true；用于多标签分类+图像等）

#### 0.7.1 -&gt; 0.7.2 迁移说明 

* 激活函数（内置）：现在使用激活枚举指定，而不是字符串（不推荐使用基于字符串的配置）

## [0.7.1发行说明](fa-xing-shuo-ming.md)

* RBM 与  AutoEncoder 键修复:
  * 确保在训练前更新和应用视觉偏置。
  * RBM HiddenUnit是该层的激活函数；因此，根据各自的HiddenUnit建立了反向传播的导数计算。
* 已修复CUDA后端的RNG性能问题
* 已修复macOS、powerpc和linux的OpenBLAS问题。
* DataVec现在回到了Java 7。
* 为ND4J/DL4J修复了多个小错误

## [0.7.0版本发行说明](fa-xing-shuo-ming.md)

* UI检修：新的训练UI有更多的信息，支持持久性（保存信息并稍后加载），日/韩/俄文支持。用Play框架替换了Dropwizard。 [Link](https://github.com/eclipse/deeplearning4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/userInterface)
* 使用[Keras](http://keras.io/)配置和训练的模型的导入
  * 导入Keras模型[配置](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/config/KerasModelConfiguration.java)和[存储的权重](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/KerasModel.java#L59)
  * 支持的模型: [Sequential](https://github.com/eclipse/deeplearning4j/blob/_old/deeplearning4j-0.7.0/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/ModelConfiguration.java) 模型 
  * 支持[层](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/config/KerasLayerConfiguration.java#L85) : _Dense, Dropout, Activation, Convolution2D, MaxPooling2D, LSTM_
* 为CNN添加更多“Same”填充（ConvolutionMode网络配置选项） [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/ConvolutionMode.java)
* 权重损失函数：损失函数现在支持每个输出权重数组（行向量）
* 为二进制分类器添加ROC和AUC [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/eval/ROC.java)
* 改进了关于无效配置或数据的错误消息；改进了对两者的验证
* 新增元数据功能：从数据导入到评估跟踪数据源（文件、行号等）。现在支持从该元数据加载示例/数据的子集。[Link](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/dataexamples/CSVExampleEvaluationMetaData.java)
* 删除了Jackson作为核心依赖项（遮挡）；用户现在可以使用Jackson的任何版本
* 添加了LossLayer：仅应用损失函数的OutputLayer版本（与OutputLayer不同：它没有权重/偏置）
* 构建三元组嵌入模型所需的功能 \(L2 vertex, LossLayer, Stack/Unstack vertices 等\)
* 减少了DL4J和ND4J的“冷启动”初始化/启动时间
* Pretrain 默认更改为false，backprop 默认更改为true。在设置网络配置时不再需要设置这些，除非需要更改默认值。
* 添加了TrainingListener接口（扩展了IterationListener）。提供网络训练时对更多信息/状态的访问 [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/api/TrainingListener.java)
* 在DL4J和ND4J上修复了许多缺陷
* nd4j-native 和 nd4j-cuda 后端的性能改进
* 独立Word2Vec/ParagraphVectors检修：
  * 性能改进
  * 对PV-DM和PV-DBOW都可用ParaVec推理
  * 添加了并行分词支持，以解决计算量大的分词器。
* 为在多线程执行环境中更好的重现性引入了本地RNG。
* 添加了其他RNG调用：Nd4j.choice\(\)，和BernoulliDistribution 操作。
* 引入了非gpu存储，以将大的东西保存在主机内存中，比如Word2Vec模型。可通过WordVectorSerializer.loadStaticModel\(\)获得
* nd4j-native后端性能优化的两个新选项：setTADThreshold\(int\) & setElementThreshold\(int\)

#### 0.6.0 -&gt; 0.7.0 迁移说明

升级基于0.6.0到0.7.0的代码库的显著变化：

* UI: 新的UI包名称是deeplearning4j-ui\_2.10或deeplearning4j-ui\_2.11（以前是deeplearning4j UI）。Scala版本后缀是必需的，因为现在正在使用Play 框架（用Scala编写）。
* 已弃用直方图和流迭代监听器。它们仍然可以工作，但建议使用新的UI [Link](https://github.com/eclipse/deeplearning4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/userInterface)
* DataVec ImageRecordReader:标签现在默认按字母顺序排序，然后根据文件迭代顺序为每个标签分配整数类索引（以前为0.6.0和更早版本）。如果需要，使用.setLabels\(List\)手动指定顺序。
* CNNs: 配置验证现在不那么严格了。使用新的ConvolutionMode选项，0.6.0等同于“Strict”模式，但新的默认值是“Truncate”
  * 有关更多详细信息，请参ConvolutionMode javadoc: [Link](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/ConvolutionMode.java)
* CNN和LSTM的Xavier权重初始化更改：Xavier现在可以更好地与原始Glorot论文和其他库对齐。Xavier权重初始化。相当于0.6.0的版本作为XAVIER\_LEGACY提供
* DataVec: 自定义的RecordReader和SequenceRecordReader类需要其他方法才能实现新的元数据功能。参考如何实现这些方法的现有记录读取器实现。
* Word2Vec/ParagraphVectors:
  * 少量新的builder方法：
    * allowParallelTokenization\(boolean\)
    * useHierarchicSoftmax\(boolean\)
  * 行为更改：批大小：现在批大小还用作sg/cbow执行计算批数量的阈值

## [0.6.0版本发行说明](fa-xing-shuo-ming.md)

* 自定义层支持
* 支持自定义损失函数
* 支持压缩的INDArrays，在海量数据上节省内存
* 适用于布尔值索引的原生支持
* 对CUDA联合操作的初步支持
* CPU和CUDA后端的显著性能改进
* 更好地支持使用CUDA和cuDNN以及多gpu集群的Spark环境
* 新的UI工具：FlowIterationListener和ConfluctionIterationListener，用于更好地洞察NN中的进程
* 用于性能跟踪的特殊IterationListener实现：PerformanceListener
* 为ParagraphVectors添加的推理实现，以及使用现有的Word2Vec模型的选项
* Deeplearning4J api上的文件大小减小很多
* `nd4j-cuda-8.0` 后端现在可用于cuda 8 RC
* 新增多个内置损失函数
* 自定义预处理器支持
* 提高Spark训练实施的性能
* 使用InputType功能改进网络配置验证

## [0.5.0版本发行说明](fa-xing-shuo-ming.md)

* FP16对CUDA的支持
* 为多gpu提供更好的性能
* 包括可选的P2P内存访问支持
* 对时间序列和图像的归一化支持
* 对标签的归一化支持
* 移除Canova并转移到DataVec: [Javadoc](http://deeplearning4j.org/datavecdoc/), [Github Repo](https://github.com/deeplearning4j/datavec)
* 大量的错误修复
* Spark 改进

### [0.4.0版本发行说明](fa-xing-shuo-ming.md)

* 独立和Spark最初的多GPU支持是可行的
* 显著地重构了Spark API
* 添加了CuDNN包装器
* ND4J的性能改进
* 引入 [DataVec](https://github.com/deeplearning4j/datavec): 许多新的功能，用于转换，预处理，清理数据。（这代替了Canova）
* 用现有数据馈送神经网络的新DataSetIterators：现有ExistingDataSetIterator，Floats\(Double\)DataSetIterator, IteratorDataSetIterator
* Word2Vec和ParagraphVectors的新学习算法：CBOW和PV-DM
* 新的本地操作以获得更好的性能： DropOut, DropOutInverted, CompareAndSet, ReplaceNaNs
* 默认情况下为多层网络和计算图启用影异步数据集预取
* 使用JVM GC和CUDA后端更好地处理内存，从而显著降低内存占用

### [资源](fa-xing-shuo-ming.md)

* [Deeplearning4j on Maven Central](https://search.maven.org/#search%7Cga%7C1%7Cdeeplearning4j)
* [Deeplearning4j Source Code](https://github.com/eclipse/deeplearning4j/)
* [ND4J Source Code](https://github.com/deeplearning4j/nd4j/)
* [libnd4j Source Code](https://github.com/deeplearning4j/libnd4j/)

### [2016年秋季路线图](fa-xing-shuo-ming.md)

* [ScalNet Scala API](https://github.com/deeplearning4j/scalnet) \(WIP!\)
* 与Keras共享的标准NN配置文件
* CGANs
* 模型可解释性


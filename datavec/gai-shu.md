---
description: DL4J向量化和ETL库概述。
---

# 概述

## 数据向量：一个向量化的ETL（抽取、转换和加载）库

数据向量解决了有效机器或深度学习的最重要障碍之一：将数据转换成神经网络可以理解的格式。神经网络理解向量。向量化是数据科学家开始在数据上训练他们的算法之前必须解决的首要问题。数据向量应该适用于你99%的数据转换，如果你不确定它是否适用于你，请在[gitter](https://gitter.im/deeplearning4j/deeplearning4j) 上咨询。数据向量支持大多数数据格式，但是您也可以实现自己的自定义记录读取器。如果你的数据是以CSV（逗号分割值）格式存在文本文件中，必须转换为数值并攫取，或者您的数据是标记图像的目录结构，那么数据向量就是帮助您组织这些数据以便在Deeping4J中使用的工具。在使用数据向量之前请阅读这一整页，特别是下面的记录读取章节。

### 视频介绍

这个视频描述了图片数据到向量的转换

{% embed url="https://youtu.be/EHHtyRKQIJ0" %}

## 关键方面

* 数据向量使用一个输入／输出格式系统（类似于MapReduce使用InputFormat来决定InputSplits（输入分割器）和RecordReaders（记录读取器）的某些方式，数据向量也提供了RecordReaders来系列化数据 ）
* 设计为支持所有主要输入数据类型（文本, CSV, 音频, 图像和视频）
* 使用一个输出格式系统来指定一个实现- 中性型向量格式（ARFF, SVMLight,等.）
* 可被扩展且于特殊的输入格式（例如exotic图片格式 ）；你可以写你自己的定制的输入格式并让代码库的其余部分处理转换管道。
* 把向量化当作一等公民
* 内置转换工具用于转换和归一化数据
* 请查看[DataVec Javadoc](https://deeplearning4j.org/api/1.0.0-beta2/)

  下面有一个简短的教程。

### 一些示例

* 将基于CSV的UCI Iris数据集转换为svmLight开放矢量文本格式 
* 从原始的二进制文件将MNIST数据集转换为svmLight开放矢量文本格式
* 转换原始文本为metronome向量格式
* 在一个文本向量格式{svmLight, metronome, arff}中将原始文本转换为基于TF-IDF\(词频-逆文件频率\) 的向量
* 在一个文本向量格式{svmLight, metronome, arff}中将原始文本转换为word2vec向量

### 定向向量化引擎

* 用脚本转换语言将任意的CSV转换为向量
* MNIST 转换为向量
* 文本转换为向量
  * TF-IDF（词频-逆文件频率）
  * 词袋
  * word2vec

### CSV转换引擎

如果数据是数字的和适当的格式，那么CSVRecordReader可能是令人满意的。然而如果你的数据有非数字属性比如有代表布尔值（T／F）的字符或是用于标签的字符那么概要转换是有必要的。数据向量使用apache spark 来完成转换操作。请注意你不需要知道spark内部机置就可以成功地用数据向量完成转换。

### 概要转换视频演示

一个简单的数据向量转换视频教程和一个如下可用的代码。

{% embed url="https://youtu.be/MLEMw2NxjxE" %}

## JAVA示例代码

我们的例子包括一系列数据向量的例子。

### 记录读取，数据迭代

如下代码展示了一个例子如何工作，原始图片，把它们转换为可以和dl4j和nd4j一起工作的格式。

```java
// 初始化 RecordReader.指定图片的高、宽、和通道.
// 注意到灰度输出通道= 1, 当用作RGB图片时, 通道=3
RecordReader recordReader = new ImageRecordReader(28, 28, 3);

// 指向一个数据路径. 
recordReader.initialize(new FileSplit(new File(labeledPath)));
```

RecordReader是数据向量中的一个类，帮助把面向字节的输入转换为面向记录的数据；元素的一个集合是以一个数字固定的并以一个惟一ID索引。把数据转换为记录是向量化的过程。记录本身是个向量，每个元素都是一个特征。[ImageRecordReader](https://github.com/deeplearning4j/DataVec/blob/a64389c08396bb39626201beeabb7c4d5f9288f9/datavec-data/datavec-data-image/src/main/java/org/datavec/image/recordreader/ImageRecordReader.java) 是 RecordReader的子类并且它内置自动摄取28×28像素图像。

因此，LFW图像被缩放到28像素×28像素。你可以通过更改传给ImageRecordReader的参数来更改维度以匹配自定义图像，只要你确保适应nIn超参数即可，nIn超参数等于图像高度x图像宽度的乘积。上面显示的其他参数包括true，它指示读取器将标签附加到记录中，标签是用于验证神经网络模型结果的一组监督值（例如，目标） 这里是所有来自数据向量预构建的记录读取器的扩展（在IntelliJ中，你可以通过在RecordReader右击，在下拉菜单中再点击 Go go 并选择 Implementations 来找到它们）

![Alt text](http://upload-images.jianshu.io/upload_images/14495907-0fc0149ed945e760.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

DataSetIterator 是Deeplearning4J 中 用于访问列表元素的类。

迭代器遍历数据列表，顺序访问每个元素项，通过指向其当前元素跟踪其进度，并修改自身以指向遍历中的每个新步骤的下一个元素。

```java
// DataVec to DL4J
DataSetIterator iter = new RecordReaderDataSetIterator(recordReader, 784, labels.size());
```

DataSetIterator迭代输入数据集，每次迭代获取一个或多个新示例，并将这些示例加载到神经网络可以使用的数据集对象中。

需要注意的是 ImageRecordReader产生4维图像数据，匹配dl4j所需要的激活层。因此，每一个28x28 RGB图像用一个4维数组表示，例如维度\[小批量,通首，高，宽\]=\[1,3,28,28\]。注意到上面的构造器行也指明了标签的数量。注意到 ImageRecordReader 不会规一化图片数据，因此每个像素／通道值将会在0到255之间（一般应分别归一化-例如使用ND4J的 ImagePreProcessingScaler 和其它的规一化器\)。`RecordReaderDataSetIterator` 可以作为你想指定的记录读取器的参数（图片，声音）和批量大小。 对于有监督学习，它还将采取标签索引和可应用于输入的可能标签的数量（对于LFW，标签的数量是5749）。

### 执行

作为本地串行进程和一个MapReduce进程来运行，没有代码更改的扩展过程。

### 定向向量格式

* svmLight
* libsvm
* Metronome
* ARFF

### 内置通用功能

* 理解如何通用文本和文本转换为向量，并把它与例如核散列和TF-IDF库存技术结合使用。


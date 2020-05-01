---
description: 'Spark上的DL4J: 介绍'
---

# 介绍与入门

DL4J支持使用Apache Spark在CPU或GPU机器集群上进行神经网络训练。DL4J还支持分布式评估以及使用Spark的分布式推理。

### DL4J的分布式训练实现 <a id="dl4js-distributed-training-implementations"></a>

DL4J有两种分布式训练实现。

* 梯度共享，1.0.0-beta版本可用：基于Nikko Strom的[论文](http://nikkostrom.com/publications/interspeech2015/strom_interspeech2015.pdf)，是一种异步SGD实现，在Spark+Aeron中实现量化和压缩更新
* 参数平均：一个同步SGD实现，带有一个完全用Spark实现的参数服务器。

用户被引导到取代参数平均实现的梯度共享实现。梯度共享实现导致更快的训练时间，并且实现为可伸缩和容错（从1.0.0-beta3开始）。为了完整起见，本页还将介绍参数平均方法。[技术参考](https://deeplearning4j.org/docs/latest/deeplearning4j-scaleout-technicalref)部分涵盖了实现的细节。  
除了分布式训练之外，DL4J还允许用户进行分布式评估（包括同时进行多次评估）和分布式推理。请参阅[Spark上的DL4J](https://deeplearning4j.org/docs/latest/deeplearning4j-scaleout-howto)[：如何指导](https://deeplearning4j.org/docs/latest/deeplearning4j-scaleout-howto) 获取更多细节。

#### 何时使用Spark训练神经网络 <a id="when-to-use-spark-for-training-neural-networks"></a>

Spark并不总是训练神经网络最合适的工具。  
你应该使用Spark训练的场景如下：

1. 你有一组用于训练的机器集群（不仅仅是一台机器——这包括多GPU机器）
2. 你需要多台机器来训练网络
3. 你的网络大到足以证明有理由用分布式实现。

对于具有多个GPU或多个物理处理器的单台机器，用户应考虑使用DL4J的ParallelWrapper实现，如[本示例](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-cuda-specific-examples/src/main/java/org/deeplearning4j/examples/multigpu/MultiGpuLenetMnistExample.java)所示。ParallelWrapper允许在具有多个核的单台机器上轻松进行网络的数据并行训练。与ParallelWrapper相比，Spark用于单机训练的开销更高。

类似地，如果你不需要Spark（更小的网络和/或数据集）-建议使用单机训练，这通常更容易设置。

对于一个足够大的网络：这里有一个粗略的指南。如果网络需要100ms或更长时间来执行一次迭代（每个小批次的拟合操作为100ms），则分布式训练应该工作良好，具有良好的可扩展性。在每次迭代为10ms时，我们可能期望性能与节点数量的次线性缩放。在每次迭代大约1ms或更低时，通信开销可能太大：集群上的训练可能不会比单台机器上的训练更快（或者甚至更慢）。为了并行性优于通信开销，用户应该考虑网络传输时间与计算时间的比率，并确保计算时间足够大，以掩盖分布式训练的额外开销。

#### 设置与依赖 <a id="setup-and-dependencies"></a>

为了在GPU上运行训练，请确保你在pom文件中指定了正确的后端（GPU的nd4j-cuda-x.x相对于CPU的nd4j-native 后端），并且已经用适当的CUDA库设置了机器。请参阅[Spark上的DL4J：如何指导](https://deeplearning4j.org/docs/latest/deeplearning4j-scaleout-howto) 获取更多细节。

使用梯度共享实现包括以下依赖性：

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>dl4j-spark-parameterserver_${scala.binary.version}</artifactId>
    <version>${dl4j.spark.version}</version>
</dependency>
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如果使用参数平均实现（同样，梯度共享实现应该是优选的），则包括：

```markup
<dependency>
        <groupId>org.deeplearning4j</groupId>
        <artifactId>dl4j-spark_${scala.binary.version}</artifactId>
        <version>${dl4j.spark.version}</version>
</dependency>
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

注意，${scala.binary.version}是一个Maven属性，值为2.10或2.11，应该与你正在使用的Spark版本匹配。

### 关键概念 <a id="key-concepts"></a>

以下是用户开始使用DL4J进行分布式训练时应该熟悉的关键类。

* **TrainingMaster**: 指定如何在实践中进行分布式训练。实现包括梯度共享（SharedTrainingMaster）或参数平均（ParameterAveragingTrainingMaster）
* **SparkDl4jMultiLayer 与 SparkComputationGraph**: 它们是DL4J中的MultiLayerNetwork和ComutationGraph类的包装器，支持与分布式训练相关的功能。为了训练，他们配置了一个TrainingMaster。
* **`RDD<DataSet>` 与 `RDD<MultiDataSet>`**: 具有DL4J的DataSet或MultiDataSet类的Spark RDD定义训练数据（或评估数据）的源。注意，推荐的最佳实践是只对数据进行一次预处理，并将其保存到诸如HDFS之类的网络存储中。有关更多细节，请参考[Spark上的DL4J：如何构建数据管道](https://deeplearning4j.org/docs/latest/deeplearning4j-scaleout-data-howto)。

训练工作流程通常如下：

1. 准备带有几个组件的训练代码：a.神经网络配置b.数据管道c.SparkDl4jMultiLayer/SparkComputationGraph加上Trainingmaster
2. 创建 uber-JAR 文件\(请参阅[Spark上的DL4J：如何指导](https://deeplearning4j.org/docs/latest/deeplearning4j-scaleout-howto) 获取更多细节\)
3. 确定Spark提交的参数（内存、节点数量等）
4. 带着必要参数提交uber-JAR 到 Spark 

### 极小的示例 <a id="minimal-examples"></a>

下面的代码片段概述了所需的一般设置。[API参考](https://deeplearning4j.org/docs/latest/deeplearning4j-scaleout-apiref)概述了各个类的详细用法。用户可以向Spark Submit提交一个uber jar，以便使用正确的选项执行。请参阅[Spark DL4J：如何指导](https://deeplearning4j.org/docs/latest/deeplearning4j-scaleout-howto) 获取更多细节。

#### 梯度共享（首选实现） <a id="gradient-sharing-preferred-implementation"></a>

```java
JavaSparkContext sc = ...;
JavaRDD<DataSet> trainingData = ...;

//单节点上的模型设置。一个 MultiLayerConfiguration 或一个 ComputationGraphConfiguration
MultiLayerConfiguration model = ...;

//配置梯度共享实现所需的分布式训练
VoidConfiguration conf = VoidConfiguration.builder()
				.unicastPort(40123) //工作机将使用于通信的端口。使用任意空闲的端口
				.networkMask(“10.0.0.0/16”)     //用于通信的网络掩码。示例100.0.0/24，或192.1680.0/16等
				.controllerAddress("10.0.2.4")  //主/驱动器IP
				.build();

//创建TrainingMaster实例
TrainingMaster trainingMaster = new SharedTrainingMaster.Builder(conf)
				.batchSizePerWorker(batchSizePerWorker) //训练批量大小
				.updatesThreshold(1e-3)                 //更新量化/压缩阈值。见技术说明页
				.workersPerNode(numWorkersPerNode)      // numWorkersPerNode等于GPU的数量。对于CPU：numWorkersPerNode为1；CPU大核心numWorkersPerNode大于1
                .meshBuildMode(MeshBuildMode.MESH)      // 对于32个节点使用 MeshBuildMode.PLAIN 
				.build();

//创建SparkDl4jMultiLayer实例
SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, model, trainingMaster);

//执行训练:
for (int i = 0; i < numEpochs; i++) {
    sparkNet.fit(trainingData);
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

#### 参数平均实现 <a id="parameter-averaging-implementation"></a>

```java
JavaSparkContext sc = ...;
JavaRDD<DataSet> trainingData = ...;

//单节点上的模型设置。一个 MultiLayerConfiguration 或一个 ComputationGraphConfiguration
MultiLayerConfiguration model = ...;

//创建TrainingMaster 实例
int examplesPerDataSetObject = 1;
TrainingMaster trainingMaster = new ParameterAveragingTrainingMaster.Builder(examplesPerDataSetObject)
				.(other configuration options)
				.build();

//创建SparkDl4jMultiLayer实例并使用训练数据拟合网络
SparkDl4jMultiLayer sparkNetwork = new SparkDl4jMultiLayer(sc, model, trainingMaster);

//执行训练:
for (int i = 0; i < numEpochs; i++) {
    sparkNet.fit(trainingData);
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 进一步阅读 <a id="further-reading"></a>

* [Spark上的DL4J: 技术说明](https://deeplearning4j.org/docs/latest/deeplearning4j-scaleout-technicalref)
* [Spark上的DL4J: 操作指南](https://deeplearning4j.org/docs/latest/deeplearning4j-scaleout-howto)
* [Spark上的DL4J: 如何构建数据管道](https://deeplearning4j.org/docs/latest/deeplearning4j-scaleout-data-howto)
* [Spark上的DL4J: API 参考](https://deeplearning4j.org/docs/latest/deeplearning4j-scaleout-apiref)
* [DL4J示例代码仓库](https://github.com/deeplearning4j/dl4j-examples)包含一些可供用户使用作为参考的Spark示例。


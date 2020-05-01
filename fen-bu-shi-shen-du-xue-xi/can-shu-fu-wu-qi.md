---
description: DL4J支持使用Spark和参数服务器进行快速分布式训练。
---

# 参数服务器

DL4J支持Apache Spark环境和[Aeron](https://github.com/real-logic/Aeron)中的分布式训练，用于Spark之外的高性能节点间通信。这个思想相对简单：单个工作节点在他们的数据集上计算梯度。

在将梯度应用于网络权重之前，它们被累积在中间存储机制（每台机器一个）中。聚合之后，超过某些可配置阈值的更新值作为稀疏二进制数组在网络上传播。低于阈值的值被存储并添加到将来的更新中，因此它们不会丢失，而只是在通信中延迟。

与发送整个密集更新或参数向量的天真方法相比，这种阈值化方法将网络通信需求减少了许多数量级，同时保持了高精度。

关于阈值化方法的更多细节，参见[Strom, 2015 - 使用商业GPU云计算进行弹性分布式DNN训练](http://nikkostrom.com/publications/interspeech2015/strom_interspeech2015.pdf)。

下面是Nikko Strom提出的原始算法中添加的一些额外优点：

* 可变阈值：如果每次迭代的更新次数太少，则阈值将自动被一个可配置的步骤值减少。
* 密集位图编码：如果更新次数太高，则使用另一种编码方案，它为任何给定的更新消息提供在线发送的“最大字节数”保证。
* 周期性地，我们发送“抖动”消息，用明显更小的阈值编码，以共享不能超过当前阈值的延迟权重。

![](../.gitbook/assets/image%20%287%29.png)

注意，使用Spark需要开销。为了确定Spark是否会帮助你，考虑使用[Performance Listener](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/PerformanceListener.java)并查看毫秒迭代时间。如果是&lt;150毫秒，Spark可能不值得。

### 设置你的集群 <a id="setting-up-your-cluster"></a>

为了运行训练你只需要一个Spark 1.x/2.x集群和至少一个开放的UDP端口（入站/出站）。

#### 集群设置 <a id="cluster-setup"></a>

如上所述，DL4J同时支持Spark 1.x和Spark 2.x集群。但是，这个特定的实现也需要Java 8 +来运行。如果你的群集运行Java 7，则你必须升级或使用我们的[参数平均训练模式](https://deeplearning4j.org/docs/latest/deeplearning4j-spark-training)。

#### 网络环境 <a id="network-environment"></a>

梯度共享严重依赖于UDP协议用于训练期间主节点和从节点之间的通信。如果你在诸如AWS或Azure之类的云环境中运行集群，则需要允许一个UDP端口用于入站/出站连接，并且必须在传递给SharedTrainingMaster构造函数的VoidConfiguration.unicastPort\(int\)bean中指定该端口。

另一个需要记住的选项：如果使用YARN（或任何其他处理Spark网络的资源管理器），则必须指定用于UDP通信的网络的网络掩码。这可以通过以下方式完成：VoidConfiguration.setNetworkMask\(“10.1.1.0/24”\)。

选择IP地址的最后一种选择是DL4J\_VOID\_IP 环境变量。在你正在运行的每个节点上设置该变量，并使用本地IP地址进行通信。

#### 子网掩码 <a id="netmask"></a>

网络掩码是CIDR符号，只是告诉软件应该使用哪些网络接口进行通信的一种方式。例如，如果你的集群有3个具有以下IP地址的盒子：192.168.1.23、192.168.1.78、192.168.2.133，则它们的网络地址的公共部分是192.168.\*，因此子网掩码是192.168.0/16。你还可以在维基百科:https://en.wikipedia.org/wiki/Subnetwork中找到子掩网码的详细解释。

当Spark集群运行在hadoop之上，或者任何其他不假定Spark IP地址已公布的环境时，我们将使用网络掩码。在这种情况下，应在VoidConfiguration bean中提供有效的网络掩码，并将使用它来选择Spark之外的通信接口。

#### 依赖 <a id="dependencies"></a>

下面是唯一需要的依赖项的模板：

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>dl4j-spark-parameterserver_${scala.binary.version}</artifactId>
    <version>${dl4j.spark.version}</version>
</dependency>
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

例如:

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>dl4j-spark-parameterserver_2.11</artifactId>
    <version>0.9.1_spark_2</version>
</dependency>
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

#### 示例配置: <a id="example-configuration"></a>

以下是来自[Github上的示例仓库](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-spark-examples/dl4j-spark/src/main/java/org/deeplearning4j/legacyExamples/mlp/MnistMLPDistributedExample.java)的一个示例项目的片段

```java
SparkConf sparkConf = new SparkConf();
sparkConf.setAppName("DL4J Spark Example");
JavaSparkContext sc = new JavaSparkContext(sparkConf);

MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
            .seed(12345)
            .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
            ...
            .build();

/*
  这是一个参数服务器配置bean。你需要用到的唯一选项是 .unicastPort(int)
*/
VoidConfiguration voidConfiguration = VoidConfiguration.builder()
            .unicastPort(40123)
            .build();

/*
    SharedTrainingMaster 是分布式训练的基础。它保存训练要求的所有逻辑
*/
TrainingMaster tm = new SharedTrainingMaster.Builder(voidConfiguration,batchSizePerWorker)
            .updatesThreshold(1e-3)
            .rddTrainingApproach(RDDTrainingApproach.Export)
            .batchSizePerWorker(batchSizePerWorker)
            .workersPerNode(4)
            .build();

//创建Spark网络
SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, conf, tm);

//执行训练:
for (int i = 0; i < numEpochs; i++) {
    sparkNet.fit(trainData);
    log.info("Completed Epoch {}", i);
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

请注意：此配置假定在集群内的所有节点上都打开了UDP端口40123。

### 有效可扩展性 <a id="effective-scalability"></a>

网络IO有自己的代价，该算法也做了一些IO。训练时间的额外开销可以计算为

更新编码时间+消息序列化时间+来自其他工作节点的更新应用程序。

原始迭代时间越长，共享带来的相对影响就越小，并且你将获得更好的假设可伸缩性。  
这是一个简单的表单，可以在扩展性预期上帮助你：

| 迭代时间 | 编码时间 | 解码时间 | 更新时间 | 服务开销 |
| :--- | :--- | :--- | :--- | :--- |
| 550 | 50 | 5 | 50 | 20 |

| 节点数量 | 每个节点的工作节点 |
| :--- | :--- |
| 8 | 4 |

可伸缩率: 70.51%

### 性能提示 执行器、核心、并行性 <a id="performance-hints"></a>

通过设计，Spark允许你为任务配置执行器的数量和每个执行器的内核数量。假设你有一个18个节点的集群 ，在每个节点中有32个核心。

在这种情况下，你的--num-executors值是18，而建议的--executor-core值在2到32之间。这个选项将基本定义RDD将分成多少个分区。

另外，你可以手动设置每个节点上使用的DL4J工作节点的具体数量。这可以通过SharedTrainingMaster.Builder\(\) workersPerNode\(int\)方法来完成。

如果你的节点使用GPU，那么通常最好将workersPerNode\(int\)设置为每盒GPU的数量，或者保持其默认值以进行自动调优。

#### 编码阈值 <a id="encoding-threshold"></a>

较高的阈值将给你提供更多的稀疏更新，这将提高网络IO性能，但它可能会（也可能会）影响神经网络的学习性能。

较低的阈值将给予你更密集的更新，因此每个单独的更新消息将变得更大。这将降低网络IO性能。单独的“最佳阈值”是不可能预测的，因为它可能因不同的结构而变化，但是1e-3的默认值是一个好的开始值。

#### 网络延迟与带宽 <a id="network-latency-vs-bandwidth"></a>

经验法则很简单：网络越快，性能就越好。1GBe网络应该被认为是绝对最小值，但是10GBe由于延迟较小而性能更好。

当然，性能取决于神经网络大小和计算量。较大的神经网络需要更大的带宽，但是每次迭代也需要更多的时间（因此可能为异步通信留下更多的时间）。

#### UDP单播与UDP广播 <a id="udp-unicast-vs-udp-broadcast"></a>

为了确保最大的兼容性（例如，对于不支持多播的云计算环境，如AWS和Azure），DL4J当前只使用UDP单播。

UDP广播传输应该更快，但是对于训练性能，差异不应该显著（除了可能非常小的工作负载）。

根据设计，每个工作节点每次迭代发送1条更新消息，并且无论UDP传输类型如何，都不会改变。由于UDP单播传输中的消息重传由主节点处理（通常利用率较低），并且由于消息传递是异步的。为了获得性能，我们只需要更新通信时间小于网络迭代时间-通常是这种情况。

#### 多GPU环境 <a id="multi-gpu-environments"></a>

在设备之间PCIe/NVLink P2P连通性盒子，可以预期最佳结果。然而，即使没有P2P，一切仍然可以正常工作。只是“稍慢”。：）


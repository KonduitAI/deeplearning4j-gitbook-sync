---
description: 本页提供了在Spark上使用DL4J进行分布式训练所需的关键类的API参考。确保您已经阅读了深入DL4J Spark训练入门指南。
---

# API参考

#### SharedTrainingMaster [\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-scaleout/spark/dl4j-spark-parameterserver/src/main/java/org/deeplearning4j/spark/parameterserver/training/SharedTrainingMaster.java) <a id="sharedtrainingmaster"></a>

SharedTrainingMaster基于Strom 2015论文“使用有价值的GPU云计算的可扩展分布式DNN训练”： [https://s3-us-west-2.amazonaws.com/amazon.jobs-public-documents/strom\_interspeech2015.pdf](https://s3-us-west-2.amazonaws.com/amazon.jobs-public-documents/strom_interspeech2015.pdf)，使用压缩量化梯度（更新）共享实现神经网络的分布式训练。DL4J实现进行了许多修改，例如可以选项，在没有多播网络支持的情况下使用基于容错和执行实现的参数服务器 。

**fromJson**

```java
public static SharedTrainingMaster fromJson(String jsonStr) 
```

通过反序列化用{-link\#toJson\(\)}序列化的JSON字符串来创建SharedTrainingMaster实例

* param jsonStr SharedTrainingMaster配置序列化为JSON

**fromYaml**

```java
public static SharedTrainingMaster fromYaml(String yamlStr) 
```

通过反序列化已用{-link\#toYaml\(\)序列化的YAML字符串来创建SharedTrainingMaster实例

* param yamlStr SharedTrainingMaster 配置序列化为YAML

**collectTrainingStats**

```java
public Builder collectTrainingStats(boolean enable) 
```

使用默认值创建SharedTrainingMaster，而不是RDD示例数

* param rddDataSetNumExamples 当从{code RDD}拟合时，每个数据集中有多少个示例？

**repartitionData**

```java
public Builder repartitionData(Repartition repartition) 
```

此参数定义何时应用重新分区（如果应用）。

* param repartition 重新分区设置
* deprecated Use {- link \#repartitioner\(Repartitioner\)}

**repartitionStrategy**

```java
public Builder repartitionStrategy(RepartitionStrategy repartitionStrategy) 
```

repartitionStrategy与{-link\#repartitionData\(Repartition\)}（定义何时应执行重新分区）一起使用，它定义如何执行重新分区。有关详细信息，请参见{-link RepartitionStrategy}

* param repartitionStrategy 要使用的重新分区策略
* deprecated Use {- link \#repartitioner\(Repartitioner\)}

**storageLevel**

```java
public Builder storageLevel(StorageLevel storageLevel) 
```

设置{-code RDD}s的存储级别。 默认值：StorageLevel.MEMORY\_ONLY SER\(\)-即，以序列化形式存储在内存中，若要使用RDD持久化，请使用{-code null}注意，仅当使用{-code RDDTrainingApproach.Direct}时（这不是默认值），以及从{-code RDD}拟合时，此选项才有效。

注意：Spark的StorageLevel.MEMORY\_ONLY\(\)和StorageLevel.MEMORY\_AND\_DISK\(\)在处理堆外数据（DL4J/ND4J广泛使用）时可能会出现问题。在决定是否/何时删除块以确保足够的可用内存时，Spark不考虑堆外内存；因此，对于大于（堆外）内存总量的数据集RDD，这可能会导致OOM问题。换句话说：Spark只计算DataSet和INDArray对象的堆上大小（这是可以忽略的），从而大大低估了真正的DataSet对象大小。因此，内存中保存的数据集比我们实际负担得起的还要多。 

还要注意，不建议直接从{-code RDD}进行拟合-最好只一次导出准备好的数据并调用（例如}{-code SparkDl4jMultiLayer.fit\(String savedDataDirectory\)}。详情请参阅DL4J的Spark网站文档。  


* param storageLevel 用于数据集RDD的存储级别

**rddTrainingApproach**

```java
public Builder rddTrainingApproach(RDDTrainingApproach rddTrainingApproach) 
```

在{-code RDD}或{-code RDD}上进行训练时使用的方法。默认值：{-link RDDTrainingApproach\#Export}，它首先将数据导出到临时目录。 

虽然可以使用{-link\#exportDirectory\(String\)}配置默认的集群临时目录，但也请注意，不建议直接从{-code RDD}进行拟合-最好只一次导出准备好的数据并调用（例如}{-code SparkDl4jMultiLayer.fit\(String savedDataDirectory\)}。详情请参阅DL4J的Spark网站文档。  


* param rddTrainingApproach  {- code RDD}从{-code RDD}或{-code RDD}进行训练时要使用的训练方法

**exportDirectory**

```java
public Builder exportDirectory(String exportDirectory) 
```

当{-link\#rddTrainingApproach\(RDDTrainingApproach\)}设置为{-link rddTrainingApproach\#Export}（默认情况下）时，首先将数据导出到临时目录。 

默认值：null。-&gt;使用{hadoop.tmp.dir}/dl4j/。在本例中，数据被导出到{hadoop.tmp.dir}/dl4j/SOME\_UNIQUE\_ID/ 如果指定目录，则将使用目录{exportDirectory}/SOME\_UNIQUE\_ID/。

* param exportDirectory 导出数据的基本目录

**rngSeed**

```java
public Builder rngSeed(long rngSeed) 
```

随机数生成器种子，主要用于在RDD上强制执行可重复的拆分/重新分区  默认值：无种子集（即随机种子）

* param rngSeed 随机数生成器种子

**updatesThreshold**

```java
public Builder updatesThreshold(double updatesThreshold)
```

* 不推荐使用{link\#thresholdAlgorithm（ThresholdAlgorithm）}和（例如）{link AdaptiveThresholdAlgorithm}

**thresholdAlgorithm**

```java
public Builder thresholdAlgorithm(ThresholdAlgorithm thresholdAlgorithm)
```

用于确定更新编码阈值的算法。较低的值可能会改善收敛性，但会增加网络通信量 值太低也可能影响网络收敛。如果观察到收敛问题，尝试将其增加或减少10倍，例如1e-4和1e-2。 有关技术细节，请参阅论文“[使用有用的GPU云计算的可扩展分布式DNN训练](https://s3-us-west-2.amazonaws.com/amazon.jobs-public-documents/strom_interspeech2015.pdf)” 另请参见{link ThresholdAlgorithm}  
  
默认值：{- link AdaptiveThresholdAlgorithm} 带有默认参数

* param thresholdAlgorithm 用于确定编码阈值的阈值算法

**residualPostProcessor**

```java
public Builder residualPostProcessor(ResidualPostProcessor residualPostProcessor)
```

残余后置处理器。有关详细信息，请参见{-link ResidualPostProcessor}。

 默认值：{code new ResidualClippingPostProcessor\(5.0，5\)}-即{link ResidualClippingPostProcessor}，每5次迭代，将残余值剪裁为当前阈值的+/- 5倍。

* param residualPostProcessor 要使用的残余后置处理器

**batchSizePerWorker**

```java
public Builder batchSizePerWorker(int batchSize) 
```

训练工作节点时使用的小批量大小。原则上，源数据（即{-code RDD}等）在每个{-code DataSet}中的示例数可以不同于我们在训练时要使用的示例数。i、 如果需要，我们可以拆分或合并数据集。

* param batchSize  拟合每个工作节点时使用的小批量大小

**workersPerNode**

```java
public Builder workersPerNode(int numWorkers) 
```

此方法允许配置每个群集节点的网络训练线程数。

 默认值：-1，根据系统中存在的硬件（即，如果在启用了GPU的系统上进行训练，则为GPU的数量）定义自动选择的工作节点数量。 在GPU上进行训练时，每个GPU应使用1个工作节点（这是默认值）。对于CPU，通常首选每个节点1个工作线程，尽管在增加工作线程数时，多个CPU（即多个物理CPU）或具有大量核心计数的CPU可能具有更好的吞吐量（即每秒更多示例），但代价是消耗更多内存。请注意，如果增加CPU系统上的工作节点线程数，则应使用{- code OMP\_NUM\_THREADS}属性设置OpenMP线程数-有关详细信息，请参阅{- link org.nd4j.config.ND4JEnvironmentVars\#OMP\_NUM\_THREADS}。例如，一台具有32个物理核心的机器可以使用4个工作节点线程，其中{- code OMP\_NUM\_THREADS=8}

* param numWorkers 每个节点上的工作节点线程数。

**debugLongerIterations**

```java
public Builder debugLongerIterations(long timeMs) 
```

此方法允许您在给定时间内使用Thread.sleep\(\)人为地延长迭代时间。 

请注意：不要在生产环境中使用该选项。它只适合调试。

* param timeMs
* return

**transport**

```java
public Builder transport(Transport transport) 
```

可选方法：VoidParameterAveraging的传输实现TransportType.CUSTOM 方法用户通常不使用

* param transport 要使用的传输
* return

**workerPrefetchNumBatches**

```java
public Builder workerPrefetchNumBatches(int prefetchNumBatches)
```

训练时要在每个工作节点上异步预取的小批量数。默认值：2，通常适用于大多数情况。在某些情况下，增加这一点可能有助于ETL（数据加载）瓶颈的解决，但代价是更大的内存消耗。

* param prefetchNumBatches 要预取的批数

**repartitioner**

```java
public Builder repartitioner(Repartitioner repartitioner)
```

要在拟合之前用于重新分区数据的重新分区程序。 DL4J执行MapPartitions操作以进行训练，因此如何对数据进行分区对性能影响很大—分区太少（或分区非常不平衡会导致群集利用率低，因为某些工作线程处于空闲状态）。较大数量的较小分区有助于避免所谓的“轮数终结”效应，即训练只能在最后一个/最慢的工作节点完成其分区后才能完成。 

默认重分区程序是{-link DefaultRepartitioner}，它平均重分区最多可达5000个分区，通常适用于大多数用途。在最坏的情况下，使用分区器时的“轮数终结”效果应限制为处理单个分区所需的最大时间。

* param repartitioner 使用的重分器程序

**workerTogglePeriodicGC**

```java
public Builder workerTogglePeriodicGC(boolean workerTogglePeriodicGC)
```

用于禁用对工作节点的定期垃圾收集调用。 相当于{- code Nd4j.getMemoryManager\(\).togglePeriodicGc\(workerTogglePeriodicGC\);} 传递false以禁用工作节点上的定期GC，或传递true（相当于默认值，或不设置它）以保持启用状态。

* param workerTogglePeriodicGC 工作节点定期垃圾收集设置

**workerPeriodicGCFrequency**

```java
public Builder workerPeriodicGCFrequency(int workerPeriodicGCFrequency)
```

用于设置工作节点的定期垃圾收集频率。 相当于对每个工作进程调用{- code Nd4j.getMemoryManager\(\).setAutoGcWindow\(workerPeriodicGCFrequency\);}

如果{- link \#workerTogglePeriodicGC\(boolean\)} 设置为false，则没有任何效果。

* param workerPeriodicGCFrequency 对工作节点使用的周期性GC频率

**encodingDebugMode**

```java
public Builder encodingDebugMode(boolean enabled)
```

启用阈值编码的调试模式。启用时，将计算阈值和残余值的各种统计信息，并记录在每个工作节点（在信息日志级别）。 此信息可用于检查编码阈值是否太大（例如，几乎所有更新都远小于阈值）或太大（大多数更新远大于阈值）。 

默认情况下禁用encodingDebugMode。

**重要提示**：启用此选项会产生性能开销，除非实际需要调试信息，否则不应启用此选项。  


* param enabled 设置为true来启用 

#### SparkComputationGraph [\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-scaleout/spark/dl4j-spark/src/main/java/org/deeplearning4j/spark/impl/graph/SparkComputationGraph.java) <a id="sparkcomputationgraph"></a>

使用Spark训练计算图网络的主要类。也用于在这些网络上执行分布式评估和推理

**getSparkContext**

```java
public JavaSparkContext getSparkContext() 
```

使用给定的上下文、网络和训练主机实例化计算图实例。

* param sparkContext 要使用的spark上下文
* param network 要使用的网络
* param trainingMaster 训练所需，如果SparkComputationGraph仅用于评估或推理，则训练所需的值可能为空

**getNetwork**

```java
public ComputationGraph getNetwork() 
```

* 返回训练的ComputationGraph

**getTrainingMaster**

```java
public TrainingMaster getTrainingMaster() 
```

* 返回此网络的TrainingMaster

**setNetwork**

```java
public void setNetwork(ComputationGraph network) 
```

* param network 用于任何后续训练、推理和评估步骤的网络

**getDefaultEvaluationWorkers**

```java
public int getDefaultEvaluationWorkers()
```

返回当前设置的默认评估工作节点／线程数。请注意，当在评估方法中显式提供工作节点数时，不使用默认值。 在许多情况下，我们可能希望它小于Spark线程的数量，以减少内存需求。例如，对于32个Spark线程和一个大型网络，我们不希望启动32个网络实例来执行评估。最好（为了满足内存需求，减少缓存抖动）使用4个工作线程。 如果未显式设置，则将使用{- link \#DEFAULT\_EVAL\_WORKERS}

* 返回默认的工作节点（线程）数。

**setDefaultEvaluationWorkers**

```java
public void setDefaultEvaluationWorkers(int workers)
```

设置默认的评估工作节点／线程数。请注意，当在评估方法中显式提供工作节点数，不使用默认值。 

在许多情况下，我们可能希望它小于Spark线程的数量，以减少内存需求。例如，对于32个Spark线程和一个大型网络，我们不希望启动32个网络实例来执行评估。最好（为了满足内存需求，减少缓存抖动）使用4个工作节点。 如果未显式设置，则将使用{- link \#DEFAULT\_EVAL\_WORKERS}

**fit**

```java
public ComputationGraph fit(RDD<DataSet> rdd) 
```

用给定的数据集拟合计算图

* param rdd 用于训练的数据
* return  要训练的网络 

**fit**

```java
public ComputationGraph fit(JavaRDD<DataSet> rdd) 
```

用给定的数据集拟合计算图

* param rdd 用于训练的数据
* return  要训练的网络 

**fit**

```java
public ComputationGraph fit(String path) 
```

使用序列化数据集对象的目录拟合SparkComputationGraph网络。这里的假设是该目录包含许多{-link DataSet}对象，每个对象都使用 {- link DataSet\#save\(OutputStream\)}序列化

* param path 包含序列化DataSet对象集的目录的路径
* return 训练后的 MultiLayerNetwork

**fit**

```java
public ComputationGraph fit(String path, int minPartitions) 
```

* 弃用 {- link \#fit\(String\)}

**fitPaths**

```java
public ComputationGraph fitPaths(JavaRDD<String> paths) 
```

使用序列化DataSet对象的路径列表拟合网络。

* param paths 路径列表
* return 训练的网络

**fitPathsMultiDataSet**

```java
public ComputationGraph fitPathsMultiDataSet(JavaRDD<String> paths) 
```

用给定的数据集拟合计算图

* param rdd 用于训练的数据
* return  要训练的网络 

**fitMultiDataSet**

```java
public ComputationGraph fitMultiDataSet(String path, int minPartitions) 
```

* 已弃用{- link \#fitMultiDataSet\(String\)}

**getScore**

```java
public double getScore() 
```

从调用fit获取上一个（平均）小批量分数。这是每个工作进程中执行的最后一个小批量的所有执行器的平均得分。

**calculateScore**

```java
public double calculateScore(JavaRDD<DataSet> data, boolean average) 
```

通过对整个数据集求和或求平均值，计算所提供{code JavaRDD}中所有示例的得分。要单独计算每个示例的分数，请使用{-link\#scoreExamples\(JavaPairRDD,boolean\)}或类似的方法之一。在每个工作节点中使用默认的小批量大小{- link SparkComputationGraph\#DEFAULT\_EVAL\_SCORE\_BATCH\_SIZE}

* param data 要评分的数据 
* param average 是否要加总或是平均分数

**calculateScore**

```java
public double calculateScore(JavaRDD<DataSet> data, boolean average, int minibatchSize) 
```

通过对整个数据集求和或求平均值，计算所提供{code JavaRDD}中所有示例的得分。要单独计算每个示例的分数，请使用{-link\#scoreExamples\(JavaPairRDD,boolean\)}或类似的方法之一。

* param data 要评分的数据 
* param average 是否要加总或是平均分数
* param minibatchSize 评分时在每个小批量中使用的示例数。如果一个分区中的示例多于此数目，则将执行多个评分操作（通过一次性完成整个分区，避免使用过多内存）

**calculateScoreMultiDataSet**

```java
public double calculateScoreMultiDataSet(JavaRDD<MultiDataSet> data, boolean average) 
```

通过对整个数据集求和或求平均值，计算所提供{code JavaRDD}中所有示例的得分。要单独计算每个示例的分数，在每个工作节点中使用默认的小批量大小{- link SparkComputationGraph\#DEFAULT\_EVAL\_SCORE\_BATCH\_SIZE}

* param data 要评分的数据 
* param average 是否要加总或是平均分数

**calculateScoreMultiDataSet**

```java
public double calculateScoreMultiDataSet(JavaRDD<MultiDataSet> data, boolean average, int minibatchSize) 
```

通过对整个数据集求和或求平均值，计算所提供{code JavaRDD}中所有示例的得分。要单独计算每个示例的分数。

* param data 要评分的数据 
* param average 是否要加总或是平均分数
* param minibatchSize 评分时在每个小批量中使用的示例数。如果一个分区中的示例多于此数目，则将执行多个评分操作（通过一次性完成整个分区，避免使用过多内存）

**scoreExamples**

```java
public JavaDoubleRDD scoreExamples(JavaRDD<DataSet> data, boolean includeRegularizationTerms) 
```

 {- link \#scoreExamples\(JavaRDD, boolean\)}的DataSet 版本

**scoreExamples**

```java
public JavaDoubleRDD scoreExamples(JavaRDD<DataSet> data, boolean includeRegularizationTerms, int batchSize) 
```

{- link \#scoreExamples\(JavaPairRDD, boolean, int\)}DataSet 版本

**scoreExamplesMultiDataSet**

```java
public JavaDoubleRDD scoreExamplesMultiDataSet(JavaRDD<MultiDataSet> data, boolean includeRegularizationTerms) 
```

 {- link \#scoreExamples\(JavaPairRDD, boolean\)}DataSet 版本

**scoreExamplesMultiDataSet**

```java
public JavaDoubleRDD scoreExamplesMultiDataSet(JavaRDD<MultiDataSet> data, boolean includeRegularizationTerms,
                    int batchSize) 
```

使用指定的批处理大小对示例进行单独评分。与 {- link \#calculateScore\(JavaRDD, boolean\)}不同，此方法分别返回每个示例的分数。如果特定示例需要评分，请使用 {- link \#scoreExamples\(JavaPairRDD, boolean\)}或{- link \#scoreExamples\(JavaPairRDD, boolean, int\)}，每个示例都可以有一个键。

* param data 要评分的数据
* param includeRegularizationTerms 如果为true：在分数中包含l1/l2正则化项（如果有）
* param batchSize 评分时要使用的批大小
* return 包含每个示例分数的JavaDoubleRDD
* 查看ComputationGraph\#scoreExamples\(MultiDataSet, boolean\)

**evaluate**

```java
public Evaluation evaluate(String path, DataSetLoader loader)
```

使用默认的批大小{- link \#DEFAULT\_EVAL\_SCORE\_BATCH\_SIZE}对示例进行单独评分。与{- link \#calculateScore\(JavaRDD, boolean\)}不同，此方法分别为每个示例返回一个分数 。

**注意**：提供的JavaPairRDD有一个与每个示例和返回的分数相关的键。

 **注意**：传入的数据集对象中必须只有一个示例（否则：在要评分的键和数据集之间不能有1:1的关联）

* param data 要评分的数据
* param includeRegularizationTerms 如果为true：在分数中包含l1/l2正则化项（如果有
* param Key 类型
* return 包含每个示例分数的{- code JavaPairRDD}
* 查看 MultiLayerNetwork\#scoreExamples\(DataSet, boolean\)

**evaluate**

```java
public Evaluation evaluate(String path, MultiDataSetLoader loader)
```

在包含一组要用{-link MultiDataSetLoader}加载的MultiDataSet对象的目录上计算单个输出网络。使用默认的批大小{- link \#DEFAULT\_EVAL\_SCORE\_BATCH\_SIZE}

* param path 包含要加载的数据集的目录的路径/URI
* return Evaluation

**evaluateROCMDS**

```java
public ROC evaluateROCMDS(JavaRDD<MultiDataSet> data) 
```

 {- link \#evaluate\(JavaRDD\)}的重载

#### SparkDl4jMultiLayer [\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-scaleout/spark/dl4j-spark/src/main/java/org/deeplearning4j/spark/impl/multilayer/SparkDl4jMultiLayer.java) <a id="sparkdl4jmultilayer"></a>

使用Spark训练多层网络的主要类。也用于在这些网络上执行分布式评估和推理

**getSparkContext**

```java
public JavaSparkContext getSparkContext() 
```

使用给定的上下文和网络实例化多层spark实例。这是预测构造函数

* param sparkContext 要使用的spark上下文
* param network 要使用的网络

**getNetwork**

```java
public MultiLayerNetwork getNetwork() 
```

* return SparkDl4jMultiLayer底层的多层网络

**getTrainingMaster**

```java
public TrainingMaster getTrainingMaster() 
```

* return 这个网络的TrainingMaster

**setNetwork**

```java
public void setNetwork(MultiLayerNetwork network) 
```

设置这个SparkDl4jMultiLayer 实例下面的网络

* param network 要设置的网络

**getDefaultEvaluationWorkers**

```java
public int getDefaultEvaluationWorkers()
```

返回当前设置的默认评估工作节点/线程数。请注意，当在评估方法中显式提供工作节点数，不使用默认值。 

在许多情况下，我们可能希望它小于Spark线程的数量，以减少内存需求。例如，对于32个Spark线程和一个大型网络，我们不希望启动32个网络实例来执行评估。最好（为了满足内存需求，减少缓存抖动）使用4个工作节点。 如果未显式设置，则将使用{- link \#DEFAULT\_EVAL\_WORKERS}

* 返回默认的评估工作节点数。

**setDefaultEvaluationWorkers**

```java
public void setDefaultEvaluationWorkers(int workers)
```

设置默认的评估工作节点／线程数。请注意，当在评估方法中显式提供工作节点数，不使用默认值。 

在许多情况下，我们可能希望它小于Spark线程的数量，以减少内存需求。例如，对于32个Spark线程和一个大型网络，我们不希望启动32个网络实例来执行评估。最好（为了满足内存需求，减少缓存抖动）使用4个工作节点。 如果未显式设置，则将使用{- link\#DEFAULT\_EVAL\_WORKERS}

**setCollectTrainingStats**

```java
public void setCollectTrainingStats(boolean collectTrainingStats) 
```

设置是否应为调试目的收集训练统计信息。默认情况下禁用统计信息收集

* param collectTrainingStats 如果为 true: 收集训练统计数据。如果为 false: 不收集

**getSparkTrainingStats**

```java
public SparkTrainingStats getSparkTrainingStats() 
```

使用 {- link \#setCollectTrainingStats\(boolean\)}启用统计信息收集后，获取训练统计信息

* return 训练统计信息

**predict**

```java
public Matrix predict(Matrix features) 
```

预测给定的特征矩阵

* param features 给定的特征矩阵
* return 预言

**predict**

```java
public Vector predict(Vector point) 
```

预测给定向量

* param point 要预测的向量
* return 预测向量

**fit**

```java
public MultiLayerNetwork fit(RDD<DataSet> trainingData) 
```

拟合 DataSet RDD。相当于fit\(trainingData.toJavaRDD\(\)\)

* param trainingData 到fitDataSet的训练数据RDD
* return  训练后的MultiLayerNetwork

**fit**

```java
public MultiLayerNetwork fit(JavaRDD<DataSet> trainingData) 
```

拟合 DataSet RDD

* param trainingData 传递到fitDataSet的训练数据RDD
* return 训练后的MultiLayerNetwork

**fit**

```java
public MultiLayerNetwork fit(String path) 
```

使用序列化数据集对象的目录拟合SparkDl4jMultiLayer网络。这里的假设是该目录包含许多{-link DataSet}对象，每个对象都使用{-link DataSet.save\(OutputStream\)}序列化

* param path 包含序列化DataSet对象的目录的路径
* return 训练后的多层网络

**fit**

```java
public MultiLayerNetwork fit(String path, int minPartitions) 
```

* 已弃用 {- link \#fit\(String\)}

**fitPaths**

```java
public MultiLayerNetwork fitPaths(JavaRDD<String> paths) 
```

使用序列化DataSet对象的路径列表拟合网络。

* param paths 路径列表
* return 已训练网络

**fitLabeledPoint**

```java
public MultiLayerNetwork fitLabeledPoint(JavaRDD<LabeledPoint> rdd) 
```

使用Spark MLLib LabeledPoint实例拟合多层网络。这将把标记的点转换为内部的DL4J数据格式，并在此基础上训练模型

* param rdd 传递到fitDataSet的rdd
* return fitDataSet的多层网络

**fitContinuousLabeledPoint**

```java
public MultiLayerNetwork fitContinuousLabeledPoint(JavaRDD<LabeledPoint> rdd) 
```

使用Spark MLLib LabeledPoint实例拟合多层网络这将把具有用于回归的连续标签的标签点转换为内部DL4J数据格式，并在此基础上训练模型

* param rdd 包含标记点的javaRDD
* return 多层网络

**getScore**

```java
public double getScore() 
```

从调用fit获取上一个\(平均\)小批量分数。这是每个工作进程中执行的最后一个小批量的所有执行器的平均得分。

**calculateScore**

```java
public double calculateScore(RDD<DataSet> data, boolean average) 
```

{-code RDD}而不是{-code JavaRDD}的{-link\#calculateScore\(JavaRDD，boolean\)}重载

**calculateScore**

```java
public double calculateScore(JavaRDD<DataSet> data, boolean average) 
```

通过对整个数据集求和或求平均值，计算所提供{code JavaRDD}中所有示例的得分。要单独计算每个示例的分数，请使用 {- link \#scoreExamples\(JavaPairRDD, boolean\)}或类似的方法之一。在每个工作节点中使用默认的小批量大小{- link SparkDl4jMultiLayer\#DEFAULT\_EVAL\_SCORE\_BATCH\_SIZE}

* param data 要评分的数据 
* param average 是否要加总或是平均分数

**calculateScore**

```java
public double calculateScore(JavaRDD<DataSet> data, boolean average, int minibatchSize) 
```



通过对整个数据集求和或求平均值，计算所提供{code JavaRDD}中所有示例的得分。要单独计算每个示例的分数，请使用 {- link \#scoreExamples\(JavaPairRDD, boolean\)}或类似的方法之一。

* param data 要评分的数据 
* param average 是否要加总或是平均分数
* param minibatchSize 评分时在每个小批量中使用的示例数。如果一个分区中的示例多于此数目，则将执行多个评分操作（通过一次性完成整个分区，避免使用过多内存）

**scoreExamples**

```java
public JavaDoubleRDD scoreExamples(RDD<DataSet> data, boolean includeRegularizationTerms) 
```

{-code RDD}重载{- link \#scoreExamples\(JavaPairRDD, boolean\)}

**scoreExamples**

```java
public JavaDoubleRDD scoreExamples(JavaRDD<DataSet> data, boolean includeRegularizationTerms) 
```

使用指定的批处理大小 {- link \#DEFAULT\_EVAL\_SCORE\_BATCH\_SIZE}对示例进行单独评分。与 {- link \#calculateScore\(JavaRDD, boolean\)}不同，此方法分别返回每个示例的分数。如果特定示例需要评分，请使用 {- link \#scoreExamples\(JavaPairRDD, boolean\)}或{- link \#scoreExamples\(JavaPairRDD, boolean, int\)}，每个示例都可以有一个键。

* param data 要评分的数据
* param includeRegularizationTerms 如果为true：在分数中包含l1/l2正则化项（如果有）
* param batchSize 评分时要使用的批大小
* return 包含每个示例分数的JavaDoubleRDD
* 查看 MultiLayerNetwork\#scoreExamples\(DataSet, boolean\)

**scoreExamples**

```java
public JavaDoubleRDD scoreExamples(RDD<DataSet> data, boolean includeRegularizationTerms, int batchSize) 
```

{-code RDD}重载{- link \#scoreExamples\(JavaRDD, boolean, int\)}

**scoreExamples**

```java
public JavaDoubleRDD scoreExamples(JavaRDD<DataSet> data, boolean includeRegularizationTerms, int batchSize) 
```

使用指定的批处理大小 {- link \#DEFAULT\_EVAL\_SCORE\_BATCH\_SIZE}对示例进行单独评分。与 {- link \#calculateScore\(JavaRDD, boolean\)}不同，此方法分别返回每个示例的分数。如果特定示例需要评分，请使用 {- link \#scoreExamples\(JavaPairRDD, boolean\)}或{- link \#scoreExamples\(JavaPairRDD, boolean, int\)}，每个示例都可以有一个键。

* param data 要评分的数据
* param includeRegularizationTerms 如果为true：在分数中包含l1/l2正则化项（如果有）
* param batchSize 评分时要使用的批大小
* return 包含每个示例分数的JavaDoubleRDD
* 查看 MultiLayerNetwork\#scoreExamples\(DataSet, boolean\)

#### ParameterAveragingTrainingMaster [\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-scaleout/spark/dl4j-spark/src/main/java/org/deeplearning4j/spark/impl/paramavg/ParameterAveragingTrainingMaster.java) <a id="parameteraveragingtrainingmaster"></a>

Spark上训练网络的实现。这是具有可配置平均周期的标准参数平均值。

**removeHook**

```java
public void removeHook(TrainingHook trainingHook) 
```

* param saveUpdater 如果为true：在进行参数平均时保存（并平均）更新器状态
* param numWorkers 群集的工作节点数（每个执行器的执行器线程数）
* param rddDataSetNumExamples {-code RDD}中每个数据集对象中的示例数
* param batchSizePerWorker 每个工作节点每次似合使用的示例数
* param averagingFrequency 平均参数的频率（以小批量为单位）
* param aggregationDepth 参数聚合中使用的聚合级别数
* param prefetchNumBatches 要异步预取的批数（0:禁用）
* param repartition 设置是否/何时对训练数据进行重新分区
* param repartitionStrategy 要使用的重新分区策略。见 {- link RepartitionStrategy}
* param collectTrainingStats 如果为true：收集用于调试/优化目的的训练统计信息

**addHook**

```java
public void addHook(TrainingHook trainingHook) 
```

为主节点添加一个钩子，用于训练前和训练后

* param trainingHook 要添加的训练钩子

**fromJson**

```java
public static ParameterAveragingTrainingMaster fromJson(String jsonStr) 
```

通过反序列化已用{- link \#toJson\(\)}序列化的JSON字符串，创建ParameterAveragingTrainingMaster实例

* param jsonStr ParameterAveragingTrainingMaster配置序列化为JSON

**fromYaml**

```java
public static ParameterAveragingTrainingMaster fromYaml(String yamlStr) 
```

通过反序列化已用{- link \#toYaml\(\)}序列化的YAML字符串，创建ParameterAveragingTrainingMaster实例

* param yamlStr ParameterAveragingTrainingMaster配置序列化为YAML

**trainingHooks**

```java
public Builder trainingHooks(Collection<TrainingHook> trainingHooks) 
```

将训练钩子添加到主节点。训练主节点将为工作节点设置所需的钩子进行训练。这可以允许像参数服务器、异步更新以及收集统计数据之类的设置。

* param trainingHooks 要添加的训练钩子
* return

**trainingHooks**

```java
public Builder trainingHooks(TrainingHook... hooks) 
```

将训练钩子添加到主节点。训练主节点将为工作节点设置所需的钩子进行训练。这可以允许像参数服务器、异步更新以及收集统计数据之类的设置。

* param hooks 要添加的训练钩子
* return

**batchSizePerWorker**

```java
public Builder batchSizePerWorker(int batchSizePerWorker) 
```

与{- link \#Builder\(Integer, int\)}相同，但基于JavaSparkContext.defaultParallelism\(\)自动设置工作节点数

* param rddDataSetNumExamples {-code RDD}中每个数据集对象中的示例数

**averagingFrequency**

```java
public Builder averagingFrequency(int averagingFrequency) 
```

平均工作节点参数的频率。 

**注意**：太高或太低可能有不同的原因。

* 过低（如1）会导致大量网络流量 
* 过高（&gt;&gt;20左右）可能导致精度问题或网络收敛问题
* param averagingFrequency 平均参数的频率（以“batchSizePerWorker”大小的小批量数为单位）

**aggregationDepth**

```java
public Builder aggregationDepth(int aggregationDepth) 
```

聚合树中用于参数同步的级别数。（默认值：2）**注意**：对于使用多个分区训练的大型模型，增加此数字将减少driver的负载，并有助于防止它成为瓶颈。

* param aggregationDepth 平均参数更新时的RDD树聚合通道。

**workerPrefetchNumBatches**

```java
public Builder workerPrefetchNumBatches(int prefetchNumBatches) 
```

在工作节点中设置要异步预取的小批量数。 

默认值：0（无预取）

* param prefetchNumBatches 要获取的小批量（batchSizePerWorker大小的数据集）的数量

**saveUpdater**

```java
public Builder saveUpdater(boolean saveUpdater) 
```

设置是否应保存更新器（即动量、adagrad等的历史状态）。注意：这可以使每个方向上的网络流量增加一倍（或更多），但可能会提高网络训练性能（对于某些更新器，如adagrad，可能更稳定）。  


这在默认情况下是启用的。

* param saveUpdater 如果为true：保留更新器状态（默认）。如果为false，则不保留（更新器将在平均后在每个工作节点中重新调整大小写）。

**repartionData**

```java
public Builder repartionData(Repartition repartition) 
```

设置是否/何时对训练数据进行重新分区。 默认值：总是重新分区（如果需要确保每个分区中的分区数和示例数正确）。

* param repartition 重新分区的设置

**repartitionStrategy**

```java
public Builder repartitionStrategy(RepartitionStrategy repartitionStrategy) 
```

repartitionStrategy与{- link \#repartionData\(Repartition\)}（定义何时应执行重新分区）一起使用，它定义了如何执行重新分区。有关详细信息，请参见{-link RepartitionStrategy}

* param repartitionStrategy 要使用的重新分区策略

**storageLevel**

```java
public Builder storageLevel(StorageLevel storageLevel) 
```

设置{-code RDD}s的存储级别。 默认值：StorageLevel.MEMORY\_ONLY\_SER\(\)-即，存储在内存中，以序列化的形式使用非RDD持久化，请使用{-code null}  


注意：Spark的StorageLevel.MEMORY\_ONLY\(\)和StorageLevel.MEMORY\_AND\_DISK\(\)在处理堆外数据（DL4J/ND4J广泛使用）时可能会出现问题。在决定是否/何时删除块以确保足够的可用内存时，Spark不考虑堆外内存；因此，对于大于（堆外）内存总量的数据集RDD，这可能会导致OOM问题。换句话说：Spark只计算DataSet和INDArray对象的堆上大小（这是可以忽略的），从而大大低估了真正的DataSet对象大小。因此，内存中保存的数据集比我们实际负担得起的还要多。

* param storageLevel 用于数据集RDD的存储级别

**storageLevelStreams**

```java
public Builder storageLevelStreams(StorageLevel storageLevelStreams) 
```

设置从流中拟合数据时使用的存储级别RDD:  PortableDataStreams \(sc.binaryFiles 通过 {- link SparkDl4jMultiLayer\#fit\(String\)} 和 {- link SparkComputationGraph\#fit\(String\)}\) 或字符路径  \(通过 {- link SparkDl4jMultiLayer\#fitPaths\(JavaRDD\)}, {- link SparkComputationGraph\#fitPaths\(JavaRDD\)} 和 {- link SparkComputationGraph\#fitPathsMultiDataSet\(JavaRDD\)}\).  


默认的存储级别是StorageLevel.MEMORY\_ONLY\(\)，这在大多数情况下应该是合适的。

* param storageLevelStreams 存储级别

**rddTrainingApproach**

```java
public Builder rddTrainingApproach(RDDTrainingApproach rddTrainingApproach) 
```

在{-code RDD}或{-code RDD}上进行训练时使用的方法。默认值：{link RDDTrainingApproach\#Export}，它首先将数据导出到临时目录

* param rddTrainingApproach 从{-code RDD}或{-code RDD}进行训练时要使用的训练方法

**exportDirectory**

```java
public Builder exportDirectory(String exportDirectory) 
```

当{-link\#rddTrainingApproach\(RDDTrainingApproach\)}设置为{- link RDDTrainingApproach\#Export}（默认情况下）时，首先将数据导出到临时目录。

 默认值：null。-&gt;使用{hadoop.tmp.dir}/dl4j/。在本例中，数据被导出到{hadoop.tmp.dir}/dl4j/SOME\_UNIQUE\_ID/ 如果指定目录，则将使用目录{exportDirectory}/SOME\_UNIQUE\_ID/。

* param exportDirectory 导出数据的基本目录

**rngSeed**

```java
public Builder rngSeed(long rngSeed) 
```

随机数生成器种子，主要用于强制RDD上的可重复拆分默认值：无种子集（即随机种子）

* param rngSeed 随机数生成器种子
* return

**collectTrainingStats**

```java
public Builder collectTrainingStats(boolean collectTrainingStats)
```

是否应启用训练统计信息收集（默认情况下禁用）。

* 查看 ParameterAveragingTrainingMaster\#setCollectTrainingStats\(boolean\)
* 查看org.deeplearning4j.spark.stats.StatsUtils\#exportStatsAsHTML\(SparkTrainingStats, OutputStream\)
* param collectTrainingStats


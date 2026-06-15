---
title: "Spark API Reference"
description: "API reference for SparkDl4jMultiLayer, SparkComputationGraph, and TrainingMaster"
---

# Spark API Reference

This page documents the key classes for distributed training with DL4J on Spark. For setup and how-to guides, see the [Spark How-To guide](spark-howto.md). For an introduction to the architecture, see the [Distributed Training Overview](overview.md).

**Contents:**
- [SparkDl4jMultiLayer](#sparkdl4jmultilayer)
- [SparkComputationGraph](#sparkcomputationgraph)
- [SharedTrainingMaster](#sharedtrainingmaster)
- [ParameterAveragingTrainingMaster](#parameteraveragingtm)

---

## <a name="sparkdl4jmultilayer"></a>SparkDl4jMultiLayer

[[source]](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-scaleout/spark/dl4j-spark/src/main/java/org/deeplearning4j/spark/impl/multilayer/SparkDl4jMultiLayer.java)

Main class for training `MultiLayerNetwork` networks using Spark. Also supports distributed evaluation and inference.

### Constructor

```java
SparkDl4jMultiLayer(JavaSparkContext sc, MultiLayerConfiguration conf, TrainingMaster trainingMaster)
SparkDl4jMultiLayer(JavaSparkContext sc, MultiLayerNetwork network, TrainingMaster trainingMaster)
```

`trainingMaster` may be `null` when the instance is used only for evaluation or inference (not training).

### Network Access

```java
public MultiLayerNetwork getNetwork()
public void setNetwork(MultiLayerNetwork network)
public JavaSparkContext getSparkContext()
public TrainingMaster getTrainingMaster()
```

### Training

```java
public MultiLayerNetwork fit(RDD<DataSet> trainingData)
public MultiLayerNetwork fit(JavaRDD<DataSet> trainingData)
```
Train from an RDD of DataSet objects. Note: fitting directly from `RDD<DataSet>` is not the recommended approach — prefer saving data to disk and using `fit(String)`.

```java
public MultiLayerNetwork fit(String path)
```
Train from a directory of serialized `DataSet` objects on network storage (HDFS, S3, etc.). The directory must contain files serialized using `DataSet.save(OutputStream)`. This is the preferred training method.

```java
public MultiLayerNetwork fitPaths(JavaRDD<String> paths)
```
Train from an RDD of paths pointing to serialized `DataSet` objects.

```java
public MultiLayerNetwork fitPaths(JavaRDD<String> paths, DataSetLoader loader)
```
Train from an RDD of paths using a custom `DataSetLoader` to deserialize each file.

```java
public MultiLayerNetwork fitLabeledPoint(JavaRDD<LabeledPoint> rdd)
public MultiLayerNetwork fitContinuousLabeledPoint(JavaRDD<LabeledPoint> rdd)
```
Convenience methods for compatibility with Spark MLLib `LabeledPoint` format. `fitContinuousLabeledPoint` is for regression targets.

### Scoring

```java
public double getScore()
```
Returns the average minibatch loss from the most recent `fit` call, averaged across all workers.

```java
public double calculateScore(JavaRDD<DataSet> data, boolean average)
public double calculateScore(JavaRDD<DataSet> data, boolean average, int minibatchSize)
```
Calculate the total or average loss across an entire RDD. `minibatchSize` controls memory use during scoring; default is `DEFAULT_EVAL_SCORE_BATCH_SIZE`.

```java
public JavaDoubleRDD scoreExamples(JavaRDD<DataSet> data, boolean includeRegularizationTerms)
public JavaDoubleRDD scoreExamples(JavaRDD<DataSet> data, boolean includeRegularizationTerms, int batchSize)
```
Return a per-example loss. Unlike `calculateScore`, this returns one value per example (not an aggregate).

### Evaluation

```java
public Evaluation evaluate(JavaRDD<DataSet> data)
public Evaluation evaluate(JavaRDD<DataSet> data, List<String> labelsList)
public Evaluation evaluate(JavaRDD<DataSet> data, List<String> labelsList, int evalNumWorkers, int evalBatchSize)
```
Classification metrics: accuracy, F1, precision, recall. `evalNumWorkers` controls how many network copies are used per Spark executor (reduces memory usage for large networks). Default is `DEFAULT_EVAL_WORKERS`.

```java
public ROC evaluateROC(JavaRDD<DataSet> data)
public ROC evaluateROC(JavaRDD<DataSet> data, int thresholdSteps, int evalNumWorkers, int evalBatchSize)
```
ROC curve evaluation for single-output binary classifiers.

```java
public ROCMultiClass evaluateROCMultiClass(JavaRDD<DataSet> data)
```
ROC evaluation for multi-class classifiers (one ROC curve per class).

```java
public RegressionEvaluation evaluateRegression(JavaRDD<DataSet> data)
```
Regression metrics: MSE, MAE, R2, etc.

```java
public IEvaluation[] doEvaluation(JavaRDD<DataSet> data, int evalNumWorkers, int evalBatchSize, IEvaluation... evaluations)
```
Perform multiple evaluations in a single pass over the data — more efficient than calling evaluation methods sequentially.

Example:
```java
IEvaluation[] results = sparkNet.doEvaluation(
    rddData, /*workers=*/ 4, /*batchSize=*/ 64,
    new Evaluation(), new ROCMultiClass());
Evaluation eval        = (Evaluation) results[0];
ROCMultiClass rocMulti = (ROCMultiClass) results[1];
```

### Distributed Inference

```java
public <K> JavaPairRDD<K, INDArray> feedForwardWithKey(
    JavaPairRDD<K, INDArray> featuresData, int batchSize)
```
Run inference on a keyed RDD of feature arrays. Returns a keyed RDD of predictions. The key `K` is used to associate inputs with outputs (Spark RDDs are unordered). Does not support mask arrays.

```java
public <K> JavaPairRDD<K, INDArray> feedForwardWithKey(
    JavaPairRDD<K, INDArray> featuresData, INDArray featureMask, int batchSize)
```
Overload that accepts an input mask array (for variable-length sequences).

### Statistics and Debugging

```java
public void setCollectTrainingStats(boolean collect)
public SparkTrainingStats getSparkTrainingStats()
```
Enable/disable detailed training statistics collection. Disabled by default. When enabled, requires internet access to an NTP server unless the time source is overridden (see [troubleshooting guide](spark-howto.md#ntperror)).

```java
public int getDefaultEvaluationWorkers()
public void setDefaultEvaluationWorkers(int workers)
```
Get/set the default number of network instances used for distributed evaluation per executor. Setting this lower than the number of Spark threads per executor reduces memory consumption for large models.

---

## <a name="sparkcomputationgraph"></a>SparkComputationGraph

[[source]](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-scaleout/spark/dl4j-spark/src/main/java/org/deeplearning4j/spark/impl/graph/SparkComputationGraph.java)

Main class for training `ComputationGraph` networks using Spark. Mirrors `SparkDl4jMultiLayer` but supports multi-input/multi-output networks via `MultiDataSet`.

### Constructor

```java
SparkComputationGraph(JavaSparkContext sc, ComputationGraphConfiguration conf, TrainingMaster trainingMaster)
SparkComputationGraph(JavaSparkContext sc, ComputationGraph network, TrainingMaster trainingMaster)
```

### Network Access

```java
public ComputationGraph getNetwork()
public void setNetwork(ComputationGraph network)
public JavaSparkContext getSparkContext()
public TrainingMaster getTrainingMaster()
```

### Training

```java
public ComputationGraph fit(RDD<DataSet> rdd)
public ComputationGraph fit(JavaRDD<DataSet> rdd)
public ComputationGraph fit(String path)
public ComputationGraph fitPaths(JavaRDD<String> paths)
public ComputationGraph fitPathsMultiDataSet(JavaRDD<String> paths)
public ComputationGraph fitMultiDataSet(String path)
```
Training methods mirror `SparkDl4jMultiLayer`. The `fitMultiDataSet` and `fitPathsMultiDataSet` variants accept `MultiDataSet` objects, enabling multi-input/multi-output training.

### Scoring

```java
public double getScore()
public double calculateScore(JavaRDD<DataSet> data, boolean average)
public double calculateScore(JavaRDD<DataSet> data, boolean average, int minibatchSize)
public double calculateScoreMultiDataSet(JavaRDD<MultiDataSet> data, boolean average)
public double calculateScoreMultiDataSet(JavaRDD<MultiDataSet> data, boolean average, int minibatchSize)
public JavaDoubleRDD scoreExamples(JavaRDD<DataSet> data, boolean includeRegularizationTerms)
public JavaDoubleRDD scoreExamplesMultiDataSet(JavaRDD<MultiDataSet> data, boolean includeRegularizationTerms)
public JavaDoubleRDD scoreExamplesMultiDataSet(JavaRDD<MultiDataSet> data, boolean includeRegularizationTerms, int batchSize)
```

### Evaluation

```java
public Evaluation evaluate(JavaRDD<DataSet> data)
public Evaluation evaluate(String path, DataSetLoader loader)
public Evaluation evaluate(String path, MultiDataSetLoader loader)
public ROC evaluateROCMDS(JavaRDD<MultiDataSet> data)
public IEvaluation[] doEvaluation(JavaRDD<DataSet> data, int evalNumWorkers, int evalBatchSize, IEvaluation... evaluations)
public IEvaluation[] doEvaluationMDS(JavaRDD<MultiDataSet> data, int evalNumWorkers, int evalBatchSize, IEvaluation... evaluations)
```

### Distributed Inference

```java
public <K> JavaPairRDD<K, INDArray[]> feedForwardWithKey(
    JavaPairRDD<K, INDArray[]> featuresData, int batchSize)
```
Returns `INDArray[]` per example (one array per output node) rather than a single `INDArray`.

### Evaluation Workers

```java
public int getDefaultEvaluationWorkers()
public void setDefaultEvaluationWorkers(int workers)
```

---

## <a name="sharedtrainingmaster"></a>SharedTrainingMaster

[[source]](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-scaleout/spark/dl4j-spark-parameterserver/src/main/java/org/deeplearning4j/spark/parameterserver/training/SharedTrainingMaster.java)

Implements distributed training using the Strom 2015 compressed gradient sharing algorithm. This is the recommended `TrainingMaster` implementation.

### Serialization

```java
public static SharedTrainingMaster fromJson(String jsonStr)
public static SharedTrainingMaster fromYaml(String yamlStr)
public String toJson()
public String toYaml()
```
Serialize/deserialize the configuration. Useful for saving the training configuration alongside saved models.

### Builder

```java
new SharedTrainingMaster.Builder(VoidConfiguration voidConf)
new SharedTrainingMaster.Builder(VoidConfiguration voidConf, int rddDataSetNumExamples)
```

#### Core Training Parameters

```java
public Builder batchSizePerWorker(int batchSize)
```
Minibatch size on each worker. The source RDD DataSets may have a different size — DL4J will split or combine them as needed.

```java
public Builder workersPerNode(int numWorkers)
```
Number of training threads per cluster node. Default: `-1` (auto-detect based on hardware). On GPU nodes, set to the number of GPUs. On CPU nodes, typically `1`; for machines with many cores and large core counts, you may increase this (set `OMP_NUM_THREADS` accordingly to avoid over-subscription).

#### Threshold and Residual Configuration

```java
public Builder thresholdAlgorithm(ThresholdAlgorithm thresholdAlgorithm)
```
Algorithm that determines the gradient encoding threshold. Default: `AdaptiveThresholdAlgorithm` which adjusts the threshold to keep sparsity between 0.0001 and 0.01. See [Spark How-To: Encoding Thresholds](spark-howto.md#threshold) for details.

```java
public Builder updatesThreshold(double updatesThreshold)
```
Deprecated. Use `thresholdAlgorithm(new FixedThresholdAlgorithm(value))` instead.

```java
public Builder residualPostProcessor(ResidualPostProcessor residualPostProcessor)
```
Controls how the residual vector (un-communicated gradient accumulation) is post-processed. Default: `ResidualClippingPostProcessor(5.0, 5)` — clips the residual to 5x the threshold every 5 steps, preventing residual explosion.

#### Cluster Topology

```java
public Builder meshBuildMode(MeshBuildMode mode)
```
Communication topology. Options:
- `MeshBuildMode.PLAIN`: Master relays all updates. Suitable for clusters with fewer than ~32 nodes.
- `MeshBuildMode.MESH`: Non-binary tree topology. Reduces master load. Recommended for larger clusters.

#### Data Handling

```java
public Builder rddTrainingApproach(RDDTrainingApproach approach)
```
How to handle `RDD<DataSet>` training data:
- `RDDTrainingApproach.Export` (default): exports to temporary HDFS directory before training.
- `RDDTrainingApproach.Direct`: uses data directly from the RDD.

Prefer `Export` — it avoids redundant recomputation and is more memory-efficient.

```java
public Builder exportDirectory(String exportDirectory)
```
Base directory for temporary data export when using `RDDTrainingApproach.Export`. Default: `{hadoop.tmp.dir}/dl4j/`.

```java
public Builder storageLevel(StorageLevel storageLevel)
```
Storage level for `RDD<DataSet>` persistence when using `RDDTrainingApproach.Direct`. Default: `MEMORY_ONLY_SER`. See [caching guidance](spark-howto.md#caching) — never use `MEMORY_ONLY` with DL4J RDDs.

```java
public Builder repartitioner(Repartitioner repartitioner)
```
Controls how data is repartitioned before training. Default: `DefaultRepartitioner` (equalizes up to 5000 partitions). Imbalanced partitions cause "end-of-epoch" stalls where the cluster waits for the slowest partition.

#### Worker Configuration

```java
public Builder workerPrefetchNumBatches(int prefetchNumBatches)
```
Number of minibatches to asynchronously prefetch on each worker. Default: `2`. Increase if ETL is a bottleneck; reduce if memory is tight.

```java
public Builder workerTogglePeriodicGC(boolean enabled)
public Builder workerPeriodicGCFrequency(int frequencyMs)
```
Configure periodic garbage collection on workers. Default (1.0.0-beta3+): GC every 5000 ms. Disable or increase the interval when using workspaces to avoid unnecessary GC pauses.

#### Debugging

```java
public Builder encodingDebugMode(boolean enabled)
```
When enabled, logs threshold, sparsity ratio, and encoding statistics on each worker at each iteration. Useful for diagnosing threshold issues. Has performance overhead — use only during investigation.

```java
public Builder collectTrainingStats(boolean enable)
```
Enable Spark-level training statistics collection. Disabled by default.

```java
public Builder debugLongerIterations(long timeMs)
```
Artificially extends each iteration by sleeping for `timeMs` milliseconds. For debugging only — never use in production.

#### Miscellaneous

```java
public Builder rngSeed(long rngSeed)
```
RNG seed for repeatable data partitioning.

```java
public Builder transport(Transport transport)
```
Custom Aeron transport implementation. Not required for standard UDP communication.

---

## <a name="parameteraveragingtm"></a>ParameterAveragingTrainingMaster

[[source]](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-scaleout/spark/dl4j-spark/src/main/java/org/deeplearning4j/spark/impl/paramavg/ParameterAveragingTrainingMaster.java)

Synchronous SGD implementation via Spark. Workers train independently for `averagingFrequency` minibatches, then parameters are averaged on the master. Superseded by `SharedTrainingMaster` — prefer gradient sharing for new projects.

### Serialization

```java
public static ParameterAveragingTrainingMaster fromJson(String jsonStr)
public static ParameterAveragingTrainingMaster fromYaml(String yamlStr)
```

### Builder

```java
new ParameterAveragingTrainingMaster.Builder(int rddDataSetNumExamples)
new ParameterAveragingTrainingMaster.Builder(Integer numWorkers, int rddDataSetNumExamples)
```

`rddDataSetNumExamples` is the number of examples per `DataSet` object in the source RDD.

#### Core Parameters

```java
public Builder batchSizePerWorker(int batchSizePerWorker)
```
Minibatch size per worker per averaging step.

```java
public Builder averagingFrequency(int averagingFrequency)
```
How often (in number of minibatches) workers synchronize with the master. Too low (e.g., 1) creates excessive network traffic. Too high (e.g., > 20) can hurt convergence. A value of 5–10 is a reasonable starting point.

```java
public Builder aggregationDepth(int aggregationDepth)
```
Depth of the aggregation tree used to reduce parameters back to the master. Default: `2`. Increase for large clusters with many partitions to avoid the driver becoming a bottleneck.

```java
public Builder saveUpdater(boolean saveUpdater)
```
Whether to include the optimizer state (momentum buffers, AdaGrad accumulators, etc.) in the averaged parameters. Default: `true`. Setting to `false` doubles or more the effective parameter server bandwidth but disables updater state sharing, which may harm convergence for adaptive optimizers.

#### Data Handling

```java
public Builder workerPrefetchNumBatches(int prefetchNumBatches)
```
Number of minibatches to asynchronously prefetch on each worker. Default: `0` (no prefetching).

```java
public Builder repartionData(Repartition repartition)
```
When to repartition training data (default: always repartition to ensure balanced partitions). Values: `Always`, `Never`, `NumPartitionsWorkersDiffers`.

```java
public Builder repartitionStrategy(RepartitionStrategy repartitionStrategy)
```
How to repartition. `SparkDefault` uses Spark's built-in shuffle; `Balanced` balances the number of examples per partition (not just the number of partitions).

```java
public Builder storageLevel(StorageLevel storageLevel)
```
Storage level for `RDD<DataSet>` persistence. Default: `MEMORY_ONLY_SER`. See [caching guidance](spark-howto.md#caching).

```java
public Builder storageLevelStreams(StorageLevel storageLevelStreams)
```
Storage level for path-based data (PortableDataStream RDDs from `fit(String)` or `fitPaths`). Default: `MEMORY_ONLY`.

```java
public Builder rddTrainingApproach(RDDTrainingApproach rddTrainingApproach)
public Builder exportDirectory(String exportDirectory)
```
Same semantics as in `SharedTrainingMaster.Builder`.

#### Miscellaneous

```java
public Builder rngSeed(long rngSeed)
public Builder collectTrainingStats(boolean collectTrainingStats)
public Builder trainingHooks(Collection<TrainingHook> trainingHooks)
public Builder trainingHooks(TrainingHook... hooks)
```

### Training Hook Interface

```java
public void addHook(TrainingHook trainingHook)
public void removeHook(TrainingHook trainingHook)
```
`TrainingHook` instances receive callbacks before and after each training step on workers. Can be used for custom monitoring or parameter manipulation.

---

## VoidConfiguration

`VoidConfiguration` is a required companion to `SharedTrainingMaster` that configures the Aeron-based communication layer.

```java
VoidConfiguration conf = VoidConfiguration.builder()
    .unicastPort(40123)              // UDP port — must be open inbound/outbound on all nodes
    .networkMask("10.0.0.0/16")      // CIDR notation; selects which NIC to use for Aeron communication
    .controllerAddress("10.0.2.4")   // IP of the Spark driver/master
    .build();
```

**`unicastPort`**: Any available UDP port. Must be open (both inbound and outbound) on all cluster nodes. Configure your firewall/security groups accordingly.

**`networkMask`**: CIDR-format network mask that selects the network interface used for Aeron communication. Required when running on YARN or in environments (AWS, Azure) where Spark's detected IP may differ from the desired communication interface. Example: `192.168.0.0/16`, `10.1.2.0/24`.

**`controllerAddress`**: The IP address of the Spark master/driver. Workers use this to connect to the parameter server master.

As a fallback when automatic interface selection fails, set the `DL4J_VOID_IP` environment variable on each node to the IP address to use for Aeron communication.

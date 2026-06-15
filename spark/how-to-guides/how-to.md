---
title: "Spark Training How-To"
description: "Step-by-step guide to distributed training with Apache Spark — setup, data loading, training, and evaluation"
---

# Deeplearning4j on Spark: How-To Guide

This page covers practical how-to tasks for training neural networks with DL4J on Apache Spark. For data pipeline guides, see [Spark Data Pipelines](spark-data-howto.md). For a conceptual introduction, see the [Distributed Training Overview](overview.md).

**Contents**

Before Training:
- [Build an uber-JAR with Maven](#uberjar)
- [Use GPUs for training on Spark](#gpus)
- [Use CPUs on master, GPUs on workers](#cpusgpus)
- [Configure memory settings for Spark](#memory)
- [Configure garbage collection on workers](#gc)
- [Use Kryo serialization](#kryo)
- [Use YARN with GPUs](#yarngpus)
- [Configure Spark locality](#locality)

During and After Training:
- [Configure encoding thresholds](#threshold)
- [Perform distributed evaluation](#evaluation)
- [Save and load networks trained on Spark](#saveload)
- [Perform distributed inference](#inference)

Troubleshooting:
- [Debug dependency problems](#dependencyproblems)
- [Fix "Error querying NTP server"](#ntperror)
- [Cache RDD DataSet objects safely](#caching)
- [Fix libgomp issues on Amazon EMR](#libgomp)
- [Failed training on Ubuntu 16.04](#ubuntu16)

---

## Before Training

### <a name="uberjar"></a>Build an Uber-JAR for Spark Submit

When submitting a training job to a cluster, you need an "uber-jar" — a single JAR file containing all dependencies required to run the job. Spark submit adds Spark itself to the classpath; everything else must be bundled in your JAR.

**Step 1: Decide on dependencies**

For CPU training on Spark with gradient sharing, include at minimum:

```xml
<!-- Core DL4J -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-core</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- Gradient sharing Spark module -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>dl4j-spark-parameterserver_2.11</artifactId>
    <version>${dl4j.spark.version}</version>
</dependency>

<!-- CPU backend -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native-platform</artifactId>
    <version>${nd4j.version}</version>
</dependency>
```

For Spark 2 / Scala 2.11 (most common):
```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>dl4j-spark_2.11</artifactId>
    <version>1.0.0-beta7_spark_2</version>
</dependency>
```

For Spark 1 / Scala 2.10:
```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>dl4j-spark_2.10</artifactId>
    <version>1.0.0-beta7_spark_1</version>
</dependency>
```

The Scala version suffix (`_2.10` or `_2.11`) must match your cluster's Spark build exactly. Mismatches cause runtime `AbstractMethodError` or `ClassNotFoundException` failures.

You can set the Spark dependency to `provided` scope if you only need it at compile time and your cluster provides it:
```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.11</artifactId>
    <version>2.4.0</version>
    <scope>provided</scope>
</dependency>
```

**GPU dependencies:**

Case 1 — CUDA toolkit is installed on cluster nodes:
```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-10.2</artifactId>   <!-- match installed CUDA version -->
    <version>${nd4j.version}</version>
</dependency>
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-cuda-10.2</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

Case 2 — CUDA toolkit is NOT installed on cluster nodes, include the platform variant to bundle native libraries:
```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-10.2-platform</artifactId>
    <version>${nd4j.version}</version>
</dependency>
```

**Step 2: Configure the Maven shade plugin**

Use the Maven shade plugin to produce the uber-jar. Add this to your `pom.xml`:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.2.4</version>
            <configuration>
                <shadedArtifactAttached>true</shadedArtifactAttached>
                <shadedClassifierName>bin</shadedClassifierName>
                <createDependencyReducedPom>true</createDependencyReducedPom>
                <filters>
                    <filter>
                        <artifact>*:*</artifact>
                        <excludes>
                            <exclude>org/datanucleus/**</exclude>
                            <exclude>META-INF/*.SF</exclude>
                            <exclude>META-INF/*.DSA</exclude>
                            <exclude>META-INF/*.RSA</exclude>
                        </excludes>
                    </filter>
                </filters>
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals><goal>shade</goal></goals>
                    <configuration>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                <resource>reference.conf</resource>
                            </transformer>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer"/>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

The `ServicesResourceTransformer` is required for ND4J's `ServiceLoader`-based backend discovery. Without it, the CUDA or native backend may not load correctly on the cluster.

**Step 3: Build and submit**

```bash
mvn package -DskipTests
# The shaded jar is at: target/<project>-bin.jar

spark-submit \
  --class com.example.MyTrainingJob \
  --master spark://master:7077 \
  target/myproject-bin.jar
```

---

### <a name="gpus"></a>Use GPUs for Training on Spark

DL4J's backend is configured by which ND4J dependency is on the classpath. Switch from CPU to GPU by replacing `nd4j-native-platform` with `nd4j-cuda-x.x` (or `nd4j-cuda-x.x-platform` if CUDA is not installed on the cluster).

No code changes are required — the same network configuration and training code runs on both CPU and GPU.

For cuDNN acceleration (recommended for convolutional and LSTM layers):
```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-cuda-10.2</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

This requires cuDNN library files to be present on each node (or bundled via `nd4j-cuda-x.x-platform`). See the [CuDNN configuration guide](../config/cudnn.md) for setup instructions.

---

### <a name="cpusgpus"></a>Use CPUs on Master, GPUs on Workers

If your master/driver runs on a CPU-only machine while workers have GPUs, include both backends:

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-10.2</artifactId>
    <version>${nd4j.version}</version>
</dependency>
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${nd4j.version}</version>
</dependency>
```

When both are on the classpath, ND4J tries the CUDA backend first, then falls back to native CPU. On a CPU-only driver, the CUDA backend load will fail and ND4J will automatically use CPU. On GPU workers, CUDA will be used.

You can override the backend priority via environment variables on the driver:
```bash
# Force CPU on the driver
export BACKEND_PRIORITY_CPU=100
export BACKEND_PRIORITY_GPU=0
```

The exact mechanism for setting per-role environment variables depends on your cluster manager (YARN, Mesos, Spark standalone). Consult your cluster manager's documentation.

---

### <a name="memory"></a>Configure Memory for Spark

DL4J/ND4J uses significant off-heap memory via JavaCPP. Spark's default memory settings are designed for JVM-heap-resident data and are often insufficient.

You need to configure four values:
1. Worker on-heap memory (`--executor-memory` in Spark submit)
2. Worker off-heap memory (`org.bytedeco.javacpp.maxbytes` system property)
3. Driver on-heap memory (`--driver-memory`)
4. Driver off-heap memory

**YARN example** (4 GB on-heap, 5 GB off-heap, 6 GB YARN overhead):
```bash
spark-submit \
  --executor-memory 4G \
  --driver-memory 4G \
  --conf "spark.executor.extraJavaOptions=-Dorg.bytedeco.javacpp.maxbytes=5G" \
  --conf "spark.driver.extraJavaOptions=-Dorg.bytedeco.javacpp.maxbytes=5G" \
  --conf spark.yarn.executor.memoryOverhead=6144 \
  --conf spark.yarn.driver.memoryOverhead=6144 \
  ...
```

On YARN, always set `spark.yarn.executor.memoryOverhead` and `spark.yarn.driver.memoryOverhead` to account for off-heap usage. The default values (a small fixed amount) are far too low for DL4J.

**Spark standalone** — set in `conf/spark-env.sh` on each node:
```bash
SPARK_DRIVER_OPTS=-Dorg.bytedeco.javacpp.maxbytes=12G
SPARK_DRIVER_MEMORY=8G
SPARK_WORKER_OPTS=-Dorg.bytedeco.javacpp.maxbytes=18G
SPARK_WORKER_MEMORY=12G
```

A good starting point: off-heap should be 1.5–2x on-heap for most workloads.

---

### <a name="gc"></a>Configure Garbage Collection on Workers

DL4J uses memory workspaces that keep most allocations off-heap. Frequent JVM garbage collection is therefore wasteful and can slow training. The default in 1.0.0-beta3+ is to run GC every 5 seconds on workers.

To configure GC frequency on workers via the TrainingMaster:

```java
new SharedTrainingMaster.Builder(voidConfiguration, minibatch)
    .workerTogglePeriodicGC(true)        // enable periodic GC
    .workerPeriodicGCFrequency(5000)     // run GC every 5 seconds
    .build();
```

To disable periodic GC entirely on workers:

```java
new SharedTrainingMaster.Builder(voidConfiguration, minibatch)
    .workerTogglePeriodicGC(false)
    .build();
```

Note: setting `Nd4j.getMemoryManager().setAutoGcWindow(5000)` on the driver affects only the driver, not the workers. Use the `SharedTrainingMaster.Builder` methods above to control worker GC.

---

### <a name="kryo"></a>Use Kryo Serialization

Kryo serialization can speed up Spark's shuffle operations. DL4J and ND4J provide Kryo registrators for their types.

Add the dependency:
```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-kryo_2.11</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

Then configure Spark before creating your context:
```java
SparkConf conf = new SparkConf();
conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer");
conf.set("spark.kryo.registrator", "org.nd4j.Nd4jRegistrator");
JavaSparkContext sc = new JavaSparkContext(conf);
```

If Kryo is not configured correctly, `SparkDl4jMultiLayer` and `SparkComputationGraph` will log a warning at startup. Note: because INDArrays are stored primarily off-heap, the Kryo performance benefit is smaller than for normal Java objects, but it is still generally recommended.

---

### <a name="yarngpus"></a>Use YARN with GPUs

For recent YARN versions (3.1+), YARN has built-in GPU resource scheduling. See the [YARN GPU documentation](https://hadoop.apache.org/docs/r3.1.0/hadoop-yarn/hadoop-yarn-site/UsingGpus.html) for cluster configuration.

For older YARN versions (2.7.x and earlier), GPU resource awareness is not built in. Options:
- Use node labels to target GPU nodes: [YARN Node Labels docs](https://hadoop.apache.org/docs/r2.7.3/hadoop-yarn/hadoop-yarn-site/NodeLabel.html).
- Manually specify the number of executors and ensure they are scheduled on GPU nodes.

In all cases, YARN memory overhead configuration (see [memory section](#memory)) is required when using GPUs with DL4J.

---

### <a name="locality"></a>Configure Spark Locality

Adding `--conf spark.locality.wait=0` to your Spark submit can marginally reduce training times by scheduling network fit operations sooner, at the cost of potentially less data-local task placement:

```bash
spark-submit --conf spark.locality.wait=0 ...
```

This is optional and has varying impact depending on your cluster and data layout. See the [Spark tuning guide](https://spark.apache.org/docs/latest/tuning.html#data-locality) for details.

---

## During and After Training

### <a name="threshold"></a>Configure Encoding Thresholds

Gradient sharing uses a threshold to decide which updates to communicate. Updates smaller than the threshold are stored in a residual and applied later. This is the main hyperparameter specific to distributed training.

- **Too large a threshold**: updates communicated infrequently; convergence may suffer.
- **Too small a threshold**: updates communicated more often; more network traffic per iteration.

DL4J defaults to `AdaptiveThresholdAlgorithm`, which automatically adjusts the threshold to keep the sparsity ratio (fraction of parameters communicated per update) between 0.0001 and 0.01.

Available threshold algorithms:
- `AdaptiveThresholdAlgorithm` (default): adjusts threshold to hit a target sparsity range.
- `FixedThresholdAlgorithm`: fixed threshold, no adaptation.
- `TargetSparsityThresholdAlgorithm`: adapts to hit a specific target sparsity.

Configure the threshold algorithm:
```java
TrainingMaster tm = new SharedTrainingMaster.Builder(voidConfiguration, minibatch)
    .thresholdAlgorithm(new AdaptiveThresholdAlgorithm(1e-3))   // initial threshold
    .residualPostProcessor(new ResidualClippingPostProcessor(5, 5))
    .build();
```

The `ResidualClippingPostProcessor(5, 5)` clips the residual to 5x the current threshold every 5 steps, preventing residual explosion.

Enable debug mode to log per-worker threshold statistics:
```java
new SharedTrainingMaster.Builder(...)
    .encodingDebugMode(true)
    .build();
```

Debug mode logs the threshold, sparsity ratio, and encoding statistics for each worker at each iteration. Useful for diagnosing threshold issues but has performance overhead — disable in production.

---

### <a name="evaluation"></a>Perform Distributed Evaluation

All of DL4J's standard evaluation metrics can be computed in a distributed manner on Spark.

**Step 1: Load the network on the driver**

```java
MultiLayerNetwork net = ModelSerializer.restoreMultiLayerNetwork(new File("model.bin"));
SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, net, null);
// Pass null for TrainingMaster — not needed for evaluation
```

Or for ComputationGraph:
```java
ComputationGraph net = ComputationGraph.load(new File("model.bin"), true);
SparkComputationGraph sparkNet = new SparkComputationGraph(sc, net, null);
```

**Step 2: Prepare evaluation data**

Evaluation data uses the same formats as training data:
- `JavaRDD<DataSet>` for single input/output networks
- `JavaRDD<MultiDataSet>` for multi-input/output networks
- `JavaRDD<String>` where each string is a path to a serialized DataSet on HDFS

**Step 3: Run evaluation**

```java
// Classification metrics (accuracy, F1, etc)
Evaluation eval = sparkNet.evaluate(rddDataSet);

// ROC for binary classification
ROC roc = sparkNet.evaluateROC(rddDataSet);

// Regression metrics
RegressionEvaluation regrEval = sparkNet.evaluateRegression(rddDataSet);
```

For multiple evaluations in a single pass (more efficient):
```java
IEvaluation[] evals = new IEvaluation[]{ new Evaluation(), new ROCMultiClass() };
sparkNet.doEvaluation(rddDataSet, /*batchSize=*/ 64, evals);
```

Key parameters available on evaluation methods:
- `evalNumWorkers`: number of network copies used for evaluation per node. Reduce if memory is tight.
- `evalBatchSize`: minibatch size for evaluation. 32–128 is usually a good starting range.

**Saving evaluation results to HDFS:**
```java
String json = eval.toJson();
SparkUtils.writeStringToFile("hdfs:///output/eval.json", json, sc);

// Load later:
String json = SparkUtils.readStringFromFile("hdfs:///output/eval.json", sc);
Evaluation loaded = Evaluation.fromJson(json);
```

---

### <a name="saveload"></a>Save and Load Networks Trained on Spark

`SparkDl4jMultiLayer` and `SparkComputationGraph` wrap the standard `MultiLayerNetwork` and `ComputationGraph` classes. Access the underlying network with `getNetwork()`.

**Save to local filesystem (driver):**
```java
MultiLayerNetwork net = sparkNet.getNetwork();
net.save(new File("/local/path/model.bin"));
// Or use ModelSerializer:
ModelSerializer.writeModel(net, new File("/local/path/model.bin"), true);
```

**Save to HDFS:**
```java
FileSystem fs = FileSystem.get(sc.hadoopConfiguration());
String path = "hdfs:///models/my_model.bin";
try (BufferedOutputStream os = new BufferedOutputStream(fs.create(new Path(path)))) {
    ModelSerializer.writeModel(sparkNet.getNetwork(), os, true);
}
```

**Load from HDFS:**
```java
FileSystem fs = FileSystem.get(sc.hadoopConfiguration());
String path = "hdfs:///models/my_model.bin";
MultiLayerNetwork net;
try (BufferedInputStream is = new BufferedInputStream(fs.open(new Path(path)))) {
    net = ModelSerializer.restoreMultiLayerNetwork(is);
}
```

It is good practice to save the model after each epoch during long training runs. If the master node fails, training must be restarted, and you want to resume from the most recent checkpoint rather than from scratch.

---

### <a name="inference"></a>Perform Distributed Inference

DL4J supports distributed inference — generating predictions on a cluster using a `JavaPairRDD` of inputs.

```java
// SparkDl4jMultiLayer
JavaPairRDD<String, INDArray> features = ...;  // key -> feature array
JavaPairRDD<String, INDArray> predictions =
    sparkNet.feedForwardWithKey(features, /*batchSize=*/ 64);

// SparkComputationGraph (multi-input)
JavaPairRDD<String, INDArray[]> featuresMulti = ...;
JavaPairRDD<String, INDArray[]> predictions =
    sparkCG.feedForwardWithKey(featuresMulti, 64);
```

The generic key type `K` is user-defined. It is used purely to correlate inputs with outputs, since Spark RDDs are unordered. Common key types are `String` (file paths or example IDs) or `Long` (sequence numbers).

The `batchSize` parameter controls memory/throughput tradeoff. A value of 64 is a reasonable starting point.

---

## Troubleshooting

### <a name="dependencyproblems"></a>Debug Spark Dependency Problems

Dependency conflicts at runtime produce exceptions like `NoSuchMethodException`, `ClassNotFoundException`, `AbstractMethodError`, or `UnsupportedClassVersionError`.

**Step 1: Generate a dependency tree**
```bash
mvn dependency:tree
mvn dependency:tree -Dverbose  # shows version conflict details
```

**Step 2: Check Spark version compatibility**

The artifact suffix must match your cluster:
- Spark 2, Scala 2.11: `dl4j-spark_2.11` with version ending in `_spark_2`
- Spark 1, Scala 2.11: `dl4j-spark_2.11` with version ending in `_spark_1`

Trying to run Spark 1 artifacts on a Spark 2 cluster typically produces:
```
java.lang.AbstractMethodError: org.deeplearning4j.spark.api.worker.ExecuteWorkerPathMDSFlatMap.call(...)
```

**Step 3: Check Scala version consistency**

Scan the dependency tree for mixed `_2.10` and `_2.11` suffixes. All Spark-related artifacts must use the same Scala version.

**Step 4: Check for conflicting transitive dependencies**

Common troublemakers: Jackson, Guava. These are used by Spark and many other libraries. Use Maven exclusions or explicit version declarations to pin conflicting dependencies:

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>27.0-jre</version>
</dependency>
```

**Step 5: User classpath ordering**

As a last resort for stubborn conflicts, try:
```bash
spark-submit \
  --conf spark.driver.userClassPathFirst=true \
  --conf spark.executor.userClassPathFirst=true \
  ...
```

This makes Spark load your JAR's classes before Spark's bundled classes, which can resolve cases where Spark ships an older version of a library that you've upgraded.

---

### <a name="ntperror"></a>Fix "Error querying NTP server" Errors

This error occurs when `setCollectTrainingStats(true)` is enabled and workers cannot reach an NTP server.

Solutions:
1. **Don't use `setCollectTrainingStats(true)`** — it is optional and disabled by default.
2. **Use the local system clock as the time source:**
```bash
spark-submit \
  --conf "spark.driver.extraJavaOptions=-Dorg.deeplearning4j.spark.time.TimeSource=org.deeplearning4j.spark.time.SystemClockTimeSource" \
  --conf "spark.executor.extraJavaOptions=-Dorg.deeplearning4j.spark.time.TimeSource=org.deeplearning4j.spark.time.SystemClockTimeSource" \
  ...
```

Note: using the system clock means timing statistics may be inaccurate if clocks are not synchronized across the cluster.

---

### <a name="caching"></a>Cache RDD DataSet Objects Safely

Spark's memory estimation for DL4J objects is inaccurate because `DataSet` and `INDArray` objects hold their data primarily off-heap. Spark only sees a tiny on-heap footprint and will cache far more objects than memory can actually hold, leading to OOM errors.

Rules:
- Never use `MEMORY_ONLY` or `MEMORY_AND_DISK` persistence for `RDD<DataSet>` or `RDD<INDArray>`.
- Always use `MEMORY_ONLY_SER` or `MEMORY_AND_DISK_SER`. Spark can accurately estimate serialized object sizes (which are fully on-heap).

```java
JavaRDD<DataSet> rdd = ...;
rdd.persist(StorageLevel.MEMORY_ONLY_SER());
```

---

### <a name="libgomp"></a>Fix libgomp Issues on Amazon EMR

Some Amazon EMR configurations encounter issues with OpenMP (libgomp) when running ND4J-native workloads. If you see errors related to `libgomp.so`, try setting the number of OpenMP threads explicitly:

```bash
spark-submit \
  --conf "spark.executor.extraJavaOptions=-DOMP_NUM_THREADS=1" \
  ...
```

Or set the environment variable on each node: `export OMP_NUM_THREADS=1`.

---

### <a name="ubuntu16"></a>Failed Training on Ubuntu 16.04

On Ubuntu 16.04 with Spark on YARN, all processes owned by the YARN user may be killed after a job completes. This is caused by a known Ubuntu 16.04 bug ([launchpad.net/ubuntu/+source/procps/+bug/1610499](https://bugs.launchpad.net/ubuntu/+source/procps/+bug/1610499)).

Options:
1. Add `KillUserProcesses=no` to `/etc/systemd/logind.conf` and reboot.
2. Replace `/bin/kill` with the Ubuntu 14.04 version.
3. Downgrade to Ubuntu 14.04.
4. Run `sudo loginctl enable-linger <hadoop_user>` on each cluster node.

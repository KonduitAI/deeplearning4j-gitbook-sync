---
title: "Distributed Training Overview"
description: "When and how to use distributed training — architecture overview, Spark integration, and ParallelWrapper"
---

# Distributed Training with Deeplearning4j

Deeplearning4j supports neural network training across a cluster of CPU or GPU machines using Apache Spark. It also supports distributed evaluation and distributed inference. This page introduces the architecture, explains when distributed training is worth using, and points you toward the right tool for your workload.

---

## When to Use Distributed Training

Distributed training adds operational complexity. Before reaching for Spark, ask whether you actually need it.

**Use Spark when:**
- You have a cluster of two or more machines and your network is large enough that a single machine cannot train it in a reasonable time.
- You need more compute than any single machine provides.
- Your per-iteration time is 100 ms or longer — at that level you can expect good linear scaling across nodes.

**Do not use Spark when:**
- You have a single machine with multiple GPUs. Use `ParallelWrapper` instead. It has far lower overhead than Spark for this case.
- Your per-iteration time is under 10 ms. At that scale the communication cost will dominate and Spark may be slower than single-machine training.
- Your dataset and network are small. Single-machine training is simpler to set up and debug.

A rough rule of thumb: if one forward-backward pass takes less than ~10 ms, distributed training will likely not help. If it takes 100 ms or more, you should see near-linear speed gains from adding nodes.

---

## DL4J's Two Distributed Training Implementations

### Gradient Sharing (Recommended)

Available since 1.0.0-beta. Based on the [Strom 2015 paper](http://nikkostrom.com/publications/interspeech2015/strom_interspeech2015.pdf), this is an asynchronous SGD implementation using quantized, compressed gradient updates communicated via Spark and [Aeron](https://github.com/real-logic/Aeron) (a high-performance UDP messaging library).

How it works:
1. Each worker computes gradients on its local minibatch.
2. Gradients are accumulated in an intermediate buffer on each machine.
3. Only updates above a configurable threshold are communicated — encoded as a sparse binary array.
4. Updates below the threshold are stored in a residual vector and added to the next iteration's update, so they are delayed but never lost.
5. This sparse encoding reduces network traffic by several orders of magnitude compared to sending raw parameter vectors.

DL4J extends the Strom algorithm with:
- **Adaptive thresholds**: the threshold is automatically stepped up or down to keep updates appropriately sparse.
- **Dual encoding schemes**: threshold encoding (sparse integer indexes) and bitmap encoding (fixed 2-bit-per-parameter), switching dynamically to whichever is smaller.
- **Residual clipping**: prevents "residual explosion" where accumulated un-communicated updates grow unbounded.
- **Local parallelism**: `ParallelWrapper` can be used inside each node to exploit multiple GPUs/CPUs.

This implementation is fault-tolerant as of 1.0.0-beta3. It uses Aeron for out-of-Spark UDP communication, which is essential for the low-latency messaging the algorithm requires.

### Parameter Averaging (Legacy)

A synchronous SGD implementation built entirely on Spark. The Spark master node acts as the single parameter server. At each averaging step:
1. The master broadcasts the current parameters to all workers.
2. Each worker trains on its partition for `averagingFrequency` minibatches.
3. Workers send their updated parameters back to the master, which averages them.
4. Training continues from the averaged parameters.

Parameter averaging has been superseded by gradient sharing and is included here for completeness. For new projects, use gradient sharing.

---

## Key Classes

| Class | Role |
|---|---|
| `TrainingMaster` | Interface that controls how distributed training is conducted. Implementations: `SharedTrainingMaster` (gradient sharing) and `ParameterAveragingTrainingMaster` (parameter averaging). |
| `SparkDl4jMultiLayer` | Wrapper around `MultiLayerNetwork` that adds distributed training, evaluation, and inference on Spark. |
| `SparkComputationGraph` | Same as `SparkDl4jMultiLayer` but for `ComputationGraph` networks. |
| `VoidConfiguration` | Network configuration for the Aeron-based parameter server (port, network mask, controller IP). Required for gradient sharing. |
| `RDD<DataSet>` / `RDD<MultiDataSet>` | Spark RDDs containing DL4J training data. Best practice is to preprocess and serialize these to HDFS, then load paths. |

---

## Architecture Overview

### Gradient Sharing Architecture

```
Driver (Spark Master)
    |
    +-- Worker 1 (GPU x N)
    |       local gradient accumulation
    |       threshold encoding
    |       <-- Aeron UDP -->
    +-- Worker 2 (GPU x N)
    |       local gradient accumulation
    |       threshold encoding
    |       <-- Aeron UDP -->
    +-- Worker N ...
```

Workers communicate directly with each other (and the master) via Aeron UDP, bypassing Spark's RPC layer for gradient updates. Spark is still used for data distribution and job scheduling.

Two topology modes are available:

- **Plain mode**: Each worker sends encoded updates to the master; the master relays them to all other workers. Use for clusters with fewer than ~32 nodes. The master is a potential bottleneck at scale.
- **Mesh mode**: Workers form a non-binary tree rooted at the master. Each node relays updates to its children and parent. Reduces master load significantly. Recommended for clusters larger than ~32 nodes.

### Parameter Averaging Architecture

```
Driver (Spark Master)  <-- broadcasts params, receives averages
    |
    +-- Worker 1: trains for N minibatches, sends params back
    +-- Worker 2: trains for N minibatches, sends params back
    +-- Worker N: ...
```

All communication goes through Spark. Simple but less scalable than gradient sharing.

---

## Maven Dependencies

For gradient sharing (recommended):

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>dl4j-spark-parameterserver_${scala.binary.version}</artifactId>
    <version>${dl4j.spark.version}</version>
</dependency>
```

For parameter averaging only:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>dl4j-spark_${scala.binary.version}</artifactId>
    <version>${dl4j.spark.version}</version>
</dependency>
```

`${scala.binary.version}` is `2.11` (for Spark 2.x, the most common case). The artifact suffix must match your cluster's Spark and Scala versions exactly.

---

## Minimal Example: Gradient Sharing

```java
JavaSparkContext sc = ...;
JavaRDD<DataSet> trainingData = ...;
MultiLayerConfiguration modelConf = ...;

// Aeron/parameter-server network config
VoidConfiguration voidConf = VoidConfiguration.builder()
        .unicastPort(40123)             // open UDP port on all nodes
        .networkMask("10.0.0.0/16")     // CIDR mask for the cluster network
        .controllerAddress("10.0.2.4")  // IP of the Spark driver
        .build();

TrainingMaster trainingMaster = new SharedTrainingMaster.Builder(voidConf)
        .batchSizePerWorker(32)
        .updatesThreshold(1e-3)
        .workersPerNode(4)              // typically: number of GPUs per node
        .meshBuildMode(MeshBuildMode.MESH)
        .build();

SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, modelConf, trainingMaster);

for (int epoch = 0; epoch < numEpochs; epoch++) {
    sparkNet.fit(trainingData);
}
```

---

## Minimal Example: Parameter Averaging

```java
JavaSparkContext sc = ...;
JavaRDD<DataSet> trainingData = ...;
MultiLayerConfiguration modelConf = ...;

TrainingMaster trainingMaster = new ParameterAveragingTrainingMaster.Builder(/*examplesPerDataSet=*/ 1)
        .batchSizePerWorker(32)
        .averagingFrequency(5)
        .build();

SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, modelConf, trainingMaster);

for (int epoch = 0; epoch < numEpochs; epoch++) {
    sparkNet.fit(trainingData);
}
```

---

## ParallelWrapper for Multi-GPU Single-Machine Training

When you have a single machine with multiple GPUs, `ParallelWrapper` is a simpler and faster alternative to Spark. It runs data-parallel training across all local GPUs with no network overhead.

```java
MultiLayerNetwork model = ...;
ParallelWrapper wrapper = new ParallelWrapper.Builder(model)
        .prefetchBuffer(24)
        .workers(4)               // number of GPUs
        .averagingFrequency(3)
        .reportScoreAfterAveraging(true)
        .build();

wrapper.fit(dataSetIterator);
```

See the [ParallelWrapper guide](parallel-wrapper.md) for full details and `ParallelInference` for multi-threaded serving.

---

## Typical Workflow

1. Write your network configuration, data pipeline, and training loop.
2. Build an uber-JAR with Maven shade plugin.
3. Determine Spark submit arguments: executor memory, off-heap memory, number of executors, cores per executor.
4. Submit to `spark-submit`.

The [Spark How-To guide](spark-howto.md) covers each of these steps in detail, including memory configuration, GPU setup, Kryo serialization, and troubleshooting.

---

## Further Reading

- [Spark Training How-To](spark-howto.md) — step-by-step guide from Maven to `spark-submit`
- [Spark Data Pipelines](spark-data-howto.md) — loading and preprocessing data on Spark
- [Spark API Reference](spark-api-reference.md) — `SparkDl4jMultiLayer`, `SparkComputationGraph`, `TrainingMaster`
- [Parameter Server](parameter-server.md) — Aeron-based gradient sharing architecture
- [Technical Reference](technical-reference.md) — Strom ASGD algorithm, encoding schemes, fault tolerance
- [ParallelWrapper](parallel-wrapper.md) — multi-GPU data-parallel training on a single machine

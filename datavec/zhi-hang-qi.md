---
description: 在本地实例中执行ETL和向量化。
---

# 执行器

## 本地还是远程执行？

因为数据集通常是比较大的，所以你可以决定最适合你需要的执行机制。例如，如果你正在对大型训练数据集进行向量化，则可以在分布式spark集群中处理它。但是，如果需要进行实时推理，数据向量还提供不需要任何附加设置的本地执行器。

## 执行一个转换过程

一旦你已经用概要创建了你的转换过程，并且你已经加载你的数据到了一个 Apache Spark `JavaRDD`或有一个记录读取器来加载你的数据集，你就可以执行一个转换了 。

本地执行如下：

```java
import org.datavec.local.transforms.LocalTransformExecutor;

List<List<Writable>> transformed = LocalTransformExecutor.execute(recordReader, transformProcess)

List<List<List<Writable>>> transformedSeq = LocalTransformExecutor.executeToSequence(sequenceReader, transformProcess)

List<List<Writable>> joined = LocalTransformExecutor.executeJoin(join, leftReader, rightReader)
```

当使用Spark的时候看起来是这样子的：

```java
import org.datavec.spark.transforms.SparkTransformExecutor;

JavaRDD<List<Writable>> transformed = SparkTransformExecutor.execute(inputRdd, transformProcess)

JavaRDD<List<List<Writable>>> transformedSeq = SparkTransformExecutor.executeToSequence(inputSequenceRdd, transformProcess)

JavaRDD<List<Writable>> joined = SparkTransformExecutor.executeJoin(join, leftRdd, rightRdd)
```

## 可用的执行器


---
description: 收集数据集的统计信息。
---

# 分析

## 数据分析

有时候数据集太大或格式太抽象以致于在某些列或模式上无法分析和评估统计。

数据向量附带一些辅助工具来执行数据分析，以及最大值、平均值、最小值和其他有用的度量。

## 用spark进行数据分析

如果你已经把你的数据加载到Apache Spark，数据向量有一个特殊的AnalyzeSpark类可以生成直方图，收集统计数据，并返回数据质量信息。假设你你已经把数据加载到一个Spark RDD，把 `JavaRDD` and `Schema`传到这个类。

如果你在scala中使用数据向量并且你的数据已经被加载到一个常规的`RDD` class，你可以通过调用 `.toJavaRDD()`转换它，它返回一个JavaRDD。如果你需要把它还原，调用`rdd()。`

如下的代码演示了在Spark中使用 RDD `javaRdd` 和 概要mySchema 对2D数据集的分析。

```java
import org.datavec.spark.transform.AnalyzeSpark;
import org.datavec.api.writable.Writable;
import org.datavec.api.transform.analysis.*;

int maxHistogramBuckets = 10
DataAnalysis analysis = AnalyzeSpark.analyze(mySchema, javaRdd, maxHistogramBuckets)

DataQualityAnalysis analysis = AnalyzeSpark.analyzeQuality(mySchema, javaRdd)

Writable max = AnalyzeSpark.max(javaRdd, "myColumn", mySchema)

int numSamples = 5
List<Writable> sample = AnalyzeSpark.sampleFromColumn(numSamples, "myColumn", mySchema, javaRdd)
```

注意到如果你有序列数据，这里同样有特别的方法：

```java
SequenceDataAnalysis seqAnalysis = AnalyzeSpark.analyzeSequence(mySchema, sequenceRdd)

List<Writable> uniqueSequence = AnalyzeSpark.getUniqueSequence("myColumn", seqSchema, sequenceRdd)
```

## 本地分析

AnalyzeLocal类工作起来和Spark的副本非常类似并且有相同的API。传一个可以允许迭代数据集的RecordReader来替代传RDD。

```java
import org.datavec.local.transforms.AnalyzeLocal;

int maxHistogramBuckets = 10
DataAnalysis analysis = AnalyzeLocal.analyze(mySchema, csvRecordReader, maxHistogramBuckets)
```

## 实用工具


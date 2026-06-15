---
title: "Executors"
description: "Running DataVec transform processes — LocalTransformExecutor and SparkTransformExecutor"
---

# Executors

Once you have defined a `TransformProcess`, an executor runs it against your actual data. DataVec provides two executor implementations:

- **LocalTransformExecutor** — runs transforms in the current JVM, processing records sequentially. No additional infrastructure required.
- **SparkTransformExecutor** — runs transforms distributed across a Spark cluster using `JavaRDD<List<Writable>>` as input and output.

Both executors take the same `TransformProcess` and produce the same output schema. You can develop and test locally, then switch to Spark for production-scale data without changing your transform definition.

## LocalTransformExecutor

The local executor is the easiest way to run a `TransformProcess`. Pass either a `RecordReader` or an in-memory `List<List<Writable>>`.

### Execute on a List

```java
import org.datavec.local.transforms.LocalTransformExecutor;

// Load all records into memory first (suitable for small datasets)
List<List<Writable>> originalData = new ArrayList<>();
while (reader.hasNext()) {
    originalData.add(reader.next());
}

List<List<Writable>> processedData = LocalTransformExecutor.execute(originalData, transformProcess);
```

### Execute on a RecordReader

```java
// Stream from reader without loading everything into memory
List<List<Writable>> processedData = LocalTransformExecutor.execute(reader, transformProcess);
```

### Execute Sequences (2D records -> sequences)

When a `TransformProcess` ends with a `convertToSequence` step, use `executeToSequence`:

```java
List<List<List<Writable>>> sequences =
    LocalTransformExecutor.executeToSequence(sequenceReader, transformProcess);
```

The outer list is the collection of sequences; the middle list is the time steps within one sequence; the inner list is the column values at one time step.

### Execute a Join Locally

```java
Join join = new Join.Builder(Join.JoinType.Inner)
    .setJoinColumns("customerID")
    .setSchemas(leftSchema, rightSchema)
    .build();

List<List<Writable>> joined =
    LocalTransformExecutor.executeJoin(join, leftReader, rightReader);
```

### Error Handling

By default, the local executor will throw an exception if any record fails to transform. To log errors and skip bad records instead:

```java
LocalTransformExecutor.setTryCatch(true);
List<List<Writable>> result = LocalTransformExecutor.execute(data, tp);
```

When `tryCatch` is enabled, records that cause exceptions are silently dropped and a warning is logged. Disable this in production pipelines where dropping records silently would be a problem.

## SparkTransformExecutor

The Spark executor applies the same `TransformProcess` to a `JavaRDD<List<Writable>>`. The Spark context and data loading are your responsibility; the executor handles the distributed application of the transforms.

### Setup

Add the Spark dependency to your project:

```xml
<dependency>
    <groupId>org.datavec</groupId>
    <artifactId>datavec-spark_2.11</artifactId>
    <version>${datavec.version}</version>
</dependency>
```

### Basic Execution

```java
import org.datavec.spark.transform.SparkTransformExecutor;

// Load data into Spark RDD
JavaSparkContext sc = new JavaSparkContext(sparkConf);
CSVRecordReader rr = new CSVRecordReader(1, ',');
JavaRDD<List<Writable>> inputRdd = sc.textFile(dataPath)
    .map(new StringToWritablesFunction(rr));

// Execute transform
JavaRDD<List<Writable>> transformed = SparkTransformExecutor.execute(inputRdd, transformProcess);

// Collect or save results
List<List<Writable>> results = transformed.collect();
// or
transformed.saveAsTextFile("/output/path/");
```

### Execute to Sequence

When the transform produces sequence data:

```java
JavaRDD<List<List<Writable>>> sequences =
    SparkTransformExecutor.executeToSequence(inputRdd, transformProcess);
```

### Execute from Sequence

When input is sequences but output is flat records:

```java
JavaRDD<List<Writable>> flat =
    SparkTransformExecutor.executeSequenceToSeparate(sequenceRdd, transformProcess);
```

### Execute a Join on Spark

```java
JavaRDD<List<Writable>> joined =
    SparkTransformExecutor.executeJoin(join, leftRdd, rightRdd);
```

### Error Handling on Spark

```java
SparkTransformExecutor executor = new SparkTransformExecutor();
executor.setTryCatch(true);

JavaRDD<List<Writable>> result = executor.execute(inputRdd, tp);
```

## Choosing Local vs. Spark

| Criterion | LocalTransformExecutor | SparkTransformExecutor |
|---|---|---|
| Dataset size | Up to ~1M records comfortably | Millions to billions of records |
| Infrastructure | None — runs in current JVM | Requires Spark cluster (local or distributed) |
| Development speed | Fast — no cluster startup | Slower — cluster overhead |
| Production batch | Small datasets, real-time inference | Large offline batch preprocessing |
| Real-time inference | Yes | No |

A common pattern is to use `LocalTransformExecutor` during development and testing, then switch to `SparkTransformExecutor` for the production batch preprocessing job. Because the `TransformProcess` is shared between both, the switch requires only changing the executor call.

## Using TransformProcessRecordReader Instead

For training workflows where you want to apply transforms record-by-record during DataSetIterator iteration (rather than materializing all transformed data at once), use `TransformProcessRecordReader`:

```java
RecordReader base = new CSVRecordReader(1, ',');
base.initialize(new FileSplit(new File("data.csv")));

// Transform is applied lazily on each next() call
RecordReader transformed = new TransformProcessRecordReader(base, tp);

DataSetIterator iter = new RecordReaderDataSetIterator(transformed, 32, labelIdx, numClasses);
model.fit(iter);
```

This avoids materializing the full transformed dataset in memory and integrates naturally with the DL4J training loop.

---
title: "Operations"
description: "DataVec operations — calculators, reducers, and data analysis tools"
---

# Operations

DataVec's operations layer provides low-level computational primitives used internally by transforms, reducers, and Spark executors. Most users interact with these indirectly through `TransformProcess.Builder` methods, but understanding the operations layer is useful when building custom transforms, integrating with Spark, or debugging pipeline behavior.

## Overview

Operations in DataVec are organized around two concepts:

1. **Aggregable operations** — operations that can be applied to a stream of values and combined together (sum, mean, count, etc.). These power the `Reducer` and Spark-based aggregation.
2. **Writable-typed operations** — thin wrappers around aggregable operations that handle type coercion to specific `Writable` types.

## Loading Data into Spark

When using Apache Spark, a common first step is loading a CSV file into a `JavaRDD<List<Writable>>`. The `StringToWritablesFunction` converts string lines from `sc.textFile()` into parsed `Writable` lists using a `RecordReader`:

```java
import org.datavec.api.writable.Writable;
import org.datavec.api.records.reader.impl.csv.CSVRecordReader;
import org.datavec.spark.transform.misc.StringToWritablesFunction;

SparkConf conf = new SparkConf().setAppName("DataVecExample");
JavaSparkContext sc = new JavaSparkContext(conf);

CSVRecordReader rr = new CSVRecordReader(1, ',');  // skip header

String dataPath = new ClassPathResource("data.csv").getFile().getPath();
JavaRDD<List<Writable>> data = sc.textFile(dataPath)
    .map(new StringToWritablesFunction(rr));
```

Once loaded into an RDD, you can apply `SparkTransformExecutor.execute(data, transformProcess)` or pass to `AnalyzeSpark` for statistics.

## AggregableCheckingOp

A validation wrapper around another aggregable operation. During aggregation, it checks that each value being aggregated is compatible with the expected type. Used internally during reduction operations to catch type mismatches early.

## AggregableMultiOp

Runs multiple reduction operations on the same column in a single pass. For example, computing both the mean and the standard deviation of a column simultaneously without reading the data twice. Used internally by `Reducer` when multiple operations are specified for one column.

## Type-Specific Writable Ops

Each of these classes adapts a generic aggregable operation to a specific `Writable` output type, handling the necessary conversion:

| Class | Output Type |
|---|---|
| `ByteWritableOp` | `ByteWritable` |
| `DoubleWritableOp` | `DoubleWritable` |
| `FloatWritableOp` | `FloatWritable` |
| `IntWritableOp` | `IntWritable` |
| `LongWritableOp` | `LongWritable` |
| `StringWritableOp` | `Text` |

These are used by the `Reducer` infrastructure when constructing the output record after aggregation.

## DispatchOp

Routes different columns of a multi-column record to different operations. Given a multi-column record, `DispatchOp` extracts the relevant column and delegates to the per-column operation. This is the mechanism that allows a `Reducer` to apply `mean` to one column, `sum` to another, and `max` to a third — all in one pass.

## DispatchWithConditionOp

Like `DispatchOp`, but first evaluates a condition on each element before dispatching to the appropriate column operation. Used for conditional aggregation patterns.

## CalculateSortedRank

Adds a Long column to the dataset containing the rank of each example after sorting by a specified column.

```java
import org.datavec.api.transform.rank.CalculateSortedRank;
import org.nd4j.linalg.dataset.api.preprocessor.comparator.WritableComparator;

// Add "scoreRank" column: 0 = highest score (descending sort)
Transform rankTransform = new CalculateSortedRank(
    "scoreRank",                      // new column name
    "score",                          // sort on this column
    new DoubleWritableComparator(),   // comparator
    false                             // false = descending
);

// Apply via transform step
.transform(rankTransform)
// or via convenience method
.calculateSortedRank("scoreRank", "score", new DoubleWritableComparator(), false)
```

Rank values run from 0 to `dataSetSize - 1`. This can only be applied to non-sequence (2D) data, and currently only supports sorting on a single column.

`CalculateSortedRank` is particularly useful for evaluation: after scoring all examples with your model, rank them and compute metrics at different cutoffs (e.g., precision@K).

## WritableComparators

To use `CalculateSortedRank`, you need a `WritableComparator` that defines the sort order. DataVec includes:

- `DoubleWritableComparator` — sort by double value
- `FloatWritableComparator` — sort by float value
- `IntWritableComparator` — sort by integer value
- `LongWritableComparator` — sort by long value
- `Text.Comparator` — sort by string value

Implement your own by extending `WritableComparator`:

```java
WritableComparator myComp = new WritableComparator() {
    @Override
    public int compare(Writable o1, Writable o2) {
        // custom comparison logic
        return Double.compare(o1.toDouble(), o2.toDouble());
    }
};
```

## When You Need These

For most DataVec pipelines, you will not interact with `AggregableCheckingOp`, `DispatchOp`, or the typed writable ops directly. They are used internally by:

- `Reducer.Builder` when assembling per-column reduction operations (see [Reductions](reductions.md))
- `SparkTransformExecutor` when distributing aggregation across Spark partitions
- Custom `Transform` implementations that need to produce aggregate outputs

If you are building a production pipeline that needs custom aggregation beyond what `Reducer` provides, understanding `AggregableMultiOp` and `DispatchOp` will let you compose new operations from existing primitives without re-implementing the parallelism infrastructure.

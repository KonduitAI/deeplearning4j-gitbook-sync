---
title: "Analysis"
description: "DataVec data analysis tools — profiling datasets, detecting quality issues, and computing statistics locally and on Spark"
---

# Analysis

Before transforming data, it helps to understand what you are working with. DataVec provides analysis tools that scan a dataset and return per-column statistics: type distribution, missing value counts, histograms, min/max, mean, and standard deviation. The same API works locally (via a `RecordReader`) and at scale (via Apache Spark).

## Local analysis

`AnalyzeLocal` scans data through a `RecordReader` and returns a `DataAnalysis` object.

```java
import org.datavec.local.transforms.AnalyzeLocal;
import org.datavec.api.records.reader.impl.csv.CSVRecordReader;
import org.datavec.api.split.FileSplit;
import org.datavec.api.transform.analysis.DataAnalysis;
import org.datavec.api.transform.analysis.DataQualityAnalysis;

Schema schema = new Schema.Builder()
    .addColumnString("Name")
    .addColumnDouble("Score")
    .addColumnInteger("Age")
    .build();

RecordReader rr = new CSVRecordReader(1, ',');
rr.initialize(new FileSplit(new File("data.csv")));

int maxHistogramBuckets = 10;
DataAnalysis analysis = AnalyzeLocal.analyze(schema, rr, maxHistogramBuckets);
System.out.println(analysis);
```

### Quality analysis

Quality analysis reports missing values, values that cannot be parsed according to the schema, and values that violate column metadata constraints.

```java
RecordReader rr = new CSVRecordReader(0, ',');
rr.initialize(new FileSplit(new File("data.csv")));

DataQualityAnalysis quality = AnalyzeLocal.analyzeQuality(schema, rr);
System.out.println(quality);
```

For sequence data:

```java
DataQualityAnalysis seqQuality = AnalyzeLocal.analyzeQualitySequence(schema, sequenceRecordReader);
```

### `AnalyzeLocal` API

[source](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-local/src/main/java/org/datavec/local/transforms/AnalyzeLocal.java)

```java
// Full analysis with histogram buckets
public static DataAnalysis analyze(Schema schema, RecordReader rr, int maxHistogramBuckets)

// Quality report
public static DataQualityAnalysis analyzeQuality(Schema schema, RecordReader data)

// Quality report for sequence data
public static DataQualityAnalysis analyzeQualitySequence(Schema schema, SequenceRecordReader data)
```

---

## Spark analysis

`AnalyzeSpark` mirrors the local API but operates on `JavaRDD<List<Writable>>` datasets already loaded into Apache Spark. It also adds methods for sampling individual column values.

```java
import org.datavec.spark.transform.AnalyzeSpark;
import org.datavec.api.transform.analysis.DataAnalysis;
import org.datavec.api.transform.analysis.DataQualityAnalysis;
import org.datavec.api.transform.analysis.SequenceDataAnalysis;
import org.datavec.api.writable.Writable;

// Full statistical analysis
int maxHistogramBuckets = 10;
DataAnalysis analysis = AnalyzeSpark.analyze(schema, javaRdd, maxHistogramBuckets);
System.out.println(analysis);

// Quality analysis
DataQualityAnalysis quality = AnalyzeSpark.analyzeQuality(schema, javaRdd);

// Sequence analysis
SequenceDataAnalysis seqAnalysis = AnalyzeSpark.analyzeSequence(schema, sequenceRdd, maxHistogramBuckets);
```

### Extracting min and max

```java
Writable min = AnalyzeSpark.min(javaRdd, "Score", schema);
Writable max = AnalyzeSpark.max(javaRdd, "Score", schema);
```

### Sampling values

```java
int numSamples = 5;
List<Writable> sample = AnalyzeSpark.sampleFromColumn(numSamples, "Score", schema, javaRdd);

// Sample only values that are invalid according to the schema
List<Writable> invalidSamples = AnalyzeSpark.sampleInvalidFromColumn(numSamples, "Score", schema, javaRdd);

// Get all unique values in a column
List<Writable> unique = AnalyzeSpark.getUnique("Category", schema, javaRdd);

// Get all unique values in a sequence column
List<Writable> uniqueSeq = AnalyzeSpark.getUniqueSequence("Category", seqSchema, sequenceRdd);
```

### `AnalyzeSpark` API

[source](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-spark/src/main/java/org/datavec/spark/transform/AnalyzeSpark.java)

```java
public static DataAnalysis analyze(Schema schema, JavaRDD<List<Writable>> data)
public static DataAnalysis analyze(Schema schema, JavaRDD<List<Writable>> data, int maxHistogramBuckets)
public static DataQualityAnalysis analyzeQuality(Schema schema, JavaRDD<List<Writable>> data)
public static SequenceDataAnalysis analyzeSequence(Schema schema, JavaRDD<List<List<Writable>>> data, int maxHistogramBuckets)
public static DataQualityAnalysis analyzeQualitySequence(Schema schema, JavaRDD<List<List<Writable>>> data)
public static Writable min(JavaRDD<List<Writable>> allData, String columnName, Schema schema)
public static Writable max(JavaRDD<List<Writable>> allData, String columnName, Schema schema)
public static List<Writable> sampleFromColumn(int count, String columnName, Schema schema, JavaRDD<List<Writable>> data)
public static List<Writable> sampleInvalidFromColumn(int numToSample, String columnName, Schema schema, JavaRDD<List<Writable>> data)
public static List<Writable> getUnique(String columnName, Schema schema, JavaRDD<List<Writable>> data)
public static List<Writable> getUniqueSequence(String columnName, Schema schema, JavaRDD<List<List<Writable>>> data)
```

---

## Analysis result types

### DataAnalysis

Contains a `ColumnAnalysis` entry for each column in the schema.

```java
DataAnalysis analysis = AnalyzeLocal.analyze(schema, rr, 10);

// Print the full analysis summary
System.out.println(analysis);

// Access per-column analysis
List<ColumnAnalysis> columnAnalyses = analysis.getColumnAnalysis();
for (ColumnAnalysis ca : columnAnalyses) {
    System.out.println(ca.toString());
}
```

### ColumnAnalysis implementations

| Class | Applies to |
|---|---|
| `IntegerAnalysis` | Integer columns |
| `LongAnalysis` | Long columns |
| `DoubleAnalysis` | Double/float columns |
| `StringAnalysis` | String columns |
| `CategoricalAnalysis` | Categorical columns |
| `TimeAnalysis` | Time columns |
| `BytesAnalysis` | Binary (bytes) columns |
| `NDArrayAnalysis` | NDArray columns |

Each numeric analysis includes: count, min, max, mean, standard deviation, and a histogram. String and categorical analyses include count, unique-value count, and length statistics.

### DataQualityAnalysis

Reports data quality per column.

```java
System.out.println(quality);
// Output includes per-column counts of: total values, invalid values,
// null/missing values, and out-of-range values.
```

### SequenceDataAnalysis

In addition to per-column statistics, reports sequence-level information: min/max/mean sequence length, and histogram of sequence lengths.

---

## Workflow: analyze then transform

A common pattern is to analyze the data first, then use the results to configure a `TransformProcess`.

```java
// Step 1: analyze
DataAnalysis analysis = AnalyzeLocal.analyze(schema, rr, 10);

// Step 2: build a transform process that uses the analysis
//         e.g., to normalize using discovered min/max values
TransformProcess tp = new TransformProcess.Builder(schema)
    .normalize("Score", Normalize.MinMax, analysis)
    .build();

// Step 3: execute
rr.reset();
List<List<Writable>> processed = LocalTransformExecutor.execute(rr, tp);
```

The `normalize` builder method accepts a `DataAnalysis` directly, extracting the per-column statistics needed to apply min-max or standardize normalization inline within a transform pipeline.

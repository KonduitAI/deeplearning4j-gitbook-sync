---
title: "DataVec Overview"
description: "DataVec ETL framework — purpose, architecture, and the data pipeline from raw data to DataSet"
---

# DataVec Overview

DataVec is the data transformation and vectorization library for the Eclipse Deeplearning4j ecosystem. It solves one of the most common obstacles in applied machine learning: getting raw data into the format that neural networks expect. Neural networks consume vectors and tensors, but raw data comes as CSV files, images on disk, log lines, JSON documents, and dozens of other formats. DataVec provides the tooling to bridge that gap.

The name reflects its mission: DataVec = Data Vectorization.

## What DataVec Does

DataVec handles the Extract, Transform, Load (ETL) phase of a machine learning workflow:

- **Extract**: Read data from files, directories, in-memory collections, or distributed storage using `RecordReader` implementations.
- **Transform**: Apply an ordered sequence of operations — type conversions, column manipulations, categorical encoding, filtering, normalization — via `TransformProcess`.
- **Load**: Deliver the processed data as `DataSet` objects to DL4J model training via `DataSetIterator`.

DataVec also integrates with Apache Spark, so the same transform definitions can run locally on a developer laptop or distributed across a cluster without code changes.

## When to Use DataVec

Use DataVec when:

- Your data is in CSV, TSV, JSON, XML, or other structured text formats
- Your data is a labeled image directory and you need to feed images into a CNN
- You need to convert categorical string columns to one-hot or integer representations
- You need to filter out bad records, normalize numeric columns, or parse timestamps
- You want a reusable, serializable transformation pipeline that can run both offline and in production inference

You may not need DataVec if:

- Your data is already in a numeric NDArray format that maps directly to your model inputs
- You are only loading simple pre-formatted datasets (e.g., MNIST via the built-in fetcher)

## Core Pipeline

The standard DataVec pipeline has four stages:

```
Raw Data on Disk
      |
  InputSplit          <-- defines which files or records to load
      |
  RecordReader        <-- parses raw bytes into List<Writable> records
      |
  TransformProcess    <-- applies ordered transforms to each record
      |
  DataSetIterator     <-- batches records into DataSet for DL4J training
```

### Stage 1: InputSplit

An `InputSplit` tells the `RecordReader` where the data lives. Common splits:

- `FileSplit(File rootDir)` — all files under a directory, recursively
- `FileSplit(File rootDir, String[] allowedExtensions, Random rng)` — filtered by extension
- `NumberedFileInputSplit(String basePattern, int minIdx, int maxIdx)` — for numbered files like `record_0001.csv` through `record_9999.csv`
- `CollectionInputSplit(List<URI> uris)` — from an arbitrary list of URIs
- `InputStreamInputSplit(InputStream is)` — from any input stream

### Stage 2: RecordReader

A `RecordReader` iterates over the `InputSplit` and converts each unit of data (a line, a file, a JSON object) into a `List<Writable>`. Each `Writable` in the list corresponds to one column.

```java
// Initialize reader on a directory of CSV files
RecordReader reader = new CSVRecordReader(1, ','); // skip 1 header line
reader.initialize(new FileSplit(new File("/data/train/")));

// Iterate manually
while (reader.hasNext()) {
    List<Writable> record = reader.next();
    // process record...
}
```

DataVec ships with readers for CSV, JSON/XML/YAML, images, log lines, audio, LibSVM, and more. See [Record Readers](readers.md) for the full list.

### Stage 3: TransformProcess

A `TransformProcess` is an ordered list of operations applied to each record, defined against a `Schema` that describes the layout of the input data.

```java
Schema schema = new Schema.Builder()
    .addColumnString("timestamp")
    .addColumnDouble("temperature")
    .addColumnCategorical("sensor", Arrays.asList("A", "B", "C"))
    .build();

TransformProcess tp = new TransformProcess.Builder(schema)
    .stringToTimeTransform("timestamp", "YYYY-MM-DD HH:mm:ss", DateTimeZone.UTC)
    .renameColumn("timestamp", "time")
    .doubleMathOp("temperature", MathOp.Subtract, 273.15)   // K to C
    .categoricalToOneHot("sensor")
    .build();
```

The transform process validates each operation against the schema at build time, so errors (referencing a non-existent column, applying a numeric op to a String column, etc.) are caught before any data is processed.

### Stage 4: DataSetIterator

Once you have a reader and optionally a transform process, wrap them in a `RecordReaderDataSetIterator` to produce `DataSet` objects that DL4J can train on directly.

```java
// Apply transform inline via TransformProcessRecordReader
RecordReader transformedReader = new TransformProcessRecordReader(reader, tp);

// labelIndex = column index of the label, numClasses = number of label classes
DataSetIterator iterator = new RecordReaderDataSetIterator(
    transformedReader,
    batchSize,
    labelIndex,
    numClasses
);

// Use with DL4J model
model.fit(iterator);
```

## Supported Data Formats

DataVec has built-in support for:

| Format | RecordReader Class |
|---|---|
| CSV / TSV | `CSVRecordReader` |
| CSV sequences (one file per sequence) | `CSVSequenceRecordReader` |
| JSON, XML, YAML | `JacksonRecordReader` |
| Log lines (regex parsing) | `RegexLineRecordReader` |
| Raw text lines | `LineRecordReader` |
| Labeled images (directory structure) | `ImageRecordReader` |
| LibSVM sparse format | `LibSvmRecordReader` |
| SVMLight format | `SVMLightRecordReader` |
| MATLAB .mat files | `MatlabRecordReader` |
| Apache Arrow columnar | `ArrowRecordReader` |
| WAV audio | `WavFileRecordReader` |
| TF-IDF vectors | `TfidfRecordReader` |
| In-memory collections | `CollectionRecordReader` |

## Architecture

DataVec is organized into several Maven modules:

- `datavec-api` — core interfaces: `RecordReader`, `Writable`, `Schema`, `TransformProcess`, `Filter`, `Condition`
- `datavec-local` — local (non-Spark) executors: `LocalTransformExecutor`, `AnalyzeLocal`
- `datavec-spark` — Spark executors: `SparkTransformExecutor`, `AnalyzeSpark`
- `datavec-data-image` — image readers: `ImageRecordReader`, `NativeImageLoader`
- `datavec-data-audio` — audio readers
- `datavec-data-nlp` — NLP readers including TF-IDF
- `datavec-arrow` — Apache Arrow integration

## Data Types

DataVec uses a typed column model. Every column in a `Schema` has a `ColumnType`:

- `Integer` — 32-bit signed integer
- `Long` — 64-bit signed integer
- `Double` — 64-bit floating point
- `Float` — 32-bit floating point
- `String` — arbitrary text
- `Categorical` — a fixed set of string labels (like an enum)
- `Time` — stored as epoch milliseconds (Long), but carries timezone info
- `Bytes` — raw byte array
- `NDArray` — an embedded multidimensional array
- `Boolean` — true/false

At the record level, each column is stored as a `Writable` — a lightweight value holder. `IntWritable`, `DoubleWritable`, `Text`, `NDArrayWritable`, etc. implement this interface.

## A Complete Example

Here is a concise end-to-end example loading a CSV, applying transforms, and producing a `DataSet`:

```java
// 1. Define schema
Schema schema = new Schema.Builder()
    .addColumnsString("name", "city")
    .addColumnInteger("age")
    .addColumnDouble("income")
    .addColumnCategorical("label", Arrays.asList("low", "medium", "high"))
    .build();

// 2. Define transform process
TransformProcess tp = new TransformProcess.Builder(schema)
    .removeColumns("name", "city")                   // drop irrelevant columns
    .doubleMathOp("income", MathOp.Divide, 1000.0)  // scale income
    .categoricalToInteger("label")                   // convert to integer 0/1/2
    .build();

// 3. Initialize reader
RecordReader rr = new CSVRecordReader(1, ',');       // skip header
rr.initialize(new FileSplit(new File("data.csv")));

// 4. Apply transforms
RecordReader transformedRr = new TransformProcessRecordReader(rr, tp);

// 5. Create iterator: label is last column (index 2), 3 classes
DataSetIterator iter = new RecordReaderDataSetIterator(transformedRr, 32, 2, 3);

// 6. Normalize
NormalizerStandardize normalizer = new NormalizerStandardize();
normalizer.fit(iter);
iter.reset();
iter.setPreProcessor(normalizer);

// 7. Train
model.fit(iter);
```

## Relationship to Other DL4J Components

- **ND4J**: DataVec's output is eventually consumed as ND4J `INDArray` objects. `NDArrayWritable` bridges the two.
- **DL4J**: `RecordReaderDataSetIterator` (and related iterators) wrap DataVec readers to produce `DataSet` and `MultiDataSet` objects that DL4J `MultiLayerNetwork` and `ComputationGraph` consume.
- **SameDiff**: SameDiff training also accepts `DataSetIterator`, so DataVec pipelines work unchanged.
- **Spark**: `SparkTransformExecutor` lets you apply the same `TransformProcess` to a Spark `JavaRDD<List<Writable>>`.

## Further Reading

- [Schema](schema.md) — defining the structure of your data
- [Record Readers](readers.md) — reading different file formats
- [Transforms](transforms.md) — the full transform API
- [Normalization](normalization.md) — scaling and standardizing features
- [Executors](executors.md) — running transforms locally or on Spark
- [Image Data](image.md) — image-specific pipeline

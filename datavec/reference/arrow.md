---
title: "Apache Arrow"
description: "Apache Arrow integration in DataVec — ArrowRecordReader and zero-copy data exchange"
---

# Apache Arrow

Apache Arrow is an in-memory columnar data format designed for high-performance analytics. It defines a standard binary layout for batches of typed columnar data that enables zero-copy reads between systems — no serialization or deserialization overhead when passing data between a DataVec pipeline and an Arrow-aware consumer.

DataVec's Arrow integration lets you use Arrow-formatted data as input to record readers and as an exchange format for large batch operations.

## What Arrow Provides

Traditional row-oriented formats (like Java `ArrayList<Map<String,Object>>`) scatter each record's fields across memory. Arrow lays out each column's values contiguously in memory:

```
Row-oriented:    [row0_col0, row0_col1, row0_col2], [row1_col0, row1_col1, row1_col2], ...
Column-oriented: [row0_col0, row1_col0, row2_col0, ...], [row0_col1, row1_col1, row2_col1, ...]
```

Columnar layout enables:

- **Vectorized computation**: apply an operation to an entire column using SIMD CPU instructions
- **Better compression**: repeated values and similar values are adjacent, compressing well
- **Zero-copy sharing**: share data between JVM and native code (Python, C++) without copying
- **Selective column reads**: read only the columns you need without deserializing the rest

Arrow is used by Apache Spark (Spark's vectorized execution engine uses Arrow internally), Pandas (via PyArrow), Apache Parquet (Arrow as the in-memory representation after Parquet deserialization), and many other data tools.

## ArrowRecordReader

`ArrowRecordReader` reads Arrow-formatted data and produces `List<Writable>` records compatible with the rest of the DataVec pipeline.

```java
import org.datavec.arrow.recordreader.ArrowRecordReader;

ArrowRecordReader reader = new ArrowRecordReader();
reader.initialize(new FileSplit(new File("/data/batch.arrow")));

while (reader.hasNext()) {
    List<Writable> record = reader.next();
    // use the record
}
```

Arrow files can be created by Python (via PyArrow), Spark (via `df.write.format("arrow")`), or any Arrow IPC writer. The Arrow IPC file format (sometimes called "Feather v2") is the standard format for on-disk Arrow data.

## ArrowWritableRecordBatch

`ArrowWritableRecordBatch` is a `List<List<Writable>>` backed directly by Arrow columnar memory. It provides zero-copy access to Arrow batches — individual `Writable` values are read directly from the Arrow buffer without copying into Java heap objects.

```java
import org.datavec.arrow.recordreader.ArrowWritableRecordBatch;

// Read a full Arrow batch
ArrowWritableRecordBatch batch = reader.next(batchSize);

// Access individual records
List<Writable> record = batch.get(0);
double value = record.get(2).toDouble();
```

This is particularly useful when processing large batches where copying each value would be expensive. The batch holds a reference to the underlying Arrow buffers; individual `Writable` objects returned from `batch.get(i).get(j)` read directly from those buffers.

## Converting Between Arrow and DataVec

### From Arrow VectorSchemaRoot to List<List<Writable>>

```java
import org.datavec.arrow.ArrowConverter;
import org.apache.arrow.vector.VectorSchemaRoot;

// Given an Arrow VectorSchemaRoot (a batch of columnar data)
VectorSchemaRoot root = ...; // loaded from Arrow IPC reader

Schema datavecSchema = ArrowConverter.toDatavecSchema(root.getSchema());
List<List<Writable>> records = ArrowConverter.toArrowWritables(
    ArrowConverter.toArrowColumns(bufferAllocator, datavecSchema, root),
    datavecSchema
);
```

### From List<List<Writable>> to Arrow

```java
import org.datavec.arrow.ArrowConverter;
import org.apache.arrow.memory.BufferAllocator;
import org.apache.arrow.memory.RootAllocator;

BufferAllocator allocator = new RootAllocator(Long.MAX_VALUE);

List<List<Writable>> records = ...; // your DataVec records
Schema schema = ...;                // matching DataVec schema

// Convert to Arrow columnar format
List<FieldVector> arrowVectors = ArrowConverter.toArrowColumns(
    allocator, schema, records);
```

This conversion enables sending DataVec-processed data to Arrow-compatible consumers (Python via IPC, Spark, etc.) without an intermediate CSV or JSON step.

## Use Cases

### High-Performance Batch Loading

When loading large pre-processed datasets for repeated training epochs, Arrow's columnar format allows reading an entire column in a single contiguous memory region, which is much faster than reading individual records from CSV.

```java
// Pre-process once, save as Arrow
List<List<Writable>> processed = LocalTransformExecutor.execute(reader, tp);
ArrowConverter.writeArrowFile(processed, schema, new File("processed.arrow"));

// Training: reload as Arrow for each epoch
ArrowRecordReader arrowReader = new ArrowRecordReader();
arrowReader.initialize(new FileSplit(new File("processed.arrow")));
DataSetIterator iter = new RecordReaderDataSetIterator(arrowReader, 256, labelIdx, numClasses);
```

### Python-Java Interoperability

If your data preparation runs in Python (feature engineering in Pandas, image preprocessing, NLP tokenization), you can write Arrow format from Python and read it in DataVec without going through CSV:

**Python side:**

```python
import pandas as pd
import pyarrow as pa

df = pd.DataFrame({"feature1": [1.0, 2.0], "label": [0, 1]})
table = pa.Table.from_pandas(df)

import pyarrow.ipc as ipc
with ipc.new_file("data.arrow", table.schema) as writer:
    writer.write(table)
```

**Java side:**

```java
ArrowRecordReader reader = new ArrowRecordReader();
reader.initialize(new FileSplit(new File("data.arrow")));
```

### Spark to DataVec

Spark's Arrow-based Pandas UDFs and the `spark.createDataFrame` path can produce Arrow IPC files. These can be read directly by `ArrowRecordReader` for local fine-tuning or evaluation after Spark-based feature engineering.

## Type Mapping

Arrow types map to DataVec `Writable` types as follows:

| Arrow Type | DataVec Writable |
|---|---|
| Int8, Int16, Int32 | `IntWritable` |
| Int64 | `LongWritable` |
| Float32 | `FloatWritable` |
| Float64 | `DoubleWritable` |
| Utf8 (string) | `Text` |
| Bool | `BooleanWritable` |
| FixedSizeBinary, Binary | `BytesWritable` |
| FixedSizeList / Tensor | `NDArrayWritable` |
| Null | `NullWritable` |

## Memory Management

Arrow allocates memory outside the JVM heap using `BufferAllocator`. Always close Arrow resources when done to avoid native memory leaks:

```java
BufferAllocator allocator = new RootAllocator(Long.MAX_VALUE);

try (VectorSchemaRoot root = VectorSchemaRoot.create(schema, allocator)) {
    // use root
} // automatically closed; memory returned to allocator

allocator.close();
```

`ArrowRecordReader` manages its own allocator internally and releases memory when `close()` is called. Always call `reader.close()` (or use try-with-resources) when done reading.

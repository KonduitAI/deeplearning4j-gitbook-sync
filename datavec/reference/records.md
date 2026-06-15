---
title: "Records and Writables"
description: "DataVec record format — Writable types, Record, and the data representation layer"
---

# Records and Writables

At the lowest level of DataVec, every data element is a `Writable`. A `Writable` is a lightweight, type-safe value holder — the atomic unit of data in DataVec. A **record** is a `List<Writable>` where each element corresponds to one column in the schema.

Understanding `Writable` is useful when writing custom transforms, implementing custom record readers, or manually iterating records to feed into inference.

## The Writable Interface

`Writable` provides a minimal interface for reading and converting values:

```java
public interface Writable {
    void write(DataOutput out) throws IOException;
    void readFields(DataInput in) throws IOException;

    double toDouble();
    float toFloat();
    int toInt();
    long toLong();
    String toString();     // the human-readable string form of the value

    WritableType getType();
}
```

Every column value in a DataVec record is one of the concrete `Writable` implementations listed below.

## Concrete Writable Types

### IntWritable

Holds a 32-bit signed integer.

```java
import org.datavec.api.writable.IntWritable;

Writable w = new IntWritable(42);
int value = w.toInt();        // 42
double d = w.toDouble();      // 42.0
String s = w.toString();      // "42"
```

### LongWritable

Holds a 64-bit signed integer. Used for large IDs, timestamps (epoch milliseconds), and any integer that may exceed `Integer.MAX_VALUE`.

```java
import org.datavec.api.writable.LongWritable;

Writable w = new LongWritable(System.currentTimeMillis());
long value = w.toLong();
```

### DoubleWritable

Holds a 64-bit double-precision floating point value.

```java
import org.datavec.api.writable.DoubleWritable;

Writable w = new DoubleWritable(3.14159);
double value = w.toDouble();
```

### FloatWritable

Holds a 32-bit single-precision floating point value. Preferred for memory-sensitive applications.

```java
import org.datavec.api.writable.FloatWritable;

Writable w = new FloatWritable(2.718f);
float value = w.toFloat();
```

### Text

Holds a string value. This is the default type produced by `CSVRecordReader` and `LineRecordReader` before any type conversion.

```java
import org.datavec.api.writable.Text;

Writable w = new Text("hello world");
String value = w.toString();
```

Note that `Text` is modeled after Hadoop's `Text` class but is not the same class. Do not confuse the two.

### BooleanWritable

Holds a boolean value.

```java
import org.datavec.api.writable.BooleanWritable;

Writable w = new BooleanWritable(true);
boolean value = Boolean.parseBoolean(w.toString());
int asInt = w.toInt();    // 1 for true, 0 for false
```

### BytesWritable

Holds a raw byte array. Used for binary data, serialized objects, or any blob-style column.

```java
import org.datavec.api.writable.BytesWritable;

byte[] data = {0x01, 0x02, 0x03};
Writable w = new BytesWritable(data);
```

### NDArrayWritable

Holds an ND4J `INDArray`. This is the bridge between DataVec's record model and ND4J's tensor model, used when a single column represents an entire feature vector or image.

```java
import org.datavec.api.writable.NDArrayWritable;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.ndarray.INDArray;

INDArray arr = Nd4j.create(new double[]{1.0, 2.0, 3.0, 4.0});
Writable w = new NDArrayWritable(arr);

INDArray retrieved = ((NDArrayWritable) w).get();
```

`NDArrayWritable` is used by `ImageRecordReader` to represent image data, and by `ArrowRecordReader` when reading columnar Arrow batches.

### NullWritable

Represents a missing or null value. Used as a placeholder when a join produces rows that have no matching record from the other side.

```java
import org.datavec.api.writable.NullWritable;

Writable w = NullWritable.INSTANCE;
boolean isNull = (w instanceof NullWritable);
```

## Record

A `Record` combines a `List<Writable>` with optional `RecordMetaData`. The metadata tracks where the record came from (which file, which line number, which URI), enabling error reporting and selective record reloading.

```java
import org.datavec.api.records.impl.Record;
import org.datavec.api.records.metadata.RecordMetaData;

// Read a record with metadata
Record record = reader.nextRecord();
List<Writable> values = record.getRecord();
RecordMetaData meta = record.getMetaData();

// Reload a specific record by its metadata (for error diagnosis)
Record reloaded = reader.loadFromMetaData(meta);
```

## SequenceRecord

A `SequenceRecord` wraps a sequence (`List<List<Writable>>`) with optional `RecordMetaData`. The outer list is the sequence of time steps; each inner list is one time step's column values.

```java
import org.datavec.api.records.impl.SequenceRecord;

SequenceRecord seqRecord = sequenceReader.nextSequence();
List<List<Writable>> sequence = seqRecord.getSequenceRecord();

// Access time step 0
List<Writable> firstStep = sequence.get(0);
double sensorValue = firstStep.get(1).toDouble();
```

## Working with Records Manually

Most DataVec pipelines do not require direct `Writable` manipulation — `TransformProcess`, `DataSetIterator`, and `LocalTransformExecutor` handle it internally. The cases where you interact with `Writable` directly are:

**Custom RecordReaders**: Implement `RecordReader.next()` by constructing a `List<Writable>`:

```java
@Override
public List<Writable> next() {
    List<Writable> record = new ArrayList<>();
    record.add(new LongWritable(nextId++));
    record.add(new DoubleWritable(parseValue(rawLine)));
    record.add(new Text(parseLabel(rawLine)));
    return record;
}
```

**Custom Transforms**: The `map` method of a custom `Transform` receives and returns `List<Writable>`:

```java
@Override
public List<Writable> map(List<Writable> writables) {
    List<Writable> out = new ArrayList<>(writables);
    double original = writables.get(2).toDouble();
    out.set(2, new DoubleWritable(Math.log1p(original)));
    return out;
}
```

**Manual Inference**: Building a record from a single input for real-time prediction:

```java
List<Writable> singleRecord = Arrays.asList(
    new DoubleWritable(age),
    new DoubleWritable(income),
    new Text(country)
);

// Apply the same TransformProcess used during training
List<Writable> processed = transformProcess.execute(singleRecord);

// Convert to INDArray for model input
DataSet ds = new RecordReaderDataSetIterator(
    new CollectionRecordReader(Collections.singletonList(processed)),
    1, -1, -1
).next();

INDArray prediction = model.output(ds.getFeatures());
```

## WritableType Enum

The `WritableType` enum identifies the type of a `Writable` without needing `instanceof` checks:

```java
Writable w = record.get(0);

switch (w.getType()) {
    case Int:     int i = w.toInt(); break;
    case Long:    long l = w.toLong(); break;
    case Double:  double d = w.toDouble(); break;
    case Float:   float f = w.toFloat(); break;
    case Bytes:   byte[] b = ((BytesWritable)w).getContent(); break;
    case NDArray: INDArray a = ((NDArrayWritable)w).get(); break;
    case Null:    // handle missing value
    case Boolean: boolean b2 = Boolean.parseBoolean(w.toString()); break;
}
```

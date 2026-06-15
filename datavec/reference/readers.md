---
title: "Record Readers"
description: "RecordReader implementations — CSV, JSON, image, regex, line, and custom readers"
---

# Record Readers

A `RecordReader` is the entry point for data into DataVec. It reads raw bytes from an `InputSplit` and converts them into `List<Writable>` records — one list per data example, where each element corresponds to a column in your `Schema`.

## The RecordReader Interface

Every reader implements `RecordReader` and provides:

| Method | Description |
|---|---|
| `initialize(InputSplit split)` | Set up the reader against a data source |
| `initialize(Configuration conf, InputSplit split)` | Set up with additional configuration |
| `hasNext()` | True if another record is available |
| `next()` | Return the next record as `List<Writable>` |
| `nextRecord()` | Return the next `Record` with optional `RecordMetaData` |
| `reset()` | Restart iteration from the beginning |
| `close()` | Release resources |

After calling `initialize`, use `hasNext` / `next` in a loop, or pass the reader directly to a `DataSetIterator`.

## InputSplit

An `InputSplit` tells the reader where to find data. The main implementations:

### FileSplit

Points to a directory or single file. By default, all files recursively under the directory are included.

```java
// All files under a directory
InputSplit split = new FileSplit(new File("/data/train/"));

// Only CSV files, shuffled
InputSplit split = new FileSplit(
    new File("/data/train/"),
    new String[]{"csv"},
    new Random(42)
);

// A single file
InputSplit split = new FileSplit(new File("/data/train.csv"));
```

### NumberedFileInputSplit

For files named with sequential numbers in a format string:

```java
// matches seq_0000.csv through seq_9999.csv
InputSplit split = new NumberedFileInputSplit("/data/seq_%04d.csv", 0, 9999);
```

### CollectionInputSplit

For an explicit list of URIs:

```java
List<URI> uris = Arrays.asList(
    new URI("file:///data/a.csv"),
    new URI("file:///data/b.csv")
);
InputSplit split = new CollectionInputSplit(uris);
```

### InputStreamInputSplit

For streaming data from any `InputStream`:

```java
InputStream is = getClass().getResourceAsStream("/data.csv");
InputSplit split = new InputStreamInputSplit(is);
```

## CSV Readers

### CSVRecordReader

The most commonly used reader. Reads a CSV (or TSV, or any delimiter-separated) file line by line, producing one `List<Writable>` per line.

```java
// Default: comma delimiter, no header skip
RecordReader rr = new CSVRecordReader();
rr.initialize(new FileSplit(new File("data.csv")));

// Skip 1 header line, comma delimiter
RecordReader rr = new CSVRecordReader(1, ',');
rr.initialize(new FileSplit(new File("data.csv")));

// Tab delimiter
RecordReader rr = new CSVRecordReader(0, '\t');
```

All values are returned as `Text` (string) `Writable` objects. Numeric conversion happens automatically in the `TransformProcess` or during `DataSetIterator` construction.

When your CSV has mixed quoted fields:

```java
// Handle quoted fields with embedded commas
RecordReader rr = new CSVRecordReader(1, ',', '"');
```

### CSVSequenceRecordReader

Reads multiple files, treating each file as one sequence. Each line in a file is one time step; each value in a line is one feature at that time step.

This reader implements `SequenceRecordReader`, so use it with `SequenceRecordReaderDataSetIterator`.

```java
// One CSV file per sequence, one line per time step
SequenceRecordReader features = new CSVSequenceRecordReader(1, ',');
features.initialize(new NumberedFileInputSplit("/data/features_%d.csv", 0, 999));

SequenceRecordReader labels = new CSVSequenceRecordReader(1, ',');
labels.initialize(new NumberedFileInputSplit("/data/labels_%d.csv", 0, 999));

DataSetIterator iter = new SequenceRecordReaderDataSetIterator(
    features, labels, batchSize, numClasses,
    false,   // not regression
    SequenceRecordReaderDataSetIterator.AlignmentMode.ALIGN_END
);
```

### CSVRegexRecordReader

Splits columns using regex patterns rather than a simple delimiter. Useful for CSV files with inconsistent spacing or mixed delimiters.

```java
RecordReader rr = new CSVRegexRecordReader(0, ',');
```

### CSVVariableSlidingWindowRecordReader

Reads an entire CSV and produces subsequences using a variable sliding window. The window starts at size 1, grows to `maxLinesPerSequence`, then shrinks back. Useful for training on all possible subsequences of a dataset.

## Text Readers

### LineRecordReader

Reads a file line by line. Each line becomes a single-element record containing a `Text` writable. No parsing is done — you receive the raw line. Useful when you want to apply your own parsing in a `TransformProcess` or custom transform.

```java
RecordReader rr = new LineRecordReader();
rr.initialize(new FileSplit(new File("/data/corpus.txt")));

while (rr.hasNext()) {
    List<Writable> line = rr.next();  // single-element list
    String text = line.get(0).toString();
}
```

### RegexLineRecordReader

Reads a file line by line and splits each line into fields using a regex with capture groups. Each capture group becomes one `Text` writable in the record.

```java
// Parse log lines: "2024-01-15 14:32:01.123 42 WARN Message text here"
String regex = "(\\d{4}-\\d{2}-\\d{2} \\d{2}:\\d{2}:\\d{2}\\.\\d{3}) (\\d+) ([A-Z]+) (.+)";
int skipLines = 0;

RecordReader rr = new RegexLineRecordReader(regex, skipLines);
rr.initialize(new FileSplit(new File("/var/log/app.log")));

// Each record: ["2024-01-15 14:32:01.123", "42", "WARN", "Message text here"]
```

Lines that do not match the regex result in an exception by default.

### RegexSequenceRecordReader

Like `RegexLineRecordReader`, but reads an entire file as a sequence, with one time step per line. Supports three invalid-line handling modes:

- `FailOnInvalid` — throw an exception (default)
- `SkipInvalid` — silently skip non-matching lines
- `SkipInvalidWithWarning` — skip but log a warning

```java
RecordReader rr = new RegexSequenceRecordReader(regex, skipLines,
    RegexSequenceRecordReader.LineErrorHandling.SkipInvalidWithWarning);
```

### ListStringRecordReader

Reads from an in-memory list of strings. Each string is parsed as a single-column record. Useful for testing or when you have already loaded text into memory.

```java
List<List<String>> data = Arrays.asList(
    Arrays.asList("cat"),
    Arrays.asList("dog"),
    Arrays.asList("bird")
);
InputSplit split = new ListStringSplit(data);
RecordReader rr = new ListStringRecordReader();
rr.initialize(split);
```

## JSON / XML / YAML Readers

### JacksonRecordReader

Reads JSON, XML, or YAML files using Jackson. Each file (or each element in an array) becomes one record. You specify a `FieldSelection` to pull out the fields you need.

```java
import org.datavec.api.records.reader.impl.jackson.JacksonRecordReader;
import org.datavec.api.records.reader.impl.jackson.FieldSelection;
import com.fasterxml.jackson.databind.ObjectMapper;

FieldSelection fields = new FieldSelection.Builder()
    .addField("userId")
    .addField("amount")
    .addField("category")
    .build();

RecordReader rr = new JacksonRecordReader(
    fields,
    new ObjectMapper(),      // ObjectMapper for JSON
    false,                   // not append label
    -1,                      // label index (not used here)
    new FileSplit(new File("/data/events/"))
);
```

For XML, replace `new ObjectMapper()` with `new XmlMapper()` from the Jackson XML module.

## Image Reader

### ImageRecordReader

Reads a directory of images, where each subdirectory is treated as a class label (one-of-K labeling). All images are resized to the specified height, width, and channel count.

```java
import org.datavec.image.recordreader.ImageRecordReader;
import org.datavec.image.transform.ImageTransform;

int height = 224;
int width = 224;
int channels = 3;  // RGB; use 1 for grayscale

// Construct a label generator from directory names
ParentPathLabelGenerator labelMaker = new ParentPathLabelGenerator();

ImageRecordReader rr = new ImageRecordReader(height, width, channels, labelMaker);
rr.initialize(new FileSplit(new File("/data/images/train/")));
```

Expected directory structure:
```
/data/images/train/
    cat/
        img001.jpg
        img002.jpg
    dog/
        img003.jpg
        img004.jpg
```

With this structure, images in `cat/` get label index 0 and images in `dog/` get label index 1 (alphabetical ordering).

For image augmentation and transforms, see [Image Data](image.md).

## File Reader

### FileRecordReader

Reads individual files, returning the file path as a `Text` writable and the label derived from the parent directory name. Most commonly used as a base class rather than directly.

```java
RecordReader rr = new FileRecordReader();
rr.initialize(new FileSplit(new File("/data/files/")));

int currentLabel = ((FileRecordReader) rr).getCurrentLabel();
```

## Sparse Format Readers

### LibSvmRecordReader and SVMLightRecordReader

These readers parse sparse feature formats widely used in linear model and kernel method communities. The format encodes each example as:

```
LABEL index1:value1 index2:value2 ...
```

Zero-valued features are omitted. `LibSvmRecordReader` is a subclass of `SVMLightRecordReader` with minor format differences.

```java
// Configure for specific number of features
Configuration conf = new Configuration();
conf.set(SVMLightRecordReader.NUM_FEATURES, "10000");
conf.setBoolean(SVMLightRecordReader.ZERO_BASED_INDEXING, false);

RecordReader rr = new LibSvmRecordReader();
rr.initialize(conf, new FileSplit(new File("data.svm")));
```

## Collection Readers

### CollectionRecordReader

Wraps an in-memory `List<List<Writable>>` as a reader. Primarily used in unit tests.

```java
List<List<Writable>> data = new ArrayList<>();
data.add(Arrays.asList(new IntWritable(1), new DoubleWritable(3.14)));
data.add(Arrays.asList(new IntWritable(2), new DoubleWritable(2.72)));

RecordReader rr = new CollectionRecordReader(data);
```

### CollectionSequenceRecordReader

Like `CollectionRecordReader` but for sequence data: wraps `List<List<List<Writable>>>`.

## Combining Readers

### ConcatenatingRecordReader

Chains multiple readers sequentially. When the first reader is exhausted, reading continues with the second, and so on. Useful for combining training files across multiple directories.

```java
RecordReader r1 = new CSVRecordReader(1, ',');
r1.initialize(new FileSplit(new File("/data/train_2022/")));

RecordReader r2 = new CSVRecordReader(1, ',');
r2.initialize(new FileSplit(new File("/data/train_2023/")));

RecordReader combined = new ConcatenatingRecordReader(r1, r2);
```

### TransformProcessRecordReader

Wraps another reader and applies a `TransformProcess` to every record before returning it. Useful when you want to inline transformation without a separate executor step.

```java
RecordReader base = new CSVRecordReader(1, ',');
base.initialize(new FileSplit(new File("data.csv")));

TransformProcess tp = new TransformProcess.Builder(schema)
    .removeColumns("id")
    .categoricalToOneHot("color")
    .build();

RecordReader transformed = new TransformProcessRecordReader(base, tp);
```

For sequence readers, use `TransformProcessSequenceRecordReader` instead.

## Adding Listeners

You can attach a `RecordListener` to any reader for debugging or monitoring:

```java
rr.addListener(new LogRecordListener());  // logs every record to SLF4J
```

Custom listeners implement the `RecordListener` interface:

```java
rr.addListener(new RecordListener() {
    @Override
    public void recordRead(RecordReader reader, Object record) {
        System.out.println("Read: " + record);
    }
});
```

## Choosing the Right Reader

| Your data | Use |
|---|---|
| CSV or TSV files | `CSVRecordReader` |
| One sequence per CSV file | `CSVSequenceRecordReader` |
| JSON / XML / YAML files | `JacksonRecordReader` |
| Log files with structured format | `RegexLineRecordReader` (single record per line) or `RegexSequenceRecordReader` (whole file as sequence) |
| Labeled image directories | `ImageRecordReader` |
| Sparse feature vectors | `LibSvmRecordReader` / `SVMLightRecordReader` |
| In-memory data (testing) | `CollectionRecordReader` |
| Multiple files to concatenate | `ConcatenatingRecordReader` |
| Any reader + inline transforms | `TransformProcessRecordReader` |

---
title: DataVec Readers
short_title: Readers
description: Read individual records from different formats.
category: DataVec
weight: 2
---

# Readers

## Why readers?

Readers iterate records from a dataset in storage and load the data into DataVec. The usefulness of readers beyond individual entries in a dataset includes: what if you wanted to train a text generator on a corpus? Or programmatically compose two entries together to form a new record? Reader implementations are useful for complex file types or distributed storage mechanisms.

Readers return `Writable` classes that describe each column in a `Record`. These classes are used to convert each record to a tensor/ND-Array format.

## Usage

Each reader implementation extends `BaseRecordReader` and provides a simple API for selecting the next record in a dataset, acting similarly to iterators.

Useful methods include:

* `next`: Return a batch of `Writable`.
* `nextRecord`: Return a single `Record`, optionally with `RecordMetaData`.
* `reset`: Reset the underlying iterator.
* `hasNext`: Iterator method to determine if another record is available.

## Listeners

You can hook a custom `RecordListener` to a record reader for debugging or visualization purposes. Pass your custom listener to the `addListener` base method immediately after initializing your class.

## Types of readers

### ComposableRecordReader

RecordReader for each pipeline. Individual record is a concatenation of the two collections. Create a recordreader that takes recordreaders and iterates over them and concatenates them hasNext would be the & of all the recordreaders concatenation would be next & addAll on the collection return one record

**initialize**

```text
public void initialize(InputSplit split) throws IOException, InterruptedException
```

### ConcatenatingRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/ConcatenatingRecordReader.java)

Combine multiple readers into a single reader. Records are read sequentially - thus if the first reader has 100 records, and the second reader has 200 records, ConcatenatingRecordReader will have 300 records.

### FileRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/FileRecordReader.java)

File reader/writer

**getCurrentLabel**

```text
public int getCurrentLabel()
```

Return the current label. The index of the current file’s parent directory in the label list

* return The index of the current file’s parent directory

### LineRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/LineRecordReader.java)

Reads files line by line

### CollectionRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/collection/CollectionRecordReader.java)

Collection record reader. Mainly used for testing.

### CollectionSequenceRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/collection/CollectionSequenceRecordReader.java)

Collection record reader for sequences. Mainly used for testing.

**initialize**

```text
public void initialize(InputSplit split) throws IOException, InterruptedException
```

* param records Collection of sequences. For example, List&lt;List&lt;List&gt;&gt; where the inner two lists are a sequence, and the outer list/collection is a list of sequences

### ListStringRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/collection/ListStringRecordReader.java)

Iterates through a list of strings return a record.

**initialize**

```text
public void initialize(InputSplit split) throws IOException, InterruptedException
```

Called once at initialization.

* param split the split that defines the range of records to read
* throws IOException
* throws InterruptedException

**initialize**

```text
public void initialize(Configuration conf, InputSplit split) throws IOException, InterruptedException
```

Called once at initialization.

* param conf a configuration for initialization
* param split the split that defines the range of records to read
* throws IOException
* throws InterruptedException

**hasNext**

```text
public boolean hasNext()
```

Get the next record

* return The list of next record

**reset**

```text
public void reset()
```

List of label strings

* return

**nextRecord**

```text
public Record nextRecord()
```

Load the record from the given DataInputStream Unlike {- link \#next\(\)} the internal state of the RecordReader is not modified Implementations of this method should not close the DataInputStream

* param uri
* param dataInputStream
* throws IOException if error occurs during reading from the input stream

**close**

```text
public void close() throws IOException
```

Closes this stream and releases any system resources associated with it. If the stream is already closed then invoking this method has no effect.

As noted in {- link AutoCloseable\#close\(\)}, cases where the close may fail require careful attention. It is strongly advised to relinquish the underlying resources and to internally _mark_ the {- code Closeable} as closed, prior to throwing the {- code IOException}.

* throws IOException if an I/O error occurs

**setConf**

```text
public void setConf(Configuration conf)
```

Set the configuration to be used by this object.

* param conf

**getConf**

```text
public Configuration getConf()
```

Return the configuration used by this object.

### CSVRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/csv/CSVRecordReader.java)

Simple csv record reader.

**initialize**

```text
public void initialize(Configuration conf, InputSplit split) throws IOException, InterruptedException
```

Skip first n lines

* param skipNumLines the number of lines to skip

### CSVRegexRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/csv/CSVRegexRecordReader.java)

A CSVRecordReader that can split each column into additional columns using regexs.

### CSVSequenceRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/csv/CSVSequenceRecordReader.java)

CSV Sequence Record Reader This reader is intended to read sequences of data in CSV format, where each sequence is defined in its own file \(and there are multiple files\) Each line in the file represents one time step

### CSVVariableSlidingWindowRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/csv/CSVVariableSlidingWindowRecordReader.java)

A sliding window of variable size across an entire CSV.

In practice the sliding window size starts at 1, then linearly increase to maxLinesPer sequence, then linearly decrease back to 1.

**initialize**

```text
public void initialize(Configuration conf, InputSplit split) throws IOException, InterruptedException
```

No-arg constructor with the default number of lines per sequence \(10\)

### LibSvmRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/misc/LibSvmRecordReader.java)

Record reader for libsvm format, which is closely related to SVMLight format. Similar to scikit-learn we use a single reader for both formats, so this class is a subclass of SVMLightRecordReader.

Further details on the format can be found at

* [http://svmlight.joachims.org/](http://svmlight.joachims.org/) 
* [http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/multilabel.html](http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/multilabel.html) 
* [http://scikit-learn.org/stable/modules/generated/sklearn.datasets.load\_svmlight\_file.html](http://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_svmlight_file.html)

### MatlabRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/misc/MatlabRecordReader.java)

Matlab record reader

### SVMLightRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/misc/SVMLightRecordReader.java)

Record reader for SVMLight format, which can generally be described as

LABEL INDEX:VALUE INDEX:VALUE …

SVMLight format is well-suited to sparse data \(e.g., bag-of-words\) because it omits all features with value zero.

We support an “extended” version that allows for multiple targets \(or labels\) separated by a comma, as follows:

LABEL1,LABEL2,… INDEX:VALUE INDEX:VALUE …

This can be used to represent either multitask problems or multilabel problems with sparse binary labels \(controlled via the “MULTILABEL” configuration option\).

Like scikit-learn, we support both zero-based and one-based indexing.

Further details on the format can be found at

* [http://svmlight.joachims.org/](http://svmlight.joachims.org/) 
* [http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/multilabel.html](http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/multilabel.html) 
* [http://scikit-learn.org/stable/modules/generated/sklearn.datasets.load\_svmlight\_file.html](http://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_svmlight_file.html)

**initialize**

```text
public void initialize(Configuration conf, InputSplit split) throws IOException, InterruptedException
```

Must be called before attempting to read records.

* param conf DataVec configuration
* param split FileSplit
* throws IOException
* throws InterruptedException

**setConf**

```text
public void setConf(Configuration conf)
```

Set configuration.

* param conf DataVec configuration
* throws IOException
* throws InterruptedException

**hasNext**

```text
public boolean hasNext()
```

Helper function to help detect lines that are commented out. May read ahead and cache a line.

* return

**nextRecord**

```text
public Record nextRecord()
```

Return next record as list of Writables.

* return

### RegexLineRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/regex/RegexLineRecordReader.java)

RegexLineRecordReader: Read a file, one line at a time, and split it into fields using a regex. To load an entire file using a

Example: Data in format “2016-01-01 23:59:59.001 1 DEBUG First entry message!”  
using regex String “\(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}.\d{3}\) \(\d+\) \(\[A-Z\]+\) \(.\)”  
would be split into 4 Text writables: \[“2016-01-01 23:59:59.001”, “1”, “DEBUG”, “First entry message!”\]

### RegexSequenceRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/regex/RegexSequenceRecordReader.java)

RegexSequenceRecordReader: Read an entire file \(as a sequence\), one line at a time and split each line into fields using a regex.

Example: Data in format “2016-01-01 23:59:59.001 1 DEBUG First entry message!”  
using regex String “\(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}.\d{3}\) \(\d+\) \(\[A-Z\]+\) \(.\)”  
would be split into 4 Text writables: \[“2016-01-01 23:59:59.001”, “1”, “DEBUG”, “First entry message!”\]

lines that don’t match the provided regex can result in an exception \(FailOnInvalid\), can be skipped silently \(SkipInvalid\), or skip invalid but log a warning \(SkipInvalidWithWarning\)

### TransformProcessRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/transform/TransformProcessRecordReader.java)

to have a transform process applied before being returned.

**initialize**

```text
public void initialize(InputSplit split) throws IOException, InterruptedException
```

Called once at initialization.

* param split the split that defines the range of records to read
* throws IOException
* throws InterruptedException

**initialize**

```text
public void initialize(Configuration conf, InputSplit split) throws IOException, InterruptedException
```

Called once at initialization.

* param conf a configuration for initialization
* param split the split that defines the range of records to read
* throws IOException
* throws InterruptedException

**hasNext**

```text
public boolean hasNext()
```

Get the next record

* return

**reset**

```text
public void reset()
```

List of label strings

* return

**nextRecord**

```text
public Record nextRecord()
```

Load the record from the given DataInputStream Unlike {- link \#next\(\)} the internal state of the RecordReader is not modified Implementations of this method should not close the DataInputStream

* param uri
* param dataInputStream
* throws IOException if error occurs during reading from the input stream

**loadFromMetaData**

```text
public Record loadFromMetaData(RecordMetaData recordMetaData) throws IOException
```

Load a single record from the given {- link RecordMetaData} instance  
Note: that for data that isn’t splittable \(i.e., text data that needs to be scanned/split\), it is more efficient to load multiple records at once using {- link \#loadFromMetaData\(List\)}

* param recordMetaData Metadata for the record that we want to load from
* return Single record for the given RecordMetaData instance
* throws IOException If I/O error occurs during loading

**setListeners**

```text
public void setListeners(RecordListener... listeners)
```

Load multiple records from the given a list of {- link RecordMetaData} instances

* param recordMetaDatas Metadata for the records that we want to load from
* return Multiple records for the given RecordMetaData instances
* throws IOException If I/O error occurs during loading

**setListeners**

```text
public void setListeners(Collection<RecordListener> listeners)
```

Set the record listeners for this record reader.

* param listeners

**close**

```text
public void close() throws IOException
```

Closes this stream and releases any system resources associated with it. If the stream is already closed then invoking this method has no effect.

As noted in {- link AutoCloseable\#close\(\)}, cases where the close may fail require careful attention. It is strongly advised to relinquish the underlying resources and to internally _mark_ the {- code Closeable} as closed, prior to throwing the {- code IOException}.

* throws IOException if an I/O error occurs

**setConf**

```text
public void setConf(Configuration conf)
```

Set the configuration to be used by this object.

* param conf

**getConf**

```text
public Configuration getConf()
```

Return the configuration used by this object.

### TransformProcessSequenceRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/reader/impl/transform/TransformProcessSequenceRecordReader.java)

to be transformed before being returned.

**setConf**

```text
public void setConf(Configuration conf)
```

Set the configuration to be used by this object.

* param conf

**getConf**

```text
public Configuration getConf()
```

Return the configuration used by this object.

**batchesSupported**

```text
public boolean batchesSupported()
```

Returns a sequence record.

* return a sequence of records

**nextSequence**

```text
public SequenceRecord nextSequence()
```

Load a sequence record from the given DataInputStream Unlike {- link \#next\(\)} the internal state of the RecordReader is not modified Implementations of this method should not close the DataInputStream

* param uri
* param dataInputStream
* throws IOException if error occurs during reading from the input stream

**loadSequenceFromMetaData**

```text
public SequenceRecord loadSequenceFromMetaData(RecordMetaData recordMetaData) throws IOException
```

Load a single sequence record from the given {- link RecordMetaData} instance  
Note: that for data that isn’t splittable \(i.e., text data that needs to be scanned/split\), it is more efficient to load multiple records at once using {- link \#loadSequenceFromMetaData\(List\)}

* param recordMetaData Metadata for the sequence record that we want to load from
* return Single sequence record for the given RecordMetaData instance
* throws IOException If I/O error occurs during loading

**initialize**

```text
public void initialize(InputSplit split) throws IOException, InterruptedException
```

Load multiple sequence records from the given a list of {- link RecordMetaData} instances

* param recordMetaDatas Metadata for the records that we want to load from
* return Multiple sequence record for the given RecordMetaData instances
* throws IOException If I/O error occurs during loading

**initialize**

```text
public void initialize(Configuration conf, InputSplit split) throws IOException, InterruptedException
```

Called once at initialization.

* param conf a configuration for initialization
* param split the split that defines the range of records to read
* throws IOException
* throws InterruptedException

**hasNext**

```text
public boolean hasNext()
```

Get the next record

* return

**reset**

```text
public void reset()
```

List of label strings

* return

**nextRecord**

```text
public Record nextRecord()
```

Load the record from the given DataInputStream Unlike {- link \#next\(\)} the internal state of the RecordReader is not modified Implementations of this method should not close the DataInputStream

* param uri
* param dataInputStream
* throws IOException if error occurs during reading from the input stream

**loadFromMetaData**

```text
public Record loadFromMetaData(RecordMetaData recordMetaData) throws IOException
```

Load a single record from the given {- link RecordMetaData} instance  
Note: that for data that isn’t splittable \(i.e., text data that needs to be scanned/split\), it is more efficient to load multiple records at once using {- link \#loadFromMetaData\(List\)}

* param recordMetaData Metadata for the record that we want to load from
* return Single record for the given RecordMetaData instance
* throws IOException If I/O error occurs during loading

**setListeners**

```text
public void setListeners(RecordListener... listeners)
```

Load multiple records from the given a list of {- link RecordMetaData} instances

* param recordMetaDatas Metadata for the records that we want to load from
* return Multiple records for the given RecordMetaData instances
* throws IOException If I/O error occurs during loading

**setListeners**

```text
public void setListeners(Collection<RecordListener> listeners)
```

Set the record listeners for this record reader.

* param listeners

**close**

```text
public void close() throws IOException
```

Closes this stream and releases any system resources associated with it. If the stream is already closed then invoking this method has no effect.

As noted in {- link AutoCloseable\#close\(\)}, cases where the close may fail require careful attention. It is strongly advised to relinquish the underlying resources and to internally _mark_ the {- code Closeable} as closed, prior to throwing the {- code IOException}.

* throws IOException if an I/O error occurs

### NativeAudioRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-data/datavec-data-audio/src/main/java/org/datavec/audio/recordreader/NativeAudioRecordReader.java)

Native audio file loader using FFmpeg.

### WavFileRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-data/datavec-data-audio/src/main/java/org/datavec/audio/recordreader/WavFileRecordReader.java)

Wav file loader

### ImageRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-data/datavec-data-image/src/main/java/org/datavec/image/recordreader/ImageRecordReader.java)

Image record reader. Reads a local file system and parses images of a given height and width. All images are rescaled and converted to the given height, width, and number of channels.

Also appends the label if specified \(one of k encoding based on the directory structure where each subdir of the root is an indexed label\)

### TfidfRecordReader

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-data/datavec-data-nlp/src/main/java/org/datavec/nlp/reader/TfidfRecordReader.java)

TFIDF record reader \(wraps a tfidf vectorizer for delivering labels and conforming to the record reader interface\)


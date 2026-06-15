---
title: "Data Iterators"
description: "DataSetIterator implementations — built-in iterators, custom iterators, async loading, and data splitting"
---

# Data Iterators

A `DataSetIterator` is the primary interface for feeding training and evaluation data into `MultiLayerNetwork` and `ComputationGraph`. It produces `DataSet` objects (features + labels pairs) one minibatch at a time. Eclipse Deeplearning4j ships built-in iterators for common benchmark datasets, iterators that wrap DataVec `RecordReader` instances for custom data, and utility iterators for async prefetching and train/test splitting.

## The DataSetIterator Interface

`DataSetIterator` extends `java.util.Iterator<DataSet>` with additional methods:

```java
public interface DataSetIterator extends Iterator<DataSet>, Iterable<DataSet> {

    // Returns the next minibatch (use batch() to retrieve the configured batch size)
    DataSet next();

    // Returns the next minibatch with a specific size
    DataSet next(int num);

    // Number of input features per example
    int inputColumns();

    // Number of output labels per example
    int totalOutcomes();

    // Whether reset() is supported
    boolean resetSupported();

    // Whether async prefetching is safe to use with this iterator
    boolean asyncSupported();

    // Reset to the beginning of the dataset
    void reset();

    // Configured minibatch size
    int batch();

    // Optional preprocessor applied to each DataSet before it is returned
    void setPreProcessor(DataSetPreProcessor preProcessor);
    DataSetPreProcessor getPreProcessor();

    // List of label names (may return null)
    List<String> getLabels();

    boolean hasNext();
}
```

The simplest training loop:

```java
DataSetIterator trainIter = /* any iterator */;
MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();

for (int epoch = 0; epoch < numEpochs; epoch++) {
    model.fit(trainIter);      // internally calls next() in a loop then resets
    trainIter.reset();         // explicit reset if needed between epochs
}
```

---

## Built-in Dataset Iterators

### MnistDataSetIterator

60,000 training / 10,000 test grayscale digit images, 28x28 pixels, 10 classes. Data is automatically downloaded and cached on first use.

```java
import org.deeplearning4j.datasets.iterator.impl.MnistDataSetIterator;

int batchSize = 64;
boolean train = true;
int rngSeed   = 12345;

DataSetIterator mnistTrain = new MnistDataSetIterator(batchSize, train, rngSeed);
DataSetIterator mnistTest  = new MnistDataSetIterator(batchSize, false, rngSeed);
```

Output shape per batch: features `[batch, 784]`, labels `[batch, 10]` (one-hot).

### Cifar10DataSetIterator

50,000 training / 10,000 test RGB images, 32x32 pixels, 10 classes. Uses a cached PNG version of the dataset.

```java
import org.deeplearning4j.datasets.iterator.impl.Cifar10DataSetIterator;

// Training iterator, random order, RNG seed 123
DataSetIterator cifarTrain = new Cifar10DataSetIterator(batchSize);

// Test set
DataSetIterator cifarTest  = new Cifar10DataSetIterator(batchSize, false);
```

Output shape per batch: features `[batch, 3, 32, 32]` (channels-first), labels `[batch, 10]`.

### IrisDataSetIterator

150 examples, 4 features, 3 classes. The classic Fisher Iris dataset.

```java
import org.deeplearning4j.datasets.iterator.impl.IrisDataSetIterator;

// Load all 150 examples with batch size 150 (single batch)
DataSetIterator irisIter = new IrisDataSetIterator(150, 150);
DataSet iris = irisIter.next();
```

### EmnistDataSetIterator

Extended MNIST with multiple subset splits:

| Subset constant | Classes | Training examples |
|---|---|---|
| `COMPLETE` (ByClass) | 62 | ~697,932 |
| `MERGE` (ByMerge) | 47 | ~697,932 |
| `BALANCED` | 47 | 112,800 |
| `LETTERS` | 26 | 124,800 |
| `DIGITS` | 10 | 240,000 |

```java
import org.deeplearning4j.datasets.iterator.impl.EmnistDataSetIterator;
import org.deeplearning4j.datasets.iterator.impl.EmnistDataSetIterator.Set;

DataSetIterator emnistTrain = new EmnistDataSetIterator(Set.BALANCED, batchSize, true);
DataSetIterator emnistTest  = new EmnistDataSetIterator(Set.BALANCED, batchSize, false);

// Useful static helpers
int numLabels = EmnistDataSetIterator.numLabels(Set.BALANCED);    // 47
int numTrain  = EmnistDataSetIterator.numExamplesTrain(Set.BALANCED); // 112800
```

### UciSequenceDataSetIterator

Univariate time series dataset from UCI, 6 classes (Normal, Cyclic, Increasing Trend, Decreasing Trend, Upward Shift, Downward Shift). Useful for testing sequence classifiers.

```java
import org.deeplearning4j.datasets.iterator.impl.UciSequenceDataSetIterator;

DataSetIterator uciTrain = new UciSequenceDataSetIterator(batchSize);
```

### LFWDataSetIterator

Labeled Faces in the Wild: 13,233 images across 5,749 identity classes. Supports train/test split, custom image transforms, and label generation.

```java
import org.deeplearning4j.datasets.iterator.impl.LFWDataSetIterator;

int[] imageDimensions = {128, 128, 3};  // height, width, channels
double splitRatio = 0.8;               // 80% train

DataSetIterator lfwTrain = new LFWDataSetIterator(
    batchSize, numExamples, imageDimensions,
    numLabels, false,
    new ParentPathLabelGenerator(),
    true,         // train = true
    splitRatio,
    null,         // no image transform
    new Random(12345));
```

### TinyImageNetDataSetIterator

200-class subset of ImageNet, 500 training images per class, 64x64 RGB. Used in Stanford CS231n.

```java
import org.deeplearning4j.datasets.iterator.impl.TinyImageNetDataSetIterator;

DataSetIterator tinyTrain = new TinyImageNetDataSetIterator(batchSize);
DataSetIterator tinyTest  = new TinyImageNetDataSetIterator(batchSize, DataSetType.TEST);
```

---

## RecordReaderDataSetIterator

`RecordReaderDataSetIterator` bridges DataVec `RecordReader` instances (CSV, images, audio, etc.) with the DL4J training API. Use its fluent `Builder` for clean configuration.

### Classification from CSV

```java
import org.datavec.api.records.reader.impl.csv.CSVRecordReader;
import org.datavec.api.split.FileSplit;
import org.deeplearning4j.datasets.datavec.RecordReaderDataSetIterator;

CSVRecordReader rr = new CSVRecordReader(0, ',');  // 0 header lines, comma delimiter
rr.initialize(new FileSplit(new File("/path/to/data.csv")));

DataSetIterator iter = new RecordReaderDataSetIterator.Builder(rr, batchSize)
    // Column 4 contains the class index (0 to numClasses-1)
    .classification(4, numClasses)
    .build();
```

### Regression from CSV

```java
DataSetIterator iter = new RecordReaderDataSetIterator.Builder(rr, batchSize)
    // Single regression target in column 3
    .regression(3)
    .build();

// Multiple contiguous regression targets (columns 3 through 6)
DataSetIterator iter2 = new RecordReaderDataSetIterator.Builder(rr, batchSize)
    .regression(3, 6)
    .build();
```

### Image Classification

```java
import org.datavec.api.io.labels.ParentPathLabelGenerator;
import org.datavec.image.recordreader.ImageRecordReader;

ImageRecordReader imageRR = new ImageRecordReader(height, width, channels,
    new ParentPathLabelGenerator());
imageRR.initialize(new FileSplit(new File("/path/to/imageDir")));

DataSetIterator imgIter = new RecordReaderDataSetIterator.Builder(imageRR, batchSize)
    .classification(1, numClasses)          // label index is always 1 for ImageRecordReader
    .preProcessor(new ImagePreProcessingScaler(0, 1))  // scale pixels to [0, 1]
    .build();
```

### Builder Reference

| Method | Description |
|---|---|
| `classification(int labelIndex, int numClasses)` | Configure for classification. `labelIndex` is the column containing the integer class index. |
| `regression(int labelIndex)` | Single-output regression. |
| `regression(int from, int to)` | Multi-output regression with contiguous label columns. |
| `preProcessor(DataSetPreProcessor)` | Optional: apply a preprocessor to each batch before returning it. |
| `maxNumBatches(int)` | Limit the number of batches returned per epoch. |
| `collectMetaData(boolean)` | Include `RecordMetaData` in the returned `DataSet` for traceability. |
| `writableConverter(WritableConverter)` | Override how `Writable` values are converted to numeric. |

### Loading by Metadata

After setting `collectMetaData(true)`, individual examples can be reloaded:

```java
DataSet batch = iter.next();
List<RecordMetaData> meta = batch.getExampleMetaData(RecordMetaData.class);

// Reload specific examples
DataSet reloaded = iter.loadFromMetaData(meta.subList(0, 3));
```

---

## SequenceRecordReaderDataSetIterator

Produces time series `DataSet` objects from `SequenceRecordReader` sources. Features and labels can come from separate readers (separate files) or the same reader.

```java
import org.datavec.api.records.reader.impl.csv.CSVSequenceRecordReader;
import org.deeplearning4j.datasets.datavec.SequenceRecordReaderDataSetIterator;
import org.deeplearning4j.datasets.datavec.SequenceRecordReaderDataSetIterator.AlignmentMode;

CSVSequenceRecordReader featuresReader = new CSVSequenceRecordReader(0, ",");
featuresReader.initialize(new NumberedFileInputSplit("/data/features_%d.csv", 0, numFiles - 1));

CSVSequenceRecordReader labelsReader = new CSVSequenceRecordReader(0, ",");
labelsReader.initialize(new NumberedFileInputSplit("/data/labels_%d.csv", 0, numFiles - 1));

DataSetIterator seqIter = new SequenceRecordReaderDataSetIterator(
    featuresReader,
    labelsReader,
    batchSize,
    numClasses,
    false,                           // regression = false (classification)
    AlignmentMode.ALIGN_END);        // Align labels to end for sequence classification
```

**Alignment modes:**

| Mode | Description |
|---|---|
| `EQUAL_LENGTH` | Features and labels have the same number of time steps. |
| `ALIGN_START` | Labels are aligned to the start of the feature sequence; remainder is padded. |
| `ALIGN_END` | Labels are aligned to the end (typical for sequence classification — one label per sequence). |

---

## RecordReaderMultiDataSetIterator

For `ComputationGraph` with multiple inputs and/or multiple outputs. Allows columns from one or more `RecordReader` instances to be routed to different network inputs and outputs.

```java
import org.deeplearning4j.datasets.datavec.RecordReaderMultiDataSetIterator;

CSVRecordReader rr = new CSVRecordReader(0, ',');
rr.initialize(new FileSplit(new File("/path/to/data.csv")));

MultiDataSetIterator iter = new RecordReaderMultiDataSetIterator.Builder(batchSize)
    .addReader("data", rr)
    // Input 1: columns 0-9 (features)
    .addInput("data", 0, 9)
    // Input 2: columns 10-19 (auxiliary features)
    .addInput("data", 10, 19)
    // Output 1: column 20, classification with 5 classes
    .addOutputOneHot("data", 20, 5)
    // Output 2: columns 21-22, regression targets
    .addOutput("data", 21, 22)
    .build();
```

---

## Async Data Loading

### AsyncDataSetIterator

Wraps any `DataSetIterator` and prefetches minibatches on a background thread, overlapping data loading with GPU computation. DL4J's `model.fit(iterator)` methods automatically apply async prefetching when `iterator.asyncSupported()` returns `true`, so manual wrapping is usually not required.

```java
import org.deeplearning4j.datasets.iterator.AsyncDataSetIterator;

// Manual wrapping (usually not needed)
DataSetIterator baseIter = new RecordReaderDataSetIterator(...);
DataSetIterator asyncIter = new AsyncDataSetIterator(baseIter, 8); // prefetch queue size 8
```

Key characteristics:
- Uses a separate thread to call `next()` on the underlying iterator.
- By default, uses a cyclical workspace to avoid off-heap memory accumulation.
- Call `asyncIter.shutdown()` to cleanly stop the background thread when done.
- `asyncSupported()` returns `true`.

### AsyncMultiDataSetIterator

The `MultiDataSetIterator` equivalent. Wraps any `MultiDataSetIterator` for background prefetching.

```java
import org.deeplearning4j.datasets.iterator.AsyncMultiDataSetIterator;

MultiDataSetIterator asyncMulti = new AsyncMultiDataSetIterator(baseMultiIter, 4);
```

### AsyncShieldDataSetIterator

The inverse of `AsyncDataSetIterator`. Wraps an iterator and forces `asyncSupported()` to return `false`, preventing DL4J from automatically applying async prefetching. Use this when your iterator is not thread-safe or manages its own internal buffering.

```java
import org.deeplearning4j.datasets.iterator.AsyncShieldDataSetIterator;

DataSetIterator shielded = new AsyncShieldDataSetIterator(myThreadUnsafeIter);
// Now model.fit(shielded) will not wrap it in async prefetch
```

---

## Utility Iterators

### WorkspacesShieldDataSetIterator

Detaches all `INDArray` objects coming out of the wrapped iterator from any memory workspace, producing "safe" arrays that can be held across workspace scopes. Intended for debugging and testing.

```java
import org.deeplearning4j.datasets.iterator.WorkspacesShieldDataSetIterator;

DataSetIterator safe = new WorkspacesShieldDataSetIterator(baseIter);
```

### INDArrayDataSetIterator

Creates a `DataSetIterator` directly from an `Iterable` of `(features, labels)` `INDArray` pairs. Useful for synthetic data or in-memory datasets.

```java
import org.deeplearning4j.datasets.iterator.INDArrayDataSetIterator;

List<Pair<INDArray, INDArray>> data = new ArrayList<>();
data.add(Pair.of(features1, labels1));
data.add(Pair.of(features2, labels2));

DataSetIterator iter = new INDArrayDataSetIterator(data, batchSize);
```

### DoublesDataSetIterator

Same as `INDArrayDataSetIterator` but accepts `double[]` pairs rather than `INDArray` pairs.

```java
import org.deeplearning4j.datasets.iterator.DoublesDataSetIterator;

List<Pair<double[], double[]>> data = /* ... */;
DataSetIterator iter = new DoublesDataSetIterator(data, batchSize);
```

### SamplingDataSetIterator

Randomly samples (with replacement) from an in-memory `DataSet`.

```java
import org.deeplearning4j.datasets.iterator.SamplingDataSetIterator;

DataSet fullData = /* all data in memory */;
int totalSamples = 10000;
DataSetIterator sampled = new SamplingDataSetIterator(fullData, batchSize, totalSamples);
```

---

## Train/Test Splitting

### DataSetIteratorSplitter

Splits a single `DataSetIterator` into train and test iterators based on a ratio. The underlying iterator is read sequentially — the first portion becomes train, the remainder becomes test.

```java
import org.deeplearning4j.datasets.iterator.DataSetIteratorSplitter;

DataSetIterator base = new RecordReaderDataSetIterator(...);
long totalBatches = 1000;  // total number of batches in the base iterator
double trainFraction = 0.8;

DataSetIteratorSplitter splitter = new DataSetIteratorSplitter(base, totalBatches, trainFraction);

DataSetIterator trainIter = splitter.getTrainIterator();  // first 800 batches
DataSetIterator testIter  = splitter.getTestIterator();   // last 200 batches
```

**Constraints:**
- Do not use the test iterator twice in a row without first resetting the train iterator.
- Do not use with iterators that shuffle data between epochs (splitter assumes deterministic order).

### MultiDataSetIteratorSplitter

The `MultiDataSetIterator` equivalent:

```java
import org.deeplearning4j.datasets.iterator.MultiDataSetIteratorSplitter;

MultiDataSetIterator base = /* ... */;
MultiDataSetIteratorSplitter splitter =
    new MultiDataSetIteratorSplitter(base, totalBatches, 0.8);

MultiDataSetIterator trainIter = splitter.getTrainIterator();
MultiDataSetIterator testIter  = splitter.getTestIterator();
```

---

## Creating Custom Iterators

For datasets not covered by built-in iterators, extend `BaseDatasetIterator` or implement `DataSetIterator` directly.

### Minimal Implementation

```java
import org.nd4j.linalg.dataset.DataSet;
import org.nd4j.linalg.dataset.api.DataSetPreProcessor;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;
import org.nd4j.linalg.factory.Nd4j;

public class MyCustomIterator implements DataSetIterator {

    private final int batchSize;
    private final int numFeatures;
    private final int numClasses;
    private int cursor = 0;
    private final int totalExamples;
    private DataSetPreProcessor preProcessor;

    public MyCustomIterator(int batchSize, int numFeatures, int numClasses, int totalExamples) {
        this.batchSize = batchSize;
        this.numFeatures = numFeatures;
        this.numClasses = numClasses;
        this.totalExamples = totalExamples;
    }

    @Override
    public DataSet next(int num) {
        int actualBatch = Math.min(num, totalExamples - cursor);
        // Build features and labels INDArrays for this batch
        INDArray features = Nd4j.create(actualBatch, numFeatures);
        INDArray labels   = Nd4j.create(actualBatch, numClasses);

        for (int i = 0; i < actualBatch; i++) {
            // Fill features and labels from your data source at index (cursor + i)
            // features.putRow(i, ...);
            // labels.putRow(i, ...);
        }
        cursor += actualBatch;

        DataSet ds = new DataSet(features, labels);
        if (preProcessor != null) preProcessor.preProcess(ds);
        return ds;
    }

    @Override public DataSet next()            { return next(batchSize); }
    @Override public int inputColumns()         { return numFeatures; }
    @Override public int totalOutcomes()        { return numClasses; }
    @Override public boolean resetSupported()   { return true; }
    @Override public boolean asyncSupported()   { return true; }
    @Override public void reset()               { cursor = 0; }
    @Override public int batch()                { return batchSize; }
    @Override public boolean hasNext()          { return cursor < totalExamples; }

    @Override
    public void setPreProcessor(DataSetPreProcessor preProcessor) {
        this.preProcessor = preProcessor;
    }

    @Override
    public DataSetPreProcessor getPreProcessor() { return preProcessor; }

    @Override public List<String> getLabels()   { return null; }
    @Override public void remove()              { throw new UnsupportedOperationException(); }
}
```

### Using a PreProcessor

Preprocessors are applied inside `next()` before the batch is returned. Common built-in preprocessors:

```java
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerMinMaxScaler;
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerStandardize;
import org.nd4j.linalg.dataset.api.preprocessor.ImagePreProcessingScaler;

// Fit normalizer on training data, then apply to both train and test iterators
NormalizerStandardize normalizer = new NormalizerStandardize();
normalizer.fit(trainIter);
trainIter.reset();

trainIter.setPreProcessor(normalizer);
testIter.setPreProcessor(normalizer);  // use same statistics
```

Normalizers can be saved alongside model files using `ModelSerializer`:

```java
ModelSerializer.writeModel(model, modelFile, true, normalizer);
```

---

## Interface Reference

| Class / Interface | Package |
|---|---|
| `DataSetIterator` | `org.nd4j.linalg.dataset.api.iterator` |
| `MultiDataSetIterator` | `org.nd4j.linalg.dataset.api.iterator` |
| `MnistDataSetIterator` | `org.deeplearning4j.datasets.iterator.impl` |
| `Cifar10DataSetIterator` | `org.deeplearning4j.datasets.iterator.impl` |
| `EmnistDataSetIterator` | `org.deeplearning4j.datasets.iterator.impl` |
| `IrisDataSetIterator` | `org.deeplearning4j.datasets.iterator.impl` |
| `LFWDataSetIterator` | `org.deeplearning4j.datasets.iterator.impl` |
| `TinyImageNetDataSetIterator` | `org.deeplearning4j.datasets.iterator.impl` |
| `UciSequenceDataSetIterator` | `org.deeplearning4j.datasets.iterator.impl` |
| `RecordReaderDataSetIterator` | `org.deeplearning4j.datasets.datavec` |
| `SequenceRecordReaderDataSetIterator` | `org.deeplearning4j.datasets.datavec` |
| `RecordReaderMultiDataSetIterator` | `org.deeplearning4j.datasets.datavec` |
| `AsyncDataSetIterator` | `org.deeplearning4j.datasets.iterator` |
| `AsyncMultiDataSetIterator` | `org.deeplearning4j.datasets.iterator` |
| `AsyncShieldDataSetIterator` | `org.deeplearning4j.datasets.iterator` |
| `WorkspacesShieldDataSetIterator` | `org.deeplearning4j.datasets.iterator` |
| `INDArrayDataSetIterator` | `org.deeplearning4j.datasets.iterator` |
| `DoublesDataSetIterator` | `org.deeplearning4j.datasets.iterator` |
| `SamplingDataSetIterator` | `org.deeplearning4j.datasets.iterator` |
| `DataSetIteratorSplitter` | `org.deeplearning4j.datasets.iterator` |
| `MultiDataSetIteratorSplitter` | `org.deeplearning4j.datasets.iterator` |

---
title: "Data Pipelines"
description: "Loading, transforming, and feeding data for training — RecordReader, DataSetIterator, normalization, and mini-batching"
---

# Data Pipelines

Getting data into a neural network is often the most involved part of a project. DL4J deliberately separates data loading from training, which makes it possible to swap data sources, normalize differently, or experiment with mini-batch sizes without touching the model definition. This page walks through each layer of that pipeline.

## 1. The Data Pipeline

Data moves from disk (or memory) to model weights through a fixed sequence of components:

```
Raw Data → InputSplit → RecordReader → RecordReaderDataSetIterator → DataSet/DataSetIterator → Model.fit()
```

- **InputSplit** — tells the reader where to find the raw files
- **RecordReader** — reads each file/row and converts it into a `List<Writable>` record
- **RecordReaderDataSetIterator** — converts records into `DataSet` objects and groups them into mini-batches
- **DataSet / DataSetIterator** — the in-memory representation that `MultiLayerNetwork` and `ComputationGraph` consume via `fit()`

The following sections cover each component in detail.

---

## 2. InputSplit

An `InputSplit` defines which files or URIs a `RecordReader` should consume. The three most common variants are:

```java
import org.datavec.api.split.FileSplit;
import org.datavec.api.split.NumberedFileInputSplit;
import org.datavec.api.split.CollectionInputSplit;

// Point at a directory and filter by extension
FileSplit fileSplit = new FileSplit(new File("/path/to/data"), new String[]{"csv"});

// Point at a numbered sequence of files (e.g., train_0.csv … train_99.csv)
NumberedFileInputSplit numberedSplit = new NumberedFileInputSplit("/path/train_%d.csv", 0, 99);

// Point at an explicit list of URIs
CollectionInputSplit collSplit = new CollectionInputSplit(uriList);
```

`FileSplit` can also recurse into subdirectories, which is useful for image datasets where each class lives in its own folder. Pass `true` as the third argument to enable recursive traversal.

---

## 3. RecordReader

A `RecordReader` reads raw bytes from an `InputSplit` and converts each example into a `List<Writable>`. A `Writable` is a lightweight, type-safe wrapper around primitive values (integers, floats, text, and so on).

### CSV Data

```java
import org.datavec.api.records.reader.impl.csv.CSVRecordReader;

// Skip 1 header line, use comma as delimiter
CSVRecordReader csvReader = new CSVRecordReader(1, ',');
csvReader.initialize(new FileSplit(new File("data.csv")));
```

Each row becomes one record. Numeric columns become `DoubleWritable` or `IntWritable`; text columns become `Text`.

### Image Data

```java
import org.datavec.image.recordreader.ImageRecordReader;
import org.datavec.image.loader.NativeImageLoader;
import org.datavec.api.io.labels.ParentPathLabelGenerator;

// 224×224 RGB images; label is inferred from the parent directory name
ImageRecordReader imgReader = new ImageRecordReader(224, 224, 3);  // height, width, channels
imgReader.initialize(
    new FileSplit(imageDir, NativeImageLoader.ALLOWED_FORMATS),
    new ParentPathLabelGenerator()
);
```

`ParentPathLabelGenerator` assigns labels by directory name, which is the standard layout for classification datasets (one subdirectory per class).

### Sequence (Time Series) Data

```java
import org.datavec.api.records.reader.impl.csv.CSVSequenceRecordReader;

// Each file is one sequence; skip 0 header rows; comma delimiter
CSVSequenceRecordReader seqReader = new CSVSequenceRecordReader(0, ",");
seqReader.initialize(new NumberedFileInputSplit("/path/seq_%d.csv", 0, 49));
```

Each file represents a complete time series. Rows within the file are individual time steps.

### Other RecordReader Implementations

| Class | Use case |
|---|---|
| `JacksonRecordReader` | JSON or YAML files |
| `LineRecordReader` | Raw text, one example per line |
| `RegexLineRecordReader` | Text files with regex-based field extraction |
| `LibSvmRecordReader` | Sparse feature vectors in LibSVM format |

---

## 4. DataSet and MultiDataSet

`DataSet` is the central data container in DL4J. It holds:

- **features** — an `INDArray` of input values
- **labels** — an `INDArray` of target values
- **feature mask** (optional) — used for variable-length sequences
- **label mask** (optional) — used for variable-length sequences

```java
DataSet ds = iterator.next();
INDArray features = ds.getFeatures();  // shape [batchSize, numFeatures]
INDArray labels   = ds.getLabels();    // shape [batchSize, numClasses]
```

### Array Shapes by Network Type

| Network type | Features shape | Labels shape |
|---|---|---|
| Feed-forward (dense) | `[batchSize, numFeatures]` | `[batchSize, numClasses]` |
| Convolutional (CNN) | `[batchSize, channels, height, width]` | `[batchSize, numClasses]` |
| Recurrent (RNN) | `[batchSize, features, timeSteps]` | `[batchSize, numClasses, timeSteps]` |

Labels for classification are stored as **one-hot vectors** — a `[batchSize, numClasses]` array where each row has a 1.0 at the index of the true class and 0.0 everywhere else. `RecordReaderDataSetIterator` handles this conversion automatically.

### MultiDataSet

`MultiDataSet` extends this concept to networks with multiple input arrays and/or multiple output arrays. It is used exclusively with `ComputationGraph` and `RecordReaderMultiDataSetIterator`. For a `MultiLayerNetwork` with a single input and output, `DataSet` is sufficient.

---

## 5. RecordReaderDataSetIterator

`RecordReaderDataSetIterator` is the bridge between DataVec (data loading) and DL4J (training). It reads records from a `RecordReader`, converts them to `INDArray` features and labels, and groups them into mini-batches of the requested size.

### Classification

```java
import org.deeplearning4j.datasets.datavec.RecordReaderDataSetIterator;

// Column 4 is the integer class label (0-indexed); 3 possible classes
DataSetIterator trainIter = new RecordReaderDataSetIterator.Builder(csvReader, batchSize)
    .classification(4, 3)   // labelColumnIndex, numClasses
    .build();
```

### Regression

```java
// Columns 0–3 are features (implied); columns 4–5 are regression targets
DataSetIterator regIter = new RecordReaderDataSetIterator.Builder(csvReader, batchSize)
    .regression(4, 5)       // labelColumnFrom, labelColumnTo (inclusive)
    .build();
```

### Image Classification

```java
DataSetIterator imageIter = new RecordReaderDataSetIterator.Builder(imgReader, batchSize)
    .classification(1, numClasses)  // label index is always 1 for ImageRecordReader
    .build();
```

### Sequence Data

For time series, use `SequenceRecordReaderDataSetIterator` instead:

```java
import org.deeplearning4j.datasets.datavec.SequenceRecordReaderDataSetIterator;

DataSetIterator seqIter = new SequenceRecordReaderDataSetIterator(
    featureReader,          // SequenceRecordReader for inputs
    labelReader,            // SequenceRecordReader for labels
    miniBatchSize,
    numPossibleLabels
);
```

### Multiple Inputs and Outputs

For `ComputationGraph` models that consume more than one input array or produce more than one output, use `RecordReaderMultiDataSetIterator`:

```java
import org.deeplearning4j.datasets.datavec.RecordReaderMultiDataSetIterator;

MultiDataSetIterator multiIter = new RecordReaderMultiDataSetIterator.Builder(batchSize)
    .addReader("features", csvReader)
    .addInput("features", 0, 3)      // columns 0–3 as input
    .addOutputOneHot("features", 4, numClasses)  // column 4 as one-hot label
    .build();
```

---

## 6. Mini-Batching

Neural networks learn by computing gradients over small random samples of the training data, called mini-batches, and updating the weights after each one. This is the core of stochastic gradient descent.

The `batchSize` parameter on the iterator controls how many examples are returned per call to `next()`.

```java
int batchSize = 64;
DataSetIterator iter = new RecordReaderDataSetIterator.Builder(reader, batchSize)
    .classification(labelIndex, numClasses)
    .build();

while (iter.hasNext()) {
    DataSet batch = iter.next();  // 64 examples per batch
    model.fit(batch);
}
iter.reset();  // reset to beginning for the next epoch
```

### Choosing a Batch Size

| Batch size | Effect |
|---|---|
| 1 (online learning) | Very noisy gradient updates; slow convergence but good regularization |
| 32–256 (typical) | Good balance of gradient quality and memory use |
| Full dataset (batch GD) | Low noise, but high memory cost and fewer weight updates per epoch |

For most tasks, start with 32 or 64 and increase if GPU utilization is low. Values that are powers of 2 allow the hardware to use memory alignment optimizations more effectively.

When training for multiple epochs with `model.fit(iterator, numEpochs)`, DL4J calls `iterator.reset()` automatically between epochs. If you manage the training loop manually, call `iter.reset()` yourself at the start of each epoch.

---

## 7. Built-in Dataset Iterators

DL4J ships with ready-to-use iterators for several standard benchmark datasets. These download and cache the data automatically on first use.

```java
import org.deeplearning4j.datasets.iterator.impl.MnistDataSetIterator;
import org.deeplearning4j.datasets.iterator.impl.CifarDataSetIterator;
import org.deeplearning4j.datasets.iterator.impl.IrisDataSetIterator;
import org.deeplearning4j.datasets.iterator.impl.EmnistDataSetIterator;

// MNIST: 60,000 training / 10,000 test grayscale digit images (28×28)
DataSetIterator mnist = new MnistDataSetIterator(batchSize, true);   // true = training set

// CIFAR-10: 50,000 training RGB images (32×32), 10 classes
DataSetIterator cifar = new CifarDataSetIterator(batchSize, numExamples);

// Iris: classic 4-feature, 3-class tabular dataset (150 total examples)
DataSetIterator iris = new IrisDataSetIterator(batchSize, 150);

// EMNIST: extended MNIST with digits, letters, and merged variants
DataSetIterator emnist = new EmnistDataSetIterator(
    EmnistDataSetIterator.Set.BALANCED, batchSize, true
);
```

These iterators are useful for quickly verifying that a model architecture works before moving to a production dataset.

---

## 8. Normalization

Neural networks are sensitive to the scale of their inputs. Weights are typically initialized to small values near zero, and the optimizer step size (learning rate) is calibrated for inputs that are also near zero. Raw features — pixel values in the range 0–255, sensor readings spanning thousands of units, or mixed-scale tabular data — will cause very slow or unstable training unless normalized first.

DL4J provides normalizers that attach directly to iterators via `setPreProcessor()`.

### NormalizerStandardize (zero mean, unit variance)

This is the default choice for tabular data. It subtracts the mean and divides by the standard deviation so that each feature ends up with mean 0 and standard deviation 1.

```java
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerStandardize;

NormalizerStandardize normalizer = new NormalizerStandardize();
normalizer.fit(trainIter);           // scan the training set to compute mean and std

trainIter.setPreProcessor(normalizer);
testIter.setPreProcessor(normalizer); // apply the SAME statistics to test data
```

### NormalizerMinMaxScaler (range scaling)

Scales each feature to a target range — typically [0, 1] or [-1, 1].

```java
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerMinMaxScaler;

NormalizerMinMaxScaler minMax = new NormalizerMinMaxScaler(0, 1);
minMax.fit(trainIter);
trainIter.setPreProcessor(minMax);
testIter.setPreProcessor(minMax);
```

### ImagePreProcessingScaler

Divides pixel values (0–255) by a configurable factor to produce values in [0, 1].

```java
import org.nd4j.linalg.dataset.api.preprocessor.ImagePreProcessingScaler;

ImagePreProcessingScaler imageScaler = new ImagePreProcessingScaler(0, 1);
imageIter.setPreProcessor(imageScaler);
```

### VGG16ImagePreProcessor

For fine-tuning pretrained ImageNet models, this preprocessor subtracts the per-channel ImageNet mean (the same preprocessing VGGNet was trained with).

```java
import org.deeplearning4j.nn.modelimport.keras.preprocessors.KerasImageDataFormat;
import org.deeplearning4j.zoo.util.darknet.DarknetLabels;
import org.nd4j.linalg.dataset.api.preprocessor.VGG16ImagePreProcessor;

VGG16ImagePreProcessor vggPreProcessor = new VGG16ImagePreProcessor();
imageIter.setPreProcessor(vggPreProcessor);
```

### Critical Rule: Fit on Training Data Only

Always call `normalizer.fit()` on the **training** iterator and then apply the resulting normalizer to **both** training and test/validation iterators. Fitting on the full dataset (including test data) leaks information from the test set into training and inflates reported accuracy.

When you save a model for inference, save the normalizer alongside it so that the same transformation is applied to new inputs at prediction time:

```java
import org.nd4j.linalg.dataset.api.preprocessor.serializer.NormalizerSerializer;

// Save
NormalizerSerializer.getDefault().write(normalizer, new File("normalizer.bin"));

// Load
NormalizerStandardize loaded = NormalizerSerializer.getDefault()
    .restore(new File("normalizer.bin"));
```

---

## 9. Async Data Loading

On GPU-accelerated systems, data loading on the CPU can become the bottleneck: the GPU sits idle waiting for the next batch. Wrapping an iterator in `AsyncDataSetIterator` causes a background thread to prefetch batches while the GPU is processing the current one.

```java
import org.deeplearning4j.datasets.iterator.AsyncDataSetIterator;

// Prefetch up to 3 batches ahead
DataSetIterator asyncIter = new AsyncDataSetIterator(baseIter, 3);
```

Note that `MultiLayerNetwork.fit(DataSetIterator)` and `ComputationGraph.fit(DataSetIterator)` automatically wrap compatible iterators in `AsyncDataSetIterator` internally, so you normally do not need to add this wrapper yourself. It is useful when you are calling `iterator.next()` in a manual training loop.

For multi-input networks, the equivalent wrapper is `AsyncMultiDataSetIterator`.

---

## Putting It Together: A Complete CSV Example

```java
import org.datavec.api.records.reader.impl.csv.CSVRecordReader;
import org.datavec.api.split.FileSplit;
import org.deeplearning4j.datasets.datavec.RecordReaderDataSetIterator;
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerStandardize;

int batchSize   = 64;
int labelIndex  = 4;   // column index of the class label
int numClasses  = 3;

// --- Training iterator ---
CSVRecordReader trainReader = new CSVRecordReader(1, ',');  // skip header
trainReader.initialize(new FileSplit(new File("train.csv")));
DataSetIterator trainIter = new RecordReaderDataSetIterator.Builder(trainReader, batchSize)
    .classification(labelIndex, numClasses)
    .build();

// --- Test iterator ---
CSVRecordReader testReader = new CSVRecordReader(1, ',');
testReader.initialize(new FileSplit(new File("test.csv")));
DataSetIterator testIter = new RecordReaderDataSetIterator.Builder(testReader, batchSize)
    .classification(labelIndex, numClasses)
    .build();

// --- Normalization ---
NormalizerStandardize normalizer = new NormalizerStandardize();
normalizer.fit(trainIter);
trainIter.setPreProcessor(normalizer);
testIter.setPreProcessor(normalizer);

// --- Training ---
model.fit(trainIter, numEpochs);
```

This pattern — separate readers for train and test, normalization fit on training data only, iterator reset handled by `fit()` — is the foundation for virtually every DL4J training workflow.

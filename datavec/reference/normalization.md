---
title: "Normalization"
description: "Data normalization in DataVec — normalizer implementations and serialization"
---

# Normalization

Neural networks are sensitive to the scale of input features. When one feature ranges from 0 to 1,000,000 and another from 0.0 to 1.0, the large-scale feature tends to dominate gradients during training, causing slow convergence or numeric instability. Normalizing your data to a consistent scale is one of the most impactful preprocessing steps you can take.

DataVec provides normalizer classes that:
1. **Fit** — scan your training data once to compute statistics (mean, std, min, max)
2. **Transform** — apply the normalization to each batch during training
3. **Revert** — undo normalization (useful for interpreting model predictions in original units)
4. **Serialize** — save and reload normalizer parameters so you can apply the same transformation at inference time

Normalizers are applied as `DataSetPreProcessor` objects on a `DataSetIterator`, not as `TransformProcess` steps. This means they operate on `DataSet` (INDArray) objects, not on `List<Writable>` records.

## NormalizerStandardize

Standardizes features to have **zero mean and unit variance** (Z-score normalization).

```
x_normalized = (x - mean) / std
```

This is the most common normalization strategy for neural networks.

```java
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerStandardize;

// 1. Create normalizer
NormalizerStandardize normalizer = new NormalizerStandardize();

// 2. Fit on training data (scans all batches to compute mean and std per feature)
normalizer.fit(trainIterator);

// 3. Attach to iterators — normalization is applied automatically on each next() call
trainIterator.reset();
trainIterator.setPreProcessor(normalizer);

// Apply the same normalizer (with the same statistics) to validation data
valIterator.setPreProcessor(normalizer);

// 4. Train
model.fit(trainIterator);
```

To also normalize labels (useful for regression tasks):

```java
normalizer.fitLabel(true);
normalizer.fit(trainIterator);
```

### Reverting Predictions

After the model produces predictions on normalized inputs, revert to the original scale:

```java
INDArray predictions = model.output(normalizedFeatures);
normalizer.revertLabels(predictions);   // now in original units
```

## NormalizerMinMaxScaler

Scales features to a target range, by default **[0, 1]**. For each feature:

```
x_normalized = (x - min) / (max - min) * (targetMax - targetMin) + targetMin
```

```java
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerMinMaxScaler;

// Default: scale to [0, 1]
NormalizerMinMaxScaler normalizer = new NormalizerMinMaxScaler();

// Custom range: scale to [-1, 1]
NormalizerMinMaxScaler normalizer = new NormalizerMinMaxScaler(-1.0, 1.0);

// Fit and attach
normalizer.fit(trainIterator);
trainIterator.reset();
trainIterator.setPreProcessor(normalizer);
```

MinMax scaling is sensitive to outliers: a single very large value in the training data will compress the majority of values into a narrow portion of the target range. Consider clipping outliers with a `TransformProcess` before applying min-max scaling.

## ImagePreProcessingScaler

Specialized for image data. Scales pixel values from the 0–255 range to a target range (default 0–1), without fitting. No statistics need to be computed because the input range is known from the pixel bit depth.

```java
import org.nd4j.linalg.dataset.api.preprocessor.ImagePreProcessingScaler;

// Scale 8-bit pixels (0-255) to [0, 1]
DataSetPreProcessor scaler = new ImagePreProcessingScaler(0, 1);

// Scale 8-bit pixels to [-1, 1]
DataSetPreProcessor scaler = new ImagePreProcessingScaler(-1, 1);

// 16-bit image pixels (0-65535) to [0, 1]
DataSetPreProcessor scaler = new ImagePreProcessingScaler(0, 1, 16);

// For floating point images where pixel values are already [0.0, 1.0]
DataSetPreProcessor scaler = new ImagePreProcessingScaler(0, 1, 1);

iterator.setPreProcessor(scaler);
```

Unlike `NormalizerStandardize` and `NormalizerMinMaxScaler`, `ImagePreProcessingScaler` does not require a `fit` call.

## MultiNormalizerStandardize

The multi-input/output equivalent of `NormalizerStandardize`, for use with `MultiDataSet` and `MultiDataSetIterator` (which are used with `ComputationGraph` networks that have multiple input or output arrays).

```java
import org.nd4j.linalg.dataset.api.preprocessor.MultiNormalizerStandardize;

MultiNormalizerStandardize normalizer = new MultiNormalizerStandardize();

// Fit on all input/output arrays
normalizer.fit(multiDataSetIterator);

multiDataSetIterator.reset();
multiDataSetIterator.setPreProcessor(normalizer);
```

## MultiNormalizerMinMaxScaler

Like `MultiNormalizerStandardize` but applies min-max scaling:

```java
import org.nd4j.linalg.dataset.api.preprocessor.MultiNormalizerMinMaxScaler;

MultiNormalizerMinMaxScaler normalizer = new MultiNormalizerMinMaxScaler(0, 1);
normalizer.fit(multiDataSetIterator);
multiDataSetIterator.setPreProcessor(normalizer);
```

## MultiNormalizerHybrid

For `ComputationGraph` networks where different inputs need different normalization strategies — or no normalization at all (e.g., embedding layer inputs that should stay as integer indices).

```java
import org.nd4j.linalg.dataset.api.preprocessor.MultiNormalizerHybrid;

MultiNormalizerHybrid normalizer = new MultiNormalizerHybrid()
    .standardizeAllInputs()           // default: standardize all inputs
    .minMaxScaleInput(1, 0, 1)        // override: min-max scale input index 1
    // no normalization for input index 2 (embedding input)
    .standardizeAllOutputs();         // standardize all outputs

normalizer.fit(iterator);
iterator.setPreProcessor(normalizer);
```

## CompositeDataSetPreProcessor

Apply multiple pre-processors in sequence. Useful when you need both image scaling and label normalization:

```java
import org.nd4j.linalg.dataset.api.preprocessor.CompositeDataSetPreProcessor;

DataSetPreProcessor composite = new CompositeDataSetPreProcessor(
    new ImagePreProcessingScaler(0, 1),
    new NormalizerStandardize()   // fine-tune around the image scale
);

iterator.setPreProcessor(composite);
```

## VGG16ImagePreProcessor

Subtracts the channel-wise mean RGB values computed on the ImageNet training set, following the preprocessing described in the VGG paper. Use this when loading a pretrained VGG16 model from the DL4J model zoo.

```java
import org.nd4j.linalg.dataset.api.preprocessor.VGG16ImagePreProcessor;

iterator.setPreProcessor(new VGG16ImagePreProcessor());
```

No `fit` call is required; the channel means are hardcoded from the original paper.

## Serializing Normalizers

Normalizer parameters (mean, std, min, max) must be saved alongside your model so that production inference applies exactly the same scaling as training. DL4J's `ModelSerializer` handles this automatically when you include the normalizer:

```java
import org.deeplearning4j.util.ModelSerializer;

// Save model and normalizer together
File modelFile = new File("model.zip");
ModelSerializer.writeModel(model, modelFile, true, normalizer);

// Load both
MultiLayerNetwork restoredModel = ModelSerializer.restoreMultiLayerNetwork(modelFile);
NormalizerStandardize restoredNormalizer =
    ModelSerializer.restoreNormalizer(modelFile);

// Apply to a new iterator at inference time
inferenceIterator.setPreProcessor(restoredNormalizer);
```

### Using NormalizerSerializer Directly

For more control, use `NormalizerSerializer`:

```java
import org.nd4j.linalg.dataset.api.preprocessor.serializer.NormalizerSerializer;

NormalizerSerializer serializer = NormalizerSerializer.getDefault();

// Save
serializer.write(normalizer, new File("normalizer.bin"));

// Load
NormalizerStandardize loaded = serializer.restore(new File("normalizer.bin"));
```

The serializer supports all normalizer types automatically. It stores the statistics in a compact binary format.

## Choosing a Normalizer

| Scenario | Recommended Normalizer |
|---|---|
| Tabular data, regression or classification | `NormalizerStandardize` |
| Tabular data with bounded features (percentages, probabilities) | `NormalizerMinMaxScaler` |
| Image pixel data | `ImagePreProcessingScaler` |
| Multi-input `ComputationGraph`, all inputs same type | `MultiNormalizerStandardize` or `MultiNormalizerMinMaxScaler` |
| Multi-input `ComputationGraph`, mixed input types | `MultiNormalizerHybrid` |
| Transfer learning from VGG16 | `VGG16ImagePreProcessor` |
| Sequential application of multiple normalizers | `CompositeDataSetPreProcessor` |

## Common Mistakes

**Fitting on validation or test data**: Always fit the normalizer on training data only, then apply the fitted normalizer to validation and test iterators. Fitting on all data introduces leakage.

**Not resetting iterators after fitting**: `normalizer.fit(iter)` exhausts the iterator. Always call `iter.reset()` before using it for training.

**Not saving the normalizer with the model**: If you save the model but not the normalizer, inference in production will receive un-normalized inputs and produce incorrect predictions.

**Applying normalizer before and after `TransformProcess`**: The `TransformProcess` operates on `List<Writable>` records; normalizers operate on `INDArray` batches. They run at different points in the pipeline. The typical order is: `RecordReader` → `TransformProcess` → `DataSetIterator` → `normalizer.transform()`.

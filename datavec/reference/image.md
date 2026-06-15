---
title: "Image Data"
description: "Loading and preprocessing image data — ImageRecordReader, NativeImageLoader, and image transforms"
---

# Image Data

DataVec provides specialized tooling for image datasets. The image pipeline handles the full journey from a directory of JPEG/PNG files through to normalized `DataSet` tensors ready for convolutional neural network training.

## Directory Structure for Labeled Images

The most common image dataset format is a directory where each subdirectory is a class label:

```
/data/images/
    train/
        cat/
            cat_001.jpg
            cat_002.jpg
        dog/
            dog_001.jpg
            dog_002.jpg
    val/
        cat/
            cat_val_001.jpg
        dog/
            dog_val_001.jpg
```

DataVec's `ImageRecordReader` handles this structure automatically using a label generator that derives the label from the parent directory name.

## ImageRecordReader

`ImageRecordReader` reads images from a `FileSplit` and resizes them to a fixed height, width, and channel count. It appends a one-hot label based on the directory structure.

```java
import org.datavec.image.recordreader.ImageRecordReader;
import org.datavec.image.loader.NativeImageLoader;
import org.datavec.api.split.FileSplit;
import org.datavec.api.io.labels.ParentPathLabelGenerator;

int height = 224;
int width = 224;
int channels = 3;  // RGB; use 1 for grayscale

// Label generator: uses the name of the parent directory as the class label
ParentPathLabelGenerator labelMaker = new ParentPathLabelGenerator();

ImageRecordReader rr = new ImageRecordReader(height, width, channels, labelMaker);
rr.initialize(new FileSplit(new File("/data/images/train/")));
```

The reader produces records with one `NDArrayWritable` (the image pixels) plus one `IntWritable` (the label index). When used with `RecordReaderDataSetIterator`, the label index tells the iterator which column is the label:

```java
DataSetIterator iter = new RecordReaderDataSetIterator(
    rr,
    batchSize,
    1,          // label is at index 1
    numClasses  // number of distinct class labels
);
```

### Getting the List of Labels

```java
List<String> labels = rr.getLabels();
// ["cat", "dog"] — in alphabetical order
```

### Image Format Support

`ImageRecordReader` uses JavaCV / OpenCV under the hood via `NativeImageLoader`. Supported formats include JPEG, PNG, BMP, GIF, TIFF, and WebP.

## Image Transforms (Augmentation)

`ImageTransform` applies geometric or color transformations to images as they are read. Transforms are applied before the image is returned by the reader, making them transparent to the rest of the pipeline.

Pass an `ImageTransform` (or a list of them) to the `ImageRecordReader`:

```java
import org.datavec.image.transform.*;

// Resize to exactly the specified dimensions (the reader also does this, but
// explicit ResizeImageTransform can be useful in pipelines)
ImageTransform resize = new ResizeImageTransform(224, 224);

// Randomly flip horizontally (50% chance)
ImageTransform flip = new FlipImageTransform(1);  // 1 = flip around y-axis

// Randomly rotate by up to 15 degrees
ImageTransform rotate = new RotateImageTransform(new Random(42), 15);

// Scale brightness/contrast randomly in range [0.9, 1.1]
ImageTransform scale = new ScaleImageTransform(new Random(42), 0.1f);

// Crop a random region and scale back to target size
ImageTransform crop = new CropImageTransform(new Random(42), 10);

// Warp the image randomly
ImageTransform warp = new WarpImageTransform(new Random(42), 5);
```

Apply a single transform at reader initialization:

```java
ImageRecordReader rr = new ImageRecordReader(height, width, channels, labelMaker);
rr.initialize(new FileSplit(new File("/data/train/")), flip);
```

### Pipeline of Transforms

Use `PipelineImageTransform` to apply multiple transforms in sequence:

```java
import org.datavec.image.transform.PipelineImageTransform;

PipelineImageTransform pipeline = new PipelineImageTransform.Builder()
    .addImageTransform(new FlipImageTransform(new Random(42)))
    .addImageTransform(new RotateImageTransform(new Random(42), 10))
    .addImageTransform(new ScaleImageTransform(new Random(42), 0.05f))
    .build();

rr.initialize(new FileSplit(new File("/data/train/")), pipeline);
```

### Random Choice Transform

Randomly apply one transform from a list (useful for diverse augmentation):

```java
List<ImageTransform> transforms = new ArrayList<>();
transforms.add(new FlipImageTransform(1));
transforms.add(new RotateImageTransform(new Random(42), 20));
transforms.add(new CropImageTransform(new Random(42), 20));

// Pick one transform randomly for each image
ImageTransform random = new MultiImageTransform(new Random(42),
    transforms.toArray(new ImageTransform[0]));
```

## NativeImageLoader

`NativeImageLoader` loads individual image files or `BufferedImage` objects directly into ND4J `INDArray` tensors, without the `RecordReader` abstraction. Use this for single-image inference or when building custom data pipelines.

```java
import org.datavec.image.loader.NativeImageLoader;
import org.nd4j.linalg.api.ndarray.INDArray;

NativeImageLoader loader = new NativeImageLoader(224, 224, 3);

// Load from file
INDArray image = loader.asMatrix(new File("/path/to/image.jpg"));
// shape: [1, 3, 224, 224]  (minibatch=1, channels, height, width)

// Load from BufferedImage (e.g., from a web upload)
BufferedImage buffered = ImageIO.read(inputStream);
INDArray image = loader.asMatrix(buffered);

// Load from byte array
byte[] bytes = Files.readAllBytes(Paths.get("/path/to/image.jpg"));
INDArray image = loader.asMatrix(bytes);
```

### Normalization After Loading

`NativeImageLoader` returns raw pixel values in the range [0, 255]. Normalize them before passing to a model:

```java
// Scale to [0, 1]
DataSetPreProcessor scaler = new ImagePreProcessingScaler(0, 1);
DataSet ds = new DataSet(image, null);
scaler.preProcess(ds);
INDArray normalized = ds.getFeatures();

// Or apply directly to INDArray
image.divi(255.0);
```

## Complete Image Training Pipeline

A complete pipeline from labeled image directory to model training:

```java
int height = 224, width = 224, channels = 3;
int batchSize = 32;
int numClasses = 10;
int numEpochs = 50;

ParentPathLabelGenerator labelMaker = new ParentPathLabelGenerator();

// --- Training data with augmentation ---
ImageTransform augmentation = new PipelineImageTransform.Builder()
    .addImageTransform(new FlipImageTransform(new Random(42)))
    .addImageTransform(new RotateImageTransform(new Random(42), 15))
    .build();

ImageRecordReader trainRr = new ImageRecordReader(height, width, channels, labelMaker);
trainRr.initialize(new FileSplit(new File("/data/train/")), augmentation);

DataSetIterator trainIter = new RecordReaderDataSetIterator(
    trainRr, batchSize, 1, numClasses);

// --- Validation data without augmentation ---
ImageRecordReader valRr = new ImageRecordReader(height, width, channels, labelMaker);
valRr.initialize(new FileSplit(new File("/data/val/")));

DataSetIterator valIter = new RecordReaderDataSetIterator(
    valRr, batchSize, 1, numClasses);

// --- Normalize pixel values to [0, 1] ---
ImagePreProcessingScaler scaler = new ImagePreProcessingScaler(0, 1);
trainIter.setPreProcessor(scaler);
valIter.setPreProcessor(scaler);

// --- Train ---
for (int epoch = 0; epoch < numEpochs; epoch++) {
    model.fit(trainIter);
    Evaluation eval = model.evaluate(valIter);
    System.out.println("Epoch " + epoch + ": " + eval.accuracy());
    trainIter.reset();
    valIter.reset();
}
```

## Single Image Inference

For real-time inference on a single image:

```java
NativeImageLoader loader = new NativeImageLoader(224, 224, 3);
INDArray image = loader.asMatrix(new File("/path/to/query.jpg"));
image.divi(255.0);  // normalize

// Add batch dimension if needed: shape [1, C, H, W]
INDArray output = model.output(image);
int predictedClass = Nd4j.argMax(output, 1).getInt(0);
System.out.println("Predicted: " + labels.get(predictedClass));
```

## Label Generators

DataVec provides two label generator strategies:

**ParentPathLabelGenerator** (most common): derives the label from the immediate parent directory name. The class `cat` in `/data/cat/img001.jpg` becomes label "cat".

```java
ParentPathLabelGenerator labelMaker = new ParentPathLabelGenerator();
```

**PathLabelGenerator**: a functional interface you can implement for custom label derivation logic:

```java
PathLabelGenerator customLabels = uri -> {
    // Example: extract label from file name prefix
    String filename = new File(uri).getName();
    return new Text(filename.split("_")[0]);
};

ImageRecordReader rr = new ImageRecordReader(height, width, channels, customLabels);
```

## Tips for Large Image Datasets

- Keep images on a fast local SSD rather than a network mount — image loading is I/O-bound
- Use multiple worker threads in `AsyncDataSetIterator` to prefetch batches while the GPU trains
- Pre-resize images to your target dimensions offline to avoid repeated resize overhead at training time
- Use `FileSplit` with `Random` to shuffle the file order for each epoch

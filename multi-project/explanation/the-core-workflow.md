---
title: "Ecosystem Overview"
description: "Overview of the Eclipse Deeplearning4j ecosystem — ND4J, DL4J, DataVec, SameDiff, Python4J, and OmniHub"
---

# Ecosystem Overview

Eclipse Deeplearning4j is a suite of JVM-based libraries for building, training, and deploying deep learning models. The project is hosted as a single monorepo on GitHub and ships six user-facing libraries that cover every stage of a machine learning project — from raw data ingestion to distributed training and production serving.

DL4J runs on Java 11 and later. It targets x86\_64 (with AVX2 and AVX512 acceleration), ARM (AArch64), and PowerPC (PPC64LE) CPUs, as well as NVIDIA GPUs through a CUDA backend. Windows, Linux, and macOS are all first-class platforms.

---

## The Library Stack

The six libraries are layered. Lower layers provide the compute substrate; upper layers provide higher-level abstractions. Understanding the boundaries between layers prevents confusion when debugging dependency issues or choosing which API to use.

### libnd4j (C++)

libnd4j is the native C++ foundation. It provides hand-tuned kernel implementations for tensor operations — element-wise math, BLAS routines, convolutions, reductions, random number generation — compiled separately for each target platform. The x86 builds use AVX2 or AVX512 intrinsics; the CUDA build links against cuBLAS and cuDNN.

Users never import libnd4j directly. It is bundled inside the platform-specific JAR artifacts for ND4J. Its existence matters when diagnosing native crashes or when building from source for a custom platform.

### ND4J (Java)

ND4J is the tensor library for the JVM, analogous in purpose to NumPy. Every numerical operation in the DL4J ecosystem flows through ND4J.

The central abstraction is `INDArray` — an n-dimensional array that may live in CPU RAM or GPU VRAM depending on the active backend. The `Nd4j` factory class creates arrays:

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

// Create a 3x4 matrix of zeros
INDArray zeros = Nd4j.zeros(3, 4);

// Create from Java array
INDArray a = Nd4j.create(new float[]{1, 2, 3, 4, 5, 6}, new int[]{2, 3});

// Element-wise multiply
INDArray b = Nd4j.ones(2, 3);
INDArray c = a.mul(b);

// Matrix multiply
INDArray result = a.mmul(b.transpose()); // shape [2, 2]
```

ND4J also ships activations (`Nd4j.getActivations()`), loss functions (`LossFunctions`), updaters (Adam, SGD, RMSProp), and evaluation classes (`Evaluation`, `RegressionEvaluation`).

The backend is pluggable at the dependency level. Swap `nd4j-native` for `nd4j-cuda` and the same Java code executes on GPU — no source changes required.

### SameDiff (inside ND4J)

SameDiff is ND4J's automatic differentiation framework. It lives in the `nd4j-api` module alongside the `INDArray` API. SameDiff lets you define computation graphs symbolically using `SDVariable` nodes, execute them with concrete data, and differentiate through them automatically.

```java
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.autodiff.samediff.SDVariable;

SameDiff sd = SameDiff.create();

SDVariable x = sd.placeHolder("x", DataType.FLOAT, 2, 3);
SDVariable w = sd.var("w", Nd4j.randn(3, 1));
SDVariable b = sd.var("b", Nd4j.zeros(1));

SDVariable pred = sd.nn.linear(x, w, b);  // x * w + b
SDVariable loss = sd.loss.meanSquaredError("loss", pred, sd.placeHolder("label", DataType.FLOAT, 2, 1));

sd.fit(...);  // trains via backprop
```

SameDiff can import pre-trained TensorFlow SavedModel and frozen graph files, as well as ONNX models, making it the primary entry point for running Python-trained models inside the JVM without a Python runtime.

### DataVec

DataVec is the data ETL (extract, transform, load) library. Raw data in CSV, image directories, JSON, sequence files, JDBC, or dozens of other formats flows in through a `RecordReader` and comes out as `DataSet` objects ready for training.

The two core components are:

- **RecordReader** — reads raw bytes and emits `List<Writable>` records. Implementations include `CSVRecordReader`, `ImageRecordReader`, `JDBCRecordReader`, and many others.
- **TransformProcess** — a chainable pipeline that maps, filters, normalizes, and reorders records according to a declared `Schema`.

```java
Schema inputSchema = new Schema.Builder()
    .addColumnString("label")
    .addColumnsFloat("feature1", "feature2", "feature3")
    .build();

TransformProcess tp = new TransformProcess.Builder(inputSchema)
    .stringToOneHot("label", Arrays.asList("cat", "dog", "bird"))
    .normalize("feature1", NormalizerType.STANDARDIZE)
    .build();

RecordReader rr = new CSVRecordReader(1, ',');  // skip header
rr.initialize(new FileSplit(new File("data.csv")));

RecordReader transformed = new TransformProcessRecordReader(rr, tp);
DataSetIterator iter = new RecordReaderDataSetIterator(transformed, 32, 0, 3);
```

DataVec pipelines run locally or scale out on Apache Spark with no code changes to the transform logic.

### Deeplearning4j (DL4J)

DL4J is the high-level neural network API. It sits on top of ND4J and DataVec and provides two model types:

- **`MultiLayerNetwork`** — a sequential stack of layers, suitable for feedforward, convolutional, and recurrent networks.
- **`ComputationGraph`** — a directed acyclic graph of layers, required for multi-input/multi-output architectures, skip connections (ResNet), and any topology that `MultiLayerNetwork` cannot express.

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .updater(new Adam(1e-3))
    .list()
    .layer(new DenseLayer.Builder().nIn(784).nOut(256).activation(Activation.RELU).build())
    .layer(new DenseLayer.Builder().nIn(256).nOut(128).activation(Activation.RELU).build())
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nIn(128).nOut(10).activation(Activation.SOFTMAX).build())
    .build();

MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
model.fit(trainIter, 10);  // 10 epochs

Evaluation eval = model.evaluate(testIter);
System.out.println(eval.stats());
```

DL4J also includes:

- **NLP utilities** — Word2Vec, Doc2Vec, GloVe, and tokenizers.
- **Model Zoo** — pretrained weights for VGG16, ResNet50, YOLO, InceptionV3, and others via the `deeplearning4j-zoo` module.
- **Distributed training** — gradient sharing and parameter averaging on Apache Spark clusters via `deeplearning4j-scaleout-spark`.
- **Training UI** — a local web server (port 9000 by default) that streams loss curves and weight histograms to a browser during training.

### Python4J

Python4J embeds CPython 3.10 into the JVM via JavaCPP-packaged binaries. This allows Java code to call Python functions, execute scripts, and pass data between the two runtimes without serialization overhead.

The `python4j-numpy` extension provides zero-copy interop between `INDArray` and `numpy.ndarray` by sharing the underlying memory buffer:

```java
PythonCondaEnvironment env = PythonCondaEnvironment.ofDirectory("/opt/conda/envs/myenv");
Python.setContext(env);

INDArray data = Nd4j.linspace(0, 9, 10).reshape(2, 5);

PythonVariables inputs = new PythonVariables();
inputs.addNDArray("x", data);

PythonVariables outputs = new PythonVariables();
outputs.addNDArray("result");

Python.exec("import numpy as np; result = np.square(x)", inputs, outputs);

INDArray result = outputs.getNDArrayValue("result");
```

Python4J is useful for calling scipy routines, custom preprocessing logic, or model inference libraries that do not yet have a JVM equivalent.

### OmniHub

OmniHub is a model hub for the DL4J ecosystem. It provides a registry of pretrained models in DL4J (`MultiLayerNetwork`/`ComputationGraph`) and SameDiff formats, downloadable with a single API call:

```java
ZooModel zooModel = OmniHubModel.builder()
    .modelName("VGG16")
    .pretrained(PretrainedType.IMAGENET)
    .build();

ComputationGraph model = (ComputationGraph) zooModel.initPretrained();
```

OmniHub handles checksum verification, caching to `~/.deeplearning4j/models/`, and version resolution.

---

## Dependency Diagram

```
libnd4j  (C++, platform-native kernels)
    ^
    | JavaCPP bindings
    |
ND4J  (nd4j-native or nd4j-cuda)  <-- SameDiff (autodiff, inside nd4j-api)
    ^
    |
DataVec (ETL)    DL4J (neural networks)    Python4J    OmniHub
```

`DataVec`, `DL4J`, `Python4J`, and `OmniHub` all declare a dependency on `nd4j-api`. Your application must supply exactly one backend implementation (`nd4j-native` or `nd4j-cuda`) on the classpath at runtime.

---

## Typical Workflow

A complete DL4J project follows this path:

1. **Raw data** (CSV files, image folders, database tables) is pointed to by a `RecordReader`.
2. **DataVec** applies a `TransformProcess` to clean, type-cast, and normalize the records.
3. A **`DataSetIterator`** (usually `RecordReaderDataSetIterator`) wraps the reader and batches records into `DataSet` objects.
4. **DL4J** trains a `MultiLayerNetwork` or `ComputationGraph` by iterating over the `DataSetIterator`.
5. An **`Evaluation`** object scores the model on a held-out test iterator.
6. **`ModelSerializer.writeModel()`** saves the trained model and normalizer to disk.
7. At inference time, **`ModelSerializer.restoreMultiLayerNetwork()`** reloads the model, which can then score new `INDArray` inputs directly.

---

## Maven Setup for M2.1

Add the version property and the two core dependencies to your `pom.xml`:

```xml
<properties>
    <dl4j.version>1.0.0-M2.1</dl4j.version>
</properties>

<dependencies>
    <!-- DL4J high-level API. Transitively pulls in deeplearning4j-nn and nd4j-api. -->
    <dependency>
        <groupId>org.deeplearning4j</groupId>
        <artifactId>deeplearning4j-core</artifactId>
        <version>${dl4j.version}</version>
    </dependency>

    <!-- CPU backend with natives bundled for all supported OS/arch combos. -->
    <dependency>
        <groupId>org.nd4j</groupId>
        <artifactId>nd4j-native-platform</artifactId>
        <version>${dl4j.version}</version>
    </dependency>
</dependencies>
```

`deeplearning4j-core` is a convenience aggregate that pulls in `deeplearning4j-nn` (the layer and model classes) and `nd4j-api` (the `INDArray` interface and SameDiff). It does not pull in a backend — that is always your choice.

For DataVec, add:

```xml
<dependency>
    <groupId>org.datavec</groupId>
    <artifactId>datavec-api</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

Add format-specific modules as needed — for example `datavec-data-image` for `ImageRecordReader`, or `datavec-data-codec` for video.

---

## Backend Selection

### Platform artifacts vs. classifier-specific artifacts

The `-platform` suffix (`nd4j-native-platform`, `nd4j-cuda-platform`) causes Maven to download native JARs for all supported OS and architecture combinations. This is the recommended approach during development because it produces a portable artifact — the same JAR runs on any developer machine regardless of OS or CPU brand.

In production, where the target hardware is known, use classifier-specific artifacts to avoid shipping unnecessary natives. For example, to target Linux on x86\_64 with AVX2:

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${dl4j.version}</version>
    <classifier>linux-x86_64-avx2</classifier>
</dependency>
```

### Switching to GPU

Replace `nd4j-native-platform` with `nd4j-cuda-platform`. No Java source code changes are required:

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.6-platform</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

The CUDA version suffix (`11.6`) must match the CUDA toolkit installed on the machine. At runtime ND4J detects available GPUs through JavaCPP's CUDA bindings and allocates device memory automatically. You can control device selection with `Nd4j.getAffinityManager()`.

Only one backend may be active per JVM process. If both `nd4j-native` and `nd4j-cuda` appear on the classpath, `nd4j-cuda` wins by default; set the system property `-Dorg.nd4j.linalg.factoryclass=org.nd4j.linalg.cpu.nativecpu.CpuNDArrayFactory` to force CPU.

---

## Where to Go Next

With the ecosystem map in mind, the remaining core-concepts pages cover each layer in depth:

- **INDArray and ND4J Operations** — shapes, strides, views, broadcasting rules, and the full operation API.
- **SameDiff and Automatic Differentiation** — defining graphs, custom ops, and importing TensorFlow/ONNX models.
- **DataVec ETL Pipelines** — schemas, all built-in `RecordReader` implementations, `TransformProcess` in detail, and Spark execution.
- **MultiLayerNetwork and ComputationGraph** — layer catalog, configuration options, training callbacks, and the training UI.
- **Backend Configuration** — memory management, workspace configuration, cuDNN integration, and profiling native performance.

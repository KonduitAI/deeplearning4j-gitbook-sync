---
title: "Core Concepts"
description: "Architecture overview of Deeplearning4j — MultiLayerNetwork, ComputationGraph, the training pipeline, and DL4J's relationship to ND4J"
---

# Core Concepts

This page describes the architectural ideas that underpin every DL4J program. Reading it before diving into code will help you understand why the API is shaped the way it is and how the pieces fit together.

---

## DL4J's Place in the Ecosystem

Deeplearning4j (DL4J) is the high-level neural network library in the Eclipse Deeplearning4j ecosystem. It is built on top of ND4J, which itself sits on top of libnd4j, a native C++ library. Understanding the layer boundaries saves debugging time.

```
Your Java code
      │
      ▼
Deeplearning4j  (MultiLayerNetwork, ComputationGraph, layers, training)
      │
      ▼
ND4J            (INDArray, math ops, automatic differentiation via SameDiff)
      │
      ▼
libnd4j (C++)   (CPU kernels with AVX2/AVX512, CUDA kernels, BLAS, cuDNN)
```

DL4J provides:

- The **layer catalog** — dense, convolutional, recurrent, normalization, attention, and more
- **MultiLayerNetwork** and **ComputationGraph** — the two network execution engines
- **NeuralNetConfiguration.Builder** — a declarative DSL for specifying network hyperparameters
- **DataSetIterator** — the data pipeline abstraction
- **Evaluation** classes — accuracy, F1, ROC, regression metrics
- **Listeners** — hooks for logging, visualization, and early stopping

Everything DL4J does with numbers ultimately becomes `INDArray` operations executed by ND4J and dispatched to the native backend.

---

## ND4J: The Numerical Foundation

ND4J is DL4J's tensor library. It is analogous in purpose to NumPy. Every input, output, weight matrix, gradient, and activation in a DL4J network is an `INDArray`.

`INDArray` is an interface in `org.nd4j.linalg.api.ndarray`. You never construct it with `new`; you use the static factory on `Nd4j`:

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.factory.Nd4j;

// 2D matrix of zeros, shape [3, 4]
INDArray zeros = Nd4j.zeros(DataType.FLOAT, 3, 4);

// From a Java array, shape [2, 3]
INDArray a = Nd4j.create(new float[]{1, 2, 3, 4, 5, 6}, new long[]{2, 3});

// Matrix multiply: [2, 3] x [3, 2] → [2, 2]
INDArray result = a.mmul(a.transpose());
```

The data lives **off-heap** in native memory, not in the Java heap. This allows zero-copy transfers to GPU memory and avoids GC pressure on large tensors. You should be aware of this when profiling memory usage — Java heap profilers will not show ND4J tensor data.

### DataType

In M2.1 you should set the network data type explicitly. The recommended type for most workloads is `DataType.FLOAT` (32-bit). `DataType.DOUBLE` is available when extra precision is needed, and `DataType.HALF` (16-bit) or `DataType.BFLOAT16` are available for memory-constrained GPU workloads that support reduced precision.

Set the data type on the network configuration builder:

```java
new NeuralNetConfiguration.Builder()
    .dataType(DataType.FLOAT)
    ...
```

This controls the data type of all trainable parameters (weights and biases) in the network. Input data passed via `DataSetIterator` is cast to match automatically.

---

## The Training Pipeline

A DL4J training run has four stages:

```
DataSetIterator  →  model.fit()  →  Evaluation  →  ModelSerializer
     │                  │               │                │
 Load & batch       Forward pass    Compute metrics   Save to disk
 Normalize          Compute loss
                    Backward pass
                    Update weights
```

### Stage 1: DataSetIterator

`DataSetIterator` (interface in `org.nd4j.linalg.dataset.api.iterator`) is the contract between your data pipeline and the network. It produces `DataSet` objects on demand. Each `DataSet` holds:

- `features`: an `INDArray` of shape `[batchSize, numFeatures]` (or `[batchSize, channels, height, width]` for images)
- `labels`: an `INDArray` of shape `[batchSize, numClasses]` for classification (one-hot encoded) or `[batchSize, 1]` for regression

DL4J ships several built-in iterators:

| Iterator | Data Source |
|----------|------------|
| `MnistDataSetIterator` | MNIST handwritten digits (downloaded automatically) |
| `CifarDataSetIterator` | CIFAR-10 image dataset |
| `IrisDataSetIterator` | UCI Iris dataset |
| `RecordReaderDataSetIterator` | DataVec `RecordReader` — CSV, images, audio, video |
| `EarlyTerminationDataSetIterator` | Wraps another iterator; stops after N batches |

For custom data, the most common path is `RecordReaderDataSetIterator` backed by a DataVec reader:

```java
import org.datavec.api.records.reader.impl.csv.CSVRecordReader;
import org.datavec.api.split.FileSplit;
import org.deeplearning4j.datasets.datavec.RecordReaderDataSetIterator;

CSVRecordReader reader = new CSVRecordReader(0, ',');
reader.initialize(new FileSplit(new File("data/train.csv")));

// 4 feature columns, label column index 4, 3 classes
DataSetIterator iter = new RecordReaderDataSetIterator(reader, 32, 4, 3);
```

**Normalization** is applied via a `DataNormalization` preprocessor. Set it once on the iterator and it runs automatically on every batch:

```java
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerStandardize;

NormalizerStandardize normalizer = new NormalizerStandardize();
normalizer.fit(trainIter);           // compute mean and std from training data
trainIter.setPreProcessor(normalizer);
testIter.setPreProcessor(normalizer); // use the same statistics on test data
```

### Stage 2: model.fit()

`model.fit(DataSetIterator)` runs one epoch — one complete pass through the iterator. DL4J handles the inner loop:

```
for each batch from iterator:
    1. forward pass  → predictions
    2. compute loss  → scalar
    3. backward pass → gradients for every parameter
    4. updater step  → adjust parameters
```

To train for multiple epochs, call `fit` in a loop and reset the iterator each time:

```java
for (int epoch = 0; epoch < numEpochs; epoch++) {
    model.fit(trainIter);
    trainIter.reset();
    // optionally evaluate here
}
```

Alternatively, `MultiLayerNetwork.fit(DataSetIterator, int numEpochs)` handles this for you:

```java
model.fit(trainIter, numEpochs);  // resets iterator between epochs automatically
```

### Stage 3: Evaluation

The `Evaluation` class (in `org.nd4j.evaluation.classification`) computes metrics by iterating through a `DataSetIterator` and comparing network outputs to ground truth labels:

```java
import org.nd4j.evaluation.classification.Evaluation;

Evaluation eval = model.evaluate(testIter);
System.out.println(eval.stats());  // prints accuracy, precision, recall, F1 per class
```

For regression use `RegressionEvaluation`. For binary classification use `EvaluationBinary` or check `eval.auc()` after computing the ROC with `ROC`.

### Stage 4: Saving and Loading

```java
import org.deeplearning4j.util.ModelSerializer;

// Save (true = save updater state for resuming training)
ModelSerializer.writeModel(model, new File("model.zip"), true);

// Load
MultiLayerNetwork loaded = ModelSerializer.restoreMultiLayerNetwork(
    new File("model.zip"));
```

The `.zip` archive contains the network configuration JSON, the parameter `INDArray`s as binary, and (optionally) the updater state. Loading restores the model to exactly the state it was in when saved.

---

## NeuralNetConfiguration.Builder

All network construction goes through `NeuralNetConfiguration.Builder`. It sets global hyperparameters that apply to every layer unless overridden at the layer level.

```java
import org.deeplearning4j.nn.conf.NeuralNetConfiguration;
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.learning.config.Adam;

new NeuralNetConfiguration.Builder()
    .seed(42)                     // random seed for reproducibility
    .dataType(DataType.FLOAT)     // parameter data type
    .updater(new Adam(1e-3))      // optimizer + learning rate
    .l2(1e-4)                     // L2 weight decay (global)
    .weightInit(WeightInit.XAVIER) // weight initialization (global)
    ...
```

### Updaters in M2.1

The updater (optimizer) is specified as a class instance. The old enum-based API (`Updater.ADAM`) is removed in M2.1. Common updater classes and their key parameters:

| Class | Constructor | Notes |
|-------|------------|-------|
| `Adam` | `new Adam(lr)` | Good default for most tasks |
| `AdamW` | `new AdamW(lr, weightDecay)` | Adam with decoupled weight decay |
| `Sgd` | `new Sgd(lr)` | Stochastic gradient descent |
| `Nesterovs` | `new Nesterovs(lr, momentum)` | SGD with Nesterov momentum |
| `RmsProp` | `new RmsProp(lr)` | Adaptive per-parameter LR |
| `AdaGrad` | `new AdaGrad(lr)` | Accumulates squared gradients |
| `AdaDelta` | `new AdaDelta()` | No LR required |

All updater classes are in `org.nd4j.linalg.learning.config`.

Learning rate schedules are supported:

```java
import org.nd4j.linalg.schedule.ExponentialSchedule;
import org.nd4j.linalg.schedule.ScheduleType;

// Decay LR by 0.95 every epoch
ISchedule schedule = new ExponentialSchedule(ScheduleType.EPOCH, 1e-3, 0.95);
.updater(new Adam(schedule))
```

### Per-layer Overrides

Any global setting from the builder can be overridden inside a specific layer's builder:

```java
.layer(new DenseLayer.Builder()
    .nIn(512).nOut(256)
    .activation(Activation.RELU)
    .weightInit(WeightInit.RELU)  // overrides global XAVIER for this layer
    .updater(new Sgd(0.001))      // overrides global Adam for this layer
    .l2(0.0)                      // disable L2 for this layer
    .build())
```

---

## MultiLayerNetwork

`MultiLayerNetwork` is DL4J's network type for **strictly sequential** architectures: a chain of layers where the output of layer N is the input to layer N+1.

### When to Use MultiLayerNetwork

- Feedforward (dense) networks
- Convolutional networks without skip connections
- Simple RNNs / LSTMs
- Autoencoders with a single encoder and decoder path
- Any architecture that can be described as "one input, one output, layers in a line"

### Configuration and Initialization

```java
import org.deeplearning4j.nn.conf.MultiLayerConfiguration;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.deeplearning4j.nn.conf.layers.*;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.learning.config.Adam;
import org.nd4j.linalg.lossfunctions.LossFunctions;

MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .l2(1e-4)
    .list()
    .layer(new DenseLayer.Builder().nIn(784).nOut(256)
        .activation(Activation.RELU).build())
    .layer(new DenseLayer.Builder().nIn(256).nOut(128)
        .activation(Activation.RELU).build())
    .layer(new OutputLayer.Builder(
                LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nIn(128).nOut(10)
        .activation(Activation.SOFTMAX).build())
    .build();

MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
```

The `.list()` call on the builder transitions to `ListBuilder`, which collects layers in order. Layers do not need index numbers; they are appended in call order.

### Key MultiLayerNetwork Methods

| Method | Description |
|--------|-------------|
| `model.init()` | Initialize weights, allocate parameter arrays |
| `model.fit(iterator)` | Train for one epoch |
| `model.fit(iterator, epochs)` | Train for N epochs, auto-resetting |
| `model.output(input)` | Run forward pass, return predictions |
| `model.evaluate(iterator)` | Compute evaluation metrics |
| `model.setListeners(listeners)` | Attach training event listeners |
| `model.numParams()` | Total number of trainable parameters |
| `model.params()` | All parameters as a flat `INDArray` |
| `model.getLayer(index)` | Access a specific layer at runtime |
| `model.summary()` | Print layer-by-layer parameter counts |

---

## ComputationGraph

`ComputationGraph` is DL4J's network type for **directed acyclic graphs (DAGs)** — architectures where layers can have multiple inputs, multiple outputs, or skip connections.

### When to Use ComputationGraph

- ResNet / DenseNet with skip connections
- Encoder-decoder with attention
- Multi-input networks (e.g., image + text features merged)
- Multi-output networks (e.g., shared backbone with multiple heads)
- Siamese networks (shared weights, two input branches)
- Any architecture that cannot be expressed as a strict linear chain

### Configuration

`ComputationGraph` uses `ComputationGraphConfiguration` instead of `MultiLayerConfiguration`. Layers are given string names, and you declare their inputs by name:

```java
import org.deeplearning4j.nn.conf.ComputationGraphConfiguration;
import org.deeplearning4j.nn.conf.graph.MergeVertex;
import org.deeplearning4j.nn.graph.ComputationGraph;

ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .graphBuilder()
    // Declare network inputs by name
    .addInputs("input")
    // Each addLayer() specifies: name, layer, input names
    .addLayer("dense1",
        new DenseLayer.Builder().nIn(784).nOut(512)
            .activation(Activation.RELU).build(),
        "input")
    .addLayer("dense2",
        new DenseLayer.Builder().nIn(784).nOut(512)
            .activation(Activation.RELU).build(),
        "input")
    // MergeVertex concatenates tensors along the feature dimension
    .addVertex("merge",
        new MergeVertex(),
        "dense1", "dense2")            // two inputs → concatenated [batchSize, 1024]
    .addLayer("output",
        new OutputLayer.Builder(
                LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
            .nIn(1024).nOut(10)
            .activation(Activation.SOFTMAX).build(),
        "merge")
    .setOutputs("output")
    .build();

ComputationGraph model = new ComputationGraph(conf);
model.init();
```

### ResNet-style Skip Connection Example

```java
ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .graphBuilder()
    .addInputs("input")
    .addLayer("layer1",
        new DenseLayer.Builder().nIn(256).nOut(256)
            .activation(Activation.RELU).build(),
        "input")
    .addLayer("layer2",
        new DenseLayer.Builder().nIn(256).nOut(256)
            .activation(Activation.RELU).build(),
        "layer1")
    // ElementWiseVertex adds the skip connection: input + layer2
    .addVertex("skip",
        new ElementWiseVertex(ElementWiseVertex.Op.Add),
        "input", "layer2")
    .addLayer("output",
        new OutputLayer.Builder(LossFunctions.LossFunction.MSE)
            .nIn(256).nOut(10)
            .activation(Activation.IDENTITY).build(),
        "skip")
    .setOutputs("output")
    .build();
```

### MultiLayerNetwork vs ComputationGraph: Summary

| Feature | MultiLayerNetwork | ComputationGraph |
|---------|-----------------|-----------------|
| Architecture | Sequential only | Arbitrary DAG |
| Configuration class | `MultiLayerConfiguration` | `ComputationGraphConfiguration` |
| Layer ordering | Implicit (call order) | Explicit (by name + declared inputs) |
| Multiple inputs | No | Yes |
| Multiple outputs | No | Yes |
| Skip connections | No | Yes |
| API complexity | Lower | Higher |
| Typical use | MLP, simple CNN/RNN | ResNet, multi-task, Siamese |

If your architecture fits `MultiLayerNetwork`, prefer it — the simpler API is less error-prone. When you need branches, merges, or multiple I/O heads, use `ComputationGraph`.

---

## Layers

DL4J layers are configured with builder objects in `org.deeplearning4j.nn.conf.layers`. Every layer has at minimum:

- `nIn(int)` — number of inputs (can often be inferred by DL4J; required when DL4J cannot infer it)
- `nOut(int)` — number of outputs
- `activation(Activation)` — non-linearity applied after the linear transformation

### Feed-Forward Layers

`DenseLayer` — fully connected (linear) layer, the workhorse of MLPs:

```java
new DenseLayer.Builder()
    .nIn(512)
    .nOut(256)
    .activation(Activation.RELU)
    .weightInit(WeightInit.XAVIER)
    .dropOut(0.5)      // optional Bernoulli dropout
    .build()
```

`OutputLayer` — final layer with an attached loss function:

```java
new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
    .nIn(256)
    .nOut(10)
    .activation(Activation.SOFTMAX)
    .build()
```

### Convolutional Layers

`ConvolutionLayer` — 2D convolution for image data:

```java
new ConvolutionLayer.Builder(3, 3)   // kernel size 3x3
    .nIn(3)                          // 3 input channels (RGB)
    .nOut(64)                        // 64 filters
    .stride(1, 1)
    .padding(1, 1)
    .activation(Activation.RELU)
    .build()
```

`SubsamplingLayer` — max or average pooling:

```java
new SubsamplingLayer.Builder(SubsamplingLayer.PoolingType.MAX)
    .kernelSize(2, 2)
    .stride(2, 2)
    .build()
```

`BatchNormalization` — normalize activations per mini-batch to stabilize training:

```java
new BatchNormalization.Builder()
    .nIn(64)
    .nOut(64)
    .build()
```

### Recurrent Layers

`LSTM` — Long Short-Term Memory for sequential data:

```java
import org.deeplearning4j.nn.conf.layers.LSTM;

new LSTM.Builder()
    .nIn(100)     // input size per time step
    .nOut(256)    // hidden state size
    .activation(Activation.TANH)
    .build()
```

For sequence-to-label tasks, pair an `LSTM` with a `RnnOutputLayer` and `RNNFormat.NCW` input type.

### Normalization

| Layer | Purpose |
|-------|---------|
| `BatchNormalization` | Normalize per mini-batch (most common) |
| `LayerNormalization` | Normalize per example (better for RNNs/transformers) |
| `LocalResponseNormalization` | Cross-channel normalization (AlexNet era) |

---

## Listeners

Listeners are called during training to observe network state without interrupting the training loop. Set them on the model before calling `fit`:

```java
import org.deeplearning4j.optimize.listeners.ScoreIterationListener;
import org.deeplearning4j.optimize.listeners.PerformanceListener;

model.setListeners(
    new ScoreIterationListener(10),   // print loss every 10 mini-batches
    new PerformanceListener(10, true) // print samples/sec every 10 mini-batches
);
```

### Built-in Listeners

| Listener | Description |
|----------|-------------|
| `ScoreIterationListener(n)` | Logs training loss every N iterations |
| `PerformanceListener(n, true)` | Logs throughput (examples/sec) every N iterations |
| `EvaluativeListener(iter, n)` | Evaluates on a separate `DataSetIterator` every N epochs |
| `CheckpointListener` | Saves the model periodically during training |
| `TimeIterationListener` | Logs time per iteration |
| `CollectScoresListener` | Accumulates scores for programmatic access |
| `UIServer` listener | Sends stats to the training visualization web UI |

### Training Visualization UI

The `UIServer` streams training stats to a browser interface on `http://localhost:9000`:

```java
import org.deeplearning4j.ui.api.UIServer;
import org.deeplearning4j.ui.model.stats.StatsListener;
import org.deeplearning4j.ui.model.storage.InMemoryStatsStorage;

UIServer uiServer = UIServer.getInstance();
InMemoryStatsStorage statsStorage = new InMemoryStatsStorage();
uiServer.attach(statsStorage);

model.setListeners(new StatsListener(statsStorage));
```

Open `http://localhost:9000` in a browser while training to see the loss curve, activation histograms, parameter update magnitudes, and more.

---

## Data Flow Through a Network

Understanding what shape data must be in at each stage prevents the most common beginner errors.

### MultiLayerNetwork Data Flow

For a feed-forward network:

```
Input:   INDArray shape [batchSize, numFeatures]
          ↓
DenseLayer (nIn=numFeatures, nOut=256)
          ↓
Intermediate: INDArray shape [batchSize, 256]
          ↓
DenseLayer (nIn=256, nOut=128)
          ↓
Intermediate: INDArray shape [batchSize, 128]
          ↓
OutputLayer (nIn=128, nOut=10)
          ↓
Output:  INDArray shape [batchSize, 10]  ← softmax probabilities
```

For a convolutional network, `InputType.convolutionalFlat(height, width, channels)` tells DL4J to interpret flat image vectors as spatial data. DL4J then infers `nIn` for subsequent layers automatically — you do not need to compute the size manually:

```java
.setInputType(InputType.convolutionalFlat(28, 28, 1))
```

For recurrent networks:

```
Input:  INDArray shape [batchSize, numFeatures, sequenceLength]  (NCW format)
         ↓
LSTM   (nIn=numFeatures, nOut=hiddenSize)
         ↓
Output: INDArray shape [batchSize, hiddenSize, sequenceLength]
```

### Mini-batch Dimension

The first dimension of every array passed to `model.output()` or `model.fit()` is always the mini-batch dimension, regardless of the network type. Passing a single example still requires shape `[1, numFeatures]`, not `[numFeatures]`.

---

## DL4J's Relationship to SameDiff

SameDiff is ND4J's automatic differentiation framework. It is a lower-level API than DL4J: you define operations symbolically, execute them, and SameDiff differentiates the graph to compute gradients.

DL4J uses SameDiff internally as its computational backend. When you call `model.fit()`, DL4J translates the layer configuration into a SameDiff graph, executes the forward and backward passes, and uses the gradients to update parameters via the configured updater.

You normally do not need to touch SameDiff directly when using DL4J's high-level API. However, SameDiff becomes useful when you need:

- **Custom loss functions** not available in `LossFunctions`
- **Custom layer types** with non-standard forward/backward behavior
- **Pure autodiff workflows** without the layer/network abstraction (similar to PyTorch's functional API)
- **Model import** from ONNX or TensorFlow SavedModel (these are imported directly as SameDiff graphs)

A basic SameDiff example for context:

```java
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.autodiff.samediff.SDVariable;

SameDiff sd = SameDiff.create();

SDVariable x = sd.placeHolder("x", DataType.FLOAT, -1, 4);
SDVariable W = sd.var("W", Nd4j.randn(DataType.FLOAT, 4, 3));
SDVariable b = sd.var("b", Nd4j.zeros(DataType.FLOAT, 1, 3));

SDVariable out = x.mmul(W).add(b);        // linear layer
SDVariable softmax = sd.nn.softmax(out);  // softmax activation

// Execute with concrete data
Map<String, INDArray> inputs = Map.of("x", Nd4j.rand(DataType.FLOAT, 5, 4));
INDArray result = sd.output(inputs, "softmax")[0];
```

DL4J layers that expose a `SameDiff`-based implementation path allow fully custom gradient computation, making it possible to implement novel layer types that participate correctly in backpropagation without writing native C++ code.

---

## Key Package Reference

| Package | Contents |
|---------|---------|
| `org.deeplearning4j.nn.conf` | Configuration classes: `NeuralNetConfiguration`, `MultiLayerConfiguration`, `ComputationGraphConfiguration` |
| `org.deeplearning4j.nn.conf.layers` | All layer builder classes |
| `org.deeplearning4j.nn.multilayer` | `MultiLayerNetwork` |
| `org.deeplearning4j.nn.graph` | `ComputationGraph` |
| `org.deeplearning4j.nn.weights` | `WeightInit` enum |
| `org.deeplearning4j.optimize.listeners` | `ScoreIterationListener`, `PerformanceListener`, etc. |
| `org.deeplearning4j.util` | `ModelSerializer` |
| `org.nd4j.linalg.factory` | `Nd4j` factory, backend selection |
| `org.nd4j.linalg.api.ndarray` | `INDArray` interface |
| `org.nd4j.linalg.api.buffer` | `DataType` enum |
| `org.nd4j.linalg.learning.config` | `Adam`, `Sgd`, `RmsProp`, and all other updater classes |
| `org.nd4j.linalg.activations` | `Activation` enum |
| `org.nd4j.linalg.lossfunctions` | `LossFunctions` and individual loss classes |
| `org.nd4j.evaluation.classification` | `Evaluation`, `EvaluationBinary`, `ROC` |
| `org.nd4j.linalg.dataset.api.iterator` | `DataSetIterator` interface |
| `org.nd4j.linalg.dataset.api.preprocessor` | Normalization classes |
| `org.nd4j.autodiff.samediff` | `SameDiff`, `SDVariable` |

---

## Next Steps

- **Quickstart:** Follow the [Quickstart guide](quickstart.md) for an end-to-end MNIST example
- **Training details:** [The Training Loop](../../core-concepts/training-loop.md) covers updater options, schedules, and listener patterns in depth
- **Layer reference:** [Neural Network Fundamentals](../../core-concepts/neural-net-fundamentals.md) lists every available layer type with usage guidance
- **Data pipelines:** [Data Pipelines](../../core-concepts/data-pipelines.md) covers DataVec integration, custom iterators, and normalization
- **Evaluation:** [Evaluation](../../core-concepts/evaluation.md) covers all metric classes and how to interpret them
- **SameDiff:** See the SameDiff documentation for custom layers and low-level autodiff usage
- **API reference:** Browse the [Deeplearning4j Javadoc](/api/latest/)

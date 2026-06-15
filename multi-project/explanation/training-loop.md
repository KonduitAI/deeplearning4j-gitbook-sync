---
title: "The Training Loop"
description: "Building, configuring, and training neural networks — NeuralNetConfiguration, updaters, fit(), listeners, and ComputationGraph"
---

# The Training Loop

This page covers how to configure, build, and train neural networks in Deeplearning4j using the M2.1 API. The two network types — `MultiLayerNetwork` and `ComputationGraph` — share a common configuration system.

## NeuralNetConfiguration.Builder

All network configuration starts with `NeuralNetConfiguration.Builder`. This sets global hyperparameters that apply to every layer unless overridden:

```java
import org.deeplearning4j.nn.conf.NeuralNetConfiguration;
import org.deeplearning4j.nn.conf.MultiLayerConfiguration;
import org.deeplearning4j.nn.conf.layers.*;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.deeplearning4j.nn.weights.WeightInit;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.learning.config.Adam;
import org.nd4j.linalg.lossfunctions.impl.LossMCXENT;

MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)                                   // reproducibility
    .dataType(DataType.FLOAT)                   // network parameter type
    .updater(new Adam(1e-3))                    // optimizer with learning rate
    .l2(1e-4)                                   // L2 regularization
    .list()
    .layer(new DenseLayer.Builder()
        .nIn(784).nOut(256)
        .activation(Activation.RELU)
        .weightInit(WeightInit.RELU)
        .build())
    .layer(new DenseLayer.Builder()
        .nIn(256).nOut(128)
        .activation(Activation.RELU)
        .weightInit(WeightInit.RELU)
        .build())
    .layer(new OutputLayer.Builder(new LossMCXENT())
        .nIn(128).nOut(10)
        .activation(Activation.SOFTMAX)
        .build())
    .build();

MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
```

### Builder Methods Reference

| Method | Purpose | Default |
|--------|---------|---------|
| `seed(long)` | Random seed for reproducibility | System time |
| `dataType(DataType)` | Data type for parameters | `DataType.FLOAT` |
| `updater(IUpdater)` | Optimization algorithm | `new Sgd()` |
| `l2(double)` | L2 regularization coefficient | 0.0 |
| `l1(double)` | L1 regularization coefficient | 0.0 |
| `weightInit(WeightInit)` | Default weight initialization | `XAVIER` |
| `activation(Activation)` | Default activation function | `SIGMOID` |
| `dropOut(double)` | Dropout probability (all layers) | None |
| `gradientNormalization(GradientNormalization)` | Gradient clipping strategy | `None` |
| `gradientNormalizationThreshold(double)` | Clipping threshold | 1.0 |
| `trainingWorkspaceMode(WorkspaceMode)` | Memory management for training | `ENABLED` |
| `inferenceWorkspaceMode(WorkspaceMode)` | Memory management for inference | `ENABLED` |
| `convolutionMode(ConvolutionMode)` | Convolution padding mode | `Truncate` |
| `cudnnAlgoMode(AlgoMode)` | cuDNN algorithm selection | `PREFER_FASTEST` |
| `miniBatch(boolean)` | Enable mini-batch mode | `true` |

> **Migration note (beta4 → M2.1):** The `.learningRate()` method no longer exists — pass the learning rate to the updater constructor: `new Adam(1e-3)`. The `.pretrain(false).backprop(true)` calls have been removed. The `.updater(Updater.NESTEROVS)` enum-based form is replaced by `new Nesterovs(lr, momentum)`.

## Updaters (Optimizers)

Updaters control how gradients are used to update parameters. All are in `org.nd4j.linalg.learning.config`.

```java
// Usage: pass to NeuralNetConfiguration.Builder
.updater(new Adam(1e-3))
```

### Available Updaters

| Updater | Constructor | Notes |
|---------|-------------|-------|
| Adam | `new Adam(learningRate)` | Default choice for most tasks |
| AdaGrad | `new AdaGrad(learningRate)` | Per-parameter adaptive learning rate |
| AdaDelta | `new AdaDelta()` | No learning rate parameter |
| AdaMax | `new AdaMax(learningRate)` | Adam variant using infinity norm |
| AMSGrad | `new AMSGrad(learningRate)` | Adam with guaranteed convergence |
| AdaBelief | `new AdaBelief(learningRate)` | Adapts to gradient belief |
| Nadam | `new Nadam(learningRate)` | Adam + Nesterov momentum |
| Nesterovs | `new Nesterovs(learningRate, momentum)` | SGD with Nesterov momentum |
| RmsProp | `new RmsProp(learningRate)` | RNN-friendly adaptive rate |
| SGD | `new Sgd(learningRate)` | Basic stochastic gradient descent |
| NoOp | `new NoOp()` | Freezes parameters (no updates) |

### Learning Rate Schedules

Instead of a fixed learning rate, pass a schedule to the updater:

```java
import org.nd4j.linalg.schedule.*;

// Exponential decay
ISchedule expSchedule = new ExponentialSchedule(ScheduleType.EPOCH, 1e-3, 0.95);
.updater(new Adam(expSchedule))

// Step decay (halve every 10 epochs)
ISchedule stepSchedule = new StepSchedule(ScheduleType.EPOCH, 1e-3, 0.5, 10);
.updater(new Adam(stepSchedule))

// Polynomial decay
ISchedule polySchedule = new PolySchedule(ScheduleType.ITERATION, 1e-3, 1, 10000);
.updater(new Adam(polySchedule))

// Cyclic learning rate
ISchedule cycleSchedule = new CycleSchedule(ScheduleType.ITERATION, 1e-4, 1e-2, 1000);
.updater(new Adam(cycleSchedule))

// Custom map-based schedule
Map<Integer, Double> lrMap = new HashMap<>();
lrMap.put(0, 1e-3);
lrMap.put(5, 5e-4);
lrMap.put(10, 1e-4);
ISchedule mapSchedule = new MapSchedule(ScheduleType.EPOCH, lrMap);
.updater(new Adam(mapSchedule))
```

Available schedules: `ExponentialSchedule`, `StepSchedule`, `PolySchedule`, `SigmoidSchedule`, `InverseSchedule`, `CycleSchedule`, `RampSchedule`, `MapSchedule`, `FixedSchedule`.

`ScheduleType.EPOCH` changes the rate once per epoch. `ScheduleType.ITERATION` changes the rate every mini-batch.

## The Training Loop

### Basic Training

```java
int numEpochs = 20;
for (int epoch = 0; epoch < numEpochs; epoch++) {
    model.fit(trainIter);       // one pass through the training data
    trainIter.reset();          // reset iterator to beginning

    // Evaluate on test data
    Evaluation eval = model.evaluate(testIter);
    testIter.reset();

    System.out.println("Epoch " + epoch + ": accuracy = " + eval.accuracy());
}
```

### Epochs vs Iterations

- **Epoch**: One full pass through the entire training dataset.
- **Iteration**: One parameter update, which processes one mini-batch. If your dataset has 10,000 examples and your batch size is 100, then 1 epoch = 100 iterations.

Listeners fire on **iterations** by default. Learning rate schedules can use either.

### Single-Call Training

If you don't need evaluation between epochs:

```java
model.fit(trainIter, numEpochs);  // train for N epochs in one call
```

## Listeners

Listeners hook into the training loop to monitor progress. Attach them before training:

```java
import org.deeplearning4j.optimize.listeners.*;

model.setListeners(
    new ScoreIterationListener(100),  // print loss every 100 iterations
    new PerformanceListener(100)      // print throughput every 100 iterations
);
```

### Available Listeners

| Listener | Class | Purpose |
|----------|-------|---------|
| Score | `ScoreIterationListener(n)` | Print loss every n iterations |
| Performance | `PerformanceListener(n)` | Print examples/sec and batches/sec |
| Evaluative | `EvaluativeListener(testIter, n, InvocationType.EPOCH_END)` | Run evaluation every n epochs |
| Checkpoint | `CheckpointListener.Builder(dir).keepAll().saveEveryNEpochs(5).build()` | Save model checkpoints |
| Collect Scores | `CollectScoresIterationListener(1)` | Collect loss values for plotting |
| Time | `TimeIterationListener(n)` | Log time every n iterations |
| Composable | `ComposableIterationListener(listeners...)` | Combine multiple listeners |

### Checkpoint Listener Example

```java
CheckpointListener checkpoint = new CheckpointListener.Builder("/path/to/checkpoints")
    .keepAll()                     // keep all checkpoints (vs. keepLast(n))
    .saveEveryNEpochs(5)           // save every 5 epochs
    .build();
model.setListeners(checkpoint);
```

## Early Stopping

Train until performance stops improving, saving the best model:

```java
import org.deeplearning4j.earlystopping.*;
import org.deeplearning4j.earlystopping.saver.*;
import org.deeplearning4j.earlystopping.scorecalc.*;
import org.deeplearning4j.earlystopping.termination.*;
import org.deeplearning4j.earlystopping.trainer.*;

EarlyStoppingConfiguration<MultiLayerNetwork> esConf = new EarlyStoppingConfiguration.Builder<MultiLayerNetwork>()
    .epochTerminationConditions(new MaxEpochsTerminationCondition(100))
    .iterationTerminationConditions(new MaxTimeIterationTerminationCondition(60, TimeUnit.MINUTES))
    .scoreCalculator(new DataSetLossCalculator(testIter, true))
    .evaluateEveryNEpochs(1)
    .modelSaver(new LocalFileModelSaver("/path/to/best-model"))
    .build();

EarlyStoppingTrainer trainer = new EarlyStoppingTrainer(esConf, model, trainIter);
EarlyStoppingResult<MultiLayerNetwork> result = trainer.fit();

System.out.println("Best epoch: " + result.getBestModelEpoch());
System.out.println("Best score: " + result.getBestModelScore());
MultiLayerNetwork bestModel = result.getBestModel();
```

Termination conditions: `MaxEpochsTerminationCondition`, `MaxTimeIterationTerminationCondition`, `MaxScoreIterationTerminationCondition`, `ScoreImprovementEpochTerminationCondition`.

## ComputationGraph

`ComputationGraph` supports architectures that `MultiLayerNetwork` cannot: multiple inputs, multiple outputs, skip connections, and arbitrary DAG topologies.

```java
import org.deeplearning4j.nn.conf.ComputationGraphConfiguration;
import org.deeplearning4j.nn.graph.ComputationGraph;

ComputationGraphConfiguration graphConf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .updater(new Adam(1e-3))
    .graphBuilder()
    .addInputs("input")                      // name the input(s)
    .addLayer("dense1", new DenseLayer.Builder()
        .nIn(784).nOut(256)
        .activation(Activation.RELU)
        .build(), "input")                     // connect to "input"
    .addLayer("dense2", new DenseLayer.Builder()
        .nIn(256).nOut(128)
        .activation(Activation.RELU)
        .build(), "dense1")                    // connect to "dense1"
    .addLayer("output", new OutputLayer.Builder(new LossMCXENT())
        .nIn(128).nOut(10)
        .activation(Activation.SOFTMAX)
        .build(), "dense2")                    // connect to "dense2"
    .setOutputs("output")                     // name the output(s)
    .build();

ComputationGraph graph = new ComputationGraph(graphConf);
graph.init();
graph.fit(trainIter, numEpochs);
```

### When to Use ComputationGraph

- **Multiple inputs**: e.g., image + text combined
- **Multiple outputs**: e.g., classification + regression heads
- **Skip/residual connections**: e.g., ResNet-style architectures
- **Shared layers**: same weights used for different inputs (Siamese networks)

### Graph Vertices

In addition to layers, `ComputationGraph` supports vertices that combine or transform data:

| Vertex | Class | Purpose |
|--------|-------|---------|
| Merge | `MergeVertex` | Concatenate inputs along feature dimension |
| Element-wise | `ElementWiseVertex` | Add, subtract, multiply, average, or max element-wise |
| Subset | `SubsetVertex` | Extract a range of features from input |
| Stack | `StackVertex` | Stack along mini-batch dimension |
| Unstack | `UnstackVertex` | Split along mini-batch dimension |
| Reshape | `ReshapeVertex` | Reshape input tensor |
| L2 Normalize | `L2NormalizeVertex` | L2 normalize along a dimension |
| Scale | `ScaleVertex` | Multiply by a scalar |
| Shift | `ShiftVertex` | Add a scalar |
| Preprocessor | `PreprocessorVertex` | Apply an InputPreProcessor |

```java
// Example: skip connection (residual)
.addLayer("conv1", convLayer1, "input")
.addLayer("conv2", convLayer2, "conv1")
.addVertex("add", new ElementWiseVertex(ElementWiseVertex.Op.Add), "conv1", "conv2")
.addLayer("output", outputLayer, "add")
```

## Saving and Loading Models

Save a trained model:

```java
import org.deeplearning4j.util.ModelSerializer;

// Save with normalizer
ModelSerializer.writeModel(model, new File("model.zip"), true);
ModelSerializer.addNormalizerToModel(new File("model.zip"), normalizer);

// Load
MultiLayerNetwork restored = ModelSerializer.restoreMultiLayerNetwork(new File("model.zip"));
NormalizerStandardize restoredNorm = ModelSerializer.restoreNormalizerFromFile(new File("model.zip"));

// For ComputationGraph
ModelSerializer.writeModel(graph, new File("graph.zip"), true);
ComputationGraph restoredGraph = ModelSerializer.restoreComputationGraph(new File("graph.zip"));
```

The saved `.zip` file contains the network configuration (JSON), parameters (binary), and optionally the updater state (for resuming training).

## Complete Training Example

Putting it all together — an MNIST classifier:

```java
// 1. Data
DataSetIterator trainIter = new MnistDataSetIterator(64, true, 42);
DataSetIterator testIter = new MnistDataSetIterator(64, false, 42);

// 2. Network configuration
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .l2(1e-4)
    .list()
    .layer(new DenseLayer.Builder().nIn(784).nOut(256)
        .activation(Activation.RELU).weightInit(WeightInit.RELU).build())
    .layer(new DenseLayer.Builder().nIn(256).nOut(128)
        .activation(Activation.RELU).weightInit(WeightInit.RELU).build())
    .layer(new OutputLayer.Builder(new LossMCXENT()).nIn(128).nOut(10)
        .activation(Activation.SOFTMAX).build())
    .build();

// 3. Build model
MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
model.setListeners(new ScoreIterationListener(100));

// 4. Train
for (int epoch = 0; epoch < 15; epoch++) {
    model.fit(trainIter);
    trainIter.reset();

    Evaluation eval = model.evaluate(testIter);
    testIter.reset();
    System.out.printf("Epoch %d: accuracy=%.4f, f1=%.4f%n", epoch, eval.accuracy(), eval.f1());
}

// 5. Save
ModelSerializer.writeModel(model, new File("mnist-model.zip"), true);
```

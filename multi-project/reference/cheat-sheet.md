---
title: "Cheat Sheet"
description: "Quick reference cheat sheet for Deeplearning4j — common configurations, layer types, and API patterns"
---

# Deeplearning4j Cheat Sheet

Quick reference for the most common DL4J API patterns. Entries are organized by task.

---

## Network Builder Patterns

### MultiLayerNetwork (Sequential)

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(123)
    .updater(new Adam(1e-3))
    .weightInit(WeightInit.XAVIER)
    .l2(1e-4)
    .list()
    .layer(new DenseLayer.Builder().nOut(256).activation(Activation.RELU).build())
    .layer(new DropoutLayer(0.5))
    .layer(new DenseLayer.Builder().nOut(128).activation(Activation.RELU).build())
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.MCXENT)
           .activation(Activation.SOFTMAX).nOut(10).build())
    .setInputType(InputType.feedForward(784))  // infers nIn automatically
    .build();

MultiLayerNetwork net = new MultiLayerNetwork(conf);
net.init();
net.setListeners(new ScoreIterationListener(10));
```

### ComputationGraph (DAG)

```java
ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(123)
    .updater(new Adam(1e-3))
    .graphBuilder()
    .addInputs("input")
    .addLayer("dense1",
        new DenseLayer.Builder().nIn(784).nOut(256).activation(Activation.RELU).build(),
        "input")
    .addLayer("dense2",
        new DenseLayer.Builder().nIn(256).nOut(128).activation(Activation.RELU).build(),
        "dense1")
    .addLayer("output",
        new OutputLayer.Builder(LossFunctions.LossFunction.MCXENT)
            .nIn(128).nOut(10).activation(Activation.SOFTMAX).build(),
        "dense2")
    .setOutputs("output")
    .build();

ComputationGraph net = new ComputationGraph(conf);
net.init();
```

### Training Loop

```java
for (int epoch = 0; epoch < numEpochs; epoch++) {
    net.fit(trainIterator);       // MLN: single call fits one epoch
    trainIterator.reset();

    Evaluation eval = net.evaluate(testIterator);
    System.out.println("Epoch " + epoch + ": " + eval.accuracy());
    testIterator.reset();
}
```

---

## Layer Quick Reference

### Feed-Forward Layers

| Layer | Key Config Options |
|---|---|
| `DenseLayer` | `nIn`, `nOut`, `activation` |
| `EmbeddingLayer` | `nIn` (vocab size), `nOut` (embedding dim) — input as integer index |
| `EmbeddingSequenceLayer` | same as Embedding but outputs 3D (sequence) |
| `ActivationLayer` | `activation` only — no weights |
| `DropoutLayer` | `dropOut` rate (fraction retained) |
| `BatchNormalization` | `nIn`, `decay`, `eps` |

```java
// Dense with batch norm pattern
.layer(new DenseLayer.Builder().nOut(256).activation(Activation.IDENTITY).build())
.layer(new BatchNormalization.Builder().nOut(256).build())
.layer(new ActivationLayer(Activation.RELU))
```

### Output Layers

| Layer | Use Case |
|---|---|
| `OutputLayer` | Multi-class or regression; has built-in dense connection |
| `LossLayer` | No weights; input size must equal output size |
| `RnnOutputLayer` | Time-series classification/regression (3D output) |
| `CnnLossLayer` | Per-pixel prediction (segmentation); no weights |
| `Yolo2OutputLayer` | Object detection |

### Convolutional Layers

```java
// Standard 2D convolution
new ConvolutionLayer.Builder(kernelH, kernelW)
    .nIn(channels).nOut(filters)
    .stride(1, 1).padding(0, 0)
    .activation(Activation.RELU)
    .build()

// 2D max pooling
new SubsamplingLayer.Builder(PoolingType.MAX)
    .kernelSize(2, 2).stride(2, 2)
    .build()

// Batch norm after conv (channels first)
new BatchNormalization.Builder().build()
```

Use `.setInputType(InputType.convolutional(height, width, channels))` on the network builder to avoid manually setting `nIn` on conv layers.

### Recurrent Layers

| Layer | Notes |
|---|---|
| `LSTM` | Standard LSTM; supports CuDNN |
| `GravesLSTM` | LSTM with peephole connections; no CuDNN support |
| `SimpleRnn` | Vanilla RNN; rarely used for long sequences |
| `Bidirectional` | Wrapper around any RNN layer |
| `LastTimeStep` | Wrapper that extracts the last time step from a 3D RNN output |

```java
// Bidirectional LSTM
new Bidirectional(Bidirectional.Mode.CONCAT,
    new LSTM.Builder().nIn(128).nOut(64).build())

// Sequence → vector
new LastTimeStep(new LSTM.Builder().nIn(128).nOut(64).build())
```

### Utility Layers

| Layer | Purpose |
|---|---|
| `GlobalPoolingLayer` | Collapses spatial or time dimensions (max, avg, sum) |
| `LocalResponseNormalization` | LRN for older CNN architectures |
| `FrozenLayer` | Wraps a layer and freezes its parameters (transfer learning) |
| `ZeroPaddingLayer` | Pad spatial dimensions with zeros |

### Graph Vertices (ComputationGraph Only)

| Vertex | Purpose |
|---|---|
| `ElementWiseVertex(Op.Add)` | Element-wise addition (residual connections) |
| `MergeVertex` | Concatenate along channel/feature dimension |
| `SubsetVertex` | Extract a slice along dimension 1 |
| `ScaleVertex(scalar)` | Multiply all activations by a constant |
| `L2NormalizeVertex` | Normalize each example to unit L2 norm |
| `UnstackVertex` | Split minibatch along batch dimension |

```java
// Residual connection example
.addLayer("conv1", new ConvolutionLayer.Builder(3,3).nOut(64).build(), "input")
.addLayer("conv2", new ConvolutionLayer.Builder(3,3).nOut(64).build(), "conv1")
.addVertex("residual", new ElementWiseVertex(ElementWiseVertex.Op.Add), "input", "conv2")
```

---

## Updater (Optimizer) Selection

```java
// Adam — best default choice for most tasks
.updater(new Adam(1e-3))
.updater(new Adam(new ExponentialSchedule(ScheduleType.ITERATION, 1e-3, 0.995)))

// Nesterov momentum
.updater(new Nesterovs(1e-2, 0.9))

// RMSProp — works well for RNNs
.updater(new RmsProp(1e-3))

// AdaGrad — adapts per-parameter; decays fast
.updater(new AdaGrad(1e-2))

// SGD — simple, needs careful LR tuning
.updater(new Sgd(1e-2))
```

Typical starting learning rates: `1e-3` for Adam/RMSProp, `1e-2` for Nesterovs/SGD.

---

## Learning Rate Schedules

Pass any `ISchedule` as the learning rate argument to an updater:

```java
// Exponential decay: lr(i) = lr0 * gamma^i
new ExponentialSchedule(ScheduleType.ITERATION, 0.001, 0.999)

// Step decay: lr halves every 1000 iterations
new StepSchedule(ScheduleType.ITERATION, 0.01, 0.5, 1000)

// Inverse: lr(i) = lr0 / (1 + gamma*i)^power
new InverseSchedule(ScheduleType.EPOCH, 0.1, 0.01, 0.75)

// Manual schedule via map
new MapSchedule.Builder(ScheduleType.EPOCH)
    .add(0, 1e-3)
    .add(10, 5e-4)
    .add(20, 1e-4)
    .build()
```

---

## Activation and Loss Pairings

| Task | Output Activation | Loss Function |
|---|---|---|
| Multi-class classification | `SOFTMAX` | `MCXENT` or `NEGATIVELOGLIKELIHOOD` |
| Binary classification (single output) | `SIGMOID` | `XENT` |
| Multi-label classification | `SIGMOID` | `XENT` |
| Regression | `IDENTITY` | `MSE` |
| Regression (bounded 0–1) | `SIGMOID` | `MSE` |
| Autoencoder reconstruction | `SIGMOID` or `IDENTITY` | `MSE` or `RECONSTRUCTION_CROSSENTROPY` |

---

## Weight Initialization

```java
.weightInit(WeightInit.XAVIER)          // general; tanh/sigmoid networks
.weightInit(WeightInit.RELU)            // relu/leakyrelu networks
.weightInit(WeightInit.XAVIER_UNIFORM)  // uniform variant of Xavier
.weightInit(WeightInit.NORMAL)          // LECUN_NORMAL; SELU networks
.weightInit(WeightInit.ZERO)            // debugging only
.weightInit(new NormalDistribution(0, 0.01))  // custom distribution
```

---

## Regularization

```java
// Global (applied to all layers unless overridden)
new NeuralNetConfiguration.Builder()
    .l2(1e-4)
    .l1(0)
    .dropOut(0.5)          // 50% retention probability

// Per-layer override
new DenseLayer.Builder()
    .l2(1e-3)              // stronger regularization on this layer
    .dropOut(0.3)
    .build()

// Weight constraints
.constrainWeights(new MaxNormConstraint(3.0, 1))   // per incoming unit

// Gradient clipping
.gradientNormalization(GradientNormalization.ClipElementWiseAbsoluteValue)
.gradientNormalizationThreshold(1.0)
```

---

## Data Pipeline

### CSV Data

```java
RecordReader rr = new CSVRecordReader(1, ','); // skip 1 header line
rr.initialize(new FileSplit(new File("data.csv")));

DataSetIterator iter = new RecordReaderDataSetIterator.Builder(rr, batchSize)
    .classification(labelColumnIndex, numClasses)
    .build();
```

### Image Data

```java
File imageDir = new File("/path/to/images"); // subdirs = class names
FileSplit split = new FileSplit(imageDir, NativeImageLoader.ALLOWED_FORMATS, rng);

ImageRecordReader rr = new ImageRecordReader(height, width, channels, labelMaker);
rr.initialize(split);

DataSetIterator iter = new RecordReaderDataSetIterator(rr, batchSize, 1, numClasses);
```

### Normalization

```java
// Zero-mean unit-variance (tabular data)
NormalizerStandardize normalizer = new NormalizerStandardize();
normalizer.fit(trainIter);
trainIter.setPreProcessor(normalizer);
testIter.setPreProcessor(normalizer);

// Scale images to [0, 1]
DataNormalization imageNorm = new ImagePreProcessingScaler(0, 1);
imageNorm.fit(trainIter);
trainIter.setPreProcessor(imageNorm);
testIter.setPreProcessor(imageNorm);

// MinMax to [0, 1]
NormalizerMinMaxScaler minMax = new NormalizerMinMaxScaler(0, 1);
minMax.fit(trainIter);
trainIter.setPreProcessor(minMax);
```

Save and restore normalizer with model:

```java
ModelSerializer.addNormalizerToModel(modelFile, normalizer);
NormalizerStandardize loaded =
    ModelSerializer.restoreNormalizerFromFile(modelFile);
```

### MultiDataSet (Multiple Inputs/Outputs)

```java
MultiDataSetIterator iter = new RecordReaderMultiDataSetIterator.Builder(batchSize)
    .addReader("features", featuresReader)
    .addReader("labels",   labelsReader)
    .addInput("features", 0, numFeatures - 1)
    .addOutputOneHot("labels", labelColIdx, numClasses)
    .build();
```

---

## Evaluation

### Classification

```java
Evaluation eval = net.evaluate(testIter);
System.out.println(eval.stats());
// Prints accuracy, precision, recall, F1, confusion matrix

// Specific metrics
double acc = eval.accuracy();
double f1  = eval.f1();
```

### Regression

```java
RegressionEvaluation eval = net.evaluateRegression(testIter);
System.out.println(eval.stats());
// Prints MSE, MAE, RMSE, R^2 per output column
```

### Binary Classification (ROC / AUC)

```java
ROC roc = net.evaluateROC(testIter, 100);  // 100 thresholds
System.out.println("AUC:   " + roc.calculateAUC());
System.out.println("AUPRC: " + roc.calculateAUCPR());
```

### Multi-class ROC

```java
ROCMultiClass roc = net.evaluateROCMultiClass(testIter, 100);
System.out.println("AUC class 0: " + roc.calculateAUC(0));
```

### ComputationGraph

```java
// evaluateROC, evaluateRegression, evaluate all available on ComputationGraph
// specify output index for multi-output networks
Evaluation eval = cgNet.evaluate(testIter, "outputLayerName");
```

---

## Model Save and Load

```java
// Save (includes updater state by default)
net.save(new File("model.zip"));

// Load for continued training (restores updater state)
MultiLayerNetwork net = MultiLayerNetwork.load(new File("model.zip"), true);

// Load for inference only
MultiLayerNetwork net = MultiLayerNetwork.load(new File("model.zip"), false);

// ComputationGraph
cgNet.save(new File("cgmodel.zip"));
ComputationGraph cgNet =
    ComputationGraph.load(new File("cgmodel.zip"), true);
```

Using ModelSerializer directly:

```java
ModelSerializer.writeModel(net, new File("model.zip"), true);
MultiLayerNetwork net =
    ModelSerializer.restoreMultiLayerNetwork(new File("model.zip"));
ComputationGraph cg =
    ModelSerializer.restoreComputationGraph(new File("cgmodel.zip"));
```

The model file is a ZIP archive. You can inspect its contents with any ZIP tool.

---

## Transfer Learning

```java
// Freeze all layers except the output
TransferLearning.Builder tlBuilder = new TransferLearning.Builder(pretrainedNet)
    .fineTuneConfiguration(new FineTuneConfiguration.Builder()
        .updater(new Adam(1e-4))
        .build())
    .setFeatureExtractor("dense2")   // freeze up to and including this layer
    .removeOutputLayer()
    .addLayer(new OutputLayer.Builder(LossFunctions.LossFunction.MCXENT)
              .nIn(256).nOut(newNumClasses)
              .activation(Activation.SOFTMAX).build());

MultiLayerNetwork tNet = tlBuilder.build();
```

---

## Training Listeners

```java
// Print score every N iterations
net.setListeners(new ScoreIterationListener(10));

// Training UI
UIServer ui = UIServer.getInstance();
StatsStorage ss = new InMemoryStatsStorage();
ui.attach(ss);
net.setListeners(new StatsListener(ss));

// Checkpoint every epoch
net.setListeners(new CheckpointListener.Builder("/tmp/checkpoints")
    .keepLastAndBest()
    .saveEveryNEpochs(1)
    .build());

// Performance diagnostics
net.setListeners(new PerformanceListener(10, true));

// Evaluate on test set every epoch
net.setListeners(new EvaluativeListener(testIter, 1, InvocationType.EPOCH_END));
```

---

## Early Stopping

```java
EarlyStoppingConfiguration<MultiLayerNetwork> esConf =
    new EarlyStoppingConfiguration.Builder<MultiLayerNetwork>()
        .epochTerminationConditions(
            new MaxEpochsTerminationCondition(100),
            new ScoreImprovementEpochTerminationCondition(10))
        .iterationTerminationConditions(
            new MaxTimeIterationTerminationCondition(2, TimeUnit.HOURS))
        .scoreCalculator(new DataSetLossCalculator(valIter, true))
        .evaluateEveryNEpochs(1)
        .modelSaver(new LocalFileModelSaver("/tmp/es"))
        .build();

EarlyStoppingResult<MultiLayerNetwork> result =
    new EarlyStoppingTrainer(esConf, netConf, trainIter).fit();

MultiLayerNetwork best = result.getBestModel();
```

---

## Useful ND4J Snippets

```java
// Create arrays
INDArray a = Nd4j.rand(DataType.FLOAT, 32, 128);
INDArray b = Nd4j.zeros(DataType.FLOAT, 32, 128);
INDArray c = Nd4j.ones(DataType.FLOAT, 32, 128);

// Shapes
long[] shape = a.shape();   // [32, 128]
long rows = a.rows();       // 32
long cols = a.columns();    // 128

// Math
INDArray sum   = a.add(b);
INDArray mm    = a.mmul(b.transpose());
INDArray relu  = Transforms.relu(a);
INDArray norm  = a.norm2(1);  // L2 norm along dim 1

// Indexing
INDArray row0  = a.getRow(0);
INDArray col5  = a.getColumn(5);
INDArray slice = a.get(NDArrayIndex.interval(0, 16), NDArrayIndex.all());

// Convert to/from Java
double[][] java2d = a.toDoubleMatrix();
INDArray fromJava = Nd4j.create(java2d);

// Argmax / max
INDArray argmax = a.argMax(1);   // index of max along columns
INDArray maxVal = a.max(1);      // max value along columns
```

---

## Common Import Packages

```java
import org.deeplearning4j.nn.conf.*;
import org.deeplearning4j.nn.conf.layers.*;
import org.deeplearning4j.nn.conf.layers.recurrent.*;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.deeplearning4j.nn.graph.ComputationGraph;
import org.deeplearning4j.optimize.listeners.*;
import org.deeplearning4j.datasets.datavec.*;
import org.deeplearning4j.datasets.iterator.*;
import org.deeplearning4j.earlystopping.*;
import org.deeplearning4j.earlystopping.trainer.*;
import org.deeplearning4j.earlystopping.termination.*;
import org.deeplearning4j.earlystopping.scorecalc.*;
import org.deeplearning4j.util.ModelSerializer;
import org.deeplearning4j.ui.api.UIServer;
import org.deeplearning4j.ui.model.stats.StatsListener;
import org.deeplearning4j.ui.model.storage.InMemoryStatsStorage;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.lossfunctions.LossFunctions;
import org.nd4j.linalg.learning.config.*;
import org.nd4j.linalg.schedule.*;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;
import org.nd4j.linalg.dataset.api.preprocessor.*;
import org.nd4j.evaluation.classification.*;
import org.nd4j.evaluation.regression.*;
import org.datavec.api.records.reader.*;
import org.datavec.api.records.reader.impl.csv.*;
import org.datavec.api.split.*;
import org.datavec.image.recordreader.*;
```

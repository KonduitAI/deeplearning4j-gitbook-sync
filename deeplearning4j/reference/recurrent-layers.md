---
title: "Recurrent Layers"
description: "RNN layers in Deeplearning4j — LSTM, GRU, Bidirectional wrapper, masking, TBPTT, and sequence data handling"
---

## Overview

Deeplearning4j provides a complete set of recurrent neural network (RNN) layers for processing sequential and time-series data. The framework supports variable-length sequences via masking, truncated backpropagation through time (TBPTT) for long sequences, and step-by-step inference for online/streaming use cases.

This page assumes familiarity with RNN concepts (LSTM gates, backpropagation through time, sequence labelling). For an introduction to RNNs see the [deep learning conceptual overview](../../../core-concepts).

---

## Data Format

All RNN layers in DL4J use the format:

```
[minibatch, features, timeSteps]
```

- Dimension 0: minibatch size
- Dimension 1: number of features per time step
- Dimension 2: sequence length (number of time steps)

This is the "channels-first" or NCL (batch, channels, length) layout. This applies to both input and output activations.

Example: a minibatch of 32 sequences, each with 10 features over 100 time steps would have shape `[32, 10, 100]`.

For `RnnOutputLayer` labels used in classification, the shape is `[minibatch, numClasses, timeSteps]`.

---

## Available Layers

### LSTM

**Class:** `org.deeplearning4j.nn.conf.layers.LSTM`
**Source:** [LSTM.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LSTM.java)

Long Short-Term Memory layer without peephole connections. This is the preferred LSTM implementation in M2.1 — it supports CuDNN acceleration on NVIDIA GPUs automatically.

#### Builder Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `nIn` | int | required | Input feature size |
| `nOut` | int | required | Hidden state (cell) size |
| `activation` | Activation | TANH | Activation for cell state |
| `gateActivationFunction` | Activation | SIGMOID | Gate activation (should be bounded 0-1) |
| `forgetGateBiasInit` | double | 1.0 | Initial forget gate bias; values 1-5 help retain longer dependencies |
| `weightInit` | WeightInit | global | Weight initializer |
| `l1` / `l2` | double | global | Regularization |
| `dropOut` | double | global | Input dropout |

#### Example

```java
import org.deeplearning4j.nn.conf.layers.LSTM;
import org.nd4j.linalg.activations.Activation;

new LSTM.Builder()
    .nIn(64)
    .nOut(128)
    .activation(Activation.TANH)
    .gateActivationFunction(Activation.SIGMOID)
    .forgetGateBiasInit(1.0)
    .build()
```

---

### GravesLSTM

**Class:** `org.deeplearning4j.nn.conf.layers.GravesLSTM`
**Source:** [GravesLSTM.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/GravesLSTM.java)

LSTM with peephole connections as described in Graves (2013) "Supervised Sequence Labelling with Recurrent Neural Networks". Peephole connections give gate computations direct access to the cell state.

**Note:** `GravesLSTM` does not support CuDNN acceleration. Use `LSTM` for GPU-optimized training unless you specifically need peephole connections.

#### Builder Parameters

Same as `LSTM`, plus:

| Parameter | Type | Description |
|-----------|------|-------------|
| `forgetGateBiasInit` | double | Forget gate bias initialization |
| `gateActivationFunction` | Activation | Bounded gate activation |

#### Example

```java
new GravesLSTM.Builder()
    .nIn(32)
    .nOut(64)
    .activation(Activation.TANH)
    .gateActivationFunction(Activation.HARDSIGMOID)
    .build()
```

---

### SimpleRnn

**Class:** `org.deeplearning4j.nn.conf.layers.recurrent.SimpleRnn`
**Source:** [SimpleRnn.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/recurrent/SimpleRnn.java)

Vanilla Elman recurrent network. Computes:

```
h_t = activation(W_in * x_t + W_rec * h_{t-1} + b)
```

Very fast to compute but struggles with long-term dependencies. Recommended only when temporal dependencies span a few steps.

#### Example

```java
import org.deeplearning4j.nn.conf.layers.recurrent.SimpleRnn;

new SimpleRnn.Builder()
    .nIn(32)
    .nOut(64)
    .activation(Activation.TANH)
    .build()
```

---

### Bidirectional (Wrapper)

**Class:** `org.deeplearning4j.nn.conf.layers.recurrent.Bidirectional`
**Source:** [Bidirectional.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/recurrent/Bidirectional.java)

Wraps any unidirectional RNN layer to make it bidirectional. The layer runs two independent copies of the wrapped layer — one forward, one backward — and combines their outputs.

#### Combination Modes

| Mode | Output Size | Description |
|------|-------------|-------------|
| `ADD` | nOut | Element-wise addition of forward and backward activations |
| `MUL` | nOut | Element-wise multiplication |
| `AVERAGE` | nOut | `0.5 * (forward + backward)` |
| `CONCAT` | 2 * nOut | Concatenation along feature dimension |

#### Example

```java
import org.deeplearning4j.nn.conf.layers.recurrent.Bidirectional;

// Bidirectional LSTM with concatenated outputs (output size = 2 * nOut = 256)
new Bidirectional(Bidirectional.Mode.CONCAT,
    new LSTM.Builder().nIn(64).nOut(128).activation(Activation.TANH).build())
```

#### In a MultiLayerNetwork

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .list()
    .layer(new Bidirectional(Bidirectional.Mode.CONCAT,
        new LSTM.Builder().nIn(100).nOut(64).activation(Activation.TANH).build()))
    // CONCAT mode: output is 64 * 2 = 128 features
    .layer(new RnnOutputLayer.Builder(LossFunctions.LossFunction.MCXENT)
        .nIn(128).nOut(numClasses).activation(Activation.SOFTMAX).build())
    .build();
```

---

### LastTimeStep (Wrapper)

**Class:** `org.deeplearning4j.nn.conf.layers.recurrent.LastTimeStep`
**Source:** [LastTimeStep.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/recurrent/LastTimeStep.java)

Wraps any RNN (or Conv1D) layer and extracts only the output at the last valid time step, returning a 2D array `[minibatch, nOut]` instead of the full 3D sequence `[minibatch, nOut, timeSteps]`. Mask-aware: if masking arrays are present, it returns the last non-masked time step for each example independently.

Use `LastTimeStep` when you want sequence-to-vector encoding (many-to-one).

#### Example — Sequence Classification

```java
.layer(new LastTimeStep(
    new LSTM.Builder().nIn(64).nOut(128).activation(Activation.TANH).build()))
// Output is now 2D: [mb, 128]
.layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
    .nIn(128).nOut(numClasses).activation(Activation.SOFTMAX).build())
```

---

### RnnOutputLayer

**Class:** `org.deeplearning4j.nn.conf.layers.RnnOutputLayer`
**Source:** [RnnOutputLayer.java](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/RnnOutputLayer.java)

The RNN counterpart of `OutputLayer`. Handles time-distributed loss computation. Input and label shapes are both `[minibatch, size, timeSteps]`.

- Supports mask arrays for variable-length sequence training.
- Also works for Conv1D output (same shape convention).

#### Example

```java
new RnnOutputLayer.Builder(LossFunctions.LossFunction.MCXENT)
    .nIn(128)
    .nOut(numClasses)
    .activation(Activation.SOFTMAX)
    .build()
```

---

### RnnLossLayer

**Class:** `org.deeplearning4j.nn.conf.layers.RnnLossLayer`

Time-distributed loss layer without learnable parameters. Use when the previous layer already outputs the correct number of features and you only need a loss function applied across time.

```java
new RnnLossLayer.Builder(LossFunctions.LossFunction.MCXENT)
    .activation(Activation.SOFTMAX)
    .build()
```

---

## Truncated Backpropagation Through Time (TBPTT)

Standard backpropagation through time (BPTT) for long sequences (>500 steps) is computationally expensive and can suffer from vanishing gradients. TBPTT breaks sequences into shorter segments and performs a forward-backward pass on each segment, giving more frequent parameter updates.

### Configuration

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    // ... global settings ...
    .list()
    .layer(new LSTM.Builder().nIn(64).nOut(128).build())
    .layer(new RnnOutputLayer.Builder(LossFunctions.LossFunction.MCXENT)
        .nIn(128).nOut(10).activation(Activation.SOFTMAX).build())
    .backpropType(BackpropType.TruncatedBPTT)
    .tBPTTLength(100)      // segment length; typically 50-200
    .build();
```

Setting | Description
--------|------------
`BackpropType.Standard` | Full BPTT (default)
`BackpropType.TruncatedBPTT` | TBPTT with segments of `.tBPTTLength(n)` steps
`.tBPTTLength(int)` | Number of time steps per TBPTT segment (default: 20)

**Guidelines:**
- Use TBPTT when sequences are longer than ~200 time steps.
- `tBPTTLength` should be a fraction of the total sequence length (e.g., 100-200 for 1000-step sequences).
- Variable-length sequences in the same minibatch work correctly with TBPTT.
- TBPTT can learn shorter dependencies than full BPTT because gradients don't flow beyond the segment boundary.

---

## Masking: Variable-Length Sequences

DL4J supports one-to-many, many-to-one, and variable-length many-to-many training via padding and mask arrays.

### Padding and Mask Arrays

When sequences in a minibatch have different lengths, shorter sequences are padded with zeros to match the longest. Mask arrays (shape `[minibatch, timeSteps]` with values 0 or 1) record which time steps are real data vs. padding.

```
Example mask for 3 sequences with lengths [4, 2, 3] in a batch of length 4:
[[1, 1, 1, 1],   <- sequence 0: all 4 steps are real
 [1, 1, 0, 0],   <- sequence 1: only first 2 steps are real
 [1, 1, 1, 0]]   <- sequence 2: first 3 steps are real
```

The mask array is stored in the `DataSet` object:

```java
DataSet ds = new DataSet(features, labels, featuresMask, labelsMask);
```

When a `DataSet` contains mask arrays, `MultiLayerNetwork.fit()` and evaluation methods automatically use them.

### Many-to-One (Sequence Classification)

For classifying an entire sequence with a single label, use a labels mask with a single `1` at the last valid time step:

```java
// Using LastTimeStep wrapper (preferred for M2.1):
.layer(new LastTimeStep(
    new LSTM.Builder().nIn(64).nOut(128).build()))
.layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
    .nIn(128).nOut(numClasses).activation(Activation.SOFTMAX).build())
```

Or with `RnnOutputLayer` and an output mask:

```java
// The labels mask has a single 1 at the last time step per sequence
// The loss is only computed at that time step
```

### Evaluation with Masks

```java
import org.nd4j.evaluation.classification.Evaluation;

Evaluation eval = new Evaluation(numClasses);
INDArray predictions = model.output(features);
eval.evalTimeSeries(labels, predictions, outputMaskArray);
System.out.println(eval.stats());
```

### Loading Variable-Length Data

```java
import org.deeplearning4j.datasets.datavec.SequenceRecordReaderDataSetIterator;

SequenceRecordReader featureReader = new CSVSequenceRecordReader(0, ",");
SequenceRecordReader labelReader   = new CSVSequenceRecordReader(0, ",");

featureReader.initialize(new NumberedFileInputSplit("/data/features_%d.csv", 0, 99));
labelReader.initialize(new NumberedFileInputSplit("/data/labels_%d.csv", 0, 99));

// ALIGN_END: align the last label time step with the last feature time step
DataSetIterator iter = new SequenceRecordReaderDataSetIterator(
    featureReader, labelReader,
    miniBatchSize, numClasses, false,
    SequenceRecordReaderDataSetIterator.AlignmentMode.ALIGN_END);
```

Alignment modes:

| Mode | Description |
|------|-------------|
| `ALIGN_END` | Align the end of sequences (many-to-one: label at the last step) |
| `ALIGN_START` | Align the start of sequences (one-to-many: label at the first step) |

---

## Combining RNN with Other Layer Types

### RNN + Dense (Many-to-One Classification)

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .list()
    .layer(new LSTM.Builder().nIn(inputSize).nOut(128).build())
    .layer(new LSTM.Builder().nIn(128).nOut(64).build())
    // LastTimeStep extracts [mb, 64] from [mb, 64, T]
    .layer(new LastTimeStep(new SimpleRnn.Builder().nIn(64).nOut(64).build()))
    .layer(new DenseLayer.Builder().nIn(64).nOut(32).activation(Activation.RELU).build())
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nIn(32).nOut(numClasses).activation(Activation.SOFTMAX).build())
    .build();
```

### CNN + RNN (Video Classification)

Convolutional layers process each frame independently; the RNN processes the sequence of frame features. DL4J automatically inserts the required `CnnToRnnPreProcessor`:

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .list()
    .layer(new ConvolutionLayer.Builder(3, 3).nIn(3).nOut(32)
        .activation(Activation.RELU).build())
    .layer(new SubsamplingLayer.Builder(PoolingType.MAX)
        .kernelSize(2, 2).stride(2, 2).build())
    .layer(new ConvolutionLayer.Builder(3, 3).nOut(64)
        .activation(Activation.RELU).build())
    .layer(new SubsamplingLayer.Builder(PoolingType.MAX)
        .kernelSize(2, 2).stride(2, 2).build())
    // Pre-processor inserted automatically by setInputType:
    // CnnToFeedForwardPreProcessor -> FeedForwardToRnnPreProcessor
    .layer(new LSTM.Builder().nOut(256).activation(Activation.TANH).build())
    .layer(new RnnOutputLayer.Builder(LossFunctions.LossFunction.MCXENT)
        .nIn(256).nOut(numClasses).activation(Activation.SOFTMAX).build())
    .setInputType(InputType.convolutional(frameHeight, frameWidth, channels))
    .build();
```

### Manual Pre-Processor Insertion

If automatic pre-processor detection doesn't work for a custom topology:

```java
// Add preprocessor between layers 2 and 3 explicitly
.inputPreProcessor(3, new RnnToFeedForwardPreProcessor())
// Or:
.inputPreProcessor(3, new FeedForwardToRnnPreProcessor())
.inputPreProcessor(3, new CnnToRnnPreProcessor(height, width, channels))
```

---

## Step-by-Step Inference (rnnTimeStep)

Use `rnnTimeStep()` for real-time or online inference where preserving RNN hidden state between calls is important.

```java
// Initialize: clear any previous hidden state
model.rnnClearPreviousState();

// Feed one time step at a time (input shape: [numExamples, nIn])
INDArray singleStepInput = Nd4j.create(1, nIn);   // one example, one step
INDArray output = model.rnnTimeStep(singleStepInput);
// output shape: [1, nOut] (2D — single step returns 2D, not 3D)

// The hidden state is automatically stored between calls
INDArray nextOutput = model.rnnTimeStep(nextStepInput);

// For a new independent sequence, always clear state first
model.rnnClearPreviousState();
```

Multi-step input is also supported:

```java
// Feed 10 steps at once, preserving state across calls
INDArray tenStepsInput = Nd4j.create(1, nIn, 10);  // [1, nIn, 10]
INDArray tenStepsOutput = model.rnnTimeStep(tenStepsInput);
// output shape: [1, nOut, 10]
```

Managing state manually (e.g., for serialization):

```java
// Save state after processing some steps
Map<String, INDArray> layerState = model.rnnGetPreviousState(layerIndex);

// Later, restore and continue
model.rnnSetPreviousState(layerIndex, layerState);
```

---

## Complete Example: Sequence Classification with LSTM

```java
import org.deeplearning4j.nn.conf.*;
import org.deeplearning4j.nn.conf.layers.*;
import org.deeplearning4j.nn.conf.layers.recurrent.*;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.learning.config.Adam;
import org.nd4j.linalg.lossfunctions.LossFunctions;

int inputSize  = 32;
int hiddenSize = 128;
int numClasses = 5;
int numEpochs  = 10;

MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .weightInit(WeightInit.XAVIER)
    .l2(1e-5)
    .list()
    // Stack two LSTM layers
    .layer(new LSTM.Builder()
        .nIn(inputSize).nOut(hiddenSize)
        .activation(Activation.TANH)
        .build())
    .layer(new LSTM.Builder()
        .nIn(hiddenSize).nOut(hiddenSize / 2)
        .activation(Activation.TANH)
        .build())
    // Extract last time step: [mb, hiddenSize/2, T] -> [mb, hiddenSize/2]
    .layer(new LastTimeStep(
        new SimpleRnn.Builder().nIn(hiddenSize / 2).nOut(hiddenSize / 2).build()))
    // Classification output
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nIn(hiddenSize / 2).nOut(numClasses)
        .activation(Activation.SOFTMAX)
        .build())
    .build();

MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
model.setListeners(new ScoreIterationListener(50));

DataSetIterator trainIter = /* SequenceRecordReaderDataSetIterator or similar */;
model.fit(trainIter, numEpochs);
```

---

## Key API Summary

| Method | Description |
|--------|-------------|
| `fit(DataSetIterator)` | Train with full sequence data |
| `output(INDArray)` | Forward pass, returns full output sequence `[mb, nOut, T]` |
| `rnnTimeStep(INDArray)` | Step-by-step inference with state retention |
| `rnnClearPreviousState()` | Reset hidden state for all RNN layers |
| `rnnGetPreviousState(int)` | Get hidden state for a specific layer |
| `rnnSetPreviousState(int, Map)` | Restore hidden state for a specific layer |
| `evaluate(DataSetIterator)` | Classification evaluation |
| `Evaluation.evalTimeSeries(...)` | Evaluation with mask arrays for variable-length sequences |

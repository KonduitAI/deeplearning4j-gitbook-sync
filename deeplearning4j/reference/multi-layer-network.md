---
title: "MultiLayerNetwork"
description: "The MultiLayerNetwork API — building, configuring, training, evaluating, and using sequential neural networks"
---

## Overview

`MultiLayerNetwork` is the primary API for building sequential (stack-of-layers) neural networks in Eclipse Deeplearning4j. It covers the vast majority of practical use cases: feedforward classifiers and regressors, CNNs for image recognition, and RNNs for sequence data.

Use `MultiLayerNetwork` when:
- Your network has a single input and a single output.
- Layers connect in a straight chain: input -> layer 0 -> layer 1 -> ... -> output.

Use `ComputationGraph` instead when you need skip/residual connections, multiple inputs, or multiple outputs.

---

## Building a Network

### NeuralNetConfiguration.Builder (M2.1 API)

All network configuration starts with `NeuralNetConfiguration.Builder`. In M2.1 the global updater, weight initializer, regularization, and data type are set here and apply to every layer unless overridden at the layer level.

```java
import org.deeplearning4j.nn.conf.MultiLayerConfiguration;
import org.deeplearning4j.nn.conf.NeuralNetConfiguration;
import org.deeplearning4j.nn.conf.layers.DenseLayer;
import org.deeplearning4j.nn.conf.layers.OutputLayer;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.deeplearning4j.nn.weights.WeightInit;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.learning.config.Adam;
import org.nd4j.linalg.lossfunctions.LossFunctions;

MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)                          // use 32-bit floats
    .weightInit(WeightInit.XAVIER)
    .updater(new Adam(1e-3))                           // M2.1: pass lr to constructor
    .l2(1e-4)                                          // L2 regularization
    .list()
    .layer(new DenseLayer.Builder()
        .nIn(784).nOut(256)
        .activation(Activation.RELU)
        .build())
    .layer(new DenseLayer.Builder()
        .nIn(256).nOut(128)
        .activation(Activation.RELU)
        .build())
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nIn(128).nOut(10)
        .activation(Activation.SOFTMAX)
        .build())
    .build();
```

Key M2.1 differences from older API:
- `dataType(DataType.FLOAT)` replaces the old global float/double flags.
- `new Adam(lr)` — the updater is constructed directly with the learning rate. No separate `.learningRate()` call.
- `.pretrain(false).backprop(true)` is **removed** — standard backprop is always used.
- Layer indices are optional: use `.layer(layerConf)` without an index for automatic ordering.

### Global Builder Options

| Method | Description |
|--------|-------------|
| `.seed(long)` | Random seed for reproducibility |
| `.dataType(DataType)` | Numeric precision: `DataType.FLOAT` or `DataType.DOUBLE` |
| `.weightInit(WeightInit)` | Default weight initializer for all layers |
| `.updater(IUpdater)` | Optimizer: `new Adam(lr)`, `new Sgd(lr)`, `new Nesterovs(lr, momentum)`, etc. |
| `.l1(double)` | Global L1 regularization coefficient |
| `.l2(double)` | Global L2 regularization coefficient |
| `.dropout(double)` | Global dropout retain probability (applied after each layer) |
| `.activation(Activation)` | Default activation for layers that do not specify their own |
| `.gradientNormalization(GradientNormalization)` | Clip gradients by value or norm |
| `.gradientNormalizationThreshold(double)` | Threshold for gradient clipping |

---

## Initializing and Inspecting the Network

```java
MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
```

`init()` allocates parameter arrays, runs weight initialization, and wires up the internal computation graph. It must be called before any training or inference.

### Printing a Summary

```java
System.out.println(model.summary());
```

Output includes each layer's name, type, output shape, number of parameters, and connected layers. Extremely useful for catching misconfigured `nIn`/`nOut` values.

---

## Training

### Fitting with a DataSetIterator

The standard training loop passes a `DataSetIterator` directly to `fit()`:

```java
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;

DataSetIterator trainIter = /* RecordReaderDataSetIterator or similar */;

for (int epoch = 0; epoch < numEpochs; epoch++) {
    model.fit(trainIter);
    trainIter.reset();
}
```

### Fitting a Single DataSet

For small datasets held in memory:

```java
import org.nd4j.linalg.dataset.DataSet;

DataSet ds = /* DataSet with features and labels */;
model.fit(ds);
```

### Fitting with Raw Arrays

```java
import org.nd4j.linalg.api.ndarray.INDArray;

INDArray features = /* shape [minibatch, nIn] */;
INDArray labels   = /* shape [minibatch, nOut] */;
model.fit(features, labels);
```

### Attaching a ScoreIterationListener

```java
import org.deeplearning4j.optimize.listeners.ScoreIterationListener;

model.setListeners(new ScoreIterationListener(10)); // print score every 10 iterations
```

Other useful listeners: `PerformanceListener`, `EvaluativeListener`, `CheckpointListener`.

---

## Inference

### output()

Runs a full forward pass and returns the activations of the last layer:

```java
INDArray input = /* shape [minibatch, nIn] */;
INDArray predictions = model.output(input);          // default: test mode (dropout off)
INDArray trainPred   = model.output(input, true);    // train mode (dropout on)
```

### feedForward()

Returns activations of every layer as a `List<INDArray>`:

```java
List<INDArray> activations = model.feedForward(input, false);
// activations.get(0) = input, activations.get(1) = layer 0 output, ...
```

Useful for extracting intermediate representations for transfer learning or visualization.

### predict()

Returns predicted class index for each example (classification only):

```java
int[] classes = model.predict(input);
```

---

## Evaluation

### Classification

```java
import org.nd4j.evaluation.classification.Evaluation;

Evaluation eval = new Evaluation(numClasses);
DataSetIterator testIter = /* test iterator */;

while (testIter.hasNext()) {
    DataSet batch = testIter.next();
    INDArray predictions = model.output(batch.getFeatures());
    eval.eval(batch.getLabels(), predictions);
}

System.out.println(eval.stats());
// Prints accuracy, precision, recall, F1, confusion matrix
```

### Regression

```java
import org.nd4j.evaluation.regression.RegressionEvaluation;

RegressionEvaluation regEval = new RegressionEvaluation();
while (testIter.hasNext()) {
    DataSet batch = testIter.next();
    INDArray predictions = model.output(batch.getFeatures());
    regEval.eval(batch.getLabels(), predictions);
}
System.out.println(regEval.stats());
// Prints MSE, MAE, RMSE, R^2 per output column
```

### Using evaluateDataSet Convenience Method

```java
Evaluation eval = model.evaluate(testIter);
System.out.println(eval.accuracy());
```

---

## Parameter Access

### Viewing All Parameters

```java
// Flat vector of all parameters
INDArray allParams = model.params();

// Parameters as a map: "0_W", "0_b", "1_W", "1_b", ...
Map<String, INDArray> paramMap = model.paramTable();
for (Map.Entry<String, INDArray> entry : paramMap.entrySet()) {
    System.out.println(entry.getKey() + " shape: " + Arrays.toString(entry.getValue().shape()));
}
```

### Getting and Setting Layer Parameters

```java
// Get weight matrix of layer 0
org.deeplearning4j.nn.api.Layer layer0 = model.getLayer(0);
INDArray weights = layer0.getParam("W");
INDArray biases  = layer0.getParam("b");

// Set a parameter directly
layer0.setParam("W", myCustomWeights);
```

Standard parameter keys: `"W"` for weights, `"b"` for bias. Convolutional layers use `"W"` for the filter tensor.

---

## Gradient Access

Gradients are available after a backward pass (which happens automatically during training):

```java
import org.deeplearning4j.nn.gradient.Gradient;
import org.nd4j.linalg.primitives.Pair;

// Run one forward + backward pass manually
INDArray features = /* input */;
INDArray labels   = /* labels */;

model.setInput(features);
model.setLabels(labels);
model.computeGradientAndScore();

Gradient gradient = model.gradient();
Map<String, INDArray> gradMap = gradient.gradientForVariable();
INDArray weightGrad = gradMap.get("0_W"); // gradient for layer 0 weights
```

---

## Saving and Loading

### ModelSerializer (recommended)

```java
import org.deeplearning4j.util.ModelSerializer;
import java.io.File;

// Save (include updater state for continued training)
File modelFile = new File("myModel.zip");
ModelSerializer.writeModel(model, modelFile, true);

// Load
MultiLayerNetwork loaded = ModelSerializer.restoreMultiLayerNetwork(modelFile);
```

Pass `false` as the third argument to `writeModel` if you only need inference (smaller file, no updater state).

### JSON / YAML Configuration

```java
// Save configuration only
String json = model.getLayerWiseConfigurations().toJson();
String yaml = model.getLayerWiseConfigurations().toYaml();

// Restore configuration from JSON
MultiLayerConfiguration confFromJson = MultiLayerConfiguration.fromJson(json);
MultiLayerNetwork restored = new MultiLayerNetwork(confFromJson);
restored.init();
// Note: parameters are not included in JSON — restore params separately if needed
restored.setParams(model.params().dup());
```

---

## Common Patterns

### Simple MLP Classifier (M2.1)

```java
int numInputs  = 784;   // e.g., flattened 28x28 MNIST
int numHidden  = 256;
int numClasses = 10;

MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(123)
    .dataType(DataType.FLOAT)
    .updater(new Adam(0.001))
    .weightInit(WeightInit.XAVIER)
    .l2(1e-5)
    .list()
    .layer(new DenseLayer.Builder().nIn(numInputs).nOut(numHidden)
        .activation(Activation.RELU).build())
    .layer(new DenseLayer.Builder().nIn(numHidden).nOut(64)
        .activation(Activation.RELU).build())
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nIn(64).nOut(numClasses)
        .activation(Activation.SOFTMAX).build())
    .build();

MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
model.setListeners(new ScoreIterationListener(100));

model.fit(trainIter, numEpochs);

Evaluation eval = model.evaluate(testIter);
System.out.printf("Accuracy: %.4f%n", eval.accuracy());
```

### LeNet-Style CNN

```java
int channels = 1, height = 28, width = 28, numClasses = 10;

MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(0.001))
    .weightInit(WeightInit.XAVIER)
    .list()
    .layer(new ConvolutionLayer.Builder(5, 5)
        .nIn(channels).nOut(20)
        .stride(1, 1)
        .activation(Activation.IDENTITY)
        .build())
    .layer(new SubsamplingLayer.Builder(SubsamplingLayer.PoolingType.MAX)
        .kernelSize(2, 2).stride(2, 2).build())
    .layer(new ConvolutionLayer.Builder(5, 5)
        .nOut(50).stride(1, 1)
        .activation(Activation.IDENTITY)
        .build())
    .layer(new SubsamplingLayer.Builder(SubsamplingLayer.PoolingType.MAX)
        .kernelSize(2, 2).stride(2, 2).build())
    .layer(new DenseLayer.Builder().nOut(500)
        .activation(Activation.RELU).build())
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nOut(numClasses)
        .activation(Activation.SOFTMAX).build())
    .setInputType(InputType.convolutionalFlat(height, width, channels))
    .build();
```

`setInputType()` automatically inserts the necessary `FeedForwardToCnnPreProcessor` and calculates `nIn` for the first dense layer. You do not need to specify `nIn` on layers that follow convolutional blocks when using this feature.

### Regression with MSE Loss

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .list()
    .layer(new DenseLayer.Builder().nIn(10).nOut(64)
        .activation(Activation.RELU).build())
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.MSE)
        .nIn(64).nOut(1)
        .activation(Activation.IDENTITY).build())
    .build();
```

### LSTM for Sequence Classification

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    .dataType(DataType.FLOAT)
    .updater(new Adam(1e-3))
    .list()
    .layer(new LSTM.Builder().nIn(inputSize).nOut(128)
        .activation(Activation.TANH).build())
    .layer(new RnnOutputLayer.Builder(LossFunctions.LossFunction.MCXENT)
        .nIn(128).nOut(numClasses)
        .activation(Activation.SOFTMAX).build())
    .backpropType(BackpropType.TruncatedBPTT)
    .tBPTTLength(50)
    .build();
```

---

## Step-by-Step Inference (RNNs)

For RNNs where you want to feed one time step at a time and preserve state between calls:

```java
// Initialize state
model.rnnClearPreviousState();

// Feed one step at a time (input shape: [1, nIn])
INDArray stepInput = /* single time step */;
INDArray stepOutput = model.rnnTimeStep(stepInput);

// Continue feeding subsequent steps — state is maintained automatically
INDArray stepOutput2 = model.rnnTimeStep(nextStepInput);

// When starting a new independent sequence, clear state
model.rnnClearPreviousState();
```

---

## Key API Reference

| Method | Description |
|--------|-------------|
| `init()` | Allocate and initialize parameters |
| `fit(DataSetIterator)` | Train for one epoch over the iterator |
| `fit(DataSet)` | Train on a single batch |
| `output(INDArray)` | Forward pass, returns last layer activations |
| `feedForward(INDArray, boolean)` | Forward pass, returns all layer activations |
| `predict(INDArray)` | Returns predicted class indices |
| `evaluate(DataSetIterator)` | Returns `Evaluation` object |
| `evaluateRegression(DataSetIterator)` | Returns `RegressionEvaluation` object |
| `params()` | Returns flat INDArray of all parameters |
| `paramTable()` | Returns `Map<String, INDArray>` of named parameters |
| `getLayer(int)` | Returns a specific layer by index |
| `numLayers()` | Total number of layers |
| `summary()` | Human-readable architecture summary |
| `rnnTimeStep(INDArray)` | One-step RNN inference with state maintenance |
| `rnnClearPreviousState()` | Reset RNN hidden state |
| `setListeners(TrainingListener...)` | Attach training listeners |
| `score()` | Returns the most recent training loss |
| `computeGradientAndScore()` | Manually trigger forward + backward pass |
| `gradient()` | Returns `Gradient` object after backward pass |

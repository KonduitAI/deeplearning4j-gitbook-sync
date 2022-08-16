---
description: Adding hooks and listeners on DL4J models.
---

# Model Listeners

## What are listeners?

Listeners allow users to "hook" into certain events in Eclipse Deeplearning4j. This allows you to collect or print information useful for tasks like training. For example, a `ScoreIterationListener` allows you to print training scores from the output layer of a neural network.

## Usage

To add one or more listeners to a `MultiLayerNetwork` or `ComputationGraph`, use the `addListener` method:

```java
MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
//print the score with every 1 iteration
model.setListeners(new ScoreIterationListener(1));
```

## Available listeners

### EvaluativeListener

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/EvaluativeListener.java)

This TrainingListener implementation provides simple way for model evaluation during training. It can be launched every Xth Iteration/Epoch, depending on frequency and InvocationType constructor arguments

**EvaluativeListener**

```
public EvaluativeListener(@NonNull DataSetIterator iterator, int frequency)
```

This callback will be invoked after evaluation finished

**iterationDone**

```
public void iterationDone(Model model, int iteration, int epoch)
```

* param iterator Iterator to provide data for evaluation
* param frequency Frequency (in number of iterations/epochs according to the invocation type) to perform evaluation
* param type Type of value for ‘frequency’ - iteration end, epoch end, etc

### ScoreIterationListener

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/ScoreIterationListener.java)

Score iteration listener. Reports the score (value of the loss function )of the network during training every N iterations

**ScoreIterationListener**

```
public ScoreIterationListener(int printIterations)
```

* param printIterations frequency with which to print scores (i.e., every printIterations parameter updates)

### ComposableIterationListener

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/ComposableIterationListener.java)

A group of listeners

### CollectScoresIterationListener

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/CollectScoresIterationListener.java)

CollectScoresIterationListener simply stores the model scores internally (along with the iteration) every 1 or N iterations (this is configurable). These scores can then be obtained or exported.

**CollectScoresIterationListener**

```
public CollectScoresIterationListener()
```

Constructor for collecting scores with default saving frequency of 1

**iterationDone**

```
public void iterationDone(Model model, int iteration, int epoch)
```

Constructor for collecting scores with the specified frequency.

* param frequency Frequency with which to collect/save scores

**exportScores**

```
public void exportScores(OutputStream outputStream) throws IOException
```

Export the scores in tab-delimited (one per line) UTF-8 format.

**exportScores**

```
public void exportScores(OutputStream outputStream, String delimiter) throws IOException
```

Export the scores in delimited (one per line) UTF-8 format with the specified delimiter

* param outputStream Stream to write to
* param delimiter Delimiter to use

**exportScores**

```
public void exportScores(File file) throws IOException
```

Export the scores to the specified file in delimited (one per line) UTF-8 format, tab delimited

* param file File to write to

**exportScores**

```
public void exportScores(File file, String delimiter) throws IOException
```

Export the scores to the specified file in delimited (one per line) UTF-8 format, using the specified delimiter

* param file File to write to
* param delimiter Delimiter to use for writing scores

### CheckpointListener

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/CheckpointListener.java)

CheckpointListener: The goal of this listener is to periodically save a copy of the model during training..\
Model saving may be done:

1. Every N epochs&#x20;
2. Every N iterations&#x20;
3. Every T time units (every 15 minutes, for example) Or some combination of the 3.   **Example 1**: Saving a checkpoint every 2 epochs, keep all model files

```
.keepAll() //Don't delete any models
.saveEveryNEpochs(2)
.build()
}
```

**Example 2**: Saving a checkpoint every 1000 iterations, but keeping only the last 3 models (all older model files will be automatically deleted)

```
.keepLast(3)
.saveEveryNIterations(1000)
.build();
}
```

**Example 3**: Saving a checkpoint every 15 minutes, keeping the most recent 3 and otherwise every 4th checkpoint file:

```
.keepLastAndEvery(3, 4)
.saveEvery(15, TimeUnit.MINUTES)
.build();
}
```

Note that you can mix these: for example, to save every epoch and every 15 minutes (independent of last save time):\
To save every epoch, and every 15 minutes, since the last model save use:\
Note that is this last example, the sinceLast parameter is true. This means the 15-minute counter will be reset any time a model is saved.

**CheckpointListener**

```
public CheckpointListener build()
```

List all available checkpoints. A checkpoint is ‘available’ if the file can be loaded. Any checkpoint files that have been automatically deleted (given the configuration) will not be returned here.

* return List of checkpoint files that can be loaded

### SharedGradient

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/SharedGradient.java)

### SleepyTrainingListener

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/SleepyTrainingListener.java)

This TrainingListener implementation provides a way to “sleep” during specific Neural Network training phases.\
Suitable for debugging/testing purposes only.

PLEASE NOTE: All timers treat time values as milliseconds. PLEASE NOTE: Do not use it in production environment.

**onEpochStart**

```
public void onEpochStart(Model model)
```

In this mode parkNanos() call will be used, to make process really idle

### CollectScoresListener

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/CollectScoresListener.java)

A simple listener that collects scores to a list every N iterations. Can also optionally log the score.

### PerformanceListener

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/PerformanceListener.java)

Simple IterationListener that tracks time spend on training per iteration.

**PerformanceListener**

```
public PerformanceListener build()
```

This method defines, if iteration number should be reported together with other data

* param reportIteration
* return

### ParamAndGradientIterationListener

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/ParamAndGradientIterationListener.java)

An iteration listener that provides details on parameters and gradients at each iteration during traning. Attempts to provide much of the same information as the UI histogram iteration listener, but in a text-based format (for example, when learning on a system accessed via SSH etc). i.e., is intended to aid network tuning and debugging\
This iteration listener is set up to calculate mean, min, max, and mean absolute value of each type of parameter and gradient in the network at each iteration.

### TimeIterationListener

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/listeners/TimeIterationListener.java)

Time Iteration Listener. This listener displays into INFO logs the remaining time in minutes and the date of the end of the process. Remaining time is estimated from the amount of time for training so far, and the total number of iterations specified by the user

**TimeIterationListener**

```
public TimeIterationListener(int iterationCount)
```

Constructor

* param iterationCount The global number of iteration for training (all epochs)

---
title: "Listeners"
description: "Training listeners — ScoreIterationListener, PerformanceListener, EvaluativeListener, CheckpointListener, and custom listeners"
---

# Listeners

Listeners let you hook into events during `MultiLayerNetwork` or `ComputationGraph` training. Common uses include logging loss scores, measuring throughput, saving periodic checkpoints, and running evaluation on a test set. Listeners implement `org.deeplearning4j.optimize.api.TrainingListener` and are added with `setListeners(...)`.

## Attaching Listeners

```java
import org.deeplearning4j.optimize.listeners.ScoreIterationListener;
import org.deeplearning4j.optimize.listeners.PerformanceListener;

MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();

// One listener
model.setListeners(new ScoreIterationListener(10));

// Multiple listeners at once
model.setListeners(
    new ScoreIterationListener(10),
    new PerformanceListener(10, true)
);

// Add to a ComputationGraph the same way
ComputationGraph graph = new ComputationGraph(graphConf);
graph.init();
graph.setListeners(new ScoreIterationListener(1));
```

---

## ScoreIterationListener

Logs the current loss value to `slf4j` (INFO level) every N iterations. The "score" is the network's loss function value for the most recently processed minibatch.

```java
import org.deeplearning4j.optimize.listeners.ScoreIterationListener;

// Print score every 10 iterations
model.setListeners(new ScoreIterationListener(10));
```

Example output:

```
o.d.o.l.ScoreIterationListener - Score at iteration 0 is 2.302
o.d.o.l.ScoreIterationListener - Score at iteration 10 is 1.847
o.d.o.l.ScoreIterationListener - Score at iteration 20 is 1.523
```

**Constructor:** `ScoreIterationListener(int printIterations)` — frequency in parameter update steps.

---

## PerformanceListener

Reports training throughput (examples per second and minibatches per second) and optionally the current score. Useful for profiling training speed across hardware configurations.

```java
import org.deeplearning4j.optimize.listeners.PerformanceListener;

// Report every 10 iterations, include score in output
model.setListeners(
    new PerformanceListener.Builder()
        .reportIteration(true)
        .reportSample(true)       // samples/sec
        .reportBatch(true)        // batches/sec
        .reportScore(true)        // current loss
        .reportTime(true)         // elapsed wall time
        .setFrequency(10)
        .build()
);
```

Or use the simple constructor:

```java
// Every 10 iterations, report score=true
model.setListeners(new PerformanceListener(10, true));
```

Example output:

```
o.d.o.l.PerformanceListener - iteration 10; iteration time: 45 ms; samples/sec: 1422.22; score: 1.432
```

---

## EvaluativeListener

Runs a full evaluation pass on a held-out `DataSetIterator` every N iterations or every N epochs. The `InvocationType` controls whether frequency counts iterations or epochs.

```java
import org.deeplearning4j.optimize.listeners.EvaluativeListener;
import org.deeplearning4j.optimize.listeners.EvaluativeListener.InvocationType;

DataSetIterator testIter = /* test data */;

// Evaluate every 100 iterations
model.setListeners(
    new EvaluativeListener(testIter, 100, InvocationType.ITERATION_END)
);

// Evaluate once per epoch
model.setListeners(
    new EvaluativeListener(testIter, 1, InvocationType.EPOCH_END)
);
```

The listener logs the result of `model.evaluate(iterator)` (for classification) or `model.evaluateRegression(iterator)` depending on the network configuration.

You can supply a callback to receive the `IEvaluation` result:

```java
EvaluativeListener listener = new EvaluativeListener(testIter, 1, InvocationType.EPOCH_END);
listener.setCallback((evaluation, net, iter, epoch) -> {
    System.out.println("Epoch " + epoch + ": " + evaluation.stats());
});
model.setListeners(listener);
```

---

## CheckpointListener

Periodically saves the model to disk during training. The three trigger types (epochs, iterations, time) can be combined freely.

```java
import org.deeplearning4j.optimize.listeners.CheckpointListener;
import java.util.concurrent.TimeUnit;

File checkpointDir = new File("/tmp/checkpoints");

// Example 1: Save every 2 epochs, keep all files
model.setListeners(
    new CheckpointListener.Builder(checkpointDir)
        .keepAll()
        .saveEveryNEpochs(2)
        .build()
);

// Example 2: Save every 1000 iterations, keep only the 3 most recent
model.setListeners(
    new CheckpointListener.Builder(checkpointDir)
        .keepLast(3)
        .saveEveryNIterations(1000)
        .build()
);

// Example 3: Save every 15 minutes, keep the 3 most recent and every 4th
model.setListeners(
    new CheckpointListener.Builder(checkpointDir)
        .keepLastAndEvery(3, 4)
        .saveEvery(15, TimeUnit.MINUTES)
        .build()
);

// Example 4: Save every epoch AND every 15 minutes (since last save)
model.setListeners(
    new CheckpointListener.Builder(checkpointDir)
        .keepAll()
        .saveEveryNEpochs(1)
        .saveEvery(15, TimeUnit.MINUTES, true)  // sinceLast=true resets 15-min counter on save
        .build()
);
```

### CheckpointListener Builder Reference

| Method | Description |
|---|---|
| `saveEveryNEpochs(int n)` | Save after every n completed epochs. |
| `saveEveryNIterations(int n)` | Save after every n parameter update iterations. |
| `saveEvery(long amount, TimeUnit unit)` | Save when the specified wall-clock time has elapsed since training started. |
| `saveEvery(long amount, TimeUnit unit, boolean sinceLast)` | If `sinceLast=true`, reset timer after each save. |
| `keepAll()` | Never delete checkpoint files. |
| `keepLast(int n)` | Keep only the most recent n checkpoint files; older ones are deleted. |
| `keepLastAndEvery(int nLast, int every)` | Keep the most recent `nLast` files plus every `every`-th checkpoint. |

### Restoring from a Checkpoint

```java
import org.deeplearning4j.util.ModelSerializer;

// List available checkpoints (files not yet deleted)
List<Checkpoint> available = listener.availableCheckpoints();

// Restore the most recent checkpoint
Checkpoint latest = available.get(available.size() - 1);
MultiLayerNetwork restored = ModelSerializer.restoreMultiLayerNetwork(latest.getFile());
```

---

## CollectScoresIterationListener

Stores the loss score at each iteration (or every N iterations) in memory for later programmatic access or export to a file.

```java
import org.deeplearning4j.optimize.listeners.CollectScoresIterationListener;

// Collect every iteration
CollectScoresIterationListener scoreCollector = new CollectScoresIterationListener();

// Collect every 5 iterations
CollectScoresIterationListener scoreCollector5 = new CollectScoresIterationListener(5);

model.setListeners(scoreCollector);
model.fit(trainIter);

// Retrieve scores as a list of (iteration, score) pairs
List<Double> scores = scoreCollector.getListOfScores();

// Export to file (tab-delimited by default)
scoreCollector.exportScores(new File("/tmp/scores.txt"));

// Export with custom delimiter
scoreCollector.exportScores(new File("/tmp/scores.csv"), ",");

// Export to an OutputStream
try (OutputStream os = new FileOutputStream("/tmp/scores.tsv")) {
    scoreCollector.exportScores(os);
}
```

---

## TimeIterationListener

Estimates and logs the remaining training time and projected finish time. Requires the total number of iterations (across all epochs) to be specified upfront.

```java
import org.deeplearning4j.optimize.listeners.TimeIterationListener;

int totalIterations = numEpochs * (trainingSetSize / batchSize);
model.setListeners(new TimeIterationListener(totalIterations));
```

Output each iteration:

```
o.d.o.l.TimeIterationListener - Remaining: 42 minutes, expected finish: 2026-06-15T14:35:00
```

---

## ComposableIterationListener

Groups multiple `IterationListener` instances into a single listener. Useful when you need to wrap listeners in a context that accepts only one.

```java
import org.deeplearning4j.optimize.listeners.ComposableIterationListener;

ComposableIterationListener composite = new ComposableIterationListener(
    new ScoreIterationListener(10),
    new PerformanceListener(10, true),
    scoreCollector
);
model.setListeners(composite);
```

---

## ParamAndGradientIterationListener

Logs statistics (mean, min, max, mean absolute value) for all parameters and gradients at each iteration. Text-based alternative to the UI histogram when training on remote machines.

```java
import org.deeplearning4j.optimize.listeners.ParamAndGradientIterationListener;

model.setListeners(
    new ParamAndGradientIterationListener.Builder()
        .printMean(true)
        .printMinMax(true)
        .printMeanAbsValue(true)
        .outputToFile(true)
        .file(new File("/tmp/gradients.txt"))
        .delimiter("\t")
        .build()
);
```

---

## Custom Listeners

Implement `TrainingListener` (or extend `BaseTrainingListener` for default no-op implementations of unused methods):

```java
import org.deeplearning4j.nn.api.Model;
import org.deeplearning4j.optimize.api.BaseTrainingListener;

public class MyListener extends BaseTrainingListener {

    private final int frequency;

    public MyListener(int frequency) {
        this.frequency = frequency;
    }

    @Override
    public void iterationDone(Model model, int iteration, int epoch) {
        if (iteration % frequency == 0) {
            double score = model.score();
            System.out.printf("Epoch %d, Iteration %d: score = %.4f%n", epoch, iteration, score);
        }
    }

    @Override
    public void onEpochStart(Model model) {
        System.out.println("Starting epoch");
    }

    @Override
    public void onEpochEnd(Model model) {
        System.out.println("Epoch finished. Final score: " + model.score());
    }
}
```

The full `TrainingListener` interface also exposes:

| Method | When called |
|---|---|
| `onForwardPass(Model, List<INDArray>)` | After each forward pass (activations available) |
| `onBackwardPass(Model)` | After each backward pass (gradients available) |
| `onGradientCalculation(Model)` | After gradient computation, before parameter update |
| `iterationDone(Model, int, int)` | After each parameter update (iteration, epoch) |
| `onEpochStart(Model)` | Before the first iteration of each epoch |
| `onEpochEnd(Model)` | After the last iteration of each epoch |

---

## API Reference

| Class | Package |
|---|---|
| `TrainingListener` (interface) | `org.deeplearning4j.optimize.api` |
| `BaseTrainingListener` | `org.deeplearning4j.optimize.api` |
| `ScoreIterationListener` | `org.deeplearning4j.optimize.listeners` |
| `PerformanceListener` | `org.deeplearning4j.optimize.listeners` |
| `EvaluativeListener` | `org.deeplearning4j.optimize.listeners` |
| `CheckpointListener` | `org.deeplearning4j.optimize.listeners` |
| `CollectScoresIterationListener` | `org.deeplearning4j.optimize.listeners` |
| `TimeIterationListener` | `org.deeplearning4j.optimize.listeners` |
| `ComposableIterationListener` | `org.deeplearning4j.optimize.listeners` |
| `ParamAndGradientIterationListener` | `org.deeplearning4j.optimize.listeners` |

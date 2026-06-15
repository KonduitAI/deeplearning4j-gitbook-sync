---
title: "Early Stopping"
description: "Early stopping in Deeplearning4j — EarlyStoppingConfiguration, termination conditions, score calculators, and model savers"
---

# Early Stopping

## What Is Early Stopping?

When training neural networks, one of the most important hyperparameters is the number of training epochs. Too few epochs and the network underfits; too many and it overfits, memorizing noise rather than signal.

Early stopping removes the need to choose this value manually. It is also a form of regularization — analogous to L1/L2 weight decay or dropout — because it prevents the model from continuing to fit the training set once it has stopped improving on held-out data.

The idea is simple:

1. Split data into training and validation sets.
2. At the end of each epoch (or every N epochs), evaluate the network on the validation set.
3. If the network outperforms all previous checkpoints, save a copy of the model.
4. Stop training when a termination condition is satisfied.
5. Return the saved model with the best validation score.

```
         loss
          |
          | \
          |   \
          |    \___
          |        \____
          |              \___________/
          |                          ^--- best model saved here
          +---------------------------------> epoch
```

## Maven Dependency

Early stopping is included in the core DL4J dependency. No additional artifact is required.

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-core</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

## Quick Start

```java
MultiLayerConfiguration netConf = new NeuralNetConfiguration.Builder()
        // ... your layer configuration ...
        .build();

DataSetIterator trainData = ...;
DataSetIterator testData  = ...;

EarlyStoppingConfiguration<MultiLayerNetwork> esConf =
    new EarlyStoppingConfiguration.Builder<MultiLayerNetwork>()
        // Stop after at most 50 epochs
        .epochTerminationConditions(new MaxEpochsTerminationCondition(50))
        // Also stop if training takes more than 30 minutes
        .iterationTerminationConditions(new MaxTimeIterationTerminationCondition(30, TimeUnit.MINUTES))
        // Stop if loss on testData doesn't improve for 5 consecutive epochs
        .epochTerminationConditions(new ScoreImprovementEpochTerminationCondition(5))
        // Evaluate score once per epoch using testData loss
        .scoreCalculator(new DataSetLossCalculator(testData, true))
        .evaluateEveryNEpochs(1)
        // Save intermediate models to disk
        .modelSaver(new LocalFileModelSaver("/tmp/my-early-stopping"))
        .build();

EarlyStoppingTrainer trainer =
    new EarlyStoppingTrainer(esConf, netConf, trainData);

EarlyStoppingResult<MultiLayerNetwork> result = trainer.fit();

System.out.println("Termination reason:  " + result.getTerminationReason());
System.out.println("Termination details: " + result.getTerminationDetails());
System.out.println("Total epochs:        " + result.getTotalEpochs());
System.out.println("Best epoch:          " + result.getBestModelEpoch());
System.out.println("Best epoch score:    " + result.getBestModelScore());

MultiLayerNetwork bestModel = result.getBestModel();
```

For `ComputationGraph`, substitute `EarlyStoppingGraphTrainer` and use a `ComputationGraphConfiguration`.

---

## EarlyStoppingConfiguration

`EarlyStoppingConfiguration` is constructed via its inner `Builder`. The builder accepts four categories of settings.

### Epoch Termination Conditions

Evaluated once at the end of each epoch. Training stops if **any** epoch termination condition returns `true`.

Multiple epoch conditions can be added with repeated calls to `.epochTerminationConditions(...)`. They are evaluated with OR semantics — the first one that fires ends training.

### Iteration Termination Conditions

Evaluated once per minibatch (i.e., after every parameter update). Checked more frequently than epoch conditions, suitable for time limits.

### Score Calculator

Defines how the model's performance is measured after each evaluation cycle. The result of the score calculator determines which epoch produced the "best" model, and is also used by score-based termination conditions.

### Model Saver

Controls how intermediate and best models are persisted.

---

## Termination Conditions

### MaxEpochsTerminationCondition

Stops training after a fixed number of epochs.

```java
// Stop after 100 epochs at the latest
new MaxEpochsTerminationCondition(100)
```

This is typically used as a safety cap when combined with other conditions.

### MaxTimeIterationTerminationCondition

Stops training after a wall-clock time limit. Evaluated every iteration (minibatch), so it can interrupt mid-epoch.

```java
// Stop after 2 hours regardless of epoch count
new MaxTimeIterationTerminationCondition(2, TimeUnit.HOURS)
```

This is an *iteration* termination condition, so it is passed via `.iterationTerminationConditions(...)`.

### ScoreImprovementEpochTerminationCondition

Stops training when the best validation score has not improved for N consecutive epochs. This is the classic "patience" criterion.

```java
// Stop if no improvement in the best score for 10 epochs
new ScoreImprovementEpochTerminationCondition(10)
```

An optional minimum improvement threshold can be specified:

```java
// Stop if improvement is less than 0.001 for 10 consecutive epochs
new ScoreImprovementEpochTerminationCondition(10, 1e-3)
```

### BestScoreEpochTerminationCondition

Stops training once the score crosses a target threshold.

```java
// Stop as soon as validation loss drops below 0.05
new BestScoreEpochTerminationCondition(0.05)
```

By default this checks for scores *below* the threshold (i.e., minimization). Set `lesserBetter = false` for maximization metrics such as accuracy.

### MaxScoreIterationTerminationCondition

Stops training if the **current iteration's** score exceeds a maximum. Used to bail out early when training is clearly diverging (e.g., loss is exploding).

```java
// Abort immediately if minibatch loss exceeds 5.0
new MaxScoreIterationTerminationCondition(5.0)
```

This is an *iteration* termination condition.

### Implementing Custom Conditions

Implement `EpochTerminationCondition` or `IterationTerminationCondition` to define your own logic.

```java
public class MyEpochCondition implements EpochTerminationCondition {
    @Override
    public void initialize(EarlyStoppingConfiguration esConfig) { }

    @Override
    public boolean shouldStop(int epochNum, double score, boolean minimize) {
        // custom logic
        return epochNum > 20 && score > 2.0;
    }

    @Override
    public String toString() { return "MyEpochCondition"; }
}
```

---

## Score Calculators

A score calculator computes a single `double` after each evaluation. A lower score is treated as better by default (minimization). Override `minimizeScore()` to return `false` if using a higher-is-better metric.

### DataSetLossCalculator

Computes the average network loss (value of the configured loss function) over all examples in a `DataSetIterator`. Suitable for `MultiLayerNetwork`.

```java
// true = average loss per example; false = total loss
new DataSetLossCalculator(testDataIterator, true)
```

### DataSetLossCalculatorCG

Same as above, for `ComputationGraph`.

```java
new DataSetLossCalculatorCG(testDataIterator, true)
```

### ClassificationScoreCalculator

Uses a classification metric (accuracy, F1, etc.) as the score. Higher-is-better by default.

```java
// Use F1 score on test set
new ClassificationScoreCalculator(Evaluation.Metric.F1, testDataIterator)
```

Available `Evaluation.Metric` values: `ACCURACY`, `F1`, `PRECISION`, `RECALL`, `GMEASURE`, `MCC`.

### ROCScoreCalculator

Scores the model using area under the ROC curve (AUC) or area under the precision-recall curve (AUPRC).

```java
new ROCScoreCalculator(ROCScoreCalculator.ROCType.BINARY,
                       ROCScoreCalculator.Metric.AUC,
                       testDataIterator)
```

### RegressionScoreCalculator

Scores regression models using metrics such as MSE, MAE, RMSE, R² etc.

```java
new RegressionScoreCalculator(RegressionEvaluation.Metric.MSE, testDataIterator)
```

### AutoencoderScoreCalculator

Scores an autoencoder using reconstruction loss.

```java
new AutoencoderScoreCalculator(testDataIterator)
```

### VAEReconErrorScoreCalculator / VAEReconProbScoreCalculator

Scores variational autoencoders using reconstruction error or reconstruction (log) probability. The VAE layer must be the first layer in the network.

---

## Model Savers

### LocalFileModelSaver

Saves each evaluated model to a directory on disk. The best model is written to `bestModel.bin`; intermediate models use epoch number in the filename.

```java
new LocalFileModelSaver("/path/to/checkpoints/")
```

Models can be restored with `ModelSerializer.restoreMultiLayerNetwork(...)` or `ModelSerializer.restoreComputationGraph(...)`.

### InMemoryModelSaver

Keeps the best model in JVM heap memory. No I/O overhead, but the model is lost if the process exits.

```java
new InMemoryModelSaver<>()
```

Suitable for small models or experimentation. Not recommended for long runs where a process crash would be costly.

### Implementing a Custom Saver

Implement `EarlyStoppingModelSaver<T>` where `T` is `MultiLayerNetwork` or `ComputationGraph`.

```java
public class MyModelSaver implements EarlyStoppingModelSaver<MultiLayerNetwork> {
    @Override
    public void saveBestModel(MultiLayerNetwork net, double score) throws IOException {
        net.save(new File("best_model.zip"));
    }

    @Override
    public void saveLatestModel(MultiLayerNetwork net, double score) throws IOException {
        net.save(new File("latest_model.zip"));
    }

    @Override
    public MultiLayerNetwork getBestModel() throws IOException {
        return MultiLayerNetwork.load(new File("best_model.zip"), true);
    }

    @Override
    public MultiLayerNetwork getLatestModel() throws IOException {
        return MultiLayerNetwork.load(new File("latest_model.zip"), true);
    }
}
```

---

## EarlyStoppingTrainer

`EarlyStoppingTrainer` wraps a `MultiLayerConfiguration` (or a pre-built `MultiLayerNetwork`) and a `DataSetIterator`, and drives the training loop.

```java
// From configuration (network built internally)
EarlyStoppingTrainer trainer =
    new EarlyStoppingTrainer(esConf, netConf, trainData);

// From a pre-built network (resume training)
MultiLayerNetwork existingNet = ...;
EarlyStoppingTrainer trainer =
    new EarlyStoppingTrainer(esConf, existingNet, trainData);
```

For `ComputationGraph`:

```java
EarlyStoppingGraphTrainer trainer =
    new EarlyStoppingGraphTrainer(esConf, cgConf, trainData, null);
```

### EarlyStoppingResult

The `EarlyStoppingResult` returned by `trainer.fit()` provides:

| Method | Returns |
|---|---|
| `getTerminationReason()` | Why training stopped (`EpochTerminationCondition`, `IterationTerminationCondition`, `Error`) |
| `getTerminationDetails()` | Human-readable description of the termination condition |
| `getTotalEpochs()` | Number of epochs that were actually executed |
| `getBestModelEpoch()` | Epoch number at which the best model was recorded |
| `getBestModelScore()` | Score (from the score calculator) at the best epoch |
| `getScoreVsEpoch()` | `Map<Integer, Double>` of score at each evaluated epoch |
| `getBestModel()` | The best model (loaded from the model saver) |

---

## Parallel Training with EarlyStoppingParallelTrainer

For multi-GPU or multi-CPU training, `EarlyStoppingParallelTrainer` wraps the model in a `ParallelWrapper`:

```java
EarlyStoppingParallelTrainer trainer =
    new EarlyStoppingParallelTrainer(esConf, netConf, trainData,
                                     null,     // listeners
                                     4,        // number of workers
                                     8,        // prefetch buffer
                                     2);       // averaging frequency
```

Constraints to be aware of:

- The training UI (`StatsListener`) is not compatible with `EarlyStoppingParallelTrainer`.
- Complex custom `IterationListener` implementations may not behave correctly due to model copying between workers.
- For most use cases, single-device early stopping is simpler and sufficient.

---

## Common Patterns

### Using Both Time and Epoch Limits

```java
new EarlyStoppingConfiguration.Builder<MultiLayerNetwork>()
    .epochTerminationConditions(
        new MaxEpochsTerminationCondition(200),
        new ScoreImprovementEpochTerminationCondition(15))
    .iterationTerminationConditions(
        new MaxTimeIterationTerminationCondition(4, TimeUnit.HOURS))
    .scoreCalculator(new DataSetLossCalculator(valIterator, true))
    .evaluateEveryNEpochs(1)
    .modelSaver(new LocalFileModelSaver(checkpointDir))
    .build();
```

### Evaluating Every N Epochs

Evaluation is expensive for large datasets. Use `evaluateEveryNEpochs(n)` to reduce overhead:

```java
.evaluateEveryNEpochs(5)  // evaluate only every 5 epochs
```

Note that score-based termination conditions count their patience in *evaluation cycles*, not raw epochs, when this setting is active.

### Maximizing a Metric (e.g., Accuracy)

```java
.scoreCalculator(
    new ClassificationScoreCalculator(Evaluation.Metric.ACCURACY, valIterator))
```

`ClassificationScoreCalculator` sets `minimizeScore() = false`, so the trainer automatically looks for the highest score rather than the lowest.

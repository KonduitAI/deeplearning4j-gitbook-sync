---
title: "SameDiff Training"
description: "Training SameDiff models — TrainingConfig, fit(), listeners, loss curves, and evaluation"
---

# SameDiff Training

Once you have defined a SameDiff computation graph, training it is a matter of configuring an optimizer and a loss variable, then calling `fit()`. SameDiff handles the forward pass, backward pass, and parameter updates automatically.

## Overview of the Training Flow

1. **Define the graph** — declare placeholders, variables, ops, and a scalar loss `SDVariable`.
2. **Create a `TrainingConfig`** — specify the optimizer, which placeholder names correspond to features and labels, and any listeners.
3. **Attach the config** with `sd.setTrainingConfig(config)`.
4. **Call `sd.fit()`** — pass a data iterator and a number of epochs. SameDiff returns a `History` object with loss and metric values per epoch.

## TrainingConfig

`TrainingConfig` is the single configuration object for a SameDiff training run. It is built with a fluent builder API.

```java
import org.nd4j.autodiff.samediff.TrainingConfig;
import org.nd4j.linalg.learning.config.Adam;
import org.nd4j.linalg.api.buffer.DataType;

TrainingConfig config = TrainingConfig.builder()
    .updater(new Adam(1e-3))                      // optimizer with learning rate
    .dataSetFeatureMapping("input")               // feature array -> placeholder
    .dataSetLabelMapping("labels")                // label array   -> placeholder
    .lossVariables("loss")                        // which SDVariable holds the scalar loss
    .build();

sd.setTrainingConfig(config);
```

### Required settings

| Setting | Method | Notes |
|---|---|---|
| Optimizer | `.updater(IUpdater)` | Must always be set |
| Feature mapping | `.dataSetFeatureMapping(String...)` | Maps `DataSet`/`MultiDataSet` feature arrays to placeholder names (in order) |
| Label mapping | `.dataSetLabelMapping(String...)` | Maps label arrays to placeholder names |
| Loss variable | `.lossVariables(String...)` | Name of the scalar `SDVariable` used as the training loss |

### Optimizer (IUpdater)

SameDiff reuses the same `IUpdater` implementations as DL4J's `MultiLayerNetwork`:

```java
// Stochastic Gradient Descent with momentum
new Sgd(0.01)

// Adam
new Adam(1e-3)
new Adam(1e-3, 0.9, 0.999, 1e-8)   // lr, beta1, beta2, epsilon

// AdaGrad
new AdaGrad(0.01)

// RMSProp
new RmsProp(1e-3)

// Nesterov momentum SGD
new Nesterovs(0.01, 0.9)

// No-op (update weights manually)
new NoOp()
```

All optimizers support a learning rate schedule via `LearningRateSchedule`:

```java
import org.nd4j.linalg.schedule.StepSchedule;

ISchedule lrSchedule = new StepSchedule(ScheduleType.EPOCH, 1e-3, 0.5, 5);
// halves the LR every 5 epochs
TrainingConfig config = TrainingConfig.builder()
    .updater(new Adam(lrSchedule))
    .dataSetFeatureMapping("input")
    .dataSetLabelMapping("labels")
    .lossVariables("loss")
    .build();
```

### MultiDataSet mappings

When using `MultiDataSetIterator` (multiple feature and/or label arrays), list placeholder names in the same order as the arrays:

```java
TrainingConfig config = TrainingConfig.builder()
    .updater(new Adam(1e-3))
    .dataSetFeatureMapping("encoder_input", "decoder_input")  // two feature arrays
    .dataSetLabelMapping("target_sequence")                   // one label array
    .lossVariables("seq2seq_loss")
    .build();
```

### Data type conversion

If your data iterator produces arrays in a different type than your model's placeholders expect, use `.dataSetFeatureMappingDtype()` and `.dataSetLabelMappingDtype()` to request automatic casting:

```java
TrainingConfig config = TrainingConfig.builder()
    .updater(new Adam(1e-3))
    .dataSetFeatureMapping("input")
    .dataSetLabelMapping("labels")
    .dataSetFeatureMappingDtype(DataType.FLOAT)
    .dataSetLabelMappingDtype(DataType.FLOAT)
    .lossVariables("loss")
    .build();
```

## fit() — Running Training

### With a `DataSetIterator`

```java
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;

DataSetIterator trainIter = /* your iterator */ null;
int numEpochs = 20;

History history = sd.fit(trainIter, numEpochs);
```

Each epoch runs through all batches in `trainIter`, executes the forward pass, computes the loss, runs the backward pass, and updates all `VARIABLE`-type parameters.

### With a `MultiDataSetIterator`

```java
import org.nd4j.linalg.dataset.api.iterator.MultiDataSetIterator;

MultiDataSetIterator trainIter = /* your iterator */ null;
History history = sd.fit(trainIter, numEpochs);
```

The `MultiDataSetIterator` form is needed when your model has more than one input or output array per sample.

### With a validation set

Pass a second iterator to evaluate at the end of every epoch:

```java
DataSetIterator valIter = /* validation iterator */ null;

History history = sd.fit()
    .train(trainIter, numEpochs)
    .validate(valIter, 1)          // evaluate every 1 epoch
    .execute();
```

### Fitting a single batch manually

For custom training loops, drive the iteration yourself:

```java
while (trainIter.hasNext()) {
    DataSet batch = trainIter.next();
    sd.fit(batch);
}
```

## History and LossCurve

`fit()` returns a `History` object that records training progress.

```java
History history = sd.fit(trainIter, 10);

// Loss values per epoch
List<Double> trainLosses = history.trainingLoss().meanLoss();
System.out.println("Final train loss: " + trainLosses.get(trainLosses.size() - 1));

// Print a formatted loss curve
history.trainingLoss().print();
```

`History` also exposes raw per-iteration loss values if you need finer-grained monitoring:

```java
// Returns a List<Double> of loss values, one per training step
List<Double> stepLosses = history.trainingLoss().lossValues();
```

If you ran validation, validation losses are available separately:

```java
List<Double> valLosses = history.validationLoss().meanLoss();
```

## Listeners

Listeners let you observe and react to events during training. Attach them to the `TrainingConfig` or directly to the `SameDiff` instance.

### Available built-in listeners

#### ScoreIterationListener

Prints the loss every N iterations:

```java
import org.deeplearning4j.optimize.listeners.ScoreIterationListener;

TrainingConfig config = TrainingConfig.builder()
    .updater(new Adam(1e-3))
    .dataSetFeatureMapping("input")
    .dataSetLabelMapping("labels")
    .lossVariables("loss")
    .trainEvaluations("output", 0, new Accuracy())
    .addListener(new ScoreIterationListener(10))   // print every 10 iterations
    .build();
```

#### SameDiffListener interface

Implement `SameDiffListener` for fully custom callbacks:

```java
import org.nd4j.autodiff.listeners.At;
import org.nd4j.autodiff.listeners.Loss;
import org.nd4j.autodiff.listeners.SameDiffListener;

SameDiffListener myListener = new SameDiffListener() {
    @Override
    public boolean isActive(Operation op) {
        return true;  // active during all operations
    }

    @Override
    public boolean isActivate(SameDiff sd, At at) {
        return true;
    }

    @Override
    public void epochStart(SameDiff sd, At at) {
        System.out.println("Starting epoch " + at.epoch());
    }

    @Override
    public void epochEnd(SameDiff sd, At at, LossCurve lossCurve, long epochTimeMs) {
        System.out.printf("Epoch %d done in %dms, loss=%.4f%n",
            at.epoch(), epochTimeMs, lossCurve.meanLoss(at.epoch()));
    }

    @Override
    public void iterationDone(SameDiff sd, At at, MultiDataSet ds, Loss loss) {
        // called after each iteration
    }
};
```

Attach the listener:

```java
sd.addListeners(myListener);
// or via TrainingConfig:
TrainingConfig config = TrainingConfig.builder()
    // ...
    .addListener(myListener)
    .build();
```

#### CheckpointListener

Save a checkpoint to disk every N epochs or every N minutes:

```java
import org.nd4j.autodiff.listeners.checkpoint.CheckpointListener;

CheckpointListener ckpt = new CheckpointListener.Builder("/path/to/checkpoints")
    .keepLast(3)                  // keep only the 3 most recent checkpoints
    .saveEveryNEpochs(5)          // save every 5 epochs
    .build();

sd.addListeners(ckpt);
```

## Adding Evaluation During Training

SameDiff can compute evaluation metrics (accuracy, F1, etc.) automatically at the end of each epoch without writing extra evaluation code.

Specify which output variable and which evaluation to use via `TrainingConfig`:

```java
import org.nd4j.evaluation.classification.Evaluation;
import org.nd4j.evaluation.classification.Evaluation.Metric;

TrainingConfig config = TrainingConfig.builder()
    .updater(new Adam(1e-3))
    .dataSetFeatureMapping("input")
    .dataSetLabelMapping("labels")
    .lossVariables("loss")
    // Evaluate "output" variable against the 0th label array using Accuracy
    .trainEvaluations("output", 0, new Evaluation())
    .build();
```

For validation evaluations, use `.validationEvaluations(...)` instead.

After training, retrieve evaluation results from the `History` object:

```java
History history = sd.fit(trainIter, 10);
Evaluation eval = (Evaluation) history.finalTrainEvaluations().evaluation("output");
System.out.println(eval.stats());
```

Supported evaluation classes include `Evaluation`, `RegressionEvaluation`, `ROC`, `ROCMultiClass`, and others from the `org.nd4j.evaluation` package.

## End-to-End Training Example

The following is a complete example of training a two-layer classifier on MNIST:

```java
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.autodiff.samediff.SDVariable;
import org.nd4j.autodiff.samediff.TrainingConfig;
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.learning.config.Adam;
import org.nd4j.evaluation.classification.Evaluation;
import org.deeplearning4j.datasets.iterator.impl.MnistDataSetIterator;
import org.nd4j.weightinit.impl.XavierInitScheme;

// 1. Build graph
SameDiff sd = SameDiff.create();

SDVariable input  = sd.placeHolder("input",  DataType.FLOAT, -1, 784);
SDVariable labels = sd.placeHolder("labels", DataType.FLOAT, -1, 10);

SDVariable w1 = sd.var("w1", new XavierInitScheme('c', 784, 256), DataType.FLOAT, 784, 256);
SDVariable b1 = sd.var("b1", DataType.FLOAT, 256);
SDVariable w2 = sd.var("w2", new XavierInitScheme('c', 256, 10), DataType.FLOAT, 256, 10);
SDVariable b2 = sd.var("b2", DataType.FLOAT, 10);

SDVariable hidden = sd.nn.relu("hidden", input.mmul(w1).add(b1), 0);
SDVariable logits = hidden.mmul(w2).add(b2);
SDVariable output = sd.nn.softmax("output", logits);
SDVariable loss   = sd.loss.softmaxCrossEntropy("loss", labels, logits, null);

// 2. Configure training
TrainingConfig config = TrainingConfig.builder()
    .updater(new Adam(1e-3))
    .dataSetFeatureMapping("input")
    .dataSetLabelMapping("labels")
    .lossVariables("loss")
    .trainEvaluations("output", 0, new Evaluation())
    .build();

sd.setTrainingConfig(config);

// 3. Load data
MnistDataSetIterator trainIter = new MnistDataSetIterator(64, true,  12345);
MnistDataSetIterator testIter  = new MnistDataSetIterator(64, false, 12345);

// 4. Train
History history = sd.fit()
    .train(trainIter, 10)
    .validate(testIter, 1)
    .execute();

// 5. Report results
System.out.println("Train losses: " + history.trainingLoss().meanLoss());

Evaluation eval = (Evaluation) history.finalTrainEvaluations().evaluation("output");
System.out.println(eval.stats());
```

## Controlling Which Parameters Are Trained

By default, all `VARIABLE`-type `SDVariable` instances in the graph are trained. To freeze specific parameters, convert them to constants before calling `fit()`:

```java
// Freeze encoder weights — they will not be updated
sd.convertToConstant(sd.getVariable("encoder_w1"));
sd.convertToConstant(sd.getVariable("encoder_w2"));

// Only decoder weights will be trained
sd.fit(trainIter, 10);
```

To unfreeze later:

```java
sd.convertToVariable(sd.getVariable("encoder_w1"));
```

## Gradient Clipping

Apply gradient clipping globally via the `TrainingConfig`:

```java
TrainingConfig config = TrainingConfig.builder()
    .updater(new Adam(1e-3))
    .dataSetFeatureMapping("input")
    .dataSetLabelMapping("labels")
    .lossVariables("loss")
    .gradientClipping(GradientClipping.ClipL2Norm, 1.0)   // clip gradient L2 norm to 1.0
    .build();
```

Available clipping strategies: `ClipL2Norm`, `ClipElementWiseAbsoluteValue`.

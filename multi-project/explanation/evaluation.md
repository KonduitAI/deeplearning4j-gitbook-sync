---
title: "Evaluation"
description: "Evaluating model performance — classification metrics, ROC curves, regression metrics, and evaluation during training"
---

# Evaluation

Evaluating model performance is essential for understanding whether your network is learning and for comparing different configurations. DL4J provides a comprehensive set of evaluation classes.

> **Migration note (beta4 → M2.1):** All evaluation classes moved from `org.deeplearning4j.eval` to `org.nd4j.evaluation`. If upgrading from beta4, update your imports.

## Classification: `Evaluation`

The primary evaluation class for multi-class classification tasks. Located at `org.nd4j.evaluation.classification.Evaluation`.

### Running Evaluation

```java
import org.nd4j.evaluation.classification.Evaluation;

// Option 1: Evaluate directly from a model
Evaluation eval = model.evaluate(testIter);
System.out.println(eval.stats());

// Option 2: Evaluate manually
Evaluation eval = new Evaluation(numClasses);
while (testIter.hasNext()) {
    DataSet batch = testIter.next();
    INDArray predictions = model.output(batch.getFeatures());
    eval.eval(batch.getLabels(), predictions);
}
testIter.reset();
System.out.println(eval.stats());
```

### Available Metrics

```java
double accuracy = eval.accuracy();                // overall accuracy
double precision = eval.precision();              // macro-averaged precision
double recall = eval.recall();                    // macro-averaged recall
double f1 = eval.f1();                            // macro-averaged F1

// Per-class metrics
double precisionClass0 = eval.precision(0);       // precision for class 0
double recallClass1 = eval.recall(1);             // recall for class 1
double f1Class2 = eval.f1(2);                     // F1 for class 2

// Confusion matrix
System.out.println(eval.confusionMatrix());

// Full stats table
System.out.println(eval.stats());
```

The `eval.stats()` output looks like:

```
========================Evaluation Metrics========================
 # of classes:    10
 Accuracy:        0.9823
 Precision:       0.9822  (macro)
 Recall:          0.9821  (macro)
 F1 Score:        0.9821  (macro)

=========================Confusion Matrix=========================
     0    1    2    3    4    5    6    7    8    9
----------------------------------------------------
   974    0    1    0    0    1    2    1    1    0 | 0 = 0
     0 1130    2    0    0    1    1    1    0    0 | 1 = 1
     ...
```

### Averaging Methods

By default, metrics are macro-averaged (average per-class metrics equally). You can also get:

```java
eval.precision(EvaluationAveraging.Macro);     // default — equal weight per class
eval.precision(EvaluationAveraging.Micro);     // weight by support (number of examples)
```

## Binary Classification: `EvaluationBinary`

For multi-label binary classification where each output is an independent binary decision (sigmoid outputs):

```java
import org.nd4j.evaluation.classification.EvaluationBinary;

EvaluationBinary evalBinary = new EvaluationBinary();
// ... eval loop same as above
evalBinary.eval(labels, predictions);

// Per-output metrics
double precisionOutput0 = evalBinary.precision(0);
double recallOutput0 = evalBinary.recall(0);
double f1Output0 = evalBinary.f1(0);
double accuracy0 = evalBinary.accuracy(0);
```

## ROC Curves

ROC (Receiver Operating Characteristic) curves measure the trade-off between true positive rate and false positive rate at various classification thresholds.

### Binary ROC

```java
import org.nd4j.evaluation.classification.ROC;

ROC roc = new ROC(100);    // 100 threshold steps
while (testIter.hasNext()) {
    DataSet ds = testIter.next();
    INDArray predictions = model.output(ds.getFeatures());
    roc.eval(ds.getLabels(), predictions);
}
testIter.reset();

double auc = roc.calculateAUC();                  // area under curve
double auprc = roc.calculateAUCPR();               // area under precision-recall curve
System.out.println("AUC: " + auc);
System.out.println("AUPRC: " + auprc);

// Get ROC curve data points
RocCurve rocCurve = roc.getRocCurve();
PrecisionRecallCurve prCurve = roc.getPrecisionRecallCurve();
```

### Multi-Class ROC

Computes one ROC curve per class (one-vs-all):

```java
import org.nd4j.evaluation.classification.ROCMultiClass;

ROCMultiClass rocMC = new ROCMultiClass(100);
// ... eval loop
rocMC.eval(labels, predictions);

double aucClass0 = rocMC.calculateAUC(0);    // AUC for class 0
double avgAuc = rocMC.calculateAverageAUC(); // average across all classes
```

### Multi-Label Binary ROC

For multi-label tasks (multiple independent binary outputs):

```java
import org.nd4j.evaluation.classification.ROCBinary;

ROCBinary rocBin = new ROCBinary(100);
// ... eval loop
rocBin.eval(labels, predictions);

double aucOutput0 = rocBin.calculateAUC(0);
```

## Regression: `RegressionEvaluation`

For regression tasks (continuous output values):

```java
import org.nd4j.evaluation.regression.RegressionEvaluation;

RegressionEvaluation regEval = new RegressionEvaluation(numOutputs);
while (testIter.hasNext()) {
    DataSet ds = testIter.next();
    INDArray predictions = model.output(ds.getFeatures());
    regEval.eval(ds.getLabels(), predictions);
}
testIter.reset();

// Per-output metrics
double mse = regEval.meanSquaredError(0);          // column 0
double mae = regEval.meanAbsoluteError(0);
double rmse = regEval.rootMeanSquaredError(0);
double r2 = regEval.rSquared(0);                   // R² (coefficient of determination)
double corrCoef = regEval.correlationR2(0);        // Pearson correlation

System.out.println(regEval.stats());
```

## Calibration: `EvaluationCalibration`

Measures how well predicted probabilities match actual frequencies. Useful for assessing whether a model's softmax outputs are reliable as confidence scores.

```java
import org.nd4j.evaluation.classification.EvaluationCalibration;

EvaluationCalibration calibration = new EvaluationCalibration(20, 20); // bins
// ... eval loop
calibration.eval(labels, predictions);

// Get reliability diagram data
ReliabilityDiagram diagram = calibration.getReliabilityDiagram(0); // class 0
```

## Evaluation During Training

### Using EvaluativeListener

Run evaluation automatically at the end of each epoch:

```java
import org.deeplearning4j.optimize.listeners.EvaluativeListener;

model.setListeners(
    new ScoreIterationListener(100),
    new EvaluativeListener(testIter, 1, InvocationType.EPOCH_END)
);
```

The `EvaluativeListener` prints evaluation metrics after every N epochs (or iterations).

### Manual Evaluation in the Training Loop

For more control, evaluate manually between epochs:

```java
for (int epoch = 0; epoch < numEpochs; epoch++) {
    model.fit(trainIter);
    trainIter.reset();

    // Classification metrics
    Evaluation eval = model.evaluate(testIter);
    testIter.reset();

    System.out.printf("Epoch %d: acc=%.4f, f1=%.4f, precision=%.4f, recall=%.4f%n",
        epoch, eval.accuracy(), eval.f1(), eval.precision(), eval.recall());

    // Optionally compute ROC
    ROCMultiClass roc = model.evaluateROCMultiClass(testIter, 100);
    testIter.reset();
    System.out.printf("  AUC=%.4f%n", roc.calculateAverageAUC());
}
```

## Evaluating ComputationGraph

`ComputationGraph` has the same evaluation methods:

```java
Evaluation eval = graph.evaluate(testIter);
ROCMultiClass roc = graph.evaluateROCMultiClass(testIter, 100);
RegressionEvaluation regEval = graph.evaluateRegression(testIter);
```

For multi-output graphs, specify which output to evaluate:

```java
Evaluation eval = graph.evaluate(testIter, Collections.singletonList("outputLayerName"));
```

## Evaluating Recurrent Networks

For sequence-to-sequence tasks, evaluation works the same way but considers all time steps:

```java
Evaluation eval = model.evaluate(sequenceTestIter);
```

If your sequences have different lengths and you use masking, the evaluation automatically accounts for mask arrays — time steps where the mask is 0 are excluded from metrics.

## Custom Evaluation

All evaluation classes implement the `IEvaluation` interface. You can run multiple evaluations in a single pass:

```java
Evaluation classEval = new Evaluation(numClasses);
ROCMultiClass rocEval = new ROCMultiClass(100);

while (testIter.hasNext()) {
    DataSet ds = testIter.next();
    INDArray output = model.output(ds.getFeatures());
    classEval.eval(ds.getLabels(), output);
    rocEval.eval(ds.getLabels(), output);
}
testIter.reset();

System.out.println(classEval.stats());
System.out.println("Average AUC: " + rocEval.calculateAverageAUC());
```

## Quick Reference

| Task | Evaluation Class | Key Metric |
|------|-----------------|------------|
| Multi-class classification | `Evaluation` | `accuracy()`, `f1()` |
| Multi-label binary | `EvaluationBinary` | per-output `accuracy()`, `f1()` |
| Binary ROC | `ROC` | `calculateAUC()` |
| Multi-class ROC | `ROCMultiClass` | `calculateAverageAUC()` |
| Regression | `RegressionEvaluation` | `meanSquaredError()`, `rSquared()` |
| Probability calibration | `EvaluationCalibration` | reliability diagram |

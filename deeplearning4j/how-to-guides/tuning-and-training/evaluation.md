---
title: "Evaluation"
description: "Detailed evaluation guide — classification metrics, regression metrics, ROC curves, calibration, and multi-output evaluation"
---

# Evaluation

Evaluating a trained model provides quantitative measures of how well it is performing. Eclipse Deeplearning4j provides a comprehensive set of evaluation classes covering classification, binary classification, regression, ROC analysis, and calibration.

> **Package migration (beta4 → M2.1):** All evaluation classes have moved from `org.deeplearning4j.eval` to `org.nd4j.evaluation`. Update your imports accordingly. The classes in the old package are kept as deprecated wrappers.

---

## Classification: `Evaluation`

`org.nd4j.evaluation.classification.Evaluation` is the primary class for evaluating multi-class classifiers, including time-series classifiers. It accumulates predictions and labels across multiple minibatches before computing metrics.

### Running Evaluation

**Shortcut — using `model.evaluate()`:**

```java
import org.nd4j.evaluation.classification.Evaluation;

DataSetIterator testIter = /* your test data */;

// Most convenient: model handles iteration internally
Evaluation eval = model.evaluate(testIter);
System.out.println(eval.stats());
```

**Manual evaluation over minibatches:**

```java
Evaluation eval = new Evaluation(numClasses);
while (testIter.hasNext()) {
    DataSet batch = testIter.next();
    INDArray predictions = model.output(batch.getFeatures(), false);
    eval.eval(batch.getLabels(), predictions);
}
testIter.reset();
System.out.println(eval.stats());
```

### Available Metrics

```java
double accuracy  = eval.accuracy();
double precision = eval.precision();   // macro-averaged by default
double recall    = eval.recall();      // macro-averaged by default
double f1        = eval.f1();          // macro-averaged by default

// Per-class metrics (zero-indexed)
double precisionClass0 = eval.precision(0);
double recallClass1    = eval.recall(1);
double f1Class2        = eval.f1(2);

// Matthews Correlation Coefficient
double mcc = eval.matthewsCorrelation(EvaluationAveraging.Macro);

// False positive / false negative rates
double fpr = eval.falsePositiveRate(classIndex);
double fnr = eval.falseNegativeRate(classIndex);
```

### Confusion Matrix

```java
// Text table
System.out.println(eval.confusionToString());

// Structured access
ConfusionMatrix<Integer> cm = eval.getConfusionMatrix();
int truePositive = cm.getCount(actual, predicted);

// Export
String html = cm.toHTML();
String csv  = cm.toCSV();
```

Example `eval.stats()` output:

```
========================Evaluation Metrics========================
 # of classes:    3
 Accuracy:        0.9811
 Precision:       0.9815
 Recall:          0.9722
 F1 Score:        0.9760
Precision, recall & F1: macro-averaged (equally weighted avg. of 3 classes)
=================================================================
```

### Averaging Modes

Metrics that aggregate across classes support two averaging modes:

| Mode | Description |
|---|---|
| `Macro` (default) | Unweighted mean across all classes — treats each class equally. |
| `Micro` | Compute metric globally by counting total TPs, FPs, FNs across all classes. Appropriate for imbalanced datasets. |

```java
import org.nd4j.evaluation.classification.EvaluationAveraging;

double macroPrecision = eval.precision(EvaluationAveraging.Macro);
double microPrecision = eval.precision(EvaluationAveraging.Micro);
```

---

## Binary Classification: `EvaluationBinary`

`org.nd4j.evaluation.classification.EvaluationBinary` is for networks with multiple binary outputs — typically with Sigmoid activation and binary cross-entropy loss. It computes the full set of classification metrics independently for each output.

```java
import org.nd4j.evaluation.classification.EvaluationBinary;

// size = number of binary outputs
EvaluationBinary evalBin = new EvaluationBinary(numOutputs);

while (testIter.hasNext()) {
    DataSet batch = testIter.next();
    INDArray preds = model.output(batch.getFeatures());
    evalBin.eval(batch.getLabels(), preds);
}

System.out.println(evalBin.stats());

// Per-output metrics
double acc     = evalBin.accuracy(outputIndex);
double f1      = evalBin.f1(outputIndex);
double auprc   = evalBin.averagePrecisionScore(outputIndex);
```

Or using `model.evaluate()`:

```java
EvaluationBinary evalBin = model.evaluateBinary(testIter);
```

---

## Regression: `RegressionEvaluation`

`org.nd4j.evaluation.regression.RegressionEvaluation` computes standard regression metrics independently for each output column.

```java
import org.nd4j.evaluation.regression.RegressionEvaluation;

RegressionEvaluation evalReg = new RegressionEvaluation(numOutputs);

while (testIter.hasNext()) {
    DataSet batch = testIter.next();
    INDArray preds = model.output(batch.getFeatures());
    evalReg.eval(batch.getLabels(), preds);
}

System.out.println(evalReg.stats());
```

Or via shortcut:

```java
RegressionEvaluation evalReg = model.evaluateRegression(testIter);
```

The stats output reports per-column:

```
Column    MSE            MAE            RMSE           RSE            R^2
col_0     7.98925e+00    2.00648e+00    2.82653e+00    5.01481e-01    7.25783e-01
```

| Column | Metric |
|---|---|
| MSE | Mean Squared Error |
| MAE | Mean Absolute Error |
| RMSE | Root Mean Squared Error |
| RSE | Relative Squared Error |
| R^2 | Coefficient of Determination |

Access individual metrics programmatically:

```java
import org.nd4j.evaluation.regression.RegressionEvaluation.Metric;

double mse  = evalReg.scoreForMetric(Metric.MSE,  columnIndex);
double mae  = evalReg.scoreForMetric(Metric.MAE,  columnIndex);
double rmse = evalReg.scoreForMetric(Metric.RMSE, columnIndex);
double r2   = evalReg.scoreForMetric(Metric.R2,   columnIndex);
```

---

## ROC Curves

Three ROC classes cover different classification scenarios. All support two computation modes:

- **Exact** (`new ROC()` or `new ROC(0)`) — exact AUROC/AUPRC calculation. Can require significant memory with very large datasets.
- **Thresholded** (`new ROC(numBins)`) — approximate calculation using a fixed number of threshold bins. Constant memory. Recommended for large datasets.

### ROC — Single Binary Label

For networks with a single binary output (single Sigmoid, or 2-class Softmax):

```java
import org.nd4j.evaluation.classification.ROC;

ROC roc = new ROC(100);  // 100 threshold bins (thresholded mode)
// or: ROC roc = new ROC();  // exact mode

while (testIter.hasNext()) {
    DataSet batch = testIter.next();
    INDArray preds = model.output(batch.getFeatures());
    roc.eval(batch.getLabels(), preds);
}

double auroc = roc.calculateAUC();
double auprc = roc.calculateAUPRC();

// Get curve data points
RocCurve rocCurve = roc.getRocCurve();
PrecisionRecallCurve prCurve = roc.getPrecisionRecallCurve();

System.out.println("AUROC: " + auroc);
System.out.println("AUPRC: " + auprc);
```

Or via shortcut:

```java
ROC roc = model.evaluateROC(testIter, 100);
```

### ROCBinary — Multiple Binary Labels

For networks with multiple binary outputs (multiple Sigmoid neurons):

```java
import org.nd4j.evaluation.classification.ROCBinary;

ROCBinary rocBin = new ROCBinary(100);

while (testIter.hasNext()) {
    DataSet batch = testIter.next();
    rocBin.eval(batch.getLabels(), model.output(batch.getFeatures()));
}

// Per-output AUROC
for (int i = 0; i < numOutputs; i++) {
    System.out.printf("Output %d AUROC: %.4f%n", i, rocBin.calculateAUC(i));
}

// Average AUROC across all outputs
double avgAuroc = rocBin.calculateAverageAUC();
```

### ROCMultiClass — Multi-class One-vs-All

For Softmax classifiers, computes ROC for each class using a one-versus-all strategy:

```java
import org.nd4j.evaluation.classification.ROCMultiClass;

ROCMultiClass rocMulti = new ROCMultiClass(100);

while (testIter.hasNext()) {
    DataSet batch = testIter.next();
    rocMulti.eval(batch.getLabels(), model.output(batch.getFeatures()));
}

for (int c = 0; c < numClasses; c++) {
    System.out.printf("Class %d AUROC: %.4f%n", c, rocMulti.calculateAUC(c));
}
```

Or via shortcut:

```java
ROCMultiClass rocMulti = model.evaluateROCMultiClass(testIter, 100);
```

### Exporting ROC Charts to HTML

```java
import org.deeplearning4j.evaluation.EvaluationTools;

// Single ROC: generates HTML with both ROC and P-R curves
EvaluationTools.exportRocChartsToHtmlFile(roc, new File("/tmp/roc.html"));
```

---

## Calibration: `EvaluationCalibration`

`org.nd4j.evaluation.classification.EvaluationCalibration` analyses how well predicted probabilities align with actual outcome frequencies (calibration). A well-calibrated model that predicts 70% probability for a class should be correct roughly 70% of the time.

```java
import org.nd4j.evaluation.classification.EvaluationCalibration;

EvaluationCalibration cal = new EvaluationCalibration(numBins, numBins);

while (testIter.hasNext()) {
    DataSet batch = testIter.next();
    cal.eval(batch.getLabels(), model.output(batch.getFeatures()));
}

System.out.println(cal.stats());
```

The calibration evaluator provides:

- **Reliability diagram** (calibration curve) — predicted probability vs. actual frequency.
- **Residual histogram** — distribution of prediction errors.
- **Probability histograms** — overall and per-class.

Export all plots to HTML:

```java
import org.deeplearning4j.evaluation.EvaluationTools;

EvaluationTools.exportEvaluationCalibrationToHtmlFile(cal, new File("/tmp/calibration.html"));
```

---

## Performing Multiple Evaluations in One Pass

Running several evaluation types on the same test data requires only a single pass through the dataset when using `model.doEvaluation(...)`:

```java
DataSetIterator testIter = /* test data */;

Evaluation eval        = new Evaluation(numClasses);
ROCMultiClass rocMulti = new ROCMultiClass(100);
EvaluationCalibration cal = new EvaluationCalibration(20, 20);

// One pass, multiple evaluation objects updated simultaneously
model.doEvaluation(testIter, eval, rocMulti, cal);

System.out.println(eval.stats());
System.out.println("Avg AUROC: " + rocMulti.calculateAverageAUC());
```

This is significantly more efficient than iterating the dataset three separate times.

---

## Time Series Evaluation

Evaluation of RNNs proceeds in the same way as feedforward networks. DL4J evaluates all (non-masked) time steps independently. A sequence of length 10 contributes 10 prediction–label pairs per example.

Mask arrays (marking padding time steps) are handled automatically when using the model shortcut methods:

```java
// Masks are automatically respected
Evaluation eval = model.evaluate(testIter);
```

When evaluating manually, pass the mask array to `eval`:

```java
eval.evalTimeSeries(labels, predictions, labelsMask);
```

---

## Multi-task Evaluation

For `ComputationGraph` networks with multiple outputs, use `MultiTaskGraphEvaluation` or evaluate each output head separately using `model.doEvaluation()` with explicit output indices:

```java
// Evaluate output 0 as classification, output 1 as regression
Evaluation classEval = new Evaluation(numClasses);
RegressionEvaluation regEval = new RegressionEvaluation(1);

while (testIter.hasNext()) {
    MultiDataSet batch = testIter.next();
    INDArray[] outputs = graph.output(batch.getFeatures());

    classEval.eval(batch.getLabels(0), outputs[0]);
    regEval.eval(batch.getLabels(1), outputs[1]);
}
```

---

## Distributed (Spark) Evaluation

For Spark-distributed training, use evaluation methods on `SparkDl4jMultiLayer` or `SparkComputationGraph`:

```java
// Single evaluation type
Evaluation eval = sparkModel.evaluate(testRdd);

// Multiple types in one pass
sparkModel.doEvaluation(testRdd, batchSizePerWorker, eval, roc);
```

---

## Serialization

All evaluation objects implement `IEvaluation` and can be serialised to JSON or YAML for storage and later combination (e.g., collecting partial results from distributed workers):

```java
String json = eval.toJson();
Evaluation restored = Evaluation.fromJson(json);

// Merge results from two partial evaluations
Evaluation eval1 = /* from partition 1 */;
Evaluation eval2 = /* from partition 2 */;
eval1.merge(eval2);
System.out.println(eval1.stats());
```

---

## API Reference

| Class | Package |
|---|---|
| `Evaluation` | `org.nd4j.evaluation.classification` |
| `EvaluationBinary` | `org.nd4j.evaluation.classification` |
| `ROC` | `org.nd4j.evaluation.classification` |
| `ROCBinary` | `org.nd4j.evaluation.classification` |
| `ROCMultiClass` | `org.nd4j.evaluation.classification` |
| `EvaluationCalibration` | `org.nd4j.evaluation.classification` |
| `RegressionEvaluation` | `org.nd4j.evaluation.regression` |
| `ConfusionMatrix` | `org.nd4j.evaluation.classification` |
| `EvaluationAveraging` | `org.nd4j.evaluation.classification` |
| `IEvaluation` (interface) | `org.nd4j.evaluation` |
| `EvaluationTools` | `org.deeplearning4j.evaluation` |

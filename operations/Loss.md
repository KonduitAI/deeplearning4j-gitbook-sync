---
title: Loss
short_title: Loss
description: 
category: Operations
weight: 80
---
# Operation classes
## absoluteDifference
```JAVA
INDArray absoluteDifference(INDArray label, INDArray predictions, INDArray weights, LossReduce lossReduce)

SDVariable absoluteDifference(SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce)
SDVariable absoluteDifference(String name, SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce)
INDArray absoluteDifference(INDArray label, INDArray predictions, INDArray weights)

SDVariable absoluteDifference(SDVariable label, SDVariable predictions, SDVariable weights)
SDVariable absoluteDifference(String name, SDVariable label, SDVariable predictions, SDVariable weights)
```
Absolute difference loss: `sum_i abs( label[i] - predictions[i] )`

* **label**  (NUMERIC) - Label array
* **predictions**  (NUMERIC) - Predictions array
* **weights**  (NUMERIC) - Weights array. May be null. If null, a weight of 1.0 is used
* **lossReduce** - Reduction type for the loss. See LossReduce for more details. Default: LossReduce#MEAN_BY_NONZERO_WEIGHT_COUNT - default = LossReduce.MEAN_BY_NONZERO_WEIGHT_COUNT

## cosineDistance
```JAVA
INDArray cosineDistance(INDArray label, INDArray predictions, INDArray weights, LossReduce lossReduce, int dimension)

SDVariable cosineDistance(SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce, int dimension)
SDVariable cosineDistance(String name, SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce, int dimension)
INDArray cosineDistance(INDArray label, INDArray predictions, INDArray weights, int dimension)

SDVariable cosineDistance(SDVariable label, SDVariable predictions, SDVariable weights, int dimension)
SDVariable cosineDistance(String name, SDVariable label, SDVariable predictions, SDVariable weights, int dimension)
```
Cosine distance loss: `1 - cosineSimilarity(x,y)` or `1 - sum_i label[i] * prediction[i]`, which is

equivalent to cosine distance when both the predictions and labels are normalized.<br>
<b>Note</b>: This loss function assumes that both the predictions and labels are normalized to have unit l2 norm.

If this is not the case, you should normalize them first by dividing by norm2(String, SDVariable, boolean, int...)

along the cosine distance dimension (with keepDims=true).

* **label**  (NUMERIC) - Label array
* **predictions**  (NUMERIC) - Predictions array
* **weights**  (NUMERIC) - Weights array. May be null. If null, a weight of 1.0 is use
* **lossReduce** - Reduction type for the loss. See LossReduce for more details. Default: LossReduce#MEAN_BY_NONZERO_WEIGHT_COUNT - default = LossReduce.MEAN_BY_NONZERO_WEIGHT_COUNT
* **dimension** - Dimension to perform the cosine distance over

## hingeLoss
```JAVA
INDArray hingeLoss(INDArray label, INDArray predictions, INDArray weights, LossReduce lossReduce)

SDVariable hingeLoss(SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce)
SDVariable hingeLoss(String name, SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce)
INDArray hingeLoss(INDArray label, INDArray predictions, INDArray weights)

SDVariable hingeLoss(SDVariable label, SDVariable predictions, SDVariable weights)
SDVariable hingeLoss(String name, SDVariable label, SDVariable predictions, SDVariable weights)
```
Hinge loss: a loss function used for training classifiers.

Implements `L = max(0, 1 - t * predictions)` where t is the label values after internally converting to {-1,1`

from the user specified {0,1`. Note that Labels should be provided with values {0,1`.

* **label**  (NUMERIC) - Label array. Each value should be 0.0 or 1.0 (internally -1 to 1 is used)
* **predictions**  (NUMERIC) - Predictions array
* **weights**  (NUMERIC) - Weights array. May be null. If null, a weight of 1.0 is used
* **lossReduce** - Reduction type for the loss. See LossReduce for more details. Default: LossReduce#MEAN_BY_NONZERO_WEIGHT_COUNT - default = LossReduce.MEAN_BY_NONZERO_WEIGHT_COUNT

## huberLoss
```JAVA
INDArray huberLoss(INDArray label, INDArray predictions, INDArray weights, LossReduce lossReduce, double delta)

SDVariable huberLoss(SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce, double delta)
SDVariable huberLoss(String name, SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce, double delta)
INDArray huberLoss(INDArray label, INDArray predictions, INDArray weights, double delta)

SDVariable huberLoss(SDVariable label, SDVariable predictions, SDVariable weights, double delta)
SDVariable huberLoss(String name, SDVariable label, SDVariable predictions, SDVariable weights, double delta)
```
Huber loss function, used for robust regression. It is similar both squared error loss and absolute difference loss,

though is less sensitive to outliers than squared error.<br>
Huber loss implements:

<pre>

`L = 0.5 * (label[i] - predictions[i])^2 if abs(label[i] - predictions[i]) < delta`

`L = delta * abs(label[i] - predictions[i]) - 0.5 * delta^2 otherwise`

</pre>

* **label**  (NUMERIC) - Label array
* **predictions**  (NUMERIC) - Predictions array
* **weights**  (NUMERIC) - Weights array. May be null. If null, a weight of 1.0 is used
* **lossReduce** - Reduction type for the loss. See LossReduce for more details. Default: LossReduce#MEAN_BY_NONZERO_WEIGHT_COUNT - default = LossReduce.MEAN_BY_NONZERO_WEIGHT_COUNT
* **delta** - Loss function delta value

## l2Loss
```JAVA
INDArray l2Loss(INDArray var)

SDVariable l2Loss(SDVariable var)
SDVariable l2Loss(String name, SDVariable var)
```
L2 loss: 1/2 * sum(x^2)

* **var**  (NUMERIC) - Variable to calculate L2 loss of

## logLoss
```JAVA
INDArray logLoss(INDArray label, INDArray predictions, INDArray weights, LossReduce lossReduce, double epsilon)

SDVariable logLoss(SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce, double epsilon)
SDVariable logLoss(String name, SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce, double epsilon)
INDArray logLoss(INDArray label, INDArray predictions)

SDVariable logLoss(SDVariable label, SDVariable predictions)
SDVariable logLoss(String name, SDVariable label, SDVariable predictions)
```
Log loss, i.e., binary cross entropy loss, usually used for binary multi-label classification. Implements:

`-1/numExamples * sum_i (labels[i] * log(predictions[i] + epsilon) + (1-labels[i]) * log(1-predictions[i] + epsilon))`

* **label**  (NUMERIC) - Label array
* **predictions**  (NUMERIC) - Predictions array
* **weights**  (NUMERIC) - Weights array. May be null. If null, a weight of 1.0 is used
* **lossReduce** - Reduction type for the loss. See LossReduce for more details. Default: LossReduce#MEAN_BY_NONZERO_WEIGHT_COUNT - default = LossReduce.MEAN_BY_NONZERO_WEIGHT_COUNT
* **epsilon** - epsilon - default = 0.0

## logPoisson
```JAVA
INDArray logPoisson(INDArray label, INDArray predictions, INDArray weights, LossReduce lossReduce, boolean full)

SDVariable logPoisson(SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce, boolean full)
SDVariable logPoisson(String name, SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce, boolean full)
INDArray logPoisson(INDArray label, INDArray predictions, INDArray weights, boolean full)

SDVariable logPoisson(SDVariable label, SDVariable predictions, SDVariable weights, boolean full)
SDVariable logPoisson(String name, SDVariable label, SDVariable predictions, SDVariable weights, boolean full)
```
Log poisson loss: a loss function used for training classifiers.

Implements `L = exp(c) - z * c` where c is log(predictions) and z is labels.

* **label**  (NUMERIC) - Label array. Each value should be 0.0 or 1.0
* **predictions**  (NUMERIC) - Predictions array (has to be log(x) of actual predictions)
* **weights**  (NUMERIC) - Weights array. May be null. If null, a weight of 1.0 is used
* **lossReduce** - Reduction type for the loss. See LossReduce for more details. Default: LossReduce#MEAN_BY_NONZERO_WEIGHT_COUNT - default = LossReduce.MEAN_BY_NONZERO_WEIGHT_COUNT
* **full** - Boolean flag. true for logPoissonFull, false for logPoisson

## meanPairwiseSquaredError
```JAVA
INDArray meanPairwiseSquaredError(INDArray label, INDArray predictions, INDArray weights, LossReduce lossReduce)

SDVariable meanPairwiseSquaredError(SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce)
SDVariable meanPairwiseSquaredError(String name, SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce)
INDArray meanPairwiseSquaredError(INDArray label, INDArray predictions, INDArray weights)

SDVariable meanPairwiseSquaredError(SDVariable label, SDVariable predictions, SDVariable weights)
SDVariable meanPairwiseSquaredError(String name, SDVariable label, SDVariable predictions, SDVariable weights)
```
Mean pairwise squared error.<br>
MPWSE loss calculates the difference between pairs of consecutive elements in the predictions and labels arrays.

For example, if predictions = [p0, p1, p2] and labels are [l0, l1, l2] then MPWSE is:

{@code [((p0-p1) - (l0-l1))^2 + ((p0-p2) - (l0-l2))^2 + ((p1-p2) - (l1-l2))^2] / 3}<br>
* **label**  (NUMERIC) - Label array
* **predictions**  (NUMERIC) - Predictions array
* **weights**  (NUMERIC) - Weights array. May be null. If null, a weight of 1.0 is used. Must be either null, scalar, or have shape [batchSize]
* **lossReduce** - Reduction type for the loss. See LossReduce for more details. Default: LossReduce#MEAN_BY_NONZERO_WEIGHT_COUNT - default = LossReduce.MEAN_BY_NONZERO_WEIGHT_COUNT

## meanSquaredError
```JAVA
INDArray meanSquaredError(INDArray label, INDArray predictions, INDArray weights, LossReduce lossReduce)

SDVariable meanSquaredError(SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce)
SDVariable meanSquaredError(String name, SDVariable label, SDVariable predictions, SDVariable weights, LossReduce lossReduce)
INDArray meanSquaredError(INDArray label, INDArray predictions, INDArray weights)

SDVariable meanSquaredError(SDVariable label, SDVariable predictions, SDVariable weights)
SDVariable meanSquaredError(String name, SDVariable label, SDVariable predictions, SDVariable weights)
```
Mean squared error loss function. Implements `(label[i] - prediction[i])^2` - i.e., squared error on a per-element basis.

When averaged (using LossReduce#MEAN_BY_WEIGHT or LossReduce#MEAN_BY_NONZERO_WEIGHT_COUNT (the default))

this is the mean squared error loss function.

* **label**  (NUMERIC) - Label array
* **predictions**  (NUMERIC) - Predictions array
* **weights**  (NUMERIC) - Weights array. May be null. If null, a weight of 1.0 is used
* **lossReduce** - Reduction type for the loss. See LossReduce for more details. Default: LossReduce#MEAN_BY_NONZERO_WEIGHT_COUNT - default = LossReduce.MEAN_BY_NONZERO_WEIGHT_COUNT

## sigmoidCrossEntropy
```JAVA
INDArray sigmoidCrossEntropy(INDArray label, INDArray predictionLogits, INDArray weights, LossReduce lossReduce, double labelSmoothing)

SDVariable sigmoidCrossEntropy(SDVariable label, SDVariable predictionLogits, SDVariable weights, LossReduce lossReduce, double labelSmoothing)
SDVariable sigmoidCrossEntropy(String name, SDVariable label, SDVariable predictionLogits, SDVariable weights, LossReduce lossReduce, double labelSmoothing)
INDArray sigmoidCrossEntropy(INDArray label, INDArray predictionLogits, INDArray weights)

SDVariable sigmoidCrossEntropy(SDVariable label, SDVariable predictionLogits, SDVariable weights)
SDVariable sigmoidCrossEntropy(String name, SDVariable label, SDVariable predictionLogits, SDVariable weights)
```
Sigmoid cross entropy: applies the sigmoid activation function on the input logits (input "pre-sigmoid preductions")

and implements the binary cross entropy loss function. This implementation is numerically more stable than using

standard (but separate) sigmoid activation function and log loss (binary cross entropy) loss function.<br>
Implements:

`-1/numExamples * sum_i (labels[i] * log(sigmoid(logits[i])) + (1-labels[i]) * log(1-sigmoid(logits[i])))`

though this is done in a mathematically equivalent but more numerical stable form.<br>
<br>
When label smoothing is > 0, the following label smoothing is used:<br>
<pre>

`numClasses = labels.size(1);

label = (1.0 - labelSmoothing) * label + 0.5 * labelSmoothing`

</pre>

* **label**  (NUMERIC) - Label array
* **predictionLogits**  (NUMERIC) - Predictions array
* **weights**  (NUMERIC) - Weights array. May be null. If null, a weight of 1.0 is used
* **lossReduce** - Reduction type for the loss. See LossReduce for more details. Default: LossReduce#MEAN_BY_NONZERO_WEIGHT_COUNT - default = LossReduce.MEAN_BY_NONZERO_WEIGHT_COUNT
* **labelSmoothing** - Label smoothing value. Default value: 0 - default = 0.0

## softmaxCrossEntropy
```JAVA
INDArray softmaxCrossEntropy(INDArray oneHotLabels, INDArray logitPredictions, INDArray weights, LossReduce lossReduce, double labelSmoothing)

SDVariable softmaxCrossEntropy(SDVariable oneHotLabels, SDVariable logitPredictions, SDVariable weights, LossReduce lossReduce, double labelSmoothing)
SDVariable softmaxCrossEntropy(String name, SDVariable oneHotLabels, SDVariable logitPredictions, SDVariable weights, LossReduce lossReduce, double labelSmoothing)
INDArray softmaxCrossEntropy(INDArray oneHotLabels, INDArray logitPredictions, INDArray weights)

SDVariable softmaxCrossEntropy(SDVariable oneHotLabels, SDVariable logitPredictions, SDVariable weights)
SDVariable softmaxCrossEntropy(String name, SDVariable oneHotLabels, SDVariable logitPredictions, SDVariable weights)
```
Applies the softmax activation function to the input, then implement multi-class cross entropy:<br>
{@code -sum_classes label[i] * log(p[c])} where {@code p = softmax(logits)}<br>
If LossReduce#NONE is used, returned shape is [numExamples] out for [numExamples, numClasses] predicitons/labels;

otherwise, the output is a scalar.<br>
<p>

When label smoothing is > 0, the following label smoothing is used:<br>
<pre>

`numClasses = labels.size(1);

oneHotLabel = (1.0 - labelSmoothing) * oneHotLabels + labelSmoothing/numClasses`

</pre>

* **oneHotLabels**  (NUMERIC) - Label array. Should be one-hot per example and same shape as predictions (for example, [mb, nOut])
* **logitPredictions**  (NUMERIC) - Predictions array (pre-softmax)
* **weights**  (NUMERIC) - Weights array. May be null. If null, a weight of 1.0 is used
* **lossReduce** - Reduction type for the loss. See LossReduce for more details. Default: LossReduce#MEAN_BY_NONZERO_WEIGHT_COUNT - default = LossReduce.MEAN_BY_NONZERO_WEIGHT_COUNT
* **labelSmoothing** - Label smoothing value. Default value: 0 - default = 0.0

## sparseSoftmaxCrossEntropy
```JAVA
INDArray sparseSoftmaxCrossEntropy(INDArray logits, INDArray labels)

SDVariable sparseSoftmaxCrossEntropy(SDVariable logits, SDVariable labels)
SDVariable sparseSoftmaxCrossEntropy(String name, SDVariable logits, SDVariable labels)
```
As per softmaxCrossEntropy(String, SDVariable, SDVariable, LossReduce) but the labels variable

is represented as an integer array instead of the equivalent one-hot array.<br>
i.e., if logits are rank N, then labels have rank N-1

* **logits**  (NUMERIC) - Logits array ("pre-softmax activations")
* **labels**  (INT) - Labels array. Must be an integer type.

## weightedCrossEntropyWithLogits
```JAVA
INDArray weightedCrossEntropyWithLogits(INDArray targets, INDArray inputs, INDArray weights)

SDVariable weightedCrossEntropyWithLogits(SDVariable targets, SDVariable inputs, SDVariable weights)
SDVariable weightedCrossEntropyWithLogits(String name, SDVariable targets, SDVariable inputs, SDVariable weights)
```
Weighted cross entropy loss with logits

* **targets**  (NUMERIC) - targets array
* **inputs**  (NUMERIC) - input array
* **weights**  (NUMERIC) - eights array. May be null. If null, a weight of 1.0 is used


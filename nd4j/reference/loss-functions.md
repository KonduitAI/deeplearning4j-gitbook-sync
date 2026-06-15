---
title: "Loss Functions"
description: "All loss functions in ND4J — ILossFunction implementations, usage in output layers, weighted loss, and custom loss functions"
---

# Loss Functions

Loss functions measure the discrepancy between a network's predictions and the true labels. During training, the optimizer minimizes the loss to improve predictions. All loss functions in the DL4J ecosystem implement the `ILossFunction` interface at `org.nd4j.linalg.lossfunctions.ILossFunction`.

## Usage

### In Output Layers (Preferred)

Pass an `ILossFunction` instance to the output layer builder:

```java
import org.nd4j.linalg.lossfunctions.impl.LossMCXENT;

new OutputLayer.Builder(new LossMCXENT())
    .nIn(256).nOut(10)
    .activation(Activation.SOFTMAX)
    .build()
```

### Using the LossFunction Enum (Legacy)

The convenience enum still works but instantiating `ILossFunction` directly is preferred in M2.1:

```java
import org.nd4j.linalg.lossfunctions.LossFunctions.LossFunction;

new OutputLayer.Builder(LossFunction.MSE)
    .nIn(256).nOut(1)
    .activation(Activation.IDENTITY)
    .build()
```

### In SameDiff

Use the `sd.loss` namespace:

```java
SDVariable loss = sd.loss.softmaxCrossEntropy("loss", labels, logits, null);
```

## Classification Loss Functions

### LossMCXENT — Multi-Class Cross Entropy

```java
new LossMCXENT()
new LossMCXENT(weights)  // class weights as INDArray
```

The standard loss for **multi-class classification** tasks. Measures the cross entropy between the true distribution (one-hot labels) and the predicted distribution (softmax outputs).

**Formula:** `L = -sum(y_true * log(y_pred))`

**Pair with:** `Activation.SOFTMAX`

**Class weighting:** Pass an `INDArray` of shape `[1, numClasses]` to handle class imbalance:

```java
INDArray classWeights = Nd4j.create(new double[]{1.0, 2.5, 1.0, 3.0});
new OutputLayer.Builder(new LossMCXENT(classWeights))
    .activation(Activation.SOFTMAX)
    .nIn(128).nOut(4)
    .build()
```

### LossSparseMCXENT — Sparse Multi-Class Cross Entropy

```java
new LossSparseMCXENT()
```

Same as `LossMCXENT` but accepts **integer labels** instead of one-hot encoded labels. Labels should be a column vector of class indices (shape `[batchSize, 1]`).

**Pair with:** `Activation.SOFTMAX`

### LossNegativeLogLikelihood — Negative Log Likelihood

```java
new LossNegativeLogLikelihood()
new LossNegativeLogLikelihood(weights)
```

Functionally equivalent to `LossMCXENT` for softmax outputs. The difference is in how the gradient is computed internally.

**Pair with:** `Activation.SOFTMAX`

### LossBinaryXENT — Binary Cross Entropy

```java
new LossBinaryXENT()
new LossBinaryXENT(weights)
```

For **binary classification** or **multi-label classification** where each output is an independent binary decision.

**Formula:** `L = -[y * log(p) + (1-y) * log(1-p)]`

**Pair with:** `Activation.SIGMOID`

```java
// Binary classification (single output)
new OutputLayer.Builder(new LossBinaryXENT())
    .nIn(128).nOut(1)
    .activation(Activation.SIGMOID)
    .build()

// Multi-label classification (multiple independent outputs)
new OutputLayer.Builder(new LossBinaryXENT())
    .nIn(128).nOut(5)    // 5 independent binary labels
    .activation(Activation.SIGMOID)
    .build()
```

### LossHinge — Hinge Loss

```java
new LossHinge()
```

SVM-style loss for classification. Labels should be -1 or +1.

**Formula:** `L = max(0, 1 - y_true * y_pred)`

**Pair with:** `Activation.TANH` (output range -1 to 1)

### LossSquaredHinge — Squared Hinge Loss

```java
new LossSquaredHinge()
```

Smooth variant of hinge loss: `L = max(0, 1 - y_true * y_pred)^2`

### LossFMeasure — F-Measure Loss

```java
new LossFMeasure()
new LossFMeasure(beta)
```

Directly optimizes the F-measure (F1 score by default). For binary classification only.

- `beta = 1.0` (default): F1 score (equal weight to precision and recall)
- `beta < 1.0`: Favors precision
- `beta > 1.0`: Favors recall

### LossMultiLabel — Multi-Label Loss

```java
new LossMultiLabel()
```

Specialized loss for multi-label ranking tasks.

## Regression Loss Functions

### LossMSE — Mean Squared Error

```java
new LossMSE()
```

Standard regression loss. Heavily penalizes large errors.

**Formula:** `L = mean((y_true - y_pred)^2)`

**Pair with:** `Activation.IDENTITY`

```java
new OutputLayer.Builder(new LossMSE())
    .nIn(128).nOut(1)
    .activation(Activation.IDENTITY)
    .build()
```

### LossMAE — Mean Absolute Error

```java
new LossMAE()
```

More robust to outliers than MSE.

**Formula:** `L = mean(|y_true - y_pred|)`

**Pair with:** `Activation.IDENTITY`

### LossL1 — L1 Loss

```java
new LossL1()
```

Sum of absolute differences (not averaged). Encourages sparse predictions.

**Formula:** `L = sum(|y_true - y_pred|)`

### LossL2 — L2 Loss

```java
new LossL2()
```

Sum of squared differences (not averaged).

**Formula:** `L = sum((y_true - y_pred)^2)`

### LossMSLE — Mean Squared Logarithmic Error

```java
new LossMSLE()
```

Useful when target values span several orders of magnitude.

**Formula:** `L = mean((log(y_true + 1) - log(y_pred + 1))^2)`

**Pair with:** `Activation.IDENTITY` (predictions should be non-negative)

### LossMAPE — Mean Absolute Percentage Error

```java
new LossMAPE()
```

Percentage-based regression error.

**Formula:** `L = mean(|y_true - y_pred| / |y_true|) * 100`

### LossPoisson — Poisson Loss

```java
new LossPoisson()
```

For count data regression where the target follows a Poisson distribution.

**Formula:** `L = mean(y_pred - y_true * log(y_pred))`

## Distribution and Similarity Loss Functions

### LossKLD — Kullback-Leibler Divergence

```java
new LossKLD()
```

Measures the divergence between two probability distributions. Used in variational autoencoders and distribution matching.

**Formula:** `L = sum(y_true * log(y_true / y_pred))`

### LossCosineProximity — Cosine Proximity Loss

```java
new LossCosineProximity()
```

Measures the cosine distance between predictions and labels. Useful for similarity learning tasks where direction matters more than magnitude.

**Formula:** `L = -sum(y_true * y_pred) / (||y_true|| * ||y_pred||)`

### LossWasserstein — Wasserstein Loss

```java
new LossWasserstein()
```

Earth Mover's Distance loss. Commonly used in WGAN (Wasserstein GAN) training.

**Formula:** `L = mean(y_true * y_pred)`

## Specialized Loss Functions

### LossMixtureDensity — Mixture Density Network Loss

```java
new LossMixtureDensity(numMixtures)
```

For mixture density networks that output parameters of a Gaussian mixture model (means, variances, mixing coefficients).

## Quick Reference

| Task | Loss Function | Activation | Labels |
|------|--------------|-----------|--------|
| Multi-class classification | `LossMCXENT` | `SOFTMAX` | One-hot |
| Multi-class (integer labels) | `LossSparseMCXENT` | `SOFTMAX` | Integer indices |
| Binary classification | `LossBinaryXENT` | `SIGMOID` | 0/1 |
| Multi-label classification | `LossBinaryXENT` | `SIGMOID` | Binary vector |
| Regression | `LossMSE` | `IDENTITY` | Continuous |
| Robust regression | `LossMAE` | `IDENTITY` | Continuous |
| Log-scale regression | `LossMSLE` | `IDENTITY` | Positive continuous |
| Count data | `LossPoisson` | `IDENTITY` | Non-negative integer |
| Distribution matching | `LossKLD` | `SOFTMAX` | Probabilities |
| Similarity learning | `LossCosineProximity` | varies | Normalized vectors |
| SVM-style | `LossHinge` | `TANH` | -1/+1 |
| GAN (Wasserstein) | `LossWasserstein` | `IDENTITY` | -1/+1 |
| Optimize F1 directly | `LossFMeasure` | `SIGMOID` | 0/1 |

## Custom Loss Functions

Implement `ILossFunction`:

```java
import org.nd4j.linalg.lossfunctions.ILossFunction;
import org.nd4j.linalg.activations.IActivation;
import org.nd4j.common.primitives.Pair;

public class HuberLoss implements ILossFunction {

    private final double delta;

    public HuberLoss(double delta) {
        this.delta = delta;
    }

    @Override
    public double computeScore(INDArray labels, INDArray preOutput,
                               IActivation activationFn, INDArray mask, boolean average) {
        INDArray output = activationFn.getActivation(preOutput.dup(), true);
        INDArray diff = labels.sub(output);
        INDArray absDiff = Transforms.abs(diff);

        // Huber: quadratic for |diff| <= delta, linear for |diff| > delta
        INDArray quadratic = Transforms.min(absDiff, delta);
        INDArray linear = absDiff.sub(quadratic);
        INDArray loss = quadratic.mul(quadratic).mul(0.5).add(linear.mul(delta));

        double score = loss.sumNumber().doubleValue();
        return average ? score / labels.size(0) : score;
    }

    @Override
    public INDArray computeScoreArray(INDArray labels, INDArray preOutput,
                                      IActivation activationFn, INDArray mask) {
        // Return per-example loss
        INDArray output = activationFn.getActivation(preOutput.dup(), true);
        INDArray diff = labels.sub(output);
        INDArray absDiff = Transforms.abs(diff);
        INDArray quadratic = Transforms.min(absDiff, delta);
        INDArray linear = absDiff.sub(quadratic);
        return quadratic.mul(quadratic).mul(0.5).add(linear.mul(delta)).sum(1);
    }

    @Override
    public INDArray computeGradient(INDArray labels, INDArray preOutput,
                                    IActivation activationFn, INDArray mask) {
        INDArray output = activationFn.getActivation(preOutput.dup(), true);
        INDArray diff = output.sub(labels);
        // Clip gradient to [-delta, delta]
        INDArray grad = Transforms.min(Transforms.max(diff, -delta), delta);

        // Chain rule: multiply by activation gradient
        INDArray dLda = grad.div(labels.size(0));
        Pair<INDArray, INDArray> p = activationFn.backprop(preOutput, dLda);
        return p.getFirst();
    }

    @Override
    public Pair<Double, INDArray> computeGradientAndScore(
            INDArray labels, INDArray preOutput, IActivation activationFn,
            INDArray mask, boolean average) {
        return new Pair<>(
            computeScore(labels, preOutput, activationFn, mask, average),
            computeGradient(labels, preOutput, activationFn, mask)
        );
    }

    @Override
    public String name() { return "HuberLoss(delta=" + delta + ")"; }
}
```

Use it like any built-in loss:

```java
new OutputLayer.Builder(new HuberLoss(1.0))
    .nIn(128).nOut(1)
    .activation(Activation.IDENTITY)
    .build()
```

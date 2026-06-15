---
title: "Keras Optimizers"
description: "Keras to DL4J optimizer mapping for model import"
---

## Keras Optimizers

All standard Keras optimizers are supported for import. Optimizer settings are preserved when the Keras model was compiled and saved with `model.save()`. The mapping is implemented in [KerasOptimizerUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasOptimizerUtils.java).

The `TFOptimizer` (a TensorFlow-specific wrapper) is not supported.

---

## Optimizer Mapping Table

| Keras Optimizer | DL4J Equivalent | Supported |
|---|---|---|
| SGD | `Sgd` | Yes |
| RMSprop | `RmsProp` | Yes |
| Adagrad | `AdaGrad` | Yes |
| Adadelta | `AdaDelta` | Yes |
| Adam | `Adam` | Yes |
| Adamax | `AdaMax` | Yes |
| Nadam | `Nadam` | Yes |
| TFOptimizer | — | No |

---

## Optimizer Descriptions

### SGD

Stochastic Gradient Descent with optional momentum, learning rate decay, and Nesterov momentum.

**Keras parameters:**

| Parameter | Default | Description |
|---|---|---|
| `lr` | 0.01 | Learning rate |
| `momentum` | 0.0 | Momentum factor |
| `decay` | 0.0 | Learning rate decay per update |
| `nesterov` | False | Whether to apply Nesterov momentum |

**DL4J mapping:** `org.nd4j.linalg.learning.config.Sgd`

---

### RMSprop

Root Mean Square Propagation. Adapts the learning rate by dividing by a running average of recent gradients.

**Keras parameters:**

| Parameter | Default | Description |
|---|---|---|
| `lr` | 0.001 | Learning rate |
| `rho` | 0.9 | Discounting factor for old gradients |
| `epsilon` | 1e-8 | Fuzz factor for numerical stability |
| `decay` | 0.0 | Learning rate decay |

**DL4J mapping:** `org.nd4j.linalg.learning.config.RmsProp`

---

### Adagrad

Adapts the learning rate for each parameter individually based on the accumulated sum of squared gradients. Good for sparse data.

**Keras parameters:**

| Parameter | Default | Description |
|---|---|---|
| `lr` | 0.01 | Learning rate |
| `epsilon` | 1e-8 | Fuzz factor |
| `decay` | 0.0 | Learning rate decay |

**DL4J mapping:** `org.nd4j.linalg.learning.config.AdaGrad`

---

### Adadelta

An extension of Adagrad that adapts learning rates based on a moving window of gradient updates. No manual learning rate setting required.

**Keras parameters:**

| Parameter | Default | Description |
|---|---|---|
| `lr` | 1.0 | Learning rate (scaling factor) |
| `rho` | 0.95 | Decay factor |
| `epsilon` | 1e-8 | Fuzz factor |
| `decay` | 0.0 | Learning rate decay |

**DL4J mapping:** `org.nd4j.linalg.learning.config.AdaDelta`

---

### Adam

Adaptive Moment Estimation. Combines the advantages of AdaGrad and RMSProp. The most commonly used optimizer for deep learning.

**Keras parameters:**

| Parameter | Default | Description |
|---|---|---|
| `lr` | 0.001 | Learning rate |
| `beta_1` | 0.9 | Exponential decay rate for first moment |
| `beta_2` | 0.999 | Exponential decay rate for second moment |
| `epsilon` | 1e-8 | Fuzz factor |
| `decay` | 0.0 | Learning rate decay |
| `amsgrad` | False | Whether to apply AMSGrad variant |

**DL4J mapping:** `org.nd4j.linalg.learning.config.Adam`

---

### Adamax

A variant of Adam based on the infinity norm. More stable than Adam in some cases.

**Keras parameters:**

| Parameter | Default | Description |
|---|---|---|
| `lr` | 0.002 | Learning rate |
| `beta_1` | 0.9 | Exponential decay rate for first moment |
| `beta_2` | 0.999 | Exponential decay rate for second moment |
| `epsilon` | 1e-8 | Fuzz factor |
| `decay` | 0.0 | Learning rate decay |

**DL4J mapping:** `org.nd4j.linalg.learning.config.AdaMax`

---

### Nadam

Nesterov Adam. Combines Adam with Nesterov momentum for faster convergence in some settings.

**Keras parameters:**

| Parameter | Default | Description |
|---|---|---|
| `lr` | 0.002 | Learning rate |
| `beta_1` | 0.9 | Exponential decay rate for first moment |
| `beta_2` | 0.999 | Exponential decay rate for second moment |
| `epsilon` | 1e-8 | Fuzz factor |
| `schedule_decay` | 0.004 | Decay for the momentum schedule |

**DL4J mapping:** `org.nd4j.linalg.learning.config.Nadam`

---

## Notes on Training Configuration Import

Optimizer settings are only available in DL4J after import when:
1. The Keras model was compiled before saving (`model.compile(...)`)
2. The model was saved with `model.save('model.h5')` (not weights-only or config-only)
3. `enforceTrainingConfig=true` (the default) when calling the import method

If the training configuration is absent or `enforceTrainingConfig=false`, the imported model can still be used for inference but will not have an optimizer configured.

To add an optimizer to an imported model for continued training:

```java
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights("model.h5", false);

// Re-configure for continued training with a new optimizer
model.setLearningRate(0.001);
```

Or rebuild with a `NeuralNetConfiguration.Builder` using the imported configuration as a starting point.

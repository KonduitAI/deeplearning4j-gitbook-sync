---
title: Keras Import Regularizers
description: Mapping of Keras regularizers to DL4J regularization for model import in Eclipse Deeplearning4j 1.0.0-M2.1.
---

## Keras Regularizer Import

Keras allows L1 and L2 regularization penalties to be applied to layer kernels, biases,
and activations. DL4J supports all standard [Keras regularizers](https://keras.io/regularizers)
and maps them to equivalent regularization parameters in the layer configuration.

The mapping is implemented in
[KerasRegularizerUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasRegularizerUtils.java).

### Supported Regularizers

| Keras regularizer | DL4J equivalent             | Description                              |
|-------------------|-----------------------------|------------------------------------------|
| `l1`              | `L1Regularization`          | L1 (Lasso) weight penalty                |
| `l2`              | `L2Regularization`          | L2 (Ridge) weight penalty                |
| `l1_l2`           | `L1L2Regularization`        | Combined L1 and L2 penalty (Elastic Net) |

### Regularizer Descriptions

#### l1

Adds the sum of the absolute values of the weights to the loss function, scaled by the
regularization coefficient. L1 regularization encourages sparsity — many weights will be
driven to exactly zero.

```python
# Keras
from keras.regularizers import l1
layer = Dense(64, kernel_regularizer=l1(0.01))
```

```java
// DL4J equivalent (applied via layer configuration)
.l1(0.01)
```

**Keras parameter:**

| Parameter | Default | Description                       |
|-----------|---------|-----------------------------------|
| `l`       | 0.01    | Regularization coefficient        |

#### l2

Adds the sum of the squared weights to the loss function, scaled by the regularization
coefficient. L2 regularization (weight decay) penalises large weights and generally
produces smoother, more generalisable models.

```python
# Keras
from keras.regularizers import l2
layer = Dense(64, kernel_regularizer=l2(0.01))
```

```java
// DL4J equivalent
.l2(0.01)
```

**Keras parameter:**

| Parameter | Default | Description                       |
|-----------|---------|-----------------------------------|
| `l`       | 0.01    | Regularization coefficient        |

#### l1_l2

Combines L1 and L2 penalties (Elastic Net regularization). Both coefficients can be set
independently.

```python
# Keras
from keras.regularizers import l1_l2
layer = Dense(64, kernel_regularizer=l1_l2(l1=0.01, l2=0.001))
```

```java
// DL4J equivalent
.l1(0.01).l2(0.001)
```

**Keras parameters:**

| Parameter | Default | Description                       |
|-----------|---------|-----------------------------------|
| `l1`      | 0.01    | L1 regularization coefficient     |
| `l2`      | 0.01    | L2 regularization coefficient     |

### Regularization Targets

In Keras, regularization can be applied separately to kernels, biases, and activations:

```python
Dense(64,
      kernel_regularizer=l2(0.01),
      bias_regularizer=l1(0.001),
      activity_regularizer=l2(0.0001))
```

DL4J maps `kernel_regularizer` and `bias_regularizer` directly to the layer's `l1`/`l2`
settings. `activity_regularizer` (applied to the layer output rather than the weights)
is not currently supported during import.

### Usage Example

```java
import org.deeplearning4j.nn.modelimport.keras.KerasModelImport;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;

// Regularizer coefficients are automatically read from the Keras config
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights("model.h5");
```

### Notes

- Regularization affects the **training loss**, not inference. For inference-only import,
  the regularizer config is loaded but has no practical effect.
- Custom Python regularizer classes cannot be imported.
- `activity_regularizer` is not supported; a warning is logged if it is present.

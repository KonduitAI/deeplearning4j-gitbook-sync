---
title: Keras Import Constraints
description: Mapping of Keras weight constraints to DL4J LayerConstraint implementations for model import in Eclipse Deeplearning4j 1.0.0-M2.1.
---

## Keras Constraint Import

Keras allows weight constraints to be applied to layer parameters during training. These
constraints clip or rescale the weights after each gradient update to keep them within a
specified range. DL4J supports all standard Keras constraints via corresponding
`LayerConstraint` implementations.

The mapping is implemented in
[KerasConstraintUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasConstraintUtils.java).

### Supported Constraints

| Keras constraint | DL4J LayerConstraint      | Description                                   |
|------------------|---------------------------|-----------------------------------------------|
| `max_norm`       | `MaxNormConstraint`       | Clips incoming weights so ||w|| ≤ max_value   |
| `non_neg`        | `NonNegativeConstraint`   | Forces all weights to be non-negative         |
| `unit_norm`      | `UnitNormConstraint`      | Scales incoming weights to have unit L2 norm  |
| `min_max_norm`   | `MinMaxNormConstraint`    | Clips weights so min_value ≤ ||w|| ≤ max_value|

### Constraint Descriptions

#### max_norm

Clips the incoming weights for each neuron so that their L2 norm does not exceed
`max_value`. Applied along the `axis` dimension (default: 0, i.e., incoming connections
per unit).

```python
# Keras
from keras.constraints import max_norm
layer = Dense(64, kernel_constraint=max_norm(2.0))
```

```java
// DL4J equivalent
new MaxNormConstraint(2.0, 1)
```

**Keras parameters:**

| Parameter   | Default | Description                              |
|-------------|---------|------------------------------------------|
| `max_value` | 2       | Maximum L2 norm value                    |
| `axis`      | 0       | Axis along which to constrain            |

#### non_neg

Forces all weights to be greater than or equal to zero after each update. Any negative
weights are clipped to zero. Useful for non-negative matrix factorisation tasks.

```python
# Keras
from keras.constraints import non_neg
layer = Dense(64, kernel_constraint=non_neg())
```

```java
// DL4J equivalent
new NonNegativeConstraint()
```

#### unit_norm

Scales incoming weights so that their L2 norm equals 1. Effectively normalises the
weight vector for each unit.

```python
# Keras
from keras.constraints import unit_norm
layer = Dense(64, kernel_constraint=unit_norm())
```

```java
// DL4J equivalent
new UnitNormConstraint(1)
```

#### min_max_norm

Clips incoming weights so that their L2 norm is between `min_value` and `max_value`.
The `rate` parameter controls the speed of clipping (rate=1.0 applies the constraint
exactly; values less than 1.0 apply it more slowly).

```python
# Keras
from keras.constraints import min_max_norm
layer = Dense(64, kernel_constraint=min_max_norm(min_value=0.5, max_value=2.0))
```

```java
// DL4J equivalent
new MinMaxNormConstraint(0.5, 2.0, 1.0, 1)
```

**Keras parameters:**

| Parameter   | Default | Description                              |
|-------------|---------|------------------------------------------|
| `min_value` | 0.0     | Minimum L2 norm value                    |
| `max_value` | 1.0     | Maximum L2 norm value                    |
| `rate`      | 1.0     | Rate at which the constraint is applied  |
| `axis`      | 0       | Axis along which to constrain            |

### Usage Example

```java
import org.deeplearning4j.nn.modelimport.keras.KerasModelImport;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;

// Constraints are automatically read and applied to the imported network configuration
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights("model.h5");
```

### Notes

- Constraints are only relevant when the imported model is used for **continued training**
  in DL4J. For inference, constraints have no effect.
- The constraint `axis` parameter from Keras is mapped to the equivalent DL4J axis
  convention. Verify constraint behaviour when porting models with non-default axis values.
- Custom Python constraint classes cannot be imported.

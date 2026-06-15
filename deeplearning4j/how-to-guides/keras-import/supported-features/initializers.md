---
title: Keras Import Weight Initializers
description: Mapping of Keras weight initializers to DL4J WeightInit implementations for model import in Eclipse Deeplearning4j 1.0.0-M2.1.
---

## Keras Weight Initializer Import

When importing a Keras model, DL4J reads the weight initializer configuration from each
layer and maps it to a DL4J `WeightInit` enum value or `IWeightInit` implementation.
The mapping is implemented in
[KerasInitilizationUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasInitilizationUtils.java).

DL4J supports all [Keras initializers](https://keras.io/initializers).

### Initializer Mapping Table

| Keras initializer   | DL4J WeightInit / IWeightInit          | Notes                               |
|---------------------|----------------------------------------|-------------------------------------|
| `Zeros`             | `WeightInit.ZERO`                      | All weights set to 0                |
| `Ones`              | `WeightInit.ONES`                      | All weights set to 1                |
| `Constant`          | `WeightInitConst`                      | All weights set to a fixed value    |
| `RandomNormal`      | `WeightInit.NORMAL`                    | Gaussian with given mean/stddev     |
| `RandomUniform`     | `WeightInit.UNIFORM`                   | Uniform in [min, max]               |
| `TruncatedNormal`   | `WeightInit.TRUNCATED_NORMAL`          | Gaussian, values >2σ resampled      |
| `VarianceScaling`   | `WeightInitVarScaling`                 | Adapts scale to fan_in/out/avg      |
| `Orthogonal`        | `WeightInit.IDENTITY` (approximation)  | Orthogonal matrix via SVD           |
| `Identity`          | `WeightInit.IDENTITY`                  | Identity matrix (square layers)     |
| `lecun_uniform`     | `WeightInit.LECUN_UNIFORM`             | LeCun uniform fan_in scaling        |
| `lecun_normal`      | `WeightInit.LECUN_NORMAL`              | LeCun normal fan_in scaling         |
| `glorot_normal`     | `WeightInit.XAVIER`                    | Xavier/Glorot normal                |
| `glorot_uniform`    | `WeightInit.XAVIER_UNIFORM`            | Xavier/Glorot uniform               |
| `he_normal`         | `WeightInit.RELU`                      | He normal (for ReLU activations)    |
| `he_uniform`        | `WeightInit.RELU_UNIFORM`              | He uniform                          |

### Initializer Descriptions

#### Zeros and Ones

Constant initializers that set all parameters to a fixed value. Usually used for bias
vectors rather than weight matrices.

```java
// Keras: kernel_initializer='zeros'
// DL4J:
.weightInit(WeightInit.ZERO)
```

#### Constant

Sets all weights to a configurable constant value `c`.

```java
// Keras: kernel_initializer=Constant(value=0.1)
// DL4J (via IWeightInit):
.weightInit(new WeightInitConst(0.1))
```

#### RandomNormal

Draws samples from a normal (Gaussian) distribution with configurable mean and standard
deviation.

```java
// Keras: kernel_initializer=RandomNormal(mean=0.0, stddev=0.05)
// DL4J:
.weightInit(WeightInit.NORMAL)
// Std dev can be configured via distribution:
.dist(new NormalDistribution(0.0, 0.05))
```

#### RandomUniform

Draws samples from a uniform distribution between `minval` and `maxval`.

```java
// Keras: kernel_initializer=RandomUniform(minval=-0.05, maxval=0.05)
// DL4J:
.weightInit(WeightInit.UNIFORM)
.dist(new UniformDistribution(-0.05, 0.05))
```

#### TruncatedNormal

Like `RandomNormal` but values more than two standard deviations from the mean are
discarded and redrawn. Useful for preventing extremely large initial weights.

```java
// Keras: kernel_initializer='truncated_normal'
// DL4J:
.weightInit(WeightInit.TRUNCATED_NORMAL)
```

#### VarianceScaling

Scales the initialisation by a factor based on the number of input units, output units,
or their average, depending on the `mode` and `distribution` settings.

| Keras mode      | Keras distribution | DL4J equivalent            |
|-----------------|--------------------|----------------------------|
| `fan_in`        | `normal`           | `WeightInit.LECUN_NORMAL`  |
| `fan_in`        | `uniform`          | `WeightInit.LECUN_UNIFORM` |
| `fan_avg`       | `normal`           | `WeightInit.XAVIER`        |
| `fan_avg`       | `uniform`          | `WeightInit.XAVIER_UNIFORM`|
| `fan_out`       | `normal`           | `WeightInit.RELU`          |
| `fan_out`       | `uniform`          | `WeightInit.RELU_UNIFORM`  |

#### Orthogonal

Initialises weights as an (approximately) orthogonal matrix, computed via SVD. Helpful for
very deep networks and gradient flow.

#### Identity

Initialises the weight matrix as the identity matrix. Only applicable to square (2D)
weight matrices.

#### Glorot (Xavier) Initializers

The Glorot initializers (`glorot_normal` and `glorot_uniform`) scale the variance of the
initial weights based on the number of input and output units. They are commonly used as
defaults for dense and convolutional layers.

```java
// glorot_uniform (default in many Keras layers)
.weightInit(WeightInit.XAVIER_UNIFORM)

// glorot_normal
.weightInit(WeightInit.XAVIER)
```

#### He Initializers

The He initializers (`he_normal` and `he_uniform`) are designed for layers with ReLU
activations. They use only the fan-in (number of input units) for scaling.

```java
// he_normal
.weightInit(WeightInit.RELU)

// he_uniform
.weightInit(WeightInit.RELU_UNIFORM)
```

### Usage Example

```java
import org.deeplearning4j.nn.modelimport.keras.KerasModelImport;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;

// Weight initializers are automatically mapped during import.
// If the model has already been trained and weights are saved, the
// initializer config is informational only — actual weights are loaded
// from the HDF5 file.
MultiLayerNetwork model = KerasModelImport.importKerasSequentialModelAndWeights("model.h5");
```

### Notes

- When importing a **trained** model (with saved weights), the initializer configuration
  is read but the actual parameter values from the checkpoint are used instead.
- When importing only a **model architecture** (no weights), the initializer determines
  the starting values for any subsequent training in DL4J.
- Custom Python initializers cannot be imported.

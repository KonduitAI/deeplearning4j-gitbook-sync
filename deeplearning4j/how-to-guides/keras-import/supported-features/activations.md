---
title: "Keras Activations"
description: "Keras to DL4J activation function mapping for model import"
---

## Keras Activations

All standard Keras activation functions are supported for model import. Activation functions are mapped by [KerasActivationUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasActivationUtils.java).

Activations can appear in two forms in a Keras model:
1. As a parameter to a layer (e.g., `Dense(64, activation='relu')`)
2. As a standalone `Activation` layer (e.g., `model.add(Activation('relu'))`)

Both forms are supported.

---

## Activation Mapping Table

| Keras Activation | DL4J Activation Class | Supported |
|---|---|---|
| `softmax` | `ActivationSoftmax` | Yes |
| `elu` | `ActivationELU` | Yes |
| `selu` | `ActivationSELU` | Yes |
| `softplus` | `ActivationSoftPlus` | Yes |
| `softsign` | `ActivationSoftSign` | Yes |
| `relu` | `ActivationReLU` | Yes |
| `tanh` | `ActivationTanH` | Yes |
| `sigmoid` | `ActivationSigmoid` | Yes |
| `hard_sigmoid` | `ActivationHardSigmoid` | Yes |
| `linear` | `ActivationIdentity` | Yes |

---

## Activation Descriptions

### softmax

Normalizes a vector of real numbers to a probability distribution over K categories. Used in the output layer for multi-class classification.

```
softmax(x)_i = exp(x_i) / sum(exp(x_j))
```

### elu

Exponential Linear Unit. For positive inputs, acts as identity; for negative inputs, exponentially approaches a negative saturation value. Mitigates the "dying ReLU" problem.

```
elu(x) = x         if x >= 0
         alpha * (exp(x) - 1)  if x < 0
```

Default `alpha` = 1.0.

### selu

Scaled ELU. A self-normalizing variant of ELU. Used with `lecun_normal` initializer and `AlphaDropout` for self-normalizing neural networks (SNNs).

```
selu(x) = lambda * x                    if x > 0
          lambda * alpha * (exp(x) - 1) if x <= 0
```

Where `lambda = 1.0507` and `alpha = 1.6733`.

### softplus

Smooth approximation to ReLU. Always differentiable.

```
softplus(x) = log(1 + exp(x))
```

### softsign

Bounded, smooth activation similar to tanh but with lighter tails.

```
softsign(x) = x / (1 + |x|)
```

### relu

Rectified Linear Unit. The most common hidden layer activation. Simple and effective, but can suffer from "dying ReLU" if neurons become permanently inactive.

```
relu(x) = max(0, x)
```

### tanh

Hyperbolic tangent. Output range [-1, 1]. Commonly used in recurrent networks.

```
tanh(x) = (exp(x) - exp(-x)) / (exp(x) + exp(-x))
```

### sigmoid

Logistic sigmoid. Output range [0, 1]. Used in binary classification output layers.

```
sigmoid(x) = 1 / (1 + exp(-x))
```

### hard_sigmoid

Piecewise linear approximation of sigmoid. Faster to compute.

```
hard_sigmoid(x) = 0             if x < -2.5
                  1             if x > 2.5
                  0.2*x + 0.5  otherwise
```

### linear

Identity function. No transformation. Used when you want a regression output without activation.

```
linear(x) = x
```

---

## Using Activations in Keras

### As a Layer Parameter

```python
from keras.layers import Dense

# Activation as a string parameter
model.add(Dense(64, activation='relu'))
model.add(Dense(10, activation='softmax'))
```

### As a Standalone Layer

```python
from keras.layers import Dense, Activation

model.add(Dense(64))
model.add(Activation('relu'))
```

Both forms produce equivalent DL4J models when imported.

---

## Advanced Activations

Activations with learnable parameters (LeakyReLU, PReLU, ELU with alpha, ThresholdedReLU) are implemented as separate Keras layers in the `keras.layers.advanced_activations` module. See [Advanced Activation Layers](./layers-advanced-activations) for details.

---

## Notes

- The Keras `relu` activation does not include a learnable negative slope. For a learnable slope, use `PReLU` or `LeakyReLU` from the advanced activations module.
- The `selu` activation is designed to be used with specific initialization (`lecun_normal`) and dropout (`AlphaDropout`). If a model uses `selu` with other configurations, the theoretical self-normalizing properties will not apply, though the activation itself will still import and run correctly.
- Custom activation functions defined in Keras via `keras.backend` operations cannot be imported. Only the standard named activations listed above are supported.

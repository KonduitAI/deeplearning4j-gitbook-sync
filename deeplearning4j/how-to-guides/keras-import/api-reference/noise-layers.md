---
title: Keras Noise Layer Import
description: DL4J equivalents and API reference for Keras noise layers — GaussianNoise, GaussianDropout, and AlphaDropout.
---

## Keras Noise Layer Import

Noise layers add stochastic regularization during training. Support is implemented in the [layers/noise](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/noise) package.

### Support Summary

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| GaussianNoise | DropoutLayer (GaussianNoise) | Yes |
| GaussianDropout | DropoutLayer (GaussianDropout) | Yes |
| AlphaDropout | DropoutLayer (AlphaDropout) | Yes |

---

## KerasGaussianNoise

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/noise/KerasGaussianNoise.java)

Imports a Keras `GaussianNoise` layer as a DL4J `DropoutLayer` with `GaussianNoise` applied to the activations during training. The standard deviation `stddev` is read from the Keras configuration.

```
output = input + N(0, stddev)  during training
output = input                 during inference
```

### Constructor

```java
public KerasGaussianNoise(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

### getGaussianNoiseLayer

```java
public DropoutLayer getGaussianNoiseLayer()
```

Returns the DL4J `DropoutLayer` configured with Gaussian noise.

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

---

## KerasGaussianDropout

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/noise/KerasGaussianDropout.java)

Imports a Keras `GaussianDropout` layer as a DL4J `DropoutLayer` with `GaussianDropout`. This multiplies activations by a Gaussian noise factor with mean 1 and variance derived from the dropout rate `rate`:

```
stddev = sqrt(rate / (1 - rate))
output = input * N(1, stddev)
```

### Constructor

```java
public KerasGaussianDropout(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

### getGaussianDropoutLayer

```java
public DropoutLayer getGaussianDropoutLayer()
```

Returns the DL4J `DropoutLayer` configured with Gaussian dropout.

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

---

## KerasAlphaDropout

[source](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/noise/KerasAlphaDropout.java)

Imports a Keras `AlphaDropout` layer as a DL4J `DropoutLayer` with `AlphaDropout`. AlphaDropout is designed for use with SELU activations; it preserves the mean and variance of activations after applying dropout by setting dropped-out units to the negative saturation value of SELU.

### Constructor

```java
public KerasAlphaDropout(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

### getAlphaDropoutLayer

```java
public DropoutLayer getAlphaDropoutLayer()
```

Returns the DL4J `DropoutLayer` configured with AlphaDropout.

### getOutputType

```java
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

---

## Notes

- All noise layers are applied only during training. At inference time they are identity functions. DL4J handles this through the `LayerWorkspaceMgr` training/test mode flag.
- `AlphaDropout` is intended to be used with `SELU` activations and the `lecun_normal` initializer. If your Keras model used this combination, verify that the DL4J model is also initialized with `lecun_normal` (which it will be if weights are loaded from HDF5).
- Noise layers have no trainable parameters; `setWeights` is a no-op for these classes.

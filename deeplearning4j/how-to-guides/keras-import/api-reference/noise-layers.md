# Noise Layers

## KerasGaussianNoise

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/noise/KerasGaussianNoise.java)

Keras wrapper for DL4J dropout layer with GaussianNoise.

**KerasGaussianNoise**

```
public KerasGaussianNoise(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getOutputType**

```
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getGaussianNoiseLayer**

```
public DropoutLayer getGaussianNoiseLayer()
```

Get DL4J DropoutLayer with Gaussian dropout.

* return DropoutLayer

## KerasAlphaDropout

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/noise/KerasAlphaDropout.java)

Keras wrapper for DL4J dropout layer with AlphaDropout.

**KerasAlphaDropout**

```
public KerasAlphaDropout(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getOutputType**

```
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getAlphaDropoutLayer**

```
public DropoutLayer getAlphaDropoutLayer()
```

Get DL4J DropoutLayer with Alpha dropout.

* return DropoutLayer

## KerasGaussianDropout

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/noise/KerasGaussianDropout.java)

Keras wrapper for DL4J dropout layer with GaussianDropout.

**KerasGaussianDropout**

```
public KerasGaussianDropout(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Invalid Keras config

**getOutputType**

```
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getGaussianDropoutLayer**

```
public DropoutLayer getGaussianDropoutLayer()
```

Get DL4J DropoutLayer with Gaussian dropout.

* return DropoutLayer

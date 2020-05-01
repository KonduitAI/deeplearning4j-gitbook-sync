# Advanced Activations

## KerasPReLU

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/advanced/activations/KerasPReLU.java)

Imports PReLU layer from Keras

**KerasPReLU**

```text
public KerasPReLU(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Invalid Keras config

**getOutputType**

```text
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Invalid Keras config

**getPReLULayer**

```text
public PReLULayer getPReLULayer() 
```

Get DL4J ActivationLayer.

* return ActivationLayer

**setWeights**

```text
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException 
```

Set weights for layer.

* param weights Dense layer weights

## KerasThresholdedReLU

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/advanced/activations/KerasThresholdedReLU.java)

Imports ThresholdedReLU layer from Keras

**KerasThresholdedReLU**

```text
public KerasThresholdedReLU(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Invalid Keras config

**getOutputType**

```text
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Invalid Keras config

**getActivationLayer**

```text
public ActivationLayer getActivationLayer() 
```

Get DL4J ActivationLayer.

* return ActivationLayer

## KerasLeakyReLU

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/advanced/activations/KerasLeakyReLU.java)

Imports LeakyReLU layer from Keras

**KerasLeakyReLU**

```text
public KerasLeakyReLU(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Invalid Keras config

**getOutputType**

```text
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Invalid Keras config

**getActivationLayer**

```text
public ActivationLayer getActivationLayer() 
```

Get DL4J ActivationLayer.

* return ActivationLayer


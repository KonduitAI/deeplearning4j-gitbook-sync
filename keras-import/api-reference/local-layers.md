# Local Layers

## KerasLocallyConnected1D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/local/KerasLocallyConnected1D.java)

Imports a 1D locally connected layer from Keras.

**KerasLocallyConnected1D**

```text
public KerasLocallyConnected1D(Integer kerasVersion) throws UnsupportedKerasConfigurationException 
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getLocallyConnected1DLayer**

```text
public LocallyConnected1D getLocallyConnected1DLayer() 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getOutputType**

```text
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException 
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException Invalid Keras config

**setWeights**

```text
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException 
```

Set weights for 1D locally connected layer.

* param weights Map from parameter name to INDArray.

## KerasLocallyConnected2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/local/KerasLocallyConnected2D.java)

Imports a 2D locally connected layer from Keras.

**KerasLocallyConnected2D**

```text
public KerasLocallyConnected2D(Integer kerasVersion) throws UnsupportedKerasConfigurationException 
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getLocallyConnected2DLayer**

```text
public LocallyConnected2D getLocallyConnected2DLayer() 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getOutputType**

```text
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException 
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException Invalid Keras config

**setWeights**

```text
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException 
```

Set weights for 2D locally connected layer.

* param weights Map from parameter name to INDArray.


# Normalization Layers

#### KerasBatchNormalization <a id="kerasbatchnormalization"></a>

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/normalization/KerasBatchNormalization.java)

Imports a BatchNormalization layer from Keras.

**KerasBatchNormalization**

```text
public KerasBatchNormalization(Integer kerasVersion) throws UnsupportedKerasConfigurationException 
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getBatchNormalizationLayer**

```text
public BatchNormalization getBatchNormalizationLayer() 
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

**getNumParams**

```text
public int getNumParams() 
```

Returns number of trainable parameters in layer.

* return number of trainable parameters \(4\)

**setWeights**

```text
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException 
```

Set weights for layer.

* param weights Map from parameter name to INDArray.


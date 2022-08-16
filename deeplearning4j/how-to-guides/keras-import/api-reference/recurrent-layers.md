# Recurrent Layers

## KerasSimpleRnn

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/recurrent/KerasSimpleRnn.java)

Imports a Keras SimpleRNN layer as a DL4J SimpleRnn layer.

**KerasSimpleRnn**

```
public KerasSimpleRnn(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getSimpleRnnLayer**

```
public Layer getSimpleRnnLayer()
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getOutputType**

```
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException Invalid Keras config

**getNumParams**

```
public int getNumParams()
```

Returns number of trainable parameters in layer.

* return number of trainable parameters (12)

**getInputPreprocessor**

```
public InputPreProcessor getInputPreprocessor(InputType... inputType) throws InvalidKerasConfigurationException
```

Gets appropriate DL4J InputPreProcessor for given InputTypes.

* param inputType Array of InputTypes
* return DL4J InputPreProcessor
* throws InvalidKerasConfigurationException Invalid Keras configuration exception
* see org.deeplearning4j.nn.conf.InputPreProcessor

**getUnroll**

```
public boolean getUnroll()
```

Get whether SimpleRnn layer should be unrolled (for truncated BPTT).

* return whether RNN should be unrolled (boolean)

**setWeights**

```
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

Set weights for layer.

* param weights Simple RNN weights
* throws InvalidKerasConfigurationException Invalid Keras configuration exception

## KerasRnnUtils

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/recurrent/KerasRnnUtils.java)

Utility functions for Keras RNN layers

**getUnrollRecurrentLayer**

```
public static boolean getUnrollRecurrentLayer(KerasLayerConfiguration conf, Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException
```

Get unroll parameter to decide whether to unroll RNN with BPTT or not.

* param conf KerasLayerConfiguration
* param layerConfig dictionary containing Keras layer properties
* return boolean unroll parameter
* throws InvalidKerasConfigurationException Invalid Keras configuration

**getRecurrentDropout**

```
public static double getRecurrentDropout(KerasLayerConfiguration conf, Map<String, Object> layerConfig)
            throws UnsupportedKerasConfigurationException, InvalidKerasConfigurationException
```

Get recurrent weight dropout from Keras layer configuration. Non-zero dropout rates are currently not supported.

* param conf KerasLayerConfiguration
* param layerConfig dictionary containing Keras layer properties
* return recurrent dropout rate
* throws InvalidKerasConfigurationException Invalid Keras configuration

## KerasLSTM

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/recurrent/KerasLSTM.java)

Imports a Keras LSTM layer as a DL4J LSTM layer.

**KerasLSTM**

```
public KerasLSTM(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getLSTMLayer**

```
public Layer getLSTMLayer()
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getOutputType**

```
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException Invalid Keras config

**getNumParams**

```
public int getNumParams()
```

Returns number of trainable parameters in layer.

* return number of trainable parameters (12)

**getInputPreprocessor**

```
public InputPreProcessor getInputPreprocessor(InputType... inputType) throws InvalidKerasConfigurationException
```

Gets appropriate DL4J InputPreProcessor for given InputTypes.

* param inputType Array of InputTypes
* return DL4J InputPreProcessor
* throws InvalidKerasConfigurationException Invalid Keras configuration exception
* see org.deeplearning4j.nn.conf.InputPreProcessor

**setWeights**

```
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

Set weights for layer.

* param weights LSTM layer weights

**getUnroll**

```
public boolean getUnroll()
```

Get whether LSTM layer should be unrolled (for truncated BPTT).

* return whether to unroll the LSTM

**getGateActivationFromConfig**

```
public IActivation getGateActivationFromConfig(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Get LSTM gate activation function from Keras layer configuration.

* param layerConfig dictionary containing Keras layer configuration
* return LSTM inner activation function
* throws InvalidKerasConfigurationException Invalid Keras config

**getForgetBiasInitFromConfig**

```
public double getForgetBiasInitFromConfig(Map<String, Object> layerConfig, boolean train)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Get LSTM forget gate bias initialization from Keras layer configuration.

* param layerConfig dictionary containing Keras layer configuration
* return LSTM forget gate bias init
* throws InvalidKerasConfigurationException Unsupported Keras config

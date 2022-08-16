# Pooling Layers

## KerasPooling1D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPooling1D.java)

Imports a Keras 1D Pooling layer as a DL4J Subsampling layer.

**KerasPooling1D**

```
public KerasPooling1D(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getSubsampling1DLayer**

```
public Subsampling1DLayer getSubsampling1DLayer()
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
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

## KerasPoolingUtils

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPoolingUtils.java)

Utility functionality for Keras pooling layers.

**mapPoolingType**

```
public static PoolingType mapPoolingType(String className, KerasLayerConfiguration conf)
            throws UnsupportedKerasConfigurationException
```

Map Keras pooling layers to DL4J pooling types.

* param className name of the Keras pooling class
* return DL4J pooling type
* throws UnsupportedKerasConfigurationException Unsupported Keras config

## KerasPooling3D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPooling3D.java)

Imports a Keras 3D Pooling layer as a DL4J Subsampling3D layer.

**KerasPooling3D**

```
public KerasPooling3D(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getSubsampling3DLayer**

```
public Subsampling3DLayer getSubsampling3DLayer()
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
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

## KerasGlobalPooling

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasGlobalPooling.java)

Imports a Keras Pooling layer as a DL4J Subsampling layer.

**KerasGlobalPooling**

```
public KerasGlobalPooling(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getGlobalPoolingLayer**

```
public GlobalPoolingLayer getGlobalPoolingLayer()
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getInputPreprocessor**

```
public InputPreProcessor getInputPreprocessor(InputType... inputType) throws InvalidKerasConfigurationException
```

Gets appropriate DL4J InputPreProcessor for given InputTypes.

* param inputType Array of InputTypes
* return DL4J InputPreProcessor
* throws InvalidKerasConfigurationException Invalid Keras config
* see org.deeplearning4j.nn.conf.InputPreProcessor

**getOutputType**

```
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException Invalid Keras config

## KerasPooling2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPooling2D.java)

Imports a Keras 2D Pooling layer as a DL4J Subsampling layer.

**KerasPooling2D**

```
public KerasPooling2D(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getSubsampling2DLayer**

```
public SubsamplingLayer getSubsampling2DLayer()
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
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

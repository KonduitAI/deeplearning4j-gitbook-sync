# Convolutional Layers

## KerasConvolution2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasConvolution2D.java)

Imports a 2D Convolution layer from Keras.

**KerasConvolution2D**

```text
public KerasConvolution2D(Integer kerasVersion) throws UnsupportedKerasConfigurationException 
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getConvolution2DLayer**

```text
public ConvolutionLayer getConvolution2DLayer() 
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

## KerasCropping2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasCropping2D.java)

Imports a Keras Cropping 2D layer.

**KerasCropping2D**

```text
public KerasCropping2D(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getCropping2DLayer**

```text
public Cropping2D getCropping2DLayer() 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
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

## KerasUpsampling3D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasUpsampling3D.java)

Keras Upsampling3D layer support

**KerasUpsampling3D**

```text
public KerasUpsampling3D(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras configuration exception
* throws UnsupportedKerasConfigurationException Unsupported Keras configuration exception

**getUpsampling3DLayer**

```text
public Upsampling3D getUpsampling3DLayer() 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras configuration exception
* throws UnsupportedKerasConfigurationException Invalid Keras configuration exception

**getOutputType**

```text
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException 
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException Invalid Keras config

## KerasConvolution1D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasConvolution1D.java)

Imports a 1D Convolution layer from Keras.

**KerasConvolution1D**

```text
public KerasConvolution1D(Integer kerasVersion) throws UnsupportedKerasConfigurationException 
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException

**getConvolution1DLayer**

```text
public Convolution1DLayer getConvolution1DLayer() 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException
* throws UnsupportedKerasConfigurationException

**getOutputType**

```text
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException 
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException

**getInputPreprocessor**

```text
public InputPreProcessor getInputPreprocessor(InputType... inputType) throws InvalidKerasConfigurationException 
```

Gets appropriate DL4J InputPreProcessor for given InputTypes.

* param inputType Array of InputTypes
* return DL4J InputPreProcessor
* throws InvalidKerasConfigurationException Invalid Keras configuration exception
* see org.deeplearning4j.nn.conf.InputPreProcessor

**setWeights**

```text
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException 
```

Set weights for layer.

* param weights Map from parameter name to INDArray.

## KerasUpsampling1D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasUpsampling1D.java)

Keras Upsampling1D layer support

**KerasUpsampling1D**

```text
public KerasUpsampling1D(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras configuration exception
* throws UnsupportedKerasConfigurationException Unsupported Keras configuration exception

**getUpsampling1DLayer**

```text
public Upsampling1D getUpsampling1DLayer() 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras configuration exception
* throws UnsupportedKerasConfigurationException Invalid Keras configuration exception

**getOutputType**

```text
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException 
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException Invalid Keras config

## KerasAtrousConvolution2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasAtrousConvolution2D.java)

Keras 1D atrous / dilated convolution layer. Note that in keras 2 this layer has been removed and dilations are now available through the “dilated” argument in regular Conv1D layers

author: Max Pumperla

**KerasAtrousConvolution2D**

```text
public KerasAtrousConvolution2D(Integer kerasVersion) throws UnsupportedKerasConfigurationException 
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getAtrousConvolution2D**

```text
public ConvolutionLayer getAtrousConvolution2D() 
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

## KerasAtrousConvolution1D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasAtrousConvolution1D.java)

Keras 1D atrous / dilated convolution layer. Note that in keras 2 this layer has been removed and dilations are now available through the “dilated” argument in regular Conv1D layers

author: Max Pumperla

**KerasAtrousConvolution1D**

```text
public KerasAtrousConvolution1D(Integer kerasVersion) throws UnsupportedKerasConfigurationException 
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getAtrousConvolution1D**

```text
public Convolution1DLayer getAtrousConvolution1D() 
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

## KerasCropping3D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasCropping3D.java)

Imports a Keras Cropping 3D layer.

**KerasCropping3D**

```text
public KerasCropping3D(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getCropping3DLayer**

```text
public Cropping3D getCropping3DLayer() 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
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

## KerasZeroPadding2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasZeroPadding2D.java)

Imports a Keras ZeroPadding 2D layer.

**KerasZeroPadding2D**

```text
public KerasZeroPadding2D(Map<String, Object> layerConfig)
                    throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getZeroPadding2DLayer**

```text
public ZeroPaddingLayer getZeroPadding2DLayer() 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
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

## KerasConvolution3D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasConvolution3D.java)

Imports a 3D Convolution layer from Keras.

**KerasConvolution3D**

```text
public KerasConvolution3D(Integer kerasVersion) throws UnsupportedKerasConfigurationException 
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getConvolution3DLayer**

```text
public ConvolutionLayer getConvolution3DLayer() 
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

## KerasDeconvolution2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasDeconvolution2D.java)

Imports a 2D Deconvolution layer from Keras.

**KerasDeconvolution2D**

```text
public KerasDeconvolution2D(Integer kerasVersion) throws UnsupportedKerasConfigurationException 
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getDeconvolution2DLayer**

```text
public Deconvolution2D getDeconvolution2DLayer() 
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

## KerasZeroPadding3D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasZeroPadding3D.java)

Imports a Keras ZeroPadding 3D layer.

**KerasZeroPadding3D**

```text
public KerasZeroPadding3D(Map<String, Object> layerConfig)
                    throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getZeroPadding3DLayer**

```text
public ZeroPadding3DLayer getZeroPadding3DLayer() 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
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

## KerasConvolutionUtils

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasConvolutionUtils.java)

Utility functionality for Keras convolution layers.

**getConvolutionModeFromConfig**

```text
public static ConvolutionMode getConvolutionModeFromConfig(Map<String, Object> layerConfig,
                                                               KerasLayerConfiguration conf)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Get \(convolution\) stride from Keras layer configuration.

* param layerConfig dictionary containing Keras layer configuration
* return Strides array from Keras configuration
* throws InvalidKerasConfigurationException Invalid Keras config

## KerasZeroPadding1D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasZeroPadding1D.java)

Imports a Keras ZeroPadding 1D layer.

**KerasZeroPadding1D**

```text
public KerasZeroPadding1D(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getZeroPadding1DLayer**

```text
public ZeroPadding1DLayer getZeroPadding1DLayer() 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
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

## KerasCropping1D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasCropping1D.java)

Imports a Keras Cropping 1D layer.

**KerasCropping1D**

```text
public KerasCropping1D(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getCropping1DLayer**

```text
public Cropping1D getCropping1DLayer() 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
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

## KerasSpaceToDepth

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasSpaceToDepth.java)

Constructor from parsed Keras layer configuration dictionary.

**KerasSpaceToDepth**

```text
public KerasSpaceToDepth(Map<String, Object> layerConfig, boolean enforceTrainingConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras configuration exception
* throws UnsupportedKerasConfigurationException Unsupported Keras configuration exception

**getSpaceToDepthLayer**

```text
public SpaceToDepthLayer getSpaceToDepthLayer() 
```

Get DL4J SpaceToDepth layer.

* return SpaceToDepth layer

**getOutputType**

```text
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException 
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException Invalid Keras config

## KerasUpsampling2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasUpsampling2D.java)

Keras Upsampling2D layer support

**KerasUpsampling2D**

```text
public KerasUpsampling2D(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras configuration exception
* throws UnsupportedKerasConfigurationException Unsupported Keras configuration exception

**getUpsampling2DLayer**

```text
public Upsampling2D getUpsampling2DLayer() 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras configuration exception
* throws UnsupportedKerasConfigurationException Invalid Keras configuration exception

**getOutputType**

```text
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException 
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException Invalid Keras config

## KerasSeparableConvolution2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasSeparableConvolution2D.java)

Keras separable convolution 2D layer support

**KerasSeparableConvolution2D**

```text
public KerasSeparableConvolution2D(Integer kerasVersion) throws UnsupportedKerasConfigurationException 
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras configuration

**setWeights**

```text
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras configuration
* throws UnsupportedKerasConfigurationException Unsupported Keras configuration

**getSeparableConvolution2DLayer**

```text
public SeparableConvolution2D getSeparableConvolution2DLayer() 
```

Get DL4J SeparableConvolution2D.

* return SeparableConvolution2D

**getOutputType**

```text
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException 
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException Invalid Keras config

## KerasDepthwiseConvolution2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasDepthwiseConvolution2D.java)

Keras depth-wise convolution 2D layer support

**KerasDepthwiseConvolution2D**

```text
public KerasDepthwiseConvolution2D(Integer kerasVersion) throws UnsupportedKerasConfigurationException 
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras configuration

**setWeights**

```text
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException 
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras configuration
* throws UnsupportedKerasConfigurationException Unsupported Keras configuration

**getDepthwiseConvolution2DLayer**

```text
public DepthwiseConvolution2D getDepthwiseConvolution2DLayer() 
```

Get DL4J DepthwiseConvolution2D.

* return DepthwiseConvolution2D

**getOutputType**

```text
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException 
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException Invalid Keras config


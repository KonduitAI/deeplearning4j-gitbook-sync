# Core Layers

## KerasPermute

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasPermute.java)

Imports Permute layer from Keras

**KerasPermute**

```
public KerasPermute(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**isInputPreProcessor**

```
public boolean isInputPreProcessor()
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getInputPreprocessor**

```
public InputPreProcessor getInputPreprocessor(InputType... inputType) throws
            InvalidKerasConfigurationException
```

Gets appropriate DL4J InputPreProcessor for given InputTypes.

* param inputType Array of InputTypes
* return DL4J InputPreProcessor
* throws InvalidKerasConfigurationException Invalid Keras config
* see InputPreProcessor

**getOutputType**

```
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Get layer output type.

* param inputType Array of InputTypes
* return output type as InputType
* throws InvalidKerasConfigurationException Invalid Keras config

## KerasFlatten

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasFlatten.java)

Imports a Keras Flatten layer as a DL4J {Cnn,Rnn}ToFeedForwardInputPreProcessor.

**KerasFlatten**

```
public KerasFlatten(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**isInputPreProcessor**

```
public boolean isInputPreProcessor()
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

## KerasReshape

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasReshape.java)

Imports Reshape layer from Keras

**KerasReshape**

```
public KerasReshape(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**isInputPreProcessor**

```
public boolean isInputPreProcessor()
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

## KerasMerge

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasMerge.java)

Imports a Keras Merge layer as a DL4J Merge (graph) vertex.

TODO: handle axes arguments that alter merge behavior (requires changes to DL4J?)

**KerasMerge**

```
public KerasMerge(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getOutputType**

```
public InputType getOutputType(InputType... inputType)
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration.
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

## KerasDropout

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasDropout.java)

Imports a Dropout layer from Keras.

**KerasDropout**

```
public KerasDropout(Map<String, Object> layerConfig)
                    throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getOutputType**

```
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getDropoutLayer**

```
public DropoutLayer getDropoutLayer()
```

Get DL4J DropoutLayer.

* return DropoutLayer

## KerasMasking

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasMasking.java)

Imports Keras masking layers.

**KerasMasking**

```
public KerasMasking(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getOutputType**

```
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getMaskingLayer**

```
public MaskZeroLayer getMaskingLayer()
```

Get DL4J MaskZeroLayer.

* return MaskZeroLayer

## KerasSpatialDropout

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasSpatialDropout.java)

Keras wrapper for DL4J dropout layer with SpatialDropout, works 1D-3D.

**KerasSpatialDropout**

```
public KerasSpatialDropout(Integer kerasVersion) throws UnsupportedKerasConfigurationException
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

**getSpatialDropoutLayer**

```
public DropoutLayer getSpatialDropoutLayer()
```

Get DL4J DropoutLayer with spatial dropout.

* return DropoutLayer

## KerasLambda

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasLambda.java)

Wraps a DL4J SameDiffLambda into a KerasLayer

**KerasLambda**

```
public KerasLambda(Map<String, Object> layerConfig, SameDiffLayer sameDiffLayer)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getOutputType**

```
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getSameDiffLayer**

```
public SameDiffLayer getSameDiffLayer()
```

Get DL4J SameDiffLayer.

* return SameDiffLayer

## KerasActivation

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasActivation.java)

Imports an Activation layer from Keras.

**KerasActivation**

```
public KerasActivation(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getOutputType**

```
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getActivationLayer**

```
public ActivationLayer getActivationLayer()
```

Get DL4J ActivationLayer.

* return ActivationLayer

## KerasDense

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasDense.java)

Imports a Dense layer from Keras.

**KerasDense**

```
public KerasDense(Integer kerasVersion) throws UnsupportedKerasConfigurationException
```

Pass-through constructor from KerasLayer

* param kerasVersion major keras version
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getDenseLayer**

```
public DenseLayer getDenseLayer()
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
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

* return number of trainable parameters (2)

**setWeights**

```
public void setWeights(Map<String, INDArray> weights) throws InvalidKerasConfigurationException
```

Set weights for layer.

* param weights Dense layer weights

## KerasRepeatVector

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasRepeatVector.java)

Imports a Keras RepeatVector layer

**KerasRepeatVector**

```
public KerasRepeatVector(Map<String, Object> layerConfig)
            throws InvalidKerasConfigurationException, UnsupportedKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getOutputType**

```
public InputType getOutputType(InputType... inputType) throws InvalidKerasConfigurationException
```

Constructor from parsed Keras layer configuration dictionary.

* param layerConfig dictionary containing Keras layer configuration
* param enforceTrainingConfig whether to enforce training-related configuration options
* throws InvalidKerasConfigurationException Invalid Keras config
* throws UnsupportedKerasConfigurationException Unsupported Keras config

**getRepeatVectorLayer**

```
public RepeatVector getRepeatVectorLayer()
```

Get DL4J RepeatVector.

* return RepeatVector

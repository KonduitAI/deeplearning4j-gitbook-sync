---
title: Keras Import Supported Features
short_title: Supported Features
description: Supported Keras features.
category: Keras Import
weight: 2
---

# Supported Features

{% hint style="danger" %}
tf.keras import is not supported yet.
{% endhint %}

## Keras Model Import: Supported Features

While not every concept in DL4J has an equivalent in Keras and vice versa, many of the key concepts can be matched. Importing keras models into DL4J is done in our [deeplearning4j-modelimport](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras) module. Below is a comprehensive list of currently supported features.

* [Layers](./#layers)
* [Losses](./#losses)
* [Activations](./#activations)
* [Initializers](./#initializers)
* [Regularizers](./#regularizers)
* [Constraints](./#constraints)
* [Metrics](./#metrics)
* [Optimizers](./#optimizers)

## [Layers](./)

Mapping keras to DL4J layers is done in the [layers](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers) sub-module of model import. The structure of this project loosely reflects the structure of Keras.

### [Core Layers](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core)

* ✅ [Dense](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasDense.java)
* ✅ [Activation](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasActivation.java)
* ✅ [Dropout](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasDropout.java)
* ✅ [Flatten](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasFlatten.java)
* ✅ [Reshape](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasReshape.java)
* ✅ [Merge](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasMerge.java)
* ✅ [Permute](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasPermute.java)
* ✅ [RepeatVector](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasRepeatVector.java)
* ✅ [Lambda](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasLambda.java)
* ❌ ActivityRegularization
* ✅ [Masking](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasMasking.java)
* ✅ [SpatialDropout1D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasSpatialDropout.java)
* ✅ [SpatialDropout2D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasSpatialDropout.java)
* ✅ [SpatialDropout3D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasSpatialDropout.java)

### [Convolutional Layers](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional)

* ✅ [Conv1D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasConvolution1D.java)
* ✅ [Conv2D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasConvolution2D.java)
* ✅ [Conv3D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasConvolution3D.java)
* ✅ [AtrousConvolution1D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasAtrousConvolution1D.java)
* ✅ [AtrousConvolution2D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasAtrousConvolution1D.java)
* ❌ SeparableConv1D
* ✅ [SeparableConv2D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasSeparableConvolution2D.java)
* ✅ [Conv2DTranspose](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasDeconvolution2D.java)
* ❌ Conv3DTranspose
* ✅ [Cropping1D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasCropping1D.java)
* ✅ [Cropping2D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasCropping2D.java)
* ✅ [Cropping3D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasCropping3D.java)
* ✅ [UpSampling1D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasUpsampling1D.java)
* ✅ [UpSampling2D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasUpsampling2D.java)
* ✅ [UpSampling3D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasUpsampling2D.java)
* ✅ [ZeroPadding1D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasZeroPadding1D.java)
* ✅ [ZeroPadding2D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasZeroPadding2D.java)
* ✅ [ZeroPadding3D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/convolutional/KerasZeroPadding3D.java)

### [Pooling Layers](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling)

* ✅ [MaxPooling1D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPooling1D.java)
* ✅ [MaxPooling2D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPooling2D.java)
* ✅ [MaxPooling3D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPooling3D.java)
* ✅ [AveragePooling1D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPooling1D.java)
* ✅ [AveragePooling2D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPooling2D.java)
* ✅ [AveragePooling3D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasPooling3D.java)
* ✅ [GlobalMaxPooling1D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasGlobalPooling.java)
* ✅ [GlobalMaxPooling2D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasGlobalPooling.java)
* ✅ [GlobalMaxPooling3D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasGlobalPooling.java)
* ✅ [GlobalAveragePooling1D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasGlobalPooling.java)
* ✅ [GlobalAveragePooling2D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasGlobalPooling.java)
* ✅ [GlobalAveragePooling3D](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/pooling/KerasGlobalPooling.java)

### [Locally-connected Layers](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/local)

* ✅ [LocallyConnected1D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/local/KerasLocallyConnected1D.java)
* ✅ [LocallyConnected2D](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/local/KerasLocallyConnected2D.java)

### [Recurrent Layers](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/recurrent)

* ✅ [SimpleRNN](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/recurrent/KerasSimpleRnn.java)
* ❌ GRU
* ✅ [LSTM](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/recurrent/KerasLSTM.java)
* ❌ ConvLSTM2D

### [Embedding Layers](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/embeddings)

* ✅ [Embedding](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/embeddings/KerasEmbedding.java)

### [Merge Layers](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/core/KerasMerge.java)

* ✅ Add / add
* ✅ Multiply / multiply
* ✅ Subtract / subtract
* ✅ Average / average
* ✅ Maximum / maximum
* ✅ Concatenate / concatenate
* ❌ Dot / dot

### [Advanced Activation Layers](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/advanced/activations)

* ✅ [LeakyReLU](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/advanced/activations/KerasLeakyReLU.java)
* ✅ [PReLU](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/advanced/activations/KerasPReLU.java)
* ✅ ELU
* ✅ [ThresholdedReLU](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/advanced/activations/KerasThresholdedReLU.java)

### [Normalization Layers](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/normalization)

* ✅ [BatchNormalization](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/normalization/KerasBatchNormalization.java)

### Noise Layers

* ✅ [GaussianNoise](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/noise/KerasGaussianNoise.java)
* ✅ [GaussianDropout](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/noise/KerasGaussianDropout.java)
* ✅ [AlphaDropout](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/noise/KerasAlphaDropout.java)

### Layer Wrappers

* ❌ TimeDistributed
* ✅ [Bidirectional](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/wrappers/KerasBidirectional.java)

## [Losses](./)

* ✅ mean\_squared\_error
* ✅ mean\_absolute\_error
* ✅ mean\_absolute\_percentage\_error
* ✅ mean\_squared\_logarithmic\_error
* ✅ squared\_hinge
* ✅ hinge
* ✅ categorical\_hinge
* ❌ logcosh
* ✅ categorical\_crossentropy
* ✅ sparse\_categorical\_crossentropy
* ✅ binary\_crossentropy
* ✅ kullback\_leibler\_divergence
* ✅ poisson
* ✅ cosine\_proximity

## [Activations](./)

* ✅ softmax
* ✅ elu
* ✅ selu
* ✅ softplus
* ✅ softsign
* ✅ relu
* ✅ tanh
* ✅ sigmoid
* ✅ hard\_sigmoid
* ✅ linear

## [Initializers](./)

* ✅ Zeros
* ✅ Ones
* ✅ Constant
* ✅ RandomNormal
* ✅ RandomUniform
* ✅ TruncatedNormal
* ✅ VarianceScaling
* ✅ Orthogonal
* ✅ Identity
* ✅ lecun\_uniform
* ✅ lecun\_normal
* ✅ glorot\_normal
* ✅ glorot\_uniform
* ✅ he\_normal
* ✅ he\_uniform

## [Regularizers](./)

* ✅ l1
* ✅ l2
* ✅ l1\_l2

## [Constraints](./)

* ✅ max\_norm
* ✅ non\_neg
* ✅ unit\_norm
* ✅ min\_max\_norm

## [Optimizers](./)

* ✅ SGD
* ✅ RMSprop
* ✅ Adagrad
* ✅ Adadelta
* ✅ Adam
* ✅ Adamax
* ✅ Nadam
* ❌ TFOptimizer


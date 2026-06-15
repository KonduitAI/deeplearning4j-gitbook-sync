---
title: "Keras Supported Features"
description: "Full support matrix for Keras model import — layers, activations, losses, and optimizers"
---

## Keras Model Import: Supported Features

This page provides the complete support matrix for Keras model import into DL4J. All mapping is implemented in the [deeplearning4j-modelimport](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras) module.

- Supported
- Not supported

---

## Layers

Mapping of Keras layers to DL4J is implemented in the [layers](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers) sub-module.

### Core Layers

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| Dense | DenseLayer | Yes |
| Activation | ActivationLayer | Yes |
| Dropout | DropoutLayer | Yes |
| Flatten | CnnToFeedForwardPreProcessor / RnnToFeedForwardPreProcessor | Yes |
| Reshape | Reshape (via input preprocessor) | Yes |
| Merge | MergeVertex | Yes |
| Permute | PermutePreProcessor | Yes |
| RepeatVector | RepeatVector | Yes |
| Lambda | SameDiffLambda | Yes |
| ActivityRegularization | — | No |
| Masking | MaskZeroLayer | Yes |
| SpatialDropout1D | DropoutLayer (spatial) | Yes |
| SpatialDropout2D | DropoutLayer (spatial) | Yes |
| SpatialDropout3D | DropoutLayer (spatial) | Yes |

### Convolutional Layers

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| Conv1D | Convolution1DLayer | Yes |
| Conv2D | ConvolutionLayer | Yes |
| Conv3D | ConvolutionLayer (3D) | Yes |
| AtrousConvolution1D | Convolution1DLayer (with dilation) | Yes |
| AtrousConvolution2D | ConvolutionLayer (with dilation) | Yes |
| SeparableConv1D | — | No |
| SeparableConv2D | SeparableConvolution2D | Yes |
| DepthwiseConv2D | DepthwiseConvolution2D | Yes |
| Conv2DTranspose | Deconvolution2D | Yes |
| Conv3DTranspose | — | No |
| Cropping1D | Cropping1D | Yes |
| Cropping2D | Cropping2D | Yes |
| Cropping3D | Cropping3D | Yes |
| UpSampling1D | Upsampling1D | Yes |
| UpSampling2D | Upsampling2D | Yes |
| UpSampling3D | Upsampling3D | Yes |
| ZeroPadding1D | ZeroPadding1DLayer | Yes |
| ZeroPadding2D | ZeroPaddingLayer | Yes |
| ZeroPadding3D | ZeroPadding3DLayer | Yes |

### Pooling Layers

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| MaxPooling1D | Subsampling1DLayer (MAX) | Yes |
| MaxPooling2D | SubsamplingLayer (MAX) | Yes |
| MaxPooling3D | Subsampling3DLayer (MAX) | Yes |
| AveragePooling1D | Subsampling1DLayer (AVG) | Yes |
| AveragePooling2D | SubsamplingLayer (AVG) | Yes |
| AveragePooling3D | Subsampling3DLayer (AVG) | Yes |
| GlobalMaxPooling1D | GlobalPoolingLayer (MAX) | Yes |
| GlobalMaxPooling2D | GlobalPoolingLayer (MAX) | Yes |
| GlobalMaxPooling3D | GlobalPoolingLayer (MAX) | Yes |
| GlobalAveragePooling1D | GlobalPoolingLayer (AVG) | Yes |
| GlobalAveragePooling2D | GlobalPoolingLayer (AVG) | Yes |
| GlobalAveragePooling3D | GlobalPoolingLayer (AVG) | Yes |

### Locally-Connected Layers

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| LocallyConnected1D | LocallyConnected1D | Yes |
| LocallyConnected2D | LocallyConnected2D | Yes |

### Recurrent Layers

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| SimpleRNN | SimpleRnn | Yes |
| GRU | — | No |
| LSTM | LSTM | Yes |
| ConvLSTM2D | — | No |

### Embedding Layers

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| Embedding | EmbeddingSequenceLayer | Yes |

### Merge Layers

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| Add / add | MergeVertex (add) | Yes |
| Multiply / multiply | MergeVertex (mul) | Yes |
| Subtract / subtract | MergeVertex (sub) | Yes |
| Average / average | MergeVertex (avg) | Yes |
| Maximum / maximum | MergeVertex (max) | Yes |
| Concatenate / concatenate | MergeVertex (concat) | Yes |
| Dot / dot | — | No |

### Advanced Activation Layers

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| LeakyReLU | ActivationLayer (LeakyReLU) | Yes |
| PReLU | PReLULayer | Yes |
| ELU | ActivationLayer (ELU) | Yes |
| ThresholdedReLU | ActivationLayer (ThresholdedReLU) | Yes |

### Normalization Layers

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| BatchNormalization | BatchNormalization | Yes |

### Noise Layers

| Keras Layer | DL4J Equivalent | Supported |
|---|---|---|
| GaussianNoise | DropoutLayer (GaussianNoise) | Yes |
| GaussianDropout | DropoutLayer (GaussianDropout) | Yes |
| AlphaDropout | DropoutLayer (AlphaDropout) | Yes |

### Layer Wrappers

| Keras Wrapper | DL4J Equivalent | Supported |
|---|---|---|
| TimeDistributed | — | No |
| Bidirectional | Bidirectional | Yes |

---

## Losses

[Source: KerasLossUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasLossUtils.java)

| Keras Loss | DL4J Equivalent | Supported |
|---|---|---|
| mean_squared_error | MSE | Yes |
| mean_absolute_error | MAE | Yes |
| mean_absolute_percentage_error | MAPE | Yes |
| mean_squared_logarithmic_error | MSLE | Yes |
| squared_hinge | SquaredHinge | Yes |
| hinge | Hinge | Yes |
| categorical_hinge | CategoricalHinge | Yes |
| logcosh | — | No |
| categorical_crossentropy | CategoricalCrossEntropy | Yes |
| sparse_categorical_crossentropy | SparseMCXENT | Yes |
| binary_crossentropy | BinaryCrossEntropy | Yes |
| kullback_leibler_divergence | KullbackLeiblerDivergence | Yes |
| poisson | Poisson | Yes |
| cosine_proximity | CosineSimilarity | Yes |

---

## Activations

[Source: KerasActivationUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasActivationUtils.java)

All standard Keras activations are supported:

| Keras Activation | DL4J Equivalent | Supported |
|---|---|---|
| softmax | Softmax | Yes |
| elu | ELU | Yes |
| selu | SELU | Yes |
| softplus | Softplus | Yes |
| softsign | Softsign | Yes |
| relu | ReLU | Yes |
| tanh | Tanh | Yes |
| sigmoid | Sigmoid | Yes |
| hard_sigmoid | HardSigmoid | Yes |
| linear | Identity | Yes |

---

## Initializers

[Source: KerasInitilizationUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasInitilizationUtils.java)

All standard Keras weight initializers are supported:

| Keras Initializer | DL4J Equivalent | Supported |
|---|---|---|
| Zeros | ZeroInitScheme | Yes |
| Ones | OneInitScheme | Yes |
| Constant | ConstantDistribution | Yes |
| RandomNormal | NormalDistribution | Yes |
| RandomUniform | UniformDistribution | Yes |
| TruncatedNormal | TruncatedNormalDistribution | Yes |
| VarianceScaling | VarianceScalingInitScheme | Yes |
| Orthogonal | OrthogonalInitScheme | Yes |
| Identity | IdentityInitScheme | Yes |
| lecun_uniform | WeightInitLecunUniform | Yes |
| lecun_normal | WeightInitLecunNormal | Yes |
| glorot_normal | GlorotNormal (Xavier) | Yes |
| glorot_uniform | GlorotUniform (Xavier) | Yes |
| he_normal | HeNormal | Yes |
| he_uniform | HeUniform | Yes |

---

## Regularizers

[Source: KerasRegularizerUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasRegularizerUtils.java)

All standard Keras regularizers are supported:

| Keras Regularizer | DL4J Equivalent | Supported |
|---|---|---|
| l1 | L1Regularization | Yes |
| l2 | L2Regularization | Yes |
| l1_l2 | L1L2Regularization | Yes |

---

## Constraints

[Source: KerasConstraintUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasConstraintUtils.java)

All standard Keras constraints are supported:

| Keras Constraint | DL4J Equivalent | Supported |
|---|---|---|
| max_norm | MaxNormConstraint | Yes |
| non_neg | NonNegativeConstraint | Yes |
| unit_norm | UnitNormConstraint | Yes |
| min_max_norm | MinMaxNormConstraint | Yes |

---

## Optimizers

[Source: KerasOptimizerUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasOptimizerUtils.java)

All standard Keras optimizers are supported. TFOptimizer (a TensorFlow-specific optimizer wrapper) is not:

| Keras Optimizer | DL4J Equivalent | Supported |
|---|---|---|
| SGD | Sgd | Yes |
| RMSprop | RmsProp | Yes |
| Adagrad | AdaGrad | Yes |
| Adadelta | AdaDelta | Yes |
| Adam | Adam | Yes |
| Adamax | AdaMax | Yes |
| Nadam | Nadam | Yes |
| TFOptimizer | — | No |

---

## Notes on Partial Support

- **GRU**: not currently supported. The Keras GRU has a slightly different recurrent formulation from DL4J's. Consider replacing with LSTM for import, or implement a custom layer mapper.
- **TimeDistributed wrapper**: not supported. In many cases, you can restructure your model to achieve the same effect with 1D convolutions or RNNs directly.
- **ConvLSTM2D**: not supported. No equivalent exists in DL4J core at this time.
- **Dot merge layer**: the dot product merge mode is not supported. Concatenate followed by a Dense layer is a workable alternative in many cases.
- **logcosh loss**: not implemented. `mean_squared_error` is a reasonable substitute for smooth losses.
- **ActivityRegularization**: this layer type is not supported. Apply regularization directly in the layer configuration instead.
- **Custom TensorFlow optimizers (TFOptimizer)**: cannot be imported. Use a standard Keras optimizer.

---

## Keras Version Compatibility

DL4J model import supports both Keras 1.x and Keras 2.x. The importer detects the Keras version from the HDF5 metadata and adjusts config key names accordingly. Both TensorFlow and Theano backends produce compatible HDF5 files.

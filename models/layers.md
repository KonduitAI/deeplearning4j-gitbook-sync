---
title: Supported Layers
short_title: Layers
description: Supported neural network layers.
category: Models
weight: 3
---

# Layers

## What are layers?

Each layer in a neural network configuration represents a unit of hidden units. When layers are stacked together, they represent a _deep neural network_.

## Using layers

All layers available in Eclipse Deeplearning4j can be used either in a `MultiLayerNetwork` or `ComputationGraph`. When configuring a neural network, you pass the layer configuration and the network will instantiate the layer for you.

## Layers vs. vertices

If you are configuring complex networks such as InceptionV4, you will need to use the `ComputationGraph` API and join different branches together using vertices. Check the vertices for more information.

## General layers

### ActivationLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/ActivationLayer.java)

Activation layer is a simple layer that applies the specified activation function to the input activations

**clone**

```text
public ActivationLayer clone()
```

* param activation Activation function for the layer

**activation**

```text
public Builder activation(String activationFunction)
```

Activation function for the layer

**activation**

```text
public Builder activation(IActivation activationFunction)
```

* param activationFunction Activation function for the layer

**activation**

```text
public Builder activation(Activation activation)
```

* param activation Activation function for the layer

### DenseLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/DenseLayer.java)

Dense layer: a standard fully connected feed forward layer

**hasBias**

```text
public Builder hasBias(boolean hasBias)
```

If true \(default\): include bias parameters in the model. False: no bias.

**hasLayerNorm**

```text
public Builder hasLayerNorm(boolean hasLayerNorm)
```

If true \(default = false\): enable layer normalization on this layer

### DropoutLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/DropoutLayer.java)

Dropout layer. This layer simply applies dropout at training time, and passes activations through unmodified at test

**build**

```text
public DropoutLayer build()
```

Create a dropout layer with standard {- link Dropout}, with the specified probability of retaining the input activation. See {- link Dropout} for the full details

* param dropout Activation retain probability.

### EmbeddingLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/EmbeddingLayer.java)

Embedding layer: feed-forward layer that expects single integers per example as input \(class numbers, in range 0 to the equivalent one-hot representation. Mathematically, EmbeddingLayer is equivalent to using a DenseLayer with a one-hot representation for the input; however, it can be much more efficient with a large number of classes \(as a dense layer + one-hot input does a matrix multiply with all but one value being zero\).  
**Note**: can only be used as the first layer for a network  
**Note 2**: For a given example index i, the output is activationFunction\(weights.getRow\(i\) + bias\), hence the weight rows can be considered a vector/embedding for each example.  
Note also that embedding layer has an activation function \(set to IDENTITY to disable\) and optional bias \(which is disabled by default\)

**hasBias**

```text
public Builder hasBias(boolean hasBias)
```

If true: include bias parameters in the layer. False \(default\): no bias.

**weightInit**

```text
public Builder weightInit(EmbeddingInitializer embeddingInitializer)
```

Initialize the embedding layer using the specified EmbeddingInitializer - such as a Word2Vec instance

* param embeddingInitializer Source of the embedding layer weights

**weightInit**

```text
public Builder weightInit(INDArray vectors)
```

Initialize the embedding layer using values from the specified array. Note that the array should have shape \[vocabSize, vectorSize\]. After copying values from the array to initialize the network parameters, the input array will be discarded \(so that, if necessary, it can be garbage collected\)

* param vectors Vectors to initialize the embedding layer with

### EmbeddingSequenceLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/EmbeddingSequenceLayer.java)

Embedding layer for sequences: feed-forward layer that expects fixed-length number \(inputLength\) of integers/indices per example as input, ranged from 0 to numClasses - 1. This input thus has shape \[numExamples, inputLength\] or shape \[numExamples, 1, inputLength\].  
The output of this layer is 3D \(sequence/time series\), namely of shape \[numExamples, nOut, inputLength\]. **Note**: can only be used as the first layer for a network  
**Note 2**: For a given example index i, the output is activationFunction\(weights.getRow\(i\) + bias\), hence the weight rows can be considered a vector/embedding of each index.  
Note also that embedding layer has an activation function \(set to IDENTITY to disable\) and optional bias \(which is disabled by default\)

**hasBias**

```text
public Builder hasBias(boolean hasBias)
```

If true: include bias parameters in the layer. False \(default\): no bias.

**inputLength**

```text
public Builder inputLength(int inputLength)
```

Set input sequence length for this embedding layer.

* param inputLength input sequence length
* return Builder

**inferInputLength**

```text
public Builder inferInputLength(boolean inferInputLength)
```

Set input sequence inference mode for embedding layer.

* param inferInputLength whether to infer input length
* return Builder

**weightInit**

```text
public Builder weightInit(EmbeddingInitializer embeddingInitializer)
```

Initialize the embedding layer using the specified EmbeddingInitializer - such as a Word2Vec instance

* param embeddingInitializer Source of the embedding layer weights

**weightInit**

```text
public Builder weightInit(INDArray vectors)
```

Initialize the embedding layer using values from the specified array. Note that the array should have shape \[vocabSize, vectorSize\]. After copying values from the array to initialize the network parameters, the input array will be discarded \(so that, if necessary, it can be garbage collected\)

* param vectors Vectors to initialize the embedding layer with

### GlobalPoolingLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/GlobalPoolingLayer.java)

Global pooling layer - used to do pooling over time for RNNs, and 2d pooling for CNNs.  
Supports the following

Global pooling layer can also handle mask arrays when dealing with variable length inputs. Mask arrays are assumed to be 2d, and are fed forward through the network during training or post-training forward pass:

* Time series: mask arrays are shape \[miniBatchSize, maxTimeSeriesLength\] and contain values 0 or 1 only  
* CNNs: mask have shape \[miniBatchSize, height\] or \[miniBatchSize, width\]. Important: the current implementation assumes that for CNNs + variable length \(masking\), the input shape is \[miniBatchSize, channels, height, 1\] or \[miniBatchSize, channels, 1, width\] respectively. This is the case with global pooling in architectures like CNN for sentence classification.  

Behaviour with default settings:

* 3d \(time series\) input with shape \[miniBatchSize, vectorSize, timeSeriesLength\] -&gt; 2d output \[miniBatchSize, vectorSize\]  
* 4d \(CNN\) input with shape \[miniBatchSize, channels, height, width\] -&gt; 2d output \[miniBatchSize, channels\]  
* 5d \(CNN3D\) input with shape \[miniBatchSize, channels, depth, height, width\] -&gt; 2d output \[miniBatchSize, channels\]  

Alternatively, by setting collapseDimensions = false in the configuration, it is possible to retain the reduced dimensions as 1s: this gives

* \[miniBatchSize, vectorSize, 1\] for RNN output,  
* \[miniBatchSize, channels, 1, 1\] for CNN output, and  
* \[miniBatchSize, channels, 1, 1, 1\] for CNN3D output.  

**poolingDimensions**

```text
public Builder poolingDimensions(int... poolingDimensions)
```

Pooling type for global pooling

**poolingType**

```text
public Builder poolingType(PoolingType poolingType)
```

* param poolingType Pooling type for global pooling

**collapseDimensions**

```text
public Builder collapseDimensions(boolean collapseDimensions)
```

Whether to collapse dimensions when pooling or not. Usually you do want to do this. Default: true. If true:

* 3d \(time series\) input with shape \[miniBatchSize, vectorSize, timeSeriesLength\] -&gt; 2d output \[miniBatchSize, vectorSize\]  
* 4d \(CNN\) input with shape \[miniBatchSize, channels, height, width\] -&gt; 2d output \[miniBatchSize, channels\]  
* 5d \(CNN3D\) input with shape \[miniBatchSize, channels, depth, height, width\] -&gt; 2d output \[miniBatchSize, channels\]  

If false:

* 3d \(time series\) input with shape \[miniBatchSize, vectorSize, timeSeriesLength\] -&gt; 3d output \[miniBatchSize, vectorSize, 1\]  
* 4d \(CNN\) input with shape \[miniBatchSize, channels, height, width\] -&gt; 2d output \[miniBatchSize, channels, 1, 1\]  
* 5d \(CNN3D\) input with shape \[miniBatchSize, channels, depth, height, width\] -&gt; 2d output \[miniBatchSize, channels, 1, 1, 1\]  
* param collapseDimensions Whether to collapse the dimensions or not

**pnorm**

```text
public Builder pnorm(int pnorm)
```

P-norm constant. Only used if using {- link PoolingType\#PNORM} for the pooling type

* param pnorm P-norm constant

#### LocalResponseNormalization <a id="localresponsenormalization"></a>

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LocalResponseNormalization.java)

Local response normalization layer  
See section 3.3 of [http://www.cs.toronto.edu/~fritz/absps/imagenet.pdf](http://www.cs.toronto.edu/~fritz/absps/imagenet.pdf)

**k**

```text
public Builder k(double k)
```

LRN scaling constant k. Default: 2

**n**

```text
public Builder n(double n)
```

Number of adjacent kernel maps to use when doing LRN. default: 5

* param n Number of adjacent kernel maps

**alpha**

```text
public Builder alpha(double alpha)
```

LRN scaling constant alpha. Default: 1e-4

* param alpha Scaling constant

**beta**

```text
public Builder beta(double beta)
```

Scaling constant beta. Default: 0.75

* param beta Scaling constant

**cudnnAllowFallback**

```text
public Builder cudnnAllowFallback(boolean allowFallback)
```

When using CuDNN and an error is encountered, should fallback to the non-CuDNN implementatation be allowed? If set to false, an exception in CuDNN will be propagated back to the user. If false, the built-in \(non-CuDNN\) implementation for BatchNormalization will be used

* param allowFallback Whether fallback to non-CuDNN implementation should be used

### LocallyConnected1D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LocallyConnected1D.java)

SameDiff version of a 1D locally connected layer.

**nIn**

```text
public Builder nIn(int nIn)
```

Number of inputs to the layer \(input size\)

**nOut**

```text
public Builder nOut(int nOut)
```

* param nOut Number of outputs \(output size\)

**activation**

```text
public Builder activation(Activation activation)
```

* param activation Activation function for the layer

**kernelSize**

```text
public Builder kernelSize(int k)
```

* param k Kernel size for the layer

**stride**

```text
public Builder stride(int s)
```

* param s Stride for the layer

**padding**

```text
public Builder padding(int p)
```

* param p Padding for the layer. Not used if {- link ConvolutionMode\#Same} is set

**convolutionMode**

```text
public Builder convolutionMode(ConvolutionMode cm)
```

* param cm Convolution mode for the layer. See {- link ConvolutionMode} for details

**dilation**

```text
public Builder dilation(int d)
```

* param d Dilation for the layer

**hasBias**

```text
public Builder hasBias(boolean hasBias)
```

* param hasBias If true \(default is false\) the layer will have a bias

**setInputSize**

```text
public Builder setInputSize(int inputSize)
```

Set input filter size for this locally connected 1D layer

* param inputSize height of the input filters
* return Builder

### LocallyConnected2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LocallyConnected2D.java)

SameDiff version of a 2D locally connected layer.

**setKernel**

```text
public void setKernel(int... kernel)
```

Number of inputs to the layer \(input size\)

**setStride**

```text
public void setStride(int... stride)
```

* param stride Stride for the layer. Must be 2 values \(height/width\)

**setPadding**

```text
public void setPadding(int... padding)
```

* param padding Padding for the layer. Not used if {- link ConvolutionMode\#Same} is set. Must be 2 values \(height/width\)

**setDilation**

```text
public void setDilation(int... dilation)
```

* param dilation Dilation for the layer. Must be 2 values \(height/width\)

**nIn**

```text
public Builder nIn(int nIn)
```

* param nIn Number of inputs to the layer \(input size\)

**nOut**

```text
public Builder nOut(int nOut)
```

* param nOut Number of outputs \(output size\)

**activation**

```text
public Builder activation(Activation activation)
```

* param activation Activation function for the layer

**kernelSize**

```text
public Builder kernelSize(int... k)
```

* param k Kernel size for the layer. Must be 2 values \(height/width\)

**stride**

```text
public Builder stride(int... s)
```

* param s Stride for the layer. Must be 2 values \(height/width\)

**padding**

```text
public Builder padding(int... p)
```

* param p Padding for the layer. Not used if {- link ConvolutionMode\#Same} is set. Must be 2 values \(height/width\)

**convolutionMode**

```text
public Builder convolutionMode(ConvolutionMode cm)
```

* param cm Convolution mode for the layer. See {- link ConvolutionMode} for details

**dilation**

```text
public Builder dilation(int... d)
```

* param d Dilation for the layer. Must be 2 values \(height/width\)

**hasBias**

```text
public Builder hasBias(boolean hasBias)
```

* param hasBias If true \(default is false\) the layer will have a bias

**setInputSize**

```text
public Builder setInputSize(int... inputSize)
```

Set input filter size \(h,w\) for this locally connected 2D layer

* param inputSize pair of height and width of the input filters to this layer
* return Builder

### LossLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/LossLayer.java)

LossLayer is a flexible output layer that performs a loss function on an input without MLP logic.  
LossLayer is does not have any parameters. Consequently, setting nIn/nOut isn’t supported - the output size is the same size as the input activations.

**nIn**

```text
public Builder nIn(int nIn)
```

* param lossFunction Loss function for the loss layer

### OutputLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/OutputLayer.java)

Output layer used for training via backpropagation based on labels and a specified loss function. Can be configured for both classification and regression. Note that OutputLayer has parameters - it contains a fully-connected layer \(effectively contains a DenseLayer\) internally. This allows the output size to be different to the layer input size.

**build**

```text
public OutputLayer build()
```

* param lossFunction Loss function for the output layer

### Pooling1D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Pooling1D.java)

Supports the following pooling types: MAX, AVG, SUM, PNORM, NONE

### Pooling2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Pooling2D.java)

Supports the following pooling types: MAX, AVG, SUM, PNORM, NONE

### Subsampling1DLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Subsampling1DLayer.java)

sequenceLength\]}. This layer accepts RNN InputTypes instead of CNN InputTypes.

Supports the following pooling types: MAX, AVG, SUM, PNORM

**setKernelSize**

```text
public void setKernelSize(int... kernelSize)
```

Kernel size

* param kernelSize kernel size

**setStride**

```text
public void setStride(int... stride)
```

Stride

* param stride stride value

**setPadding**

```text
public void setPadding(int... padding)
```

Padding

* param padding padding value

### Upsampling1D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Upsampling1D.java)

sequenceLength\]}  
Example:

```text
If input (for a single example, with channels down page, and sequence from left to right) is:
[ A1, A2, A3]
[ B1, B2, B3]
Then output with size = 2 is:
[ A1, A1, A2, A2, A3, A3]
[ B1, B1, B2, B2, B3, B2]
```

**size**

```text
public Builder size(int size)
```

Upsampling size

* param size upsampling size in single spatial dimension of this 1D layer

**size**

```text
public Builder size(int[] size)
```

Upsampling size int array with a single element. Array must be length 1

* param size upsampling size in single spatial dimension of this 1D layer

### Upsampling2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Upsampling2D.java)

Upsampling 2D layer  
Repeats each value \(or rather, set of depth values\) in the height and width dimensions by

```text
Input (slice for one example and channel)
[ A, B ]
[ C, D ]
Size = [2, 2]
Output (slice for one example and channel)
[ A, A, B, B ]
[ A, A, B, B ]
[ C, C, D, D ]
[ C, C, D, D ]
```

**size**

```text
public Builder size(int size)
```

Upsampling size int, used for both height and width

* param size upsampling size in height and width dimensions

**size**

```text
public Builder size(int[] size)
```

Upsampling size array

* param size upsampling size in height and width dimensions

### Upsampling3D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Upsampling3D.java)

Upsampling 3D layer  
Repeats each value \(all channel values for each x/y/z location\) by size\[0\], size\[1\] and \[minibatch, channels, size\[0\] depth, size\[1\] height, size\[2\] width\]}

**size**

```text
public Builder size(int size)
```

Upsampling size as int, so same upsampling size is used for depth, width and height

* param size upsampling size in height, width and depth dimensions

**size**

```text
public Builder size(int[] size)
```

Upsampling size as int, so same upsampling size is used for depth, width and height

* param size upsampling size in height, width and depth dimensions

### ZeroPadding1DLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/ZeroPadding1DLayer.java)

Zero padding 1D layer for convolutional neural networks. Allows padding to be done separately for top and bottom.

**setPadding**

```text
public void setPadding(int... padding)
```

Padding value for left and right. Must be length 2 array

**build**

```text
public ZeroPadding1DLayer build()
```

* param padding Padding for both the left and right

### ZeroPadding3DLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/ZeroPadding3DLayer.java)

Zero padding 3D layer for convolutional neural networks. Allows padding to be done separately for “left” and “right” in all three spatial dimensions.

**setPadding**

```text
public void setPadding(int... padding)
```

\[padLeftD, padRightD, padLeftH, padRightH, padLeftW, padRightW\]

**build**

```text
public ZeroPadding3DLayer build()
```

* param padding Padding for both the left and right in all three spatial dimensions

### ZeroPaddingLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/ZeroPaddingLayer.java)

Zero padding layer for convolutional neural networks \(2D CNNs\). Allows padding to be done separately for top/bottom/left/right

**setPadding**

```text
public void setPadding(int... padding)
```

Padding value for top, bottom, left, and right. Must be length 4 array

**build**

```text
public ZeroPaddingLayer build()
```

* param padHeight Padding for both the top and bottom
* param padWidth Padding for both the left and right

### ElementWiseMultiplicationLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/misc/ElementWiseMultiplicationLayer.java)

is a learnable weight vector of length nOut

* “.” is element-wise multiplication  
* b is a bias vector  

Note that the input and output sizes of the element-wise layer are the same for this layer

created by jingshu

**getMemoryReport**

```text
public LayerMemoryReport getMemoryReport(InputType inputType)
```

This is a report of the estimated memory consumption for the given layer

* param inputType Input type to the layer. Memory consumption is often a function of the input type
* return Memory report for the layer

### RepeatVector

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/misc/RepeatVector.java)

RepeatVector layer configuration.

RepeatVector takes a mini-batch of vectors of shape \(mb, length\) and a repeat factor n and outputs a 3D tensor of shape \(mb, n, length\) in which x is repeated n times.

**getRepetitionFactor**

```text
public int getRepetitionFactor()
```

Set repetition factor for RepeatVector layer

**setRepetitionFactor**

```text
public void setRepetitionFactor(int n)
```

Set repetition factor for RepeatVector layer

* param n upsampling size in height and width dimensions

**repetitionFactor**

```text
public Builder repetitionFactor(int n)
```

Set repetition factor for RepeatVector layer

* param n upsampling size in height and width dimensions

### Yolo2OutputLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/objdetect/Yolo2OutputLayer.java)

Output \(loss\) layer for YOLOv2 object detection model, based on the papers: YOLO9000: Better, Faster, Stronger - Redmon & Farhadi \(2016\) - [https://arxiv.org/abs/1612.08242](https://arxiv.org/abs/1612.08242)  
and  
You Only Look Once: Unified, Real-Time Object Detection - Redmon et al. \(2016\) - [http://www.cv-foundation.org/openaccess/content\_cvpr\_2016/papers/Redmon\_You\_Only\_Look\_CVPR\_2016\_paper.pdf](http://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Redmon_You_Only_Look_CVPR_2016_paper.pdf)  
This loss function implementation is based on the YOLOv2 version of the paper. However, note that it doesn’t currently support simultaneous training on both detection and classification datasets as described in the YOlO9000 paper.

Note: Input activations to the Yolo2OutputLayer should have shape: \[minibatch, b\(5+c\), H, W\], where:  
b = number of bounding boxes \(determined by config - see papers for details\)  
c = number of classes  
H = output/label height  
W = output/label width

Important: In practice, this means that the last convolutional layer before your Yolo2OutputLayer should have output depth of b\(5+c\). Thus if you change the number of bounding boxes, or change the number of object classes, the number of channels \(nOut of the last convolution layer\) needs to also change.  
Label format: \[minibatch, 4+C, H, W\]  
Order for labels depth: \[x1,y1,x2,y2,\(class labels\)\]  
x1 = box top left position  
y1 = as above, y axis  
x2 = box bottom right position  
y2 = as above y axis  
Note: labels are represented as a multiple of grid size - for a 13x13 grid, \(0,0\) is top left, \(13,13\) is bottom right  
Note also that mask arrays are not required - this implementation infers the presence or absence of objects in each grid cell from the class labels \(which should be 1-hot if an object is present, or all 0s otherwise\).

**lambdaCoord**

```text
public Builder lambdaCoord(double lambdaCoord)
```

Loss function coefficient for position and size/scale components of the loss function. Default \(as per paper\): 5

**lambbaNoObj**

```text
public Builder lambbaNoObj(double lambdaNoObj)
```

Loss function coefficient for the “no object confidence” components of the loss function. Default \(as per paper\): 0.5

* param lambdaNoObj Lambda value for no-object \(confidence\) component of the loss function

**lossPositionScale**

```text
public Builder lossPositionScale(ILossFunction lossPositionScale)
```

Loss function for position/scale component of the loss function

* param lossPositionScale Loss function for position/scale

**lossClassPredictions**

```text
public Builder lossClassPredictions(ILossFunction lossClassPredictions)
```

Loss function for the class predictions - defaults to L2 loss \(i.e., sum of squared errors, as per the paper\), however Loss MCXENT could also be used \(which is more common for classification\).

* param lossClassPredictions Loss function for the class prediction error component of the YOLO loss function

**boundingBoxPriors**

```text
public Builder boundingBoxPriors(INDArray boundingBoxes)
```

Bounding box priors dimensions \[width, height\]. For N bounding boxes, input has shape \[rows, columns\] = \[N, 2\] Note that dimensions should be specified as fraction of grid size. For example, a network with 13x13 output, a value of 1.0 would correspond to one grid cell; a value of 13 would correspond to the entire image.

* param boundingBoxes Bounding box prior dimensions \(width, height\)

### MaskLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/util/MaskLayer.java)

MaskLayer applies the mask array to the forward pass activations, and backward pass gradients, passing through this layer. It can be used with 2d \(feed-forward\), 3d \(time series\) or 4d \(CNN\) activations.

### MaskZeroLayer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/util/MaskZeroLayer.java)

Wrapper which masks timesteps with activation equal to the specified masking value \(0.0 default\). Assumes that the input shape is \[batch\_size, input\_size, timesteps\].


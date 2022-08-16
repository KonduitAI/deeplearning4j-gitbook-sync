---
description: Also known as CNN.
---

# Convolutional Layers

## Available layers

### Convolution1D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Convolution1D.java)

1D convolution layer. Expects input activations of shape \[minibatch,channels,sequenceLength]

### Convolution2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Convolution2D.java)

2D convolution layer

### Convolution3D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Convolution3D.java)

3D convolution layer configuration

**hasBias**

```
public boolean hasBias()
```

An optional dataFormat: “NDHWC” or “NCDHW”. Defaults to “NCDHW”.\
The data format of the input and output data.\
For “NCDHW” (also known as ‘channels first’ format), the data storage order is: \[batchSize, inputChannels, inputDepth, inputHeight, inputWidth].\
For “NDHWC” (‘channels last’ format), the data is stored in the order of: \[batchSize, inputDepth, inputHeight, inputWidth, inputChannels].

**kernelSize**

```
public Builder kernelSize(int... kernelSize)
```

The data format for input and output activations.\
NCDHW: activations (in/out) should have shape \[minibatch, channels, depth, height, width]\
NDHWC: activations (in/out) should have shape \[minibatch, depth, height, width, channels]

**stride**

```
public Builder stride(int... stride)
```

Set stride size for 3D convolutions in (depth, height, width) order

* param stride kernel size
* return 3D convolution layer builder

**padding**

```
public Builder padding(int... padding)
```

Set padding size for 3D convolutions in (depth, height, width) order

* param padding kernel size
* return 3D convolution layer builder

**dilation**

```
public Builder dilation(int... dilation)
```

Set dilation size for 3D convolutions in (depth, height, width) order

* param dilation kernel size
* return 3D convolution layer builder

**dataFormat**

```
public Builder dataFormat(DataFormat dataFormat)
```

The data format for input and output activations.\
NCDHW: activations (in/out) should have shape \[minibatch, channels, depth, height, width]\
NDHWC: activations (in/out) should have shape \[minibatch, depth, height, width, channels]

* param dataFormat Data format to use for activations

**setKernelSize**

```
public void setKernelSize(int... kernelSize)
```

Set kernel size for 3D convolutions in (depth, height, width) order

* param kernelSize kernel size

**setStride**

```
public void setStride(int... stride)
```

Set stride size for 3D convolutions in (depth, height, width) order

* param stride kernel size

**setPadding**

```
public void setPadding(int... padding)
```

Set padding size for 3D convolutions in (depth, height, width) order

* param padding kernel size

**setDilation**

```
public void setDilation(int... dilation)
```

Set dilation size for 3D convolutions in (depth, height, width) order

* param dilation kernel size

### Deconvolution2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/Deconvolution2D.java)

2D deconvolution layer configuration

Deconvolutions are also known as transpose convolutions or fractionally strided convolutions. In essence, deconvolutions swap forward and backward pass with regular 2D convolutions.

See the paper by Matt Zeiler for details: [http://www.matthewzeiler.com/wp-content/uploads/2017/07/cvpr2010.pdf](http://www.matthewzeiler.com/wp-content/uploads/2017/07/cvpr2010.pdf)

For an intuitive guide to convolution arithmetic and shapes, see: [https://arxiv.org/abs/1603.07285v1](https://arxiv.org/abs/1603.07285v1)

**hasBias**

```
public boolean hasBias()
```

Deconvolution2D layer nIn in the input layer is the number of channels nOut is the number of filters to be used in the net or in other words the channels The builder specifies the filter/kernel size, the stride and padding The pooling layer takes the kernel size

**convolutionMode**

```
public Builder convolutionMode(ConvolutionMode convolutionMode)
```

Set the convolution mode for the Convolution layer. See {- link ConvolutionMode} for more details

* param convolutionMode Convolution mode for layer

**kernelSize**

```
public Builder kernelSize(int... kernelSize)
```

Size of the convolution rows/columns

* param kernelSize the height and width of the kernel

### Cropping1D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/convolutional/Cropping1D.java)

Cropping layer for convolutional (1d) neural networks. Allows cropping to be done separately for top/bottom

**getOutputType**

```
public InputType getOutputType(int layerIndex, InputType inputType)
```

* param cropTopBottom Amount of cropping to apply to both the top and the bottom of the input activations

**setCropping**

```
public void setCropping(int... cropping)
```

Cropping amount for top/bottom (in that order). Must be length 1 or 2 array.

**build**

```
public Cropping1D build()
```

* param cropping Cropping amount for top/bottom (in that order). Must be length 1 or 2 array.

### Cropping2D

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/convolutional/Cropping2D.java)

Cropping layer for convolutional (2d) neural networks. Allows cropping to be done separately for top/bottom/left/right

**getOutputType**

```
public InputType getOutputType(int layerIndex, InputType inputType)
```

* param cropTopBottom Amount of cropping to apply to both the top and the bottom of the input activations
* param cropLeftRight Amount of cropping to apply to both the left and the right of the input activations

**setCropping**

```
public void setCropping(int... cropping)
```

Cropping amount for top/bottom/left/right (in that order). A length 4 array.

**build**

```
public Cropping2D build()
```

* param cropping Cropping amount for top/bottom/left/right (in that order). Must be length 4 array.

#### Cropping3D <a href="#cropping3d" id="cropping3d"></a>

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/convolutional/Cropping3D.java)

Cropping layer for convolutional (3d) neural networks. Allows cropping to be done separately for upper and lower bounds of depth, height and width dimensions.

**getOutputType**

```
public InputType getOutputType(int layerIndex, InputType inputType)
```

* param cropDepth Amount of cropping to apply to both depth boundaries of the input activations
* param cropHeight Amount of cropping to apply to both height boundaries of the input activations
* param cropWidth Amount of cropping to apply to both width boundaries of the input activations

**setCropping**

```
public void setCropping(int... cropping)
```

Cropping amount, a length 6 array, i.e. crop left depth, crop right depth, crop left height, crop right height, crop left width, crop right width

**build**

```
public Cropping3D build()
```

* param cropping Cropping amount, must be length 3 or 6 array, i.e. either crop depth, crop height, crop width or crop left depth, crop right depth, crop left height, crop right height, crop left width, crop right width

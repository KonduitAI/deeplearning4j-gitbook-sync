---
title: CNN
short_title: CNN
description: 
category: Operations
weight: 40
---
# CNN Namespace
# Operation classes
## <a name="avgPooling2d">avgPooling2d</a>
```JAVA
INDArray avgPooling2d(INDArray input, Pooling2DConfig pooling2DConfig)

SDVariable avgPooling2d(SDVariable input, Pooling2DConfig pooling2DConfig)
SDVariable avgPooling2d(String name, SDVariable input, Pooling2DConfig pooling2DConfig)
```
2D Convolution layer operation - average pooling 2d

* **input** - the input to average pooling 2d operation - 4d CNN (image) activations in NCHW format
                        (shape [minibatch, channels, height, width]) or NHWC format (shape [minibatch, height, width, channels]) (NUMERIC type)
* **Pooling2DConfig** - [Pooling2DConfig]

## <a name="avgPooling3d">avgPooling3d</a>
```JAVA
INDArray avgPooling3d(INDArray input, Pooling3DConfig pooling3DConfig)

SDVariable avgPooling3d(SDVariable input, Pooling3DConfig pooling3DConfig)
SDVariable avgPooling3d(String name, SDVariable input, Pooling3DConfig pooling3DConfig)
```
3D convolution layer operation - average pooling 3d 

* **input** - the input to average pooling 3d operation - 5d activations in NCDHW format
                        (shape [minibatch, channels, depth, height, width]) or NDHWC format
                        (shape [minibatch, depth, height, width, channels]) (NUMERIC type)
* **Pooling3DConfig** - [Pooling3DConfig]

## <a name="batchToSpace">batchToSpace</a>
```JAVA
INDArray batchToSpace(INDArray x, int[] blocks, int[] croppingTop, int[] croppingBottom)

SDVariable batchToSpace(SDVariable x, int[] blocks, int[] croppingTop, int[] croppingBottom)
SDVariable batchToSpace(String name, SDVariable x, int[] blocks, int[] croppingTop, int[] croppingBottom)
```
Convolution 2d layer batch to space operation on 4d input.

Reduces input batch dimension by rearranging data into a larger spatial dimensions

* **x** - Input variable. 4d input (NUMERIC type)
* **blocks** - Block size, in the height/width dimension (Size: Exactly(count=2)
* **croppingTop** -  (Size: Exactly(count=2)
* **croppingBottom** -  (Size: Exactly(count=2)

## <a name="col2Im">col2Im</a>
```JAVA
INDArray col2Im(INDArray in, Conv2DConfig conv2DConfig)

SDVariable col2Im(SDVariable in, Conv2DConfig conv2DConfig)
SDVariable col2Im(String name, SDVariable in, Conv2DConfig conv2DConfig)
```
col2im operation for use in 2D convolution operations. Outputs a 4d array with shape

[minibatch, inputChannels, height, width]

* **in** - Input - rank 6 input with shape [minibatch, inputChannels, kernelHeight, kernelWidth, outputHeight, outputWidth] (NUMERIC type)
* **Conv2DConfig** - [Conv2DConfig]

## <a name="conv1d">conv1d</a>
```JAVA
INDArray conv1d(INDArray input, INDArray weights, INDArray bias, Conv1DConfig conv1DConfig)

SDVariable conv1d(SDVariable input, SDVariable weights, SDVariable bias, Conv1DConfig conv1DConfig)
SDVariable conv1d(String name, SDVariable input, SDVariable weights, SDVariable bias, Conv1DConfig conv1DConfig)
INDArray conv1d(INDArray input, INDArray weights, Conv1DConfig conv1DConfig)

SDVariable conv1d(SDVariable input, SDVariable weights, Conv1DConfig conv1DConfig)
SDVariable conv1d(String name, SDVariable input, SDVariable weights, Conv1DConfig conv1DConfig)
```
Conv1d operation.

* **input** - the inputs to conv1d (NUMERIC type)
* **weights** - weights for conv1d op - rank 3 array with shape [kernelSize, inputChannels, outputChannels] (NUMERIC type)
* **bias** - bias for conv1d op - rank 1 array with shape [outputChannels]. May be null. (NUMERIC type)
* **Conv1DConfig** - [Conv1DConfig]
* **input** - the inputs to conv1d (NUMERIC type)
* **weights** - weights for conv1d op - rank 3 array with shape [kernelSize, inputChannels, outputChannels] (NUMERIC type)
* **Conv1DConfig** - [Conv1DConfig]

## <a name="conv2d">conv2d</a>
```JAVA
INDArray conv2d(INDArray layerInput, INDArray weights, INDArray bias, Conv2DConfig conv2DConfig)

SDVariable conv2d(SDVariable layerInput, SDVariable weights, SDVariable bias, Conv2DConfig conv2DConfig)
SDVariable conv2d(String name, SDVariable layerInput, SDVariable weights, SDVariable bias, Conv2DConfig conv2DConfig)
INDArray conv2d(INDArray layerInput, INDArray weights, Conv2DConfig conv2DConfig)

SDVariable conv2d(SDVariable layerInput, SDVariable weights, Conv2DConfig conv2DConfig)
SDVariable conv2d(String name, SDVariable layerInput, SDVariable weights, Conv2DConfig conv2DConfig)
```
2D Convolution operation with optional bias

* **layerInput** - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format (NUMERIC type)
* **weights** - Weights for the convolution operation. 4 dimensions with format [kernelHeight, kernelWidth, inputChannels, outputChannels] (NUMERIC type)
* **bias** - Optional 1D bias array with shape [outputChannels]. May be null. (NUMERIC type)
* **Conv2DConfig** - [Conv2DConfig]
* **layerInput** - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format (NUMERIC type)
* **weights** - Weights for the convolution operation. 4 dimensions with format [kernelHeight, kernelWidth, inputChannels, outputChannels] (NUMERIC type)
* **Conv2DConfig** - [Conv2DConfig]

## <a name="conv3d">conv3d</a>
```JAVA
INDArray conv3d(INDArray input, INDArray weights, INDArray bias, Conv3DConfig conv3DConfig)

SDVariable conv3d(SDVariable input, SDVariable weights, SDVariable bias, Conv3DConfig conv3DConfig)
SDVariable conv3d(String name, SDVariable input, SDVariable weights, SDVariable bias, Conv3DConfig conv3DConfig)
INDArray conv3d(INDArray input, INDArray weights, Conv3DConfig conv3DConfig)

SDVariable conv3d(SDVariable input, SDVariable weights, Conv3DConfig conv3DConfig)
SDVariable conv3d(String name, SDVariable input, SDVariable weights, Conv3DConfig conv3DConfig)
```
Convolution 3D operation with optional bias 

* **input** - the input to average pooling 3d operation - 5d activations in NCDHW format
(shape [minibatch, channels, depth, height, width]) or NDHWC format
(shape [minibatch, depth, height, width, channels]) (NUMERIC type)
* **weights** -  Weights for conv3d. Rank 5 with shape [kernelDepth, kernelHeight, kernelWidth, inputChannels, outputChannels]. (NUMERIC type)
* **bias** -  Optional 1D bias array with shape [outputChannels]. May be null. (NUMERIC type)
* **Conv3DConfig** - [Conv3DConfig]
* **input** - the input to average pooling 3d operation - 5d activations in NCDHW format
(shape [minibatch, channels, depth, height, width]) or NDHWC format
(shape [minibatch, depth, height, width, channels]) (NUMERIC type)
* **weights** -  Weights for conv3d. Rank 5 with shape [kernelDepth, kernelHeight, kernelWidth, inputChannels, outputChannels]. (NUMERIC type)
* **Conv3DConfig** - [Conv3DConfig]

## <a name="deconv2d">deconv2d</a>
```JAVA
INDArray deconv2d(INDArray layerInput, INDArray weights, INDArray bias, DeConv2DConfig deConv2DConfig)

SDVariable deconv2d(SDVariable layerInput, SDVariable weights, SDVariable bias, DeConv2DConfig deConv2DConfig)
SDVariable deconv2d(String name, SDVariable layerInput, SDVariable weights, SDVariable bias, DeConv2DConfig deConv2DConfig)
INDArray deconv2d(INDArray layerInput, INDArray weights, DeConv2DConfig deConv2DConfig)

SDVariable deconv2d(SDVariable layerInput, SDVariable weights, DeConv2DConfig deConv2DConfig)
SDVariable deconv2d(String name, SDVariable layerInput, SDVariable weights, DeConv2DConfig deConv2DConfig)
```
2D deconvolution operation with optional bias

* **layerInput** - the input to deconvolution 2d operation - 4d CNN (image) activations in NCHW format
(shape [minibatch, channels, height, width]) or NHWC format (shape [minibatch, height, width, channels]) (NUMERIC type)
* **weights** - Weights for the 2d deconvolution operation. 4 dimensions with format [inputChannels, outputChannels, kernelHeight, kernelWidth] (NUMERIC type)
* **bias** - Optional 1D bias array with shape [outputChannels]. May be null. (NUMERIC type)
* **DeConv2DConfig** - [DeConv2DConfig]
* **layerInput** - the input to deconvolution 2d operation - 4d CNN (image) activations in NCHW format
(shape [minibatch, channels, height, width]) or NHWC format (shape [minibatch, height, width, channels]) (NUMERIC type)
* **weights** - Weights for the 2d deconvolution operation. 4 dimensions with format [inputChannels, outputChannels, kernelHeight, kernelWidth] (NUMERIC type)
* **DeConv2DConfig** - [DeConv2DConfig]

## <a name="deconv3d">deconv3d</a>
```JAVA
INDArray deconv3d(INDArray input, INDArray weights, INDArray bias, DeConv3DConfig deConv3DConfig)

SDVariable deconv3d(SDVariable input, SDVariable weights, SDVariable bias, DeConv3DConfig deConv3DConfig)
SDVariable deconv3d(String name, SDVariable input, SDVariable weights, SDVariable bias, DeConv3DConfig deConv3DConfig)
INDArray deconv3d(INDArray input, INDArray weights, DeConv3DConfig deConv3DConfig)

SDVariable deconv3d(SDVariable input, SDVariable weights, DeConv3DConfig deConv3DConfig)
SDVariable deconv3d(String name, SDVariable input, SDVariable weights, DeConv3DConfig deConv3DConfig)
```
3D CNN deconvolution operation with or without optional bias

* **input** - Input array - shape [bS, iD, iH, iW, iC] (NDHWC) or [bS, iC, iD, iH, iW] (NCDHW) (NUMERIC type)
* **weights** - Weights array - shape [kD, kH, kW, oC, iC] (NUMERIC type)
* **bias** - Bias array - optional, may be null. If non-null, must have shape [outputChannels] (NUMERIC type)
* **DeConv3DConfig** - [DeConv3DConfig]
* **input** - Input array - shape [bS, iD, iH, iW, iC] (NDHWC) or [bS, iC, iD, iH, iW] (NCDHW) (NUMERIC type)
* **weights** - Weights array - shape [kD, kH, kW, oC, iC] (NUMERIC type)
* **DeConv3DConfig** - [DeConv3DConfig]

## <a name="depthToSpace">depthToSpace</a>
```JAVA
INDArray depthToSpace(INDArray x, int blockSize, DataFormat dataFormat)

SDVariable depthToSpace(SDVariable x, int blockSize, DataFormat dataFormat)
SDVariable depthToSpace(String name, SDVariable x, int blockSize, DataFormat dataFormat)
```
Convolution 2d layer batch to space operation on 4d input.<br>
Reduces input channels dimension by rearranging data into a larger spatial dimensions<br>
Example: if input has shape [mb, 8, 2, 2] and block size is 2, then output size is [mb, 8/(2*2), 2*2, 2*2]

= [mb, 2, 4, 4]

* **x** - the input to depth to space pooling 2d operation - 4d activations in NCHW format
                   (shape [minibatch, channels, height, width]) or NHWC format (shape [minibatch, height, width, channels]) (NUMERIC type)
* **blockSize** - Block size, in the height/width dimension
* **dataFormat** - Data format: "NCHW" or "NHWC"

## <a name="depthWiseConv2d">depthWiseConv2d</a>
```JAVA
INDArray depthWiseConv2d(INDArray layerInput, INDArray depthWeights, INDArray bias, Conv2DConfig conv2DConfig)

SDVariable depthWiseConv2d(SDVariable layerInput, SDVariable depthWeights, SDVariable bias, Conv2DConfig conv2DConfig)
SDVariable depthWiseConv2d(String name, SDVariable layerInput, SDVariable depthWeights, SDVariable bias, Conv2DConfig conv2DConfig)
INDArray depthWiseConv2d(INDArray layerInput, INDArray depthWeights, Conv2DConfig conv2DConfig)

SDVariable depthWiseConv2d(SDVariable layerInput, SDVariable depthWeights, Conv2DConfig conv2DConfig)
SDVariable depthWiseConv2d(String name, SDVariable layerInput, SDVariable depthWeights, Conv2DConfig conv2DConfig)
```
Depth-wise 2D convolution operation with optional bias 

* **layerInput** - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format (NUMERIC type)
* **depthWeights** - Depth-wise conv2d weights. 4 dimensions with format [kernelHeight, kernelWidth, inputChannels, depthMultiplier] (NUMERIC type)
* **bias** - Optional 1D bias array with shape [outputChannels]. May be null. (NUMERIC type)
* **Conv2DConfig** - [Conv2DConfig]
* **layerInput** - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format (NUMERIC type)
* **depthWeights** - Depth-wise conv2d weights. 4 dimensions with format [kernelHeight, kernelWidth, inputChannels, depthMultiplier] (NUMERIC type)
* **Conv2DConfig** - [Conv2DConfig]

## <a name="dilation2D">dilation2D</a>
```JAVA
INDArray dilation2D(INDArray df, INDArray weights, int[] strides, int[] rates, boolean isSameMode)

SDVariable dilation2D(SDVariable df, SDVariable weights, int[] strides, int[] rates, boolean isSameMode)
SDVariable dilation2D(String name, SDVariable df, SDVariable weights, int[] strides, int[] rates, boolean isSameMode)
```
TODO doc string

* **df** -  (NUMERIC type)
* **weights** - df (NUMERIC type)
* **strides** - weights (Size: Exactly(count=2)
* **rates** - strides (Size: Exactly(count=2)
* **isSameMode** - isSameMode

## <a name="extractImagePatches">extractImagePatches</a>
```JAVA
INDArray extractImagePatches(INDArray input, int kH, int kW, int sH, int sW, int rH, int rW, boolean sameMode)

SDVariable extractImagePatches(SDVariable input, int kH, int kW, int sH, int sW, int rH, int rW, boolean sameMode)
SDVariable extractImagePatches(String name, SDVariable input, int kH, int kW, int sH, int sW, int rH, int rW, boolean sameMode)
```
Extract image patches 

* **input** - Input array. Must be rank 4, with shape [minibatch, height, width, channels] (NUMERIC type)
* **kH** - Kernel height
* **kW** - Kernel width
* **sH** - Stride height
* **sW** - Stride width
* **rH** - Rate height
* **rW** - Rate width
* **sameMode** - If true: use same mode padding. If false

## <a name="im2Col">im2Col</a>
```JAVA
INDArray im2Col(INDArray in, Conv2DConfig conv2DConfig)

SDVariable im2Col(SDVariable in, Conv2DConfig conv2DConfig)
SDVariable im2Col(String name, SDVariable in, Conv2DConfig conv2DConfig)
```
im2col operation for use in 2D convolution operations. Outputs a 6d array with shape

[minibatch, inputChannels, kernelHeight, kernelWidth, outputHeight, outputWidth]   

* **in** - Input - rank 4 input with shape [minibatch, inputChannels, height, width] (NUMERIC type)
* **Conv2DConfig** - [Conv2DConfig]

## <a name="localResponseNormalization">localResponseNormalization</a>
```JAVA
INDArray localResponseNormalization(INDArray input, LocalResponseNormalizationConfig localResponseNormalizationConfig)

SDVariable localResponseNormalization(SDVariable input, LocalResponseNormalizationConfig localResponseNormalizationConfig)
SDVariable localResponseNormalization(String name, SDVariable input, LocalResponseNormalizationConfig localResponseNormalizationConfig)
```
2D convolution layer operation - local response normalization

* **input** - the inputs to lrn (NUMERIC type)
* **LocalResponseNormalizationConfig** - [LocalResponseNormalizationConfig]

## <a name="maxPoolWithArgmax">maxPoolWithArgmax</a>
```JAVA
INDArray[] maxPoolWithArgmax(INDArray input, Pooling2DConfig pooling2DConfig)

SDVariable[] maxPoolWithArgmax(SDVariable input, Pooling2DConfig pooling2DConfig)
SDVariable[] maxPoolWithArgmax(String name, SDVariable input, Pooling2DConfig pooling2DConfig)
```
2D Convolution layer operation - Max pooling on the input and outputs both max values and indices 

* **input** - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format
                        (shape [minibatch, channels, height, width]) or NHWC format (shape [minibatch, height, width, channels]) (NUMERIC type)
* **Pooling2DConfig** - [Pooling2DConfig]

## <a name="maxPooling2d">maxPooling2d</a>
```JAVA
INDArray maxPooling2d(INDArray input, Pooling2DConfig pooling2DConfig)

SDVariable maxPooling2d(SDVariable input, Pooling2DConfig pooling2DConfig)
SDVariable maxPooling2d(String name, SDVariable input, Pooling2DConfig pooling2DConfig)
```
2D Convolution layer operation - max pooling 2d 

* **input** - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format
                        (shape [minibatch, channels, height, width]) or NHWC format (shape [minibatch, height, width, channels]) (NUMERIC type)
* **Pooling2DConfig** - [Pooling2DConfig]

## <a name="maxPooling3d">maxPooling3d</a>
```JAVA
INDArray maxPooling3d(INDArray input, Pooling3DConfig pooling3DConfig)

SDVariable maxPooling3d(SDVariable input, Pooling3DConfig pooling3DConfig)
SDVariable maxPooling3d(String name, SDVariable input, Pooling3DConfig pooling3DConfig)
```
3D convolution layer operation - max pooling 3d operation.

* **input** - the input to average pooling 3d operation - 5d activations in NCDHW format
                        (shape [minibatch, channels, depth, height, width]) or NDHWC format
                        (shape [minibatch, depth, height, width, channels]) (NUMERIC type)
* **Pooling3DConfig** - [Pooling3DConfig]

## <a name="separableConv2d">separableConv2d</a>
```JAVA
INDArray separableConv2d(INDArray layerInput, INDArray depthWeights, INDArray pointWeights, INDArray bias, Conv2DConfig conv2DConfig)

SDVariable separableConv2d(SDVariable layerInput, SDVariable depthWeights, SDVariable pointWeights, SDVariable bias, Conv2DConfig conv2DConfig)
SDVariable separableConv2d(String name, SDVariable layerInput, SDVariable depthWeights, SDVariable pointWeights, SDVariable bias, Conv2DConfig conv2DConfig)
INDArray separableConv2d(INDArray layerInput, INDArray depthWeights, INDArray pointWeights, Conv2DConfig conv2DConfig)

SDVariable separableConv2d(SDVariable layerInput, SDVariable depthWeights, SDVariable pointWeights, Conv2DConfig conv2DConfig)
SDVariable separableConv2d(String name, SDVariable layerInput, SDVariable depthWeights, SDVariable pointWeights, Conv2DConfig conv2DConfig)
```
Separable 2D convolution operation with optional bias 

* **layerInput** - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format
                     (shape [minibatch, channels, height, width]) or NHWC format (shape [minibatch, height, width, channels]) (NUMERIC type)
* **depthWeights** - Separable conv2d depth weights. 4 dimensions with format [kernelHeight, kernelWidth, inputChannels, depthMultiplier] (NUMERIC type)
* **pointWeights** - Point weights, rank 4 with format [1, 1, inputChannels*depthMultiplier, outputChannels]. May be null (NUMERIC type)
* **bias** - Optional bias, rank 1 with shape [outputChannels]. May be null. (NUMERIC type)
* **Conv2DConfig** - [Conv2DConfig]
* **layerInput** - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format
                     (shape [minibatch, channels, height, width]) or NHWC format (shape [minibatch, height, width, channels]) (NUMERIC type)
* **depthWeights** - Separable conv2d depth weights. 4 dimensions with format [kernelHeight, kernelWidth, inputChannels, depthMultiplier] (NUMERIC type)
* **pointWeights** - Point weights, rank 4 with format [1, 1, inputChannels*depthMultiplier, outputChannels]. May be null (NUMERIC type)
* **Conv2DConfig** - [Conv2DConfig]

## <a name="spaceToBatch">spaceToBatch</a>
```JAVA
INDArray spaceToBatch(INDArray x, int[] blocks, int[] paddingTop, int[] paddingBottom)

SDVariable spaceToBatch(SDVariable x, int[] blocks, int[] paddingTop, int[] paddingBottom)
SDVariable spaceToBatch(String name, SDVariable x, int[] blocks, int[] paddingTop, int[] paddingBottom)
```
Convolution 2d layer space to batch operation on 4d input.

Increases input batch dimension by rearranging data from spatial dimensions into batch dimension 

* **x** - Input variable. 4d input (NUMERIC type)
* **blocks** - Block size, in the height/width dimension (Size: Exactly(count=2)
* **paddingTop** - Optional 2d int[] array for padding the result: values [[pad top, pad bottom], [pad left, pad right]] (Size: Exactly(count=2)
* **paddingBottom** - Optional 2d int[] array for padding the result: values [[pad top, pad bottom], [pad left, pad right]] (Size: Exactly(count=2)

## <a name="spaceToDepth">spaceToDepth</a>
```JAVA
INDArray spaceToDepth(INDArray x, int blockSize, DataFormat dataFormat)

SDVariable spaceToDepth(SDVariable x, int blockSize, DataFormat dataFormat)
SDVariable spaceToDepth(String name, SDVariable x, int blockSize, DataFormat dataFormat)
```
Convolution 2d layer space to depth operation on 4d input.<br>
Increases input channels (reduced spatial dimensions) by rearranging data into a larger channels dimension<br>
Example: if input has shape [mb, 2, 4, 4] and block size is 2, then output size is [mb, 8/(2*2), 2*2, 2*2]

= [mb, 2, 4, 4] 

* **x** - the input to depth to space pooling 2d operation - 4d activations in NCHW format
                   (shape [minibatch, channels, height, width]) or NHWC format (shape [minibatch, height, width, channels]) (NUMERIC type)
* **blockSize** -  Block size, in the height/width dimension
* **dataFormat** - Data format: "NCHW" or "NHWC"

## <a name="upsampling2d">upsampling2d</a>
```JAVA
INDArray upsampling2d(INDArray input, int scale)

SDVariable upsampling2d(SDVariable input, int scale)
SDVariable upsampling2d(String name, SDVariable input, int scale)
```
Upsampling layer for 2D inputs.

scale is used for both height and width dimensions. 

* **input** - Input in NCHW format (NUMERIC type)
* **scale** - The scale for both height and width dimensions.

## <a name="upsampling2d">upsampling2d</a>
```JAVA
INDArray upsampling2d(INDArray input, int scaleH, int scaleW, boolean nchw)

SDVariable upsampling2d(SDVariable input, int scaleH, int scaleW, boolean nchw)
SDVariable upsampling2d(String name, SDVariable input, int scaleH, int scaleW, boolean nchw)
```
2D Convolution layer operation - Upsampling 2d 

* **input** - Input in NCHW format (NUMERIC type)
* **scaleH** - Scale to upsample in height dimension
* **scaleW** - Scale to upsample in width dimension
* **nchw** - If true: input is in NCHW (minibatch, channels, height, width) format. False: NHWC format

# Configuration Classes <configs>
## <a name="Conv1DConfig"></a>
* **k** - Kernel (LONG type) - default = -1
* **s** - stride (LONG type) - default = 1
* **p** - padding (LONG type) - default = 0
* **d** - dilation (LONG type) - default = 1
* **isSameMode** - Same mode (BOOL type) - default = true
* **dataFormat** - Data format (STRING type) - default = NCW

Used in these ops: 
[conv1d](#conv1d)
## <a name="Conv2DConfig"></a>
* **kH** - Kernel height (LONG type) - default = -1
* **kW** - Kernel width (LONG type) - default = -1
* **sH** - Stride along height dimension (LONG type) - default = 1
* **sW** - Stride along width dimension (LONG type) - default = 1
* **pH** - Padding along height dimension (LONG type) - default = 0
* **pW** - Padding along width dimension (LONG type) - default = 0
* **dH** - Dilation along height dimension (LONG type) - default = 1
* **dW** - Dilation along width dimension (LONG type) - default = 1
* **isSameMode** - Same mode (BOOL type) - default = true
* **dataFormat** - Data format (STRING type) - default = NCHW

Used in these ops: 
[col2Im](#col2Im)
[conv2d](#conv2d)
[depthWiseConv2d](#depthWiseConv2d)
[im2Col](#im2Col)
[separableConv2d](#separableConv2d)
## <a name="Conv3DConfig"></a>
* **kD** - Kernel depth (LONG type) - default = -1
* **kW** - Kernel width (LONG type) - default = -1
* **kH** - Kernel height (LONG type) - default = -1
* **sD** - Stride depth (LONG type) - default = 1
* **sW** - Stride width (LONG type) - default = 1
* **sH** - Stride height (LONG type) - default = 1
* **pD** - Padding depth (LONG type) - default = 0
* **pW** - Padding width (LONG type) - default = 0
* **pH** - Padding height (LONG type) - default = 0
* **dD** - Dilation depth (LONG type) - default = 1
* **dW** - Dilation width (LONG type) - default = 1
* **dH** - Dilation height (LONG type) - default = 1
* **biasUsed** - biasUsed (BOOL type) - default = false
* **isSameMode** - Same mode (BOOL type) - default = true
* **dataFormat** - Data format (STRING type) - default = NDHWC

Used in these ops: 
[conv3d](#conv3d)
## <a name="DeConv2DConfig"></a>
* **kH** - Kernel height (LONG type) - default = -1
* **kW** - Kernel width (LONG type) - default = -1
* **sH** - Stride along height dimension (LONG type) - default = 1
* **sW** - Stride along width dimension (LONG type) - default = 1
* **pH** - Padding along height dimension (LONG type) - default = 0
* **pW** - Padding along width dimension (LONG type) - default = 0
* **dH** - Dilation along height dimension (LONG type) - default = 1
* **dW** - Dilation along width dimension (LONG type) - default = 1
* **isSameMode** - Same mode (BOOL type) - default = false
* **dataFormat** - Data format (STRING type) - default = NCHW

Used in these ops: 
[deconv2d](#deconv2d)
## <a name="DeConv3DConfig"></a>
* **kD** - Kernel depth (LONG type) - default = -1
* **kW** - Kernel width (LONG type) - default = -1
* **kH** - Kernel height (LONG type) - default = -1
* **sD** - Stride depth (LONG type) - default = 1
* **sW** - Stride width (LONG type) - default = 1
* **sH** - Stride height (LONG type) - default = 1
* **pD** - Padding depth (LONG type) - default = 0
* **pW** - Padding width (LONG type) - default = 0
* **pH** - Padding height (LONG type) - default = 0
* **dD** - Dilation depth (LONG type) - default = 1
* **dW** - Dilation width (LONG type) - default = 1
* **dH** - Dilation height (LONG type) - default = 1
* **isSameMode** - Same mode (BOOL type) - default = false
* **dataFormat** - Data format (STRING type) - default = NCDHW

Used in these ops: 
[deconv3d](#deconv3d)
## <a name="Pooling2DConfig"></a>
* **kH** - Kernel height (LONG type) - default = -1
* **kW** - Kernel width (LONG type) - default = -1
* **sH** - Stride along height dimension (LONG type) - default = 1
* **sW** - Stride along width dimension (LONG type) - default = 1
* **pH** - Padding along height dimension (LONG type) - default = 0
* **pW** - Padding along width dimension (LONG type) - default = 0
* **dH** - Dilation along height dimension (LONG type) - default = 1
* **dW** - Dilation along width dimension (LONG type) - default = 1
* **isSameMode** - Same mode (BOOL type) - default = true
* **dataFormat** - Data format (STRING type) - default = nchw

Used in these ops: 
[avgPooling2d](#avgPooling2d)
[maxPoolWithArgmax](#maxPoolWithArgmax)
[maxPooling2d](#maxPooling2d)
## <a name="Pooling3DConfig"></a>
* **kD** - Kernel depth (LONG type) - default = -1
* **kW** - Kernel width (LONG type) - default = -1
* **kH** - Kernel height (LONG type) - default = -1
* **sD** - Stride depth (LONG type) - default = 1
* **sW** - Stride width (LONG type) - default = 1
* **sH** - Stride height (LONG type) - default = 1
* **pD** - Padding depth (LONG type) - default = 0
* **pW** - Padding width (LONG type) - default = 0
* **pH** - Padding height (LONG type) - default = 0
* **dD** - Dilation depth (LONG type) - default = 1
* **dW** - Dilation width (LONG type) - default = 1
* **dH** - Dilation height (LONG type) - default = 1
* **isSameMode** - Same mode (BOOL type) - default = true
* **dataFormat** - Data format (STRING type) - default = NCDHW

Used in these ops: 
[avgPooling3d](#avgPooling3d)
[maxPooling3d](#maxPooling3d)
## <a name="LocalResponseNormalizationConfig"></a>
* **alpha** - alpha (NUMERIC type) - default = 1
* **beta** - beta (NUMERIC type) - default = 0.5
* **bias** - bias (NUMERIC type) - default = 1
* **depth** - depth (INT type) - default = 5

Used in these ops: 
[localResponseNormalization](#localResponseNormalization)

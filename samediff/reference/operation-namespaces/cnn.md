# CNN

## Operation classes

### avgPooling2d

```java
INDArray avgPooling2d(INDArray input, Pooling2DConfig pooling2DConfig)

SDVariable avgPooling2d(SDVariable input, Pooling2DConfig pooling2DConfig)
SDVariable avgPooling2d(String name, SDVariable input, Pooling2DConfig pooling2DConfig)
```

2D Convolution layer operation - average pooling 2d

* **input**  (NUMERIC) - the input to average pooling 2d operation - 4d CNN (image) activations in NCHW format (shape \[minibatch, channels, height, width]) or NHWC format (shape \[minibatch, height, width, channels])
* **Pooling2DConfig** - see [Pooling2DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#pooling-2-dconfig)

### avgPooling3d

```java
INDArray avgPooling3d(INDArray input, Pooling3DConfig pooling3DConfig)

SDVariable avgPooling3d(SDVariable input, Pooling3DConfig pooling3DConfig)
SDVariable avgPooling3d(String name, SDVariable input, Pooling3DConfig pooling3DConfig)
```

3D convolution layer operation - average pooling 3d

* **input**  (NUMERIC) - the input to average pooling 3d operation - 5d activations in NCDHW format (shape \[minibatch, channels, depth, height, width]) or NDHWC format (shape \[minibatch, depth, height, width, channels])
* **Pooling3DConfig** - see [Pooling3DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#pooling-3-dconfig)

### batchToSpace

```java
INDArray batchToSpace(INDArray x, int[] blocks, int[] croppingTop, int[] croppingBottom)

SDVariable batchToSpace(SDVariable x, int[] blocks, int[] croppingTop, int[] croppingBottom)
SDVariable batchToSpace(String name, SDVariable x, int[] blocks, int[] croppingTop, int[] croppingBottom)
```

Convolution 2d layer batch to space operation on 4d input.

Reduces input batch dimension by rearranging data into a larger spatial dimensions

* **x**  (NUMERIC) - Input variable. 4d input
* **blocks** - Block size, in the height/width dimension (Size: Exactly(count=2))
* **croppingTop** -  (Size: Exactly(count=2))
* **croppingBottom** -  (Size: Exactly(count=2))

### col2Im

```java
INDArray col2Im(INDArray in, Conv2DConfig conv2DConfig)

SDVariable col2Im(SDVariable in, Conv2DConfig conv2DConfig)
SDVariable col2Im(String name, SDVariable in, Conv2DConfig conv2DConfig)
```

col2im operation for use in 2D convolution operations. Outputs a 4d array with shape

\[minibatch, inputChannels, height, width]

* **in**  (NUMERIC) - Input - rank 6 input with shape \[minibatch, inputChannels, kernelHeight, kernelWidth, outputHeight, outputWidth]
* **Conv2DConfig** - see [Conv2DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#conv-2-dconfig)

### conv1d

```java
INDArray conv1d(INDArray input, INDArray weights, INDArray bias, Conv1DConfig conv1DConfig)
INDArray conv1d(INDArray input, INDArray weights, Conv1DConfig conv1DConfig)

SDVariable conv1d(SDVariable input, SDVariable weights, SDVariable bias, Conv1DConfig conv1DConfig)
SDVariable conv1d(SDVariable input, SDVariable weights, Conv1DConfig conv1DConfig)
SDVariable conv1d(String name, SDVariable input, SDVariable weights, SDVariable bias, Conv1DConfig conv1DConfig)
SDVariable conv1d(String name, SDVariable input, SDVariable weights, Conv1DConfig conv1DConfig)
```

Conv1d operation.

* **input**  (NUMERIC) - the inputs to conv1d
* **weights**  (NUMERIC) - weights for conv1d op - rank 3 array with shape \[kernelSize, inputChannels, outputChannels]
* **bias**  (NUMERIC) - bias for conv1d op - rank 1 array with shape \[outputChannels]. May be null.
* **Conv1DConfig** - see [Conv1DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#conv-1-dconfig)

### conv2d

```java
INDArray conv2d(INDArray layerInput, INDArray weights, INDArray bias, Conv2DConfig conv2DConfig)
INDArray conv2d(INDArray layerInput, INDArray weights, Conv2DConfig conv2DConfig)

SDVariable conv2d(SDVariable layerInput, SDVariable weights, SDVariable bias, Conv2DConfig conv2DConfig)
SDVariable conv2d(SDVariable layerInput, SDVariable weights, Conv2DConfig conv2DConfig)
SDVariable conv2d(String name, SDVariable layerInput, SDVariable weights, SDVariable bias, Conv2DConfig conv2DConfig)
SDVariable conv2d(String name, SDVariable layerInput, SDVariable weights, Conv2DConfig conv2DConfig)
```

2D Convolution operation with optional bias

* **layerInput**  (NUMERIC) - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format
* **weights**  (NUMERIC) - Weights for the convolution operation. 4 dimensions with format \[kernelHeight, kernelWidth, inputChannels, outputChannels]
* **bias**  (NUMERIC) - Optional 1D bias array with shape \[outputChannels]. May be null.
* **Conv2DConfig** - see [Conv2DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#conv-2-dconfig)

### conv3d

```java
INDArray conv3d(INDArray input, INDArray weights, INDArray bias, Conv3DConfig conv3DConfig)
INDArray conv3d(INDArray input, INDArray weights, Conv3DConfig conv3DConfig)

SDVariable conv3d(SDVariable input, SDVariable weights, SDVariable bias, Conv3DConfig conv3DConfig)
SDVariable conv3d(SDVariable input, SDVariable weights, Conv3DConfig conv3DConfig)
SDVariable conv3d(String name, SDVariable input, SDVariable weights, SDVariable bias, Conv3DConfig conv3DConfig)
SDVariable conv3d(String name, SDVariable input, SDVariable weights, Conv3DConfig conv3DConfig)
```

Convolution 3D operation with optional bias

* **input**  (NUMERIC) - the input to average pooling 3d operation - 5d activations in NCDHW format (shape \[minibatch, channels, depth, height, width]) or NDHWC format (shape \[minibatch, depth, height, width, channels])
* **weights**  (NUMERIC) -  Weights for conv3d. Rank 5 with shape \[kernelDepth, kernelHeight, kernelWidth, inputChannels, outputChannels].
* **bias**  (NUMERIC) -  Optional 1D bias array with shape \[outputChannels]. May be null.
* **Conv3DConfig** - see [Conv3DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#conv-3-dconfig)

### deconv2d

```java
INDArray deconv2d(INDArray layerInput, INDArray weights, INDArray bias, DeConv2DConfig deConv2DConfig)
INDArray deconv2d(INDArray layerInput, INDArray weights, DeConv2DConfig deConv2DConfig)

SDVariable deconv2d(SDVariable layerInput, SDVariable weights, SDVariable bias, DeConv2DConfig deConv2DConfig)
SDVariable deconv2d(SDVariable layerInput, SDVariable weights, DeConv2DConfig deConv2DConfig)
SDVariable deconv2d(String name, SDVariable layerInput, SDVariable weights, SDVariable bias, DeConv2DConfig deConv2DConfig)
SDVariable deconv2d(String name, SDVariable layerInput, SDVariable weights, DeConv2DConfig deConv2DConfig)
```

2D deconvolution operation with optional bias

* **layerInput**  (NUMERIC) - the input to deconvolution 2d operation - 4d CNN (image) activations in NCHW format (shape \[minibatch, channels, height, width]) or NHWC format (shape \[minibatch, height, width, channels])
* **weights**  (NUMERIC) - Weights for the 2d deconvolution operation. 4 dimensions with format \[inputChannels, outputChannels, kernelHeight, kernelWidth]
* **bias**  (NUMERIC) - Optional 1D bias array with shape \[outputChannels]. May be null.
* **DeConv2DConfig** - see [DeConv2DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#deconv-2-dconfig)

### deconv3d

```java
INDArray deconv3d(INDArray input, INDArray weights, INDArray bias, DeConv3DConfig deConv3DConfig)
INDArray deconv3d(INDArray input, INDArray weights, DeConv3DConfig deConv3DConfig)

SDVariable deconv3d(SDVariable input, SDVariable weights, SDVariable bias, DeConv3DConfig deConv3DConfig)
SDVariable deconv3d(SDVariable input, SDVariable weights, DeConv3DConfig deConv3DConfig)
SDVariable deconv3d(String name, SDVariable input, SDVariable weights, SDVariable bias, DeConv3DConfig deConv3DConfig)
SDVariable deconv3d(String name, SDVariable input, SDVariable weights, DeConv3DConfig deConv3DConfig)
```

3D CNN deconvolution operation with or without optional bias

* **input**  (NUMERIC) - Input array - shape \[bS, iD, iH, iW, iC] (NDHWC) or \[bS, iC, iD, iH, iW] (NCDHW)
* **weights**  (NUMERIC) - Weights array - shape \[kD, kH, kW, oC, iC]
* **bias**  (NUMERIC) - Bias array - optional, may be null. If non-null, must have shape \[outputChannels]
* **DeConv3DConfig** - see [DeConv3DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#deconv-3-dconfig)

### depthToSpace

```java
INDArray depthToSpace(INDArray x, int blockSize, DataFormat dataFormat)

SDVariable depthToSpace(SDVariable x, int blockSize, DataFormat dataFormat)
SDVariable depthToSpace(String name, SDVariable x, int blockSize, DataFormat dataFormat)
```

Convolution 2d layer batch to space operation on 4d input.\
Reduces input channels dimension by rearranging data into a larger spatial dimensions\
Example: if input has shape \[mb, 8, 2, 2] and block size is 2, then output size is \[mb, 8/(2\_2), 2\_2, 2\*2]

\= \[mb, 2, 4, 4]

* **x**  (NUMERIC) - the input to depth to space pooling 2d operation - 4d activations in NCHW format (shape \[minibatch, channels, height, width]) or NHWC format (shape \[minibatch, height, width, channels])
* **blockSize** - Block size, in the height/width dimension
* **dataFormat** - Data format: "NCHW" or "NHWC"

### depthWiseConv2d

```java
INDArray depthWiseConv2d(INDArray layerInput, INDArray depthWeights, INDArray bias, Conv2DConfig conv2DConfig)
INDArray depthWiseConv2d(INDArray layerInput, INDArray depthWeights, Conv2DConfig conv2DConfig)

SDVariable depthWiseConv2d(SDVariable layerInput, SDVariable depthWeights, SDVariable bias, Conv2DConfig conv2DConfig)
SDVariable depthWiseConv2d(SDVariable layerInput, SDVariable depthWeights, Conv2DConfig conv2DConfig)
SDVariable depthWiseConv2d(String name, SDVariable layerInput, SDVariable depthWeights, SDVariable bias, Conv2DConfig conv2DConfig)
SDVariable depthWiseConv2d(String name, SDVariable layerInput, SDVariable depthWeights, Conv2DConfig conv2DConfig)
```

Depth-wise 2D convolution operation with optional bias

* **layerInput**  (NUMERIC) - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format
* **depthWeights**  (NUMERIC) - Depth-wise conv2d weights. 4 dimensions with format \[kernelHeight, kernelWidth, inputChannels, depthMultiplier]
* **bias**  (NUMERIC) - Optional 1D bias array with shape \[outputChannels]. May be null.
* **Conv2DConfig** - see [Conv2DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#conv-2-dconfig)

### dilation2D

```java
INDArray dilation2D(INDArray df, INDArray weights, int[] strides, int[] rates, boolean isSameMode)

SDVariable dilation2D(SDVariable df, SDVariable weights, int[] strides, int[] rates, boolean isSameMode)
SDVariable dilation2D(String name, SDVariable df, SDVariable weights, int[] strides, int[] rates, boolean isSameMode)
```

TODO doc string

* **df**  (NUMERIC) -&#x20;
* **weights**  (NUMERIC) - df
* **strides** - weights (Size: Exactly(count=2))
* **rates** - strides (Size: Exactly(count=2))
* **isSameMode** - isSameMode

### extractImagePatches

```java
INDArray extractImagePatches(INDArray input, int kH, int kW, int sH, int sW, int rH, int rW, boolean sameMode)

SDVariable extractImagePatches(SDVariable input, int kH, int kW, int sH, int sW, int rH, int rW, boolean sameMode)
SDVariable extractImagePatches(String name, SDVariable input, int kH, int kW, int sH, int sW, int rH, int rW, boolean sameMode)
```

Extract image patches

* **input**  (NUMERIC) - Input array. Must be rank 4, with shape \[minibatch, height, width, channels]
* **kH** - Kernel height
* **kW** - Kernel width
* **sH** - Stride height
* **sW** - Stride width
* **rH** - Rate height
* **rW** - Rate width
* **sameMode** - If true: use same mode padding. If false

### im2Col

```java
INDArray im2Col(INDArray in, Conv2DConfig conv2DConfig)

SDVariable im2Col(SDVariable in, Conv2DConfig conv2DConfig)
SDVariable im2Col(String name, SDVariable in, Conv2DConfig conv2DConfig)
```

im2col operation for use in 2D convolution operations. Outputs a 6d array with shape

\[minibatch, inputChannels, kernelHeight, kernelWidth, outputHeight, outputWidth]

* **in**  (NUMERIC) - Input - rank 4 input with shape \[minibatch, inputChannels, height, width]
* **Conv2DConfig** - see [Conv2DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#conv-2-dconfig)

### localResponseNormalization

```java
INDArray localResponseNormalization(INDArray input, LocalResponseNormalizationConfig localResponseNormalizationConfig)

SDVariable localResponseNormalization(SDVariable input, LocalResponseNormalizationConfig localResponseNormalizationConfig)
SDVariable localResponseNormalization(String name, SDVariable input, LocalResponseNormalizationConfig localResponseNormalizationConfig)
```

2D convolution layer operation - local response normalization

* **input**  (NUMERIC) - the inputs to lrn
* **LocalResponseNormalizationConfig** - see [LocalResponseNormalizationConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#localresponsenormalizationconfig)

### maxPoolWithArgmax

```java
INDArray[] maxPoolWithArgmax(INDArray input, Pooling2DConfig pooling2DConfig)

SDVariable[] maxPoolWithArgmax(SDVariable input, Pooling2DConfig pooling2DConfig)
SDVariable[] maxPoolWithArgmax(String name, SDVariable input, Pooling2DConfig pooling2DConfig)
```

2D Convolution layer operation - Max pooling on the input and outputs both max values and indices

* **input**  (NUMERIC) - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format (shape \[minibatch, channels, height, width]) or NHWC format (shape \[minibatch, height, width, channels])
* **Pooling2DConfig** - see [Pooling2DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#pooling-2-dconfig)

### maxPooling2d

```java
INDArray maxPooling2d(INDArray input, Pooling2DConfig pooling2DConfig)

SDVariable maxPooling2d(SDVariable input, Pooling2DConfig pooling2DConfig)
SDVariable maxPooling2d(String name, SDVariable input, Pooling2DConfig pooling2DConfig)
```

2D Convolution layer operation - max pooling 2d

* **input**  (NUMERIC) - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format (shape \[minibatch, channels, height, width]) or NHWC format (shape \[minibatch, height, width, channels])
* **Pooling2DConfig** - see [Pooling2DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#pooling-2-dconfig)

### maxPooling3d

```java
INDArray maxPooling3d(INDArray input, Pooling3DConfig pooling3DConfig)

SDVariable maxPooling3d(SDVariable input, Pooling3DConfig pooling3DConfig)
SDVariable maxPooling3d(String name, SDVariable input, Pooling3DConfig pooling3DConfig)
```

3D convolution layer operation - max pooling 3d operation.

* **input**  (NUMERIC) - the input to average pooling 3d operation - 5d activations in NCDHW format (shape \[minibatch, channels, depth, height, width]) or NDHWC format (shape \[minibatch, depth, height, width, channels])
* **Pooling3DConfig** - see [Pooling3DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#pooling-3-dconfig)

### separableConv2d

```java
INDArray separableConv2d(INDArray layerInput, INDArray depthWeights, INDArray pointWeights, INDArray bias, Conv2DConfig conv2DConfig)
INDArray separableConv2d(INDArray layerInput, INDArray depthWeights, INDArray pointWeights, Conv2DConfig conv2DConfig)

SDVariable separableConv2d(SDVariable layerInput, SDVariable depthWeights, SDVariable pointWeights, SDVariable bias, Conv2DConfig conv2DConfig)
SDVariable separableConv2d(SDVariable layerInput, SDVariable depthWeights, SDVariable pointWeights, Conv2DConfig conv2DConfig)
SDVariable separableConv2d(String name, SDVariable layerInput, SDVariable depthWeights, SDVariable pointWeights, SDVariable bias, Conv2DConfig conv2DConfig)
SDVariable separableConv2d(String name, SDVariable layerInput, SDVariable depthWeights, SDVariable pointWeights, Conv2DConfig conv2DConfig)
```

Separable 2D convolution operation with optional bias

* **layerInput**  (NUMERIC) - the input to max pooling 2d operation - 4d CNN (image) activations in NCHW format (shape \[minibatch, channels, height, width]) or NHWC format (shape \[minibatch, height, width, channels])
* **depthWeights**  (NUMERIC) - Separable conv2d depth weights. 4 dimensions with format \[kernelHeight, kernelWidth, inputChannels, depthMultiplier]
* **pointWeights**  (NUMERIC) - Point weights, rank 4 with format \[1, 1, inputChannels\*depthMultiplier, outputChannels]. May be null
* **bias**  (NUMERIC) - Optional bias, rank 1 with shape \[outputChannels]. May be null.
* **Conv2DConfig** - see [Conv2DConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#conv-2-dconfig)

### spaceToBatch

```java
INDArray spaceToBatch(INDArray x, int[] blocks, int[] paddingTop, int[] paddingBottom)

SDVariable spaceToBatch(SDVariable x, int[] blocks, int[] paddingTop, int[] paddingBottom)
SDVariable spaceToBatch(String name, SDVariable x, int[] blocks, int[] paddingTop, int[] paddingBottom)
```

Convolution 2d layer space to batch operation on 4d input.

Increases input batch dimension by rearranging data from spatial dimensions into batch dimension

* **x**  (NUMERIC) - Input variable. 4d input
* **blocks** - Block size, in the height/width dimension (Size: Exactly(count=2))
* **paddingTop** - Optional 2d int\[] array for padding the result: values \[\[pad top, pad bottom], \[pad left, pad right]] (Size: Exactly(count=2))
* **paddingBottom** - Optional 2d int\[] array for padding the result: values \[\[pad top, pad bottom], \[pad left, pad right]] (Size: Exactly(count=2))

### spaceToDepth

```java
INDArray spaceToDepth(INDArray x, int blockSize, DataFormat dataFormat)

SDVariable spaceToDepth(SDVariable x, int blockSize, DataFormat dataFormat)
SDVariable spaceToDepth(String name, SDVariable x, int blockSize, DataFormat dataFormat)
```

Convolution 2d layer space to depth operation on 4d input.\
Increases input channels (reduced spatial dimensions) by rearranging data into a larger channels dimension\
Example: if input has shape \[mb, 2, 4, 4] and block size is 2, then output size is \[mb, 8/(2\_2), 2\_2, 2\*2]

\= \[mb, 2, 4, 4]

* **x**  (NUMERIC) - the input to depth to space pooling 2d operation - 4d activations in NCHW format (shape \[minibatch, channels, height, width]) or NHWC format (shape \[minibatch, height, width, channels])
* **blockSize** -  Block size, in the height/width dimension
* **dataFormat** - Data format: "NCHW" or "NHWC"

### upsampling2d

```java
INDArray upsampling2d(INDArray input, int scale)

SDVariable upsampling2d(SDVariable input, int scale)
SDVariable upsampling2d(String name, SDVariable input, int scale)
```

Upsampling layer for 2D inputs.

scale is used for both height and width dimensions.

* **input**  (NUMERIC) - Input in NCHW format
* **scale** - The scale for both height and width dimensions.

### upsampling2d

```java
INDArray upsampling2d(INDArray input, int scaleH, int scaleW, boolean nchw)

SDVariable upsampling2d(SDVariable input, int scaleH, int scaleW, boolean nchw)
SDVariable upsampling2d(String name, SDVariable input, int scaleH, int scaleW, boolean nchw)
```

2D Convolution layer operation - Upsampling 2d

* **input**  (NUMERIC) - Input in NCHW format
* **scaleH** - Scale to upsample in height dimension
* **scaleW** - Scale to upsample in width dimension
* **nchw** - If true: input is in NCHW (minibatch, channels, height, width) format. False: NHWC format

### upsampling3d

```java
INDArray upsampling3d(INDArray input, boolean ncdhw, int scaleD, int scaleH, int scaleW)

SDVariable upsampling3d(SDVariable input, boolean ncdhw, int scaleD, int scaleH, int scaleW)
SDVariable upsampling3d(String name, SDVariable input, boolean ncdhw, int scaleD, int scaleH, int scaleW)
```

3D Convolution layer operation - Upsampling 3d

* **input**  (NUMERIC) - Input in NCHW format
* **ncdhw** - If true: input is in NCDHW (minibatch, channels, depth, height, width) format. False: NDHWC format
* **scaleD** - Scale to upsample in depth dimension
* **scaleH** - Scale to upsample in height dimension
* **scaleW** - Scale to upsample in width dimension

## Configuration Classes

### Conv1DConfig

* **k** (LONG) - Kernel - default = -1
* **s** (LONG) - stride - default = 1
* **p** (LONG) - padding - default = 0
* **d** (LONG) - dilation - default = 1
* **isSameMode** (BOOL) - Same mode - default = true
* **dataFormat** (STRING) - Data format - default = NCW

Used in these ops: [conv1d](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#conv-1-d)

### Conv2DConfig

* **kH** (LONG) - Kernel height - default = -1
* **kW** (LONG) - Kernel width - default = -1
* **sH** (LONG) - Stride along height dimension - default = 1
* **sW** (LONG) - Stride along width dimension - default = 1
* **pH** (LONG) - Padding along height dimension - default = 0
* **pW** (LONG) - Padding along width dimension - default = 0
* **dH** (LONG) - Dilation along height dimension - default = 1
* **dW** (LONG) - Dilation along width dimension - default = 1
* **isSameMode** (BOOL) - Same mode - default = true
* **dataFormat** (STRING) - Data format - default = NCHW

Used in these ops: [col2Im](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#col-2-im) [conv2d](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#conv-2-d) [depthWiseConv2d](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#depthwiseconv-2-d) [im2Col](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#im-2-col) [separableConv2d](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#separableconv-2-d)

### Conv3DConfig

* **kD** (LONG) - Kernel depth - default = -1
* **kW** (LONG) - Kernel width - default = -1
* **kH** (LONG) - Kernel height - default = -1
* **sD** (LONG) - Stride depth - default = 1
* **sW** (LONG) - Stride width - default = 1
* **sH** (LONG) - Stride height - default = 1
* **pD** (LONG) - Padding depth - default = 0
* **pW** (LONG) - Padding width - default = 0
* **pH** (LONG) - Padding height - default = 0
* **dD** (LONG) - Dilation depth - default = 1
* **dW** (LONG) - Dilation width - default = 1
* **dH** (LONG) - Dilation height - default = 1
* **biasUsed** (BOOL) - biasUsed - default = false
* **isSameMode** (BOOL) - Same mode - default = true
* **dataFormat** (STRING) - Data format - default = NDHWC

Used in these ops: [conv3d](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#conv-3-d)

### DeConv2DConfig

* **kH** (LONG) - Kernel height - default = -1
* **kW** (LONG) - Kernel width - default = -1
* **sH** (LONG) - Stride along height dimension - default = 1
* **sW** (LONG) - Stride along width dimension - default = 1
* **pH** (LONG) - Padding along height dimension - default = 0
* **pW** (LONG) - Padding along width dimension - default = 0
* **dH** (LONG) - Dilation along height dimension - default = 1
* **dW** (LONG) - Dilation along width dimension - default = 1
* **isSameMode** (BOOL) - Same mode - default = false
* **dataFormat** (STRING) - Data format - default = NCHW

Used in these ops: [deconv2d](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#deconv-2-d)

### DeConv3DConfig

* **kD** (LONG) - Kernel depth - default = -1
* **kW** (LONG) - Kernel width - default = -1
* **kH** (LONG) - Kernel height - default = -1
* **sD** (LONG) - Stride depth - default = 1
* **sW** (LONG) - Stride width - default = 1
* **sH** (LONG) - Stride height - default = 1
* **pD** (LONG) - Padding depth - default = 0
* **pW** (LONG) - Padding width - default = 0
* **pH** (LONG) - Padding height - default = 0
* **dD** (LONG) - Dilation depth - default = 1
* **dW** (LONG) - Dilation width - default = 1
* **dH** (LONG) - Dilation height - default = 1
* **isSameMode** (BOOL) - Same mode - default = false
* **dataFormat** (STRING) - Data format - default = NCDHW

Used in these ops: [deconv3d](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#deconv-3-d)

### Pooling2DConfig

* **kH** (LONG) - Kernel height - default = -1
* **kW** (LONG) - Kernel width - default = -1
* **sH** (LONG) - Stride along height dimension - default = 1
* **sW** (LONG) - Stride along width dimension - default = 1
* **pH** (LONG) - Padding along height dimension - default = 0
* **pW** (LONG) - Padding along width dimension - default = 0
* **dH** (LONG) - Dilation along height dimension - default = 1
* **dW** (LONG) - Dilation along width dimension - default = 1
* **isSameMode** (BOOL) - Same mode - default = true
* **dataFormat** (STRING) - Data format - default = nchw

Used in these ops: [avgPooling2d](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#avgpooling-2-d) [maxPoolWithArgmax](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#maxpoolwithargmax) [maxPooling2d](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#maxpooling-2-d)

### Pooling3DConfig

* **kD** (LONG) - Kernel depth - default = -1
* **kW** (LONG) - Kernel width - default = -1
* **kH** (LONG) - Kernel height - default = -1
* **sD** (LONG) - Stride depth - default = 1
* **sW** (LONG) - Stride width - default = 1
* **sH** (LONG) - Stride height - default = 1
* **pD** (LONG) - Padding depth - default = 0
* **pW** (LONG) - Padding width - default = 0
* **pH** (LONG) - Padding height - default = 0
* **dD** (LONG) - Dilation depth - default = 1
* **dW** (LONG) - Dilation width - default = 1
* **dH** (LONG) - Dilation height - default = 1
* **isSameMode** (BOOL) - Same mode - default = true
* **dataFormat** (STRING) - Data format - default = NCDHW

Used in these ops: [avgPooling3d](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#avgpooling-3-d) [maxPooling3d](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#maxpooling-3-d)

### LocalResponseNormalizationConfig

* **alpha** (NUMERIC) - alpha - default = 1
* **beta** (NUMERIC) - beta - default = 0.5
* **bias** (NUMERIC) - bias - default = 1
* **depth** (INT) - depth - default = 5

Used in these ops: [localResponseNormalization](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/cnn.md#localresponsenormalization)

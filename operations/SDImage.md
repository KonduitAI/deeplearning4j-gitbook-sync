---
title: Image
short_title: Image
description: 
category: Operations
weight: 30
---
# Operation classes
## CropAndResize
```JAVA
INDArray CropAndResize(INDArray image, INDArray cropBoxes, INDArray boxIndices, INDArray cropOutSize, double extrapolationValue)

SDVariable CropAndResize(SDVariable image, SDVariable cropBoxes, SDVariable boxIndices, SDVariable cropOutSize, double extrapolationValue)
SDVariable CropAndResize(String name, SDVariable image, SDVariable cropBoxes, SDVariable boxIndices, SDVariable cropOutSize, double extrapolationValue)
INDArray CropAndResize(INDArray image, INDArray cropBoxes, INDArray boxIndices, INDArray cropOutSize)

SDVariable CropAndResize(SDVariable image, SDVariable cropBoxes, SDVariable boxIndices, SDVariable cropOutSize)
SDVariable CropAndResize(String name, SDVariable image, SDVariable cropBoxes, SDVariable boxIndices, SDVariable cropOutSize)
```
Given an input image and some crop boxes, extract out the image subsets and resize them to the specified size.

* **image**  (NUMERIC type) - Input image, with shape [batch, height, width, channels]
* **cropBoxes**  (NUMERIC type) - Float32 crop, shape [numBoxes, 4] with values in range 0 to 1
* **boxIndices**  (NUMERIC type) - Indices: which image (index to dimension 0) the cropBoxes belong to. Rank 1, shape [numBoxes]
* **cropOutSize**  (INT type) - Output size for the images - int32, rank 1 with values [outHeight, outWidth]
* **extrapolationValue** - Used for extrapolation, when applicable. 0.0 should be used for the default - default = 0.0

## adjustContrast
```JAVA
INDArray adjustContrast(INDArray in, double factor)

SDVariable adjustContrast(SDVariable in, double factor)
SDVariable adjustContrast(String name, SDVariable in, double factor)
```
Adjusts contrast of RGB or grayscale images.

* **in**  (NUMERIC type) - images to adjust. 3D shape or higher
* **factor** - multiplier for adjusting contrast

## adjustHue
```JAVA
INDArray adjustHue(INDArray in, double delta)

SDVariable adjustHue(SDVariable in, double delta)
SDVariable adjustHue(String name, SDVariable in, double delta)
```
Adjust hue of RGB image 

* **in**  (NUMERIC type) - image as 3D array
* **delta** - value to add to hue channel

## adjustSaturation
```JAVA
INDArray adjustSaturation(INDArray in, double factor)

SDVariable adjustSaturation(SDVariable in, double factor)
SDVariable adjustSaturation(String name, SDVariable in, double factor)
```
Adjust saturation of RGB images

* **in**  (NUMERIC type) - RGB image as 3D array
* **factor** - factor for saturation

## extractImagePatches
```JAVA
INDArray extractImagePatches(INDArray image, int[] kSizes, int[] strides, int[] rates, boolean sameMode)

SDVariable extractImagePatches(SDVariable image, int[] kSizes, int[] strides, int[] rates, boolean sameMode)
SDVariable extractImagePatches(String name, SDVariable image, int[] kSizes, int[] strides, int[] rates, boolean sameMode)
```
Given an input image, extract out image patches (of size kSizes - h x w) and place them in the depth dimension. 

* **image**  (NUMERIC type) - Input image to extract image patches from - shape [batch, height, width, channels]
* **kSizes** - Kernel size - size of the image patches, [height, width] (Size: Exactly(count=2))
* **strides** - Stride in the input dimension for extracting image patches, [stride_height, stride_width] (Size: Exactly(count=2))
* **rates** - Usually [1,1]. Equivalent to dilation rate in dilated convolutions - how far apart the output pixels
                 in the patches should be, in the input. A dilation of [a,b] means every {@code a}th pixel is taken
                 along the height/rows dimension, and every {@code b}th pixel is take along the width/columns dimension (Size: AtLeast(min=0))
* **sameMode** - Padding algorithm. If true: use Same padding

## hsvToRgb
```JAVA
INDArray hsvToRgb(INDArray input)

SDVariable hsvToRgb(SDVariable input)
SDVariable hsvToRgb(String name, SDVariable input)
```
Converting image from HSV to RGB format 

* **input**  (NUMERIC type) - 3D image

## nonMaxSuppression
```JAVA
INDArray nonMaxSuppression(INDArray boxes, INDArray scores, int maxOutSize, double iouThreshold, double scoreThreshold)

SDVariable nonMaxSuppression(SDVariable boxes, SDVariable scores, int maxOutSize, double iouThreshold, double scoreThreshold)
SDVariable nonMaxSuppression(String name, SDVariable boxes, SDVariable scores, int maxOutSize, double iouThreshold, double scoreThreshold)
```
Greedily selects a subset of bounding boxes in descending order of score

* **boxes**  (NUMERIC type) - Might be null. Name for the output variable
* **scores**  (NUMERIC type) - vector of shape [num_boxes]
* **maxOutSize** - scalar representing the maximum number of boxes to be selected
* **iouThreshold** - threshold for deciding whether boxes overlap too much with respect to IOU
* **scoreThreshold** - threshold for deciding when to remove boxes based on score

## randomCrop
```JAVA
INDArray randomCrop(INDArray input, INDArray shape)

SDVariable randomCrop(SDVariable input, SDVariable shape)
SDVariable randomCrop(String name, SDVariable input, SDVariable shape)
```
Randomly crops image

* **input**  (NUMERIC type) - input array
* **shape**  (INT type) - shape for crop

## rgbToHsv
```JAVA
INDArray rgbToHsv(INDArray input)

SDVariable rgbToHsv(SDVariable input)
SDVariable rgbToHsv(String name, SDVariable input)
```
Converting array from HSV to RGB format

* **input**  (NUMERIC type) - 3D image

## rgbToYiq
```JAVA
INDArray rgbToYiq(INDArray input)

SDVariable rgbToYiq(SDVariable input)
SDVariable rgbToYiq(String name, SDVariable input)
```
Converting array from RGB to YIQ format 

* **input**  (NUMERIC type) - 3D image

## rgbToYuv
```JAVA
INDArray rgbToYuv(INDArray input)

SDVariable rgbToYuv(SDVariable input)
SDVariable rgbToYuv(String name, SDVariable input)
```
Converting array from RGB to YUV format 

* **input**  (NUMERIC type) - 3D image

## yiqToRgb
```JAVA
INDArray yiqToRgb(INDArray input)

SDVariable yiqToRgb(SDVariable input)
SDVariable yiqToRgb(String name, SDVariable input)
```
Converting image from YIQ to RGB format 

* **input**  (NUMERIC type) - 3D image

## yuvToRgb
```JAVA
INDArray yuvToRgb(INDArray input)

SDVariable yuvToRgb(SDVariable input)
SDVariable yuvToRgb(String name, SDVariable input)
```
Converting image from YUV to RGB format 

* **input**  (NUMERIC type) - 3D image


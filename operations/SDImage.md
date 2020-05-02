---
title: Image
short_title: Image
description: 
category: Operations
weight: 30
---
# Image Namespace
# Operation classes
## <a name="CropAndResize">CropAndResize</a>
```JAVA
INDArray CropAndResize(INDArray image, INDArray cropBoxes, INDArray boxIndices, INDArray cropOutSize, double extrapolationValue)

SDVariable CropAndResize(SDVariable image, SDVariable cropBoxes, SDVariable boxIndices, SDVariable cropOutSize, double extrapolationValue)
SDVariable CropAndResize(String name, SDVariable image, SDVariable cropBoxes, SDVariable boxIndices, SDVariable cropOutSize, double extrapolationValue)
INDArray CropAndResize(INDArray image, INDArray cropBoxes, INDArray boxIndices, INDArray cropOutSize)

SDVariable CropAndResize(SDVariable image, SDVariable cropBoxes, SDVariable boxIndices, SDVariable cropOutSize)
SDVariable CropAndResize(String name, SDVariable image, SDVariable cropBoxes, SDVariable boxIndices, SDVariable cropOutSize)
```
Given an input image and some crop boxes, extract out the image subsets and resize them to the specified size.

* **image** - Input image, with shape [batch, height, width, channels] (NUMERIC type)
* **cropBoxes** - Float32 crop, shape [numBoxes, 4] with values in range 0 to 1 (NUMERIC type)
* **boxIndices** - Indices: which image (index to dimension 0) the cropBoxes belong to. Rank 1, shape [numBoxes] (NUMERIC type)
* **cropOutSize** - Output size for the images - int32, rank 1 with values [outHeight, outWidth] (INT type)
* **extrapolationValue** - Used for extrapolation, when applicable. 0.0 should be used for the default - default = 0.0
* **image** - Input image, with shape [batch, height, width, channels] (NUMERIC type)
* **cropBoxes** - Float32 crop, shape [numBoxes, 4] with values in range 0 to 1 (NUMERIC type)
* **boxIndices** - Indices: which image (index to dimension 0) the cropBoxes belong to. Rank 1, shape [numBoxes] (NUMERIC type)
* **cropOutSize** - Output size for the images - int32, rank 1 with values [outHeight, outWidth] (INT type)

## <a name="adjustContrast">adjustContrast</a>
```JAVA
INDArray adjustContrast(INDArray in, double factor)

SDVariable adjustContrast(SDVariable in, double factor)
SDVariable adjustContrast(String name, SDVariable in, double factor)
```
Adjusts contrast of RGB or grayscale images.

* **in** - images to adjust. 3D shape or higher (NUMERIC type)
* **factor** - multiplier for adjusting contrast

## <a name="adjustHue">adjustHue</a>
```JAVA
INDArray adjustHue(INDArray in, double delta)

SDVariable adjustHue(SDVariable in, double delta)
SDVariable adjustHue(String name, SDVariable in, double delta)
```
Adjust hue of RGB image 

* **in** - image as 3D array (NUMERIC type)
* **delta** - value to add to hue channel

## <a name="adjustSaturation">adjustSaturation</a>
```JAVA
INDArray adjustSaturation(INDArray in, double factor)

SDVariable adjustSaturation(SDVariable in, double factor)
SDVariable adjustSaturation(String name, SDVariable in, double factor)
```
Adjust saturation of RGB images

* **in** - RGB image as 3D array (NUMERIC type)
* **factor** - factor for saturation

## <a name="extractImagePatches">extractImagePatches</a>
```JAVA
INDArray extractImagePatches(INDArray image, int[] kSizes, int[] strides, int[] rates, boolean sameMode)

SDVariable extractImagePatches(SDVariable image, int[] kSizes, int[] strides, int[] rates, boolean sameMode)
SDVariable extractImagePatches(String name, SDVariable image, int[] kSizes, int[] strides, int[] rates, boolean sameMode)
```
Given an input image, extract out image patches (of size kSizes - h x w) and place them in the depth dimension. 

* **image** - Input image to extract image patches from - shape [batch, height, width, channels] (NUMERIC type)
* **kSizes** - Kernel size - size of the image patches, [height, width] (Size: Exactly(count=2)
* **strides** - Stride in the input dimension for extracting image patches, [stride_height, stride_width] (Size: Exactly(count=2)
* **rates** - Usually [1,1]. Equivalent to dilation rate in dilated convolutions - how far apart the output pixels
                 in the patches should be, in the input. A dilation of [a,b] means every {@code a}th pixel is taken
                 along the height/rows dimension, and every {@code b}th pixel is take along the width/columns dimension (Size: AtLeast(min=0)
* **sameMode** - Padding algorithm. If true: use Same padding

## <a name="hsvToRgb">hsvToRgb</a>
```JAVA
INDArray hsvToRgb(INDArray input)

SDVariable hsvToRgb(SDVariable input)
SDVariable hsvToRgb(String name, SDVariable input)
```
Converting image from HSV to RGB format 

* **input** - 3D image (NUMERIC type)

## <a name="nonMaxSuppression">nonMaxSuppression</a>
```JAVA
INDArray nonMaxSuppression(INDArray boxes, INDArray scores, int maxOutSize, double iouThreshold, double scoreThreshold)

SDVariable nonMaxSuppression(SDVariable boxes, SDVariable scores, int maxOutSize, double iouThreshold, double scoreThreshold)
SDVariable nonMaxSuppression(String name, SDVariable boxes, SDVariable scores, int maxOutSize, double iouThreshold, double scoreThreshold)
```
Greedily selects a subset of bounding boxes in descending order of score

* **boxes** - Might be null. Name for the output variable (NUMERIC type)
* **scores** - vector of shape [num_boxes] (NUMERIC type)
* **maxOutSize** - scalar representing the maximum number of boxes to be selected
* **iouThreshold** - threshold for deciding whether boxes overlap too much with respect to IOU
* **scoreThreshold** - threshold for deciding when to remove boxes based on score

## <a name="randomCrop">randomCrop</a>
```JAVA
INDArray randomCrop(INDArray input, INDArray shape)

SDVariable randomCrop(SDVariable input, SDVariable shape)
SDVariable randomCrop(String name, SDVariable input, SDVariable shape)
```
Randomly crops image

* **input** - input array (NUMERIC type)
* **shape** - shape for crop (INT type)

## <a name="rgbToHsv">rgbToHsv</a>
```JAVA
INDArray rgbToHsv(INDArray input)

SDVariable rgbToHsv(SDVariable input)
SDVariable rgbToHsv(String name, SDVariable input)
```
Converting array from HSV to RGB format

* **input** - 3D image (NUMERIC type)

## <a name="rgbToYiq">rgbToYiq</a>
```JAVA
INDArray rgbToYiq(INDArray input)

SDVariable rgbToYiq(SDVariable input)
SDVariable rgbToYiq(String name, SDVariable input)
```
Converting array from RGB to YIQ format 

* **input** - 3D image (NUMERIC type)

## <a name="rgbToYuv">rgbToYuv</a>
```JAVA
INDArray rgbToYuv(INDArray input)

SDVariable rgbToYuv(SDVariable input)
SDVariable rgbToYuv(String name, SDVariable input)
```
Converting array from RGB to YUV format 

* **input** - 3D image (NUMERIC type)

## <a name="yiqToRgb">yiqToRgb</a>
```JAVA
INDArray yiqToRgb(INDArray input)

SDVariable yiqToRgb(SDVariable input)
SDVariable yiqToRgb(String name, SDVariable input)
```
Converting image from YIQ to RGB format 

* **input** - 3D image (NUMERIC type)

## <a name="yuvToRgb">yuvToRgb</a>
```JAVA
INDArray yuvToRgb(INDArray input)

SDVariable yuvToRgb(SDVariable input)
SDVariable yuvToRgb(String name, SDVariable input)
```
Converting image from YUV to RGB format 

* **input** - 3D image (NUMERIC type)


---
title: Image
short_title: Image
description: null
category: Operations
weight: 30
---

# Image

## CropAndResize

```java
INDArray CropAndResize(INDArray image, INDArray cropBoxes, INDArray boxIndices, INDArray cropOutSize, double extrapolationValue)
INDArray CropAndResize(INDArray image, INDArray cropBoxes, INDArray boxIndices, INDArray cropOutSize)

SDVariable CropAndResize(SDVariable image, SDVariable cropBoxes, SDVariable boxIndices, SDVariable cropOutSize, double extrapolationValue)
SDVariable CropAndResize(SDVariable image, SDVariable cropBoxes, SDVariable boxIndices, SDVariable cropOutSize)
SDVariable CropAndResize(String name, SDVariable image, SDVariable cropBoxes, SDVariable boxIndices, SDVariable cropOutSize, double extrapolationValue)
SDVariable CropAndResize(String name, SDVariable image, SDVariable cropBoxes, SDVariable boxIndices, SDVariable cropOutSize)
```

Given an input image and some crop boxes, extract out the image subsets and resize them to the specified size.

* **image**  \(NUMERIC\) - Input image, with shape \[batch, height, width, channels\]
* **cropBoxes**  \(NUMERIC\) - Float32 crop, shape \[numBoxes, 4\] with values in range 0 to 1
* **boxIndices**  \(NUMERIC\) - Indices: which image \(index to dimension 0\) the cropBoxes belong to. Rank 1, shape \[numBoxes\]
* **cropOutSize**  \(INT\) - Output size for the images - int32, rank 1 with values \[outHeight, outWidth\]
* **extrapolationValue** - Used for extrapolation, when applicable. 0.0 should be used for the default - default = 0.0

## adjustContrast

```java
INDArray adjustContrast(INDArray in, double factor)

SDVariable adjustContrast(SDVariable in, double factor)
SDVariable adjustContrast(String name, SDVariable in, double factor)
```

Adjusts contrast of RGB or grayscale images.

* **in**  \(NUMERIC\) - images to adjust. 3D shape or higher
* **factor** - multiplier for adjusting contrast

## adjustHue

```java
INDArray adjustHue(INDArray in, double delta)

SDVariable adjustHue(SDVariable in, double delta)
SDVariable adjustHue(String name, SDVariable in, double delta)
```

Adjust hue of RGB image

* **in**  \(NUMERIC\) - image as 3D array
* **delta** - value to add to hue channel

## adjustSaturation

```java
INDArray adjustSaturation(INDArray in, double factor)

SDVariable adjustSaturation(SDVariable in, double factor)
SDVariable adjustSaturation(String name, SDVariable in, double factor)
```

Adjust saturation of RGB images

* **in**  \(NUMERIC\) - RGB image as 3D array
* **factor** - factor for saturation

## extractImagePatches

```java
INDArray extractImagePatches(INDArray image, int[] kSizes, int[] strides, int[] rates, boolean sameMode)

SDVariable extractImagePatches(SDVariable image, int[] kSizes, int[] strides, int[] rates, boolean sameMode)
SDVariable extractImagePatches(String name, SDVariable image, int[] kSizes, int[] strides, int[] rates, boolean sameMode)
```

Given an input image, extract out image patches \(of size kSizes - h x w\) and place them in the depth dimension.

* **image**  \(NUMERIC\) - Input image to extract image patches from - shape \[batch, height, width, channels\]
* **kSizes** - Kernel size - size of the image patches, \[height, width\] \(Size: Exactly\(count=2\)\)
* **strides** - Stride in the input dimension for extracting image patches, \[stride\_height, stride\_width\] \(Size: Exactly\(count=2\)\)
* **rates** - Usually \[1,1\]. Equivalent to dilation rate in dilated convolutions - how far apart the output pixels

  ```text
             in the patches should be, in the input. A dilation of [a,b] means every {@code a}th pixel is taken
             along the height/rows dimension, and every {@code b}th pixel is take along the width/columns dimension (Size: AtLeast(min=0))
  ```

* **sameMode** - Padding algorithm. If true: use Same padding

## hsvToRgb

```java
INDArray hsvToRgb(INDArray input)

SDVariable hsvToRgb(SDVariable input)
SDVariable hsvToRgb(String name, SDVariable input)
```

Converting image from HSV to RGB format

* **input**  \(NUMERIC\) - 3D image

## imageResize

```java
INDArray imageResize(INDArray input, INDArray size, boolean preserveAspectRatio, boolean antialis, ImageResizeMethod ImageResizeMethod)
INDArray imageResize(INDArray input, INDArray size, ImageResizeMethod ImageResizeMethod)

SDVariable imageResize(SDVariable input, SDVariable size, boolean preserveAspectRatio, boolean antialis, ImageResizeMethod ImageResizeMethod)
SDVariable imageResize(SDVariable input, SDVariable size, ImageResizeMethod ImageResizeMethod)
SDVariable imageResize(String name, SDVariable input, SDVariable size, boolean preserveAspectRatio, boolean antialis, ImageResizeMethod ImageResizeMethod)
SDVariable imageResize(String name, SDVariable input, SDVariable size, ImageResizeMethod ImageResizeMethod)
```

Resize images to size using the specified method.

* **input**  \(NUMERIC\) - 4D image \[NHWC\]
* **size**  \(INT\) - new height and width
* **preserveAspectRatio** - Whether to preserve the aspect ratio. If this is set, then images will be resized to a size that fits in size while preserving the aspect ratio of the original image. Scales up the image if size is bigger than the current size of the image. Defaults to False. - default = false
* **antialis** - Whether to use an anti-aliasing filter when downsampling an image - default = false
* **ImageResizeMethod** - ResizeBilinear: Bilinear interpolation. If 'antialias' is true, becomes a hat/tent filter function with radius 1 when downsampling.

  ResizeLanczos5: Lanczos kernel with radius 5. Very-high-quality filter but may have stronger ringing.

  ResizeBicubic: Cubic interpolant of Keys. Equivalent to Catmull-Rom kernel. Reasonably good quality and faster than Lanczos3Kernel, particularly when upsampling.

  ResizeGaussian: Gaussian kernel with radius 3, sigma = 1.5 / 3.0.

  ResizeNearest: Nearest neighbor interpolation. 'antialias' has no effect when used with nearest neighbor interpolation.

  ResizeArea: Anti-aliased resampling with area interpolation. 'antialias' has no effect when used with area interpolation; it always anti-aliases.

  ResizeMitchelcubic: Mitchell-Netravali Cubic non-interpolating filter. For synthetic images \(especially those lacking proper prefiltering\), less ringing than Keys cubic kernel but less sharp.

## nonMaxSuppression

```java
INDArray nonMaxSuppression(INDArray boxes, INDArray scores, int maxOutSize, double iouThreshold, double scoreThreshold)

SDVariable nonMaxSuppression(SDVariable boxes, SDVariable scores, int maxOutSize, double iouThreshold, double scoreThreshold)
SDVariable nonMaxSuppression(String name, SDVariable boxes, SDVariable scores, int maxOutSize, double iouThreshold, double scoreThreshold)
```

Greedily selects a subset of bounding boxes in descending order of score

* **boxes**  \(NUMERIC\) - Might be null. Name for the output variable
* **scores**  \(NUMERIC\) - vector of shape \[num\_boxes\]
* **maxOutSize** - scalar representing the maximum number of boxes to be selected
* **iouThreshold** - threshold for deciding whether boxes overlap too much with respect to IOU
* **scoreThreshold** - threshold for deciding when to remove boxes based on score

## randomCrop

```java
INDArray randomCrop(INDArray input, INDArray shape)

SDVariable randomCrop(SDVariable input, SDVariable shape)
SDVariable randomCrop(String name, SDVariable input, SDVariable shape)
```

Randomly crops image

* **input**  \(NUMERIC\) - input array
* **shape**  \(INT\) - shape for crop

## rgbToHsv

```java
INDArray rgbToHsv(INDArray input)

SDVariable rgbToHsv(SDVariable input)
SDVariable rgbToHsv(String name, SDVariable input)
```

Converting array from HSV to RGB format

* **input**  \(NUMERIC\) - 3D image

## rgbToYiq

```java
INDArray rgbToYiq(INDArray input)

SDVariable rgbToYiq(SDVariable input)
SDVariable rgbToYiq(String name, SDVariable input)
```

Converting array from RGB to YIQ format

* **input**  \(NUMERIC\) - 3D image

## rgbToYuv

```java
INDArray rgbToYuv(INDArray input)

SDVariable rgbToYuv(SDVariable input)
SDVariable rgbToYuv(String name, SDVariable input)
```

Converting array from RGB to YUV format

* **input**  \(NUMERIC\) - 3D image

## yiqToRgb

```java
INDArray yiqToRgb(INDArray input)

SDVariable yiqToRgb(SDVariable input)
SDVariable yiqToRgb(String name, SDVariable input)
```

Converting image from YIQ to RGB format

* **input**  \(NUMERIC\) - 3D image

## yuvToRgb

```java
INDArray yuvToRgb(INDArray input)

SDVariable yuvToRgb(SDVariable input)
SDVariable yuvToRgb(String name, SDVariable input)
```

Converting image from YUV to RGB format

* **input**  \(NUMERIC\) - 3D image


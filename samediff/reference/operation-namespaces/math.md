---
title: Math
short_title: Math
description: null
category: Operations
weight: 60
---

# Math

## ClipByAvgNorm

```java
INDArray ClipByAvgNorm(INDArray x, double clipValue, int[] dimensions)

SDVariable ClipByAvgNorm(SDVariable x, double clipValue, int[] dimensions)
SDVariable ClipByAvgNorm(String name, SDVariable x, double clipValue, int[] dimensions)
```

Clips tensor values to a maximum average L2-norm.

* **x**  \(NUMERIC\) - Input variable
* **clipValue** - Value for clipping
* **dimensions** - Dimensions to reduce over \(Size: AtLeast\(min=0\)\)

## EmbeddingLookup

```java
INDArray EmbeddingLookup(INDArray x, INDArray indices, PartitionMode PartitionMode)

SDVariable EmbeddingLookup(SDVariable x, SDVariable indices, PartitionMode PartitionMode)
SDVariable EmbeddingLookup(String name, SDVariable x, SDVariable indices, PartitionMode PartitionMode)
```

Looks up ids in a list of embedding tensors.

* **x**  \(NUMERIC\) - Input tensor
* **indices**  \(INT\) - A Tensor containing the ids to be looked up.
* **PartitionMode** - partition\_mode == 0 - i.e. 'mod' , 1 - 'div'

## MergeMaxIndex

```java
INDArray MergeMaxIndex(INDArray x, DataType dataType)
INDArray MergeMaxIndex(INDArray x)

SDVariable MergeMaxIndex(SDVariable x, DataType dataType)
SDVariable MergeMaxIndex(SDVariable x)
SDVariable MergeMaxIndex(String name, SDVariable x, DataType dataType)
SDVariable MergeMaxIndex(String name, SDVariable x)
```

Return array of max elements indices with along tensor dimensions

* **x**  \(NUMERIC\) - Input tensor
* **dataType** - Data type - default = DataType.INT

## abs

```java
INDArray abs(INDArray x)

SDVariable abs(SDVariable x)
SDVariable abs(String name, SDVariable x)
```

Elementwise absolute value operation: out = abs\(x\)

* **x**  \(NUMERIC\) - Input variable

## acos

```java
INDArray acos(INDArray x)

SDVariable acos(SDVariable x)
SDVariable acos(String name, SDVariable x)
```

Elementwise acos \(arccosine, inverse cosine\) operation: out = arccos\(x\)

* **x**  \(NUMERIC\) - Input variable

## acosh

```java
INDArray acosh(INDArray x)

SDVariable acosh(SDVariable x)
SDVariable acosh(String name, SDVariable x)
```

Elementwise acosh \(inverse hyperbolic cosine\) function: out = acosh\(x\)

* **x**  \(NUMERIC\) - Input variable

## add

```java
INDArray add(INDArray x, INDArray y)

SDVariable add(SDVariable x, SDVariable y)
SDVariable add(String name, SDVariable x, SDVariable y)
```

Pairwise addition operation, out = x + y

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape \[1,10\] and Y has shape \[5,10\] then op\(X,Y\) has output shape \[5,10\]

Broadcast rules are the same as NumPy: [https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

* **x**  \(NUMERIC\) - Input variable
* **y**  \(NUMERIC\) - Input variable

## add

```java
INDArray add(INDArray x, double value)

SDVariable add(SDVariable x, double value)
SDVariable add(String name, SDVariable x, double value)
```

Scalar add operation, out = in + scalar

* **x**  \(NUMERIC\) - Input variable
* **value** - Scalar value for op

## amax

```java
INDArray amax(INDArray in, int[] dimensions)

SDVariable amax(SDVariable in, int[] dimensions)
SDVariable amax(String name, SDVariable in, int[] dimensions)
```

Absolute max array reduction operation, optionally along specified dimensions: out = max\(abs\(x\)\)

* **in**  \(NUMERIC\) - Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=0\)\)

## amean

```java
INDArray amean(INDArray in, int[] dimensions)

SDVariable amean(SDVariable in, int[] dimensions)
SDVariable amean(String name, SDVariable in, int[] dimensions)
```

Absolute mean array reduction operation, optionally along specified dimensions: out = mean\(abs\(x\)\)

* **in**  \(NUMERIC\) - Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=0\)\)

## amin

```java
INDArray amin(INDArray in, int[] dimensions)

SDVariable amin(SDVariable in, int[] dimensions)
SDVariable amin(String name, SDVariable in, int[] dimensions)
```

Absolute min array reduction operation, optionally along specified dimensions: out = min\(abs\(x\)\)

* **in**  \(NUMERIC\) - Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=0\)\)

## and

```java
INDArray and(INDArray x, INDArray y)

SDVariable and(SDVariable x, SDVariable y)
SDVariable and(String name, SDVariable x, SDVariable y)
```

Boolean AND operation: elementwise \(x != 0\) && \(y != 0\)

If x and y arrays have equal shape, the output shape is the same as these inputs.

Note: supports broadcasting if x and y have different shapes and are broadcastable.

Returns an array with values 1 where condition is satisfied, or value 0 otherwise.

* **x**  \(BOOL\) - Input 1
* **y**  \(BOOL\) - Input 2

## asin

```java
INDArray asin(INDArray x)

SDVariable asin(SDVariable x)
SDVariable asin(String name, SDVariable x)
```

Elementwise asin \(arcsin, inverse sine\) operation: out = arcsin\(x\)

* **x**  \(NUMERIC\) - Input variable

## asinh

```java
INDArray asinh(INDArray x)

SDVariable asinh(SDVariable x)
SDVariable asinh(String name, SDVariable x)
```

Elementwise asinh \(inverse hyperbolic sine\) function: out = asinh\(x\)

* **x**  \(NUMERIC\) - Input variable

## asum

```java
INDArray asum(INDArray in, int[] dimensions)

SDVariable asum(SDVariable in, int[] dimensions)
SDVariable asum(String name, SDVariable in, int[] dimensions)
```

Absolute sum array reduction operation, optionally along specified dimensions: out = sum\(abs\(x\)\)

* **in**  \(NUMERIC\) - Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=0\)\)

## atan

```java
INDArray atan(INDArray x)

SDVariable atan(SDVariable x)
SDVariable atan(String name, SDVariable x)
```

Elementwise atan \(arctangent, inverse tangent\) operation: out = arctangent\(x\)

* **x**  \(NUMERIC\) - Input variable

## atan2

```java
INDArray atan2(INDArray y, INDArray x)

SDVariable atan2(SDVariable y, SDVariable x)
SDVariable atan2(String name, SDVariable y, SDVariable x)
```

Elementwise atan \(arctangent, inverse tangent\) operation: out = atan2\(x,y\).

Similar to atan\(y/x\) but sigts of x and y are used to determine the location of the result

* **y**  \(NUMERIC\) - Input Y variable
* **x**  \(NUMERIC\) - Input X variable

## atanh

```java
INDArray atanh(INDArray x)

SDVariable atanh(SDVariable x)
SDVariable atanh(String name, SDVariable x)
```

Elementwise atanh \(inverse hyperbolic tangent\) function: out = atanh\(x\)

* **x**  \(NUMERIC\) - Input variable

## bitShift

```java
INDArray bitShift(INDArray x, INDArray shift)

SDVariable bitShift(SDVariable x, SDVariable shift)
SDVariable bitShift(String name, SDVariable x, SDVariable shift)
```

Bit shift operation

* **x**  \(NUMERIC\) - input
* **shift**  \(NUMERIC\) - shift value

## bitShiftRight

```java
INDArray bitShiftRight(INDArray x, INDArray shift)

SDVariable bitShiftRight(SDVariable x, SDVariable shift)
SDVariable bitShiftRight(String name, SDVariable x, SDVariable shift)
```

Right bit shift operation

* **x**  \(NUMERIC\) - Input tensor
* **shift**  \(NUMERIC\) - shift argument

## bitShiftRotl

```java
INDArray bitShiftRotl(INDArray x, INDArray shift)

SDVariable bitShiftRotl(SDVariable x, SDVariable shift)
SDVariable bitShiftRotl(String name, SDVariable x, SDVariable shift)
```

Cyclic bit shift operation

* **x**  \(NUMERIC\) - Input tensor
* **shift**  \(NUMERIC\) - shift argy=ument

## bitShiftRotr

```java
INDArray bitShiftRotr(INDArray x, INDArray shift)

SDVariable bitShiftRotr(SDVariable x, SDVariable shift)
SDVariable bitShiftRotr(String name, SDVariable x, SDVariable shift)
```

Cyclic right shift operation

* **x**  \(NUMERIC\) - Input tensor
* **shift**  \(NUMERIC\) - Shift argument

## ceil

```java
INDArray ceil(INDArray x)

SDVariable ceil(SDVariable x)
SDVariable ceil(String name, SDVariable x)
```

Element-wise ceiling function: out = ceil\(x\).

Rounds each value up to the nearest integer value \(if not already an integer\)

* **x**  \(NUMERIC\) - Input variable

## clipByNorm

```java
INDArray clipByNorm(INDArray x, double clipValue, int[] dimensions)

SDVariable clipByNorm(SDVariable x, double clipValue, int[] dimensions)
SDVariable clipByNorm(String name, SDVariable x, double clipValue, int[] dimensions)
```

Clipping by L2 norm, optionally along dimension\(s\)

if l2Norm\(x,dimension\) &lt; clipValue, then input is returned unmodifed

Otherwise, out\[i\] = in\[i\] \* clipValue / l2Norm\(in, dimensions\) where each value is clipped according

to the corresponding l2Norm along the specified dimensions

* **x**  \(NUMERIC\) - Input variable
* **clipValue** - Clipping value \(maximum l2 norm\)
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=0\)\)

## clipByValue

```java
INDArray clipByValue(INDArray x, double clipValueMin, double clipValueMax)

SDVariable clipByValue(SDVariable x, double clipValueMin, double clipValueMax)
SDVariable clipByValue(String name, SDVariable x, double clipValueMin, double clipValueMax)
```

Element-wise clipping function:

out\[i\] = in\[i\] if in\[i\] &gt;= clipValueMin and in\[i\] &lt;= clipValueMax

out\[i\] = clipValueMin if in\[i\] &lt; clipValueMin

out\[i\] = clipValueMax if in\[i\] &gt; clipValueMax

* **x**  \(NUMERIC\) - Input variable
* **clipValueMin** - Minimum value for clipping
* **clipValueMax** - Maximum value for clipping

## confusionMatrix

```java
INDArray confusionMatrix(INDArray labels, INDArray pred, DataType dataType)

SDVariable confusionMatrix(SDVariable labels, SDVariable pred, DataType dataType)
SDVariable confusionMatrix(String name, SDVariable labels, SDVariable pred, DataType dataType)
```

Compute the 2d confusion matrix of size \[numClasses, numClasses\] from a pair of labels and predictions, both of

which are represented as integer values. This version assumes the number of classes is 1 + max\(max\(labels\), max\(pred\)\)

For example, if labels = \[0, 1, 1\] and predicted = \[0, 2, 1\] then output is:

\[1, 0, 0\]

\[0, 1, 1\]

\[0, 0, 0\]

* **labels**  \(NUMERIC\) - Labels - 1D array of integer values representing label values
* **pred**  \(NUMERIC\) - Predictions - 1D array of integer values representing predictions. Same length as labels
* **dataType** - Data type

## confusionMatrix

```java
INDArray confusionMatrix(INDArray labels, INDArray pred, int numClasses)

SDVariable confusionMatrix(SDVariable labels, SDVariable pred, int numClasses)
SDVariable confusionMatrix(String name, SDVariable labels, SDVariable pred, int numClasses)
```

Compute the 2d confusion matrix of size \[numClasses, numClasses\] from a pair of labels and predictions, both of

which are represented as integer values.

For example, if labels = \[0, 1, 1\], predicted = \[0, 2, 1\], and numClasses=4 then output is:

\[1, 0, 0, 0\]

\[0, 1, 1, 0\]

\[0, 0, 0, 0\]

\[0, 0, 0, 0\]

* **labels**  \(NUMERIC\) - Labels - 1D array of integer values representing label values
* **pred**  \(NUMERIC\) - Predictions - 1D array of integer values representing predictions. Same length as labels
* **numClasses** - Number of classes

## confusionMatrix

```java
INDArray confusionMatrix(INDArray labels, INDArray pred, INDArray weights)

SDVariable confusionMatrix(SDVariable labels, SDVariable pred, SDVariable weights)
SDVariable confusionMatrix(String name, SDVariable labels, SDVariable pred, SDVariable weights)
```

Compute the 2d confusion matrix of size \[numClasses, numClasses\] from a pair of labels and predictions, both of

which are represented as integer values. This version assumes the number of classes is 1 + max\(max\(labels\), max\(pred\)\)

For example, if labels = \[0, 1, 1\], predicted = \[0, 2, 1\] and weights = \[1, 2, 3\]

\[1, 0, 0\]

\[0, 3, 2\]

\[0, 0, 0\]

* **labels**  \(NUMERIC\) - Labels - 1D array of integer values representing label values
* **pred**  \(NUMERIC\) - Predictions - 1D array of integer values representing predictions. Same length as labels
* **weights**  \(NUMERIC\) - Weights - 1D array of values \(may be real/decimal\) representing the weight/contribution of each prediction. Must be same length as both labels and predictions arrays

## confusionMatrix

```java
INDArray confusionMatrix(INDArray labels, INDArray pred, INDArray weights, int numClasses)

SDVariable confusionMatrix(SDVariable labels, SDVariable pred, SDVariable weights, int numClasses)
SDVariable confusionMatrix(String name, SDVariable labels, SDVariable pred, SDVariable weights, int numClasses)
```

Compute the 2d confusion matrix of size \[numClasses, numClasses\] from a pair of labels and predictions, both of

which are represented as integer values.

For example, if labels = \[0, 1, 1\], predicted = \[0, 2, 1\], numClasses = 4, and weights = \[1, 2, 3\]

\[1, 0, 0, 0\]

\[0, 3, 2, 0\]

\[0, 0, 0, 0\]

\[0, 0, 0, 0\]

* **labels**  \(NUMERIC\) - Labels - 1D array of integer values representing label values
* **pred**  \(NUMERIC\) - Predictions - 1D array of integer values representing predictions. Same length as labels
* **weights**  \(NUMERIC\) - Weights - 1D array of values \(may be real/decimal\) representing the weight/contribution of each prediction. Must be same length as both labels and predictions arrays
* **numClasses** - 

## cos

```java
INDArray cos(INDArray x)

SDVariable cos(SDVariable x)
SDVariable cos(String name, SDVariable x)
```

Elementwise cosine operation: out = cos\(x\)

* **x**  \(NUMERIC\) - Input variable

## cosh

```java
INDArray cosh(INDArray x)

SDVariable cosh(SDVariable x)
SDVariable cosh(String name, SDVariable x)
```

Elementwise cosh \(hyperbolic cosine\) operation: out = cosh\(x\)

* **x**  \(NUMERIC\) - Input variable

## cosineDistance

```java
INDArray cosineDistance(INDArray x, INDArray y, int[] dimensions)

SDVariable cosineDistance(SDVariable x, SDVariable y, int[] dimensions)
SDVariable cosineDistance(String name, SDVariable x, SDVariable y, int[] dimensions)
```

Cosine distance reduction operation. The output contains the cosine distance for each

tensor/subset along the specified dimensions:

out = 1.0 - cosineSimilarity\(x,y\)

* **x**  \(NUMERIC\) - Input variable x
* **y**  \(NUMERIC\) - Input variable y
* **dimensions** - Dimensions to calculate cosineDistance over \(Size: AtLeast\(min=0\)\)

## cosineSimilarity

```java
INDArray cosineSimilarity(INDArray x, INDArray y, int[] dimensions)

SDVariable cosineSimilarity(SDVariable x, SDVariable y, int[] dimensions)
SDVariable cosineSimilarity(String name, SDVariable x, SDVariable y, int[] dimensions)
```

Cosine similarity pairwise reduction operation. The output contains the cosine similarity for each tensor/subset

along the specified dimensions:

out = \(sum\_i x\[i\] _y\[i\]\) / \( sqrt\(sum\_i x\[i\]^2\)_ sqrt\(sum\_i y\[i\]^2\)

* **x**  \(NUMERIC\) - Input variable x
* **y**  \(NUMERIC\) - Input variable y
* **dimensions** - Dimensions to calculate cosineSimilarity over \(Size: AtLeast\(min=0\)\)

## countNonZero

```java
INDArray countNonZero(INDArray in, int[] dimensions)

SDVariable countNonZero(SDVariable in, int[] dimensions)
SDVariable countNonZero(String name, SDVariable in, int[] dimensions)
```

Count non zero array reduction operation, optionally along specified dimensions: out = count\(x != 0\)

* **in**  \(NUMERIC\) - Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=0\)\)

## countZero

```java
INDArray countZero(INDArray in, int[] dimensions)

SDVariable countZero(SDVariable in, int[] dimensions)
SDVariable countZero(String name, SDVariable in, int[] dimensions)
```

Count zero array reduction operation, optionally along specified dimensions: out = count\(x == 0\)

* **in**  \(NUMERIC\) - Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=0\)\)

## cross

```java
INDArray cross(INDArray a, INDArray b)

SDVariable cross(SDVariable a, SDVariable b)
SDVariable cross(String name, SDVariable a, SDVariable b)
```

Returns the pair-wise cross product of equal size arrays a and b: a x b = \|\|a\|\|x\|\|b\|\| sin\(theta\).

Can take rank 1 or above inputs \(of equal shapes\), but note that the last dimension must have dimension 3

* **a**  \(NUMERIC\) - First input
* **b**  \(NUMERIC\) - Second input

## cube

```java
INDArray cube(INDArray x)

SDVariable cube(SDVariable x)
SDVariable cube(String name, SDVariable x)
```

Element-wise cube function: out = x^3

* **x**  \(NUMERIC\) - Input variable

## diag

```java
INDArray diag(INDArray x)

SDVariable diag(SDVariable x)
SDVariable diag(String name, SDVariable x)
```

Returns an output variable with diagonal values equal to the specified values; off-diagonal values will be set to 0

For example, if input = \[1,2,3\], then output is given by:

\[ 1, 0, 0\]

\[ 0, 2, 0\]

\[ 0, 0, 3\]

Higher input ranks are also supported: if input has shape \[a,...,R-1\] then output\[i,...,k,i,...,k\] = input\[i,...,k\].

i.e., for input rank R, output has rank 2R

* **x**  \(NUMERIC\) - Input variable

## diagPart

```java
INDArray diagPart(INDArray x)

SDVariable diagPart(SDVariable x)
SDVariable diagPart(String name, SDVariable x)
```

Extract the diagonal part from the input array.

If input is

\[ 1, 0, 0\]

\[ 0, 2, 0\]

\[ 0, 0, 3\]

then output is \[1, 2, 3\].

Supports higher dimensions: in general, out\[i,...,k\] = in\[i,...,k,i,...,k\]

* **x**  \(NUMERIC\) - Input variable

## div

```java
INDArray div(INDArray x, INDArray y)

SDVariable div(SDVariable x, SDVariable y)
SDVariable div(String name, SDVariable x, SDVariable y)
```

Pairwise division operation, out = x / y

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape \[1,10\] and Y has shape \[5,10\] then op\(X,Y\) has output shape \[5,10\]

Broadcast rules are the same as NumPy: [https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

* **x**  \(NUMERIC\) - Input variable
* **y**  \(NUMERIC\) - Input variable

## div

```java
INDArray div(INDArray x, double value)

SDVariable div(SDVariable x, double value)
SDVariable div(String name, SDVariable x, double value)
```

Scalar division operation, out = in / scalar

* **x**  \(NUMERIC\) - Input variable
* **value** - Scalar value for op

## entropy

```java
INDArray entropy(INDArray in, int[] dimensions)

SDVariable entropy(SDVariable in, int[] dimensions)
SDVariable entropy(String name, SDVariable in, int[] dimensions)
```

Entropy reduction: -sum\(x \* log\(x\)\)

* **in**  \(NUMERIC\) - Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=0\)\)

## erf

```java
INDArray erf(INDArray x)

SDVariable erf(SDVariable x)
SDVariable erf(String name, SDVariable x)
```

Element-wise Gaussian error function - out = erf\(in\)

* **x**  \(NUMERIC\) - Input variable

## erfc

```java
INDArray erfc(INDArray x)

SDVariable erfc(SDVariable x)
SDVariable erfc(String name, SDVariable x)
```

Element-wise complementary Gaussian error function - out = erfc\(in\) = 1 - erf\(in\)

* **x**  \(NUMERIC\) - Input variable

## euclideanDistance

```java
INDArray euclideanDistance(INDArray x, INDArray y, int[] dimensions)

SDVariable euclideanDistance(SDVariable x, SDVariable y, int[] dimensions)
SDVariable euclideanDistance(String name, SDVariable x, SDVariable y, int[] dimensions)
```

Euclidean distance \(l2 norm, l2 distance\) reduction operation. The output contains the Euclidean distance for each

tensor/subset along the specified dimensions:

out = sqrt\( sum\_i \(x\[i\] - y\[i\]\)^2 \)

* **x**  \(NUMERIC\) - Input variable x
* **y**  \(NUMERIC\) - Input variable y
* **dimensions** - Dimensions to calculate euclideanDistance over \(Size: AtLeast\(min=0\)\)

## exp

```java
INDArray exp(INDArray x)

SDVariable exp(SDVariable x)
SDVariable exp(String name, SDVariable x)
```

Elementwise exponent function: out = exp\(x\) = 2.71828...^x

* **x**  \(NUMERIC\) - Input variable

## expm1

```java
INDArray expm1(INDArray x)

SDVariable expm1(SDVariable x)
SDVariable expm1(String name, SDVariable x)
```

Elementwise 1.0 - exponent function: out = 1.0 - exp\(x\) = 1.0 - 2.71828...^x

* **x**  \(NUMERIC\) - Input variable

## eye

```java
INDArray eye(int rows)

SDVariable eye(int rows)
SDVariable eye(String name, int rows)
```

Generate an identity matrix with the specified number of rows and columns.

* **rows** - Number of rows

## eye

```java
INDArray eye(int rows, int cols)

SDVariable eye(int rows, int cols)
SDVariable eye(String name, int rows, int cols)
```

As per eye\(String, int, int, DataType\) but with the default datatype, Eye.DEFAULT\_DTYPE

* **rows** - Number of rows
* **cols** - Number of columns

## eye

```java
INDArray eye(int rows, int cols, DataType dataType, int[] dimensions)

SDVariable eye(int rows, int cols, DataType dataType, int[] dimensions)
SDVariable eye(String name, int rows, int cols, DataType dataType, int[] dimensions)
```

Generate an identity matrix with the specified number of rows and columns

Example:

```text
`INDArray eye = eye(3,2)

eye:

[ 1, 0]

[ 0, 1]

[ 0, 0]`
```

* **rows** - Number of rows
* **cols** - Number of columns
* **dataType** - Data type
* **dimensions** -  \(Size: AtLeast\(min=0\)\)

## eye

```java
INDArray eye(INDArray rows, INDArray cols)

SDVariable eye(SDVariable rows, SDVariable cols)
SDVariable eye(String name, SDVariable rows, SDVariable cols)
```

As per eye\(int, int\) bit with the number of rows/columns specified as scalar INDArrays

* **rows**  \(INT\) - Number of rows
* **cols**  \(INT\) - Number of columns

## eye

```java
INDArray eye(INDArray rows)

SDVariable eye(SDVariable rows)
SDVariable eye(String name, SDVariable rows)
```

As per eye\(String, int\) but with the number of rows specified as a scalar INDArray

* **rows**  \(INT\) - Number of rows

## firstIndex

```java
INDArray firstIndex(INDArray in, Condition condition, int[] dimensions)
INDArray firstIndex(INDArray in, Condition condition, boolean keepDims, int[] dimensions)

SDVariable firstIndex(SDVariable in, Condition condition, int[] dimensions)
SDVariable firstIndex(SDVariable in, Condition condition, boolean keepDims, int[] dimensions)
SDVariable firstIndex(String name, SDVariable in, Condition condition, int[] dimensions)
SDVariable firstIndex(String name, SDVariable in, Condition condition, boolean keepDims, int[] dimensions)
```

First index reduction operation.

Returns a variable that contains the index of the first element that matches the specified condition \(for each

slice along the specified dimensions\)

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations \(such as subtracting

the mean along a dimension\).

Example: if input has shape \[a,b,c\] and dimensions=\[1\] then output has shape:

keepDims = true: \[a,1,c\]

keepDims = false: \[a,c\]

* **in**  \(NUMERIC\) - Input variable
* **condition** - Condition to check on input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=1\)\)
* **keepDims** - If true: keep the dimensions that are reduced on \(as length 1\). False: remove the reduction dimensions - default = false

## floor

```java
INDArray floor(INDArray x)

SDVariable floor(SDVariable x)
SDVariable floor(String name, SDVariable x)
```

Element-wise floor function: out = floor\(x\).

Rounds each value down to the nearest integer value \(if not already an integer\)

* **x**  \(NUMERIC\) - Input variable

## floorDiv

```java
INDArray floorDiv(INDArray x, INDArray y)

SDVariable floorDiv(SDVariable x, SDVariable y)
SDVariable floorDiv(String name, SDVariable x, SDVariable y)
```

Pairwise floor division operation, out = floor\(x / y\)

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape \[1,10\] and Y has shape \[5,10\] then op\(X,Y\) has output shape \[5,10\]

Broadcast rules are the same as NumPy: [https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

* **x**  \(NUMERIC\) - Input variable
* **y**  \(NUMERIC\) - Input variable

## floorMod

```java
INDArray floorMod(INDArray x, INDArray y)

SDVariable floorMod(SDVariable x, SDVariable y)
SDVariable floorMod(String name, SDVariable x, SDVariable y)
```

Pairwise Modulus division operation

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape \[1,10\] and Y has shape \[5,10\] then op\(X,Y\) has output shape \[5,10\]

Broadcast rules are the same as NumPy: [https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

* **x**  \(NUMERIC\) - Input variable
* **y**  \(NUMERIC\) - Input variable

## floorMod

```java
INDArray floorMod(INDArray x, double value)

SDVariable floorMod(SDVariable x, double value)
SDVariable floorMod(String name, SDVariable x, double value)
```

Scalar floor modulus operation

* **x**  \(NUMERIC\) - Input variable
* **value** - Scalar value for op

## hammingDistance

```java
INDArray hammingDistance(INDArray x, INDArray y, int[] dimensions)

SDVariable hammingDistance(SDVariable x, SDVariable y, int[] dimensions)
SDVariable hammingDistance(String name, SDVariable x, SDVariable y, int[] dimensions)
```

Hamming distance reduction operation. The output contains the cosine distance for each

tensor/subset along the specified dimensions:

out = count\( x\[i\] != y\[i\] \)

* **x**  \(NUMERIC\) - Input variable x
* **y**  \(NUMERIC\) - Input variable y
* **dimensions** - Dimensions to calculate hammingDistance over \(Size: AtLeast\(min=0\)\)

## iamax

```java
INDArray iamax(INDArray in, int[] dimensions)
INDArray iamax(INDArray in, boolean keepDims, int[] dimensions)

SDVariable iamax(SDVariable in, int[] dimensions)
SDVariable iamax(SDVariable in, boolean keepDims, int[] dimensions)
SDVariable iamax(String name, SDVariable in, int[] dimensions)
SDVariable iamax(String name, SDVariable in, boolean keepDims, int[] dimensions)
```

Index of the max absolute value: argmax\(abs\(in\)\)

see argmax\(String, INDArray, boolean, int...\)

* **in**  \(NUMERIC\) - Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=1\)\)
* **keepDims** - If true: keep the dimensions that are reduced on \(as length 1\). False: remove the reduction dimensions - default = false

## iamin

```java
INDArray iamin(INDArray in, int[] dimensions)
INDArray iamin(INDArray in, boolean keepDims, int[] dimensions)

SDVariable iamin(SDVariable in, int[] dimensions)
SDVariable iamin(SDVariable in, boolean keepDims, int[] dimensions)
SDVariable iamin(String name, SDVariable in, int[] dimensions)
SDVariable iamin(String name, SDVariable in, boolean keepDims, int[] dimensions)
```

Index of the min absolute value: argmin\(abs\(in\)\)

see argmin\(String, INDArray, boolean, int...\)

* **in**  \(NUMERIC\) - Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=1\)\)
* **keepDims** - If true: keep the dimensions that are reduced on \(as length 1\). False: remove the reduction dimensions - default = false

## isFinite

```java
INDArray isFinite(INDArray x)

SDVariable isFinite(SDVariable x)
SDVariable isFinite(String name, SDVariable x)
```

Is finite operation: elementwise isFinite\(x\)

Returns an array with the same shape/size as the input, with values 1 where condition is satisfied, or

value 0 otherwise

* **x**  \(NUMERIC\) - Input variable

## isInfinite

```java
INDArray isInfinite(INDArray x)

SDVariable isInfinite(SDVariable x)
SDVariable isInfinite(String name, SDVariable x)
```

Is infinite operation: elementwise isInfinite\(x\)

Returns an array with the same shape/size as the input, with values 1 where condition is satisfied, or

value 0 otherwise

* **x**  \(NUMERIC\) - Input variable

## isMax

```java
INDArray isMax(INDArray x)

SDVariable isMax(SDVariable x)
SDVariable isMax(String name, SDVariable x)
```

Is maximum operation: elementwise x == max\(x\)

Returns an array with the same shape/size as the input, with values 1 where condition is satisfied, or

value 0 otherwise

* **x**  \(NUMERIC\) - Input variable

## isNaN

```java
INDArray isNaN(INDArray x)

SDVariable isNaN(SDVariable x)
SDVariable isNaN(String name, SDVariable x)
```

Is Not a Number operation: elementwise isNaN\(x\)

Returns an array with the same shape/size as the input, with values 1 where condition is satisfied, or

value 0 otherwise

* **x**  \(NUMERIC\) - Input variable

## isNonDecreasing

```java
INDArray isNonDecreasing(INDArray x)

SDVariable isNonDecreasing(SDVariable x)
SDVariable isNonDecreasing(String name, SDVariable x)
```

Is the array non decreasing?

An array is non-decreasing if for every valid i, x\[i\] &lt;= x\[i+1\]. For Rank 2+ arrays, values are compared

in 'c' \(row major\) order

* **x**  \(NUMERIC\) - Input variable

## isStrictlyIncreasing

```java
INDArray isStrictlyIncreasing(INDArray x)

SDVariable isStrictlyIncreasing(SDVariable x)
SDVariable isStrictlyIncreasing(String name, SDVariable x)
```

Is the array strictly increasing?

An array is strictly increasing if for every valid i, x\[i\] &lt; x\[i+1\]. For Rank 2+ arrays, values are compared

in 'c' \(row major\) order

* **x**  \(NUMERIC\) - Input variable

## jaccardDistance

```java
INDArray jaccardDistance(INDArray x, INDArray y, int[] dimensions)

SDVariable jaccardDistance(SDVariable x, SDVariable y, int[] dimensions)
SDVariable jaccardDistance(String name, SDVariable x, SDVariable y, int[] dimensions)
```

Jaccard similarity reduction operation. The output contains the Jaccard distance for each

```text
            tensor along the specified dimensions.
```

* **x**  \(NUMERIC\) - Input variable x
* **y**  \(NUMERIC\) - Input variable y
* **dimensions** - Dimensions to calculate jaccardDistance over \(Size: AtLeast\(min=0\)\)

## lastIndex

```java
INDArray lastIndex(INDArray in, Condition condition, int[] dimensions)
INDArray lastIndex(INDArray in, Condition condition, boolean keepDims, int[] dimensions)

SDVariable lastIndex(SDVariable in, Condition condition, int[] dimensions)
SDVariable lastIndex(SDVariable in, Condition condition, boolean keepDims, int[] dimensions)
SDVariable lastIndex(String name, SDVariable in, Condition condition, int[] dimensions)
SDVariable lastIndex(String name, SDVariable in, Condition condition, boolean keepDims, int[] dimensions)
```

Last index reduction operation.

Returns a variable that contains the index of the last element that matches the specified condition \(for each

slice along the specified dimensions\)

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations \(such as subtracting

the mean along a dimension\).

Example: if input has shape \[a,b,c\] and dimensions=\[1\] then output has shape:

keepDims = true: \[a,1,c\]

keepDims = false: \[a,c\]

* **in**  \(NUMERIC\) - Input variable
* **condition** - Condition to check on input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=1\)\)
* **keepDims** - If true: keep the dimensions that are reduced on \(as length 1\). False: remove the reduction dimensions - default = false

## listDiff

```java
INDArray[] listDiff(INDArray x, INDArray y)

SDVariable[] listDiff(SDVariable x, SDVariable y)
SDVariable[] listDiff(String name, SDVariable x, SDVariable y)
```

Calculates difference between inputs X and Y.

* **x**  \(NUMERIC\) - Input variable X
* **y**  \(NUMERIC\) - Input variable Y

## log

```java
INDArray log(INDArray x)

SDVariable log(SDVariable x)
SDVariable log(String name, SDVariable x)
```

Element-wise logarithm function \(base e - natural logarithm\): out = log\(x\)

* **x**  \(NUMERIC\) - Input variable

## log

```java
INDArray log(INDArray x, double base)

SDVariable log(SDVariable x, double base)
SDVariable log(String name, SDVariable x, double base)
```

Element-wise logarithm function \(with specified base\): out = log\_{base\`\(x\)

* **x**  \(NUMERIC\) - Input variable
* **base** - Logarithm base

## log1p

```java
INDArray log1p(INDArray x)

SDVariable log1p(SDVariable x)
SDVariable log1p(String name, SDVariable x)
```

Elementwise natural logarithm function: out = log\_e \(1 + x\)

* **x**  \(NUMERIC\) - Input variable

## logEntropy

```java
INDArray logEntropy(INDArray in, int[] dimensions)

SDVariable logEntropy(SDVariable in, int[] dimensions)
SDVariable logEntropy(String name, SDVariable in, int[] dimensions)
```

Log entropy reduction: log\(-sum\(x \* log\(x\)\)\)

* **in**  \(NUMERIC\) - Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=0\)\)

## logSumExp

```java
INDArray logSumExp(INDArray input, int[] dimensions)

SDVariable logSumExp(SDVariable input, int[] dimensions)
SDVariable logSumExp(String name, SDVariable input, int[] dimensions)
```

Log-sum-exp reduction \(optionally along dimension\).

Computes log\(sum\(exp\(x\)\)

* **input**  \(NUMERIC\) - Input variable
* **dimensions** - Optional dimensions to reduce along \(Size: AtLeast\(min=0\)\)

## manhattanDistance

```java
INDArray manhattanDistance(INDArray x, INDArray y, int[] dimensions)

SDVariable manhattanDistance(SDVariable x, SDVariable y, int[] dimensions)
SDVariable manhattanDistance(String name, SDVariable x, SDVariable y, int[] dimensions)
```

Manhattan distance \(l1 norm, l1 distance\) reduction operation. The output contains the Manhattan distance for each

tensor/subset along the specified dimensions:

out = sum\_i abs\(x\[i\]-y\[i\]\)

* **x**  \(NUMERIC\) - Input variable x
* **y**  \(NUMERIC\) - Input variable y
* **dimensions** - Dimensions to calculate manhattanDistance over \(Size: AtLeast\(min=0\)\)

## matrixDeterminant

```java
INDArray matrixDeterminant(INDArray in)

SDVariable matrixDeterminant(SDVariable in)
SDVariable matrixDeterminant(String name, SDVariable in)
```

Matrix determinant op. For 2D input, this returns the standard matrix determinant.

For higher dimensional input with shape \[..., m, m\] the matrix determinant is returned for each

shape \[m,m\] sub-matrix.

* **in**  \(NUMERIC\) - Input

## matrixInverse

```java
INDArray matrixInverse(INDArray in)

SDVariable matrixInverse(SDVariable in)
SDVariable matrixInverse(String name, SDVariable in)
```

Matrix inverse op. For 2D input, this returns the standard matrix inverse.

For higher dimensional input with shape \[..., m, m\] the matrix inverse is returned for each

shape \[m,m\] sub-matrix.

* **in**  \(NUMERIC\) - Input

## max

```java
INDArray max(INDArray x, INDArray y)

SDVariable max(SDVariable x, SDVariable y)
SDVariable max(String name, SDVariable x, SDVariable y)
```

Pairwise max operation, out = max\(x, y\)

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape \[1,10\] and Y has shape \[5,10\] then op\(X,Y\) has output shape \[5,10\]

Broadcast rules are the same as NumPy: [https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

* **x**  \(NUMERIC\) - First input variable, x
* **y**  \(NUMERIC\) - Second input variable, y

## mergeAdd

```java
INDArray mergeAdd(INDArray inputs)

SDVariable mergeAdd(SDVariable inputs)
SDVariable mergeAdd(String name, SDVariable inputs)
```

Merge add function: merges an arbitrary number of equal shaped arrays using element-wise addition:

out = sum\_i in\[i\]

* **inputs**  \(NUMERIC\) - Input variables

## mergeAvg

```java
INDArray mergeAvg(INDArray inputs)

SDVariable mergeAvg(SDVariable inputs)
SDVariable mergeAvg(String name, SDVariable inputs)
```

Merge average function: merges an arbitrary number of equal shaped arrays using element-wise mean operation:

out = mean\_i in\[i\]

* **inputs**  \(NUMERIC\) - Input variables

## mergeMax

```java
INDArray mergeMax(INDArray inputs)

SDVariable mergeMax(SDVariable inputs)
SDVariable mergeMax(String name, SDVariable inputs)
```

Merge max function: merges an arbitrary number of equal shaped arrays using element-wise maximum operation:

out = max\_i in\[i\]

* **inputs**  \(NUMERIC\) - Input variables

## meshgrid

```java
INDArray[] meshgrid(INDArray inputs, boolean cartesian)

SDVariable[] meshgrid(SDVariable inputs, boolean cartesian)
SDVariable[] meshgrid(String name, SDVariable inputs, boolean cartesian)
```

Broadcasts parameters for evaluation on an N-D grid.

* **inputs**  \(NUMERIC\) - 
* **cartesian** - 

## min

```java
INDArray min(INDArray x, INDArray y)

SDVariable min(SDVariable x, SDVariable y)
SDVariable min(String name, SDVariable x, SDVariable y)
```

Pairwise max operation, out = min\(x, y\)

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape \[1,10\] and Y has shape \[5,10\] then op\(X,Y\) has output shape \[5,10\]

Broadcast rules are the same as NumPy: [https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

* **x**  \(NUMERIC\) - First input variable, x
* **y**  \(NUMERIC\) - Second input variable, y

## mod

```java
INDArray mod(INDArray x, INDArray y)

SDVariable mod(SDVariable x, SDVariable y)
SDVariable mod(String name, SDVariable x, SDVariable y)
```

Pairwise modulus \(remainder\) operation, out = x % y

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape \[1,10\] and Y has shape \[5,10\] then op\(X,Y\) has output shape \[5,10\]

Broadcast rules are the same as NumPy: [https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

* **x**  \(NUMERIC\) - Input variable
* **y**  \(NUMERIC\) - Input variable

## moments

```java
INDArray[] moments(INDArray input, int[] axes)

SDVariable[] moments(SDVariable input, int[] axes)
SDVariable[] moments(String name, SDVariable input, int[] axes)
```

Calculate the mean and \(population\) variance for the input variable, for the specified axis

* **input**  \(NUMERIC\) - Input to calculate moments for
* **axes** - Dimensions to perform calculation over \(Size: AtLeast\(min=0\)\)

## mul

```java
INDArray mul(INDArray x, INDArray y)

SDVariable mul(SDVariable x, SDVariable y)
SDVariable mul(String name, SDVariable x, SDVariable y)
```

Pairwise multiplication operation, out = x \* y

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape \[1,10\] and Y has shape \[5,10\] then op\(X,Y\) has output shape \[5,10\]

Broadcast rules are the same as NumPy: [https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

* **x**  \(NUMERIC\) - Input variable
* **y**  \(NUMERIC\) - Input variable

## mul

```java
INDArray mul(INDArray x, double value)

SDVariable mul(SDVariable x, double value)
SDVariable mul(String name, SDVariable x, double value)
```

Scalar multiplication operation, out = in \* scalar

* **x**  \(NUMERIC\) - Input variable
* **value** - Scalar value for op

## neg

```java
INDArray neg(INDArray x)

SDVariable neg(SDVariable x)
SDVariable neg(String name, SDVariable x)
```

Elementwise negative operation: out = -x

* **x**  \(NUMERIC\) - Input variable

## normalizeMoments

```java
INDArray[] normalizeMoments(INDArray counts, INDArray means, INDArray variances, double shift)

SDVariable[] normalizeMoments(SDVariable counts, SDVariable means, SDVariable variances, double shift)
SDVariable[] normalizeMoments(String name, SDVariable counts, SDVariable means, SDVariable variances, double shift)
```

Calculate the mean and variance from the sufficient statistics

* **counts**  \(NUMERIC\) - Rank 0 \(scalar\) value with the total number of values used to calculate the sufficient statistics
* **means**  \(NUMERIC\) - Mean-value sufficient statistics: this is the SUM of all data values
* **variances**  \(NUMERIC\) - Variaance sufficient statistics: this is the squared sum of all data values
* **shift** - Shift value, possibly 0, used when calculating the sufficient statistics \(for numerical stability\)

## or

```java
INDArray or(INDArray x, INDArray y)

SDVariable or(SDVariable x, SDVariable y)
SDVariable or(String name, SDVariable x, SDVariable y)
```

Boolean OR operation: elementwise \(x != 0\) \|\| \(y != 0\)

If x and y arrays have equal shape, the output shape is the same as these inputs.

Note: supports broadcasting if x and y have different shapes and are broadcastable.

Returns an array with values 1 where condition is satisfied, or value 0 otherwise.

* **x**  \(BOOL\) - Input 1
* **y**  \(BOOL\) - Input 2

## pow

```java
INDArray pow(INDArray x, double value)

SDVariable pow(SDVariable x, double value)
SDVariable pow(String name, SDVariable x, double value)
```

Element-wise power function: out = x^value

* **x**  \(NUMERIC\) - Input variable
* **value** - Scalar value for op

## pow

```java
INDArray pow(INDArray x, INDArray y)

SDVariable pow(SDVariable x, SDVariable y)
SDVariable pow(String name, SDVariable x, SDVariable y)
```

Element-wise \(broadcastable\) power function: out = x\[i\]^y\[i\]

* **x**  \(NUMERIC\) - Input variable
* **y**  \(NUMERIC\) - Power

## rationalTanh

```java
INDArray rationalTanh(INDArray x)

SDVariable rationalTanh(SDVariable x)
SDVariable rationalTanh(String name, SDVariable x)
```

Rational Tanh Approximation elementwise function, as described in the paper:

Compact Convolutional Neural Network Cascade for Face Detection

This is a faster Tanh approximation

* **x**  \(NUMERIC\) - Input variable

## rdiv

```java
INDArray rdiv(INDArray x, INDArray y)

SDVariable rdiv(SDVariable x, SDVariable y)
SDVariable rdiv(String name, SDVariable x, SDVariable y)
```

Pairwise reverse division operation, out = y / x

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape \[1,10\] and Y has shape \[5,10\] then op\(X,Y\) has output shape \[5,10\]

Broadcast rules are the same as NumPy: [https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

* **x**  \(NUMERIC\) - Input variable
* **y**  \(NUMERIC\) - Input variable

## rdiv

```java
INDArray rdiv(INDArray x, double value)

SDVariable rdiv(SDVariable x, double value)
SDVariable rdiv(String name, SDVariable x, double value)
```

Scalar reverse division operation, out = scalar / in

* **x**  \(NUMERIC\) - Input variable
* **value** - Scalar value for op

## reciprocal

```java
INDArray reciprocal(INDArray x)

SDVariable reciprocal(SDVariable x)
SDVariable reciprocal(String name, SDVariable x)
```

Element-wise reciprocal \(inverse\) function: out\[i\] = 1 / in\[i\]

* **x**  \(NUMERIC\) - Input variable

## rectifiedTanh

```java
INDArray rectifiedTanh(INDArray x)

SDVariable rectifiedTanh(SDVariable x)
SDVariable rectifiedTanh(String name, SDVariable x)
```

Rectified tanh operation: max\(0, tanh\(in\)\)

* **x**  \(NUMERIC\) - Input variable

## round

```java
INDArray round(INDArray x)

SDVariable round(SDVariable x)
SDVariable round(String name, SDVariable x)
```

Element-wise round function: out = round\(x\).

Rounds \(up or down depending on value\) to the nearest integer value.

* **x**  \(NUMERIC\) - Input variable

## rsqrt

```java
INDArray rsqrt(INDArray x)

SDVariable rsqrt(SDVariable x)
SDVariable rsqrt(String name, SDVariable x)
```

Element-wise reciprocal \(inverse\) of square root: out = 1.0 / sqrt\(x\)

* **x**  \(NUMERIC\) - Input variable

## rsub

```java
INDArray rsub(INDArray x, INDArray y)

SDVariable rsub(SDVariable x, SDVariable y)
SDVariable rsub(String name, SDVariable x, SDVariable y)
```

Pairwise reverse subtraction operation, out = y - x

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape \[1,10\] and Y has shape \[5,10\] then op\(X,Y\) has output shape \[5,10\]

Broadcast rules are the same as NumPy: [https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

* **x**  \(NUMERIC\) - Input variable
* **y**  \(NUMERIC\) - Input variable

## rsub

```java
INDArray rsub(INDArray x, double value)

SDVariable rsub(SDVariable x, double value)
SDVariable rsub(String name, SDVariable x, double value)
```

Scalar reverse subtraction operation, out = scalar - in

* **x**  \(NUMERIC\) - Input variable
* **value** - Scalar value for op

## setDiag

```java
INDArray setDiag(INDArray in, INDArray diag)

SDVariable setDiag(SDVariable in, SDVariable diag)
SDVariable setDiag(String name, SDVariable in, SDVariable diag)
```

Set the diagonal value to the specified values

If input is

\[ a, b, c\]

\[ d, e, f\]

\[ g, h, i\]

and diag = \[ 1, 2, 3\] then output is

\[ 1, b, c\]

\[ d, 2, f\]

\[ g, h, 3\]

* **in**  \(NUMERIC\) - Input variable
* **diag**  \(NUMERIC\) - Diagonal

## shannonEntropy

```java
INDArray shannonEntropy(INDArray in, int[] dimensions)

SDVariable shannonEntropy(SDVariable in, int[] dimensions)
SDVariable shannonEntropy(String name, SDVariable in, int[] dimensions)
```

Shannon Entropy reduction: -sum\(x \* log2\(x\)\)

* **in**  \(NUMERIC\) - Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed \(Size: AtLeast\(min=0\)\)

## sign

```java
INDArray sign(INDArray x)

SDVariable sign(SDVariable x)
SDVariable sign(String name, SDVariable x)
```

Element-wise sign \(signum\) function:

out = -1 if in &lt; 0

out = 0 if in = 0

out = 1 if in &gt; 0

* **x**  \(NUMERIC\) - Input variable

## sin

```java
INDArray sin(INDArray x)

SDVariable sin(SDVariable x)
SDVariable sin(String name, SDVariable x)
```

Elementwise sine operation: out = sin\(x\)

* **x**  \(NUMERIC\) - Input variable

## sinh

```java
INDArray sinh(INDArray x)

SDVariable sinh(SDVariable x)
SDVariable sinh(String name, SDVariable x)
```

Elementwise sinh \(hyperbolic sine\) operation: out = sinh\(x\)

* **x**  \(NUMERIC\) - Input variable

## sqrt

```java
INDArray sqrt(INDArray x)

SDVariable sqrt(SDVariable x)
SDVariable sqrt(String name, SDVariable x)
```

Element-wise square root function: out = sqrt\(x\)

* **x**  \(NUMERIC\) - Input variable

## square

```java
INDArray square(INDArray x)

SDVariable square(SDVariable x)
SDVariable square(String name, SDVariable x)
```

Element-wise square function: out = x^2

* **x**  \(NUMERIC\) - Input variable

## squaredDifference

```java
INDArray squaredDifference(INDArray x, INDArray y)

SDVariable squaredDifference(SDVariable x, SDVariable y)
SDVariable squaredDifference(String name, SDVariable x, SDVariable y)
```

Pairwise squared difference operation.

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape \[1,10\] and Y has shape \[5,10\] then op\(X,Y\) has output shape \[5,10\]

Broadcast rules are the same as NumPy: [https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

* **x**  \(NUMERIC\) - Input variable
* **y**  \(NUMERIC\) - Input variable

## standardize

```java
INDArray standardize(INDArray x, int[] dimensions)

SDVariable standardize(SDVariable x, int[] dimensions)
SDVariable standardize(String name, SDVariable x, int[] dimensions)
```

Standardize input variable along given axis

out = \(x - mean\) / stdev

with mean and stdev being calculated along the given dimension.

For example: given x as a mini batch of the shape \[numExamples, exampleLength\]:

* use dimension 1 too use the statistics \(mean, stdev\) for each example
* use dimension 0 if you want to use the statistics for each column across all examples
* use dimensions 0,1 if you want to use the statistics across all columns and examples
* **x**  \(NUMERIC\) - Input variable
* **dimensions** -  \(Size: AtLeast\(min=1\)\)

## step

```java
INDArray step(INDArray x, double value)

SDVariable step(SDVariable x, double value)
SDVariable step(String name, SDVariable x, double value)
```

Elementwise step function:

out\(x\) = 1 if x &gt;= cutoff

out\(x\) = 0 otherwise

* **x**  \(NUMERIC\) - Input variable
* **value** - Scalar value for op

## sub

```java
INDArray sub(INDArray x, INDArray y)

SDVariable sub(SDVariable x, SDVariable y)
SDVariable sub(String name, SDVariable x, SDVariable y)
```

Pairwise subtraction operation, out = x - y

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape \[1,10\] and Y has shape \[5,10\] then op\(X,Y\) has output shape \[5,10\]

Broadcast rules are the same as NumPy: [https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

* **x**  \(NUMERIC\) - Input variable
* **y**  \(NUMERIC\) - Input variable

## sub

```java
INDArray sub(INDArray x, double value)

SDVariable sub(SDVariable x, double value)
SDVariable sub(String name, SDVariable x, double value)
```

Scalar subtraction operation, out = in - scalar

* **x**  \(NUMERIC\) - Input variable
* **value** - Scalar value for op

## tan

```java
INDArray tan(INDArray x)

SDVariable tan(SDVariable x)
SDVariable tan(String name, SDVariable x)
```

Elementwise tangent operation: out = tan\(x\)

* **x**  \(NUMERIC\) - Input variable

## tanh

```java
INDArray tanh(INDArray x)

SDVariable tanh(SDVariable x)
SDVariable tanh(String name, SDVariable x)
```

Elementwise tanh \(hyperbolic tangent\) operation: out = tanh\(x\)

* **x**  \(NUMERIC\) - Input variable

## trace

```java
INDArray trace(INDArray in)

SDVariable trace(SDVariable in)
SDVariable trace(String name, SDVariable in)
```

Matrix trace operation

For rank 2 matrices, the output is a scalar vith the trace - i.e., sum of the main diagonal.

For higher rank inputs, output\[a,b,c\] = trace\(in\[a,b,c,:,:\]\)

* **in**  \(NUMERIC\) - Input variable

## xor

```java
INDArray xor(INDArray x, INDArray y)

SDVariable xor(SDVariable x, SDVariable y)
SDVariable xor(String name, SDVariable x, SDVariable y)
```

Boolean XOR \(exclusive OR\) operation: elementwise \(x != 0\) XOR \(y != 0\)

If x and y arrays have equal shape, the output shape is the same as these inputs.

Note: supports broadcasting if x and y have different shapes and are broadcastable.

Returns an array with values 1 where condition is satisfied, or value 0 otherwise.

* **x**  \(BOOL\) - Input 1
* **y**  \(BOOL\) - Input 2

## zeroFraction

```java
INDArray zeroFraction(INDArray input)

SDVariable zeroFraction(SDVariable input)
SDVariable zeroFraction(String name, SDVariable input)
```

Full array zero fraction array reduction operation, optionally along specified dimensions: out = \(count\(x == 0\) / length\(x\)\)

* **input**  \(NUMERIC\) - Input variable


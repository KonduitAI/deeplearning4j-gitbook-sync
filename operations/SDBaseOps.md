---
title: BaseOps
short_title: BaseOps
description: 
category: Operations
weight: 70
---
# Operation classes
## all
```JAVA
INDArray all(INDArray x, int[] dimensions)

SDVariable all(SDVariable x, int[] dimensions)
SDVariable all(String name, SDVariable x, int[] dimensions)
```
Boolean and array reduction operation, optionally along specified dimensions

* **x**  (BOOL type) - Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## any
```JAVA
INDArray any(INDArray x, int[] dimensions)

SDVariable any(SDVariable x, int[] dimensions)
SDVariable any(String name, SDVariable x, int[] dimensions)
```
Boolean or array reduction operation, optionally along specified dimensions

* **x**  (BOOL type) -  Input variable
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## argmax
```JAVA
INDArray argmax(INDArray in, boolean keepDims, int[] dimensions)

SDVariable argmax(SDVariable in, boolean keepDims, int[] dimensions)
SDVariable argmax(String name, SDVariable in, boolean keepDims, int[] dimensions)
INDArray argmax(INDArray in, int[] dimensions)

SDVariable argmax(SDVariable in, int[] dimensions)
SDVariable argmax(String name, SDVariable in, int[] dimensions)
```
Argmax array reduction operation, optionally along specified dimensions.

Output values are the index of the maximum value of each slice along the specified dimension.

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **in**  (NUMERIC type) - Input variable
* **keepDims** - If true: keep the dimensions that are reduced on (as size 1). False: remove the reduction dimensions - default = false
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## argmin
```JAVA
INDArray argmin(INDArray in, boolean keepDims, int[] dimensions)

SDVariable argmin(SDVariable in, boolean keepDims, int[] dimensions)
SDVariable argmin(String name, SDVariable in, boolean keepDims, int[] dimensions)
INDArray argmin(INDArray in, int[] dimensions)

SDVariable argmin(SDVariable in, int[] dimensions)
SDVariable argmin(String name, SDVariable in, int[] dimensions)
```
Argmin array reduction operation, optionally along specified dimensions.

Output values are the index of the minimum value of each slice along the specified dimension.

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape [1,10] and Y has shape [5,10] then op(X,Y) has output shape [5,10]<br>
Broadcast rules are the same as NumPy: https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html<br>
* **in**  (NUMERIC type) - Input variable
* **keepDims** - If true: keep the dimensions that are reduced on (as size 1). False: remove the reduction dimensions - default = false
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## batchMmul
```JAVA
INDArray batchMmul(INDArray inputsA, INDArray inputsB, boolean transposeA, boolean transposeB)

SDVariable batchMmul(SDVariable inputsA, SDVariable inputsB, boolean transposeA, boolean transposeB)
SDVariable batchMmul(String name, SDVariable inputsA, SDVariable inputsB, boolean transposeA, boolean transposeB)
INDArray batchMmul(INDArray inputsA, INDArray inputsB)

SDVariable batchMmul(SDVariable inputsA, SDVariable inputsB)
SDVariable batchMmul(String name, SDVariable inputsA, SDVariable inputsB)
```
Matrix multiply a batch of matrices. matricesA and matricesB have to be arrays of same

length and each pair taken from these sets has to have dimensions (M, N) and (N, K),

respectively. If transposeA is true, matrices from matricesA will have shape (N, M) instead.

Likewise, if transposeB is true, matrices from matricesB will have shape (K, N).



The result of this operation will be a batch of multiplied matrices. The

result has the same length as both input batches and each output matrix is of shape (M, K).

* **inputsA**  (NUMERIC type) - First array of input matrices, all of shape (M, N) or (N, M)
* **inputsB**  (NUMERIC type) -  Second array of input matrices, all of shape (N, K) or (K, N)
* **transposeA** - Whether to transpose A arrays or not - default = false
* **transposeB** - Whether to transpose B arrays or not - default = false

## castTo
```JAVA
INDArray castTo(INDArray arg, DataType datatype)

SDVariable castTo(SDVariable arg, DataType datatype)
SDVariable castTo(String name, SDVariable arg, DataType datatype)
```
Cast the array to a new datatype - for example, Integer -> Float

* **arg**  (NDARRAY type) - Input variable to cast
* **datatype** - Datatype to cast to

## concat
```JAVA
INDArray concat(INDArray inputs, int dimension)

SDVariable concat(SDVariable inputs, int dimension)
SDVariable concat(String name, SDVariable inputs, int dimension)
```
Concatenate a set of inputs along the specified dimension.

Note that inputs must have identical rank and identical dimensions, other than the dimension to stack on.

For example, if 2 inputs have shape [a, x, c] and [a, y, c] and dimension = 1, then the output has shape [a, x+y, c]

* **inputs**  (NUMERIC type) - Input variables
* **dimension** - Dimension to concatenate on

## cumprod
```JAVA
INDArray cumprod(INDArray in, boolean exclusive, boolean reverse, int[] axis)

SDVariable cumprod(SDVariable in, boolean exclusive, boolean reverse, int[] axis)
SDVariable cumprod(String name, SDVariable in, boolean exclusive, boolean reverse, int[] axis)
INDArray cumprod(INDArray in, int[] axis)

SDVariable cumprod(SDVariable in, int[] axis)
SDVariable cumprod(String name, SDVariable in, int[] axis)
```
Cumulative product operation.

For input: [ a, b, c], output is:

exclusive=false, reverse=false: [a, a*b, a*b*c]

exclusive=true, reverse=false, [0, a, a*b]

exclusive=false, reverse=true: [a*b*c, b*c, c]

exclusive=true, reverse=true: [b*c, c, 0]

* **in**  (NUMERIC type) - Input variable
* **exclusive** - If true: exclude the first value - default = false
* **reverse** - If true: reverse the direction of the accumulation - default = false
* **axis** - Scalar axis argument for dimension to perform cumululative sum operations along (Size: AtLeast(min=1))

## cumsum
```JAVA
INDArray cumsum(INDArray in, boolean exclusive, boolean reverse, int[] axis)

SDVariable cumsum(SDVariable in, boolean exclusive, boolean reverse, int[] axis)
SDVariable cumsum(String name, SDVariable in, boolean exclusive, boolean reverse, int[] axis)
INDArray cumsum(INDArray in, int[] axis)

SDVariable cumsum(SDVariable in, int[] axis)
SDVariable cumsum(String name, SDVariable in, int[] axis)
```
Cumulative sum operation.

For input: [ a, b, c], output is:

exclusive=false, reverse=false: [a, a+b, a+b+c]

exclusive=true, reverse=false, [0, a, a+b]

exclusive=false, reverse=true: [a+b+c, b+c, c]

exclusive=true, reverse=true: [b+c, c, 0]

* **in**  (NUMERIC type) - Input variable
* **exclusive** - If true: exclude the first value - default = false
* **reverse** - If true: reverse the direction of the accumulation - default = false
* **axis** - Scalar axis argument for dimension to perform cumululative sum operations along (Size: AtLeast(min=1))

## dot
```JAVA
INDArray dot(INDArray x, INDArray y, int[] dimensions)

SDVariable dot(SDVariable x, SDVariable y, int[] dimensions)
SDVariable dot(String name, SDVariable x, SDVariable y, int[] dimensions)
```
Pairwise dot product reduction along dimension

output = sum(i=0 ... size(dim)-1) x[i] * y[i]

* **x**  (NUMERIC type) - first input
* **y**  (NUMERIC type) - second input
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## dynamicPartition
```JAVA
INDArray dynamicPartition(INDArray x, INDArray partitions, int numPartitions)

SDVariable dynamicPartition(SDVariable x, SDVariable partitions, int numPartitions)
SDVariable dynamicPartition(String name, SDVariable x, SDVariable partitions, int numPartitions)
```
Dynamically partition the input variable values into the specified number of paritions, using the indices.

Example:

<pre>

input = [1,2,3,4,5]

numPartitions = 2

partitions = [1,0,0,1,0]

out[0] = [2,3,5]

out[1] = [1,4] `

</pre>

* **x**  (NUMERIC type) - Input variable
* **partitions**  (INT type) - 1D input with values 0 to numPartitions-1
* **numPartitions** - Number of partitions, >= 1

## dynamicStitch
```JAVA
INDArray dynamicStitch(INDArray indices, INDArray x)

SDVariable dynamicStitch(SDVariable indices, SDVariable x)
SDVariable dynamicStitch(String name, SDVariable indices, SDVariable x)
```
Dynamically merge the specified input arrays into a single array, using the specified indices

* **indices**  (INT type) - Indices to use when merging. Must be >= 1, same length as input variables
* **x**  (NUMERIC type) - Input variables.

## eq
```JAVA
INDArray eq(INDArray x, double y)

SDVariable eq(SDVariable x, double y)
SDVariable eq(String name, SDVariable x, double y)
```
Equals operation: elementwise x == y

Return boolean array with values true where satisfied, or false otherwise.

* **x**  (NUMERIC type) - Input array
* **y** - Double value argument to use in operation

## eq
```JAVA
INDArray eq(INDArray x, INDArray y)

SDVariable eq(SDVariable x, SDVariable y)
SDVariable eq(String name, SDVariable x, SDVariable y)
```
Equal to operation: elementwise x == y

If x and y arrays have equal shape, the output shape is the same as these inputs.

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape [1,10] and Y has shape [5,10] then op(X,Y) has output shape [5,10]<br>
Broadcast rules are the same as NumPy: https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html<br>
Return boolean array with values true where satisfied, or false otherwise.

* **x**  (NUMERIC type) - Input 1
* **y**  (NUMERIC type) - Input 2

## expandDims
```JAVA
INDArray expandDims(INDArray x, int axis)

SDVariable expandDims(SDVariable x, int axis)
SDVariable expandDims(String name, SDVariable x, int axis)
```
Reshape the input by adding a 1 at the specified location.

For example, if input has shape [a, b], then output shape is:

axis = 0: [1, a, b]

axis = 1: [a, 1, b]

axis = 2: [a, b, 1]

* **x**  (NDARRAY type) - Input variable
* **axis** - Axis to expand

## fill
```JAVA
INDArray fill(INDArray shape, DataType dataType, double value)

SDVariable fill(SDVariable shape, DataType dataType, double value)
SDVariable fill(String name, SDVariable shape, DataType dataType, double value)
```
Generate an output variable with the specified (dynamic) shape with all elements set to the specified value

* **shape**  (INT type) - Shape: must be a 1D array/variable
* **dataType** - Datatype of the output array
* **value** - Value to set all elements to

## gather
```JAVA
INDArray gather(INDArray df, int[] indices, int axis)

SDVariable gather(SDVariable df, int[] indices, int axis)
SDVariable gather(String name, SDVariable df, int[] indices, int axis)
```
Gather slices from the input variable where the indices are specified as fixed int[] values.

Output shape is same as input shape, except for axis dimension, which has size equal to indices.length.

* **df**  (NUMERIC type) - Input variable
* **indices** - Indices to get (Size: AtLeast(min=1))
* **axis** - Axis that the indices refer to

## gather
```JAVA
INDArray gather(INDArray df, INDArray indices, int axis)

SDVariable gather(SDVariable df, SDVariable indices, int axis)
SDVariable gather(String name, SDVariable df, SDVariable indices, int axis)
```
Gather slices from the input variable where the indices are specified as dynamic array values.

Output shape is same as input shape, except for axis dimension, which has size equal to indices.length.

* **df**  (NUMERIC type) - Input variable
* **indices**  (INT type) - Indices to get slices for. Rank 0 or 1 input
* **axis** - Axis that the indices refer to

## gatherNd
```JAVA
INDArray gatherNd(INDArray df, INDArray indices)

SDVariable gatherNd(SDVariable df, SDVariable indices)
SDVariable gatherNd(String name, SDVariable df, SDVariable indices)
```
Gather slices from df with shape specified by indices. 

* **df**  (NUMERIC type) - 
* **indices**  (NUMERIC type) - 

## gt
```JAVA
INDArray gt(INDArray x, double y)

SDVariable gt(SDVariable x, double y)
SDVariable gt(String name, SDVariable x, double y)
```
Greater than operation: elementwise x > y

Return boolean array with values true where satisfied, or false otherwise.

* **x**  (NUMERIC type) - Input array
* **y** - Double value argument to use in operation

## gt
```JAVA
INDArray gt(INDArray x, INDArray y)

SDVariable gt(SDVariable x, SDVariable y)
SDVariable gt(String name, SDVariable x, SDVariable y)
```
Greater than operation: elementwise x > y

If x and y arrays have equal shape, the output shape is the same as these inputs.

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape [1,10] and Y has shape [5,10] then op(X,Y) has output shape [5,10]<br>
Broadcast rules are the same as NumPy: https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html<br>
Return boolean array with values true where satisfied, or false otherwise.

* **x**  (NUMERIC type) - Input 1
* **y**  (NUMERIC type) - Input 2

## gte
```JAVA
INDArray gte(INDArray x, double y)

SDVariable gte(SDVariable x, double y)
SDVariable gte(String name, SDVariable x, double y)
```
Greater than or equals operation: elementwise x >= y

Return boolean array with values true where satisfied, or false otherwise.

* **x**  (NUMERIC type) - Input array
* **y** - Double value argument to use in operation

## gte
```JAVA
INDArray gte(INDArray x, INDArray y)

SDVariable gte(SDVariable x, SDVariable y)
SDVariable gte(String name, SDVariable x, SDVariable y)
```
Greater than or equal to operation: elementwise x >= y

If x and y arrays have equal shape, the output shape is the same as these inputs.

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape [1,10] and Y has shape [5,10] then op(X,Y) has output shape [5,10]<br>
Broadcast rules are the same as NumPy: https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html<br>
Return boolean array with values true where satisfied, or false otherwise.

* **x**  (NUMERIC type) - Input 1
* **y**  (NUMERIC type) - Input 2

## identity
```JAVA
INDArray identity(INDArray input)

SDVariable identity(SDVariable input)
SDVariable identity(String name, SDVariable input)
```
Elementwise identity operation: out = x

* **input**  (NUMERIC type) - Input variable

## invertPermutation
```JAVA
INDArray invertPermutation(INDArray input)

SDVariable invertPermutation(SDVariable input)
SDVariable invertPermutation(String name, SDVariable input)
```
Compute the inverse permutation indices for a permutation operation

Example: if input is [2, 0, 1] then output is [1, 2, 0]

The idea is that x.permute(input).permute(invertPermutation(input)) == x

* **input**  (INT type) - 1D indices for permutation

## isNumericTensor
```JAVA
INDArray isNumericTensor(INDArray x)

SDVariable isNumericTensor(SDVariable x)
SDVariable isNumericTensor(String name, SDVariable x)
```
Is the director a numeric tensor? In the current version of ND4J/SameDiff, this always returns true/1

* **x**  (NUMERIC type) - Input variable

## linspace
```JAVA
INDArray linspace(DataType dataType, double start, double stop, long number)

SDVariable linspace(DataType dataType, double start, double stop, long number)
SDVariable linspace(String name, DataType dataType, double start, double stop, long number)
```
Create a new 1d array with values evenly spaced between values 'start' and 'stop'

For example, linspace(start=3.0, stop=4.0, number=3) will generate [3.0, 3.5, 4.0]

* **dataType** - Data type of the output array
* **start** - Start value
* **stop** - Stop value
* **number** - Number of values to generate

## linspace
```JAVA
INDArray linspace(INDArray start, INDArray stop, INDArray number, DataType dataType)

SDVariable linspace(SDVariable start, SDVariable stop, SDVariable number, DataType dataType)
SDVariable linspace(String name, SDVariable start, SDVariable stop, SDVariable number, DataType dataType)
```
Create a new 1d array with values evenly spaced between values 'start' and 'stop'

For example, linspace(start=3.0, stop=4.0, number=3) will generate [3.0, 3.5, 4.0]

* **start**  (NUMERIC type) - Start value
* **stop**  (NUMERIC type) - Stop value
* **number**  (LONG type) - Number of values to generate
* **dataType** - Data type of the output array

## lt
```JAVA
INDArray lt(INDArray x, double y)

SDVariable lt(SDVariable x, double y)
SDVariable lt(String name, SDVariable x, double y)
```
Less than operation: elementwise x < y

Return boolean array with values true where satisfied, or false otherwise.

* **x**  (NUMERIC type) - Input array
* **y** - Double value argument to use in operation

## lt
```JAVA
INDArray lt(INDArray x, INDArray y)

SDVariable lt(SDVariable x, SDVariable y)
SDVariable lt(String name, SDVariable x, SDVariable y)
```
Less than operation: elementwise x < y

If x and y arrays have equal shape, the output shape is the same as these inputs.

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape [1,10] and Y has shape [5,10] then op(X,Y) has output shape [5,10]<br>
Broadcast rules are the same as NumPy: https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html<br>
Return boolean array with values true where satisfied, or false otherwise.

* **x**  (NUMERIC type) - Input 1
* **y**  (NUMERIC type) - Input 2

## lte
```JAVA
INDArray lte(INDArray x, double y)

SDVariable lte(SDVariable x, double y)
SDVariable lte(String name, SDVariable x, double y)
```
Less than or equals operation: elementwise x <= y

Return boolean array with values true where satisfied, or false otherwise.

* **x**  (NUMERIC type) - Input array
* **y** - Double value argument to use in operation

## lte
```JAVA
INDArray lte(INDArray x, INDArray y)

SDVariable lte(SDVariable x, SDVariable y)
SDVariable lte(String name, SDVariable x, SDVariable y)
```
Less than or equal to operation: elementwise x <= y

If x and y arrays have equal shape, the output shape is the same as these inputs.

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape [1,10] and Y has shape [5,10] then op(X,Y) has output shape [5,10]<br>
Broadcast rules are the same as NumPy: https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html<br>
Return boolean array with values true where satisfied, or false otherwise.

* **x**  (NUMERIC type) - Input 1
* **y**  (NUMERIC type) - Input 2

## matchCondition
```JAVA
INDArray matchCondition(INDArray in, Condition condition)

SDVariable matchCondition(SDVariable in, Condition condition)
SDVariable matchCondition(String name, SDVariable in, Condition condition)
```
Returns a boolean mask of equal shape to the input, where the condition is satisfied - value 1 where satisfied, 0 otherwise

* **in**  (NUMERIC type) - Input
* **condition** - Condition

## matchConditionCount
```JAVA
INDArray matchConditionCount(INDArray in, Condition condition)

SDVariable matchConditionCount(SDVariable in, Condition condition)
SDVariable matchConditionCount(String name, SDVariable in, Condition condition)
```
Returns a count of the number of elements that satisfy the condition

* **in**  (NUMERIC type) - Input
* **condition** - Condition

## matchConditionCount
```JAVA
INDArray matchConditionCount(INDArray in, Condition condition, boolean keepDim, int[] dimensions)

SDVariable matchConditionCount(SDVariable in, Condition condition, boolean keepDim, int[] dimensions)
SDVariable matchConditionCount(String name, SDVariable in, Condition condition, boolean keepDim, int[] dimensions)
INDArray matchConditionCount(INDArray in, Condition condition, int[] dimensions)

SDVariable matchConditionCount(SDVariable in, Condition condition, int[] dimensions)
SDVariable matchConditionCount(String name, SDVariable in, Condition condition, int[] dimensions)
```
Returns a count of the number of elements that satisfy the condition (for each slice along the specified dimensions)

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **in**  (NUMERIC type) - Input variable
* **condition** - Condition
* **keepDim** - If true: keep the dimensions that are reduced on (as size 1). False: remove the reduction dimensions - default = false
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## max
```JAVA
INDArray max(INDArray x, boolean keepDims, int[] dimensions)

SDVariable max(SDVariable x, boolean keepDims, int[] dimensions)
SDVariable max(String name, SDVariable x, boolean keepDims, int[] dimensions)
INDArray max(INDArray x, int[] dimensions)

SDVariable max(SDVariable x, int[] dimensions)
SDVariable max(String name, SDVariable x, int[] dimensions)
```
Max array reduction operation, optionally along specified dimensions

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **x**  (NUMERIC type) - Input variable
* **keepDims** - If true: keep the dimensions that are reduced on (as size 1). False: remove the reduction dimensions - default = false
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## max
```JAVA
INDArray max(INDArray first, INDArray second)

SDVariable max(SDVariable first, SDVariable second)
SDVariable max(String name, SDVariable first, SDVariable second)
```
Element-wise maximum operation: out[i] = max(first[i], second[i])

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape [1,10] and Y has shape [5,10] then op(X,Y) has output shape [5,10]<br>
Broadcast rules are the same as NumPy: https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html<br>
* **first**  (NUMERIC type) - First input array
* **second**  (NUMERIC type) - Second input array

## mean
```JAVA
INDArray mean(INDArray x, boolean keepDims, int[] dimensions)

SDVariable mean(SDVariable x, boolean keepDims, int[] dimensions)
SDVariable mean(String name, SDVariable x, boolean keepDims, int[] dimensions)
INDArray mean(INDArray x, int[] dimensions)

SDVariable mean(SDVariable x, int[] dimensions)
SDVariable mean(String name, SDVariable x, int[] dimensions)
```
Mean (average) array reduction operation, optionally along specified dimensions

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **x**  (NUMERIC type) - Input variable
* **keepDims** - If true: keep the dimensions that are reduced on (as size 1). False: remove the reduction dimensions - default = false
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## min
```JAVA
INDArray min(INDArray x, boolean keepDims, int[] dimensions)

SDVariable min(SDVariable x, boolean keepDims, int[] dimensions)
SDVariable min(String name, SDVariable x, boolean keepDims, int[] dimensions)
INDArray min(INDArray x, int[] dimensions)

SDVariable min(SDVariable x, int[] dimensions)
SDVariable min(String name, SDVariable x, int[] dimensions)
```
Minimum array reduction operation, optionally along specified dimensions. out = min(in)

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **x**  (NUMERIC type) - Input variable
* **keepDims** - If true: keep the dimensions that are reduced on (as size 1). False: remove the reduction dimensions - default = false
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## min
```JAVA
INDArray min(INDArray first, INDArray second)

SDVariable min(SDVariable first, SDVariable second)
SDVariable min(String name, SDVariable first, SDVariable second)
```
Element-wise minimum operation: out[i] = min(first[i], second[i])

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape [1,10] and Y has shape [5,10] then op(X,Y) has output shape [5,10]<br>
Broadcast rules are the same as NumPy: https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html<br>
* **first**  (NUMERIC type) - First input array
* **second**  (NUMERIC type) - Second input array

## mmul
```JAVA
INDArray mmul(INDArray x, INDArray y, boolean transposeX, boolean transposeY, boolean transposeZ)

SDVariable mmul(SDVariable x, SDVariable y, boolean transposeX, boolean transposeY, boolean transposeZ)
SDVariable mmul(String name, SDVariable x, SDVariable y, boolean transposeX, boolean transposeY, boolean transposeZ)
INDArray mmul(INDArray x, INDArray y)

SDVariable mmul(SDVariable x, SDVariable y)
SDVariable mmul(String name, SDVariable x, SDVariable y)
```
Matrix multiplication: out = mmul(x,y)

Supports specifying transpose argument to perform operation such as mmul(a^T, b), etc.

* **x**  (NUMERIC type) - First input variable
* **y**  (NUMERIC type) - Second input variable
* **transposeX** - Transpose x (first argument) - default = false
* **transposeY** - Transpose y (second argument) - default = false
* **transposeZ** - Transpose result array - default = false

## neq
```JAVA
INDArray neq(INDArray x, double y)

SDVariable neq(SDVariable x, double y)
SDVariable neq(String name, SDVariable x, double y)
```
Not equals operation: elementwise x != y

Return boolean array with values true where satisfied, or false otherwise.

* **x**  (NUMERIC type) - Input array
* **y** - Double value argument to use in operation

## neq
```JAVA
INDArray neq(INDArray x, INDArray y)

SDVariable neq(SDVariable x, SDVariable y)
SDVariable neq(String name, SDVariable x, SDVariable y)
```
Not equal to operation: elementwise x != y

If x and y arrays have equal shape, the output shape is the same as these inputs.

Note: supports broadcasting if x and y have different shapes and are broadcastable.

For example, if X has shape [1,10] and Y has shape [5,10] then op(X,Y) has output shape [5,10]<br>
Broadcast rules are the same as NumPy: https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html<br>
Return boolean array with values true where satisfied, or false otherwise.

* **x**  (NUMERIC type) - Input 1
* **y**  (NUMERIC type) - Input 2

## norm1
```JAVA
INDArray norm1(INDArray x, boolean keepDims, int[] dimensions)

SDVariable norm1(SDVariable x, boolean keepDims, int[] dimensions)
SDVariable norm1(String name, SDVariable x, boolean keepDims, int[] dimensions)
INDArray norm1(INDArray x, int[] dimensions)

SDVariable norm1(SDVariable x, int[] dimensions)
SDVariable norm1(String name, SDVariable x, int[] dimensions)
```
Norm1 (L1 norm) reduction operation: The output contains the L1 norm for each tensor/subset along the specified dimensions: 

out = sum_i abs(x[i])

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **x**  (NUMERIC type) - Input variable
* **keepDims** - If true: keep the dimensions that are reduced on (as size 1). False: remove the reduction dimensions - default = false
* **dimensions** - dimensions to reduce over (Size: AtLeast(min=0))

## norm2
```JAVA
INDArray norm2(INDArray x, boolean keepDims, int[] dimensions)

SDVariable norm2(SDVariable x, boolean keepDims, int[] dimensions)
SDVariable norm2(String name, SDVariable x, boolean keepDims, int[] dimensions)
INDArray norm2(INDArray x, int[] dimensions)

SDVariable norm2(SDVariable x, int[] dimensions)
SDVariable norm2(String name, SDVariable x, int[] dimensions)
```
Norm2 (L2 norm) reduction operation: The output contains the L2 norm for each tensor/subset along the specified dimensions:

out = sqrt(sum_i x[i]^2)

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **x**  (NUMERIC type) - Input variable
* **keepDims** - If true: keep the dimensions that are reduced on (as size 1). False: remove the reduction dimensions - default = false
* **dimensions** - dimensions dimensions to reduce over (Size: AtLeast(min=0))

## normmax
```JAVA
INDArray normmax(INDArray x, boolean keepDims, int[] dimensions)

SDVariable normmax(SDVariable x, boolean keepDims, int[] dimensions)
SDVariable normmax(String name, SDVariable x, boolean keepDims, int[] dimensions)
INDArray normmax(INDArray x, int[] dimensions)

SDVariable normmax(SDVariable x, int[] dimensions)
SDVariable normmax(String name, SDVariable x, int[] dimensions)
```
Max norm (infinity norm) reduction operation: The output contains the max norm for each tensor/subset along the

specified dimensions:

out = max(abs(x[i]))

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **x**  (NUMERIC type) - Input variable
* **keepDims** - If true: keep the dimensions that are reduced on (as size 1). False: remove the reduction dimensions - default = false
* **dimensions** - dimensions to reduce over (Size: AtLeast(min=0))

## oneHot
```JAVA
INDArray oneHot(INDArray indices, int depth, int axis, double on, double off, DataType dataType)

SDVariable oneHot(SDVariable indices, int depth, int axis, double on, double off, DataType dataType)
SDVariable oneHot(String name, SDVariable indices, int depth, int axis, double on, double off, DataType dataType)
INDArray oneHot(INDArray indices, int depth, int axis, double on, double off)

SDVariable oneHot(SDVariable indices, int depth, int axis, double on, double off)
SDVariable oneHot(String name, SDVariable indices, int depth, int axis, double on, double off)
```
Convert the array to a one-hot array with walues and  for each entry

If input has shape [ a, ..., n] then output has shape [ a, ..., n, depth],

with {out[i, ..., j, in[i,...,j]]  with other values being set to

* **indices**  (NUMERIC type) - Indices - value 0 to depth-1
* **depth** - Number of classes
* **axis** - 
* **on** - 
* **off** - 
* **dataType** - Output data type - default = DataType.FLOAT

## oneHot
```JAVA
INDArray oneHot(INDArray indices, int depth)

SDVariable oneHot(SDVariable indices, int depth)
SDVariable oneHot(String name, SDVariable indices, int depth)
```
Convert the array to a one-hot array with walues 0 and 1 for each entry

If input has shape [ a, ..., n] then output has shape [ a, ..., n, depth],

with out[i, ..., j, in[i,...,j]] = 1 with other values being set to 0

see oneHot(SDVariable, int, int, double, double)

* **indices**  (NUMERIC type) - Indices - value 0 to depth-1
* **depth** - Number of classes

## onesLike
```JAVA
INDArray onesLike(INDArray input)

SDVariable onesLike(SDVariable input)
SDVariable onesLike(String name, SDVariable input)
```
Return a variable of all 1s, with the same shape as the input variable. Note that this is dynamic:

if the input shape changes in later execution, the returned variable's shape will also be updated

* **input**  (NUMERIC type) - Input INDArray 

## onesLike
```JAVA
INDArray onesLike(INDArray input, DataType dataType)

SDVariable onesLike(SDVariable input, DataType dataType)
SDVariable onesLike(String name, SDVariable input, DataType dataType)
```
As per onesLike(String, SDVariable) but the output datatype may be specified

* **input**  (NUMERIC type) - 
* **dataType** - 

## permute
```JAVA
INDArray permute(INDArray x, INDArray dimensions)

SDVariable permute(SDVariable x, SDVariable dimensions)
SDVariable permute(String name, SDVariable x, SDVariable dimensions)
```
Array permutation operation: permute the dimensions according to the specified permutation indices.

Example: if input has shape [a,b,c] and dimensions = [2,0,1] the output has shape [c,a,b]

* **x**  (NUMERIC type) - Input variable
* **dimensions**  (INT type) - Permute dimensions

## permute
```JAVA
INDArray permute(INDArray x, int[] dimensions)

SDVariable permute(SDVariable x, int[] dimensions)
SDVariable permute(String name, SDVariable x, int[] dimensions)
```
Array permutation operation: permute the dimensions according to the specified permutation indices.

Example: if input has shape [a,b,c] and dimensions = [2,0,1] the output has shape [c,a,b]

* **x**  (NUMERIC type) - Input variable
* **dimensions** -  (Size: AtLeast(min=0))

## prod
```JAVA
INDArray prod(INDArray x, boolean keepDims, int[] dimensions)

SDVariable prod(SDVariable x, boolean keepDims, int[] dimensions)
SDVariable prod(String name, SDVariable x, boolean keepDims, int[] dimensions)
INDArray prod(INDArray x, int[] dimensions)

SDVariable prod(SDVariable x, int[] dimensions)
SDVariable prod(String name, SDVariable x, int[] dimensions)
```
Product array reduction operation, optionally along specified dimensions

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **x**  (NUMERIC type) - Input variable
* **keepDims** - If true: keep the dimensions that are reduced on (as size 1). False: remove the reduction dimensions - default = false
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## range
```JAVA
INDArray range(double from, double to, double step, DataType dataType)

SDVariable range(double from, double to, double step, DataType dataType)
SDVariable range(String name, double from, double to, double step, DataType dataType)
```
Create a new variable with a 1d array, where the values start at from and increment by step

up to (but not including) limit.

For example, range(1.0, 3.0, 0.5) will return [1.0, 1.5, 2.0, 2.5]

* **from** - Initial/smallest value
* **to** - Largest value (exclusive)
* **step** - Step size
* **dataType** - 

## range
```JAVA
INDArray range(INDArray from, INDArray to, INDArray step, DataType dataType)

SDVariable range(SDVariable from, SDVariable to, SDVariable step, DataType dataType)
SDVariable range(String name, SDVariable from, SDVariable to, SDVariable step, DataType dataType)
```
Create a new variable with a 1d array, where the values start at from and increment by step

up to (but not including) limit.

For example, range(1.0, 3.0, 0.5) will return [1.0, 1.5, 2.0, 2.5]

* **from**  (NUMERIC type) - Initial/smallest value
* **to**  (NUMERIC type) - Largest value (exclusive)
* **step**  (NUMERIC type) - Step size
* **dataType** - 

## rank
```JAVA
INDArray rank(INDArray in)

SDVariable rank(SDVariable in)
SDVariable rank(String name, SDVariable in)
```
Returns the rank (number of dimensions, i.e., length(shape)) of the specified INDArray  as a 0D scalar variable

* **in**  (NUMERIC type) - Input variable

## replaceWhere
```JAVA
INDArray replaceWhere(INDArray update, INDArray from, Condition condition)

SDVariable replaceWhere(SDVariable update, SDVariable from, Condition condition)
SDVariable replaceWhere(String name, SDVariable update, SDVariable from, Condition condition)
```
Element-wise replace where condition:

out[i] = from[i] if condition(update[i]) is satisfied, or

out[i] = update[i] if condition(update[i]) is NOT satisfied

* **update**  (NUMERIC type) - Source array
* **from**  (NUMERIC type) - Replacement values array (used conditionally). Must be same shape as 'update' array
* **condition** - Condition to check on update array elements

## replaceWhere
```JAVA
INDArray replaceWhere(INDArray update, double value, Condition condition)

SDVariable replaceWhere(SDVariable update, double value, Condition condition)
SDVariable replaceWhere(String name, SDVariable update, double value, Condition condition)
```
Element-wise replace where condition:

out[i] = value if condition(update[i]) is satisfied, or

out[i] = update[i] if condition(update[i]) is NOT satisfied

* **update**  (NUMERIC type) - Source array
* **value** - Value to set at the output, if the condition is satisfied
* **condition** - Condition to check on update array elements

## reshape
```JAVA
INDArray reshape(INDArray x, INDArray shape)

SDVariable reshape(SDVariable x, SDVariable shape)
SDVariable reshape(String name, SDVariable x, SDVariable shape)
```
Reshape the input variable to the specified (fixed) shape. The output variable will have the same values as the

input, but with the specified shape.

Note that prod(shape) must match length(input) == prod(input.shape)

* **x**  (NUMERIC type) - Input variable
* **shape**  (NUMERIC type) - New shape for variable

## reshape
```JAVA
INDArray reshape(INDArray x, long[] shape)

SDVariable reshape(SDVariable x, long[] shape)
SDVariable reshape(String name, SDVariable x, long[] shape)
```
Reshape the input variable to the specified (fixed) shape. The output variable will have the same values as the

input, but with the specified shape.

Note that prod(shape) must match length(input) == prod(input.shape)

* **x**  (NUMERIC type) - Input variable
* **shape** - New shape for variable (Size: AtLeast(min=0))

## reverse
```JAVA
INDArray reverse(INDArray x, int[] dimensions)

SDVariable reverse(SDVariable x, int[] dimensions)
SDVariable reverse(String name, SDVariable x, int[] dimensions)
```
Reverse the values of an array for the specified dimensions

If input is:

[ 1, 2, 3]

[ 4, 5, 6]

then

reverse(in, 0):

[3, 2, 1]

[6, 5, 4]

reverse(in, 1):

[4, 5, 6]

[1, 2 3]

* **x**  (NUMERIC type) - Input variable
* **dimensions** - Input variable (Size: AtLeast(min=0))

## reverseSequence
```JAVA
INDArray reverseSequence(INDArray x, INDArray seq_lengths, int seqDim, int batchDim)

SDVariable reverseSequence(SDVariable x, SDVariable seq_lengths, int seqDim, int batchDim)
SDVariable reverseSequence(String name, SDVariable x, SDVariable seq_lengths, int seqDim, int batchDim)
INDArray reverseSequence(INDArray x, INDArray seq_lengths)

SDVariable reverseSequence(SDVariable x, SDVariable seq_lengths)
SDVariable reverseSequence(String name, SDVariable x, SDVariable seq_lengths)
```
Reverse sequence op: for each slice along dimension seqDimension, the first seqLength values are reversed

* **x**  (NUMERIC type) - Input variable
* **seq_lengths**  (INT type) - Length of the sequences
* **seqDim** - Sequence dimension - default = -1
* **batchDim** - Batch dimension - default = 0

## scalarFloorMod
```JAVA
INDArray scalarFloorMod(INDArray in, double value)

SDVariable scalarFloorMod(SDVariable in, double value)
SDVariable scalarFloorMod(String name, SDVariable in, double value)
```
Element-wise scalar floor modulus operation: out = floorMod(in, value).

i.e., returns the remainder after division by 'value'

* **in**  (NUMERIC type) - Input variable
* **value** - Scalar value to compare

## scalarMax
```JAVA
INDArray scalarMax(INDArray in, double value)

SDVariable scalarMax(SDVariable in, double value)
SDVariable scalarMax(String name, SDVariable in, double value)
```
Element-wise scalar maximum operation: out = max(in, value)

* **in**  (NUMERIC type) - Input variable
* **value** - Scalar value to compare

## scalarMin
```JAVA
INDArray scalarMin(INDArray in, double value)

SDVariable scalarMin(SDVariable in, double value)
SDVariable scalarMin(String name, SDVariable in, double value)
```
Element-wise scalar minimum operation: out = min(in, value)

* **in**  (NUMERIC type) - Input variable
* **value** - Scalar value to compare

## scalarSet
```JAVA
INDArray scalarSet(INDArray in, double set)

SDVariable scalarSet(SDVariable in, double set)
SDVariable scalarSet(String name, SDVariable in, double set)
```
Return a variable with equal shape to the input, but all elements set to value 'set'

* **in**  (NUMERIC type) - Input variable
* **set** - Value to set

## scatterAdd
```JAVA
INDArray scatterAdd(INDArray ref, INDArray indices, INDArray updates)

SDVariable scatterAdd(SDVariable ref, SDVariable indices, SDVariable updates)
SDVariable scatterAdd(String name, SDVariable ref, SDVariable indices, SDVariable updates)
```
Scatter addition operation.

If indices is rank 0 (a scalar), then out[index, ...] = out[index, ...] + op(updates[...])

If indices is rank 1 (a vector), then for each position i, out[indices[i], ...] = out[indices[i], ...] + op(updates[i, ...])

If indices is rank 2+, then for each position (i,...,k), out[indices[i], ..., indices[k], ...] = out[indices[i], ..., indices[k], ...]  + op(updates[i, ..., k, ...]) 

Note that if multiple indices refer to the same location, the contributions from each is handled correctly. 

* **ref**  (NUMERIC type) - Initial/source variable
* **indices**  (NUMERIC type) - Indices array
* **updates**  (NUMERIC type) - Updates to add to the initial/source array

## scatterDiv
```JAVA
INDArray scatterDiv(INDArray ref, INDArray indices, INDArray updates)

SDVariable scatterDiv(SDVariable ref, SDVariable indices, SDVariable updates)
SDVariable scatterDiv(String name, SDVariable ref, SDVariable indices, SDVariable updates)
```
Scatter division operation.

If indices is rank 0 (a scalar), then out[index, ...] = out[index, ...] + op(updates[...])

If indices is rank 1 (a vector), then for each position i, out[indices[i], ...] = out[indices[i], ...] + op(updates[i, ...])

If indices is rank 2+, then for each position (i,...,k), out[indices[i], ..., indices[k], ...] = out[indices[i], ..., indices[k], ...]  + op(updates[i, ..., k, ...]) 

Note that if multiple indices refer to the same location, the contributions from each is handled correctly. 

* **ref**  (NUMERIC type) - Initial/source variable
* **indices**  (NUMERIC type) - Indices array
* **updates**  (NUMERIC type) - Updates to add to the initial/source array

## scatterMax
```JAVA
INDArray scatterMax(INDArray ref, INDArray indices, INDArray updates)

SDVariable scatterMax(SDVariable ref, SDVariable indices, SDVariable updates)
SDVariable scatterMax(String name, SDVariable ref, SDVariable indices, SDVariable updates)
```
Scatter max operation.

If indices is rank 0 (a scalar), then out[index, ...] = out[index, ...] + op(updates[...])

If indices is rank 1 (a vector), then for each position i, out[indices[i], ...] = out[indices[i], ...] + op(updates[i, ...])

If indices is rank 2+, then for each position (i,...,k), out[indices[i], ..., indices[k], ...] = out[indices[i], ..., indices[k], ...]  + op(updates[i, ..., k, ...]) 

Note that if multiple indices refer to the same location, the contributions from each is handled correctly. 

* **ref**  (NUMERIC type) - Initial/source variable
* **indices**  (NUMERIC type) - Indices array
* **updates**  (NUMERIC type) - Updates to add to the initial/source array

## scatterMin
```JAVA
INDArray scatterMin(INDArray ref, INDArray indices, INDArray updates)

SDVariable scatterMin(SDVariable ref, SDVariable indices, SDVariable updates)
SDVariable scatterMin(String name, SDVariable ref, SDVariable indices, SDVariable updates)
```
Scatter min operation.

If indices is rank 0 (a scalar), then out[index, ...] = out[index, ...] + op(updates[...])

If indices is rank 1 (a vector), then for each position i, out[indices[i], ...] = out[indices[i], ...] + op(updates[i, ...])

If indices is rank 2+, then for each position (i,...,k), out[indices[i], ..., indices[k], ...] = out[indices[i], ..., indices[k], ...]  + op(updates[i, ..., k, ...]) 

Note that if multiple indices refer to the same location, the contributions from each is handled correctly. 

* **ref**  (NUMERIC type) - Initial/source variable
* **indices**  (NUMERIC type) - Indices array
* **updates**  (NUMERIC type) - Updates to add to the initial/source array

## scatterMul
```JAVA
INDArray scatterMul(INDArray ref, INDArray indices, INDArray updates)

SDVariable scatterMul(SDVariable ref, SDVariable indices, SDVariable updates)
SDVariable scatterMul(String name, SDVariable ref, SDVariable indices, SDVariable updates)
```
Scatter multiplication operation.

If indices is rank 0 (a scalar), then out[index, ...] = out[index, ...] + op(updates[...])

If indices is rank 1 (a vector), then for each position i, out[indices[i], ...] = out[indices[i], ...] + op(updates[i, ...])

If indices is rank 2+, then for each position (i,...,k), out[indices[i], ..., indices[k], ...] = out[indices[i], ..., indices[k], ...]  + op(updates[i, ..., k, ...]) 

Note that if multiple indices refer to the same location, the contributions from each is handled correctly. 

* **ref**  (NUMERIC type) - Initial/source variable
* **indices**  (NUMERIC type) - Indices array
* **updates**  (NUMERIC type) - Updates to add to the initial/source array

## scatterSub
```JAVA
INDArray scatterSub(INDArray ref, INDArray indices, INDArray updates)

SDVariable scatterSub(SDVariable ref, SDVariable indices, SDVariable updates)
SDVariable scatterSub(String name, SDVariable ref, SDVariable indices, SDVariable updates)
```
Scatter subtraction operation.

If indices is rank 0 (a scalar), then out[index, ...] = out[index, ...] + op(updates[...])

If indices is rank 1 (a vector), then for each position i, out[indices[i], ...] = out[indices[i], ...] + op(updates[i, ...])

If indices is rank 2+, then for each position (i,...,k), out[indices[i], ..., indices[k], ...] = out[indices[i], ..., indices[k], ...]  + op(updates[i, ..., k, ...]) 

Note that if multiple indices refer to the same location, the contributions from each is handled correctly. 

* **ref**  (NUMERIC type) - Initial/source variable
* **indices**  (NUMERIC type) - Indices array
* **updates**  (NUMERIC type) - Updates to add to the initial/source array

## scatterUpdate
```JAVA
INDArray scatterUpdate(INDArray ref, INDArray indices, INDArray updates)

SDVariable scatterUpdate(SDVariable ref, SDVariable indices, SDVariable updates)
SDVariable scatterUpdate(String name, SDVariable ref, SDVariable indices, SDVariable updates)
```
Scatter update operation.

If indices is rank 0 (a scalar), then out[index, ...] = out[index, ...] + op(updates[...])

If indices is rank 1 (a vector), then for each position i, out[indices[i], ...] = out[indices[i], ...] + op(updates[i, ...])

If indices is rank 2+, then for each position (i,...,k), out[indices[i], ..., indices[k], ...] = out[indices[i], ..., indices[k], ...]  + op(updates[i, ..., k, ...]) 

Note that if multiple indices refer to the same location, the contributions from each is handled correctly. 

* **ref**  (NUMERIC type) - Initial/source variable
* **indices**  (NUMERIC type) - Indices array
* **updates**  (NUMERIC type) - Updates to add to the initial/source array

## segmentMax
```JAVA
INDArray segmentMax(INDArray data, INDArray segmentIds)

SDVariable segmentMax(SDVariable data, SDVariable segmentIds)
SDVariable segmentMax(String name, SDVariable data, SDVariable segmentIds)
```
Segment max operation.

If data =     [3, 6, 1, 4, 9, 2, 8]

segmentIds =  [0, 0, 1, 1, 1, 2, 2]

then output = [6, 9, 8] = [op(3,6), op(1,4,9), op(2,8)]

Note that the segment IDs must be sorted from smallest to largest segment.

See {unsortedSegment (String, SDVariable, SDVariable, int) ops

for the same op without this sorted requirement

* **data**  (NDARRAY type) - Data to perform segment max on
* **segmentIds**  (NUMERIC type) - Variable for the segment IDs

## segmentMean
```JAVA
INDArray segmentMean(INDArray data, INDArray segmentIds)

SDVariable segmentMean(SDVariable data, SDVariable segmentIds)
SDVariable segmentMean(String name, SDVariable data, SDVariable segmentIds)
```
Segment mean operation.

If data =     [3, 6, 1, 4, 9, 2, 8]

segmentIds =  [0, 0, 1, 1, 1, 2, 2]

then output = [6, 9, 8] = [op(3,6), op(1,4,9), op(2,8)]

Note that the segment IDs must be sorted from smallest to largest segment.

See {unsortedSegment (String, SDVariable, SDVariable, int) ops

for the same op without this sorted requirement

* **data**  (NDARRAY type) - Data to perform segment max on
* **segmentIds**  (NUMERIC type) - Variable for the segment IDs

## segmentMin
```JAVA
INDArray segmentMin(INDArray data, INDArray segmentIds)

SDVariable segmentMin(SDVariable data, SDVariable segmentIds)
SDVariable segmentMin(String name, SDVariable data, SDVariable segmentIds)
```
Segment min operation.

If data =     [3, 6, 1, 4, 9, 2, 8]

segmentIds =  [0, 0, 1, 1, 1, 2, 2]

then output = [6, 9, 8] = [op(3,6), op(1,4,9), op(2,8)]

Note that the segment IDs must be sorted from smallest to largest segment.

See {unsortedSegment (String, SDVariable, SDVariable, int) ops

for the same op without this sorted requirement

* **data**  (NDARRAY type) - Data to perform segment max on
* **segmentIds**  (NUMERIC type) - Variable for the segment IDs

## segmentProd
```JAVA
INDArray segmentProd(INDArray data, INDArray segmentIds)

SDVariable segmentProd(SDVariable data, SDVariable segmentIds)
SDVariable segmentProd(String name, SDVariable data, SDVariable segmentIds)
```
Segment product operation.

If data =     [3, 6, 1, 4, 9, 2, 8]

segmentIds =  [0, 0, 1, 1, 1, 2, 2]

then output = [6, 9, 8] = [op(3,6), op(1,4,9), op(2,8)]

Note that the segment IDs must be sorted from smallest to largest segment.

See {unsortedSegment (String, SDVariable, SDVariable, int) ops

for the same op without this sorted requirement

* **data**  (NDARRAY type) - Data to perform segment max on
* **segmentIds**  (NUMERIC type) - Variable for the segment IDs

## segmentSum
```JAVA
INDArray segmentSum(INDArray data, INDArray segmentIds)

SDVariable segmentSum(SDVariable data, SDVariable segmentIds)
SDVariable segmentSum(String name, SDVariable data, SDVariable segmentIds)
```
Segment sum operation.

If data =     [3, 6, 1, 4, 9, 2, 8]

segmentIds =  [0, 0, 1, 1, 1, 2, 2]

then output = [6, 9, 8] = [op(3,6), op(1,4,9), op(2,8)]

Note that the segment IDs must be sorted from smallest to largest segment.

See {unsortedSegment (String, SDVariable, SDVariable, int) ops

for the same op without this sorted requirement

* **data**  (NDARRAY type) - Data to perform segment max on
* **segmentIds**  (NUMERIC type) - Variable for the segment IDs

## sequenceMask
```JAVA
INDArray sequenceMask(INDArray lengths, int maxLen, DataType dataType)

SDVariable sequenceMask(SDVariable lengths, int maxLen, DataType dataType)
SDVariable sequenceMask(String name, SDVariable lengths, int maxLen, DataType dataType)
```
Generate a sequence mask (with values 0 or 1) based on the specified lengths 

Specifically, out[i, ..., k, j] = (j < lengths[i, ..., k] ? 1.0 : 0.0)

* **lengths**  (NUMERIC type) - Lengths of the sequences
* **maxLen** - Maximum sequence length
* **dataType** - 

## sequenceMask
```JAVA
INDArray sequenceMask(INDArray lengths, INDArray maxLen, DataType dataType)

SDVariable sequenceMask(SDVariable lengths, SDVariable maxLen, DataType dataType)
SDVariable sequenceMask(String name, SDVariable lengths, SDVariable maxLen, DataType dataType)
```
Generate a sequence mask (with values 0 or 1) based on the specified lengths 

Specifically, out[i, ..., k, j] = (j < lengths[i, ..., k] ? 1.0 : 0.0)

* **lengths**  (NUMERIC type) - Lengths of the sequences
* **maxLen**  (INT type) - Maximum sequence length
* **dataType** - 

## sequenceMask
```JAVA
INDArray sequenceMask(INDArray lengths, DataType dataType)

SDVariable sequenceMask(SDVariable lengths, DataType dataType)
SDVariable sequenceMask(String name, SDVariable lengths, DataType dataType)
```
see sequenceMask(String, SDVariable, SDVariable, DataType)

* **lengths**  (NUMERIC type) - 
* **dataType** - 

## shape
```JAVA
INDArray shape(INDArray input)

SDVariable shape(SDVariable input)
SDVariable shape(String name, SDVariable input)
```
Returns the shape of the specified INDArray  as a 1D INDArray 

* **input**  (NUMERIC type) - Input variable

## size
```JAVA
INDArray size(INDArray in)

SDVariable size(SDVariable in)
SDVariable size(String name, SDVariable in)
```
Returns the size (number of elements, i.e., prod(shape)) of the specified INDArray  as a 0D scalar variable

* **in**  (NUMERIC type) - Input variable

## sizeAt
```JAVA
INDArray sizeAt(INDArray in, int dimension)

SDVariable sizeAt(SDVariable in, int dimension)
SDVariable sizeAt(String name, SDVariable in, int dimension)
```
Returns a rank 0 (scalar) variable for the size of the specified dimension.

For example, if X has shape [10,20,30] then sizeAt(X,1)=20. Similarly, sizeAt(X,-1)=30

* **in**  (NUMERIC type) - Input variable
* **dimension** - Dimension to get size of

## slice
```JAVA
INDArray slice(INDArray input, int[] begin, int[] size)

SDVariable slice(SDVariable input, int[] begin, int[] size)
SDVariable slice(String name, SDVariable input, int[] begin, int[] size)
```
Get a subset of the specified input, by specifying the first element and the size of the array.

For example, if input is:

[a, b, c]

[d, e, f]

then slice(input, begin=[0,1], size=[2,1] will return:

[b]

[e]

Note that for each dimension i, begin[i] + size[i] <= input.size(i)

* **input**  (NUMERIC type) - input Variable to get subset of
* **begin** - Beginning index. Must be same length as rank of input array (Size: AtLeast(min=1))
* **size** - Size of the output array. Must be same length as rank of input array (Size: AtLeast(min=1))

## slice
```JAVA
INDArray slice(INDArray input, INDArray begin, INDArray size)

SDVariable slice(SDVariable input, SDVariable begin, SDVariable size)
SDVariable slice(String name, SDVariable input, SDVariable begin, SDVariable size)
```
Get a subset of the specified input, by specifying the first element and the size of the array.

For example, if input is:

[a, b, c]

[d, e, f]

then slice(input, begin=[0,1], size=[2,1] will return:

[b]

[e]

Note that for each dimension i, begin[i] + size[i] <= input.size(i)

* **input**  (NUMERIC type) - input Variable to get subset of
* **begin**  (INT type) - Beginning index. Must be same length as rank of input array
* **size**  (INT type) - Size of the output array. Must be same length as rank of input array

## squaredNorm
```JAVA
INDArray squaredNorm(INDArray x, boolean keepDims, int[] dimensions)

SDVariable squaredNorm(SDVariable x, boolean keepDims, int[] dimensions)
SDVariable squaredNorm(String name, SDVariable x, boolean keepDims, int[] dimensions)
INDArray squaredNorm(INDArray x, int[] dimensions)

SDVariable squaredNorm(SDVariable x, int[] dimensions)
SDVariable squaredNorm(String name, SDVariable x, int[] dimensions)
```
Squared L2 norm: see norm2(String, SDVariable, boolean, int...)

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **x**  (NUMERIC type) - 
* **keepDims** -  - default = false
* **dimensions** -  (Size: AtLeast(min=0))

## squeeze
```JAVA
INDArray squeeze(INDArray x, int axis)

SDVariable squeeze(SDVariable x, int axis)
SDVariable squeeze(String name, SDVariable x, int axis)
```
Remove a single dimension of size 1.

For example, if input has shape [a,b,1,c] then squeeze(input, 2) returns an array of shape [a,b,c]

* **x**  (NUMERIC type) - Input variable
* **axis** - Size 1 dimension to remove

## stack
```JAVA
INDArray stack(INDArray values, int axis)

SDVariable stack(SDVariable values, int axis)
SDVariable stack(String name, SDVariable values, int axis)
```
Stack a set of N INDArray of rank X into one rank X+1 variable.

If inputs have shape [a,b,c] then output has shape:

axis = 0: [N,a,b,c]

axis = 1: [a,N,b,c]

axis = 2: [a,b,N,c]

axis = 3: [a,b,c,N]

see unstack(String[], SDVariable, int, int)

* **values**  (NDARRAY type) - Input variables to stack. Must have the same shape for all inputs
* **axis** - Axis to stack on

## standardDeviation
```JAVA
INDArray standardDeviation(INDArray x, boolean biasCorrected, boolean keepDims, int[] dimensions)

SDVariable standardDeviation(SDVariable x, boolean biasCorrected, boolean keepDims, int[] dimensions)
SDVariable standardDeviation(String name, SDVariable x, boolean biasCorrected, boolean keepDims, int[] dimensions)
INDArray standardDeviation(INDArray x, boolean biasCorrected, int[] dimensions)

SDVariable standardDeviation(SDVariable x, boolean biasCorrected, int[] dimensions)
SDVariable standardDeviation(String name, SDVariable x, boolean biasCorrected, int[] dimensions)
```
Stardard deviation array reduction operation, optionally along specified dimensions

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **x**  (NUMERIC type) - Input variable
* **biasCorrected** - If true: divide by (N-1) (i.e., sample stdev). If false: divide by N (population stdev)
* **keepDims** - If true: keep the dimensions that are reduced on (as size 1). False: remove the reduction dimensions - default = false
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## stridedSlice
```JAVA
INDArray stridedSlice(INDArray in, long[] begin, long[] end, long[] strides, int beginMask, int endMask, int ellipsisMask, int newAxisMask, int shrinkAxisMask)

SDVariable stridedSlice(SDVariable in, long[] begin, long[] end, long[] strides, int beginMask, int endMask, int ellipsisMask, int newAxisMask, int shrinkAxisMask)
SDVariable stridedSlice(String name, SDVariable in, long[] begin, long[] end, long[] strides, int beginMask, int endMask, int ellipsisMask, int newAxisMask, int shrinkAxisMask)
INDArray stridedSlice(INDArray in, long[] begin, long[] end, long[] strides)

SDVariable stridedSlice(SDVariable in, long[] begin, long[] end, long[] strides)
SDVariable stridedSlice(String name, SDVariable in, long[] begin, long[] end, long[] strides)
```
Get a subset of the specified input, by specifying the first element, last element, and the strides.

For example, if input is:

[a, b, c]

[d, e, f]

[g, h, i]

then stridedSlice(input, begin=[0,1], end=[2,2], strides=[2,1], all masks = 0) will return:

[b, c]

[h, i]

* **in**  (NUMERIC type) - Variable to get subset of
* **begin** - Beginning index (Size: AtLeast(min=1))
* **end** - End index (Size: AtLeast(min=1))
* **strides** - Stride ("step size") for each dimension. For example, stride of 2 means take every second element. (Size: AtLeast(min=1))
* **beginMask** - Bit mask: If the ith bit is set to 1, then the value in the begin long[] is ignored, and a value of 0 is used instead for the beginning index for that dimension - default = 0
* **endMask** - Bit mask: If the ith bit is set to 1, then the value in the end long[] is ignored, and a value of size(i)-1 is used instead for the end index for that dimension - default = 0
* **ellipsisMask** - Bit mask: only one non-zero value is allowed here. If a non-zero value is set, then other dimensions are inserted as required at the specified position - default = 0
* **newAxisMask** - Bit mask: if the ith bit is set to 1, then the begin/end/stride values are ignored, and a size 1 dimension is inserted at this point - default = 0
* **shrinkAxisMask** - Bit mask: if the ith bit is set to 1, then the begin/end/stride values are ignored, and a size 1 dimension is removed at this point. Note that begin/end/stride values must result in a size 1 output for these dimensions - default = 0

## sum
```JAVA
INDArray sum(INDArray x, boolean keepDims, int[] dimensions)

SDVariable sum(SDVariable x, boolean keepDims, int[] dimensions)
SDVariable sum(String name, SDVariable x, boolean keepDims, int[] dimensions)
INDArray sum(INDArray x, int[] dimensions)

SDVariable sum(SDVariable x, int[] dimensions)
SDVariable sum(String name, SDVariable x, int[] dimensions)
```
Sum array reduction operation, optionally along specified dimensions.

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **x**  (NUMERIC type) - Input variable
* **keepDims** - If true: keep the dimensions that are reduced on (as length 1). False: remove the reduction dimensions - default = false
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## tensorMmul
```JAVA
INDArray tensorMmul(INDArray x, INDArray y, int[] dimensionsX, int[] dimensionsY, boolean transposeX, boolean transposeY, boolean transposeZ)

SDVariable tensorMmul(SDVariable x, SDVariable y, int[] dimensionsX, int[] dimensionsY, boolean transposeX, boolean transposeY, boolean transposeZ)
SDVariable tensorMmul(String name, SDVariable x, SDVariable y, int[] dimensionsX, int[] dimensionsY, boolean transposeX, boolean transposeY, boolean transposeZ)
INDArray tensorMmul(INDArray x, INDArray y, int[] dimensionsX, int[] dimensionsY)

SDVariable tensorMmul(SDVariable x, SDVariable y, int[] dimensionsX, int[] dimensionsY)
SDVariable tensorMmul(String name, SDVariable x, SDVariable y, int[] dimensionsX, int[] dimensionsY)
```
//TODO: Ops must be documented.

* **x**  (NUMERIC type) - Input variable x
* **y**  (NUMERIC type) - Input variable y
* **dimensionsX** - dimensions for first input array (x) (Size: AtLeast(min=1))
* **dimensionsY** - dimensions for second input array (y) (Size: AtLeast(min=1))
* **transposeX** - Transpose x (first argument) - default = false
* **transposeY** - Transpose y (second argument) - default = false
* **transposeZ** - Transpose result array - default = false

## tile
```JAVA
INDArray tile(INDArray x, INDArray repeat)

SDVariable tile(SDVariable x, SDVariable repeat)
SDVariable tile(String name, SDVariable x, SDVariable repeat)
```
Repeat (tile) the input tensor the specified number of times.

For example, if input is

[1, 2]

[3, 4]

and repeat is [2, 3]

then output is

[1, 2, 1, 2, 1, 2]

[3, 4, 3, 4, 3, 4]

[1, 2, 1, 2, 1, 2]

[3, 4, 3, 4, 3, 4]

* **x**  (NDARRAY type) - Input variable
* **repeat**  (INT type) - Number of times to repeat in each axis. Must have length equal to the rank of the input array

## tile
```JAVA
INDArray tile(INDArray x, int[] repeat)

SDVariable tile(SDVariable x, int[] repeat)
SDVariable tile(String name, SDVariable x, int[] repeat)
```
see tile(String, SDVariable, int...)

* **x**  (NDARRAY type) - 
* **repeat** -  (Size: AtLeast(min=1))

## transpose
```JAVA
INDArray transpose(INDArray x)

SDVariable transpose(SDVariable x)
SDVariable transpose(String name, SDVariable x)
```
Matrix transpose operation: If input has shape [a,b] output has shape [b,a]

* **x**  (NDARRAY type) - Input variable

## unsortedSegmentMax
```JAVA
INDArray unsortedSegmentMax(INDArray data, INDArray segmentIds, int numSegments)

SDVariable unsortedSegmentMax(SDVariable data, SDVariable segmentIds, int numSegments)
SDVariable unsortedSegmentMax(String name, SDVariable data, SDVariable segmentIds, int numSegments)
```
Unsorted segment max operation. As per segmentMax(String, SDVariable, SDVariable) but without

the requirement for the indices to be sorted.

If data =     [1, 3, 2, 6, 4, 9, 8]

segmentIds =  [1, 0, 2, 0, 1, 1, 2]

then output = [6, 9, 8] = [max(3,6), max(1,4,9), max(2,8)]

* **data**  (NUMERIC type) - Data (variable) to perform unsorted segment max on
* **segmentIds**  (NUMERIC type) - Variable for the segment IDs
* **numSegments** - Number of segments

## unsortedSegmentMean
```JAVA
INDArray unsortedSegmentMean(INDArray data, INDArray segmentIds, int numSegments)

SDVariable unsortedSegmentMean(SDVariable data, SDVariable segmentIds, int numSegments)
SDVariable unsortedSegmentMean(String name, SDVariable data, SDVariable segmentIds, int numSegments)
```
Unsorted segment mean operation. As per segmentMean(String, SDVariable, SDVariable) but without

the requirement for the indices to be sorted.

If data =     [1, 3, 2, 6, 4, 9, 8]

segmentIds =  [1, 0, 2, 0, 1, 1, 2]

then output = [4.5, 4.666, 5] = [mean(3,6), mean(1,4,9), mean(2,8)]

* **data**  (NUMERIC type) - Data (variable) to perform unsorted segment max on
* **segmentIds**  (NUMERIC type) - Variable for the segment IDs
* **numSegments** - Number of segments

## unsortedSegmentMin
```JAVA
INDArray unsortedSegmentMin(INDArray data, INDArray segmentIds, int numSegments)

SDVariable unsortedSegmentMin(SDVariable data, SDVariable segmentIds, int numSegments)
SDVariable unsortedSegmentMin(String name, SDVariable data, SDVariable segmentIds, int numSegments)
```
Unsorted segment min operation. As per segmentMin(String, SDVariable, SDVariable) but without

the requirement for the indices to be sorted.

If data =     [1, 3, 2, 6, 4, 9, 8]

segmentIds =  [1, 0, 2, 0, 1, 1, 2]

then output = [3, 1, 2] = [min(3,6), min(1,4,9), min(2,8)]

* **data**  (NUMERIC type) - Data (variable) to perform unsorted segment max on
* **segmentIds**  (NUMERIC type) - Variable for the segment IDs
* **numSegments** - Number of segments

## unsortedSegmentProd
```JAVA
INDArray unsortedSegmentProd(INDArray data, INDArray segmentIds, int numSegments)

SDVariable unsortedSegmentProd(SDVariable data, SDVariable segmentIds, int numSegments)
SDVariable unsortedSegmentProd(String name, SDVariable data, SDVariable segmentIds, int numSegments)
```
Unsorted segment product operation. As per segmentProd(String, SDVariable, SDVariable) but without

the requirement for the indices to be sorted.

If data =     [1, 3, 2, 6, 4, 9, 8]

segmentIds =  [1, 0, 2, 0, 1, 1, 2]

then output = [4.5, 4.666, 5] = [mean(3,6), mean(1,4,9), mean(2,8)]

* **data**  (NUMERIC type) - Data (variable) to perform unsorted segment max on
* **segmentIds**  (NUMERIC type) - Variable for the segment IDs
* **numSegments** - Number of segments

## unsortedSegmentSqrtN
```JAVA
INDArray unsortedSegmentSqrtN(INDArray data, INDArray segmentIds, int numSegments)

SDVariable unsortedSegmentSqrtN(SDVariable data, SDVariable segmentIds, int numSegments)
SDVariable unsortedSegmentSqrtN(String name, SDVariable data, SDVariable segmentIds, int numSegments)
```
Unsorted segment sqrtN operation. Simply returns the sqrt of the count of the number of values in each segment

If data =     [1, 3, 2, 6, 4, 9, 8]

segmentIds =  [1, 0, 2, 0, 1, 1, 2]

then output = [1.414, 1.732, 1.414] = [sqrt(2), sqrtN(3), sqrtN(2)]

* **data**  (NUMERIC type) - Data (variable) to perform unsorted segment max on
* **segmentIds**  (NUMERIC type) - Variable for the segment IDs
* **numSegments** - Number of segments

## unsortedSegmentSum
```JAVA
INDArray unsortedSegmentSum(INDArray data, INDArray segmentIds, int numSegments)

SDVariable unsortedSegmentSum(SDVariable data, SDVariable segmentIds, int numSegments)
SDVariable unsortedSegmentSum(String name, SDVariable data, SDVariable segmentIds, int numSegments)
```
Unsorted segment sum operation. As per segmentSum(String, SDVariable, SDVariable) but without

the requirement for the indices to be sorted.

If data =     [1, 3, 2, 6, 4, 9, 8]

segmentIds =  [1, 0, 2, 0, 1, 1, 2]

then output = [9, 14, 10] = [sum(3,6), sum(1,4,9), sum(2,8)]

* **data**  (NUMERIC type) - Data (variable) to perform unsorted segment max on
* **segmentIds**  (NUMERIC type) - Variable for the segment IDs
* **numSegments** - Number of segments

## unstack
```JAVA
void unstack(INDArray value, int axis, int num)

void unstack(SDVariable value, int axis, int num)
void unstack(String name, SDVariable value, int axis, int num)
```
Unstack a variable of rank X into N rank X-1 variables by taking slices along the specified axis.

If input has shape [a,b,c] then output has shape:

axis = 0: [b,c]

axis = 1: [a,c]

axis = 2: [a,b]

* **value**  (NDARRAY type) - Input variable to unstack
* **axis** - Axis to unstack on
* **num** - Number of output variables

## variance
```JAVA
INDArray variance(INDArray x, boolean biasCorrected, boolean keepDims, int[] dimensions)

SDVariable variance(SDVariable x, boolean biasCorrected, boolean keepDims, int[] dimensions)
SDVariable variance(String name, SDVariable x, boolean biasCorrected, boolean keepDims, int[] dimensions)
INDArray variance(INDArray x, boolean biasCorrected, int[] dimensions)

SDVariable variance(SDVariable x, boolean biasCorrected, int[] dimensions)
SDVariable variance(String name, SDVariable x, boolean biasCorrected, int[] dimensions)
```
Variance array reduction operation, optionally along specified dimensions

Note that if keepDims = true, the output variable has the same rank as the input variable,

with the reduced dimensions having size 1. This can be useful for later broadcast operations (such as subtracting

the mean along a dimension).

Example: if input has shape [a,b,c] and dimensions=[1] then output has shape:

keepDims = true: [a,1,c]

keepDims = false: [a,c]

* **x**  (NUMERIC type) - Input variable
* **biasCorrected** - If true: divide by (N-1) (i.e., sample variable). If false: divide by N (population variance)
* **keepDims** - If true: keep the dimensions that are reduced on (as size 1). False: remove the reduction dimensions - default = false
* **dimensions** - Dimensions to reduce over. If dimensions are not specified, full array reduction is performed (Size: AtLeast(min=0))

## zerosLike
```JAVA
INDArray zerosLike(INDArray input)

SDVariable zerosLike(SDVariable input)
SDVariable zerosLike(String name, SDVariable input)
```
Return a variable of all 0s, with the same shape as the input variable. Note that this is dynamic:

if the input shape changes in later execution, the returned variable's shape will also be updated

* **input**  (NUMERIC type) - Input 


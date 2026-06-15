---
title: "Matrix Manipulation"
description: "Reshaping, transposing, permuting, concatenating, sorting, and other shape manipulation operations on INDArrays"
---

## Overview

Shape manipulation covers all operations that change how an `INDArray` is structured — its rank, its dimension sizes, the order of its axes, or how multiple arrays are combined into one. These operations are frequent in neural network code (batching, flattening activations, preparing inputs) and in general scientific computing.

**Views vs. copies is the central concern.** Many shape operations return a *view* — a new `INDArray` object backed by the same off-heap memory buffer as the original. Modifying a view modifies the original. Operations that return copies are explicitly labeled. When in doubt, call `.dup()` on the result to guarantee independence.


## Reshape

Reshape gives an array a new shape without changing its element values or the order they appear in the underlying buffer. The product of all dimensions must remain identical.

### Basic reshape

```java
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.ndarray.INDArray;

// 12-element source
INDArray src = Nd4j.arange(12);
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]

INDArray mat = src.reshape(3, 4);   // 3 rows x 4 cols
/*
[[         0,    1.0000,    2.0000,    3.0000],
 [    4.0000,    5.0000,    6.0000,    7.0000],
 [    8.0000,    9.0000,   10.0000,   11.0000]]
*/

INDArray cube = src.reshape(2, 3, 2);  // rank-3
/*
[[[         0,    1.0000],
  [    2.0000,    3.0000],
  [    4.0000,    5.0000]],

 [[    6.0000,    7.0000],
  [    8.0000,    9.0000],
  [   10.0000,   11.0000]]]
*/
```

### Specifying memory order

```java
// 'c' = row-major (default), 'f' = column-major
INDArray cOrder = src.reshape('c', 3, 4);
INDArray fOrder = src.reshape('f', 3, 4);
// The elements traversed when reading row-by-row differ between the two.
```

### reshape returns a view (shared data!)

`reshape` tries to return a view whenever the layout allows it. When ND4J can satisfy the new shape without rearranging bytes, the returned array shares the original buffer:

```java
INDArray original = Nd4j.arange(6).reshape(2, 3);
INDArray flat     = original.reshape(6);     // view when possible

flat.putScalar(0, -99.0);          // modifies shared buffer

System.out.println(original.getDouble(0, 0));  // -99.0  — original changed!
```

If the requested layout requires data movement (e.g., converting F-order to a C-order reshape), ND4J allocates a new buffer. **Never assume reshape is always a view or always a copy.** Call `.dup()` if you need an independent result:

```java
INDArray safeCopy = original.reshape(6).dup();  // always an independent copy
```

### Using -1 as a wildcard

Pass `-1` for one dimension to let ND4J infer it:

```java
INDArray x = Nd4j.arange(24);
INDArray a = x.reshape(-1, 6);   // shape [4, 6] — 24 / 6 = 4
INDArray b = x.reshape(4, -1);   // shape [4, 6]
INDArray c = x.reshape(2, 3, -1); // shape [2, 3, 4]
```


## Transpose

For a 2D matrix, transpose swaps rows and columns: element `[i, j]` moves to `[j, i]`. The diagonal is unchanged.

### Out-of-place transpose (returns a view)

```java
INDArray mat = Nd4j.createFromArray(new float[][]{{1,2,3},{4,5,6}});
// shape [2, 3]:
// [[1, 2, 3],
//  [4, 5, 6]]

INDArray t = mat.transpose();
// shape [3, 2]:
// [[1, 4],
//  [2, 5],
//  [3, 6]]
```

`transpose()` returns a **view** with reordered strides. Mutating `t` mutates `mat`:

```java
t.putScalar(new long[]{0, 0}, 99.0);
System.out.println(mat.getDouble(0, 0));  // 99.0
```

For an independent transposed copy:

```java
INDArray tCopy = mat.transpose().dup();
```

### In-place transpose

`transposei()` modifies the array's strides in place, making it its own transpose. The underlying data buffer is not copied:

```java
INDArray m = Nd4j.arange(6).reshape(2, 3);
m.transposei();
// m is now shape [3, 2] — the same object, no new allocation
```

Use `transposei()` when you are done with the original shape and want to avoid even the lightweight view-object allocation of `transpose()`.

### Non-square matrices

```java
INDArray wide = Nd4j.create(new float[]{1,2,3,4,5,6,7,8,9,10,11,12},
                             new int[]{2, 6});
// shape [2, 6]:
// [[1, 3, 5, 7,  9, 11],
//  [2, 4, 6, 8, 10, 12]]

INDArray tall = wide.transpose();
// shape [6, 2]:
// [[1,  2],
//  [3,  4],
//  [5,  6],
//  [7,  8],
//  [9, 10],
//  [11,12]]
```


## Permute

For arrays with more than two dimensions, `permute(int...)` reorders axes in an arbitrary way. Pass the desired axis order as integers.

```java
// Shape [2, 3, 4]
INDArray x = Nd4j.arange(24).reshape(2, 3, 4);

// Move axis 2 to position 0: result shape [4, 2, 3]
INDArray p = x.permute(2, 0, 1);
// p.shape() == [4, 2, 3]

// For a 4D tensor [batch, channels, height, width]
// Permute to [batch, height, width, channels] (channels-last layout):
INDArray bchw  = Nd4j.rand(8, 3, 64, 64);
INDArray bhwc  = bchw.permute(0, 2, 3, 1);
// bhwc.shape() == [8, 64, 64, 3]
```

`permute` always returns a **view** — strides are reordered but the buffer is shared. For a copy with contiguous memory:

```java
INDArray contiguous = bchw.permute(0, 2, 3, 1).dup();
```

Transpose on a 2D array is equivalent to `permute(1, 0)`.


## Ravel and Flatten

Both operations reduce an array to one dimension. They differ in whether they share data.

### ravel() — view when possible

```java
INDArray mat = Nd4j.rand(3, 4);   // shape [3, 4], 12 elements

INDArray flat = mat.ravel();
// flat.shape() == [12]
// flat shares the buffer with mat when mat is C-contiguous
```

Because `ravel()` may return a view, writing into the result can change `mat`:

```java
flat.putScalar(0, -1.0);
System.out.println(mat.getDouble(0, 0));  // -1.0 (shared buffer)
```

### Nd4j.toFlattened() — always a copy

When you need a guaranteed copy, or when you want to flatten multiple arrays into one in a specified order:

```java
// Flatten one array with explicit order
INDArray copy = Nd4j.toFlattened('c', mat);         // row-major copy
INDArray fCopy = Nd4j.toFlattened('f', mat);        // column-major copy

// Flatten and concatenate several arrays into one vector
INDArray a = Nd4j.arange(6).reshape(2, 3);
INDArray b = Nd4j.arange(6, 12).reshape(3, 2);
INDArray combined = Nd4j.toFlattened('c', a, b);
// combined.shape() == [12] — all 12 elements of a then b, row-major
// combined.getDouble(0) == 0.0, combined.getDouble(6) == 6.0
```

`toFlattened` always allocates a new buffer regardless of input layout.


## Concatenation

### Nd4j.concat — along any dimension

```java
INDArray x = Nd4j.ones(2, 3);    // 2x3 of ones
INDArray y = Nd4j.zeros(4, 3);   // 4x3 of zeros

// Concatenate along dimension 0 (stack rows): shape [6, 3]
INDArray rows = Nd4j.concat(0, x, y);

// Concatenate along dimension 1 (stack columns): arrays must have same row count
INDArray a = Nd4j.ones(3, 2);
INDArray b = Nd4j.zeros(3, 5);
INDArray cols = Nd4j.concat(1, a, b);   // shape [3, 7]
```

`concat` accepts a varargs list so you can combine more than two arrays:

```java
INDArray all = Nd4j.concat(0, arr1, arr2, arr3, arr4);
```

### Nd4j.vstack — vertical stack (along rows)

`vstack` is shorthand for `concat(0, ...)`. All arrays must have the same number of columns.

```java
INDArray top    = Nd4j.rand(2, 4);
INDArray bottom = Nd4j.rand(3, 4);

INDArray stacked = Nd4j.vstack(top, bottom);   // shape [5, 4]
```

### Nd4j.hstack — horizontal stack (along columns)

`hstack` is shorthand for `concat(1, ...)`. All arrays must have the same number of rows.

```java
INDArray left  = Nd4j.rand(3, 2);
INDArray right = Nd4j.rand(3, 5);

INDArray side = Nd4j.hstack(left, right);   // shape [3, 7]
```

`concat`, `vstack`, and `hstack` always return **copies** — the new array has its own buffer.


## Stack and Unstack

Where `concat` joins arrays along an existing dimension, `stack` creates a new dimension. This is useful for batching multiple samples into a single tensor.

### Nd4j.stack

```java
INDArray sample1 = Nd4j.rand(3, 4);   // one sample, shape [3, 4]
INDArray sample2 = Nd4j.rand(3, 4);
INDArray sample3 = Nd4j.rand(3, 4);

// Stack along a new dimension 0 (batch dimension): shape [3, 3, 4]
INDArray batch = Nd4j.stack(0, sample1, sample2, sample3);
// batch.shape() == [3, 3, 4]
// batch.slice(0) is sample1 equivalent

// Stack along dimension 1: shape [3, 3, 4] — axes interpreted differently
INDArray altBatch = Nd4j.stack(1, sample1, sample2, sample3);
```

All inputs must have exactly the same shape. The result has rank one higher than the inputs.

### Unstacking

To reverse the operation and split a batched tensor back into individual slices, use `Nd4j.unstack`:

```java
INDArray[] samples = Nd4j.unstack(batch, 0, 3);
// samples[0].shape() == [3, 4]
// samples[1].shape() == [3, 4]
// samples[2].shape() == [3, 4]
```

The second argument is the axis along which to unstack; the third is the number of arrays to produce.


## Squeeze and Unsqueeze

### Squeeze — remove size-1 dimensions

`Nd4j.squeeze(INDArray, int dimension)` removes the specified dimension if its size is 1. This commonly arises after reductions that preserve dimensions.

```java
// A column vector stored as shape [4, 1]
INDArray col = Nd4j.rand(4, 1);
// col.shape() == [4, 1]

INDArray squeezed = Nd4j.squeeze(col, 1);   // remove dimension 1
// squeezed.shape() == [4]

// A 1x1x5 tensor
INDArray singleton = Nd4j.rand(1, 1, 5);
INDArray flat = Nd4j.squeeze(singleton, 0);   // shape [1, 5]
flat = Nd4j.squeeze(flat, 0);                  // shape [5]
```

Squeezing a dimension whose size is not 1 throws an exception.

### Unsqueeze — add a size-1 dimension

`reshape` is the most direct way to insert a size-1 dimension:

```java
INDArray vec = Nd4j.arange(5);        // shape [5]

// Add a batch dimension at position 0: shape [1, 5]
INDArray row = vec.reshape(1, 5);

// Treat as a column vector: shape [5, 1]
INDArray col = vec.reshape(5, 1);

// Add a dimension in the middle of a 3D array [2, 4, 6] -> [2, 1, 4, 6]
INDArray x    = Nd4j.rand(2, 4, 6);
INDArray xExp = x.reshape(2, 1, 4, 6);
```

The result of `reshape` may be a view, so the same shared-data caution applies.


## Sort

### Sort along a dimension

`Nd4j.sort` sorts elements along a specified axis in ascending or descending order. It returns a new sorted array.

```java
INDArray x = Nd4j.createFromArray(new float[][]{{3, 1, 4}, {1, 5, 9}, {2, 6, 5}});
/*
[[3, 1, 4],
 [1, 5, 9],
 [2, 6, 5]]
*/

// Sort each row (along dimension 1), ascending
INDArray sortedRows = Nd4j.sort(x, 1, true);
/*
[[1, 3, 4],
 [1, 5, 9],
 [2, 5, 6]]
*/

// Sort each column (along dimension 0), descending
INDArray sortedCols = Nd4j.sort(x, 0, false);
/*
[[3, 6, 9],
 [2, 5, 5],
 [1, 1, 4]]
*/
```

The boolean third argument is `true` for ascending, `false` for descending. `Nd4j.sort` returns a **copy**.

### sortRows and sortColumns

`Nd4j.sortRows(INDArray, int column, boolean ascending)` and `Nd4j.sortColumns(INDArray, int row, boolean ascending)` sort the rows (or columns) of a 2D array by the values in a specified column (or row):

```java
INDArray mat = Nd4j.createFromArray(new float[][]{{3, 7}, {1, 2}, {5, 4}});
/*
[[3, 7],
 [1, 2],
 [5, 4]]
*/

// Sort rows by the values in column 0, ascending
INDArray byFirstCol = Nd4j.sortRows(mat, 0, true);
/*
[[1, 2],
 [3, 7],
 [5, 4]]
*/

// Sort columns by the values in row 0, descending
INDArray byFirstRow = Nd4j.sortColumns(mat, 0, false);
/*
[[7, 3],
 [2, 1],
 [4, 5]]
*/
```

Both methods return copies. Rows or columns are moved as units, preserving their internal structure.


## Repeat

`INDArray.repeat(int dimension, long... repeats)` tiles an array's elements along the specified dimension.

```java
INDArray vec = Nd4j.createFromArray(1.0f, 2.0f, 3.0f);   // shape [3]

// Repeat each element twice along dimension 0
INDArray rep = vec.repeat(0, 2);
// [1, 1, 2, 2, 3, 3]   shape [6]

INDArray mat = Nd4j.createFromArray(new float[][]{{1, 2}, {3, 4}});
/*
[[1, 2],
 [3, 4]]
*/

// Repeat each row twice (dimension 0)
INDArray repRows = mat.repeat(0, 2);
/*
[[1, 2],
 [1, 2],
 [3, 4],
 [3, 4]]
shape [4, 2]
*/

// Repeat each column element twice (dimension 1)
INDArray repCols = mat.repeat(1, 2);
/*
[[1, 1, 2, 2],
 [3, 3, 4, 4]]
shape [2, 4]
*/
```

`repeat` returns a **copy**.


## Pad

`Nd4j.pad(INDArray, int[][], PadMode)` adds border elements around an array. This is used in convolution operations (zero-padding) and in batching sequences to a uniform length.

```java
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.ops.impl.transforms.Pad.Mode;

INDArray x = Nd4j.createFromArray(new float[][]{{1, 2, 3}, {4, 5, 6}});
// shape [2, 3]

// Pad 1 row on each side (top/bottom) and 2 columns on each side (left/right)
// padWidth[i][0] = pad before on dimension i
// padWidth[i][1] = pad after on dimension i
int[][] padWidth = {{1, 1}, {2, 2}};

// Constant mode: fill padding with zeros (the default constant is 0)
INDArray padded = Nd4j.pad(x, padWidth, Nd4j.PadMode.CONSTANT);
/*
shape [4, 7]:
[[0, 0, 0, 0, 0, 0, 0],
 [0, 0, 1, 2, 3, 0, 0],
 [0, 0, 4, 5, 6, 0, 0],
 [0, 0, 0, 0, 0, 0, 0]]
*/

// Reflect mode: mirror the edge values
INDArray reflected = Nd4j.pad(x, new int[][]{{1, 1}, {1, 1}}, Nd4j.PadMode.REFLECT);
/*
shape [4, 5]:
[[5, 4, 5, 6, 5],
 [2, 1, 2, 3, 2],
 [5, 4, 5, 6, 5],
 [2, 1, 2, 3, 2]]
*/
```

Available `PadMode` values: `CONSTANT` (fill with a fixed value), `REFLECT` (mirror without repeating the edge), `SYMMETRIC` (mirror repeating the edge), `EDGE` (replicate the edge value).

`Nd4j.pad` always returns a **copy**.


## Swap Axes

`swapAxes(int dim1, int dim2)` exchanges two dimensions of an array. For a 2D matrix this is identical to `transpose()`. It is more useful for higher-rank tensors.

```java
// Shape [2, 3, 4]
INDArray x = Nd4j.arange(24).reshape(2, 3, 4);

// Swap axes 0 and 2: result shape [4, 3, 2]
INDArray swapped = x.swapAxes(0, 2);
// swapped.shape() == [4, 3, 2]

// For a 2D matrix, swapAxes(0, 1) == transpose()
INDArray mat = Nd4j.rand(3, 5);
INDArray t1  = mat.transpose();
INDArray t2  = mat.swapAxes(0, 1);
// t1 and t2 have the same content
```

`swapAxes` returns a **view** with reordered strides, not a copy. The same shared-data caution applies.


## Views vs. Copies Reference

Getting this right prevents subtle bugs. The table below summarises which operations return views and which always allocate new memory.

| Operation | Returns |
|---|---|
| `reshape(long...)` | View when layout permits; copy otherwise |
| `reshape(char, long...)` | View when layout permits; copy otherwise |
| `transpose()` | View |
| `transposei()` | In-place (same object) |
| `permute(int...)` | View |
| `swapAxes(int, int)` | View |
| `ravel()` | View when array is C-contiguous; copy otherwise |
| `Nd4j.toFlattened(char, INDArray...)` | Always a copy |
| `Nd4j.concat(int, INDArray...)` | Always a copy |
| `Nd4j.vstack(INDArray...)` | Always a copy |
| `Nd4j.hstack(INDArray...)` | Always a copy |
| `Nd4j.stack(int, INDArray...)` | Always a copy |
| `Nd4j.unstack(INDArray, int, int)` | Always copies |
| `Nd4j.squeeze(INDArray, int)` | Copy |
| `repeat(int, long...)` | Always a copy |
| `Nd4j.pad(INDArray, int[][], PadMode)` | Always a copy |
| `Nd4j.sort(INDArray, int, boolean)` | Always a copy |
| `Nd4j.sortRows / sortColumns` | Always copies |

When you are unsure, check by modifying the result and seeing whether the original changes. Alternatively, call `.dup()` on any operation result to guarantee an independent copy at the cost of a memory allocation.

```java
// Defensive pattern: force a copy from any operation
INDArray independentResult = anyShapeOp(arr).dup();
```


## Practical Example: Preparing a Batch

This example shows how shape manipulation operations compose in a typical deep learning workflow — loading individual samples and assembling them into a batched tensor.

```java
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.ndarray.INDArray;

// Simulate three images, each stored as a flat vector of 12 values
INDArray img1 = Nd4j.arange(12).castTo(org.nd4j.linalg.api.buffer.DataType.FLOAT);
INDArray img2 = Nd4j.arange(12, 24).castTo(org.nd4j.linalg.api.buffer.DataType.FLOAT);
INDArray img3 = Nd4j.arange(24, 36).castTo(org.nd4j.linalg.api.buffer.DataType.FLOAT);

// Reshape each to [channels=1, height=3, width=4]
INDArray t1 = img1.reshape(1, 3, 4);
INDArray t2 = img2.reshape(1, 3, 4);
INDArray t3 = img3.reshape(1, 3, 4);

// Stack into a batch: [batch=3, channels=1, height=3, width=4]
INDArray batch = Nd4j.stack(0, t1, t2, t3);
System.out.println(java.util.Arrays.toString(batch.shape()));
// [3, 1, 3, 4]

// Permute to channels-last for frameworks that expect [batch, height, width, channels]
INDArray batchChannelsLast = batch.permute(0, 2, 3, 1).dup();
System.out.println(java.util.Arrays.toString(batchChannelsLast.shape()));
// [3, 3, 4, 1]

// Sort batch by the value at position [sample, 0, 0, 0] — just to demonstrate sortRows
// Flatten to [3, 12] first
INDArray flat = batch.reshape(3, 12);
INDArray sortedBatch = Nd4j.sortRows(flat, 0, true);
```


## API Reference Links

- [INDArray.reshape](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#reshape-long...-)
- [INDArray.transpose](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#transpose--)
- [INDArray.permute](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#permute-int...-)
- [INDArray.ravel](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#ravel--)
- [INDArray.repeat](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#repeat-int-long...-)
- [INDArray.swapAxes](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#swapAxes-int-int-)
- [Nd4j.concat](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#concat-int-org.nd4j.linalg.api.ndarray.INDArray...-)
- [Nd4j.hstack](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#hstack-org.nd4j.linalg.api.ndarray.INDArray...-)
- [Nd4j.vstack](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#vstack-org.nd4j.linalg.api.ndarray.INDArray...-)
- [Nd4j.toFlattened](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#toFlattened-char-org.nd4j.linalg.api.ndarray.INDArray...-)
- [Nd4j.squeeze](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#squeeze-org.nd4j.linalg.api.ndarray.INDArray-int-)
- [Nd4j.sort](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#sort-org.nd4j.linalg.api.ndarray.INDArray-int-boolean-)
- [Nd4j.sortRows](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#sortRows-org.nd4j.linalg.api.ndarray.INDArray-int-boolean-)
- [Nd4j.sortColumns](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#sortColumns-org.nd4j.linalg.api.ndarray.INDArray-int-boolean-)
- [Nd4j.pad](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#pad-org.nd4j.linalg.api.ndarray.INDArray-int:A:A-org.nd4j.linalg.factory.Nd4j.PadMode-)

---
title: "Creating NDArrays"
description: "All methods for creating INDArrays — factory methods, from Java arrays, random, combining, and typed creation"
---

## Overview

Every NDArray in ND4J is created through static factory methods on `org.nd4j.linalg.factory.Nd4j`. There is no public constructor. This page covers every creation path available in M2.1, from simple zeros to multi-dimensional arrays built from Java primitives, combined arrays, and typed arrays.

**Key M2.1 change:** The global `DataBuffer.Type` enum used to configure a single default type for the whole JVM has been replaced by the per-array `DataType` enum. Each array carries its own type, and the default type for new arrays is `FLOAT`. See [Typed Creation](#typed-creation) for details.

Standard imports used throughout this page:

```java
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.api.buffer.DataType;
```


## Factory Methods

### Nd4j.zeros()

Creates an array filled entirely with zeros. The shape is given as a varargs list of `long` dimensions, or as a `long[]` shape array.

```java
// 1D: five zeros
INDArray v = Nd4j.zeros(5);
System.out.println(v);
// [         0,         0,         0,         0,         0]

// 2D: 3 rows, 4 columns
INDArray m = Nd4j.zeros(3, 4);
System.out.println(m);
// [[         0,         0,         0,         0],
//  [         0,         0,         0,         0],
//  [         0,         0,         0,         0]]

// Shape given as an array
long[] shape = {2, 3, 4};
INDArray t = Nd4j.zeros(shape);
// rank 3, shape [2, 3, 4], 24 elements, all zero
```

The default `DataType` is `FLOAT`. To request a specific type pass it as the first argument:

```java
INDArray d = Nd4j.zeros(DataType.DOUBLE, 3, 4);
System.out.println(d.dataType());  // DOUBLE
```

### Nd4j.ones()

Creates an array filled entirely with ones. Same shape syntax as `zeros`.

```java
INDArray ones = Nd4j.ones(2, 3);
System.out.println(ones);
// [[1.0000, 1.0000, 1.0000],
//  [1.0000, 1.0000, 1.0000]]

// With explicit DataType
INDArray onesDouble = Nd4j.ones(DataType.DOUBLE, 4, 4);
```

### Nd4j.valueArrayOf()

Creates an array of a given shape where every element has the same scalar value. This is the most direct way to fill an array with a constant without a follow-up scalar add.

```java
// All elements equal to 7.0
INDArray sevens = Nd4j.valueArrayOf(new long[]{3, 3}, 7.0);
System.out.println(sevens);
// [[7.0000, 7.0000, 7.0000],
//  [7.0000, 7.0000, 7.0000],
//  [7.0000, 7.0000, 7.0000]]

// Equivalent but two-step:
INDArray alt = Nd4j.zeros(3, 3).addi(7.0);
```

You can also use `zeros` combined with in-place operations for derived fill values:

```java
// All elements equal to 10
INDArray tens = Nd4j.zeros(3, 5).addi(10);
```


## Random Arrays

### Uniform random: Nd4j.rand()

Produces an array with values drawn uniformly from `[0, 1)`.

```java
// 2D: rows and columns as separate arguments
INDArray u2d = Nd4j.rand(3, 4);
System.out.println(u2d);
// [[0.7231, 0.0892, 0.4563, 0.1720],
//  [0.3388, 0.9174, 0.2041, 0.8853],
//  [0.5510, 0.6632, 0.0075, 0.3317]]

// 3D or higher: pass a shape array
int[] shape = {2, 3, 4};
INDArray u3d = Nd4j.rand(shape);

// With explicit DataType
INDArray uDouble = Nd4j.rand(DataType.DOUBLE, 3, 4);
System.out.println(uDouble.dataType());  // DOUBLE
```

### Gaussian random: Nd4j.randn()

Produces an array with values drawn from a standard normal distribution N(0, 1) — mean zero, standard deviation one.

```java
// 2D Gaussian
INDArray g2d = Nd4j.randn(3, 4);
System.out.println(g2d);
// [[ 0.3145, -1.2081,  0.7932, -0.0443],
//  [-0.5521,  1.1034, -0.2178,  0.8820],
//  [ 0.0671, -0.3952,  0.4417, -1.6193]]

// Shape array form
int[] shape = {5, 5};
INDArray g = Nd4j.randn(shape);

// With DataType
INDArray gDouble = Nd4j.randn(DataType.DOUBLE, 4, 4);
```

### Seeding the random number generator

For reproducible experiments, seed ND4J's random number generator before creating random arrays:

```java
Nd4j.getRandom().setSeed(12345L);
INDArray a = Nd4j.rand(3, 3);

Nd4j.getRandom().setSeed(12345L);
INDArray b = Nd4j.rand(3, 3);

// a and b have identical values
System.out.println(a.equals(b));  // true
```


## From Java Arrays

### Nd4j.createFromArray()

`createFromArray` is the modern, overloaded method for creating NDArrays directly from Java primitive arrays. It infers the shape automatically from the array dimensions and has overloads for `double`, `float`, `int`, and `long` in 1D through 4D.

**1D arrays:**

```java
double[] d1 = {1.0, 2.0, 3.0, 4.0, 5.0};
INDArray arr1d = Nd4j.createFromArray(d1);
System.out.println(arr1d);
// [1.0000, 2.0000, 3.0000, 4.0000, 5.0000]

float[] f1 = {1.0f, 2.0f, 3.0f};
INDArray arrFloat = Nd4j.createFromArray(f1);

int[] i1 = {10, 20, 30};
INDArray arrInt = Nd4j.createFromArray(i1);
System.out.println(arrInt.dataType());  // INT32

long[] l1 = {100L, 200L, 300L};
INDArray arrLong = Nd4j.createFromArray(l1);
System.out.println(arrLong.dataType());  // INT64
```

**2D arrays:**

```java
double[][] d2 = {
    {1.0, 2.0, 3.0},
    {4.0, 5.0, 6.0},
    {7.0, 8.0, 9.0}
};
INDArray arr2d = Nd4j.createFromArray(d2);
System.out.println(arr2d);
// [[1.0000, 2.0000, 3.0000],
//  [4.0000, 5.0000, 6.0000],
//  [7.0000, 8.0000, 9.0000]]
System.out.println(arr2d.shape());  // [3, 3]

float[][] f2 = {{1f, 2f}, {3f, 4f}};
INDArray arrF2 = Nd4j.createFromArray(f2);

int[][] i2 = {{1, 2, 3}, {4, 5, 6}};
INDArray arrI2 = Nd4j.createFromArray(i2);

long[][] l2 = {{10L, 20L}, {30L, 40L}};
INDArray arrL2 = Nd4j.createFromArray(l2);
```

**3D and 4D arrays** follow the same pattern — `createFromArray` has overloads for `float[][][]`, `double[][][]`, `int[][][]`, `long[][][]`, and their 4D equivalents.

### Nd4j.create() from flat Java arrays

The older `Nd4j.create` methods accept a flat Java array and an optional explicit shape. These remain fully supported.

```java
// 1D row vector
double[] data = {1.0, 2.0, 3.0, 4.0, 5.0};
INDArray rowVec = Nd4j.create(data);
System.out.println(rowVec);
// [1.0000, 2.0000, 3.0000, 4.0000, 5.0000]

// Column vector: provide shape [length, 1]
INDArray colVec = Nd4j.create(data, new int[]{5, 1});
System.out.println(colVec);
// [[1.0000],
//  [2.0000],
//  [3.0000],
//  [4.0000],
//  [5.0000]]

// 2D from a float array — shape is inferred from Java array structure
float[][] f2 = {{1f, 2f, 3f}, {4f, 5f, 6f}};
INDArray m2 = Nd4j.create(f2);
System.out.println(m2);
// [[1.0000, 2.0000, 3.0000],
//  [4.0000, 5.0000, 6.0000]]

// 2D from double[][]
double[][] d2 = {{9.0, 8.0}, {7.0, 6.0}};
INDArray m2d = Nd4j.create(d2);
```

### Higher-dimensional arrays from nested Java arrays

For 3D and deeper structures using raw Java arrays, the standard approach is to flatten the nested array into a 1D buffer and supply the shape explicitly:

```java
import org.nd4j.linalg.util.ArrayUtil;

double[][][] d3 = {
    {{1, 2}, {3, 4}},
    {{5, 6}, {7, 8}}
};

double[] flat = ArrayUtil.flattenDoubleArray(d3);
int[] shape = {2, 2, 2};
INDArray arr3d = Nd4j.create(flat, shape, 'c');
System.out.println(arr3d.shapeInfoToString());  // [2, 2, 2]
System.out.println(arr3d);
// [[[1.0000, 2.0000],
//   [3.0000, 4.0000]],
//
//  [[5.0000, 6.0000],
//   [7.0000, 8.0000]]]
```

The `'c'` argument specifies C (row-major) order. Use `'f'` for Fortran (column-major) order. For 3D and 4D input, `createFromArray` is generally easier to use and should be preferred where available.


## From Other NDArrays

### dup() — deep copy

`dup()` returns a completely independent copy of the array. The copy and the original share no underlying memory — modifying one does not affect the other.

```java
INDArray original = Nd4j.createFromArray(new float[][]{{1, 2, 3}, {4, 5, 6}});
INDArray copy = original.dup();

copy.putScalar(new int[]{0, 0}, 99.0);
System.out.println(original.getFloat(0, 0));  // 1.0 — original unchanged
System.out.println(copy.getFloat(0, 0));      // 99.0
```

By default `dup()` uses C order. To dup with a specific memory order:

```java
INDArray fOrderCopy = original.dup('f');  // Fortran-order copy
INDArray cOrderCopy = original.dup('c');  // C-order copy (the default)
```

### getRow() and getColumn() — views

`getRow(int i)` and `getColumn(int j)` return **views** of the original array. Modifying the returned view modifies the original.

```java
INDArray matrix = Nd4j.createFromArray(new double[][]{
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
});

INDArray row1 = matrix.getRow(1);   // view of [4, 5, 6]
row1.addi(10);                       // in-place add on the view

System.out.println(matrix);
// [[1.0000,  2.0000,  3.0000],
//  [14.0000, 15.0000, 16.0000],   <-- modified through the view
//  [7.0000,  8.0000,  9.0000]]
```

To get an independent copy of a row, use `getRow(i).dup()`:

```java
INDArray rowCopy = matrix.getRow(0).dup();  // independent copy
rowCopy.addi(100);
// matrix row 0 is unchanged
```

### Views from get() and NDArrayIndex

`get(NDArrayIndex...)` returns a view of any sub-array:

```java
import org.nd4j.linalg.indexing.NDArrayIndex;

INDArray m = Nd4j.linspace(1, 12, 12).reshape(3, 4);
// [[ 1,  2,  3,  4],
//  [ 5,  6,  7,  8],
//  [ 9, 10, 11, 12]]

// Rows 0 and 1, all columns — returns a view
INDArray top2 = m.get(NDArrayIndex.interval(0, 2), NDArrayIndex.all());
System.out.println(top2);
// [[1.0000, 2.0000, 3.0000,  4.0000],
//  [5.0000, 6.0000, 7.0000,  8.0000]]

// Column 2, all rows
INDArray col2 = m.get(NDArrayIndex.all(), NDArrayIndex.point(2));
System.out.println(col2);
// [3.0000, 7.0000, 11.0000]
```

These are views — use `.dup()` when you need an independent copy.


## Combining Arrays

### Nd4j.hstack() — horizontal stack

`hstack` concatenates arrays along dimension 1 (columns). All input arrays must have the same number of rows.

```java
INDArray zeros = Nd4j.zeros(2, 2);
INDArray ones  = Nd4j.ones(2, 2);

INDArray hstacked = Nd4j.hstack(ones, zeros);
System.out.println(hstacked);
// [[1.0000, 1.0000, 0.0000, 0.0000],
//  [1.0000, 1.0000, 0.0000, 0.0000]]
// Shape: [2, 4]
```

### Nd4j.vstack() — vertical stack

`vstack` concatenates arrays along dimension 0 (rows). All input arrays must have the same number of columns.

```java
INDArray zeros = Nd4j.zeros(2, 2);
INDArray ones  = Nd4j.ones(2, 2);

INDArray vstacked = Nd4j.vstack(ones, zeros);
System.out.println(vstacked);
// [[1.0000, 1.0000],
//  [1.0000, 1.0000],
//  [0.0000, 0.0000],
//  [0.0000, 0.0000]]
// Shape: [4, 2]
```

Both `hstack` and `vstack` accept varargs, so you can stack more than two arrays at once:

```java
INDArray a = Nd4j.ones(2, 3);
INDArray b = Nd4j.zeros(2, 3);
INDArray c = Nd4j.valueArrayOf(new long[]{2, 3}, 5.0);

INDArray tall = Nd4j.vstack(a, b, c);  // shape [6, 3]
```

### Nd4j.concat() — concatenate along any dimension

`concat(int dimension, INDArray... arrays)` generalises hstack and vstack to any dimension.

```java
INDArray zeros = Nd4j.zeros(2, 2);
INDArray ones  = Nd4j.ones(2, 2);

// Along dimension 0 — same result as vstack
INDArray along0 = Nd4j.concat(0, zeros, ones);
System.out.println(along0);
// [[0.0000, 0.0000],
//  [0.0000, 0.0000],
//  [1.0000, 1.0000],
//  [1.0000, 1.0000]]

// Along dimension 1 — same result as hstack
INDArray along1 = Nd4j.concat(1, zeros, ones);
System.out.println(along1);
// [[0.0000, 0.0000, 1.0000, 1.0000],
//  [0.0000, 0.0000, 1.0000, 1.0000]]
```

`concat` also works on higher-rank arrays. For a set of rank-3 arrays with shape `[batch, height, width]`, concatenating on dimension 0 combines them along the batch axis.

### Nd4j.pad() — pad an array

`pad` surrounds an array with constant values (zero by default). The padding amounts are specified per dimension.

```java
INDArray ones = Nd4j.ones(2, 2);

// Pad 1 element on each side of each dimension
INDArray padded = Nd4j.pad(ones, new int[]{1, 1}, Nd4j.PadMode.CONSTANT);
System.out.println(padded);
// [[0.0000, 0.0000, 0.0000, 0.0000],
//  [0.0000, 1.0000, 1.0000, 0.0000],
//  [0.0000, 1.0000, 1.0000, 0.0000],
//  [0.0000, 0.0000, 0.0000, 0.0000]]
// Shape: [4, 4]
```

The `PadMode.CONSTANT` mode fills the padded region with zeros. Other modes (e.g., `PadMode.REFLECT`, `PadMode.SYMMETRIC`) mirror existing values into the padded region.


## Special Creation Methods

### Nd4j.eye() — identity matrix

Creates an NxN identity matrix: ones on the main diagonal, zeros everywhere else.

```java
INDArray eye3 = Nd4j.eye(3);
System.out.println(eye3);
// [[1.0000, 0.0000, 0.0000],
//  [0.0000, 1.0000, 0.0000],
//  [0.0000, 0.0000, 1.0000]]
```

### Nd4j.linspace() — evenly spaced values

`linspace(start, stop, count)` generates `count` evenly spaced values from `start` to `stop` inclusive.

```java
// 5 values from 1 to 5 inclusive
INDArray ls = Nd4j.linspace(1, 5, 5);
System.out.println(ls);
// [1.0000, 2.0000, 3.0000, 4.0000, 5.0000]

// 5 values from 0 to 1
INDArray ls2 = Nd4j.linspace(0, 1, 5);
System.out.println(ls2);
// [0.0000, 0.2500, 0.5000, 0.7500, 1.0000]

// With explicit DataType
INDArray lsDouble = Nd4j.linspace(DataType.DOUBLE, 0.0, 10.0, 100);
```

Linspace is commonly combined with `reshape` to produce initialised matrices of arbitrary shape:

```java
// 5x5 matrix with values 1 to 25
INDArray grid = Nd4j.linspace(1, 25, 25).reshape(5, 5);
System.out.println(grid);
// [[ 1.0000,  2.0000,  3.0000,  4.0000,  5.0000],
//  [ 6.0000,  7.0000,  8.0000,  9.0000, 10.0000],
//  [11.0000, 12.0000, 13.0000, 14.0000, 15.0000],
//  [16.0000, 17.0000, 18.0000, 19.0000, 20.0000],
//  [21.0000, 22.0000, 23.0000, 24.0000, 25.0000]]

// Evaluate sin over 100 points
import static org.nd4j.linalg.ops.transforms.Transforms.sin;
INDArray x = Nd4j.linspace(DataType.DOUBLE, 0.0, Math.PI, 100);
INDArray y = sin(x);
```

### Nd4j.arange() — integer-spaced values

`arange` creates a 1D array of consecutive integers, following the same convention as NumPy's `arange`.

```java
// [0, 1, 2, 3, 4]
INDArray a0 = Nd4j.arange(5);
System.out.println(a0);
// [         0,    1.0000,    2.0000,    3.0000,    4.0000]

// [2, 3, 4, 5, 6] — start inclusive, stop exclusive
INDArray a1 = Nd4j.arange(2, 7);
System.out.println(a1);
// [    2.0000,    3.0000,    4.0000,    5.0000,    6.0000]
```

`arange` is often combined with `reshape` to produce structured matrices:

```java
// 3x4 matrix with values 0..11
INDArray m = Nd4j.arange(12).reshape(3, 4);
System.out.println(m);
// [[         0,    1.0000,    2.0000,    3.0000],
//  [    4.0000,    5.0000,    6.0000,    7.0000],
//  [    8.0000,    9.0000,   10.0000,   11.0000]]
```

### Nd4j.diag() — diagonal matrix or vector

`diag` has two complementary behaviours depending on the rank of the input:

- If the input is a **vector** (rank 1), `diag` produces an NxN matrix with those values on the main diagonal.
- If the input is a **matrix** (rank 2), `diag` extracts the main diagonal and returns a vector.

```java
// Vector to diagonal matrix
INDArray v = Nd4j.createFromArray(new double[]{1.0, 2.0, 3.0});
INDArray diagMatrix = Nd4j.diag(v);
System.out.println(diagMatrix);
// [[1.0000, 0.0000, 0.0000],
//  [0.0000, 2.0000, 0.0000],
//  [0.0000, 0.0000, 3.0000]]

// Matrix to diagonal vector
INDArray extracted = Nd4j.diag(diagMatrix);
System.out.println(extracted);
// [1.0000, 2.0000, 3.0000]
```


## Typed Creation

### The DataType enum

In M2.1, every `INDArray` carries its own `DataType`. You can read it at any time:

```java
INDArray a = Nd4j.zeros(3, 4);
System.out.println(a.dataType());   // FLOAT (the default)

INDArray b = Nd4j.zeros(DataType.DOUBLE, 3, 4);
System.out.println(b.dataType());   // DOUBLE
```

### Passing DataType to creation methods

All major creation methods accept a `DataType` as the first argument. When omitted, the default is `FLOAT`.

```java
// Zeros
INDArray zF  = Nd4j.zeros(DataType.FLOAT,   5);
INDArray zD  = Nd4j.zeros(DataType.DOUBLE,  5);
INDArray zI  = Nd4j.zeros(DataType.INT32,   5);
INDArray zL  = Nd4j.zeros(DataType.INT64,   3, 4);

// Ones
INDArray oF  = Nd4j.ones(DataType.FLOAT,    2, 3);
INDArray oD  = Nd4j.ones(DataType.DOUBLE,   2, 3);

// Random
INDArray rF  = Nd4j.rand(DataType.FLOAT,    4, 4);
INDArray rD  = Nd4j.rand(DataType.DOUBLE,   4, 4);
INDArray rnD = Nd4j.randn(DataType.DOUBLE,  4, 4);

// Linspace
INDArray lsD = Nd4j.linspace(DataType.DOUBLE, 0.0, 1.0, 10);
```

### Casting an existing array

If you already have an array but need a different type, use `castTo`:

```java
INDArray floatArr  = Nd4j.rand(3, 4);           // FLOAT
INDArray doubleArr = floatArr.castTo(DataType.DOUBLE);
INDArray intArr    = floatArr.castTo(DataType.INT32);

System.out.println(doubleArr.dataType());  // DOUBLE
System.out.println(intArr.dataType());     // INT32
```

`castTo` returns a new array; the original is unchanged.

### Setting the global default type

To change the default for all subsequent array creation in your application, call this once during startup before creating any arrays:

```java
Nd4j.setDefaultDataTypes(DataType.DOUBLE, DataType.INT64);
```

The two arguments are the default floating-point type and the default integer type respectively. After this call, `Nd4j.zeros(3, 4)` will produce a `DOUBLE` array.

### Migration from beta4: DataBuffer.Type is gone

In releases prior to M2.1, the type was controlled globally via `DataBuffer.Type`:

```java
// OLD — does not compile in M2.1
Nd4j.setDataType(DataBuffer.Type.DOUBLE);
```

In M2.1, `DataBuffer.Type` no longer exists. Use per-array `DataType` arguments instead:

```java
// NEW — M2.1
INDArray arr = Nd4j.zeros(DataType.DOUBLE, 3, 4);
```


## Empty Arrays

`Nd4j.empty(DataType)` creates a zero-element array of the given type. This is useful as a sentinel value or as a placeholder that can be detected with `isEmpty()`.

```java
INDArray empty = Nd4j.empty(DataType.FLOAT);
System.out.println(empty.isEmpty());   // true
System.out.println(empty.length());    // 0
System.out.println(empty.dataType());  // FLOAT
```

Empty arrays of a specific shape (with zero in one or more dimensions) can be constructed with `zeros` by including a zero dimension:

```java
// Shape [0, 4] — zero rows, four columns
INDArray emptyRows = Nd4j.zeros(DataType.FLOAT, 0, 4);
System.out.println(emptyRows.shape());   // [0, 4]
System.out.println(emptyRows.length());  // 0
```


## Quick Reference

| Goal | Method |
|---|---|
| All zeros | `Nd4j.zeros(rows, cols)` |
| All ones | `Nd4j.ones(rows, cols)` |
| Constant fill | `Nd4j.valueArrayOf(shape, value)` |
| Uniform random [0,1) | `Nd4j.rand(rows, cols)` |
| Gaussian N(0,1) | `Nd4j.randn(rows, cols)` |
| From `double[][]` | `Nd4j.createFromArray(double[][])` |
| From `float[][]` | `Nd4j.createFromArray(float[][])` |
| From `int[][]` | `Nd4j.createFromArray(int[][])` |
| From `long[][]` | `Nd4j.createFromArray(long[][])` |
| Row vector from `double[]` | `Nd4j.create(double[])` |
| Column vector from `double[]` | `Nd4j.create(double[], new int[]{n,1})` |
| 3D+ from nested Java array | `ArrayUtil.flattenDoubleArray` + `Nd4j.create(flat, shape, 'c')` |
| Deep copy | `arr.dup()` |
| Row view | `arr.getRow(i)` |
| Row copy | `arr.getRow(i).dup()` |
| Horizontal stack | `Nd4j.hstack(a, b)` |
| Vertical stack | `Nd4j.vstack(a, b)` |
| Concat along axis | `Nd4j.concat(dim, a, b)` |
| Pad with zeros | `Nd4j.pad(arr, padding, Nd4j.PadMode.CONSTANT)` |
| Identity matrix | `Nd4j.eye(n)` |
| Evenly spaced values | `Nd4j.linspace(start, stop, count)` |
| Integer range | `Nd4j.arange(start, stop)` |
| Diagonal matrix/vector | `Nd4j.diag(arr)` |
| Typed zeros | `Nd4j.zeros(DataType.DOUBLE, rows, cols)` |
| Empty array | `Nd4j.empty(DataType.FLOAT)` |
| Change type | `arr.castTo(DataType.DOUBLE)` |

---

See [Tensors and NDArrays](../core-concepts/tensors-and-ndarrays) for the full description of rank, shape, stride, and memory layout. See the Operations page for how to manipulate and compute with arrays once created.

---
title: "Tensors and NDArrays"
description: "INDArray fundamentals — shape, rank, stride, DataType, creating arrays, views vs copies, and off-heap memory in ND4J"
---

## What is an INDArray?

`INDArray` is the fundamental data structure in ND4J. It represents an N-dimensional array — a tensor — that can hold numerical data in any number of dimensions. Every operation in ND4J, and by extension all of DL4J, produces and consumes `INDArray` instances.

The Java interface is `org.nd4j.linalg.api.ndarray.INDArray`. You never instantiate this directly; instead you use the static factory methods on `org.nd4j.linalg.factory.Nd4j`.

### Key Properties

Every `INDArray` has four defining properties:

**Rank** — the number of dimensions. A scalar has rank 0, a vector rank 1, a matrix rank 2, and so on. There is no hard upper limit on rank.

**Shape** — a `long[]` giving the size of each dimension. A 3x4x5 array has shape `[3, 4, 5]`. The shape determines what indices are valid when you access elements.

**Length** — the total number of elements, which is the product of the shape dimensions. A 3x4x5 array has length 60.

**Stride** — a `long[]` giving the separation in the underlying flat buffer between consecutive elements along each dimension. Stride is what makes views (slices, transposes, sub-arrays) possible without copying data. For a C-order (row-major) 3x4 matrix the strides are `[4, 1]` — moving one row costs 4 positions in the buffer, moving one column costs 1.

### Indexing Convention

ND4J uses zero-based indexing throughout. For a 2D array (matrix):

- Dimension 0 is the row dimension
- Dimension 1 is the column dimension

This generalises for higher ranks: dimension 0 is the outermost (slowest-varying) dimension in C order.


## DataType

Every `INDArray` has a numeric type, represented by the `org.nd4j.linalg.api.buffer.DataType` enum. Choosing the right type matters for precision, memory use, and hardware compatibility.

### Available Types in M2.1

**Floating point:**

| Preferred Name | Bits | Notes |
|---|---|---|
| `DOUBLE` | 64 | Double-precision float |
| `FLOAT` | 32 | Single-precision float (default) |
| `FLOAT16` | 16 | Half-precision float |
| `BFLOAT16` | 16 | Brain float, wider exponent range than FLOAT16 |

**Signed integer:**

| Preferred Name | Bits |
|---|---|
| `INT64` | 64 |
| `INT32` | 32 |
| `INT16` | 16 |
| `INT8` | 8 |

**Unsigned integer:**

| Preferred Name | Bits |
|---|---|
| `UINT64` | 64 |
| `UINT32` | 32 |
| `UINT16` | 16 |
| `UINT8` | 8 |

**Other:** `BOOL`, `UTF8`

### Deprecated Aliases

The following names still compile but you should prefer the names above in new code:

| Old name | Preferred name |
|---|---|
| `HALF` | `FLOAT16` |
| `LONG` | `INT64` |
| `INT` | `INT32` |
| `SHORT` | `INT16` |
| `BYTE` | `INT8` |
| `UBYTE` | `UINT8` |

`FLOAT16` and `HALF` are the same enum constant — `FLOAT16` is a static alias for `HALF`. The same relationship holds for the integer pairs (`INT32`/`INT`, `INT64`/`LONG`, etc.). Both names are interchangeable at the JVM level, but the preferred names make code easier to read.

### Migration Note from beta4

If you are migrating from an earlier beta release, note that `DataBuffer.Type.DOUBLE` and `DataBuffer.Type.FLOAT` no longer exist. Replace all uses of `DataBuffer.Type` with `DataType`:

```java
// OLD (beta4) — does not compile in M2.1
DataBuffer.Type.DOUBLE

// NEW (M2.1)
DataType.DOUBLE
```

### Default DataType and Global Configuration

The default type for new arrays is `FLOAT`. You can change this globally at application startup:

```java
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.factory.Nd4j;

// Set both the default floating-point and integer types
Nd4j.setDefaultDataTypes(DataType.FLOAT, DataType.INT32);
```

Calling this once before any array creation ensures consistent types throughout your application.


## Creating NDArrays

All array creation goes through static methods on `Nd4j`. The most commonly used ones are shown below.

### Zeros and Ones

```java
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.api.ndarray.INDArray;

INDArray zeros = Nd4j.zeros(DataType.FLOAT, 3, 5);   // 3x5, all zeros
INDArray ones  = Nd4j.ones(DataType.FLOAT, 2, 4);    // 2x4, all ones
```

### Scalar-Filled Arrays

Use `zeros` combined with an in-place add to fill with any constant:

```java
INDArray tens = Nd4j.zeros(3, 5).addi(10);   // 3x5, all values 10
```

### From Java Arrays

```java
// 2D from a Java 2D array — shape is inferred automatically
INDArray fromArr = Nd4j.createFromArray(new float[][]{{1, 2, 3}, {4, 5, 6}});

// 1D row vector from a flat Java array
INDArray rowVec = Nd4j.create(new float[]{1, 2, 3});

// Column vector: pass shape explicitly
INDArray colVec = Nd4j.create(new float[]{1, 2, 3}, new int[]{3, 1});
```

### Random Arrays

```java
// Uniform random values in [0, 1)
INDArray uniformRand = Nd4j.rand(DataType.FLOAT, 3, 4);

// Standard normal distribution N(0, 1)
INDArray gaussianRand = Nd4j.randn(DataType.FLOAT, 3, 4);
```

### Linspace

```java
// 100 evenly-spaced points from 0 to 10 (inclusive)
INDArray lin = Nd4j.linspace(DataType.FLOAT, 0, 10, 100);
```

### Identity Matrix

```java
// 5x5 identity matrix (ones on diagonal, zeros elsewhere)
INDArray eye = Nd4j.eye(5);
```

### Stacking and Concatenation

```java
// Horizontal stack — arrays must have the same number of rows
INDArray hstacked = Nd4j.hstack(arr1, arr2);

// Vertical stack — arrays must have the same number of columns
INDArray vstacked = Nd4j.vstack(arr1, arr2);

// Concatenate along any dimension
INDArray concatenated = Nd4j.concat(0, arr1, arr2);   // along rows
INDArray sideBy = Nd4j.concat(1, arr1, arr2);          // along columns
```


## Shape Introspection

Once you have an array, these methods tell you its structure:

```java
long[] shape    = arr.shape();        // e.g., [3, 4, 5]
int    rank     = arr.rank();          // 3
long   length   = arr.length();        // 60

// 2D-specific convenience methods
long   rows     = arr.rows();          // 3
long   cols     = arr.columns();       // 4

// Size of a specific dimension
long   dimSize  = arr.size(2);         // 5

// Type checks
boolean isMatrix = arr.isMatrix();     // true if rank == 2
boolean isVector = arr.isVector();     // true if rank == 1, or if one dim is 1
boolean isScalar = arr.isScalar();     // true if rank == 0 or length == 1
```

Note that `rows()` and `columns()` are only meaningful for 2D arrays. For higher-rank tensors, use `shape()` and `size(dimension)`.


## Views vs. Copies

Understanding the difference between a view and a copy is one of the most important concepts in ND4J. Getting it wrong leads to bugs that are hard to track down.

### Views Share Underlying Memory

Many operations return a **view** — a new `INDArray` object that refers to the same backing memory buffer as the original. Modifying the view modifies the original:

```java
INDArray matrix = Nd4j.createFromArray(new float[][]{{1,2,3},{4,5,6},{7,8,9}});

INDArray row = matrix.getRow(0);  // view of first row: [1, 2, 3]
row.addi(10);                     // modify in-place

// row is now [11, 12, 13]
// matrix row 0 is ALSO [11, 12, 13] — same underlying memory!
System.out.println(matrix.getRow(0));  // prints [11.0000, 12.0000, 13.0000]
```

Operations that commonly return views: `getRow`, `getColumn`, `reshape`, `transpose`, `get` with `NDArrayIndex`, sub-array slicing.

### Copies with dup()

If you need an independent copy, call `.dup()`:

```java
INDArray rowCopy = matrix.getRow(1).dup();  // independent copy of row 1
rowCopy.addi(100);                          // modifies only rowCopy

// matrix is unchanged
System.out.println(matrix.getRow(1));  // still [4.0000, 5.0000, 6.0000]
```

### In-Place vs. Out-of-Place Operations

ND4J follows a naming convention for element-wise operations:

- Methods ending in **`i`** (e.g., `addi`, `muli`, `subi`) are **in-place**: they modify the array they are called on and return the same object.
- Methods without the `i` suffix (e.g., `add`, `mul`, `sub`) are **out-of-place**: they allocate a new array and leave the original unchanged.

```java
INDArray x = Nd4j.create(new float[]{1, 2, 3});
INDArray y = Nd4j.create(new float[]{4, 5, 6});

// Out-of-place: x is NOT modified
INDArray z = x.add(y);
// z is [5, 7, 9]
// x is still [1, 2, 3]

// In-place: x IS modified
INDArray w = x.addi(y);
// w is [5, 7, 9]
// x is ALSO [5, 7, 9]
// w == x (literally the same Java object returned)
```

The in-place variants avoid allocating a new array, which matters when performance is critical. However, be careful when calling in-place operations on views — you will modify the original array's data.

### Transpose is a View

`transpose()` returns a view with modified strides, not a copy:

```java
INDArray mat = Nd4j.rand(DataType.FLOAT, 3, 4);
INDArray t   = mat.transpose();   // view — same data, different shape/strides

// t.shape() is [4, 3]
// Modifying t via in-place operations modifies mat
```

If you need an independent transposed copy: `mat.transpose().dup()`.


## Memory Layout

### Off-Heap Storage

NDArrays in ND4J are stored in **off-heap memory** — outside the JVM's garbage-collected heap. This is managed via JavaCPP and direct native memory. The benefits are significant:

- **BLAS interoperability**: Native linear algebra libraries (OpenBLAS, cuBLAS) can operate directly on the data without copying.
- **No 2^31 element limit**: Java arrays are limited to `Integer.MAX_VALUE` elements. Off-heap buffers have no such restriction.
- **No GC pressure**: Large arrays do not contribute to GC pauses because they are not on the JVM heap. However, you must be aware of memory lifecycle — see the Memory and Workspaces page for details on workspace-based memory management.

### C Order and F Order

ND4J supports two memory layouts for 2D and higher-rank arrays:

- **C order (row-major)**: Elements within a row are contiguous in memory. This is the default. Consistent with NumPy's default and with C arrays.
- **F order (column-major)**: Elements within a column are contiguous in memory. This is Fortran order, used internally by some BLAS routines.

Most operations work correctly regardless of order. You can check the order with `arr.ordering()` which returns `'c'` or `'f'`. If you need a specific order, use `arr.dup('c')` or `arr.dup('f')`.

For most users, the default C order is the right choice.


## Getting and Setting Values

### Individual Elements

```java
// Get a single value as a double (works regardless of array's DataType)
double val = arr.getDouble(row, col);

// Get as float
float f = arr.getFloat(row, col);

// Set a single value
arr.putScalar(new int[]{row, col}, 3.14);

// For higher-rank arrays, provide all indices
arr.putScalar(new int[]{batch, channel, height, width}, 0.5);
```

### Rows and Columns

```java
// Get a row as a view (modifying it modifies arr)
INDArray row = arr.getRow(i);

// Replace a row with another array
arr.putRow(i, someRowVector);

// Get and replace columns similarly
INDArray col = arr.getColumn(j);
arr.putColumn(j, someColumnVector);
```

### Sub-Arrays with NDArrayIndex

`NDArrayIndex` gives you flexible sub-array access:

```java
import org.nd4j.linalg.indexing.NDArrayIndex;

// Rows 0..2 (exclusive of 3), all columns
INDArray sub = arr.get(NDArrayIndex.interval(0, 3), NDArrayIndex.all());

// Single row at index 2, columns 1..3 (exclusive of 4)
INDArray specific = arr.get(NDArrayIndex.point(2), NDArrayIndex.interval(1, 4));

// Last element along a dimension
INDArray last = arr.get(NDArrayIndex.point(arr.rows() - 1), NDArrayIndex.all());
```

`NDArrayIndex.interval` and `NDArrayIndex.point` return views by default. Use `.dup()` if you need a copy.


## Basic Operations Preview

The full operations reference is covered in the Operations page. Here is a quick overview of the most common patterns.

### Element-Wise Arithmetic

```java
INDArray sum     = a.add(b);   // element-wise addition, new array
INDArray diff    = a.sub(b);   // element-wise subtraction
INDArray product = a.mul(b);   // element-wise multiplication
INDArray quot    = a.div(b);   // element-wise division
```

### Matrix Multiplication

```java
// a is [m x k], b is [k x n], result is [m x n]
INDArray result = a.mmul(b);
```

### Reductions

```java
// Total sum as a scalar Java number
double total = arr.sumNumber().doubleValue();

// Sum along dimension 0 (collapse rows, result has shape [1, cols])
INDArray colSums = arr.sum(0);

// Mean along dimension 1 (collapse columns, result has shape [rows, 1])
INDArray rowMeans = arr.mean(1);

// Max and min
INDArray colMaxes = arr.max(0);
double   globalMax = arr.maxNumber().doubleValue();
```

### Shape Transformations

```java
// Transpose (view — see Views section above)
INDArray transposed = arr.transpose();

// Reshape — total elements must be the same
// For a [3, 4] array (12 elements):
INDArray reshaped = arr.reshape(2, 6);    // [2, 6] — same 12 elements
INDArray flat     = arr.reshape(1, 12);   // flatten to [1, 12]

// Flatten to 1D
INDArray flat1d = arr.reshape(arr.length());
```

### Scalar Operations

```java
// Add a scalar to every element
INDArray plusOne = arr.add(1.0);

// Multiply every element by a scalar
INDArray doubled = arr.mul(2.0);

// In-place scalar operations
arr.addi(1.0);
arr.muli(2.0);
```

---

For memory management and workspace configuration, see the Memory and Workspaces page. For the full catalog of mathematical operations including transforms, linear algebra, and reductions, see the Operations page.

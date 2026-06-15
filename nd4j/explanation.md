---
title: "ND4J Overview"
description: "Architecture, INDArray interface, memory model, data types, and backend system for the ND4J tensor library"
---

# ND4J Overview

ND4J is the tensor computation library for the JVM. Every numerical operation in the DL4J ecosystem — from simple element-wise arithmetic to distributed GPU training — is executed through ND4J. This page explains what ND4J is, how its memory model works, how the backend system is structured, and how it relates to SameDiff and the rest of the DL4J stack.

---

## What ND4J Is

ND4J stands for N-Dimensional Arrays for Java. It is a scientific computing library designed for production JVM applications, serving the same role that NumPy serves in the Python ecosystem, but with first-class support for:

- Off-heap memory and direct interaction with native BLAS libraries (OpenBLAS, oneMKL, cuBLAS)
- Hardware acceleration on x86 (AVX2/AVX512), ARM (AArch64), PowerPC (PPC64LE), and NVIDIA GPUs via CUDA
- A pluggable backend architecture that lets you swap from CPU to GPU by changing a single Maven/Gradle dependency
- Per-array data types rather than a single global precision setting

ND4J is not a neural network library. It provides the tensor substrate on which DL4J and SameDiff are built. You can use ND4J independently for any scientific computing task that benefits from fast native math on the JVM.

### Key Design Goals

**Production-grade**: ND4J is designed for deployment, not just experimentation. The off-heap memory model, workspace-based memory reuse, and native backend are all oriented toward throughput and stability in long-running JVM processes.

**API stability**: The `INDArray` Java interface is the stable contract. Backend implementations may change; the interface does not.

**Backend transparency**: The same Java code runs on CPU or GPU. No conditional logic, no separate code paths.

---

## The INDArray Interface

`org.nd4j.linalg.api.ndarray.INDArray` is the central abstraction. It represents an N-dimensional array — a tensor — with a numeric `DataType`, a shape, and data stored off-heap.

You never instantiate `INDArray` directly. All creation goes through static factory methods on `org.nd4j.linalg.factory.Nd4j`:

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.factory.Nd4j;

// 3x4 matrix of zeros, FLOAT type (the default)
INDArray zeros = Nd4j.zeros(3, 4);

// Explicitly specifying DataType
INDArray doubleMat = Nd4j.zeros(DataType.DOUBLE, 3, 4);

// From a Java 2D array — shape is inferred
INDArray a = Nd4j.createFromArray(new float[][]{{1, 2, 3}, {4, 5, 6}});

// Random normal values
INDArray rand = Nd4j.randn(DataType.FLOAT, 100, 100);
```

### Core Properties

Every `INDArray` has four defining characteristics:

**Rank** — the number of dimensions. A scalar has rank 0, a vector rank 1, a matrix rank 2, a batch of images rank 4. There is no upper limit on rank.

**Shape** — a `long[]` giving the size of each dimension. A 3x4x5 array has shape `[3, 4, 5]`. Shape determines which indices are valid.

**Length** — the total number of elements, equal to the product of the shape dimensions. Shape `[3, 4, 5]` gives length 60.

**Stride** — a `long[]` giving the distance in the underlying flat buffer between adjacent elements along each dimension. Stride is the mechanism that makes views, transposes, and non-contiguous slices possible without copying data.

```java
INDArray arr = Nd4j.rand(DataType.FLOAT, 3, 4, 5);

long[] shape  = arr.shape();     // [3, 4, 5]
int    rank   = arr.rank();       // 3
long   length = arr.length();     // 60
long[] stride = arr.stride();     // e.g. [20, 5, 1] for C-order
```

### Shape Introspection

```java
// Number of dimensions
int rank = arr.rank();

// Total element count
long len = arr.length();

// Size of a specific dimension
long dim0 = arr.size(0);
long dim1 = arr.size(1);

// The full shape array
long[] shape = arr.shape();

// 2D convenience methods — only meaningful for matrices
long rows = arr.rows();     // same as arr.size(0)
long cols = arr.columns();  // same as arr.size(1)

// Type checks
boolean isMatrix = arr.isMatrix();    // rank == 2
boolean isVector = arr.isVector();    // rank == 1, or one dim is 1
boolean isScalar = arr.isScalar();    // rank == 0 or length == 1

// Runtime type
DataType type = arr.dataType();       // e.g. DataType.FLOAT
```

### The `Nd4j` Factory Class

`Nd4j` is the entry point for all array creation and for many utility operations. Its most frequently used methods:

```java
// Constant-filled arrays
INDArray zeros   = Nd4j.zeros(3, 4);             // all zeros, default DataType (FLOAT)
INDArray ones    = Nd4j.ones(DataType.FLOAT, 3, 4);

// Random arrays
INDArray uniform = Nd4j.rand(DataType.FLOAT, 3, 4);   // uniform [0, 1)
INDArray normal  = Nd4j.randn(DataType.FLOAT, 3, 4);  // N(0, 1)

// Seeded random (reproducible)
Nd4j.getRandom().setSeed(12345L);
INDArray seeded  = Nd4j.rand(DataType.FLOAT, 10, 10);

// Sequences
INDArray lin     = Nd4j.linspace(DataType.FLOAT, 0, 10, 100); // 100 pts from 0..10
INDArray eye     = Nd4j.eye(5);                // 5x5 identity matrix

// Stack and concatenate
INDArray hstacked = Nd4j.hstack(arr1, arr2);   // horizontal stack (same row count)
INDArray vstacked = Nd4j.vstack(arr1, arr2);   // vertical stack (same column count)
INDArray catd0    = Nd4j.concat(0, arr1, arr2); // concat along dimension 0
INDArray catd1    = Nd4j.concat(1, arr1, arr2); // concat along dimension 1

// Diagonal
INDArray diag = Nd4j.diag(vector);  // vector -> NxN matrix with vector on diagonal
                                     // NxN matrix -> vector of diagonal values
```

---

## Memory Layout: Off-Heap via JavaCPP

### Off-Heap Storage

`INDArray` data is not stored on the JVM heap. It lives in **native memory** managed by JavaCPP, outside the reach of the garbage collector. The JVM-side `INDArray` object holds only a small Java pointer; the actual tensor data is in a `DataBuffer` backed by a direct `ByteBuffer` or a CUDA memory region.

This design has three significant consequences:

**Interoperability with native libraries.** OpenBLAS, oneMKL, and cuBLAS all accept raw memory pointers. Off-heap storage means data can be passed to these libraries with zero copy. Matrix multiplications and convolutions call directly into optimised BLAS routines.

**No 2^31 element limit.** Java arrays (`float[]`, `double[]`) are indexed by `int`, capping their size at about 2.1 billion elements. Off-heap buffers use `long` indexing and have no such limit. A single `INDArray` can hold tens of billions of elements.

**Reduced GC pressure.** Large tensors do not participate in garbage collection cycles. This matters for training jobs where GC pauses can be a significant source of latency.

The tradeoff: you must configure both JVM heap memory and off-heap memory explicitly. See the Memory and Workspaces page for JVM launch flags.

### C Order and F Order

ND4J supports two physical memory layouts:

**C order (row-major)** — the default. For a 2D matrix, elements within the same row are contiguous in memory. For a shape `[rows, cols]` matrix with C order, the strides are `[cols, 1]`. Moving from element `[i, j]` to `[i, j+1]` costs 1 position in the buffer; moving from `[i, j]` to `[i+1, j]` costs `cols` positions. This matches NumPy's default and the layout of C arrays.

**F order (column-major)** — Fortran order. For a 2D matrix, elements within the same column are contiguous. For a shape `[rows, cols]` matrix with F order, the strides are `[1, rows]`. Some BLAS routines return F-order results internally.

Concretely, for a 3x3 matrix:

```
Values:   [1, 2, 3]
          [4, 5, 6]
          [7, 8, 9]

C-order buffer: [1, 2, 3, 4, 5, 6, 7, 8, 9]   strides: [3, 1]
F-order buffer: [1, 4, 7, 2, 5, 8, 3, 6, 9]   strides: [1, 3]
```

You can inspect and control ordering:

```java
char order = arr.ordering();    // 'c' or 'f'

// Get a C-order copy of an F-order array
INDArray cOrder = arr.dup('c');

// Get an F-order copy
INDArray fOrder = arr.dup('f');
```

For most users, the default C order is the right choice. Mixed-order arrays work correctly in all ND4J operations — order is an implementation detail about how data is laid out in memory, not a semantic constraint on what operations are valid.

### Strides in Depth

Strides explain exactly how multi-dimensional index tuples map to positions in a flat buffer. For an array with shape `[d0, d1, ..., dN]` and strides `[s0, s1, ..., sN]`, the buffer offset for element at index `[i0, i1, ..., iN]` is:

```
offset = i0*s0 + i1*s1 + ... + iN*sN
```

For a contiguous C-order array of shape `[3, 4, 5]`, the strides are `[20, 5, 1]`. To reach element `[1, 2, 3]`, the offset is `1*20 + 2*5 + 3*1 = 33`. This is precisely what makes views efficient: a transposed array or a row slice just changes the strides and/or offset into the same underlying buffer, with no data copy.

---

## Views vs. Copies

Understanding the difference between views and copies is essential for writing correct, efficient ND4J code.

### What Is a View?

A **view** is an `INDArray` that shares the same underlying `DataBuffer` as another array. The view may have a different shape, different strides, or a different starting offset into the same buffer, but any modification to the view's data is visible in the original array, and vice versa.

Many common operations return views rather than copies:

- `getRow(int)` — row slice
- `getColumn(int)` — column slice
- `transpose()` — dimension reordering
- `reshape(long...)` — shape change (when possible)
- `get(NDArrayIndex...)` — sub-array access

```java
INDArray matrix = Nd4j.createFromArray(new float[][]{{1,2,3},{4,5,6},{7,8,9}});

// getRow(0) returns a VIEW of the first row
INDArray row0 = matrix.getRow(0);
row0.addi(10);                      // modify in-place

// row0 is now [11, 12, 13]
// matrix row 0 is ALSO [11, 12, 13] — same underlying data
System.out.println(matrix);
// [[11.0000, 12.0000, 13.0000],
//  [4.0000, 5.0000, 6.0000],
//  [7.0000, 8.0000, 9.0000]]
```

### Transpose Is a View

`transpose()` returns a view with swapped strides — no data is moved:

```java
INDArray mat    = Nd4j.rand(DataType.FLOAT, 3, 4);
INDArray tMat   = mat.transpose();   // shape [4, 3], same buffer as mat
// tMat.strides()[0] == 1, tMat.strides()[1] == 4  (inverted from C order)

// Transposing a large matrix is O(1) — only metadata changes
INDArray bigMat = Nd4j.rand(DataType.FLOAT, 10000, 10000);
INDArray bigT   = bigMat.transpose();   // returns immediately, no data copy
```

If you need an independent transposed copy: `bigMat.transpose().dup()`.

### Reshape: Views When Possible

`reshape` returns a view when the array is contiguous in memory, and a copy otherwise:

```java
INDArray a = Nd4j.rand(DataType.FLOAT, 3, 4);   // shape [3, 4], length 12
INDArray b = a.reshape(2, 6);                    // view — same 12 elements, shape [2, 6]
INDArray c = a.reshape(12);                      // view — shape [12]
INDArray d = a.reshape(1, 12);                   // view — shape [1, 12]
```

After a `transpose()`, the array is no longer contiguous, so `reshape` on a transposed array will produce a copy:

```java
INDArray t = a.transpose();        // view, non-contiguous
INDArray r = t.reshape(2, 6);      // COPY — t is non-contiguous
```

### Making Explicit Copies

Use `dup()` when you need an independent array with the same values:

```java
// Independent copy — changes to rowCopy do not affect matrix
INDArray rowCopy = matrix.getRow(1).dup();
rowCopy.addi(100);
// matrix row 1 is unchanged
```

You can also request a specific memory order in the copy:

```java
INDArray copy_c = arr.dup('c');    // C-order copy
INDArray copy_f = arr.dup('f');    // F-order copy
```

### In-Place vs. Out-of-Place Operations

ND4J follows a naming convention that matters especially when working with views:

- Methods ending in **`i`** (`addi`, `muli`, `subi`, `divi`) are **in-place**: they modify the receiver and return it. The returned object is the same Java instance.
- Methods without the `i` suffix (`add`, `mul`, `sub`, `div`) are **out-of-place**: they allocate a new array, leave the receiver unchanged, and return the new array.

```java
INDArray x = Nd4j.create(new float[]{1, 2, 3});
INDArray y = Nd4j.create(new float[]{4, 5, 6});

// Out-of-place: x is not modified, z is a new array
INDArray z = x.add(y);
// z: [5, 7, 9]   x: [1, 2, 3]

// In-place: x is modified, w is the same object as x
INDArray w = x.addi(y);
// x: [5, 7, 9]   (w == x in terms of Java identity)
```

**Be careful with in-place operations on views.** Calling `addi` on a view modifies the original array's data. This is often what you want (for example, `matrix.getRow(0).addi(1.0)` to increment the first row in place), but it can also introduce subtle bugs if you forget a variable is a view.

---

## Data Types

Every `INDArray` has a data type represented by the `org.nd4j.linalg.api.buffer.DataType` enum. In M2.1, data types are **per-array** — different arrays in the same JVM can have different types simultaneously.

### Available Types

**Floating point:**

| Type | Bits | Notes |
|------|------|-------|
| `DOUBLE` | 64 | IEEE 754 double precision |
| `FLOAT` | 32 | IEEE 754 single precision — **default** |
| `FLOAT16` | 16 | Half precision (alias: `HALF`) |
| `BFLOAT16` | 16 | Brain float — wider exponent range than FLOAT16 |

**Signed integer:**

| Type | Bits | Alias (deprecated) |
|------|------|---------------------|
| `INT64` | 64 | `LONG` |
| `INT32` | 32 | `INT` |
| `INT16` | 16 | `SHORT` |
| `INT8` | 8 | `BYTE` |

**Unsigned integer:**

| Type | Bits | Alias (deprecated) |
|------|------|---------------------|
| `UINT64` | 64 | — |
| `UINT32` | 32 | — |
| `UINT16` | 16 | — |
| `UINT8` | 8 | `UBYTE` |

**Other:** `BOOL`, `UTF8`

### Migration from Earlier Releases

Prior to M2.1, ND4J used `DataBuffer.Type` and a single global type setting:

```java
// OLD (beta4 and earlier) — does not compile in M2.1
Nd4j.setDataType(DataBuffer.Type.DOUBLE);
DataBuffer.Type.FLOAT

// NEW (M2.1)
import org.nd4j.linalg.api.buffer.DataType;
DataType.DOUBLE
DataType.FLOAT
```

Replace all occurrences of `DataBuffer.Type` with `DataType` when migrating.

### Default Data Type and Global Configuration

The default type for newly created arrays is `FLOAT`. To change the default at application startup:

```java
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.factory.Nd4j;

// Set the default floating-point type and the default integer type
Nd4j.setDefaultDataTypes(DataType.DOUBLE, DataType.INT64);
```

Call this once before any array creation. All subsequent `Nd4j.zeros(...)`, `Nd4j.rand(...)`, etc. calls that do not specify a type will use the new default.

### Creating Typed Arrays

```java
// Explicit type in creation
INDArray fp64  = Nd4j.zeros(DataType.DOUBLE, 3, 4);
INDArray fp16  = Nd4j.rand(DataType.FLOAT16, 128, 128);
INDArray int32 = Nd4j.zeros(DataType.INT32, 10);

// Cast an existing array to a different type (returns a copy)
INDArray asDouble = fp16.castTo(DataType.DOUBLE);

// Check the type
DataType dt = arr.dataType();
boolean isFloat = (dt == DataType.FLOAT);
```

For full coverage of type semantics, casting rules, and best practices for mixed-precision workflows, see the [Data Types page](data-types.md).

---

## Creating NDArrays: Reference

### Zeros, Ones, and Scalar Fill

```java
// Zero-filled
INDArray z3x4 = Nd4j.zeros(3, 4);                        // shape [3,4], FLOAT
INDArray z3x4d = Nd4j.zeros(DataType.DOUBLE, 3, 4);      // shape [3,4], DOUBLE

// One-filled
INDArray o2x5 = Nd4j.ones(2, 5);

// Arbitrary scalar fill: create zeros then add in-place
INDArray fives = Nd4j.zeros(3, 4).addi(5.0);             // all 5s

// Scalar INDArray (rank 0)
INDArray scalar = Nd4j.scalar(DataType.FLOAT, 3.14f);
```

### From Java Arrays

```java
// 1D row vector
INDArray row = Nd4j.create(new float[]{1, 2, 3, 4});    // shape [4]

// Column vector
INDArray col = Nd4j.create(new float[]{1, 2, 3}, new int[]{3, 1}); // shape [3,1]

// 2D from a nested Java array — shape inferred
INDArray mat = Nd4j.createFromArray(new float[][]{{1,2,3},{4,5,6}});

// 3D and higher: flatten then provide shape
float[] flat = new float[]{1,2,3,4,5,6,7,8};
INDArray tensor = Nd4j.create(flat, new int[]{2, 2, 2}, 'c');
```

### Random Arrays

```java
// Uniform [0, 1)
INDArray u = Nd4j.rand(DataType.FLOAT, 3, 4);

// Standard normal N(0, 1)
INDArray n = Nd4j.randn(DataType.FLOAT, 3, 4);

// Seeded for reproducibility
Nd4j.getRandom().setSeed(42L);
INDArray seeded = Nd4j.rand(DataType.FLOAT, 3, 4);
```

### Sequences and Structured Arrays

```java
// Linspace: 50 values from 0 to 1 inclusive
INDArray lin = Nd4j.linspace(DataType.FLOAT, 0, 1, 50);

// Reshaped linspace: 5x5 matrix with values 1..25
INDArray grid = Nd4j.linspace(DataType.FLOAT, 1, 25, 25).reshape(5, 5);

// Identity matrix
INDArray eye = Nd4j.eye(5);                    // 5x5, DOUBLE
```

### From Other NDArrays

```java
// Deep copy
INDArray copy = original.dup();

// Cast to new type (copy)
INDArray fp64 = original.castTo(DataType.DOUBLE);

// Stack multiple arrays
INDArray h = Nd4j.hstack(a, b);  // horizontal — same row count
INDArray v = Nd4j.vstack(a, b);  // vertical — same column count

// Concatenate along a dimension
INDArray cat = Nd4j.concat(0, a, b, c);   // cat along rows
```

---

## Getting and Setting Values

### Individual Elements

```java
// Read a single value as double (works for any DataType)
double v = arr.getDouble(i, j);

// Read as float
float f = arr.getFloat(i, j);

// For higher-rank arrays, provide all indices
double val = arr.getDouble(new long[]{i, j, k});

// Set a single value
arr.putScalar(new int[]{i, j}, 3.14);
arr.putScalar(new int[]{i, j, k, l}, 0.5);
```

Iterating element by element is slow. Prefer vectorised operations whenever possible.

### Rows and Columns

```java
// Get a row — returns a VIEW
INDArray row = arr.getRow(0);

// Get a column — returns a VIEW
INDArray col = arr.getColumn(1);

// Set a row (arr must be 2D; row must have the right column count)
arr.putRow(0, someRowVector);

// Set a column
arr.putColumn(1, someColVector);

// Get multiple rows as a new matrix (returns a COPY, not a view)
INDArray multiRow = arr.getRows(0, 2, 4);   // rows 0, 2, 4
```

### Sub-Arrays with NDArrayIndex

`NDArrayIndex` provides flexible sub-array access for arbitrary dimensionality:

```java
import org.nd4j.linalg.indexing.NDArrayIndex;

// Single row, all columns — VIEW
INDArray row2 = arr.get(NDArrayIndex.point(2), NDArrayIndex.all());

// Rows 1..3 (exclusive), all columns — VIEW
INDArray rows1to3 = arr.get(NDArrayIndex.interval(1, 3), NDArrayIndex.all());

// All rows, every other column — VIEW
INDArray altCols = arr.get(NDArrayIndex.all(), NDArrayIndex.interval(0, 2, arr.columns()));

// Set values in a sub-array
arr.put(new INDArrayIndex[]{NDArrayIndex.interval(0, 2), NDArrayIndex.all()},
        Nd4j.zeros(2, arr.columns()));
```

`NDArrayIndex.interval`, `NDArrayIndex.point`, and `NDArrayIndex.all` return views. Use `.dup()` on the result if you need a copy.

---

## Key Operations

### Scalar Operations

Add, subtract, multiply, divide every element by a constant:

```java
INDArray r1 = arr.add(5.0);     // copy — arr unchanged
INDArray r2 = arr.sub(2.0);     // copy
arr.muli(0.1);                  // in-place — arr is modified
arr.divi(3.0);                  // in-place
```

### Element-Wise Operations

```java
INDArray c = a.add(b);    // element-wise add, copy
a.addi(b);                // element-wise add, in-place

INDArray p = a.mul(b);    // element-wise multiply, copy
INDArray q = a.div(b);    // element-wise divide, copy
```

### Reductions

Reductions can run over the entire array or along specific dimensions:

```java
// Global reductions — return a Java number
double total     = arr.sumNumber().doubleValue();
double product   = arr.prodNumber().doubleValue();
double mean      = arr.meanNumber().doubleValue();
double stdDev    = arr.stdNumber(true).doubleValue();   // true = population std
double maxVal    = arr.maxNumber().doubleValue();

// Dimensional reductions — return an INDArray
INDArray colSums  = arr.sum(0);      // sum across rows -> shape [1, cols]
INDArray rowMeans = arr.mean(1);     // mean across cols -> shape [rows, 1]
INDArray colMax   = arr.max(0);      // max in each column
INDArray rowArgmax = arr.argMax(1);  // index of max in each row
```

### Linear Algebra

```java
// Matrix multiplication (a is [m x k], b is [k x n], result is [m x n])
INDArray result = a.mmul(b);

// In-place matrix multiplication into a pre-allocated output
INDArray out = Nd4j.zeros(m, n);
a.mmuli(b, out);

// Transpose
INDArray t = mat.transpose();       // view
INDArray tCopy = mat.transpose().dup();   // copy

// Norm
double l1 = arr.norm1Number().doubleValue();
double l2 = arr.norm2Number().doubleValue();

// Matrix inverse (from nd4j-api)
import org.nd4j.linalg.inverse.InvertMatrix;
INDArray inv = InvertMatrix.invert(squareMat, false);  // false = return copy
```

### Element-Wise Transforms

```java
import org.nd4j.linalg.ops.transforms.Transforms;

INDArray sigmoid = Transforms.sigmoid(arr);          // copy
INDArray tanh    = Transforms.tanh(arr);             // copy
INDArray relu    = Transforms.relu(arr);             // copy
INDArray log     = Transforms.log(arr);              // copy
INDArray exp     = Transforms.exp(arr);              // copy
INDArray abs     = Transforms.abs(arr);              // copy

// In-place versions
Transforms.sigmoid(arr, false);   // false = in-place
```

### Reshape, Flatten, and Permute

```java
// Reshape — total elements must match
INDArray r = arr.reshape(2, 6);    // [3,4] -> [2,6] (view if contiguous)

// Flatten to a row vector
INDArray flat = arr.reshape(1, arr.length());

// Permute dimensions (like NumPy's transpose for arbitrary dimension orders)
// For a [batch, height, width, channels] array, convert to [batch, channels, height, width]
INDArray permuted = arr.permute(0, 3, 1, 2);   // reorders axes

// Flatten to 1D using Nd4j.toFlattened with explicit traversal order
INDArray flatC = Nd4j.toFlattened('c', arr1, arr2);  // C-order traversal
INDArray flatF = Nd4j.toFlattened('f', arr1, arr2);  // F-order traversal
```

---

## Architecture: The Backend System

### Overview

ND4J uses a **Service Provider Interface (SPI)** to decouple the Java API from the native implementation. The `INDArray` interface and `Nd4j` factory class are defined in the `nd4j-api` module and carry no native code. The actual computation is provided by a backend JAR that is discovered at runtime via `java.util.ServiceLoader`.

Two production backends ship with M2.1:

| Backend | Maven artifact | Target hardware |
|---------|---------------|-----------------|
| `nd4j-native` | `org.nd4j:nd4j-native` | CPU (x86, ARM, PPC) |
| `nd4j-cuda` | `org.nd4j:nd4j-cuda-12.x` | NVIDIA GPU via CUDA |

Exactly one backend should be on the classpath at runtime.

### nd4j-native (CPU Backend)

`nd4j-native` links against libnd4j, the C++ kernel library, via JavaCPP. It supports:

- x86\_64 with AVX2 acceleration (default)
- x86\_64 with AVX512 acceleration (via the `avx512` classifier)
- AArch64 (ARM 64-bit)
- PPC64LE (IBM Power)

The native platform binaries are bundled in classifier JARs. If you let Maven/Gradle resolve the platform automatically, the right native binary is pulled for your OS and CPU.

```xml
<!-- Maven: CPU backend with automatic platform detection -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${dl4j.version}</version>
</dependency>
<!-- JavaCPP platform dependency — resolves native binary for current OS/arch -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${dl4j.version}</version>
    <classifier>${javacpp.platform}</classifier>
</dependency>
```

### nd4j-cuda (GPU Backend)

`nd4j-cuda` links against libnd4j compiled for CUDA and uses cuBLAS and cuDNN for accelerated operations. Requirements:

- NVIDIA GPU with CUDA Compute Capability 3.5 or higher
- CUDA toolkit installed and matching the artifact version (`12.x` for M2.1)

```xml
<!-- Maven: CUDA 12.x GPU backend -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-12.x</artifactId>
    <version>${dl4j.version}</version>
</dependency>
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-12.x</artifactId>
    <version>${dl4j.version}</version>
    <classifier>linux-x86_64-cudnn</classifier>
</dependency>
```

From a Java code perspective, switching from CPU to GPU is a dependency swap only — no source changes required. All `Nd4j.*` calls, all `INDArray` operations, and all DL4J/SameDiff code work identically on both backends.

### SPI Mechanism

At startup, `Nd4j` calls `ServiceLoader.load(NDArrayFactory.class)` to discover the backend. The factory loaded from the classpath determines the concrete `INDArray` implementation and the native dispatch layer. If no factory is found, `Nd4j` throws a `RuntimeException` immediately.

You can query the active backend at runtime:

```java
import org.nd4j.linalg.factory.Nd4j;

// Get the backend class name
String backendClass = Nd4j.getBackend().getClass().getName();
// e.g. "org.nd4j.linalg.cpu.nativecpu.CpuBackend"
//   or "org.nd4j.linalg.jcublas.JCublasBackend"
```

### libnd4j (C++ Layer)

Below the Java backend lies libnd4j — the C++ kernel library. It provides all the compute kernels: element-wise ops, BLAS calls, convolutions, reductions, and RNG. libnd4j is compiled separately for each target platform and bundled inside the classifier JARs. It is not a user-facing API; you do not need to interact with it directly.

Its existence matters for two scenarios:

1. **Native crash diagnosis.** If ND4J throws a `java.lang.UnsatisfiedLinkError` or the JVM crashes with a native stack trace, libnd4j is involved. Check that the classifier JAR for your OS/CPU/CUDA version is on the classpath.
2. **Building from source.** If you need to add a custom kernel or support a new hardware target, libnd4j is where you write the C++ code.

---

## Workspaces and Memory Management

Workspaces are ND4J's mechanism for reusing native memory allocations across iterations of a processing loop. Rather than allocating and deallocating off-heap memory on each training step, a workspace pre-allocates a memory block and recycles it.

For a complete treatment see the Memory and Workspaces page. The summary:

```java
import org.nd4j.linalg.api.memory.MemoryWorkspace;

// Workspace recycles memory across iterations
for (int i = 0; i < iterations; i++) {
    try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
            .getAndActivateWorkspace("TRAINING_WS")) {

        INDArray batch = loadBatch(i);       // allocated inside workspace
        INDArray output = model.output(batch);
        // ...
        // workspace closes here; batch and output memory is recycled
    }
}
```

**Important**: arrays allocated inside a workspace are invalid after the workspace closes. Use `INDArray.detach()` to move an array out of a workspace into regular heap-managed off-heap memory when you need to retain it:

```java
INDArray retained;
try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace("WS")) {
    retained = Nd4j.rand(DataType.FLOAT, 100).detach();
    // detach() copies the data out of the workspace
}
// retained is now safe to use outside the try block
```

---

## Serialization

ND4J supports saving and loading `INDArray`s in binary, text, and NumPy-compatible formats.

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.serde.binary.BinarySerde;

import java.io.*;
import java.nio.ByteBuffer;

INDArray arr = Nd4j.linspace(DataType.FLOAT, 1, 10, 10);

// --- Binary format (compact, recommended for production) ---
try (DataOutputStream dos = new DataOutputStream(new FileOutputStream("array.bin"))) {
    Nd4j.write(arr, dos);
}
try (DataInputStream dis = new DataInputStream(new FileInputStream("array.bin"))) {
    INDArray loaded = Nd4j.read(dis);
}

// --- ByteBuffer (useful for embedding in messages or custom I/O) ---
ByteBuffer buf    = BinarySerde.toByteBuffer(arr);
INDArray fromBuf  = BinarySerde.toArray(buf);

// --- Text format (human-readable, slower) ---
Nd4j.writeTxt(arr, "array.txt");
INDArray fromTxt = Nd4j.readTxt("array.txt");

// --- NumPy-compatible CSV ---
INDArray fromCsv = Nd4j.readNumpy("array.csv", ",");
```

The `nd4j-serde` module also provides Jackson, Kryo, and Aeron serializers for integration with common Java serialization frameworks.

---

## Relationship to SameDiff

SameDiff is ND4J's automatic differentiation framework. It lives in the `nd4j-api` module alongside `INDArray` and shares the same backend infrastructure. The key distinction:

| | INDArray (eager) | SameDiff (graph) |
|-|-------------------|-----------------|
| **Execution** | Immediate — operations run when called | Deferred — operations build a graph, execution triggered separately |
| **Gradients** | Not automatic — you implement backprop manually | Automatic — call `sd.execBackwards()` |
| **Use case** | Data preprocessing, feature engineering, one-off computations | Neural network training, optimisation loops |
| **Primary type** | `INDArray` | `SDVariable` (wraps `INDArray` at execution time) |

SameDiff operates on the same backends and produces `INDArray` results when executed:

```java
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.autodiff.samediff.SDVariable;

SameDiff sd = SameDiff.create();

// Define graph symbolically
SDVariable x     = sd.var("x", DataType.FLOAT, 3, 4);
SDVariable w     = sd.var("w", DataType.FLOAT, 4, 2);
SDVariable b     = sd.var("b", DataType.FLOAT, 1, 2);
SDVariable logit = x.mmul(w).add(b);
SDVariable out   = sd.nn().sigmoid(logit);

// Associate concrete data and execute
sd.associateArrayWithVariable(Nd4j.rand(DataType.FLOAT, 3, 4), x);
sd.associateArrayWithVariable(Nd4j.rand(DataType.FLOAT, 4, 2), w);
sd.associateArrayWithVariable(Nd4j.zeros(DataType.FLOAT, 1, 2), b);

INDArray result = out.eval();   // triggers graph execution, returns INDArray
```

DL4J's `MultiLayerNetwork` and `ComputationGraph` are built on SameDiff internally as of M2.1. Custom layers and loss functions can be written in either the INDArray eager API or the SameDiff graph API.

For a full treatment of SameDiff, including defining custom operations, exporting to ONNX/TensorFlow SavedModel, and mixed-precision training, see the SameDiff section.

---

## Capability Map

The following table maps common tasks to the relevant ND4J API and links to more detailed pages in this section.

| Task | API entry point | Detail page |
|------|----------------|-------------|
| Array creation and destruction | `Nd4j.*` factory methods | [Tensors and NDArrays](../core-concepts/tensors-and-ndarrays.md) |
| Data types and casting | `DataType` enum, `arr.castTo()` | [Data Types](data-types.md) |
| Element access and slicing | `INDArrayIndex`, `getRow`, `get` | [Tensors and NDArrays](../core-concepts/tensors-and-ndarrays.md) |
| Math operations | `INDArray.*`, `Transforms.*`, `Nd4j.getExecutioner()` | Operations (forthcoming) |
| Linear algebra | `mmul`, `transpose`, `InvertMatrix` | Operations (forthcoming) |
| Off-heap memory | JavaCPP, `DataBuffer` | [Memory and Workspaces](../core-concepts/memory-and-workspaces.md) |
| Workspace-based memory reuse | `Nd4j.getWorkspaceManager()` | [Memory and Workspaces](../core-concepts/memory-and-workspaces.md) |
| Backend selection (CPU/GPU) | Maven/Gradle dependency | [Backends](backends/index.md) |
| Automatic differentiation | `SameDiff`, `SDVariable` | [SameDiff](samediff/index.md) |

---

## Quick Reference

### Creating Arrays

```java
Nd4j.zeros(3, 4)                             // [3,4] zeros, FLOAT
Nd4j.zeros(DataType.DOUBLE, 3, 4)            // [3,4] zeros, DOUBLE
Nd4j.ones(3, 4)                              // [3,4] ones, FLOAT
Nd4j.rand(DataType.FLOAT, 3, 4)              // uniform random
Nd4j.randn(DataType.FLOAT, 3, 4)             // normal random
Nd4j.linspace(DataType.FLOAT, 0, 1, 100)     // 100 points 0..1
Nd4j.eye(5)                                  // 5x5 identity
Nd4j.createFromArray(new float[][]{{1,2},{3,4}}) // from Java array
arr.dup()                                    // deep copy
arr.castTo(DataType.DOUBLE)                  // type cast (copy)
```

### Shape and Type

```java
arr.shape()         // long[]
arr.rank()          // int
arr.length()        // long
arr.size(dim)       // long
arr.stride()        // long[]
arr.ordering()      // 'c' or 'f'
arr.dataType()      // DataType enum value
```

### Indexing

```java
arr.getRow(i)                                   // view
arr.getColumn(j)                                // view
arr.getRows(0, 2, 4)                            // copy of rows 0, 2, 4
arr.get(NDArrayIndex.point(i), NDArrayIndex.all())          // view
arr.get(NDArrayIndex.interval(a, b), NDArrayIndex.all())    // view
arr.getDouble(i, j)
arr.putScalar(new int[]{i, j}, value)
```

### Operations

```java
// Scalar
arr.add(5.0)  arr.addi(5.0)     // copy / in-place
arr.sub(2.0)  arr.subi(2.0)
arr.mul(3.0)  arr.muli(3.0)
arr.div(4.0)  arr.divi(4.0)

// Element-wise
a.add(b)  a.addi(b)
a.mul(b)  a.muli(b)

// Reductions
arr.sumNumber()       arr.sum(dim)
arr.meanNumber()      arr.mean(dim)
arr.maxNumber()       arr.max(dim)
arr.norm1Number()     arr.norm2Number()
arr.argMax(dim)

// Linear algebra
a.mmul(b)
a.mmuli(b, out)
mat.transpose()         // view
InvertMatrix.invert(mat, false)

// Transforms
Transforms.sigmoid(arr)
Transforms.tanh(arr)
Transforms.relu(arr)
Transforms.log(arr)
Transforms.exp(arr)

// Shape
arr.reshape(newShape)
arr.transpose()
arr.permute(0, 2, 1)
Nd4j.toFlattened('c', arr1, arr2)
```

### Serialization

```java
Nd4j.write(arr, dataOutputStream)
Nd4j.read(dataInputStream)
BinarySerde.toByteBuffer(arr)
BinarySerde.toArray(byteBuffer)
Nd4j.writeTxt(arr, filename)
Nd4j.readTxt(filename)
```

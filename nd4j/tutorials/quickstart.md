---
title: "ND4J Quickstart"
description: "Hands-on quickstart guide for ND4J — creating arrays, basic operations, indexing, and shape manipulation"
---

ND4J is a scientific computing library for the JVM designed for production use. It provides a versatile N-dimensional array object (`INDArray`), linear algebra and signal processing routines, and multi-platform execution including CPU and GPU. This guide follows the structure of the [NumPy quickstart](https://numpy.org/doc/stable/user/quickstart.html) so that Python users can map familiar concepts directly to ND4J equivalents.


## Maven Setup

Add the following to your `pom.xml`. You need the core platform bundle and a backend. Use `nd4j-native-platform` for CPU or `nd4j-cuda-11.x-platform` for GPU.

```xml
<properties>
  <dl4j.version>1.0.0-M2.1</dl4j.version>
</properties>

<dependencies>
  <!-- ND4J API -->
  <dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-api</artifactId>
    <version>${dl4j.version}</version>
  </dependency>

  <!-- CPU backend — replaces this with nd4j-cuda-11.x-platform for GPU -->
  <dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native-platform</artifactId>
    <version>${dl4j.version}</version>
  </dependency>
</dependencies>
```

If you are using the broader DL4J stack (DataVec, DL4J training), add the BOM instead to keep versions consistent:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.deeplearning4j</groupId>
      <artifactId>deeplearning4j-parent</artifactId>
      <version>${dl4j.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```


## Imports Used in This Guide

All examples below assume these imports are present:

```java
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.indexing.NDArrayIndex;
import org.nd4j.linalg.ops.transforms.Transforms;
```


## Array Creation

ND4J arrays are created through static factory methods on `Nd4j`. You never call `new INDArray(...)` directly.

### zeros, ones, rand

```java
// 1D array of five zeros (default type: FLOAT)
INDArray z = Nd4j.zeros(5);
// [0.0000, 0.0000, 0.0000, 0.0000, 0.0000]

// 1D array of five ones, explicitly typed
INDArray o = Nd4j.ones(DataType.FLOAT, 5);
// [1.0000, 1.0000, 1.0000, 1.0000, 1.0000]

// 3x4 matrix of uniform random values in [0, 1)
INDArray r = Nd4j.rand(DataType.FLOAT, 3, 4);

// 3x4 matrix of standard-normal random values N(0, 1)
INDArray n = Nd4j.randn(DataType.FLOAT, 3, 4);

// Explicit shape array — same as Nd4j.rand(DataType.FLOAT, 4, 5)
long[] shape = {4, 5};
INDArray r2 = Nd4j.rand(DataType.FLOAT, shape);
```

### createFromArray

`Nd4j.createFromArray` is overloaded for `float`, `double`, `int`, and `long` up to 4D. Shape is inferred from the Java array structure.

```java
// 1D from a float array
float[] data1d = {1.0f, 2.0f, 3.0f, 4.0f};
INDArray v = Nd4j.createFromArray(data1d);
// [1.0000, 2.0000, 3.0000, 4.0000]

// 2D from a 2D float array
float[][] data2d = {{1.0f, 2.0f, 3.0f},
                    {4.0f, 5.0f, 6.0f}};
INDArray m = Nd4j.createFromArray(data2d);
// [[1.0000, 2.0000, 3.0000],
//  [4.0000, 5.0000, 6.0000]]

// Double precision — use double arrays
double[][] data2d_d = {{1.0, 2.0}, {3.0, 4.0}};
INDArray md = Nd4j.createFromArray(data2d_d);
```

### arange and linspace

```java
// Integer-stepped range [0, 5)
INDArray range = Nd4j.arange(5);
// [0.0000, 1.0000, 2.0000, 3.0000, 4.0000]

// Range starting at 2, stopping before 7
INDArray range2 = Nd4j.arange(2, 7);
// [2.0000, 3.0000, 4.0000, 5.0000, 6.0000]

// 5 evenly-spaced points from 0.0 to 1.0 inclusive
INDArray lin = Nd4j.linspace(DataType.FLOAT, 0.0, 1.0, 5);
// [0.0000, 0.2500, 0.5000, 0.7500, 1.0000]
```


## Array Properties

```java
INDArray x = Nd4j.zeros(DataType.FLOAT, 3, 4);

// Number of dimensions
int rank = x.rank();              // 2

// Size of each dimension
long[] shape = x.shape();         // [3, 4]

// Total number of elements
long length = x.length();         // 12

// Element type
DataType dt = x.dataType();       // FLOAT
```

The default `DataType` for arrays created without an explicit type argument is `FLOAT`. In M2.1 the type is represented by the `DataType` enum — the old `DataBuffer.Type` enum was removed.


## Printing Arrays

`INDArray.toString()` produces NumPy-style output. Call it via `System.out.println` or string concatenation.

```java
// 1D
INDArray a = Nd4j.arange(6);
System.out.println(a);
// [0.0000, 1.0000, 2.0000, 3.0000, 4.0000, 5.0000]

// 2D — reshape from 1D (returns a view)
INDArray b = Nd4j.arange(12).reshape(4, 3);
System.out.println(b);
// [[0.0000, 1.0000, 2.0000],
//  [3.0000, 4.0000, 5.0000],
//  [6.0000, 7.0000, 8.0000],
//  [9.0000, 10.0000, 11.0000]]

// 3D
INDArray c = Nd4j.arange(24).reshape(2, 3, 4);
System.out.println(c);
// [[[0.0000,  1.0000,  2.0000,  3.0000],
//   [4.0000,  5.0000,  6.0000,  7.0000],
//   [8.0000,  9.0000, 10.0000, 11.0000]],
//  [[12.0000, 13.0000, 14.0000, 15.0000],
//   [16.0000, 17.0000, 18.0000, 19.0000],
//   [20.0000, 21.0000, 22.0000, 23.0000]]]
```


## Basic Operations

### Copy operations (non-destructive)

Copy operations allocate a new array and leave the original unchanged.

```java
INDArray a = Nd4j.createFromArray(new float[]{1, 2, 3});
INDArray b = Nd4j.createFromArray(new float[]{10, 20, 30});

INDArray sumArr  = a.add(b);   // [11.0000, 22.0000, 33.0000]
INDArray diffArr = a.sub(b);   // [-9.0000, -18.0000, -27.0000]
INDArray prodArr = a.mul(b);   // [10.0000, 40.0000, 90.0000]
INDArray quotArr = a.div(b);   // [0.1000, 0.1000, 0.1000]

// Scalar variants work the same way
INDArray plus2 = a.add(2.0);   // [3.0000, 4.0000, 5.0000]
INDArray times3 = a.mul(3.0);  // [3.0000, 6.0000, 9.0000]

// a is still [1, 2, 3] — none of these modified it
```

### In-place operations (i suffix)

In-place operations modify the array they are called on. They return the same Java object for method chaining.

```java
INDArray x = Nd4j.createFromArray(new float[]{1, 2, 3});

x.addi(10);    // x is now [11.0000, 12.0000, 13.0000]
x.muli(2);     // x is now [22.0000, 24.0000, 26.0000]
x.subi(4);     // x is now [18.0000, 20.0000, 22.0000]
x.divi(2);     // x is now [9.0000, 10.0000, 11.0000]

// Chain in-place operations in one expression
INDArray y = Nd4j.zeros(3).addi(5).muli(2);   // [10.0000, 10.0000, 10.0000]
```

The complete set of element-wise arithmetic methods:

| Operation | Copy | In-place |
|---|---|---|
| Addition | `add` | `addi` |
| Subtraction | `sub` | `subi` |
| Multiplication | `mul` | `muli` |
| Division | `div` | `divi` |

### Data type requirement

Operands must share the same `DataType`. Mixing types throws `IllegalArgumentException`.

```java
INDArray floatArr  = Nd4j.zeros(DataType.FLOAT, 5);
INDArray doubleArr = Nd4j.zeros(DataType.DOUBLE, 5);

// This throws:
// java.lang.IllegalArgumentException: Op.X and Op.Y must have the same data type,
//   but got FLOAT vs DOUBLE
INDArray bad = floatArr.add(doubleArr);

// Fix: cast one operand before the operation
INDArray result = floatArr.add(doubleArr.castTo(DataType.FLOAT));

// Or cast the other direction
INDArray result2 = floatArr.castTo(DataType.DOUBLE).add(doubleArr);
```

`castTo` returns a new array of the requested type. The original is not modified.


## Reduction Operations

Reductions collapse one or more dimensions down to a scalar or a lower-rank array.

```java
INDArray x = Nd4j.rand(DataType.FLOAT, 2, 3);
// Example values:
// [[0.8621, 0.9224, 0.8407],
//  [0.1504, 0.5489, 0.9584]]

// Global reductions — return a scalar INDArray
INDArray total = x.sum();    // 4.2829
INDArray lo    = x.min();    // 0.1504
INDArray hi    = x.max();    // 0.9584
INDArray avg   = x.mean();   // 0.7138

// Extract as a Java number
double totalVal = x.sumNumber().doubleValue();
double maxVal   = x.maxNumber().doubleValue();
```

Pass a dimension argument to reduce along a specific axis. Dimension 0 collapses rows (result has one entry per column); dimension 1 collapses columns (result has one entry per row).

```java
INDArray m = Nd4j.arange(12).reshape(3, 4);
// [[0,  1,  2,  3],
//  [4,  5,  6,  7],
//  [8,  9, 10, 11]]

INDArray colSums = m.sum(0);   // sum each column → [12, 15, 18, 21]
INDArray rowMins = m.min(1);   // min of each row  → [0, 4, 8]
INDArray colMaxs = m.max(0);   // max of each column → [8, 9, 10, 11]
INDArray rowMeans = m.mean(1); // mean of each row   → [1.5, 5.5, 9.5]
```


## Transform Operations

Transforms apply a mathematical function element-wise and return a new array. They live in `org.nd4j.linalg.ops.transforms.Transforms`.

```java
import static org.nd4j.linalg.ops.transforms.Transforms.sin;
import static org.nd4j.linalg.ops.transforms.Transforms.cos;
import static org.nd4j.linalg.ops.transforms.Transforms.exp;
import static org.nd4j.linalg.ops.transforms.Transforms.sqrt;
import static org.nd4j.linalg.ops.transforms.Transforms.log;
import static org.nd4j.linalg.ops.transforms.Transforms.abs;

INDArray x = Nd4j.linspace(DataType.FLOAT, 0, 2, 4);
// [0.0000, 0.6667, 1.3333, 2.0000]

INDArray sinX  = sin(x);    // [0.0000, 0.6184, 0.9694, 0.9093]
INDArray cosX  = cos(x);    // [1.0000, 0.7859, 0.2350, -0.4161]
INDArray expX  = exp(x);    // [1.0000, 1.9477, 3.7937, 7.3891]
INDArray sqrtX = sqrt(x);   // [0.0000, 0.8165, 1.1547, 1.4142]
INDArray logX  = log(x.add(1));  // add 1 to avoid log(0)

// Evaluate sin over 100 points in [0, pi] — a common NumPy pattern
INDArray xpts  = Nd4j.linspace(DataType.DOUBLE, 0.0, Math.PI, 100);
INDArray ypts  = sin(xpts);
```

A second boolean argument on most transforms controls whether the result is written in-place:

```java
// false (default) → new array; true → modify x directly
INDArray y = sin(x, false);  // new array, x unchanged
sin(x, true);                // x modified in-place
```


## Matrix Multiplication

Element-wise multiplication uses `mul` / `muli` (covered above). For true matrix products use `mmul`, and for dot products use `Transforms.dot`.

```java
// mmul: matrix product
// x is [3 x 4], y is [4 x 3] → result is [3 x 3]
INDArray x = Nd4j.arange(12).reshape(3, 4);
INDArray y = Nd4j.arange(12).reshape(4, 3);

INDArray product = x.mmul(y);
// [[42,  48,  54],
//  [114, 136, 158],
//  [186, 224, 262]]

// dot product of two 1D vectors
INDArray u = Nd4j.arange(4);
INDArray v = Nd4j.arange(4);
INDArray dotResult = Nd4j.dot(u, v);   // 0*0 + 1*1 + 2*2 + 3*3 = 14

// Transpose before multiply — returns a view, not a copy
INDArray xT = x.transpose();   // shape [4, 3]
INDArray xtx = xT.mmul(x);     // [4 x 3] · [3 x 4] → [4 x 4]
```


## Indexing and Slicing

### Single-element access

```java
INDArray x = Nd4j.createFromArray(new float[]{10, 20, 30, 40, 50});

float  val = x.getFloat(2);     // 30.0  — zero-based
double d   = x.getDouble(0);    // 10.0
```

For 2D arrays, pass row and column:

```java
INDArray m = Nd4j.createFromArray(new float[][]{{1, 2, 3}, {4, 5, 6}});

float v = m.getFloat(1, 2);    // 6.0  (row 1, col 2)
```

### Exporting to Java arrays

```java
INDArray x = Nd4j.arange(5);

float[]  fv = x.toFloatVector();    // [0.0, 1.0, 2.0, 3.0, 4.0]
double[] dv = x.toDoubleVector();   // same values as double[]

// For 2D arrays
INDArray m = Nd4j.rand(DataType.FLOAT, 2, 3);
float[][] fm  = m.toFloatMatrix();
double[][] dm = m.toDoubleMatrix();
```

### Slicing with NDArrayIndex

Use `NDArrayIndex` to extract contiguous and strided sub-arrays. Slices are **views** — modifying the slice modifies the source array.

```java
INDArray x = Nd4j.arange(10);
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

// Elements at indices 2..5 (exclusive of 6)
INDArray slice = x.get(NDArrayIndex.interval(2, 6));
// [2.0000, 3.0000, 4.0000, 5.0000]

// Every 2nd element from index 0 to 8 (exclusive)
INDArray strided = x.get(NDArrayIndex.interval(0, 2, 8));
// [0.0000, 2.0000, 4.0000, 6.0000]
```

For 2D arrays, pass one `NDArrayIndex` per dimension:

```java
INDArray m = Nd4j.rand(DataType.FLOAT, 4, 5);

// Row 1 (all columns)
INDArray row1 = m.get(NDArrayIndex.point(1), NDArrayIndex.all());

// Rows 0..2, columns 1..3
INDArray sub = m.get(NDArrayIndex.interval(0, 3),
                     NDArrayIndex.interval(1, 4));

// Iterate over rows
for (int i = 0; i < m.rows(); i++) {
    INDArray row = m.get(NDArrayIndex.point(i), NDArrayIndex.all());
    // row is a view of row i
}
```

`getRow(i)` and `getColumn(j)` are convenience shortcuts for the common 2D case:

```java
INDArray row0 = m.getRow(0);    // view of row 0
INDArray col2 = m.getColumn(2); // view of column 2
```

### Assigning into a slice

Because slices are views, you can assign into them using `assign`:

```java
INDArray x = Nd4j.zeros(10);

// Set elements 3..6 to 1.0
x.get(NDArrayIndex.interval(3, 7)).assign(1.0);
// [0,0,0,1,1,1,1,0,0,0]
```


## Shape Manipulation

### reshape and ravel

`reshape` returns a **view** — it does not copy data. The total number of elements must remain the same.

```java
INDArray x = Nd4j.rand(DataType.FLOAT, 3, 4);
// shape [3, 4], 12 elements

INDArray flat = x.reshape(12);       // shape [12]  — view
INDArray grid = x.reshape(2, 6);     // shape [2,6] — view
INDArray cube = x.reshape(2, 2, 3);  // shape [2,2,3] — view

// ravel() is equivalent to reshape(-1) — flattens to 1D
INDArray flat2 = x.ravel();          // shape [12]  — view
```

Because `reshape` and `ravel` return views, a mutation in one propagates to all:

```java
INDArray a = Nd4j.rand(DataType.FLOAT, 3, 4);
INDArray b = a.ravel();

// Writes -1 at flat index 5, which is a[1][1] in the 3x4 array
b.putScalar(5, -1.0f);

System.out.println(a.getFloat(1, 1));  // -1.0  — a was also changed
```

Use `dup()` after reshape if you need an independent copy:

```java
INDArray independent = x.reshape(2, 6).dup();
```

### Transpose

`transpose()` returns a view with swapped strides. For 2D arrays this swaps rows and columns:

```java
INDArray m = Nd4j.createFromArray(new float[][]{{1,2,3},{4,5,6}});
// shape [2, 3]

INDArray t = m.transpose();
// shape [3, 2]
// [[1, 4],
//  [2, 5],
//  [3, 6]]

// Transpose is a view — dup() for an independent copy
INDArray tCopy = m.transpose().dup();
```


## Stacking Arrays

`vstack` concatenates along dimension 0 (rows); `hstack` concatenates along dimension 1 (columns). Both accept varargs.

```java
INDArray a = Nd4j.createFromArray(new float[][]{{1, 2}, {3, 4}});
INDArray b = Nd4j.createFromArray(new float[][]{{5, 6}, {7, 8}});

INDArray v = Nd4j.vstack(a, b);
// [[1, 2],
//  [3, 4],
//  [5, 6],
//  [7, 8]]   shape [4, 2]

INDArray h = Nd4j.hstack(a, b);
// [[1, 2, 5, 6],
//  [3, 4, 7, 8]]  shape [2, 4]

// General concatenation along any axis
INDArray cat0 = Nd4j.concat(0, a, b);   // same as vstack
INDArray cat1 = Nd4j.concat(1, a, b);   // same as hstack
```


## Copies vs Views

Getting this right prevents subtle bugs.

### Reference assignment — no copy

Java passes objects by reference. Assigning one `INDArray` variable to another gives you a second reference to the same object and the same underlying memory.

```java
INDArray x = Nd4j.rand(DataType.FLOAT, 2, 3);
INDArray y = x;          // y IS x — same object, same data

y.addi(1);               // modifies x too, since y and x point to the same buffer
System.out.println(x);  // x has changed
```

### Views — shallow copy

`reshape`, `ravel`, `transpose`, `getRow`, `getColumn`, and `NDArrayIndex` slices all return **views**: new `INDArray` objects that share the same backing buffer. Mutating a view mutates the original.

```java
INDArray x = Nd4j.rand(DataType.FLOAT, 3, 4);
INDArray flat = x.ravel();          // view — same buffer
INDArray reshaped = x.reshape(6, 2); // view — same buffer

flat.putScalar(2, -99f);

// All three variables now show -99 at the corresponding position
System.out.println(x);         // element [0][2] is -99
System.out.println(flat);       // element [2] is -99
System.out.println(reshaped);   // element [1][0] is -99
```

### Deep copy with dup()

`dup()` allocates a new buffer and copies all data into it. After calling `dup()`, the two arrays are completely independent.

```java
INDArray x = Nd4j.rand(DataType.FLOAT, 3, 4);
INDArray flat = x.ravel().dup();   // independent copy — new buffer

flat.putScalar(2, -99f);

// x is unchanged
System.out.println(x.getFloat(0, 2));  // original value, not -99
```

As a rule: if you are going to modify a slice or reshaped view and do not want the original to change, call `.dup()` on it first.


## Quick Reference

### Array creation

| Method | Description |
|---|---|
| `Nd4j.zeros(DataType, long...)` | All-zeros array |
| `Nd4j.ones(DataType, long...)` | All-ones array |
| `Nd4j.rand(DataType, long...)` | Uniform random in [0, 1) |
| `Nd4j.randn(DataType, long...)` | Standard-normal random |
| `Nd4j.createFromArray(float[][])` | From Java array (shape inferred) |
| `Nd4j.arange(start, stop)` | Integer range, exclusive of stop |
| `Nd4j.linspace(DataType, start, stop, count)` | Evenly-spaced points |
| `Nd4j.eye(n)` | n x n identity matrix |

### Element-wise arithmetic

| Op | Copy | In-place |
|---|---|---|
| Addition | `a.add(b)` | `a.addi(b)` |
| Subtraction | `a.sub(b)` | `a.subi(b)` |
| Multiplication | `a.mul(b)` | `a.muli(b)` |
| Division | `a.div(b)` | `a.divi(b)` |

### Reductions

| Method | Result |
|---|---|
| `x.sum()` | Global sum (scalar INDArray) |
| `x.sum(dim)` | Sum along dimension `dim` |
| `x.min()` / `x.min(dim)` | Minimum |
| `x.max()` / `x.max(dim)` | Maximum |
| `x.mean()` / `x.mean(dim)` | Mean |
| `x.sumNumber().doubleValue()` | Global sum as Java `double` |

### Transforms (Transforms class)

| Method | Description |
|---|---|
| `Transforms.sin(x)` | Sine |
| `Transforms.cos(x)` | Cosine |
| `Transforms.exp(x)` | Exponential e^x |
| `Transforms.sqrt(x)` | Square root |
| `Transforms.log(x)` | Natural logarithm |
| `Transforms.abs(x)` | Absolute value |
| `Transforms.relu(x)` | max(0, x) |
| `Transforms.sigmoid(x)` | 1 / (1 + e^-x) |

### Shape manipulation

| Method | Returns | Notes |
|---|---|---|
| `x.reshape(long...)` | INDArray | View; total elements unchanged |
| `x.ravel()` | INDArray | View; 1D flat |
| `x.transpose()` | INDArray | View; swapped strides |
| `x.dup()` | INDArray | Independent deep copy |
| `Nd4j.vstack(a, b)` | INDArray | Concatenate along rows |
| `Nd4j.hstack(a, b)` | INDArray | Concatenate along columns |

---

For a deeper treatment of `DataType`, memory ordering, and workspace-based memory management, see the [Tensors and NDArrays](../core-concepts/tensors-and-ndarrays) page.

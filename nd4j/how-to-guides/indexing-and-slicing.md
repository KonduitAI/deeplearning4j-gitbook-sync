---
title: "Indexing and Slicing"
description: "Accessing and modifying elements, rows, columns, and sub-arrays of INDArrays using NDArrayIndex"
---

# Indexing and Slicing NDArrays

This page explains how to read and write individual values, rows, columns, and arbitrary sub-arrays of an `INDArray`. It also covers sequential iteration, tensor-along-dimension, boolean indexing, and converting NDArrays to plain Java arrays.

## A note on views vs. copies

Many of the operations described on this page return a **view**, not a copy. A view is an `INDArray` that shares the same underlying off-heap memory as the original array. Mutations made through the view are immediately visible in the original, and vice-versa.

Operations that return views include `getRow()`, `getColumn()`, `get(NDArrayIndex...)` with `NDArrayIndex.point()`, `.all()`, or `.interval()`, `tensorAlongDimension()`, `transpose()`, and `reshape()`.

Operations that return copies (new allocations) include `getRows(int...)`, `dup()`, and `toDoubleVector()` / `toDoubleMatrix()`.

When you need an independent copy of a view, call `.dup()` on the result:

```java
INDArray rowCopy = myArray.getRow(0).dup(); // independent copy
```

---

## 1. Getting and Setting Individual Values

For an `INDArray` of rank N, you need N indices to address a single element. Indexing is **zero-based**: rows range from `0` to `size(0)-1`, columns from `0` to `size(1)-1`, and so on.

> **Performance note:** Reading or writing one element at a time in a loop is expensive because each call crosses the JVM/off-heap boundary. Prefer bulk operations (ops, slices, `assign()`) whenever possible.

### Reading single values

```java
INDArray arr = Nd4j.create(new double[][]{
    {1.0, 2.0, 3.0},
    {4.0, 5.0, 6.0},
    {7.0, 8.0, 9.0}
});

// getDouble(int row, int col)  -- 2D shorthand
double val = arr.getDouble(1, 2);
// 6.0

// getDouble(int...)  -- works for any rank
double same = arr.getDouble(1, 2);
// 6.0

// getFloat / getInt -- same signatures, different return type
float f = arr.getFloat(0, 0);
// 1.0

int i = arr.getInt(2, 1);
// 8
```

For 3D or higher arrays supply three or more indices:

```java
INDArray cube = Nd4j.arange(24).reshape(2, 3, 4);
// shape [2, 3, 4]

double v = cube.getDouble(1, 2, 3);
// 23.0  (element at depth=1, row=2, col=3)
```

### Writing single values: `putScalar`

```java
INDArray arr = Nd4j.zeros(3, 3);

// putScalar(int[], double)
arr.putScalar(new int[]{0, 0}, 10.0);
arr.putScalar(new int[]{1, 1}, 20.0);
arr.putScalar(new int[]{2, 2}, 30.0);

System.out.println(arr);
// [[10.00,  0.00,  0.00],
//  [ 0.00, 20.00,  0.00],
//  [ 0.00,  0.00, 30.00]]
```

Overloads also accept `float` and `int` values:

```java
arr.putScalar(new int[]{0, 1}, 5.0f);  // float overload
arr.putScalar(new int[]{0, 2}, 7);     // int overload
```

---

## 2. Row and Column Access

### Getting rows

`getRow(int)` returns a **view** of a single row as a row vector:

```java
INDArray arr = Nd4j.create(new double[][]{
    {1.0, 2.0, 3.0},
    {4.0, 5.0, 6.0},
    {7.0, 8.0, 9.0}
});

INDArray row1 = arr.getRow(1);
System.out.println(row1);
// [4.00, 5.00, 6.00]
```

Because `row1` is a view, modifying it modifies `arr` as well:

```java
row1.addi(100.0);
System.out.println(arr);
// [[  1.00,   2.00,   3.00],
//  [104.00, 105.00, 106.00],
//  [  7.00,   8.00,   9.00]]
```

To add to a row without keeping a reference, chain the call directly:

```java
arr.getRow(0).addi(1.0);  // adds 1.0 to every element in row 0 in place
```

### Getting multiple rows

`getRows(int...)` stacks the requested rows into a new matrix. This returns a **copy**, not a view:

```java
INDArray rows = arr.getRows(0, 2);
System.out.println(rows);
// [[1.00, 2.00, 3.00],
//  [7.00, 8.00, 9.00]]
```

### Setting a row: `putRow`

```java
INDArray newRow = Nd4j.create(new double[]{10.0, 20.0, 30.0});
arr.putRow(0, newRow);
System.out.println(arr);
// [[10.00, 20.00, 30.00],
//  [ 4.00,  5.00,  6.00],
//  [ 7.00,  8.00,  9.00]]
```

### Getting a column

`getColumn(int)` returns a **view** of a single column as a column vector:

```java
INDArray col0 = arr.getColumn(0);
System.out.println(col0);
// [[10.00],
//  [ 4.00],
//  [ 7.00]]
```

Like `getRow`, this is a view -- writes to `col0` affect `arr`.

---

## 3. NDArrayIndex-Based Access

The `INDArray.get(NDArrayIndex...)` family provides the most general sub-array access. You supply one `NDArrayIndex` per dimension; ND4J resolves which elements to include along each axis and returns a view of the result.

Import required:

```java
import org.nd4j.linalg.indexing.NDArrayIndex;
```

### `NDArrayIndex.point(int)`

Selects a single index along a dimension, collapsing that dimension:

```java
INDArray arr = Nd4j.arange(12).reshape(4, 3);
// [[ 0,  1,  2],
//  [ 3,  4,  5],
//  [ 6,  7,  8],
//  [ 9, 10, 11]]

// Single row, all columns
INDArray row2 = arr.get(NDArrayIndex.point(2), NDArrayIndex.all());
System.out.println(row2);
// [6.00, 7.00, 8.00]
```

### `NDArrayIndex.all()`

Selects every index along a dimension (equivalent to `:` in NumPy):

```java
// All rows, single column
INDArray col1 = arr.get(NDArrayIndex.all(), NDArrayIndex.point(1));
System.out.println(col1);
// [1.00, 4.00, 7.00, 10.00]
```

### `NDArrayIndex.interval(int from, int to)`

Selects indices `[from, to)` (inclusive start, exclusive end):

```java
// Rows 1 to 2 (inclusive), all columns
INDArray sub = arr.get(NDArrayIndex.interval(1, 3), NDArrayIndex.all());
System.out.println(sub);
// [[ 3.00,  4.00,  5.00],
//  [ 6.00,  7.00,  8.00]]
```

### `NDArrayIndex.interval(int from, int stride, int to)`

Selects every `stride`-th index between `from` (inclusive) and `to` (exclusive):

```java
// All rows, every second column (columns 0, 2)
INDArray everyOther = arr.get(NDArrayIndex.all(), NDArrayIndex.interval(0, 2, 3));
System.out.println(everyOther);
// [[ 0.00,  2.00],
//  [ 3.00,  5.00],
//  [ 6.00,  8.00],
//  [ 9.00, 11.00]]
```

### `NDArrayIndex.specified(long...)`

Selects an explicit, possibly non-contiguous set of indices. This is the equivalent of fancy/advanced indexing:

```java
// Columns 0 and 2 only
INDArray specified = arr.get(
    NDArrayIndex.all(),
    NDArrayIndex.specified(0, 2)
);
System.out.println(specified);
// [[ 0.00,  2.00],
//  [ 3.00,  5.00],
//  [ 6.00,  8.00],
//  [ 9.00, 11.00]]
```

### Combining NDArrayIndex types

All four index types can be mixed freely across dimensions:

```java
INDArray cube = Nd4j.arange(60).reshape(3, 4, 5);

// Depth 0 only, rows 1-2, columns 2-4
INDArray slice = cube.get(
    NDArrayIndex.point(0),
    NDArrayIndex.interval(1, 3),
    NDArrayIndex.interval(2, 5)
);
// shape [2, 3]
```

### Extending to 1D arrays

For a 1D array (vector) provide a single `NDArrayIndex`:

```java
INDArray x = Nd4j.arange(12);
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]

INDArray slice = x.get(NDArrayIndex.interval(2, 6));
System.out.println(slice);
// [2.00, 3.00, 4.00, 5.00]
```

---

## 4. Put Operations: Writing to Sub-Arrays

### `put(INDArrayIndex[], INDArray)`

The counterpart to `get` -- writes the values of `toPut` into the sub-array identified by the index array:

```java
INDArray arr = Nd4j.zeros(4, 4);

INDArray patch = Nd4j.ones(2, 2).muli(7.0);
arr.put(
    new INDArrayIndex[]{
        NDArrayIndex.interval(1, 3),
        NDArrayIndex.interval(1, 3)
    },
    patch
);

System.out.println(arr);
// [[0.00, 0.00, 0.00, 0.00],
//  [0.00, 7.00, 7.00, 0.00],
//  [0.00, 7.00, 7.00, 0.00],
//  [0.00, 0.00, 0.00, 0.00]]
```

The shape of `patch` must match the shape implied by the provided indices, or ND4J will throw.

### Equivalence with `get().assign()`

Because `get(NDArrayIndex...)` returns a view, the two forms below are exactly equivalent:

```java
// Form 1: put
arr.put(new INDArrayIndex[]{NDArrayIndex.point(0), NDArrayIndex.all()}, newRow);

// Form 2: get view and assign
arr.get(NDArrayIndex.point(0), NDArrayIndex.all()).assign(newRow);
```

Both overwrite the data in `arr` in place. Choose whichever reads more clearly.

### Assigning a scalar to a slice

Combine `get()` and `assign(double)` to fill a region with a constant:

```java
INDArray arr = Nd4j.arange(12).reshape(3, 4);

// Set columns 1 and 2 of all rows to -1
arr.get(NDArrayIndex.all(), NDArrayIndex.interval(1, 3)).assign(-1.0);

System.out.println(arr);
// [[ 0.00, -1.00, -1.00,  3.00],
//  [ 4.00, -1.00, -1.00,  7.00],
//  [ 8.00, -1.00, -1.00, 11.00]]
```

---

## 5. Tensor Along Dimension

Tensor Along Dimension (TAD) extracts a lower-rank sub-array from a higher-rank array. The result is always a **view**. TAD is particularly useful when you need to apply the same operation to every "slice" of an array along some set of dimensions.

### Core method signatures

```java
// Return the i-th tensor along the specified dimension(s)
INDArray INDArray.tensorAlongDimension(int index, int... dimensions)

// Return the total number of tensors along the specified dimension(s)
long INDArray.tensorssAlongDimension(int... dimensions)
```

Note the double-s in `tensorssAlongDimension` -- that is the actual method name.

### 2D example

```java
// 3x3 matrix, values 1-9
INDArray arr = Nd4j.create(new double[]{1,5,2,4,3,5,5,4,5}, new int[]{3,3});
// [[1, 5, 2],
//  [4, 3, 5],
//  [5, 4, 5]]

// Number of tensors along dim 0 (rows)
long n0 = arr.tensorssAlongDimension(0);  // 3

// 0th tensor along dim 0: values down column 0 -> [1, 4, 5]
INDArray t0 = arr.tensorAlongDimension(0, 0);
System.out.println(t0);
// [1.00, 4.00, 5.00]

// 1st tensor along dim 0: values down column 1 -> [5, 3, 4]
INDArray t1 = arr.tensorAlongDimension(1, 0);
System.out.println(t1);
// [5.00, 3.00, 4.00]

// 1st tensor along dim 1: values across row 1 -> [4, 3, 5]
INDArray t2 = arr.tensorAlongDimension(1, 1);
System.out.println(t2);
// [4.00, 3.00, 5.00]
```

### Shape rules for TAD

The shape of each returned tensor and the number of tensors are determined as follows:

| Input shape | TAD dimensions | Number of tensors | Tensor shape |
|-------------|---------------|-------------------|--------------|
| `[a, b, c]` | `(0)`         | `b * c`           | `[1, a]`     |
| `[a, b, c]` | `(1)`         | `a * c`           | `[1, b]`     |
| `[a, b, c]` | `(0, 1)`      | `c`               | `[a, b]`     |
| `[a, b, c]` | `(1, 2)`      | `a`               | `[b, c]`     |
| `[a,b,c,d]` | `(1, 2)`      | `a * d`           | `[b, c]`     |
| `[a,b,c,d]` | `(0, 2, 3)`   | `b`               | `[a, c, d]`  |

### 3D example: iterating over 2D slices

```java
INDArray volume = Nd4j.arange(24).reshape(2, 3, 4);
// shape [2, 3, 4]

long numSlices = volume.tensorssAlongDimension(1, 2); // = 2
for (int i = 0; i < numSlices; i++) {
    INDArray slice = volume.tensorAlongDimension(i, 1, 2);
    // Each slice has shape [3, 4]
    System.out.println("Slice " + i + ":\n" + slice);
}
// Slice 0:
// [[ 0,  1,  2,  3],
//  [ 4,  5,  6,  7],
//  [ 8,  9, 10, 11]]
// Slice 1:
// [[12, 13, 14, 15],
//  [16, 17, 18, 19],
//  [20, 21, 22, 23]]
```

Because TAD returns views, you can mutate each slice and the changes propagate back to `volume`.

---

## 6. Converting to Java Arrays

Sometimes you need a plain Java array for interoperability with existing code. ND4J provides several conversion methods. All of them allocate new Java heap arrays (copies, not views).

### 1D arrays

```java
INDArray vec = Nd4j.arange(5);

double[] dv = vec.toDoubleVector();
// [0.0, 1.0, 2.0, 3.0, 4.0]

float[] fv = vec.toFloatVector();
// [0.0, 1.0, 2.0, 3.0, 4.0]
```

### 2D arrays

```java
INDArray mat = Nd4j.create(new double[][]{{1,2,3},{4,5,6}});

double[][] dm = mat.toDoubleMatrix();
// [[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]]

float[][] fm = mat.toFloatMatrix();
// [[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]]
```

These methods only work for 1D and 2D arrays respectively. For higher-rank arrays, flatten first with `ravel()` or use `getDouble(int[])` in a loop.

---

## 7. NdIndexIterator: Sequential Traversal

`NdIndexIterator` traverses all elements of an `INDArray` in C-order (row-major), producing the index tuple for each element in turn. The traversal order for a rank-2 array is `[0,0], [0,1], ..., [0,n-1], [1,0], ...`.

```java
import org.nd4j.linalg.util.NdIndexIterator;  // also written NdIndexIterator

INDArray arr = Nd4j.create(new double[][]{{1,2,3},{4,5,6}});
// [[1, 2, 3],
//  [4, 5, 6]]

NdIndexIterator iter = new NdIndexIterator(arr.rows(), arr.columns());
while (iter.hasNext()) {
    long[] idx = iter.next();
    double val = arr.getDouble(idx);
    System.out.printf("[%d, %d] = %.1f%n", idx[0], idx[1], val);
}
// [0, 0] = 1.0
// [0, 1] = 2.0
// [0, 2] = 3.0
// [1, 0] = 4.0
// [1, 1] = 5.0
// [1, 2] = 6.0
```

For arrays with 3 or more dimensions, pass the full shape to the constructor:

```java
long[] shape = arr.shape();
NdIndexIterator iter3d = new NdIndexIterator(shape[0], shape[1], shape[2]);
```

`NdIndexIterator` is useful when you genuinely need to visit every element, for example to build a sparse representation or apply a predicate that cannot be vectorised. For most bulk operations, use ND4J ops instead.

---

## 8. Boolean Indexing

The `BooleanIndexing` class applies operations to elements of an array that satisfy a condition, without needing explicit index calculations.

```java
import org.nd4j.linalg.indexing.BooleanIndexing;
import org.nd4j.linalg.indexing.conditions.Conditions;
```

### Replace values that match a condition

```java
INDArray arr = Nd4j.create(new double[]{-3.0, -1.0, 0.0, 2.0, 5.0});

// Replace all negative values with 0
BooleanIndexing.replaceWhere(arr, 0.0, Conditions.lessThan(0.0));

System.out.println(arr);
// [0.00, 0.00, 0.00, 2.00, 5.00]
```

### Apply a value from a second array where condition holds

```java
INDArray arr  = Nd4j.create(new double[]{1.0, 2.0, 3.0, 4.0, 5.0});
INDArray mask = Nd4j.create(new double[]{9.0, 9.0, 9.0, 9.0, 9.0});

// Where arr > 3, take the value from mask instead
BooleanIndexing.replaceWhere(arr, mask, Conditions.greaterThan(3.0));

System.out.println(arr);
// [1.00, 2.00, 3.00, 9.00, 9.00]
```

### Available conditions

| Factory method | Meaning |
|----------------|---------|
| `Conditions.lessThan(x)` | element < x |
| `Conditions.lessThanOrEqual(x)` | element <= x |
| `Conditions.greaterThan(x)` | element > x |
| `Conditions.greaterThanOrEqual(x)` | element >= x |
| `Conditions.equals(x)` | element == x |
| `Conditions.notEquals(x)` | element != x |
| `Conditions.isNan()` | element is NaN |
| `Conditions.isInfinite()` | element is +/-Inf |
| `Conditions.isFinite()` | element is finite (not NaN, not Inf) |
| `Conditions.absGreaterThan(x)` | |element| > x |

The full list and the unit tests for `BooleanIndexing` can be found in the
[BooleanIndexingTest](https://github.com/deeplearning4j/nd4j/blob/master/nd4j-backends/nd4j-tests/src/test/java/org/nd4j/linalg/indexing/BooleanIndexingTest.java)
source file.

---

## 9. Views in Depth: What Is and Is Not a View

Understanding whether an operation returns a view or a copy prevents subtle bugs.

### Operations that return views

```java
INDArray a = Nd4j.arange(12).reshape(3, 4);

INDArray v1 = a.getRow(0);                                    // view
INDArray v2 = a.getColumn(2);                                 // view
INDArray v3 = a.get(NDArrayIndex.interval(0, 2), NDArrayIndex.all()); // view
INDArray v4 = a.get(NDArrayIndex.point(1), NDArrayIndex.all()); // view
INDArray v5 = a.transpose();                                  // view
INDArray v6 = a.reshape(4, 3);                                // view (usually)
INDArray v7 = a.tensorAlongDimension(0, 1);                   // view
```

Modifying any of `v1`-`v7` will modify `a` (and vice versa).

### Operations that return copies

```java
INDArray c1 = a.getRows(0, 2);    // copy -- non-contiguous rows cannot share buffer
INDArray c2 = a.dup();            // explicit deep copy
double[] c3 = a.toDoubleVector(); // Java array copy
```

### Forcing a copy

Call `.dup()` on any view to obtain an independent array:

```java
INDArray rowView = a.getRow(1);
INDArray rowCopy = a.getRow(1).dup();

rowCopy.addi(999.0); // does NOT affect a
rowView.addi(999.0); // DOES affect a
```

---

## Quick Reference

| Goal | Method | Returns |
|------|--------|---------|
| Single element (double) | `arr.getDouble(int, int)` | double |
| Single element (float) | `arr.getFloat(int)` | float |
| Single element (any rank) | `arr.getDouble(new int[]{i,j,k})` | double |
| Write single element | `arr.putScalar(new int[]{i,j}, v)` | INDArray (this) |
| Whole row | `arr.getRow(int)` | VIEW |
| Multiple rows | `arr.getRows(int...)` | copy |
| Write a row | `arr.putRow(int, INDArray)` | INDArray (this) |
| Whole column | `arr.getColumn(int)` | VIEW |
| Point index | `arr.get(NDArrayIndex.point(i), ...)` | VIEW |
| Range | `arr.get(NDArrayIndex.interval(a,b), ...)` | VIEW |
| Strided range | `arr.get(NDArrayIndex.interval(a,stride,b), ...)` | VIEW |
| All elements on axis | `arr.get(NDArrayIndex.all(), ...)` | VIEW |
| Explicit indices | `arr.get(NDArrayIndex.specified(0,2), ...)` | VIEW |
| Write sub-array | `arr.put(INDArrayIndex[], INDArray)` | INDArray (this) |
| TAD count | `arr.tensorssAlongDimension(int...)` | long |
| TAD slice | `arr.tensorAlongDimension(int, int...)` | VIEW |
| Java double[] | `arr.toDoubleVector()` | copy |
| Java double[][] | `arr.toDoubleMatrix()` | copy |
| Java float[] | `arr.toFloatVector()` | copy |
| Sequential iteration | `new NdIndexIterator(long...)` | iterator |
| Boolean replace | `BooleanIndexing.replaceWhere(arr, val, cond)` | void (in-place) |

---

## See Also

- [ND4J Overview](../nd4j-overview) -- full programming guide including creation, operations, and serialization
- [ND4J Quickstart](../nd4j-quickstart) -- brief tour of the most common operations
- [INDArray Javadoc](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html)
- [NDArrayIndex Javadoc](https://deeplearning4j.org/api/latest/org/nd4j/linalg/indexing/NDArrayIndex.html)
- [BooleanIndexing Javadoc](https://deeplearning4j.org/api/latest/org/nd4j/linalg/indexing/BooleanIndexing.html)

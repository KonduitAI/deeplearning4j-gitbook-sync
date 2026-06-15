---
title: "Operations"
description: "Scalar, element-wise, transform, reduction, broadcast, comparison, and linear algebra operations on INDArrays"
---

# ND4J Operations

ND4J defines five categories of operations that can be performed on `INDArray` objects:

- **Scalar ops** — apply a single number to every element
- **Transform ops** — element-wise mathematical functions (tanh, exp, log, etc.)
- **Accumulation (reduction) ops** — collapse an array to a scalar or lower-rank array
- **Index accumulation ops** — return the index of a value (argmax, argmin)
- **Broadcast / vector ops** — add or multiply a row/column vector across a matrix

Each category can be executed over the entire array or along one or more dimensions.

---

## In-Place vs. Copy Operations

Every mutating operation in ND4J exists in two variants. The naming convention is consistent across the entire API:

| Variant | Suffix | Effect |
|---------|--------|--------|
| Copy | *(none)* | Returns a new `INDArray`; the original is unchanged |
| In-place | `i` | Modifies the original array and returns a reference to it |

```java
INDArray x = Nd4j.create(new double[]{1, 2, 3, 4}, new int[]{2, 2});
INDArray y = Nd4j.create(new double[]{10, 20, 30, 40}, new int[]{2, 2});

// Copy: x is not modified; z is a new array
INDArray z = x.add(y);

// In-place: x is modified directly; result is the same object as x
x.addi(y);
```

After `x.add(y)`, the variable `x` is still `[[1,2],[3,4]]` and `z` holds `[[11,22],[33,44]]`.  
After `x.addi(y)`, `x` itself becomes `[[11,22],[33,44]]`.

This convention applies uniformly: `sub`/`subi`, `mul`/`muli`, `div`/`divi`, `rsub`/`rsubi`, `rdiv`/`rdivi`, and so on.

> **Tip:** When chaining operations on a view (e.g., a row returned by `getRow()`), in-place operations modify the backing array without allocating new memory. Use this deliberately:
> ```java
> // Add 1.0 to every element of row 2 in-place, no allocation
> myMatrix.getRow(2).addi(1.0);
> ```

---

## Data Type Requirement

ND4J requires that both operands in a binary operation share the same `DataType`. Mixing float and double arrays, for example, will produce an error at runtime.

Use `castTo(DataType)` to convert an array before operating on it:

```java
import org.nd4j.linalg.api.buffer.DataType;

INDArray floatArr  = Nd4j.ones(DataType.FLOAT, 3, 3);
INDArray doubleArr = Nd4j.ones(DataType.DOUBLE, 3, 3);

// Cast float to double before the operation
INDArray result = floatArr.castTo(DataType.DOUBLE).add(doubleArr);
```

Common `DataType` values: `FLOAT`, `DOUBLE`, `LONG`, `INT`, `SHORT`, `HALF`, `BFLOAT16`.

To change the default type for all newly created arrays:

```java
Nd4j.setDefaultDataTypes(DataType.DOUBLE, DataType.DOUBLE);
```

---

## 1. Scalar Operations

Scalar ops apply a single numeric value to every element of an `INDArray`. ND4J exposes the most common ones directly on the `INDArray` interface.

### Standard scalar ops

```java
INDArray arr = Nd4j.create(new double[]{1, 2, 3, 4, 5, 6}, new int[]{2, 3});

// Copy variants — arr is unchanged, result is a new array
INDArray added   = arr.add(10.0);    // every element + 10
INDArray subbed  = arr.sub(1.0);     // every element - 1
INDArray mulled  = arr.mul(2.0);     // every element * 2
INDArray divided = arr.div(4.0);     // every element / 4

// In-place variants — arr is modified directly
arr.addi(10.0);
arr.subi(1.0);
arr.muli(2.0);
arr.divi(4.0);
```

### Reverse scalar ops

The "reverse" variants swap the operands, computing `scalar OP element` instead of `element OP scalar`. This is most useful for subtraction and division:

```java
INDArray arr = Nd4j.linspace(1, 4, 4).reshape(2, 2);

// rsub: scalar - each element  → 10 - [1,2,3,4] = [9,8,7,6]
INDArray rsubResult = arr.rsub(10.0);

// rdiv: scalar / each element  → 1.0 / [1,2,3,4] = [1, 0.5, 0.333, 0.25]
INDArray rdivResult = arr.rdiv(1.0);

// In-place versions
arr.rsubi(10.0);
arr.rdivi(1.0);
```

### Scalar ops via the executor (advanced)

You can also invoke scalar ops directly through the op executor when you need finer control:

```java
import org.nd4j.linalg.api.ops.impl.scalar.ScalarAdd;

INDArray arr = Nd4j.ones(3, 3);
// Note: this modifies arr in-place
Nd4j.getExecutioner().execAndReturn(new ScalarAdd(arr, 1.0));
```

---

## 2. Element-Wise Operations

Element-wise (Hadamard) ops pair each element of one array with the corresponding element of a second array of the same shape.

### Basic element-wise ops

```java
INDArray a = Nd4j.create(new double[]{1, 2, 3, 4}, new int[]{2, 2});
INDArray b = Nd4j.create(new double[]{5, 6, 7, 8}, new int[]{2, 2});

// Copy variants
INDArray sum    = a.add(b);   // [[6,8],[10,12]]
INDArray diff   = a.sub(b);   // [[-4,-4],[-4,-4]]
INDArray product = a.mul(b);  // [[5,12],[21,32]]  — Hadamard product
INDArray quot   = a.div(b);   // [[0.2,0.333],[0.428,0.5]]

// In-place variants (a is modified)
a.addi(b);
a.subi(b);
a.muli(b);
a.divi(b);
```

### Assignment

`assign` copies all values from one array into another. The target array is modified in-place:

```java
INDArray target = Nd4j.zeros(2, 2);
INDArray source = Nd4j.ones(2, 2);

target.assign(source);   // target is now [[1,1],[1,1]]
// source is unchanged
```

This is equivalent to `target.get(...).assign(source)` when operating on a view, and is the preferred way to set a block of values without allocating a new array.

---

## 3. Transform Operations

Transform ops apply a mathematical function element-wise across an entire array. ND4J provides two convenient entry points: the `Transforms` utility class and `Nd4j.getExecutioner().execAndReturn()`.

### Using the Transforms class

```java
import org.nd4j.linalg.ops.transforms.Transforms;

INDArray arr = Nd4j.linspace(-1, 1, 9).reshape(3, 3);

// Trigonometric
INDArray sinResult  = Transforms.sin(arr);
INDArray cosResult  = Transforms.cos(arr);
INDArray tanResult  = Transforms.tan(arr);

// Exponential / logarithmic
INDArray expResult  = Transforms.exp(arr);         // e^x
INDArray logResult  = Transforms.log(arr);         // natural log ln(x)
INDArray sqrtResult = Transforms.sqrt(arr);        // √x

// Activation functions
INDArray tanhResult    = Transforms.tanh(arr);
INDArray sigResult     = Transforms.sigmoid(arr);
INDArray reluResult    = Transforms.relu(arr);     // max(0, x)
INDArray softplusRes   = Transforms.softPlus(arr);

// Other
INDArray absResult     = Transforms.abs(arr);
INDArray floorResult   = Transforms.floor(arr);
INDArray ceilResult    = Transforms.ceil(arr);
INDArray roundResult   = Transforms.round(arr);
```

By default, `Transforms` methods return a **copy** (the original is unchanged). Pass `false` as the second argument to operate in-place:

```java
// in-place tanh — arr itself is overwritten
Transforms.tanh(arr, false);

// copy tanh — arr is unchanged, result is a new array
INDArray result = Transforms.tanh(arr, true);
// equivalently:
INDArray result2 = Transforms.tanh(arr);
```

### Softmax along a dimension

Softmax is typically applied along dimension 1 (across columns of each row):

```java
INDArray logits = Nd4j.create(new double[]{1, 2, 3, 2, 1, 0}, new int[]{2, 3});
INDArray probs  = Transforms.softmax(logits);    // applied along last dimension
```

### Using the executor directly

```java
import org.nd4j.linalg.api.ops.impl.transforms.strict.Tanh;

INDArray arr = Nd4j.rand(3, 3);

// In-place: arr is modified and returned
INDArray tanhArr = Nd4j.getExecutioner().execAndReturn(new Tanh(arr));

// To preserve the original, dup() first:
INDArray tanhCopy = Nd4j.getExecutioner().execAndReturn(new Tanh(arr.dup()));
```

You can also create a transform by name using the op factory:

```java
INDArray result = Nd4j.getExecutioner().execAndReturn(
    Nd4j.getOpFactory().createTransform("tanh", arr)
);
```

---

## 4. Accumulation / Reduction Operations

Reduction ops collapse array values into a summary statistic. They can run over the **entire array** (returning a scalar) or **along one or more dimensions** (returning a lower-rank array).

### Full-array reductions

```java
INDArray arr = Nd4j.create(new double[]{1, 2, 3, 4, 5, 6}, new int[]{2, 3});

double sum    = arr.sumNumber().doubleValue();      // 21.0
double mean   = arr.meanNumber().doubleValue();     // 3.5
double min    = arr.minNumber().doubleValue();      // 1.0
double max    = arr.maxNumber().doubleValue();      // 6.0
double std    = arr.stdNumber(true).doubleValue();  // population std dev
double var    = arr.varNumber(true).doubleValue();  // population variance
double norm1  = arr.norm1Number().doubleValue();    // L1 norm: sum of |x_i|
double norm2  = arr.norm2Number().doubleValue();    // L2 norm: √(sum of x_i²)
```

All of these return a `Number`; call `.doubleValue()`, `.floatValue()`, or `.longValue()` as needed.

### Dimension-wise reductions

Passing a dimension index to the reduction narrows the scope of the operation. The result has one fewer dimension for each axis reduced.

For a 2D array with shape `[rows, cols]`:
- `sum(0)` sums down each column → result shape `[1, cols]`  
- `sum(1)` sums across each row → result shape `[rows, 1]`

```java
// Shape: [3, 4]
INDArray arr = Nd4j.rand(3, 4);

INDArray colSums  = arr.sum(0);     // shape [1, 4] — sum of each column
INDArray rowSums  = arr.sum(1);     // shape [3, 1] — sum of each row
INDArray colMeans = arr.mean(0);    // shape [1, 4]
INDArray rowMeans = arr.mean(1);    // shape [3, 1]
INDArray colMax   = arr.max(0);     // shape [1, 4]
INDArray rowMin   = arr.min(1);     // shape [3, 1]
INDArray colNorm2 = arr.norm2(0);   // shape [1, 4]
INDArray rowNorm1 = arr.norm1(1);   // shape [3, 1]
INDArray colStd   = arr.std(0);     // shape [1, 4]
INDArray rowVar   = arr.var(1);     // shape [3, 1]
```

Dimension-wise reductions generalize to higher-rank arrays. For a rank-3 array of shape `[a, b, c]`, calling `sum(1)` produces an array of shape `[a, 1, c]`.

### Reduction via executor (advanced)

```java
import org.nd4j.linalg.api.ops.impl.reduce.floating.Mean;

// Sum of the entire array
double sum = Nd4j.getExecutioner()
                 .execAndReturn(new org.nd4j.linalg.api.ops.impl.reduce.floating.Sum(arr))
                 .getFinalResult()
                 .doubleValue();

// Sum along dimension 0
INDArray colSums = Nd4j.getExecutioner().exec(
    new org.nd4j.linalg.api.ops.impl.reduce.floating.Sum(arr), 0
);
```

---

## 5. Index Accumulation Operations

Index accumulation ops return the **index** of the element that satisfies some condition (maximum, minimum, absolute maximum). They are the ND4J equivalents of NumPy's `argmax` / `argmin`.

### Full-array index accumulation

```java
import org.nd4j.linalg.api.ops.impl.indexaccum.IMax;
import org.nd4j.linalg.api.ops.impl.indexaccum.IMin;

INDArray arr = Nd4j.create(new double[]{3, 1, 4, 1, 5, 9, 2, 6}, new int[]{2, 4});

// Index of the maximum value in the flattened array
int idxMax = Nd4j.getExecutioner().execAndReturn(new IMax(arr)).getFinalResult();

// Index of the minimum value
int idxMin = Nd4j.getExecutioner().execAndReturn(new IMin(arr)).getFinalResult();
```

### Dimension-wise index accumulation

More commonly, you want the index of the max/min within each row or column:

```java
// argmax along dimension 0 (which row holds the max for each column?)
INDArray argmaxCols = Nd4j.getExecutioner().exec(new IMax(arr), 0);  // shape [1, 4]

// argmax along dimension 1 (which column holds the max for each row?)
INDArray argmaxRows = Nd4j.getExecutioner().exec(new IMax(arr), 1);  // shape [2, 1]

// argmin along dimension 1
INDArray argminRows = Nd4j.getExecutioner().exec(new IMin(arr), 1);  // shape [2, 1]
```

**Visual example** — given a 3×3 array:

```
[[4, 7, 2],
 [9, 1, 5],
 [3, 8, 6]]
```

`argmax(dim=0)` (index of max in each column): `[1, 2, 2]`  
`argmax(dim=1)` (index of max in each row): `[1, 0, 1]`

### IAMax — argmax of absolute values

```java
import org.nd4j.linalg.api.ops.impl.indexaccum.IAMax;

INDArray arr = Nd4j.create(new double[]{-7, 3, -2, 5});
// IAMax returns 0, because |-7| = 7 is the largest absolute value
int idx = Nd4j.getExecutioner().execAndReturn(new IAMax(arr)).getFinalResult();
```

---

## 6. Broadcast and Vector Operations

Vector ops broadcast a 1D (or rank-2 vector) array across every row or every column of a matrix.

### addRowVector / addColumnVector

```java
INDArray matrix = Nd4j.create(new double[]{1, 2, 3, 4, 5, 6}, new int[]{3, 2});
// [[1, 2],
//  [3, 4],
//  [5, 6]]

INDArray rowVec = Nd4j.create(new double[]{10, 20}, new int[]{1, 2});  // shape [1, 2]
INDArray colVec = Nd4j.create(new double[]{10, 20, 30}, new int[]{3, 1}); // shape [3, 1]

// Add the row vector to every row:
INDArray rResult = matrix.addRowVector(rowVec);
// [[11, 22],
//  [13, 24],
//  [15, 26]]

// Add the column vector to every column:
INDArray cResult = matrix.addColumnVector(colVec);
// [[11, 12],
//  [23, 24],
//  [35, 36]]
```

### muliColumnVector / muliRowVector (in-place)

```java
// Multiply each column by the column vector in-place
matrix.muliColumnVector(colVec);

// Multiply each row by the row vector in-place
matrix.muliRowVector(rowVec);
```

### Full set of broadcast vector methods

The following methods all follow the same in-place (`i`) / copy pattern:

| Copy | In-place | Operation |
|------|----------|-----------|
| `addRowVector(v)` | `addiRowVector(v)` | add row vector to each row |
| `subRowVector(v)` | `subiRowVector(v)` | subtract row vector from each row |
| `mulRowVector(v)` | `muliRowVector(v)` | multiply each row by row vector |
| `divRowVector(v)` | `diviRowVector(v)` | divide each row by row vector |
| `addColumnVector(v)` | `addiColumnVector(v)` | add column vector to each column |
| `subColumnVector(v)` | `subiColumnVector(v)` | subtract column vector from each column |
| `mulColumnVector(v)` | `muliColumnVector(v)` | multiply each column by column vector |
| `divColumnVector(v)` | `diviColumnVector(v)` | divide each column by column vector |

### General broadcast via Nd4j.exec

For more complex broadcast patterns, use `BroadcastAddOp` and related ops:

```java
import org.nd4j.linalg.api.ops.impl.broadcast.BroadcastAddOp;

INDArray matrix = Nd4j.rand(4, 5);
INDArray rowBias = Nd4j.rand(1, 5);

// Broadcast add along dimension 1
Nd4j.getExecutioner().exec(new BroadcastAddOp(matrix, rowBias, matrix, 1));
```

---

## 7. Comparison Operations

Comparison ops evaluate a condition element-wise and return a binary mask: `1.0` where the condition is true, `0.0` where it is false. The result is always a new array of the same shape; the original is never modified.

### Element-wise comparison against another array

```java
INDArray a = Nd4j.create(new double[]{1, 5, 3, 7, 2, 6}, new int[]{2, 3});
INDArray b = Nd4j.create(new double[]{2, 4, 3, 6, 3, 5}, new int[]{2, 3});

INDArray gtMask  = a.gt(b);   // a > b  → [[0,1,0],[1,0,1]]
INDArray ltMask  = a.lt(b);   // a < b  → [[1,0,0],[0,1,0]]
INDArray eqMask  = a.eq(b);   // a == b → [[0,0,1],[0,0,0]]
INDArray neqMask = a.neq(b);  // a != b → [[1,1,0],[1,1,1]]
```

### Element-wise comparison against a scalar

```java
INDArray arr = Nd4j.create(new double[]{1, 2, 3, 4, 5}, new int[]{1, 5});

INDArray above3 = arr.gt(3.0);   // [0, 0, 0, 1, 1]
INDArray below3 = arr.lt(3.0);   // [1, 1, 0, 0, 0]
INDArray is3    = arr.eq(3.0);   // [0, 0, 1, 0, 0]
INDArray not3   = arr.neq(3.0);  // [1, 1, 0, 1, 1]
```

### Using masks

Binary masks can be multiplied against an array to zero out unwanted values:

```java
// Keep only elements greater than 3
INDArray mask   = arr.gt(3.0);
INDArray filtered = arr.mul(mask);   // [0, 0, 0, 4, 5]
```

For more sophisticated conditional selection (applying an operation only where a mask is true), see the boolean indexing APIs in `BooleanIndexing`.

---

## 8. Linear Algebra Operations

### Matrix multiplication (mmul)

`mmul` computes the standard matrix product, not the Hadamard product. The number of columns in the left operand must equal the number of rows in the right operand.

```java
INDArray A = Nd4j.rand(3, 4);   // [3 × 4]
INDArray B = Nd4j.rand(4, 5);   // [4 × 5]

INDArray C = A.mmul(B);         // [3 × 5]
```

Inner product (row vector × column vector → scalar):

```java
INDArray row = Nd4j.create(new double[]{1, 2, 3}, new int[]{1, 3});
INDArray col = Nd4j.create(new double[]{4, 5, 6}, new int[]{3, 1});

INDArray dot = row.mmul(col);   // [[32]]  (1×4 + 2×5 + 3×6)
```

Outer product (column × row → matrix):

```java
INDArray outer = col.mmul(row); // [3 × 3]
```

### Transpose

`transpose()` returns a view (no copy) of the transposed array. Modifying the transpose modifies the original.

```java
INDArray mat = Nd4j.linspace(1, 6, 6).reshape(2, 3);  // [2 × 3]
INDArray t   = mat.transpose();                         // [3 × 2] view

// To get a standalone copy:
INDArray tCopy = mat.transpose().dup();
```

Transposing is O(1) because it only changes the stride metadata, not the underlying data buffer.

### Combined: transpose then multiply (t().mmul())

A common pattern is computing Aᵀ × A or A × Bᵀ:

```java
INDArray A = Nd4j.rand(4, 3);

// Compute AᵀA: [3×4] * [4×3] = [3×3]
INDArray AtA = A.transpose().mmul(A);

// Compute AAᵀ: [4×3] * [3×4] = [4×4]
INDArray AAt = A.mmul(A.transpose());
```

### Diagonal matrix

`Nd4j.diag()` has two behaviours depending on the shape of its argument:

```java
// Vector in → diagonal matrix out
INDArray vec  = Nd4j.create(new double[]{1, 2, 3});
INDArray diag = Nd4j.diag(vec);
// [[1, 0, 0],
//  [0, 2, 0],
//  [0, 0, 3]]

// Square matrix in → diagonal vector out
INDArray diagVec = Nd4j.diag(diag);   // [1, 2, 3]
```

### Matrix inverse

```java
import org.nd4j.linalg.inverse.InvertMatrix;

INDArray square = Nd4j.create(new double[]{1, 2, 3, 4}, new int[]{2, 2});

// Second argument: false = return new array, true = in-place
INDArray inv = InvertMatrix.invert(square, false);

// Verify: A * A⁻¹ ≈ I
INDArray identity = square.mmul(inv);
```

The invert method uses LU decomposition internally and works for any square non-singular matrix.

---

## Quick Reference

The table below summarises the most common operations and their in-place variants.

### Scalar ops

| Copy | In-place | Description |
|------|----------|-------------|
| `arr.add(n)` | `arr.addi(n)` | element + scalar |
| `arr.sub(n)` | `arr.subi(n)` | element - scalar |
| `arr.mul(n)` | `arr.muli(n)` | element × scalar |
| `arr.div(n)` | `arr.divi(n)` | element ÷ scalar |
| `arr.rsub(n)` | `arr.rsubi(n)` | scalar - element |
| `arr.rdiv(n)` | `arr.rdivi(n)` | scalar ÷ element |

### Element-wise (array-array) ops

| Copy | In-place | Description |
|------|----------|-------------|
| `a.add(b)` | `a.addi(b)` | element-wise addition |
| `a.sub(b)` | `a.subi(b)` | element-wise subtraction |
| `a.mul(b)` | `a.muli(b)` | element-wise (Hadamard) product |
| `a.div(b)` | `a.divi(b)` | element-wise division |
| `a.assign(b)` | *(always in-place)* | copy values from b into a |

### Transform ops (`Transforms.`)

| Method | Function |
|--------|----------|
| `Transforms.tanh(arr)` | tanh(x) |
| `Transforms.sigmoid(arr)` | 1 / (1 + e^−x) |
| `Transforms.relu(arr)` | max(0, x) |
| `Transforms.exp(arr)` | e^x |
| `Transforms.log(arr)` | ln(x) |
| `Transforms.sqrt(arr)` | √x |
| `Transforms.abs(arr)` | \|x\| |
| `Transforms.sin(arr)` | sin(x) |
| `Transforms.cos(arr)` | cos(x) |
| `Transforms.softmax(arr)` | softmax(x) |

### Reduction ops

| Method | Returns | Description |
|--------|---------|-------------|
| `arr.sumNumber()` | `Number` | sum of all elements |
| `arr.meanNumber()` | `Number` | mean of all elements |
| `arr.minNumber()` | `Number` | minimum value |
| `arr.maxNumber()` | `Number` | maximum value |
| `arr.norm1Number()` | `Number` | L1 norm |
| `arr.norm2Number()` | `Number` | L2 norm |
| `arr.stdNumber(b)` | `Number` | standard deviation |
| `arr.varNumber(b)` | `Number` | variance |
| `arr.sum(dim)` | `INDArray` | sum along dimension |
| `arr.mean(dim)` | `INDArray` | mean along dimension |
| `arr.max(dim)` | `INDArray` | max along dimension |
| `arr.min(dim)` | `INDArray` | min along dimension |
| `arr.norm1(dim)` | `INDArray` | L1 norm along dimension |
| `arr.norm2(dim)` | `INDArray` | L2 norm along dimension |

### Index accumulation ops

| Op class | Description |
|----------|-------------|
| `IMax` | index of maximum value (argmax) |
| `IMin` | index of minimum value (argmin) |
| `IAMax` | index of maximum absolute value |

### Comparison ops (return binary mask)

| Method | Condition |
|--------|-----------|
| `a.gt(b)` | a > b |
| `a.lt(b)` | a < b |
| `a.eq(b)` | a == b |
| `a.neq(b)` | a != b |
| `a.gte(b)` | a >= b |
| `a.lte(b)` | a <= b |

### Linear algebra

| Method | Description |
|--------|-------------|
| `a.mmul(b)` | matrix multiplication |
| `arr.transpose()` | transpose (returns a view) |
| `Nd4j.diag(v)` | create diagonal matrix from vector, or extract diagonal |
| `InvertMatrix.invert(m, false)` | matrix inverse (copy) |

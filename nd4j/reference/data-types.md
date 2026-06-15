---
title: "Data Types"
description: "The DataType enum, per-array typing, type casting, mixed precision, and migration from global data type"
---

# Data Types

One of the most significant changes in M2.1 is the move from a **global data type** to **per-array data types**. Every `INDArray` now carries its own `DataType`, and operations are type-aware. This page covers the full `DataType` enum, how to create typed arrays, how to cast between types, and how to take advantage of mixed precision training.

---

## The DataType Enum

All numeric types in ND4J are represented by the enum `org.nd4j.linalg.api.buffer.DataType`. Import it with:

```java
import org.nd4j.linalg.api.buffer.DataType;
```

### Floating-Point Types

Floating-point types are used for neural network weights, activations, and most numeric computation.

| Enum Constant | Bits | Description |
|---|---|---|
| `DOUBLE` | 64 | IEEE 754 double-precision float. Highest precision, largest memory footprint. |
| `FLOAT` | 32 | IEEE 754 single-precision float. Default type. Best balance of precision and performance on most hardware. |
| `FLOAT16` | 16 | Half-precision float. Reduced memory and bandwidth; supported on modern NVIDIA GPUs and some CPUs. Same constant as the deprecated `HALF`. |
| `BFLOAT16` | 16 | Brain float 16. Same 8-bit exponent range as `FLOAT`, but only 7 mantissa bits. Numerically safer than `FLOAT16` for training. Supported on TPUs, A100/H100 GPUs, and recent Intel CPUs. |

### Signed Integer Types

| Enum Constant | Bits | Notes |
|---|---|---|
| `INT64` | 64 | 64-bit signed integer. Same constant as the deprecated `LONG`. |
| `INT32` | 32 | 32-bit signed integer. Same constant as the deprecated `INT`. |
| `INT16` | 16 | 16-bit signed integer. Same constant as the deprecated `SHORT`. |
| `INT8` | 8 | 8-bit signed integer, range −128 to 127. Same constant as the deprecated `BYTE`. |

### Unsigned Integer Types

| Enum Constant | Bits |
|---|---|
| `UINT64` | 64 |
| `UINT32` | 32 |
| `UINT16` | 16 |
| `UINT8` | 8 — range 0 to 255. Same constant as the deprecated `UBYTE`. |

### Other Types

| Enum Constant | Description |
|---|---|
| `BOOL` | Boolean — stores `true`/`false` per element. Used by comparison ops that return masks. |
| `UTF8` | Variable-length UTF-8 string data. Used for label arrays and metadata. |
| `COMPRESSED` | Internal placeholder for compressed arrays. Not for direct use. |
| `UNKNOWN` | Sentinel value. Indicates an uninitialized or invalid type. |

### Deprecated Aliases

Several names from earlier releases are still present in the enum but are annotated `@Deprecated`. They map to the same underlying constant as their preferred replacement.

| Deprecated Name | Preferred Name |
|---|---|
| `HALF` | `FLOAT16` |
| `LONG` | `INT64` |
| `INT` | `INT32` |
| `SHORT` | `INT16` |
| `BYTE` | `INT8` |
| `UBYTE` | `UINT8` |

`FLOAT16` and `HALF` are literally the same enum constant — `FLOAT16` is a static field on the enum that points to `HALF`. The same relationship holds for each deprecated/preferred pair. Both names work identically at runtime, but new code should use the preferred names to keep intent clear.

---

## Memory Footprint

Every element of an `INDArray` occupies a fixed number of bytes determined by its `DataType`. Understanding this is essential for planning GPU memory budgets and comparing model footprints.

| DataType | Bytes per element |
|---|---|
| `DOUBLE` / `INT64` / `UINT64` | 8 |
| `FLOAT` / `INT32` / `UINT32` | 4 |
| `FLOAT16` / `BFLOAT16` / `INT16` / `UINT16` | 2 |
| `INT8` / `UINT8` / `BOOL` | 1 |

A 1000-element array uses 8 KB as `DOUBLE`, 4 KB as `FLOAT`, and 2 KB as `FLOAT16` or `BFLOAT16`. For a 100M-parameter model this difference is 800 MB vs 400 MB vs 200 MB — a decisive factor in whether a model fits in GPU memory.

---

## Migration from Beta4 and Earlier

In beta4 and all earlier releases, data type was a global, process-wide setting applied to all arrays:

```java
// OLD beta4 code — does not compile in M2.1
import org.nd4j.linalg.api.buffer.DataBuffer;

Nd4j.setDataType(DataBuffer.Type.DOUBLE);
// All subsequent Nd4j.zeros(), Nd4j.rand() etc. would produce DOUBLE arrays
```

In M2.1, `DataBuffer.Type` has been removed entirely. Each `INDArray` carries its own `DataType`, and creation methods accept it as an explicit argument. The global concept still exists as a **default** (see below), but it is no longer the only mechanism.

### What Changed

| Concept | Beta4 | M2.1 |
|---|---|---|
| Type class | `DataBuffer.Type` | `DataType` |
| Set global type | `Nd4j.setDataType(DataBuffer.Type.X)` | `Nd4j.setDefaultDataTypes(DataType.X, DataType.Y)` |
| Per-array type | Not supported | Every `INDArray` has a `DataType` |
| Type checking | N/A | `arr.dataType()` |
| Casting | N/A | `arr.castTo(DataType.X)` |

Replace every `DataBuffer.Type` reference in your codebase:

```java
// OLD
DataBuffer.Type.DOUBLE
DataBuffer.Type.FLOAT

// NEW
DataType.DOUBLE
DataType.FLOAT
```

---

## Default Data Type

The default type for newly created arrays is `FLOAT`. When you call a creation method without an explicit `DataType` argument, ND4J uses the current default.

You can change the default at application startup with `Nd4j.setDefaultDataTypes`. It accepts two arguments: the default floating-point type and the default integer type.

```java
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.factory.Nd4j;

// Set default float type to FLOAT, default integer type to INT32
Nd4j.setDefaultDataTypes(DataType.FLOAT, DataType.INT32);

// From this point on, creation methods without an explicit DataType
// will use FLOAT for floating-point arrays and INT32 for integer arrays
INDArray a = Nd4j.zeros(3, 4);    // DataType.FLOAT
INDArray b = Nd4j.ones(2, 5);     // DataType.FLOAT
```

Call `setDefaultDataTypes` once at startup, before any array creation. Changing it mid-run can produce inconsistent types that are hard to debug.

---

## Creating Typed Arrays

All `Nd4j` creation methods accept a `DataType` as their first argument. Pass the type explicitly whenever you need a specific type, regardless of what the default is.

### Zeros, Ones, and Value-Filled Arrays

```java
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

// All-zero arrays
INDArray dblZeros  = Nd4j.zeros(DataType.DOUBLE, 3, 4);   // shape [3,4], DOUBLE
INDArray fltZeros  = Nd4j.zeros(DataType.FLOAT,  3, 4);   // shape [3,4], FLOAT
INDArray intZeros  = Nd4j.zeros(DataType.INT32,  3, 4);   // shape [3,4], INT32

// All-one arrays
INDArray ones = Nd4j.ones(DataType.FLOAT, 2, 5);

// Explicit-value creation
INDArray mat  = Nd4j.create(DataType.FLOAT,  new long[]{3, 4});  // uninitialized, FLOAT
INDArray imat = Nd4j.create(DataType.INT32,  new long[]{3, 4});  // uninitialized, INT32
```

### Random Arrays

```java
// Uniform random in [0, 1) — only meaningful for floating-point types
INDArray uniformFloat  = Nd4j.rand(DataType.FLOAT,   3, 4);
INDArray uniformDouble = Nd4j.rand(DataType.DOUBLE,  3, 4);
INDArray uniformHalf   = Nd4j.rand(DataType.FLOAT16, 3, 4);

// Standard normal N(0,1)
INDArray gaussFloat    = Nd4j.randn(DataType.FLOAT,  3, 4);
INDArray gaussBf16     = Nd4j.randn(DataType.BFLOAT16, 3, 4);
```

### Linspace

```java
// 100 evenly-spaced points from 0.0 to 1.0, type DOUBLE
INDArray lin = Nd4j.linspace(DataType.DOUBLE, 0.0, 1.0, 100);
```

### From Java Arrays

Type inference uses the Java primitive type when creating from Java arrays. To force a specific type, create with an explicit `DataType` and then fill, or create and cast:

```java
// Inferred as FLOAT from float[]
INDArray fromFloat = Nd4j.createFromArray(new float[]{1f, 2f, 3f});

// Inferred as DOUBLE from double[]
INDArray fromDouble = Nd4j.createFromArray(new double[]{1.0, 2.0, 3.0});

// Inferred as INT32 from int[]
INDArray fromInt = Nd4j.createFromArray(new int[]{1, 2, 3});
```

---

## Checking the Type of an Array

Call `arr.dataType()` to retrieve the `DataType` of any `INDArray`:

```java
INDArray a = Nd4j.zeros(DataType.DOUBLE, 3, 4);
DataType t = a.dataType();            // DataType.DOUBLE
System.out.println(t);               // prints: DOUBLE
System.out.println(t == DataType.DOUBLE);  // true

// Use in a conditional
if (a.dataType() != DataType.FLOAT) {
    a = a.castTo(DataType.FLOAT);
}
```

---

## Casting Between Types

To convert an `INDArray` from one `DataType` to another, call `castTo`. This always returns a **new array** (a copy) in the target type. The original is not modified.

```java
INDArray original = Nd4j.rand(DataType.DOUBLE, 3, 4);
System.out.println(original.dataType());   // DOUBLE

// Cast to FLOAT — returns a new array, original unchanged
INDArray floatCopy = original.castTo(DataType.FLOAT);
System.out.println(floatCopy.dataType());  // FLOAT

// Cast to FLOAT16 for reduced memory
INDArray halfCopy = original.castTo(DataType.FLOAT16);
System.out.println(halfCopy.dataType());   // FLOAT16
```

### Precision Loss

Casting from a higher-precision type to a lower-precision type loses information:

```java
INDArray precise = Nd4j.scalar(DataType.DOUBLE, Math.PI);   // 3.141592653589793
INDArray half    = precise.castTo(DataType.FLOAT16);
System.out.println(half.getDouble(0));     // approximately 3.140625 — reduced precision
```

This is expected and intentional when using reduced precision. For training stability, keep high-precision master copies and cast only for the forward/backward pass.

### Integer Truncation

Casting a floating-point array to an integer type truncates (not rounds) the fractional part:

```java
INDArray floats = Nd4j.createFromArray(new float[]{1.9f, 2.1f, -0.7f});
INDArray ints   = floats.castTo(DataType.INT32);
// ints contains: [1, 2, 0]   (truncation, not rounding)
```

---

## Type Matching for Operations

In M2.1, the two operands of a binary operation must have the same `DataType`. If types differ, ND4J throws an exception:

```java
INDArray x = Nd4j.zeros(DataType.FLOAT,  3, 4);
INDArray y = Nd4j.zeros(DataType.DOUBLE, 3, 4);

// Throws: operands must have matching types
INDArray result = x.add(y);   // IllegalStateException
```

This is a deliberate design choice: silent implicit promotion (as in many scripting languages) hides bugs where the user accidentally mixes types and gets unexpected precision or memory use.

**Solution**: cast one operand before the operation:

```java
INDArray x = Nd4j.rand(DataType.FLOAT,  3, 4);
INDArray y = Nd4j.rand(DataType.DOUBLE, 3, 4);

// Cast x up to DOUBLE, then add
INDArray result = x.castTo(DataType.DOUBLE).add(y);    // result is DOUBLE

// Or cast y down to FLOAT, then add
INDArray result2 = x.add(y.castTo(DataType.FLOAT));    // result2 is FLOAT
```

Choose which direction to cast based on whether you need the precision of `DOUBLE` or the memory savings of `FLOAT`.

### Type-Safe Helper Pattern

When writing utility code that must accept any numeric type, check and cast defensively:

```java
public INDArray ensureFloat(INDArray arr) {
    if (arr.dataType() == DataType.FLOAT) {
        return arr;
    }
    return arr.castTo(DataType.FLOAT);
}
```

---

## Mixed Precision Training

Mixed precision training uses lower-precision types (typically `FLOAT16` or `BFLOAT16`) for most computation while keeping higher-precision master weights to accumulate gradients accurately. The result is significantly lower GPU memory use and higher throughput on hardware with native half-precision support (NVIDIA Volta and later, AMD RDNA2+).

### Strategy

1. **Master weights** are stored as `FLOAT` (32-bit).
2. Before each forward pass, cast the master weights to `FLOAT16` or `BFLOAT16`.
3. The forward and backward passes execute in half precision.
4. Gradients are cast back to `FLOAT` and accumulated into the master weights.
5. The optimizer step runs in `FLOAT`.

```java
// Master weights stored in full precision
INDArray masterWeights = Nd4j.rand(DataType.FLOAT, 512, 512);

// Half-precision copy for forward/backward pass
INDArray halfWeights = masterWeights.castTo(DataType.FLOAT16);

// ... forward and backward pass using halfWeights ...

// Assume gradientsHalf is the gradient computed in FLOAT16
INDArray gradientsHalf = computeGradients(halfWeights);

// Cast gradients back to FLOAT for the optimizer update
INDArray gradientsFloat = gradientsHalf.castTo(DataType.FLOAT);

// Optimizer step on master weights
masterWeights.subi(gradientsFloat.muli(learningRate));
```

### FLOAT16 vs BFLOAT16

Both use 2 bytes per element, but they allocate those bits differently:

| Type | Exponent bits | Mantissa bits | Max value |
|---|---|---|---|
| `FLOAT16` | 5 | 10 | ~65,504 |
| `BFLOAT16` | 8 | 7 | ~3.4 × 10^38 (same as FLOAT) |

`FLOAT16` offers higher precision for values in its range, but overflows easily — activations or gradients that exceed ~65,504 become `Inf`. This requires **loss scaling** (multiplying the loss by a large constant before the backward pass) to keep gradients in range.

`BFLOAT16` has the same exponent range as `FLOAT`, so it does not overflow under conditions that `FLOAT` would not overflow. Gradients stay finite without loss scaling in most cases. The trade-off is lower mantissa precision (~2 decimal digits vs ~3 for `FLOAT16`).

**Recommendation**: prefer `BFLOAT16` when your hardware supports it. It requires less engineering overhead to use safely.

### Setting a Default for Half-Precision Training

```java
// Use BFLOAT16 as the default for all new arrays
Nd4j.setDefaultDataTypes(DataType.BFLOAT16, DataType.INT32);

// Create compute arrays without specifying type explicitly
INDArray input = Nd4j.rand(3, 512);       // DataType.BFLOAT16
INDArray kernel = Nd4j.rand(512, 256);    // DataType.BFLOAT16
```

---

## Reference: DataType Summary

| DataType | Bytes | Floating-point? | Notes |
|---|---|---|---|
| `DOUBLE` | 8 | Yes | 64-bit IEEE 754. Use for scientific computing where precision matters. |
| `FLOAT` | 4 | Yes | 32-bit IEEE 754. Default. Good choice for most training workloads. |
| `FLOAT16` | 2 | Yes | 16-bit half. GPU-accelerated mixed precision. Requires loss scaling. `HALF` is deprecated alias. |
| `BFLOAT16` | 2 | Yes | Brain float. Same exponent range as FLOAT. Preferred for training without loss scaling. |
| `INT64` | 8 | No | 64-bit signed. `LONG` is deprecated alias. |
| `INT32` | 4 | No | 32-bit signed. `INT` is deprecated alias. |
| `INT16` | 2 | No | 16-bit signed. `SHORT` is deprecated alias. |
| `INT8` | 1 | No | 8-bit signed, −128 to 127. `BYTE` is deprecated alias. |
| `UINT64` | 8 | No | 64-bit unsigned. |
| `UINT32` | 4 | No | 32-bit unsigned. |
| `UINT16` | 2 | No | 16-bit unsigned. |
| `UINT8` | 1 | No | 8-bit unsigned, 0 to 255. `UBYTE` is deprecated alias. |
| `BOOL` | 1 | No | Boolean. Returned by comparison ops. |
| `UTF8` | variable | No | String data. Used for label arrays. |

---

## Key API Signatures

```java
// Set process-wide defaults (call once at startup)
Nd4j.setDefaultDataTypes(DataType floatType, DataType intType);

// Check default
DataType defaultFP  = Nd4j.defaultFloatingPointType();   // default: FLOAT

// Get the type of an existing array
DataType t = arr.dataType();

// Cast to a new type (returns copy, never in-place)
INDArray cast = arr.castTo(DataType.FLOAT);

// Creation methods accepting DataType
Nd4j.zeros(DataType, long... shape);
Nd4j.ones(DataType, long... shape);
Nd4j.create(DataType, long... shape);
Nd4j.rand(DataType, long... shape);
Nd4j.randn(DataType, long... shape);
Nd4j.linspace(DataType, double from, double to, long length);
```

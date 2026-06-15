---
title: "NumPy Bridge"
description: "Zero-copy data exchange between INDArray and numpy.ndarray via python4j-numpy"
---

# NumPy Bridge

The `python4j-numpy` module adds a `PythonType` implementation called `NumpyArray` that lets you pass ND4J `INDArray` objects into Python as `numpy.ndarray` views, and convert Python numpy arrays back to `INDArray` objects — in most cases without copying the underlying data buffer.

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>python4j-numpy</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

Adding this module to the classpath is sufficient. `NumpyArray` registers itself via Java's `ServiceLoader` mechanism, so it becomes available to `PythonTypes.get()` automatically. No additional configuration is required to enable the conversion.

---

## How Zero-Copy Works

An ND4J `INDArray` on the CPU backend stores its data in a `DataBuffer` that wraps a native (off-heap) memory block. A `numpy.ndarray` is also backed by a contiguous block of native memory. When the two have compatible data types, the `NumpyArray` converter creates the NumPy array using `np.frombuffer` (via ctypes) pointing at the same address, avoiding a copy.

On the Java-to-Python direction, `NumpyArray.toPython(INDArray)` obtains the buffer's native address and creates a NumPy array view pointing to it. The `DataBuffer` reference is retained in a static cache keyed by `(address, length, dtype)` to prevent premature garbage collection.

On the Python-to-Java direction, `NumpyArray.toJava(PythonObject)` reads the `PyArrayObject` struct to extract shape, strides, and data type, then wraps the numpy array's data pointer in an ND4J `DataBuffer`. The resulting `INDArray` shares the same memory as the Python array.

**CUDA note.** Before passing an `INDArray` backed by GPU memory to Python, the converter calls `Nd4j.getAffinityManager().ensureLocation(array, HOST)` to synchronize the array to host memory. The Python side always sees a CPU buffer; GPU synchronization back from the Python side is not automatic.

**BFLOAT16 note.** NumPy has no native bfloat16 dtype. If you pass a `BFLOAT16` `INDArray` through the bridge, a cast to `FLOAT32` is made automatically and a warning is logged. The returned `INDArray` will be `FLOAT32`.

---

## Data Type Mapping

| ND4J `DataType` | NumPy dtype |
|-----------------|-------------|
| `DOUBLE` | `float64` |
| `FLOAT` | `float32` |
| `BFLOAT16` | `float32` (cast) |
| `HALF` (float16) | `float16` |
| `INT32` | `int32` |
| `INT64` (LONG) | `int64` |
| `INT16` (SHORT) | `int16` |
| `INT8` (BYTE) | `int8` |
| `UINT8` (UBYTE) | `uint8` |
| `UINT16` | `uint16` |
| `UINT32` | `uint32` |
| `UINT64` | `uint64` |
| `BOOL` | `bool` |

---

## Passing an INDArray to Python

```java
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.python4j.*;
import org.nd4j.python4j.numpy.NumpyArray;
import java.util.Arrays;
import java.util.List;

// Create a 3x4 float32 array
INDArray javaArray = Nd4j.rand(3, 4);

try (PythonGIL gil = PythonGIL.lock()) {
    List<PythonVariable> inputs = Arrays.asList(
        new PythonVariable<>("arr", NumpyArray.INSTANCE, javaArray)
    );

    List<PythonVariable> outputs = Arrays.asList(
        new PythonVariable<>("result", NumpyArray.INSTANCE)
    );

    String code =
        "import numpy as np\n" +
        "result = arr * 2.0";

    PythonExecutioner.exec(code, inputs, outputs);

    INDArray result = (INDArray) outputs.get(0).getValue();
    System.out.println(result);
}
```

The `PythonVariable` constructor accepts a `PythonType<T>` as its second argument. Passing `NumpyArray.INSTANCE` tells Python4J to use the NumPy bridge for that variable.

---

## Reading a numpy.ndarray Back from Python

When an output variable is declared with type `NumpyArray.INSTANCE`, Python4J calls `NumpyArray.toJava()` after execution. The returned `INDArray` wraps the same data buffer as the Python array, so further Python-side modifications to that array will be reflected in the Java `INDArray` until one of the two is garbage-collected or reallocated.

```java
try (PythonGIL gil = PythonGIL.lock()) {
    PythonExecutioner.exec(
        "import numpy as np\n" +
        "matrix = np.eye(4, dtype=np.float32)"
    );

    PythonVariable<INDArray> out =
        PythonExecutioner.getVariable("matrix", NumpyArray.INSTANCE);

    INDArray identity = out.getValue();
    System.out.println(identity.shapeInfoToString()); // [4, 4]
}
```

---

## Memory Considerations

Because the bridge shares native memory rather than copying it, lifetime management is important:

1. **Keep the source alive.** If you pass an `INDArray` to Python and the Java reference goes out of scope before Python is done with it, the garbage collector may reclaim the `DataBuffer` while the numpy array still points to that memory. Retain a Java reference for the entire duration of the Python execution block.

2. **Do not rely on the Python array after the INDArray is deallocated.** Conversely, if you return a numpy array from Python, the wrapped `INDArray` will hold the numpy array's data alive via the cache. Calling `Py_DecRef` on the underlying `PyObject` while the `INDArray` still exists will cause a use-after-free.

3. **PythonGC can interfere.** If you use `PythonGC.watch()` blocks (see [Advanced Usage](advanced.md)), be careful that `PythonObject` wrappers for numpy arrays are not collected before you extract the `INDArray` from them.

4. **Workspace scoping.** `NumpyArray.toJava()` creates the wrapping `DataBuffer` outside any active memory workspace (`Nd4j.getMemoryManager().scopeOutOfWorkspaces()`). This prevents the buffer from being unexpectedly freed when a workspace closes.

---

## Checking Whether the Bridge Is Active

The `NumpyArray` type is initialized once when the class is first loaded. If numpy is not importable on the Python path, the init will fail with a `PythonException`. You can verify the Python path includes numpy by running:

```java
try (PythonGIL gil = PythonGIL.lock()) {
    PythonExecutioner.exec("import numpy as np; print(np.__version__)");
}
```

To disable automatic numpy import (for example, when running in an environment where numpy is absent but `python4j-numpy` is on the classpath), set the system property before any Python4J class loads:

```
-Dorg.eclipse.python4j.numpyimport=false
```

---

## Next Steps

- [Advanced Usage](advanced.md) — GIL management across threads, context isolation, `PythonGC`, and subprocess utilities.

---
title: "Serialization"
description: "Saving and loading INDArrays in binary, text, NumPy, and ByteBuffer formats"
---

## Serialization

ND4J provides several serialization formats for `INDArray` instances. Choosing the right one depends on your priorities: raw throughput, human-readable output, or interoperability with Python/NumPy pipelines.

| Format | Write | Read | Best for |
|--------|-------|------|----------|
| Binary (`Nd4j`) | `Nd4j.write` | `Nd4j.read` | Fast file persistence |
| ByteBuffer (`BinarySerde`) | `BinarySerde.toByteBuffer` | `BinarySerde.toArray` | In-memory / IPC transfer |
| Text | `Nd4j.writeTxt` | `Nd4j.readTxt` | Debugging, inspection |
| NumPy text (CSV) | — | `Nd4j.readNumpy` | Import CSV/NumPy text |
| NumPy binary (`.npy`) | — | `Nd4j.createFromNpyFile` | Python interoperability |

---

## 1. Binary Format

The binary format is the most compact and fastest option for disk persistence. It serializes the full array — shape, data type, order, and element values — into a compact byte stream using Java's `DataOutputStream` / `DataInputStream`.

**Write:**

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

import java.io.*;

INDArray arr = Nd4j.linspace(1, 10, 10).reshape(2, 5);

try (DataOutputStream out = new DataOutputStream(
        new BufferedOutputStream(new FileOutputStream("array.bin")))) {
    Nd4j.write(arr, out);
}
```

**Read:**

```java
INDArray loaded;

try (DataInputStream in = new DataInputStream(
        new BufferedInputStream(new FileInputStream("array.bin")))) {
    loaded = Nd4j.read(in);
}
```

The deserialized array is identical to the original: same shape, same data type, same element ordering. Wrapping the underlying stream in a `BufferedOutputStream` / `BufferedInputStream` is recommended for large arrays to avoid excessive system-call overhead.

**When to use:** File checkpoints between JVM sessions, persisting intermediate computation results, or anywhere you need the smallest on-disk footprint with the fastest read-back time.

---

## 2. ByteBuffer Format

`BinarySerde` serializes an `INDArray` into a `java.nio.ByteBuffer`. When the array's underlying data is already stored in a direct (off-heap) buffer, this conversion can avoid a data copy entirely, making it well-suited for inter-process communication, shared-memory transfers, or handing data off to a native library.

**Maven dependency** — `BinarySerde` lives in the `nd4j-serde` module:

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-serde</artifactId>
    <version>${nd4j.version}</version>
</dependency>
```

**Write to ByteBuffer:**

```java
import org.nd4j.serde.binary.BinarySerde;

import java.nio.ByteBuffer;

INDArray arr = Nd4j.rand(3, 4);

// Serialize to a ByteBuffer (direct buffer, zero-copy when possible)
ByteBuffer buffer = BinarySerde.toByteBuffer(arr);
```

**Read from ByteBuffer:**

```java
INDArray recovered = BinarySerde.toArray(buffer);
```

**Writing / reading via a file channel** (useful for memory-mapped files):

```java
import java.nio.channels.FileChannel;
import java.nio.file.*;

Path path = Paths.get("array.buf");

// Write
try (FileChannel ch = FileChannel.open(path,
        StandardOpenOption.CREATE, StandardOpenOption.WRITE)) {
    BinarySerde.writeArrayToChannel(arr, ch);
}

// Read
try (FileChannel ch = FileChannel.open(path, StandardOpenOption.READ)) {
    INDArray loaded = BinarySerde.readFromChannel(ch);
}
```

**When to use:** In-memory handoffs between components running in the same JVM, native-bridge scenarios, or when using memory-mapped files for very large arrays. The zero-copy property only applies when the array's data buffer is already in a compatible direct-memory layout; otherwise a copy is made transparently.

---

## 3. Text Format

The text format writes each element as a human-readable floating-point number. Rows are delimited by newlines and columns by commas (the exact delimiter is locale-independent).

**Write:**

```java
Nd4j.writeTxt(arr, "array.txt");
```

**Read:**

```java
INDArray loaded = Nd4j.readTxt("array.txt");
```

**Example output** for a 2x3 array:

```
1.000000,2.000000,3.000000
4.000000,5.000000,6.000000
```

Text serialization round-trips correctly for the default float precision, but be aware that the conversion to decimal representation and back introduces a tiny floating-point rounding error. For double-precision arrays where bit-exact round-tripping matters, prefer the binary format.

**When to use:** Spot-checking intermediate values during development, sharing arrays with non-Java tooling that can read CSV, or generating test fixtures that should be human-auditable.

---

## 4. NumPy Text (CSV) Format

`Nd4j.readNumpy` reads a plain-text file whose rows are whitespace- or comma-separated numbers — the same format produced by NumPy's `numpy.savetxt` and by many CSV export tools.

```python
# Python side — generate the file
import numpy as np
a = np.array([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])
np.savetxt("array.csv", a, delimiter=",")
```

```java
// Java side — read it back
// Second argument is the delimiter used in the file
INDArray arr = Nd4j.readNumpy("array.csv", ",");
```

The result is always a 2D row-major `INDArray`. The delimiter must match exactly what was used during export. Common choices are `","` for CSV and `" "` for space-separated files.

**Note:** `Nd4j.writeNumpy` (which wrote a compatible text file) was deprecated in earlier releases. For writing, use `Nd4j.writeTxt` or the NumPy binary format described below.

**When to use:** Importing datasets or weights that were exported from a Python/NumPy workflow using `numpy.savetxt` or any CSV-producing tool.

---

## 5. NumPy Binary (`.npy`) Format

The `.npy` binary format is the standard serialization format for single NumPy arrays. It stores the array's dtype, shape, byte order, and raw element data in a compact binary file that NumPy can read and write natively.

**Python side — save a `.npy` file:**

```python
import numpy as np
arr = np.array([[1.0, 2.0], [3.0, 4.0]], dtype=np.float32)
np.save("array.npy", arr)
```

**Java side — load the `.npy` file:**

```java
import java.io.File;

INDArray arr = Nd4j.createFromNpyFile(new File("array.npy"));
```

ND4J reads the header to determine the shape and dtype automatically — no manual configuration is needed. The returned array preserves the original dtype (e.g., `float32` maps to `DataType.FLOAT`, `float64` maps to `DataType.DOUBLE`).

**Saving from Java for NumPy to read:**

```java
// Write a .npy file that Python can load with numpy.load()
Nd4j.writeAsNumpy(arr, new File("output.npy"));
```

```python
# Python side — verify
import numpy as np
a = np.load("output.npy")
print(a.shape, a.dtype)
```

**When to use:** Any time you need to exchange arrays between a Java/DL4J pipeline and a Python/NumPy/PyTorch/TensorFlow pipeline. The `.npy` format is the most reliable interoperability path because it preserves dtype, shape, and byte order without ambiguity.

---

## 6. Additional Serialization Back-ends (nd4j-serde)

The `nd4j-serde` module provides adapters for other serialization ecosystems:

| Sub-module | Description |
|------------|-------------|
| `nd4j-serde/nd4j-jackson` | Jackson `JsonSerializer` / `JsonDeserializer` for `INDArray` — embeds arrays as Base64-encoded binary inside JSON documents |
| `nd4j-serde/nd4j-kryo` | Kryo serializer for `INDArray` — useful in Apache Spark or other distributed frameworks that use Kryo as their wire format |
| `nd4j-serde/nd4j-aeron` | Aeron-compatible serialization for low-latency messaging |
| `nd4j-serde/nd4j-camel-routes` | Apache Camel data-format adapter |

The Jackson integration is particularly common when embedding arrays in REST API payloads:

```java
import org.nd4j.serde.jackson.ndarray.NDArrayDeSerializer;
import org.nd4j.serde.jackson.ndarray.NDArraySerializer;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.module.SimpleModule;

ObjectMapper mapper = new ObjectMapper();
SimpleModule module = new SimpleModule();
module.addSerializer(INDArray.class, new NDArraySerializer());
module.addDeserializer(INDArray.class, new NDArrayDeSerializer());
mapper.registerModule(module);

// Serialize to JSON string
String json = mapper.writeValueAsString(arr);

// Deserialize back
INDArray loaded = mapper.readValue(json, INDArray.class);
```

---

## Choosing a Format

**Use binary (`Nd4j.write` / `Nd4j.read`) when:**
- You need maximum speed and minimum file size for JVM-to-JVM persistence.
- You are checkpointing intermediate results in a training loop.
- Portability outside the JVM is not required.

**Use `BinarySerde` when:**
- You are passing arrays between threads or processes within the same machine.
- You want zero-copy semantics or need to work with memory-mapped files.
- You are integrating with a native library through a `ByteBuffer` API.

**Use text (`Nd4j.writeTxt` / `Nd4j.readTxt`) when:**
- You want to visually inspect values in a text editor or log file.
- You are writing test fixtures that need to be human-readable and diffable.
- Absolute bit-exact round-tripping is not required.

**Use NumPy text (`Nd4j.readNumpy`) when:**
- You are importing data that was exported by `numpy.savetxt` or another CSV tool.
- The producing system cannot write `.npy` binary files.

**Use NumPy binary (`Nd4j.createFromNpyFile` / `Nd4j.writeAsNumpy`) when:**
- You are integrating with a Python-based ecosystem (NumPy, pandas, PyTorch, TensorFlow).
- You need dtype and shape metadata to be preserved automatically.
- You want the most portable and widely supported binary array format available.

---

## Complete Example

The following snippet demonstrates all formats in a single class:

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.serde.binary.BinarySerde;

import java.io.*;
import java.nio.ByteBuffer;

public class SerializationDemo {

    public static void main(String[] args) throws Exception {

        INDArray original = Nd4j.linspace(1, 12, 12).reshape(3, 4);
        System.out.println("Original:\n" + original);

        // ---- 1. Binary (DataStream) ----
        try (DataOutputStream dos = new DataOutputStream(
                new FileOutputStream("array.bin"))) {
            Nd4j.write(original, dos);
        }
        INDArray fromBin;
        try (DataInputStream dis = new DataInputStream(
                new FileInputStream("array.bin"))) {
            fromBin = Nd4j.read(dis);
        }
        System.out.println("From binary file:\n" + fromBin);

        // ---- 2. ByteBuffer ----
        ByteBuffer buf = BinarySerde.toByteBuffer(original);
        INDArray fromBuf = BinarySerde.toArray(buf);
        System.out.println("From ByteBuffer:\n" + fromBuf);

        // ---- 3. Text ----
        Nd4j.writeTxt(original, "array.txt");
        INDArray fromTxt = Nd4j.readTxt("array.txt");
        System.out.println("From text file:\n" + fromTxt);

        // ---- 4. NumPy text (read only; file produced externally) ----
        // INDArray fromCsv = Nd4j.readNumpy("array.csv", ",");

        // ---- 5. NumPy binary ----
        Nd4j.writeAsNumpy(original, new File("array.npy"));
        INDArray fromNpy = Nd4j.createFromNpyFile(new File("array.npy"));
        System.out.println("From .npy file:\n" + fromNpy);
    }
}
```

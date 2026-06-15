---
title: "Python4J Overview"
description: "Embedding CPython in JVM applications — what Python4J is, use cases, and architecture"
---

# Python4J Overview

Python4J embeds CPython 3.x directly inside a running JVM process. It uses JavaCPP's CPython bindings to link against the native Python interpreter, which means your Java application can execute Python code, import any installed Python library, and exchange data with the Python runtime — all in the same process, without spawning a subprocess.

---

## What Python4J Provides

The core capability is executing arbitrary Python source code from Java and sharing variables bidirectionally. From the Java side you hand a string of Python code to `PythonExecutioner`, set input variables, and read output variables back after execution. The Python interpreter runs inside the JVM's address space via JavaCPP's `cpython` preset, so native extension modules (NumPy, SciPy, scikit-learn, PyTorch, and so on) load exactly as they do in a standalone Python process.

The library consists of two Maven modules:

- `python4j-core` — the interpreter bootstrap, GIL management, variable I/O, and core type converters.
- `python4j-numpy` — an optional extension that adds a `NumpyArray` type converter, enabling zero-copy exchange of `INDArray` and `numpy.ndarray` objects.

---

## Use Cases

**Calling Python ML libraries from Java.** If your production stack is JVM-based but your model or preprocessing logic is written in Python (scikit-learn pipelines, custom PyTorch inference, Hugging Face tokenizers), Python4J lets you invoke that code without a network boundary or inter-process serialization.

**NumPy interoperability.** Through the `python4j-numpy` module, ND4J `INDArray` objects can be passed into Python as `numpy.ndarray` views of the same memory. Results come back as `INDArray` objects sharing the same buffer, so large arrays do not get copied.

**Running Python scripts as part of a Java workflow.** Data preparation, feature extraction, or visualization steps written in `.py` files can be executed directly from Java using `PythonExecutioner.exec()` or via the subprocess utilities in `PythonProcess`.

**Embedding Python in a Java server.** REST services, Spark executors, or long-running daemon processes can initialize the Python interpreter once at startup and then call Python routines on each request, avoiding per-call interpreter startup cost.

---

## Architecture

### PythonExecutioner

`PythonExecutioner` is the central class. It handles:

1. **Initialization** (`PythonExecutioner.init()`). Called once, lazily, when the class is first loaded. It invokes `Py_InitializeEx(0)` to start the CPython runtime, sets up the Python path (including JavaCPP's bundled packages), initializes all registered `PythonType` instances, establishes the main thread state for GIL management, and releases the GIL so other threads can acquire it.

2. **Code execution** (`PythonExecutioner.exec(String code)`). Wraps the user's code in a try/except harness that captures Python exceptions into a known variable, calls `PyRun_SimpleStringFlags`, and then checks whether the exception variable was set. If it was, a `PythonException` is thrown on the Java side.

3. **Variable I/O** (`setVariables`, `getVariables`, `getVariable`). Variables are transferred by writing to and reading from the `__main__` module's global dictionary. Each variable has a name and a `PythonType` that governs serialization to and from CPython objects.

### Python Path Control

The interpreter path is controlled by the system property `org.eclipse.python4j.path`. A second property, `org.eclipse.python4j.path.append`, governs how this custom path interacts with JavaCPP's bundled packages. The three possible values are:

| Value | Effect |
|-------|--------|
| `before` (default) | JavaCPP packages are prepended before the custom path |
| `after` | Custom path is prepended; JavaCPP packages follow |
| `none` | Only the custom path is used; JavaCPP packages are excluded |

This matters when you need to use a specific version of NumPy or another library that differs from the one bundled with the JavaCPP preset.

### GIL Management

CPython's Global Interpreter Lock (GIL) serializes access to the interpreter. Python4J tracks GIL ownership through the `PythonGIL` class. The main thread releases the GIL after initialization so worker threads can acquire it. Any thread that calls `PythonExecutioner` methods must hold the GIL. Violations are caught by `PythonGIL.assertThreadSafe()`, which throws an `IllegalStateException` with a clear message directing you to use `try(PythonGIL gil = PythonGIL.lock()) { ... }`.

### Type System

Python4J's type system is extensible. The base types defined in `PythonTypes` cover:

| Python type | Java type |
|-------------|-----------|
| `str` | `String` |
| `int` | `Long` |
| `float` | `Double` |
| `bool` | `Boolean` |
| `bytes` | `byte[]` |
| `list` | `List` |
| `dict` | `Map` |

Additional types (such as `numpy.ndarray`) register themselves via Java's `ServiceLoader` mechanism. The `NumpyArray` type in `python4j-numpy` is the canonical example.

### Context Management

`PythonContextManager` provides named interpreter contexts. Each context is a named namespace: switching contexts collapses the current global namespace into prefixed keys and expands the target context's keys. This allows different parts of an application to maintain isolated Python namespaces without restarting the interpreter.

---

## Limitations

- **Single interpreter per JVM.** CPython can only be initialized once per process. You cannot run two independent Python environments in the same JVM.
- **GIL serializes Python execution.** Concurrent Java threads that call into Python will queue behind the GIL; Python code does not run in parallel threads even on a multi-core machine (unless using Python's `multiprocessing` or native extension code that releases the GIL internally).
- **BFLOAT16 is not supported by NumPy.** When passing a BFLOAT16 `INDArray` through the NumPy bridge, a cast to FLOAT32 is made automatically. A warning is logged.
- **Initialization must precede GIL acquisition.** If your application code acquires the GIL before `PythonExecutioner` initializes, the `NumpyArray` type will fail to initialize (`PythonException: Can not initialize numpy - GIL already acquired`).
- **No re-initialization.** Once `Py_InitializeEx` has been called, properties like `org.eclipse.python4j.path` have no further effect. Set them before any Python4J class is loaded.

---

## Maven Dependency

Add the following to your `pom.xml`. Replace `${dl4j.version}` with the current release version.

```xml
<!-- Core Python4J (required) -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>python4j-core</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- NumPy bridge (optional, adds INDArray <-> numpy.ndarray conversion) -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>python4j-numpy</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

The `python4j-core` artifact depends on `org.bytedeco:cpython` (the JavaCPP CPython preset). That preset bundles a platform-specific CPython 3.x shared library. No separate Python installation is required unless you need packages beyond what the preset supplies — in which case, point `org.eclipse.python4j.path` at a local Python environment.

---

## Next Steps

- [Getting Started](getting-started.md) — execute Python code and exchange variables from Java.
- [NumPy Bridge](numpy-bridge.md) — zero-copy `INDArray` / `numpy.ndarray` exchange.
- [Advanced Usage](advanced.md) — GIL management, context isolation, subprocess mode, and error handling.

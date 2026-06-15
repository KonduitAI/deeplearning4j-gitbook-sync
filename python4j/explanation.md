---
title: "Python4J Advanced Usage"
description: "Advanced Python4J features — GIL management, context managers, garbage collection, and subprocess mode"
---

# Python4J Advanced Usage

This page covers the lower-level mechanisms in Python4J that matter when you integrate the interpreter into a multi-threaded application, need strict variable namespace isolation, must manage native memory carefully, or want to run Python in a separate OS process.

---

## GIL Management

### Why the GIL Matters

CPython's Global Interpreter Lock (GIL) ensures that only one OS thread executes Python bytecode at a time. Python4J exposes this constraint explicitly so that threading errors are caught at the Java boundary with a clear message rather than producing a JVM crash or silent data corruption.

### PythonGIL.lock()

`PythonGIL` is an `AutoCloseable` that acquires the GIL on construction and releases it when the try-with-resources block exits.

```java
try (PythonGIL gil = PythonGIL.lock()) {
    PythonExecutioner.exec("x = 42");
}
```

Under the hood, `lock()` calls `PyGILState_Ensure()` on non-main threads, and `PyEval_RestoreThread()` on the main thread (which released the GIL after initialization). Release calls the corresponding `PyGILState_Release()` or `PyEval_SaveThread()`.

### Multi-threaded Usage

Each Java thread that calls into Python must acquire the GIL first. Because GIL access is serialized, no Python code from different threads runs in parallel. You can, however, have multiple Java threads that take turns holding the GIL:

```java
Runnable task = () -> {
    try (PythonGIL gil = PythonGIL.lock()) {
        PythonExecutioner.exec("import time; time.sleep(0.1)");
    }
};

Thread t1 = new Thread(task);
Thread t2 = new Thread(task);
t1.start();
t2.start();
t1.join();
t2.join();
```

Both threads will succeed; they run their Python code sequentially, not in parallel.

### assertThreadSafe()

`PythonGIL.assertThreadSafe()` is called internally by every `PythonExecutioner` method. If no GIL is held and the caller is not the default single-threaded thread, it throws:

```
RuntimeException: Attempt to use Python4j from multiple threads without acquiring GIL.
Enclose your code in a try(PythonGIL gil = PythonGIL.lock()){...} block.
```

This is intentional. Do not suppress or work around the check.

### Embedded Python (Disabling Automatic GIL Release)

When Python4J is used inside an environment that already manages the GIL (for example, a Jython host or a native Python extension that calls back into Java), automatic GIL release after initialization may conflict with the host's state management. Disable it with:

```
-Dorg.eclipse.python4j.release_gil_automatically=false
```

or at runtime before initialization:

```java
PythonConstants.setReleaseGilAutomatically(false);
```

---

## PythonContextManager

### The Problem Context Isolation Solves

All Python code executed by `PythonExecutioner` shares a single `__main__` namespace. If two parts of your application define a variable named `model`, they will overwrite each other.

`PythonContextManager` provides named namespaces that isolate Python globals. Switching contexts collapses the current globals under prefixed keys and expands the target context's keys into the main namespace.

### Creating and Switching Contexts

```java
try (PythonGIL gil = PythonGIL.lock()) {
    // Work in a named context
    PythonContextManager.setContext("inference_worker");
    PythonExecutioner.exec("model_name = 'resnet'");

    // Switch to another context
    PythonContextManager.setContext("preprocessing_worker");
    PythonExecutioner.exec("model_name = 'tokenizer'");

    // Variables are isolated
    PythonContextManager.setContext("inference_worker");
    System.out.println(
        PythonExecutioner.getVariable("model_name", PythonTypes.STR).getValue()
    ); // resnet
}
```

### PythonContextManager.Context (AutoCloseable)

The inner class `PythonContextManager.Context` provides RAII-style context switching:

```java
try (PythonGIL gil = PythonGIL.lock()) {
    try (PythonContextManager.Context ctx = new PythonContextManager.Context("my_context")) {
        PythonExecutioner.exec("x = 100");
    } // automatically reverts to the previous context
}
```

Using a no-argument constructor creates a temporary context with a random UUID name that is automatically deleted when closed:

```java
try (PythonContextManager.Context ctx = new PythonContextManager.Context()) {
    PythonExecutioner.exec("temp_var = 'ephemeral'");
} // context deleted; temp_var is gone
```

### Resetting and Deleting Contexts

```java
// Reset the current context (clear all its variables)
PythonContextManager.reset();

// Delete a specific non-current context
PythonContextManager.deleteContext("old_context");

// Delete all contexts except "main"
PythonContextManager.deleteNonMainContexts();
```

---

## PythonGC — Python Object Garbage Collection

### Why Manual GC Exists

`PythonObject` wraps a `PyObject*` from CPython. Reference counting in CPython is manual: you must call `Py_DecRef` for every borrowed reference you are done with. Python4J introduces `PythonGC` as a frame-based collector to simplify this.

### PythonGC.watch()

Open a `PythonGC` frame with `watch()`. Every `PythonObject` created inside the frame is registered for collection when the frame closes.

```java
try (PythonGC gc = PythonGC.watch()) {
    PythonObject result = Python.eval("1 + 1");
    // use result...
} // Py_DecRef called on result automatically
```

### PythonGC.keep()

If you want to retain a `PythonObject` beyond the current frame, call `keep()` to promote it to the enclosing frame:

```java
PythonObject retained;
try (PythonGC gc = PythonGC.watch()) {
    PythonObject temp = Python.eval("compute_something()");
    PythonGC.keep(temp);
    retained = temp;
} // temp is NOT collected; it was promoted
// use retained...
```

### PythonGC.pause() and resume()

Use `pause()` to temporarily disable collection inside a nested scope when you know objects will outlive the inner frame:

```java
try (PythonGC outer = PythonGC.watch()) {
    try (PythonGC paused = PythonGC.pause()) {
        // PythonObjects created here are not tracked
    }
}
```

---

## Subprocess Mode — PythonProcess

`PythonProcess` provides utilities for running Python in a separate OS process using the CPython executable bundled by JavaCPP. Unlike `PythonExecutioner`, subprocess mode does not share memory with the JVM — it is suitable for heavy isolation or for cases where the in-process interpreter is not appropriate.

### Running a Command and Collecting Output

```java
String output = PythonProcess.runAndReturn("-c", "print('hello from subprocess')");
System.out.println(output); // hello from subprocess
```

`runAndReturn` starts the bundled Python interpreter with the given arguments, waits for completion, and returns stdout as a `String`. Stderr is not captured.

### Running a Script File

```java
PythonProcess.run("/path/to/myscript.py");
```

`run` inherits the JVM's stdin/stdout/stderr (`ProcessBuilder.inheritIO()`), so script output goes directly to the console.

### Package Management

`PythonProcess` exposes pip operations against the bundled interpreter:

```java
// Install a package
PythonProcess.pipInstall("requests");

// Install a specific version
PythonProcess.pipInstall("scikit-learn", "1.2.0");

// Install from a requirements file
PythonProcess.pipInstallFromRequirementsTxt("/path/to/requirements.txt");

// Check if a package is installed
boolean installed = PythonProcess.isPackageInstalled("numpy");

// Get a package's version string
String version = PythonProcess.getPackageVersion("numpy");

// Uninstall
PythonProcess.pipUninstall("requests");
```

These methods throw `PythonException` if the pip command fails.

---

## Error Handling

`PythonException` is an unchecked exception. It is thrown in these situations:

- `exec()` detects that the Python-side exception variable was set after execution.
- `simpleExec()` gets a non-zero return code (typically a syntax error).
- A type conversion fails (e.g., trying to get an `int` variable that holds a `str`).
- The GIL is not held when a `PythonExecutioner` method is called from a non-default thread.
- `PythonGC` or `NumpyArray` initialization fails.

```java
try (PythonGIL gil = PythonGIL.lock()) {
    PythonExecutioner.exec("1 / 0");
} catch (PythonException e) {
    System.err.println("Caught: " + e.getMessage());
    // e.getMessage() contains the Python ZeroDivisionError message
}
```

Syntax errors produce the generic message `"Execution failed, unable to retrieve python exception."` because the error occurs before the wrapper harness can run. Look at JVM stderr for the Python traceback.

---

## System Properties Reference

| Property | Default | Description |
|----------|---------|-------------|
| `org.eclipse.python4j.path` | (none) | Custom Python path prepended/appended to JavaCPP packages |
| `org.eclipse.python4j.path.append` | `before` | How to combine custom and JavaCPP paths: `before`, `after`, or `none` |
| `org.eclipse.python4j.release_gil_automatically` | `true` | Set to `false` when embedding in an existing Python host |
| `org.eclipse.python4j.python.initialize` | `true` | Set to `false` to skip `Py_InitializeEx` (manual init) |
| `org.eclipse.python4j.numpyimport` | `true` | Set to `false` to skip numpy import in `NumpyArray.init()` |
| `org.eclipse.python4j.create_npy_python` | `false` | Use Python ctypes path for numpy array creation instead of C API |

All properties must be set before any Python4J class is loaded.

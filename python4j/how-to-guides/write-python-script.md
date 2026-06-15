---
title: "Python4J Getting Started"
description: "Setting up Python4J — Maven dependencies, executing Python code, and variable I/O"
---

# Python4J Getting Started

This page walks through adding Python4J to a Maven project, executing Python code from Java, and passing variables back and forth across the Java/Python boundary.

---

## Maven Setup

Add `python4j-core` to your `pom.xml`. The version should match your ND4J/DL4J version.

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>python4j-core</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

If you also need to exchange `INDArray` objects as NumPy arrays, add the NumPy bridge module:

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>python4j-numpy</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

The `cpython` preset bundled by JavaCPP provides a self-contained Python runtime. No separate Python installation is needed for basic usage. To use packages like scikit-learn or PyTorch that are not bundled, point the interpreter at a local Python environment by setting the system property `org.eclipse.python4j.path` before any Python4J class loads:

```java
System.setProperty("org.eclipse.python4j.path", "/home/user/myenv/lib/python3.9");
```

---

## First Execution

`PythonExecutioner` is the entry point for running Python code. Its static initializer boots the CPython runtime the first time any of its methods are called, so no explicit setup step is required.

```java
import org.nd4j.python4j.PythonExecutioner;
import org.nd4j.python4j.PythonGIL;

try (PythonGIL gil = PythonGIL.lock()) {
    PythonExecutioner.exec("print('Hello from Python')");
}
```

`PythonGIL.lock()` acquires CPython's Global Interpreter Lock and returns an `AutoCloseable` that releases it when the try block exits. You must hold the GIL any time you call `PythonExecutioner` from a thread that is not the JVM main thread. On the main thread, if GIL release has not yet happened, a direct call without `PythonGIL.lock()` works, but using the lock block consistently is the safest pattern.

The `exec()` method wraps your code in a try/except harness. If Python raises an exception, a `PythonException` is thrown on the Java side with the exception message.

---

## Passing Variables into Python

Use `PythonVariable` to bind a Java value to a Python name before execution, and `PythonExecutioner.exec(code, inputs, outputs)` to set inputs, run the code, and collect outputs in one call.

```java
import org.nd4j.python4j.*;
import java.util.Arrays;
import java.util.List;

try (PythonGIL gil = PythonGIL.lock()) {
    // Inputs
    List<PythonVariable> inputs = Arrays.asList(
        new PythonVariable<>("x", PythonTypes.INT, 10L),
        new PythonVariable<>("y", PythonTypes.FLOAT, 3.14)
    );

    // Outputs (values will be populated after exec)
    List<PythonVariable> outputs = Arrays.asList(
        new PythonVariable<>("result", PythonTypes.FLOAT)
    );

    String code = "result = x * y";
    PythonExecutioner.exec(code, inputs, outputs);

    Double result = (Double) outputs.get(0).getValue();
    System.out.println("result = " + result); // result = 31.4
}
```

`PythonVariable<T>` is a triple of (name, type, value). The type is a `PythonType<T>` constant from `PythonTypes` that governs how the value is converted to and from a CPython object.

---

## Supported Types

The following type constants are defined in `PythonTypes`:

| Constant | Python type | Java type |
|----------|-------------|-----------|
| `PythonTypes.STR` | `str` | `String` |
| `PythonTypes.INT` | `int` | `Long` |
| `PythonTypes.FLOAT` | `float` | `Double` |
| `PythonTypes.BOOL` | `bool` | `Boolean` |
| `PythonTypes.BYTES` | `bytes` | `byte[]` |
| `PythonTypes.LIST` | `list` | `List` |
| `PythonTypes.DICT` | `dict` | `Map` |

When `python4j-numpy` is on the classpath, `NumpyArray.INSTANCE` handles `INDArray` / `numpy.ndarray` conversion. See [NumPy Bridge](numpy-bridge.md) for details.

**List elements.** When you put a Java `List` into Python, each element is converted individually using type auto-detection. Mixed-type lists are supported. When you read a Python list back, each element is converted to its closest Java equivalent.

**Dict keys and values.** Keys must be hashable (`str`, `int`, `float`, `bool`). Values can be any supported type.

---

## Reading Variables Back

You can read a single variable by name:

```java
try (PythonGIL gil = PythonGIL.lock()) {
    PythonExecutioner.exec("greeting = 'hello world'");
    PythonVariable<String> v = PythonExecutioner.getVariable("greeting", PythonTypes.STR);
    System.out.println(v.getValue()); // hello world
}
```

Or read all non-private variables that have a recognized type:

```java
try (PythonGIL gil = PythonGIL.lock()) {
    PythonExecutioner.exec("a = 1\nb = 2.5\nc = [1, 2, 3]");
    PythonVariables all = PythonExecutioner.getAllVariables();
    for (PythonVariable<?> pv : all) {
        System.out.println(pv.getName() + " = " + pv.getValue());
    }
}
```

`getAllVariables()` skips names that start with `_` and skips objects whose type is not recognized by any registered `PythonType`.

---

## Running a Python Script File

There is no dedicated "run script file" method, but you can read the file contents and pass them to `exec`:

```java
import java.nio.file.Files;
import java.nio.file.Paths;

String code = new String(Files.readAllBytes(Paths.get("/path/to/script.py")));
try (PythonGIL gil = PythonGIL.lock()) {
    PythonExecutioner.exec(code);
}
```

---

## A Worked Example: Calling scikit-learn

The following example assumes scikit-learn is available on the Python path.

```java
import org.nd4j.python4j.*;
import java.util.*;

try (PythonGIL gil = PythonGIL.lock()) {
    // Pass training data as Python lists
    List<PythonVariable> inputs = Arrays.asList(
        new PythonVariable<>("X_train", PythonTypes.LIST,
            Arrays.asList(
                Arrays.asList(1.0, 2.0),
                Arrays.asList(3.0, 4.0),
                Arrays.asList(5.0, 6.0)
            )
        ),
        new PythonVariable<>("y_train", PythonTypes.LIST,
            Arrays.asList(0L, 1L, 0L)
        )
    );

    List<PythonVariable> outputs = Arrays.asList(
        new PythonVariable<>("prediction", PythonTypes.INT)
    );

    String code =
        "from sklearn.linear_model import LogisticRegression\n" +
        "clf = LogisticRegression()\n" +
        "clf.fit(X_train, y_train)\n" +
        "prediction = int(clf.predict([[2.0, 3.0]])[0])";

    PythonExecutioner.exec(code, inputs, outputs);

    Long prediction = (Long) outputs.get(0).getValue();
    System.out.println("Predicted class: " + prediction);
}
```

---

## Error Handling

Python exceptions are caught by the wrapper harness inside `exec()` and re-raised as `PythonException` on the Java side. `PythonException` is an unchecked (`RuntimeException`) subclass.

```java
try (PythonGIL gil = PythonGIL.lock()) {
    PythonExecutioner.exec("raise ValueError('something went wrong')");
} catch (PythonException e) {
    System.err.println("Python error: " + e.getMessage());
}
```

Syntax errors in the Python code cause `simpleExec` to return a non-zero status code, which `exec` translates into a `PythonException` with the message `"Execution failed, unable to retrieve python exception."`. Inspect the stderr output in your logs for the underlying traceback.

---

## Next Steps

- [NumPy Bridge](numpy-bridge.md) — pass `INDArray` to Python as `numpy.ndarray` without copying data.
- [Advanced Usage](advanced.md) — GIL management across threads, context isolation, subprocess utilities, and garbage collection.

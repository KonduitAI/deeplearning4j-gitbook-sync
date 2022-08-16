---
description: Python4j Key features and brief samples.
---

# Quickstart

Notes to write on: Hello world Python context management Type conversion Collections Garbage collection/memory management Multi threading

## Overview

Python4j is a python execution framework for the JVM based on [javacpp's python distribution](https://github.com/bytedeco/javacpp-presets/tree/master/cpython) and [javacpp's numpy bindings](https://github.com/bytedeco/javacpp-presets/tree/master/numpy) enabling seamless execution of python scripts from the JVM.

A quick example:

```java
    try(PythonGIL pythonGIL = PythonGIL.lock()) {
          try(PythonGC gc = PythonGC.watch()) {
               //execute your code
          }
        }
```

This will execute a code block expicitly locking the GIL. Using java's try/with syntax it will automatically unlock the GIL after the python code has been executed. This concept is used whenever you execute python code. Note, that the GC is also watched here. This is to make sure that anything allocated in python memory gets garbage collected after the code execution is over.

The reason this is needed is because we're effectively spinning up a python interpreter and asking it to create objects and run code for us. If that memory isn't managed properly it can cause memory leaks.

## Python Executioner

Execution of python code happens using the PythonExecutioner. The PythonExecutioner is what manages creating a python interpreter, running some code and extracting the results out.

PythonExecutioner has several methods for executing python code. First, let's show python execution using a standard string.

```java
        try(PythonGIL gil = PythonGIL.lock()){
                try(PythonGC gc = PythonGC.watch()){
                    List<PythonVariable> inputs = new ArrayList<>();
                    inputs.add(new PythonVariable<>("x", PythonTypes.STR, "Hello "));
                    inputs.add(new PythonVariable<>("y", PythonTypes.STR, "World"));
                    PythonVariable out = new PythonVariable<>("z", PythonTypes.STR);
                    String code = "z = x + y";
                    PythonExecutioner.exec(code, inputs, Collections.singletonList(out));
                    System.out.println(out.getValue());
                }
            }catch (Throwable e){
                e.printStackTrace();
            }
```

There are a few new concepts to pay attention to here:

1. PythonVariable: PythonVariable is an object which wraps a variable for python interop. A PythonVariable has the components you would expect: a name, type, and value.
2. PythonTypes: PythonTypes is a wrapper class containing a list of PythonType representing a python class in java. Each PythonType has common utilities for converting to and from java. PythonTypes can be found [here](https://github.com/eclipse/deeplearning4j/blob/%20c715aea405980eb043420af86c671032bbb78ec6/python4j/python4j-core/src/main/java/org/nd4j/python4j/PythonTypes.java) Generally, the default types present in the wrapper class for PythonType are all you need. However, you can extend PythonType for custom types in your python code as needed.
3.  PythonExecutioner.exec(..): This is the entry point for executing python code.  It takes in a series of inputs

    and output names to know what the expected inputs and outputs are for the script. The code passed in will reference

    these variable names within the code. The PythonExecutioner will inject these variables dynamically in to the python

    execution context.

Note, since this depends on javacpp's cpython bindings, this means cpython's native binaries are managed by javacpp. Javacpp will bundle all native binaries for all platforms by default unless you specify a platform. You can do this by specifying a -Dplatform=your-platform-of-choice. You may find more [here](https://github.com/bytedeco/javacpp-presets/wiki/Reducing-the-Number-of-Dependencies) in the javacpp docs.

---
title: Python4j Python Script Execution
short_title: Python4j Python Script Execution
description: Python4j Python Script Execution
category: Python4j
weight: 1
---

# Python Script Execution

## Script execution overview

Python4j runs and manages python interpreters as well as the GIL for a user. When a user attempts to execute a python script, they are calling in to cpython's PyRun rountines. This invokes cpython directly. This functionality can be found in the [PythonExecutioner](https://github.com/eclipse/deeplearning4j/blob/master/python4j/python4j-core/src/main/java/org/nd4j/python4j/PythonExecutioner.java)

When invoking cpython, we actually also wrap the execution code the user specifies in a python script. This script can be found [here](https://github.com/eclipse/deeplearning4j/blob/master/python4j/python4j-core/src/main/resources/org/nd4j/python4j/pythonexec/pythonexec.py). This script mainly ensures we can properly print python stack traces if an error occurs. It does this by ensuring stdout/stderr for the given in memory python interpreter are flushed properly.

## Initialization

Note that before a python script can execute, python4j needs to initialize itself. This happens in the [static initialization block of PythonExecutioner](https://github.com/eclipse/deeplearning4j/blob/7c7f9db097e9f1ff45e137546cabe16b0ed9c78d/python4j/python4j-core/src/main/java/org/nd4j/python4j/PythonExecutioner.java#L51) Be aware of that when executing python scripts.

## Python variable input and output

Python4j allows users to pass in and retrieve in memory python variables. A simple example:

```java
try(PythonGIL pythonGIL = PythonGIL.lock()) {
            List<PythonVariable> inputs = new ArrayList<>();
            inputs.add(new PythonVariable<>("x", PythonTypes.STR, "Hello "));
            inputs.add(new PythonVariable<>("y", PythonTypes.STR, "World"));
            String code = "print(x + y)";
            PythonExecutioner.exec(code, inputs, null);
        }
```

In our case here, we're not retreiving variables, but just passing 2 strings in for printing.

For retrieving variables, we can do either of the following:

```java
 String code = "a = 5\nb = '10'\nc = 20.0";
 List<PythonVariable> vars = PythonExecutioner.execAndReturnAllVariables(code);
```

Exec and return all variables allows us to retrieve any variable tha twas created during the python execution by name. The returned PythonVariables will be named as they were in the python script.

Optionally, a user may also specify a list of variables to be returned. This can be achieved by passing in an output variable list to PythonExecutioner.exec\(..\) as follows:

```java
 try(PythonGIL pythonGIL = PythonGIL.lock()) {
            List<PythonVariable> inputs = new ArrayList<>();
            inputs.add(new PythonVariable<>("x", PythonTypes.STR, "Hello "));
            inputs.add(new PythonVariable<>("y", PythonTypes.STR, "World"));
            PythonVariable out = new PythonVariable<>("z", PythonTypes.STR);
            String code = "z = x + y";
            PythonExecutioner.exec(code, inputs, Collections.singletonList(out));

        }
```

## Execution in a multi threaded environment

Python4j is capable of multi threaded execution of python scripts. The user can manage multiple threads by ensuring that all python calls are wrapped in the double try/with block mentioned in [the overview](../overview) By locking the GIL and watching the garbage collection in any python call, the GIL management is automatically handled for the user.

Optionally, a user may also use the [PythonContextManager](https://github.com/eclipse/deeplearning4j/blob/master/python4j/python4j-core/src/main/java/org/nd4j/python4j/PythonContextManager.java) - creating one context per thread. A "context" is essentially a separate python interpreter with its own variables, memory etc.


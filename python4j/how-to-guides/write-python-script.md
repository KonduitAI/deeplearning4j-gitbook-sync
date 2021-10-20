---
description: How to write a python script for python4j
---

# Write Python Script

## Introduction

Writing a python script in python4j involves first understanding what variables you want to pass in and what variables you want to retrieve, very similar to writing any function in a programming language. In order to learn more about this, please see our [execution overview](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/python4j/reference/execution)

When writing a python script, a user should try to write the script to be as minimal as possible. Focus on the minimal set of inputs, outputs, and code you want to run within a python script. As this is an embedded interpreter, too many complexities arise when trying to run a full blown application. Some complexities include [garbage collection understanding](../reference/garbage-collection.md) and [debugging script execution](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/python4j/reference/execution)

If you are using external libraries, then you need to understand how our [custom python path support](../reference/python-path.md) works.

## Before you write your first script

It is advised to test your python script in a real environment first. This would mean testing in a code editor and debugger for python like pycharm or visual studio code. This will also help you to determine what the script should look like for python4j. The following considerations should be thought about:

1. inputs: the inputs to the script will be passed in from java and should not be declared as explicit variables in your script that's running embedded. These variable declarations will be dynamically created and inserted in to a real python script that gets executed by our execution framework.
2. outputs: the outputs of the script will be passed from real python memory and are kept in memory within the scope of the try/with execution block.
3. dependencies: the dependencies of your application should be bundled separately. The developers recommend a standalone miniconda installation for the target operating system. This version should match the version of cpython provided by python4j to avoid clashing. See \[../reference/python-path] for more information on this topic.

## Writing and executing your first script

Hello world is pretty straightforward. We'll do this write in line:

```java
PythonExecutioner.exec("print('hello world')");
```

This will do as you would expect and print hello world to the console.

Next , we can add concateneate 2 strings. In this example, we pass hello world in as strings. Note that we pass in 2 variables of type string:

```java
try(PythonGIL pythonGIL = PythonGIL.lock()) {
            List<PythonVariable> inputs = new ArrayList<>();
            inputs.add(new PythonVariable<>("x", PythonTypes.STR, "Hello "));
            inputs.add(new PythonVariable<>("y", PythonTypes.STR, "World"));
            String code = "print(x + y)";
            PythonExecutioner.exec(code, inputs, null);
        }
```

We could also pass in 2 ints and add them as well:

```java
try(PythonGIL pythonGIL = PythonGIL.lock()) {
            List<PythonVariable> inputs = new ArrayList<>();
            inputs.add(new PythonVariable<>("x", PythonTypes.INT, 1));
            inputs.add(new PythonVariable<>("y", PythonTypes.INT, 2));
            String code = "print(x + y)";
            PythonExecutioner.exec(code, inputs, null);
        }
```

The supported python types can be found [here](https://github.com/eclipse/deeplearning4j/blob/master/python4j/python4j-core/src/main/java/org/nd4j/python4j/PythonTypes.java#L34) More on types can be found [here](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/python4j/reference/python-types)

If we want to write an actual python script and have python4j load it, we need to read the script in to memory.

This can be achieved with the following:

```java
    String code = FileUtils.readFileToString(new File("path/to/pythonfile.py"), StandardCharsets.UTF_8);
```

From there, we can pass the code to PythonExecutioner.exec(..) as follows:

```java
try(PythonGIL pythonGIL = PythonGIL.lock()) {
            List<PythonVariable> inputs = new ArrayList<>();
            inputs.add(new PythonVariable<>("x", PythonTypes.INT, 1));
            inputs.add(new PythonVariable<>("y", PythonTypes.INT, 2));
             String code = FileUtils.readFileToString(new File("path/to/pythonfile.py"), StandardCharsets.UTF_8);
            PythonExecutioner.exec(code, inputs, null);
        }
```

## Retrieving results

Up till now, we haven't actually retrieved results from the python script, just passed them in. Below is how to retrieve results:

```java
    try(PythonGIL pythonGIL = PythonGIL.lock()) {
            String code = "a = 5\nb = '10'\nc = 20.0";
            List<PythonVariable> vars = PythonExecutioner.execAndReturnAllVariables(code);
        }
```

This will retrieve all results output from the executed python code. If you only want certain variables, then you can do the following:

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

Afterwards, you can read the value from out post execution using:

```java
String value = out.getValue();
```

Note that out is a parameterized type. When retrieving the value, the java runtime will automatically try to cast whatever the output result is from python to the specified type. For more information on types, please see our [types reference](../reference/python-types.md)

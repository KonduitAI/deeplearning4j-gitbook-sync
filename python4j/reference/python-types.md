---
description: Python4j and python types
---

# Python Types

## Types overview

Python4j's types by default come with all of the python primitive types supported. These types include: 1. STR: the string type, mapped to java's string type 2. INT: the integer type, mapped to java's long type 3. FLOAT: the float type, mapped to java's double type 4. BOOL: the boolean type, mapped to java's boolean type 5. BYTES: the bytes type, mapped to java's byte array

These built in types are wrapped in a [PythonTypes](https://github.com/eclipse/deeplearning4j/blob/master/python4j/python4j-core/src/main/java/org/nd4j/python4j/PythonTypes.java#L34) class for users.

## Numpy arrays

If a user wants direct support for numpy arrays as types, a separate dependency of python4j-numpy is required. Including python4j-numpy in your project for a number of projects can be found [here](https://search.maven.org/artifact/org.nd4j/python4j-numpy/1.0.0-M1/jar)

Our numpy support means seamless interop with the numpy format and using numpy arrays inside and outside of python scripts using the exact same memory references with zero copy. This will allow users to benefit from their numpy expertise, but get reasonable performance using the nd4j framework for the core data structure in java.

Note that only dense arrays are supported at this time. When nd4j supports sparse and complex types, we will update this page. Please feel free to ask about this on [our support forums](https://community.konduit.ai/)

## Custom types

If user needs to specify a custom type, all you need to do is extend our [PythonType](https://github.com/eclipse/deeplearning4j/blob/master/python4j/python4j-core/src/main/java/org/nd4j/python4j/PythonType.java#L27-L26) class.

In order for python4j to discover which custom types are available. These are loaded using the java service loader interface. This means a user needs to have a META-INF/services folder on the classpath (usually under src/main/resources) in their project with the fully qualified class name: org.nd4j.python4j.PythonType . All types specified should be fully qualified class names like the abstract class org.nd4j.python4j.PythonType.

For sample implementations of PythonType, please see the inline declarations in our PythonTypes top level class and use those as a starting point. You will need to implement an adapt method and specify the appropriate mapping of python type to java type.

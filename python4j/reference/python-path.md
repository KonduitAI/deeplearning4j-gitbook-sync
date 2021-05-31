---
title: Python4j and custom python path
short_title: Python4j and custom python path
description: Python4j and custom python path
category: Python4j
weight: 1
---

## The default python path

By default, javacpp provides a python path for us by listing its bundled dependencies
that are provided in the javacpp jar files. This includes numpy as well if the user
is using our python4j-numpy module. However, in real world applications many users will need additional libraries in order to run the scripts they built in other environments.

[##Specifying a custom python path](#custom-python-path)

Python4j allows a user to specify a custom python path. This python path should be the same version
as the python version being provided by python4j. In order to specify a custom python path, a user should be aware of 3 properties:
1. org.eclipse.python4j.path: This system property is where a user can specify which python path to use.
A user can obtain this python path with this small snippet:
```python
import sys
import os
print(os.pathsep.join(sys.path))
```
 
2. org.eclipse.python4j.path.append: This system property is how to interoperate with the python path provided
by javacpp. A user can select none, before, or after. This affects the loading order for all libraries.
A dependency clash can happen if a user uses a different version of numpy from the one in javacpp for example.
In order to avoid clashes, specify none for the system property.


## Creating a custom python path setup using miniconda

It is recommended to use an embedded miniconda zipped up as an archive for distributing any dependencies needed for a target platform. In order to setup miniconda, please see [anaconda's installation guide](https://conda.io/projects/conda/en/latest/user-guide/install/index.html)

Afterwards, run the needed conda install commands from the miniconda install directory on the target system. From there, run the command specified above in [our custom python path section](#custom-python-path)

This will print the python path you need to pass to python4j before it initializes.
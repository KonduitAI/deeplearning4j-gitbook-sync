---
title: Python4j and Garbage Collection
short_title: Python4j and Garbage Collection
description: Python4j Garbage Collection and interactions with the JVM
category: Python4j
weight: 1
---

## Python garbage collection

Python garbage collection uses [reference counting](https://devguide.python.org/garbage_collector/)
This happens all within the cpython runtime. Python4j bundles this runtime.

## Javacpp's memory management

Javacpp uses reference counters and phantom references for all of its pointer objects. Please see [here](https://github.com/bytedeco/javacpp/blob/master/src/main/java/org/bytedeco/javacpp/Pointer.java#L282)
for the core behavior. 

## Javacpp garbage collection with python GC

End users should be aware of potential race conditions between javacpp's memory management and deleting pointers with python's GC. When accessing in memory python variables from java, depending on what code is written in python and how variables are managed, variables maybe garbage collected before referencing causing a crash.

In order to avoid issues, try to use python scripts transactionally. This means within a try/with block
that locks the gil as described in [the overview](../overview)

If python variables need to be kept in memory, ensure proper context management of the in memory python variables. Context Management in this case means using a separate python context (essentially a separate interpreter) if a user wants full isolation.
Full isolation is not required in most cases, but maybe desirable for certain use cases.
For now, if you would like more information on this topic, please ask on [our forums](https://community.konduit.ai/) and see our [unit test](https://github.com/eclipse/deeplearning4j/blob/master/python4j/python4j-core/src/test/java/PythonContextManagerTest.java#L48)
for usage.




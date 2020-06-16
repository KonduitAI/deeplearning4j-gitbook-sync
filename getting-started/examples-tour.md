---
title: Tour of Eclipse Deeplearning4j Examples
short_title: Examples Tour
description: Brief tour of available examples in DL4J.
category: Get Started
weight: 10
---

# Examples Tour

Deeplearning4J has a wealth of examples of how to use its many parts. You can find the examples in the [Examples Repository](https://github.com/eclipse/deeplearning4j-examples).

### Prerequisites

The [example repository](https://github.com/eclipse/deeplearning4j-examples) consists of several separate Maven Java projects, each with their own pom files. Maven is a popular build automation tool for Java Projects. The contents of a "pom.xml" file dictate the configurations. Read more about how to configure Maven [here](../config/maven.md).

Users can also refer to the [simple sample project provided](https://github.com/eclipse/deeplearning4j-examples/tree/master/mvn-project-template) to get started with a clean project from scratch.

Build tools are considered standard software engineering best practice. Besides this the complexities posed by the projects in the DL4J ecosystem make dependencies too difficult to manage manually. All the projects in the DL4J ecosystem can be used with other build tools like Gradle, SBT etc. More information on that can be found [here](../config/buildtools.md).

### Example Content

Projects are based on what functionality the included examples demonstrate to the user and not necessarily which library in the DL4J stack the functionality lives in.

Examples in a project are in general separated into "quickstart" and "advanced".

Each project README also lists all the examples it contains, with a recommended order to explore them in.

* [dl4j-examples](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/README.md) This project contains a set of examples that demonstrate use of the high level DL4J API to build a variety of neural networks. Some of these examples are end to end, in the sense they start with raw data, process it and then build and train neural networks on it.
* [tensorflow-keras-import-examples](https://github.com/eclipse/deeplearning4j-examples/blob/master/tensorflow-keras-import-examples/README.md) This project contains a set of examples that demonstrate how to import Keras h5 models and TensorFlow frozen pb models into the DL4J ecosystem. Once imported into DL4J these models can be treated like any other DL4J model - meaning you can continue to run training on them or modify them with the transfer learning API or simply run inference on them.
* [dl4j-distributed-training-examples](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-distributed-training-examples/README.md) This project contains a set of examples that demonstrate how to do distributed training, inference and evaluation in DL4J on Apache Spark. DL4J distributed training employs a "hybrid" asynchronous SGD approach - further details can be found in the distributed deep learning documentation [here](https://deeplearning4j.konduit.ai/distributed-deep-learning/intro)
* [cuda-specific-examples](https://github.com/eclipse/deeplearning4j-examples/blob/master/cuda-specific-examples/README.md) This project contains a set of examples that demonstrate how to leverage multiple GPUs for data-parallel training of neural networks for increased performance.
* [samediff-examples](https://github.com/eclipse/deeplearning4j-examples/blob/master/samediff-examples/README.md) This project contains a set of examples that demonstrate the SameDiff API. SameDiff \(which is part of the ND4J library\) can be used to build lower level auto-differentiating computation graphs. An analogue to the SameDiff API vs the DL4J API is the low level TensorFlow API vs the higher level of abstraction Keras API.
* [data-pipeline-examples](https://github.com/eclipse/deeplearning4j-examples/blob/master/data-pipeline-examples/README.md) This project contains a set of examples that demonstrate how raw data in various formats can be loaded, split and preprocessed to build serializable \(and hence reproducible\) ETL pipelines.
* [nd4j-ndarray-examples](https://github.com/eclipse/deeplearning4j-examples/blob/master/nd4j-ndarray-examples/README.md) This project contains a set of examples that demonstrate how to manipulate NDArrays. The functionality of ND4J demonstrated here can be likened to NumPy.
* [arbiter-examples](https://github.com/eclipse/deeplearning4j-examples/blob/master/arbiter-examples/README.md) This project contains a set of examples that demonstrate usage of the Arbiter library for hyperparameter tuning of Deeplearning4J neural networks.
* [rl4j-examples](https://github.com/eclipse/deeplearning4j-examples/blob/master/rl4j-examples/README.md) This project contains examples of using RL4J, the reinforcement learning library in DL4J.
* [android-examples](https://github.com/eclipse/deeplearning4j-examples/blob/master/android-examples/README.md) This project contains an Android example project, that shows DL4J being used in an Android application.

### Feedback & Contributions

While these set of examples don't cover all the features available in DL4J the intent is to cover functionality required for most users - beginners and advanced. File an issue [here](https://github.com/eclipse/deeplearning4j-examples/issues) if you have feedback or feature requests that are not covered here. We are also available via our [community forum](https://community.konduit.ai/) for questions.  
We welcome contributions from the community. More information can be found [here](https://github.com/eclipse/deeplearning4j/blob/master/CONTRIBUTING.md)  
We **love** hearing from you. Cheers!


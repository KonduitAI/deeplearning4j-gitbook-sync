---
title: Core Concepts in Deeplearning4j
short_title: Core Concepts
description: Introduction to core Deeplearning4j concepts.
category: Get Started
weight: 1
---

# Deeplearning4j Suite Overview

## Eclipse DeepLearning4J

Eclipse Deeplearning4j is a suite of tools for running deep learning on the JVM. It's the only framework that allows you to train models from java while interoperating with the python ecosystem through a mix of python execution via our cpython bindings, model import support, and interop of other runtimes such as tensorflow-java and onnxruntime.

The use cases include importing and retraining models \(Pytorch, Tensorflow, Keras\) models and deploying in JVM Micro service environments, mobile devices, IoT, and Apache Spark. It is a great compliment to your python environment for running models built in python, deployed to or packaged for other environments.

Deeplearning4j has several submodules including:

1. **Samediff**: a tensorflow/pytorch like framework for execution of complex graphs. This framework is lower level, but very flexible. It's also the base api for running onnx and tensorflow graphs.
2. **Nd4j**: numpy ++ for java. Contains a mix of numpy operations and tensorflow/pytorch operations.
3. **Libnd4j**: A lightweight, standalone c++ library enable math code to run on different devices. Optimizable for running on a wide variety of devices.
4. **Python4j**: A python script execution framework easing deployment of python scripts in to production.
5. **Apache Spark Integration**: An integration with the Apache Spark framework enabling execution of deep learning pipelines on spark
6. **Datavec**: A data transformation library converting raw input data to tensors suitable for running neural networks on.

## How to use this website

This website follows the [divio framework](https://documentation.divio.com/) layout. This website has several sections of documentation following this layout. Below is an overview of the sections of the site:

1. **Multi project** contains all cross project documentation such as end to end training and other whole project related documentation. This should be the default entry point for those getting started.
2. **Deeplearning4j** contains all of the documentation related to the core deeplearning4j apis such as the multi layer network and the computation graph. Consider this the high level framework for building neural networks. If you would like something lower level like tensorflow or pytorch, consider using samediff
3. **Samediff** contains all the documentation related to the samediff submodule of ND4j. Samediff is a lower level api for building neural networks similar to pytorch or tensorflow with built in automatic differentiation.
4. **Datavec** contains all the documentation related to our data transformation library datavec.
5. **Python4j** contains all the documentation related to our cpython execution framework python4j.
6. **Libnd4j** contains all the documentation related to our underlying C++ framework libnd4j.
7. **Apache Spark** contains all of the documentation related to our Apache Spark integration.
8. **Concepts/Theory** contains all of the documentation related to general mathematical or computer science theory needed to understand various aspects of the framework.

**Open Source**

The libraries are completely open-source, Apache 2.0 under open governance at the [Eclipse foundation](https://eclipse.org/). The Eclipse Deeplearning4j project welcomes all contributions. See our [community](https://community.konduit.ai/) and our [Contribution guide](https://github.com/eclipse/deeplearning4j/blob/master/CONTRIBUTING.md) to get involved.

**JVM/Python/C++**

Deeplearning4j can either be a compliment to your existing workflows in python and c++ or a standalone library for you to build and deploy models. Use what components you find useful.


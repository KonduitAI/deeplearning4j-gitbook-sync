---
title: Core Concepts in Deeplearning4j
short_title: Core Concepts
description: Introduction to core Deeplearning4j concepts.
category: Get Started
weight: 1
---

# Eclipse DeepLearning4J

{% hint style="info" %}
如果您希望阅读中文文档，请查看[中文文档](https://deeplearning4j.konduit.ai/v/zhong-wen-v1.0.0/)。
{% endhint %}

Eclipse Deeplearning4j is a model import deployment framework for retraining models (pytorch, tensorflow,keras) and deploying in JVM Micro service environments, mobile devices, iot, and Apache Spark. It is a great compliment to your python environment for running models built in python, deployed to or packaged for other environments.

Deeplearning4j has several submodules including:

1. Samediff: a tensorflow/pytorch like framework for execution of complex graphs. This framework is lower level, but very flexible. It's also the base api for running onnx and tensorflow graphs.

2. Nd4j: numpy ++ for java. Contains a mix of numpy operations and tensorflow/pytorch operations.

3. Libnd4j: A lightweight, standalone c++ library enable math code to run on different devices. Optimizable for running on a wide variety of devices.

4. Python4j: A python script execution framework easing deployment of python scripts in to production.

5. Apache spark integration: An integration with the apache spark framework enabling execution of deep learning pipeliens on spark

6. Datavec: A data transformation library converting raw input data to tensors suitable for running neural networks on.



**Open Source**

The libraries are completely open-source, Apache 2.0 under open governance at the [Eclipse foundation](https://eclipse.org/).
The Eclipse Deeplearning4j project welcomes all contributions. See our [community](https://community.konduit.ai/) and our 
[Contribution guide](https://github.com/eclipse/deeplearning4j/blob/master/CONTRIBUTING.md) to get involved.


**JVM/Python/C++**

Deeplearning4j can either be a compliment to your existing workflows in python and c++ or a standalone library for you to build and deploy models.
Use what components you find useful.



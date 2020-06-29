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

Eclipse Deeplearning4j is the first commercial-grade, open-source, distributed deep-learning library written for Java and Scala. Integrated with Hadoop and Apache Spark, DL4J brings AI to business environments for use on distributed GPUs and CPUs.

**Distributed**

DL4J takes advantage of the latest distributed computing frameworks including Apache Spark and Hadoop to accelerate training. On multi-GPUs, it is equal to Caffe in performance.

**Open Source**

The libraries are completely open-source, Apache 2.0, and maintained by the developer community and Konduit team.

**JVM/Python/C++**

Deeplearning4j is written in Java and is compatible with any JVM language, such as Scala, Clojure or Kotlin. The underlying computations are written in C, C++ and Cuda. Keras will serve as the Python API.

### What's included?

Deep neural nets are capable of record-breaking accuracy. For a quick neural net introduction, please visit our [overview ](getting-started/core-concepts.md)page. In a nutshell, Deeplearning4j lets you compose deep neural nets from various shallow nets, each of which form a so-called \`layer\`. This flexibility lets you combine variational autoencoders, sequence-to-sequence autoencoders, convolutional nets or recurrent nets as needed in a distributed, production-grade framework that works with Spark and Hadoop on top of distributed CPUs or GPUs.

There are a lot of parameters to adjust when you're training a deep-learning network. We've done our best to explain them, so that Deeplearning4j can serve as a DIY tool for Java, Scala, Clojure and Kotlin programmers.  



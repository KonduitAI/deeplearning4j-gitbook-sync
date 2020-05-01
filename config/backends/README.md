---
title: Deeplearning4j Hardware and CPU/GPU Setup
short_title: GPU/CPU Setup
description: 'Hardware setup for Eclipse Deeplearning4j, including GPUs and CUDA.'
category: Configuration
weight: 1
---

# Backends

## ND4J backends for GPUs and CPUs

You can choose GPUs or native CPUs for your backend linear algebra operations by changing the dependencies in ND4J's POM.xml file. Your selection will affect both ND4J and DL4J being used in your application.

If you have CUDA v9.2+ installed and NVIDIA-compatible hardware, then your dependency declaration will look like:

```markup
<dependency>
 <groupId>org.nd4j</groupId>
 <artifactId>nd4j-cuda-10.2</artifactId>
 <version>1.0.0-beta6</version>
</dependency>
```

As of now, the `artifactId` for the CUDA versions can be one of `nd4j-cuda-9.2`, `nd4j-cuda-10.0`, `nd4j-cuda-10.1` or `nd4j-cuda-10.2`.

You can also find the available CUDA versions via [Maven Central search](https://search.maven.org/search?q=nd4j-cuda) or in the [Release Notes](https://deeplearning4j.org/release-notes.html).

Otherwise you will need to use the native implementation of ND4J as a CPU backend:

```markup
<dependency>
 <groupId>org.nd4j</groupId>
 <artifactId>nd4j-native</artifactId>
 <version>1.0.0-beta6</version>
</dependency>
```

## Building for Multiple Operating Systems

If you are developing your project on multiple operating systems/system architectures, you can add `-platform` to the end of your `artifactId` which will download binaries for most major systems.

```markup
<dependency>
 ...
 <artifactId>nd4j-native-platform</artifactId>
 ...
</dependency>
```

## CuDNN

See our page on [CuDNN](config-cudnn.md).

## CUDA Installation

Check the NVIDIA guides for instructions on setting up CUDA on the NVIDIA [website](http://docs.nvidia.com/cuda/).


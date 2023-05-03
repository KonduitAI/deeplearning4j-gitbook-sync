---
title: Deeplearning4j Hardware and CPU/GPU Setup
short_title: GPU/CPU Setup
description: 'Hardware setup for Eclipse Deeplearning4j, including GPUs and CUDA.'
category: Configuration
weight: 1
---

# Backends

ND4J works atop so-called backends, or linear-algebra libraries, such as Native nd4j-native and nd4j-cuda-10.2 \(GPUs\), which you can select by pasting the right dependency into your project’s POM.xml file.

## ND4J backends for GPUs and CPUs

You can choose GPUs or native CPUs for your backend linear algebra operations by changing the dependencies in ND4J's POM.xml file. Your selection will affect both ND4J and DL4J being used in your application.

If you have CUDA v9.2+ installed and NVIDIA-compatible hardware, then your dependency declaration will look like:

```markup
<dependency>
 <groupId>org.nd4j</groupId>
 <artifactId>nd4j-cuda-10.2</artifactId>
 <version>1.0.0-beta7</version>
</dependency>
```

As of now, the `artifactId` for the CUDA versions can be one of `nd4j-cuda-9.2`, `nd4j-cuda-10.0`, `nd4j-cuda-10.1` or `nd4j-cuda-10.2`.

You can also find the available CUDA versions via [Maven Central search](https://search.maven.org/search?q=nd4j-cuda) or in the [Release Notes](../../getting-started/release-notes.md).

Otherwise you will need to use the native implementation of ND4J as a CPU backend:

```markup
<dependency>
 <groupId>org.nd4j</groupId>
 <artifactId>nd4j-native</artifactId>
 <version>1.0.0-beta7</version>
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

## Bundling multiple Backends

For enabling different backends at runtime, you set the priority with your environment via the environment variable

```text
BACKEND_PRIORITY_CPU=SOME_NUM
BACKEND_PRIORITY_GPU=SOME_NUM
```

Relative to the priority, it will allow you to dynamically set the backend type.

## CuDNN

See our page on [CuDNN](config-cudnn.md).

## CUDA Installation

Check the NVIDIA guides for instructions on setting up CUDA on the NVIDIA [website](http://docs.nvidia.com/cuda/).

## Troubleshooting

### Nd4jBackend$NoAvailableBackendException

```markup
 org.nd4j.linalg.factory.Nd4jBackend$NoAvailableBackendException: Please ensure that you have an nd4j backend on your classpath. Please see: https://deeplearning4j.konduit.ai/multi-project/explanation/configuration/backends#nd4jbackendusdnoavailablebackendexception
	at org.nd4j.linalg.factory.Nd4jBackend.load(Nd4jBackend.java:221)
	at org.nd4j.linalg.factory.Nd4j.initContext(Nd4j.java:5091)
	... 2 more
```

There are multiple reasons why you might run into this error message.

1. You haven't configured an ND4J backend at all. 
2. You have a jar file that doesn't contain a backend for your platform.
3. You have a jar file that doesn't contain service loader files.

#### You haven't configured any ND4J Backend

Read this page and add a ND4J Backend to your dependencies:

{% page-ref page="./" %}

#### You have a jar file that doesn't contain a backend for your platform.

This happens when you use a non `-platform` type backend dependency definition. In this case, only the Backend for the system that the jar file was built on will be included.  

To solve this issue, use `nd4j-native-platform` instead of `nd4j-native`, if you are running on CPU and `nd4j-cuda-10.2-platform`  instead of `nd4j-cuda-10.2` when using the GPU backend.

If the jar file only contains the GPU backend, but your system has no CUDA capable \(CC &gt;= 3.5\) GPU or CUDA isn't installed on the system, the CPU Backend should be used instead.

#### You have a jar file that doesn't contain service loader files.

ND4J uses the Java [ServiceLoader](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/ServiceLoader.html) in order to detect which backends are available on the class path. Depending on your uberjar packaging configuration, those files might be stripped away or broken.

To double check that the required files are included, open your uberjar and make sure it contains `/META-INF/services/org.nd4j.linalg.factory.Nd4jBackend`. Then open the file, and make sure there are entries for all of your configured backends.

If your uberjar does not contain that file, or if not all of the configured backends are listed there, you will have to reconfigure your shade plugin. See [ServicesResourceTransformer ](https://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#ServicesResourceTransformer)documentation for how to do that.


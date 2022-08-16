---
description: CPU and AVX support in ND4J/Deeplearning4j
---

# CPU

## What is AVX, and why does it matter?

AVX (Advanced Vector Extensions) is a set of CPU instructions for accelerating numerical computations. See [Wikipedia](https://en.wikipedia.org/wiki/Advanced\_Vector\_Extensions) for more details.

Note that AVX only applies to nd4j-native (CPU) backend for x86 devices, not GPUs and not ARM/PPC devices.

Why AVX matters: performance. You want to use the version of ND4J compiled with the highest level of AVX supported by your system.

AVX support for different CPUs - summary:

* Most modern x86 CPUs: AVX2 is supported
* Some high-end server CPUs: AVX512 may be supported&#x20;
* Old CPUs (pre 2012) and low power x86 (Atom, Celeron): No AVX support (usually)&#x20;

Note that CPUs supporting later versions of AVX include all earlier versions also. This means it's possible run a generic x86 or AVX2 binary on a system supporting AVX512. However it is not possible to run binaries built for later versions (such as avx512) on a CPU that doesn't have support for those instructions.

In version 1.0.0-beta6 and later you may get a warning as follows, if AVX is not configured optimally:

```
*********************************** CPU Feature Check Warning ***********************************
Warning: Initializing ND4J with Generic x86 binary on a CPU with AVX/AVX2 support
Using ND4J with AVX/AVX2 will improve performance. See deeplearning4j.org/cpu for more details
Or set environment variable ND4J_IGNORE_AVX=true to suppress this warning
************************************************************************************************
```

This warning has been removed in more recent versions as it's more confusing to users and out of date.

## Configure mkl usage

When using the nd4j-native backend on intel platforms, our openblas bindings give the ability to also use mkl instead. In order to use mkl, set the system property as follows eitehr on launch or before Nd4j is initialized with Nd4j.create():

```java
 System.setProperty("org.bytedeco.openblas.load", "mkl");
```

## Configuring AVX in ND4J/DL4J

As noted earlier, for best performance you should use the version of ND4J that matches your CPU's supported AVX level.

ND4J defaults configuration (when just including the nd4j-native or nd4j-native-platform dependencies without maven classifier configuration) is "generic x86" (no AVX) for nd4j/nd4j-platform dependencies.

To configure AVX2 and AVX512, you need to specify a classifier for the appropriate architecture.

The following binaries (nd4j-native classifiers) are provided for x86 architectures:

* Generic x86 (no AVX): `linux-x86_64`, `windows-x86_64`, `macosx-x86_64`&#x20;
* AVX2: `linux-x86_64-avx2`, `windows-x86_64-avx2`, `macosx-x86_64-avx2`
* AVX512: `linux-x86_64-avx512`

As of 1.0.0-M1, the following combinations are also possible with [onednn](https://github.com/oneapi-src/oneDNN):

* Generic x86 (no AVX): `linux-x86_64-onednn`, `windows-x86_64-onednn`, `macosx-x86_64-onednn`&#x20;
* AVX2: `linux-x86_64-onednn-avx2`, `windows-x86_64-onednn-avx2`, `macosx-x86_64-onednn-avx2`
* AVX512: `linux-x86_64-onednn-avx512`

**Example: Configuring AVX2 on Windows (Maven pom.xml)**

```markup
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${nd4j.version}</version>
</dependency>

<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${nd4j.version}</version>
    <classifier>windows-x86_64-avx2</classifier>
</dependency>
```

**Example: Configuring AVX512 on Linux (Maven pom.xml)**

```markup
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${nd4j.version}</version>
</dependency>

<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${nd4j.version}</version>
    <classifier>linux-x86_64-avx512</classifier>
</dependency>
```

**Example: Configuring AVX512 on Linux with onednn(Maven pom.xml)**

```markup
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${nd4j.version}</version>
</dependency>

<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${nd4j.version}</version>
    <classifier>linux-x86_64-onednn-avx512</classifier>
</dependency>
```

Note that you need _both_ nd4j-native dependencies - with and without the classifier.

In the examples above, it is assumed that a Maven property `nd4j.version` is set to an appropriate ND4J version such as `1.0.0-M1.1`

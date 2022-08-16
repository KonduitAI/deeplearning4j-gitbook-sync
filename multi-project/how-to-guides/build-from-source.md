---
description: Instructions to build all DL4J libraries from source.
---

# Build From Source

A reference for building dl4j from source can be found for every platform in our [workflows](https://github.com/eclipse/deeplearning4j/tree/master/.github/workflows). For maintenance reasons, we would prefer to have a canonical source of up to date build information for users rather than out of date install instructions in this guide. This guide will contain specific long lived tips for how to interpret the workflows and what to consider when building.

For an overview of the GitHub actions workflows see the [overview doc](developer-docs/github-actions-build-infra.md)

This document will cover the specific components of the build by platform rather than step through what's already in the workflows. If you have suggestions for improving this document, please comment over at [the community forums](https://community.konduit.ai/)

Core steps:

1. Building libnd4j for your specific platform
2. Linking the nd4j backend you want to compile for against libnd4j via JavaCPP
3. Compiling the rest of the code in to jar files

## Key concepts

1. Libnd4j is a CMake based c++ project that supports running optimized math code on different architectures. Its sole focus is being a tiny self contained library for running math kernels. It can link against optimized BLAS routines, platform specific CNN libraries such as OneDNN and CuDNN, and contains hundreds of math kernels for implementing neural networks and other math routines.
2. Maven: Maven is the core build tool for deeplearning4j. Understanding maven is key to building deeplearning4j from source
3. Maven and CMake: For compiling libnd4j, we invoke a buildnativeoperations.sh wrapper script via maven. buildnativeoperations.sh in turn automatically sets up CMake to then build the c++ project
4. pi\_build.sh: This is our build script for embedded and ARM based platforms. It focuses on cross compilation running on a Linux x86 based platform.
5. buildnativeoperations.sh: The main build script for libnd4j. It initializes CMake and invokes CMake compilation for the user on whatever platform the user is currently on unless the user specifies an alternative platform. Specifying a different platform is possible for android for example.

## Building for x86\_64

The main considerations for building on x86\_64 are:&#x20;

1. Whether to compile for avx2 or avx512&#x20;
2. Whether to use OpenBLAS or MKL
3. Whether to link against OneDNN

From there, the normal platform specific libraries should be installed before hand. Up to date install instructions can be found in our CPU builds for [Windows](https://github.com/eclipse/deeplearning4j/blob/master/.github/workflows/build-deploy-windows.yml), [Mac](https://github.com/eclipse/deeplearning4j/blob/master/.github/workflows/build-deploy-mac.yml) and [Linux](https://github.com/eclipse/deeplearning4j/blob/master/.github/workflows/build-deploy-linux-x86\_64.yml)

## Building for ARM

ARM based builds all link against the [armcompute library](https://github.com/ARM-software/ComputeLibrary/tree/master/arm\_compute) by default and, as mentioned above, use the pi\_build.sh script for building libnd4j on specific platforms. Note that pi\_build.sh can also be used to compile all of dl4j for a specific project.

pi\_build.sh mainly focuses on cross compilation.

In order to properly use the pi\_build.sh script, a number of environment variables should be set. Per platform, you can find these environment variables in the final build step under the environment section.

If you would like to compile deeplearning4j on an actual ARM device, please use the normal buildnativeoperations.sh workflow.

## Building for CUDA

In order to compile deeplearning4j for a particular version, you must first invoke `change-cuda-versions.sh` in the root directory:

```bash
./change-cuda-versions.sh $YOUR_CUDA_VERSION
```

This will ensure that all library versions are set to the appropriate version. Ensure that the CUDA toolkit you need is installed. If you intend on using CuDNN, ensure that is also installed correctly. For installing CUDA, consider using [our install scripts](https://github.com/KonduitAI/cuda-install/tree/master/.github/actions/) as a reference if you intend on doing automated installs.

{% hint style="info" %}
Jetson nano users: please see [this thread](https://community.konduit.ai/t/cuda-on-jetson-nano/1364) for successfully compiling deeplearning4j on Jetson nano.

In short: It relies on CUDA 10.0. The [JavaCPP presets](https://github.com/bytedeco/javacpp-presets) for CUDA are also only compiled for arm64 for CUDA 10.0. You can find the supported CUDA versions for CUDA 10.0 [here](https://repo1.maven.org/maven2/org/bytedeco/cuda/10.0-7.4-1.5/) If you would like something more up to date, please feel free to contact us over at [our forums](https://community.ko)\
As of 1.0.0-M1.1 you can also use updated dependencies:
{% endhint %}

```
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-cuda-10.2</artifactId>
  <version>1.0.0-M1.1</version>
</dependency>
```

## Note for windows users

We use msys2 for compiling libnd4j. CUDA requires MSVC in order to be installed in order to properly compile CUDA kernels. If you want to compile libnd4j for CUDA from source, please ensure you first invoke the `vcvars.bat` script in a cmd terminal, then launch msys2 manually. For more specifics, please see our Windows [CUDA 11](https://github.com/eclipse/deeplearning4j/blob/master/.github/workflows/build-deploy-windows-cuda-11.0.yml) and [11.2](https://github.com/eclipse/deeplearning4j/blob/master/.github/workflows/build-deploy-windows-cuda-11.0.yml) build files.

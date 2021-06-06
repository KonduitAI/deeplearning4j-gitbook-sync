---
title: Buidling Deeplearning4j from Source
short_title: Build from Source
description: Instructions to build all DL4J libraries from source.
category: Get Started
weight: 10
---

# Build from Source

A reference for building dl4j from source can be found for every platform in our [workflows](https://github.com/eclipse/deeplearning4j/tree/master/.github/workflows). For maintenance reasons, we would prefer to have a canonicial source of up to date build information for users
rather than out of date install instructions in this guide. This guide will contain specific long lived tips for how to interpret the workflows and what to consider when building.

For an overview of the github actions workflows see the [overview doc](../developer-docs/github-actions-overview.md)

This document will cover the specific components of the build by platform rather than step through what's already in the workflows. 
If you have suggestions for improving this document, please comment over at [the community forums](https://community.konduit.ai/)


Core steps:

1. Building libnd4j for your specific platform
2. Linking the nd4j backend you want to compile for against libnd4j via javacpp
3. Compiling the rest of the code in to jar files


## Key concepts

1. Libnd4j is a cmake based c++ project that supports running optimized math code on different architectures. 
Its sole focus is being a tiny self contained library for running math kernels.
It can link against optimized BLAS routines, platform specific cnn libraries such as onednn and cudnn, and 
contains hundreds of math kernels for implementing neural networks and other math routines.

2. Maven: Maven is the core build tool for deeplearning4j. Understanding maven is key to building deeplearning4j from source

3. Maven and cmake: For compilng libnd4j, we invoke a  buildnativeoperations.sh wrapper script via maven.
buildnativeoperations.sh in turn automatically sets up cmake to then build the c++ project

4. pi_build.sh: This is our build script for embedded and ARM based platforms. It focuses on cross compilation running on a linux x86
based platform.

5. buildnativeoperations.sh: The main build script for libnd4j. It initializes cmake and invokes cmake compilation for the user
on whatever platform the user is currently on unless the user specifies an alternative platform. Specifying a different platform is possible
for android for example.



x86_64 based builds:


The main considerations for building on x86_64 are:
1. Whether to compile for avx2 or avx512
2. Whether to use openblas or mkl
3. Whether to link against onednn

From there, the normal platform specific libraries should be installed before hand. Up to date install instructions can be found in our cpu builds
for [windows](https://github.com/eclipse/deeplearning4j/blob/master/.github/workflows/build-deploy-windows.yml),[mac](https://github.com/eclipse/deeplearning4j/blob/master/.github/workflows/build-deploy-mac.yml) and [linux](https://github.com/eclipse/deeplearning4j/blob/master/.github/workflows/build-deploy-linux-x86_64.yml)



ARM based builds:

ARM based builds all link against the [armcompute library](https://github.com/ARM-software/ComputeLibrary/tree/master/arm_compute) by default and, as mentioned above, use the pi_build.sh script for building libnd4j on specific platforms. Note that pi_build.sh can also be used to compile all of dl4j for a specific project.

pi_build.sh mainly focuses on cross compilation.


In order to properly use the pi_build.sh script, a number of environment variables should be set. Per platform, you can find these environment variables in the final build step under the environment section.


If you would like to compile deeplearning4j on an actual ARM device, please use the normal buildnativeoperations.sh workflow.

Note that the pi_build.sh 

Cuda builds:

In order to compile deeplearning4j for a particular version, note that you have to first invoke:
```bash
./change-cuda-versions.sh $YOUR_CUDA_VERSION
```

in the root directory. This will ensure that all library versions are set to the appropriate version.
Ensure that the cuda toolkit you need is installed. If you intend on using cudnn, ensure that is also installed correctly.
For installing cuda, consider using [our install scripts](https://github.com/KonduitAI/cuda-install/tree/master/.github/actions/)
as a reference if you intend on doing automated installs.


Note for jetson nano users, please see [this thread](https://community.konduit.ai/t/cuda-on-jetson-nano/1364)
for successfully compiling deeplearning4j on jetson nano.

In short: relies on cuda  10.0. The [javacpp presets](https://github.com/bytedeco/javacpp-presets) for cuda are also only compiled for arm64 for cuda 10.0. You can find the supported cuda versions for cuda 10.0 [here](https://repo1.maven.org/maven2/org/bytedeco/cuda/10.0-7.4-1.5/)
If you would like something more up to date, please feel free to contact us over at [our forums](https://community.ko)


Note for windows users:

We use msys2 for compiling libnd4j. Cuda requires MSVC in order to be installed in order to properly compile cuda kernels.
If you want to compile libnd4j for cuda from source, please ensure you first invoke the vcvars.bat script in a cmd terminal, then launch
msys2 manually.
For more specifics, please see our windows [cuda 11](https://github.com/eclipse/deeplearning4j/blob/master/.github/workflows/build-deploy-windows-cuda-11.0.yml) and [11.2](https://github.com/eclipse/deeplearning4j/blob/master/.github/workflows/build-deploy-windows-cuda-11.0.yml) build files.


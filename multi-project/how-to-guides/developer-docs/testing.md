---
title: 'How to conduct a release to Maven Central'
short_title: Conducting a release to Maven Central
description: 'How to conduct a release to Maven Central'
category: Developer
weight: 3
---
# How a release works

Deeplearning4j has several steps to a release. Below is a brief outline
with follow on descriptions.


1. Compile libnd4j for different cpu architectures
2. Ensure the current javacpp dependencies such as python, mkldnn, cuda, .. are up to date
3. Run all integration tests on core platforms (windows, mac, linux) with both cpu and gpu
4. Create a staging repository for testing using github actions running manually on each platform
5. Update the examples to be compatible with the latest release
6. Run the deeplearning4j-examples as a litmus tests on all platforms (including embedded)
to sanity check platform specific numerical bugs using the staging repository
7. Double check any user related bugs to see if they should block a release 
8. Hit release button 





## Compile libnd4j on different cpu architectures

Compiling libnd4j on different cpu architectures ensures there is platform optimized math in c++ for each platform. The [single code base](https://github.com/eclipse/deeplearning4j/tree/master/libnd4j) is a self contained cmake project that can be run on different platforms.
In each [github actions workflow](https://github.com/eclipse/deeplearning4j/tree/master/.github/workflows) there are steps for deploying for each platform.

At the core of compiling from source for libnd4j is a maven pom.xml that is run as part of the overall build process that invokes our [build script](https://github.com/eclipse/deeplearning4j/blob/master/libnd4j/buildnativeoperations.sh) with various parameters that then get passed to our overall cmake structure for compilation. This script exists to formalize some of the required parameters for invokving cmake. Any developer is welcome to invoke cmake directly.

* Platform compatibility
We currently compile libnd4j on ubuntu 16.04. This means glibc 2.23.
For our cuda builds, we use gcc7.
Users of older glibc versions may need to compile from source. For our standard release, we try to keep it reasonably old, but do not support end of lifed
end of linux distributions for public builds.


* Platform specific helpers

Each build of libnd4j links against an accelerated backend for [blas](http://www.netlib.org/blas/) and convolution operations such as
[onednn](https://github.com/oneapi-src/oneDNN), [cudnn](https://developer.nvidia.com/cudnn), or [armcompute](https://www.arm.com/why-arm/technologies/compute-library) The implementations for each platform can be found [here](https://github.com/eclipse/deeplearning4j/tree/18d165c915738793a30087f9a39adaa0d132a692/libnd4j/include/ops/declarable/platform)


##  Ensure the current javacpp dependencies such as python, mkldnn, cuda, .. are up to date

This is a step that just ensures that the dl4j release matches the current state of the dependencies provided by javacpp on maven central.
This affects every module including python4j, nd4j-native/cuda, datavec-image, among others. The versions of everything can be found in the
top level [deeplearning4j pom](https://github.com/eclipse/deeplearning4j/blob/821a0a4727bfd5bc312723b8864330566706bf9b/pom.xml#L199)
The general convention is library version followed by a - and the version of javacpp that that version uses.

Of note here is that certain older versions of libraries can use older javacpp versions. It is recommended that that the desired version
be up to date if possible. Otherwise, if an older version of javacpp is the only version available, this is generally ok.


## Run all integration tests on core platforms (windows, mac, linux) with both cpu and gpu

We run all of the major integration tests on the core major platforms where higher end compute is accessible. This is generally a bigger machine.
It is expected that some builds can take up to 2 hours depending on the specs of the desired machine.

This step may also involve invoking tests with specific tags if only running a subset of tests is desired. This can be achived using the [surefire plugin](https://maven.apache.org/surefire/maven-surefire-plugin/examples/junit.html) -Dgroups flag.

## Update the examples to be compatible with the latest release

To ensure the examples stay compatible with the current release, we also tag the release version to be the latest version found on maven central.
This step may also involve adding or removing examples for new or deprecated features respectivley.


##  Run the deeplearning4j-examples as a litmus tests on all platforms (including embedded)

The examples contain a set of tests which just allow us to run maven clean test on a small number of examples. Instead of us picking examples manually, 
we can just run mvn clean test on any platform we need by just specifying a version of dl4j to depend on and *usually* a [staging repository](https://central.sonatype.org/publish/publish-guide/)


## Double check any user related bugs to see if they should block a release 

Generally, sometimes users will raise issues right before a release that can be critical. It is the sole discretion of the maintainers
to ask the user to use snapshots or to wait for a follow on version. For certain fixes, we will publish quick bugfix releases.
If your team has specific requirements on a release, please contact us on the [community forums](https://community.konduit.ai)

## Hit release button 

This means after [closing a staging repository](https://central.sonatype.org/publish/publish-guide/), hitting the release button
initiating a sync of the staging repository with the desired version to maven central. Sync usually takes 2 hours or less.









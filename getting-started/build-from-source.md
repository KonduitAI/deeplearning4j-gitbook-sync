---
title: Buidling Deeplearning4j from Source
short_title: Build from Source
description: Instructions to build all DL4J libraries from source.
category: Get Started
weight: 10
---

# Build from Source

**Note: This guide is outdated. We are working on updating it as soon as possible.**

## Build Locally from Master

**NOTE: MOST USERS SHOULD USE THE RELEASES ON MAVEN CENTRAL AS PER THE QUICK START GUIDE, AND NOT BUILD FROM SOURCE**

_Unless you have a very good reason to build from source \(such as developing new features - excluding custom layers, custom activation functions, custom loss functions, etc - all of which can be added without modifying DL4J directly\) then you shouldn't build from source. Building from source can be quite complex, with no benefit in a lot of cases._

For those developers and engineers who prefer to use the most up-to-date version of Deeplearning4j or fork and build their own version, these instructions will walk you through building and installing Deeplearning4j. The preferred installation destination is to your machine's local maven repository. If you are not using the master branch, you can modify these steps as needed \(i.e.: switching GIT branches and modifying the `build-dl4j-stack.sh` script\).

Building locally requires that you build the entire Deeplearning4j stack which includes:

* [libnd4j](https://github.com/eclipse/deeplearning4j/tree/master/libnd4j)
* [nd4j](https://github.com/eclipse/deeplearning4j/tree/master/nd4j)
* [datavec](https://github.com/eclipse/deeplearning4j/tree/master/datavec)
* [deeplearning4j](https://github.com/eclipse/deeplearning4j)

Note that Deeplearning4j is designed to work on most platforms \(Windows, OS X, and Linux\) and is also includes multiple "flavors" depending on the computing architecture you choose to utilize. This includes CPU \(OpenBLAS, MKL, ATLAS\) and GPU \(CUDA\). The DL4J stack also supports x86 and PowerPC architectures.

## Prerequisites

Your local machine will require some essential software and environment variables set _before_ you try to build and install the DL4J stack. Depending on your platform and the version of your operating system, the instructions may vary in getting them to work. This software includes:

* git
* cmake \(3.2 or higher\)
* OpenMP
* gcc \(4.9 or higher\)
* maven \(3.3 or higher\)

Architecture-specific software includes:

CPU options:

* Intel MKL
* OpenBLAS
* ATLAS

GPU options:

* CUDA

IDE-specific requirements:

* IntelliJ Lombok plugin

DL4J testing dependencies:

* dl4j-test-resources

### Installing Prerequisite Tools

#### Linux

**Ubuntu** Assuming you are using Ubuntu as your flavor of Linux and you are running as a non-root user, follow these steps to install prerequisite software:

```text
sudo apt-get purge maven maven2 maven3
sudo add-apt-repository ppa:natecarlson/maven3
sudo apt-get update
sudo apt-get install maven build-essentials cmake libgomp1
```

#### OS X

Homebrew is the accepted method of installing prerequisite software. Assuming you have Homebrew installed locally, follow these steps to install your necessary tools.

First, before using Homebrew we need to ensure an up-to-date version of Xcode is installed \(it is used as a primary compiler\):

```text
xcode-select --install
```

Finally, install prerequisite tools:

```text
brew update
brew install maven gcc5
```

Note: You can _not_ use clang. You also can _not_ use a new version of gcc. If you have a newer version of gcc, please switch versions with [this link](https://apple.stackexchange.com/questions/190684/homebrew-how-to-switch-between-gcc-versions-gcc49-and-gcc)

#### Windows

libnd4j depends on some Unix utilities for compilation. So in order to compile it you will need to install [Msys2](https://msys2.github.io/).

After you have setup Msys2 by following [their instructions](https://msys2.github.io/), you will have to install some additional development packages. Start the msys2 shell and setup the dev environment with:

```text
pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-cmake mingw-w64-x86_64-extra-cmake-modules make pkg-config grep sed gzip tar mingw64/mingw-w64-x86_64-openblas
```

This will install the needed dependencies for use in the msys2 shell.

You will also need to setup your PATH environment variable to include `C:\msys64\mingw64\bin` \(or where ever you have decided to install msys2\). If you have IntelliJ \(or another IDE\) open, you will have to restart it before this change takes effect for applications started through them. If you don't, you probably will see a "Can't find dependent libraries" error.

### Installing Prerequisite Architectures

Once you have installed the prerequisite tools, you can now install the required architectures for your platform.

#### Intel MKL

Of all the existing architectures available for CPU, Intel MKL is currently the fastest. However, it requires some "overhead" before you actually install it.

1. Apply for a license at [Intel's site](https://software.intel.com/en-us/intel-mkl)
2. After a few steps through Intel, you will receive a download link
3. Download and install Intel MKL using [the setup guide](https://software.intel.com/en-us/articles/intel-math-kernel-library-intel-mkl-2020-install-guide)

#### OpenBLAS

**Linux**

**Ubuntu** Assuming you are using Ubuntu, you can install OpenBLAS via:

```text
sudo apt-get install libopenblas-dev
```

You will also need to ensure that `/opt/OpenBLAS/lib` \(or any other home directory for OpenBLAS\) is on your `PATH`. In order to get OpenBLAS to work with Apache Spark, you will also need to do the following:

```text
sudo cp libopenblas.so liblapack.so.3
sudo cp libopenblas.so libblas.so.3
```

**CentOS** Enter the following in your terminal \(or ssh session\) as a root user:

```text
yum groupinstall 'Development Tools'
```

After that, you should see a lot of activity and installs on the terminal. To verify that you have, for example, _gcc_, enter this line:

```text
gcc --version
```

For more complete instructions, [go here](http://www.cyberciti.biz/faq/centos-linux-install-gcc-c-c-compiler/).

**OS X**

You can install OpenBLAS on OS X with Homebrew:

```text
brew install openblas
```

**Windows**

An OpenBLAS package is available for `msys2`. You can install it using the `pacman` command.

#### ATLAS

**Linux**

**Ubuntu** An apt package is available for ATLAS on Ubuntu:

```text
sudo apt-get install libatlas-base-dev libatlas-dev
```

**CentOS** You can install ATLAS on CentOS using:

```text
sudo yum install atlas-devel
```

**OS X**

Installing ATLAS on OS X is a somewhat complicated and lengthy process. However, the following commands will work on most machines:

```text
wget --content-disposition https://sourceforge.net/projects/math-atlas/files/latest/download?source=files
tar jxf atlas*.tar.bz2
mkdir atlas (Creating a directory for ATLAS)
mv ATLAS atlas/src-3.10.1
cd atlas/src-3.10.1
wget http://www.netlib.org/lapack/lapack-3.5.0.tgz (It may be possible that the atlas download already contains this file in which case this command is not needed)
mkdir intel(Creating a build directory)
cd intel
cpufreq-selector -g performance (This command requires root access. It is recommended but not essential)
../configure --prefix=/path to the directory where you want ATLAS installed/ --shared --with-netlib-lapack-tarfile=../lapack-3.5.0.tgz
make
make check
make ptcheck
make time
make install
```

#### CUDA

**Linux & OS X**

Detailed instructions for installing GPU architectures such as CUDA can be found [here](../config/backends/).

**Windows**

The CUDA Backend has some additional requirements before it can be built:

* [CUDA SDK](https://developer.nvidia.com/cuda-downloads)
* [Visual Studio 2012 or 2013](https://www.visualstudio.com/en-us/news/vs2013-community-vs.aspx) \(Please note: Visual Studio 2015 is _NOT SUPPORTED_ by CUDA 7.5 and below\)

In order to build the CUDA backend you will have to setup some more environment variables first, by calling `vcvars64.bat`. But first, set the system environment variable `SET_FULL_PATH` to `true`, so all of the variables that `vcvars64.bat` sets up, are passed to the mingw shell.

1. Inside a normal cmd.exe command prompt, run `C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\amd64\vcvars64.bat`
2. Run `c:\msys64\mingw64_shell.bat` inside that
3. Change to your libnd4j folder
4. `./buildnativeoperations.sh -c cuda`

This builds the CUDA nd4j.dll.

#### IDE Requirements

If you are building Deeplearning4j through an IDE such as IntelliJ, you will need to install certain plugins to ensure your IDE renders code highlighting appropriately. You will need to install a plugin for Lombok:

* IntelliJ Lombok Plugin: [https://plugins.jetbrains.com/plugin/6317-lombok-plugin](https://plugins.jetbrains.com/plugin/6317-lombok-plugin)
* Eclipse Lombok Plugin: Follow instructions at [https://projectlombok.org/download.html](https://projectlombok.org/download.html)

If you want to work on ScalNet, the Scala API, or on certain modules such as the DL4J UI, you will need to ensure your IDE has Scala support installed and available to you.

#### Testing

Deeplearning4j uses a separate repository that contains all resources necessary for testing. This is to keep the central DL4J repository lightweight and avoid large blobs in the GIT history. To run the tests you need to install the test-resources from [https://github.com/KonduitAI/dl4j-test-resources](https://github.com/KonduitAI/dl4j-test-resources) \(~10gb\). If you don't care about history, do a shallow clone only with

```bash
git clone --depth 1 --branch master https://github.com/KonduitAI/dl4j-test-resources
cd dl4j-test-resources
mvn install
```

Tests will run **only** when `testresources` and a backend profile \(such as `test-nd4j-native`\) are selected

```bash
mvn clean test -P  testresources,test-nd4j-native
```

Running the tests will take a while. To run tests of just a single maven module you can add a module constraint with `-pl deeplearning4j-core` \(for details see [here](https://stackoverflow.com/questions/11869762/maven-run-only-single-test-in-multi-module-project)\)

## Installing the DL4J Stack

## OS X & Linux

### Checking ENV

Before running the DL4J stack build script, you must ensure certain environment variables are defined before running your build. These are outlined below depending on your architecture.

#### LIBND4J\_HOME

You will need to know the exact path of the directory where you are running the DL4J build script \(you are encouraged to use a clean empty directory\). Otherwise, your build will fail. Once you determine this path, add `/libnd4j` to the end of that path and export it to your local environment. This will look like:

```text
export LIBND4J_HOME=/home/user/directory/libnd4j
```

#### CPU architecture w/ MKL

You can link with MKL either at build time, or at runtime with binaries initially linked with another BLAS implementation such as OpenBLAS. To build against MKL, simply add the path containing `libmkl_rt.so` \(or `mkl_rt.dll` on Windows\), say `/path/to/intel64/lib/`, to the `LD_LIBRARY_PATH` environment variable on Linux \(or `PATH` on Windows\) and build like before. On Linux though, to make sure it uses the correct version of OpenMP, we also might need to set these environment variables:

```bash
export MKL_THREADING_LAYER=GNU
export LD_PRELOAD=/lib64/libgomp.so.1
```

When libnd4j cannot be rebuilt, we can use the MKL libraries after the facts and get them loaded instead of OpenBLAS at runtime, but things are a bit trickier. Please additionally follow the instructions below.

1. Make sure that files such as `/lib64/libopenblas.so.0` and `/lib64/libblas.so.3` are not available \(or appear after in the `PATH` on Windows\), or they will get loaded by libnd4j by their absolute paths, before anything else.
2. Inside `/path/to/intel64/lib/`, create a symbolic link or copy of `libmkl_rt.so` \(or `mkl_rt.dll` on Windows\) to the name that libnd4j expect to load, for example:

```bash
ln -s libmkl_rt.so libopenblas.so.0
ln -s libmkl_rt.so libblas.so.3
```

```text
copy mkl_rt.dll libopenblas.dll
copy mkl_rt.dll libblas3.dll
```

1. Finally, add `/path/to/intel64/lib/` to the `LD_LIBRARY_PATH` environment variable \(or early in the `PATH` on Windows\) and run your Java application as usual.

### Building Manually

If you prefer, you can build each piece in the DL4J stack by hand. The procedure for each piece of software is essentially:

1. Git clone
2. Build
3. Install

The overall procedure looks like the following commands below, with the exception that libnd4j's `./buildnativeoperations.sh` accepts parameters based on the backend you are building for. You need to follow these instructions in the order they're given. If you don't, you'll run into errors. The GPU-specific instructions below have been commented out, but should be substituted for the CPU-specific commands when building for a GPU backend.

```text
# removes any existing repositories to ensure a clean build
rm -rf libnd4j
rm -rf nd4j
rm -rf datavec
rm -rf deeplearning4j

# compile libnd4j
git clone https://github.com/eclipse/deeplearning4j.git
cd libnd4j
./buildnativeoperations.sh
# and/or when using GPU
# ./buildnativeoperations.sh -c cuda -cc INSERT_YOUR_DEVICE_ARCH_HERE 
# i.e. if you have GTX 1070 device, use -cc 61
export LIBND4J_HOME=`pwd`
cd ..

# build and install nd4j to maven locally
git clone https://github.com/eclipse/deeplearning4j.git
cd nd4j
# cross-build across Scala versions (recommended)
bash buildmultiplescalaversions.sh clean install -DskipTests -Dmaven.javadoc.skip=true -pl '!:nd4j-cuda-9.0,!:nd4j-cuda-9.0-platform,!:nd4j-tests'
# or build for a single scala version
# mvn clean install -DskipTests -Dmaven.javadoc.skip=true -pl '!:nd4j-cuda-9.0,!:nd4j-cuda-9.0-platform,!:nd4j-tests'
# or when using GPU
# mvn clean install -DskipTests -Dmaven.javadoc.skip=true -pl '!:nd4j-tests'
cd ..

# build and install datavec
git clone https://github.com/eclipse/deeplearning4j.git
cd datavec
if [ "$SCALAV" == "" ]; then
  bash buildmultiplescalaversions.sh clean install -DskipTests -Dmaven.javadoc.skip=true
else
  mvn clean install -DskipTests -Dmaven.javadoc.skip=true -Dscala.binary.version=$SCALAV -Dscala.version=$SCALA
fi
cd ..

# build and install deeplearning4j
git clone https://github.com/eclipse/deeplearning4j.git
cd deeplearning4j
# cross-build across Scala versions (recommended)
./buildmultiplescalaversions.sh clean install -DskipTests -Dmaven.javadoc.skip=true
# or build for a single scala version
# mvn clean install -DskipTests -Dmaven.javadoc.skip=true
# If you skipped CUDA you may need to add
# -pl '!./deeplearning4j-cuda/'
# to the mvn clean install command to prevent the build from looking for cuda libs
cd ..
```

## Using Local Dependencies

Once you've installed the DL4J stack to your local maven repository, you can now include it in your build tool's dependencies. Follow the typical [Getting Started](quickstart/) instructions for Deeplearning4j, and appropriately replace versions with the SNAPSHOT version currently on the [master POM](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/pom.xml).

Note that some build tools such as Gradle and SBT don't properly pull in platform-specific binaries. You can follow instructions [here](../config/buildtools.md) for setting up your favorite build tool.

## Support and Assistance

If you encounter issues while building locally, please reach out on our [discourse](https://community.konduit.ai/c/dl4j) for help.


# Overview of working with libnd4j

Native operations for nd4j. Build using cmake

## Prerequisites

* GCC 4.9+
* CUDA Toolkit Versions 10 or 11
* CMake 3.8 (as of Nov 2017, in near future will require 3.9)

### Additional build arguments

There's few additional arguments for `buildnativeoperations.sh` script you could use:

```bash
 -a XXXXXXXX// shortcut for -march/-mtune, i.e. -a native
 -b release OR -b debug // enables/desables debug builds. release is considered by default
 -j XX // this argument defines how many threads will be used to binaries on your box. i.e. -j 8 
 -cc XX// CUDA-only argument, builds only binaries for target GPU architecture. use this for fast builds
 --check-vectorization  auto-vectorization report for developers. (Currently, only GCC is supported)
```

[More about AutoVectorization report](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/libnd4j/reference/auto\_vectorization/AutoVectorization.md)

You can provide the compute capability for your card [on the NVIDIA website here](https://developer.nvidia.com/cuda-gpus) or use auto.\
Please also check your Cuda Toolkit Release notes for supported and dropped features.\
Here is [the latest CUDA Toolkit Release note](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html#deprecated-features).\
You can find the same information for the older Toolkit versions [in the CUDA archives](https://docs.nvidia.com/cuda/archive/).

| -cc and --compute option examples | description                                                                      |
| --------------------------------- | -------------------------------------------------------------------------------- |
| -cc all                           | builds for common GPUs                                                           |
| -cc auto                          | tries to detect automatically                                                    |
| -cc Maxwell                       | GPU microarchitecture codename                                                   |
| -cc 75                            | compute capability 7.5 without a dot                                             |
| -cc 7.5                           | compute capability 7.5 with a dot                                                |
| -cc "Maxwell 6.0 7.5"             | space-separated multiple arguments within quotes (note: numbers only with a dot) |

## OS Specific Requirements

### Android

[Download the NDK](https://developer.android.com/ndk/downloads/), extract it somewhere, and execute the following commands, replacing `android-xxx` with either `android-arm` or `android-x86`:

```bash
git clone https://github.com/eclipse/deeplearning4j
export ANDROID_NDK=/path/to/android-ndk/
cd deeplearning4j/libnd4j
bash buildnativeoperations.sh -platform android-xxx
cd ../nd4j
mvn clean install -Djavacpp.platform=android-xxx -DskipTests -pl '!:nd4j-cuda-9.0,!:nd4j-cuda-9.0-platform,!:nd4j-tests'
```

### OSX

Run ./setuposx.sh (Please ensure you have brew installed)

See [macOSx10 CPU only.md](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/libnd4j/reference/macOSx10%20\(CPU%20only\).md)

### Linux

Depends on the distro - ask in the earlyadopters channel for specifics on distro

#### Ubuntu Linux 15.10

```bash
wget http://developer.download.nvidia.com/compute/cuda/7.5/Prod/local_installers/cuda-repo-ubuntu1504-7-5-local_7.5-18_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1504-7-5-local_7.5-18_amd64.deb
sudo apt-get update
sudo apt-get install cuda
sudo apt-get install cmake
sudo apt-get install gcc-4.9
sudo apt-get install g++-4.9
sudo apt-get install git
git clone https://github.com/deeplearning4j/libnd4j
cd libnd4j/
export LIBND4J_HOME=~/libnd4j/
sudo rm /usr/bin/gcc
sudo rm /usr/bin/g++
sudo ln -s /usr/bin/gcc-4.9 /usr/bin/gcc
sudo ln -s /usr/bin/g++-4.9 /usr/bin/g++
./buildnativeoperations.sh
./buildnativeoperations.sh -c cuda -сс YOUR_DEVICE_ARCH
```

#### Ubuntu Linux 16.04

```bash
sudo apt install cmake
sudo apt install nvidia-cuda-dev nvidia-cuda-toolkit nvidia-361
export TRICK_NVCC=YES
./buildnativeoperations.sh
./buildnativeoperations.sh -c cuda -сс YOUR_DEVICE_ARCH
```

The standard development headers are needed.

#### CentOS 6

```bash
yum install centos-release-scl-rh epel-release
yum install devtoolset-3-toolchain maven30 cmake3 git
scl enable devtoolset-3 maven30 bash
./buildnativeoperations.sh
./buildnativeoperations.sh -c cuda -сс YOUR_DEVICE_ARCH
```

### Windows

See [Windows.md](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/libnd4j/reference/windows.md)

## Setup for All OS

1. Set a LIBND4J\_HOME as an environment variable to the libnd4j folder you've obtained from GIT
   * Note: this is required for building nd4j as well.
2. Setup cpu followed by gpu, run the following on the command line:
   *   For standard builds:

       ```bash
        ./buildnativeoperations.sh
        ./buildnativeoperations.sh -c cuda -сс YOUR_DEVICE_ARCH
       ```
   *   For Debug builds:

       ```bash
        ./buildnativeoperations.sh blas -b debug
        ./buildnativeoperations.sh blas -c cuda -сс YOUR_DEVICE_ARCH -b debug
       ```
   *   For release builds (default):

       ```bash
        ./buildnativeoperations.sh
        ./buildnativeoperations.sh -c cuda -сс YOUR_DEVICE_ARCH
       ```

## OpenMP support

OpenMP 4.0+ should be used to compile libnd4j. However, this shouldn't be any trouble, since OpenMP 4 was released in 2015 and should be available on all major platforms.

## Linking with MKL

We can link with MKL either at build time, or at runtime with binaries initially linked with another BLAS implementation such as OpenBLAS. In either case, simply add the path containing `libmkl_rt.so` (or `mkl_rt.dll` on Windows), say `/path/to/intel64/lib/`, to the `LD_LIBRARY_PATH` environment variable on Linux (or `PATH` on Windows), and build or run your Java application as usual. If you get an error message like `undefined symbol: omp_get_num_procs`, it probably means that `libiomp5.so`, `libiomp5.dylib`, or `libiomp5md.dll` is not present on your system. In that case though, it is still possible to use the GNU version of OpenMP by setting these environment variables on Linux, for example:

```bash
export MKL_THREADING_LAYER=GNU
export LD_PRELOAD=/usr/lib64/libgomp.so.1
```

## Troubleshooting MKL

Sometimes the above steps might not be all you need to do. Another additional step might be the need to add:

```bash
export LD_LIBRARY_PATH=/opt/intel/lib/intel64/:/opt/intel/mkl/lib/intel64
```

This ensures that mkl will be found first and liked to.

## Packaging

If on Ubuntu (14.04 or above) or CentOS (6 or above), this repository is also set to create packages for your distribution. Let's assume you have built:

* for the cpu, your command-line was `./buildnativeoperations.sh ...`:

```bash
cd blasbuild/cpu
make package
```

* for the gpu, your command-line was `./buildnativeoperations.sh -c cuda ...`:

```bash
cd blasbuild/cuda
make package
```

## Uploading package to Bintray

The package upload script is in packaging. The upload command for an rpm built for cpu is:

```bash
./packages/push_to_bintray.sh myAPIUser myAPIKey deeplearning4j blasbuild/cpu/libnd4j-0.8.0.fc7.3.1611.x86_64.rpm https://github.com/deeplearning4j
```

The upload command for a deb package built for cuda is:

```bash
./packages/push_to_bintray.sh myAPIUser myAPIKey deeplearning4j blasbuild/cuda/libnd4j-0.8.0.fc7.3.1611.x86_64.deb https://github.com/deeplearning4j
```

## Running tests

Tests are written with [gtest](https://github.com/google/googletest), run using cmake. Tests are currently under tests\_cpu/

There are 2 directories for running tests:

1. libnd4j\_tests: These are older legacy ops tests.
2. layers\_tests: This covers the newer graph operations and ops associated with samediff.

For running the tests, we currently use cmake or CLion to run the tests.

To run tests using CUDA backend it's pretty much similar process:

1. ./buildnativeoperations.h -c cuda -cc  -b debug -t -j&#x20;
2. ./blasbuild/cuda/tests\_cpu/layers\_tests/runtests (.exe on Windows)

## Development

In order to extend and update libnd4j, understanding libnd4j's various cmake flags is the key. Many of them are in buildnativeoperations.sh. The pom.xml is used to integrate and auto configure the project for building with deeplearning4j.

At a minimum, you will want to enable tests. An example default set of flags for running tests and getting cpu builds working is as follows:

```bash
-DSD_CPU=true -DBLAS=TRUE -DSD_ARCH=x86-64 -DSD_EXTENSION= -DSD_LIBRARY_NAME=nd4jcpu -DSD_CHECK_VECTORIZATION=OFF  -DSD_SHARED_LIB=ON -DSD_STATIC_LIB=OFF -DSD_BUILD_MINIFIER=false -DSD_ALL_OPS=true -DCMAKE_BUILD_TYPE=Release -DPACKAGING=none  -DSD_BUILD_TESTS=OFF -DCOMPUTE=all -DOPENBLAS_PATH=C:/Users/agibs/.javacpp/cache/openblas-0.3.10-1.5.4-windows-x86_64.jar/org/bytedeco/openblas/windows-x86_64 -DDEV=FALSE -DCMAKE_NEED_RESPONSE=YES -DMKL_MULTI_THREADED=TRUE -DSD_BUILD_TESTS=YES
```

The way the main build script works, it dynamically generates a set of flags suitable for use for building the projects. Understanding the build script will go a long way in to configuring cmake for your particular IDE.

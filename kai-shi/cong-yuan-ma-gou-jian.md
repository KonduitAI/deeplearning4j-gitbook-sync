---
description: 从源代码构建所有DL4J库的说明。
---

# 从源码构建

### 在本地从主干构建

**注意：大多数用户应该使用Maven Central上的快速入门指南，而不是从源代码构建。**

_除非你有一个非常好的从源码构建的理由（例如开发新的特性——不包括自定义层、自定义激活函数、自定义丢失函数等——所有这些都可以在不直接修改DL4J的情况下添加），否则不应该从源码构建。从源码上构建可能相当复杂，在很多情况下都没有好处。_

对于那些喜欢使用最新版本的Deeplearning4j或新建分支并构建自己的版本的开发人员和工程师，这些说明将指导你构建和安装Deeplearning4j。首选安装目的地是你本地机器的Maven仓库。如果不使用主分支，可以根据需要修改这些步骤（即：切换GIT分支并修改build-dl4j-stack.sh脚本）。

本地构建需要构建完整的Deeplearning4j栈，包括：

* [libnd4j](https://github.com/deeplearning4j/libnd4j)
* [nd4j](https://github.com/deeplearning4j/nd4j)
* [datavec](https://github.com/deeplearning4j/datavec)
* [deeplearning4j](https://github.com/deeplearning4j/deeplearning4j)

请注意，Deeplearning4j被设计用于大多数平台（Windows、OS X和Linux），并且还包括多种“风格”，取决于你选择使用的计算架构。这包括CPU（OpenBLAS，MKL，ATLAS）和GPU（CUDA）。DL4J堆栈还支持X86和PowerPC架构。

### 先决条件

在尝试构建和安装DL4J栈之前，本地机器将需要设置一些基本的软件和环境变量。根据你的平台和操作系统的版本，在使它们工作时指令可能有所不同。该软件包括：

* git
* cmake \(3.2或更高版本\)
* OpenMP
* gcc \(4.9 或更高版本\)
* maven \(3.3 或更高版本\)

架构专用软件包括:

CPU 选项:

* Intel MKL
* OpenBLAS
* ATLAS

GPU 选项:

* CUDA

IDE-指定要求:

* IntelliJ Lombok plugin

DL4J 测试依赖:

* dl4j-test-resources

### 安装必备工具

#### Linux

**Ubuntu**假设你使用Ubuntu作为你的Linux偏好，并且作为非root用户运行，请按照以下步骤安装必备软件：

```bash
sudo apt-get purge maven maven2 maven3
sudo add-apt-repository ppa:natecarlson/maven3
sudo apt-get update
sudo apt-get install maven build-essentials cmake libgomp1
```

#### OS X

Homebrew是安装必备软件的公认方法。假设你在本地安装了Homebrew，请按照以下步骤安装必要的工具。

首先，在使用Homebrew之前，我们需要确保安装一个最新版本的XCODE（它被用作主编译器）：

```bash
xcode-select --install
```

最后, 安装必备工具:

```bash
brew update
brew install maven gcc5
```

注意：你不能使用CLANG。你也不使能用GCC的新版本。如果你有一个更新版本的GCC，请用这个[这个链接](https://apple.stackexchange.com/questions/190684/homebrew-how-to-switch-between-gcc-versions-gcc49-and-gcc)切换版本。

#### Windows

libnd4j 依赖 Unix 工具包来编译。 所以为了编译它你需要安装[Msys2](https://msys2.github.io/) 。

按[他们的说明](https://msys2.github.io/)安装了MyS2之后，你将不得不安装一些额外的开发包。启动msys2 shell并设置DEV环境：

```bash
pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-cmake mingw-w64-x86_64-extra-cmake-modules make pkg-config grep sed gzip tar mingw64/mingw-w64-x86_64-openblas
```

这将在MyS2 shell中安装所需使用的依赖项。

你还需要设置PATH环境变量来包括`C:\msys64\mingw64\bin`（或者你决定安装msys2的任何位置）。如果打开了IntelliJ（或其他IDE），则必须在此更改对通过它们启动的应用程序生效之前重新启动它。如果不这样做，你可能会看到一个“Can’t find dependent libraries”的错误。

### 安装必备架构

一旦安装了必备工具，现在就可以为平台安装所需的架构。

#### 英特尔 MKL

在所有现有的CPU架构中，英特尔MKL是目前最快的。但是，在实际安装之前，它需要一些“开销”。

1. [英特尔网站](https://software.intel.com/en-us/intel-mkl)申请一个许可
2. 通过英特尔网站执行几步之后，你将收到下载链接。
3. 下载MKL并按[安装向导](https://software.intel.com/en-us/articles/intel-math-kernel-library-intel-mkl-2020-install-guide)进行安装

#### OpenBLAS

#### Linux

**Ubuntu** 假设你正在使用Ubuntu，你可以通过如下安装 OpenBLAS :

```bash
sudo apt-get install libopenblas-dev
```

你还需要确保`/opt/OpenBLAS/lib`（或OpenBLAS的任何其他主目录）在你的路径上。为了让OpenBLAS与Apache Spark一起工作，你还需要做如下操作：

```bash
sudo cp libopenblas.so liblapack.so.3
sudo cp libopenblas.so libblas.so.3
```

**CentOS** 用root用户在终端（或ssh会话）输入如下:

```bash
yum groupinstall 'Development Tools'
```

之后，你 应该在终端上看到很多活动和安装。为了验证你有，例如，GCC，请输入此行：

```bash
gcc --version
```

[去这里](http://www.cyberciti.biz/faq/centos-linux-install-gcc-c-c-compiler/)获得更多完整说明。

#### OS X

你可以在OS X用Homebrew安装OpenBLAS:

```bash
brew install homebrew/science/openblas
```

#### Windows

`msys2 的`OpenBLAS包已经可用，你可以用`pacman命令安装。`

#### ATLAS

#### Linux

**Ubuntu** 在Ubuntu上有一个ATLAS的apt包可以用:

```bash
sudo apt-get install libatlas-base-dev libatlas-dev
```

**CentOS** 你可以在 CentOS 上按如下按装ATLAS:

```bash
sudo yum install atlas-devel
```

#### OS X

在OS X上安装ATLAS是一个有点复杂和漫长的过程。但是，下面的命令将适用于大多数机器：

```bash
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

#### Linux & OS X

在[这里](https://deeplearning4j.org/docs/latest/deeplearning4j-config-gpu-cpu)可以找到安装GPU架构（如CUDA）的详细说明。

#### Windows

CUDA后端在构建之前有一些附加的要求：

* [CUDA SDK](https://developer.nvidia.com/cuda-downloads)
* [Visual Studio 2012 or 2013](https://www.visualstudio.com/en-us/news/vs2013-community-vs.aspx) \(Please note请注意: CUDA 7.5及以下版本不支持Visual Studio 2015\)

为了构建CUDA后端，你必须首先调用`vcvars64.bat`来设置更多的环境变量。但首先，设置系统环境变量 `SET_FULL_PATH`为`true`，所以所有`vcvars64.bat设置`的变量都传递到mingw shell。

1. 在正常的cmd.exe命令提示符内，运行`C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\amd64\vcvars64.bat`
2. 在理面运行 `c:\msys64\mingw64_shell.bat` 
3. 切换到 libnd4j 目录 
4. `./buildnativeoperations.sh -c cuda`

这将构建CUDA nd4j.dll.

#### IDE 要求

如果你是通过一个IDE如IntelliJ构建Deeplearning4j，你将需要安装一些插件，以确保你的IDE使得代码高亮适当。你需要为Lombok安装一个插件：

* IntelliJ Lombok 插件: [https://plugins.jetbrains.com/plugin/6317-lombok-plugin](https://plugins.jetbrains.com/plugin/6317-lombok-plugin)
* Eclipse Lombok 插件: Follow instructions at [https://projectlombok.org/download.html](https://projectlombok.org/download.html)

如果你想在ScalNet、Scala API或某些模块（如DL4J UI）上工作，则需要确保IDE安装了Scala支持，并且该支持可用。

#### 测试

Deeplearning4j使用一个单独的存储库，包含测试所需的所有资源。这是为了保持中央DL4J存储库轻量级，避免Git历史中的大blobs。为了运行测试，你需要从[https://github.com/deeplearning4j/dl4j-test-resources](https://github.com/KonduitAI/dl4j-test-resources)（约10GB）安装测试资源。如果你不关心历史，只做一个浅克隆。

```text
git clone --depth 1 --branch master https://github.com/deeplearning4j/dl4j-test-resources
cd dl4j-test-resources
mvn install
```

测试将只在`testresources`和后端profile（例如，test-nd4j-native）被选择时运行。

```text
mvn clean test -P  testresources,test-nd4j-native
```

\[图片上传中...\(image-13982b-1540534628348-8\)\]

运行测试需要一段时间。要运行一个单独的Maven模块的测试，可以添加一个模块约束，用`-pl deeplearning4j-core（`详情见[此处](https://stackoverflow.com/questions/11869762/maven-run-only-single-test-in-multi-module-project)）。

## 安装DL4J栈

### OS X & Linux

#### 检查环境

在运行DL4J栈构建脚本之前，必须确保在运行构建之前定义了某些环境变量。下面将根据你的架构进行概述。

#### LIBND4J\_HOME

你将需要知道运行DL4J构建脚本的目录的确切路径（鼓励你使用干净的空目录）。否则，你的构建将失败。一旦确定了这条路径，就将`/libnd4j`添加到该路径的末端，并将其导出到本地环境中。这看起来像：

```text
export LIBND4J_HOME=/home/user/directory/libnd4j
```

#### CPU 架构 w/ MKL

你可以在构建时连接MKL或是在运行时连接一个二进制文件，初始化时连接另外的BLAS实现，例如OpenBLAS。要根据MKL进行构建，只需将包含`libmkl_rt.so`（或Windows上的`mkl_rt.dll`）的路径（例如`/path/to/intel64/lib/`）添加到Linux上的`LD_LIBRARY_PATH`环境变量（或Windows上的PATH）中，然后像以前一样构建。在Linux上，为了确保它使用OpenMP的正确版本，我们也可能需要设置这些环境变量：

```text
export MKL_THREADING_LAYER=GNU
export LD_PRELOAD=/lib64/libgomp.so.1
```

当libnd4j无法重建时，我们可以在事实之后使用MKL库，并在运行时加载它们，而不是OpenBLAS，但是事情有点棘手。请另外按照下面的说明进行操作。

1. 确保这些文件例如 `/lib64/libopenblas.so.0` 和 `/lib64/libblas.so.3` 都不可用 \(或出现在Windows的`PATH参数之后`\), 或者他们将被LBND4J通过他们的绝对路径加载，在其他事情之前。
2. 在`/path/to/intel64/lib/`内部，创建`libmkl_rt.so`（或Windows上的 `mkl_rt.dll`）到libnd4j需要加载的名称的符号链接或副本，例如：

```text
ln -s libmkl_rt.so libopenblas.so.0
ln -s libmkl_rt.so libblas.so.3
```

```text
copy mkl_rt.dll libopenblas.dll
copy mkl_rt.dll libblas3.dll
```

1. 最后，将`/path/to/intel64/lib/` 添加到`LD_LIBRARY_PATH`环境变量（或者在Windows上的`PATH`路径），并像往常一样运行Java应用程序。

#### 

#### 手动构建

如果你愿意，你可以手动在DL4J栈中构建每一块。每个软件块的过程基本上是：

1. Git clone
2. Build
3. Install

整个过程如下面的命令所示，但libnd4j’s `./buildnativeoperations.sh`接受基于你正在构建的后端的参数。你需要按照他们给出的顺序来执行这些指令。如果你不这样做，你的执行就会出错误。下面的特定于GPU的指令已经被注释掉了，但是在构建GPU后端时，应该取代特定于CPU的命令。

```text
# removes any existing repositories to ensure a clean build
rm -rf libnd4j
rm -rf nd4j
rm -rf datavec
rm -rf deeplearning4j

# compile libnd4j
git clone https://github.com/deeplearning4j/libnd4j.git
cd libnd4j
./buildnativeoperations.sh
# and/or when using GPU
# ./buildnativeoperations.sh -c cuda -cc INSERT_YOUR_DEVICE_ARCH_HERE 
# i.e. if you have GTX 1070 device, use -cc 61
export LIBND4J_HOME=`pwd`
cd ..

# build and install nd4j to maven locally
git clone https://github.com/deeplearning4j/nd4j.git
cd nd4j
# cross-build across Scala versions (recommended)
bash buildmultiplescalaversions.sh clean install -DskipTests -Dmaven.javadoc.skip=true -pl '!:nd4j-cuda-9.0,!:nd4j-cuda-9.0-platform,!:nd4j-tests'
# or build for a single scala version
# mvn clean install -DskipTests -Dmaven.javadoc.skip=true -pl '!:nd4j-cuda-9.0,!:nd4j-cuda-9.0-platform,!:nd4j-tests'
# or when using GPU
# mvn clean install -DskipTests -Dmaven.javadoc.skip=true -pl '!:nd4j-tests'
cd ..

# build and install datavec
git clone https://github.com/deeplearning4j/datavec.git
cd datavec
if [ "$SCALAV" == "" ]; then
  bash buildmultiplescalaversions.sh clean install -DskipTests -Dmaven.javadoc.skip=true
else
  mvn clean install -DskipTests -Dmaven.javadoc.skip=true -Dscala.binary.version=$SCALAV -Dscala.version=$SCALA
fi
cd ..

# build and install deeplearning4j
git clone https://github.com/deeplearning4j/deeplearning4j.git
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

### 使用本地依赖

一旦你将DL4J栈安装到本地maven存储库之后，现在可以将其引入在构建工具的依赖项中。按照Deeplearning4j的[经典入门](kuai-su-ru-men.md)说明，在[主 POM](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/pom.xml)上用SNAPSHOT版本适当地替换当前使用的版本。

请注意，一些构建工具，如Gradle和SBT没有正确地引入平台特定的二进制文件。你可以按照[这里](http://nd4j.org/dependencies.html)的说明来设置你最喜欢的构建工具。

### 支持与援助

如果在本地构建时遇到问题，Deeplearning4j的[Early Adopters Channel](https://gitter.im/deeplearning4j/deeplearning4j/earlyadopters)是专门用于帮助解决构建问题和其他源代码问题的通道。请在Gitter上获取帮助。




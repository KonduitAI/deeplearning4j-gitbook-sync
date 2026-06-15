---
title: "Building from Source"
description: "How to build Deeplearning4j from source — prerequisites, Maven build, libnd4j compilation, and common build issues"
---

> **Most users should use the releases on Maven Central** and do not need to build from source. Building from source is intended for contributors, those testing unreleased features, or teams maintaining a custom fork. Custom layers, loss functions, and activation functions can all be added without modifying DL4J directly.

## Overview

The DL4J stack is a monorepo at [github.com/eclipse/deeplearning4j](https://github.com/eclipse/deeplearning4j). A full source build produces the following components:

- **libnd4j** — native C++ backend (compiled with CMake)
- **nd4j** — Java bindings and math backend
- **datavec** — data pipeline and ETL
- **deeplearning4j** — neural network training framework and utilities

## Prerequisites

### Required Tools

| Tool | Minimum Version | Notes |
|------|----------------|-------|
| JDK | 11 | JDK 17 recommended for M2.1 |
| Maven | 3.6.3 | Earlier 3.x releases have known issues with the monorepo |
| Git | 2.x | Any recent version |
| CMake | 3.9 | Required to compile libnd4j |
| gcc / g++ | 7.x | Clang is not supported on Linux |

### Platform-Specific Setup

#### Linux (Ubuntu / Debian)

```bash
sudo apt-get update
sudo apt-get install -y \
    build-essential cmake git \
    libgomp1 openjdk-17-jdk maven
```

Verify versions before proceeding:

```bash
java -version
mvn -version
cmake --version
gcc --version
```

#### Linux (CentOS / RHEL)

```bash
sudo yum groupinstall 'Development Tools'
sudo yum install cmake3 java-17-openjdk-devel maven
```

#### macOS

Install Xcode command-line tools and [Homebrew](https://brew.sh/), then:

```bash
xcode-select --install
brew update
brew install cmake maven openjdk@17
```

Note: The system `clang` is used on macOS for the JNI layer, but libnd4j's native C++ code requires GCC. Install GCC via Homebrew if needed:

```bash
brew install gcc
```

#### Windows

libnd4j requires a Unix-compatible toolchain. Install [MSYS2](https://www.msys2.org/) and then run inside the MSYS2 shell:

```bash
pacman -S \
    mingw-w64-x86_64-gcc \
    mingw-w64-x86_64-cmake \
    mingw-w64-x86_64-extra-cmake-modules \
    make pkg-config grep sed gzip tar \
    mingw64/mingw-w64-x86_64-openblas
```

Add `C:\msys64\mingw64\bin` to your system `PATH`. Restart any open IDE after doing so.

### CPU Math Libraries

Choose one of the following for CPU computation:

**OpenBLAS** (recommended, open source):
```bash
# Ubuntu/Debian
sudo apt-get install libopenblas-dev

# macOS
brew install openblas

# Windows (via MSYS2)
pacman -S mingw64/mingw-w64-x86_64-openblas
```

**Intel MKL** (best single-machine CPU performance):
Download from [Intel's developer site](https://www.intel.com/content/www/us/en/developer/tools/oneapi/onemkl.html). After installation, add the MKL library directory to `LD_LIBRARY_PATH` (Linux) or `PATH` (Windows).

To link MKL at runtime against an OpenBLAS-linked binary, create symbolic links:
```bash
# Linux example
ln -s libmkl_rt.so libopenblas.so.0
ln -s libmkl_rt.so libblas.so.3
export MKL_THREADING_LAYER=GNU
export LD_PRELOAD=/lib64/libgomp.so.1
```

## Cloning the Repository

```bash
git clone https://github.com/eclipse/deeplearning4j.git
cd deeplearning4j
```

To work on a specific release tag (for example, the M2.1 tag):

```bash
git checkout deeplearning4j-2.1.0
```

## Building libnd4j (Native Backend)

libnd4j is the C++ compute engine. It must be compiled before the Java modules can be built.

```bash
cd libnd4j
./buildnativeoperations.sh
```

After compilation, export the path so Maven can locate the native libraries:

```bash
export LIBND4J_HOME=$(pwd)
cd ..
```

### Common libnd4j Build Flags

| Flag | Description |
|------|-------------|
| `-c cuda` | Build with CUDA GPU support |
| `-cc <arch>` | Target a specific CUDA compute capability (e.g., `-cc 86` for Ampere) |
| `-j <n>` | Use `n` parallel compiler jobs |
| `--build-type Release` | Release build (default); use `Debug` for debugging |

Example for CUDA with compute capability 8.6 (RTX 30xx):
```bash
./buildnativeoperations.sh -c cuda -cc 86 -j 8
```

## Building with CUDA

### Prerequisites

- NVIDIA CUDA Toolkit 11.x or 12.x (M2.1 supports both)
- A supported NVIDIA GPU driver
- On Windows: Visual Studio 2019 or 2022 (Community edition is sufficient)

Install CUDA from [developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads).

### Linux / macOS Build

```bash
cd libnd4j
./buildnativeoperations.sh -c cuda -cc INSERT_YOUR_DEVICE_ARCH
export LIBND4J_HOME=$(pwd)
cd ..

mvn clean install -DskipTests -Dmaven.javadoc.skip=true
```

Find your GPU's compute capability at [developer.nvidia.com/cuda-gpus](https://developer.nvidia.com/cuda-gpus). For example, an RTX 4090 uses `-cc 89`.

### Windows Build

1. Open a standard `cmd.exe` prompt and run:
   ```
   "C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat"
   ```
2. From that same prompt, launch the MSYS2 shell:
   ```
   c:\msys64\mingw64_shell.bat
   ```
3. Inside the MSYS2 shell:
   ```bash
   cd /path/to/deeplearning4j/libnd4j
   ./buildnativeoperations.sh -c cuda -cc INSERT_YOUR_DEVICE_ARCH
   ```

## Full Maven Build

After libnd4j is built and `LIBND4J_HOME` is set, build all Java modules from the repo root:

```bash
mvn clean install -DskipTests -Dmaven.javadoc.skip=true
```

This installs all artifacts to your local Maven repository (`~/.m2/repository`). A first build typically takes 15–45 minutes depending on hardware.

### Building Specific Modules

Maven's `-pl` flag restricts the build to specific submodules:

```bash
# Build only nd4j
mvn clean install -DskipTests -pl nd4j/nd4j-backends/nd4j-backend-impls/nd4j-native

# Build only the core DL4J training module
mvn clean install -DskipTests -pl deeplearning4j/deeplearning4j-core

# Exclude CUDA modules when building without a GPU
mvn clean install -DskipTests -pl '!deeplearning4j/deeplearning4j-cuda'
```

Use the `-am` flag (`--also-make`) to include transitive dependencies of a module:

```bash
mvn clean install -DskipTests -pl deeplearning4j/deeplearning4j-core -am
```

### Using the Build Script

A convenience script is provided for building the entire stack:

```bash
# CPU build
./build-dl4j-stack.sh

# CUDA build
./build-dl4j-stack.sh -c cuda

# CUDA build targeting a specific compute capability
./build-dl4j-stack.sh -c cuda -cc 86
```

## Running Tests

DL4J tests depend on a separate test-resources repository (~10 GB). Clone it with a shallow fetch if history is not needed:

```bash
git clone --depth 1 --branch master \
    https://github.com/deeplearning4j/dl4j-test-resources
cd dl4j-test-resources
mvn install
```

Run tests against the native CPU backend:

```bash
mvn clean test -P testresources,test-nd4j-native
```

Run tests for a single module:

```bash
mvn clean test -P testresources,test-nd4j-native \
    -pl deeplearning4j/deeplearning4j-core
```

## IDE Setup: IntelliJ IDEA

IntelliJ IDEA is the recommended IDE.

1. **Clone** the repository and **open** it as a Maven project (File > Open, select `pom.xml` at the repo root).
2. **Install the Lombok plugin**: Settings > Plugins > search "Lombok" > Install. Without it, the IDE will show false errors throughout the codebase.
3. **Enable annotation processing**: Settings > Build, Execution, Deployment > Compiler > Annotation Processors > check "Enable annotation processing".
4. **Import Maven profiles** as needed for your backend (e.g., `native` or `cuda`).
5. If working on the Scala API or the DL4J UI, install the Scala plugin.

For large monorepo projects, increase IntelliJ's JVM heap:
Help > Change Memory Settings > set 4096 MB or higher.

## Using Local Build Artifacts

After a successful build, use the local snapshot in a downstream project by matching the version in your `pom.xml`:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-core</artifactId>
    <version>2.1.0-SNAPSHOT</version>
</dependency>
```

Check the current snapshot version in the [root POM](https://github.com/eclipse/deeplearning4j/blob/master/pom.xml).

## Common Build Issues

### `LIBND4J_HOME` not set

**Symptom:** Maven build fails with errors about missing native libraries or JNI classes.

**Fix:** Export the variable pointing to the compiled libnd4j directory:
```bash
export LIBND4J_HOME=/path/to/deeplearning4j/libnd4j
```

### CMake version too old

**Symptom:** `cmake: command not found` or CMake policy errors.

**Fix:** Install CMake 3.9 or newer. On older Ubuntu/CentOS systems, install via `pip install cmake` or download from [cmake.org](https://cmake.org/download/).

### `clang` used instead of `gcc` on macOS

**Symptom:** Linker errors or missing OpenMP support.

**Fix:** Force GCC explicitly:
```bash
export CC=$(brew --prefix gcc)/bin/gcc-13
export CXX=$(brew --prefix gcc)/bin/g++-13
./buildnativeoperations.sh
```

### Out-of-memory during Maven build

**Symptom:** Build fails with `java.lang.OutOfMemoryError` or `GC overhead limit exceeded`.

**Fix:**
```bash
export MAVEN_OPTS="-Xmx4g -XX:MaxMetaspaceSize=1g"
```

### CUDA version mismatch

**Symptom:** Errors like `no kernel image is available for execution on the device`.

**Fix:** Ensure the `-cc` flag matches your GPU's actual compute capability. Check it with:
```bash
nvidia-smi --query-gpu=compute_cap --format=csv,noheader
```

### Windows: DLL not found at runtime

**Symptom:** `Can't find dependent libraries` when launching from an IDE.

**Fix:** Add `C:\msys64\mingw64\bin` to your system `PATH` and restart the IDE.

## Support

If you encounter build issues not covered here, please reach out on the [DL4J GitHub Discussions](https://github.com/eclipse/deeplearning4j/discussions) or the community [Gitter channel](https://gitter.im/deeplearning4j/deeplearning4j).

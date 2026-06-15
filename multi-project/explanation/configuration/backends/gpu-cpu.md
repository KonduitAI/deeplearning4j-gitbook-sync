---
title: "GPU and CPU Setup"
description: "Configuring GPU and CPU backends — CUDA setup, multi-GPU, CPU optimizations, and backend switching"
---

## Overview

DL4J delegates all numerical computation to ND4J (N-Dimensional Arrays for Java). ND4J supports two backends: a CPU backend (`nd4j-native`) that uses OpenBLAS and AVX-optimized C++ code, and a CUDA GPU backend (`nd4j-cuda-*`) that targets NVIDIA GPUs. You select the backend purely through your project dependencies — no code changes are needed to switch between CPU and GPU.

This page covers CPU backend setup, GPU backend setup, CUDA requirements, switching between backends, multi-GPU configuration, and how to verify which backend is active.

## CPU Backend Setup

### Maven

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native-platform</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

The `-platform` artifact includes native binaries for Linux x86_64, Linux ARM64, macOS x86_64, macOS ARM64 (Apple Silicon), and Windows x86_64. For a single-platform deployment, use `nd4j-native` with an explicit classifier (e.g., `linux-x86_64`) to reduce JAR size.

### Gradle

```groovy
implementation "org.nd4j:nd4j-native-platform:1.0.0-M2.1"
```

### What the CPU backend uses

The CPU backend links against OpenBLAS for BLAS operations (matrix multiply, etc.) and uses platform-optimized code paths for AVX2 and AVX-512 where the CPU supports them. Intel MKL is used when available and detected automatically. The startup log will report which BLAS vendor was selected:

```
o.n.l.a.o.e.DefaultOpExecutioner - Blas vendor: [MKL]
```
or
```
o.n.l.a.o.e.DefaultOpExecutioner - Blas vendor: [OPENBLAS]
```

## GPU Backend Setup

### Prerequisites

Before using the CUDA backend, ensure the following are installed on your system:

1. **NVIDIA GPU** with compute capability 3.5 or higher (Kepler or newer).
2. **CUDA Toolkit** matching the version in the ND4J artifact name. For `nd4j-cuda-11.6`, install CUDA 11.6 or a compatible 11.x release.
3. **NVIDIA Driver** compatible with the installed CUDA version. Consult the [NVIDIA CUDA release notes](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/) for driver version requirements.

To verify CUDA is installed:

```shell
nvcc --version
nvidia-smi
```

`nvidia-smi` shows the driver version and all detected GPUs.

### Maven — CUDA Backend

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.6-platform</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

### Gradle — CUDA Backend

```groovy
implementation "org.nd4j:nd4j-cuda-11.6-platform:1.0.0-M2.1"
```

**Important:** Do not include both `nd4j-native-platform` and `nd4j-cuda-*-platform` in the same project. ND4J will pick one backend at startup (typically the first one found on the classpath), which may not be the one you intend.

## Switching Between CPU and GPU

The cleanest approach is to use a Maven property (or Gradle variable) to select the backend:

### Maven

```xml
<properties>
    <!-- Change to nd4j-cuda-11.6-platform to use GPU -->
    <nd4j.backend>nd4j-native-platform</nd4j.backend>
    <dl4j.version>1.0.0-M2.1</dl4j.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.nd4j</groupId>
        <artifactId>${nd4j.backend}</artifactId>
        <version>${dl4j.version}</version>
    </dependency>
</dependencies>
```

You can then override on the command line without editing the file:

```shell
mvn package -Dnd4j.backend=nd4j-cuda-11.6-platform
```

### Gradle

```groovy
ext {
    nd4jBackend = project.findProperty('nd4jBackend') ?: 'nd4j-native-platform'
    dl4jVersion = '1.0.0-M2.1'
}

dependencies {
    implementation "org.nd4j:${nd4jBackend}:${dl4jVersion}"
}
```

Override at build time:

```shell
gradle build -Pnd4jBackend=nd4j-cuda-11.6-platform
```

## Verifying Which Backend Is Active

ND4J logs which backend it loads at startup. Look for these lines:

**CPU:**
```
o.n.l.f.Nd4jBackend       - Loaded [CpuBackend] backend
o.n.l.a.o.e.DefaultOpExecutioner - Backend used: [CPU]; OS: [Linux]
o.n.l.a.o.e.DefaultOpExecutioner - Cores: [8]; Memory: [31.3GB];
o.n.l.a.o.e.DefaultOpExecutioner - Blas vendor: [OPENBLAS]
```

**GPU:**
```
o.n.l.f.Nd4jBackend       - Loaded [JCublasBackend] backend
o.n.l.a.o.e.DefaultOpExecutioner - Backend used: [CUDA]; OS: [Linux]
o.n.l.a.o.e.DefaultOpExecutioner - Cores: [16]; Memory: [31.3GB];
o.n.l.a.o.e.DefaultOpExecutioner - Blas vendor: [CUBLAS]
o.n.l.a.o.e.DefaultOpExecutioner - Device Name: [NVIDIA GeForce RTX 3090]; CC: [8.6]; Total/free memory: [25769803776]
```

You can also check programmatically:

```java
System.out.println("Backend: " + Nd4j.getBackend().getClass().getName());
// CPU:  org.nd4j.linalg.cpu.nativecpu.CpuBackend
// GPU:  org.nd4j.linalg.jcublas.JCublasBackend
```

## Multi-GPU Configuration

If the host has multiple GPUs and CUDA is configured to expose only one, you can enable multi-GPU usage at the start of your `main()` method:

```java
CudaEnvironment.getInstance().getConfiguration().allowMultiGPU(true);
```

### ParallelWrapper for Data-Parallel Training

For training a single model across multiple GPUs using data parallelism, use `ParallelWrapper`:

```java
MultiLayerNetwork model = ...; // build or load your model

ParallelWrapper wrapper = new ParallelWrapper.Builder(model)
    // Number of prefetch DataSets per worker
    .prefetchBuffer(8)
    // One worker per GPU
    .workers(4)
    // Average gradients every N iterations (higher = faster, potentially less stable)
    .averagingFrequency(3)
    // Log score after each averaging step
    .reportScoreAfterAveraging(true)
    // ENABLED uses workspace-based memory management (default)
    .workspaceMode(WorkspaceMode.ENABLED)
    .build();

wrapper.fit(trainIterator);
```

Each worker thread gets its own GPU context. `workers` should be set equal to the number of physical GPUs available.

### Controlling Which GPU Is Used

To pin the process to a specific GPU device:

```java
// Use device 1 (second GPU) instead of device 0
CudaEnvironment.getInstance().getConfiguration().setDeviceLocalThread(1);
```

To query available devices:

```java
int numGpus = CudaEnvironment.getInstance().getConfiguration().getAvailableDevices().size();
System.out.println("Available GPUs: " + numGpus);
```

### Memory Management with Multiple GPUs

When using multiple GPUs, each device has its own memory pool. Total off-heap memory allocation is shared across all devices. To ensure each GPU has sufficient memory:

```shell
-Dorg.bytedeco.javacpp.maxbytes=16G   # total off-heap, shared across all GPUs
-Dorg.bytedeco.javacpp.maxphysicalbytes=20G
```

See the [Memory Configuration](./memory) page for detailed guidance.

## CPU Optimizations

### AVX Extensions

ND4J's CPU backend automatically uses the best available AVX instruction set (SSE4.2, AVX2, AVX-512) supported by the CPU. No manual configuration is needed. However, when comparing performance across machines, be aware that newer CPUs with AVX-512 support will significantly outperform older hardware.

To check which AVX level is active, look for log lines at startup or run:

```java
System.out.println(System.getProperty("os.arch"));
// Check CPU features on Linux:
// cat /proc/cpuinfo | grep flags | head -1
```

### OpenMP Threads

The CPU backend uses OpenMP for thread-level parallelism within each operation. By default it uses the number of physical CPU cores. If you are running multiple DL4J models or processes on the same machine, reduce the thread count to avoid resource contention:

```shell
# Set before launching the JVM
export OMP_NUM_THREADS=4
```

Or in Java (must be set before ND4J initializes):

```java
System.setProperty("OMP_NUM_THREADS", "4");
```

Setting `OMP_NUM_THREADS` lower than the core count is beneficial when running many concurrent inference threads (e.g., in a web server), where the total parallelism from multiple Java threads already saturates the CPU.

### Disabling Periodic GC During Training

With CPU backend and workspaces enabled, periodic GC calls add latency. Reduce or disable them:

```java
// Reduce GC frequency to every 10 seconds
Nd4j.getMemoryManager().setAutoGcWindow(10000);

// Or disable entirely (only safe when workspaces are enabled)
Nd4j.getMemoryManager().togglePeriodicGc(false);
```

## GPU Memory Management

GPU memory is managed via the off-heap JavaCPP allocator. The `-Dorg.bytedeco.javacpp.maxbytes` flag controls how much GPU memory ND4J may allocate.

Set off-heap to match or slightly exceed the GPU's VRAM capacity:

```shell
# For a GPU with 24 GB VRAM:
-Xms2G -Xmx4G -Dorg.bytedeco.javacpp.maxbytes=22G -Dorg.bytedeco.javacpp.maxphysicalbytes=28G
```

If the GPU OOMs during training, the first things to try are:
1. Reduce batch size.
2. Lower the `-Dorg.bytedeco.javacpp.maxbytes` value to leave room for other allocations.
3. Check that workspaces are enabled (`WorkspaceMode.ENABLED`) so memory is reused between iterations.

## Performance Comparison: CPU vs GPU

As a general guide:

| Workload | CPU | GPU |
|---|---|---|
| Small networks (<1M params) | Competitive | Overhead may dominate |
| CNNs on images | Slower | Significantly faster |
| Large RNNs/Transformers | Much slower | Strongly preferred |
| Inference, single sample | Competitive | Overhead per call |
| Batch inference | Slower | Faster with large batches |

GPUs shine with large batch sizes and computationally intensive layers (convolutions, attention). For low-latency single-sample inference, CPU is often faster due to the absence of GPU launch overhead.

## Related Pages

- [Maven Setup](./maven) — dependency declarations
- [cuDNN](./cudnn) — further GPU acceleration with cuDNN
- [Memory Configuration](./memory) — JVM and off-heap memory flags
- [Performance Debugging](./performance-debugging) — diagnosing backend and performance issues

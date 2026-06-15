---
title: "Backends Overview"
description: "How ND4J's backend system works — SPI mechanism, backend selection, and the relationship between nd4j-native and nd4j-cuda"
---

# ND4J Backends Overview

ND4J is a backend-agnostic numerical computing library. The `nd4j-api` module defines all public interfaces — `INDArray`, `Nd4j`, `DataBuffer`, ops — but contains no native execution code. Actual computation is provided by a separate backend JAR that is resolved at startup. This separation means you can switch between a CPU and a GPU implementation purely by swapping a Maven dependency, with no changes to your application code.

This page explains the architecture that makes this possible, how ND4J discovers and loads the backend, and the practical rules for choosing and placing backends on the classpath.


## Backend Architecture

The central abstraction is `org.nd4j.linalg.factory.Nd4jBackend`. Every backend provides a concrete subclass that:

1. Reports whether the backend is available in the current environment (e.g., checks that CUDA drivers are present for the GPU backend).
2. Returns a `DataBufferFactory` and an `NDArrayFactory` specific to that backend.
3. Specifies the memory model (on-heap vs. off-heap) and any configuration properties.

The two production backends in M2.1 are:

| Backend | Artifact | Hardware |
|---|---|---|
| `nd4j-native` | `org.nd4j:nd4j-native` | CPU (x86, ARM, PowerPC) |
| `nd4j-cuda` | `org.nd4j:nd4j-cuda-11.6` | NVIDIA GPU via CUDA 11.6 |

Both backends delegate to **libnd4j**, a C++ compute engine compiled with platform-specific optimizations. The Java layer is a thin JNI wrapper; nearly all arithmetic runs inside native code, which is why ND4J throughput is comparable to native Python frameworks like NumPy.


## SPI (Service Provider Interface) Mechanism

ND4J uses the standard Java SPI mechanism defined in `java.util.ServiceLoader`. When `Nd4j` is first referenced in your application, it calls `ServiceLoader.load(Nd4jBackend.class)` and iterates over all registered providers.

Each backend JAR ships a service registration file at:

```
META-INF/services/org.nd4j.linalg.factory.Nd4jBackend
```

The file contains one line: the fully-qualified class name of the backend implementation. For example, inside `nd4j-native`:

```
org.nd4j.linalg.cpu.nativecpu.CpuBackend
```

And inside `nd4j-cuda-11.6`:

```
org.nd4j.linalg.jcublas.JCublasBackend
```

When the JVM's class loader scans the classpath at startup, it picks up every `META-INF/services/org.nd4j.linalg.factory.Nd4jBackend` entry it finds and makes those implementations available to `ServiceLoader`.


## Backend Selection and Initialization

ND4J's initialization sequence, condensed:

1. `ServiceLoader.load(Nd4jBackend.class)` collects all available backend implementations found on the classpath.
2. Each discovered backend is polled with `backend.isAvailable()`. For `nd4j-native` this always returns `true`; for `nd4j-cuda` it returns `true` only when a compatible CUDA runtime and at least one CUDA-capable GPU are detected.
3. The first available backend is selected. Priority among multiple available backends is determined by the `getPriority()` method on each `Nd4jBackend`; higher numbers win. The CUDA backend has higher priority than the CPU backend so that, on a machine where both JARs are present and CUDA is available, the GPU backend is chosen automatically.
4. The chosen backend initializes its native libraries, allocates internal memory pools, and registers its factories with the `Nd4j` class.

You can inspect which backend was loaded at runtime:

```java
System.out.println(Nd4j.getBackend().getClass().getName());
// org.nd4j.linalg.cpu.nativecpu.CpuBackend  (CPU)
// org.nd4j.linalg.jcublas.JCublasBackend     (GPU)
```

To force a specific backend regardless of priority, set the system property before any ND4J class is loaded:

```
-Dbackend.type=CPU
```

Or configure it programmatically before the first `Nd4j` call:

```java
System.setProperty("backend.type", "CPU");
```


## Practical Rule: One Backend on the Classpath

You should place **exactly one backend** on the classpath. Having both `nd4j-native` and `nd4j-cuda` on the classpath at the same time is unsupported — while the priority mechanism will pick one, the presence of both JARs can cause classpath conflicts, unexpected native library loading, and hard-to-debug initialization errors.

The clean approach is to use Maven profiles to switch backends between environments:

```xml
<profiles>
  <!-- CPU profile (default) -->
  <profile>
    <id>cpu</id>
    <activation><activeByDefault>true</activeByDefault></activation>
    <dependencies>
      <dependency>
        <groupId>org.nd4j</groupId>
        <artifactId>nd4j-native-platform</artifactId>
        <version>${dl4j.version}</version>
      </dependency>
    </dependencies>
  </profile>

  <!-- GPU profile -->
  <profile>
    <id>cuda</id>
    <dependencies>
      <dependency>
        <groupId>org.nd4j</groupId>
        <artifactId>nd4j-cuda-11.6-platform</artifactId>
        <version>${dl4j.version}</version>
      </dependency>
    </dependencies>
  </profile>
</profiles>
```

Build for CPU: `mvn package` (default profile).
Build for GPU: `mvn package -P cuda`.


## Platform JARs and Native Classifier Resolution

Native libraries (`.so`, `.dll`, `.dylib`) are bundled inside the backend JARs using [JavaCPP Presets](https://github.com/bytedeco/javacpp-presets). There are two packaging options:

### `-platform` artifact (recommended)

`nd4j-native-platform` and `nd4j-cuda-11.6-platform` are aggregator POMs that pull in native JARs for **all supported operating systems and CPU architectures**: Linux x86_64, Linux ARM64, macOS x86_64, Windows x86_64, and more.

Use `-platform` for:
- Projects distributed as fat JARs or Docker images that may run on multiple operating systems.
- CI/CD pipelines that build on one OS and deploy to another.
- Situations where you want to avoid thinking about classifier management.

```xml
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-native-platform</artifactId>
  <version>1.0.0-M2.1</version>
</dependency>
```

### Non-platform artifact with classifier

When you know the exact target platform and want a smaller JAR, add only the native JAR for that platform using the `javacpp.platform` classifier:

```xml
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-native</artifactId>
  <version>1.0.0-M2.1</version>
</dependency>

<!-- Natives for current host platform only -->
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-native</artifactId>
  <version>1.0.0-M2.1</version>
  <classifier>linux-x86_64</classifier>
</dependency>
```

Available classifiers for `nd4j-native`:

| Classifier | Platform |
|---|---|
| `linux-x86_64` | Linux 64-bit x86 |
| `linux-x86_64-avx2` | Linux 64-bit x86 with AVX2 |
| `linux-x86_64-avx512` | Linux 64-bit x86 with AVX-512 |
| `linux-arm64` | Linux ARM64 (AArch64) |
| `linux-ppc64le` | Linux PowerPC 64-bit little-endian |
| `windows-x86_64` | Windows 64-bit x86 |
| `macosx-x86_64` | macOS 64-bit x86 |
| `macosx-arm64` | macOS Apple Silicon (M1/M2) |

The `-platform` artifact simply depends on all of the classifiers above simultaneously.


## Classpath Resolution at Startup

When your application starts, the JVM loads classes on demand. The backend is initialized the first time any code touches the `Nd4j` class. The sequence:

1. `Nd4j` static initializer fires.
2. `ServiceLoader` scans the classpath for `META-INF/services/org.nd4j.linalg.factory.Nd4jBackend`.
3. Each registered backend class is instantiated and `isAvailable()` is tested.
4. The winning backend calls its `load()` method, which:
   - Extracts native libraries from the JAR to a temp directory if needed (JavaCPP handles this).
   - Calls `System.load()` on the native shared library.
   - Allocates workspace memory and initializes thread pools.
5. `Nd4j.factory()`, `Nd4j.getBlasWrapper()`, and other singletons are pointed at the backend's implementations.

From this point on every `Nd4j.create(...)`, `INDArray.mmul(...)`, etc. dispatches through the native backend.

If no backend is found on the classpath, ND4J throws `ND4JIllegalStateException: No nd4jbackend found` at step 2. The fix is always to add a backend dependency, not to modify ND4J source code.


## Performance: Both Backends Are Native

A common misconception is that the CPU backend is a "pure Java fallback." It is not. Both backends call into libnd4j C++ code via JNI:

- `nd4j-native` uses **BLAS** (OpenBLAS or MKL) for matrix operations and uses SIMD intrinsics (SSE4, AVX2, AVX-512) compiled into libnd4j for element-wise ops.
- `nd4j-cuda` uses **cuBLAS** for matrix operations and custom CUDA kernels for element-wise ops.

The only Java-side difference between them is which native shared library is loaded and which factory classes are registered. All INDArray method signatures, broadcasting rules, indexing semantics, and op semantics are identical across both backends.


## See Also

- [CPU Backend (nd4j-native)](./cpu) — Maven setup, AVX tuning, BLAS configuration, threading
- [CUDA Backend (nd4j-cuda)](./cuda) — CUDA version matrix, cuDNN, multi-GPU, memory management
- [Memory and Workspaces](../../core-concepts/memory-and-workspaces) — off-heap memory, workspace scopes, leak detection

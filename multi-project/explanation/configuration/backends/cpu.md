---
title: "CPU Backend (nd4j-native)"
description: "Setting up the nd4j-native CPU backend — Maven dependencies, AVX2/AVX512 optimizations, OpenBLAS, MKL, and multi-threading"
---

# CPU Backend (nd4j-native)

`nd4j-native` is the CPU backend for ND4J. It executes all array operations through **libnd4j**, a C++ compute engine that calls into BLAS routines and uses SIMD vector instructions where available. This page covers Maven setup, AVX optimization classifiers, BLAS configuration, threading controls, and memory tuning.


## Maven Dependencies

### Recommended: `-platform` artifact

The simplest setup uses the `-platform` artifact, which bundles natives for all supported operating systems and CPU architectures in one dependency block:

```xml
<properties>
  <dl4j.version>1.0.0-M2.1</dl4j.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native-platform</artifactId>
    <version>${dl4j.version}</version>
  </dependency>
</dependencies>
```

This is appropriate for most projects. Maven resolves the native JARs for Linux x86_64, Windows x86_64, macOS x86_64, macOS ARM64, Linux ARM64, and Linux ppc64le simultaneously. At runtime JavaCPP extracts only the library matching the current OS and CPU.

### Minimal: current-platform only

To reduce JAR size for a known deployment target, omit `-platform` and add an explicit classifier:

```xml
<dependencies>
  <!-- API + Java code -->
  <dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${dl4j.version}</version>
  </dependency>

  <!-- Natives for one platform only -->
  <dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${dl4j.version}</version>
    <classifier>linux-x86_64</classifier>
  </dependency>
</dependencies>
```

### Using the DL4J BOM

If you are using the broader DL4J stack (DataVec, DL4J training layers, etc.), import the BOM to keep all version numbers consistent:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.deeplearning4j</groupId>
      <artifactId>deeplearning4j-parent</artifactId>
      <version>${dl4j.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native-platform</artifactId>
  </dependency>
</dependencies>
```

With the BOM the `<version>` tag on individual dependencies is optional — the BOM pins them.


## Platform Classifiers

Available native classifiers for `nd4j-native` in M2.1:

| Classifier | OS | Architecture | Notes |
|---|---|---|---|
| `linux-x86_64` | Linux | x86-64 | Baseline SSE4.1 |
| `linux-x86_64-avx2` | Linux | x86-64 | AVX2 SIMD (Haswell+) |
| `linux-x86_64-avx512` | Linux | x86-64 | AVX-512 (Skylake-X+, Ice Lake+) |
| `linux-arm64` | Linux | AArch64 | AWS Graviton, Raspberry Pi 64-bit |
| `linux-ppc64le` | Linux | POWER8/9 | IBM Power Systems |
| `windows-x86_64` | Windows | x86-64 | Baseline SSE4.1 |
| `windows-x86_64-avx2` | Windows | x86-64 | AVX2 SIMD |
| `macosx-x86_64` | macOS | x86-64 | Intel Mac |
| `macosx-arm64` | macOS | AArch64 | Apple Silicon (M1/M2/M3) |

### `-compile` classifiers (1.0.0-rewrite)

The 1.0.0-rewrite release adds `-compile` variants for each platform. These bundle the DSP JIT compilation stack (Triton, MLIR) into the native binary, enabling kernel fusion and JIT-compiled execution. The base classifiers above run standard ops and CUDA graph capture/replay but do not include JIT fusion.

| Classifier | OS | Architecture | Includes |
|---|---|---|---|
| `linux-x86_64-compile` | Linux | x86-64 | Triton + MLIR + oneDNN |
| `linux-arm64-compile` | Linux | AArch64 | MLIR |
| `macosx-arm64-compile` | macOS | AArch64 | MLIR + MLX |
| `android-arm64-compile` | Android | AArch64 | MLIR |
| `android-arm64-compile-nnapi` | Android | AArch64 | MLIR + NNAPI |

**Trade-off:** `-compile` classifiers produce a larger binary with more native dependencies (LLVM/Triton), but enable the full DSP JIT pipeline for maximum performance. The base classifiers are smaller and simpler to deploy. See [Hardware Backends — Classifier Variants](./hardware-backends#2-classifier-variants-base-vs-compile) for the complete trade-off guide.

```xml
<!-- CPU with full DSP JIT on Linux x86-64 -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${dl4j.version}</version>
    <classifier>linux-x86_64-compile</classifier>
</dependency>
```

When using `-platform`, Maven pulls all of these classifiers. When specifying one manually, pick the classifier that matches your deployment target.


## AVX2 and AVX-512 Optimizations

libnd4j is compiled in multiple variants corresponding to SIMD instruction sets. The `-avx2` and `-avx512` classifier variants contain a libnd4j compiled with those instruction sets enabled, delivering significant throughput improvements for element-wise and reduction operations compared to the baseline SSE4.1 build.

### Selecting AVX via Maven classifier

Replace the plain classifier with the AVX-enabled one:

```xml
<!-- AVX2 — works on Intel Haswell (2013+), AMD Ryzen (2017+) -->
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-native</artifactId>
  <version>${dl4j.version}</version>
  <classifier>linux-x86_64-avx2</classifier>
</dependency>

<!-- AVX-512 — works on Intel Skylake-X, Ice Lake, Cascade Lake and newer -->
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-native</artifactId>
  <version>${dl4j.version}</version>
  <classifier>linux-x86_64-avx512</classifier>
</dependency>
```

### Selecting AVX via system property (with `-platform`)

When using the `-platform` artifact all classifier variants are on the classpath. JavaCPP picks the correct native at startup, but you can steer it toward the AVX variant with:

```
-Djavacpp.platform.extension=-avx2
```

or

```
-Djavacpp.platform.extension=-avx512
```

Set this property on the JVM command line before the application starts. The value is appended to the detected platform string (e.g., `linux-x86_64`) to form `linux-x86_64-avx2`, and JavaCPP loads that native library instead of the baseline one.

**Verification:** check that the AVX library was loaded:

```java
System.out.println(System.getProperty("javacpp.platform"));
// linux-x86_64
System.out.println(System.getProperty("javacpp.platform.extension"));
// -avx2
```

If the property is set but the classifier is not on the classpath, JavaCPP falls back to the baseline library and logs a warning.

### Which AVX level to use

| Instruction set | Minimum CPU | Expected speedup over baseline |
|---|---|---|
| Baseline (SSE4.1) | Any x86-64 since ~2007 | — |
| AVX2 | Intel Haswell (2013), AMD Ryzen (2017) | 1.5–2× for float ops |
| AVX-512 | Intel Skylake-X (2017), Ice Lake (2019) | 2–4× for float ops |

Run `lscpu | grep -o 'avx[^ ]*'` on Linux to check which instruction sets your CPU supports. Do not specify `-avx512` on a machine that lacks AVX-512 — the process will crash with `SIGILL`.


## BLAS Libraries

BLAS (Basic Linear Algebra Subprograms) is used for matrix multiplication (`mmul`), dot products, and similar level-3 operations. ND4J ships with **OpenBLAS** bundled inside libnd4j and uses it by default.

### OpenBLAS (bundled, default)

No additional configuration is required. OpenBLAS is statically linked into libnd4j. It auto-detects the number of physical CPU cores and sets its internal thread count accordingly.

### Intel MKL (optional, higher performance)

On Intel hardware, Intel MKL typically delivers 10–40% higher throughput for matrix operations compared to OpenBLAS. MKL is not bundled with ND4J but can be provided on the `LD_LIBRARY_PATH` and activated automatically.

Steps to use MKL:

1. Install Intel oneAPI Math Kernel Library (MKL). The free `intel-mkl` package is available via the Intel apt/yum repository or the `mkl` conda package.

2. Add the MKL library path to `LD_LIBRARY_PATH` before launching the JVM:

```bash
export LD_LIBRARY_PATH=/opt/intel/mkl/lib/intel64:$LD_LIBRARY_PATH
java -jar myapp.jar
```

3. Tell ND4J to prefer MKL by setting the BLAS library name:

```
-Dorg.bytedeco.openblas.load=mkl_rt
```

4. Verify MKL is in use at runtime:

```java
String blasLib = NativeOpsHolder.getInstance().getDeviceNativeOps().getBlasLibraryName();
System.out.println(blasLib);  // should contain "mkl"
```

If MKL is not found on `LD_LIBRARY_PATH`, libnd4j silently falls back to the bundled OpenBLAS.

### Disabling multi-threaded BLAS

Some workloads perform many small matrix multiplications in parallel (e.g., batched inference on a thread pool). In this case it is more efficient to use single-threaded BLAS and let your own thread pool provide parallelism:

```bash
export MKL_NUM_THREADS=1
export OPENBLAS_NUM_THREADS=1
```

or equivalently:

```
-Dorg.bytedeco.openblas.num.threads=1
```


## Thread Configuration

### OMP_NUM_THREADS

libnd4j uses OpenMP for parallelizing element-wise operations and reductions. By default it uses all available logical CPUs. Set `OMP_NUM_THREADS` to restrict it:

```bash
export OMP_NUM_THREADS=4
java -jar myapp.jar
```

Keeping `OMP_NUM_THREADS` at or below the number of **physical** CPU cores (not hyperthreads) usually yields better sustained throughput.

### Nd4j.setNumThreads()

Thread count can also be set programmatically at any point before or during computation:

```java
// Set to 4 threads for all subsequent ops
Nd4j.setNumThreads(4);

// Read the current setting
int n = Nd4j.getNumThreads();
System.out.println("Using " + n + " threads");
```

The programmatic setting overrides the `OMP_NUM_THREADS` environment variable.

### Thread recommendations by workload

| Workload | Recommendation |
|---|---|
| Single large matmul or FFT | Use all physical cores (`OMP_NUM_THREADS` = core count) |
| Many small ops in a Java thread pool | Set `OMP_NUM_THREADS=1`, parallelize at the Java level |
| Training a neural network (DL4J) | Default (all cores); DL4J manages its own parallelism |
| Inference server under load | Tune empirically; often `OMP_NUM_THREADS=2` or `4` |


## Memory Configuration

ND4J uses off-heap memory (native C heap) for array data, managed by JavaCPP. The following JVM system properties control the memory limits.

### Maximum off-heap allocation

```
-Dorg.bytedeco.javacpp.maxbytes=4g
```

This caps the total off-heap memory JavaCPP will allocate before triggering garbage collection to reclaim unused native arrays. Values accept `k`, `m`, `g` suffixes. Default is unlimited (or a JVM-version-specific default).

### Maximum physical bytes

```
-Dorg.bytedeco.javacpp.maxphysicalbytes=8g
```

Sets a hard ceiling on the combined Java heap + off-heap usage. If the process exceeds this limit, JavaCPP raises `OutOfMemoryError`. Set this slightly below the machine's available RAM to prevent swapping.

### Typical production invocation

```bash
java \
  -Xmx4g \
  -Dorg.bytedeco.javacpp.maxbytes=8g \
  -Dorg.bytedeco.javacpp.maxphysicalbytes=12g \
  -Djavacpp.platform.extension=-avx2 \
  -Dorg.bytedeco.openblas.num.threads=8 \
  -jar myapp.jar
```

Here the Java heap is capped at 4 GB and the ND4J off-heap pool is allowed up to 8 GB, for a combined maximum of 12 GB of physical memory.

### Workspace-based memory management

For training loops and repeated inference, enable ND4J workspaces to reuse off-heap buffers without waiting for garbage collection. See [Memory and Workspaces](../../core-concepts/memory-and-workspaces) for the full guide.


## Verifying the CPU Backend Is Active

```java
import org.nd4j.linalg.factory.Nd4j;

// Print the backend class name
System.out.println(Nd4j.getBackend().getClass().getName());
// Expected: org.nd4j.linalg.cpu.nativecpu.CpuBackend

// Print the native library that was loaded
System.out.println(Nd4j.getEnvironment().isCPU());
// true

// Create a small array as a smoke test
INDArray x = Nd4j.linspace(0, 9, 10);
System.out.println(x);
// [0.0000, 1.0000, 2.0000, 3.0000, 4.0000, 5.0000, 6.0000, 7.0000, 8.0000, 9.0000]
```

If initialization fails, the most common causes are:

- No `nd4j-native` or `nd4j-native-platform` on the classpath. Add the dependency.
- The AVX classifier was specified but the CPU does not support it. Use the baseline classifier or check `lscpu`.
- A conflicting native library (`libopenblas.so`) on `LD_LIBRARY_PATH` that mismatches the bundled version. Remove or reorder `LD_LIBRARY_PATH`.


## Quick Reference

| Goal | Setting |
|---|---|
| Add CPU backend (all platforms) | `org.nd4j:nd4j-native-platform:1.0.0-M2.1` |
| Add CPU backend (Linux x86_64 only) | `org.nd4j:nd4j-native:1.0.0-M2.1:linux-x86_64` |
| Enable AVX2 via system property | `-Djavacpp.platform.extension=-avx2` |
| Enable AVX-512 via system property | `-Djavacpp.platform.extension=-avx512` |
| Set OpenMP thread count | `export OMP_NUM_THREADS=N` or `Nd4j.setNumThreads(N)` |
| Limit off-heap memory | `-Dorg.bytedeco.javacpp.maxbytes=Ng` |
| Limit total physical memory | `-Dorg.bytedeco.javacpp.maxphysicalbytes=Ng` |
| Use Intel MKL instead of OpenBLAS | `-Dorg.bytedeco.openblas.load=mkl_rt` |
| Check active backend | `Nd4j.getBackend().getClass().getName()` |


## See Also

- [Backends Overview](./overview) — SPI mechanism, backend discovery, classpath rules
- [CUDA Backend (nd4j-cuda)](./cuda) — GPU setup, multi-GPU, VRAM management
- [Memory and Workspaces](../../core-concepts/memory-and-workspaces) — off-heap memory, workspace scopes

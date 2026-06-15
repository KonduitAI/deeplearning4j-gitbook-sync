---
title: "CUDA Backend (nd4j-cuda)"
description: "Setting up the nd4j-cuda GPU backend — CUDA versions, cuDNN integration, multi-GPU, and GPU memory management"
---

# CUDA Backend (nd4j-cuda)

`nd4j-cuda` is the GPU backend for ND4J. It executes array operations through **libnd4j** CUDA kernels and **cuBLAS** for matrix operations, delivering significant throughput improvements over the CPU backend for large batch sizes and wide matrix operations. This page covers Maven setup, CUDA version compatibility, cuDNN integration, multi-GPU configuration, and GPU memory management.


## Maven Dependencies

### CUDA version in the artifact name

Unlike most Maven artifacts, the CUDA toolkit version is encoded directly in the artifact ID. M2.1 ships with CUDA 11.6 support:

```xml
<properties>
  <dl4j.version>1.0.0-M2.1</dl4j.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.6-platform</artifactId>
    <version>${dl4j.version}</version>
  </dependency>
</dependencies>
```

The `-platform` suffix bundles native JARs for all supported OS + CUDA combinations (Linux x86_64, Windows x86_64). Use it unless you have strong JAR-size constraints.

### Minimal: explicit classifier

```xml
<dependencies>
  <!-- API + Java code -->
  <dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.6</artifactId>
    <version>${dl4j.version}</version>
  </dependency>

  <!-- Natives for Linux x86_64 only -->
  <dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.6</artifactId>
    <version>${dl4j.version}</version>
    <classifier>linux-x86_64</classifier>
  </dependency>
</dependencies>
```

Available classifiers for `nd4j-cuda-11.6`:

| Classifier | Platform |
|---|---|
| `linux-x86_64` | Linux 64-bit x86 |
| `windows-x86_64` | Windows 64-bit x86 |

macOS is not supported by the CUDA backend (NVIDIA does not ship CUDA for macOS).

### `-compile` classifier (1.0.0-rewrite)

The 1.0.0-rewrite release adds a `-compile` variant for the CUDA backend that bundles the Triton MLIR GPU JIT compiler, NVRTC runtime compiler, and PTX string-template backend. This enables DSP kernel fusion — where consecutive element-wise ops are compiled into a single GPU kernel at runtime — on top of CUDA graph capture/replay.

| Classifier | Description |
|---|---|
| `linux-x86_64-cuda-12.9-compile` | CUDA 12.9 with Triton + NVRTC + PTX JIT |

**Trade-off:** The base CUDA classifier already supports CUDA graph capture/replay (which eliminates per-kernel launch overhead). The `-compile` variant adds JIT kernel fusion on top, reducing global memory traffic between ops. This is most impactful for transformer/LLM inference at low batch sizes. The cost is a larger binary that includes the Triton/LLVM compiler stack.

```xml
<!-- CUDA with full Triton JIT -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-12.9</artifactId>
    <version>${dl4j.version}</version>
    <classifier>linux-x86_64-cuda-12.9-compile</classifier>
</dependency>
```

When using `-platform`, select the `-compile` variant at runtime:

```
-Djavacpp.platform.extension=-compile
```

See [Hardware Backends — Classifier Variants](../backends/hardware-backends#2-classifier-variants-base-vs-compile) for the complete trade-off analysis.

### Using the DL4J BOM

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
    <artifactId>nd4j-cuda-11.6-platform</artifactId>
  </dependency>
</dependencies>
```


## CUDA Version Compatibility

The CUDA version in the artifact ID refers to the **CUDA toolkit** that libnd4j was compiled against. The CUDA runtime on the host machine must be compatible with this version.

| ND4J Artifact | Required CUDA Runtime | Minimum Driver Version (Linux) |
|---|---|---|
| `nd4j-cuda-11.6` | CUDA 11.6 | 510.39 |

**Forward compatibility:** CUDA is forward-compatible at the minor version level. A CUDA 11.6 binary runs on a machine with CUDA 11.7 or 11.8 drivers. It does not run on CUDA 10.x or earlier.

**Checking installed CUDA version:**

```bash
nvidia-smi          # shows driver version and CUDA version in the top right
nvcc --version      # shows toolkit version installed with nvcc
```

**Checking minimum GPU compute capability:**

ND4J M2.1 requires compute capability 3.5 (Kepler) or higher. Modern training workloads perform best on compute capability 7.0 (Volta) or later, which enables Tensor Cores for mixed-precision matrix multiplication.

```bash
nvidia-smi --query-gpu=compute_cap --format=csv,noheader
# 8.6  (e.g., RTX 3090)
```


## cuDNN Integration

cuDNN (CUDA Deep Neural Network library) provides highly optimized implementations of convolutions, pooling, batch normalization, and RNN cells. It accelerates DL4J neural network training but is not required for raw ND4J array operations.

To enable cuDNN acceleration, add the `deeplearning4j-cuda` artifact to your project alongside `nd4j-cuda`:

```xml
<dependencies>
  <!-- ND4J CUDA backend -->
  <dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.6-platform</artifactId>
    <version>${dl4j.version}</version>
  </dependency>

  <!-- DL4J cuDNN helpers -->
  <dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-cuda-11.6</artifactId>
    <version>${dl4j.version}</version>
  </dependency>
</dependencies>
```

When both JARs are present and cuDNN is installed on the host, DL4J automatically detects and uses cuDNN for supported layer types (Conv2D, LSTM, BatchNormalization, etc.). No additional code changes are required in the model definition.

**cuDNN installation:** Download cuDNN from [developer.nvidia.com/cudnn](https://developer.nvidia.com/cudnn). The cuDNN version must match the CUDA toolkit version (cuDNN 8.x for CUDA 11.6). Place the cuDNN shared libraries on `LD_LIBRARY_PATH` or in `/usr/local/cuda/lib64`.

**Checking cuDNN availability at runtime:**

```java
import org.deeplearning4j.nn.conf.layers.ConvolutionLayer;
// cuDNN is used automatically; check logs for "Using cuDNN" messages
// Or check via CudaEnvironment:
System.out.println(CudaEnvironment.getInstance().getConfiguration().toString());
```


## GPU Memory Management

GPU memory (VRAM) is a finite resource. ND4J allocates GPU memory for array data using a pooling allocator that reduces the overhead of frequent `cudaMalloc`/`cudaFree` calls.

### maxbytes: capping VRAM usage

```
-Dorg.bytedeco.javacpp.maxbytes=6g
```

This property caps the total GPU memory ND4J will allocate. Set it to a value below the GPU's total VRAM to leave room for the CUDA runtime, cuDNN workspace buffers, and the operating system.

A 16 GB GPU used for training should leave 1–2 GB headroom:

```bash
java \
  -Xmx4g \
  -Dorg.bytedeco.javacpp.maxbytes=14g \
  -jar myapp.jar
```

### Memory mode: IMMEDIATE vs. DELAYED

ND4J on CUDA supports two memory allocation modes controlled at startup:

```java
import org.nd4j.jita.conf.CudaEnvironment;
import org.nd4j.jita.conf.Configuration;

CudaEnvironment.getInstance().getConfiguration()
    .setMemoryModel(Configuration.AllocationModel.IMMEDIATE);
    // or: AllocationModel.DELAYED
```

- `IMMEDIATE` (default): allocate GPU memory as soon as an array is created.
- `DELAYED`: defer GPU allocation until the array is actually used in a computation. Useful when constructing large model parameter arrays where only a subset will be used in any given forward pass.

### GPU memory fragmentation

Long-running training jobs can fragment the GPU memory pool, leading to `cudaMalloc` failures even when `nvidia-smi` shows available VRAM. If you encounter this, enable the memory defragmentation by periodically calling:

```java
Nd4j.getWorkspaceManager().destroyAllWorkspacesForCurrentThread();
System.gc();
```

For training loops, wrapping mini-batch computation in ND4J workspaces is the most effective strategy — see [Memory and Workspaces](../../core-concepts/memory-and-workspaces).


## Multi-GPU Configuration

ND4J supports using multiple GPUs within a single JVM process. By default only device 0 is used. Enable multi-GPU support before any ND4J operations:

```java
import org.nd4j.jita.conf.CudaEnvironment;

CudaEnvironment.getInstance().getConfiguration()
    .allowMultiGPU(true);
```

### Device affinity per thread

ND4J maintains a thread-to-device mapping. All arrays created by a given thread are allocated on that thread's assigned device. Use the affinity manager to read or change the assignment:

```java
import org.nd4j.linalg.api.concurrency.AffinityManager;
import org.nd4j.linalg.factory.Nd4j;

AffinityManager mgr = Nd4j.getAffinityManager();

// Get the device index for the current thread
int device = mgr.getDeviceForCurrentThread();
System.out.println("Current thread on GPU: " + device);

// Pin the current thread to GPU 1
mgr.unsafeSetDevice(1);

// Move an array to a specific device
INDArray x = Nd4j.rand(1024, 1024);
mgr.ensureLocation(x, AffinityManager.Location.DEVICE);
```

### Multi-GPU with DL4J ParallelWrapper

For data-parallel training across multiple GPUs, use `ParallelWrapper`:

```java
import org.deeplearning4j.parallelism.ParallelWrapper;

MultiLayerNetwork model = buildModel();

ParallelWrapper wrapper = new ParallelWrapper.Builder(model)
    .prefetchBuffer(4)
    .workers(4)              // number of GPU workers (usually = number of GPUs)
    .averagingFrequency(3)   // average gradients every N mini-batches
    .reportScoreAfterAveraging(true)
    .build();

wrapper.fit(trainDataset);
```

Each worker thread is pinned to a different GPU. Gradients are averaged across workers every `averagingFrequency` mini-batches, and the averaged weights are copied back to all workers.

### Listing available GPUs

```java
import org.nd4j.jita.conf.CudaEnvironment;

int numGpus = CudaEnvironment.getInstance().getConfiguration().getAvailableDevices().size();
System.out.println("Available GPUs: " + numGpus);

for (Integer deviceId : CudaEnvironment.getInstance().getConfiguration().getAvailableDevices()) {
    System.out.println("Device " + deviceId);
}
```

Or from the command line:

```bash
nvidia-smi --list-gpus
# GPU 0: NVIDIA A100-SXM4-40GB (UUID: ...)
# GPU 1: NVIDIA A100-SXM4-40GB (UUID: ...)
```


## Device Selection

When a machine has multiple GPUs but you want to restrict ND4J to a specific subset, use the CUDA environment variable:

```bash
export CUDA_VISIBLE_DEVICES=0,2   # expose only GPU 0 and GPU 2 to the process
java -jar myapp.jar
```

`CUDA_VISIBLE_DEVICES` renumbers devices from the process's perspective: GPU 0 inside the process is the physical GPU 0, and GPU 1 inside the process is physical GPU 2. ND4J respects this environment variable and will only see the enumerated devices.

To select a device programmatically within the process:

```java
// Before any Nd4j operation, set the default device for the main thread
CudaEnvironment.getInstance().getConfiguration().setDefaultDevice(1);
```


## Checking GPU Backend Availability

```java
import org.nd4j.linalg.factory.Nd4j;

// Print the backend class name
System.out.println(Nd4j.getBackend().getClass().getName());
// Expected: org.nd4j.linalg.jcublas.JCublasBackend

// Check whether the active backend is GPU-based
boolean isGpu = Nd4j.getBackend().getClass().getName().contains("Cublas");
System.out.println("GPU backend active: " + isGpu);

// Print the number of available CUDA devices
System.out.println("CUDA devices: " +
    CudaEnvironment.getInstance().getConfiguration().getAvailableDevices().size());

// Smoke test
INDArray x = Nd4j.rand(DataType.FLOAT, 1000, 1000);
INDArray y = Nd4j.rand(DataType.FLOAT, 1000, 1000);
INDArray z = x.mmul(y);
System.out.println("mmul shape: " + Arrays.toString(z.shape()));
// mmul shape: [1000, 1000]
```

### Common initialization errors

| Error | Cause | Fix |
|---|---|---|
| `No nd4jbackend found` | `nd4j-cuda-11.6` not on classpath | Add the Maven dependency |
| `CUDA driver version is insufficient` | Driver too old for CUDA 11.6 | Upgrade NVIDIA driver to 510.39+ |
| `no CUDA-capable device is detected` | No GPU, or CUDA_VISIBLE_DEVICES="" | Check `nvidia-smi` output |
| `cudaMalloc failed: out of memory` | VRAM exhausted | Reduce batch size, lower `maxbytes`, use workspaces |
| `cuDNN not found` | cuDNN not installed or not on LD_LIBRARY_PATH | Install cuDNN matching CUDA 11.6 |


## Performance Tips

### Use FLOAT, not DOUBLE, on GPU

Consumer GPUs (GeForce/RTX series) run `FLOAT32` operations at full throughput and `FLOAT64` at 1/32 or 1/64 of that. Volta and Ampere data-center GPUs have better FP64 throughput, but FP32 is still faster. Always prefer `DataType.FLOAT` for training unless double precision is a hard requirement.

```java
INDArray x = Nd4j.rand(DataType.FLOAT, batchSize, features);
```

### Minimize host-device transfers

Every call to `getDouble(i, j)`, `toDoubleVector()`, or `toDoubleMatrix()` copies data from the GPU back to the Java heap. In a training loop this is expensive. Avoid reading individual elements inside mini-batch loops; read scalars (loss, accuracy) only after an epoch or at checkpoint intervals.

### Batch size and occupancy

CUDA kernels achieve full GPU occupancy only when there are enough parallel work items to fill all streaming multiprocessors. For matrix operations this typically means a batch size of at least 32, and ideally 128 or higher. Very small batch sizes (1–8) will under-utilize the GPU.


## Quick Reference

| Goal | Setting |
|---|---|
| Add CUDA 11.6 backend (all platforms) | `org.nd4j:nd4j-cuda-11.6-platform:1.0.0-M2.1` |
| Add cuDNN support | `org.deeplearning4j:deeplearning4j-cuda-11.6:1.0.0-M2.1` |
| Limit VRAM usage | `-Dorg.bytedeco.javacpp.maxbytes=Ng` |
| Enable multi-GPU | `CudaEnvironment.getInstance().getConfiguration().allowMultiGPU(true)` |
| Get current thread's device | `Nd4j.getAffinityManager().getDeviceForCurrentThread()` |
| Restrict to specific GPUs | `export CUDA_VISIBLE_DEVICES=0,1` |
| Check backend class name | `Nd4j.getBackend().getClass().getName()` |
| Set default device | `CudaEnvironment.getInstance().getConfiguration().setDefaultDevice(N)` |


## See Also

- [Backends Overview](./overview) — SPI mechanism, backend discovery, classpath rules
- [CPU Backend (nd4j-native)](./cpu) — AVX tuning, BLAS configuration, threading
- [Memory and Workspaces](../../core-concepts/memory-and-workspaces) — workspace scopes, GPU memory reuse

---
title: Hardware Backends
description: GPU, TPU, DSP, and CPU acceleration backends — CUDA, TPU (PJRT), Hexagon (QNN), ZLUDA, ARM ACL, Apple Accelerate, cuDNN, MPS, MLIR, and multi-backend dispatch
---

# Hardware Backends

Deeplearning4j 1.0.0-rewrite extends ND4J's backend system well beyond the original CPU and CUDA pairing. The rewrite introduces dedicated backends for Google Cloud TPUs, Qualcomm Hexagon DSPs, AMD and Intel GPUs via ZLUDA, Snapdragon X, and a substantially expanded set of CPU acceleration libraries. It also ships a new multi-backend infrastructure layer that unifies device switching, memory tracking, and workspace management across all of these targets.

This page documents every new and expanded backend, their configuration, and the shared infrastructure that coordinates them.


## 1. Overview and Backend Selection

ND4J uses the standard Java SPI mechanism to discover backends at startup. Each backend JAR ships a `META-INF/services/org.nd4j.linalg.factory.Nd4jBackend` registration file. When `Nd4j` is first referenced, `ServiceLoader` collects all registered backends, calls `isAvailable()` on each, and selects the one with the highest `getPriority()` return value that reports itself as available.

The priority ladder in the rewrite:

| Backend | Priority | Notes |
|---|---|---|
| `nd4j-native` (CPU) | 0 | Always available; baseline fallback |
| `nd4j-tpu` | 50 | Selected over CPU when TPU hardware is detected |
| `nd4j-hexagon` | 60 | Selected on Snapdragon SoCs when QNN runtime is present |
| `nd4j-cuda` (CUDA/ZLUDA) | 100 | Highest priority; used when NVIDIA or ZLUDA GPU is present |

When multiple backends are on the classpath but only one is desired, override selection with:

```
-Dbackend.type=CPU        # force CPU regardless of available hardware
-Dbackend.type=TPU        # force TPU
-Dbackend.type=HEXAGON    # force Hexagon DSP
```

Or programmatically before the first `Nd4j` call:

```java
System.setProperty("backend.type", "CPU");
```

The new `DeviceType` enum enumerates every supported target:

```java
DeviceType.CPU
DeviceType.CUDA
DeviceType.ROCM
DeviceType.TPU
DeviceType.HEXAGON
DeviceType.OPENCL
DeviceType.METAL
DeviceType.VULKAN
```


## 2. CUDA Backend (with cuDNN Expansion)

The existing `nd4j-cuda` backend is unchanged in its public API. The rewrite adds 20 new and updated cuDNN helper files under `deeplearning4j-cuda`, along with structural changes for stream-capture safety.

### New cuDNN Operations

The following cuDNN-backed op implementations are new in this release:

| File | Op |
|---|---|
| `CudnnFlashAttentionHelper` | Flash Attention stub (multi-head attention with memory-efficient kernel) |
| `CudnnBiasAddHelper` | Bias-add fused with activation |
| `CudnnConv1dHelper` | 1-D convolution |
| `CudnnDeconv2dHelper` | 2-D transposed convolution (deconvolution) |
| `CudnnDeconv3dHelper` | 3-D transposed convolution |
| `CudnnDropoutHelper` | Stateful cuDNN dropout |
| `CudnnGlobalPoolingHelper` | Global average/max pooling |
| `CudnnGruHelper` | GRU cell forward and backward |
| `CudnnInstanceNormHelper` | Instance normalization |
| `CudnnLayerNormHelper` | Layer normalization |
| `CudnnLogSoftmaxHelper` | Log-softmax |
| `CudnnLrnHelper` | Local response normalization |
| `CudnnOpTensorHelper` | Pointwise tensor operations (add, mul, min, max) |
| `CudnnReduceHelper` | Reduce over arbitrary axes |
| `CudnnSimpleRnnHelper` | Simple RNN cell |
| `CudnnSpatialTransformerHelper` | Spatial transformer network |

### Updated cuDNN Operations

The following existing helpers were updated for CUDA stream-capture safety and DSP compatibility:

- `CudnnBatchNormHelper` — per-stream handle caching; correct behavior when CUDA graph capture is active
- `CudnnConv2dHelper` and `CudnnConv3dHelper` — stream-safe workspace allocation
- `CudnnCtcHelper` — CTC loss; rewritten to avoid illegal API calls inside CUDA graph capture
- `CudnnLSTMHelper` — updated cuDNN RNN API v8
- `CudnnDepthwiseConv2dHelper` — aligned with updated depthwise conv semantics

### Per-Stream cuDNN Handle Caching

The rewrite introduces centralized cuDNN handle caching keyed by CUDA stream. Each time a cuDNN operation is dispatched, the infrastructure checks whether a handle already exists for the current stream; if not, it creates and registers one. This eliminates handle creation overhead in tight loops and is the underlying change that makes stream capture safe across all cuDNN helpers.

### Maven Setup

```xml
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-cuda-11.6-platform</artifactId>
  <version>1.0.0-rewrite</version>
</dependency>

<!-- cuDNN helpers (optional, for neural network layer acceleration) -->
<dependency>
  <groupId>org.deeplearning4j</groupId>
  <artifactId>deeplearning4j-cuda-11.6</artifactId>
  <version>1.0.0-rewrite</version>
</dependency>
```

See [CUDA Backend (nd4j-cuda)](./cuda) for full CUDA setup, multi-GPU configuration, and memory management.


## 3. TPU Backend (nd4j-tpu)

`nd4j-tpu` is a new backend targeting Google Cloud TPU v4 and v5 hardware. It uses Google's **PJRT** (Portable JIT Runtime) API through JNI, so the Java layer never calls XLA or HLO directly.

### Java Components (7 files)

**`JTpuBackend`** — The `Nd4jBackend` subclass registered with the SPI. On `isAvailable()`, it fires a JNI probe that calls `PjrtClientManager::HasTpuDevice()` on the native side. If that returns true and a PJRT client can be created, the backend is considered available. Priority is 50, placing it above the CPU backend and below CUDA.

**`JTpuNDArray`** — The `INDArray` implementation for TPU. Array data is held in XLA buffer handles allocated through PJRT rather than in CPU or GPU memory. Operations on `JTpuNDArray` are routed through `TpuExecutioner` which compiles and dispatches HLO programs.

**`TpuEnvironment`** — Holds TPU-wide configuration. Key defaults:
- Data type default: `bfloat16` (the native TPU format; training in bfloat16 is strongly recommended)
- Compilation cache: HLO programs are cached by signature to avoid recompilation per step
- Device count: read from the PJRT client at startup

**`TpuExecutioner`** — Routes ND4J op calls to PJRT XLA execution. Each op call results in an HLO program fragment that is compiled (or retrieved from cache) and executed on a TPU device via `PjrtClientManager`.

### Native Components

**`TpuGraphBackend`** — The C++ entry point called from `TpuExecutioner` via JNI. Manages the top-level execution pipeline.

**`HloIRBuilder`** — Translates op descriptors into XLA HLO (High Level Optimizer) programs. Each op generates the appropriate HLO computation; multiple ops in a SameDiff graph can be fused into a single HLO program before dispatch.

**`PjrtClientManager`** — Manages the PJRT client lifecycle: device enumeration, memory allocation on TPU HBM (High Bandwidth Memory), and execution submission. The manager is a singleton per process.

### Setup

TPU support requires:

1. A Google Cloud VM with a TPU v4 or v5 pod slice attached, or a Cloud TPU node accessible via PJRT network endpoint.
2. The `libtpu.so` shared library, available from the Google Cloud TPU apt repository or bundled inside `nd4j-tpu`.
3. The environment variable `TPU_NAME` set to the TPU resource name (e.g., `local` for a TPU VM, or `projects/PROJECT/locations/ZONE/nodes/NODE_NAME` for a TPU node).

```xml
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-tpu</artifactId>
  <version>1.0.0-rewrite</version>
</dependency>
```

```bash
export TPU_NAME=local
java -jar myapp.jar
```

### Configuration

```java
import org.nd4j.linalg.tpu.TpuEnvironment;

// Check detected TPU device count
System.out.println(TpuEnvironment.getInstance().getDeviceCount());

// Override default data type (bfloat16 by default)
TpuEnvironment.getInstance().setDefaultDataType(DataType.FLOAT);

// Flush the HLO compilation cache (useful during development)
TpuEnvironment.getInstance().clearCompilationCache();
```

The `JTpuBackend` backend class name as reported at runtime:

```java
System.out.println(Nd4j.getBackend().getClass().getName());
// org.nd4j.linalg.tpu.JTpuBackend
```

### HLO Compilation and bfloat16

TPUs execute XLA HLO programs compiled ahead of execution. The first call to an op compiles an HLO program and caches it; subsequent calls with the same shapes hit the cache. Shape changes invalidate the cache entry and trigger recompilation.

bfloat16 is the recommended data type for TPU. It has the same dynamic range as float32 (8-bit exponent) but reduces mantissa precision to 7 bits. TPU matrix units run bfloat16 multiplications natively. Accumulations inside the matrix unit use float32, so effective precision for large matrix multiplications is higher than the storage format implies.

```java
// Create a bfloat16 array on TPU
INDArray x = Nd4j.rand(DataType.BFLOAT16, 1024, 1024);
INDArray y = Nd4j.rand(DataType.BFLOAT16, 1024, 1024);
INDArray z = x.mmul(y);  // dispatched through PJRT as HLO dot_general
```


## 4. Hexagon DSP Backend (nd4j-hexagon)

`nd4j-hexagon` targets Qualcomm Hexagon DSPs available on Snapdragon SoCs. It dispatches through the **Qualcomm Neural Network (QNN)** runtime, which in turn can use SNPE (Snapdragon Neural Processing Engine) or the newer QNN SDK.

### Java Components (6 files)

**`HexagonBackend`** — `Nd4jBackend` subclass. `isAvailable()` probes for the QNN shared libraries (`libQnnHtp.so`, `libQnnSystem.so`) via JNI. Returns available when running on a Snapdragon device with Hexagon support and the QNN runtime installed.

**`HexagonExecutioner`** — Dispatches ops to the QNN runtime. Converts ND4J op calls into Hexagon network graph operations and submits them for DSP execution.

### Native Components

**`HexagonGraphBackend`** — C++ entry point. Coordinates graph-level compilation and execution.

**`HexagonIRBuilder`** — Generates Hexagon network graph descriptors from op calls. Each ND4J op is translated into the corresponding QNN graph node type.

**`HexagonRuntimeManager`** — Manages QNN context handles, Hexagon DSP session lifecycle, and memory handles for DSP-accessible buffers.

### Setup

QNN setup requires:
1. A Snapdragon 8 Gen 2 or later SoC (or compatible Hexagon DSP).
2. Qualcomm QNN SDK installed, with `libQnnHtp.so` on `LD_LIBRARY_PATH`.
3. The `nd4j-hexagon` artifact on the classpath.

```xml
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-hexagon</artifactId>
  <version>1.0.0-rewrite</version>
</dependency>
```

```bash
export LD_LIBRARY_PATH=/opt/qcom/qnn/lib:$LD_LIBRARY_PATH
java -jar myapp.jar
```

Check the active backend:

```java
System.out.println(Nd4j.getBackend().getClass().getName());
// org.nd4j.linalg.hexagon.HexagonBackend
```

### Quantization

Hexagon DSPs deliver peak performance on INT8 and INT16 fixed-point operations. The QNN backend supports PTQ (post-training quantization) directly in the `HexagonIRBuilder` layer. Inputs are quantized per-tensor; the quantization parameters (scale and zero-point) are derived from calibration data passed before compilation.


## 5. ZLUDA (AMD and Intel GPU Support)

ZLUDA is a drop-in CUDA compatibility layer that translates CUDA API calls at runtime to AMD HIP/ROCm (for AMD GPUs) or Intel Level Zero (for Intel GPUs). The rewrite integrates ZLUDA support into the `nd4j-cuda` backend so that AMD and Intel GPUs become supported targets without requiring a separate backend JAR.

### How ZLUDA Works

ZLUDA intercepts calls to the CUDA runtime library (`libcuda.so`, `nvcuda.dll`) and redirects them to the appropriate native GPU SDK. From ND4J's perspective, the CUDA backend loads and operates normally; ZLUDA handles the translation transparently.

- **AMD GPUs (HIP/ROCm):** cuDNN calls are translated to **MIOpen** equivalents. cuBLAS calls are translated to **rocBLAS**.
- **Intel GPUs (Level Zero):** cuDNN calls are translated to **oneDNN** equivalents.

### Auto-Download

When the CUDA backend initializes and detects an AMD or Intel GPU, the native side (`ZludaConfiguration.cmake`) automatically downloads the appropriate ZLUDA build for the detected hardware. No manual installation of ZLUDA is required.

### Build Configuration

```cmake
# ZludaConfiguration.cmake — automatically included when targeting AMD/Intel
# Sets:
#   ZLUDA_ENABLED=ON
#   ZLUDA_TARGET=HIP    # or LEVEL_ZERO
#   CUDNN_SUBSTITUTE=MIOpen  # or oneDNN
```

From the Java side there is no configuration change; just add the CUDA backend dependency and target an AMD or Intel GPU machine.

```xml
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-cuda-11.6-platform</artifactId>
  <version>1.0.0-rewrite</version>
</dependency>
```

### Limitations

ZLUDA translation is not zero-overhead. Workloads that are heavily bottlenecked on cuBLAS or cuDNN will see near-native performance because ROCm and oneDNN are mature. Workloads that use custom CUDA kernels (some advanced sampler or attention kernels) may fall back to a slower translated path.


## 6. Snapdragon X (SDX) Cross-Device Dispatch

The Snapdragon X backend (`nd4j-sdx`) is a cross-device dispatch backend for Snapdragon X Elite and Snapdragon X Plus platforms. Rather than implementing a new execution engine, SDX routes ops to the most appropriate available device on the SoC: the ARM CPU, the Hexagon DSP, or the Adreno GPU, based on op type and tensor size heuristics.

Build support is provided by `BuildSDX.cmake`. No separate Java configuration is required; the SDX backend registers itself and its routing logic is internal.

```xml
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-sdx</artifactId>
  <version>1.0.0-rewrite</version>
</dependency>
```


## 7. ARM Compute Library (ACL) Backend

ARM Compute Library is a highly optimized collection of functions for ARM CPUs (Cortex-A) and Mali GPUs. The rewrite adds approximately 124 new op implementations under the ACL platform backend. These are registered through the `DECLARE_PLATFORM` / `PLATFORM_IMPL` / `PLATFORM_CHECK` macro system and dispatch on `ENGINE_CPU` when running on ARM hardware.

### Op Coverage

#### Activations

| Op | Notes |
|---|---|
| relu | Standard and leaky variants |
| elu | Exponential linear unit |
| gelu | Gaussian error linear unit |
| selu | Scaled exponential linear unit |
| sigmoid | Logistic sigmoid |
| silu | Sigmoid linear unit (x * sigmoid(x)) |
| softmax | Row-wise softmax |
| softplus | log(1 + exp(x)) |
| swish | x * sigmoid(beta * x) |
| tanh | Hyperbolic tangent |

#### Reductions

| Op | Notes |
|---|---|
| reduce_max | Reduce to max along specified axes |
| reduce_mean | Reduce to mean |
| reduce_min | Reduce to min |
| reduce_prod | Reduce to product |
| reduce_sum | Reduce to sum |

#### Convolutions and Attention

| Op | Notes |
|---|---|
| conv1d | 1-D convolution |
| depthwiseConv2d | Depthwise separable 2-D convolution |
| grouped_query_attention | Multi-head attention with grouped queries (GQA) |

#### Normalization

| Op | Notes |
|---|---|
| batchnorm | Batch normalization (inference and training) |
| instance_norm | Instance normalization |
| layer_norm | Layer normalization |
| l2_normalize | L2 normalization along specified axis |
| rms_norm | Root mean square normalization (LLM-specific) |

#### LLM-Specific

| Op | Notes |
|---|---|
| rope | Rotary position embeddings |
| rms_norm | See Normalization above |

#### Embeddings and Gather

| Op | Notes |
|---|---|
| embedding_lookup | Embedding table lookup |
| gather | Gather slices along an axis |
| gather_nd | Gather slices at multi-dimensional indices |

#### Scatter and Shape Ops

All scatter variants (`scatter_add`, `scatter_update`, `scatter_mul`, etc.) and all shape manipulation ops (`reshape`, `transpose`, `squeeze`, `unsqueeze`, `tile`, `repeat`, `stack`, `unstack`, `split`, `concat`) are covered.

#### Binary and Comparison Ops

All arithmetic binary ops and all comparison ops (`equal`, `not_equal`, `greater`, `greater_equal`, `less`, `less_equal`) are covered by ACL implementations.

### Maven Setup

ACL support is included in the ARM64 variant of `nd4j-native`. On AArch64 Linux or macOS Apple Silicon, ACL ops are used automatically when ARM Compute Library is detected.

```xml
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-native</artifactId>
  <version>1.0.0-rewrite</version>
  <classifier>linux-arm64</classifier>
</dependency>
```


## 8. Apple Accelerate Backend

The Apple Accelerate framework provides hardware-optimized math routines on macOS and iOS. The rewrite adds 28 new op implementations using Accelerate APIs. These are active on the `macosx-arm64` and `macosx-x86_64` classifiers of `nd4j-native`.

### Op Coverage

#### BLAS

| Op | Accelerate API |
|---|---|
| mmul (matrix-matrix) | `cblas_sgemm` |
| mmul (matrix-vector) | `cblas_sgemv` |
| dot | `cblas_sdot` |
| nrm2 | `cblas_snrm2` |
| scale | `cblas_sscal` |

#### FFT

| Op | Accelerate API |
|---|---|
| fft (real, radix-2) | `vDSP_fft_zrip` |

#### Convolutions

| Op | Accelerate API |
|---|---|
| conv1d | `vDSP_conv` |
| conv2d | `vDSP_conv` (via tiling) |

#### Normalization

| Op | Accelerate API |
|---|---|
| layer_norm | `vDSP` vector mean and variance ops |
| batchnorm | `vDSP` vector mean and variance ops |

#### Element-Wise Math

| Op | Accelerate API |
|---|---|
| sin | `vvsin` |
| cos | `vvcos` |
| exp | `vvexp` |
| log | `vvlog` |
| sqrt | `vvsqrt` |
| pow | `vvpow` |

#### Additional Coverage

Pooling operations (max pool, avg pool), comparison ops, cumulative sum (`cumsum`), cumulative product (`cumprod`), rounding ops (`floor`, `ceil`, `round`), conditional selection (`where`), gradient accumulation, and linear algebra operations (`svd`, `solve`) are all provided by Accelerate-backed implementations.

### Maven Setup

Accelerate support is included in the macOS classifier variants automatically. No additional dependency is required beyond `nd4j-native-platform` or the `macosx-arm64` / `macosx-x86_64` classifier.


## 9. llama.cpp / GGML Backend

The rewrite introduces a 60-file native backend for executing GGML (the tensor library underlying llama.cpp) models directly from ND4J. This backend enables loading and running quantized LLM weights (GGUF format) on CPU, Metal, and CUDA without converting them to ND4J's native format first.

The GGML backend sits alongside the standard `nd4j-native` execution path. When a GGUF model is loaded, ops that GGML can handle natively (matrix multiplication with quantized weights, attention, feed-forward blocks) are dispatched to the GGML execution path; the result arrays are then materialized as standard `INDArray` instances for the rest of the DL4J graph.

```xml
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-ggml</artifactId>
  <version>1.0.0-rewrite</version>
</dependency>
```


## 10. MLIR JIT, Apple MPS, MIOpen, and oneDNN

### MLIR JIT

`MlirCpuGraphBackend` is a new native backend module that compiles SameDiff graphs to MLIR (Multi-Level Intermediate Representation) and executes them via the MLIR Linalg and arith dialects. This path is used when the native side detects that JIT compilation via MLIR would be advantageous (e.g., operator fusion across a large subgraph).

The MLIR JIT path is transparent to the Java layer. Ops dispatched through SameDiff may be compiled into MLIR programs and executed; the results are returned as standard `INDArray` values.

### Apple Metal Performance Shaders (MPS)

`nd4j-mps` targets Apple Silicon GPU via Metal Performance Shaders. MPS provides GPU-accelerated matrix operations and neural network primitives on M1/M2/M3 Macs. The backend uses the Metal command queue for dispatch and shares the no-copy zero-copy buffer model with the CPU backend on unified-memory Apple Silicon systems.

```xml
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-mps</artifactId>
  <version>1.0.0-rewrite</version>
</dependency>
```

MPS is selected automatically on `macosx-arm64` when the MPS framework is available and the backend JAR is on the classpath.

### MIOpen (AMD GPU)

MIOpen is AMD's alternative to cuDNN. When ZLUDA routes CUDA traffic to HIP/ROCm, cuDNN calls are translated to MIOpen. The `nd4j-cuda` backend plus ZLUDA is the supported path; there is no separate `nd4j-miopen` artifact.

### oneDNN (Intel, formerly MKL-DNN)

oneDNN provides optimized operator implementations for Intel CPUs and Intel GPUs. In the rewrite, oneDNN is updated for DSP integration — the oneDNN execution path can be called from the Hexagon SDX dispatch layer when the target is an Intel CPU on a mixed platform. On x86 Intel CPUs, oneDNN is accessed through the existing MKL integration in `nd4j-native`; see [CPU Backend](./cpu) for setup.

### OpenVINO

OpenVINO integration allows `nd4j-native` on Intel hardware to dispatch inference graphs through the OpenVINO runtime. This is activated when `libopenvino.so` is detected on `LD_LIBRARY_PATH` and the model has been exported in a compatible format.


## 11. Multi-Backend Infrastructure

PR #10447 introduces a shared infrastructure layer used by all backends. These classes are in `nd4j-api` and are implemented by each backend.

### DeviceType and DeviceDescriptor

`DeviceType` is an enum with values for every supported target:

```java
DeviceType.CPU
DeviceType.CUDA
DeviceType.ROCM
DeviceType.TPU
DeviceType.HEXAGON
DeviceType.OPENCL
DeviceType.METAL
DeviceType.VULKAN
```

`DeviceDescriptor` is the base interface for describing a specific device. The concrete implementations are:

- **`CudaDeviceDescriptor`** — wraps a CUDA device index and CUDA stream handle
- **`CpuDeviceDescriptor`** — wraps a CPU thread identifier and NUMA node
- **`StubDeviceDescriptor`** — no-op implementation used in unit tests

### CudaDeviceContextProvider

`CudaDeviceContextProvider` consolidates the 15+ scattered device-switch call sites that existed in the previous codebase into a single canonical path. All code that needs to switch the active CUDA device now goes through this provider.

```java
// Previous pattern (scattered, now replaced):
// JCudaDriver.cuCtxSetCurrent(ctx);
// ... work ...
// JCudaDriver.cuCtxSetCurrent(prevCtx);

// New canonical pattern:
DeviceContextProvider provider = new CudaDeviceContextProvider();
try (DeviceContext ctx = provider.acquireContext(deviceDescriptor)) {
    // all work here; ctx.close() restores previous device automatically
}
```

### DeviceMemoryManager

`DeviceMemoryManager` provides per-device allocation tracking with configurable caps:

```java
import org.nd4j.linalg.device.DeviceMemoryManager;

DeviceMemoryManager mgr = DeviceMemoryManager.getInstance();

// Get total allocated bytes on device 0
long allocatedBytes = mgr.getAllocatedBytes(DeviceType.CUDA, 0);

// Set a cap for device 1 (16 GB)
mgr.setAllocationCap(DeviceType.CUDA, 1, 16L * 1024 * 1024 * 1024);

// Check remaining headroom
long available = mgr.getRemainingCapacity(DeviceType.CUDA, 1);
```

When an allocation would exceed the cap, `DeviceMemoryManager` throws `DeviceOutOfMemoryException` with a clear message showing current usage and the configured limit, rather than propagating an opaque native OOM error.

### DeviceContextProvider and DeviceContext

`DeviceContextProvider` is an interface implemented by each backend:

```java
public interface DeviceContextProvider {
    DeviceContext acquireContext(DeviceDescriptor descriptor);
}

public interface DeviceContext extends AutoCloseable {
    DeviceDescriptor getDescriptor();
    Object getNativeStreamHandle();  // CUDA stream, Metal command queue, etc.
    void close();  // restores previous device context
}
```

Using `try-with-resources` on `DeviceContext` guarantees that the previous context is always restored, even if the work block throws.

### MultiBackendWorkspace

`MultiBackendWorkspace` extends the existing ND4J workspace concept to span multiple devices. It maintains MSI (Memory Sharing Interface) coherence — when an array is accessed on a device where it was not most recently written, the workspace layer transparently copies the data before the access proceeds.

```java
import org.nd4j.linalg.device.MultiBackendWorkspace;

try (MultiBackendWorkspace ws = MultiBackendWorkspace.open("train-step")) {
    INDArray x = Nd4j.rand(DataType.FLOAT, 1024, 1024);  // allocated on default device
    ws.migrateToDevice(x, DeviceType.TPU, 0);           // move to TPU 0
    INDArray z = x.mmul(x.T());                          // executed on TPU
    ws.migrateToDevice(z, DeviceType.CPU, 0);            // bring result to CPU
    System.out.println(z.meanNumber());
}
```

### DeviceWorkspaceManager

`DeviceWorkspaceManager` is a thread-local registry of open `MultiBackendWorkspace` instances. Each thread has its own workspace stack; opening a workspace on thread A does not affect thread B.

```java
DeviceWorkspaceManager.getInstance().openWorkspace("scope-name");
// ... work ...
DeviceWorkspaceManager.getInstance().closeWorkspace("scope-name");
```

### DeviceRoutingConfiguration and MultiGpuTracer

`DeviceRoutingConfiguration` allows the application to specify routing rules — which device types are eligible for which op categories, and what fallback order to use when the preferred device is unavailable:

```java
DeviceRoutingConfiguration config = new DeviceRoutingConfiguration.Builder()
    .preferDevice(OpCategory.MATRIX_MULTIPLY, DeviceType.CUDA)
    .fallbackDevice(OpCategory.MATRIX_MULTIPLY, DeviceType.CPU)
    .preferDevice(OpCategory.ATTENTION, DeviceType.TPU)
    .build();

DeviceAwareOpExecutioner executioner = new DeviceAwareOpExecutioner(config);
```

`MultiGpuTracer` is a diagnostic utility that logs device transitions, allocation events, and cross-device copies during a traced execution window:

```java
try (MultiGpuTracer tracer = MultiGpuTracer.start()) {
    // ... operations ...
} // prints trace summary on close
```

### DeviceAwareNDArrayFactory and BackendRoutingStrategy

`DeviceAwareNDArrayFactory` is an `NDArrayFactory` implementation that consults the active `BackendRoutingStrategy` when creating arrays, routing allocation to the appropriate device:

```java
INDArray x = Nd4j.create(DataType.FLOAT, 1024, 1024);
// If routing strategy says CUDA for this size, x is allocated on GPU
// If it says TPU, x is an XLA buffer
// Application code does not change
```


## 12. Device Auto-Detection

When the backend is not forced via `backend.type`, ND4J probes available hardware in order of priority:

1. **CUDA:** calls `cudaGetDeviceCount()`. If one or more CUDA devices are found and the CUDA runtime is the expected version, `nd4j-cuda` is selected.
2. **TPU:** `PjrtClientManager::HasTpuDevice()` JNI probe. Requires `TPU_NAME` environment variable and `libtpu.so` accessible.
3. **Hexagon:** probes for `libQnnHtp.so` on `LD_LIBRARY_PATH`.
4. **CPU:** always available.

To inspect which backend was selected at runtime:

```java
System.out.println(Nd4j.getBackend().getClass().getName());

// Check device type via DeviceDescriptor
DeviceDescriptor desc = Nd4j.getBackend().getActiveDeviceDescriptor();
System.out.println(desc.getDeviceType());   // e.g. DeviceType.CUDA
System.out.println(desc.getDeviceIndex());  // e.g. 0
```


## 13. GraphExecutionMode Reference

SameDiff graph execution supports 17 execution modes. Modes are set per-graph and control the tradeoff between compilation overhead, runtime speed, device placement, and fallback behavior. This is documented in full in the [SameDiff Execution Modes](../../samediff/execution-modes) page; a condensed reference follows.

| Mode | Description |
|---|---|
| `EAGER` | Execute each op immediately as it is added; no graph compilation |
| `GRAPH` | Build full graph first, then execute; enables fusion |
| `GRAPH_CACHED` | `GRAPH` with compiled program cached by input shapes |
| `JIT_CPU` | JIT-compile graph for CPU; uses MLIR Linalg/arith |
| `JIT_CUDA` | JIT-compile for CUDA; produces PTX |
| `JIT_TPU` | JIT-compile for TPU; produces HLO programs |
| `JIT_HEXAGON` | JIT-compile for Hexagon DSP; produces QNN graph |
| `STREAMING` | Process inputs as a stream; constant memory footprint |
| `BATCHED` | Accumulate inputs and execute in one batched pass |
| `DISTRIBUTED` | Partition graph across multiple devices |
| `ONNX_EXPORT` | Execute and simultaneously export to ONNX |
| `ONNX_IMPORT` | Execute an imported ONNX graph |
| `DEBUG` | Execute with per-op shape and value checks |
| `PROFILE` | Execute with timing and memory usage instrumentation |
| `FALLBACK_CPU` | Attempt preferred device; fall back to CPU on failure |
| `FALLBACK_CHAIN` | Attempt preferred device, then each fallback in priority order |
| `DRY_RUN` | Trace execution without computing output values |

Fallback chain example:

```java
SameDiff sd = SameDiff.create();
sd.setExecutionMode(GraphExecutionMode.FALLBACK_CHAIN);
// Attempts CUDA → TPU → CPU in priority order
```


## 14. Configuration Reference

### System Properties

| Property | Default | Description |
|---|---|---|
| `backend.type` | (auto) | Force a specific backend: `CPU`, `CUDA`, `TPU`, `HEXAGON` |
| `nd4j.tpu.name` | (from `TPU_NAME` env) | TPU resource name for PJRT client |
| `nd4j.tpu.default.dtype` | `BFLOAT16` | Default data type for TPU arrays |
| `nd4j.hexagon.lib.path` | (from `LD_LIBRARY_PATH`) | Override path to QNN libraries |
| `nd4j.zluda.auto.download` | `true` | Whether to auto-download ZLUDA on AMD/Intel GPU |
| `nd4j.device.memory.cap.CUDA.0` | (unlimited) | Per-device allocation cap in bytes |
| `nd4j.multibackend.trace` | `false` | Enable `MultiGpuTracer` for all executions |
| `org.bytedeco.javacpp.maxbytes` | (unlimited) | Off-heap/VRAM cap passed to JavaCPP |

### Environment Variables

| Variable | Description |
|---|---|
| `TPU_NAME` | TPU resource name (`local` for TPU VM, full path for TPU node) |
| `CUDA_VISIBLE_DEVICES` | Restrict CUDA device set visible to the process |
| `LD_LIBRARY_PATH` | Must include QNN libs for Hexagon, cuDNN for CUDA cuDNN, MKL for oneDNN |
| `ZLUDA_DEVICE` | Selects the AMD/Intel device when ZLUDA is active |

### Backend Priority Summary

```
CUDA (100) > HEXAGON (60) > TPU (50) > CPU (0)
```

When the ZLUDA path is active, the CUDA backend handles AMD and Intel GPUs at the same priority (100). When the SDX backend is present, it dispatches internally to CPU/Hexagon/Adreno depending on op type, so its effective priority in the SPI chain is separate from those individual backends.


## See Also

- [Backends Overview](./overview) — SPI mechanism, backend discovery, classpath rules
- [CPU Backend (nd4j-native)](./cpu) — AVX tuning, BLAS configuration, threading
- [CUDA Backend (nd4j-cuda)](./cuda) — CUDA version matrix, cuDNN, multi-GPU, memory management
- [Memory and Workspaces](../workspaces) — off-heap memory, workspace scopes
- [SameDiff Execution Modes](../../samediff/execution-modes) — full GraphExecutionMode documentation

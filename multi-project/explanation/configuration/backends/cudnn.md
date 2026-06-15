---
title: "cuDNN"
description: "cuDNN integration — installation, configuration, and performance benefits"
---

## What Is cuDNN?

cuDNN (CUDA Deep Neural Network library) is NVIDIA's GPU-accelerated library for deep learning primitives. It provides highly optimized implementations of convolutions, pooling, batch normalization, dropout, and RNN operations that are faster than the generic CUDA implementations used by default.

DL4J integrates cuDNN via an optional helper module. When cuDNN is available on the classpath and cuDNN is installed on the host, DL4J uses it automatically for supported layer types without any change to the model definition.

## Supported Layers

The following DL4J layer types benefit from cuDNN acceleration:

- `ConvolutionLayer` (2D convolution)
- `SubsamplingLayer` (pooling)
- `BatchNormalization`
- `Dropout`
- `LocalResponseNormalization`
- `LSTM`

Layers without cuDNN support fall back to the standard CUDA implementation automatically.

## Maven Dependency

Add the cuDNN helper module alongside the CUDA ND4J backend:

```xml
<!-- CUDA ND4J backend (required) -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.6-platform</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>

<!-- cuDNN helper module -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-cuda-11.6</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

The `deeplearning4j-cuda-11.6` module does not bundle the cuDNN library itself — it provides the Java bridge that DL4J uses to call cuDNN. The actual cuDNN shared library must be installed separately on the host.

## Installing cuDNN

Download cuDNN from the NVIDIA Developer Portal:

[https://developer.nvidia.com/cudnn](https://developer.nvidia.com/cudnn)

An NVIDIA Developer account is required. Select the cuDNN version compatible with your installed CUDA version:

| CUDA Version | Supported cuDNN Versions |
|---|---|
| 11.6 | 8.x |
| 11.2 | 8.x |
| 10.2 | 7.6+ |

### Linux Installation

```shell
# Extract the cuDNN archive and copy to the CUDA installation directory
tar -xzvf cudnn-linux-x86_64-*.tar.xz
sudo cp cudnn-linux-x86_64-*/include/cudnn*.h  /usr/local/cuda/include/
sudo cp cudnn-linux-x86_64-*/lib/libcudnn*     /usr/local/cuda/lib64/
sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```

### Windows Installation

Copy the cuDNN DLLs to the CUDA `bin` directory:

```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.6\bin\
```

Copy headers to:
```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.6\include\
```

### Using the JavaCPP Presets (Alternative)

For CUDA 11.x, cuDNN can be bundled via the JavaCPP Presets for CUDA, which include the cuDNN shared libraries as Maven/Gradle dependencies. This avoids a system-level installation but requires accepting the NVIDIA license:

```xml
<!-- Accept license at: https://github.com/bytedeco/javacpp-presets/tree/master/cuda#license-agreements -->
<dependency>
    <groupId>org.bytedeco</groupId>
    <artifactId>cuda</artifactId>
    <version>11.6-8.4-1.5.8</version>
    <classifier>linux-x86_64-redist</classifier>
</dependency>
```

Available classifiers: `linux-x86_64-redist`, `windows-x86_64-redist`, `macosx-x86_64-redist`.

## Verifying cuDNN Is Loaded

After adding the dependency and installing cuDNN, run any inference or training call that exercises a supported layer. If cuDNN is NOT available, you will see:

```
o.d.n.l.c.ConvolutionLayer - cuDNN not found: use cuDNN for better GPU performance by
    including the deeplearning4j-cuda module. For more information, please refer to:
    https://deeplearning4j.org/cudnn
java.lang.ClassNotFoundException: org.deeplearning4j.nn.layers.convolution.CudnnConvolutionHelper
```

If cuDNN is loaded successfully, no warning is logged. You can confirm programmatically:

```java
// Perform at least one forward pass first to trigger helper initialization
model.output(someInput);

LayerHelper helper = net.getLayer(0).getHelper();  // layer 0 must be a ConvolutionLayer
System.out.println("Layer helper: " + (helper == null ? null : helper.getClass().getName()));
// Expected: org.deeplearning4j.nn.layers.convolution.CudnnConvolutionHelper
// Not loaded: null
```

## AlgoMode Configuration

cuDNN supports multiple convolution algorithms with different speed/memory tradeoffs. DL4J exposes this via `ConvolutionLayer.AlgoMode`:

### PREFER_FASTEST (default)

Uses the fastest cuDNN algorithm, determined by a benchmark run. This typically gives the best throughput but can use a lot of workspace memory.

```java
new NeuralNetConfiguration.Builder()
    .cudnnAlgoMode(ConvolutionLayer.AlgoMode.PREFER_FASTEST)
    // ...
```

### NO_WORKSPACE

Uses algorithms that do not require cuDNN workspace memory. Slower than `PREFER_FASTEST`, but avoids the large memory allocations that can cause out-of-memory errors on GPUs with limited VRAM.

```java
// Apply to the whole network
new NeuralNetConfiguration.Builder()
    .cudnnAlgoMode(ConvolutionLayer.AlgoMode.NO_WORKSPACE)
    // ...

// Or apply to a specific layer only
new ConvolutionLayer.Builder(kernelH, kernelW)
    .cudnnAlgoMode(ConvolutionLayer.AlgoMode.NO_WORKSPACE)
    // ...
```

Use `NO_WORKSPACE` when you see errors like:

```
CUDA error: CUDA_ERROR_OUT_OF_MEMORY
cuDNN error: CUDNN_STATUS_ALLOC_FAILED
```

### USER_SPECIFIED

Allows you to specify a particular cuDNN algorithm by index. This is an advanced option for users who have benchmarked specific algorithms on their hardware:

```java
new ConvolutionLayer.Builder(kernelH, kernelW)
    .cudnnAlgoMode(ConvolutionLayer.AlgoMode.USER_SPECIFIED)
    .cudnnFwdAlgoMode(fwdAlgoId)
    .cudnnBwdFilterAlgoMode(bwdFilterAlgoId)
    .cudnnBwdDataAlgoMode(bwdDataAlgoId)
    // ...
```

## Performance Impact

The performance improvement from cuDNN depends on the model architecture, batch size, and GPU. As a rough guide:

| Layer Type | Expected Speedup (vs. plain CUDA) |
|---|---|
| ConvolutionLayer (large kernels) | 2x–5x |
| ConvolutionLayer (3x3, many channels) | 1.5x–3x |
| LSTM | 2x–4x |
| BatchNormalization | 1.5x–2x |
| SubsamplingLayer (pooling) | 1.3x–2x |

These are estimates; actual speedup varies with GPU model, batch size, and input dimensions.

## Troubleshooting

**`java.lang.UnsatisfiedLinkError` at startup**

The cuDNN shared library is not on the library search path. On Linux, add the CUDA lib directory to `LD_LIBRARY_PATH`:

```shell
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

On Windows, ensure the CUDA `bin` directory is on `PATH`.

**`CUDNN_STATUS_NOT_INITIALIZED`**

The cuDNN version installed does not match what the `deeplearning4j-cuda-*` module was compiled against. Verify that the cuDNN major version matches the `deeplearning4j-cuda-*` artifact you are using.

**Out-of-memory errors with `PREFER_FASTEST`**

Switch to `NO_WORKSPACE` mode or reduce batch size. Alternatively, increase the off-heap memory limit:

```shell
-Dorg.bytedeco.javacpp.maxbytes=20G
```

**cuDNN not used even after adding dependency**

Make sure the `deeplearning4j-cuda-*` version exactly matches the `nd4j-cuda-*` and `deeplearning4j-core` version. A version mismatch will prevent the helper from loading.

## Related Pages

- [GPU and CPU Setup](./gpu-cpu) — CUDA backend configuration
- [Memory Configuration](./memory) — managing GPU memory
- [Performance Debugging](./performance-debugging) — identifying cuDNN and performance issues

---
title: Performance and Debugging
description: Diagnosing and resolving performance issues in DL4J and ND4J — profiling, OpProfiler, ETL bottlenecks, GC overhead, and backend verification.
---

## Overview

DL4J and ND4J are built on optimized native C++ code (OpenBLAS, cuDNN, MKL) and should provide excellent
performance in most cases. When performance is below expectations, the cause is usually one of a small number
of well-understood issues. This page walks through them in order from most common to least common.

Performance issues generally appear as:

- Poor CPU or GPU utilization (hardware is idle while training is slow)
- Training or inference taking longer than expected
- Memory errors or excessive GC pauses

---

## Step 1: Verify the Correct Backend Is Active

The most common cause of unexpectedly poor performance is training on CPU when GPU is intended.

ND4J logs the backend at startup:

**CPU backend:**
```
o.n.l.f.Nd4jBackend - Loaded [CpuBackend] backend
o.n.l.a.o.e.DefaultOpExecutioner - Backend used: [CPU]; OS: [Linux]
o.n.l.a.o.e.DefaultOpExecutioner - Blas vendor: [MKL]
```

**GPU backend:**
```
o.n.l.f.Nd4jBackend - Loaded [JCublasBackend] backend
o.n.l.a.o.e.DefaultOpExecutioner - Backend used: [CUDA]; OS: [Linux]
o.n.l.a.o.e.DefaultOpExecutioner - Device Name: [NVIDIA GeForce RTX 3090]; CC: [8.6]
```

Check at runtime:

```java
System.out.println("Backend: " + Nd4j.getBackend().getClass().getSimpleName());
// CPU:  CpuBackend
// GPU:  JCublasBackend
```

If you see `CpuBackend` when GPU is expected, verify:

1. `nd4j-cuda-*-platform` is on the classpath, not `nd4j-native-platform`.
2. Both CPU and CUDA platform artifacts are not present simultaneously — ND4J loads whichever appears first on the classpath, which is non-deterministic.

---

## Step 2: Check for cuDNN (GPU Only)

Without cuDNN, convolution and LSTM layers run at a fraction of peak GPU performance. DL4J logs a warning when a supported layer cannot find cuDNN:

```
o.d.n.l.c.ConvolutionLayer - cuDNN not found: use cuDNN for better GPU performance by
    including the deeplearning4j-cuda module.
```

Confirm programmatically after at least one forward pass:

```java
LayerHelper helper = net.getLayer(0).getHelper();  // layer 0 must be ConvolutionLayer
System.out.println(helper == null ? "cuDNN NOT loaded" : helper.getClass().getName());
// Expected: org.deeplearning4j.nn.layers.convolution.CudnnConvolutionHelper
```

See the [cuDNN page](./cudnn) for installation and dependency instructions.

---

## Step 3: Check for ETL Bottlenecks

If the GPU or CPU is occasionally idle during training, data loading may be the bottleneck. Add `PerformanceListener` to measure ETL time:

```java
net.setListeners(new PerformanceListener(1, true));
```

Sample output:

```
o.d.o.l.PerformanceListener - ETL: 0 ms; iteration 50; iteration time: 65 ms; samples/sec: 492
o.d.o.l.PerformanceListener - ETL: 120 ms; iteration 51; iteration time: 185 ms; samples/sec: 173
```

`ETL` consistently above 0 (after the first iteration) indicates a data loading bottleneck. Common causes:

- Slow disk I/O — use an SSD or pre-load data to RAM.
- Per-iteration image decoding — pre-process and serialize to binary format.
- Complex on-the-fly augmentations — move augmentation offline.
- Network storage (NFS, cloud storage) with high latency.

---

## Step 4: Reduce Garbage Collection Overhead

GC pauses temporarily halt Java threads. Even with off-heap memory for array data, a large number of JVM objects can cause significant GC time.

### Measuring GC Impact

```java
// Enable GC reporting in PerformanceListener
net.setListeners(new PerformanceListener(1, true, true));
```

Output:
```
GC: [G1 Young: 2 (8ms)], [G1 Old: 1 (85ms)]
```

With JVM flags:

```shell
-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps
```

### Reducing GC Impact

Disable or reduce ND4J's periodic `System.gc()` calls:

```java
// At most every 10 seconds
Nd4j.getMemoryManager().setAutoGcWindow(10000);

// Disable entirely (safe when workspaces are ENABLED)
Nd4j.getMemoryManager().togglePeriodicGc(false);
```

Place these calls before `model.fit(...)`. Ensure workspaces are enabled:

```java
System.out.println(net.getLayerWiseConfigurations().getTrainingWorkspaceMode());
// Should print: ENABLED
```

---

## Step 5: Check Minibatch Size

Very small minibatch sizes reduce hardware utilization. General guidelines:

| Device | Recommended minimum batch |
|---|---|
| CPU training | 32 |
| GPU training | 32–256 |
| GPU inference (throughput) | 32–128 |
| GPU inference (latency) | 1 (with `ParallelInference`) |

A batch size of 1 for training is almost always a performance mistake. For low-latency inference from multiple threads, use `ParallelInference`:

```java
ParallelInference pi = new ParallelInference.Builder(net)
    .inferenceMode(InferenceMode.BATCHED)
    .workers(4)
    .build();
```

---

## Step 6: Avoid Using One Model from Multiple Threads

`MultiLayerNetwork` and `ComputationGraph` are not thread-safe. Their `synchronized` methods prevent crashes but serialize all calls, reducing multi-threaded throughput to single-thread levels.

```java
// Correct: one model per thread
ThreadLocal<MultiLayerNetwork> threadModel = ThreadLocal.withInitial(() -> loadModel());
```

---

## Step 7: Verify Data Types

`DataType.DOUBLE` (64-bit) is roughly 2x slower than `DataType.FLOAT` (32-bit) on CPU, and much more on consumer GPUs that lack double-precision hardware.

```java
System.out.println("ND4J DataType: " + Nd4j.dataType());
// Should be: FLOAT

// Change globally if needed (before any network is constructed):
Nd4j.setDefaultDataTypes(DataType.FLOAT, DataType.FLOAT);
```

---

## Step 8: Verify Workspaces Are Enabled

```java
// Check
System.out.println(net.getLayerWiseConfigurations().getTrainingWorkspaceMode());

// Set
net.getLayerWiseConfigurations().setTrainingWorkspaceMode(WorkspaceMode.ENABLED);
net.getLayerWiseConfigurations().setInferenceWorkspaceMode(WorkspaceMode.ENABLED);
```

See [Workspace Configuration](./workspaces) for full details.

---

## Step 9: Check for Network Architecture Bottlenecks

Unusually large networks have legitimate performance costs. Check size with:

```java
System.out.println("Parameters: " + net.numParams());
System.out.println(net.summary());
```

Rough upper limits to investigate before exceeding:

- CNN: ~100 layers
- MLP: ~20 layers, ~2048 units/layer
- RNN/LSTM: ~10 layers

---

## Step 10: Check for CPU-Only Ops (GPU Builds)

Some operations may not yet have GPU kernels. When such ops appear in a model, they execute on CPU even with the CUDA backend, causing device transfers that dominate iteration time. Use `nvidia-smi dmon` to observe if GPU utilization drops during specific iterations.

---

## Step 11: OMP_NUM_THREADS for Concurrent Threads

When many Java threads each run ND4J operations simultaneously (e.g., a multi-threaded inference server), each thread's internal OpenMP parallelism contends for the same CPU cores.

```shell
export OMP_NUM_THREADS=4
```

Rule of thumb: `OMP_NUM_THREADS = ceil(physicalCores / numConcurrentJavaThreads)`.

---

## Step 12: Check Other Processes Using Resources

**CPU:** use `top` (Linux) or Task Manager (Windows).

**GPU:** use `nvidia-smi`:

```shell
nvidia-smi
nvidia-smi dmon     # continuous monitoring
```

If `GPU-Util` is low while training, the GPU is underutilized — likely due to a small batch size, ETL bottleneck, or CPU-only ops.

---

## JVM Profiling

For cases where the above checklist does not identify the problem, profiling provides method-level timing.

### YourKit Java Profiler

[YourKit](https://www.yourkit.com/) supports CPU sampling, tracing, and memory profiling:

1. Install YourKit and attach to the running process.
2. Start a CPU tracing session.
3. Run training for a representative number of iterations.
4. Stop and examine the call tree for hot paths.

### VisualVM

[VisualVM](https://visualvm.github.io/) is free and bundled with the JDK:

1. Start the application.
2. Connect VisualVM to the JVM process.
3. Go to Profiler, start CPU profiling, run the workload, then take a snapshot.

### Profiling on Spark

Attach the YourKit Java agent to Spark executor and driver processes:

```shell
spark-submit \
  --conf 'spark.executor.extraJavaOptions=-agentpath:/opt/yourkit/bin/linux-x86-64/libyjpagent.so=tracing,port=10001,dir=/tmp/yk_snapshots/' \
  --conf 'spark.driver.extraJavaOptions=-agentpath:/opt/yourkit/bin/linux-x86-64/libyjpagent.so=tracing,port=10001,dir=/tmp/yk_snapshots/' \
  ...
```

Snapshots are saved when the job completes.

---

## ND4J OpProfiler

ND4J includes a built-in operation profiler that records timing for every native op call:

```java
// Configure and enable
OpProfiler.getInstance().setConfig(
    ProfilerConfig.builder()
        .notifySingleOpSlow(1)   // warn if any single op > 1ms
        .notifyStacksToNano(true)
        .build()
);
OpProfiler.getInstance().reset();

// Run iterations
for (int i = 0; i < 100; i++) {
    net.fit(dataSet);
}

// Print timing dashboard
OpProfiler.getInstance().printOutDashboard();
```

The dashboard shows cumulative time, call count, and average latency per operation type. The operations at the top with high cumulative time are the primary optimization targets.

---

## Common Anti-Patterns Summary

| Anti-pattern | Symptom | Fix |
|---|---|---|
| `nd4j-native-platform` when GPU intended | Slow training, CPU only | Replace with `nd4j-cuda-*-platform` |
| Both CPU and GPU backends on classpath | Wrong backend loaded | Remove one |
| `WorkspaceMode.NONE` during training | High GC, slow iterations | Switch to `ENABLED` |
| Batch size = 1 for training | Low GPU utilization | Use >= 32 |
| Sharing one model across threads | Serialized throughput | One model per thread or `ParallelInference` |
| `DataType.DOUBLE` | 2x–10x slower on GPU | Use `DataType.FLOAT` |
| Periodic GC enabled during workspace training | Latency spikes | `togglePeriodicGc(false)` |

---

## Related Pages

- [GPU and CPU Setup](./gpu-cpu) — backend selection and CUDA configuration
- [cuDNN](./cudnn) — cuDNN integration for GPU acceleration
- [Memory Configuration](./memory) — JVM and off-heap memory flags
- [Workspace Configuration](./workspaces) — workspace-based memory management

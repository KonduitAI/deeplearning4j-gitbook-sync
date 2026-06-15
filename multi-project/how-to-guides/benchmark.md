---
title: "Benchmarking"
description: "How to benchmark DL4J and ND4J — OpProfiler, performance metrics, and comparing configurations"
---

## Benchmarking DL4J and ND4J

Benchmarking neural network code is harder than benchmarking most software because of JVM warmup, garbage collection pauses, workspace memory learning phases, and the interaction between native libraries, BLAS implementations, and hardware. This guide covers the common pitfalls and the right tools for measuring performance accurately.

---

## Why Benchmark?

Benchmarking answers questions like:

- Is GPU training actually faster than CPU training for my model and batch size?
- Is my data pipeline the bottleneck, or is it the forward/backward pass?
- Will switching from OpenBLAS to MKL improve throughput on CPU?
- Is minibatch size 64, 128, or 256 fastest for my hardware?
- Has a code change regressed performance?

Without careful benchmarking, these questions are answered by intuition, which is frequently wrong for deep learning workloads.

---

## Common Pitfalls

### 1. No JVM Warmup

The first several hundred iterations of any DL4J/ND4J workload are slower than steady-state for three reasons:

1. **JIT compilation** — The JVM interprets bytecode until the JIT compiler has profiled enough executions to compile hot methods to native code.
2. **Library initialization** — ND4J and DL4J perform one-off initialization on the first operation.
3. **Workspace memory learning** — When workspaces are enabled (the default), DL4J observes memory allocation patterns for the first few iterations before settling into an optimal allocation strategy.

**Fix:** Run at least 50–100 warmup iterations before starting any timer. Do not include warmup in your reported numbers.

```java
// Warmup — not timed
for (int i = 0; i < 100; i++) {
    net.fit(trainData.next());
}
trainData.reset();

// Timed benchmark
long start = System.currentTimeMillis();
for (int i = 0; i < numIterations; i++) {
    net.fit(trainData.next());
}
long end = System.currentTimeMillis();
double iterationsPerSec = numIterations / ((end - start) / 1000.0);
```

### 2. Too Few Iterations

Network throughput is not perfectly deterministic due to GC pauses, OS scheduling, and shared resources on cloud hardware. Running only a handful of iterations produces noisy, unreliable numbers.

**Fix:** Run at least 100 timed iterations and report mean ± standard deviation. Without standard deviation, you cannot tell whether two configurations differ in performance or are within noise.

### 3. Measuring the Wrong Thing

Verify that your timer wraps exactly the code you intend to benchmark. Common accidental inclusions:

- JVM startup time
- Library initialization
- Array allocation and zero-initialization
- Data loading and preprocessing (ETL)
- Garbage collection pauses (especially if GC triggers inside the timing window)

If you are benchmarking only the neural network forward/backward pass, pre-allocate all arrays and pre-fetch your data before starting the timer.

### 4. Wrong Native Libraries

ND4J supports multiple BLAS backends:

- **CPU:** OpenBLAS (default) or Intel MKL. MKL is typically 1.3–8× faster than OpenBLAS depending on array dimensions, though OpenBLAS is faster for some specific shapes.
- **GPU:** CuBLAS (always used when CUDA backend is active). CuDNN additionally accelerates convolution — make sure it is enabled for CNN benchmarks.

ND4J logs which BLAS backend it is using at startup:

```
INFO ~ Blas vendor: [OPENBLAS]
```

If you are comparing DL4J to another framework, make sure both are using the same BLAS library. Otherwise you are measuring the BLAS difference, not the framework difference.

To use MKL, download it from [Intel's website](https://software.intel.com/en-us/mkl) and ensure it is on the library path before ND4J initializes.

### 5. Minibatch Size of 1

GPUs are throughput-optimized for large parallel workloads. A minibatch of 1 is almost always slower on GPU than CPU because the GPU cannot be fully utilized. Do not benchmark with minibatch size 1 unless minibatch size 1 is actually your inference use case.

**Rule of thumb for GPU:** Use minibatch sizes that are multiples of 32 (or at least 8). For CPU training, larger batches reduce Python-to-Java boundary overhead when using SameDiff, and also reduce per-iteration overhead.

### 6. Benchmarking Only One Array Shape

BLAS operation performance is sensitive to matrix dimensions. A benchmark with `[128, 512]` × `[512, 1024]` matrix multiplication does not predict performance for `[1, 512]` × `[512, 1024]`. Run your benchmark with a range of batch sizes and layer sizes to get a complete picture.

---

## JVM Configuration for Benchmarking

### Heap Space

DL4J uses both on-heap and off-heap memory. JavaCPP manages off-heap memory (including GPU memory) and uses the JVM GC to trigger deallocation. Setting heap too low causes frequent GC, which introduces pauses into your timing.

Set `-Xms` and `-Xmx` to the same value to avoid gradual heap expansion overhead:

```
java -Xms4g -Xmx4g -cp ... YourBenchmarkClass
```

A common starting point is half your available RAM.

### Garbage Collection

Use the G1GC garbage collector, which provides more predictable pause behavior:

```
java -XX:+UseG1GC -Xms4g -Xmx4g ...
```

For benchmarks where GC pauses would corrupt your timing, use `-Xlog:gc` to log GC events and exclude iterations that had a GC pause from your statistics.

---

## PerformanceListener

`PerformanceListener` is a DL4J training listener that logs throughput (examples/sec and iterations/sec) to the console at a configurable frequency. It is the easiest way to measure training throughput without writing a custom benchmark loop.

```java
import org.deeplearning4j.optimize.listeners.PerformanceListener;

// Report throughput every 10 iterations
net.addListeners(new PerformanceListener(10, true));

// Train normally
net.fit(dataSetIterator, numEpochs);
```

Output example:
```
o.d.o.l.PerformanceListener - Iteration 10, thread 1:
    Score: 1.2345, examples/sec: 4821.3, batches/sec: 75.3
```

`PerformanceListener` is also useful for detecting ETL bottlenecks: if `examples/sec` is lower than expected for your hardware, and GPU utilization is low, the data loading pipeline is likely the bottleneck.

---

## Profiling with OpProfiler

ND4J's `OpProfiler` records timing and call counts for every native operation, allowing you to identify which ops are taking the most time.

```java
import org.nd4j.linalg.profiler.OpProfiler;

// Enable profiling
OpProfiler.getInstance().reset();

// Run your workload
net.fit(data);

// Print the profiling report
OpProfiler.getInstance().printOutDashboard();
```

The report shows each op sorted by total time, with call count and mean/max latency. This is useful for:

- Finding which layer types are the bottleneck (e.g., a specific activation function or normalization layer)
- Verifying that CuDNN is being used for convolution (CuDNN ops appear as `cudnnConvolutionForward` rather than a generic `conv2d`)
- Identifying ops that are called far more times than expected (indicating a loop or algorithm issue)

For CUDA profiling with external tools (NVIDIA Nsight, nvprof), ND4J exposes NVTX markers that annotate GPU kernel launches with their logical op names.

---

## CPU vs GPU Comparison

When comparing CPU and GPU performance:

- **Favor large batch sizes for GPU.** The GPU becomes worthwhile when the parallelism of matrix operations can be fully exploited. A rule of thumb: for a layer with `N` outputs, you want a minibatch of at least `N/32` to saturate a modern GPU.
- **MKL on CPU** is competitive with GPU for small models and small batch sizes. Do not assume GPU is faster without measuring.
- **Input dimensions matter for CUDA.** Sizes that are even multiples of 32 (or 64) typically perform better on GPU due to warp alignment. Avoid odd sizes in benchmarks unless those are your production sizes.
- **Array order matters.** ND4J defaults to column-major ('f') order for BLAS result arrays (required by CuBLAS). Mismatched array orders between operations add transpose overhead. In benchmarks, track array orders explicitly.

---

## Batch Size Optimization

The optimal batch size is a function of your hardware, model architecture, and training objective. Guidelines:

- Start at 32 and double until you hit memory limits or diminishing throughput returns.
- For GPU: multiples of 32 are generally most efficient. Multiples of 8 are the minimum.
- For training quality: very large batches (> 2048) often require learning rate scaling (e.g., linear scaling rule: multiply LR by `batch_size / base_batch_size`) to maintain convergence speed.
- Measure both throughput (examples/sec) and convergence (accuracy per epoch) — the fastest batch size in examples/sec may not yield the fastest convergence in wall-clock time if it hurts generalization.

---

## ETL Benchmarking and Async Loading

A common mistake when comparing DL4J to Python frameworks is including ETL time in the DL4J timing but not in the Python timing (because Python frameworks are usually compared with pre-cached pickled data). Measure ETL and computation separately.

To detect an ETL bottleneck with `PerformanceListener`: if GPU utilization is low and `PerformanceListener` shows low throughput, the DataSetIterator is the bottleneck.

**Fix: Use AsyncDataSetIterator**

```java
import org.deeplearning4j.datasets.iterator.AsyncDataSetIterator;

DataSetIterator underlying = new ImageDataSetIterator(...);
DataSetIterator async = new AsyncDataSetIterator(underlying, prefetchSize);

net.fit(async, numEpochs);
```

`prefetchSize` controls how many minibatches are pre-fetched in a background thread. A value of 2–8 is typical.

For `ComputationGraph` with `MultiDataSetIterator`:

```java
import org.deeplearning4j.datasets.iterator.AsyncMultiDataSetIterator;
MultiDataSetIterator async = new AsyncMultiDataSetIterator(underlying, prefetchSize);
```

**Fix: Pre-save datasets**

For datasets where preprocessing is expensive, pre-save the transformed DataSet objects to disk and load them directly during training:

```java
// Pre-save (run once)
DataSetIterator raw = new RecordReaderDataSetIterator(...);
int i = 0;
while (raw.hasNext()) {
    DataSet ds = raw.next();
    DataSetWriterIterator.save(ds, new File("presaved/batch_" + i++ + ".bin"));
}

// Load pre-saved (fast at training time)
DataSetIterator presaved = new ExistingMinibatchDataSetIterator(new File("presaved/"));
DataSetIterator async = new AsyncDataSetIterator(presaved, 4);
net.fit(async, numEpochs);
```

---

## Memory Profiling

If you suspect memory pressure is degrading throughput (frequent GC pauses, out-of-memory errors, or slow throughput despite high GPU utilization):

1. **Log GC:** Add `-Xlog:gc` to the JVM arguments and observe how often major collections occur during training.
2. **Check workspace configuration:** See the [Workspaces](config/workspaces.md) guide. The default workspace configuration is good for most cases, but disabling workspaces for debugging can help isolate whether workspace overhead is the issue.
3. **Off-heap monitoring:** JavaCPP off-heap usage is bounded by the heap size (set via `-Xmx`). If you have 16 GB of RAM and set `-Xmx4g`, ND4J's off-heap will not exceed 4 GB.
4. **VisualVM or YourKit:** For deep profiling, attach a Java profiler to your training process to see heap allocation rate, GC frequency, and hot allocation paths.

---

## Reproducibility and Reporting

A benchmark is only useful if it can be reproduced. When reporting results, always include:

- DL4J and ND4J version (include snapshot versions if applicable)
- JVM version and GC configuration
- BLAS backend (MKL or OpenBLAS; CuDNN version for GPU)
- Hardware (CPU model / GPU model, RAM)
- Minibatch size and number of iterations (warmup + timed)
- Mean and standard deviation of the metric

Without these details, another person cannot reproduce your results or identify whether a difference in your benchmarks reflects framework performance or environmental differences.

If you identify a performance bottleneck, open an issue on the [DL4J GitHub](https://github.com/eclipse/deeplearning4j/issues) with a minimal reproducible benchmark. The developers actively investigate and fix performance regressions.

---

## Quick Checklist

- [ ] JVM warmup period of at least 50–100 iterations before timing
- [ ] 100+ timed iterations; report mean and standard deviation
- [ ] Timer wraps only the code you intend to measure
- [ ] Correct BLAS backend confirmed in ND4J startup log
- [ ] CuDNN enabled and confirmed for GPU convolution benchmarks
- [ ] Minibatch size is realistic (not 1 on GPU)
- [ ] Array allocation is outside the timing window
- [ ] ETL is benchmarked separately from computation
- [ ] Results include version, hardware, and BLAS backend information

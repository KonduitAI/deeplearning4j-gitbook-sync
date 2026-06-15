---
title: "Memory Configuration"
description: "JVM memory flags, off-heap configuration, and memory management for ND4J and DL4J"
---

## Overview

DL4J and ND4J use two distinct memory regions:

1. **JVM heap** — managed by the Java garbage collector. Holds Java objects, model configurations, and metadata.
2. **Off-heap memory** — allocated outside the JVM, managed by JavaCPP. Holds all `INDArray` data (tensor contents). This memory is shared with native C++ code and, when using CUDA, with GPU memory.

Understanding both regions and setting appropriate limits is critical to avoiding out-of-memory (OOM) errors and achieving good performance.

## JVM Heap Flags

| Flag | Purpose |
|---|---|
| `-Xms<size>` | Initial JVM heap size. JVM allocates this at startup. |
| `-Xmx<size>` | Maximum JVM heap size. JVM will not exceed this limit. |

Examples:

```shell
-Xms2G -Xmx8G   # Start with 2 GB, allow up to 8 GB
-Xms512m -Xmx2G  # Lightweight process
```

**Recommendation:** Keep the JVM heap relatively small. DL4J's training data and model parameters live in off-heap memory, not on the JVM heap. A typical setting is `Xmx2G` to `Xmx8G`. Setting `Xmx` too high leaves less room for off-heap memory.

## Off-Heap Memory Flags

| Flag | Purpose |
|---|---|
| `-Dorg.bytedeco.javacpp.maxbytes=<size>` | Maximum off-heap memory for JavaCPP (and ND4J). On GPU systems, this also controls how much GPU memory ND4J may allocate. |
| `-Dorg.bytedeco.javacpp.maxphysicalbytes=<size>` | Maximum total process memory. Should be set to `maxbytes + Xmx + overhead`. Optional but useful to prevent runaway allocation. |

Size suffixes: `K`, `M`, `G` (e.g., `8G` = 8 gigabytes).

Example — 1 GB JVM, 2 GB max JVM, 8 GB off-heap, 11 GB total process cap:

```shell
-Xms1G -Xmx2G -Dorg.bytedeco.javacpp.maxbytes=8G -Dorg.bytedeco.javacpp.maxphysicalbytes=11G
```

If `maxbytes` is not set, it defaults to the value of `-Xmx`. This means a process with `-Xmx8G` would allow 8 GB for JVM heap AND 8 GB for off-heap — totalling up to 16 GB of RAM usage.

## Recommended Configurations

### Development workstation (16 GB RAM, CPU only)

```shell
-Xms1G -Xmx4G -Dorg.bytedeco.javacpp.maxbytes=10G -Dorg.bytedeco.javacpp.maxphysicalbytes=14G
```

### Server training (64 GB RAM, CPU only)

```shell
-Xms2G -Xmx8G -Dorg.bytedeco.javacpp.maxbytes=48G -Dorg.bytedeco.javacpp.maxphysicalbytes=58G
```

### GPU training (24 GB VRAM, 64 GB system RAM)

```shell
-Xms2G -Xmx6G -Dorg.bytedeco.javacpp.maxbytes=22G -Dorg.bytedeco.javacpp.maxphysicalbytes=30G
```

Set `maxbytes` slightly below VRAM capacity to leave room for CUDA runtime overhead and cuDNN workspace allocations.

### Inference server (low latency, 32 GB RAM, CPU)

```shell
-Xms512m -Xmx2G -Dorg.bytedeco.javacpp.maxbytes=16G -Dorg.bytedeco.javacpp.maxphysicalbytes=20G
```

## GPU Memory Management

When using the CUDA backend (`nd4j-cuda-*`), off-heap memory is mapped to GPU memory. The `maxbytes` flag controls how much GPU RAM ND4J is permitted to allocate. The GPU and CPU off-heap pools share this limit.

ND4J also allocates a CPU-side off-heap mirror buffer for each GPU array to allow efficient CPU-GPU communication. This is why CPU RAM usage will always be higher than GPU VRAM usage in a CUDA setup.

### Rule of thumb for GPU memory

Set `maxbytes` close to — but not exceeding — your GPU's available VRAM. Subtract approximately 1 GB to 2 GB for CUDA runtime and driver overhead:

```
GPU with 16 GB VRAM: -Dorg.bytedeco.javacpp.maxbytes=14G
GPU with 40 GB VRAM: -Dorg.bytedeco.javacpp.maxbytes=36G
```

### Minimum GPU VRAM requirements

Deep learning workloads generally require:
- 4 GB VRAM minimum (small networks, small batches)
- 8 GB VRAM recommended
- 16 GB+ for large CNNs or transformers with moderate batch sizes

GPUs with less than 2 GB VRAM are not suitable for DL4J training.

### Using HOST_ONLY memory with CUDA

In some cases you may need arrays that reside in CPU RAM even when using the CUDA backend. Use `MirroringPolicy.HOST_ONLY` in a workspace configuration:

```java
WorkspaceConfiguration hostOnlyConfig = WorkspaceConfiguration.builder()
    .policyAllocation(AllocationPolicy.STRICT)
    .policyLearning(LearningPolicy.FIRST_LOOP)
    .policyMirroring(MirroringPolicy.HOST_ONLY)
    .policySpill(SpillPolicy.EXTERNAL)
    .build();

try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace(hostOnlyConfig, "HOST_WS")) {
    INDArray cpuArray = Nd4j.create(10000);
    // cpuArray data stays in CPU RAM, not GPU VRAM
}
```

This is only recommended for in-memory cache scenarios where you use `INDArray.unsafeDuplication()`. Host-only arrays are slow to use in computation because they must be copied to GPU for each operation.

## Memory-Mapped Files

The `nd4j-native` (CPU) backend supports memory-mapped files, allowing you to work with `INDArray` data that exceeds available RAM:

```java
WorkspaceConfiguration mmapConfig = WorkspaceConfiguration.builder()
    .initialSize(1_000_000_000L)  // 1 GB mapped file
    .policyLocation(LocationPolicy.MMAP)
    .build();

try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace(mmapConfig, "MMAP_WS")) {
    INDArray largeArray = Nd4j.create(250_000_000);  // 1 GB float array
    // largeArray data is backed by a temporary mmap file
}
```

The file is created as a temp file and cleaned up when the workspace is closed. Performance is lower than RAM-backed arrays but allows processing datasets that do not fit in memory.

## Garbage Collection Configuration

The JVM garbage collector can cause "stop-the-world" pauses that disrupt training. Since ND4J manages array memory off-heap through workspaces, GC pauses primarily affect the JVM-side object lifecycle.

### ND4J's periodic GC

ND4J calls `System.gc()` periodically to trigger cleanup of `WeakReference` objects that track off-heap allocations. By default this occurs every 5 seconds. During training with workspaces enabled, this is usually unnecessary and can introduce latency.

Reduce GC frequency:

```java
// Call System.gc() at most every 10 seconds (10000 ms)
Nd4j.getMemoryManager().setAutoGcWindow(10000);
```

Disable periodic GC entirely (safe when workspaces are enabled for all operations):

```java
Nd4j.getMemoryManager().togglePeriodicGc(false);
```

Place these calls before `model.fit(...)`.

### JVM GC tuning flags

For training workloads, G1GC is a reasonable default on Java 11+:

```shell
-XX:+UseG1GC
-XX:G1HeapRegionSize=32m
-XX:MaxGCPauseMillis=200
```

If you have a large JVM heap (>16 GB), ZGC or Shenandoah can reduce pause times further:

```shell
# ZGC (Java 15+, low latency)
-XX:+UseZGC

# Shenandoah (OpenJDK, low latency)
-XX:+UseShenandoahGC
```

## Diagnosing OOM Errors

### `Can't allocate [HOST] memory`

```
RuntimeException: Can't allocate [HOST] memory: 1073741824; threadId: 1
```

This means the off-heap memory limit was exceeded. Solutions:

1. Increase `maxbytes`: `-Dorg.bytedeco.javacpp.maxbytes=16G`
2. Enable workspaces so memory is reused instead of newly allocated each iteration.
3. Reduce batch size to lower peak memory usage per iteration.
4. Check for memory leaks: arrays created in loops without a workspace scope will accumulate.

### `CUDA out of memory`

```
org.nd4j.jita.handler.impl.CudaZeroHandler - Can't allocate [DEVICE] memory...
```

This means GPU VRAM was exhausted. Solutions:

1. Reduce batch size.
2. Switch to `NO_WORKSPACE` cuDNN algo mode if using cuDNN, as `PREFER_FASTEST` allocates large workspace buffers.
3. Verify `maxbytes` is not set higher than available VRAM.
4. Check that no leftover arrays from previous iterations are being retained in memory.

### JVM heap OOM

```
java.lang.OutOfMemoryError: Java heap space
```

This is a JVM-side issue, not off-heap. Solutions:

1. Increase `-Xmx`.
2. Check for accumulation of Java objects (e.g., storing DataSet objects in a large list).
3. Use a profiler to identify which objects dominate heap usage.

### Diagnosing with heap dumps

To capture a heap dump for analysis:

```shell
# Get PID
jps -lv

# Create heap dump
jmap -dump:format=b,file=heap.hprof <PID>
```

Open the `.hprof` file in VisualVM or YourKit to see object counts by type.

## Monitoring Memory Usage

### At runtime

```java
// JVM heap
long usedHeap = Runtime.getRuntime().totalMemory() - Runtime.getRuntime().freeMemory();
long maxHeap  = Runtime.getRuntime().maxMemory();
System.out.printf("Heap: %d MB used / %d MB max%n",
    usedHeap / 1_000_000, maxHeap / 1_000_000);

// Off-heap via JavaCPP
long offHeapUsed = Pointer.totalBytes();
System.out.printf("Off-heap: %d MB used%n", offHeapUsed / 1_000_000);
```

### GPU memory

```java
// When using CUDA backend
long[] gpuMem = CudaEnvironment.getInstance()
    .getConfiguration()
    .getAvailableDevices()
    .get(0)
    .getFreeAndTotalMemory();
System.out.printf("GPU free: %d MB / total: %d MB%n",
    gpuMem[0] / 1_000_000, gpuMem[1] / 1_000_000);
```

## Summary of Common Pitfalls

| Pitfall | Effect | Fix |
|---|---|---|
| High `-Xms` + large `-Xmx` on a constrained system | No room for off-heap | Keep `-Xms` small; reduce `-Xmx` |
| No `maxbytes` set | Off-heap defaults to `-Xmx` value, may be insufficient | Set `maxbytes` explicitly |
| `maxbytes` > GPU VRAM | CUDA OOM | Set `maxbytes` to VRAM - 1–2 GB |
| Arrays created outside workspaces in training loop | Slow GC pressure, OOM | Enable workspaces; see [Workspaces](./workspaces) |
| Periodic GC enabled during workspace-based training | Latency spikes | `setAutoGcWindow(10000)` or disable |

## Related Pages

- [Workspace Configuration](./workspaces) — workspace-based memory management
- [GPU and CPU Setup](./gpu-cpu) — backend selection
- [Performance Debugging](./performance-debugging) — diagnosing slowdowns and OOM errors

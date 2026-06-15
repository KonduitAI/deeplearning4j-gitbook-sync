---
title: "Memory and Workspaces"
description: "Off-heap memory management, JVM configuration, workspace modes, and troubleshooting memory issues in ND4J and DL4J"
---

# Memory and Workspaces

Understanding how memory works in the DL4J ecosystem is essential for avoiding out-of-memory errors and getting optimal performance. ND4J uses a fundamentally different memory model than typical Java applications.

## Off-Heap Memory

All `INDArray` data is stored **off-heap** — outside the Java Virtual Machine's managed heap. The JVM holds only a small pointer object; the actual tensor data lives in native memory managed by JavaCPP.

This design has several benefits:

- **BLAS interoperability**: Native memory can be passed directly to OpenBLAS, MKL, and cuBLAS without copying
- **No 2^31 element limit**: Java arrays are limited to ~2.1 billion elements due to int indexing; off-heap has no such limit
- **Reduced GC pressure**: Large arrays don't contribute to GC pause times
- **GPU compatibility**: CUDA memory is inherently off-heap

The downside: you must configure **both** JVM heap memory and off-heap memory.

## JVM Memory Configuration

A typical launch configuration:

```bash
java -Xms2G -Xmx4G \
     -Dorg.bytedeco.javacpp.maxbytes=8G \
     -Dorg.bytedeco.javacpp.maxphysicalbytes=12G \
     -jar your-app.jar
```

### Flags Explained

| Flag | Purpose | Recommendation |
|------|---------|----------------|
| `-Xms` | Initial JVM heap size | Set equal to `-Xmx` to avoid resizing |
| `-Xmx` | Maximum JVM heap size | Keep small relative to total RAM (2-4G usually sufficient) |
| `-Dorg.bytedeco.javacpp.maxbytes` | Maximum off-heap memory for ND4J | Set based on your model and data size |
| `-Dorg.bytedeco.javacpp.maxphysicalbytes` | Maximum total physical memory (heap + off-heap) | Set to ~80% of available RAM |

### Key Rules

1. **Keep `-Xmx` small.** The JVM heap holds configuration objects, iterators, and string data — not tensor data. 2-4G is usually enough.
2. **Allocate most memory off-heap.** Set `maxbytes` to several times `-Xmx`.
3. **Leave room for the OS.** Don't allocate 100% of system RAM. Leave at least 1-2G for the OS and other processes.
4. **For GPU:** `maxbytes` controls both CPU off-heap and GPU memory allocation. On GPU machines, set this based on your GPU VRAM.

### Example Configurations

**Development machine (16G RAM, CPU):**
```bash
-Xms2G -Xmx4G -Dorg.bytedeco.javacpp.maxbytes=8G -Dorg.bytedeco.javacpp.maxphysicalbytes=12G
```

**Training server (64G RAM, GPU with 16G VRAM):**
```bash
-Xms4G -Xmx8G -Dorg.bytedeco.javacpp.maxbytes=40G -Dorg.bytedeco.javacpp.maxphysicalbytes=52G
```

**Small model / embedded (4G RAM):**
```bash
-Xms512M -Xmx1G -Dorg.bytedeco.javacpp.maxbytes=2G -Dorg.bytedeco.javacpp.maxphysicalbytes=3G
```

## Workspaces

Workspaces are ND4J's mechanism for **reusing memory allocations** across iterations of a loop. Instead of allocating and deallocating native memory on every training iteration (which is slow), workspaces pre-allocate a block of memory and recycle it.

### How Workspaces Work

1. First iteration: workspace allocates memory as needed, tracking the total
2. Subsequent iterations: workspace reuses the same memory block, overwriting previous values
3. No allocation/deallocation overhead after the first iteration

This is especially effective for neural network training, where each iteration uses roughly the same amount of memory.

### WorkspaceMode

`WorkspaceMode` is at `org.deeplearning4j.nn.conf.WorkspaceMode`:

| Mode | Description |
|------|-------------|
| `ENABLED` | **Default.** Workspaces active for both training and inference. Best performance. |
| `NONE` | Workspaces disabled. Useful for debugging memory issues. |
| `SINGLE` | *Deprecated.* Use `ENABLED` instead. |
| `SEPARATE` | *Deprecated.* Use `ENABLED` instead. |

Configure via the builder:

```java
new NeuralNetConfiguration.Builder()
    .trainingWorkspaceMode(WorkspaceMode.ENABLED)     // default
    .inferenceWorkspaceMode(WorkspaceMode.ENABLED)    // default
    // ...
```

To disable workspaces for debugging:

```java
new NeuralNetConfiguration.Builder()
    .trainingWorkspaceMode(WorkspaceMode.NONE)
    .inferenceWorkspaceMode(WorkspaceMode.NONE)
    // ...
```

### Manual Workspace Usage

For custom code that needs workspace control:

```java
import org.nd4j.linalg.api.memory.MemoryWorkspace;

// Open a workspace (creates or reuses)
try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace("MY_WORKSPACE")) {
    INDArray arr = Nd4j.create(DataType.FLOAT, 1000, 1000);
    // arr is allocated within the workspace
    // arr's memory is invalid after the try block closes
}

// Reuse the workspace in a loop
for (int i = 0; i < 100; i++) {
    try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
            .getAndActivateWorkspace("LOOP_WS")) {
        INDArray temp = Nd4j.rand(DataType.FLOAT, 1000, 1000);
        // Same memory is reused each iteration — no allocation after iteration 0
    }
}
```

## Scope Panic: Leaked and Outdated Workspace Pointers

When using workspaces (including the default workspace mode in DL4J), you may encounter exceptions like:

```
org.nd4j.linalg.exception.ND4JIllegalStateException:
  Op [set] Y argument uses leaked workspace pointer from workspace [LOOP_EXTERNAL]
```

or:

```
org.nd4j.linalg.exception.ND4JIllegalStateException:
  Op [set] Y argument uses outdated workspace pointer from workspace [LOOP_EXTERNAL]
```

### What These Mean

- **Leaked pointer**: An `INDArray` allocated inside a workspace is being used **after the workspace was closed**. The memory it points to has been reclaimed.
- **Outdated pointer**: An `INDArray` from a **previous iteration** of a workspace loop is being used in the current iteration. The memory has been overwritten.

### Leaked Pointer Example

```java
INDArray leaked;
try (MemoryWorkspace ws = Nd4j.getWorkspaceManager().getAndActivateWorkspace("WS")) {
    leaked = Nd4j.create(DataType.FLOAT, 100);
}
// leaked is now INVALID — workspace is closed, memory reclaimed
leaked.addi(1.0);  // THROWS ND4JIllegalStateException
```

### Outdated Pointer Example

```java
INDArray fromPrevIteration = null;
for (int i = 0; i < 10; i++) {
    try (MemoryWorkspace ws = Nd4j.getWorkspaceManager().getAndActivateWorkspace("WS")) {
        INDArray current = Nd4j.create(DataType.FLOAT, 100);
        if (fromPrevIteration != null) {
            current.add(fromPrevIteration);  // THROWS — fromPrevIteration memory was overwritten
        }
        fromPrevIteration = current;
    }
}
```

### Fixes

**1. Detach from workspace** — copies the array to independent off-heap memory:

```java
INDArray safe;
try (MemoryWorkspace ws = Nd4j.getWorkspaceManager().getAndActivateWorkspace("WS")) {
    INDArray temp = Nd4j.create(DataType.FLOAT, 100);
    safe = temp.detach();   // independent copy, not tied to workspace
}
safe.addi(1.0);  // works fine
```

**2. Allocate outside workspace** — temporarily disable workspace:

```java
try (MemoryWorkspace ws = Nd4j.getWorkspaceManager().getAndActivateWorkspace("WS")) {
    // Workspace-allocated arrays (normal)
    INDArray wsArray = Nd4j.create(DataType.FLOAT, 100);

    // Allocate outside workspace
    try (MemoryWorkspace ignored = Nd4j.getWorkspaceManager().scopeOutOfWorkspaces()) {
        INDArray heapArray = Nd4j.create(DataType.FLOAT, 100);
        // heapArray is NOT in any workspace — safe to use anywhere
    }
}
```

**3. Migrate to parent workspace** — moves the array to an enclosing workspace:

```java
INDArray migrated;
try (MemoryWorkspace outer = Nd4j.getWorkspaceManager().getAndActivateWorkspace("OUTER")) {
    try (MemoryWorkspace inner = Nd4j.getWorkspaceManager().getAndActivateWorkspace("INNER")) {
        INDArray temp = Nd4j.create(DataType.FLOAT, 100);
        migrated = temp.leverage();      // moves to outer workspace
        // or: temp.leverageTo("OUTER")  // explicit target
    }
    migrated.addi(1.0);  // works — migrated lives in OUTER
}
```

### If You Didn't Write Custom Code

If you encounter scope panic using standard DL4J APIs (no custom layers, no custom data pipeline), it is likely a bug. Workarounds:

```java
// Disable workspaces entirely
.trainingWorkspaceMode(WorkspaceMode.NONE)
.inferenceWorkspaceMode(WorkspaceMode.NONE)

// Wrap iterator to prevent async workspace conflicts
DataSetIterator safeIter = new AsyncShieldDataSetIterator(originalIter);
```

## Garbage Collection Tuning

ND4J's off-heap memory is eventually freed by Java's garbage collector when the `INDArray` pointer objects are collected. For tight loops without workspaces, GC may not run frequently enough, causing off-heap memory to grow.

### Periodic GC

ND4J can trigger periodic GC calls:

```java
// Enable periodic GC (default: enabled)
Nd4j.getMemoryManager().togglePeriodicGc(true);

// Set frequency (every N milliseconds)
Nd4j.getMemoryManager().setOccasionalGcFrequency(5000);  // every 5 seconds
```

### Manual GC

For inference workloads with variable batch sizes:

```java
// After processing a large batch
System.gc();
Nd4j.getMemoryManager().invokeGc();
```

## Monitoring Memory Usage

```java
// Check off-heap memory usage
long usedBytes = Nd4j.getMemoryManager().getCurrentWorkspaceAllocations();

// Log workspace info
Nd4j.getWorkspaceManager().printAllocationStatisticsForCurrentThread();
```

## Common Memory Issues and Solutions

| Symptom | Cause | Fix |
|---------|-------|-----|
| `OutOfMemoryError: Java heap space` | JVM heap too small | Increase `-Xmx` |
| `OutOfMemoryError: Direct buffer memory` | Off-heap limit reached | Increase `maxbytes` |
| Gradual memory increase during training | GC not collecting INDArray pointers | Enable periodic GC or use workspaces |
| `ND4JIllegalStateException: leaked workspace` | Array used outside its workspace | Use `.detach()` or `scopeOutOfWorkspaces()` |
| `ND4JIllegalStateException: outdated workspace` | Array from previous iteration | Use `.detach()` before storing across iterations |
| OOM on GPU | VRAM exhausted | Reduce batch size, reduce model size, or use mixed precision |
| Slow first epoch, fast subsequent | Normal — workspace warm-up | Expected behavior; first iteration allocates, subsequent reuse |

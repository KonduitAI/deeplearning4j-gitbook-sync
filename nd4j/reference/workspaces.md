---
title: "Workspaces"
description: "MemoryWorkspace API — configuration, policies, nested workspaces, scope panic, and lifecycle management"
---

# Workspaces

Workspaces are ND4J's mechanism for managing off-heap memory efficiently in cyclical workloads such as neural network training and inference. Instead of allocating and deallocating native memory on every iteration, a workspace pre-allocates a memory block once and reuses it in subsequent loops — eliminating allocation overhead after the first pass.

This page covers the low-level `MemoryWorkspace` API. For a high-level conceptual introduction, see [Memory and Workspaces](../core-concepts/memory-and-workspaces.md).


## Workspace Concepts

### Pre-Allocated Memory Blocks

A workspace is a contiguous block of off-heap (native) memory that ND4J manages outside the JVM heap. When you open a workspace and allocate arrays inside it, those arrays draw from the block's free space rather than making OS-level allocation calls.

The block's lifetime is independent of the JVM's garbage collector. Memory is **not freed** when the Java `INDArray` object is collected — instead, the entire block is reset or destroyed when you explicitly close or destroy the workspace.

### Cyclical Reuse

Workspaces are designed for loops. On the first iteration, ND4J tracks every allocation and records the high-water mark. On subsequent iterations, the workspace's internal position pointer is simply reset to zero, and new allocations overwrite the old data. No `free()` call is made; the same physical bytes are reused.

The result: after warm-up, each iteration runs with near-zero allocation cost.

### Off-Heap Management

Because array data lives outside the JVM heap, several properties follow:

- Arrays allocated inside a workspace are valid only while the workspace is open and in the same iteration cycle.
- Closing a workspace or starting a new iteration **invalidates** all pointers allocated in the previous iteration.
- Workspaces can mirror data to GPU memory (controlled by `MirroringPolicy`), enabling the same lifecycle semantics for CUDA allocations.


## WorkspaceConfiguration

`WorkspaceConfiguration` is the builder-style configuration object that controls how a workspace behaves. Import:

```java
import org.nd4j.linalg.api.memory.conf.WorkspaceConfiguration;
import org.nd4j.linalg.api.memory.enums.AllocationPolicy;
import org.nd4j.linalg.api.memory.enums.LearningPolicy;
import org.nd4j.linalg.api.memory.enums.MirroringPolicy;
import org.nd4j.linalg.api.memory.enums.ResetPolicy;
import org.nd4j.linalg.api.memory.enums.SpillPolicy;
```

### Full Builder Example

```java
WorkspaceConfiguration config = WorkspaceConfiguration.builder()
    .initialSize(100 * 1024 * 1024L)          // 100 MB initial block
    .overallocationLimit(0.2)                  // allow 20% overallocation
    .policyAllocation(AllocationPolicy.OVERALLOCATE)
    .policyLearning(LearningPolicy.FIRST_LOOP)
    .policyMirroring(MirroringPolicy.FULL)
    .policySpill(SpillPolicy.REALLOCATE)
    .policyReset(ResetPolicy.BLOCK_LEFT)
    .build();
```

### Configuration Parameters

#### `.initialSize(long bytes)`

Sets the initial size of the workspace memory block, in bytes. If zero (the default), the workspace starts with no pre-allocated memory and grows as needed during the first iteration (controlled by `LearningPolicy`).

```java
// 50 MB initial block
.initialSize(50 * 1024 * 1024L)

// Let the workspace learn its size from the first loop (start at 0)
.initialSize(0)
```

#### `.overallocationLimit(double fraction)`

When `AllocationPolicy.OVERALLOCATE` is active, this controls how much extra memory to allocate beyond the observed high-water mark. A value of `0.1` means allocate 10% more than the maximum observed usage.

```java
.overallocationLimit(0.1)   // 10% buffer above observed peak
.overallocationLimit(0.3)   // 30% buffer — useful when iteration sizes vary
```

#### `.policyAllocation(AllocationPolicy)`

Governs how the workspace responds when an allocation request cannot be satisfied from the current block. See [AllocationPolicy](#allocationpolicy) below.

#### `.policyLearning(LearningPolicy)`

Governs how (and whether) the workspace learns its required size over iterations. See [LearningPolicy](#learningpolicy) below.

#### `.policyMirroring(MirroringPolicy)`

Controls whether workspace memory is mirrored to device (GPU) memory. See [MirroringPolicy](#mirroringpolicy) below.

#### `.policySpill(SpillPolicy)`

Governs what happens when an allocation exceeds the workspace block even after applying the allocation policy. See [SpillPolicy](#spillpolicy) below.

#### `.policyReset(ResetPolicy)`

Controls what happens to the workspace position pointer at the start of each new iteration cycle. See [ResetPolicy](#resetpolicy) below.


## Policy Enums

### AllocationPolicy

`org.nd4j.linalg.api.memory.enums.AllocationPolicy`

| Value | Behavior |
|-------|----------|
| `STRICT` | Only use exactly as much memory as the block contains. If an allocation cannot fit, defer to `SpillPolicy`. |
| `OVERALLOCATE` | When the block must grow, allocate `(required * (1 + overallocationLimit))` bytes. Reduces future reallocations. |
| `NO_WORKSPACE` | Treat this workspace as a no-op; all allocations fall through to regular off-heap memory. Useful for debugging. |

`OVERALLOCATE` is the most common choice for training loops where iteration sizes are roughly stable but may vary slightly.

```java
.policyAllocation(AllocationPolicy.OVERALLOCATE)
.overallocationLimit(0.15)   // 15% padding
```

### LearningPolicy

`org.nd4j.linalg.api.memory.enums.LearningPolicy`

| Value | Behavior |
|-------|----------|
| `FIRST_LOOP` | Learn the required workspace size during the first iteration. Resize (if necessary) at the end of that loop, then lock the size for all subsequent iterations. |
| `OVER_TIME` | Continuously adapt the workspace size across iterations. Allows the workspace to grow if later iterations require more memory than earlier ones. |
| `NONE` | Do not learn. The workspace uses exactly `initialSize` and never grows beyond it (combined with `SpillPolicy` to handle overflows). |

`FIRST_LOOP` is the most common setting for training, because training iterations are nearly identical in memory footprint.

```java
.policyLearning(LearningPolicy.FIRST_LOOP)
```

Use `OVER_TIME` if your iteration sizes are variable (for example, variable-length sequence batches):

```java
.policyLearning(LearningPolicy.OVER_TIME)
```

### SpillPolicy

`org.nd4j.linalg.api.memory.enums.SpillPolicy`

Controls what happens when an allocation request cannot be satisfied by the existing block, and `LearningPolicy` is `NONE` or the workspace is past its learning phase.

| Value | Behavior |
|-------|----------|
| `EXTERNAL` | Allocate the overflow array outside the workspace (standard off-heap allocation). The array is valid indefinitely, not subject to workspace reset. |
| `REALLOCATE` | Grow the workspace block to accommodate the request and copy existing data. The workspace continues to manage the allocation. |
| `FAIL` | Throw an exception immediately if the block is full. Use this to detect unexpected memory growth. |

```java
// Safe fallback — overflow goes to regular off-heap memory
.policySpill(SpillPolicy.EXTERNAL)

// Strict mode — fail loudly if iteration memory grows unexpectedly
.policySpill(SpillPolicy.FAIL)
```

### ResetPolicy

`org.nd4j.linalg.api.memory.enums.ResetPolicy`

Controls how the workspace position pointer is managed when the workspace is reset (i.e., at the start of each iteration cycle).

| Value | Behavior |
|-------|----------|
| `BLOCK_LEFT` | Reset the position pointer to zero, treating the entire block as available. All previous data is considered invalid. This is the standard mode for training loops. |
| `ENDOFBUFFER_REACHED` | A circular-buffer mode: wrap around to the beginning only after reaching the end of the block. Used for circular (RNN) workspaces. |

```java
// Standard: every iteration starts fresh from position 0
.policyReset(ResetPolicy.BLOCK_LEFT)

// Circular: wrap around — used for time-series / RNN rolling windows
.policyReset(ResetPolicy.ENDOFBUFFER_REACHED)
```

### MirroringPolicy

`org.nd4j.linalg.api.memory.enums.MirroringPolicy`

Controls whether the workspace is mirrored to device memory (relevant for CUDA backends).

| Value | Behavior |
|-------|----------|
| `FULL` | Mirror the entire workspace to device memory. Both host and device copies are kept in sync. |
| `HOST_ONLY` | Only allocate on the host (CPU). No device mirror is created. |

```java
// GPU training — mirror workspace to CUDA device memory
.policyMirroring(MirroringPolicy.FULL)

// CPU-only training
.policyMirroring(MirroringPolicy.HOST_ONLY)
```


## Creating and Using Workspaces

The entry point is `Nd4j.getWorkspaceManager()`, which returns the thread-local `MemoryWorkspaceManager`.

### Basic Usage: Try-With-Resources

```java
import org.nd4j.linalg.api.memory.MemoryWorkspace;
import org.nd4j.linalg.factory.Nd4j;

WorkspaceConfiguration config = WorkspaceConfiguration.builder()
    .initialSize(10 * 1024 * 1024L)          // 10 MB
    .policyAllocation(AllocationPolicy.OVERALLOCATE)
    .policyLearning(LearningPolicy.FIRST_LOOP)
    .policySpill(SpillPolicy.EXTERNAL)
    .policyReset(ResetPolicy.BLOCK_LEFT)
    .build();

try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace(config, "TRAINING_WS")) {

    INDArray weights = Nd4j.rand(DataType.FLOAT, 512, 256);
    INDArray bias    = Nd4j.zeros(DataType.FLOAT, 256);
    INDArray output  = weights.mmul(Nd4j.rand(DataType.FLOAT, 256, 32)).addRowVector(bias);

    // weights, bias, and output are all workspace-allocated
    // Their memory is valid only within this try block
}
// workspace is closed here; all arrays allocated above are now invalid
```

`getAndActivateWorkspace(config, name)` opens the named workspace, creating it with the given configuration on the first call. On subsequent calls with the same name, it reuses the existing workspace and resets its position pointer according to `ResetPolicy`.

### Usage in a Training Loop

```java
WorkspaceConfiguration loopConfig = WorkspaceConfiguration.builder()
    .initialSize(0)                                    // learn size on first iteration
    .policyAllocation(AllocationPolicy.OVERALLOCATE)
    .overallocationLimit(0.1)
    .policyLearning(LearningPolicy.FIRST_LOOP)
    .policySpill(SpillPolicy.REALLOCATE)
    .policyReset(ResetPolicy.BLOCK_LEFT)
    .build();

for (int epoch = 0; epoch < numEpochs; epoch++) {
    while (dataIterator.hasNext()) {
        DataSet batch = dataIterator.next();

        try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
                .getAndActivateWorkspace(loopConfig, "TRAINING_LOOP")) {

            INDArray features = batch.getFeatures();
            INDArray labels   = batch.getLabels();
            // forward pass, backward pass, parameter update ...
            // all intermediate arrays reuse the same workspace memory each iteration

            // If you need a result that survives past the try block:
            INDArray persistentResult = someArray.detach();
        }
        // workspace resets here; next iteration reuses the same block
    }
}
```

### Opening a Workspace Without a Configuration

If a workspace with the given name already exists (e.g., created earlier in the same thread), you can open it without passing a config:

```java
try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace("TRAINING_LOOP")) {
    // uses the existing workspace's configuration
}
```


## Nested Workspaces

Workspaces can be nested. The innermost open workspace is the **active** workspace, meaning new allocations go into it. When the inner workspace closes, the previously open outer workspace becomes active again.

```java
WorkspaceConfiguration outerConfig = WorkspaceConfiguration.builder()
    .initialSize(50 * 1024 * 1024L)
    .policyAllocation(AllocationPolicy.OVERALLOCATE)
    .policyLearning(LearningPolicy.FIRST_LOOP)
    .policyReset(ResetPolicy.BLOCK_LEFT)
    .build();

WorkspaceConfiguration innerConfig = WorkspaceConfiguration.builder()
    .initialSize(10 * 1024 * 1024L)
    .policyAllocation(AllocationPolicy.OVERALLOCATE)
    .policyLearning(LearningPolicy.FIRST_LOOP)
    .policyReset(ResetPolicy.BLOCK_LEFT)
    .build();

try (MemoryWorkspace outer = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace(outerConfig, "OUTER")) {

    INDArray outerArray = Nd4j.rand(DataType.FLOAT, 1000, 1000);  // allocated in OUTER

    try (MemoryWorkspace inner = Nd4j.getWorkspaceManager()
            .getAndActivateWorkspace(innerConfig, "INNER")) {

        INDArray innerArray = Nd4j.rand(DataType.FLOAT, 100, 100);  // allocated in INNER

        // Move innerArray to outer workspace so it survives inner closing:
        INDArray promoted = innerArray.leverage();   // now lives in OUTER
    }
    // INNER is closed; innerArray is invalid, promoted is still valid

    outerArray.addi(promoted);   // valid — both are in OUTER
}
// OUTER is closed; all arrays invalid
```

### Parent/Child Relationships

The workspace manager maintains a stack of active workspaces per thread. `leverage()` always targets the immediately enclosing parent. Use `leverageTo(String name)` when you need to target a specific ancestor:

```java
try (MemoryWorkspace grandparent = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace(config, "GRANDPARENT")) {
    try (MemoryWorkspace parent = Nd4j.getWorkspaceManager()
            .getAndActivateWorkspace(config, "PARENT")) {
        try (MemoryWorkspace child = Nd4j.getWorkspaceManager()
                .getAndActivateWorkspace(config, "CHILD")) {

            INDArray arr = Nd4j.rand(DataType.FLOAT, 64, 64);

            INDArray inParent      = arr.leverage();             // moves to PARENT
            INDArray inGrandparent = arr.leverageTo("GRANDPARENT");  // skips to GRANDPARENT
        }
    }
}
```


## Scope Panic

**Scope panic** is the term for the family of `ND4JIllegalStateException` exceptions thrown when an `INDArray` allocated inside a workspace is used incorrectly. ND4J validates array pointers at op execution time (when profiling mode is active) to catch these errors early rather than producing silent data corruption.

### Leaked Workspace Pointer

```
org.nd4j.linalg.exception.ND4JIllegalStateException:
  Op [set] Y argument uses leaked workspace pointer from workspace [LOOP_EXTERNAL]
  For more details, see the ND4J User Guide: nd4j.org/userguide#workspaces-panic
```

A **leaked pointer** means an `INDArray` that was allocated inside a workspace is being used **after that workspace was closed**. The memory it points to has been reclaimed by the workspace and may have been overwritten.

Event sequence:
1. Workspace `W` is opened.
2. `INDArray X` is allocated in workspace `W`.
3. Workspace `W` is closed — the memory backing `X` is now invalid.
4. `X` is passed to an op — exception thrown.

```java
INDArray leaked;
try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace(config, "WS")) {
    leaked = Nd4j.rand(DataType.FLOAT, 100);
}
leaked.addi(1.0f);  // THROWS: leaked workspace pointer
```

### Outdated Workspace Pointer

```
org.nd4j.linalg.exception.ND4JIllegalStateException:
  Op [set] Y argument uses outdated workspace pointer from workspace [LOOP_EXTERNAL]
  For more details, see the ND4J User Guide: nd4j.org/userguide#workspaces-panic
```

An **outdated pointer** means an `INDArray` from a **previous iteration** of a workspace loop is being used in the current iteration. The workspace has been reset, overwriting that memory with new data.

Event sequence:
1. Workspace `W` is opened (iteration 1).
2. `INDArray X` is allocated in workspace `W` (iteration 1).
3. Workspace `W` is closed (iteration 1).
4. Workspace `W` is opened again (iteration 2) — position pointer resets.
5. `X` (from iteration 1) is used — its memory has been overwritten. Exception thrown.

```java
INDArray saved = null;
for (int i = 0; i < 10; i++) {
    try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
            .getAndActivateWorkspace(config, "LOOP")) {
        INDArray current = Nd4j.rand(DataType.FLOAT, 100);
        if (saved != null) {
            current.add(saved);  // THROWS on i=1: saved is from previous iteration
        }
        saved = current;         // storing workspace-allocated array across iterations
    }
}
```


## Fixes for Scope Panic

### 1. `detach()` — Copy to Independent Memory

`INDArray.detach()` returns a copy of the array that is not associated with any workspace. It lives in regular off-heap memory and remains valid indefinitely (until GC reclaims the Java pointer object).

```java
INDArray safe;
try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace(config, "WS")) {
    INDArray temp = Nd4j.rand(DataType.FLOAT, 512, 512);
    safe = temp.detach();   // independent copy — no workspace association
}
safe.addi(1.0f);   // fine — not tied to any workspace
```

Use `detach()` when you need a result from inside a workspace to persist across iterations or be returned from a method.

### 2. `leverage()` — Move to Enclosing Parent Workspace

`INDArray.leverage()` copies the array into the immediately enclosing (parent) workspace. The result lives in the parent's lifetime rather than the child's.

```java
try (MemoryWorkspace outer = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace(config, "OUTER")) {
    INDArray inOuter;
    try (MemoryWorkspace inner = Nd4j.getWorkspaceManager()
            .getAndActivateWorkspace(config, "INNER")) {
        INDArray temp = Nd4j.rand(DataType.FLOAT, 256);
        inOuter = temp.leverage();   // now lives in OUTER, not INNER
    }
    inOuter.addi(1.0f);   // valid — OUTER is still open
}
```

### 3. `leverageTo(String)` — Move to a Named Workspace

When the target workspace is not the immediate parent, use `leverageTo(String name)` to specify it by name:

```java
INDArray result = someArray.leverageTo("TRAINING_LOOP");
```

If the named workspace is not currently open on the calling thread, `leverageTo` falls back to a regular `detach()`.

### 4. `migrate()` — Move to the Currently Active Workspace

`INDArray.migrate()` copies the array into whichever workspace is currently active on the calling thread. If no workspace is active, it behaves like `detach()`.

```java
try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace(config, "TARGET")) {
    INDArray migrated = arrayFromElsewhere.migrate();
    // migrated is now in TARGET
}
```

### 5. `scopeOutOfWorkspaces()` — Allocate Outside All Workspaces

Use `Nd4j.getWorkspaceManager().scopeOutOfWorkspaces()` to temporarily suspend all active workspaces. Arrays created inside this scope use standard off-heap allocation and are not subject to workspace reset.

```java
try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
        .getAndActivateWorkspace(config, "LOOP")) {

    INDArray workspaceArray = Nd4j.rand(DataType.FLOAT, 1000);

    try (MemoryWorkspace ignored = Nd4j.getWorkspaceManager().scopeOutOfWorkspaces()) {
        // All workspaces suspended inside this block
        INDArray persistent = Nd4j.zeros(DataType.FLOAT, 10);
        // persistent lives in regular off-heap memory — valid after LOOP closes
    }
    // LOOP workspace is active again
}
```

`scopeOutOfWorkspaces` is particularly useful for allocating arrays that will be returned from a method or stored as instance fields, while the body of the method operates inside a workspace.

### Disabling Scope Panic Validation

If you have verified your code is correct and want to eliminate the validation overhead, you can disable scope panic detection. **This is not recommended for production code** — a real issue will cause the JVM to crash rather than throw a controlled exception.

```java
import org.nd4j.linalg.api.ops.executioner.OpExecutioner;

Nd4j.getExecutioner().setProfilingMode(OpExecutioner.ProfilingMode.DISABLED);
```

To re-enable:

```java
Nd4j.getExecutioner().setProfilingMode(OpExecutioner.ProfilingMode.SCOPE_PANIC);
```


## Circular Workspaces for RNN Training

Recurrent neural networks require holding activations from multiple timesteps simultaneously (for backpropagation through time). A circular workspace (`ENDOFBUFFER_REACHED` reset policy) handles this naturally by allocating timestep activations sequentially across a ring buffer rather than overwriting them each step.

```java
WorkspaceConfiguration rnnConfig = WorkspaceConfiguration.builder()
    .initialSize(200 * 1024 * 1024L)              // 200 MB ring buffer
    .policyAllocation(AllocationPolicy.STRICT)
    .policyLearning(LearningPolicy.NONE)           // fixed size ring buffer
    .policyReset(ResetPolicy.ENDOFBUFFER_REACHED)  // wrap around, not overwrite
    .policySpill(SpillPolicy.FAIL)                 // fail loudly if ring is too small
    .build();

String WS_RNN = "RNN_CIRCULAR";

for (int minibatch = 0; minibatch < numMinibatches; minibatch++) {
    // Open the circular workspace once per minibatch
    try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
            .getAndActivateWorkspace(rnnConfig, WS_RNN)) {

        List<INDArray> timestepActivations = new ArrayList<>();

        for (int t = 0; t < sequenceLength; t++) {
            INDArray activation = computeTimestep(t);    // allocated in ring buffer
            timestepActivations.add(activation);          // all timesteps valid simultaneously
        }

        // BPTT — all timesteps are accessible because the ring buffer has not wrapped
        backpropagateThroughTime(timestepActivations);
    }
    // ring buffer resets; all timestep arrays are now invalid
}
```

The key property of the circular mode: within a single open/close cycle, allocations accumulate from position 0 up to the end of the buffer. No overwriting occurs mid-cycle. When the workspace is closed and reopened for the next minibatch, the position resets to 0.

Size the ring buffer to hold all timestep activations for one minibatch: `timesteps * batchSize * hiddenSize * bytesPerElement`.


## Workspace Statistics

`Nd4j.getWorkspaceManager().printAllocationStatisticsForCurrentThread()` prints a summary of all workspaces opened on the current thread, including their sizes, peak usage, and spill counts. This is useful for sizing `initialSize` appropriately.

```java
// After running a few training iterations:
Nd4j.getWorkspaceManager().printAllocationStatisticsForCurrentThread();
```

Example output:

```
Workspace: TRAINING_LOOP
  Current size:  104857600 bytes (100 MB)
  Peak usage:    87654321 bytes (83.6 MB)
  Spill count:   0
  Cycles:        150
```

Use the peak usage figure to set `initialSize` plus a buffer matching `overallocationLimit`. If spill count is non-zero, the workspace block is undersized for that workload.


## Destroying Workspaces

Workspaces persist on a thread until they are explicitly destroyed, even after all `try` blocks have closed. Destroying a workspace frees the underlying native memory block entirely.

### Destroy a Single Workspace

```java
// Destroy a specific workspace by name on the current thread
Nd4j.getWorkspaceManager().destroyWorkspace(
    Nd4j.getWorkspaceManager().getWorkspaceForCurrentThread("TRAINING_LOOP")
);
```

Or retrieve and destroy in one step:

```java
MemoryWorkspace ws = Nd4j.getWorkspaceManager()
    .getWorkspaceForCurrentThread(config, "TRAINING_LOOP");
Nd4j.getWorkspaceManager().destroyWorkspace(ws);
```

### Destroy All Workspaces on the Current Thread

```java
Nd4j.getWorkspaceManager().destroyAllWorkspacesForCurrentThread();
```

This frees every workspace block that has been created on the calling thread. Call this at the end of a worker thread's lifecycle to avoid native memory leaks in long-running server applications with thread pools.

### Typical Lifecycle for a Worker Thread

```java
// Thread body
try {
    WorkspaceConfiguration config = WorkspaceConfiguration.builder()
        .initialSize(0)
        .policyAllocation(AllocationPolicy.OVERALLOCATE)
        .policyLearning(LearningPolicy.FIRST_LOOP)
        .policyReset(ResetPolicy.BLOCK_LEFT)
        .build();

    for (DataSet batch : dataSource) {
        try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
                .getAndActivateWorkspace(config, "WORKER_WS")) {
            processBatch(batch);
        }
    }
} finally {
    // Always clean up workspace memory when the thread exits
    Nd4j.getWorkspaceManager().destroyAllWorkspacesForCurrentThread();
}
```

Failure to call `destroyAllWorkspacesForCurrentThread()` on pool threads will cause native memory to accumulate, since thread-pool threads are reused and their workspaces are never GC'd.


## Summary of Key Methods

| Method | Description |
|--------|-------------|
| `Nd4j.getWorkspaceManager().getAndActivateWorkspace(config, name)` | Open (or reuse) a named workspace with the given configuration |
| `Nd4j.getWorkspaceManager().getAndActivateWorkspace(name)` | Open a previously created workspace by name |
| `Nd4j.getWorkspaceManager().scopeOutOfWorkspaces()` | Suspend all workspaces; allocations fall through to regular off-heap |
| `Nd4j.getWorkspaceManager().printAllocationStatisticsForCurrentThread()` | Print per-workspace usage statistics for the current thread |
| `Nd4j.getWorkspaceManager().destroyWorkspace(ws)` | Free a workspace's native memory block |
| `Nd4j.getWorkspaceManager().destroyAllWorkspacesForCurrentThread()` | Free all workspace blocks on the current thread |
| `arr.detach()` | Copy array to independent off-heap memory; no workspace association |
| `arr.leverage()` | Copy array into the immediately enclosing parent workspace |
| `arr.leverageTo(String name)` | Copy array into a named ancestor workspace |
| `arr.migrate()` | Copy array into the currently active workspace (or detach if none) |

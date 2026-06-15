---
title: "Workspace Configuration"
description: "WorkspaceMode configuration for training and inference memory management"
---

## What Are Workspaces?

ND4J workspaces are a memory management model designed for cyclic workloads like neural network training and inference. Instead of allocating and releasing off-heap memory for each `INDArray` independently (which triggers garbage collection), workspaces maintain a fixed-size memory region that is reused across iterations.

At the end of each workspace cycle, all `INDArray` objects created within the workspace have their memory invalidated and made available for reuse in the next cycle. No deallocation or GC pressure occurs. This makes workspace-based memory management significantly faster and more predictable than relying on the JVM GC for off-heap memory reclamation.

DL4J enables workspaces by default for all `MultiLayerNetwork` and `ComputationGraph` instances from version 1.0.0-alpha onwards. For most users, no explicit configuration is needed.

## WorkspaceMode Enum

`WorkspaceMode` controls how workspaces are used during training and inference.

### ENABLED (recommended)

```java
new NeuralNetConfiguration.Builder()
    .trainingWorkspaceMode(WorkspaceMode.ENABLED)
    .inferenceWorkspaceMode(WorkspaceMode.ENABLED)
    // ...
```

`ENABLED` is the default and the recommended setting. All intermediate activations, gradients, and updater state during a training or inference pass are allocated within a workspace and reused across iterations. This results in:

- Minimal GC pressure during training
- Consistent per-iteration memory usage after the first few warmup iterations
- Better performance than `NONE`

### NONE

```java
new NeuralNetConfiguration.Builder()
    .trainingWorkspaceMode(WorkspaceMode.NONE)
    .inferenceWorkspaceMode(WorkspaceMode.NONE)
    // ...
```

`NONE` disables workspace-based memory management entirely. Every `INDArray` is allocated independently and freed by the JVM GC when no longer referenced. This is slower and less memory-efficient but is occasionally useful for debugging.

When to use `NONE`:
- Debugging a `ND4JIllegalStateException` related to workspace pointer leaks
- Isolating whether a bug is workspace-related or not
- Custom layers or data pipelines that are not workspace-aware

### SEPARATE and SINGLE (deprecated)

Prior to 1.0.0-alpha, DL4J used `SEPARATE` and `SINGLE` modes:

- `SEPARATE` — used separate workspaces for different stages of the forward/backward pass. Slightly slower, less memory.
- `SINGLE` — used one workspace for all stages. Slightly faster, more memory.

Both are deprecated and functionally replaced by `ENABLED`. Do not use them in new code.

## Checking and Changing Workspace Configuration

### MultiLayerNetwork

```java
// Check current workspace mode
System.out.println("Training workspace: " +
    net.getLayerWiseConfigurations().getTrainingWorkspaceMode());
System.out.println("Inference workspace: " +
    net.getLayerWiseConfigurations().getInferenceWorkspaceMode());

// Change workspace mode on an existing network
net.getLayerWiseConfigurations().setTrainingWorkspaceMode(WorkspaceMode.ENABLED);
net.getLayerWiseConfigurations().setInferenceWorkspaceMode(WorkspaceMode.ENABLED);
```

### ComputationGraph

```java
System.out.println("Training workspace: " +
    cg.getConfiguration().getTrainingWorkspaceMode());
System.out.println("Inference workspace: " +
    cg.getConfiguration().getInferenceWorkspaceMode());

cg.getConfiguration().setTrainingWorkspaceMode(WorkspaceMode.ENABLED);
cg.getConfiguration().setInferenceWorkspaceMode(WorkspaceMode.ENABLED);
```

## Garbage Collector Interaction

When workspaces are enabled, ND4J's periodic `System.gc()` calls become less important because off-heap memory is managed by the workspace rather than by GC-driven WeakReference cleanup. It is beneficial to reduce or disable them:

```java
// Reduce GC call frequency to once every 10 seconds
Nd4j.getMemoryManager().setAutoGcWindow(10000);

// Or disable periodic GC entirely (safe with ENABLED workspaces)
Nd4j.getMemoryManager().togglePeriodicGc(false);
```

Place these before `model.fit(...)` or before the inference loop.

## ParallelWrapper and Multi-GPU

`ParallelWrapper` uses one workspace per training thread:

```java
ParallelWrapper wrapper = new ParallelWrapper.Builder(model)
    .prefetchBuffer(8)
    .workers(4)  // one per GPU
    .averagingFrequency(3)
    .workspaceMode(WorkspaceMode.ENABLED)
    .build();
```

## Moving Arrays Out of a Workspace

Arrays created inside a workspace are valid only within the workspace's scope. If you need to retain an array after the workspace cycle ends, call `detach()`:

```java
INDArray predictions = model.output(input);
// 'predictions' is detached from the workspace by model.output() automatically.

// For custom code inside a workspace scope:
try (MemoryWorkspace ws = Nd4j.getWorkspaceManager().getAndActivateWorkspace("MY_WS")) {
    INDArray temp = Nd4j.create(1000);
    INDArray persistent = temp.detach();  // copy out of workspace into GC-managed memory
}
// temp is now invalidated; persistent is still valid
```

## Async Iterators and Workspaces

`AsyncDataSetIterator` and `AsyncMultiDataSetIterator` use a cyclic workspace internally to prefetch minibatches. This means the `DataSet` objects returned by `next()` are workspace-managed and will be overwritten on subsequent `next()` calls.

**Important:** Consume each `DataSet` from an async iterator before calling `next()` again. Do not store `DataSet` references from an async iterator without calling `detach()` on its arrays first.

If you need to disable async prefetch for debugging:

```java
DataSetIterator wrapped = new AsyncShieldDataSetIterator(yourIterator);
```

## Destroying Workspaces

In low-memory situations or between training runs, you can release all workspaces for the current thread:

```java
Nd4j.getWorkspaceManager().destroyAllWorkspacesForCurrentThread();
```

To release workspaces in a specific thread from another thread, call the same method from within that thread.

## Debugging Workspace Issues

If you see an error like:

```
org.nd4j.linalg.exception.ND4JIllegalStateException: Op [set] Y argument uses leaked workspace
pointer from workspace [LOOP_EXTERNAL]
For more details, see the ND4J User Guide: nd4j.org/userguide#workspaces-panic
```

This means an array allocated inside a workspace is being used outside its scope. Common causes:

1. A custom layer is storing an activation array reference across iterations without calling `detach()`.
2. A custom data iterator returns arrays backed by workspace memory that is later invalidated.
3. An array is passed to code that stores it, such as a listener or evaluator that retains it beyond the iteration.

**Debugging approach:**

1. Temporarily switch to `WorkspaceMode.NONE` to see if the error disappears. If it does, the issue is a workspace scope violation.
2. Audit custom layer code: all returned arrays from `activate()` should be in `ArrayType.ACTIVATIONS`, and arrays from `backpropGradient()` should be in `ArrayType.ACTIVATION_GRAD`, using `LayerWorkspaceMgr`.
3. Use `LayerWorkspaceMgr.noWorkspaces()` when doing forward passes outside of a `MultiLayerNetwork`/`ComputationGraph` context.

## Evaluation Without Creating New Arrays

The preferred way to evaluate a model during training avoids creating extra `INDArray` allocations:

```java
Evaluation eval = new Evaluation(numClasses);
ROC roc = new ROC(100);

// Runs a single pass over the iterator, updating all IEvaluation instances
model.doEvaluation(testIterator, eval, roc);

System.out.println(eval.stats());
```

This is more efficient than calling `model.output()` in a loop and storing predictions.

## Custom Workspace Usage

For custom cyclic workloads outside of DL4J's training loop:

```java
WorkspaceConfiguration config = WorkspaceConfiguration.builder()
    .initialSize(100 * 1024 * 1024L)  // 100 MB initial size
    .overallocationLimit(0.1)          // Allow 10% overallocation
    .policyAllocation(AllocationPolicy.OVERALLOCATE)
    .policyLearning(LearningPolicy.FIRST_LOOP)
    .policySpill(SpillPolicy.REALLOCATE)
    .policyReset(ResetPolicy.BLOCK_LEFT)
    .build();

for (int i = 0; i < numIterations; i++) {
    try (MemoryWorkspace ws = Nd4j.getWorkspaceManager()
            .getAndActivateWorkspace(config, "MY_CYCLIC_WS")) {
        // All Nd4j.create() calls here are allocated in the workspace
        INDArray a = Nd4j.create(1000);
        INDArray b = Nd4j.create(1000);
        INDArray result = a.add(b);
        // result is valid until end of this try block
    }
    // workspace memory is reset for next iteration; no GC needed
}
```

## Related Pages

- [Memory Configuration](./memory) — JVM and off-heap memory flags
- [Performance Debugging](./performance-debugging) — diagnosing GC and memory issues
- [GPU and CPU Setup](./gpu-cpu) — backend setup

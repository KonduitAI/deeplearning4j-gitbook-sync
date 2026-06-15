---
title: ParallelWrapper and ParallelInference
description: Multi-GPU training with ParallelWrapper and high-throughput inference with ParallelInference in DL4J 1.0.0-M2.1.
---

## ParallelWrapper and ParallelInference

`ParallelWrapper` and `ParallelInference` are DL4J's tools for taking advantage of multiple GPUs (or CPU cores) on a single machine — without the complexity of a Spark cluster.

- **ParallelWrapper** — data-parallel training across multiple GPUs on one node. Each worker holds a copy of the model and trains on its own partition of the data; parameters are periodically averaged or gradient-sharing is used to keep the copies in sync.
- **ParallelInference** — batched, multi-threaded inference using multiple model replicas (one per GPU). Useful when a production service receives many concurrent requests and a single-threaded `model.output()` call cannot keep up.

Both classes are in the `deeplearning4j-parallelwrapper` module:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-parallel-wrapper</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

---

## ParallelWrapper

### Overview

`ParallelWrapper` wraps a `MultiLayerNetwork` or `ComputationGraph` and replaces the single-threaded `model.fit(iterator)` call with a multi-worker training loop. Internally, it:

1. Creates one model replica per worker (one per GPU by default).
2. Distributes each minibatch across the replicas.
3. Each replica computes its gradient independently.
4. Gradients or parameters are aggregated according to the configured `TrainingMode`.
5. The master model is updated and the cycle repeats.

### Training Modes

`ParallelWrapper.TrainingMode` determines how replicas are kept in sync:

| Mode | Description |
|---|---|
| `AVERAGING` | Model parameters are averaged across all workers every N iterations. Simple and correct; adds some communication overhead. |
| `SHARED_GRADIENTS` | Workers share encoded gradient updates each iteration via DL4J's `EncodedGradientsAccumulator` (the same Strom-style algorithm as the Spark implementation). Lower overhead than full averaging. |
| `CUSTOM` | Bring your own `GradientsAccumulator`. Advanced use only. |

For most multi-GPU single-node training, `SHARED_GRADIENTS` is the recommended mode as it is faster and produces results comparable to `AVERAGING`.

### Quick Start Example

```java
import org.deeplearning4j.parallelism.ParallelWrapper;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;

// Build and initialize your model as normal
MultiLayerNetwork model = buildModel();
model.init();

// Wrap it in ParallelWrapper
ParallelWrapper wrapper = new ParallelWrapper.Builder<>(model)
        .workers(4)                                        // 1 worker per GPU
        .prefetchBuffer(24)                                // async data prefetching
        .trainingMode(ParallelWrapper.TrainingMode.SHARED_GRADIENTS)
        .averagingFrequency(3)                             // used only for AVERAGING mode
        .reportScoreAfterAveraging(true)
        .build();

DataSetIterator trainIter = getTrainingIterator();

// Train — same API as model.fit()
wrapper.fit(trainIter);

// After training, `model` holds the trained parameters
wrapper.close();
```

After training, call `wrapper.close()` to release worker threads and GPU memory. The original `model` object you passed in will contain the final trained parameters.

### Using with ComputationGraph

The API is identical for `ComputationGraph`:

```java
ComputationGraph cgModel = buildComputationGraph();
cgModel.init();

ParallelWrapper wrapper = new ParallelWrapper.Builder<>(cgModel)
        .workers(Nd4j.getAffinityManager().getNumberOfDevices())
        .trainingMode(ParallelWrapper.TrainingMode.SHARED_GRADIENTS)
        .prefetchBuffer(24)
        .build();

wrapper.fit(multiDataSetIter);
wrapper.close();
```

### Builder API

```java
public ParallelWrapper.Builder<T extends Model>(T model)
```
Pass your initialized `MultiLayerNetwork` or `ComputationGraph`.

---

**`.workers(int num)`**

Number of worker threads. Default: the number of available devices (GPUs) as detected by `Nd4j.getAffinityManager().getNumberOfDevices()`. Minimum value: 2.

On a 4-GPU machine, use `.workers(4)`. On a CPU-only machine, this is the number of CPU threads; setting it to the number of physical CPU sockets is usually best.

---

**`.trainingMode(TrainingMode mode)`**

`AVERAGING`, `SHARED_GRADIENTS`, or `CUSTOM`. Default: `AVERAGING`.

---

**`.averagingFrequency(int freq)`**

Only used when `trainingMode` is `AVERAGING`. Controls how often (in iterations) the worker parameters are averaged and synchronized. Lower values give better model quality but more communication overhead. Default: 1.

---

**`.prefetchBuffer(int size)`**

Number of minibatches to asynchronously prefetch for each worker. Default: 16. Increasing this can help when the data pipeline (ETL) is slower than the GPU compute.

---

**`.reportScoreAfterAveraging(boolean report)`**

When true, the current loss is printed after each averaging step. Useful for monitoring training without a full `ScoreIterationListener`. Default: false.

---

**`.averageUpdaters(boolean averageUpdaters)`**

Whether to also average the updater state (momentum buffers, adaptive learning rate accumulators, etc.) across workers during averaging. Default: true. Setting this to false can improve performance slightly but may cause instability with adaptive optimizers like Adam.

---

**`.workspaceMode(WorkspaceMode mode)`**

Override the workspace mode for all worker models. Default: `WorkspaceMode.ENABLED`. Usually does not need to be changed.

---

**`.build()`**

Returns a `ParallelWrapper` instance ready for use.

### SHARED_GRADIENTS Mode in Detail

When using `SHARED_GRADIENTS`, `ParallelWrapper` internally creates an `EncodedGradientsAccumulator`. The accumulator uses the same Strom-style threshold encoding as DL4J's Spark distributed training:

- Each worker encodes its gradient update as a sparse binary vector (only updates above a threshold `τ` are communicated).
- The accumulator broadcasts these sparse updates to all workers every iteration.
- Below-threshold updates are stored in a per-worker residual vector and carried forward.

The default `AdaptiveThresholdAlgorithm` automatically adjusts `τ` to maintain an appropriate sparsity level.

You can configure the threshold algorithm explicitly:

```java
import org.deeplearning4j.optimize.solvers.accumulation.encoding.threshold.AdaptiveThresholdAlgorithm;
import org.deeplearning4j.optimize.solvers.accumulation.encoding.residual.ResidualClippingPostProcessor;

ParallelWrapper wrapper = new ParallelWrapper.Builder<>(model)
        .workers(4)
        .trainingMode(ParallelWrapper.TrainingMode.SHARED_GRADIENTS)
        .thresholdAlgorithm(new AdaptiveThresholdAlgorithm(1e-3))  // initial threshold
        .residualPostProcessor(new ResidualClippingPostProcessor(5.0, 5)) // clip to 5x threshold every 5 steps
        .build();
```

### Performance Tips

**1. Set workers to number of GPUs**

```java
int numGPUs = Nd4j.getAffinityManager().getNumberOfDevices();
wrapper = new ParallelWrapper.Builder<>(model).workers(numGPUs).build();
```

**2. Use SHARED_GRADIENTS for best throughput**

`AVERAGING` sends the entire parameter vector each time it synchronizes. `SHARED_GRADIENTS` typically sends only a tiny fraction of it (1–5%), greatly reducing PCIe/NVLink bandwidth usage.

**3. Tune prefetchBuffer**

If the GPU is idle waiting for data (visible as low GPU utilization in `nvidia-smi`), increase `prefetchBuffer`. A value of 2–4× the number of workers is a reasonable starting point.

**4. Match batch size to GPU count**

If you trained single-GPU with a batch size of 64, use 64 per GPU when going to 4 GPUs (total effective batch size of 256). Adjust the learning rate accordingly (often by the same scale factor, or use a learning rate warmup).

**5. Use try-with-resources**

`ParallelWrapper` implements `AutoCloseable`, so it can be used with try-with-resources to ensure worker threads and GPU memory are always released:

```java
try (ParallelWrapper wrapper = new ParallelWrapper.Builder<>(model)
        .workers(numGPUs)
        .build()) {
    wrapper.fit(trainIter);
}
// workers are shut down here
```

---

## ParallelInference

### Overview

`ParallelInference` is the inference counterpart to `ParallelWrapper`. Rather than training, it handles prediction requests from multiple threads concurrently by maintaining a pool of model replicas (one per GPU) and routing requests to the least loaded replica.

This is particularly useful in production serving scenarios where:
- Multiple request threads call `model.output()` simultaneously.
- A single-threaded model cannot saturate a GPU (small batch sizes per request).
- You want to batch individual requests together to improve GPU utilization.

### Quick Start Example

```java
import org.deeplearning4j.parallelism.ParallelInference;
import org.deeplearning4j.parallelism.inference.InferenceMode;

// Initialize inference server
ParallelInference inference = new ParallelInference.Builder(trainedModel)
        .inferenceMode(InferenceMode.BATCHED)
        .workers(4)         // 1 thread per GPU
        .batchLimit(32)     // maximum batch size for auto-batching
        .queueLimit(64)     // max number of queued requests
        .build();

// Use from multiple threads — thread-safe
INDArray input  = Nd4j.rand(1, 784);
INDArray output = inference.output(input);

// Shut down when done
inference.shutdown();
```

### Inference Modes

`InferenceMode` controls how individual requests are handled:

| Mode | Description |
|---|---|
| `BATCHED` | Incoming single-example requests are automatically grouped into mini-batches and submitted to the GPU together. Best GPU utilization; adds small latency (~microseconds to milliseconds). |
| `SEQUENTIAL` | Each request is processed individually by an available worker in FIFO order. Lower latency per request; lower GPU utilization under light load. |
| `INPLACE` | Uses the model's own `output()` method directly on the calling thread. Effectively disables parallelism — use only for single-threaded testing. |

`BATCHED` is recommended for production when throughput matters. `SEQUENTIAL` is appropriate when latency is the priority and requests arrive with low concurrency.

### Builder API

```java
public ParallelInference.Builder(Model model)
```
Accepts `MultiLayerNetwork` or `ComputationGraph`.

---

**`.inferenceMode(InferenceMode mode)`**

`BATCHED`, `SEQUENTIAL`, or `INPLACE`. Default: `BATCHED`.

---

**`.workers(int workers)`**

Number of parallel inference threads. Default: number of available GPUs (`Nd4j.getAffinityManager().getNumberOfDevices()`). Each worker maintains its own model replica.

---

**`.batchLimit(int limit)`**

Only applies to `BATCHED` mode. Maximum number of individual requests that will be batched together for a single forward pass. Default: 32. Setting this too high increases latency; setting it too low reduces GPU utilization.

---

**`.queueLimit(int limit)`**

Maximum number of requests that can be queued before the calling thread blocks. Default: 64. Increasing this allows more burst capacity at the cost of memory.

---

**`.loadBalanceMode(LoadBalanceMode mode)`**

Controls how requests are distributed across workers. Options: `FIFO` (default) — requests go to the next available worker in round-robin order.

---

**`.build()`**

Returns a `ParallelInference` instance. Workers are started immediately on `build()`.

### Inference Methods

All `output()` methods on `ParallelInference` are **thread-safe**.

```java
// Single input
INDArray output = inference.output(INDArray input);
INDArray output = inference.output(INDArray input, INDArray inputMask);

// Multi-input (ComputationGraph)
INDArray[] outputs = inference.output(INDArray[] inputs);
INDArray[] outputs = inference.output(INDArray[] inputs, INDArray[] inputMasks);

// From a DataSet
INDArray output = inference.output(DataSet dataSet);

// From primitive arrays
INDArray output = inference.output(float[] input);
INDArray output = inference.output(double[] input);
```

### Update the Model at Runtime

You can replace the model while inference is running without restarting the server — useful for rolling updates in production:

```java
// Train a new model
MultiLayerNetwork newModel = trainNewModel();

// Swap the model — all in-flight requests complete with the old model first
inference.updateModel(newModel);
```

### Shutdown

Always call `shutdown()` when the inference server is no longer needed to release GPU memory and threads:

```java
inference.shutdown();
```

---

## Combining ParallelWrapper and ParallelInference

A typical workflow:

1. Train using `ParallelWrapper` on your training machine (4 GPUs, single node).
2. Save the model.
3. Deploy using `ParallelInference` on your serving machine.

```java
// --- Training ---
MultiLayerNetwork model = buildAndInitModel();
try (ParallelWrapper wrapper = new ParallelWrapper.Builder<>(model)
        .workers(4)
        .trainingMode(ParallelWrapper.TrainingMode.SHARED_GRADIENTS)
        .build()) {
    for (int epoch = 0; epoch < numEpochs; epoch++) {
        wrapper.fit(trainIter);
        trainIter.reset();
    }
}
model.save(new File("trained-model.zip"));

// --- Serving ---
MultiLayerNetwork loaded = MultiLayerNetwork.load(new File("trained-model.zip"), false);

ParallelInference server = new ParallelInference.Builder(loaded)
        .inferenceMode(InferenceMode.BATCHED)
        .workers(Nd4j.getAffinityManager().getNumberOfDevices())
        .batchLimit(32)
        .build();

// In request handler threads (concurrently safe):
INDArray prediction = server.output(requestInput);

// At shutdown:
server.shutdown();
```

---

## Comparing Single-GPU, ParallelWrapper, and Spark

| Approach | Best For | Complexity |
|---|---|---|
| `model.fit()` | Single GPU / CPU; prototyping | Low |
| `ParallelWrapper` | 2–8 GPUs on one machine | Low–Medium |
| DL4J Spark + `SharedTrainingMaster` | Clusters of 2+ machines | High |

For teams with a single 4–8 GPU machine, `ParallelWrapper` provides near-linear training speedup with minimal setup. Spark adds significant operational overhead and is only worthwhile when a cluster is already available or the dataset is too large for one machine.

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-parallel-wrapper</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

---

## See Also

- [Distributed Training Overview](overview.md) — Introduction to DL4J distributed training approaches
- [Parameter Server](parameter-server.md) — Gradient sharing details for multi-machine Spark training
- [Spark API Reference](spark-api-reference.md) — `SparkDl4jMultiLayer`, `SparkComputationGraph`, `SharedTrainingMaster`
- [Benchmarking](../benchmarking.md) — How to measure training and inference throughput accurately

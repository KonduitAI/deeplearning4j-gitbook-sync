---
title: "Technical Reference"
description: "Technical details of distributed training — Strom ASGD algorithm, mesh networking, and fault tolerance"
---

# Distributed Training: Technical Reference

This page covers the technical internals of DL4J's distributed training implementations. It is intended for readers who want to understand the algorithmic details, implementation choices, and performance characteristics. For setup and configuration, see the [Spark How-To guide](spark-howto.md) and [Parameter Server guide](parameter-server.md).

This page assumes familiarity with the basics of data-parallel distributed training, stochastic gradient descent, and the distinction between synchronous and asynchronous SGD. For background, see [An Introduction to Distributed Training of Neural Networks](http://engineering.skymind.io/distributed-deep-learning-part-1-an-introduction-to-distributed-training-of-neural-networks/).

---

## Asynchronous SGD with Gradient Sharing

### The Strom 2015 Algorithm

DL4J's primary distributed training implementation is based on the paper [Scalable Distributed DNN Training Using Commodity GPU Cloud Computing](http://nikkostrom.com/publications/interspeech2015/strom_interspeech2015.pdf) by Nikko Strom. The core insight is that full gradient vectors do not need to be communicated between workers — only the large elements matter for convergence.

**How it works:**

At each iteration, each worker computes a gradient update vector (one value per parameter). Instead of sending this dense vector over the network, the worker:

1. Adds the current gradient to its residual vector (accumulated un-communicated updates).
2. Selects elements of the residual whose absolute value exceeds a threshold `τ`.
3. Encodes selected elements as `+τ` or `-τ` (one bit each, plus an integer index). This is a sparse binary vector.
4. Clears the encoded elements from the residual (they have been communicated).
5. Broadcasts the sparse binary vector to other workers, who apply it to their local parameters.

Elements not selected (below the threshold) remain in the residual and accumulate for future iterations. They are never lost — merely delayed.

**Update vectors are:**
- **Sparse**: only some gradients are communicated (the remainder are zero for this step).
- **1-bit quantized**: each element is either `+τ` or `-τ`.
- **Index-encoded**: integer indexes identify the positions in the parameter array.

The paper reports compression ratios of 1000x or more over naive dense parameter communication, with minimal impact on final model accuracy.

**Stale gradients:**

Asynchronous SGD introduces the concern of stale gradients — workers applying updates based on outdated parameter values. In Strom's approach, this is handled implicitly: because only large updates are communicated, the parameters are (approximately) synchronized across workers at most iterations. In practice, the paper reports that stale gradients are not a significant problem for the workloads studied.

**Trade-offs introduced by the algorithm:**
1. Two hyperparameters: the threshold `τ` and whether to use entropy coding for further compression. (DL4J handles the encoding scheme selection automatically.)
2. Small extra computation per iteration: threshold encoding and residual maintenance.
3. Slight memory overhead per worker: the residual vector (same size as the parameter vector).
4. Convergence can be slightly slower in early training. The paper suggests using fewer nodes at the start, then scaling up.

### DL4J's Modifications

DL4J extends the Strom algorithm in several ways:

**1. Adaptive threshold**

Strom used a fixed threshold. DL4J uses an adaptive threshold that adjusts based on observed update sparsity. The `ThresholdAlgorithm` interface defines the adaptation strategy:

- `AdaptiveThresholdAlgorithm` (default): After each iteration, measures the sparsity ratio (fraction of parameters encoded). If sparsity is too high (almost nothing communicated), the threshold is decreased. If sparsity is too low (almost everything communicated), the threshold is increased. Target range: 0.0001–0.01.
- `FixedThresholdAlgorithm`: Uses a fixed threshold throughout training.
- `TargetSparsityThresholdAlgorithm`: Adapts to hit a user-specified target sparsity.

Adaptive thresholding removes the need to manually tune `τ` for each model.

**2. Dual encoding schemes**

DL4J dynamically selects between two encoding schemes per iteration:

*Threshold encoding* (sparse):
- Sends a list of integers, one per parameter that exceeded the threshold.
- Positive integer: parameter takes value `+τ`. Negative integer: parameter takes value `-τ`.
- Compression ratio: proportional to sparsity. At 1% sparsity: ~100x compression.

*Bitmap encoding* (dense):
- Uses two bits per parameter to encode four states: no change, `+τ`, `-τ`, or `τ/2`.
- The `τ/2` state is used for periodic "shake-up" messages to flush long-accumulated residuals.
- Fixed 16x compression ratio (2 bits vs. 32 bits per parameter).

The encoding with the smaller output size is sent. Threshold encoding wins when updates are very sparse; bitmap encoding wins when updates are denser (e.g., early in training, or with a low threshold).

Both encoding/decoding paths are implemented in native C++ with GPU parallelization for performance.

**3. Residual clipping**

Without intervention, if the learning rate produces updates much larger than `τ`, the residual can grow to hundreds or thousands of times `τ`. When this happens, these large residuals take many iterations to communicate (one `τ`-sized chunk at a time), creating what DL4J calls "residual explosion" — effective gradient staleness becomes very large.

`ResidualClippingPostProcessor` addresses this by periodically clipping the residual to a maximum of `clipMultiple × τ` (default: 5x). The clip runs every `clipFrequency` steps (default: 5). This prevents residuals from growing unbounded while still allowing short-term accumulation for small updates.

**4. Shake-up messages**

Periodically, DL4J sends messages encoded with a smaller threshold than the current `τ`. This flushes weights that have accumulated in the residual but are unlikely to exceed the normal threshold any time soon. Without shake-up messages, some parameters could be essentially frozen in their communicated state while significant but sub-threshold changes accumulate.

**5. Local parallelism**

Within each cluster node, `ParallelWrapper` is used to coordinate multiple GPUs. Workers on the same machine share updates through shared memory rather than UDP, which is significantly faster for intra-node communication.

---

## Network Topology: Plain Mode and Mesh Mode

### Plain Mode

In plain mode, the Spark master is the central communication hub:

```
                   Master
                  /  |  \
               W1   W2   W3
```

Each worker sends its encoded updates to the master. The master relays them to all other workers. This ensures the master always has the current model state, simplifying fault tolerance.

Limitation: the master is a bottleneck. At high worker counts, the master must receive `N` messages and relay `N*(N-1)` messages per iteration. For most clusters with fewer than ~32 nodes this is not a problem. Beyond that, mesh mode is preferred.

### Mesh Mode

In mesh mode, workers form a tree structure rooted at the master:

```
                   Master
                 /         \
              Node A       Node B
             /     \       /    \
           W1      W2    W3     W4
```

Each node relays encoded updates to all nodes directly connected to it (children and parent). Each node aggregates updates from all connected nodes. The master's direct communication load is proportional to its number of children in the tree (not to the total cluster size).

Configuration:
- Max children per node: 8 (default)
- Max tree depth: 5 levels
- This supports clusters of up to 8^5 = 32,768 nodes in the tree (though practical limits are much smaller)

Mesh mode requires slightly more complex fault recovery (see [Fault Tolerance](#fault-tolerance) below) but scales to much larger clusters without the master becoming a bottleneck.

Unicast and multicast are both supported as of 1.0.0-beta3. Cloud environments (AWS, Azure) use unicast only.

---

## Parameter Averaging Implementation

Parameter averaging is DL4J's original distributed training implementation, now superseded by gradient sharing. It is documented here for completeness and for users migrating from older versions.

### Algorithm

1. The Spark master holds the initial network configuration and parameters.
2. Data is split into subsets according to `TrainingMaster` configuration.
3. For each data subset:
   a. The master broadcasts current parameters (and optionally optimizer state) to all workers.
   b. Each worker trains on its portion for `averagingFrequency` minibatches.
   c. Workers send updated parameters back to the master.
   d. The master averages all received parameters and stores the result.
4. Repeat for the next subset / epoch.

This is synchronous SGD: all workers must complete before parameters are averaged and the next round begins.

### Implementation Details

DL4J uses Spark's `treeAggregate` for the parameter collection and averaging step. Tree aggregation reduces the load on the master compared to a flat reduce, making it practical for moderate cluster sizes.

The `aggregationDepth` parameter controls the tree depth (default: 2). For large models with many partitions, increasing this reduces driver load during the aggregation phase.

The `saveUpdater` option controls whether optimizer state (momentum, AdaGrad history, etc.) is included in the averaging. Including it (`true`, the default) doubles the network traffic but preserves adaptive learning rate information across workers, which is important for optimizers like Adam, AdaGrad, and RMSProp.

### Why Gradient Sharing is Preferred

Parameter averaging has several disadvantages vs. gradient sharing:
- **Synchronous barrier**: workers must wait for all other workers to finish before the next round begins. Slow nodes ("stragglers") block the entire cluster.
- **More communication**: sending full parameter vectors is more expensive than sparse encoded gradient updates.
- **Single parameter server**: the master is a single point of failure and communication bottleneck.
- **Convergence**: averaging parameters rather than sharing gradients can require more frequent synchronization to maintain convergence quality.

---

## <a name="fault-tolerance"></a>Fault Tolerance

### Overview

DL4J's gradient sharing implementation is fully fault-tolerant as of 1.0.0-beta3. Parameter averaging has always been fault-tolerant via Spark's built-in RDD re-computation.

### Gradient Sharing Fault Tolerance

The challenge: Spark tracks RDD lineage back to the initial parameters. When Spark replaces a failed node, it recomputes the RDD from the start, giving the new node the original (not current) parameters. If allowed to continue from that initial state, the restored node would be far out of sync with the other nodes, causing training to diverge.

**DL4J's solution:**

DL4J maintains an out-of-Spark heartbeat mechanism. When the parameter server detects a node failure and recovery:

1. The restored node reconnects to the master.
2. It immediately begins receiving new encoded updates (not yet applied).
3. It requests the current parameter vector, optimizer state, and iteration/epoch number from the master.
4. The master provides the current parameters directly (in plain mode) or via proxy from another worker (for optimizer state — the master does not itself train, so does not hold optimizer state).
5. The restored node applies only the updates that arrived after the parameter snapshot, identified by unique update IDs.
6. Training continues in sync.

Receiving updates before requesting the parameters (step 2 before step 3) ensures no updates are missed: the node buffers incoming updates and applies only those with IDs newer than the received snapshot.

### Mesh Mode Recovery

When a node fails in mesh mode, its children in the tree are remapped to preserve the tree's integrity:

If node `F` fails and has children `C1`, `C2`, `C3`:
- One child (say `C1`) is remapped directly to the master.
- The remaining children (`C2`, `C3`) are remapped to `C1`.

The master is chosen as the primary remapping target because it is assumed to be the most reliable node in the cluster. This maximizes the reliability of the remaining tree structure.

### Known Limitations

**1. Duplicate example processing**: A failed node may have processed some of its assigned partition and sent updates before failing. When the partition is reassigned to a new node, it will be processed from the beginning. A small number of examples (up to one full partition) may therefore be processed twice. For multi-epoch training this effect is negligible.

**2. Master failure**: If the Spark driver/master fails, training stops. This is a fundamental Spark limitation. Mitigation: save checkpoints frequently (e.g., after every epoch) and restart from the latest checkpoint if the master fails.

### Parameter Averaging Fault Tolerance

Parameter averaging uses Spark's native RDD fault tolerance. Since all state (parameters) is stored on the master at the end of each averaging step, and Spark can recompute any failed RDD from lineage, no special handling is required. However, this comes at the cost of the synchronous barrier: if a node fails mid-averaging, the entire averaging round must restart.

---

## Aeron and the Communication Layer

### Why Not Spark RPC?

Spark's built-in communication layer (RPC) adds significant overhead for small, frequent messages like per-iteration gradient updates. Spark was designed for batch operations with large payloads, not for the high-frequency low-latency messaging required by ASGD.

DL4J uses [Aeron](https://github.com/real-logic/aeron/wiki) as its communication layer, operating entirely outside of Spark's data flow for gradient messages:
- **UDP unicast** for standard Ethernet clusters
- **Infiniband/RDMA** support for high-performance interconnects
- **Shared memory** for intra-machine worker communication
- Designed for the lowest possible latency and highest throughput in messaging systems

Aeron's architecture allows DL4J to control memory allocation precisely, avoiding garbage collection pressure from message passing.

### Communication Cost Model

Additional overhead per iteration from gradient sharing:
```
overhead = encoding_time + serialization_time + update_application_time
```

This overhead is constant regardless of network size (for a fixed update message size). Since encoding is done in optimized native code, encoding time is typically well under 1 ms for most model sizes.

The key ratio is `overhead / iteration_time`. As long as `iteration_time >> overhead`, scaling efficiency is good. This is why the 100 ms threshold is a useful rule of thumb: at 100 ms/iteration, 1–2 ms of communication overhead is a 1–2% tax.

### UDP Unicast vs. Broadcast

DL4J uses UDP unicast only (no multicast or broadcast), which ensures compatibility with cloud providers that do not support multicast on their virtual networks. In unicast mode, the master retransmits messages it receives to each other node individually. Because the master typically has low utilization (it does not train), this retransmission overhead is acceptable, and the asynchronous nature of the updates means communication can overlap with computation.

Message retransmission for reliability is handled by DL4J's own protocol on top of Aeron, not by TCP. UDP is used because its lower overhead and latency characteristics are more appropriate for the small, frequent messages of gradient sharing.

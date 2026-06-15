---
title: "Parameter Server"
description: "Gradient sharing via the Aeron-based parameter server for distributed training"
---

# Parameter Server and Gradient Sharing

DL4J's primary distributed training implementation uses an Aeron-based parameter server for high-performance gradient sharing. This page covers the architecture, network requirements, configuration, and performance tuning for this approach.

For an introduction, see the [Distributed Training Overview](overview.md). For API details, see [SharedTrainingMaster in the API Reference](spark-api-reference.md#sharedtrainingmaster).

---

## Architecture

### The Gradient Sharing Algorithm

Each worker computes a gradient update on its local minibatch. Rather than sending the full gradient vector to a centralized server, only updates that exceed a threshold are communicated:

1. Each worker computes its parameter update (gradient times learning rate).
2. Updates are accumulated in an intermediate buffer.
3. Updates whose absolute value exceeds the threshold `τ` are encoded as a sparse binary vector and broadcast to other nodes.
4. Updates below `τ` are stored in a **residual vector** — they are not discarded. They accumulate and will be communicated in future iterations once the cumulative residual exceeds `τ`.

This produces a sparse, quantized communication message. The threshold encoding represents each communicated element as either `+τ` or `-τ`, requiring only one bit per element (plus an integer index). In practice this reduces communication volume by orders of magnitude vs. sending the raw update.

### DL4J Extensions to the Strom Algorithm

The original [Strom 2015 paper](http://nikkostrom.com/publications/interspeech2015/strom_interspeech2015.pdf) assumed a fixed threshold and point-to-point communication. DL4J's implementation differs in several important ways:

**Adaptive threshold**: The threshold `τ` is adjusted automatically after each iteration to maintain a target sparsity ratio (fraction of parameters communicated). If updates are too sparse (almost nothing communicated), the threshold is reduced. If too dense (nearly everything communicated), the threshold is increased. This is implemented via the `ThresholdAlgorithm` interface; the default is `AdaptiveThresholdAlgorithm`.

**Dual encoding schemes**: DL4J dynamically selects between two encodings:
- *Threshold encoding*: a list of integer indexes, one per communicated parameter. Provides very high compression for sparse updates.
- *Bitmap encoding*: two bits per parameter, encoding states: no change, `+τ`, `-τ`, and `τ/2` (a "shake-up" value for delayed weights). Results in exactly 16x compression vs. the raw update vector. Used when updates are dense enough that the bitmap is more compact than the index list.

**Residual clipping**: If updates are much larger than the threshold, the residual can grow to many multiples of `τ`, taking many iterations to communicate (residual explosion). `ResidualClippingPostProcessor` clips the residual to a maximum of 5x the current threshold every 5 steps by default.

**Shake-up messages**: Periodically, special messages are sent with a much smaller threshold, to flush delayed weights that would otherwise take too long to be communicated through the normal threshold process.

### Aeron for Out-of-Spark Communication

Spark's RPC layer is too slow for the per-iteration gradient messages required by ASGD. DL4J uses [Aeron](https://github.com/real-logic/aeron/wiki) for out-of-band communication:

- Aeron is a high-performance messaging library designed for minimum latency and maximum throughput.
- It runs over **UDP unicast** (also supporting InfiniBand and shared memory, though UDP is the standard Spark cluster choice).
- Cloud environments (AWS, Azure) that do not support multicast are supported — DL4J uses unicast only.
- All gradient messages bypass Spark's shuffle and RPC layers entirely.

The trade-off: you must open a UDP port on all nodes and configure network addressing explicitly via `VoidConfiguration`.

---

## Network and Cluster Setup

### Requirements

- Spark 2.x cluster (Spark 1.x is also supported but less tested).
- Java 8 or later.
- At least one UDP port open for **inbound and outbound** traffic on all nodes (driver and workers).

### Cloud Environments (AWS, Azure, GCP)

You must open the UDP port in your security group / firewall rules for all nodes in the cluster. Inbound and outbound must both be permitted.

Set the network mask to the CIDR range of your cluster's internal network:
```java
VoidConfiguration conf = VoidConfiguration.builder()
    .unicastPort(40123)
    .networkMask("10.0.0.0/16")       // adjust to match your VPC CIDR
    .controllerAddress("10.0.2.4")    // driver's internal IP
    .build();
```

### YARN Environments

When running Spark on YARN, Spark may assign IP addresses that are not reachable for direct node-to-node communication. Specify the network mask of the interface to use:

```java
VoidConfiguration conf = VoidConfiguration.builder()
    .unicastPort(40123)
    .networkMask("192.168.1.0/24")    // subnet where all nodes can reach each other
    .build();
```

If automatic interface selection still fails, set the `DL4J_VOID_IP` environment variable on each node to force a specific IP address for Aeron communication:
```bash
export DL4J_VOID_IP=192.168.1.45   # set to the node's correct IP
```

### Network Mask Reference

A network mask (in CIDR notation) identifies which addresses share a subnet. Examples:
- Cluster with IPs `192.168.1.23`, `192.168.1.78`, `192.168.2.133` — use `192.168.0.0/16`
- Cluster with IPs `10.1.2.x` — use `10.1.2.0/24` or `10.0.0.0/8`

The mask selects which network interface Aeron binds to for inter-node communication.

---

## Configuration

### Complete SharedTrainingMaster Setup

```java
VoidConfiguration voidConf = VoidConfiguration.builder()
    .unicastPort(40123)
    .networkMask("10.0.0.0/16")
    .controllerAddress("10.0.2.4")
    .build();

TrainingMaster tm = new SharedTrainingMaster.Builder(voidConf)
    .batchSizePerWorker(32)
    .workersPerNode(4)                              // number of GPUs per node
    .thresholdAlgorithm(new AdaptiveThresholdAlgorithm(1e-3))
    .residualPostProcessor(new ResidualClippingPostProcessor(5, 5))
    .meshBuildMode(MeshBuildMode.MESH)              // use PLAIN for < 32 nodes
    .rddTrainingApproach(RDDTrainingApproach.Export)
    .workerTogglePeriodicGC(true)
    .workerPeriodicGCFrequency(5000)
    .build();

SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, modelConf, tm);
```

### Plain Mode vs. Mesh Mode

**Plain mode** (`MeshBuildMode.PLAIN`):

Each worker sends encoded updates to the master; the master relays them to all other workers. The master always holds the current model state, making it the authoritative checkpoint for fault tolerance. The master is a potential bottleneck with large clusters.

Use plain mode when:
- Cluster size is less than ~32 nodes.
- You want a simpler topology that is easier to debug.

**Mesh mode** (`MeshBuildMode.MESH`):

Workers form a non-binary tree rooted at the master. Each node relays updates to its directly connected neighbors. The default configuration allows up to 8 children per node and a maximum tree depth of 5 levels (supporting thousands of nodes in theory).

Benefits:
- The master's communication load is reduced (it only communicates directly with its immediate children in the tree).
- Scales to much larger clusters.

Use mesh mode when:
- Cluster size exceeds ~32 nodes.
- The master would otherwise become a communication bottleneck.

---

## Fault Tolerance

The gradient sharing implementation is fully fault-tolerant as of 1.0.0-beta3.

### What Happens When a Worker Node Fails

DL4J's parameter server maintains an internal heartbeat mechanism outside of Spark to detect node failures and recoveries. When a worker fails and is restarted by Spark, it will initially be out of sync with other nodes (since Spark's RDD lineage tracks back to the initial parameters). To prevent training divergence:

1. The restored node reconnects to the master.
2. It begins receiving new gradient updates from other nodes.
3. It sends a request to the master for the current parameter state, optimizer state, and iteration/epoch number.
4. The master fulfills these requests (either directly or by proxying to another worker for the optimizer state).
5. The restored node applies only the updates it missed (tracked by unique update IDs).
6. Training continues in sync.

**Important**: updates are tagged with unique IDs. No update is applied twice, even if the timing of the parameter sync overlaps with new incoming updates.

### Mesh Mode: Node Failure Remapping

In mesh mode, when a node fails, its children in the tree are remapped. For example, if node 2 fails and its children are nodes 5, 6, and 7:
- Node 5 is remapped directly to the master.
- Nodes 6 and 7 are remapped to node 5.

The master is chosen as the remapping target because it is the most reliable node in the cluster.

### Limitations

1. **Duplicate minibatches**: A failed node may have processed some minibatches (sending updates) before failing. Those examples may be processed again on the replacement node, since the RDD partition is recomputed from the beginning. In practice this affects only a small number of examples and is not a problem for multi-epoch training.

2. **Master is a single point of failure**: If the Spark driver/master fails, training cannot continue. This is a Spark limitation. Mitigate by saving model checkpoints frequently — if the master fails, restart training from the latest checkpoint.

---

## Performance Tuning

### Iteration Time is the Key Metric

The parameter server's communication overhead is fixed per iteration. The longer each iteration takes (more computation), the smaller the relative overhead from gradient sharing. A rough guide:
- Iteration time > 150 ms: gradient sharing provides good speedup.
- Iteration time 10–150 ms: sub-linear scaling, but still typically beneficial.
- Iteration time < 10 ms: communication overhead may dominate; Spark may not help.

Use `PerformanceListener` to measure per-iteration time before moving to distributed training.

### Executors and Core Count

Set `--num-executors` to the number of machines. Set `--executor-cores` based on your hardware. For GPU nodes, 1–2 Spark executor cores per GPU is typical (Spark cores control data pipeline threads, not GPU threads).

`workersPerNode` controls DL4J training threads per node:
- GPU nodes: set equal to the number of GPUs per machine.
- CPU nodes: `1` is usually best; for large core counts (32+), experiment with higher values paired with `OMP_NUM_THREADS`.

Example for a node with 4 GPUs:
```java
.workersPerNode(4)
```

### Encoding Threshold

The threshold controls the sparsity of gradient communication. Higher threshold = fewer updates communicated = less network traffic but potentially slower convergence. Lower threshold = more updates = more traffic but better model quality.

The `AdaptiveThresholdAlgorithm` default targets a sparsity ratio of 0.0001–0.01, meaning 0.01%–1% of parameters are communicated per step. This works well for most models.

Enable encoding debug mode during development to monitor whether the threshold is appropriate:
```java
.encodingDebugMode(true)
```

Watch the logged sparsity ratio. If it is consistently very close to 0 or to 1, the threshold needs adjustment.

### Network Bandwidth

Minimum: 1 GbE. Recommended: 10 GbE or faster.

With a well-configured threshold, gradient messages are very small (sparse encoding). The bottleneck for most well-sized models is computation, not network. However, very large models or very short iteration times can make network a bottleneck.

Infiniband and RDMA are supported via Aeron's transport layer for clusters with specialized interconnects.

### Multi-GPU Nodes

On multi-GPU boxes, DL4J uses `ParallelWrapper` internally within each node to coordinate the GPUs. PCIe or NVLink peer-to-peer GPU connectivity improves performance but is not required. P2P transfers are faster, but training will still work correctly (just somewhat slower) without P2P.

---

## Dependency

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>dl4j-spark-parameterserver_${scala.binary.version}</artifactId>
    <version>${dl4j.spark.version}</version>
</dependency>
```

Replace `${scala.binary.version}` with `2.11` for Spark 2.x clusters.

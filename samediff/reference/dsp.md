---
title: DSP Execution Engine
description: Complete guide to the Dynamic Shape Plan execution engine — compiled graph runtime with CUDA graph capture/replay, Triton/NVRTC/PTX JIT, 26-pass optimizer, and multi-backend dispatch
---

# DSP Execution Engine

The Dynamic Shape Plan (DSP) engine is the compiled graph execution runtime introduced in DL4J 1.0.0-rewrite. It replaces the previous interpreter-style graph execution with a compile-once, replay-many architecture that delivers substantially lower inference latency — especially on NVIDIA GPUs, where it eliminates kernel launch overhead entirely via CUDA graph capture and replay.

## Overview

Before DSP, executing a SameDiff graph meant traversing the graph node by node on every call, dispatching each operation through the op registry, and paying full kernel launch overhead for every operation. For large transformer models with hundreds of operations per forward pass, this launch tax dominates latency at batch size 1.

DSP changes this model. The first time a SameDiff graph is executed, DSP compiles it into a `DynamicShapePlan`: a flat, ordered sequence of "slots," one per SameDiff variable. After a warmup phase that learns the concrete shapes of every intermediate tensor, DSP freezes the shapes, captures the entire sequence of GPU operations into a CUDA graph, and replays that single pre-captured graph on every subsequent call. On the replay path, no individual kernel is launched from the Java side — the CUDA driver replays the full captured op sequence in a single API call.

The same compilation pipeline runs on non-CUDA hardware via alternative graph backends (Metal, Vulkan, oneDNN, NNAPI, MLX, and others) that provide analogous captured-graph replay semantics on their respective platforms.

### Why Compilation Matters

A compiled plan has several advantages over interpreted execution:

- **Reduced launch overhead.** Each kernel launch from Java costs 5–50 µs. A 200-op transformer graph accumulates 1–10 ms of pure overhead per forward pass. CUDA graph replay collapses all 200 launches to a single call costing under 10 µs.
- **Constant memory pointers.** Once shapes are frozen, buffer addresses do not change between calls. The CUDA graph can be captured with baked-in pointers and replayed without pointer fixup.
- **Graph-level optimization.** The 26-pass optimizer (described below) operates on the full graph and performs cross-op fusions — attention fusion, horizontal QKV projection fusion, algebraic simplifications — that are impossible to apply op-by-op at runtime.
- **JIT specialization.** The Triton, NVRTC, and PTX JIT backends generate kernel code specialized to the exact shapes and data layouts of the compiled plan, enabling further micro-optimizations.

---

## Plan Lifecycle

A `DynamicShapePlan` progresses through four phases. The active phase governs which execution path is taken on each call.

```
WARMUP  →  SHAPES_FROZEN  →  CUDA_GRAPH_CAPTURED  →  REPLAYING
```

### Phase 1: WARMUP

During warmup, DSP executes the graph slot by slot — one operation at a time — using the standard ND4J op dispatch. The purpose of warmup is to observe the concrete output shape of every intermediate tensor for the given input shapes. Each slot records its output shape and stores it in the plan's shape table.

The `DynamicShapePlanExecutor` maintains one execution stream per thread (`tl_dspExecutionStream`) plus a secondary stream for "gap" ops (`tl_dspGapStream`) — operations that fall outside the captured region and must continue running via normal dispatch even after capture.

Warmup ends after the configured number of warmup iterations (default: 1). With dynamic batch sizes, warmup may be repeated for each unique input shape signature.

### Phase 2: SHAPES_FROZEN

After warmup, the plan calls `sdxFreezeShapes()`. At this point, all intermediate buffer allocations are finalized. Buffers are pre-allocated at their frozen sizes and their device pointers are recorded. The plan verifies that subsequent calls supply inputs whose shapes match the frozen shape signature. If an input arrives with a different shape, DSP falls back to slot-by-slot execution for that call.

Variable states also transition: every slot moves from `WARMUP` to `FROZEN` state. The `PlanIntrospection` API can be used to inspect the shape table at this point.

### Phase 3: CUDA_GRAPH_CAPTURED

With shapes frozen and pointers stable, DSP initiates CUDA graph capture using `cudaStreamBeginCapture()`. It then replays the slot sequence in capture mode — each op records its GPU work into the capture stream rather than executing immediately. At the end of capture, `cudaStreamEndCapture()` yields a `cudaGraph_t`, which is then instantiated into a `cudaGraphExec_t` for later replay.

Non-CUDA backends perform an equivalent step: HIP records a `hipGraph_t`, Metal records a `MTLCommandBuffer`, Vulkan records a `VkCommandBuffer`, and Level Zero records a `ze_command_list_t`.

Slots that cannot participate in capture (e.g., operations involving CPU-side conditional logic or ops not yet supported by the backend's capture mechanism) are marked as `GAP` nodes. Gap slots continue to execute via `tl_dspGapStream` during replay.

### Phase 4: REPLAYING

Once a valid captured graph exists, all subsequent calls enter replay mode. The executor calls `cudaGraphLaunch(graphExec, stream)` — a single API call that kicks off the entire forward pass — and waits for completion. The per-thread `argTableStable` fast path skips pointer-stability checks when all input pointers are confirmed stable, shaving a further few microseconds per call.

Slot states in replay: `FROZEN` slots that are inside the captured region transition to `REPLAYING`. Gap slots remain in their own state and continue to be dispatched individually.

### Slot States

| State | Meaning |
|---|---|
| `UNINITIALIZED` | Slot has been defined but warmup has not yet run |
| `WARMUP` | Actively executing during warmup phase |
| `FROZEN` | Shape recorded; buffer allocated; pointer stable |
| `CAPTURING` | Inside a CUDA/HIP/Vulkan graph capture |
| `REPLAYING` | Being replayed via captured graph |
| `EVICTED` | Plan was evicted from cache; slot must be re-warmed |

### Graph Node Phases

Each node in the compiled plan carries a phase tag that the executor uses to decide how to dispatch it:

| Node Phase | Dispatch Method |
|---|---|
| `SLOT_BY_SLOT` | Standard ND4J op dispatch (warmup and gap ops) |
| `CAPTURED` | Included in CUDA/backend graph capture; replayed atomically |
| `GAP` | Op excluded from capture; runs via gap stream each call |
| `TRITON` | Dispatched to Triton JIT kernel; participates in captured graph via pre-compiled PTX |

---

## Graph Optimizer

Before building the execution plan, DSP runs a 26-pass optimizer over the SameDiff graph. Passes are organized into three outer iterations, allowing earlier simplifications to expose new opportunities for later passes.

The optimizer is configurable:
- Skip individual passes: `-Dnd4j.optimizer.skip=PassName` (comma-separated for multiple)
- Log applied rewrites: `-Dnd4j.optimizer.logApplied=true`

### Pass List

**Pass 1 — Dead Code Elimination (`UnusedFunctionOptimizations`)**  
Removes ops whose outputs are not consumed by any requested output or by any op that transitively feeds a requested output. On large models with optional diagnostic branches, this can remove 10–20% of nodes before any other pass runs.

**Pass 2 — Constant Propagation (`ConstantFunctionOptimizations`)**  
Evaluates ops whose all inputs are constants at compile time. The computed result is folded into a new `CONSTANT` node and the original op is removed. Shape computations on static inputs are almost always eliminated here.

**Pass 3 — Broadcast Elimination and Commutative Canonicalization (`BroadcastEliminationOptimizations`)**  
Removes unnecessary explicit broadcast ops when the underlying kernel already handles broadcasting natively. Also canonicalizes commutative ops (e.g., swapping operands so the larger tensor is always the first argument) to maximize cache reuse.

**Pass 4 — Reassociation and Double Transpose Elimination (`ReorderingOptimizations`)**  
Reassociates chains of associative operations to minimize intermediate allocations. Eliminates `transpose(transpose(x))` → `x` and `transpose(x)` immediately followed by an op that accepts a transposed-input flag (e.g., `mmul(transpose(A), B)` → `mmul(A, B, transposeA=true)`).

**Pass 5 — Algebraic Identities (`AlgebraicOptimizations`)**  
Rewrites:
- `add(x, 0)` → `x`
- `mul(x, 1)` → `x`
- `mul(x, 0)` → `zeros_like(x)`
- `sqrt(square(x))` → `abs(x)`
- `pow(x, 0.5)` → `sqrt(x)`
- `log(1)` → `0`

**Pass 6 — Peephole Simplifications (`PeepholeOptimizations`)**  
Handles composite patterns:
- `relu(relu(x))` → `relu(x)` (idempotency)
- `log(exp(x))` → `x`
- `exp(log(x))` → `x`
- `neg(neg(x))` → `x`
- `abs(abs(x))` → `abs(x)`

**Pass 7 — Arithmetic Chain Folding (`ArithmeticChainOptimizations`)**  
Collapses chains of adds or multiplies involving constants into a single op:
- `add(add(x, c1), c2)` → `add(x, c1+c2)`
- `mul(mul(x, c1), c2)` → `mul(x, c1*c2)`
- `add(mul(x, c1), mul(x, c2))` → `mul(x, c1+c2)`

This is particularly effective after constant propagation exposes scalar constants in weight scaling paths.

**Pass 8 — Strength Reduction (`StrengthReductionOptimizations`)**  
Replaces expensive ops with cheaper equivalents:
- `pow(x, 2)` → `square(x)` (avoids the general pow kernel)
- `div(x, c)` → `mul(x, 1.0/c)` (multiplication is faster than division on most hardware)
- `pow(x, 0.5)` → `sqrt(x)`
- `pow(x, -1)` → `reciprocal(x)`

**Pass 9 — Identity Removal (`IdentityFunctionOptimizations`)**  
Removes `identity(x)` nodes that are sometimes inserted by the graph-building API or by import pipelines. Replaces all consumer edges with direct references to the identity's input.

**Pass 10 — Concat/Split Simplification (`ConcatSplitOptimizations`)**  
- `split(concat(a, b), axis=N, numSplits=2)` → `[a, b]` when the split exactly inverts the concat.
- `concat([x])` → `x` when concatenating a single tensor.
- `split(x, axis=N, numSplits=1)` → `[x]`.

**Pass 11 — Constant-Condition Select/Where (`SelectWhereOptimizations`)**  
When the condition tensor of a `select` or `where` op is a compile-time constant, replaces the entire op with the statically-chosen branch.

**Pass 12 — Redundancy Elimination (`RedundancyEliminationOptimizations`)**  
Removes duplicate computations that are not caught by CSE (pass 14) because they have distinct node identities in the graph. Specifically handles patterns from model import where the same subexpression may be emitted multiple times for different output consumers.

**Pass 13 — Shape Constant Folding (`ShapeFunctionOptimizations`)**  
Evaluates shape-query ops (`shape_of`, `rank`, `size_at`) whose inputs have statically known shapes, replacing them with constant integers. This often enables pass 2 (constant propagation) to eliminate further downstream computations on shapes.

**Pass 14 — Common Subexpression Elimination (`CommonSubexpressionElimination`)**  
Detects identical subgraphs (same op type, same inputs in the same order, same attributes) and merges them into a single node, redirecting all consumers to the merged output. Effective for attention masks, positional encodings, and layer norm statistics that are computed once but referenced in multiple branches.

**Pass 15 — Attention Fusion (`AttentionFusionOptimizations`)**  
The key pass for transformer acceleration. Detects the attention computation pattern:

```
scores = matmul(Q, transpose(K)) * scale
weights = softmax(scores)
output = matmul(weights, V)
```

and replaces it with a single `dot_product_attention_v2` op. On CUDA, this maps to FlashAttention or cuDNN fused attention, reducing memory traffic from O(N²) to O(N) in the sequence length. **This pass must run before HorizontalFusion (pass 16)** because horizontal fusion may merge the Q, K, V matmuls before attention fusion has a chance to detect the pattern.

**Pass 16 — Horizontal Fusion (`HorizontalFusionOptimizations`)**  
Detects parallel matmul chains that share a common input — most commonly the fused QKV projection in transformer self-attention:

```
Q = matmul(x, Wq)
K = matmul(x, Wk)
V = matmul(x, Wv)
```

These are fused into a single batched matmul against the stacked weight matrix `[Wq | Wk | Wv]`, followed by a slice to recover Q, K, and V. On Ampere and later GPUs, a single large GEMM achieves much higher FLOP utilization than three separate smaller GEMMs.

**Passes 17–26 — Additional Fusions and Specializations**  
The remaining passes include:
- Normalization fusion (fusing adjacent layer norm components into a single kernel)
- Gated delta network fusion (for architectures using gated linear units)
- Quantization optimization (folding quantization scale/zero-point constants and fusing dequant + op + requant)
- Further platform-specific strength reductions and memory layout canonicalization

---

## GPU JIT Compilation

When the execution mode includes JIT (Triton, NVRTC, or PTX), DSP groups consecutive element-compatible ops into fused kernel segments and compiles them into a single GPU kernel. Fusion eliminates redundant reads and writes of intermediate tensors.

### Kernel Segment Types

The `TritonIRBuilder` classifies each op into one of the following segment categories, used to determine fusion eligibility:

| Category | Examples |
|---|---|
| `ELEMENTWISE` | relu, add, mul, exp, log, sigmoid, tanh, cast |
| `MATMUL` | mmul, gemm, dot |
| `REDUCTION` | sum, mean, max, min, norm2 |
| `NORMALIZATION` | layer_norm, batch_norm, rms_norm |
| `ATTENTION` | dot_product_attention_v2, flash_attention |
| `GATHER` | gather, embedding_lookup |
| `SCATTER` | scatter_add, scatter_nd |
| `SLICE` | slice, strided_slice, gather_nd |
| `TILE` | tile, broadcast_to |
| `CONCAT` | concat |
| `SPLIT` | split |
| `SHAPE_MANIPULATION` | reshape, transpose, squeeze, unsqueeze |
| `CONV` | conv2d, depthwise_conv2d, conv_transpose |
| `IMAGE2COL` | im2col, col2im |

Consecutive ELEMENTWISE ops are always fusible. REDUCTION, NORMALIZATION, and ATTENTION ops form their own segment boundaries.

### Triton JIT Backend

Triton (MLIR-based GPU compiler) is the highest-fidelity JIT backend. It generates correct, highly optimized kernels for the full range of segment types.

`TritonIRBuilder` decomposes the fused segment into a sequence of Triton IR operations, then hands the IR to `TritonTargetDispatch` which selects the appropriate compilation target:

| Hardware | Target |
|---|---|
| NVIDIA GPUs | NVIDIA PTX (via Triton's LLVM NVPTX backend) |
| AMD GPUs | AMD AMDGCN (via Triton's LLVM AMDGPU backend) |
| Intel GPUs | Intel SPIR-V (via Triton's LLVM SPIR-V backend) |

A background precompilation thread compiles Triton kernels during warmup, so compiled kernels are ready before the CUDA graph capture begins. This avoids any JIT stall on the first post-warmup call.

`FusionScoring` heuristics evaluate whether fusing a group of ops will yield a net benefit: they estimate the memory-bandwidth savings from eliminating intermediates against the register pressure cost of the larger fused kernel. Groups that score below a threshold are left unfused.

### NVRTC JIT Backend

NVRTC (NVIDIA Runtime Compilation) compiles CUDA C++ source code at runtime using the NVRTC library bundled with the CUDA toolkit.

The NVRTC backend:
1. Generates a CUDA C++ translation unit from the fused segment's slot list.
2. Calls `nvrtcCompileProgram()` with architecture flags matching the active GPU's compute capability.
3. Extracts PTX from the compilation result via `nvrtcGetPTX()`.
4. Loads the PTX with `cuModuleLoadDataEx()` and retrieves the kernel entry point.

NVRTC produces code equivalent to Triton for ELEMENTWISE segments but is generally less effective for MATMUL and REDUCTION segments, where Triton's tiling and pipeline scheduling logic produces better code. NVRTC serves as the fallback when Triton is unavailable.

### PTX Backend

The PTX backend generates PTX assembly text directly via string templates, bypassing any higher-level compilation step. This gives the fastest compile time (usually under 1 ms for small elementwise kernels) at the cost of lower code quality for complex segments.

PTX assembly is loaded directly via `cuModuleLoadDataEx()`. The PTX backend is mainly used for simple elementwise fusions where template-generated PTX is already near-optimal, and as the final JIT fallback when neither Triton nor NVRTC are available.

### JIT Fallback Order

On CUDA hardware:
```
Triton  →  NVRTC  →  PTX  →  CUDA Graphs (no JIT)  →  slot-by-slot
```

---

## Multi-Backend Dispatch

DSP supports 17 execution modes, covering every major hardware platform. The active mode is controlled by the `GraphExecutionMode` enum.

### GraphExecutionMode Enum

| Value | ID | Description |
|---|---|---|
| `AUTO` | 0 | Let DSP choose the best available mode for the current hardware |
| `SLOT_BY_SLOT` | 1 | Interpreted slot-by-slot dispatch; always available |
| `CUDA_GRAPHS` | 2 | CUDA graph capture + replay; no JIT fusion |
| `TRITON` | 3 | Triton JIT + CUDA graph replay |
| `NVRTC` | 4 | NVRTC JIT + CUDA graph replay |
| `PTX` | 5 | PTX string-template JIT + CUDA graph replay |
| `ROCm` | 6 | AMD HIP graph capture + replay |
| `MLX` | 7 | Apple MLX graph backend (Apple Silicon) |
| `VULKAN` | 8 | Vulkan compute graphs |
| `LEVEL_ZERO` | 9 | Intel Level Zero command lists |
| `METAL` | 10 | Apple Metal command buffers |
| `ONNX_RUNTIME` | 11 | Delegate to ONNX Runtime execution provider |
| `OPENVINO` | 12 | Intel OpenVINO inference engine |
| `ACL` | 13 | ARM Compute Library graph backend |
| `NNAPI` | 14 | Android Neural Networks API |
| `ARM_HYBRID` | 15 | Hybrid CPU/GPU dispatch on ARM (Cortex-M + Mali) |
| `MLIR_CPU` | 16 | MLIR CPU dialects for JIT-compiled CPU kernels |

### AUTO Fallback Chains

When `AUTO` is selected, DSP probes available hardware and libraries to choose the best mode, falling back to the next option if a backend is unavailable:

**On CUDA hardware:**
```
Triton  →  NVRTC  →  PTX  →  CUDA_GRAPHS  →  SLOT_BY_SLOT
```

**On non-CUDA hardware:**
```
Triton  →  MLX  →  oneDNN  →  ACL  →  NNAPI  →  ARM_HYBRID  →  MLIR_CPU  →  SLOT_BY_SLOT
```

### Backend Implementations

Each backend provides a `GraphReplayHandle` implementation that wraps the platform's captured-graph replay API:

| Backend Class | Platform API |
|---|---|
| `CudaGraphReplayHandle` | `cudaGraphExec_t` / `cudaGraphLaunch()` |
| `HipGraphReplayHandle` | `hipGraphExec_t` / `hipGraphLaunch()` |
| `MetalReplayHandle` | `MTLCommandBuffer` / `[buffer commit]` |
| `VulkanReplayHandle` | `VkCommandBuffer` / `vkQueueSubmit()` |
| `LevelZeroReplayHandle` | `ze_command_list_t` / `zeCommandQueueExecuteCommandLists()` |
| `TpuReplayHandle` | XLA executable replay |
| `HexagonReplayHandle` | Qualcomm FastRPC handle |

CPU-side graph backends that provide op fusion without captured replay:

| Backend Class | Library |
|---|---|
| `OneDnnGraphBackend` | Intel oneDNN graph API |
| `AclGraphBackend` | ARM Compute Library graph mode |
| `MlxGraphBackend` | Apple MLX compute graph |
| `MlirCpuGraphBackend` | MLIR CPU dialects (Linalg, Vector, SCF) |
| `ArmHybridGraphBackend` | Mixed Cortex + Mali dispatch |
| `NnapiGraphBackend` | Android NNAPI model compilation |
| `OpenVinoGraphBackend` | OpenVINO CompiledModel |

### NativePlanCache

The native layer maintains an LRU plan cache (`NativePlanCache`) keyed on a content-based hash:

```
key = hash(outputSetHash, phShapeContentHash, phCount, graphExecutionMode, threadId)
```

`outputSetHash` encodes which output variables are requested. `phShapeContentHash` encodes the shapes of all placeholder inputs. Together these two hash components mean that different output sets or different input shapes produce different cache entries, each with their own warmup and captured graph.

Eviction uses a dual policy:
- **Count cap**: when the number of cached plans exceeds the configured maximum, the least-recently-used plan is evicted.
- **Memory budget**: when total plan memory exceeds a configurable fraction of available device memory, plans are evicted until the memory usage drops below the budget.

On eviction, all slot states for the evicted plan revert to `EVICTED`, triggering re-warmup on the next call.

---

## Java API Reference

### DynamicShapePlanCompiler

`DynamicShapePlanCompiler` converts a `ForwardExecutionDAG` into a `DynamicShapePlan`. You do not call this directly during normal inference — DSP invokes it automatically the first time you call `sd.output()` with DSP enabled. It is useful to call explicitly when you want to pre-compile the plan at startup.

**Compilation pipeline:**
1. Filter ops: remove dead nodes (applies optimizer first if enabled)
2. Build external input index maps: assign integer indices to placeholder variables
3. Assign slot indices: give each surviving node a sequential slot index
4. Build input wiring: record which slot's output each slot reads
5. Liveness analysis: determine the last step at which each slot's output is needed
6. Build `releaseAtStep` table: encode when each intermediate buffer can be freed
7. Pre-allocate `OpContext` pool: create a pool of reusable op execution contexts

```java
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.autodiff.samediff.dsp.DynamicShapePlanCompiler;
import org.nd4j.autodiff.samediff.dsp.DynamicShapePlan;

SameDiff sd = SameDiff.load(new File("model.fb"), true);

// Explicitly pre-compile the plan for a given input shape
DynamicShapePlanCompiler compiler = new DynamicShapePlanCompiler(sd);
DynamicShapePlan plan = compiler.compile(
    List.of("softmax"),           // requested outputs
    Map.of("input", new long[]{1, 768})  // placeholder shape hints
);
```

### DynamicShapePlanExecutor

`DynamicShapePlanExecutor` drives the full lifecycle. It owns the plan-phase state machine and selects the appropriate dispatch path (warmup, slot-by-slot, capture, or replay) for each call.

```java
import org.nd4j.autodiff.samediff.dsp.DynamicShapePlanExecutor;
import org.nd4j.linalg.api.ndarray.INDArray;

// The executor is normally created and managed by SameDiff internally.
// Access it for direct control:
DynamicShapePlanExecutor executor = sd.getDspExecutor();

// Run a forward pass; internally selects the correct phase dispatch
Map<String, INDArray> result = executor.execute(
    Map.of("input", inputTensor),
    List.of("softmax")
);
```

Key internal fields (useful for diagnostics):
- `tl_dspExecutionStream` — per-thread CUDA stream used for main execution
- `tl_dspGapStream` — per-thread stream for gap ops
- `argTableStable` — boolean fast path: when true, skips pointer-stability verification on replay

### PlanIntrospection

`PlanIntrospection` provides read-only access to the compiled plan's internal structure. Use it to verify compilation results or to diagnose unexpected fallbacks.

```java
import org.nd4j.autodiff.samediff.dsp.PlanIntrospection;

PlanIntrospection intro = sd.getDspExecutor().getIntrospection();

int slotCount = intro.getSlotCount();
System.out.println("Compiled slot count: " + slotCount);

for (int i = 0; i < slotCount; i++) {
    String state     = intro.getSlotState(i);       // e.g., "REPLAYING"
    int[] wiring     = intro.getInputWiring(i);     // slot indices of inputs
    int[] liveness   = intro.getLivenessRange(i);   // [firstStep, lastStep]

    System.out.printf("Slot %3d  state=%-12s  inputs=%s  live=%d..%d%n",
        i, state, Arrays.toString(wiring), liveness[0], liveness[1]);
}
```

### DspDiagnostics

`DspDiagnostics` provides structured performance and correctness diagnostics across 20 categories. It can emit a full JSON report for offline analysis.

```java
import org.nd4j.autodiff.samediff.dsp.DspDiagnostics;
import org.nd4j.autodiff.samediff.dsp.DspDiagnostics.Category;
import org.nd4j.autodiff.samediff.dsp.DspDiagnostics.Level;

DspDiagnostics diag = sd.getDspExecutor().getDiagnostics();

// Enable specific categories at the desired verbosity level
diag.enable(Category.COMPILE,      Level.DETAILED);
diag.enable(Category.EXECUTE,      Level.SUMMARY);
diag.enable(Category.GRAPH_REPLAY, Level.FULL);
diag.enable(Category.MEMORY,       Level.SUMMARY);

// Run some inference
sd.output(Map.of("input", testBatch), "softmax");

// Print a JSON report to stdout or a file
String json = diag.toJson();
System.out.println(json);
Files.writeString(Path.of("dsp_report.json"), json);
```

Available diagnostic categories: `COMPILE`, `EXECUTE`, `MEMORY`, `GRAPH_REPLAY`, `JIT`, `OPTIMIZER`, `CACHE`, `WARMUP`, `CAPTURE`, `LIVENESS`, `WIRING`, `SHAPES`, `GAPS`, `TENSOR_PARALLEL`, `PIPELINE_PARALLEL`, `SLOT_STATE`, `FALLBACK`, `STREAM`, `POINTER`, `LATENCY`.

### DspDebugger

`DspDebugger` enables interactive step-by-step execution for diagnosing numerical issues or unexpected behavior. It inserts breakpoints at specified slot indices and pauses execution so you can inspect intermediate tensor values.

```java
import org.nd4j.autodiff.samediff.dsp.DspDebugger;

DspDebugger dbg = new DspDebugger(sd);

// Set a breakpoint at slot 42
dbg.addBreakpoint(42, slot -> {
    System.out.println("At slot " + slot.getIndex()
        + " (" + slot.getVariableName() + "): "
        + slot.getValue().shapeInfoToString());
    // Inspect value:
    System.out.println(slot.getValue());
});

// Execute with debugger active (forces SLOT_BY_SLOT mode)
Map<String, INDArray> result = dbg.execute(
    Map.of("input", testBatch),
    List.of("softmax")
);
```

---

## Disk Cache and Diagnostics

### DspPlanDiskCache

Compiled plans are serialized to disk so that the warmup and JIT compilation costs are paid only once across JVM restarts. The disk cache uses FNV-1a hashing for cache key construction.

**Cache location:** `~/.kompile/cache/dsp/`  
**Cache version:** `DSP_VERSION=5` (cache entries from older versions are automatically invalidated)  
**Write protocol:** Atomic temp-file + `Files.move(ATOMIC_MOVE)` to prevent corrupt entries on crash.

```java
// Enable/disable disk cache (enabled by default when nd4j-cuda is present)
// -Dnd4j.dsp.planCache.diskEnabled=true|false

// The cache location can be overridden:
// -Dnd4j.dsp.planCache.dir=/path/to/custom/cache

// Check cache hit/miss via diagnostics
DspDiagnostics diag = sd.getDspExecutor().getDiagnostics();
diag.enable(Category.CACHE, Level.SUMMARY);
String report = diag.toJson();
// Look for "cacheHit": true/false in the report
```

Cache entries are keyed on the same content hash used by `NativePlanCache`:
```
hash(outputSetHash + phShapeContentHash + phCount + graphExecutionMode + threadId)
```

When a disk-cached plan is loaded, the executor skips warmup and proceeds directly to the CUDA graph capture phase (if CUDA is available and the shapes match).

### NativePlanCache

The in-process native cache is separate from the disk cache. It is an LRU cache with:
- Maximum entry count (configurable; default varies by available GPU memory)
- Memory budget: combined size of all cached plans must stay below a fraction of device memory

When device memory is tight (e.g., running multiple models concurrently), the memory-budget eviction policy takes precedence and can evict plans even if the count cap has not been reached.

---

## Parallel Execution

DSP includes native support for both tensor parallelism and pipeline parallelism, allowing large models to span multiple GPUs.

### Tensor Parallelism

Tensor parallelism splits individual weight matrices across GPUs. For a linear layer with weight matrix W, `ColumnParallelLinear` shards W along the column dimension across N GPUs, so each GPU holds W[:,k/N:(k+1)/N]. `RowParallelLinear` shards along the row dimension.

Each GPU computes a partial result and an all-reduce collective synchronizes the results.

```java
import org.nd4j.autodiff.samediff.dsp.parallel.TensorParallelConfig;
import org.nd4j.autodiff.samediff.dsp.parallel.TensorParallelRunner;

TensorParallelConfig config = TensorParallelConfig.builder()
    .numDevices(4)          // spread across 4 GPUs
    .useNccl(true)          // use NCCL for all-reduce
    .build();

TensorParallelRunner runner = new TensorParallelRunner(sd, config);
Map<String, INDArray> result = runner.execute(
    Map.of("input", inputBatch),
    List.of("logits")
);
```

The `CollectiveCommunicator` interface abstracts the all-reduce communication:
- `LocalCollectiveCommunicator` — uses shared CPU memory for multi-GPU machines where all GPUs have NVLink or are on the same PCIe root complex
- `NcclCommunicator` — uses NCCL for higher-throughput all-reduce, required for multi-node configurations

### Pipeline Parallelism

Pipeline parallelism partitions model layers across GPUs. Each GPU holds a "stage" of the model. Micro-batches are interleaved through the pipeline to maintain GPU utilization.

```java
import org.nd4j.autodiff.samediff.dsp.parallel.PipelineParallelRunner;

PipelineParallelRunner pipeline = PipelineParallelRunner.builder()
    .model(sd)
    .numStages(4)                       // 4 GPUs, one stage each
    .microBatchSize(8)                  // split each batch into micro-batches of 8
    .interleave(true)                   // 1F1B interleaved schedule
    .build();

Map<String, INDArray> result = pipeline.execute(
    Map.of("input", largeBatch),
    List.of("logits")
);
```

With `interleave=true`, `PipelineParallelRunner` uses a 1F1B (one-forward-one-backward) schedule for training or a fill-and-drain schedule for inference, keeping all stages occupied with different micro-batches simultaneously.

---

## DSP Runtime SDK

The DSP Runtime SDK exposes a stable C ABI defined in `dsp_runtime_c.h`. The ABI version is `SDX_RUNTIME_ABI_VERSION=1`.

All structs in the C ABI use a sized-struct pattern for forward compatibility: the first member of every struct is `size_t struct_size`, which the caller sets to `sizeof(the_struct)`. The runtime checks this field and zero-initializes any fields the caller did not set (because the caller's struct is smaller than the runtime's version). This allows new fields to be added in future ABI versions without breaking existing binaries.

### C API Lifecycle

```c
#include "dsp_runtime_c.h"

// 1. Create the runtime context
sdx_create_info_t create_info = {
    .struct_size = sizeof(sdx_create_info_t),
    .execution_mode = SDX_MODE_AUTO,
    .num_warmup_iterations = 1
};
sdx_runtime_t* runtime = NULL;
sdx_status_t status = sdxCreateRuntime(&create_info, &runtime);

// 2. Load a compiled plan bundle from disk
sdx_bundle_t* bundle = NULL;
status = sdxLoadBundle(runtime, "/path/to/model.sdx", &bundle);

// 3. Create an inference context (one per thread for concurrent inference)
sdx_context_t* ctx = NULL;
status = sdxCreateContext(runtime, bundle, &ctx);

// 4. Mark inputs and run inference
sdx_tensor_t input_tensor = {
    .struct_size = sizeof(sdx_tensor_t),
    .data = myFloatBuffer,
    .dims = (int64_t[]){1, 768},
    .ndim = 2,
    .dtype = SDX_DTYPE_FLOAT32
};
sdxMarkInputPlaceholder(ctx, "input", &input_tensor);

sdx_run_info_t run_info = { .struct_size = sizeof(sdx_run_info_t) };
sdx_execution_report_t report;
status = sdxRun(ctx, &run_info, &report);

printf("Mode: %d, Replay count: %d, Avg latency: %.1f µs, Peak mem: %zu bytes\n",
       report.executionMode, report.replayCount,
       report.avgLatencyUs, report.peakMemoryBytes);

// 5. Retrieve outputs, then clean up
sdxDestroyContext(ctx);
sdxUnloadBundle(bundle);
sdxDestroyRuntime(runtime);
```

### sdx_execution_report_t Fields

| Field | Type | Description |
|---|---|---|
| `struct_size` | `size_t` | Sized-struct discriminator |
| `slotCount` | `int32_t` | Number of slots in the compiled plan |
| `executionMode` | `int32_t` | Actual mode used (may differ from requested `AUTO`) |
| `replayCount` | `int64_t` | Number of times the captured graph has been replayed |
| `avgLatencyUs` | `float` | Rolling average per-call latency in microseconds |
| `peakMemoryBytes` | `size_t` | Peak device memory consumed by the plan |

### Shape Control Functions

```c
// Freeze shapes explicitly (normally done automatically after warmup)
sdxFreezeShapes(ctx);

// Query the current plan phase
sdx_plan_phase_t phase = sdxGetPlanPhase(ctx);
// Returns: SDX_PHASE_WARMUP, SDX_PHASE_SHAPES_FROZEN,
//          SDX_PHASE_CUDA_GRAPH_CAPTURED, or SDX_PHASE_REPLAYING

// Mark a variable as a named model input
sdxMarkInputVariable(ctx, "embedding_weight", &weight_tensor);

// Mark a placeholder input for the current call
sdxMarkInputPlaceholder(ctx, "tokens", &token_tensor);
```

### Language Bindings

#### Java (JNA)

```java
import com.kompile.dsp.runtime.DspRuntime;
import com.kompile.dsp.runtime.DspContext;
import com.kompile.dsp.runtime.DspBundle;

DspRuntime runtime = DspRuntime.create(DspRuntime.Mode.AUTO);
DspBundle bundle = runtime.loadBundle(Path.of("model.sdx"));
DspContext ctx = runtime.createContext(bundle);

ctx.setInputPlaceholder("input", inputArray);   // INDArray
DspExecutionReport report = ctx.run();

INDArray output = ctx.getOutput("softmax");

System.out.printf("Latency: %.1f µs%n", report.avgLatencyUs());

ctx.close();
bundle.close();
runtime.close();
```

#### Kotlin (JNA Wrappers)

```kotlin
import com.kompile.dsp.runtime.DspRuntime

DspRuntime.create(DspRuntime.Mode.AUTO).use { runtime ->
    runtime.loadBundle(Path.of("model.sdx")).use { bundle ->
        runtime.createContext(bundle).use { ctx ->
            ctx.setInputPlaceholder("input", inputArray)
            val report = ctx.run()
            val output = ctx.getOutput("softmax")
            println("Latency: ${report.avgLatencyUs} µs")
        }
    }
}
```

#### Python (ctypes + NumPy)

```python
import numpy as np
from kompile.dsp import DspRuntime, Mode

runtime = DspRuntime(mode=Mode.AUTO)
bundle = runtime.load_bundle("model.sdx")
ctx = runtime.create_context(bundle)

# Pass a NumPy array directly
input_data = np.random.randn(1, 768).astype(np.float32)
ctx.set_input("input", input_data)

report = ctx.run()
output = ctx.get_output("softmax")   # returns np.ndarray

print(f"Mode: {report.execution_mode}, Latency: {report.avg_latency_us:.1f} µs")
print(f"Output shape: {output.shape}")

ctx.close()
bundle.close()
runtime.close()
```

The Python binding also exposes a gRPC server mode for serving DSP plans over a network:

```python
from kompile.dsp.server import DspGrpcServer

server = DspGrpcServer(bundle="model.sdx", port=50051, mode=Mode.AUTO)
server.serve()   # blocks; accepts gRPC inference requests
```

#### Rust (FFI + RAII)

```rust
use kompile_dsp::{Runtime, Mode};

let runtime = Runtime::new(Mode::Auto)?;
let bundle = runtime.load_bundle("model.sdx")?;
let mut ctx = runtime.create_context(&bundle)?;

let input = ndarray::Array2::<f32>::zeros((1, 768));
ctx.set_input_placeholder("input", &input)?;

let report = ctx.run()?;
let output = ctx.get_output::<f32>("softmax")?;

println!("Latency: {:.1} µs", report.avg_latency_us);
println!("Output shape: {:?}", output.shape());
// RAII: ctx, bundle, runtime drop in reverse order
```

The Rust binding uses a thin FFI layer over `dsp_runtime_c.h` with RAII wrappers (`Drop` implementations) ensuring correct cleanup order.

#### Swift (Swift Package Manager)

```swift
import DspRuntime

let runtime = try DspRuntime(mode: .auto)
let bundle  = try runtime.loadBundle(url: URL(fileURLWithPath: "model.sdx"))
let ctx     = try runtime.createContext(bundle: bundle)

let input = [Float](repeating: 0.0, count: 768)
try ctx.setInputPlaceholder("input", data: input, shape: [1, 768])

let report = try ctx.run()
let output: [Float] = try ctx.getOutput("softmax")

print("Latency: \(report.avgLatencyUs) µs")
print("Output count: \(output.count)")
```

The Swift binding is distributed as a Swift Package and supports both macOS (Metal backend) and iOS (Metal + CoreML backends).

#### C# (P/Invoke)

```csharp
using Kompile.Dsp;

using var runtime = new DspRuntime(Mode.Auto);
using var bundle  = runtime.LoadBundle("model.sdx");
using var ctx     = runtime.CreateContext(bundle);

float[] inputData = new float[768];
ctx.SetInputPlaceholder("input", inputData, new long[] { 1, 768 });

var report = ctx.Run();
float[] output = ctx.GetOutput("softmax");

Console.WriteLine($"Latency: {report.AvgLatencyUs:F1} µs");
Console.WriteLine($"Output length: {output.Length}");
```

---

## Configuration Reference

All DSP behaviors are controlled via JVM system properties, environment variables, or the `DspConfig` builder API.

### Core Execution

| Property | Default | Description |
|---|---|---|
| `-Dnd4j.dsp.enabled` | `true` | Enable DSP compilation. Set `false` to fall back to the legacy InferenceSession executor. |
| `-Dnd4j.dsp.executionMode` | `AUTO` | Override the `GraphExecutionMode`. Accepts mode names (`TRITON`, `CUDA_GRAPHS`, `SLOT_BY_SLOT`, etc.) or integer IDs. |
| `-Dnd4j.dsp.warmupIterations` | `1` | Number of slot-by-slot warmup iterations before shape freeze. Increase if input shapes vary across the first few calls. |
| `-Dnd4j.dsp.argTableStable` | `true` | Enable the pointer-stability fast path on replay. Disable only when input buffer addresses change between calls. |
| `-Dnd4j.dsp.gapStreamEnabled` | `true` | Enable the separate gap stream. Disable to route gap ops to the main execution stream (simpler but may serialize with captured graph). |

### Graph Optimizer

| Property | Default | Description |
|---|---|---|
| `-Dnd4j.optimizer.skip` | _(none)_ | Comma-separated list of optimizer pass class names to skip. Example: `AttentionFusionOptimizations,HorizontalFusionOptimizations` |
| `-Dnd4j.optimizer.logApplied` | `false` | Log each rewrite rule application (op before → after) to the SLF4J logger at DEBUG level. |
| `-Dnd4j.optimizer.maxIterations` | `3` | Number of outer optimization iterations. Increase to allow more fixpoint convergence at the cost of compile time. |
| `-Dnd4j.optimizer.enabled` | `true` | Enable the graph optimizer entirely. Set `false` to disable all 26 passes. |

### JIT Compilation

| Property | Default | Description |
|---|---|---|
| `-Dnd4j.dsp.jit.tritonEnabled` | `true` | Enable the Triton JIT backend. |
| `-Dnd4j.dsp.jit.nvrtcEnabled` | `true` | Enable the NVRTC JIT backend. |
| `-Dnd4j.dsp.jit.ptxEnabled` | `true` | Enable the PTX string-template JIT backend. |
| `-Dnd4j.dsp.jit.backgroundCompile` | `true` | Compile Triton kernels on a background thread during warmup. |
| `-Dnd4j.dsp.jit.fusionScoreThreshold` | `0.5` | Minimum FusionScoring score to apply a fusion. Lower values fuse more aggressively; higher values are more conservative. |

### Disk Cache

| Property | Default | Description |
|---|---|---|
| `-Dnd4j.dsp.planCache.diskEnabled` | `true` | Enable serializing compiled plans to disk. |
| `-Dnd4j.dsp.planCache.dir` | `~/.kompile/cache/dsp/` | Directory for disk-cached plans. |
| `-Dnd4j.dsp.planCache.maxEntries` | `64` | Maximum number of plans to retain on disk. |
| `-Dnd4j.dsp.planCache.version` | `5` | Cache format version. Do not change; used for automatic invalidation. |

### Native Plan Cache

| Property | Default | Description |
|---|---|---|
| `-Dnd4j.dsp.nativeCache.maxCount` | `32` | Maximum number of plans in the in-process LRU cache. |
| `-Dnd4j.dsp.nativeCache.memoryBudgetFraction` | `0.25` | Maximum fraction of device memory that in-process cached plans may consume. |

### Diagnostics

| Property | Default | Description |
|---|---|---|
| `-Dnd4j.dsp.diagnostics.enabled` | `false` | Enable the `DspDiagnostics` subsystem. |
| `-Dnd4j.dsp.diagnostics.categories` | _(all)_ | Comma-separated list of `DspDiagnostics.Category` names to enable. |
| `-Dnd4j.dsp.diagnostics.level` | `SUMMARY` | Default verbosity level (`SUMMARY`, `DETAILED`, or `FULL`). |
| `-Dnd4j.dsp.diagnostics.jsonOutput` | _(none)_ | Path to write the JSON diagnostic report on JVM exit. |

### Parallel Execution

| Property | Default | Description |
|---|---|---|
| `-Dnd4j.dsp.tensorParallel.numDevices` | `1` | Number of GPUs for tensor parallelism. `1` disables tensor parallelism. |
| `-Dnd4j.dsp.tensorParallel.useNccl` | `true` | Use NCCL for all-reduce collectives. Set `false` to use `LocalCollectiveCommunicator` (shared memory). |
| `-Dnd4j.dsp.pipelineParallel.numStages` | `1` | Number of pipeline stages. `1` disables pipeline parallelism. |
| `-Dnd4j.dsp.pipelineParallel.microBatchSize` | `8` | Micro-batch size for pipeline interleaving. |

---

## See Also

- [SameDiff Execution and Inference](./execution) — standard `sd.output()` / `sd.exec()` API without DSP
- [SameDiff Overview](./overview) — define-and-run graph model, variable types, training
- [CUDA Backend](../backends/cuda) — GPU memory management, multi-GPU setup, cuDNN
- [CPU Backend](../backends/cpu) — AVX tuning, BLAS configuration, threading
- [Memory and Workspaces](../../core-concepts/memory-and-workspaces) — workspace scopes, GPU memory reuse

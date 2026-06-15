---
title: New Operations Reference
description: Complete reference for ~130 new native operations — fused attention, KV cache, PEFT linear layers, normalization, positional encoding, quantization, SSM, MoE, and audio/signal processing
---

# New Operations Reference

This page documents the approximately 130 new native operations introduced in Deeplearning4j 1.0.0-rewrite (PRs #10446 and #10445). All operations extend `DynamicCustomOp` and are registered in the ND4J op registry. Backward passes (`*Bp` variants) are included for every differentiable operation.

Operations span ten functional areas:

- [Fused Attention Ops](#1-fused-attention-ops)
- [KV Cache Ops](#2-kv-cache-ops)
- [PEFT Linear Layers](#3-peft-linear-layers)
- [Normalization Ops](#4-normalization-ops)
- [Activation Ops](#5-activation-ops)
- [Positional Encoding Ops](#6-positional-encoding-ops)
- [Quantization and Sampling Ops](#7-quantization-and-sampling-ops)
- [SSM / Recurrent Ops](#8-ssm--recurrent-ops)
- [Mixture of Experts](#9-mixture-of-experts)
- [Audio and Signal Processing](#10-audio-and-signal-processing)
- [Usage Examples](#11-usage-examples)

---

## 1. Fused Attention Ops

These operations implement the full family of modern attention mechanisms used in transformer architectures. All fused kernels avoid materializing the full attention matrix where possible, reducing memory footprint from O(n²) to O(n) in compatible configurations.

### FlashAttention / FlashAttentionBp

Implements the Flash Attention algorithm (Dao et al., 2022 / 2023) for IO-aware exact attention using tiled SRAM computation.

| Item | Detail |
|------|--------|
| **Inputs** | `Q` (query), `K` (key), `V` (value) — all shape `[batch, heads, seq, head_dim]`; optional causal mask `[batch, 1, seq, seq]` |
| **Outputs** | attention output `[batch, heads, seq, head_dim]` |
| **Float args** | `scale` — QK scale factor, typically `1 / sqrt(head_dim)` |
| **Bool args** | `causal` — whether to apply a causal (lower-triangular) mask |
| **Backward** | `FlashAttentionBp` receives upstream gradient and recomputes tiles on the fly without storing the N×N attention matrix |

Supports both multi-head attention (MHA) and grouped-query attention (GQA) layouts. When the number of KV heads is less than query heads, keys and values are broadcast across the corresponding query head groups automatically.

### GroupedQueryAttention / GroupedQueryAttentionBp

Grouped-Query Attention (GQA) as used in LLaMA 3 and Gemma. Reduces KV cache memory by sharing key/value heads across groups of query heads.

| Item | Detail |
|------|--------|
| **Inputs** | `Q` `[batch, num_heads, seq, head_dim]`, `K` `[batch, num_kv_heads, seq, head_dim]`, `V` same as K |
| **Int args** | `num_heads`, `num_kv_heads` — must satisfy `num_heads % num_kv_heads == 0` |
| **Float args** | `scale` |
| **Notes** | Each KV head is shared by `num_heads / num_kv_heads` query heads. Use `num_kv_heads == 1` for Multi-Query Attention (MQA). |

### MLAAttention

Multi-head Latent Attention as introduced in DeepSeek-V3. Compresses the KV cache by projecting keys and values into a low-dimensional latent space before attention computation.

| Item | Detail |
|------|--------|
| **Inputs** | `Q`, compressed latent `C_KV`, projection matrices `W_UK`, `W_UV` |
| **Purpose** | Reduces the per-token KV cache size from `2 * num_heads * head_dim` to a fixed latent dimension, enabling much longer context at equal memory |

### CascadeAttention

Chunked-prefill and long-context decoding via cascade attention. Splits the sequence into chunks, computes local attention per chunk, then merges outputs with a log-sum-exp reduction over chunk softmax normalizers.

| Item | Detail |
|------|--------|
| **Inputs** | `Q`, `K`, `V`, chunk boundary indices `[num_chunks]` |
| **Use case** | Sequences longer than SRAM capacity; streaming / incremental decoding |

### DecoderMaskedMha

Decoder-only masked multi-head attention with explicit KV injection for inference. Applies the causal mask and attends over a provided KV cache tensor rather than recomputing keys and values from the current sequence.

| Item | Detail |
|------|--------|
| **Inputs** | `Q` (current token queries), `K_cache` (cached keys), `V_cache` (cached values), optional attention bias |
| **Int args** | `past_seq_len` — number of already-decoded tokens present in the cache |

### LightningAttention

Linear attention for efficient sequence modeling. Replaces the softmax kernel with a feature map approximation, reducing attention complexity from O(n²d) to O(nd²).

| Item | Detail |
|------|--------|
| **Inputs** | `Q`, `K`, `V`; optional feature map type selector |
| **Int args** | `feature_map` — 0 for `elu+1`, 1 for random Fourier features |
| **Notes** | Exact when `feature_map == -1` (falls back to standard dot-product attention) |

### SlidingWindowAttention

Mistral-style sliding-window attention. Each token attends only to a fixed-size local window of past tokens, reducing memory from O(n²) to O(n * window_size).

| Item | Detail |
|------|--------|
| **Inputs** | `Q`, `K`, `V` |
| **Int args** | `window_size` — number of tokens each position can attend to |
| **Notes** | Tokens outside the window receive `-inf` before softmax |

### OnnxMultiHeadAttention

ONNX-compatible multi-head attention conforming to the ONNX opset 17 `MultiHeadAttention` specification. Accepts packed QKV or separate Q/K/V inputs plus optional bias tensors.

| Item | Detail |
|------|--------|
| **Inputs** | `Q`, `K`, `V`, `bias` (optional), `key_padding_mask` (optional), `attention_bias` (optional) |
| **Int args** | `num_heads` |

### TwoWayCrossAttention / TwoWayCrossAttentionBp

Bidirectional cross-attention used in vision-language models such as SmolDocling and SAM (Segment Anything Model). Interleaves two streams of cross-attention: image-to-text and text-to-image in a single fused kernel.

| Item | Detail |
|------|--------|
| **Inputs** | `Q1` (stream A queries), `K1`/`V1` (stream A KV), `Q2` (stream B queries), `K2`/`V2` (stream B KV) |
| **Outputs** | Two attention outputs, one per stream |

### DotProductAttentionV2 / DotProductAttentionV2Bp

Updated V2 dot-product attention with improved numerical stability and support for both pre-scale and post-scale modes. Replaces the original `DotProductAttention` op.

| Item | Detail |
|------|--------|
| **Inputs** | `Q`, `K`, `V`, optional mask |
| **Float args** | `scale`, `dropout_prob` |
| **Bool args** | `pre_scale` — apply scale before or after QK^T |

### PagedAttentionForward / PagedKvAppend

Paged KV cache attention, analogous to virtual memory paging for KV tensors. `PagedAttentionForward` performs attention over a non-contiguous block table. `PagedKvAppend` appends new key/value vectors to the paged store.

| Item | Detail |
|------|--------|
| **Inputs (Forward)** | `Q`, `K_cache`, `V_cache`, `block_table` `[batch, max_blocks_per_seq]`, `seq_lens` `[batch]` |
| **Inputs (Append)** | new `K` and `V` slices, `block_table`, `slot_mapping` |
| **Int args** | `block_size` — number of tokens per block (typically 16 or 32) |

---

## 2. KV Cache Ops

These operations manage the key-value cache used during autoregressive decoding. They are stateful operations intended to be executed in-place against a persistent cache tensor.

### KVCache

An in-place state-holder for the KV cache. Allocates or validates a pre-allocated buffer that will hold key and value tensors across decoding steps.

| Item | Detail |
|------|--------|
| **Inputs** | initial `K` and `V` tensors, or shape specification for pre-allocation |
| **Int args** | `max_seq_len`, `num_heads`, `head_dim` |

### KVCacheUpdate

Scatters new key and value vectors into the KV cache at the current decode position.

| Item | Detail |
|------|--------|
| **Inputs** | `K_cache` (mutable), `V_cache` (mutable), new `K` slice, new `V` slice, position index |
| **Notes** | Performs in-place scatter; the cache tensors must be writable and allocated to their full capacity |

### KVCacheQuantize / KVCacheDeQuantize

INT8 and FP8 quantization of KV cache entries to reduce memory bandwidth during long-context generation.

| Item | Detail |
|------|--------|
| **KVCacheQuantize inputs** | FP16 or BF16 `K` and `V`, output scale and zero-point tensors |
| **KVCacheQuantize outputs** | quantized `K_q`, `V_q` plus per-tensor or per-channel `scale` and `zero_point` |
| **KVCacheDeQuantize inputs** | quantized `K_q`, `V_q`, `scale`, `zero_point` |
| **Int args** | `quant_type` — 0 for INT8, 1 for FP8 E4M3, 2 for FP8 E5M2 |

### KvScatter

Scatters key and value vectors into paged KV blocks according to a slot mapping, used together with `PagedAttentionForward`.

| Item | Detail |
|------|--------|
| **Inputs** | `K`, `V` (new tokens), `K_cache`, `V_cache` (block storage), `slot_mapping` `[num_tokens]` |
| **Notes** | Each entry in `slot_mapping` identifies the physical block slot for the corresponding token |

### SharedKvAttention

Multi-query attention with explicitly shared KV heads across all query heads. Equivalent to GQA with `num_kv_heads == 1` but with a dedicated optimized kernel path.

| Item | Detail |
|------|--------|
| **Inputs** | `Q` `[batch, num_heads, seq, head_dim]`, single `K` `[batch, 1, seq, head_dim]`, single `V` same shape |
| **Float args** | `scale` |

---

## 3. PEFT Linear Layers

Parameter-Efficient Fine-Tuning (PEFT) linear layers implement various low-rank adaptation strategies. All follow the convention that the base weight is frozen and only the adapter parameters are trained. All variants include backward passes for adapter weight gradients.

### LoraMatMul / LoraMatMulBp

Low-Rank Adaptation (LoRA). Computes:

```
output = input @ W^T + scaling * (input @ A^T @ B^T)
```

where `W` is the frozen base weight, `A` is shape `[rank, in_features]`, `B` is shape `[out_features, rank]`, and `scaling = lora_alpha / rank`.

| Item | Detail |
|------|--------|
| **Inputs** | `input`, `W` (frozen), `A`, `B` |
| **Float args** | `scaling` — the combined `alpha / rank` scalar |
| **Backward** | Computes gradients for `A` and `B` only; `W` gradient is zeroed |

### DoraMatMul / DoraMatMulBp

Weight-Decomposed LoRA (DoRA). Decomposes the effective weight into a magnitude vector and a directional component:

```
W_eff = m * (W + delta_W) / ||W + delta_W||_col
output = input @ W_eff^T
```

where `m` is a learnable per-column magnitude vector and `delta_W = B @ A` is the LoRA delta.

| Item | Detail |
|------|--------|
| **Inputs** | `input`, `W` (frozen), `A`, `B`, `m` (magnitude vector, shape `[out_features]`) |

### LohaMatMul / LohaMatMulBp

Low-rank Hadamard adaptation (LoHa). Adapts the weight via element-wise (Hadamard) product of two low-rank factor pairs:

```
W_eff = W + W1 ⊙ W2
W1 = B1 @ A1,   W2 = B2 @ A2
```

| Item | Detail |
|------|--------|
| **Inputs** | `input`, `W` (frozen), `A1`, `B1`, `A2`, `B2` |

### LokrMatMul / LokrMatMulBp

Low-rank Kronecker product adaptation (LoKr). Builds the adapter as a Kronecker product of small factor matrices:

```
delta_W = (B1 ⊗ B2) @ (A1 ⊗ A2)
output  = input @ (W + scaling * delta_W)^T
```

| Item | Detail |
|------|--------|
| **Inputs** | `input`, `W`, `A1`, `A2`, `B1`, `B2` |
| **Float args** | `scaling` |

### MultiLoraMatmul

Batched multi-adapter inference. Applies multiple distinct LoRA adapters to a single base weight in a single kernel launch, with each sample in the batch potentially using a different adapter index.

| Item | Detail |
|------|--------|
| **Inputs** | `input` `[batch, seq, in_features]`, `W`, stacked `A` `[num_adapters, rank, in_features]`, stacked `B` `[num_adapters, out_features, rank]`, `adapter_ids` `[batch]` |
| **Notes** | Avoids loop overhead and separate kernel launches per adapter |

### ColumnParallelLinear / RowParallelLinear

Tensor-parallel linear layers for multi-GPU model parallelism. Split the weight matrix across devices by columns or rows respectively, following Megatron-LM conventions.

| Item | Detail |
|------|--------|
| **ColumnParallel** | Each rank holds `W[:, start:end]`; all-gather is applied to outputs |
| **RowParallel** | Each rank holds `W[start:end, :]`; all-reduce is applied to partial sums |
| **Int args** | `tp_rank`, `tp_world_size` |

---

## 4. Normalization Ops

### RmsNorm / RmsNormBp

Root Mean Square Layer Normalization. Normalizes by RMS rather than mean and variance, omitting the re-centering step:

```
output = x * rsqrt(mean(x²) + eps) * gamma
```

| Item | Detail |
|------|--------|
| **Inputs** | `x`, learnable scale `gamma` (shape `[hidden_size]`) |
| **Float args** | `eps` — small constant for numerical stability (default `1e-6`) |
| **Notes** | No `beta` (bias) parameter; this matches the LLaMA and Mistral implementations |

### RmsNormLinear / RmsNormLinearBp

Fused RMSNorm followed immediately by a linear projection. Avoids a separate kernel launch and intermediate tensor allocation:

```
output = RmsNorm(x, gamma, eps) @ W^T
```

| Item | Detail |
|------|--------|
| **Inputs** | `x`, `gamma`, `W` |
| **Float args** | `eps` |

### SkipRmsNorm

RMSNorm with a residual skip connection added before normalization. Common in models that apply the skip inside the norm rather than outside:

```
output = RmsNorm(x + residual, gamma, eps)
```

| Item | Detail |
|------|--------|
| **Inputs** | `x`, `residual` (same shape as `x`), `gamma` |
| **Outputs** | normalized output; optionally also `x + residual` for the next residual path |

### FusedRmsNormSwiGLU / FusedRmsNormSwiGLUBp

Fused RMSNorm + SwiGLU gate for MLP blocks. Applies normalization then the SwiGLU activation in one kernel, eliminating the intermediate normalized tensor:

```
normed   = RmsNorm(x, gamma, eps)
gate, up = split(normed @ W_gate^T, normed @ W_up^T)
output   = (gate * sigmoid(gate)) * up
```

| Item | Detail |
|------|--------|
| **Inputs** | `x`, `gamma`, `W_gate`, `W_up` |

### FusedLayerNorm / FusedLayerNormBp

Standard Layer Normalization with fused mean/variance computation and affine transform:

```
output = (x - mean) / sqrt(var + eps) * gamma + beta
```

| Item | Detail |
|------|--------|
| **Inputs** | `x`, `gamma`, `beta` |
| **Float args** | `eps` |

---

## 5. Activation Ops

### SiLU / SiLUBp

Sigmoid Linear Unit activation:

```
SiLU(x) = x * sigmoid(x) = x / (1 + exp(-x))
```

| Item | Detail |
|------|--------|
| **Inputs** | `x` |
| **Notes** | Identical to Swish with `beta = 1`. Used throughout LLaMA, Mistral, and Falcon |

### SiluAndMul

Fused SiLU gate + element-wise multiply implementing the SwiGLU nonlinearity:

```
SiluAndMul(gate, up) = SiLU(gate) * up
```

| Item | Detail |
|------|--------|
| **Inputs** | `gate`, `up` — both shape `[..., hidden_size]` |
| **Notes** | Equivalent to `silu(gate) * up`; fused to avoid materializing the intermediate SiLU output |

### GeluAndMul

Fused GELU gate + element-wise multiply implementing the GeGLU nonlinearity:

```
GeluAndMul(gate, up) = GELU(gate) * up
```

| Item | Detail |
|------|--------|
| **Inputs** | `gate`, `up` |
| **Bool args** | `approximate` — use `tanh` approximation (as in GPT-2) or the exact erf formulation |

### FusedGELU / FusedGELUBp

Standalone GELU activation with optional fast-approximate mode:

```
GELU(x) ≈ 0.5 * x * (1 + tanh(sqrt(2/π) * (x + 0.044715 * x³)))   [approximate]
GELU(x) = x * Φ(x)                                                   [exact]
```

| Item | Detail |
|------|--------|
| **Bool args** | `approximate` |

### SwishMul / SwishMulBp

Swish-gated multiplication with a learnable or fixed beta:

```
SwishMul(x, beta) = x * sigmoid(beta * x)
```

| Item | Detail |
|------|--------|
| **Inputs** | `x`, optional scalar `beta` (defaults to 1.0, recovering SiLU) |

### SquaredReLU

Squared ReLU activation used in Primer and compatible with Triton-optimized kernels:

```
SquaredReLU(x) = max(0, x)²
```

| Item | Detail |
|------|--------|
| **Inputs** | `x` |
| **Notes** | Provides faster convergence than ReLU in some transformer MLP configurations |

---

## 6. Positional Encoding Ops

### RoPE / RoPEBp

Rotary Position Embedding (Su et al., 2021). Applies a rotation to query and key vectors based on their absolute sequence position, enabling relative position information to emerge from the dot product:

```
q_rot[i], q_rot[i+1] = q[i] * cos(θ) - q[i+1] * sin(θ),
                        q[i] * sin(θ) + q[i+1] * cos(θ)
```

| Item | Detail |
|------|--------|
| **Inputs** | `Q`, `K`, `positions` `[seq_len]` |
| **Float args** | `base` — frequency base, typically 10000 or 500000 |
| **Int args** | `head_dim`, optional `rope_scaling_type` |

### FusedRoPE / FusedRoPEBp

Single-kernel RoPE that applies precomputed `cos` and `sin` tables without constructing explicit rotation matrices. Reduces memory and compute versus the reference RoPE.

| Item | Detail |
|------|--------|
| **Inputs** | `Q`, `K`, `cos_table` `[max_seq, head_dim/2]`, `sin_table` `[max_seq, head_dim/2]` |

### FusedMRoPE

Multi-modal RoPE for Vision-Language Models. Applies separate rotation frequencies to different portions of the head dimension, allowing the model to encode spatial (image) and temporal (text) positions with different frequency schedules.

| Item | Detail |
|------|--------|
| **Inputs** | `Q`, `K`, `cos_tables` `[num_modalities, max_seq, head_dim/2]`, `sin_tables` same, `modality_ids` `[seq]` |

### DualRoPE

Applies two separate RoPE encodings to different head groups. Used in models with heterogeneous position representations between content heads and positional heads.

| Item | Detail |
|------|--------|
| **Inputs** | `Q`, `K`, `cos1`, `sin1`, `cos2`, `sin2`, head split index |

### ApplyAlibi

Attention with Linear Biases (ALiBi, Press et al., 2021). Adds a position-dependent linear bias to attention logits rather than injecting positional information into the embeddings:

```
Attention(Q, K, V) = softmax(QK^T / sqrt(d) + m * [-0, -1, -2, ...]) V
```

where `m` is a head-specific slope.

| Item | Detail |
|------|--------|
| **Inputs** | attention logits `[batch, heads, seq, seq]`, head slopes `[heads]` |
| **Notes** | Enables zero-shot length generalization beyond training context |

### RelativePositionBias

T5-style relative position bias. Bins relative distances into a learned bias lookup table:

```
logits += bias_table[clip(relative_position, -max_dist, max_dist)]
```

| Item | Detail |
|------|--------|
| **Inputs** | `Q`, `K`, learnable `bias_table` `[num_heads, 2*max_distance+1]` |
| **Int args** | `max_distance`, `num_buckets` |

### PerLayerEmbedding

Layer-dependent position frequency scaling. Adjusts RoPE frequencies per transformer layer, allowing early layers to capture local patterns and later layers to capture long-range dependencies with different effective context windows.

| Item | Detail |
|------|--------|
| **Inputs** | `Q`, `K`, base `cos`/`sin` tables, per-layer scaling factors `[num_layers]`, current `layer_idx` |

---

## 7. Quantization and Sampling Ops

### AwqMatmul

Activation-aware Weight Quantization (AWQ) fused matmul. Applies per-channel weight dequantization inline with the matrix multiplication to avoid storing a full-precision weight tensor:

```
output = (input * input_scale) @ dequant(W_int4, scale, zero)^T
```

| Item | Detail |
|------|--------|
| **Inputs** | `input`, `W_int4` (packed 4-bit weights), `scale` `[out_features, groups]`, `zero_point` same |
| **Int args** | `group_size` — number of columns sharing a scale/zero pair (typically 64 or 128) |

### Fp8Quantize / Fp8Dequantize / Fp8Matmul

FP8 support in E4M3 and E5M2 formats.

- **Fp8Quantize**: Converts FP16/BF16 tensors to FP8 with per-tensor or per-channel scaling.
- **Fp8Dequantize**: Restores FP16/BF16 from FP8 representation.
- **Fp8Matmul**: Performs matrix multiplication in FP8 arithmetic with accumulation in FP16 or FP32.

| Item | Detail |
|------|--------|
| **Int args (all three)** | `fp8_type` — 0 for E4M3, 1 for E5M2 |
| **Float args** | `scale` (quantize/dequantize), `amax` — for dynamic scaling |

### QuantizedMatmul

General INT4/INT8 matrix multiplication with scale and zero-point dequantization:

```
output = (A @ dequant(B, scale, zero_point)^T)
dequant(x, s, z) = (x - z) * s
```

| Item | Detail |
|------|--------|
| **Inputs** | `A` (activations, full precision), `B_quant` (packed integers), `scale`, `zero_point` |
| **Int args** | `bits` — 4 or 8; `group_size` |

### GGMLDequantize

Block-quantized dequantization for GGML/llama.cpp weight formats. Supports Q4_0, Q4_1, Q5_0, Q5_1, Q8_0, and Q8_1 block quantization schemes.

| Item | Detail |
|------|--------|
| **Inputs** | `data` (raw block-quantized bytes), `type_id` (integer encoding the GGML quant type) |
| **Outputs** | dequantized FP16 or FP32 tensor |

### SmoothQuant

Per-channel activation scaling for smooth INT8 quantization (Xiao et al., 2022). Migrates quantization difficulty from activations to weights by scaling channels:

```
Y = (X / s) @ (W * s)^T
```

| Item | Detail |
|------|--------|
| **Inputs** | `X`, per-channel `s` (migration scale), `W` |
| **Notes** | `s` is typically computed offline from calibration data |

### GpuTopKSample

GPU-accelerated top-k sampling. Selects from the `k` highest-probability tokens after softmax, then samples from the resulting categorical distribution.

| Item | Detail |
|------|--------|
| **Inputs** | `logits` `[batch, vocab_size]` |
| **Int args** | `k` |
| **Float args** | `temperature` |
| **Outputs** | sampled token ids `[batch]` |

### GpuTopPSample

GPU-accelerated nucleus (top-p) sampling. Samples from the smallest set of tokens whose cumulative probability exceeds `p`.

| Item | Detail |
|------|--------|
| **Inputs** | `logits` `[batch, vocab_size]` |
| **Float args** | `p`, `temperature` |

### TokenSample

Combined sampling with temperature rescaling, repetition penalty, and top-k/top-p in a single operation.

| Item | Detail |
|------|--------|
| **Inputs** | `logits`, `input_ids` (for repetition penalty), optional presence mask |
| **Float args** | `temperature`, `top_p`, `repetition_penalty` |
| **Int args** | `top_k` |

### SamplingPenalties

Standalone repetition and frequency penalties applied to logits prior to sampling. Allows the penalty computation to be separated from sampling for caching or multi-step adjustments.

| Item | Detail |
|------|--------|
| **Inputs** | `logits`, `input_ids`, `frequency_counts` (per-token occurrence counts in generated text) |
| **Float args** | `repetition_penalty`, `frequency_penalty`, `presence_penalty` |

---

## 8. SSM / Recurrent Ops

State Space Models and recurrent operations for sequence modeling without full attention.

### SelectiveScan

The core Mamba SSM (Gu and Dao, 2023) selective scan operation. Implements the discretized linear recurrence:

```
h_t = A_bar * h_{t-1} + B_bar * x_t
y_t = C * h_t + D * x_t
```

where `A_bar` and `B_bar` are input-dependent (selective) discretizations of the continuous-time SSM parameters.

| Item | Detail |
|------|--------|
| **Inputs** | `u` (input sequence), `delta`, `A`, `B`, `C`, optional `D`, optional `z` (gate) |
| **Bool args** | `delta_softplus` — apply softplus to delta before discretization |
| **Outputs** | `y` (output sequence), optionally the final hidden state |

### Mamba2SSM

Mamba-2 structured state space duality (Dao and Gu, 2024). Extends Mamba with multi-head SSM structure and SSD (Structured State Space Duality) formulation for improved parallelism.

| Item | Detail |
|------|--------|
| **Inputs** | `X`, `dt`, `A`, `B`, `C`, chunk boundary info |
| **Int args** | `chunk_size`, `num_heads` |

### GatedDeltaNetBlock / GatedDeltaRule

Gated Delta Networks (Yang et al., 2024). Implements a delta rule update with gating:

```
h_t = (1 - beta_t * k_t^T) * h_{t-1} + beta_t * v_t * k_t^T
y_t = gate_t * (h_t * q_t)
```

| Item | Detail |
|------|--------|
| **GatedDeltaNetBlock inputs** | full sequence `Q`, `K`, `V`, `gate`, `beta` |
| **GatedDeltaRule inputs** | single-step `q_t`, `k_t`, `v_t`, `gate_t`, `beta_t`, recurrent state `h` |

### LinearAttentionDecode

Efficient single-step recurrent inference for linear attention models. Updates the linear attention state and computes the output for one new token without materializing the full attention matrix.

| Item | Detail |
|------|--------|
| **Inputs** | `q_t`, `k_t`, `v_t`, recurrent state `S` `[batch, heads, head_dim, head_dim]` |
| **Outputs** | output `y_t`, updated state `S'` |

### EmaUpdate / EmaUpdateBp

Exponential Moving Average state update for recurrent models (e.g., MEGA):

```
h_t = alpha * h_{t-1} + (1 - alpha) * x_t
```

| Item | Detail |
|------|--------|
| **Inputs** | `x_t`, previous state `h_{t-1}`, decay coefficients `alpha` (may be per-channel) |
| **Backward** | Computes gradients through the recurrence for truncated BPTT |

---

## 9. Mixture of Experts

### MixtureOfExperts

Sparse MoE feed-forward computation. Routes each token to a fixed number of expert networks (`top_k`), computes expert outputs, then combines them with router probabilities as weights:

```
output_i = sum_{j in topk(i)} router_prob(i,j) * Expert_j(token_i)
```

| Item | Detail |
|------|--------|
| **Inputs** | `hidden_states` `[tokens, hidden]`, expert weight tensors (stacked), `router_logits` `[tokens, num_experts]` |
| **Int args** | `num_experts`, `top_k`, `expert_capacity` (optional token-dropping threshold) |
| **Float args** | `expert_scale` — multiplier applied to routed outputs |
| **Notes** | Includes auxiliary load-balancing loss computation when `compute_balance_loss == true` |

### MoeGate

Top-k expert selection gate. Computes softmax over router logits, selects top-k expert indices, and returns normalized routing weights.

```
scores     = softmax(hidden @ W_gate^T)
indices    = argtopk(scores, k)
weights    = scores[indices] / sum(scores[indices])   [normalized]
```

| Item | Detail |
|------|--------|
| **Inputs** | `hidden` `[tokens, hidden_size]`, gate weight `W_gate` `[num_experts, hidden_size]` |
| **Int args** | `top_k` |
| **Bool args** | `normalize_topk_scores` |
| **Outputs** | `weights` `[tokens, top_k]`, `expert_indices` `[tokens, top_k]` |

---

## 10. Audio and Signal Processing

Operations in the `NDAudio` and `NDSignal` namespaces provide native-speed audio feature extraction and signal processing.

### Mel Spectrogram

Computes the Mel-frequency spectrogram from a raw waveform. Applies a Short-Time Fourier Transform (STFT), converts power to the Mel scale, and optionally applies logarithmic compression.

| Item | Detail |
|------|--------|
| **Inputs** | `waveform` `[batch, samples]`, optional pre-emphasis coefficient |
| **Int args** | `n_fft`, `hop_length`, `win_length`, `n_mels`, `sample_rate` |
| **Float args** | `fmin`, `fmax` — frequency bounds for the Mel filter bank |
| **Bool args** | `log_scale` |

### MFCC (Mel-Frequency Cepstral Coefficients)

Computes MFCCs from a waveform by applying the Discrete Cosine Transform (DCT) to log Mel spectrogram features.

| Item | Detail |
|------|--------|
| **Inputs** | `waveform` `[batch, samples]` |
| **Int args** | `n_mfcc`, `n_mels`, `n_fft`, `hop_length`, `sample_rate` |
| **Outputs** | `mfcc` `[batch, n_mfcc, frames]` |

### Pitch Detection

Estimates fundamental frequency (F0) from an audio waveform using an autocorrelation-based algorithm.

| Item | Detail |
|------|--------|
| **Inputs** | `waveform` `[batch, samples]` |
| **Int args** | `sample_rate`, `hop_length` |
| **Float args** | `fmin`, `fmax` |
| **Outputs** | `f0` `[batch, frames]`, `voiced_flag` `[batch, frames]` |

### Griffin-Lim Reconstruction

Iterative phase reconstruction from a magnitude spectrogram using the Griffin-Lim algorithm.

| Item | Detail |
|------|--------|
| **Inputs** | `magnitude` `[batch, n_fft//2+1, frames]` |
| **Int args** | `n_fft`, `hop_length`, `win_length`, `n_iter` |
| **Outputs** | reconstructed `waveform` `[batch, samples]` |

### DFT (Discrete Fourier Transform)

One-dimensional Discrete Fourier Transform and its inverse.

| Item | Detail |
|------|--------|
| **Inputs** | real or complex input `[..., N]` |
| **Bool args** | `inverse` — compute IDFT when true |
| **Outputs** | complex output `[..., N]` (split into real/imaginary channels) |

### STFT (Short-Time Fourier Transform)

Computes the STFT of a signal using an analysis window.

| Item | Detail |
|------|--------|
| **Inputs** | `signal` `[batch, samples]`, optional pre-computed `window` `[win_length]` |
| **Int args** | `n_fft`, `hop_length`, `win_length` |
| **Bool args** | `center` — pad signal to center frames; `onesided` — return only positive frequencies |
| **Outputs** | complex spectrogram `[batch, n_fft//2+1, frames, 2]` (real, imag) |

### Window Functions

Native implementations of common analysis windows:

| Op | Formula |
|----|---------|
| **HannWindow** | `w[n] = 0.5 * (1 - cos(2π*n/(N-1)))` |
| **HammingWindow** | `w[n] = 0.54 - 0.46 * cos(2π*n/(N-1))` |
| **BlackmanWindow** | `w[n] = 0.42 - 0.5*cos(2πn/N) + 0.08*cos(4πn/N)` |

All three accept a single int arg `window_length` and an optional `periodic` bool (matches PyTorch's `periodic=True` convention for STFT use).

---

## 11. Usage Examples

The examples below show how to invoke selected new ops through the SameDiff and Nd4j APIs.

### Flash Attention in SameDiff

```java
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.autodiff.samediff.SDVariable;
import org.nd4j.linalg.factory.Nd4j;

SameDiff sd = SameDiff.create();

// Q, K, V: [batch=2, heads=8, seq=512, head_dim=64]
SDVariable Q = sd.var("Q", Nd4j.randn(2, 8, 512, 64));
SDVariable K = sd.var("K", Nd4j.randn(2, 8, 512, 64));
SDVariable V = sd.var("V", Nd4j.randn(2, 8, 512, 64));

double scale = 1.0 / Math.sqrt(64);

// Causal FlashAttention
SDVariable out = sd.nn().flashAttention(Q, K, V, /*mask=*/null, scale, /*causal=*/true);
// out: [2, 8, 512, 64]
```

### RMSNorm

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

// x: [batch=4, hidden=2048]
INDArray x     = Nd4j.randn(4, 2048);
INDArray gamma = Nd4j.ones(2048);        // learnable scale, initialized to 1
float    eps   = 1e-6f;

INDArray normed = Nd4j.exec(new RmsNorm(x, gamma, eps))[0];
```

### LoRA MatMul

```java
// Frozen base weight W: [out=4096, in=4096]
// Adapter A: [rank=8, in=4096],  B: [out=4096, rank=8]
INDArray W   = Nd4j.randn(4096, 4096);
INDArray A   = Nd4j.randn(8,    4096).muli(0.01);
INDArray B   = Nd4j.zeros(4096, 8);

float scaling = 16.0f / 8;  // alpha / rank

INDArray input = Nd4j.randn(1, 4096);  // single token
INDArray out   = Nd4j.exec(new LoraMatMul(input, W, A, B, scaling))[0];
// out: [1, 4096]
```

### Grouped-Query Attention

```java
// LLaMA 3 style: 32 query heads, 8 KV heads
SDVariable Q  = sd.var("Q", Nd4j.randn(1, 32, 128, 128));  // 32 query heads
SDVariable K  = sd.var("K", Nd4j.randn(1,  8, 128, 128));  // 8 KV heads
SDVariable V  = sd.var("V", Nd4j.randn(1,  8, 128, 128));

SDVariable out = sd.nn().groupedQueryAttention(Q, K, V,
        /*num_heads=*/32, /*num_kv_heads=*/8, /*scale=*/1.0 / Math.sqrt(128));
```

### Selective Scan (Mamba)

```java
// Mamba SSM dimensions: batch=2, seq=256, d_model=512, d_state=16
INDArray u     = Nd4j.randn(2, 512, 256);   // [batch, d_model, seq]
INDArray delta = Nd4j.rand( 2, 512, 256);
INDArray A     = Nd4j.randn(512, 16);        // [d_model, d_state]
INDArray B     = Nd4j.randn(2, 16,  256);   // [batch, d_state, seq]
INDArray C     = Nd4j.randn(2, 16,  256);
INDArray D     = Nd4j.ones(512);

INDArray[] result = Nd4j.exec(new SelectiveScan(u, delta, A, B, C, D,
        /*delta_softplus=*/true));
INDArray y = result[0];  // [2, 512, 256]
```

### KV Cache Update and Paged Attention

```java
// Allocate a paged KV store: 1024 blocks, 16 tokens/block, 8 heads, 128 head_dim
INDArray K_cache = Nd4j.zeros(1024, 16, 8, 128);
INDArray V_cache = Nd4j.zeros(1024, 16, 8, 128);

// On each decode step:
//   slot_mapping maps new token positions to physical block slots
INDArray newK        = Nd4j.randn(1, 8, 128);   // [batch=1, heads, head_dim]
INDArray newV        = Nd4j.randn(1, 8, 128);
INDArray slotMapping = Nd4j.createFromArray(new int[]{42});  // write to slot 42

Nd4j.exec(new KvScatter(newK, newV, K_cache, V_cache, slotMapping));

// Attend over paged cache
INDArray Q         = Nd4j.randn(1, 8, 1, 128);
INDArray blockTable = Nd4j.createFromArray(new int[][]{{0, 1, 2}});  // block ids for seq
INDArray seqLens   = Nd4j.createFromArray(new int[]{43});

INDArray attnOut = Nd4j.exec(
        new PagedAttentionForward(Q, K_cache, V_cache, blockTable, seqLens,
                                  /*block_size=*/16, /*scale=*/0.088f))[0];
```

### Mel Spectrogram

```java
// 1-second audio at 16kHz
INDArray waveform = Nd4j.randn(1, 16000);

INDArray melSpec = Nd4j.exec(new MelSpectrogram(
        waveform,
        /*n_fft=*/400,
        /*hop_length=*/160,
        /*win_length=*/400,
        /*n_mels=*/80,
        /*sample_rate=*/16000,
        /*fmin=*/0.0f,
        /*fmax=*/8000.0f,
        /*log_scale=*/true))[0];
// melSpec: [1, 80, 101]
```

### Top-P Sampling with Repetition Penalty

```java
// Logits from an LLM forward pass
INDArray logits   = Nd4j.randn(1, 50257);  // [batch=1, vocab=50257]
INDArray inputIds = Nd4j.createFromArray(new int[][]{{1, 2, 3, 100, 200}});

INDArray penalized = Nd4j.exec(new SamplingPenalties(
        logits, inputIds, /*frequency_counts=*/null,
        /*repetition_penalty=*/1.1f,
        /*frequency_penalty=*/0.0f,
        /*presence_penalty=*/0.0f))[0];

INDArray tokenId = Nd4j.exec(new GpuTopPSample(
        penalized, /*p=*/0.9f, /*temperature=*/0.8f))[0];
```

---

## See Also

- [ND4J Operations Overview](operations.md) — scalar, transform, reduction, and broadcast ops
- [SameDiff Overview](../samediff/overview.md) — automatic differentiation and model building
- [Data Types Reference](data-types.md) — FP8, BF16, and quantized data type support
- [Release Notes](../../release-notes/release-notes.md) — full 1.0.0-rewrite changelog

---
title: LLM & VLM Application Stack
description: Complete guide to the samediff-llm generation pipeline, KV cache management, speculative decoding, continuous batching, tokenizers, evaluation framework, and model editing
---

# LLM & VLM Application Stack

Deeplearning4j 1.0.0-rewrite ships a full large language model (LLM) and vision-language model (VLM) application stack built on top of SameDiff. The stack is organized into six new Maven modules that together cover every layer of inference: tokenization, generation, KV cache management, speculative decoding, continuous batching, evaluation, benchmarking, model editing, audio transcription, and a web frontend for ND4J graphs.

This page gives a complete reference for all six modules, with API details and working Java code examples for the most important classes.

---

## 1. Overview and Module Map

The LLM stack sits above the existing SameDiff execution engine. SameDiff handles op dispatch and graph execution; the new modules provide the inference-specific infrastructure that production LLM serving requires.

```
Your Application
       │
       ▼
samediff-llm        ← generation pipeline, KV cache, speculative decoding,
                       continuous batching, tokenizers, evaluation, benchmarking,
                       model editing
samediff-vlm        ← vision-language model support (image+text)
samediff-audio      ← Whisper ASR support
nd4j-tokenizers     ← Rust-backed HuggingFace / SentencePiece / CLIP tokenizers
nd4j-torchscript    ← TorchScript / PyTorch model import
nd4j-web            ← TypeScript / FlatBuffers frontend for ND4J graphs
       │
       ▼
SameDiff (ND4J)     ← op execution, graph optimization, DSP plan lifecycle
       │
       ▼
libnd4j (C++)       ← CPU/CUDA kernels, BLAS, cuDNN
```

The six modules are independent of each other except that `samediff-vlm` and `samediff-audio` depend on `samediff-llm`, and all of them depend on `nd4j-tokenizers`.

---

## 2. Maven Dependencies

Add only the modules you need. All modules share the same version string.

```xml
<properties>
    <dl4j.version>1.0.0-rewrite</dl4j.version>
</properties>

<!-- Core LLM generation pipeline, KV cache, speculative decoding,
     continuous batching, evaluation, benchmarking, model editing -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>samediff-llm</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- Vision-language models (requires samediff-llm) -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>samediff-vlm</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- Whisper ASR (requires samediff-llm) -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>samediff-audio</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- Rust-backed tokenizers: HuggingFace, SentencePiece, CLIP -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-tokenizers</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- TorchScript / PyTorch model import -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-torchscript</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- TypeScript / FlatBuffers frontend for ND4J graphs -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-web</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

---

## 3. Generation Pipeline

The generation pipeline is the unified entry point for all text generation tasks. It handles model I/O auto-discovery, embedding extraction, tokenization, decode loop construction, and configuration-driven optimization.

### Core Classes

| Class | Role |
|-------|------|
| `GenerationPipeline` | Top-level entry point; owns the decode loop and lifecycle |
| `GenerationPipelineConfig` | Builder-style configuration for the pipeline |
| `DecodeOptions` | Per-call generation parameters (temperature, top-k, etc.) |
| `GenerationResult` | Output: token IDs, decoded text, timing data |
| `TextGenerator` | Higher-level API with streaming callback support |
| `Sampler` / `GreedySampler` / `CompositeSampler` | Sampling strategy hierarchy |
| `SamplingConfig` / `SamplerUtils` | Temperature / top-k / top-p config and utilities |
| `DecoderInputBuilder` / `DecoderUtils` | Tensor construction for each decode step |
| `DecodeStepDiagnostics` | Per-step diagnostics: token IDs, logit stats, timing |

### Building a Pipeline

`GenerationPipelineConfig` uses a fluent builder. All fields are optional; the pipeline performs auto-discovery for any field not set.

```java
import org.deeplearning4j.llm.generation.GenerationPipeline;
import org.deeplearning4j.llm.generation.GenerationPipelineConfig;
import org.deeplearning4j.llm.generation.DecodeOptions;
import org.deeplearning4j.llm.generation.GenerationResult;
import org.deeplearning4j.llm.generation.SamplingConfig;
import org.deeplearning4j.llm.kv.KvCacheStrategy;
import org.nd4j.autodiff.samediff.SameDiff;

// Load or import your model
SameDiff model = SameDiff.load(new File("llama-3.1-8b.fb"), true);

// Configure the pipeline
GenerationPipelineConfig config = GenerationPipelineConfig.builder()
    .decoder(model)                           // SameDiff graph containing the decoder
    .tokenizer("tokenizer.json")              // path or classpath resource
    .maxTokens(2048)                          // maximum context length
    .samplingConfig(SamplingConfig.builder()
        .temperature(0.7f)
        .topK(50)
        .topP(0.9f)
        .repetitionPenalty(1.1f)
        .build())
    .kvCacheStrategy(KvCacheStrategy.PAGED)   // see KV Cache section
    .build();

GenerationPipeline pipeline = new GenerationPipeline(config);
```

### Running Generation

Pass per-call options via `DecodeOptions`. Settings in `DecodeOptions` override the pipeline-level `SamplingConfig` for that call only.

```java
DecodeOptions opts = DecodeOptions.builder()
    .temperature(0.8f)
    .topK(40)
    .maxNewTokens(512)
    .build();

GenerationResult result = pipeline.generate("Explain quantum entanglement.", opts);

System.out.println(result.getText());         // decoded string
System.out.println(result.getTokenIds());     // List<Integer> of generated token IDs
System.out.printf("%.1f tok/s%n",
    result.getTokensPerSecond());             // throughput from timing data
```

### Streaming with TextGenerator

`TextGenerator` wraps `GenerationPipeline` and adds token-by-token streaming via a callback.

```java
import org.deeplearning4j.llm.generation.TextGenerator;

TextGenerator generator = new TextGenerator(pipeline);

generator.generate("Write a haiku about neural networks.", opts,
    token -> System.out.print(token));        // callback receives each decoded token

System.out.println();  // newline after streaming completes
```

### Sampling Strategies

`GreedySampler` always picks the highest-probability token. `CompositeSampler` chains a sequence of sampling transforms — temperature scaling, then top-k filtering, then top-p nucleus filtering — before the final argmax or categorical sample.

```java
import org.deeplearning4j.llm.generation.sampling.GreedySampler;
import org.deeplearning4j.llm.generation.sampling.CompositeSampler;
import org.deeplearning4j.llm.generation.sampling.SamplerUtils;

// Greedy decoding — deterministic, fast
Sampler greedy = new GreedySampler();

// Nucleus sampling: temperature → top-k → top-p → sample
Sampler nucleus = CompositeSampler.builder()
    .temperature(0.9f)
    .topK(100)
    .topP(0.95f)
    .build();
```

### Per-Step Diagnostics

Enable `DecodeStepDiagnostics` to capture detailed information about each decode step. This is useful for debugging generation quality issues.

```java
import org.deeplearning4j.llm.generation.DecodeStepDiagnostics;

pipeline.enableDiagnostics(true);

GenerationResult result = pipeline.generate("Hello, world!", opts);

for (DecodeStepDiagnostics step : result.getDiagnostics()) {
    System.out.printf("Step %d: token=%d logit_max=%.3f logit_entropy=%.3f time_ms=%d%n",
        step.getStep(),
        step.getSelectedTokenId(),
        step.getMaxLogit(),
        step.getLogitEntropy(),
        step.getStepTimeMs());
}
```

---

## 4. KV Cache Management

The key-value (KV) cache stores the attention keys and values computed during the prefill and previous decode steps. Good cache management is the single largest lever for improving LLM serving throughput. The LLM stack provides a comprehensive hierarchy of cache implementations.

### Cache Strategy Overview

| Strategy | Class | When to Use |
|----------|-------|-------------|
| Paged | `PagedKVCache` | Default; best memory utilization |
| Paged + eviction | `EvictablePagedKVCache` | Long conversations; evict old pages |
| Per-layer policy | `PerLayerPagedKVCache` | Different eviction per transformer layer |
| Quantized | `QuantizedPagedKVCache` | Memory-constrained GPUs; INT8/FP16 pages |
| MLA | `MLAKVCache` | DeepSeek Multi-head Latent Attention |
| Beam search | `BeamKVCacheManager` | Beam decoding with K beams |
| Speculative | `SpeculativeKVCacheManager` | Speculative decoding draft/verify |
| Tiered | `TieredKVCacheManager` | GPU → host DRAM → disk tiering |
| Unified | `UnifiedKvCacheManager` | Single manager across all strategies |

### PagedKVCache

`PagedKVCache` partitions the cache into fixed-size pages. Sequences are allocated pages on demand; eviction is O(1) — just reclaim a page. This is the vLLM-style approach and is the default strategy.

```java
import org.deeplearning4j.llm.kv.PagedKVCache;

PagedKVCache cache = PagedKVCache.builder()
    .pageSize(16)           // tokens per page
    .maxPages(1024)         // total pages (controls max memory)
    .numLayers(32)          // transformer layer count
    .numHeads(8)            // KV heads (use GQA count, not full head count)
    .headDim(128)           // per-head dimension
    .dtype(DataType.FLOAT16)
    .build();
```

### Eviction Policies

`EvictablePagedKVCache` adds eviction support. Three built-in eviction policies are provided:

| Policy | Class | Description |
|--------|-------|-------------|
| LRU | default | Evict least recently used page |
| H2O | `H2OEvictionPolicy` | Heavy Hitter Oracle: evict low-importance tokens based on accumulated attention scores |
| StreamingLLM | `StreamingLLMEvictionPolicy` | Preserve attention sink tokens + recent sliding window |

```java
import org.deeplearning4j.llm.kv.EvictablePagedKVCache;
import org.deeplearning4j.llm.kv.eviction.H2OEvictionPolicy;
import org.deeplearning4j.llm.kv.eviction.StreamingLLMEvictionPolicy;
import org.deeplearning4j.llm.kv.eviction.AttentionSinkDetector;

// H2O eviction — works well for long document summarization
EvictablePagedKVCache h2oCache = EvictablePagedKVCache.builder()
    .pageSize(16)
    .maxPages(512)
    .numLayers(32)
    .numHeads(8)
    .headDim(128)
    .evictionPolicy(new H2OEvictionPolicy())
    .build();

// StreamingLLM — preserves attention sinks, keeps recent window
AttentionSinkDetector sinkDetector = new AttentionSinkDetector(numSinkTokens: 4);

EvictablePagedKVCache streamingCache = EvictablePagedKVCache.builder()
    .pageSize(16)
    .maxPages(512)
    .numLayers(32)
    .numHeads(8)
    .headDim(128)
    .evictionPolicy(new StreamingLLMEvictionPolicy(sinkDetector, windowSize: 256))
    .build();
```

### Per-Layer Eviction

`PerLayerPagedKVCache` assigns a different `PerLayerKVPolicy` to each transformer layer. This is useful because attention patterns differ significantly between early and late layers.

```java
import org.deeplearning4j.llm.kv.PerLayerPagedKVCache;
import org.deeplearning4j.llm.kv.PerLayerKVPolicy;
import org.deeplearning4j.llm.kv.eviction.StreamingLLMEvictionPolicy;
import org.deeplearning4j.llm.kv.eviction.H2OEvictionPolicy;

List<PerLayerKVPolicy> policies = new ArrayList<>();
for (int layer = 0; layer < 32; layer++) {
    if (layer < 4) {
        // Early layers: protect attention sinks with StreamingLLM
        policies.add(PerLayerKVPolicy.of(new StreamingLLMEvictionPolicy(sinkDetector, 512)));
    } else {
        // Later layers: H2O importance-based eviction
        policies.add(PerLayerKVPolicy.of(new H2OEvictionPolicy()));
    }
}

PerLayerPagedKVCache perLayerCache = PerLayerPagedKVCache.builder()
    .pageSize(16)
    .maxPages(512)
    .perLayerPolicies(policies)
    .numHeads(8)
    .headDim(128)
    .build();
```

### Quantized KV Cache

`QuantizedPagedKVCache` stores pages in INT8 or FP16 and dequantizes on read. This roughly halves or quarters the memory footprint of the cache with minimal accuracy impact on most models.

```java
import org.deeplearning4j.llm.kv.QuantizedPagedKVCache;
import org.deeplearning4j.llm.kv.QuantizationMode;

QuantizedPagedKVCache quantCache = QuantizedPagedKVCache.builder()
    .pageSize(16)
    .maxPages(2048)          // 2x more pages for same memory vs FP32
    .numLayers(32)
    .numHeads(8)
    .headDim(128)
    .quantizationMode(QuantizationMode.INT8)  // or FP16
    .build();
```

### KV Cache Offloading

For very long contexts, the cache can be offloaded from GPU VRAM to host DRAM or disk.

```java
import org.deeplearning4j.llm.kv.offload.KVCacheHostOffloader;
import org.deeplearning4j.llm.kv.offload.KVCacheDiskOffloader;

// Offload evicted pages to host DRAM (PCIe transfer on demand)
KVCacheHostOffloader hostOffloader = KVCacheHostOffloader.builder()
    .maxHostBytes(8L * 1024 * 1024 * 1024)   // 8 GB host RAM
    .asyncTransfer(true)
    .build();

// Offload evicted pages to disk (NVMe SSD recommended)
KVCacheDiskOffloader diskOffloader = KVCacheDiskOffloader.builder()
    .storagePath(Path.of("/tmp/kvcache"))
    .maxDiskBytes(64L * 1024 * 1024 * 1024)  // 64 GB
    .build();
```

Use `TieredKVCacheManager` to combine GPU, host, and disk tiers automatically:

```java
import org.deeplearning4j.llm.kv.TieredKVCacheManager;

TieredKVCacheManager tieredManager = TieredKVCacheManager.builder()
    .gpuCache(cache)
    .hostOffloader(hostOffloader)
    .diskOffloader(diskOffloader)
    .build();
```

### Prefix Sharing

`KVCachePrefixTree` and `RadixPrefixCache` enable sharing KV cache pages across requests that share a common prompt prefix (e.g., a system prompt). Matching prefixes are detected and their cached pages are reused rather than recomputed.

```java
import org.deeplearning4j.llm.kv.prefix.RadixPrefixCache;

RadixPrefixCache prefixCache = RadixPrefixCache.builder()
    .pageSize(16)
    .maxEntries(10000)
    .build();

// The pipeline will check prefixCache before computing prefill
pipeline.setPrefixCache(prefixCache);
```

### KV Cache Checkpointing

Save and restore cache state to disk, enabling pause-and-resume of long generation sessions.

```java
import org.deeplearning4j.llm.kv.checkpoint.KVCacheCheckpointManager;

KVCacheCheckpointManager checkpointMgr = new KVCacheCheckpointManager(cache);

// Save
checkpointMgr.checkpoint(Path.of("/tmp/kvcache_checkpoint.bin"));

// Restore
checkpointMgr.restore(Path.of("/tmp/kvcache_checkpoint.bin"));
```

---

## 5. Speculative Decoding

Speculative decoding uses a fast draft model (or an n-gram heuristic) to propose multiple tokens ahead, then verifies them in a single forward pass of the full target model. Accepted tokens come for free; only rejected tokens require additional passes. On hardware where the target model is memory-bandwidth-bound, speculative decoding commonly delivers 2-3x throughput improvement.

### Speculator Implementations

| Class | Draft Source | Notes |
|-------|-------------|-------|
| `NgramSpeculator` | N-gram from generated context | No secondary model required |
| `DraftModelSpeculator` | Smaller SameDiff model | Highest acceptance rate |

### NgramSpeculator

Uses an n-gram index built from the tokens already generated in the current sequence. No additional model weights are required.

```java
import org.deeplearning4j.llm.speculative.NgramSpeculator;
import org.deeplearning4j.llm.speculative.SpeculativeDecodeLoop;

NgramSpeculator speculator = NgramSpeculator.builder()
    .ngramOrder(4)          // use 4-gram drafts
    .draftLength(5)         // propose up to 5 tokens per step
    .build();

SpeculativeDecodeLoop loop = SpeculativeDecodeLoop.builder()
    .targetModel(model)
    .speculator(speculator)
    .kvCache(cache)
    .build();

GenerationResult result = loop.generate("Once upon a time", opts);
System.out.printf("Accepted %.1f%% of draft tokens%n",
    result.getSpeculativeAcceptanceRate() * 100);
```

### DraftModelSpeculator

Uses a smaller, faster model to generate draft tokens. The draft model should share the same vocabulary as the target model.

```java
import org.deeplearning4j.llm.speculative.DraftModelSpeculator;

SameDiff draftModel = SameDiff.load(new File("llama-3.1-1b.fb"), true);

DraftModelSpeculator speculator = DraftModelSpeculator.builder()
    .draftModel(draftModel)
    .draftLength(7)            // draft up to 7 tokens
    .draftKvCache(draftCache)  // separate smaller cache for the draft model
    .build();

SpeculativeDecodeLoop loop = SpeculativeDecodeLoop.builder()
    .targetModel(model)
    .speculator(speculator)
    .kvCache(cache)
    .verifier(new TreeAttentionVerifier())   // parallel tree-based verification
    .build();
```

### Tree Attention Verification

`TreeAttentionVerifier` organizes draft tokens into a tree structure and verifies all candidates in parallel with a single batched forward pass of the target model. This maximizes GPU utilization during the verification step.

The tree verifier is selected automatically when `draftLength > 1` and is the recommended choice for `DraftModelSpeculator`. It requires no additional configuration beyond being set on the `SpeculativeDecodeLoop`.

---

## 6. Continuous Batching

Continuous batching (sometimes called in-flight batching) keeps the GPU fully saturated by interleaving prefill and decode steps across multiple requests. Unlike static batching, where a batch waits until all sequences in it complete, continuous batching allows new requests to be admitted and completed sequences to exit at any decode step.

### Architecture

```
Incoming requests
       │
       ▼
ContinuousBatchScheduler
  │  assigns requests to batch slots
  │  manages per-request BatchGenerationState
  │
  ├──► ChunkedPrefillEngine    ← breaks long prompts into chunks
  │       processed in decode steps alongside ongoing sequences
  │
  └──► Decode step (full batch)
          │
          ▼
       BatchCompactor
          removes completed sequences, compacts active slots
```

### ContinuousBatchScheduler

```java
import org.deeplearning4j.llm.batch.ContinuousBatchScheduler;
import org.deeplearning4j.llm.batch.ChunkedPrefillEngine;

ChunkedPrefillEngine prefillEngine = ChunkedPrefillEngine.builder()
    .chunkSize(512)           // tokens of prefill to process per step
    .build();

ContinuousBatchScheduler scheduler = ContinuousBatchScheduler.builder()
    .maxBatchSize(32)         // max concurrent sequences
    .maxSeqLen(4096)
    .model(model)
    .kvCache(cache)
    .prefillEngine(prefillEngine)
    .build();

scheduler.start();

// Submit requests (thread-safe; can be called from multiple threads)
CompletableFuture<GenerationResult> future1 =
    scheduler.submit("Summarize: " + longDocument, opts);
CompletableFuture<GenerationResult> future2 =
    scheduler.submit("Translate to French: Hello world", opts);

GenerationResult r1 = future1.get();
GenerationResult r2 = future2.get();

scheduler.shutdown();
```

### BatchGenerationState

`BatchGenerationState` tracks per-sequence state within the batch: current token position, KV cache page assignments, sampling state, and completion status. It is managed automatically by `ContinuousBatchScheduler` and is not normally accessed directly.

### BatchCompactor

`BatchCompactor` runs at the end of each decode step to remove completed sequences and compact the batch tensor so that the GPU kernel always operates on a dense, full-occupancy batch. It is attached to the scheduler automatically.

---

## 7. Tokenizers

The `nd4j-tokenizers` module provides tokenizers backed by Rust-native implementations for correctness and performance. All tokenizers implement the `Tokenizer` interface.

### Tokenizer Interface

```java
import org.nd4j.tokenizers.Tokenizer;
import org.nd4j.tokenizers.Encoding;

public interface Tokenizer {
    Encoding encode(String text);
    String decode(List<Integer> ids);
    Map<String, Integer> specialTokens();
    int vocabSize();
    void close();
}
```

`Encoding` holds the token IDs, attention mask, and (optionally) token type IDs.

### HuggingFaceTokenizer

Loads any tokenizer in the standard `tokenizer.json` format as exported by Hugging Face `transformers`. Supports BPE, WordPiece, and Unigram models.

```java
import org.nd4j.tokenizers.HuggingFaceTokenizer;

HuggingFaceTokenizer tokenizer =
    HuggingFaceTokenizer.fromFile(Path.of("path/to/tokenizer.json"));

Encoding enc = tokenizer.encode("The quick brown fox");
System.out.println(enc.getIds());           // [791, 4996, 14198, 39935]

String decoded = tokenizer.decode(enc.getIds());
System.out.println(decoded);                // "The quick brown fox"

tokenizer.close();
```

### SentencePieceTokenizer

Loads SentencePiece BPE models (`.model` files), used by LLaMA, Gemma, Mistral, and other models that do not use the HuggingFace format.

```java
import org.nd4j.tokenizers.SentencePieceTokenizer;

SentencePieceTokenizer tokenizer =
    SentencePieceTokenizer.fromFile(Path.of("tokenizer.model"));

Encoding enc = tokenizer.encode("Hello, SentencePiece!");
tokenizer.close();
```

### CLIPTokenizer

A specialized tokenizer for CLIP-family vision-language models, following the byte-pair encoding used by the original OpenAI CLIP implementation.

```java
import org.nd4j.tokenizers.CLIPTokenizer;

CLIPTokenizer tokenizer =
    CLIPTokenizer.fromFiles(
        Path.of("vocab.json"),
        Path.of("merges.txt"));

// Encode a text prompt for CLIP image-text alignment
Encoding enc = tokenizer.encode("a photo of a cat");
tokenizer.close();
```

### Chat Templates

`ChatTemplate` renders structured chat conversations into the prompt format expected by an instruction-tuned model. It implements a Jinja2-subset template engine compatible with the `chat_template` field in HuggingFace `tokenizer_config.json`.

```java
import org.nd4j.tokenizers.ChatTemplate;
import org.nd4j.tokenizers.ChatMessage;

ChatTemplate template = ChatTemplate.fromTokenizerConfig(
    Path.of("tokenizer_config.json"));

List<ChatMessage> messages = List.of(
    ChatMessage.system("You are a helpful assistant."),
    ChatMessage.user("What is the capital of France?"),
    ChatMessage.assistant("The capital of France is Paris."),
    ChatMessage.user("What is its population?"));

String prompt = template.apply(messages, addGenerationPrompt: true);
System.out.println(prompt);
// Produces the model-specific formatted prompt string
```

### TokenizerFactory

`TokenizerFactory` auto-detects the tokenizer type from the files present in a directory and instantiates the correct implementation.

```java
import org.nd4j.tokenizers.TokenizerFactory;

// Auto-detect from a directory containing tokenizer.json or tokenizer.model
Tokenizer tokenizer = TokenizerFactory.fromDirectory(Path.of("model-dir/"));
```

---

## 8. Evaluation Framework

The evaluation framework provides automated benchmarking of LLM capabilities across standard academic benchmarks and custom datasets.

### Core Evaluation Classes

| Class | Role |
|-------|------|
| `EvalRunner` | Orchestrates evaluation runs; parallelizes across dataset examples |
| `EvalConfig` | Dataset, benchmark, metric, and generation options |
| `EvalResult` | Aggregated result: per-benchmark scores, timing, sample results |
| `SampleResult` | Per-example output, prediction, and score |
| `PerplexityEvaluator` | Computes log-perplexity over a reference corpus |
| `GenerationQualityValidator` | Validates generation coherence (length, repetition, entropy) |
| `AnswerExtractor` | Extracts structured answers from free-form generated text |

### Running a Standard Benchmark

```java
import org.deeplearning4j.llm.eval.EvalRunner;
import org.deeplearning4j.llm.eval.EvalConfig;
import org.deeplearning4j.llm.eval.EvalResult;
import org.deeplearning4j.llm.eval.benchmarks.MMLUBenchmark;
import org.deeplearning4j.llm.eval.datasets.HuggingFaceDataset;

HuggingFaceDataset dataset = HuggingFaceDataset.load("cais/mmlu", split: "test");

EvalConfig config = EvalConfig.builder()
    .benchmark(new MMLUBenchmark())
    .dataset(dataset)
    .pipeline(pipeline)
    .numShots(5)             // 5-shot evaluation
    .numWorkers(4)           // parallel evaluation workers
    .build();

EvalRunner runner = new EvalRunner(config);
EvalResult result = runner.run();

System.out.printf("MMLU accuracy: %.2f%%%n", result.getScore() * 100);
result.getPerSubjectScores().forEach((subject, score) ->
    System.out.printf("  %s: %.2f%%%n", subject, score * 100));
```

### Available Benchmarks

| Benchmark | Class | Measures |
|-----------|-------|---------|
| MMLU | `MMLUBenchmark` | Massive Multitask Language Understanding (57 subjects) |
| ARC | `ArcBenchmark` | AI2 Reasoning Challenge (grade-school science) |
| GSM8K | `Gsm8kBenchmark` | Grade school math word problems |
| HellaSwag | `HellaSwagBenchmark` | Commonsense reasoning / sentence completion |
| TruthfulQA | `TruthfulQABenchmark` | Truthfulness and calibration |
| WinoGrande | `WinograndeBenchmark` | Pronoun coreference resolution |

### Metrics

| Metric | Class | Description |
|--------|-------|-------------|
| Exact Match | `ExactMatch` | Binary: prediction equals gold label |
| F1 | `F1` | Token-level F1 between prediction and gold |
| BLEU | `BLEU` | N-gram precision (translation quality) |
| ROUGE | `ROUGE` | Recall-oriented n-gram overlap (summarization) |
| ANLS | `ANLS` | Average Normalized Levenshtein Similarity (document QA) |
| VQA Accuracy | `VqaAccuracy` | Soft accuracy for visual question answering |
| Relaxed Accuracy | `RelaxedAccuracy` | Case/punctuation-insensitive exact match |
| Multiple Choice | `MultipleChoiceAccuracy` | Accuracy over A/B/C/D choices |

```java
import org.deeplearning4j.llm.eval.metrics.ROUGE;
import org.deeplearning4j.llm.eval.metrics.RougeVariant;

ROUGE rouge = new ROUGE(RougeVariant.ROUGE_L);
double score = rouge.compute(prediction, reference);
```

### Dataset Sources

| Class | Loads From |
|-------|-----------|
| `HuggingFaceDataset` | HuggingFace Hub (requires network) |
| `JsonlDataset` | Local JSONL file |
| `CsvDataset` | Local CSV file |
| `CustomDataset` | In-memory list of `(input, label)` pairs |
| `DatasetCache` | Wraps any dataset; caches to disk to avoid re-download |

```java
import org.deeplearning4j.llm.eval.datasets.JsonlDataset;
import org.deeplearning4j.llm.eval.datasets.DatasetCache;

JsonlDataset raw = JsonlDataset.builder()
    .path(Path.of("gsm8k_test.jsonl"))
    .inputField("question")
    .labelField("answer")
    .build();

// Cache to avoid re-reading the file on each evaluation run
DatasetCache cached = DatasetCache.wrap(raw, Path.of(".cache/gsm8k"));
```

### Perplexity

```java
import org.deeplearning4j.llm.eval.PerplexityEvaluator;

PerplexityEvaluator ppl = new PerplexityEvaluator(pipeline);
double perplexity = ppl.evaluate(Path.of("wikitext-103-test.txt"));
System.out.printf("Perplexity: %.2f%n", perplexity);
```

---

## 9. Model Editing / Abliteration

The model editing module provides tools for modifying model behavior by directly editing weight matrices. The primary use case implemented is _abliteration_: removing a model's refusal directions to understand or modify how refusal behavior is encoded in the model's weights. This is useful for research into model internals and for running ablations on safety-trained models in controlled research environments.

**Important:** These tools modify model weights irreversibly. Always work on a copy. Abliterated models should be used only within the bounds of your organization's AI safety policies.

### Abliteration Workflow

Abliteration works by:
1. Collecting activations for harmful and harmless prompt pairs (contrastive pairs).
2. Computing the mean activation difference between the two sets — the "refusal direction".
3. Orthogonalizing all weight matrices in the model against the refusal direction using Gram-Schmidt.

This removes the direction from the model's weight space so the model cannot activate along it, effectively removing the refusal behavior.

```java
import org.deeplearning4j.llm.edit.AbliterationWorkflow;
import org.deeplearning4j.llm.edit.AbliterationConfig;
import org.deeplearning4j.llm.edit.AbliterationResult;
import org.deeplearning4j.llm.edit.DefaultPromptSets;

AbliterationConfig config = AbliterationConfig.builder()
    .model(model)
    .tokenizer(tokenizer)
    .harmfulPrompts(DefaultPromptSets.HARMFUL_PROMPTS)        // built-in set
    .harmlessPrompts(DefaultPromptSets.HARMLESS_PROMPTS)      // built-in set
    .targetLayers(List.of(15, 16, 17, 18))                    // layers to edit
    .numActivationSamples(64)                                 // prompts per direction
    .build();

AbliterationWorkflow workflow = new AbliterationWorkflow(config);
AbliterationResult result = workflow.run();

System.out.printf("Edited %d weight matrices%n",
    result.getNumEditedMatrices());

// Save the modified model
SameDiff editedModel = result.getEditedModel();
editedModel.save(new File("model-abliterated.fb"), true);
```

### RefusalDirectionFinder

Used internally by `AbliterationWorkflow`, but can also be used standalone to analyze where refusal behavior is most strongly encoded across layers.

```java
import org.deeplearning4j.llm.edit.RefusalDirectionFinder;
import org.deeplearning4j.llm.edit.RefusalDirection;

RefusalDirectionFinder finder = new RefusalDirectionFinder(model, tokenizer);

List<RefusalDirection> directions = finder.find(
    harmfulPrompts, harmlessPrompts, layers: List.of(0, 8, 16, 24, 31));

for (RefusalDirection dir : directions) {
    System.out.printf("Layer %d: direction norm=%.4f%n",
        dir.getLayer(), dir.getDirection().norm2Number().floatValue());
}
```

### WeightOrthogonalizer

Applies the Gram-Schmidt orthogonalization to remove a direction from a weight matrix. Used by `AbliterationWorkflow` but also available directly.

```java
import org.deeplearning4j.llm.edit.WeightOrthogonalizer;

INDArray weightMatrix = model.getVariable("decoder/layer.16/mlp/down_proj/W").getArr();
INDArray direction   = refusalDirection.getDirection();

INDArray edited = WeightOrthogonalizer.orthogonalize(weightMatrix, direction);
```

---

## 10. Benchmarking

The benchmark framework measures LLM inference throughput under controlled conditions. It distinguishes between three throughput regimes that capture different aspects of serving performance.

### Throughput Metrics

| Metric | Description |
|--------|-------------|
| `lateSteady tok/s` | Tokens per second after full JIT warmup and cache warmup |
| `steady tok/s` | Tokens per second during the steady decode phase (most representative) |
| `decode tok/s` | Tokens per second for the decode phase only (excludes prefill) |

### BenchmarkConfig Presets

`BenchmarkConfig` ships four presets that control how the SameDiff graph is executed during the benchmark run.

| Preset | Constant | Description |
|--------|----------|-------------|
| Optimal | `BenchmarkConfig.OPTIMAL` | Lets the system select the best execution mode automatically |
| Slot-by-slot | `BenchmarkConfig.SLOT_BY_SLOT` | Executes one op at a time; useful for per-op profiling |
| Triton | `BenchmarkConfig.TRITON` | Routes eligible ops through Triton kernels (requires `tritonEnabled=true`) |
| CUDA Graphs | `BenchmarkConfig.CUDA_GRAPHS` | Captures and replays CUDA graphs; lowest decode latency on GPU |

### Running a Benchmark

```java
import org.deeplearning4j.llm.benchmark.BenchmarkRunner;
import org.deeplearning4j.llm.benchmark.BenchmarkConfig;
import org.deeplearning4j.llm.benchmark.BenchmarkResult;

BenchmarkConfig config = BenchmarkConfig.OPTIMAL
    .withFp16PreCast(true)          // cast weights to FP16 before benchmarking
    .withGraphOptimizer(true)       // enable SameDiff graph fusion passes
    .withTritonEnabled(false);      // set true to enable Triton kernel routing

BenchmarkRunner runner = BenchmarkRunner.builder()
    .pipeline(pipeline)
    .config(config)
    .prompt("The quick brown fox jumps over the lazy dog.")
    .warmupIterations(50)
    .benchmarkIterations(200)
    .build();

BenchmarkResult result = runner.run();

System.out.printf("steady tok/s:     %.1f%n", result.getSteadyThroughput());
System.out.printf("decode tok/s:     %.1f%n", result.getDecodeThroughput());
System.out.printf("lateSteady tok/s: %.1f%n", result.getLateSteadyThroughput());
System.out.printf("mean decode ms:   %.2f%n", result.getMeanDecodeMs());
System.out.printf("p99 decode ms:    %.2f%n", result.getP99DecodeMs());
```

### BenchmarkConfigApplier

`BenchmarkConfigApplier` is the only legitimate caller of `setGraphExecutionMode` on a `SameDiff` instance. If you need to apply a `BenchmarkConfig` to an existing pipeline outside of `BenchmarkRunner`, use it rather than calling `SameDiff` execution mode methods directly.

```java
import org.deeplearning4j.llm.benchmark.BenchmarkConfigApplier;

BenchmarkConfigApplier.apply(model, BenchmarkConfig.CUDA_GRAPHS);
```

### Decode Step Validation

The benchmark framework ships a suite of validation utilities for verifying that optimization changes do not alter numerical outputs.

```java
import org.deeplearning4j.llm.benchmark.DecodeValidationFramework;
import org.deeplearning4j.llm.benchmark.MultiLevelComparator;

DecodeValidationFramework validator = new DecodeValidationFramework(
    referenceModel,
    optimizedModel,
    new MultiLevelComparator(atol: 1e-3f, rtol: 1e-3f));

boolean pass = validator.validate("Test prompt for numerical equivalence.");
System.out.println("Validation: " + (pass ? "PASS" : "FAIL"));
```

---

## 11. VLM, Audio, and Other Modules

### samediff-vlm: Vision-Language Models

`samediff-vlm` extends the generation pipeline with image conditioning. The module handles image preprocessing (resize, normalize, patch extraction), image encoding via a vision encoder SameDiff graph, cross-attention injection into the language model, and the combined text-image generation loop.

```java
import org.deeplearning4j.vlm.VlmPipeline;
import org.deeplearning4j.vlm.VlmPipelineConfig;
import org.deeplearning4j.vlm.VlmGenerationResult;

SameDiff visionEncoder = SameDiff.load(new File("clip-vit-large.fb"), true);
SameDiff languageModel  = SameDiff.load(new File("llava-1.6-mistral-7b.fb"), true);

VlmPipelineConfig config = VlmPipelineConfig.builder()
    .visionEncoder(visionEncoder)
    .languageModel(languageModel)
    .tokenizer(TokenizerFactory.fromDirectory(Path.of("llava-tokenizer/")))
    .imageSize(336)                    // model-specific image resolution
    .build();

VlmPipeline vlm = new VlmPipeline(config);

BufferedImage image = ImageIO.read(new File("photo.jpg"));
String prompt = "Describe what is happening in this image in detail.";

VlmGenerationResult result = vlm.generate(image, prompt, DecodeOptions.defaults());
System.out.println(result.getText());
```

The `CLIPTokenizer` in `nd4j-tokenizers` is used by `samediff-vlm` to tokenize text for CLIP-family vision encoders. Text embeddings and image patch embeddings are concatenated in the language model's embedding space before the decode loop begins.

### samediff-audio: Whisper ASR

`samediff-audio` provides a complete Whisper automatic speech recognition pipeline, including mel spectrogram extraction, audio chunking for long audio, beam search decoding, and optional language detection.

```java
import org.deeplearning4j.audio.WhisperPipeline;
import org.deeplearning4j.audio.WhisperConfig;
import org.deeplearning4j.audio.TranscriptionResult;

SameDiff whisperModel = SameDiff.load(new File("whisper-large-v3.fb"), true);

WhisperConfig config = WhisperConfig.builder()
    .model(whisperModel)
    .language("en")           // or null for auto-detect
    .task(WhisperTask.TRANSCRIBE)
    .beamSize(5)
    .chunkLengthSeconds(30)   // Whisper processes 30-second chunks
    .build();

WhisperPipeline whisper = new WhisperPipeline(config);

// Input: 16 kHz mono PCM as INDArray
INDArray audio = loadAudio("interview.wav");

TranscriptionResult result = whisper.transcribe(audio);
System.out.println(result.getText());

// With timestamps
result.getSegments().forEach(seg ->
    System.out.printf("[%.2f → %.2f] %s%n",
        seg.getStart(), seg.getEnd(), seg.getText()));
```

### nd4j-torchscript: PyTorch Model Import

`nd4j-torchscript` imports TorchScript (`.pt`) files exported from PyTorch into native SameDiff graphs. This allows any PyTorch model that can be `torch.jit.traced` or `torch.jit.scripted` to be run without any Python dependency at inference time.

```java
import org.nd4j.torchscript.TorchScriptImporter;

// Export from PyTorch:
// traced = torch.jit.trace(model, example_input)
// traced.save("model.pt")

SameDiff sd = TorchScriptImporter.importModel(Path.of("model.pt"));

// Run inference
Map<String, INDArray> inputs = Map.of("input", inputTensor);
Map<String, INDArray> outputs = sd.outputAll(inputs);
```

Supported op coverage includes all ops commonly used in transformer architectures: matrix multiply, layer norm, softmax, attention, RoPE, SiLU/GELU activations, and element-wise operations. Unsupported ops will raise `TorchScriptImportException` with the op name.

### nd4j-web: Browser Frontend for ND4J Graphs

`nd4j-web` provides a TypeScript/FlatBuffers-based web frontend for visualizing and executing ND4J computation graphs in a browser. Graphs are serialized to FlatBuffers format and served over a lightweight HTTP endpoint. This is primarily useful for debugging graph structure and for building web-based tooling around ND4J models.

```java
import org.nd4j.web.Nd4jWebServer;

Nd4jWebServer server = Nd4jWebServer.builder()
    .port(8080)
    .graph(model)
    .build();

server.start();
System.out.println("ND4J graph viewer at http://localhost:8080");
```

Navigate to `http://localhost:8080` to see the graph structure, inspect variable shapes, and trigger execution from the browser.

---

## Next Steps

- **Getting Started:** See the [Quickstart](../quickstart.md) for setting up the Maven project and running your first model.
- **SameDiff Graph Execution:** Review the [SameDiff Execution documentation](../../nd4j/samediff/execution.md) to understand how `GenerationPipeline` integrates with the DSP plan lifecycle.
- **OmniHub Model Zoo:** Use [OmniHub](../../omnihub/usage.md) to download pre-converted LLM weights in the SameDiff FlatBuffers format without manual conversion.
- **Performance Tuning:** See [GPU/CPU Configuration](../../config/gpu-cpu.md) and [Memory and Workspaces](../../core-concepts/memory-and-workspaces.md) for hardware-specific tuning guidance that applies to LLM inference.
- **CUDA Graphs:** The `BenchmarkConfig.CUDA_GRAPHS` preset delivers the lowest decode latency on NVIDIA GPUs; see the [CUDA backend documentation](../../nd4j/backends/cuda.md) for prerequisites.

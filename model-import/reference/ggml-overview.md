---
title: GGML/GGUF Model Import
description: Import quantized LLMs from GGUF files — architecture handlers, quantization codecs, adaptive quantization, round-trip export, and pipeline modules
---

## GGML/GGUF Model Import

Eclipse Deeplearning4j 1.0.0-rewrite introduces native support for loading GGUF and GGML model files directly into SameDiff. This enables the JVM ecosystem to consume the enormous library of community-quantized models distributed through Hugging Face and other repositories — LLaMA, Gemma, Mistral, Phi, Qwen, Whisper, and many others — without any Python tooling or intermediate conversion step.

The implementation is split across the `nd4j-ggml` Maven module (87 files) and three pipeline SPI modules (34 files) that provide a pluggable format layer on top of SameDiff.

---

## When to Use GGML Import

| Scenario | Recommended approach |
|---|---|
| Run a community quantized LLM (.gguf) on the JVM | `GGMLModelImport.importModel(File)` |
| Inspect metadata and tensor layout before loading | `GGMLModelImport.inspectModel(File)` |
| Convert to DL4J native format for repeated use | `GGMLModelImport.convertToSDZ(src, dst)` |
| Export a SameDiff model back to GGUF | `GGMLModelExport.exportModel(SameDiff, File, ExportOptions)` |
| Load split multimodal GGUF bundles (e.g., Qwen3-VL) | `MultimodalGGUFLoader` |

---

## Maven Setup

The core import capability lives in `nd4j-ggml`. Add it alongside the ND4J backend for your platform.

```xml
<!-- GGML/GGUF model import -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-ggml</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- ND4J CPU backend (choose one) -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native-platform</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- ND4J CUDA backend (alternative) -->
<!--
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-12.3-platform</artifactId>
    <version>${dl4j.version}</version>
</dependency>
-->
```

For pipeline integration (format-agnostic loading across GGUF, SafeTensors, and ONNX), add the relevant SPI modules:

```xml
<!-- Pipeline core (shared SPI interfaces) -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>samediff-pipeline-core</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- GGUF pipeline adapter -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>samediff-pipeline-ggml</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- SafeTensors pipeline adapter (optional) -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>samediff-pipeline-safetensors</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- ONNX pipeline adapter (optional) -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>samediff-pipeline-onnx</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

Replace `${dl4j.version}` with your project version, for example `1.0.0-M2.1`.

---

## Quick Start

### Import a GGUF model into SameDiff

```java
import org.nd4j.ggml.GGMLModelImport;
import org.nd4j.autodiff.samediff.SameDiff;

import java.io.File;

File ggufFile = new File("llama-3-8b-instruct.Q4_K_M.gguf");

// Reads the GGUF file, dequantizes all tensors, and maps them to SameDiff variables
SameDiff model = GGMLModelImport.importModel(ggufFile);

System.out.println("Variables loaded: " + model.variables().size());
```

### Inspect a model without loading all weights

```java
import org.nd4j.ggml.GGMLModelImport;
import org.nd4j.ggml.format.GGMLMetadata;

File ggufFile = new File("model.gguf");
GGMLMetadata metadata = GGMLModelImport.inspectModel(ggufFile);

System.out.println("Architecture : " + metadata.getArchitecture());
System.out.println("Tensor count : " + metadata.getTensorCount());
System.out.println("Context length: " + metadata.getContextLength());
System.out.println("Quant type   : " + metadata.getQuantizationType());
```

### Convert to DL4J native format (SDZ)

```java
import org.nd4j.ggml.GGMLModelImport;

import java.io.File;

File src = new File("llama-3-8b-instruct.Q4_K_M.gguf");
File dst = new File("llama-3-8b-instruct.sdz");

// One-time conversion; subsequent loads from the .sdz are faster
GGMLModelImport.convertToSDZ(src, dst);
```

### Run a forward pass

```java
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.ggml.GGMLModelImport;

import java.io.File;
import java.util.Map;

SameDiff model = GGMLModelImport.importModel(new File("model.gguf"));

// Token IDs for a sample prompt (shape: [batch, seq_len])
INDArray inputIds = Nd4j.createFromArray(new long[][]{{1, 15043, 29892, 3186}});

Map<String, INDArray> outputs = model.output(
        Map.of("input_ids", inputIds),
        "logits"
);

INDArray logits = outputs.get("logits");
System.out.println("Logits shape: " + java.util.Arrays.toString(logits.shape()));
```

---

## GGUF Format Support

### File Format Versions

`GGUFReader` supports all three released versions of the GGUF binary format:

| Version | Notes |
|---|---|
| GGUF v1 | Original release format |
| GGUF v2 | Adds alignment padding for tensors |
| GGUF v3 | Extended metadata KV type set |

All versions share the same outer structure:

1. **Magic bytes** — `0x46554747` (`GGUF` in ASCII, little-endian)
2. **Version** — uint32
3. **Tensor count** — uint64
4. **Metadata KV count** — uint64
5. **Metadata KV pairs** — typed key-value entries (strings, scalars, arrays)
6. **Tensor descriptors** — name, shape, quantization type, offset
7. **Tensor data** — raw quantized bytes, padded to alignment boundary

`GGMLFormatDetector` reads the first four bytes of any file and selects either `GGUFReader` (magic `0x46554747`) or the legacy `GGMLReader` (older magic `0x67676d6c` / `0x67676d66`). You never need to choose the reader manually; `GGMLModelImport` calls the detector automatically.

### Legacy GGML Format

For pre-GGUF models (GGML format v1–v3), `GGMLReader` and `GGMLWriter` provide compatible reading and writing. These files lack the structured metadata KV section; architecture detection falls back to heuristics based on tensor name patterns.

---

## Supported Architectures

Architecture detection is handled by `ArchitectureRegistry`, which uses `ServiceLoader` auto-discovery and a priority ordering. Each handler implements the `ModelArchitecture` interface:

```java
public interface ModelArchitecture {
    boolean isCompatible(GGMLMetadata metadata);
    SameDiff buildSameDiff(GGUFReader reader, ConversionOptions options);
}
```

The registry iterates handlers in priority order and delegates to the first compatible one. `GenericArchitecture` is always last and accepts any model as a fallback.

### Architecture Handler Reference

| Architecture class | Model families | Notes |
|---|---|---|
| `LLaMAArchitecture` | LLaMA 1, LLaMA 2, LLaMA 3 | Standard dense transformer; RoPE positional encoding |
| `Llama4Architecture` | LLaMA 4 | Interleaved mixture-of-experts (MoE) layers |
| `GemmaArchitecture` | Gemma 2, Gemma 3 | Google's open models; grouped-query attention |
| `MistralArchitecture` | Mistral 7B, Mixtral | Sliding-window attention; optional MoE |
| `PhiArchitecture` | Phi-3, Phi-3.5 | Microsoft; ROPE + QKV fused projection |
| `GLMArchitecture` | ChatGLM, CodeGLM | Zhipu AI; bidirectional prefix attention |
| `GraniteArchitecture` | IBM Granite | IBM Research code/language models |
| `LFM2Architecture` | Liquid LFM-2 | State-space model (SSM) hybrid |
| `NemotronArchitecture` | NVIDIA Nemotron | NVIDIA instruction-tuned models |
| `OLMoArchitecture` | OLMo | Allen AI; no bias in attention |
| `OpenELMArchitecture` | OpenELM | Apple; layer-wise head count variation |
| `Qwen3VLArchitecture` | Qwen3-VL | Alibaba multimodal; loads split GGUF shards |
| `SmolVLM2Architecture` | SmolVLM2 | HuggingFace compact vision-language model |
| `MiniCPMVArchitecture` | MiniCPM-V | ModelBest multimodal |
| `WhisperArchitecture` | Whisper (tiny/base/small/medium/large) | OpenAI ASR; encoder-decoder |
| `GenericArchitecture` | Any | Fallback; maps tensors by name without graph rewiring |

`LayerTensorDiscovery` resolves GGUF tensor naming conventions (e.g., `blk.0.attn_q.weight`, `blk.0.ffn_gate.weight`) to the canonical SameDiff variable names used by each architecture.

---

## Quantization Formats

### Quantization Type Reference

`GGMLQuantType` enumerates every supported dtype with its bits-per-weight value. `GGMLDataType` provides the corresponding GGML integer dtype codes used in the binary format.

#### Standard quantization types

| Type | Bits/weight | Block size | Notes | When to use |
|---|---|---|---|---|
| `F32` | 32.0 | — | Full precision float32 | Accuracy-critical research; very large GPU |
| `F16` | 16.0 | — | Half precision float16 | Good balance; standard GPU inference |
| `Q8_0` | 8.5 | 32 | 8-bit, zero-point offset | Near-lossless; reference quality |
| `Q8_K` | 8.5 | 256 | 8-bit, K-quant block | Used as intermediate for K-quant dequant |
| `Q6_K` | 6.5625 | 256 | 6-bit K-quant | Excellent quality; fits larger models in RAM |
| `Q5_K_M` | 5.6875 | 256 | 5-bit K-quant, mixed precision | Recommended for quality-size balance |
| `Q5_K_S` | 5.5 | 256 | 5-bit K-quant, small | Slightly smaller than Q5_K_M |
| `Q5_1` | 5.5 | 32 | 5-bit with non-zero min | Legacy; prefer Q5_K_M |
| `Q5_0` | 5.5 | 32 | 5-bit, zero min | Legacy; prefer Q5_K_S |
| `Q4_K_M` | 4.8 | 256 | 4-bit K-quant, mixed precision | Most popular community choice |
| `Q4_K_S` | 4.375 | 256 | 4-bit K-quant, small | Compact; slightly lower quality |
| `Q4_1` | 4.5 | 32 | 4-bit with non-zero min | Legacy; prefer Q4_K_M |
| `Q4_0` | 4.0 | 32 | 4-bit, zero min | Smallest standard quant; legacy use |
| `Q3_K_L` | 3.4375 | 256 | 3-bit K-quant, large | Very compressed; some quality loss |
| `Q3_K_M` | 3.28125 | 256 | 3-bit K-quant, medium | Aggressive compression |
| `Q3_K_S` | 3.0 | 256 | 3-bit K-quant, small | Extreme compression |
| `Q2_K` | 2.5625 | 256 | 2-bit K-quant | Maximum compression; significant degradation |

#### I-quant types (importance-matrix quantization)

I-quants use a calibration dataset to assign higher precision to the weights that matter most. They require an importance matrix (imatrix) generated during quantization and generally outperform equivalent-BPW standard quants.

| Type | Bits/weight | Notes |
|---|---|---|
| `IQ4_XS` | 4.25 | 4-bit imatrix; best 4-bit quality |
| `IQ4_NL` | 4.0 | 4-bit non-linear imatrix |
| `IQ3_XXS` | 3.0625 | 3-bit ultra-small imatrix |
| `IQ3_S` | 3.4375 | 3-bit imatrix, standard |
| `IQ2_XXS` | 2.0625 | 2-bit ultra-small imatrix |
| `IQ2_XS` | 2.3125 | 2-bit imatrix, extra small |
| `IQ2_S` | 2.5 | 2-bit imatrix, standard |
| `IQ1_S` | 1.5625 | 1-bit imatrix; extreme compression |
| `IQ1_M` | 1.75 | 1-bit imatrix, mixed |

#### Ternary types

| Type | Bits/weight | Notes |
|---|---|---|
| `TQ1_0` | ~1.69 | Ternary quant v1 |
| `TQ2_0` | ~2.06 | Ternary quant v2; better accuracy than TQ1_0 |

### Dequantization at Import Time

During `GGMLModelImport.importModel()`, each tensor is dequantized to `float32` (or `float16` depending on `ConversionOptions`) before being stored as a SameDiff variable. A dedicated dequantizer class handles each format:

- Standard: `Q4_0Dequantizer`, `Q4_1Dequantizer`, `Q5_0Dequantizer`, `Q5_1Dequantizer`, `Q8_0Dequantizer`, `Q8_KDequantizer`, `Q4_KDequantizer`, `Q5_KDequantizer`, `Q6_KDequantizer`, `Q2_KDequantizer`, `Q3_KDequantizer`
- I-quant: `IQ1_MDequantizer`, `IQ1_SDequantizer`, `IQ2_SDequantizer`, `IQ2_XSDequantizer`, `IQ2_XXSDequantizer`, `IQ3_SDequantizer`, `IQ3_XXSDequantizer`, `IQ4_NLDequantizer`, `IQ4_XSDequantizer`
- Ternary: `TQ1_0Dequantizer`, `TQ2_0Dequantizer`

`GGMLToSameDiffConverter` coordinates this process: it iterates the tensor descriptors from `GGUFReader`, dispatches to the appropriate dequantizer, and creates the resulting `SDVariable` in the target `SameDiff` graph.

---

## Adaptive Quantization

When exporting a SameDiff model back to GGUF, you rarely want uniform quantization across all layers. The adaptive quantization subsystem assigns per-layer quantization types to meet a target model size budget while preserving quality in the most sensitive weight matrices.

### How it works

`AdaptiveLayerQuantizer` uses a two-pass algorithm:

1. **Analysis pass** — `DynamicQuantizationAnalyzer` inspects each weight tensor's value distribution (min, max, kurtosis, outlier ratio) and computes a recommended precision level.
2. **Budget pass** — `AdaptiveLayerQuantizer` maps the recommendations to concrete `GGMLQuantType` values, then adjusts to meet the total size budget. Attention projection matrices (`attn_q`, `attn_k`, `attn_v`, `attn_output`) are assigned a higher-quality quantization type than feed-forward matrices, since attention weights are more sensitive to precision loss.

### Quantizer classes (for export)

The following quantizer classes are available for re-quantizing `float32` tensors during export:

| Class | Output type |
|---|---|
| `Q4_0Quantizer` | Q4_0 |
| `Q4_1Quantizer` | Q4_1 |
| `Q4_KQuantizer` | Q4_K |
| `Q5_0Quantizer` | Q5_0 |
| `Q5_1Quantizer` | Q5_1 |
| `Q5_KQuantizer` | Q5_K |
| `Q6_KQuantizer` | Q6_K |
| `Q8_0Quantizer` | Q8_0 |

### Configuring adaptive quantization

```java
import org.nd4j.ggml.export.GGMLModelExport;
import org.nd4j.ggml.export.ExportOptions;
import org.nd4j.ggml.quantization.GGMLQuantType;

ExportOptions options = ExportOptions.builder()
        // Target model size in bytes (e.g., 4 GB)
        .targetSizeBytes(4L * 1024 * 1024 * 1024)
        // Default quant type for non-attention layers
        .defaultQuantType(GGMLQuantType.Q4_K_M)
        // Higher quality for attention projections
        .attentionQuantType(GGMLQuantType.Q6_K)
        // Enable per-layer dynamic analysis
        .adaptiveQuantization(true)
        .build();

GGMLModelExport.exportModel(sameDiff, new File("output.gguf"), options);
```

---

## Round-Trip Export

`GGMLModelExport` writes a SameDiff graph back to GGUF format, enabling workflows that:

- Fine-tune a model inside SameDiff, then re-export for use with llama.cpp or other GGUF-native runtimes.
- Quantize a float32 SameDiff model to GGUF for distribution.
- Convert between quantization levels (e.g., Q8_0 -> Q4_K_M) without leaving the JVM.

Export counterpart architecture classes (e.g., `LLaMAExportArchitecture`) handle the reverse tensor name mapping from SameDiff variable names back to the GGUF `blk.N.*` naming convention. `ExportArchitectureRegistry` mirrors `ArchitectureRegistry` and uses the same `ServiceLoader` discovery mechanism.

### Basic export

```java
import org.nd4j.ggml.export.GGMLModelExport;
import org.nd4j.ggml.export.ExportOptions;
import org.nd4j.ggml.quantization.GGMLQuantType;
import org.nd4j.autodiff.samediff.SameDiff;

import java.io.File;

SameDiff model = /* load or fine-tune your model */;

ExportOptions options = ExportOptions.builder()
        .defaultQuantType(GGMLQuantType.Q4_K_M)
        .build();

GGMLModelExport.exportModel(model, new File("finetuned-llama.gguf"), options);
```

`GGUFWriter` handles alignment padding so that the output file is compatible with any standard GGUF reader.

---

## Pipeline SPI Modules

The pipeline SPI provides a format-agnostic loading interface. When multiple format adapters are on the classpath, code written against `samediff-pipeline-core` works with any supported format without change.

### Module overview

| Maven artifact | Purpose |
|---|---|
| `samediff-pipeline-core` | Shared SPI interfaces (`ModelPipelineLoader`, `PipelineFormat`, etc.) |
| `samediff-pipeline-ggml` | Adapts `nd4j-ggml` behind the SPI; auto-registers via `ServiceLoader` |
| `samediff-pipeline-safetensors` | Loads Hugging Face SafeTensors (`.safetensors`) files |
| `samediff-pipeline-onnx` | Loads ONNX models through the SameDiff ONNX importer |

### Using the pipeline API

```java
import org.nd4j.pipeline.ModelPipelineLoader;
import org.nd4j.pipeline.PipelineFormat;
import org.nd4j.autodiff.samediff.SameDiff;

import java.io.File;

// The loader auto-detects the format from the file magic bytes / extension.
// If both samediff-pipeline-ggml and samediff-pipeline-safetensors are on
// the classpath, this single call works for .gguf or .safetensors files.
SameDiff model = ModelPipelineLoader.load(new File("model.gguf"));
```

Explicit format selection:

```java
SameDiff model = ModelPipelineLoader.load(new File("model.gguf"), PipelineFormat.GGUF);
SameDiff model2 = ModelPipelineLoader.load(new File("model.safetensors"), PipelineFormat.SAFETENSORS);
SameDiff model3 = ModelPipelineLoader.load(new File("model.onnx"), PipelineFormat.ONNX);
```

---

## Multimodal Model Support

Several vision-language and audio models distribute as multiple GGUF shards — a language model shard and one or more projection/vision encoder shards. `MultimodalGGUFLoader` assembles these into a unified `SameDiff` graph.

### Supported multimodal families

| Model family | Architecture handler | Shards |
|---|---|---|
| Qwen3-VL | `Qwen3VLArchitecture` | Language + vision encoder |
| SmolVLM2 | `SmolVLM2Architecture` | Language + vision encoder |
| MiniCPM-V | `MiniCPMVArchitecture` | Language + vision encoder |

### Loading a multimodal GGUF

```java
import org.nd4j.ggml.multimodal.MultimodalGGUFLoader;
import org.nd4j.autodiff.samediff.SameDiff;

import java.io.File;
import java.util.List;

// Provide the language model shard first, then vision encoder shard(s)
List<File> shards = List.of(
        new File("qwen3-vl-7b-language.gguf"),
        new File("qwen3-vl-7b-vision.gguf")
);

SameDiff model = MultimodalGGUFLoader.load(shards);
```

`MultimodalGGUFLoader` reads the metadata from each shard to identify its role, delegates to the appropriate `Qwen3VLArchitecture` (or equivalent), and merges the resulting SameDiff sub-graphs with shared variable namespaces.

---

## API Reference

### `GGMLModelImport`

Primary entry point for all import operations.

| Method | Signature | Description |
|---|---|---|
| `importModel` | `static SameDiff importModel(File file)` | Reads GGUF or GGML file, dequantizes all tensors, returns populated SameDiff graph |
| `importModel` | `static SameDiff importModel(File file, ConversionOptions options)` | Import with custom conversion options (target dtype, layer filter, etc.) |
| `inspectModel` | `static GGMLMetadata inspectModel(File file)` | Reads header and metadata only; does not load tensor data |
| `convertToSDZ` | `static void convertToSDZ(File src, File dst)` | Converts GGUF to DL4J native SDZ format for fast subsequent loads |

### `GGMLModelExport`

Round-trip export from SameDiff back to GGUF.

| Method | Signature | Description |
|---|---|---|
| `exportModel` | `static void exportModel(SameDiff sd, File dst, ExportOptions options)` | Writes SameDiff graph to GGUF with the given quantization options |

### `GGUFReader`

Low-level reader for the GGUF binary format.

| Method | Description |
|---|---|
| `readHeader()` | Parses magic, version, tensor count, KV count |
| `readMetadata()` | Returns the full metadata KV map |
| `readTensorDescriptors()` | Returns list of `TensorDescriptor` (name, shape, quant type, data offset) |
| `readTensorData(TensorDescriptor)` | Returns raw quantized bytes for a single tensor |

### `GGMLMetadata`

Structured view of GGUF metadata KV entries.

| Method | Description |
|---|---|
| `getArchitecture()` | Returns the `general.architecture` string (e.g., `"llama"`, `"gemma2"`) |
| `getTensorCount()` | Total number of tensors in the file |
| `getContextLength()` | `llm.context_length` KV value |
| `getQuantizationType()` | Most common quant type across all tensors |
| `get(String key)` | Returns raw KV value by key |

### `ArchitectureRegistry`

| Method | Description |
|---|---|
| `findCompatible(GGMLMetadata)` | Returns first compatible `ModelArchitecture` in priority order |
| `register(ModelArchitecture, int priority)` | Registers a custom architecture handler |
| `listAll()` | Returns all registered handlers |

### `GGMLQuantType`

Enum of quantization types with bits-per-weight metadata.

```java
GGMLQuantType qt = GGMLQuantType.Q4_K_M;
double bpw = qt.getBitsPerWeight();  // 4.8
int blockSize = qt.getBlockSize();   // 256
String name = qt.name();             // "Q4_K_M"
```

### `AdaptiveLayerQuantizer`

```java
import org.nd4j.ggml.quantization.AdaptiveLayerQuantizer;
import org.nd4j.ggml.quantization.QuantizationBudget;

QuantizationBudget budget = QuantizationBudget.ofBytes(4L * 1024 * 1024 * 1024);
AdaptiveLayerQuantizer quantizer = new AdaptiveLayerQuantizer(budget);

// Returns a map of SameDiff variable name -> assigned GGMLQuantType
Map<String, GGMLQuantType> plan = quantizer.buildQuantizationPlan(sameDiff);
```

### `DynamicQuantizationAnalyzer`

```java
import org.nd4j.ggml.quantization.DynamicQuantizationAnalyzer;
import org.nd4j.ggml.quantization.QuantizationRecommendation;
import org.nd4j.linalg.api.ndarray.INDArray;

DynamicQuantizationAnalyzer analyzer = new DynamicQuantizationAnalyzer();
INDArray weights = /* your weight tensor */;

QuantizationRecommendation rec = analyzer.analyze(weights);
System.out.println("Recommended type: " + rec.getRecommendedType());
System.out.println("Outlier ratio:    " + rec.getOutlierRatio());
```

### `MultimodalGGUFLoader`

```java
SameDiff model = MultimodalGGUFLoader.load(List<File> shards);
SameDiff model = MultimodalGGUFLoader.load(List<File> shards, ConversionOptions options);
```

---

## Format Detection and Custom Architectures

### Adding a custom architecture

Implement `ModelArchitecture` and register it either programmatically or via `ServiceLoader`:

```java
public class MyCustomArchitecture implements ModelArchitecture {

    @Override
    public boolean isCompatible(GGMLMetadata metadata) {
        return "my-arch".equals(metadata.getArchitecture());
    }

    @Override
    public SameDiff buildSameDiff(GGUFReader reader, ConversionOptions options) {
        SameDiff sd = SameDiff.create();
        // map tensors from reader to SameDiff variables
        for (TensorDescriptor td : reader.readTensorDescriptors()) {
            INDArray data = reader.readAndDequantize(td);
            sd.var(td.getName(), data);
        }
        return sd;
    }
}
```

Register programmatically (highest priority wins):

```java
ArchitectureRegistry.getInstance().register(new MyCustomArchitecture(), 100);
```

Or register via `ServiceLoader` by adding a file:

```
src/main/resources/META-INF/services/org.nd4j.ggml.architecture.ModelArchitecture
```

containing the fully qualified class name of your implementation. The registry picks it up automatically at startup.

---

## Troubleshooting

**`UnsupportedFormatException: Not a GGUF file`**: the file does not start with magic bytes `0x46554747`. Verify the file is a valid GGUF and was not corrupted during download. Use `GGMLFormatDetector.detect(file)` to check before importing.

**`UnknownArchitectureException`**: none of the registered architecture handlers returned `true` from `isCompatible()`. The model's `general.architecture` metadata key may be absent or set to an unrecognized value. `GenericArchitecture` should handle this as a fallback; if it does not, inspect the metadata with `GGMLModelImport.inspectModel()` and register a custom handler.

**Out of memory during dequantization**: dequantizing to float32 expands each tensor significantly. A 4-bit quantized 8B model is ~4 GB on disk but expands to ~16 GB in float32. Use `ConversionOptions.targetDtype(DataType.FLOAT16)` to halve the memory footprint, or process tensors one at a time using `GGUFReader` directly.

**Slow first load**: `importModel` dequantizes every tensor synchronously. For repeated use, call `convertToSDZ` once to produce a native `.sdz` file, which loads approximately 3-5x faster on subsequent runs.

**Split shard ordering**: `MultimodalGGUFLoader` expects shards in a specific order (language model first). Pass the files in the order listed in the model card. Incorrect ordering results in mismatched tensor namespaces.

**`ServiceLoader` not finding pipeline adapters**: ensure the `samediff-pipeline-ggml` jar is on the runtime classpath (not just compile scope). The SPI registration in `META-INF/services/` must be present in the deployed artifact.

---

## Further Reading

- [Model Import Overview](../overview) — comparison of all import paths in DL4J
- [SameDiff TF/ONNX Import](../samediff-import/overview) — SameDiff-native import for TF and ONNX
- [ONNX Runtime](../onnx-runtime/overview) — direct ONNX inference without graph conversion
- [nd4j-ggml source code](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/ggml)
- [GGUF format specification](https://github.com/ggerganov/ggml/blob/master/docs/gguf.md)
- [DL4J examples repository](https://github.com/eclipse/deeplearning4j-examples)

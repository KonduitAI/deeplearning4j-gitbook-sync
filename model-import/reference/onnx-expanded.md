---
title: ONNX Import & Export (Expanded)
description: ~120 new ONNX op implementations including Microsoft LLM contrib ops, ONNX ML domain classifiers, quantized inference ops, and bidirectional SameDiff-to-ONNX export
---

## ONNX Import & Export (Expanded)

PR #10450 adds approximately 120 new op implementations to `nd4j-samediff-import-onnx` and introduces a full bidirectional pipeline: SameDiff graphs can now be **exported** to ONNX in addition to being imported. This page covers all newly implemented operators, updated ops, and the export API.

For the baseline import guide and `OnnxGraphMapper` entry point, see the [ONNX Import page](./onnx).

---

## 1. Scope of Expansion

The expansion covers four areas:

- **Standard ONNX ops** — pooling variants, advanced indexing, sequence types, signal processing, image preprocessing, and control-flow ops reaching opset 17/18.
- **Microsoft `com.microsoft` contrib ops** — LLM-specific fused operators used by Phi, Mistral, LLaMA, and T5 models (exported via Optimum or onnxruntime-genai).
- **ONNX ML domain (`ai.onnx.ml`)** — preprocessing and classical ML operators for scikit-learn / XGBoost / LightGBM models exported with sklearn-onnx.
- **Quantized inference ops** — INT8 and dynamic quantization operators for quantized model deployment without host-side dequantization.

A companion export pipeline (`OnnxExporter`, `OnnxExportConfig`, `SameDiffToOnnxOpMapper`) serializes any SameDiff graph to a valid ONNX `ModelProto`. The `samediff-pipeline-onnx` module registers an SPI `ONNXPipelineLoader` so ONNX models integrate with the nd4j pipeline abstraction automatically.

---

## 2. Maven Setup

```xml
<!-- Core ONNX import and export -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-samediff-import-onnx</artifactId>
    <version>${dl4j.version}</version>
</dependency>

<!-- Pipeline SPI loader (optional) -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>samediff-pipeline-onnx</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

---

## 3. Standard ONNX Ops (Newly Added)

### Pooling

| Op | Description |
|---|---|
| `AdaptiveAvgPool` | Average pooling with dynamic output size (PyTorch `AdaptiveAvgPool2d`-equivalent) |
| `AdaptiveMaxPool` | Max pooling with dynamic output size |
| `GlobalLpPool` | Global Lp-norm pooling across spatial dims |
| `LpPool` | Windowed Lp-norm pooling |
| `MaxUnpool` | Unpools using recorded argmax indices (inverse of MaxPool) |

### Activation Functions

| Op | Description |
|---|---|
| `Celu` | Continuously differentiable ELU: `max(0,x) + min(0,alpha*(exp(x/alpha)-1))` |
| `Gelu` | Gaussian error linear unit |
| `HardSwish` | `x * ReLU6(x+3) / 6`; efficient mobile activation |
| `Hardmax` | One-hot argmax across a specified axis |
| `Mish` | `x * tanh(softplus(x))` |
| `ParametricSoftplus` | `alpha * log(1 + exp(beta * x))` with learnable parameters |
| `QuickGelu` | `x * sigmoid(1.702 * x)`; approximate GELU used in CLIP/GPT |
| `ScaledTanh` | `alpha * tanh(beta * x)` |
| `ThresholdedRelu` | `x if x > theta else 0` |

### Convolution Variants

| Op | Description |
|---|---|
| `ConvInteger` | INT8 convolution; accumulates to INT32 |
| `ConvTranspose` | Transposed (fractionally-strided) convolution |
| `DeformableConv` | Deformable convolution with offset and mask inputs (DCNv2) |
| `NhwcConv` | Convolution with NHWC layout (avoids transpose for channels-last inputs) |
| `QLinearConv` | Quantized convolution with per-tensor or per-channel scale/zero-point |
| `QLinearMatMul` | Quantized matrix multiply |

### Normalization

| Op | Description |
|---|---|
| `GroupNormalization` | Groups channels then normalizes (ONNX opset 18) |
| `InstanceNormalization` | Per-sample, per-channel normalization |
| `LpNormalization` | Normalizes slices to unit Lp-norm |
| `MeanVarianceNormalization` | Subtracts mean, divides by std across specified axes |

### Recurrent Layers

| Op | Description |
|---|---|
| `LSTM` | Long short-term memory; full bidirectional support; optional peephole connections |
| `RNN` | Single-layer Elman RNN with configurable activation |

### Loss Functions

| Op | Description |
|---|---|
| `LogSoftmax` | Log-domain softmax; numerically stable |
| `NegativeLogLikelihoodLoss` | NLL loss with optional weight and ignore-index |
| `SoftmaxCrossEntropyLoss` | Fused softmax + cross-entropy with optional label smoothing |

### Image Preprocessing

| Op | Description |
|---|---|
| `AffineGrid` | Generates a sampling grid from an affine theta matrix (spatial transformer) |
| `CenterCropPad` | Crops or pads a tensor to a target shape centered on spatial dimensions |
| `Col2Im` | Reconstructs an image from column form (inverse of Im2Col) |
| `GridSample` | Samples input at grid coordinates with bilinear or nearest interpolation |
| `Im2Col` | Extracts sliding patches from a batched image |
| `ImageScaler` | Scales and offsets image pixel values |

### Advanced Indexing

| Op | Description |
|---|---|
| `GatherElements` | Gathers values along an axis using an indices tensor (`torch.gather` equivalent) |
| `GatherND` | Gathers slices using a multi-dimensional indices tensor |
| `ScatterElements` | Scatters values into a tensor at specified indices |
| `ScatterND` | Scatters updates using ND index slices |

### Sequence, Control Flow, and Misc

| Op | Description |
|---|---|
| `Bitshift` | Bitwise left or right shift |
| `CTCGreedyDecoder` | CTC greedy decoder for sequence transduction |
| `ConcatFromSequence` | Concatenates all tensors in an ONNX sequence along a given axis |
| `Compress` | Selects slices where a condition mask is true |
| `DepthToSpace` | Rearranges depth blocks into spatial blocks (pixel shuffle) |
| `Einsum` | Einstein summation over any combination of operand indices |
| `EyeLike` | Creates an identity-like matrix matching the shape/dtype of a reference |
| `Mean` / `Sum` | Element-wise mean and sum across a list of inputs |
| `Multinomial` | Draws random samples from a multinomial distribution |
| `OptionalOps` | `Optional`, `OptionalGetElement`, `OptionalHasElement` sequence type support |
| `RandomNormalLike` / `RandomUniformLike` | Fills a tensor shaped like a reference with random values |
| `ReverseSequence` | Reverses variable-length sequences within a batch |
| `Scan` | Recurrent scan operator for general loop-body sub-graphs |
| `SpaceToDepth` | Collapses spatial blocks into the depth dimension (inverse of DepthToSpace) |
| `SplitToSequence` | Splits a tensor into an ONNX sequence along an axis |
| `Trilu` | Extracts upper or lower triangle of a matrix |
| `Unique` | Returns unique elements with optional sorted order and counts |

### Reductions

`ReduceLogSum`, `ReduceLogSumExp`, `ReduceMax`, `ReduceMean`, `ReduceMin`, `ReduceProd`, `ReduceSum`, `ReduceSumSquare` — all updated to handle the opset-18 axis-as-tensor-input signature.

### Object Detection

| Op | Description |
|---|---|
| `MaxRoiPool` | Region-of-interest max pooling (Faster R-CNN) |
| `NonMaxSuppression` | Filters bounding boxes using IoU and score thresholds |

### Signal Processing

| Op | Description |
|---|---|
| `DFT` | Discrete Fourier Transform (opset 17) |
| `STFT` | Short-Time Fourier Transform with configurable window and hop length |

---

## 4. Microsoft Contrib Ops (`com.microsoft`)

These ops appear in models exported by Hugging Face Optimum (`--optimize`) or onnxruntime-genai and are required for Phi, Mistral, LLaMA, and T5 families.

| Op | Description |
|---|---|
| `FusedConv` | Convolution fused with an activation in a single kernel |
| `FusedGemm` | General matrix multiply fused with activation |
| `FusedMatMul` | Matrix multiply with fused activation; supports transA/transB, alpha/beta |
| `GroupQueryAttention` | Multi-head attention with grouped KV heads (GQA); used in Mistral, LLaMA-2 |
| `MixtureOfExperts` | Sparse MoE routing layer with top-K expert selection |
| `MultiHeadAttention` | Standard MHA; routes through native `OnnxMultiHeadAttention` op |
| `RelativePositionBias` | T5-style learned relative position bias added to attention logits |
| `RotaryEmbedding` | Rotary Position Embedding (RoPE); used by LLaMA, Mistral, Phi, Falcon |
| `SimplifiedLayerNormalization` | RMS-only layer norm without mean subtraction (LLaMA) |
| `SkipSimplifiedLayerNormalization` | Fused skip-connection + simplified layer norm |
| `WindowedAttention` | Sliding-window (local) attention for long-context inference |

**Note:** `GroupQueryAttention` validates that `kv_num_heads` divides `num_heads` at import time. An informative error is raised if the model's attention configuration is inconsistent.

---

## 5. ONNX ML Domain Ops (`ai.onnx.ml`)

### Preprocessing

| Op | Description |
|---|---|
| `CastMap` | Maps integer keys to float/string values from a dictionary |
| `CategoryMapper` | Bidirectional mapping between integer categories and string labels |
| `DictVectorizer` | Converts a map (string→float or int64→float) to a dense vector |
| `FeatureVectorizer` | Concatenates multiple numeric feature inputs into one vector |
| `Imputer` | Replaces missing or NaN values with a user-specified constant |
| `LabelEncoder` | Encodes string labels as integers or vice versa |
| `Normalizer` | Normalizes rows to unit L1, L2, or max norm |
| `Scaler` | Subtracts offset and multiplies by scale per feature |
| `TfIdfVectorizer` | Converts token sequences to TF-IDF weighted feature vectors |

### Classical ML Classifiers and Regressors

| Op | Description |
|---|---|
| `LinearClassifier` | Linear classification (logistic regression, linear SVM) |
| `LinearRegressor` | Linear regression |
| `SVMClassifier` | SVM classifier (RBF, polynomial, sigmoid kernels) |
| `SVMRegressor` | SVM regressor |
| `TreeEnsembleClassifier` | Random forest / gradient-boosted tree classifier |
| `TreeEnsembleRegressor` | Random forest / gradient-boosted tree regressor |

---

## 6. Quantized Inference Ops

| Op | Description |
|---|---|
| `DequantizeLinear` | Converts INT8/UINT8 to float using per-tensor or per-channel scale/zero-point |
| `DynamicQuantizeLinear` | Dynamically quantizes float to UINT8; returns scale and zero-point as side outputs |
| `MatMulInteger` | INT8 matrix multiply accumulating to INT32 |
| `QuantizeLinear` | Converts float to INT8/UINT8 using scale and zero-point |

`ConvInteger` and `QLinearConv` / `QLinearMatMul` are listed under Section 3 (Convolution Variants). Per-axis quantization for `DequantizeLinear` / `QuantizeLinear` requires opset 13 or higher.

---

## 7. Updated Ops and Opset Compatibility

Twenty existing op mappers were revised for opset 13–18 behavioral changes:

`Cast` (saturate attribute), `Clip` (min/max as tensor inputs), `ConstantOfShape`, `Conv` (grouped/dilated fixes), `CumSum`, `Dropout` (ratio as tensor), `Equal`, `Expand`, `Gather` (negative index clamping), `GRU` (bidirectional output ordering), `GlobalAveragePooling`, `GlobalMaxPooling`, `Greater`, `MatMul` (batched beyond 3D), `PRelu`, `Reshape` (zero-copy), `Resize` (antialias/cubic), `RoiAlign` (coordinate_transformation_mode), `Shape` (start/end attributes opset 15+), `Slice` (tensor-form steps), `Split` (dynamic split tensor), `Transpose`, `Unsqueeze` (axes as tensor), `Where` (scalar condition broadcasting).

**Recommended opset range:** 11–18. Opset 19 attributes are accepted but may fall back to opset-18 semantics for non-critical differences.

---

## 8. ONNX Export Pipeline

### Classes

| Class | Role |
|---|---|
| `OnnxExporter` | Main entry point; converts `SameDiff` to `Onnx.ModelProto`; writes to `File` or `OutputStream` |
| `OnnxExportConfig` | Configuration: opset version, external data threshold, training state flag |
| `SameDiffToOnnxOpMapper` | Per-op translation table mapping SameDiff op names to ONNX proto nodes |
| `PostExportHook` | Interface for post-processing the `ModelProto` after initial translation |
| `TrainingStateExporter` | Serializes Adam optimizer states (m/v) into ONNX initializers |
| `hooks/BatchNormExportHook` | Converts fused batchnorm to the ONNX `BatchNormalization` attribute layout |
| `hooks/ConvExportHook` | Adjusts weight tensor layout from SameDiff's internal order to ONNX's expected ordering |

### OnnxExportConfig

```java
OnnxExportConfig config = OnnxExportConfig.builder()
    .opsetVersion(17)                          // target opset (11–18; default 17)
    .includeTrainingState(false)               // embed Adam m/v states for checkpoint export
    .externalDataThreshold(1024L * 1024L)      // bytes; tensors above threshold written externally
    .build();
```

### Export to File

```java
SameDiff sd = buildOrLoadYourModel();

OnnxExporter exporter = new OnnxExporter();
exporter.export(sd, new File("my_model.onnx"),
    OnnxExportConfig.builder().opsetVersion(17).build());
```

### Export to OutputStream

```java
ByteArrayOutputStream baos = new ByteArrayOutputStream();
new OnnxExporter().export(sd, baos,
    OnnxExportConfig.builder().opsetVersion(17).build());
byte[] onnxBytes = baos.toByteArray();
```

### Export with Training State

```java
OnnxExportConfig config = OnnxExportConfig.builder()
    .opsetVersion(17)
    .includeTrainingState(true)   // embeds {param}__adam_m and {param}__adam_v initializers
    .build();
new OnnxExporter().export(trainedSd, new File("checkpoint.onnx"), config);
```

### Custom PostExportHook

```java
public class FuseReluHook implements PostExportHook {
    @Override
    public Onnx.ModelProto postProcess(Onnx.ModelProto model, SameDiff originalGraph) {
        // Modify the proto — e.g. merge consecutive Gemm + Relu nodes
        return model.toBuilder()
            // ... transformations ...
            .build();
    }
}

OnnxExporter exporter = new OnnxExporter();
exporter.addPostExportHook(new FuseReluHook());
exporter.export(sd, new File("fused.onnx"), config);
```

---

## 9. Pipeline SPI Module

`samediff-pipeline-onnx` registers `ONNXPipelineLoader` via `META-INF/services`. Adding the artifact to the classpath is all that is needed:

```java
// ServiceLoader discovers ONNXPipelineLoader automatically
SameDiff sd = PipelineLoader.load(new File("model.onnx"));
```

Swapping between TF and ONNX loading requires only a classpath change — no code modification.

---

## 10. Import Examples

### Vision Model (NCHW)

```java
SameDiff sd = OnnxGraphMapper.importGraph(new File("efficientnet_b0.onnx"));

INDArray image = Nd4j.rand(1, 3, 224, 224);   // NCHW
Map<String, INDArray> out = sd.output(
    Collections.singletonMap("input", image), "output");
int cls = out.get("output").argMax(1).getInt(0);
```

### LLM with Microsoft Contrib Ops (LLaMA / Mistral)

```java
// Model exported via: optimum-cli export onnx --model meta-llama/Llama-2-7b-hf llama_onnx/
SameDiff sd = OnnxGraphMapper.importGraph(new File("llama_onnx/model.onnx"));

INDArray inputIds      = Nd4j.createFromArray(new long[][]{{1, 450, 3437, 310, 5113}});
INDArray attentionMask = Nd4j.ones(1, 5);
INDArray positionIds   = Nd4j.arange(5).reshape(1, 5);

Map<String, INDArray> inputs = Map.of(
    "input_ids",      inputIds,
    "attention_mask", attentionMask,
    "position_ids",   positionIds);

INDArray logits = sd.output(inputs, "logits").get("logits");
// logits: [1, seqLen, vocabSize]
```

### scikit-learn Pipeline (ONNX ML Ops)

```java
SameDiff sd = OnnxGraphMapper.importGraph(new File("sklearn_pipeline.onnx"));
INDArray features = Nd4j.createFromArray(new float[][]{{5.1f, 3.5f, 1.4f, 0.2f}});
Map<String, INDArray> out = sd.output(
    Collections.singletonMap("float_input", features),
    "output_label", "output_probability");
System.out.println("Label: " + out.get("output_label").getInt(0));
```

### Quantized INT8 Model (BERT)

```java
SameDiff sd = OnnxGraphMapper.importGraph(new File("bert_int8.onnx"));

long seqLen = 128;
DataType i64 = DataType.INT64;
Map<String, INDArray> inputs = Map.of(
    "input_ids",      Nd4j.zeros(i64, 1, seqLen),
    "attention_mask", Nd4j.ones(i64,  1, seqLen),
    "token_type_ids", Nd4j.zeros(i64, 1, seqLen));

INDArray embeddings = sd.output(inputs, "last_hidden_state").get("last_hidden_state");
```

---

## 11. Export Examples

### Train in SameDiff, Export to ONNX

```java
SameDiff sd = SameDiff.create();
SDVariable x      = sd.placeHolder("input",   DataType.FLOAT, -1, 784);
SDVariable w      = sd.var("weights",          DataType.FLOAT, 784, 10);
SDVariable b      = sd.var("bias",             DataType.FLOAT, 10);
SDVariable output = sd.math.softmax("output",  sd.nn.linear(x, w, b), 1);

// ... sd.fit(...) ...

new OnnxExporter().export(sd, new File("mlp.onnx"),
    OnnxExportConfig.builder().opsetVersion(17).build());
```

### Import, Fine-Tune, Re-Export with Checkpoint

```java
SameDiff sd = OnnxGraphMapper.importGraph(new File("pretrained.onnx"));

// sd.fit(datasetIterator, epochs);

new OnnxExporter().export(sd, new File("finetuned_checkpoint.onnx"),
    OnnxExportConfig.builder()
        .opsetVersion(17)
        .includeTrainingState(true)   // preserves Adam m/v for training resumption
        .build());
```

### Large Model with External Data

```java
new OnnxExporter().export(sd, new File("large_model.onnx"),
    OnnxExportConfig.builder()
        .opsetVersion(17)
        .externalDataThreshold(100L * 1024 * 1024)   // 100 MB threshold
        .build());
// Produces large_model.onnx + large_model.onnx.data
```

---

## Troubleshooting

**`UnsupportedOperationException: No mapper found for ONNX op: <name>`** — The model uses an op not yet mapped. File a GitHub issue with the ONNX model's op list (use `python -c "import onnx; m=onnx.load('model.onnx'); print(set(n.op_type for n in m.graph.node))"`). For `com.microsoft` ops, confirm the artifact version is M2.1 or later.

**`IllegalArgumentException: GroupQueryAttention: kv_num_heads must divide num_heads`** — Inspect the exported ONNX node attributes and verify the source-framework attention configuration.

**Export produces invalid ONNX proto** — Run `python -c "import onnx; onnx.checker.check_model('my_model.onnx')"`. Ensure `BatchNormExportHook` and `ConvExportHook` are registered before calling `export`.

**Training state initializers missing after re-import** — Verify with `sd.getVariable("{param_name}__adam_m")`. Name mismatches mean the optimizer state mapping will be empty.

**Quantized op outputs differ from ONNX Runtime** — Use tolerance `1e-2` rather than strict equality. INT8 accumulation order can vary across backends.

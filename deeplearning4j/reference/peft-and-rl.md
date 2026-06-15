---
title: PEFT & RL Alignment Training
description: Parameter-efficient fine-tuning (12 methods), reinforcement learning alignment (9 trainers), mixed-precision training, knowledge distillation, and dataset curation
---

# PEFT & RL Alignment Training

Deeplearning4j 1.0.0-rewrite adds a complete suite of tools for fine-tuning large pretrained models without retraining all parameters, aligning language model outputs using reinforcement learning feedback, training with reduced numerical precision, and curating datasets for domain adaptation.

---

## 1. Overview

The following capability groups are covered in this page:

| Area | Key Classes | What It Provides |
|------|-------------|-----------------|
| PEFT | `PeftModel`, `PeftModelFactory`, `LoraLayer` | 13 adapter types; freeze base weights, train only adapter parameters |
| RL Alignment | `GRPOTrainer`, `DPOTrainer`, `PPOTrainer`, and 7 more | Human-preference and reward-signal alignment |
| Training Pipelines | `SFTTrainingPipeline`, `RLAlignmentPipeline` | End-to-end supervised and RLHF workflows |
| Mixed Precision | `FP8ScaleManager`, `LossScaler` | FP8 forward/backward, dynamic loss scaling |
| 8-bit Adam | `Adam8bit`, `Adam8bitUpdater` | ~4x optimizer-state memory reduction |
| Knowledge Distillation | `DistillationTrainer` | Teacher/student training with KL, attention, and feature losses |
| Dataset Curation | `TextDeduplicator`, `SequencePacker`, and 18 more | Deduplication, contamination removal, curriculum, bin-packing |
| Transfer Learning | `TransferLearning`, `TransferLearningHelper` | Layer freezing and head replacement on existing models |

All SameDiff models gain these extension methods automatically:

```java
sd.applyPeft(PeftConfig config);
sd.getTrainableParameters();
sd.printTrainableParameters();
sd.distillFrom(SameDiff teacher, DistillationConfig config);
sd.fitGRPO(GRPOConfig config, MultiDataSetIterator data);
sd.saveAdapters(Path dir);
sd.loadAdapters(Path dir);
```

---

## 2. Maven Dependencies

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-core</artifactId>
    <version>1.0.0-rewrite</version>
</dependency>
<!-- PEFT and RL alignment trainers -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-peft</artifactId>
    <version>1.0.0-rewrite</version>
</dependency>
<!-- 8-bit Adam and FP8 mixed precision -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-12.x</artifactId>
    <version>1.0.0-rewrite</version>
</dependency>
```

---

## 3. PEFT Methods

### 3.1 PeftType Enum

Every PEFT method is identified by a `PeftType` constant:

| PeftType | Description |
|----------|-------------|
| `LORA` | Low-Rank Adaptation — low-rank A/B matrices added to linear projections |
| `QLORA` | LoRA on NF4-quantized base weights |
| `DORA` | Weight-Decomposed LoRA with learned magnitude vector |
| `LOHA` | Low-rank Hadamard product adaptation |
| `LOKR` | Low-rank Kronecker product adaptation |
| `IA3` | Infused Adapter by Inhibiting and Amplifying Inner Activations |
| `VERA` | Vector-based Random Matrix Adaptation |
| `PREFIX_TUNING` | Trainable key/value prefix tokens prepended to each layer |
| `PROMPT_TUNING` | Soft prompt tokens prepended to the input embedding |
| `ADAPTER` | Small bottleneck feed-forward modules inserted between layers |
| `LOFTQ` | LoRA initialized to minimize NF4 quantization error |
| `ADALORA` | Adaptive rank allocation across layers |
| `DYLORA` | Dynamic rank search during training |

### 3.2 PeftModel

`PeftModel` wraps an existing `SameDiff` base model and injects adapter layers without modifying the base graph:

```java
import org.deeplearning4j.peft.PeftModel;
import org.deeplearning4j.peft.config.LoraConfig;

SameDiff baseModel = SameDiff.load(Paths.get("llama-7b.fb"), true);

LoraConfig config = LoraConfig.builder()
    .r(16)
    .loraAlpha(32)
    .loraDropout(0.05)
    .targetModules(List.of("q_proj", "v_proj"))
    .biasMode(BiasMode.NONE)
    .initLoraWeights(InitLoraWeights.KAIMING)
    .build();

PeftModel peft = PeftModelFactory.fromConfig(baseModel, config);

// Print how many parameters are actually trained
peft.printTrainableParameters();
// Output: trainable params: 4,194,304 || all params: 6,742,609,920 || trainable%: 0.0623
```

Key `PeftModel` capabilities:

| Method | Description |
|--------|-------------|
| `printTrainableParameters()` | Logs trainable vs total parameter count and percentage |
| `getTrainableParameterCount()` | Returns the count of adapter (non-frozen) parameters |
| `addAdapter(String name, PeftConfig config)` | Inject an additional named adapter onto the same base model |
| `setActiveAdapter(String name)` | Switch which adapter is active for inference |
| `mergeAndUnload()` | Merge adapter weights into base weights and return a plain `SameDiff` |
| `saveAdapters(Path dir)` | Serialize only adapter weights (small files) |
| `loadAdapters(Path dir)` | Restore previously saved adapter weights |

Base model parameters are automatically frozen when `PeftModel` is created. Only the adapter parameters listed in `targetModules` are in the computation graph as trainable variables.

### 3.3 PeftModelFactory

`PeftModelFactory.fromConfig(SameDiff baseSd, PeftConfig config)` inspects `config.getPeftType()` and dispatches to the correct adapter injection strategy:

```java
// Dispatches based on PeftType
PeftModel model = PeftModelFactory.fromConfig(baseSd, loraConfig);   // LORA
PeftModel model = PeftModelFactory.fromConfig(baseSd, qloraConfig);  // QLORA
PeftModel model = PeftModelFactory.fromConfig(baseSd, adapConfig);   // ADAPTER
```

### 3.4 LoraLayer

`LoraLayer` is the core adapter unit for all LoRA variants. It wraps an existing linear op (matrix multiply) with low-rank side-path A and B matrices:

```
output = base_weight @ x + (lora_B @ lora_A @ x) * (alpha / r)
```

Fields tracked per layer:

| Field | Description |
|-------|-------------|
| `r` | Rank of the factorization |
| `alpha` | Scaling factor; effective scale = alpha / r |
| `dropout` | Dropout probability applied to the adapter path |

`mergeWeights()` adds `(lora_B @ lora_A) * (alpha / r)` directly into the base weight matrix, eliminating the side path and returning a standard linear op with no inference overhead.

### 3.5 LoraAdapterCache

`LoraAdapterCache` provides thread-safe, LRU-evicted in-memory caching of adapter weights for multi-adapter serving scenarios:

```java
import org.deeplearning4j.peft.cache.LoraAdapterCache;

LoraAdapterCache cache = new LoraAdapterCache(maxAdapters /* e.g., 10 */);

// Load adapter into cache keyed by (modelId, adapterName)
cache.put("llama-7b", "customer-a-adapter", adapterWeights);

// Retrieve
Map<String, INDArray> weights = cache.get("llama-7b", "customer-a-adapter");
```

The LRU policy evicts the least-recently-used adapter when the cache is full, making it suitable for serving many fine-tuned variants from a single base model without loading all adapters into GPU memory simultaneously.

### 3.6 LoftQInitializer

`LoftQInitializer` produces LoRA A/B initializations that specifically minimize the error introduced by NF4 quantization:

```
error = base_weight - dequantize(nf4_quantize(base_weight))
Initialize A, B such that B @ A ≈ error
```

```java
import org.deeplearning4j.peft.init.LoftQInitializer;

LoftQInitializer init = new LoftQInitializer(numBits /* 4 */, numIter /* 5 */);
init.initialize(loraLayer, baseWeight);
```

Use `initLoraWeights(InitLoraWeights.LOFTQ)` in `LoraConfig` to apply this automatically via `PeftModelFactory`.

---

## 4. LoRA Deep Dive

### 4.1 LoraConfig

```java
import org.deeplearning4j.peft.config.LoraConfig;

LoraConfig config = LoraConfig.builder()
    .r(16)                                              // rank
    .loraAlpha(32)                                      // scaling; effective_scale = alpha/r = 2.0
    .loraDropout(0.05)                                  // dropout on adapter path
    .targetModules(List.of("q_proj", "k_proj",
                           "v_proj", "o_proj"))         // layers to adapt
    .biasMode(BiasMode.NONE)                            // NONE | ALL | LORA_ONLY
    .initLoraWeights(InitLoraWeights.KAIMING)           // GAUSSIAN | KAIMING | LOFTQ
    .build();
```

`biasMode` controls which bias vectors receive gradients:

| BiasMode | Effect |
|----------|--------|
| `NONE` | No bias vectors are trained (default) |
| `ALL` | All bias vectors (base + adapter layers) are trained |
| `LORA_ONLY` | Only bias vectors in LoRA-targeted layers are trained |

### 4.2 QLoraConfig

`QLoraConfig` extends `LoraConfig` and adds NF4 quantization of the base weights. The base model loads in 4-bit and stays frozen; only the LoRA adapter trains in full precision:

```java
import org.deeplearning4j.peft.config.QLoraConfig;

QLoraConfig config = QLoraConfig.builder()
    .r(64)
    .loraAlpha(16)
    .loraDropout(0.1)
    .targetModules(List.of("q_proj", "v_proj"))
    .quantBits(4)                   // NF4 quantization
    .doubleQuant(true)              // quantize the quantization constants as well
    .computeDtype(DataType.BFLOAT16)
    .build();
```

### 4.3 AdaLoraConfig — Adaptive Rank Allocation

AdaLoRA starts all adapters at `initR` and iteratively prunes ranks toward `targetR` using singular value decomposition-based importance scoring. Orthogonal regularization keeps the A/B decomposition valid:

```java
import org.deeplearning4j.peft.config.AdaLoraConfig;

AdaLoraConfig config = AdaLoraConfig.builder()
    .initR(12)                  // starting rank
    .targetR(4)                 // final average rank budget
    .deltaT(200)                // steps between rank updates
    .betaK(0.85)                // EMA coefficient for importance scores
    .totalSteps(10000)
    .orthogonalRegCoeff(0.1)    // coefficient for orthogonal regularization loss
    .targetModules(List.of("q_proj", "v_proj"))
    .build();
```

### 4.4 DyLoraConfig — Dynamic Rank

DyLoRA trains a single adapter that can serve any rank in `[minR, maxR]` by randomly sampling a rank each forward pass. This produces an adapter where you pick the serving rank at inference time based on latency vs. quality trade-offs:

```java
import org.deeplearning4j.peft.config.DyLoraConfig;

DyLoraConfig config = DyLoraConfig.builder()
    .minR(1)
    .maxR(16)
    .loraAlpha(16)
    .targetModules(List.of("q_proj", "v_proj"))
    .build();
```

### 4.5 Other Adapter Configs

| Config Class | Key Parameters |
|--------------|----------------|
| `DoraConfig` | `useDoraDecomposition`, `magnitudeLr` (learning rate for the magnitude vector) |
| `LohaConfig` | Hadamard rank `r`, `lohaAlpha`, `targetModules` |
| `LokrConfig` | Kronecker factor sizes, `alpha`, `targetModules` |
| `IA3Config` | `targetModulesForFeedforward`, scales feedforward activations only |
| `VeraConfig` | `projectionRank`, per-layer scaling vectors; shared random projection matrices are not stored per-layer |
| `LoftQConfig` | `numBits`, `numIter` for quantization error minimization |
| `PrefixTuningConfig` | `numVirtualTokens`, `encoderHiddenSize` |
| `PromptTuningConfig` | `numVirtualTokens`, `promptTuningInit` (RANDOM or TEXT) |
| `AdapterConfig` | `bottleneckDim`, `nonLinearity`, placement (AFTER_ATTN, AFTER_FF, BOTH) |

### 4.6 Multi-Adapter Serving

Multiple adapters may be attached to the same base model simultaneously. Switch between them without reloading the base weights:

```java
PeftModel peft = PeftModelFactory.fromConfig(baseModel, loraConfigA);

// Attach a second adapter while keeping the first
peft.addAdapter("task-b", loraConfigB);

// Switch to task-b for inference
peft.setActiveAdapter("task-b");
INDArray output = peft.output(input);

// Switch back to task-a
peft.setActiveAdapter("task-a");
```

`LoraAdapterCache` manages hot-swapping in multi-tenant serving: the cache holds pre-loaded adapter weight maps and swaps them into the model graph on demand with LRU eviction.

### 4.7 Merge and Unload

After training, merge the adapter into the base weights to get a standard model with no inference overhead:

```java
// Merge adapter weights into base weights, return plain SameDiff
SameDiff mergedModel = peft.mergeAndUnload();

// Save as a normal model — no adapter overhead at inference
mergedModel.save(Paths.get("merged-model.fb"), true);
```

Under the hood, `mergeAndUnload()` calls `LoraLayer.mergeWeights()` on every adapted layer, which adds `(B @ A) * (alpha / r)` to the base weight in place, then removes the adapter variables from the graph.

---

## 5. RL Alignment Trainers

All RL alignment trainers operate on a `SameDiff` policy model and are configured with builder-pattern config objects. They implement a common interface that exposes a `train(MultiDataSetIterator)` method.

### 5.1 GRPOTrainer — Group Relative Policy Optimization

For each prompt, GRPO generates `groupSize` completions, scores them with a reward function, computes z-score normalized advantages within the group, and optimizes a clipped surrogate objective:

```
L = -min(ratio * adv, clip(ratio, 1-eps, 1+eps) * adv) + kl_coeff * KL(pi || ref)
```

where `ratio = pi(token) / pi_old(token)`.

SameDiff placeholders used by GRPOTrainer:

| Placeholder | Shape | Description |
|-------------|-------|-------------|
| `_grpo_tokens` | `[batch, seq]` | Token IDs for generated completions |
| `_grpo_old_log_probs` | `[batch, seq]` | Log probabilities from the old policy |
| `_grpo_advantages` | `[batch]` | Z-score normalized group advantages |
| `_grpo_ref_log_probs` | `[batch, seq]` | Log probabilities from the frozen reference model |

```java
import org.deeplearning4j.rl.GRPOTrainer;
import org.deeplearning4j.rl.config.GRPOConfig;

GRPOConfig grpoConfig = GRPOConfig.builder()
    .groupSize(8)            // completions per prompt
    .clipEpsilon(0.2)        // ratio clipping threshold
    .klCoeff(0.04)           // KL penalty coefficient
    .maxNewTokens(256)
    .temperature(0.9)
    .build();

GRPOTrainer trainer = new GRPOTrainer(peftModel, refModel, rewardFn, grpoConfig);
trainer.train(promptIterator);
```

### 5.2 DPOTrainer — Direct Preference Optimization

DPO eliminates the need for a separate reward model by directly optimizing on preference pairs (chosen vs. rejected):

```
L = -log(sigmoid(beta * (log_pi_chosen - log_pi_ref_chosen
                        - log_pi_rejected + log_pi_ref_rejected)))
```

```java
import org.deeplearning4j.rl.DPOTrainer;
import org.deeplearning4j.rl.config.DPOConfig;

DPOConfig dpoConfig = DPOConfig.builder()
    .beta(0.1)               // temperature for preference scaling
    .labelSmoothing(0.0)
    .lossType(DPOLossType.SIGMOID)
    .build();

DPOTrainer trainer = new DPOTrainer(peftModel, refModel, dpoConfig);
trainer.train(preferenceDataIterator);  // iterator yields (prompt, chosen, rejected) triples
```

### 5.3 PPOTrainer — Proximal Policy Optimization

PPO uses a value network to estimate baselines and clips the policy update ratio:

```java
import org.deeplearning4j.rl.PPOTrainer;
import org.deeplearning4j.rl.config.PPOConfig;

PPOConfig ppoConfig = PPOConfig.builder()
    .clipEpsilon(0.2)
    .valueClipEpsilon(0.2)
    .entropyCoeff(0.01)
    .valueCoeff(0.5)
    .numEpochs(4)            // PPO epochs per rollout
    .miniBatchSize(8)
    .build();

PPOTrainer trainer = new PPOTrainer(policyModel, valueModel, rewardFn, ppoConfig);
trainer.train(promptIterator);
```

### 5.4 DAPOTrainer — Decoupled Clip and Dynamic Sampling Policy Optimization

DAPO decouples the clip range for chosen vs. rejected samples and uses dynamic sampling to focus the rollout budget on difficult prompts:

```java
import org.deeplearning4j.rl.DAPOTrainer;

DAPOConfig dapoConfig = DAPOConfig.builder()
    .clipEpsilonHigh(0.3)    // clip for chosen completions
    .clipEpsilonLow(0.1)     // clip for rejected completions
    .dynamicSamplingTemp(0.8)
    .groupSize(8)
    .build();

DAPOTrainer trainer = new DAPOTrainer(peftModel, refModel, rewardFn, dapoConfig);
```

### 5.5 DrGRPOTrainer — Variance-Reduced GRPO

DrGRPO adds advantage whitening (zero mean, unit variance across the full batch rather than per-group) to reduce gradient variance:

```java
import org.deeplearning4j.rl.DrGRPOTrainer;

DrGRPOConfig config = DrGRPOConfig.builder()
    .groupSize(8)
    .clipEpsilon(0.2)
    .klCoeff(0.02)
    .whitenAdvantages(true)      // global whitening, not per-group z-score
    .build();

DrGRPOTrainer trainer = new DrGRPOTrainer(peftModel, refModel, rewardFn, config);
```

### 5.6 KTOTrainer — Kahneman-Tversky Optimization

KTO applies prospect theory loss functions that model asymmetric human preferences (losses are felt more strongly than gains):

```java
import org.deeplearning4j.rl.KTOTrainer;

KTOConfig config = KTOConfig.builder()
    .beta(0.1)
    .desirabilityThreshold(0.5)
    .undesirabilityWeight(1.0)
    .desirabilityWeight(1.0)
    .build();

KTOTrainer trainer = new KTOTrainer(peftModel, refModel, config);
```

### 5.7 ORPOTrainer — Odds Ratio Preference Optimization

ORPO eliminates the reference model by adding an odds-ratio penalty directly to the SFT cross-entropy loss:

```java
import org.deeplearning4j.rl.ORPOTrainer;

ORPOConfig config = ORPOConfig.builder()
    .lambda(0.1)             // weight of the OR penalty term
    .build();

ORPOTrainer trainer = new ORPOTrainer(peftModel, config);
// No reference model needed
trainer.train(preferenceIterator);
```

### 5.8 SimPOTrainer — Simple Preference Optimization

SimPO removes the reference model and uses length-normalized rewards to avoid verbosity bias:

```java
import org.deeplearning4j.rl.SimPOTrainer;

SimPOConfig config = SimPOConfig.builder()
    .beta(2.5)
    .gamma(1.4)              // reward margin threshold
    .lengthNormalize(true)
    .build();

SimPOTrainer trainer = new SimPOTrainer(peftModel, config);
```

### 5.9 GSPOTrainer — Grouped Sampling Policy Optimization

GSPO uses sampling-based grouping of completions and optimizes group-level preference signals:

```java
import org.deeplearning4j.rl.GSPOTrainer;

GSPOConfig config = GSPOConfig.builder()
    .groupSize(4)
    .samplingTemperature(1.0)
    .klCoeff(0.01)
    .build();
```

### 5.10 VlmGRPOTrainer — GRPO for Vision-Language Models

`VlmGRPOTrainer` extends GRPO for multimodal models. The reward function receives both the generated text and the conditioning image:

```java
import org.deeplearning4j.rl.VlmGRPOTrainer;

VlmGRPOConfig config = VlmGRPOConfig.builder()
    .groupSize(4)
    .clipEpsilon(0.2)
    .imageRewardCoeff(0.5)       // weight for image-conditioned reward
    .textRewardCoeff(0.5)
    .build();

VlmGRPOTrainer trainer = new VlmGRPOTrainer(vlmPeftModel, refModel,
                                             imageTextRewardFn, config);
trainer.train(visionLanguageIterator);
```

### 5.11 RewardModelTrainer

`RewardModelTrainer` trains a scalar reward model from preference data. It supports three reward function implementations:

| Class | Description |
|-------|-------------|
| `CompositeRewardFunction` | Weighted sum of multiple reward signals |
| `RuleBasedRewardFunction` | Hand-crafted heuristics (format compliance, length penalties) |
| `SameDiffRewardFunction` | Trainable neural reward model implemented as a `SameDiff` graph |

```java
import org.deeplearning4j.rl.RewardModelTrainer;

CompositeRewardFunction rewardFn = new CompositeRewardFunction()
    .add(new RuleBasedRewardFunction()
             .penalizeLength(maxLen, penalty)
             .requireJsonFormat(0.5), weight: 0.3)
    .add(new SameDiffRewardFunction(rewardModel), weight: 0.7);

RewardModelTrainer rmTrainer = new RewardModelTrainer(rewardFn, rmConfig);
rmTrainer.train(preferenceIterator);
```

---

## 6. Training Pipelines

### 6.1 SFTTrainingPipeline

`SFTTrainingPipeline` handles the full supervised fine-tuning loop: data loading, loss computation, gradient accumulation, optimizer stepping, checkpointing, and optional evaluation:

```java
import org.deeplearning4j.training.SFTTrainingPipeline;
import org.deeplearning4j.training.config.SFTConfig;

SFTConfig sftConfig = SFTConfig.builder()
    .maxSeqLength(2048)
    .batchSize(4)
    .gradientAccumulationSteps(8)    // effective batch = 4 * 8 = 32
    .numEpochs(3)
    .learningRate(2e-4)
    .warmupSteps(100)
    .saveEveryNSteps(500)
    .checkpointDir(Paths.get("/checkpoints/sft"))
    .evalEveryNSteps(200)
    .build();

SFTTrainingPipeline pipeline = new SFTTrainingPipeline(peftModel, sftConfig);
pipeline.train(trainIterator, evalIterator);
```

Internally, `SFTTrainingPipeline` uses `GradientAccumulator` to split each logical batch into micro-batches, accumulating gradients across `accumulationSteps` calls to `SameDiff.execBackwards()` before applying the optimizer update.

### 6.2 RLAlignmentPipeline

`RLAlignmentPipeline` coordinates the full RLHF loop:

1. Freeze reference model (copy of base policy before RL training)
2. Rollout generation — sample completions from current policy
3. Reward scoring — score with `RewardModelTrainer` or a `RewardFunction`
4. Advantage computation — normalize per group or globally
5. Trainer update — call the configured RL trainer (GRPO, PPO, DPO, etc.)

```java
import org.deeplearning4j.training.RLAlignmentPipeline;
import org.deeplearning4j.training.config.RLAlignmentConfig;

RLAlignmentConfig rlConfig = RLAlignmentConfig.builder()
    .trainer(TrainerType.GRPO)
    .grpoConfig(grpoConfig)
    .rewardFunction(compositeRewardFn)
    .rolloutBatchSize(16)
    .numRlSteps(1000)
    .klCoeff(0.04)
    .saveEveryNSteps(100)
    .checkpointDir(Paths.get("/checkpoints/rl"))
    .build();

RLAlignmentPipeline rlPipeline = new RLAlignmentPipeline(peftModel, rlConfig);
rlPipeline.train(promptIterator);
```

### 6.3 GradientAccumulator

```java
import org.deeplearning4j.training.GradientAccumulator;

GradientAccumulator accumulator = new GradientAccumulator(accumulationSteps /* 8 */);

for (MultiDataSet microBatch : microBatches) {
    accumulator.accumulate(sd, microBatch);   // calls sd.execBackwards() per micro-batch
}
accumulator.applyUpdate(sd, optimizer);       // applies accumulated gradients once
accumulator.reset();
```

### 6.4 CheckpointManager

```java
import org.deeplearning4j.training.CheckpointManager;

CheckpointManager ckpt = CheckpointManager.builder()
    .checkpointDir(Paths.get("/checkpoints"))
    .saveEveryNSteps(500)
    .keepLast(3)                         // delete older checkpoints automatically
    .saveOnBestMetric("eval_loss", true) // also save when validation loss improves
    .build();

// Register with a pipeline or call manually
ckpt.maybeCheckpoint(sd, step, metrics);
```

### 6.5 ContinuedPretrainingWorkflow

For domain-specific continued pretraining, `ContinuedPretrainingWorkflow` mixes a domain dataset with a general-purpose dataset at a configurable ratio:

```java
import org.deeplearning4j.training.ContinuedPretrainingWorkflow;

ContinuedPretrainingWorkflow workflow = ContinuedPretrainingWorkflow.builder()
    .domainIterator(domainIter)
    .generalIterator(generalIter)
    .domainMixRatio(0.7)              // 70% domain, 30% general
    .peftConfig(loraConfig)
    .sftConfig(sftConfig)
    .build();

workflow.run();
```

---

## 7. Mixed Precision Training

### 7.1 FP8ScaleManager

`FP8ScaleManager` tracks per-tensor absolute maximums and updates dynamic scales for FP8 forward and backward passes:

| Tensor direction | FP8 format | Max representable value |
|-----------------|-----------|------------------------|
| Forward activations | E4M3 | 448.0 |
| Backward gradients | E5M2 | 57344.0 |

```java
import org.deeplearning4j.training.precision.FP8ScaleManager;

FP8ScaleManager scaleManager = new FP8ScaleManager(rollingWindowSize /* 16 */);

// Called before each forward pass
INDArray activationScale = scaleManager.getForwardScale("layer_name", activation);

// Called before each backward pass
INDArray gradScale = scaleManager.getBackwardScale("layer_name", gradient);

// Update rolling absmax after each step
scaleManager.update("layer_name", absmax);
```

### 7.2 LossScaler — Dynamic Loss Scaling

`LossScaler` prevents underflow in FP16/BF16 training by scaling the loss up before the backward pass and scaling gradients back down before the optimizer step:

```java
import org.deeplearning4j.training.precision.LossScaler;

LossScaler scaler = new LossScaler(
    initialScale   /* 65536.0 */,
    scaleWindow    /* 2000 */,    // double scale every 2000 steps without overflow
    scaleFactor    /* 2.0 */      // halve scale on NaN/Inf gradient detection
);

// Scale loss before backward
SDVariable scaledLoss = scaler.scale(loss);
sd.execBackwards(scaledLoss);

// Unscale and check for NaN/Inf before optimizer step
boolean validStep = scaler.unscaleAndCheck(sd.getGradients());
if (validStep) {
    optimizer.step();
}
scaler.update(validStep);
```

### 7.3 Gradient Checkpointing

Gradient checkpointing trades compute for memory by not storing all intermediate activations during the forward pass. Activations are recomputed during the backward pass as needed:

```java
// Enable on a SameDiff model before training
sd.enableGradientCheckpointing(true);

// Fine-grained control: checkpoint every N layers
sd.setGradientCheckpointingInterval(4);
```

---

## 8. 8-bit Adam Optimizer

`Adam8bit` stores the first and second moment vectors (`m` and `v`) in INT8 with per-block absolute maximum quantization, reducing optimizer state memory by approximately 4x:

| Precision | Memory for 7B parameter model |
|-----------|-------------------------------|
| BF16 Adam (standard) | ~56 GB optimizer state |
| Adam8bit | ~14 GB optimizer state |

Default block size is 2048 elements per quantization block (`DEFAULT_BLOCK_SIZE = 2048`).

```java
import org.nd4j.linalg.learning.config.Adam8bit;

// Use Adam8bit as a drop-in replacement for Adam in TrainingConfig
TrainingConfig config = TrainingConfig.builder()
    .updater(new Adam8bit(
        2e-4,      // learning rate
        0.9,       // beta1
        0.999,     // beta2
        1e-8,      // epsilon
        2048       // block size (default)
    ))
    .dataSetFeatureMapping("input_ids")
    .dataSetLabelMapping("labels")
    .lossVariables("lm_loss")
    .build();
```

Internally, `Adam8bitUpdater` performs per-block updates:

1. Dequantize INT8 `m` and `v` blocks to float using stored absmax values
2. Apply the standard Adam moment update in float
3. Compute the parameter update
4. Requantize updated `m` and `v` back to INT8

`BlockQuantizationUtils` provides the low-level INT8 quantize/dequantize operations used by the updater.

---

## 9. Knowledge Distillation

`DistillationTrainer` trains a smaller student model to mimic a larger teacher model. Three loss components are available and can be combined:

| Loss Class | What It Minimizes |
|------------|------------------|
| `DistillationKLLoss` | KL divergence between teacher and student output distributions at temperature T |
| `AttentionDistillationLoss` | MSE between teacher and student attention weight matrices |
| `FeatureDistillationLoss` | MSE between intermediate hidden state representations |

```java
import org.deeplearning4j.training.DistillationTrainer;
import org.deeplearning4j.training.config.DistillationConfig;

DistillationConfig distConfig = DistillationConfig.builder()
    .temperature(4.0)                           // soften teacher probabilities
    .klLossWeight(0.7)                          // weight for output KL loss
    .attentionLossWeight(0.2)                   // weight for attention map loss
    .featureLossWeight(0.1)                     // weight for hidden state loss
    .teacherLayersForFeature(List.of(8, 16, 24))   // teacher layers to distill from
    .studentLayersForFeature(List.of(2,  4,  6))   // corresponding student layers
    .hardLabelWeight(0.1)                       // weight for ground-truth cross-entropy
    .build();

DistillationTrainer distTrainer = new DistillationTrainer(
    teacherModel,    // large frozen teacher SameDiff
    studentModel,    // smaller student SameDiff (may be a PeftModel)
    distConfig
);

distTrainer.train(trainIterator, numEpochs);
```

The `temperature` parameter scales logits before the softmax: higher values produce softer probability distributions that expose more information about inter-class similarities.

Alternatively, use the `SameDiff` extension:

```java
studentSd.distillFrom(teacherSd, distConfig);
studentSd.fit(trainIter, numEpochs);
```

---

## 10. Dataset Curation Toolkit

The dataset curation package provides 20 utilities for preparing high-quality training data.

### 10.1 Deduplication

`TextDeduplicator` uses MinHash Locality-Sensitive Hashing to detect and remove near-duplicate documents efficiently:

```java
import org.deeplearning4j.data.TextDeduplicator;
import org.deeplearning4j.data.MinHasher;

MinHasher hasher = new MinHasher(numHashFunctions /* 128 */, ngramSize /* 5 */);
TextDeduplicator deduplicator = new TextDeduplicator(hasher, similarityThreshold /* 0.85 */);

List<String> deduplicated = deduplicator.deduplicate(documents);
System.out.printf("Removed %d duplicates%n",
    documents.size() - deduplicated.size());
```

### 10.2 Benchmark Contamination Removal

`NGramDecontaminator` removes documents containing n-gram overlaps with benchmark evaluation sets:

```java
import org.deeplearning4j.data.NGramDecontaminator;

NGramDecontaminator decon = new NGramDecontaminator(
    benchmarkDocuments,
    ngramSize /* 13 */,
    overlapThreshold /* 0.2 */   // fraction of document n-grams that must overlap
);

List<String> clean = decon.filter(trainingDocuments);
```

### 10.3 Quality Filtering

`TextQualityFilter` applies heuristics to remove low-quality text:

```java
import org.deeplearning4j.data.TextQualityFilter;

TextQualityFilter filter = TextQualityFilter.builder()
    .minTokens(50)
    .maxTokens(8192)
    .maxSymbolToWordRatio(0.1)
    .maxLineRepetitionFraction(0.3)
    .minMeanWordLength(3.0)
    .removeHtml(true)
    .build();

List<String> quality = filter.filter(documents);
```

### 10.4 Curriculum Learning

`CurriculumIterator` wraps a dataset iterator and yields examples in order of difficulty, starting with easier examples and progressing to harder ones:

```java
import org.deeplearning4j.data.CurriculumIterator;
import org.deeplearning4j.data.DifficultyScorer;

DifficultyScorer scorer = new PerplexityDifficultyScorer(referenceModel);

CurriculumIterator curriculumIter = new CurriculumIterator(
    baseIterator,
    scorer,
    CurriculumSchedule.LINEAR,    // LINEAR | EXPONENTIAL | STEP
    numWarmupSteps /* 1000 */
);
```

### 10.5 Sequence Packing

`SequencePacker` bin-packs variable-length sequences into fixed-size windows, minimizing padding tokens and maximizing GPU utilization:

```java
import org.deeplearning4j.data.SequencePacker;
import org.deeplearning4j.data.PackedSequence;

SequencePacker packer = new SequencePacker(maxSeqLength /* 2048 */);
List<PackedSequence> packed = packer.pack(sequences);

// PackedSequence contains the concatenated tokens and a position-ids array
// to distinguish document boundaries within the packed window
```

### 10.6 Domain Mixing

`WeightedDataMixer` samples from multiple domain iterators with configurable weights:

```java
import org.deeplearning4j.data.WeightedDataMixer;

WeightedDataMixer mixer = WeightedDataMixer.builder()
    .addSource("code",    codeIterator,    0.30)
    .addSource("math",    mathIterator,    0.20)
    .addSource("general", generalIterator, 0.50)
    .build();
```

### 10.7 Padding-Free Batching

`LengthBucketingIterator` groups sequences into length buckets before batching, reducing within-batch padding. `PaddingFreeBatch` represents a batch where all padding has been eliminated by packing:

```java
import org.deeplearning4j.data.LengthBucketingIterator;

LengthBucketingIterator bucketIter = new LengthBucketingIterator(
    baseIterator,
    bucketBoundaries /* new int[]{128, 256, 512, 1024, 2048} */,
    batchSize /* 8 */
);
```

### 10.8 Instruction Formatting and Chat Templates

`InstructionDataFormatter` and `ChatTemplate` apply model-specific chat templates to raw instruction datasets:

```java
import org.deeplearning4j.data.InstructionDataFormatter;
import org.deeplearning4j.data.ChatTemplate;

ChatTemplate llamaTemplate = ChatTemplate.forModel("llama-3");

InstructionDataFormatter formatter = new InstructionDataFormatter(llamaTemplate);
List<String> formatted = formatter.format(rawInstructions);
// Applies system prompt, user/assistant turn markers, and EOS tokens
```

### 10.9 Stratified Splitting

`StratifiedSplitter` creates train/validation/test splits that preserve category distributions:

```java
import org.deeplearning4j.data.StratifiedSplitter;

StratifiedSplitter splitter = new StratifiedSplitter(
    trainFraction /* 0.80 */,
    valFraction   /* 0.10 */,
    testFraction  /* 0.10 */,
    seed          /* 42 */
);

StratifiedSplitter.Split split = splitter.split(labeledDocuments, labels);
// split.train(), split.validation(), split.test()
```

---

## 11. Transfer Learning API

The classical transfer learning API (`TransferLearning` / `TransferLearningHelper`) applies to `MultiLayerNetwork` and `ComputationGraph` models. For large generative models, prefer the `PeftModel` API described in sections 3–4.

### 11.1 Freeze Layers by Name Pattern

```java
import org.deeplearning4j.nn.transferlearning.TransferLearning;

ComputationGraph model = new TransferLearning.GraphBuilder(pretrainedGraph)
    .fineTuneConfiguration(new FineTuneConfiguration.Builder()
        .updater(new Adam(1e-4))
        .build())
    .setFeatureExtractor("encoder_layer_11")   // freeze through this layer
    .removeVertexKeepConnections("output_head")
    .addLayer("output_head",
        new OutputLayer.Builder()
            .nIn(768).nOut(numClasses)
            .activation(Activation.SOFTMAX)
            .build(),
        "encoder_layer_11")
    .build();
```

### 11.2 SameDiff-Based Freezing

For `SameDiff` models, freeze parameters by converting them to constants:

```java
// Freeze the embedding layer
sd.convertToConstant(sd.getVariable("token_embeddings"));
sd.convertToConstant(sd.getVariable("position_embeddings"));

// All remaining VARIABLE-type parameters will be trained
sd.fit(trainIter, numEpochs);

// Unfreeze later for a second fine-tuning phase
sd.convertToVariable(sd.getVariable("token_embeddings"));
```

### 11.3 TransferLearningHelper — Pre-computed Feature Cache

For large frozen feature extractors, pre-computing activations at the freeze boundary dramatically reduces training time:

```java
import org.deeplearning4j.nn.transferlearning.TransferLearningHelper;

// Featurize dataset once (runs forward through frozen layers)
TransferLearningHelper helper = new TransferLearningHelper(pretrainedNet, "block5_pool");

List<DataSet> featurized = new ArrayList<>();
while (trainIter.hasNext()) {
    featurized.add(helper.featurize(trainIter.next()));
}

// Train only the head on cached features
for (int epoch = 0; epoch < numEpochs; epoch++) {
    for (DataSet batch : featurized) {
        helper.fitFeaturized(batch);
    }
}
```

---

## 12. Configuration Reference

### LoraConfig

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `r` | `int` | 8 | Adapter rank |
| `loraAlpha` | `int` | 8 | Scaling numerator; effective_scale = loraAlpha / r |
| `loraDropout` | `double` | 0.0 | Dropout on adapter input |
| `targetModules` | `List<String>` | `[]` | Layer name substrings to target |
| `biasMode` | `BiasMode` | `NONE` | Which bias vectors to train |
| `initLoraWeights` | `InitLoraWeights` | `KAIMING` | Initialization strategy |

### GRPOConfig

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `groupSize` | `int` | 8 | Completions generated per prompt |
| `clipEpsilon` | `double` | 0.2 | Policy ratio clipping threshold |
| `klCoeff` | `double` | 0.04 | KL divergence penalty weight |
| `maxNewTokens` | `int` | 256 | Maximum tokens generated per completion |
| `temperature` | `double` | 1.0 | Sampling temperature |

### SFTConfig

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `maxSeqLength` | `int` | 2048 | Maximum input sequence length |
| `batchSize` | `int` | 8 | Micro-batch size per step |
| `gradientAccumulationSteps` | `int` | 1 | Steps before optimizer update |
| `numEpochs` | `int` | 3 | Training epochs |
| `learningRate` | `double` | 2e-4 | Peak learning rate |
| `warmupSteps` | `int` | 0 | Linear warmup steps |
| `saveEveryNSteps` | `int` | 500 | Checkpoint frequency |

### FP8ScaleManager

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `rollingWindowSize` | `int` | 16 | Steps in the rolling absmax window |
| `forwardFormat` | `FP8Format` | `E4M3` | FP8 format for forward activations |
| `backwardFormat` | `FP8Format` | `E5M2` | FP8 format for backward gradients |
| `forwardMaxValue` | `double` | 448.0 | Saturation value for E4M3 |
| `backwardMaxValue` | `double` | 57344.0 | Saturation value for E5M2 |

### LossScaler

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `initialScale` | `double` | 65536.0 | Starting loss scale |
| `scaleWindow` | `int` | 2000 | Steps before doubling the scale |
| `scaleFactor` | `double` | 2.0 | Factor used to halve (on overflow) or double |

### Adam8bit

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `lr` | `double` | 1e-3 | Learning rate |
| `beta1` | `double` | 0.9 | First moment decay |
| `beta2` | `double` | 0.999 | Second moment decay |
| `epsilon` | `double` | 1e-8 | Numerical stability constant |
| `blockSize` | `int` | 2048 | Elements per quantization block |

### DistillationConfig

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `temperature` | `double` | 4.0 | Softmax temperature for teacher logits |
| `klLossWeight` | `double` | 1.0 | Weight for KL divergence loss component |
| `attentionLossWeight` | `double` | 0.0 | Weight for attention map distillation |
| `featureLossWeight` | `double` | 0.0 | Weight for hidden state distillation |
| `hardLabelWeight` | `double` | 0.0 | Weight for ground-truth cross-entropy |

---

## See Also

- [SameDiff Training](../../nd4j/samediff/training.md) — base training loop and `TrainingConfig`
- [Transfer Learning](../nn/transfer-learning.md) — `TransferLearning.Builder` for `MultiLayerNetwork` / `ComputationGraph`
- [OmniHub Model Hub](../../omnihub/overview.md) — downloading pretrained base models to fine-tune
- [SameDiff Serialization](../../nd4j/samediff/serialization.md) — saving and loading models and adapter weights

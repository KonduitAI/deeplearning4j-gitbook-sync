---
title: "Troubleshooting"
description: "Common training problems and solutions — NaN loss, slow convergence, overfitting, memory errors, and debugging tips"
---

# Troubleshooting

Neural networks are sensitive to configuration. A poorly chosen hyperparameter can cause training to diverge, stall, or silently produce a bad model. This guide covers the most common failure modes, their root causes, and concrete fixes.

---

## Debugging Checklist

Run through this list before investigating specific symptoms:

1. **Data normalization**: Are inputs normalized to roughly `[-1, 1]` or `[0, 1]`, or standardized to zero mean / unit variance? Unnormalized inputs are one of the most common causes of divergence.
2. **Label correctness**: Are one-hot labels actually one-hot? Are class indices within `[0, numClasses)`?
3. **Shuffling**: Is training data shuffled before each epoch? Unshuffled data (all examples of class 0, then class 1, ...) causes very noisy or diverging loss.
4. **Same normalization at inference**: Is the exact same normalization applied to test data that was used for training?
5. **Weight initialization**: Are you using `WeightInit.XAVIER` for tanh/sigmoid, or `WeightInit.RELU` for relu/leakyrelu?
6. **Output layer pairing**: Is the output activation function consistent with the loss function? (See Activation/Loss pairings below.)
7. **Learning rate sanity**: Does the loss decrease at all in the first few iterations? If not, the learning rate may be off by several orders of magnitude.
8. **Gradient norm check**: Add a `ScoreIterationListener(10)` and watch the first 50 iterations. A rapidly increasing or immediately NaN loss is usually a learning rate or data problem.

---

## NaN or Infinity in the Loss

NaN (Not a Number) or `Infinity` in the loss is the most common catastrophic failure. Training cannot continue once this occurs.

### Cause 1: Learning Rate Too High

The most common cause. When the learning rate is too large, parameter updates overshoot, activations explode, and the loss overflows.

**Fix**: Reduce the learning rate by an order of magnitude and retry. Start from `1e-3` (Adam) or `1e-2` (SGD) and tune from there.

```java
// Adam with a conservative default
.updater(new Adam(1e-4))

// SGD — generally needs a lower LR than adaptive methods
.updater(new Sgd(1e-2))
```

### Cause 2: Gradient Explosion

Gradients grow exponentially during backprop, especially in deep or recurrent networks.

**Fix**: Add gradient clipping.

```java
new NeuralNetConfiguration.Builder()
    .gradientNormalization(GradientNormalization.ClipElementWiseAbsoluteValue)
    .gradientNormalizationThreshold(1.0)
    // or:
    // .gradientNormalization(GradientNormalization.ClipL2PerParamType)
    // .gradientNormalizationThreshold(1.0)
    ...
```

For RNNs, `ClipElementWiseAbsoluteValue` with threshold 1.0 is a common default.

### Cause 3: Mismatched Output Activation and Loss

Applying `log` inside a loss function to values that can be zero or negative produces NaN.

| Loss Function | Required Output Activation |
|---|---|
| `MCXENT` (multi-class cross entropy) | `SOFTMAX` |
| `XENT` (binary cross entropy) | `SIGMOID` |
| `MSE` | `IDENTITY` (or any bounded activation) |
| `NEGATIVELOGLIKELIHOOD` | `SOFTMAX` |
| `COSINE_PROXIMITY` | `IDENTITY` (normalized inputs) |

**Fix**: Ensure the output layer's activation matches its loss function. Example:

```java
new OutputLayer.Builder(LossFunctions.LossFunction.MCXENT)
    .activation(Activation.SOFTMAX)
    .nIn(256).nOut(10)
    .build()
```

### Cause 4: Unnormalized Input Data

Inputs with very large magnitudes (e.g., pixel values in range [0, 255] rather than [0, 1]) cause activations and gradients to overflow.

**Fix**: Normalize all inputs. For images:

```java
DataNormalization scaler = new ImagePreProcessingScaler(0, 1);
scaler.fit(trainIterator);
trainIterator.setPreProcessor(scaler);
testIterator.setPreProcessor(scaler);
```

For tabular data:

```java
NormalizerStandardize normalizer = new NormalizerStandardize();
normalizer.fit(trainIterator);
trainIterator.setPreProcessor(normalizer);
testIterator.setPreProcessor(normalizer);
ModelSerializer.addNormalizerToModel(modelFile, normalizer);  // save with model
```

### Cause 5: Bad Weight Initialization

Very large initial weights cause the activation function to saturate immediately, producing NaN gradients.

**Fix**: Use a standard initializer:

```java
.weightInit(WeightInit.XAVIER)        // general purpose, tanh/sigmoid
.weightInit(WeightInit.RELU)          // for relu/leakyrelu networks
.weightInit(WeightInit.XAVIER_UNIFORM) // uniform variant of Xavier
```

### Cause 6: Numerical Underflow (Deep Networks)

In very deep networks, gradients may underflow to zero (not NaN, but effectively zero gradient). This is distinct from NaN but leads to dead training.

**Fix**: Use batch normalization between layers; use residual connections; prefer `RELU` over `SIGMOID`/`TANH`.

---

## Slow or No Convergence

### Loss Decreases Very Slowly

- Learning rate is too low. Increase it by 5–10× and retry.
- Gradient updates are too small. Check the Update:Parameter ratio in the training UI — it should be around `1e-3`. If it is `1e-5` or smaller, raise the learning rate.
- Data not being shuffled between epochs. Call `trainIterator.reset()` before each epoch if your iterator does not reset automatically.

### Loss Decreases Initially Then Plateaus

- The model may have reached a local or saddle point with SGD. Switch to an adaptive optimizer: Adam is a good default.
- Learning rate schedule may be decaying too fast. Review the schedule.
- Network capacity may be too low for the task (underfitting). Increase the number of layers or neurons.

### Training Loss Oscillates Without Decreasing

- Data is not shuffled — minibatches are non-i.i.d. Fix shuffling.
- Minibatch size is very small (1–4). Increase to 32 or 64.
- Learning rate is slightly too high. Reduce by 2–5×.

### Learning Rate Selection Guide

As a starting point, try:

| Updater | Typical starting LR |
|---|---|
| Adam, Nadam | `1e-3` |
| RMSProp | `1e-3` |
| Nesterovs (momentum) | `1e-2` |
| AdaGrad | `1e-2` |
| SGD (vanilla) | `1e-1` to `1e-2` |

Use the training UI's Score vs. Iteration chart and Update:Parameter ratio chart to guide further adjustment.

---

## Overfitting

Overfitting occurs when training loss keeps decreasing while validation loss stagnates or increases. The model has memorized the training set.

### Symptoms

- Training accuracy >> validation accuracy.
- Training loss continues falling while validation loss rises after an initial decrease.

### Solutions

**Increase regularization:**

```java
new NeuralNetConfiguration.Builder()
    .l2(1e-4)             // L2 weight decay (most common)
    .l1(1e-5)             // L1 sparsity regularization (optional)
    .dropOut(0.5)         // Standard dropout (50% retention)
    ...
```

**Use early stopping** (see the Early Stopping guide) to halt training at the validation optimum.

**Add more data or data augmentation.** More training examples are the most reliable cure for overfitting.

**Reduce model capacity.** Fewer layers or smaller layer sizes may generalize better on small datasets.

**Batch normalization.** Acts as a form of regularization in addition to stabilizing training.

---

## Underfitting

The model fails to achieve good performance on either training or validation data.

### Symptoms

- Training loss is high and does not decrease meaningfully.
- Training and validation accuracy are both poor and similar to random chance.

### Solutions

- Increase model capacity: more layers, larger hidden sizes.
- Train for more epochs.
- Reduce regularization: lower L1/L2 coefficients; reduce dropout rate.
- Increase the learning rate or use a better optimizer.
- Verify that the task is actually learnable from the features provided.

---

## Out-of-Memory Errors

### Java Heap OOM

```
java.lang.OutOfMemoryError: Java heap space
```

- Increase heap: `-Xmx8g` (or more) in JVM arguments.
- Reduce minibatch size.
- Avoid holding large INDArrays in long-lived Java references. DL4J tensors are typically allocated off-heap; holding the Java wrapper keeps it from being released.

### ND4J Off-Heap (Native) OOM

```
Cannot allocate [X] bytes from [deallocator-based memory]
```

ND4J allocates GPU/native memory off-heap. This is separate from Java heap.

- Set `-Dorg.bytedeco.javacpp.maxbytes=8G` (or appropriate value).
- Enable workspace memory management (default in DL4J; see Workspace Exceptions below for pitfalls).
- Call `Nd4j.getMemoryManager().togglePeriodicGc(true)` if memory is accumulating.

### GPU OOM (CUDA)

```
CUDA error 2: out of memory at ...
```

- Reduce minibatch size.
- Reduce model size.
- Free unused models or tensors: `net.close()` for ComputationGraph.
- Run `Nd4j.getWorkspaceManager().destroyAllWorkspacesForCurrentThread()` if using workspaces manually.

---

## Workspace Exceptions

Workspaces are DL4J's memory management system for avoiding repeated allocation during training. Misuse leads to exceptions.

### "Array is not attached to any workspace"

This usually occurs when you store an `INDArray` that was allocated inside a workspace, then try to use it after the workspace is closed. The underlying memory has been reused.

**Fix**: Detach the array before storing it outside the training loop:

```java
// Inside a fit() or output() call, if you need to keep the result:
INDArray output = net.output(input);
INDArray detached = output.detach();  // copies to non-workspace memory
// use detached, not output
```

### "Cannot allocate array: workspace is closed"

You are trying to allocate inside a workspace that has already been closed.

**Fix**: Do not use `try-with-resources` on workspaces if the code that uses the arrays outlives the block. Use explicit `.open()` and `.close()` only if you control the full lifecycle.

### Workspace Memory Growing Without Bound

If off-heap memory grows continuously, a workspace may not be closing properly.

**Fix**: Ensure symmetric open/close pairs. If using `LayerWorkspaceMgr` directly, call `close()` in a `finally` block.

---

## Performance Issues

### Training Is Slower Than Expected

- **ETL bottleneck**: If the training loop stalls waiting for data, the data pipeline is the bottleneck. Use `AsyncDataSetIterator` (applied automatically in most cases) or pre-load data. Monitor ETL time with `PerformanceListener`.
- **Missing CuDNN**: For GPU training, CuDNN dramatically accelerates convolutions and LSTMs. Ensure `cudnn` is on the classpath and the correct backend is loaded.
- **Wrong backend**: Confirm CUDA backend is active: `System.out.println(Nd4j.getBackend())`.
- **Workspace disabled**: If workspaces are disabled (e.g., `WorkspaceMode.NONE`), allocation overhead increases. Only disable for debugging.

```java
// Add performance listener to diagnose ETL vs compute time
net.setListeners(new PerformanceListener(10, true));
```

### GPU Utilization Is Low

- Minibatch too small: GPUs need large batches to saturate parallelism. Try 64, 128, or 256.
- ETL too slow: feeding the GPU faster requires optimized data loading (pre-fetched, pre-processed batches).
- Model is too small: tiny models complete in microseconds; GPU kernel launch overhead dominates.

---

## Common Error Messages and Solutions

### "Cannot serialize class ..."

Custom layers must be registered for JSON serialization. See the Custom Layers guide.

### "Shape mismatch: ..."

Typically an `nIn`/`nOut` mismatch between adjacent layers. Use `.setInputType(InputType.feedForward(N))` on the network builder to let DL4J infer and validate shapes automatically.

```java
new NeuralNetConfiguration.Builder()
    ...
    .list()
    .layer(new DenseLayer.Builder().nOut(256).build())
    .layer(new OutputLayer.Builder().nOut(10).build())
    .setInputType(InputType.feedForward(784))  // infers nIn automatically
    .build();
```

### "No updater state loaded"

When loading a model to continue training, you must load the updater state (e.g., momentum history, Adam m/v vectors). If the model was saved without updater state, it can only be used for inference.

```java
// Save WITH updater state (default)
net.save(new File("model.zip"));

// Load WITH updater state for continued training
MultiLayerNetwork restored = MultiLayerNetwork.load(new File("model.zip"), true);

// Load WITHOUT updater state (inference only, smaller file)
MultiLayerNetwork restored = MultiLayerNetwork.load(new File("model.zip"), false);
```

### "Minibatch size mismatch"

Recurrent networks require all sequences in a minibatch to be the same length, or use masking. If using `RecordReaderDataSetIterator` for sequence data, configure masking:

```java
new RecordReaderDataSetIterator.Builder(...)
    .maxNumBatches(numBatches)
    .sequenceAlignmentMode(SequenceRecordReaderDataSetIterator.AlignmentMode.ALIGN_END)
    .build();
```

### NaN from a Specific Layer

Narrow down which layer is producing NaN by activating per-layer output inspection:

```java
net.feedForward(input, false).forEach((key, value) ->
    System.out.println(key + " contains NaN: " +
        Transforms.isNaN(value).castTo(DataType.INT).sumNumber()));
```

The first layer that produces NaN is where the problem originates. Common causes per layer type:

- `BatchNormalization`: can produce NaN if all values in a minibatch are identical (zero variance). Add a small epsilon: `.eps(1e-5)`.
- `LSTM`: exploding gradients from long sequences — add gradient clipping.
- `OutputLayer (MCXENT)`: non-softmax output feeding a cross-entropy loss.

---

## Summary: First-Response Protocol

When something goes wrong, try in order:

1. Add `new ScoreIterationListener(1)` and watch the first 10 iterations.
2. Check data normalization. Visualize a batch.
3. Verify output activation / loss function pairing.
4. Reduce learning rate by 10× and try again.
5. Enable the training UI and inspect the Update:Parameter ratio and activation standard deviations.
6. Add gradient clipping if activations or gradients are exploding.
7. Check available memory and reduce minibatch size if OOM.
8. Run a gradient check if you have a custom layer (traditional approach).

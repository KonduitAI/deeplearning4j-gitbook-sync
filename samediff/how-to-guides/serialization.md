---
title: "SameDiff Serialization"
description: "Saving and loading SameDiff graphs in FlatBuffers format"
---

# SameDiff Serialization

SameDiff graphs — including their structure, variables, constants, and trained parameters — can be saved to and loaded from files using the FlatBuffers binary format. This enables model deployment, checkpointing during training, and sharing trained models.

## Saving a SameDiff Graph

### save()

Save the complete graph including all variables, constants, and parameter values:

```java
import org.nd4j.autodiff.samediff.SameDiff;
import java.io.File;

// Save with updater state (for resuming training)
sd.save(new File("model.fb"), true);

// Save without updater state (smaller file, for inference only)
sd.save(new File("model.fb"), false);
```

The second argument controls whether to include the updater state (momentum buffers, adaptive learning rate accumulators, etc.). Include it if you plan to resume training; omit it for inference-only deployment.

### asFlatGraph()

Convert the graph to a FlatBuffers byte buffer in memory (useful for embedding in other formats or sending over the network):

```java
import java.nio.ByteBuffer;

ByteBuffer buffer = sd.asFlatGraph(true);  // true = include updater state
```

### asFlatPrint()

Get a human-readable string representation of the graph (for debugging):

```java
String graphString = sd.asFlatPrint();
System.out.println(graphString);
```

This prints the graph structure, variable names, shapes, and operation types.

## Loading a SameDiff Graph

### load()

Load a previously saved graph:

```java
// Load with updater state (for resuming training)
SameDiff sd = SameDiff.load(new File("model.fb"), true);

// Load without updater state (for inference)
SameDiff sd = SameDiff.load(new File("model.fb"), false);
```

### fromFlatGraph()

Load from a FlatBuffers byte buffer:

```java
import org.nd4j.autodiff.samediff.SameDiff;
import java.nio.ByteBuffer;

SameDiff sd = SameDiff.fromFlatGraph(byteBuffer);
```

## What Gets Saved

| Component | Saved | Notes |
|-----------|-------|-------|
| Graph structure (ops, connections) | Always | The computation graph topology |
| VARIABLE values (weights, biases) | Always | Trainable parameters |
| CONSTANT values | Always | Non-trainable stored values |
| PLACEHOLDER definitions | Always | Shape and type info (not values) |
| ARRAY definitions | Always | Shape and type info (not values — computed at runtime) |
| Updater state | Optional | Momentum/adaptive rate buffers. Only if `saveUpdater=true` |
| TrainingConfig | With updater | Optimizer settings |

## Training Checkpoints

Save periodic checkpoints during training for recovery:

```java
for (int epoch = 0; epoch < numEpochs; epoch++) {
    sd.fit(trainIter, 1);
    trainIter.reset();

    // Save checkpoint every 5 epochs (with updater state)
    if (epoch % 5 == 0) {
        sd.save(new File("checkpoint_epoch_" + epoch + ".fb"), true);
    }
}

// Save final model (without updater state — smaller)
sd.save(new File("final_model.fb"), false);
```

Resume training from a checkpoint:

```java
SameDiff sd = SameDiff.load(new File("checkpoint_epoch_10.fb"), true);

// Re-set training config if needed
sd.setTrainingConfig(TrainingConfig.builder()
    .updater(new Adam(1e-4))    // can change learning rate
    .dataSetFeatureMapping("input")
    .dataSetLabelMapping("label")
    .build());

// Continue training
sd.fit(trainIter, remainingEpochs);
```

## Interop with Model Import

SameDiff is also the target format for model import from other frameworks. When you import a TensorFlow or ONNX model, the result is a `SameDiff` graph:

```java
// TensorFlow import
import org.nd4j.imports.graphmapper.tf.TFGraphMapper;
SameDiff sd = TFGraphMapper.importGraph(new File("frozen_model.pb"));

// ONNX import
import org.nd4j.imports.graphmapper.onnx.OnnxGraphMapper;
SameDiff sd = OnnxGraphMapper.importGraph(new File("model.onnx"));

// Run inference on the imported model
Map<String, INDArray> placeholders = new HashMap<>();
placeholders.put("input:0", inputData);
INDArray output = sd.outputSingle(placeholders, "output:0");

// Save as FlatBuffers for faster subsequent loading
sd.save(new File("converted_model.fb"), false);
```

This workflow (import once, save as FlatBuffers, load FlatBuffers for serving) avoids repeated parsing of the original model format.

## File Format Details

SameDiff uses [FlatBuffers](https://google.github.io/flatbuffers/) as its serialization format:

- **Binary format**: Compact, fast to serialize/deserialize
- **No schema evolution issues**: Forward and backward compatible
- **Zero-copy reads**: FlatBuffers can be read directly from the buffer without unpacking
- **Cross-platform**: Same file works on any OS/architecture

Typical file sizes depend on model complexity:
- Simple MLP (784→256→10): ~1-2 MB
- ResNet-18: ~45 MB
- Large transformer: 100+ MB

## Best Practices

1. **Save without updater state for deployment** — reduces file size significantly (updater state can be 2-3x the model parameters for Adam)
2. **Save with updater state for checkpoints** — allows seamless training resumption
3. **Convert imported models to FlatBuffers** — much faster to load than parsing TF/ONNX format each time
4. **Version your saved models** — include epoch/date in filenames for traceability
5. **Verify after loading** — run a sample inference to confirm the loaded model produces expected results

```java
// Quick verification after loading
SameDiff sd = SameDiff.load(new File("model.fb"), false);
INDArray testInput = Nd4j.rand(DataType.FLOAT, 1, inputSize);
Map<String, INDArray> ph = Collections.singletonMap("input", testInput);
INDArray output = sd.outputSingle(ph, "predictions");
System.out.println("Output shape: " + Arrays.toString(output.shape()));
System.out.println("Output: " + output);
```

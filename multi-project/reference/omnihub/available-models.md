---
title: "Available Models"
description: "Pretrained models available through OmniHub — DL4J and SameDiff model catalog"
---

# Available Models

OmniHub currently hosts models in two frameworks: DL4J (ComputationGraph format) and SameDiff (FlatBuffers format). All models are accessed through the `Pretrained` class or directly via `OmniHubUtils`.

---

## DL4J Models

DL4J models are loaded as `ComputationGraph` objects. They are stored as zip archives in the `dl4j/` directory of the zoo repository.

### VGG19 (no top)

| Property | Value |
|----------|-------|
| Method | `Pretrained.dl4j().vgg19noTop(forceDownload)` |
| File | `vgg19_weights_tf_dim_ordering_tf_kernels_notop.zip` |
| Return type | `ComputationGraph` |
| Source | Keras Applications `VGG19` (TensorFlow dimension ordering) |
| Input shape | `[batch, 3, 224, 224]` (channels-first) |
| Output | Feature map from the last pooling layer, shape `[batch, 512, 7, 7]` |
| Pretrained dataset | ImageNet |

This model is the VGG19 convolutional base without the fully connected classifier head (hence "no top"). It is suitable as a feature extractor or as a base for transfer learning. Weights were converted from the Keras Applications VGG19 checkpoint using TensorFlow channel ordering (`tf_dim_ordering`).

**Loading example:**

```java
import org.eclipse.deeplearning4j.omnihub.models.Pretrained;
import org.deeplearning4j.nn.graph.ComputationGraph;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.ndarray.INDArray;

ComputationGraph vgg19 = Pretrained.dl4j().vgg19noTop(false);

// Input: one RGB image at 224x224
INDArray input = Nd4j.rand(new int[]{1, 3, 224, 224});
INDArray[] features = vgg19.output(input);

System.out.println(features[0].shapeInfoToString());
// [1, 512, 7, 7]
```

**Preprocessing.** VGG19 expects pixel values in `[0, 255]` range with per-channel mean subtraction (ImageNet means: R=103.939, G=116.779, B=123.68 in BGR order). The Keras Applications preprocessing function is the reference. Apply the same preprocessing to your inputs when using this model for feature extraction.

---

## SameDiff Models

SameDiff models are loaded as `SameDiff` objects. They are stored as FlatBuffers files (`.fb`) in the `samediff/` directory of the zoo repository. Each was converted to SameDiff from an ONNX checkpoint or a PyTorch torchvision model via ONNX export.

### AgeGoogLeNet

| Property | Value |
|----------|-------|
| Method | `Pretrained.samediff().ageGooglenet(forceDownload)` |
| File | `age_googlenet.fb` |
| Return type | `SameDiff` |
| Source | ONNX Model Zoo — `age_googlenet.onnx` |
| Original source | [ONNX Models on GitHub](https://github.com/onnx/models) |
| Input name | `data` |
| Input shape | `[1, 3, 224, 224]` (NCHW) |
| Output name | `loss3/loss3_Y` |
| Output shape | `[1, 8]` |
| Output classes | 8 age groups |
| Pretrained dataset | Adience benchmark |

AgeGoogLeNet is a GoogLeNet/Inception variant trained to predict human age from a face image. The output is a softmax distribution over 8 age buckets: `(0–2)`, `(4–6)`, `(8–13)`, `(15–20)`, `(25–32)`, `(38–43)`, `(48–53)`, `(60+)`.

The model was originally published as part of the [ONNX Model Zoo](https://github.com/onnx/models) and converted to SameDiff format via the DL4J ONNX importer.

**Loading and inference example:**

```java
import org.eclipse.deeplearning4j.omnihub.models.Pretrained;
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.ndarray.INDArray;
import java.util.Map;

SameDiff ageModel = Pretrained.samediff().ageGooglenet(false);

// Input: face image normalized to [0,1], shape [1, 3, 224, 224]
INDArray faceImage = Nd4j.rand(new int[]{1, 3, 224, 224});

Map<String, INDArray> outputs = ageModel.output(
    null,
    new String[]{"loss3/loss3_Y"},
    Map.of("data", faceImage)
);

INDArray ageProbs = outputs.get("loss3/loss3_Y"); // shape [1, 8]
int predictedBucket = ageProbs.argMax(1).getInt(0);
String[] ageBuckets = {"0-2","4-6","8-13","15-20","25-32","38-43","48-53","60+"};
System.out.println("Predicted age group: " + ageBuckets[predictedBucket]);
```

---

### ResNet18

| Property | Value |
|----------|-------|
| Method | `Pretrained.samediff().resnet18(forceDownload)` |
| File | `resnet18.fb` |
| Return type | `SameDiff` |
| Source | PyTorch torchvision `resnet18` — exported via ONNX |
| Original source | [torchvision models](https://pytorch.org/vision/stable/models.html) |
| Input name | `input.1` |
| Input shape | `[1, 3, 224, 224]` (NCHW) |
| Output name | `495` |
| Output shape | `[1, 1000]` |
| Output classes | 1000 ImageNet classes |
| Pretrained dataset | ImageNet (ILSVRC 2012) |

ResNet18 is an 18-layer residual network. The torchvision pretrained weights achieve approximately 69.8% top-1 accuracy on the ImageNet validation set. The model was exported from PyTorch via ONNX and imported into SameDiff.

The output tensor contains raw logits (not softmax probabilities). Apply a softmax if you need class probabilities.

**Loading and inference example:**

```java
import org.eclipse.deeplearning4j.omnihub.models.Pretrained;
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.ops.transforms.Transforms;
import java.util.Map;

SameDiff resnet = Pretrained.samediff().resnet18(false);

// Input: ImageNet-normalized image, shape [1, 3, 224, 224]
// Normalization: mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]
INDArray image = Nd4j.rand(new int[]{1, 3, 224, 224});

Map<String, INDArray> outputs = resnet.output(
    null,
    new String[]{"495"},
    Map.of("input.1", image)
);

INDArray logits = outputs.get("495"); // [1, 1000]
INDArray probs = Transforms.softmax(logits, false);
int topClass = probs.argMax(1).getInt(0);
System.out.println("Top ImageNet class index: " + topClass);
```

**Preprocessing.** The torchvision preprocessing pipeline normalizes each channel by subtracting the ImageNet mean and dividing by the standard deviation:

- Mean: `[0.485, 0.456, 0.406]` (RGB)
- Std: `[0.229, 0.224, 0.225]` (RGB)

Pixels should be in `[0.0, 1.0]` before normalization.

---

## Downloading All Models Upfront

If you want to pre-populate the cache (for example, during a Docker image build), call each model with `forceDownload = false`. You can also call `OmniHubUtils.downloadAndLoadFromZoo` directly without loading the model:

```java
import org.eclipse.deeplearning4j.omnihub.OmniHubUtils;

// Pre-download without loading into memory
OmniHubUtils.downloadAndLoadFromZoo("dl4j",
    "vgg19_weights_tf_dim_ordering_tf_kernels_notop.zip", false);
OmniHubUtils.downloadAndLoadFromZoo("samediff", "age_googlenet.fb", false);
OmniHubUtils.downloadAndLoadFromZoo("samediff", "resnet18.fb", false);
```

---

## Model Count Summary

| Framework | Count | Models |
|-----------|-------|--------|
| DL4J (ComputationGraph) | 1 | VGG19 (no top) |
| SameDiff | 2 | AgeGoogLeNet, ResNet18 |

The zoo is actively expanded. Models are added as new files to the zoo GitHub repository without requiring a library release. Check the [omnihub-zoo repository](https://github.com/KonduitAI/omnihub-zoo) for the current file listing.

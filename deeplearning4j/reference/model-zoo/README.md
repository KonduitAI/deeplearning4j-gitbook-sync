---
title: "Model Zoo"
description: "Pretrained models in Deeplearning4j — available architectures, using pretrained weights, and transfer learning from zoo models"
---

## About the Deeplearning4j Model Zoo

Deeplearning4j ships with a native model zoo that lets you instantiate well-known neural network architectures directly from Java, with no external downloads or manual configuration beyond adding a single Maven dependency. The zoo also provides pretrained weights for popular datasets — ImageNet, MNIST, CIFAR-10, and VGGFace — that are downloaded automatically and verified with a checksum on first use.

The model zoo covers the most widely used image classification and object detection architectures as well as a text-generation LSTM. Each model can be used in three modes:

1. Fresh initialization — a randomly initialized network with the original architecture, ready for training from scratch.
2. Pretrained weights — weights transferred from a known training run on a reference dataset, ready for inference or fine-tuning.
3. Custom input/output — a pretrained backbone with the classification head replaced to match your own number of classes.

### Maven Dependency

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-zoo</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

---

## The ZooModel Interface

Every model in the zoo extends the abstract class `ZooModel` and implements the `InstantiableModel` interface. The key methods are:

| Method | Description |
|---|---|
| `init()` | Returns a fresh `Model` (MultiLayerNetwork or ComputationGraph) with random weights |
| `initPretrained(PretrainedType)` | Downloads (if needed) and returns a model loaded with pretrained weights |
| `pretrainedAvailable(PretrainedType)` | Returns `true` if weights are available for the given dataset |
| `setInputShape(int[][])` | Override the default input shape before calling `init()` |
| `conf()` | Returns the underlying `MultiLayerConfiguration` for inspection or modification |

The `PretrainedType` enum specifies which dataset's weights to load:

- `PretrainedType.IMAGENET` — ImageNet (1000 classes, ILSVRC)
- `PretrainedType.MNIST` — MNIST handwritten digits
- `PretrainedType.CIFAR10` — CIFAR-10 (10 classes)
- `PretrainedType.VGGFACE` — VGGFace (face recognition)

Input shapes follow the NCHW convention: `{channels, height, width}`. For example, `{3, 224, 224}` means 3 RGB channels at 224 × 224 pixels.

---

## Initializing a Fresh Network

Use `.init()` to get a randomly initialized network for training from scratch. You must specify the number of output classes and a random seed via the builder:

```java
import org.deeplearning4j.zoo.model.AlexNet;
import org.deeplearning4j.zoo.ZooModel;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;

int numClasses = 1000;
int seed = 123;

ZooModel zooModel = AlexNet.builder()
        .numClasses(numClasses)
        .seed(seed)
        .build();

MultiLayerNetwork net = (MultiLayerNetwork) zooModel.init();
net.init();
System.out.println(net.summary());
```

To inspect or modify the configuration before building the network:

```java
ZooModel zooModel = AlexNet.builder()
        .numClasses(numClasses)
        .seed(seed)
        .build();
MultiLayerConfiguration conf = ((AlexNet) zooModel).conf();
// modify conf here, then build a MultiLayerNetwork from it
```

### Changing the Input Shape

By default each model has a fixed input shape. For models that support multiple resolutions (such as Darknet19, which supports 224 × 224 and 448 × 448), call `setInputShape()` before `init()`. This does not affect pretrained models.

```java
ZooModel zooModel = ResNet50.builder()
        .numClasses(10)
        .seed(42)
        .build();
zooModel.setInputShape(new int[][]{{3, 28, 28}});
Model net = zooModel.init();
```

---

## Loading Pretrained Weights

Call `initPretrained(PretrainedType)` to get a model loaded with weights from a reference training run. The weights file is downloaded to the DL4J cache directory on first use and verified via SHA-256 checksum on subsequent uses.

```java
import org.deeplearning4j.zoo.model.VGG16;
import org.deeplearning4j.zoo.PretrainedType;
import org.deeplearning4j.nn.graph.ComputationGraph;

ZooModel zooModel = VGG16.builder().build();
ComputationGraph net = (ComputationGraph) zooModel.initPretrained(PretrainedType.IMAGENET);
```

To check availability before loading:

```java
ZooModel zooModel = VGG16.builder().build();
if (zooModel.pretrainedAvailable(PretrainedType.VGGFACE)) {
    ComputationGraph faceModel = (ComputationGraph) zooModel.initPretrained(PretrainedType.VGGFACE);
}
```

Some models offer more than one set of pretrained weights. VGG16, for example, has ImageNet, CIFAR-10, and VGGFace variants.

---

## Transfer Learning

Pretrained zoo models are the natural starting point for transfer learning. The general workflow is:

1. Load a pretrained model via `initPretrained`.
2. Use `TransferLearning.Builder` (for `MultiLayerNetwork`) or `TransferLearning.GraphBuilder` (for `ComputationGraph`) to freeze earlier layers and replace the output layer.
3. Train the modified network on your own dataset.

### Feature Extraction (Frozen Backbone)

In feature extraction mode you freeze all layers in the pretrained model and replace only the final classification head. The frozen layers act as a fixed feature extractor.

```java
import org.deeplearning4j.zoo.model.VGG16;
import org.deeplearning4j.nn.transferlearning.FineTuneConfiguration;
import org.deeplearning4j.nn.transferlearning.TransferLearning;
import org.deeplearning4j.nn.graph.ComputationGraph;
import org.nd4j.linalg.learning.config.Adam;

ComputationGraph vgg16 = (ComputationGraph)
        VGG16.builder().build().initPretrained(PretrainedType.IMAGENET);

FineTuneConfiguration fineTuneConf = new FineTuneConfiguration.Builder()
        .updater(new Adam(1e-3))
        .seed(123)
        .build();

ComputationGraph transferModel = new TransferLearning.GraphBuilder(vgg16)
        .fineTuneConfiguration(fineTuneConf)
        .setFeatureExtractor("fc2")          // freeze everything up to and including fc2
        .removeVertexKeepConnections("predictions")
        .addLayer("predictions",
                new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
                        .nIn(4096).nOut(numYourClasses)
                        .activation(Activation.SOFTMAX).build(),
                "fc2")
        .build();

transferModel.init();
```

### Fine-Tuning (Partial Unfreezing)

Fine-tuning unfreezes some of the later layers so they can adapt to the new dataset, while keeping early layers (which capture low-level features) frozen.

```java
ComputationGraph transferModel = new TransferLearning.GraphBuilder(vgg16)
        .fineTuneConfiguration(fineTuneConf)
        .setFeatureExtractor("block4_pool")  // freeze through block4_pool
        .removeVertexKeepConnections("predictions")
        .addLayer("predictions",
                new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
                        .nIn(4096).nOut(numYourClasses)
                        .activation(Activation.SOFTMAX).build(),
                "fc2")
        .build();
```

For a complete transfer learning guide, see the [Transfer Learning](../nn/transfer-learning.md) documentation.

---

## Memory and Workspace Configuration

Initialization methods accept an optional `workspaceMode` parameter. Most users will not need to change this. If you are running a model with a very large number of parameters (such as VGG19 with 143 million parameters) on a machine with ample RAM, passing `WorkspaceMode.SINGLE` can reduce memory overhead:

```java
ZooModel zooModel = VGG19.builder().build();
ComputationGraph net = (ComputationGraph)
        zooModel.initPretrained(PretrainedType.IMAGENET, WorkspaceMode.SINGLE);
```

For general workspace configuration guidance, see the [Workspaces](../config/workspaces.md) documentation.

---

## Available Models at a Glance

| Model | Input Shape | Pretrained Datasets |
|---|---|---|
| AlexNet | 3 × 224 × 224 | — |
| Darknet19 | 3 × 224 × 224, 3 × 448 × 448 | ImageNet |
| FaceNetNN4Small2 | 3 × 96 × 96 | — |
| InceptionResNetV1 | 3 × 160 × 160 | VGGFace |
| LeNet | 1 × 28 × 28 | MNIST |
| NASNet | 3 × 224 × 224 | ImageNet |
| ResNet50 | 3 × 224 × 224 | ImageNet |
| SimpleCNN | 3 × 224 × 224 | — |
| SqueezeNet | 3 × 227 × 227 | ImageNet |
| TextGenerationLSTM | — | Walt Whitman corpus |
| TinyYOLO | 3 × 416 × 416 | ImageNet + VOC |
| UNet | 1 × 512 × 512 | Synthetic segmentation |
| VGG16 | 3 × 224 × 224 | ImageNet, CIFAR-10, VGGFace |
| VGG19 | 3 × 224 × 224 | ImageNet |
| Xception | 3 × 299 × 299 | ImageNet |
| YOLO2 | 3 × 608 × 608 | ImageNet + COCO |

For full per-model details including parameter counts, paper references, and code links, see the [Available Models](models.md) page.

---

## Source Code

All zoo model implementations are in the `deeplearning4j-zoo` module:
[github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model)

---
title: "Available Models"
description: "Complete list of pretrained models — VGG, ResNet, AlexNet, LeNet, YOLO, UNet, and other architectures"
---

## Available Models in the DL4J Model Zoo

All models are in the `org.deeplearning4j.zoo.model` package. Add the `deeplearning4j-zoo` dependency to your project to use them (see the [Model Zoo Overview](overview.md) for setup instructions).

Input shapes follow NCHW convention: `{channels, height, width}`. Pretrained weights are downloaded automatically to the DL4J cache directory on first use and verified by checksum.

---

### AlexNet

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/AlexNet.java)

DL4J's implementation of AlexNet, based on the original ImageNet Classification paper. The architecture uses five convolutional layers followed by three fully connected layers with local response normalization and dropout.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.AlexNet` |
| Input shape | 3 × 224 × 224 |
| Pretrained datasets | None |
| Use case | Image classification baseline, educational reference |

**Paper:** [ImageNet Classification with Deep Convolutional Neural Networks](http://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf) (Krizhevsky et al., NeurIPS 2012)

```java
ZooModel model = AlexNet.builder()
        .numClasses(1000)
        .seed(123)
        .build();
MultiLayerNetwork net = (MultiLayerNetwork) model.init();
```

---

### Darknet19

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/Darknet19.java)

Darknet19 is the feature-extraction backbone used in YOLO v2. It uses 19 convolutional layers and 5 max-pooling layers. Two pretrained resolutions are available: 224 × 224 and 448 × 448. Call `setInputShape()` to select which resolution you need before initialization.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.Darknet19` |
| Input shape | 3 × 224 × 224 or 3 × 448 × 448 |
| Pretrained datasets | ImageNet |
| Use case | Object detection backbone, image classification |

Input images must have channels in RGB order (not BGR) with pixel values normalized to [0, 1]. Output labels match the [ImageNet shortnames list](https://github.com/pjreddie/darknet/blob/master/data/imagenet.shortnames.list).

**Paper:** [YOLO9000: Better, Faster, Stronger](https://arxiv.org/pdf/1612.08242.pdf) (Redmon & Farhadi, 2016)

```java
ZooModel model = Darknet19.builder().build();
model.setInputShape(new int[][]{{3, 224, 224}});
ComputationGraph net = (ComputationGraph) model.initPretrained(PretrainedType.IMAGENET);
```

---

### FaceNetNN4Small2

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/FaceNetNN4Small2.java)

A compact variant of the FaceNet model that produces 128-dimensional face embeddings. Uses triplet loss for training. The "NN4Small2" name refers to the NN4 inception module variant at a smaller scale. Suitable for face verification and recognition tasks where a similarity metric between face embeddings is used at inference time.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.FaceNetNN4Small2` |
| Input shape | 3 × 96 × 96 |
| Pretrained datasets | None |
| Use case | Face verification, face recognition, embedding extraction |

**Papers:** [FaceNet: A Unified Embedding for Face Recognition and Clustering](https://arxiv.org/abs/1503.03832); [OpenFace: A general-purpose face recognition library](http://reports-archive.adm.cs.cmu.edu/anon/2016/CMU-CS-16-118.pdf)

---

### InceptionResNetV1

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/InceptionResNetV1.java)

Inception-ResNet-V1 combines the inception module design with residual connections. This DL4J implementation is adapted for face recognition (trained with triplet loss on VGGFace data) rather than general ImageNet classification, which distinguishes it from the standard InceptionResNetV1 used for image classification.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.InceptionResNetV1` |
| Input shape | 3 × 160 × 160 |
| Pretrained datasets | VGGFace |
| Use case | Face recognition, face verification |

**Paper:** [FaceNet: A Unified Embedding for Face Recognition and Clustering](https://arxiv.org/abs/1503.03832)

```java
ZooModel model = InceptionResNetV1.builder().build();
ComputationGraph net = (ComputationGraph) model.initPretrained(PretrainedType.VGGFACE);
```

---

### LeNet

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/LeNet.java)

LeNet-5 is one of the earliest convolutional neural networks, developed by Yann LeCun for handwritten digit recognition. Despite its age and simplicity (two convolutional layers, two pooling layers, two fully connected layers), it remains a useful baseline and educational reference. MNIST pretrained weights are available.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.LeNet` |
| Input shape | 1 × 28 × 28 |
| Parameters | ~60,000 |
| Pretrained datasets | MNIST |
| Use case | Digit recognition, educational baseline |

**Paper:** [Gradient-Based Learning Applied to Document Recognition](http://yann.lecun.com/exdb/publis/pdf/lecun-98.pdf) (LeCun et al., 1998)

```java
ZooModel model = LeNet.builder().build();
MultiLayerNetwork net = (MultiLayerNetwork) model.initPretrained(PretrainedType.MNIST);
```

---

### NASNet

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/NASNet.java)

Neural Architecture Search Network (NASNet-A) was designed by learning the model architecture directly on the dataset of interest using reinforcement learning. The DL4J implementation uses 1056 penultimate filters and the standard 224 × 224 input. ImageNet weights are available, converted from the Keras Applications implementation.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.NASNet` |
| Input shape | 3 × 224 × 224 |
| Pretrained datasets | ImageNet |
| Use case | High-accuracy image classification |

**Paper:** [Learning Transferable Architectures for Scalable Image Recognition](https://arxiv.org/abs/1707.07012) (Zoph et al., 2017)

```java
ZooModel model = NASNet.builder().build();
ComputationGraph net = (ComputationGraph) model.initPretrained(PretrainedType.IMAGENET);
```

---

### ResNet50

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/ResNet50.java)

ResNet-50 is a 50-layer residual network that uses skip connections to allow gradients to flow through very deep networks without vanishing. It achieved state-of-the-art accuracy on ImageNet in 2015 and remains a widely used backbone for both classification and feature extraction. ImageNet weights are available, converted from Keras Applications.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.ResNet50` |
| Input shape | 3 × 224 × 224 |
| Parameters | ~25 million |
| Pretrained datasets | ImageNet |
| Use case | Image classification, transfer learning backbone |

**Paper:** [Deep Residual Learning for Image Recognition](https://arxiv.org/abs/1512.03385) (He et al., 2015)

```java
ZooModel model = ResNet50.builder().build();
ComputationGraph net = (ComputationGraph) model.initPretrained(PretrainedType.IMAGENET);
```

---

### SimpleCNN

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/SimpleCNN.java)

A compact convolutional network for generic image classification, adapted from an open-source face classification implementation. Useful as a lightweight baseline or starting point when you need a small model that trains quickly. No pretrained weights are available; this model is intended to be trained on your own data.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.SimpleCNN` |
| Input shape | 3 × 224 × 224 |
| Pretrained datasets | None |
| Use case | Custom image classification, fast prototyping |

```java
ZooModel model = SimpleCNN.builder()
        .numClasses(10)
        .seed(42)
        .build();
MultiLayerNetwork net = (MultiLayerNetwork) model.init();
```

---

### SqueezeNet

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/SqueezeNet.java)

SqueezeNet achieves AlexNet-level accuracy on ImageNet with approximately 50× fewer parameters through the use of "fire modules" (1 × 1 squeeze convolutions followed by a mix of 1 × 1 and 3 × 3 expand convolutions). It is well-suited for deployment in memory-constrained environments. ImageNet weights are available, converted from the keras-squeezenet implementation.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.SqueezeNet` |
| Input shape | 3 × 227 × 227 |
| Parameters | ~1.2 million |
| Pretrained datasets | ImageNet |
| Use case | Embedded/mobile image classification |

**Paper:** [SqueezeNet: AlexNet-level accuracy with 50x fewer parameters](https://arxiv.org/abs/1602.07360) (Iandola et al., 2016)

```java
ZooModel model = SqueezeNet.builder().build();
ComputationGraph net = (ComputationGraph) model.initPretrained(PretrainedType.IMAGENET);
```

---

### TextGenerationLSTM

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/TextGenerationLSTM.java)

An LSTM recurrent neural network designed for character-level text generation. The architecture follows the Keras LSTM text generation example. `numClasses` must be set to the size of the character vocabulary for your corpus. Pretrained weights trained on the complete works of Walt Whitman are available.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.TextGenerationLSTM` |
| Input type | Character sequences |
| Pretrained datasets | Walt Whitman corpus |
| Use case | Character-level text generation, sequence modeling |

```java
int charVocabSize = 83;  // size of your character set
ZooModel model = TextGenerationLSTM.builder()
        .numClasses(charVocabSize)
        .build();
MultiLayerNetwork net = (MultiLayerNetwork) model.init();
```

---

### TinyYOLO

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/TinyYOLO.java)

Tiny YOLO is a compact, fast variant of the YOLO object detection architecture built on top of Darknet. It trades some accuracy for significantly faster inference, making it suitable for real-time applications on modest hardware. Pretrained weights covering ImageNet and PASCAL VOC object categories are available, converted from the original Darknet implementation via YAD2K.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.TinyYOLO` |
| Input shape | 3 × 416 × 416 |
| Pretrained datasets | ImageNet + VOC (20 object categories) |
| Use case | Real-time object detection |

Input images must have channels in RGB order (not BGR) with pixel values normalized to [0, 1].

**Paper:** [YOLO9000: Better, Faster, Stronger](https://arxiv.org/pdf/1612.08242.pdf) (Redmon & Farhadi, 2016)

```java
ZooModel model = TinyYOLO.builder().build();
ComputationGraph net = (ComputationGraph) model.initPretrained(PretrainedType.IMAGENET);
```

---

### UNet

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/UNet.java)

U-Net is a convolutional encoder-decoder architecture designed for biomedical image segmentation. Its "U"-shaped design uses skip connections between the encoder and decoder paths to preserve spatial information lost during downsampling. It set the state of the art on the ISBI cell-tracking challenge and is widely used in medical imaging. Pretrained weights trained on a synthetic segmentation dataset are available.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.UNet` |
| Input shape | 1 × 512 × 512 |
| Pretrained datasets | Synthetic segmentation data |
| Use case | Image segmentation, medical imaging |

**Paper:** [U-Net: Convolutional Networks for Biomedical Image Segmentation](https://arxiv.org/abs/1505.04597) (Ronneberger et al., 2015)

```java
ZooModel model = UNet.builder().build();
ComputationGraph net = (ComputationGraph) model.initPretrained(PretrainedType.SEGMENT);
```

---

### VGG16

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/VGG16.java)

VGG-16 is a 16-layer network from the Oxford Visual Geometry Group, notable for its uniform architecture of 3 × 3 convolutions throughout. Despite being over a decade old, it remains popular for transfer learning due to its straightforward structure. Three pretrained weight sets are available: ImageNet classification, CIFAR-10, and VGGFace (face recognition).

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.VGG16` |
| Input shape | 3 × 224 × 224 |
| Parameters | ~138 million |
| Pretrained datasets | ImageNet, CIFAR-10, VGGFace |
| Use case | Image classification, transfer learning, face recognition |

**Papers:** [Very Deep Convolutional Networks for Large-Scale Image Recognition](https://arxiv.org/abs/1409.1556); [Deep Face Recognition](http://www.robots.ox.ac.uk/~vgg/publications/2015/Parkhi15/parkhi15.pdf)

```java
// ImageNet weights
ZooModel model = VGG16.builder().build();
ComputationGraph imagenetNet = (ComputationGraph) model.initPretrained(PretrainedType.IMAGENET);

// VGGFace weights
ZooModel faceModel = VGG16.builder().build();
ComputationGraph faceNet = (ComputationGraph) faceModel.initPretrained(PretrainedType.VGGFACE);
```

---

### VGG19

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/VGG19.java)

VGG-19 extends VGG-16 with three additional convolutional layers for a total of 19 weight layers. It has slightly higher capacity but also substantially more parameters. Due to its size, consider using `WorkspaceMode.SINGLE` on large machines to reduce memory overhead. ImageNet weights are available.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.VGG19` |
| Input shape | 3 × 224 × 224 |
| Parameters | ~143 million |
| Pretrained datasets | ImageNet |
| Use case | Image classification, style transfer |

**Paper:** [Very Deep Convolutional Networks for Large-Scale Image Recognition](https://arxiv.org/abs/1409.1556)

```java
ZooModel model = VGG19.builder().build();
ComputationGraph net = (ComputationGraph)
        model.initPretrained(PretrainedType.IMAGENET, WorkspaceMode.SINGLE);
```

---

### Xception

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/Xception.java)

Xception ("Extreme Inception") replaces standard Inception modules with depthwise separable convolutions, which decouple the mapping of cross-channel correlations from spatial correlations. This results in a more parameter-efficient architecture that outperforms Inception V3 on ImageNet while using fewer parameters. ImageNet weights are available, converted from Keras Applications.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.Xception` |
| Input shape | 3 × 299 × 299 |
| Parameters | ~22 million |
| Pretrained datasets | ImageNet |
| Use case | Image classification, efficient transfer learning |

**Paper:** [Xception: Deep Learning with Depthwise Separable Convolutions](https://arxiv.org/abs/1610.02357) (Chollet, 2016)

```java
ZooModel model = Xception.builder().build();
ComputationGraph net = (ComputationGraph) model.initPretrained(PretrainedType.IMAGENET);
```

---

### YOLO2

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/YOLO2.java)

YOLO v2 (full resolution) uses Darknet19 as its backbone and processes 608 × 608 input images for higher accuracy than TinyYOLO. Pretrained weights covering ImageNet pretraining followed by fine-tuning on COCO object detection categories are available, converted from the original Darknet implementation. A `Yolo2OutputLayer` is appended during fine-tuning on custom datasets.

| Property | Value |
|---|---|
| Class | `org.deeplearning4j.zoo.model.YOLO2` |
| Input shape | 3 × 608 × 608 |
| Pretrained datasets | ImageNet + COCO (80 object categories) |
| Use case | Full-resolution object detection |

Input images must have channels in RGB order (not BGR) with pixel values normalized to [0, 1].

**Paper:** [YOLO9000: Better, Faster, Stronger](https://arxiv.org/pdf/1612.08242.pdf) (Redmon & Farhadi, 2016)

```java
ZooModel model = YOLO2.builder().build();
ComputationGraph net = (ComputationGraph) model.initPretrained(PretrainedType.IMAGENET);
```

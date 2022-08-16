---
description: Prebuilt model architectures and weights for out-of-the-box application.
---

# Model Zoo

Deeplearning4j has native model zoo that can be accessed and instantiated directly from DL4J. The model zoo also includes pretrained weights for different datasets that are downloaded automatically and checked for integrity using a checksum mechanism.

If you want to use the new model zoo, you will need to add it as a dependency. A Maven POM would add the following:

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-zoo</artifactId>
    <version>1.0.0-M1.1</version>
</dependency>
```

## Getting started

Once you've successfully added the zoo dependency to your project, you can start to import and use models. Each model extends the `ZooModel` abstract class and uses the `InstantiableModel` interface. These classes provide methods that help you initialize either an empty, fresh network or a pretrained network.

### Initializing fresh configurations

You can instantly instantiate a model from the zoo using the `.init()` method. For example, if you want to instantiate a fresh, untrained network of AlexNet you can use the following code:

```java
import org.deeplearning4j.zoo.model.AlexNet
import org.deeplearning4j.zoo.*;

...

int numberOfClassesInYourData = 1000;
int randomSeed = 123;

ZooModel zooModel = AlexNet.builder()
                .numClasses(numberOfClassesInYourData)
                .seed(randomSeed)
                .build();
Model net = zooModel.init();
```

If you want to tune parameters or change the optimization algorithm, you can obtain a reference to the underlying network configuration:

```java
ZooModel zooModel = AlexNet.builder()
                .numClasses(numberOfClassesInYourData)
                .seed(randomSeed)
                .build();
MultiLayerConfiguration net = ((AlexNet) zooModel).conf();
```

### Initializing pretrained weights

Some models have pretrained weights available, and a small number of models are pretrained across different datasets. `PretrainedType` is an enumerator that outlines different weight types, which includes `IMAGENET`, `MNIST`, `CIFAR10`, and `VGGFACE`.

For example, you can initialize a VGG-16 model with ImageNet weights like so:

```java
import org.deeplearning4j.zoo.model.VGG16;
import org.deeplearning4j.zoo.*;

...

ZooModel zooModel = VGG16.builder().build();;
Model net = zooModel.initPretrained(PretrainedType.IMAGENET);
```

And initialize another VGG16 model with weights trained on VGGFace:

```java
ZooModel zooModel = VGG16.builder().build();
Model net = zooModel.initPretrained(PretrainedType.VGGFACE);
```

If you're not sure whether a model contains pretrained weights, you can use the `.pretrainedAvailable()` method which returns a boolean. Simply pass a `PretrainedType` enum to this method, which returns true if weights are available.

Note that for convolutional models, input shape information follows the NCHW convention. So if a model's input shape default is `new int[]{3, 224, 224}`, this means the model has 3 channels and height/width of 224.

## What's in the zoo?

The model zoo comes with well-known image recognition configurations in the deep learning community. The zoo also includes an LSTM for text generation, and a simple CNN for general image recognition.

You can find a complete list of models using this [deeplearning4j-zoo Github link](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model).

This includes ImageNet models such as VGG-16, ResNet-50, AlexNet, Inception-ResNet-v1, LeNet, and more.

* [AlexNet](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/AlexNet.java)   &#x20;
* [Darknet19](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/Darknet19.java)   &#x20;
* [FaceNetNN4Small2](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/FaceNetNN4Small2.java)   &#x20;
* [InceptionResNetV1](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/InceptionResNetV1.java)   &#x20;
* [LeNet](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/LeNet.java)
* [ResNet50](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/ResNet50.java)
* [SimpleCNN](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/SimpleCNN.java)
* [TextGenerationLSTM](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/TextGenerationLSTM.java)
* [TinyYOLO](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/TinyYOLO.java)
* [VGG16](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/VGG16.java)   &#x20;
* [VGG19](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-zoo/src/main/java/org/deeplearning4j/zoo/model/VGG19.java)

## Advanced usage

The zoo comes with a couple additional features if you're looking to use the models for different use cases.

### Changing Inputs

Aside from passing certain configuration information to the constructor of a zoo model, you can also change its input shape using `.setInputShape()`.

NOTE: this applies to fresh configurations only, and will not affect pretrained models:

```java
int numberOfClassesInYourData = 10;
int randomSeed = 123;

ZooModel zooModel = ResNet50.builder()
        .numClasses(numberOfClassesInYourData)
        .seed(randomSeed)
        .build();
zooModel.setInputShape(new int[][]{{3, 28, 28}});
```

### Transfer Learning

Pretrained models are perfect for transfer learning! You can read more about transfer learning using DL4J [here](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/tuning-and-training/transfer-learning.md).

### Workspaces

Initialization methods often have an additional parameter named `workspaceMode`. For the majority of users you will not need to use this; however, if you have a large machine that has "beefy" specifications, you can pass `WorkspaceMode.SINGLE` for models such as VGG-19 that have many millions of parameters. To learn more about workspaces, please see [this section](../../../multi-project/explanation/configuration/memory/workspaces.md).

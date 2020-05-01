---
description: java与maven快速入门
---

# 快速入门

## 开始

这是运行DL4J示例所需的一切，并开始自己的项目。

我们建议你加入我们的 [Gitter Live Chat](https://gitter.im/deeplearning4j/deeplearning4j)。Gitter是你可以请求帮助和提供反馈的地方，但是在问下面我们已经回答的问题之前，请务必使用这个指南。如果你是深度学习的新手，我们已经为初学者提供了[路线图](https://deeplearning4j.org/docs/latest/deeplearning4j-beginners)，链接到课程、阅读和其他资源。

## 代码尝试

DL4J是一种特定领域的语言，用于配置由多层构成的深度神经网络。一切都从MultiLayerConfiguration开始，这些MultiLayerConfiguration组织这些层和它们的超参数。

超参数是决定神经网络如何学习的变量。它们包括更新模型权重的次数、如何初始化这些权重、将哪个激活函数附加到节点、使用哪个优化算法以及模型应该学习多快。这就是一个配置的样子：

```java
    MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
        .weightInit(WeightInit.XAVIER)
        .activation("relu")
        .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
        .updater(new Sgd(0.05))
        // ... other hyperparameters
        .list()
        .backprop(true)
        .build();
```

使用Deeplearning4j，可以通过调用NeuralNetConfiguration.Builder\(\)上的layer方法来添加层，按照层的顺序（下面的零索引层是输入层）指定其位置、输入和输出节点的数量是nIn和nOut以及类型为：DenseLayer。

```java
        .layer(0, new DenseLayer.Builder().nIn(784).nOut(250)
                .build())
```

一旦你已经配置好你的网络，你可以用 `model.fit`来训练你的网络

## 先决条件

* [Java \(developer version\)](kuai-su-ru-men.md#java) 1.7 或更高版本 \(仅支持64位版本\)
* [Apache Maven](kuai-su-ru-men.md#apache-maven) \(自动化构建和依赖管理器\)
* [IntelliJ IDEA](kuai-su-ru-men.md#intellij-idea) 或 Eclipse
* [Git](kuai-su-ru-men.md#git)

你应该安装这些来使用这个快速入门指南。DL4J针对熟悉生产部署、IDE和自动化构建工具的专业Java开发人员。如果你已经有了这些经验，使用DL4J将是最简单的。如果您是Java新手或不熟悉这些工具，请阅读下面的详细信息以帮助安装和设置。否则，跳到[DL4J示例](https://deeplearning4j.org/docs/latest/deeplearning4j-quickstart#examples)。

## **Java**

如果你没有Java 1.7 或更高版版的Java，下载当前最新的[Java Development Kit \(JDK\) ](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)。用以下的命令来检查你是否有安装可兼容的Java版本。

```bash
java -version
```

确保你有一个64位版本的JAVA已被安装，如果你决定用32位版本来尝试，你会看到一个错误告诉你no jnind4j in java.library.path

确保JAVA\_HOME 环境变量已被设置。

## **Apache Maven**

Maven是Java项目的依赖管理和自动构建工具。它很好地与IntelliJ等IDE一起工作，并允许你轻松安装DL4J项目库。按照他们的说明为您的系统安装或更新Maven最新版本。若要检查是否安装了最新版本的Maven，请输入以下内容：

```text
mvn --version
```

如果你在Mac上工作，你可以简单的输入如下的命令行：

```text
brew install maven
```

Maven在Java开发人员中被广泛使用，它与DL4J一起工作是非常必要的。如果你来自不同的背景，Maven对你来说是新的，请检查Apache的Maven概览和我们对非Java程序员的Maven的介绍，其中包括一些额外的疑难解答。其他构建工具，如常ivy和Dradle也可以工作，但我们对Maven支持最好。

* [Paul Dubs’ guide to maven](http://www.dubs.tech/guides/maven-essentials/)
* [Maven In Five Minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)

## **IntelliJ IDEA**

集成开发环境（IDE）允许你使用我们的API并在几个步骤中配置神经网络。我们强烈建议使用IntelliJ，它与Maven通信来处理依赖关系。IntelliJ社区版是免费的。

还有其他流行的IDE，如Eclipse和NETBeans。但是，IntelliJ是首选，如果需要的话，使用它会让你在Gitter Live Chat 上更容易找到帮助。

## **Git**

安装Git的最新版本。如果你已经拥有Git，你可以使用Git本身更新到最新版本：

```text
$ git clone git://git.kernel.org/pub/scm/git/git.git
```

### 几个简单步骤中的DL4J例子

1. 使用命令行输入以下内容:

```text
$ git clone https://github.com/deeplearning4j/dl4j-examples.git
$ cd dl4j-examples/
$ mvn clean install
```

1. 打开IntelliJ并选择Import Project。然后选择dl4j-examples主要的目录（注意：以下的示例中阐述的是一个过期的仓库名为dl4j-0.4-examples。尽管如此，你将要下载和安装的仓库为 dl4j-examples）。  ![](../.gitbook/assets/install_intj_1.png) 
2. 选择 ‘Import project from external model’ 并确保Maven被选择  ![](../.gitbook/assets/install_intj_2%20%284%29.png) 
3. 通过向导选项继续。选择以jdk开头的SDK。（你也许需要点击一个加号来查看你的选项）然后点击 finish。等待IntelliJ下载完所有的依赖。你将看到右下角水平的进度条。
4. 在左边文件树中选择一个例子 ，右击文件来运行 ![&#x70B9;&#x51FB;&#x67E5;&#x770B;&#x539F;&#x59CB;&#x5927;&#x5C0F;&#x56FE;&#x7247;](http://upload-images.jianshu.io/upload_images/14495907-0d9da1e2f4aaaaf1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 在你的工程中使用DL4J：配置POM.xml文件

为了在你自己的工程中运行DL4J，我们强烈推荐使用Maven用于Java用户，或者为scala使用SBT工具。基本的依赖集及其版本如下所示。这包括：

* `deeplearning4j-core`, 包括神经网络的实现
* `nd4j-native-platform`, CPU版本的ND4J库为DL4J提供支持
* `datavec-api` - Datavec是我们用于向量化和加载数据的库

每个Maven工程都有一个POM文件。[这里](https://github.com/deeplearning4j/dl4j-examples/blob/master/pom.xml)是运行示例时应该出现的POM文件。

在IntelliJ内部，你需要选择你要运行的第一个深度学习4J示例。我们建议MLPClassifierLinear，因为你几乎会立即看到网络将两组数据分类在我们的UI中。GITHUB上的文件可以在[这里](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/feedforward/classification/MLPClassifierLinear.java)找到。

若要运行该示例，请右键单击它并选择下拉菜单中的绿色按钮。你会看到，在IntelliJ的底部窗口，一系列的分数。最右边的数字是网络分类的错误分数。如果你的网络正在学习，那么随着它处理的每个批次，这个数字会随着时间的推移而减少。最后，这个窗口会告诉你你的神经网络模型变得有多精确：

![mlp classifier results](http://upload-images.jianshu.io/upload_images/14495907-d38ddebb95dc4603.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在另一个窗口中，将出现一个图表，显示多层感知器（MLP）如何对示例中的数据进行分类。它看起来像这样：

![mlp classifier viz](http://upload-images.jianshu.io/upload_images/14495907-933afbb9caba1db6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

恭喜，你已用DL4J训练了你的第一个神经网络。

## 接下来的步骤

1. 在Gitter上加入我们。我们有三个大的社区渠道。
   * [DL4J Live Chat](https://gitter.im/deeplearning4j/deeplearning4j) 是有关DL4J问题的主要渠道。大多数人出现在这儿。
   * [Tuning Help](https://gitter.im/deeplearning4j/deeplearning4j/tuninghelp) 是提供给刚开始学习神经网络的人。初学者从这访问我们。
   * [Early Adopters](https://gitter.im/deeplearning4j/deeplearning4j/earlyadopters) 是提供给检查和改进版本的人。警告：这个是提供给更多经验的爱好者。
2. 阅读 [神经网络介绍](https://skymind.ai/wiki/neural-network).
3. 查看更详细的  [全面设置指南](https://deeplearning4j.org/docs/latest/deeplearning4j-quickstart).
4. 浏览 [DL4J 文档 ](https://deeplearning4j.org/docs/latest/).
5. Python 爱好者 : 如果您计划在Deeplearning4j上运行基准测试，并将其与著名的Python框架\[x\]进行比较，请阅读这些关于如何在JVM上优化堆空间、垃圾收集以及ETL的[说明](https://deeplearning4j.org/docs/latest/deeplearning4j-benchmark)。通过参照它们，你会看到至少10倍的加速训练时间。

## 其它链接

* [Deeplearning4j artifacts on Maven Central](https://search.maven.org/#search%7Cga%7C1%7Cdeeplearning4j)
* [ND4J artifacts on Maven Central](https://search.maven.org/#search%7Cga%7C1%7Cnd4j)
* [Datavec artifacts on Maven Central](https://search.maven.org/#search%7Cga%7C1%7Cdatavec)
* [Scala code for UCI notebook](https://github.com/SkymindIO/SKIL_Examples/blob/master/skil_example_notebooks/scala/uci_quickstart_notebook.scala)

## 故障排查

问: 我在Windows上使用64位的JAVA仍然得到 `no jnind4j in java.library.path` 错误

答: 你可能有不兼容的 DLLs 在你的 PATH中. 为了让 DL4J忽略这些, 你必须添加如下作为 VM 参数 \(Run -&gt; Edit Configurations -&gt; VM Options in IntelliJ\):

```text
-Djava.library.path=""
```

问: 我正在运行示例，并且基于Spark的示例存在问题，例如分布式培训或datavec转换选项。

答: 你可能丢失了Spark要求的一些依赖。查看这个[Stack Overflow discussion](https://stackoverflow.com/a/38735202/3892515)来获得一个一个潜在的依赖问题讨论。windows用户可能需要来自Hadoop的winutils.exe。 从[https://github.com/steveloughran/winutils](https://github.com/steveloughran/winutils) and put it into the null/bin/winutils.exe下载winutils.exe （或创建一个hadoop文件夹并添加它到HADOOP\_HOME）。

#### 故障排查: 在Windows上调试UnsatisfiedLinkError

Windows用户可能看到如下信息：

```text
Exception in thread "main" java.lang.ExceptionInInitializerError
at org.deeplearning4j.nn.conf.NeuralNetConfiguration$Builder.seed(NeuralNetConfiguration.java:624)
at org.deeplearning4j.examples.feedforward.anomalydetection.MNISTAnomalyExample.main(MNISTAnomalyExample.java:46)
Caused by: java.lang.RuntimeException: org.nd4j.linalg.factory.Nd4jBackend$NoAvailableBackendException: Please ensure that you have an nd4j backend on your classpath. Please see: http://nd4j.org/getstarted.html
at org.nd4j.linalg.factory.Nd4j.initContext(Nd4j.java:5556)
at org.nd4j.linalg.factory.Nd4j.(Nd4j.java:189)
... 2 more
Caused by: org.nd4j.linalg.factory.Nd4jBackend$NoAvailableBackendException: Please ensure that you have an nd4j backend on your classpath. Please see: http://nd4j.org/getstarted.html
at org.nd4j.linalg.factory.Nd4jBackend.load(Nd4jBackend.java:259)
at org.nd4j.linalg.factory.Nd4j.initContext(Nd4j.java:5553)
... 3 more
```

如果是这个问题，请参阅[本页](https://github.com/bytedeco/javacpp-presets/wiki/Debugging-UnsatisfiedLinkError-on-Windows#using-dependency-walker)。在这种情况下，替换为“Nd4jcpu”。

#### Eclipse 不使用 Maven的设置

我们推荐使用Maven 和 Intellij。如果你更喜欢Eclipse并不喜欢Maven 这里有一篇很好的[博客](https://electronsfree.blogspot.com/2016/10/how-to-setup-dl4j-project-with-eclipse.html)让你过度到Eclipse配置

## 快速入门模版

现在，您已经了解了如何运行不同的示例，我们已经为你提供了一个模板，该模版具有一个基本的EMNIST训练器，具有早停和评估代码。

快速入门模版在此[https://github.com/deeplearning4j/dl4j-quickstart](https://github.com/deeplearning4j/dl4j-quickstart).

使用模版:

1. 克隆到你的本地机器  `git clone https://github.com/deeplearning4j/dl4j-quickstart.git`
2. 导入 `dl4j-quickstart` 主文件夹到 IntelliJ.
3. 开始编码！

## Eclipse Deeplearning4j 的更多信息

Deeplearning4j是一个可以让你从一开始就可以选择一切的框架。我们不是Tensorflow（一个具有自动微分的低级数值计算库）或是Pytorch。

Deeplearning4j有几个子项目，使其易于构建端到端应用程序。

如果你想将模型部署到生产中，你可能会喜欢我们的从Keras导入的模型。

Deeplearning4j有几个子模块。这些范围从可视化UI到spark分布式训练。对于这些模块的概述，请查看Github上的[Deeplearning4j示例](https://github.com/deeplearning4j/dl4j-examples)。

如果你想要一个简单的桌面应用作为开始，你需要两件个东西：一个 [nd4j backend](http://nd4j.org/backend.html) 和 `deeplearning4j-core`。更多代码请查看 [simpler examples submodule](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/pom.xml#L64).

如果你想要一个灵活的深度学习API，这有两种方式。你可以单独使用nd4j，查看我们的 [nd4j ](https://github.com/deeplearning4j/dl4j-examples/tree/master/nd4j-examples)示例 或 计算图API.

如果你想要在 Spark进行分布式训练，你可以查看我们的 Spark page记住我们不会为你设置spark

如果你想要设置spark和GPU,这在很大程度上取决于你。Deeplearning4j简单的作为一个jar文件部署在一个存在的Spark集群上。

如果你想要Spark和GPU一起工作，我们推荐你看[Spark with Mesos](https://spark.apache.org/docs/latest/running-on-mesos.html)。

如果你想在移动端部署，你可以看我们的[Android page](https://deeplearning4j.org/docs/latest/deeplearning4j-android)。

我们为各种硬件架构部署优化后的代码。我们使用基于C++的循环，就像其他人一样。请查看[C++ framework libnd4j](https://github.com/deeplearning4j/libnd4j)。

Deeplearning4j有其它两个值得注意的组件：

* Arbiter: 超参数优化和模型评估
* DataVec: 用于机器学习数据管道的内置ETL工具

Deeplearning4j是构建真实应用程序的端到端平台，而不仅仅是具有自动微分的张量库。如果你想要一个带有autodiff的张量库，请参阅ND4J和samediff。samediff仍然在测试，但是如果你想做出贡献，请加入我们的[live chat on Gitter](https://gitter.im/deeplearning4j/deeplearning4j)。

最后，如果你正在测试DL4J，请考虑进入我们的在线聊天并获取提示。DL4J有所有的模块但可能不像Python框架那样工作。你必须为一些应用从源码来构建DL4J。


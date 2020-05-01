---
description: 如何可视化、监控和调试神经网络学习。
---

# 可视化

## 内容

* [用DL4J UI可视化网络训练](ke-shi-hua.md#yong-dl-4-j-ui-ke-shi-hua-wang-luo-xun-lian)
  * [DL4J UI：概述](ke-shi-hua.md#dl-4-j-ui-gai-lan-ye)
  * [DL4J UI：模型](ke-shi-hua.md#dl-4-j-ui-mo-xing-ye-mian)
* [DL4J UI与Spark训练](ke-shi-hua.md#dl-4-j-ui-yu-spark-xun-lian)
* [使用 DL4J UI调整你的网络](ke-shi-hua.md#shi-yong-ui-tiao-zheng-wang-luo)
* [TSNE与Word2Vec](ke-shi-hua.md#tsne-yu-word-2-vec)
* [修复UI问题：“No configuration setting”异常](ke-shi-hua.md#xiu-fu-wen-ti-no-configuration-setting-yi-chang)

### 用DL4J UI可视化网络训练

注意：这里的信息属于DL4J版本0.7.0和更高版本。

DL4J在浏览器中提供了一个用户界面来可视化当前的网络状态和训练过程。这个界面通常用于帮助调整神经网络，即选择超参数（例如学习速率）以获得网络的良好性能。

**第1步：向项目中添加DL4J UI依赖项。**

```markup
    <dependency>
        <groupId>org.deeplearning4j</groupId>
        <artifactId>deeplearning4j-ui_2.10</artifactId>
        <version>1.0.0-beta3</version>
    </dependency>
```

请注意 `_2.10`后缀：这是Scala版本（由于使用了Play框架，一个Scala库，用于后端）。如果你没有使用其他Scala库，`_2.10` 或 `_2.11`都可以。

**第2步：启用项目中的用户界面**

这是比较直截了当的：

```java
    //初始化用户界面后端
    UIServer uiServer = UIServer.getInstance();

    //配置要存储网络信息（梯度、分数、时间等）的位置。这里：存储在内存中。
    StatsStorage statsStorage = new InMemoryStatsStorage(); //可选的: new FileStatsStorage(File), 保存用于稍后加载

    // 将StatsStorage实例附加到UI：这允许StatsStorage的内容可视化
    uiServer.attach(statsStorage);

    //然后添加StatsListener来收集网络上的信息
    net.setListeners(new StatsListener(statsStorage));
```

`要访问UI，请打开浏览器，转到http://localhost:9000/train。可以使用org.deeplearning4j.ui.port系统属性设置端口：即，使用端口9001，在启动时将下列内容传递给JVM：-Dorg.deeplearning4j.ui.port=9001`

然后，当你在网络上调用fit\(\)方法时，信息将被收集并路由到UI。

**示例:** [在这里查看UI示例](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/userInterface/UIExample.java)

完整的UI示例集在[这里](https://github.com/deeplearning4j/dl4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/userInterface)是可用的。

#### DL4J UI: 概览页

 ​

概述页面（3个可用页面之一）包含以下信息：

* 左上角：分数与迭代图-这是当前小批量的损失函数的值
* 右上角：模型与训练信息
* 左下角：所有网络权重与迭代的参数更新（逐层）的比率
* 右下角：激活、梯度和更新的标准差（时间）。

请注意，对于下面两个图表，这些图表被显示为值的对数（底数为10）。因此，更新值为-3：参数比率图对应于10的-3次方＝0.001的比率。

参数更新的比率具体地是这些值的平均幅度的比率\(即，log10\(mean\(abs\(updates\)\)/mean\(abs\(parameters\)\) \)。

请参阅本页后面的部分，说明如何在实践中使用这些值。

#### DL4J UI: 模型页面

![](http://upload-images.jianshu.io/upload_images/14495907-eb42257fbdff44b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ​模型页面包含神经网络层的图形，其作为选择机制运行。点击一个图层来显示它的信息。

在右边，在选择图层之后可以得到以下图表：

* 图层信息表
* 更新此层的参数比例，如概述页。这个比率的组成部分（参数和更新平均幅度）也可以通过标签来获得。
* 随着时间的推移，层激活（平均和平均+/- 2标准偏差）
* 每个参数类型的参数和更新的直方图
* 学习速率与时间（注意这将是平的，除非使用学习速调度）

_注：参数标注如下：权重（W）和偏置（B）。对于循环神经网络，W是指连接层到下层的权重，RW指的是循环权重（即，时间步之间的权重）。_

### DL4J UI 与 Spark 训练

DL4J UI可以与Spark一起使用。然而，对于0.7.0版本，冲突依赖意味着在相同的JVM中运行UI和Spark可能是困难的。

有两种选择：

1. 收集并保存相关的统计数据，以便在稍后的时间内可视化（离线）
2. 在单独的服务器中运行UI，并使用远程UI功能将数据从Spark master上传到UI实例。

**收集用于以后离线使用的统计信息**

```java
    SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, conf, tm);

    StatsStorage ss = new FileStatsStorage(new File("myNetworkTrainingStats.dl4j"));
    sparkNet.setListeners(ss, Collections.singletonList(new StatsListener(null)));
```

然后，你可以使用以下方式加载和显示保存的信息：

```java
    StatsStorage statsStorage = new FileStatsStorage(statsFile);//如果文件已经存在：从中加载数据
    UIServer uiServer = UIServer.getInstance();
    uiServer.attach(statsStorage);
```

**使用远程UI功能**

首先，在JVM中用于UI（注意这是服务器）：

```java
    UIServer uiServer = UIServer.getInstance();
    uiServer.enableRemoteListener(); //必要：默认远程支持不启用
```

这将需要`deeplearning4j-ui_2.10`或`deeplearning4j-ui_2.11依赖`。（注意，这不是客户端，这是你的服务器-下面的客户端使用：deeplearning4j-ui-model）

第一，客户端（spark和独立神经网络都使用简单的deeplearning4j-nn）第二，对于你的神经网络（注意，这个例子是用于spark的，但是计算图和多层网络都具有相同的用法，setListeners方法，[这里](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/userInterface/RemoteUIExample.java)找到的示例）：

```java
    SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, conf, tm);

    StatsStorageRouter remoteUIRouter = new RemoteUIStatsStorageRouter("http://UI_MACHINE_IP:9000");
    sparkNet.setListeners(remoteUIRouter, Collections.singletonList(new StatsListener(null)));
```

为了避免与Spark的依赖冲突，你应该使用`deeplearning4j-ui-model`依赖来获得StatsListener，而不是完整的`deeplearning4j-ui_2.10 UI`依赖。

**Scala用户注意事项：**

如果你在一个新的Scala版本上，你需要使用上面的方法。请参阅客户端的链接示例。

注意：你应该用运行用户界面实例的机器的IP地址替换`UI_MACHINE_IP`。

### 使用UI调整网络

这是一个很好的网页，由Andrej Karpathy编写的关于[可视化神经网络训练](https://cs231n.github.io/neural-networks-3/#baby)。第一页是值得阅读和理解的。

调整神经网络往往是一门艺术而不是一门科学。然而，这里有一些有用的想法：

**概述页面-模型得分与迭代图**

分数与迭代应该（总体）随着时间推移而下降。 

* 如果分数持续上升，你的学习率可能会太高。尽量减少，直到分数变得更稳定。
* 增加分数也可以指示其他网络问题，例如不正确的数据归一化。
* 如果分数是平坦的或下降非常缓慢（超过几百次迭代）（a）你的学习率可能太低，（b）你可能在优化上有困难。在后一种情况下，如果使用SGD更新器，则尝试不同的更新程序，如Nesterovs（动量）、RMSProp或Adagrad。
* 注意，未被混洗的数据（即，对于分类，每个小批次只包含一个类）可能导致非常粗糙的或异常的得分与迭代图
* 该线图中的一些噪声是被期待的（即，线将在一个小范围内上下）。但是，如果在运行之间分数相差很大，这可能有问题。
  * 上面提到的问题（学习速率、归一化、数据混洗）可能有助于这一点。
  * 将小批量大小设置为非常小的例子也可以导致有噪声的分数与迭代图，并且可能导致优化困难。

**概述页面和模型页面-使用更新：参数比率图表** 

* 参数更新的平均大小的比值在概述和模型页上提供。
  * “平均级数”=当前时间步中参数或更新的绝对值的平均值
* 这个比率的最重要的用途是选择学习率。作为经验法则：这个比率应该在1:1000＝0.001左右。在（log10）图上，这对应于一个值-3（即10的-3次方）＝0.001）。
  * 请注意，这只是一个粗略的指南，可能不适合所有的网络。然而，这通常是一个很好的起点。
  * 如果比率与此显著不同（例如&gt;-2（即，10的负-2次方=0.01）或&lt;-4（即，10的-4次方=0.0001），则参数可能太不稳定而无法学习有用的特性，或者变化太慢而无法学习有用的特性。
  * 要改变这个比率，调整你的学习速率（或者有时，参数初始化）。在一些网络中，你可能需要对不同的层设置不同的学习速率。
* 注意这个比例的异常大的尖峰：这可能意味着梯度爆炸。

**模型页面：层激活（vs.时间）图**

这个图表可以用于检测消失或爆炸的激活（由于恬当的权重初始化、太多的规则化、缺乏数据归一化或学习率太高）。 

* 这个图表应该理想情况下会随时间稳定（通常是几百次迭代）。
* 一个良好的标准差的激活是在0.5至2的顺序。显著地超出这个范围可以指示出现了上面提到的问题之一。

**模型页：层参数直方图**

仅在最近迭代中显示层参数直方图。 

* 经过一段时间后，对于权重，这些直方图应该有一个近似高斯（正态）分布。
* 对于偏置，这些直方图通常在0开始，并且通常以高斯近似结束。
  * 对此的一个例外是LSTM循环神经网络层：默认情况下，一个（遗忘门 ）的偏置被设置为1.0（默认情况下，尽管这是可配置的），以帮助在长时间段学习依赖性。这导致偏置图最初具有0附近的多个偏置，在1附近具有另一组偏置。
* 注意那些发散到+/-无限的参数：这可能是由于学习率太高或者正则化不足（尝试向网络中添加一些L2正则化）。
* 留意那些变得非常大的偏置。如果类的分布非常不平衡，则有时会出现在用于分类的输出层中。

**模型页：层更新直方图**

只显示最近一次迭代的层更新直方图。 

* 注意，这些是更新，即应用学习速率、动量、正则化等之后的梯度。
* 与参数图一样，这些图应该具有近似高斯（正态）分布。
* 注意非常大的值：这可以指示网络中的爆炸梯度。
  * 爆炸梯度是有问题的，因为它们可以“弄乱”网络的参数。
  * 在这种情况下，它可以指示权重初始化、学习速率或输入/标签数据归一化问题。
  * 在循环神经网络的情况下，添加一些[梯度归一化或梯度裁剪](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/GradientNormalization.java)可能有帮助。

**模型页：参数学习率图表**

这个图表简单地显示了选定层随着时间的推移的参数的学习速率。 ![6.png](https://upload-images.jianshu.io/upload_images/14495907-0497a2ed76f9b907.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果你不使用学习率调度，图表将是平的。如果你使用的是学习率调度，则可以使用这个图表来跟踪随着时间推移的学习率的当前值（对于每个参数）。

## TSNE 与 Word2vec

我们依赖于[TSNE](https://lvdmaaten.github.io/tsne/)，将[单词特征向量](https://deeplearning4j.org/docs/latest/deeplearning4j-nlp-word2vec)的维数降到二维或三维空间。这里有一些使用TSNE与Word2Vec的代码：

```java
log.info("Plot TSNE....");
BarnesHutTsne tsne = new BarnesHutTsne.Builder()
        .setMaxIter(1000)
        .stopLyingIteration(250)
        .learningRate(500)
        .useAdaGrad(false)
        .theta(0.5)
        .setMomentum(0.5)
        .normalize(true)
        .usePca(false)
        .build();
vec.lookupTable().plotVocab(tsne);
```

## 修复问题: “No configuration setting” 异常

DL4J UI可能发生的异常如下：

```java
com.typesafe.config.ConfigException$Missing: No configuration setting found for key 'play.crypto.provider'
        at com.typesafe.config.impl.SimpleConfig.findKeyOrNull(SimpleConfig.java:152)
        at com.typesafe.config.impl.SimpleConfig.findOrNull(SimpleConfig.java:170)
        ...
        at play.server.Server.forRouter(Server.java:96)
        at org.deeplearning4j.ui.play.PlayUIServer.runMain(PlayUIServer.java:206)
        at org.deeplearning4j.ui.api.UIServer.getInstance(UIServer.java:27)
```

![image.gif](https://upload-images.jianshu.io/upload_images/14495907-58ad5e17e51a7eb2.gif?imageMogr2/auto-orient/strip)

这个异常不是直接由于DL4J造成的，而是由于缺少application.conf文件，这个文件是Play框架（DL4J的UI所基于的库）需要的。这最初存在于deeplearning4j-play依赖项中：但是，如果uber-jar（即，具有依赖项的JAR文件）（例如，通过mvn包）被构建，uber-jar则可能无法被正确地复制。例如，使用`maven-assembly-plugin已经对`一些用户造成了这个异常。

推荐的解决方案（对于Maven）是使用Maven Shade插件生成Uber jar，配置如下：

```markup
    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>${exec-maven-plugin.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>exec</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <executable>java</executable>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>${maven-shade-plugin.version}</version>
                <configuration>
                    <shadedArtifactAttached>true</shadedArtifactAttached>
                    <shadedClassifierName>${shadedClassifier}</shadedClassifierName>
                    <createDependencyReducedPom>true</createDependencyReducedPom>
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <!--<exclude>org/datanucleus/**</exclude>-->
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>

                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                    <resource>reference.conf</resource>
                                </transformer>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer" />
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        <plugins>
    <build>
```

然后，用MVN包创建你的uber.jar，并通过cd 目标 和 `java -cp dl4j-examples-0.9.1-bin.jar org.deeplearning4j.examples.userInterface.UIExample`。 注意生成的JAR文件的“-bin”后缀：这包括所有依赖项。

还要注意，这个Maven Shade方法是为DL4J的示例库配置的。


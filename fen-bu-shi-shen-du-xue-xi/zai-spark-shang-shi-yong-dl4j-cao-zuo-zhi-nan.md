# 在Spark上使用DL4J：操作指南

此页包含一些常见分布式训练任务的操作指南。请注意，有关构建数据管道的指南，请参见此处。 

在阅读这些指南之前，请确保您已经阅读了此处的DL4J Spark训练入门指南。

训练前指南

* 如何使用Maven通过Spark submit构建用于训练的uber JAR
* 如何利用GPU进行Spark训练
* 如何在master上使用CPU，在workers上使用GPU
* 如何为Spark配置内存设置
* 如何为workers配置垃圾回收
* 如何与DL4J和ND4J一起使用Kryo序列化
* 如何使用YARN和GPU
* 如何配置Spark本地配置

训练时与训练后指南

* 如何配置编码阈值
* 如何进行分布式测试集评估
* 如何保存（和加载）Spark训练的神经网络
* 如何进行分布式推理

问题和故障排除指南

* 如何调试常见的Spark依赖问题（NoClassDefFoundException和类似）
* 如何修复“Error querying NTP server”错误
* 如何安全缓存RDD\[INDArray\]和RDD\[DataSet\]
* 在Ubuntu16.04上训练失败（Ubuntu Bug可能会影响DL4J Spark用户）

## 训练前指南

### 如何使用Maven通过Spark submit构建用于训练的uber JAR

当向集群提交一个训练作业时，一个典型的工作流是构建一个提交给Spark submit的“uber jar”。uber jar是包含运行作业所需的所有依赖项（库、类文件等）的单个jar文件。请注意，Spark submit是一个带有Spark发行版的脚本，用户将其作业（以JAR文件的形式）提交到该发行版，以便开始执行其Spark作业。

本指南假设您已经设置了用于在Spark上训练网络的代码。

**步骤1：决定所需的依赖项。**

与DL4J和ND4J的单机训练有很多重叠。例如，对于单机训练和Spark训练 ，您应该包括标准的deeplearning4j依赖项集，例如：

* deeplearning4j-core
* deeplearning4j-spark
* nd4j-native-platform \(仅用于CPU训练\)

此外，您还需要包括Deeplearning4j的Spark模块、`dl4j-spark_2.10`或`dl4j-spark_2.11`。本模块是开发和执行Deeplearning4j Spark作业所必需的。小心使用与集群匹配的spark版本-spark版本（Spark 1 vs.Spark 2）和Scala版本（2.10 vs.2.11）。如果这些不匹配，则您的作业可能在运行时失败。

依赖示例: Spark 2, Scala 2.11:

```markup
<dependency>
  <groupId>org.deeplearning4j</groupId>
  <artifactId>dl4j-spark_2.11</artifactId>
  <version>1.0.0-beta2_spark_2</version>
</dependency>
```

依赖示例, Spark 1, Scala 2.10:

```markup
<dependency>
  <groupId>org.deeplearning4j</groupId>
  <artifactId>dl4j-spark_2.10</artifactId>
  <version>1.0.0-beta2_spark_1</version>
</dependency>
```

请注意，如果添加Spark依赖项，如spark-core\_2.11，则可以在pom.xml中设置scope为`provided` scope（有关更多详细信息，请参阅[Maven文档](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Scope)），因为Spark submit将添加Spark到类路径。在集群上执行时不需要添加此依赖项，但如果要在本地计算机上测试或调试基于Spark的作业，则可能需要添加此依赖项。

在使用CUDA GPU进行训练时，在添加CUDA依赖项时有两种可能的情况：

**案例1：集群节点在主节点和工作节点上安装了CUDA工具包**

当CUDA工具包和CuDNN在集群节点上可用时，我们可以使用更小的依赖：

* 如果构建uber-jar的操作系统与集群的操作系统相同：包含 nd4j-cuda-x.x
* 如果构建uber jar的操作系统与集群操作系统不同（即，在Windows上构建，在Linux集群上执行Spark）：包含 nd4j-cuda-x.x-platform
* 在这两种情况下，包括x.x是CUDA版本的地方——例如，x.x=9.2是CUDA 9.2。

**案例2：集群节点没有在主节点和工作节点上安装CUDA工具包**

当群集节点上未安装CUDA/CuDNN时，我们可以执行以下操作：

* 首先，按照上面的“案例1”包含依赖项
* 然后包括集群操作系统的“redist” javacpp-presets，如下所述：[DL4J CuDNN Docs](deeplearning4j-config-cudnn)

**步骤2：配置pom.xml文件以构建uber jar**

使用Spark submit时，您需要一个uber jar来提交以启动和运行您的作业。在步骤1中配置了相关的依赖项之后，我们需要配置pom.xml文件来正确构建uber-jar。

我们建议您使用maven shade插件来构建uber-jar。有其他工具/插件可以用于此目的，但这些工具/插件并不总是包含源jar中的所有相关文件，例如Java的ServiceLoader机制正常运行所需的文件。（ServiceLoader机制被ND4J和许多其他软件库使用）。

独立示例项目[pom.xml](https://github.com/eclipse/deeplearning4j-examples/blob/master/standalone-sample-project/pom.xml)文件中提供了适用于此目的的Maven shade配置：

```markup
<build>
    <plugins>
        <!-- 如果需要，这里是其它插件 -->

        <!-- 配置maven shade在运行"mvn package"时生成一个uber-jar -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>${maven-shade-plugin.version}</version>
            <configuration>
                <shadedArtifactAttached>true</shadedArtifactAttached>
                <shadedClassifierName>bin</shadedClassifierName>
                <createDependencyReducedPom>true</createDependencyReducedPom>
                <filters>
                    <filter>
                        <artifact>*:*</artifact>
                        <excludes>
                            <exclude>org/datanucleus/**</exclude>
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
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**步骤3: 构建 uber jar**

最后，打开一个命令行窗口（Linux上的bash、Windows上的cmd等），只需运行`mvn package-DskipTests`为您的项目构建uber-jar。注意，uber-jar应该出现在`<project_root>/target/<project_name>-bin.jar`下。一定要使用大的…-bin.jar文件，因为这是一个带有所有依赖项的着色jar文件。 

这是-你现在应该有一个uber jar，适合提交到Spark submit，用于spark上与cpu或NVIDA（CUDA）gpu的训练网络。

### 如何利用GPU进行Spark训练

DL4J和ND4J使用NVIDA GPU支持GPU加速。DL4J Spark训练也可以使用GPU执行。

DL4J和ND4J的设计方式使得代码（神经网络配置，数据管道代码）是“后端独立的”。也就是说，您可以只编写一次代码，然后在CPU或GPU上执行它，只需包含适当的后端（nd4j-native用于CPU，nd4j-cuda-x.x用于GPU）。在Spark上执行与在单个节点上执行在这方面没有区别：您只需要包括适当的ND4J后端，并确保您的计算机（在本例中是主/工作节点）与CUDA库进行了适当的设置（有关在CUDA上运行而无需在每个节点上安装CUDA/cuDNN的信息，请参阅uber-jar指南）。

在GPU上运行时，有几个组件：（a）ND4J CUDA后端（nd4j-cuda-x.x 依赖项）（b）CUDA工具包（c）获得cuDNN支持的Deeplearning4j CUDA依赖项（deeplearning4j-cuda-x.x）（d）cuDNN库文件

（a）和（b）都必须可供ND4J/DL4J使用可用的CUDA GPU运行。（c） 和（d）是可选的，但建议获得最佳性能-NVIDIA的cuDNN库能够显著加快许多层的训练，例如卷积层（ConvolutionLayer, SubsamplingLayer, BatchNormalization等）和LSTM RNN层。

有关为Spark作业配置依赖项的信息，请参阅上面的uber-jar部分。要在单个节点上配置cuDNN，请参阅将Deeplearning4j与cuDNN一起使用

### 如何在master上使用CPU，在workers上使用GPU

在某些情况下，只使用CPU运行master和使用GPU运行workers可能是有意义的。如果资源（即，可用GPU机器的数量）不受限制，那么使用同构集群可能更容易：即，设置集群，以便主机也使用GPU执行。

假设master/driver在CPU机器上执行，而workers在GPU机器上执行，那么您可以简单地包括两个后端（即nd4j-cuda-x.x和nd4j-native依赖项，如uber-jar部分所述）。

当类路径上存在多个后端时，默认情况下将首先尝试CUDA后端。如果无法加载，则将第二次加载CPU（nd4j-native）后端。因此，如果驱动程序没有GPU，它应该回到使用CPU。但是，可以通过在master/driver上设置`BACKEND_PRIORITY_CPU`或`BACKEND_PRIORITY_GPU`环境变量来更改此默认行为，如[此处](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-common/src/main/java/org/nd4j/config/ND4JEnvironmentVars.java)所述。设置环境变量的确切过程可能取决于集群管理器-Spark 独立 vs.YARN vs.Mesos。关于如何为driver/master设置Spark作业的环境变量，请参考每个相应的文档。

### 如何为Spark配置内存设置

有关DL4J和ND4J的内存和内存配置工作原理的重要背景，请从阅读[ND4J/DL4J的内存管理](../pei-zhi/nei-cun-guan-li.md)开始。

Spark上的内存管理类似于单节点训练的内存管理：

* 堆内存是使用标准Java Xms和Xmx内存配置设置配置的
* 堆外内存是使用javacpp系统属性配置的

然而，Spark上下文中的内存配置增加了一些额外的复杂性：1、通常，对于driver/master和workers序必须分别进行内存配置（有时使用不同的机制）。2、配置内存的方法可以依赖于集群资源管理器-Spark Standalone 和YARN 和Mesos，等等 。3、群集资源管理器默认内存设置通常不适用于严重依赖堆外内存的库（如DL4J/ND4J） 

请参阅群集管理器的Spark文档：

* [YARN](https://spark.apache.org/docs/latest/running-on-yarn.html)
* [Mesos](https://spark.apache.org/docs/latest/running-on-mesos.html)
* [Spark Standalone](https://spark.apache.org/docs/latest/spark-standalone.html)

你应该设置4个内容：1、堆内存上的worker（Xmx）-通常设置为Spark submit的参数（例如，`--executor-memory 4g` 用于 YARN）2、worker堆外内存（javacpp系统属性选项）（例如，`--conf "spark.executor.extraJavaOptions=-Dorg.bytedeco.javacpp.maxbytes=8G"`）。3、堆内存上的驱动程序-通常设置为。4、驱动程序堆外内存 

一些注释：

* 在YARN上，通常需要设置`spark.yarn.driver.memoryOverhead`和`spark.yarn.executor.memoryOverhead`属性。对于DL4J训练，默认设置太小。
* 在Spark standalone上，还可以通过修改每个节点上的`conf/spark-env.s`文件来配置内存，如[Spark配置文档](https://spark.apache.org/docs/latest/configuration.html#environment-variables)中所述。例如，可以添加以下行来设置driver的8GB堆内存、driver的12 GB堆外内存、workers的12GB堆内存和18GB堆外内存：
  * `SPARK_DRIVER_OPTS=-Dorg.bytedeco.javacpp.maxbytes=12G`
  * `SPARK_DRIVER_MEMORY=8G`
  * `SPARK_WORKER_OPTS=-Dorg.bytedeco.javacpp.maxbytes=18G`
  * `SPARK_WORKER_MEMORY=12G`

总的来说，这可能看起来像（对于YARN，有4GB的堆内存，5GB的堆外内存，6GB的YARN堆外开销）：

```text
--class my.class.name.here --num-executors 4 --executor-cores 8 --executor-memory 4G --driver-memory 4G --conf "spark.executor.extraJavaOptions=-Dorg.bytedeco.javacpp.maxbytes=5G" --conf "spark.driver.extraJavaOptions=-Dorg.bytedeco.javacpp.maxbytes=5G" --conf spark.yarn.executor.memoryOverhead=6144
```

### 如何为workers配置垃圾回收

训练效果的一个决定因素是垃圾收集的频率。使用默认启用的[工作间](../pei-zhi/nei-cun-guan-li.md)（另请参阅[这里](../pei-zhi/nei-cun-gong-zuo-jian.md)）时，减少垃圾收集的频率可能会有所帮助。对于简单的机器训练（在driver上），这很简单：

训练效果的一个决定因素是垃圾收集的频率。使用默认启用的[工作区](../pei-zhi/nei-cun-guan-li.md)（另请参[此处](../pei-zhi/nei-cun-gong-zuo-jian.md)）时，减少垃圾收集的频率可能会有所帮助。对于简单的机器训练（以及对driver的训练），这很简单：

```java

//这将gc调用的频率限制为5000毫秒
Nd4j.getMemoryManager().setAutoGcWindow(5000)

//或者你可以完全禁用它
Nd4j.getMemoryManager().togglePeriodicGc(false);
```

但是，在driver上设置此项不会更改workers上的设置。相反，可以为workers设置如下：

```java
new SharedTrainingMaster.Builder(voidConfiguration, minibatch)
    <other configuration>
     //定期垃圾回收已启用。。。
    .workerTogglePeriodicGC(true)     
     //…并配置为每5秒执行一次（每5000毫秒）
     .workerPeriodicGCFrequency(5000)  
    .build();
```

默认值（从1.0.0-beta3开始）是每5秒对workers执行一次定期垃圾收集。

### 如何与DL4J和ND4J一起使用Kryo序列化

DL4J和ND4J可以利用Kryo序列化，配置适当。请注意，由于INDArrays的堆外内存，与在其他上下文中使用Kryo相比，Kryo提供的性能优势要小一些。 

要启用Kryo序列化，首先添加[nd4j-kryo依赖项](https://search.maven.org/search?q=nd4j-kryo)：

```markup
<dependency>
  <groupId>org.nd4j</groupId>
  <artifactId>nd4j-kryo_2.11</artifactId>
  <version>${dl4j-version}</version>
</dependency>
```

其中`${dl4j-version}`是用于DL4J和ND4J的版本。 

然后，在训练工作开始时，添加以下代码：

```java
    SparkConf conf = new SparkConf();
    conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer");
    conf.set("spark.kryo.registrator", "org.nd4j.Nd4jRegistrator");
```

注意，当使用DL4J的SparkDl4jMultiLayer或SparkComputationGraph类时，如果Kryo配置不正确，将记录一个警告。

### 如何使用YARN和GPU

对于DL4J，CUDA GPU的唯一要求是使用适当的后端，在每个节点上安装适当的NVIDIA库，或者在uber-JAR中提供（有关更多详细信息，请参阅Spark操作指南）。对于最新版本的YARN，在某些情况下可能需要一些额外的配置-有关更多详细信息，请参阅[YARN GPU文档](https://hadoop.apache.org/docs/r3.1.0/hadoop-yarn/hadoop-yarn-site/UsingGpus.html)。

 早期版本的YARN（例如，2.7.x和类似版本）本地不支持GPU。对于这些版本，可以利用节点标签来确保将作业调度到仅GPU的节点上。有关更多详细信息，请参见[Hadoop YARN文档](https://hadoop.apache.org/docs/r2.7.3/hadoop-yarn/hadoop-yarn-site/NodeLabel.html)。

 请注意，还需要特定于YARN的内存配置（请参阅[内存操作指南](../pei-zhi/nei-cun-guan-li.md)）。

### 如何配置Spark本地配置

配置Spark位置设置是一个可选的配置选项，可以提高训练性能。

小结：在Spark submit 配置中添加`--conf spark.locality.wait=0`可能会稍微减少训练时间，方法是将network fit操作安排为更早开始。

 有关详细信息，请参见[链接1](https://spark.apache.org/docs/latest/tuning.html#data-locality)和[链接2](https://spark.apache.org/docs/latest/configuration.html#scheduling)。

## 训练时与训练后指南

### 如何配置编码阈值

DL4J的Spark实现使用阈值编码方案在节点之间发送参数更新。这种编码方案产生一个小的量化消息，这大大降低了网络通信更新的成本。有关此编码过程的详细信息，请参阅[技术说明](ji-shu-shuo-ming.md)页。 

这个阈值编码过程引入了一个“分布式训练特定”超参数-编码阈值。太大的阈值和太小的阈值都可能导致次优性能：

* 较大的阈值意味着不频繁的通信-太不频繁收敛可能会受到影响
* 较小的阈值意味着更频繁的通信，但在每个步骤中都会进行较小的更改

要使用的编码阈值由 [ThresholdAlgorithm](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/solvers/accumulation/encoding/ThresholdAlgorithm.java)控制。ThresholdAlgorithm的具体实现决定了应该使用什么阈值。

 DL4J的默认行为是使用 [AdaptiveThresholdAlgorithm](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/solvers/accumulation/encoding/threshold/AdaptiveThresholdAlgorithm.java) 算法，该算法尝试将稀疏率保持在一定范围内。

* 稀疏比定义为numValues（encodedUpdate）/numParameters-1.0表示完全密集（所有值都已传递），0.0表示完全稀疏（没有值传递）
* 较大的阈值意味着更多的稀疏值（更少的网络通信），较小的阈值意味着更少的稀疏值（更多的网络通信）
* 默认情况下，AdaptiveThresholdAlgorithm

  尝试将稀疏度比率保持在0.01到0.0001之间。如果更新的稀疏性不在此范围内，则阈值将增加或减少，直到它在此范围内为止。

* 仍然需要设置初始阈值-我们发现

在实践中，我们已经看到了这种自适应阈值过程能够很好地工作。阈值算法的内置实现包括：

* AdaptiveThresholdAlgorithm
* [FixedThresholdAlgorithm](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/solvers/accumulation/encoding/threshold/FixedThresholdAlgorithm.java): 使用指定编码阈值的固定、非自适应阈值。
* [TargetSparsityThresholdAlgorithm](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/solvers/accumulation/encoding/threshold/TargetSparsityThresholdAlgorithm.java): 一种自适应阈值算法，针对特定的稀疏性，增加或减少阈值以尝试匹配目标。

此外，DL4J还有一个[ResidualPostProcessor](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/solvers/accumulation/encoding/ResidualPostProcessor.java)接口，默认实现是[ResidualClippingPostProcessor](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/optimize/solvers/accumulation/encoding/residual/ResidualClippingPostProcessor.java)，它每5步将残余向量裁剪到当前阈值的最大5倍。这样做的动机是，更新的“残余”部分（即，那些未通信的部分）存储在残余向量中。如果更新远远大于阈值，我们可以有一个我们称之为“残余爆炸”的现象，也就是说，残余值可以继续增长到阈值的许多倍（因此需要采取许多步骤来传达梯度）。为了避免这种现象，采用了残余后处理器。 

阈值算法（和初始阈值）和残余后处理器可以设置如下：

```java
TrainingMaster tm = new SharedTrainingMaster.Builder(voidConfiguration, minibatch)
    .thresholdAlgorithm(new AdaptiveThresholdAlgorithm(this.gradientThreshold))
    .residualPostProcessor(new ResidualClippingPostProcessor(5, 5))
    <other config>
    .build();
```

最后，DL4J的SharedTrainingMaster还有一个编码调试模式，通过在SharedTrainingMaster builder中设置`.encodingDebugMode(true)`来启用。启用此选项后，每个工作节点进程都将记录当前阈值、稀疏性和有关编码的各种其他统计信息。这些统计数据可用于确定是否适当设置了阈值：例如，许多更新是阈值的数十倍或数百倍，可能表明阈值太低，应增加阈值；在频谱的另一端，非常稀疏的更新（少于10000个值中的一个值被传送）可能表示应该降低阈值。

### 如何进行分布式测试集评估

DL4J支持神经网络的大多数标准评估指标。有关评估的基本信息，请参阅[DL4J评估页](../tiao-you-yu-xun-lian/ping-gu.md)。 

DL4J支持的所有评估指标都可以使用Spark以分布式方式计算。

**步骤1：准备数据**

Spark上的DL4J评估数据与训练数据非常相似。也就是说，您可以使用：

* `RDD<DataSet>` 或 `JavaRDD<DataSet>` 用于评估单输入/输出网络
* `RDD<MultiDataSet>` 或 `JavaRDD<MultiDataSet>` 用于评估多输入/输出网络
* `RDD<String>` 或 `JavaRDD<String>` HDFS.其中每个字符串都是指向网络存储（如HDFS）上序列化DataSet/MultiDataSet（或其他基于小批量文件的格式）的路径。

**步骤2：准备网络**

创建你的网络很简单。首先，使用以下指南中的信息将网络（多层网络或计算图）加载到driver的内存中：如何保存（和加载）在Spark上训练的神经网络

然后，只需使用以下方法创建网络：

```java
JavaSparkContext sc = new JavaSparkContext();
MultiLayerNetwork net = <code to load network>
SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, cgForEval, null);
```

```java
JavaSparkContext sc = new JavaSparkContext();
ComputationGraph net = <code to load network>
SparkComputationGraph sparkNet = new SparkComputationGraph(sc, net, null);
```

注意，您不需要配置TrainingMaster（即上面的第三个参数为空），因为评估不使用它。

**步骤3：调用适当的评估方法**

对于常见情况，可以调用SparkDl4jMultiLayer或SparkComputationGraph上的标准评估方法之一：

```java
//分类器准确度/F1等
evaluate(RDD<DataSet>)               
//分类器准确度/F1等
evaluate(JavaRDD<DataSet>)            
//单输出二进制分类器的ROC
evaluateROC(JavaRDD<DataSet>)        
//用于评估度量
evaluateRegression(JavaRDD<DataSet>) 
```

要同时执行多个评估（比按顺序执行效率更高），可以使用以下方法：

```java
IEvaluation[] evaluations = new IEvaluation[]{new Evaluation(), new ROCMultiClass()};
JavaRDD<DataSet> data = ...;
sparkNet.doEvaluation(data, 32, evaluations);
```

请注意，某些计算方法具有带额外参数的重载，包括：

* `int evalNumWorkers` - 评估工作节点数-即每个节点上用于评估的网络副本数（最多为每个工作节点的最大Spark线程数）。对于大型网络（或有限的群集内存），您可能希望减少这个值，以避免遇到内存问题。
* `int evalBatchSize` - 执行评估时要使用的小批量大小。这需要足够大以有效地使用硬件资源，但要足够小以不耗尽内存。32-128的值无疑是一个很好的起点；当有更多可用内存和较小的网络时增加；如果内存有问题，则减少。
* `DataSetLoader loader` 和 `MultiDataSetLoader loader` - 这些将在 `RDD<String>` 或  `JavaRDD<String>`  上评估时可用。它们是使用自定义用户定义函数将路径加载到DataSet或MultiDataSet

  的接口。大多数用户不需要使用这些功能，但是提供的功能具有更大的灵活性。例如，如果保存的minibatch文件格式不是DataSet/MultiDataSet，而是其他（可能是自定义）格式，则将使用它们。

Finally, if you want to save the results of evaluation \(of any type\) you can save it to JSON format directly to remote storage such as HDFS as follows:最后，如果要保存评估结果（任何类型），可以直接将其保存为JSON格式，保存到远程存储，如HDFS，如下所示：

```java
JavaSparkContext sc = new JavaSparkContext();
Evaluation eval = ...
String json = eval.toJson();
String writeTo = "hdfs:///output/directory/evaluation.json";
//还支持本地文件路径 - file://
SparkUtils.writeStringToFile(writeTo, json, sc); 
```

 `SparkUtils` 的导入是 `org.datavec.spark.transform.utils.SparkUtils`

可以使用以下方法加载评估：

```java
String json = SparkUtils.readStringFromFile(writeTo, sc);
Evaluation eval = Evaluation.fromJson(json);
```

### 如何保存（和加载）Spark训练的神经网络

DL4J的Spark功能是围绕包装类的思想构建的，即`SparkDl4jMultiLayer`和`SparkComputationGraph`在内部使用标准的`MultiLayerNetwork` 和  `ComputationGraph`类。您可以分别使用SparkDl4jMultiLayer.getNetwork\(\)和SparkComputationGraph.getNetwork\(\)访问内部MultiLayerNetwork/ComputationGraph类。

若要保存在主节点/driver的本地文件系统上，请获取如上所述的网络，然后仅使用ModelSerializer类或`MultiLayerNetwork.save(File)/.load(File)` 和  `ComputationGraph.save(File)/.load(File)`方法

要保存到（或从其中加载）远程位置或分布式文件系统（如HDFS）中，可以使用输入和输出流。

 例如，

```java
JavaSparkContext sc = new JavaSparkContext();
FileSystem fileSystem = FileSystem.get(sc.hadoopConfiguration());
String outputPath = "hdfs:///my/output/location/file.bin";
MultiLayerNetwork net = sparkNet.getNetwork();
try (BufferedOutputStream os = new BufferedOutputStream(fileSystem.create(new Path(outputPath)))) {
    ModelSerializer.writeModel(net, os, true);
}
```

读取是类似的流程：

```java
JavaSparkContext sc = new JavaSparkContext();
FileSystem fileSystem = FileSystem.get(sc.hadoopConfiguration());
String outputPath = "hdfs:///my/output/location/file.bin";
MultiLayerNetwork net;
try(BufferedInputStream is = new BufferedInputStream(fileSystem.open(new Path(outputPath)))){
    net = ModelSerializer.restoreMultiLayerNetwork(is);
}
```

### 如何进行分布式推理

DL4J的Spark实现支持分布式推理。也就是说，我们可以使用机器集群轻松地在输入的RDD上生成预测。这种分布式推理也可用于在单机上训练并用Spark加载的网络（有关如何用Spark加载保存网络的详细信息，请参阅保存/加载部分）。 

注意：如果要执行评估（即计算准确率、F1、MSE等），请参考[评估操作指南](../tiao-you-yu-xun-lian/ping-gu.md)。 

用于执行分布式推理的方法签名如下：

```java
SparkDl4jMultiLayer.feedForwardWithKey(JavaPairRDD<K, INDArray> featuresData, int batchSize) : JavaPairRDD<K, INDArray>
SparkComputationGraph.feedForwardWithKey(JavaPairRDD<K, INDArray[]> featuresData, int batchSize) : JavaPairRDD<K, INDArray[]>
```

如果需要，也有接受输入掩码数组的重载

注意参数`K`-这是一个泛型类型，表示用于标识每个示例的唯一“key”。键值不作为推理过程的一部分使用。这个键是必需的，因为Spark的RDD是无序的-没有这个键，我们就无法知道预测RDD中的哪个元素对应于输入RDD中的哪个元素。在执行推理时，batch size参数用于指定小批量大小。它不影响返回的值，而是用来平衡内存使用和计算效率：大批量的计算可能会更快，但需要更多的内存。在许多情况下，如果您不确定要使用什么，那么批处理大小为64是一个很好的尝试起点。

## 问题和故障排除指南

### 如何调试常见的Spark依赖问题（NoClassDefFoundException和类似）

不幸的是，如果您的项目配置不正确，则在运行时群集上可能会发生依赖关系问题。这些问题可以在任何Spark作业中发生，而不仅仅是那些使用DL4J的作业，它们可能是由类路径上的其他依赖项或库引起的，而不是由DL4J依赖项引起的。

当发生依赖性问题时，它们通常会产生如下异常：

* NoSuchMethodException
* ClassNotFoundException
* AbstractMethodError

例如，不匹配的Spark版本（尝试在Spark 2群集上使用Spark 1）可能看起来像：

```text
java.lang.AbstractMethodError: org.deeplearning4j.spark.api.worker.ExecuteWorkerPathMDSFlatMap.call(Ljava/lang/Object;)Ljava/util/Iterator;
```

另一类错误是`UnsupportedClassVersionError`，例如`java.lang.UnsupportedClassVersionError:XYZ:Unsupported major.minor version 52.0`-这可能是由于试图在仅使用java 7 JRE/JDK设置的集群上运行（例如）java 8代码所致。

如何调试依赖问题：

**步骤1：收集依赖项信息**

第一步（使用Maven时）是生成一个可以参考的依赖树。打开一个命令行窗口（例如，Linux上的bash，Windows上的cmd），导航到Maven项目的根目录并运行mvn dependency:tree这将为您提供一个依赖项列表（直接的和暂时的），这有助于准确理解类路径上的内容和原因。 

还要注意，`mvn dependency:tree  -Dverbose`将提供额外的信息，并且在调试与不匹配的库版本相关的问题时非常有用。

**步骤2：检查你的Spark版本**

当遇到依赖性问题时，请检查以下内容。 

首先：检查Spark版本如果集群运行的是Spark 2，那么应该使用以`_spark_2`结尾的deeplearning4j-spark\_2.10/2.11（和DataVec）版本 

看穿 

如果发现问题，应按如下方式更改项目依赖项：在Spark 2（Scala 2.11）群集上，使用：

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>dl4j-spark_2.11</artifactId>
    <version>1.0.0-beta2_spark_2</version>
</dependency>
```

而在Spark 1（Scala 2.11）集群上，应该使用：

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>dl4j-spark_2.11</artifactId>
    <version>1.0.0-beta2_spark_1</version>
</dependency>
```

**步骤3：检查Scala版本**

Apache Spark的发行版本同时支持Scala 2.10和scala2.11。

为了避免Scala版本出现问题，您需要做两件事：（a）确保您的项目类路径上没有Scala 2.10和scala2.11（或2.12）依赖项的混合。检查依赖关系树中以`_2.10`或`_2.11`结尾的条目：例如，`org.apache.spark:spark-core_2.11:jar:1.6.3:compile`是一个使用Scala 2.11的spark 1（1.6.3）依赖关系（b）确保您的项目与集群使用的匹配。例如，如果集群使用Scala 2.11运行spark 2，那么所有Scala依赖项也应该使用2.11。请注意，Scala 2.11在Spark集群中更为常见。

如果发现不匹配的Scala版本，则需要通过更改pom.xml（或其他依赖项管理系统的类似配置文件）中的依赖项版本来对齐它们。许多库（包括Spark和DL4J）都发布了Scala 2.10和2.11版本的依赖项。

**步骤4：检查不匹配的库版本**

在Java生态系统中广泛使用的一些公共实用程序库在不同版本之间不兼容。例如，Spark可能依赖于库 x 版本 y，当库 x 版本 z位于类路径上时，Spark将无法运行。此外，这些库中的许多被分为多个模块（即多个独立的模块依赖项），在混合不同版本时无法正常工作。

一些常见的问题包括：

* Jackson
* Guava

DL4J和ND4J使用这些库的版本，应该避免与Spark的依赖冲突。但是，其他（第三方库）可能会引入这些依赖项的版本。 

通常，异常会提示查找位置，即堆栈跟踪可能包含一个特定的类，该类可用于标识有问题的库。

**步骤5：一旦确定，修复依赖冲突**

要调试这些问题，请仔细检查依赖树（`mvn dependency:tree -Dverbose`的输出）。如有必要，可以使用[exclusions](https://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html)或将有问题的依赖项作为直接依赖项添加到问题中以强制其版本。为此，需要将所需版本的依赖项直接添加到项目中。通常，这足以解决问题。

请记住，在使用Spark submit时，Spark将向driver和工作节点类路径添加Spark及其依赖库的副本。这意味着，对于Spark添加的依赖项，不能简单地在项目中排除它们，Spark submit将在运行时添加它们，无论您是否在项目中排除它们。

另一个值得了解的设置是（实验性的）Spark配置选项`spark.driver.userClassPathFirst` 和  `spark.executor.userClassPathFirst`（有关更多详细信息，请参阅[Spark配置文档](https://spark.apache.org/docs/latest/configuration.html)）。在某些情况下，这些选项可能会解决依赖性问题。

### 如何安全缓存RDD\[INDArray\]和RDD\[DataSet\]

Spark在如何处理带有大型堆外组件的Java对象方面存在一些问题，例如DL4J中使用的DataSet和INDArray对象。本节解释与缓存/持久化这些对象相关的问题。 

要知道的要点是：

* 由于Spark没有正确估计RDD中对象的大小，因此MEMORY\_ONLY和MEMORY\_AND\_DISK

  持久性可能会对堆外内存造成问题。这可能导致堆外内存问题。

* 当持久化一个  `RDD<DataSet>` 或 `RDD<INDArray>` 用于重用，使用 MEMORY\_ONLY\_SER 或 MEMORY\_AND\_DISK\_SER

**为什么MEMORY\_ONLY\_SER或MEMORY\_AND\_DISK\_SER被推荐**

Apache Spark提高性能的方法之一是允许用户在内存中缓存数据。这可以使用`RDD.cache()`或`RDD.persist(StorageLevel.MEMORY_ONLY())`以反序列化（即标准Java对象）的形式将内容存储在内存中。基本思想很简单：如果您持久化一个RDD，您可以从内存（或磁盘，取决于配置）中重用它，而不必重新计算它。然而，大型RDD可能不能完全装入内存。在这种情况下，RDD的某些部分必须重新计算或从磁盘加载，具体取决于使用的存储级别。此外，为了避免使用太多内存，Spark会在需要时丢弃RDD的部分（块）。

Spark中可用的主要存储级别如下所示。有关这些的说明，请参阅[Spark编程指南](https://spark.apache.org/docs/1.6.2/programming-guide.html#rdd-persistence)。

* MEMORY\_ONLY
* MEMORY\_AND\_DISK
* MEMORY\_ONLY\_SER
* MEMORY\_AND\_DISK\_SER
* DISK\_ONLY

Spark的问题是如何处理内存。特别是，Spark将根据一个RDD（块）的估计大小丢弃该块的一部分。Spark估计块大小的方式取决于持久性级别。对于`MEMORY_ONLY`和`MEMORY_AND_DISK` 持久化 ，这是通过遍历Java对象图来完成的，即查看对象中的字段并递归地估计这些对象的大小。然而，这个过程没有考虑到DL4J或ND4J使用的堆外内存。对于像DataSet和INDArray这样的对象（它们几乎完全存储在堆外），Spark大大低估了使用这个过程的对象的真实大小。此外，Spark在决定是否保留或丢弃块时只考虑堆上内存的使用量。由于DataSet和INDArray对象的堆上大小非常小，Spark会将它们中的太多对象保留在`MEMORY_ONLY`和`MEMORY_AND_DISK`上，从而导致堆外内存耗尽，导致内存不足问题。 

但是，对于`MEMORY_ONLY_SER`和`MEMORY_AND_DISK_SER` Spark，它将块以序列化的形式存储在Java堆中。Spark可以准确估计以序列化形式存储的对象的大小（序列化对象没有堆外内存组件），因此Spark将在需要时丢弃块，从而避免任何内存不足问题。

### 如何修复“Error querying NTP server”错误

DL4J的参数平均实现可以选择使用SparkDl4jMultiLayer.setCollectTrainingStats\(true\)来收集训练状态。启用时，需要internet访问才能连接到NTP（网络时间协议）服务器。 

可能会出现`NTPTimeSource: Error querying NTP server, attempt 1 of 10`。有时，这些失败是暂时的（稍后的重试将起作用），可以忽略。但是，如果Spark集群的配置使一个或多个工作进程无法访问internet（特别是NTP服务器），则所有重试都可能失败。 

有两种解决方案：

1. 不使用 `sparkNet.setCollectTrainingStats(true)` -此功能是可选的（训练时不需要），默认情况下禁用
2. 将系统设置为使用本地计算机时钟而不是NTP服务器作为时间源（但请注意，时间线信息可能因此非常不准确） 要使用系统时钟时间源，请在Spark submit中添加以下内容：

   ```text
   --conf spark.driver.extraJavaOptions=-Dorg.deeplearning4j.spark.time.TimeSource=org.deeplearning4j.spark.time.SystemClockTimeSource
   --conf spark.executor.extraJavaOptions=-Dorg.deeplearning4j.spark.time.TimeSource=org.deeplearning4j.spark.time.SystemClockTimeSource
   ```

### 在Ubuntu16.04上训练失败（Ubuntu Bug可能会影响DL4J Spark用户）

在Ubuntu 16.04机器上运行YARN集群的Spark时，很可能在完成一个作业之后，运行Hadoop/YARN的用户拥有的所有进程都会被终止。这与Ubuntu中的一个bug有关，该bug记录在[https://bugs.launchpad.net/ubuntu/+source/procps/+bug/1610499](https://bugs.launchpad.net/ubuntu/+source/procps/+bug/1610499)上。 在 [https://stackoverflow.com/questions/38419078/logouts-while-running-hadoop-under-ubuntu-16-04](https://stackoverflow.com/questions/38419078/logouts-while-running-hadoop-under-ubuntu-16-04)。 上有一个Stackoverflow讨论。 

建议采取一些解决办法。

**选项1**

添加

```text
[login]
KillUserProcesses=no
```

到  /etc/systemd/logind.conf, 并重启。

**选项 2**

从Ubuntu 14.04复制/bin/kill二进制文件并使用它。

**选项3**

降级为Ubuntu 14.04

**选项 4**

在 `sudo loginctl enable-linger hadoop_user_name` 集群节点上运行


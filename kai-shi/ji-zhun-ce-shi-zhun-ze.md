---
description: DL4J和ND4J中基准通用准则。
---

# 基准测试准则

### 通用基准测试准则

**准则1: 在基准测试之前运行预热迭代**

预热期是你开始计时进行更多迭代之前，在没有计时的情况下运行多个（例如 几百个）迭代。

为什么有预热期？任意ND4J/DL4J 执行的前几次迭代可能比后面的迭代要慢，有以下原因：

1. 在初始化基准测试迭代中，JVM没有时间去进行代码的即时编译。一旦即时编译完成，对于所有后续操作，代码可能执行得更快。
2. 在ND4J 和 DL4J \(和其它库\)都有某种程度的懒初始化：第一次操作可能触发一些一次性执行的代码。
3. DL4J 或 ND4J \(当使用工作间\) 可以用一些迭代来学习执行的内存要求。在这个学习过程中，性能会比在它完成之前低。

**准则 2: 为所有基准测试运行多个迭代**

你的基准测试不是运行在你电脑上的惟一的东西（更不用说如果你使用的是云硬件，可能有共享资源）。并且操作运行时间不是完全确定的。

为了让基准测试可信，多次迭代是很重要的- 并且理想地报告运行时的均值和标准差。没有这一点，就不可能比较操作的性能，因为性能差异可能只是由于随机变化造成的。

**准则3: 特别注意你基准测试的是什么**

当比较框架时，这尤其重要。在声明“X操作的性能为Y”或“A比B快”之前，请确保：

你只对你感兴趣的操作进行基准测试。

如果您的目标是检查操作的性能，请确保只有此操作正被计时。

您应该仔细检查是否无意中包括其他内容——例如，是否包括：JVM初始化时间？库初始化时间？结果数组分配时间？垃圾收集时间？数据加载时间？

理想情况下，这些应在你报告的任何时间/性能结果中排除。如果他们不能被排除在外，请在制定性能要求时务必注意这点。

1. 你使用的是哪些本地库?  
  
   例如：BLAS的实现是什么（MKL, OpenBLAS, 等）？如果你正在使用CUD，你也使用了CuDNN吗？ ND4J 和 DL4J 可使用这些库\(MKL, CuDNN\)，当它们可用的时候。但通常默认是不可用的。如果它们没有被设为可用，性能会比较低，有时候相当低。

   在比较库之间的结果时，这一点尤其重要：例如，如果比较两个库（一个使用OpenBLAS，另一个使用MLK），那么结果可能只是反映了正在使用的BLAS库的性能差异，而不是别的正在测试的库的性能。类似地，一个CuDNN和另一个没有CuDNN的库可能简单地反映了使用CuDNN的性能收益。  

2. 如何配置?  
  
   不论好坏，DL4J和ND4J允许很多配置。对于大多数用户来说，这种配置的默认值是足够的，但有时需要手动配置才能获得最佳性能。这在某些基准测试中尤其适用。例如，这些配置选项中的一些允许用户以更高的内存使用以获得更好的性能。需要注意的一些配置选项：\(a\)内存配置\(b\)工作区和垃圾收集\(c\)CuDNN\(d\)DL4J缓存模式\(启用.`cacheMode`\(`CacheMode.DEVICE`\)\)

  
   如果你不确定你是否只是在运行DL4J或ND4J代码时测量您想要测量的内容，那么可以使用分析器，例如VisualVM或YourKit Profilers。  

3. 你正在使用的是哪个版本?  在基准测试时，您应该使用最新版本的任何基准测试库。识别和报告一个6个月前修复的瓶颈是没有意义的。当您在版本间比较性能时，这将是一个例外。还请注意，DL4J和ND4J的快照版本也是可用的——它们可能包含性能改进（可以随时询问）

**准则 4: 关注现实世界的用例——并运行一定范围的大小**

例如，考虑一个基准测试，两个数字相加:

```java
double x = 0;
//<start timing>
x += 1.0;
//<end timing>
```

在 ND4J 中等价于:

```java
INDArray x = Nd4j.create(1);
//<start timing>
x.addi(1.0);
//<end timing>
```

当然，上面的ND4J基准测试要慢得多—需要方法调用、执行输入验证、必须调用本地代码（具有上下文切换开销）等等。一个必须问的问题：这个问题是用户实际使用ND4J还是等价的线性代数库？这是一个极端的例子，但一般的观点是有效的。

还要注意，数学运算的性能可是特定的大小和形状。例如，如果你对矩阵乘法的性能进行基准测试，那么矩阵维数可能非常重要。在一些内部基准测试中，我们发现不同的BLAS实现（MKL 对比 OpenBLAS）和不同的后端（CPU 对比 GPU）可以在不同的矩阵维度下表现非常不同。对于所有输入形状和大小，我们在内部测试的BLAS实现（OpenBLAS、MKL、CUDA）没有一个比其他BLAS实现本质上更快。

因此，无论何时运行基准测试，都必须以多种不同的输入形状/大小运行这些基准测试，以获得完整的性能图。

**准则 5: 理解你的硬件**

当比较不同的硬件时，重要的是要知道它擅长什么。例如，你可能会发现神经网络小批量大小为1的时候，在CPU上训练比在GPU上执行得更快—但是更大的小批量大小正好相反。类似地，小的网络层尺寸可能不能充分利用GPU的计算能力。

此外，可能需要特别编译一些深度学习发行版来提供对诸如AVX2之类的硬件特性的支持（注意，ND4J的最新版本封装有支持这些特性的CPU的二进制文件）。当运行基准时，这些特征的利用（或缺乏）会对性能造成相当大的差异。

**准则 6: 让它可重现**

当运行基准测试时，重要的是使基准测试可重现。为什么？好或坏的表现可能只在某些有限的情况下发生。

最后，记住（a）ND4J和DL4J在不断发展，（b）基准测试有时的确会识别性能瓶颈（毕竟我们-ND4J实际上包括数百个不同的操作）。如果你确定了性能瓶颈，我们很想知道它，这样我们可以修复它。每当发现一个潜在的瓶颈时，我们首先需要重现它—这样我们就可以研究它、理解它并最终修复它。

**准则 7: 理解你的基准测试的局限**

线性代数库包含数百个不同的操作。神经网络库包含几十种层类型。当基准测试时，了解这些基准测试的局限性是很重要的。对一个类型的操作或层进行基准测试不能告诉你关于其他类型层或操作的任何性能，除非它们共享已被识别为性能瓶颈的代码。

**准则 8: 如果你不确定 - 咨询**

DL4J/ND4J开发者可在Gitter上使用。你可以在那里问一些关于基准测试和性能的问题: [https://gitter.im/deeplearning4j/deeplearning4j](https://gitter.im/deeplearning4j/deeplearning4j)

如果你碰巧发现一个性能问题-让我们知道！

### ND4J 特定的基准测试

**一个关于BLAS和数组顺序的标注**

BLAS或基本线性代数子程序-是指用于线性代数运算的接口和方法集。一些例子包括“gemm”-通用矩阵乘法和“axpy”，它实现了`Y = a*X+Y`.

ND4J可以使用多个BLAS实现，1.0.0-beta版本及以上已经默认为OpenBLAS。然而，如果安装了英特尔MKL\([这里](https://software.intel.com/en-us/mkl)提供免费版本\)，ND4J将与其链接，以提高许多BLAS操作的性能。

注意ND4J在初始化的时候会打印在后台使用的BLAS信息，例如:

```java
14:17:34,169 INFO  ~ Loaded [CpuBackend] backend
14:17:34,672 INFO  ~ Number of threads used for NativeOps: 8
14:17:34,823 INFO  ~ Number of threads used for BLAS: 8
14:17:34,831 INFO  ~ Backend used: [CPU]; OS: [Windows 10]
14:17:34,831 INFO  ~ Cores: [16]; Memory: [7.1GB];
14:17:34,831 INFO  ~ Blas vendor: [OPENBLAS]
```

性能取决于可用的BLAS库—在内部测试中，我们发现OpenBLAS比MKL快30%或慢8倍——这取决于阵列大小和数组顺序。

考虑到数组顺序，这也是影响性能的重要因素。ND4J有可能行优先\(‘c’\)也有可能列优先 \(‘f’\) 顺序呈现数组。查看[这个维基百科页面](https://en.wikipedia.org/wiki/Row-_and_column-major_order) 获取更多详情。矩阵乘法操作的性能和更通用的ND4J操作-取决于输入和结果顺序。

对于矩阵乘法，这意味着存在8个数组顺序的可能组合（对于输入1、输入2和结果阵列中的每个都为c/f）。对于所有的情况，性能都是不一样的。

类似地，对于某些输入顺序的组合，诸如逐元素相加\(即，z=x+y\)的操作将比其他操作快得多，尤其是当x、y和z都是相同顺序时。简而言之，这是由于内存跨步造成的：当这些内存地址在存储器中彼此相邻时，读取一系列内存地址要比分散到很远的地方成本更低。

注意，在缺省情况下，ND4J期望结果数组（用于矩阵乘法）按照列优先（“f”）顺序定义，以便在后端保持一致，因为CuBLAS（即，NVIDIA的用于CUDA的BLAS库）要求结果为f顺序。因此，与用“f”顺序数组执行相同操作相比，用结果数组按c顺序执行矩阵乘法的一些方法性能更低。

最后，当说到CUDA：数组顺序/跨步比在CPU上运行更重要。例如某些顺序组合可以比其他组合快得多-当输入输出的维度是32 或64的偶数倍的时候要比不是32的偶数倍的时候快（有时候快得多）。

### DL4J 特定的基准测试

对于ND4J所说的大部分也适用于DL4J。

另外:

1. 如果你正在使用nd4j-native \(CPU\) 后台，确保你正在使用 Intel MKL。在多数情况下这比默认的OpenBLAS要快。
2. 如果你正在使用CUDA, 确保你正在使用 CuDNN \([链接](https://deeplearning4j.org/docs/latest/deeplearning4j-config-cudnn)
3. 检查[工作间](https://deeplearning4j.org/docs/latest/deeplearning4j-config-workspaces)和[内存](https://deeplearning4j.org/docs/latest/deeplearning4j-config-memory)向导。默认值通常是很好的，但有时可以通过调整来获得更好的性能。如果在训练时内存中有很多Java对象（例如，Word2VEC向量），这一点尤其重要。
4. 注意ETL瓶颈。你可以在网络训练中添加PerformanceListener以查看ETL是否是瓶颈。
5. 别忘了性能取决于小批量的大小。不要使用微批量为1用来基准测试-使用更真实的数字。
6. 如果你需要多GPU 训练或推理 ，请使用ParallelWrapper 或ParallelInference。
7. 别忘了CuDNN 是可配置的: 你可以指定 DL4J/CuDNN 来获取更好的性能 -以内存消耗为代价 - 在卷积层上使用 `.cudnnAlgoMode(ConvolutionLayer.AlgoMode.PREFER_FASTEST)`配置
8. 当使用 GPU的时候, 对于输入大小或层的大小 8 \(或 32\) 的倍数可能性能更好。
9. 当使用 RNNs \(并且手动创建INDArrays\), 为特征和\(RnnOutputLayer\) 标签使用 ‘f’ 有序数组. 否则，使用“C”有序数组。这是为了更快的内存访问。

### 常见的基准测试错误

最后，这里列出了常见的基准测试错误：

1. 没有使用ND4J/DL4J的最新版本（没有发现瓶颈，因为在前几个版本就已经修复）。考虑尝试快照以获得最新的性能改进。
2. 没有注意正在使用什么本地库\(MKL, OpenBLAS, CuDNN 等\)
3. 在基准测试开始之前没有提供预热期
4. 只运行一个（或很少）的迭代，或没有报告均值，方差和迭代次数。
5. 没有配置工作间，垃圾回收等。
6. 只运行一种可能的情况-例如，在BLAS基准测试操作中对一单个数组维度/顺序集合进行基准测试。
7. 运行异常小的输入-例如，在GPU上微批次大小为1（这可能是慢的，但不真实！）
8. 不精确地测量，只测量你要测量的（例如，不考虑数组分配、初始化或垃圾收集时间）
9. 没有让你的基准测试可重现（基准结测试论可归纳吗？基准测试有问题吗？我们能做些什么来修复它？）
10. 跨硬件来比较结果，没有计算差异（例如，一个在带有AVX2的机器上测试，一个在没有AVX2的机器上测试）
11. 没有咨询开发者（通过[DL4J/ND4J](https://gitter.im/deeplearning4j/deeplearning4j) 极客频道）  我们很高兴提供建议和对性能不正常时进行调查。

## 如何运行Deeplearning4j 基准测试 - 指南

总训练时间通常是ETL时间加上计算时间 。也就是说，数据管道和矩阵操作一起决定了神经网络在数据集上训练的时间。

当一个熟悉Python的程序员试图将 Deeplearning4j基准测试与著名的Python框架的基准测试进行比较，他们通常在DL4J上用ETL+计算时间与Python框架的计算时间作为比较。那就是说，他们在用苹果与桔子作比较。下面我们将解释如何优化一些参数。

JVM有调整的旋钮，如果你知道如何调整它们，你可以使它成为一个非常快速的深度学习环境。在JVM上有几件事要牢记。你需要：

* 增加[堆空间](https://javarevisited.blogspot.com/2011/05/java-heap-space-memory-size-jvm.html)
* 让垃圾回收正确
* 让ETL异步进行
* 预保存数据集

### 堆空间设置

用户必须自己重新配置JVM，包括设置堆空间。我们不能把预先配置好的给你，但是我们可以告诉你怎么做。这里是堆空间的两个最重要的旋钮。

* Xms 设置最小的堆空间
* Xmx 设置最大的推空间

你可以将这些设置在IntelliJ和Eclipse等IDE中，也可以通过客户端这样设置：

```java
    java -Xms256m -Xmx1024m YourClassNameHere
```

在 [IntelliJ中，这是一个虚拟机参数](https://www.jetbrains.com/help/idea/2016.3/setting-configuration-options.html), 不是一个程序参数。当你在IntelliJ中点击（绿色按钮）运行时，会设置运行时配置。IJ以你指定的配置启动Java VM。

设置XMX的理想量是多少？这取决于计算机上有多少RAM。一般来说，按照JVM需要完成的任务分配尽可能多的堆空间。假设你在一个16G RAM的笔记本电脑上，向JVM分配8G的RAM。更小RAM的笔记本应设为3G，所以

```java
    java -Xmx3g
```

这似乎是违反直觉的，但你希望堆空间最小值和堆空间最大值是相同的，即`Xms`应该等于`Xmx`。如果它们不相等，JVM将根据需要逐步分配更多的内存，直到达到最大值，并且逐渐分配的过程会减慢速度。你希望在开始时预先分配它。所以

```java
    java -Xms3g -Xmx3g YourClassNameHere
```

IntelliJ 将自动指定所涉及的 [Java 主类](https://docs.oracle.com/javase/tutorial/getStarted/application/) 。

另一种方法是设置环境变量。在这里，你将更改隐藏的`.bash_profile`文件，它将环境变量添加到BASH中。若要查看这些变量，请在命令行中输入`env`。若要添加更多的堆空间，请在控制台中输入此命令：

```java
    echo "export MAVEN_OPTS="-Xmx512m -XX:MaxPermSize=512m"" > ~/.bash_profile
```

我们需要增加堆空间，因为Deeplearning4j在后台加载数据，这意味着我们在内存中占用更多的RAM。通过为JVM提供更多的堆空间，我们可以在内存中缓存更多的数据。

### 垃圾回收

垃圾回收器是在JVM上运行并清除Java应用程序不再使用的对象的程序。它是自动内存管理。在Java中创建一个新对象占用堆内存：默认情况下，一个新的Java对象占用8字节的内存。因此，创建的每个新的`DatasetIterator`都需要额外8个字节。

你可能需要更改Java正在使用的垃圾回收算法。这可以通过命令行完成，例如：

```java
    java -XX:+UseG1GC
```

更好的垃圾收集算法增加了吞吐量。关于这个问题的更详细的探索，请阅读这篇[InfoQ](https://www.infoq.com/articles/Make-G1-Default-Garbage-Collector-in-Java-9) 文章。

DL4J与垃圾回收器紧密相连。JavaCPP是JVM和C++之间的桥梁，它遵循你用Xmx设置的堆空间，并广泛使用堆外内存。堆外内存不会超过指定的堆空间。

JavaCPP，由Skymind工程师创建，依赖于垃圾回收器告诉它什么已经完成。我们依赖Java垃圾回收器告诉我们什么要被收集；Java垃圾回收器指向我们知道如何用JavaCPP去销毁它们的对象。这同样适用于我们如何使用GPU。

你使用的批次越大，内存占用的RAM越多。

### ETL & 异步ETL

在我们的`dl4j-examples中，我们没有使用异步的`ETL，因为我们的目标是让它们更简单。但对于真实问题，你需要使用异步的ETL，我们将在示例中展示如何使用它。

数据存储在磁盘上，磁盘速度慢，这是默认的。因此，当将数据加载到硬盘上时，会遇到瓶颈。当优化吞吐量时，最慢的组件始终是瓶颈。例如，使用三GPU的机器和一个CPU的机器分布式spark作业将有CPU瓶颈。GPU必须等待CPU完成。

Deeplearning4j 的 `DatasetIterator` 类隐藏了从磁盘加载数据的复杂性。用于任何Datasetiterator的代码通常是一样的，调用看上去是一样的，但它们以不同的方式工作。

* 一个从磁盘加载
* 一个异步加载
* 一个从RAM加载预保存的数据

下面是对于MISIST如何统一调用DatasetIterator的方法：

```java
        while(mnistTest.hasNext()){
                DataSet ds = mnistTest.next();
                INDArray output = model.output(ds.getFeatures(), false);
                eval.eval(ds.getLabels(), output);
        }
```

你可以通过在后台使用一个异步加载器来优化。Java可以实现真正的多线程。它可以在后台加载数据，而其他线程则负责计算。因此，你正在运行计算的同时，将数据加载到GPU中。甚至当你从内存中获取新数据时，神经网络也在进行训练。

这是 [相关代码](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-scaleout/deeplearning4j-scaleout-parallelwrapper/src/main/java/org/deeplearning4j/parallelism/ParallelWrapper.java#L136)，第三行比较特殊:

```java
    MultiDataSetIterator iterator;
    if (prefetchSize > 0 && source.asyncSupported()) {
        iterator = new AsyncMultiDataSetIterator(source, prefetchSize);
    } else iterator = source;
```

实际上有两种类型的异步数据集迭代器。“`AsyncDataSetIterator`”是你在大多数时间使用的。它在 [Javadoc](https://deeplearning4j.org/api/1.0.0-beta2/org/deeplearning4j/datasets/iterator/AsyncDataSetIterator.html) 中被描述。

对于特殊情况，如应用于时间序列的循环神经网络，或用于计算图，你将使用一个 `AsyncMultiDataSetIterator`, 在 [Javadoc ](https://deeplearning4j.org/api/1.0.0-beta2/org/deeplearning4j/datasets/iterator/AsyncMultiDataSetIterator.html) 中被描述。

在上面的代码中注意，`prefetchSize`是另一个要设置的参数。正常的批大小可能是1000个示例，但是如果将`prefetchSize`设为3，它将预获取3000个实例。

### ETL: Python框架与Deeplearning4j对比

在Python中，程序员将它们的数据转换为[泡菜](https://docs.python.org/2/library/pickle.html)或二进制数据对象。如果他们用一个小玩具数据集工作，他们将所有的泡菜装入RAM中。因此，它们有效地归避了处理较大数据集的主要任务。同时，当对DL4J进行基准测试时，它们不将所有数据加载到RAM上。所以他们实际上将Dl4j的训练计算+ETL的时间与Python框架的训练计算时间进行了比较。

但是Java有强大的工具来移动大数据，如果比较正确，比Python快得多。Deeplearning4j社区报告说，在优化ETL和计算时，与Python框架相比，Deeplearning4j速度提高了3700%。

Deeplearning4j使用DataVec作为ETL和向量化库。与其他深度学习工具不同，DataVec不强制数据集是一种特定格式。（例如，Caffe强制你使用[hdf5](https://support.hdfgroup.org/HDF5/)）。

我们试着变得更灵活。这意味着你可以将DL4J指向原始照片，它会加载图像、运行转换并将其放入NDArray中动态生成数据集。

但是如果但如果你的训练管道每次都这样做，Deeplearning4j看起来会比其他框架慢10倍，因为你把时间花费在了创建数据集上。每次你调用“`fit`”，你都会一遍又一遍地创建数据集。为了方便我们允许这样使用它，但是我们可以告诉你如何加快速度。有多种办法使它变快。

一种类似于Python框架的方式预先保存数据集的方法。（泡菜是预先格式化的数据）。当你预先保存数据集时，你将创建一个单独的类。

这里你是如何 [预保存数据集](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/presave/PreSave.java) 的例子

`Recordreaderdatasetiterator` 与 Datavec 交互并为DL4J输出数据。

这里是你如何 [加载预保存数据](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/presave/LoadPreSavedLenetMnistExample.java) 的例子。

第90行是你看到异步ETL的地方。在本例中，它包装了预先保存的迭代器，因此你可以利用这两种方法，通过异步将预先保存的数据加载到后台作为网络训练。

### CPU上的MKL和推理

如果您在CPU上运行推理基准测试，请确保你正在和Intel的MKL库一起使用Deeplearning4j，该库可以通过单击安装包获得；即，Deeplearning4j不像Anaconda那样捆绑MKL，Anaconda是PyTorch等库使用的。






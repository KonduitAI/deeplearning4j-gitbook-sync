---
description: 为DL4J应用程序设置可用内存/RAM
---

# 内存管理

### ND4J/DL4J内存管理: 它是如何工作的?

ND4J使用堆外内存来存储NDArray，以便在与来自本地代码（如BLAS和CUDA库）的NDAArray一起工作时提供更好的性能。

“堆外”意味着内存被分配到JVM（Java虚拟机）之外，因此不受JVM垃圾收集（GC）的管理。在Java/JVM方面，我们只保存指向堆外内存的指针，这些指针可以通过JNI传递给底层C++代码，用于ND4J操作。

为了管理内存分配，我们使用两种方法：

* JVM 垃圾回收 \(GC\) 和弱引用追踪
* 内存工作间 - 阅读 [内存工作间指引](https://deeplearning4j.org/workspaces) 获取详情

尽管这两种方法之间存在差异，但想法是相同的：一旦Java端不再需要NDArray，与其关联的堆外内存应该被释放，以便以后再使用。GC和“内存工作空间”方法之间的区别在于何时以及如何释放内存。

* 对于JVM/GC内存：每当垃圾收集器收集INDArray时，它的堆外内存将被释放，假设它没有被其他地方使用。
* 对于内存工作间：每当INDArray离开工作区范围时——例如，当一个层完成前向传递/预测时——其内存可以在不释放和重新分配的情况下被重用。这导致如神经网络训练和推理这样周期性的工作有更好的性能。

### 配置内存限制

对于DL4J/ND4J，有两种类型的内存限制需要注意和配置：堆上JVM内存限制和堆外内存限制（NDArray所在的位置）。这两个限制都是通过Java命令行参数来控制的：

* `-Xms` -这定义了JVM堆将在应用程序启动时使用多少内存。
* `-Xmx` - 这允许你指定JVM堆内存限制（最大值，在任何点）。如果需要的话，最多只分配给这个数量（在JVM的自由裁量权下）。
* `-Dorg.bytedeco.javacpp.maxbytes` - 这允许你指定堆外内存限制。
* `-Dorg.bytedeco.javacpp.maxphysicalbytes` - 这指定了整个进程的最大字节-通常设置为maxbytes+Xmx加上一些额外的字节，以防其它库还需要一些堆外内存。与设置`maxbytes`的设置不同，`maxphysicalbytes`是可选的。

示例: 配置 1GB 初始化堆内存, 2GB 最大堆内存, 8GB 堆外内存, 10GB 进程最大内存:

```java
-Xms1G -Xmx2G -Dorg.bytedeco.javacpp.maxbytes=8G -Dorg.bytedeco.javacpp.maxphysicalbytes=10G
```

### Gotchas: 几件值得注意的事情

* 对于GPU系统，maxbytes和maxphysicalbytes设置当前也有效地定义了GPU的内存限制，因为堆外内存被（通过NDArray）映射到GPU—请阅读下面的GPU部分中的更多信息。
* 对于许多应用程序，你希望JVM堆中使用的RAM更少，堆外使用的RAM更多，因为所有NDArray都存储在那里。如果对JVM堆分配太多，就不会有足够的内存留给堆外内存。
* 如果你得到一个“RuntimeException: Can’t allocate \[HOST\] memory: xxx; threadId: yyy”，你已经用完了堆外内存。你应该最常使用一个工作间配置来处理你的NDArrays分配，特别的在例如训练或评估/推断循环-如果你没有这么做，NDArray及其堆外（和GPU）资源使用JVM GC回收，这可能引入严重的延迟和可能的内存不足情况。
* 如果不指定JVM堆限制，默认情况下，它将使用系统总RAM的1/4作为限制。
* 如果不指定堆外内存限制，默认情况下将使用JVM堆限制（Xmx）的2倍。即-XMX8G将意味着8GB可以被JVM堆使用，而16GB可以被堆外的ND4J使用。
* 在有限的内存环境中，使用把-`Xmx值`和`-Xms值都设为很大`通常是一个坏主意。这是因为这样做不会留下足够的堆外内存。好比一个16GB系统，其中你设置-Xms14G：16GB的14GB将分配给JVM，仅留下2GB用于堆外内存、OS和所有其他程序，这样并不合理。

## 内存映射文件

当使用`nd4j-native`后端时，ND4J支持使用内存映射文件代替RAM。一方面，它比RAM慢，但另一方面，它允许你以不可能的方式分配内存块。

以下是示例代码：

```java
WorkspaceConfiguration mmap = WorkspaceConfiguration.builder()
                .initialSize(1000000000)
                .policyLocation(LocationPolicy.MMAP)
                .build();

try (MemoryWorkspace ws = Nd4j.getWorkspaceManager().getAndActivateWorkspace(mmap, "M2")) {
    INDArray x = Nd4j.create(10000);
}
```

在这种情况下，将创建一个1GB临时文件并会被映射，NDArray X将在该空间中被创建。显然，当你需要的NDArrays不能适合放入你的RAM时，这个选项是可行的。

### GPU

当使用GPU时，通常你的CPU使用的RAM往往会大于GPU使用的RAM。当GPU RAM小于CPU RAM时，需要监视堆外使用了多少RAM。你可以根据上面指定的JavaCPP选项来检查。

我们在GPU上分配你指定大小的堆外内存的。我们不会使用超过GPU的内存。也允许你指定大于GPU的堆空间（这是不鼓励的，但这是可能的）。如果这样做，当运行作业时，GPU将耗尽RAM。

我们也在CPU RAM上分配堆外内存。这是为了CPU与GPU的有效通信，以及CPU从NDArray访问数据，而无需每次调用时都从GPU获取数据。

如果JavaCPP或GPU抛出内存不足错误（OOM），或者即使由于GPU内存有限导致计算速度减慢，如果可能，你可能希望减小批处理大小或增加允许JavaCPP分配的堆外内存量。

尝试运行一个堆外内存等于你的GPU的RAM。同时，请记住使用`Xmx`选项设置一个小的JVM堆空间。

注意，如果你的GPU的RAM小于2G，它可能无法用于深度学习。如果情况是这样，你应该考虑使用你的CPU。典型的深度学习工作负载应该至少有4GB的RAM。尽管这样也是很小。GPU上的8GB RAM被推荐用于深度学习工作负载。

可以使用具有CUDA后端的HOST-only的内存。这可以使用工作区来完成。

示例:

```java
WorkspaceConfiguration basicConfig = WorkspaceConfiguration.builder()
    .policyAllocation(AllocationPolicy.STRICT)
    .policyLearning(LearningPolicy.FIRST_LOOP)
    .policyMirroring(MirroringPolicy.HOST_ONLY) // <--- 这个选项做这个小把戏
    .policySpill(SpillPolicy.EXTERNAL)
    .build();
```

不建议直接使用只使用HOST-only数组，因为它们会极大地降低性能。但是，它们与`INDArray.unsafeDuplication()` 方法一起作为内存缓存对，可能是很有用的 。


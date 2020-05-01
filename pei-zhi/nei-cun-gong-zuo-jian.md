---
description: 在DL4J中，工作间是一种有效的内存分页模型。
---

# 内存工作间

### 什么是工作间?

ND4J提供了一个额外的内存管理模型：工作间。这允许你在没有用于堆外内存跟踪的JVM垃圾回收器的情况下，重用循环工作负载的内存。换句话说，在工作间循环结束时，所有的数组内存内容都会失效。工作间被集成到DL4J中进行训练和推理。

基本思想很简单：你可以在工作间（或空间）内执行你需要的操作，并且如果你要从其去除一个INDArray（即，将结果移出工作空间），只需调用INDArray.detach\(\)，你将获得一个独立的INDArray副本。

### 神经网络

对于DL4J用户来说，工作间提供了更好的性能，并且默认情况下从1.0.0-alpha开始启用。因此，对于大多数用户，不需要明确的工作间配置。

为了从工作间中受益，它们需要被启用。你可以使用以下方式配置工作间模式：

`在你的神经网络中使用 .trainingWorkspaceMode(WorkspaceMode.SEPARATE)` 或 `.inferenceWorkspaceMode(WorkspaceMode.SINGLE)` 。

**SEPARATE**工作间和**SINGLE**工作间之间的差异是性能和内存占用之间的折衷：

* **SEPARATE** 稍微慢一点，但是使用更少的内存。
* **SINGLE** 稍微快一点，但使用更多内存。

也就是说，可以使用不同的模式进行训练和推理（即，使用SEPARATE进行训练，并使用SINGLE进行推理，由于推理仅涉及前馈循环，而不涉及反向传播或更新器）。

通过启用工作间，在训练期间使用的所有内存将被重用和跟踪，而不会受到JVM GC干扰。唯一例外的是在内部使用前馈循环的工作间（如果启用的话）的`output()`方法。随后，它将产生的`INDArray`从工作间中分离出来，从而为您提供独立的`INDArray`，该`INDArray`将由JVM GC处理。

请注意：在1.0.0-alpha发行版之后，DL4J中的工作间被重构——SEPARATE/SINGLE模式已经被废弃，并且用户应该使用ENABLED来代替。

### 垃圾回收器

如果你的训练过程使用工作间，建议你禁用（或减少周期性GC调用的频率）。可以这样做：

```java
// 这将限制GC调用的频率为5000毫秒。
Nd4j.getMemoryManager().setAutoGcWindow(5000)

// 或者你可以完全禁用它
Nd4j.getMemoryManager().togglePeriodicGc(false);
```

把它放到 `model.fit(...)` 调用之前。

### ParallelWrapper & ParallelInference

对于`ParallelWrapper`，还添加了工作间模式配置选项。这样，每个训练线程将使用一个单独的工作间，连接到指定的设备。

```java
ParallelWrapper wrapper = new ParallelWrapper.Builder(model)
      // DataSets 预获取选项。每个工作机器的缓存大小。
      .prefetchBuffer(8)

      //设置工作机器的数量为GPU的数量
      .workers(2)

      // 罕见的平均性能提高，但可能会降低模型准确率
      .averagingFrequency(5)

      // 如果设置为TRUE，则每个平均模型得分将被报告。
      .reportScoreAfterAveraging(false)

      // 3 选项在这里: NONE, SINGLE, SEPARATE
      .workspaceMode(WorkspaceMode.SINGLE)

      .build();
```

### 迭代器

我们提供异步预获取迭代器、`AsyncDataSetIterator`和`AsyncMultiDataSetIterator`，它们通常在内部使用。

这些迭代器可选地使用特殊的循环工作间模式来获得较小的内存占用。在这种情况下，工作空间的大小将由从底层迭代器出来的第一个`DataSet`的内存需求决定，而缓冲区大小由用户定义。如果内存需求随时间变化（例如，如果使用可变长度的时间序列），则工作间将被调整。

警告：如果你使用的是自定义迭代器或RecordReader，请确保在第一次next\(\)调用中没有初始化大型对象。在构造函数中这样做，以避免不希望的工作间增长。

警告：使用了`AsyncDataSetIterator`，`DataSets应`在调用`next()`数据集之前被使用。你不应该以任何方式存储它们，也不应不调用`detach()` 。否则，在DataSet内用于`INDArrays的内存最终将在AsyncDataSetIterator`内被覆盖。

如果由于某种原因，你不希望将迭代器包装到异步预获取器中（例如，用于调试目的），则提供特殊的包装器：`AsyncShieldDataSetIterator`和`AsyncShieldMultiDataSetIterator`。基本上，这些只是薄的包装，防止预获取。

### 评估

通常，评估假设使用model.output\(\)方法，该方法本质上返回与工作空间分离的`INDArray`。在训练过程中定期评估的情况下，最好使用内置的方法进行评估。例如：

```java
Evaluation eval = new Evaluation(outputNum);
ROC roceval = new ROC(outputNum);
model.doEvaluation(iteratorTest, eval, roceval);
```

这段代码将在`iteratorTest`上运行一个单独的周期，并且它将在不需要任何额外`INDArray`分配的情况下更新两个`IEvaluation`实现（或更少/更多，如果你要求）。

### 工作间破坏

也有一些情况，比如，你缺少RAM，可能希望释放你无法控制的所有工作空间；例如，在评估或训练期间。

`那么可以这样做：Nd4j.getWorkspaceManager().destroyAllWorkspacesForCurrentThread();`

此方法将销毁在调用线程中创建的所有工作空间。如果你自己在一些外部线程中创建了工作间，那么可以在不再需要工作间之后，在该线程中使用相同的方法。

### 工作间异常

如果工作区使用不正确（例如，在自定义层或数据管道中的缺陷），你可能会看到一个错误消息，如：

```java
org.nd4j.linalg.exception.ND4JIllegalStateException: Op [set] Y argument uses leaked workspace pointer from workspace [LOOP_EXTERNAL]
For more details, see the ND4J User Guide: nd4j.org/userguide#workspaces-panic
```

### DL4J的层工作间管理器

DL4J的层API包含一个“层工作区管理器”的概念。

这个类的思想是，它允许我们在给定工作间的不同的可能配置的情况下，轻松且精确地控制给定数组的位置。例如，层外的激活可以在推理期间放置在一个工作间中，而在训练期间放置在另一个工作间中；这是出于性能原因。然而，使用层工作间管理器设计，层的实现者不需要为此而烦恼。

这在实践中意味着什么？通常很简单…

* 当返回 \(`activate(boolean training, LayerWorkspaceMgr workspaceMgr)` 方法\)，确保返回的数组已在 `ArrayType.ACTIVATIONS` \(i.e., 使用 LayerWorkspaceMgr.create\(ArrayType.ACTIVATIONS, …\) 或类似\)中定义 
* 当返回激活梯度 \(`backpropGradient(INDArray epsilon, LayerWorkspaceMgr workspaceMgr)`\)，类似的返回一个在 `ArrayType.ACTIVATION_GRAD 中定义的数组。`

`你还可以在适合的工作间使用一个在任何工作间定义的数组，例如：LayerWorkspaceMgr.leverageTo(ArrayType.ACTIVATIONS, myArray)`

注意，如果你没有实现自定义层（而是只想对MultiLayerNetwork/ComputationGraph之外的层执行转发），那么可以使用`LayerWorkspaceMgr.noWorkspaces()`。


---
description: 如何在SameDiff图中添加微分函数和其他操作。
---

# 添加操作

## SameDiff快速概述

要开始使用SameDiff，请熟悉GitHub上ND4J API的autodiff模块。

不管好坏，SameDiff代码只组织在几个关键的地方。对于SameDiff的基本使用和测试，以下模块是关键。我们将更详细地讨论其中的一些。

* `functions`: 这个模块有基本的构建块来构建SameDiff变量和图。
* `execution`: 拥有与SameDiff图执行相关的所有内容。
* `gradcheck`: 用于检查SameDiff梯度的实用程序功能，其结构类似于DL4J中的相应工具。
* `loss`: SameDiff的损失函数
* `samediff`: 主要用于定义、设置和运行SameDiff操作和图形的SameDiff模块。

##  `functions` 模块中的微分函数

请参阅[GitHub](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/functions)上的`functions`模块。

`functions`模块的中心抽象是`DifferentialFunction`，它几乎是SameDiff中所有内容的基础。在数学上，我们在SameDiff中所做的是建立一个有向无环图，它的节点是微分函数，我们可以计算梯度。在这方面，`DifferentialFunction`构成了一个基本层次上的SameDiff图。

 注意，每个`DifferentialFunction`函数都有一个SameDiff实例。我们稍后再讨论`SameDiff`和这段关系。另外，虽然只有很少的关键抽象，但它们实际上在任何地方都被使用，所以几乎不可能单独讨论SameDiff概念。最后，我们将讨论每个部分。

### 属性与映射

每个微分函数都有属性。在最简单的情况下，微分函数只有一个名字。根据所讨论的操作，通常会有更多的属性（考虑卷积中的步幅或内核大小）。当我们从其他项目（TensorFlow、ONNX等）导入计算图时，这些属性需要映射到我们内部使用的约定。`attributeAdaptersForFunction`、`mappingsForFunction`、`propertiesForFunction`和`resolvePropertiesFromSameDiffBeforExecution`方法是您希望在开始时查看的内容。 

定义属性并正确映射后，分别为TensorFlow和ONNX import调用initFromTensorFlow和initFromOnnx。稍后，当我们讨论构建SameDiff操作时，将详细介绍这一点。

### 输入与输出

使用函数属性对输入列表执行微分函数，并生成一个或多个输出变量。您可以访问许多帮助函数来设置或访问这些变量：

* `args()`: 返回所有输入变量。
* `arg()`: 返回第一个输入变量（唯一一个用于一元操作）。
* `larg()` 与 `rarg()`: 返回二进制操作的第一个和第二个（读“left”和“right”）参数
* `outputVariables()`: 返回所有输出变量的列表。这取决于操作，可以动态计算。正如我们稍后将看到的，要获得具有单个输出的ops的结果，我们将调用.outputVariables\(\)\[0\]。

处理输出变量是很棘手的，也是使用和扩展SameDiff的一个陷阱。例如，可能需要为微分函数实现`calculateOutputShape`，但如果实现不正确，则可能导致难以调试的失败。（注意，SameDiff最终将调用`libnd4j`中的操作执行，动态自定义操作要么推断输出形状，要么需要提供正确的输出形状。）

### 自动微分

微分函数的自动微分是用一种方法实现的：doDiff。每个操作都必须提供doDiff的实现。如果您正在为`libnd4j` op x实现`SameDiff`操作，并且幸运地找到了`x_bp`（如“反向传播”），那么您可以使用它，并且`doDiff`实现基本上是自由的。

您还将看到内部使用的`diff`实现并调用`doDiff`。

### 微分函数工厂

重要的是，每个微分函数都可以通过调用`f()`访问factory（`DifferentialFunctionFactory`的实例）。更准确地说，这将返回微分函数具有的SameDiff实例的工厂：

```java
public DifferentialFunctionFactory f() {
    return sameDiff.f();
}
```

这在许多地方都有使用，并允许您访问当前在SameDiff中注册的所有微分函数。把这家工厂看作是一个操作提供者。下面是一个将`sum`暴露给`DifferentialFunctionFactory`的示例：

```java
public SDVariable sum(...) {
    return new Sum(...).outputVariables()[0];
}
```

我们故意省略了函数参数。注意，我们所做的只是重定向到ND4J中其他地方定义的Sum操作，然后返回第一个输出变量（`SDVariable`类型，将在第二个中讨论）。现在不考虑实现细节，这允许您从任何可以访问微分函数工厂的地方调用`f().sum(...)`。例如，当实现SameDiff op `x`并且函数工厂中已经有`x_bp`时，可以重写`x`的`doDiff`

```java
@Override
public List<SDVariable> doDiff(List<SDVariable> grad) {
    ...
    return Arrays.asList(f().x_bp(...));
}
```

## `SameDiff`中图的构建与执行

请参阅[GitHub](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff)上的`samediff`模块。

不足为奇，这就是魔法发生的地方。这个模块具有SameDiff操作的核心结构。首先，让我们看看构成SameDiff操作的变量。

### SameDiff 变量

`SDVariable`（读取SameDiff变量）是`DifferentialFunction`的扩展，它对SameDiff的作用就像`INDArray`对老ND4J的作用一样，特别是SameDiff图对这些变量进行操作，每个单独的操作都会接收并输出一个SDVariable列表。SDVariable带有一个名称，配备一个SameDiff实例，具有形状信息，并且知道如何使用ND4J `WeightInitScheme`初始化自身。您还可以找到一些助手来设置和获取这些属性。

`SDVariable`可以做的少数事情之一就是`DifferentialFunction`不能通过调用`eval()`评估其结果并返回底层的`INDArray`。这将在内部运行SameDiff并获取结果。类似的getter是getArr\(\)，您可以在任何时候调用它来获取此变量的当前值。此功能广泛用于测试，以断言正确的结果。`SDVariable`还可以通过`gradient()`访问其当前梯度。初始化时不会有任何梯度，通常在稍后的点计算。

除了这些方法之外，SDVariable还提供了具体操作的方法（在这方面与`DifferentialFunctionFactory`有点相似）。例如，定义`add`如下：

```java
public SDVariable add(double sameDiffVariable) {
    return add(sameDiff.generateNewVarName(new AddOp().opName(),0),sameDiffVariable);
}
```

允许对两个SameDiff变量调用`c=a.add(b)`，其结果可由`c.eval()`访问。

### SameDiff

`SameDiff`类是该模块的主要工作程序，它汇集了迄今为止讨论的大多数概念。有点不幸的是，反过来也是正确的，`SameDiff`实例在某种程度上是所有其他`SameDiff`模块抽象的一部分（这就是为什么您已经多次看到它）。一般来说，`SameDiff`是自动微分的主要入口点，您可以使用它来定义一个符号图，该图对`SDVariables`执行操作。一旦构建，`SameDiff`图可以通过几种方式运行，例如`exec()`和`execandResult()`。 

让自己相信调用`SameDiff()`会创建[很多很多东西](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff/SameDiff.java#L817-L846)！本质上，SameDiff将收集并允许您访问（就getter和setter而言）

* 图的所有微分函数及其所有属性，可以通过各种方式（如名称或id）访问。
* 所述功能的所有输入和输出信息。
* 所有函数属性以及如何映射它们、`propertiesToResolve`和`propertiesForFunction`都特别值得注意。

`SameDiff`也是向`SameDiff`模块公开新操作的地方。实际上，您可以为`DifferentialFunctionFactory`实例`f()`中的各个操作编写一个小包装器。下面是交叉积的一个例子：

```java
public SDVariable cross(SDVariable a, SDVariable b) {
    return cross(null, a, b);
}

public SDVariable cross(String name, SDVariable a, SDVariable b) {
    SDVariable ret = f().cross(a, b);
    return updateVariableNameAndReference(ret, name);
}
```

### SameDiff 执行示例和测试

在这一点上，查看并运行几个示例可能是一个好主意。`SameDiff`测试是一个很好的来源。下面是一个如何将两个`SameDiff`变量相乘的示例

```java
SameDiff sd = SameDiff.create();

INDArray inArr = Nd4j.linspace(1, n, n).reshape(inOrder, d0, d1, d2);
INDArray inMul2Exp = inArr.mul(2);

SDVariable in = sd.var("in", inArr);
SDVariable inMul2 = in.mul(2.0);

sd.exec();
```

这个例子取自[SameDiffTests](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-tests/src/test/java/org/nd4j/autodiff/samediff/SameDiffTests.java)，它是一个主要的测试源，在这里您还可以找到一些完整的端到端的例子。 

你发现测试的第二个地方是[samediff](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-tests/src/test/java/org/nd4j/autodiff/samediff) 。无论何时向SameDiff添加新操作，都要为前向传播和梯度检查添加测试。 

第三组相关测试存储在[imports](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-tests/src/test/java/org/nd4j/imports)中，包含用于导入TensorFlow和ONNX图的测试。另外，这些导入测试的资源是在我们的[TFOpsTests](%20https://github.com/kgnandu/TFOpTests)项目中生成的。

## 创建和公开新的SameDiff操作

我们已经看到了DifferentialFunctionFactory和SameDiff如何获取ND4J操作，以便在不同级别将它们暴露给SameDiff。至于实际实现这些操作，您需要知道一些事情。在libnd4j中，您可以找到两类操作，[这里](https://github.com/eclipse/deeplearning4j/blob/master/libnd4j/AddingNewOps.md)将详细介绍这两类操作。我们将演示如何实现这两种操作类型。

 所有的操作都是在[这里](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl)进行的，而且大多数情况下，很明显要把操作放在哪里。要特别注意层，这是为深度学习层实现（如[Conv2D](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/layers/convolution/Conv2D.java)）保留的。这些高级操作基于[模块](https://github.com/eclipse/deeplearning4j/blob/_old/deeplearning4j-1.0.0-beta4/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/BaseModule.java)的概念，类似于pytorch中的模块或TensorFlow中的层。这些层操作实现还提供了更多涉及操作实现的源。

### 实现遗留操作

遗留（或XYZ）操作是具有特征“XYZ”签名的老一代ND4J操作。下面是如何在ND4J中通过包装libn4j中的cos遗留操作来实现cosine:[cosine实现](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/strict/Cos.java)。说到SameDiff，遗留操作的好处是它们已经在ND4J中可用，但是需要通过SameDiff特定的功能来增强才能通过测试。因为余弦函数没有任何属性，所以这个实现很简单。使此操作SameDiff兼容的部分包括：

* 指定SameDiff构造函数 [这里](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/strict/Cos.java#L37-L39) 
* 你实现 `doDiff` [这里](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/Cos.java#L38-L51)
* 在[此处](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/Cos.java#L74-L93)指定`SameDiff` `opName`、TensorFlow `tensorflowName`和ONNX `onnxName`。

如果仔细观察，这只是事实的一部分，因为`Cos`扩展了实现其他SameDiff功能的`BaseTransformOp`。（注意，`BaseTransformOp`是一个[`BaseOp`](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/BaseOp.java)，它早期扩展了`DifferentialFunction`。）例如，`calculateOutputShape`就是在[这里](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/BaseTransformOp.java#L195-L207)实现的。如果您想实现一个新的转换，也可以简单地从`BaseTransformOp`继承。对于其他的操作类型，如reductions等，也可以使用操作基类，这意味着您只需要解决上面的三个要点。

 在极少数情况下，您需要从头开始编写一个遗留操作，您需要从libn4j中找到相应的操作编号，可以在`legacy-ops.h`中找到。

### 实现动态自定义操作

\`\`[`DynamicCustomOp`](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/DynamicCustomOp.java)是libnd4j中的一种新操作，所有最近添加的操作都是这样实现的。ND4J中的这种操作类型直接扩展了`DifferentialFunction`。 

[这里](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/custom/BatchToSpace.java)是从`DynamicCustomOp`继承的`BatchToSpace`操作的示例：

* BatchToSpace由两个属性（block和crops）[初始化](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/BatchToSpace.java#L49-L67)。请注意，blocks和crops都是整数类型的，它们是如何通过调用`addIArgument`添加到操作的整数参数中的。对于float参数和其他类型，请改用`addTArgument`。
* 操作获取自己的名称和[要导入的名称](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/BatchToSpace.java#L69-L82)，
* 和`doDiff` [被实现](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/BatchToSpace.java#L84-L89) 

BatchToSpace操作在[这里](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/functions/DifferentialFunctionFactory.java#L840-L844)集成到`DifferentialFunctionFactory`中，在[这里](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff/SameDiff.java#L2105-L2107)暴露在`SameDiff`中并在[这里](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-tests/src/test/java/org/nd4j/autodiff/gradcheck/GradCheckTransforms.java#L151-L191)测试。

 BatchToSpace当前唯一缺少的是属性映射。我们调用这个操作block和crops的属性，但是在ONNX或TensorFlow中，它们的调用和存储方式可能完全不同。要查找正确映射的差异，请参见Tensorflow的[ops.proto](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/resources/ops.proto)和ONNX的[onnxops.json](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/resources/onnxops.json)。 

让我们看看另一个正确执行属性映射的操作，即[`DynamicPartition`](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/custom/DynamicPartition.java)。这个操作只有一个属性，在SameDiff中称为`numPartitions`。要映射和使用此属性，请执行以下操作：

* 实现一个名为[`addArgs`](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/DynamicPartition.java#L59-L61)的小助手方法，该方法用于op的构造函数和导入助手中，下面一行我们将讨论。这是没有必要的，但鼓励这样做，并一致称之为`addArgs`，为了清晰。
* 重写[`initFromTensorFlow`](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/DynamicPartition.java#L63-L67)方法，该方法使用`TFGraphMapper`实例为我们映射属性，并使用`addArgs`添加参数。注意，由于ONNX在编写本文时不支持动态分区（因此没有`onnxName`），因此也没有`initFromOnnx`方法，其工作方式与`initFromTensorFlow`几乎相同。
* 为了使TensorFlow导入可以工作，我们还需要重写[mappingsForFunction](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/DynamicPartition.java#L70-L83)。这个映射示例非常简单，它所做的只是将TensorFlow的属性名称 `num_partitions`映射到我们的名称 `numPartitions`。

请注意，虽然`DynamicPartition`具有正确的属性映射，但它当前没有一个有效的`doDiff`实现。

作为最后一个例子，我们展示了一个更有趣的属性映射设置，即[Dilation2d](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/custom/Dilation2D.java)。正如您在[`mappingsForFunction`](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/Dilation2D.java#L59-L104)中看到的，这个操作不仅要映射更多的属性，而且属性还带有属性值，如[`attributeAdaptersForFunction`](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/impl/transforms/Dilation2D.java#L106-L132)中定义的。我们之所以选择显示此操作，是因为它具有属性映射，但既不向`DifferentialFunctionFactory`公开，也不向`SameDiff`公开。

因此，所示的三个DynamicCustomOp示例都有自己的缺陷，并代表了必须为SameDiff完成的工作的示例。总之，要添加新的SameDiff 操作，您需要：

* 在ND4J中创建扩展`DifferentialFunction`的新操作。具体如何设置此实现取决于
  * 操作生成 \(遗留与动态定制\)
  * 操作类型 \(变换、缩减等\)
* 定义自己的操作名，以及TensorFlow和ONNX名称。
* 定义必要的SameDiff构造函数
* 使用`addArgs`以可重用的方式添加操作参数。
* 首先在`DifferentialFunctionFactory`中公开操作，然后将其包装在`SameDiff`（或变量方法的`SDVariable`）中。
* 实现`doDiff`的自动微分。
* 重写`mappingsForFunction`以映射TensorFlow和ONNX的属性
* If necessary, also provide an attribute adapter by overriding `attributeAdaptersForFunction`.如果需要，还可以通过重写`attributeAdaptersForFunction`来提供属性适配器。
* 通过添加`initFromTensorFlow`和`initFromOnnx`（使用`addArgs`），为TensorFlow和ONNX添加import一行。
* 测试，测试，测试


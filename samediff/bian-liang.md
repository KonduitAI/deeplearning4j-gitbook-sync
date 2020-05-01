---
description: SameDiff中使用的变量类型、它们的属性以及如何切换这些类型。
---

# 变量

## 什么是变量

定义或传递每个`SameDiff`实例的所有值（无论是权重、偏差、输入、激活或常规参数）都由`SDVariable`类的对象处理。

观察到变量，我们通常指的不仅仅是单个值——就像在描述自动微分的各种在线示例中所做的那样——而是它们的整个多维数组。

## 变量类型

 `SameDiff`中的所有变量都属于四种变量类型之一，构成枚举`VariableType`。它们在这里：

* `VARIABLE`:  是网络的可训练参数，例如层的权重和偏差。当然，我们希望它们都被存储起来以供进一步使用——我们说，它们是持久化的——也在训练期间被更新。
* `CONSTANT`:  是那些参数，像变量一样，对于网络来说是持久化的，但是没有被训练；但是，它们可以由用户在外部更改。
* `PLACEHOLDER`:  存储要从外部提供的临时值，如输入和标签。 因此，由于新占位符的值是在每次迭代时提供的，所以它们不会被存储：换句话说，与`VARIABLE`和`CONSTANT`不同，`PLACEHOLDER`不是持久化的。
* `ARRAY`: 也是临时值，表示SameDiff中操作的输出，例如向量的和、层的激活等等。它们在每次迭代时都要重新计算，因此，与`PLACEHOLDER`一样，它们不是持久化的。

要推断特定变量的类型，可以使用`getVariableType`方法，如下所示：

```java
VariableType varType = yourVariable.getVariableType();
```

可以使用`getArr`或`getArr(true)`获取`INDArray`形式的变量的当前值，如果希望程序在变量值未初始化时抛出异常，则使用后者。

## 数据类型

每个变量中的数据也有其数据类型，包含在`DataType`中。目前，在`DataType`中有三种浮点类型：FLOAT、DOUBLE和HALF；四种整数类型：LONG、INT、SHORT和UBYTE；一种布尔类型`BOOL`—所有这些类型都称为数值类型。此外，还有一个字符串类型称为`UTF8`；还有两个helper数据类型`COMPRESSED`和`UNKNOWN`。16位浮点格式`BFLOAT16`和无符号整数类型（`UINT16`、`UINT32`和`UINT64`）将在1.0.0-beta5中提供。

要推断变量的数据类型，请使用

```java
DataType dataType = yourVariable.dataType();
```

您可能需要跟踪变量的数据类型，因为有时它确实很重要，您在操作中使用哪些类型。例如，一个卷积积，比如这个

```java
SDVariable prod = samediff.cnn.conv1d(input, weights, config);
```

将要求其`SDVariable`参数`input`和`weights`为浮点数据类型之一，否则将抛出异常。另外，正如我们下面将要讨论的，`VARIABLE`类型的所有`SDVariables`都应该是浮点类型的。

## 变量的共同特征

在我们讨论变量之间的差异之前，让我们先看看它们共享的属性

* 所有变量最终都来自`SameDiff`的一个实例，作为其图。实际上，每个变量都有一个`SameDiff`作为其字段之一。
* 所有操作的结果（输出）均为`ARRAY`类型。
* 操作中涉及的所有`SDVariable`都属于同一个`SameDiff`。
* 所有变量都可能有名字，也可能没有名字——在后一种情况下，名字实际上是自动创建的。无论哪种方式，名称都必须是唯一的。我们将回到下面的命名。

## 变量类型之间的差异

现在让我们更仔细地看看每种类型的变量，以及它们之间的区别。

### 变量

变量是网络的可训练参数。这就预先决定了他们在`SameDiff`中的本性。如前所述，变量的值既需要为应用程序保留，也需要在训练期间更新。训练意味着，我们迭代地用梯度的一小部分来更新值，这只有当变量是浮点类型时才有意义（见上面的数据类型）。

变量可以使用`SameDiff`实例中不同版本的var函数添加到`SameDiff`中。例如，代码

```java
SDVariable weights = samediff.var("weights", DataType.FLOAT, 784, 10);
```

将一个784x10个`float`数字数组的变量，一个单层MNIST感知器中的权重，这是一个预先存在的`SameDiff`实例`samediff`。

但是，通过这种方式，变量中的值将被设置为零。您也可以使用预设`INDArray`中的值创建变量。这样

```java
SDVariable weights = samediff.var("weigths", Nd4j.nrand(784, 10).div(28));
```

将创建一个充满正态分布随机生成数的变量，方差为`1/28`。当然，您可以使用任何其他数组创建方法，而不是`nrand`或任何预设数组。此外，您还可以使用一些流行的初始化方案，例如：

```java
SDVariable weights = samediff.var("weights", new XavierInitScheme('c', 784, 10), DataType.FLOAT, 784, 10);
```

现在，权重将使用Xavier方案随机初始化。有其他方法可以创建和填充变量：您可以在\[我们的javadoc\]的'known subclasses'部分中查找它们\([https://deeplearning4j.org/api/latest/org/nd4j/weightinit/WeightInitScheme.html](https://deeplearning4j.org/api/latest/org/nd4j/weightinit/WeightInitScheme.html)"\).

### 常数

常量保存存储的值，但与变量不同的是，在训练期间保持不变。例如，这些可能是您希望在网络中拥有的一些超参数，可以从外部访问。或者它们可能是您希望保持不变的神经网络的预训练权重（请参阅下面更[改变量类型](https://deeplearning4j.org/api/latest/)中的更多内容）。常数可以是任何数据类型。

* 例如，`int`和`boolean`可以与`float`和`double`一起使用。

通常，常量是通过`constant`方法添加到`SameDiff`中的。常数可以从`INDArray`创建，如下所示：

```java
SDVariable constant = samediff.constant("constants", Nd4j.create(new float[] {3.1415f, 42f}));
```

可以使用`scalar`方法之一创建由单个标量值组成的常量：

```java
INDArray someScalar = samediff.scalar("scalar", 42);
```

同样，我们引用[javadoc](https://deeplearning4j.org/api/latest/)作为整个参考。

### 占位符

`SameDiff`中最常见的占位符通常是输入和标签（如果适用）。可以创建任何数据类型的占位符，具体取决于在其中使用它们的操作。要向`SameDiff`添加占位符，可以调用占位符方法之一，例如：

```java
SDVariable in = samediff.placeHolder("input", DataType.FLOAT, -1, 784);
```

如MNIST中。在这里，我们指定你的占位符的名称，数据类型和形状-在这里，我们有28x28个灰度图片呈现为1d向量（因此784）来成批的长度，我们事先不知道（因此-1）。

### 数组

`ARRAY`类型的变量显示为`SameDiff`中操作的输出。因此，数组类型变量的数据类型取决于它所生成的操作类型和它的参数的变量类型。数组不是持久化的——它们是一次性的值，将在下一步从头开始重新计算。但是，与占位符不同，梯度是为它们计算的，因为这些是更新`VARIABLE`值所必需的。

创建数组类型变量的方法和创建操作的方法一样多，因此您最好关注我们的操作部分、[javadoc](https://deeplearning4j.org/api/latest/)和示例。

## 汇总表

让我们在一个表中总结变量类型的主要属性：

| 标题 | 可训练 | 梯度 | 持久化 | 工作间 | 数据类型 | 从何处实例化 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `VARIABLE` | Yes | Yes | Yes | Yes | Float only | Instance |
| `CONSTANT` | No | No | Yes | No | Any | Instance |
| `PLACEHOLDER` | No | No | No | No | Any | Instance |
| `ARRAY` | No | Yes | No | Yes | Any | Operations |

我们还没有讨论“工作间”的含义—如果您不知道，不用担心，这是一个内部技术术语，基本上描述了内存是如何在内部管理的。

## 修改变量类型

您也可以更改变量类型。目前，有三种选择：

### 变量到常量

有时-例如，如果执行迁移学习-您可能希望将变量转换为常量。这样做：

```java
samediff.convertToConstant(someVariable);
```

其中`someVariable`是`VARIABLE`类型的。变量`someVariable`将不再被训练。

### 常量到变量

相反，如果常量是浮点数据类型，则可以转换为变量。所以，举个例子，如果你希望你的冻结权重能够再次训练

```java
samediff.convertToVariable(frozenWeights); //不再冻结
```

### 占位符到常量

占位符也可以转换为常量-例如，如果需要冻结其中一个输入。对数据类型没有任何限制，但是，由于占位符值不是持久化的，在将它们转换为常量之前，应该设置它们的值。这可以按如下方式进行

```java
placeHolder.setArray(someArray);
samediff.convertToConstant(placeHolder);
```

现在不可能将常量转换回占位符，如果需要，我们可以考虑添加此功能。现在，如果您希望有效地冻结占位符，但能够再次使用它，请考虑为它提供常量值，而不是将其转换为常量。

## 变量的名称和值

### 从`SameDiff`获取变量

回想一下`SameDiff`实例中的每个变量都有其唯一的`String`名。`SameDiff`实际上是按变量名跟踪变量，并允许您使用`getVariable(String name)`方法检索它们。

 请考虑以下几行：

```java
SDVariable regressionCost = weights.mmul(input).sub("regression_prediction", bias).squaredDifference(labels);
```

这里，在函数`sub`中，我们实际上已经隐式地引入了一个变量（`ARRAY`类型），它保存了减法的结果。通过在操作的参数中添加一个名称，我们保证了从其他地方检索变量的可能性：例如，如果以后需要推断标签和作为向量的预测之间的差异，可以只写：

```java
SDVariable errorVector = samediff.getVariable("regressionPrediction").sub(labels);
```

如果您的整个`SameDiff`实例在其他地方初始化，并且您仍然需要掌握它的一些变量（例如，多个输出），那么这将变得特别方便。 

您可以获取和设置`SDVariable`的名称，方法分别是`getVarName`和`setVarName`。重命名时，请注意变量的名称在其`SameDiff`中保持唯一。

### 获取变量值

可以使用`eval()`方法以`INDArray`的形式检索任何变量的当前值。注意，对于非持久化变量，应该首先设置该值。对于具有梯度的变量，也可以使用`getGradient`方法推断梯度的值。


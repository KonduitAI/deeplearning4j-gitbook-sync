---
description: “SameDiff”中有哪些操作以及如何使用它们
---

# 操作

`SameDiff`中的操作基本上是按照您预期的方式进行的。在我们的框架中，变量是`SDVariable`类型的对象，对它们执行apply操作，从而生成新的变量。在对可用操作进行概述之前，让我们列出它们的一些公共属性。

## 操作的一般属性

* 任何变量类型的变量都可以在任何操作中使用，只要它们的数据类型与操作所需的数据类型匹配（同样，请参阅变量部分了解变量类型）。大多数情况下，操作将要求其`SDVariable`具有浮点数据类型。
* 由操作创建的变量具有`ARRAY`变量类型。
* 对于所有操作，您可以定义结果变量的字符串名称，尽管对于大多数操作，这不是必需的。名称作为每个操作中的第一个参数，如下所示：

  ```java
  SDVariable linear = weights.mmul("matrix_product", input).add(bias); 
  SDVariable output = sameDiff.nn.sigmoid("output", linear);
  ```

  可以使用`SameDiff`方法`getVariable(String name)`从外部访问命名变量。对于上面的代码，此方法将允许您推断`output`和mmul操作的结果。注意我们 甚至没有显式地将此结果定义为单独的`SDVariable`，但是相应的`SDVariable`将在内部创建并添加到SameDiff的实例中，字符串名为“matrix\_product”。事实上字符串名称是为操作生成的每个`SDVariable`指定的：如果不显式指定名称，则会根据操作的名称自动将其指定给生成的`SDVariable`。

## 操作概览

当前可用的操作的数量，包括重载总计数百，它们按复杂度排序，从通过卷积层产生输出的简单加法和乘法到创建专用的循环神经网络模块，以及更多。操作的数量之多会使得将它们全部列在一页上变得很麻烦。因此，如果您已经在寻找特定的东西，那么最好检查我们的[javadoc](https://deeplearning4j.org/api/latest/)，它已经包含了每个操作的详细信息，或者只需浏览自动完成提示（如果您的IDE支持的话）。在这里，我们试图给你一个主意，你可能会找到什么样的行动，并在哪里寻找他们。 

所有操作可以分为两个主要分支：`SDVariable`方法和`SameDiff`类方法。让我们仔细看看每一个：

### `SDVariable` 操作

我们已经在前面的例子中看到了`SDVariable`操作，比如

```java
SDVariable z = x.add(y);
```

其中`x`和`y`是`SDVariable`的。

在`SDVariable`方法中，您将发现：

* 执行线性代数的`BLAS`类型操作：例如`add`, `neg`, `mul` （用于缩放和元素相乘）和`mmul`（矩阵相乘）、`dot`、`rdiv`等。；
* 比较操作，如`gt`或`lte`，用于将每个元素与固定的双精度值进行比较，以及与相同形状的另一个`SDVariable`进行元素级比较；
*  基本缩减运算：如`min`、`sum`、`prod`（数组中元素的乘积）、`mean`、`norm2`、`argmax`（最大元素的索引）、`squaredDifference`，可以沿着指定的维度进行；
* 计算给定尺寸的平均值和标准差的基本统计操作：平均值和标准差。
* 重组底层数组的操作：`reshape`和`permute`，以及`shape`——一个将变量的形状作为整数数组传递的操作——维度大小；

`SDVariable` 操作很容易链接，生产线如下：

```java
SDVariable regressionCost = weights.mmul(input).add("regression_prediction", bias).squaredDifference(labels);
```

### `SameDiff` 操作

`SameDiff`方法的操作是通过每一个`SameDiff`中存在的6个辅助对象之一调用的，该操作将所有操作分割成6个不均匀分支：

* `math` - 一般数学运算；
* `random` - 创建不同的随机数生成器；
* `nn` - 通用神经网络工具；
* `cnn` - 卷积神经网络工具；
* `rnn` - 循环神经网络工具；
* `loss` - 损失函数;

  为了使用特定的操作，您需要从`SameDiff`实例调用这6个对象中的一个，然后调用操作本身，如下所示：

  ```java
  SDVariable y = sameDiff.math.sin(x);
  ```

  或 

  ```java
  SDVariable y = samediff.math().sin(x);
  ```

辅助对象之间的操作分布在结构上不具有更直观的方式来组织事物。所以，举个例子，如果你不确定是在`math`还是`nn`中寻找`tanh`运算，不用担心：我们都有。

让我们简要描述一下您希望在每个分支中找到的操作类型：

### `math` - 基本数学运算

数学模块主要由通用数学函数和统计方法组成。其中包括：

* 幂函数，例如 `square`, `cube`, `sqrt`, `pow`, `reciprocal` 等;
* 三角函数，例如。 `sin`, `atan` 等;
* 指数/双曲函数，如 `exp`, `sinh`, `log`, `atanh` 等;
* 各种各样的元素操作，比如取绝对值、舍入和剪裁，比如 `abs`, `sign`, `ceil`, `round`, `clipByValue`, `clipByNorm` 等; 
* 指定维度的缩减: `min`, `amax`, `mean`, `asum`, `logEntropy`, 以及类似的; 
* distance \(reduction\) operations, such as距离缩减操作，例如 `euclideanDistance`, `manhattanDistance`, `jaccardDistance`, `cosineDistance`, 

  `hammingDistance`, `cosineSimilarity`, 沿指定维度，对于两个形状相同的 `SDVariables`;

* 特定矩阵运算: `matrixInverse`, `matrixDeterminant`, `diag` \(创建对角矩阵\), `trace`, `eye` 

  \(变维恒等式矩阵的建立\), 和一些其它的;

* 更多统计操作: `standardize`, `moment`, `normalizeMoments`, `erf` 和 `erfc` \(高斯误差函数及其互补\);
* 计数和索引缩减：类似方法 `conuntZero` \(零元素数字\), `iamin` \(具有最小绝对值的元素的索引\), `firstIndex` （满足指定`Condition`函数的第一元素的索引）；
* 表示底层数组属性的缩减。这些包括例如。 `isNaN` \(按元素检查\), `isMax` 

  （沿指定维度保持形状）, `isNonDecreasing` \(沿指定维度缩减\);

* 元素逻辑操作： `and`, `or`, `xor`, `not`.

`math`中的大多数运算都有非常简单的结构，并且是这样推断的：

```java
SDVariable activation = sameDiff.math.cube(input);
```

操作可以是链式的，尽管与`SDVariable`操作相比，其方式更为繁琐，例如：

```java
SDVariable matrixNorm1 = sameDiff.math.max(sameDiff.math.sum(sameDiff.math.abs(matrix), 1));
```

观察到`sum`操作中的（整数）参数1告诉我们必须沿着1维，即矩阵的列取最大绝对值。

### `random` - 创建随机值

这些操作创建变量，其基础数组将按某些分布填充随机数

* 比如，伯努利，正态，二项式等等。这些值将在每次迭代时重置。例如，如果希望创建一个变量，将高斯噪声添加到MNIST数据集的条目中，可以执行以下操作：

  ```java
  double mean = 0.;
  double deviation = 0.05;
  long[] shape = new long[28, 28];
  SDVariable noise_mnist = sameDiff.random.normal("noise_mnist", mean, deviation, shape);
  ```

  随机变量的形状可能会有所不同。例如，假设你有不同长度的音频信号，你想给它们添加噪声。然后，您需要指定一个`SDVariable`，例如，`windowShape`和一个整数数据类型，然后继续这样做

  ```java
  SDVariabel noise_audio = sameDiff.random.normal("noise_audio", mean, deviation, windowShape);
  ```

### `nn` - 通用神经网络工具

这里我们存储的神经网络方法不一定与卷积的方法相关。其中包括

* 创建密连的线性层和ReLU层（带或不带偏置），并单独添加偏置：`linear`, `reluLayer`, `biasAdd`;
* 常用的激活函数，例如。 `relu`, `sigmoid`, `tanh`, `softmax` 以及使用较少的版本，如

  `leakyRelu`, `elu`, `hardTanh`, 还有更多；

* 使用`pad`方法填充的二维数组，支持多种填充类型，具有恒定和可变的填充宽度；
* 梯度爆炸/过拟合预防，如层`dropout`、`layerNorm`、`batchNorm`。批量归一化；

有些方法是为内部使用而创建的，但可以公开使用。其中包括：

* 一些流行的激活函数的导数-这些主要是为了加速反向传播；
* attention模块-基本上，我们将在下面讨论循环神经网络的构建模块。

虽然`nn`中的激活相当简单，但其他操作变得更加复杂。例如，要创建线性层或ReLU层，可能需要多达三个预定义的`SDVariable`对象，如下代码所示：

```java
SDVariable denseReluLayer = sameDiff.nn.reluLayer(input, weights, bias);
```

其中`input`、`weights`和`bias`需要具有相互匹配的维度。

要创建具有softmax激活的密连层，可以按以下步骤进行：

```java
SDVariable linear = sameDiff.nn.linear(input, weight, bias);
SDVariable output = sameDiff.nn.softmax(linear);
```

### `cnn` - 卷积神经网络工具

`cnn`模块包含通常用于卷积神经网络的层和操作-可以从`nn`模块中提取不同的激活。在`cnn`的操作中，我们目前已经创建了：

*  线性卷积层，目前用于尺寸小于等于3的张量（不包括小批量）：`conv1d`, `conv2d`, 

  `conv3d`, `depthWiseConv2d`, `separableConv2D`/`sconv2d`; 

* 线性反卷积层，目前 `deconv1d`, `deconv2d`, `deconv3d`; 
* 池化，例如 `maxPoooling2D`, `avgPooling1D`;
* 特殊变形方法： `batchToSpace`, `spaceToDepth`, `col2Im` 和类似的
* 上采样，目前由`upsampling2d`操作提出；
* 局部响应归一化：`localResponseNormalization`，目前仅用于二维卷积层；

卷积和反卷积操作由许多静态参数指定，如核大小、膨胀、有无偏置等。为了简化创建过程，我们将所需的参数打包成易于构造和更改的配置对象。所需的激活可以从`nn`模块借用。因此，例如，如果我们想要创建一个具有`relu`激活的3x3卷积层，我们可以如下进行：

```java
Conv2DConfig config2d = new Conv2DConfig().builder().kW(3).kH(3).pW(2).pH(2).build();
SDVariable convolution2dLinear = sameDiff.cnn.conv2d(input, weights, config2d);
SDVariable convolution2dOutput = sameDiff.nn.relu(convolution2dLinear);
```

在第一行中，我们使用它的默认构造函数构造卷积配置。然后我们指定内核大小（这是必需的）和可选的填充大小，并保持其他默认设置（单位步幅、无伸缩、无偏置 、NCHW数据格式）。然后，我们使用此配置创建一个线性卷积，其中预定义了用于输入和权重的`SDVariables`；`weights`的形状将调整为`input`和预先`config`的形状。因此，在上面的例子中，如果`input`有形状，比如说，`[-1，nIn，height，width]`，那么`weights`的形式就是`[nIn，nOut，3，3]`（因为我们有3x3卷积核）。结果变量`convoluton2d`的形状将由这些参数预先确定（在我们的情况下，它将是`[-1，nOut，height，width]`）。最后，在最后一行中，我们应用relu激活。

### `rnn` - 循环神经网络

这个模块可以说包含了框架中最复杂的方法。目前它允许您创建

* 简单循环单元，使用 `sru` 和 `sruCell` 方法;
* LSTM 单元，使用 `lstmCell`, `lstmBlockCell` 和  `lstmLayer`;
* Graves LSTM 单元，使用 `gru` 方法。

到目前为止，循环操作需要特殊的配置对象作为输入，在这些对象中，您需要打包将在一个单元中使用的所有变量。这可能会在以后的版本中发生更改。例如，要创建一个简单的循环单元，您需要这样进行：

```java
SRUConfiguration sruConfig = new SRUConfiguration(input, weights, bias, init);
SDVariable sruOutput = samediff.rnn().sru(sruConfig);
```

这里，`SRUConfiguration`构造函数中的参数是预先定义的变量。显然，它们的形状应该是匹配的，并且这些形状预先确定了`output`的形状。

### `loss` - 损失函数

在这个分支中，我们保留了共同的损失函数。大多数损失函数可以非常简单地创建，例如：

```java
SDVariable logLoss = sameDiff.loss.logLoss("logLoss", label, predictions);
```

其中`labels`和`predictions`是`SDVariable`的。在大多数`loss`方法中，字符串名称是必需的参数，但它可能被设置为空-在这种情况下，将自动生成名称。您还可以通过添加另一个包含权重的SDVariable参数来创建加权损失函数，并指定小批量损失的减少方法（见下文）。因此，一个全面的`logLoss`操作可能看起来像：

```java
SDVariable wLogLossMean = sameDiff.loss.logLoss("wLogLossMean", label, predictions, weights, LossReduce.MEAN_BY_WEIGHT);
```

一些损失操作可能允许/需要进一步的参数，这取决于它们的类型：例如，计算损失的维度（如`cosineLoss`），或一些实数参数。 

至于缩减方法，在小批中，目前有4种可用。因此，首先计算每个小批量样例的损失值，然后乘以权重（如果指定），最后执行下列程序之一：

* `NONE` - 保持结果（权重）损失值不变；结果是具有小批量长度的`INDArray`:`sum_loss=sum(weights*loss_per_sample)`。
* `SUM` - 对值求和，生成标量结果。
* `MEAN_BY_WEIGHT` -  首先计算上述总和，然后除以所有权重之和，生成标量值：`mean_loss = sum(weights * loss_per_sample) / sum(weights)`。  如果未指定权重，则将所有权重都设置为1.0，此缩减等于获得小批量的平均损失值。
* `MEAN_BY_NONZERO_WEIGHT_COUNT` - 将加权和除以非零权重的个数，生成标量：

  `mean_count_loss = sum(weights * loss_per_sample) / count(weights != 0)`。有用的，例如，当你只想计算有效样本子集的平均值时，将权重设置为0或1。当没有给出权重时，它只产生平均值，因此等价于`MEAN_BY_WEIGHT`。

## 操作注意事项

为了使`SameDiff`操作正常工作，需要遵守几个主要规则。如果不这样做，可能会导致异常，甚至会导致工作代码产生不希望的结果。我们在本节提到的所有事情都描述了什么是你最好不要做的。

* 一个操作中的所有变量都必须属于`SamdeDiff`的同一个实例（请参阅变量部分，了解如何将变量添加到`SameDiff`实例）。换句话说，你最好不要

  ```java
  SDVariable x = sameDiff0.var(DataType.FLOAT, 1);
  SDVariable y = sameDiff1.placeHolder(DataType.FLOAT, 1);
  SDVariable z = x.add(y);
  ```

* 充其量，为一个操作或一系列操作的结果创建一个新变量。换句话说，最好不要重新定义现有变量，最好不要让操作返回结果。换言之，尽量避免这样的代码：

  ```java
  SDVariable z = x.add(y);
  //DON'T!!!
  z.mul(2);
  x = z.mul(y);
  ```

  上述代码的正确工作版本（如果我们希望以一种不寻常的方式获得2xy+2y2）将是

  ```java
  SDVariable z = x.add(y);
  SDVariable _2z = z.mul(2);
  w = _2z.mul(y);
  ```

  要了解它为什么会这样工作，请参阅我们的图章节。


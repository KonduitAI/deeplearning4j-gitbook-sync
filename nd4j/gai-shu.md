---
description: ND4J综合编程指南。
---

# 概述

## 介绍

NDArray本质上是n维数组：即矩形数字数组，具有一定维数。

一些你应该熟悉的概念:

* NDArray的_rank_是维度数。二维数组的_rank_为2，三维数组的_rank_为3，依此类推。你可以创建具有任意_rank_的NDArrays。
* NDArray`的(shape)形状定义了每个维度的大小。假设我们有一个有3行5列的二维数组。这个`NDArray`的形状是[3,5]`
* NDArray的长度定义了数组中元素的总数。长度始终等于构成形状的值的乘积。
* NDArray的步幅定义为每个维度中相邻元素的间隔（在底层数据缓冲区中）。步幅是按维度定义的，因此一个rank 为 n的 NDArray有n个步幅值，每个维度一个。请注意，大多数情况下，你不需要了解（或关注）步幅-只需注意这是ND4J内部的运作方式。下一节有一个步幅的例子。
* NDArray的数据类型指的是一个NDArray的数据类型（例如， float 或 double 精度）。注意在nd4j中是全局的设置，所以所有的NDArrays应该有相同的数据类型。设置数据类型会在这个文档的后面再讨论。

就索引而言这里有一些事情需要知道。首先，维度0是行，维度1是列:因此`INDArray.size(0)是行的数量，INDArray.size(1)是列的数量，索引是0开始的：因此行有从0到INDArray.size(0)-1的索引，对于其他维度，依此类推。`

在本文档中，我们将使用`NDArray`术语来描述n维数组的一般概念；术语`NDArray`具体指的是ND4J定义的Java接口。实际上，这两个术语可以互换使用。

#### NDArrays:它们在内存中是如何存储的？

接下来的几段描述了ND4J背后的一些架构。理解这一点并不是使用ND4J所必需的，但它可能有助于你了解幕后发生的事情。NDArrays作为一个单一的扁平的数字数组（或者更一般地说，作为单个连续的内存块）存储在内存中，因此与典型的Java多维数组（例如，浮点\[\]\]\[2\] \[\]\[\]\[\]\[\]有很大的不同。

物理上，INDArray背后的数据是堆外存储的：也就是说，它存储在Java虚拟机（JVM）之外。这具有许多优点，包括性能、与高性能BLAS库的互操作性以及避免JVM在高性能计算中的一些缺点（例如，由于整数索引，Java数组限于2 ^ 31 - 1（21亿4000万）个元素）。

在编码方面，可以按C（行主要）或Fortran（列主要）顺序对NDArray进行编码。有关行与列主顺序的更多详细信息，请参阅维基百科。ND4J可以同时使用C和F顺序数组的组合。大多数用户只能使用默认的数组排序，但请注意，如果需要，可以对给定的数组使用特定的排序。

下图显示了简单的3x3（2d）NDArray是如何存储在内存中的，

![C vs. F order](http://upload-images.jianshu.io/upload_images/14495907-e6c9384ba1b25652.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在如下的数组，我们有:

* `Shape = [3,3]` \(3 行 , 3 列\)
* `Rank = 2` \(2 维\)
* `Length = 9` \(3x3=9\)
* Stride
  * C顺序步幅:`[3,1]: 在缓冲区中连续行中的值按3分割`，`在缓冲区中连续列中的值按1分割`
  * F顺序步幅:`[1,3]: 在缓冲区中连续行中的值按1分割`，`在缓冲区中连续列中的值按3分割`

#### 视图：当两个或更多NDArrays引用相同的数据

NDArrays中的一个关键概念是两个NDArrays实际上可以指向内存中相同的底层数据。通常，我们有一个数组引用另一个数组的某个子集，而这只发生在某些操作（如`INDArray.get()`, `INDArray.transpose()`, `INDArray.getRow()`等）中。这是一个强大的概念，值得理解。

这主要有两个动机：

1. 有相当大的性能优势，尤其是在避免复制数组方面
2. 我们在NDArrays上如何执行操作方面获得了很大的权力。

考虑一个简单的操作，比如一个大型（10000x1000）矩阵上的矩阵转置。使用视图，我们可以在不执行任何复制的情况下（即，大O标记法中的O（1））在恒定时间内执行矩阵转置，从而避免复制所有数组元素的相当大的成本。当然，有时我们确实想制作一份副本-这时我们可以使用`INDArray.dup()`来获取副本。例如，要获得转置矩阵的副本，请使用`INDArray out = myMatrix.transpose().dup()`。在这个dup（）调用之后，原始数组myMatrix和数组out之间将没有链接（因此，对其中一个的更改不会影响另一个）。

所以看看视图是如何强大的，考虑一个简单的任务：将1.0添加到更大数组的第一行，myarray。我们可以很容易地做到这一点，在一行中：

`myArray.getRow(0).addi(1.0)`

让我们来分析一下这里发生的事情。首先，`getRow(0)`操作返回一个INDArray，是原始数据的视图。注意myarray和 `myArray.getRow(0)`都指向内存中的相同区域：

![getRow\(0\)](http://upload-images.jianshu.io/upload_images/14495907-d2f48788521a8143.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后，在addi\(1.0\)被执行后，我们有如下状态

![getRow\(0\).addi\(1.0\)](http://upload-images.jianshu.io/upload_images/14495907-fc8c09f6da1ca344.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

​如我们所见，对`myArray.getRow(0)`返回的NDArray的更改将反映在原始数组`myArray`中；同样，对`myArray`的更改将反映在行向量中。

## 创建NDArrays

### 0,1和标量值初始化数组

创建数组最常用的两种方法是：

* `Nd4j.zeros(int...)`
* `Nd4j.ones(int...)`

数组的形状被指定为整数。例如，要创建一个由3行5列组成的零填充数组，请使用 `Nd4j.zeros(3,5)`。

这些通常可以与其他操作结合使用其他值创建数组。例如，要创建一个填充了10的数组：

`INDArray tens = Nd4j.zeros(3,5).addi(10)`

上面的初始化工作分为两个步骤：首先分配一个3x5数组，用零填充，然后向每个值加10。

### 随机数组

Nd4j`提供了一些创建`INDArrays`的方法，其中内容是伪随机数。`

`要在0到1范围内生成统一的随机数，请使用Nd4j.rand(int nRows, int nCols)（对于二维数组）或Nd4j.rand(int[])` `（对于3个或更多维度）。`

`同样，要生成平均零和标准偏差为1的高斯随机数，请使用` `Nd4j.randn(int nRows, int nCols)` or `Nd4j.randn(int[])。`

`对于可重复性（即设置nd4j的随机数生成器种子），可以使用Nd4j.getRandom().setSeed(long)`

### 从Java数组创建NDArrays

ND4J为从Java float和double组创建数组提供了方便的方法。

要从1D Java数组创建1D NDArray，请使用：

* 行向量: `Nd4j.create(float[])` 或  `Nd4j.create(double[])`
* 列向量: `Nd4j.create(float[],new int[]{length,1})` 或  `Nd4j.create(double[],new int[]{length,1})`

对于 2维数组, 使用 `Nd4j.create(float[][])` 或 `Nd4j.create(double[][])`.

为了从具有3个或多个维度的Java原始数组（double \[\]\[\]\[\]等）创建NDArrays，一种方法是使用以下：

```java
double[] flat = ArrayUtil.flattenDoubleArray(myDoubleArray);
int[] shape = ...;    //这是数组形状
INDArray myArr = Nd4j.create(flat,shape,'c');
```

### 从其它NDArrays创建NDArrays

从其它数组创建数组的方法主要有三种：

* `使用INDArray.dup()创建现有`NDArray`的精确副本`
* 将数组创建为现有NDArray的子集
* 合并多个现有NDArrays以创建新的NDArrays

对于第二种情况，您可以使用getRow\(\), get\(\)等。有关详细信息，请参见获取和设置NDArrays的部分。

合并NDArrays的两种方法是`Nd4j.hstack(INDArray...)` 和 `Nd4j.vstack(INDArray...)`。

hstack（水平堆叠）将具有相同行数的多个矩阵作为参数，并将它们水平堆叠以生成新的数组。但是，输入NDArrays可以有不同数量的列。

示例:

```java
int nRows = 2;
int nColumns = 2;
//创建值为0的INDArray
INDArray zeros = Nd4j.zeros(nRows, nColumns);
//创建所有值为1的INDArray 
INDArray ones = Nd4j.ones(nRows, nColumns);
//水平堆叠
INDArray hstack = Nd4j.hstack(ones,zeros);
System.out.println("### HSTACK ####");
System.out.println(hstack);
```

输出:

```java
### HSTACK ####
[[1.00, 1.00, 0.00, 0.00],
[1.00, 1.00, 0.00, 0.00]]
```

`vstack` \(垂直堆叠\) 是hstack的垂直等效值。输入数组的列数必须相同。

示例:

```java
int nRows = 2;
int nColumns = 2;
// 创建值为0的INDArray
INDArray zeros = Nd4j.zeros(nRows, nColumns);
// 创建所有值为1的INDArray
INDArray ones = Nd4j.ones(nRows, nColumns);
//垂直堆叠
INDArray vstack = Nd4j.vstack(ones,zeros);
System.out.println("### VSTACK ####");
System.out.println(vstack);
```

输出:

```java
### VSTACK ####
[[1.00, 1.00],
 [1.00, 1.00],
 [0.00, 0.00],
 [0.00, 0.00]]
```

`ND4J.concat` 沿维度组合数组。

示例:

```java
int nRows = 2;
int nColumns = 2;
//创建一个所有值为0的INDArray
INDArray zeros = Nd4j.zeros(nRows, nColumns);
//创建一个所有值为1的INDArray
INDArray ones = Nd4j.ones(nRows, nColumns);
// 在0维上连接
INDArray combined = Nd4j.concat(0,zeros,ones);
System.out.println("### COMBINED dimension 0####");
System.out.println(combined);
//在1维上连接
INDArray combined2 = Nd4j.concat(1,zeros,ones);
System.out.println("### COMBINED dimension 1 ####");
System.out.println(combined2);
```

输出:

```java
### COMBINED dimension 0####
[[0.00, 0.00],
 [0.00, 0.00],
 [1.00, 1.00],
 [1.00, 1.00]]
### COMBINED dimension 1 ####
[[0.00, 0.00, 1.00, 1.00],
 [0.00, 0.00, 1.00, 1.00]]
```

`ND4J.pad` 用于填充数组

示例:

```java
int nRows = 2;
int nColumns = 2;
// 创建一个所有值为1的INDArray
INDArray ones = Nd4j.ones(nRows, nColumns);
// 填充INDArray
INDArray padded = Nd4j.pad(ones, new int[]{1,1}, Nd4j.PadMode.CONSTANT );
System.out.println("### Padded ####");
System.out.println(padded);
```

输出:

```java
### Padded ####
[[0.00, 0.00, 0.00, 0.00],
 [0.00, 1.00, 1.00, 0.00],
 [0.00, 1.00, 1.00, 0.00],
 [0.00, 0.00, 0.00, 0.00]]
```

另一种偶尔有用的方法是`Nd4j.diag(INDArray in)`。此方法有两种用途，具体取决于中的参数：

* 如果in是向量，diag输出一个对角线等于数组in（其中N是in的长度）的NxN矩阵。
* `如果in是一个`NxN`矩阵，diag将输出一个从in的对角线获取的向量。`

#### 混杂的NDArray创建方法

要创建大小为N的[单位矩阵](https://en.wikipedia.org/wiki/Identity_matrix)（对角线上的值都为1，其它值都为零），可以使用`Nd4j.eye(N)`。

要使用元素`[a, a+1, a+2, ..., b]`创建行向量，可以使用linspace命令：

`Nd4j.linspace(a, b, b-a+1)`

Linspace可以与reshape操作结合使用以获得其他形状。例如，如果要一个包含5行和5列、值为1到25（包括1到25）的二维NDArray，可以使用以下内容：

`Nd4j.linspace(1,25,25).reshape(5,5)`

### 获取和设置单个值

对于INDArray，可以使用要获取或设置的元素的索引来获取或设置值。对于维度为n数组（即具有n个维度的数组），需要n个索引。

注意：单独获取或设置值（例如，在for循环中一次一个值）在性能方面通常是一个坏主意。如果可能，尝试使用其他一次对大量元素进行操作的INDArray方法。

要从二维数组中获取值，可以使用：`INDArray.getDouble(int row, int column)`

对于任何维度的数组，可以使用`INDArray.getDouble(int...)`。例如，要获取索引i、j、k处的值，请使用 `INDArray.getDouble(i,j,k)`

要设置值，请使用putScalar方法之一：

* `INDArray.putScalar(int[],double)`
* `INDArray.putScalar(int[],float)`
* `INDArray.putScalar(int[],int)`

这里，int\[\]是索引，double/float/int是要放置在该索引上的值。

在某些情况下可能有用的一些附加功能是`NDIndexIterator`类。`NDIndexIterator`允许你按定义的顺序获取索引（具体来说，对于rank为3的数组，C顺序遍历顺序为\[0,0,0\]、\[0,0,1\]、\[0,0,2\]、…、\[0,1,0\]、…等）。

要迭代二维数组中的值，可以使用：

```text
NdIndexIterator iter = new NdIndexIterator(nRows, nCols);
while (iter.hasNext()) {
    int[] nextIndex = iter.next();
    double nextVal = myArray.getDouble(nextIndex);
    //用这个值做些事情
}
```

## 获取和设置NDArrays的部分值

### getRow\(\) 与 putRow\(\)

要从INDArray中获取单行，可以使用`INDArray.getRow(int)`。这显然会返回一个行向量。这里需要注意的是，这一行是一个视图：对返回行的更改将影响原始数组。这有时非常有用（例如：`myArr.getRow(3).addi(1.0)` 将1.0添加到较大数组的第三行）；如果需要行的副本，请使用`getRow(int).dup()`。

同样，要获取多行，请使用`INDArray.getRows(int...)`。这将返回一个行已堆叠的数组；但是请注意，这将是原始行的副本（而不是视图），由于NDArrays存储在内存中的方式，因此此处不可能有视图。

对于设置单行，可以使用 `myArray.putRow(int rowIdx,INDArray row)`。这会将myarray的rowidxth行设置为包含在indarray行中的值。

### 子数组 : get\(\), put\(\) 与 NDArrayIndex

**Get:**

更强大和通用的方法是使用`INDArray.get(NDArrayIndex...)`。此功能允许你基于某些索引获取任意子数组。这也许最好用一些例子来解释：

要获取单行（和所有列），可以使用： `myArray.get(NDArrayIndex.point(rowIdx), NDArrayIndex.all())`

要获取行（行A（含）到行（不含）和所有列的范围，可以使用： `myArray.get(NDArrayIndex.interval(a,b), NDArrayIndex.all())`

要获取所有行和第二列，可以使用： `myArray.get(NDArrayIndex.all(),NDArrayIndex.interval(0,2,nCols))`

尽管上面的示例仅适用于二维数组，但NDArrayIndex方法扩展到3个或更多维度。对于三维，你将提供3个NDArrayIndex对象，而不是两个，如上所述。 请注意，`NDArrayIndex.interval(...)`, `.all()` 与 `.point(int)`方法始终返回底层数组的视图。因此，.get（）返回的数组更改将反映在原始数组中。

**Put:**

同样的NDArrayIndex方法也用于将元素放入另一个数组：在本例中，你使用`INDArray.put(INDArrayIndex[], INDArray toPut)`方法。显然，NDArray toput的大小必须与提供的索引所暗示的大小匹配。

还要注意`myArray.put(NDArrayIndex[],INDArray other)`在功能上等同于执行`myArray.get(INDArrayIndex...).assign(INDArray other)`。同样，这是因为`.get(INDArrayIndex...)`返回底层数组的视图，而不是副本。

### 沿维张量

（注：与当前版本相比，0.4-rc3.8及更早版本的ND4J返回的沿维度张量结果略有不同）。

沿维张量是一种强大的技术，但一开始可能有点难以理解。沿维张量（以下简称为TAD）背后的思想是得到一个低阶子数组，它是原始数组的视图。

“沿维张量”方法采用两个参数：

* 要返回的张量的索引（在0到numTensors-1的范围内）
* 执行TAD操作的维度（1个或多个值）

The simplest case is a tensor along a single row or column of a 2d array. Consider the following diagram \(where dimension 0 \(rows\) are indexed going down the page, and dimension 1 \(columns\) are indexed going across the page\):

最简单的情况是沿二维数组的单个行或列的张量。考虑下面的关系图（其中维度0（行）在页面下方被索引，维度1（列）在页面上方被索引）：

![Tensor Along Dimension](http://upload-images.jianshu.io/upload_images/14495907-24cac6158227907b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ​注意，在所有情况下，一个维的tensorAlongDimension调用的输出都是行向量。

为了理解我们为什么得到这个输出：考虑上图中的第一个案例。在那里，我们取第0（第一）个张量沿维数0（维数0为行）；当我们沿维数0移动时，值（1,5,2）在一条线上，因此输出。同样地，`tensorAlongDimension(1,1)`是第二个（index=1）沿维数1的张量；当我们沿维数1移动时，值（5,3,5）在一条线上。

TAD操作也可以沿多个维度执行。例如，通过指定两个维度来执行TAD操作，我们可以使用它从3D（或4D或5D…）数组中获取二维子数组。同样，通过指定3个维度，我们可以使用它从4d或更高维度获得3d。

有两件事我们需要知道的输出，为TAD操作是有用的。

首先，我们需要我们能得到的张量的数量，对于一组给定的维度。为了确定这一点，我们可以使用“沿维度的张量数量”方法，即`INDArray.tensorssAlongDimension(int... dimensions)`。此方法只返回沿指定维度的张量数。在上面的例子中，我们有：

* `myArray.tensorssAlongDimension(0) = 3`
* `myArray.tensorssAlongDimension(1) = 3`
* `myArray.tensorssAlongDimension(0,1) = 1`
* `myArray.tensorssAlongDimension(1,0) = 1`

（在后两种情况下，请注意，沿维度的张量将提供与原始数组相同的数组输出，即，我们从二维数组获得二维输出）。

一般来说，张量的个数是由剩余尺寸的乘积给出的，张量的形是由原形状中指定维度的大小给出的。

以下是一些例子：

* 对于输入形状\[a,b,c\]，tensorssAlongDimension\(0\)给出b\*c张量，tensorAlongDimension\(i,0\)返回形状 \[1,a\]的张量。
* 对于输入形状\[a,b,c\]，tensorssAlongDimension\(1\)给出一个a\*c张量，tensorAlongDimension\(i,1\)返回形状\[1,b\]的张量。
* 对于输入形状\[a,b,c\]，tensorsalongdimension（0,1）给出c张量，tensorAlongDimension\(i,0,1\)返回形状\[a,b\]的张量。
* 对于输入形状\[a,b,c\]，tensorssAlongDimension\(1,2\)给出张量，tensorAlongDimension\(i,1,2\)返回形状\[b,c\]的张量。
* 对于输入形状\[a,b,c,d\]， tensorssAlongDimension\(1,2\) 给出了一个a\*d张量，tensorAlongDimension\(i,1,2\)返回了形状\[b,c\]的张量。
* 对于输入形状\[a,b,c,d\]，tensorssAlongDimension\(0,2,3\)给出b张量，tensorAlongDimension\(i,0,2,3\)返回形状为\[a，c，d\]的张量。

#### 切分

\[本节：即将到来。\]

## 在NDArrays上进行操作

ND4J有操作的概念，你可能想用一个INDArray来做（或去做）很多事情。例如，ops用于应用tanh操作、添加标量或执行元素操作。

ND4J定义了五种类型的操作：

* Scalar（标量）
* Transform（转换）
* Accumulation（累加）
* Index Accumulation（索引累加）
* Broadcast（广播）

以及两种执行方法：

* 直接在整个INDArray上，或
* 沿一个维度

在讨论这些操作的细节之前，让我们先考虑一下就地和复制操作之间的区别。

许多操作既有就地操作，也有复制操作。假设我们要添加两个数组。Nd4j为此定义了两种方法：`INDArray.add(INDArray)`和`INDArray.addi(INDArray)`。前者（add）是一个复制操作；后者是一个就地操作-i在addi中表示就地操作。此约定（…i表示就地，没有 i表示复制）适用于通过INDArray接口可访问的其他操作。

假设我们有两个INDArrays x和y，我们做`INDArray z = x.add(y)`或 `INDArray z = x.addi(y)操作`。这些操作的结果如下所示。

![add](http://upload-images.jianshu.io/upload_images/14495907-468c852ab4657cc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![addi](http://upload-images.jianshu.io/upload_images/14495907-59587ecb9c60f514.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意，使用`x.add（y）`操作时，不会修改原始数组`x`。相比之下，对于就地版本`x.addi（y）`，数组x被修改。在添加操作的两个版本中，都会返回包含结果的INDArray。但是请注意，在`addi`操作的情况下，结果数组实际上只是原始数组`x`。

### 标量操作

标量操作是一种元素操作，它也接受一个标量（即一个数字）。标量操作的例子有add、max、multiple、set和divide操作（有关完整列表，请参见上一链接）。

许多方法，例如`INDArray.addi(Number)`和`INDArray.divi(Number)`实际上在后台执行标量操作，因此，在可用时，使用这些方法更方便。

要更直接地执行标量操作，可以使用以下示例：

`Nd4j.getExecutioner().execAndReturn(new ScalarAdd(myArray,1.0))`

请注意，myarray是通过此操作修改的。如果这不是你想要的，请使用 `myArray.dup()`。

与其余的操作不同，标量操作没有对沿维度执行它们的合理解释。

### 转换操作

转换操作是诸如元素方向的对数、余弦、tanh、校正线性等操作。其他示例包括加法、减法和复制操作。转换操作通常以元素为导向的方式使用（例如，对每个元素使用tanh），但情况并非总是如此——例如，softmax通常沿维度执行。

要直接（在完整的NDArray上）执行元素相关的tanh操作，可以使用：

`INDArray tanh = Nd4j.getExecutioner().execAndReturn(new Tanh(myArr))`与上面提到的标量操作一样，使用上面方法的转换操作是就地操作：即修改了NDArray myArr，并且返回的数组tanh实际上是与输入myarr相同的对象。同样，如果需要副本，可以使用`myArr.dup()`。

[Transforms](https://github.com/deeplearning4j/nd4j/blob/master/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/ops/transforms/Transforms.java)类还定义了一些方便的方法，例如：`INDArray tanh = Transforms.tanh(INDArray in,boolean copy)`；这相当于使用 `Nd4j.getExecutioner()的`方法。

### 累加（缩减）操作

在执行累加时，在整个NDArray上执行累加与在特定维度（或维度）上执行累加之间有一个关键区别。在第一种情况下（对整个数组执行），只返回一个值。在第二种情况下（沿维度累加），将返回一个新的NDArray。

要获取数组中所有值的总和，请执行以下操作：

`double sum = Nd4j.getExecutioner().execAndReturn(new Sum(myArray)).getFinalResult().doubleValue();`

或同等（更方便）

`double sum = myArray.sumNumber().doubleValue();`

还可以沿维度执行累加操作。例如，要获取每列中所有值的总和（每列=沿维度0或“每行中的值”），可以使用：

`INDArray sumOfColumns = Nd4j.getExecutioner().exec(new Sum(myArray),0);`

或同等,

`INDArray sumOfColumns = myArray.sum(0)`

假设这是在3x3输入数组上执行的。从视觉上看，这个维度0操作的求和操作如下：

![Sum along dimension 0](http://upload-images.jianshu.io/upload_images/14495907-66fa7e3c5e2685ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.gif](https://upload-images.jianshu.io/upload_images/14495907-f5e657f9dfe84923.gif?imageMogr2/auto-orient/strip) ​

请注意，这里输入的形状为\[3,3\]（3行，3列），输出的形状为\[1,3\]（即，我们的输出是行向量）。如果我们改为沿着维度1进行操作，我们将得到一个形状为\[3,1\]且值为（12,13,11）的列向量。

沿维度的累加也概括为具有3个或更多维度的NDArrays。

### 索引累加操作

索引累加操作与累加操作非常相似。区别在于它们返回的是一个整数索引，而不是一个双精度值。 索引累加操作的示例有IMax（argmax）、IMin（argmin）和IAMax（绝对值的argmax）。 要获取数组中最大值的索引，请执行以下操作：

`int idx = Nd4j.getExecutioner().execAndReturn(new IAMax(myArray)).getFinalResult();`

索引累加操作通常在沿维度执行时最有用。例如，要获取每列（每列=沿维度0）中最大值的索引，可以使用：

`INDArray idxOfMaxInEachColumn = Nd4j.getExecutioner().exec(new IAMax(myArray),0);`

假设这是在3x3输入数组上执行的。从视觉上看，沿维度0操作的argmax/IAMax操作如下：

![Argmax / IAMax](http://upload-images.jianshu.io/upload_images/14495907-7e97cd6ffc375c86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

与上述累加运算一样，输出的形状为\[1,3\]。同样，如果我们沿着维度1进行操作，我们将得到一个形状为\[3,1\]，值为（1,0,2）的列向量。

### 广播与向量操作

ND4J还定义了广播和向量操作。 一些更有用的操作是向量操作，如addRowVector和muliColumnVector。 例如，考虑操作 `x.addRowVector(y)`，其中x是矩阵，而y是行向量。在这种情况下，`addRowVector`操作将行向量y添加到矩阵x的每一行，如下所示。

![addRowVector](http://upload-images.jianshu.io/upload_images/14495907-d8913d1e9a7fc4a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

与其他操作一样，有就地和复制版本。这些操作还有列-列版本，例如`addColumnVector`，它将列向量添加到原始INDArray的每一列。

## 布尔索引：根据条件有选择地应用操作

\[本节：即将到来\]

[Link: Boolean Indexing Unit Tests](https://github.com/deeplearning4j/nd4j/blob/master/nd4j-backends/nd4j-tests/src/test/java/org/nd4j/linalg/indexing/BooleanIndexingTest.java)

## 工作间

工作间是ND4J的一个特性，通过更有效的内存分配和管理，可以提高性能。具体来说，工作空间是为周期性工作负载而设计的，例如训练神经网络，因为它们允许堆外内存重用（而不是在循环的每次迭代中持续分配和释放内存）。净效果是提高性能和减少内存使用。 有关工作间的详细信息，请参见以下链接：

* [Deeplearning4j Guide to Workspaces](https://deeplearning4j.org/workspaces)
* [Workspaces Examples](https://github.com/deeplearning4j/dl4j-examples/blob/master/nd4j-examples/src/main/java/org/nd4j/examples/Nd4jEx15_Workspaces.java)

#### 工作空间：范围恐慌

有时，对于工作区，你可能会遇到一个异常，例如：

```java
org.nd4j.linalg.exception.ND4JIllegalStateException: Op [set] Y argument uses leaked workspace pointer from workspace [LOOP_EXTERNAL]
For more details, see the ND4J User Guide: nd4j.org/userguide#workspaces-panic
```

或

```java
org.nd4j.linalg.exception.ND4JIllegalStateException: Op [set] Y argument uses outdated workspace pointer from workspace [LOOP_EXTERNAL]
For more details, see the ND4J User Guide: nd4j.org/userguide#workspaces-panic
```

**理解范围恐慌异常**

简而言之：这些异常意味着在工作区中分配的INDArray被错误地使用（例如，缺陷或某些方法的错误实现）。这有两个原因：

1. INDArray已从定义的工作区“泄漏”出去。
2. INDArray在正确的工作区内使用，但来自上一次迭代。

在这两种情况下，INDArray指向的底层堆外内存都已失效，无法再使用。 导致工作区泄漏的事件序列示例：

1. 工作间 W 被打开
2. INDArray X 分配在了工作间W
3. 工作间 W 被关闭, 因此X的内存不再有效。
4. INDArray X在某些操作中使用，导致异常。

导致过期工作间指针的事件序列示例：

1. 工作间 W被打开 \(第1次迭代\)
2. INDArray X 在工作间W中分配 \(第1次迭代\)
3. 工作间 W 被关闭 \(第1次迭代\)
4. 工作间 W被打开 \(第2次迭代\)
5. INDArray X \(来自第1次迭代\) 在其它操作中被使用，抛出异常。

**范围恐慌异常的解决方法和修复方法**

根据原因，有两种基本解决方案。 第一。如果已经实现了一些自定义代码（或者正在手动使用工作间），这通常表示代码中存在错误。通常，您有两种选项：

1. 使用`INDArray.detach()`方法从所有工作间分离INDArray。其结果是，返回的数组不再与工作区关联，可以在任何工作区内部或外部自由使用。
2. 首先不要在工作间中分配数组。您可以使用：`try(MemoryWorkspace scopedOut = Nd4j.getWorkspaceManager().scopeOutOfWorkspaces()){ <your code here> }`暂时“关闭”工作间。其结果是，try块中的任何新数组（例如，通过nd4j.create创建的）都不会与工作区关联，并且可以在工作区之外使用。
3. 使用`INDArray.leverage()`或`leverageTo(String)` 或`migrate()`方法之一，将数组移动/复制到父工作间。有关更多详细信息，请参见这些方法的javadoc。

第二，如果你使用工作间为Deeplearning4j的一部分，并且没有实现任何自定义功能（即，你没有编写自己的层、数据管道等），那么（在遇到这种情况的时候），这很可能表示底层库中存在一个bug，通常应该通过GitHub报告问题。一个可能的解决方法是使用以下代码禁用工作区：

```java
.trainingWorkspaceMode(WorkspaceMode.NONE)
.inferenceWorkspaceMode(WorkspaceMode.NONE)
```

如果异常是由于数据管道中的问题造成的，则可以在`AsyncShieldDataSetIterator` 或 `AsyncShieldMultiDataSetIterator中`尝试包装 `DataSetIterator` 或 `MultiDataSetIterator`

无论是哪种原因，如果你确定你的代码是正确的，最后的解决方案是尝试禁用范围恐慌。请注意，这是不推荐的，如果存在合法问题，可能会导致JVM崩溃。为了实现禁用范围恐慌，请在执行代码之前使用`Nd4j.getExecutioner().setProfilingMode(OpExecutioner.ProfilingMode.DISABLED)`。

## 高级和混杂话题

### 设置数据类型

ND4J目前允许用浮点或双精度值来支持INDArrays。默认值为单精度（浮点）。要将nd4j全局用于数组的顺序设置为双精度，可以使用：

```java
Nd4j.setDataType(DataBuffer.Type.DOUBLE);
```

注意，这应该在使用ND4J操作或创建数组之前完成。 或者，可以在启动JVM时设置属性：

```java
-Ddtype=double
```

### 变形

\[本节：即将到来\]

### 扁平化

扁平化是给定数组的一些遍历顺序，将一个或多个INDArrays转换为单个平面数组（行向量）的过程。 ND4J为此提供了以下方法：

```java
Nd4j.toFlattened(char order, INDArray... arrays)
Nd4j.toFlattened(char order, Collection<INDArray>)
```

Nd4j还提供了带有默认顺序的重载toFlattened方法。参数order必须是“c”或“f”，并定义从数组中获取值的顺序：参数order为c导致使用数组索引对数组进行扁平化，其顺序为\[0,0,0\]、\[0,0,1\]等（对于三维数组），而参数order为f导致按\[0,0,0\]、\[1,0,0\]等顺序获取值。

### Permute

\[本节：即将到来\]

### sortRows/sortColumns

\[本节：即将到来\]

### 直接访问BLAS操作

\[本节：即将到来\]

### 系列化

ND4J提供了许多格式的INDArrays序列化。下面是一些二进制和文本序列化的示例：

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.serde.binary.BinarySerde;

import java.io.*;
import java.nio.ByteBuffer;

INDArray arrWrite = Nd4j.linspace(1,10,10);
INDArray arrRead;

//1\. 二进制格式
//   Close the streams manually or use try with resources.
try (DataOutputStream sWrite = new DataOutputStream(new FileOutputStream(new File("tmp.bin")))) {
    Nd4j.write(arrWrite, sWrite);
    }

try (DataInputStream sRead = new DataInputStream(new FileInputStream(new File("tmp.bin")))) {
    arrRead = Nd4j.read(sRead);
    }

//2.使用java.nio.ByteBuffer的二进制格式;
ByteBuffer buffer = BinarySerde.toByteBuffer(arrWrite);
arrRead = BinarySerde.toArray(buffer);

//3\. 文本格式
Nd4j.writeTxt(arrWrite, "tmp.txt");
arrRead = Nd4j.readTxt("tmp.txt");

// 读取csv格式:
// 被声明的writeNumpy方法 
arrRead =Nd4j.readNumpy("tmp.csv", ", ");
```

 [nd4j-serde](https://github.com/deeplearning4j/nd4j/tree/master/nd4j-serde) 目录为  Aeron, base64, camel-routes, gsom, jackson 和 kryo提供包。

### 速查：ND4J方法概述

本节以摘要形式列出了ND4J中最常用的操作。关于其中大部分的更多详细信息，可以在本页后面找到。 在本节中，假设arr、arr1等为INDArrays。

**创建NDArrays**:

* `创建一个零初始化数组：Nd4j.zeros(nRows, nCols)或` `Nd4j.zeros(int...)`
* `创建一个一初始化数组`: `Nd4j.ones(nRows, nCols)`
* `创建一个`NDArray`的副本（复制）：arr.dup()`
* `从double[]创建行/列向量`: `myRow = Nd4j.create(myDoubleArr)`, `myCol = Nd4j.create(myDoubleArr,new int[]{10,1})`
* `从double[][]创建二维`NDArray: `Nd4j.create(double[][])`
* 堆叠一组数组以形成较大的数组：`Nd4j.hstack(INDArray...)`, `Nd4j.vstack(INDArray...)`分别用于水平和垂直
* 均匀随机数NDArrays：`Nd4j.rand(int,int)`, `Nd4j.rand(int[])`等
* Normal\(0,1\) 随机 NDArrays: `Nd4j.randn(int,int)`, `Nd4j.randn(int[])`

确定**INDArray**的大小/尺寸：

以下方法由INDArray接口定义：

* 获取维度数: `rank()`
* 对于  2维 NDArrays 仅有: `rows()`, `columns()`
* 第i维的大小: `size(i)`
* 获取所有维度的大小，以int\[\]返回: `shape()`
* 确定数组中元素的总数: `arr.length()`
* 也参见: `isMatrix()`, `isVector()`, `isRowVector()`, `isColumnVector()`

**获取和设置单个值**:

* 获取第i行第j列的值: `arr.getDouble(i,j)`
* 从一个三维以上的数组获取一个值: `arr.getDouble(int[])`
* 在一个数组中设置单个值: `arr.putScalar(int[],double)`

标量操作：标量操作接受一个double/float/int值，并对每个值执行一个操作，就像元素操作一样，有就地操作和复制操作。

* 添加标量: arr1.add\(myDouble\)
* 减去一个标量: arr1.sub\(myDouble\)
* 乘以一个标量: arr.mul\(myDouble\)
* 除以标量: arr.div\(myDouble\)
* 反向减法 \(scalar - arr1\): arr1.rsub\(myDouble\)
* 反向除法 \(scalar / arr1\): arr1.rdiv\(myDouble\)

元素操作：注意：有复制（添加、mul等）和就地（addi、muli）操作。前者：arr1未修改。后者中：arr1被修改

* 加: `arr1.add(arr2)`
* 减: `arr.sub(arr2)`
* 乘: `add1.mul(arr2)`
* 除: `arr1.div(arr2)`
* `赋值（将arr1中的每个值设置为arr2中的值）：arr1.assign（arr2）`

缩减操作（SUM等）；请注意，这些操作对整个数组进行操作。调用`.doubleValue()`从返回的数字中获取一个双精度值。

* 所有元素的总和: `arr.sumNumber()`
* 所有元素的乘积: `arr.prod()`
* L1和L2范数: `arr.norm1()` 与 `arr.norm2()`
* 各元素标准差: `arr.stdNumber()`

**线性代数操作**:

* 矩阵乘法: `arr1.mmul(arr2)`
* 矩阵转置: `transpose()`
* 求矩阵的对角线: `Nd4j.diag(INDArray)`
* 矩阵求逆: `InvertMatrix.invert(INDArray,boolean)`

获取更大的NDArray的一部分：注意：所有这些方法都返回

* 获取一行 \(仅二维数组NDArrays\): `getRow(int)`
* 获取多行作为一个矩阵\(仅二维数组\): `getRows(int...)`
* 设置一行 \(仅二维数组NDArrays\): `putRow(int,INDArray)`
* 获取前3行，所有列: `Nd4j.create(0).get(NDArrayIndex.interval(0,3),NDArrayIndex.all());`

**元素转换 \(Tanh, Sigmoid, Sin, Log etc\)**:

* 使用 [Transforms](https://github.com/deeplearning4j/nd4j/blob/master/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/ops/transforms/Transforms.java): `Transforms.sin(INDArray)`, `Transforms.log(INDArray)`, `Transforms.sigmoid(INDArray)` 等
* 直接地 \(方法 1\): `Nd4j.getExecutioner().execAndReturn(new Tanh(INDArray))`
* 直接地 \(方法 2\) `Nd4j.getExecutioner().execAndReturn(Nd4j.getOpFactory().createTransform("tanh",INDArray))`

## 常见问题解答：常见问题

**Q: ND4J支持稀疏数组吗？**

目前：不支持，未来计划支持。

**Q:** **是否可以动态增大或缩小INDArray上的大小？**

在当前版本的ND4J中，这是不可能的。不过，我们将来可能会添加此功能。

有两种可能的解决办法：

1. 分配一个新数组并进行复制（例如.put\(\)操作）
2. 最初，预先分配一个大于所需的NDArray，然后对该数组的视图进行操作。然后，由于你需要一个更大的数组，请在原始预分配的数组上获得更大的视图。


---
description: ND4J的主要功能和简要示例。
---

# 快速入门

## 介绍

ND4J是JVM的科学计算库。它是用来在生产环境中使用的，而不是作为一个研究工具，这意味着常规的设计是为了以最低的RAM需求快速运行。主要特点是：

* 一个多功能的n维数组对象。
* 线性代数和信号处理函数。
* 多平台功能，包括GPU。
  * 所有主要操作系统: win/linux/osx/android.
  * 架构: x86, arm, ppc.

 此快速入门遵循与[Numpy快速入门](https://docs.scipy.org/doc/numpy/user/quickstart.html)相同的布局和方法。这将帮助熟悉Python和Numpy的人快速开始使用Nd4J。

## 先决条件

您可以从任何[JVM语言](https://en.wikipedia.org/wiki/List_of_JVM_languages)中使用Nd4J。（例如：Scala、Kotlin）。可以将Nd4J与任何构建工具一起使用。此快速入门中的示例代码使用以下内容：

* [Java \(developer version\)](../kai-shi/kuai-su-ru-men.md#java) 1.7或更高版本（仅支持64位版本）
* [Apache Maven](../kai-shi/kuai-su-ru-men.md#apache-maven) \(自动构建和依赖关系管理器\)
* [Git ](../kai-shi/kuai-su-ru-men.md#git)\(分布式版本控制系统\)

为了提高可读性，我们向您展示`System.out.println(...)`的输出。但是我们还没有在示例代码中显示print语句。如果您有信心知道如何使用maven和git，请随时跳到基础部分。在本节的其余部分中，我们将构建一个小型的“hello ND4J”应用程序，以验证先决条件设置是否正确。

执行以下命令从github获取项目。

```text
git clone https://github.com/RobAltena/HelloNd4J.git

cd HelloNd4J

mvn install

mvn exec:java -Dexec.mainClass="HelloNd4j"
```

当一切设置正确时，您应该看到以下输出：

```text
[         0,         0]
```

## 基础

Nd4j的主要特点是具有多功能的n维阵列接口INDArray。为了提高性能，Nd4j使用堆外内存来存储数据。INDArray不同于标准Java数组。

 INDArray x的一些关键属性和方法如下：

```java
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.buffer.DataType;

INDArray x = Nd4j.zeros(3,4);

// 数组的轴数（维度）。
int dimensions = x.rank();

// 数组的维数。每个维度的大小。
long[] shape = x.shape();

// 元素的总数。
long length = x.length();

// 数组元素的类型。 
DataType dt = x.dataType();
```

### 数组创建

要创建INDArray，可以使用[Nd4j](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html%20)类的静态工厂方法。 

`Nd4j.createFromArray`函数被重载，以便于从常规Java数组创建INDArrays。下面的示例使用Java`double`数组。类似的create方法对于float、int和long重载。对于所有类型，`Nd4j.createFromArray`函数都有高达4d的重载。

```java
double arr_2d[][]={{1.0,2.0,3.0},{4.0,5.0,6.0},{7.0,8.0,9.0}};
INDArray x_2d = Nd4j.createFromArray(arr_2d);

double arr_1d[]={1.0,2.0,3.0};
INDArray  x_1d = Nd4j.createFromArray(arr_1d);
```



Nd4j可以使用函数`zeros`和`ones`创建用0和1初始化的数组。rand函数允许您创建用随机值初始化的数组。创建的INDArray的默认数据类型是float。有些重载允许您设置数据类型。

```java
INDArray  x = Nd4j.zeros(5);
//[         0,         0,         0,         0,         0], FLOAT

int [] shape = {5};
x = Nd4j.zeros(DataType.DOUBLE, 5);
//[         0,         0,         0,         0,         0], DOUBLE

// 对于更高的维度，可以提供形状数组。二维随机矩阵示例：
int rows = 4;
int cols = 5;
int[] shape = {rows, cols};
INDArray x = Nd4j.rand(shape);
```

使用`arange`函数创建一个均匀空间值数组：

```java
INDArray  x = Nd4j.arange(5);
// [         0,    1.0000,    2.0000,    3.0000,    4.0000]

INDArray  x = Nd4j.arange(2, 7);
// [    2.0000,    3.0000,    4.0000,    5.0000,    6.0000]
```

`linspace`函数允许您指定生成的点数：

```java
//开始数, 停止数, 个数.
INDArray  x = Nd4j.linspace(1, 10, 5); 
// [    1.0000,    3.2500,    5.5000,    7.7500,   10.0000]

// 对函数进行多点评估。
import static org.nd4j.linalg.ops.transforms.Transforms.sin;
INDArray  x = Nd4j.linspace(0.0, Math.PI, 100, DataType.DOUBLE);
INDArray  y = sin(x);
```

### 打印数组

INDArray支持Java的`toString()`方法。当前的实现具有有限的精度和有限的元素数。输出类似于打印NumPy数组：

```java
//1维数组
INDArray  x = Nd4j.arange(6);
//我们现在只给出print命令的输出。
System.out.println(x);  
// [         0,    1.0000,    2.0000,    3.0000,    4.0000,    5.0000]

int [] shape = {4,3};
//2维数组
x = Nd4j.arange(12).reshape(shape);   
/*
[[         0,    1.0000,    2.0000], 
 [    3.0000,    4.0000,    5.0000], 
 [    6.0000,    7.0000,    8.0000], 
 [    9.0000,   10.0000,   11.0000]]
*/

int [] shape2 = {2,3,4};
//3维数组
x = Nd4j.arange(24).reshape(shape2); 
/*
[[[         0,    1.0000,    2.0000,    3.0000], 
  [    4.0000,    5.0000,    6.0000,    7.0000], 
  [    8.0000,    9.0000,   10.0000,   11.0000]], 

 [[   12.0000,   13.0000,   14.0000,   15.0000], 
  [   16.0000,   17.0000,   18.0000,   19.0000], 
  [   20.0000,   21.0000,   22.0000,   23.0000]]]
*/
```

### 基本操作

必须使用INDArray方法对数组执行操作。有就地（in-place）和复制重载、标量和元素级重载版本。就地（in-place）运算符返回对数组的引用，因此可以方便地将操作链接在一起。尽可能使用就地（in-place）运算符来提高性能。复制运算符有新的数组创建开销。

```java
//复制
//返回一个新数组，并将标量添加到arr的每个元素。
arr_new = arr.add(scalar);  
//返回一个新数组，它是arr和其他arr元素级别的加法。  
arr_new = arr.add(other_arr); 

//就地
arr_new = arr.addi(scalar); 
arr_new = arr.addi(other_arr);
```

加法: arr.add\(...\), arr.addi\(...\) 减法: arr.sub\(...\), arr.subi\(...\) 乘法: arr.mul\(...\), arr.muli\(...\) 除法 : arr.div\(...\), arr.divi\(...\)

执行基本操作时，必须确保基础数据类型相同。

```java
int [] shape = {5};
INDArray  x = Nd4j.zeros(shape, DataType.DOUBLE);
INDArray  x2 = Nd4j.zeros(shape, DataType.INT);
INDArray  x3 = x.add(x2);
// java.lang.IllegalArgumentException: Op.X 和 Op.Y must have the same data type, but got INT vs DOUBLE

// 将x2转换为DOUBLE可以解决以下问题：
INDArray x3 = x.add(x2.castTo(DataType.DOUBLE));
```

INDArray有实现缩减/累加操作的方法，如 `sum`, `min`, `max`.

```java
int [] shape = {2,3};
INDArray  x = Nd4j.rand(shape);
x;
x.sum();
x.min();
x.max();
/*
[[    0.8621,    0.9224,    0.8407], 
 [    0.1504,    0.5489,    0.9584]]
4.2830
0.1504
0.9584
*/
```

提供维度参数以在指定维度上应用操作：

```java
INDArray x = Nd4j.arange(12).reshape(3, 4);
/*
[[         0,    1.0000,    2.0000,    3.0000], 
 [    4.0000,    5.0000,    6.0000,    7.0000], 
 [    8.0000,    9.0000,   10.0000,   11.0000]]
*/        

//每列的总和。
x.sum(0); 
//[   12.0000,   15.0000,   18.0000,   21.0000]

//每行最小值
x.min(1); 
//[         0,    4.0000,    8.0000]

//每行的累计和，
x.cumsum(1); 
/*
[[         0,    1.0000,    3.0000,    6.0000], 
 [    4.0000,    9.0000,   15.0000,   22.0000], 
 [    8.0000,   17.0000,   27.0000,   38.0000]]
*/
```

### 转换操作

Nd4j提供熟悉的数学函数，如sin、cos和exp，这些称为转换操作。结果作为INDArray返回。

```java
import static org.nd4j.linalg.ops.transforms.Transforms.exp;
import static org.nd4j.linalg.ops.transforms.Transforms.sqrt;

INDArray x = Nd4j.arange(3);
// [         0,    1.0000,    2.0000]
exp(x);
// [    1.0000,    2.7183,    7.3891]
sqrt(x);
// [         0,    1.0000,    1.4142]
```

您可以在[Javadoc](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ops/impl/transforms/package-summary.html)中查看转换操作的完整列表

### 矩陈乘法

我们已经在基本操作中看到了元素的乘法运算。其他矩阵运算有自己的方法：

```java
INDArray x = Nd4j.arange(12).reshape(3, 4);
/*
[[         0,    1.0000,    2.0000,    3.0000], 
 [    4.0000,    5.0000,    6.0000,    7.0000], 
 [    8.0000,    9.0000,   10.0000,   11.0000]]
*/

INDArray y = Nd4j.arange(12).reshape(4, 3);
/*
[[         0,    1.0000,    2.0000], 
 [    3.0000,    4.0000,    5.0000], 
 [    6.0000,    7.0000,    8.0000], 
 [    9.0000,   10.0000,   11.0000]]
*/
// 矩阵乘积.
x.mmul(y);  
/*
[[   42.0000,   48.0000,   54.0000], 
 [  114.0000,  136.0000,  158.0000], 
 [  186.0000,  224.0000,  262.0000]]
*/

//点积
INDArray x = Nd4j.arange(12);
INDArray y = Nd4j.arange(12);
dot(x, y);  
//506.0000
```

### 索引、切片和迭代

索引、切片和迭代在Java中比Python更困难。 要从INDArray检索单个值，可以使用`getDouble`、`getFloat`或`getInt`方法。INDArrays不能像Java数组那样被索引。可以使用`toDoubleVector()`、`toDoubleMatrix()`、`toFloatVector()`和`toFloatMatrix()`从INDArray获取Java数组。

```java
INDArray x = Nd4j.arange(12);
// [         0,    1.0000,    2.0000,    3.0000,    4.0000,    5.0000,    6.0000,    7.0000,    8.0000,    9.0000,   10.0000,   11.0000]
//单一元素访问。其他方法: getDouble, getInt, ...
float f = x.getFloat(3);  
// 3.0
//转换为Java数组。
float []  fArr = x.toFloatVector();
// [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0]

INDArray x2 = x.get(NDArrayIndex.interval(2, 6));
// [    2.0000,    3.0000,    4.0000,    5.0000]


//在x的副本上：从开始到位置6（不包括），将每2个元素设置为-1.0
INDArray y = x.dup();
y.get(NDArrayIndex.interval(0, 2, 6)).assign(-1.0);
//[   -1.0000,    1.0000,   -1.0000,    3.0000,   -1.0000,    5.0000,    6.0000,    7.0000,    8.0000,    9.0000,   10.0000,   11.0000]

//y的反向副本。
INDArray y2 = Nd4j.reverse(y.dup());
//[   11.0000,   10.0000,    9.0000,    8.0000,    7.0000,    6.0000,    5.0000,   -1.0000,    3.0000,   -1.0000,    1.0000,   -1.0000]
```



对于多维数组，应该使用`INDArray.get(NDArrayIndex...)`。下面的示例演示如何遍历二维数组的行和列。注意，对于2D数组，我们可以使用`getColumn`和`getRow`便利方法。

```java
// 在2d数组的行和列上迭代。
int rows = 4;
int cols = 5;
int[] shape = {rows, cols};

INDArray x = Nd4j.rand(shape);
/*
[[    0.2228,    0.2871,    0.3880,    0.7167,    0.9951], 
 [    0.7181,    0.8106,    0.9062,    0.9291,    0.5115], 
 [    0.5483,    0.7515,    0.3623,    0.7797,    0.5887], 
 [    0.6822,    0.7785,    0.4456,    0.4231,    0.9157]]
*/

for (int row=0; row<rows; row++) {
    INDArray y = x.get(NDArrayIndex.point(row), NDArrayIndex.all());
    }
/*
[    0.2228,    0.2871,    0.3880,    0.7167,    0.9951]
[    0.7181,    0.8106,    0.9062,    0.9291,    0.5115]
[    0.5483,    0.7515,    0.3623,    0.7797,    0.5887]
[    0.6822,    0.7785,    0.4456,    0.4231,    0.9157]
*/

for (int col=0; col<cols; col++) {
    INDArray y = x.get(NDArrayIndex.all(), NDArrayIndex.point(col));
    }
/*
[    0.2228,    0.7181,    0.5483,    0.6822]
[    0.2871,    0.8106,    0.7515,    0.7785]
[    0.3880,    0.9062,    0.3623,    0.4456]
[    0.7167,    0.9291,    0.7797,    0.4231]
[    0.9951,    0.5115,    0.5887,    0.9157]
*/
```

## 形状操作

### 改变数组的形状

沿着每个轴的元素的数量是形状。形状可以用各种方法改变。

```java
INDArray x = Nd4j.rand(3,4);
x.shape();
// [3, 4]

INDArray x2 = x.ravel();
x2.shape();
// [12]

INDArray x3 = x.reshape(6,2).shape();
x3.shape();
//[6, 2]

//注意x、x2和x3共享相同的数据。 
x2.putScalar(5, -1.0);

System.out.println( x);
/*
[[    0.0270,    0.3799,    0.5576,    0.3086], 
 [    0.2266,   -1.0000,    0.1107,    0.4895], 
 [    0.8431,    0.6011,    0.2996,    0.7500]]
*/

System.out.println( x2);
// [    0.0270,    0.3799,    0.5576,    0.3086,    0.2266,   -1.0000,    0.1107,    0.4895,    0.8431,    0.6011,    0.2996,    0.7500]

System.out.println( x3);
/*        
[[    0.0270,    0.3799], 
 [    0.5576,    0.3086], 
 [    0.2266,   -1.0000], 
 [    0.1107,    0.4895], 
 [    0.8431,    0.6011], 
 [    0.2996,    0.7500]]
*/
```

### 将不同的数组堆叠在一起

可以使用`vstack`和`hstack`方法将数组堆叠在一起。

```java
INDArray x = Nd4j.rand(2,2);
INDArray y = Nd4j.rand(2,2);

x
/*
[[    0.1462,    0.5037], 
 [    0.1418,    0.8645]]
*/

y;
/*
[[    0.2305,    0.4798], 
 [    0.9407,    0.9735]]
*/

Nd4j.vstack(x, y);
/*
[[    0.1462,    0.5037], 
 [    0.1418,    0.8645], 
 [    0.2305,    0.4798], 
 [    0.9407,    0.9735]]
*/

Nd4j.hstack(x, y);
/*
[[    0.1462,    0.5037,    0.2305,    0.4798], 
 [    0.1418,    0.8645,    0.9407,    0.9735]]
*/
```

## 副本和视图

使用INDArrays时，并不总是复制数据。这里有三种情况你应该注意。

### 完全没有复制

简单的指派不会复制数据。Java通过引用传递对象。在方法调用上不生成任何副本。

```java
INDArray x = Nd4j.rand(2,2);
//y和x指向同一个INDArray对象。
INDArray y = x; 

public static void f(INDArray x){
   //没有副本。对x的任何更改在函数调用之后都是可见的。
}
```

### 视图或浅复制

一些函数将返回数组的视图。

```java
INDArray x = Nd4j.rand(3,4);
INDArray  x2 = x.ravel();
INDArray  x3 = x.reshape(6,2);

// 修改 x, x2 和 x3
x2.putScalar(5, -1.0); 

x
/*
[[    0.8546,    0.1509,    0.0331,    0.1308], 
 [    0.1753,   -1.0000,    0.2277,    0.1998], 
 [    0.2741,    0.8257,    0.6946,    0.6851]]
*/

x2
// [    0.8546,    0.1509,    0.0331,    0.1308,    0.1753,   -1.0000,    0.2277,    0.1998,    0.2741,    0.8257,    0.6946,    0.6851]

x3
/*
[[    0.8546,    0.1509], 
 [    0.0331,    0.1308], 
 [    0.1753,   -1.0000], 
 [    0.2277,    0.1998], 
 [    0.2741,    0.8257], 
 [    0.6946,    0.6851]]
*/
```

### 深考贝

要复制数组，请使用`dup`方法。这将为您提供一个包含新数据的新数组。

```java
INDArray x = Nd4j.rand(3,4);
INDArray  x2 = x.ravel().dup();

//现在只改变x2。
x2.putScalar(5, -1.0); 
x
/*
[[    0.1604,    0.0322,    0.8910,    0.4604], 
 [    0.7724,    0.1267,    0.1617,    0.7586], 
 [    0.6117,    0.5385,    0.1251,    0.6886]]
*/

x2
// [    0.1604,    0.0322,    0.8910,    0.4604,    0.7724,   -1.0000,    0.1617,    0.7586,    0.6117,    0.5385,    0.1251,    0.6886]
```

## 功能和方法概述

### 数组创建

[arange](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#arange-double-double-%20), [create](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#create-org.nd4j.linalg.api.buffer.DataBuffer-%20), [copy](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#copy-org.nd4j.linalg.api.ndarray.INDArray-org.nd4j.linalg.api.ndarray.INDArray-%20), [empty](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#empty-org.nd4j.linalg.api.buffer.DataBuffer.Type-%20), [empty\_like](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#emptyLike-org.nd4j.linalg.api.ndarray.INDArray-%20), [eye](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#eye-long-%20), [linspace](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#linspace-double-double-long-%20), [meshgrid](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#meshgrid-org.nd4j.linalg.api.ndarray.INDArray-org.nd4j.linalg.api.ndarray.INDArray-%20), [ones](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#ones-int...-%20), [ones\_like](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#onesLike-org.nd4j.linalg.api.ndarray.INDArray-%20), [rand](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#rand-int-int-%20), [readTxt](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#readTxt-java.lang.String-%20), [zeros](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#zeros-int...-%20), [zeros\_like](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#zerosLike-org.nd4j.linalg.api.ndarray.INDArray-%20)

### 转换

[convertToDoubles](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#convertToDoubles--%20), [convertToFloats](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#convertToFloats--%20), [convertToHalfs](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#convertToHalfs--%20)

### 操作

[concatenate](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#concat-int-org.nd4j.linalg.api.ndarray.INDArray...-%20), [hstack](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#hstack-org.nd4j.linalg.api.ndarray.INDArray...-%20), [ravel](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#ravel--%20), [repeat](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#repeat-int-long...-%20), [reshape](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#reshape-long...-%20), [squeeze](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#squeeze-org.nd4j.linalg.api.ndarray.INDArray-int-%20), [swapaxes](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#swapAxes-int-int-%20), [tear](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#tear-org.nd4j.linalg.api.ndarray.INDArray-int...-%20), [transpose](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#transpose--%20), [vstack](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#vstack-org.nd4j.linalg.api.ndarray.INDArray...-%20)

### 排序

[argmax](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#argMax-int...-%20), [max](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#max-int...-%20), [min](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#min-int...-%20), [sort](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#sort-org.nd4j.linalg.api.ndarray.INDArray-int-boolean-%20)

### 操作

[choice](https://deeplearning4j.org/api/latest/org/nd4j/linalg/factory/Nd4j.html#choice-org.nd4j.linalg.api.ndarray.INDArray-org.nd4j.linalg.api.ndarray.INDArray-org.nd4j.linalg.api.ndarray.INDArray-%20), [cumsum](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#cumsum-int-%20), [mmul](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#mmul-org.nd4j.linalg.api.ndarray.INDArray-%20), [prod](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#prod-int...-%20), [put](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#put-org.nd4j.linalg.api.ndarray.INDArray-org.nd4j.linalg.api.ndarray.INDArray-%20), [putWhere](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#putWhere-org.nd4j.linalg.api.ndarray.INDArray-org.nd4j.linalg.api.ndarray.INDArray-org.nd4j.linalg.indexing.conditions.Condition-%20), [sum](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#sum-int...-%20)

### 基本统计

[covarianceMatrix](https://deeplearning4j.org/api/latest/org/nd4j/linalg/dimensionalityreduction/PCA.html#getCovarianceMatrix--), [mean](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#mean-int...-%20), [std](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#std-int...-%20), [var](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#var-int...-%20)

### 基本线性代数

[cross](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ops/impl/shape/Cross.html%20), [dot](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ops/impl/accum/Dot.html%20), [gesvd](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/blas/Lapack.html#gesvd-org.nd4j.linalg.api.ndarray.INDArray-org.nd4j.linalg.api.ndarray.INDArray-org.nd4j.linalg.api.ndarray.INDArray-org.nd4j.linalg.api.ndarray.INDArray-%20), [mmul](https://deeplearning4j.org/api/latest/org/nd4j/linalg/api/ndarray/INDArray.html#mmul-org.nd4j.linalg.api.ndarray.INDArray-)


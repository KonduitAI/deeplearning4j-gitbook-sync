# 参数空间

### 参数层空间

#### 布尔空间

[\[源码\]](https://github.com/deeplearning4j/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/BooleanSpace.java)

如果参数被设置的值小于或等于0.5它将返回true，否则是false。

#### 固定值

[\[源码\]](https://github.com/deeplearning4j/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/FixedValue.java)

固定值是只定义单个固定值的参数空间。

#### 连续型参数空间

[\[源码\]](https://github.com/deeplearning4j/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/continuous/ContinuousParameterSpace.java)

**getValue**

```java
public Double getValue(double[] input)
```

最小值与最大值之间均匀分布的连续参数空间

* 参数 min 可生成的最小值
* 参数 max 可生成的最大值

#### 离散型参数空间

[\[源码\]](https://github.com/deeplearning4j/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/discrete/DiscreteParameterSpace.java)

用于未排序值集合

#### 整型参数空间

[\[源码\]](https://github.com/deeplearning4j/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/integer/IntegerParameterSpace.java)

一些最小值和最大值

**getMin**

```text
public int getMin()
```

在指定的最小/最大（包含）之间创建一个具有均匀分布的整数参数空间

* 参数 min  的最小值, 包括最小值
* 参数 max 的最大值, 包括最大值

#### 数学运算

[\[源码\]](https://github.com/deeplearning4j/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/math/MathOp.java)

在另一个参数空间上实现标量数学运算的简单参数空间。这允许你做像 Y=2X,X是参数空间。例如，可以将一个层大小超参数用前一层的大小2x来设置 。

#### 配对数学运算

[\[源码\]](https://github.com/deeplearning4j/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/math/PairMathOp.java)

在另一个参数空间上实现成对数学运算的简单参数空间。这允许你做诸如z＝x+y之类的事情，其中x和y是参数空间。


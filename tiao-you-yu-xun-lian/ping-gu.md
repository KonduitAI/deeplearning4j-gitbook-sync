---
description: 评估神经网络性能的工具和类
---

# 评估

### 为什么要评估? <a id="why-evaluate"></a>

当训练或部署神经网络时，了解模型的准确性是有用的。在DL4J中，评估类和评估类的变体可用于评估模型的性能。

#### 分类评价 <a id="evaluation-for-classification"></a>

评估类用于评估二分类和多类分类器（包括时间序列分类器）的性能。本节介绍了评估类的基本用法。

给定一个DataSetIterator形式的数据集，执行评估的最简单方法是使用MultiLayerNetwork和ComutationGraph上的内置评估方法：

```java
DataSetIterator myTestData = ...
Evaluation eval = model.evaluate(myTestData);
```

然而，也可以对单个小批量进行评价。这里是一个例子，从我们的[示例项目](https://github.com/deeplearning4j/dl4j-examples)中数据实例/CSV示例中获得。

CSV的例子有3类花的CSV数据，建立了一个简单的前馈神经网络用于对基于4个测量值的花的分类。

```java
Evaluation eval = new Evaluation(3);
INDArray output = model.output(testData.getFeatures());
eval.eval(testData.getLabels(), output);
log.info(eval.stats());
```

第一行创建一个具有3个类的评估对象。第二行从模型中获取我们测试数据集的标签。第三行使用eval方法将来自testdata的标签数组与从模型生成的标签进行比较。第四行将评估数据记录到控制台。

输出

```text
Examples labeled as 0 classified by model as 0: 24 times
Examples labeled as 1 classified by model as 1: 11 times
Examples labeled as 1 classified by model as 2: 1 times
Examples labeled as 2 classified by model as 2: 17 times


==========================Scores========================================
 # of classes:    3
 Accuracy:        0.9811
 Precision:       0.9815
 Recall:          0.9722
 F1 Score:        0.9760
Precision, recall & F1: macro-averaged (equally weighted avg. of 3 classes)
========================================================================
```

默认情况下，.stats\(\) 方法显示混淆矩阵条目（每行一个）、准确度、精度、召回率和F1分数。此外，评估类还可以计算并返回以下值：

* 混淆矩阵
* 假阳性/阴性率
* 真阳性/阴性
* 类别计数
* F-beta, G-measure, Matthews 关系数及更多, 查看 [Evaluation JavaDoc](https://deeplearning4j.org/api/latest/org/deeplearning4j/eval/Evaluation.html)

显示混淆矩阵。

```java
System.out.println(eval.confusionToString());
```

显示

```text
Predicted:         0      1      2
Actual:
0  0          |      16      0      0
1  1          |       0     19      0
2  2          |       0      0     18
```

此外，可以直接访问混淆矩阵，使用CSV或HTML转换。

```java
eval.getConfusionMatrix() ;
eval.getConfusionMatrix().toHTML();
eval.getConfusionMatrix().toCSV();
```

### 回归评估

为了评估执行回归的网络，使用回归评估类。

带着评估类，一个DataSetIterator上的回归评估可以执行如下：

```java
DataSetIterator myTestData = ...
RegressionEvaluation eval = model.evaluateRegression(myTestData);
```

这里有一个单列的代码片段，在这种情况下，神经网络是根据测量值来预测自己的年龄。

```java
RegressionEvaluation eval =  new RegressionEvaluation(1);
```

打印评估的统计数据。

```java
System.out.println(eval.stats());
```

返回

```text
Column    MSE            MAE            RMSE           RSE            R^2            
col_0     7.98925e+00    2.00648e+00    2.82653e+00    5.01481e-01    7.25783e-01    
```

列是均方误差、均方绝对误差、均方根误差、相对平方误差和R^2决定系数。

查看 [回归评估JavaDoc](https://deeplearning4j.org/api/latest/org/deeplearning4j/eval/RegressionEvaluation.html)

### 同时进行多个评估

当执行多种类型的评估时（例如，在同一网络和数据集上执行评估和ROC），在数据集的一次传递中执行以下操作更有效：

```java
DataSetIterator testData = ...
Evaluation eval = new Evaluation();
ROC roc = new ROC();
model.doEvaluation(testdata, eval, roc);
```

### 时间序列评估

时间序列评估与上述评估方法非常相似。DL4J中的评估对所有（非掩码的）时间步分别执行——例如，长度为10的时间序列将为评估对象贡献10个预测/标签。与时间序列的一个不同之处在于掩码数组是（可选的），这些掩码数组用于将一些时间步标记为丢失或不存在。请参阅[使用RNNS掩码](https://deeplearning4j.org/docs/latest/deeplearning4j-nn-recurrent)以获得更多关于掩码的细节。

对于大多数用户来说，仅仅使用 `MultiLayerNetwork.evaluate(DataSetIterator)` 或  `MultiLayerNetwork.evaluateRegression(DataSetIterator)` 和类似的方法就足够了。如果掩码数组存在，这些方法将正确地处理掩码。

### 二分类器评估

EvaluationBinary用于评估具有二分类输出的网络——这些网络通常具有Sigmoid激活函数和XENT损失函数。为每个输出计算典型的分类度量，例如准确度、精度、召回率、F1得分等。

```java
EvaluationBinary eval = new EvaluationBinary(int size)
```

查看 [EvaluationBinary JavaDoc](https://deeplearning4j.org/api/latest/org/deeplearning4j/eval/EvaluationBinary.html)

### ROC

ROC（接收者操作特征）是另一种常用的评估分类器的评估指标。DL4J中存在三个ROC变体：

* ROC -用“一对全部”的方法评估非二分类器
* ROCBinary - 用于单二分类标签（作为单列概率，或两列的softmax概率分布）
* ROCMultiClass - 用于多二分类标签

这些类具有通过calculateAUC\(\)和calculateAUPRC\(\)方法计算ROC曲线下面积\(AUROC\)和精确度-召回曲线下面积\(AUPRC\)的能力。此外，可以使用`getRocCurve()`和`getPrecisionRecallCurve()`获得ROC和精确度-召回曲线。

ROC和精确度-召回曲线可以导出到HTML以便查看，使用:“EvaluationTools.exportRocChartsToHtmlFile\(ROC,File\)”，该文件将导出具有ROC和精确度-召回曲线的HTML文件，可以在浏览器中查看。

注意，所有三种支持两种操作/计算模式。

* 阈值（近似AUROC/AUPRC计算，无内存问题）
* 精确（精确的AUROC/AUPRC计算，但是对于非常大的数据集（即具有数百万个示例的数据集）可能需要大量的内存

可以使用构造函数设置容器的数量。可以使用默认构造函数`new ROC()`来精确设置，或者显式地使用`new ROC(0)`。

参见[ROCBinary JavaDoc](https://deeplearning4j.org/api/latest/org/deeplearning4j/eval/ROC.html)用于评估二元分类器。

### 评估分类器校准

DL4J还具有评估校准类，它被设计用于分析分类器的校准。它提供了许多的工具用于如下目的：

* 每个类别的标签数量和预测的计数
* 可靠性图（或可靠性曲线）
* 残差图（直方图）
* 概率直方图，包括每个类的概率

使用评估校准的分类器评估方式与其它评估类相似。可以使用`EvaluationTools.exportevaluationCalibrationToHtmlFile(EvaluationCalibration, File)`将各种绘图/直方图导出到HTML以便查看。

### Spark网络的分布式评估

SparkDl4jMultiLayer 和 SparkComputationGraph 都有相似的评估方法:

```text
Evaluation eval = SparkDl4jMultiLayer.evaluate(JavaRDD<DataSet>);

//一次传递多次评价：
SparkDl4jMultiLayer.doEvaluation(JavaRDD<DataSet>, IEvaluation...);
```

### 多任务网络评估

多任务网络是经过训练以产生多个输出的网络。例如，可以对给定音频样本的网络进行训练，以预测说话者的语言和说话人的性别。[这里](https://deeplearning4j.org/docs/latest/deeplearning4j-nn-computationgraph)简要描述了多任务配置。

适用于多任务网络的评估类

查看 [ROCMultiClass JavaDoc](https://deeplearning4j.org/api/latest/org/deeplearning4j/eval/ROCMultiClass.html)

查看 [ROCBinary JavaDoc](https://deeplearning4j.org/api/latest/org/deeplearning4j/eval/ROC.html)

## 可用的评估


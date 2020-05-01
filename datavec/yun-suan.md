---
description: 高级转换的实现。
---

# 运算

## 使用

运算，就像一个函数，帮我们执行一个转换并加载数据到数据向量。运算的概念是很底层的，这意味着大多数时间你不需要关心它们。

### 加载数据到Spark如果你正在使用Apache Spark,函数将迭代数据集并加载它到一个Spark `RDD`里并把原始数据转换为一个Writable。

```java
import org.datavec.api.writable.Writable;
import org.datavec.api.records.reader.impl.csv.CSVRecordReader;
import org.datavec.spark.transform.misc.StringToWritablesFunction;

SparkConf conf = new SparkConf();
JavaSparkContext sc = new JavaSparkContext(conf)

String customerInfoPath = new ClassPathResource("CustomerInfo.csv").getFile().getPath();
JavaRDD<List<Writable>> customerInfo = sc.textFile(customerInfoPath).map(new StringToWritablesFunction(rr));
```

以上代码加载一个CSV文件到一个 2D java RDD。一旦你的RDD被加载，你可以转换它，执行连接并使用缩减器以任何方式处理数据。


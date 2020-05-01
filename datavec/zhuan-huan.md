# 转换

## 数据整理

数据向量的关键工具之一是转换。数据向量帮助用户将数据集从一个概要映射到另一个概要，并提供一个操作列表来转换类型，格式化数据，把一个2D数据集转换成系列数据。

## 构建一个转换过程

一个转换过程需要一个概要来成功地转换数据。概要和转换过程类都附带一个帮助构建器类，对于组织代码和避免复杂的构建器来说是很有用的。当两者结合起来它们看起来像如下的样例代码。请注意inputDataSchema是如何传到Builder构造器的。没有它，你的转换过程将会编译失败。

```java
import org.datavec.api.transform.TransformProcess;

TransformProcess tp = new TransformProcess.Builder(inputDataSchema)
    .removeColumns("CustomerID","MerchantID")
    .filter(new ConditionFilter(new CategoricalColumnCondition("MerchantCountryCode", ConditionOp.NotInSet, new HashSet<>(Arrays.asList("USA","CAN")))))
    .conditionalReplaceValueTransform(
        "TransactionAmountUSD",     //被操作的列
        new DoubleWritable(0.0),    //如果条件满足，用新的值
        new DoubleColumnCondition("TransactionAmountUSD",ConditionOp.LessThan, 0.0)) //条件: amount < 0.0
    .stringToTimeTransform("DateTimeString","YYYY-MM-DD HH:mm:ss.SSS", DateTimeZone.UTC)
    .renameColumn("DateTimeString", "DateTime")
    .transform(new DeriveColumnsFromTimeTransform.Builder("DateTime").addIntegerDerivedColumn("HourOfDay", DateTimeFieldType.hourOfDay()).build())
    .removeColumns("DateTime")
    .build();
```

## 执行一个转换

现在有不同的执行器后台可以使用。使用上面的转换过程对象tp，这里是你如何用一个数据向量在本地执行它。

```java
import org.datavec.local.transforms.LocalTransformExecutor;

List<List<Writable>> processedData = LocalTransformExecutor.execute(originalData, tp);
```

## 调式

在概要发生变化的时候在一个转换过程中的每个操作代表一个步骤。有时候，转换的结果不是预期想要的。你可以按如下通过打印转换过程对象tp中每个步骤调试这它。

```java
//Now, print the schema after each time step:
int numActions = tp.getActionList().size();

for(int i=0; i<numActions; i++ ){
    System.out.println("\n\n==================================================");
    System.out.println("-- Schema after step " + i + " (" + tp.getActionList().get(i) + ") --");

    System.out.println(tp.getSchemaAfterStep(i));
}
```

## 可用的转换和变换


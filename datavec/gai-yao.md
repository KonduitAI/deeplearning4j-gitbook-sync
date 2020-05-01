---
description: 数据集和转换概要
---

# 概要

### 为什么要使用概要？

现实中的不幸是数据是脏的。当为了深度学习而试图向量化一个数据集时，很少能找到没有错的文件。在使用神经网络训练神经网络之前，概要对于维护数据的意义是很重要的。

### 使用概要

概要基本上用于程序设计变换。在正确执行转换过程之前，需要传递正在转换的数据的概要。一个用于商家记录的概要的例子看起来如下：

```java
Schema inputDataSchema = new Schema.Builder()
    .addColumnsString("DateTimeString", "CustomerID", "MerchantID")
    .addColumnInteger("NumItemsInTransaction")
    .addColumnCategorical("MerchantCountryCode", Arrays.asList("USA","CAN","FR","MX"))
    .addColumnDouble("TransactionAmountUSD",0.0,null,false,false)   //$0.0 or more, no maximum limit, no NaN and no Infinite values
    .addColumnCategorical("FraudLabel", Arrays.asList("Fraud","Legit"))
    .build();
```

### 概要连接

如果你有两个你想要合并的不同的数据集，数据向理提供一个Join连接类，它有不同的连接策略，例如 Inner内连和RightOuter右外连

```java
Schema customerInfoSchema = new Schema.Builder()
    .addColumnLong("customerID")
    .addColumnString("customerName")
    .addColumnCategorical("customerCountry", Arrays.asList("USA","France","Japan","UK"))
    .build();

Schema customerPurchasesSchema = new Schema.Builder()
    .addColumnLong("customerID")
    .addColumnTime("purchaseTimestamp", DateTimeZone.UTC)
    .addColumnLong("productID")
    .addColumnInteger("purchaseQty")
    .addColumnDouble("unitPriceUSD")
    .build();

Join join = new Join.Builder(Join.JoinType.Inner)
    .setJoinColumns("customerID")
    .setSchemas(customerInfoSchema, customerPurchasesSchema)
    .build();
```

一旦你已经定义了你的连接并且你已经加载了数据到数据向量，你必须使用一个Executor执行器来完成连接。

### Classes and utilities 类和实用工具


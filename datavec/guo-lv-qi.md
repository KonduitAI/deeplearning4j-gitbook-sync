---
description: 数据使用条件的选择。
---

# 过滤器

## 使用过滤器

过滤器是转换的一部分，并为你保留你数据集的一部分给定了一个领域特定语言。过滤器可以是单一条件的一个内衬，也可以包括复杂的布尔逻辑。

```java
TransformProcess tp = new TransformProcess.Builder(inputDataSchema)
    .filter(new ConditionFilter(new CategoricalColumnCondition("MerchantCountryCode", ConditionOp.NotInSet, new HashSet<>(Arrays.asList("USA","CAN")))))
    .build();
```

你也可以通过实现接Filter口来写自己的过滤器，尽管在更多的时候你想要创建一个定制的条件来替代。


---
description: "从一个概要到另一个\b概要的数据整理和映射。"
---

# 序列化

## 序列化转换

数据向量附带系列化转换的能力，这允许在生产环境需要转换时它们更加可移植。一个TransformProcess（转换过程）被序列化为一个人类可读的格式，例如JSON，并且可以保存为文件。

## 序列化

如下这行代码展示了你可以如何系列化转换过程 tp对象

```java
String serializedTransformString = tp.toJson()
```

## 反序列化

当你想要重新实例化转换过程的时候，调用静态方法fromJson

```java
TransformProcess tp = TransformProcess.fromJson(serializedTransformString)
```


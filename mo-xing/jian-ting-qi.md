---
description: 在DL4J模型上添加钩子和监听器。
---

# 监听器

## 什么是监听器?

监听器允许用户在Eclipse DL4J中“挂钩”到某些事件中。这允许你收集或打印对训练等任务有用的信息。例如，一个`ScoreIterationListener`允许你从神经网络的输出层打印训练分数。

## 用法

要将一个或多个监听器添加到一个多层网络或计算图中，请使用`addListener`方法：

```java
MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
//打印每次迭代的分数
model.setListeners(new ScoreIterationListener(1));
```




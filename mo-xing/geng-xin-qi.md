---
description: 梯度下降的特殊算法。
---

# 更新器

## 什么是更新器?

更新器之间的主要区别是他们如何对待学习率。随机梯度下降是深度学习中最常用的学习算法，它依赖于`Theta`\(隐藏层中的权重\)和`alpha`\(学习率\)。不同的更新器有助于优化学习速率，直到神经网络收敛到其最高性能状态为止。

## 用法

若要使用更新器，请将一个新类传递到计算图或多层网络中的`updater()`方法。

```java
ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .updater(new Adam(0.01))
    // 在下面添加你的层和超参数
    .build();
```




---
description: 梯度下降的特殊算法。
---

# 激活

## 什么是激活？

在一个简单的层次上，激活函数有助于决定神经元是否应该被激活。这有助于确定神经元接收的信息是否与输入相关。激活函数是在输入信号上发生的非线性变换，并且变换后的输出被发送到下一个神经元。

## 用法

使用激活的推荐方法是在你的神经网络中添加激活层，并配置所需的激活：

```java
GraphBuilder graphBuilder = new NeuralNetConfiguration.Builder()
    // 添加超参数和其他层
    .addLayer("softmax", new ActivationLayer(Activation.SOFTMAX), "previous_input")
    // 添加更多的层和输出
    .build();
```


---
description: 关于Eclipse DeepLearning4J、深度学习和人工智能的常见问题。
---

# FAQ

##  为什么要用Deeplearning4J？

DL4J有一个用于Java和Scala的通用n维数组类，它可以在Hadoop上进行扩展，利用GPU支持AWS上的扩展，包括一个用于机器学习libs的通用矢量化工具，而且大多数都依赖于ND4J：一个比Numpy快得多的矩阵库，并且大部分是用C++编写的。我们还构建了RL4J:使用Deep Q learning和A3C的Java强化学习。

##  人工智能和机器学习的使用场景是什么？

像Deeplearning4j这样的人工智能工具可以应用于机器人过程自动化（RPA）、欺诈检测、网络入侵检测、推荐系统（CRM、adtech、客户流失预防）、回归和预测分析、人脸/图像识别、语音搜索、语音到文本（转录）和预防性硬件监控（异常检测）。

##  我该怎么贡献？

 希望为Deeplearning4j做出贡献的开发人员可以从阅读我们的贡献者指南开始。

##  DL4J是并行的多线程的吗？

Deeplearning4j包括一个分布式的多线程深度学习框架和一个普通的单线程深度学习框架。训练在集群中进行，这意味着它可以快速处理大量数据。网络通过迭代reduce进行并行训练，并且它们与Java、Scala、Clojure和Kotlin具有同等的兼容性。Deeplearning4j作为开放堆栈中的模块化组件，使其成为第一个适用于微服务架构的深度学习框架。


---
description: 如何贡献Eclipse Deeplearning4j源代码。
---

# 贡献

## 先决条件

在做出贡献之前，确保你知道所有的Eclipse DL4J库的结构。早在2018年初，所有的库都入住在Deeplearning4j [monorepo](https://github.com/deeplearning4j/deeplearning4j)。这些包括：

* DeepLearning4J: 包含用于既在单个机器上，又在分布式上学习神经网络的所有代码。
* ND4J: “Java的n维数组”。ND4J是建立DL4J的数学后端。所有的DL4J神经网络都是使用ND4J中的运算（矩阵乘法、向量运算等）来构建的。ND4J是DL4J实现在没有改变网络本身的情况化，即可以CPU又可以GPU训练网络的原因。  没有Nd4J，就不会有DL4J。
* DataVec: DataVec处理管道侧的数据导入和转换。如果你  想将图像、视频、音频或简单CSV数据导入DL4J：你可能想要使用DataVec来实现。
* Arbiter: Arbiter是一种用于神经网络超参数优化的软件包。超参数优化是指自动选择网络超参数（学习速率、层数等）以获得良好性能的过程。

我们也有一个额外的示例仓库在 [dl4j-examples](https://github.com/deeplearning4j/dl4j-examples).

## 贡献的方式

有多种方式为DeepLearning4J \(和相关项目\)做贡献，这取决于你的兴趣和经验。这有一些建议：

* 添加一个新的神经网络类型 \(例如: 不同类型的 RNNs, 本地连接网络等\)
* 添加一个新的训练特征
* 修复缺陷
* DL4J 示例: 那儿有我们没有示例的程序或神经网络架构吗？
* 测试性能和识别瓶颈或可以改进的地方。
* 改进网站的文档（或写教程等）
* 改进java文档

这有很多不同的方法来寻找工作。这些包括：

* 查看缺陷跟踪: [https://github.com/deeplearning4j/deeplearning4j/issues](https://github.com/deeplearning4j/deeplearning4j/issues) [https://github.com/deeplearning4j/dl4j-examples/issues](https://github.com/deeplearning4j/dl4j-examples/issues)
* 回顾我们的路线图
* 与开发人员交谈，特别是我们的早期使用者栏目
* 回顾最近的论文和博客文章关于训练特征、神经网络架构和应用。
* 回顾网站和例子-有什么什么似乎缺少，不完整，或只是有用的（或酷）？

## 一般准则

在你开始做之前，有一些事情你需要知道。特别是我们使用的工具:

* Maven: 一个依赖性管理和构建工具，用于我们所有的项目。有关Maven的详细信息，请参见此。
* Git: 我们使用的版本控制系统
* Project Lombok:  Lombok项目是一种代码生成/注释工具，其目的是减少Java中所需的“多余”代码（即标准重复代码）的数量。要使用源代码，你需要为IDE安装Lombok插件。
* VisualVM:分析工具，最有效的识别性能问题和瓶颈。
* IntelliJ IDEA: 这是我们的IDE选择，当然，你可能会使用替代品，如Eclipse和NetBeans，你可能会发现使用与开发人员相同的IDE更容易避免出现问题。但这取决于你。

要记住的事情:

* 代码应该符合Java 7 或以上 
* 如果你添加一个新的方法或类：添加java文档
* 为一个明显的功能添加添加一个作者标签是受大家欢迎的。这也可以帮助特征贡献者，以防他们需要问原作者问题。如果多个作者出现在一个类中：提供谁做了什么的细节（“初始实现”，“添加一个特征”等）
* 在你的代码中提供有益的注释。这有助于保持所有代码的可维护性。
* 任何新的功能都应该包括单元测试（使用JUnit）来测试代码。这应该包括边缘情况。
* 如果添加新的层类型，则必须按照这些单元测试包含数值梯度检查。这些是必要的，以确认计算的梯度是正确的。
* 如果你正在添加重要的新功能，请考虑更新网站的相关部分，并提供一个示例。毕竟，没有人知道的功能（或者没有人知道如何使用）是没有用的。添加合适的文档是非常受到鼓励的，但不是严格要求的。
* 如果你对某些事不确定，请在Gitter 上咨询我们。




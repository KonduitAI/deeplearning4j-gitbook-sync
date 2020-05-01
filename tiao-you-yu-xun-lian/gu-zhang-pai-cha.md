---
description: 了解常见错误如NaNs和调整超参数。
---

# 故障排查

### 神经网络训练的故障排查

神经网络很难调优。如果网络超参数选择不当，网络学习可能会慢，或者根本不学习。本页旨在提供在调优网络时应采取的一些基准步骤。

这些技巧中的许多已经在学术文献中讨论过。我们的目的是把它们合并在一个网站中，并尽可能清楚表达他们。

### 内容

* [数据归一化](gu-zhang-pai-cha.md#shu-ju-gui-yi-hua)
* [权重初始化](gu-zhang-pai-cha.md#quan-zhong-chu-shi-hua)
* [Epoch（轮）与迭代](gu-zhang-pai-cha.md#epoch-lun-shu-liang-yu-die-dai-shu-liang)
* [学习率](gu-zhang-pai-cha.md#xue-xi-lv)
* [激活函数](gu-zhang-pai-cha.md#ji-huo-han-shu)
* [损失函数](gu-zhang-pai-cha.md#sun-shi-han-shu) 
* [正则化 ](gu-zhang-pai-cha.md#zheng-ze-hua)
* [小批量大小](gu-zhang-pai-cha.md#xiao-pi-liang-da-xiao)
* [更新器与优化算法 ](gu-zhang-pai-cha.md#geng-xin-qi-he-you-hua-suan-fa)
* [梯度归一化](gu-zhang-pai-cha.md#ti-du-gui-yi-hua)
* [循环神经网络](gu-zhang-pai-cha.md#xun-huan-shen-jing-wang-luo-tong-guo-shi-jian-jie-duan-de-fan-xiang-chuan-bo)
* [深度信念网络](gu-zhang-pai-cha.md#ke-jian-yin-cang-dan-yuan)
* [受限波尔滋曼机 ](gu-zhang-pai-cha.md#shou-xian-bo-er-zi-man-ji)
* [NaN不是数字问题](gu-zhang-pai-cha.md#nan-bu-shi-yi-ge-shu-zi-cuo-wu)

## [数据归一化](gu-zhang-pai-cha.md)

你的数据分布是什么？你正在适当地缩放它吗？作为一般规则：

* 对于连续值：你希望这些值在-1到1、0到1的范围内，或者以平均值0和标准偏差1正态分布。这不一定是准确的，但确保在训练期间你的输入大约在这个范围内会有所帮助。缩小大输入，扩大小输入。
* 对于离散类（以及对于输出的分类问题），通常使用one-hot表示。也就是说，如果你有3个类，那么对于3个类中的每一个，你的数据将分别重新表示为 \[1,0,0\]、\[0,1,0\]或\[0,0,1\]。

注意，对于训练数据和测试数据，使用完全相同的归一化方法是非常重要的。

## [权重初始化](gu-zhang-pai-cha.md)

DL4J支持几种不同类型的权重初始化，使用权重初始化参数。在你的配置中使用.weightInit\(WeightInit\)方法来设置。

你需要确保你的权重既不太大也不太小。Xavier权重初始化通常是一个很好的选择。对于具有整流线性（Relu）或leaky relu激活的网络，Relu权重初始化是明智的选择。

## [Epoch（轮）数量与迭代数量](gu-zhang-pai-cha.md)

一个epoch被定义为数据集的完整传递。

太少的epoch没有给你的网络足够的时间来学习好的参数；太多，你可能会过度拟合训练数据。选择epoch数量的一种方法是使用早停。早停还可以帮助防止神经网络过拟合\(即，可以帮助网络更好地对看不见的数据进行泛化\)。

## [学习率](gu-zhang-pai-cha.md)

学习率不是最重要的超参数之一。如果这是太大或太小，你的网络可能会学习很差，非常缓慢，或根本没有。学习速率的典型值在0.1至1e-6的范围内，然而最佳学习速率通常是特定于数据（和网络架构）的。一些简单的建议是先尝试三种不同的学习速率——1e-1、1e-3和1e-6——在进一步调优之前大致了解应该做什么。理想情况下，他们同时运行具有不同学习速率的模型，以节省时间。

选择适当学习率的通常方法是使用DL4J的可视化界面来可视化训练过程。你需要同时注意时间上的损失，以及更新量与参数量的比率（大约1:1000的比例是一个好的开始）。有关调整学习速率的更多信息，请参见[此链接](https://cs231n.github.io/neural-networks-3/#baby)。

与在单台机器上训练相同的网络相比，以分布式方式训练神经网络，可能需要不同的（经常更高的）学习速率。

## [策略与调度](gu-zhang-pai-cha.md)

你可以为你的神经网络选择一个学习速率策略。策略会随着时间的推移而改变学习率，获得更好的结果，因为学习速率可以“减速”，以找到更接近的局部极小收敛。常用的策略是调度。有关实践中使用的学率调度，请参阅[LeNet](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/convolution/LenetMnistExample.java) 示例。

请注意，如果使用多个GPU，这将影响你的调度。例如，如果你有2个 GPU，那么你需要将你的调度中的迭代分成2份，因为你的训练过程的吞吐量将是两倍，并且学习速率调度只适用于本地GPU。

## [激活函数](gu-zhang-pai-cha.md)

关于激活函数的选择，有两个方面需要注意。

首先，隐藏（非输出）层的激活函数。一般来说，“Relu”或“leakyrelu”激活是很好的选择。一些其他的激活函数（tanh、sigmoid等）更容易出现梯度消失问题，这使得深度神经网络的学习更加困难。然而，对于LSTM层，TANH激活函数仍然是常用的。

第二，关于输出层的激活函数：这通常是特定于应用的。对于分类问题，通常需要使用softmax激活函数，结合负对数似然度/MCXENT.（多类交叉熵）。softmax激活函数为你提供了分类的概率分布（即，输出总和为1）。对于回归问题，结合均方误差（MSE）损失函数，“identity”激活函数通常是一个很好的选择。

## [损失函数](gu-zhang-pai-cha.md)

每个神经网络层的损失函数既可以用于预训练、学习更好的权重，也可以用于分类（在输出层）以获得一些结果。（在上面的例子中，分类发生在覆盖部分）。

你的网络的目的将决定你使用的损失函数。对于预训练，选择重建熵。对于分类，使用多类交叉熵。

## [正则化](gu-zhang-pai-cha.md)

正则化方法有助于避免训练过程中的过拟合。当网络很好地预测训练集，但是对网络从未见过的数据做出糟糕的预测时，就会发生过拟合。一种考虑过拟合的方法是网络记忆了训练数据（而不是学习其中的总体关系）。

正则化的常见类型包括：

* L1和L2正则化惩罚了大的网络权重，并且避免了权重变得太大。L2正则化的一些级别在实践中是常用的。然而，请注意，如果l1或l2正则化系数太高，它们可能对网络造成过度惩罚，并阻止其学习。L2正则化的通常值是1E-3至1E-6。
* [丢弃](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/dropout/Dropout.java)，是一种常用的正则化方法，可以非常有效。最常用的丢弃率为0.5。
* 丢弃连接（概念上类似于丢弃，但使用得不太频繁）
* 限制网络大小的总数（即，限制每个层的层数和大小）
* [早停](zao-ting.md)

使用L1/L2/dropout正则化，分别使用regularization（TRUE）和L1（x）、L2（y）、.dropout\(z\)。注意，在dropout\(z\)中的z是保持激活的概率。

## [小批量大小](gu-zhang-pai-cha.md)

minibatch指的是在计算梯度和参数更新时一次使用的实例的数量。在实践中（除了最小的数据集之外），将数据设置成多个小批量是标准的做法。

理想的小批量大小将有所不同。例如，一个大小为10的小批量对于GPU来说常常太小，但是可以在CPU上工作。大小为1的小批量将允许网络进行训练，但不会获得并行性的好处。32可以是尝试的合理起点，其中16-128\(有时小于或大于或取决于应用和网络类型\)范围内的小批量是常见的。

## [更新器和优化算法](gu-zhang-pai-cha.md)

在DL4J中，术语“更新器”指的是训练机制，如动量、RMSProp、adagrad等。与“vanilla”随机梯度下降相结合，使用这些方法中的一种可以导致更快的网络训练。可以使用.updater\(Updater\) 配置选项设置更新器。

给定梯度，优化算法是如何进行更新。最简单（也是最常用的）的方法是随机梯度下降（SGD），但是DL4J也为SGD提供了线性搜索、共轭梯度和LBFGS优化算法。与SGD相比，后者的算法更强大，但是由于线性搜索组件，每次参数更新的成本要高得多，并且在实践中没有使用太多。请注意，原则上可以将任何更新器与任何优化算法相结合。

在大多数情况下，一个好的默认选择是使用随机梯度下降优化算法结合动量/rmsprop/adagrad更新器之一，动量在实践中经常使用。注意，对于动量，更新器称为NESTEROVS，动量速率可以通过 .momentum\(double\) 选项设置。

## [梯度归一化](gu-zhang-pai-cha.md)

在训练神经网络时，有时应用梯度归一化是有帮助的，以避免梯度太大（所谓的梯度爆炸问题，在循环神经网络中常见）或太小。这可以用.gradientNormalization\(GradientNormalization\) 和.gradientNormalizationThreshould\(double\) 方法来实现。对于梯度归一化的例子参见，[GradientNormalization.java](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/GradientNormalization.java)。这个例子的测试代码在[这里](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-core/src/test/java/org/deeplearning4j/nn/updater/TestGradientNormalization.java)。

## [循环神经网络：通过时间截断的反向传播](gu-zhang-pai-cha.md)

在训练具有长时间序列的循环网络时，一般建议使用截断反向传播算法。在“标准”的反向传播（在DL4J中默认），每个参数更新的成本可以变为禁止的。有关详细信息，请参阅[本页](../mo-xing/xun-huan-shen-jing-wang-luo.md#tong-guo-shi-jian-jie-duan-de-fan-xiang-chuan-bo)。

## [可见／隐藏单元](gu-zhang-pai-cha.md)

当使用深度信念网络时，请密切关注。受限波尔滋曼机（用于特征提取的DBN的组件）是随机的，并且将从相对于指定的可见或隐藏单元的不同概率分布中采样。

有关所有不同概率分布的列表，请参阅Geoff Hinton的定论[《训练受限波尔滋曼机实用指南》](https://www.cs.toronto.edu/~hinton/absps/guideTR.pdf)。

## [受限波尔滋曼机](gu-zhang-pai-cha.md)

当为执行压缩的自编码器创建隐藏层时，给它们比输入数据更少的神经元。如果隐藏层节点太接近输入节点的数量，则有可能有重建恒等函数的风险。太多隐层神经元增加了噪声和过拟合的可能性。对于784的输入层，你可以选择500的初始隐藏层和250的第二隐藏层。没有隐藏层应该少于输入层节点的四分之一。输出层将仅仅是与标签的数量相等。

较大的数据集需要更多的隐藏层。脸谱网的Deep Face使用了九个隐藏层，我们只能假设它是一个巨大的语料库。许多较小的数据集可能只需要三或四个隐藏层，其精度降低到超过该深度。通常情况下，较大的数据集包含更多的变化，需要更多的特征/神经元来获得准确的结果。典型的机器学习，当然，有一个隐藏层，而那些浅网被称为感知器。

大型数据集需要预先训练受限波尔滋曼机多次。只有通过多个预训练，算法才能够正确地在数据集的上下文中学习权重特征。也就是说，你可以并行地运行数据，或者通过集群来加速预训练。

## [NaN,不是一个数字错误](gu-zhang-pai-cha.md)

问：为什么我的神经网络会抛出NaN值？

答：反向传播涉及非常小的梯度的乘法，因为在表示非常接近于零的实数值时精度有限，无法表示。这个问题的术语是算术下溢。如果你的神经网络正在抛出NaN值，那么解决方案是重新调整你的网络以避免非常小的梯度。这更可能是一个更深层次的神经网络问题。

你可以尝试使用double数据类型，但通常建议先重新调整网络。

遵循基本的调整技巧和监控结果是确保NAN不再出现的方法。


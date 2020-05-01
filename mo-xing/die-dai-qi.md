---
description: 用于加载到神经网络的数据迭代工具。
---

# 迭代器

## 什么是迭代器?

数据集迭代器允许将数据轻松加载到神经网络中，并帮助组织批处理、转换和掩码。包含在Eclipse DL4J中的迭代器有助于用户提供的数据，或者自动加载公共的基准数据集如MNIST和IRIS。

### 用法

对于大多数用例，初始化迭代器和传递一个引用到`MultiLayerNetwork`或`ComputationGraph 的fit`\(\)方法是开始训练任务所需的全部内容：

```java
MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();

// 传入一个 MNIST 数据集迭代器，自动获取数据 
DataSetIterator mnistTrain = new MnistDataSetIterator(batchSize, true, rngSeed);
net.fit(mnistTrain);
```

许多其他方法也接受迭代器来完成任务，例如评估：

```java
// 直接传递给神经网络
DataSetIterator mnistTest = new MnistDataSetIterator(batchSize, false, rngSeed);
net.eval(mnistTest);

//使用一个评估类
Evaluation eval = new Evaluation(10); //创建一个带有10个可能分类的评估对象
while(mnistTest.hasNext()){
    DataSet next = mnistTest.next();
    INDArray output = model.output(next.getFeatureMatrix()); //得到网络预测
    eval.eval(next.getLabels(), output); //检查对真实分类的预测
}
```


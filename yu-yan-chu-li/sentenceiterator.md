---
description: DL4J中用于语言处理的单词、文档和句子的迭代。
---

# SentenceIterator

Sentence Iterator （句子迭代器）用于 [Word2vec](word2vec.md) 和 [词袋](https://deeplearning4j.org/docs/latest/bagofwords-tf-idf.html) 。

它将一些文本以向量的形式输入到神经网络中，也涵盖了文本处理中的文档概念。

在自然语言处理中，文档或句子通常用来封装算法应该学习的上下文。

一些例子包括分析推文和成熟的新闻文章。句子迭代器的目的是把文本分成可处理的位。注意句子迭代器是输入不可知的。因此，一些文本（文档）可以来自文件系统、Twitter API或Hadoop。

根据如何处理输入，句子迭代器的输出将被传递给分词器，用于处理单个词，这些通常是单词，但也可以是ngram、skipgrams或其他单元。分词器是由一个[Tokenizer Factory](tokenization.md)（分词器工厂）根据每句话创建的。分词器工厂是被传递到文本处理向量化器中的。

一些典型的例子如下：

```java
SentenceIterator iter = new LineSentenceIterator(new File("your file"));
```

假设文件中的每一行都是一个句子。

还可以将字符串列表作为如下语句：

```java
Collection<String> sentences = ...;
SentenceIterator iter = new CollectionSentenceIterator(sentences);
```

这将假定每个字符串是一个句子（文档）。记住，这可能是一个推文或文章列表，两者都适用。

可以对文件进行如下迭代：

```java
SentenceIterator iter = new FileSentenceIterator(new File("your dir or file"));
```

这将逐行解析文件，并在每行返回单个句子。

对于任何复杂的情况，我们推荐一个实际的机器学习级管道，UimaSentenceIterator 。

UimaSentenceIterator够进行分词、词性标注和语义化等。UimaSentenceIterator迭代一组文件并可以分割句子。你可以根据传入的AnalysisEngine来定制它的行为。

AnalysisEngine是[UIMA](https://uima.apache.org/)文本处理管道概念。DL4J附带了所有这些常见任务的标准分析引擎，允许你自定义传入的文本以及如何定义语句。AnalysisEngines是[OpenNLP](https://opennlp.apache.org/)管道的线程安全版本。我们还包括基于[cleartk](https://cleartk.github.io/cleartk/)的用于处理常见任务的管道。

对于那些使用UIMA或者对UIMA感到好奇的人来说，在类型系统内，它使用cleartk类型系统用于分词、句子和其他注释。

下面是如何创建UimaSentenceItrator：

```java
SentenceIterator iter = UimaSentenceIterator.create("path/to/your/text/documents");
```

你也可以直接实例化：

```java
SentenceIterator iter = new UimaSentenceIterator(path,AnalysisEngineFactory.createEngine(AnalysisEngineFactory.createEngineDescription(TokenizerAnnotator.getDescription(), SentenceAnnotator.getDescription())));
```

对于熟悉Uima的人来说，这是广泛使用Uimafit创建分析引擎。还可以通过扩展SentenceIterator来创建自定义语句迭代器。


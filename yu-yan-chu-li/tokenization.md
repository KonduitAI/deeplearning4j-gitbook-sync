---
description: 在DL4J中把文本分解成单个单词进行语言处理。
---

# Tokenization

## 什么是分词?

分词是将文本分解成单个单词的过程。单词窗口也是由词组成。 Word2Vec还可以输出文本窗口，这些文本窗口包括用于输入神经网络中的训练示例，如本文所见。

## 示例

下面是一个用DL4J工具进行分词的例子：

```java
     //带有词形还原，词性标注，句子分割的分词
     TokenizerFactory tokenizerFactory = new UimaTokenizerFactory();
     Tokenizer tokenizer = tokenizerFactory.tokenize("mystring");

      //迭代
      while(tokenizer.hasMoreTokens()) {
             String token = tokenizer.nextToken();
      }

      //得到词的整个列表
      List<String> tokens = tokenizer.getTokens();
```

上面的代码段创建了一个能够词干提取的分词器。

在Word2Vec中，那是创建词汇表的推荐方法，因为它避免了各种词汇上的巧合，例如同一名词的单数和复数被计算为两个不同的单词。


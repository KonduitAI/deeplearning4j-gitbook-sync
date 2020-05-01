---
description: DL4J语言处理概述
---

# 概述

尽管没有设计成可以与诸如Stanford CoreNLP或NLTK之类的工具相提并论，但DL4J确实包括本文描述的一些核心文本处理工具。

DL4J的NLP依赖 [ClearTK](https://cleartk.github.io/cleartk/)，一个开源的机器学习和Apache[非结构化信息管理架构](https://uima.apache.org/)的自然语言处理框架，或UIMA。UIMA的使我们能够执行的语言识别，特定语言的分割，句子边界检测和实体检测（专有名词：个人、企业、地方和事物）。

## SentenceIterator 句子迭代器

处理自然语言有几个步骤。第一个是遍历你的语料库创建一个文件列表，它可以和个推文这么短，或和一篇报纸的文章这么长。这是由一个句子迭代器来执行的，它将是这样的：

```java
       // 在行的空白处分割来得到单词        
      TokenizerFactory t = new DefaultTokenizerFactory();       
      /*            
      CommonPreprocessor 将以下正则表达式应用于每个词: [\d\.:,"'\(\)\[\]|/?!;]+            所以，有效地删除所有的数字，标点符号和一些特殊符号。另外它强制 
      把所有词转小写。        
      */
```

SentenceIterator封装一个语料库或文本，组织它，比如说，每行一条推文。它负责将文本逐条输入到你的自然语言处理器中。SentenceIterator与类似命名的类不同，如DatasetIterator创建用于训练神经网络的数据集。相反，它通过分割语料库来创建字符串集合。

## 分词器

分词器进一步分割文本为单个词，或是作为n-grams。ClearTK包括了底层的分词器，例如词性和解析树，允许依赖和选区解析，就像递归神经张量网络（RNTN）所使用的一样。

分词器由[TokenizerFactory](https://github.com/deeplearning4j/deeplearning4j/blob/6f027fd5075e3e76a38123ae5e28c00c17db4361/deeplearning4j-scaleout/deeplearning4j-nlp/src/main/java/org/deeplearning4j/text/tokenization/tokenizerfactory/UimaTokenizerFactory.java) 创建和包装。默认的词元是被空格分割的单词。分词进程还引入了一些机器学习来区分模棱两可的符号。哪些句子和缩写词如Mr. 和 vs。

Tokenizers\(分词器\)和SentenceIterator\(句子迭代器\)都与Preprocessors\(预处理器\)一起处理像Unicode这样的混乱文本中的异常，并统一地呈现这些文本，比如小写字符。

```java
        log.info("Building model....");      
        Word2Vec vec = new Word2Vec.Builder()
                          .minWordFrequency(5)
                          .iterations(1)
                          .layerSize(100)
                          .seed(42)
                          .windowSize(5)
                          .iterate(iter)
                          .tokenizerFactory(t) 
                          .build();
        log.info("Fitting Word2Vec model....");
        vec.fit();
```

## Vocab词汇

每一个文档都必须分割为词汇，用于文档或语料库的单词集合。这些单词存在词汇缓存中，它包含关于文档中计数的单词子集的统计信息。区分重要和不重要单词的线条是移动的，但是区分两组的基本思想是单词只出现一次（或少于五次）是很难学习的，并且它们的存在代表了无用的噪音。

词汇缓存了例如Word2vec和词袋方法的元数据，以极端不同的方式对待单词。Word2vec以数百个系数长的向量形式创建单词或神经单词嵌入的表示。这些系数帮助神经网络预测一个词在任何给定上下文中出现的可能性；例如，在另一个单词之后。这里是Word2vec，配置如下：

```java
package org.deeplearning4j.examples.nlp.word2vec;
import org.datavec.api.util.ClassPathResource;
import org.deeplearning4j.models.word2vec.Word2Vec;
import org.deeplearning4j.text.sentenceiterator.BasicLineIterator;
import org.deeplearning4j.text.sentenceiterator.SentenceIterator;
import org.deeplearning4j.text.tokenization.tokenizer.preprocessor.CommonPreprocesor;
import org.deeplearning4j.text.tokenization.tokenizerfactory.DefaultTokenizerFactory;
import org.deeplearning4j.text.tokenization.tokenizerfactory.TokenizerFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.Collection;
/**  
 * Created by agibsonccc on 10/9/14.
 * 将文本处理成词向量的神经网络，查看下面的URL获取深入解释。 
 * https://deeplearning4j.org/word2vec.html 
*/
public class Word2VecRawTextExample {    

private static Logger log = 
 LoggerFactory.getLogger(Word2VecRawTextExample.class);   

 public static void main(String[] args) throws Exception {       
 // 得到文本文件的路径        
String filePath = new ClassPathResource("raw_sentences.txt").getFile().getAbsolutePath();        

log.info("加载 & 向量化句子....");        

  // 每行前后空白间隔       
  SentenceIterator iter = new BasicLineIterator(filePath);       
  // 每行用空格分割以获取单词        
  TokenizerFactory t = new DefaultTokenizerFactory();      
  /*            CommonPreprocessor 将应用如下正则表达式到每个词: [\d\.:,"'\(\)\[\]|/?!;]+            
所以有效的删除所有数字，标点符号，和特符号，并把所有词转换为小写。         */      
 t.setTokenPreProcessor(new CommonPreprocessor());      
 log.info("构建模型....");       
 Word2Vec vec = new Word2Vec.Builder()                
                  .minWordFrequency(5)
                  .iterations(1)
                  .layerSize(100) 
                  .seed(42)
                  .windowSize(5)
                  .iterate(iter) 
                  .tokenizerFactory(t)
                  .build(); 
log.info("拟合 Word2Vec 模型....");
vec.fit(); 

log.info("将词向量写入文本文件....");
// 打印出最接近“day”的10个词。一个如何处理这些单词向量的例子。        

log.info("最接近的词:"); 

Collection<String> lst = vec.wordsNearestSum("day", 10);
log.info("10 个最接近 'day': {}", lst); 

 // TODO 解决丢失的 UiServer
 //        UiServer server = UiServer.getInstance();
 //        System.out.println("Started on port " + server.getPort());    }}
```

一旦你获得单词向量，你就可以把它们输入深度网络进行分类、预测、情感分析等。


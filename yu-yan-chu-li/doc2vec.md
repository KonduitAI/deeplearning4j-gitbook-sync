---
description: 在DL4J中用于语言处理的Doc2Vec和任意文档。
---

# Doc2Vec

Doc2Vec的主要目的是将任意文档与标签关联，因此需要标签。Doc2Vec是Word2Vec的一个扩展，它学习关联标签和单词，而不是用单词关联单词。DL4J实现它的意图是为了服务于Java、Scala和Culjule社区。

第一步是提出一个表示文档“含义”的向量，然后可以将其用作有监督的机器学习算法的输入，来把文档与标签相关联。

在ParagraphVectors构建器模式中，`labels()` 方法指向用于训练的标签。在下面的示例中，你可以看到与情感分析相关的标签：

```java
    .labels(Arrays.asList("negative", "neutral","positive"))
```

下面是段落向量分类的可运行的完整[示例](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/paragraphvectors/ParagraphVectorsClassifierExample.java)：

```java
    public void testDifferentLabels() throws Exception {
        ClassPathResource resource = new ClassPathResource("/labeled");
        File file = resource.getFile();
        LabelAwareSentenceIterator iter = LabelAwareUimaSentenceIterator.createWithPath(file.getAbsolutePath());

        TokenizerFactory t = new UimaTokenizerFactory();

        ParagraphVectors vec = new ParagraphVectors.Builder()
                .minWordFrequency(1).labels(Arrays.asList("negative", "neutral","positive"))
                .layerSize(100)
                .stopWords(new ArrayList<String>())
                .windowSize(5).iterate(iter).tokenizerFactory(t).build();

        vec.fit();

        assertNotEquals(vec.lookupTable().vector("UNK"), vec.lookupTable().vector("negative"));
        assertNotEquals(vec.lookupTable().vector("UNK"),vec.lookupTable().vector("positive"));
        assertNotEquals(vec.lookupTable().vector("UNK"),vec.lookupTable().vector("neutral"));}
```

## 延伸阅读

* [句子和文档的分布式表示](https://cs.stanford.edu/~quocle/paragraph_vector.pdf)
* [Word2vec](word2vec.md): 教程


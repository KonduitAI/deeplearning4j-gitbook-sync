---
description: DL4J中NLP神经词嵌入
---

# Word2Vec

### Word2Vec, Doc2vec & GloVe: 用于自然语言处理的神经词嵌入

内容

* [介绍](word2vec.md#word-2-vec-jie-shao)
* [神经词嵌入](word2vec.md#shen-jing-ci-qian-ru)
* [有趣的Word2Vec结果   ](word2vec.md#you-qu-de-word-2-vec-jie-guo)
* [给我代码](word2vec.md#gei-wo-dai-ma)
* [Word2Vec 剖析](word2vec.md#dl-4-j-zhong-word-2-vec-de-pou-xi)
* [安装，加载与训练](word2vec.md#word-2-vec-she-zhi)
* [代码示例](word2vec.md#gong-zuo-shi-li)
* [故障排查与Word2Vec调试](word2vec.md#word-2-vec-gu-zhang-pai-chu-yu-tiao-zheng)
* [Word2Vec用例](word2vec.md#google-de-word-2-vec-zhuan-li)
* [外语](word2vec.md#wai-yu)
* [GloVe\(全局向量\)与Doc2Vec](word2vec.md#glove-quan-ju-xiang-liang)

### Word2Vec介绍

Word2Vec是一个处理文本的两层神经网络。它的输入是一个文本语料库，它的输出是一组向量：语料库中的单词的特征向量。Word2Vec不是一个[深度神经网络](https://skymind.ai/wiki/neural-network)，它将文本转换成一个深度网络可以理解的数值形式。[DL4J](https://deeplearning4j.org/docs/latest/deeplearning4j-quickstart)实现了一个分布式的Word2Vec，用于Java和Scala，它在Spark的GPU上工作。

Word2Vec的应用扩展了自然界的句子解析。它也可以同样地应用于[基因、代码、喜欢、播放列表、社交媒体图表](https://deeplearning4j.org/docs/latest/deeplearning4j-nlp-word2vec#sequence)和其他可以识别模式的语言或符号系列。

为什么？因为单词只是像上面提到的其他数据一样的离散状态，我们只是在寻找这些状态之间的转移概率：它们将同时发生的可能性。所以gene2vec，like2vec和follower2vec 都是可能的。记住这一点，下面的教程将帮助你理解如何为任意一组离散和共现状态创建神经嵌入。

Word2Vec的目的和实用性是将相似词的向量分组到向量空间中。也就是说，它在数学上检测相似性。Word2Vec创建向量，这些向量是单词特征（例如单个单词的上下文）的分布式数字表示。这样做没有人为干预。

给定足够的数据、用法和上下文，Word2Vec可以基于过去的出现对单词的意义做出高度准确的猜测。这些猜测可以用来建立一个单词与其他单词的关联（例如，“男人”是“男孩”，“女人”是“女孩”），或者是聚类文档，并按主题分类。这些聚类可以构成搜索的基础、[情感分析](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/word2vecsentiment/Word2VecSentimentRNN.java)和在科学研究、法律发现、电子商务和客户关系管理等多个领域的建议。

Word2Vec神经网络的输出是一个词汇表，其中每个项目都有一个附加到它的向量，它可以被送入深度学习网络或简单地查询以检测词之间的关系。

测量[余弦相似度](https://skymind.ai/wiki/glossary#cosine)，90度角表示没有相似度，而总的相似度是1是0度角，完全重叠；即Sweden等于Sweden，而Norway到Sweden的余弦距离是0.760124，是任何其他国家中最高的。

这是一个使用Word2Vec生成的与“Sweden”相关的单词列表，按接近顺序排序:

![Cosine Distance](http://upload-images.jianshu.io/upload_images/14495907-6f9fb3d82ba62b36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

斯堪的纳维亚的国家和几个富裕的北欧、日耳曼国家跻身前九位。

### 神经词嵌入

我们用来表示单词的向量称为神经词嵌入，表示是奇怪的。一件事描述了另一件事，尽管这两件事是根本不同的。正如Elvis Costello所说：“写作对于音乐就像跳舞对于建筑。”Word2Vec对单词“向量化”，通过这样做，它使得自然语言可以被计算机阅读——我们可以开始对单词执行强大的数学运算以检测它们的相似性。

因此，神经词嵌入用数字代表一个单词。这是一个简单但不太可能的翻译。

Word2Vec类似于一个自动编码器，将每个单词编码在一个向量中，而不是通过[重建](https://skymind.ai/wiki/variational-autoencoder)对输入单词进行训练，Word2Vec在语料库中将单词和与它们相邻的其他单词进行训练。

它以两种方式中的其中一种来实现，或者使用上下文来预测目标单词（一种称为连续词袋或CBOW的方法），或者使用单词来预测目标上下文，即skip-gram。我们使用后一种方法，因为它对大数据集产生更精确的结果。

![word2vec diagram](http://upload-images.jianshu.io/upload_images/14495907-5e54b555e667b7dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当分配给单词的特征向量不能用于精确预测该单词的上下文时，向量的组成部分会被调整。语料库中的每个单词的上下文是老师，往回发送错误信号以调整特征向量。通过调整在向量中数值凑在一起的上下文，单词的向量被它们判断为相似的。

正如梵高的向日葵画是油画布上的二维混合物，代表了1880年代末巴黎三维空间中的植物物质，所以以向量排列的500个数字可以代表一个词或一组词。

这些数字将每个单词定位为500维向量空间中的一个点。超过三个维度的空间难以可视化。（Geoff Hinton教授人们想象13维空间，建议学生首先想象3维空间，然后对自己说：“13、13、13”：）

一组训练有素的单词向量将在那个空间中放置相似的单词。“橡树”、“榆树”和“桦树”可能会聚集在一个角落，而战争、冲突和争斗则聚集在另一个角落。

类似的事情和想法被证明是“接近的”。它们的相对意义已经转化为可测量的距离。质量变成数量，算法可以完成他们的工作。但相似性只是Word2Vec可以学习的许多关联的基础。例如，它可以衡量一种语言的单词之间的关系，并将它们映射到另一种语言。

![word2vec translation](http://upload-images.jianshu.io/upload_images/14495907-e60c054e70ef350f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这些向量是更全面的词汇几何的基础。如图所示，像罗马、巴黎、柏林和北京这样的首都城市相互靠近，在向量空间上它们各自具有与其国家相似的距离，即罗马-意大利=北京-中国。如果你只知道罗马是意大利的首都，并想知道中国的首都，那么等式罗马-意大利+中国将返回北京。这不是玩笑。

![capitals output](http://upload-images.jianshu.io/upload_images/14495907-f8d5ffa0283e0499.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 有趣的Word2Vec结果

让我们看看Word2Vec可以产生的其他关联。

我们将用逻辑类比的符号代替加减等号，给出结果，其中`:`是 “对于”的意思和`::`“等同”的意思，例如“罗马对意大利就像北京对中国一样”=罗马`:`意大利`::`北京`:`中国。在最后一点，当给出前三个元素时，我们将给出Word2vec模型建议的单词列表，而不是提供“答案”：

```java
king:queen::man:[woman, Attempted abduction, teenager, girl] 
//很怪异，但你可以看到

China:Taiwan::Russia:[Ukraine, Moscow, Moldova, Armenia]
//两个大国和他们小的远离的邻居

house:roof::castle:[dome, bell_tower, spire, crenellations, turrets]

knee:leg::elbow:[forearm, arm, ulna_bone]

New York Times:Sulzberger::Fox:[Murdoch, Chernin, Bancroft, Ailes]
//Sulzberger-Ochs家族拥有并经营NYT。
//Murdoch 家族拥有新闻公司，此家族有福克斯新闻。 
//Peter Chernin是新闻公司的13年的首席运营官。
//Roger Ailes是福克斯新闻的主席。 
//Bancroft家族把《华尔街日报》卖给了新闻集团。

love:indifference::fear:[apathy, callousness, timidity, helplessness, inaction]
//这首诗的诗集简直令人惊叹。

Donald Trump:Republican::Barack Obama:[Democratic, GOP, Democrats, McCain]
//有趣的是，正如奥巴马和麦凯恩是对手一样
//同样，Word2Vec认为特朗普与共和党的观点有对立。

monkey:human::dinosaur:[fossil, fossilized, Ice_Age_mammals, fossilization]
//人类是化石猴子？人类就是剩下的
//猴子？人类是打败猴子的物种。
//就像冰河时代哺乳动物打败恐龙一样？貌似有理的。

building:architect::software:[programmer, SecurityCenter, WinPcap]
```

这个模型是在谷歌新闻vocab上进行训练的，你可以导入并玩一玩。考虑片刻，Word2Vec算法从来没有被教过一条英语语法规则。它对世界一无所知，与任何基于规则的符号逻辑或知识图无关。然而，比在多年的人力学习后大的大多数知识图的学习，它以更灵活和自动化的方式学习。它把Google新闻的文档看作一张白板，训练结束后，它可以计算对人类有意义的复杂类推。

你还可以查询Word2Vec模型进行其他关联。并不是每件事都必须有两个相互镜像的类推。（我们解释如下……）

* 地缘政治学：伊拉克-暴力=约旦
* 区分：人类-动物=伦理
* _总统-权力=总理_
* _图书馆-图书=大厅_
* 类推：股票市场≈温度计

通过构建一个单词与其他类似单词的邻近场景，这些单词不一定包含相同的字母，我们已经从硬标记，进入了更平滑和更普遍的意义的场景 。

## 给我代码

### DL4J中Word2Vec的剖析

这些是DL4J自然语言处理的组件：

* **SentenceIterator/DocumentIterator**: 用于迭代一个数据集。 SentenceIterator 返回一个字符串 ， DocumentIterator 与输入流一起工作。
* **Tokenizer/TokenizerFactory**: 用于对文本进行分词。 在NLP术语中，句子被表示为一系列词。TokenizerFactory为一个句子创建一个分词器的实例。
* **VocabCache**: 用于跟踪元数据，包括单词计数、文档出现、词集（本例中不是vocab，而是已经发生的令牌词）、vocab（词袋和单词向量查找表中包括的特性） 
* **Inverted Index**: 存储有关单词发生的元数据。可以用于理解数据集。自动创建具有Lucene实现（1）的Lucene索引。

Word2vec是指一系列相关算法，该实现采用[负采样](https://skymind.ai/wiki/glossary#skipgram)。

### Word2Vec 设置

使用Maven在IntelliJ中创建一个新项目。如果你不知道怎么做，请看我们的快速入门页面。然后在项目的根目录的POM.xml文件中指定这些属性和依赖项（你可以检查Maven以获得最新版本，请使用这些版本…）。

#### 加载数据

现在在Java中创建并命名一个新类。之后，你将在.txt文件中获取原始语句，用迭代器遍历它们，并使它们接受某种预处理，例如将所有单词转换为小写。

```java
        String filePath = new ClassPathResource("raw_sentences.txt").getFile().getAbsolutePath();

        log.info("加载并向量化句子....");
        //每一行之间用空格分割
        SentenceIterator iter = new BasicLineIterator(filePath);
```

如果你想加载一个文本文件，用我们的例子中提供的句子之外的句子，你这样做：

```java
        log.info("Load data....");
        SentenceIterator iter = new LineSentenceIterator(new File("/Users/cvn/Desktop/file.txt"));
        iter.setPreProcessor(new SentencePreProcessor() {
            @Override
            public String preProcess(String sentence) {
                return sentence.toLowerCase();
            }
        });
```

也就是说，去掉`ClassPathResource`，并将你的.txt文件的绝对路径填入到`LineSentenceIterator`中。

```java
SentenceIterator iter = new LineSentenceIterator(new File("/your/absolute/file/path/here.txt"));
```

在bash中，通过在命令行中从同一目录中键入pwd，可以找到任何目录的绝对文件路径。对于该路径，你将添加文件名。

#### 数据分词

Word2Vec需要用词而不是完整的句子，所以下一步就是把数据分词。把文本分词是把它分解成原子单位，例如，每次你点击一个空白处时，创建一个新的分词。

```java
        //在每行用用空格分割以得到单词
        TokenizerFactory t = new DefaultTokenizerFactory();
        t.setTokenPreProcessor(new CommonPreprocessor());
```

那样它会给你每行一个词。

#### 训练模型

现在数据已准备就绪，你可以配置Word2Vec神经网络并输入分词。

```java
        log.info("Building model....");
        Word2Vec vec = new Word2Vec.Builder()
                .minWordFrequency(5)
                .layerSize(100)
                .seed(42)
                .windowSize(5)
                .iterate(iter)
                .tokenizerFactory(t)
                .build();

        log.info("Fitting Word2Vec model....");
        vec.fit();
```

此配置接受许多超参数。 一些需要一些解释：

* _batchSize_ 是你一次处理的单词数量。
* _minWordFrequency_ 是单词必须出现在语料库中的最小次数。 在这里，如果它出现少于5次，则不会学习。 单词必须出现在多个上下文中才能学习有关它们的有用特征。 在非常大的语料库中，提高最小值是合理的。
* _useAdaGrad_ - Adagrad为每个特征创建不同的梯度。 在这里，我们并不关心这一点。
* _layerSize_ 指定单词向量中的特征数。这等于特征空间中的维数。由500个特征表示的词成为500维空间中的点。
* _learningRate_ 是每个更新系数的步长，因为单词在特征空间中被重新定位。
* _minLearningRate_ 是学习率的底板。学习速率随着你训练的单词数量的减少而衰减。如果学习率下降太多，网络的学习就不再有效了。这保持系数移动。
* _iterate_ 告诉网络它正在训练的数据集的批次。
* _tokenizer_ 从当前批次中为它提供单词。
* _vec.fit\(\)_ 告诉配置的网络开始训练。

[这里](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/word2vec/Word2VecUptrainingExample.java)是训练你以前训练过的单词向量的示例。

#### 使用Word2Vec评估模型

下一步是评估特征向量的质量。

```java
        // 写入词向量
        WordVectorSerializer.writeWordVectors(vec, "pathToWriteto.txt");

        log.info("最接近的10个词:");
        Collection<String> lst = vec.wordsNearest("day", 10);
        System.out.println(lst);
        UiServer server = UiServer.getInstance();
        System.out.println("启动端口：" + server.getPort());

        //输出: [night, week, year, game, season, during, office, until, -]
```

`vec.similarity("word1","word2")这行`将返回输入的两个词的余弦相似度。越接近1，网络就理解为越类似于那些词（参见上面的瑞典-挪威例子）。例如：

```java
        double cosSim = vec.similarity("day", "night");
        System.out.println(cosSim);
        //输出: 0.7704452276229858
```

使用`vec.wordsNearest("word1", numWordsNearest)`，打印到屏幕上的单词允许你查看网络是否聚集了语义上相似的单词。你可以用wordsNearest方法的第二个参数来设置你想要的最近单词的数量。例如：

```java
        Collection<String> lst3 = vec.wordsNearest("man", 10);
        System.out.println(lst3);
        //输出: [director, company, program, former, university, family, group, such, general]
```

#### 模型可视化

我们依赖于[TSNE](https://lvdmaaten.github.io/tsne/)来把单词特征向量和项目词的维数减少到两个或三维空间。TSNE的完整的DL4J/ND4J例子在[这里](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/tsne/TSNEStandardExample.java)。

```java
        Nd4j.setDataType(DataBuffer.Type.DOUBLE);
        List<String> cacheList = new ArrayList<>(); //cacheList 是一种动态字符串数组，用于保存所有单词。

        //步骤2：将文本输入转换成单词列表
        log.info("加载并向量化数据....");
        File wordFile = new ClassPathResource("words.txt").getFile();   //打开文件
        //获取所有唯一词向量的数据
        Pair<InMemoryLookupTable,VocabCache> vectors = WordVectorSerializer.loadTxt(wordFile);
        VocabCache cache = vectors.getSecond();
        INDArray weights = vectors.getFirst().getSyn0();    //将独特词的权重分成自己的列表

        for(int i = 0; i < cache.numWords(); i++)   //把字串分隔成自己的列表
            cacheList.add(cache.wordAtIndex(i));

        //步骤3：构建双树TSNE以供以后使用
        log.info("Build model....");
        BarnesHutTsne tsne = new BarnesHutTsne.Builder()
                .setMaxIter(iterations).theta(0.5)
                .normalize(false)
                .learningRate(500)
                .useAdaGrad(false)
//                .usePca(false)
                .build();

        //步骤4：建立TSNE值并将其保存到文件中
        log.info("存储TSNE坐标用于绘制....");
        String outputFile = "target/archive-tmp/tsne-standard-coords.csv";
        (new File(outputFile)).getParentFile().mkdirs();

        tsne.fit(weights);
        tsne.saveAsFile(cacheList, outputFile);
```

#### 保存，重新加载并使用模型

你会想保存这个模型。在DL4J中保存模型的常规方法是通过序列化工具（Java序列化类似于Python的pickling，将一个对象转换成一系列字节）。

```java
        log.info("保存向量....");
        WordVectorSerializer.writeWord2VecModel(vec, "pathToSaveModel.txt");
```

这将将向量保存到一个名为`pathToSaveModel.txt`的文件中，该文件将出现在Word2Vec被训练的目录的根目录中。文件中的输出每行应该有一个单词，后面是一系列数字，它们一起表示它的向量。

为了继续使用向量，简单地像这样调用关于vec的方法：

```java
Collection<String> kingList = vec.wordsNearest(Arrays.asList("king", "woman"), Arrays.asList("queen"), 10);
```

Word2Vec的词算术的经典例子是 国王-皇后=男人-女人，它的逻辑扩展是 国王-皇后+女人=男人。

上面的例子将把10个最近的单词输出到向量 国王-皇后+女人 ，这应该包括“男人”。wordsNearest的第一个参数必须包括“正”单词国王 和女人，它们具有与之关联的+符号；第二个参数包括“负”单词皇后，它与负符号关联（这里正和负没有情感内涵）；第三是你想看的最接近单词列表的长度。请记住将此添加到文件的顶部：`import java.util.Arrays;`

任何数量的组合都是可能的，但只有在语料库中出现足够频繁的查询词时，它们才会返回合理的结果。显然，返回相似词（或文档）的能力是搜索引擎和推荐引擎的基础。

你可以像这样把向量重新加载到内存中：

```java
        Word2Vec word2Vec = WordVectorSerializer.readWord2VecModel("pathToSaveModel.txt");
```

然后，您可以使用Word2Vec作为查找表：

```java
        WeightLookupTable weightLookupTable = word2Vec.lookupTable();
        Iterator<INDArray> vectors = weightLookupTable.vectors();
        INDArray wordVectorMatrix = word2Vec.getWordVectorMatrix("myword");
        double[] wordVector = word2Vec.getWordVector("myword");
```

如果单词不在词汇表中，Word2Vec返回零。

#### 导入Word2Vec模型

在S3托管的[谷歌新闻语料库模型](https://deeplearning4jblob.blob.core.windows.net/resources/wordvectors/GoogleNews-vectors-negative300.bin.gz)，我们用来测试我们的训练网的准确性。对于那些在大型语料库上训练当前硬件需要很长时间的用户，可以简单地下载它来探索Word2Vec模型，而不需要前奏。

如果你用[C vectors](https://docs.google.com/file/d/0B7XkCwpI5KDYaDBDQm1tZGNDRHc/edit)或Gensimm进行训练，此行将导入模型。

```java
    File gModel = new File("/Developer/Vector Models/GoogleNews-vectors-negative300.bin.gz");
    Word2Vec vec = WordVectorSerializer.readWord2VecModel(gModel);
```

记得添加 `import java.io.File;`到你引入的包。

对于大型模型，你可能会遇到堆空间的问题。Google模型可能需要多达10G的RAM，而JVM只使用256MB的RAM启动，因此必须调整堆空间。你可以用一个`bash_profile`文件（参见我们的[故障排查部分](https://deeplearning4j.org/docs/latest/deeplearning4j-troubleshooting-training)），或者通过IntelliJ本身来做：

```java
    //Click:
    IntelliJ Preferences > Compiler > Command Line Options 
    //Then paste:
    -Xms1024m
    -Xmx10g
    -XX:MaxPermSize=2g
```

#### N-grams & Skip-grams

单词被一次读入到向量，并在一定范围内来回扫描。这些范围是n-gram，一个 n-gram是给定语言序列中n个项目的连续序列；它是unigram、bigram、trigram、4-gram或5-gram的第n个版本。skip-gram简单地从N-gram中删除项目。

Mikolov推广并在DL4J实现中使用的skip-gram被证明比其他模型（如连续词袋）更精确，这是因为生成的上下文更具有通用性。

然后将该n-gram输入到神经网络以学习给定词向量的重要性；即，重要性被定义为其实用性，作为作为某些更大含义或标签的指示器。

#### 工作实例

请注意：下面的代码可能过时了。有关更新的示例，请参阅Github上的我们的[DL4J示例库](https://github.com/deeplearning4j/dl4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp)。

既然你已经有了一个关于如何建立Word2Vec的基本思想，这里有一个[例子](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/word2vec/Word2VecRawTextExample.java)，它是如何与DL4J的API一起使用的：

在按照快速入门的说明后，你可以在IntelliJ中打开这个示例并点击Run运行它。如果你在Word2Vec模型中查询一个不包含在训练语料库的单词，它将返回NULL。

#### Word2Vec故障排除与调整

_问：我有很多这样的堆栈跟踪_

```java
       java.lang.StackOverflowError: null
       at java.lang.ref.Reference.<init>(Reference.java:254) ~[na:1.8.0_11]
       at java.lang.ref.WeakReference.<init>(WeakReference.java:69) ~[na:1.8.0_11]
       at java.io.ObjectStreamClass$WeakClassKey.<init>(ObjectStreamClass.java:2306) [na:1.8.0_11]
       at java.io.ObjectStreamClass.lookup(ObjectStreamClass.java:322) ~[na:1.8.0_11]
       at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1134) ~[na:1.8.0_11]
       at java.io.ObjectOutputStream.defaultWriteFields(ObjectOutputStream.java:1548) ~[na:1.8.0_11]
```

答:看看你启动Word2Vec应用程序的目录里面。例如，这可以是一个IntelliJ项目主目录或在命令行键入Java的目录。它应该有一些目录看起来像：

```text
       ehcache_auto_created2810726831714447871diskstore  
       ehcache_auto_created4727787669919058795diskstore
       ehcache_auto_created3883187579728988119diskstore  
       ehcache_auto_created9101229611634051478diskstore
```

你可以关闭你的Word2Vec应用程序并尝试删除这些目录。

问：不是所有来自我原始文本数据的单词都出现在我的Word2Vec对象中…

答: 试着在你的Word2Vec对象上通过**.layerSize\(\)** 来提高图层大小，像这样

```java
        Word2Vec vec = new Word2Vec.Builder().layerSize(300).windowSize(5)
                .layerSize(300).iterate(iter).tokenizerFactory(t).build();
```

问：如何加载我的数据？为什么训练会永远持续下去？

答：如果你所有的句子都被作为一个句子被加载，Word2Vec训练可能需要很长的时间。这是因为Word2Vec是一个句子级别的算法，所以句子边界非常重要，因为共现统计是逐句收集的。（对于GloVe来说，句子边界并不重要，因为它关注于语料库范围的共现。对于许多语料库，平均句子长度为六个单词。这意味着在窗口大小为5的情况下，有30个（随机数）回合的skip-gram计算。如果你忘记指定句子的边界，你可能加载一个“10000个单词”长的句子。在这种情况下，Word2Vec将为整个10000个单词“句子”尝试全skip-gram循环。在DL4J的实现中，假定一行是一个句子。你需要插入你自己的句子迭代器和分词器。通过要求你指定你的句子如何结束，DL4J仍然是语言不可知论者。UimaSentenceIterator是这样做的一种方式。使用OpenNLP进行句子边界检测。

问：为什么把整个文档作为一个“句子”而不是分割成句子时，在性能上有如此不同？

答：如果平均句子包含6个单词，窗口大小为5，那么理论上最多10个skipgram回合的次数是0字。句子不够长，不能用文字表达完整的窗口。在这句话中所有单词的粗略最大数目为5个skipgram回合。但如果你的“句子”有1000k个单词的长度，这个句子中的每个单词就有10个skipgram回合，不包括前5个和最后5个。因此，你将不得不花费大量时间来构建模型+由于缺少句子边界，协同统计将会发生变化。

问：Word2Vec是如何使用内存的？

答：Word2Vec中的主要内存消耗是权重矩阵。数学是简单的：单词数x维度数x 2 x数据类型 内存占用。因此，如果使用浮点数和100维来构建100k字的Word2Vec模型，那么内存占用将是100kx100x2x4（浮点数大小）=80MB RAM，仅用于矩阵+用于字符串、变量、线程等的一些空间。如果加载预构建的模型，则在构建时间中使用大约1/2的RAM，因此它是40MB RAM。目前使用的最流行的模型是谷歌新闻模型。有3百万字，向量大小为300。这就使我们需要3.6G RAM仅加载模型。而且必须添加3M的字符串，这些字符串在Java中没有固定的大小。所以，通常是大约4-6GB用于加载模型，这取决于JVM版本/供应商，GC状态和月球的相位。

问：我做了你说的每一件事，结果还是不对头。

答：确保你正遇到不是正常性问题。一些任务，如wordsNearest\(\)，默认使用标准化的权重，而其他的则需要非标准化的权重。注意这个区别。

#### 用例

谷歌学者保存了论文记录，这里引用了[Word2Vec的DL4J实现](https://scholar.google.com/scholar?hl=en&q=deeplearning4j+word2vec&btnG=&as_sdt=1%2C5&as_sdtp=)。

来自比利时的数据科学家Kenny Helsens将Word2Vec的DL4J实现应用于NCBI的在线孟德尔人类继承\(OMIM\)数据库。然后，他寻找与alk（一种已知的非小细胞肺癌的致癌基因）最相似的单词，Word2vec返回：“nonsmall, carcinomas, carcinoma, mapdkd”。从那里，他建立了其他癌症表型和基因型之间的类比。这只是Word2Vec在大型语料库上可以学习的一个例子。发现重要疾病新方面的潜力才刚刚开始，在医学之外，机会也同样多样。

Andreas Klintberg在瑞典训练了Word2Vec的DL4J实现，并在媒体上写下了一个[完整的指导](https://medium.com/@klintcho/training-a-word2vec-model-for-swedish-e14b15be6cb)。

Word2Vec在信息检索准备基于文本的数据和问答系统中特别有用，DL4J通过深度自动编码器来实现这些系统。

营销人员可能寻求建立产品间的关系来建立推荐引擎。调查者可能会分析一个社会图表，以显示单个群体的成员，或者他们可能必须定位或资助的其他关系。

#### Google的 Word2vec 专利

Word2Vec是由Tomas Mikolov领导的谷歌研究团队介绍的一种[计算单词向量表示的方法](https://arxiv.org/pdf/1301.3781.pdf)。谷歌托管了一个[开源版本的Word2Vec](https://code.google.com/p/word2vec/)，它是在Apache 2许可下发布的。在2014，Mikolov离开谷歌去了Facebook，并在2015年5月，[谷歌被授予获得此专利](http://patft.uspto.gov/netacgi/nph-Parser?Sect1=PTO2&Sect2=HITOFF&p=1&u=%2Fnetahtml%2FPTO%2Fsearch-bool.html&r=1&f=G&l=50&co1=AND&d=PTXT&s1=9037464&OS=9037464&RS=9037464)，已发布的版本没有废除Apache许可证。

#### 外语

虽然所有语言中的单词都可以用Word2Vec转换为向量，并且这些向量通过DL4J学习，但是NLP预处理可以非常特定于语言，并且需要超出我们库的工具。[斯坦福自然语言处理小组有许多基于Java的工具](http://nlp.stanford.edu/software/)，用于语言的分词、词性标注和命名实体识别，例如[普通话](http://nlp.stanford.edu/projects/chinese-nlp.shtml)、阿拉伯语、法语、德语和西班牙语。对于日本人来说，像[Kuromoji](http://www.atilika.org/)之类的NLP工具是有用的。其他的外语资源，包括文本语料库，都在[这里](http://www-nlp.stanford.edu/links/statnlp.html)。

#### GloVe: 全局向量

加载和保存GloVe模型到Word2Vec可以这样做：

```java
        WordVectors wordVectors = WordVectorSerializer.loadTxtVectors(new File("glove.6B.50d.txt"));
```

#### 序列向量

DL4J具有一个名为[SequenceVectors](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nlp-parent/deeplearning4j-nlp/src/main/java/org/deeplearning4j/models/sequencevectors/SequenceVectors.java)的类，它是单词向量之上的抽象级别，并且允许你从任何序列中提取特征，包括社交媒体概要、事务、蛋白质等。 如果数据可以被描述为序列，它可以通过skip-gram和层次化的softmax与AbstractVectors类来学习。这与[深度算法](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-graph/src/main/java/org/deeplearning4j/graph/models/deepwalk/DeepWalk.java)相兼容，也在DL4J中实现。

#### DL4L的Word2Vec特征

* 模型序列化/反序列化 被添加后的权重会更新。也就是说，你可以通过调用loadFullModel、向其中添加TokenizerFactory和SentenceIterator、以及调用还原的模型上的`fit()`来使用200GB的新文本更新模型状态。
* 用于词汇构建的多个数据源的选项被添加。
* 训练和迭代可以单独指定，尽管它们通常都是“1”。
* Word2Vec.Builder 有这个选项: `hugeModelExpected`. 如果设为 `true`, 在构建过程中，词汇将被周期性的截断。
* `minWordFrequency` 有助于忽略语料库中的稀有词，可以排除任何数量的词来定制。
* 两个新的WordVectorsSerialiaztion 方法已被介绍: `writeFullModel` 和  `loadFullModel`. 这些保存和加载一个完整的模型状态。
* 一个体面的工作站应该能够处理一个有几百万单词的词汇量。DL4J的Word2Vec实现可以在一台机器上对兆兆字节的数据进行建模。大致来说，计算公式 是：`vectorSize * 4 * 3 * vocab.size()。`

#### Doc2vec & 其它 NLP 资源

* [用Word2Vec和RNN进行文本分类的DL4J实例](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/word2vecsentiment/Word2VecSentimentRNN.java)
* [段落向量文本分类的DL4J实例](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/paragraphvectors/ParagraphVectorsClassifierExample.java)
* [Doc2vec,或段落向量,用DL4J实现](https://deeplearning4j.org/docs/latest/deeplearning4j-nlp-doc2vec)
* [思维向量、自然语言处理与人工智能的未来](https://skymind.ai/wiki/thought-vectors)
* [Quora:Word2Vec是如何工作的？](https://www.quora.com/How-does-word2vec-work)
* [Quora:什么是有趣的Word2VEC结果？](https://www.quora.com/Word2vec/What-are-some-interesting-Word2Vec-results/answer/Omer-Levy)
* [Mikolov的 Word2vec 最始代码 @Google](https://code.google.com/p/word2vec/)
* [word2vec 解释: Deriving Mikolov et al.’s Negative-Sampling Word-Embedding Method](https://arxiv.org/pdf/1402.3722v1.pdf); Yoav Goldberg and Omer Levy
* [Advances in Pre-Training Distributed Word Representations - by Mikolov et al](https://arxiv.org/abs/1712.09405)

#### 文学中的Word2Vec

```text
It's like numbers are language, like all the letters in the language are turned into numbers, and so it's something that everyone understands the same way. You lose the sounds of the letters and whether they click or pop or touch the palate, or go ooh or aah, and anything that can be misread or con you with its music or the pictures it puts in your mind, all of that is gone, along with the accent, and you have a new understanding entirely, a language of numbers, and everything becomes as clear to everyone as the writing on the wall. So as I say there comes a certain time for the reading of the numbers.
    -- E.L. Doctorow, Billy Bathgate
```


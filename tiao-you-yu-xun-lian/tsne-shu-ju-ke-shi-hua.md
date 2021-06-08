---
description: 高维数据的t-SNE可视化。
---

# T-SNE数据可视化

[T-分布式随机相邻嵌入\(T-SNE\)](https://en.wikipedia.org/wiki/T-distributed_stochastic_neighbor_embedding)是由Delft技术大学的Laurens van der Maaten创建的数据可视化工具。

虽然它可以用于任何数据，但是t-SNE（发音为Tee-Snee）只对标记数据有意义，这说明输入是如何聚类的。下面，你可以看到使用t-SNE处理MNIST数据在DL4J中生成的图形类型。

![](../.gitbook/assets/image%20%281%29.png)

仔细看，你可以看到数字聚集在它们相似的地方，旁边的点。  
下面是T-SNE如何出现在DL4J中的代码。

```java
public class TSNEStandardExample {

    private static Logger log = LoggerFactory.getLogger(TSNEStandardExample.class);

    public static void main(String[] args) throws Exception  {
        //第1步：初始化
        int iterations = 100;
        //create an n-dimensional array of doubles
        DataTypeUtil.setDTypeForContext(DataBuffer.Type.DOUBLE);
        List<String> cacheList = new ArrayList<>(); //cacheList is a dynamic array of strings used to hold all words

        //第2步：将文本输入转换成单词列表
        log.info("Load & Vectorize data....");
        File wordFile = new ClassPathResource("words.txt").getFile();   //打开文件
        //获取所有唯一词向量的数据
        Pair&lt;InMemoryLookupTable,VocabCache&gt; vectors = WordVectorSerializer.loadTxt(wordFile);
        VocabCache cache = vectors.getSecond();
        INDArray weights = vectors.getFirst().getSyn0();    //将独特词的权重分成自己的列表
        for(int i = 0; i &lt; cache.numWords(); i++)   //把字串分隔成自己的列表
            cacheList.add(cache.wordAtIndex(i));

        //第3步：构建双树TSNE以供以后使用
        log.info("Build model....");
        BarnesHutTsne tsne = new BarnesHutTsne.Builder()
                .setMaxIter(iterations).theta(0.5)
                .normalize(false)
                .learningRate(500)
                .useAdaGrad(false)
//                .usePca(false)
                .build();

        //第4步：建立TSNE值并将其保存到文件中
        log.info("Store TSNE Coordinates for Plotting....");
        String outputFile = "target/archive-tmp/tsne-standard-coords.csv";
        (new File(outputFile)).getParentFile().mkdirs();
        tsne.plot(weights,2,cacheList,outputFile);
      //这个tsne将使用向量的权重作为它的矩阵，具有两个维度，使用单词字符串作为标签，并将其写入在前一行创建的outputFile

    }



}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​这是使用gnuplot绘制的tsne-standard-coords.csv文件的图像。

![Tsne data plot](../.gitbook/assets/tsne_output%20%284%29%20%284%29.png)


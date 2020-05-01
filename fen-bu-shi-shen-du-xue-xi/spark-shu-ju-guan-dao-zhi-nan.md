---
description: 'Spark上的DL4J: 如何构建数据管道'
---

# Spark数据管道指南

本页提供了一些关于当在Spark使用DL4J时如何创建用于训练和评估的数据管道的指导。

本页面假设你对Spark（RDD、主节点、工作节点等 ）和DL4J（网络、DataSet等）有一些了解。

与单台机器上的训练一样，数据管道的最后一步应该是生成DataSet（单个特征数组、单个标签数组）或MultiDataSet（一个或多个特征数组、一个或多个标签数组）。在DL4J 在 Spark的情况下，数据管道的最后一步是以下格式之一的数据：\(a\)`RDD<DataSet>/JavaRDD<DataSet>`\(b\)`RDD<MultiDataSet>/JavaRDD<MultiDataSet>`\(c\)网络存储上的串行化数据集/MultiDataSet\(小批量\)对象的目录，如HDFS、S3或Azure blob存储（d）其他格式的小批量目录

一旦数据是上述四种格式之一，就可以将其用于训练或评估。

**注意：**当在单个数据集上训练多个模型时，最佳做法是一次预处理数据，并将其保存到网络存储，如HDFS。然后，当训练网络时，可以调用`SparkDl4jMultiLayer.fit(String path)` 或 `SparkComputationGraph.fit(String path)`，其中path是保存文件的目录。

Spark数据准备：操作指南

* [如何从CSV数据准备用于分类或回归的RDD\[DataSet\]](spark-shu-ju-guan-dao-zhi-nan.md#ru-he-cong-csv-shu-ju-zhun-bei-yong-yu-fen-lei-huo-hui-gui-de-rdddataset)
* [如何从一个或多个RDD\[List\[Writable\]\]创建RDD\[MultiDataSet\]](spark-shu-ju-guan-dao-zhi-nan.md#ru-he-cong-yi-ge-huo-duo-ge-rddlistwritable-chuang-jian-rddmultidataset)
* [如何将RDD\[DataSet\]或RDD\[MultiDataSet\]保存到网络存储并使用它进行训练](spark-shu-ju-guan-dao-zhi-nan.md#ru-he-jiang-rdddataset-huo-rddmultidataset-bao-cun-dao-wang-luo-cun-chu-bing-shi-yong-ta-jin-hang-xun-lian)
* [如何在单个机器上准备集群上使用的数据：保存DataSet](spark-shu-ju-guan-dao-zhi-nan.md#ru-he-zai-dan-ge-ji-qi-shang-zhun-bei-ji-qun-shang-shi-yong-de-shu-ju-bao-cun-dataset)
* [如何在单个机器上准备集群上使用的数据：映射/序列 文件](spark-shu-ju-guan-dao-zhi-nan.md#ru-he-zai-dan-ge-ji-qi-shang-zhun-bei-ji-qun-shang-shi-yong-de-shu-ju-bao-cun-dataset)
* [如何为RNN数据管道加载多个CSV（每个文件一个序列）](spark-shu-ju-guan-dao-zhi-nan.md#ru-he-wei-rnn-shu-ju-guan-dao-jia-zai-duo-ge-csv-mei-ge-wen-jian-yi-ge-xu-lie)
* [如何创建用于图像训练的Spark数据管道](spark-shu-ju-guan-dao-zhi-nan.md#ru-he-chuang-jian-yong-yu-tu-xiang-xun-lian-de-spark-shu-ju-guan-dao)
* [如何加载准备好的定制格式的小批量](spark-shu-ju-guan-dao-zhi-nan.md#ru-he-jia-zai-zhun-bei-hao-de-ding-zhi-ge-shi-de-xiao-pi-liang)

## [如何从CSV数据准备用于分类或回归的RDD\[DataSet\]](spark-shu-ju-guan-dao-zhi-nan.md)

本指南展示了如何加载包含在一个或多个CSV文件中的数据，并产生一个“JavaDRD＜DataSet＞”，用于在Spark上导出、训练或评估。

这个过程相当简单。请注意，`DataVecDataSetFunction`函数与非常常用于单机训练的`RecordReaderDataSetIterator`非常类似。

例如，假设CSV具有以下格式——6个总列：5个特征，然后是一个用于分类的整数类索引，以及10个可能的分类

```text
1.0,3.2,4.5,1.1,6.3,0
1.6,2.4,5.9,0.2,2.2,1
...
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

我们可以使用下面的代码加载这些数据进行分类：

```java
String filePath = "hdfs:///your/path/some_csv_file.csv";
JavaSparkContext sc = new JavaSparkContext();
JavaRDD<String> rddString = sc.textFile(filePath);
RecordReader recordReader = new CSVRecordReader(',');
JavaRDD<List<Writable>> rddWritables = rddString.map(new StringToWritablesFunction(recordReader));

int labelIndex = 5;         //标签索引
int numLabelClasses = 10;   //10 个分类的标签
JavaRDD<DataSet> rddDataSetClassification = rddWritables.map(new DataVecDataSetFunction(labelIndex, numLabelClasses, false));
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

然而，如果这个数据集是用于回归的，那么再有6个总列、3个特征列（文件行中的位置0、1和2）和3个标签列（位置3、4和5），我们可以使用与上面相同的过程加载它，但是将最后3行改为：

```java
int firstLabelColumn = 3;   //标签的第一列索引
int lastLabelColumn = 5;    //标签的最后一列索引
JavaRDD<DataSet> rddDataSetRegression = rddWritables.map(new DataVecDataSetFunction(firstColumnLabel, lastColumnLabel, true, null, null));
```

## 如何从一个或多个RDD\[List\[Writable\]\]创建RDD\[MultiDataSet\]

RecordReaderMultiDataSetIterator \(RRMDSI\) 是为单机训练数据管道创建MultiDataSet实例的最常用方法。可以将RRMDSI用于Spark数据管道，其中数据来自一个或多个`RDD<List<Writable>>`（对于“标准”数据）或`RDD<List<List<Writable>>`（对于序列数据）。

**案例1: Single单 `RDD<List<Writable>>` 到 `RDD<MultiDataSet>`**

考虑CSV分类任务的以下单节点（非Spark）数据管道。

```java
RecordReader recordReader = new CSVRecordReader(numLinesToSkip,delimiter);
recordReader.initialize(new FileSplit(new ClassPathResource("iris.txt").getFile()));

int batchSize = 32;
int labelColumn = 4;
int numClasses = 3;
MultiDataSetIterator iter = new RecordReaderMultiDataSetIterator.Builder(batchSize)
    .addReader("data", recordReader)
    .addInput("data", 0, labelColumn-1)
    .addOutputOneHot("data", labelColumn, numClasses)
    .build();
```

相当于以下Spark数据管道：

```java
JavaRDD<List<Writable>> rdd = sc.textFile(f.getPath()).map(new StringToWritablesFunction(new CSVRecordReader()));

MultiDataSetIterator iter = new RecordReaderMultiDataSetIterator.Builder(batchSize)
    .addReader("data", new SparkSourceDummyReader(0))        //Note the use of the "SparkSourceDummyReader"
    .addInput("data", 0, labelColumn-1)
    .addOutputOneHot("data", labelColumn, numClasses)
    .build();
JavaRDD<MultiDataSet> mdsRdd = IteratorUtils.mapRRMDSI(rdd, rrmdsi2);
```

对于序列数据 \(`List<List<Writable>>`\) 你可以使用SparkSourceDummySeqReader 来代替.

**案例2: Multiple多 `RDD<List<Writable>>` 或  `RDD<List<List<Writable>>` 到 `RDD<MultiDataSet>`**

在这种情况下，过程基本相同。但是，在内部，使用连接。

```java
JavaRDD<List<Writable>> rdd1 = ...
JavaRDD<List<Writable>> rdd2 = ...

RecordReaderMultiDataSetIterator rrmdsi = new RecordReaderMultiDataSetIterator.Builder(batchSize)
    .addReader("rdd1", new SparkSourceDummyReader(0))        //0 =使用列表中的第一个rdd 
    .addReader("rdd2", new SparkSourceDummyReader(1))        //1 =使用列表中的第二个rdd 
    .addInput("rdd1", 1, 2)            //
    .addOutput("rdd2", 1, 2)
    .build();

List<JavaRDD<List<Writable>>> list = Arrays.asList(rdd1, rdd2);
int[] keyIdxs = new int[]{0,0};        //rdd1和rdd2中的列0是用于连接的“键”
boolean filterMissing = false;       //如果为true：过滤掉所有rdd中没有匹配键的所有记录 
JavaRDD<MultiDataSet> mdsRdd = IteratorUtils.mapRRMDSI(list, null, keyIdxs, null, filterMissing, rrmdsi);
```

## [如何将RDD\[DataSet\]或RDD\[MultiDataSet\]保存到网络存储并使用它进行训练](spark-shu-ju-guan-dao-zhi-nan.md)

如本页开头所述，预处理和导出数据一次（即，保存到诸如HDFS之类的网络存储和重用）而不是在每个训练作业中直接从RDD&lt;DataSet&gt;或RDD&lt;MultiDataSet&gt;拟合，被认为是最佳做法。

这有很多原因：

* 更好的性能（避免冗余加载/计算）：当拟合来自同一数据集的多个模型时，一次预处理该数据并将其保存到磁盘要比每次训练运行再次预处理更快。
* 最小化内存和其他资源：通过从磁盘导出和拟合，我们只需要将当前使用的DataSets（加上一个小的异步预取缓冲区）保存在内存中，而不需要将许多未使用的DataSet对象保存在内存中。导出导致较低的总内存使用量，因此我们可以使用更大的网络、更大的小批量大小，或者为作业分配更少的资源。
* 避免重新计算：当RDD太大而不能放入内存时，可能需要在使用RDD之前重新计算RDD的某些部分（取决于缓存设置）。当这种情况发生时，Spark将多次重新计算数据管道的一部分，这会耗费时间和内存。预导出步骤完全避免了这种重新计算。

**第1步：保存**

一旦有了RDD&lt;DataSet&gt;，保存DataSet对象是非常简单的：

```java
JavaRDD<DataSet> rddDataSet = ...
int minibatchSize = 32;     //Minibatch size of the saved DataSet objects
String exportPath = "hdfs:///path/to/export/data";
JavaRDD<String> paths = rddDataSet.mapPartitionsWithIndex(new BatchAndExportDataSetsFunction(minibatchSize, exportPath), true);
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

请记住，这是一个映射函数，因此在执行路径RDD之前，不会保存任何数据——即，你应该执行以下操作：

```java
paths.saveAsTextFile("hdfs:///path/to/text/file.txt");  //指定文件将包含所有保存的数据集对象的路径/URI。
```

或

```java
List<String> paths = paths.collect();    //所有保存的数据集对象的路径/ URI的集合
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

或

```java
paths.foreach(new VoidFunction<String>() {
    @Override
    public void call(String path) {
        //Some operation on each path
    }
});
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

可以使用BatchAndExportMultiDataSetsFunction以相同的方式保存RDD&lt;MultiDataSet&gt;，它采用相同的参数。

**第2步：加载和拟合**

导出的数据可以以几种方式使用。首先，它可以直接用于拟合网络：

```java
String exportPath = "hdfs:///path/to/export/data";
SparkDl4jMultiLayer net = ...
net.fit(exportPath);      //加载在“exportPath”目录中找到的序列化数据集对象
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

类似地，如果我们保存RDD&lt;MultiDataSet&gt;，则可以使用SparkComputationGraph.fitMultiDataSet\(String path\)。  
或者，我们可以以几种不同的方式加载路径，这取决于我们是否或者如何保存它们：

```java
JavaSparkContext sc = new JavaSparkContext();

//如果我们使用saveAsTextFile:
String saveTo = "hdfs:///path/to/text/file.txt";
paths.saveAsTextFile(saveTo);                         //Save
JavaRDD<String> loadedPaths = sc.textFile(saveTo);    //Load

//如果我们使用collecting:
List<String> paths = paths.collect();                 //Collect
JavaRDD<String> loadedPaths = sc.parallelize(paths);  //Parallelize

//如果我们想要列出目录的内容:
String exportPath = "hdfs:///path/to/export/data";
JavaRDD<String> loadedPaths = SparkUtils.listPaths(sc, exportPath);   //使用 org.deeplearning4j.spark.util.SparkUtils 列出目录
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

然后我们可以使用诸如SparkDl4jMultiLayer.fitPaths\(JavaRDD&lt;String&gt;\)之类的方法在这些路径上执行训练。

## [如何在单个机器上准备集群上使用的数据：保存DataSet](spark-shu-ju-guan-dao-zhi-nan.md)

另一种可能的工作流程是从单台机器上的数据管道开始，导出DataSet或MultiDataSet对象以便在集群上使用。显然，该工作流不像在集群上准备数据那样具有可伸缩性（你只使用一台机器来准备数据），但是在某些情况下，它可以是一个简单的选项，尤其是在你拥有现有数据管道的情况下。

本节假设你有一个用于单机训练的现有DataSetIterator或MultiDataSetIterator。有许多不同的方法可以创建一个，这超出了本指南的范围。

**第1步: 保存 DataSets 或 MultiDataSets**

可以使用以下代码将DataSet的内容保存到本地目录：

```java
DataSetIterator iter = ...
File rootDir = new File("/saving/directory/");
int count = 0;
while(iter.hasNext()){
  DataSet ds = iter.next();
  File outFile = new File(rootDir, "dataset_" + (count++) + ".bin");
  ds.save(outFile);
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

注意，对于Spark的目的，确切的文件名并不重要。保存MultiDataSet的过程几乎是相同的。  
另外：可以使用FileDataSetIterator在单个机器上读取这些保存的DataSet对象（用于非Spark训练）。  
另一种方法是使用输出流直接保存到集群，例如，保存到（例如）HDFS。这只有在运行代码的机器正确配置了所需的库和访问权限时才能实现。例如，将数据集直接保存到HDFS，你可以使用：

```java
JavaSparkContext sc = new JavaSparkContext();
FileSystem fileSystem = FileSystem.get(sc.hadoopConfiguration());
String outputDir = "hdfs:///my/output/location/";

DataSetIterator iter = ...
int count = 0;
while(iter.hasNext()){
  DataSet ds = iter.next();
  String filePath = outputDir + "dataset_" + (count++) + ".bin";
  try (OutputStream os = new BufferedOutputStream(fileSystem.create(new Path(outputPath)))) {
    ds.save(os);
  }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**第2步：在集群上加载和训练** 

然后可以将保存的DataSet对象复制到集群或网络文件存储（例如，使用Hadoop集群上的Hadoop FS实用程序），并且如下使用：

```java
String dir = "hdfs:///data/copied/here";
SparkDl4jMultiLayer net = ...
net.fit(dir);      //Loads the serialized DataSet objects found in the 'dir' directory
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

或者可选地/等价地，我们可以使用RDD列出路径：

```java
String dir = "hdfs:///data/copied/here";
JavaRDD<String> paths = SparkUtils.listPaths(sc, dir);   //List paths using org.deeplearning4j.spark.util.SparkUtils
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## [如何在单个机器上准备集群上使用的数据：映射/序列 文件](spark-shu-ju-guan-dao-zhi-nan.md)

另一种方法是使用Hadoop MapFile和SequenceFiles，它们是有效的二进制存储格式。这可以用于将任何DataVec RecordReader或SequenceRecordReader（包括自定义记录读取器）的输出转换为可用于Spark的格式。MapFileRecordWriter和MapFileSequenceRecordWriter需要以下依赖项：

```markup
<dependency>
    <groupId>org.datavec</groupId>
    <artifactId>datavec-hadoop</artifactId>
    <version>${datavec.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>${hadoop.version}</version>
    <!-- Optional exclusion for log4j in case you are using other logging frameworks -->
    <!--
    <exclusions>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
    -->
</dependency>
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**第1步：在本地创建一个MapFile**

在下面的示例中，将使用CSVRecordReader，但是任何其他RecordReader都可以在其位置使用：

```java
File csvFile = new File("/path/to/file.csv")
RecordReader recordReader = new CSVRecordReader();
recordReader.initialize(new FileSplit(csvFile));

//创建映射文件写入器
String outPath = "/map/file/root/dir"
MapFileRecordWriter writer = new MapFileRecordWriter(new File(outPath));

//转换为MapFile二进制格式:
RecordReaderConverter.convert(recordReader, writer);
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

使用SequenceRecordReader结合MapFileSequenceRecordWriter的过程实际上是相同的。

还请注意，MapFileRecordWriter和MapFileSequenceRecordWriter都支持拆分，即创建多个较小的映射文件而不是创建单个（可能为多GB）的映射文件。当以这种方式保存数据以便与Spark一起使用时，建议使用拆分。

**第2步: 复制到HDFS或其他网络文件存储**

确切的过程超出了本指南的范围。但是，只要将目录（上面示例中的“/map/file/root/dir”）复制到HDFS上的位置就足够了。

**第3步: 为训练读取和转换为`RDD<DataSet>`** 

我们可以使用以下方法加载数据进行训练：

```java
JavaSparkContext sc = new JavaSparkContext();
String pathOnHDFS = "hdfs:///map/file/directory";
JavaRDD<List<Writable>> rdd = SparkStorageUtils.restoreMapFile(pathOnHDFS, sc);     //import: org.datavec.spark.storage.SparkStorageUtils

//Note at this point: it's the same as the latter part of the CSV how-to guide
int labelIndex = 5;         //Labels: a single integer representing the class index in column number 5
int numLabelClasses = 10;   //10 classes for the label
JavaRDD<DataSet> rddDataSetClassification = rdd.map(new DataVecDataSetFunction(labelIndex, numLabelClasses, false));
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## [如何为RNN数据管道加载多个CSV（每个文件一个序列）](spark-shu-ju-guan-dao-zhi-nan.md)

本指南展示了如何加载训练RNN的CSV文件。假设数据集由多个CSV文件组成，其中：

* 每个CSV文件代表一个序列
* CSV的每一行包含一个时间步的值（一个或多个列/值，所有文件的所有行中的相同数量的值）
* 每个CSV可以包含到其他CSV的不同数量的行（即，可变长度序列在这里是可以的）
* 标题行既不存在于任何文件中，也不存在于所有文件中。

可以使用以下过程创建数据管道：

```java
String directoryWithCsvFiles = "hdfs:///path/to/directory";
JavaPairRDD<String, PortableDataStream> origData = sc.binaryFiles(directoryWithCsvFiles);

int numHeaderLinesEachFile = 0; //No header lines
int delimiter = ",";            //Comma delimited files
SequenceRecordReader seqRR = new CSVSequenceRecordReader(numHeaderLinesEachFile, delimiter);

JavaRDD<List<List<Writable>>> sequencesRdd = origData.map(new SequenceRecordReaderFunction(seqRR));

//Similar to the non-sequence CSV guide using DataVecDataSetFunction. Assuming classification here:
int labelIndex = 5;             //Index of the label column. Occurs at position/column 5
int numClasses = 10;            //Number of classes for classification
JavaRDD<DataSet> dataSetRdd = sequencesRdd.map(new DataVecSequenceDataSetFunction(labelIndex, numClasses, false));
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## [如何创建用于图像训练的Spark数据管道](spark-shu-ju-guan-dao-zhi-nan.md)

本指南说明如何从本地或HDFS等网络文件系统存储的图像开始，创建用于图像分类的`RDD<DataSet>`。

这里使用的方法（在1.0.0-beta3中添加）是首先将图像预处理成批文件—[FileBatch](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-common/src/main/java/org/nd4j/api/loader/FileBatch.java)对象。这种方法的动机很简单：原始图像文件通常使用高效的压缩（例如，JPEG），这比位图（int8或32位浮点）表示更有效。然而，在集群中，我们希望最小化磁盘读取，由于远程存储延迟导致的问题——一个文件读取/传输将比minibatchSize远程文件读取更快。

“[TinyImageNet](https://github.com/deeplearning4j/dl4j-examples/tree/master/dl4j-spark-examples/dl4j-spark/src/main/java/org/deeplearning4j/tinyimagenet)”示例也说明了如何做到这一点。  
注意，该实现的一个限制是需要手动知道、提供或收集一组类（即，在进行分类时类/类别标签）。这与在单个机器上使用ImageRecordReader进行分类不同，后者可以自动推断类标签集。

首先，假设图像是在基于它们的类标签的子目录。例如，假设存在两个类“cat”和“dog”，则目录结构如下所示：

```text
rootDir/cat/img0.jpg
rootDir/cat/img1.jpg
...
rootDir/dog/img0.jpg
rootDir/dog/img1.jpg
...
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

（注意，在这个示例中，文件名并不重要，但是，父目录名是类标签）

**第1步（2的选项1）：在本地进行预处理**

本地预处理可以按如下完成：

```java
String sourceDirectory = "/home/user/my_images";            //你数据的位置
String destinationDirectory = "/home/user/preprocessed";    //预处理数据将要保存的地方
int batchSize = 32;                                         //每个FileBatch对象中示例的数量
SparkDataUtils.createFileBatchesLocal(sourceDirectory, NativeImageLoader.ALLOWED_FORMATS, true, saveDirTrain, batchSize);
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

SparkDataUtils 的完整导入是 `org.deeplearning4j.spark.util.SparkDataUtils`.

在完成预处理之后，可以将目录复制到集群中用于训练（步骤2）。

**第1步（2的选项2）：使用Spark进行预处理**

或者，如果原始图像在远程文件存储（例如HDFS）上，则可以使用以下方法：

```java
String sourceDirectory = “hdfs:///data/my_images”; //你数据的位置 
destinationDirectory = “hdfs:///data/preprocessed”; //预处理数据将要保存的地方 
written int batchSize = 32; //每个FileBatch对象中示例的数量 
SparkDataUtils.createFileBatchesSpark(sourceDirectory, destinationDirectory, batchSize, sparkContext);
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**第2步: 训练** 

图像分类的数据管道可以构造如下。这个代码取自[TinyImageNet](https://github.com/deeplearning4j/dl4j-examples/blob/master/dl4j-spark-examples/dl4j-spark/src/main/java/org/deeplearning4j/tinyimagenet/TrainSpark.java) 示例：

```java
//创建数据加载器
int imageHeightWidth = 64;      //输入到网络的64x64像素
int imageChannels = 3;          //RGB
PathLabelGenerator labelMaker = new ParentPathLabelGenerator();
ImageRecordReader rr = new ImageRecordReader(imageHeightWidth, imageHeightWidth, imageChannels, labelMaker);
rr.setLabels(Arrays.asList("cat", "dog"));
int numClasses = 2;
RecordReaderFileBatchLoader loader = new RecordReaderFileBatchLoader(rr, minibatch, 1, numClasses);
loader.setPreProcessor(new ImagePreProcessingScaler()); //缩放0-255值像素到0-1范围


//拟合网络
String trainDataPath = "hdfs:///data/preprocessed"; //预处理数据所在的位置
JavaRDD<String> pathsTrain = SparkUtils.listPaths(sc, trainDataPath);
for (int i = 0; i < numEpochs; i++) {
    sparkNet.fitPaths(pathsTrain, loader);
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

就是这样。  
注意：对于其他标签生成情况（比如从文件名而不是父目录中提供的标签），或者对于诸如语义分割之类的任务，你可以替换不同的PathLabelGenerator而不是默认的。例如，如果标签应该来自文件名，则可以使用PatternPathLabelGenerator。假设图像的格式为“cat\_img1234.jpg”、“dog\_2309.png”等。

```java
PathLabelGenerator labelGenerator = new PatternPathLabelGenerator("_", 0);  //用"_"分割，并用分割后的第一个值
ImageRecordReader imageRecordReader = new ImageRecordReader(imageHW, imageHW, imageChannels, labelGenerator);
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

注意，PathLabelGenerator返回一个Writable对象，所以对于像图像分割这样的任务，可以使用自定义PathLabelGenerator中的NDArrayWritable类返回INDArray。

## [如何加载准备好的定制格式的小批量](spark-shu-ju-guan-dao-zhi-nan.md)

DL4J Spark训练支持加载以自定义格式系列化的数据的能力。假设远程/网络存储中的每个文件都以某种可读格式表示单个小批量数据。

请注意，此方法通常不需要或不推荐给大多数用户，但作为高级用户或这些以自定义格式或不由DL4J本地支持的格式预先准备好的数据的用户提供附加选项。当文件以自定义格式表示单个记录/示例（而不是小批量）时，可以使用自定义RecordReader。

需要注意的接口是：

* [DataSetLoader](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-core/src/main/java/org/deeplearning4j/api/loader/DataSetLoader.java)
* [MultiDataSetLoader](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-core/src/main/java/org/deeplearning4j/api/loader/MultiDataSetLoader.java)

这两种方法都扩展了单方法[Loader](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-common/src/main/java/org/nd4j/api/loader/Loader.java)接口。  
假设HDFS目录包含许多文件，每个文件都是某种定制格式的小批量文件。这些可以使用以下过程加载：

```java
JavaSparkContext sc = new JavaSparkContext();
String dataDirectory = "hdfs:///path/with/data";
JavaRDD<String> loadedPaths = SparkUtils.listPaths(sc, dataDirectory);   //使用 org.deeplearning4j.spark.util.SparkUtils 列出路径

SparkDl4jMultiLayer net = ...
Loader<DataSet> myCustomLoader = new MyCustomLoader();
net.fitPaths(loadedPaths, myCustomLoader);
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

自定义加载器类看起来像：

```java
public class MyCustomLoader implements DataSetLoader {
    @Override
    public DataSet load(Source source) throws IOException {
        InputStream inputStream = source.getInputStream();
        <load custom data format here> 
        INDArray features = ...;
        INDArray labels = ...;
        return new DataSet(features, labels);
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)


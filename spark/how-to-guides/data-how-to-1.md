---
title: "Spark Data Pipelines"
description: "Loading and preprocessing data on Apache Spark for distributed DL4J training"
---

# Spark Data Pipelines for Distributed Training

This page covers how to build data pipelines for DL4J distributed training on Apache Spark. It assumes familiarity with basic Spark concepts (RDDs, partitions, executors) and DL4J's `DataSet`/`MultiDataSet` classes.

The final output of any data pipeline must be one of:
- `JavaRDD<DataSet>` — for single input/output networks
- `JavaRDD<MultiDataSet>` — for multi-input/output networks
- A directory of serialized `DataSet`/`MultiDataSet` files on network storage (HDFS, S3, Azure Blob)
- A directory of minibatches in a custom format

**Best practice:** Preprocess your data once and save it to HDFS. Then train by pointing `SparkDl4jMultiLayer.fit(String path)` at that directory. This avoids recomputing the pipeline on every training run and reduces memory pressure during training.

**Contents:**
- [CSV data for classification or regression](#csv)
- [Image classification pipelines](#images)
- [MultiDataSet from multiple RDDs](#multidataset)
- [Save and load RDD DataSet to network storage](#saveloadrdd)
- [Prepare data on a single machine, use on a cluster](#singletocluster)
- [Hadoop MapFile/SequenceFile format](#singletocluster2)
- [RNN data from multiple CSV files](#csvseq)
- [Custom minibatch format](#customformat)

---

## <a name="csv"></a>CSV Data for Classification or Regression

To load a CSV file from HDFS and produce a `JavaRDD<DataSet>`:

**Classification** (6 columns total: columns 0–4 are features, column 5 is an integer class index, 10 classes):

```java
String filePath = "hdfs:///data/train.csv";
JavaSparkContext sc = new JavaSparkContext();

JavaRDD<String> lines = sc.textFile(filePath);
RecordReader rr = new CSVRecordReader(',');
JavaRDD<List<Writable>> records = lines.map(new StringToWritablesFunction(rr));

int labelIndex = 5;
int numClasses = 10;
JavaRDD<DataSet> rddDataSet = records.map(
    new DataVecDataSetFunction(labelIndex, numClasses, false));
```

**Regression** (6 columns total: columns 0–2 are features, columns 3–5 are labels):

```java
int firstLabel = 3;
int lastLabel  = 5;
JavaRDD<DataSet> rddDataSet = records.map(
    new DataVecDataSetFunction(firstLabel, lastLabel, true, null, null));
```

`DataVecDataSetFunction` is the Spark equivalent of `RecordReaderDataSetIterator` for single-machine pipelines.

---

## <a name="images"></a>Image Classification Pipelines

Image pipelines on Spark use a two-step process: first batch images into `FileBatch` objects (which preserve efficient compression like JPEG), then load those batches during training. This avoids per-image remote reads during training.

**Step 1a: Preprocess locally**

```java
String sourceDir = "/home/user/my_images";       // local or accessible path
String destDir   = "/home/user/preprocessed";
int batchSize    = 32;

// Requires images organized in class-named subdirs: sourceDir/cat/img.jpg, etc.
SparkDataUtils.createFileBatchesLocal(
    sourceDir, NativeImageLoader.ALLOWED_FORMATS, true, destDir, batchSize);
```

Then copy `destDir` to HDFS: `hadoop fs -put /home/user/preprocessed hdfs:///data/preprocessed`

**Step 1b: Preprocess using Spark** (if images are already on HDFS)

```java
String sourceDir = "hdfs:///data/raw_images";
String destDir   = "hdfs:///data/preprocessed";
int batchSize    = 32;

SparkDataUtils.createFileBatchesSpark(sourceDir, destDir, batchSize, sc);
```

**Step 2: Training from preprocessed batches**

```java
int imageHW      = 64;   // height and width in pixels
int imageChannels = 3;   // RGB

PathLabelGenerator labelMaker = new ParentPathLabelGenerator();
ImageRecordReader rr = new ImageRecordReader(imageHW, imageHW, imageChannels, labelMaker);
rr.setLabels(Arrays.asList("cat", "dog"));

RecordReaderFileBatchLoader loader = new RecordReaderFileBatchLoader(rr, minibatch, 1, numClasses);
loader.setPreProcessor(new ImagePreProcessingScaler());  // scale [0,255] -> [0,1]

JavaRDD<String> paths = SparkUtils.listPaths(sc, "hdfs:///data/preprocessed");
for (int epoch = 0; epoch < numEpochs; epoch++) {
    sparkNet.fitPaths(paths, loader);
}
```

For labels from filenames rather than parent directories (e.g., `cat_img1234.jpg`):
```java
PathLabelGenerator labelGen = new PatternPathLabelGenerator("_", 0); // split on "_", take first token
ImageRecordReader rr = new ImageRecordReader(imageHW, imageHW, imageChannels, labelGen);
```

---

## <a name="multidataset"></a>MultiDataSet from Multiple RDDs

Use `RecordReaderMultiDataSetIterator` (RRMDSI) with `SparkSourceDummyReader` to bridge between Spark RDDs and the multi-dataset pipeline API.

**Case 1: Single `RDD<List<Writable>>` to `RDD<MultiDataSet>`**

Single-machine equivalent:
```java
RecordReaderMultiDataSetIterator iter = new RecordReaderMultiDataSetIterator.Builder(batchSize)
    .addReader("data", csvRecordReader)
    .addInput("data", 0, 3)            // columns 0-3 as features
    .addOutputOneHot("data", 4, 3)     // column 4 as one-hot label (3 classes)
    .build();
```

Spark equivalent:
```java
JavaRDD<List<Writable>> rdd = sc.textFile(path)
    .map(new StringToWritablesFunction(new CSVRecordReader()));

RecordReaderMultiDataSetIterator rrmdsi = new RecordReaderMultiDataSetIterator.Builder(batchSize)
    .addReader("data", new SparkSourceDummyReader(0))   // 0 = use first RDD
    .addInput("data", 0, 3)
    .addOutputOneHot("data", 4, 3)
    .build();

JavaRDD<MultiDataSet> mdsRdd = IteratorUtils.mapRRMDSI(rdd, rrmdsi);
```

**Case 2: Multiple RDDs joined into `RDD<MultiDataSet>`**

```java
JavaRDD<List<Writable>> rdd1 = ...;   // features
JavaRDD<List<Writable>> rdd2 = ...;   // labels

RecordReaderMultiDataSetIterator rrmdsi = new RecordReaderMultiDataSetIterator.Builder(batchSize)
    .addReader("rdd1", new SparkSourceDummyReader(0))   // 0 = rdd1
    .addReader("rdd2", new SparkSourceDummyReader(1))   // 1 = rdd2
    .addInput("rdd1", 1, 2)
    .addOutput("rdd2", 1, 2)
    .build();

List<JavaRDD<List<Writable>>> rdds = Arrays.asList(rdd1, rdd2);
int[] keyColumns = new int[]{0, 0};   // column 0 in each RDD is the join key
boolean filterMissing = false;

JavaRDD<MultiDataSet> mdsRdd = IteratorUtils.mapRRMDSI(
    rdds, null, keyColumns, null, filterMissing, rrmdsi);
```

For sequence data (`RDD<List<List<Writable>>>`), use `SparkSourceDummySeqReader` in place of `SparkSourceDummyReader`.

---

## <a name="saveloadrdd"></a>Save and Load RDD DataSet to Network Storage

Saving preprocessed data to HDFS and loading it for training avoids recomputing the pipeline and reduces memory use during training. This is the recommended workflow for multi-run experiments.

**Save a `JavaRDD<DataSet>` to HDFS:**

```java
JavaRDD<DataSet> rddDataSet = ...;
int minibatchSize = 32;
String exportPath = "hdfs:///data/preprocessed";

JavaRDD<String> savedPaths = rddDataSet.mapPartitionsWithIndex(
    new BatchAndExportDataSetsFunction(minibatchSize, exportPath), true);

// The map is lazy — trigger execution by collecting paths:
List<String> paths = savedPaths.collect();
```

For `JavaRDD<MultiDataSet>`, use `BatchAndExportMultiDataSetsFunction` instead. It takes the same arguments.

**Load and train directly from the saved directory:**

```java
SparkDl4jMultiLayer net = ...;
net.fit("hdfs:///data/preprocessed");   // loads all DataSets in the directory
```

Or for `MultiDataSet`:
```java
net.fitMultiDataSet("hdfs:///data/preprocessed");
```

**Manual path loading:**

```java
// From a text file of paths written with saveAsTextFile:
JavaRDD<String> paths = sc.textFile("hdfs:///path/to/paths.txt");

// From listing a directory:
JavaRDD<String> paths = SparkUtils.listPaths(sc, "hdfs:///data/preprocessed");

// Fit from explicit paths:
net.fitPaths(paths);
```

---

## <a name="singletocluster"></a>Prepare Data on a Single Machine, Use on a Cluster

If you have an existing single-machine data pipeline, you can export `DataSet` objects locally and copy them to HDFS.

**Step 1: Save DataSets locally**

```java
DataSetIterator iter = ...;
File rootDir = new File("/local/output/datasets/");
rootDir.mkdirs();

int count = 0;
while (iter.hasNext()) {
    DataSet ds = iter.next();
    ds.save(new File(rootDir, "dataset_" + count++ + ".bin"));
}
```

To save directly to HDFS (if the machine has HDFS client access):
```java
FileSystem fs = FileSystem.get(sc.hadoopConfiguration());
String outputDir = "hdfs:///data/datasets/";
int count = 0;
while (iter.hasNext()) {
    DataSet ds = iter.next();
    String path = outputDir + "dataset_" + count++ + ".bin";
    try (OutputStream os = new BufferedOutputStream(fs.create(new Path(path)))) {
        ds.save(os);
    }
}
```

**Step 2: Copy to HDFS and train**

```bash
hadoop fs -put /local/output/datasets/ hdfs:///data/datasets/
```

```java
SparkDl4jMultiLayer net = ...;
net.fit("hdfs:///data/datasets/");
```

Note: you can also use `FileDataSetIterator` to read locally saved DataSets on a single machine without Spark.

---

## <a name="singletocluster2"></a>Hadoop MapFile/SequenceFile Format

An alternative to serialized DataSet files is Hadoop's MapFile/SequenceFile binary format. This can convert any `RecordReader` or `SequenceRecordReader` output into a Spark-compatible format.

**Dependencies:**

```xml
<dependency>
    <groupId>org.datavec</groupId>
    <artifactId>datavec-hadoop</artifactId>
    <version>${datavec.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>${hadoop.version}</version>
</dependency>
```

**Step 1: Create a MapFile locally**

```java
RecordReader rr = new CSVRecordReader();
rr.initialize(new FileSplit(new File("/data/file.csv")));

MapFileRecordWriter writer = new MapFileRecordWriter(new File("/output/mapfile/"));
RecordReaderConverter.convert(rr, writer);
```

`MapFileRecordWriter` supports splitting into multiple smaller files, which is recommended for Spark parallelism.

**Step 2: Copy to HDFS**

```bash
hadoop fs -put /output/mapfile/ hdfs:///data/mapfile/
```

**Step 3: Load into `RDD<DataSet>` for training**

```java
String hdfsPath = "hdfs:///data/mapfile/";
JavaRDD<List<Writable>> rdd = SparkStorageUtils.restoreMapFile(hdfsPath, sc);

int labelIndex = 5;
int numClasses = 10;
JavaRDD<DataSet> rddDataSet = rdd.map(new DataVecDataSetFunction(labelIndex, numClasses, false));
```

---

## <a name="csvseq"></a>RNN Data from Multiple CSV Files

For RNN/sequence datasets where each CSV file is one sequence:
- Each row of the CSV is one time step.
- Files may have different numbers of rows (variable-length sequences are supported).
- All files must have the same number of columns.

```java
String directory = "hdfs:///data/sequences/";
JavaPairRDD<String, PortableDataStream> rawData = sc.binaryFiles(directory);

int numHeaderLines = 0;
String delimiter   = ",";
SequenceRecordReader seqRR = new CSVSequenceRecordReader(numHeaderLines, delimiter);

JavaRDD<List<List<Writable>>> sequencesRdd =
    rawData.map(new SequenceRecordReaderFunction(seqRR));

int labelIndex = 5;
int numClasses = 10;
JavaRDD<DataSet> rddDataSet =
    sequencesRdd.map(new DataVecSequenceDataSetFunction(labelIndex, numClasses, false));
```

---

## <a name="customformat"></a>Custom Minibatch Format

For data stored in a custom binary format (one minibatch per file), implement the `DataSetLoader` or `MultiDataSetLoader` interface:

```java
public class MyCustomLoader implements DataSetLoader {
    @Override
    public DataSet load(Source source) throws IOException {
        try (InputStream is = source.getInputStream()) {
            // deserialize your custom format
            INDArray features = ...;
            INDArray labels   = ...;
            return new DataSet(features, labels);
        }
    }
}
```

Then use it at training time:

```java
JavaRDD<String> paths = SparkUtils.listPaths(sc, "hdfs:///data/custom/");
SparkDl4jMultiLayer net = ...;
net.fitPaths(paths, new MyCustomLoader());
```

This approach is typically not needed unless you have pre-existing data in a format DL4J does not natively support (e.g., a proprietary binary protocol or a Parquet-based format).

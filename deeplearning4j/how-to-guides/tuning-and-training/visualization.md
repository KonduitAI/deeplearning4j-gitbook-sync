---
title: "Training Visualization"
description: "Visualizing training progress — VertxUIServer, StatsListener, the training dashboard, and t-SNE visualization"
---

# Training Visualization

Deeplearning4j ships with a browser-based training dashboard and support for t-SNE embedding visualization. Both tools help diagnose training problems and understand what a network has learned.

---

## Part 1: The Training Dashboard

### Maven Dependency

The training UI requires the `deeplearning4j-ui` artifact:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-ui</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

For Spark or remote-UI clients that only need `StatsListener` without running the full UI server, use the lighter-weight model artifact:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-ui-model</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

### Setup

Three objects are involved: the UI server, a stats storage backend, and the `StatsListener` attached to the network.

```java
import org.deeplearning4j.ui.api.UIServer;
import org.deeplearning4j.ui.model.stats.StatsListener;
import org.deeplearning4j.ui.model.storage.InMemoryStatsStorage;

// 1. Start the embedded Vertx-based UI server (singleton)
UIServer uiServer = UIServer.getInstance();

// 2. Choose a storage backend
//    InMemoryStatsStorage: fast, data lost on exit
//    FileStatsStorage:     persists to disk, can be replayed later
StatsStorage statsStorage = new InMemoryStatsStorage();
// -- or --
// StatsStorage statsStorage = new FileStatsStorage(new File("training-stats.dl4j"));

// 3. Attach storage to the server so the UI can read from it
uiServer.attach(statsStorage);

// 4. Attach StatsListener to the network
//    The integer argument is the reporting frequency (every N iterations)
net.setListeners(new StatsListener(statsStorage, 1));
```

Open `http://localhost:9000/train` in a browser. Statistics are pushed to the UI as the network trains.

To change the port, set the system property before starting the server:

```bash
-Dorg.deeplearning4j.ui.port=9001
```

Or in code:

```java
System.setProperty("org.deeplearning4j.ui.port", "9001");
UIServer uiServer = UIServer.getInstance();
```

### FileStatsStorage — Saving and Replaying Stats

Use `FileStatsStorage` to persist training statistics to disk. The file can be loaded and visualized offline after training completes.

**Writing during training:**

```java
StatsStorage statsStorage = new FileStatsStorage(new File("run1-stats.dl4j"));
uiServer.attach(statsStorage);
net.setListeners(new StatsListener(statsStorage));
```

**Reading after training:**

```java
StatsStorage statsStorage = new FileStatsStorage(new File("run1-stats.dl4j"));
UIServer uiServer = UIServer.getInstance();
uiServer.attach(statsStorage);
// Navigate to http://localhost:9000/train to view
```

---

## Dashboard Pages

### Overview Page

The Overview page is the primary diagnostic view. It contains four panels:

**Top left — Score vs. Iteration**

Shows the training loss on the current minibatch at each iteration. This is the raw, per-minibatch value.

- Loss should generally trend downward.
- A consistently *increasing* loss suggests the learning rate is too high.
- A flat or nearly flat loss suggests the learning rate is too low, or the network is stuck.
- Large, irregular spikes may indicate exploding gradients.
- Very jagged plots (high variance between consecutive iterations) can be caused by a very small minibatch size or data that is not shuffled.

**Top right — Model and training information**

Displays the current epoch and iteration count, model architecture summary, and hardware info.

**Bottom left — Update:Parameter Ratio (by layer)**

Plots `log10( mean(|updates|) / mean(|params|) )` for each layer's weight matrix over time.

- A value of `-3` (ratio ≈ 0.001) is the commonly cited target.
- Values significantly above `-2` (ratio > 0.01) indicate updates are too large relative to parameters — try lowering the learning rate.
- Values below `-4` (ratio < 0.0001) indicate very slow learning — try raising the learning rate.
- Large sudden spikes indicate gradient explosions.

**Bottom right — Standard deviations over time**

Shows the log10 standard deviation of activations, gradients, and updates for each layer.

- Activations should be roughly in the range of 0.5 to 2.0 (log10 scale: approximately -0.3 to 0.3). Values outside this range may indicate poor weight initialization, wrong data normalization, or vanishing/exploding activations.

### Model Page

The Model page provides per-layer diagnostics. Click a layer in the network graph on the left to display its statistics on the right.

**Layer information table**

Shows the layer type, configuration, and parameter count.

**Update:Parameter ratio for the selected layer**

Same as the Overview page chart, but filtered to one layer. Tabs switch between displaying the ratio, mean parameter magnitude, and mean update magnitude separately.

**Layer activations over time**

Plots the mean activation and mean ± 2 standard deviations for the selected layer. This is the most direct indicator of vanishing or exploding activations:

- Activations that collapse to near zero: vanishing activations. Common causes: saturating activation functions (tanh/sigmoid) with poor initialization; too-large L2 regularization; learning rate too small.
- Activations that grow without bound: exploding activations. Causes: learning rate too large; poor initialization; missing batch normalization in deep networks.

**Parameters histogram**

Histogram of the current parameter values. After training for a while:

- Weight histograms should approximate a Gaussian distribution.
- Bias histograms start near zero and often remain approximately Gaussian; LSTM forget-gate biases start at 1.0 by design.
- Parameters diverging toward ±infinity: learning rate too large, or insufficient regularization.

**Updates histogram**

Histogram of the update values (after applying the updater). Should also approximate Gaussian. Very heavy tails indicate gradient explosions; update values stuck near zero indicate vanishing gradients or too small a learning rate.

**Parameter learning rates**

Shows the effective per-parameter learning rate over time. Useful when using learning rate schedules.

### System Page

Displays JVM heap usage, off-heap (native) memory usage, and garbage collection metrics over time. Useful for diagnosing out-of-memory conditions and memory leaks.

---

## Using the UI to Tune Hyperparameters

### Diagnosing Learning Rate Issues

The Score vs. Iteration and Update:Parameter ratio charts together tell you whether the learning rate needs adjustment:

| Symptom | Likely cause | Fix |
|---|---|---|
| Loss increases or oscillates wildly | Learning rate too high | Reduce by 5-10× |
| Loss decreases very slowly; ratio << -4 | Learning rate too low | Increase by 5-10× |
| Loss plateaus then improves with SGD | Stuck in local minimum | Switch to Adam, RMSProp, or Nesterovs |
| Loss decreases then diverges | Unstable at current LR | Reduce LR; add gradient clipping |

### Diagnosing Vanishing/Exploding Gradients

Use the standard deviation chart on the Overview page and the activations chart on the Model page:

- Activations converging to zero in early layers but not later: vanishing gradients. Solutions: use `RELU`/`LEAKYRELU` activations; use `XAVIER` or `RELU` weight initialization; add batch normalization; reduce network depth.
- Gradients growing across layers: exploding gradients. Solutions: add gradient clipping (`.gradientNormalization(GradientNormalization.ClipElementWiseAbsoluteValue).gradientNormalizationThreshold(1.0)`); reduce learning rate.

### Tips for Recurrent Networks

- Truncated backpropagation through time (TBPTT) should be configured when sequences are long. See the RNN guide.
- LSTM forget gate biases start at 1.0 — the parameters histogram will show a bimodal distribution initially; this is expected.
- For RNNs, monitor the updates histogram for each recurrent weight matrix separately (the Model page distinguishes `W` from `RW`).

---

## Spark Training and the Remote UI

When training on Spark, the network runs in a separate JVM (the Spark executor), while the UI server typically runs on a separate machine or the driver. Two patterns are available.

### Pattern 1: Collect Stats to File During Spark Training

```java
SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, conf, trainingMaster);

StatsStorage ss = new FileStatsStorage(new File("spark-stats.dl4j"));
sparkNet.setListeners(ss, Collections.singletonList(new StatsListener(null)));

sparkNet.fit(trainingData);
```

After training, load the file on any machine with a UI server:

```java
StatsStorage statsStorage = new FileStatsStorage(new File("spark-stats.dl4j"));
UIServer uiServer = UIServer.getInstance();
uiServer.attach(statsStorage);
```

### Pattern 2: Remote UI (Live Streaming)

Run the UI server on a dedicated machine (the "server"):

```java
UIServer uiServer = UIServer.getInstance();
uiServer.enableRemoteListener();  // must call this explicitly
```

On the Spark driver or executor (the "client"), use `RemoteUIStatsStorageRouter`:

```java
// Use deeplearning4j-ui-model dependency on the client, not the full UI
StatsStorageRouter remoteUIRouter =
    new RemoteUIStatsStorageRouter("http://UI_SERVER_IP:9000");

sparkNet.setListeners(remoteUIRouter,
    Collections.singletonList(new StatsListener(null)));
```

Replace `UI_SERVER_IP` with the IP address of the machine running `UIServer.getInstance()`.

---

## Troubleshooting: "No configuration setting" Exception

If you see:

```
com.typesafe.config.ConfigException$Missing: No configuration setting found
for key 'play.crypto.provider'
```

This is caused by a missing `application.conf` file from the Play framework (bundled inside the `deeplearning4j-ui` dependency). It occurs when assembling an uber-JAR with the Maven Assembly Plugin, which does not merge configuration files correctly.

**Solution**: use the Maven Shade Plugin with an `AppendingTransformer` for `reference.conf`:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <configuration>
        <filters>
            <filter>
                <artifact>*:*</artifact>
                <excludes>
                    <exclude>META-INF/*.SF</exclude>
                    <exclude>META-INF/*.DSA</exclude>
                    <exclude>META-INF/*.RSA</exclude>
                </excludes>
            </filter>
        </filters>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals><goal>shade</goal></goals>
            <configuration>
                <transformers>
                    <transformer implementation=
                        "org.apache.maven.plugins.shade.resource.AppendingTransformer">
                        <resource>reference.conf</resource>
                    </transformer>
                    <transformer implementation=
                        "org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                    <transformer implementation=
                        "org.apache.maven.plugins.shade.resource.ManifestResourceTransformer"/>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

---

## Part 2: t-SNE Visualization

[t-Distributed Stochastic Neighbor Embedding](https://lvdmaaten.github.io/tsne/) (t-SNE) is a dimensionality reduction technique for visualizing high-dimensional data in 2D or 3D. In DL4J it is most commonly used to:

- Visualize learned word embeddings from Word2Vec.
- Inspect the geometry of intermediate network activations.
- Verify that an embedding layer is separating classes in a meaningful way.

t-SNE is only meaningful for **labeled** data — the labels (color-coded in the plot) reveal whether similar inputs are clustering together in the embedding space.

### Maven Dependency

t-SNE functionality is in the DL4J NLP module:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-nlp</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

### t-SNE with Word2Vec Embeddings

```java
import org.deeplearning4j.models.embeddings.loader.WordVectorSerializer;
import org.deeplearning4j.models.word2vec.wordstore.VocabCache;
import org.deeplearning4j.models.embeddings.inmemory.InMemoryLookupTable;
import org.deeplearning4j.plot.BarnesHutTsne;
import org.nd4j.linalg.primitives.Pair;

// Load pre-trained word vectors (e.g., from Word2Vec.fit())
File wordVectorFile = new File("word-vectors.txt");
Pair<InMemoryLookupTable, VocabCache> vectors =
    WordVectorSerializer.loadTxt(wordVectorFile);

VocabCache  cache   = vectors.getSecond();
INDArray    weights = vectors.getFirst().getSyn0();

List<String> labels = new ArrayList<>();
for (int i = 0; i < cache.numWords(); i++)
    labels.add(cache.wordAtIndex(i));

// Build and run t-SNE
BarnesHutTsne tsne = new BarnesHutTsne.Builder()
    .setMaxIter(1000)          // number of optimization iterations
    .stopLyingIteration(250)   // iteration at which early exaggeration ends
    .learningRate(500)
    .useAdaGrad(false)
    .theta(0.5)                // Barnes-Hut approximation accuracy (0=exact, 1=fast)
    .setMomentum(0.5)
    .normalize(true)
    .usePca(false)             // pre-reduce with PCA before t-SNE if true
    .build();

// Project vocabulary into 2 dimensions; write coordinates to CSV
String outputPath = "target/tsne-coords.csv";
new File(outputPath).getParentFile().mkdirs();
tsne.plot(weights, 2, labels, outputPath);
```

The output CSV can be plotted with gnuplot, Python/matplotlib, or any other plotting tool. Each row is a word with its 2D coordinates and its label string.

### t-SNE on Arbitrary Activation Vectors

You can extract activations from any intermediate layer and pass them directly to t-SNE:

```java
// Collect activations from layer 2 across a test set
INDArray activations = net.feedForwardToLayer(2, testInput).get(2);
// activations shape: [numExamples, layerSize]

// Build label list from test labels (for coloring)
List<String> labelStrings = new ArrayList<>();
for (int i = 0; i < testLabels.rows(); i++)
    labelStrings.add(String.valueOf(testLabels.getRow(i).argMax(1).getInt(0)));

BarnesHutTsne tsne = new BarnesHutTsne.Builder()
    .setMaxIter(500)
    .theta(0.5)
    .normalize(false)
    .learningRate(200)
    .useAdaGrad(true)
    .build();

tsne.plot(activations, 2, labelStrings, "activations-tsne.csv");
```

### Tuning t-SNE

| Parameter | Effect |
|---|---|
| `setMaxIter` | More iterations gives a better layout but takes longer. 500–1000 is typical. |
| `stopLyingIteration` | Controls when early exaggeration (which helps clusters form) ends. Default 250. |
| `learningRate` | Typical values 100–1000. Too small: clusters don't spread. Too large: points explode. |
| `theta` | Barnes-Hut accuracy: 0.0 is exact (slow), 0.5 is standard. |
| `normalize` | Normalize input vectors before embedding. Usually `true` for word vectors. |
| `usePca` | Pre-reduce high-dimensional inputs (e.g., >50 dimensions) with PCA first. Recommended for faster convergence on large embedding matrices. |

### Interpreting t-SNE Plots

- Well-separated, tight clusters: the representation is meaningful for the task. Words or examples of the same class are near each other.
- Overlapping clouds: the representation is not discriminative. Try more training, a wider embedding dimension, or a different architecture.
- A single large blob: t-SNE may need more iterations, a different learning rate, or the activations may genuinely lack structure.

t-SNE is a stochastic algorithm. Run it multiple times and compare: if the cluster structure is consistent, the embedding is stable.

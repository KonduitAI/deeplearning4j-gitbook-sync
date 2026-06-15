---
title: "Training UI"
description: "Web-based training visualization — VertxUIServer, StatsListener, training dashboard, and t-SNE visualization"
---

## Visualize, Monitor, and Debug Neural Network Training

Deeplearning4j provides a browser-based training UI that displays real-time charts and statistics as your network trains. The UI is the primary tool for diagnosing training problems — choosing learning rates, detecting vanishing or exploding gradients, identifying ETL bottlenecks, and verifying that your network is learning as expected.

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-ui_2.11</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

The `_2.11` suffix refers to the Scala version required by the Play framework backend used by the UI. Either `_2.10` or `_2.11` works if you are not using other Scala libraries.

For Spark or remote training clients that only need to send stats without hosting the UI, use the lighter `deeplearning4j-ui-model` dependency instead:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-ui-model</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

---

## Setting Up the UI

### Step 1: Start the UI Server

```java
import org.deeplearning4j.ui.api.UIServer;
import org.deeplearning4j.ui.stats.StatsListener;
import org.deeplearning4j.ui.storage.InMemoryStatsStorage;
import org.deeplearning4j.core.storage.StatsStorage;

// Start the UI server (binds to port 9000 by default)
UIServer uiServer = UIServer.getInstance();

// Choose where to store training statistics
StatsStorage statsStorage = new InMemoryStatsStorage();

// Attach storage to the server so the UI can read from it
uiServer.attach(statsStorage);
```

### Step 2: Attach the StatsListener to Your Network

```java
// Add the listener — training stats are now collected and routed to the UI
net.setListeners(new StatsListener(statsStorage));
```

### Step 3: Train

```java
for (int epoch = 0; epoch < numEpochs; epoch++) {
    net.fit(trainData);
}
```

Navigate to `http://localhost:9000/train` in your browser. You will see the UI update in real time as each iteration completes.

To change the port, set the system property at JVM startup:

```
-Dorg.deeplearning4j.ui.port=9001
```

A complete working example is available in the examples repository:
[UIExample.java](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/userInterface/UIExample.java)

---

## Storage Backends

### InMemoryStatsStorage

The simplest option. Statistics are held in heap memory and are lost when the JVM exits. Use this for interactive training sessions where you are monitoring the run live.

```java
StatsStorage statsStorage = new InMemoryStatsStorage();
```

### FileStatsStorage

Statistics are written to disk and can be reloaded later, making this suitable for long training runs or headless servers.

```java
import org.deeplearning4j.ui.storage.FileStatsStorage;
import java.io.File;

// Write stats to a file during training
StatsStorage statsStorage = new FileStatsStorage(new File("training-stats.dl4j"));
net.setListeners(new StatsListener(statsStorage));

// Later, load the file and view it
StatsStorage loadedStats = new FileStatsStorage(new File("training-stats.dl4j"));
UIServer uiServer = UIServer.getInstance();
uiServer.attach(loadedStats);
```

---

## Web Dashboard Pages

The training UI presents three pages, accessible via tabs at the top of the dashboard.

### Overview Page

The overview page gives a high-level picture of training progress.

- **Score vs. Iteration (top left):** The value of the loss function on the current minibatch. Should decrease overall. If it increases consistently, your learning rate is too high. If it is flat or drops very slowly, the learning rate may be too low or an incompatible optimizer is being used.
- **Model and training information (top right):** Network type, number of parameters, and current training configuration.
- **Update:Parameter ratio chart (bottom left):** Ratio of mean magnitude of weight updates to mean magnitude of the parameters, plotted per layer on a log10 scale. A healthy value is around 10^-3 (i.e., -3 on the chart). Values consistently above -2 or below -4 indicate tuning is needed. Large spikes may indicate exploding gradients.
- **Standard deviations over time (bottom right):** Shows the standard deviation of activations, gradients, and updates across layers. Diverging trends here often indicate initialization or normalization issues.

### Model Page

The model page displays an interactive graph of your network's layers. Click on any layer to inspect it in detail.

Per-layer charts available after selection:

- **Layer information table:** Type, configuration, number of inputs/outputs.
- **Update:Parameter ratio:** Same ratio as the overview page, but for this layer only. Tabs show the raw parameter and update magnitudes separately.
- **Layer activations:** Mean activation value (and mean ± 2 standard deviations) over time. Healthy activations for most layers should stabilize with a standard deviation of roughly 0.5 to 2.0. Values outside this range suggest vanishing or exploding activations.
- **Parameter histograms:** Distribution of weights and biases for the most recent iteration. Weights should settle into an approximately Gaussian distribution. Biases should also become approximately Gaussian, except in LSTM forget gates (which are initialized to 1.0 by default). Watch for divergence to ±∞.
- **Update histograms:** Distribution of weight updates. Should also be approximately Gaussian. Large-magnitude outliers indicate exploding gradients.
- **Learning rate chart:** The current learning rate per parameter. Flat unless you are using a learning rate schedule.

### Performance Page

The performance page shows throughput metrics — examples per second and minibatches per second — allowing you to identify ETL bottlenecks. If GPU utilization is low but training is slow, the data pipeline is usually the bottleneck. See the [Benchmarking](../benchmarking.md) guide for remedies.

---

## Using the UI to Tune Your Network

The most common adjustments identified via the UI:

**Learning rate too high:** Score increases consistently or oscillates wildly. Reduce the learning rate by a factor of 3–10.

**Learning rate too low:** Score decreases very slowly. Increase the learning rate, or switch to an adaptive optimizer (Adam, RMSProp, Adagrad) rather than vanilla SGD.

**Exploding gradients:** Large spikes in the update:parameter ratio chart and in the update histogram. Add gradient normalization or gradient clipping to your configuration:
```java
.gradientNormalization(GradientNormalization.ClipElementWiseAbsoluteValue)
.gradientNormalizationThreshold(1.0)
```

**Vanishing activations:** Activation mean collapses toward zero over training. Reconsider weight initialization (try Xavier/Glorot or He initialization), add batch normalization, or use a non-saturating activation function such as ReLU.

**Class imbalance:** Output layer biases may grow very large if one class dominates. Consider oversampling the minority class or using a weighted loss function.

For more on interpreting training curves, see [Andrej Karpathy's neural network tuning guide](http://cs231n.github.io/neural-networks-3/#baby).

---

## Remote UI for Spark Training

Running the UI in the same JVM as Spark training creates dependency conflicts. Two options are available:

### Option A: Save Stats to a File and View Later

On the Spark master:

```java
SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, conf, tm);

StatsStorage ss = new FileStatsStorage(new File("myNetworkTrainingStats.dl4j"));
sparkNet.setListeners(ss, Collections.singletonList(new StatsListener(null)));
```

After training completes, view the file on any machine:

```java
StatsStorage statsStorage = new FileStatsStorage(new File("myNetworkTrainingStats.dl4j"));
UIServer uiServer = UIServer.getInstance();
uiServer.attach(statsStorage);
// Now open http://localhost:9000/train
```

### Option B: Remote UI Streaming

Run the UI in a separate JVM (the UI server):

```java
UIServer uiServer = UIServer.getInstance();
uiServer.enableRemoteListener();  // required — remote support is off by default
```

On the Spark master (client side, using `deeplearning4j-ui-model` dependency only):

```java
StatsStorageRouter remoteUIRouter =
        new RemoteUIStatsStorageRouter("http://UI_MACHINE_IP:9000");
sparkNet.setListeners(remoteUIRouter, Collections.singletonList(new StatsListener(null)));
```

Replace `UI_MACHINE_IP` with the IP address of the machine hosting the UI server. A complete example is at [RemoteUIExample.java](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/userInterface/RemoteUIExample.java).

---

## t-SNE Visualization

[t-Distributed Stochastic Neighbor Embedding](http://homepage.tudelft.nl/19j49/t-SNE.html) (t-SNE) is a dimensionality reduction technique that projects high-dimensional data into 2D or 3D for visualization. It is particularly useful for visualizing how well a model has learned to cluster semantically related inputs.

Common uses with DL4J:

- Visualizing word embedding spaces (e.g., from Word2Vec or GloVe) to confirm that semantically similar words cluster together.
- Plotting the activations of a penultimate layer to verify that a classifier has learned a meaningful representation before the output layer.

### Word Vector Visualization with t-SNE

```java
import org.deeplearning4j.plot.BarnesHutTsne;
import org.deeplearning4j.models.embeddings.inmemory.InMemoryLookupTable;
import org.deeplearning4j.models.word2vec.wordstore.VocabCache;

// Load pre-trained word vectors
Pair<InMemoryLookupTable, VocabCache> vectors =
        WordVectorSerializer.loadTxt(new ClassPathResource("words.txt").getFile());
VocabCache cache = vectors.getSecond();
INDArray weights = vectors.getFirst().getSyn0();

List<String> labels = new ArrayList<>();
for (int i = 0; i < cache.numWords(); i++) {
    labels.add(cache.wordAtIndex(i));
}

// Build and run t-SNE
BarnesHutTsne tsne = new BarnesHutTsne.Builder()
        .setMaxIter(1000)
        .stopLyingIteration(250)
        .learningRate(500)
        .useAdaGrad(false)
        .theta(0.5)
        .setMomentum(0.5)
        .normalize(true)
        .usePca(false)
        .build();

// Write 2D coordinates + labels to a CSV file
String outputFile = "target/tsne-coords.csv";
new File(outputFile).getParentFile().mkdirs();
tsne.plot(weights, 2, labels, outputFile);
```

The resulting CSV file can be plotted with gnuplot, Python/matplotlib, or any other plotting tool to produce a labeled scatter plot.

### Parameters

| Parameter | Method | Description |
|---|---|---|
| Max iterations | `setMaxIter(int)` | Number of optimization steps (100–1000 is typical) |
| Perplexity / theta | `theta(double)` | Barnes-Hut approximation quality (0.5 is a common default) |
| Stop lying iteration | `stopLyingIteration(int)` | When to stop the early exaggeration phase |
| Learning rate | `learningRate(double)` | Gradient step size (100–1000 typical) |
| Normalize | `normalize(boolean)` | Whether to normalize input vectors before running |

---

## Troubleshooting: "No configuration setting" Exception

If you see the following exception when starting the UI:

```
com.typesafe.config.ConfigException$Missing: No configuration setting found for key 'play.crypto.provider'
```

This is caused by a missing `application.conf` file from the Play framework, which is not copied correctly when building an uber-JAR with `maven-assembly-plugin`. Use the Maven Shade plugin instead, with an `AppendingTransformer` for `reference.conf`:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <configuration>
        <shadedArtifactAttached>true</shadedArtifactAttached>
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
                    <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                        <resource>reference.conf</resource>
                    </transformer>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer"/>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

Build with `mvn package` and run the resulting `-bin.jar` (the shaded artifact).

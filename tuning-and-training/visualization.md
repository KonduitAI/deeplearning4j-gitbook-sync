---
title: 'Visualize, Monitor and Debug Neural Network Learning'
short_title: Visualization
description: 'How to visualize, monitor and debug neural network learning.'
category: Tuning & Training
weight: 2
---

# Visualization

## Contents

* [Visualizing Network Training with the Deeplearning4j Training UI](visualization.md#visualizing-network-training-with-the-deeplearning-4-j-training-ui)
  * [Deeplearning4j UI: The Overview Page](visualization.md#deeplearning-4-j-ui-the-overview-page)
  * [Deeplearning4j UI: The Model Page](visualization.md#deeplearning-4-j-ui-the-model-page)
* [Deeplearning4J UI and Spark Training](visualization.md#deeplearning-4-j-ui-and-spark-training)
* [Using the UI to Tune Your Network](visualization.md#using-the-ui-to-tune-your-network)
* [TSNE and Word2Vec](visualization.md#tsne-and-word-2-vec)
* [Fixing UI Issue: "No configuration setting" exception](visualization.md#fixing-ui-issue-no-configuration-setting-exception)

## [Visualizing Network Training with the Deeplearning4j Training UI](visualization.md)

**Note**: This information here pertains to DL4J versions 1.0.0-beta6 and later.

DL4J Provides a user interface to visualize in your browser \(in real time\) the current network status and progress of training. The UI is typically used to help with tuning neural networks - i.e., the selection of hyperparameters \(such as learning rate\) to obtain good performance for a network.

**Step 1: Add the Deeplearning4j UI dependency to your project.**

```markup
    <dependency>
        <groupId>org.deeplearning4j</groupId>
        <artifactId>deeplearning4j-ui</artifactId>
        <version>{{ page.version }}</version>
    </dependency>
```

**Step 2: Enable the UI in your project**

This is relatively straightforward:

```java
    //Initialize the user interface backend
    UIServer uiServer = UIServer.getInstance();

    //Configure where the network information (gradients, score vs. time etc) is to be stored. Here: store in memory.
    StatsStorage statsStorage = new InMemoryStatsStorage();         //Alternative: new FileStatsStorage(File), for saving and loading later

    //Attach the StatsStorage instance to the UI: this allows the contents of the StatsStorage to be visualized
    uiServer.attach(statsStorage);

    //Then add the StatsListener to collect this information from the network, as it trains
    net.setListeners(new StatsListener(statsStorage));
```

To access the UI, open your browser and go to `http://localhost:9000/train/overview`. You can set the port by using the `org.deeplearning4j.ui.port` system property: i.e., to use port 9001, pass the following to the JVM on launch: `-Dorg.deeplearning4j.ui.port=9001`

Information will then be collected and routed to the UI when you call the `fit` method on your network.

**Example:** [See a UI example here](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/userInterface/UIExample.java)

The full set of UI examples are available [here](https://github.com/eclipse/deeplearning4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/userInterface).

### [Deeplearning4j UI: The Overview Page](visualization.md)

![](../.gitbook/assets/dl4j_ui_01%20%284%29%20%281%29.png)

The overview page \(one of 3 available pages\) contains the following information:

* Top left: score vs iteration chart - this is the value of the loss function on the current minibatch
* Top right: model and training information
* Bottom left: Ratio of parameters to updates \(by layer\) for all network weights vs. iteration
* Bottom right: Standard deviations \(vs. time\) of: activations, gradients and updates

Note that for the bottom two charts, these are displayed as the logarithm \(base 10\) of the values. Thus a value of -3 on the update: parameter ratio chart corresponds to a ratio of 10-3 = 0.001.

The ratio of updates to parameters is specifically the ratio of mean magnitudes of these values \(i.e., log10\(mean\(abs\(updates\)\)/mean\(abs\(parameters\)\)\).

See the later section of this page on how to use these values in practice.

### [Deeplearning4j UI: The Model Page](visualization.md)

![](../.gitbook/assets/dl4j_ui_02%20%284%29%20%284%29.png)

The model page contains a graph of the neural network layers, which operates as a selection mechanism. Click on a layer to display information for it.

On the right, the following charts are available, after selecting a layer:

* Table of layer information
* Update to parameter ratio for this layer, as per the overview page. The components of this ratio \(the parameter and update mean magnitudes\) are also available via tabs.
* Layer activations \(mean and mean +/- 2 standard deviations\) over time
* Histograms of parameters and updates, for each parameter type
* Learning rate vs. time \(note this will be flat, unless learning rate schedules are used\)

_Note: parameters are labeled as follows: weights \(W\) and biases \(b\). For recurrent neural networks, W refers to the weights connecting the layer to the layer below, and RW refers to the recurrent weights \(i.e., those between time steps\)._

## [Deeplearning4J UI and Spark Training](visualization.md)

The DL4J UI can be used with Spark. However, as of 0.7.0, conflicting dependencies mean that running the UI and Spark is the same JVM can be difficult.

Two alternatives are available:

1. Collect and save the relevant stats, to be visualized \(offline\) at a later point
2. Run the UI in a separate server, and Use the remote UI functionality to upload the data from the Spark master to your UI instance

**Collecting Stats for Later Offline Use**

```java
    SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, conf, tm);

    StatsStorage ss = new FileStatsStorage(new File("myNetworkTrainingStats.dl4j"));
    sparkNet.setListeners(ss, Collections.singletonList(new StatsListener(null)));
```

Then, later you can load and display the saved information using:

```java
    StatsStorage statsStorage = new FileStatsStorage(statsFile);    //If file already exists: load the data from it
    UIServer uiServer = UIServer.getInstance();
    uiServer.attach(statsStorage);
```

**Using the Remote UI Functionality**

First, in the JVM running the UI \(note this is the server\):

```java
    UIServer uiServer = UIServer.getInstance();
    uiServer.enableRemoteListener();        //Necessary: remote support is not enabled by default
```

This will require the `deeplearning4j-ui` dependency. \(NOTE THIS IS NOT THE CLIENT THIS IS YOUR SERVER - SEE BELOW FOR THE CLIENT WHICH USES: deeplearning4j-ui-model\)

Client \(both spark and standalone neural networks using simple deeplearning4j-nn\) Second, for your neural net \(Note this example is for spark, but computation graph and multi layer network both have the equivalemtn setListeners method with the same usage, [example found here](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/userInterface/RemoteUIExample.java)\):

```java
    SparkDl4jMultiLayer sparkNet = new SparkDl4jMultiLayer(sc, conf, tm);

    StatsStorageRouter remoteUIRouter = new RemoteUIStatsStorageRouter("http://UI_MACHINE_IP:9000");
    sparkNet.setListeners(remoteUIRouter, Collections.singletonList(new StatsListener(null)));
```

To avoid dependency conflicts with Spark, you should use the `deeplearning4j-ui-model` dependency to get the StatsListener, _not_ the full `deeplearning4j-ui` UI dependency.

Note: you should replace `UI_MACHINE_IP` with the IP address of the machine running the user interface instance.

## [Using the UI to Tune Your Network](visualization.md)

Here's an excellent [web page by Andrej Karpathy](http://cs231n.github.io/neural-networks-3/#baby) about visualizing neural net training. It is worth reading and understanding that page first.

Tuning neural networks is often more an art than a science. However, here's some ideas that may be useful:

**Overview Page - Model Score vs. Iteration Chart**

The score vs. iteration should \(overall\) go down over time.

* If the score increases consistently, your learning rate is likely set too high. Try reducing it until scores become more stable.
* Increasing scores can also be indicative of other network issues, such as incorrect data normalization
* If the score is flat or decreases very slowly \(over a few hundred iterations\) \(a\) your learning rate may be too low, or \(b\) you might be having difficulties with optimization. In the latter case, if you are using the SGD updater, try a different updater such as Nesterovs \(momentum\), RMSProp or Adagrad.
* Note that data that isn't shuffled \(i.e., each minibatch contains only one class, for classification\) can result in very rough or abnormal-looking score vs. iteration graphs
* Some noise in this line chart is expected \(i.e., the line will go up and down within a small range\). However, if the scores vary quite significantly between runs variation is very large, this can be a problem
  * The issues mentioned above \(learning rate, normalization, data shuffling\) may contribute to this.
  * Setting the minibatch size to a very small number of examples can also contribute to noisy score vs. iteration graphs, and _might_ lead to optimization difficulties

**Overview Page and Model Page - Using the Update: Parameter Ratio Chart**

* The ratio of mean magnitude of updates to parameters is provided on both the overview and model pages
  * "Mean magnitude" = the average of the absolute value of the parameters or updates at the current time step
* The most important use of this ratio is in selecting a learning rate. As a rule of thumb: this ratio should be around 1:1000 = 0.001. On the \(log10\) chart, this corresponds to a value of -3 \(i.e., 10-3 = 0.001\)
  * Note that is a rough guide only, and may not be appropriate for all networks. It's often a good starting point, however.
  * If the ratio diverges significantly from this \(for example, &gt; -2 \(i.e., 10-2=0.01\) or &lt; -4 \(i.e., 10-4=0.0001\), your parameters may be too unstable to learn useful features, or may change too slowly to learn useful features
  * To change this ratio, adjust your learning rate \(or sometimes, parameter initialization\). In some networks, you may need to set the learning rate differently for different layers.
* Keep an eye out for unusually large spikes in the ratio: this may indicate exploding gradients

**Model Page: Layer Activations \(vs. Time\) Chart**

This chart can be used to detect vanishing or exploding activations \(due to poor weight initialization, too much regularization, lack of data normalization, or too high a learning rate\).

* This chart should ideally stabilize over time \(usually a few hundred iterations\)
* A good standard deviation for the activations is on the order of 0.5 to 2.0. Significantly outside of this range may indicate one of the problems mentioned above.

**Model Page: Layer Parameters Histogram**

The layer parameters histogram is displayed for the most recent iteration only.

* For weights, these histograms should  have an approximately Gaussian \(normal\) distribution, after some time
* For biases, these histograms will generally start at 0, and will usually end up being approximately Gaussian
  * One exception to this is for LSTM recurrent neural network layers: by default, the biases for one gate \(the forget gate\) are set to 1.0 \(by default, though this is configurable\), to help in learning dependencies across long time periods. This results in the bias graphs initially having many biases around 0.0, with another set of biases around 1.0
* Keep an eye out for parameters that are diverging to +/- infinity: this may be due to too high a learning rate, or insufficient regularization \(try adding some L2 regularization to your network\).
* Keep an eye out for biases that become very large. This can sometimes occur in the output layer for classification, if the distribution of classes is very imbalanced

**Model Page: Layer Updates Histogram**

The layer update histogram is displayed for the most recent iteration only.

* Note that these are the updates - i.e., the gradients _after_ applying learning rate, momentum, regularization etc
* As with the parameter graphs, these should have an approximately Gaussian \(normal\) distribution
* Keep an eye out for very large values: this can indicate exploding gradients in your network
  * Exploding gradients are problematic as they can 'mess up' the parameters of your network
  * In this case, it may indicate a weight initialization, learning rate or input/labels data normalization issue
  * In the case of recurrent neural networks, adding some [gradient normalization or gradient clipping](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/GradientNormalization.java) may help

**Model Page: Parameter Learning Rates Chart**

This chart simply shows the learning rates of the parameters of selected layer, over time.

If you are not using learning rate schedules, the chart will be flat. If you _are_ using learning rate schedules, you can use this chart to track the current value of the learning rate \(for each parameter\), over time.

## [TSNE and Word2vec](visualization.md)

We rely on [TSNE](https://lvdmaaten.github.io/tsne/) to reduce the dimensionality of [word feature vectors](../language-processing/word2vec.md) and project words into a two or three-dimensional space. Here's some code for using TSNE with Word2Vec:

```java
log.info("Plot TSNE....");
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
vec.lookupTable().plotVocab(tsne);
```

## [Fixing UI Issue: "No configuration setting" exception](visualization.md)

A possible exception that can occur with the DL4J UI is the following:

```text
com.typesafe.config.ConfigException$Missing: No configuration setting found for key 'play.crypto.provider'
        at com.typesafe.config.impl.SimpleConfig.findKeyOrNull(SimpleConfig.java:152)
        at com.typesafe.config.impl.SimpleConfig.findOrNull(SimpleConfig.java:170)
        ...
        at play.server.Server.forRouter(Server.java:96)
        at org.deeplearning4j.ui.play.PlayUIServer.runMain(PlayUIServer.java:206)
        at org.deeplearning4j.ui.api.UIServer.getInstance(UIServer.java:27)
```

This exception is not due to DL4J directly, but is due to a missing application.conf file, required by the Play framework \(the library that DL4J's UI is based on\). This is originally present in the deeplearning4j-play dependency: however, if an uber-jar \(i.e., a JAR file with dependencies\) is built \(say, via `mvn package`\), it may not be copied over correctly. For example, using the `maven-assembly-plugin` has caused this exception for some users.

The recommended solution \(for Maven\) is to use the Maven Shade plugin to produce an uber-jar, configured as follows:

```markup
    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>${exec-maven-plugin.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>exec</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <executable>java</executable>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>${maven-shade-plugin.version}</version>
                <configuration>
                    <shadedArtifactAttached>true</shadedArtifactAttached>
                    <shadedClassifierName>${shadedClassifier}</shadedClassifierName>
                    <createDependencyReducedPom>true</createDependencyReducedPom>
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <!--<exclude>org/datanucleus/**</exclude>-->
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
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                    <resource>reference.conf</resource>
                                </transformer>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer" />
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        <plugins>
    <build>
```

Then, create your uber-jar with `mvn package` and run via `cd target && java -cp dl4j-examples-0.9.1-bin.jar org.deeplearning4j.examples.userInterface.UIExample`. Note the "-bin" suffix for the generated JAR file: this includes all dependencies.

Note also that this Maven Shade approach is configured for DL4J's examples repository.


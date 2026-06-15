---
title: "Deeplearning4j Quickstart"
description: "End-to-end quickstart guide — from Maven setup to training an MNIST classifier in Deeplearning4j"
---

# Deeplearning4j Quickstart

This guide takes you from zero to a trained MNIST digit classifier in a single self-contained Java project. By the end you will have:

- A working Maven project with the correct M2.1 dependencies
- A complete `MultiLayerNetwork` that classifies MNIST handwritten digits
- A training loop with evaluation and score logging
- A saved model you can reload later

No prior DL4J experience is assumed. You do need Java 11+ and Apache Maven installed.

---

## Prerequisites

**Java 11 or later (64-bit)**

```shell
java -version
```

You need a 64-bit JVM. If you see `no jnind4j in java.library.path` at runtime you are almost certainly running a 32-bit JVM.

**Apache Maven 3.6+**

```shell
mvn --version
```

If you are on macOS with Homebrew:

```shell
brew install maven
```

**An IDE (recommended)**

IntelliJ IDEA Community Edition works best because it has first-class Maven support. Eclipse and VS Code work too.

---

## Step 1 — Create a Maven Project

Create a new Maven project, then replace the generated `pom.xml` with the one below (or add the relevant sections to your existing one).

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>dl4j-mnist-quickstart</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>11</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <dl4j.version>1.0.0-M2.1</dl4j.version>
    </properties>

    <dependencies>
        <!-- Core DL4J library -->
        <dependency>
            <groupId>org.deeplearning4j</groupId>
            <artifactId>deeplearning4j-core</artifactId>
            <version>${dl4j.version}</version>
        </dependency>

        <!-- MNIST dataset loader (ships with DL4J) -->
        <dependency>
            <groupId>org.deeplearning4j</groupId>
            <artifactId>deeplearning4j-datasets</artifactId>
            <version>${dl4j.version}</version>
        </dependency>

        <!-- UI and training visualization (optional but useful) -->
        <dependency>
            <groupId>org.deeplearning4j</groupId>
            <artifactId>deeplearning4j-ui</artifactId>
            <version>${dl4j.version}</version>
        </dependency>

        <!-- CPU backend — works on any platform without a GPU -->
        <dependency>
            <groupId>org.nd4j</groupId>
            <artifactId>nd4j-native-platform</artifactId>
            <version>${dl4j.version}</version>
        </dependency>

        <!-- Logging -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.11</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Shade plugin makes a fat JAR for easy execution -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.4.1</version>
                <configuration>
                    <shadedArtifactAttached>false</shadedArtifactAttached>
                    <createDependencyReducedPom>false</createDependencyReducedPom>
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
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

### Dependency notes

| Artifact | Purpose |
|----------|---------|
| `deeplearning4j-core` | `MultiLayerNetwork`, `ComputationGraph`, all layer types |
| `deeplearning4j-datasets` | Built-in dataset downloaders including MNIST |
| `deeplearning4j-ui` | Web-based training visualization UI (port 9000) |
| `nd4j-native-platform` | CPU math backend with pre-built natives for Linux, macOS, Windows |
| `logback-classic` | Logging backend required by DL4J's SLF4J calls |

**GPU alternative:** Replace `nd4j-native-platform` with `nd4j-cuda-11.8-platform` (or your CUDA version) and add `deeplearning4j-cuda-11.8`. The rest of the code stays identical.

Install dependencies:

```shell
mvn dependency:resolve
```

---

## Step 2 — Write the MNIST Classifier

Create the file `src/main/java/com/example/MnistClassifier.java` with the content below.

```java
package com.example;

import org.deeplearning4j.datasets.iterator.impl.MnistDataSetIterator;
import org.deeplearning4j.nn.conf.MultiLayerConfiguration;
import org.deeplearning4j.nn.conf.NeuralNetConfiguration;
import org.deeplearning4j.nn.conf.layers.DenseLayer;
import org.deeplearning4j.nn.conf.layers.OutputLayer;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.deeplearning4j.nn.weights.WeightInit;
import org.deeplearning4j.optimize.listeners.ScoreIterationListener;
import org.deeplearning4j.util.ModelSerializer;
import org.nd4j.evaluation.classification.Evaluation;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.api.buffer.DataType;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;
import org.nd4j.linalg.learning.config.Adam;
import org.nd4j.linalg.lossfunctions.LossFunctions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;

public class MnistClassifier {

    private static final Logger log = LoggerFactory.getLogger(MnistClassifier.class);

    // -------------------------------------------------------------------
    // Hyperparameters
    // -------------------------------------------------------------------
    private static final int SEED          = 123;
    private static final int BATCH_SIZE    = 64;
    private static final int NUM_EPOCHS    = 5;
    private static final int NUM_INPUTS    = 784;   // 28 x 28 pixels flattened
    private static final int NUM_HIDDEN_1  = 512;
    private static final int NUM_HIDDEN_2  = 256;
    private static final int NUM_OUTPUTS   = 10;    // digits 0-9
    private static final double LEARNING_RATE = 1e-3;
    private static final double L2_LAMBDA     = 1e-4;

    public static void main(String[] args) throws Exception {

        // ---------------------------------------------------------------
        // 1. Load MNIST
        //    MnistDataSetIterator downloads the dataset on first run
        //    (~12 MB) and caches it in ~/.deeplearning4j/
        // ---------------------------------------------------------------
        log.info("Loading MNIST dataset...");
        DataSetIterator trainIter = new MnistDataSetIterator(BATCH_SIZE, true,  SEED);
        DataSetIterator testIter  = new MnistDataSetIterator(BATCH_SIZE, false, SEED);

        // ---------------------------------------------------------------
        // 2. Configure the network
        // ---------------------------------------------------------------
        log.info("Building network configuration...");
        MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
            .seed(SEED)
            // M2.1: use DataType.FLOAT, not the old setDataType() approach
            .dataType(DataType.FLOAT)
            // M2.1: pass an IUpdater instance — new Adam(lr), not the
            // deprecated .updater(Updater.ADAM).learningRate() chain
            .updater(new Adam(LEARNING_RATE))
            // L2 regularization applied globally to all layers
            .l2(L2_LAMBDA)
            // Global weight initializer (can be overridden per layer)
            .weightInit(WeightInit.XAVIER)
            .list()
            // Hidden layer 1: 784 → 512
            .layer(new DenseLayer.Builder()
                .nIn(NUM_INPUTS)
                .nOut(NUM_HIDDEN_1)
                .activation(Activation.RELU)
                .build())
            // Hidden layer 2: 512 → 256
            .layer(new DenseLayer.Builder()
                .nIn(NUM_HIDDEN_1)
                .nOut(NUM_HIDDEN_2)
                .activation(Activation.RELU)
                .build())
            // Output layer: 256 → 10 with softmax + cross-entropy
            // M2.1: no .pretrain(false).backprop(true) needed — these are
            // no-ops / removed in M2.1
            .layer(new OutputLayer.Builder(
                        LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
                .nIn(NUM_HIDDEN_2)
                .nOut(NUM_OUTPUTS)
                .activation(Activation.SOFTMAX)
                .build())
            .build();

        // ---------------------------------------------------------------
        // 3. Initialize the model
        // ---------------------------------------------------------------
        MultiLayerNetwork model = new MultiLayerNetwork(conf);
        model.init();

        // Print the number of trainable parameters
        log.info("Number of parameters: {}", model.numParams());

        // ScoreIterationListener prints the loss every N mini-batches
        model.setListeners(new ScoreIterationListener(100));

        // ---------------------------------------------------------------
        // 4. Train
        // ---------------------------------------------------------------
        log.info("Starting training for {} epochs...", NUM_EPOCHS);
        for (int epoch = 1; epoch <= NUM_EPOCHS; epoch++) {
            model.fit(trainIter);
            trainIter.reset();

            // Evaluate on the test set at the end of each epoch
            Evaluation eval = model.evaluate(testIter);
            testIter.reset();

            log.info("--- Epoch {} ---", epoch);
            log.info("Accuracy:  {}", eval.accuracy());
            log.info("Precision: {}", eval.precision());
            log.info("Recall:    {}", eval.recall());
            log.info("F1 Score:  {}", eval.f1());
        }

        // ---------------------------------------------------------------
        // 5. Full evaluation on the test set
        // ---------------------------------------------------------------
        log.info("\nFinal evaluation on test set:");
        Evaluation finalEval = model.evaluate(testIter);
        log.info(finalEval.stats());

        // ---------------------------------------------------------------
        // 6. Save the model
        // ---------------------------------------------------------------
        File modelFile = new File("mnist-model.zip");
        ModelSerializer.writeModel(model, modelFile, true);
        log.info("Model saved to {}", modelFile.getAbsolutePath());
    }
}
```

---

## Step 3 — Build and Run

```shell
mvn clean package -q
java -cp target/dl4j-mnist-quickstart-1.0-SNAPSHOT.jar com.example.MnistClassifier
```

On first run, DL4J downloads the MNIST binary files (~12 MB) and caches them. Subsequent runs use the cache.

Expected training output (times vary by machine):

```
[main] INFO MnistClassifier - Loading MNIST dataset...
[main] INFO MnistClassifier - Building network configuration...
[main] INFO MnistClassifier - Number of parameters: 535,818
[main] INFO ScoreIterationListener - Score at iteration 100 is 0.3241
[main] INFO ScoreIterationListener - Score at iteration 200 is 0.1987
...
[main] INFO MnistClassifier - --- Epoch 1 ---
[main] INFO MnistClassifier - Accuracy:  0.9712
...
[main] INFO MnistClassifier - --- Epoch 5 ---
[main] INFO MnistClassifier - Accuracy:  0.9831
```

A well-configured MLP on MNIST reaches 97-98% accuracy within five epochs on the CPU. If you are seeing much lower accuracy, check the troubleshooting section below.

---

## Understanding the Configuration

### M2.1 API Differences from Older Versions

If you are migrating from DL4J 1.0.0-beta4 or earlier, the key API changes are:

**Updaters:** The enum-based updater API is removed. Use updater class instances:

```java
// M2.1 (correct)
.updater(new Adam(1e-3))
.updater(new Sgd(0.01))
.updater(new RmsProp(1e-3))
.updater(new AdaGrad(0.1))
.updater(new Nesterovs(0.01, 0.9))

// Old API (do not use)
// .updater(Updater.ADAM).learningRate(1e-3)
// .updater(Updater.SGD).learningRate(0.01)
```

**DataType:** Set it explicitly on the builder:

```java
// M2.1 (correct)
.dataType(DataType.FLOAT)

// M2.1 also supports DOUBLE, HALF, BFLOAT16:
.dataType(DataType.DOUBLE)
```

**pretrain / backprop flags:** These were removed. Standard supervised training always uses backpropagation. Remove any `.pretrain(false).backprop(true)` calls — they will cause a compilation error or warning.

**Layer index:** In M2.1 you can omit the integer index when adding layers with `.layer(LayerBuilder)` and they are added in order. The old ``.layer(0, new DenseLayer...)`` style still compiles but the index is redundant for `MultiLayerNetwork`.

### Network Architecture Explained

The example uses a simple **multilayer perceptron (MLP)**:

```
Input (784)  →  Dense-ReLU (512)  →  Dense-ReLU (256)  →  Softmax Output (10)
```

- **Input:** MNIST images are 28x28 = 784 pixels, flattened to a 1D vector
- **Hidden layers:** `DenseLayer` with ReLU activation learns non-linear representations
- **Output:** `OutputLayer` with softmax produces a probability distribution over 10 classes; negative log-likelihood (cross-entropy) is the loss function
- **Xavier initialization:** Keeps activation variance stable at initialization, important for deep networks
- **Adam optimizer:** Adaptive learning rate per parameter; usually converges faster than plain SGD
- **L2 regularization:** Penalizes large weights to reduce overfitting

---

## Step 4 — Load and Use the Saved Model

```java
import org.deeplearning4j.util.ModelSerializer;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

// Reload a previously saved model
MultiLayerNetwork loaded = ModelSerializer.restoreMultiLayerNetwork(
    new File("mnist-model.zip"));

// Create a fake 28x28 image (all zeros) as a row vector [1, 784]
INDArray input = Nd4j.zeros(DataType.FLOAT, 1, 784);

// Run forward pass — output shape is [1, 10]
INDArray output = loaded.output(input);

// argmax gives the predicted class
int predictedClass = output.argMax(1).getInt(0);
System.out.println("Predicted digit: " + predictedClass);
```

`ModelSerializer` stores both the network configuration and the trained weights in a single `.zip` file. The normalizer (if you used one) can also be saved alongside:

```java
// Save with normalizer
ModelSerializer.writeModel(model, modelFile, true, normalizer);

// Restore with normalizer
NormalizerStandardize restoredNorm = ModelSerializer.restoreNormalizerFromFile(modelFile);
```

---

## Adding a Convolutional Network (Optional)

For images, a convolutional network significantly outperforms an MLP. Here is the configuration change — everything else (training loop, saving) stays the same:

```java
import org.deeplearning4j.nn.conf.inputs.InputType;
import org.deeplearning4j.nn.conf.layers.ConvolutionLayer;
import org.deeplearning4j.nn.conf.layers.SubsamplingLayer;
import org.deeplearning4j.nn.conf.layers.SubsamplingLayer.PoolingType;

MultiLayerConfiguration cnnConf = new NeuralNetConfiguration.Builder()
    .seed(SEED)
    .dataType(DataType.FLOAT)
    .updater(new Adam(LEARNING_RATE))
    .l2(L2_LAMBDA)
    .weightInit(WeightInit.XAVIER)
    .list()
    // Conv block 1
    .layer(new ConvolutionLayer.Builder(5, 5)
        .nIn(1)          // 1 channel (grayscale)
        .nOut(32)
        .stride(1, 1)
        .activation(Activation.RELU)
        .build())
    .layer(new SubsamplingLayer.Builder(PoolingType.MAX)
        .kernelSize(2, 2)
        .stride(2, 2)
        .build())
    // Conv block 2
    .layer(new ConvolutionLayer.Builder(5, 5)
        .nOut(64)
        .stride(1, 1)
        .activation(Activation.RELU)
        .build())
    .layer(new SubsamplingLayer.Builder(PoolingType.MAX)
        .kernelSize(2, 2)
        .stride(2, 2)
        .build())
    // Dense + output
    .layer(new DenseLayer.Builder()
        .nOut(512)
        .activation(Activation.RELU)
        .build())
    .layer(new OutputLayer.Builder(
                LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nOut(NUM_OUTPUTS)
        .activation(Activation.SOFTMAX)
        .build())
    // Tell DL4J the input shape so it can auto-infer nIn for conv layers
    .setInputType(InputType.convolutionalFlat(28, 28, 1))
    .build();
```

This CNN typically reaches 99%+ accuracy on MNIST within five epochs.

---

## Troubleshooting

**`NoAvailableBackendException` at startup**

The ND4J backend JAR is missing from the classpath. Confirm `nd4j-native-platform` is in your `pom.xml` and that `mvn dependency:resolve` completed without errors. If you are running from an IDE, reimport the Maven project.

**`no jnind4j in java.library.path`**

You are running a 32-bit JVM. Install a 64-bit JDK. Check with `java -d64 -version`.

**Very low accuracy (below 80% after epoch 1)**

Common causes:
- Data not normalized — MNIST loaded via `MnistDataSetIterator` is normalized to [0, 1] automatically, but if you load your own images you need a `DataNormalization` preprocessor
- Learning rate too high or too low — try values between `1e-4` and `1e-2`
- Batch size too small — try 32 to 256

**`OutOfMemoryError` (heap)**

Increase JVM heap: `java -Xmx4g -cp ...`

DL4J allocates tensor data off-heap (in native memory). If you see off-heap OOM errors, set `Nd4j.getMemoryManager().setAutoGcWindow(5000)` to trigger GC more frequently, or tune the off-heap limit with `-Dorg.bytedeco.javacpp.maxbytes=2G`.

**Windows: `UnsatisfiedLinkError`**

Conflicting native DLLs on `PATH`. Add `-Djava.library.path=""` to VM options in your IDE run configuration.

---

## Next Steps

- **Core Concepts:** Read [Core Concepts](concepts.md) to understand `MultiLayerNetwork` vs `ComputationGraph`, the training pipeline, and the ND4J relationship
- **More examples:** Clone the [DL4J examples repository](https://github.com/eclipse/deeplearning4j-examples) for CNNs, RNNs, transfer learning, and more
- **Custom layers:** See the [custom layers guide](../nn/custom-layers.md)
- **Training on Spark:** See the [distributed training guide](../scaleout/spark.md)
- **Keras import:** Trained a model in Keras? Import it with the [Keras model import guide](../../model-import/keras.md)
- **Hyperparameter tuning:** Use [Arbiter](../../arbiter/overview.md) for automated hyperparameter search
- **API reference:** Browse the [Deeplearning4j Javadoc](/api/latest/)

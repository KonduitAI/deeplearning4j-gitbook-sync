---
title: Arbiter Visualization
description: Monitoring Arbiter hyperparameter optimization runs with ArbiterStatusListener and the UIServer.
---

## Overview

Arbiter provides a web-based UI for monitoring hyperparameter optimization runs in real time. The UI shows:

- All candidate configurations evaluated so far and their scores
- Score history across candidates
- Hyperparameter value distributions for evaluated candidates
- Best candidate information
- Optimization run status (running, complete, failed)

The UI is served by the same `UIServer` used for DL4J training visualization (via `deeplearning4j-ui`). The Arbiter-specific content is served at:

```
http://localhost:9000/arbiter
```

---

## Dependencies

The UI requires the `arbiter-ui` artifact in addition to `arbiter-deeplearning4j`:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>arbiter-deeplearning4j</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>

<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>arbiter-ui_2.11</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

The `arbiter-ui` artifact depends on `deeplearning4j-ui_2.11`. If you already have the DL4J UI dependency, you do not need to add it again.

---

## ArbiterStatusListener

`ArbiterStatusListener` is the bridge between the optimization runner and the UIServer. Attach it to the runner before calling `execute()`:

```java
// Start the UIServer (port 9000 by default)
UIServer uiServer = UIServer.getInstance();

// Create a StatsStorage backend
StatsStorage statsStorage = new InMemoryStatsStorage();

// Register the storage backend with the UI
uiServer.attach(statsStorage);

// Add the listener to the optimization runner
runner.addListeners(new ArbiterStatusListener(statsStorage));

// Run optimization — the UI will update as candidates are evaluated
runner.execute();
```

After calling `runner.execute()`, open a browser and navigate to `http://localhost:9000/arbiter` to see the live results.

---

## StatsStorage Backends

### InMemoryStatsStorage

Stores stats in JVM memory. Fast and requires no external setup, but data is lost when the JVM exits. Suitable for interactive experimentation.

```java
StatsStorage statsStorage = new InMemoryStatsStorage();
```

### FileStatsStorage

Persists stats to disk. Data survives JVM restarts, allowing you to resume monitoring or post-hoc analysis:

```java
StatsStorage statsStorage = new FileStatsStorage(new File("/tmp/arbiter_stats.dl4j"));
```

To restore and attach a previously saved stats file to the UI without re-running optimization:

```java
StatsStorage stored = new FileStatsStorage(new File("/tmp/arbiter_stats.dl4j"));
UIServer.getInstance().attach(stored);
// navigate to http://localhost:9000/arbiter to view historical results
```

---

## Full Example

```java
import org.deeplearning4j.api.storage.StatsStorage;
import org.deeplearning4j.arbiter.MultiLayerSpace;
import org.deeplearning4j.arbiter.conf.updater.AdamSpace;
import org.deeplearning4j.arbiter.layers.DenseLayerSpace;
import org.deeplearning4j.arbiter.layers.OutputLayerSpace;
import org.deeplearning4j.arbiter.optimize.api.CandidateGenerator;
import org.deeplearning4j.arbiter.optimize.api.OptimizationResult;
import org.deeplearning4j.arbiter.optimize.api.ParameterSpace;
import org.deeplearning4j.arbiter.optimize.api.score.ScoreFunction;
import org.deeplearning4j.arbiter.optimize.api.termination.MaxCandidatesCondition;
import org.deeplearning4j.arbiter.optimize.api.termination.MaxTimeCondition;
import org.deeplearning4j.arbiter.optimize.api.termination.TerminationCondition;
import org.deeplearning4j.arbiter.optimize.config.OptimizationConfiguration;
import org.deeplearning4j.arbiter.optimize.generator.RandomSearchGenerator;
import org.deeplearning4j.arbiter.optimize.parameter.continuous.ContinuousParameterSpace;
import org.deeplearning4j.arbiter.optimize.parameter.integer.IntegerParameterSpace;
import org.deeplearning4j.arbiter.optimize.runner.IOptimizationRunner;
import org.deeplearning4j.arbiter.optimize.runner.LocalOptimizationRunner;
import org.deeplearning4j.arbiter.saver.local.FileModelSaver;
import org.deeplearning4j.arbiter.scoring.impl.EvaluationScoreFunction;
import org.deeplearning4j.arbiter.task.MultiLayerNetworkTaskCreator;
import org.deeplearning4j.arbiter.ui.listener.ArbiterStatusListener;
import org.deeplearning4j.eval.Evaluation;
import org.deeplearning4j.ui.api.UIServer;
import org.deeplearning4j.ui.stats.StatsListener;
import org.deeplearning4j.ui.storage.InMemoryStatsStorage;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.lossfunctions.LossFunctions;
import java.util.Properties;
import java.util.concurrent.TimeUnit;

public class ArbiterUIExample {
    public static void main(String[] args) throws Exception {

        // 1. Define search space
        MultiLayerSpace searchSpace = new MultiLayerSpace.Builder()
            .seed(42)
            .updater(new AdamSpace(new ContinuousParameterSpace(1e-4, 1e-2)))
            .addLayer(new DenseLayerSpace.Builder()
                .nIn(784)
                .nOut(new IntegerParameterSpace(64, 256))
                .activation(Activation.RELU)
                .build())
            .addLayer(new OutputLayerSpace.Builder()
                .nOut(10)
                .activation(Activation.SOFTMAX)
                .lossFunction(LossFunctions.LossFunction.MCXENT)
                .build())
            .numEpochs(5)
            .build();

        // 2. Candidate generator
        CandidateGenerator candidateGenerator = new RandomSearchGenerator(searchSpace);

        // 3. Optimization configuration
        Properties dataProps = new Properties();
        dataProps.setProperty("batchSize", "64");

        OptimizationConfiguration configuration = new OptimizationConfiguration.Builder()
            .candidateGenerator(candidateGenerator)
            .dataSource(MnistDataSource.class, dataProps)
            .modelSaver(new FileModelSaver("/tmp/arbiter_models/"))
            .scoreFunction(new EvaluationScoreFunction(Evaluation.Metric.ACCURACY))
            .terminationConditions(
                new MaxTimeCondition(15, TimeUnit.MINUTES),
                new MaxCandidatesCondition(10)
            )
            .build();

        // 4. Set up UI
        UIServer uiServer = UIServer.getInstance();
        StatsStorage statsStorage = new InMemoryStatsStorage();
        uiServer.attach(statsStorage);

        // 5. Create and configure runner
        IOptimizationRunner runner = new LocalOptimizationRunner(
            configuration,
            new MultiLayerNetworkTaskCreator()
        );
        runner.addListeners(new ArbiterStatusListener(statsStorage));

        // 6. Run — open http://localhost:9000/arbiter to monitor
        System.out.println("UI: http://localhost:9000/arbiter");
        runner.execute();

        // 7. Print best result
        System.out.println(runner.toSummaryString());
    }
}
```

---

## What the Arbiter UI Shows

### Candidates Table

A sortable table of all evaluated candidates showing:

- Candidate index
- Score (from the score function)
- Training duration
- Number of parameters
- Status (complete, running, failed)

Click a row to expand the hyperparameter details for that candidate.

### Score vs. Candidate Index

A chart showing how the best score (and the current candidate's score) changes as more candidates are evaluated. An upward trend indicates the search is finding progressively better configurations.

### Hyperparameter Distribution

For each hyperparameter in the search space, a visualization of the values that have been sampled and the corresponding scores. This makes it easy to see whether the search has found a concentration of good values in a particular region.

### Best Candidate

A summary panel showing the hyperparameter configuration of the current best candidate, its score, and a link to download the model (when using `FileModelSaver`).

---

## Using Both Arbiter UI and DL4J Training UI Simultaneously

Both UIs are served by the same `UIServer` instance. If you also want per-candidate training curves (loss by iteration), add a `StatsListener` to each candidate model during training. This requires a custom `TaskCreator` that injects the listener:

```java
public class UIAwareTaskCreator extends MultiLayerNetworkTaskCreator {
    private final StatsStorage statsStorage;

    public UIAwareTaskCreator(StatsStorage statsStorage) {
        this.statsStorage = statsStorage;
    }

    @Override
    public OptimizationTask create(Candidate candidate, DataSource dataSource,
                                   ResultSaver saver, ScoreFunction score,
                                   List<Listener> listeners) {
        // Inject StatsListener into the candidate model before training
        MultiLayerNetworkOptimizationTask task = (MultiLayerNetworkOptimizationTask)
            super.create(candidate, dataSource, saver, score, listeners);
        task.addNetworkListener(new StatsListener(statsStorage));
        return task;
    }
}
```

Then use `UIAwareTaskCreator` instead of `MultiLayerNetworkTaskCreator`.

---

## Programmatic Access to Results (No UI)

If you do not need the web UI, you can inspect results programmatically after the run completes:

```java
runner.execute();

// List all results sorted by score (best first)
List<OptimizationResult> results = runner.getResults();
results.sort(Comparator.comparingDouble(r -> -r.getScore()));

for (int i = 0; i < Math.min(5, results.size()); i++) {
    OptimizationResult r = results.get(i);
    System.out.printf("Rank %d: score=%.4f, index=%d%n",
        i + 1, r.getScore(), r.getIndex());
}

// Get the best model
MultiLayerNetwork bestModel = (MultiLayerNetwork)
    runner.getBestResult().getResultReference().getResultModel();
```

---

## Related Pages

- [Arbiter Overview](./overview.md) — full optimization configuration guide
- [Parameter Spaces](./parameter-spaces.md) — search space primitives
- [Layer Spaces](./layer-spaces.md) — layer-specific search spaces

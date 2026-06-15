---
title: Arbiter Overview
description: Arbiter hyperparameter optimization for DL4J — architecture, search strategies, configuration, and usage.
---

## What Is Arbiter?

Arbiter is the hyperparameter optimization module in the Eclipse Deeplearning4j ecosystem. It automates the search for optimal neural network configurations by systematically evaluating candidate architectures across a user-defined hyperparameter search space.

Neural networks require many decisions before training begins: learning rate, number of layers, layer sizes, regularization strength, batch size, activation functions, and more. Choosing these manually is time-consuming and often suboptimal. Arbiter replaces manual search with automated strategies — random search or grid search — that explore many candidates and report the best ones.

**When to use Arbiter:**

- You have a rough idea of the right architecture but want to tune precise hyperparameter values.
- You are willing to spend more compute time in exchange for better model performance.
- You want a reproducible, principled way to document which hyperparameter configurations you tried.

**When not to use Arbiter:**

- You already have a known-good architecture from a published paper or prior work. Tuning costs time.
- Your compute budget is very small. Random search needs at least 10–20 candidates to be useful.
- Your search space is poorly defined. Arbiter cannot find good models if the search space does not include any good configurations.

---

## Maven Dependency

```xml
<!-- Core hyperparameter optimization -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>arbiter-deeplearning4j</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>

<!-- UI visualization (optional) -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>arbiter-ui_2.11</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

---

## Architecture Overview

An Arbiter optimization run is described by an `OptimizationConfiguration` that ties together five components:

| Component | Interface | Purpose |
|---|---|---|
| Candidate generator | `CandidateGenerator` | Generates candidate hyperparameter configurations |
| Data source | `DataSource` | Provides training and test data to each candidate |
| Model saver | `ResultSaver` | Persists results from each candidate evaluation |
| Score function | `ScoreFunction` | Produces a single numeric score for each candidate |
| Termination conditions | `TerminationCondition[]` | Stops the optimization run |

The `OptimizationConfiguration` is then passed to an `IOptimizationRunner` which executes the candidates.

---

## Setting Up an Optimization Run

### 1. Define the Hyperparameter Search Space

Use `MultiLayerSpace` (for `MultiLayerNetwork` models) or `ComputationGraphSpace` (for `ComputationGraph` models) to define the range of valid hyperparameter values. These mirror DL4J's `MultiLayerConfiguration` and `ComputationGraphConfiguration` builders, with each hyperparameter taking either a fixed value or a `ParameterSpace<T>`.

```java
MultiLayerSpace searchSpace = new MultiLayerSpace.Builder()
    .seed(12345)
    .updater(new AdamSpace(new ContinuousParameterSpace(1e-4, 1e-2)))
    .l2(new ContinuousParameterSpace(1e-5, 1e-3))
    .addLayer(new DenseLayerSpace.Builder()
        .nIn(784)
        .nOut(new IntegerParameterSpace(64, 512))
        .activation(new DiscreteParameterSpace<>(Activation.RELU, Activation.TANH))
        .build())
    .addLayer(new OutputLayerSpace.Builder()
        .nOut(10)
        .activation(Activation.SOFTMAX)
        .lossFunction(LossFunctions.LossFunction.MCXENT)
        .build())
    .numEpochs(10)
    .build();
```

See [Parameter Spaces](./parameter-spaces.md) and [Layer Spaces](./layer-spaces.md) for the full reference.

### 2. Choose a Candidate Generator

**Random search** (recommended for most cases):

```java
CandidateGenerator candidateGenerator = new RandomSearchGenerator(searchSpace);
```

Random search samples hyperparameter configurations uniformly at random from the search space. It is typically more efficient than grid search for high-dimensional spaces because it does not waste evaluations on redundant combinations.

**Grid search:**

```java
CandidateGenerator candidateGenerator = new GridSearchCandidateGenerator(
    searchSpace,
    4,                                          // discretization count for continuous params
    GridSearchCandidateGenerator.Mode.Sequential
);
```

`discretizationCount = 4` converts a continuous range like `[0.0001, 0.01]` into four discrete values. `Mode.Sequential` evaluates them in order; `Mode.RandomOrder` shuffles the order. Grid search is only practical for small, low-dimensional search spaces.

### 3. Implement a DataSource

`DataSource` provides data to each candidate model during training and evaluation. It must have a no-argument constructor. Optional configuration can be injected via `Properties`.

```java
public class MnistDataSource implements DataSource {
    private int batchSize;

    public MnistDataSource() {}

    @Override
    public void configure(Properties properties) {
        this.batchSize = Integer.parseInt(properties.getProperty("batchSize", "32"));
    }

    @Override
    public Object trainData() {
        try {
            return new MnistDataSetIterator(batchSize, true, 12345);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public Object testData() {
        try {
            return new MnistDataSetIterator(batchSize, false, 12345);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public Class<?> getDataType() {
        return DataSetIterator.class;
    }
}
```

### 4. Choose a Model Saver

**Save to disk:**

```java
ResultSaver modelSaver = new FileModelSaver("/tmp/arbiter_results/");
```

Results are saved under `baseDir/0/`, `baseDir/1/`, etc., indexed by `OptimizationResult.getIndex()`. Each directory contains the model configuration, parameters, and score.

**In-memory (small models only):**

```java
ResultSaver modelSaver = new InMemoryResultSaver();
```

### 5. Choose a Score Function

The score function assigns a single scalar to each candidate. Arbiter selects the candidate with the best score.

**Classification accuracy:**

```java
ScoreFunction scoreFunction = new EvaluationScoreFunction(Evaluation.Metric.ACCURACY);
```

Available `Evaluation.Metric` values: `ACCURACY`, `F1`, `PRECISION`, `RECALL`, `GMEASURE`, `MCC`.

**ROC AUC:**

```java
ScoreFunction scoreFunction = new ROCScoreFunction(
    ROCScoreFunction.ROCType.BINARY,
    ROCScoreFunction.Metric.AUC
);
```

**Regression (MSE):**

```java
ScoreFunction scoreFunction = new RegressionScoreFunction(RegressionEvaluation.Metric.MSE);
```

For regression, Arbiter minimizes the score (lower MSE is better). For classification, it maximizes the score (higher accuracy is better).

### 6. Set Termination Conditions

```java
TerminationCondition[] terminationConditions = {
    new MaxTimeCondition(30, TimeUnit.MINUTES),  // run for at most 30 minutes
    new MaxCandidatesCondition(20)               // or 20 candidates, whichever comes first
};
```

### 7. Build the OptimizationConfiguration

```java
Properties dataSourceProperties = new Properties();
dataSourceProperties.setProperty("batchSize", "64");

OptimizationConfiguration configuration = new OptimizationConfiguration.Builder()
    .candidateGenerator(candidateGenerator)
    .dataSource(MnistDataSource.class, dataSourceProperties)
    .modelSaver(modelSaver)
    .scoreFunction(scoreFunction)
    .terminationConditions(terminationConditions)
    .build();
```

### 8. Run with the OptimizationRunner

**For MultiLayerNetwork candidates:**

```java
IOptimizationRunner runner = new LocalOptimizationRunner(
    configuration,
    new MultiLayerNetworkTaskCreator()
);
runner.execute();
```

**For ComputationGraph candidates:**

```java
IOptimizationRunner runner = new LocalOptimizationRunner(
    configuration,
    new ComputationGraphTaskCreator()
);
runner.execute();
```

`LocalOptimizationRunner` runs all candidates sequentially in the current JVM. It is the only runner available in 1.0.0-M2.1.

---

## Inspecting Results

After the run completes:

```java
// Print a summary of all candidates and their scores
String summary = runner.toSummaryString();
System.out.println(summary);

// Get the best result
OptimizationResult best = runner.getBestResult();
MultiLayerNetwork bestModel = (MultiLayerNetwork) best.getResultReference().getResultModel();
```

---

## Variable-Depth Networks

Arbiter can vary the number of layers in a `MultiLayerSpace`:

```java
MultiLayerSpace mls = new MultiLayerSpace.Builder()
    .updater(new Sgd(0.01))
    .addLayer(
        new DenseLayerSpace.Builder().nIn(100).nOut(50).build(),
        new IntegerParameterSpace(1, 4)  // 1 to 4 identical layers
    )
    .addLayer(new OutputLayerSpace.Builder().nIn(50).nOut(2).build())
    .numEpochs(5)
    .build();
```

The layers created within a repeated stack are identical (stacked). Arbiter does not support independent configuration of each layer within a variable-depth stack.

---

## JSON Serialization

`OptimizationConfiguration`, `MultiLayerSpace`, and `ComputationGraphSpace` can be serialized to JSON for storage and reproducibility:

```java
String configJson = configuration.toJson();

// Restore
OptimizationConfiguration restored = OptimizationConfiguration.fromJson(configJson);
```

---

## Tips for Effective Hyperparameter Search

1. **Use random search over grid search.** Random search is more efficient for high-dimensional spaces because it covers the space more evenly and is less likely to waste evaluations on unimportant dimensions.

2. **Search from coarse to fine.** Run a short coarse search (1–2 epochs per candidate, wide ranges) to find promising regions. Then run a fine search in the promising region with more epochs.

3. **Use log-uniform distributions for scale-sensitive parameters.** Learning rate and regularization parameters span multiple orders of magnitude. Use `ContinuousParameterSpace` with the `LogUniformDistribution` for these.

4. **Watch for boundary clustering.** If the best candidates repeatedly cluster near the boundary of a search range, expand that range in the next search.

5. **Allocate more candidates than you think you need.** With random search, approximately 60 candidates are needed to have 95% probability of finding a configuration in the top 5% of the space.

---

## Related Pages

- [Layer Spaces](./layer-spaces.md) — layer-specific parameter spaces
- [Parameter Spaces](./parameter-spaces.md) — primitive and composite parameter space types
- [Visualization](./visualization.md) — Arbiter UI and result monitoring

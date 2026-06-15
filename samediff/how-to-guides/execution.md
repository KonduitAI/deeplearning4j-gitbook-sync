---
title: "SameDiff Execution and Inference"
description: "Running SameDiff graphs — exec(), output(), batchOutput(), placeholders, and InferenceSession"
---

# SameDiff Execution and Inference

Defining a SameDiff graph is separate from running it. Once the graph is built, you execute it by supplying values for all `PLACEHOLDER` variables and specifying which output variables you want computed. SameDiff then evaluates just the subgraph necessary to produce those outputs.

## How Execution Works

SameDiff uses an `InferenceSession` internally to execute the graph. The session:

1. Accepts a map of placeholder name → `INDArray` value.
2. Determines which nodes are needed to compute the requested outputs (topological ordering).
3. Evaluates each node in order, caching intermediate results.
4. Returns the requested output arrays.

You rarely interact with `InferenceSession` directly; instead you use the higher-level methods on `SameDiff` described below.

## Setting Placeholder Values

Before execution, every `PLACEHOLDER` variable in the graph must have a value. Values are supplied as a `Map<String, INDArray>`:

```java
import org.nd4j.linalg.factory.Nd4j;
import java.util.HashMap;
import java.util.Map;

INDArray inputBatch = Nd4j.create(/* your data */);

Map<String, INDArray> placeholders = new HashMap<>();
placeholders.put("input", inputBatch);
```

If your graph also has label placeholders (e.g. for computing a validation loss), include those too:

```java
placeholders.put("labels", labelBatch);
```

You do not need to supply values for `VARIABLE` or `CONSTANT` nodes — they are stored inside the `SameDiff` instance and used automatically.

## sd.output() — Standard Inference

`sd.output()` is the primary method for running the forward pass and retrieving results. It returns a `Map<String, INDArray>` whose keys are the names of the requested output variables.

```java
// Request a single output
Map<String, INDArray> results = sd.output(placeholders, "softmax");
INDArray predictions = results.get("softmax");
```

Request multiple outputs in one call to avoid recomputing the graph twice:

```java
Map<String, INDArray> results = sd.output(placeholders, "hidden", "softmax", "loss");

INDArray hiddenActivations = results.get("hidden");
INDArray outputProbabilities = results.get("softmax");
INDArray lossValue = results.get("loss");
```

Only the nodes required to compute the listed outputs are executed. If you do not request a particular output, its subgraph may be skipped entirely.

## outputSingle() — Convenience for One Output

When you only need a single output array and do not want to unwrap a map, use `outputSingle()`:

```java
INDArray predictions = sd.outputSingle(placeholders, "softmax");
```

This is equivalent to `sd.output(placeholders, "softmax").get("softmax")` but saves a map lookup.

## Evaluating Persistent Variables

For `VARIABLE` and `CONSTANT` nodes, you can retrieve their stored values directly without running the graph:

```java
INDArray weights = sd.getVariable("w1").eval();
INDArray bias    = sd.getVariable("b1").getArr();
```

`eval()` is equivalent to `getArr()` for persistent variables. For `ARRAY` or `PLACEHOLDER` nodes, you need to have executed the graph first.

## exec() — Low-Level Graph Execution

`exec()` runs the full forward (and optionally backward) pass and returns a `Map<String, INDArray>` of all computed values. It is lower-level than `output()` and is mainly used when you need access to every intermediate result or when you are driving the training loop manually.

```java
import org.nd4j.autodiff.samediff.execution.ExecResult;

// Forward pass only
Map<String, INDArray> allOutputs = sd.exec(placeholders, sd.outputs());

// Access any variable value
INDArray hiddenOut = allOutputs.get("hidden");
```

For most inference use cases, prefer `output()` over `exec()` because it executes only the necessary subgraph.

## Batch Inference

For large datasets, iterate and accumulate predictions batch by batch:

```java
DataSetIterator testIter = /* your iterator */;
List<INDArray> allPredictions = new ArrayList<>();

while (testIter.hasNext()) {
    DataSet batch = testIter.next();

    Map<String, INDArray> pv = Map.of("input", batch.getFeatures());
    INDArray batchPred = sd.outputSingle(pv, "softmax");
    allPredictions.add(batchPred);
}

// Concatenate all predictions
INDArray predictions = Nd4j.vstack(allPredictions);
```

If you are computing a metric over the whole test set, use the built-in evaluation API instead — it is more efficient and avoids materialising all predictions in memory at once:

```java
import org.nd4j.evaluation.classification.Evaluation;

Evaluation eval = new Evaluation();
sd.evaluate(testIter, "output", 0, eval);
System.out.println(eval.stats());
```

The `evaluate()` method feeds batches through the graph and accumulates metric statistics incrementally.

## Querying Output Variable Names

To see which variables are marked as "outputs" of the graph (i.e. the terminal nodes that produce final results):

```java
List<String> outputNames = sd.outputs();
```

You can also list all variable names:

```java
List<String> allVarNames = sd.variableNames();
```

## Placeholder Shape Inference

SameDiff propagates shape information through the graph at graph-definition time where possible. Use `-1` for dimensions that are only known at runtime (typically the batch dimension):

```java
SDVariable input = sd.placeHolder("input", DataType.FLOAT, -1, 784);
```

After execution, the actual shape of any `ARRAY` variable can be retrieved:

```java
// Before execution: may contain -1 for unknown dims
long[] inferredShape = sd.getShapeForVarName("hidden");

// After execution: concrete shape from the actual data
INDArray result = sd.outputSingle(placeholders, "hidden");
long[] actualShape = result.shape();
```

## Performance Considerations

### Avoid recreating the SameDiff graph per request

Building a `SameDiff` graph (calling `sd.var()`, `sd.placeHolder()`, etc.) is expensive. Build the graph once — at application startup or model load time — and reuse the same `SameDiff` instance for all inference requests.

```java
// At startup:
SameDiff model = SameDiff.load(new File("model.fb"), true);

// Per request:
INDArray pred = model.outputSingle(Map.of("input", request), "softmax");
```

### Thread safety

A single `SameDiff` instance is not safe to call from multiple threads concurrently during inference, because execution caches results in the instance's internal state. Options:

- **Lock per call**: `synchronized(model) { model.outputSingle(...); }`
- **Pool of instances**: pre-load N copies of the model from the same file and distribute requests round-robin.
- **Use separate instances per thread** via `ThreadLocal<SameDiff>`.

### Minimise requested outputs

Only request the output variables you actually need. Requesting fewer outputs means fewer graph nodes are evaluated:

```java
// Good: only compute the softmax output
INDArray pred = sd.outputSingle(placeholders, "softmax");

// Less efficient if you only need predictions: also computes hidden + loss
Map<String, INDArray> all = sd.output(placeholders, "hidden", "softmax", "loss");
```

### Reuse INDArray input buffers

Where possible, reuse the same `INDArray` object across calls (refilling its contents) rather than allocating a new one per batch. This reduces garbage-collection pressure:

```java
INDArray inputBuffer = Nd4j.create(DataType.FLOAT, 64, 784);

while (source.hasNext()) {
    source.fillBatch(inputBuffer);  // write new data in-place
    INDArray pred = sd.outputSingle(Map.of("input", inputBuffer), "softmax");
    // process pred ...
}
```

## Working with InferenceSession Directly

Advanced users can interact with `InferenceSession` directly for fine-grained control:

```java
import org.nd4j.autodiff.samediff.internal.InferenceSession;

InferenceSession session = new InferenceSession(sd);

Map<String, INDArray> placeholderValues = Map.of("input", myInput);
List<String> requiredOutputs = List.of("softmax");
Set<String> requiredActivations = Collections.emptySet();

Map<String, INDArray> result = session.output(
    requiredOutputs,
    requiredActivations,
    placeholderValues,
    Collections.emptyList(),   // listeners
    At.defaultAt(),
    MultiDataSets.singleton(null, null)
);

INDArray softmax = result.get("softmax");
```

This level of control is rarely needed. Use `sd.output()` or `sd.outputSingle()` in all normal circumstances.

## Checking Graph Validity Before Execution

SameDiff can validate the graph structure before you run it. This is useful during development to catch wiring errors early:

```java
// Validate that all required inputs are connected and shapes are consistent
sd.validate();
```

If the graph has any disconnected nodes, missing inputs, or shape mismatches that can be detected statically, `validate()` will throw a descriptive exception.

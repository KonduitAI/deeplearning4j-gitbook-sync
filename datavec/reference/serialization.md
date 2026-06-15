---
title: "Serialization"
description: "Data serialization in DataVec — saving and loading schemas, transform processes, and normalized data"
---

# Serialization

DataVec's serialization support lets you save your data pipeline definitions and normalizer statistics so they can be reloaded in production without rerunning analysis or redefining transforms in code.

The three objects you typically need to serialize are:

1. **Schema** — the column layout description
2. **TransformProcess** — the ordered list of transforms
3. **Normalizer** — fitted mean/std or min/max statistics

## Serializing a TransformProcess

`TransformProcess` serializes to JSON or YAML. The result captures every transform step and can be written to a file or database.

```java
// Serialize
String json = tp.toJson();
String yaml = tp.toYaml();

// Write to file
Files.write(Paths.get("transform_process.json"), json.getBytes(StandardCharsets.UTF_8));

// Deserialize
String savedJson = new String(Files.readAllBytes(Paths.get("transform_process.json")));
TransformProcess loaded = TransformProcess.fromJson(savedJson);
TransformProcess loadedYaml = TransformProcess.fromYaml(yaml);
```

The JSON includes the full schema and every transform step with all parameters. It is human-readable and can be version-controlled alongside your model artifacts.

## JsonSerializer and YamlSerializer

For lower-level serialization of individual DataVec objects (transforms, conditions, filters):

```java
import org.datavec.api.transform.serde.JsonSerializer;
import org.datavec.api.transform.serde.YamlSerializer;

JsonSerializer js = new JsonSerializer();
String transformJson = js.serialize(myTransform);

YamlSerializer ys = new YamlSerializer();
String conditionYaml = ys.serialize(myCondition);
```

These serializers handle DataVec's polymorphic object model by embedding type information in the JSON/YAML output.

## Serializing a Schema

```java
// Serialize
String schemaJson = schema.toJson();
String schemaYaml = schema.toYaml();

// Save
Files.write(Paths.get("schema.json"), schemaJson.getBytes(StandardCharsets.UTF_8));

// Reload
Schema reloaded = Schema.fromJson(
    new String(Files.readAllBytes(Paths.get("schema.json"))));
Schema reloadedYaml = Schema.fromYaml(schemaYaml);
```

## Serializing Normalizers

Normalizers hold the statistics computed during `fit()`. Two approaches are available.

### NormalizerSerializer (Recommended)

```java
import org.nd4j.linalg.dataset.api.preprocessor.serializer.NormalizerSerializer;

NormalizerSerializer serializer = NormalizerSerializer.getDefault();

// Save
normalizer.fit(trainIterator);
serializer.write(normalizer, new File("normalizer.bin"));

// Load
NormalizerStandardize loaded = serializer.restore(new File("normalizer.bin"));
```

`NormalizerSerializer` supports all normalizer types and detects the type automatically during restore.

### ModelSerializer (Bundle with Model)

The easiest approach for production is to bundle the normalizer with the model file:

```java
import org.deeplearning4j.util.ModelSerializer;

// Save model and normalizer together
ModelSerializer.writeModel(model, new File("model.zip"), true, normalizer);

// Load
MultiLayerNetwork loadedModel = ModelSerializer.restoreMultiLayerNetwork(new File("model.zip"));
NormalizerStandardize loadedNormalizer = ModelSerializer.restoreNormalizer(new File("model.zip"));

// Apply at inference time
inferenceIterator.setPreProcessor(loadedNormalizer);
```

Bundling keeps preprocessing and model weights in a single artifact, reducing the risk of version mismatches.

## Full Pipeline Serialization Pattern

```java
// --- Training time ---

Schema schema = new Schema.Builder()
    .addColumnDouble("feature1")
    .addColumnDouble("feature2")
    .addColumnCategorical("label", Arrays.asList("A", "B", "C"))
    .build();

TransformProcess tp = new TransformProcess.Builder(schema)
    .categoricalToInteger("label")
    .build();

RecordReader rr = new CSVRecordReader(1, ',');
rr.initialize(new FileSplit(new File("train.csv")));
RecordReader transformed = new TransformProcessRecordReader(rr, tp);
DataSetIterator iter = new RecordReaderDataSetIterator(transformed, 32, 2, 3);

NormalizerStandardize normalizer = new NormalizerStandardize();
normalizer.fit(iter);
iter.reset();
iter.setPreProcessor(normalizer);

model.fit(iter);

// Save artifacts
Files.write(Paths.get("schema.json"), schema.toJson().getBytes(StandardCharsets.UTF_8));
Files.write(Paths.get("transform.json"), tp.toJson().getBytes(StandardCharsets.UTF_8));
ModelSerializer.writeModel(model, new File("model.zip"), true, normalizer);


// --- Inference time ---

Schema inferSchema = Schema.fromJson(
    new String(Files.readAllBytes(Paths.get("schema.json"))));
TransformProcess inferTp = TransformProcess.fromJson(
    new String(Files.readAllBytes(Paths.get("transform.json"))));
MultiLayerNetwork inferModel = ModelSerializer.restoreMultiLayerNetwork(new File("model.zip"));
NormalizerStandardize inferNorm = ModelSerializer.restoreNormalizer(new File("model.zip"));

RecordReader newRr = new CSVRecordReader(0, ',');
newRr.initialize(new FileSplit(new File("new_data.csv")));
RecordReader newTransformed = new TransformProcessRecordReader(newRr, inferTp);
DataSetIterator inferIter = new RecordReaderDataSetIterator(newTransformed, 1, -1, -1);
inferIter.setPreProcessor(inferNorm);

INDArray output = inferModel.output(inferIter.next().getFeatures());
```

## What to Save

| Artifact | Why |
|---|---|
| `schema.json` | Describes expected input columns and types |
| `transform.json` | Defines preprocessing steps |
| `model.zip` | Contains model weights and embedded normalizer |

If you used the `normalize()` step inside `TransformProcess`, also save the `DataAnalysis` object that was passed to it, as it contains the statistics used for that normalization.

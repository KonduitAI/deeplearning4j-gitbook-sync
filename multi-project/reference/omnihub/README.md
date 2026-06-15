---
title: "OmniHub Overview"
description: "OmniHub model registry — downloading and using pretrained models from the DL4J ecosystem"
---

# OmniHub Overview

OmniHub is a pretrained model registry for the Eclipse Deeplearning4j ecosystem. It provides a unified API for downloading, caching, and loading pretrained models in either DL4J or SameDiff format, backed by a remote model zoo hosted on GitHub.

---

## What OmniHub Provides

The DL4J ecosystem has historically offered pretrained models through the `deeplearning4j-zoo` module. OmniHub extends this with a simpler, more extensible approach: models are stored as serialized files in a versioned GitHub repository and fetched on demand. The local cache lives in `~/.omnihub` (or a custom directory set via the `OMNIHUB_HOME` environment variable), so each model is downloaded at most once.

The API surface is small: one utility class (`OmniHubUtils`), one configuration class (`OmnihubConfig`), one enum of supported frameworks (`Framework`), and two generated model classes (`Dl4jModels`, `SameDiffModels`) that expose named methods for each available model. The `Pretrained` class ties the two model classes together as a single access point.

---

## Framework Enum

The `Framework` enum categorizes all frameworks that OmniHub understands:

```java
public enum Framework {
    SAMEDIFF,   // DL4J's SameDiff graph execution engine
    PYTORCH,    // PyTorch (input format for conversion)
    TENSORFLOW, // TensorFlow (input format for conversion)
    KERAS,      // Keras H5 (input format for conversion)
    DL4J,       // DL4J MultiLayerNetwork / ComputationGraph
    ONNX,       // ONNX (input format for conversion)
    HUGGINGFACE // Hugging Face (reserved)
}
```

OmniHub distinguishes between **input frameworks** (PYTORCH, TENSORFLOW, KERAS, ONNX — formats that are imported and converted) and **output frameworks** (SAMEDIFF, DL4J — the formats in which models are stored and served). The static helpers `Framework.isInput()` and `Framework.isOutput()` reflect this distinction.

For the model zoo, the two relevant output frameworks are:

- **DL4J** — models stored as `ComputationGraph` or `MultiLayerNetwork` zip files. Loaded via `OmniHubUtils.loadCompGraph()` or `OmniHubUtils.loadNetwork()`.
- **SAMEDIFF** — models stored as FlatBuffers (`.fb`) files. Loaded via `OmniHubUtils.loadSameDiffModel()`.

---

## How Models Are Stored

The remote model zoo is a GitHub repository at:

```
https://raw.githubusercontent.com/KonduitAI/omnihub-zoo/main
```

Models are organized into two subdirectories within that repository:

```
omnihub-zoo/
  dl4j/
    vgg19_weights_tf_dim_ordering_tf_kernels_notop.zip
  samediff/
    age_googlenet.fb
    resnet18.fb
```

The local cache mirrors this structure under `~/.omnihub`:

```
~/.omnihub/
  dl4j/
    vgg19_weights_tf_dim_ordering_tf_kernels_notop.zip
  samediff/
    age_googlenet.fb
    resnet18.fb
```

When a model is requested, `OmniHubUtils.downloadAndLoadFromZoo()` checks whether the file exists locally. If it does, the cached copy is used. If not, the file is streamed from the remote URL with a progress indicator, then written to the cache directory.

---

## Comparison with DL4J Zoo

The original `deeplearning4j-zoo` module embeds model weights directly in JAR artifacts. This approach has the advantage of fully offline availability but requires large JAR downloads for every model, even models the application does not use.

OmniHub takes the opposite approach: the Maven dependency is lightweight and model weights are downloaded lazily at runtime. This keeps the classpath small and makes it easy to add new models to the zoo without releasing a new library version.

| Aspect | deeplearning4j-zoo | OmniHub |
|--------|-------------------|---------|
| Distribution | Weights in JAR | Downloaded at runtime |
| Offline use | Yes (after JAR download) | Yes (after first use, from cache) |
| Adding new models | New library release required | New file in zoo repo |
| Model types | DL4J MultiLayerNetwork | DL4J ComputationGraph, MultiLayerNetwork, SameDiff |

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>omnihub</artifactId>
    <version>${dl4j.version}</version>
</dependency>
```

---

## Configuration

| Environment Variable | Default | Description |
|---------------------|---------|-------------|
| `OMNIHUB_HOME` | `~/.omnihub` | Local cache directory for downloaded models |
| `OMNIHUB_URL` | `https://raw.githubusercontent.com/KonduitAI/omnihub-zoo/main` | Remote model zoo base URL |

Both variables are read at runtime, so you can override them without recompiling. Set `OMNIHUB_URL` to point at a private mirror if you need air-gapped deployment after an initial sync.

---

## Next Steps

- [OmniHub Usage](usage.md) — downloading and loading models with the `OmniHubUtils` and `Pretrained` APIs.
- [Available Models](available-models.md) — catalog of pretrained models currently in the zoo.

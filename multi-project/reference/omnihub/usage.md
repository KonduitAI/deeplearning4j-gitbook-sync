---
title: "OmniHub Usage"
description: "Downloading and loading models from OmniHub — OmniHubUtils API"
---

# OmniHub Usage

OmniHub exposes two entry points for loading pretrained models: the `Pretrained` facade (the recommended path for named models) and the lower-level `OmniHubUtils` class (useful when you have a filename and framework string and want direct control).

---

## The Pretrained Facade

`org.eclipse.deeplearning4j.omnihub.models.Pretrained` is the simplest way to load a known model. It provides static accessor methods for the two framework namespaces:

```java
import org.eclipse.deeplearning4j.omnihub.models.Pretrained;

// Load the VGG19 (no-top) DL4J ComputationGraph
ComputationGraph vgg19 = Pretrained.dl4j().vgg19noTop(false);

// Load the AgeGoogLeNet SameDiff model
SameDiff ageGooglenet = Pretrained.samediff().ageGooglenet(false);

// Load ResNet18 SameDiff model
SameDiff resnet18 = Pretrained.samediff().resnet18(false);
```

The boolean argument is `forceDownload`. Pass `true` to delete the cached copy and re-download even if it already exists locally.

`Pretrained.dl4j()` returns a `Dl4jModels` instance. `Pretrained.samediff()` returns a `SameDiffModels` instance. Both classes are code-generated from the zoo manifest; their methods map directly to named model files.

---

## OmniHubUtils — Loading DL4J Models

### Loading a MultiLayerNetwork

```java
import org.eclipse.deeplearning4j.omnihub.OmniHubUtils;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;

// Load from cache (download if not cached)
MultiLayerNetwork net = OmniHubUtils.loadNetwork("mymodel.zip");

// Force re-download even if cached
MultiLayerNetwork net = OmniHubUtils.loadNetwork("mymodel.zip", true);
```

`loadNetwork` downloads the file from the `dl4j/` directory of the zoo, stores it in `~/.omnihub/dl4j/`, and loads it using `MultiLayerNetwork.load(file, true)` (with the parameter updater restored).

### Loading a ComputationGraph

```java
import org.deeplearning4j.nn.graph.ComputationGraph;

ComputationGraph graph = OmniHubUtils.loadCompGraph("vgg19_weights_tf_dim_ordering_tf_kernels_notop.zip");

// With force download
ComputationGraph graph = OmniHubUtils.loadCompGraph(
    "vgg19_weights_tf_dim_ordering_tf_kernels_notop.zip",
    true
);
```

`loadCompGraph` is identical to `loadNetwork` except it calls `ComputationGraph.load(file, true)`.

---

## OmniHubUtils — Loading SameDiff Models

```java
import org.nd4j.autodiff.samediff.SameDiff;

SameDiff model = OmniHubUtils.loadSameDiffModel("age_googlenet.fb");

// With force download
SameDiff model = OmniHubUtils.loadSameDiffModel("age_googlenet.fb", true);
```

SameDiff models are stored as FlatBuffers files (`.fb`). They are downloaded from the `samediff/` directory of the zoo and cached under `~/.omnihub/samediff/`. Loading uses `SameDiff.load(file, true)`.

---

## Direct Download with downloadAndLoadFromZoo

For cases where you want the local `File` handle rather than an already-loaded model object, call `downloadAndLoadFromZoo` directly:

```java
import java.io.File;

File modelFile = OmniHubUtils.downloadAndLoadFromZoo(
    "samediff",           // framework subdirectory
    "resnet18.fb",        // filename
    false                 // forceDownload
);

// Now load it yourself
SameDiff sd = SameDiff.load(modelFile, true);
```

The framework string must match the subdirectory name in the zoo repository (`"dl4j"` or `"samediff"`).

---

## Caching Behavior

Models are cached in the directory returned by `OmnihubConfig.getOmnihubHome()`:

- Default: `~/.omnihub`
- Custom: set the `OMNIHUB_HOME` environment variable before the JVM starts.

The cache structure is flat per-framework:

```
~/.omnihub/
  dl4j/
    vgg19_weights_tf_dim_ordering_tf_kernels_notop.zip
  samediff/
    age_googlenet.fb
    resnet18.fb
```

A download is skipped if the destination file already exists (unless `forceDownload` is `true`). There is no checksum verification in the current implementation — if you suspect a file is corrupt, pass `forceDownload = true` to replace it.

---

## Overriding the Zoo URL

By default, models are fetched from:

```
https://raw.githubusercontent.com/KonduitAI/omnihub-zoo/main
```

To point at a private mirror or a local HTTP server (for air-gapped environments), set the `OMNIHUB_URL` environment variable:

```bash
export OMNIHUB_URL=http://internal-mirror.example.com/omnihub
```

The URL is read by `OmnihubConfig.getOmnihubUrl()` each time a download is needed. The file is resolved at:

```
${OMNIHUB_URL}/${framework}/${name}
```

For example, `OmniHubUtils.loadSameDiffModel("resnet18.fb")` would fetch:

```
http://internal-mirror.example.com/omnihub/samediff/resnet18.fb
```

---

## Download Progress

`OmniHubUtils` wraps the download stream in a `ProgressInputStream` that prints a progress indicator to the console. The indicator is sized using an HTTP `HEAD` request to obtain the `Content-Length` before streaming begins. No configuration is needed; it activates automatically for every download.

---

## Full Example

```java
import org.eclipse.deeplearning4j.omnihub.models.Pretrained;
import org.deeplearning4j.nn.graph.ComputationGraph;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.ndarray.INDArray;

public class OmniHubExample {
    public static void main(String[] args) throws Exception {
        // Load VGG19 (no top layers) — downloads on first run, cached thereafter
        ComputationGraph vgg19 = Pretrained.dl4j().vgg19noTop(false);

        // VGG19 no-top expects input shape [batch, 3, 224, 224] (channels-first)
        INDArray input = Nd4j.rand(new int[]{1, 3, 224, 224});
        INDArray[] output = vgg19.output(input);

        System.out.println("VGG19 feature map shape: " + output[0].shapeInfoToString());
    }
}
```

---

## Next Steps

- [Available Models](available-models.md) — full catalog of models in the zoo, with input/output details.
- [OmniHub Overview](overview.md) — architecture, configuration, and comparison with deeplearning4j-zoo.

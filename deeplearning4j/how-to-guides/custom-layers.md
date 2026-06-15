---
title: "Custom Layers"
description: "Writing custom layers in Deeplearning4j — extending Layer, SameDiff-backed custom layers, and custom graph vertices"
---

# Custom Layers

## When to Write a Custom Layer

DL4J ships with a large library of built-in layers (dense, CNN, RNN, batch norm, etc.). Before writing a custom layer, check whether your operation can be expressed with:

- A combination of existing layers in a `ComputationGraph`.
- A `SameDiff` `Lambda` applied inline to activations.
- A custom activation function (extend `BaseActivationFunction`).

If none of those cover your case, you need a custom layer. Common reasons include:

- A novel activation shape or parameterization not captured by existing layers.
- A specialized loss term that must reach inside the forward/backward pass.
- Porting a research paper that defines a non-standard layer with its own parameters.

DL4J provides two implementation strategies: the **SameDiff approach** (recommended for most cases) and the **traditional approach** (extending `BaseLayer` directly).

---

## Recommended: SameDiff-Backed Custom Layers

SameDiff is DL4J's built-in automatic differentiation framework. When a custom layer defines its forward pass using SameDiff operations, backpropagation is computed automatically — you do not implement gradients by hand.

There are four SameDiff layer base classes:

| Class | Has parameters? | Input count | Use case |
|---|---|---|---|
| `SameDiffLayer` | Yes | 1 | Parameterized layers (learned weights/biases) |
| `SameDiffLambdaLayer` | No | 1 | Stateless element-wise or reshape transforms |
| `SameDiffOutputLayer` | Yes | 1 | Custom output layers with their own loss |
| `SameDiffVertex` | No | N | Graph vertices for `ComputationGraph` with multiple inputs |

### SameDiffLayer — Parameterized Custom Layer

Extend `SameDiffLayer` when your layer has trainable parameters (weights, biases, etc.).

You must implement four methods:

```java
import org.deeplearning4j.nn.conf.layers.samediff.SameDiffLayer;
import org.nd4j.autodiff.samediff.SDVariable;
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.linalg.api.ndarray.INDArray;

public class MyDenseLayer extends SameDiffLayer {

    private int nIn;
    private int nOut;

    // No-arg constructor required for JSON deserialization
    protected MyDenseLayer() { }

    public MyDenseLayer(int nIn, int nOut) {
        this.nIn  = nIn;
        this.nOut = nOut;
    }

    /**
     * Declare the parameter shapes. Keys are arbitrary strings that
     * identify each parameter; they must be consistent with defineLayer().
     */
    @Override
    public Map<String, long[]> defineParameterShape(InputType inputType,
                                                     int minibatchSize) {
        Map<String, long[]> m = new LinkedHashMap<>();
        m.put("W", new long[]{nIn, nOut});
        m.put("b", new long[]{1, nOut});
        return m;
    }

    /**
     * Initialize parameter values. Called once at network initialization.
     * Use the provided initializer helpers, or set values directly.
     */
    @Override
    public void initializeParameters(Map<String, INDArray> params) {
        initWeights(nIn, nOut, WeightInit.XAVIER, params.get("W"));
        params.get("b").assign(0.0);
    }

    /**
     * Define the forward pass as a SameDiff graph.
     * - layerInput:  the SDVariable representing this layer's input
     * - paramTable:  map from parameter name to SDVariable
     * Returns the layer output SDVariable.
     */
    @Override
    public SDVariable defineLayer(SameDiff sd,
                                  SDVariable layerInput,
                                  Map<String, SDVariable> paramTable,
                                  SDVariable mask) {
        SDVariable W = paramTable.get("W");
        SDVariable b = paramTable.get("b");
        // out = relu(input * W + b)
        return sd.nn.relu(layerInput.mmul(W).add(b), 0);
    }
}
```

Usage in a network configuration:

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .updater(new Adam(1e-3))
    .list()
    .layer(new MyDenseLayer(784, 256))
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.MCXENT)
           .activation(Activation.SOFTMAX).nIn(256).nOut(10).build())
    .build();
```

### SameDiffLambdaLayer — Stateless Transform

Use `SameDiffLambdaLayer` for transformations that have no learnable parameters. You only implement `defineLayer`.

```java
import org.deeplearning4j.nn.conf.layers.samediff.SameDiffLambdaLayer;

public class L2NormalizeLayer extends SameDiffLambdaLayer {

    @Override
    public SDVariable defineLayer(SameDiff sd,
                                  SDVariable layerInput,
                                  Map<String, SDVariable> paramTable,
                                  SDVariable mask) {
        // Normalize each example vector to unit L2 norm
        SDVariable norm = layerInput.norm2(true, 1);
        return layerInput.div(norm.add(1e-8));
    }
}
```

Because there are no parameters, you do not implement `defineParameterShape` or `initializeParameters`. Both SameDiff layer types fully participate in standard serialization, Spark training, and the training UI.

### SameDiffOutputLayer — Custom Output with Loss

Extend `SameDiffOutputLayer` when you need a custom loss function that must be part of the layer definition.

```java
public class HuberOutputLayer extends SameDiffOutputLayer {

    private double delta;

    protected HuberOutputLayer() { }

    public HuberOutputLayer(double delta) {
        this.delta = delta;
    }

    @Override
    public Map<String, long[]> defineParameterShape(InputType inputType,
                                                     int minibatchSize) {
        // Output layers may have a weight matrix connecting from nIn to nOut
        // Return empty map if truly parameterless at this level
        return Collections.emptyMap();
    }

    @Override
    public void initializeParameters(Map<String, INDArray> params) { }

    @Override
    public SDVariable defineLayer(SameDiff sd,
                                  SDVariable layerInput,
                                  Map<String, SDVariable> paramTable,
                                  SDVariable mask) {
        // layerInput is the prediction; labels are accessed via sd.getVariable("label")
        SDVariable label = sd.getVariable("label");
        SDVariable diff  = layerInput.sub(label).abs();
        // Huber: if |y - y_hat| <= delta => 0.5 * diff^2, else delta * (diff - 0.5*delta)
        SDVariable quadratic = sd.math.square(diff).mul(0.5);
        SDVariable linear    = diff.sub(delta * 0.5).mul(delta);
        SDVariable loss      = sd.math.min(quadratic, linear);
        return loss.mean();  // scalar loss
    }
}
```

### SameDiffVertex — Multi-Input Graph Vertex

Use `SameDiffVertex` in `ComputationGraph` configurations when your operation takes multiple inputs.

```java
import org.deeplearning4j.nn.conf.layers.samediff.SameDiffVertex;

public class WeightedAddVertex extends SameDiffVertex {

    private double alpha;
    private double beta;

    public WeightedAddVertex(double alpha, double beta) {
        this.alpha = alpha;
        this.beta  = beta;
    }

    @Override
    public Map<String, long[]> defineParameterShape(InputType[] inputTypes,
                                                     int minibatchSize) {
        return Collections.emptyMap();
    }

    @Override
    public void initializeParameters(Map<String, INDArray> params) { }

    /**
     * inputs: array of SDVariables, one per graph input to this vertex.
     */
    @Override
    public SDVariable defineVertex(SameDiff sd,
                                   Map<String, SDVariable> inputs,
                                   Map<String, SDVariable> paramTable,
                                   SDVariable mask,
                                   SDVariable minibatchSize) {
        SDVariable a = inputs.get(GraphVertex.VERTEX_INPUT_PREFIX + "0");
        SDVariable b = inputs.get(GraphVertex.VERTEX_INPUT_PREFIX + "1");
        return a.mul(alpha).add(b.mul(beta));
    }
}
```

In a `ComputationGraph` configuration:

```java
ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
    .graphBuilder()
    .addInputs("input1", "input2")
    .addVertex("wsum", new WeightedAddVertex(0.6, 0.4), "input1", "input2")
    .addLayer("output", new OutputLayer.Builder(...).build(), "wsum")
    .setOutputs("output")
    .build();
```

---

## Traditional Approach: Extending BaseLayer

When the SameDiff approach is not suitable (e.g., you need explicit control over the backward pass, or you are porting a highly optimized kernel), you can implement a layer by extending DL4J's lower-level Java API.

This approach requires implementing **two** classes:

1. A **configuration class** that extends `org.deeplearning4j.nn.conf.layers.Layer`.
2. An **implementation class** that implements `org.deeplearning4j.nn.api.Layer`.

### Configuration Class

```java
import org.deeplearning4j.nn.conf.layers.FeedForwardLayer;
import org.deeplearning4j.nn.api.ParamInitializer;

public class MyLayerConf extends FeedForwardLayer {

    // Custom hyperparameters
    private double myAlpha;

    protected MyLayerConf() { }

    private MyLayerConf(Builder b) {
        super(b);
        this.myAlpha = b.myAlpha;
    }

    @Override
    public Layer instantiate(NeuralNetConfiguration conf,
                             Collection<TrainingListener> trainingListeners,
                             int layerIndex,
                             INDArray layerParamsView,
                             boolean initializeParams,
                             DataType networkDataType) {
        MyLayerImpl impl = new MyLayerImpl(conf, networkDataType);
        impl.setListeners(trainingListeners);
        impl.setIndex(layerIndex);
        impl.setParamsViewArray(layerParamsView);
        Map<String, INDArray> paramTable =
            initializer().init(conf, layerParamsView, initializeParams);
        impl.setParamTable(paramTable);
        impl.setConf(conf);
        return impl;
    }

    @Override
    public ParamInitializer initializer() {
        return DefaultParamInitializer.getInstance();
    }

    public static class Builder extends FeedForwardLayer.Builder<Builder> {
        private double myAlpha = 0.01;

        public Builder myAlpha(double alpha) {
            this.myAlpha = alpha;
            return this;
        }

        @Override
        public MyLayerConf build() {
            return new MyLayerConf(this);
        }
    }
}
```

### Implementation Class

```java
import org.deeplearning4j.nn.layers.BaseLayer;
import org.nd4j.linalg.primitives.Pair;

public class MyLayerImpl extends BaseLayer<MyLayerConf> {

    public MyLayerImpl(NeuralNetConfiguration conf, DataType dataType) {
        super(conf, dataType);
    }

    @Override
    public Pair<Gradient, INDArray> backpropGradient(INDArray epsilon,
                                                      LayerWorkspaceMgr workspaceMgr) {
        // epsilon: gradient from the layer above (dL/dOutput)
        // Must compute:
        //   - gradient w.r.t. this layer's parameters (stored in Gradient object)
        //   - gradient w.r.t. this layer's input (dL/dInput, returned as INDArray)

        INDArray input = this.input; // set by the training loop

        // --- compute dL/dW and dL/db ---
        INDArray dLdW = input.transpose().mmul(epsilon);
        INDArray dLdb = epsilon.sum(true, 0);

        Gradient gradient = new DefaultGradient();
        gradient.gradientForVariable().put("W", dLdW);
        gradient.gradientForVariable().put("b", dLdb);

        // --- compute dL/dInput to pass to layers below ---
        INDArray W = getParam("W");
        INDArray dLdInput = epsilon.mmul(W.transpose());

        return new Pair<>(gradient, dLdInput);
    }

    @Override
    public INDArray activate(boolean training, LayerWorkspaceMgr workspaceMgr) {
        INDArray W = getParam("W");
        INDArray b = getParam("b");
        INDArray z = input.mmul(W).addiRowVector(b);
        return Nd4j.getExecutioner().execAndReturn(
            new RectifedLinear(z, z, 0));  // relu in-place
    }

    @Override
    public boolean isPretrainLayer() { return false; }
}
```

Implementing `backpropGradient` correctly requires computing gradients analytically. This is error-prone. Use the gradient checker (see Testing, below) to verify your implementation.

---

## Registering Custom Layers for Serialization

DL4J uses JSON serialization for model save/load and Spark. Custom layer configuration classes must be registered so Jackson can deserialize them.

### Option 1: @JsonTypeInfo / @JsonSubTypes (automatic)

If your configuration class extends a DL4J abstract base that already has `@JsonTypeInfo` (e.g., `FeedForwardLayer`), your class may be discovered automatically via classpath scanning, depending on DL4J version. Check whether the annotation `@JsonSubTypes` on the parent class includes an explicit list; if so, you may need Option 2.

### Option 2: NeuralNetConfiguration Registry

Register your class before any save/load operations:

```java
NeuralNetConfiguration.registerLegacyCustomClassesForJSON(MyLayerConf.class);
```

This method accepts a varargs list of classes. Call it once at application startup (e.g., in a static initializer block or `main`).

### Verifying Serialization

Always test round-trip serialization before deploying:

```java
MultiLayerNetwork net = new MultiLayerNetwork(conf);
net.init();

// Save to JSON and restore
String json = net.getLayerWiseConfigurations().toJson();
MultiLayerConfiguration restored = MultiLayerConfiguration.fromJson(json);
MultiLayerNetwork restoredNet = new MultiLayerNetwork(restored);
restoredNet.init();
// Verify params are equal
assert net.params().equalsWithEps(restoredNet.params(), 1e-6);
```

---

## Testing Your Custom Layer

### 1. Serialization Test

Every custom layer must survive a JSON round-trip (required for model saving and Spark).

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .list()
    .layer(new MyLayerConf.Builder().nIn(10).nOut(5).build())
    .build();

String json    = conf.toJson();
MultiLayerConfiguration fromJson = MultiLayerConfiguration.fromJson(json);
assertEquals(conf, fromJson);
```

### 2. Gradient Check (Traditional Layers Only)

DL4J includes a `GradientCheckUtil` that performs numerical gradient checking (finite differences) against your analytical gradients.

```java
boolean passed = GradientCheckUtil.checkGradients(new GradientCheckUtil.MLNConfig()
    .net(net)
    .input(input)
    .labels(labels)
    .subset(true)
    .maxPerParam(50));
assertTrue("Gradient check failed", passed);
```

A relative error below 1e-5 is generally considered acceptable.

SameDiff-backed layers do not require manual gradient checking because backprop is computed automatically. Running the gradient check on them is still useful to verify that the forward pass is correctly implemented.

### 3. Output Shape Test

Confirm the layer produces the expected output shape:

```java
net.init();
INDArray input  = Nd4j.rand(32, 784);  // minibatch=32
INDArray output = net.output(input);
assertArrayEquals(new long[]{32, 10}, output.shape());
```

---

## Complete SameDiff Layer Example

The following example implements a simple gating layer that multiplies two linear projections element-wise (similar to a Gated Linear Unit).

```java
public class GluLayer extends SameDiffLayer {

    private int nIn;
    private int nOut;

    protected GluLayer() { }

    public GluLayer(int nIn, int nOut) {
        this.nIn  = nIn;
        this.nOut = nOut;
    }

    @Override
    public Map<String, long[]> defineParameterShape(InputType inputType,
                                                     int minibatchSize) {
        Map<String, long[]> m = new LinkedHashMap<>();
        m.put("W1", new long[]{nIn, nOut});
        m.put("W2", new long[]{nIn, nOut});
        m.put("b1", new long[]{1, nOut});
        m.put("b2", new long[]{1, nOut});
        return m;
    }

    @Override
    public void initializeParameters(Map<String, INDArray> params) {
        initWeights(nIn, nOut, WeightInit.XAVIER, params.get("W1"));
        initWeights(nIn, nOut, WeightInit.XAVIER, params.get("W2"));
        params.get("b1").assign(0.0);
        params.get("b2").assign(0.0);
    }

    @Override
    public SDVariable defineLayer(SameDiff sd,
                                  SDVariable input,
                                  Map<String, SDVariable> p,
                                  SDVariable mask) {
        SDVariable linear  = input.mmul(p.get("W1")).add(p.get("b1"));
        SDVariable gate    = sd.nn.sigmoid(input.mmul(p.get("W2")).add(p.get("b2")));
        return linear.mul(gate);
    }
}
```

This layer is immediately usable in any `MultiLayerConfiguration` and supports all standard DL4J features including model serialization, distributed training, and the training UI.

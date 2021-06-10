---
description: How to implement custom Keras layers for import in Deeplearning4J.
---

# Custom Layers

Many more advanced models will contain custom layers, i.e. layers that aren't included in Keras.

You can import those models too, but you will have to provide an implementation of that layer yourself, as the exported model file only provides us with a name for it.

Usually, you will have found out about needing to implement a custom layer, when you saw an exception like the following:

```text
org.deeplearning4j.nn.modelimport.keras.exceptions.UnsupportedKerasConfigurationException:
No SameDiff Lambda layer found for Lambda layer lambda_123. You can register a SameDiff Lambda layer using 
KerasLayer.registerLambdaLayer(lambdaLayerName, sameDiffLambdaLayer);
```

or

```text
org.deeplearning4j.nn.modelimport.keras.exceptions.UnsupportedKerasConfigurationException: 
Unsupported keras layer type LayerName.
```

## Implementing a custom layer for Keras import

There are two ways of implementing a custom layer for Keras import. Which one is the right approach for you, depends on the type of layer you need to implement.

1. `SameDiffLambdaLayer` Use this approach if your layer doesn't have any weights and defines just a computation. It is most useful when you have to define a custom layer because you are using a `lambda` in your model definition.   This is the approach you should be using when you've gotten the exception about no lambda layer being found. 
2. `KerasLayer` Use this approach if your layer needs its own weights. It is most useful when you have to define some complex layer that is more than just a simple computation.  This is the approach you should be using when you've gotten the exception about an unsupported layer type. 

### SameDiffLambdaLayer

Using a `SameDiffLambdaLayer` is pretty easy. You create a new class that extends it, and override the `defineLayer` and `getOutputType` methods.

```java
public class TimesThreeLambda extends SameDiffLambdaLayer {
    @Override
    public SDVariable defineLayer(SameDiff sd, SDVariable x) { 
        return x.mul(3); 
    }

    @Override
    public InputType getOutputType(int layerIndex, InputType inputType) {
        return inputType; 
    }
}
```

This simple lambda layer just multiplies its input by 3.

{% hint style="danger" %}
`defineLayer` will only be called once to create the SameDiff graph that is used as the definition of this layer. Do not use information about the size of the inputs or other non-static sizes, like batch size, when defining the layer, or it may fail later on.
{% endhint %}

After defining your layer, you have to register it to make it available on import.

```java
KerasLayer.registerLambdaLayer("lambda_2", new TimesThreeLambda());
```

The correct name for your lambda layer will depend on the model you are importing. As you, most likely, were made aware of needing to implement the lambda layer by an exception, this exception should have given you the proper name already.

### KerasLayer

Implementing a full layer with weights is more complex than defining a lambda layer. You will have to create a new class that extends `KerasLayer` and that reads the configuration of that layer and defines it appropriately.

For examples on how this was done, take a look at [KerasLRN](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/custom/KerasLRN.java) and [KerasPoolHelper](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/layers/custom/KerasPoolHelper.java) which are custom layers that were needed to be able to import GoogLeNet.

After you've defined your layer, you will have to register it to make it available on import:

```java
KerasLayer.registerCustomLayer("PoolHelper", KerasPoolHelper.class);
```

Again, the appropriate name will we apparent from the exception that has notified you about needing to implement the custom layer in the first place.


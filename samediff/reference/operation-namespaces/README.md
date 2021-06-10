# Operation Namespaces

All operations in ND4J and SameDiff are available in "Operation Namespaces". Each namespace is available on the `Nd4j` and `SameDiff` classes with its lowercase name.

For example, if you want to use the [absoluteDifference](loss.md#absolutedifference) operation it would look like this

```java
// ND4J mode
INDArray output = Nd4j.loss.absoluteDifference(labels, predictions, null);

// SameDiff mode
SDVariable output = SameDiff.loss.absoluteDifference(labels, predictions, null);
```

## Namespaces


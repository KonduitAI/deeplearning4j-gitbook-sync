---
description: Configure the Maven build tool for Deeplearning4j.
---

# Maven

## Configuring the Maven build tool

You can use Deeplearning4j with Maven by adding the following to your `pom.xml`:

```markup
<dependencies>
  <dependency>
      <groupId>org.deeplearning4j</groupId>
      <artifactId>deeplearning4j-core</artifactId>
      <version>1.0.0-M1.1</version>
  </dependency>
</dependencies>
```

The instructions below apply to all DL4J and ND4J submodules, such as `deeplearning4j-api`, `deeplearning4j-scaleout`, and ND4J backends.

## Add a backend

DL4J relies on ND4J for hardware-specific implementations and tensor operations. Add a backend by pasting the following snippet into your `pom.xml`:

```markup
<dependencies>
  <dependency>
      <groupId>org.nd4j</groupId>
      <artifactId>nd4j-native-platform</artifactId>
      <version>1.0.0-M1.1</version>
  </dependency>
</dependencies>
```

You can also swap the standard CPU implementation for [GPUs](configuration/backends/cudnn.md#cudnn-setup).

---
title: "Maven Setup"
description: "Maven dependencies for Deeplearning4j — BOM, backend selection, platform classifiers, and version management"
---

## Overview

Maven is the recommended build tool for Deeplearning4j projects. DL4J and ND4J publish a bill of materials (BOM) that makes version management straightforward, and platform-specific classifiers that bundle native binaries for all major operating systems in a single dependency.

This page covers the full Maven setup: BOM usage, core dependencies, backend selection, GPU dependencies, a complete example `pom.xml`, the classifier system, and how to resolve version conflicts.

## Using the DL4J BOM (Bill of Materials)

The BOM centralizes version management so you only specify the DL4J version in one place. Import it in the `<dependencyManagement>` section of your `pom.xml`:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.deeplearning4j</groupId>
            <artifactId>deeplearning4j-bom</artifactId>
            <version>1.0.0-M2.1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

With the BOM in place you can declare DL4J and ND4J dependencies without repeating the version:

```xml
<dependencies>
    <dependency>
        <groupId>org.deeplearning4j</groupId>
        <artifactId>deeplearning4j-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.nd4j</groupId>
        <artifactId>nd4j-native-platform</artifactId>
    </dependency>
</dependencies>
```

## Core Dependencies

### deeplearning4j-core

The main DL4J library. Includes `MultiLayerNetwork`, `ComputationGraph`, built-in layers, training infrastructure, and listeners.

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-core</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

### ND4J Backend (CPU)

DL4J relies on ND4J for all tensor operations. You must add exactly one ND4J backend. For CPU:

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native-platform</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

The `-platform` suffix bundles native binaries for Linux x86_64, Linux ARM64, macOS x86_64, macOS ARM64 (Apple Silicon), and Windows x86_64 in one JAR. See the [Platform Classifiers](#platform-classifiers) section for single-platform alternatives.

## GPU Dependencies

### ND4J CUDA Backend

To run on NVIDIA GPUs, replace `nd4j-native-platform` with the CUDA backend:

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.6-platform</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

CUDA toolkit 11.6 (or a compatible version) must be installed on the host. See the [GPU/CPU Setup](./gpu-cpu) page for the full CUDA installation guide.

### cuDNN Acceleration

To enable cuDNN for CNN and LSTM layers, add the cuDNN helper module alongside the CUDA ND4J backend:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-cuda-11.6</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

See the [cuDNN](./cudnn) page for details.

## Complete pom.xml Example

The following `pom.xml` is a minimal but complete starting point for a DL4J project using CPU:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-dl4j-project</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <dl4j.version>1.0.0-M2.1</dl4j.version>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.deeplearning4j</groupId>
                <artifactId>deeplearning4j-bom</artifactId>
                <version>${dl4j.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- Core DL4J library -->
        <dependency>
            <groupId>org.deeplearning4j</groupId>
            <artifactId>deeplearning4j-core</artifactId>
        </dependency>

        <!-- CPU backend (swap for nd4j-cuda-11.6-platform to use GPU) -->
        <dependency>
            <groupId>org.nd4j</groupId>
            <artifactId>nd4j-native-platform</artifactId>
        </dependency>

        <!-- Logging -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.11</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Build a fat JAR for deployment -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.4</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals><goal>shade</goal></goals>
                        <configuration>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

### Switching to GPU

To switch from CPU to GPU, replace the `nd4j-native-platform` dependency with:

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.6-platform</artifactId>
</dependency>
```

Using a property makes this easy to switch at build time:

```xml
<properties>
    <nd4j.backend>nd4j-native-platform</nd4j.backend>
    <!-- Change to nd4j-cuda-11.6-platform for GPU -->
</properties>

<dependencies>
    <dependency>
        <groupId>org.nd4j</groupId>
        <artifactId>${nd4j.backend}</artifactId>
    </dependency>
</dependencies>
```

## Platform Classifiers

DL4J's native libraries are distributed using [JavaCPP](https://github.com/bytedeco/javacpp) platform JARs. The `-platform` artifact is a multi-platform convenience wrapper that pulls in native binaries for all supported platforms.

If you are building and deploying on a single OS, you can use the non-platform artifact with an explicit classifier to reduce JAR size:

```xml
<!-- Linux x86_64 only -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>1.0.0-M2.1</version>
    <classifier>linux-x86_64</classifier>
</dependency>
```

Available classifiers:

| Classifier | Platform |
|---|---|
| `linux-x86_64` | Linux 64-bit (Intel/AMD) |
| `linux-arm64` | Linux ARM 64-bit (e.g., Graviton) |
| `macosx-x86_64` | macOS Intel |
| `macosx-arm64` | macOS Apple Silicon (M1/M2/M3) |
| `windows-x86_64` | Windows 64-bit |

When using classifiers for CUDA backends, you also need the CUDA-specific classifier:

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.6</artifactId>
    <version>1.0.0-M2.1</version>
    <classifier>linux-x86_64</classifier>
</dependency>
```

**Note:** Snapshot builds can have transient issues with `-platform` artifacts when cross-platform builds are not yet synchronized. In that case, using single-platform classifiers is more reliable. See the [Snapshots](./snapshots) page for more details.

## Additional Modules

Beyond the core, DL4J has several optional modules:

```xml
<!-- DL4J UI (training visualization) -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-ui</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>

<!-- Keras model import -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-modelimport</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>

<!-- DataVec (data loading and ETL) -->
<dependency>
    <groupId>org.datavec</groupId>
    <artifactId>datavec-api</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>

<!-- Arbiter (hyperparameter optimization) -->
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>arbiter-deeplearning4j</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

## Resolving Version Conflicts

DL4J pulls in several transitive dependencies — notably JavaCPP, OpenBLAS, and various BLAS implementations. If other libraries in your project use incompatible versions of these, Maven's dependency mediation rules (nearest-wins) may choose the wrong version.

### Symptoms of version conflicts

- `UnsatisfiedLinkError` when loading native libraries
- `NoSuchMethodError` from ND4J or JavaCPP classes
- Silent use of wrong backend (e.g., CPU when GPU expected)

### Diagnosing conflicts

Run Maven's dependency tree to inspect resolved versions:

```shell
mvn dependency:tree -Dincludes=org.nd4j:*,org.bytedeco:*
```

### Forcing consistent versions with the BOM

The DL4J BOM pins the versions of all ND4J and JavaCPP artifacts. Importing it (as shown above) is the most reliable way to avoid conflicts. If you must exclude a transitive dependency and pin it yourself:

```xml
<dependency>
    <groupId>org.bytedeco</groupId>
    <artifactId>javacpp</artifactId>
    <version>1.5.8</version> <!-- match what DL4J BOM specifies -->
</dependency>
```

### Excluding conflicting transitive dependencies

If a third-party library brings in an old version of a shared dependency, exclude it explicitly:

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>some-library</artifactId>
    <version>1.0</version>
    <exclusions>
        <exclusion>
            <groupId>org.bytedeco</groupId>
            <artifactId>javacpp</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## Java Version Requirements

DL4J 1.0.0-M2.1 requires Java 11 or later. Java 17 is supported. Set the compiler plugin accordingly:

```xml
<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
</properties>
```

## Related Pages

- [GPU and CPU Setup](./gpu-cpu) — backend selection and CUDA configuration
- [cuDNN](./cudnn) — cuDNN installation and configuration
- [Build Tools](./build-tools) — Gradle, SBT, and Leiningen setup
- [Snapshots](./snapshots) — using nightly builds

---
title: "Build Tools"
description: "Configuring Deeplearning4j with Gradle, SBT, and other build tools"
---

## Overview

While Maven is the primary build tool used and tested by the DL4J team, the library works with other JVM build systems. This page documents setup for Gradle (Groovy and Kotlin DSL), SBT, Leiningen (Clojure), and Ivy.

The core dependency pattern is the same across all tools: you need `deeplearning4j-core` and exactly one ND4J backend (`nd4j-native-platform` for CPU or `nd4j-cuda-11.6-platform` for GPU).

## Gradle

### Groovy DSL (build.gradle)

```groovy
plugins {
    id 'java'
}

repositories {
    mavenCentral()
}

ext {
    dl4jVersion = '1.0.0-M2.1'
}

dependencies {
    // Core DL4J
    implementation "org.deeplearning4j:deeplearning4j-core:${dl4jVersion}"

    // CPU backend (replace with nd4j-cuda-11.6-platform for GPU)
    implementation "org.nd4j:nd4j-native-platform:${dl4jVersion}"

    // Logging
    implementation 'ch.qos.logback:logback-classic:1.2.11'
}
```

### Kotlin DSL (build.gradle.kts)

```kotlin
plugins {
    java
}

repositories {
    mavenCentral()
}

val dl4jVersion = "1.0.0-M2.1"

dependencies {
    implementation("org.deeplearning4j:deeplearning4j-core:$dl4jVersion")
    implementation("org.nd4j:nd4j-native-platform:$dl4jVersion")
    implementation("ch.qos.logback:logback-classic:1.2.11")
}
```

### GPU with Gradle

Replace the CPU backend:

```groovy
// Groovy
implementation "org.nd4j:nd4j-cuda-11.6-platform:${dl4jVersion}"

// Add cuDNN helper if needed
implementation "org.deeplearning4j:deeplearning4j-cuda-11.6:${dl4jVersion}"
```

```kotlin
// Kotlin DSL
implementation("org.nd4j:nd4j-cuda-11.6-platform:$dl4jVersion")
implementation("org.deeplearning4j:deeplearning4j-cuda-11.6:$dl4jVersion")
```

### Additional Modules with Gradle

```groovy
dependencies {
    implementation "org.deeplearning4j:deeplearning4j-core:${dl4jVersion}"
    implementation "org.nd4j:nd4j-native-platform:${dl4jVersion}"

    // Optional: UI for training visualization
    implementation "org.deeplearning4j:deeplearning4j-ui:${dl4jVersion}"

    // Optional: Keras model import
    implementation "org.deeplearning4j:deeplearning4j-modelimport:${dl4jVersion}"

    // Optional: DataVec for data loading
    implementation "org.datavec:datavec-api:${dl4jVersion}"

    // Optional: Arbiter hyperparameter optimization
    implementation "org.deeplearning4j:arbiter-deeplearning4j:${dl4jVersion}"
}
```

### Dependency Resolution in Gradle

Gradle's dependency resolution differs from Maven's. If you encounter version conflicts:

```groovy
configurations.all {
    resolutionStrategy {
        // Force a specific JavaCPP version to match DL4J's expectation
        force 'org.bytedeco:javacpp:1.5.8'
    }
}
```

To exclude a transitive dependency:

```groovy
implementation("com.example:some-library:1.0") {
    exclude group: 'org.bytedeco', module: 'javacpp'
}
```

### Gradle and Snapshot Builds

Gradle has a known issue with snapshot artifacts that use Maven classifiers (the `-platform` artifacts). If you need snapshots, add the Sonatype snapshots repository and prefer non-platform classifiers with an explicit OS classifier:

```groovy
repositories {
    maven { url 'https://oss.sonatype.org/content/repositories/snapshots' }
    mavenCentral()
}

dependencies {
    implementation "org.deeplearning4j:deeplearning4j-core:1.0.0-SNAPSHOT"
    implementation "org.nd4j:nd4j-native:1.0.0-SNAPSHOT"
    // Add the OS-specific classifier explicitly
    implementation "org.nd4j:nd4j-native:1.0.0-SNAPSHOT:linux-x86_64"
}
```

See the [Snapshots](./snapshots) page for the full workaround. For production use, Maven is recommended for snapshot dependencies due to [a Gradle bug](https://github.com/gradle/gradle/issues/2882) with classifier-based snapshot resolution.

## SBT

### build.sbt

```scala
ThisBuild / scalaVersion := "2.13.10"
ThisBuild / version      := "0.1.0-SNAPSHOT"

lazy val dl4jVersion = "1.0.0-M2.1"

lazy val root = (project in file("."))
  .settings(
    name := "my-dl4j-project",
    libraryDependencies ++= Seq(
      "org.deeplearning4j" % "deeplearning4j-core"     % dl4jVersion,
      "org.nd4j"           % "nd4j-native-platform"    % dl4jVersion,
      "ch.qos.logback"     % "logback-classic"         % "1.2.11"
    )
  )
```

### SBT with GPU

```scala
libraryDependencies ++= Seq(
  "org.deeplearning4j" % "deeplearning4j-core"          % dl4jVersion,
  "org.nd4j"           % "nd4j-cuda-11.6-platform"      % dl4jVersion,
  "org.deeplearning4j" % "deeplearning4j-cuda-11.6"     % dl4jVersion
)
```

### SBT Resolvers for Snapshots

```scala
resolvers += "Sonatype Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"

lazy val dl4jVersion = "1.0.0-SNAPSHOT"
```

### SBT Assembly (fat JAR)

Add the sbt-assembly plugin to `project/plugins.sbt`:

```scala
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "2.1.1")
```

Then in `build.sbt`:

```scala
assembly / assemblyMergeStrategy := {
  case PathList("META-INF", _*) => MergeStrategy.discard
  case _                        => MergeStrategy.first
}
```

Run with `sbt assembly` to produce a fat JAR.

**Note:** SBT, like Gradle, requires explicit openblas dependencies when using `nd4j-native` without the `-platform` wrapper. The `-platform` artifact handles this automatically. See the [nd4j-native-platform pom](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-backend-impls/nd4j-native-platform/pom.xml) to check the required openblas version if you manage dependencies manually.

## Leiningen (Clojure)

[Leiningen](https://leiningen.org/) uses a Clojure-based build file. Add DL4J via the `:dependencies` vector in `project.clj`:

```clojure
(defproject my-dl4j-project "0.1.0-SNAPSHOT"
  :description "DL4J Clojure project"
  :dependencies [[org.clojure/clojure "1.11.1"]
                 [org.deeplearning4j/deeplearning4j-core "1.0.0-M2.1"]
                 [org.nd4j/nd4j-native-platform "1.0.0-M2.1"]
                 [ch.qos.logback/logback-classic "1.2.11"]]
  :jvm-opts ["-Xmx4g"
             "-Dorg.bytedeco.javacpp.maxbytes=6g"]
  :main ^:skip-aot my-dl4j-project.core
  :target-path "target/%s"
  :profiles {:uberjar {:aot :all}})
```

### Leiningen with GPU

```clojure
:dependencies [[org.deeplearning4j/deeplearning4j-core "1.0.0-M2.1"]
               [org.nd4j/nd4j-cuda-11.6-platform "1.0.0-M2.1"]
               [org.deeplearning4j/deeplearning4j-cuda-11.6 "1.0.0-M2.1"]]
```

### Leiningen Snapshots

```clojure
:repositories [["sonatype-snapshots" "https://oss.sonatype.org/content/repositories/snapshots"]]
:dependencies [[org.deeplearning4j/deeplearning4j-core "1.0.0-SNAPSHOT"]
               [org.nd4j/nd4j-native-platform "1.0.0-SNAPSHOT"]]
```

**Note:** After downloading via Leiningen, you may need to double-click the downloaded JAR files to register them in your local Maven repository if you are working with Eclipse or another IDE.

## Ivy

For Ant-based builds using Ivy, add the following to your `ivy.xml`:

```xml
<ivy-module version="2.0">
    <info organisation="com.example" module="my-dl4j-project"/>
    <dependencies>
        <dependency org="org.deeplearning4j"
                    name="deeplearning4j-core"
                    rev="1.0.0-M2.1"
                    conf="compile->default"/>
        <dependency org="org.nd4j"
                    name="nd4j-native-platform"
                    rev="1.0.0-M2.1"
                    conf="compile->default"/>
    </dependencies>
</ivy-module>
```

Add the Maven Central resolver to `ivysettings.xml`:

```xml
<ivysettings>
    <settings defaultResolver="chain"/>
    <resolvers>
        <chain name="chain">
            <ibiblio name="central" m2compatible="true"/>
        </chain>
    </resolvers>
</ivysettings>
```

## Dependency Resolution Tips

### Verify which backend is loaded

Regardless of build tool, add this to your main method to confirm the correct ND4J backend is active:

```java
System.out.println(Nd4j.getBackend().getClass().getName());
// CPU: org.nd4j.linalg.cpu.nativecpu.CpuBackend
// GPU: org.nd4j.linalg.jcublas.JCublasBackend
```

### Common build issues

**Issue:** `UnsatisfiedLinkError` or missing native library at runtime.
**Cause:** The platform classifier was not resolved, or openblas is missing.
**Fix:** Use the `-platform` artifact, or manually add the openblas dependency matching the version in the [nd4j-native-platform pom](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-backend-impls/nd4j-native-platform/pom.xml).

**Issue:** Multiple backends on classpath (both `nd4j-native` and `nd4j-cuda` present).
**Cause:** Transitive dependency pulled in an unwanted backend.
**Fix:** Exclude the unwanted backend explicitly in your build file.

**Issue:** Wrong Java version.
**Cause:** DL4J 1.0.0-M2.1 requires Java 11+.
**Fix:** Set `sourceCompatibility = JavaVersion.VERSION_11` in Gradle or `<maven.compiler.source>11</maven.compiler.source>` in Maven.

## Related Pages

- [Maven Setup](./maven) — full Maven pom.xml reference
- [GPU and CPU Setup](./gpu-cpu) — backend selection
- [Snapshots](./snapshots) — using nightly builds

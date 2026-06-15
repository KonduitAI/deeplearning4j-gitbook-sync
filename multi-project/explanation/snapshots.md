---
title: Snapshot Builds
description: Using nightly snapshot builds of Eclipse Deeplearning4j 1.0.0-M2.1 — repository configuration, version identifiers, Maven and Gradle setup.
---

## Overview

Snapshot builds of Eclipse Deeplearning4j are published automatically after each successful CI build. They include the latest bug fixes, new operations, and experimental features ahead of the next stable release. Snapshots are served from the Sonatype OSS snapshot repository rather than Maven Central.

**Stability warning:** Snapshots are development builds. Breaking changes or regressions may be introduced at any point. Use the latest stable release in production; use snapshots only when you need a specific fix or feature that has not yet been released.

---

## Snapshot Version Identifier

The current snapshot version is:

```
1.0.0-SNAPSHOT
```

This tracks the development branch for the next release after 1.0.0-M2.1.

---

## Repository Configuration

### Maven

Add the Sonatype snapshot repository to your `pom.xml`:

```xml
<repositories>
    <repository>
        <id>sonatype-snapshots</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        <releases>
            <enabled>false</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
            <updatePolicy>daily</updatePolicy>
        </snapshots>
    </repository>
</repositories>
```

Then set the version property:

```xml
<properties>
    <dl4j.version>1.0.0-SNAPSHOT</dl4j.version>
    <nd4j.version>1.0.0-SNAPSHOT</nd4j.version>
</properties>
```

### Gradle

```groovy
repositories {
    maven {
        url "https://oss.sonatype.org/content/repositories/snapshots"
    }
    mavenCentral()
}

ext {
    dl4jVersion = '1.0.0-SNAPSHOT'
}

dependencies {
    implementation "org.deeplearning4j:deeplearning4j-core:${dl4jVersion}"
    implementation "org.nd4j:nd4j-native-platform:${dl4jVersion}"
}
```

> **Gradle and classifiers:** Due to a [known Gradle bug](https://github.com/gradle/gradle/issues/2882), snapshot dependencies that include Maven classifiers (e.g., platform artifacts with OS classifiers) do not resolve correctly in Gradle. If you encounter resolution failures with `-platform` artifacts, use Maven or manually list the OS-specific artifacts.

---

## Sample Minimal pom.xml (CPU)

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>dl4j-snapshot-test</artifactId>
    <version>1.0</version>

    <properties>
        <dl4j.version>1.0.0-SNAPSHOT</dl4j.version>
    </properties>

    <repositories>
        <repository>
            <id>sonatype-snapshots</id>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
            <snapshots><enabled>true</enabled></snapshots>
            <releases><enabled>false</enabled></releases>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.deeplearning4j</groupId>
            <artifactId>deeplearning4j-core</artifactId>
            <version>${dl4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.nd4j</groupId>
            <artifactId>nd4j-native-platform</artifactId>
            <version>${dl4j.version}</version>
        </dependency>
    </dependencies>
</project>
```

---

## ND4J Backend Configuration for Snapshots

The backend is selected by the dependency you include, exactly as with stable releases.

**CPU backend:**

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native-platform</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

**GPU backend (CUDA 11.6):**

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.6-platform</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

If you need cuDNN integration with a snapshot build:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-cuda-11.6</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

---

## Forcing Snapshot Updates

By default, Maven checks for new snapshots based on the `updatePolicy` set in the repository configuration. To force Maven to download the latest snapshot immediately:

```shell
mvn package -U
```

The `-U` flag forces Maven to check for new snapshot versions regardless of the update policy.

To prevent Maven from checking for updates (useful in offline or CI cache scenarios):

```shell
mvn package -nsu
```

`-nsu` (no snapshot updates) requires that the snapshot artifact is already in the local `.m2` cache. The build will fail if the snapshot is not cached.

---

## Platform Artifacts and Build Timing

Because snapshot builds are produced for multiple platforms (Linux x86_64, Linux ARM64, macOS x86_64, macOS ARM64, Windows x86_64) and not all platform builds complete simultaneously, `-platform` artifacts can temporarily become inconsistent. This may cause build errors such as missing classifiers.

If you are building and deploying to a single known platform, use the single-platform artifact to avoid this:

```xml
<!-- Linux x86_64 only, avoids multi-platform sync issues -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
<!-- plus the platform-specific native binary: -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <classifier>linux-x86_64</classifier>
</dependency>
```

---

## Snapshot Limitations

| Limitation | Detail |
|---|---|
| No semantic versioning guarantee | Snapshots may contain breaking API changes |
| Gradle classifiers | Platform artifacts with OS classifiers may not resolve due to a Gradle bug |
| Temporary build failures | Snapshots can be unavailable for short periods during CI runs |
| Not recommended for production | Use a stable release (`1.0.0-M2.1`) in production environments |

---

## Checking Which Snapshot You Have

To see the exact snapshot build date and commit hash:

```shell
# Look at the artifact jar manifest or check the Sonatype repository browser
mvn dependency:list -Dincludes=org.nd4j -Dverbose
```

Or browse the Sonatype snapshot repository directly:

```
https://oss.sonatype.org/content/repositories/snapshots/org/nd4j/nd4j-native/
```

Each snapshot sub-version shows a timestamp in the format `1.0.0-YYYYMMDD.HHMMSS-N`.

---

## Related Pages

- [Maven Setup](./maven) — stable release dependency configuration
- [Build Tools](./build-tools) — Gradle and other build tool configuration
- [GPU and CPU Setup](./gpu-cpu) — backend selection

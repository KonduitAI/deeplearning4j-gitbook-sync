---
description: DL4J and Javacpp
---

# Javacpp

## DL4J and Javacpp overview

DL4J heavily depends on [javacpp](https://github.com/bytedeco/javacpp) for its interop between java and platform optimized c++ libraries. However, due to our usage of JNI this comes with certain complexities in the build anyone should be aware of.

The following modules rely on javacpp as part of their build process: 1. nd4j-native 2. nd4j-native-presets 3. nd4j-cuda 4. nd4j-cuda-presets

Each of these libraries are what comprise our nd4j backends. Leveraging \[libnd4j], javacpp handles linking each nd4j-backend against the libnd4j c++ codebase. This linking is done using a libnd4j home. This will contain all of the include files and necessary binary files for specific platforms. By default, nd4j backends and the libnd4j code base are compiled within the same build step. This is the recommended default, but for specific circumstances. A libnd4j release is also uploaded to maven central as a zip file and can be used in place of libnd4j compilation. See our [Github actions overview libnd4jUrl parameter](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/multi-project/how-to-guides/developer-docs/github-actions-overview.md) for more information on this.

Each backend consists of 2 modules

1. The codebase: This represents the actual nd4j backend logic for specific platforms. Conceptually, this logic will be anything that a developer should need to control such as memory management, environment variables, or other execution logic.
2. The presets: This is a similar concept in spirit to the [official javacpp presets](https://github.com/bytedeco/javacpp-presets) In order to avoid a race condition between the backend and the presets compilation, this is a separate dependency that just exists to handle interop between the libnd4j code base and the java frontend. The above backend then contains the rest of the logic needed for execution of the math operations on specific platforms.

## Compilation flow

After a libnd4j build is executed for a specific platform, we need to leverage javacpp to actually link against libnd4j to create a complete libnd4j backend. When invoking a maven build, the [javacpp maven plugin](http://bytedeco.org/javacpp/apidocs/org/bytedeco/javacpp/tools/BuildMojo.html) is used to actually invoke a build. The presets will be compiled first. Generally the presets are just 1 or 2 classes containing a description of how to map the actual nd4j code base to the libnd4j codebase.

Next, the actual backend is compiled with a dependency on the above presets code base. The javacpp plugin will leverage the description from the presets we specify as a dependency and facilitate linking against a LIBND4J\_HOME (a folder which contains the platform specific libnd4j binaries and include sources) specified by the user. In the actual plugin declaration on the backend pom.xml we include the target presets class to use for our particular backend.

Note: This still requires the native platform specific tools to be installed since binaries are generated for each platform. Please see our github actions for instructions on specific platforms.

## -platform dependencies

Nd4j reuses javacpp's notion of a -platform library. This is a curated set of dependencies most users will use as part of a build. Each backend will have an associated -platform artifact so users don't have to deal with maven classifiers. See [docs from javacpp](https://github.com/bytedeco/javacpp-presets/wiki/Reducing-the-Number-of-Dependencies) for how to leverage this artifact.

Caution to users: By default, this means that a large number of dependencies for all platforms will be included. If you do not need dependencies for all platforms, then please read the above documentation to figure out how to build a jar for your specific platform.

Generally, the main thing to know is when you build your application, use:

```bash
mvn -Djavacpp.platform=your-target-platform
```

A comprehensive list of classifiers can be found [here](https://repo1.maven.org/maven2/org/nd4j/nd4j-native/1.0.0-beta7/) Note that each library we link against such as [openblas](https://repo1.maven.org/maven2/org/bytedeco/openblas/0.3.13-1.5.5/) will also have a similar set of classifiers.

## Javacpp platform specific profiles

Throughout the dl4j pom.xml files, platform specific profiles that setup dependencies exist. An [example](https://github.com/eclipse/deeplearning4j/blob/821a0a4727bfd5bc312723b8864330566706bf9b/pom.xml#L1096) can be found here. This helps us dynamically figure out which platform someone is building for.

## Running javacpp on termux + android/lineagos

A testing setup the team uses for testing android involves lineageos, termux, and some arm32 based open jdk debian files that can be found [here](https://ia802807.us.archive.org/19/items/openjdk-9-jre-headless\_9.2017.8.20-1\_x86\_64)

In order to bootstrap this environment, a from scratch install of the latest lineageos flashed on an sd card using the raspberry pi is suggested.

Afterwards, install

In order to properly setup the test environment,

you need to execute your test from the command line as follows:

```bash
mvn -DargLine="org.bytedeco.javacpp.pathsfirst=true -Djavacpp.platform=android-arm" -Dorg.bytedeco.javacpp.pathsfirst=true -Djavcpp.platform=android-arm clean test
```

A proper execution environment after the above jdk is installed involves manually setting the environment as follows:

```bash
export JAVA_HOME=/data/data/com.termux/files/usr/lib/jvm/openjdk-9
export PATH=$PATH:$HOME/apache-maven-3.8.1/bin
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$JAVA_HOME/lib/:$JAVA_HOME/lib/jli"
export MAVEN_OPTS="-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=ture -Dmaven.wagon.http.ssl.ignore.validity.dates=true"
```

This will setup the jdk + maven to ignore ssl errors due to issues with cacerts + termux. This is largely irrelevant for our small testing use case, but not recommended for production environments.

## Redist artifacts

Redist artifacts are easy ways of distributing dependencies without installation.

Note that for the presets that are part of nd4j (nd4j-cuda-presets and nd4j-native-presets) only the latest versions support redist artifacts. The presets preload versions only support pre loading (eg: linking against libraries from the javacpp cache) against the latest version. This is because during pre loading, certain version numbers are checked for.

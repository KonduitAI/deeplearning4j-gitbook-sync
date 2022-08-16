# Import in to your favorite IDE

## Pre requisites

Ensure that you clone the deeplearning4j project locally.

```
git clone https://github.com/eclipse/deeplearning4j
```

Before importing the project, a few things of note no matter what IDE  you use:

1. One submodule (libnd4j) is a c++ project that uses maven to invoke a cmake build. You may wish to edit libnd4j separately in a cmake oriented IDE like VS Code, Clion, or Eclipse c/c++. In order to build a particular nd4j backend, libnd4j should already be compiled. By default, relevant nd4j backends all look for a pre compiled libnd4j in the libnd4j directory included within the same project.
2. Maven profiles for deeplearning4j matter a lot. Especially if you want to run tests. Read more on the test profiles [here](developer-docs/testing.md#parameters-for-testing). For most code nd4j-tests-cpu should probably be the main profile you use.
3. Deeplearning4j uses lombok for its dependencies. Ensure you install lombok for your favorite IDE in order to use the project. Please follow the [**baeldung guide** ](https://www.baeldung.com/lombok-ide)for setting this up in your IDE.

## Intellij

Once cloned locally, open intellij.  Please follow the guide [here ](https://www.jetbrains.com/help/idea/maven-support.html#maven\_import\_project\_start)to import from external maven sources.

Once imported, please give the project time to download associated dependencies. You can verify the status of the project in the bottom right corner.

In order to enable the project to work, the following modifications need to be made.

### Shaded modules

Eclipse Deeplearning4j has a set of shaded modules. Shaded modules are artifacts that re namespace a dependency to a different location in order to use it as a set of private dependencies that do not clash with other libraries that may also share the dependency.

Intellij does not handle this very well. In order to work around this, you need to exclude all projects under the nd4j/nd4j-shade folder individually. Right click on each folder. Go to Maven -> Ignore Projects.

Assuming you follow the other steps above (lombok,libdn4j,..) then you should be able to run any module you want.

## Eclipse

Note: for now the latest version of eclipse appears to fail upon first import. Any suggestions maybe reported on the [community forums](https://community.konduit.ai/).&#x20;

Once cloned locally, open eclipse. Please follow the guide [here ](https://dzone.com/articles/importing-a-maven-project-in-eclipse-1)to import from external maven sources. Importing your project in to eclipse may take a while. Of note is due to the profile sensitive nature of the deeplearning4j suite, there maybe issues when opening and building the project.

When first finishing import of the project, a number of maven connector errors should be highlighted. Afterwards, just click resolve all later and finish. Let eclipse finish downloading sources and javadoc.

As of the latest version of eclipse, build errors may occur.

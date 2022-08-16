---
description: Github actions Configuration Overview
---

# Github Actions/Build Infra

## Overview of a Github Actions Configuration

Each [github actions workflow](https://github.com/eclipse/deeplearning4j/tree/master/.github/workflows) has 10 parameters for manually invoking builds. The reason this is manual is due to the different ways a release can break. Being manual also allows us to re invoke only the parts of a build we need, rather than the whole release pipeline.

Most workflows implement a matrix structure for handling different combinations of builds related to the following: 1. Platform specific optimizations: On windows/linux/mac we allow cpu + optional linking against mkldnn. Each combination is enumerated and ran as part of a matrix build on github actions.

1. Cuda, optional cudnn: We also allow optional linking against cudnn for gpu routines.

## Input parameters:

1. buildThreads: This is the number of builds threads used for compilation in linbnd4j. This is the equivalent of make -j. For specific platforms that use more memory, 1 is the recommended value. On self hosted setups, you may use more threads to make builds run faster.
2. deployToReleaseStaging: 0 or 1. If 1, this will create a staging repository on oss sonatype. Otherwise, it will deploy to ossrh snapshots. Snapshots is the default.
3. releaseVersion: This is the intended release version to be converted to from snapshots. The [update-versions.sh](https://github.com/eclipse/deeplearning4j/blob/19ebc1e774c125359d672fb24103048276d417a1/update-versions.sh) script is run converting the versions of every module to that specific version intended for release. This is what will get uploaded to a staging repository for release. Otherwise, all intended versions should be SNAPSHOT.
4. snapshotVersion: The current in development snapshot version
5. releaseRepoId: If blank, then a new staging repository for a version is created. Otherwise, a staging repository id should be obtained from the ossrh nexus sonatype. This releaseRepoId should be passed to subsequent builds so all of the artifacts associated with a version get propagated to 1 place.
6. serverId: This should be ossrh 90% of the time. A github profile is also available for use with github actions.
7.  modules: The maven modules to build. This is fairly raw and error prone. The intended usage is with the [-pl/--projects flag](https://maven.apache.org/guides/mini/guide-multiple-modules) Typical usage is to skip libnd4j builds with something like:

    ```bash
    --pl !libnd4j
    ```

    to skip a libnd4j compile. This can speed builds up significantly.
8. libnd4jDownload/libnd4jUrl: In tandem with modules, you can specify a libnd4j zip file distribution that was compiled before for download. The builds will download a libnd4j distribution and use that for linking. This can be handy when recompiling the nd4j-native/nd4j-cuda backends for a specific platform without needing to recompile the whole c++ codebase. A url in a matrix build will be sourced from a hard coded file name from [this repo](https://github.com/KonduitAI/gh-actions-libnd4j-urls) - each file name will be updated to point to a zip file distribution appropriate for an individual matrix build. This was done because 1 url is not going to be suitable for individual matrix builds.
9. runsOn: This is the operating system upon which to run the build. For linux, this defaults to ubuntu-16.04. For windows, windows-2019. self-hosted can also be specified for faster builds.

## Matrix builds

Many configurations on cpu and cuda require a matrix based build structure to capture the various combinations of optimization and software versions people may want to use. In order to accomodate these workflows, we need to attach variables proxying the values of the manual inputs to the individual matrix workers themselves. These parameters are analogous to the above described parameters. Note we will not repeat the descriptions here, but the values can be seen from their values in the form of $ where SOME\_VALUE is one of the values above.

The configuration to look for is as follows:

```
          - mvn_ext: ${{ github.event.inputs.mvnFlags }}
            experimental: true
            name: Extra maven flags

          - debug_enabled: ${{ github.event.inputs.debug_enabled }}
            experimental: true
            name: Debug enabled

          - runs_on: ${{ github.event.inputs.runsOn }}
            experimental: true
            name: OS to run on

          - libnd4j_file_download: ${{ github.event.inputs.libnd4jDownload }}
            experimental: true
            name: OS to run on

          - deploy_to_release_staging: ${{ github.event.inputs.deployToReleaseStaging }}
            experimental: true
            name: Whether to deploy to release staging or not

          - release_version: ${{ github.event.inputs.releaseVersion }}
            experimental: true
            name: Release version

          - snapshot_version: ${{ github.event.inputs.snapshotVersion }}
            experimental: true
            name: Snapshot version

          - server_id: ${{ github.event.inputs.serverId }}
            experimental: true
            name: Server id

          - release_repo_id: ${{ github.event.inputs.releaseRepoId }}
            experimental: true
            name: The release repository to run on

          - mvn_flags: ${{ github.event.inputs.mvnFlags }}
            experimental: true
            name: Extra maven flags to use as part of the build

          - build_threads: ${{ github.event.inputs.buildThreads }}
            experimental: true
            name: The number of threads to build libnd4j with
```

## Expected timings

1. CUDA: Most cuda builds take 4-5 hours. Both windows and linux on GH actions just download the cuda distribution and compile things on their respective platforms.
2. CPU builds: From scratch libnd4j + cpu builds typically take 1-2 hours max. Anything more than that, your build may have something wrong.

## Build error causes

1. Out of disk: It is very common for a github actions VM to run out of disk. If a build fails with no logs after and all steps terminated, this maybe one of the reasons.
2. Out of memory: Sometimes builds run out of memory. A few common causes include:
   * Clang out of memory on android, depending on the number of builds threads assigned, it is easy for clang to run out of memory
   * Maven javadoc: The maven javadoc plugin for bigger projects can use a ton of ram and crash a job
3. Network failures: Maven can sometimes (rarely) fail to download certain dependencies in the middle of a job

## Environment variables:

1. MAVEN\_GPG\_KEY: The maven gpg key secret for a release
2.  CROSS\_COMPILER\_DIR: For the pi\_build.sh script in libnd4j. This contains the root directory

    for cross compiler invocation. We need this because all cross compilation for various libnd4j builds happens

    on x86. We cross compile for speed reasons also easily allowing us to run on github actions.
3. Debian frontend: This is to ensure that all debian commands by default don't prompt for yes/no
4. GITHUB\_TOKEN: This is for authentication with github actions
5.  BUILD\_USING\_MAVEN: This is for pi\_build.sh. This toggles (0 or 1) whether to use maven or buildnativeoperation.sh

    in the libnd4j root directory directly.
6. NDK\_VERSION: Default is r21d. Libnd4j's android is compiled with the android r21 currently.
7. CURRENT\_TARGET: This variable is for pi\_build.sh. It tells pi\_build.sh which architecture to build for.
8.  PUBLISH\_TO: The repo to publish to for releases or snapshots. Valid values are github or ossrh.

    These are repositories defined in the deeplearning4j root pom.
9.  OPENBLAS\_PATH: We compile libnd4j against openblas for several different cpus. Openblas is manually downloaded and linked against.&#x20;

    This specifies the path to the download for the libnd4j cmake invocation.
10. MAVEN\_USERNAME: The user name to login to for the ossrh maven repository
11. MAVEN\_PASSWORD: The password to login to for the ossrh maven repository
12. MAVEN\_GPG\_PASSPHRSE: The gpg password for signing artifacts for uploading to maven central
13. DEPLOY\_TO> Valid values are either ossrh or github.&#x20;
14. LIBND4J\_BUILD\_THREADS: This is the equivalent of make -j. It specifies the number of threads

    that should be used to compile libnd4j
15. PERFORM\_RELEASE: Whether to perform a release or not (0 or 1)
16. RELEASE\_VERSION: The version to be released to maven central. change-versions.sh will be run

    to change versions throughout the code base from the snapshot verison to the intended release version.
17. SNAPSHOT\_VERSION: The current snapshot version to be changed when performing a release.

    After a release is conducted, this should generally be the next development version.
18. RELEASE\_REPO\_ID: Leave this empty when first creating a release repository in combination with&#x20;

    DEPLOY set to 1. Afterwards, note which staging repository id gets created in the ossrh interface when publishing

    to maven central. Use that id for further buidls to ensure that all uploads for 1 version are synchronized to 1 staging repository.
19. MODULES: Extra maven flags for pi\_build.sh if more flags are needed (such as for debugging or only building specific modules)
20. LIBND4J\_URL: Used when building nd4j-native. If a user does not want to recompile libnd4j for their particular build, you can instead

    skip this step and specify a libnd4j zip file download (generally built with the maven assembly plugin)

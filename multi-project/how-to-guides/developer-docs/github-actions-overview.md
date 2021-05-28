---
title: 'Github actions Configuration Overview'
short_title: Github actions Configuration Overview
description: 'Github actions Configuration Overview'
category: Developer
weight: 3
---
# Overview of a Github Actions Configuration

Each [github actions workflow](https://github.com/eclipse/deeplearning4j/tree/master/.github/workflows)
has 10 parameters for manually invoking builds. The reason this is manual is due to the different ways a release can break.
Being manual also allows us to re invoke only the parts of a build we need, rather than the  whole release pipeline.

Most workflows implement a matrix structure for handling different combinations of builds related to the following:
1. Platform specific optimizations: On windows/linux/mac we allow cpu + optional linking against mkldnn. Each combination is enumerated and ran as part of a matrix build on github actions.

2. Cuda, optional cudnn: We also allow optional linking against cudnn for gpu routines.



# Input parameters:

1. buildThreads: This is the number of builds threads used for compilation in linbnd4j. This is the equivalent of make -j. For specific platforms
that use more memory, 1 is the recommended value. On self hosted setups, you may use more threads to make builds run faster.

2. deployToReleaseStaging: 0 or 1. If 1, this will create a staging repository on oss sonatype. Otherwise, it will deploy to ossrh snapshots.
Snapshots is the default.

3. releaseVersion: This is the intended release version to be converted to from snapshots. The [update-versions.sh](https://github.com/eclipse/deeplearning4j/blob/19ebc1e774c125359d672fb24103048276d417a1/update-versions.sh) script is run converting the versions of every module to that specific version intended for release.
This is what will get uploaded to a staging repository for release. Otherwise, all intended versions should be SNAPSHOT.

4. snapshotVersion: The current in development snapshot version

5. releaseRepoId: If blank, then a new staging repository for a version is created. Otherwise, a staging repository id should be obtained from the ossrh
nexus sonatype. This releaseRepoId should be passed to subsequent builds so all of the artifacts associated with a version get propagated to 1 place.

6. serverId: This should be ossrh 90% of the time. A github profile is also available for use with github actions.

7. modules: The maven modules to build. This is fairly raw and error prone. The intended usage is with the [-pl/--projects flag](https://maven.apache.org/guides/mini/guide-multiple-modules) Typical usage is to skip libnd4j builds with something like: 
```bash
--pl !libnd4j
```
to skip a libnd4j compile. This can speed builds up significantly. 

8. libnd4jUrl: In tandem with modules, you can specify a libnd4j zip file distribution that was compiled before for download. The builds will download a libnd4j 
distribution and use that for linking. This can be handy when recompiling the nd4j-native/nd4j-cuda backends for a specific platform without needing to recompile
the whole c++ codebase.

9. runsOn: This is the operating system upon which to run the build. For linux, this defaults to ubuntu-16.04. For windows, windows-2019.
self-hosted can also be specified for faster builds.



# Expected timings

1. CUDA: Most cuda builds take 4-5 hours. Both windows and linux on GH actions just download the cuda distribution and compile things
on their respective platforms.

2. CPU builds: From scratch libnd4j + cpu builds typically take 1-2  hours max. Anything more than that, your build may have something wrong.


# Build error causes

1. Out of disk: It is very common for a github actions VM to run out of disk.  If a build fails with no logs after and
all steps terminated, this maybe one of the reasons.

2. Out of memory: Sometimes builds run out of memory. A few common causes include:
   * Clang out of memory on android, depending on the number of builds threads assigned, it is easy for clang to run out of memory
   * Maven javadoc: The maven javadoc plugin for bigger projects can use a ton of ram and crash a job

3. Network failures: Maven can sometimes (rarely) fail to download certain dependencies in the middle of a job 
 


# Environment variables:
1. MAVEN_GPG_KEY: The maven gpg key secrete for a release
2. CROSS_COMPILER_DIR: For the pi_build.sh script in libnd4j. This contains the root directory
for cross compiler invocation. We need this because all cross compilation for various libnd4j builds happens
on x86. We cross compile for speed reasons also easily allowing us to run on github actions.
3. Debian frontend: This is to ensure that all debian commands by default don't prompt for yes/no
4. GITHUB_TOKEN: This is for authentication with github actions
5. BUILD_USING_MAVEN: This is for pi_build.sh. This toggles (0 or 1) whether to use maven or buildnativeoperation.sh
in the libnd4j root directory directly.
6. NDK_VERSION: Default is r21d. Libnd4j's android is compiled with the android r21 currently.
7. CURRENT_TARGET: This variable is for pi_build.sh. It tells pi_build.sh which architecture to build for.
8. PUBLISH_TO: The repo to publish to for releases or snapshots. Valid values are github or ossrh.
These are repositories defined in the deeplearning4j root pom.
9. OPENBLAS_PATH: We compile libnd4j against openblas for several different cpus. Openblas is manually downloaded and linked against. 
This specifies the path to the download for the libnd4j cmake invocation.
10. MAVEN_USERNAME: The user name to login to for the ossrh maven repository
11. MAVEN_PASSWORD: The password to login to for the ossrh maven repository
12. MAVEN_GPG_PASSPHRSE: The gpg password for signing artifacts for uploading to maven central
13. DEPLOY_TO> Valid values are either ossrh or github. 
14. LIBND4J_BUILD_THREADS: This is the equivalent of make -j. It specifies the number of threads
that should be used to compile libnd4j
15. PERFORM_RELEASE: Whether to perform a release or not (0 or 1)
16. RELEASE_VERSION: The version to be released to maven central. change-versions.sh will be run
to change versions throughout the code base from the snapshot verison to the intended release version.
17. SNAPSHOT_VERSION: The current snapshot version to be changed when performing a release.
After a release is conducted, this should generally be the next development version.
18. RELEASE_REPO_ID: Leave this empty when first creating a release repository in combination with 
DEPLOY set to 1. Afterwards, note which staging repository id gets created in the ossrh interface when publishing
to maven central. Use that id for further buidls to ensure that all uploads for 1 version are synchronized to 1 staging repository.
19. MODULES: Extra maven flags for pi_build.sh if more flags are needed (such as for debugging or only building specific modules)
20. LIBND4J_URL: Used when building nd4j-native. If a user does not want to recompile libnd4j for their particular build, you can instead
skip this step and specify a libnd4j zip file download (generally built with the maven assembly plugin)
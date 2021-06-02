---
title: 'How to conduct a release to Maven Central'
short_title: Conducting a release to Maven Central
description: 'How to conduct a release to Maven Central'
category: Developer
weight: 3
---
# Test Execution

## Parameters for testing
1. test.heap.size: The heap size used for maven surefire plugin sub processes
2. test.offheap.size: The off heap size used for maven surefire sub processes. This is very important for 
configuration (especially on gpu systems)


## Test resources
In order to run the deeplearning4j tests, many pretrained models and other resources are required.
Ensure  [dl4j test resources](https://github.com/KonduitAI/dl4j-test-resources) as a dependency on your classpath.
It is a big repository that needs to be mvn clean installed in order to run the tests properly.
You can do this by adding -Ptestresources to your test execution when running the tests from maven.


## Test profiles for enabling nd4j backends
When running deeplearning4j's tests, there are 2 main profiles to be aware of: nd4j-tests-cpu and nd4j-tests-cuda.
These each enable running cpu or gpu tests respectively across the whole code base. Please ensure one of these is selected when
running tests.

testresources: Used to add the test resources used for nd4j.

### Test categories
Deeplearning4j uses' junit 5's tags to categorize tests in to different types. All of the tag names used throughout the code base can be found
[here](https://github.com/eclipse/deeplearning4j/blob/b4047006ac8175df295c2f3c008e7601437ea4dc/nd4j/nd4j-common-tests/src/main/java/org/nd4j/common/tests/tags/TagNames.java)
Nd4j-common-tests is included as a dependency for all tests and has a few reusable utilities used throughout the code base for tests.
This makes it a great location to put common utilities we want to use throughout the code base.
The tag names are mainly there to categorize tests that can take longer or use more resources so we can avoid running those dynamically depending on the size of the machine we are running tests on.



### GPUs and multi threaded boxes
Note when running gpu tests on a box with more than 1 gpu, it can/will run out of memory if test.heap.size is at not at least 4g.
Also of note, is when running tests
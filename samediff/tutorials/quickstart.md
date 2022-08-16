---
description: Samediff Quickstart
---

# Quickstart

Samediff is a lower level and more flexible api for composing neural networks. It is a compliment to nd4j. It provides a declarative api for creating computation graphs. If you are not familiar with the idea of computation graphs, here are a few references: 1. [https://www.codingame.com/playgrounds/9487/deep-learning-from-scratch---theory-and-implementation/computational-graphs](https://www.codingame.com/playgrounds/9487/deep-learning-from-scratch---theory-and-implementation/computational-graphs) 2. [https://www.machinecurve.com/index.php/2020/09/13/tensorflow-eager-execution-what-is-it/](https://www.machinecurve.com/index.php/2020/09/13/tensorflow-eager-execution-what-is-it/)

It is very similar to more popular frameworks such as tensorflow and pytorch.

In short, a computation graph allows a user to declare a series of math operats to get to a desired output. Deeplearning4j itself has a computation graph api. More about that can be found [here](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/models/computationgraph)

Note that the samediff api is a completely separate, standalone api that shares some code via nd4j, but is otherwise a standalone framework for executing neural networks. It has a distinct file format, training/graph execution, and apis for execution. Samediff will be the main api going forward. The dl4j api most people know will still be supported, but should otherwise be treated as a feature in maintenance mode. Consider samediff a rewrite of sorts. Samediff has a set of examples [here](https://github.com/eclipse/deeplearning4j-examples/tree/master/samediff-examples) that cover the basics.

Follow this [example](https://github.com/eclipse/deeplearning4j-examples/blob/master/samediff-examples/src/main/java/org/nd4j/examples/samediff/quickstart/basics/Ex1\_SameDiff\_Basics.java) for imports and other important aspects. Please treat this page as a simple walkthrough of the concepts in samediff for people unfamiliar with the framework itself.

In short, in order to create a graph a user starts by calling Samediff.create() as follows:

```java
   SameDiff sd = SameDiff.create();
```

Nd4j's INDArray is used as the core data structure for holding results that have been compute. Graph declarations are done via samediff's SDVariable. Assuming you have read the above references on computation graphs, a samediff graph declaration can happen as follows:

```java
/*
        Variables can be added to a graph in a number of ways.
        You can think of variables as "holders" of an n-dimensional array - specifically, an ND4J INDArray
        First, let's create a variable based on a specified array, with the name "myVariable"
        */
        INDArray values = Nd4j.ones(3,4);
        SDVariable variable = sd.var("myVariable", values);

        //We can then perform operations on the variable:
        SDVariable plusOne = variable.add(1.0);                               //Name: automatically generated as "add"
        SDVariable mulTen = variable.mul("mulTen", 10.0);        //Name: Defined to be "mulTen"
```

Note we pass an INDArray in here, then call sd.var(..). Within the samediff graph that was declared, a variable referencing this INDArray is created and stored and can then be references within the graph. The SDVariable then holds a reference to the samediff graph that created it. This allows us to then chain operations off those variables delegating the call to samediff graph internally. In essence, all we're doing above is declaring a graph that adds 1 to the input, then multiplies it by 10.

To print out the graph, you can use sd.summary() as follows:

```java
         //Let's inspect the graph. We currently have 3 variables, with names "myVariable", "add", and "mulTen",
        // and two functions - our "add 1.0" and our "multiply by 10" functions. These are shown in the summary:
        System.out.println(sd.summary());
        System.out.println("===================================");
```

In order to execute the graph and get some real results out, there are a few ways to do this. One can call sd.output(..) as follows:

```java
       //We are passing in any empty map since this graph does not need any inputs for the forward pass
        sd.output(Collections.<String,INDArray>emptyMap(), "mulTen");
```

The user will notice we pass an empty map in. The reason you have to pass in variables, is inputs in to a neural network tend to be dynamic. This graph does not have dynamic inputs and can there for execute without user input. A dynamic input is typically called a placeholder.

In order to obtain results, (read: real INDArrays with actual data) there are a few ways to do so:

```java
        INDArray variableArr = variable.getArr();               //We can get arrays directly from the variables
        INDArray mulTenArr = sd.getArrForVarName("mulTen");     //Or also by name, from the Samediff instance
```

Another way of executing results is to use the SDVariable eval(..) api as follows:

```java
        //We can also do the forward pass by calling eval on the SDVariable directly.
        //Note that this will clear the sd graph and set all it's variable arrays that are not on the path of the forward pass to null
        plusOneArr = plusOne.eval();
        variableArr = variable.getArr();               //We can get arrays directly from the variables
        mulTenArr = sd.getArrForVarName("mulTen");     //Or also by name, from the Samediff instance
```

eval(..) returns the resulting value for that variable from the forward pass. You may also obtain the result from getArr(..) since the actual SDVariable stores the current state of the computation.

Breaking down what happens a bit:

1. A neural network forward pass requires computing state in a sequence construted as a DAG.
2. Each node in the graph is an operation that has input and output variables.&#x20;
3. Variables represent a named parameter for an operation and may or may not contain a current value as an INDArray.

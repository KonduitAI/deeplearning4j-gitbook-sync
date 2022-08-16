# Element wise Operations

Elementwise operations are more intuitive than vectorwise operations, because the elements of one matrix map clearly onto the other, and to obtain the result, you have to perform just one arithmetical operation.

With vectorwise matrix operations, you will have to first build intuition and also perform multiple steps. There are two basic types of matrix multiplication: inner (dot) product and outer product. The inner product results in a matrix of reduced dimensions, the outer product results in one of expanded dimensions. A helpful mnemonic: Expand outward, contract inward.

## Inner product <a href="#inner-product" id="inner-product"></a>

Unlike Hadamard products, which require that both matrices have equal rows and columns, inner products simply require that the number of columns of the first matrix equal the number of rows of the second. For example, this works

```
             [3.0]
[1.0 ,2.0] * [4.0] = (1.0 * 3.0) + (2.0 * 4.0) = 11
```

Notice a 1 x 2 row times a 2 x 1 column produces a scalar. This operation reduces the dimensions to 1,1. You can imagine rotating the row vector \[1.0 ,2.0] clockwise to stand on its end, placed against the column vector. The two top elements are then multiplied by each other, as are the bottom two, and the two products are added to consolidate in a single scalar.

In ND4J, you would create the two vectors like this:

```
INDArray nd = Nd4j.create(new float[]{1,2},new int[]{2}); //row vector
INDArray nd2 = Nd4j.create(new float[]{3,4},new int[]{2, 1}); //column vector
```

And multiply them like this

```
nd.mmul(nd2);
```

Notice ND4J code mirrors the equation in that nd \* nd2 is row vector times column vector. The method is mmul, rather than the mul we used for elementwise operations, and the extra “m” stands for “matrix.”

Now let’s take the same operation, while adding an additional column to a new array we’ll call nd4.

```
INDArray nd4 = Nd4j.create(new float[]{3,4,5,6},new int[]{2, 2});
nd.mmul(nd4);                                                                                                                                                                                                                                                   
             [3.0 ,5.0]
[1.0 ,2.0] * [4.0 ,6.0] = [(1.0 * 3.0) + (2.0 * 4.0), (1.0 * 5.0) + (2.0 * 6.0)] = [11, 17]
```

Now let’s add an extra row to the first matrix, call it nd3, and multiply it by nd4

```
INDArray nd3 = Nd4j.create(new float[]{1,3,2,4},new int[]{2,2});
nd3.mmul(nd4);
```

The equation will look like this

```
[1.0 ,2.0]   [3.0 ,5.0]   [(1.0 * 3.0) + (2.0 * 4.0), (1.0 * 5.0) + (2.0 * 6.0),    [11, 17]
[3.0 ,4.0] * [4.0 ,6.0] = (3.0 * 3.0) + (4.0 * 4.0), (3.0 * 5.0) + (4.0 * 6.0),] =  [25, 39]
```

## Outer product <a href="#outer-product" id="outer-product"></a>

Taking the outer product of the two vectors we first worked with is as simple as reversing their order.

```
nd2.mmul(nd);

 [3.0]                [(3.0 * 1.0), (3.0 * 2.0)   [3.0 ,6.0]   [3.0]   [1.0 ,2.0]
 [4.0] * [1.0 ,2.0] =  (4.0 * 1.0), (4.0 * 2.0) = [4.0 ,8.0] = [4.0] * [1.0 ,2.0]
```

It turns out the multiplying nd2 by nd is the same as multiplying it by two nd’s stacked on top of each other. That’s an outer product. As you can see, outer products also require fewer operations, since they don’t combine two products into one element in the final matrix.

A few aspects of ND4J code should be noted here. Firstly, the method mmul takes two parameters.

```
nd.mmul(MATRIX TO MULTIPLY WITH, MATRIX TO WHICH THE PRODUCT SHOULD BE ASSIGNED);
```

which could be expressed like this

```
nd.mmul(nd2, ndv);
```

which is the same as this line

```
ndv = nd.mmul(nd2);
```

Using the second parameter to specify the nd-array to which the product should be assigned is a convention common in ND4J.

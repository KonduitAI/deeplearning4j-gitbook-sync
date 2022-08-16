# Matrix Manipulation

There are several other basic matrix manipulations to highlight as you learn ND4J’s workings.

## Transpose <a href="#transpose" id="transpose"></a>

The transpose of a matrix is its mirror image. An element located in row 1, column 2, in matrix A will be located in row 2, column 1, in the transpose of matrix A, whose mathematical notation is A to the T, or A^T. Notice that the elements along the diagonal of a square matrix do not move – they are at the hinge of the reflection. In ND4J, transpose matrices like this:

```
INDArray nd = Nd4j.create(new float[]{1, 2, 3, 4}, new int[]{2, 2});

[1.0 ,3.0]
[2.0 ,4.0]                                                                                                                      
nd.transpose();

[1.0 ,2.0]
[3.0 ,4.0]
```

And a long matrix like this

```
[1.0 ,3.0 ,5.0 ,7.0 ,9.0 ,11.0]
[2.0 ,4.0 ,6.0 ,8.0 ,10.0 ,12.0]
```

Looks like this when it is transposed

```
[1.0 ,2.0]
[3.0 ,4.0]
[5.0 ,6.0]
[7.0 ,8.0]
[9.0 ,10.0]
[11.0 ,12.0]
```

In fact, transpose is just an important subset of a more general operation: reshape.

## Reshape <a href="#reshape" id="reshape"></a>

Yes, matrices can be reshaped. You can change the number of rows and columns they have. The reshaped matrix has to fulfill one condition: the product of its rows and columns must equal the product of the row and columns of the original matrix. For example, proceeding columnwise, you can reshape a 3 by 4 matrix into a 2 by 6 matrix:

```
    INDArray nd2 = Nd4j.create(new float[]{1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12}, new int[]{2, 6});
```

The array nd2 looks like this

```
[1.0 ,3.0 ,5.0 ,7.0 ,9.0 ,11.0]
[2.0 ,4.0 ,6.0 ,8.0 ,10.0 ,12.0]
```

Reshaping it is easy, and follows the same convention by which we gave it shape to begin with

```
nd2.reshape(3,4);

[1.0 ,4.0 ,7.0 ,10.0]
[2.0 ,5.0 ,8.0 ,11.0]
[3.0 ,6.0 ,9.0 ,12.0]
```

## Broadcast <a href="#broadcast" id="broadcast"></a>

Broadcast is advanced. It usually happens in the background without having to be called. The simplest way to understand it is by working with one long row vector, like the one above.

```
nd2 = Nd4j.create(new float[]{1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12});
```

Broadcasting will actually take multiple copies of that row vector and put them together into a larger matrix. The first parameter is the number of copies you want “broadcast,” as well as the number of rows involved. In order not to throw a compiler error, make the second parameter of broadcast equal to the number of elements in your row vector.

```
nd2.broadcast(new int[]{3,12});

[1.0 ,4.0 ,7.0 ,10.0 ,1.0 ,4.0 ,7.0 ,10.0 ,1.0 ,4.0 ,7.0 ,10.0]
[2.0 ,5.0 ,8.0 ,11.0 ,2.0 ,5.0 ,8.0 ,11.0 ,2.0 ,5.0 ,8.0 ,11.0]
[3.0 ,6.0 ,9.0 ,12.0 ,3.0 ,6.0 ,9.0 ,12.0 ,3.0 ,6.0 ,9.0 ,12.0]

nd2.broadcast(new int[]{6,12});

[1.0 ,7.0 ,1.0 ,7.0 ,1.0 ,7.0 ,1.0 ,7.0 ,1.0 ,7.0 ,1.0 ,7.0]
[2.0 ,8.0 ,2.0 ,8.0 ,2.0 ,8.0 ,2.0 ,8.0 ,2.0 ,8.0 ,2.0 ,8.0]
[3.0 ,9.0 ,3.0 ,9.0 ,3.0 ,9.0 ,3.0 ,9.0 ,3.0 ,9.0 ,3.0 ,9.0]
[4.0 ,10.0 ,4.0 ,10.0 ,4.0 ,10.0 ,4.0 ,10.0 ,4.0 ,10.0 ,4.0 ,10.0]
[5.0 ,11.0 ,5.0 ,11.0 ,5.0 ,11.0 ,5.0 ,11.0 ,5.0 ,11.0 ,5.0 ,11.0]
[6.0 ,12.0 ,6.0 ,12.0 ,6.0 ,12.0 ,6.0 ,12.0 ,6.0 ,12.0 ,6.0 ,12.0]
```

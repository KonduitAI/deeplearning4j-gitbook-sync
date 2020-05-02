---
title: Linalg
short_title: Linalg
description: 
category: Operations
weight: 90
---
# Operation classes
## Cholesky
```JAVA
INDArray Cholesky(INDArray input)

SDVariable Cholesky(SDVariable input)
SDVariable Cholesky(String name, SDVariable input)
```
Computes the Cholesky decomposition of one or more square matrices.

* **input**  (NUMERIC type) - Input tensor with inner-most 2 dimensions forming square matrices

## Lstsq
```JAVA
INDArray Lstsq(INDArray matrix, INDArray rhs, double l2_reguralizer, boolean fast)

SDVariable Lstsq(SDVariable matrix, SDVariable rhs, double l2_reguralizer, boolean fast)
SDVariable Lstsq(String name, SDVariable matrix, SDVariable rhs, double l2_reguralizer, boolean fast)
INDArray Lstsq(INDArray matrix, INDArray rhs, double l2_reguralizer)

SDVariable Lstsq(SDVariable matrix, SDVariable rhs, double l2_reguralizer)
SDVariable Lstsq(String name, SDVariable matrix, SDVariable rhs, double l2_reguralizer)
```
Solver for linear squares problems.

* **matrix**  (NUMERIC type) - input tensor
* **rhs**  (NUMERIC type) - input tensor
* **l2_reguralizer** - regularizer
* **fast** - fast mode, defaults to True - default = true

## Lu
```JAVA
INDArray Lu(INDArray input)

SDVariable Lu(SDVariable input)
SDVariable Lu(String name, SDVariable input)
```
Computes LU decomposition.

* **input**  (NUMERIC type) - input tensor

## Matmul
```JAVA
INDArray Matmul(INDArray a, INDArray b)

SDVariable Matmul(SDVariable a, SDVariable b)
SDVariable Matmul(String name, SDVariable a, SDVariable b)
```
Performs matrix mutiplication on input tensors.

* **a**  (NUMERIC type) - input tensor
* **b**  (NUMERIC type) - input tensor

## MatrixBandPart
```JAVA
INDArray[] MatrixBandPart(INDArray input, int minLower, int maxUpper)

SDVariable[] MatrixBandPart(SDVariable input, int minLower, int maxUpper)
SDVariable[] MatrixBandPart(String name, SDVariable input, int minLower, int maxUpper)
```
Copy a tensor setting outside a central band in each innermost matrix.

* **input**  (NUMERIC type) - input tensor
* **minLower** - lower diagonal count
* **maxUpper** - upper diagonal count

## Qr
```JAVA
INDArray[] Qr(INDArray input, boolean full)

SDVariable[] Qr(SDVariable input, boolean full)
SDVariable[] Qr(String name, SDVariable input, boolean full)
INDArray[] Qr(INDArray input)

SDVariable[] Qr(SDVariable input)
SDVariable[] Qr(String name, SDVariable input)
```
Computes the QR decompositions of input matrix.

* **input**  (NUMERIC type) - input tensor
* **full** - full matrices mode - default = false

## Solve
```JAVA
INDArray Solve(INDArray matrix, INDArray rhs, boolean adjoint)

SDVariable Solve(SDVariable matrix, SDVariable rhs, boolean adjoint)
SDVariable Solve(String name, SDVariable matrix, SDVariable rhs, boolean adjoint)
INDArray Solve(INDArray matrix, INDArray rhs)

SDVariable Solve(SDVariable matrix, SDVariable rhs)
SDVariable Solve(String name, SDVariable matrix, SDVariable rhs)
```
Solver for systems of linear equations.

* **matrix**  (NUMERIC type) - input tensor
* **rhs**  (NUMERIC type) - input tensor
* **adjoint** - adjoint mode, defaults to False - default = false

## TriangularSolve
```JAVA
INDArray TriangularSolve(INDArray matrix, INDArray rhs, boolean lower, boolean adjoint)

SDVariable TriangularSolve(SDVariable matrix, SDVariable rhs, boolean lower, boolean adjoint)
SDVariable TriangularSolve(String name, SDVariable matrix, SDVariable rhs, boolean lower, boolean adjoint)
```
Solver for systems of linear questions.

* **matrix**  (NUMERIC type) - input tensor
* **rhs**  (NUMERIC type) - input tensor
* **lower** - defines whether innermost matrices in matrix are lower or upper triangular
* **adjoint** - adjoint mode

## cross
```JAVA
INDArray cross(INDArray a, INDArray b)

SDVariable cross(SDVariable a, SDVariable b)
SDVariable cross(String name, SDVariable a, SDVariable b)
```
Computes pairwise cross product.

* **a**  (NUMERIC type) - 
* **b**  (NUMERIC type) - 

## diag
```JAVA
INDArray diag(INDArray input)

SDVariable diag(SDVariable input)
SDVariable diag(String name, SDVariable input)
```
Calculates diagonal tensor.

* **input**  (NUMERIC type) - 

## diag_part
```JAVA
INDArray diag_part(INDArray input)

SDVariable diag_part(SDVariable input)
SDVariable diag_part(String name, SDVariable input)
```
Calculates diagonal tensor.

* **input**  (NUMERIC type) - 

## logdet
```JAVA
INDArray logdet(INDArray input)

SDVariable logdet(SDVariable input)
SDVariable logdet(String name, SDVariable input)
```
Calculates log of determinant.

* **input**  (NUMERIC type) - 

## mmul
```JAVA
INDArray mmul(INDArray x, INDArray y, boolean transposeX, boolean transposeY, boolean transposeZ)

SDVariable mmul(SDVariable x, SDVariable y, boolean transposeX, boolean transposeY, boolean transposeZ)
SDVariable mmul(String name, SDVariable x, SDVariable y, boolean transposeX, boolean transposeY, boolean transposeZ)
INDArray mmul(INDArray x, INDArray y)

SDVariable mmul(SDVariable x, SDVariable y)
SDVariable mmul(String name, SDVariable x, SDVariable y)
```
Matrix multiplication: out = mmul(x,y)

Supports specifying transpose argument to perform operation such as mmul(a^T, b), etc.

* **x**  (NUMERIC type) - First input variable
* **y**  (NUMERIC type) - Second input variable
* **transposeX** - Transpose x (first argument) - default = false
* **transposeY** - Transpose y (second argument) - default = false
* **transposeZ** - Transpose result array - default = false

## svd
```JAVA
INDArray svd(INDArray input, boolean fullUV, boolean computeUV, int switchNum)

SDVariable svd(SDVariable input, boolean fullUV, boolean computeUV, int switchNum)
SDVariable svd(String name, SDVariable input, boolean fullUV, boolean computeUV, int switchNum)
INDArray svd(INDArray input, boolean fullUV, boolean computeUV)

SDVariable svd(SDVariable input, boolean fullUV, boolean computeUV)
SDVariable svd(String name, SDVariable input, boolean fullUV, boolean computeUV)
```
Calculates singular value decomposition.

* **input**  (NUMERIC type) - 
* **fullUV** - 
* **computeUV** - 
* **switchNum** -  - default = 16


---
title: Linalg
short_title: Linalg
description: 
category: Operations
weight: 90
---
# Linalg Namespace
# Operation classes
## <a name="Cholesky">Cholesky</a>
```JAVA
INDArray Cholesky(INDArray input)

SDVariable Cholesky(SDVariable input)
SDVariable Cholesky(String name, SDVariable input)
```
Computes the Cholesky decomposition of one or more square matrices.

* **input** - Input tensor with inner-most 2 dimensions forming square matrices (NUMERIC type)

## <a name="Lstsq">Lstsq</a>
```JAVA
INDArray Lstsq(INDArray matrix, INDArray rhs, double l2_reguralizer, boolean fast)

SDVariable Lstsq(SDVariable matrix, SDVariable rhs, double l2_reguralizer, boolean fast)
SDVariable Lstsq(String name, SDVariable matrix, SDVariable rhs, double l2_reguralizer, boolean fast)
INDArray Lstsq(INDArray matrix, INDArray rhs, double l2_reguralizer)

SDVariable Lstsq(SDVariable matrix, SDVariable rhs, double l2_reguralizer)
SDVariable Lstsq(String name, SDVariable matrix, SDVariable rhs, double l2_reguralizer)
```
Solver for linear squares problems.

* **matrix** - input tensor (NUMERIC type)
* **rhs** - input tensor (NUMERIC type)
* **l2_reguralizer** - regularizer
* **fast** - fast mode, defaults to True - default = true
* **matrix** - input tensor (NUMERIC type)
* **rhs** - input tensor (NUMERIC type)
* **l2_reguralizer** - regularizer

## <a name="Lu">Lu</a>
```JAVA
INDArray Lu(INDArray input)

SDVariable Lu(SDVariable input)
SDVariable Lu(String name, SDVariable input)
```
Computes LU decomposition.

* **input** - input tensor (NUMERIC type)

## <a name="Matmul">Matmul</a>
```JAVA
INDArray Matmul(INDArray a, INDArray b)

SDVariable Matmul(SDVariable a, SDVariable b)
SDVariable Matmul(String name, SDVariable a, SDVariable b)
```
Performs matrix mutiplication on input tensors.

* **a** - input tensor (NUMERIC type)
* **b** - input tensor (NUMERIC type)

## <a name="MatrixBandPart">MatrixBandPart</a>
```JAVA
INDArray[] MatrixBandPart(INDArray input, int minLower, int maxUpper)

SDVariable[] MatrixBandPart(SDVariable input, int minLower, int maxUpper)
SDVariable[] MatrixBandPart(String name, SDVariable input, int minLower, int maxUpper)
```
Copy a tensor setting outside a central band in each innermost matrix.

* **input** - input tensor (NUMERIC type)
* **minLower** - lower diagonal count
* **maxUpper** - upper diagonal count

## <a name="Qr">Qr</a>
```JAVA
INDArray[] Qr(INDArray input, boolean full)

SDVariable[] Qr(SDVariable input, boolean full)
SDVariable[] Qr(String name, SDVariable input, boolean full)
INDArray[] Qr(INDArray input)

SDVariable[] Qr(SDVariable input)
SDVariable[] Qr(String name, SDVariable input)
```
Computes the QR decompositions of input matrix.

* **input** - input tensor (NUMERIC type)
* **full** - full matrices mode - default = false
* **input** - input tensor (NUMERIC type)

## <a name="Solve">Solve</a>
```JAVA
INDArray Solve(INDArray matrix, INDArray rhs, boolean adjoint)

SDVariable Solve(SDVariable matrix, SDVariable rhs, boolean adjoint)
SDVariable Solve(String name, SDVariable matrix, SDVariable rhs, boolean adjoint)
INDArray Solve(INDArray matrix, INDArray rhs)

SDVariable Solve(SDVariable matrix, SDVariable rhs)
SDVariable Solve(String name, SDVariable matrix, SDVariable rhs)
```
Solver for systems of linear equations.

* **matrix** - input tensor (NUMERIC type)
* **rhs** - input tensor (NUMERIC type)
* **adjoint** - adjoint mode, defaults to False - default = false
* **matrix** - input tensor (NUMERIC type)
* **rhs** - input tensor (NUMERIC type)

## <a name="TriangularSolve">TriangularSolve</a>
```JAVA
INDArray TriangularSolve(INDArray matrix, INDArray rhs, boolean lower, boolean adjoint)

SDVariable TriangularSolve(SDVariable matrix, SDVariable rhs, boolean lower, boolean adjoint)
SDVariable TriangularSolve(String name, SDVariable matrix, SDVariable rhs, boolean lower, boolean adjoint)
```
Solver for systems of linear questions.

* **matrix** - input tensor (NUMERIC type)
* **rhs** - input tensor (NUMERIC type)
* **lower** - defines whether innermost matrices in matrix are lower or upper triangular
* **adjoint** - adjoint mode

## <a name="cross">cross</a>
```JAVA
INDArray cross(INDArray a, INDArray b)

SDVariable cross(SDVariable a, SDVariable b)
SDVariable cross(String name, SDVariable a, SDVariable b)
```
Computes pairwise cross product.

* **a** -  (NUMERIC type)
* **b** -  (NUMERIC type)

## <a name="diag">diag</a>
```JAVA
INDArray diag(INDArray input)

SDVariable diag(SDVariable input)
SDVariable diag(String name, SDVariable input)
```
Calculates diagonal tensor.

* **input** -  (NUMERIC type)

## <a name="diag_part">diag_part</a>
```JAVA
INDArray diag_part(INDArray input)

SDVariable diag_part(SDVariable input)
SDVariable diag_part(String name, SDVariable input)
```
Calculates diagonal tensor.

* **input** -  (NUMERIC type)

## <a name="logdet">logdet</a>
```JAVA
INDArray logdet(INDArray input)

SDVariable logdet(SDVariable input)
SDVariable logdet(String name, SDVariable input)
```
Calculates log of determinant.

* **input** -  (NUMERIC type)

## <a name="mmul">mmul</a>
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

* **x** - First input variable (NUMERIC type)
* **y** - Second input variable (NUMERIC type)
* **transposeX** - Transpose x (first argument) - default = false
* **transposeY** - Transpose y (second argument) - default = false
* **transposeZ** - Transpose result array - default = false
* **x** - First input variable (NUMERIC type)
* **y** - Second input variable (NUMERIC type)

## <a name="svd">svd</a>
```JAVA
INDArray svd(INDArray input, boolean fullUV, boolean computeUV, int switchNum)

SDVariable svd(SDVariable input, boolean fullUV, boolean computeUV, int switchNum)
SDVariable svd(String name, SDVariable input, boolean fullUV, boolean computeUV, int switchNum)
INDArray svd(INDArray input, boolean fullUV, boolean computeUV)

SDVariable svd(SDVariable input, boolean fullUV, boolean computeUV)
SDVariable svd(String name, SDVariable input, boolean fullUV, boolean computeUV)
```
Calculates singular value decomposition.

* **input** -  (NUMERIC type)
* **fullUV** - 
* **computeUV** - 
* **switchNum** -  - default = 16
* **input** -  (NUMERIC type)
* **fullUV** - 
* **computeUV** - 


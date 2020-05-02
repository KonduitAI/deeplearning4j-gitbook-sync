---
title: Bitwise
short_title: Bitwise
description: 
category: Operations
weight: 0
---
# Operation classes
## and
```JAVA
INDArray and(INDArray x, INDArray y)

SDVariable and(SDVariable x, SDVariable y)
SDVariable and(String name, SDVariable x, SDVariable y)
```
Bitwise AND operation. Supports broadcasting.

* **x**  (INT) - First input array
* **y**  (INT) - Second input array

## bitRotl
```JAVA
INDArray bitRotl(INDArray x, INDArray shift)

SDVariable bitRotl(SDVariable x, SDVariable shift)
SDVariable bitRotl(String name, SDVariable x, SDVariable shift)
```
Roll integer bits to the left, i.e. var << 4 | var >> (32 - 4)

* **x**  (INT) - Input 1
* **shift**  (INT) - Number of bits to shift.

## bitRotr
```JAVA
INDArray bitRotr(INDArray x, INDArray shift)

SDVariable bitRotr(SDVariable x, SDVariable shift)
SDVariable bitRotr(String name, SDVariable x, SDVariable shift)
```
Roll integer bits to the right, i.e. var >> 4 | var << (32 - 4)

* **x**  (INT) - Input 1
* **shift**  (INT) - Number of bits to shift.

## bitShift
```JAVA
INDArray bitShift(INDArray x, INDArray shift)

SDVariable bitShift(SDVariable x, SDVariable shift)
SDVariable bitShift(String name, SDVariable x, SDVariable shift)
```
Shift integer bits to the left, i.e. var << 4

* **x**  (INT) - Input 1
* **shift**  (INT) - Number of bits to shift.

## bitShiftRight
```JAVA
INDArray bitShiftRight(INDArray x, INDArray shift)

SDVariable bitShiftRight(SDVariable x, SDVariable shift)
SDVariable bitShiftRight(String name, SDVariable x, SDVariable shift)
```
Shift integer bits to the right, i.e. var >> 4

* **x**  (INT) - Input 1
* **shift**  (INT) - Number of bits to shift.

## bitsHammingDistance
```JAVA
INDArray bitsHammingDistance(INDArray x, INDArray y)

SDVariable bitsHammingDistance(SDVariable x, SDVariable y)
SDVariable bitsHammingDistance(String name, SDVariable x, SDVariable y)
```
Bitwise Hamming distance reduction over all elements of both input arrays.<br>
For example, if x=01100000 and y=1010000 then the bitwise Hamming distance is 2 (due to differences at positions 0 and 1)

* **x**  (INT) - First input array.
* **y**  (INT) - Second input array.

## leftShift
```JAVA
INDArray leftShift(INDArray x, INDArray y)

SDVariable leftShift(SDVariable x, SDVariable y)
SDVariable leftShift(String name, SDVariable x, SDVariable y)
```
Bitwise left shift operation. Supports broadcasting.

* **x**  (INT) - Input to be bit shifted
* **y**  (INT) - Amount to shift elements of x array

## leftShiftCyclic
```JAVA
INDArray leftShiftCyclic(INDArray x, INDArray y)

SDVariable leftShiftCyclic(SDVariable x, SDVariable y)
SDVariable leftShiftCyclic(String name, SDVariable x, SDVariable y)
```
Bitwise left cyclical shift operation. Supports broadcasting.

Unlike #leftShift(INDArray, INDArray) the bits will "wrap around":

`leftShiftCyclic(01110000, 2) -> 11000001`

* **x**  (INT) - Input to be bit shifted
* **y**  (INT) - Amount to shift elements of x array

## or
```JAVA
INDArray or(INDArray x, INDArray y)

SDVariable or(SDVariable x, SDVariable y)
SDVariable or(String name, SDVariable x, SDVariable y)
```
Bitwise OR operation. Supports broadcasting.

* **x**  (INT) - First input array
* **y**  (INT) - First input array

## rightShift
```JAVA
INDArray rightShift(INDArray x, INDArray y)

SDVariable rightShift(SDVariable x, SDVariable y)
SDVariable rightShift(String name, SDVariable x, SDVariable y)
```
Bitwise right shift operation. Supports broadcasting. 

* **x**  (INT) - Input to be bit shifted
* **y**  (INT) - Amount to shift elements of x array

## rightShiftCyclic
```JAVA
INDArray rightShiftCyclic(INDArray x, INDArray y)

SDVariable rightShiftCyclic(SDVariable x, SDVariable y)
SDVariable rightShiftCyclic(String name, SDVariable x, SDVariable y)
```
Bitwise right cyclical shift operation. Supports broadcasting.

Unlike rightShift(INDArray, INDArray) the bits will "wrap around":

`rightShiftCyclic(00001110, 2) -> 10000011`

* **x**  (INT) - Input to be bit shifted
* **y**  (INT) - Amount to shift elements of x array

## xor
```JAVA
INDArray xor(INDArray x, INDArray y)

SDVariable xor(SDVariable x, SDVariable y)
SDVariable xor(String name, SDVariable x, SDVariable y)
```
Bitwise XOR operation (exclusive OR). Supports broadcasting.

* **x**  (INT) - First input array
* **y**  (INT) - First input array


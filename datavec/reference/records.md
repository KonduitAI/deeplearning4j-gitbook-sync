---
description: How to use data records in DataVec.
---

# Records

## What is a record?

In the DataVec world a Record represents a single entry in a dataset. DataVec differentiates types of records to make data manipulation easier with built-in APIs. Sequences and 2D records are distinguishable.

## Using records

Most of the time you do not need to interact with the record classes directly, unless you are manually iterating records for the purpose of forwarding through a neural network.

## Types of records

### Record

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/impl/Record.java)

A standard implementation of the Record interface

### SequenceRecord

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/records/impl/SequenceRecord.java)

A standard implementation of the SequenceRecord interface.

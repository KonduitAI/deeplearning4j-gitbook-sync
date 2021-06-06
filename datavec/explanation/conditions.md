# Conditions

## BooleanCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/BooleanCondition.java)

BooleanCondition: used for creating compound conditions, such as AND\(ConditionA, ConditionB, …\)  
As a BooleanCondition is a condition, these can be chained together, like NOT\(OR\(AND\(…\),AND\(…\)\)\)

#### **outputColumnName**

```text
public String outputColumnName() 
```

The output column name after the operation has been applied

* return the output column name

#### **columnName**

```text
public String columnName() 
```

The output column names This will often be the same as the input

* return the output column names

**condition**

```text
public boolean condition(Object input) 
```

Condition on arbitrary input

* param input the input to return the condition for
* return true if the condition is met false otherwise

**conditionSequence**

```text
public boolean conditionSequence(Object sequence) 
```

Condition on arbitrary input

* param sequence the sequence to do a condition on
* return true if the condition for the sequence is met false otherwise

**transform**

```text
public Schema transform(Schema inputSchema) 
```

Get the output schema for this transformation, given an input schema

* param inputSchema

### **AND**

```text
public static Condition AND(Condition... conditions) 
```

And of all the given conditions

* param conditions the conditions to and
* return a joint and of all these conditions

### **OR**

```text
public static Condition OR(Condition... conditions) 
```

Or of all the given conditions

* param conditions the conditions to or
* return a joint and of all these conditions

### **NOT**

```text
public static Condition NOT(Condition condition) 
```

Not of the given condition

* param condition the conditions to and
* return a joint and of all these condition

### **XOR**

```text
public static Condition XOR(Condition first, Condition second) 
```

And of all the given conditions

* param first the first condition
* param second the second condition for xor
* return the xor of these 2 conditions

#### SequenceConditionMode <a id="sequenceconditionmode"></a>

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/SequenceConditionMode.java)

For certain single-column conditions: how should we apply these to sequences?  
**And**: Condition applies to sequence only if it applies to ALL time steps  
**Or**: Condition applies to sequence if it applies to ANY time steps  
**NoSequencMode**: Condition cannot be applied to sequences at all \(error condition\)

## BooleanColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/column/BooleanColumnCondition.java)

Created by agibsonccc on 11/26/16.

**columnCondition**

```text
public boolean columnCondition(Writable writable) 
```

Returns whether the given element meets the condition set by this operation

* param writable the element to test
* return true if the condition is met false otherwise

**condition**

```text
public boolean condition(Object input) 
```

Condition on arbitrary input

* param input the input to return the condition for
* return true if the condition is met false otherwise

## CategoricalColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/column/CategoricalColumnCondition.java)

**columnCondition**

```text
public boolean columnCondition(Writable writable) 
```

Constructor for conditions equal or not equal. Uses default sequence condition mode, {- link BaseColumnCondition\#DEFAULT\_SEQUENCE\_CONDITION\_MODE}

* param columnName Column to check for the condition
* param op Operation \(== or != only\)
* param value Value to use in the condition

**condition**

```text
public boolean condition(Object input) 
```

Condition on arbitrary input

* param input the input to return the condition for
* return true if the condition is met false otherwise

## DoubleColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/column/DoubleColumnCondition.java)

**columnCondition**

```text
public boolean columnCondition(Writable writable) 
```

Constructor for operations such as less than, equal to, greater than, etc. Uses default sequence condition mode, {- link BaseColumnCondition\#DEFAULT\_SEQUENCE\_CONDITION\_MODE}

* param columnName Column to check for the condition
* param op Operation \(&lt;, &gt;=, !=, etc\)
* param value Value to use in the condition

**condition**

```text
public boolean condition(Object input) 
```

Condition on arbitrary input

* param input the input to return the condition for
* return true if the condition is met false otherwise

## InfiniteColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/column/InfiniteColumnCondition.java)

A column condition that simply checks whether a floating point value is infinite

**columnCondition**

```text
public boolean columnCondition(Writable writable) 
```

* param columnName Column check for the condition

## IntegerColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/column/IntegerColumnCondition.java)

**columnCondition**

```text
public boolean columnCondition(Writable writable) 
```

Constructor for operations such as less than, equal to, greater than, etc. Uses default sequence condition mode, {- link BaseColumnCondition\#DEFAULT\_SEQUENCE\_CONDITION\_MODE}

* param columnName Column to check for the condition
* param op Operation \(&lt;, &gt;=, !=, etc\)
* param value Value to use in the condition

**condition**

```text
public boolean condition(Object input) 
```

Condition on arbitrary input

* param input the input to return the condition for
* return true if the condition is met false otherwise

## InvalidValueColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/column/InvalidValueColumnCondition.java)

A Condition that applies to a single column. Whenever the specified value is invalid according to the schema, the condition applies.

For example, if a Writable contains String values in an Integer column \(and these cannot be parsed to an integer\), then the condition would return true, as these values are invalid according to the schema.

**condition**

```text
public boolean condition(Object input) 
```

Condition on arbitrary input

* param input the input to return the condition for
* return true if the condition is met false otherwise

## LongColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/column/LongColumnCondition.java)

**columnCondition**

```text
public boolean columnCondition(Writable writable) 
```

Constructor for operations such as less than, equal to, greater than, etc. Uses default sequence condition mode, {- link BaseColumnCondition\#DEFAULT\_SEQUENCE\_CONDITION\_MODE}

* param columnName Column to check for the condition
* param op Operation \(&lt;, &gt;=, !=, etc\)
* param value Value to use in the condition

**condition**

```text
public boolean condition(Object input) 
```

Condition on arbitrary input

* param input the input to return the condition for
* return true if the condition is met false otherwise

## NaNColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/column/NaNColumnCondition.java)

A column condition that simply checks whether a floating point value is NaN

**columnCondition**

```text
public boolean columnCondition(Writable writable) 
```

* param columnName Name of the column to check the condition for

## NullWritableColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/column/NullWritableColumnCondition.java)

Condition that applies to the values in any column. Specifically, condition is true if the Writable value is a NullWritable, and false for any other value

**condition**

```text
public boolean condition(Object input) 
```

Condition on arbitrary input

* param input the input to return the condition for
* return true if the condition is met false otherwise

## StringColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/column/StringColumnCondition.java)

**columnCondition**

```text
public boolean columnCondition(Writable writable) 
```

Constructor for conditions equal or not equal Uses default sequence condition mode, {- link BaseColumnCondition\#DEFAULT\_SEQUENCE\_CONDITION\_MODE}

* param columnName Column to check for the condition
* param op Operation \(== or != only\)
* param value Value to use in the condition

**condition**

```text
public boolean condition(Object input) 
```

Condition on arbitrary input

* param input the input to return the condition for
* return true if the condition is met false otherwise

## TimeColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/column/TimeColumnCondition.java)

Condition that applies to the values

**columnCondition**

```text
public boolean columnCondition(Writable writable) 
```

Constructor for operations such as less than, equal to, greater than, etc. Uses default sequence condition mode, {- link BaseColumnCondition\#DEFAULT\_SEQUENCE\_CONDITION\_MODE}

* param columnName Column to check for the condition
* param op Operation \(&lt;, &gt;=, !=, etc\)
* param value Time value \(in epoch millisecond format\) to use in the condition

**condition**

```text
public boolean condition(Object input) 
```

Condition on arbitrary input

* param input the input to return the condition for
* return true if the condition is met false otherwise

## TrivialColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/column/TrivialColumnCondition.java)

Created by huitseeker on 5/17/17.

## SequenceLengthCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/sequence/SequenceLengthCondition.java)

A condition on sequence lengths

## StringRegexColumnCondition

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/condition/string/StringRegexColumnCondition.java)

Condition that applies to the values in a String column, using a provided regex. Condition return true if the String matches the regex, or false otherwise  
**Note:** Uses Writable.toString\(\), hence can potentially be applied to non-String columns

**condition**

```text
public boolean condition(Object input) 
```

Condition on arbitrary input

* param input the input to return the condition for
* return true if the condition is met false otherwise


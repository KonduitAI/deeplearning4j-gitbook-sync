---
description: Selection of data using conditions.
---

# Filters

## Using filters

Filters are a part of transforms and gives a DSL for you to keep parts of your dataset. Filters can be one-liners for single conditions or include complex boolean logic.

```java
TransformProcess tp = new TransformProcess.Builder(inputDataSchema)
    .filter(new ConditionFilter(new CategoricalColumnCondition("MerchantCountryCode", ConditionOp.NotInSet, new HashSet<>(Arrays.asList("USA","CAN")))))
    .build();
```

You can also write your own filters by implementing the `Filter` interface, though it is much more often that you may want to create a custom condition instead.

## Available filters

### ConditionFilter

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/filter/ConditionFilter.java)

If condition is satisfied (returns true): remove the example or sequence\
If condition is not satisfied (returns false): keep the example or sequence

**removeExample**

```
public boolean removeExample(Object writables)
```

* param writables Example
* return true if example should be removed, false to keep

**removeSequence**

```
public boolean removeSequence(Object sequence)
```

* param sequence sequence example
* return true if example should be removed, false to keep

**transform**

```
public Schema transform(Schema inputSchema)
```

Get the output schema for this transformation, given an input schema

* param inputSchema

**outputColumnName**

```
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### Filter

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/filter/Filter.java)

Filter: a method of removing examples (or sequences) according to some condition

### FilterInvalidValues

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/filter/FilterInvalidValues.java)

FilterInvalidValues: a filter operation that removes any examples (or sequences) if the examples/sequences contains invalid values in any of a specified set of columns. Invalid values are determined with respect to the schema

**transform**

```
public Schema transform(Schema inputSchema)
```

* param columnsToFilterIfInvalid Columns to check for invalid values

**removeExample**

```
public boolean removeExample(Object writables)
```

* param writables Example
* return true if example should be removed, false to keep

**removeSequence**

```
public boolean removeSequence(Object sequence)
```

* param sequence sequence example
* return true if example should be removed, false to keep

**outputColumnName**

```
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### InvalidNumColumns

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/filter/InvalidNumColumns.java)

Remove invalid records of a certain size.

**removeExample**

```
public boolean removeExample(Object writables)
```

* param writables Example
* return true if example should be removed, false to keep

**removeSequence**

```
public boolean removeSequence(Object sequence)
```

* param sequence sequence example
* return true if example should be removed, false to keep

**removeExample**

```
public boolean removeExample(List<Writable> writables)
```

* param writables Example
* return true if example should be removed, false to keep

**removeSequence**

```
public boolean removeSequence(List<List<Writable>> sequence)
```

* param sequence sequence example
* return true if example should be removed, false to keep

**transform**

```
public Schema transform(Schema inputSchema)
```

Get the output schema for this transformation, given an input schema

* param inputSchema

**outputColumnName**

```
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

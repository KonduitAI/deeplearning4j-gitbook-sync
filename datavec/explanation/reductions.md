# Reductions

## Available reductions

### GeographicMidpointReduction

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/reduce/impl/GeographicMidpointReduction.java)

delimiter is configurable\), determine the geographic midpoint. See “geographic midpoint” at: [http://www.geomidpoint.com/methods.html](http://www.geomidpoint.com/methods.html) For implementation algorithm, see: [http://www.geomidpoint.com/calculation.html](http://www.geomidpoint.com/calculation.html)

**transform**

```text
public Schema transform(Schema inputSchema) 
```

* param delim Delimiter for the coordinates in text format. For example, if format is “lat,long” use “,”

### StringReducer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/stringreduce/StringReducer.java)

A StringReducer is used to take a set of examples and reduce them. The idea: suppose you have a large number of columns, and you want to combine/reduce the values in each column.  
StringReducer allows you to specify different reductions for differently for different columns: min, max, sum, mean etc.

Uses are: \(1\) Reducing examples by a key \(2\) Reduction operations in time series \(windowing ops, etc\)

**transform**

```text
public Schema transform(Schema schema) 
```

Get the output schema, given the input schema

**outputColumnName**

```text
public Builder outputColumnName(String outputColumnName) 
```

Create a StringReducer builder, and set the default column reduction operation. For any columns that aren’t specified explicitly, they will use the default reduction operation. If a column does have a reduction operation explicitly specified, then it will override the default specified here.

* param defaultOp Default reduction operation to perform

**appendColumns**

```text
public Builder appendColumns(String... columns) 
```

Reduce the specified columns by taking the minimum value

**prependColumns**

```text
public Builder prependColumns(String... columns) 
```

Reduce the specified columns by taking the maximum value

**mergeColumns**

```text
public Builder mergeColumns(String... columns) 
```

Reduce the specified columns by taking the sum of values

**replaceColumn**

```text
public Builder replaceColumn(String... columns) 
```

Reduce the specified columns by taking the mean of the values

**customReduction**

```text
public Builder customReduction(String column, ColumnReduction columnReduction) 
```

Reduce the specified column using a custom column reduction functionality.

* param column Column to execute the custom reduction functionality on
* param columnReduction Column reduction to execute on that column

**setIgnoreInvalid**

```text
public Builder setIgnoreInvalid(String... columns) 
```

When doing the reduction: set the specified columns to ignore any invalid values. Invalid: defined as being not valid according to the ColumnMetaData: {- link ColumnMetaData\#isValid\(Writable\)}. For numerical columns, this typically means being unable to parse the Writable. For example, Writable.toLong\(\) failing for a Long column. If the column has any restrictions \(min/max values, regex for Strings etc\) these will also be taken into account.

* param columns Columns to set ‘ignore invalid’ for


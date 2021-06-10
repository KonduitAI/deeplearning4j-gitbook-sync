---
title: DataVec Schema
short_title: Schema
description: Schemas for datasets and transformation.
category: DataVec
weight: 1
---

# Schemas

## Why use schemas?

The unfortunate reality is that data is _dirty_. When trying to vecotrize a dataset for deep learning, it is quite rare to find files that have zero errors. Schema is important for maintaining the meaning of the data before using it for something like training a neural network.

## Using schemas

Schemas are primarily used for programming transformations. Before you can properly execute a `TransformProcess` you will need to pass the schema of the data being transformed.

An example of a schema for merchant records may look like:

```java
Schema inputDataSchema = new Schema.Builder()
    .addColumnsString("DateTimeString", "CustomerID", "MerchantID")
    .addColumnInteger("NumItemsInTransaction")
    .addColumnCategorical("MerchantCountryCode", Arrays.asList("USA","CAN","FR","MX"))
    .addColumnDouble("TransactionAmountUSD",0.0,null,false,false)   //$0.0 or more, no maximum limit, no NaN and no Infinite values
    .addColumnCategorical("FraudLabel", Arrays.asList("Fraud","Legit"))
    .build();
```

## Joining schemas

If you have two different datasets that you want to merge together, DataVec provides a `Join` class with different join strategies such as `Inner` or `RightOuter`.

```java
Schema customerInfoSchema = new Schema.Builder()
    .addColumnLong("customerID")
    .addColumnString("customerName")
    .addColumnCategorical("customerCountry", Arrays.asList("USA","France","Japan","UK"))
    .build();

Schema customerPurchasesSchema = new Schema.Builder()
    .addColumnLong("customerID")
    .addColumnTime("purchaseTimestamp", DateTimeZone.UTC)
    .addColumnLong("productID")
    .addColumnInteger("purchaseQty")
    .addColumnDouble("unitPriceUSD")
    .build();

Join join = new Join.Builder(Join.JoinType.Inner)
    .setJoinColumns("customerID")
    .setSchemas(customerInfoSchema, customerPurchasesSchema)
    .build();
```

Once you've defined your join and you've loaded the data into DataVec, you must use an `Executor` to complete the join.

## Classes and utilities

DataVec comes with a few `Schema` classes and helper utilities for 2D and sequence types of data.

### Join

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/join/Join.java)

Join class: used to specify a join \(like an SQL join\)

**setSchemas**

```text
public Builder setSchemas(Schema left, Schema right)
```

Type of join  
Inner: Return examples where the join column values occur in both LeftOuter: Return all examples from left data, whether there is a matching right value or not. \(If not: right values will have NullWritable instead\)  
RightOuter: Return all examples from the right data, whether there is a matching left value or not. \(If not: left values will have NullWritable instead\)  
FullOuter: return all examples from both left/right, whether there is a matching value from the other side or not. \(If not: other values will have NullWritable instead\)

**setKeyColumns**

```text
public Builder setKeyColumns(String... keyColumnNames)
```

* deprecated Use {- link \#setJoinColumns\(String…\)}

**setKeyColumnsLeft**

```text
public Builder setKeyColumnsLeft(String... keyColumnNames)
```

* deprecated Use {- link \#setJoinColumnsLeft\(String…\)}

**setKeyColumnsRight**

```text
public Builder setKeyColumnsRight(String... keyColumnNames)
```

* deprecated Use {- link \#setJoinColumnsRight\(String…\)}

**setJoinColumnsLeft**

```text
public Builder setJoinColumnsLeft(String... joinColumnNames)
```

Specify the names of the columns to join on, for the left data\) The idea: join examples where firstDataValues\(joinColumNamesLeft\[i\]\) == secondDataValues\(joinColumnNamesRight\[i\]\) for all i

* param joinColumnNames Names of the columns to join on \(for left data\)

**setJoinColumnsRight**

```text
public Builder setJoinColumnsRight(String... joinColumnNames)
```

Specify the names of the columns to join on, for the right data\) The idea: join examples where firstDataValues\(joinColumNamesLeft\[i\]\) == secondDataValues\(joinColumnNamesRight\[i\]\) for all i

* param joinColumnNames Names of the columns to join on \(for left data\)

### InferredSchema

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/schema/InferredSchema.java)

If passed a CSV file that contains a header and a single row of sample data, it will return a Schema.

Only Double, Integer, Long, and String types are supported. If no number type can be inferred, the field type will become the default type. Note that if your column is actually categorical but is represented as a number, you will need to do additional transformation. Also, if your sample field is blank/null, it will also become the default type.

### Schema

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/schema/Schema.java)

A Schema defines the layout of tabular data. Specifically, it contains names f or each column, as well as details of types \(Integer, String, Long, Double, etc\).  
Type information for each column may optionally include restrictions on the allowable values for each column.

**sameTypes**

```text
public boolean sameTypes(Schema schema)
```

Create a schema based on the given metadata

* param columnMetaData the metadata to create the schema from

**newSchema**

```text
public Schema newSchema(List<ColumnMetaData> columnMetaData)
```

Compute the difference in {- link ColumnMetaData} between this schema and the passed in schema. This is useful during the {- link org.datavec.api.transform.TransformProcess} to identify what a process will do to a given {- link Schema}.

* param schema the schema to compute the difference for
* return the metadata that is different \(in order\) between this schema and the other schema

**numColumns**

```text
public int numColumns()
```

Returns the number of columns or fields for this schema

* return the number of columns or fields for this schema

**getName**

```text
public String getName(int column)
```

Returns the name of a given column at the specified index

* param column the index of the column to get the name for
* return the name of the column at the specified index

**getType**

```text
public ColumnType getType(int column)
```

Returns the {- link ColumnType} for the column at the specified index

* param column the index of the column to get the type for
* return the type of the column to at the specified inde

**getType**

```text
public ColumnType getType(String columnName)
```

Returns the {- link ColumnType} for the column at the specified index

* param columnName the index of the column to get the type for
* return the type of the column to at the specified inde

**getMetaData**

```text
public ColumnMetaData getMetaData(int column)
```

Returns the {- link ColumnMetaData} at the specified column index

* param column the index to get the metadata for
* return the metadata at ths specified index

**getMetaData**

```text
public ColumnMetaData getMetaData(String column)
```

Retrieve the metadata for the given column name

* param column the name of the column to get metadata for
* return the metadata for the given column name

**getIndexOfColumn**

```text
public int getIndexOfColumn(String columnName)
```

Return a copy of the list column names

* return a copy of the list of column names for this schema

**hasColumn**

```text
public boolean hasColumn(String columnName)
```

Return the indices of the columns, given their namess

* param columnNames Name of the columns to get indices for
* return Column indexes

**toJson**

```text
public String toJson()
```

Serialize this schema to json

* return a json representation of this schema

**toYaml**

```text
public String toYaml()
```

Serialize this schema to yaml

* return the yaml representation of this schema

**fromJson**

```text
public static Schema fromJson(String json)
```

Create a schema from a given json string

* param json the json to create the schema from
* return the created schema based on the json

**fromYaml**

```text
public static Schema fromYaml(String yaml)
```

Create a schema from the given yaml string

* param yaml the yaml to create the schema from
* return the created schema based on the yaml

**addColumnFloat**

```text
public Builder addColumnFloat(String name)
```

Add a Float column with no restrictions on the allowable values, except for no NaN/infinite values allowed

* param name Name of the column

**addColumnFloat**

```text
public Builder addColumnFloat(String name, Float minAllowedValue, Float maxAllowedValue)
```

Add a Float column with the specified restrictions \(and no NaN/Infinite values allowed\)

* param name Name of the column
* param minAllowedValue Minimum allowed value \(inclusive\). If null: no restriction
* param maxAllowedValue Maximum allowed value \(inclusive\). If null: no restriction
* return

**addColumnFloat**

```text
public Builder addColumnFloat(String name, Float minAllowedValue, Float maxAllowedValue, boolean allowNaN,
                                       boolean allowInfinite)
```

Add a Float column with the specified restrictions

* param name Name of the column
* param minAllowedValue Minimum allowed value \(inclusive\). If null: no restriction
* param maxAllowedValue Maximum allowed value \(inclusive\). If null: no restriction
* param allowNaN If false: don’t allow NaN values. If true: allow.
* param allowInfinite If false: don’t allow infinite values. If true: allow

**addColumnsFloat**

```text
public Builder addColumnsFloat(String... columnNames)
```

Add multiple Float columns with no restrictions on the allowable values of the columns \(other than no NaN/Infinite\)

* param columnNames Names of the columns to add

**addColumnsFloat**

```text
public Builder addColumnsFloat(String pattern, int minIdxInclusive, int maxIdxInclusive)
```

A convenience method for adding multiple Float columns. For example, to add columns “myFloatCol\_0”, “myFloatCol\_1”, “myFloatCol\_2”, use {- code addColumnsFloat\(“myFloatCol\_%d”,0,2\)}

* param pattern Pattern to use \(via String.format\). “%d” is replaced with column numbers
* param minIdxInclusive Minimum column index to use \(inclusive\)
* param maxIdxInclusive Maximum column index to use \(inclusive\)

**addColumnsFloat**

```text
public Builder addColumnsFloat(String pattern, int minIdxInclusive, int maxIdxInclusive,
                                        Float minAllowedValue, Float maxAllowedValue, boolean allowNaN, boolean allowInfinite)
```

A convenience method for adding multiple Float columns, with additional restrictions that apply to all columns For example, to add columns “myFloatCol\_0”, “myFloatCol\_1”, “myFloatCol\_2”, use {- code addColumnsFloat\(“myFloatCol\_%d”,0,2,null,null,false,false\)}

* param pattern Pattern to use \(via String.format\). “%d” is replaced with column numbers
* param minIdxInclusive Minimum column index to use \(inclusive\)
* param maxIdxInclusive Maximum column index to use \(inclusive\)
* param minAllowedValue Minimum allowed value \(inclusive\). If null: no restriction
* param maxAllowedValue Maximum allowed value \(inclusive\). If null: no restriction
* param allowNaN If false: don’t allow NaN values. If true: allow.
* param allowInfinite If false: don’t allow infinite values. If true: allow

**addColumnDouble**

```text
public Builder addColumnDouble(String name)
```

Add a Double column with no restrictions on the allowable values, except for no NaN/infinite values allowed

* param name Name of the column

**addColumnDouble**

```text
public Builder addColumnDouble(String name, Double minAllowedValue, Double maxAllowedValue)
```

Add a Double column with the specified restrictions \(and no NaN/Infinite values allowed\)

* param name Name of the column
* param minAllowedValue Minimum allowed value \(inclusive\). If null: no restriction
* param maxAllowedValue Maximum allowed value \(inclusive\). If null: no restriction
* return

**addColumnDouble**

```text
public Builder addColumnDouble(String name, Double minAllowedValue, Double maxAllowedValue, boolean allowNaN,
                        boolean allowInfinite)
```

Add a Double column with the specified restrictions

* param name Name of the column
* param minAllowedValue Minimum allowed value \(inclusive\). If null: no restriction
* param maxAllowedValue Maximum allowed value \(inclusive\). If null: no restriction
* param allowNaN If false: don’t allow NaN values. If true: allow.
* param allowInfinite If false: don’t allow infinite values. If true: allow

**addColumnsDouble**

```text
public Builder addColumnsDouble(String... columnNames)
```

Add multiple Double columns with no restrictions on the allowable values of the columns \(other than no NaN/Infinite\)

* param columnNames Names of the columns to add

**addColumnsDouble**

```text
public Builder addColumnsDouble(String pattern, int minIdxInclusive, int maxIdxInclusive)
```

A convenience method for adding multiple Double columns. For example, to add columns “myDoubleCol\_0”, “myDoubleCol\_1”, “myDoubleCol\_2”, use {- code addColumnsDouble\(“myDoubleCol\_%d”,0,2\)}

* param pattern Pattern to use \(via String.format\). “%d” is replaced with column numbers
* param minIdxInclusive Minimum column index to use \(inclusive\)
* param maxIdxInclusive Maximum column index to use \(inclusive\)

**addColumnsDouble**

```text
public Builder addColumnsDouble(String pattern, int minIdxInclusive, int maxIdxInclusive,
                        Double minAllowedValue, Double maxAllowedValue, boolean allowNaN, boolean allowInfinite)
```

A convenience method for adding multiple Double columns, with additional restrictions that apply to all columns For example, to add columns “myDoubleCol\_0”, “myDoubleCol\_1”, “myDoubleCol\_2”, use {- code addColumnsDouble\(“myDoubleCol\_%d”,0,2,null,null,false,false\)}

* param pattern Pattern to use \(via String.format\). “%d” is replaced with column numbers
* param minIdxInclusive Minimum column index to use \(inclusive\)
* param maxIdxInclusive Maximum column index to use \(inclusive\)
* param minAllowedValue Minimum allowed value \(inclusive\). If null: no restriction
* param maxAllowedValue Maximum allowed value \(inclusive\). If null: no restriction
* param allowNaN If false: don’t allow NaN values. If true: allow.
* param allowInfinite If false: don’t allow infinite values. If true: allow

**addColumnInteger**

```text
public Builder addColumnInteger(String name)
```

Add an Integer column with no restrictions on the allowable values

* param name Name of the column

**addColumnInteger**

```text
public Builder addColumnInteger(String name, Integer minAllowedValue, Integer maxAllowedValue)
```

Add an Integer column with the specified min/max allowable values

* param name Name of the column
* param minAllowedValue Minimum allowed value \(inclusive\). If null: no restriction
* param maxAllowedValue Maximum allowed value \(inclusive\). If null: no restriction

**addColumnsInteger**

```text
public Builder addColumnsInteger(String... names)
```

Add multiple Integer columns with no restrictions on the min/max allowable values

* param names Names of the integer columns to add

**addColumnsInteger**

```text
public Builder addColumnsInteger(String pattern, int minIdxInclusive, int maxIdxInclusive)
```

A convenience method for adding multiple Integer columns. For example, to add columns “myIntegerCol\_0”, “myIntegerCol\_1”, “myIntegerCol\_2”, use {- code addColumnsInteger\(“myIntegerCol\_%d”,0,2\)}

* param pattern Pattern to use \(via String.format\). “%d” is replaced with column numbers
* param minIdxInclusive Minimum column index to use \(inclusive\)
* param maxIdxInclusive Maximum column index to use \(inclusive\)

**addColumnsInteger**

```text
public Builder addColumnsInteger(String pattern, int minIdxInclusive, int maxIdxInclusive,
                        Integer minAllowedValue, Integer maxAllowedValue)
```

A convenience method for adding multiple Integer columns. For example, to add columns “myIntegerCol\_0”, “myIntegerCol\_1”, “myIntegerCol\_2”, use {- code addColumnsInteger\(“myIntegerCol\_%d”,0,2\)}

* param pattern Pattern to use \(via String.format\). “%d” is replaced with column numbers
* param minIdxInclusive Minimum column index to use \(inclusive\)
* param maxIdxInclusive Maximum column index to use \(inclusive\)
* param minAllowedValue Minimum allowed value \(inclusive\). If null: no restriction
* param maxAllowedValue Maximum allowed value \(inclusive\). If null: no restriction

**addColumnCategorical**

```text
public Builder addColumnCategorical(String name, String... stateNames)
```

Add a Categorical column, with the specified state names

* param name Name of the column
* param stateNames Names of the allowable states for this categorical column

**addColumnCategorical**

```text
public Builder addColumnCategorical(String name, List<String> stateNames)
```

Add a Categorical column, with the specified state names

* param name Name of the column
* param stateNames Names of the allowable states for this categorical column

**addColumnLong**

```text
public Builder addColumnLong(String name)
```

Add a Long column, with no restrictions on the min/max values

* param name Name of the column

**addColumnLong**

```text
public Builder addColumnLong(String name, Long minAllowedValue, Long maxAllowedValue)
```

Add a Long column with the specified min/max allowable values

* param name Name of the column
* param minAllowedValue Minimum allowed value \(inclusive\). If null: no restriction
* param maxAllowedValue Maximum allowed value \(inclusive\). If null: no restriction

**addColumnsLong**

```text
public Builder addColumnsLong(String... names)
```

Add multiple Long columns, with no restrictions on the allowable values

* param names Names of the Long columns to add

**addColumnsLong**

```text
public Builder addColumnsLong(String pattern, int minIdxInclusive, int maxIdxInclusive)
```

A convenience method for adding multiple Long columns. For example, to add columns “myLongCol\_0”, “myLongCol\_1”, “myLongCol\_2”, use {- code addColumnsLong\(“myLongCol\_%d”,0,2\)}

* param pattern Pattern to use \(via String.format\). “%d” is replaced with column numbers
* param minIdxInclusive Minimum column index to use \(inclusive\)
* param maxIdxInclusive Maximum column index to use \(inclusive\)

**addColumnsLong**

```text
public Builder addColumnsLong(String pattern, int minIdxInclusive, int maxIdxInclusive, Long minAllowedValue,
                        Long maxAllowedValue)
```

A convenience method for adding multiple Long columns. For example, to add columns “myLongCol\_0”, “myLongCol\_1”, “myLongCol\_2”, use {- code addColumnsLong\(“myLongCol\_%d”,0,2\)}

* param pattern Pattern to use \(via String.format\). “%d” is replaced with column numbers
* param minIdxInclusive Minimum column index to use \(inclusive\)
* param maxIdxInclusive Maximum column index to use \(inclusive\)
* param minAllowedValue Minimum allowed value \(inclusive\). If null: no restriction
* param maxAllowedValue Maximum allowed value \(inclusive\). If null: no restriction

**addColumn**

```text
public Builder addColumn(ColumnMetaData metaData)
```

Add a column

* param metaData metadata for this column

**addColumnString**

```text
public Builder addColumnString(String name)
```

Add a String column with no restrictions on the allowable values.

* param name Name of the column

**addColumnsString**

```text
public Builder addColumnsString(String... columnNames)
```

Add multiple String columns with no restrictions on the allowable values

* param columnNames Names of the String columns to add

**addColumnString**

```text
public Builder addColumnString(String name, String regex, Integer minAllowableLength,
                        Integer maxAllowableLength)
```

Add a String column with the specified restrictions

* param name Name of the column
* param regex Regex that the String must match in order to be considered valid. If null: no regex restriction
* param minAllowableLength Minimum allowable length for the String to be considered valid
* param maxAllowableLength Maximum allowable length for the String to be considered valid

**addColumnsString**

```text
public Builder addColumnsString(String pattern, int minIdxInclusive, int maxIdxInclusive)
```

A convenience method for adding multiple numbered String columns. For example, to add columns “myStringCol\_0”, “myStringCol\_1”, “myStringCol\_2”, use {- code addColumnsString\(“myStringCol\_%d”,0,2\)}

* param pattern Pattern to use \(via String.format\). “%d” is replaced with column numbers
* param minIdxInclusive Minimum column index to use \(inclusive\)
* param maxIdxInclusive Maximum column index to use \(inclusive\)

**addColumnsString**

```text
public Builder addColumnsString(String pattern, int minIdxInclusive, int maxIdxInclusive, String regex,
                        Integer minAllowedLength, Integer maxAllowedLength)
```

A convenience method for adding multiple numbered String columns. For example, to add columns “myStringCol\_0”, “myStringCol\_1”, “myStringCol\_2”, use {- code addColumnsString\(“myStringCol\_%d”,0,2\)}

* param pattern Pattern to use \(via String.format\). “%d” is replaced with column numbers
* param minIdxInclusive Minimum column index to use \(inclusive\)
* param maxIdxInclusive Maximum column index to use \(inclusive\)
* param regex Regex that the String must match in order to be considered valid. If null: no regex restriction
* param minAllowedLength Minimum allowed length of strings \(inclusive\). If null: no restriction
* param maxAllowedLength Maximum allowed length of strings \(inclusive\). If null: no restriction

**addColumnTime**

```text
public Builder addColumnTime(String columnName, TimeZone timeZone)
```

Add a Time column with no restrictions on the min/max allowable times **NOTE**: Time columns are represented by LONG \(epoch millisecond\) values. For time values in human-readable formats, use String columns + StringToTimeTransform

* param columnName Name of the column
* param timeZone Time zone of the time column

**addColumnTime**

```text
public Builder addColumnTime(String columnName, DateTimeZone timeZone)
```

Add a Time column with no restrictions on the min/max allowable times **NOTE**: Time columns are represented by LONG \(epoch millisecond\) values. For time values in human-readable formats, use String columns + StringToTimeTransform

* param columnName Name of the column
* param timeZone Time zone of the time column

**addColumnTime**

```text
public Builder addColumnTime(String columnName, DateTimeZone timeZone, Long minValidValue, Long maxValidValue)
```

Add a Time column with the specified restrictions **NOTE**: Time columns are represented by LONG \(epoch millisecond\) values. For time values in human-readable formats, use String columns + StringToTimeTransform

* param columnName Name of the column
* param timeZone Time zone of the time column
* param minValidValue Minumum allowable time \(in milliseconds\). May be null.
* param maxValidValue Maximum allowable time \(in milliseconds\). May be null.

**addColumnNDArray**

```text
public Builder addColumnNDArray(String columnName, long[] shape)
```

Add a NDArray column

* param columnName Name of the column
* param shape shape of the NDArray column. Use -1 in entries to specify as “variable length” in that dimension

**build**

```text
public Schema build()
```

Create the Schema

**inferMultiple**

```text
public static Schema inferMultiple(List<List<Writable>> record)
```

Infers a schema based on the record. The column names are based on indexing.

* param record the record to infer from
* return the infered schema

**infer**

```text
public static Schema infer(List<Writable> record)
```

Infers a schema based on the record. The column names are based on indexing.

* param record the record to infer from
* return the infered schema

### SequenceSchema

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/schema/SequenceSchema.java)

**inferSequenceMulti**

```text
public static SequenceSchema inferSequenceMulti(List<List<List<Writable>>> record)
```

Infers a sequence schema based on the record

* param record the record to infer the schema based on
* return the inferred sequence schema

**inferSequence**

```text
public static SequenceSchema inferSequence(List<List<Writable>> record)
```

Infers a sequence schema based on the record

* param record the record to infer the schema based on
* return the inferred sequence schema


---
title: DataVec Transforms
short_title: Transforms
description: Data wrangling and mapping from one schema to another.
category: DataVec
weight: 1
---

# Transforms

## Data wrangling

One of the key tools in DataVec is transformations. DataVec helps the user map a dataset from one schema to another, and provides a list of operations to convert types, format data, and convert a 2D dataset to sequence data.

## Building a transform process

A transform process requires a `Schema` to successfully transform data. Both schema and transform process classes come with a helper `Builder` class which are useful for organizing code and avoiding complex constructors.

When both are combined together they look like the sample code below. Note how `inputDataSchema` is passed into the `Builder` constructor. Your transform process will fail to compile without it.

```java
import org.datavec.api.transform.TransformProcess;

TransformProcess tp = new TransformProcess.Builder(inputDataSchema)
    .removeColumns("CustomerID","MerchantID")
    .filter(new ConditionFilter(new CategoricalColumnCondition("MerchantCountryCode", ConditionOp.NotInSet, new HashSet<>(Arrays.asList("USA","CAN")))))
    .conditionalReplaceValueTransform(
        "TransactionAmountUSD",     //Column to operate on
        new DoubleWritable(0.0),    //New value to use, when the condition is satisfied
        new DoubleColumnCondition("TransactionAmountUSD",ConditionOp.LessThan, 0.0)) //Condition: amount < 0.0
    .stringToTimeTransform("DateTimeString","YYYY-MM-DD HH:mm:ss.SSS", DateTimeZone.UTC)
    .renameColumn("DateTimeString", "DateTime")
    .transform(new DeriveColumnsFromTimeTransform.Builder("DateTime").addIntegerDerivedColumn("HourOfDay", DateTimeFieldType.hourOfDay()).build())
    .removeColumns("DateTime")
    .build();
```

## Executing a transformation

Different "backends" for executors are available. Using the `tp` transform process above, here's how you can execute it locally using plain DataVec.

```java
import org.datavec.local.transforms.LocalTransformExecutor;

List<List<Writable>> processedData = LocalTransformExecutor.execute(originalData, tp);
```

## Debugging

Each operation in a transform process represents a "step" in schema changes. Sometimes, the resulting transformation is not the intended result. You can debug this by printing each step in the transform `tp` with the following:

```java
//Now, print the schema after each time step:
int numActions = tp.getActionList().size();

for(int i=0; i<numActions; i++ ){
    System.out.println("\n\n==================================================");
    System.out.println("-- Schema after step " + i + " (" + tp.getActionList().get(i) + ") --");

    System.out.println(tp.getSchemaAfterStep(i));
}
```

## Available transformations and conversions

### TransformProcess

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/TransformProcess.java)

A TransformProcess defines an ordered list of transformations to be executed on some data

**getFinalSchema**

```text
public Schema getFinalSchema()
```

Get the action list that this transform process will execute

* return

**getSchemaAfterStep**

```text
public Schema getSchemaAfterStep(int step)
```

Return the schema after executing all steps up to and including the specified step. Steps are indexed from 0: so getSchemaAfterStep\(0\) is after one transform has been executed.

* param step Index of the step
* return Schema of the data, after that \(and all prior\) steps have been executed

**toJson**

```text
public String toJson()
```

Execute the full sequence of transformations for a single example. May return null if example is filtered **NOTE:** Some TransformProcess operations cannot be done on examples individually. Most notably, ConvertToSequence and ConvertFromSequence operations require the full data set to be processed at once

* param input
* return

**toYaml**

```text
public String toYaml()
```

Convert the TransformProcess to a YAML string

* return TransformProcess, as YAML

**fromJson**

```text
public static TransformProcess fromJson(String json)
```

Deserialize a JSON String \(created by {- link \#toJson\(\)}\) to a TransformProcess

* return TransformProcess, from JSON

**fromYaml**

```text
public static TransformProcess fromYaml(String yaml)
```

Deserialize a JSON String \(created by {- link \#toJson\(\)}\) to a TransformProcess

* return TransformProcess, from JSON

**transform**

```text
public Builder transform(Transform transform)
```

Infer the categories for the given record reader for a particular column Note that each “column index” is a column in the context of: List record = ...; record.get\(columnIndex\);

Note that anything passed in as a column will be automatically converted to a string for categorical purposes.

The expected input is strings or numbers \(which have sensible toString\(\) representations\)

Note that the returned categories will be sorted alphabetically

* param recordReader the record reader to iterate through
* param columnIndex te column index to get categories for
* return

**filter**

```text
public Builder filter(Filter filter)
```

Add a filter operation to be executed after the previously-added operations have been executed

* param filter Filter operation to execute

**filter**

```text
public Builder filter(Condition condition)
```

Add a filter operation, based on the specified condition.

If condition is satisfied \(returns true\): remove the example or sequence  
If condition is not satisfied \(returns false\): keep the example or sequence

* param condition Condition to filter on

**removeColumns**

```text
public Builder removeColumns(String... columnNames)
```

Remove all of the specified columns, by name

* param columnNames Names of the columns to remove

**removeColumns**

```text
public Builder removeColumns(Collection<String> columnNames)
```

Remove all of the specified columns, by name

* param columnNames Names of the columns to remove

**removeAllColumnsExceptFor**

```text
public Builder removeAllColumnsExceptFor(String... columnNames)
```

Remove all columns, except for those that are specified here

* param columnNames Names of the columns to keep

**removeAllColumnsExceptFor**

```text
public Builder removeAllColumnsExceptFor(Collection<String> columnNames)
```

Remove all columns, except for those that are specified here

* param columnNames Names of the columns to keep

**renameColumn**

```text
public Builder renameColumn(String oldName, String newName)
```

Rename a single column

* param oldName Original column name
* param newName New column name

**renameColumns**

```text
public Builder renameColumns(List<String> oldNames, List<String> newNames)
```

Rename multiple columns

* param oldNames List of original column names
* param newNames List of new column names

**reorderColumns**

```text
public Builder reorderColumns(String... newOrder)
```

Reorder the columns using a partial or complete new ordering. If only some of the column names are specified for the new order, the remaining columns will be placed at the end, according to their current relative ordering

* param newOrder Names of the columns, in the order they will appear in the output

**duplicateColumn**

```text
public Builder duplicateColumn(String column, String newName)
```

Duplicate a single column

* param column Name of the column to duplicate
* param newName Name of the new \(duplicate\) column

**duplicateColumns**

```text
public Builder duplicateColumns(List<String> columnNames, List<String> newNames)
```

Duplicate a set of columns

* param columnNames Names of the columns to duplicate
* param newNames Names of the new \(duplicated\) columns

**integerMathOp**

```text
public Builder integerMathOp(String column, MathOp mathOp, int scalar)
```

Perform a mathematical operation \(add, subtract, scalar max etc\) on the specified integer column, with a scalar

* param column The integer column to perform the operation on
* param mathOp The mathematical operation
* param scalar The scalar value to use in the mathematical operation

**integerColumnsMathOp**

```text
public Builder integerColumnsMathOp(String newColumnName, MathOp mathOp, String... columnNames)
```

Calculate and add a new integer column by performing a mathematical operation on a number of existing columns. New column is added to the end.

* param newColumnName Name of the new/derived column
* param mathOp Mathematical operation to execute on the columns
* param columnNames Names of the columns to use in the mathematical operation

**longMathOp**

```text
public Builder longMathOp(String columnName, MathOp mathOp, long scalar)
```

Perform a mathematical operation \(add, subtract, scalar max etc\) on the specified long column, with a scalar

* param columnName The long column to perform the operation on
* param mathOp The mathematical operation
* param scalar The scalar value to use in the mathematical operation

**longColumnsMathOp**

```text
public Builder longColumnsMathOp(String newColumnName, MathOp mathOp, String... columnNames)
```

Calculate and add a new long column by performing a mathematical operation on a number of existing columns. New column is added to the end.

* param newColumnName Name of the new/derived column
* param mathOp Mathematical operation to execute on the columns
* param columnNames Names of the columns to use in the mathematical operation

**floatMathOp**

```text
public Builder floatMathOp(String columnName, MathOp mathOp, float scalar)
```

Perform a mathematical operation \(add, subtract, scalar max etc\) on the specified double column, with a scalar

* param columnName The float column to perform the operation on
* param mathOp The mathematical operation
* param scalar The scalar value to use in the mathematical operation

**floatColumnsMathOp**

```text
public Builder floatColumnsMathOp(String newColumnName, MathOp mathOp, String... columnNames)
```

Calculate and add a new float column by performing a mathematical operation on a number of existing columns. New column is added to the end.

* param newColumnName Name of the new/derived column
* param mathOp Mathematical operation to execute on the columns
* param columnNames Names of the columns to use in the mathematical operation

**floatMathFunction**

```text
public Builder floatMathFunction(String columnName, MathFunction mathFunction)
```

Perform a mathematical operation \(such as sin\(x\), ceil\(x\), exp\(x\) etc\) on a column

* param columnName Column name to operate on
* param mathFunction MathFunction to apply to the column

**doubleMathOp**

```text
public Builder doubleMathOp(String columnName, MathOp mathOp, double scalar)
```

Perform a mathematical operation \(add, subtract, scalar max etc\) on the specified double column, with a scalar

* param columnName The double column to perform the operation on
* param mathOp The mathematical operation
* param scalar The scalar value to use in the mathematical operation

**doubleColumnsMathOp**

```text
public Builder doubleColumnsMathOp(String newColumnName, MathOp mathOp, String... columnNames)
```

Calculate and add a new double column by performing a mathematical operation on a number of existing columns. New column is added to the end.

* param newColumnName Name of the new/derived column
* param mathOp Mathematical operation to execute on the columns
* param columnNames Names of the columns to use in the mathematical operation

**doubleMathFunction**

```text
public Builder doubleMathFunction(String columnName, MathFunction mathFunction)
```

Perform a mathematical operation \(such as sin\(x\), ceil\(x\), exp\(x\) etc\) on a column

* param columnName Column name to operate on
* param mathFunction MathFunction to apply to the column

**timeMathOp**

```text
public Builder timeMathOp(String columnName, MathOp mathOp, long timeQuantity, TimeUnit timeUnit)
```

Perform a mathematical operation \(add, subtract, scalar min/max only\) on the specified time column

* param columnName The integer column to perform the operation on
* param mathOp The mathematical operation
* param timeQuantity The quantity used in the mathematical op
* param timeUnit The unit that timeQuantity is specified in

**categoricalToOneHot**

```text
public Builder categoricalToOneHot(String... columnNames)
```

Convert the specified column\(s\) from a categorical representation to a one-hot representation. This involves the creation of multiple new columns each.

* param columnNames Names of the categorical column\(s\) to convert to a one-hot representation

**categoricalToInteger**

```text
public Builder categoricalToInteger(String... columnNames)
```

Convert the specified column\(s\) from a categorical representation to an integer representation. This will replace the specified categorical column\(s\) with an integer repreesentation, where each integer has the value 0 to numCategories-1.

* param columnNames Name of the categorical column\(s\) to convert to an integer representation

**integerToCategorical**

```text
public Builder integerToCategorical(String columnName, List<String> categoryStateNames)
```

Convert the specified column from an integer representation \(assume values 0 to numCategories-1\) to a categorical representation, given the specified state names

* param columnName Name of the column to convert
* param categoryStateNames Names of the states for the categorical column

**integerToCategorical**

```text
public Builder integerToCategorical(String columnName, Map<Integer, String> categoryIndexNameMap)
```

Convert the specified column from an integer representation to a categorical representation, given the specified mapping between integer indexes and state names

* param columnName Name of the column to convert
* param categoryIndexNameMap Names of the states for the categorical column

**integerToOneHot**

```text
public Builder integerToOneHot(String columnName, int minValue, int maxValue)
```

Convert an integer column to a set of 1 hot columns, based on the value in integer column

* param columnName Name of the integer column
* param minValue Minimum value possible for the integer column \(inclusive\)
* param maxValue Maximum value possible for the integer column \(inclusive\)

**addConstantColumn**

```text
public Builder addConstantColumn(String newColumnName, ColumnType newColumnType, Writable fixedValue)
```

Add a new column, where all values in the column are identical and as specified.

* param newColumnName Name of the new column
* param newColumnType Type of the new column
* param fixedValue Value in the new column for all records

**addConstantDoubleColumn**

```text
public Builder addConstantDoubleColumn(String newColumnName, double value)
```

Add a new double column, where the value for that column \(for all records\) are identical

* param newColumnName Name of the new column
* param value Value in the new column for all records

**addConstantIntegerColumn**

```text
public Builder addConstantIntegerColumn(String newColumnName, int value)
```

Add a new integer column, where th e value for that column \(for all records\) are identical

* param newColumnName Name of the new column
* param value Value of the new column for all records

**addConstantLongColumn**

```text
public Builder addConstantLongColumn(String newColumnName, long value)
```

Add a new integer column, where the value for that column \(for all records\) are identical

* param newColumnName Name of the new column
* param value Value in the new column for all records

**convertToString**

```text
public Builder convertToString(String inputColumn)
```

Convert the specified column to a string.

* param inputColumn the input column to convert
* return builder pattern

**convertToDouble**

```text
public Builder convertToDouble(String inputColumn)
```

Convert the specified column to a double.

* param inputColumn the input column to convert
* return builder pattern

**convertToInteger**

```text
public Builder convertToInteger(String inputColumn)
```

Convert the specified column to an integer.

* param inputColumn the input column to convert
* return builder pattern

**normalize**

```text
public Builder normalize(String column, Normalize type, DataAnalysis da)
```

Normalize the specified column with a given type of normalization

* param column Column to normalize
* param type Type of normalization to apply
* param da DataAnalysis object

**convertToSequence**

```text
public Builder convertToSequence(String keyColumn, SequenceComparator comparator)
```

Convert a set of independent records/examples into a sequence, according to some key. Within each sequence, values are ordered using the provided {- link SequenceComparator}

* param keyColumn Column to use as a key \(values with the same key will be combined into sequences\)
* param comparator A SequenceComparator to order the values within each sequence \(for example, by time or String order\)

**convertToSequence**

```text
public Builder convertToSequence()
```

Convert a set of independent records/examples into a sequence; each example is simply treated as a sequence of length 1, without any join/group operations. Note that more commonly, joining/grouping is required; use {- link \#convertToSequence\(List, SequenceComparator\)} for this functionality

**convertToSequence**

```text
public Builder convertToSequence(List<String> keyColumns, SequenceComparator comparator)
```

Convert a set of independent records/examples into a sequence, where each sequence is grouped according to one or more key values \(i.e., the values in one or more columns\) Within each sequence, values are ordered using the provided {- link SequenceComparator}

* param keyColumns Column to use as a key \(values with the same key will be combined into sequences\)
* param comparator A SequenceComparator to order the values within each sequence \(for example, by time or String order\)

**convertFromSequence**

```text
public Builder convertFromSequence()
```

Convert a sequence to a set of individual values \(by treating each value in each sequence as a separate example\)

**splitSequence**

```text
public Builder splitSequence(SequenceSplit split)
```

Split sequences into 1 or more other sequences. Used for example to split large sequences into a set of smaller sequences

* param split SequenceSplit that defines how splits will occur

**trimSequence**

```text
public Builder trimSequence(int numStepsToTrim, boolean trimFromStart)
```

SequenceTrimTranform removes the first or last N values in a sequence. Note that the resulting sequence may be of length 0, if the input sequence is less than or equal to N.

* param numStepsToTrim Number of time steps to trim from the sequence
* param trimFromStart If true: Trim values from the start of the sequence. If false: trim values from the end.

**offsetSequence**

```text
public Builder offsetSequence(List<String> columnsToOffset, int offsetAmount,
                                      SequenceOffsetTransform.OperationType operationType)
```

Perform a sequence of operation on the specified columns. Note that this also truncates sequences by the specified offset amount by default. Use {- code transform\(new SequenceOffsetTransform\(…\)} to change this. See {- link SequenceOffsetTransform} for details on exactly what this operation does and how.

* param columnsToOffset Columns to offset
* param offsetAmount Amount to offset the specified columns by \(positive offset: ‘columnsToOffset’ are moved to later time steps\)
* param operationType Whether the offset should be done in-place or by adding a new column

**reduce**

```text
public Builder reduce(IAssociativeReducer reducer)
```

Reduce \(i.e., aggregate/combine\) a set of examples \(typically by key\). **Note**: In the current implementation, reduction operations can be performed only on standard \(i.e., non-sequence\) data

* param reducer Reducer to use

**reduceSequence**

```text
public Builder reduceSequence(IAssociativeReducer reducer)
```

Reduce \(i.e., aggregate/combine\) a set of sequence examples - for each sequence individually. **Note**: This method results in non-sequence data. If you would instead prefer sequences of length 1 after the reduction, use {- code transform\(new ReduceSequenceTransform\(reducer\)\)}.

* param reducer Reducer to use to reduce each window

**reduceSequenceByWindow**

```text
public Builder reduceSequenceByWindow(IAssociativeReducer reducer, WindowFunction windowFunction)
```

Reduce \(i.e., aggregate/combine\) a set of sequence examples - for each sequence individually - using a window function. For example, take all records/examples in each 24-hour period \(i.e., using window function\), and convert them into a singe value \(using the reducer\). In this example, the output is a sequence, with time period of 24 hours.

* param reducer Reducer to use to reduce each window
* param windowFunction Window function to find apply on each sequence individually

**sequenceMovingWindowReduce**

```text
public Builder sequenceMovingWindowReduce(String columnName, int lookback, ReduceOp op)
```

SequenceMovingWindowReduceTransform: Adds a new column, where the value is derived by:  
\(a\) using a window of the last N values in a single column,  
\(b\) Apply a reduction op on the window to calculate a new value  
for example, this transformer can be used to implement a simple moving average of the last N values, or determine the minimum or maximum values in the last N time steps.

For example, for a simple moving average, length 20: {- code new SequenceMovingWindowReduceTransform\(“myCol”, 20, ReduceOp.Mean\)}

* param columnName Column name to perform windowing on
* param lookback Look back period for windowing
* param op Reduction operation to perform on each window

**calculateSortedRank**

```text
public Builder calculateSortedRank(String newColumnName, String sortOnColumn, WritableComparator comparator)
```

CalculateSortedRank: calculate the rank of each example, after sorting example. For example, we might have some numerical “score” column, and we want to know for the rank \(sort order\) for each example, according to that column.  
The rank of each example \(after sorting\) will be added in a new Long column. Indexing is done from 0; examples will have values 0 to dataSetSize-1.

Currently, CalculateSortedRank can only be applied on standard \(i.e., non-sequence\) data Furthermore, the current implementation can only sort on one column

* param newColumnName Name of the new column \(will contain the rank for each example\)
* param sortOnColumn Column to sort on
* param comparator Comparator used to sort examples

**calculateSortedRank**

```text
public Builder calculateSortedRank(String newColumnName, String sortOnColumn, WritableComparator comparator,
                                           boolean ascending)
```

CalculateSortedRank: calculate the rank of each example, after sorting example. For example, we might have some numerical “score” column, and we want to know for the rank \(sort order\) for each example, according to that column.  
The rank of each example \(after sorting\) will be added in a new Long column. Indexing is done from 0; examples will have values 0 to dataSetSize-1.

Currently, CalculateSortedRank can only be applied on standard \(i.e., non-sequence\) data Furthermore, the current implementation can only sort on one column

* param newColumnName Name of the new column \(will contain the rank for each example\)
* param sortOnColumn Column to sort on
* param comparator Comparator used to sort examples
* param ascending If true: sort ascending. False: descending

**stringToCategorical**

```text
public Builder stringToCategorical(String columnName, List<String> stateNames)
```

Convert the specified String column to a categorical column. The state names must be provided.

* param columnName Name of the String column to convert to categorical
* param stateNames State names of the category

**stringRemoveWhitespaceTransform**

```text
public Builder stringRemoveWhitespaceTransform(String columnName)
```

Remove all whitespace characters from the values in the specified String column

* param columnName Name of the column to remove whitespace from

**stringMapTransform**

```text
public Builder stringMapTransform(String columnName, Map<String, String> mapping)
```

Replace one or more String values in the specified column with new values.

Keys in the map are the original values; the Values in the map are their replacements. If a String appears in the data but does not appear in the provided map \(as a key\), that String values will not be modified.

* param columnName Name of the column in which to do replacement
* param mapping Map of oldValues -&gt; newValues

**stringToTimeTransform**

```text
public Builder stringToTimeTransform(String column, String format, DateTimeZone dateTimeZone)
```

Convert a String column \(containing a date/time String\) to a time column \(by parsing the date/time String\)

* param column String column containing the date/time Strings
* param format Format of the strings. Time format is specified as per [http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)
* param dateTimeZone Timezone of the column

**stringToTimeTransform**

```text
public Builder stringToTimeTransform(String column, String format, DateTimeZone dateTimeZone, Locale locale)
```

Convert a String column \(containing a date/time String\) to a time column \(by parsing the date/time String\)

* param column String column containing the date/time Strings
* param format Format of the strings. Time format is specified as per [http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)
* param dateTimeZone Timezone of the column
* param locale Locale of the column

**appendStringColumnTransform**

```text
public Builder appendStringColumnTransform(String column, String toAppend)
```

Append a String to a specified column

* param column Column to append the value to
* param toAppend String to append to the end of each writable

**conditionalReplaceValueTransform**

```text
public Builder conditionalReplaceValueTransform(String column, Writable newValue, Condition condition)
```

Replace the values in a specified column with a specified new value, if some condition holds. If the condition does not hold, the original values are not modified.

* param column Column to operate on
* param newValue Value to use as replacement, if condition is satisfied
* param condition Condition that must be satisfied for replacement

**conditionalReplaceValueTransformWithDefault**

```text
public Builder conditionalReplaceValueTransformWithDefault(String column, Writable yesVal, Writable noVal, Condition condition)
```

Replace the values in a specified column with a specified “yes” value, if some condition holds. Replace it with a “no” value, otherwise.

* param column Column to operate on
* param yesVal Value to use as replacement, if condition is satisfied
* param noVal Value to use as replacement, if condition is not satisfied
* param condition Condition that must be satisfied for replacement

**conditionalCopyValueTransform**

```text
public Builder conditionalCopyValueTransform(String columnToReplace, String sourceColumn, Condition condition)
```

Replace the value in a specified column with a new value taken from another column, if a condition is satisfied/true.  
Note that the condition can be any generic condition, including on other column\(s\), different to the column that will be modified if the condition is satisfied/true.

* param columnToReplace Name of the column in which values will be replaced \(if condition is satisfied\)
* param sourceColumn Name of the column from which the new values will be
* param condition Condition to use

**replaceStringTransform**

```text
public Builder replaceStringTransform(String columnName, Map<String, String> mapping)
```

Replace one or more String values in the specified column that match regular expressions.

Keys in the map are the regular expressions; the Values in the map are their String replacements. For example:

> | Original | Regex | Replacement | Result |
> | :--- | :--- | :--- | :--- |
> | Data\_Vec | \_ |  | DataVec |
> | B1C2T3 | \d | one | BoneConeTone |
> | '  4.25 ' | ^\s+\|\s+$ |  | '4.25' |

* param columnName Name of the column in which to do replacement
* param mapping Map of old values or regular expression to new values

**ndArrayScalarOpTransform**

```text
public Builder ndArrayScalarOpTransform(String columnName, MathOp op, double value)
```

Element-wise NDArray math operation \(add, subtract, etc\) on an NDArray column

* param columnName Name of the NDArray column to perform the operation on
* param op Operation to perform
* param value Value for the operation

**ndArrayColumnsMathOpTransform**

```text
public Builder ndArrayColumnsMathOpTransform(String newColumnName, MathOp mathOp, String... columnNames)
```

Perform an element wise mathematical operation \(such as add, subtract, multiply\) on NDArray columns. The existing columns are unchanged, a new NDArray column is added

* param newColumnName Name of the new NDArray column
* param mathOp Operation to perform
* param columnNames Name of the columns used as input to the operation

**ndArrayMathFunctionTransform**

```text
public Builder ndArrayMathFunctionTransform(String columnName, MathFunction mathFunction)
```

Apply an element wise mathematical function \(sin, tanh, abs etc\) to an NDArray column. This operation is performed in place.

* param columnName Name of the column to perform the operation on
* param mathFunction Mathematical function to apply

**ndArrayDistanceTransform**

```text
public Builder ndArrayDistanceTransform(String newColumnName, Distance distance, String firstCol,
                                                String secondCol)
```

Calculate a distance \(cosine similarity, Euclidean, Manhattan\) on two equal-sized NDArray columns. This operation adds a new Double column \(with the specified name\) with the result.

* param newColumnName Name of the new column \(result\) to add
* param distance Distance to apply
* param firstCol first column to use in the distance calculation
* param secondCol second column to use in the distance calculation

**firstDigitTransform**

```text
public Builder firstDigitTransform(String inputColumn, String outputColumn)
```

FirstDigitTransform converts a column to a categorical column, with values being the first digit of the number.  
For example, “3.1415” becomes “3” and “2.0” becomes “2”.  
Negative numbers ignore the sign: “-7.123” becomes “7”.  
Note that two {- link FirstDigitTransform.Mode}s are supported, which determines how non-numerical entries should be handled:  
EXCEPTION\_ON\_INVALID: output has 10 category values \(“0”, …, “9”\), and any non-numerical values result in an exception  
INCLUDE\_OTHER\_CATEGORY: output has 11 category values \(“0”, …, “9”, “Other”\), all non-numerical values are mapped to “Other”

FirstDigitTransform is useful \(combined with {- link CategoricalToOneHotTransform} and Reductions\) to implement [Benford’s law](https://en.wikipedia.org/wiki/Benford%27s_law).

* param inputColumn Input column name
* param outputColumn Output column name. If same as input, input column is replaced

**firstDigitTransform**

```text
public Builder firstDigitTransform(String inputColumn, String outputColumn, FirstDigitTransform.Mode mode)
```

FirstDigitTransform converts a column to a categorical column, with values being the first digit of the number.  
For example, “3.1415” becomes “3” and “2.0” becomes “2”.  
Negative numbers ignore the sign: “-7.123” becomes “7”.  
Note that two {- link FirstDigitTransform.Mode}s are supported, which determines how non-numerical entries should be handled:  
EXCEPTION\_ON\_INVALID: output has 10 category values \(“0”, …, “9”\), and any non-numerical values result in an exception  
INCLUDE\_OTHER\_CATEGORY: output has 11 category values \(“0”, …, “9”, “Other”\), all non-numerical values are mapped to “Other”

FirstDigitTransform is useful \(combined with {- link CategoricalToOneHotTransform} and Reductions\) to implement [Benford’s law](https://en.wikipedia.org/wiki/Benford%27s_law).

* param inputColumn Input column name
* param outputColumn Output column name. If same as input, input column is replaced
* param mode See {- link FirstDigitTransform.Mode}

**build**

```text
public TransformProcess build()
```

Create the TransformProcess object

### CategoricalToIntegerTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/categorical/CategoricalToIntegerTransform.java)

Created by Alex on 4/03/2016.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### CategoricalToOneHotTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/categorical/CategoricalToOneHotTransform.java)

Created by Alex on 4/03/2016.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### IntegerToCategoricalTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/categorical/IntegerToCategoricalTransform.java)

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

### PivotTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/categorical/PivotTransform.java)

Pivot transform operates on two columns:

* a categorical column that operates as a key, and
* Another column that contains a value Essentially, Pivot transform takes keyvalue pairs and breaks them out into separate columns.

For example, with schema \[col0, key, value, col3\] and values with key in {a,b,c} Output schema is \[col0, key\[a\], key\[b\], key\[c\], col3\] and input \(col0Val, b, x, col3Val\) gets mapped to \(col0Val, 0, x, 0, col3Val\).

When expanding columns, a default value is used - for example 0 for numerical columns.

**transform**

```text
public Schema transform(Schema inputSchema)
```

* param keyColumnName Key column to expand
* param valueColumnName Name of the column that contains the value

### StringToCategoricalTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/categorical/StringToCategoricalTransform.java)

Convert a String column to a categorical column

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

### AddConstantColumnTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/column/AddConstantColumnTransform.java)

Add a new column, where the values in that column for all records are identical \(according to the specified value\)

### DuplicateColumnsTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/column/DuplicateColumnsTransform.java)

Duplicate one or more columns. The duplicated columns are placed immediately after the original columns

**transform**

```text
public Schema transform(Schema inputSchema)
```

* param columnsToDuplicate List of columns to duplicate
* param newColumnNames List of names for the new \(duplicate\) columns

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### RemoveAllColumnsExceptForTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/column/RemoveAllColumnsExceptForTransform.java)

Transform that removes all columns except for those that are explicitly specified as ones to keep To specify only the columns

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### RemoveColumnsTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/column/RemoveColumnsTransform.java)

Remove the specified columns from the data. To specify only the columns to keep,

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### RenameColumnsTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/column/RenameColumnsTransform.java)

Rename one or more columns

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### ReorderColumnsTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/column/ReorderColumnsTransform.java)

Rearrange the order of the columns. Note: A partial list of columns can be used here. Any columns that are not explicitly mentioned will be placed after those that are in the output, without changing their relative order.

**transform**

```text
public Schema transform(Schema inputSchema)
```

* param newOrder A partial or complete order of the columns in the output

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### ConditionalCopyValueTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/condition/ConditionalCopyValueTransform.java)

Replace the value in a specified column with a new value taken from another column, if a condition is satisfied/true.  
Note that the condition can be any generic condition, including on other column\(s\), different to the column that will be modified if the condition is satisfied/true.

**Note**: For sequences, this transform use the convention that each step in the sequence is passed to the condition, and replaced \(or not\) separately \(i.e., Condition.condition\(List\) is used on each time step individually\)

**transform**

```text
public Schema transform(Schema inputSchema)
```

* param columnToReplace Name of the column in which to replace the old value
* param sourceColumn Name of the column to get the new value from
* param condition Condition

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### ConditionalReplaceValueTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/condition/ConditionalReplaceValueTransform.java)

Replace the value in a specified column with a new value, if a condition is satisfied/true.  
Note that the condition can be any generic condition, including on other column\(s\), different to the column that will be modified if the condition is satisfied/true.

**Note**: For sequences, this transform use the convention that each step in the sequence is passed to the condition, and replaced \(or not\) separately \(i.e., Condition.condition\(List\) is used on each time step individually\)

**transform**

```text
public Schema transform(Schema inputSchema)
```

* param columnToReplace Name of the column in which to replace the old value with ‘newValue’, if the condition holds
* param newValue New value to use
* param condition Condition

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### ConditionalReplaceValueTransformWithDefault

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/condition/ConditionalReplaceValueTransformWithDefault.java)

Replace the value in a specified column with a ‘yes’ value, if a condition is satisfied/true.  
Replace the value of this same column with a ‘no’ value otherwise. Note that the condition can be any generic condition, including on other column\(s\), different to the column that will be modified if the condition is satisfied/true.

**Note**: For sequences, this transform use the convention that each step in the sequence is passed to the condition, and replaced \(or not\) separately \(i.e., Condition.condition\(List\) is used on each time step individually\)

### ConvertToDouble

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/doubletransform/ConvertToDouble.java)

Convert any value to an Double

**map**

```text
public DoubleWritable map(Writable writable)
```

* param column Name of the column to convert to a Double column

### DoubleColumnsMathOpTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/doubletransform/DoubleColumnsMathOpTransform.java)

Add a new double column, calculated from one or more other columns. A new column \(with the specified name\) is added as the final column of the output. No other columns are modified.  
For example, if newColumnName==”newCol”, mathOp==Add, and columns=={“col1”,”col2”}, then the output column with name “newCol” has value col1+col2.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

### DoubleMathFunctionTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/doubletransform/DoubleMathFunctionTransform.java)

A simple transform to do common mathematical operations, such as sin\(x\), ceil\(x\), etc.

### DoubleMathOpTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/doubletransform/DoubleMathOpTransform.java)

Double mathematical operation.  
This is an in-place operation of the double column value and a double scalar.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

### Log2Normalizer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/doubletransform/Log2Normalizer.java)

Normalize by taking scale log2\(\(in-columnMin\)/\(mean-columnMin\) + 1\) Maps values in range \(columnMin to infinity\) to \(0 to infinity\) Most suitable for values with a geometric/negative exponential type distribution.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### MinMaxNormalizer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/doubletransform/MinMaxNormalizer.java)

Normalizer to map \(min to max\) -&gt; \(newMin-to newMax\) linearly.

Mathematically: \(newMax-newMin\)/\(max-min\) \(x-min\) + newMin

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### StandardizeNormalizer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/doubletransform/StandardizeNormalizer.java)

Normalize using \(x-mean\)/stdev. Also known as a standard score, standardization etc.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### SubtractMeanNormalizer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/doubletransform/SubtractMeanNormalizer.java)

Normalize by substracting the mean

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### ConvertToInteger

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/integer/ConvertToInteger.java)

Convert any value to an Integer.

**map**

```text
public IntWritable map(Writable writable)
```

* param column Name of the column to convert to an integer

### IntegerColumnsMathOpTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/integer/IntegerColumnsMathOpTransform.java)

Add a new integer column, calculated from one or more other columns. A new column \(with the specified name\) is added as the final column of the output. No other columns are modified.  
For example, if newColumnName==”newCol”, mathOp==MathOp.Add, and columns=={“col1”,”col2”}, then the output column with name “newCol” has value col1+col2.  
**NOTE**: Division here is using if a decimal output value is required.

**toString**

```text
public String toString()
```

* param newColumnName Name of the new column \(output column\)
* param mathOp Mathematical operation. Only Add/Subtract/Multiply/Divide/Modulus is allowed here
* param columns Columns to use in the mathematical operation

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

### IntegerMathOpTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/integer/IntegerMathOpTransform.java)

Integer mathematical operation.  
This is an in-place operation of the integer column value and an integer scalar.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### IntegerToOneHotTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/integer/IntegerToOneHotTransform.java)

Convert an integer column to a set of one-hot columns.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### ReplaceEmptyIntegerWithValueTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/integer/ReplaceEmptyIntegerWithValueTransform.java)

Replace an empty/missing integer with a certain value.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### ReplaceInvalidWithIntegerTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/integer/ReplaceInvalidWithIntegerTransform.java)

Replace an invalid \(non-integer\) value in a column with a specified integer

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### LongColumnsMathOpTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/longtransform/LongColumnsMathOpTransform.java)

Add a new long column, calculated from one or more other columns. A new column \(with the specified name\) is added as the final column of the output. No other columns are modified.  
For example, if newColumnName==”newCol”, mathOp==MathOp.Add, and columns=={“col1”,”col2”}, then the output column with name “newCol” has value col1+col2.  
if a decimal output value is required.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

### LongMathOpTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/longtransform/LongMathOpTransform.java)

Long mathematical operation.  
This is an in-place operation of the long column value and an long scalar.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### TextToCharacterIndexTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/nlp/TextToCharacterIndexTransform.java)

Convert each text value in a sequence to a longer sequence of integer indices. For example, “abc” would be converted to \[1, 2, 3\]. Values in other columns will be duplicated.

### TextToTermIndexSequenceTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/nlp/TextToTermIndexSequenceTransform.java)

Convert each text value in a sequence to a longer sequence of integer indices. For example, “zero one two” would be converted to \[0, 1, 2\]. Values in other columns will be duplicated.

### SequenceDifferenceTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/sequence/SequenceDifferenceTransform.java)

SequenceDifferenceTransform: for an input sequence, calculate the difference on one column.  
For each time t, calculate someColumn\(t\) - someColumn\(t-s\), where s &gt;= 1 is the ‘lookback’ period.

Note: at t=0 \(i.e., the first step in a sequence; or more generally, for all times t &lt; s\), there is no previous value these time steps:

1. Default: output = someColumn\(t\) - someColumn\(max\(t-s, 0\)\)
2. SpecifiedValue: output = someColumn\(t\) - someColumn\(t-s\) if t-s &gt;= 0, or a custom Writable object \(for example, a DoubleWritable\(0\) or NullWritable\).

Note: this is an in-place operation: i.e., the values in each column are modified. If the original values are and apply the difference operation in-place on the copy.

**outputColumnName**

```text
public String outputColumnName()
```

Create a SequenceDifferenceTransform with default lookback of 1, and using FirstStepMode.Default. Output column name is the same as the input column name.

* param columnName Name of the column to perform the operation on.

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

#### SequenceMovingWindowReduceTransform <a id="sequencemovingwindowreducetransform"></a>

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/sequence/SequenceMovingWindowReduceTransform.java)

SequenceMovingWindowReduceTransform Adds a new column, where the value is derived by:  
\(a\) using a window of the last N values in a single column,  
\(b\) Apply a reduction op on the window to calculate a new value  
for example, this transformer can be used to implement a simple moving average of the last N values, or determine the minimum or maximum values in the last N time steps.

**defaultOutputColumnName**

```text
public static String defaultOutputColumnName(String originalName, int lookback, ReduceOp op)
```

Enumeration to specify how each cases are handled: For example, for a look back period of 20, how should the first 19 output values be calculated?  
Default: Perform your former reduction as normal, with as many values are available  
SpecifiedValue: use the given/specified value instead of the actual output value. For example, you could assign values of 0 or NullWritable to positions 0 through 18 of the output.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### SequenceOffsetTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/sequence/SequenceOffsetTransform.java)

Sequence offset transform takes a sequence, and shifts The values in one or more columns by a specified number of times steps. It has 2 modes of operation \(OperationType enum\), with respect to the columns it operates on:  
InPlace: operations may be performed in-place, modifying the values in the specified columns  
NewColumn: operations may produce new columns, with the original \(source\) columns remaining unmodified

Additionally, there are 2 modes for handling values outside the original sequence \(EdgeHandling enum\): TrimSequence: the entire sequence is trimmed \(start or end\) by a specified number of steps  
SpecifiedValue: for any values outside of the original sequence, they are given a specified value

Note 1: When specifying offsets, they are done as follows: Positive offsets: move the values in the specified columns to a later time. Earlier time steps are either be trimmed or Given specified values; the last values in these columns will be truncated/removed.

Note 2: Care must be taken when using TrimSequence: for example, if we chain multiple sequence offset transforms on the one dataset, we may end up trimming much more than we want. In this case, it may be better to use SpecifiedValue, at the end.

### AppendStringColumnTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/string/AppendStringColumnTransform.java)

Append a String to the values in a single column

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### ChangeCaseStringTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/string/ChangeCaseStringTransform.java)

Change case \(to, e.g, all lower case\) of String column.

### ConcatenateStringColumns

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/string/ConcatenateStringColumns.java)

Concatenate values of one or more String columns into a new String column. Retains the constituent String columns so user must remove those manually, if desired.

TODO: use new String Reduce functionality in DataVec?

**transform**

```text
public Schema transform(Schema inputSchema)
```

* param columnsToConcatenate A partial or complete order of the columns in the output

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### ConvertToString

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/string/ConvertToString.java)

Convert any value to a string.

**map**

```text
public Text map(Writable writable)
```

Transform the writable in to a string

* param writable the writable to transform
* return the string form of this writable

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### MapAllStringsExceptListTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/string/MapAllStringsExceptListTransform.java)

This method maps all String values, except those is the specified list, to a single String value

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### RemoveWhiteSpaceTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/string/RemoveWhiteSpaceTransform.java)

String transform that removes all whitespace charaters

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### ReplaceEmptyStringTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/string/ReplaceEmptyStringTransform.java)

Replace empty String values with the specified String

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### ReplaceStringTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/string/ReplaceStringTransform.java)

Replaces String values that match regular expressions.

**map**

```text
public Text map(final Writable writable)
```

Constructs a new ReplaceStringTransform using the specified

* param columnName Name of the column
* param map Key: regular expression; Value: replacement value

### StringListToCategoricalSetTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/string/StringListToCategoricalSetTransform.java)

Convert a delimited String to a list of binary categorical columns. Suppose the possible String values were {“a”,”b”,”c”,”d”} and the String column value to be converted contained the String “a,c”, then the 4 output columns would have values \[“true”,”false”,”true”,”false”\]

**transform**

```text
public Schema transform(Schema inputSchema)
```

* param columnName The name of the column to convert
* param newColumnNames The names of the new columns to create
* param categoryTokens The possible tokens that may be present. Note this list must have the same length and order as the newColumnNames list
* param delimiter The delimiter for the Strings to convert

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### StringListToCountsNDArrayTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/string/StringListToCountsNDArrayTransform.java)

Converts String column into a bag-of-words \(BOW\) represented as an NDArray of “counts.”  
Note that the original column is removed in the process

**transform**

```text
public Schema transform(Schema inputSchema)
```

* param columnName The name of the column to convert
* param vocabulary The possible tokens that may be present.
* param delimiter The delimiter for the Strings to convert
* param ignoreUnknown Whether to ignore unknown tokens

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**outputColumnName**

```text
public String outputColumnName()
```

The output column name after the operation has been applied

* return the output column name

**columnName**

```text
public String columnName()
```

The output column names This will often be the same as the input

* return the output column names

### StringListToIndicesNDArrayTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/string/StringListToIndicesNDArrayTransform.java)

Converts String column into a sparse bag-of-words \(BOW\) represented as an NDArray of indices. Appropriate for embeddings or as efficient storage before being expanded into a dense array.

### StringMapTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/string/StringMapTransform.java)

A simple String -&gt; String map function.

Keys in the map are the original values; the Values in the map are their replacements. If a String appears in the data but does not appear in the provided map \(as a key\), that String values will not be modified.

**map**

```text
public Text map(Writable writable)
```

* param columnName Name of the column
* param map Key: From. Value: To

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### DeriveColumnsFromTimeTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/time/DeriveColumnsFromTimeTransform.java)

Create a number of new columns by deriving their values from a Time column. Can be used for example to create new columns with the year, month, day, hour, minute, second etc.

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

**mapSequence**

```text
public Object mapSequence(Object sequence)
```

Transform a sequence

* param sequence

**toString**

```text
public String toString()
```

The output column name after the operation has been applied

* return the output column name

### StringToTimeTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/time/StringToTimeTransform.java)

Convert a String column to a time column by parsing the date/time String, using a JodaTime.

Time format is specified as per [http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)

**getNewColumnMetaData**

```text
public ColumnMetaData getNewColumnMetaData(String newName, ColumnMetaData oldColumnType)
```

Instantiate this without a time format specified. If this constructor is used, this transform will be allowed to handle several common transforms as defined in the static formats array.

* param columnName Name of the String column
* param timeZone Timezone for time parsing

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable

### TimeMathOpTransform

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/transform/time/TimeMathOpTransform.java)

Transform math op on a time column

Note: only the following MathOps are supported: Add, Subtract, ScalarMin, ScalarMax  
For ScalarMin/Max, the TimeUnit must be milliseconds - i.e., value must be in epoch millisecond format

**map**

```text
public Object map(Object input)
```

Transform an object in to another object

* param input the record to transform
* return the transformed writable


---
title: "Reductions"
description: "DataVec reduction operations — aggregating, grouping, and summarizing records and sequences"
---

# Reductions

Reductions aggregate multiple records (or time steps within a sequence) into a single record. They are the primary tool for:

- Collapsing a group of rows that share the same key into one summarized row
- Reducing sequences into a fixed-size feature vector
- Computing geographic midpoints from coordinate strings

DataVec provides two core reducer classes: `Reducer` for numerical and general column data, and `StringReducer` for string-column aggregation.

## Reducer

`Reducer` (in `org.datavec.api.transform.reduce`) collapses groups of records into single records. The columns you specify for reduction get aggregated; the remaining key columns are left as-is.

### Basic usage

```java
import org.datavec.api.transform.reduce.Reducer;
import org.datavec.api.transform.ReduceOp;

Reducer reducer = new Reducer.Builder(ReduceOp.Mean)
    .keyColumns("CustomerID")          // group by this column
    .meanColumns("TransactionAmount")  // compute mean of this column
    .countColumns("TransactionAmount") // add a count column
    .sumColumns("Refunds")
    .build();
```

### Reduction operations

`ReduceOp` specifies the default operation applied to any column not individually configured.

| `ReduceOp` | Description |
|---|---|
| `Sum` | Sum of all values |
| `Mean` | Arithmetic mean |
| `Count` | Number of records |
| `CountUnique` | Number of distinct values |
| `TakeFirst` | First value (in encounter order) |
| `TakeLast` | Last value (in encounter order) |
| `Min` | Minimum value |
| `Max` | Maximum value |
| `Range` | max - min |
| `Stdev` | Sample standard deviation |
| `Variance` | Sample variance |
| `UncorrectedStdDev` | Population standard deviation |
| `PopulationVariance` | Population variance |
| `Prod` | Product of all values |
| `Append` | Concatenate string values |
| `Prepend` | Prepend-concatenate string values |

### Column-level configuration

You can override the default operation per column using the builder methods:

```java
Reducer reducer = new Reducer.Builder(ReduceOp.TakeFirst)
    .keyColumns("OrderID")
    .sumColumns("Quantity", "Price")
    .maxColumns("Rating")
    .minColumns("DeliveryDays")
    .countColumns("ItemID")
    .customReduction("Tags", myCustomColumnReduction)
    .setIgnoreInvalid("Price")     // skip rows where Price is invalid
    .build();
```

### Ignoring invalid values

`setIgnoreInvalid(String... columns)` configures the reducer to skip invalid values in the listed columns when computing the aggregate. Invalid is defined relative to the column's `ColumnMetaData`.

### Custom column reductions

Implement `ColumnReduction` to provide a custom aggregation function for a specific column:

```java
public interface ColumnReduction {
    Writable reduceColumn(List<Writable> columnValues);
    String getColumnOutputName(String columnInputName);
    ColumnMetaData getColumnOutputMetaData(String newColumnName, ColumnMetaData columnInputMeta);
}
```

Then register it:

```java
reducer.customReduction("myColumn", new MyColumnReduction());
```

---

## StringReducer

`StringReducer` is a reducer that operates on string columns. It supports the same grouping concept as `Reducer` but provides string-specific operations: append, prepend, merge, and replace.

[source](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/stringreduce/StringReducer.java)

```java
import org.datavec.api.transform.stringreduce.StringReducer;

StringReducer reducer = new StringReducer.Builder(StringReduceOp.Merge)
    .keyColumns("ProductID")
    .appendColumns("Tags")       // append all tag values
    .prependColumns("Prefix")    // prepend all prefix values
    .replaceColumn("Status")     // use last value (replace)
    .mergeColumns("Description") // merge all values with separator
    .build();
```

### Builder methods

| Method | Effect |
|---|---|
| `appendColumns(String... cols)` | Concatenate all values, appending each new value to the end |
| `prependColumns(String... cols)` | Concatenate values, prepending each new value to the start |
| `mergeColumns(String... cols)` | Merge all values using a separator |
| `replaceColumn(String... cols)` | Replace previous value with each new value (keeps last) |
| `customReduction(String col, ColumnReduction r)` | Use a custom reduction for the named column |
| `setIgnoreInvalid(String... cols)` | Skip invalid values during reduction |
| `outputColumnName(String name)` | Set the output column name |

---

## GeographicMidpointReduction

[source](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/reduce/impl/GeographicMidpointReduction.java)

A specialized reduction that computes the geographic midpoint from a column of latitude/longitude coordinate strings. This is useful when you have a set of GPS pings or location records and want to reduce them to a single representative point.

The algorithm follows the method described at [geomidpoint.com](http://www.geomidpoint.com/methods.html), which converts spherical coordinates to Cartesian, computes the average, and converts back.

```java
import org.datavec.api.transform.reduce.impl.GeographicMidpointReduction;

// Column "Coordinates" contains strings like "lat,long"
GeographicMidpointReduction geoReduction = new GeographicMidpointReduction(",");
```

Constructor parameter: `delim` — the delimiter used to separate latitude and longitude within the string, for example `","` for `"40.7128,-74.0060"`.

The output column will also be a string in `"lat,long"` format representing the computed midpoint.

---

## Executing reductions

### Locally

```java
import org.datavec.local.transforms.LocalTransformExecutor;
import org.datavec.api.transform.join.Join;

// Execute a reduction inline in a transform process
TransformProcess tp = new TransformProcess.Builder(schema)
    .reduce(reducer)
    .build();

List<List<Writable>> reduced = LocalTransformExecutor.execute(data, tp);
```

### On Spark

```java
import org.datavec.spark.transform.SparkTransformExecutor;

JavaRDD<List<Writable>> reduced = SparkTransformExecutor.execute(inputRdd, tp);
```

---

## Joins (group and combine)

`Join` combines two datasets on a common key. Combined with `Reducer`, joins can produce rich aggregated views over multiple data sources.

```java
import org.datavec.api.transform.join.Join;

Join join = new Join.Builder(Join.JoinType.Inner)
    .setJoinColumns("CustomerID")
    .setSchemas(leftSchema, rightSchema)
    .build();

List<List<Writable>> joined = LocalTransformExecutor.executeJoin(join, leftData, rightData);
```

Join types:

| Type | Description |
|---|---|
| `Inner` | Only records with matching keys in both datasets |
| `LeftOuter` | All left records; matched right records or nulls |
| `RightOuter` | All right records; matched left records or nulls |
| `FullOuter` | All records from both datasets; unmatched sides get nulls |

---

## Sequence reduction

When working with sequence data (time series), reductions can collapse a variable-length sequence into a fixed-size feature vector. The same `Reducer` API applies — each time step is treated as one record, and the resulting summary is one record per sequence.

```java
// Reduce each sequence to one record using per-column operations
TransformProcess tp = new TransformProcess.Builder(sequenceSchema)
    .reduceSequenceByWindow(reducer, windowFunction)
    .build();
```

See `ReduceSequenceByWindowTransform` for windowed reductions, which are useful for sliding-window feature engineering on time series.

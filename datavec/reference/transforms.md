---
title: "Transforms"
description: "TransformProcess — data transformations, column operations, type conversions, and sequences"
---

# Transforms

A `TransformProcess` is an ordered list of operations applied sequentially to every record in a dataset. Each operation sees the record as it was left by the previous operation, allowing you to chain complex multi-step pipelines from simple building blocks.

The `TransformProcess.Builder` API is fluent and validates each operation against the current schema state at build time. If you reference a column that does not exist, apply a numeric operation to a string column, or produce a result that breaks the schema, you get an exception when calling `.build()` rather than at runtime.

## Building a TransformProcess

```java
import org.datavec.api.transform.TransformProcess;
import org.datavec.api.transform.condition.ConditionOp;
import org.datavec.api.transform.condition.column.*;

TransformProcess tp = new TransformProcess.Builder(inputDataSchema)
    // Remove columns that aren't features
    .removeColumns("CustomerID", "MerchantID")
    // Filter: keep only USA and CAN transactions
    .filter(new ConditionFilter(
        new CategoricalColumnCondition("MerchantCountryCode",
            ConditionOp.NotInSet, new HashSet<>(Arrays.asList("USA","CAN")))
    ))
    // Replace negative amounts with 0
    .conditionalReplaceValueTransform(
        "TransactionAmountUSD",
        new DoubleWritable(0.0),
        new DoubleColumnCondition("TransactionAmountUSD", ConditionOp.LessThan, 0.0)
    )
    // Parse a date string into epoch milliseconds
    .stringToTimeTransform("DateTimeString", "YYYY-MM-DD HH:mm:ss.SSS", DateTimeZone.UTC)
    .renameColumn("DateTimeString", "DateTime")
    // Extract hour-of-day as a new integer column
    .transform(new DeriveColumnsFromTimeTransform.Builder("DateTime")
        .addIntegerDerivedColumn("HourOfDay", DateTimeFieldType.hourOfDay())
        .build())
    .removeColumns("DateTime")
    .build();
```

## Column Management

### Removing Columns

```java
// Remove specific columns by name
.removeColumns("id", "timestamp", "raw_text")

// Remove all columns except the ones you want to keep
.removeAllColumnsExceptFor("feature1", "feature2", "label")
```

### Renaming Columns

```java
.renameColumn("OldName", "NewName")

// Rename multiple at once
.renameColumns(
    Arrays.asList("col_a", "col_b"),
    Arrays.asList("alpha", "beta")
)
```

### Reordering Columns

```java
// Named columns go first in the given order; remaining columns follow
.reorderColumns("label", "feature1", "feature2")
```

### Duplicating Columns

```java
.duplicateColumn("income", "income_backup")
```

### Adding Constant Columns

Useful when downstream systems require a fixed column that is the same for all records.

```java
.addConstantDoubleColumn("bias", 1.0)
.addConstantIntegerColumn("version", 3)
.addConstantLongColumn("batchID", 12345L)
.addConstantColumn("flag", ColumnType.String, new Text("active"))
```

## Type Conversions

### String to Numeric

```java
.convertToDouble("priceStr")      // "19.99" -> 19.99 (Double column)
.convertToInteger("countStr")     // "42" -> 42 (Integer column)
.convertToString("userId")        // any column -> String
```

### Categorical Encoding

**One-Hot Encoding**: Replaces a categorical column with N binary columns (one per state).

```java
// "color" with states ["red","green","blue"] becomes three Integer columns:
// "color[red]", "color[green]", "color[blue]"
.categoricalToOneHot("color")
.categoricalToOneHot("color", "size")   // multiple columns
```

**Integer Encoding**: Replaces the categorical column with a single integer (0 to numStates-1).

```java
.categoricalToInteger("priority")   // "low"->0, "medium"->1, "high"->2
```

**Integer to Categorical**: The reverse — assign state names to integer values.

```java
.integerToCategorical("labelIdx", Arrays.asList("cat", "dog", "bird"))
// or with explicit mapping
.integerToCategorical("labelIdx", Map.of(0, "cat", 1, "dog", 2, "bird"))
```

**Integer to One-Hot**: Directly expand an integer column to one-hot, given known min/max.

```java
.integerToOneHot("classId", 0, 9)   // integers 0-9 -> 10 binary columns
```

**String to Categorical**: Attach known state names to an existing string column.

```java
.stringToCategorical("tier", Arrays.asList("bronze", "silver", "gold"))
```

## Mathematical Operations

### Scalar Operations on a Single Column

```java
// MathOp values: Add, Subtract, Multiply, Divide, Modulus, ScalarMin, ScalarMax
.doubleMathOp("price", MathOp.Multiply, 1.1)      // 10% markup
.integerMathOp("count", MathOp.Add, 1)             // increment
.longMathOp("timestamp", MathOp.Subtract, 3600000L) // subtract 1 hour
.floatMathOp("weight", MathOp.Divide, 1000.0f)     // grams to kg
```

### Math Functions on a Column

```java
// MathFunction values: Abs, Ceil, Floor, Log, Log2, Log10, Exp, Sin, Cos, Sqrt...
.doubleMathFunction("logReturn", MathFunction.Log)
.floatMathFunction("angle", MathFunction.Sin)
```

### Derived Column from Multiple Columns

```java
// Adds "totalRevenue" = col1 + col2 + col3
.doubleColumnsMathOp("totalRevenue", MathOp.Add, "q1Rev", "q2Rev", "q3Rev", "q4Rev")
.integerColumnsMathOp("totalCount", MathOp.Add, "clicks", "impressions")
```

### Time Arithmetic

```java
// Shift a time column by a quantity
.timeMathOp("eventTime", MathOp.Add, 30, TimeUnit.MINUTES)
```

## String Operations

### Remove Whitespace

```java
.stringRemoveWhitespaceTransform("zipCode")
```

### Append to String

```java
.appendStringColumnTransform("urlPath", "?ref=datavec")
```

### Map Replacements

Replace specific string values with new values:

```java
Map<String, String> replacements = new HashMap<>();
replacements.put("n/a", "");
replacements.put("N/A", "");
replacements.put("none", "");

.stringMapTransform("description", replacements)
```

## Time Transforms

### Parse String to Time

```java
// Converts "2024-01-15 14:32:01.000" -> epoch milliseconds as Long
.stringToTimeTransform("dateStr", "YYYY-MM-DD HH:mm:ss.SSS", DateTimeZone.UTC)

// With locale
.stringToTimeTransform("dateStr", "MMM dd, YYYY", DateTimeZone.UTC, Locale.ENGLISH)
```

### Derive Integer Columns from Time

Extract date/time components from a Time column as new Integer columns:

```java
.transform(new DeriveColumnsFromTimeTransform.Builder("eventTime")
    .addIntegerDerivedColumn("year", DateTimeFieldType.year())
    .addIntegerDerivedColumn("month", DateTimeFieldType.monthOfYear())
    .addIntegerDerivedColumn("dayOfWeek", DateTimeFieldType.dayOfWeek())
    .addIntegerDerivedColumn("hourOfDay", DateTimeFieldType.hourOfDay())
    .build())
```

## Conditional Transforms

### Replace a Value When Condition Is True

```java
// If amount < 0, replace with 0.0
.conditionalReplaceValueTransform(
    "amount",
    new DoubleWritable(0.0),
    new DoubleColumnCondition("amount", ConditionOp.LessThan, 0.0)
)
```

### Replace with One of Two Values Based on Condition

```java
// If isActive is true, use 1; otherwise use 0
.conditionalReplaceValueTransformWithDefault(
    "activeFlag",
    new IntWritable(1),    // yesVal
    new IntWritable(0),    // noVal
    new BooleanColumnCondition("isActive", ConditionOp.Equal, true)
)
```

### Copy a Value from Another Column Conditionally

```java
// If "override" is non-null, copy its value to "price"
.conditionalCopyValueTransform("price", "override", condition)
```

## Filtering

Filters are applied inline during `TransformProcess` execution. A record is removed if the condition returns true.

```java
// Remove records where country is not in the allowed set
.filter(new ConditionFilter(
    new CategoricalColumnCondition("country",
        ConditionOp.NotInSet, new HashSet<>(Arrays.asList("USA","CAN","GBR")))
))

// Shorthand: pass condition directly
.filter(new DoubleColumnCondition("price", ConditionOp.LessThan, 0.0))
```

## Normalization

DataVec includes a built-in `normalize` operation that applies standard statistical normalization inline in the transform, given a pre-computed `DataAnalysis` object. The type parameter controls which normalization strategy to apply.

```java
DataAnalysis da = AnalyzeLocal.analyze(schema, reader, 100);

.normalize("income", Normalize.Standardize, da)    // zero mean, unit variance
.normalize("age", Normalize.MinMax, da)             // scale to [0,1]
.normalize("score", Normalize.Log2Mean, da)         // log2 normalization
```

For full control over normalizer fitting and serialization, use `NormalizerStandardize` or `NormalizerMinMaxScaler` as a pre-processor on the `DataSetIterator` instead. See [Normalization](normalization.md).

## Sorting and Ranking

### Calculate Sorted Rank

Adds a new Long column containing the rank (0 to N-1) of each record when sorted by a given column:

```java
.calculateSortedRank(
    "scoreRank",          // new column name
    "score",              // sort column
    new DoubleWritableComparator(),  // comparator
    false                 // false = descending (highest score = rank 0)
)
```

## Sequence Transforms

### Convert Records to Sequences

Group flat records into sequences by a key column:

```java
// Group by "userID", order within each group by "timestamp"
.convertToSequence("userID", new NumericalColumnComparator("timestamp", true))

// Group by multiple keys
.convertToSequence(
    Arrays.asList("userID", "sessionID"),
    new NumericalColumnComparator("timestamp", true)
)
```

### Convert Sequences Back to Records

Flatten sequences into individual records:

```java
.convertFromSequence()
```

### Split Sequences

Break large sequences into smaller ones:

```java
// Split every 50 time steps
.splitSequence(new NumStepsSequenceSplit(50, 1))

// Split at specific values in a column
.splitSequence(new ConditionSequenceSplit(
    new IntegerColumnCondition("eventType", ConditionOp.Equal, 0)
))
```

### Trim Sequences

Remove time steps from the start or end:

```java
.trimSequence(5, true)    // remove first 5 time steps
.trimSequence(5, false)   // remove last 5 time steps
```

### Offset Sequence Columns

Shift column values forward or backward in time (for creating prediction targets):

```java
// Shift "target" column forward 1 step (use current features to predict next target)
.offsetSequence(Arrays.asList("target"), 1,
    SequenceOffsetTransform.OperationType.InPlace)
```

### Moving Window Reduction

Compute a rolling statistic over the last N time steps:

```java
// Add "priceAvg5" = mean of last 5 values of "price"
.sequenceMovingWindowReduce("price", 5, ReduceOp.Mean)
```

### Reduce Sequences

Aggregate a whole sequence into a single record:

```java
Reducer reducer = new Reducer.Builder(ReduceOp.Mean)
    .meanColumns("feature1", "feature2")
    .maxColumns("maxSignal")
    .countColumns("numEvents")
    .build();

.reduceSequence(reducer)
```

## Applying Custom Transforms

If none of the built-in transforms meet your needs, implement the `Transform` interface directly and add it via `.transform(yourCustomTransform)`.

```java
public class MyTransform implements Transform {
    @Override
    public Schema transform(Schema inputSchema) {
        // Return modified schema after this transform
    }

    @Override
    public List<Writable> map(List<Writable> writables) {
        // Apply your custom logic and return new record
    }

    // ... other interface methods
}

// Register in builder
.transform(new MyTransform())
```

## Executing the Transform

After building a `TransformProcess`, execute it with `LocalTransformExecutor` or `SparkTransformExecutor`:

```java
import org.datavec.local.transforms.LocalTransformExecutor;

List<List<Writable>> processed = LocalTransformExecutor.execute(originalData, tp);
```

See [Executors](executors.md) for full details.

## Debugging Transforms

Print the schema state after each step to diagnose unexpected results:

```java
int numSteps = tp.getActionList().size();
for (int i = 0; i < numSteps; i++) {
    System.out.println("After step " + i + " (" + tp.getActionList().get(i) + "):");
    System.out.println(tp.getSchemaAfterStep(i));
}

System.out.println("Final schema:");
System.out.println(tp.getFinalSchema());
```

This is particularly helpful when a later transform fails because the schema no longer matches expectations after earlier steps.

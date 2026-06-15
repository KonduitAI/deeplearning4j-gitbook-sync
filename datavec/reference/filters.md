---
title: "Filters"
description: "Data filtering in DataVec — removing records based on conditions"
---

# Filters

Filters remove records from a dataset during the `TransformProcess` execution. A filter is evaluated against every record; if the filter's `removeExample` method returns `true`, that record is dropped from the output.

Filters are the primary mechanism for data cleaning. Real-world datasets commonly contain records with missing values, out-of-range numbers, wrong column counts, or constraint violations. Rather than letting these records corrupt your training data, filters remove them early in the pipeline.

## Using Filters in a TransformProcess

Filters are applied inline as steps in a `TransformProcess`:

```java
TransformProcess tp = new TransformProcess.Builder(schema)
    // First filter: remove records from countries outside our scope
    .filter(new ConditionFilter(
        new CategoricalColumnCondition("country",
            ConditionOp.NotInSet,
            new HashSet<>(Arrays.asList("USA", "CAN", "GBR")))
    ))
    // Second filter: remove records with invalid values in numeric columns
    .filter(new FilterInvalidValues("age", "income"))
    .build();
```

Filters are applied in order. A record that survives the first filter is evaluated by the second, and so on. Multiple filters compound: a record is kept only if it passes every filter.

## ConditionFilter

`ConditionFilter` wraps any `Condition` into a filter. The semantics are:
- If the condition returns **true**: the record is **removed**
- If the condition returns **false**: the record is **kept**

```java
import org.datavec.api.transform.filter.ConditionFilter;
import org.datavec.api.transform.condition.column.DoubleColumnCondition;
import org.datavec.api.transform.condition.ConditionOp;

// Remove records where "price" is negative
Filter negativePriceFilter = new ConditionFilter(
    new DoubleColumnCondition("price", ConditionOp.LessThan, 0.0)
);

// Remove records where "status" is "deleted"
Filter deletedFilter = new ConditionFilter(
    new CategoricalColumnCondition("status", ConditionOp.Equal, "deleted")
);

// Remove records where score is NaN OR Infinite
Filter badScoreFilter = new ConditionFilter(
    BooleanCondition.OR(
        new NaNColumnCondition("score"),
        new InfiniteColumnCondition("score")
    )
);
```

Any `Condition` — including compound `AND`, `OR`, `NOT` combinations — can be wrapped in a `ConditionFilter`. See [Conditions](conditions.md) for the full condition API.

As a shorthand, `TransformProcess.Builder.filter(Condition condition)` automatically wraps the condition in a `ConditionFilter`:

```java
// Equivalent to .filter(new ConditionFilter(new DoubleColumnCondition(...)))
.filter(new DoubleColumnCondition("price", ConditionOp.LessThan, 0.0))
```

## FilterInvalidValues

`FilterInvalidValues` removes any record that contains values that are invalid according to the column's declared type and constraints in the `Schema`.

"Invalid" means:
- A value that cannot be parsed as the declared type (e.g., the string `"abc"` in a Double column)
- A numeric value outside the declared min/max range
- A categorical value not in the declared state list
- A string that fails the declared regex or length constraints

```java
import org.datavec.api.transform.filter.FilterInvalidValues;

// Remove records with invalid values in "age" or "income"
Filter invalidFilter = new FilterInvalidValues("age", "income");

// Remove records with invalid values in ANY column
Filter allColumnsFilter = new FilterInvalidValues();
```

When no column names are given, the filter checks all columns in the schema. This is useful as a broad sanity check at the start of a pipeline before more targeted transforms.

## InvalidNumColumns

`InvalidNumColumns` removes records that do not have the expected number of columns. This is useful for CSV files where some rows are corrupted or have stray extra delimiters.

```java
import org.datavec.api.transform.filter.InvalidNumColumns;

// Schema has 5 columns; remove records with != 5 values
Filter wrongWidth = new InvalidNumColumns(schema);
```

This filter compares the actual number of `Writable` values in each record against `schema.numColumns()`. Records with too few or too many columns are removed.

## Implementing a Custom Filter

If the built-in filters do not cover your use case, implement the `Filter` interface:

```java
import org.datavec.api.transform.filter.Filter;

public class MyCustomFilter implements Filter {
    @Override
    public boolean removeExample(Object writables) {
        List<Writable> record = (List<Writable>) writables;
        // Return true to REMOVE the record, false to keep it
        double value = record.get(2).toDouble();
        return Double.isNaN(value) || value < 0;
    }

    @Override
    public boolean removeSequence(Object sequence) {
        // Called for sequence data
        List<List<Writable>> seq = (List<List<Writable>>) sequence;
        return seq.isEmpty();
    }

    @Override
    public Schema transform(Schema inputSchema) {
        return inputSchema; // filters don't change the schema
    }
}
```

Add the custom filter to a `TransformProcess`:

```java
TransformProcess tp = new TransformProcess.Builder(schema)
    .filter(new MyCustomFilter())
    .build();
```

## Filter vs. Conditional Replace

Filters and conditional transforms address the same class of problems — records with bad values — but with different strategies:

| Strategy | When to use |
|---|---|
| **Filter** (remove the record) | When bad records are unrecoverable and you have enough data to spare |
| **Conditional replace** (fix the value) | When you can substitute a sensible default (e.g., 0.0 for negative prices) |

In practice, use filtering for structural problems (wrong column count, completely missing values) and conditional replace for recoverable data quality issues (out-of-range but fixable values).

## Order Matters

Because filters in a `TransformProcess` are applied in sequence, earlier filters can simplify later ones:

```java
TransformProcess tp = new TransformProcess.Builder(schema)
    // Step 1: Remove structurally broken records first
    .filter(new InvalidNumColumns(schema))
    // Step 2: Now safe to check values knowing all records have the right shape
    .filter(new FilterInvalidValues("age", "income"))
    // Step 3: Business logic filtering
    .filter(new ConditionFilter(
        new DoubleColumnCondition("income", ConditionOp.GreaterThan, 1_000_000.0)
    ))
    .build();
```

Placing cheap structural checks (like `InvalidNumColumns`) before expensive value-level checks reduces unnecessary work on malformed records.

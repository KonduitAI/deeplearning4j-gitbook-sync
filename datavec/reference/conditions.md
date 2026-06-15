---
title: "Conditions"
description: "Conditional operations in DataVec — filtering and transforming data based on conditions"
---

# Conditions

A `Condition` is a predicate over a record (or sequence) that returns true or false. Conditions are the building blocks of two things in DataVec:

1. **Filters** — remove records where a condition is true
2. **Conditional transforms** — replace or copy values in a column when a condition is met

Most conditions are column-level: they inspect the value of a specific column and compare it against a threshold, set, or pattern.

## The Condition Interface

All conditions implement `Condition`:

```java
public interface Condition {
    boolean condition(Object input);             // evaluate on a full record
    boolean conditionSequence(Object sequence); // evaluate on a sequence
    Schema transform(Schema inputSchema);        // schema is unchanged for conditions
}
```

When used in a filter, a record is removed if `condition(record)` returns **true**. Keep this direction in mind when writing conditions — it is the opposite of what some filter libraries use.

## Column Conditions

Column conditions apply to a specific named column, using a `ConditionOp` to specify the comparison.

### ConditionOp Values

| ConditionOp | Meaning |
|---|---|
| `Equal` | value == threshold |
| `NotEqual` | value != threshold |
| `LessThan` | value < threshold |
| `LessThanOrEqual` | value <= threshold |
| `GreaterThan` | value > threshold |
| `GreaterThanOrEqual` | value >= threshold |
| `InSet` | value is in a set |
| `NotInSet` | value is not in a set |

### DoubleColumnCondition

Checks a double-precision column against a threshold:

```java
import org.datavec.api.transform.condition.column.DoubleColumnCondition;
import org.datavec.api.transform.condition.ConditionOp;

// True if "price" < 0.0
Condition negativePrice = new DoubleColumnCondition("price", ConditionOp.LessThan, 0.0);

// True if "score" >= 0.9
Condition highScore = new DoubleColumnCondition("score", ConditionOp.GreaterThanOrEqual, 0.9);
```

### IntegerColumnCondition

```java
import org.datavec.api.transform.condition.column.IntegerColumnCondition;

// True if "age" < 18
Condition minor = new IntegerColumnCondition("age", ConditionOp.LessThan, 18);

// True if "retryCount" > 3
Condition tooManyRetries = new IntegerColumnCondition("retryCount", ConditionOp.GreaterThan, 3);
```

### LongColumnCondition

```java
import org.datavec.api.transform.condition.column.LongColumnCondition;

// True if "timestamp" is before a certain epoch millisecond value
long cutoff = DateTime.parse("2024-01-01").getMillis();
Condition beforeCutoff = new LongColumnCondition("timestamp", ConditionOp.LessThan, cutoff);
```

### StringColumnCondition

Supports only `Equal` and `NotEqual` operators on string columns:

```java
import org.datavec.api.transform.condition.column.StringColumnCondition;

// True if "status" == "active"
Condition isActive = new StringColumnCondition("status", ConditionOp.Equal, "active");

// True if "status" != "deleted"
Condition notDeleted = new StringColumnCondition("status", ConditionOp.NotEqual, "deleted");
```

### CategoricalColumnCondition

Applies to categorical columns. Supports `Equal`, `NotEqual`, `InSet`, and `NotInSet`:

```java
import org.datavec.api.transform.condition.column.CategoricalColumnCondition;

// True if "tier" == "gold"
Condition isGold = new CategoricalColumnCondition("tier", ConditionOp.Equal, "gold");

// True if "country" is NOT in the allowed set
Condition notAllowed = new CategoricalColumnCondition(
    "country",
    ConditionOp.NotInSet,
    new HashSet<>(Arrays.asList("USA", "CAN", "GBR"))
);
```

### TimeColumnCondition

Compares a Time column (stored as epoch milliseconds) against a threshold:

```java
import org.datavec.api.transform.condition.column.TimeColumnCondition;

long oneDayAgoMs = System.currentTimeMillis() - (24 * 60 * 60 * 1000L);

// True if "eventTime" < one day ago (i.e., old records)
Condition oldRecord = new TimeColumnCondition("eventTime", ConditionOp.LessThan, oneDayAgoMs);
```

### BooleanColumnCondition

```java
import org.datavec.api.transform.condition.column.BooleanColumnCondition;

// True if "isActive" == true
Condition active = new BooleanColumnCondition("isActive", ConditionOp.Equal, true);
```

## Null and Invalid Value Conditions

### NullWritableColumnCondition

True when the value in the specified column is a `NullWritable` (the DataVec representation of a missing value):

```java
import org.datavec.api.transform.condition.column.NullWritableColumnCondition;

// True if "email" is null/missing
Condition emailMissing = new NullWritableColumnCondition("email");
```

### NaNColumnCondition

True when a floating-point column contains NaN:

```java
import org.datavec.api.transform.condition.column.NaNColumnCondition;

Condition hasNaN = new NaNColumnCondition("sensorReading");
```

### InfiniteColumnCondition

True when a floating-point column contains positive or negative infinity:

```java
import org.datavec.api.transform.condition.column.InfiniteColumnCondition;

Condition isInfinite = new InfiniteColumnCondition("logLoss");
```

### InvalidValueColumnCondition

True whenever a column's value cannot be parsed as its declared type (e.g., a string where a Long is expected, or a value outside the declared min/max range):

```java
import org.datavec.api.transform.condition.column.InvalidValueColumnCondition;

// True if "age" contains a value invalid for its Integer column type
Condition invalidAge = new InvalidValueColumnCondition("age");
```

This is particularly useful with `FilterInvalidValues` when you want to remove rather than fix bad records.

## Regex Condition

### StringRegexColumnCondition

True if the string value in a column matches (or does not match) a regex:

```java
import org.datavec.api.transform.condition.string.StringRegexColumnCondition;

// True if "zipCode" matches exactly 5 digits
Condition validZip = new StringRegexColumnCondition("zipCode", "\\d{5}");

// Can be applied to non-String columns too — uses Writable.toString()
```

## Sequence Length Condition

### SequenceLengthCondition

True when a sequence's length satisfies a comparison:

```java
import org.datavec.api.transform.condition.sequence.SequenceLengthCondition;

// True if the sequence has fewer than 10 time steps
Condition tooShort = new SequenceLengthCondition(ConditionOp.LessThan, 10);

// True if the sequence has exactly 100 steps
Condition exactLength = new SequenceLengthCondition(ConditionOp.Equal, 100);
```

## Boolean Logic: AND, OR, NOT, XOR

`BooleanCondition` provides static factory methods to combine conditions:

### AND

True only if all component conditions are true:

```java
import org.datavec.api.transform.condition.BooleanCondition;

Condition richAdult = BooleanCondition.AND(
    new IntegerColumnCondition("age", ConditionOp.GreaterThanOrEqual, 18),
    new DoubleColumnCondition("income", ConditionOp.GreaterThan, 50000.0)
);
```

### OR

True if any component condition is true:

```java
Condition badRecord = BooleanCondition.OR(
    new NaNColumnCondition("score"),
    new InfiniteColumnCondition("score"),
    new NullWritableColumnCondition("score")
);
```

### NOT

Inverts a condition:

```java
// True when "status" is NOT "active"
Condition notActive = BooleanCondition.NOT(
    new CategoricalColumnCondition("status", ConditionOp.Equal, "active")
);
```

### XOR

True when exactly one of the two conditions is true:

```java
Condition xorCondition = BooleanCondition.XOR(conditionA, conditionB);
```

### Nesting

Boolean conditions can be nested to arbitrary depth:

```java
// Remove records that are either:
// (a) from an unknown country, OR
// (b) from an allowed country but with a negative price
Condition toFilter = BooleanCondition.OR(
    new CategoricalColumnCondition("country", ConditionOp.NotInSet,
        new HashSet<>(Arrays.asList("USA", "CAN"))),
    BooleanCondition.AND(
        new CategoricalColumnCondition("country", ConditionOp.InSet,
            new HashSet<>(Arrays.asList("USA", "CAN"))),
        new DoubleColumnCondition("price", ConditionOp.LessThan, 0.0)
    )
);
```

## Sequence Condition Mode

For single-column conditions applied to sequences, you can control how the condition is evaluated across all time steps:

- `SequenceConditionMode.And` — the condition is true for the sequence only if it is true at **every** time step
- `SequenceConditionMode.Or` — the condition is true for the sequence if it is true at **any** time step
- `SequenceConditionMode.NoSequenceMode` — applying this condition to a sequence throws an error

Most column condition constructors accept an optional `SequenceConditionMode` parameter:

```java
// True for a sequence if ANY time step has price < 0
Condition anyNegative = new DoubleColumnCondition(
    "price",
    ConditionOp.LessThan,
    0.0,
    SequenceConditionMode.Or
);
```

## Using Conditions in a TransformProcess

### As a Filter

```java
TransformProcess tp = new TransformProcess.Builder(schema)
    // Remove records where country is not in allowed set
    .filter(new ConditionFilter(
        new CategoricalColumnCondition("country",
            ConditionOp.NotInSet, new HashSet<>(Arrays.asList("USA","CAN")))
    ))
    // Shorthand: pass condition directly (creates a ConditionFilter internally)
    .filter(new DoubleColumnCondition("price", ConditionOp.LessThan, 0.0))
    .build();
```

### In a Conditional Replace

```java
TransformProcess tp = new TransformProcess.Builder(schema)
    // Replace negative prices with 0.0
    .conditionalReplaceValueTransform(
        "price",
        new DoubleWritable(0.0),
        new DoubleColumnCondition("price", ConditionOp.LessThan, 0.0)
    )
    // Replace with one of two values based on a boolean condition
    .conditionalReplaceValueTransformWithDefault(
        "flag",
        new Text("yes"),
        new Text("no"),
        new BooleanColumnCondition("isActive", ConditionOp.Equal, true)
    )
    .build();
```

Conditions are evaluated at runtime for every record. Constructing complex nested conditions has essentially no overhead compared to the I/O of reading the data itself.

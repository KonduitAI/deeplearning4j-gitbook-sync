---
title: "Schema"
description: "Defining data schemas — Schema, ColumnType, SequenceSchema, and schema inference"
---

# Schema

A `Schema` is the cornerstone of every DataVec transform pipeline. It defines the structure of your tabular data: how many columns there are, what each column is named, and what type of values it holds. The `TransformProcess` requires a schema to validate every operation at build time, so mismatches between your data and your transformation logic are caught early rather than at runtime.

## Why Schemas Matter

Data is rarely clean. Columns have unexpected types, values fall outside expected ranges, and string columns contain numbers that look categorical. By declaring a schema explicitly, you give DataVec:

1. A reference for type-checking each transform operation
2. A source of truth for valid value ranges, which filters can use to remove bad records
3. A serializable representation of your data layout that can travel with your model to production

## Building a Schema

Use `Schema.Builder` to construct a schema. The builder provides a fluent API where each `addColumn*` call appends a column definition.

```java
import org.datavec.api.transform.schema.Schema;
import org.joda.time.DateTimeZone;
import java.util.Arrays;

Schema schema = new Schema.Builder()
    .addColumnString("customerID")
    .addColumnString("email")
    .addColumnInteger("age")
    .addColumnDouble("accountBalanceUSD", 0.0, null, false, false)
    .addColumnCategorical("country", Arrays.asList("USA", "CAN", "GBR", "DEU", "FRA"))
    .addColumnTime("registrationDate", DateTimeZone.UTC)
    .addColumnCategorical("tier", Arrays.asList("bronze", "silver", "gold", "platinum"))
    .build();
```

The schema above declares seven columns in order. Order matters: column index 0 is `customerID`, index 1 is `email`, and so on. Any transform that references a column by index uses this ordering.

## Column Types

### Integer

A 32-bit signed integer. Optionally constrained to a min/max range.

```java
.addColumnInteger("age")
.addColumnInteger("quantity", 0, 10000)   // min 0, max 10000
```

Add multiple integer columns with a pattern:

```java
// Adds columns "feature_0", "feature_1", ..., "feature_9"
.addColumnsInteger("feature_%d", 0, 9)
```

### Long

A 64-bit signed integer. Useful for IDs, timestamps stored as epoch milliseconds, or any value that overflows int.

```java
.addColumnLong("transactionID")
.addColumnLong("eventTimestampMs", 0L, null)   // must be non-negative
```

### Double

A 64-bit floating point. By default, NaN and infinite values are rejected. You can allow them explicitly.

```java
.addColumnDouble("price")
.addColumnDouble("ratio", 0.0, 1.0)                          // bounded
.addColumnDouble("logLoss", null, null, false, false)        // unbounded, no NaN, no Inf
.addColumnDouble("rawScore", null, null, true, false)        // allow NaN, no Inf
```

### Float

A 32-bit floating point with the same options as Double.

```java
.addColumnFloat("embedding_0")
.addColumnFloat("embedding_1", null, null, false, false)

// Convenient multi-column add: "emb_0", "emb_1", ..., "emb_127"
.addColumnsFloat("emb_%d", 0, 127)
```

### String

Arbitrary text. Optionally constrained by a regex, minimum length, and maximum length.

```java
.addColumnString("description")
.addColumnString("zipCode", "\\d{5}", 5, 5)      // exactly 5 digits
.addColumnString("username", null, 3, 32)         // length 3 to 32
```

Add multiple string columns at once:

```java
.addColumnsString("field_0", "field_1", "field_2")
// or
.addColumnsString("field_%d", 0, 2)
```

### Categorical

A string column with a fixed, known set of allowed values (the "state names"). Categorical columns can be converted to one-hot encoding or integer encoding via `TransformProcess`.

```java
.addColumnCategorical("color", Arrays.asList("red", "green", "blue"))
.addColumnCategorical("priority", "low", "medium", "high")   // varargs form
```

Every value in the column must match one of the declared state names; values outside this set are considered invalid by schema validation.

### Time

A time column is stored internally as epoch milliseconds (a `Long`), but carries timezone information and optional min/max bounds. For data where timestamps arrive as human-readable strings, use a `String` column plus `stringToTimeTransform` in your `TransformProcess`.

```java
.addColumnTime("eventTime", DateTimeZone.UTC)
.addColumnTime("eventTime", DateTimeZone.UTC, 0L, null)   // must be after epoch
```

### NDArray

An embedded multidimensional array. Use -1 for variable-length dimensions.

```java
.addColumnNDArray("imageVector", new long[]{1, 28, 28})
.addColumnNDArray("embedding", new long[]{-1})    // variable length
```

### Boolean

A true/false column.

```java
.addColumnBoolean("isActive")
```

### Bytes

A raw byte array. Primarily used for binary blob columns.

```java
.addColumnBytes("rawPayload")
```

## Inspecting a Schema

```java
Schema schema = ...;

int n = schema.numColumns();                         // number of columns
String name = schema.getName(0);                     // name of column 0
ColumnType type = schema.getType("customerID");      // type by name
ColumnMetaData meta = schema.getMetaData("age");     // full metadata
int idx = schema.getIndexOfColumn("email");          // index by name
boolean exists = schema.hasColumn("missingCol");     // existence check
```

## Serializing a Schema

Schemas serialize to JSON or YAML, which allows you to save and reload them alongside your trained model.

```java
// Serialize
String json = schema.toJson();
String yaml = schema.toYaml();

// Deserialize
Schema reloaded = Schema.fromJson(json);
Schema reloadedYaml = Schema.fromYaml(yaml);
```

Storing the schema alongside your `TransformProcess` (also JSON-serializable) means you can reconstruct the full preprocessing pipeline at inference time without re-specifying it in code.

## SequenceSchema

`SequenceSchema` extends `Schema` for time series and sequence data. It has the same column definitions but is used with sequence-aware readers (like `CSVSequenceRecordReader`) and sequence transforms.

```java
import org.datavec.api.transform.schema.SequenceSchema;

SequenceSchema seqSchema = new SequenceSchema.Builder()
    .addColumnLong("timestampMs")
    .addColumnDouble("sensorValue")
    .addColumnCategorical("event", Arrays.asList("none", "alert", "critical"))
    .build();
```

`SequenceSchema` is passed to `TransformProcess.Builder` just like a regular schema:

```java
TransformProcess tp = new TransformProcess.Builder(seqSchema)
    .categoricalToInteger("event")
    .build();
```

## Schema Inference

If you have a sample record and want DataVec to guess a schema rather than declaring one manually, use the static inference methods on `Schema`:

```java
// Infer from a single record (List<Writable>)
List<Writable> sampleRecord = reader.next();
Schema inferred = Schema.infer(sampleRecord);

// Infer from multiple records (List<List<Writable>>)
List<List<Writable>> samples = new ArrayList<>();
for (int i = 0; i < 100 && reader.hasNext(); i++) {
    samples.add(reader.next());
}
Schema inferred = Schema.inferMultiple(samples);
```

Inferred schemas use column names `0`, `1`, `2`, ... and can only detect `Integer`, `Long`, `Double`, and `String` types. Any column that cannot be parsed as a number becomes `String`. Always review and refine inferred schemas before use in production — in particular, columns that are actually categorical (but represented as small integers) will be inferred as numeric.

For CSV files that have a header row and at least one data row, use `InferredSchema`:

```java
import org.datavec.api.transform.schema.InferredSchema;

// Reads header line for column names, first data line to infer types
Schema schema = new InferredSchema("/path/to/data.csv").build();
```

## Joining Two Schemas

When you have two datasets with a common key column, use `Join` to combine them:

```java
import org.datavec.api.transform.join.Join;

Schema customerSchema = new Schema.Builder()
    .addColumnLong("customerID")
    .addColumnString("customerName")
    .addColumnCategorical("country", Arrays.asList("USA", "GBR", "DEU"))
    .build();

Schema purchaseSchema = new Schema.Builder()
    .addColumnLong("customerID")
    .addColumnTime("purchaseTime", DateTimeZone.UTC)
    .addColumnDouble("amountUSD")
    .build();

Join join = new Join.Builder(Join.JoinType.Inner)
    .setJoinColumns("customerID")
    .setSchemas(customerSchema, purchaseSchema)
    .build();
```

Join types:

- `Inner` — only records with matching keys in both datasets
- `LeftOuter` — all records from the left dataset; right side fills `NullWritable` for unmatched rows
- `RightOuter` — all records from the right dataset; left side fills `NullWritable` for unmatched rows
- `FullOuter` — all records from both datasets; unmatched sides fill `NullWritable`

After building the `Join`, execute it with `LocalTransformExecutor.executeJoin` or `SparkTransformExecutor.executeJoin`. See [Executors](executors.md).

## Common Mistakes

**Column order mismatch**: The schema column order must exactly match the order that your `RecordReader` produces values. If your CSV has columns in a different order than your schema, transforms will silently operate on the wrong columns.

**Declaring categorical columns for numeric data**: If a column contains the integers 0, 1, 2, 3 and you want to use it as a label, declare it as `Integer` (not `Categorical`) and use `categoricalToInteger` transform only when the raw column actually contains strings.

**Forgetting timezone on Time columns**: Time columns always require a timezone. Use `DateTimeZone.UTC` as the default unless your data is explicitly in a local timezone.

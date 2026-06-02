# 🔥 PySpark Certification Practice Test — Part 2
### Intermediate to Advanced | Logic & Concept Focused

> **Instructions:** Read each scenario carefully and select the best answer before expanding the hidden answer block. Each explanation covers *why* the correct answer is right and *why* the other options are wrong. Topics cover DataFrame API, RDDs, window functions, UDFs, streaming, Delta Lake, and production tuning.

---

## Section 6: DataFrame API — Joins, Aggregations & Column Operations

---

**Q21.** A data engineer runs the following join and gets **duplicate rows** in the output:

```python
result = orders_df.join(customers_df, on="customer_id", how="inner")
result.show()
# Expected: 1 row per order
# Actual: many rows per order
```

What is the most likely cause?

- A) `inner` join is the wrong join type — use `left` join to avoid duplicates.
- B) The `customers_df` has **multiple rows per `customer_id`** (it is not deduplicated), so each order row fan-outs by matching multiple customer rows.
- C) PySpark's `.join()` automatically creates a cross join when both DataFrames share a column name.
- D) The `on="customer_id"` syntax is invalid — you must use `on=orders_df["customer_id"] == customers_df["customer_id"]`.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

A SQL join matches **every row on the left** with **every matching row on the right**. If `customers_df` has 3 rows for the same `customer_id` (e.g., due to SCD Type 2 history or a missing deduplication step), each order row will produce 3 output rows — one per matching customer record. This is the most common source of "row explosion" bugs in Spark pipelines. Always validate the cardinality of your join keys with `.groupBy("customer_id").count().filter(col("count") > 1)` before joining.

**Why the others are wrong:**
- **(A)** Switching to `left` join would *also* produce duplicates — the join type controls which unmatched rows are included, not whether duplicates from the right side are multiplied.
- **(C)** Spark does not auto-create cross joins on shared column names. Using `on="customer_id"` is the correct and idiomatic syntax for equi-joins.
- **(D)** The `on="column_name"` string syntax is perfectly valid for simple equi-joins in PySpark. The explicit column expression syntax is needed for non-equi joins or when column names differ.

</details>

---

**Q22.** You need to find the **top 3 products by revenue** within each **region**. Which PySpark approach is correct?

```python
# Option A
df.groupBy("region").agg(sum("revenue")).orderBy("revenue", ascending=False).limit(3)

# Option B
from pyspark.sql.window import Window
from pyspark.sql.functions import rank

w = Window.partitionBy("region").orderBy(col("revenue").desc())
df.withColumn("rnk", rank().over(w)).filter(col("rnk") <= 3)

# Option C
df.orderBy("revenue", ascending=False).groupBy("region").agg(first("product_id"))

# Option D
df.filter(col("revenue") > df.agg(avg("revenue")).collect()[0][0])
```

- A) Option A — `groupBy` + `orderBy` + `limit(3)` gives top 3 per region.
- B) Option C — sorting first and then grouping preserves the order within each group.
- C) Option D — filtering by above-average revenue approximates the top entries.
- D) Option B — a Window function with `rank()` partitioned by region correctly assigns ranks within each region and allows filtering to top 3.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: D**

**Window functions** are designed exactly for this use case: ranked aggregations *within groups* without collapsing the rows. `Window.partitionBy("region")` creates an independent ranking context per region, and `rank().over(w)` assigns a rank to each product within its region. Filtering `rnk <= 3` then returns the top 3 per region. This is the idiomatic and correct PySpark pattern for "top-N per group" queries.

**Why the others are wrong:**
- **(A)** Option A aggregates the *entire* dataset into one row per region (summing all revenues), then takes 3 regions globally. It does not give top-3 products *within each* region.
- **(C)** In distributed Spark, `orderBy` before `groupBy` does **not** guarantee order within groups. `first()` after a sort may not return the highest-revenue product due to distributed execution semantics. This is a common and dangerous misconception.
- **(D)** Option D filters rows above the global average — this is unrelated to "top 3 per region" and is both logically wrong and extremely inefficient (it calls `.collect()` to pull a scalar to the Driver).

</details>

---

**Q23.** What is the output of this PySpark code?

```python
from pyspark.sql.functions import coalesce, lit

data = [(1, None), (2, "hello"), (3, None)]
df = spark.createDataFrame(data, ["id", "name"])
df.withColumn("result", coalesce(col("name"), lit("unknown"))).show()
```

- A) All rows will have `result = "unknown"` because `coalesce` returns the first non-null argument, and `lit("unknown")` is always non-null.
- B) Rows with `None` in `name` will have `result = None` — `coalesce` does not handle Python `None`.
- C) Rows with `None` in `name` will have `result = "unknown"`; rows with a value will retain their original `name`.
- D) The code throws an error because `coalesce` requires all arguments to be the same column type.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

`coalesce(col1, col2, ...)` returns the **first non-null value** from its arguments, evaluated left to right. For rows where `name` is `None` (null), `coalesce` skips it and returns `lit("unknown")`. For rows where `name = "hello"`, it returns `"hello"` immediately. This is the PySpark equivalent of SQL's `COALESCE()` function and is the idiomatic way to handle null replacement without `when/otherwise` chains.

**Why the others are wrong:**
- **(A)** Not all rows return `"unknown"` — only those where `name` is null. `coalesce` returns the first *non-null* argument, so `"hello"` is returned for row 2.
- **(B)** PySpark maps Python `None` to SQL `NULL` when creating DataFrames, so `coalesce` absolutely handles it. `col("name")` being null in Spark SQL means `coalesce` will skip it.
- **(D)** `coalesce` works across compatible types. `name` is `StringType` and `lit("unknown")` is also a string — no type error occurs.

</details>

---

**Q24.** A data engineer wants to add a column that labels each row as `"high"`, `"medium"`, or `"low"` based on a `score` column. They write:

```python
df.withColumn("label",
    when(col("score") >= 80, "high")
    .when(col("score") >= 50, "medium")
    .otherwise("low")
)
```

A colleague says this is wrong and should use `if/elif` in a UDF instead. Who is correct, and why?

- A) The colleague is correct — `when/otherwise` does not support chaining for more than 2 conditions.
- B) The data engineer is correct — `when/otherwise` is a **native Catalyst expression** that runs directly in the JVM without Python serialization overhead, making it far more efficient than a UDF.
- C) The colleague is correct — UDFs are always more readable and Spark automatically optimizes them using Catalyst.
- D) Both are equally correct — PySpark UDFs are compiled to JVM bytecode and run at native speed.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

`when/otherwise` is a **Catalyst-native expression** — it is compiled and executed directly in the JVM's optimized execution engine with no Python overhead. A Python UDF, by contrast, requires Spark to **serialize each row from JVM → Python process → JVM**, paying serialization and context-switching costs on every row. For a simple conditional like this, `when/otherwise` can be **10–100x faster** than a UDF. Always prefer built-in Spark functions over UDFs when the operation can be expressed natively.

**Why the others are wrong:**
- **(A)** `when/otherwise` absolutely supports chaining for any number of conditions — each `.when()` call appends another branch, just like `if/elif/else`.
- **(C)** UDFs are *not* optimized by Catalyst. They are opaque black boxes to the optimizer — Catalyst cannot push predicates through them, cannot eliminate them, and cannot optimize their internals.
- **(D)** Python UDFs are **not** compiled to JVM bytecode. They run in a separate Python process. Pandas UDFs (via Apache Arrow) are faster but still not as fast as native Catalyst expressions.

</details>

---

## Section 7: Window Functions

---

**Q25.** What is the difference between `rank()`, `dense_rank()`, and `row_number()` window functions in PySpark? Given scores `[100, 100, 90, 80]`, what does each produce?

- A) All three produce `[1, 2, 3, 4]` — they are aliases.
- B) `rank()` → `[1, 1, 3, 4]`, `dense_rank()` → `[1, 1, 2, 3]`, `row_number()` → `[1, 2, 3, 4]`.
- C) `rank()` → `[1, 2, 3, 4]`, `dense_rank()` → `[1, 1, 2, 3]`, `row_number()` → `[1, 1, 3, 4]`.
- D) `rank()` → `[1, 1, 2, 3]`, `dense_rank()` → `[1, 1, 3, 4]`, `row_number()` → `[1, 2, 3, 4]`.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

These three functions differ only in how they handle **ties**:

- **`rank()`** assigns the same rank to ties but then **skips** the next rank(s). Two rows tied at 1st place both get rank 1, and the next row gets rank 3 (not 2) — like Olympic medals where two gold means no silver.
- **`dense_rank()`** assigns the same rank to ties but does **not skip**. Two rows tied at 1st both get 1, and the next distinct value gets 2.
- **`row_number()`** assigns a **unique sequential number** regardless of ties — the ordering between tied rows is arbitrary (non-deterministic) but each gets a distinct number.

Think of it as: `rank` = competition ranking (gaps exist), `dense_rank` = category ranking (no gaps), `row_number` = just numbering the rows.

**Why the others are wrong:**
- **(A)** They produce different results specifically when ties are present, which is their primary distinguishing behavior.
- **(C)** `row_number` produces 1,2,3,4 (never repeats), not 1,1,3,4.
- **(D)** The definitions of `rank` and `dense_rank` are swapped.

</details>

---

**Q26.** A data engineer writes this window query to compute a **7-day rolling average** of sales:

```python
w = Window.partitionBy("store_id") \
          .orderBy("sale_date") \
          .rowsBetween(-6, 0)

df.withColumn("rolling_avg", avg("sales").over(w))
```

A colleague suggests changing `.rowsBetween(-6, 0)` to `.rangeBetween(-6, 0)`. What is the **critical difference** between `rowsBetween` and `rangeBetween` in this context?

- A) They are identical — `rowsBetween` and `rangeBetween` produce the same result when ordered by date.
- B) `rowsBetween(-6, 0)` includes the **6 physically preceding rows** regardless of date gaps; `rangeBetween(-6, 0)` includes rows within a **value range of 6 units from the current row's order-by value** — critical if there are missing dates in the data.
- C) `rangeBetween` is always preferred — it automatically handles gaps in date sequences.
- D) `rowsBetween` includes future rows; `rangeBetween` is restricted to past rows only.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

This is a subtle but critical distinction in time-series analytics. **`rowsBetween(-6, 0)`** counts **physical row positions**: "include the 6 rows before the current row in the partition ordering." If there are missing days (e.g., store was closed on weekends), it still includes 6 rows regardless of how many days those rows span. **`rangeBetween(-6, 0)`** interprets offsets as **value ranges on the order-by column**: "include rows where `sale_date` is within 6 *value units* of the current row's date." For date columns, this means exactly 6 days back — but only if the column is numeric or a long timestamp. For the intended "7 calendar day" rolling average, you'd typically convert dates to Unix timestamps and use `rangeBetween`.

**Why the others are wrong:**
- **(A)** They produce the same result only when data is perfectly dense (no gaps) and each row represents exactly one unit. In practice, they diverge as soon as gaps appear.
- **(C)** `rangeBetween` doesn't automatically handle gaps better in all cases — it handles *calendar windows* better, but requires the order-by column to be numeric/long. It still depends on how gaps map to the numeric range.
- **(D)** Both can include past and future rows — you control direction with the start/end parameters. `Window.unboundedPreceding` and `Window.unboundedFollowing` extend to all rows.

</details>

---

**Q27.** A pipeline needs to compute the **cumulative sum** of `revenue` per `customer_id`, ordered by `transaction_date`. Which code is correct?

```python
# Option A
w = Window.partitionBy("customer_id").orderBy("transaction_date")
df.withColumn("cumsum", sum("revenue").over(w))

# Option B
df.groupBy("customer_id").agg(sum("revenue").alias("cumsum"))

# Option C
w = Window.partitionBy("customer_id")
df.withColumn("cumsum", sum("revenue").over(w))

# Option D
df.orderBy("transaction_date").withColumn("cumsum",
    sum("revenue").over(Window.partitionBy("customer_id")))
```

- A) Option B — `groupBy` with `sum` computes cumulative totals per customer.
- B) Option C — partitioning by `customer_id` without `orderBy` gives the cumulative sum.
- C) Option A — a Window with `partitionBy` + `orderBy` creates a running sum frame by default, giving cumulative sum up to each row's date.
- D) Option D — sorting the DataFrame before applying the window is the correct approach.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

When you define a Window with **`partitionBy` + `orderBy`** and apply `sum()`, Spark uses a **default frame** of `rowsBetween(Window.unboundedPreceding, Window.currentRow)`. This means: sum all rows from the beginning of the partition up to and including the current row, sorted by date. This is exactly the definition of a cumulative (running) sum.

**Why the others are wrong:**
- **(A)** `groupBy` with `sum` collapses all rows per customer into a **single total** — it destroys the individual transaction rows and produces one aggregated row per customer, not a running sum per row.
- **(C)** Option C uses a Window with `partitionBy` but **no `orderBy`**. Without an ordering, the entire partition is the frame for every row, which gives the **total sum** repeated on every row — not a cumulative sum. Order is essential for meaningful running aggregations.
- **(D)** `orderBy` on the DataFrame level does not affect Window function frames. Window functions always use their own internal ordering defined by `Window.orderBy()`, not the DataFrame's sort order.

</details>

---

## Section 8: User-Defined Functions (UDFs) & Pandas UDFs

---

**Q28.** A data engineer registers a Python UDF to parse a JSON string column:

```python
import json
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

def extract_city(json_str):
    return json.loads(json_str).get("city", "unknown")

city_udf = udf(extract_city, StringType())
df.withColumn("city", city_udf(col("address_json")))
```

A senior engineer flags this as a performance anti-pattern. What is the **correct, performant alternative**?

- A) Rewrite the UDF using `@pandas_udf` with `PandasUDFType.SCALAR`.
- B) Use PySpark's native `get_json_object(col("address_json"), "$.city")` function — it's a Catalyst-native expression that runs in the JVM without Python serialization.
- C) Register the UDF as a SQL function using `spark.udf.register()` to let Catalyst optimize it.
- D) Use `df.rdd.map()` to parse the JSON in Python and reconstruct the DataFrame.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

PySpark has a built-in `get_json_object(column, "$.path")` function (and `from_json` for full parsing) that runs entirely in the **JVM via Catalyst** — no Python serialization overhead, no row-by-row Python invocation. This is orders of magnitude faster than a Python UDF for JSON parsing. Always check the `pyspark.sql.functions` module for a native function before reaching for a UDF.

**Why the others are wrong:**
- **(A)** A Pandas UDF (Arrow-based) is significantly faster than a row-at-a-time Python UDF, but it still requires Python-JVM data transfer via Apache Arrow. For JSON parsing, it's still slower than `get_json_object()` which never leaves the JVM.
- **(C)** `spark.udf.register()` makes the UDF available in Spark SQL syntax, but does *nothing* to change its execution model. Catalyst still cannot optimize inside a registered Python UDF — it remains an opaque black box.
- **(D)** Converting to RDD, operating in Python, and converting back is even more expensive than a UDF — you lose DataFrame optimizations entirely, incur full serialization overhead, and break the Catalyst pipeline.

</details>

---

**Q29.** When is a **Pandas UDF (`@pandas_udf`)** the right choice over a standard Python UDF?

- A) When the function is simple (e.g., string length) — Pandas UDFs are always faster.
- B) When you need to apply NumPy, SciPy, or ML inference operations that require vectorized computation and cannot be expressed with native PySpark functions.
- C) When you need Catalyst to optimize the function — Pandas UDFs enable full Catalyst optimization.
- D) When running on a local cluster — Pandas UDFs are only optimized for local mode.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

**Pandas UDFs** (also called vectorized UDFs) use **Apache Arrow** to transfer entire column batches between JVM and Python in one shot, avoiding the per-row serialization overhead of standard UDFs. They're the right tool when: (1) you need Python/NumPy/SciPy operations not available as native Spark functions, (2) you're running ML model inference on batches of rows, or (3) you need operations on column arrays (e.g., signal processing). For simple string/math operations expressible natively, built-in Spark functions are still faster.

**Why the others are wrong:**
- **(A)** For simple operations like string length, native `length(col("str"))` is faster than any UDF — it never leaves the JVM. Pandas UDFs add Arrow serialization overhead that only pays off for complex, vectorized operations.
- **(C)** Catalyst cannot optimize inside Pandas UDFs either. They are still opaque to the optimizer — Catalyst just treats them as a function call boundary. The speedup comes from Arrow batching, not Catalyst optimization.
- **(D)** Pandas UDFs perform well on real clusters — that's exactly where their batch Arrow transfer provides the most benefit over row-at-a-time Python UDFs.

</details>

---

## Section 9: RDDs vs DataFrames

---

**Q30.** A data engineer inherits a codebase that uses RDDs extensively. Their tech lead asks them to migrate to DataFrames. Which of the following is the **strongest technical reason** to prefer DataFrames over RDDs?

- A) DataFrames can only be used with structured data; RDDs are required for semi-structured data.
- B) RDDs support Python, Scala, and Java, while DataFrames only support Scala.
- C) DataFrames leverage the **Catalyst optimizer and Tungsten execution engine**, giving automatic query optimization, code generation, and memory management — often 5–10x faster than hand-written RDD transformations.
- D) DataFrames are fault-tolerant; RDDs are not.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

The **Catalyst optimizer** analyzes your DataFrame query plan and rewrites it to be more efficient (predicate pushdown, constant folding, join reordering). The **Tungsten engine** then executes the optimized plan using off-heap memory management and whole-stage code generation — compiling query plans directly to JVM bytecode. RDD operations bypass both: every RDD transformation you write is executed exactly as written, with no optimization. Even a poorly-written DataFrame query often outperforms a carefully hand-crafted RDD pipeline.

**Why the others are wrong:**
- **(A)** DataFrames handle semi-structured data excellently — Spark has native support for JSON, Parquet, Avro, and complex nested types. You can also use `from_json`, `explode`, and struct functions.
- **(B)** Both RDDs and DataFrames are available in Python (PySpark), Scala, Java, and R. This is not a differentiating factor.
- **(D)** Both RDDs and DataFrames are fault-tolerant — they maintain lineage graphs (DAGs) that allow recomputation of lost partitions. Fault tolerance is equally fundamental to both abstractions.

</details>

---

**Q31.** A data engineer uses `rdd.map()` to apply a transformation. A colleague points out this creates a **Python UDF-equivalent overhead** for DataFrames. Which of the following RDD operations is a **narrow transformation**?

- A) `rdd.groupByKey()`
- B) `rdd.reduceByKey(lambda a, b: a + b)`
- C) `rdd.join(other_rdd)`
- D) `rdd.map(lambda x: (x[0], x[1] * 2))`

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: D**

`rdd.map()` applies a function to **each element independently** within its partition. No data needs to move between partitions — each element is transformed in place. This makes it a **narrow transformation**: each output partition depends only on one input partition, with no network shuffle.

**Why the others are wrong:**
- **(A)** `groupByKey()` is a **wide transformation** — it shuffles all values with the same key to the same partition across the network. It is also famously inefficient because it transfers all values (not just aggregated results) over the network before reducing.
- **(B)** `reduceByKey()` is a **wide transformation** — it requires a shuffle by key to co-locate all values for each key. (It is more efficient than `groupByKey` because it performs a map-side partial aggregation first, but it still shuffles.)
- **(C)** `rdd.join()` is a **wide transformation** — it requires both RDDs to be shuffled so matching keys land on the same partition, similar to a SQL join.

</details>

---

## Section 10: Reading & Writing Data — Formats, Partitioning & Schema

---

**Q32.** A data engineer writes a DataFrame to Parquet partitioned by `year` and `month`:

```python
df.write.partitionBy("year", "month").parquet("s3://datalake/events/")
```

A downstream job then reads this data and filters by `year=2024` and `month=3`. What happens at the storage layer?

- A) Spark reads all Parquet files and filters rows after loading them into memory.
- B) Spark reads only the files inside the `year=2024/month=3/` directory — this is **Partition Pruning**, and only matching directory partitions are scanned.
- C) Spark reads the `year=2024/` directory but still scans all month subdirectories within it.
- D) `partitionBy` only affects how the data is organized on disk; filtering behavior is unchanged.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

When you write with `partitionBy("year", "month")`, Spark creates a **Hive-style directory hierarchy**: `year=2024/month=3/part-*.parquet`. The Catalyst optimizer performs **partition pruning**: when a filter matches the partition columns, Spark lists only the matching directory and reads only those files. For a dataset with 3 years × 12 months = 36 partitions, this means reading only 1/36th of the data — a massive I/O reduction.

**Why the others are wrong:**
- **(A)** This is what would happen if the data were *not* partitioned by those columns. With Hive-style partitioning, Spark leverages the directory structure to skip irrelevant partitions entirely.
- **(C)** Both partition columns are evaluated together. With a filter on both `year=2024` and `month=3`, Spark navigates directly to `year=2024/month=3/` — it does not scan all months within 2024.
- **(D)** Partition pruning is one of the most impactful data layout optimizations in Spark. The write-time `partitionBy` directly enables read-time partition pruning.

</details>

---

**Q33.** A data pipeline needs to read a CSV file but the schema keeps changing over time (new columns are added). A junior engineer uses:

```python
df = spark.read.option("inferSchema", "true").csv("s3://data/file.csv")
```

Why is `inferSchema=True` a **production anti-pattern**, and what is the correct approach?

- A) `inferSchema=True` is always correct and is recommended for production pipelines.
- B) `inferSchema=True` causes Spark to **read the entire file twice** (once to infer types, once to load data), and may infer incorrect types for edge cases (e.g., a column with all nulls becomes `StringType`). Define an explicit schema using `StructType`.
- C) `inferSchema=True` only works for local files — it fails on S3.
- D) `inferSchema=True` is fine for small files but causes memory errors on large files.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

`inferSchema=True` is problematic in production for three reasons: (1) it requires a **full pass over the data** to sample types — for large files this means extra I/O before any processing begins; (2) it can **infer wrong types** — a column with only integer values today might get `LongType`, but tomorrow's file with a decimal breaks the pipeline; (3) it ties your schema to the actual data, making your pipeline **brittle to upstream changes**. The correct approach is to define an explicit `StructType` schema and use `spark.read.schema(my_schema).csv(...)` — your pipeline documents its contract and fails fast if the data doesn't match.

**Why the others are wrong:**
- **(A)** `inferSchema=True` is a convenience for exploration, not production. It lacks the stability and performance guarantees needed in production.
- **(C)** `inferSchema` works on any supported file system including S3, ADLS, and GCS. File system location is irrelevant to schema inference.
- **(D)** The double-read problem occurs regardless of file size. Memory is not the issue — extra I/O and type instability are.

</details>

---

**Q34.** A team writes a Spark job that produces **thousands of small Parquet files** (each ~1MB). Downstream tools that read this output are very slow. What caused this, and how do you fix it?

- A) Parquet is the wrong format for large outputs — switch to CSV.
- B) The job likely has **too many shuffle partitions or output partitions**, causing one small file per partition. Fix: call `.coalesce(N)` or `.repartition(N)` before writing to reduce the output partition count, or enable Spark's **Adaptive Query Execution (AQE)** with `spark.sql.adaptive.enabled=true` which can coalesce small shuffle partitions automatically.
- C) The output files are small because the input data was small — there is no fix needed.
- D) Use `.write.mode("overwrite")` instead of `"append"` — overwrite mode compacts files automatically.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

The **small files problem** is a classic Spark issue. Each output partition becomes one file. If `spark.sql.shuffle.partitions=200` (default) and your output is only 2GB of data, you get 200 files of ~10MB each — or worse, if the pre-shuffle data is small, files can be 1MB or smaller. Solutions: (1) `.coalesce(N)` merges existing partitions without a full shuffle — good for reducing without reshuffling; (2) `.repartition(N)` does a full shuffle to exactly N partitions — use when you also want to repartition by a key; (3) **AQE's coalesce** (`spark.sql.adaptive.coalescePartitions.enabled=true`) automatically detects small partitions and merges them after shuffles.

**Why the others are wrong:**
- **(A)** Parquet is actually better than CSV for downstream tools precisely because it's columnar and splittable. The format isn't the issue — the file count is.
- **(C)** The number of output files is determined by Spark's partition count, not the input data size. You can have 200 files from 100MB of data if partitions aren't controlled.
- **(D)** Write mode (`overwrite` vs `append`) controls whether existing data is replaced — it has zero effect on file size or count.

</details>

---

## Section 11: Structured Streaming

---

**Q35.** A data engineer is building a real-time pipeline that reads from Kafka and writes aggregated results to a database every 30 seconds. They configure:

```python
stream_df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "events") \
    .load()

query = stream_df \
    .groupBy(window(col("timestamp"), "5 minutes"), col("event_type")) \
    .count() \
    .writeStream \
    .trigger(processingTime="30 seconds") \
    .outputMode("complete") \
    .format("console") \
    .start()
```

What does `outputMode("complete")` mean, and when would you choose `"update"` instead?

- A) `"complete"` writes only new rows since the last trigger; `"update"` writes the full result table every trigger.
- B) `"complete"` outputs the **entire result table** on every trigger (all groups rewritten); `"update"` outputs **only the rows that changed** since the last trigger — better for large result tables where most rows don't change each batch.
- C) Both modes are identical for windowed aggregations.
- D) `"complete"` is for batch jobs; `"update"` is for streaming jobs.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

Structured Streaming has three output modes. **`complete`** rewrites the entire aggregated result table on every trigger — useful for small result sets (e.g., global counts) where you want the full picture every time. **`update`** only emits rows whose values changed since the previous trigger — much more efficient for large tables where only a fraction of groups receive new events each batch (e.g., a table with 10,000 product groups where only 50 received events in the last 30 seconds). **`append`** only works for queries where rows, once output, never change (no aggregations, or append-only windowing with watermarks).

**Why the others are wrong:**
- **(A)** This has the definitions reversed. `complete` = full table; `update` = only changed rows.
- **(C)** They are meaningfully different. For a windowed aggregation over Kafka data, `complete` could write thousands of window/event_type combinations every 30 seconds, while `update` writes only those windows that received new events.
- **(D)** Both modes are streaming-specific concepts. Batch jobs use static reads and writes, not streaming output modes.

</details>

---

**Q36.** In Structured Streaming, what is a **watermark** and why is it important?

```python
stream_df \
    .withWatermark("event_time", "10 minutes") \
    .groupBy(window(col("event_time"), "5 minutes")) \
    .count()
```

- A) A watermark is a checkpoint that saves the stream state to disk every N minutes.
- B) A watermark tells Spark to **ignore events older than a threshold** relative to the maximum observed event time, allowing Spark to safely drop late data and **clean up state** for old windows — preventing unbounded state growth.
- C) A watermark sets the trigger interval for how often the stream processes data.
- D) A watermark defines the maximum number of rows Spark buffers in memory before writing to the sink.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

In event-time streaming, events arrive **out of order** — a mobile app might buffer events for minutes before sending them. Without a watermark, Spark must keep state for every open window forever (since a late event could theoretically arrive at any time). A **watermark** says: "I'm willing to wait up to 10 minutes for late data. Any event older than `max_observed_event_time - 10 minutes` will be dropped." This gives Spark permission to **close and evict old window state**, preventing unbounded memory growth. Without watermarks, long-running streaming jobs eventually OOM from accumulated window state.

**Why the others are wrong:**
- **(A)** Checkpointing is a separate mechanism (`checkpointLocation`) that saves progress and state to a fault-tolerant store. Watermarks are about late-data handling, not checkpointing.
- **(C)** Trigger intervals are set with `.trigger(processingTime="X seconds")` — completely separate from watermarks.
- **(D)** Row buffering limits are not a Spark concept in this context. Watermarks control state *eviction*, not ingest buffering.

</details>

---

## Section 12: Delta Lake & Production Patterns

---

**Q37.** A production Delta Lake table is queried by analysts 50+ times per day. A senior engineer recommends running `OPTIMIZE` on the table regularly. What does `OPTIMIZE` do?

- A) It rebuilds the table's schema and fixes any data type mismatches.
- B) It **compacts small files** in the Delta table into larger, more efficient files and rebuilds the data skipping statistics — reducing the number of files analysts' queries must scan and improving read performance.
- C) It re-partitions the table by a new column without rewriting the data.
- D) It clears the Delta transaction log and resets the table to its initial state.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

Delta Lake tables that receive many small writes (streaming micro-batches, frequent appends) accumulate **thousands of small files** over time. `OPTIMIZE` performs two jobs: (1) **bin-packs small files** into larger target-size files (default: 1GB), reducing the file count that each query must open and scan; (2) **updates data skipping statistics** (min/max values per column per file), so queries with filters can skip entire files without reading them. This is especially important for tables with high write frequency. `OPTIMIZE ZORDER BY (column)` additionally co-locates related data for common filter patterns.

**Why the others are wrong:**
- **(A)** `OPTIMIZE` does not touch schema — that's done via `ALTER TABLE` or `mergeSchema` options.
- **(C)** Changing partition columns requires rewriting the table. `OPTIMIZE` reorganizes *within* the existing partition structure.
- **(D)** Clearing the transaction log is `VACUUM` (which removes old data files no longer referenced). The transaction log itself is never cleared — it's the audit trail that enables time travel.

</details>

---

**Q38.** What is the difference between `VACUUM` and `OPTIMIZE` in Delta Lake?

- A) `VACUUM` runs on the Driver; `OPTIMIZE` runs on Executors.
- B) `VACUUM` **removes old data files** no longer referenced by the Delta transaction log (enabling storage reclaim after deletes/updates), subject to a retention period; `OPTIMIZE` **compacts live files** to improve read performance — both operate on the same table but serve different purposes.
- C) They are the same operation — `VACUUM` is the SQL alias for `OPTIMIZE`.
- D) `OPTIMIZE` is for partitioned tables; `VACUUM` is for unpartitioned tables.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

Delta Lake's ACID guarantees mean that `UPDATE`, `DELETE`, and `MERGE` operations don't physically delete old files — they write new versions and record the change in the transaction log (enabling time travel). This means old file versions accumulate on disk. **`VACUUM`** physically deletes data files that are older than the retention period (default: 7 days) and no longer referenced by any active table version — reclaiming storage. **`OPTIMIZE`** operates only on currently live files, compacting them for better read performance. Think: VACUUM = garbage collection; OPTIMIZE = defragmentation.

**Why the others are wrong:**
- **(A)** Both operations distribute work across Executors. Driver/Executor split is not the distinguishing factor.
- **(C)** They are completely different operations with different purposes and different effects on the table.
- **(D)** Both work on partitioned and unpartitioned tables. `OPTIMIZE` can target specific partitions with a `WHERE` clause, but both are table-wide by default.

</details>

---

**Q39.** A data engineer needs to **upsert** (insert new rows, update existing rows) into a Delta table based on a matching key. Which PySpark code pattern is correct?

```python
from delta.tables import DeltaTable

# Option A
delta_table = DeltaTable.forPath(spark, "s3://datalake/customers/")
delta_table.merge(
    source=new_df,
    condition="target.customer_id = source.customer_id"
).whenMatchedUpdateAll() \
 .whenNotMatchedInsertAll() \
 .execute()

# Option B
new_df.write.mode("append").format("delta").save("s3://datalake/customers/")

# Option C
new_df.write.mode("overwrite").format("delta").save("s3://datalake/customers/")

# Option D
delta_table.update(condition="...", set={"col": "value"})
```

- A) Option B — `append` mode adds new rows and automatically updates existing ones.
- B) Option C — `overwrite` mode replaces old data with new data, effectively upserting.
- C) Option A — `DeltaTable.merge()` with `whenMatchedUpdateAll` and `whenNotMatchedInsertAll` is the correct ACID-compliant upsert pattern.
- D) Option D — `delta_table.update()` handles both inserts and updates.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

`DeltaTable.merge()` is Delta Lake's **MERGE / UPSERT** operation — the exact equivalent of SQL's `MERGE INTO`. It matches source rows to target rows using the condition, then: (1) **updates** matched rows with source data (`whenMatchedUpdateAll`), and (2) **inserts** unmatched rows from the source (`whenNotMatchedInsertAll`). All of this happens atomically (ACID transaction), so partial failures don't corrupt the table.

**Why the others are wrong:**
- **(A)** `append` mode adds all rows from `new_df` as new rows — it does not update existing rows with matching keys. This would create duplicate records.
- **(C)** `overwrite` mode replaces the **entire table** (or partition) with `new_df`. If `new_df` is only a delta of new/updated records, you'd destroy all existing records not in `new_df`.
- **(D)** `delta_table.update()` only updates rows that match a condition — it does not insert new rows. It's for targeted updates to existing data, not upserts.

</details>

---

## Section 13: Performance Tuning — AQE, Speculation & Memory

---

**Q40.** A Spark job runs well in testing (10GB) but slows dramatically in production (10TB). The Spark UI shows the production job has **200 shuffle partitions of ~500MB each** — far too large per task. The team doesn't want to hardcode a partition count. What is the best solution?

- A) Set `spark.sql.shuffle.partitions=2000` manually in the job configuration.
- B) Enable **Adaptive Query Execution (AQE)** with `spark.sql.adaptive.enabled=true` — AQE dynamically adjusts the number of post-shuffle partitions at runtime based on actual data sizes, splitting large partitions and coalescing small ones without manual tuning.
- C) Add more Executors — more parallelism will handle the large partitions.
- D) Repartition the input data to 2000 partitions before the shuffle.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

**Adaptive Query Execution (AQE)**, introduced in Spark 3.0, solves the `spark.sql.shuffle.partitions` tuning problem automatically. AQE collects **runtime statistics** after each shuffle stage and dynamically adjusts the downstream partition count: it coalesces many small partitions (solving the tiny-task anti-pattern from Q11) and can also split large partitions (solving the scenario in this question). It also enables dynamic join strategy switching (converting SMJ to BHJ when runtime data is found to be small enough). This makes Spark self-tuning within a job.

**Why the others are wrong:**
- **(A)** Hardcoding `shuffle.partitions=2000` is a static fix that works today but may be wrong next month when data volume changes. It's also wrong for the test environment (2000 partitions of ~5MB each on 10GB data).
- **(C)** Adding more Executors increases parallelism but doesn't fix partition sizes — you'd still have 200 tasks of 500MB each, just scheduled faster. The root problem is partition size, not CPU availability.
- **(D)** Repartitioning the input changes pre-shuffle partition count, but the shuffle (e.g., `groupBy`) will still create `spark.sql.shuffle.partitions` post-shuffle partitions — the config is the real control point.

</details>

---

**Q41.** A PySpark application intermittently fails with `GC overhead limit exceeded` on Executors. What is the **most likely cause** and the **recommended fix**?

- A) The Driver has too little memory — increase `spark.driver.memory`.
- B) The Executors are spending too much time in **Garbage Collection**, typically caused by storing too many objects in JVM heap (e.g., large datasets cached as `MEMORY_ONLY` with deserialized Java objects). Fix: switch to `MEMORY_AND_DISK_SER` or tune Executor memory with `spark.executor.memoryFraction`.
- C) The cluster has too many Executors — reduce the Executor count to lower GC pressure.
- D) The GC error is caused by the Cluster Manager, not Spark — restart the YARN ResourceManager.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

`GC overhead limit exceeded` means the JVM is spending **>98% of time garbage collecting** and recovering less than 2% of heap per GC cycle — effectively thrashing. This is a symptom of **heap memory pressure** in Executor JVMs. Common causes: (1) `MEMORY_ONLY` caching with large deserialized objects; (2) overly large Executor memory allocated per task; (3) too many shuffle buffers in memory simultaneously. Fixes: use `MEMORY_AND_DISK_SER` (serialized storage uses less heap), increase Executor memory, reduce `spark.sql.shuffle.partitions` (fewer concurrent shuffle buffers), or tune `spark.memory.fraction`.

**Why the others are wrong:**
- **(A)** `GC overhead limit exceeded` specifically refers to the JVM process's heap — in this context, it's the **Executor** JVM (on Worker nodes), not the Driver. The Driver has its own heap separately.
- **(C)** Fewer Executors means each Executor processes more data — this would likely *worsen* memory pressure per Executor, not help it.
- **(D)** The YARN ResourceManager runs in its own JVM with a modest memory footprint — it doesn't process data and is not susceptible to GC pressure from data operations.

</details>

---

**Q42.** Your Spark job always has **2-3 tasks that run 3x longer than all others** in the final stage. Speculative execution is disabled. A colleague enables it:

```
spark.speculation=true
```

What does speculative execution do, and what is the risk of enabling it?

- A) Speculative execution preloads data for future tasks, reducing startup latency. The risk is higher memory usage.
- B) Speculative execution **launches duplicate copies** of slow-running tasks on other Executors and uses whichever finishes first, discarding the slower copy. The risk: if the slowness is caused by **data skew** (not hardware), the duplicate tasks will also be slow — and you double resource consumption without fixing the root cause.
- C) Speculative execution moves slow tasks to faster Executors automatically. There is no performance risk.
- D) Speculative execution retries failed tasks on different Executors. It has no effect on slow but successful tasks.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

Spark's **speculative execution** monitors running tasks and, if a task is running significantly slower than the median for that stage, launches a **speculative duplicate** of that task on another Executor. Whichever copy finishes first "wins" — the other is killed. This is effective against **straggler tasks caused by bad hardware** (a degraded disk, a noisy neighbor on the node). However, if the slowness is caused by **data skew** (the task has genuinely more data to process), the speculative copy will be equally slow — and you've doubled the compute cost. Fixing skew (salting, better partitioning) is the proper solution for skew-related stragglers.

**Why the others are wrong:**
- **(A)** Speculative execution has nothing to do with data preloading or prefetching — that's a different concept related to read-ahead optimization.
- **(C)** Spark does not dynamically move tasks between Executors mid-execution. A launched task runs on its assigned Executor. Speculative execution *adds a new task*, it doesn't migrate one.
- **(D)** Retry logic handles *failed* tasks (after an error/exception). Speculative execution is specifically for *slow but still running* tasks. These are different mechanisms.

</details>

---

## 📊 Score Tracker — Part 2

| Section | Questions | Your Score |
|---|---|---|
| DataFrame API | Q21 – Q24 | &nbsp;&nbsp;&nbsp;/ 4 |
| Window Functions | Q25 – Q27 | &nbsp;&nbsp;&nbsp;/ 3 |
| UDFs & Pandas UDFs | Q28 – Q29 | &nbsp;&nbsp;&nbsp;/ 2 |
| RDDs vs DataFrames | Q30 – Q31 | &nbsp;&nbsp;&nbsp;/ 2 |
| Reading & Writing Data | Q32 – Q34 | &nbsp;&nbsp;&nbsp;/ 3 |
| Structured Streaming | Q35 – Q36 | &nbsp;&nbsp;&nbsp;/ 2 |
| Delta Lake | Q37 – Q39 | &nbsp;&nbsp;&nbsp;/ 3 |
| Performance Tuning | Q40 – Q42 | &nbsp;&nbsp;&nbsp;/ 3 |
| **Total** | **Q21 – Q42** | **&nbsp;&nbsp;&nbsp;/ 22** |

---

## 🧠 Key Concepts Cheat Sheet — Part 2

| Concept | One-Line Summary |
|---|---|
| **Join Row Explosion** | Always verify right-side key cardinality before joining |
| **`when/otherwise`** | Catalyst-native conditional — always prefer over UDF for logic |
| **`coalesce(col, lit(...))`** | Null-safe substitution; returns first non-null from left to right |
| **`rank()` vs `dense_rank()` vs `row_number()`** | rank skips; dense_rank doesn't skip; row_number always unique |
| **`rowsBetween` vs `rangeBetween`** | rowsBetween = physical rows; rangeBetween = value offset from key |
| **Python UDF** | Black box to Catalyst; per-row JVM↔Python serialization — avoid |
| **Pandas UDF** | Vectorized via Arrow; good for NumPy/ML inference batches |
| **`get_json_object()`** | Native JVM JSON parser — faster than any Python UDF |
| **Catalyst + Tungsten** | The "why DataFrames beat RDDs" answer — optimizer + code gen |
| **`partitionBy` on write** | Enables partition pruning on reads — huge I/O savings |
| **`inferSchema=True`** | Double-read + type instability — always use explicit `StructType` in prod |
| **Small Files Problem** | Too many shuffle partitions → too many small output files → slow reads |
| **AQE** | Spark 3.0+ self-tuning: auto-coalesces partitions, switches join strategies |
| **Watermark** | Max late-data tolerance; allows Spark to evict old streaming state |
| **Delta `MERGE`** | ACID upsert: update matched rows, insert unmatched rows atomically |
| **Delta `OPTIMIZE`** | Compacts small files + refreshes data skipping stats |
| **Delta `VACUUM`** | Physically deletes old file versions — reclaims storage after updates/deletes |
| **Speculative Execution** | Duplicates slow tasks; fixes hardware stragglers, not data skew |
| **GC Overhead** | JVM heap pressure — use serialized storage levels or tune memory config |

---

*Combined with Part 1, you now have 42 scenario-based questions covering the full PySpark certification syllabus. Good luck! 🚀*
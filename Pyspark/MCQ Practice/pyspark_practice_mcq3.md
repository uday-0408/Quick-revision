# 🔥 PySpark Certification Practice Test — Part 3
### Advanced | Logic & Concept Focused

> **Instructions:** Read each scenario carefully and select the best answer before expanding the hidden answer block. Each explanation covers *why* the correct answer is right and *why* the other options are wrong. Topics cover broadcast joins, caching, skew handling, schema evolution, Catalyst internals, checkpointing, partitioning strategies, and more.

---

## Section 14: Joins — Strategies, Skew & Broadcast

---

**Q43.** A Spark job joins a **500GB fact table** with a **2MB lookup table** (country codes). The job is slow due to an expensive sort-merge join. What is the correct fix?

```python
# Current code
result = fact_df.join(lookup_df, on="country_code", how="left")
```

- A) Increase `spark.sql.shuffle.partitions` to reduce partition size during the join.
- B) Use a `right` join instead of `left` — Spark optimizes right joins differently.
- C) Wrap the small table in `broadcast()` to force a **Broadcast Hash Join (BHJ)**, which sends the small table to every Executor and eliminates the shuffle entirely.
- D) Repartition both DataFrames to the same number of partitions before joining.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

```python
from pyspark.sql.functions import broadcast
result = fact_df.join(broadcast(lookup_df), on="country_code", how="left")
```

A **Broadcast Hash Join** avoids the shuffle entirely. Spark sends (broadcasts) the small table to every Executor's memory. Each Executor then performs the join locally by probing the in-memory hash table — no data movement across the network. For a 2MB table, this is trivially small. By contrast, a Sort-Merge Join (SMJ) shuffles *both* tables by the join key, causing massive network I/O on the 500GB fact table. BHJ is the single biggest join optimization available in Spark. Spark may auto-broadcast if the table is under `spark.sql.autoBroadcastJoinThreshold` (default 10MB), but explicit `broadcast()` is more reliable.

**Why the others are wrong:**
- **(A)** More shuffle partitions makes the shuffle cheaper per partition, but you still pay the full shuffle cost on 500GB. It doesn't eliminate the shuffle.
- **(B)** Join direction (`left` vs `right`) does not change the join strategy selection. Spark chooses join strategy based on data size estimates, not join direction.
- **(D)** Pre-repartitioning to matching partition counts can slightly reduce the SMJ shuffle work, but you're still paying the shuffle cost. It doesn't come close to eliminating it like BHJ does.

</details>

---

**Q44.** A Spark join stage has 200 tasks, but **3 tasks always run for 20 minutes** while all others finish in 30 seconds. The Spark UI shows those 3 tasks processing **50x more data** than the rest. What is the root cause and the fix?

- A) The slow tasks are on bad nodes — enable speculative execution to relaunch them.
- B) The join key has **data skew** — a small number of key values (e.g., `customer_id = NULL` or a single viral product) have disproportionately many rows, landing all on the same partition. Fix with **salting**: add a random suffix to the key on both sides to spread load across multiple partitions.
- C) The Executors on those nodes have less memory — add more memory to the cluster.
- D) The slow tasks are reading from a slow storage node — use a different input format.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

**Data skew** is one of the most common production Spark performance problems. When a few join key values have vastly more rows than others (e.g., `NULL` keys, a single top customer, a popular product), all rows for that key land on the same partition — one task processes gigabytes while others process megabytes. The canonical fix is **salting**:

```python
import pyspark.sql.functions as F

# Add a random salt suffix (0–9) to the skewed key
fact_df = fact_df.withColumn("salted_key",
    F.concat(col("skewed_key"), F.lit("_"), (F.rand() * 10).cast("int")))

# Explode the small table to match all salt values
lookup_df = lookup_df.withColumn("salt", F.explode(F.array([F.lit(i) for i in range(10)])))
lookup_df = lookup_df.withColumn("salted_key",
    F.concat(col("skewed_key"), F.lit("_"), col("salt")))

result = fact_df.join(lookup_df, on="salted_key")
```

This distributes the hot key across 10 partitions instead of 1.

**Why the others are wrong:**
- **(A)** Speculative execution launches a duplicate of the slow task — but that duplicate gets the *same skewed data* and will be equally slow. Speculative execution fixes hardware stragglers, not data skew.
- **(C)** Memory amount doesn't cause 50x data imbalance. The slow tasks have more data to process, not less memory to work with. Even 10x more memory wouldn't help if the task has 50x the data.
- **(D)** The slow tasks are processing Spark partition data already in memory/shuffle storage — it's not a storage I/O problem at this stage.

</details>

---

**Q45.** A Spark job performs a **self-join** on a large DataFrame to find matching pairs. The job fails with an `AnalysisException: Resolved attribute(s) missing from child`. What is the most likely cause?

```python
df.join(df, df["id"] == df["partner_id"])  # Fails
```

- A) Self-joins are not supported in PySpark.
- B) The issue is that both sides of the join reference the **same DataFrame object**, so Spark cannot disambiguate which `id` column belongs to which side. Fix by creating an **alias** for one copy.
- C) You must use a `crossJoin()` for self-joins.
- D) The `id` column is not unique, causing the join to fail.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

When you join a DataFrame to itself without aliasing, both sides reference the same logical plan. Spark's Analyzer cannot distinguish `df["id"]` on the left from `df["id"]` on the right — they resolve to the same attribute. The fix is to alias one (or both) copies:

```python
df_left = df.alias("left")
df_right = df.alias("right")

result = df_left.join(
    df_right,
    col("left.id") == col("right.partner_id"),
    how="inner"
)
```

Using `.alias()` creates logically distinct references that the Catalyst Analyzer can resolve unambiguously.

**Why the others are wrong:**
- **(A)** PySpark fully supports self-joins — they're common for hierarchical data, graph traversal, and finding pairs. The syntax just requires aliasing.
- **(C)** `crossJoin()` produces a Cartesian product of all rows with all rows — not a self-join with a condition. It would produce `N²` rows and is almost never what you want.
- **(D)** Non-unique keys cause row multiplication (as in Q21), not an `AnalysisException`. The error here is a query planning failure, not a data problem.

</details>

---

## Section 15: Caching & Persistence

---

**Q46.** A Spark pipeline computes an expensive DataFrame `base_df` and then uses it in **5 subsequent transformations**. A data engineer adds `.cache()` before the 5 operations. Which statement best describes what happens?

```python
base_df = spark.read.parquet("...").filter(...).groupBy(...).agg(...)
base_df.cache()

result1 = base_df.withColumn(...)
result2 = base_df.join(other_df, ...)
result3 = base_df.filter(...)
# ...
```

- A) `cache()` immediately materializes and stores `base_df` in memory.
- B) `cache()` is lazy — `base_df` is **not yet materialized** when `.cache()` is called. It is computed and stored in memory the **first time an action triggers its evaluation**. On subsequent uses, Spark reads from the cached copy instead of recomputing.
- C) `cache()` writes the DataFrame to disk for fault tolerance.
- D) `cache()` only works for the next immediate action; subsequent actions recompute from source.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

In Spark, **all transformations are lazy** — they build a logical plan but don't execute. `.cache()` (equivalent to `.persist(StorageLevel.MEMORY_AND_DISK)`) marks the DataFrame for caching, but the data is only materialized when the **first action** (`count()`, `show()`, `write()`, etc.) is triggered. At that point, Spark computes `base_df` and stores it in the Executor's memory. All 5 subsequent operations then read from cache instead of re-executing the full computation (parquet read + filter + groupBy + agg) 5 times. This is the key optimization for **iterative algorithms** and **multi-branch pipelines**.

Pro tip: call `base_df.count()` immediately after `cache()` to force materialization at a known point.

**Why the others are wrong:**
- **(A)** Nothing in Spark is eager unless you call an action. `cache()` just adds a tag to the DAG — no data moves yet.
- **(C)** `cache()` defaults to `MEMORY_AND_DISK` — it tries memory first and spills to disk only if memory is insufficient. But its primary purpose is reuse, not fault tolerance. Checkpointing is the fault-tolerance mechanism.
- **(D)** `cache()` persists across multiple actions until you explicitly call `.unpersist()` or the data is evicted by memory pressure. Its entire purpose is multi-action reuse.

</details>

---

**Q47.** What is the difference between `cache()` and `persist(StorageLevel.DISK_ONLY)` in PySpark?

- A) They are identical — `cache()` is just a shortcut for `persist()`.
- B) `cache()` stores data as **deserialized JVM objects in memory** (`MEMORY_AND_DISK`). `persist(DISK_ONLY)` serializes the data and writes it to the **local disk of each Executor** — uses no Executor heap memory but is slower to read back (disk I/O vs RAM).
- C) `persist(DISK_ONLY)` replicates data across 2 Executors for fault tolerance; `cache()` does not.
- D) `cache()` is only available for DataFrames; `persist()` is only for RDDs.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

PySpark's storage levels offer a spectrum of speed vs. memory trade-offs:

| Storage Level | Memory | Disk | Serialized | Description |
|---|---|---|---|---|
| `MEMORY_ONLY` | ✅ | ❌ | ❌ | Fastest; OOM if too large |
| `MEMORY_AND_DISK` | ✅ | ✅ (spill) | ❌ | Default `cache()` — spills to disk on pressure |
| `MEMORY_ONLY_SER` | ✅ | ❌ | ✅ | Smaller memory footprint; CPU to deserialize |
| `DISK_ONLY` | ❌ | ✅ | ✅ | No heap pressure; slowest read-back |

Use `DISK_ONLY` when your cached data is too large for Executor memory but you still need to avoid recomputing it (e.g., an expensive join result used 3 times in a complex pipeline).

**Why the others are wrong:**
- **(A)** `cache()` specifically uses `MEMORY_AND_DISK` (deserialized). `persist()` without arguments also uses `MEMORY_AND_DISK`, but `persist(DISK_ONLY)` is a distinctly different storage level.
- **(C)** Replication is a separate dimension of storage levels (`MEMORY_ONLY_2`, `MEMORY_AND_DISK_2`). `DISK_ONLY` does not imply replication.
- **(D)** Both `cache()` and `persist()` are available on both DataFrames and RDDs in PySpark.

</details>

---

## Section 16: Catalyst Optimizer & Query Plans

---

**Q48.** A data engineer runs the following and is surprised the output is the same as without the filter:

```python
df = spark.read.parquet("s3://datalake/events/")
result = df.filter(col("event_date") == "2024-01-01") \
           .select("user_id", "event_type")

result.explain(True)
```

Looking at the `explain()` output, they see the **Physical Plan** shows the filter *before* the parquet scan. Is this a problem?

- A) Yes — filters should always be applied after reading all data for correctness.
- B) No — this is **Predicate Pushdown**, a Catalyst optimization that pushes the filter into the Parquet reader. The reader applies the filter at the file/row-group level before loading data into Spark memory, drastically reducing I/O.
- C) Yes — Catalyst is incorrectly reordering operations, which will cause wrong results.
- D) No, but it only works for string columns — numeric column filters are not pushed down.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

**Predicate Pushdown** is one of Catalyst's most impactful optimizations. Parquet files store metadata about row groups (min/max values, null counts). When Catalyst pushes a filter like `event_date = "2024-01-01"` into the Parquet reader, the reader can: (1) skip entire row groups whose min/max range doesn't include the target date, and (2) for partition-pruned paths, skip entire files. This means Spark may read only 1% of a large dataset before any in-memory filtering occurs. This is visible in `explain()` output as `PushedFilters: [EqualTo(event_date, 2024-01-01)]` inside the `FileScan` node.

**Why the others are wrong:**
- **(A)** Pushdown is purely an optimization — it produces identical results to post-scan filtering but with far less I/O. Correctness is guaranteed by Catalyst's optimizer rules.
- **(C)** Catalyst's rule-based and cost-based rewrites are provably equivalent transformations. They cannot change query results — only execution efficiency.
- **(D)** Predicate pushdown works for all data types supported by the file format's filter API — strings, integers, dates, timestamps, etc. Parquet's predicate pushdown is particularly powerful for numeric and date range filters.

</details>

---

**Q49.** What does `df.explain("formatted")` show, and why should a data engineer read it?

- A) It shows the DataFrame's schema — column names and data types.
- B) It shows the **Parsed → Analyzed → Optimized Logical Plan → Physical Plan** pipeline that Catalyst generates for the query. Reading it reveals join strategies (BHJ vs SMJ), partition pruning, predicate pushdown, and whether expected optimizations fired — essential for diagnosing slow queries.
- C) It shows the number of rows and partitions in the DataFrame.
- D) It shows the Spark configuration settings active for the current session.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

`df.explain("formatted")` (Spark 3.0+) is the **X-ray of your query**. It reveals four plan stages:

1. **Parsed Logical Plan** — what you wrote, before analysis
2. **Analyzed Logical Plan** — after resolving column names and types
3. **Optimized Logical Plan** — after Catalyst applies optimization rules (predicate pushdown, constant folding, etc.)
4. **Physical Plan** — the actual execution strategy (which join algorithm, how many stages, exchange/shuffle points)

Key things to look for:
- `BroadcastHashJoin` vs `SortMergeJoin` (did the broadcast work?)
- `PushedFilters` inside `FileScan` (is predicate pushdown happening?)
- `Exchange` nodes (every Exchange = a shuffle = expensive)
- `*(n)` prefix on operators (whole-stage code generation enabled)

**Why the others are wrong:**
- **(A)** Schema is shown with `df.printSchema()`, not `explain()`.
- **(C)** Row/partition counts require actions (`df.count()`, `df.rdd.getNumPartitions()`). `explain()` is a plan description — it executes nothing.
- **(D)** Active Spark configs are shown with `spark.conf.get("key")` or `spark.sparkContext.getConf().getAll()`.

</details>

---

## Section 17: Partitioning Strategies

---

**Q50.** A data engineer has a 1TB dataset and uses:

```python
df.repartition(10)
```

A colleague says `.coalesce(10)` would be better. What is the **technical difference**, and when should you choose one over the other?

- A) They are identical in output — `coalesce` is just a shorthand for `repartition`.
- B) `repartition(N)` performs a **full shuffle** to redistribute data into exactly N evenly-sized partitions. `coalesce(N)` **avoids a full shuffle** by merging existing partitions locally — but may produce uneven partition sizes. Use `repartition` when you need even distribution (before a heavy join); use `coalesce` when reducing partitions before a write (you want fewer files, don't care about perfect balance).
- C) `coalesce` can only reduce to 1 partition; `repartition` can target any N.
- D) `repartition` works on DataFrames only; `coalesce` works on both DataFrames and RDDs.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

This is a critical distinction for performance:

| Operation | Shuffle? | Even partitions? | Best for |
|---|---|---|---|
| `repartition(N)` | ✅ Full shuffle | ✅ Yes | Pre-join even distribution, increasing partition count |
| `coalesce(N)` | ❌ No shuffle (merge only) | ⚠️ May be uneven | Reducing partitions before write, avoiding shuffle cost |

`coalesce` works by combining adjacent partitions on the same Executor — no data moves across the network. This makes it cheap but potentially unbalanced. `repartition` round-robins (or hashes) all data across N new partitions — balanced but expensive. A common pattern: `df.repartition(200).join(...).coalesce(10).write.parquet(...)` — repartition for the join, coalesce cheaply before write.

**Why the others are wrong:**
- **(A)** They are fundamentally different in whether they trigger a shuffle. This has major performance implications.
- **(C)** `coalesce(N)` works for any N ≤ current partition count. To *increase* partition count, you must use `repartition`.
- **(D)** Both `repartition` and `coalesce` are available on both DataFrames and RDDs.

</details>

---

**Q51.** A data engineer writes a large dataset with:

```python
df.write.partitionBy("country", "event_type").parquet("s3://datalake/events/")
```

There are 200 countries × 50 event types = **10,000 partitions**. The job runs out of memory. What is the cause?

- A) Parquet does not support more than 1,000 partitions.
- B) With 10,000 directory partitions, Spark must maintain **open file handles and write buffers** for all 10,000 output partitions simultaneously (one per partition directory per Executor task). This causes **memory pressure and file handle exhaustion**. Fix: increase `spark.sql.files.maxPartitionBytes`, or reduce cardinality by choosing lower-cardinality partition columns.
- C) The `country` column has too many distinct values — use `repartition("country")` first.
- D) `partitionBy` with two columns is not supported — only one column is allowed.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

This is the **partition explosion problem**. When Spark writes with `partitionBy`, each Executor task may need to write to multiple output directories simultaneously. With 10,000 partitions and, say, 50 concurrent tasks, each task could theoretically write to all 10,000 output directories — each requiring an open write buffer. This causes OOM and OS file handle limits (typically 65,536 per process). Solutions:

1. **Reduce output partition cardinality**: partition only by `country` (200 dirs) instead of both columns.
2. **Pre-sort/repartition by the partition columns** before writing: `df.repartition("country", "event_type").write.partitionBy(...)` — each task writes to exactly one partition directory.
3. **`spark.sql.files.maxRecordsPerFile`**: limits records per file, avoiding huge single files without partition explosion.

**Why the others are wrong:**
- **(A)** Parquet has no partition count limit. The constraint is Spark's memory and OS file handles.
- **(C)** `repartition("country")` would repartition by country only — tasks would still write to all 50 event_type subdirs under their country. The fix is to repartition by *both* partition columns.
- **(D)** `partitionBy` supports multiple columns. The Hive-style directory hierarchy (`country=US/event_type=click/`) is well-supported.

</details>

---

## Section 18: Schema Evolution & Data Quality

---

**Q52.** A Delta Lake table receives daily appends from different upstream teams. Over time, **new columns are added by some teams** but not others. Without any special configuration, what happens when Spark tries to write a DataFrame with extra columns to a Delta table?

- A) Spark automatically adds the new column to the table schema.
- B) Spark throws an `AnalysisException: A schema mismatch detected` error and rejects the write — Delta Lake enforces **schema enforcement** (write compatibility) by default.
- C) Spark silently drops the extra columns and writes only the columns that match the existing schema.
- D) Spark writes the extra columns as JSON blobs in a `_metadata` column.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

Delta Lake enforces **schema validation on every write** by default. If the incoming DataFrame has columns not present in the table's schema (or has incompatible types), Delta rejects the write with an `AnalysisException`. This is an intentional safety feature — it prevents accidental schema corruption. To allow schema evolution, you must explicitly opt in using `mergeSchema`:

```python
df.write.format("delta") \
    .option("mergeSchema", "true") \
    .mode("append") \
    .save("s3://datalake/table/")
```

Or at the session level: `spark.conf.set("spark.databricks.delta.schema.autoMerge.enabled", "true")`.

**Why the others are wrong:**
- **(A)** Auto-adding columns requires `mergeSchema=true`. Without it, Delta is strict.
- **(C)** Silent column dropping would cause silent data loss — Delta never does this. Explicit opt-in is required for any schema change.
- **(D)** There is no `_metadata` JSON blob fallback. Delta's schema enforcement is binary: match or fail.

</details>

---

**Q53.** A pipeline needs to validate that no `user_id` values are null before writing to a critical table. A junior engineer proposes:

```python
assert df.filter(col("user_id").isNull()).count() == 0, "Null user_ids found!"
```

A senior engineer says this will **work but is inefficient in production**. Why, and what is the better approach?

- A) `.count()` is not supported on filtered DataFrames — use `.collect()` instead.
- B) The `assert` will silently pass in production when Python assertions are disabled (`python -O`). Use explicit `if/raise` logic instead.
- C) `.filter().count()` triggers a **full scan of the DataFrame** just to check for nulls — it recomputes the entire pipeline to get a single number. A more efficient approach uses `df.agg(count(when(col("user_id").isNull(), 1)).alias("null_count")).collect()`, or better yet, validates within a single pass using `DataFrame.summary()` or a data quality framework like Great Expectations.
- D) Nulls are automatically handled by Delta Lake — explicit validation is unnecessary.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

The junior engineer's approach is logically correct but has two production problems: (1) it triggers a **separate full scan** just for validation — if the pipeline already computed `df`, this recomputes it (unless cached), doubling the cost; (2) it's fragile and ad-hoc. Better approaches:

```python
# Option 1: Single-pass null check using agg
null_count = df.agg(
    F.count(F.when(F.col("user_id").isNull(), 1)).alias("nulls")
).collect()[0]["nulls"]
if null_count > 0:
    raise ValueError(f"Found {null_count} null user_ids!")

# Option 2: Cache df before validation so the scan isn't repeated
df.cache()
assert df.filter(col("user_id").isNull()).count() == 0

# Option 3 (best for production): Use a data quality framework
# Great Expectations, Deequ (AWS), or Databricks data quality rules
```

For large production pipelines, **Deequ** (Amazon's PySpark-native data quality library) computes all checks in a single optimized pass.

**Why the others are wrong:**
- **(A)** `.count()` is perfectly valid on filtered DataFrames. The issue is performance, not correctness.
- **(B)** While the `-O` optimization flag point is technically valid (Python 3 does disable `assert` with `-O`), this is not the *senior engineer's* primary concern in a data pipeline context — the performance issue is. However, using explicit `if/raise` is indeed better practice.
- **(D)** Delta Lake enforces schema constraints but does not automatically validate null values in user-defined business key columns — that is application-layer responsibility.

</details>

---

## Section 19: Advanced Streaming Patterns

---

**Q54.** A Structured Streaming job reads from Kafka and writes to a Delta table. After a cluster restart, the job resumes from where it left off without reprocessing messages. What mechanism enables this?

```python
query = df.writeStream \
    .format("delta") \
    .option("checkpointLocation", "s3://checkpoints/my_job/") \
    .outputMode("append") \
    .start("s3://datalake/output/")
```

- A) Kafka's built-in consumer group offsets track what has been consumed.
- B) The **checkpoint location** stores the streaming query's progress metadata: Kafka offsets consumed, current watermark state, and aggregation state. On restart, Spark reads this checkpoint and resumes exactly from the last committed offset, ensuring **exactly-once processing**.
- C) Delta Lake's transaction log tracks all inserts, so Spark knows which records have been written and skips duplicates automatically.
- D) Spark replays the entire Kafka topic from the beginning but deduplicates using `dropDuplicates()`.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

The **checkpoint location** is the backbone of Structured Streaming's fault tolerance. It stores three types of state to a fault-tolerant filesystem (S3, ADLS, HDFS):

1. **Source offsets**: the exact Kafka partition offsets processed in each micro-batch
2. **Watermark and window state**: for stateful operations like aggregations
3. **Commit log**: records which micro-batches have been durably written to the sink

On restart, Spark reads the checkpoint, determines the last successfully committed offset, and resumes from there — not from the beginning, not from the current Kafka head. Combined with a transactional sink (like Delta Lake), this provides **end-to-end exactly-once semantics**.

**Why the others are wrong:**
- **(A)** Kafka consumer group offsets track consumption at the consumer level, not Spark's processing/sink-commit level. Spark manages its own offset tracking through checkpoints, which is more granular and tied to write commit status.
- **(C)** Delta's transaction log records what *was written*, but it doesn't know what Kafka offsets correspond to those writes. The checkpoint bridges the source (Kafka offset) to the sink (Delta commit).
- **(D)** Replaying from the beginning and deduplicating is neither what Spark does nor a scalable approach for large Kafka topics with weeks of history.

</details>

---

**Q55.** A streaming pipeline aggregates events by a 10-minute tumbling window. After a production incident, you find that **late-arriving events (arriving 15 minutes after event time)** are being dropped silently. How do you fix this while still bounding state size?

- A) Increase `trigger(processingTime="15 minutes")` — the larger trigger window captures late events.
- B) Set a watermark of 20 minutes: `.withWatermark("event_time", "20 minutes")` — this tells Spark to wait up to 20 minutes for late data before finalizing a window, while still evicting state for windows older than `max_event_time - 20 minutes`.
- C) Switch `outputMode` from `"update"` to `"complete"` — `complete` mode includes all historical data.
- D) Use `rowsBetween(Window.unboundedPreceding, Window.currentRow)` to include all prior events.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

The watermark defines **how long Spark waits for late data**. If events are arriving up to 15 minutes late and the current watermark is 10 minutes (or none), those events fall below the watermark and are dropped. Setting the watermark to 20 minutes ensures:

1. Windows are not finalized until `current_max_event_time - 20 minutes`
2. Events arriving up to 20 minutes late are included in their correct window
3. State for windows older than the watermark threshold is safely evicted

There's a deliberate trade-off: a longer watermark = more late data captured = more state held in memory = more latency before windows are finalized. Choose your watermark based on your SLA (latency tolerance) vs. data completeness requirements.

**Why the others are wrong:**
- **(A)** `processingTime` in `trigger()` controls how frequently Spark runs micro-batches (how often it polls Kafka/processes new data). It has no effect on event-time windows or late data handling.
- **(C)** `outputMode("complete")` re-emits the entire result table on every trigger — it doesn't affect whether late-arriving events update their correct windows. State eviction is still controlled by watermarks.
- **(D)** `rowsBetween`/`rangeBetween` are batch window function frame boundaries (Section 7). They don't apply to Structured Streaming event-time windows.

</details>

---

## Section 20: Advanced DataFrame Operations

---

**Q56.** A data engineer needs to **flatten a nested JSON column** in a DataFrame where `details` is a struct type:

```python
# Schema:
# root
#  |-- user_id: string
#  |-- details: struct
#  |    |-- age: integer
#  |    |-- city: string
#  |    |-- preferences: array<string>
```

Which code correctly extracts all struct fields and explodes the array?

- A) `df.select("user_id", "details.*")` — then explode separately.
- B) `df.select("user_id", col("details.age"), col("details.city"), explode(col("details.preferences")).alias("preference"))`
- C) `df.flatten("details")` is the correct built-in function for this.
- D) You must use a UDF to unpack struct types — native PySpark cannot access nested fields.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

PySpark natively handles nested types without UDFs:

```python
from pyspark.sql.functions import col, explode

result = df.select(
    "user_id",
    col("details.age"),
    col("details.city"),
    explode(col("details.preferences")).alias("preference")
)
```

- `col("details.age")` uses dot notation to access struct fields
- `explode()` creates one row per array element — if a user has 3 preferences, they get 3 rows
- For the full struct expansion shorthand, `col("details.*")` can expand all struct fields at once

Note: `explode()` drops rows where the array is null/empty. Use `explode_outer()` to keep null rows.

**Why the others are wrong:**
- **(A)** `df.select("user_id", "details.*")` is actually valid syntax and does expand all struct fields — but this option says "then explode separately," implying a two-step approach that creates an intermediate DataFrame. Option B is cleaner.
- **(C)** There is no `df.flatten()` method in PySpark's DataFrame API. This is a confusion with pandas.
- **(D)** Struct field access (`col("details.age")`) and `explode()` are fully native Catalyst operations. UDFs are never needed for standard struct/array manipulation.

</details>

---

**Q57.** What is the output of the following code, and what is the common mistake it demonstrates?

```python
from pyspark.sql.functions import col, sum as spark_sum

df = spark.createDataFrame([(1, 100), (1, 200), (2, 300)], ["dept_id", "salary"])

total = df.agg(spark_sum("salary")).collect()[0][0]  # = 600

result = df.filter(col("salary") > total / 3)
result.show()
```

- A) The code throws an error because you cannot use a Python variable inside `.filter()`.
- B) The code works correctly and filters rows where `salary > 200`. No mistake.
- C) The code works, but `.collect()[0][0]` is a **Driver-side scalar operation** — it pulls a single value to the Driver and then sends it back to Executors as a broadcast variable. For a single scalar, this is fine. The mistake to watch for is doing this inside a loop or for large result sets.
- D) `total / 3` performs integer division and silently produces `200` (not `200.0`), potentially causing wrong filter results in Python 2-style code, but is correct in Python 3.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

The code is functionally correct — it outputs rows where `salary > 200` (i.e., the 200 and 300 salary rows). The pattern of calling `.collect()` to get a scalar, then using it in a subsequent transformation, is the **Driver scalar anti-pattern** when overused. For a single value like a global threshold, it's acceptable. The risks occur when:

1. You call `.collect()` **inside a loop** (triggers a full Spark job per iteration)
2. You use it to collect **large datasets** instead of a single scalar (OOM on Driver)
3. You do this for **multiple scalars** — each `.collect()` is a separate Spark job; batching them into a single `agg()` call is more efficient

The truly correct approach for a scalar threshold used in a filter:
```python
# Efficient: one job to compute the threshold
threshold_row = df.agg(spark_sum("salary") / 3).collect()[0][0]
result = df.filter(col("salary") > threshold_row)
```

**Why the others are wrong:**
- **(A)** Python variables holding scalar values (integers, floats, strings) can absolutely be used inside `.filter()` — Spark serializes them as literals in the query plan.
- **(B)** The code works correctly, but saying there is "no mistake" misses the teaching point about the pattern's risks at scale.
- **(D)** In Python 3, `/` always produces float division. `600 / 3 = 200.0`, not `200`. This is not a concern in modern PySpark (Python 3 only since Spark 3.0+).

</details>

---

**Q58.** A data engineer uses `df.groupBy("dept").agg(collect_list("name").alias("members"))`. The resulting `members` column contains **duplicates**. How do you deduplicate within the collected list?

- A) Chain `.distinct()` after the `groupBy` — it deduplicates within each group.
- B) Replace `collect_list` with `collect_set` — it collects **unique values only** (but does not guarantee order within the set).
- C) Apply a UDF after the aggregation to call `list(set(...))` on each row's array.
- D) Use `dropDuplicates(["dept", "name"])` before the `groupBy`.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

`collect_set("name")` is the native PySpark function that collects **distinct values** into a set (array without duplicates). No UDF needed:

```python
from pyspark.sql.functions import collect_set, sort_array

df.groupBy("dept").agg(
    sort_array(collect_set("name")).alias("unique_members")
)
```

Note: `collect_set` returns elements in an **arbitrary order** (set semantics). If order matters, wrap with `sort_array()`. Also note that `collect_set` (and `collect_list`) can be memory-intensive for groups with many elements — for very large groups, consider counting/aggregating instead.

**Why the others are wrong:**
- **(A)** `.distinct()` deduplicates **rows** of the DataFrame, not values *within* an array column. After `groupBy`, there's one row per department — `.distinct()` has no effect on the array contents.
- **(C)** A UDF works but is unnecessary — `collect_set` is a native Catalyst function, making it far faster than calling Python `set()` via a UDF.
- **(D)** `dropDuplicates(["dept", "name"])` before the `groupBy` is actually a valid and correct approach — it removes duplicate `(dept, name)` pairs before collecting. However, `collect_set` achieves the same result in one step and is the idiomatic answer.

</details>

---

## Section 21: SparkContext, SparkSession & Configuration

---

**Q59.** A data engineer writes a PySpark script that creates a new `SparkSession` inside a function that is called 100 times. What happens on the 2nd through 100th calls?

```python
def process_batch(data):
    spark = SparkSession.builder \
        .appName("BatchProcessor") \
        .getOrCreate()
    df = spark.createDataFrame(data)
    # ...
```

- A) Each call creates a new `SparkSession` with a new `SparkContext`, causing 100 JVM processes to be launched.
- B) `getOrCreate()` returns the **existing active `SparkSession`** if one already exists in the JVM — no new session or context is created. The 2nd–100th calls are effectively free lookups. This is the safe and idiomatic way to access Spark inside utility functions.
- C) The 2nd call throws `IllegalStateException: SparkContext already running`.
- D) Each call creates a new `SparkSession` but reuses the existing `SparkContext` — causing 100 sessions but 1 context.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

`SparkSession.builder.getOrCreate()` is specifically designed for this pattern. It checks if an active `SparkSession` already exists in the current JVM/thread context:
- If **yes**: returns the existing session (ignores any new config in `.config()` calls)
- If **no**: creates a new session with the specified configuration

This is the **recommended pattern** for accessing Spark in utility functions, libraries, and notebooks — you never need to pass the `spark` object around as a parameter. It's safe to call inside loops, functions, and even separate modules. The only caveat: if the existing session has different config than what you specified, your config is silently ignored (the existing session wins).

**Why the others are wrong:**
- **(A)** Spark is designed to have one SparkContext per JVM. `getOrCreate()` was built explicitly to prevent duplicate context creation.
- **(C)** `SparkContext already running` was a behavior in older Spark versions when using `SparkContext()` constructor directly. `getOrCreate()` was introduced to avoid exactly this error.
- **(D)** `SparkSession` is a thin wrapper around `SparkContext`. `getOrCreate()` returns the same session object, not a new one wrapping the same context.

</details>

---

**Q60.** A data engineer sets `spark.conf.set("spark.sql.shuffle.partitions", "50")` mid-script. Which statement is correct?

- A) The setting takes effect for all past and future Spark operations in the session.
- B) The setting takes effect only for **new query executions after this point** — already-running jobs and already-planned queries are unaffected. The setting persists for the remainder of the `SparkSession` unless changed again.
- C) `spark.conf.set()` only affects the Driver — Executors use `spark.sparkContext.getConf()`.
- D) Runtime config changes require a session restart to take effect.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

`spark.conf.set()` modifies the **session-level SQL configuration** at runtime. The change applies to all **subsequent query plans** — Spark reads these configs when building query plans (e.g., when determining how many shuffle partitions to create). Queries already in execution use the configuration that was active when their plan was compiled. This means:

```python
# First job uses 200 shuffle partitions (default)
df.groupBy("key").count().write.parquet("out1/")

# Change the config
spark.conf.set("spark.sql.shuffle.partitions", "50")

# Second job uses 50 shuffle partitions
df.groupBy("key").count().write.parquet("out2/")
```

This is useful in scripts that process datasets of different sizes sequentially — tune the partition count per job based on data volume.

**Why the others are wrong:**
- **(A)** Past operations are already compiled or executing — they cannot retroactively pick up new configs.
- **(C)** SQL configs like `spark.sql.shuffle.partitions` are session-level and are automatically communicated to Executors when they're used in physical plan compilation. Executors don't read `spark.conf` directly — the Driver embeds the values into the task serialization.
- **(D)** Runtime config changes via `spark.conf.set()` are designed to take effect immediately without a restart. That's their purpose.

</details>

---

## Section 22: Testing & Debugging PySpark

---

**Q61.** A data engineer wants to **unit test a PySpark transformation function**. What is the recommended approach?

```python
def add_full_name(df):
    return df.withColumn("full_name",
        concat_ws(" ", col("first_name"), col("last_name")))
```

- A) Unit testing PySpark code is not possible — only integration tests on a real cluster are reliable.
- B) Use a **local SparkSession** (`master("local[*]")`) in the test file, create a small test DataFrame with known inputs, call the function, and assert the output using `.collect()` or Spark's built-in comparison utilities.
- C) Mock the `col()` and `concat_ws()` functions using `unittest.mock` to isolate the logic.
- D) Write the test in Scala — Python unit tests cannot validate distributed execution.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

PySpark unit testing is well-supported using a local SparkSession:

```python
import pytest
from pyspark.sql import SparkSession
from pyspark.sql.functions import col

@pytest.fixture(scope="session")
def spark():
    return SparkSession.builder \
        .master("local[2]") \
        .appName("unit-tests") \
        .getOrCreate()

def test_add_full_name(spark):
    input_data = [("John", "Doe"), ("Jane", "Smith")]
    df = spark.createDataFrame(input_data, ["first_name", "last_name"])
    
    result = add_full_name(df)
    
    names = [row["full_name"] for row in result.collect()]
    assert names == ["John Doe", "Jane Smith"]
```

`local[2]` runs Spark with 2 local threads — fast enough for small test DataFrames with no cluster needed. Libraries like `chispa` provide DataFrame equality assertions (`assert_df_equality(result, expected)`).

**Why the others are wrong:**
- **(A)** Local mode makes PySpark unit testing straightforward. The Spark team and community have extensive guidance on this pattern.
- **(C)** Mocking Spark SQL functions is extremely complex and doesn't test actual execution — you'd be testing the mock, not the code. Local Spark is far simpler and tests real behavior.
- **(D)** Python PySpark tests run on a local JVM via the Python↔JVM bridge. They test the same execution engine as Scala tests.

</details>

---

**Q62.** A Spark job fails with `java.lang.OutOfMemoryError: Java heap space` on the **Driver**. What are the most likely causes?

- A) The Executors ran out of memory — increase `spark.executor.memory`.
- B) The Driver JVM ran out of memory, most commonly caused by: (1) calling `.collect()` on a large DataFrame, pulling too much data to the Driver; (2) using `.toPandas()` on a large DataFrame; (3) broadcasting a very large variable; or (4) accumulating too many results in Driver-side variables in a loop.
- C) The Spark shuffle is producing too many partitions — reduce `spark.sql.shuffle.partitions`.
- D) The Parquet files are corrupted — re-read with `option("mode", "DROPMALFORMED")`.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

The Driver is a single JVM process that runs the `main()` function, coordinates the DAG, collects results, and manages broadcasts. Driver OOM is caused by pulling too much data *to the Driver*, not by distributed processing. Common culprits:

```python
# ❌ Pulls entire DataFrame to Driver memory
all_data = df.collect()

# ❌ Converts Spark DataFrame to pandas in Driver memory
pandas_df = large_df.toPandas()

# ❌ Broadcasting a 10GB "small" table
spark.sparkContext.broadcast(huge_dict)

# ❌ Collecting in a loop
results = [df.filter(col("id") == i).collect() for i in range(10000)]
```

Fixes: `df.limit(N).collect()` for sampling, `df.write` instead of `.collect()`, `df.sample()` before `.toPandas()`, increase `spark.driver.memory`.

**Why the others are wrong:**
- **(A)** Executor OOM gives a different error or `SparkException: Task failed` — Driver OOM specifically means the Driver JVM is exhausted.
- **(C)** Shuffle partition count affects Executor memory (individual task size). The Driver doesn't participate in shuffles.
- **(D)** Corrupt Parquet files cause read errors or `SparkException` — not Driver heap OOM.

</details>

---

## 📊 Score Tracker — Part 3

| Section | Questions | Your Score |
|---|---|---|
| Joins — Strategies, Skew & Broadcast | Q43 – Q45 | &nbsp;&nbsp;&nbsp;/ 3 |
| Caching & Persistence | Q46 – Q47 | &nbsp;&nbsp;&nbsp;/ 2 |
| Catalyst Optimizer & Query Plans | Q48 – Q49 | &nbsp;&nbsp;&nbsp;/ 2 |
| Partitioning Strategies | Q50 – Q51 | &nbsp;&nbsp;&nbsp;/ 2 |
| Schema Evolution & Data Quality | Q52 – Q53 | &nbsp;&nbsp;&nbsp;/ 2 |
| Advanced Streaming Patterns | Q54 – Q55 | &nbsp;&nbsp;&nbsp;/ 2 |
| Advanced DataFrame Operations | Q56 – Q58 | &nbsp;&nbsp;&nbsp;/ 3 |
| SparkContext, SparkSession & Config | Q59 – Q60 | &nbsp;&nbsp;&nbsp;/ 2 |
| Testing & Debugging | Q61 – Q62 | &nbsp;&nbsp;&nbsp;/ 2 |
| **Total** | **Q43 – Q62** | **&nbsp;&nbsp;&nbsp;/ 20** |

---

## 🧠 Key Concepts Cheat Sheet — Part 3

| Concept | One-Line Summary |
|---|---|
| **Broadcast Hash Join** | Sends small table to all Executors — eliminates shuffle on the large table |
| **Data Skew** | Hot keys flood one partition; fix with salting (random suffix + explode small side) |
| **Self-Join Ambiguity** | Alias both sides with `.alias()` so Catalyst can distinguish left vs right columns |
| **`cache()` is Lazy** | Data is materialized on first action, not on the `.cache()` call itself |
| **`MEMORY_AND_DISK` vs `DISK_ONLY`** | Disk-only = no heap pressure; useful when cached data exceeds Executor RAM |
| **Predicate Pushdown** | Catalyst pushes filters into file readers — skips row groups before loading into Spark |
| **`explain("formatted")`** | Reveals join strategy, pushed filters, exchanges (shuffles) — essential for diagnosis |
| **`repartition` vs `coalesce`** | repartition = full shuffle + even partitions; coalesce = local merge + cheaper |
| **Partition Explosion** | Too many `partitionBy` columns = too many open write buffers = OOM on write |
| **Delta Schema Enforcement** | Rejects writes with extra/incompatible columns by default; opt in with `mergeSchema` |
| **Checkpoint Location** | Stores Kafka offsets + watermark state; enables exactly-once on restart |
| **Watermark vs Trigger** | Watermark = late data tolerance; trigger = how often micro-batches run |
| **`collect_set` vs `collect_list`** | `collect_set` = deduplicated array; `collect_list` = all values including duplicates |
| **`getOrCreate()`** | Safe SparkSession access from any function — returns existing session, not a new one |
| **`spark.conf.set()` at Runtime** | Takes effect on future query plans only; useful for per-job tuning in sequential scripts |
| **Driver OOM** | Caused by `.collect()`, `.toPandas()`, or large broadcasts — keep data on Executors |
| **Data Quality Validation** | Use `collect_set`/`agg` in one pass; avoid repeated `.filter().count()` scans |
| **PySpark Unit Testing** | `local[*]` SparkSession + small test DataFrames + `chispa` for assertions |
| **`dropDuplicates` before groupBy** | Alternative to `collect_set` for deduplication — valid but less idiomatic |
| **`explode` vs `explode_outer`** | `explode` drops null/empty arrays; `explode_outer` keeps them as null rows |

---

*Combined with Parts 1 & 2, you now have 62 scenario-based questions covering the full PySpark certification syllabus. Good luck! 🚀*
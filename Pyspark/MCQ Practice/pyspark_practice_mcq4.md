# 🔥 PySpark Certification Practice Test — Part 4
### Advanced | Logic & Concept Focused

> **Instructions:** Read each scenario carefully and select the best answer before expanding the hidden answer block. Each explanation covers *why* the correct answer is right and *why* the other options are wrong. Topics cover UDFs & performance, memory management, Delta Lake internals, window functions, job tuning, file formats, and more.

---

## Section 23: UDFs — Performance, Serialization & Alternatives

---

**Q63.** A data engineer replaces a native Spark expression with a Python UDF for string normalization. The job becomes **10× slower**. What is the root cause?

```python
# Before (native)
df.withColumn("clean", lower(trim(col("name"))))

# After (UDF)
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

@udf(returnType=StringType())
def normalize(name):
    return name.strip().lower() if name else None

df.withColumn("clean", normalize(col("name")))
```

- A) UDFs are single-threaded — they bypass Spark's parallel execution model entirely.
- B) Python UDFs break **Catalyst optimization** and require **row-by-row data serialization** between the JVM and the Python interpreter via Py4J/pickle. Each row is serialized from JVM to Python, processed in Python, then serialized back. This kills vectorized execution, predicate pushdown, and whole-stage code generation.
- C) The `@udf` decorator adds overhead per call — use `spark.udf.register()` instead.
- D) UDFs only work on string columns when registered with `spark.udf.register()`.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

Python UDFs are the single biggest performance anti-pattern in PySpark. Here's what happens under the hood:

```
JVM Executor → serialize row to Python pickle → send over socket
→ Python process deserializes → runs Python function → serializes result
→ sends back to JVM → JVM deserializes
```

This happens **once per row**. For 100M rows, that's 100M serialization round trips. Contrast with native Spark functions: they operate entirely in the JVM on columnar data using whole-stage code generation — no serialization, no Python overhead.

**The hierarchy of performance (best to worst):**
1. **Native Spark SQL functions** (`lower()`, `trim()`, etc.) — JVM, vectorized, Catalyst-optimized
2. **Pandas UDFs (vectorized UDFs)** — data sent as Arrow batches, not row-by-row
3. **Python UDFs** — row-by-row serialization, slowest

Always check if a native function exists before writing a UDF. If you must write a UDF, use a **Pandas UDF** (`@pandas_udf`):

```python
from pyspark.sql.functions import pandas_udf
from pyspark.sql.types import StringType
import pandas as pd

@pandas_udf(StringType())
def normalize_pandas(s: pd.Series) -> pd.Series:
    return s.str.strip().str.lower()

df.withColumn("clean", normalize_pandas(col("name")))
```

**Why the others are wrong:**
- **(A)** UDFs do run in parallel across partitions — each Executor runs the Python function on its partition. The bottleneck is serialization overhead, not single-threading.
- **(C)** `@udf` decorator and `spark.udf.register()` have identical performance. The decorator is just syntactic sugar. `register()` also makes the UDF available in Spark SQL string expressions.
- **(D)** UDFs work on any column type — the `returnType` just needs to be declared correctly.

</details>

---

**Q64.** When should you use a **Pandas UDF** (`@pandas_udf`) over a native Spark function?

- A) Always — Pandas UDFs are faster than all native Spark functions.
- B) When you need a **complex custom transformation** that cannot be expressed with native Spark functions (e.g., custom NLP, proprietary business logic, ML inference per group), and you want better performance than a row-by-row Python UDF by leveraging **Apache Arrow** for columnar batch transfer.
- C) Only for numeric columns — Pandas UDFs do not support string types.
- D) Pandas UDFs are only available in Databricks — not in open-source PySpark.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

Pandas UDFs (also called Vectorized UDFs) use **Apache Arrow** as the serialization format between JVM and Python. Instead of sending one row at a time, Arrow sends entire columns as contiguous memory buffers. This is **10–100× faster** than row-by-row Python UDFs.

```python
from pyspark.sql.functions import pandas_udf
import pandas as pd

# Scalar Pandas UDF — processes a column (pd.Series) at a time
@pandas_udf("double")
def fahrenheit_to_celsius(temp: pd.Series) -> pd.Series:
    return (temp - 32) * 5 / 9

# Grouped Map UDF — processes a group (pd.DataFrame) at a time
# Useful for applying scikit-learn model per group, time-series per entity
from pyspark.sql.functions import PandasUDFType

@pandas_udf(schema, PandasUDFType.GROUPED_MAP)
def forecast_per_store(pdf: pd.DataFrame) -> pd.DataFrame:
    # Apply custom forecasting model per store group
    return my_model.predict(pdf)

df.groupBy("store_id").apply(forecast_per_store)
```

**Use Pandas UDFs when:** native functions can't express the logic, and you need Python/pandas/scikit-learn/numpy operations.
**Use native functions when:** the operation exists natively — they're still faster than Pandas UDFs.

**Why the others are wrong:**
- **(A)** Native Spark functions like `sum()`, `lower()`, `date_add()` run entirely in the JVM with whole-stage code generation — they're faster than Pandas UDFs for operations they support. Pandas UDFs are faster than Python row UDFs, not faster than native functions.
- **(C)** Pandas UDFs support all types that Arrow supports: strings, numerics, dates, booleans, arrays, and structs.
- **(D)** Pandas UDFs are part of open-source PySpark since Spark 2.3. They require PyArrow to be installed (`pip install pyarrow`).

</details>

---

## Section 24: Memory Management & Spilling

---

**Q65.** A Spark job's Executor logs show `SpillSize: 45.6 GB` during a shuffle stage. What does this mean, and what are the consequences?

- A) 45.6 GB of data was written to S3 — this is normal behavior for large jobs.
- B) During the shuffle, Executors ran out of heap memory and **spilled 45.6 GB of shuffle data to local disk**. Spark then had to read that data back from disk for downstream stages. Spilling itself doesn't cause job failure, but it causes **severe slowdowns** (disk I/O is 100–1000× slower than memory) and **excessive GC pressure**.
- C) 45.6 GB of data was duplicated across Executors for fault tolerance.
- D) The shuffle produced 45.6 GB of output data — this is informational only.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

**Spilling** is Spark's safety valve when in-memory shuffle buffers overflow. The write path is:
```
Executor memory fills up → serialize data → write to local disk (spill file)
→ next stage reads from disk instead of memory → massive slowdown
```

Spilling is **not** a crash, but it signals the job is memory-constrained. Strategies to reduce spilling:

1. **Increase `spark.executor.memory`** — more heap for shuffle buffers
2. **Increase `spark.memory.fraction`** (default 0.6) — allocates more JVM heap to Spark execution vs. user data storage
3. **Reduce shuffle data size** — use filters/projections earlier, use `reduceByKey` instead of `groupByKey` (pre-aggregates on mapper side)
4. **Increase shuffle partitions** (`spark.sql.shuffle.partitions`) — smaller data per partition = less memory per task
5. **Use broadcast joins** to eliminate the shuffle entirely for small tables

```python
# Before: causes large shuffle
df.groupByKey().mapValues(sum)  # sends all values to reducer

# After: pre-aggregates on mapper side, less shuffle data
df.reduceByKey(lambda a, b: a + b)  # combines locally first
```

**Why the others are wrong:**
- **(A)** Spill refers to **local disk on the Executor node**, not external storage like S3. S3 writes are controlled by `write()` calls.
- **(C)** Data replication for fault tolerance is controlled by `StorageLevel` settings (e.g., `MEMORY_AND_DISK_2`). Spilling is a memory pressure response, not a replication mechanism.
- **(D)** Shuffle output size is tracked separately. SpillSize specifically means data that overflowed memory and hit disk.

</details>

---

**Q66.** What is the difference between **Spark's Execution Memory** and **Storage Memory**, and what happens when one needs more space?

- A) Execution Memory is on the Driver; Storage Memory is on Executors.
- B) Under the **Unified Memory Model** (Spark 1.6+), both Execution Memory (used for shuffles, joins, sorts, aggregations) and Storage Memory (used for `cache()`/`persist()`) share a single memory pool. If Execution needs more space, it can **evict cached data** from Storage. Storage cannot evict Execution memory (in-progress computations cannot be interrupted). This is why caching too aggressively can hurt active job performance.
- C) Execution Memory is managed by the JVM GC; Storage Memory bypasses GC using off-heap storage.
- D) They are always equal halves of `spark.executor.memory`.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

The **Unified Memory Manager** (default since Spark 1.6) replaced the old static split model. Key properties:

```
Total Executor Memory
├── Reserved Memory (300MB fixed — Spark internals)
├── User Memory (1 - spark.memory.fraction = 40% default)
│     └── User data structures, UDF objects
└── Spark Memory Pool (spark.memory.fraction = 0.6 default)
      ├── Execution Memory (shuffles, sorts, joins, aggs)
      └── Storage Memory (cache/persist)
            ↑ These two share the pool dynamically ↑
```

**Eviction rules:**
- Execution can evict Storage blocks (cached DataFrames get dropped to make room for active computation)
- Storage **cannot** evict Execution (you can't stop a running sort to free memory)

**Practical implication:** If you cache a large DataFrame and then run an expensive join, the join may evict your cache. The cache will be recomputed on next access (lazy re-execution from the original plan), causing unexpected slowdowns.

**Why the others are wrong:**
- **(A)** Both Execution and Storage Memory reside on Executors. The Driver has its own separate memory (for `.collect()`, broadcasts, driver-side state).
- **(C)** Off-heap memory is a separate optional feature (`spark.memory.offHeap.enabled=true`). By default, both Execution and Storage are on-heap JVM memory.
- **(D)** The old static model (Spark < 1.6) split memory into fixed fractions. The modern unified model is dynamic. `spark.memory.fraction` (default 0.6) sets the combined pool size, not a 50/50 split.

</details>

---

## Section 25: Delta Lake Internals

---

**Q67.** What is the **Delta Lake Transaction Log** (`_delta_log`), and why is it critical for ACID guarantees?

- A) It's a backup copy of all data files stored in a separate location.
- B) The `_delta_log` is a **directory of JSON commit files** where every table operation (INSERT, UPDATE, DELETE, MERGE, schema change) is recorded as an atomic JSON entry. Each entry lists which files were added/removed. This log is what enables **ACID transactions**: readers use the log to determine which files constitute the current table snapshot; writers use **optimistic concurrency** (compare-and-swap on log files) to prevent conflicts.
- C) It's a Spark SQL metadata catalog used only by Databricks Unity Catalog.
- D) It's an audit trail used only for compliance — removing it doesn't affect query results.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

The `_delta_log` is the heart of Delta Lake. Every operation appends a numbered JSON file:

```
_delta_log/
  00000000000000000000.json  ← initial table creation
  00000000000000000001.json  ← first INSERT
  00000000000000000002.json  ← UPDATE
  00000000000000000003.json  ← DELETE
  00000000000000000010.checkpoint.parquet ← checkpoint every 10 commits
```

Each JSON file contains:
- **`add`** entries: files added in this commit
- **`remove`** entries: files logically deleted (tombstones)
- **`metaData`**: schema changes
- **`commitInfo`**: operation type, timestamp, user

**Why this enables ACID:**
- **Atomicity**: a commit either fully writes its JSON file or doesn't — no partial states
- **Isolation**: readers snapshot the log at a point in time; concurrent writers use log-file CAS
- **Time Travel**: `VERSION AS OF 5` just means "replay log up to commit 5"

```python
# Time travel — reads table state at version 5
spark.read.format("delta").option("versionAsOf", 5).load("s3://table/")

# Or by timestamp
spark.read.format("delta").option("timestampAsOf", "2024-01-01").load("s3://table/")
```

**Why the others are wrong:**
- **(A)** The `_delta_log` contains only **metadata** (file paths and statistics), not data copies. The actual data lives in Parquet files alongside the log.
- **(C)** The `_delta_log` is part of the open-source Delta Lake protocol. It works with any Spark environment, not just Databricks.
- **(D)** Deleting `_delta_log` would make the table unreadable — Delta readers parse the log to determine which Parquet files are current. Without it, you'd have orphaned files with no way to know the table's current state.

</details>

---

**Q68.** A Delta table has been running for 6 months with daily writes. The `_delta_log` has 180 commit files and the data directory has **thousands of small Parquet files** from incremental appends. Queries are getting slower. What two operations fix this?

- A) Delete old commit files and re-create the table from scratch.
- B) Run **`OPTIMIZE`** to compact small Parquet files into larger ones (improving read performance via larger row groups), and **`VACUUM`** to physically delete Parquet files that were logically removed by prior DELETEs/UPDATEs but are still on disk for time travel.
- C) Increase `spark.sql.shuffle.partitions` — more partitions will spread the read load.
- D) Run `df.repartition(1).write.format("delta").mode("overwrite")` to consolidate into one file.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

Two key Delta Lake maintenance operations:

**`OPTIMIZE`** — File Compaction:
```sql
OPTIMIZE delta.`s3://table/`
-- Or with Z-ordering for better data skipping:
OPTIMIZE delta.`s3://table/` ZORDER BY (user_id, event_date)
```
Merges small files (e.g., hundreds of 1MB files) into larger files (~1GB target). Dramatically reduces the number of file-open operations during reads. Z-ordering co-locates related data in the same files, boosting data skipping effectiveness.

**`VACUUM`** — Physical Deletion:
```sql
VACUUM delta.`s3://table/` RETAIN 168 HOURS  -- keep 7 days for time travel
```
Deletes Parquet files that are no longer referenced by the current (or recent) table version. Without VACUUM, logically deleted data stays on disk forever, increasing storage costs and slowing file listing operations.

**Important:** Never VACUUM with retention < 7 days (168 hours) unless you're sure no concurrent readers are using older versions. The default is 7 days.

**Why the others are wrong:**
- **(A)** Deleting `_delta_log` destroys the table's ACID guarantees. Never manually delete log files.
- **(C)** Shuffle partitions affect write-time data distribution, not read-time file listing. The slow queries are caused by too many small files, not shuffle configuration.
- **(D)** Writing with `.mode("overwrite")` would work to compact, but it discards all Delta history and schema metadata. It's also dangerous for concurrent readers. `OPTIMIZE` is the safe, designed mechanism.

</details>

---

**Q69.** A team runs `UPDATE` and `DELETE` operations on a Delta table daily. After a month, they notice **storage costs have doubled** even though the logical data size hasn't changed. What is the cause?

- A) Delta Lake makes full copies of the table on every write.
- B) Delta Lake uses **copy-on-write (CoW) semantics**: UPDATE and DELETE do not modify files in place. Instead, they write **new Parquet files** with the changed rows and tombstone the old files in the `_delta_log`. The old files remain on disk for time travel. Over time, this accumulates **dead files** consuming storage. `VACUUM` is required to physically delete them.
- C) Delta Lake stores data in both Parquet and ORC format for compatibility.
- D) The `_delta_log` itself is growing and consuming the extra storage.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

Delta Lake's **Copy-on-Write** model for row-level operations:

```
Original file: [row1, row2, row3, row4]  ← file_001.parquet

DELETE row2:
→ Read file_001.parquet
→ Write new file: [row1, row3, row4]  ← file_002.parquet (new)
→ Log entry: {add: file_002, remove: file_001}
→ file_001.parquet still on disk! (for time travel)
```

After 30 days of daily DELETEs, you have 30 generations of files. The `remove` entries in the log mark them as logically deleted, but physically they sit on S3/ADLS. `VACUUM` with appropriate retention cleans them up:

```python
from delta.tables import DeltaTable
dt = DeltaTable.forPath(spark, "s3://table/")
dt.vacuum(retentionHours=168)  # Delete files older than 7 days
```

**Deletion Vectors** (Delta Lake 2.3+) partially address this by recording deleted row positions within files rather than rewriting files — reducing write amplification for delete-heavy workloads.

**Why the others are wrong:**
- **(A)** Delta does not copy the entire table on writes. CoW only rewrites **affected files** (the files containing updated/deleted rows), not the entire table.
- **(C)** Delta Lake stores data exclusively in Parquet format (plus JSON log files). It doesn't maintain ORC copies.
- **(D)** JSON commit files are tiny (kilobytes each). 30 daily commits = ~30 KB of log files — negligible compared to GB/TB of data files.

</details>

---

## Section 26: Window Functions — Advanced Patterns

---

**Q70.** A data engineer needs to assign a **running total that resets every month** within each `customer_id`. Which window specification is correct?

```python
# Table: customer_id | order_date | amount
# Goal: running_total resets at the start of each month per customer
```

- A) `Window.partitionBy("customer_id").orderBy("order_date")`
- B) `Window.partitionBy("customer_id", month("order_date"), year("order_date")).orderBy("order_date")` with `rowsBetween(Window.unboundedPreceding, Window.currentRow)`
- C) `Window.partitionBy("customer_id").orderBy(month("order_date"))`
- D) You cannot reset window functions mid-partition — use `groupBy` then `join` instead.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

The key insight is that **partitioning** defines the reset boundary. By partitioning by `(customer_id, year, month)`, each customer-month combination is a separate partition — the running total starts fresh for each:

```python
from pyspark.sql.functions import sum, month, year, col
from pyspark.sql.window import Window

w = Window.partitionBy(
    "customer_id",
    year("order_date"),
    month("order_date")
).orderBy("order_date") \
 .rowsBetween(Window.unboundedPreceding, Window.currentRow)

df.withColumn("monthly_running_total", sum("amount").over(w))
```

- `partitionBy(customer_id, year, month)` → each partition = one customer's one month
- `orderBy(order_date)` → rows within the partition are ordered chronologically
- `rowsBetween(unboundedPreceding, currentRow)` → sum from start of partition to current row = running total that resets each month

**Why the others are wrong:**
- **(A)** Partitioning only by `customer_id` means the running total accumulates across all months — it never resets.
- **(C)** Ordering by `month()` alone loses the chronological ordering within the month, and also doesn't handle year boundaries correctly (December 2023 and December 2024 would be mixed).
- **(D)** While a `groupBy + join` pattern can work, it's verbose and typically slower. Window functions are designed precisely for this pattern.

</details>

---

**Q71.** What is the difference between `lag()` and `lead()` window functions, and what is a common use case for each?

- A) `lag()` returns the next row's value; `lead()` returns the previous row's value.
- B) `lag(col, n)` returns the value of `col` from **n rows before** the current row in the window. `lead(col, n)` returns the value from **n rows after**. Common use: `lag` for computing period-over-period change (today's value minus yesterday's); `lead` for computing time-to-next-event or forward-looking metrics.
- C) They are identical — `lag` and `lead` produce the same result but in different sort orders.
- D) `lag` and `lead` only work with numeric columns.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

```python
from pyspark.sql.functions import lag, lead, col
from pyspark.sql.window import Window

w = Window.partitionBy("stock_ticker").orderBy("trade_date")

df.withColumn("prev_close", lag("close_price", 1).over(w)) \
  .withColumn("next_close", lead("close_price", 1).over(w)) \
  .withColumn("daily_change", col("close_price") - col("prev_close")) \
  .withColumn("days_to_next_earnings",
      lead("earnings_date", 1).over(w))
```

**`lag()` use cases:**
- Day-over-day / period-over-period change: `revenue - lag(revenue, 1)`
- Detecting state changes: `status != lag(status, 1)` (session detection)
- Computing time gaps: `timestamp - lag(timestamp, 1)`

**`lead()` use cases:**
- Time to next event: `lead(event_time) - event_time`
- Forward-fill validation: check if next row belongs to same session
- Lookahead features in ML feature engineering

Both return `null` for rows where the offset extends beyond the partition boundary. Use the default value parameter to handle this: `lag("col", 1, 0)` returns 0 instead of null for the first row.

**Why the others are wrong:**
- **(A)** This is the opposite of the correct definition. `lag` = look back, `lead` = look forward. A common confusion on exams.
- **(C)** They produce different results unless the window has only one row. They are not symmetric operations.
- **(D)** `lag` and `lead` work on any column type — strings, dates, booleans, etc. The offset `n` must be an integer, but the column itself can be any type.

</details>

---

**Q72.** A data engineer uses `dense_rank()` to rank employees by salary. What is the output for the following data, and how does it differ from `rank()` and `row_number()`?

```python
# Data: salaries [100, 100, 90, 80, 80, 70]
# All employees, ordered by salary DESC
```

- A) `dense_rank`: [1,1,2,3,3,4] | `rank`: [1,1,3,4,4,6] | `row_number`: [1,2,3,4,5,6]
- B) `dense_rank`: [1,1,2,3,3,4] | `rank`: [1,1,2,3,3,4] | `row_number`: [1,2,3,4,5,6]
- C) `dense_rank`: [1,2,3,4,5,6] | `rank`: [1,1,3,4,4,6] | `row_number`: [1,1,2,3,3,4]
- D) All three produce [1,1,2,3,3,4] — they differ only in null handling.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: A**

This is a classic interview question. The three ranking functions handle ties differently:

| Salary | `dense_rank()` | `rank()` | `row_number()` |
|---|---|---|---|
| 100 | 1 | 1 | 1 |
| 100 | 1 | 1 | 2 |
| 90 | 2 | 3 | 3 |
| 80 | 3 | 4 | 4 |
| 80 | 3 | 4 | 5 |
| 70 | 4 | 6 | 6 |

- **`dense_rank()`**: No gaps after ties. Tied rows share rank N; next distinct value gets rank N+1.
- **`rank()`**: Gaps after ties. Tied rows share rank N; next distinct value gets rank N + (number of tied rows). Like sports rankings: two gold medals means no silver.
- **`row_number()`**: Strictly sequential, no ties. Duplicate rows get arbitrary but unique numbers (non-deterministic for ties unless a tiebreaker column is added to `orderBy`).

```python
from pyspark.sql.functions import dense_rank, rank, row_number
from pyspark.sql.window import Window

w = Window.orderBy(col("salary").desc())

df.withColumn("dense_r", dense_rank().over(w)) \
  .withColumn("rank_r", rank().over(w)) \
  .withColumn("row_num", row_number().over(w))
```

**Use `dense_rank`** for: "Top N salary bands", competitive rankings without gaps.  
**Use `rank`** for: Sports-style rankings where gaps signal skipped positions.  
**Use `row_number`** for: Deduplication (keep the first row per group), pagination.

**Why the others are wrong:**
- **(B)** `rank()` does NOT produce [1,1,2,3,3,4] — that's `dense_rank()`. `rank()` skips numbers after ties.
- **(C)** `row_number()` produces unique sequential integers — it cannot produce ties like [1,1,2,3,3,4].
- **(D)** The three functions are fundamentally different in tie-handling semantics, not just null handling.

</details>

---

## Section 27: File Formats & I/O Optimization

---

**Q73.** A data pipeline writes results to Parquet. A downstream team reports that their queries using `SELECT *` are very slow even on small row counts. The engineer finds 50,000 tiny files (avg 10KB each). What caused this and how do you fix it?

- A) Parquet has a 10KB per-file limit — switch to ORC format.
- B) The **small file problem**: each Spark task writes one file per output partition, and if there are 50,000 tasks (due to a high `spark.sql.shuffle.partitions` or over-repartitioning), 50,000 files are created. Each file open operation on S3/HDFS has ~5–20ms overhead. For 50,000 files: up to 1,000 seconds of file-open latency before reading a single byte. Fix: use `coalesce()` before writing, or `OPTIMIZE` on Delta, or reduce shuffle partitions.
- C) Parquet files should be compressed — add `.option("compression", "snappy")` to the write.
- D) Use `.write.mode("overwrite")` instead of `"append"` — append mode creates too many files.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

The **small file problem** is one of the most common production data lake issues. The file open overhead dominates total query time:

```
Query latency = (N_files × per-file-open overhead) + actual read time
             = (50,000 × 10ms) + (500MB / read_throughput)
             = 500 seconds + 0.5 seconds
```

The file open latency completely dominates!

**Prevention strategies:**
```python
# Before writing: reduce to reasonable file count
df.coalesce(100).write.parquet("s3://output/")  # Merge first, then write

# Or repartition by a column to co-locate related data
df.repartition(100, "country").write.partitionBy("country").parquet("s3://output/")

# Tune shuffle partitions to match data volume
spark.conf.set("spark.sql.shuffle.partitions", "200")  # Default 200; tune for your data
```

**Target file size:** Aim for 128MB–1GB per Parquet file. `spark.sql.files.maxPartitionBytes` (default 128MB) controls input splits; for output, control via partition count and data volume.

**Why the others are wrong:**
- **(A)** Parquet has no file size limit. ORC would have the same problem with many small files.
- **(C)** Snappy compression reduces file size but doesn't address the number of files. 50,000 × 8KB files are still 50,000 file-open operations.
- **(D)** `append` vs `overwrite` mode doesn't affect how many files are created per write. The file count is determined by the number of output partitions at write time.

</details>

---

**Q74.** What is **Parquet column pruning**, and how does it improve query performance?

- A) Parquet removes duplicate column values to reduce file size.
- B) When a query reads only a subset of columns (e.g., `SELECT user_id, amount` from a 100-column table), **Parquet's columnar format allows Spark to read only those columns' byte ranges** from disk — skipping all other columns entirely. For a 100-column table where only 3 are selected, this can reduce I/O by 97%.
- C) Column pruning re-orders columns alphabetically for faster binary search.
- D) Column pruning only works when all columns are the same data type.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

**Why columnar format enables pruning:**

In row-based formats (CSV, JSON, Avro):
```
Row 1: [col1, col2, col3, ... col100]
Row 2: [col1, col2, col3, ... col100]
```
To read `col3`, you must parse every row's full serialized record.

In columnar Parquet:
```
col1 data: [r1_col1, r2_col1, r3_col1, ...]  ← contiguous on disk
col2 data: [r1_col2, r2_col2, r3_col2, ...]  ← contiguous on disk
col3 data: [r1_col3, r2_col3, r3_col3, ...]  ← contiguous on disk
```
To read `col3`, you seek directly to `col3`'s byte offset and read only that column's data.

Catalyst automatically applies column pruning — it analyzes your query's `SELECT` clause and pushes the column selection into the `FileScan` operator. Visible in `explain()` as `ReadSchema: struct<user_id:string,amount:double>` (only selected columns, not all 100).

Combined with **row group filtering** (predicate pushdown), Parquet can skip both irrelevant columns AND irrelevant rows — making it ideal for analytical queries on wide tables.

**Why the others are wrong:**
- **(A)** Value deduplication in Parquet is done via **dictionary encoding** (a separate feature). Column pruning is about reading fewer columns, not deduplicating values.
- **(C)** Columns in Parquet are stored in their defined schema order, not alphabetically. Access is by byte offset, not linear search.
- **(D)** Column pruning works regardless of data types. All columns in Parquet are stored independently and can be selectively read.

</details>

---

## Section 28: Job Tuning & Cluster Configuration

---

**Q75.** A Spark job with 1,000 tasks runs on a cluster with **10 Executors, each with 4 cores**. Most tasks finish in 10 seconds, but the job takes 5 minutes. What is the likely cause, and how do you diagnose it?

- A) 10 Executors × 4 cores = 40 concurrent tasks; 1,000 tasks ÷ 40 = 25 waves. Expected time ≈ 25 × 10s = 250s ≈ 4 minutes. The extra 1 minute suggests a few **straggler tasks** — check the Spark UI's **Stage detail** for tasks with anomalously long durations and large data sizes (data skew) or GC time.
- B) The cluster needs 1,000 cores to run 1,000 tasks simultaneously.
- C) Too many Executors are competing for the same HDFS blocks — reduce to 5 Executors.
- D) The job is waiting for 1,000 sequential tasks — Spark cannot parallelize tasks within a stage.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: A**

This is a **task wave analysis** problem:

```
Parallel capacity = 10 Executors × 4 cores = 40 concurrent tasks
Task waves = ceil(1,000 / 40) = 25 waves
Expected time = 25 × 10s = 250s ≈ 4.2 minutes
Actual time = 5 minutes = 300s → ~50s extra
```

The 50-second extra comes from **straggler tasks** in the final waves. Common causes:

1. **Data skew**: a few partitions have 10× more data (diagnose: Spark UI → Stage → Task metrics → "Input Size / Records" column)
2. **GC pressure**: tasks spending time in JVM garbage collection (diagnose: "GC Time" column in task metrics)
3. **Speculative execution stragglers**: hardware issues on one node (diagnose: look for tasks on the same Executor being consistently slow)

**Spark UI checklist for a slow stage:**
- Sort tasks by "Duration" descending — are the slowest tasks on the same Executor?
- Check "Shuffle Read Size" — are a few tasks reading 10× more shuffle data?
- Check "GC Time" — is it > 10% of task duration?
- Check "Spill (Memory)" — are tasks spilling to disk?

**Why the others are wrong:**
- **(B)** Spark doesn't require 1 core per task simultaneously. Tasks run in waves across available cores — 40 tasks at a time is correct for 40 cores.
- **(C)** Multiple Executors reading different HDFS blocks is standard and efficient. HDFS is designed for parallel reads.
- **(D)** All tasks within a stage run in **parallel** across available cores. Sequential execution would only occur between stages (stage dependencies).

</details>

---

**Q76.** What does `spark.sql.adaptive.enabled = true` (AQE) do, and what are its three main optimizations?

- A) AQE enables asynchronous query execution — queries run in the background without blocking the Driver.
- B) **Adaptive Query Execution (AQE)** re-optimizes query plans at **runtime** using actual shuffle statistics (not just estimates). Its three main optimizations are: (1) **dynamically coalescing shuffle partitions** to reduce the number of post-shuffle tasks, (2) **switching join strategies** (e.g., converting a planned Sort-Merge Join to Broadcast Hash Join when runtime statistics show a table is smaller than estimated), and (3) **skew join optimization** (automatically splitting skewed partitions).
- C) AQE enables multi-threaded execution within a single task.
- D) AQE is only available in Spark 4.0+ and requires a separate installation.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

**AQE** (default enabled in Spark 3.2+) is one of the most impactful recent Spark features. The three key AQE features:

**1. Dynamic Partition Coalescing:**
```
Before: 200 shuffle partitions (spark.sql.shuffle.partitions)
AQE measures: most partitions are tiny (< 64MB)
After: merges 200 partitions into 20 larger ones automatically
Result: fewer, more efficient downstream tasks
```

**2. Dynamic Join Strategy Switching:**
```
Optimizer estimates: Table B = 500MB → plans SortMergeJoin
Runtime reality: Table B after filter = 8MB
AQE detects: 8MB < autoBroadcastJoinThreshold (10MB)
AQE switches: SortMergeJoin → BroadcastHashJoin automatically
Result: no shuffle on the large table
```

**3. Dynamic Skew Join Optimization:**
```
AQE detects: partition #42 is 10× larger than median
AQE splits: partition #42 into 10 sub-partitions
AQE duplicates: corresponding right-side partition 10×
Result: skew handled without manual salting
```

```python
# Enable AQE (default in Spark 3.2+)
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
```

**Why the others are wrong:**
- **(A)** "Adaptive" refers to runtime plan adaptation, not asynchronous execution. Queries still block unless you use async patterns (`df.writeStream`, threading).
- **(C)** Task-level parallelism is controlled by the Executor model, not AQE. Each task still runs on one core.
- **(D)** AQE was introduced in Spark 3.0 (experimental) and enabled by default in Spark 3.2. It requires no separate installation.

</details>

---

## Section 29: Error Handling & Fault Tolerance

---

**Q77.** A Spark Structured Streaming job processes a Kafka topic with 100 partitions. A message arrives with **malformed JSON** and the UDF throws an exception. What happens by default, and how should you handle this gracefully?

```python
from pyspark.sql.functions import from_json, col
from pyspark.sql.types import StructType

schema = StructType().add("user_id", "string").add("amount", "double")

df = spark \
    .readStream.format("kafka") \
    .option("subscribe", "payments") \
    .load() \
    .select(from_json(col("value").cast("string"), schema).alias("data")) \
    .select("data.*")
```

- A) The entire streaming job crashes and requires manual restart.
- B) `from_json()` handles malformed JSON **gracefully by default** — it returns `null` for unparseable records rather than throwing an exception. The malformed record results in a null row, which you should detect and route to a dead-letter sink for investigation.
- C) The malformed record is automatically skipped and logged to `spark.eventLog`.
- D) Spark retries the malformed record 3 times before skipping it.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

`from_json()` is intentionally fault-tolerant — it returns **null** for records that don't match the schema, rather than failing. This is the correct behavior for streaming: you don't want one bad message to crash a job processing millions of messages.

**Handling nulls from `from_json()` — Dead Letter Queue pattern:**

```python
# Parse JSON
parsed = raw_df.select(
    col("value").cast("string").alias("raw"),
    from_json(col("value").cast("string"), schema).alias("data")
)

# Split: valid records and dead letters
valid_records = parsed.filter(col("data").isNotNull()).select("data.*")
dead_letters = parsed.filter(col("data").isNull()).select("raw")

# Route valid records to main sink
valid_records.writeStream.format("delta").start("s3://output/events/")

# Route dead letters for investigation
dead_letters.writeStream.format("delta").start("s3://output/dead_letter/")
```

**`from_json()` column modes** (for more control):
```python
# PERMISSIVE (default): nullify entire record on bad JSON
# DROPMALFORMED: drop the record (use .option("mode", "DROPMALFORMED"))
# FAILFAST: throw exception on bad JSON
from_json(col("value"), schema, {"mode": "PERMISSIVE"})
```

**Why the others are wrong:**
- **(A)** `from_json()` is designed not to crash on malformed input. Python UDF exceptions *would* crash a task, but `from_json()` is a native Catalyst function with built-in error handling.
- **(C)** Spark's event log (`spark.eventLog`) captures job metrics and events, not individual malformed records. Record-level logging is the application's responsibility.
- **(D)** Spark does retry failed tasks (3 times by default via `spark.task.maxFailures`), but `from_json()` returning null is not a failure — it's expected behavior. No retry occurs.

</details>

---

**Q78.** A data engineer sees this error: `org.apache.spark.shuffle.FetchFailedException`. What does it mean and how do you fix it?

- A) The job tried to read a file from S3 that doesn't exist.
- B) During the **reduce phase of a shuffle**, an Executor tried to **fetch shuffle blocks** from another Executor that had **died or become unresponsive**. The map-side Executor's shuffle files are gone. Spark automatically retries the stage, but if the Executor keeps dying (OOM, node failure), the job eventually fails. Fix: increase Executor memory, enable External Shuffle Service, or reduce shuffle data size.
- C) A network timeout occurred while reading from the Kafka cluster.
- D) The `fetch()` method was called on a future that timed out in the driver program.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

`FetchFailedException` is the classic shuffle failure signature in Spark:

```
Stage 2 (reduce stage) needs shuffle data from Stage 1 (map stage)
Stage 1's Executor dies (OOM, node failure, GC overhead limit)
Stage 2's tasks attempt to fetch shuffle blocks from dead Executor
FetchFailedException: Unable to fetch blocks from dead Executor
Spark retries: re-runs Stage 1 on surviving Executors
If Executor keeps dying: FetchFailed again → job eventually fails
```

**Diagnosis checklist:**
1. Check Driver logs for `ExecutorLostFailure` preceding the FetchFailed
2. Check YARN/Kubernetes for node failures, OOM kills
3. Check Executor GC logs for `GC overhead limit exceeded`

**Fixes:**
```python
# 1. More Executor memory
spark.conf.set("spark.executor.memory", "8g")
spark.conf.set("spark.executor.memoryOverhead", "2g")  # Off-heap for native/Python memory

# 2. External Shuffle Service (ESS) — persists shuffle files even if Executor dies
# Configured in spark-defaults.conf:
# spark.shuffle.service.enabled = true

# 3. Reduce shuffle data
spark.conf.set("spark.sql.shuffle.partitions", "400")  # Smaller per-partition size

# 4. Enable speculation to relaunch stragglers before they OOM
spark.conf.set("spark.speculation", "true")
```

**Why the others are wrong:**
- **(A)** Missing S3 files cause `FileNotFoundException` or `AnalysisException`, not `FetchFailedException`. The fetch in `FetchFailedException` refers to shuffle block fetching, not file I/O.
- **(C)** Kafka read failures produce Kafka-specific exceptions, not `FetchFailedException`. The shuffle fetch is Spark-internal communication between Executors.
- **(D)** `FetchFailedException` is a JVM-level exception in Spark's shuffle subsystem. It has nothing to do with Scala/Python `Future.fetch()` calls.

</details>

---

## Section 30: Advanced Aggregations & Performance

---

**Q79.** What is the difference between `groupBy().agg()` and `groupBy().apply()` (Grouped Map Pandas UDF), and when should you use each?

- A) They are functionally identical — `apply()` is just a newer syntax.
- B) `groupBy().agg()` uses **built-in Catalyst aggregation functions** (sum, count, avg, etc.) entirely in the JVM — highly optimized. `groupBy().apply()` (Grouped Map Pandas UDF) sends each **group as a pandas DataFrame** to a Python function — enables arbitrary Python logic per group, but pays the Arrow serialization cost and bypasses Catalyst. Use `agg()` for standard aggregations; use `apply()` for complex per-group operations like custom ML inference, time-series fitting, or business logic that can't be expressed in Spark SQL.
- C) `apply()` can only be used when each group fits in Executor memory.
- D) `groupBy().agg()` only supports one aggregation at a time; `apply()` supports multiple.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

```python
# groupBy().agg() — native Catalyst, always prefer when possible
df.groupBy("store_id").agg(
    F.sum("revenue").alias("total_revenue"),
    F.avg("items_sold").alias("avg_items"),
    F.count("*").alias("transaction_count")
)

# groupBy().apply() — Grouped Map Pandas UDF
# Use when native agg() can't express the logic
from pyspark.sql.functions import pandas_udf, PandasUDFType

output_schema = "store_id string, forecast double, model_r2 double"

@pandas_udf(output_schema, PandasUDFType.GROUPED_MAP)
def fit_forecast_model(pdf: pd.DataFrame) -> pd.DataFrame:
    # Train a scikit-learn model per store
    model = LinearRegression().fit(pdf[["day_of_week", "holiday"]], pdf["revenue"])
    pdf["forecast"] = model.predict(pdf[["day_of_week", "holiday"]])
    pdf["model_r2"] = model.score(pdf[["day_of_week", "holiday"]], pdf["revenue"])
    return pdf[["store_id", "forecast", "model_r2"]]

df.groupBy("store_id").apply(fit_forecast_model)
```

**Performance comparison:**
- `agg()`: JVM, vectorized, Catalyst-optimized — fastest for supported operations
- `apply()`: Arrow serialization + Python execution — slower, but enables Python ecosystem

**Why the others are wrong:**
- **(A)** They are fundamentally different execution models. `agg()` is JVM-native; `apply()` crosses the JVM-Python boundary with full group data.
- **(C)** Each group does need to fit in a single Executor task's memory (since the whole group is sent to one Python process). This is a real constraint of Grouped Map Pandas UDFs — but it's not the defining difference between them.
- **(D)** `groupBy().agg()` supports multiple simultaneous aggregations: `agg(sum("a"), avg("b"), count("c"))`. This is one of its strengths.

</details>

---

**Q80.** A data engineer writes:

```python
df = spark.read.parquet("s3://events/")
df2 = df.filter(col("country") == "US")
df3 = df.filter(col("country") == "UK")
df4 = df2.union(df3)
df4.write.parquet("s3://output/")
```

A colleague points out this is **inefficient**. Why, and what is the correct rewrite?

- A) `union()` is not supported in PySpark — use `unionAll()` instead.
- B) The code reads the Parquet source **twice** — once for the US filter and once for the UK filter. Since there's no `cache()`, Spark's optimizer treats `df2` and `df3` as independent plans both rooted at the same Parquet read. Fix: either `cache()` the base `df`, or rewrite as a single `isin()` filter.
- C) `union()` doesn't deduplicate rows — use `union().distinct()` to remove duplicates.
- D) Filtering before `union()` is less efficient than unioning first and filtering after.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

Spark's DAG for the original code:

```
Read Parquet ──→ filter(US) ──┐
                               ├──→ union ──→ write
Read Parquet ──→ filter(UK) ──┘
```

Two separate reads of the same Parquet source — double the I/O, double the cost.

**Fix 1 — Single filter with `isin()` (best for this case):**
```python
df = spark.read.parquet("s3://events/")
df.filter(col("country").isin("US", "UK")) \
  .write.parquet("s3://output/")
```

**Fix 2 — Cache the base DataFrame:**
```python
df = spark.read.parquet("s3://events/").cache()
df2 = df.filter(col("country") == "US")
df3 = df.filter(col("country") == "UK")
df2.union(df3).write.parquet("s3://output/")
df.unpersist()
```

**Fix 3 — If filters are complex and unrelated, cache is the right tool:**
```python
# When you genuinely need two different subsets for different purposes
df = spark.read.parquet("s3://events/").cache()
us_agg = df.filter(col("country") == "US").groupBy("product").agg(sum("revenue"))
uk_agg = df.filter(col("country") == "UK").groupBy("product").agg(avg("revenue"))
# ... two separate write operations using the cached base
```

**Why the others are wrong:**
- **(A)** `union()` is valid PySpark. `unionAll()` was the old name (Spark 1.x) and is an alias for `union()` in modern Spark. Both keep duplicates.
- **(C)** Whether to deduplicate depends on requirements. For this use case (combining US and UK events), there are no cross-country duplicates, so `.distinct()` would be unnecessary overhead.
- **(D)** Filtering before union is *more* efficient — you process less data through the union. The problem here is the duplicate reads, not the filter position.

</details>

---

## 📊 Score Tracker — Part 4

| Section | Questions | Your Score |
|---|---|---|
| UDFs — Performance & Serialization | Q63 – Q64 | &nbsp;&nbsp;&nbsp;/ 2 |
| Memory Management & Spilling | Q65 – Q66 | &nbsp;&nbsp;&nbsp;/ 2 |
| Delta Lake Internals | Q67 – Q69 | &nbsp;&nbsp;&nbsp;/ 3 |
| Window Functions — Advanced | Q70 – Q72 | &nbsp;&nbsp;&nbsp;/ 3 |
| File Formats & I/O Optimization | Q73 – Q74 | &nbsp;&nbsp;&nbsp;/ 2 |
| Job Tuning & Cluster Config | Q75 – Q76 | &nbsp;&nbsp;&nbsp;/ 2 |
| Error Handling & Fault Tolerance | Q77 – Q78 | &nbsp;&nbsp;&nbsp;/ 2 |
| Advanced Aggregations | Q79 – Q80 | &nbsp;&nbsp;&nbsp;/ 2 |
| **Total** | **Q63 – Q80** | **&nbsp;&nbsp;&nbsp;/ 18** |

---

## 🧠 Key Concepts Cheat Sheet — Part 4

| Concept | One-Line Summary |
|---|---|
| **Python UDF overhead** | Row-by-row JVM↔Python serialization via Py4J — kills vectorization and Catalyst |
| **Pandas UDF (Vectorized UDF)** | Uses Apache Arrow for batch column transfer — 10–100× faster than row UDFs |
| **Spilling** | Shuffle data that overflowed Executor heap and wrote to local disk — severe slowdown |
| **Unified Memory Model** | Execution and Storage share one pool; Execution can evict cached Storage blocks |
| **Delta `_delta_log`** | JSON commit log of all operations — backbone of ACID, time travel, and schema history |
| **`OPTIMIZE` + `ZORDER`** | Compact small files into large ones; Z-order co-locates related data for better skipping |
| **`VACUUM`** | Physically deletes tombstoned files — required to reclaim storage after DELETEs/UPDATEs |
| **Copy-on-Write (CoW)** | UPDATE/DELETE rewrites affected Parquet files; old files kept for time travel until VACUUMed |
| **Window reset with partitionBy** | Reset running totals per period by including the period column in `partitionBy` |
| **`lag` vs `lead`** | `lag` = look back N rows; `lead` = look forward N rows — both return null at partition boundary |
| **`dense_rank` vs `rank` vs `row_number`** | dense_rank: no gaps; rank: gaps after ties; row_number: always unique sequential |
| **Small file problem** | Too many tiny files = high file-open latency; fix with `coalesce()` before write or `OPTIMIZE` |
| **Parquet column pruning** | Columnar storage lets Spark read only selected columns' byte ranges — skip irrelevant columns |
| **Task wave analysis** | Job time ≈ ceil(tasks/cores) × avg_task_time; excess time indicates stragglers |
| **AQE (Adaptive Query Execution)** | Runtime re-optimization: dynamic partition coalescing, join switching, skew splitting |
| **`FetchFailedException`** | Reduce-side Executor couldn't fetch shuffle blocks from a dead map-side Executor |
| **`from_json()` null on bad input** | Returns null for malformed JSON — use dead-letter queue pattern to capture bad records |
| **`groupBy().apply()` Pandas UDF** | Sends full group as pandas DataFrame to Python — enables arbitrary per-group logic |
| **Double-read anti-pattern** | Two filters on same un-cached DataFrame = two full source scans; fix with `isin()` or `cache()` |
| **`spark.conf.set()` at runtime** | Tune per-job settings (shuffle partitions, broadcast threshold) between sequential pipeline steps |

---

*This is Part 4 of the PySpark Certification Practice Series. Combined with Parts 1–3, you now have 80 scenario-based questions covering the full certification syllabus. 🚀*
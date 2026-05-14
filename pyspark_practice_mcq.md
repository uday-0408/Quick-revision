# 🔥 PySpark Certification Practice Test
### Beginner to Intermediate | Logic & Concept Focused

> **Instructions:** Read each scenario carefully and select the best answer before expanding the hidden answer block. Each explanation covers *why* the correct answer is right and *why* the other options are wrong.

---

## Section 1: Spark Architecture — Driver, Executors & Cluster Manager

---

**Q1.** A junior data engineer runs a PySpark job on a YARN cluster. The job fails with an `OutOfMemoryError` on a specific node, but the Spark UI shows the Driver process is healthy. Which component most likely caused the failure, and what is its primary responsibility?

- A) The Cluster Manager — it ran out of memory while scheduling tasks across nodes.
- B) An Executor — it ran out of memory while processing or caching partition data assigned to it.
- C) The Driver — it ran out of memory while collecting results from all worker nodes.
- D) The Worker Node OS — it killed the process because the JVM heap was misconfigured at the cluster level.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

**Executors** are the JVM processes launched on Worker Nodes that are responsible for executing the actual Tasks (i.e., processing data partitions) and storing cached data. An `OutOfMemoryError` on a worker node, with a healthy Driver, is a classic symptom of an Executor running out of heap space — most commonly caused by a skewed partition holding too much data or by aggressive in-memory caching.

**Why the others are wrong:**
- **(A)** The Cluster Manager (YARN ResourceManager in this case) handles resource negotiation and container allocation — it does not process data and has minimal memory footprint.
- **(C)** If the Driver had OOM'd, the entire application would have died and the Spark UI would be unreachable, not "healthy."
- **(D)** While the OS can kill JVM processes, the question asks about the *Spark component* responsible for data processing on that node — which is the Executor.

</details>

---

**Q2.** In a Spark application running on a cluster, your PySpark script contains this line:

```python
results = df.filter(col("status") == "active").collect()
```

Which component is responsible for *executing the plan* across the cluster, and which component *receives the final `results` list*?

- A) Executors execute the plan; the Cluster Manager receives the results.
- B) The Driver executes the plan; Executors receive the results.
- C) Executors execute the plan; the Driver receives the results.
- D) Executors execute the plan; results are written directly to HDFS without returning to the Driver.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

**Executors** do all the data processing work — they filter partitions in parallel across the cluster. The `.collect()` action then pulls all resulting rows back to the **Driver** program, where they are materialized as a Python list assigned to `results`. This is also why `.collect()` on a massive dataset is dangerous: it can OOM the Driver by pulling terabytes of data into a single process.

**Why the others are wrong:**
- **(A)** The Cluster Manager is an infrastructure coordinator (YARN, Mesos, Kubernetes). It never touches data.
- **(B)** The Driver *plans* and *orchestrates* the job (building the DAG, scheduling stages), but it does not process the partitions — Executors do.
- **(D)** `.collect()` specifically means "bring data back to the Driver." If you wanted to write to HDFS, you'd use `.write.save()`.

</details>

---

**Q3.** Your team is deciding between running Spark in *local mode* versus *cluster mode*. A colleague claims: *"In local mode, we still get full parallelism because Spark spins up multiple Executor JVMs on the same machine."* Is this claim correct?

- A) Yes — local mode spawns multiple Executor processes, one per CPU core.
- B) No — local mode runs everything (Driver + Executor logic) inside a **single JVM process**, using threads for parallelism.
- C) Yes — local mode is identical to cluster mode, except the Cluster Manager is skipped.
- D) No — local mode runs only the Driver; no Executor is created, so no data processing is possible.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

In **local mode** (`local[*]` or `local[N]`), the entire Spark application — Driver and Executor logic — runs inside a **single JVM** on your machine, using threads to simulate parallelism. There are no separate Executor processes or Worker Nodes. This is why local mode is great for development and testing but doesn't reflect true distributed behavior.

**Why the others are wrong:**
- **(A)** Multiple threads are used, not multiple Executor JVM *processes*. The isolation and fault tolerance of a real cluster does not exist.
- **(C)** Cluster mode has fundamentally different behavior: separate JVMs for Driver/Executors, network I/O between nodes, and a Cluster Manager. Local mode is a single process simulation.
- **(D)** Data processing absolutely happens — but it happens within the single JVM using threaded tasks, not distributed Executors.

</details>

---

## Section 2: Execution Hierarchy — Jobs, Stages, Tasks & Partitions

---

**Q4.** A data engineer runs the following PySpark code:

```python
df = spark.read.parquet("s3://data/transactions/")
df_filtered = df.filter(col("amount") > 1000)
df_grouped = df_filtered.groupBy("region").agg(sum("amount").alias("total"))
df_grouped.write.parquet("s3://output/")
```

How many **Jobs** will Spark most likely create for this entire pipeline?

- A) 4 Jobs — one for each line of code.
- B) 1 Job — because there is only one Action (`.write`).
- C) 2 Jobs — Spark always splits reads and writes into separate jobs.
- D) 3 Jobs — one for reading, one for grouping, and one for writing.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

In Spark, a **Job** is triggered by an **Action** (e.g., `.collect()`, `.write()`, `.count()`). Transformations like `.filter()` and `.groupBy().agg()` are **lazy** — they build up the execution plan (DAG) but don't execute. Since there is only **one Action** here (`.write.parquet()`), Spark creates **one Job**. That one job may internally be split into multiple **Stages** due to the shuffle caused by `groupBy`.

**Why the others are wrong:**
- **(A)** Lines of code do not map to Jobs. Transformations are lazy and don't trigger execution on their own.
- **(C)** Spark does not automatically split reads and writes into separate jobs — a single write action drives the whole pipeline.
- **(D)** `groupBy` is a transformation, not an action. Only actions create Jobs.

</details>

---

**Q5.** In the pipeline from Q4, Spark internally splits the Job into **Stages**. What determines the boundary between Stage 1 and Stage 2?

- A) The `.filter()` operation — filters always create a new stage.
- B) The `.read.parquet()` operation — reading from cloud storage starts a new stage.
- C) The `groupBy().agg()` operation — it requires a **shuffle**, which forces a stage boundary.
- D) The `.write.parquet()` operation — writing to storage always forces a stage boundary.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

**Stage boundaries** are created wherever Spark needs to **shuffle data** across the network — i.e., at **wide transformations**. `groupBy()` requires all rows with the same key to be moved to the same partition (a shuffle), so Spark must materialize the shuffle output before Stage 2 can begin. Stage 1 handles reading and filtering; Stage 2 handles the post-shuffle aggregation and write.

**Why the others are wrong:**
- **(A)** `.filter()` is a **narrow transformation** — it can be applied to each partition independently with no data movement, so it does NOT cause a stage boundary.
- **(B)** Reading data initializes the first stage; it is not a boundary between stages.
- **(D)** The write operation lives within the final stage — it is the action that *triggers* the job, but the write itself doesn't create an additional stage boundary.

</details>

---

**Q6.** Your Spark job reads a Parquet file that has **200 partitions**. After a `groupBy` shuffle, Spark writes the shuffled output into a new set of partitions. How many Tasks will run in the **post-shuffle stage** by default, and what configuration controls this?

- A) 200 Tasks — Spark always preserves the original partition count after a shuffle.
- B) 1 Task — shuffles collapse all data into a single partition for aggregation.
- C) **200 Tasks** — controlled by `spark.default.parallelism`.
- D) **200 Tasks** — controlled by `spark.sql.shuffle.partitions`, which defaults to 200.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: D**

After a shuffle (wide transformation), Spark creates a new set of partitions for the downstream stage. For **Spark SQL and DataFrame operations**, the number of these post-shuffle partitions is controlled by `spark.sql.shuffle.partitions`, which **defaults to 200**. Each post-shuffle partition becomes one **Task** in the new stage. This default is famously a common performance tuning target — 200 is often too many for small datasets and too few for very large ones.

**Why the others are wrong:**
- **(A)** The original partition count (from the file read) only determines the *pre-shuffle* stage tasks. The shuffle redefines the partition count using `spark.sql.shuffle.partitions`.
- **(B)** Shuffles distribute data into multiple partitions by key — not into one.
- **(C)** `spark.default.parallelism` applies to **RDD operations** (e.g., `sc.parallelize`, `rdd.reduceByKey`), not DataFrame/SQL shuffle operations.

</details>

---

## Section 3: Lazy Evaluation & the DAG

---

**Q7.** A data engineer writes the following code and is confused why it runs instantly:

```python
df = spark.read.csv("s3://huge-bucket/10TB-file.csv", header=True)
df2 = df.filter(col("country") == "IN")
df3 = df2.withColumn("tax", col("price") * 0.18)
print("Pipeline defined!")
```

Why does this code execute immediately without any noticeable delay?

- A) Spark is very fast and has already processed the 10TB file in-memory.
- B) PySpark is lazily evaluated — these are all Transformations, not Actions. Spark only builds the logical DAG and defers actual execution.
- C) The filter `col("country") == "IN"` caused Spark to skip most of the file, so very little data was read.
- D) The CSV file was cached from a previous run, so Spark read it from memory.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

This is the core principle of **lazy evaluation**. `.read.csv()`, `.filter()`, and `.withColumn()` are all **transformations** — they instruct Spark on *what* to do but don't trigger any actual execution. Spark records these operations as nodes in the **DAG (Directed Acyclic Graph)** and waits for an **Action** (like `.count()`, `.show()`, `.collect()`, or `.write()`) to submit the job to the cluster. No data is read from S3 until an action is called.

**Why the others are wrong:**
- **(A)** No data processing has occurred whatsoever. Processing 10TB instantly would also be physically impossible.
- **(C)** Predicate pushdown (skipping data at the source) is an optimization that only applies *during actual execution*, which hasn't been triggered.
- **(D)** Caching is explicit and manual in Spark (`.cache()` or `.persist()`), and even then, the cache is populated on first action execution.

</details>

---

**Q8.** A senior engineer reviews your code and says: *"Your DAG is doing redundant work — you're recomputing the same expensive transformation three times."* Which of the following code patterns is she most likely pointing at?

```python
# Pattern A
df_clean = raw_df.filter(...).dropDuplicates().withColumn(...)
count = df_clean.count()
sample = df_clean.limit(10).collect()
schema = df_clean.schema

# Pattern B
df_clean = raw_df.filter(...).dropDuplicates().withColumn(...).cache()
count = df_clean.count()
sample = df_clean.limit(10).collect()
```

- A) Pattern B — `.cache()` causes the DataFrame to be computed three times to populate the cache.
- B) Pattern A — because `df_clean` is used three times without caching, Spark will **re-execute the full DAG** from the source for each action.
- C) Pattern A — `.schema` is an action that triggers a full recomputation.
- D) Both patterns are equivalent — Spark automatically caches DataFrames used more than once.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

In **Pattern A**, `df_clean` is a lazily defined transformation chain. Every time an **Action** is called on it (`.count()`, `.collect()`), Spark walks the entire DAG from the original source and recomputes `filter → dropDuplicates → withColumn`. For three actions, the expensive `dropDuplicates` is executed three times. **Pattern B** correctly uses `.cache()` so that after the first action (`.count()`) populates the cache, subsequent actions (`limit(10).collect()`) read from memory.

**Why the others are wrong:**
- **(A)** `.cache()` marks the DataFrame to be stored in memory *after* it is first computed — it does not cause triple computation. The first action populates the cache; subsequent actions use it.
- **(C)** Accessing `.schema` does **not** trigger a full job — Spark can often infer schema from metadata (e.g., Parquet file footers) without reading data.
- **(D)** Spark does **not** auto-cache. Caching is always explicit.

</details>

---

**Q9.** Which of the following is an **Action** in PySpark, and why does it matter?

- A) `df.select("name", "age")` — it selects specific columns.
- B) `df.filter(col("age") > 30)` — it filters rows from the DataFrame.
- C) `df.write.parquet("s3://output/")` — it triggers execution and materializes data to storage.
- D) `df.groupBy("dept").agg(count("*"))` — it aggregates data, which requires computation.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

An **Action** is any operation that triggers the Spark job scheduler to execute the accumulated DAG of transformations and produce a result — either to the Driver, to storage, or to a display. `.write.parquet()` is a classic write action that forces Spark to compute and materialize the entire pipeline to disk. It matters because this is the moment Spark stops planning and starts doing work.

**Why the others are wrong:**
- **(A)** `.select()` is a **transformation** — it adds a projection node to the DAG but does not execute.
- **(B)** `.filter()` is a **transformation** — it adds a filter predicate to the DAG lazily.
- **(D)** `.groupBy().agg()` is a **transformation chain** — it defines the aggregation logic and shuffle in the DAG but does not execute until an action is called on it.

</details>

---

## Section 4: Shuffling, Wide vs. Narrow Transformations & Network I/O

---

**Q10.** A data pipeline processes 500GB of raw event logs. A colleague proposes two approaches:

- **Approach X:** `df.filter(col("event_type") == "purchase").select("user_id", "amount")`
- **Approach Y:** `df.groupBy("user_id").agg(sum("amount").alias("total_spend"))`

Which approach involves a **shuffle**, and what is the concrete performance implication?

- A) Both involve a shuffle — any operation that touches multiple columns causes network I/O.
- B) Approach X involves a shuffle because filtering requires comparing rows across partitions.
- C) Approach Y involves a shuffle because `groupBy` is a **wide transformation** that must redistribute all rows by key across the network before aggregation.
- D) Neither involves a shuffle — Spark processes each partition independently for both operations.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

`groupBy` is a **wide transformation**: to correctly sum `amount` per `user_id`, every row belonging to the same `user_id` must be co-located on the same partition. Since user events are spread randomly across 500GB of input partitions, Spark must **shuffle** — serializing rows, transferring them over the network to the correct partition, and deserializing them. This is expensive: it causes disk I/O (shuffle write/read), network I/O, and serialization overhead.

**Why the others are wrong:**
- **(A)** Column count has nothing to do with shuffling. Shuffles are caused by needing to move rows *between* partitions.
- **(B)** Filtering is a **narrow transformation** — each output partition depends only on one input partition. No row ever needs to move to a different partition.
- **(D)** Approach Y absolutely involves a shuffle. Approach X does not, but they are not equivalent in this regard.

</details>

---

**Q11.** Your Spark SQL job runs a `groupBy` aggregation on a dataset that is only **50MB** after filtering. The job runs slowly, and the Spark UI shows **200 tiny tasks** in the shuffle stage, each processing only ~250KB. What is the most likely cause, and what is the fix?

- A) The dataset is corrupted. Re-read the source data.
- B) The default value of `spark.sql.shuffle.partitions` is 200, which is too high for a 50MB post-filter dataset. Reduce it to a smaller number (e.g., 5-10).
- C) Increase the number of Executors — 200 tasks need at least 200 CPU cores to run efficiently.
- D) The `groupBy` key has high cardinality. Switch to `distinct()` instead.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

`spark.sql.shuffle.partitions` defaults to **200**, which was designed for large-scale workloads. For a small 50MB dataset, creating 200 shuffle partitions means each task is only processing ~250KB of data — the overhead of scheduling, launching, and serializing for each task far exceeds the actual compute work. Reducing `spark.sql.shuffle.partitions` to a small value (e.g., `spark.conf.set("spark.sql.shuffle.partitions", "5")`) eliminates this "tiny task" anti-pattern.

**Why the others are wrong:**
- **(A)** Data corruption would cause errors or incorrect results, not just slow tasks.
- **(C)** Adding more Executors to handle more tiny tasks makes the problem *worse*, not better — you're adding infrastructure overhead to an already over-partitioned job.
- **(D)** `distinct()` is not a substitute for `groupBy` aggregations and would produce completely different (and wrong) output.

</details>

---

**Q12.** Classify the following PySpark operations as either **Narrow** or **Wide** transformations:

1. `df.withColumn("new_col", col("a") + col("b"))`
2. `df.join(other_df, on="id", how="inner")`
3. `df.map(lambda x: x)` *(on an RDD)*
4. `df.repartition(50)`

- A) All four are Narrow transformations.
- B) 1 and 3 are Narrow; 2 and 4 are Wide.
- C) 1, 2, and 3 are Narrow; 4 is Wide.
- D) Only 2 is Wide; the rest are Narrow.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

A **narrow transformation** is one where each output partition depends on at most one input partition — no shuffle required. A **wide transformation** requires data from multiple partitions to be combined, causing a shuffle.

- `withColumn` — Narrow: the new column is computed row-by-row within each partition.
- `join` — **Wide**: rows must be redistributed by key to co-locate matching rows across the two DataFrames.
- `map` (RDD) — Narrow: each element is transformed independently within its partition.
- `repartition(50)` — **Wide**: data is redistributed randomly across the network to create exactly 50 new partitions (full shuffle). Note: `coalesce()` is narrow because it only *merges* existing partitions without a full shuffle.

</details>

---

## Section 5: Optimization — Broadcast Joins, Shuffle Sort-Merge Joins & Caching

---

**Q13.** You are joining a **2TB transactions table** with a **5MB lookup table** of country codes. Your colleague writes:

```python
result = transactions_df.join(country_df, on="country_code", how="left")
```

The job runs for 45 minutes. A senior engineer suggests adding one hint. What is it, and why does it work?

- A) `transactions_df.repartition(2000).join(country_df, ...)` — more partitions speed up the join.
- B) `transactions_df.join(broadcast(country_df), ...)` — the small table is sent to every Executor, eliminating the shuffle entirely.
- C) `country_df.repartition(1).join(transactions_df, ...)` — consolidating the small table reduces network traffic.
- D) Set `spark.sql.shuffle.partitions=2000` — more shuffle partitions improve join performance on large tables.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

A **Broadcast Hash Join** works by sending the *entire small table* (5MB) to every Executor node, where it is held in memory as a hash map. Each Executor can then probe this hash map locally for every row in its partition of the large table — **no shuffle of the large table is needed at all**. This converts an expensive wide transformation (requiring 2TB of data to be shuffled) into a cheap narrow one. Spark may even do this automatically if the table is under `spark.sql.autoBroadcastJoinThreshold` (default: 10MB).

**Why the others are wrong:**
- **(A)** More partitions on the large table still require a full shuffle of both sides for the join — it doesn't eliminate the shuffle.
- **(C)** Consolidating the small table to 1 partition makes the problem *worse* — everything would funnel to a single task.
- **(D)** More shuffle partitions reduce partition size but don't eliminate the shuffle of 2TB of data, which is the root cause of the slowness.

</details>

---

**Q14.** Spark chooses between a **Broadcast Hash Join (BHJ)** and a **Shuffle Sort-Merge Join (SMJ)** based on table sizes. For two tables that are **both large** (e.g., 500GB each), which join strategy does Spark use and why?

- A) BHJ — Spark always prefers BHJ because it avoids shuffling.
- B) SMJ — When neither table fits in Executor memory for broadcasting, Spark shuffles both tables by join key, sorts them, and merges matching rows from each sorted partition.
- C) SMJ — Spark sorts both tables globally before joining, which requires only one pass over the data.
- D) BHJ — Spark uses disk-based broadcasting to handle tables larger than memory.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

When both tables are too large to broadcast, Spark falls back to **Shuffle Sort-Merge Join**: it shuffles *both* DataFrames by the join key (so matching keys land on the same partition), sorts each partition, and then linearly merges the two sorted streams. This is more expensive than BHJ (requires shuffling two large datasets), but it's scalable to any data size since it works partition-by-partition without requiring either table to fit in memory.

**Why the others are wrong:**
- **(A)** BHJ is only viable when one table is small enough to fit in Executor memory (controlled by `spark.sql.autoBroadcastJoinThreshold`). You cannot broadcast a 500GB table.
- **(C)** SMJ does *not* sort tables globally — it shuffles data by key so each partition contains all rows for a subset of keys, then sorts within each partition. A global sort would be prohibitively expensive.
- **(D)** There is no disk-based broadcasting in standard Spark. Broadcasting is an in-memory operation.

</details>

---

**Q15.** Your pipeline reads a large raw dataset, applies expensive transformations, and then uses the result **four times** in downstream processing: for writing to a data lake, sending to a report, a quality check count, and a sample preview.

```python
df_transformed = raw_df.filter(...).withColumn(...).dropDuplicates()

df_transformed.write.parquet("s3://datalake/")
df_transformed.write.parquet("s3://reports/")
quality_count = df_transformed.count()
preview = df_transformed.limit(5).show()
```

What is the **most serious performance problem** here, and what is the correct fix?

- A) Using `.show()` after `.limit()` is redundant. Replace with `.collect()`.
- B) Without `.cache()`, the full transformation chain (filter → withColumn → dropDuplicates) is **re-executed from scratch four times**. Add `df_transformed.cache()` before the first action.
- C) `dropDuplicates` is not compatible with `.write` operations. Use `.distinct()` instead.
- D) Writing to two different S3 paths in sequence causes I/O contention. Write both with a single `.write` call.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

Because `df_transformed` is a lazily-defined transformation chain with **no cache**, every Action (each `.write`, `.count()`, `.show()`) re-triggers the full DAG from `raw_df` all the way through `filter → withColumn → dropDuplicates`. `dropDuplicates` is particularly expensive as it involves a shuffle. By calling `df_transformed.cache()` before the first action, Spark stores the result in Executor memory after the first computation, and all three subsequent actions read from the cache instead of recomputing.

**Why the others are wrong:**
- **(A)** `.limit(5).show()` is a perfectly valid and common pattern. The real problem is the lack of caching, not this pattern.
- **(C)** `dropDuplicates()` and `.distinct()` are functionally identical in Spark; both work with `.write`.
- **(D)** Two `.write` calls to different paths are a normal and valid pattern — S3 I/O contention is not a concern here and is not the primary performance problem.

</details>

---

**Q16.** A data engineer needs to persist a DataFrame that will be reused 10+ times during a long-running job. The dataset is **8GB** and the total Executor memory across the cluster is **20GB**. Which persistence level is the **safest** choice, and why?

- A) `MEMORY_ONLY` — always use this; disk I/O should always be avoided.
- B) `DISK_ONLY` — persisting to disk guarantees the data survives Executor failures.
- C) `MEMORY_AND_DISK` — if the data doesn't fit entirely in memory, Spark spills overflow partitions to disk instead of recomputing them, balancing speed and reliability.
- D) `MEMORY_ONLY_SER` — serialized storage is always faster than deserialized storage.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

`MEMORY_AND_DISK` is the pragmatic choice here: Spark tries to keep partitions in memory for fast access, but when memory pressure builds up (other DataFrames, execution overhead also consume memory), it **spills excess partitions to disk** rather than dropping them. This prevents the silent fallback behavior of `MEMORY_ONLY` (dropping evicted partitions and recomputing them on next access), which could be very costly for a 10-access pattern. It is the default behavior of `.cache()`.

**Why the others are wrong:**
- **(A)** `MEMORY_ONLY` sounds ideal, but if the 8GB dataset doesn't cleanly fit (after accounting for Executor overhead, other cached objects, etc.), Spark will **silently evict and recompute** partitions — which defeats the purpose of caching.
- **(B)** `DISK_ONLY` gives up the entire speed benefit of in-memory caching. You'd read from disk every single time, which may be slower than recomputation for some workloads.
- **(D)** `MEMORY_ONLY_SER` stores data as serialized bytes, which reduces the memory *footprint* at the cost of CPU deserialization on every access. It is not universally faster — and it still has the same eviction-and-recompute risk as `MEMORY_ONLY`.

</details>

---

**Q17.** Which of the following correctly describes the **difference between `.cache()` and `.persist(MEMORY_AND_DISK)`** in PySpark?

- A) They are functionally identical — `.cache()` is just an alias for `.persist(MEMORY_AND_DISK)`.
- B) `.cache()` is an Action that immediately stores data; `.persist()` is a Transformation that waits for an Action.
- C) `.cache()` is an alias for `.persist(MEMORY_ONLY)`. Both `.cache()` and `.persist()` are lazy — data is only stored when the first Action is called.
- D) `.persist()` is only available for RDDs; `.cache()` is used for DataFrames.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

`.cache()` is syntactic sugar for `.persist(StorageLevel.MEMORY_AND_DISK)` in **DataFrames** but is equivalent to `.persist(MEMORY_ONLY)` for **RDDs** — this is a subtle but important distinction. Critically, *both* `.cache()` and `.persist()` are **lazy**: they do not immediately store data. They mark the DataFrame/RDD as "to be cached" and the actual caching happens when the first Action triggers computation and the results flow through the cache layer.

**Why the others are wrong:**
- **(A)** For DataFrames specifically, this is actually true (`.cache()` = `MEMORY_AND_DISK`), but the statement misses the critical nuance about **laziness** — neither caches data immediately.
- **(B)** Neither is an Action. Both are lazy and only mark the dataset for caching.
- **(D)** Both `.cache()` and `.persist()` are available for both RDDs and DataFrames.

</details>

---

**Q18.** You are debugging a slow Spark job in the **Spark UI**. You notice that Stage 3 has one task that takes **12 minutes**, while all other 199 tasks finish in under **10 seconds**. What is this problem called, and what is the most common root cause?

- A) **Executor Starvation** — not enough CPU cores were allocated to the slow task.
- B) **Data Skew** — one partition contains a disproportionately large amount of data (e.g., a highly frequent key in a `groupBy` or `join`), causing one task to do the majority of the work.
- C) **DAG Fragmentation** — the DAG optimizer failed to coalesce transformations before Stage 3.
- D) **Shuffle Spill** — Stage 3's shuffle data exceeded memory and was spilled to disk, causing the delay.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: B**

This is the classic symptom of **data skew**: one partition holds far more rows than the others, so its Task takes orders of magnitude longer. This typically happens when a `groupBy` or `join` key has a highly skewed distribution — for example, `groupBy("country")` when 80% of rows have `country = "US"`. The entire job is bottlenecked by that one task. Solutions include salting the skewed key, using `skewHint`, or filtering and processing the skewed key separately.

**Why the others are wrong:**
- **(A)** Executor starvation affects all tasks uniformly — tasks would queue up across the board, not create one uniquely slow task.
- **(C)** DAG fragmentation is not a standard Spark problem. The Catalyst optimizer handles DAG planning well and doesn't cause single-task slowness in this pattern.
- **(D)** Shuffle spill can slow down tasks, but it would typically affect multiple tasks (those that hit memory limits), not exclusively *one* task 72x slower than the rest.

</details>

---

**Q19.** A data engineer wants to read a large partitioned Parquet dataset, filter it down to one month of data, and then perform a heavy aggregation. They write:

```python
df = spark.read.parquet("s3://data/events/")
df_jan = df.filter(col("event_date") >= "2024-01-01") \
           .filter(col("event_date") < "2024-02-01")
result = df_jan.groupBy("product_id").agg(sum("revenue"))
result.show()
```

The Parquet dataset is partitioned on disk by `event_date`. What Spark optimization **automatically** reduces the amount of data read from S3?

- A) Catalyst Optimizer's Column Pruning — Spark reads only the `event_date`, `product_id`, and `revenue` columns from Parquet files.
- B) **Partition Pruning** — Spark's Catalyst optimizer detects the date filter and only reads the physical directory partitions matching January 2024, skipping all other months entirely.
- C) Predicate Pushdown to the Shuffle — Spark applies the date filter after the `groupBy` shuffle to reduce output size.
- D) Both A and B — Spark applies both column pruning and partition pruning simultaneously.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: D**

Spark's **Catalyst optimizer** applies two powerful optimizations here simultaneously. **Partition Pruning** skips entire directory partitions on S3 that don't match the date range — so if the dataset has 36 monthly directories, Spark only reads 1. **Column Pruning** (from Parquet's columnar format) means Spark only reads the bytes for the 3 columns used (`event_date`, `product_id`, `revenue`), ignoring all other columns. Together, these can reduce I/O by orders of magnitude.

**Why the others are wrong:**
- **(A)** Column pruning alone is a correct and real optimization, but it's incomplete — partition pruning is *also* happening and is arguably more impactful here.
- **(B)** Partition pruning alone is also correct, but again incomplete.
- **(C)** This is wrong — predicates are pushed *down* to the data source (earlier), not applied after the shuffle. Applying filters after a shuffle would mean shuffling data you don't need.

</details>

---

**Q20.** A data engineer is designing a daily pipeline that:
1. Reads 500GB of raw logs
2. Applies 15 chained transformations (filters, joins, derived columns)
3. Writes **three** different output datasets (summary, detail, audit)

All three outputs are derived from the **same intermediate cleaned DataFrame** (after step 2). What is the optimal caching strategy?

- A) Cache the raw logs DataFrame to avoid re-reading from disk for each output.
- B) Don't cache anything — Spark's DAG optimizer will automatically reuse intermediate results.
- C) Cache the intermediate cleaned DataFrame (after all 15 transformations, before step 3), so the expensive transformation chain is computed once and all three writes read from the cache.
- D) Cache all three output DataFrames before writing, so each write is instantaneous.

<details>
<summary>💡 Reveal Answer & Explanation</summary>

**✅ Correct Answer: C**

The expensive work is the **15-step transformation chain** on 500GB of data. Without caching, each of the three `.write` actions would trigger a full re-execution of all 15 transformations from the raw source. By caching the cleaned DataFrame at the **"fan-out" point** (right before the three diverging write paths), the pipeline computes the expensive chain exactly **once**, materializes it in memory/disk, and then serves all three writers from the cache — reducing compute cost by ~3x.

**Why the others are wrong:**
- **(A)** Caching the raw logs doesn't help much — the bottleneck is the 15 transformations, not the initial read. Caching the input means you still recompute all 15 transformations three times.
- **(B)** Spark does **not** automatically reuse intermediate results across multiple Actions. This is a common misconception. Without explicit caching, each Action re-executes the full DAG.
- **(D)** Caching the *output* DataFrames (after they are computed) doesn't prevent the transformation chain from running multiple times — you'd still recompute during the write action, and the cached versions would never actually be used by the `.write` calls.

</details>

---

## 📊 Score Tracker

| Section | Questions | Your Score |
|---|---|---|
| Spark Architecture | Q1 – Q3 | &nbsp;&nbsp;&nbsp;/ 3 |
| Execution Hierarchy | Q4 – Q6 | &nbsp;&nbsp;&nbsp;/ 3 |
| Lazy Evaluation & DAG | Q7 – Q9 | &nbsp;&nbsp;&nbsp;/ 3 |
| Shuffling & Network I/O | Q10 – Q12 | &nbsp;&nbsp;&nbsp;/ 3 |
| Optimization Techniques | Q13 – Q20 | &nbsp;&nbsp;&nbsp;/ 8 |
| **Total** | **Q1 – Q20** | **&nbsp;&nbsp;&nbsp;/ 20** |

---

## 🧠 Key Concepts Cheat Sheet

| Concept | One-Line Summary |
|---|---|
| **Driver** | Orchestrates the job: builds DAG, schedules stages, collects results |
| **Executor** | Runs tasks, processes partitions, stores cached data |
| **Cluster Manager** | Allocates resources (YARN / Kubernetes / Mesos / Standalone) |
| **Job** | Triggered by one Action |
| **Stage** | Separated by shuffle boundaries (wide transformations) |
| **Task** | One unit of work = one partition in one stage |
| **Narrow Transform** | Each output partition ← one input partition (no shuffle) |
| **Wide Transform** | Output partition ← multiple input partitions (shuffle required) |
| **`spark.sql.shuffle.partitions`** | Controls post-shuffle partition count; default 200 |
| **Broadcast Hash Join** | Small table sent to all Executors; eliminates shuffle |
| **Shuffle Sort-Merge Join** | Both large tables shuffled by key; scalable but expensive |
| **`MEMORY_ONLY`** | Fast but evicts partitions silently; triggers recompute |
| **`MEMORY_AND_DISK`** | Safe fallback; spills to disk instead of evicting |
| **Data Skew** | One partition >> others; one task bottlenecks the whole stage |

---

*Good luck with your certification! 🚀*

# Apache Spark Deep Dive Guide

> 🔥 A comprehensive, interview-ready, exam-focused deep dive into Apache Spark internals, architecture, optimization, and tricky edge cases. Built for engineers who want to truly understand Spark — not just use it.

---

## 📌 Table of Contents

1. [🏗️ Spark Architecture](#️-spark-architecture)
2. [🔹 RDD Basics](#-rdd-basics)
3. [🔄 Transformations vs Actions (Lazy Evaluation)](#-transformations-vs-actions-lazy-evaluation)
4. [🔀 Wide vs Narrow Transformations](#-wide-vs-narrow-transformations)
5. [📊 DAG & Execution Plan](#-dag--execution-plan)
6. [🔁 Partitioning & Shuffle](#-partitioning--shuffle)
7. [🧾 DataFrame & Spark SQL API](#-dataframe--spark-sql-api)
8. [💾 Caching & Storage Levels](#-caching--storage-levels)
9. [📁 File Formats (Parquet, Columnar)](#-file-formats-parquet-columnar)
10. [🔗 Joins (Broadcast vs Shuffle)](#-joins-broadcast-vs-shuffle)
11. [🧩 Spark Components](#-spark-components)
12. [⚙️ Execution Hierarchy (Job → Stage → Task → Slot)](#️-execution-hierarchy-job--stage--task--slot)
13. [🚀 Deployment Modes](#-deployment-modes)
14. [🧠 Catalyst Optimizer](#-catalyst-optimizer)
15. [⚡ spark.sql.shuffle.partitions](#-sparksqlshufflepartitions)
16. [🧪 Advanced Practice MCQs](#-advanced-practice-mcqs)
17. [🔥 Interview Cheat Sheet](#-interview-cheat-sheet)
18. [⚡ Performance Optimization Tips](#-performance-optimization-tips)
19. [🚨 Common Pitfalls](#-common-pitfalls)

---

## 🏗️ Spark Architecture

### Definition
Apache Spark is a unified analytics engine for large-scale data processing. Its architecture follows a **master-worker** (driver-executor) model with a centralized `SparkContext` and a cluster manager for resource negotiation.

### Internal Working

```
┌─────────────────────────────────────────────────────────────────┐
│                        SPARK APPLICATION                        │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    DRIVER PROGRAM                        │   │
│  │  ┌──────────────┐    ┌────────────────────────────────┐  │   │
│  │  │ SparkContext │    │     DAGScheduler               │  │   │
│  │  │  (Entry pt)  │───▶│  (Builds execution plan)       │  │   │
│  │  └──────────────┘    └───────────┬────────────────────┘  │   │
│  │                                  │                        │   │
│  │                       ┌──────────▼──────────┐            │   │
│  │                       │   TaskScheduler     │            │   │
│  │                       │ (Assigns tasks)      │            │   │
│  └───────────────────────┼─────────────────────┼────────────┘   │
│                          │                     │                 │
│         ┌────────────────▼──────────────────┐  │                 │
│         │        CLUSTER MANAGER            │  │                 │
│         │  (YARN / Mesos / Standalone / K8s)│  │                 │
│         └────────────────┬──────────────────┘  │                 │
│                          │                     │                 │
│        ┌─────────────────┼─────────────────────┼──────────┐      │
│        │                 │                     │          │      │
│  ┌─────▼──────┐   ┌──────▼─────┐   ┌──────────▼────┐     │      │
│  │  Executor  │   │  Executor  │   │   Executor    │     │      │
│  │  ┌──────┐  │   │  ┌──────┐  │   │   ┌──────┐   │     │      │
│  │  │ Task │  │   │  │ Task │  │   │   │ Task │   │     │      │
│  │  │ Task │  │   │  │ Task │  │   │   │ Task │   │     │      │
│  │  └──────┘  │   │  └──────┘  │   │   └──────┘   │     │      │
│  │  [Cache]   │   │  [Cache]   │   │   [Cache]    │     │      │
│  └────────────┘   └────────────┘   └──────────────┘     │      │
└─────────────────────────────────────────────────────────────────┘
```

### Key Components

| Component       | Role                                          | Runs Where               |
| --------------- | --------------------------------------------- | ------------------------ |
| Driver          | Hosts `SparkContext`, builds DAG, coordinates | Master node              |
| SparkContext    | Entry point to Spark functionality            | Driver JVM               |
| DAGScheduler    | Converts RDD lineage to stage DAG             | Driver                   |
| TaskScheduler   | Assigns tasks to executors                    | Driver                   |
| Cluster Manager | Resource negotiation                          | External (YARN/K8s/etc.) |
| Executor        | Executes tasks, caches data                   | Worker nodes             |

### Real-World Analogy
Think of it like a **restaurant**:
- **Driver** = Head chef who plans the entire menu (execution plan)
- **Cluster Manager** = Restaurant manager who assigns stations
- **Executors** = Line cooks who do the actual cooking
- **Tasks** = Individual cooking steps

### ⚠️ Edge Cases

- **Driver failure = job failure.** The driver is a single point of failure. There is no automatic driver HA in most modes (exception: YARN with `--supervise`).
- **Executor failure ≠ job failure.** Spark can retry tasks on other executors using lineage.
- **More slots than tasks:** Extra slots sit idle. Tasks are NOT split further — a task is atomic.
- **Fewer partitions than cores:** Some cores are idle the entire job. This is a common misconfiguration.

### ❌ Common Mistakes

- Assuming Spark is always faster than MapReduce — for small datasets with no reuse, the overhead may actually make Spark slower.
- Assuming driver crash is recoverable — it is not without checkpointing or external orchestration.

### 🎯 Interview Traps

> **"Is the driver a bottleneck?"**  
> **YES.** All metadata, DAG construction, and result collection happen on the driver. If you `collect()` a 10TB dataset to the driver — it will OOM crash.

---

## 🔹 RDD Basics

### Definition
**Resilient Distributed Dataset (RDD)** is Spark's fundamental abstraction — an immutable, fault-tolerant, distributed collection of elements that can be processed in parallel.

### The 5 RDD Properties

```
RDD = ( Partitions[] + Dependencies[] + compute() + 
        Optional[Partitioner] + Optional[PreferredLocations[]] )
```

| Property            | Description                                      |
| ------------------- | ------------------------------------------------ |
| Partitions          | List of partition objects                        |
| Dependencies        | Parent RDDs (narrow or wide)                     |
| compute()           | Function to compute each partition               |
| Partitioner         | Optional: how data is partitioned (hash/range)   |
| Preferred Locations | Data locality hints (e.g., HDFS block locations) |

### Internal Working

```python
# Creating RDDs
sc = SparkContext("local[*]", "RDDDemo")

# From collection (parallelized) — data sent from driver to executors
rdd1 = sc.parallelize([1, 2, 3, 4, 5], numSlices=3)

# From external storage — read directly into executor memory
rdd2 = sc.textFile("hdfs://path/to/file.txt", minPartitions=10)

# From another RDD (transformation — lazy!)
rdd3 = rdd1.map(lambda x: x * 2)

# Checking partition count
print(rdd3.getNumPartitions())  # 3

# Viewing RDD lineage
print(rdd3.toDebugString())
```

### Real-World Analogy
An RDD is like a **distributed spreadsheet shredded into pieces** (partitions) stored across multiple filing cabinets (nodes). The "resilient" part means: if one cabinet burns down, Spark knows the recipe to recreate that piece from scratch using the lineage graph.

### Fault Tolerance Mechanism

```
RDD_C = RDD_A.flatMap(...).RDD_B.reduceByKey(...)
          ↑                          ↑
    Partition A1              Partition B1
    [if lost]                 [if lost]
         ↓                          ↓
    Recompute from          Recompute from
    RDD_A lineage           RDD_A → flatMap
```

### Performance Impact

- RDDs have **no schema** → no Catalyst optimization → slower than DataFrames for structured data
- RDDs use **Java serialization** by default → more memory overhead vs DataFrames (Tungsten binary format)
- RDDs are best for **unstructured data** or when you need fine-grained control

### ⚠️ Edge Cases

- `sc.parallelize([1,2,3])` puts data on the **driver first**, then distributes it. For large data, use `sc.textFile()` instead.
- **Empty partitions are valid.** `sc.parallelize([], 5)` creates 5 empty partitions — no error.
- Setting `numSlices > number of items` → some partitions are empty. Tasks still get scheduled for them.

### ❌ Common Mistakes

```python
# ❌ WRONG: collecting inside a transformation (runs on driver, defeats parallelism)
rdd.map(lambda x: some_rdd.collect()[x])  # catastrophic

# ✅ CORRECT: use joins or broadcast variables instead
broadcast_var = sc.broadcast(lookup_dict)
rdd.map(lambda x: broadcast_var.value.get(x))
```

### 🎯 Interview Traps

> **"RDDs are immutable. So how does Spark modify data?"**  
> It doesn't. Each transformation creates a NEW RDD. The original is never changed. This immutability is what enables fault tolerance via lineage recomputation.

---

## 🔄 Transformations vs Actions (Lazy Evaluation)

### Definition

| Concept            | Definition                                       | Returns                   |
| ------------------ | ------------------------------------------------ | ------------------------- |
| **Transformation** | Defines a new RDD/DataFrame from an existing one | RDD/DataFrame (lazy)      |
| **Action**         | Triggers actual computation and returns a result | Value / writes to storage |

### The Lazy Evaluation Model

```
User Code (Driver)              What Spark Actually Does
─────────────────────           ────────────────────────────────

rdd.filter(...)         →       [Records transformation in DAG]
   .map(...)            →       [Records transformation in DAG]
   .groupByKey()        →       [Records transformation in DAG]
   .count()             →       ← TRIGGERS ACTUAL EXECUTION HERE
                                  Spark builds a physical plan
                                  Tasks are scheduled
                                  Data actually moves
```

### PySpark Example

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("LazyDemo").getOrCreate()
sc = spark.sparkContext

# All of these are LAZY — zero computation happens
rdd = sc.textFile("hdfs://data/logs.txt")           # No read yet
words = rdd.flatMap(lambda line: line.split(" "))   # No compute
filtered = words.filter(lambda w: len(w) > 3)       # No compute
pairs = filtered.map(lambda w: (w, 1))              # No compute
counts = pairs.reduceByKey(lambda a, b: a + b)      # No compute

# THIS triggers everything above:
result = counts.take(10)   # ← Execution starts here
print(result)
```

### Common Transformations vs Actions

| Transformations (Lazy) | Actions (Eager)            |
| ---------------------- | -------------------------- |
| `map()`, `flatMap()`   | `count()`                  |
| `filter()`             | `collect()`                |
| `groupByKey()`         | `take(n)`                  |
| `reduceByKey()`        | `first()`                  |
| `join()`               | `saveAsTextFile()`         |
| `distinct()`           | `foreach()`                |
| `union()`              | `show()` (DataFrame)       |
| `select()`, `where()`  | `write.save()` (DataFrame) |

### Why Lazy Evaluation?

1. **Optimization:** Spark can merge operations (pipelining) and eliminate redundant steps
2. **Fault tolerance:** Only recompute what's needed when a partition is lost
3. **Efficiency:** Avoids unnecessary computation if action only needs partial results (e.g., `take(5)`)

### ⚠️ Edge Cases

```python
# ⚠️ TRAP: This LOOKS like it runs twice, but it DOES run twice
# (unless rdd is cached)
rdd = sc.textFile("huge_file.txt").map(expensive_function)
count = rdd.count()      # Full execution #1
samples = rdd.take(10)   # Full execution #2 — reads file AGAIN!

# ✅ FIX: Cache the RDD
rdd = sc.textFile("huge_file.txt").map(expensive_function).cache()
count = rdd.count()      # Executes and caches
samples = rdd.take(10)   # Uses cache — no recomputation
```

- **`show()` in DataFrame IS an action.** It triggers execution — do not confuse it with a transformation.
- **`printSchema()` is NOT an action.** It doesn't trigger execution (schema is inferred lazily but not data).

### ❌ Common Mistakes

```python
# ❌ Using collect() on large data — OOM on driver
all_data = rdd.collect()  # Pulls ALL data to driver memory — crashes on big data

# ✅ Use take(), first(), or write to storage
sample = rdd.take(100)
rdd.saveAsTextFile("output/path")
```

### 🎯 Interview Traps

> **"If transformations are lazy, does Spark validate my code before an action?"**  
> Only **syntactically**. Logical errors (wrong column names, type mismatches in RDDs) are only caught at **runtime when the action triggers execution**. DataFrames do more analysis-time checking, but even then — only when `explain()` or an action is called.

---

## 🔀 Wide vs Narrow Transformations

### Definition

| Type       | Definition                                                    | Shuffle Required? | Examples                                                       |
| ---------- | ------------------------------------------------------------- | ----------------- | -------------------------------------------------------------- |
| **Narrow** | Each output partition depends on at most one input partition  | ❌ No              | `map`, `filter`, `flatMap`, `union`, `mapPartitions`           |
| **Wide**   | Each output partition may depend on multiple input partitions | ✅ Yes             | `groupByKey`, `reduceByKey`, `join`, `distinct`, `repartition` |

### Visual Explanation

```
NARROW TRANSFORMATION (e.g., filter)
────────────────────────────────────
Partition 1 ──────────────▶ Partition 1'
Partition 2 ──────────────▶ Partition 2'
Partition 3 ──────────────▶ Partition 3'

Each input partition maps to exactly ONE output partition.
Data does NOT cross partition boundaries. No shuffle.

WIDE TRANSFORMATION (e.g., groupByKey)
───────────────────────────────────────
Partition 1 ─────────┬─────▶ Partition 1'
                     │ ╲
Partition 2 ─────────┤  ╲──▶ Partition 2'
                     │   ╲
Partition 3 ─────────┘    ──▶ Partition 3'

Records from ALL input partitions may contribute to ANY
output partition. SHUFFLE required — data moves across network.
```

### PySpark Example

```python
rdd = sc.parallelize([(1, 'a'), (2, 'b'), (1, 'c'), (2, 'd')], 3)

# NARROW: filter — no shuffle, data stays in its partition
narrow_result = rdd.filter(lambda x: x[0] > 1)

# WIDE: groupByKey — shuffle, data moves across network
# Spark must send all records with same key to same partition
wide_result = rdd.groupByKey()

# Compare the DAG
narrow_result.toDebugString()  # No ShuffleRDD in lineage
wide_result.toDebugString()    # ShuffleRDD appears in lineage
```

### Why Shuffles Are Expensive

```
Shuffle Cost = Serialize data + Write to disk + Network transfer + Deserialize + Re-sort
                   ↑                ↑                ↑                ↑            ↑
              CPU cost         I/O cost          Network I/O       CPU cost     CPU cost
```

### Stage Boundaries

> **Wide transformations create stage boundaries in the DAG.**

```
Stage 1 (no shuffle):        Stage 2 (after shuffle):
  textFile                     reduceByKey output
  → flatMap                    → map
  → map                        → filter
  [SHUFFLE WRITE]          [SHUFFLE READ] → [SHUFFLE WRITE]   Stage 3:
                                                               → collect
```

### ⚠️ Edge Cases

- **`repartition()` is always WIDE** (shuffle), even if you're repartitioning to the same number. Use `coalesce()` for reducing partitions without a full shuffle.
- **`union()` is NARROW** — no shuffle. It just concatenates partition lists.
- **`join()` behavior depends on partitioner.** Two RDDs with the same partitioner and same number of partitions join as a narrow transformation (co-partitioned join). Otherwise, it's wide.
- **`coalesce(n)` with n < current partitions can be narrow** (merges partitions locally). But `coalesce(n, shuffle=True)` forces a wide transformation.

### ❌ Common Mistakes

```python
# ❌ groupByKey is almost always wrong — it collects ALL values per key
# into memory on one executor, causing OOM for high-cardinality keys
rdd.groupByKey().mapValues(sum)  # BAD

# ✅ reduceByKey combines locally BEFORE shuffle (map-side aggregation)
rdd.reduceByKey(lambda a, b: a + b)  # GOOD — much less data transferred

# ❌ Using distinct() when you just want to deduplicate keys
rdd.map(lambda x: x[0]).distinct()  # Triggers shuffle

# ✅ If possible, use reduceByKey with a dummy value
rdd.map(lambda x: (x[0], None)).reduceByKey(lambda a, b: a).keys()
```

### 🎯 Interview Traps

> **"Why does filter() not cause a shuffle but distinct() does?"**  
> `filter()` can be applied per-partition independently — each record is evaluated in isolation. `distinct()` requires comparing records ACROSS ALL partitions to find duplicates — so data must be shuffled to a common destination keyed by value hash.

---

## 📊 DAG & Execution Plan

### Definition
A **Directed Acyclic Graph (DAG)** in Spark represents the logical sequence of computations. Each node is an RDD/DataFrame and each edge is a transformation. The DAGScheduler converts this into physical stages.

### DAG Lifecycle

```
User Code
    │
    ▼
Logical Plan (unresolved)
    │  [Catalyst: Analysis]
    ▼
Analyzed Logical Plan (resolved)
    │  [Catalyst: Logical Optimization]
    ▼
Optimized Logical Plan
    │  [Catalyst: Physical Planning]
    ▼
Physical Plans (multiple candidates)
    │  [Catalyst: Cost Model]
    ▼
Selected Physical Plan
    │  [Tungsten: Code Generation]
    ▼
RDD DAG → DAGScheduler → Stages → Tasks → Execution
```

### DAG Example

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("DAGDemo").getOrCreate()

df = spark.read.parquet("hdfs://sales/")
result = (df
    .filter(df.amount > 100)          # Stage 1: narrow
    .groupBy("region")                 # Stage boundary (wide)
    .agg({"amount": "sum"})            # Stage 2: aggregation
    .orderBy("sum(amount)", ascending=False)  # Stage boundary (wide)
    .limit(10)                         # Stage 3: narrow
)

# View physical plan
result.explain(extended=True)

# View DAG in Spark UI: http://localhost:4040/stages/
```

### Reading `explain()` Output

```
== Physical Plan ==
TakeOrderedAndProject(limit=10, orderBy=[sum(amount) DESC])    ← Stage 3
+- *(2) HashAggregate(keys=[region], functions=[sum(amount)])  ← Stage 2 start
   +- Exchange hashpartitioning(region, 200)                   ← SHUFFLE (stage boundary)
      +- *(1) HashAggregate(keys=[region], functions=[partial_sum(amount)])
         +- *(1) Filter (amount > 100)                         ← Stage 1 (narrow)
            +- *(1) FileScan parquet [region, amount]          ← Stage 1 start
```

### Stage Boundaries Rule

```
A new STAGE is created whenever there is a WIDE transformation (shuffle).
Everything between two shuffles is ONE stage.
Tasks within a stage can be pipelined together.
```

### ⚠️ Edge Cases

- **Spark can merge multiple narrow transformations** into a single stage. A chain of `map → filter → map` is executed as one stage, often in a single pass over each partition (pipelining).
- **`explain()` shows the PLANNED DAG**, not necessarily what was actually executed (e.g., AQE may modify the plan at runtime).
- **The Spark UI DAG visualization shows stages, not individual transformations.** One stage can contain multiple transformations.

### ❌ Common Mistakes

```python
# ❌ Calling explain() thinking it shows the final runtime plan
df.explain()  # This is the STATIC optimizer plan, not AQE-adjusted runtime plan

# ✅ Use explain(mode="formatted") for cleaner output (Spark 3.0+)
df.explain(mode="formatted")
```

### 🎯 Interview Traps

> **"If I have 5 transformations and 2 shuffles, how many stages do I have?"**  
> **3 stages.** Each shuffle creates a stage boundary, dividing N shuffles into N+1 stages. The exact count also depends on actions, which can also force stage boundaries.

---

## 🔁 Partitioning & Shuffle

### Definition
**Partitioning** is how data is divided across executors. **Shuffle** is the process of redistributing data across partitions — the most expensive operation in Spark.

### Types of Partitioners

| Partitioner                   | How it works                      | Best for                     |
| ----------------------------- | --------------------------------- | ---------------------------- |
| **HashPartitioner** (default) | `key.hashCode() % numPartitions`  | General key-value operations |
| **RangePartitioner**          | Samples data to assign key ranges | `sortByKey`, ordered data    |
| **Custom Partitioner**        | User-defined                      | Domain-specific locality     |

### Shuffle Internals

```
SHUFFLE WRITE (Map Side):
   Executor 1                    Executor 2
   ──────────                    ──────────
   Partition 0 data              Partition 0 data
        │                              │
        ▼                              ▼
   Hash each key                 Hash each key
        │                              │
   ┌────┴─────┐                  ┌─────┴────┐
   │ Bucket 0 │ Bucket 1│        │ Bucket 0 │ Bucket 1│
   └────┬─────┘ ────────┘        └──────────┘ ────────┘
        │
        ▼
   Write shuffle files to LOCAL DISK

SHUFFLE READ (Reduce Side):
   Each reducer fetches its bucket FROM ALL map outputs:
   Reducer 0: fetches Bucket 0 from Executor 1 + Bucket 0 from Executor 2
   Reducer 1: fetches Bucket 1 from Executor 1 + Bucket 1 from Executor 2
```

### PySpark Partitioning Control

```python
from pyspark.sql import SparkSession
from pyspark import StorageLevel

spark = SparkSession.builder.appName("PartitionDemo").getOrCreate()
sc = spark.sparkContext

# Check current partition count
rdd = sc.textFile("data.txt")
print(f"Partitions: {rdd.getNumPartitions()}")

# Increase partitions (always triggers shuffle)
rdd_more = rdd.repartition(100)

# Decrease partitions (no shuffle if only reducing — avoids full shuffle)
rdd_less = rdd.coalesce(10)  # NARROW — just merges partitions locally

# Custom partitioner
rdd_pairs = rdd.map(lambda x: (x[0], x))
rdd_custom = rdd_pairs.partitionBy(20, lambda key: hash(key) % 20)

# DataFrame: control partitions after shuffle
spark.conf.set("spark.sql.shuffle.partitions", "50")
df = spark.read.parquet("sales/")
df_partitioned = df.repartition(50, "region")  # Hash partition by column
df_range = df.repartitionByRange(50, "date")   # Range partition by column
```

### Partition Size Guidelines

| Scenario            | Recommendation                         |
| ------------------- | -------------------------------------- |
| Too few partitions  | Cores sit idle, OOM risk per task      |
| Too many partitions | Task scheduling overhead dominates     |
| Ideal target        | 128MB–256MB uncompressed per partition |
| Ratio               | ~2–4x number of cores in cluster       |

### Skew Problem

```python
# Data skew: one key has 90% of records
rdd = sc.parallelize([("hot_key", i) for i in range(1_000_000)] + 
                     [("cold_key", i) for i in range(1000)])

# ❌ Hot key causes one reducer to get 1M records, others get 1K
rdd.reduceByKey(lambda a, b: a + b)

# ✅ Salting technique: add random prefix to distribute hot key
import random
salted = rdd.map(lambda x: (f"{x[0]}_{random.randint(0, 9)}", x[1]))
reduced = salted.reduceByKey(lambda a, b: a + b)
# Then strip the salt prefix and reduce again
final = reduced.map(lambda x: (x[0].rsplit("_", 1)[0], x[1])) \
               .reduceByKey(lambda a, b: a + b)
```

### ⚠️ Edge Cases

- **`coalesce()` without shuffle can create unbalanced partitions.** It just merges adjacent partitions. If data is skewed, `repartition()` (with shuffle) distributes more evenly.
- **Partition count affects downstream operations.** Calling `repartition(200)` then `coalesce(1)` still shuffles for the repartition step.
- **HDFS block size = default partition size** for `sc.textFile()`. A 1GB file with 128MB blocks → 8 partitions by default.

### ❌ Common Mistakes

```python
# ❌ Using repartition() to reduce partitions — unnecessary shuffle
df.repartition(10)  # Always shuffles even when reducing

# ✅ Use coalesce() when reducing partition count
df.coalesce(10)  # Merges without full shuffle (narrow transformation)

# ❌ Setting global shuffle.partitions to 200 for a 10MB dataset
spark.conf.set("spark.sql.shuffle.partitions", "200")
# Creates 200 tasks for tiny data — overhead > actual work

# ✅ Tune per-job or use AQE
spark.conf.set("spark.sql.adaptive.enabled", "true")  # AQE auto-tunes
```

### 🎯 Interview Traps

> **"Does `coalesce()` always avoid a shuffle?"**  
> No. `coalesce(n, shuffle=True)` explicitly forces a shuffle. Also, if you call `coalesce()` after a transformation that produces a `ShuffledRDD` in the same job, the physical optimizer may still insert a shuffle.

---

## 🧾 DataFrame & Spark SQL API

### Definition
**DataFrames** are distributed collections of data organized into named columns — conceptually equivalent to a SQL table. They are built on top of RDDs but use Spark's **Catalyst optimizer** and **Tungsten execution engine** for superior performance.

### DataFrame vs RDD vs Dataset

| Feature                  | RDD            | DataFrame      | Dataset (Scala/Java) |
| ------------------------ | -------------- | -------------- | -------------------- |
| Type safety              | ✅ Compile-time | ❌ Runtime only | ✅ Compile-time       |
| Catalyst optimization    | ❌ No           | ✅ Yes          | ✅ Yes                |
| Tungsten (binary format) | ❌ No           | ✅ Yes          | ✅ Yes                |
| Python support           | ✅ Yes          | ✅ Yes          | ❌ Not in PySpark     |
| Schema                   | ❌ None         | ✅ Enforced     | ✅ Enforced           |
| Performance              | Lowest         | High           | High                 |

### PySpark DataFrame Operations

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

spark = SparkSession.builder \
    .appName("DataFrameDemo") \
    .config("spark.sql.shuffle.partitions", "50") \
    .getOrCreate()

# Schema definition (always prefer explicit schema for production)
schema = StructType([
    StructField("id", IntegerType(), nullable=False),
    StructField("name", StringType(), nullable=True),
    StructField("dept", StringType(), nullable=True),
    StructField("salary", IntegerType(), nullable=True)
])

df = spark.read.schema(schema).csv("employees.csv", header=True)

# Transformations (lazy)
result = (df
    .filter(F.col("salary") > 50000)
    .withColumn("bonus", F.col("salary") * 0.1)
    .groupBy("dept")
    .agg(
        F.avg("salary").alias("avg_salary"),
        F.count("*").alias("emp_count"),
        F.sum("bonus").alias("total_bonus")
    )
    .orderBy(F.desc("avg_salary"))
)

# Actions (trigger execution)
result.show(20, truncate=False)
result.write.mode("overwrite").parquet("output/dept_summary/")

# Spark SQL interface
df.createOrReplaceTempView("employees")
spark.sql("""
    SELECT dept, AVG(salary) as avg_salary
    FROM employees
    WHERE salary > 50000
    GROUP BY dept
    ORDER BY avg_salary DESC
""").show()
```

### Column Operations

```python
# Selecting and renaming
df.select("id", F.col("name").alias("employee_name"))

# Conditional logic
df.withColumn("grade", 
    F.when(F.col("salary") > 100000, "Senior")
     .when(F.col("salary") > 60000, "Mid")
     .otherwise("Junior"))

# Window functions
from pyspark.sql.window import Window
window = Window.partitionBy("dept").orderBy(F.desc("salary"))
df.withColumn("rank", F.rank().over(window))

# Handling nulls
df.fillna({"salary": 0, "name": "Unknown"})
df.dropna(subset=["id", "salary"])
df.filter(F.col("name").isNotNull())
```

### ⚠️ Edge Cases

- **`df.schema` is resolved lazily.** Calling `printSchema()` does NOT trigger data read for Parquet (schema is in file metadata), but DOES trigger a partial read for CSV/JSON (Spark samples rows to infer schema).
- **`createOrReplaceTempView()` creates a session-scoped view.** It disappears when the SparkSession ends. Use `createOrReplaceGlobalTempView()` for cross-session views, accessed as `global_temp.view_name`.
- **Python UDFs are serialization bottlenecks.** Each row is serialized from JVM → Python → JVM. Use Pandas UDFs (vectorized) or built-in SQL functions instead.

### ❌ Common Mistakes

```python
# ❌ Using Python UDF when a built-in function exists
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType
upper_udf = udf(lambda x: x.upper(), StringType())
df.withColumn("upper_name", upper_udf(F.col("name")))  # Slow (Python UDF)

# ✅ Use built-in function
df.withColumn("upper_name", F.upper(F.col("name")))  # Fast (JVM native)

# ❌ Chaining too many withColumn() calls (creates deep nested plan)
for col_name in many_columns:
    df = df.withColumn(col_name, some_func(col_name))

# ✅ Use select() for multiple column transformations
df.select("*", *[some_func(c).alias(c + "_new") for c in many_columns])
```

### 🎯 Interview Traps

> **"Is a DataFrame just an alias for a Dataset[Row] in PySpark?"**  
> Technically yes — in Scala/Java, `DataFrame = Dataset[Row]`. But in **PySpark**, the `Dataset` API doesn't exist. PySpark's DataFrame is the primary API, and it lacks the compile-time type safety of Scala's Dataset. This is a common source of confusion.

---

## 💾 Caching & Storage Levels

### Definition
**Caching** in Spark stores the results of an RDD/DataFrame computation in memory (and/or disk) so subsequent actions can reuse the cached data without recomputing from scratch.

### Storage Level Options

| Storage Level         | Memory       | Disk | Serialized | Replicated | Description                                        |
| --------------------- | ------------ | ---- | ---------- | ---------- | -------------------------------------------------- |
| `MEMORY_ONLY`         | ✅            | ❌    | ❌          | 1x         | Default for RDD `cache()`. Drops partitions if OOM |
| `MEMORY_AND_DISK`     | ✅            | ✅    | ❌          | 1x         | Spills to disk if OOM. Safer                       |
| `MEMORY_ONLY_SER`     | ✅            | ❌    | ✅          | 1x         | Smaller footprint, more CPU                        |
| `MEMORY_AND_DISK_SER` | ✅            | ✅    | ✅          | 1x         | Compressed in memory and disk                      |
| `DISK_ONLY`           | ❌            | ✅    | ✅          | 1x         | Slowest but tolerates large data                   |
| `MEMORY_ONLY_2`       | ✅            | ❌    | ❌          | 2x         | Replicated across 2 nodes                          |
| `OFF_HEAP`            | ✅ (off-heap) | ❌    | ✅          | 1x         | Avoids GC overhead                                 |

### PySpark Caching Example

```python
from pyspark import StorageLevel

# DataFrame caching
df = spark.read.parquet("large_dataset/")

# Method 1: Simple cache (MEMORY_AND_DISK for DataFrames)
df.cache()

# Method 2: Explicit storage level
df.persist(StorageLevel.MEMORY_AND_DISK_SER)

# Trigger cache population (cache is lazy — filled on first action)
df.count()  # ← This fills the cache

# Second action uses cache
df.groupBy("region").count().show()  # ← No reread from disk!

# Release cache when done
df.unpersist()

# RDD caching
rdd = sc.textFile("logs/").map(parse_line)
rdd.persist(StorageLevel.MEMORY_ONLY)
rdd.count()   # Fills cache
rdd.take(10)  # Uses cache
rdd.unpersist()
```

### When to Cache

```
USE CACHE when:                     SKIP CACHE when:
─────────────────────               ─────────────────────────────
• Same DF/RDD used 2+ times         • Used only once
• Iterative algorithms (ML)         • Larger than available memory
• Interactive exploration            • Simple pipelines
• After expensive joins/aggregations• Data is already on fast SSD
```

### Cache vs Checkpoint

| Feature         | Cache/Persist                | Checkpoint              |
| --------------- | ---------------------------- | ----------------------- |
| Storage         | Memory/Disk (executor local) | HDFS / reliable storage |
| Lineage         | Preserved                    | Truncated (lineage cut) |
| Speed           | Faster                       | Slower (HDFS write)     |
| Fault tolerance | Recomputed if executor lost  | Survives driver restart |
| Use case        | Performance optimization     | Long lineage, Streaming |

```python
# Checkpointing: truncates lineage, saves to reliable storage
sc.setCheckpointDir("hdfs://checkpoints/")
rdd = sc.textFile("data/").map(f1).filter(f2).flatMap(f3)  # Long lineage
rdd.checkpoint()  # Will be saved to HDFS on first action
rdd.count()       # Checkpoint written here
```

### ⚠️ Edge Cases

- **`cache()` on a DataFrame uses `MEMORY_AND_DISK` by default** (not `MEMORY_ONLY`). This differs from RDD `cache()` which uses `MEMORY_ONLY`.
- **Caching is lazy.** Even after calling `.cache()`, data is NOT stored until an action triggers computation.
- **Cached data is partition-granular.** If a cached partition is evicted (LRU), it will be recomputed from lineage on next access — not the entire RDD.
- **`unpersist()` is synchronous by default** in Spark 3.x. Call `unpersist(blocking=True)` to ensure cleanup before next step.

### ❌ Common Mistakes

```python
# ❌ Caching data that's used only once (wastes memory)
df.cache()
df.write.parquet("output/")  # Only one action — cache pointless

# ❌ Forgetting to unpersist — leads to memory pressure
df.cache()
df.count()
# ... hours later, df is still in memory eating executor RAM

# ❌ Caching after a wide transformation without triggering
df = df.groupBy("key").agg(F.sum("val")).cache()
# Cache is NOT populated yet — next line fills it
result = df.count()
```

### 🎯 Interview Traps

> **"If I cache a DataFrame and then an executor dies, what happens?"**  
> The **lost partition** is recomputed from the original lineage (since cache is non-replicated by default). The other cached partitions on surviving executors remain intact. Use `MEMORY_ONLY_2` or checkpointing if you need true resilience.

---

## 📁 File Formats (Parquet, Columnar)

### Comparison of Common Formats

| Format         | Structure      | Compression   | Schema             | Splittable | Best for                         |
| -------------- | -------------- | ------------- | ------------------ | ---------- | -------------------------------- |
| **Parquet**    | Columnar       | Snappy/ZSTD   | Embedded           | ✅ Yes      | Analytics queries, Spark default |
| **ORC**        | Columnar       | ZLIB/Snappy   | Embedded           | ✅ Yes      | Hive-heavy ecosystems            |
| **Avro**       | Row-based      | Deflate       | Embedded           | ✅ Yes      | Kafka, write-heavy workloads     |
| **CSV**        | Row-based      | None native   | None               | ✅ Yes      | Human-readable, interchange      |
| **JSON**       | Row-based      | None native   | None               | ✅ (lines)  | Semi-structured data             |
| **Delta Lake** | Columnar + Log | Parquet-level | Schema + evolution | ✅ Yes      | ACID transactions, time travel   |

### Columnar vs Row Storage

```
ROW-BASED (CSV, JSON, Avro):
Row 1: [id=1, name="Alice", salary=90000, dept="Eng"]
Row 2: [id=2, name="Bob",   salary=75000, dept="HR"]
Row 3: [id=3, name="Carol", salary=85000, dept="Eng"]

Query: SELECT AVG(salary) WHERE dept = 'Eng'
  → Must read ALL columns of ALL rows, then filter
  → Reads: id, name, salary, dept × 3 rows = 12 field reads

COLUMNAR (Parquet, ORC):
Col id:     [1, 2, 3]
Col name:   ["Alice", "Bob", "Carol"]
Col salary: [90000, 75000, 85000]
Col dept:   ["Eng", "HR", "Eng"]

Query: SELECT AVG(salary) WHERE dept = 'Eng'
  → Read only 'dept' column → filter rows 1 and 3
  → Read only 'salary' for rows 1 and 3
  → Reads: 6 field reads — 50% less I/O
```

### Parquet Internals

```
PARQUET FILE STRUCTURE:
┌─────────────────────────────────────────────┐
│  4-byte magic: PAR1                         │
├─────────────────────────────────────────────┤
│  Row Group 0 (e.g., 128MB)                  │
│  ┌─────────────┐  ┌─────────────┐           │
│  │  Col Chunk  │  │  Col Chunk  │  ...      │
│  │  (id col)   │  │  (salary)   │           │
│  │  ┌────────┐ │  │  ┌────────┐ │           │
│  │  │  Page  │ │  │  │  Page  │ │           │
│  │  │  Page  │ │  │  │  Page  │ │           │
│  │  └────────┘ │  │  └────────┘ │           │
│  └─────────────┘  └─────────────┘           │
├─────────────────────────────────────────────┤
│  Row Group 1                                │
├─────────────────────────────────────────────┤
│  File Metadata (schema + row group offsets) │
│  4-byte metadata length                     │
│  4-byte magic: PAR1                         │
└─────────────────────────────────────────────┘
```

### PySpark File I/O Best Practices

```python
# Reading Parquet (preferred format)
df = spark.read.parquet("hdfs://data/sales/")

# Predicate pushdown: filter pushed to file scan level
df_filtered = df.filter(F.col("year") == 2024)  # Only reads 2024 row groups

# Column pruning: only reads needed columns from disk
df_pruned = df.select("region", "amount")  # Only 2 columns read from Parquet

# Writing with partitioning (partition pruning on read)
df.write \
  .mode("overwrite") \
  .partitionBy("year", "month") \
  .parquet("hdfs://output/sales/")
# Creates: output/sales/year=2024/month=01/part-00001.parquet

# Reading with partition filter (only reads year=2024 folders)
df_2024 = spark.read.parquet("hdfs://output/sales/").filter(F.col("year") == 2024)

# Controlling file size
df.write \
  .option("maxRecordsPerFile", 1_000_000) \
  .parquet("output/")
```

### ⚠️ Edge Cases

- **Small files problem:** Writing 200 shuffle partitions to Parquet = 200 small files. Use `coalesce()` before writing or enable auto-compaction in Delta Lake.
- **Schema evolution:** Parquet supports adding new columns. Reading old files with a new schema works — missing columns return `null`. Removing columns requires rewriting.
- **Predicate pushdown doesn't work for all predicates.** Complex UDFs, Python UDFs, and some functions cannot be pushed to Parquet reader.
- **CSV schema inference reads the entire file** by default (or `samplingRatio` fraction). For large CSVs, always provide explicit schema.

### ❌ Common Mistakes

```python
# ❌ Writing to CSV for large datasets (no compression, no columnar benefits)
df.write.csv("output/")  # Huge files, slow reads

# ✅ Write as Parquet
df.write.parquet("output/")

# ❌ Not partitioning write output for frequently filtered columns
df.write.parquet("output/sales/")  # Every read scans ALL files

# ✅ Partition by frequently filtered columns
df.write.partitionBy("year", "month").parquet("output/sales/")
```

### 🎯 Interview Traps

> **"Does Spark always apply predicate pushdown for Parquet filters?"**  
> No. Pushdown only works for supported predicates and supported data types. Custom UDFs, `NOT IN` with large lists, and certain nested types may not be pushed down. Always verify with `explain()` — look for `PushedFilters` in the physical plan output.

---

## 🔗 Joins (Broadcast vs Shuffle)

### Join Types in Spark

| Join Strategy             | When Used                   | Shuffle?             | Memory Requirement                               |
| ------------------------- | --------------------------- | -------------------- | ------------------------------------------------ |
| **Broadcast Hash Join**   | Small table < threshold     | ❌ No                 | Broadcast table fits in driver + executor memory |
| **Shuffle Hash Join**     | Medium tables, non-sortable | ✅ Yes                | Hash table of one side fits in memory            |
| **Sort Merge Join**       | Large tables, sortable keys | ✅ Yes                | No in-memory hash table needed                   |
| **Cartesian Join**        | No join key (CROSS JOIN)    | ✅ Yes                | Huge — avoid                                     |
| **Broadcast Nested Loop** | Non-equi joins              | ✅ Yes (or broadcast) | Small table broadcast                            |

### Broadcast Hash Join (BHJ)

```python
from pyspark.sql import functions as F

# Small lookup table (< spark.sql.autoBroadcastJoinThreshold = 10MB default)
countries = spark.read.parquet("countries/")  # 500 rows

# Large fact table
sales = spark.read.parquet("sales/")          # 500M rows

# Automatic broadcast (if countries < 10MB, Spark auto-broadcasts)
result = sales.join(countries, "country_code")

# Force broadcast manually (when Spark doesn't auto-detect)
result = sales.join(F.broadcast(countries), "country_code")

# Disable broadcast join
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", "-1")
```

```
BROADCAST HASH JOIN:
Driver:
  1. Collects 'countries' table entirely
  2. Serializes and broadcasts to ALL executors

Each Executor:
  3. Receives full 'countries' table → builds hash map in memory
  4. Scans local partition of 'sales'
  5. Looks up each sales row in local hash map

NO SHUFFLE of 'sales' data — each executor works independently!
```

### Sort Merge Join (SMJ)

```
SORT MERGE JOIN:
Phase 1 (SHUFFLE):
  Both tables shuffled by join key → same key goes to same partition

Phase 2 (SORT):
  Each partition sorted independently by join key

Phase 3 (MERGE):
  Two sorted streams merged like merge-sort
  (Only works because both sides are sorted — no hash table needed)

Memory efficient — reads data sequentially, no large hash table
But requires 2 shuffles + 2 sorts
```

### Join Performance Comparison

```python
# Check which join strategy Spark chose
sales.join(countries, "country_code").explain()

# Physical Plan shows:
# BroadcastHashJoin → best, no shuffle
# SortMergeJoin     → 2 shuffles, disk spill possible
# ShuffledHashJoin  → 1 shuffle, hash table in memory

# Hint to force join type (Spark 3.0+)
sales.join(countries.hint("broadcast"), "country_code")   # BHJ
sales.join(countries.hint("shuffle_merge"), "country_code")  # SMJ
sales.join(countries.hint("shuffle_hash"), "country_code")   # SHJ
```

### Join Types (SQL Semantics)

```python
# Inner join (default)
df1.join(df2, "key", "inner")

# Left outer
df1.join(df2, "key", "left")

# Right outer
df1.join(df2, "key", "right")

# Full outer
df1.join(df2, "key", "full_outer")

# Left semi (return only left rows that have a match — like EXISTS)
df1.join(df2, "key", "left_semi")

# Left anti (return only left rows with NO match — like NOT EXISTS)
df1.join(df2, "key", "left_anti")

# Cross join (cartesian — dangerous!)
df1.crossJoin(df2)  # Produces M × N rows
```

### ⚠️ Edge Cases

- **`autoBroadcastJoinThreshold` is based on estimated size, not actual size.** For tables with inaccurate statistics, Spark may wrongly broadcast a large table → OOM. Run `ANALYZE TABLE` to update stats.
- **Two large tables joining on the same partitioner = narrow join.** If both sides use a `HashPartitioner` with the same `numPartitions`, the join is co-located (no shuffle needed). This happens automatically for PairRDDs.
- **Broadcast join with `NULL` keys:** NULL keys never match in joins (including broadcast). Watch for unexpected data loss with nullable join keys.
- **Skew in joins:** If one key value has millions of records and others have few, the reducer for that key becomes a bottleneck. Use skew hints (AQE) or salting.

### ❌ Common Mistakes

```python
# ❌ Joining on non-indexed columns without any optimization
huge_df.join(other_huge_df, "low_cardinality_col")  # Creates massive skew

# ❌ Multiple joins without caching intermediate result
df1.join(df2, "key").join(df3, "key").join(df4, "key")
# If df1.join(df2) is expensive and reused, cache it first

# ❌ Not specifying join type — gets INNER join by default
df1.join(df2, "key")  # Silently drops non-matching rows
```

### 🎯 Interview Traps

> **"Does `df.join(other, 'key')` always shuffle both DataFrames?"**  
> No. If one side qualifies for broadcast (< `autoBroadcastJoinThreshold`), Spark performs a Broadcast Hash Join — only the small table is broadcasted (sent to all executors), and the large table is NOT shuffled. With AQE, Spark can also dynamically switch to BHJ at runtime even if statistics were initially wrong.

---

## 🧩 Spark Components

### Core Components Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SPARK ECOSYSTEM                              │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                   SPARK SQL / DataFrames                      │  │
│  │         (Structured API: SQL, DataFrames, Datasets)           │  │
│  └───────────────────────────────────────────────────────────────┘  │
│  ┌─────────────┐  ┌─────────────┐  ┌───────────┐  ┌────────────┐  │
│  │  Spark      │  │ Spark       │  │  MLlib    │  │  GraphX    │  │
│  │  Streaming  │  │  (Core RDD) │  │  (ML)     │  │  (Graph)   │  │
│  └─────────────┘  └─────────────┘  └───────────┘  └────────────┘  │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                     SPARK CORE                                │  │
│  │  (Task Scheduling, Memory Mgmt, Fault Tolerance, I/O)        │  │
│  └───────────────────────────────────────────────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐    │
│  │  Standalone  │  │    YARN      │  │  Mesos / Kubernetes    │    │
│  │  Scheduler   │  │  (Hadoop)    │  │                        │    │
│  └──────────────┘  └──────────────┘  └────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Key Components Deep Dive

#### SparkContext vs SparkSession

| Feature         | SparkContext         | SparkSession               |
| --------------- | -------------------- | -------------------------- |
| Introduced      | Spark 1.x            | Spark 2.0+                 |
| Entry point for | RDD operations       | DataFrames, SQL, Streaming |
| Contains        | Nothing higher-level | SparkContext internally    |
| Preferred for   | Legacy RDD code      | All modern Spark code      |
| Config          | `SparkConf`          | `.config()` builder        |

```python
# SparkSession (modern approach — wraps SparkContext)
spark = SparkSession.builder \
    .appName("MyApp") \
    .master("yarn") \
    .config("spark.executor.memory", "4g") \
    .config("spark.executor.cores", "2") \
    .enableHiveSupport() \
    .getOrCreate()

# Access underlying SparkContext
sc = spark.sparkContext

# SparkSession creates only one session per JVM (singleton pattern)
spark2 = SparkSession.builder.getOrCreate()  # Returns same session
assert spark is spark2  # True
```

#### Memory Management

```
EXECUTOR MEMORY LAYOUT:
┌────────────────────────────────────────────────────┐
│            spark.executor.memory = 4g              │
│                                                    │
│  ┌────────────────────────────────────────────┐    │
│  │  Reserved Memory (300MB fixed)             │    │
│  └────────────────────────────────────────────┘    │
│  ┌────────────────────────────────────────────┐    │
│  │  Usable Memory = (4g - 300MB) × 0.6 heap  │    │
│  │                                            │    │
│  │  ┌──────────────────────────────────────┐  │    │
│  │  │  Execution Memory (60% of usable)    │  │    │
│  │  │  (Shuffle, Sort, Join hash tables)   │  │    │
│  │  └──────────────────────────────────────┘  │    │
│  │  ┌──────────────────────────────────────┐  │    │
│  │  │  Storage Memory (40% of usable)      │  │    │
│  │  │  (Cached RDDs/DataFrames)            │  │    │
│  │  └──────────────────────────────────────┘  │    │
│  │  (Execution can borrow from Storage)        │    │
│  └────────────────────────────────────────────┘    │
│  ┌────────────────────────────────────────────┐    │
│  │  User Memory (remaining heap)              │    │
│  │  (Python UDFs, data structures, etc.)      │    │
│  └────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────┘
```

#### Shuffle Service & External Shuffle Service

```python
# Enable external shuffle service (allows executor decommission without losing shuffle files)
spark.conf.set("spark.shuffle.service.enabled", "true")

# Tungsten: Spark's in-house binary memory format (bypasses Java objects)
# Automatic in DataFrames — no user config needed
# Benefits: less GC pressure, cache-friendly memory layout, SIMD optimization
```

### ⚠️ Edge Cases

- **SparkContext is NOT thread-safe.** Don't share it across threads. SparkSession IS thread-safe and designed for multi-threaded use.
- **One SparkContext per JVM.** Trying to create a second one throws an exception unless the first is stopped.
- **MLlib has two APIs:** `spark.ml` (DataFrame-based, preferred) and `spark.mllib` (RDD-based, deprecated). Don't confuse them.

### 🎯 Interview Traps

> **"Is SparkSession a superset of SparkContext?"**  
> Yes — SparkSession internally holds a SparkContext and additionally provides the DataFrame API, Catalog API, and Hive integration. For modern Spark (2.0+), you should only create a SparkSession; never create SparkContext directly unless for RDD-only work.

---

## ⚙️ Execution Hierarchy (Job → Stage → Task → Slot)

### The Full Hierarchy

```
APPLICATION
└── JOB (triggered by one action, e.g., count())
    ├── STAGE 1 (ends at first shuffle boundary)
    │   ├── TASK 0 → runs on Executor 1, Core 1 (Slot)
    │   ├── TASK 1 → runs on Executor 1, Core 2 (Slot)
    │   └── TASK 2 → runs on Executor 2, Core 1 (Slot)
    └── STAGE 2 (starts after shuffle)
        ├── TASK 0 → runs on Executor 2, Core 2 (Slot)
        ├── TASK 1 → runs on Executor 3, Core 1 (Slot)
        └── TASK 2 → runs on Executor 3, Core 2 (Slot)
```

### Detailed Definitions

| Unit            | Definition                                                   | Count determined by                |
| --------------- | ------------------------------------------------------------ | ---------------------------------- |
| **Application** | One SparkSession lifecycle                                   | One per `spark-submit`             |
| **Job**         | Computation triggered by one action                          | Number of action calls             |
| **Stage**       | Set of tasks with no shuffle boundary between them           | Number of wide transformations + 1 |
| **Task**        | Unit of work applied to one partition                        | Number of partitions               |
| **Slot**        | One available unit of parallelism (one core of one executor) | `num_executors × executor_cores`   |

### Total Slots Calculation

```
Total Slots = Number of Executors × Executor Cores per Executor

Example:
  10 executors × 4 cores each = 40 slots (40 tasks can run simultaneously)

Parallelism:
  If you have 200 partitions and 40 slots:
  200 / 40 = 5 waves of tasks
  Ideal: num_partitions ≈ 2-4× total_slots (keeps all slots busy)
```

### PySpark Example

```python
# Configure executor resources
spark = SparkSession.builder \
    .config("spark.executor.instances", "10") \
    .config("spark.executor.cores", "4") \
    .config("spark.executor.memory", "8g") \
    .getOrCreate()

# Total slots = 10 × 4 = 40
# If RDD has 200 partitions → 200 tasks → 5 waves

rdd = sc.parallelize(range(1_000_000), 200)  # 200 partitions → 200 tasks

# One action = one job
rdd.count()  # Job 0: 1 stage, 200 tasks

# Two actions = two jobs
rdd.count()        # Job 0
rdd.sum()          # Job 1

# One action after wide transformation = multiple stages in one job
rdd.map(lambda x: (x % 10, x)) \
   .reduceByKey(lambda a, b: a + b) \
   .count()  # Job 0: Stage 0 (map, partial reduce) + Stage 1 (final reduce)
```

### What Happens in Each Scenario

```
Scenario: More Slots Than Tasks
  40 slots, 10 tasks → only 10 slots used, 30 slots IDLE
  Tasks are NOT split further — task is atomic
  Solution: increase partition count

Scenario: Fewer Partitions Than Cores
  Same as above — just with different framing
  4 executors × 4 cores = 16 slots
  With 8 partitions → 8 tasks → only 8/16 slots busy (50% utilization)

Scenario: Executor Fails During Stage
  Tasks on that executor are marked FAILED
  Spark retries those tasks on other executors (default: 4 retries)
  If failure was at shuffle write → downstream tasks also fail and retry
  If stage was checkpointed → restart from checkpoint

Scenario: Driver Fails
  ENTIRE JOB FAILS — no recovery possible (except with cluster-mode + supervise)
  All executor results are lost
```

### ⚠️ Edge Cases

- **One action can trigger MULTIPLE jobs.** For example, `df.write.parquet()` may internally trigger multiple jobs (one for file writing, one for metadata/manifest updates in Delta Lake).
- **Speculative execution:** Spark can launch a second copy of a slow task (`spark.speculation = true`). The first one to finish wins, and the other is killed. This can cause double-writes for non-idempotent operations.
- **Task locality levels:** `PROCESS_LOCAL > NODE_LOCAL > RACK_LOCAL > ANY`. Spark waits for better locality before accepting worse (`spark.locality.wait = 3s` default).

### ❌ Common Mistakes

```python
# ❌ Thinking one action = one stage
df.groupBy("key").count().show()
# This is: 1 action → 1 job → 2 stages (before and after shuffle)

# ❌ Setting executor cores too high causes GC pressure
# spark.executor.cores = 8 (8 tasks sharing 8GB RAM → 1GB each → often too low)
# Recommended: 2-5 cores per executor for balanced throughput/memory

# ❌ Under-configuring parallelism
spark.conf.set("spark.default.parallelism", "2")  # Only 2 tasks for RDD ops — terrible
```

### 🎯 Interview Traps

> **"If a task fails, does the entire stage restart?"**  
> No. Only the **failed task** is retried (up to `spark.task.maxFailures` times, default 4). The stage fails only if all retries for a task are exhausted. Other successful tasks in the stage are NOT re-run. However, if a shuffle stage's output is lost (executor with shuffle files died), the upstream stage producing those files is re-run.

---

## 🚀 Deployment Modes

### Modes Overview

| Mode                   | Driver Location             | When to Use                        |
| ---------------------- | --------------------------- | ---------------------------------- |
| **Local**              | Same machine as tasks       | Development, testing               |
| **Standalone Client**  | Developer machine (client)  | Interactive, debugging             |
| **Standalone Cluster** | Worker node in cluster      | Production (client can disconnect) |
| **YARN Client**        | Developer machine           | Interactive on YARN                |
| **YARN Cluster**       | ApplicationMaster container | Production on Hadoop               |
| **Kubernetes**         | Pod in K8s cluster          | Cloud-native, containerized        |
| **Mesos**              | Mesos scheduler             | Legacy enterprise                  |

### Client vs Cluster Mode

```
CLIENT MODE:
  Client Machine                 Cluster
  ──────────────                 ────────────────────────────
  ┌────────────┐  submits        ┌──────────────────────────┐
  │   Driver   │ ──────────────▶ │  Worker 1: Executor      │
  │  (here!)   │ ◀─────────────  │  Worker 2: Executor      │
  └────────────┘  results        │  Worker 3: Executor      │
                                 └──────────────────────────┘
  ⚠️ If client machine dies → job fails
  ✅ Good for interactive/debugging (see logs in real time)

CLUSTER MODE:
  Client Machine                 Cluster
  ──────────────                 ────────────────────────────
  ┌────────────┐  submits        ┌──────────────────────────┐
  │  Submit    │ ──────────────▶ │  AM/Worker: Driver       │
  │  (then     │                 │  Worker 1: Executor      │
  │  exits)    │                 │  Worker 2: Executor      │
  └────────────┘                 └──────────────────────────┘
  ✅ Client can disconnect — job runs independently
  ✅ Production-safe: driver failure handled by cluster restart
```

### spark-submit Examples

```bash
# Local mode (all on one machine, 4 threads)
spark-submit \
  --master local[4] \
  my_job.py

# YARN cluster mode (production)
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --num-executors 20 \
  --executor-cores 4 \
  --executor-memory 8g \
  --driver-memory 4g \
  --conf spark.sql.shuffle.partitions=400 \
  my_job.py

# Kubernetes
spark-submit \
  --master k8s://https://k8s-cluster:8443 \
  --deploy-mode cluster \
  --conf spark.kubernetes.container.image=my-spark:3.4 \
  my_job.py

# Standalone cluster
spark-submit \
  --master spark://master-host:7077 \
  --deploy-mode cluster \
  my_job.py
```

### PySpark Example — SparkSession for YARN

```python
from pyspark.sql import SparkSession

# In cluster mode, configuration is set via spark-submit flags
# or in the code (code configs override submit flags for session-level settings)
spark = SparkSession.builder \
    .appName("ProductionJob") \
    .config("spark.sql.adaptive.enabled", "true") \
    .config("spark.sql.adaptive.coalescePartitions.enabled", "true") \
    .config("spark.speculation", "true") \
    .getOrCreate()
```

### ⚠️ Edge Cases

- **In YARN cluster mode, `--files` paths must be accessible from the cluster** (e.g., HDFS), not just the submitting client.
- **Python dependencies in cluster mode:** Use `--py-files` to ship `.zip` or `.egg` files. For complex environments, use Docker (K8s) or a pre-built Conda env on workers.
- **Driver OOM in client mode:** If `collect()` brings too much data to the driver and the client machine is underpowered, you get an OOM on your laptop.

### 🎯 Interview Traps

> **"Is `local[*]` the same as `local[4]` on a 4-core machine?"**  
> `local[*]` uses as many threads as logical cores available at runtime. On a 4-core machine, `local[*]` and `local[4]` behave the same. But `local[*]` is environment-adaptive — on an 8-core machine, it uses 8 threads. `local` (no number) uses just 1 thread (no parallelism).

---

## 🧠 Catalyst Optimizer

### Definition
The **Catalyst Optimizer** is Spark's extensible query optimization framework that transforms logical query plans into efficient physical execution plans through a set of rule-based and cost-based optimizations.

### Catalyst Pipeline

```
SQL Query / DataFrame API
          │
          ▼
┌─────────────────────────────┐
│    UNRESOLVED LOGICAL PLAN  │ ← Column names unresolved, no types
│    (AST - Abstract Syntax   │
│     Tree)                   │
└─────────────┬───────────────┘
              │  Analysis phase (uses Catalog: schema, table metadata)
              ▼
┌─────────────────────────────┐
│    RESOLVED LOGICAL PLAN    │ ← All columns/types resolved
│    (Analyzed)               │
└─────────────┬───────────────┘
              │  Logical Optimization
              │  (rule-based: predicate pushdown, constant folding,
              │   column pruning, null propagation...)
              ▼
┌─────────────────────────────┐
│   OPTIMIZED LOGICAL PLAN    │
└─────────────┬───────────────┘
              │  Physical Planning
              │  (multiple physical plan candidates generated)
              ▼
┌─────────────────────────────┐
│   PHYSICAL PLANS (multiple) │
│   Plan 1: SortMergeJoin     │
│   Plan 2: BroadcastHashJoin │
└─────────────┬───────────────┘
              │  Cost Model (CBO if stats available)
              ▼
┌─────────────────────────────┐
│   SELECTED PHYSICAL PLAN    │
└─────────────┬───────────────┘
              │  Code Generation (Tungsten Whole-Stage CodeGen)
              ▼
       JVM Bytecode → Execution
```

### Key Optimizations Catalyst Performs

| Optimization               | What it does                                   | Example                                       |
| -------------------------- | ---------------------------------------------- | --------------------------------------------- |
| **Predicate Pushdown**     | Moves filters as early as possible             | `WHERE` on large table pushed to scan         |
| **Column Pruning**         | Reads only needed columns                      | `SELECT a, b` → only a,b columns from Parquet |
| **Constant Folding**       | Evaluates constant expressions at compile time | `1 + 1` → `2` in plan                         |
| **Null Propagation**       | Simplifies null logic                          | `null + x = null` always                      |
| **Boolean Simplification** | Simplifies boolean expressions                 | `true AND x = x`                              |
| **Join Reordering (CBO)**  | Reorders joins based on table stats            | Small table join first                        |
| **Subquery Elimination**   | Converts subqueries to joins                   | `EXISTS` → `LEFT SEMI JOIN`                   |

### PySpark — Viewing Optimizer Output

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F

spark = SparkSession.builder.appName("CatalystDemo").getOrCreate()

sales = spark.read.parquet("sales/")
products = spark.read.parquet("products/")

# Query
result = sales \
    .join(products, "product_id") \
    .filter(F.col("category") == "Electronics") \
    .select("sale_id", "product_name", "amount")

# View all plan stages
result.explain(mode="extended")
# Shows: Parsed → Analyzed → Optimized → Physical

# Check if predicate pushdown occurred
result.explain()
# In Physical Plan, look for:
# PushedFilters: [IsNotNull(category), EqualTo(category,Electronics)]
# This means filter was pushed to the file scan level!

# Enable CBO (Cost-Based Optimization)
spark.conf.set("spark.sql.cbo.enabled", "true")
spark.conf.set("spark.sql.statistics.histogram.enabled", "true")

# Analyze table stats for CBO
spark.sql("ANALYZE TABLE sales COMPUTE STATISTICS FOR ALL COLUMNS")
```

### Adaptive Query Execution (AQE) — Catalyst at Runtime

```python
# AQE: Catalyst re-optimizes AT RUNTIME based on actual shuffle stats
spark.conf.set("spark.sql.adaptive.enabled", "true")

# AQE Feature 1: Dynamically coalesce shuffle partitions
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
# 200 shuffle partitions → AQE sees most are tiny → merges to 5 large ones

# AQE Feature 2: Dynamic switch to broadcast join
spark.conf.set("spark.sql.adaptive.localShuffleReader.enabled", "true")
# If after shuffle, one side is small → convert SortMergeJoin to BroadcastHashJoin

# AQE Feature 3: Skew join optimization
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
# Splits skewed partitions into smaller sub-partitions automatically
```

### ⚠️ Edge Cases

- **Catalyst cannot optimize non-deterministic functions.** `rand()`, `uuid()`, and custom Python UDFs break optimization — they must be re-evaluated each time.
- **CBO requires up-to-date statistics.** If you never run `ANALYZE TABLE`, CBO falls back to rule-based optimization and may choose suboptimal join orders.
- **AQE and predicate pushdown work at different phases.** Predicate pushdown is a static optimization; AQE is dynamic. They are complementary.

### ❌ Common Mistakes

```python
# ❌ Using Python UDF kills Catalyst optimization for that column
@udf(returnType=StringType())
def my_func(x): return x.strip()
df.filter(my_func(F.col("name")) == "Alice")  # Cannot push filter to scan

# ✅ Use built-in Spark functions
df.filter(F.trim(F.col("name")) == "Alice")  # Can be optimized

# ❌ Caching before all filters are applied
df.cache()
df_filtered = df.filter(F.col("year") == 2024)
# Cache stores ALL data, but you only need 2024!

# ✅ Filter first, then cache
df_filtered = df.filter(F.col("year") == 2024).cache()
```

### 🎯 Interview Traps

> **"Does Catalyst guarantee the fastest possible plan?"**  
> No. Catalyst uses a set of heuristic rules and (optionally) a cost model, but it doesn't explore ALL possible plans exhaustively. It can miss optimizations, especially without accurate table statistics. AQE partially addresses this by re-optimizing at runtime, but it's still not optimal in all cases.

---

## ⚡ spark.sql.shuffle.partitions

### Definition
`spark.sql.shuffle.partitions` controls the **number of partitions** created after a shuffle operation (like `groupBy`, `join`, `distinct`) in the DataFrame/SQL API. The default is **200**.

```python
# Default: 200 shuffle partitions
spark.conf.get("spark.sql.shuffle.partitions")  # "200"

# Change it
spark.conf.set("spark.sql.shuffle.partitions", "50")
```

### Why Default 200 Can Hurt Performance

```
SCENARIO 1: Small dataset (10MB) with default 200 shuffle partitions
  → 200 tasks created after shuffle
  → Each task processes ~50KB of data (nearly zero actual work)
  → Task scheduling overhead >> actual computation
  → Job takes 30 seconds when it should take 1 second

SCENARIO 2: Large dataset (10TB) with default 200 shuffle partitions
  → 200 tasks created after shuffle
  → Each task processes ~50GB of data (way too much for one task!)
  → Disk spill, OOM, extremely slow
  → Job should use 5000+ partitions

SCENARIO 3: AQE enabled (best case)
  → 200 initial partitions
  → AQE observes actual data sizes
  → Dynamically coalesces to optimal N partitions
  → No manual tuning needed (mostly)
```

### Tuning Guide

```python
# Rule of thumb for manual tuning:
# Target partition size: 128MB - 256MB
# shuffle_partitions = total_shuffle_data_size / target_partition_size

# Example:
# 1TB shuffle data, target 200MB per partition
# shuffle_partitions = 1,000,000MB / 200MB = 5,000

spark.conf.set("spark.sql.shuffle.partitions", "5000")

# With AQE (Spark 3.0+): let Spark decide
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.minPartitionSize", "64mb")
spark.conf.set("spark.sql.adaptive.advisoryPartitionSizeInBytes", "128mb")
# Now start with a high number (e.g., 1000) and AQE coalesces down

# Note: spark.sql.shuffle.partitions ≠ spark.default.parallelism
# spark.default.parallelism: controls RDD operations (not DataFrame)
# spark.sql.shuffle.partitions: controls DataFrame/SQL shuffle operations
```

### Key Distinction

| Config                         | Affects                                 | Default                  |
| ------------------------------ | --------------------------------------- | ------------------------ |
| `spark.sql.shuffle.partitions` | DataFrame/SQL shuffle output partitions | 200                      |
| `spark.default.parallelism`    | RDD operations (e.g., `reduceByKey`)    | Depends on cluster cores |

```python
# RDD API uses spark.default.parallelism
sc.parallelize(data).reduceByKey(f)  # Uses spark.default.parallelism

# DataFrame API uses spark.sql.shuffle.partitions
df.groupBy("key").count()  # Uses spark.sql.shuffle.partitions
```

### ⚠️ Edge Cases

- **AQE's coalescing only works after the first shuffle.** The initial `spark.sql.shuffle.partitions` value is still used to write shuffle files — AQE just merges them for the read side. So setting it to 1,000,000 is wasteful (creates 1M small files to write, even if AQE merges them for reading).
- **Setting this too low AND disabling AQE is dangerous** for large datasets — causes massive per-partition work and potential OOM.
- **`repartition()` within a DataFrame job also uses this setting** when no explicit partition count is given.
- **Changing `spark.sql.shuffle.partitions` mid-job** (after some stages have run) only affects future shuffles, not already-completed ones.

### ❌ Common Mistakes

```python
# ❌ Never changing it for small data jobs
spark.conf.set("spark.sql.shuffle.partitions", "200")  # Default kept for 1MB dataset
# Creates 200 near-empty partitions — huge overhead

# ❌ Setting it globally for all jobs in a shared cluster
# Different jobs need different values!

# ✅ Set it per SparkSession or use AQE
spark.conf.set("spark.sql.shuffle.partitions", "50")  # Set for this specific job

# ✅ Best practice: enable AQE and use a high initial value
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.shuffle.partitions", "500")
# AQE will coalesce down to the optimal number
```

### 🎯 Interview Traps

> **"Can `spark.sql.shuffle.partitions` = 200 ever be the right value?"**  
> Yes — for datasets that happen to produce ~128-256MB per partition with 200 partitions. But this is coincidental, not by design. The right value is always dataset-specific and should be tuned per job. With AQE enabled, this becomes much less critical.

---

## 🧪 Advanced Practice MCQs

### Part 1: Lazy Evaluation & Actions

**Q1.** You write the following code. How many times is `textFile` read from HDFS?
```python
rdd = sc.textFile("hdfs://data.txt")
rdd.filter(lambda x: "error" in x).count()
rdd.filter(lambda x: "warn" in x).count()
```

- A) 0 times (Spark uses caching automatically)  
- B) 1 time  
- C) 2 times ✅  
- D) 4 times  

**Explanation:** Each action triggers a full re-execution from the source. Without explicit `.cache()`, `textFile` is read twice — once per action.

---

**Q2.** Which of the following is NOT a lazy operation?

- A) `df.filter(F.col("x") > 5)`  
- B) `df.select("a", "b")`  
- C) `df.show()` ✅  
- D) `df.groupBy("k")`  

**Explanation:** `show()` is an action. It triggers computation and prints rows. All others are transformations.

---

**Q3.** What does `df.printSchema()` trigger?

- A) Full data scan  
- B) Schema inference scan (reads sample rows for CSV/JSON, reads metadata for Parquet) ✅  
- C) Nothing — schema is always already known  
- D) A new Spark job  

**Explanation:** For Parquet, schema is in file metadata (no data read). For CSV, Spark samples rows. This is NOT an action in the traditional sense — it doesn't return a computed value from data.

---

**Q4.** How many jobs does this code create?
```python
rdd = sc.parallelize(range(100))
rdd.map(lambda x: x * 2).filter(lambda x: x > 50).collect()
```

- A) 3 (one per transformation + one for action)  
- B) 1 ✅  
- C) 2 (one for map, one for filter+collect)  
- D) 0 (transformations don't create jobs)  

**Explanation:** One action (`collect()`) = one job. All transformations are part of the same job's stage(s).

---

**Q5.** You call `df.cache()` and then `df.count()`. Then you call `df.show()`. How many times is the original data source read?

- A) 2  
- B) 3  
- C) 1 ✅  
- D) Depends on storage level  

**Explanation:** `df.count()` fills the cache. `df.show()` uses the cache — source is only read once (unless cache was evicted due to memory pressure).

---

### Part 2: Transformations & Shuffles

**Q6.** Which of the following transformations creates a stage boundary?

- A) `map()`  
- B) `filter()`  
- C) `flatMap()`  
- D) `groupByKey()` ✅  

**Explanation:** Wide transformations (those requiring a shuffle) create stage boundaries. `groupByKey()` requires data from all partitions to be consolidated by key.

---

**Q7.** You have an RDD with a `HashPartitioner(10)`. You apply `filter()`. What is the partitioner of the result?

- A) `HashPartitioner(10)` (preserved)  
- B) `None` ✅  
- C) `HashPartitioner(5)` (halved)  
- D) `RangePartitioner(10)`  

**Explanation:** `filter()` is a narrow transformation but it **destroys the partitioner** because it may remove keys — Spark can't guarantee the partitioning invariant after filtering. Check with `rdd.partitioner`.

---

**Q8.** `rdd.reduceByKey(f)` is preferred over `rdd.groupByKey().mapValues(lambda vs: reduce(f, vs))` because:

- A) `reduceByKey` is a narrow transformation  
- B) `reduceByKey` performs map-side aggregation (combine before shuffle) ✅  
- C) `groupByKey` always causes more stages  
- D) `reduceByKey` preserves the partitioner  

**Explanation:** `reduceByKey` performs partial aggregation within each partition BEFORE shuffling — massively reducing data transfer. `groupByKey` shuffles ALL raw values.

---

**Q9.** How many shuffles does `rdd.distinct()` perform?

- A) 0  
- B) 1 ✅  
- C) 2  
- D) Depends on data  

**Explanation:** `distinct()` maps each element to `(element, null)`, then calls `reduceByKey` — which requires one shuffle.

---

**Q10.** `coalesce(5)` vs `repartition(5)` when reducing from 200 partitions. Which statement is TRUE?

- A) Both always produce balanced partitions  
- B) `coalesce(5)` may produce unbalanced partitions; `repartition(5)` produces balanced ones ✅  
- C) `coalesce(5)` shuffles data; `repartition(5)` does not  
- D) They produce identical results  

**Explanation:** `coalesce()` merges adjacent partitions without shuffling → may be very unbalanced. `repartition()` shuffles data → balanced distribution.

---

### Part 3: Execution Hierarchy

**Q11.** You have 5 executors with 3 cores each. You submit a job with one stage of 200 tasks. How many waves of tasks are executed?

- A) 200  
- B) 40  
- C) 14 (ceil(200/15)) ✅  
- D) 1  

**Explanation:** Total slots = 5 × 3 = 15. Waves = ceil(200/15) = ceil(13.33) = 14 waves.

---

**Q12.** A task fails 3 times. The default `spark.task.maxFailures` is 4. What happens?

- A) Job fails immediately  
- B) Task is retried one more time (4th attempt) ✅  
- C) Stage is restarted from scratch  
- D) Executor is killed  

**Explanation:** `spark.task.maxFailures = 4` means Spark retries up to 4 times total. After 3 failures, one retry remains.

---

**Q13.** An executor holding shuffle output files crashes AFTER Stage 1 completes but BEFORE Stage 2 reads those files. What happens?

- A) Stage 2 fails and job fails  
- B) Stage 1 is re-run to regenerate shuffle files, then Stage 2 proceeds ✅  
- C) Spark uses lineage to regenerate only the lost partition  
- D) Stage 2 reads from a backup shuffle location  

**Explanation:** Shuffle files are written to local executor disk. If that executor dies, the shuffle files are lost. The upstream stage (Stage 1) must be re-run to regenerate them.

---

**Q14.** Which is NOT a task locality level in Spark?

- A) `PROCESS_LOCAL`  
- B) `NODE_LOCAL`  
- C) `EXECUTOR_LOCAL` ✅  
- D) `RACK_LOCAL`  

**Explanation:** Spark's locality levels are: `PROCESS_LOCAL`, `NODE_LOCAL`, `NO_PREF`, `RACK_LOCAL`, `ANY`. There is no `EXECUTOR_LOCAL` level.

---

**Q15.** A Spark job has 3 wide transformations. How many stages does it have?

- A) 3  
- B) 4 ✅  
- C) Depends on the action  
- D) 2  

**Explanation:** N wide transformations = N shuffle boundaries = N+1 stages. 3 wide transformations → 4 stages.

---

### Part 4: Joins

**Q16.** Spark's `autoBroadcastJoinThreshold` is 10MB. One side of your join is 15MB after stats are analyzed. Will Spark use a Broadcast Hash Join?

- A) Yes, because 15MB is still relatively small  
- B) No ✅  
- C) Only with AQE enabled  
- D) Depends on the join type  

**Explanation:** The threshold is a hard cutoff. 15MB > 10MB → no auto-broadcast. You must either increase the threshold or use `F.broadcast()` hint manually.

---

**Q17.** Which join type returns only rows from the LEFT table that have NO matching row in the RIGHT table?

- A) `LEFT OUTER`  
- B) `LEFT SEMI`  
- C) `LEFT ANTI` ✅  
- D) `INNER`  

**Explanation:** `LEFT ANTI` is the opposite of `LEFT SEMI`. It returns left rows with no match — functionally equivalent to `NOT EXISTS`.

---

**Q18.** Two large DataFrames use Sort Merge Join. How many shuffles occur?

- A) 0  
- B) 1  
- C) 2 ✅  
- D) 3  

**Explanation:** Sort Merge Join requires both sides to be shuffled (partitioned by join key), then sorted within each partition. That's one shuffle per side = 2 shuffles total.

---

**Q19.** You force `F.broadcast(large_df)` where `large_df` is 5GB. What happens?

- A) Spark ignores the hint and uses Sort Merge Join  
- B) The driver OOMs trying to collect 5GB ✅  
- C) Spark automatically breaks it into chunks  
- D) AQE detects the size and cancels the broadcast  

**Explanation:** Broadcast joins require the small table to be collected on the driver and then sent to every executor. Forcing broadcast on a 5GB table will likely crash the driver with OOM.

---

**Q20.** When do two RDDs join as a narrow (not wide) transformation?

- A) Always — all joins are wide  
- B) When both have the same `HashPartitioner` with the same number of partitions ✅  
- C) When one is broadcastable  
- D) Never — joins always shuffle  

**Explanation:** Co-partitioned RDDs (same partitioner, same numPartitions) join locally — no data needs to cross partition boundaries. Spark recognizes this and skips the shuffle.

---

### Part 5: Partitioning

**Q21.** You call `spark.conf.set("spark.sql.shuffle.partitions", "200")`. You then call `rdd.reduceByKey(f)`. How many output partitions does `reduceByKey` produce?

- A) 200  
- B) The default parallelism (`spark.default.parallelism`) ✅  
- C) Same as input partitions  
- D) 1  

**Explanation:** `spark.sql.shuffle.partitions` only affects DataFrame/SQL operations. RDD operations use `spark.default.parallelism`. These are separate configurations.

---

**Q22.** You have 10 partitions and 100 cores. What is the cluster utilization during this stage?

- A) 100%  
- B) 10% ✅  
- C) 50%  
- D) Cannot determine  

**Explanation:** Only 10 tasks can run (one per partition), on 10 of the 100 cores. The other 90 cores sit idle = 10% utilization.

---

**Q23.** `df.repartition(50, "country")` creates partitions where:

- A) Each partition has exactly the same number of rows  
- B) All rows with the same `country` value go to the same partition ✅  
- C) Rows are sorted by `country` within each partition  
- D) 50 partitions are created, each with a random sample of all countries  

**Explanation:** `repartition(n, col)` uses hash partitioning on `col` — all same-value rows hash to the same partition. But multiple country values can end up in the same partition (hash collisions).

---

**Q24.** After calling `df.write.partitionBy("year").parquet("output/")`, how many output files are created if `df` has 200 shuffle partitions and data for 3 years?

- A) 3  
- B) 200  
- C) Up to 200 × 3 = 600 ✅  
- D) 1  

**Explanation:** `partitionBy` in write creates directory structure (year=2022, year=2023, year=2024). Each Spark partition (task) can write to any year directory it has data for → up to 200 tasks × 3 directories = 600 files in worst case.

---

**Q25.** Which is the SAFEST way to handle a severely skewed key in a join?

- A) Increase `spark.sql.shuffle.partitions`  
- B) Use AQE skew join optimization or manually salt the key ✅  
- C) Use `coalesce()` before the join  
- D) Use `repartition()` with a different column  

**Explanation:** AQE can automatically split skewed partitions, or you can use key salting (adding a random prefix to split one hot key across multiple partitions). Neither `coalesce` nor general `repartition` address the root cause (hot keys).

---

### Part 6: Catalyst & Performance

**Q26.** Which optimization does Catalyst perform that CANNOT be done with RDDs?

- A) Lazy evaluation  
- B) Predicate pushdown ✅  
- C) Fault tolerance  
- D) Data partitioning  

**Explanation:** Catalyst's predicate pushdown is a query plan optimization — it moves filter conditions closer to the data source (e.g., into Parquet file scans). RDDs have no concept of a query plan — they're just function chains with no optimizer.

---

**Q27.** `explain()` shows `PushedFilters: []` for a Parquet scan. What does this mean?

- A) No filters were applied to this DataFrame  
- B) Filters exist but were NOT pushed down to the file scan level ✅  
- C) The Parquet file has no data  
- D) Catalyst is disabled  

**Explanation:** Empty `PushedFilters` means Catalyst could not push any filter predicates into the Parquet scan — perhaps because a Python UDF was used, or the filter is on a non-supported type.

---

**Q28.** You enable CBO (`spark.sql.cbo.enabled = true`) but forget to run `ANALYZE TABLE`. What happens?

- A) CBO throws an error  
- B) CBO uses estimated statistics (default/guessed values) and may make wrong decisions ✅  
- C) CBO is automatically disabled  
- D) CBO falls back to Catalyst rule-based optimization  

**Explanation:** Without `ANALYZE TABLE`, CBO lacks real statistics. It falls back to default size estimates, which may lead to suboptimal join ordering or strategy selection.

---

**Q29.** AQE dynamically switched your Sort Merge Join to a Broadcast Hash Join at runtime. What likely happened?

- A) You manually applied `F.broadcast()`  
- B) After the shuffle, one side of the join was smaller than `autoBroadcastJoinThreshold` ✅  
- C) The join key cardinality changed  
- D) AQE detected a skew and split the join  

**Explanation:** AQE observes actual shuffle output sizes. If after the first stage, one side is small enough to broadcast, AQE dynamically rewrites the remaining plan to use BHJ instead of SMJ.

---

**Q30.** Which of the following will BREAK Catalyst's column pruning optimization?

- A) `df.select("a", "b")`  
- B) `df.filter(F.col("a") > 0)`  
- C) `df.select("*")` ✅  
- D) `df.groupBy("a")`  

**Explanation:** `select("*")` (SELECT *) forces Spark to read ALL columns — there's nothing to prune. Always use explicit column selection (`select("col1", "col2")`) to enable column pruning in Parquet/ORC reads.

---

### Part 7: Edge Case Scenarios

**Q31.** `sc.parallelize([], 5).count()` returns:

- A) Error (empty RDD)  
- B) 0 ✅  
- C) 5  
- D) None  

**Explanation:** An empty RDD is valid. It has 5 partitions, each empty. `count()` returns 0. No error.

---

**Q32.** A DataFrame is cached. One executor fails and its cached partition is evicted. What does Spark do on next access?

- A) Fails with `CacheNotFoundException`  
- B) Returns `null` for rows in the lost partition  
- C) Recomputes the lost partition from lineage ✅  
- D) Skips the lost partition silently  

**Explanation:** RDD/DataFrame lineage is always preserved (even with cache). If a cached partition is lost, Spark recomputes it from the original lineage — maintaining correctness at the cost of recomputation.

---

**Q33.** What happens when `spark.speculation = true` and a task is running slowly?

- A) The slow task is killed and restarted  
- B) A duplicate copy of the task is launched; whichever finishes first wins ✅  
- C) The executor is replaced  
- D) Nothing — speculation only applies to streaming  

**Explanation:** Speculative execution launches a copy of slow tasks. The first to complete "wins" — the other is killed. This helps with stragglers but risks issues with non-idempotent write operations.

---

**Q34.** You have `spark.sql.shuffle.partitions = 200` and your DataFrame after shuffle has only 1MB of data. AQE is disabled. What is the likely result?

- A) Spark auto-reduces to the optimal partition count  
- B) 200 nearly empty partitions are created, causing scheduling overhead ✅  
- C) The job fails due to empty partitions  
- D) Spark automatically uses 1 partition  

**Explanation:** Without AQE, Spark creates exactly 200 partitions regardless of data size. 1MB / 200 = 5KB per partition — the task scheduling overhead alone dominates runtime.

---

**Q35.** A job has Stage 1 (Map) and Stage 2 (Reduce after shuffle). Stage 2 task fails to read shuffle output. Spark retries the Stage 2 task. The Stage 2 task retries fail because the shuffle file is on a dead executor. What happens next?

- A) The job fails immediately  
- B) Stage 1 is re-submitted to regenerate lost shuffle files, then Stage 2 retries ✅  
- C) Spark reads from a shuffle backup  
- D) The Stage 2 task marks those records as null  

**Explanation:** When a task fails because the shuffle data it needs is unavailable (dead executor), Spark schedules a re-run of the upstream stage that produced those shuffle files. This is lineage-based fault tolerance.

---

### ✅ Answer Key + Explanations

| Q#  | Answer                         | Core Concept                                        |
| --- | ------------------------------ | --------------------------------------------------- |
| Q1  | C — 2 times                    | Each action re-executes full lineage without cache  |
| Q2  | C — `show()`                   | `show()` is an action (triggers execution)          |
| Q3  | B — metadata/sample            | Schema inference depends on file format             |
| Q4  | B — 1                          | One action = one job                                |
| Q5  | C — 1                          | Cache prevents re-read on 2nd action                |
| Q6  | D — `groupByKey()`             | Wide transformation = stage boundary                |
| Q7  | B — None                       | `filter()` destroys partitioner                     |
| Q8  | B — map-side aggregation       | `reduceByKey` combines before shuffle               |
| Q9  | B — 1                          | `distinct()` = map + reduceByKey (1 shuffle)        |
| Q10 | B — unbalanced vs balanced     | coalesce = merge adjacent, repartition = shuffle    |
| Q11 | C — 14 waves                   | ceil(200/15) = 14                                   |
| Q12 | B — one retry remains          | maxFailures = 4 = 4 total attempts                  |
| Q13 | B — Stage 1 re-runs            | Shuffle files on dead executors must be regenerated |
| Q14 | C — EXECUTOR_LOCAL             | Not a real locality level                           |
| Q15 | B — 4 stages                   | N wide transforms = N+1 stages                      |
| Q16 | B — No                         | 15MB > 10MB threshold                               |
| Q17 | C — LEFT ANTI                  | Opposite of LEFT SEMI                               |
| Q18 | C — 2 shuffles                 | Sort Merge Join = 2 shuffles                        |
| Q19 | B — Driver OOM                 | 5GB broadcast = driver collects 5GB                 |
| Q20 | B — Same HashPartitioner       | Co-partitioned = narrow join                        |
| Q21 | B — spark.default.parallelism  | RDD uses different config than DataFrame            |
| Q22 | B — 10%                        | 10 tasks / 100 cores = 10% utilization              |
| Q23 | B — same-value rows co-located | Hash partitioning by column                         |
| Q24 | C — up to 600                  | 200 tasks × 3 year directories                      |
| Q25 | B — AQE/salting                | Salt key to distribute hot key                      |
| Q26 | B — Predicate pushdown         | RDDs have no query plan to optimize                 |
| Q27 | B — Filters NOT pushed         | Empty PushedFilters = no pushdown                   |
| Q28 | B — Wrong decisions            | CBO needs real stats                                |
| Q29 | B — Runtime size < threshold   | AQE converts SMJ → BHJ dynamically                  |
| Q30 | C — SELECT *                   | Star selection prevents column pruning              |
| Q31 | B — 0                          | Empty RDD returns 0 for count()                     |
| Q32 | C — Recomputes from lineage    | Lineage preserved through cache                     |
| Q33 | B — Duplicate launched         | Speculative execution = racing copies               |
| Q34 | B — 200 near-empty partitions  | No AQE = no auto-coalescing                         |
| Q35 | B — Stage 1 re-submitted       | Upstream stage regenerates shuffle files            |

---

## 🔥 Interview Cheat Sheet

### The 20 Most Important Facts

```
1.  Transformations are LAZY — only actions trigger execution
2.  Wide transforms = shuffle = stage boundary
3.  N wide transforms = N+1 stages in a job
4.  One action = one job
5.  Tasks = number of partitions (one task per partition)
6.  Total slots = num_executors × executor_cores
7.  Driver failure = job failure (single point of failure)
8.  Executor failure = task retry (fault tolerant via lineage)
9.  reduceByKey > groupByKey (map-side aggregation)
10. coalesce(n) = narrow (unbalanced); repartition(n) = wide (balanced)
11. DataFrame > RDD (Catalyst + Tungsten optimization)
12. cache() is lazy — data filled on first action
13. Broadcast join: small table → driver → all executors (no shuffle)
14. spark.sql.shuffle.partitions ≠ spark.default.parallelism
15. Default 200 shuffle partitions is almost always wrong
16. AQE dynamically optimizes: coalesces partitions, fixes skew, converts joins
17. Parquet: columnar, predicate pushdown, column pruning
18. CBO requires ANALYZE TABLE to have real statistics
19. Python UDFs break Catalyst optimization (use built-in functions)
20. More slots than tasks → idle slots (tasks NOT split further)
```

### Quick Reference: Key Configurations

| Config                                 | Default           | Purpose                             |
| -------------------------------------- | ----------------- | ----------------------------------- |
| `spark.sql.shuffle.partitions`         | 200               | DataFrame shuffle output partitions |
| `spark.default.parallelism`            | cluster-dependent | RDD shuffle output partitions       |
| `spark.sql.autoBroadcastJoinThreshold` | 10MB              | Max size for auto-broadcast join    |
| `spark.sql.adaptive.enabled`           | true (Spark 3.2+) | Enable AQE                          |
| `spark.executor.memory`                | 1g                | Memory per executor                 |
| `spark.executor.cores`                 | 1                 | Cores per executor                  |
| `spark.task.maxFailures`               | 4                 | Max task retry attempts             |
| `spark.locality.wait`                  | 3s                | Wait time for better data locality  |
| `spark.speculation`                    | false             | Enable speculative execution        |
| `spark.sql.cbo.enabled`                | false             | Enable cost-based optimization      |

### Transformation Decision Tree

```
Need to aggregate?
├── Small reduction (sum, avg, count) → Use DataFrame agg() or reduceByKey()
├── Complex aggregation → groupBy().agg() (DataFrame)
└── Key-value grouping (RDD) → reduceByKey() NOT groupByKey()

Need to join?
├── One side small (< 10MB) → Broadcast Hash Join (auto or F.broadcast())
├── Both large, sortable → Sort Merge Join (Spark default for large tables)
└── Both large, unsortable/skewed → Shuffle Hash Join + AQE skew join

Need to change partition count?
├── Reducing partitions → coalesce() (no shuffle, possibly unbalanced)
├── Increasing partitions → repartition() (always shuffles)
└── By column value → repartition(n, col) (hash partition by column)
```

---

## ⚡ Performance Optimization Tips

### 1. Use DataFrames Over RDDs
```python
# ❌ RDD: no Catalyst, Java serialization
rdd.map(lambda row: (row[0], row[1] * 2)).filter(lambda r: r[1] > 100)

# ✅ DataFrame: Catalyst + Tungsten binary format
df.withColumn("val_doubled", F.col("val") * 2).filter(F.col("val_doubled") > 100)
```

### 2. Enable AQE (Spark 3.0+)
```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
```

### 3. Use Broadcast Joins for Small Tables
```python
small_lookup = spark.read.parquet("lookup/")  # < 10MB
large_fact = spark.read.parquet("facts/")
result = large_fact.join(F.broadcast(small_lookup), "id")
```

### 4. Filter Early, Select Only Needed Columns
```python
# ✅ Filters and column selection pushed to scan level in Parquet
df.filter(F.col("year") == 2024) \
  .select("id", "amount", "region") \
  .groupBy("region") \
  .agg(F.sum("amount"))
```

### 5. Partition by Query Columns When Writing
```python
df.write.partitionBy("year", "month").parquet("output/")
# Reading: spark reads only relevant year/month directories
```

### 6. Use `reduceByKey` Over `groupByKey`
```python
# ✅ reduceByKey: combine locally, then shuffle reduced results
rdd.reduceByKey(lambda a, b: a + b)
```

### 7. Cache Strategically
```python
# Only cache when used 2+ times AND fits in memory
expensive_df = df.join(other, "key").filter(...).cache()
count = expensive_df.count()   # Fills cache
result = expensive_df.agg(...)  # Uses cache
expensive_df.unpersist()         # Free memory when done
```

### 8. Tune Parallelism
```python
# Set shuffle partitions based on data size
data_size_mb = 50_000  # 50GB
target_partition_mb = 200
optimal_partitions = data_size_mb // target_partition_mb  # 250

spark.conf.set("spark.sql.shuffle.partitions", str(optimal_partitions))
```

### 9. Avoid Small Files
```python
# Before writing: coalesce to reasonable file count
df.coalesce(50).write.parquet("output/")  # 50 files instead of 200
```

### 10. Use Vectorized UDFs (Pandas UDF) Instead of Python UDFs
```python
from pyspark.sql.functions import pandas_udf
import pandas as pd

# ✅ Pandas UDF: operates on batches (Arrow-based, ~10-100x faster than Python UDF)
@pandas_udf("double")
def multiply_pandas(s: pd.Series) -> pd.Series:
    return s * 2.5

df.withColumn("val_new", multiply_pandas(F.col("val")))
```

---

## 🚨 Common Pitfalls

### 1. Calling `collect()` on Large DataFrames
```python
# ❌ CRASH: pulls terabytes to driver
all_data = df.collect()

# ✅ Write to storage instead
df.write.parquet("output/")
# Or sample
sample = df.take(1000)
```

### 2. Repeated Actions Without Caching
```python
# ❌ Each action re-reads and recomputes everything
df.count()      # Full scan
df.show()       # Full scan again

# ✅ Cache before multiple actions
df.cache()
df.count()
df.show()
df.unpersist()
```

### 3. Using `groupByKey` Instead of `reduceByKey`
```python
# ❌ Shuffles ALL raw values → OOM risk
rdd.groupByKey().mapValues(sum)

# ✅ Partial aggregation before shuffle → much less data
rdd.reduceByKey(lambda a, b: a + b)
```

### 4. SELECT * Killing Column Pruning
```python
# ❌ Reads all columns from Parquet — even unused ones
df.select("*").groupBy("region").count()

# ✅ Only reads needed columns
df.select("region").groupBy("region").count()
```

### 5. Forgetting to Tune Shuffle Partitions
```python
# ❌ Default 200 for every job regardless of data size
# For small data: 200 is too many; for large data: 200 is too few

# ✅ Tune per job, or use AQE
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.shuffle.partitions", "500")  # AQE will coalesce down
```

### 6. Python UDFs Breaking Optimization
```python
# ❌ Python UDF breaks Catalyst — no pushdown, no vectorization
@udf(returnType=DoubleType())
def my_calc(x): return x * 2.5
df.filter(my_calc(F.col("val")) > 100)  # Cannot push filter to scan

# ✅ Use built-in functions
df.filter(F.col("val") * 2.5 > 100)  # Pushdown works!
```

### 7. Data Skew Causing Long-Tail Jobs
```python
# ❌ Hot key = one reducer gets 99% of data
df.groupBy("user_id").count()  # One user_id has 100M records

# ✅ Enable AQE skew join handling
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")

# Or manually salt the key
df.withColumn("salted_key", 
    F.concat(F.col("user_id"), F.lit("_"), (F.rand() * 10).cast("int")))
```

### 8. Broadcasting Large Tables → Driver OOM
```python
# ❌ Forcing broadcast on large table
result = huge_df.join(F.broadcast(also_huge_df), "key")  # Driver OOM

# ✅ Only broadcast truly small tables (< autoBroadcastJoinThreshold)
result = huge_df.join(F.broadcast(small_df), "key")
```

### 9. Ignoring Partition Count After coalesce()
```python
# ❌ coalesce() without checking can serialize entire job
df.coalesce(1).write.parquet("output/")  # Single file = single task = no parallelism!

# ✅ Use reasonable file count
df.coalesce(20).write.parquet("output/")
```

### 10. Not Using Explicit Schema for CSV/JSON
```python
# ❌ Schema inference reads the data twice (once to infer, once to process)
df = spark.read.csv("data.csv", header=True, inferSchema=True)

# ✅ Provide explicit schema
from pyspark.sql.types import *
schema = StructType([
    StructField("id", LongType()),
    StructField("name", StringType()),
    StructField("amount", DoubleType())
])
df = spark.read.schema(schema).csv("data.csv", header=True)
```

---

<div align="center">

**⭐ Star this repo if it helped you crack your Spark interview!**

*Built with ❤️ for Spark engineers and data enthusiasts*

</div>
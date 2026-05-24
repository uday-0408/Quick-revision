# Advanced SQL — Complete Syntax + How It Works Guide

> **How to use this file:** Every section gives you the exact syntax AND a plain-English explanation of what is actually happening under the hood when that command runs. Syntax is in code blocks; explanations are right below each block.

---

## Table of Contents
1. [SQL Performance Tuning](#1-sql-performance-tuning)
2. [Security Management](#2-security-management)
3. [Advanced SQL Features](#3-advanced-sql-features)
4. [PL/SQL Fundamentals](#4-plsql-fundamentals)
5. [PL/SQL Database Objects](#5-plsql-database-objects)
6. [Concurrency & Locking](#6-concurrency--locking)
7. [SQL Testing & Validation](#7-sql-testing--validation)
8. [Quick-Reference Summary](#8-quick-reference-summary)

---

## 1. SQL Performance Tuning

---

### 1.1 How the Oracle Cost-Based Optimizer (CBO) Works

Before looking at any syntax, understand what the optimizer does:

When you submit a SQL query, Oracle does **not** run it immediately. It first passes it through the **Cost-Based Optimizer (CBO)**, which:

1. **Parses** the SQL — checks syntax and resolves object names
2. **Transforms** it — rewrites it into an equivalent but potentially better form (e.g., converting a subquery to a JOIN)
3. **Generates candidate plans** — comes up with multiple ways to execute the query (different join orders, index vs full scan, etc.)
4. **Estimates cost** for each plan — uses statistics (row counts, column distributions, block counts) to predict how much I/O and CPU each plan needs
5. **Picks the cheapest plan** — executes that one

The key insight: **the CBO is only as good as its statistics**. If stats are stale, the optimizer picks the wrong plan. That is why `DBMS_STATS` is so important.

---

### 1.2 Viewing Execution Plans

An execution plan shows you the exact steps Oracle chose to run your query — which indexes it used, how it joined tables, in what order.

#### Oracle — EXPLAIN PLAN

```sql
-- Step 1: Ask Oracle to generate and store a plan (does NOT run the query)
EXPLAIN PLAN FOR
SELECT e.employee_name, d.department_name
FROM employees e JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 50000;

-- Step 2: Read the stored plan from the PLAN_TABLE
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

> **What happens:** `EXPLAIN PLAN FOR` makes the optimizer produce a plan and write it into a session-level table called `PLAN_TABLE`. `DBMS_XPLAN.DISPLAY` then formats that table into the human-readable tree you read. The query itself is never actually executed — you're only seeing what Oracle *would* do.

```sql
-- AUTOTRACE: runs the query AND shows the plan + real execution stats
SET AUTOTRACE ON EXPLAIN STATISTICS
SELECT * FROM employees WHERE department_id = 10;
```

> **What happens:** AUTOTRACE is a SQL\*Plus-only feature. It runs the query for real, then appends the execution plan and runtime statistics (like physical reads, logical reads, sorts) below the result. Much more useful than EXPLAIN PLAN alone because you see actual numbers, not estimates.

```sql
-- Real-time SQL monitoring (Enterprise Edition only)
SELECT DBMS_SQLTUNE.REPORT_SQL_MONITOR(sql_id => 'abc123', type => 'TEXT') FROM dual;
```

> **What happens:** For long-running queries, Oracle continuously updates an in-memory monitor. This function reads that monitor and gives you a live status of each step — how many rows each operation has processed so far, what % is done, and where time is being spent *right now*.

---

#### Snowflake — Query Profile

```sql
-- Get the ID of the last query you ran
SELECT LAST_QUERY_ID();

-- Pull execution metadata for that query
SELECT * FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY_BY_USER())
WHERE QUERY_ID = '<query_id>'
ORDER BY START_TIME DESC;
```

> **What happens:** Snowflake does not have a traditional execution plan. Instead, after a query runs, it records a **Query Profile** — a visual DAG (directed acyclic graph) of operators. You can view it in the Snowflake UI under Query History → Query Profile tab. The key metrics to check are `Partitions Scanned` vs `Partitions Total` (how much data pruning happened) and `Spillage to Disk` (warehouse ran out of memory).

---

#### Databricks / Spark SQL — EXPLAIN

```sql
EXPLAIN SELECT * FROM orders WHERE order_date = '2024-01-15';
-- Shows the physical plan in compact form

EXPLAIN EXTENDED SELECT ...;
-- Shows logical plan, optimized logical plan, and physical plan

EXPLAIN COST SELECT ...;
-- Adds estimated row counts and data sizes to the plan

EXPLAIN FORMATTED SELECT ...;
-- Most readable: tree view with stage boundaries clearly marked
```

> **What happens:** Spark compiles your SQL through multiple stages: SQL → Unresolved Logical Plan → Analyzed Logical Plan → Optimized Logical Plan → Physical Plan. `EXPLAIN` shows the *Physical Plan* — what Spark's execution engine will actually do. You want to look for `BroadcastHashJoin` (good, no shuffle) vs `SortMergeJoin` (expensive, requires shuffle across the cluster), and for `PushedFilters` to confirm that your WHERE clause is being pushed down to the file scan level.

---

### 1.3 Oracle Indexes

An index is a **separate, sorted data structure** stored alongside your table. Instead of scanning every row to find matches (full table scan), Oracle looks up the value in the index tree and gets the exact row location (ROWID) directly. This works like a book's index: you look up the word, get the page number, jump straight there.

```sql
-- B-Tree Index (default) — the standard, balanced tree index
-- Best for: high-cardinality columns (many distinct values), range queries, equality checks
CREATE INDEX emp_lastname_idx ON employees(last_name);
```

> **How a B-Tree works:** Values are stored in a sorted, balanced tree. Oracle traverses from the root node down to a leaf node that contains the search value, then reads the ROWID and fetches the actual row. The tree is kept balanced (all leaf nodes same depth) so lookup is always O(log n) regardless of table size.

```sql
-- Composite Index — index on multiple columns together
-- CRITICAL RULE: The leftmost column must appear in your WHERE clause for the index to be used
CREATE INDEX emp_dept_job_idx ON employees(department_id, job_id);

-- This query USES the index (filters on the leading column: department_id)
SELECT * FROM employees WHERE department_id = 10;

-- This query CANNOT efficiently use the index (missing leading column)
SELECT * FROM employees WHERE job_id = 'CLERK';  -- Full scan or skip scan
```

> **Why column order matters:** The B-Tree is sorted first by `department_id`, then by `job_id` within each department. If you search only on `job_id`, Oracle can't use the sorted tree structure because `job_id` values are scattered across all department groups.

```sql
-- Unique Index — enforces that no two rows have the same value
CREATE UNIQUE INDEX emp_email_uk ON employees(email);

-- Primary Key automatically creates a unique index behind the scenes
ALTER TABLE employees ADD CONSTRAINT emp_pk PRIMARY KEY (employee_id);
```

```sql
-- Function-Based Index — pre-computes the function result and indexes it
-- Problem: without this, UPPER(last_name) forces a full scan because
-- the index stores raw last_name values, not upper-cased ones
CREATE INDEX emp_upper_name_idx ON employees(UPPER(last_name));

-- Now this query can use the index
SELECT * FROM employees WHERE UPPER(last_name) = 'SMITH';

-- Date part index example
CREATE INDEX emp_year_idx ON employees(EXTRACT(YEAR FROM hire_date));
```

> **How it works:** Oracle physically stores the *result* of the function (e.g., 'SMITH' not 'Smith') in the index. When your query applies the same function in the WHERE clause, Oracle recognizes the match and uses the pre-computed index instead of applying the function to every row.

```sql
-- Bitmap Index — stores a bitmap (0/1 array) per distinct value
-- Best for: very low cardinality columns (gender, status, region — few distinct values)
-- ONLY use in Data Warehouse / read-mostly environments
CREATE BITMAP INDEX emp_gender_bidx ON employees(gender);
```

> **How it works:** For a column like `gender` with values M/F, Oracle creates two bitmaps. The M-bitmap is a binary string where position N is 1 if row N is Male, 0 otherwise. To filter `WHERE gender='M' AND status='ACTIVE'`, Oracle does a bitwise AND on two bitmaps — extremely fast. **BUT:** every DML operation (INSERT/UPDATE/DELETE) must update the bitmap, which causes row-level lock escalation to the entire bitmap segment. This makes Bitmap indexes catastrophically bad in OLTP — a single update can lock thousands of rows.

```sql
-- Reverse Key Index — reverses the bytes of each key before storing
-- Problem it solves: sequential values (1, 2, 3, 4...) all insert into the
-- rightmost leaf node, causing "hot block" contention in RAC environments
CREATE INDEX order_id_ridx ON orders(order_id) REVERSE;
```

> **How it works:** Key value `1000` is stored as `0001`, `1001` as `1001`, `1002` as `2001`, etc. This scatters sequential inserts across different leaf nodes, eliminating the hot-block problem. **Trade-off:** because bytes are reversed, range scans (`BETWEEN 1000 AND 2000`) cannot use this index — the reversal destroys the natural ordering.

---

### 1.4 Oracle Statistics

Statistics are metadata about your data — row counts, column value distributions, block counts — that the CBO uses to estimate which query plan is cheapest. **Without accurate stats, the optimizer guesses wrong.**

```sql
-- Gather statistics for one table (and its indexes if cascade=TRUE)
EXEC DBMS_STATS.GATHER_TABLE_STATS(
    ownname          => 'HR',              -- schema name
    tabname          => 'EMPLOYEES',       -- table name
    estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE,  -- Oracle decides sample %
    method_opt       => 'FOR ALL COLUMNS SIZE AUTO',  -- auto histograms where needed
    cascade          => TRUE               -- also gather index stats
);
```

> **What it does:** Oracle samples the table data (or scans it fully if small), counts rows, counts distinct values per column, builds value-frequency histograms for skewed columns, and stores all of this in the data dictionary. The CBO then reads these stats at parse time to estimate plan costs.

```sql
-- Gather for every table in a schema at once
EXEC DBMS_STATS.GATHER_SCHEMA_STATS('HR');
```

```sql
-- Histogram for a skewed column (e.g., 90% of employees are in department 10)
-- Without a histogram, Oracle assumes uniform distribution and estimates wrong
EXEC DBMS_STATS.GATHER_TABLE_STATS(
    ownname => 'HR', tabname => 'EMPLOYEES',
    method_opt => 'FOR COLUMNS SIZE 254 department_id'
    -- SIZE 254 means: build a histogram with up to 254 buckets
);
```

> **What a histogram does:** A normal statistic just records "there are 5000 distinct department_ids." A histogram goes further and records the *frequency distribution* — "department 10 has 4000 employees, all others have 5 each." The optimizer uses this to correctly estimate that `WHERE department_id = 10` will return 4000 rows (use full scan) vs `WHERE department_id = 99` will return 5 rows (use index).

```sql
-- Check how old your statistics are — stale stats = wrong plans
SELECT table_name, num_rows, last_analyzed,
       ROUND((SYSDATE - last_analyzed), 1) days_old
FROM user_tables ORDER BY last_analyzed NULLS FIRST;
```

---

### 1.5 Anti-Patterns and Why They Break Indexes

| Pattern | Bad | Good | Why it breaks the index |
|---------|-----|------|------------------------|
| Function on indexed column | `TRUNC(order_date) = '2024-01-15'` | `order_date >= '2024-01-15' AND order_date < '2024-01-16'` | The index stores raw `order_date` values. Applying `TRUNC()` means Oracle must compute TRUNC for every row — it can't use the sorted index. |
| Implicit type conversion | `WHERE phone_number = 5551234` | `WHERE phone_number = '5551234'` | Oracle silently wraps: `TO_NUMBER(phone_number) = 5551234`. Now there's a function on the indexed column — same problem as above. |
| Leading wildcard | `LIKE '%SMITH%'` | `LIKE 'SMITH%'` | The B-Tree is sorted by the beginning of the string. A leading wildcard means "match anything at the start" — Oracle must scan every single leaf node. A trailing wildcard lets Oracle seek to 'SMITH' and read forward. |
| SELECT \* (Snowflake/Databricks) | `SELECT * FROM wide_table` | `SELECT id, name FROM wide_table` | Both are columnar databases. They only read the columns you request from disk. SELECT \* forces reading every column's entire file. |
| ORDER BY without LIMIT | `ORDER BY date DESC` | `ORDER BY date DESC LIMIT 1000` | Sorting requires materializing the entire result set before returning any row. Without LIMIT, Snowflake must sort millions of rows you'll never see. |

---

### 1.6 Snowflake Architecture and Tuning

**How Snowflake stores data:** Every table is split into **micro-partitions** — immutable, compressed chunks of 50–500 MB each. Snowflake stores min/max statistics for every column in every micro-partition. When you filter a query, Snowflake's optimizer compares your filter value against these min/max stats and **prunes** (skips) any micro-partition that can't contain matching rows. This is called **partition pruning** and is the #1 way to make Snowflake queries fast.

```sql
-- Check current clustering depth of a table (lower = more organized = better pruning)
SELECT SYSTEM$CLUSTERING_DEPTH('orders');
-- Returns a number; 1.0 is perfect, anything > 4-5 needs attention

-- Detailed breakdown per column
SELECT SYSTEM$CLUSTERING_INFORMATION('orders', '(o_orderdate)');
-- Returns JSON with average_depth, average_overlaps, total_partition_count
```

> **What clustering depth means:** If depth = 1, every row with a given date is in the same micro-partition — perfect pruning. If depth = 10, a single date's rows are spread across 10 different micro-partitions — Oracle must read 10x more data.

```sql
-- Add a clustering key — tells Snowflake to physically reorganize data by this column
ALTER TABLE orders CLUSTER BY (o_orderdate);

-- Multi-column clustering for queries that filter on both
ALTER TABLE sales CLUSTER BY (sale_date, region);

-- Expression-based (extract just the date from a timestamp)
ALTER TABLE events CLUSTER BY (TO_DATE(event_timestamp));
```

> **What clustering does:** After you define a clustering key, Snowflake's background **Automatic Clustering** service gradually re-sorts the micro-partitions so rows with similar clustering key values end up in the same partitions. This is not instant — it runs asynchronously in the background. The goal is to minimize partition overlap so queries can skip more partitions.

```sql
-- Snowflake's 3-layer cache:
-- Layer 1: Query Result Cache (24h) — exact same SQL returns instantly, no compute used
-- Layer 2: Local Disk Cache (warehouse SSD) — recently scanned micro-partitions cached on warehouse nodes
-- Layer 3: Remote Storage (S3/Azure/GCS) — the actual data files

-- Force bypass of result cache (useful for benchmarking or when you need fresh data)
ALTER SESSION SET USE_CACHED_RESULT = FALSE;
SELECT COUNT(*) FROM orders WHERE year = 2024;   -- hits storage, measures real speed
ALTER SESSION SET USE_CACHED_RESULT = TRUE;

-- Check if a query came from cache (PERCENTAGE_SCANNED_FROM_CACHE = 100 means full cache hit)
SELECT QUERY_ID, QUERY_TEXT, BYTES_SCANNED, PERCENTAGE_SCANNED_FROM_CACHE
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY()) ORDER BY START_TIME DESC LIMIT 10;
```

```sql
-- Create a warehouse (virtual compute cluster)
CREATE WAREHOUSE analytics_wh
    WAREHOUSE_SIZE      = 'LARGE'       -- controls CPU and memory per node
    AUTO_SUSPEND        = 300           -- suspend after 5 min of no queries (saves credits)
    AUTO_RESUME         = TRUE          -- auto-wake when a query arrives
    MIN_CLUSTER_COUNT   = 1             -- minimum number of parallel clusters
    MAX_CLUSTER_COUNT   = 4             -- scale out to 4 clusters under high concurrency
    SCALING_POLICY      = 'STANDARD';   -- add clusters when queue builds up

ALTER WAREHOUSE analytics_wh SET WAREHOUSE_SIZE = 'XLARGE';
-- Scale UP (bigger node) when a single complex query needs more memory
-- Scale OUT (more clusters, multi-cluster) when many concurrent users queue up
```

```sql
-- Materialized View: pre-computes and stores the result of an expensive aggregation
-- Snowflake automatically refreshes it when base table data changes
-- Queries that match the MV pattern are transparently redirected to use it
CREATE MATERIALIZED VIEW mv_daily_sales AS
SELECT sale_date, region, COUNT(*) AS num_sales, SUM(amount) AS total
FROM sales GROUP BY sale_date, region;

SHOW MATERIALIZED VIEWS;              -- list all, check is_stale column
ALTER MATERIALIZED VIEW mv_daily_sales REFRESH;  -- manually force refresh
```

> **How Snowflake MVs work differently from Oracle:** In Oracle, MVs are optionally refreshed and the optimizer must be configured to rewrite queries. In Snowflake, the MV is always kept fresh automatically (incremental maintenance) and query rewriting is transparent — you can query either the MV directly or the base table and Snowflake will use whichever is faster.

```sql
-- Search Optimization Service (Enterprise Edition)
-- Builds a special index structure for point lookups on large tables
-- Normal pruning is based on min/max per partition — bad for equality on high-cardinality columns
-- Search Optimization uses a separate access path optimized for exact value lookups
ALTER TABLE customers ADD SEARCH OPTIMIZATION;
SELECT SYSTEM$ESTIMATE_SEARCH_OPTIMIZATION_COSTS('customers');  -- check cost before enabling
```

---

### 1.7 Databricks / Delta Lake Tuning

**How Delta Lake stores data:** Data lives in Parquet files on cloud storage (S3, ADLS, GCS). Delta adds a **transaction log** (the `_delta_log` folder) on top of Parquet — a JSON/Parquet log of every operation. Delta also maintains **file-level statistics** (min/max per column per file), which enables **data skipping** — the same concept as Snowflake partition pruning but at the individual file level.

```sql
-- Create a partitioned Delta table
-- Partitioning physically organizes files into folders by column value
-- Every query filtering on sale_date only reads files in that date's folder
CREATE TABLE sales (
    sale_id    BIGINT,
    sale_date  DATE,
    customer_id BIGINT,
    region     STRING,
    amount     DECIMAL(18,2)
)
USING DELTA
PARTITIONED BY (sale_date);
-- Physical storage: /delta/sales/sale_date=2024-01-15/part-00001.parquet
--                  /delta/sales/sale_date=2024-01-16/part-00001.parquet
```

> **Partitioning rule of thumb:** Only partition on columns with low-to-medium cardinality (dates work well — 365 partitions per year). Never partition on user_id or order_id — millions of tiny folders destroys metadata performance (this is called "the small files problem").

```sql
-- Z-Ordering: within each partition, physically colocate related rows in the same files
-- Uses a space-filling curve to sort data so rows with similar values on multiple columns
-- end up in the same files
OPTIMIZE sales ZORDER BY (customer_id, product_id);

-- Targeted Z-Order: only re-sort recent data (faster, less compute)
OPTIMIZE events WHERE event_date >= '2024-01-01' ZORDER BY (user_id);
```

> **Why Z-Order instead of more partitions:** If you also need to filter on `customer_id` but can't partition on it (too many distinct values), Z-Order is the answer. It co-locates rows for the same customer in the same small set of Parquet files. Delta's min/max statistics per file can then skip most files when you query a specific customer.

```sql
-- OPTIMIZE: compacts many small Parquet files into fewer large ones
-- Small files hurt performance because each file requires a separate I/O call
-- After streaming inserts (which write many tiny files), always run OPTIMIZE
OPTIMIZE orders;
OPTIMIZE orders WHERE order_date >= '2024-01-01';  -- targeted: only recent data

-- VACUUM: deletes Parquet files that are no longer referenced by the transaction log
-- Delta keeps old file versions for time travel — VACUUM prunes old ones
VACUUM orders RETAIN 168 HOURS;  -- keep 7 days of history (minimum for time travel)

-- Inspect current state of a table
DESCRIBE DETAIL orders;   -- shows numFiles, sizeInBytes, numPartitions
DESCRIBE HISTORY orders;  -- shows every operation: INSERT, OPTIMIZE, VACUUM, MERGE
```

```sql
-- Auto-optimize: Databricks automatically runs optimizeWrite and autoCompact
-- optimizeWrite: instead of writing many small files, Databricks bins data into
--   fewer, optimally-sized files at write time
-- autoCompact: after writes, Databricks compacts files in the background
ALTER TABLE orders SET TBLPROPERTIES (
    'delta.autoOptimize.optimizeWrite' = 'true',
    'delta.autoOptimize.autoCompact'   = 'true'
);
```

```sql
-- Broadcast Join: copies the smaller table to every worker node so no shuffle needed
-- Default threshold: tables < 10MB are auto-broadcast
-- Hint to force it for a slightly larger table:
SELECT /*+ BROADCAST(d) */ e.name, d.dept_name
FROM employees e JOIN departments d ON e.dept_id = d.dept_id;

-- Raise the auto-broadcast threshold to 100MB
SET spark.sql.autoBroadcastJoinThreshold = 100m;
```

> **Why broadcast joins matter:** In Spark, a regular join (SortMergeJoin) requires both tables to be **shuffled** — every row moved across the network to the correct worker based on its join key. Shuffle is the most expensive operation in distributed computing. A broadcast join eliminates shuffle for the small table entirely by sending a copy to every worker, so each worker can join locally.

```sql
-- AQE (Adaptive Query Execution): Spark re-optimizes the plan at runtime using
-- actual statistics collected during execution (not just estimates from ANALYZE)
SET spark.sql.adaptive.enabled                    = true;

-- coalescePartitions: after a shuffle, if many partitions are tiny, merge them
-- (avoids "many small tasks" overhead)
SET spark.sql.adaptive.coalescePartitions.enabled = true;

-- skewJoin: if one partition has 100x more data than others (data skew),
-- AQE splits that partition to avoid one slow worker holding everyone up
SET spark.sql.adaptive.skewJoin.enabled           = true;

-- localShuffleReader: if a stage produces sorted output, the next stage can
-- read from local storage instead of over the network
SET spark.sql.adaptive.localShuffleReader.enabled = true;
```

```sql
-- Bloom Filter Index: a probabilistic data structure that answers
-- "is this value definitely NOT in this file?" with 100% accuracy
-- Used for equality/IN lookups on high-cardinality columns where Z-Order helps less
CREATE BLOOMFILTER INDEX ON TABLE orders FOR COLUMNS (order_id);
SET spark.databricks.io.skipping.bloomFilter.enabled = true;
```

> **How a Bloom filter works:** It's a bit array. When a file is written, each value (e.g., order_id=1234) is hashed multiple times and the corresponding bits are set to 1. At query time, to check if `order_id=9999` might be in a file, Oracle hashes 9999 and checks if all those bit positions are 1. If any bit is 0, the value is **definitely not** in the file — skip it. If all bits are 1, the value **might** be there — read the file. False positives are possible but false negatives never are.

```sql
-- Caching: load a table into memory on the Spark workers for reuse across queries
CACHE TABLE customers;
CACHE TABLE orders OPTIONS ('storageLevel' 'MEMORY_AND_DISK');
-- MEMORY_AND_DISK: first tries RAM, spills to local disk if RAM is full

UNCACHE TABLE customers;   -- free the memory
CLEAR CACHE;               -- free all cached tables

-- Statistics for Spark's query planner (equivalent to Oracle DBMS_STATS)
ANALYZE TABLE employees COMPUTE STATISTICS;
ANALYZE TABLE employees COMPUTE STATISTICS FOR COLUMNS department_id, salary;
```

---

## 2. Security Management

---

### 2.1 How Oracle Security Works — The Big Picture

Oracle security is layered:

- **Authentication** — "Are you who you say you are?" → passwords, wallets, Kerberos
- **Authorization** — "What are you allowed to do?" → privileges and roles
- **Data Protection** — "What data can you see?" → views, VPD, redaction
- **Auditing** — "What did you actually do?" → audit trails

Every action in Oracle is checked against the **privilege model**: you can only do something if you have been explicitly granted the right to do it (or you own the object).

---

### 2.2 Users vs Schemas

**Key concept:** In Oracle, a **user** and a **schema** are two aspects of the same entity. When you create a user, Oracle automatically creates a schema (a namespace for objects) with the same name. The user is the login account; the schema is the container for that user's tables, views, procedures, etc.

```sql
-- Create a full production application user
CREATE USER ecom_app IDENTIFIED BY "Str0ng_P@ss!2024"
    DEFAULT TABLESPACE users        -- where this user's tables/indexes are physically stored
    TEMPORARY TABLESPACE temp       -- where sort operations spill to disk
    QUOTA 500M ON users             -- max disk space this user can use on that tablespace
    QUOTA UNLIMITED ON app_indexes  -- no limit on the index tablespace
    ACCOUNT UNLOCK;                 -- account is active immediately

-- Modify existing user
ALTER USER ecom_app IDENTIFIED BY "N3w$ecureP@ss2024";  -- change password
ALTER USER former_employee ACCOUNT LOCK;    -- disable login immediately (employee leaves)
ALTER USER ecom_user ACCOUNT UNLOCK;        -- re-enable
ALTER USER contractor_user PASSWORD EXPIRE; -- force password change on next login
ALTER USER ecom_user QUOTA 2G ON ecom_data; -- increase space quota
ALTER USER ecom_user PROFILE prod_security_profile; -- apply stricter password policy

-- Remove user and ALL their objects (tables, views, procedures, etc.)
DROP USER test_user CASCADE;  -- CASCADE is required if user owns any objects

-- See all users in the database
SELECT username, account_status, lock_date, expiry_date, default_tablespace, created
FROM dba_users ORDER BY created DESC;

-- See who is currently connected
SELECT username, sid, serial#, status, logon_time, program
FROM v$session WHERE type = 'USER';
```

---

### 2.3 Profiles — Password and Resource Policies

A **profile** is a named set of rules that controls password behaviour and resource limits. Every user is assigned exactly one profile (default is the `DEFAULT` profile if you don't specify one).

```sql
CREATE PROFILE prod_security_profile LIMIT
    -- Password lifetime and rotation rules
    PASSWORD_LIFE_TIME       90       -- password expires after 90 days
    PASSWORD_REUSE_TIME      365      -- can't reuse a password used in last year
    PASSWORD_REUSE_MAX       12       -- must change 12 times before reusing
    PASSWORD_VERIFY_FUNCTION ora12c_verify_function  -- enforce complexity (min length, special chars)
    PASSWORD_GRACE_TIME      7        -- after expiry, user gets 7-day grace period before lockout
    PASSWORD_LOCK_TIME       1        -- locked account auto-unlocks after 1 day
    FAILED_LOGIN_ATTEMPTS    5        -- lock account after 5 wrong passwords in a row

    -- Resource limits per session
    SESSIONS_PER_USER        3        -- max 3 simultaneous connections for this user
    CPU_PER_CALL             60000    -- max CPU time per single SQL statement (hundredths of seconds)
    CONNECT_TIME             480      -- session auto-disconnects after 8 hours
    IDLE_TIME                30;      -- auto-disconnect after 30 min of no activity

-- Relaxed profile for service accounts (apps that connect from code, never expire)
CREATE PROFILE service_account_profile LIMIT
    PASSWORD_LIFE_TIME    UNLIMITED   -- never expires — managed by vault/secret manager
    FAILED_LOGIN_ATTEMPTS 3           -- still strict on failed attempts
    SESSIONS_PER_USER     50;         -- allow many parallel connections (connection pool)
```

> **Why profiles matter:** Without a profile, a developer account might use the same password for 5 years, never lock after brute-force attempts, and keep a connection open forever consuming memory. Profiles enforce corporate security standards automatically for every user assigned to them.

---

### 2.4 System Privileges vs Object Privileges

**System Privileges** grant the right to perform a *type* of operation anywhere in the database:
- `CREATE TABLE` — create tables in your own schema
- `CREATE ANY TABLE` — create tables in ANY schema (dangerous)
- `SELECT ANY TABLE` — read any table in any schema (very dangerous)

**Object Privileges** grant the right to do something on one *specific* object:
- `SELECT ON hr.employees` — read only this specific table
- `UPDATE (salary) ON hr.employees` — update only the salary column of this table

```sql
-- Grant system privileges (minimum needed for a developer)
GRANT CREATE SESSION   TO dev_user;   -- allow login (without this, can't connect at all)
GRANT CREATE TABLE     TO dev_user;   -- create tables in own schema
GRANT CREATE VIEW      TO dev_user;
GRANT CREATE PROCEDURE TO dev_user;
GRANT CREATE SEQUENCE  TO dev_user;

-- WITH ADMIN OPTION: this user can now grant/revoke this privilege to/from others
-- IMPORTANT: if you later REVOKE from team_lead, their grantees KEEP the privilege (no cascade)
GRANT CREATE TABLE TO team_lead WITH ADMIN OPTION;

-- Grant object privileges on a specific table
GRANT SELECT ON hr.employees TO reporting_user;
GRANT SELECT, INSERT, UPDATE ON sales.orders TO sales_app;

-- WITH GRANT OPTION: this user can pass this object privilege to others
-- IMPORTANT: if you later REVOKE from manager, cascade DOES happen — grantees lose access too
GRANT SELECT ON hr.employees TO manager WITH GRANT OPTION;

-- Column-level privilege — expose only specific columns (great for hiding salary, SSN)
GRANT SELECT (employee_id, first_name, last_name, email) ON hr.employees TO public_portal;
GRANT UPDATE (phone_number, email) ON hr.employees TO self_service_app;

-- Grant execute on a stored procedure (user can run the proc without direct table access)
GRANT EXECUTE ON hr.calculate_bonus TO payroll_app;

-- Revoke privileges
REVOKE CREATE TABLE FROM former_developer;
REVOKE SELECT ON hr.employees FROM manager;  -- cascades to manager's grantees (WITH GRANT OPTION case)
```

---

### 2.5 Roles

A **role** is a named container for a collection of privileges. Instead of granting 20 individual privileges to each of 50 users, you grant those 20 privileges to one role, then grant that one role to 50 users. When requirements change, you update the role once.

```sql
-- Create a role
CREATE ROLE sales_role;
CREATE ROLE admin_role IDENTIFIED BY "Adm1n#S3cur3";  -- password-protected role
-- A password-protected role must be explicitly enabled with SET ROLE — it's not on by default

-- Build up the role with privileges
GRANT CREATE SESSION              TO sales_role;
GRANT SELECT ON sales.customers   TO sales_role;
GRANT SELECT, INSERT, UPDATE ON sales.orders TO sales_role;
GRANT EXECUTE ON sales.calc_discount TO sales_role;

-- Role hierarchy: inherit one role inside another
-- sales_role includes everything in employee_base_role
GRANT employee_base_role TO sales_role;

-- Assign roles to users
GRANT sales_role TO john_smith;
GRANT sales_role, reporting_role TO jane_doe;

-- WITH ADMIN OPTION: this user can grant/revoke this role to/from others
GRANT manager_role TO dept_head WITH ADMIN OPTION;

-- Control which roles are active by default at login
ALTER USER john_smith DEFAULT ROLE sales_role;     -- only sales_role active at login
ALTER USER secure_user DEFAULT ROLE NONE;          -- no roles active at login (must SET ROLE manually)

-- Enable/disable roles within a session
SET ROLE admin_role IDENTIFIED BY "Adm1n#S3cur3"; -- activate password-protected role
SET ROLE ALL;                   -- activate all granted roles
SET ROLE NONE;                  -- deactivate all roles
SET ROLE sales_role, reporting_role;  -- activate only these two
```

---

### 2.6 Security Dictionary Views

```sql
-- All users in the database
SELECT username, account_status, lock_date, expiry_date, default_tablespace, created
FROM dba_users;

-- Currently active sessions
SELECT username, sid, serial#, status, logon_time FROM v$session WHERE type = 'USER';

-- What system privileges does a user have?
SELECT grantee, privilege, admin_option FROM dba_sys_privs WHERE grantee = 'SALES_USER';

-- What object privileges does a user have?
SELECT grantee, owner, table_name, privilege, grantable FROM dba_tab_privs WHERE grantee = 'SALES_USER';

-- Column-level privileges
SELECT grantee, owner, table_name, column_name, privilege FROM dba_col_privs WHERE grantee = 'SALES_USER';

-- All roles in the database
SELECT role, password_required FROM dba_roles;

-- Which roles are granted to a user?
SELECT grantee, granted_role, admin_option, default_role FROM dba_role_privs WHERE grantee = 'SALES_USER';

-- What privileges does a specific role contain?
SELECT role, privilege FROM role_sys_privs WHERE role = 'SALES_ROLE';
```

---

### 2.7 Views as a Security Layer

Views filter what a user can see — either hiding sensitive **columns** or restricting **rows** — without touching the underlying table at all.

```sql
-- Column-level security: create a view that omits sensitive columns (salary, SSN, bank account)
CREATE OR REPLACE VIEW hr.employee_directory AS
SELECT employee_id, first_name, last_name, email, phone, job_id, department_id
FROM hr.employees;
-- salary, ssn, bank_account are NOT in this view

-- Grant access to the view, not the table
GRANT SELECT ON hr.employee_directory TO public_portal_role;
-- public_portal_role users can never see salary even if they try to query hr.employees directly
-- (they don't have SELECT on hr.employees — only on hr.employee_directory)

-- Row-level security: filter rows dynamically using a session context value
CREATE OR REPLACE VIEW saas.my_customers AS
SELECT customer_id, customer_name, email, created_date
FROM saas.customers
WHERE tenant_id = SYS_CONTEXT('tenant_ctx', 'tenant_id');
-- Each connected user's tenant_id is stored in the application context at login
-- User from Tenant A literally cannot see Tenant B's rows — the WHERE clause filters them out
```

---

### 2.8 VPD — Virtual Private Database

VPD is Oracle's **row-level security engine**. You write a function that returns a WHERE clause string, and Oracle **automatically appends** that WHERE clause to every query against the protected table. The user never sees it happening.

```sql
-- Step 1: Write the policy function (returns a WHERE clause as a string)
CREATE OR REPLACE FUNCTION sales.customer_policy(p_schema VARCHAR2, p_object VARCHAR2)
RETURN VARCHAR2 AS
    v_role VARCHAR2(50) := SYS_CONTEXT('sales_ctx', 'role');
BEGIN
    IF v_role = 'SALES_REP' THEN
        -- Sales reps only see their own assigned customers
        RETURN 'assigned_rep_id = SYS_CONTEXT(''sales_ctx'', ''employee_id'')';
    ELSIF v_role = 'VP_SALES' THEN
        RETURN '1=1';   -- VP sees everything (always true)
    ELSE
        RETURN '1=0';   -- Unknown role sees nothing (always false)
    END IF;
END;
/

-- Step 2: Register the policy — attach the function to the table
BEGIN
    DBMS_RLS.ADD_POLICY(
        object_schema   => 'SALES',
        object_name     => 'CUSTOMERS',
        policy_name     => 'CUSTOMER_ACCESS_POLICY',
        function_schema => 'SALES',
        policy_function => 'CUSTOMER_POLICY',
        statement_types => 'SELECT, INSERT, UPDATE, DELETE',
        update_check    => TRUE,   -- also check on UPDATE (prevent moving rows out of visible set)
        enable          => TRUE
    );
END;
/
```

> **What happens at query time:** When a sales rep runs `SELECT * FROM sales.customers`, Oracle intercepts the query and rewrites it internally to `SELECT * FROM sales.customers WHERE assigned_rep_id = 42` (where 42 is the current user's employee_id from the context). The user never sees the added predicate and cannot bypass it.

---

### 2.9 Data Redaction

Data Redaction transforms sensitive data **at query time** — the underlying data in the table is unchanged, but what the query returns is masked. No application code changes needed.

```sql
-- PARTIAL mask: show last 4 digits of SSN, replace first 5 with X
BEGIN
    DBMS_REDACT.ADD_POLICY(
        object_schema       => 'HR',
        object_name         => 'EMPLOYEES',
        column_name         => 'SSN',
        policy_name         => 'SSN_REDACT_POLICY',
        function_type       => DBMS_REDACT.PARTIAL,
        -- format: 'input_format, output_format, replacement_char, start_pos, end_pos'
        function_parameters => 'VVVFVVFVVVV,VVV-VV-VVVV,X,1,5',
        -- expression: when is redaction applied? (here: for everyone except HR_ADMIN)
        expression          => 'SYS_CONTEXT(''USERENV'', ''SESSION_USER'') != ''HR_ADMIN'''
    );
END;
/
-- Non-admin user queries: SELECT ssn FROM employees → gets XXX-XX-6789
-- HR_ADMIN queries: SELECT ssn FROM employees → gets 123-45-6789 (real value)
-- The table data is NEVER changed — redaction happens only in the return buffer
```

---

### 2.10 Auditing

Auditing answers "who did what, when, to which data?" It is required for SOX, HIPAA, PCI-DSS compliance.

```sql
-- CREATE an audit policy (defines what to track — doesn't start auditing yet)
CREATE AUDIT POLICY sensitive_data_access
    ACTIONS SELECT ON finance.transactions,
            SELECT ON hr.employee_salaries;

-- ENABLE the policy (starts writing audit records)
AUDIT POLICY sensitive_data_access;

-- Audit all DDL in the schema
CREATE AUDIT POLICY ddl_changes
    ACTIONS CREATE TABLE, ALTER TABLE, DROP TABLE, CREATE INDEX, DROP INDEX;
AUDIT POLICY ddl_changes;

-- Audit all login/logout events
CREATE AUDIT POLICY login_activity ACTIONS LOGON, LOGOFF;
AUDIT POLICY login_activity;

-- Query the unified audit trail (where all audit records go in 12c+)
SELECT event_timestamp, dbusername, action_name, object_schema, object_name, sql_text
FROM unified_audit_trail
WHERE event_timestamp > SYSDATE - 7
ORDER BY event_timestamp DESC;

-- Fine-Grained Auditing (FGA): audit only specific rows/values, not every access
-- Here we only audit when someone looks at transactions > $100,000
BEGIN
    DBMS_FGA.ADD_POLICY(
        object_schema   => 'FINANCE',
        object_name     => 'TRANSACTIONS',
        policy_name     => 'HIGH_VALUE_TRANSACTION_AUDIT',
        audit_condition => 'amount > 100000',   -- only fires when this is true for accessed rows
        audit_column    => 'AMOUNT',
        statement_types => 'SELECT, INSERT, UPDATE'
    );
END;
/
```

---

## 3. Advanced SQL Features

---

### 3.1 How Window Functions Work

**The fundamental difference from GROUP BY:**

`GROUP BY` collapses rows — you get one row per group and lose access to individual row values. Window functions perform the same kind of calculation but **keep every row intact** — they add a new computed column alongside each row. Think of it as "GROUP BY without the collapse."

**Anatomy of a window function:**
```
function_name(arguments)  OVER  (  PARTITION BY col   ORDER BY col   ROWS BETWEEN ... AND ...  )
       |                              |                    |                    |
   What to             Define         Sort rows       Define exactly which
   compute            groups/windows  within window    rows are in the "frame"
                      (like GROUP BY)                  for this calculation
```

The `OVER()` clause is what makes it a window function. An empty `OVER()` means "consider all rows as one window."

---

### 3.2 Ranking Functions

```sql
-- ROW_NUMBER(): assigns a unique sequential number 1,2,3,4... within the partition
-- Even ties get different numbers (arbitrary tiebreak based on ORDER BY stability)
ROW_NUMBER() OVER (PARTITION BY region ORDER BY amount DESC)
```

> **Use case:** Top-N per group. After assigning row numbers, filter `WHERE rn <= 3` to get the top 3 per region. Since numbers are unique, you always get exactly 3 rows per region.

```sql
-- RANK(): same rank for ties, then SKIPS numbers
-- If rows 2 and 3 both tie for 2nd, both get rank 2, next gets rank 4 (gap)
RANK() OVER (ORDER BY amount DESC)
-- Result for values [3600, 2400, 2400, 2400, 1200]: ranks = [1, 2, 2, 2, 5]
```

> **Use case:** Actual competition ranking ("2nd place tied with someone else").

```sql
-- DENSE_RANK(): same rank for ties, NO gaps
DENSE_RANK() OVER (ORDER BY amount DESC)
-- Result for values [3600, 2400, 2400, 2400, 1200]: ranks = [1, 2, 2, 2, 3]
```

> **Use case:** Department salary grades where you want "there are 3 salary bands" not "there are 5 with gaps."

```sql
-- PERCENT_RANK(): where does this row sit in the distribution? Returns 0.0 to 1.0
-- Formula: (RANK - 1) / (total rows - 1)
ROUND(PERCENT_RANK() OVER (PARTITION BY department_id ORDER BY salary) * 100, 1) AS percentile

-- NTILE(n): divides rows into n equal buckets, assigns 1 through n
-- Used to create quartiles, deciles, or any equal-size segments
NTILE(4) OVER (ORDER BY salary)   -- 1=bottom 25%, 2=25-50%, 3=50-75%, 4=top 25%
NTILE(5) OVER (ORDER BY recency_days DESC)  -- for RFM scoring: 5=most recent, 1=oldest
```

**Practical — top 3 per group:**
```sql
SELECT region, salesperson, amount
FROM (
    SELECT region, salesperson, amount,
           ROW_NUMBER() OVER (PARTITION BY region ORDER BY amount DESC) AS rn
    FROM sales_data
)
WHERE rn <= 3;
-- ROW_NUMBER is used (not RANK) so you always get exactly 3 rows per region
-- If you used RANK, ties at position 3 would give you more than 3 rows
```

---

### 3.3 Aggregate Window Functions

```sql
-- These are standard aggregate functions (SUM, AVG, COUNT, MIN, MAX) used as window functions
-- The key: they calculate across a set of rows but don't collapse — each row keeps its own value

SELECT
    sale_date, region, salesperson, amount,
    SUM(amount) OVER ()                           AS grand_total,          -- all rows
    SUM(amount) OVER (PARTITION BY region)        AS region_total,         -- rows in same region
    AVG(amount) OVER (PARTITION BY salesperson)   AS person_avg,           -- rows by same person
    COUNT(*) OVER (PARTITION BY region)           AS region_count,
    ROUND(amount / SUM(amount) OVER (PARTITION BY region) * 100, 1) AS pct_of_region
FROM sales_data;
-- Each row shows its own amount PLUS these window calculations — no GROUP BY collapse
```

---

### 3.4 Running Totals and Moving Averages

The **frame clause** (`ROWS BETWEEN ... AND ...`) is what makes running totals and moving averages possible. It defines which rows are included in the calculation *for each row*.

```sql
-- Running total: from the very first row up to and including the current row
SUM(amount) OVER (
    ORDER BY sale_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS running_total
-- Row 1: sum of row 1 only
-- Row 2: sum of rows 1-2
-- Row 3: sum of rows 1-3  ... and so on

-- 3-day moving average: current row plus 2 rows before it
AVG(daily_total) OVER (
    ORDER BY sale_date
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
) AS moving_avg_3day
-- For each day, averages that day + the 2 days before

-- 7-day moving average
AVG(daily_total) OVER (
    ORDER BY sale_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
)

-- Centered 5-day average: 2 before + current + 2 after
AVG(daily_total) OVER (
    ORDER BY sale_date
    ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING
)
```

> **ROWS vs RANGE:** `ROWS` counts physical rows (always deterministic). `RANGE` counts rows whose ORDER BY value falls within a range (non-deterministic if ties exist). For most purposes, use `ROWS` — it's more predictable.

---

### 3.5 LAG and LEAD

LAG and LEAD let you access a different row's value from within the current row's context — without a self-join.

```sql
-- LAG(column, offset, default_if_no_row)
-- Look BACK by 'offset' rows in the ORDER BY sequence
LAG(amount, 1, 0) OVER (ORDER BY sale_date) AS prev_sale
-- For each row, this shows the amount of the row that comes 1 position before it

-- LEAD: look FORWARD
LEAD(amount, 1, 0) OVER (ORDER BY sale_date) AS next_sale

-- Month-over-month growth calculation
ROUND(
    (revenue - LAG(revenue) OVER (ORDER BY month))
    / NULLIF(LAG(revenue) OVER (ORDER BY month), 0) * 100, 1
) AS mom_growth_pct
-- NULLIF prevents divide-by-zero if previous month revenue was 0

-- Per-person trend: partition means we reset for each salesperson
LAG(amount) OVER (PARTITION BY salesperson ORDER BY sale_date) AS prev_sale_by_person

-- Year-over-year: lag by 12 positions to compare same month last year
LAG(revenue, 12) OVER (ORDER BY month) AS same_month_last_year
```

---

### 3.6 FIRST_VALUE, LAST_VALUE, NTH_VALUE

These access specific positional values within the window — the first, last, or Nth value.

```sql
-- CRITICAL: LAST_VALUE and NTH_VALUE have a default frame of
-- ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
-- This means LAST_VALUE gives the *current row's value*, not the partition's last value!
-- You MUST specify the full frame explicitly:

FIRST_VALUE(amount) OVER (
    PARTITION BY region ORDER BY amount DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING  -- entire partition
) AS highest_in_region

LAST_VALUE(amount) OVER (
    PARTITION BY region ORDER BY amount DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING  -- required!
) AS lowest_in_region

NTH_VALUE(amount, 2) OVER (
    PARTITION BY region ORDER BY amount DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
) AS second_highest
```

---

### 3.7 Hierarchical Queries — CONNECT BY

Oracle's proprietary syntax for traversing tree-structured (parent-child) data. Perfect for org charts, product categories, folder structures, BOMs.

```sql
SELECT
    LEVEL,                                            -- depth (root = 1)
    LPAD(' ', (LEVEL - 1) * 4) || employee_name AS org_chart,
    job_title, salary
FROM employees_hier
START WITH  manager_id IS NULL                        -- where to begin (root nodes)
CONNECT BY PRIOR employee_id = manager_id             -- how to walk from parent to child
ORDER SIBLINGS BY employee_name;                      -- sort within the same level
```

> **How CONNECT BY works:** Oracle starts with rows matching `START WITH`. Then it repeatedly applies the `CONNECT BY` rule — for each current row, find all rows where `manager_id` equals the current row's `employee_id` (because of `PRIOR`). `PRIOR` means "the value from the parent row." Oracle continues this recursively until no more children are found.

**Key elements:**

| Syntax | What it does |
|--------|--------------|
| `START WITH manager_id IS NULL` | Begin from root (the CEO has no manager) |
| `CONNECT BY PRIOR employee_id = manager_id` | For each row, find children where their `manager_id` = current row's `employee_id` |
| `LEVEL` | Pseudo-column: 1 for root, 2 for direct reports, 3 for grandchildren, etc. |
| `ORDER SIBLINGS BY col` | Sort nodes at the same depth — regular ORDER BY would destroy the tree structure |
| `CONNECT_BY_ISLEAF` | 1 if this row has no children (is a leaf node) |
| `SYS_CONNECT_BY_PATH(col, '/')` | Returns the path from root to current row, e.g. `/CEO/VP/Manager/Employee` |
| `CONNECT_BY_ROOT col` | Returns the root row's value of `col` for any node in the tree |

```sql
-- Full path from root to each node (breadcrumbs)
SYS_CONNECT_BY_PATH(employee_name, ' → ') AS reporting_path
-- Returns: ' → Steven King → Lex De Haan → Alexander Hunold'

-- Who is the ultimate boss for any node?
CONNECT_BY_ROOT employee_name AS ultimate_boss

-- Only leaf nodes (individual contributors with no direct reports)
SELECT employee_name FROM employees_hier
WHERE CONNECT_BY_ISLEAF = 1
START WITH manager_id IS NULL
CONNECT BY PRIOR employee_id = manager_id;

-- Limit depth (only top 3 levels of hierarchy)
WHERE LEVEL <= 3

-- BOTTOM-UP: start from a leaf and walk up to the root
-- Note: PRIOR is reversed — employee_id = PRIOR manager_id instead of PRIOR employee_id = manager_id
START WITH employee_name = 'Diana Lorentz'
CONNECT BY employee_id = PRIOR manager_id;
```

---

### 3.8 Recursive CTEs

The ANSI SQL standard alternative to CONNECT BY. Works in Oracle 11gR2+ and is portable to other databases.

```sql
WITH org_hierarchy (employee_id, employee_name, manager_id, lvl, path) AS (

    -- ANCHOR MEMBER: the starting point (runs once, no recursion)
    SELECT employee_id, employee_name, manager_id, 1, employee_name
    FROM employees_hier
    WHERE manager_id IS NULL  -- start from roots

    UNION ALL

    -- RECURSIVE MEMBER: joins back to the CTE itself
    -- Runs repeatedly: each iteration finds children of the previous iteration's rows
    SELECT e.employee_id, e.employee_name, e.manager_id,
           h.lvl + 1,                              -- depth increases by 1
           h.path || ' → ' || e.employee_name      -- build path string
    FROM employees_hier e
    INNER JOIN org_hierarchy h ON e.manager_id = h.employee_id  -- join CTE to itself
)
SELECT lvl, LPAD(' ', (lvl-1)*4) || employee_name AS chart, path
FROM org_hierarchy
ORDER BY path;
```

> **How recursion terminates:** Oracle keeps running the recursive member until it produces zero new rows. Since every iteration finds children of the previous iteration's rows, the process ends naturally when a set of rows has no children (leaf nodes). Oracle sets a default recursion depth limit to prevent infinite loops.

---

### 3.9 Subqueries

#### Correlated Subquery

A correlated subquery references a column from the outer query. It **re-executes for every row** of the outer query — making it potentially slow for large tables (but sometimes the only clean way to express the logic).

```sql
-- Find employees earning more than their own department's average
SELECT e.employee_name, e.department, e.salary,
       (SELECT AVG(salary) FROM employees_hier WHERE department = e.department) AS dept_avg
FROM employees_hier e
WHERE e.salary > (
    SELECT AVG(salary)
    FROM employees_hier
    WHERE department = e.department  -- ← correlated: references outer query's e.department
);
-- The subquery runs once for each row in the outer query, using that row's department value
```

#### EXISTS vs IN

```sql
-- EXISTS: checks for the presence of at least one row — stops at first match (short-circuit)
-- Best when the subquery might return many rows
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
    -- SELECT 1: we don't care about the value, just whether any row exists
);

-- IN: retrieves all values from the subquery, then checks membership
-- Best when the subquery returns a small, fixed set
SELECT customer_id FROM customers
WHERE region_id IN (SELECT region_id FROM regions WHERE country = 'USA');

-- NOT IN: DANGEROUS if the subquery can return NULLs
-- NULL in a NOT IN list makes the whole condition UNKNOWN → returns zero rows!
-- WRONG (if any customer_id in orders is NULL, this returns nothing):
WHERE customer_id NOT IN (SELECT customer_id FROM orders);

-- SAFE: use NOT EXISTS instead
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);

-- Or explicitly exclude NULLs from the IN list:
WHERE customer_id NOT IN (SELECT customer_id FROM orders WHERE customer_id IS NOT NULL);
```

#### Multi-Column Subquery

```sql
-- Match on a combination of columns at once
SELECT employee_name, department, salary
FROM employees_hier
WHERE (department, salary) IN (
    SELECT department, MAX(salary)
    FROM employees_hier
    GROUP BY department
);
-- Returns the highest-paid employee in each department
-- The tuple (department, salary) must match a (department, max_salary) pair from the subquery
```

---

### 3.10 CTEs (WITH Clause)

A CTE is a named temporary result set that exists only for the duration of the query. It makes complex queries readable by naming intermediate steps, and it can reference earlier CTEs.

```sql
WITH
-- CTE 1: basic aggregation
dept_stats AS (
    SELECT department, AVG(salary) AS avg_salary, COUNT(*) AS emp_count
    FROM employees_hier GROUP BY department
),
-- CTE 2: uses no other CTEs
company_stats AS (
    SELECT AVG(salary) AS company_avg FROM employees_hier
)
-- Main query references both CTEs
SELECT d.department, d.emp_count, ROUND(d.avg_salary, 2),
       CASE WHEN d.avg_salary > c.company_avg THEN 'Above Average' ELSE 'Below Average' END
FROM dept_stats d
CROSS JOIN company_stats c    -- CROSS JOIN because company_stats is a single row
ORDER BY d.avg_salary DESC;
```

> **CTE vs Inline View:** Functionally equivalent in most cases. CTEs are better for readability when the same subquery is needed multiple times, or when you need to build logic in named stages. In Oracle, a CTE can be a good place to define BULK operations or recursive queries.

---

### 3.11 MERGE (UPSERT)

MERGE combines INSERT and UPDATE (and optionally DELETE) into a single atomic operation. It's the standard way to synchronize a target table with a source.

```sql
MERGE INTO warehouse_inventory tgt         -- the table being modified
USING supplier_inventory src               -- the source of new data
ON (tgt.product_code = src.product_code)   -- the join condition to match rows

-- Branch 1: matching row found in target → update it
WHEN MATCHED THEN
    UPDATE SET
        tgt.product_name  = src.product_name,
        tgt.quantity      = src.quantity,
        tgt.last_updated  = SYSDATE
    -- Optional DELETE within WHEN MATCHED (deletes the target row if condition is true)
    DELETE WHERE src.quantity = 0

-- Branch 2: no matching row in target → insert new row
WHEN NOT MATCHED THEN
    INSERT (product_code, product_name, quantity, created_date)
    VALUES (src.product_code, src.product_name, src.quantity, SYSDATE)
    WHERE src.quantity > 0;   -- optional filter: don't insert zero-quantity items
```

> **What happens internally:** Oracle evaluates the ON condition for every row in the source. Rows where a match is found take the WHEN MATCHED path; rows with no match take the WHEN NOT MATCHED path. All of this happens in a single pass — much more efficient than a separate DELETE + INSERT or checking existence before inserting. MERGE is fully atomic (all changes or none).

---

### 3.12 Multi-Table INSERT

Distributes one source query's rows into multiple target tables in a single pass.

```sql
-- INSERT ALL: every row can go to MULTIPLE tables (conditions are independent)
INSERT ALL
    WHEN type = 'SALE' THEN
        INTO sales_log (id, txn_date, amount) VALUES (id, txn_date, amount)
    WHEN amount > 10000 THEN
        INTO high_value_txns VALUES (id, txn_date, amount, 'REVIEW')
    WHEN 1=1 THEN                  -- always true → every row goes to audit regardless of other conditions
        INTO audit_log VALUES (id, txn_date, amount, SYSDATE)
SELECT id, txn_date, amount, type FROM transactions;
-- A $15,000 SALE row goes to: sales_log, high_value_txns, AND audit_log (all conditions checked)

-- INSERT FIRST: each row goes to the FIRST matching target only (like an IF/ELSIF chain)
INSERT FIRST
    WHEN amount > 10000 THEN INTO high_value_txns VALUES (...)
    WHEN type = 'REFUND' THEN INTO refund_log VALUES (...)
    WHEN type = 'SALE'   THEN INTO sales_log VALUES (...)
    ELSE                      INTO other_txns VALUES (...)
SELECT ... FROM transactions;
-- A $15,000 SALE row ONLY goes to high_value_txns — first match wins, rest are skipped
```

---

### 3.13 RETURNING Clause

Captures the values that were inserted, updated, or deleted — useful for getting auto-generated IDs or auditing old/new values without an extra SELECT.

```sql
-- Capture the auto-generated ID after INSERT
DECLARE v_new_id NUMBER; v_created DATE;
BEGIN
    INSERT INTO customers (customer_id, customer_name, email, created_date)
    VALUES (customer_seq.NEXTVAL, 'New Customer', 'new@email.com', SYSDATE)
    RETURNING customer_id, created_date INTO v_new_id, v_created;
    -- v_new_id now holds the sequence value that was used — no need for CURRVAL query
END;
/

-- Capture new salary after UPDATE
UPDATE employees SET salary = salary * 1.1 WHERE employee_id = 104
RETURNING salary INTO v_new_salary;

-- Bulk capture with DELETE (who was deleted?)
DELETE FROM temp_customers WHERE status = 'INACTIVE'
RETURNING customer_id, customer_name BULK COLLECT INTO v_ids, v_names;
```

---

### 3.14 PIVOT

PIVOT rotates rows into columns — turns unique values in a single column into separate column headers.

```sql
SELECT *
FROM (
    -- Inner query: prepare the data (3 columns: the row identifier, the category, the value)
    SELECT region, TO_CHAR(sale_date, 'Mon') AS month, amount
    FROM sales_data
)
PIVOT (
    SUM(amount)              -- what to aggregate for each cell
    FOR month                -- which column contains the future column headers
    IN ('Jan' AS Jan, 'Feb' AS Feb, 'Mar' AS Mar,
        'Apr' AS Apr, 'May' AS May, 'Jun' AS Jun,
        'Jul' AS Jul, 'Aug' AS Aug, 'Sep' AS Sep,
        'Oct' AS Oct, 'Nov' AS Nov, 'Dec' AS Dec)
    -- IN: explicitly list all values to pivot — Oracle doesn't do dynamic PIVOT natively
)
ORDER BY region;
-- Result: one row per region, one column per month (Jan, Feb, ... Dec), cell = SUM(amount)

-- Multiple aggregates in one PIVOT: creates pairs of columns (Q1_CNT, Q1_TOTAL, Q2_CNT, Q2_TOTAL...)
PIVOT (
    COUNT(*) AS cnt,
    SUM(amount) AS total
    FOR quarter IN ('1' AS Q1, '2' AS Q2, '3' AS Q3, '4' AS Q4)
)
```

---

### 3.15 UNPIVOT

UNPIVOT is the reverse — turns columns into rows. Used to normalize denormalized/wide tables.

```sql
-- quarterly_results has columns: region, q1_sales, q2_sales, q3_sales, q4_sales
-- We want: region, quarter, sales_amount (normalized, tall format)
SELECT *
FROM quarterly_results
UNPIVOT (
    sales_amount               -- name for the new VALUE column
    FOR quarter                -- name for the new CATEGORY column
    IN (q1_sales AS 'Q1',      -- maps column q1_sales → category value 'Q1'
        q2_sales AS 'Q2',
        q3_sales AS 'Q3',
        q4_sales AS 'Q4')
)
ORDER BY region, quarter;
-- Each original row expands into 4 rows (one per quarter)

-- By default, UNPIVOT excludes NULLs (rows where the column is NULL are dropped)
-- To keep them:
UNPIVOT INCLUDE NULLS (sales_amount FOR quarter IN (...))
```

---

### 3.16 ROLLUP, CUBE, and GROUPING SETS

These are extensions of `GROUP BY` that produce multiple levels of aggregation in a single query.

```sql
-- ROLLUP: generates hierarchical subtotals
-- GROUP BY ROLLUP(region, salesperson) produces:
--   1. region + salesperson detail rows
--   2. region subtotals (salesperson is NULL)
--   3. grand total (both are NULL)
SELECT region, salesperson, SUM(amount) AS total
FROM sales_data
GROUP BY ROLLUP(region, salesperson)
ORDER BY region NULLS LAST, salesperson NULLS LAST;

-- CUBE: generates ALL possible combinations of subtotals
-- GROUP BY CUBE(region, product) produces:
--   region + product detail, region subtotals, product subtotals, grand total
GROUP BY CUBE(region, product)

-- GROUPING SETS: precisely control which combinations to compute
-- More efficient than CUBE when you only want specific breakdowns
GROUP BY GROUPING SETS (
    (region, product),   -- sales by region AND product
    (region),            -- region subtotals only
    ()                   -- grand total only
)

-- GROUPING(col): returns 1 when that column is part of the aggregation (i.e., it's NULL because of ROLLUP/CUBE)
-- Use this to distinguish "NULL because aggregated" from "NULL because data was NULL"
SELECT region, salesperson, SUM(amount),
       GROUPING(region)     AS is_region_subtotal,    -- 1 = this is a region subtotal row
       GROUPING(salesperson) AS is_person_subtotal     -- 1 = this is a person subtotal row
FROM sales_data GROUP BY ROLLUP(region, salesperson);

-- GROUPING_ID(): single integer that encodes which columns are aggregated (bitmap)
-- GROUPING_ID(region, salesperson) = 0 → both are detail values
-- GROUPING_ID(region, salesperson) = 1 → salesperson is aggregated (region subtotal)
-- GROUPING_ID(region, salesperson) = 3 → both are aggregated (grand total)
CASE GROUPING_ID(region, salesperson)
    WHEN 0 THEN 'Detail'
    WHEN 1 THEN 'Region Subtotal'
    WHEN 3 THEN 'Grand Total'
END AS row_type
```

---

## 4. PL/SQL Fundamentals

---

### 4.1 What PL/SQL Is and Why It Exists

Plain SQL is **set-based** — you describe what data you want, and the engine decides how to get it. But many real-world tasks require **procedural logic**: "if salary > X, then do Y; otherwise loop through all employees and do Z." That's what PL/SQL adds.

PL/SQL runs **inside the Oracle database engine**, not in your application server. This means:
- No network round-trips for each statement (a loop of 10,000 UPDATEs is 1 network call, not 10,000)
- Direct access to Oracle's internal machinery (cursors, bulk operations, transactions)
- Business logic lives with the data — one fix updates all applications at once

---

### 4.2 Block Structure

```sql
[<<label>>]       -- optional name (used for scoping variables in nested blocks)
DECLARE
    -- Section for variable/constant/cursor/exception/type declarations
    -- Everything you declare here is local to this block
BEGIN
    -- All executable statements go here
    -- SQL statements, assignments, loops, procedure calls
EXCEPTION
    -- Runtime error handlers
    -- If an error occurs in BEGIN section, control jumps here
    -- If no matching handler, error propagates to the enclosing block
END [label];
/                 -- the forward slash tells SQL*Plus/Developer to actually execute the block
```

> **Variable scope:** Variables declared in the DECLARE section are accessible from BEGIN through END. Inner (nested) blocks can access outer block variables. Outer blocks cannot access inner block variables. If an inner block declares a variable with the same name as an outer one, the inner one shadows it — use `<<label>>.variable_name` to access the outer one explicitly.

---

### 4.3 Variables and Data Types

```sql
DECLARE
    v_name        VARCHAR2(100);          -- uninitialized → NULL
    v_salary      NUMBER(10,2) := 0;      -- initialized to 0
    v_is_active   BOOLEAN      := TRUE;
    v_date        DATE         := SYSDATE;  -- SYSDATE is a function call, evaluated at declaration
    v_count       PLS_INTEGER  := 0;       -- use PLS_INTEGER for loop counters: faster than NUMBER

    c_max_salary  CONSTANT NUMBER    := 999999;    -- cannot be changed after declaration
    c_currency    CONSTANT VARCHAR2(3) := 'USD';

    v_required    VARCHAR2(50) NOT NULL := 'Required';  -- must initialize; cannot be set to NULL

    -- %TYPE: anchor to a column's type — automatically stays in sync if column type changes
    v_emp_id      employees.employee_id%TYPE;
    v_sal         employees.salary%TYPE;

    -- %ROWTYPE: anchor to an entire row's structure — gives you a record with all columns
    v_emp_row     employees%ROWTYPE;
BEGIN
    -- SELECT INTO fetches exactly one row into a variable
    -- Raises NO_DATA_FOUND if 0 rows, TOO_MANY_ROWS if > 1 row
    SELECT * INTO v_emp_row FROM employees WHERE employee_id = 100;

    DBMS_OUTPUT.PUT_LINE(v_emp_row.first_name || ': $' || v_emp_row.salary);
END;
/
```

---

### 4.4 Control Structures

```sql
-- IF / ELSIF / ELSE
-- ELSIF (not ELSEIF) — Oracle's syntax
IF v_salary >= 100000 THEN
    v_bonus := 0.20;
ELSIF v_salary >= 75000 THEN
    v_bonus := 0.15;
ELSIF v_salary >= 50000 THEN
    v_bonus := 0.10;
ELSE
    v_bonus := 0.05;
END IF;
```

```sql
-- CASE EXPRESSION: evaluates to a value, used on right side of assignment or in SQL
v_gpa := CASE v_grade
    WHEN 'A' THEN 4.0
    WHEN 'B' THEN 3.0
    WHEN 'C' THEN 2.0
    ELSE 0.0
END;

-- CASE STATEMENT: controls program flow (cannot be used as an expression)
CASE v_status
    WHEN 'PENDING'   THEN process_order();
    WHEN 'SHIPPED'   THEN notify_customer();
    WHEN 'CANCELLED' THEN release_inventory();
    ELSE log_unknown_status(v_status);
END CASE;
-- If no ELSE and no match: raises CASE_NOT_FOUND exception
```

```sql
-- LOOP with EXIT WHEN: runs until condition becomes true
LOOP
    v_counter := v_counter + 1;
    EXIT WHEN v_counter >= 5;   -- checked at this point each iteration
END LOOP;

-- WHILE: condition checked BEFORE each iteration (may never run if false from start)
WHILE v_counter <= 5 LOOP
    v_counter := v_counter + 1;
END LOOP;

-- Numeric FOR: counter variable is implicitly declared, read-only inside loop
FOR i IN 1..5 LOOP             -- i goes 1,2,3,4,5
    DBMS_OUTPUT.PUT_LINE(i);
END LOOP;
-- i is not accessible after the loop (goes out of scope)

FOR i IN REVERSE 1..5 LOOP    -- i goes 5,4,3,2,1
    DBMS_OUTPUT.PUT_LINE(i);
END LOOP;

-- CONTINUE: skip the rest of this iteration, go to next
FOR i IN 1..10 LOOP
    CONTINUE WHEN MOD(i, 2) = 0;    -- skip even numbers
    DBMS_OUTPUT.PUT_LINE('Odd: ' || i);
END LOOP;

-- Nested loops with labels: EXIT can jump out of a specific named loop
<<outer_loop>>
FOR i IN 1..10 LOOP
    <<inner_loop>>
    FOR j IN 1..10 LOOP
        IF i * j = 42 THEN
            EXIT outer_loop;   -- exits BOTH inner and outer loop at once
        END IF;
    END LOOP inner_loop;
END LOOP outer_loop;
```

---

### 4.5 Cursors

A **cursor** is a named pointer to the result set of a SQL query. Oracle uses cursors internally for all queries — explicit cursors just give you manual control over that process.

#### Implicit Cursor
Automatically created and destroyed for every DML statement and single-row SELECT INTO. You access its attributes right after the statement.

```sql
UPDATE employees SET salary = salary * 1.05 WHERE department_id = 60;
-- SQL% attributes are valid immediately after the statement, before any other SQL runs
IF SQL%FOUND    THEN DBMS_OUTPUT.PUT_LINE('Updated: ' || SQL%ROWCOUNT || ' rows'); END IF;
IF SQL%NOTFOUND THEN DBMS_OUTPUT.PUT_LINE('No rows matched'); END IF;
-- SQL%ROWCOUNT: how many rows were affected
-- SQL%FOUND: TRUE if at least 1 row was affected
-- SQL%NOTFOUND: TRUE if 0 rows were affected
-- SQL%ISOPEN: always FALSE for implicit cursors (auto-closed)
```

#### Explicit Cursor (full manual control)

```sql
DECLARE
    -- Declare: defines the query but doesn't run it yet
    CURSOR c_employees IS
        SELECT employee_id, first_name, salary
        FROM employees WHERE department_id = 60;
    v_emp c_employees%ROWTYPE;   -- row variable matching the cursor's column set
BEGIN
    OPEN c_employees;   -- executes the query, creates the result set in the server's memory

    LOOP
        FETCH c_employees INTO v_emp;   -- advances pointer, reads next row into v_emp
        EXIT WHEN c_employees%NOTFOUND; -- %NOTFOUND becomes TRUE after the last row is fetched
        DBMS_OUTPUT.PUT_LINE(v_emp.first_name || ': $' || v_emp.salary);
    END LOOP;

    CLOSE c_employees;  -- releases the cursor's memory — IMPORTANT: always close!
END;
/
```

#### Cursor FOR Loop (simplest — Oracle handles OPEN, FETCH, CLOSE automatically)

```sql
-- Using a declared cursor
FOR dept IN c_depts LOOP
    DBMS_OUTPUT.PUT_LINE(dept.department_name);  -- dept is auto-declared record variable
END LOOP;
-- OPEN/FETCH/CLOSE happens automatically — you never write them

-- Inline version: no cursor declaration needed at all
FOR emp IN (SELECT first_name, salary FROM employees WHERE department_id = 60) LOOP
    DBMS_OUTPUT.PUT_LINE(emp.first_name || ': $' || emp.salary);
END LOOP;
```

#### Parameterized Cursor

```sql
-- Cursor parameters let you reuse the same cursor with different values
CURSOR c_dept_emps(p_dept_id NUMBER, p_min_salary NUMBER DEFAULT 0) IS
    SELECT first_name, salary FROM employees
    WHERE department_id = p_dept_id AND salary >= p_min_salary;

-- Use it with different parameter values
FOR emp IN c_dept_emps(60)        LOOP ... END LOOP;  -- all IT employees
FOR emp IN c_dept_emps(60, 8000)  LOOP ... END LOOP;  -- IT employees earning 8000+
FOR emp IN c_dept_emps(80, 10000) LOOP ... END LOOP;  -- Sales employees earning 10000+
```

#### REF CURSOR (dynamic — query is decided at runtime)

```sql
DECLARE
    TYPE emp_cursor IS REF CURSOR;   -- or use SYS_REFCURSOR (predefined type)
    c_emps emp_cursor;
    v_name VARCHAR2(100); v_sal NUMBER;
BEGIN
    -- The OPEN FOR statement assigns the actual query to the cursor variable
    -- This happens at runtime — you can use IF/CASE to pick different queries
    OPEN c_emps FOR SELECT first_name||' '||last_name, salary FROM employees;

    LOOP
        FETCH c_emps INTO v_name, v_sal;
        EXIT WHEN c_emps%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_name || ': $' || v_sal);
    END LOOP;
    CLOSE c_emps;
END;
/
```

#### BULK COLLECT (fetch all rows at once into a collection)

```sql
DECLARE
    TYPE emp_tab IS TABLE OF employees%ROWTYPE;
    v_employees emp_tab;
BEGIN
    -- Single SQL call fetches ALL matching rows into the collection
    -- Much faster than fetching one row at a time in a loop
    SELECT * BULK COLLECT INTO v_employees FROM employees WHERE department_id = 60;

    DBMS_OUTPUT.PUT_LINE('Fetched: ' || v_employees.COUNT || ' employees');
    FOR i IN 1..v_employees.COUNT LOOP
        DBMS_OUTPUT.PUT_LINE(v_employees(i).first_name);
    END LOOP;
END;
/

-- BULK COLLECT with LIMIT: process large tables in batches to control memory usage
OPEN c_all;
LOOP
    FETCH c_all BULK COLLECT INTO v_batch LIMIT 100;  -- read 100 rows at a time
    EXIT WHEN v_batch.COUNT = 0;
    -- process v_batch...
    EXIT WHEN v_batch.COUNT < 100;  -- last batch will be smaller than LIMIT
END LOOP;
CLOSE c_all;
```

---

### 4.6 Exception Handling

When a runtime error occurs in the BEGIN section, control immediately jumps to the EXCEPTION section. Oracle matches the error against the WHEN clauses in order. If no match is found, the exception **propagates** to the enclosing block (or to the caller if no enclosing block).

```sql
DECLARE
    -- User-defined exception: a named signal with no built-in Oracle error code
    e_invalid_salary EXCEPTION;

    -- Map an Oracle error code to a named exception using PRAGMA
    e_fk_violation EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_fk_violation, -2292);   -- ORA-02292: child records exist
BEGIN
    IF v_salary < 0 THEN
        RAISE e_invalid_salary;   -- manually signal the exception
    END IF;

    -- RAISE_APPLICATION_ERROR: create a custom error message visible to the caller
    -- Error numbers must be in range -20000 to -20999
    RAISE_APPLICATION_ERROR(-20001, 'Salary cannot be negative: ' || v_salary);

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No record found');

    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('Query returned more than one row');

    WHEN DUP_VAL_ON_INDEX THEN
        DBMS_OUTPUT.PUT_LINE('Duplicate key violation');

    WHEN e_invalid_salary THEN
        DBMS_OUTPUT.PUT_LINE('Custom: salary is invalid');

    WHEN e_fk_violation THEN
        DBMS_OUTPUT.PUT_LINE('Cannot delete: child records exist');

    WHEN OTHERS THEN
        -- OTHERS catches everything not matched above — always put it last
        DBMS_OUTPUT.PUT_LINE('Error code: ' || SQLCODE);    -- negative Oracle error number
        DBMS_OUTPUT.PUT_LINE('Error text: ' || SQLERRM);    -- full error message
        DBMS_OUTPUT.PUT_LINE('Where: '      || DBMS_UTILITY.FORMAT_ERROR_BACKTRACE);
        RAISE;  -- re-raise to propagate to caller (don't silently swallow errors!)
END;
/
```

**Common predefined exceptions:**

| Exception | ORA Code | When raised |
|-----------|----------|-------------|
| `NO_DATA_FOUND` | -1403 | `SELECT INTO` returned zero rows |
| `TOO_MANY_ROWS` | -1422 | `SELECT INTO` returned more than one row |
| `ZERO_DIVIDE` | -1476 | Division by zero |
| `VALUE_ERROR` | -6502 | Type conversion failed (e.g., 'ABC' into NUMBER) |
| `DUP_VAL_ON_INDEX` | -0001 | Unique/primary key constraint violated |
| `INVALID_CURSOR` | -1001 | Tried to FETCH/CLOSE a cursor that isn't open |
| `CURSOR_ALREADY_OPEN` | -6511 | Tried to OPEN a cursor that's already open |

---

### 4.7 Collections

Collections are PL/SQL's arrays/lists — they hold multiple values of the same type.

#### Associative Array (INDEX BY)
The most flexible. Can be sparse (gaps in index). Index can be integer or string.

```sql
TYPE t_salary_tab IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
TYPE t_name_tab   IS TABLE OF VARCHAR2(100) INDEX BY VARCHAR2(20);

v_salaries t_salary_tab;
v_salaries(100) := 50000;   -- index 100, value 50000
v_salaries(500) := 75000;   -- index 500, no rows 101-499 (sparse)

-- Iterate a string-indexed associative array
v_key := v_names.FIRST;    -- gets the first (smallest) key
WHILE v_key IS NOT NULL LOOP
    DBMS_OUTPUT.PUT_LINE(v_key || ' = ' || v_names(v_key));
    v_key := v_names.NEXT(v_key);  -- advance to next key
END LOOP;
-- NEXT() returns NULL when there are no more keys — that's the loop termination condition
```

#### Nested Table
Dynamic size, starts from index 1. Must be initialized. Can be stored in a database column.

```sql
TYPE t_list IS TABLE OF NUMBER;
v_nums t_list := t_list(10, 20, 30);   -- initialized with 3 elements

v_nums.EXTEND;         -- add one NULL slot at the end (increases COUNT by 1)
v_nums(4) := 40;       -- assign value to new slot

v_nums.EXTEND(2);      -- add 2 slots at once
v_nums.EXTEND(3, 1);   -- add 3 copies of element 1 at the end

v_nums.DELETE(2);      -- remove element at index 2 (makes array sparse)
v_nums.DELETE;         -- remove ALL elements

-- Safe iteration over sparse array
FOR i IN 1..v_nums.LAST LOOP
    IF v_nums.EXISTS(i) THEN      -- check before access to avoid NO_DATA_FOUND
        DBMS_OUTPUT.PUT_LINE(v_nums(i));
    END IF;
END LOOP;
```

#### VARRAY (Variable-size Array)
Fixed maximum size. Dense (no gaps allowed). Good when you know the maximum.

```sql
TYPE t_phones IS VARRAY(5) OF VARCHAR2(20);   -- maximum 5 elements
v_phones t_phones := t_phones('555-1111', '555-2222');

v_phones.EXTEND;
v_phones(3) := '555-3333';

DBMS_OUTPUT.PUT_LINE('Count: ' || v_phones.COUNT);   -- 3 (current elements)
DBMS_OUTPUT.PUT_LINE('Limit: ' || v_phones.LIMIT);   -- 5 (maximum)
-- v_phones.EXTEND when COUNT = LIMIT raises SUBSCRIPT_BEYOND_COUNT
```

**Collection methods summary:**

| Method | Returns | Description |
|--------|---------|-------------|
| `.COUNT` | NUMBER | Current number of elements |
| `.FIRST` | Index type | First (smallest) index |
| `.LAST` | Index type | Last (largest) index |
| `.NEXT(n)` | Index type | Next index after n (NULL if none) |
| `.PRIOR(n)` | Index type | Previous index before n |
| `.EXISTS(n)` | BOOLEAN | TRUE if element at index n exists |
| `.EXTEND` | — | Add one null element (Nested Table/VARRAY) |
| `.EXTEND(n)` | — | Add n null elements |
| `.TRIM` | — | Remove last element |
| `.DELETE` | — | Remove all elements |
| `.DELETE(n)` | — | Remove element at index n |

---

### 4.8 BULK Operations

The single most impactful performance technique in PL/SQL. The problem: a loop that executes one SQL per iteration means **context-switching** between the PL/SQL engine and the SQL engine for every row. BULK operations batch all the data transfers into a single context switch.

```sql
-- FORALL: executes a single DML statement once per collection element,
-- but all iterations are submitted as a SINGLE batch to the SQL engine
-- Rule: only ONE DML statement inside FORALL (not a loop with multiple statements)
DECLARE
    TYPE t_id_tab  IS TABLE OF NUMBER;
    TYPE t_sal_tab IS TABLE OF NUMBER;
    v_ids      t_id_tab;
    v_salaries t_sal_tab;
BEGIN
    -- BULK COLLECT fills both collections in one SQL call
    SELECT employee_id, salary * 1.05
    BULK COLLECT INTO v_ids, v_salaries
    FROM employees WHERE department_id = 60;

    -- FORALL submits all updates as a single batch — avoids N context switches
    FORALL i IN 1..v_ids.COUNT
        UPDATE employees
        SET salary = v_salaries(i)
        WHERE employee_id = v_ids(i);
    -- This is one trip from PL/SQL engine to SQL engine, not v_ids.COUNT trips

    DBMS_OUTPUT.PUT_LINE('Updated: ' || SQL%ROWCOUNT || ' rows');
    COMMIT;
END;
/
```

> **Performance numbers:** A row-by-row loop updating 10,000 rows does 10,000 context switches. A FORALL over the same 10,000 elements does 1 context switch. Real-world speedup is typically 10x to 100x.

```sql
-- FORALL with SAVE EXCEPTIONS: don't stop on first error — collect all errors and continue
DECLARE
    e_bulk_errors EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_bulk_errors, -24381);  -- ORA-24381: error in bulk DML
BEGIN
    FORALL i IN 1..v_ids.COUNT SAVE EXCEPTIONS
        DELETE FROM employees WHERE employee_id = v_ids(i);

EXCEPTION
    WHEN e_bulk_errors THEN
        -- SQL%BULK_EXCEPTIONS is a collection of error info, one per failed iteration
        FOR j IN 1..SQL%BULK_EXCEPTIONS.COUNT LOOP
            DBMS_OUTPUT.PUT_LINE(
                'Failed at index ' || SQL%BULK_EXCEPTIONS(j).ERROR_INDEX ||
                ': ' || SQLERRM(-SQL%BULK_EXCEPTIONS(j).ERROR_CODE)
            );
        END LOOP;
        -- Successful iterations are still committed (unless you ROLLBACK here)
END;
/

-- FORALL with INDICES OF: use only the populated indexes of a sparse array
FORALL i IN INDICES OF v_emp_ids     -- skips missing indexes (e.g., after .DELETE)
    UPDATE employees SET salary = 0 WHERE employee_id = v_emp_ids(i);

-- FORALL with VALUES OF: the collection contains the indexes to use from another collection
FORALL i IN VALUES OF v_active_indexes
    UPDATE employees SET status = 'ACTIVE' WHERE employee_id = v_emp_ids(i);
```

---

## 5. PL/SQL Database Objects

---

### 5.1 Stored Procedures

A stored procedure is a **named, compiled PL/SQL program** stored permanently in the database. Unlike anonymous blocks, it is compiled once and its execution plan is cached. Callers need EXECUTE privilege but not SELECT/INSERT/UPDATE privileges on the underlying tables.

```sql
CREATE OR REPLACE PROCEDURE hire_employee(
    p_first_name    IN  VARCHAR2,          -- IN: caller passes value in; cannot be modified
    p_last_name     IN  VARCHAR2,
    p_salary        IN  NUMBER,
    p_department_id IN  NUMBER,
    p_employee_id   OUT NUMBER,            -- OUT: procedure writes value for caller to use
    p_message       OUT VARCHAR2           -- another OUT parameter
) IS
    -- Local variable declarations (like DECLARE section)
    v_min_salary NUMBER;
    v_max_salary NUMBER;
BEGIN
    -- Validate salary range for the job
    SELECT min_salary, max_salary INTO v_min_salary, v_max_salary
    FROM jobs WHERE job_id = 'IT_PROG';

    IF p_salary NOT BETWEEN v_min_salary AND v_max_salary THEN
        -- Write to OUT parameter (caller gets this message)
        p_message := 'Salary ' || p_salary || ' is out of range [' || v_min_salary || ',' || v_max_salary || ']';
        p_employee_id := NULL;
        RETURN;   -- exit procedure early without raising exception
    END IF;

    SELECT employees_seq.NEXTVAL INTO p_employee_id FROM dual;

    INSERT INTO employees (employee_id, first_name, last_name, salary, department_id, hire_date)
    VALUES (p_employee_id, p_first_name, p_last_name, p_salary, p_department_id, SYSDATE);

    p_message := 'Success: employee ' || p_employee_id || ' hired';
    COMMIT;

EXCEPTION
    WHEN DUP_VAL_ON_INDEX THEN
        p_message := 'Duplicate record';
        p_employee_id := NULL;
        ROLLBACK;
    WHEN OTHERS THEN
        p_message := 'Error: ' || SQLERRM;
        p_employee_id := NULL;
        ROLLBACK;
        RAISE;   -- re-raise after logging
END hire_employee;
/

-- Calling the procedure: positional notation
DECLARE v_id NUMBER; v_msg VARCHAR2(200);
BEGIN
    hire_employee('John', 'Smith', 6000, 60, v_id, v_msg);
    DBMS_OUTPUT.PUT_LINE('ID=' || v_id || ' MSG=' || v_msg);
END;
/

-- Named notation (preferred for readability and optional parameters)
BEGIN
    hire_employee(
        p_first_name    => 'Jane',
        p_last_name     => 'Doe',
        p_salary        => 7000,
        p_department_id => 80,
        p_employee_id   => v_id,
        p_message       => v_msg
    );
END;
/
```

**Parameter modes:**

| Mode | Direction | Readable? | Writable? | Default value? |
|------|-----------|-----------|-----------|----------------|
| `IN` | Caller → Procedure | ✓ | ✗ | ✓ (use DEFAULT) |
| `OUT` | Procedure → Caller | ✗ | ✓ | ✗ |
| `IN OUT` | Both directions | ✓ | ✓ | ✗ |

```sql
-- NOCOPY: for large collections, avoid copying the whole collection on call
-- By default, IN OUT passes a copy — modification is isolated until procedure ends
-- NOCOPY passes a reference — faster, but if exception occurs, partial changes may be visible
PROCEDURE process(p_data IN OUT NOCOPY large_collection_type) IS ...
```

---

### 5.2 Stored Functions

A function is like a procedure but it **must return exactly one value**, and it can be called directly from SQL statements (SELECT, WHERE, ORDER BY) as well as from PL/SQL.

```sql
CREATE OR REPLACE FUNCTION get_annual_salary(p_employee_id IN NUMBER)
RETURN NUMBER IS       -- declares the return type
    v_salary NUMBER;
BEGIN
    SELECT salary * 12 INTO v_salary FROM employees WHERE employee_id = p_employee_id;
    RETURN v_salary;   -- RETURN is mandatory and exits the function
EXCEPTION
    WHEN NO_DATA_FOUND THEN RETURN NULL;   -- safe: return NULL for missing employees
END get_annual_salary;
/

-- Use the function in SQL
SELECT employee_id, first_name, get_annual_salary(employee_id) AS annual_salary
FROM employees WHERE department_id = 60;

-- DETERMINISTIC: Oracle can cache the result per input combination
-- Required for function-based indexes (Oracle must guarantee same output for same input)
CREATE OR REPLACE FUNCTION get_tax_rate(p_income NUMBER) RETURN NUMBER
DETERMINISTIC IS
BEGIN
    RETURN CASE
        WHEN p_income > 500000 THEN 0.35
        WHEN p_income > 200000 THEN 0.28
        ELSE 0.15
    END;
END;
/

-- RESULT_CACHE: Oracle caches results in the shared pool across all sessions
-- Cache is invalidated automatically when the RELIES_ON table changes
CREATE OR REPLACE FUNCTION get_config(p_key VARCHAR2) RETURN VARCHAR2
RESULT_CACHE RELIES_ON (configuration) IS
    v_value VARCHAR2(4000);
BEGIN
    SELECT config_value INTO v_value FROM configuration WHERE config_key = p_key;
    RETURN v_value;
EXCEPTION
    WHEN NO_DATA_FOUND THEN RETURN NULL;
END;
/
-- First call for 'MAX_RETRIES': hits the table
-- Second call for 'MAX_RETRIES' (any session): returns cached value instantly
```

#### Pipelined Table Function

Returns rows one at a time using `PIPE ROW`, which allows the calling query to start reading results before the function finishes processing. Memory efficient for large datasets.

```sql
-- Step 1: Define the types (must be schema-level, not PL/SQL-level, for use in SQL)
CREATE OR REPLACE TYPE emp_row AS OBJECT (
    employee_id NUMBER,
    name        VARCHAR2(100),
    salary      NUMBER
);
/
CREATE OR REPLACE TYPE emp_table AS TABLE OF emp_row;
/

-- Step 2: Create the function with PIPELINED keyword
CREATE OR REPLACE FUNCTION get_high_earners(p_min NUMBER DEFAULT 10000)
RETURN emp_table
PIPELINED IS          -- PIPELINED: function uses PIPE ROW instead of accumulating all results
BEGIN
    FOR r IN (SELECT employee_id, first_name||' '||last_name AS name, salary
              FROM employees WHERE salary >= p_min ORDER BY salary DESC)
    LOOP
        PIPE ROW(emp_row(r.employee_id, r.name, r.salary));
        -- Each PIPE ROW sends one row to the calling query immediately
        -- The calling query can start processing these rows while the loop continues
    END LOOP;
    RETURN;  -- required syntax (no value after RETURN for pipelined functions)
END;
/

-- Usage: call it with TABLE() operator in FROM clause
SELECT * FROM TABLE(get_high_earners(15000)) ORDER BY salary DESC;
SELECT * FROM TABLE(get_high_earners()) WHERE name LIKE 'S%';
```

---

### 5.3 Packages

A package groups related procedures, functions, types, constants, and exceptions into a single named unit. It has two parts:

- **Specification** (the public interface — what callers can use)
- **Body** (the implementation — how it works internally + private members)

```sql
-- SPECIFICATION: only declares, never implements
CREATE OR REPLACE PACKAGE emp_mgmt AS
    -- Public types (callers can use these)
    TYPE t_emp_rec IS RECORD (employee_id NUMBER, name VARCHAR2(100), salary NUMBER);
    TYPE t_emp_tab IS TABLE OF t_emp_rec INDEX BY PLS_INTEGER;

    -- Public constants
    c_max_salary  CONSTANT NUMBER := 100000;

    -- Public exceptions (callers can catch these)
    e_salary_exceeded EXCEPTION;

    -- Public procedure headers (implementation is in BODY)
    PROCEDURE hire_employee(p_name VARCHAR2, p_salary NUMBER, p_emp_id OUT NUMBER);
    PROCEDURE terminate_employee(p_emp_id NUMBER);

    -- Public function headers
    FUNCTION get_employee_count(p_dept_id NUMBER) RETURN NUMBER;

END emp_mgmt;
/

-- BODY: implements everything declared in spec, plus private members
CREATE OR REPLACE PACKAGE BODY emp_mgmt AS

    -- PRIVATE variable: only accessible within this package body
    -- Persists for the entire session (package state) — shared across calls in the same session
    g_last_emp_id NUMBER;

    -- PRIVATE procedure: callers outside the package cannot call this
    PROCEDURE log_activity(p_emp_id NUMBER, p_action VARCHAR2) IS
        PRAGMA AUTONOMOUS_TRANSACTION;   -- commits independently of the main transaction
    BEGIN
        INSERT INTO activity_log VALUES (p_emp_id, p_action, SYSDATE, USER);
        COMMIT;
    END;

    -- Public procedure IMPLEMENTATION
    PROCEDURE hire_employee(p_name VARCHAR2, p_salary NUMBER, p_emp_id OUT NUMBER) IS
    BEGIN
        IF p_salary > c_max_salary THEN
            RAISE e_salary_exceeded;   -- callers will catch this
        END IF;
        SELECT emp_seq.NEXTVAL INTO p_emp_id FROM dual;
        INSERT INTO employees (employee_id, employee_name, salary)
        VALUES (p_emp_id, p_name, p_salary);
        g_last_emp_id := p_emp_id;   -- update package state
        log_activity(p_emp_id, 'HIRE');   -- call private procedure
        COMMIT;
    END;

    FUNCTION get_employee_count(p_dept_id NUMBER) RETURN NUMBER IS
        v_count NUMBER;
    BEGIN
        SELECT COUNT(*) INTO v_count FROM employees WHERE department_id = p_dept_id;
        RETURN v_count;
    END;

-- INITIALIZATION BLOCK: runs ONCE per session, the first time the package is accessed
BEGIN
    g_last_emp_id := 0;
END emp_mgmt;
/

-- Usage
DECLARE v_id NUMBER;
BEGIN
    emp_mgmt.hire_employee('John Smith', 50000, v_id);
    DBMS_OUTPUT.PUT_LINE('Count in dept 60: ' || emp_mgmt.get_employee_count(60));
    DBMS_OUTPUT.PUT_LINE('Max salary: ' || emp_mgmt.c_max_salary);
EXCEPTION
    WHEN emp_mgmt.e_salary_exceeded THEN
        DBMS_OUTPUT.PUT_LINE('Salary too high');
END;
/
```

> **Why packages beat standalone procedures:** Packages keep related code together, hide implementation details (private members), share state across calls (package variables), and allow overloading. Also, when any package member is first called, Oracle loads the entire package specification into the shared pool — subsequent calls to other package members find them already loaded (faster).

**Overloading** — same name, different signatures:
```sql
CREATE OR REPLACE PACKAGE util_pkg AS
    FUNCTION format_name(p_first VARCHAR2, p_last VARCHAR2) RETURN VARCHAR2;
    FUNCTION format_name(p_employee_id NUMBER)              RETURN VARCHAR2;
    -- Oracle picks the right one based on what arguments you pass
END;
/
```

---

### 5.4 Database Triggers

A trigger is a stored program that **automatically fires** when a specific event occurs on a table, view, schema, or database. You never call a trigger — the database calls it.

#### BEFORE Row-Level Trigger (validation + auto-fill)

```sql
CREATE OR REPLACE TRIGGER trg_employee_validation
BEFORE INSERT OR UPDATE ON employees   -- fires BEFORE the DML actually changes data
FOR EACH ROW                           -- fires once per affected row (vs once per statement)
DECLARE
    v_min NUMBER; v_max NUMBER;
BEGIN
    -- Auto-generate PK on INSERT if not provided
    IF INSERTING AND :NEW.employee_id IS NULL THEN
        :NEW.employee_id := employee_seq.NEXTVAL;
        -- :NEW.col = modify the value BEFORE it goes into the table
    END IF;

    -- Auto-set audit timestamps
    IF INSERTING THEN
        :NEW.created_date := SYSDATE;
        :NEW.created_by   := USER;
    END IF;
    :NEW.modified_date := SYSDATE;   -- runs for both INSERT and UPDATE
    :NEW.modified_by   := USER;

    -- Validate salary against job range
    SELECT min_salary, max_salary INTO v_min, v_max FROM jobs WHERE job_id = :NEW.job_id;
    IF :NEW.salary NOT BETWEEN v_min AND v_max THEN
        RAISE_APPLICATION_ERROR(-20001, 'Salary ' || :NEW.salary || ' out of range for ' || :NEW.job_id);
        -- RAISE_APPLICATION_ERROR in BEFORE trigger ROLLS BACK the entire DML
    END IF;

    -- Normalize data
    :NEW.email := LOWER(TRIM(:NEW.email));
END;
/
```

> **`:NEW` and `:OLD`:** These are special record variables available in row-level triggers. `:NEW` holds the values that will be inserted/updated into the row. Modifying `:NEW` in a BEFORE trigger changes what actually gets stored. `:OLD` holds the values before the change (NULL for INSERT). In AFTER triggers, `:NEW` is read-only (data is already committed to the block).

#### AFTER Row-Level Trigger (auditing)

```sql
CREATE OR REPLACE TRIGGER trg_salary_audit
AFTER UPDATE OF salary ON employees   -- only fires when salary column is specifically updated
FOR EACH ROW
BEGIN
    INSERT INTO salary_audit (
        employee_id, old_salary, new_salary, changed_by,
        change_type, changed_date
    ) VALUES (
        :NEW.employee_id, :OLD.salary, :NEW.salary, USER,
        CASE WHEN :NEW.salary > :OLD.salary THEN 'INCREASE' ELSE 'DECREASE' END,
        SYSDATE
    );
    -- AFTER trigger: the UPDATE is already done; we're just logging it
END;
/
```

**`:NEW` / `:OLD` availability by DML type:**

| DML | `:OLD` value | `:NEW` value |
|-----|-------------|-------------|
| INSERT | NULL (no prior data) | Values being inserted |
| UPDATE | Values before change | Values after change |
| DELETE | Values being deleted | NULL (no future data) |

```sql
-- Test which DML fired the trigger using predicates
CASE
    WHEN INSERTING THEN 'INSERT'
    WHEN UPDATING  THEN 'UPDATE'    -- UPDATING('salary') tests a specific column
    WHEN DELETING  THEN 'DELETE'
END
```

#### INSTEAD OF Trigger (make views updatable)

Normal views based on joins are not updatable. `INSTEAD OF` triggers intercept the DML and redirect it to the base tables.

```sql
CREATE OR REPLACE VIEW emp_dept_view AS
SELECT e.employee_id, e.employee_name, e.salary, d.department_name
FROM employees e JOIN departments d ON e.department_id = d.department_id;

CREATE OR REPLACE TRIGGER trg_emp_dept_insert
INSTEAD OF INSERT ON emp_dept_view    -- fires INSTEAD OF the INSERT on the view
FOR EACH ROW
DECLARE v_dept_id NUMBER;
BEGIN
    -- Look up department_id from the name the caller provided
    SELECT department_id INTO v_dept_id FROM departments
    WHERE department_name = :NEW.department_name;

    -- Redirect insert to the base table
    INSERT INTO employees (employee_id, employee_name, salary, department_id)
    VALUES (employee_seq.NEXTVAL, :NEW.employee_name, :NEW.salary, v_dept_id);
END;
/
-- Now: INSERT INTO emp_dept_view (employee_name, salary, department_name) VALUES (...)
-- works even though the view is based on a join
```

#### Compound Trigger (solves mutating table problem + efficient bulk auditing)

```sql
-- MUTATING TABLE ERROR: a row-level trigger cannot query or modify the table it's on
-- Example: a trigger on employees cannot SELECT from employees
-- Compound trigger collects data row-by-row, then acts ONCE after all rows are processed

CREATE OR REPLACE TRIGGER trg_employee_compound
FOR INSERT OR UPDATE OR DELETE ON employees
COMPOUND TRIGGER

    -- Shared state: declared here, accessible in all timing sections
    TYPE emp_id_list IS TABLE OF NUMBER;
    g_emp_ids emp_id_list := emp_id_list();

    -- Section 1: BEFORE any rows are processed (good for initialization)
    BEFORE STATEMENT IS
    BEGIN
        g_emp_ids := emp_id_list();   -- reset the collection for each statement
    END BEFORE STATEMENT;

    -- Section 2: AFTER each row is processed (collect the IDs)
    AFTER EACH ROW IS
    BEGIN
        g_emp_ids.EXTEND;
        g_emp_ids(g_emp_ids.COUNT) := NVL(:NEW.employee_id, :OLD.employee_id);
    END AFTER EACH ROW;

    -- Section 3: AFTER all rows processed (bulk insert audit records)
    AFTER STATEMENT IS
    BEGIN
        -- Now we can safely query employees table (all row-level changes are done)
        FORALL i IN 1..g_emp_ids.COUNT
            INSERT INTO employee_audit VALUES (
                audit_seq.NEXTVAL, g_emp_ids(i), 'MODIFIED', SYSDATE, USER
            );
    END AFTER STATEMENT;

END;
/
```

#### DDL and System Triggers

```sql
-- Audit every DDL operation in the schema
CREATE OR REPLACE TRIGGER trg_ddl_audit
AFTER DDL ON SCHEMA     -- ON SCHEMA = this user's schema; ON DATABASE = all schemas
BEGIN
    INSERT INTO ddl_audit_log VALUES (
        ddl_seq.NEXTVAL,
        ORA_SYSEVENT,        -- 'CREATE', 'ALTER', 'DROP', etc.
        ORA_DICT_OBJ_TYPE,   -- 'TABLE', 'INDEX', 'PROCEDURE', etc.
        ORA_DICT_OBJ_NAME,   -- name of the affected object
        USER,
        SYSTIMESTAMP
    );
END;
/

-- Prevent dropping critical tables
CREATE OR REPLACE TRIGGER trg_prevent_drop
BEFORE DROP ON SCHEMA
BEGIN
    IF ORA_DICT_OBJ_TYPE = 'TABLE' AND
       ORA_DICT_OBJ_NAME IN ('EMPLOYEES', 'DEPARTMENTS', 'ORDERS') THEN
        RAISE_APPLICATION_ERROR(-20001, 'Cannot drop critical table: ' || ORA_DICT_OBJ_NAME);
    END IF;
END;
/

-- Log every login
CREATE OR REPLACE TRIGGER trg_after_logon
AFTER LOGON ON DATABASE
BEGIN
    INSERT INTO user_login_log VALUES (
        login_seq.NEXTVAL, USER, SYSTIMESTAMP,
        SYS_CONTEXT('USERENV', 'IP_ADDRESS')
    );
    COMMIT;
EXCEPTION
    WHEN OTHERS THEN NULL;  -- never let a logon trigger block users from connecting
END;
/

-- Trigger management
ALTER TRIGGER trg_salary_audit DISABLE;      -- temporarily turn off
ALTER TRIGGER trg_salary_audit ENABLE;       -- turn back on
ALTER TABLE employees DISABLE ALL TRIGGERS;  -- disable all triggers on a table
ALTER TABLE employees ENABLE ALL TRIGGERS;
DROP TRIGGER trg_salary_audit;
```

---

### 5.5 Transaction Management

```sql
-- SAVEPOINT: creates a named checkpoint within a transaction
-- You can ROLLBACK TO a savepoint without losing earlier work
BEGIN
    INSERT INTO orders (order_id, customer_id) VALUES (seq.NEXTVAL, 101);
    SAVEPOINT after_order;               -- checkpoint 1

    INSERT INTO order_items VALUES (seq.NEXTVAL, 1, 50001, 2, 1200.00);
    SAVEPOINT after_items;               -- checkpoint 2

    -- If inventory check fails, only undo items; keep the order itself
    IF check_inventory_fails THEN
        ROLLBACK TO after_order;         -- undoes order_items, keeps orders insert
    ELSE
        UPDATE inventory SET qty = qty - 2 WHERE product_id = 50001;
        COMMIT;                          -- makes all changes permanent; releases all locks
    END IF;
END;
/

-- ROLLBACK: undoes all changes back to the last COMMIT (or to a savepoint)
ROLLBACK;               -- undo everything
ROLLBACK TO after_order;  -- undo back to savepoint
```

#### Autonomous Transaction

A pragma that makes a procedure's transaction completely independent from the caller's transaction.

```sql
-- This procedure commits its INSERT even if the caller later rolls back
CREATE OR REPLACE PROCEDURE log_error(p_error_msg VARCHAR2) IS
    PRAGMA AUTONOMOUS_TRANSACTION;   -- tells Oracle: this is a separate transaction
BEGIN
    INSERT INTO error_log (log_time, message, logged_by)
    VALUES (SYSDATE, p_error_msg, USER);
    COMMIT;   -- commits ONLY the INSERT above — does not affect caller's transaction
END;
/

-- Demonstration:
BEGIN
    INSERT INTO orders VALUES (999, 100, 1500);      -- this insert...
    log_error('Test autonomous');                     -- ...this log ALWAYS commits...
    ROLLBACK;                                         -- ...but this rollback ONLY affects orders
END;
/
-- Result: orders insert is rolled back; error_log insert is permanent
```

> **Use cases for autonomous transactions:** Error/audit logging (you want the log even when the main transaction fails), DDL inside procedures (DDL issues an implicit commit — wrapping it in an autonomous transaction isolates that commit), and activity tracking.

---

## 6. Concurrency & Locking

---

### 6.1 How Oracle's Multi-User Architecture Works

Oracle uses **MVCC (Multi-Version Concurrency Control)** — the most important thing to understand about Oracle locking:

> **Readers never block writers. Writers never block readers. Only writers block other writers (on the same rows).**

When Session A updates a row, Oracle:
1. Writes the **new value** to the data block in the buffer cache
2. Writes the **old value** to an **undo segment** (also called a rollback segment)
3. Marks the row with a lock flag

When Session B reads that same row:
- Oracle sees the lock flag and checks the undo segment
- It reconstructs the **old value** from undo and returns that to Session B
- Session B reads the committed version from before Session A's update started — no waiting

This is why `SELECT` queries in Oracle never wait for `UPDATE/INSERT/DELETE` to finish, and vice versa.

---

### 6.2 SELECT FOR UPDATE

Normal SELECT acquires no locks. When you intend to update a row immediately after reading it, you need to lock it at SELECT time to prevent another session from changing it between your SELECT and UPDATE.

```sql
-- Locks all rows returned by the query; other sessions' updates/deletes on these rows will wait
SELECT employee_id, salary FROM employees WHERE department_id = 60 FOR UPDATE;

-- Lock only on specific columns (still locks the whole row, just semantically documents intent)
SELECT * FROM employees WHERE employee_id = 100 FOR UPDATE OF salary;

-- Lock rows from only one table in a join
SELECT e.employee_id, e.salary, d.department_name
FROM employees e JOIN departments d ON e.department_id = d.department_id
WHERE e.department_id = 60
FOR UPDATE OF e.salary;    -- locks employees rows but NOT departments rows

-- NOWAIT: fail immediately if any row is already locked by another session
-- ORA-00054: resource busy and acquire with NOWAIT specified
SELECT * FROM employees WHERE employee_id = 100 FOR UPDATE NOWAIT;

-- WAIT n: wait up to n seconds, then fail
-- ORA-30006: resource busy; acquire with WAIT timeout expired
SELECT * FROM employees WHERE employee_id = 100 FOR UPDATE WAIT 5;

-- SKIP LOCKED: skip rows that are locked; process only what's available right now
-- This is the key technique for building multi-worker queue processors
SELECT job_id, job_type, payload
FROM job_queue WHERE status = 'PENDING'
ORDER BY priority
FOR UPDATE SKIP LOCKED
FETCH FIRST 10 ROWS ONLY;
-- Worker 1 gets jobs 1,2,3; Worker 2 starts and gets jobs 4,5,6 (1,2,3 are locked, so skipped)
```

#### WHERE CURRENT OF — update the exact row the cursor is on

```sql
DECLARE
    CURSOR c_employees IS
        SELECT employee_id, salary FROM employees WHERE department_id = 60
        FOR UPDATE OF salary;  -- must include FOR UPDATE for WHERE CURRENT OF to work
BEGIN
    FOR emp IN c_employees LOOP
        UPDATE employees
        SET salary = emp.salary * 1.10
        WHERE CURRENT OF c_employees;   -- Oracle knows exactly which physical row to update
        -- More efficient than WHERE employee_id = emp.employee_id
        -- because Oracle uses the ROWID directly (no index lookup needed)
    END LOOP;
    COMMIT;
END;
/
```

---

### 6.3 Explicit Table Locking

```sql
-- SHARE MODE: allows other sessions to read and place row locks, but not exclusive table ops
LOCK TABLE employees IN SHARE MODE;

-- EXCLUSIVE MODE: no other session can modify the table at all
LOCK TABLE employees IN EXCLUSIVE MODE;

-- NOWAIT/WAIT options work the same as FOR UPDATE
LOCK TABLE employees IN EXCLUSIVE MODE NOWAIT;
LOCK TABLE employees IN EXCLUSIVE MODE WAIT 10;
```

> **When to use explicit table locks:** When you're doing a batch update that reads then modifies, and you can't tolerate any concurrent changes to the table during the process. Normally row-level locking is sufficient and preferred.

---

### 6.4 Deadlocks and How to Prevent Them

A **deadlock** occurs when Session A holds a lock that Session B needs, AND Session B holds a lock that Session A needs — a circular wait. Neither can proceed. Oracle's background process detects the cycle within seconds and rolls back one session's statement (not the whole transaction) with ORA-00060.

```sql
-- Classic deadlock scenario:
-- Session 1: locks row 100, then tries to lock row 200
UPDATE accounts SET balance = 1000 WHERE account_id = 100;  -- acquires lock on 100
-- (Session 2 runs here, locks 200, then tries to lock 100 → deadlock)
UPDATE accounts SET balance = 1500 WHERE account_id = 200;  -- waits for 200

-- Session 2: locks row 200, then tries to lock row 100
UPDATE accounts SET balance = 2000 WHERE account_id = 200;  -- acquires lock on 200
UPDATE accounts SET balance = 2500 WHERE account_id = 100;  -- waits for 100 → DEADLOCK
```

**Prevention Strategy 1: Always lock in the same order**

```sql
CREATE OR REPLACE PROCEDURE transfer_funds(p_from NUMBER, p_to NUMBER, p_amount NUMBER) IS
    v_first NUMBER; v_second NUMBER;
    v_balance NUMBER;
BEGIN
    -- Determine lock order: always lock the lower account_id first
    -- This guarantees both sessions try to lock in the same order
    IF p_from < p_to THEN
        v_first := p_from; v_second := p_to;
    ELSE
        v_first := p_to;   v_second := p_from;
    END IF;

    SELECT balance INTO v_balance FROM accounts WHERE account_id = v_first  FOR UPDATE;
    SELECT balance INTO v_balance FROM accounts WHERE account_id = v_second FOR UPDATE;

    -- Now perform the transfer (both accounts are safely locked)
    UPDATE accounts SET balance = balance - p_amount WHERE account_id = p_from;
    UPDATE accounts SET balance = balance + p_amount WHERE account_id = p_to;
    COMMIT;
END;
/
```

**Prevention Strategy 2: NOWAIT with retry and exponential backoff**

```sql
DECLARE
    e_resource_busy EXCEPTION;  PRAGMA EXCEPTION_INIT(e_resource_busy, -54);
    e_deadlock      EXCEPTION;  PRAGMA EXCEPTION_INIT(e_deadlock, -60);
    v_retries       NUMBER := 0;
    c_max_retries   CONSTANT NUMBER := 3;
BEGIN
    WHILE v_retries < c_max_retries LOOP
        BEGIN
            UPDATE employees SET salary = 50000 WHERE employee_id = 100;
            COMMIT;
            EXIT;  -- success: leave the loop

        EXCEPTION
            WHEN e_resource_busy OR e_deadlock THEN
                ROLLBACK;
                v_retries := v_retries + 1;
                -- Exponential backoff: wait 0.2s, 0.4s, 0.8s between retries
                DBMS_LOCK.SLEEP(POWER(2, v_retries) * 0.1);
        END;
    END LOOP;

    IF v_retries = c_max_retries THEN
        RAISE_APPLICATION_ERROR(-20001, 'Failed after ' || c_max_retries || ' retries');
    END IF;
END;
/
```

---

### 6.5 Isolation Levels

**Read Committed (default):** Each SQL statement sees data as it was at the moment that statement started. If another session commits between your first SELECT and your second SELECT, your second SELECT will see the new data.

**Serializable:** The entire transaction sees data as it was at the moment the transaction started. No matter what other sessions commit during your transaction, you always see the same snapshot.

```sql
-- Set isolation for one transaction (applies until COMMIT/ROLLBACK)
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Set isolation for the session (all subsequent transactions use this)
ALTER SESSION SET ISOLATION_LEVEL = SERIALIZABLE;

-- Handle the case where another session changed data we're trying to update (ORA-08177)
DECLARE
    e_serialization EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_serialization, -8177);
BEGIN
    SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    -- ... read data and compute changes ...
    UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 100;
    COMMIT;

EXCEPTION
    WHEN e_serialization THEN
        ROLLBACK;
        -- Retry the whole transaction from scratch
        DBMS_OUTPUT.PUT_LINE('Serialization failure: retry the transaction');
        RAISE;
END;
/
```

---

### 6.6 Finding and Killing Blocking Sessions

```sql
-- Who is blocked and who is blocking them?
SELECT
    s1.sid         AS blocked_sid,
    s1.username    AS blocked_user,
    s1.event       AS waiting_for,
    s1.seconds_in_wait AS wait_seconds,
    s2.sid         AS blocker_sid,
    s2.username    AS blocker_user,
    s2.sql_id      AS blocker_current_sql
FROM v$session s1
JOIN v$session s2 ON s1.blocking_session = s2.sid;

-- See the blocking chain as a hierarchy (who blocks who blocks who)
SELECT level, SID, LPAD(' ', (level-1)*2) || sid AS chain,
       username, event, seconds_in_wait, blocking_session
FROM v$session
START WITH blocking_session IS NULL
    AND sid IN (SELECT blocking_session FROM v$session WHERE blocking_session IS NOT NULL)
CONNECT BY PRIOR sid = blocking_session;

-- See what SQL the root blocker is running (or last ran)
SELECT s.sid, s.username, s.program, sq.sql_text
FROM v$session s
LEFT JOIN v$sql sq ON s.sql_id = sq.sql_id AND sq.child_number = 0
WHERE s.sid = &blocking_sid;

-- Kill the blocking session
-- Format: 'SID,SERIAL#' — serial# ensures you don't accidentally kill a reused SID
ALTER SYSTEM KILL SESSION '142,5678';           -- waits for graceful cleanup
ALTER SYSTEM KILL SESSION '142,5678' IMMEDIATE; -- forces immediate termination

-- Cancel just the blocking SQL without killing the session (Oracle 12c+)
-- Less disruptive: the session stays alive and can issue new statements
ALTER SYSTEM CANCEL SQL '142,5678';
```

---

### 6.7 Dynamic SQL (EXECUTE IMMEDIATE)

Dynamic SQL lets you build and run SQL statements as strings at runtime — needed when table names, column names, or the structure of the query isn't known until runtime.

```sql
-- Basic SELECT with bind variable (ALWAYS use binds, never concatenate user input)
EXECUTE IMMEDIATE 'SELECT COUNT(*) FROM employees WHERE department_id = :1'
    INTO v_count
    USING p_dept_id;
-- The :1 placeholder is filled by the USING clause — Oracle handles quoting/escaping automatically
-- This is safe from SQL injection; concatenating p_dept_id directly would not be

-- DML with multiple bind variables
EXECUTE IMMEDIATE 'UPDATE ' || v_table || ' SET status = :1 WHERE id = :2'
    USING 'ACTIVE', v_id;
-- Note: table name is still concatenated (table names can't be bind variables)
-- Always validate v_table against a whitelist before using it in string concatenation

-- DDL: no bind variables possible; implicit commit issued after DDL
EXECUTE IMMEDIATE 'CREATE TABLE ' || v_table_name || ' AS SELECT * FROM source WHERE 1=0';

-- INSERT with RETURNING (captures the inserted value)
EXECUTE IMMEDIATE
    'INSERT INTO emp VALUES (emp_seq.NEXTVAL, :1, :2) RETURNING employee_id INTO :3'
    USING p_name, p_salary
    RETURNING INTO v_new_id;

-- Dynamic cursor (OPEN FOR): assigns a runtime SQL string to a REF CURSOR
OPEN p_results FOR
    'SELECT * FROM ' || v_table || ' WHERE last_name = :name'
    USING p_name;
-- Caller then FETCHes from p_results like any other cursor

-- VALIDATE identifiers to prevent injection when table/column names are dynamic
RETURN DBMS_ASSERT.SIMPLE_SQL_NAME(p_table_name);
-- SIMPLE_SQL_NAME raises ORA-44002 if the name contains special characters or SQL keywords
```

---

## 7. SQL Testing & Validation

---

### 7.1 Data Reconciliation

Reconciliation answers: "Does the data in system A match the data in system B?" Used after ETL loads, migrations, and system integrations.

```sql
-- Level 1: Aggregate comparison — quick sanity check
SELECT 'SOURCE' AS system, COUNT(*) AS records, SUM(order_total) AS total_amount
FROM source_orders
UNION ALL
SELECT 'TARGET',           COUNT(*),             SUM(order_total)
FROM target_orders;
-- If counts and totals match: high confidence the data matches
-- If they differ: drill down to Level 2

-- Level 2: Which records are missing or extra?
SELECT 'SOURCE_ONLY' AS status, s.order_id, s.order_total
FROM source_orders s
WHERE NOT EXISTS (SELECT 1 FROM target_orders t WHERE t.order_id = s.order_id)
UNION ALL
SELECT 'TARGET_ONLY', t.order_id, t.order_total
FROM target_orders t
WHERE NOT EXISTS (SELECT 1 FROM source_orders s WHERE s.order_id = t.order_id);

-- Level 3: FULL OUTER JOIN to see matched + unmatched + changed in one result
SELECT
    COALESCE(s.order_id, t.order_id) AS order_id,
    CASE
        WHEN s.order_id    IS NULL THEN 'TARGET_ONLY'    -- exists in target, not in source
        WHEN t.order_id    IS NULL THEN 'SOURCE_ONLY'    -- exists in source, not in target
        WHEN s.order_total != t.order_total THEN 'AMOUNT_MISMATCH'   -- both exist but differ
        ELSE 'MATCH'
    END AS status,
    s.order_total AS source_amount,
    t.order_total AS target_amount
FROM source_orders s
FULL OUTER JOIN target_orders t ON s.order_id = t.order_id
WHERE s.order_id IS NULL OR t.order_id IS NULL OR s.order_total != t.order_total;
```

---

### 7.2 Checksums

A checksum is a single value derived from the entire dataset. Comparing checksums is O(1) — instant for any size table. If checksums match, the data almost certainly matches (hash collisions are extremely rare).

```sql
-- ORA_HASH: Oracle's built-in hash function, returns a 32-bit integer (0 to 2^32-1)
-- Hash a single row by concatenating all relevant columns with a separator
SELECT order_id,
       ORA_HASH(order_id || '|' || customer_id || '|' || order_total || '|' || status) AS row_hash
FROM orders;
-- The '|' separator prevents 'A'||'BC' from hashing the same as 'AB'||'C'

-- Compare table checksums
SELECT 'Source hash' AS description,
       ORA_HASH(LISTAGG(order_id || customer_id || order_total ORDER BY order_id)) AS table_hash
FROM source_orders
UNION ALL
SELECT 'Target hash',
       ORA_HASH(LISTAGG(order_id || customer_id || order_total ORDER BY order_id))
FROM target_orders;
-- If both hashes are the same: the tables are identical

-- Compare row-level hashes to find which specific rows differ
SELECT COALESCE(s.order_id, t.order_id) AS order_id,
       CASE WHEN s.row_hash IS NULL THEN 'MISSING_IN_SOURCE'
            WHEN t.row_hash IS NULL THEN 'MISSING_IN_TARGET'
            WHEN s.row_hash != t.row_hash THEN 'DATA_DIFFERS'
            ELSE 'MATCH' END AS status
FROM (SELECT order_id, ORA_HASH(order_id||'|'||customer_id||'|'||order_total) AS row_hash
      FROM source_orders) s
FULL JOIN
     (SELECT order_id, ORA_HASH(order_id||'|'||customer_id||'|'||order_total) AS row_hash
      FROM target_orders) t
ON s.order_id = t.order_id
WHERE s.row_hash IS NULL OR t.row_hash IS NULL OR s.row_hash != t.row_hash;
```

---

### 7.3 NULL Analysis

**Three-valued logic:** SQL uses TRUE, FALSE, and **UNKNOWN**. Any comparison with NULL (except `IS NULL`) yields UNKNOWN, not FALSE. UNKNOWN in a WHERE clause means the row is excluded. This catches many developers off-guard.

```sql
-- NULL = NULL → UNKNOWN (not TRUE!)
-- NULL != 5   → UNKNOWN (not TRUE!)
-- NULL > 5    → UNKNOWN
-- Only IS NULL and IS NOT NULL return TRUE/FALSE for NULL values

-- Count nulls per column (manual version)
SELECT
    COUNT(*) AS total_rows,
    COUNT(*) - COUNT(email)   AS email_nulls,    -- COUNT(col) skips NULLs; COUNT(*) doesn't
    COUNT(*) - COUNT(phone)   AS phone_nulls,
    COUNT(*) - COUNT(address) AS address_nulls,
    ROUND((COUNT(*) - COUNT(email)) * 100.0 / COUNT(*), 2) AS email_null_pct
FROM customers;

-- NULL pattern analysis: which combination of nulls is most common?
SELECT
    CASE WHEN email   IS NULL THEN 'N' ELSE 'Y' END ||
    CASE WHEN phone   IS NULL THEN 'N' ELSE 'Y' END ||
    CASE WHEN address IS NULL THEN 'N' ELSE 'Y' END AS null_pattern,
    -- e.g., 'YNY' = has email, no phone, has address
    COUNT(*)                                         AS count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS pct
FROM customers
GROUP BY
    CASE WHEN email   IS NULL THEN 'N' ELSE 'Y' END ||
    CASE WHEN phone   IS NULL THEN 'N' ELSE 'Y' END ||
    CASE WHEN address IS NULL THEN 'N' ELSE 'Y' END
ORDER BY count DESC;
```

---

### 7.4 Duplicate Detection

```sql
-- Basic duplicate check: which email values appear more than once?
SELECT email, COUNT(*) AS occurrences,
       LISTAGG(customer_id, ', ') WITHIN GROUP (ORDER BY customer_id) AS all_ids
FROM customers
WHERE email IS NOT NULL
GROUP BY email
HAVING COUNT(*) > 1;

-- ROW_NUMBER approach: label every row as 1st, 2nd, 3rd occurrence
-- rn=1 is the "keeper"; rn>1 are duplicates
SELECT * FROM (
    SELECT c.*,
           ROW_NUMBER() OVER (
               PARTITION BY email        -- group by the key that defines a duplicate
               ORDER BY customer_id      -- within the group, keep the one with lowest ID
           ) AS rn
    FROM customers c
)
WHERE rn > 1;   -- all duplicate rows except the first occurrence

-- SOUNDEX: phonetic matching — finds names that sound alike
-- SOUNDEX('Smith') = SOUNDEX('Smyth') = SOUNDEX('Smythe') — good for catching typos
SELECT c1.customer_id, c1.last_name, c2.customer_id AS match_id, c2.last_name AS match_name
FROM customers c1
JOIN customers c2 ON SOUNDEX(c1.last_name) = SOUNDEX(c2.last_name)
WHERE c1.customer_id < c2.customer_id   -- avoid self-join and duplicating pairs
  AND c1.last_name != c2.last_name;     -- exclude exact matches (already handled)

-- UTL_MATCH.JARO_WINKLER_SIMILARITY: returns 0-100 similarity score
-- Jaro-Winkler gives extra weight to matching prefixes (good for names)
-- Score > 85: likely same person; 70-85: review manually; < 70: probably different
SELECT c1.customer_id, c1.last_name, c2.customer_id AS match_id, c2.last_name,
       UTL_MATCH.JARO_WINKLER_SIMILARITY(c1.last_name, c2.last_name) AS similarity
FROM customers c1
JOIN customers c2 ON c1.customer_id < c2.customer_id
WHERE UTL_MATCH.JARO_WINKLER_SIMILARITY(c1.last_name, c2.last_name) > 85
ORDER BY similarity DESC;
```

---

### 7.5 Referential Integrity Checks

Use these when foreign key constraints are disabled (e.g., during bulk loads, in data warehouses) or when data crosses database boundaries where FK constraints can't be enforced.

```sql
-- Simple orphan check: orders with no matching customer
SELECT COUNT(*) AS orphan_count
FROM orders o
WHERE NOT EXISTS (
    SELECT 1 FROM customers c WHERE c.customer_id = o.customer_id
);

-- Multi-table orphan check: all FK relationships in one query
WITH all_violations AS (
    SELECT 'orders.customer_id → customers' AS relationship, o.order_id AS record_id
    FROM orders o
    WHERE NOT EXISTS (SELECT 1 FROM customers c WHERE c.customer_id = o.customer_id)

    UNION ALL

    SELECT 'order_items.order_id → orders', oi.item_id
    FROM order_items oi
    WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.order_id = oi.order_id)

    UNION ALL

    SELECT 'employees.department_id → departments', e.employee_id
    FROM employees e
    WHERE e.department_id IS NOT NULL
      AND NOT EXISTS (SELECT 1 FROM departments d WHERE d.department_id = e.department_id)
)
SELECT relationship, COUNT(*) AS violation_count
FROM all_violations
GROUP BY relationship
ORDER BY violation_count DESC;

-- Self-referencing integrity: catch broken manager chains
SELECT employee_id, manager_id, 'Manager does not exist' AS issue
FROM employees e
WHERE manager_id IS NOT NULL
  AND NOT EXISTS (SELECT 1 FROM employees m WHERE m.employee_id = e.manager_id);
```

---

### 7.6 Business Rule Validation

```sql
-- Attribute rule: salary must be within job's defined range
SELECT e.employee_id, e.job_id, e.salary, j.min_salary, j.max_salary,
       CASE WHEN e.salary < j.min_salary THEN 'Below minimum by ' || (j.min_salary - e.salary)
            WHEN e.salary > j.max_salary THEN 'Above maximum by ' || (e.salary - j.max_salary)
       END AS violation
FROM employees e JOIN jobs j ON e.job_id = j.job_id
WHERE e.salary NOT BETWEEN j.min_salary AND j.max_salary;

-- Intra-record rule: date sequence must be logical
SELECT order_id, order_date, ship_date, delivery_date,
       CASE WHEN ship_date     < order_date   THEN 'Ship before order date'
            WHEN delivery_date < ship_date    THEN 'Delivery before ship date'
            WHEN ship_date     > order_date + 30 THEN 'Ship > 30 days after order (unusual)'
            ELSE 'VALID'
       END AS date_check
FROM orders
WHERE ship_date < order_date
   OR delivery_date < ship_date;

-- Conditional rule: shipped orders must have a tracking number
SELECT order_id, status, tracking_number
FROM orders
WHERE status = 'SHIPPED' AND tracking_number IS NULL;

-- Inter-record rule: order header total must match sum of line items
SELECT o.order_id,
       o.order_total           AS stored_total,
       SUM(oi.qty * oi.price)  AS calculated_total,
       ABS(o.order_total - SUM(oi.qty * oi.price)) AS discrepancy
FROM orders o JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id, o.order_total
HAVING ABS(o.order_total - SUM(oi.qty * oi.price)) > 0.01;
-- The 0.01 tolerance handles floating-point rounding differences

-- Credit limit check: open order value must not exceed customer's credit limit
SELECT c.customer_id, c.customer_name, c.credit_limit,
       SUM(o.order_total) AS open_orders_total,
       SUM(o.order_total) - c.credit_limit AS over_by
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.status NOT IN ('CANCELLED', 'DELIVERED')   -- only count open/pending orders
GROUP BY c.customer_id, c.customer_name, c.credit_limit
HAVING SUM(o.order_total) > c.credit_limit;
```

---

### 7.7 Regression Testing

Regression testing verifies that a change (new release, data migration, schema change) hasn't broken anything that worked before.

```sql
-- Step 1: BEFORE the change — capture a baseline snapshot
CREATE TABLE snapshot_orders AS
SELECT order_id, customer_id, order_total, status,
       ORA_HASH(order_id || '|' || customer_id || '|' || order_total || '|' || status) AS row_hash,
       SYSDATE AS snapshot_date
FROM orders;
-- This preserves the current state so you can compare it after the change

-- Step 2: AFTER the change — compare live data to snapshot
SELECT COALESCE(s.order_id, o.order_id) AS order_id,
       CASE WHEN s.order_id IS NULL THEN 'NEW (added after snapshot)'
            WHEN o.order_id IS NULL THEN 'DELETED (removed after snapshot)'
            WHEN s.row_hash != ORA_HASH(o.order_id||'|'||o.customer_id||'|'||o.order_total||'|'||o.status)
                                   THEN 'MODIFIED'
            ELSE 'UNCHANGED'
       END AS change_type,
       s.order_total AS before_value,
       o.order_total AS after_value
FROM snapshot_orders s
FULL JOIN orders o ON s.order_id = o.order_id
WHERE s.order_id IS NULL
   OR o.order_id IS NULL
   OR s.row_hash != ORA_HASH(o.order_id||'|'||o.customer_id||'|'||o.order_total||'|'||o.status);

-- Aggregate baseline with variance threshold
-- Capture baseline
CREATE TABLE aggregate_baseline AS
SELECT 'ORDERS' AS tbl, 'COUNT' AS metric, COUNT(*) AS baseline_value FROM orders UNION ALL
SELECT 'ORDERS', 'SUM_TOTAL', SUM(order_total) FROM orders UNION ALL
SELECT 'CUSTOMERS', 'COUNT', COUNT(*) FROM customers;

-- Compare after change (flag anything that moved more than 1%)
SELECT b.tbl, b.metric, b.baseline_value AS before,
       c.current_value AS after,
       ROUND(ABS(c.current_value - b.baseline_value) / NULLIF(b.baseline_value, 0) * 100, 2) AS pct_change
FROM aggregate_baseline b
JOIN (
    SELECT 'ORDERS' AS tbl, 'COUNT' AS metric, COUNT(*) AS current_value FROM orders UNION ALL
    SELECT 'ORDERS', 'SUM_TOTAL', SUM(order_total) FROM orders UNION ALL
    SELECT 'CUSTOMERS', 'COUNT', COUNT(*) FROM customers
) c ON b.tbl = c.tbl AND b.metric = c.metric
WHERE ABS(c.current_value - b.baseline_value) / NULLIF(b.baseline_value, 0) > 0.01;
-- Any row appearing here needs investigation
```

---

## 8. Quick-Reference Summary

### Platform Decision Matrix

| Situation | Tool to reach for |
|-----------|-------------------|
| Oracle query slow on filtered column | B-Tree index on that column |
| Oracle filter uses a function (UPPER, TRUNC) | Function-based index matching the exact function call |
| DW table with gender/status/region columns | Bitmap index (never in OLTP) |
| Sequential inserts contend on same leaf node | Reverse Key index |
| Optimizer making bad choices despite good SQL | Gather stats with `DBMS_STATS`; check for stale histograms |
| Snowflake large table + date range filter | `CLUSTER BY (date_col)` — improves micro-partition pruning |
| Snowflake repeated exact same query | Result cache handles it automatically (24h) |
| Snowflake too slow, check why | Query Profile: look at Partitions Scanned vs Total, Spillage |
| Databricks many small files (after streaming) | `OPTIMIZE table` to compact |
| Databricks also filter by customer_id | Add `ZORDER BY (customer_id)` to the OPTIMIZE command |
| Databricks small table joined to big table | `/*+ BROADCAST(small) */` hint or raise autoBroadcastThreshold |
| Databricks one query partition getting much more data | Enable AQE skewJoin handling |

---

### Window Function Quick-Reference

| Goal | Syntax | Key behaviour |
|------|--------|--------------|
| Unique sequential number | `ROW_NUMBER() OVER (PARTITION BY g ORDER BY s)` | No ties — always unique |
| Rank with gaps after ties | `RANK() OVER (ORDER BY s DESC)` | 1,2,2,4 (skips 3) |
| Rank without gaps | `DENSE_RANK() OVER (ORDER BY s DESC)` | 1,2,2,3 |
| Percentile (0-100) | `ROUND(PERCENT_RANK() OVER (...) * 100, 1)` | 100 = highest |
| Equal-size buckets | `NTILE(4) OVER (ORDER BY s)` | Quartiles: 1=bottom, 4=top |
| Running total | `SUM(c) OVER (ORDER BY d ROWS UNBOUNDED PRECEDING AND CURRENT ROW)` | Accumulates |
| 3-row moving average | `AVG(c) OVER (ORDER BY d ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)` | Smooths noise |
| Previous row | `LAG(c, 1, 0) OVER (ORDER BY d)` | 3rd arg = default when no prior row |
| Next row | `LEAD(c, 1, 0) OVER (ORDER BY d)` | |
| YoY (same month last year) | `LAG(rev, 12) OVER (ORDER BY month)` | Offset = 12 months |
| First value in partition | `FIRST_VALUE(c) OVER (PARTITION BY g ORDER BY s ROWS UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)` | Full-frame required |
| Last value in partition | `LAST_VALUE(c) OVER (...ROWS UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)` | Full-frame REQUIRED |

---

### PL/SQL Quick-Reference Card

```sql
-- BLOCK
DECLARE vars; BEGIN stmts; EXCEPTION handlers; END;

-- ANCHOR TYPES
col%TYPE          -- match one column's type
table%ROWTYPE     -- match whole row structure

-- CURSOR FOR LOOP (simplest pattern)
FOR rec IN (SELECT col1, col2 FROM table WHERE ...) LOOP
    -- use rec.col1, rec.col2
END LOOP;

-- BULK COLLECT + FORALL (fast pattern)
SELECT col BULK COLLECT INTO v_collection FROM table;
FORALL i IN 1..v_collection.COUNT
    UPDATE table SET col = v_collection(i) WHERE id = v_ids(i);

-- EXCEPTION HANDLING
WHEN NO_DATA_FOUND THEN ...  -- SELECT INTO returned 0 rows
WHEN TOO_MANY_ROWS THEN ...  -- SELECT INTO returned > 1 row
WHEN DUP_VAL_ON_INDEX THEN ...  -- unique constraint violated
WHEN OTHERS THEN DBMS_OUTPUT.PUT_LINE(SQLERRM); RAISE;

-- RAISE CUSTOM ERROR
RAISE_APPLICATION_ERROR(-20001, 'Your message here');

-- PROCEDURE
CREATE PROCEDURE name(p_in IN type, p_out OUT type) IS BEGIN ... END;

-- FUNCTION
CREATE FUNCTION name(p IN type) RETURN return_type IS BEGIN RETURN val; END;

-- PACKAGE
CREATE PACKAGE name AS public_headers; END;
CREATE PACKAGE BODY name AS private_vars; implementations; BEGIN init; END;

-- TRIGGER
CREATE TRIGGER name BEFORE|AFTER INSERT|UPDATE|DELETE ON table
[FOR EACH ROW] BEGIN :NEW.col := ...; :OLD.col (read only); END;

-- COMPOUND TRIGGER
CREATE TRIGGER name FOR dml ON table COMPOUND TRIGGER
    AFTER EACH ROW IS BEGIN collect_data; END AFTER EACH ROW;
    AFTER STATEMENT IS BEGIN bulk_action; END AFTER STATEMENT;
END;

-- LOCKING
SELECT ... FOR UPDATE [NOWAIT | WAIT n | SKIP LOCKED];
UPDATE ... WHERE CURRENT OF cursor_name;

-- DYNAMIC SQL
EXECUTE IMMEDIATE 'sql' [INTO vars] [USING bind_vals] [RETURNING INTO vars];
OPEN cursor FOR 'sql' USING bind_vals;

-- TRANSACTION
SAVEPOINT name; ROLLBACK TO name; COMMIT;
PRAGMA AUTONOMOUS_TRANSACTION;  -- independent commit/rollback
```

---

### Common Oracle Error Codes

| ORA Code | Exception Name | Cause |
|----------|---------------|-------|
| ORA-00001 | `DUP_VAL_ON_INDEX` | Unique constraint violated |
| ORA-00054 | *(no name)* | Row locked and NOWAIT was specified |
| ORA-00060 | *(no name)* | Deadlock detected |
| ORA-01403 | `NO_DATA_FOUND` | SELECT INTO returned zero rows |
| ORA-01422 | `TOO_MANY_ROWS` | SELECT INTO returned more than one row |
| ORA-01476 | `ZERO_DIVIDE` | Division by zero |
| ORA-01555 | *(no name)* | Snapshot too old (undo data overwritten) |
| ORA-02291 | *(no name)* | FK violation: parent key not found |
| ORA-02292 | *(no name)* | FK violation: child records exist |
| ORA-06502 | `VALUE_ERROR` | Type conversion or assignment error |
| ORA-08177 | *(no name)* | Serializable transaction conflict |
| ORA-24381 | *(no name)* | Error in FORALL bulk DML |
| ORA-30006 | *(no name)* | FOR UPDATE WAIT n timeout expired |


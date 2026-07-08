# Snowflake Table Types & View Types — Complete Guide

*A simple, detailed reference built only from official Snowflake documentation (docs.snowflake.com)*

---

## Table of Contents

**Part 1 — Table Types**
1. [Permanent Tables](#1-permanent-tables)
2. [Temporary Tables](#2-temporary-tables)
3. [Transient Tables](#3-transient-tables)
4. [Apache Iceberg™ Tables](#4-apache-iceberg-tables)
5. [External Tables](#5-external-tables)
6. [Dynamic Tables](#6-dynamic-tables)
7. [Table Types — Side-by-Side Comparison](#7-table-types--side-by-side-comparison)

**Part 2 — View Types**
8. [Standard (Non-Materialized) Views](#8-standard-non-materialized-views)
9. [Materialized Views](#9-materialized-views)
10. [Secure Views](#10-secure-views)
11. [View Types — Side-by-Side Comparison](#11-view-types--side-by-side-comparison)

**[Quick Cheat Sheet](#quick-cheat-sheet)**
**[Sources](#sources)**

---

## Two ideas you need before anything else

Snowflake protects your data with two background safety nets. Almost everything about table types comes down to *how much of these two features a table gets*:

- **Time Travel** — lets you look at, query, or restore older versions of a table (or an already-dropped table) for a set number of days. You choose the retention period (0 up to 90 days, depending on edition).
- **Fail-safe** — a *non-configurable* 7-day emergency recovery window that only Snowflake itself can use to restore data after Time Travel has expired, in the event of a system failure. You cannot access Fail-safe data yourself — you'd need to contact Snowflake Support.

Every table type below either has both, one, or neither of these — and that's the main thing that separates them.

---

## Part 1 — Table Types

## 1. Permanent Tables

### What it is
This is the **default** table type. If you run a plain `CREATE TABLE` statement without saying `TEMPORARY` or `TRANSIENT`, Snowflake makes a permanent table.

### Why it exists
Most business data — customer records, orders, financial history — needs to survive forever (until you choose to delete it) *and* needs strong protection against accidental loss. Permanent tables are Snowflake's answer to "keep this safe, no matter what."

### Key characteristics
- **Persistence:** Exists until you explicitly drop it.
- **Time Travel:**
  - Standard Edition — 0 or 1 day (default is 1).
  - Enterprise Edition and higher — 0 to 90 days, and you can configure the default.
- **Fail-safe:** Always 7 days, and it cannot be turned off or shortened.
- **Storage cost:** Full storage charges, *plus* Fail-safe storage charges once data leaves the Time Travel window.
- **Cloning:** A permanent table can be cloned into a permanent, transient, or temporary table.

### Real-world example
A retail company's `customer_orders` table — this data must never silently disappear, so it's kept as a permanent table with disaster-recovery protection built in.

### Simple syntax
```sql
CREATE TABLE table_name (
  column1 datatype,
  column2 datatype
);
```

---

## 2. Temporary Tables

### What it is
A table that exists **only for the session that created it**. The moment that session ends, the table and all its data are gone — permanently, and unrecoverable even by Snowflake.

### Why it exists
A lot of work is throwaway: staging data mid-ETL, holding intermediate query results, or testing something without touching real tables. Temporary tables give you a scratchpad that cleans up after itself automatically, so you never have to remember to delete anything.

### Key characteristics
- **Visibility:** Only visible in the session that created it — no other user or session can see it, even with full privileges.
- **Time Travel:** 0 or 1 day (default 1) — but in practice it's capped by however long the session lasts, whichever is shorter.
- **Fail-safe:** None.
- **Creating one doesn't require the `CREATE TABLE` privilege** on the schema.
- **Procedure-scoped temp tables:** Inside a Snowflake Scripting stored procedure, you can create a temp table that lives only for that one procedure call (not the whole session), using `CREATE OR REPLACE PROCEDURE SCOPED TEMP TABLE`.
- **Cannot be converted** to a permanent or transient table after creation.
- **Storage:** While it exists, it still counts toward your storage bill — so for long sessions (over 24 hours) with large temp tables, Snowflake recommends dropping them explicitly rather than waiting for the session to end.

### An important quirk: naming conflicts
A temporary table can have the **exact same name** as a permanent table in the same schema. When that happens:
- The temporary table "wins" for the rest of that session — it hides the real table.
- All your queries in that session hit the temp table, not the real one.
- This gets tricky with `CREATE OR REPLACE` (which drops and recreates) and with Time Travel restores — always double-check which table you're actually working with.

### Real-world example
A data engineer loading a CSV runs some cleanup and deduplication logic in a `stg_customers_temp` table before writing final results to a permanent table. Once the ETL session ends, the scratch table vanishes with no cleanup effort needed.

### Simple syntax
```sql
CREATE TEMPORARY TABLE mytemptable (id NUMBER, creation_date DATE);
-- TEMP is a valid abbreviation for TEMPORARY
```

---

## 3. Transient Tables

### What it is
Think of it as the **middle ground** between permanent and temporary. A transient table persists like a permanent table (it doesn't disappear when your session ends) but — like a temporary table — it skips Fail-safe entirely.

### Why it exists
Sometimes you need data to outlive a single session (so other users, or tomorrow's job, can see it) but it isn't precious enough to justify paying for 7 days of Fail-safe protection. Transient tables let you get long-term storage minus the disaster-recovery overhead — and minus the cost that comes with it.

### Key characteristics
- **Persistence:** Exists until explicitly dropped; visible to any user with the right privileges (not session-locked like temp tables).
- **Time Travel:** 0 or 1 day (default is 1).
- **Fail-safe:** None — this is the key difference from permanent tables.
- **Storage cost:** Regular storage charges apply, but there are **no Fail-safe storage charges**, since there's no Fail-safe.
- **Transient databases/schemas:** You can mark an entire database or schema as transient — everything created inside automatically becomes transient too.
- **Cannot be converted** to another table type after creation.

### Cloning nuance worth knowing
If you clone a *permanent* table into a *transient* table, Snowflake creates a zero-copy clone — no extra storage is used at first because it shares the original's micro-partitions. But this can actually *delay* when the original permanent table's data enters Fail-safe after it's dropped: shared bytes only enter Fail-safe once the transient clone that still references them is also dropped.

### The trade-off to remember
Because there's no Fail-safe, if a transient table is ever lost due to a system failure, the data is **not recoverable** by you or by Snowflake once the (short) Time Travel window passes. Only use transient tables for data you could reproduce elsewhere if something went wrong.

### Real-world example
An intermediate table in a multi-step transformation pipeline — needed by several downstream jobs over the next few days, but easily rebuilt from source data if lost, so full disaster-recovery isn't worth paying for.

### Simple syntax
```sql
CREATE TRANSIENT TABLE mytranstable (id NUMBER, creation_date DATE);
CREATE TRANSIENT DATABASE my_transient_db;
CREATE TRANSIENT SCHEMA my_transient_db.my_transient_schema;
```

---

## 4. Apache Iceberg™ Tables

### What it is
Apache Iceberg tables let Snowflake query and manage data that physically lives in **your own external cloud storage** (Amazon S3, Google Cloud Storage, or Azure Storage) — using the open-source **Apache Iceberg** table format — while still giving you most of the speed and SQL experience of a normal Snowflake table.

### Why it exists
Many organizations already keep enormous data lakes in open formats, and don't want to duplicate all of that data inside Snowflake's own storage just to query it. Iceberg tables let Snowflake sit "on top of" that existing data lake, so multiple engines (Snowflake, Spark, Trino, Databricks, and others) can all read and write the *same* files using an open, shared standard.

### Key concepts

**File format:** Iceberg tables use the **Apache Parquet** file format, and the Iceberg specification adds ACID transactions, schema evolution, hidden partitioning, and table snapshots on top of those files.

**External volume:** A named, account-level Snowflake object that securely connects Snowflake to your cloud storage location, so Snowflake can read/write table data and Iceberg metadata files.

**Catalog:** The catalog tracks the "current" metadata file for a table and updates that pointer atomically whenever the table changes. Snowflake gives you two options:

| Option | What it means |
|---|---|
| **Snowflake as the catalog** | Full Snowflake platform support, read *and* write access. Data can sit in your own external storage, or (in newer previews) in Snowflake-managed storage. |
| **External catalog** | Snowflake connects to a catalog you already manage (for example AWS Glue, Snowflake Open Catalog, Databricks Unity Catalog, or a generic Iceberg REST catalog). Support is more limited, but this keeps you fully interoperable with your existing Iceberg ecosystem. |

### Key characteristics
- **Fail-safe:** Not provided — you are responsible for your external storage's own backup/recovery.
- **Storage cost inside Snowflake:** None, if you use your own external volume (your cloud provider bills you directly). If you use Snowflake-managed storage for an Iceberg table, Snowflake does bill for that storage.
- **Billing:** Snowflake charges for compute (virtual warehouses) and cloud services used to query/manage Iceberg tables, plus possible cross-cloud/cross-region data transfer fees.
- **Maintenance:** For Snowflake-managed Iceberg tables, Snowflake automatically handles compaction and lifecycle maintenance (though you can disable compaction). For externally managed tables, you're responsible for maintenance using your own Iceberg engine.
- **Dynamic Iceberg tables:** You can also build a *dynamic table* (see below) whose data source or output lives in the Iceberg format — combining automated pipeline refresh with data-lake storage.

### Notable limitations (subject to change)
- Fail-safe, hybrid tables, and Snowflake's native encryption aren't available for Iceberg tables.
- Temporary and transient Iceberg tables aren't supported.
- Cloning, clustering, and standard/append-only streams have restrictions on *externally managed* Iceberg tables (insert-only streams are supported).
- Third-party clients can't append, delete, or upsert data into Iceberg tables that use Snowflake as the catalog.

### Real-world example
A company already stores petabytes of clickstream data in S3 in Iceberg format, queried by Spark and Trino. Snowflake can create an Iceberg table pointing at that same data (via an external catalog) so their analytics team can query it directly with SQL — without copying a single byte into Snowflake.

### Simple syntax
```sql
CREATE ICEBERG TABLE my_iceberg_table (
  id INT,
  name STRING
)
EXTERNAL_VOLUME = 'my_external_volume'
CATALOG = 'SNOWFLAKE'
BASE_LOCATION = 'my_iceberg_table/';
```

---

## 5. External Tables

### What it is
An external table lets you **query files sitting in an external stage** (S3, Azure, or GCS) as though they were a normal Snowflake table — without loading the data into Snowflake at all.

### Why it exists
Sometimes you just want to explore or occasionally query files in a data lake without going through a full data-loading pipeline. External tables give you SQL access to raw files (CSV, JSON, Parquet, Avro, ORC, and more — any format `COPY INTO` supports, except XML) with minimal setup.

### Key characteristics
- **Read-only:** No `INSERT`, `UPDATE`, or `DELETE`. You *can* query it, join it with other tables, and build views on top of it.
- **Schema on read:** Every external table automatically includes:
  - `VALUE` — a VARIANT column holding the whole row.
  - `METADATA$FILENAME` — which staged file the row came from.
  - `METADATA$FILE_ROW_NUMBER` — the row's position in that file.
  - You can optionally define **virtual columns** on top of `VALUE` if you already know the file's structure.
- **Partitioning:** Strongly recommended for performance. Partitions can be added automatically (based on filename/path expressions) or manually (useful for syncing with external metastores like AWS Glue or Hive).
- **Refreshing metadata:** External tables don't automatically know about new files. You refresh them either manually (`ALTER EXTERNAL TABLE … REFRESH`) or automatically via cloud storage event notifications (S3 events, Azure Event Grid, GCS Pub/Sub).
- **Performance:** Generally slower than native Snowflake tables, since data is read live from external storage on every query. You can speed this up with a **materialized view** over the external table (Enterprise Edition), or by switching to an **Iceberg table** for Parquet-based workloads.
- **Billing:** Small cloud-services overhead for automatic refresh (shown as Snowpipe charges) and for manual refresh operations.

### Real-world example
A marketing team occasionally needs to check raw event logs sitting in an S3 bucket that a separate system owns. Rather than building a full ingestion pipeline, they create an external table pointing at that bucket so analysts can just run SQL against it directly.

### Simple syntax
```sql
CREATE EXTERNAL TABLE my_ext_table
  WITH LOCATION = @my_stage/
  FILE_FORMAT = (TYPE = CSV SKIP_HEADER = 1)
  PATTERN = '.*data.*[.]csv';
```

---

## 6. Dynamic Tables

### What it is
A dynamic table **materializes the result of a SQL `SELECT` query and keeps it up to date automatically.** You write the query and specify how fresh you need the data to be — Snowflake figures out the rest: what changed, when to refresh, and in what order.

### Why it exists
Traditionally, keeping a derived table up to date meant building a **stream** (to detect changes) plus a **task** (to run scheduled `MERGE` logic) — real orchestration code that someone has to write and maintain. Dynamic tables replace all of that with one declarative SQL statement: *"here's the query, here's how fresh I need it — you handle the rest."*

### Key concepts

**Target lag** — the maximum acceptable delay between a change in your source data and that change showing up in the dynamic table. For example, `TARGET_LAG = '10 minutes'` tells Snowflake to try to keep the table no more than 10 minutes stale (though actual lag can exceed this if refreshes take longer than expected). The minimum possible target lag is **60 seconds**. You can also set `TARGET_LAG = DOWNSTREAM` on an intermediate table in a pipeline, so it only refreshes when the tables *depending on it* need fresh data.

**Refresh modes:**
| Mode | What it does |
|---|---|
| `INCREMENTAL` | Only reprocesses the rows that changed since the last refresh. |
| `FULL` | Recomputes the entire result set every time. |
| `AUTO` | Snowflake decides at creation time, based on whether the query supports incremental refresh. |
| `ADAPTIVE` *(preview)* | Uses incremental refresh by default but automatically falls back to a full reinitialize when it detects large upstream changes. |
| `CUSTOM_INCREMENTAL` *(preview)* | Lets you define your own refresh logic with DML statements, for advanced cases. |

**Pipelines:** Dynamic tables can read from *other* dynamic tables, and Snowflake automatically figures out the dependency graph (a DAG) from your queries — no manual scheduling or dependency declarations needed. Refreshes run in the correct order so downstream tables always see a consistent snapshot of their inputs.

**Atomic refresh:** New results are applied all at once — readers never see a half-finished refresh.

### Costs
Dynamic tables have three cost categories:
1. **Warehouse compute** — running the refresh query itself.
2. **Cloud Services** — compiling the query, tracking dependencies, monitoring for changes, coordinating refresh scheduling.
3. **Storage** — each refresh can add, replace, or remove micro-partitions.

### When to use a dynamic table
- Your transformation logic can be written as a single SQL `SELECT`.
- You want a multi-step pipeline (joins, aggregations, window functions) without hand-writing orchestration.
- You want to move from batch to near-real-time freshness just by tweaking `TARGET_LAG`.

### When *not* to use one
- You need data fresher than 60 seconds, or strictly guaranteed refresh timing.
- Your logic needs stored procedures or external function calls.
- You need direct row-level modification of the output (dynamic tables are **read-only** — no `INSERT`, `UPDATE`, `DELETE`, or `TRUNCATE`).

### Real-world example
An e-commerce company builds `dt_orders` (cleans raw order data) and, on top of it, `dt_orders_daily` (aggregates daily revenue by joining to a customer dimension table). Snowflake treats these as a two-stage pipeline, automatically refreshing `dt_orders` first and `dt_orders_daily` right after, keeping both within their target lag.

### Simple syntax
```sql
CREATE OR REPLACE DYNAMIC TABLE dt_orders
  TARGET_LAG = '10 minutes'
  WAREHOUSE = transform_wh
  REFRESH_MODE = INCREMENTAL
AS
  SELECT order_id, customer_id, quantity * unit_price AS line_total
  FROM raw_orders
  WHERE order_status != 'returned';
```

---

## 7. Table Types — Side-by-Side Comparison

| Table Type | Persistence | Time Travel | Fail-safe | DML Allowed | Data Location | Best For |
|---|---|---|---|---|---|---|
| **Permanent** | Until dropped | 0–1 day (Standard) / 0–90 days (Enterprise+) | 7 days | Yes | Snowflake storage | Long-term, mission-critical data |
| **Temporary** | End of session only | 0–1 day (capped by session) | None | Yes | Snowflake storage | Session-only scratch data |
| **Transient** | Until dropped | 0–1 day | None | Yes | Snowflake storage | Multi-session data that's reproducible elsewhere |
| **Apache Iceberg** | Until dropped | Snapshot-based (varies) | Not provided by Snowflake | Yes (Snowflake-managed catalog) | Your external cloud storage (or Snowflake storage, in preview) | Big data lakes shared across multiple engines |
| **External** | Until dropped | N/A | N/A | No (read-only) | Your external cloud storage | Querying files without loading them |
| **Dynamic** | Until dropped | Inherits from underlying storage | N/A | No (read-only; refreshed automatically) | Snowflake storage | Automated, declarative data pipelines |

---

## Part 2 — View Types

## 8. Standard (Non-Materialized) Views

### What it is
A view lets the result of a query be accessed **as if it were a table**. The query itself is stored in the `CREATE VIEW` statement — but the view doesn't store any data of its own. Every time you query it, Snowflake runs the underlying query fresh.

This is the most common type of view, and when people say "view" without qualification, this is usually what they mean.

### Why it exists
Two big reasons: **modularity** and **security**.

- **Modularity:** Instead of writing one giant, hard-to-read query, you can break logic into smaller views and build on top of them — easier to understand, easier to debug one piece at a time, and reusable across many queries.
- **Security / data segregation:** You can grant a role access to a view *without* granting access to the underlying table. For example, medical staff can see diagnosis and treatment data through a `doctor_view`, while billing staff see cost data through a separate `accountant_view` — neither role needs direct privileges on the raw patient table.

### Key characteristics
- Any valid query expression can define a view — selecting specific columns, filtering rows, joining multiple tables, etc.
- Views can be used almost anywhere a table can: joins, subqueries, and so on.
- **Recursive views:** A non-materialized view can refer to itself (similar to a recursive CTE) — useful for things like organizational hierarchies.
- A view can reference another view, letting you build layered "views of views."

### Access rule worth remembering
- If a user has access to a view but **not** to its underlying table, they can still query the view — *as long as the view's owner role* has access to the underlying table.
- If the user has access to **both** the view and the table, they can query the view regardless of what the owner role can access.

### Limitations
- You **cannot** alter a view's definition — to change it, you must `CREATE OR REPLACE` it.
- Views are read-only for direct DML, but you *can* use a view inside a subquery of a DML statement that updates the underlying base table (e.g., `DELETE FROM base_table WHERE cost > (SELECT AVG(cost) FROM some_view)`).
- Changes to the base table (like dropping a column) are **not** automatically reflected in the view — this can silently make the view invalid.

### Real-world example
A hospital creates `doctor_view` (diagnosis + treatment only) and `accountant_view` (billing only) from one shared `hospital_table`, so each department only ever sees the columns relevant to their job.

### Simple syntax
```sql
CREATE VIEW doctor_view AS
  SELECT patient_id, patient_name, diagnosis, treatment
  FROM hospital_table;
```

---

## 9. Materialized Views

### What it is
A materialized view looks like a view but **behaves more like a table**: its results are actually *computed once and stored*, rather than recalculated on every query. Because the data is pre-computed, querying it is faster than running the same query against the base table — sometimes dramatically faster.

> **Edition note:** Materialized views require **Enterprise Edition** or higher.

### Why it exists
Some queries are expensive and run over and over on data that barely changes — repeated aggregations, semi-structured data parsing, or queries against external tables. Recomputing that from scratch every single time wastes both time and compute credits. A materialized view calculates the answer once and Snowflake automatically keeps that stored answer in sync as the base table changes — you get speed without giving up correctness.

### When to create a materialized view (vs. a regular view)

| Use a **materialized view** when… | Use a **regular view** when… |
|---|---|
| The query results don't change often | The results change often |
| The results are queried often | The results aren't used very often |
| The underlying query is expensive to run | The query is cheap to re-run |

Materialized views are also a strong fit when the result set is much smaller than the base table, when heavy processing is involved (semi-structured data parsing, slow aggregates), or when the base table is an external table (which is naturally slower to scan).

### Key characteristics
- **Always current:** Even if a background refresh hasn't finished yet, a query against the materialized view returns correct, up-to-date results — Snowflake fills in any gap from the base table automatically.
- **Automatic maintenance:** A background Snowflake service updates the materialized view whenever the base table changes — you never write refresh logic yourself.
- **Automatic query rewriting:** You don't even have to reference the materialized view directly. Snowflake's optimizer can silently rewrite a query against the *base table* to use the materialized view instead, if it produces the same result faster.
- **Supports clustering:** Unlike regular views, materialized views can have their own clustering key — you can even create multiple materialized views on the same base table, each clustered differently for different query patterns.
- **Can be secure:** Materialized views can also be defined with the `SECURE` keyword (see Secure Views below).

### Costs (this matters!)
- **Storage:** Each materialized view stores its own copy of query results.
- **Compute:** Automatic background maintenance consumes credits, billed under a Snowflake-managed warehouse called **MATERIALIZED_VIEW_MAINTENANCE**. Costs scale with how much data changes in the base table and how often.
- Clustering a materialized view adds further maintenance cost.
- **Recommendation:** Start with just a few materialized views on carefully chosen tables, and monitor cost before expanding.

### Important limitations
- **A materialized view can only query a single table** — no joins, including self-joins.
- It **cannot** query another materialized view, a regular view, a hybrid table, a dynamic table, or a UDTF.
- **Not allowed in the definition:** user-defined functions, window functions, `HAVING`, `ORDER BY`, `LIMIT`, `GROUP BY GROUPING SETS`/`ROLLUP`/`CUBE`, subquery nesting, or `MINUS`/`EXCEPT`/`INTERSECT`.
- Only a limited set of aggregate functions is supported (e.g. `SUM`, `COUNT`, `MIN`, `MAX`, `AVG`, `APPROX_COUNT_DISTINCT`) — and they can't be nested or combined with `DISTINCT`.
- Functions must be **deterministic** — you can't use something like `CURRENT_TIMESTAMP` in the definition.
- You cannot query a materialized view's history with **Time Travel** (though you *can* clone a schema/database containing one at a past point in time).
- No standard DML (`INSERT`/`UPDATE`/`DELETE`/`MERGE`/`COPY`) and no `TRUNCATE` directly on a materialized view.

### What happens if the base table changes
| Change to base table | Effect on materialized view |
|---|---|
| New column added | Not automatically added to the view (even if defined with `SELECT *`) |
| Existing column changed or dropped | The materialized view is **suspended** — must be recreated |
| Table renamed/swapped | View is suspended — usually must be recreated |
| Table dropped | View is suspended (not auto-dropped) — usually must be dropped |

### Real-world example
A pharmacy chain keeps a materialized view listing only the drug-interaction rules relevant to medicines they actually stock (a small subset of a huge national drug database). Since the underlying reference data changes rarely but is queried on every prescription, materializing it avoids scanning the entire national database each time.

### Simple syntax
```sql
CREATE MATERIALIZED VIEW mv_inventory AS
  SELECT product_id, wholesale_price
  FROM inventory;
```

---

## 10. Secure Views

### What it is
A secure view is a standard or materialized view created with the extra `SECURE` keyword. It adds **two protections**: the view's *definition* (its SQL text) is hidden from unauthorized users, and Snowflake disables certain internal query optimizations that could otherwise leak information about hidden rows.

### Why it exists
Two subtle problems can occur with a regular ("non-secure") view:

1. **The view's SQL definition is visible** to other users through commands like `SHOW VIEWS`, `GET_DDL`, or the Information Schema — even if they can't see the underlying data, they can potentially see *how* the view filters or transforms it, which can itself be sensitive.
2. **Internal optimizations can indirectly expose hidden data.** Snowflake's query optimizer might reorder your `WHERE` clause with the view's own filtering logic to run faster — and in rare cases, this reordering can let a clever query "leak" information about rows the user isn't supposed to see (for example, via a deliberate division-by-zero error that only triggers if a hidden row exists). Secure views turn off these optimizations to close that gap.

### When to use one
Use a secure view specifically for **data privacy** — restricting sensitive data from users who shouldn't see it. Don't use `SECURE` just for query convenience or code organization (that's what a regular view is for) — secure views can run **more slowly** than standard views because they skip certain optimizations.

### Key characteristics
- Created by adding `SECURE` to `CREATE VIEW` or `CREATE MATERIALIZED VIEW`.
- Can convert an existing view to secure (or back) using `ALTER VIEW`/`ALTER MATERIALIZED VIEW`.
- **Definition hidden:** Only users with the role that owns the view can see its definition via `SHOW VIEWS`, `GET_DDL`, or `INFORMATION_SCHEMA.VIEWS`. (Roles like `ACCOUNTADMIN` or `SNOWFLAKE.OBJECT_VIEWER` can still see it via the Account Usage `VIEWS` view for governance purposes.)
- **Query Profile is also hidden** for secure views — even the view's *owner* can't see its internal query plan in Query Profile, since other users might have access to view that profile.
- **Scanned data amount is hidden** — for queries using a secure view, Snowflake doesn't reveal how many bytes or micro-partitions were scanned, to avoid indirectly hinting at how much "hidden" data exists.
- Works well with `CURRENT_ROLE()` / `CURRENT_USER()` to build row-level access rules — e.g., a `WHERE` clause that only returns rows tied to the querying user's role.
- For views shared across Snowflake accounts (Secure Data Sharing), use `CURRENT_ACCOUNT()` instead — `CURRENT_ROLE()`/`CURRENT_USER()` return `NULL` in that context, since the data owner doesn't control the roles/users on the receiving account.

### Best practices to avoid accidental leaks
- **Don't expose sequence-generated ID columns** in a secure view if row counts are sensitive — a user could infer how many total rows exist between two IDs they *can* see. Use randomized identifiers (like `UUID_STRING()`) instead if this matters.
- For extremely high-security situations, even the approximate query timing of a secure view can hint at how much underlying data exists. In such cases, consider materializing separate physical tables per role instead of relying on a shared view.

### Real-world example
A company shares a `widgets_view` with different departments, each of which can only see products matching their assigned category (enforced via `CURRENT_ROLE()` in the `WHERE` clause). Making this view secure prevents a department from crafting a clever query (like a deliberate division-by-zero trick) to detect whether "hidden" categories exist.

### Simple syntax
```sql
CREATE OR REPLACE SECURE VIEW widgets_view AS
  SELECT w.*
  FROM widgets AS w
  WHERE w.id IN (
    SELECT widget_id FROM widget_access_rules
    WHERE UPPER(role_name) = CURRENT_ROLE()
  );
```

---

## 11. View Types — Side-by-Side Comparison

| View Type | Stores Data? | Performance | Extra Cost | Best For |
|---|---|---|---|---|
| **Standard (non-materialized)** | No — runs the query live each time | Slower than materialized, no caching | None beyond the query itself | Modular SQL, restricting access to a subset of columns/rows |
| **Materialized** | Yes — pre-computed and stored | Fast — like querying a table | Storage + background maintenance credits (Enterprise Edition+) | Expensive, frequently-run queries on slowly-changing data |
| **Secure** *(a modifier, not a separate type — can apply to standard or materialized)* | Depends on base type | Can be slightly slower (fewer optimizations) | None beyond base type's cost | Protecting sensitive data and hiding the view's definition/logic |

---

## Quick Cheat Sheet

**Choosing a table type:**
- Need it forever, with full disaster recovery? → **Permanent**
- Only need it for this one session/script? → **Temporary**
- Need it beyond the session, but don't need Fail-safe? → **Transient**
- Data lives in your own cloud storage / needs multi-engine access (Spark, Databricks, etc.)? → **Apache Iceberg**
- Just want to query files in cloud storage without loading them? → **External**
- Want an auto-refreshing, declarative transformation pipeline? → **Dynamic**

**Choosing a view type:**
- Just want to simplify or reuse a query, or restrict which columns/rows a role can see? → **Standard View**
- The same expensive query runs over and over on data that rarely changes? → **Materialized View**
- Need to hide the view's definition and close off any indirect data-leak risk? → Add **SECURE** to either type above

---

## Sources

All information in this guide was drawn directly from official Snowflake documentation:

- [Working with Temporary and Transient Tables](https://docs.snowflake.com/en/user-guide/tables-temp-transient)
- [CREATE TABLE](https://docs.snowflake.com/en/sql-reference/sql/create-table)
- [Apache Iceberg™ Tables](https://docs.snowflake.com/en/user-guide/tables-iceberg)
- [Introduction to External Tables](https://docs.snowflake.com/en/user-guide/tables-external-intro)
- [Dynamic Tables](https://docs.snowflake.com/en/user-guide/dynamic-tables/overview)
- [Overview of Views](https://docs.snowflake.com/en/user-guide/views-introduction)
- [Working with Materialized Views](https://docs.snowflake.com/en/user-guide/views-materialized)
- [Working with Secure Views](https://docs.snowflake.com/en/user-guide/views-secure)
- [Databases, Tables and Views - Overview](https://docs.snowflake.com/en/guides-overview-db)
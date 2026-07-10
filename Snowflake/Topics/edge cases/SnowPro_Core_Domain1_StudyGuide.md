# SnowPro Core – Domain 1 Study Guide
### Snowflake Architecture & Core Concepts (with Edge Cases, Gotchas & Exam Traps)

> Sourced and cross-checked against official Snowflake Documentation (docs.snowflake.com). Community/blog sources were used only to find candidate edge cases, then verified against the docs before inclusion.

---

## 1.1 Snowflake Architecture

### The Three Layers

Snowflake uses a **hybrid of shared-disk and shared-nothing architecture**, built on three independently scaling layers:

| Layer | What it does | Scales |
|---|---|---|
| **Database Storage** | Stores all table data as compressed, columnar **micro-partitions** in cloud object storage (S3/Blob/GCS) | Independently, elastic |
| **Compute (Query Processing)** | Virtual warehouses — MPP clusters that execute queries/DML | Independently, elastic |
| **Cloud Services** | Authentication, metadata, query parsing/optimization, access control, infrastructure management | Snowflake-managed, shared across the account |

**Why hybrid?**
- **Shared-disk-like**: all compute nodes see one central, shared copy of data — no data silos.
- **Shared-nothing-like**: each virtual warehouse is an independent MPP cluster with its own dedicated compute nodes, so warehouses don't compete with each other for compute.

### Database Storage Layer — Key Facts
- Data is reorganized on load into Snowflake's proprietary, compressed, **columnar** micro-partition format.
- Snowflake **manages everything** about physical storage: file size, structure, compression, metadata, statistics. You cannot access the raw files directly.
- Storage cost is based on **compressed** size.
- This layer enables: **zero-copy cloning**, **Time Travel**, and **Secure Data Sharing** — all are side effects of immutable, metadata-referenced storage rather than separate bolt-on features.

### Compute Layer — Key Facts
- A **virtual warehouse** is a cluster of compute resources (an MPP cluster).
- Each running warehouse is isolated — queries on one warehouse never affect performance on another (important exam point: **isolation guarantees consistent performance**).
- Warehouses read shared storage but do not "own" any data.
- Billed **per-second** (60-second minimum) based on warehouse size and number of clusters running.

### Cloud Services Layer — Key Facts
Services managed here include:
- Authentication & Access control (RBAC)
- Metadata management (table stats, min/max values for pruning, query history)
- Query parsing and **optimization** (this is where pruning decisions are made — *before* compute is touched)
- Infrastructure management

**⚠️ Exam trap — Cloud Services billing:**
- Cloud Services usage is billed **only when daily consumption exceeds 10% of that day's warehouse (compute) credit usage**. Below that threshold, it's effectively free.
- This is a very common trick question: "Is the Cloud Services layer always free?" → **No**, only up to the 10% adjustment.

### How Pruning Works (ties storage + cloud services together)
1. Cloud Services layer holds a **global metadata cache** (min/max values, null counts, distinct counts per micro-partition).
2. At query compile time (in Cloud Services, **before any warehouse touches data**), Snowflake determines which micro-partitions can be skipped.
3. Only necessary micro-partitions are scanned by compute — this is **partition pruning**.
4. Pruning happens in two stages: prune micro-partitions first (horizontal), then prune columns within remaining partitions (vertical, due to columnar storage).

### Edge Cases & Gotchas
- Snowflake **cannot run on private/on-premises infrastructure** — it only runs on public cloud infrastructure (AWS, Azure, GCP).
- An individual **account runs in a single region** on a single cloud platform; you can have multiple accounts across regions/clouds.
- Because compute and storage are decoupled, you can spin up **many warehouses of different sizes reading the same data simultaneously** with no contention — a classic scenario question ("how do I let both the BI team and ETL team query the same tables without one blocking the other?" → separate warehouses on shared storage).

### Exam Traps
- Don't confuse "shared-disk" (storage) with "shared-nothing" (compute) — Snowflake is **both**, at different layers.
- The three layers are **Database Storage, Compute (Query Processing), Cloud Services** — memorize this exact split; some vendor material calls compute "Query Processing Layer," same thing.
- Zero-copy cloning is a **metadata operation** — it does not duplicate physical data initially; new micro-partitions are only created once the clone diverges (DML on the clone).

---

## Comparing Snowflake Editions

| Feature/Focus | Standard | Enterprise | Business Critical | Virtual Private Snowflake (VPS) |
|---|---|---|---|---|
| Core relational DB features | ✅ | ✅ | ✅ | ✅ |
| Multi-cluster warehouses | ❌ | ✅ | ✅ | ✅ |
| Materialized views | ❌ | ✅ | ✅ | ✅ |
| Search Optimization Service | ❌ | ✅ | ✅ | ✅ |
| Extended Time Travel (up to 90 days; Standard = 1 day) | ❌ (1 day only) | ✅ | ✅ | ✅ |
| Column-level security / dynamic data masking | ❌ | ✅ | ✅ | ✅ |
| Row access policies | ❌ | ✅ | ✅ | ✅ |
| HIPAA/HITRUST/PCI support, enhanced encryption | ❌ | ❌ | ✅ | ✅ |
| Database failover/failback (business continuity/DR) | ❌ | ❌ | ✅ | ✅ |
| Tri-Secret Secure (customer-managed keys) | ❌ | ❌ | ✅ | ✅ |
| Dedicated, fully isolated infrastructure (no shared metadata store with other customers) | ❌ | ❌ | ❌ | ✅ |

**Key hierarchy fact:** Each edition is a **superset** of the previous one — Enterprise = Standard + more, Business Critical = Enterprise + more, VPS = Business Critical + full isolation.

### Edge Cases & Gotchas
- **Business Critical** was formerly called "Enterprise for Sensitive Data (ESD)" — the exam may reference either name.
- The **primary differentiator** for Enterprise vs Standard is **multi-cluster warehouses** (horizontal scaling for concurrency) — this is the single most commonly tested edition difference.
- **VPS accounts share no resources at all** with any other Snowflake accounts (not even the metadata layer) — this is the key distinguishing point vs. Business Critical, which is still on shared cloud services infrastructure.
- Per-credit cost **increases** with each edition tier.

### Exam Traps
- "Which edition is needed for row access policies / masking policies / multi-cluster warehouses / materialized views / search optimization?" → **Enterprise (or higher)**. This is a very frequently tested list — memorize it.
- Time Travel: Standard Edition caps at **1 day**; Enterprise+ can extend to **up to 90 days** (configurable via `DATA_RETENTION_TIME_IN_DAYS`).

---

## 1.2 Snowflake Interfaces and Tools

### Snowsight (Web UI)
- Snowsight is the modern web interface (replaced the "Classic Console").
- **Worksheets**: SQL or Python worksheets; each worksheet has its own **context** (role + warehouse) that is independent of your session's active role/warehouse.
- Worksheets have a **maximum size of 1 MB** — larger Classic Console worksheets fail to import.
- Supports Query Profile, chart visualizations, data loading, Streamlit app development, Notebooks, and Cortex/AI & ML Studio, all inside the same UI.
- **Query History** and **Query Profile** are accessible per worksheet for troubleshooting/performance tuning.

### Snowflake CLI
- Command-line tool for creating, managing, updating, and viewing objects/apps across workloads (SQL execution, Snowpark, Native Apps, Streamlit deployment, etc.) — the modern successor role to the older **SnowSQL** CLI client.
- **SnowSQL** (older, still supported): a separate command-line client specifically for running SQL, useful for scripting/batch/automation and scheduled tasks outside of Snowsight.

### IDE Integrations (VS Code)
- The **Snowflake Extension for Visual Studio Code** lets you connect to Snowflake, browse the Database Explorer (context-aware to your active role), write/run SQL directly against Snowflake, and view results — without leaving VS Code.
- SQL files live as **local files on disk** (not stored as Snowsight worksheets) — this gives native Git/source-control integration, but moving code back into a Snowsight worksheet is a manual step (copy/paste or import).
- Supports SSO/SAML authentication.

### Edge Cases & Gotchas
- A **worksheet's context** (role/warehouse) is saved with the worksheet and shared with anyone the worksheet is shared with — it is **not** the same as your personal "active role" selector elsewhere in Snowsight. Changing your global active role does **not** change a worksheet's saved context.
- Query results/statistics in Snowsight are available for up to **1 million rows**.

### Exam Traps
- Know that **Snowsight, SnowSQL, Snowflake CLI, and the VS Code extension** are all valid ways to interact with Snowflake — a scenario question about "developer wants Git-integrated local development" points to **VS Code extension**; "wants to script scheduled batch SQL jobs" points to **SnowSQL/Snowflake CLI**.

---

## 1.3 Snowflake Object Hierarchy and Types

### Organization & Account Hierarchy
```
Organization
  └── Account(s)  (each account = single region + single cloud platform)
        └── Database(s)
              └── Schema(s)
                    └── Objects (Tables, Views, Stages, File Formats, Sequences,
                                  UDFs, Stored Procedures, Pipes, ML Models, etc.)
```
- **Organization**: top-level container that can manage multiple accounts across regions/clouds — managed via the **ORGADMIN** role (a special role that sits outside the normal RBAC hierarchy).
- **Account**: a unique identifier; all databases belong to exactly **one** account.
- **Database** → **Schema** → **Objects**: this is the standard **namespace** — a database + schema together form a fully-qualified namespace (`db.schema.object`).
- **Account objects** (not inside a database/schema): warehouses, users, roles, shares, resource monitors, network policies.
- **Database objects** (inside a schema): tables, views, stages, file formats, sequences, UDFs, stored procedures, pipes, streams, tasks, dynamic tables.

### Key Database Objects — Quick Reference

| Object | Purpose | Key Gotchas |
|---|---|---|
| **Stage** | Named location (internal or external) for staging files for load/unload | 3 types: **User** (`@~`), **Table** (`@%table_name`, implicit, no separate privileges — must be table owner), **Named** (schema-level object, explicit privileges) |
| **Schema** | Logical container for objects | Can be **transient**; a transient database's schemas/tables are transient by definition |
| **Table** | Stores structured/semi-structured data | See table types section 1.5 |
| **View** | Named, saved query | Standard, Materialized, Secure — see 1.5 |
| **UDF** | User-Defined Function, reusable logic in SQL/JS/Python/Java/Scala | Can be secure; scalar or tabular (UDTF) |
| **File Format** | Named object describing file structure (CSV, JSON, Parquet, etc.) for loading/unloading | Reusable across multiple COPY INTO statements |
| **Stored Procedure** | Encapsulated procedural logic (SQL scripting, JS, Python, Java, Scala) | Runs with caller's or owner's rights depending on definition |
| **Pipe** | Object wrapping a COPY INTO statement for **Snowpipe** continuous/automated ingestion | Snowpipe uses **serverless compute**, not a user-managed warehouse |
| **Share** | Object enabling **Secure Data Sharing** to other Snowflake accounts | Zero-copy — no data movement/duplication |
| **Sequence** | Generates unique numbers | Independent of any table; can be referenced by multiple tables |
| **ML Model** | First-class schema-level object registered in the **Model Registry** | Versionable, one designated as default |
| **Application** | Snowflake Native App object (built from an Application Package) | Can include Streamlit, stored procs, UDFs, Snowpark Container Services workloads |

### Table Stage vs Named Stage vs User Stage (classic exam trap)
- **Table stage**: implicit, one per table, cannot be altered/dropped directly, only usable by the table owner, files here only load into that single table.
- **User stage**: implicit, one per user (`@~`), private to that user, cannot be altered/dropped, cannot be shared with others.
- **Named stage**: explicit schema-level object, can be shared via grants, most flexible, can load into **multiple** tables.

### Session & Context Variables
- Every Snowflake session has a **context**: current role, current warehouse, current database, current schema.
- Context functions: `CURRENT_ROLE()`, `CURRENT_WAREHOUSE()`, `CURRENT_DATABASE()`, `CURRENT_SCHEMA()`, `CURRENT_USER()`, etc.
- If database/schema aren't set for the session, object references **must be fully qualified**.

### Parameter Hierarchy & Precedence — Critical Exam Topic

There are **three types of parameters**:

| Type | Set at | Overridable at |
|---|---|---|
| **Account parameters** | Account level only (via `ALTER ACCOUNT`) | Cannot be overridden lower — applies account-wide |
| **Session parameters** | Account level (sets default) | → User level (`ALTER USER`) → Session level (`ALTER SESSION`, current session only) |
| **Object parameters** | Account level (sets default) | → Object level, following the **object's own container hierarchy** (e.g., Database → Schema → Table) |

**Precedence rule (most specific wins):**
```
Session parameters:  Account → User → Session  (lowest/most specific wins)
Object parameters:   Account → Database → Schema → Table/Iceberg Table (lowest/most specific wins)
```

- Only a role with the right privilege (typically **ACCOUNTADMIN** or a role granted the privilege) can set **account** parameters.
- **SECURITYADMIN** (or a role with sufficient privilege) can override session parameters for individual **users** via `ALTER USER`.
- Individual users can override session parameters for themselves via `ALTER SESSION` (their current session only, doesn't persist beyond the session unless set at the user level).
- Example: `DATA_RETENTION_TIME_IN_DAYS` is an **object** parameter — settable at Account → Database → Schema → Table, with the most specific (table-level) setting winning.
- `SHOW PARAMETERS` by default shows **session-level** parameters; you must specify scope (e.g., `IN WAREHOUSE`, `IN ACCOUNT`) to see other levels. Account parameters are **not shown by default**.

### Exam Traps
- If a parameter is set at multiple levels, the **most specific/lowest level always wins** (this is the #1 tested concept in this section).
- Warehouses **do not have a hierarchy** the way databases do — warehouse parameters go directly Account → individual Warehouse (no intermediate container).
- Database roles **cannot be activated directly in a session** — they must be granted to an account role first, then that account role is activated.
- **ORGADMIN** does not fit into the standard SYSADMIN-rooted role hierarchy; it exists purely for organization-level operations (e.g., creating new accounts).

---

## 1.4 Configuring Virtual Warehouses

### Warehouse Types

| Type | Purpose | Notes |
|---|---|---|
| **Standard** | General-purpose SQL query/DML processing | Default type; comes in Gen1 and Gen2 |
| **Snowpark-Optimized** | Large-memory workloads (e.g., ML training via Snowpark ML) | 16x memory per node vs. standard; minimum size **Medium** (no X-Small/Small); default size Medium; doesn't support Query Acceleration Service |
| **Default warehouse for Notebooks** | *(Feature will not be tested until globally GA per exam guide)* | — |

### Gen1 vs Gen2 Standard Warehouses
- **Gen2** runs on newer hardware (e.g., AWS Graviton3/C7g instances) — faster CPUs, better performance especially for **complex operations** (joins, CTAS, heavy MERGE/UPDATE/DELETE), less benefit for simple SELECTs.
- Gen2 is now the **default generation** for new standard warehouses in supported regions (was Gen1 historically).
- **Gen2 is only available for STANDARD warehouses**, sizes up to **4X-Large** (X5Large/X6Large default to Gen1).
- Cannot apply `STANDARD_GEN_2` to a Snowpark-optimized warehouse (those use `MEMORY_*` resource constraints instead).
- Converting Gen1 → Gen2 **without suspending first**: in-flight queries finish on Gen1 resources while new queries route to Gen2 — you are billed for **both** simultaneously during the transition; the warehouse won't auto-suspend during this overlap.
- When Snowflake auto-creates a new Gen2 (or multi-cluster) warehouse, **Query Acceleration Service (QAS)** is enabled by default with a max scale factor of **2** (vs. 8 for Gen1 or explicitly-enabled QAS).
- Gen2 warehouses **cannot be created via Snowsight/Classic Console UI** — only via SQL (`CREATE WAREHOUSE ... RESOURCE_CONSTRAINT = STANDARD_GEN_2` or `GENERATION = '2'`).

### Warehouse Sizes
- X-Small, Small, Medium, Large, X-Large, 2X-Large, 3X-Large, 4X-Large, 5X-Large, 6X-Large.
- Each size increase roughly **doubles** compute resources **and** credit consumption per hour.
- Billing is **per-second** with a **60-second minimum** each time the warehouse starts/resumes.
- Larger warehouse ≠ automatically faster data loading — loading performance depends more on **number and size of files** than warehouse size.

### Scaling: Up vs. Out

| Scale **Up** (vertical) | Scale **Out** (horizontal) |
|---|---|
| Resize the warehouse (e.g., Small → Large) | Add clusters to a **multi-cluster warehouse** (Enterprise Edition+) |
| Improves performance for large/complex individual queries | Handles **concurrency** (many simultaneous users/queries), not query complexity |
| Can be done any time, even while running | Requires Enterprise Edition or higher |

**⚠️ Exam trap:** Resizing (scaling up) is **not** the correct tool for concurrency/queuing problems — that's what multi-cluster (scaling out) is for.

### Multi-Cluster Warehouses
- **Enterprise Edition or higher** feature.
- Modes:
  - **Maximized**: `MIN_CLUSTER_COUNT = MAX_CLUSTER_COUNT` (all clusters run at all times, no scaling logic).
  - **Auto-scale**: `MIN_CLUSTER_COUNT < MAX_CLUSTER_COUNT` (clusters start/stop dynamically based on load).
- **Scaling Policy** only applies in Auto-scale mode:

| Policy | Behavior | Trade-off |
|---|---|---|
| **Standard** (default) | Starts a new cluster after ~20 seconds of sustained queuing | Prioritizes performance/minimizing queuing; may over-provision for brief spikes |
| **Economy** | Waits ~6 minutes to confirm sustained load before adding a cluster | Prioritizes cost; tolerates some queuing to avoid short-lived clusters |

- The old **"Legacy"** scaling policy has been **removed** — any warehouses using it now default to Standard.
- Auto-suspend/auto-resume apply to the **entire warehouse**, not individual clusters:
  - Auto-suspend triggers only once the **minimum** number of clusters is idle for the timeout.
  - Auto-resume triggers only when the **entire** warehouse (all clusters) is suspended.
- **Interactive warehouses** only support the **Standard** scaling policy (more proactive scaling to stay responsive to interactive workloads).

### Warehouse Use-Case Configuration Guidance

| Use Case | Recommended Approach |
|---|---|
| **Ad-hoc queries** | Small-to-medium warehouse, short auto-suspend (e.g., 1–5 min), single-cluster fine for individual/small teams |
| **Data loading (ETL/bulk)** | Size warehouse to match **number/size of files**, not data volume; usually small-to-medium warehouses suffice unless loading hundreds/thousands of files concurrently |
| **BI / reporting (high concurrency)** | Multi-cluster warehouse in Auto-scale mode to absorb concurrent dashboard users; Standard scaling policy if responsiveness matters more than cost |

### Best Practices

**Sizing (Up/Down)**
- Start small, monitor for **spillage** (`bytes_spilled_to_local_storage`/`bytes_spilled_to_remote_storage` in `QUERY_HISTORY`) — remote spillage is a strong signal you're undersized (memory-bound); size up.
- If average warehouse load is well under capacity, you may be oversized — size down.

**Scaling (In/Out)**
- Use multi-cluster (out) for **concurrency**, not warehouse resize (up) for concurrency.
- Start with a small Max Cluster Count (e.g., 2–3) and increase based on observed queuing.

**Auto-Suspend**
- Warehouses accrue **no credits while suspended**.
- Shorter auto-suspend = lower idle cost, but frequent resumes have a small (1–2 second) cold-start latency — balance against workload pattern.
- There is **no benefit to manually suspending** before the first 60-second billing minimum has elapsed (you're billed regardless).

**Workload Separation**
- Use **separate warehouses per team/workload type** (e.g., ETL warehouse, BI warehouse, data-science warehouse, dev warehouse) for:
  - Clean cost attribution (`WAREHOUSE_METERING_HISTORY` per warehouse).
  - Workload **isolation** — a heavy ETL job doesn't starve/queue-block interactive BI users (since each warehouse is a fully separate compute cluster).
- For **high concurrency**, prefer multi-cluster warehouses over one giant warehouse.
- For **complex queries** (large joins/aggregations), scale up (bigger single-cluster) rather than out.

### Exam Traps
- Multi-cluster warehouses = **Enterprise Edition and above only**.
- Scaling **out** solves concurrency; scaling **up** solves single-query performance/complexity — a scenario question that says "queries are queuing but each individual query runs fast" → **add clusters (scale out)**, not resize.
- Snowpark-optimized warehouses: minimum size is **Medium**; you cannot set X-Small/Small.
- QAS (Query Acceleration Service) is **not supported on Snowpark-optimized warehouses**.
- Resuming a Snowpark-optimized warehouse may take **longer** than a standard warehouse.

---

## 1.5 Snowflake Storage Concepts

### Micro-Partitions
- All Snowflake table data is **automatically** divided into micro-partitions — this is automatic, non-configurable, and applies to every table.
- Size: **50–500 MB of uncompressed data** per micro-partition (much smaller once compressed, roughly ~16 MB compressed on average).
- **Columnar** storage within each micro-partition (data stored by column, not by row).
- **Immutable**: once written, a micro-partition is never updated in place. DML (UPDATE/DELETE/MERGE) writes **new** micro-partitions and marks old ones as no-longer-current — this immutability is what enables Time Travel and zero-copy cloning.
- Snowflake automatically collects and stores **metadata** per micro-partition: min/max values, count of distinct values, null counts, etc. — all maintained in the **Cloud Services layer**.

### Pruning
- **Micro-partition pruning**: using stored metadata, Snowflake determines which micro-partitions can be skipped entirely for a given query — happens **before compute touches data**.
- **Column pruning**: within scanned micro-partitions, only the columns actually referenced are read (enabled by columnar storage).
- Pruning works on semi-structured data columns too (e.g., VARIANT).
- Dropping a column is a **metadata-only operation** — the underlying data in the dropped column's micro-partitions is **not immediately rewritten**; it just becomes inaccessible.
- Deleting **all rows** from a table can be a **metadata-only operation** in some cases (no data actually rescanned).

### Data Clustering & Clustering Keys
- Natural clustering happens as data loads (often correlates with load order, e.g., date).
- Over time, DML operations (and continuous ingestion like Snowpipe) can **degrade** natural clustering, hurting pruning efficiency and query performance.
- **Clustering key**: explicitly chosen column(s)/expression(s) that tell Snowflake how to keep data physically co-located.
  - Defined via `CLUSTER BY` at `CREATE TABLE` or added later via `ALTER TABLE ... CLUSTER BY (...)`.
  - Changing the clustering key via `ALTER TABLE` does **not retroactively reorganize existing data** until Snowflake performs reclustering.
  - Snowflake **recommends 3–4 columns maximum** in a clustering key — more tends to increase cost more than it improves query benefit.
  - **Column order matters**: generally order from **lowest to highest cardinality** (putting high-cardinality columns first reduces effectiveness of clustering on subsequent columns).
  - For string/VARCHAR columns, Snowflake only considers the **first few bytes** for clustering purposes — very high-cardinality strings may cluster poorly.
  - Wrapping a filtered column in a function (e.g., `WHERE TO_VARCHAR(date_col) = ...`) **defeats pruning** — avoid transformations on filter columns if you want pruning benefits.
- **Reclustering** is a Snowflake-managed **background process** (tracked under the `AUTOMATIC_CLUSTERING` internal warehouse) — it consumes **compute credits** (billed based on actual usage) and can increase **storage costs** too (new micro-partitions are generated even for small changes).
- **When to specify a clustering key**: only for very large tables (multi-TB range) where load order doesn't match common query filter patterns, AND query performance is degrading. Clustering small tables typically provides negligible benefit.
- **Clustering vs. indexing**: Snowflake has **no traditional indexes** — clustering + metadata-driven pruning is the mechanism used instead.
- Diagnostic functions: `SYSTEM$CLUSTERING_DEPTH()`, `SYSTEM$CLUSTERING_INFORMATION()`, `SYSTEM$CLUSTERING_RATIO()`. Clustering ratio of 100 = perfectly clustered (no overlap).

### Table Types — Full Comparison

| Table Type | Time Travel | Fail-safe | Persists beyond session? | Visible to other users? | Key Use |
|---|---|---|---|---|---|
| **Permanent** (default) | 0–90 days (Standard=1 day max; Enterprise+ up to 90) | **7 days, fixed, non-configurable** | Yes | Yes (with privileges) | Long-lived, critical/production data |
| **Temporary** | 0 or 1 day (ends when session ends, whichever is shorter) | **None** | **No** — dropped automatically when session ends | **No** — private to the creating session only | Session-scoped scratch data, ETL staging within a session |
| **Transient** | 0 or 1 day only | **None** | Yes, until explicitly dropped | Yes (with privileges) | Data needing to persist beyond a session but not needing DR — e.g., staging tables, reproducible data |
| **External** | Not supported | Not supported | Yes (metadata only; data lives outside Snowflake) | Yes (with privileges) | Query files in a data lake (S3/Blob/GCS) without loading; **read-only** |
| **Apache Iceberg** | Supported | Supported (Snowflake-managed variant) | Yes | Yes | Open table format; interoperable with Spark/Trino etc.; can be Snowflake-managed or externally managed catalog |
| **Dynamic** | Supported | 7-day default (can be made transient) | Yes | Yes | Declarative, auto-refreshing transformation pipelines (replaces manual streams+tasks in many cases) |
| **Hybrid** | N/A (OLTP-style) | N/A | Yes | Yes | Row-based, low-latency, unique/referential integrity enforced (Unistore) — for transactional workloads mixed with analytics |

### Edge Cases & Gotchas — Table Types
- **Fail-safe is NOT a Time Travel extension** — it is a **non-configurable, 7-day, Snowflake-support-only** recovery mechanism for **permanent tables only**; you (the customer) cannot self-service Fail-safe recovery — only Snowflake Support can, and only for catastrophic failure scenarios.
- If a permanent table's Time Travel retention is set to **0**, the table **immediately enters Fail-safe upon drop**.
- **Transient and Temporary tables have ZERO Fail-safe** — this is the core reason they're cheaper for large/disposable datasets.
- A **temporary table can share the same name** as an existing permanent/transient table in the same schema — the temporary table **takes precedence** for that session.
- The **TRANSIENT property is set at creation and is immutable** — you **cannot ALTER** a permanent table to transient or vice versa. To "convert," you must `CREATE TABLE ... AS SELECT` into a new table of the desired type, re-apply grants, then drop/rename.
- You **can** clone a permanent table into a transient table (zero-copy clone with a lifecycle change) — but you **cannot clone a transient table into a permanent table**.
- A long-running Time Travel query **delays purging** of temporary/transient tables until that query finishes.
- **Hybrid tables** cannot be temporary or transient, and cannot exist inside a transient database/schema.
- **External tables are read-only** — no INSERT/UPDATE/DELETE; you CAN build views (including materialized views) on top of them for performance.
- **Dynamic tables**: minimum `TARGET_LAG` is **60 seconds**; `TARGET_LAG` is a **staleness target, not a guaranteed refresh interval** — actual lag can exceed target under load. `TARGET_LAG = DOWNSTREAM` means the table only refreshes when a downstream dynamic table needs it to (and if it has no downstream consumers, it **never** auto-refreshes — no warning is given).

### Views: Standard, Materialized, Secure

| View Type | Stores data? | Performance | Key Rules |
|---|---|---|---|
| **Standard (non-materialized)** | No — query re-executed every reference | Slower (recomputed each time) | Most common type; can be recursive; read-only (no direct DML, but usable in a subquery of a DML statement) |
| **Materialized View** | **Yes** — pre-computed result stored | Fast (like scanning a table) | **Enterprise Edition+ feature**; auto-refreshed by Snowflake in the background when base table changes; consumes storage + compute |
| **Secure View** | Depends (can be standard OR materialized) | **Slower** than non-secure equivalent | Hides view **definition** from unauthorized users; disables certain query-optimizer internal shortcuts to prevent data leakage |

### Materialized View — Critical Limitations
- Can only query a **single table** — **no joins**, no self-joins.
- **No DML** allowed on a materialized view (no INSERT/UPDATE/DELETE/TRUNCATE).
- The view **definition cannot be altered** — must `CREATE OR REPLACE`.
- Restrictions on window functions, non-deterministic functions, and certain clauses.
- Can be made **secure** (`CREATE SECURE MATERIALIZED VIEW`).
- Good use case: pre-aggregating a subquery over data that changes relatively infrequently, especially useful for accelerating queries against **external tables**.

### Secure View — Critical Details
- View definition is **hidden** from users who are not the owning role — even from `SHOW VIEWS`, `GET_DDL()`, `INFORMATION_SCHEMA.VIEWS`, and Query Profile (even the view owner's Query Profile hides internals from **other roles** viewing it).
- For queries against secure views, Snowflake **does not expose bytes/micro-partitions scanned** — protects data-volume inference by users with only partial access.
- Trade-off: **query performance is worse** than a non-secure view because certain internal optimizations that would require peeking at underlying data are disabled.
- **Non-secure views should never be used for row/column-level security guarantees** if data privacy matters — always use secure views (or secure materialized views) for that purpose.
- Materialized views also support being **secure** — check the secure flag via `SHOW MATERIALIZED VIEWS`, not `SHOW VIEWS`.

### Exam Traps
- Materialized views are an **Enterprise Edition feature** — same tier gate as multi-cluster warehouses, masking, row access policies.
- **A view definition is NOT automatically updated when the base table's schema changes** — changes to underlying tables are not automatically propagated/validated into the view; you may see the view break at query time.
- Clustering keys are a **manual optimization tool for very large tables** — Snowflake performs automatic micro-partitioning/optimization for all tables regardless; you don't *need* a clustering key for most workloads.
- **Recursive views** exist in Snowflake (via `RECURSIVE` CTE syntax) — don't assume views can never reference themselves.

---

## 1.6 AI/ML and Application Development Features

### Snowflake Notebooks
- Cell-based, interactive Python/SQL/Markdown development environment running **natively inside Snowflake** — no separate environment setup, no data export needed.
- Runs on either a virtual warehouse or a **Container Runtime** (Snowpark Container Services) for GPU/heavier ML workloads.
- Comes pre-installed with Snowpark Python; supports pandas, NumPy, scikit-learn, and other common data-science libraries via package management.
- Integrates with Git (GitHub/GitLab/BitBucket/Azure DevOps) for version control.
- Can embed **Streamlit** visualizations directly in notebook cells.
- Session variable is auto-available via `get_active_session()` — no manual credential/connection management needed.

### Streamlit in Snowflake (SiS)
- Streamlit is an open-source Python library for building data apps; **Streamlit in Snowflake** runs those apps natively inside Snowflake — app code and compute stay inside Snowflake's governance boundary (no data movement to an external system).
- A Streamlit app is a first-class Snowflake object with **RBAC** controlling access.
- Snowflake **acquired Streamlit** (2022) to integrate it as a native app-development experience.
- Can leverage Snowpark, UDFs, and stored procedures from within a Streamlit app for backend logic.

### Snowpark
- The set of **libraries and runtimes** (Python, Java, Scala) that let you write non-SQL code that executes **inside Snowflake**, close to the data — no need to move data to an external application/runtime.
- Provides a DataFrame-style API similar to Spark/pandas.
- Includes **Snowpark ML** for model development and **Snowpark pandas** (via Modin) for a pandas-compatible DataFrame API that pushes computation into Snowflake.
- Snowpark workloads can run on **either Standard or Snowpark-optimized warehouses** — Snowpark-optimized is recommended specifically for **large-memory** needs (e.g., ML training).

### Snowflake Cortex (AI/ML feature suite)

**Cortex AI (SQL) Functions** — task-specific, managed LLM-powered SQL functions, no infrastructure to manage:
- `AI_COMPLETE` — general-purpose text/image completion using a selected LLM (the primary "call any model" function).
- `AI_CLASSIFY` — classify text/images into user-defined categories.
- `AI_FILTER` — returns TRUE/FALSE, usable directly in `WHERE`/`JOIN` clauses for semantic filtering.
- `AI_SENTIMENT`, `AI_TRANSLATE`, `AI_EXTRACT`, `AI_TRANSCRIBE`, `AI_AGG` (aggregate insight across rows), and more.
- Requires the **`USE AI FUNCTIONS`** account-level privilege plus the **`CORTEX_USER`** (or `AI_FUNCTIONS_USER`) database role.
- Governance is inherited automatically — if a role doesn't have SELECT on a column, it cannot run an AI function against that column (RBAC, masking policies, row access policies all still apply).
- Supports multiple third-party model providers (e.g., models from Anthropic, Meta, Mistral, Google) hosted inside Snowflake's security perimeter — **data does not leave Snowflake's governance boundary**.

**Cortex Search**
- Managed **hybrid search** (semantic + keyword) service, purpose-built for RAG (retrieval-augmented generation) patterns over unstructured/enterprise document data.
- Created via `CREATE CORTEX SEARCH SERVICE` (SQL) or via **AI & ML Studio** in Snowsight.
- Requires a **warehouse** to materialize results during creation/refresh; has its own `TARGET_LAG` for keeping the index fresh.
- Requires the **`SNOWFLAKE.CORTEX_USER`** database role.
- Can index a table/view column, or (in preview) files directly from a stage.
- Auto-suspends indexing after **5 consecutive refresh failures**; must be manually resumed (`ALTER CORTEX SEARCH SERVICE ... RESUME INDEXING`) once the underlying issue is fixed.
- Has its own `AUTO_SUSPEND` property to reduce serving cost when idle.

**Cortex Analyst**
- Managed **text-to-SQL** service for self-serve, natural-language analytics.
- Relies on a **semantic model** (YAML — legacy stage-based files are still supported, but **Semantic Views** are the recommended modern approach) that maps business terms/metrics to the underlying schema — this is what gives Cortex Analyst much higher accuracy than "raw schema" text-to-SQL.
- Generated SQL executes inside your own Snowflake virtual warehouse (governed, cost-visible, standard security).
- Access requires **`SNOWFLAKE.CORTEX_USER`** or the narrower **`SNOWFLAKE.CORTEX_ANALYST_USER`** database role (Analyst-only access, no other Cortex AI features).
- Does **not train on customer data** — the semantic model metadata is used only for SQL generation at inference time.

### Snowflake ML
- For **custom** model development (as opposed to the managed, task-specific Cortex AI functions).
- Includes: **Feature Store**, **Model Registry** (models are first-class schema-level objects, versionable, one version can be designated default), framework connectors, and immutable data snapshots for reproducibility (MLOps capabilities).
- **ML Functions** (a related but distinct capability): out-of-the-box, no-code time-series style functions for common structured-data tasks — **anomaly detection**, **forecasting**, **classification**, and **top insights** (explaining metric fluctuations) — usable directly via SQL without needing a data scientist.

### Edge Cases & Gotchas
- Don't confuse **Cortex AI Functions** (task-specific SQL LLM calls on any data) with **Snowflake ML "ML Functions"** (structured-data forecasting/anomaly-detection/classification) — they solve different problem types and are frequently mixed up on exams.
- **Cortex Analyst** needs a **semantic model**, not just a raw schema — this is the single most tested differentiator vs. generic text-to-SQL tools.
- Cortex Search and Cortex Analyst both require specific **database roles** (`CORTEX_USER` at minimum) — plain warehouse/database privileges are not sufficient by themselves.
- Streamlit in Snowflake apps and Notebooks both consume **warehouse (or compute pool) credits** while running — they are not "free" features.
- Snowpark-optimized warehouses are recommended (not strictly required) for **large-memory** Snowpark ML training — Snowpark itself can run on Standard warehouses for lighter workloads.

### Exam Traps
- "Which Cortex feature would you use for natural-language querying of structured data via SQL generation?" → **Cortex Analyst** (not Cortex Search — Search is for unstructured/document retrieval).
- "Which Cortex feature is best for semantic search over a large collection of PDFs/support tickets?" → **Cortex Search**.
- "Which feature lets you call an LLM directly in a SQL SELECT statement to summarize/translate/classify text?" → **Cortex AI (SQL) Functions**, e.g., `AI_COMPLETE`/`AI_CLASSIFY`.
- Streamlit in Snowflake vs. plain open-source Streamlit: the Snowflake version keeps everything **inside Snowflake's security perimeter** — this is the "why" behind most exam questions on this topic.

---

## Quick-Reference: Feature-to-Edition Gate (High-Yield for Exam)

| Feature | Minimum Edition |
|---|---|
| Multi-cluster warehouses | Enterprise |
| Materialized views | Enterprise |
| Search Optimization Service | Enterprise |
| Column-level security (masking policies) | Enterprise |
| Row access policies | Enterprise |
| Extended Time Travel (up to 90 days) | Enterprise |
| HIPAA/PCI/HITRUST compliance support, enhanced encryption | Business Critical |
| Database failover/failback | Business Critical |
| Tri-Secret Secure (customer keys) | Business Critical |
| Fully isolated dedicated infrastructure | VPS |

---

## Consolidated "Gotcha" Cheat Sheet

1. Cloud Services is billed only above **10%** of daily compute credit usage — not always free, not always billed.
2. Fail-safe = **7 days, fixed, permanent tables only, Snowflake-Support-recovery-only** — never confuse with Time Travel.
3. Transient/Temporary tables = **0 Fail-safe**, max **1-day** Time Travel.
4. `TRANSIENT` property is **immutable** after table creation — no ALTER conversion.
5. Materialized views: **single table only, no joins, no DML, Enterprise+ only**.
6. Secure views: hide definition + hide scan stats, but **slower** performance.
7. Scaling **out** (multi-cluster) = concurrency fix; scaling **up** (resize) = single-query performance fix.
8. Multi-cluster warehouses = **Enterprise+ only**; Standard vs Economy scaling policy = ~20 sec vs ~6 min reaction time.
9. Snowpark-optimized warehouses: **minimum size Medium**, no QAS support, longer resume time.
10. Parameter precedence = **most specific level always wins** (Account → ... → most granular object/session).
11. Gen2 warehouses: **Standard type only**, up to 4X-Large, created via SQL only (not UI).
12. A temporary table **shadows** a same-named permanent/transient table for that session only.
13. Cortex Analyst needs a **semantic model**; Cortex Search is for **unstructured/document** retrieval; Cortex AI SQL functions are **inline LLM calls**.
14. Zero-copy cloning and Time Travel both work because micro-partitions are **immutable** and referenced via metadata, not because of any special "cloning engine."
15. VPS is the only edition with **zero shared infrastructure** (not even shared cloud-services metadata) with other customers.

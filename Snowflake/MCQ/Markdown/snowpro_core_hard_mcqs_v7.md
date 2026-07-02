# SnowPro Core (COF-C03) – 100 Hard-Level Practice MCQs — Set 2

> This set is a companion to your existing `snowpro_core_hard_mcqs_v6.md`. Every question below covers a **different topic or angle** than the 100 questions in that file — no repeats. It leans into the newer COF‑C03 exam areas (Apache Iceberg tables, Hybrid tables, Snowflake Cortex AI, Git integration, Budgets/FinOps) alongside deeper cuts of classic Core topics.
>
> Click **"Show Answer"** under each question to reveal the correct option and explanation.

---

## Section 1: Architecture & AI Data Cloud Capabilities (Q1–Q20)

### Q1. You are creating an Apache Iceberg table in Snowflake that will use a remote Iceberg REST catalog (not Snowflake) to manage table metadata. What object must you create first to tell Snowflake how to connect to and interpret that external catalog's metadata?

- A) A storage integration
- B) A catalog integration
- C) A file format object
- D) An external function

<details><summary>Show Answer</summary>

**Answer: B) A catalog integration**

A catalog integration is a named, account-level object that stores information about how an Iceberg table's metadata is organized when Snowflake is not acting as the catalog itself — for example, when using AWS Glue, Snowflake Open Catalog, or another Iceberg REST catalog. A storage integration only manages cloud storage credentials, a file format defines how flat files are parsed, and external functions call out to remote services, none of which describe catalog metadata organization.
</details>

---

### Q2. Before you can create any Apache Iceberg table in Snowflake — regardless of which catalog option you use — which object must already exist to define where the table's data and metadata files are physically stored?

- A) A named stage
- B) An external volume
- C) A resource monitor
- D) A materialized view

<details><summary>Show Answer</summary>

**Answer: B) An external volume**

An external volume is a named, account-level object that stores the IAM entity Snowflake uses to securely connect to your cloud storage location for Iceberg data, metadata, and manifest files. It must be created (or set to a Snowflake-managed default) before any Iceberg table can be created. Named stages serve a different purpose for regular file staging, resource monitors control credit spend, and materialized views have nothing to do with Iceberg storage location.
</details>

---

### Q3. A team wants to create an Iceberg table where Snowflake itself is the catalog, but their external volume points to cloud storage hosted in a different region than their Snowflake account. What happens?

- A) Snowflake automatically replicates the metadata to the account's local region at no extra cost
- B) The CREATE ICEBERG TABLE statement fails, because with Snowflake as the catalog, the external volume's storage location must be in the same region as the account
- C) The table is created but remains permanently read-only
- D) Snowflake silently ignores the region mismatch and creates the table normally

<details><summary>Show Answer</summary>

**Answer: B) The CREATE ICEBERG TABLE statement fails, because with Snowflake as the catalog, the external volume's storage location must be in the same region as the account**

Cross-cloud and cross-region Iceberg tables are not currently supported when Snowflake is used as the Iceberg catalog — the external volume must use an active storage location in the same region as the Snowflake account, or the create statement errors out. This regional restriction does not apply the same way when using an external REST catalog with private connectivity options. There is no silent auto-replication or automatic read-only fallback.
</details>

---

### Q4. Which constraint type is uniquely required on every Hybrid table in Snowflake, unlike standard tables where it's entirely optional?

- A) A UNIQUE constraint
- B) A FOREIGN KEY constraint
- C) A PRIMARY KEY constraint
- D) A CHECK constraint

<details><summary>Show Answer</summary>

**Answer: C) A PRIMARY KEY constraint**

Hybrid tables require a primary key, which Snowflake enforces for uniqueness and uses for indexed, low-latency point lookups — this is what powers their OLTP-style performance. Standard Snowflake tables treat all constraints, including primary keys, as informational only and never require one. Unique, foreign key, and check constraints remain optional (and are handled differently) on Hybrid tables as well.
</details>

---

### Q5. Which combination of features is currently NOT supported on Hybrid tables?

- A) Row-level locking and referential integrity constraints
- B) Fail-safe, Streams, and cross-account Data Sharing
- C) Joining a Hybrid table with a standard table in the same query
- D) Atomic transactions that span a Hybrid table and a standard table

<details><summary>Show Answer</summary>

**Answer: B) Fail-safe, Streams, and cross-account Data Sharing**

Hybrid tables are purpose-built for low-latency transactional workloads, and as a trade-off they don't support Fail-safe, Streams, Snowpipe, materialized views, or sharing data across accounts. They fully support row-level locking, referential integrity, native joins with standard tables in the same query engine, and atomic cross-table-type transactions — all without any federation or two-phase commit orchestration.
</details>

---

### Q6. By default, if Session A writes a row to a Hybrid table and Session B immediately reads that same table, what consistency behavior should Session B expect?

- A) Session B is guaranteed to see the change instantly with zero possible delay, always
- B) Session B may see slightly stale data for up to roughly 100ms, unless READ_LATEST_WRITES is set to force strong consistency
- C) Session B will never see the change until the warehouse is restarted
- D) Session B's query is automatically blocked until Session A's transaction is visible everywhere

<details><summary>Show Answer</summary>

**Answer: B) Session B may see slightly stale data for up to roughly 100ms, unless READ_LATEST_WRITES is set to force strong consistency**

Hybrid tables use a session-based consistency model by default: a session always sees its own writes immediately, but reads from a different session may lag by under 100ms. Setting READ_LATEST_WRITES = true at the statement or session level removes this staleness at the cost of some added latency. There's no mechanism that blocks a reading session until every other session's writes propagate, nor any dependency on a warehouse restart.
</details>

---

### Q7. You run `ALTER WAREHOUSE my_wh SET ENABLE_QUERY_ACCELERATION = TRUE;` without specifying a scale factor. Separately, a brand-new Generation 2 multi-cluster warehouse has QAS enabled automatically at creation. What are the resulting default `QUERY_ACCELERATION_MAX_SCALE_FACTOR` values in each case, respectively?

- A) 8 for the explicit case, 2 for the Gen2 auto-enabled case
- B) 2 for the explicit case, 8 for the Gen2 auto-enabled case
- C) 0 for both cases, meaning unlimited scaling
- D) 4 for both cases, since that's the account-wide default

<details><summary>Show Answer</summary>

**Answer: A) 8 for the explicit case, 2 for the Gen2 auto-enabled case**

When QAS is explicitly turned on via `ENABLE_QUERY_ACCELERATION = TRUE` without setting a scale factor, the default is 8. When Snowflake automatically enables QAS on a new Gen2 or multi-cluster warehouse at creation time, the default scale factor is instead 2, which more conservatively limits the extra compute the service can lease. A scale factor of 0 is a valid setting that removes the upper bound entirely, but it is not the default in either scenario.
</details>

---

### Q8. How does Snowflake bill for Query Acceleration Service usage?

- A) It's included at no extra charge as part of standard warehouse compute credits
- B) It's billed by the second, only while the service is actively accelerating a query, as credits separate from warehouse compute
- C) It's a flat monthly subscription fee regardless of usage
- D) It's billed per query submitted to the warehouse, whether or not acceleration was used

<details><summary>Show Answer</summary>

**Answer: B) It's billed by the second, only while the service is actively accelerating a query, as credits separate from warehouse compute**

QAS credits are metered by the second and only accrue while the serverless acceleration resources are actively working on a query — they are tracked and billed separately from the credits the warehouse itself consumes. There's no flat subscription model, no inclusion in standard warehouse credits, and no charge merely for submitting a query that turns out not to need or use acceleration.
</details>

---

### Q9. What is the primary architectural benefit that Generation 2 (Gen2) standard warehouses offer over Generation 1 warehouses?

- A) Gen2 warehouses are free to run, eliminating compute credit charges entirely
- B) Gen2 warehouses use faster underlying hardware and software optimizations — especially for DML (update/delete/merge) and table scans — often finishing workloads faster despite a higher per-second credit rate
- C) Gen2 warehouses eliminate the need for clustering keys on any table
- D) Gen2 warehouses automatically convert all tables in the account to Hybrid tables

<details><summary>Show Answer</summary>

**Answer: B) Gen2 warehouses use faster underlying hardware and software optimizations — especially for DML (update/delete/merge) and table scans — often finishing workloads faster despite a higher per-second credit rate**

Gen2 standard warehouses run on enhanced hardware and include software improvements that specifically reduce write amplification on DML operations and speed up table scans, so many workloads complete measurably faster even though Gen2 charges a higher per-second credit rate than Gen1. Gen2 has no bearing on clustering key strategy, doesn't touch Hybrid table behavior, and is certainly not free.
</details>

---

### Q10. Which warehouse size option is explicitly unavailable when configuring a Generation 2 (Gen2) standard warehouse?

- A) X-Small
- B) Large
- C) X5Large and X6Large
- D) Medium

<details><summary>Show Answer</summary>

**Answer: C) X5Large and X6Large**

Gen2 standard warehouses currently top out below the largest Gen1 sizes — X5Large and X6Large are not offered for Gen2. X-Small, Medium, and Large are all valid Gen2 sizes, along with the sizes in between, so a workload requiring the very largest warehouse footprint would need to stay on Gen1 or use a different resource constraint.
</details>

---

### Q11. A team is storing scanned PDF invoices in an internal stage and wants to query file-level metadata (like file name, size, and last-modified time) directly with SQL, as well as automatically track new and removed files. What should they enable on the stage?

- A) A materialized view over the stage
- B) A directory table
- C) A masking policy on the stage
- D) A resource monitor on the stage

<details><summary>Show Answer</summary>

**Answer: B) A directory table**

Enabling a directory table on a stage (via `DIRECTORY = (ENABLE = TRUE)`) lets Snowflake maintain a queryable catalog of file-level metadata for the unstructured files sitting in that stage, refreshed automatically or on demand. Materialized views operate over structured table data, masking policies redact column values rather than track files, and resource monitors only manage credit consumption.
</details>

---

### Q12. In the Snowflake Native Apps Framework, what is the relationship between an "Application Package" and an "Application Object"?

- A) They are two names for the exact same object with no functional difference
- B) The Application Package is owned and versioned by the provider and contains the app's code and setup logic; the Application Object is the installed instance that a consumer creates from that package
- C) The Application Object must be created before the Application Package can exist
- D) Application Packages can only be created by consumers, never by providers

<details><summary>Show Answer</summary>

**Answer: B) The Application Package is owned and versioned by the provider and contains the app's code and setup logic; the Application Object is the installed instance that a consumer creates from that package**

A provider builds and version-controls an Application Package containing the app's setup script, code, and metadata; a consumer then installs it, which creates an Application Object in their own account that runs against their data under the app's defined privileges. The two serve distinct provider-side vs consumer-side roles, and the package must exist before any object can be installed from it.
</details>

---

### Q13. Through which Snowflake Marketplace capability do data providers create, configure, and publish both free and paid listings of their data or applications?

- A) Resource Monitors
- B) Provider Studio
- C) The ACCOUNT_USAGE schema
- D) Secure Views only, with no additional tooling

<details><summary>Show Answer</summary>

**Answer: B) Provider Studio**

Provider Studio is the interface Snowflake gives data and app providers to build, configure, and publish listings — including choosing between free and paid/usage-based monetization — to the Snowflake Marketplace or a private exchange. Resource Monitors control credit spend and have no listing function, ACCOUNT_USAGE is a metadata schema for auditing, and secure views are just one possible object type shared through a listing, not a publishing tool themselves.
</details>

---

### Q14. What is the core purpose of a "compute pool" in Snowpark Container Services?

- A) It is another name for a virtual warehouse used to run SQL queries
- B) It is a group of virtual machine nodes that Snowflake provisions and manages to run containerized application workloads, separate from virtual warehouses
- C) It is a caching layer that stores query results for reuse
- D) It is a Snowpipe object used exclusively for continuous file loading

<details><summary>Show Answer</summary>

**Answer: B) It is a group of virtual machine nodes that Snowflake provisions and manages to run containerized application workloads, separate from virtual warehouses**

A compute pool is a Snowflake-managed cluster of nodes dedicated to running Docker containers (services, jobs, or interactive workloads) under Snowpark Container Services — it is a distinct resource type from a virtual warehouse, which is used for SQL and Snowpark query execution. It has no relationship to the query result cache or to Snowpipe's file-loading mechanism.
</details>

---

### Q15. A user runs `SELECT COUNT(*) FROM large_table;` on a table with no filters, and the query returns almost instantly without visibly starting a warehouse. What best explains this?

- A) Snowflake always runs COUNT(*) for free, regardless of warehouse state
- B) The result can be answered directly from metadata (such as cached row counts) maintained by the cloud services layer, without scanning the actual table data
- C) The table must have been cached in the local SSD disk cache from a previous session
- D) COUNT(*) queries are automatically rewritten to use a materialized view

<details><summary>Show Answer</summary>

**Answer: B) The result can be answered directly from metadata (such as cached row counts) maintained by the cloud services layer, without scanning the actual table data**

Snowflake stores certain metadata about tables and micro-partitions — including row counts — as part of the cloud services layer, and some simple aggregate queries like an unfiltered COUNT(*) can be resolved directly from that metadata without spinning up or querying a warehouse at all. This isn't "free forever" as a blanket rule, it isn't dependent on a warehouse's local SSD cache (which requires a running warehouse), and it has nothing to do with materialized views unless one is explicitly defined.
</details>

---

### Q16. A team runs the same heavy query on warehouse `WH_A`, then resizes `WH_A` from Medium to Large, and reruns an unrelated query that happens to scan overlapping data. What happens to the warehouse's local (SSD) data cache built up before the resize?

- A) It carries over fully intact to the resized warehouse with no impact
- B) It is discarded — the local disk cache is tied to the specific running warehouse instance, and a resize (or suspend) effectively starts it fresh
- C) It automatically migrates to the account's shared remote result cache
- D) It is compressed and archived to Fail-safe for 7 days

<details><summary>Show Answer</summary>

**Answer: B) It is discarded — the local disk cache is tied to the specific running warehouse instance, and a resize (or suspend) effectively starts it fresh**

The warehouse-local SSD cache holds raw table data recently scanned by that specific warehouse's compute nodes; it isn't preserved across a resize or a suspend/resume cycle, since those operations change or reallocate the underlying compute nodes. This local cache is a completely separate mechanism from the persistent query result cache, and it has no relationship to Fail-safe, which protects historical table data, not warehouse caches.
</details>

---

### Q17. An account's total daily cloud services compute usage is being evaluated against its total daily warehouse (virtual warehouse) compute usage for billing purposes. Under what general condition does Snowflake typically waive charges for cloud services compute?

- A) Cloud services compute is always billed in full, with no exemption
- B) When cloud services usage on a given day is no more than approximately 10% of that day's total warehouse compute usage
- C) Only during the first 30 days of a new account
- D) Only if the account is on the Business Critical edition or higher

<details><summary>Show Answer</summary>

**Answer: B) When cloud services usage on a given day is no more than approximately 10% of that day's total warehouse compute usage**

Snowflake's standard billing adjustment excludes cloud services compute from a day's bill when it doesn't exceed roughly 10% of that day's warehouse (virtual warehouse) compute consumption — usage above that threshold is billed. This exemption isn't tied to a trial period and applies across editions, not just to Business Critical or higher.
</details>

---

### Q18. How does Time Travel behave differently for an Apache Iceberg table compared to a standard Snowflake table?

- A) Iceberg tables never support any form of historical data access
- B) Iceberg tables use a snapshot-based model where historical states are tracked through Iceberg metadata/manifest files rather than the standard micro-partition retention mechanism used by native tables
- C) Iceberg tables always retain 90 days of history by default with no configuration possible
- D) Iceberg tables can only access history through a completely separate FAIL_SAFE_HISTORY command

<details><summary>Show Answer</summary>

**Answer: B) Iceberg tables use a snapshot-based model where historical states are tracked through Iceberg metadata/manifest files rather than the standard micro-partition retention mechanism used by native tables**

Iceberg's format is inherently snapshot-based — every write creates a new snapshot referencing manifest and data files — so historical access for Iceberg tables in Snowflake is governed by that snapshot/metadata model rather than being identical to native table Time Travel internals. Iceberg tables do support historical access, it isn't fixed at a non-configurable 90 days, and there's no separate "FAIL_SAFE_HISTORY" command involved.
</details>

---

### Q19. A directory table has been enabled on a stage containing image files. Which column, automatically available when querying the directory table, provides a URL that can be used to reference a specific file (for example, to pass into a Cortex or external function)?

- A) FILE_HASH
- B) RELATIVE_PATH only, with no URL ever generated
- C) FILE_URL
- D) STAGE_OWNER

<details><summary>Show Answer</summary>

**Answer: C) FILE_URL**

Querying a directory table returns a FILE_URL column containing a URL you can use to reference and retrieve that specific staged file, which is commonly passed into functions that operate on unstructured data. RELATIVE_PATH is also returned but only gives the file's path within the stage, not a usable retrieval URL; FILE_HASH and STAGE_OWNER either don't exist as directory table columns or serve unrelated purposes.
</details>

---

### Q20. Which of the following operations can typically run using only the cloud services layer, without ever provisioning or billing time on a running virtual warehouse (assuming no metadata computation is required beyond cataloging)?

- A) A SELECT with a JOIN across two large fact tables
- B) A SHOW WAREHOUSES or DESCRIBE TABLE command
- C) An INSERT INTO ... SELECT statement with aggregation
- D) A CREATE TABLE ... AS SELECT from a multi-terabyte source

<details><summary>Show Answer</summary>

**Answer: B) A SHOW WAREHOUSES or DESCRIBE TABLE command**

Purely metadata-oriented operations like SHOW and DESCRIBE commands are handled entirely by the cloud services layer, since they only need to read/return object metadata rather than scan or compute over table data. Joins, aggregating inserts, and CTAS operations against real data all require compute from a running virtual warehouse to actually read and process the underlying rows.
</details>

---

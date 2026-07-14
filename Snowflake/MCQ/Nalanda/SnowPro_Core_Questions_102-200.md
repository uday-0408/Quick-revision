# SnowPro Core Practice Questions (102–200)

> Reformatted from a garbled source, cross-checked against current Snowflake documentation (July 2026). Where the original "community vote" answer was outdated or wrong based on current docs, it has been corrected and flagged with **⚠ Updated**.
>
> Click **Show Answer** to reveal each answer.

---

### Question 102
**True or False:** Snowflake bills for a minimum of five minutes each time a Virtual Warehouse is started.

<details><summary>Show Answer</summary>

**False.** Snowflake bills compute with a **60-second minimum** per start/resume, then per-second thereafter — not five minutes.
</details>

---

### Question 103
When scaling **up** a Virtual Warehouse (increasing its t-shirt size), you are primarily scaling for improved:

- -  A. Concurrency
- -  B. Performance

<details><summary>Show Answer</summary>

**-  B. Performance.** Scaling up gives a warehouse more compute power for complex/large queries. Scaling *out* (multi-cluster) is what improves concurrency.
</details>

---

### Question 104
As a best practice, clustering keys should only be considered for tables of which minimum size?

-  A. Multi-Kilobyte (KB) range
-  B. Multi-Megabyte (MB) range
-  C. Multi-Gigabyte (GB) range
-  D. Multi-Terabyte (TB) range

<details><summary>Show Answer</summary>

**-  D. Multi-Terabyte (TB) range.** Snowflake's automatic micro-partitioning is usually sufficient below this scale; clustering keys add reclustering costs, so they're recommended only for very large tables.
</details>

---

### Question 105
How are Snowpipe charges calculated?

-  A. Per-second, based on the warehouse t-shirt size used
-  B. Based on serverless compute resource consumption
-  C. Based on the number of pipes in the account
-  D. Based on total cloud storage bucket size

<details><summary>Show Answer</summary>

**-  B. Based on serverless compute resource consumption.**

**⚠ Updated:** Historically, Snowpipe was billed with **per-second/per-core granularity** on the serverless compute it consumed. As of the **2025-12-08 release**, Snowflake moved to **simplified per-GB pricing** — a fixed credit rate (0.0037 credits/GB, subject to change) per gigabyte of data ingested, rather than tracking compute-second/core utilization. Either way, Snowpipe is **not** billed by warehouse t-shirt size, pipe count, or storage bucket size, so B is still the best available answer, but the underlying billing mechanics have changed — verify current rates in the Snowflake Consumption Table.
</details>

---

### Question 106
**True or False:** A Snowflake account is charged for data stored in both internal and external stages.

<details><summary>Show Answer</summary>

**False.** Snowflake charges storage for **internal stages** (data lives in Snowflake-managed storage). Data in **external stages** resides in the customer's own cloud storage and is billed directly by the cloud provider, not by Snowflake.
</details>

---

### Question 107
**True or False:** When active, a Pipe uses a dedicated Virtual Warehouse.

<details><summary>Show Answer</summary>

**False.** Snowpipe uses Snowflake-managed **serverless compute**, not a customer-managed dedicated virtual warehouse.
</details>

---

### Question 108
**True or False:** Snowflake supports federated authentication in all editions.

<details><summary>Show Answer</summary>

**True.** Federated authentication (SSO) has been a baseline feature available in **all editions**, including Standard, since March 2019.
</details>

---

### Question 109
**True or False:** When a new Snowflake object is created, it is automatically owned by the user who created it.

<details><summary>Show Answer</summary>

**False.** In Snowflake's RBAC model, an object is owned by the **role** that was active in the session when the object was created — not the individual user.
</details>

---

### Question 110
**True or False:** A Virtual Warehouse consumes Snowflake credits even when inactive (suspended).

<details><summary>Show Answer</summary>

**False.** Suspended warehouses consume **zero credits**. Credits are only consumed while a warehouse is actively running.
</details>

---

### Question 111
**True or False:** During data unloading, only JSON and CSV files can be compressed.

<details><summary>Show Answer</summary>

**False.** Unloaded files can be compressed regardless of format (e.g., Parquet is compressed by default too), not just JSON/CSV.
</details>

---

### Question 112
Which of the following are options when creating a Virtual Warehouse? (Choose two.)

-  A. Auto-suspend
-  B. Auto-resume
-  C. Local SSD size
-  D. User count

<details><summary>Show Answer</summary>

**-  A. Auto-suspend** and **-  B. Auto-resume.** Local disk/SSD size and user count are not configurable warehouse creation parameters.
</details>

---

### Question 113
Which formats are supported for **unloading** data from Snowflake? (Choose two.)

-  A. Delimited (CSV, TSV, et-  C.)
-  B. Avro
-  C. JSON
-  D. ORC

<details><summary>Show Answer</summary>

**-  A. Delimited** and **-  C. JSON** (Parquet is also supported for unload, but wasn't offered as a valid pairing here). Avro, ORC, and XML are **load-only** formats — Snowflake cannot unload to them.
</details>

---

### Question 114
**True or False:** A Data Provider can share data with only a single Data Consumer.

<details><summary>Show Answer</summary>

**False.** A provider can share data with **multiple consumer accounts** simultaneously.
</details>

---

### Question 115
The Fail-safe retention period is how many days?

-  A. 1 day
-  B. 7 days
-  C. 45 days
-  D. 90 days

<details><summary>Show Answer</summary>

**-  B. 7 days.** This is a fixed, non-configurable period for all permanent tables in all editions.
</details>

---

### Question 116
**True or False:** Once created, a micro-partition will never be changed.

<details><summary>Show Answer</summary>

**True.** Micro-partitions are immutable. Any DML that modifies rows results in **new** micro-partitions being written; old ones are retained for Time Travel/Fail-safe until they age out.
</details>

---

### Question 117
What services does Snowflake automatically provide for customers that they may have previously been responsible for with an on-premises system? (Choose all that apply.)

-  A. Installing and configuring hardware
-  B. Patching software
-  C. Physical security
-  D. Maintaining metadata and statistics

<details><summary>Show Answer</summary>

**A, B, -  D.** Snowflake (via its cloud providers) also handles physical security, but in the classic SnowPro answer key this item is scoped to services Snowflake itself directly manages as a SaaS platform: hardware provisioning, software patching, and metadata/statistics maintenance.
</details>

---

### Question 118
Which of the following statements would be used to export/unload data from Snowflake?

-  A. `COPY INTO @stage`
-  B. `EXPORT TO @stage`
-  C. `INSERT INTO @stage`
-  D. `GET @stage`

<details><summary>Show Answer</summary>

**-  A. `COPY INTO @stage`.** This is the command used to unload table data to a stage.
</details>

---

### Question 119
**True or False:** A 4X-Large Warehouse may, at times, take longer to provision than an X-Small Warehouse.

<details><summary>Show Answer</summary>

**True.** Larger warehouses require more compute nodes to be provisioned, which can take more time, especially if there isn't spare capacity immediately available.
</details>

---

### Question 120
How would you determine the appropriate size of the virtual warehouse used for a task?

-  A. Since a root task may execute concurrently, leave margin in the execution window to avoid missed executions
-  B. Query the size of a stream's content to help determine warehouse size
-  C. If using a stored procedure to execute multiple SQL statements, test-run the procedure separately first to size the compute resource
-  D. Configure the warehouse for automatic concurrency handling using a multi-cluster warehouse to match the task schedule

<details><summary>Show Answer</summary>

**-  C.** Test-run the stored procedure separately (outside the task) to correctly size the warehouse before scheduling it as a task.
</details>

---

### Question 121
The Information Schema and Account Usage share provide storage information for which of the following objects? (Choose three.)

-  A. Users
-  B. Tables
-  C. Databases
-  D. Internal Stages

<details><summary>Show Answer</summary>

**B, C, D** — Tables, Databases, and Internal Stages. User objects don't have "storage" metrics tracked this way.
</details>

---

### Question 122
What is the default file format used in the `COPY INTO` command if one is not specified?

-  A. CSV
-  B. JSON
-  C. Parquet
-  D. XML

<details><summary>Show Answer</summary>

**-  A. CSV.**
</details>

---

### Question 123
**True or False:** Reader Accounts are able to extract data from shared data objects for use outside of Snowflake.

<details><summary>Show Answer</summary>

**False.** Reader accounts can only **query** shared data from within Snowflake — they cannot unload/export it for use outside the platform.
</details>

---

### Question 124
**True or False:** You can define multiple columns within a clustering key on a table.

<details><summary>Show Answer</summary>

**True.** A clustering key can be composed of multiple columns or expressions.
</details>

---

### Question 125
**True or False:** Snowflake enforces unique, primary key, and foreign key constraints during DML operations.

<details><summary>Show Answer</summary>

**False.** These constraint types are supported for **documentation/informational purposes** and by some tools, but are **not enforced** by Snowflake at DML time (NOT NULL is the exception — it *is* enforced).
</details>

---

### Question 126
**True or False:** Loading data into Snowflake requires that source data files be no larger than 16 M-  B.

<details><summary>Show Answer</summary>

**False.** There's no hard 16 MB limit on source file size for loading. Snowflake *recommends* compressed files in the 100–250 MB range for load efficiency, but larger files are allowed (they just load less efficiently/in parallel).
</details>

---

### Question 127
**True or False:** A Virtual Warehouse can be resized while suspended.

<details><summary>Show Answer</summary>

**True.** `ALTER WAREHOUSE ... SET WAREHOUSE_SIZE = ...` works whether the warehouse is running or suspended.
</details>

---

### Question 128
**True or False:** When you create a custom role, it is a best practice to immediately grant that role to ACCOUNTADMIN.

<details><summary>Show Answer</summary>

**False.** Best practice is to build a role hierarchy under **SYSADMIN**, not to grant custom roles directly to ACCOUNTADMIN (which should be reserved for account-level administration, not day-to-day object ownership).
</details>

---

### Question 129
Which of the following accurately represents how a table fits into Snowflake's logical container hierarchy?

-  A. Account → Table → Schema → Database
-  B. Account → Database → Schema → Table
-  C. Database → Table → Schema → Account
-  D. Table → Schema → Account → Database

<details><summary>Show Answer</summary>

**-  B. Account → Database → Schema → Table.**
</details>

---

### Question 130
**True or False:** All Snowflake table types include Fail-safe storage.

<details><summary>Show Answer</summary>

**False.** Only **permanent** tables have Fail-safe. Temporary and transient tables do not.
</details>

---

### Question 131
What are two ways to create and manage Data Shares in Snowflake? (Choose two.)

-  A. Via the Snowflake Web Interface (Snowsight)
-  B. Via a session parameter
-  C. Via SQL commands
-  D. Via Virtual Warehouses

<details><summary>Show Answer</summary>

**A** and **C** — through Snowsight or via SQL (`CREATE SHARE`, `GRANT ... TO SHARE`, et-  C.).
</details>

---

### Question 132
**True or False:** Time Travel can be completely disabled for a Snowflake account.

<details><summary>Show Answer</summary>

**False.** Time Travel cannot be turned off entirely. You can set `DATA_RETENTION_TIME_IN_DAYS = 0` at the account level (which effectively minimizes it for new objects), but the feature itself, and Fail-safe, cannot be disabled outright.
</details>

---

### Question 133
**True or False:** It is possible for a user to run a query against the query result cache without requiring an active Warehouse.

<details><summary>Show Answer</summary>

**True.** The result cache is served by the Cloud Services layer, so a **running warehouse is not required** to retrieve a previously cached result.
</details>

---

### Question 134
**True or False:** When Snowflake is configured to use Single Sign-On (SSO), Snowflake receives the usernames and credentials from the SSO service and loads them into the customer's Snowflake account.

<details><summary>Show Answer</summary>

**False.** In federated authentication, Snowflake never receives or stores the user's IdP credentials — only a signed SAML assertion confirming successful authentication.
</details>

---

### Question 135
Which of the following are best practices for loading data into Snowflake? (Choose three.)

-  A. Aim to produce compressed data files in the 100–250 MB range
-  B. Load data from a cloud storage service in a different region/platform than your Snowflake account, to save on cost
-  C. Enclose fields that contain delimiter characters in single or double quotes
-  D. Split large files into a greater number of smaller files to better distribute the load across compute resources
-  E. When planning warehouse size for loading, start with the largest warehouse possible
-  F. Partition staged data into large folders with random paths, letting Snowflake determine the best load strategy

<details><summary>Show Answer</summary>

**A, C, -  D.**
</details>

---

### Question 136
Which feature is used both for querying and for restoring data?

-  A. Clustering keys
-  B. Time Travel
-  C. Fail-safe
-  D. Cloning

<details><summary>Show Answer</summary>

**-  B. Time Travel** — you can both query historical data (`AT`/`BEFORE`) and restore dropped objects (`UNDROP`) with it. Fail-safe is restore-only (and only by Snowflake Support), not queryable by users.
</details>

---

### Question 137
What do the terms "scale up" and "scale out" refer to in Snowflake? (Choose two.)

-  A. Scaling out adds clusters of the same size to a virtual warehouse to handle more concurrent queries
-  B. Scaling out adds clusters of varying sizes to a virtual warehouse
-  C. Scaling out adds additional database servers to an existing running cluster
-  D. Snowflake recommends using both scaling up and scaling out together to handle more concurrent queries
-  E. Scaling up resizes a virtual warehouse so it can handle more complex workloads
-  F. Scaling up adds additional database servers to an existing running cluster

<details><summary>Show Answer</summary>

**A** and **-  E.**
</details>

---

### Question 138
What is the minimum Snowflake edition that has column-level security enabled?

-  A. Standard
-  B. Enterprise
-  C. Business Critical
-  D. Virtual Private Snowflake

<details><summary>Show Answer</summary>

**-  B. Enterprise** (or higher). Confirmed current in Snowflake's edition documentation.
</details>

---

### Question 139
What parameter controls whether a virtual warehouse starts immediately after the `CREATE WAREHOUSE` statement runs?

-  A. `INITIALLY_SUSPENDED = TRUE | FALSE`
-  B. `AUTO_RESUME = TRUE | FALSE`
-  C. `START_TIME = 60` (seconds from now)
-  D. `START_TIME = CURRENT_DATE()`

<details><summary>Show Answer</summary>

**-  A. `INITIALLY_SUSPENDED`.**
</details>

---

### Question 140
When cloning a database, what is cloned with it? (Choose two.)

-  A. Privileges granted **on** the database object itself
-  B. Existing child objects within the database
-  C. Future child objects (created after the clone) within the database
-  D. Privileges on the schemas/objects **within** the database
-  E. Only schemas and tables (no other object types)

<details><summary>Show Answer</summary>

**B** and **-  D.** Existing child objects are copied into the clone, and grants that exist on those child objects carry over — but privileges granted directly **on** the database object itself are not copied, and future objects obviously aren't included since they didn't exist yet.
</details>

---

### Question 141
Which of the following describes the Snowflake Cloud Services layer?

-  A. Coordinates activities across the Snowflake account (authentication, metadata, optimization, et-  C.)
-  B. Executes queries submitted by Snowflake users
-  C. Manages quotas on Snowflake account storage
-  D. Manages the virtual warehouse cache to speed up queries

<details><summary>Show Answer</summary>

**-  A.** The Cloud Services layer coordinates the platform (auth, metadata, query parsing/optimization, security) — it does **not** execute queries (that's compute) or manage the local warehouse cache.
</details>

---

### Question 142
What is the maximum total Continuous Data Protection (CDP) time incurred for a temporary table?

-  A. 30 days
-  B. 7 days
-  C. 48 hours
-  D. 24 hours

<details><summary>Show Answer</summary>

**-  D. 24 hours.** Temporary tables get up to 1 day of Time Travel and no Fail-safe, since they don't persist beyond the session/24 hours.
</details>

---

### Question 143
When reviewing a Query Profile, what is a symptom that a query is too large to fit into memory?

-  A. A single join node uses more than [X]% of query time
-  B. Partitions scanned equals partitions total
-  C. An Aggregate operator node is present
-  D. The query is spilling to local or remote storage

<details><summary>Show Answer</summary>

**-  D. The query is spilling to storage.** Spilling (especially to remote/cloud storage) indicates the warehouse doesn't have enough memory for the operation and is a classic sign to resize up.
</details>

---

### Question 144
What type of query benefits the **most** from Search Optimization?

-  A. A query using only disjunction (OR) predicates
-  B. A query that includes analytical expressions
-  C. A query that uses equality predicates or predicates using `IN`
-  D. A query that filters on semi-structured data types

<details><summary>Show Answer</summary>

**-  C.** Search Optimization is designed for point-lookup queries — equality and `IN` predicates on high-cardinality columns.
</details>

---

### Question 145
What transformations are supported in a `CREATE PIPE AS COPY FROM (SELECT ...)` statement? (Choose two.)

-  A. Data can be filtered by an optional `WHERE` clause
-  B. Incoming data can be joined with other tables
-  C. Columns can be reordered
-  D. Columns can be omitted
-  E. Row-level access can be defined

<details><summary>Show Answer</summary>

**C** and **-  D.** Snowpipe's `COPY` transformation supports column reordering, casting, and omission — but not joins, filters, or row access policies during the copy.
</details>

---

### Question 146
Which of the following are characteristics of Snowflake virtual warehouses? (Choose two.)

-  A. Auto-suspend applies only to the last-started warehouse in a multi-cluster warehouse
-  B. The ability to auto-suspend is only available in Enterprise Edition or above
-  C. SnowSQL supports both a configuration file and a command-line option for specifying a default warehouse
-  D. A user cannot specify a default warehouse when using the ODBC driver
-  E. The default virtual warehouse size can be changed at any time

<details><summary>Show Answer</summary>

**C** and **-  E.**
</details>

---

### Question 147
Which command should be used to load data from a file located in an external stage into a table in Snowflake?

-  A. `INSERT`
-  B. `PUT`
-  C. `GET`
-  D. `COPY INTO`

<details><summary>Show Answer</summary>

**-  D. `COPY INTO <table>`.**
</details>

---

### Question 148
The Snowflake Data Cloud platform is described as having which of the following architectures?

-  A. Shared-disk
-  B. Shared-nothing
-  C. Multi-cluster, shared data
-  D. Serverless query engine

<details><summary>Show Answer</summary>

**-  C. Multi-cluster, shared data architecture.**
</details>

---

### Question 149
Which of the following is a data tokenization integration partner?

-  A. Protegrity
-  B. Tableau

<details><summary>Show Answer</summary>

**-  A. Protegrity.**
</details>

---

### Question 150
Which editions of Snowflake are commonly used to help manage compliance with Personal Identifiable Information (PII) requirements? (Choose two.)

-  A. Custom Edition
-  B. Virtual Private Snowflake
-  C. Business Critical Edition
-  D. Standard Edition
-  E. Enterprise Edition

<details><summary>Show Answer</summary>

**B** and **C** — Virtual Private Snowflake and Business Critical Edition provide the enhanced data protection features most relevant to sensitive/PII data compliance requirements.
</details>

---

### Question 151
What are supported file formats for **unloading** data from Snowflake? (Choose three.)

-  A. XML
-  B. JSON
-  C. Parquet
-  D. ORC
-  E. Avro
-  F. CSV

<details><summary>Show Answer</summary>

**B, C, F — JSON, Parquet, and CSV.**

**⚠ Updated:** The original source listed the answer as JSON/Parquet/Avro. Per current Snowflake documentation, `COPY INTO <location>` only supports **delimited (CSV/TSV), JSON, and Parquet** for unloading. XML, ORC, and Avro are **load-only** formats and cannot be used to unload dat-  A.
</details>

---

### Question 152
The Snowflake Cloud Services layer is responsible for which two of the following tasks?

-  A. Local disk caching
-  B. Authentication and access control
-  C. Metadata management
-  D. Query processing (execution)
-  E. Database storage

<details><summary>Show Answer</summary>

**B** and **-  C.**
</details>

---

### Question 153
What is a key feature of Snowflake's architecture?

-  A. Zero-copy cloning creates a mirror copy of a database that updates with the original
-  B. Software updates are automatically applied on a quarterly basis
-  C. Snowflake eliminates resource contention with its virtual warehouse implementation
-  D. Multi-cluster warehouses allow users to run a single query that spans across multiple clusters
-  E. Snowflake sorts data on ingest for fast retrieval by date

<details><summary>Show Answer</summary>

**-  C.** Because each virtual warehouse is an independent compute cluster operating on the same shared storage layer, workloads in one warehouse don't compete for resources with workloads in another.
</details>

---

### Question 154
When publishing a Snowflake Data Marketplace listing into a remote region, what should be taken into consideration? (Choose two.)

-  A. There is a need to have, in the target region, a share created for each consumer
-  B. The listing metadata is replicated into all selected regions automatically, but the underlying data is not replicated until requested
-  C. The user must have the ORGADMIN role in at least one account to link accounts for replication
-  D. Shares attached to listings in remote regions can be viewed from any account in the organization
-  E. For a standard listing, the provider can wait until the first customer requests the data before replicating it to the target region

<details><summary>Show Answer</summary>

**B** and **-  E.**
</details>

---

### Question 155
When loading data into Snowflake via Snowpipe, what is the recommended compressed file size?

-  A. 10–50 MB
-  B. 100–250 MB
-  C. 300–500 MB
-  D. 1000–1500 MB

<details><summary>Show Answer</summary>

**-  B. 100–250 M-  B.**
</details>

---

### Question 156
Which Snowflake feature allows a user to substitute a randomly generated identifier for sensitive data — to prevent unauthorized users from accessing the real data — **before** loading it into Snowflake?

-  A. External Tokenization
-  B. External Tables
-  C. Materialized Views
-  D. Table Functions (UDTFs)

<details><summary>Show Answer</summary>

**-  A. External Tokenization.**
</details>

---

### Question 157
Which of the following are examples of operations that require an active Virtual Warehouse to complete, assuming no queries have been executed previously (i.e., nothing is cached)? (Choose three.)

-  A. `MIN(<column>)`
-  B. `COPY`
-  C. `SUM(<column>)`
-  D. `UPDATE`

<details><summary>Show Answer</summary>

**B, C, -  D.** `COPY`, `SUM()`, and `UPDATE` all require compute. A simple `MIN()`/`MAX()` with no `WHERE` clause can sometimes be resolved directly from micro-partition metadata (which Snowflake maintains regardless of warehouse state), without needing an active warehouse.
</details>

---

### Question 158
What `SNOWFLAK-  E.ACCOUNT_USAGE` view contains information about which objects were read by queries within the last 365 days?

-  A. `VIEWS_HISTORY`
-  B. `OBJECT_HISTORY`
-  C. `ACCESS_HISTORY`
-  D. `LOGIN_HISTORY`

<details><summary>Show Answer</summary>

**-  C. `ACCESS_HISTORY`.**
</details>

---

### Question 159
Which feature is only available in the Enterprise Edition or higher?

-  A. Column-level security
-  B. SOC 2 Type II certification
-  C. Multi-factor Authentication (MFA)
-  D. Object-level access control

<details><summary>Show Answer</summary>

**-  A. Column-level security.** SOC 2 Type II, MFA, and object-level access control are available in all editions, including Standard.
</details>

---

### Question 160
Will data cached in a warehouse be lost when the warehouse is resized?

-  A. Possibly — if resized to a smaller size, the cache may no longer fit
-  B. Yes, because the compute resource is replaced in its entirety with a new compute resource
-  C. No, because the size of the cache is independent from the warehouse size
-  D. Yes, because the compute resource will no longer have access to the cache encryption key

<details><summary>Show Answer</summary>

**-  B.** Resizing a warehouse provisions new compute nodes, so the previous local disk cache is lost regardless of direction (larger or smaller).
</details>

---

### Question 161
Which semi-structured file formats are supported when **unloading** data from a table? (Choose two.)

-  A. ORC
-  B. XML
-  C. Avro
-  D. Parquet
-  E. JSON

<details><summary>Show Answer</summary>

**D** and **E — Parquet and JSON.** ORC, XML, and Avro are load-only formats.
</details>

---

### Question 162
A running virtual warehouse is suspended, then restarted. What is the **minimum** amount of time that the warehouse will be billed for upon restart?

-  A. 1 second
-  B. 60 seconds
-  C. 5 minutes
-  D. 60 minutes

<details><summary>Show Answer</summary>

**-  B. 60 seconds.** Per-second billing kicks in after the first minute.
</details>

---

### Question 163
What are the responsibilities of Snowflake's Cloud Services layer? (Choose three.)

-  A. Authentication
-  B. Resource management
-  C. Virtual warehouse local disk caching
-  D. Query parsing and optimization
-  E. Query execution
-  F. Physical storage of micro-partitions

<details><summary>Show Answer</summary>

**A, B, -  D.** Query execution (E) happens in the compute layer, and warehouse-local caching (C) and micro-partition storage (F) belong to the compute and storage layers respectively — not Cloud Services.
</details>

---

### Question 164
How long is the Fail-safe period for temporary and transient tables?

-  A. There is no Fail-safe period for these tables
-  B. 1 day
-  C. 14 days
-  D. 31 days
-  E. 90 days

<details><summary>Show Answer</summary>

**-  A. No Fail-safe period.** Only permanent tables have Fail-safe.
</details>

---

### Question 165
Which command should be used to download files from a Snowflake stage to a local folder on a client machine?

-  A. `PUT`
-  B. `GET`
-  C. `COPY INTO`
-  D. `SELECT`

<details><summary>Show Answer</summary>

**-  B. `GET`.**
</details>

---

### Question 166
How does Snowflake Fail-safe protect data in a table?

-  A. Fail-safe makes data available for up to 1 day, recoverable by user operations
-  B. Fail-safe makes data available for 7 days, recoverable by user operations
-  C. Fail-safe makes data available for 7 days, recoverable only by Snowflake Support
-  D. Fail-safe makes data available for up to 1 day, recoverable only by Snowflake Support

<details><summary>Show Answer</summary>

**-  C.** Fail-safe is a non-configurable, 7-day, disaster-recovery-only mechanism that requires contacting Snowflake Support — end users cannot self-service recover from it.
</details>

---

### Question 167
A virtual warehouse is created:
```sql
CREATE WAREHOUSE my_wh WITH
  WAREHOUSE_SIZE = MEDIUM
  AUTO_SUSPEND = 60
  AUTO_RESUME = TRUE;
```
Its utilization graph over two days shows frequent, spiky bursts of concurrent activity throughout the day. What action should be taken to address this situation?

-  A. Increase the warehouse size from Medium to 2X-Large
-  B. Increase the value of `AUTO_SUSPEND`
-  C. Configure the warehouse as a multi-cluster warehouse
-  D. Lower the value of `AUTO_SUSPEND`

<details><summary>Show Answer</summary>

**-  C.** Bursty, concurrent workload patterns are best handled by **multi-cluster warehouses**, which spin additional clusters up/down automatically to absorb concurrency spikes — resizing (A) helps single-query performance, not concurrency.
</details>

---

### Question 168
Which minimum Snowflake edition provides a fully dedicated, isolated environment (including a dedicated metadata/cloud services layer not shared with other accounts)?

-  A. Standard
-  B. Enterprise
-  C. Business Critical
-  D. Virtual Private Snowflake

<details><summary>Show Answer</summary>

**-  D. Virtual Private Snowflake (VPS).** VPS is Snowflake's highest tier, offering a completely separate environment with no shared hardware/resources with accounts outside the VPS.
</details>

---

### Question 169
Network policies can be set at which Snowflake levels? (Choose two.)

-  A. Role
-  B. Schema
-  C. User
-  D. Database
-  E. Account
-  F. Table

<details><summary>Show Answer</summary>

**C** and **E** — User and Account levels (network policies can also be attached to security integrations in current Snowflake, but of the options given, User and Account are correct).
</details>

---

### Question 170
What are the correct default parameters for Time Travel and Fail-safe in Snowflake **Enterprise Edition**?

-  A. Default Time Travel = 0 days, Max Time Travel = 30 days, Fail-safe = 1 day
-  B. Default Time Travel = 1 day, Max Time Travel = 365 days, Fail-safe = 7 days
-  C. Default Time Travel = 0 days, Max Time Travel = 90 days, Fail-safe = 7 days
-  D. Default Time Travel = 1 day, Max Time Travel = 90 days, Fail-safe = 7 days
-  E. Default Time Travel = 7 days, Max Time Travel = 1 day, Fail-safe = 90 days

<details><summary>Show Answer</summary>

**-  D.** Default Time Travel retention is 1 day, extendable up to a maximum of 90 days with Enterprise Edition or higher, and Fail-safe is a fixed 7 days. Confirmed current against Snowflake's Time Travel documentation.
</details>

---

### Question 171
Which of the following objects are contained within a schema? (Choose two.)

-  A. Role
-  B. Table
-  C. Warehouse
-  D. External table
-  E. User
-  F. Share

<details><summary>Show Answer</summary>

**B** and **D** — Tables and External Tables. Roles, warehouses, users, and shares are all account-level objects, not schema-level objects.
</details>

---

### Question 172
Which of the following statements describe features of Snowflake data caching? (Choose two.)

-  A. When a virtual warehouse is suspended, its local disk data cache is saved to remote storage
-  B. When the data cache is full, the least-recently-used data is cleared to make room
-  C. A user can only access their own queries from the query result cache
-  D. A user must set a parameter to `TRUE` to enable the metadata cache
-  E. The `RESULT_SCAN` table function can access and filter the contents of the query result cache

<details><summary>Show Answer</summary>

**B** and **-  E.**
</details>

---

### Question 173
A table needs to be loaded. The input data is in JSON format, consisting of a concatenation of multiple JSON documents. The file is 3 GB, and an X-Small warehouse is being used with:
```sql
COPY INTO sample FROM @stage FILE_FORMAT = (TYPE = JSON)
```
The load fails with:
```
Max LOB size (16777216) exceeded. Actual size of parsed column is 17894470.
```
How can this issue be resolved?

-  A. Compress the file before loading it
-  B. Split the file into multiple files in the recommended 100–250 MB size range
-  C. Use a larger-sized warehouse
-  D. Set `STRIP_OUTER_ARRAY = TRUE` in the `COPY INTO` command

<details><summary>Show Answer</summary>

**-  D.** The error means a single parsed VARIANT value exceeds the 16 MB limit — this happens when multiple JSON documents are wrapped/concatenated into one oversized structure. `STRIP_OUTER_ARRAY = TRUE` breaks the outer array into individual rows so each parsed value stays under the limit.
</details>

---

### Question 174
What is a feature of a stored procedure in Snowflake?

-  A. They can access secured metadata across all databases regardless of role
-  B. They can only access tables from a single database
-  C. They can only contain a single SQL statement
-  D. They can be created to run with either the caller's rights or the owner's rights

<details><summary>Show Answer</summary>

**-  D.** Stored procedures support both **caller's rights** and **owner's rights** execution contexts.
</details>

---

### Question 175
Which columns are part of the result set of the `LATERAL FLATTEN` command? (Choose two.)

-  A. `CONTENT`
-  B. `PATH`
-  C. `BYTE_SIZE`
-  D. `INDEX`
-  E. `DATATYPE`

<details><summary>Show Answer</summary>

**B** and **D** — `PATH` and `INDEX`. The full `FLATTEN` output includes `SEQ`, `KEY`, `PATH`, `INDEX`, `VALUE`, and `THIS` — not `CONTENT`, `BYTE_SIZE`, or `DATATYPE`.
</details>

---

### Question 176
What is the minimum edition required to create a materialized view?

-  A. Standard Edition
-  B. Enterprise Edition
-  C. Business Critical Edition
-  D. Virtual Private Snowflake Edition

<details><summary>Show Answer</summary>

**-  B. Enterprise Edition** (or higher). Confirmed current.
</details>

---

### Question 177
Which Snowflake function interprets an input string as a JSON document and produces a VARIANT value?

<details><summary>Show Answer</summary>

**`PARSE_JSON`.**
</details>

---

### Question 178
How are serverless features generally billed?

-  A. Per second, multiplied by an automatic sizing determined for the job
-  B. Per minute, multiplied by an automatic sizing, with a minimum of one minute
-  C. Per second, multiplied by a fixed size set by a parameter
-  D. Serverless features are not billed unless the total monthly cost exceeds a set percentage of warehouse credits

<details><summary>Show Answer</summary>

**-  A.** Most serverless features (Automatic Clustering, Search Optimization, Query Acceleration, et-  C.) bill per-second based on compute that Snowflake automatically sizes for the job — with no fixed minimum.

**Note:** Snowpipe specifically switched to a flat **per-GB** pricing model as of December 2025 (see Question 105), so it's now an exception to this general per-second serverless billing pattern.
</details>

---

### Question 179
Which Snowflake architectural layer is responsible for generating a query execution plan?

-  A. Compute
-  B. Data storage
-  C. Cloud Services
-  D. Cloud provider

<details><summary>Show Answer</summary>

**-  C. Cloud Services.** Query parsing and optimization happen here before execution is handed off to the compute (warehouse) layer.
</details>

---

### Question 180
When unloading data to a stage, which of the following is a recommended practice?

-  A. Set `SINGLE = TRUE` for larger files
-  B. Use headers when unloading with Parquet
-  C. Avoid the use of the `CAST` function
-  D. Define an individual, explicit file format

<details><summary>Show Answer</summary>

**-  D. Define an individual file format** rather than relying on defaults, so unload behavior (compression, delimiters, headers, et-  C.) is explicit and predictable.
</details>

---

### Question 181
Which SQL commands, when committed, will consume a stream and advance its offset? (Choose two.)

-  A. `UPDATE ... FROM STREAM`
-  B. `SELECT * FROM STREAM`
-  C. `INSERT INTO table SELECT * FROM STREAM`
-  D. `ALTER TABLE ... AS SELECT FROM STREAM`
-  E. `BEGIN ... COMMIT` (empty transaction)

<details><summary>Show Answer</summary>

**A** and **-  C.** A DML statement that references the stream as its source and is committed advances the stream's offset. A plain `SELECT` does not consume the stream.
</details>

---

### Question 182
Which methods can be used to delete staged files from a Snowflake stage? (Choose two.)

-  A. Use the `DROP FILE` command after the load completes
-  B. Specify a purge option when creating the file format
-  C. Specify the `PURGE` copy option in the `COPY INTO <table>` command
-  D. Use the `REMOVE` command after the load completes
-  E. Use a `DELETE LOAD HISTORY` command after the load completes

<details><summary>Show Answer</summary>

**C** and **-  D.**
</details>

---

### Question 183
On which of the following cloud platforms can a Snowflake account be hosted? (Choose three.)

-  A. Amazon Web Services
-  B. Private Virtual Cloud
-  C. Oracle Cloud
-  D. Microsoft Azure
-  E. Google Cloud Platform
-  F. Alibaba Cloud

<details><summary>Show Answer</summary>

**A, D, E — AWS, Microsoft Azure, and Google Cloud Platform.**
</details>

---

### Question 184
What Snowflake role must be granted for a user to create and manage additional Snowflake **accounts**?

-  A. ACCOUNTADMIN
-  B. ORGADMIN
-  C. SECURITYADMIN
-  D. SYSADMIN

<details><summary>Show Answer</summary>

**-  B. ORGADMIN.** This role manages operations at the organization level, including creating new accounts.
</details>

---

### Question 185
Assume a table consists of five micro-partitions with values ranging from A to Z. Which layout indicates a **well-clustered** table?

<details><summary>Show Answer</summary>

A well-clustered table is one where each micro-partition contains a **narrow, largely non-overlapping** range of the clustering key's values (e.g., partition 1 = A–E, partition 2 = F–J, et-  C.), rather than every partition containing values scattered across the full A–Z range. Narrow, non-overlapping ranges allow Snowflake to prune (skip) most partitions when a query filters on the clustering key.

*(Note: the original source referenced a diagram that wasn't legible/reproducible from the source material — the concept above is what the correct diagram choice represents.)*
</details>

---

### Question 186
What feature can be used to reorganize a very large table on one or more columns to improve pruning?

-  A. Micro-partitions
-  B. Clustering keys
-  C. Key partitions
-  D. Clustered partitions

<details><summary>Show Answer</summary>

**-  B. Clustering keys.**
</details>

---

### Question 187
What is an advantage of using an Explain Plan instead of the Query Profiler to evaluate query performance?

-  A. The plan output is available graphically
-  B. An Explain Plan can be used to analyze performance **without executing** the query
-  C. An Explain Plan handles queries with temporary tables while the Query Profiler will not
-  D. An Explain Plan's output displays automatic data-skew optimization information

<details><summary>Show Answer</summary>

**-  B.** `EXPLAIN` shows the planned execution path without actually running (and paying for) the query.
</details>

---

### Question 188
Which data types are supported by Snowflake for semi-structured data? (Choose two.)

-  A. VARIANT
-  B. VARRAY
-  C. STRUCT
-  D. ARRAY
-  E. QUEUE

<details><summary>Show Answer</summary>

**A** and **D — VARIANT and ARRAY.** (OBJECT is the third semi-structured type, but wasn't offered here.) VARRAY, STRUCT, and QUEUE are not Snowflake semi-structured data types.
</details>

---

### Question 189
Why does Snowflake recommend file sizes of 100–250 MB compressed when loading data?

-  A. Optimizes the virtual warehouse's multi-cluster setting to economy mode
-  B. Allows a user to import files in a strictly sequential order
-  C. Increases latency during staging and accuracy when loading data
-  D. Allows optimization of parallel operations

<details><summary>Show Answer</summary>

**-  D.** This file size range lets Snowflake distribute the load efficiently across all available compute threads/nodes for maximum load parallelism.
</details>

---

### Question 190
Which of the following features are available with the Snowflake **Enterprise** edition? (Choose two.)

-  A. Database replication and failover
-  B. Automated index management
-  C. Customer-managed encryption keys (Tri-Secret Secure)
-  D. Extended Time Travel (up to 90 days)
-  E. Native support for geospatial data

<details><summary>Show Answer</summary>

**A** and **D — Database replication/failover, and Extended Time Travel.**

**⚠ Updated:** The original source listed D and E as correct. That's outdated/incorrect:
- **Geospatial data support (E) is available in all editions**, including Standard — it is not Enterprise-exclusive.
- **Snowflake has no concept of "indexes"** (option B doesn't exist as a real feature), so it's a distractor.
- **Tri-Secret Secure / customer-managed keys (C) require Business Critical Edition**, not Enterprise.
- Extended Time Travel up to 90 days and cross-account database replication/failover are genuinely Enterprise-tier features, so **A and D** are correct.
</details>

---

### Question 191
What is the default file size limit when unloading data from Snowflake using the `COPY INTO` command?

-  A. 1 MB
-  B. 8 GB
-  C. 16 MB
-  D. 32 MB

<details><summary>Show Answer</summary>

**-  C. 16 MB** (per unloaded file, unless `MAX_FILE_SIZE` is set otherwise).
</details>

---

### Question 192
What features that are part of the Continuous Data Protection (CDP) feature set do **not require additional configuration**? (Choose two.)

-  A. Row access policies
-  B. Data masking policies
-  C. Data encryption
-  D. Time Travel
-  E. External tokenization

<details><summary>Show Answer</summary>

**C** and **D — Data encryption and Time Travel.** Both are automatically on for every account/object with no setup required. Masking policies, row access policies, and external tokenization all require explicit configuration by an administrator.
</details>

---

### Question 193
Which Snowflake layer is always leveraged when accessing a query from the result cache?

-  A. Metadata
-  B. Data Storage
-  C. Compute
-  D. Cloud Services

<details><summary>Show Answer</summary>

**-  D. Cloud Services.** The result cache is managed and served by the Cloud Services layer, which is why it can be used without an active warehouse.
</details>

---

### Question 194
Which connectors are available in the downloads section of the Snowflake web interface? (Choose two.)

-  A. SnowSQL
-  B. JDBC
-  C. ODBC
-  D. Hive
-  E. Scala

<details><summary>Show Answer</summary>

**A** and **C — SnowSQL and ODB-  C.**
</details>

---

### Question 195
A Snowflake Administrator needs to ensure that sensitive corporate data in Snowflake tables is not visible to end users, but is partially visible to functional managers. How can this requirement be met?

-  A. Use data encryption
-  B. Use dynamic data masking
-  C. Use secure materialized views
-  D. Revoke all roles for functional managers and end users

<details><summary>Show Answer</summary>

**-  B. Dynamic data masking.** A masking policy can be written to reveal full, partial, or masked values depending on the querying role.
</details>

---

### Question 196
Users are responsible for data storage costs until what occurs?

-  A. Data expires from Time Travel
-  B. Data expires from Fail-safe
-  C. Data is deleted from a table
-  D. Data is truncated from a table

<details><summary>Show Answer</summary>

**-  B. Data expires from Fail-safe.** Storage costs continue to accrue as long as the historical data is retained anywhere — including through the entire Time Travel *and* the subsequent 7-day Fail-safe period.
</details>

---

### Question 197
A user has an application that writes a new file to a cloud storage location every 5 minutes. What is the **most efficient** way to get these files into Snowflake?

-  A. Create a task that runs a `COPY INTO` from an external stage every 5 minutes
-  B. Create a task that `PUT`s the files into an internal stage and automates the data load
-  C. Create a task that runs a `GET` operation to intermittently check for new files
-  D. Set up cloud provider event notifications on the storage location and use Snowpipe with auto-ingest

<details><summary>Show Answer</summary>

**-  D.** Snowpipe with event-based auto-ingest is designed exactly for this near-real-time, continuous ingestion pattern — it avoids the overhead and latency of a fixed polling schedule.
</details>

---

### Question 198
What affects whether the query result cache can be used?

-  A. Whether the query contains a deterministic function
-  B. Whether the virtual warehouse has been suspended
-  C. Whether the referenced data in the table has changed
-  D. Whether multiple users are using the same virtual warehouse

<details><summary>Show Answer</summary>

**-  C.** If the underlying table's data changed since the result was cached, the cache is invalidated for that query.
</details>

---

### Question 199
Which of the following is an example of an operation that can be completed **without** requiring compute, assuming no queries have been executed previously?

-  A. `SELECT AVG(ORDER_AMT) FROM SALES`
-  B. `SELECT * FROM SALES`
-  C. `SELECT MIN(ORDER_AMT) FROM SALES`
-  D. `SELECT ORDER_AMT, ORDER_QTY FROM SALES`

<details><summary>Show Answer</summary>

**-  C.** A simple, unfiltered `MIN()`/`MAX()` can be answered directly from micro-partition metadata that Snowflake already maintains, without spinning up compute — unlike `AVG()`, `SELECT *`, or multi-column projections, which require scanning actual dat-  A.
</details>

---

### Question 200
How many days is Snowpipe load history retained?

-  A. 1 day
-  B. 30 days
-  C. 14 days
-  D. 60 days

<details><summary>Show Answer</summary>

**-  C. 14 days.**
</details>

---

## Summary of Corrections Made vs. the Original Source

| Question | Change |
|---|---|
| 105 | Clarified that Snowpipe billing moved from per-second/per-core granularity to **flat per-GB pricing** as of Dec 8, 2025. |
| 151 | Corrected unload-supported formats to **JSON, Parquet, CSV** (removed Avro, which is load-only). |
| 161 | Confirmed only **Parquet and JSON** are valid semi-structured unload formats (not ORC/XML/Avro). |
| 178 | Clarified this general serverless billing rule now has an exception for Snowpipe's new flat-rate model. |
| 190 | Corrected Enterprise-exclusive features to **Database replication/failover + Extended Time Travel**, removing "native geospatial support" (available in *all* editions, not Enterprise-only) and the non-existent "automated index management." |

All other answers were checked against current Snowflake documentation (as of July 2026) and found to still be accurate.

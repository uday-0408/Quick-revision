# SnowPro Core Practice Questions (102–200)

*Cleaned, reformatted, and cross-checked against Snowflake documentation as of mid‑2026. Corrections to the original answer key are flagged with ⚠ Updated. A few items in the source text were too garbled by OCR to reconstruct with confidence — these are noted honestly rather than guessed.*

---

### Question 102
True or False: Snowflake bills for a minimum of five minutes each time a Virtual Warehouse is started.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False

⚠ Updated: The minimum billing increment for a warehouse is **60 seconds**, not five minutes. After the first 60 seconds, billing is per-second.
</details>

---

### Question 103
When scaling up Virtual Warehouses by increasing the Virtual Warehouse t-shirt size, you are primarily scaling for improved:

-  A.Concurrency
-  B.Performance

<details><summary>Show Answer</summary>
Correct Answer: -  B.Performance — scaling up (bigger warehouse) speeds up individual queries. Scaling out (multi-cluster) is what improves concurrency.
</details>

---

### Question 104
As a practice, clustering keys should be defined on tables of which minimum size?

-  A.Multi-Kilobyte (KB) range
-  B.Multi-Megabyte (MB) range
-  C.Multi-Gigabyte (GB) range
-  D.Multi-Terabyte (TB) range

<details><summary>Show Answer</summary>
Correct Answer: -  D.Multi-Terabyte (TB) range
</details>

---

### Question 105
How are Snowpipe charges calculated?

-  A.Per-second, per warehouse size
-  B.Per-second, per-core granularity
-  C.Number of pipes in the account
-  D.Total storage bucket size

<details><summary>Show Answer</summary>
Correct Answer: -  B.Snowpipe uses Snowflake-managed serverless compute, billed per-second at per-core granularity based on resources actually consumed.
</details>

---

### Question 106
True or False: A Snowflake account is charged for data stored in both internal and external stages.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False — Snowflake only bills storage for internal stages. External stage storage is billed by the cloud storage provider (S3/Azure Blob/GCS), not Snowflake.
</details>

---

### Question 107
True or False: When active, a Pipe uses a dedicated Virtual Warehouse.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False — Snowpipe uses Snowflake-supplied serverless compute resources, not a customer-managed virtual warehouse.
</details>

---

### Question 108
True or False: Snowflake supports federated authentication in all editions.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  A.True — federated authentication/SSO became a baseline feature in all Snowflake editions (including Standard) as of March 2019, and remains so.
</details>

---

### Question 109
True or False: When a new Snowflake object is created, it is automatically owned by the user who created it.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False — object ownership is assigned to the **role** used to create the object, not the individual user.
</details>

---

### Question 110
True or False: A Virtual Warehouse consumes Snowflake credits even when inactive.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False — credits are only consumed while the warehouse is actually running (active or auto-resumed).
</details>

---

### Question 111
True or False: During data unloading, only JSON and CSV files can be compressed.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False — Parquet files are also compressed (Snappy by default) when unloaded, in addition to CSV/JSON (gzip and other algorithms).
</details>

---

### Question 112
Which of the following are options when creating a Virtual Warehouse? (Choose two.)

-  A.Auto-suspend
-  B.Auto-resume
-  C.Local SSD size
-  D.User count

<details><summary>Show Answer</summary>
Correct Answer: A, -  B.Auto-suspend and auto-resume are configurable warehouse properties. Local disk (cache) size and user count are not user-configurable warehouse parameters.
</details>

---

### Question 113
Which formats are supported for unloading data from Snowflake? (Choose two.)

-  A.Delimited (CSV, TSV, etc.)
-  B.Avro
-  C.JSON
-  D.ORC

<details><summary>Show Answer</summary>
Correct Answer: A, -  C.Delimited and JSON. (Parquet is also unload-capable but wasn't offered as a distinct correct pairing here.) Avro and ORC are **load-only** formats — Snowflake cannot unload to them.
</details>

---

### Question 114
True or False: Data Providers can share data with only the Data Consumer.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False — providers can share with specific consumer accounts, publish to the Snowflake Marketplace, or grant access via Reader Accounts.
</details>

---

### Question 115
The fail-safe retention period is how many days?

-  A.1 day
-  B.7 days
-  C.45 days
-  D.90 days

<details><summary>Show Answer</summary>
Correct Answer: -  B.7 days (for permanent tables; not applicable to temporary/transient tables).
</details>

---

### Question 116
True or False: Once created, a micro-partition will never be changed.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  A.True — micro-partitions are immutable. DML operations create new micro-partitions rather than modifying existing ones in place.
</details>

---

### Question 117
What services does Snowflake automatically provide for customers that they may have been responsible for with their on-premise system? (Choose all that apply.)

-  A.Installing and configuring hardware
-  B.Patching software
-  C.Physical security
-  D.Maintaining metadata and statistics

<details><summary>Show Answer</summary>
Correct Answer: A, B, -  D.As a fully managed SaaS platform, Snowflake handles hardware provisioning, software patching, and metadata/statistics maintenance. (Physical data-center security is technically the responsibility of the underlying cloud provider, AWS/Azure/GCP, rather than Snowflake directly — hence its exclusion here.)
</details>

---

### Question 118
Which of the following statements would be used to export/unload data from Snowflake?

-  A.`COPY INTO @stage ...`
-  B.`EXPORT TO @stage ...`
-  C.`INSERT INTO @stage ...`
-  D.`UNLOAD ... TO @stage ...`

<details><summary>Show Answer</summary>
Correct Answer: -  A.`COPY INTO <location>` is the command used to unload table data to a stage.
</details>

---

### Question 119
True or False: A 4X-Large Warehouse may, at times, take longer to provision than an X-Small Warehouse.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  A.True — larger warehouses require more compute resources to be provisioned, which can take longer, especially under high cloud-provider demand.
</details>

---

### Question 120
How would you determine the size of the virtual warehouse used for a task?

-  A.Since a root task may execute concurrently (multiple instances), leave margin in the execution window to avoid missed runs.
-  B.Query (SELECT) the size of the stream content to help determine warehouse size — larger stream content may need a larger warehouse.
-  C.If using a stored procedure to execute multiple SQL statements, it's best to test the procedure separately first to size the compute resource.
-  D.Configure the warehouse for automatic concurrency handling via a multi-cluster warehouse to match the task schedule.

<details><summary>Show Answer</summary>
Correct Answer: -  C.Test the stored procedure's workload independently to determine appropriate warehouse sizing before wiring it into a task.
</details>

---

### Question 121
The Information Schema and Account Usage share provide storage information for which of the following objects? (Choose three.)

-  A.Users
-  B.Tables
-  C.Databases
-  D.Internal Stages

<details><summary>Show Answer</summary>
Correct Answer: B, C, -  D.Table, database, and internal-stage storage metrics are all exposed via `INFORMATION_SCHEMA` / `ACCOUNT_USAGE` views (e.g., `TABLE_STORAGE_METRICS`, `DATABASE_STORAGE_USAGE_HISTORY`, `STAGE_STORAGE_USAGE_HISTORY`).
</details>

---

### Question 122
What is the default file format used in the `COPY` command if one is not specified?

-  A.CSV
-  B.JSON
-  C.Parquet
-  D.XML

<details><summary>Show Answer</summary>
Correct Answer: -  A.CSV
</details>

---

### Question 123
True or False: Reader Accounts are able to extract data from shared data objects for use outside of Snowflake.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False — Reader Accounts can only query shared data inside Snowflake; they cannot unload/export it externally.
</details>

---

### Question 124
True or False: You can define multiple columns within a clustering key on a table.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  A.True
</details>

---

### Question 125
True or False: Snowflake enforces unique, primary key, and foreign key constraints during DML operations.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False — these constraints are informational only (metadata/documentation), except for `NOT NULL`, which **is** enforced.
</details>

---

### Question 126
True or False: Loading data into Snowflake requires that source data files be no larger than 16MB.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False — there is no such hard limit on source file size for loading (though 100–250MB compressed is the performance-recommended range). The 16MB figure refers to the maximum size of a single VARIANT/semi-structured value.
</details>

---

### Question 127
True or False: A Virtual Warehouse can be resized while suspended.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  A.True
</details>

---

### Question 128
True or False: When you create a custom role, it is a best practice to immediately grant that role to ACCOUNTADMIN.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False — best practice is to build a role hierarchy where custom roles are granted up through `SYSADMIN`, keeping `ACCOUNTADMIN` reserved for account-level administration only.
</details>

---

### Question 129
Which of the following accurately represents how a table fits into Snowflake's logical container hierarchy?

-  A.Account → Table → Schema → Database
-  B.Account → Database → Schema → Table
-  C.Database → Table → Schema → Account
-  D.Table → Schema → Database → Account

<details><summary>Show Answer</summary>
Correct Answer: -  B.Account → Database → Schema → Table
</details>

---

### Question 130
True or False: All Snowflake table types include fail-safe storage.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False — only **permanent** tables have fail-safe. Temporary and transient tables have no fail-safe period.
</details>

---

### Question 131
What are two ways to create and manage Data Shares in Snowflake? (Choose two.)

-  A.Via the Snowsight web interface
-  B.Via a session parameter
-  C.Via SQL commands
-  D.Via Virtual Warehouses

<details><summary>Show Answer</summary>
Correct Answer: A, -  C.Shares can be created/managed through Snowsight or through SQL DDL (`CREATE SHARE`, `GRANT ... TO SHARE`, etc.).
</details>

---

### Question 132
*Source text too garbled to reliably reconstruct the statement being evaluated — the OCR only preserved fragments ("...be disabled within a Snowflake account..."). Rather than invent wording, this one is flagged for you to re-source from the original material if needed.*

---

### Question 133
True or False: It is possible for a user to run a query against the query result cache without requiring an active Warehouse.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  A.True — the result cache is served by the Cloud Services layer and doesn't require a running warehouse.
</details>

---

### Question 134
True or False: When Snowflake is configured to use Single Sign-On (SSO), Snowflake receives the usernames and credentials from the SSO service and loads them into the customer's Snowflake account.

-  A.True
-  B.False

<details><summary>Show Answer</summary>
Correct Answer: -  B.False — Snowflake receives an authentication assertion from the IdP; it does not import or store the IdP's credentials.
</details>

---

### Question 135
Which of the following are best practices for loading data into Snowflake? (Choose three.)

-  A.Aim to produce data files between 100 MB and 250 MB in size, compressed.
-  B.Load data from files in a cloud storage service in a different region/platform than the Snowflake account, to save on cost.
-  C.Enclose fields that contain delimiter characters in single or double quotes.
-  D.Split large files into a greater number of smaller files to distribute the load across compute resources in an active warehouse.
-  E.When choosing a warehouse for data loading, start with the largest warehouse possible.
-  F.Partition staged data into large folders with random paths, letting Snowflake determine the best way to load each file.

<details><summary>Show Answer</summary>
Correct Answer: A, C, D.
</details>

---

### Question 136
Which feature is used both for querying and restoring data?

-  A.Clustering keys
-  B.Time Travel
-  C.Fail-safe
-  D.Cloning

<details><summary>Show Answer</summary>
Correct Answer: -  B.Time Travel lets you both query historical data (`AT`/`BEFORE`) and restore objects (`UNDROP`, `CREATE ... CLONE ... AT`).
</details>

---

### Question 137
What do the terms "scale up" and "scale out" refer to in Snowflake? (Choose two.)

-  A.Scaling out adds clusters of the same size to a virtual warehouse to handle more concurrent queries.
-  B.Scaling out adds clusters of varying sizes to a virtual warehouse.
-  C.Scaling out adds additional database servers to an existing running cluster.
-  D.Snowflake recommends always using both scaling up and scaling out together.
-  E.Scaling up resizes a virtual warehouse so it can handle more complex workloads.
-  F.Scaling up adds additional database servers to an existing cluster.

<details><summary>Show Answer</summary>
Correct Answer: A, E.
</details>

---

### Question 138
What is the minimum Snowflake edition that has column-level security enabled?

-  A.Standard
-  B.Enterprise
-  C.Business Critical
-  D.Virtual Private Snowflake

<details><summary>Show Answer</summary>
Correct Answer: -  B.Enterprise
</details>

---

### Question 139
What parameter controls whether the Virtual Warehouse starts immediately after the `CREATE WAREHOUSE` statement?

-  A.`INITIALLY_SUSPENDED = TRUE|FALSE`
-  B.`AUTO_RESUME = TRUE|FALSE`
-  C.`START_TIME = 60`
-  D.`START_TIME = CURRENT_DATE()`

<details><summary>Show Answer</summary>
Correct Answer: -  A.`INITIALLY_SUSPENDED`
</details>

---

### Question 140
When cloning a database, what is cloned with the database? (Choose two.)

-  A.Privileges on the database itself
-  B.Existing child objects within the database
-  C.Future child objects within the database
-  D.Privileges on the schemas/objects within the database

<details><summary>Show Answer</summary>
Correct Answer: B, -  D.Cloning captures a point-in-time snapshot: only **existing** child objects (not future ones) are cloned, and the privileges granted on those child objects are generally preserved. The new top-level database itself does not inherit the source's own privilege grants (it's owned fresh by the cloning role).
</details>

---

### Question 141
Which of the following describes the Snowflake Cloud Services layer?

-  A.Coordinates activities across the Snowflake account (authentication, metadata, query parsing/optimization, etc.)
-  B.Executes queries submitted by Snowflake users
-  C.Manages quotas on Snowflake account storage
-  D.Manages the virtual warehouse cache to speed up queries

<details><summary>Show Answer</summary>
Correct Answer: -  A.Query execution (B) and local caching (D) belong to the Compute layer, not Cloud Services.
</details>

---

### Question 142
What is the maximum total Continuous Data Protection (CDP) charges incurred for a temporary table?

-  A.30 days
-  B.7 days
-  C.48 hours
-  D.24 hours

<details><summary>Show Answer</summary>
Correct Answer: -  D.24 hours — temporary tables get at most 1 day of Time Travel and no fail-safe.
</details>

---

### Question 143
When reviewing a query profile, what is a symptom that a query is too large to fit into memory?

-  A.A single join node uses more than 50% of query time
-  B.Partitions scanned equals total partitions
-  C.An Aggregate operator node is present
-  D.The query is spilling to remote storage

<details><summary>Show Answer</summary>
Correct Answer: -  D.Spilling (especially to remote storage, which is slower than local disk spilling) indicates the warehouse doesn't have enough memory/local disk for the operation.
</details>

---

### Question 144
What type of query benefits the MOST from search optimization?

-  A.A query that uses only disjunction (OR) predicates
-  B.A query that includes analytical expressions
-  C.A query that uses equality predicates or `IN` predicates
-  D.A query that filters on semi-structured data types

<details><summary>Show Answer</summary>
Correct Answer: -  C.The Search Optimization Service is most effective for highly selective equality/`IN` lookups on columns with high cardinality.
</details>

---

### Question 145
What transformations are supported in a `CREATE PIPE AS COPY FROM (...)` statement? (Choose two.)

-  A.Data can be filtered by an optional `WHERE` clause.
-  B.Incoming data can be joined with other tables.
-  C.Columns can be reordered/cast.
-  D.Columns can be omitted.

<details><summary>Show Answer</summary>
Correct Answer: C, -  D.`COPY INTO` (and pipes built on it) supports simple column reordering, casting, and omission — but not `WHERE` filtering or joins.
</details>

---

### Question 146
Which of the following are characteristics of Snowflake virtual warehouses? (Choose two.)

-  A.Auto-suspend applies only to the last warehouse started in a multi-cluster warehouse.
-  B.The ability to auto-suspend a warehouse is only available in Enterprise edition or above.
-  C.SnowSQL supports both a configuration file and a command-line option for specifying a default warehouse.
-  D.A user cannot specify a default warehouse when using the ODBC driver.
-  E.The default virtual warehouse size can be changed at any time.

<details><summary>Show Answer</summary>
Correct Answer: C, E.
</details>

---

### Question 147
Which command should be used to load data from a file located in an external stage into a table in Snowflake?

-  A.`INSERT`
-  B.`PUT`
-  C.`GET`
-  D.`COPY`

<details><summary>Show Answer</summary>
Correct Answer: -  D.`COPY INTO <table>`
</details>

---

### Question 148
The Snowflake Data Cloud platform is described as having which of the following architectures?

-  A.Shared-disk
-  B.Shared-nothing
-  C.Multi-cluster, shared data
-  D.Serverless query engine

<details><summary>Show Answer</summary>
Correct Answer: -  C.Multi-cluster, shared data architecture — combines the benefits of shared-disk (single copy of data) and shared-nothing (independent compute clusters).
</details>

---

### Question 149
Which of the following is a data tokenization integration partner?

-  A.Protegrity
-  B.Tableau

<details><summary>Show Answer</summary>
Correct Answer: -  A.Protegrity
</details>

---

### Question 150
What editions of Snowflake should be used to manage compliance with Personal Identifiable Information (PII) requirements? (Choose two.)

-  A.Custom Edition
-  B.Virtual Private Snowflake
-  C.Business Critical Edition
-  D.Standard Edition
-  E.Enterprise Edition

<details><summary>Show Answer</summary>
Correct Answer: B, -  C.Business Critical and Virtual Private Snowflake are the editions designed for handling highly sensitive data such as PII/PHI.
</details>

---

### Question 151
What are supported file formats for unloading data from Snowflake? (Choose three.)

-  A.XML
-  B.JSON
-  C.Parquet
-  D.ORC
-  E.Avro
-  F.CSV

<details><summary>Show Answer</summary>
Correct Answer: B, C, -  F.JSON, Parquet, and CSV.

⚠ Updated: The original answer key listed **B, C, E** (JSON, Parquet, Avro). That's incorrect — per Snowflake's `CREATE FILE FORMAT` documentation, **Avro, ORC, and XML are load-only formats** and cannot be used to unload data. The correct trio is JSON, Parquet, and CSV.
</details>

---

### Question 152
The Snowflake cloud services layer is responsible for which tasks? (Choose two.)

-  A.Local disk caching
-  B.Authentication and access control
-  C.Metadata management
-  D.Query processing
-  E.Database storage

<details><summary>Show Answer</summary>
Correct Answer: B, C.
</details>

---

### Question 153
What is a key feature of Snowflake architecture?

-  A.Zero-copy cloning creates a mirror copy of a database that updates with the original.
-  B.Software updates are automatically applied on a quarterly basis.
-  C.Snowflake eliminates resource contention through its virtual warehouse implementation.
-  D.Multi-cluster warehouses allow a single query to span multiple clusters.
-  E.Data is sorted during ingest for fast retrieval by date.

<details><summary>Show Answer</summary>
Correct Answer: -  C.Because each virtual warehouse is an independent compute cluster, workloads on separate warehouses don't contend for the same compute resources.
</details>

---

### Question 154
When publishing a Snowflake Marketplace listing into a remote region, what should be taken into consideration? (Choose two.)

-  A.There needs to be a share created in the target region for each account.
-  B.The listing is replicated into all selected regions automatically; the underlying data is not.
-  C.The user must have the `ORGADMIN` role in at least one account to link accounts for replication.
-  D.Shares attached to listings in remote regions can be viewed from any account in the organization.
-  E.For a standard listing, the provider can wait until the first customer requests the data before replicating it to the target region.

<details><summary>Show Answer</summary>
Correct Answer: B, E.
</details>

---

### Question 155
When loading data into Snowflake via Snowpipe, what is the recommended compressed file size?

-  A.10–50 MB
-  B.100–250 MB
-  C.300–500 MB
-  D.1000–1500 MB

<details><summary>Show Answer</summary>
Correct Answer: -  B.100–250 MB (same recommendation as bulk loading via `COPY INTO`).
</details>

---

### Question 156
Which Snowflake feature allows a user to substitute a randomly generated identifier for sensitive data, to prevent unauthorized access, before loading it into Snowflake?

-  A.External Tokenization
-  B.External Tables
-  C.Materialized Views
-  D.User-Defined Table Functions (UDTFs)

<details><summary>Show Answer</summary>
Correct Answer: -  A.External Tokenization — data is tokenized outside Snowflake (via a partner integration like Protegrity) before it's loaded.
</details>

---

### Question 157
Which of the following are examples of operations that require a Virtual Warehouse to complete, assuming no queries have been executed previously? (Choose three.)

-  A.`MIN(<column value>)`
-  B.`COPY`
-  C.`SUM(<column value>)`
-  D.`UPDATE`

<details><summary>Show Answer</summary>
Correct Answer: B, C, -  D.`COPY`, `SUM()`, and `UPDATE` all require an active warehouse to scan/write data. `MIN()`/`MAX()` on a column can sometimes be answered directly from stored micro-partition metadata (min/max values per column), similar to `COUNT(*)`, without needing compute.
</details>

---

### Question 158
Which `SNOWFLAKE.ACCOUNT_USAGE` view contains information about which objects were read by queries within the last 365 days?

-  A.`VIEWS_HISTORY`
-  B.`OBJECT_HISTORY`
-  C.`ACCESS_HISTORY`
-  D.`LOGIN_HISTORY`

<details><summary>Show Answer</summary>
Correct Answer: -  C.`ACCESS_HISTORY`
</details>

---

### Question 159
Which feature is only available in the Enterprise or higher editions of Snowflake?

-  A.Column-level security
-  B.SOC 2 Type II certification
-  C.Multi-factor Authentication (MFA)
-  D.Object-level access control

<details><summary>Show Answer</summary>
Correct Answer: -  A.Column-level security (dynamic data masking, etc.). SOC 2 Type II, MFA, and basic object-level access control are available in all editions.
</details>

---

### Question 160
Will data cached in a warehouse be lost when the warehouse is resized?

-  A.Possibly — if resized to a smaller size, some cache may no longer fit.
-  B.Yes — because the compute resource is replaced in its entirety with a new compute resource.
-  C.No — the size of the cache is independent from the warehouse size.
-  D.Yes — because the compute resource will no longer have access to the cache encryption key.

<details><summary>Show Answer</summary>
Correct Answer: -  B.Resizing a warehouse provisions new compute nodes, so the local disk (warehouse) cache is dropped and rebuilt.
</details>

---

### Question 161
Which semi-structured file formats are supported when unloading data from a table? (Choose two.)

-  A.ORC
-  B.XML
-  C.Avro
-  D.Parquet
-  E.JSON

<details><summary>Show Answer</summary>
Correct Answer: D, -  E.Parquet and JSON. (ORC, Avro, and XML are load-only.)
</details>

---

### Question 162
A running virtual warehouse is suspended, then restarted. What is the MINIMUM amount of time the warehouse will incur charges for once restarted?

-  A.1 second
-  B.60 seconds
-  C.5 minutes
-  D.60 minutes

<details><summary>Show Answer</summary>
Correct Answer: -  B.60 seconds
</details>

---

### Question 163
What are the responsibilities of Snowflake's Cloud Services layer? (Choose three.)

-  A.Authentication
-  B.Resource management
-  C.Virtual warehouse (local disk) caching
-  D.Query parsing and optimization
-  E.Query execution
-  F.Physical storage of micro-partitions

<details><summary>Show Answer</summary>
Correct Answer: A, B, -  D.Authentication, infrastructure/resource management, and query parsing & optimization. Query execution and warehouse-local caching happen in the Compute layer; physical storage happens in the Storage layer.
</details>

---

### Question 164
How long is the fail-safe period for temporary and transient tables?

-  A.There is no fail-safe period for these tables.
-  B.1 day
-  C.14 days
-  D.31 days

<details><summary>Show Answer</summary>
Correct Answer: -  A.Temporary and transient tables have no fail-safe period at all.
</details>

---

### Question 165
Which command should be used to download a file from a Snowflake stage to a local folder on a client machine?

-  A.`PUT`
-  B.`GET`
-  C.`COPY`
-  D.`SELECT`

<details><summary>Show Answer</summary>
Correct Answer: -  B.`GET`
</details>

---

### Question 166
How does Snowflake fail-safe protect data in a table?

-  A.Fail-safe makes data available for up to 1 day, recoverable by user operations.
-  B.Fail-safe makes data available for 7 days, recoverable by users.
-  C.Fail-safe makes data available for 7 days, recoverable only by Snowflake Support.
-  D.Fail-safe makes data available for up to 1 day, recoverable only by Snowflake Support.

<details><summary>Show Answer</summary>
Correct Answer: -  C.Fail-safe provides a 7-day recovery window, but only Snowflake Support can perform the recovery (it's not self-service like Time Travel).
</details>

---

### Question 167
A virtual warehouse is created with `WAREHOUSE_SIZE = MEDIUM`, `AUTO_SUSPEND = 60`, `AUTO_RESUME = TRUE`. The utilization graph over two days shows recurring spikes suggesting queries are queuing at busy times. What action should be taken?

-  A.Increase the warehouse size from Medium to 2X-Large.
-  B.Increase the value of `AUTO_SUSPEND`.
-  C.Configure the warehouse as a multi-cluster warehouse.
-  D.Lower the value of `AUTO_SUSPEND`.

<details><summary>Show Answer</summary>
Correct Answer: -  C.Recurring concurrency-driven spikes/queuing are addressed by scaling out (multi-cluster warehouses), not scaling up.
</details>

---

### Question 168
Which minimum Snowflake edition allows for a dedicated metadata store?

-  A.Standard
-  B.Enterprise
-  C.Business Critical
-  D.Virtual Private Snowflake

<details><summary>Show Answer</summary>
Correct Answer: -  D.Virtual Private Snowflake (VPS) provides a completely isolated, dedicated environment including a dedicated metadata store, separate from Snowflake's standard multi-tenant metadata layer.
</details>

---

### Question 169
Network policies can be set at which Snowflake levels? (Choose two.)

-  A.Role
-  B.Schema
-  C.User
-  D.Database
-  E.Account

<details><summary>Show Answer</summary>
Correct Answer: C, -  E.Network policies can be applied at the account level or to individual users (and also to security integrations).
</details>

---

### Question 170
What are the correct parameters for Time Travel and fail-safe in the Snowflake Enterprise Edition?

-  A.Default Time Travel Retention = 0 days; Maximum = 30 days; Fail-safe = 1 day
-  B.Default = 1 day; Maximum = 365 days; Fail-safe = 7 days
-  C.Default = 0 days; Maximum = 90 days; Fail-safe = 7 days
-  D.Default = 1 day; Maximum = 90 days; Fail-safe = 7 days
-  E.Default = 7 days; Maximum = 1 day; Fail-safe = 90 days
-  F.Default = 90 days; Maximum = 7 days; Fail-safe = 356 days

<details><summary>Show Answer</summary>
Correct Answer: -  D.Default Time Travel retention is 1 day (configurable up to a maximum of 90 days in Enterprise+); permanent-table fail-safe is a fixed 7 days.
</details>

---

### Question 171
Which of the following objects are contained within a schema? (Choose two.)

-  A.Role
-  B.Table
-  C.Warehouse
-  D.External table
-  E.User
-  F.Share

<details><summary>Show Answer</summary>
Correct Answer: B, -  D.Tables and external tables are schema-level objects. Roles, warehouses, users, and shares are account-level objects.
</details>

---

### Question 172
Which of the following statements describe features of Snowflake data caching? (Choose two.)

-  A.When a virtual warehouse is suspended, its data cache is saved to remote storage.
-  B.When the data cache is full, the least-recently-used data is cleared to make room.
-  C.A user can only access their own queries from the query result cache.
-  D.A user must set a parameter to `TRUE` to enable the metadata cache.
-  E.The `RESULT_SCAN` table function can access and filter the contents of the query result cache.

<details><summary>Show Answer</summary>
Correct Answer: B, E.
</details>

---

### Question 173
A table needs to be loaded. The input is JSON, a concatenation of multiple JSON documents, 3 GB total, loaded with a Small warehouse via:
```sql
COPY INTO SAMPLE FROM @stage FILE_FORMAT = (TYPE = JSON);
```
The load fails with: `Max LOB size (16777216) exceeded. actual size of parsed column is 17894470.` How can this be resolved?

-  A.Compress the file and reload it.
-  B.Split the file into multiple files in the recommended size range (100–250 MB).
-  C.Use a larger warehouse.
-  D.Set `STRIP_OUTER_ARRAY = TRUE` in the `COPY INTO` command.

<details><summary>Show Answer</summary>
Correct Answer: -  D.This error means a single parsed value exceeded the 16 MB VARIANT limit — typically because the whole file is being parsed as one giant array/object. Setting `STRIP_OUTER_ARRAY = TRUE` (or otherwise correcting the JSON structure to be one document per line) resolves it — file size and warehouse size are not the cause.
</details>

---

### Question 174
What is a feature of a stored procedure in Snowflake?

-  A.They can secure and hide the underlying metadata from all roles.
-  B.They can only access tables from a single database.
-  C.They can only contain a single SQL statement.
-  D.They can be created to run with a caller's rights or an owner's rights.

<details><summary>Show Answer</summary>
Correct Answer: -  D.Stored procedures can be defined with `EXECUTE AS CALLER` or `EXECUTE AS OWNER`.
</details>

---

### Question 175
Which columns are part of the result set of the `LATERAL FLATTEN` function? (Choose two.)

-  A.`CONTENT`
-  B.`PATH`
-  C.`BYTE_SIZE`
-  D.`INDEX`
-  E.`DATATYPE`

<details><summary>Show Answer</summary>
Correct Answer: B, -  D.`FLATTEN` returns `SEQ`, `KEY`, `PATH`, `INDEX`, `VALUE`, and `THIS` — of the choices given, `PATH` and `INDEX` are real output columns.
</details>

---

### Question 176
What is the minimum edition required to create a materialized view?

-  A.Standard Edition
-  B.Enterprise Edition
-  C.Business Critical Edition
-  D.Virtual Private Snowflake Edition

<details><summary>Show Answer</summary>
Correct Answer: -  B.Enterprise Edition
</details>

---

### Question 177
Which Snowflake function will interpret an input string as a JSON document and produce a `VARIANT` value?

-  A.`TO_JSON`
-  B.`CHECK_JSON`
-  C.`TRY_PARSE_JSON`
-  D.`PARSE_JSON`

<details><summary>Show Answer</summary>
Correct Answer: -  D.`PARSE_JSON`
</details>

---

### Question 178
How are serverless features billed?

-  A.Per second, multiplied by an automatically-determined sizing for the job.
-  B.Per minute, multiplied by an automatic sizing for the job, with a minimum of one minute.
-  C.Per second, multiplied by the size set by the `SERVERLESS_TASK_SIZE` parameter.
-  D.Serverless features are not billed unless total monthly cost exceeds warehouse credit usage.

<details><summary>Show Answer</summary>
Correct Answer: -  A.Serverless features (Snowpipe, serverless tasks, automatic clustering, etc.) use Snowflake-managed compute that is automatically sized and billed per second.
</details>

---

### Question 179
Which Snowflake architectural layer is responsible for producing a query execution plan?

-  A.Compute
-  B.Data storage
-  C.Cloud Services
-  D.Cloud provider

<details><summary>Show Answer</summary>
Correct Answer: -  C.Cloud Services (query parsing, optimization, and plan generation happen here before compute executes the plan).
</details>

---

### Question 180
When unloading data to a stage, which of the following is a recommended practice?

-  A.Set `SINGLE = TRUE` for larger files.
-  B.Use `MAX_FILE_SIZE` when using Parquet.
-  C.Avoid the use of the `CAST` function.
-  D.Define a named file format.

<details><summary>Show Answer</summary>
Correct Answer: -  D.Define an individual (named) file format for reuse and consistency.
</details>

---

### Question 181
Which SQL commands, when committed, will consume a stream and advance its offset? (Choose two.)

-  A.`UPDATE ... FROM STREAM`
-  B.`SELECT * FROM STREAM`
-  C.`INSERT INTO <table> SELECT * FROM STREAM`
-  D.`CREATE TABLE ... AS SELECT * FROM STREAM`

<details><summary>Show Answer</summary>
Correct Answer: A, -  C.A plain `SELECT` against a stream does **not** advance its offset — only DML (`INSERT`, `UPDATE`, `DELETE`, `MERGE`) that consumes the stream's change data and is committed will advance it.
</details>

---

### Question 182
Which methods can be used to delete staged files from a Snowflake stage? (Choose two.)

-  A.Use the `DROP FILE` command after the load completes.
-  B.Specify a compression option when creating the file format.
-  C.Specify the `PURGE` copy option in the `COPY INTO <table>` command.
-  D.Use the `REMOVE` command after the load completes.

<details><summary>Show Answer</summary>
Correct Answer: C, D.
</details>

---

### Question 183
On which of the following cloud platforms can a Snowflake account be hosted? (Choose three.)

-  A.Amazon Web Services
-  B.Private Virtual Cloud
-  C.Oracle Cloud
-  D.Microsoft Azure
-  E.Google Cloud Platform

<details><summary>Show Answer</summary>
Correct Answer: A, D, -  E.AWS, Azure, and GCP.
</details>

---

### Question 184
What Snowflake role must be granted for a user to create and manage accounts (within an organization)?

-  A.`ACCOUNTADMIN`
-  B.`ORGADMIN`
-  C.`SECURITYADMIN`
-  D.`SYSADMIN`

<details><summary>Show Answer</summary>
Correct Answer: -  B.`ORGADMIN`
</details>

---

### Question 185
*This question referenced a visual diagram (micro-partitions with value ranges A–Z) not reproducible from the OCR text. The concept being tested is recognizing a well-clustered table: a table where each micro-partition holds a narrow, mostly non-overlapping range of the clustering key's values (visualized as little/no overlap between partitions' min/max ranges). Recommend reviewing this one against the original source image.*

---

### Question 186
What feature can be used to reorganize a very large table on one or more columns?

-  A.Micro-partitions
-  B.Clustering keys
-  C.Key partitions
-  D.Clustered partitions

<details><summary>Show Answer</summary>
Correct Answer: -  B.Clustering keys
</details>

---

### Question 187
What is an advantage of using an explain plan instead of the query profiler to evaluate the performance of a query?

-  A.The plan output is available graphically.
-  B.An explain plan can be used to conduct performance analysis without executing the query.
-  C.An explain plan handles queries with temporary tables that the profiler will not.
-  D.Explain plan output displays automatic data-skew optimization information.

<details><summary>Show Answer</summary>
Correct Answer: -  B.`EXPLAIN` estimates the plan without actually running (and paying for) the query.
</details>

---

### Question 188
Which data types are supported by Snowflake when using semi-structured data? (Choose two.)

-  A.`VARIANT`
-  B.`VARRAY`
-  C.`STRUCT`
-  D.`ARRAY`
-  E.`QUEUE`

<details><summary>Show Answer</summary>
Correct Answer: A, -  D.`VARIANT` and `ARRAY` (along with `OBJECT`) are Snowflake's semi-structured data types.
</details>

---

### Question 189
Why does Snowflake recommend file sizes of 100–250 MB compressed when loading data?

-  A.Optimizes the virtual warehouse size and multi-cluster setting to economy mode.
-  B.Allows a user to import the files in sequential order.
-  C.Increases the latency of staging and accuracy when loading the data.
-  D.Allows optimization of parallel operations.

<details><summary>Show Answer</summary>
Correct Answer: -  D.This file size range enables Snowflake to parallelize loading efficiently across the threads/nodes in a warehouse.
</details>

---

### Question 190
Which of the following features are available with the Snowflake Enterprise edition? (Choose two.)

-  A.Database replication and failover
-  B.Automated index management
-  C.Customer-managed keys (Tri-Secret Secure)
-  D.Extended Time Travel
-  E.Native support for geospatial data

<details><summary>Show Answer</summary>
Correct Answer: D, E.

Note: Extended Time Travel (up to 90 days) is a genuine Enterprise-and-above feature. Geospatial support (`GEOGRAPHY`/`GEOMETRY` types) is actually available across **all** editions today, not exclusive to Enterprise — so treat option E with some caution if the exam intends "exclusive to Enterprise." Database replication/failover and Tri-Secret Secure customer-managed keys require **Business Critical** or higher, not just Enterprise. "Automated index management" isn't a real Snowflake feature (Snowflake has no traditional indexes).
</details>

---

### Question 191
What is the default file size limit when unloading data from Snowflake using the `COPY` command?

-  A.5 MB
-  B.8 GB
-  C.16 MB
-  D.32 MB

<details><summary>Show Answer</summary>
Correct Answer: -  C.16 MB is the default `MAX_FILE_SIZE` for unloading (can be increased up to 5 GB).
</details>

---

### Question 192
What features that are part of the Continuous Data Protection (CDP) feature set in Snowflake do NOT require additional configuration? (Choose two.)

-  A.Row access policies
-  B.Data masking policies
-  C.Data encryption
-  D.Time Travel

<details><summary>Show Answer</summary>
Correct Answer: C, -  D.Data encryption and Time Travel are on by default. Row access policies and masking policies must be explicitly created and applied.
</details>

---

### Question 193
Which Snowflake layer is always leveraged when accessing a query from the result cache?

-  A.Metadata
-  B.Data Storage
-  C.Compute
-  D.Cloud Services

<details><summary>Show Answer</summary>
Correct Answer: -  D.Cloud Services
</details>

---

### Question 194
Which connectors are available in the downloads section of the Snowflake web interface? (Choose two.)

-  A.SnowSQL
-  B.JDBC
-  C.ODBC
-  D.HIVE
-  E.Scala

<details><summary>Show Answer</summary>
Correct Answer: A, -  C.SnowSQL and the ODBC driver (JDBC is also downloadable, but per the original answer key this pairing was marked correct).
</details>

---

### Question 195
A Snowflake Administrator needs sensitive corporate data to be invisible to end users, but partially visible to functional managers. How can this requirement be met?

-  A.Use data encryption.
-  B.Use dynamic data masking.
-  C.Use secure materialized views.
-  D.Revoke all roles for functional managers and end users.

<details><summary>Show Answer</summary>
Correct Answer: -  B.Dynamic data masking lets you define masking policies that reveal/hide data conditionally based on the querying role.
</details>

---

### Question 196
Users are responsible for data storage costs until what occurs?

-  A.Data expires from Time Travel.
-  B.Data expires from fail-safe.
-  C.Data is deleted from a table.
-  D.Data is truncated from a table.

<details><summary>Show Answer</summary>
Correct Answer: -  B.Storage is billed through the full Time Travel + fail-safe retention window, not just until deletion/truncation.
</details>

---

### Question 197
A user has an application that writes a new file to a cloud storage location every 5 minutes. What would be the MOST efficient way to get the files into Snowflake?

-  A.Create a task that runs a `COPY INTO` operation from an external stage every 5 minutes.
-  B.Create a task that PUTs the files into an internal stage and automates loading.
-  C.Create a task that runs a `GET` operation to intermittently check for new files.
-  D.Set up cloud provider event notifications on the location and use Snowpipe with auto-ingest.

<details><summary>Show Answer</summary>
Correct Answer: -  D.Snowpipe with auto-ingest (triggered by cloud storage event notifications) is the most efficient, near-real-time, and cost-effective option — no polling required.
</details>

---

### Question 198
What affects whether the query results cache can be used?

-  A.If the query contains a deterministic function.
-  B.If the virtual warehouse has been suspended.
-  C.If the referenced data in the table has changed.
-  D.If multiple users are using the same virtual warehouse.

<details><summary>Show Answer</summary>
Correct Answer: -  C.If the underlying table data has changed since the cached result was generated, the cache is invalidated for that query.
</details>

---

### Question 199
Which of the following is an example of an operation that can be completed without requiring compute, assuming no queries have been executed previously?

-  A.`SELECT SUM(ORDER_AMT) FROM SALES;`
-  B.`SELECT * FROM SALES`
-  C.`SELECT MIN(ORDER_AMT) FROM SALES`
-  D.`SELECT ORDER_AMT, ORDER_QTY FROM SALES`

<details><summary>Show Answer</summary>
Correct Answer: -  C.`MIN()`/`MAX()` on a column can be answered directly from micro-partition metadata (which stores per-partition min/max), without scanning data via a warehouse — similar to how `COUNT(*)` works.
</details>

---

### Question 200
How many days is load history for Snowpipe retained?

-  A.1 day
-  B.90 days
-  C.14 days
-  D.60 days

<details><summary>Show Answer</summary>
Correct Answer: -  C.14 days
</details>

---

## Summary of Corrections Made to the Original Answer Key

| Question | Original Answer | Corrected Answer | Reason |
|---|---|---|---|
| 151 | B, C, E (JSON, Parquet, Avro) | B, C, F (JSON, Parquet, CSV) | Avro is load-only; Snowflake cannot unload to Avro format. |
| 190 | D, E | D, E (with caveat) | Geospatial data types are available in all editions, not Enterprise-exclusive — flagged as a possibly outdated/ambiguous distractor rather than a hard error. |

Two items (Q132 and Q185) had source text too corrupted or image-dependent to responsibly reconstruct — noted honestly rather than guessed.

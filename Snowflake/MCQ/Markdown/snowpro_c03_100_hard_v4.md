# SnowPro Core (COF-C03) — 100 Hard Practice Questions (v3)

> **Exam Blueprint Coverage** | Domain 1: Architecture (25%) · Domain 2: Virtual Warehouses (20%) · Domain 3: Storage & Protection (15%) · Domain 4: Data Movement (15%) · Domain 5: Account & Security (15%) · Domain 6: Performance (10%)
>
> All questions are **new** — no duplicates from previous question sets. Difficulty tuned to match COF-C03 exam depth (2024–2025 documentation).

---

### Question 1
**Domain:** Domain 1 — Architecture

A Snowflake account has a replication group containing a database, a share, and an integration object. The DBA runs `ALTER REPLICATION GROUP my_rg REFRESH` on the secondary account. What is the replication behavior for the share?

- [ ] A. Shares cannot be included in replication groups; they must be re-created manually on the secondary account.
- [ ] B. The share object definition (including the objects it contains and the consumer accounts it is granted to) is replicated to the secondary, making the secondary a functional provider for those consumers.
- [ ] C. Only the share metadata is replicated; the DBA must manually grant the share to consumers on the secondary account.
- [ ] D. Shares are replicated as read-only snapshots; the secondary cannot modify share membership.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The share object definition (including the objects it contains and the consumer accounts it is granted to) is replicated to the secondary, making the secondary a functional provider for those consumers.

**Explanation:**
Snowflake's **Business Continuity replication** supports replicating share objects within a replication group. When the secondary is refreshed, the share — including its granted consumer accounts — is replicated, enabling the secondary account to serve as a fully functional provider during a failover. This is critical for disaster recovery of data-sharing workflows.
</details>

---

### Question 2
**Domain:** Domain 1 — Architecture

A developer creates a Snowflake Native App and defines an application role `app_admin`. A consumer installs the app and wants their SYSADMIN role to use the app's functionality. What is the correct mechanism?

- [ ] A. The provider grants SYSADMIN access to the application package directly.
- [ ] B. The consumer uses `GRANT APPLICATION ROLE <app_name>.app_admin TO ROLE SYSADMIN` in their account after installation.
- [ ] C. The application role is automatically assigned to ACCOUNTADMIN on the consumer account during installation.
- [ ] D. Application roles cannot be assigned to account roles; consumers must use a dedicated service account.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The consumer uses `GRANT APPLICATION ROLE <app_name>.app_admin TO ROLE SYSADMIN` in their account after installation.

**Explanation:**
In the **Snowflake Native Apps Framework**, providers define **application roles** within the app. After installation, consumers can grant these application roles to their own account roles using `GRANT APPLICATION ROLE <app_name>.<role_name> TO ROLE <account_role>`. This is the standard privilege-escalation mechanism; no automatic grants occur during installation beyond what the app's setup script explicitly requests.
</details>

---

### Question 3
**Domain:** Domain 1 — Architecture

A Snowflake account is on the **Virtual Private Snowflake (VPS)** edition. Which statement is ACCURATE regarding the infrastructure difference?

- [ ] A. VPS accounts run in a dedicated VPC per customer, but still share the Cloud Services layer with other customers.
- [ ] B. VPS accounts run entirely on dedicated hardware including the Cloud Services layer — fully isolated from all other Snowflake customers.
- [ ] C. VPS is simply a marketing term for Business Critical with a private link configured.
- [ ] D. VPS accounts cannot use Snowflake Marketplace because they are network-isolated.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. VPS accounts run entirely on dedicated hardware including the Cloud Services layer — fully isolated from all other Snowflake customers.

**Explanation:**
**Virtual Private Snowflake (VPS)** is the most isolated Snowflake deployment. Unlike Business Critical (which shares the Cloud Services layer with other customers but runs virtual warehouses in dedicated VPCs), VPS isolates **all three layers** — storage, compute, and cloud services — on dedicated hardware. It is designed for organizations with extreme regulatory requirements (defense, high-security government). VPS accounts can still access the Marketplace via Snowflake's secure data sharing, but with additional controls.
</details>

---

### Question 4
**Domain:** Domain 1 — Architecture

A Snowflake account creates an event table. A UDF throws an exception. Which statement about event table logging behavior is CORRECT?

- [ ] A. Events are written synchronously; a failed UDF blocks execution until the event is persisted.
- [ ] B. The event table only captures INFO-level log messages; exceptions are not recorded.
- [ ] C. The UDF exception is captured as a log event with severity FATAL automatically if the account has `LOG_LEVEL = FATAL` set at the account level.
- [ ] D. Event tables capture telemetry asynchronously; the exception is logged as an ERROR-level record and does not block the calling statement.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. Event tables capture telemetry asynchronously; the exception is logged as an ERROR-level record and does not block the calling statement.

**Explanation:**
Snowflake **event tables** receive telemetry (logs, traces, metrics) from UDFs, stored procedures, and Snowpark code. Logging is **asynchronous** — it does not impact query execution performance. When a UDF raises an exception, if the handler logs the error (or if auto-instrumentation is enabled), it is recorded with ERROR severity. The `LOG_LEVEL` parameter controls the minimum severity captured. Event data is queryable like regular table data.
</details>

---

### Question 5
**Domain:** Domain 1 — Architecture

A Snowflake Task DAG has a root task with schedule `USING CRON 0 * * * * UTC` and three child tasks forming a linear chain. If the root task execution takes 45 minutes, what happens when the next scheduled trigger fires?

- [ ] A. The new root task run starts immediately, running in parallel with the previous run.
- [ ] B. The new trigger is skipped entirely; only one run of a task can be active at a time, and missed triggers are not queued.
- [ ] C. The new trigger is queued and starts as soon as the in-progress run finishes.
- [ ] D. Snowflake raises an error and suspends the task until a human resumes it.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The new trigger is skipped entirely; only one run of a task can be active at a time, and missed triggers are not queued.

**Explanation:**
Snowflake Tasks enforce **single-concurrency** by default: if an instance of a task is still running when the next scheduled trigger fires, the new trigger is **skipped** (not queued). Missed executions are not backfilled. This prevents stacking of slow tasks. If consecutive skips occur (default threshold: 5), Snowflake auto-suspends the task and sends an alert. Developers should set schedule intervals longer than the expected execution time or implement idempotent logic.
</details>

---

### Question 6
**Domain:** Domain 1 — Architecture

Which of the following objects is NOT supported inside a Snowflake Database Replication Group (standard replication, not failover group)?

- [ ] A. Regular tables (permanent)
- [ ] B. Dynamic tables
- [ ] C. External tables
- [ ] D. Sequences

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. External tables

**Explanation:**
**External tables** are not replicated in standard database replication because they are metadata-only objects that reference files in external stages. The external stage itself (and the cloud storage location) is not part of Snowflake's storage, so replication of the external table definition without the corresponding stage configuration and file access would result in a broken reference. Permanent tables, dynamic tables, and sequences are supported replication objects.
</details>

---

### Question 7
**Domain:** Domain 2 — Virtual Warehouses

A warehouse has `STATEMENT_TIMEOUT_IN_SECONDS = 3600` set at the warehouse level. A user with a session parameter `STATEMENT_TIMEOUT_IN_SECONDS = 600` submits a query. Which timeout applies?

- [ ] A. 3600 seconds — warehouse-level parameters always take precedence over session-level.
- [ ] B. 600 seconds — the more restrictive (smaller) value between session and warehouse level is applied.
- [ ] C. 600 seconds — session-level parameters always take precedence over warehouse-level.
- [ ] D. Both timeouts run simultaneously; the query is terminated when either fires first.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. 600 seconds — the more restrictive (smaller) value between session and warehouse level is applied.

**Explanation:**
Snowflake evaluates `STATEMENT_TIMEOUT_IN_SECONDS` at multiple levels (account → warehouse → session). The **most restrictive (smallest non-zero) value** wins. In this case, 600 (session) < 3600 (warehouse), so the 600-second timeout is applied. This hierarchy prevents users from overriding stricter warehouse-level guardrails by setting a longer session timeout.
</details>

---

### Question 8
**Domain:** Domain 2 — Virtual Warehouses

A warehouse is configured with `MAX_CONCURRENCY_LEVEL = 4`. A 5th simultaneous query arrives. What is the warehouse's behavior?

- [ ] A. The 5th query is immediately rejected with an error.
- [ ] B. The warehouse automatically adds a cluster (if multi-cluster is enabled); otherwise the 5th query queues until one of the 4 running queries finishes.
- [ ] C. The 5th query pre-empts the oldest running query.
- [ ] D. The warehouse scales vertically to a larger size to accommodate the 5th query.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The warehouse automatically adds a cluster (if multi-cluster is enabled); otherwise the 5th query queues until one of the 4 running queries finishes.

**Explanation:**
`MAX_CONCURRENCY_LEVEL` defines the number of queries that can run simultaneously on a single cluster. When this limit is reached, additional queries **queue**. If the warehouse is a **multi-cluster warehouse**, Snowflake may spin up an additional cluster to absorb the overflow (depending on SCALING_POLICY). If it is a single-cluster warehouse, the query waits in the queue. There is no pre-emption or vertical scaling.
</details>

---

### Question 9
**Domain:** Domain 2 — Virtual Warehouses

A Snowflake warehouse uses `SCALING_POLICY = ECONOMY`. Under what condition does this policy add a cluster, compared to `SCALING_POLICY = STANDARD`?

- [ ] A. ECONOMY adds clusters immediately when any query queues; STANDARD waits 2 minutes before adding.
- [ ] B. ECONOMY favors keeping clusters running longer before shutting down, but adds clusters at the same rate as STANDARD.
- [ ] C. ECONOMY waits until it can justify running the new cluster for at least 6 minutes of utilization before starting it; STANDARD adds a cluster as soon as queries queue.
- [ ] D. ECONOMY only adds clusters when CPU utilization exceeds 90%; STANDARD adds on queue depth alone.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. ECONOMY waits until it can justify running the new cluster for at least 6 minutes of utilization before starting it; STANDARD adds a cluster as soon as queries queue.

**Explanation:**
In **STANDARD** scaling policy, Snowflake spins up an additional cluster as soon as queries start queuing, minimizing latency. In **ECONOMY** mode, Snowflake waits to add a cluster until it estimates the new cluster will be kept busy for at least **6 minutes** (the minimum billing unit). This reduces unnecessary cluster spin-ups and associated costs, at the expense of potentially longer queue wait times for transient spikes.
</details>

---

### Question 10
**Domain:** Domain 2 — Virtual Warehouses

A developer creates a warehouse with `WAREHOUSE_TYPE = SNOWPARK-OPTIMIZED` and size X-LARGE. What is the primary cost implication compared to a STANDARD X-LARGE warehouse?

- [ ] A. Snowpark-Optimized warehouses are billed at the same credit rate as Standard warehouses.
- [ ] B. Snowpark-Optimized warehouses consume more credits per hour than a Standard warehouse of the same size due to the additional memory resources provisioned.
- [ ] C. Snowpark-Optimized warehouses are cheaper because they skip micro-partition scanning.
- [ ] D. Snowpark-Optimized warehouses do not consume credits; they are billed per GB of data processed.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowpark-Optimized warehouses consume more credits per hour than a Standard warehouse of the same size due to the additional memory resources provisioned.

**Explanation:**
**Snowpark-Optimized warehouses** are provisioned with significantly more RAM per node (~16× standard) to support in-memory ML and DataFrame workloads. This additional resource provisioning means they **cost more credits per hour** than a Standard warehouse of the same T-shirt size. Organizations should only use them for workloads that actually benefit from the extra memory; using them for ordinary SQL queries wastes budget.
</details>

---

### Question 11
**Domain:** Domain 3 — Storage & Data Protection

An enterprise table has `DATA_RETENTION_TIME_IN_DAYS = 90`. The table is accidentally dropped. After 7 days, someone attempts `UNDROP TABLE`. What happens?

- [ ] A. The UNDROP fails because tables can only be undroped within 24 hours.
- [ ] B. The UNDROP succeeds because the retention period is 90 days and only 7 days have elapsed.
- [ ] C. The UNDROP fails because UNDROP only works for transient tables.
- [ ] D. The UNDROP succeeds but restores the table to the state it was in exactly 7 days ago, not at the time of drop.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The UNDROP succeeds because the retention period is 90 days and only 7 days have elapsed.

**Explanation:**
`UNDROP TABLE` uses **Time Travel** to restore a dropped table. As long as the drop occurred within the table's `DATA_RETENTION_TIME_IN_DAYS` window, `UNDROP` can recover it to the state it was in **at the time of the drop**. Here, 90-day retention means the table is recoverable for 90 days post-drop. After that window expires, the data moves into **Fail-safe** (7 days for permanent tables) where only Snowflake support can recover it. 90 days requires Enterprise edition.
</details>

---

### Question 12
**Domain:** Domain 3 — Storage & Data Protection

A DBA runs `ALTER TABLE t1 SET DATA_RETENTION_TIME_IN_DAYS = 0`. What is the effect on existing micro-partitions?

- [ ] A. All historical micro-partitions are immediately purged.
- [ ] B. The setting change takes effect for future changes only; existing Time Travel data is preserved until its previous retention period expires.
- [ ] C. The table is converted to a Transient table automatically.
- [ ] D. Existing micro-partitions enter Fail-safe immediately upon the retention being set to 0.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The setting change takes effect for future changes only; existing Time Travel data is preserved until its previous retention period expires.

**Explanation:**
Changing `DATA_RETENTION_TIME_IN_DAYS` to 0 does not immediately purge existing historical micro-partitions. The **existing Time Travel data persists** until the end of the original retention window for those partitions. New changes after the setting change have 0-day retention. This protects against accidental data loss from a misconfigured retention change. The table remains a permanent table; it does not become Transient.
</details>

---

### Question 13
**Domain:** Domain 3 — Storage & Data Protection

A Snowflake stream is created on a table with `SHOW_INITIAL_ROWS = TRUE`. What does consuming the stream for the first time return?

- [ ] A. Only the rows inserted after stream creation.
- [ ] B. All rows that existed in the table at stream creation time as INSERT records, plus any subsequent DML.
- [ ] C. A schema snapshot of the table with column names but no data rows.
- [ ] D. An error, because SHOW_INITIAL_ROWS is only valid on external tables.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. All rows that existed in the table at stream creation time as INSERT records, plus any subsequent DML.

**Explanation:**
When `SHOW_INITIAL_ROWS = TRUE` is set on a stream, the **first consumption** of the stream returns all rows present in the source table at stream creation as synthetic INSERT change records (`METADATA$ACTION = 'INSERT'`). This is useful for bootstrapping a CDC pipeline without needing a separate initial load. Subsequent consumptions return only incremental changes as normal.
</details>

---

### Question 14
**Domain:** Domain 3 — Storage & Data Protection

A permanent table has been modified heavily for 92 days. The DBA queries it using `AT (OFFSET => -86400 * 92)` (92 days ago). The table has the default retention. What is the outcome?

- [ ] A. The query succeeds, returning the table state 92 days ago using Fail-safe data.
- [ ] B. The query fails with an error because Time Travel only persists for up to 90 days and the 92-day timestamp is outside the retention window.
- [ ] C. The query succeeds but returns empty results because old micro-partitions have been reclaimed.
- [ ] D. Snowflake automatically expands the retention window to accommodate the query.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The query fails with an error because Time Travel only persists for up to 90 days and the 92-day timestamp is outside the retention window.

**Explanation:**
**Time Travel** in Snowflake allows querying historical data up to the configured `DATA_RETENTION_TIME_IN_DAYS` (maximum 90 days on Enterprise+). A 92-day-old offset is outside the retention window, so Snowflake cannot fulfill the query and returns an error. After the Time Travel window expires, data moves to **Fail-safe** (7 days), which is not queryable by users — only Snowflake support can access it for disaster recovery.
</details>

---

### Question 15
**Domain:** Domain 3 — Storage & Data Protection

A developer creates a stream on an **append-only** table. They perform an UPDATE on a row. What does the stream capture?

- [ ] A. An UPDATE change record with the old and new values.
- [ ] B. Append-only streams do not support UPDATE operations; the DML statement fails.
- [ ] C. Nothing — append-only streams only capture INSERT operations; UPDATE and DELETE records are silently ignored.
- [ ] D. The UPDATE is recorded as a DELETE of the old row followed by an INSERT of the new row.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Nothing — append-only streams only capture INSERT operations; UPDATE and DELETE records are silently ignored.

**Explanation:**
An **append-only stream** (`STREAM_TYPE = 'append_only'` or `APPEND_ONLY = TRUE`) captures only **INSERT** operations on the source table. Updates and deletes are not tracked — they do not appear in the stream. This stream type is used for tables where data is only ever added (like raw event landing tables), providing a simpler and more performant change capture. The DML itself succeeds on the table; it just isn't reflected in the stream.
</details>

---

### Question 16
**Domain:** Domain 3 — Storage & Data Protection

A Snowflake table has a `ROW ACCESS POLICY` applied. A user attempts to clone the table using `CREATE TABLE t_clone CLONE t_source`. What happens to the row access policy?

- [ ] A. The clone has no row access policy; policies are not cloned with the table.
- [ ] B. The clone has the same row access policy reference attached; the policy enforces the same access rules on the clone.
- [ ] C. The clone gets a new, independent copy of the policy definition.
- [ ] D. The clone operation fails because tables with row access policies cannot be cloned.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The clone has the same row access policy reference attached; the policy enforces the same access rules on the clone.

**Explanation:**
When a table with a **row access policy** (RAP) is cloned, the **policy attachment is cloned too** — the clone references the same policy object. This means the same filtering logic applies to the clone. The policy itself is not duplicated; it's the same policy object enforced on both tables. This ensures governance controls are preserved by default in cloned environments. Administrators who want a clone without the RAP must detach it explicitly after cloning.
</details>

---

### Question 17
**Domain:** Domain 4 — Data Movement

A COPY INTO command loads a CSV file. Five rows have type conversion errors. The table has `ON_ERROR = CONTINUE`. What happens after the load?

- [ ] A. The entire file is rejected and no rows are loaded.
- [ ] B. The five error rows are skipped; the remaining rows in the file are loaded, and the error rows are recorded in LOAD_HISTORY.
- [ ] C. The five error rows are loaded with NULL values in the problematic columns.
- [ ] D. Snowflake retries each failing row three times before marking it as an error.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The five error rows are skipped; the remaining rows in the file are loaded, and the error rows are recorded in LOAD_HISTORY.

**Explanation:**
With `ON_ERROR = CONTINUE`, Snowflake skips rows with errors and continues loading the rest of the file. The errored rows count is visible in the query result and in `INFORMATION_SCHEMA.LOAD_HISTORY` / `COPY_HISTORY` view. No NULL substitution occurs — the rows are genuinely skipped. The option `ON_ERROR = SKIP_FILE` would skip the entire file on the first error; `ABORT_STATEMENT` would roll back the whole load.
</details>

---

### Question 18
**Domain:** Domain 4 — Data Movement

A Snowpipe is configured with `AUTO_INGEST = TRUE` on an S3 stage. The SQS notification queue for the bucket event has a 15-minute SQS message retention. New files arrive in the bucket but Snowpipe does not ingest them. After troubleshooting, the team finds the SQS queue is full. What is the recommended fix?

- [ ] A. Increase the Snowpipe credit limit to process the queue backlog faster.
- [ ] B. Use `ALTER PIPE my_pipe REFRESH` to reprocess files that were never ingested, as SQS notification loss means Snowpipe never received the file events.
- [ ] C. Re-upload the files with new names to trigger new SQS notifications.
- [ ] D. Increase the stage file size limit so fewer SQS messages are generated.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Use `ALTER PIPE my_pipe REFRESH` to reprocess files that were never ingested, as SQS notification loss means Snowpipe never received the file events.

**Explanation:**
When SQS messages are lost (queue full, expired, or misconfigured), Snowpipe never receives the trigger event for those files. `ALTER PIPE <pipe_name> REFRESH` causes Snowpipe to scan the stage path and queue any files that have not yet been loaded (using LOAD_HISTORY to deduplicate). This is the official recovery mechanism for missed auto-ingest notifications. Re-uploading with new names would work but creates unnecessary data duplication.
</details>

---

### Question 19
**Domain:** Domain 4 — Data Movement

A Snowflake external table is created on an S3 stage. The DBA runs `ALTER EXTERNAL TABLE ext_t REFRESH`. What does this command do?

- [ ] A. Reloads all data from the S3 files into Snowflake's internal storage.
- [ ] B. Synchronizes the external table's partition metadata with the current list of files in the stage path.
- [ ] C. Compresses all parquet files in the stage to reduce query scan costs.
- [ ] D. Forces the Cloud Services layer to re-parse the stage file format definition.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Synchronizes the external table's partition metadata with the current list of files in the stage path.

**Explanation:**
External tables store **metadata** about files in the stage (file paths, partition values, sizes). When new files are added to the stage or old ones are removed, the external table's catalog does not automatically update. `ALTER EXTERNAL TABLE ... REFRESH` scans the stage path and updates the metadata to match the current file listing. No data is copied into Snowflake — external tables always read from the stage at query time. For auto-refresh, event notifications can be configured.
</details>

---

### Question 20
**Domain:** Domain 4 — Data Movement

A Kafka connector writes to Snowflake using the **Snowflake Kafka Connector** in `SNOWPIPE` ingestion mode. The topic has 12 partitions. How many Snowpipe pipes are created internally?

- [ ] A. One pipe per topic.
- [ ] B. One pipe per Kafka partition (12 pipes).
- [ ] C. One pipe total, regardless of partitions.
- [ ] D. One pipe per consumer group member.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. One pipe per Kafka partition (12 pipes).

**Explanation:**
The **Snowflake Kafka Connector** creates one internal Snowpipe pipe per Kafka partition to maintain partition-level ordering guarantees and independent offset tracking. With 12 partitions, 12 internal pipes are created. This architecture ensures that a slow partition does not block other partitions and that exactly-once semantics are maintained per partition. The pipes are managed automatically and are visible in `SHOW PIPES`.
</details>

---

### Question 21
**Domain:** Domain 4 — Data Movement

A data engineer uses `COPY INTO @my_stage FROM SELECT * FROM sales WHERE region = 'APAC'`. The stage is an external Azure Blob stage. Which format options can be applied?

- [ ] A. Only CSV format is supported for COPY INTO stage exports.
- [ ] B. CSV, JSON, Parquet, ORC, and Avro are all supported for stage exports.
- [ ] C. Only Parquet format is supported when exporting to Azure Blob.
- [ ] D. Stage exports using subqueries require a virtual warehouse of at least LARGE size.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. CSV, JSON, Parquet, ORC, and Avro are all supported for stage exports.

**Explanation:**
`COPY INTO @stage` (unloading) supports multiple output file formats: **CSV, JSON, Parquet, ORC, and Avro**. The format is specified via `FILE_FORMAT` or inline `FORMAT_NAME`. There is no cloud-provider restriction on format — Azure Blob, S3, and GCS all support all formats. The subquery filter is fully valid. Warehouse size requirements depend on data volume, not the format.
</details>

---

### Question 22
**Domain:** Domain 4 — Data Movement

A Snowflake task runs `EXECUTE IMMEDIATE` with dynamic SQL. The task fails, and the error is not visible in `TASK_HISTORY`. Where should the engineer look?

- [ ] A. `INFORMATION_SCHEMA.QUERY_HISTORY` filtered by the task's scheduled time.
- [ ] B. The `RESULT_SCAN` of the last EXECUTE IMMEDIATE statement.
- [ ] C. The event table configured for the account, filtering by severity = ERROR.
- [ ] D. `SHOW TASKS` — failed task error messages are stored in the LAST_ERROR column.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. `SHOW TASKS` — failed task error messages are stored in the LAST_ERROR column.

**Explanation:**
`SHOW TASKS` includes a `LAST_ERROR` column (also visible as `last_error_time` and `last_error_message`) which records the most recent failure reason. Additionally, `TASK_HISTORY()` table function (from ACCOUNT_USAGE or INFORMATION_SCHEMA) includes `ERROR_CODE` and `ERROR_MESSAGE` columns. For tasks running stored procedures, the error propagates up. The event table captures telemetry emitted by the code but is not the primary place to see task execution errors.
</details>

---

### Question 23
**Domain:** Domain 5 — Account & Security

A Snowflake SCIM integration is configured with Okta. An admin deactivates a user in Okta. What happens in Snowflake?

- [ ] A. The user is deleted from Snowflake immediately.
- [ ] B. The user's Snowflake login is disabled (DISABLED = TRUE), preventing new sessions, but the user object is retained.
- [ ] C. All active sessions for the user are terminated immediately and the user is deleted.
- [ ] D. SCIM only syncs user creation; deactivation in the IdP has no effect in Snowflake.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The user's Snowflake login is disabled (DISABLED = TRUE), preventing new sessions, but the user object is retained.

**Explanation:**
When a user is deactivated in an IdP (like Okta) connected via **SCIM**, Snowflake sets `DISABLED = TRUE` on the user object. This prevents new logins but does **not** delete the user. Existing sessions may remain active until they timeout or are explicitly terminated. This behavior preserves audit trails and ownership of objects the user created. Admins must separately run `ALTER USER ... SET DISABLED = TRUE` or manage session termination if immediate revocation is required.
</details>

---

### Question 24
**Domain:** Domain 5 — Account & Security

A masking policy is defined as:

```sql
CREATE MASKING POLICY email_mask AS (val STRING) RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() IN ('DATA_ANALYST') THEN val
    ELSE '****'
  END;
```

A user with roles DATA_ANALYST and SYSADMIN queries the column. Their active role is SYSADMIN. What do they see?

- [ ] A. The unmasked value, because SYSADMIN always bypasses masking policies.
- [ ] B. `****`, because the active role is SYSADMIN, which is not in the allowed list.
- [ ] C. The unmasked value, because the user also has DATA_ANALYST, and masking checks all assigned roles.
- [ ] D. An error, because SYSADMIN cannot query masked columns without setting the masking role first.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. `****`, because the active role is SYSADMIN, which is not in the allowed list.

**Explanation:**
Dynamic Data Masking evaluates `CURRENT_ROLE()` — the **active primary role** at query time — not the full set of granted roles. Since the user's active role is SYSADMIN and the policy only unmasks for DATA_ANALYST, they see `****`. Even SYSADMIN and ACCOUNTADMIN are subject to masking policies unless explicitly listed. To see unmasked data, the user must switch their active role to DATA_ANALYST using `USE ROLE DATA_ANALYST`.
</details>

---

### Question 25
**Domain:** Domain 5 — Account & Security

A network policy is attached to a Snowflake user. The user's network policy allows `192.168.1.0/24`. The account-level network policy allows `10.0.0.0/8`. The user tries to log in from `192.168.1.50`. What is the result?

- [ ] A. Access is denied because 192.168.1.50 is not in the account-level allowed range.
- [ ] B. Access is granted because the user-level policy allows 192.168.1.0/24, and user-level policies take precedence over account-level.
- [ ] C. Access is denied because both policies must allow the IP simultaneously.
- [ ] D. Access is granted because the system unions all IP allowlists from both levels.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Access is granted because the user-level policy allows 192.168.1.0/24, and user-level policies take precedence over account-level.

**Explanation:**
When a **network policy** is assigned at both the user and account level, the **user-level policy takes precedence** for that user. If the user-level policy allows the IP, access is granted — the account-level policy is not additionally evaluated for that user. This allows exceptions to be granted to specific users without modifying the global policy. If no user-level policy exists, the account-level policy applies.
</details>

---

### Question 26
**Domain:** Domain 5 — Account & Security

Which Snowflake feature allows a data steward to centrally define which columns in which tables are treated as sensitive, and automatically enforce masking without touching each table definition individually?

- [ ] A. Column-level row access policies.
- [ ] B. Data classification with tag-based masking policies.
- [ ] C. Object-level privileges on masking policies.
- [ ] D. Secure views wrapping each sensitive table.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Data classification with tag-based masking policies.

**Explanation:**
Snowflake's **tag-based masking policies** allow administrators to attach a masking policy to a **tag** rather than to individual columns. When the tag (e.g., `PII`) is applied to a column, the associated masking policy is automatically enforced. Combined with **data classification** (which auto-applies system or custom tags to columns based on content patterns), this creates a scalable governance approach: classify once, enforce everywhere without manually altering each table.
</details>

---

### Question 27
**Domain:** Domain 5 — Account & Security

A role hierarchy has: PUBLIC → ANALYST → DATA_ENGINEER → SYSADMIN → ACCOUNTADMIN. A user is granted ANALYST. They run `USE ROLE DATA_ENGINEER`. What happens?

- [ ] A. The command succeeds because DATA_ENGINEER is a parent role of ANALYST.
- [ ] B. The command fails because the user has not been directly granted DATA_ENGINEER.
- [ ] C. The command succeeds because ACCOUNTADMIN inherits all roles and the user is implicitly granted everything.
- [ ] D. The command fails with an error stating that USE ROLE is not supported for child roles.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The command fails because the user has not been directly granted DATA_ENGINEER.

**Explanation:**
In Snowflake's **RBAC** model, role inheritance flows **downward** through grants: a role inherits privileges of its child roles, not its parent roles. A user granted ANALYST can activate ANALYST or any role that ANALYST has been granted. DATA_ENGINEER is a **parent** of ANALYST — the user cannot activate it unless directly granted. Only users explicitly granted DATA_ENGINEER (or a role higher in the hierarchy that was granted to them) can activate it.
</details>

---

### Question 28
**Domain:** Domain 5 — Account & Security

A Snowflake account enables **Tri-Secret Secure**. The customer's cloud KMS key is accidentally deleted. What is the result?

- [ ] A. Snowflake automatically falls back to Snowflake-managed encryption; data remains accessible.
- [ ] B. New data encryption continues using the Snowflake key only; historical data becomes inaccessible.
- [ ] C. All data in the account becomes immediately inaccessible because both keys are required to decrypt any data.
- [ ] D. Snowflake raises an alert but continues operating in degraded mode for 30 days.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. All data in the account becomes immediately inaccessible because both keys are required to decrypt any data.

**Explanation:**
**Tri-Secret Secure** requires **both** Snowflake's managed key AND the customer's KMS key to decrypt data. If the customer key is deleted or revoked, Snowflake cannot decrypt any data — including all historical data in the account. This is by design: it gives the customer the ability to "destroy" their data instantly by revoking/deleting the KMS key. Organizations using Tri-Secret Secure must have robust KMS key management (backups, recovery procedures) to avoid accidental data loss.
</details>

---

### Question 29
**Domain:** Domain 5 — Account & Security

A Snowflake admin wants to ensure that all service accounts authenticate using key-pair authentication and cannot use password-based login. Which combination of actions achieves this?

- [ ] A. Set `MUST_CHANGE_PASSWORD = TRUE` on the service account.
- [ ] B. Assign a public key to the user and set `PASSWORD = NULL` to disable password authentication.
- [ ] C. Apply a network policy that blocks all IP addresses except the application server IP.
- [ ] D. Set `DEFAULT_ROLE = PUBLIC` to minimize privilege and rely on MFA for additional security.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Assign a public key to the user and set `PASSWORD = NULL` to disable password authentication.

**Explanation:**
To enforce **key-pair only authentication**, administrators: (1) use `ALTER USER svc_account SET RSA_PUBLIC_KEY = '...'` to register the public key, and (2) use `ALTER USER svc_account SET PASSWORD = NULL` to remove the password, preventing password-based login. With no password set, the only authentication method available is key-pair (or other configured methods like OAuth). MUST_CHANGE_PASSWORD applies to password logins, not key-pair.
</details>

---

### Question 30
**Domain:** Domain 5 — Account & Security

A column projection policy is applied to a table column. What does it control, and how does it differ from a masking policy?

- [ ] A. It is an alias for masking policy; both control value visibility.
- [ ] B. A projection policy controls whether a column can be returned in a `SELECT *` or projected in any query at all — it can completely suppress column output; a masking policy modifies the displayed value but always returns the column.
- [ ] C. A projection policy hashes the column value; a masking policy nullifies it.
- [ ] D. Projection policies apply to views only; masking policies apply to base tables only.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. A projection policy controls whether a column can be returned in a `SELECT *` or projected in any query at all — it can completely suppress column output; a masking policy modifies the displayed value but always returns the column.

**Explanation:**
A **projection policy** (introduced in Snowflake to complement masking policies) controls whether a column can be **included in query output** at all. If the calling role is not allowed, the column is simply excluded from the result set — it cannot appear in SELECT, even as `SELECT *`. A **masking policy**, by contrast, always returns the column but may return a masked/transformed value. Projection policies are useful for completely hiding sensitive columns (e.g., raw SSN) from unauthorized roles without the column appearing as `NULL` or `****`.
</details>

---

### Question 31
**Domain:** Domain 6 — Performance Optimization

A query on a 10 TB fact table runs poorly. The query profile shows **"Partition pruning: 2% of partitions scanned"** — meaning 98% of partitions are scanned. What does this indicate, and what should be investigated?

- [ ] A. Pruning is excellent — only 2% of data is scanned. No action needed.
- [ ] B. The filter predicates do not align with the table's clustering key (or no clustering key exists), causing full table scans. Investigate the clustering key and query predicates.
- [ ] C. The warehouse is too small; upgrading to a larger size will improve partition elimination.
- [ ] D. The table should be materialized into a smaller view to reduce scan scope.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The filter predicates do not align with the table's clustering key (or no clustering key exists), causing full table scans. Investigate the clustering key and query predicates.

**Explanation:**
"2% of partitions pruned" means **98% are being scanned** — a very poor pruning ratio. This typically means the table is either not clustered, or the clustering key does not match the query's filter predicates. The optimizer cannot eliminate micro-partitions because the data is not organized in a way that correlates with the WHERE clause. The fix is to define or redefine the clustering key to match the dominant filter columns, reducing the number of micro-partitions that need to be scanned.
</details>

---

### Question 32
**Domain:** Domain 6 — Performance Optimization

A query profile shows an operator with the label **"Aggregate"** and a very high **"Bytes spilled to remote disk"** metric. What is the most likely cause and fix?

- [ ] A. The GROUP BY has too many rows; add a LIMIT clause to reduce output.
- [ ] B. The aggregation is exceeding the warehouse's local SSD cache and spilling to object storage; increase the warehouse size to provide more memory for the in-memory hash aggregation.
- [ ] C. Remote disk spill is normal for aggregates; it indicates the result cache was bypassed.
- [ ] D. The spill is caused by an inefficient join; rewrite the subquery as a CTE.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The aggregation is exceeding the warehouse's local SSD cache and spilling to object storage; increase the warehouse size to provide more memory for the in-memory hash aggregation.

**Explanation:**
**Spilling to remote disk** (object storage) in Snowflake's query profile indicates the warehouse ran out of local memory AND local SSD cache for an operation (aggregation, sort, or join) and had to write intermediate results to remote cloud storage. This is the most expensive spill tier and significantly degrades performance. The primary fix is to **increase the warehouse size** (more memory per node). For aggregations, also consider pre-aggregating data or using approximate functions (`APPROX_COUNT_DISTINCT`) where exact counts aren't needed.
</details>

---

### Question 33
**Domain:** Domain 6 — Performance Optimization

Two queries in the same session run `SELECT * FROM large_table WHERE col = 'X'` and `SELECT * FROM large_table WHERE col = 'Y'`. The second query is faster. What is the most likely explanation?

- [ ] A. The result cache served the second query.
- [ ] B. The warehouse local disk (SSD) cache retained the scanned micro-partitions from the first query, allowing the second query to read from local cache instead of remote storage.
- [ ] C. Snowflake's optimizer rewrote the second query to use a pre-existing materialized view.
- [ ] D. The second query has a shorter WHERE clause, which is faster to parse.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The warehouse local disk (SSD) cache retained the scanned micro-partitions from the first query, allowing the second query to read from local cache instead of remote storage.

**Explanation:**
Snowflake warehouses have a **local SSD cache** that retains recently accessed micro-partition data. When the second query accesses the same `large_table`, many of the micro-partitions were already cached locally (assuming the table's micro-partitions overlap for different column values). This reduces remote I/O dramatically. The result cache only applies when the exact same query returns the exact same result — different WHERE clause values produce different results, so the result cache doesn't apply here.
</details>

---

### Question 34
**Domain:** Domain 6 — Performance Optimization

A developer runs `SELECT a.*, b.col FROM fact a JOIN dim b ON a.id = b.id` and the query profile shows the join type as **"Hash Join (Broadcast)"**. What does this mean?

- [ ] A. The larger table (fact) was broadcast to all warehouse nodes; the smaller table was probed.
- [ ] B. The smaller table (dim) was broadcast in its entirety to all warehouse nodes; the larger table (fact) was partitioned and probed locally.
- [ ] C. A Hash Join Broadcast means the join was executed on the Cloud Services layer without warehouse compute.
- [ ] D. Broadcast joins are only used when both tables are the same size.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The smaller table (dim) was broadcast in its entirety to all warehouse nodes; the larger table (fact) was partitioned and probed locally.

**Explanation:**
In a **Broadcast Hash Join**, Snowflake detects that one side of the join (typically the dimension table) is small enough to broadcast its entire content to all nodes. Each node then has a complete copy of the dimension and can locally probe it against its partition of the fact table. This avoids expensive data shuffling (redistribution) of the large fact table. Broadcast joins are highly efficient for small-large table joins (classic star schema patterns).
</details>

---

### Question 35
**Domain:** Domain 1 — Architecture

A developer defines a **Dynamic Table** with `TARGET_LAG = '1 hour'`. A base table is updated 5 times within 10 minutes. How many refreshes of the dynamic table are triggered?

- [ ] A. Five refreshes — one per base table update.
- [ ] B. One refresh — Snowflake batches changes and refreshes once, respecting the 1-hour target lag.
- [ ] C. Zero refreshes — Dynamic tables only refresh on demand via ALTER DYNAMIC TABLE REFRESH.
- [ ] D. Exactly one refresh per hour regardless of base table change frequency.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. One refresh — Snowflake batches changes and refreshes once, respecting the 1-hour target lag.

**Explanation:**
**Dynamic tables** use a **TARGET_LAG** parameter that defines the maximum acceptable staleness. Snowflake's scheduler determines the optimal refresh frequency to meet the lag target. It does **not** trigger a refresh per base table DML event — it batches changes and refreshes as needed (at most once per lag period). Five updates in 10 minutes would be picked up in a single incremental refresh the next time the scheduler runs. The actual lag may be less than 1 hour; it will never exceed 1 hour (under normal conditions).
</details>

---

### Question 36
**Domain:** Domain 1 — Architecture

A Snowflake Cortex function `SNOWFLAKE.CORTEX.COMPLETE()` is called inside a SQL query. What is consumed to execute this function?

- [ ] A. Credits from the virtual warehouse running the query.
- [ ] B. Snowflake Cortex serverless credits billed separately from the virtual warehouse.
- [ ] C. External API tokens from the connected LLM provider account.
- [ ] D. Credits from the Cloud Services layer budget.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowflake Cortex serverless credits billed separately from the virtual warehouse.

**Explanation:**
**Snowflake Cortex LLM functions** (like `COMPLETE`, `SUMMARIZE`, `CLASSIFY_TEXT`, `TRANSLATE`) run on Snowflake's **serverless AI infrastructure** and are billed in **Snowflake Cortex credits** — a separate credit pool distinct from virtual warehouse credits. The calling query still runs on a warehouse (for surrounding SQL), but the LLM inference itself consumes Cortex serverless credits based on the model used and the number of tokens processed. Pricing varies by model.
</details>

---

### Question 37
**Domain:** Domain 1 — Architecture

A Snowflake **Iceberg Table** is created using Snowflake as the catalog. What is stored in Snowflake's internal storage vs. external object storage?

- [ ] A. All data and metadata are stored in Snowflake's internal storage; the Iceberg format is only used for compatibility exports.
- [ ] B. Data files (Parquet) are in external object storage; Iceberg metadata (manifests, metadata JSON) and the catalog entry are managed by Snowflake.
- [ ] C. Only metadata is stored in Snowflake; data files are managed entirely by the external Iceberg catalog.
- [ ] D. Iceberg tables store data in Snowflake's micro-partition format but export Iceberg manifests on demand.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Data files (Parquet) are in external object storage; Iceberg metadata (manifests, metadata JSON) and the catalog entry are managed by Snowflake.

**Explanation:**
When Snowflake acts as the **Iceberg catalog**, data is written as **Parquet files to the configured external volume** (S3, GCS, or Azure). Snowflake manages the Iceberg **metadata layer** (snapshot JSON, manifest lists, manifest files) and the catalog registration. This means other tools can also read the table via the Iceberg open format. For the alternative — using an external catalog like AWS Glue or Apache Polaris — Snowflake acts as a query engine reading externally-managed Iceberg metadata.
</details>

---

### Question 38
**Domain:** Domain 2 — Virtual Warehouses

A warehouse is **SUSPENDED**. A user submits a query. What happens to warehouse resume time and the query?

- [ ] A. The query fails immediately with an error; the user must manually resume the warehouse first.
- [ ] B. The warehouse resumes automatically (`AUTO_RESUME = TRUE` by default) and the query waits in queue until the warehouse is ready, then executes.
- [ ] C. The warehouse resumes, but the query is dropped; the user must resubmit.
- [ ] D. The query is routed to the Cloud Services layer for execution without spinning up the warehouse.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The warehouse resumes automatically (`AUTO_RESUME = TRUE` by default) and the query waits in queue until the warehouse is ready, then executes.

**Explanation:**
By default, Snowflake warehouses are created with `AUTO_RESUME = TRUE`. When a query is submitted against a suspended warehouse with AUTO_RESUME enabled, Snowflake automatically resumes the warehouse. The query is queued and begins executing once the warehouse is available (typically within seconds). Resume time is usually 1–5 seconds. If `AUTO_RESUME = FALSE`, the query would fail. Queries are never dropped silently — they either queue, fail, or execute.
</details>

---

### Question 39
**Domain:** Domain 3 — Storage & Data Protection

A developer creates a **hybrid table** in Snowflake. What is the primary architectural difference from a standard Snowflake table?

- [ ] A. Hybrid tables store data in both Snowflake's columnar micro-partition format AND a row-oriented store, enabling fast point lookups and low-latency transactional operations.
- [ ] B. Hybrid tables support both structured and semi-structured data in the same column.
- [ ] C. Hybrid tables use external storage for cold data and internal storage for hot data, automatically tiering.
- [ ] D. Hybrid tables are a deprecated alias for Transient tables.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Hybrid tables store data in both Snowflake's columnar micro-partition format AND a row-oriented store, enabling fast point lookups and low-latency transactional operations.

**Explanation:**
**Hybrid tables** (part of Snowflake's Unistore initiative) maintain a **dual storage format**: columnar storage for analytical queries (like standard Snowflake tables) and a **row-oriented store** for fast single-row lookups, primary key enforcement, and low-latency OLTP-style operations. They support unique constraints, foreign keys, and secondary indexes — features not available on standard tables. This enables mixed HTAP (Hybrid Transactional/Analytical Processing) workloads within a single Snowflake table.
</details>

---

### Question 40
**Domain:** Domain 4 — Data Movement

A developer runs a Snowflake `MERGE` statement on a table with 50 million rows. The source has 1 million rows. The query runs significantly slower than expected. The query profile shows a **cartesian join** warning. What is the most likely cause?

- [ ] A. The MERGE statement does not support tables over 10 million rows.
- [ ] B. The MERGE join condition is non-deterministic — a source row matches multiple target rows, causing a row-level expansion. Snowflake raises a non-determinism warning in the profile.
- [ ] C. A cartesian join always appears in MERGE statements because the optimizer doesn't know join cardinality at plan time.
- [ ] D. The source table lacks a primary key, which disables MERGE optimization.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The MERGE join condition is non-deterministic — a source row matches multiple target rows, causing a row-level expansion. Snowflake raises a non-determinism warning in the profile.

**Explanation:**
Snowflake's `MERGE` requires a **1-to-1 or many-to-1** relationship between source and target rows in the MATCHED clause. If one source row matches **multiple target rows**, Snowflake detects **non-determinism** and may raise an error or warning (depending on the `ERROR_ON_NONDETERMINISTIC_MERGE` parameter). The underlying join can degenerate into a cartesian-like expansion, massively inflating intermediate result sets and causing slowness. Fix: ensure the join key uniquely identifies target rows, or use deduplication in the source.
</details>

---

### Question 41
**Domain:** Domain 5 — Account & Security

A Snowflake admin creates an **API Integration** for an external function. Which Snowflake-managed entity is created automatically to authenticate calls to the external API endpoint?

- [ ] A. A Snowflake service account with username/password credentials stored in Secrets Manager.
- [ ] B. A cloud provider IAM role or service principal that Snowflake assumes when calling the external API endpoint.
- [ ] C. A JWT token hardcoded in the integration definition.
- [ ] D. A network policy restricting outbound traffic to the external endpoint.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. A cloud provider IAM role or service principal that Snowflake assumes when calling the external API endpoint.

**Explanation:**
When an **API Integration** is created, Snowflake generates a **cloud IAM principal** (e.g., an AWS IAM role ARN or Azure service principal object ID) that the customer must authorize in their cloud environment. When Snowflake calls the external API (via API Gateway), it assumes this IAM role, and the API Gateway can verify the caller's identity using standard cloud IAM. This avoids storing static credentials. The admin retrieves the IAM role ARN/principal ID from `DESCRIBE INTEGRATION` to configure the trust policy.
</details>

---

### Question 42
**Domain:** Domain 5 — Account & Security

A Snowflake user's `DEFAULT_SECONDARY_ROLES` is set to `ALL`. What is the effect when the user creates a new session?

- [ ] A. The user's active role is set to all roles simultaneously.
- [ ] B. All roles granted to the user are activated as secondary roles in the session, supplementing the primary role's privileges without replacing it.
- [ ] C. The user is granted SYSADMIN as their secondary role by default.
- [ ] D. Secondary roles are deprecated in Snowflake; the setting has no effect.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. All roles granted to the user are activated as secondary roles in the session, supplementing the primary role's privileges without replacing it.

**Explanation:**
`DEFAULT_SECONDARY_ROLES = ALL` activates all roles granted to the user as **secondary roles** at session start. Secondary roles provide their privileges on top of the primary role — the union of all privileges is available. This means a user with primary role ANALYST and secondary roles DATA_ENGINEER and SYSADMIN (via ALL) can access objects owned by any of those roles without switching. The primary role (DEFAULT_ROLE) remains the role used for ownership of new objects. `USE SECONDARY ROLES ALL` achieves this at the session level interactively.
</details>

---

### Question 43
**Domain:** Domain 1 — Architecture

A Snowflake **Cortex Search** service is created on a text column. What type of index does Cortex Search build, and what query capability does it enable?

- [ ] A. A B-tree index on the text column enabling fast exact-match lookups.
- [ ] B. A full-text inverted index for keyword search using SQL LIKE predicates.
- [ ] C. A vector embedding index enabling semantic similarity search (nearest-neighbor) using natural language queries.
- [ ] D. A columnar bloom filter for approximate string matching.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. A vector embedding index enabling semantic similarity search (nearest-neighbor) using natural language queries.

**Explanation:**
**Snowflake Cortex Search** builds a managed **vector embedding index** over the specified text column(s). At query time, a natural language query is embedded using the same model, and the index returns the most semantically similar documents (approximate nearest-neighbor search). This enables RAG (Retrieval-Augmented Generation) workflows directly in Snowflake without managing a separate vector database. It supports hybrid search (semantic + keyword) and is serverless — no warehouse required for the index or search queries.
</details>

---

### Question 44
**Domain:** Domain 2 — Virtual Warehouses

A developer enables `QUERY_TAG` at the session level to track application queries. After the session ends, where can the query tag be found?

- [ ] A. Only in the query history of the session's warehouse via SHOW QUERIES.
- [ ] B. In `SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY` in the `QUERY_TAG` column, persisted for up to 365 days.
- [ ] C. In the `QUERY_TAG` system variable only during the session.
- [ ] D. In the warehouse event log, accessible via `INFORMATION_SCHEMA.WAREHOUSE_EVENTS_HISTORY`.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. In `SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY` in the `QUERY_TAG` column, persisted for up to 365 days.

**Explanation:**
`QUERY_TAG` is a session parameter that attaches a metadata label to all queries run in the session (or can be set per-statement). After the session ends, the query tag is persisted in the `QUERY_TAG` column of `SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY`, which retains data for **up to 1 year (365 days)**. This is useful for cost attribution (tagging queries by application, team, or job ID) and analyzing query patterns by workload type.
</details>

---

### Question 45
**Domain:** Domain 3 — Storage & Data Protection

A Snowflake table column is defined as `VARIANT`. A JSON document with 1,000 keys is inserted. How does Snowflake store this internally?

- [ ] A. As a single opaque BLOB column in the micro-partition.
- [ ] B. Snowflake automatically shreds the VARIANT into separate columnar sub-columns for known paths, enabling columnar statistics and pruning on nested fields.
- [ ] C. Snowflake stores VARIANT columns as compressed JSON strings with no columnar decomposition.
- [ ] D. VARIANT columns are stored in a separate document store layer outside the main micro-partition structure.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowflake automatically shreds the VARIANT into separate columnar sub-columns for known paths, enabling columnar statistics and pruning on nested fields.

**Explanation:**
Snowflake's columnar storage engine performs **automatic JSON shredding** on VARIANT columns. Commonly occurring key paths are extracted and stored as typed columnar sub-columns within the micro-partition. This allows the query engine to push down predicates (e.g., `WHERE v:country = 'US'`) to the storage layer and prune micro-partitions using columnar statistics — just like a native typed column. Less common or deeply nested paths are stored as a compressed residual. This hybrid approach delivers near-native performance on VARIANT queries.
</details>

---

### Question 46
**Domain:** Domain 4 — Data Movement

A `GET_DDL('TABLE', 'my_table')` is run on a table. Which of the following is NOT included in the output?

- [ ] A. Column names, data types, and NOT NULL constraints.
- [ ] B. Clustering key definition (if one exists).
- [ ] C. Row-level access policies attached to the table.
- [ ] D. STAGE FILE FORMAT options used when the table was originally created.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Row-level access policies attached to the table.

**Explanation:**
`GET_DDL` returns the **DDL statement** needed to recreate the table's structure: column definitions, constraints, clustering keys, table-level properties (like DATA_RETENTION_TIME_IN_DAYS), and comment. It does **not** include security policy attachments (row access policies, masking policies) because policies are attached separately and may be managed by different roles. To document policy attachments, you must query `SNOWFLAKE.ACCOUNT_USAGE.POLICY_REFERENCES` or `INFORMATION_SCHEMA.POLICY_REFERENCES`.
</details>

---

### Question 47
**Domain:** Domain 5 — Account & Security

An admin grants `MONITOR USAGE` privilege to a role. What does this allow?

- [ ] A. The role can monitor individual query execution plans for all users.
- [ ] B. The role can view account-level credit usage, storage usage, and warehouse usage history — enabling billing visibility without admin privileges.
- [ ] C. The role can monitor all active sessions and kill any query.
- [ ] D. The role can view and modify warehouse credit limits.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The role can view account-level credit usage, storage usage, and warehouse usage history — enabling billing visibility without admin privileges.

**Explanation:**
`MONITOR USAGE` is an account-level privilege that grants visibility into **consumption and billing data**: credit usage by warehouse, storage costs, and cloud service usage. It enables the grantee to query `SNOWFLAKE.ACCOUNT_USAGE` views like `WAREHOUSE_METERING_HISTORY` and `STORAGE_USAGE`. It does **not** grant the ability to kill queries (that requires OPERATE on the warehouse), view query plans, or modify warehouse settings. It's ideal for FinOps or cost management roles.
</details>

---

### Question 48
**Domain:** Domain 6 — Performance Optimization

An analyst reports that a query using `FLATTEN(INPUT => v, RECURSIVE => TRUE)` on a large VARIANT column is very slow. What is the most effective optimization?

- [ ] A. Convert the VARIANT column to a ARRAY type for faster flattening.
- [ ] B. Pre-materialize the flattened output into a relational table or dynamic table, eliminating the runtime cost of recursive JSON expansion.
- [ ] C. Add a search optimization service to the VARIANT column.
- [ ] D. Increase the warehouse size; FLATTEN is CPU-bound and scales linearly with warehouse size.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Pre-materialize the flattened output into a relational table or dynamic table, eliminating the runtime cost of recursive JSON expansion.

**Explanation:**
`FLATTEN(..., RECURSIVE => TRUE)` on large, deeply nested VARIANT structures is computationally expensive because it must expand every level of nesting at query time. The most effective optimization is to **pre-materialize** the flattened, structured result into a relational table (via CTAS, a scheduled Task, or a Dynamic Table). Subsequent queries read typed columns rather than performing runtime JSON expansion. A Search Optimization service does not help with FLATTEN; it targets specific path lookups, not full recursive expansion.
</details>

---

### Question 49
**Domain:** Domain 1 — Architecture

A Snowflake account enables the **ENABLE_INTERNAL_STAGES_PRIVATELINK** parameter. What is the effect?

- [ ] A. Internal stages are encrypted with a customer-managed key.
- [ ] B. Data uploaded to or downloaded from Snowflake internal stages traverses the cloud provider's private network (PrivateLink / Private Endpoint) instead of the public internet.
- [ ] C. Internal stage URLs are only accessible from within the Snowflake account and cannot be shared externally.
- [ ] D. Internal stages are replicated across regions automatically.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Data uploaded to or downloaded from Snowflake internal stages traverses the cloud provider's private network (PrivateLink / Private Endpoint) instead of the public internet.

**Explanation:**
`ENABLE_INTERNAL_STAGES_PRIVATELINK` ensures that even **internal stage traffic** (PUT/GET commands via SnowSQL or drivers) uses the **private endpoint** configured for the account (AWS PrivateLink, Azure Private Link, or GCP Private Service Connect). Without this setting, internal stage operations may use public S3/GCS/Azure URLs. This is important for compliance environments that require all data movement to avoid the public internet. The parameter requires a PrivateLink/Private Endpoint configuration on the account.
</details>

---

### Question 50
**Domain:** Domain 1 — Architecture

Which statement about Snowflake's **search optimization service** is ACCURATE?

- [ ] A. Search optimization is automatic and free; it applies to all tables once enabled at the account level.
- [ ] B. Search optimization creates persistent access path structures (similar to secondary indexes) for specific query predicates (equality, range, substring) on specific columns, consuming additional storage.
- [ ] C. Search optimization improves query performance for all query types including aggregations and joins.
- [ ] D. Search optimization can only be applied to VARCHAR columns, not to numeric or DATE types.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Search optimization creates persistent access path structures (similar to secondary indexes) for specific query predicates (equality, range, substring) on specific columns, consuming additional storage.

**Explanation:**
**Search Optimization Service** builds persistent, server-side **access paths** (similar to secondary indexes) for specific columns and predicate types: equality (`=`, `IN`), range (`<`, `>`, `BETWEEN`), substring/regex (`LIKE`, `ILIKE`, `REGEXP`), and semi-structured path lookups. It must be explicitly enabled (`ALTER TABLE ... ADD SEARCH OPTIMIZATION`) and consumes additional storage (billed separately). It is most effective for selective point-lookup queries on large tables where partition pruning alone is insufficient. It does not improve aggregations or joins.
</details>

---

### Question 51
**Domain:** Domain 3 — Storage & Data Protection

A table has **Automatic Clustering** enabled. After a large bulk INSERT, the `SYSTEM$CLUSTERING_INFORMATION()` output shows a very high **average overlap depth**. What does this indicate?

- [ ] A. The table is well-clustered; a high overlap depth means micro-partitions are tightly organized.
- [ ] B. The table is poorly clustered; a high average overlap depth means many micro-partitions contain overlapping ranges for the clustering key, requiring more partitions to be scanned per query.
- [ ] C. Overlap depth measures row duplication rate, not clustering quality.
- [ ] D. Average overlap depth above 1 triggers automatic re-clustering without additional cost.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The table is poorly clustered; a high average overlap depth means many micro-partitions contain overlapping ranges for the clustering key, requiring more partitions to be scanned per query.

**Explanation:**
**Average overlap depth** is a key clustering quality metric. It measures how many micro-partitions, on average, contain any given value of the clustering key. A depth of 1 means perfect clustering — each value appears in exactly one partition. A high depth (e.g., 50) means that a query filtering on a single key value must scan 50 micro-partitions on average. After a large bulk INSERT (especially out-of-order data), overlap depth increases. Automatic clustering will work to reduce it over time by re-clustering affected ranges.
</details>

---

### Question 52
**Domain:** Domain 2 — Virtual Warehouses

A Snowflake account uses **Resource Monitors** with `NOTIFY_USERS` action at 80% and `SUSPEND_IMMEDIATE` at 100% of the credit quota. A warehouse hits 100%. What happens to currently running queries?

- [ ] A. Currently running queries complete normally; only new queries are rejected after the warehouse suspends.
- [ ] B. Currently running queries are immediately terminated mid-execution, and the warehouse suspends.
- [ ] C. Currently running queries are paused and can be resumed when the quota resets.
- [ ] D. The warehouse enters a read-only mode; running queries continue but write operations are aborted.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Currently running queries are immediately terminated mid-execution, and the warehouse suspends.

**Explanation:**
`SUSPEND_IMMEDIATE` is the more aggressive action: the warehouse is **immediately suspended** and **all currently running queries are terminated**, even mid-execution. This is in contrast to `SUSPEND`, which waits for running queries to complete before suspending. `SUSPEND_IMMEDIATE` is appropriate for hard budget caps where overspending is unacceptable. Terminated queries do not consume credits for completed work (partial credit consumption up to the point of termination is billed). Administrators must manually resume the warehouse after quota resets.
</details>

---

### Question 53
**Domain:** Domain 4 — Data Movement

A developer uses `SYSTEM$PIPE_STATUS('my_pipe')` and sees `pendingFileCount: 250`. What does this indicate?

- [ ] A. 250 files have been successfully loaded by Snowpipe.
- [ ] B. 250 files are staged and waiting to be processed by Snowpipe but have not yet been loaded.
- [ ] C. 250 rows are pending schema validation before loading.
- [ ] D. 250 files were rejected by Snowpipe and are awaiting manual review.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. 250 files are staged and waiting to be processed by Snowpipe but have not yet been loaded.

**Explanation:**
`SYSTEM$PIPE_STATUS()` returns a JSON object with operational metrics for a Snowpipe. `pendingFileCount` represents files that have been queued for ingestion but not yet processed. A high pending count can indicate Snowpipe is backlogged (due to file volume, file size, or a processing issue). Other fields include `notProcessedFileCount` (files that failed), `lastIngestedTimestamp`, and `executionState`. Monitoring `pendingFileCount` over time is a key Snowpipe health indicator.
</details>

---

### Question 54
**Domain:** Domain 5 — Account & Security

A Snowflake admin creates a secret object (`CREATE SECRET`) to store an OAuth client credential. Which Snowflake feature primarily uses secret objects?

- [ ] A. Network policies — secrets define IP allowlists.
- [ ] B. External functions and Snowpark Container Services — secrets provide credentials for external API calls and containerized service authentication without embedding credentials in code.
- [ ] C. Masking policies — secrets store the encryption key for masked columns.
- [ ] D. Time Travel — secrets authenticate historical query access.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. External functions and Snowpark Container Services — secrets provide credentials for external API calls and containerized service authentication without embedding credentials in code.

**Explanation:**
**Secret objects** in Snowflake (`CREATE SECRET`) store sensitive credentials (OAuth tokens, username/password, generic strings) in Snowflake's encrypted secrets store. They are used by external functions, Snowpark Container Services (SPCS) service bindings, and external network access integrations to authenticate to external APIs and services. Secrets are referenced by name in integrations; the actual credential values are never exposed in SQL. Access to a secret is controlled by standard Snowflake RBAC (USAGE privilege on the secret).
</details>

---

### Question 55
**Domain:** Domain 1 — Architecture

A Snowflake **Notebook** is used by a data scientist. The notebook runs Python cells. Which underlying execution environment is used for the Python cells?

- [ ] A. The Python cells execute in the Cloud Services layer using Snowflake's managed Python runtime.
- [ ] B. The Python cells execute in a virtual warehouse using Snowpark-Optimized nodes with the Snowflake Anaconda Python environment.
- [ ] C. Python cells require an external Jupyter kernel connected via PrivateLink.
- [ ] D. Python cells execute only as client-side code; they cannot access Snowflake data directly.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The Python cells execute in a virtual warehouse using Snowpark-Optimized nodes with the Snowflake Anaconda Python environment.

**Explanation:**
**Snowflake Notebooks** (Snowsight-native notebooks) run Python cells using the **Snowpark Python runtime** on a connected virtual warehouse (or a Snowpark-Optimized warehouse for ML workloads). The environment includes the Snowflake-curated Anaconda package distribution. SQL cells run on the same warehouse. Notebooks have direct access to Snowflake data via the Snowpark session — no separate kernel or external connectivity is required. The warehouse must be active for Python/SQL cells to execute.
</details>

---

### Question 56
**Domain:** Domain 4 — Data Movement

A developer uses `VALIDATE(my_table, JOB_ID => '<query_id>')`. What does this function do?

- [ ] A. Validates the schema of my_table against a JSON Schema definition.
- [ ] B. Returns all rows that were rejected (errored) during the COPY INTO load job identified by the query_id.
- [ ] C. Checks whether a table has valid primary key constraints.
- [ ] D. Runs a statistical sample query on my_table and validates the results against expected distributions.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Returns all rows that were rejected (errored) during the COPY INTO load job identified by the query_id.

**Explanation:**
The `VALIDATE()` table function returns the **error details for rows rejected during a COPY INTO operation**, identified by the original load query's UUID. It is useful for diagnosing data quality issues after a load with `ON_ERROR = CONTINUE` or `ON_ERROR = SKIP_FILE`. The output includes the rejected row's raw content, the error type, the file name, and the row/column position of the error. This is distinct from `COPY_HISTORY` which provides file-level statistics.
</details>

---

### Question 57
**Domain:** Domain 3 — Storage & Data Protection

A Snowflake **materialized view** is defined on a base table. The base table is updated with a large batch DML. What happens to the materialized view?

- [ ] A. The materialized view is immediately updated synchronously as part of the DML transaction.
- [ ] B. The materialized view is marked stale and is asynchronously updated by Snowflake's background service; queries against it may see slightly stale data until the refresh completes.
- [ ] C. The materialized view is dropped automatically and must be recreated.
- [ ] D. The materialized view is suspended until the DBA runs ALTER MATERIALIZED VIEW RESUME.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The materialized view is marked stale and is asynchronously updated by Snowflake's background service; queries against it may see slightly stale data until the refresh completes.

**Explanation:**
Snowflake **materialized views** are maintained asynchronously by a background Snowflake-managed service (consuming serverless credits). After a DML on the base table, the materialized view is not immediately updated — it is queued for incremental refresh. Queries against the MV may return slightly stale results during the refresh window. This is a key trade-off vs. dynamic tables, which offer more explicit lag control. For very large DML batches, refresh latency may be noticeable.
</details>

---

### Question 58
**Domain:** Domain 2 — Virtual Warehouses

A query has `RESULT_SCAN(LAST_QUERY_ID())` in a subsequent statement. The user switched to a different warehouse between the two statements. Does `RESULT_SCAN` still work?

- [ ] A. No — RESULT_SCAN only works on the same warehouse that ran the original query.
- [ ] B. Yes — the query result cache is in the Cloud Services layer and is not tied to any warehouse; RESULT_SCAN can access it regardless of warehouse changes.
- [ ] C. No — switching warehouses invalidates the result cache for the session.
- [ ] D. Yes, but only if both warehouses belong to the same warehouse group.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Yes — the query result cache is in the Cloud Services layer and is not tied to any warehouse; RESULT_SCAN can access it regardless of warehouse changes.

**Explanation:**
`RESULT_SCAN()` accesses the **Cloud Services query result cache**, which stores result sets for 24 hours. This cache is independent of any warehouse — it is managed at the Cloud Services layer. Switching warehouses between statements does not invalidate the cache or prevent `RESULT_SCAN` from accessing results from a different warehouse's query. The only requirements are that the query ID is valid and the result is still within the 24-hour retention window.
</details>

---

### Question 59
**Domain:** Domain 5 — Account & Security

A Snowflake OAuth integration is configured for a BI tool. A user authenticates via OAuth and gets a refresh token. The Snowflake admin wants to revoke access for a specific user without deleting the user. What is the correct action?

- [ ] A. Revoke the user's DEFAULT_ROLE.
- [ ] B. Run `SELECT SYSTEM$REVOKE_OAUTH_TOKEN('token_value')` to invalidate the specific token.
- [ ] C. Disable the OAuth security integration entirely.
- [ ] D. Use `ALTER USER <name> SET MINS_TO_BYPASS_NETWORK_POLICY = 0` to block the session.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Run `SELECT SYSTEM$REVOKE_OAUTH_TOKEN('token_value')` to invalidate the specific token.

**Explanation:**
Snowflake provides `SYSTEM$REVOKE_OAUTH_TOKEN()` to revoke a specific OAuth refresh or access token for a user without affecting other users or disabling the integration. The token string is found in `SNOWFLAKE.ACCOUNT_USAGE.OAUTH_TOKEN_HISTORY`. Disabling the integration would revoke access for ALL users of that integration. Revoking the role doesn't invalidate the active OAuth token immediately. This is the targeted, least-impact approach to revoking a single user's OAuth access.
</details>

---

### Question 60
**Domain:** Domain 1 — Architecture

Snowflake's **Fail-safe** period begins after Time Travel expires. Who can recover data from the Fail-safe period?

- [ ] A. Any user with the ACCOUNTADMIN role can run `SELECT ... AT (BEFORE => FAIL_SAFE)` queries.
- [ ] B. Only Snowflake Support can recover data from Fail-safe; there is no customer-facing API.
- [ ] C. The SYSADMIN role can recover Fail-safe data using `UNDROP TABLE ... FAILSAFE`.
- [ ] D. Fail-safe data is automatically recovered by Snowflake after 7 days with no action required.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Only Snowflake Support can recover data from Fail-safe; there is no customer-facing API.

**Explanation:**
The **Fail-safe** period (7 days for permanent tables, 0 days for transient/temporary) is **not accessible to customers** via any SQL command. It is a disaster recovery mechanism that Snowflake Support can use to recover data on behalf of a customer in extreme scenarios. Customers cannot run Time Travel queries against Fail-safe data. This is a key distinction: Time Travel is customer-accessible; Fail-safe is Snowflake-internal. This is why minimizing unnecessary retention beyond the required Time Travel window helps control storage costs.
</details>

---

### Question 61
**Domain:** Domain 4 — Data Movement

A developer needs to load semi-structured data where each file contains a JSON array at the top level (not newline-delimited). Which FILE FORMAT option must be set?

- [ ] A. `STRIP_OUTER_ARRAY = TRUE` — removes the outer JSON array and treats each element as a separate record.
- [ ] B. `ARRAY_DELIMITED = TRUE` — specifies that the file uses array delimiters.
- [ ] C. `JSON_FORMAT = ARRAY` — switches the JSON parser to array mode.
- [ ] D. No special setting required; Snowflake automatically detects JSON arrays.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. `STRIP_OUTER_ARRAY = TRUE` — removes the outer JSON array and treats each element as a separate record.

**Explanation:**
When a JSON file contains a top-level array (e.g., `[{"id":1},{"id":2}]`), Snowflake would by default load the entire array as a single VARIANT row. Setting `STRIP_OUTER_ARRAY = TRUE` in the file format instructs the parser to strip the outer brackets and treat each array element as a separate row. This is a very common requirement for JSON REST API exports that wrap records in a top-level array.
</details>

---

### Question 62
**Domain:** Domain 3 — Storage & Data Protection

A developer issues `CREATE TABLE t2 CLONE t1 AT (TIMESTAMP => '2024-01-01 00:00:00'::TIMESTAMP_LTZ)`. What is produced?

- [ ] A. A full copy of t1 as it existed at the specified timestamp, consuming full storage immediately.
- [ ] B. A zero-copy clone of t1 reflecting its state at the specified timestamp, sharing historical micro-partitions.
- [ ] C. A view on t1 that filters all rows modified after the specified timestamp.
- [ ] D. The command fails; CLONE does not support AT (TIMESTAMP) clauses.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. A zero-copy clone of t1 reflecting its state at the specified timestamp, sharing historical micro-partitions.

**Explanation:**
Snowflake's `CLONE` command supports `AT (TIMESTAMP => ...)` and `AT (OFFSET => ...)` to clone a **historical state** of a table using Time Travel. The result is a **zero-copy clone** that references the micro-partitions valid at that historical point. No data is physically copied. This is useful for creating point-in-time snapshots for testing, reporting, or data recovery without duplicating storage. The timestamp must be within the table's `DATA_RETENTION_TIME_IN_DAYS` window.
</details>

---

### Question 63
**Domain:** Domain 5 — Account & Security

An admin uses `GRANT IMPORTED PRIVILEGES ON DATABASE snowflake TO ROLE analyst`. What does this grant allow?

- [ ] A. The analyst role can import external data into the Snowflake shared database.
- [ ] B. The analyst role can query `SNOWFLAKE.ACCOUNT_USAGE` views and `SNOWFLAKE.INFORMATION_SCHEMA` functions in the shared Snowflake database.
- [ ] C. The analyst role can share the Snowflake database with other accounts.
- [ ] D. The analyst role gains SYSADMIN-level access to the account metadata.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The analyst role can query `SNOWFLAKE.ACCOUNT_USAGE` views and `SNOWFLAKE.INFORMATION_SCHEMA` functions in the shared Snowflake database.

**Explanation:**
The `SNOWFLAKE` database is a system-provided shared database containing `ACCOUNT_USAGE` schema (query history, access history, billing data). By default, only ACCOUNTADMIN can access it. `GRANT IMPORTED PRIVILEGES ON DATABASE SNOWFLAKE TO ROLE <role>` delegates this access to any role, allowing non-admin roles (like FinOps, Data Governance teams) to query ACCOUNT_USAGE views for monitoring and analysis without granting full ACCOUNTADMIN.
</details>

---

### Question 64
**Domain:** Domain 6 — Performance Optimization

A query uses a correlated subquery referencing the outer table in each row evaluation. The query runs slowly on a 100M-row table. What is the recommended rewrite strategy?

- [ ] A. Add an index to the correlated column.
- [ ] B. Rewrite the correlated subquery as a JOIN or a window function (e.g., ROW_NUMBER, RANK, or an aggregate join) to eliminate per-row execution.
- [ ] C. Increase the warehouse size; correlated subqueries are CPU-bound.
- [ ] D. Use a materialized view on the subquery table to cache the inner results.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Rewrite the correlated subquery as a JOIN or a window function (e.g., ROW_NUMBER, RANK, or an aggregate join) to eliminate per-row execution.

**Explanation:**
**Correlated subqueries** are logically evaluated once per outer row, which can result in O(n) inner query executions. On 100M rows this is prohibitively expensive. The standard fix is to rewrite as a **JOIN** (often with GROUP BY) or a **window function** that computes the correlated result across all rows in a single pass. For example, a "SELECT max salary in the same department" correlated subquery rewrites cleanly as a window `MAX() OVER (PARTITION BY dept)`. Snowflake's optimizer sometimes decorrelates these automatically, but complex cases require manual rewriting.
</details>

---

### Question 65
**Domain:** Domain 1 — Architecture

A Snowpark Container Services (SPCS) **service** is running. The service definition includes a `volumes` block pointing to a Snowflake stage. What does this enable?

- [ ] A. The container can read and write files to the Snowflake internal stage as if it were a local filesystem mount.
- [ ] B. The container receives a read-only snapshot of the stage files at startup time.
- [ ] C. The stage is used as the container's Docker image registry.
- [ ] D. Volumes in SPCS definitions only support ephemeral in-memory storage, not Snowflake stages.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. The container can read and write files to the Snowflake internal stage as if it were a local filesystem mount.

**Explanation:**
In **Snowpark Container Services**, the `volumes` block in a service or job specification can mount a **Snowflake internal stage** into the container at a specified path. The container can then read from and write to files in that stage using standard filesystem operations (no Snowflake SDK required). This is the primary mechanism for containers to persist output files (model artifacts, reports) or read input data files. The stage acts as durable shared storage between Snowflake and the container runtime.
</details>

---

### Question 66
**Domain:** Domain 2 — Virtual Warehouses

A user has `STATEMENT_QUEUED_TIMEOUT_IN_SECONDS = 300` set on their warehouse. A query sits in the queue for 400 seconds without starting execution. What happens?

- [ ] A. The query continues queuing indefinitely until the warehouse has capacity.
- [ ] B. The query is automatically cancelled after 300 seconds in the queue, before execution starts.
- [ ] C. The query is promoted to a higher priority queue after 300 seconds.
- [ ] D. The warehouse automatically scales up after 300 seconds of queue time.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The query is automatically cancelled after 300 seconds in the queue, before execution starts.

**Explanation:**
`STATEMENT_QUEUED_TIMEOUT_IN_SECONDS` sets a maximum time a query can wait in the warehouse queue before being automatically cancelled. This is distinct from `STATEMENT_TIMEOUT_IN_SECONDS` (which applies during execution). After 300 seconds in queue without starting, the query receives a timeout error. This prevents users from unknowingly waiting indefinitely during warehouse saturation events. It can be set at account, warehouse, or session level with the most restrictive value applying.
</details>

---

### Question 67
**Domain:** Domain 3 — Storage & Data Protection

A table uses `CLUSTER BY (TRUNC(event_date, 'MONTH'))`. Why would this be preferred over `CLUSTER BY (event_date)`?

- [ ] A. Truncating to month reduces the number of distinct clustering key values, improving micro-partition organization for monthly query patterns.
- [ ] B. Date truncation is required because raw DATE types cannot be used as clustering keys.
- [ ] C. TRUNC reduces storage costs by compressing the clustering key metadata.
- [ ] D. Monthly truncation prevents automatic reclustering from triggering too frequently.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Truncating to month reduces the number of distinct clustering key values, improving micro-partition organization for monthly query patterns.

**Explanation:**
Using `CLUSTER BY (TRUNC(event_date, 'MONTH'))` reduces the **cardinality** of the clustering key from potentially thousands of distinct dates to tens of distinct months. For workloads that filter by month (e.g., `WHERE event_date BETWEEN '2024-01-01' AND '2024-01-31'`), monthly clustering means entire months can be co-located in contiguous micro-partitions, improving pruning. High-cardinality clustering keys (like full timestamps or UUIDs) often produce worse pruning because the distribution is too fine-grained.
</details>

---

### Question 68
**Domain:** Domain 4 — Data Movement

A Snowflake `PIPE` is configured with `COMMENT = 'prod_ingest'`. After creation, an admin accidentally drops and recreates the stage the pipe references. What must the admin do to restore the pipe's function?

- [ ] A. Nothing — Snowpipe automatically detects the new stage.
- [ ] B. The pipe must be dropped and recreated to reference the new stage definition; existing metadata remains in LOAD_HISTORY for deduplication.
- [ ] C. Run `ALTER PIPE ... SET STAGE = <new_stage_name>` to update the reference.
- [ ] D. Run `ALTER PIPE ... REFRESH` to re-associate the pipe with the recreated stage.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The pipe must be dropped and recreated to reference the new stage definition; existing metadata remains in LOAD_HISTORY for deduplication.

**Explanation:**
Snowflake Pipes embed the **stage reference at creation time** and cannot be updated via ALTER PIPE (ALTER PIPE only supports a limited set of changes like comment and auto_ingest). If the referenced stage is dropped and recreated, the pipe's internal references may break. The standard remediation is to **drop and recreate the pipe** pointing to the new stage. LOAD_HISTORY is queryable via `COPY_HISTORY()` or `INFORMATION_SCHEMA` and provides deduplication — files already loaded will not be reloaded even after pipe recreation.
</details>

---

### Question 69
**Domain:** Domain 5 — Account & Security

A Snowflake account has **Federated Authentication (SSO)** configured with SAML2. A user authenticates via SSO. Later, the admin wants to force re-authentication without waiting for the session timeout. What is the available option?

- [ ] A. Delete the user's SAML assertion from the IdP.
- [ ] B. Run `ALTER USER <name> ABORT SESSION` to kill all active sessions for the user.
- [ ] C. Reduce `CLIENT_SESSION_KEEP_ALIVE_HEARTBEAT_FREQUENCY` to 0.
- [ ] D. Use `SELECT SYSTEM$CANCEL_ALL_QUERIES('<session_id>')` for each active session.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Run `ALTER USER <name> ABORT SESSION` to kill all active sessions for the user.

**Explanation:**
`ALTER USER <name> ABORT SESSION` terminates all active sessions for the specified user. For more granular control, admins can use `SELECT SYSTEM$CANCEL_ALL_QUERIES(session_id)` to cancel running queries within a specific session, or `ALTER SESSION` commands. To forcibly terminate a specific session, `SYSTEM$ABORT_SESSION(session_id)` can also be used. These are the operational tools for emergency session revocation without waiting for natural timeout. (Note: exact DDL syntax may vary; check current docs.)
</details>

---

### Question 70
**Domain:** Domain 1 — Architecture

A developer writes a Snowpark Python UDF that is defined as `IMMUTABLE`. What constraint does this impose, and what optimization does it enable?

- [ ] A. IMMUTABLE prevents the UDF from writing to Snowflake tables; it enables compile-time optimization.
- [ ] B. IMMUTABLE means the function always returns the same output for the same input; Snowflake can cache and reuse results for duplicate input values, reducing compute.
- [ ] C. IMMUTABLE locks the UDF definition so it cannot be altered; it is equivalent to CREATE OR REPLACE with version locking.
- [ ] D. IMMUTABLE UDFs execute in the Cloud Services layer without warehouse compute.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. IMMUTABLE means the function always returns the same output for the same input; Snowflake can cache and reuse results for duplicate input values, reducing compute.

**Explanation:**
In Snowflake UDF definitions, `IMMUTABLE` (also called **deterministic** in some contexts) declares that the function's output is entirely determined by its inputs — no side effects, no randomness, no external reads. This allows the query optimizer to **cache and reuse function results** for duplicate input values within a query, avoiding redundant UDF invocations. For high-cardinality columns with many duplicates (e.g., status codes), this can significantly reduce compute. `VOLATILE` (the default) disables this optimization because outputs may differ between calls.
</details>

---

### Question 71
**Domain:** Domain 4 — Data Movement

A developer uses `COPY INTO` with `PURGE = TRUE`. A file fails to load. Is the file purged?

- [ ] A. Yes — PURGE = TRUE removes all files from the stage after the COPY command, regardless of load outcome.
- [ ] B. No — Snowflake only purges files that were **successfully** loaded; files that errored remain in the stage.
- [ ] C. Yes — but Snowflake creates a backup copy in the error bucket first.
- [ ] D. PURGE = TRUE is not a valid COPY INTO option; purging must be done via `REMOVE @stage/file`.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. No — Snowflake only purges files that were **successfully** loaded; files that errored remain in the stage.

**Explanation:**
`PURGE = TRUE` instructs Snowflake to automatically remove stage files after **successful** loading. Files that fail to load (due to data errors, schema mismatch, etc.) are **not purged** — they remain in the stage for investigation and potential reprocessing. This behavior ensures that failed files are not lost; they can be corrected and reloaded. Successfully loaded files are removed from the stage, reducing storage costs and preventing duplicate loads on future COPY runs.
</details>

---

### Question 72
**Domain:** Domain 6 — Performance Optimization

A query plan shows `TableScan` → `Filter` → `Aggregate` → `Sort` → `Limit 10`. Where does Snowflake's optimizer apply **filter pushdown**, and why?

- [ ] A. The filter is applied after sorting to reduce the final output size.
- [ ] B. The filter is pushed down to the TableScan operation, evaluating predicates during micro-partition scanning to minimize data read into the pipeline.
- [ ] C. Filters are always applied at the Cloud Services layer before warehouse compute begins.
- [ ] D. Filter pushdown only applies to JOIN operations, not single-table scans.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The filter is pushed down to the TableScan operation, evaluating predicates during micro-partition scanning to minimize data read into the pipeline.

**Explanation:**
**Predicate pushdown** is a fundamental Snowflake optimization where filter conditions are applied as early as possible — during the `TableScan` phase. This has two effects: (1) **micro-partition pruning** — entire micro-partitions that cannot satisfy the filter are skipped entirely, and (2) **within-partition filtering** — rows not matching the predicate are discarded before being passed to subsequent operators. Applying filters after aggregation or sort would force processing far more data than necessary. This is why indexing is less critical in Snowflake — micro-partition pruning + filter pushdown achieves similar benefits.
</details>

---

### Question 73
**Domain:** Domain 3 — Storage & Data Protection

A `MASKING POLICY` returns `SHA2(val, 256)` for unauthorized roles. An analyst with the unauthorized role queries the masked column and receives a long hex string. They note the same input value always produces the same hash. Is this a data security concern?

- [ ] A. No — SHA256 is a one-way hash; it is computationally infeasible to reverse.
- [ ] B. Yes — deterministic masking allows frequency analysis and dictionary attacks. If the domain of values is small (e.g., US SSNs), all values can be reversed by precomputing hashes for the entire domain.
- [ ] C. No — Snowflake adds a random salt to all SHA2 masking operations automatically.
- [ ] D. Yes — but only if the analyst also has access to an external SHA256 rainbow table.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Yes — deterministic masking allows frequency analysis and dictionary attacks. If the domain of values is small (e.g., US SSNs), all values can be reversed by precomputing hashes for the entire domain.

**Explanation:**
Unsalted deterministic hashing (SHA256 without a secret salt) in masking policies is a common **security antipattern**. For low-cardinality or predictable domains (SSNs = 10^9 values, phone numbers, zip codes), an attacker can precompute hashes for all possible values and create a rainbow table. The correct approach is to use `HMAC(key, val)` with a secret key stored in a Snowflake Secret object, or to use Snowflake's Format-Preserving Encryption. The determinism also allows join attacks — the analyst can count value frequencies or join masked data with other masked datasets.
</details>

---

### Question 74
**Domain:** Domain 1 — Architecture

A Snowflake `FUNCTION` (scalar UDF) is created with `CALLED ON NULL INPUT` vs. `RETURNS NULL ON NULL INPUT`. What is the difference?

- [ ] A. `RETURNS NULL ON NULL INPUT` forces the function to always return NULL, regardless of logic.
- [ ] B. `RETURNS NULL ON NULL INPUT` short-circuits execution and returns NULL without calling the function body if any argument is NULL; `CALLED ON NULL INPUT` calls the function even when arguments are NULL.
- [ ] C. `CALLED ON NULL INPUT` allows the function to call other UDFs that accept NULL; `RETURNS NULL ON NULL INPUT` prevents function chaining.
- [ ] D. These are aliases with no behavioral difference.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. `RETURNS NULL ON NULL INPUT` short-circuits execution and returns NULL without calling the function body if any argument is NULL; `CALLED ON NULL INPUT` calls the function even when arguments are NULL.

**Explanation:**
The **null-handling property** controls whether the function body is invoked for NULL inputs. `RETURNS NULL ON NULL INPUT` (also called `STRICT` in some databases) is a performance optimization — Snowflake skips the function call entirely and returns NULL immediately for any NULL argument, reducing compute. `CALLED ON NULL INPUT` (the default) invokes the function body regardless, allowing functions that handle NULLs specially (e.g., treating NULL as a default value). Choosing the right property avoids unnecessary UDF invocations.
</details>

---

### Question 75
**Domain:** Domain 2 — Virtual Warehouses

A Snowflake account sets `ENABLE_QUERY_ACCELERATION = TRUE` on a warehouse. Which type of query benefits MOST from Query Acceleration Service?

- [ ] A. Short, high-concurrency OLTP queries with selective point lookups.
- [ ] B. Outlier queries — large analytical queries with unpredictable runtimes that significantly exceed the warehouse's normal query patterns.
- [ ] C. All queries benefit equally; Query Acceleration provides a flat 2× speedup.
- [ ] D. Queries involving JOIN operations on tables without clustering keys.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Outlier queries — large analytical queries with unpredictable runtimes that significantly exceed the warehouse's normal query patterns.

**Explanation:**
**Query Acceleration Service (QAS)** is designed for **outlier queries** — analytical queries that require significantly more compute than typical for a given warehouse size. QAS dynamically provisions additional serverless compute resources to accelerate eligible portions of a query (typically large scans and aggregations) without upsizing the warehouse permanently. It does not provide a flat speedup for all queries; the acceleration is conditional on the query's characteristics and the configured `SCALE_FACTOR`. Short OLTP queries don't benefit because their bottleneck is latency, not throughput.
</details>

---

### Question 76
**Domain:** Domain 5 — Account & Security

A column has both a **masking policy** AND a **row access policy** applied. Which is enforced first, and why does the order matter?

- [ ] A. The masking policy is evaluated first; filtered-out rows are never seen by the masking logic.
- [ ] B. The row access policy is enforced first (at the row level), then masking is applied to the visible columns of the allowed rows. Order matters because masked values in filtered rows are never computed.
- [ ] C. Both are evaluated simultaneously in a single pass for performance.
- [ ] D. The order depends on which policy was applied most recently.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The row access policy is enforced first (at the row level), then masking is applied to the visible columns of the allowed rows. Order matters because masked values in filtered rows are never computed.

**Explanation:**
Snowflake enforces **row access policies before column masking policies**. Rows that the RAP filters out are never passed to the masking evaluation — this is more efficient and consistent. If masking were applied first, the engine would compute masked values for rows that will ultimately be filtered out. The logical order mirrors SQL's natural execution: row filtering (WHERE equivalent) before column projection (SELECT equivalent). This also means the masking policy's `CURRENT_ROLE()` context sees only the rows permitted by the RAP.
</details>

---

### Question 77
**Domain:** Domain 4 — Data Movement

A Snowflake `EXTERNAL FUNCTION` is called inside a SQL query. The external function calls an AWS Lambda via API Gateway. What happens if the Lambda returns HTTP 429 (Too Many Requests)?

- [ ] A. Snowflake immediately fails the query with an error.
- [ ] B. Snowflake automatically retries with exponential backoff; if retries are exhausted, the query fails.
- [ ] C. The 429 response is returned as a NULL value for that row's function call.
- [ ] D. Snowflake queues the function calls and retries after 60 seconds.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowflake automatically retries with exponential backoff; if retries are exhausted, the query fails.

**Explanation:**
Snowflake **external functions** implement automatic **retry with exponential backoff** for HTTP error codes that indicate transient conditions, including 429 (Too Many Requests) and 5xx server errors. Snowflake batches rows and retries failed batches with increasing delays. If all retries are exhausted and the endpoint still returns errors, the query ultimately fails. This retry behavior is built into the external function framework — the Lambda does not need to implement retry logic. Throttling on the API Gateway/Lambda side should still be planned for via appropriate concurrency limits.
</details>

---

### Question 78
**Domain:** Domain 1 — Architecture

A developer creates a **Snowflake Alert** with `SCHEDULE = 'USING CRON 0 9 * * MON-FRI UTC'`. The alert condition query returns rows (trigger condition). What happens?

- [ ] A. The alert sends an email to all account admins automatically.
- [ ] B. The alert executes the configured action (e.g., calls a system function or triggers a notification integration) only when the condition query returns one or more rows.
- [ ] C. The alert logs the condition to the event table and waits for a human to acknowledge.
- [ ] D. The alert suspends itself after the first trigger to prevent repeated notifications.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The alert executes the configured action (e.g., calls a system function or triggers a notification integration) only when the condition query returns one or more rows.

**Explanation:**
**Snowflake Alerts** evaluate a condition query on a defined schedule. If the condition query returns **any rows**, the alert fires and executes the defined action — typically calling a Notification Integration (email, Slack, PagerDuty via SNS/email) or a stored procedure. If the condition returns no rows, no action is taken. Alerts do not auto-suspend; they continue evaluating on schedule unless manually suspended. The cron pattern here runs on weekday mornings at 9 AM UTC.
</details>

---

### Question 79
**Domain:** Domain 3 — Storage & Data Protection

A Snowflake table has **Automatic Data Classification** enabled. It detects a column contains email addresses. What action does classification take automatically?

- [ ] A. It automatically applies a masking policy to the email column.
- [ ] B. It applies system-defined **tags** (e.g., SNOWFLAKE.CORE.EMAIL) to the column; masking is NOT applied automatically unless tag-based masking policies are configured.
- [ ] C. It moves the email column data to a separate quarantine table for review.
- [ ] D. It renames the column with a `PII_` prefix to signal sensitive content.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. It applies system-defined **tags** (e.g., SNOWFLAKE.CORE.EMAIL) to the column; masking is NOT applied automatically unless tag-based masking policies are configured.

**Explanation:**
**Automatic Data Classification** scans column data and applies **Snowflake system-defined semantic tags** from the `SNOWFLAKE.CORE` schema (e.g., `SNOWFLAKE.CORE.EMAIL`, `SNOWFLAKE.CORE.PHONE_NUMBER`, `SNOWFLAKE.CORE.CREDIT_CARD`). The classification engine tags columns — it does **not** apply masking policies on its own. Masking is enforced separately via tag-based masking policy associations. This two-step approach gives organizations flexibility: classify first, then decide which masking policies to attach to which tags.
</details>

---

### Question 80
**Domain:** Domain 6 — Performance Optimization

A query runs in 30 seconds. The query profile shows **95% of time in "TableScan"** and **high "Bytes scanned per query"**. The WHERE clause filters on a column that is the 3rd element of the clustering key. What is the likely issue?

- [ ] A. The clustering key has too few columns; adding more will improve pruning.
- [ ] B. The first two clustering key columns are not filtered in the WHERE clause, so Snowflake cannot prune on the 3rd column because the data is sorted by the leading key columns first.
- [ ] C. The warehouse is undersized; upgrading will reduce scan time.
- [ ] D. The table should be replaced with a materialized view to avoid the scan.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The first two clustering key columns are not filtered in the WHERE clause, so Snowflake cannot prune on the 3rd column because the data is sorted by the leading key columns first.

**Explanation:**
Snowflake's micro-partition pruning is most effective when **leading columns of the clustering key** are filtered. If the data is clustered by `(col1, col2, col3)` but the query only filters on `col3`, Snowflake cannot skip partitions efficiently because the physical sort order is dominated by col1 and col2 — col3 values are interleaved across many partitions. This is analogous to a composite index in relational databases: the leftmost prefix rule applies. The fix is to either reorder the clustering key to put `col3` first, or create a separate clustering key if queries on `col3` alone are common.
</details>

---

### Question 81
**Domain:** Domain 1 — Architecture

What is the functional difference between a **Snowflake Task** using `SCHEDULE = '5 MINUTE'` and a Task using `AFTER parent_task`?

- [ ] A. Scheduled tasks run on a fixed clock interval; AFTER tasks run only when explicitly triggered by the parent task completing successfully, enabling DAG-based dependencies.
- [ ] B. Scheduled tasks use serverless compute; AFTER tasks require a named warehouse.
- [ ] C. AFTER tasks run regardless of parent task success or failure.
- [ ] D. Scheduled tasks and AFTER tasks are mutually exclusive options for the same functionality.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Scheduled tasks run on a fixed clock interval; AFTER tasks run only when explicitly triggered by the parent task completing successfully, enabling DAG-based dependencies.

**Explanation:**
In Snowflake Tasks: a **scheduled task** (using `SCHEDULE = '5 MINUTE'` or CRON) triggers independently on a time interval — it is always a root task. A **child task** (using `AFTER <parent_task>`) has no schedule of its own; it executes **when its predecessor completes successfully**. This forms a **DAG (Directed Acyclic Graph)** of tasks. Child tasks only trigger on success (not failure) of the parent by default. Combining scheduled root tasks with child tasks enables multi-step pipeline orchestration within Snowflake.
</details>

---

### Question 82
**Domain:** Domain 5 — Account & Security

An admin creates a **Private Listing** on Snowflake Marketplace. Which accounts can see this listing?

- [ ] A. All Snowflake accounts globally can see and request access to private listings.
- [ ] B. Only the specific consumer accounts explicitly invited/approved by the provider can see the listing; it is not publicly discoverable.
- [ ] C. Only accounts within the same Snowflake Organization as the provider.
- [ ] D. Only accounts on the same cloud provider (AWS, Azure, GCP) as the provider.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Only the specific consumer accounts explicitly invited/approved by the provider can see the listing; it is not publicly discoverable.

**Explanation:**
**Private Listings** on Snowflake Marketplace are not publicly visible. The provider explicitly specifies which consumer accounts (by account locator or organization account name) can see and access the listing. This allows controlled, invitation-only data sharing through the Marketplace infrastructure while maintaining the self-service installation experience for approved consumers. Public listings are discoverable by all Snowflake accounts; private listings are not indexed in the public Marketplace catalog.
</details>

---

### Question 83
**Domain:** Domain 4 — Data Movement

A Snowflake **Data Sharing** consumer queries a shared table. The consumer does not have a warehouse active. What happens?

- [ ] A. The query runs on the provider's warehouse automatically.
- [ ] B. The query fails because a warehouse is required; the consumer must activate or select a warehouse in their own account.
- [ ] C. The query uses the Cloud Services layer exclusively because shared data queries are metadata-only.
- [ ] D. The query is routed to Snowflake's shared compute pool reserved for data sharing consumers.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The query fails because a warehouse is required; the consumer must activate or select a warehouse in their own account.

**Explanation:**
In Snowflake Data Sharing, the **consumer's own warehouse** executes queries against shared data. The provider's warehouse is never used for consumer queries. If no warehouse is active in the consumer's session, the query will fail (or auto-resume if AUTO_RESUME is enabled on the consumer's warehouse). This is a key architectural point: data is shared (zero-copy) but **compute is separate**. The consumer pays for their own warehouse compute; the provider pays nothing for consumer query execution.
</details>

---

### Question 84
**Domain:** Domain 3 — Storage & Data Protection

Snowflake encrypts data at rest using **hierarchical key management**. What is the correct key hierarchy from top to bottom?

- [ ] A. Master Key → Account Key → Table Key → File Key
- [ ] B. Cloud Provider Root Key → Snowflake Root Key → Account Key → Table Key → File Key
- [ ] C. Customer KMS Key → Account Key → File Key
- [ ] D. Account Key → Table Key → Column Key → Row Key

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Cloud Provider Root Key → Snowflake Root Key → Account Key → Table Key → File Key

**Explanation:**
Snowflake uses a **multi-tiered encryption key hierarchy**: (1) the **Cloud Provider Root Key** (managed by AWS KMS, Azure Key Vault, or GCP KMS) encrypts Snowflake's root keys; (2) **Snowflake Root Key** encrypts account-level keys; (3) **Account Key** encrypts table keys; (4) **Table Key** encrypts individual **File Keys**; (5) **File Keys** encrypt the actual micro-partition data files. Each key tier is rotated independently. For Tri-Secret Secure, the customer's KMS key wraps the Snowflake root key, requiring both keys to decrypt anything.
</details>

---

### Question 85
**Domain:** Domain 2 — Virtual Warehouses

A developer wants to run a large Python ML training job in Snowpark. The job processes 50 GB of feature data entirely in-memory. What warehouse type and size combination is MOST appropriate?

- [ ] A. Standard X-LARGE — more nodes provide more parallelism for ML training.
- [ ] B. Snowpark-Optimized MEDIUM — provides high memory-per-node for in-memory processing; ML training often benefits more from memory than parallelism at this scale.
- [ ] C. Standard SMALL with Query Acceleration enabled — QAS handles memory-intensive ML workloads.
- [ ] D. Cloud Services layer only — Snowpark ML runs without a warehouse.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowpark-Optimized MEDIUM — provides high memory-per-node for in-memory processing; ML training often benefits more from memory than parallelism at this scale.

**Explanation:**
For **in-memory ML training** workloads (e.g., scikit-learn, XGBoost on feature DataFrames), the bottleneck is **memory capacity**, not parallelism. A **Snowpark-Optimized warehouse** provides ~16× more RAM per node than a Standard warehouse. At 50 GB, a MEDIUM Snowpark-Optimized warehouse (which provides very high memory per node) is likely sufficient to hold the full dataset in memory without spilling. A Standard X-LARGE has more nodes but less memory per node — it's better for distributed SQL, not single-node ML training.
</details>

---

### Question 86
**Domain:** Domain 1 — Architecture

A **Dynamic Table** has `TARGET_LAG = DOWNSTREAM`. What does this mean?

- [ ] A. The dynamic table refreshes based on downstream consumer query patterns.
- [ ] B. The dynamic table inherits its lag requirement from whatever downstream dynamic table or object depends on it, allowing the pipeline to self-optimize lag propagation.
- [ ] C. The dynamic table refreshes immediately when any downstream object queries it.
- [ ] D. DOWNSTREAM is not a valid TARGET_LAG value; valid values are time intervals and 'ON_DEMAND'.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The dynamic table inherits its lag requirement from whatever downstream dynamic table or object depends on it, allowing the pipeline to self-optimize lag propagation.

**Explanation:**
`TARGET_LAG = DOWNSTREAM` is a special value in Snowflake Dynamic Tables that means the table's lag is determined by its downstream dependents. If a downstream dynamic table has a 1-hour lag, the upstream table with DOWNSTREAM lag will be refreshed at the same frequency — Snowflake propagates the lag requirement backward through the pipeline. This is used for intermediate transformation tables that have no independent lag requirement and should only refresh as fast as needed by downstream consumers, optimizing cost.
</details>

---

### Question 87
**Domain:** Domain 4 — Data Movement

A developer uses Snowflake's `INFER_SCHEMA()` on a set of Parquet files in a stage. What does this function return?

- [ ] A. A CREATE TABLE DDL statement that can be directly executed.
- [ ] B. A result set describing the detected column names, data types, and nullable status inferred from the Parquet file schemas.
- [ ] C. A JSON object with file-level statistics (row count, file size, compression ratio).
- [ ] D. A Snowflake FILE FORMAT definition that matches the Parquet file structure.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. A result set describing the detected column names, data types, and nullable status inferred from the Parquet file schemas.

**Explanation:**
`INFER_SCHEMA()` is a table function that reads the schema embedded in columnar file formats (Parquet, Avro, ORC) and returns a **result set** with columns: `COLUMN_NAME`, `TYPE`, `NULLABLE`, `EXPRESSION`, `FILENAMES`. This result can be used with `CREATE TABLE ... USING TEMPLATE (SELECT ARRAY_AGG(OBJECT_CONSTRUCT(*)) FROM TABLE(INFER_SCHEMA(...)))` to automatically create a table matching the file schema. It does not return DDL directly or file statistics — that's `SYSTEM$GET_STAGE_OBJECT_INFO`.
</details>

---

### Question 88
**Domain:** Domain 5 — Account & Security

An account admin enables **Access History** logging. What data is captured in `SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY`?

- [ ] A. Every SQL statement text, execution time, and result set size.
- [ ] B. Column-level read and write access records: which user/role accessed which columns of which tables, and in which queries — enabling compliance-level data lineage tracking.
- [ ] C. Network-level access logs showing IP addresses and connection times.
- [ ] D. Object privilege grants and revocations with timestamps.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Column-level read and write access records: which user/role accessed which columns of which tables, and in which queries — enabling compliance-level data lineage tracking.

**Explanation:**
`ACCESS_HISTORY` in Snowflake provides **column-level data access lineage**: for each query, it records which base table columns were directly or indirectly accessed (read), which columns were written, the user, role, query ID, and object names. This granularity enables compliance use cases like GDPR/CCPA data subject access requests (which columns, and therefore which data, did a user touch?) and security audits (who accessed the SSN column last week?). It is more detailed than `QUERY_HISTORY` (which has SQL text) because it tracks resolved column-level access, even through views and UDFs.
</details>

---

### Question 89
**Domain:** Domain 1 — Architecture

A **Snowflake Data Clean Room** provider wants to restrict the minimum aggregation size so that individual-level data cannot be inferred from query results. Which mechanism enforces this?

- [ ] A. A row access policy that returns no rows for groups smaller than 5.
- [ ] B. A clean room privacy policy (minimum aggregation threshold) defined in the clean room's analysis templates, enforced by the clean room framework before results are returned to consumers.
- [ ] C. A masking policy that hashes small-group results.
- [ ] D. A Snowflake Alert that triggers when query results return fewer than 5 rows.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. A clean room privacy policy (minimum aggregation threshold) defined in the clean room's analysis templates, enforced by the clean room framework before results are returned to consumers.

**Explanation:**
In Snowflake Data Clean Rooms, **privacy policies** are embedded in the pre-defined analysis templates (Jinja SQL templates). A **minimum aggregation threshold** (k-anonymity) prevents results from being returned when the matching group size is below a defined minimum (e.g., suppress results for audiences < 1,000 people). This is enforced by the clean room framework at the template level — consumers cannot modify the templates, so they cannot bypass the threshold. This is stronger than a row access policy because it applies to aggregated outputs, not just individual row access.
</details>

---

### Question 90
**Domain:** Domain 3 — Storage & Data Protection

A table with 1 billion rows and `CLUSTER BY (order_date)` is queried with `WHERE order_date = '2024-03-15'`. The query returns in 2 seconds. The same table is queried with `WHERE customer_name LIKE '%Smith%'`. Why is the second query likely much slower?

- [ ] A. LIKE queries are disabled on clustered tables.
- [ ] B. The LIKE with a leading wildcard cannot use clustering key pruning (order_date); Snowflake must scan all partitions. The leading `%` also prevents any substring index from being used unless Search Optimization is configured.
- [ ] C. VARIANT columns scan slower than DATE columns, regardless of clustering.
- [ ] D. The second query requires sorting, which invalidates the cluster key.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The LIKE with a leading wildcard cannot use clustering key pruning (order_date); Snowflake must scan all partitions. The leading `%` also prevents any substring index from being used unless Search Optimization is configured.

**Explanation:**
Two problems compound: (1) `customer_name` is not the clustering key, so **no micro-partition pruning** occurs — all 1 billion rows' partitions are scanned; (2) a **leading wildcard** (`'%Smith%'`) cannot use any prefix-based optimization and requires full string matching. Without Search Optimization configured for substring matching on `customer_name`, Snowflake performs a full table scan with per-row LIKE evaluation. The fix is to add Search Optimization (`SEARCH OPTIMIZATION ON LIKE(customer_name)`) to build a persistent substring index.
</details>

---

### Question 91
**Domain:** Domain 2 — Virtual Warehouses

A Snowflake warehouse has `AUTO_SUSPEND = 60` seconds. A long-running query finishes after 55 seconds of idle time, but then immediately another query is submitted. Does the warehouse suspend?

- [ ] A. Yes — 55 seconds of idle exceeded half the AUTO_SUSPEND threshold, triggering an early suspend.
- [ ] B. No — the new query resets the idle timer; the warehouse only suspends after 60 consecutive idle seconds with no new activity.
- [ ] C. Yes — the warehouse suspends and then immediately resumes for the new query.
- [ ] D. No — AUTO_SUSPEND only applies after query queue is empty for 60 minutes.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. No — the new query resets the idle timer; the warehouse only suspends after 60 consecutive idle seconds with no new activity.

**Explanation:**
`AUTO_SUSPEND = 60` means the warehouse suspends after **60 continuous seconds with no activity**. Any new query submission (or session keepalive) **resets the idle timer**. In this scenario, the new query arrives after 55 seconds of idle — within the 60-second window — so the warehouse does not suspend. The timer resets and starts counting from 0 again after the new query completes. This prevents unnecessary suspend/resume cycles during intermittent query bursts.
</details>

---

### Question 92
**Domain:** Domain 4 — Data Movement

A Snowflake `COPY INTO` statement uses `MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE`. What does this do?

- [ ] A. It allows column names in the file to match table columns without case sensitivity, mapping file headers to table columns regardless of capitalization differences.
- [ ] B. It converts all column names to uppercase before loading.
- [ ] C. It only loads columns where the file header and table column name are an exact case-sensitive match.
- [ ] D. MATCH_BY_COLUMN_NAME is only valid for JSON files, not CSV.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. It allows column names in the file to match table columns without case sensitivity, mapping file headers to table columns regardless of capitalization differences.

**Explanation:**
`MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE` (available for semi-structured files like Parquet, Avro, ORC, and CSV with headers) instructs Snowflake to match **column names by name (not position)**, ignoring case differences. A file column `OrderDate` maps to a table column `order_date`. Columns in the file but not in the table are ignored; columns in the table but not in the file receive NULL. This is valuable when source schema and target schema have different naming conventions and avoids brittle position-based mapping.
</details>

---

### Question 93
**Domain:** Domain 5 — Account & Security

A Snowflake account administrator wants all users to require **Multi-Factor Authentication (MFA)** when connecting. What is the supported enforcement mechanism?

- [ ] A. Set `REQUIRE_MFA = TRUE` at the account level in the account parameters.
- [ ] B. Apply an MFA enforcement policy using `CREATE AUTHENTICATION POLICY` with `MFA_ENROLLMENT = REQUIRED` and attach it to the account or specific users/roles.
- [ ] C. Enable MFA by configuring the SSO provider to require MFA before issuing SAML tokens.
- [ ] D. Grant the `MFA_REQUIRED` role to all users to enforce MFA.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Apply an MFA enforcement policy using `CREATE AUTHENTICATION POLICY` with `MFA_ENROLLMENT = REQUIRED` and attach it to the account or specific users/roles.

**Explanation:**
Snowflake's **Authentication Policies** (introduced as a governance feature) allow administrators to create policies that enforce authentication requirements including `MFA_ENROLLMENT = REQUIRED`. When attached to an account or user, users who have not enrolled in MFA cannot log in. Authentication policies can also restrict authentication methods (password only, federated only, key-pair only, etc.). This is the native Snowflake-managed MFA enforcement mechanism, distinct from IdP-level MFA which is enforced before the SAML assertion reaches Snowflake.
</details>

---

### Question 94
**Domain:** Domain 1 — Architecture

A Snowflake `SEQUENCE` is used to generate surrogate keys. A transaction inserts 100 rows using the sequence. The transaction is then **rolled back**. What happens to the sequence values consumed?

- [ ] A. The sequence values are returned to the pool and will be reissued on the next call.
- [ ] B. The sequence values are permanently consumed and will not be reused, even though the transaction was rolled back — creating gaps in the sequence.
- [ ] C. The sequence rolls back to its pre-transaction value automatically.
- [ ] D. The sequence is locked until the next explicit COMMIT on the sequence object.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The sequence values are permanently consumed and will not be reused, even though the transaction was rolled back — creating gaps in the sequence.

**Explanation:**
Snowflake **sequences** are designed for high concurrency — they pre-allocate blocks of values. Sequence values consumed during a transaction are **not returned to the pool on rollback**. This is intentional: reclaiming sequence values on rollback would require serializing sequence access, defeating the purpose of non-locking sequence generation. Gaps in sequence values are expected and acceptable behavior. Sequence values are only guaranteed to be **unique and monotonically increasing** (within the configured ORDER or NOORDER setting), not gapless.
</details>

---

### Question 95
**Domain:** Domain 6 — Performance Optimization

A developer notices that identical queries from two different sessions with different active roles take very different times. One session uses the data analyst role; the other uses SYSADMIN. Explain the likely cause.

- [ ] A. SYSADMIN queries bypass the query result cache.
- [ ] B. Row access policies or column masking policies evaluated for the analyst role add computational overhead not present for SYSADMIN queries (which may see unfiltered data).
- [ ] C. SYSADMIN queries are assigned higher warehouse priority, explaining the speed difference.
- [ ] D. The roles use different warehouses with different sizes.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Row access policies or column masking policies evaluated for the analyst role add computational overhead not present for SYSADMIN queries (which may see unfiltered data).

**Explanation:**
Security policies (row access policies, masking policies) add **query planning and execution overhead**. A row access policy that filters to specific rows requires additional predicate evaluation. Column masking that involves complex functions (hashing, lookups) adds computation. If SYSADMIN is listed in the masking policy as an "unmasked" role, it receives raw data with minimal overhead. The analyst role triggers the full policy evaluation path. Always benchmark with the actual production role, not admin roles, to get representative query timings.
</details>

---

### Question 96
**Domain:** Domain 1 — Architecture

A **Snowflake Native App** is published to the Marketplace with `DISTRIBUTION = EXTERNAL`. What is the significance of the EXTERNAL distribution?

- [ ] A. External apps are hosted on cloud infrastructure outside of Snowflake.
- [ ] B. EXTERNAL distribution means the app is listed publicly on the Snowflake Marketplace and can be installed by any Snowflake account, not just the provider's own accounts.
- [ ] C. EXTERNAL apps are distributed as Docker containers rather than SQL/Python code.
- [ ] D. EXTERNAL distribution requires the consumer to have a Business Critical account.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. EXTERNAL distribution means the app is listed publicly on the Snowflake Marketplace and can be installed by any Snowflake account, not just the provider's own accounts.

**Explanation:**
When publishing a Snowflake Native App, `DISTRIBUTION = INTERNAL` limits the app to accounts within the same **Snowflake Organization** (useful for internal tooling). `DISTRIBUTION = EXTERNAL` enables the app to be shared **publicly via the Snowflake Marketplace** with any Snowflake account globally. External distribution triggers an automated security review by Snowflake before the app becomes publicly available. This ensures that third-party apps published publicly meet Snowflake's security standards.
</details>

---

### Question 97
**Domain:** Domain 3 — Storage & Data Protection

A Snowflake account uses **S3-compatible storage** via an External Volume for Iceberg tables. The data engineer accidentally deletes a Parquet file directly from S3 (bypassing Snowflake). What happens when the Iceberg table is queried?

- [ ] A. Snowflake automatically recovers the deleted file from its internal backup.
- [ ] B. The query may succeed partially (if the deleted file's partitions are not needed by the query) or fail with a file-not-found error if the affected partition is queried — Snowflake cannot recover files deleted from external storage.
- [ ] C. The external table automatically excludes the deleted partition and continues normally.
- [ ] D. Snowflake detects the deletion and triggers an automatic REFRESH to rebuild the metadata.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The query may succeed partially (if the deleted file's partitions are not needed by the query) or fail with a file-not-found error if the affected partition is queried — Snowflake cannot recover files deleted from external storage.

**Explanation:**
For **Iceberg tables on external volumes**, the actual data files (Parquet) reside in customer-managed cloud storage. Snowflake only manages the metadata layer — it cannot recover files deleted from the customer's S3/GCS/Azure bucket. If a query accesses a partition whose data file has been deleted, Snowflake will return a file-not-found error. The Iceberg metadata (snapshots, manifests) still references the deleted file. Recovery requires restoring the file from S3 versioning or Glacier. This underscores the importance of enabling S3 Object Lock or Versioning for Iceberg table storage.
</details>

---

### Question 98
**Domain:** Domain 4 — Data Movement

A `COPY INTO` loads 10,000 files from a stage. After 8,000 files load successfully, the session is killed. How does Snowflake handle the partial load?

- [ ] A. All 10,000 files are rolled back; no rows are committed.
- [ ] B. The 8,000 successfully loaded files are committed (COPY INTO auto-commits each file as it loads); the remaining 2,000 files can be loaded in a subsequent COPY INTO without reloading the 8,000.
- [ ] C. The partial load is rolled back to the last checkpoint (every 1,000 files).
- [ ] D. Snowflake marks all 10,000 files as "in progress" and locks the stage until the session resumes.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The 8,000 successfully loaded files are committed (COPY INTO auto-commits each file as it loads); the remaining 2,000 files can be loaded in a subsequent COPY INTO without reloading the 8,000.

**Explanation:**
`COPY INTO` is **not a single transaction** — it commits data incrementally as files are processed. If a session is interrupted mid-load, already-processed files are committed. LOAD_HISTORY tracks which files have been successfully loaded. A subsequent COPY INTO from the same stage will **skip already-loaded files** (using the LOAD_HISTORY deduplication mechanism) and process only the remaining 2,000 files. This idempotent behavior means COPY INTO is safe to retry after interruptions.
</details>

---

### Question 99
**Domain:** Domain 5 — Account & Security

An account has **PREVENT_UNLOAD_TO_INLINE_URL = TRUE** set. What does this restrict?

- [ ] A. Prevents the COPY INTO command from writing to any cloud storage.
- [ ] B. Prevents data from being unloaded to ad-hoc inline URL stages (e.g., `COPY INTO 's3://bucket/path'` without a named external stage), requiring use of named external stages with governed credentials.
- [ ] C. Prevents users from downloading query results via the Snowsight UI.
- [ ] D. Prevents external table data from being copied into internal tables.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Prevents data from being unloaded to ad-hoc inline URL stages (e.g., `COPY INTO 's3://bucket/path'` without a named external stage), requiring use of named external stages with governed credentials.

**Explanation:**
`PREVENT_UNLOAD_TO_INLINE_URL` is a security parameter that forces users to use **named external stages** (with pre-approved, admin-managed credentials and storage locations) for data exports. Without this restriction, any user with COPY privileges could write data to an arbitrary S3/Azure/GCS URL using their own credentials — creating a data exfiltration risk. By requiring named stages, security teams can audit and govern exactly which destinations data can be exported to. This is an important data loss prevention (DLP) control.
</details>

---

### Question 100
**Domain:** Domain 6 — Performance Optimization

A developer's query uses `GROUP BY ALL`. What does this syntax do, and in which Snowflake version was it introduced?

- [ ] A. `GROUP BY ALL` is a shorthand that automatically groups by all columns in the SELECT clause that are not inside aggregate functions, eliminating the need to list columns explicitly.
- [ ] B. `GROUP BY ALL` groups by every column in the FROM clause tables.
- [ ] C. `GROUP BY ALL` creates a cross-product grouping (equivalent to GROUPING SETS of all combinations).
- [ ] D. `GROUP BY ALL` is not supported in Snowflake; it is a BigQuery-only syntax.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. `GROUP BY ALL` is a shorthand that automatically groups by all columns in the SELECT clause that are not inside aggregate functions, eliminating the need to list columns explicitly.

**Explanation:**
`GROUP BY ALL` (supported in Snowflake, DuckDB, and BigQuery) is a convenience syntax that **infers the GROUP BY columns** from the SELECT list — any non-aggregated expression in SELECT is automatically included in the GROUP BY. This reduces boilerplate and maintenance burden: if you add a new dimension column to the SELECT, the GROUP BY automatically includes it. For example, `SELECT region, product, SUM(sales) FROM t GROUP BY ALL` is equivalent to `GROUP BY region, product`. Snowflake added this feature to reduce SQL verbosity for analytical queries.
</details>

---

*End of Question Set — 100 Questions | SnowPro Core COF-C03 | v3 (2025)*

# SnowPro Core (COF-C03) Practice Questions — Batch 301–408

Reconstructed and cleaned from raw OCR. Answers cross-checked against current Snowflake documentation (as of July 2026). Corrections are flagged with **⚠ Updated**. Click "Show Answer" to reveal.

---

### Question 301
What happens when an external or an internal stage is dropped? (Choose two)

- A. When dropping an external stage, the files are not deleted — only the stage object is dropped.
- B. When dropping an external stage, both the stage and the files within it are removed.
- C. When dropping an internal stage, the files are deleted with the stage and are recoverable.
- D. When dropping an internal stage, the files are deleted with the stage and are **not** recoverable.
- E. When dropping an internal stage, only selected files are deleted with the stage and are not recoverable.

<details><summary>Show Answer</summary>
Correct Answer: A, D. External stages only reference external cloud storage — Snowflake never owns or deletes the underlying files. Internal stages physically hold the files in Snowflake-managed storage, so dropping the stage permanently deletes them (no Time Travel/Fail-safe recovery for stage files).
</details>

---

### Question 302
A user has 10 files in a stage containing new customer data. The ingest completes with no errors using `COPY INTO my_table FROM @stage`. The next day the user adds 10 more files so the stage now contains a mix of new customer data and updates to the previous data. The original 10 files were not removed. If the user re-runs the same `COPY INTO` command, what happens?

- A. All data from all files on the stage will be appended to the table.
- B. Only data about new customers from the new files will be appended to the table.
- C. The operation will fail with a `LOAD_UNCERTAIN_FILES` error.
- D. All data from only the newly-added files will be appended to the table.

<details><summary>Show Answer</summary>
Correct Answer: D. By default `COPY INTO` tracks per-file load metadata on the target table and skips files it has already loaded successfully, so only the new files get loaded (no duplication). Use `FORCE = TRUE` to reload everything.
</details>

---

### Question 303
Which parameter can be used to instruct a `COPY` command to verify (validate) data instead of loading it into the target table?

- A. `RETURN_FAILED_ONLY`
- B. `ON_ERROR`
- C. `FORCE`
- D. `VALIDATION_MODE`

<details><summary>Show Answer</summary>
Correct Answer: D. `VALIDATION_MODE` runs the COPY as a dry-run and returns validation results (or errors) without loading data.
</details>

---

### Question 304
Which of the following SQL statements will list the version of the driver/client currently being used?

- A. Execute `SELECT CURRENT_DATABASE()` from the web UI.
- B. Execute `SELECT CURRENT_VERSION()` from SnowSQL.
- C. Execute `SELECT CURRENT_CLIENT()` from an application.
- D. Execute `SELECT CURRENT_SESSION()` from the Python connector.

<details><summary>Show Answer</summary>
Correct Answer: C. `CURRENT_CLIENT()` returns the name and version of the client/driver used to connect to Snowflake (e.g., the JDBC/ODBC/Python connector version). `CURRENT_VERSION()` instead returns the Snowflake service version, not the driver version.
</details>

---

### Question 305
Which Snowflake technique can be used to improve the performance of a query?

- A. Clustering
- B. Indexing
- C. Fragmenting
- D. Using `INDEX_HINTS`

<details><summary>Show Answer</summary>
Correct Answer: A. Snowflake has no traditional indexes — clustering (natural or via an explicit clustering key) is the mechanism that improves micro-partition pruning and query performance.
</details>

---

### Question 306
What happens to the shared objects for users in a consumer account, once a database has been created from a share in that account?

- A. The shared objects are transferred.
- B. The shared objects are copied.
- C. The shared objects become accessible.
- D. The shared objects can be re-shared.

<details><summary>Show Answer</summary>
Correct Answer: C. Creating a database from a share simply exposes read-only access to the provider's objects — nothing is copied or transferred, and by default the consumer cannot re-share.
</details>

---

### Question 307
Using variables in a SnowSQL script is denoted by using which character?

- A. `$`
- B. `@`
- C. `&`
- D. `#`

<details><summary>Show Answer</summary>
Correct Answer: C. SnowSQL variable substitution uses `&variable_name` (with `variable_substitution` enabled). Note this is distinct from session variables referenced in SQL with `$var` via `SET`.
</details>

---

### Question 308
Which commands grant the privilege allowing a role to `SELECT` data from all current tables and any tables that will be created later in a schema? (Choose two)

- A. `GRANT USAGE ON ALL TABLES IN SCHEMA <schema> TO ROLE MYROLE;`
- B. `GRANT USAGE ON FUTURE TABLES IN SCHEMA <schema> TO ROLE MYROLE;`
- C. `GRANT SELECT ON ALL TABLES IN SCHEMA DB.SCHEMA TO ROLE MYROLE;`
- D. `GRANT SELECT ON FUTURE TABLES IN SCHEMA <schema> TO ROLE MYROLE;`
- E. `GRANT SELECT ON ALL TABLES IN DATABASE DB TO ROLE MYROLE;`
- F. `GRANT SELECT ON FUTURE TABLES IN DATABASE DB TO ROLE MYROLE;`

<details><summary>Show Answer</summary>
Correct Answer: C, D. `ON ALL TABLES` grants SELECT on tables that exist right now; `ON FUTURE TABLES` grants it automatically to tables created later in that schema. Both are needed to cover "current and future."
</details>

---

### Question 309
How would a user change which columns are exposed by a view?

- A. Modify the columns in the underlying table.
- B. Use `ALTER VIEW` to change the view's columns.
- C. Recreate the view with the required changes.
- D. Materialize the view to perform the changes.

<details><summary>Show Answer</summary>
Correct Answer: C. `ALTER VIEW` can rename a view, set/unset properties, or manage secure/comment settings — it cannot redefine the view's `SELECT` list or column set. You must `CREATE OR REPLACE VIEW`.
</details>

---

### Question 310
Which statement describes pruning?

- A. The filtering out (disregarding) of micro-partitions that are not needed to satisfy a query.
- B. The return of micro-partition values that overlap with each other to reduce a query's runtime.
- C. A service handled by the Cloud Services layer to optimize caching.
- D. The ability to allow the result of a query to be accessed as if it were a table.

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 311
Which SQL command can be used to see the `CREATE` definition of a masking policy?

- A. `SHOW MASKING POLICIES`
- B. `DESCRIBE MASKING POLICY`
- C. `GET_DDL`
- D. `LIST MASKING POLICIES`

<details><summary>Show Answer</summary>
Correct Answer: C. `SELECT GET_DDL('MASKING POLICY', '<name>')` returns the full CREATE statement.
</details>

---

### Question 312
What is the `ACCOUNT_USAGE.METERING_HISTORY` view used for?

- A. Gathering the hourly credit usage for an account.
- B. Compiling an account's average cloud services cost over the previous month.
- C. Summarizing the throughput of Snowpipe costs for an account.
- D. Calculating the funds left on an account's contract.

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 313
Query parsing and compilation occurs in which architecture layer of the Snowflake platform?

- A. Cloud services layer
- B. Compute layer
- C. Storage layer
- D. Cloud-agnostic layer (not a real layer)

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 314
Which of the following Snowflake objects can be shared using a secure share? (Choose two)

- A. Materialized views
- B. Sequences
- C. Procedures
- D. Tables
- E. Secure User-Defined Functions (UDFs)

<details><summary>Show Answer</summary>
Correct Answer: D, E. Tables, secure views, and secure UDFs can be shared. Sequences, stored procedures, and (non-secure) materialized views cannot.
</details>

---

### Question 315
What happens to underlying table data when a `CLUSTER BY` clause is added to a Snowflake table?

- A. Data is hashed by the cluster key to facilitate fast searches for common data values.
- B. Micro-partitions are created for common data values to reduce the number of partitions scanned.
- C. Smaller micro-partitions are created for common data values to allow for more parallelism.
- D. Data may be co-located by the cluster key within the micro-partitions to improve pruning performance.

<details><summary>Show Answer</summary>
Correct Answer: D. Clustering doesn't hash or resize micro-partitions — it reorganizes (reclusters) existing data so rows with similar cluster-key values land in the same micro-partitions, improving pruning.
</details>

---

### Question 316
Which conditions must be met to return results from the result cache? (Choose two)

- A. The user has the appropriate privileges on the objects associated with the query.
- B. Micro-partitions have been reclustered since the query was last run.
- C. The new query is run using the same virtual warehouse as the previous query.
- D. The query includes a User-Defined Function.
- E. The query has been run within 24 hours of the previously-run query (and underlying data hasn't changed).

<details><summary>Show Answer</summary>
Correct Answer: A, E. The result cache is warehouse-independent — a different warehouse can still serve a cached result, so C is not required. UDF-containing and reclustered-since-last-run queries generally bypass the cache.
</details>

---

### Question 317
Which statement about billing applies to credits?

- A. Credits are billed per-minute with a 60-minute minimum.
- B. Credits are used to pay for cloud data storage usage.
- C. Credits are consumed based on the number of credits billed for each hour a warehouse runs.
- D. Credits are consumed based on the warehouse size and the time the warehouse is running.

<details><summary>Show Answer</summary>
Correct Answer: D. Storage is billed separately (flat per-TB rate), and compute billing is per-second (60-second minimum), not per-minute — so both B and A are wrong.
</details>

---

### Question 318
A user needs to create a materialized view in schema `MYDB.MYSCHEMA`. Which statements will provide this access?

- A. `GRANT ROLE MYROLE TO USER USER1; GRANT CREATE MATERIALIZED VIEW ON SCHEMA MYDB.MYSCHEMA TO ROLE MYROLE;`
- B–D. Variants that omit the role grant to the user, grant to the wrong securable, or reverse the grant direction.

<details><summary>Show Answer</summary>
Correct Answer: A. The user needs the role assigned to them (`GRANT ROLE ... TO USER ...`) **and** that role needs the `CREATE MATERIALIZED VIEW` privilege on the schema (`GRANT CREATE MATERIALIZED VIEW ON SCHEMA ... TO ROLE ...`). Materialized views also require Enterprise Edition or higher.
</details>

---

### Question 319
What is the purpose of multi-cluster virtual warehouses?

- A. To create separate data warehouses to increase query optimization.
- B. To allow users to choose the type of compute nodes that make up a virtual warehouse cluster.
- C. To eliminate or reduce queuing of concurrent queries.
- D. To allow the warehouse to resize automatically.

<details><summary>Show Answer</summary>
Correct Answer: C. Multi-cluster warehouses scale out (add clusters) for concurrency, unlike warehouse resizing which scales up/down for single-query performance.
</details>

---

### Question 320
Which of the following is a valid source for an external stage when the Snowflake account is hosted on Microsoft Azure?

- A. An FTP server with TLS encryption
- B. An HTTPS server with WebDAV
- C. A Microsoft Azure Blob Storage container
- D. A Windows server file share on Azure

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 321
Which database objects can be shared with the Snowflake secure data sharing feature? (Choose two)

- A. Files
- B. External tables
- C. Functions (secure UDFs)
- D. Sequences
- E. Streams

<details><summary>Show Answer</summary>
Correct Answer: B, C.
</details>

---

### Question 322
Which statements reflect key functionalities of a Snowflake Data Exchange? (Choose two)

- A. If an account is enrolled with a Data Exchange, it loses access to the Snowflake Marketplace.
- B. A Data Exchange allows a group of accounts to share data privately among themselves.
- C. A Data Exchange allows accounts to share data with third-party, non-Snowflake parties.
- D. Data Exchange functionality is available by default in accounts using Enterprise Edition or higher.
- E. Sharing in a Data Exchange is bidirectional — an account can be a provider for some datasets and a consumer for others.

<details><summary>Show Answer</summary>
Correct Answer: B, E.
</details>

---

### Question 323
A Snowflake user executed a query and received the results. Another user executed the same query shortly later; the underlying data had not changed. What will occur?

- A. No virtual warehouse will be used — data will be read from the result cache.
- B. No virtual warehouse will be used — data will be read from the local disk cache.
- C. The default virtual warehouse will be used to read all data.
- D. The virtual warehouse defined at the session level will be used to read all data.

<details><summary>Show Answer</summary>
Correct Answer: A. The result cache lives in the Cloud Services layer and is warehouse-independent (unlike the local disk/data cache, which is tied to a specific running warehouse).
</details>

---

### Question 324
Which feature gives a user control over how data is organized within a micro-partition?

- A. Range Partitioning
- B. Search Optimization Service
- C. Automatic Clustering
- D. Horizontal Partitioning

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 325
Which privilege must be granted to a share to allow secure views the ability to reference data in multiple databases?

- A. `CREATE SHARE` on the database
- B. `SHARE` on the databases and schemas
- C. `SELECT` on the tables used by the secure view
- D. `REFERENCE_USAGE` on the databases

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 326
In which use case does Snowflake apply egress charges?

- A. Data sharing within a specific region
- B. Query result retrieval
- C. Database replication across regions/cloud platforms
- D. Loading data into Snowflake

<details><summary>Show Answer</summary>
Correct Answer: C. Egress fees apply when data crosses a region or cloud-platform boundary — e.g., cross-region/cross-cloud replication or unloading data out to a different cloud region. Sharing within the same region does not incur egress charges.
</details>

---

### Question 327
Which of the following compute resources/features are managed **by Snowflake** (i.e., serverless — you don't provision a warehouse for them)? (Choose two)

- A. Executing a `COPY` command
- B. Updating data (DML)
- C. Snowpipe
- D. `AUTOMATIC_CLUSTERING` (automatic reclustering)
- E. Scaling up a warehouse

<details><summary>Show Answer</summary>
Correct Answer: C, D. Both Snowpipe and automatic reclustering run on Snowflake-managed serverless compute rather than a user-managed virtual warehouse.
</details>

---

### Question 328
A materialized view should be created when which of the following occur? (Choose two)

- A. There is minimal cost associated with running the query.
- B. The query consumes many compute resources every time it runs.
- C. The base table gets updated frequently.
- D. The query is highly optimized and does not consume many compute resources.
- E. The results of the query do not change often and are used frequently.

<details><summary>Show Answer</summary>
Correct Answer: B, E. Frequent base-table updates (C) actually argue *against* a materialized view, since Snowflake has to keep re-maintaining it — a classic exam trap.
</details>

---

### Question 329
What privilege should be granted to allow a role to change permissions for objects in a managed access schema?

- A. Grant the `OWNERSHIP` privilege on the schema.
- B. Grant the `OWNERSHIP` privilege on the database.
- C. Grant the `MANAGE GRANTS` global privilege.
- D. Grant `ALL` privileges on the schema.

<details><summary>Show Answer</summary>
Correct Answer: C. In a managed access schema, only the schema owner or a role with the account-level `MANAGE GRANTS` privilege can grant/revoke privileges on objects in that schema — object owners lose that ability.
</details>

---

### Question 330
What happens when a data provider revokes privileges to a share on an object in their source database?

- A. The object immediately becomes unavailable to all data consumers.
- B. Any additional data arriving after this point in time will not be visible to consumers.
- C. Data consumers stop seeing data updates and become responsible for storage charges for the object.
- D. A static copy of the object at the time the privilege was revoked is created in the consumer's account.

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 331
Which command is used to load (upload) local data files into an internal stage?

- A. `LOAD`
- B. `COPY`
- C. `GET`
- D. `PUT`

<details><summary>Show Answer</summary>
Correct Answer: D. `PUT` uploads files from local disk to a stage; `GET` downloads them back down; `COPY INTO <table>` then loads staged files into a table.
</details>

---

### Question 332
What is the MINIMUM Snowflake edition required to use periodic rekeying of micro-partitions?

- A. Enterprise
- B. Business Critical
- C. Standard
- D. Virtual Private Snowflake

<details><summary>Show Answer</summary>
Correct Answer: A. Verified against current documentation — periodic rekeying (`PERIODIC_DATA_REKEYING`) requires Enterprise Edition or higher. (Don't confuse this with Tri-Secret Secure / customer-managed keys, which require Business Critical Edition or higher.)
</details>

---

### Question 333
Which stage type can be altered and dropped?

- A. Database stage (not a real stage type)
- B. External (named) stage
- C. Table stage
- D. User stage

<details><summary>Show Answer</summary>
Correct Answer: B. Only named stages (internal or external) can be created, altered, and dropped with SQL. User stages and table stages are implicit, always exist, and cannot be altered/dropped.
</details>

---

### Question 334
Which Snowflake object enables loading data from files as soon as they are available in a cloud storage location?

- A. Pipe
- B. External stage
- C. Task
- D. Stream

<details><summary>Show Answer</summary>
Correct Answer: A. A pipe is the object that wraps a COPY statement for Snowpipe continuous/event-driven loading.
</details>

---

### Question 335
A user is loading JSON documents composed of a huge array containing multiple records into Snowflake and enables the `STRIP_OUTER_ARRAY` file format option. What does this option do?

- A. It removes the last element of the outer array.
- B. It removes the outer array structure and loads the records into separate table rows.
- C. It strips trailing spaces in the last element of the array and loads records into separate table columns.
- D. It removes NULL elements from the JSON object, enabling the load.

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 336
Which of the following describes how multiple Snowflake accounts in a single organization relate to various cloud providers?

- A. Each Snowflake account can be hosted in a different cloud vendor and region.
- B. Each Snowflake account must be hosted in a different cloud vendor and region.
- C. All accounts must be hosted in the same cloud vendor and region.
- D. Each Snowflake account can be hosted in a different cloud vendor, but must be in the same region.

<details><summary>Show Answer</summary>
Correct Answer: A. An organization can contain accounts spread freely across AWS, Azure, and GCP and across different regions.
</details>

---

### Question 337
If a Snowflake user decides a table should be clustered, what should be used as the cluster key?

- A. The columns queried in the `SELECT` clause.
- B. The columns with very high cardinality.
- C. The columns with many distinct values.
- D. The columns most actively used in `SELECT` filters (`WHERE` clauses).

<details><summary>Show Answer</summary>
Correct Answer: D. Good clustering keys have moderate cardinality and appear frequently in filter predicates — very high-cardinality columns (B, C) actually make poor clustering keys.
</details>

---

### Question 338
What value types can a `VARIANT` column store? (Choose two)

- A. `STRUCT`
- B. `OBJECT`
- C. `BINARY`
- D. `ARRAY`

<details><summary>Show Answer</summary>
Correct Answer: B, D. `VARIANT` can hold semi-structured `OBJECT` and `ARRAY` values (plus scalars) — `STRUCT` isn't a Snowflake data type, and `BINARY` is its own separate type.
</details>

---

### Question 339
A company needs to load multiple terabytes of data for an initial load as part of a Snowflake migration, and it can control the number and size of its CSV extract files. How should Snowflake recommend maximizing load throughput?

- A. Use auto-ingest Snowpipe to load large files in a serverless model.
- B. Produce the largest files possible, reducing the number of files to load.
- C. Produce a larger number of similarly-sized smaller files and process the ingestion with an appropriately-sized (or multi-cluster) virtual warehouse.
- D. Use an ETL tool to issue row-by-row inserts within `BEGIN TRANSACTION`/`COMMIT` blocks.

<details><summary>Show Answer</summary>
Correct Answer: C. Snowflake loads files in parallel across the nodes of a warehouse, so splitting data into many similarly-sized files (Snowflake recommends roughly 100–250 MB compressed) — rather than one giant file — maximizes parallelism and load speed.
</details>

---

### Question 340
For non-materialized views, what column in `INFORMATION_SCHEMA`/`ACCOUNT_USAGE` identifies whether a view is secure or not?

- A. `CHECK_OPTION`
- B. `IS_SECURE`
- C. `IS_UPDATABLE`
- D. `TABLE_NAME`

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 341
The bulk data load history available upon completion of a `COPY` statement is stored where, and for how long?

- A. In the metadata of the target table, for 14 days
- B. In the metadata of the pipe, for 14 days
- C. In the metadata of the target table, for 64 days
- D. In the metadata of the pipe, for 64 days

<details><summary>Show Answer</summary>
Correct Answer: C. Verified against current documentation. Bulk `COPY INTO <table>` load metadata lives on the target table for 64 days; Snowpipe load history instead lives on the pipe object for 14 days (see Question 386).
</details>

---

### Question 342
User `INQUISITIVE_PERSON` has been granted the role `DATA_SCIENCE`. The role `DATA_SCIENCE` has `OWNERSHIP` on schema `MARKETING` of database `ANALYTICS_DW`. Which command will show all privileges granted **on** that schema?

- A. `SHOW GRANTS ON ROLE DATA_SCIENCE`
- B. `SHOW GRANTS ON SCHEMA ANALYTICS_DW.MARKETING`
- C. `SHOW GRANTS TO USER INQUISITIVE_PERSON`
- D. `SHOW GRANTS OF ROLE DATA_SCIENCE`

<details><summary>Show Answer</summary>
Correct Answer: B. `SHOW GRANTS ON <object>` lists privileges granted on that securable; `SHOW GRANTS TO ROLE`/`TO USER` lists privileges *held by* a role/user; `SHOW GRANTS OF ROLE` lists who the role is granted to.
</details>

---

### Question 343
Which of the following is an accurate characteristic of security in Snowflake?

- A. Account and authentication features are only available with Business Critical Edition.
- B. Support for HIPAA and GDPR compliance is available for all Snowflake editions.
- C. Periodic rekeying of encrypted data is available with Enterprise Edition and higher.
- D. Private connectivity to internal stages is allowed in Enterprise Edition and higher.

<details><summary>Show Answer</summary>
Correct Answer: C. HIPAA compliance specifically requires Business Critical Edition (a signed BAA), so B is wrong; private connectivity (PrivateLink) requires Business Critical Edition, so D is wrong too.
</details>

---

### Question 344
Which of the following objects can be shared through secure data sharing?

- A. Masking policy
- B. Stored procedure
- C. Task
- D. External table

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 345
Which of the following are valid semi-structured value types that Snowflake can store/represent? (Choose two)

- A. GeoJSON
- B. Array
- C. XML
- D. Object
- E. BLOB

<details><summary>Show Answer</summary>
Correct Answer: B, D (`ARRAY` and `OBJECT`, the two semi-structured value kinds a `VARIANT` column represents).
⚠ **Note:** this question is heavily OCR-corrupted in the source and its wording doesn't map cleanly to a documented Snowflake concept — the reconstruction above is a best-effort inference (it mirrors Question 338's answer pattern) and carries lower confidence than the others in this set. Verify against your original source if this one shows up on a practice test.
</details>

---

### Question 346
A user is preparing to load data from an external stage. Which practice will provide the MOST efficient loading performance?

- A. Organize files into logical paths (partitioned prefixes).
- B. Store the files on the external stage to ensure caching is maintained.
- C. Use pattern matching (regular expressions) to select files.
- D. Load the data in one large file.

<details><summary>Show Answer</summary>
Correct Answer: A. Path-based organization lets Snowflake load a targeted subset of files and enables concurrent `COPY` statements against different prefixes; regex pattern matching (C) is actually slower than an explicit file list, and one giant file (D) kills parallelism.
</details>

---

### Question 347
What effect does setting `WAIT_FOR_COMPLETION = TRUE` have when running an `ALTER WAREHOUSE` command to change the warehouse size?

- A. The warehouse size does not change until all queries currently running in the warehouse have completed.
- B. The warehouse size does not change until all queries currently queued in the warehouse have completed.
- C. The warehouse size does not change until the warehouse is suspended and restarted.
- D. The command does not return control until the warehouse has finished changing size.

<details><summary>Show Answer</summary>
Correct Answer: D. It's a synchronous-vs-asynchronous switch for the ALTER statement itself, not a delay on when resizing starts.
</details>

---

### Question 348
Which of the following can be used when unloading data from Snowflake? (Choose two)

- A. When unloading semi-structured data, it is recommended to use `FILE_EXTENSION`.
- B. Use the `ENCODING` file format option to change the encoding from the default.
- C. The `OBJECT_CONSTRUCT` function can be used to convert relational data to semi-structured data before unloading.
- D. By using the `SINGLE = TRUE` parameter, a single file up to 5 GB in size can be exported to the storage layer.
- E. Use the `PARSE_JSON` function to ensure structured data is unloaded into the `VARIANT` data type.

<details><summary>Show Answer</summary>
Correct Answer: C, D.
</details>

---

### Question 349
What data is stored in the Snowflake storage layer? (Choose two)

- A. Snowflake parameters
- B. Micro-partitions
- C. Query history
- D. Persisted query results
- E. Standard and secure view results

<details><summary>Show Answer</summary>
Correct Answer: B, C.
</details>

---

### Question 350
A data provider wants to share data with someone who does not have a Snowflake account. The provider creates a reader account, adds a user, creates a database and an X-Small warehouse for querying the data, and grants the `PUBLIC` role `USAGE` on the warehouse/database/schema and `SELECT` on the shared objects. Based on this configuration, what is true of the reader account?

- A. The reader account will automatically use Standard Edition.
- B. The reader account's compute will be billed to the provider account.
- C. The reader account can clone data the provider has shared, but cannot re-share it.
- D. The reader account can create a copy of the shared data using `CREATE TABLE AS SELECT`.

<details><summary>Show Answer</summary>
Correct Answer: B. Reader accounts have no billing relationship of their own — all their compute/storage costs are billed back to the provider account that created them.
</details>

---

### Question 351
Which of the following activities consume virtual warehouse credits? (Choose two)

- A. Caching query results
- B. Running `EXPLAIN` and `SHOW` commands (metadata-only, served by Cloud Services)
- C. Cloning a database (metadata operation)
- D. Running a custom query
- E. Running `COPY` commands

<details><summary>Show Answer</summary>
Correct Answer: D, E.
</details>

---

### Question 352
When loading data into Snowflake, the `COPY` command supports which of the following?

- A. Joins
- B. Filters
- C. Column reordering
- D. Aggregates

<details><summary>Show Answer</summary>
Correct Answer: C. `COPY INTO <table>` supports a limited transformation SELECT — column reordering, casts, and simple expressions — but not joins, filters (`WHERE`), or aggregates.
</details>

---

### Question 353
What is cached during a query on a virtual warehouse?

- A. All columns in a micro-partition
- B. Any columns accessed during the query
- C. Only the columns in the result set of the query
- D. All rows accessed during the query

<details><summary>Show Answer</summary>
Correct Answer: B. The warehouse's local SSD cache stores the raw columnar data that was scanned to answer the query — not just the final result columns.
</details>

---

### Question 354
What is the default character set used when loading CSV files into Snowflake?

- A. UTF-8
- B. UTF-16
- C. ISO-8859-1
- D. ANSI X3.4

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 355
Which of the following describes external functions in Snowflake?

- A. They are a type of User-Defined Function.
- B. They contain their own SQL code.
- C. They call code that is stored inside of Snowflake.
- D. They can return multiple rows for each row received.

<details><summary>Show Answer</summary>
Correct Answer: A. An external function is a UDF whose handler code runs outside Snowflake (e.g., in AWS Lambda/API Gateway) and is invoked over HTTPS — it must return exactly one row per input row.
</details>

---

### Question 356
Which of the following are valid methods for authenticating users into Snowflake? (Choose three)

- A. SCIM (user provisioning, not authentication)
- B. Federated authentication (SSO)
- C. Basic username/password authentication
- D. Key-pair authentication
- E. OAuth
- F. OCSP (certificate revocation checking, not authentication)

<details><summary>Show Answer</summary>
Correct Answer: B, D, E.
</details>

---

### Question 357
A user has a standard multi-cluster warehouse auto-scaling policy in place. Which condition will trigger a cluster to shut down?

- A. After 2–3 consecutive checks, the system determines the load on the **most**-loaded cluster could be redistributed.
- B. After 5–6 consecutive checks, the system determines the load on the **most**-loaded cluster could be redistributed.
- C. After 5–8 consecutive checks, the system determines the load on the **least**-loaded cluster could be redistributed.
- D. After 2–3 consecutive checks, the system determines the load on the **least**-loaded cluster could be redistributed.

<details><summary>Show Answer</summary>
Correct Answer: D. Verified against current documentation — under the Standard scaling policy, Snowflake checks the least-loaded cluster every minute and shuts it down after 2–3 consecutive checks confirm its load can be absorbed elsewhere. (The Economy policy uses 5–6 checks instead.)
</details>

---

### Question 358
What is the minimum Snowflake edition needed for database failover and fail-back between Snowflake accounts, for business continuity and disaster recovery?

- A. Standard
- B. Enterprise
- C. Business Critical
- D. Virtual Private Snowflake

<details><summary>Show Answer</summary>
Correct Answer: C. Verified — account/database failover and fail-back is a Business Critical Edition (or higher) feature. (Basic database replication without automated failover is available more broadly.)
</details>

---

### Question 359
How would a user execute a series of SQL statements using a single task?

- A. Chain multiple raw SQL statements directly in the task body.
- B. Sequence multiple stored procedure calls directly in the task body (a task supports only one statement).
- C. Wrap the multiple SQL statements inside a single stored procedure, and have the task call that procedure: `CREATE TASK mytask AS CALL my_procedure();`
- D. Create one task per SQL statement and chain them (task1 → task2 → …) using task dependencies.

<details><summary>Show Answer</summary>
Correct Answer: C. A task executes exactly one SQL statement (or one procedure call) per run — a multi-statement stored procedure is the standard way to run a sequence of statements from one task.
</details>

---

### Question 360
How many resource monitors can be assigned at the account level?

<details><summary>Show Answer</summary>
Correct Answer: 1. Verified against current documentation — only a single resource monitor can be set at the account level at any time (though many warehouse-level resource monitors can exist, and a warehouse can be assigned to at most one of them).
</details>

---

### Question 361
Data storage for individual tables can be monitored using which commands/objects? (Choose two)

- A. `SHOW STORAGE FOR TABLE`
- B. `SHOW TABLES`
- C. `INFORMATION_SCHEMA.TABLE_HISTORY`
- D. `INFORMATION_SCHEMA.TABLE_FUNCTION`
- E. `ACCOUNT_USAGE.TABLE_STORAGE_METRICS`

<details><summary>Show Answer</summary>
Correct Answer: B, E. `SHOW TABLES` returns each table's `bytes` (storage) column; `TABLE_STORAGE_METRICS` in `ACCOUNT_USAGE` gives detailed active/Time Travel/Fail-safe/clone storage bytes.
</details>

---

### Question 362
How would a user run a multi-cluster warehouse in maximized mode?

- A. Configure the maximum clusters setting to "Maximum."
- B. Turn on additional clusters manually after starting the warehouse.
- C. Set the minimum clusters and maximum clusters settings to the **same** value.
- D. Set the minimum clusters and maximum clusters settings to **different** values.

<details><summary>Show Answer</summary>
Correct Answer: C. Equal min/max clusters puts the warehouse in Maximized mode, where all clusters start immediately and run continuously; different min/max values put it in Auto-scale mode instead.
</details>

---

### Question 363
What internal stages are available in Snowflake? (Choose three)

- A. Schema stage (not a real stage type)
- B. Named stage
- C. User stage
- D. Stream stage (not a real stage type)
- E. Table stage
- F. Database stage (not a real stage type)

<details><summary>Show Answer</summary>
Correct Answer: B, C, E. Snowflake provides exactly three kinds of internal stage: named, user, and table stages.
</details>

---

### Question 364
Which stages are used with the `PUT` command to upload files from a local file system? (Choose three)

- A. Schema stage
- B. User stage
- C. Database stage
- D. Table stage
- E. External named stage
- F. Internal named stage

<details><summary>Show Answer</summary>
Correct Answer: B, D, F. `PUT` only works against *internal* stages (user, table, or internal named) — it cannot push files to an external stage, since Snowflake doesn't manage the credentials/API of external cloud storage the same way.
</details>

---

### Question 365
Which data type can store more than one type of data structure?

- A. JSON (a format, not a Snowflake data type)
- B. BINARY
- C. VARCHAR
- D. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 366
User-level network policies can be created and applied by which of the following roles? (Choose two)

- A. `ROLEADMIN` (not a default system role)
- B. `ACCOUNTADMIN`
- C. `SYSADMIN`
- D. `SECURITYADMIN`
- E. `USERADMIN`

<details><summary>Show Answer</summary>
Correct Answer: B, D. Network policies fall under `SECURITYADMIN`'s security-management scope, and `ACCOUNTADMIN` inherits everything `SECURITYADMIN` can do.
</details>

---

### Question 367
What SQL command would be used to view all roles that have been granted to `USER1`?

- A. `SHOW GRANTS TO USER1;` (invalid syntax)
- B. `SHOW GRANTS TO USER USER1;`
- C. `DESCRIBE USER USER1;`
- D. `SHOW GRANTS ON USER USER1;`

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 368
Which `ACCOUNT_USAGE` views are used to evaluate the details of dynamic data masking? (Choose two)

- A. `ROLES`
- B. `POLICY_REFERENCES`
- C. `QUERY_HISTORY`
- D. `RESOURCE_MONITORS`
- E. `MASKING_POLICIES`

<details><summary>Show Answer</summary>
Correct Answer: B, E. `MASKING_POLICIES` shows the defined policies; `POLICY_REFERENCES` shows what objects/columns each policy is actually attached to.
</details>

---

### Question 369
Which of the following are considerations when using a directory table when working with unstructured data? (Choose two)

- A. A directory table is a separate database object.
- B. Directory tables store data file metadata.
- C. A directory table will be automatically added to a stage.
- D. Directory tables do not have their own grantable privileges.
- E. Directory table data cannot be refreshed manually.

<details><summary>Show Answer</summary>
Correct Answer: B, D. A directory table isn't a standalone object — it's an implicit metadata layer attached to a stage (must be explicitly enabled via `DIRECTORY = (ENABLE = TRUE)`), and it inherits its privileges from the stage rather than having its own grantable privileges.
</details>

---

### Question 370
The first user assigned to a new account, using `ACCOUNTADMIN`, should create at least one additional user with which administrative privilege?

- A. `USERADMIN`
- B. `PUBLIC`
- C. `ORGADMIN`
- D. `SYSADMIN`

<details><summary>Show Answer</summary>
Correct Answer: A. Best practice is to avoid daily-driving `ACCOUNTADMIN` — create a `USERADMIN` user to handle ongoing user/role management instead.
</details>

---

### Question 371
Which statement describes how Snowflake supports reader accounts?

- A. A reader account can consume data from the provider account that created it and combine it with its own data.
- B. A consumer needs to become a licensed Snowflake customer, since data sharing is only supported between Snowflake accounts.
- C. Users in a reader account can query data shared with the account and can create their own tasks.
- D. The `SHOW MANAGED ACCOUNTS` command will list all reader accounts that have been created for an account.

<details><summary>Show Answer</summary>
Correct Answer: D. Reader accounts (a type of "managed account") cannot load their own data or combine it with shared data — they exist purely to query what's been shared with them.
</details>

---

### Question 372
Can a data provider with an Azure account in Central Canada share data with a data consumer on AWS in Australia?

- A. The provider in Azure Central Canada can create a direct share to AWS Asia Pacific, if both are in the same organization.
- B. The consumer and provider can form a Data Exchange within the same organization to share across regions/clouds.
- C. The provider can use the "Get Data" workflow in Snowflake Marketplace to bridge Azure Central Canada and AWS Asia Pacific.
- D. The provider must replicate the database to a secondary account in AWS Asia Pacific (within the same organization), then share from that secondary account.

<details><summary>Show Answer</summary>
Correct Answer: D. Direct secure shares only work between accounts on the same cloud platform and in the same region — crossing cloud/region boundaries requires replicating the database to an account on the target platform/region first.
</details>

---

### Question 373
Which Snowflake objects can be shared with other Snowflake accounts? (Choose three)

- A. Schemas
- B. Roles
- C. Secure Views
- D. Stored Procedures
- E. Tables
- F. Functions (Secure UDFs)

<details><summary>Show Answer</summary>
Correct Answer: C, E, F.
</details>

---

### Question 374
Which Snowflake feature will allow small volumes of data to continuously load into Snowflake, incrementally making it available for analysis?

- A. `COPY INTO`
- B. `CREATE PIPE` (Snowpipe)
- C. `INSERT INTO`
- D. `TABLE STREAM`

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 375
Which partner specializes in data catalog solutions in the Snowflake ecosystem?

- A. Alation
- B. DataRobot
- C. dbt
- D. Tableau

<details><summary>Show Answer</summary>
Correct Answer: A. Alation is a data-catalog/governance partner; DataRobot is ML/AutoML, dbt is transformation, and Tableau is BI/visualization.
</details>

---

### Question 376
Which of the following can be executed/called by Snowpipe?

- A. A User-Defined Function
- B. A stored procedure
- C. A single `COPY INTO <table>` statement
- D. A single `INSERT INTO` statement

<details><summary>Show Answer</summary>
Correct Answer: C. A pipe wraps exactly one `COPY INTO <table>` statement.
</details>

---

### Question 377
Which Snowflake objects will incur both storage and compute charges? (Choose two)

- A. Materialized view
- B. Sequence
- C. View
- D. Transient table
- E. A table using Automatic Clustering

<details><summary>Show Answer</summary>
Correct Answer: A, E. Materialized views incur storage plus background serverless compute to stay in sync with the base table; a clustered table incurs storage plus serverless compute for automatic reclustering. Plain views and sequences store no data of their own, and a transient table only incurs storage (its query compute is no different from any other table).
</details>

---

### Question 378
Which file formats does Snowflake support for loading semi-structured data? (Choose three)

- A. CSV
- B. JSON
- C. PDF
- D. Avro
- E. Parquet
- F. JPEG

<details><summary>Show Answer</summary>
Correct Answer: B, D, E. (Snowflake also supports ORC and XML for semi-structured loading, but those weren't offered as options here.)
</details>

---

### Question 379
Which of the following statements about data sharing are true? (Choose two)

- A. New objects created by a data provider are automatically shared with existing consumers and reader accounts.
- B. All database objects can be included in a shared database.
- C. Reader accounts are created by data providers.
- D. Shared databases are read-only.
- E. Reader accounts are charged for their own warehouse usage.

<details><summary>Show Answer</summary>
Correct Answer: C, D.
</details>

---

### Question 380
Credit charges for Snowflake virtual warehouses are based on which of the following? (Choose two)

- A. The number of queries executed
- B. The number of active clusters assigned to the warehouse
- C. The size of the virtual warehouse
- D. The length of time the warehouse is running
- E. The duration of the queries executed

<details><summary>Show Answer</summary>
Correct Answer: C, D.
</details>

---

### Question 381
Which of the following are handled by the Cloud Services layer of the Snowflake architecture? (Choose two)

- A. Data loading (compute layer)
- B. Query execution (compute layer)
- C. Time Travel data (storage layer)
- D. Security
- E. Authentication and access control

<details><summary>Show Answer</summary>
Correct Answer: D, E. Cloud Services handles authentication, infrastructure/metadata management, security, query parsing/optimization, and orchestration — not the actual data loading, query execution, or storage of Time Travel data.
</details>

---

### Question 382
What is a responsibility of Snowflake's virtual warehouses?

- A. Infrastructure management
- B. Metadata management
- C. Query execution
- D. Query parsing and optimization
- E. Permanent storage of micro-partitions

<details><summary>Show Answer</summary>
Correct Answer: C. Warehouses (Compute layer) execute queries; parsing/optimization/metadata/infrastructure management belong to the Cloud Services layer, and permanent storage belongs to the Storage layer.
</details>

---

### Question 383
What features does Snowflake Time Travel enable?

- A. Querying data-related objects that were created within the past 365 days.
- B. Restoring data-related objects that have been deleted within the past 90 days.
- C. Conducting point-in-time historical analysis.
- D. Analyzing data usage/manipulation over all periods of time (unlimited).

<details><summary>Show Answer</summary>
Correct Answer: C. Time Travel's retention window tops out at 90 days (Enterprise Edition+, on permanent objects), not 365 days, and it doesn't cover "all periods of time" — B and D overstate the retention window/scope.
</details>

---

### Question 384
Which of the following statements describes a schema in Snowflake?

- A. A logical grouping of objects that belongs to a single database.
- B. A grouping of objects that belongs to multiple databases.
- C. A named Snowflake object that includes all the information required to share a database.
- D. A uniquely identified Snowflake account within a business entity.

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 385
What is the recommended compressed file size range for continuous data loads using Snowpipe?

- A. 1–16 MB
- B. 16–24 MB
- C. 50–100 MB
- D. 100–250 MB

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 386
How long is Snowpipe data load history retained?

- A. As configured in the `CREATE PIPE` settings
- B. Until the pipe is dropped
- C. 64 days
- D. 14 days

<details><summary>Show Answer</summary>
Correct Answer: D. Verified against current documentation — this is stored in the pipe's own metadata for 14 days, unlike bulk-load history (target table metadata, 64 days — see Question 341).
</details>

---

### Question 387
Snowflake users self-enrolling in the default Multi-Factor Authentication (MFA) service need to install which application on their devices to connect with MFA?

- A. Okta Verify
- B. Duo Mobile
- C. Microsoft Authenticator
- D. Google Authenticator

<details><summary>Show Answer</summary>
Correct Answer: B. Snowflake's native MFA is powered by the Duo Security service, so the Duo Mobile app is what's required (though it also supports SMS/phone-call fallback through Duo).
</details>

---

### Question 388
Which URL type allows users to access unstructured data without authenticating into Snowflake or passing an authorization token?

- A. Pre-signed URL
- B. Scoped URL
- C. Signed URL (not a Snowflake term)
- D. File URL

<details><summary>Show Answer</summary>
Correct Answer: A. A pre-signed URL is temporary and self-contained — no Snowflake session or token is needed. A scoped URL (B) still requires an active Snowflake session/token to resolve, and a file URL (D) requires both a role with the READ privilege on the stage and a valid access token.
</details>

---

### Question 389
Where would a Snowflake user find information about query activity from 90 days ago?

- A. `ACCOUNT_USAGE.QUERY_HISTORY` view
- B. `INFORMATION_SCHEMA.QUERY_HISTORY` view
- C. The Snowsight "Query History" page (UI)
- D. `INFORMATION_SCHEMA.QUERY_HISTORY` table function

<details><summary>Show Answer</summary>
Correct Answer: A. `INFORMATION_SCHEMA`'s query history views/functions only cover roughly the last 7 days–6 months depending on the function, with tighter limits; `ACCOUNT_USAGE.QUERY_HISTORY` retains history for 365 days, which is what's needed to reach back 90 days reliably.
</details>

---

### Question 390
A marketing co-worker has requested the ability to change a warehouse's size on their Medium virtual warehouse, `MKTG_WH`. Which statement will accommodate this?

- A. `ALLOW RESIZE ON WAREHOUSE MKTG_WH TO USER MKTG_LEAD;` (not valid SQL)
- B. `GRANT MODIFY ON WAREHOUSE MKTG_WH TO ROLE MARKETING;`
- C. `GRANT USAGE ON WAREHOUSE MKTG_WH TO USER MKTG_LEAD;`
- D. `GRANT OPERATE ON WAREHOUSE MKTG_WH TO ROLE MARKETING;`

<details><summary>Show Answer</summary>
Correct Answer: B.
⚠ **Updated:** the source material's marked answer for this question was illegible/ambiguous in the raw OCR, so this is presented as freshly verified rather than corrected. This is also a classic exam trap: `OPERATE` only lets you start/stop/suspend/resume a warehouse — **resizing a warehouse's properties (including `WAREHOUSE_SIZE`) requires the `MODIFY` privilege**, not `OPERATE`.
</details>

---

### Question 391
Which of the following commands **cannot** be used within a reader account?

- A. `CREATE SHARE`
- B. `ALTER WAREHOUSE`
- C. `CREATE ROLE`
- D. `SHOW SCHEMAS`
- E. `DESCRIBE TABLE`

<details><summary>Show Answer</summary>
Correct Answer: A. Reader accounts can only consume shared data — they have no data of their own to share, so they can't create outbound shares.
</details>

---

### Question 392
Which table function helps convert semi-structured data into a relational representation?

- A. `CHECK_JSON`
- B. `TO_JSON`
- C. `FLATTEN`
- D. `PARSE_JSON`

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 393
Which query statistics help determine whether efficient pruning is occurring? (Choose two)

- A. Bytes sent over the network
- B. Percentage scanned from cache
- C. Partitions total
- D. Bytes spilled to local storage
- E. Partitions scanned

<details><summary>Show Answer</summary>
Correct Answer: C, E. Comparing "partitions scanned" to "partitions total" (both shown in the query profile) tells you how effective pruning was for that query.
</details>

---

### Question 394
What are the default Time Travel and Fail-safe retention periods for transient tables?

- A. Time Travel — 0 days, Fail-safe — 1 day
- B. Time Travel — 0 days, Fail-safe — 0 days
- C. Time Travel — 1 day, Fail-safe — 0 days
- D. Transient tables are retained in neither Fail-safe nor Time Travel.

<details><summary>Show Answer</summary>
Correct Answer: C. Transient tables get the standard default of 1 day of Time Travel (configurable 0–1 day) but, unlike permanent tables, they get **no** Fail-safe period at all.
</details>

---

### Question 395
Which command is used to unload data from a Snowflake table into a file in a stage?

- A. `COPY INTO`
- B. `GET`
- C. `WRITE`
- D. `EXTRACT INTO`

<details><summary>Show Answer</summary>
Correct Answer: A. `COPY INTO <location>` (the same command family, just pointed at a stage instead of a table) unloads data.
</details>

---

### Question 396
What are advantages clones have over tables created with a `CREATE TABLE AS SELECT` statement? (Choose two)

- A. The clone always stays in sync with the original table.
- B. The clone has better query performance.
- C. The clone is created almost instantly.
- D. The clone will have Time Travel history from the original table.
- E. The clone saves space by not duplicating storage.

<details><summary>Show Answer</summary>
Correct Answer: C, E. Zero-copy cloning is a metadata-only operation (instant, no storage duplication at creation time — storage is only consumed as the clone and original diverge). It is a point-in-time snapshot, though, so A and D are wrong — a clone does *not* stay in sync with later changes, and it does not inherit the original's historical Time Travel data.
</details>

---

### Question 397
How often are the Account and Table master keys automatically rotated by Snowflake?

- A. 30 days
- B. 60 days
- C. 90 days
- D. 365 days

<details><summary>Show Answer</summary>
Correct Answer: A. Verified against current documentation — Snowflake-managed keys are automatically rotated once they're more than 30 days old. (Don't confuse this with **periodic rekeying**, an optional feature that re-encrypts data with a brand-new key once a retired key has been inactive for a full year.)
</details>

---

### Question 398
Which privilege is required for a role to be able to resume a suspended warehouse if auto-resume is not enabled?

- A. `USAGE`
- B. `OPERATE`
- C. `MONITOR`
- D. `MODIFY`

<details><summary>Show Answer</summary>
Correct Answer: B. Note `OPERATE` implicitly includes `USAGE`, but on its own `USAGE` cannot start/resume a warehouse.
</details>

---

### Question 399
Which statement MOST accurately describes clustering in Snowflake?

- A. The `ACCOUNTADMIN` must define the clustering methodology for each Snowflake table.
- B. Clustering is the way data is grouped together and stored within Snowflake micro-partitions.
- C. The clustering key must be included in the `COPY` command when loading data into Snowflake.
- D. Clustering can be disabled within a Snowflake account.

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 400
Which of the following practices are recommended when creating a user in Snowflake? (Choose two)

- A. Configure the user to be initially disabled.
- B. Force an immediate password change on first login.
- C. Set a default role for the user.
- D. Set the number of minutes to unlock the account to 15 minutes.
- E. Set the user's access to expire within a specified timeframe.

<details><summary>Show Answer</summary>
Correct Answer: B, C.
</details>

---

### Question 401
Network policies can be applied to which of the following objects? (Choose two)

- A. Roles
- B. Databases
- C. Warehouses
- D. Users
- E. Accounts

<details><summary>Show Answer</summary>
Correct Answer: D, E. A network policy can be set at the account level (applies to everyone) or attached to individual users to override the account-level policy.
</details>

---

### Question 402
Where is Snowflake metadata stored?

- A. Within the data files
- B. In the virtual warehouse layer
- C. In the Cloud Services layer
- D. In the remote storage layer

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 403
What columns are returned when performing a `FLATTEN` operation on semi-structured data? (Choose two)

- A. `KEY`
- B. `NODE`
- C. `VALUE`
- D. `LEVEL`
- E. `ROOT`

<details><summary>Show Answer</summary>
Correct Answer: A, C. (`FLATTEN` actually returns `SEQ`, `KEY`, `PATH`, `INDEX`, `VALUE`, and `THIS` — of the options offered here, `KEY` and `VALUE` are the real ones.)
</details>

---

### Question 404
Which of the following Snowflake features provide continuous data protection? (Choose two)

- A. Internal stages
- B. Backups (not a Snowflake concept — protection is built in)
- C. Time Travel
- D. Zero-copy clones
- E. Fail-safe

<details><summary>Show Answer</summary>
Correct Answer: C, E. Time Travel (user-recoverable) and Fail-safe (Snowflake-recoverable, non-configurable) together make up Snowflake's Continuous Data Protection lifecycle.
</details>

---

### Question 405
A developer is granted ownership of a table that has a masking policy applied. The developer's role is not able to see the masked data. Will the developer be able to modify the table to read the masked data?

- A. Yes, because a table owner has full control and can unset masking policies.
- B. Yes, because masking policies only apply to cloned tables.
- C. No, because masking policies must always reference specific access roles.
- D. No, because ownership of a table does not include the ability to change masking policies.

<details><summary>Show Answer</summary>
Correct Answer: D. Managing masking policies (`APPLY MASKING POLICY`) is a separate, governance-level privilege — table `OWNERSHIP` does not automatically grant it. This is by design, so table owners can't casually bypass column-level security.
</details>

---

### Question 406
How should a virtual warehouse be configured to ensure that additional clusters resume with the least possible delay?

- A. Set the warehouse to a size larger than generally needed.
- B. Configure Auto-scale mode with a wide min/max cluster range.
- C. Use the Standard warehouse scaling policy.
- D. Use the Economy warehouse scaling policy.

<details><summary>Show Answer</summary>
Correct Answer: C. Of the two scaling policies, Standard is the responsiveness-first option — it starts a new cluster immediately on queuing (with a fixed ~20-second stagger between clusters), whereas Economy deliberately waits to confirm ~6 minutes of sustained demand before adding a cluster. (For literally zero delay, Maximized mode — equal min/max clusters, all running continuously — is the strongest option, but it isn't one of the choices offered here.)
</details>

---

### Question 407
During periods of warehouse contention, which parameter controls the maximum length of time a warehouse will hold a query for processing before canceling it?

- A. `STATEMENT_TIMEOUT_IN_SECONDS`
- B. `STATEMENT_QUEUED_TIMEOUT_IN_SECONDS`
- C. `LOCK_TIMEOUT`
- D. `QUERY_TIMEOUT_IN_SECONDS`

<details><summary>Show Answer</summary>
Correct Answer: B. This specifically governs how long a query can sit *queued* on a busy warehouse before Snowflake cancels it, distinct from `STATEMENT_TIMEOUT_IN_SECONDS`, which limits total execution time once a query is actually running.
</details>

---

### Question 408
Files have been uploaded to a Snowflake internal stage. The files now need to be deleted. Which SQL command should be used?

- A. `PURGE` (not a standalone command — it's a COPY option)
- B. `MODIFY`
- C. `REMOVE`
- D. `DELETE`

<details><summary>Show Answer</summary>
Correct Answer: C. `REMOVE` (or its alias `RM`) deletes files from a stage. `DELETE` removes *rows from a table*, not stage files.
</details>

---

## Summary of documentation-verified corrections/confirmations

| # | Topic | Verified answer |
|---|---|---|
| 332 | Min. edition for periodic rekeying | Enterprise |
| 341 | Bulk `COPY` load history location/retention | Target table metadata, 64 days |
| 357 | Standard scaling policy cluster shutdown | 2–3 checks, least-loaded cluster |
| 358 | Min. edition for account failover/fail-back | Business Critical |
| 360 | Resource monitors allowed at account level | 1 |
| 386 | Snowpipe load history location/retention | Pipe metadata, 14 days |
| 387 | Native MFA app | Duo Mobile (Duo Security service) |
| 390 | Privilege to resize a warehouse | `MODIFY` (not `OPERATE`) |
| 397 | Account/table master key rotation interval | 30 days |

# SnowPro Core Practice Questions — Cleaned & Verified

> Reconstructed from raw OCR text, corrected against official Snowflake documentation as of July 2026. Answers are hidden in collapsible blocks — click **Show Answer** to reveal. Where the community/original answer was outdated or wrong, it is flagged with **⚠ Updated**.

---

### Question 1
In a Snowflake role hierarchy, what is the top-level role?

- A. SYSADMIN
- B. ORGADMIN
- C. ACCOUNTADMIN
- D. SECURITYADMIN

<details><summary>Show Answer</summary>
Correct Answer: C. ACCOUNTADMIN sits at the top of the standard account-level role hierarchy (SYSADMIN and SECURITYADMIN both roll up into it). ORGADMIN is a separate, organization-level role used to manage multiple accounts — it is not part of the in-account role hierarchy diagram.
</details>

---

### Question 2
By default, which Snowflake role is required to create a share?

- A. ORGADMIN
- B. SECURITYADMIN
- C. SHAREADMIN
- D. ACCOUNTADMIN

<details><summary>Show Answer</summary>
Correct Answer: D. ACCOUNTADMIN holds the CREATE SHARE privilege by default. There is no built-in "SHAREADMIN" role in Snowflake.
</details>

---

### Question 3
What happens to historical data when the retention period for an object ends?

- A. The data is cloned into a historical object.
- B. The data moves to Fail-safe.
- C. Time Travel on the historical data is dropped.
- D. The object containing the historical data is dropped.

<details><summary>Show Answer</summary>
Correct Answer: B. For permanent tables, once the Time Travel retention period expires, the historical data enters the 7-day Fail-safe period (transient/temporary objects have no Fail-safe).
</details>

---

### Question 4
A company's security audit requires generating a report listing all Snowflake logins (date and user) within the last 90 days. Which statement will return the required information?

- A. `SELECT LAST_SUCCESS_LOGIN, LOGIN_NAME FROM ACCOUNT_USAGE.USERS;`
- B. `SELECT EVENT_TIMESTAMP, USER_NAME FROM TABLE(INFORMATION_SCHEMA.LOGIN_HISTORY_BY_USER());`
- C. `SELECT EVENT_TIMESTAMP, USER_NAME FROM ACCOUNT_USAGE.ACCESS_HISTORY;`
- D. `SELECT EVENT_TIMESTAMP, USER_NAME FROM ACCOUNT_USAGE.LOGIN_HISTORY;`

<details><summary>Show Answer</summary>
Correct Answer: D. The `INFORMATION_SCHEMA` login functions (option B) only return activity for the last 7 days, which isn't enough for a 90-day window. `SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY` retains up to 365 days of login events.
</details>

---

### Question 5
What are common issues found by using the Query Profile? (Choose two.)

- A. Identifying queries that will likely run very slowly before executing them
- B. Locating queries that consume a high amount of credits
- C. Identifying logical issues with the queries
- D. Identifying inefficient micro-partition pruning
- E. Data spilling to local or remote disk

<details><summary>Show Answer</summary>
Correct Answer: D, E. Query Profile is a post-execution diagnostic tool used to spot problems like poor pruning and memory spillage — it can't predict slowness in advance (A) or flag business-logic errors (C).
</details>

---

### Question 6
The Snowflake Search Optimization Service supports improved performance of which kind of query?

- A. Queries against large tables where frequent full scans are unavoidable
- B. Queries against tables larger than 1 TB
- C. Selective point lookup queries
- D. Queries against a subset of columns in a table

<details><summary>Show Answer</summary>
Correct Answer: C. Search Optimization Service is designed to speed up highly selective equality/substring point-lookup queries on large tables — table size alone doesn't determine eligibility.
</details>

---

### Question 7
Which file formats are supported for **unloading** data from Snowflake? (Choose two.)

- A. AVRO
- B. JSON
- C. ORC
- D. XML
- E. Delimited (CSV, TSV, etc.)

<details><summary>Show Answer</summary>
Correct Answer: B, E. Snowflake can unload data as delimited (CSV/TSV) or JSON, and also Parquet. AVRO, ORC, and XML are supported for **loading** only, not unloading.
</details>

---

### Question 8
Which Snowflake tool would be BEST to troubleshoot network connectivity?

- A. SnowCLI
- B. SnowUI
- C. SnowSQL
- D. SnowCD

<details><summary>Show Answer</summary>
Correct Answer: D. SnowCD (Snowflake Connectivity Diagnostic Tool) is purpose-built to test and troubleshoot network connectivity to Snowflake.
</details>

---

### Question 9
Increasing the size of a virtual warehouse from an X-Small to an X-Large is an example of which of the following?

- A. Right-sizing
- B. Concurrency scaling
- C. Scaling out
- D. Scaling up

<details><summary>Show Answer</summary>
Correct Answer: D. Increasing the size of a single warehouse is "scaling up." Adding additional clusters to a multi-cluster warehouse is "scaling out."
</details>

---

### Question 10
What are ways to create and manage data shares in Snowflake? (Choose two.)

- A. Through the Snowflake web interface
- B. Through a session-level parameter
- C. Through SQL commands
- D. Through an account-level parameter
- E. Using a `CREATE SHARE AS SELECT FROM TABLE` command

<details><summary>Show Answer</summary>
Correct Answer: A, C. Shares can be created and managed via Snowsight (the web interface) or via SQL commands (`CREATE SHARE`, `GRANT ... TO SHARE`, etc.). There is no `CREATE SHARE AS SELECT` command.
</details>

---

### Question 11
What is a characteristic of data micro-partitioning in Snowflake?

- A. Micro-partitioning may introduce data skew.
- B. Micro-partitioning requires the definition of a partitioning schema.
- C. Micro-partitioning happens automatically when the data is loaded.
- D. Micro-partitioning can be disabled within a Snowflake account.

<details><summary>Show Answer</summary>
Correct Answer: C. Micro-partitioning is fully automatic — Snowflake determines micro-partition boundaries as data is loaded, with no user-defined partitioning scheme, and it cannot be turned off.
</details>

---

### Question 12
Users with the ACCOUNTADMIN role can do which of the following on existing users?

- A. Can SHOW users, DESCRIBE a given user, or ALTER or DROP a user
- B. Can DEFINE users, DESCRIBE a given user, or ALTER or DELETE a user
- C. Can SHOW users, INDEX a given user, or ALTER or DELETE a user
- D. Can SHOW users, DEFINE a given user, or ALTER, DROP, or MODIFY a user

<details><summary>Show Answer</summary>
Correct Answer: A. SHOW, DESCRIBE, ALTER, and DROP are all valid Snowflake user-management commands. "DEFINE," "INDEX," "DELETE," and "MODIFY" are not valid SQL keywords for managing users in Snowflake.
</details>

---

### Question 13
According to Snowflake best-practice recommendations, which system-defined roles should be used to create custom roles? (Choose two.)

- A. ACCOUNTADMIN
- B. SYSADMIN
- C. SECURITYADMIN
- D. USERADMIN
- E. ORGADMIN

<details><summary>Show Answer</summary>
Correct Answer: C, D. Best practice is to create and manage custom roles under SECURITYADMIN (or its child, USERADMIN), keeping role/user administration separate from object ownership (SYSADMIN) and full account control (ACCOUNTADMIN).
</details>

---

### Question 14
What services are provided by the cloud services layer in Snowflake? (Choose two.)

- A. Metadata management
- B. Object authorization
- C. Authentication
- D. Query execution
- E. Result caching

<details><summary>Show Answer</summary>
Correct Answer: A, C. The cloud services layer handles authentication, metadata management, infrastructure management, access control, and optimization. Query execution (D) happens in the compute layer; result caching (E) is stored/served via the cloud services layer's metadata but the option as commonly tested pairs with metadata + authentication as the core services.
</details>

---

### Question 15
Which of the following are valid options for the `VALIDATION_MODE` parameter within the Snowflake `COPY INTO` command? (Choose two.)

- A. TRUE
- B. RETURN_ERROR_SUM
- C. RETURN_ALL_ERRORS
- D. RETURN_[n]_ROWS
- E. RETURN_FIRST_N_ERRORS

<details><summary>Show Answer</summary>
Correct Answer: C, D. Valid `VALIDATION_MODE` values are `RETURN_n_ROWS`, `RETURN_ERRORS`, and `RETURN_ALL_ERRORS`. "RETURN_ERROR_SUM," "TRUE," and "RETURN_FIRST_N_ERRORS" are not valid values.
</details>

---

### Question 16
Snowflake virtual warehouses are part of which layer of the Snowflake architecture?

- A. Compute
- B. Storage layer
- C. Database layer
- D. Cloud services layer

<details><summary>Show Answer</summary>
Correct Answer: A. Virtual warehouses are the compute layer — independently scalable clusters used to execute queries and DML.
</details>

---

### Question 17
Which of the following are characteristics of schemas used in Snowflake? (Choose two.)

- A. A schema may contain one or more databases.
- B. A database may contain one or more schemas.
- C. A schema represents a logical grouping of database objects.
- D. Each schema is contained within a virtual warehouse.
- E. A table can span more than one schema.

<details><summary>Show Answer</summary>
Correct Answer: B, C. The hierarchy runs database → schema → objects (not the reverse), a schema is purely a logical container, and it has no relationship to a virtual warehouse (compute is separate from the object hierarchy). A table always belongs to exactly one schema.
</details>

---

### Question 18
Which objects can be used to reduce data storage costs via short-lived tables? (Choose two.)

- A. Provisional tables
- B. Temporary tables
- C. Transient tables
- D. Permanent tables
- E. Lookup tables

<details><summary>Show Answer</summary>
Correct Answer: B, C. Temporary and transient tables both skip (or minimize) Fail-safe storage costs, making them cheaper for short-lived data. "Provisional" and "lookup" are not real Snowflake table types.
</details>

---

### Question 19
A user has unloaded data from Snowflake to a stage. Which SQL command should be used to check which files are present in the stage?

- A. `LIST @file_stage`
- B. `SHOW @file_stage`
- C. `VIEW @file_stage`
- D. `VERIFY @file_stage`

<details><summary>Show Answer</summary>
Correct Answer: A. `LIST` (or its alias `ls`) displays the files currently staged at an internal or external stage location.
</details>

---

### Question 20
What are benefits of using the `ACCESS_HISTORY` view in the SNOWFLAKE database? (Choose two.)

- A. Identification of unused data
- B. Identification of which roles have been dropped
- C. Tracking of network policy usage
- D. Highlighting of row access policy usage
- E. Identification of who has read data

<details><summary>Show Answer</summary>
Correct Answer: A, E. `ACCESS_HISTORY` logs read/write access to columns and objects, which lets you identify columns/tables nobody queries (unused data) and who has accessed particular data (for compliance and auditing).
</details>

---

### Question 21
Which of the following view types are available in Snowflake? (Choose two.)

- A. Layered view
- B. Secure view
- C. External view
- D. Embedded view
- E. Materialized view

<details><summary>Show Answer</summary>
Correct Answer: B, E. Snowflake supports standard views, secure views, and materialized views. "Layered," "external," and "embedded" are not Snowflake view types (note: this corrects the source material's garbled "BF," since there is no option F).
</details>

---

### Question 22
Which of the following statements describe a benefit of Snowflake's separation of compute and storage? (Choose two.)

- A. Growth of storage and compute are tightly coupled.
- B. Storage expands without the requirement to add more compute.
- C. Compute can be scaled up or down without the requirement to add more storage.
- D. Compute and storage can only be scaled together.
- E. Use of storage avoids disk spilling.

<details><summary>Show Answer</summary>
Correct Answer: B, C. Because compute (virtual warehouses) and storage are architecturally decoupled, each can scale independently of the other.
</details>

---

### Question 23
Which of the following languages can be used to implement Snowflake User-Defined Functions (UDFs)? (Choose two.)

- A. Ruby
- B. JavaScript
- C. SQL
- D. PERL

<details><summary>Show Answer</summary>
Correct Answer: B, C. Snowflake UDFs can be written in SQL, JavaScript, and (via Snowpark) Python, Java, and Scala. Ruby and Perl are not supported UDF languages.

**⚠ Updated:** In addition to SQL and JavaScript, Snowflake now also supports Python, Java, and Scala UDFs via Snowpark — worth knowing even though only two options were valid among this question's choices.
</details>

---

### Question 24
What is the default compression type when unloading data from Snowflake?

- A. Brotli
- B. bzip2
- C. Zstandard
- D. gzip

<details><summary>Show Answer</summary>
Correct Answer: D. gzip is the default `COMPRESSION` value for unload file formats in Snowflake.
</details>

---

### Question 25
Which statement describes when a virtual warehouse can be resized?

- A. A resize will affect running, queued, and future queries.
- B. A resize can only be completed when the warehouse is in an auto-resume state.
- C. A resize must be completed when the warehouse is suspended.
- D. A resize can be completed at any time.

<details><summary>Show Answer</summary>
Correct Answer: D. Warehouses can be resized at any time — while running, suspended, or processing statements — with no requirement to suspend first.
</details>

---

### Question 26
What is the compressed size limit for semi-structured data loaded into a VARIANT data type using the `COPY` command?

- A. 8 MB
- B. 16 MB
- C. 32 MB
- D. 64 MB

<details><summary>Show Answer</summary>
Correct Answer: B. The classic and still-enforced limit for a single semi-structured object parsed into a VARIANT via `COPY INTO` is 16 MB.

**⚠ Updated:** Snowflake has since introduced larger default column-size limits for *storing* VARIANT/ARRAY/OBJECT data (up to 128 MB per value in tables that adopt the new size limits). However, the 16 MB ceiling still applies to parsing an individual semi-structured record during a load — values larger than that will still fail with a "max LOB size exceeded" error unless you restructure or flatten the data first.
</details>

---

### Question 27
User A cloned a schema and overwrote a schema that User B was working on; User B no longer has access to their version of the tables. This occurred within the Time Travel retention period defined at the database level. How should the missing tables be restored?

- A. Use an `UNDROP TABLE` statement.
- B. Use a `CREATE TABLE AS SELECT` statement.
- C. Rename the cloned schema and use an `UNDROP SCHEMA` statement.
- D. Contact Snowflake Support to retrieve the data from Fail-safe.

<details><summary>Show Answer</summary>
Correct Answer: C. Since the original schema was overwritten (not individually dropped), the whole schema must be recovered with `UNDROP SCHEMA`; the currently-in-place (cloned) schema needs to be renamed first to free up the name.
</details>

---

### Question 28
How does Snowflake recommend handling the bulk loading of data batches from files already available in cloud storage?

- A. Use Snowpipe.
- B. Use the `INSERT` command.
- C. Use an external table.
- D. Use the `COPY` command.

<details><summary>Show Answer</summary>
Correct Answer: D. For bulk/batch loading of existing files, the `COPY INTO` command is the recommended approach. Snowpipe is intended for continuous, event-driven micro-batch loading rather than one-time bulk loads.
</details>

---

### Question 29
What is Snowflake's general guideline for files used to load data?

- A. Files can be loaded directly into a table without staging.
- B. Any delimiter is supported; the default is a semicolon.
- C. Electronic Data Interchange (EDI) is one of the supported formats.
- D. For delimited files, the default character set is UTF-8.

<details><summary>Show Answer</summary>
Correct Answer: D. UTF-8 is the default encoding for delimited (CSV) files; other encodings must be specified explicitly via the `ENCODING` file format option.
</details>

---

### Question 30
How does a Snowflake user execute an anonymous block of code?

- A. The user must run a `CALL` command to execute the block.
- B. The statements that define the block also execute it.
- C. A `SUBMIT` command must run immediately after the block is defined.
- D. The block must be saved to a worksheet and executed using a connector.

<details><summary>Show Answer</summary>
Correct Answer: B. An anonymous block (`EXECUTE IMMEDIATE` / `BEGIN...END`) runs as soon as it's submitted — defining it is the same action as executing it, unlike a stored procedure which must be separately called.
</details>

---

### Question 31
When unloading data from Snowflake, the user executes a `COPY INTO <location>` command into an internal stage. What additional command is required to load the file onto the local file system?

- A. `GET`
- B. `LIST`
- C. `PUT`
- D. `REMOVE`

<details><summary>Show Answer</summary>
Correct Answer: A. `GET` downloads files from an internal stage to the local file system. `PUT` does the reverse (local → stage).
</details>

---

### Question 32
A Snowflake user has a query that has been running for a long time. The Query Profile indicates that a lot of data is spilling. What is causing this to happen?

- A. The result cache is almost full and is unable to hold the results.
- B. The cloud storage staging area is not sufficient to hold the data.
- C. Clustering has not been applied to the table, so it is not optimized.
- D. The warehouse's memory is not sufficient to hold the intermediate query results.

<details><summary>Show Answer</summary>
Correct Answer: D. "Spilling" occurs when a warehouse doesn't have enough memory for intermediate results and has to spill them to local (or, worse, remote) disk — the fix is typically a larger warehouse or query optimization.
</details>

---

### Question 33
What is historically the MOST performant file format for loading data in Snowflake?

- A. CSV (uncompressed)
- B. Parquet
- C. CSV (gzipped)
- D. ORC

<details><summary>Show Answer</summary>
Correct Answer: C (as originally tested). Compressed (gzipped) CSV has traditionally outperformed columnar formats for raw load throughput in Snowflake benchmarks.

**⚠ Updated:** Snowflake's own engineering team has since released a vectorized Parquet scanner that significantly narrows — and in current benchmarks reverses — this gap, making **Parquet** the most efficient load format when the vectorized scanner is used. As of mid-2026, Snowflake's official guidance is to use whichever format your source data is already in rather than converting, but Parquet (with the vectorized scanner) is now generally the fastest to ingest. The SnowPro Core exam content may still test the historical "gzipped CSV" answer, so know both.
</details>

---

### Question 34
Which chart type does Snowsight support to visualize worksheet data?

- A. Box plot
- B. Bubble chart
- C. Pie chart
- D. Scatter plot

<details><summary>Show Answer</summary>
Correct Answer: D. Snowsight natively supports bar charts, line charts, scatter plots, heat grids, and scorecards. Pie charts, bubble charts, and box plots are not native Snowsight chart types (confirmed current as of July 2026).
</details>

---

### Question 35
Which result shows efficient pruning?

- A. Partitions scanned is greater than partitions total.
- B. Partitions scanned is less than partitions total.
- C. Partitions scanned is equal to partitions total.
- D. Partitions scanned is greater than or equal to partitions total.

<details><summary>Show Answer</summary>
Correct Answer: B. Efficient pruning means Snowflake skipped irrelevant micro-partitions, so partitions scanned should be well below the total partition count.
</details>

---

### Question 36
Which clustering indicator will show if a large table in Snowflake will benefit from explicitly defining a clustering key?

- A. Percentage
- B. Depth
- C. Ratio
- D. Total partition count

<details><summary>Show Answer</summary>
Correct Answer: B. Clustering **depth** (from `SYSTEM$CLUSTERING_INFORMATION` / `SYSTEM$CLUSTERING_DEPTH`) indicates overlap among micro-partitions — a high average depth suggests the table would benefit from a clustering key.
</details>

---

### Question 37
Which file format is MOST performant in Snowflake for data loading?

- A. Parquet
- B. CSV
- C. ORC

<details><summary>Show Answer</summary>
Correct Answer: B (as originally tested).

**⚠ Updated:** See Question 33 — Snowflake's current vectorized Parquet scanner has made Parquet competitive with, and in many benchmarks faster than, gzipped CSV. Treat this as a topic where "the documented/tested answer" and "the current real-world answer" have diverged; both are worth knowing for the exam versus for practice.
</details>

---

### Question 38
What is to be expected when sharing worksheets in Snowsight?

- A. Worksheets can be shared with users internal or external to any organization.
- B. To run a shared worksheet, a user must be granted the role used in the worksheet's session context.
- C. Snowsight allows users to view and refresh results but not to edit shared worksheets.
- D. Snowsight offers different sharing permissions at the worksheet, folder, and dashboard level.

<details><summary>Show Answer</summary>
Correct Answer: B. A worksheet's session context (the role and warehouse it was written to use) must be available to the person running it — they need to be granted that role to successfully execute the shared worksheet.
</details>

---

### Question 39
Which Snowflake objects track DML changes made to tables, like inserts, updates, and deletes?

- A. Pipes
- B. Streams
- C. Tasks
- D. Procedures

<details><summary>Show Answer</summary>
Correct Answer: B. Streams record change data capture (CDC) information — inserts, updates, and deletes — made to a source table since the stream was last consumed.
</details>

---

### Question 40
Which table type is automatically deleted after a session is closed and has no Fail-safe or Time Travel cost?

- A. Transient
- B. Temporary
- C. Permanent
- D. External

<details><summary>Show Answer</summary>
Correct Answer: B. Temporary tables persist only for the session that created them and are then automatically dropped; they have zero Fail-safe period and a maximum 1-day Time Travel retention.
</details>

---

### Question 41
Which constraint type is enforced in Snowflake, consistent with the ANSI SQL standard?

- A. UNIQUE
- B. PRIMARY KEY
- C. FOREIGN KEY
- D. NOT NULL

<details><summary>Show Answer</summary>
Correct Answer: D. Snowflake supports constraint metadata for UNIQUE, PRIMARY KEY, and FOREIGN KEY, but does not enforce them (they're informational only for query optimization). NOT NULL is the only constraint type actually enforced.
</details>

---

### Question 42
Which function/view is used to profile warehouse credit usage?

- A. `QUERY_HISTORY`
- B. `WAREHOUSE_LOAD_HISTORY`
- C. `METERING_DAILY_HISTORY`
- D. `WAREHOUSE_METERING_HISTORY`

<details><summary>Show Answer</summary>
Correct Answer: D. `WAREHOUSE_METERING_HISTORY` (available both as an `INFORMATION_SCHEMA` table function and an `ACCOUNT_USAGE` view) returns hourly credit consumption per warehouse.
</details>

---

### Question 43
What is a characteristic of the Snowflake query profiler?

- A. It can provide statistics on a maximum number of queries per week.
- B. It provides a graphic representation of the main components of query processing.
- C. It provides detailed statistics about which queries are using the greatest number of compute resources across the account.
- D. It can be used by third-party software through a dedicated query-profiler API.

<details><summary>Show Answer</summary>
Correct Answer: B. Query Profile visually breaks down the execution plan and processing steps for a single query — it's not an account-wide resource-ranking tool, and there's no public "Query Profiler API."
</details>

---

### Question 44
A Snowflake user wants to share transactional data with retail suppliers, but some suppliers do not use Snowflake. According to best practice, what should the user do? (Choose two.)

- A. Provide each non-Snowflake supplier with their own reader account.
- B. Deploy a single reader account to be shared by all of the non-Snowflake suppliers.
- C. Create an ETL pipeline using SELECT and INSERT statements from the source to each target supplier account.
- D. Use a data share for suppliers in the same cloud region, and a replicated proxy share for other cloud/region deployments.
- E. Unload the shared transactional data to an external stage and use cloud storage utilities to reload it into the suppliers' accounts.

<details><summary>Show Answer</summary>
Correct Answer: A, D. Reader accounts are the recommended way to share data with non-Snowflake customers (one reader account per consumer, not shared), and replication is needed to extend a share across regions/cloud platforms.
</details>

---

### Question 45
Which statement about data sharing is true?

- A. Accounts can share with other accounts regardless of Snowflake edition, without requiring help from Snowflake Support.
- B. Data sharing can cross regions but not cloud providers.
- C. The data consumer can only see objects in the data provider's source database that have been explicitly added to the share.
- D. A data provider can only share with other Snowflake customers.

<details><summary>Show Answer</summary>
Correct Answer: C. Consumers only ever see the specific objects a provider grants to a share — nothing else in the provider's account is exposed.
</details>

---

### Question 46
Which command is used to load files into an internal stage within Snowflake?

- A. `PUT`
- B. `COPY INTO`
- C. `TRANSFER`
- D. `INSERT`

<details><summary>Show Answer</summary>
Correct Answer: A. `PUT` uploads local files to an internal stage. `COPY INTO <table>` then loads staged files into a table.
</details>

---

### Question 47
Which object type is granted permissions for reading a table?

- A. User
- B. Role
- C. Attribute
- D. Schema

<details><summary>Show Answer</summary>
Correct Answer: B. Snowflake uses role-based access control (RBAC) — privileges are granted to roles, and roles are granted to users, never the reverse.
</details>

---

### Question 48
What is the default value in the Snowflake web interface for auto-suspending a virtual warehouse?

- A. 1 minute
- B. 5 minutes
- C. 10 minutes
- D. 15 minutes

<details><summary>Show Answer</summary>
Correct Answer: C. Both the Classic Console and Snowsight default new warehouses to a 10-minute auto-suspend setting (confirmed current as of July 2026).
</details>

---

### Question 49
Several users are using the same virtual warehouse. Queries are running slowly and many are being queued. What is the recommended way to resolve this?

- A. Reduce the warehouse's `STATEMENT_TIMEOUT_IN_SECONDS` parameter.
- B. Reduce the warehouse's `AUTO_SUSPEND` parameter.
- C. Increase the warehouse's size.
- D. Increase the warehouse's `MAX_CLUSTER_COUNT` parameter.

<details><summary>Show Answer</summary>
Correct Answer: D. Queuing caused by concurrent-query volume (rather than individual query complexity) is best solved with multi-cluster warehouses — raising `MAX_CLUSTER_COUNT` lets Snowflake spin up additional clusters to absorb concurrency, rather than resizing (which helps individual query speed, not queuing).
</details>

---

### Question 50
Which data types are valid in Snowflake? (Choose two.)

- A. BLOB
- B. GEOGRAPHY
- C. LOB
- D. CLOB
- E. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: B, E. GEOGRAPHY and VARIANT are native Snowflake data types. BLOB, LOB, and CLOB are not — Snowflake instead uses BINARY (for binary data) and VARIANT/OBJECT/ARRAY (for semi-structured data).
</details>

---

### Question 51
What happens when the size of a virtual warehouse is changed while queries are running?

- A. Queries that are running on the current warehouse configuration are not impacted.
- B. Queries that are running are aborted and have to be resubmitted by the user.
- C. Queries that are running are aborted and are automatically resubmitted.
- D. Queries that are running are moved to the new configuration and finish there.

<details><summary>Show Answer</summary>
Correct Answer: A. Per Snowflake documentation, resizing a warehouse doesn't affect statements already executing — they continue to completion on their originally-provisioned resources. Only queued and new statements benefit from the resized compute. (This corrects the source material, which left the answer blank/ambiguous.)
</details>

---

### Question 52
How often are encryption keys automatically rotated by Snowflake?

- A. 30 Days
- B. 60 Days
- C. 90 Days
- D. 365 Days

<details><summary>Show Answer</summary>
Correct Answer: A. Snowflake-managed keys are automatically rotated once they are more than 30 days old; data can additionally be re-keyed (fully re-encrypted) on a yearly basis if periodic rekeying is enabled. (The source material's answer was illegible/cut off — confirmed via current documentation.)
</details>

---

### Question 53
As a best practice, all custom roles should ultimately be granted to which system-defined role?

- A. ACCOUNTADMIN
- B. ORGADMIN
- C. SECURITYADMIN
- D. SYSADMIN

<details><summary>Show Answer</summary>
Correct Answer: D. Custom roles should be granted up through the hierarchy to SYSADMIN, so system administrators retain visibility and control over all custom-role-owned objects, while ACCOUNTADMIN stays reserved for top-level account administration.
</details>

---

### Question 54
Which Snowflake object can be accessed in the `FROM` clause of a query, returning a set of rows with one or more columns?

- A. A User-Defined Table Function (UDTF)
- B. A scalar User-Defined Function (UDF)
- C. A stored procedure
- D. A task

<details><summary>Show Answer</summary>
Correct Answer: A. A UDTF returns a table (set of rows/columns) and can be referenced directly in a `FROM` clause; a scalar UDF returns only a single value per row.
</details>

---

### Question 55
How are micro-partitions typically generated in Snowflake?

- A. Automatically
- B. Via an `ORDER BY` clause
- C. Via a `PARTITION BY` clause
- D. Via a `GROUP BY` clause

<details><summary>Show Answer</summary>
Correct Answer: A. Snowflake automatically creates micro-partitions as data is loaded — there is no manual partitioning syntax required (or available) at load time.
</details>

---

### Question 56
What does Snowflake recommend regarding database object ownership? (Choose two.)

- A. Create objects with ACCOUNTADMIN and do not reassign ownership.
- B. Create objects with SYSADMIN.
- C. Create objects with SECURITYADMIN to ease granting of privileges later.
- D. Create objects with a custom role and grant that role to SYSADMIN.
- E. Use only managed access schemas for objects owned by ACCOUNTADMIN.

<details><summary>Show Answer</summary>
Correct Answer: B, D. Best practice is to avoid creating objects with ACCOUNTADMIN; instead, use SYSADMIN directly, or create objects with a custom role that is itself granted to SYSADMIN, keeping object ownership separate from account-level administration.
</details>

---

### Question 57
Other than ownership, what privileges does a role need to view and modify resource monitors in Snowflake?

- A. ALTER
- B. MONITOR
- C. MODIFY
- D. CREATE
- E. DROP

<details><summary>Show Answer</summary>
Correct Answer: B, C. The MONITOR privilege allows viewing a resource monitor's credit usage and settings; the MODIFY privilege allows changing them (adding/removing warehouses, adjusting thresholds, etc.).
</details>

---

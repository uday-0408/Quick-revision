# SnowPro Core Practice Questions (401–500)

*Formatted for self-study. Answers are hidden in collapsible blocks — click "Show Answer" to reveal. Answers were cross-checked against current Snowflake documentation (as of July 2026); any corrections are flagged with ⚠ Updated.*

---

### Question 401
Network policies can be applied to which of the following objects? (Choose two.)
- A. Roles
- B. Databases
- C. Warehouses
- D. Users
- E. Accounts

<details><summary>Show Answer</summary>
Correct Answer: D, E. Network policies restrict IP-based access and can be applied at the account level or to individual users.
</details>

---

### Question 402
Where is Snowflake metadata stored?
- A. Within the data files
- B. In the virtual warehouse layer
- C. In the cloud services layer
- D. In the remote storage

<details><summary>Show Answer</summary>
Correct Answer: C. The cloud services layer manages metadata, query optimization, security, and coordination across the platform.
</details>

---

### Question 403
What columns are returned when performing a FLATTEN command on semi-structured data? (Choose two.)
- A. KEY
- B. NODE
- C. VALUE
- D. LEVEL
- E. ROOT

<details><summary>Show Answer</summary>
Correct Answer: A, C. FLATTEN returns SEQ, KEY, PATH, INDEX, VALUE, THIS as columns; KEY and VALUE are among the standard output columns.
</details>

---

### Question 404
Which of the following Snowflake features provide continuous data protection? (Choose two.)
- A. Internal stages
- B. Backups
- C. Time Travel
- D. Zero-copy clones
- E. Fail-safe

<details><summary>Show Answer</summary>
Correct Answer: C, E. Time Travel and Fail-safe together form Snowflake's Continuous Data Protection (CDP) lifecycle.
</details>

---

### Question 405
A developer is granted ownership of a table that has a masking policy applied. The developer's role is not able to see the masked data. Will the developer be able to modify the table to read the masked data?
- A. Yes, because a table owner has control and can unset masking policies.
- B. Yes, because masking policies only apply to cloned tables.
- C. No, because masking policies must always reference specific access roles.
- D. No, because ownership of a table does not include the ability to change masking policies.

<details><summary>Show Answer</summary>
Correct Answer: D. Managing masking policies requires a separate privilege (e.g., APPLY MASKING POLICY); table ownership alone does not grant this.
</details>

---

### Question 406
How should a virtual warehouse be configured if a user wants to ensure that additional multi-clusters are resumed with no delay?
- A. Set the warehouse to a size larger than generally needed
- B. Set the minimum and maximum clusters to autoscale
- C. Use the standard warehouse scaling policy
- D. Use the economy warehouse scaling policy

<details><summary>Show Answer</summary>
Correct Answer: C. The Standard scaling policy favors starting additional clusters immediately to minimize queuing, unlike Economy, which waits to conserve credits.
</details>

---

### Question 407
During periods of warehouse contention, which parameter controls the maximum length of time a warehouse will hold a query for processing?
- A. STATEMENT_TIMEOUT_IN_SECONDS
- B. STATEMENT_QUEUED_TIMEOUT_IN_SECONDS
- C. MAX_CONCURRENCY_LEVEL
- D. MAX_STATEMENT_TIME

<details><summary>Show Answer</summary>
Correct Answer: B. STATEMENT_QUEUED_TIMEOUT_IN_SECONDS controls how long a query can sit in the queue before it is canceled.
</details>

---

### Question 408
Files have been uploaded to a Snowflake internal stage. The files now need to be deleted. Which SQL command should be used to delete the files?
- A. PURGE
- B. MODIFY
- C. REMOVE
- D. DELETE

<details><summary>Show Answer</summary>
Correct Answer: C. REMOVE deletes files from an internal or external stage.
</details>

---

### Question 409
In a Snowflake role hierarchy, what is the top-level role?
- A. SYSADMIN
- B. ORGADMIN
- C. ACCOUNTADMIN
- D. SECURITYADMIN

<details><summary>Show Answer</summary>
Correct Answer: C. ACCOUNTADMIN sits at the top of the default account-level role hierarchy (ORGADMIN operates at the organization level, above individual accounts).
</details>

---

### Question 410
By default, which Snowflake role is required to create a share?
- A. ORGADMIN
- B. SECURITYADMIN
- C. SHAREADMIN
- D. ACCOUNTADMIN

<details><summary>Show Answer</summary>
Correct Answer: D. Only ACCOUNTADMIN (or a role explicitly granted the CREATE SHARE privilege) can create outbound shares by default.
</details>

---

### Question 411
What happens to historical data when the retention period for an object ends?
- A. The data is cloned into a historical object.
- B. The data moves to Fail-safe.
- C. Time Travel on the historical data is dropped.
- D. The object containing the historical data is dropped.

<details><summary>Show Answer</summary>
Correct Answer: B. Once the Time Travel retention period expires, historical data enters the 7-day Fail-safe period (for permanent tables).
</details>

---

### Question 412
A company's security audit requires generating a report listing all Snowflake logins (e.g., date and user) within the last 90 days. Which of the following statements will return the required information?
- A. SELECT LOGIN_NAME FROM ACCOUNT_USAGE.USERS;
- B. SELECT EVENT_TIMESTAMP, USER_NAME FROM table(information_schema.login_history());
- C. SELECT EVENT_TIMESTAMP, USER_NAME FROM ACCOUNT_USAGE.QUERY_HISTORY;
- D. SELECT EVENT_TIMESTAMP, USER_NAME FROM ACCOUNT_USAGE.LOGIN_HISTORY;

<details><summary>Show Answer</summary>
Correct Answer: D. ACCOUNT_USAGE.LOGIN_HISTORY retains data for 365 days, covering the required 90-day window; INFORMATION_SCHEMA.LOGIN_HISTORY() only covers the last 7 days.
</details>

---

### Question 413
What are common issues found by using the Query Profile? (Choose two.)
- A. Identifying queries that will likely run very slowly before executing them
- B. Locating queries that consume a high amount of credits
- C. Identifying logical issues with the queries
- D. Identifying inefficient micro-partition pruning
- E. Spilling to local or remote disk

<details><summary>Show Answer</summary>
Correct Answer: D, E. Query Profile is used post-execution to diagnose issues like poor pruning and memory spillage to disk.
</details>

---

### Question 414
The Snowflake Search Optimization Service supports improved performance of which kind of queries?
- A. Queries against large tables where frequent updates occur
- B. Queries against tables larger than 1 TB
- C. Selective point lookup queries
- D. Queries against a subset of columns in a table

<details><summary>Show Answer</summary>
Correct Answer: C. Search Optimization Service is designed to speed up highly selective point lookup queries on large tables.
</details>

---

### Question 415
Which file formats are supported for unloading data from Snowflake? (Choose two.)
- A. AVRO
- B. JSON
- C. ORC
- D. XML
- E. Delimited (CSV, TSV, etc.)

<details><summary>Show Answer</summary>
Correct Answer: B, E. Snowflake supports unloading to delimited (CSV/TSV) and JSON formats (Parquet is also supported for unload; AVRO, ORC, and XML are load-only formats).
</details>

---

### Question 416
Which Snowflake tool would be BEST to troubleshoot network connectivity?
- A. snowCLI
- B. SnowUI
- C. snowsql
- D. snowcd

<details><summary>Show Answer</summary>
Correct Answer: D. SnowCD (Snowflake Connectivity Diagnostic Tool) tests and troubleshoots network connectivity to Snowflake.
</details>

---

### Question 417
Increasing the size of a virtual warehouse from an X-Small to an X-Large is an example of which of the following?
- A. Right sizing
- B. Concurrent scaling
- C. Scaling out
- D. Scaling up

<details><summary>Show Answer</summary>
Correct Answer: D. Increasing a single warehouse's size is "scaling up"; adding more clusters (multi-cluster) is "scaling out."
</details>

---

### Question 418
What are ways to create and manage data shares in Snowflake? (Choose two.)
- A. Through the Snowflake web interface
- B. Through the Data Marketplace parameter
- C. Through SQL commands
- D. Through the Reader Account interface
- E. Using the CREATE SHARE AS SELECT FROM TABLE command

<details><summary>Show Answer</summary>
Correct Answer: A, C. Shares can be managed via Snowsight or via SQL (CREATE SHARE, GRANT ... TO SHARE, ALTER SHARE, etc.).
</details>

---

### Question 419
What is a characteristic of data micro-partitioning in Snowflake?
- A. Micro-partitioning may introduce data skew.
- B. Micro-partitioning requires the definition of a partitioning schema.
- C. Micro-partitioning happens automatically when the data is loaded.
- D. Micro-partitioning can be disabled within a Snowflake account.

<details><summary>Show Answer</summary>
Correct Answer: C. Micro-partitioning is automatic and transparent — Snowflake handles it without any manual partitioning scheme.
</details>

---

### Question 420
Users with the ACCOUNTADMIN role can perform which of the following commands on existing users?
- A. Can SHOW users, DESCRIBE a given user, or ALTER or DROP a user
- B. Can DEFINE users, DESCRIBE a given user, or ALTER or DELETE a user
- C. Can SHOW users, INDEX a given user, or ALTER or DELETE a user
- D. Can SHOW users, DEFINE a given user or ALTER, DROP, or MODIFY a user

<details><summary>Show Answer</summary>
Correct Answer: A. The valid DDL/DCL verbs for user objects are SHOW, DESCRIBE, ALTER, and DROP.
</details>

---

### Question 421
According to Snowflake best practice recommendations, which system-defined roles should be used to create custom roles? (Choose two.)
- A. ACCOUNTADMIN
- B. SYSADMIN
- C. SECURITYADMIN
- D. USERADMIN
- E. ORGADMIN

<details><summary>Show Answer</summary>
Correct Answer: C, D. SECURITYADMIN (or its child USERADMIN) should be used to create and manage roles, keeping role administration separate from system/object administration.
</details>

---

### Question 422
What services are provided by the cloud services layer in Snowflake? (Choose two.)
- A. Metadata management
- B. Object authorization
- C. Authentication
- D. Query execution
- E. Result caching

<details><summary>Show Answer</summary>
Correct Answer: A, C. The cloud services layer handles authentication, metadata management, infrastructure management, access control, and query parsing/optimization. (Query execution happens in the compute layer; result caching is a byproduct stored via the cloud services layer, but the clearest fits here are metadata and authentication.)
</details>

---

### Question 423
Which of the following commands are valid options for the VALIDATION_MODE parameter within the Snowflake COPY INTO command? (Choose two.)
- A. RETURN_ROWS
- B. TRUE
- C. RETURN_ERRORS
- D. FALSE

<details><summary>Show Answer</summary>
Correct Answer: A, C. Valid VALIDATION_MODE values include RETURN_n_ROWS, RETURN_ERRORS, and RETURN_ALL_ERRORS.
</details>

---

### Question 424
Snowflake virtual warehouses are part of which layer of the Snowflake architecture?
- A. Compute layer
- B. Storage layer
- C. Database layer
- D. Cloud services layer

<details><summary>Show Answer</summary>
Correct Answer: A. Virtual warehouses make up the compute (query processing) layer.
</details>

---

### Question 425
Which of the following are characteristics of schemas used in Snowflake? (Choose two.)
- A. A schema may contain one or more databases.
- B. A database may contain one or more schemas.
- C. A schema represents a logical grouping of database objects.
- D. Each schema is contained within a virtual warehouse.
- E. A table can span more than one schema.

<details><summary>Show Answer</summary>
Correct Answer: B, C. Databases contain one or more schemas, and each schema logically groups objects like tables, views, and procedures.
</details>

---

### Question 426
Which objects can be used to reduce data storage costs for short-lived tables? (Choose two.)
- A. Provisional tables
- B. Temporary tables
- C. Transient tables
- D. Permanent tables
- E. Lookup tables

<details><summary>Show Answer</summary>
Correct Answer: B, C. Temporary and transient tables skip Fail-safe (and in the case of temporary tables, are session-scoped), reducing storage costs versus permanent tables.
</details>

---

### Question 427
A user has unloaded data from Snowflake to a stage. Which SQL command should be used to validate which data was loaded into the stage?
- A. LIST @file_stage
- B. SHOW
- C. VIEW
- D. VERIFY

<details><summary>Show Answer</summary>
Correct Answer: A. LIST (or the `ls` shortcut) displays the files present in a stage.
</details>

---

### Question 428
What are benefits of using the ACCESS_HISTORY view in the SNOWFLAKE database? (Choose two.)
- A. Identification of unused data
- B. Identification of which roles have been used
- C. Tracking of network policy usage
- D. Highlighting of row access policy usage
- E. Identification of who has read data

<details><summary>Show Answer</summary>
Correct Answer: A, E. ACCESS_HISTORY tracks read/write activity on objects and columns, helping identify unused data and who accessed specific data.
</details>

---

### Question 429
Which of the following view types are available in Snowflake? (Choose two.)
- A. Layered view
- B. Secure view
- C. External view
- D. Embedded view
- E. Materialized view

<details><summary>Show Answer</summary>
Correct Answer: B, E. Snowflake supports standard (non-materialized), secure, and materialized views.
</details>

---

### Question 430
Which of the following statements describes a benefit of Snowflake's separation of compute and storage? (Choose two.)
- A. Growth of storage and compute are tightly coupled.
- B. Storage expands without the requirement to add more compute.
- C. Compute can be scaled up or down without the requirement to add more storage.
- D. Compute and storage can be scaled together.
- E. Use of storage avoids disk spilling.

<details><summary>Show Answer</summary>
Correct Answer: B, C. Snowflake's architecture decouples compute and storage, so each can scale independently.
</details>

---

### Question 431
Which of the following languages can be used to implement Snowflake User-Defined Functions (UDFs)? (Choose two.)
- A. Ruby
- B. JavaScript
- C. SQL
- D. PERL

<details><summary>Show Answer</summary>
Correct Answer: B, C. Snowflake UDFs support SQL, JavaScript, Java, Python, and Scala (Ruby and PERL are not supported).
</details>

---

### Question 432
What is the default compression type when unloading data from Snowflake?
- A. Brotli
- B. bzip2
- C. Zstandard
- D. gzip

<details><summary>Show Answer</summary>
Correct Answer: D. gzip is the default compression for unloaded files.
</details>

---

### Question 433
Which statement describes when a virtual warehouse can be resized?
- A. A resize will affect running, queued, and past queries.
- B. A resize can only be completed when the warehouse is in an auto-resume status.
- C. A resize must be completed when the warehouse is suspended.
- D. A resize can be completed at any time.

<details><summary>Show Answer</summary>
Correct Answer: D. A warehouse can be resized at any time, including while running; currently executing statements are unaffected, and new resources apply to queued and future statements.
</details>

---

### Question 434
What is the compressed size limit for semi-structured data loaded into a VARIANT data type using the COPY command?
- A. 8 MB
- B. 16 MB
- C. 32 MB
- D. 64 MB

<details><summary>Show Answer</summary>
Correct Answer: B. The maximum compressed size for a VARIANT value is 16 MB.
</details>

---

### Question 435
User A cloned a schema and overwrote a schema that User B was working on. User B no longer has access to their version of the tables. However, this all occurred within the Time Travel retention period defined at the database level. How should the missing tables be restored?
- A. Use an UNDROP TABLE statement
- B. Use a CREATE TABLE AS SELECT statement
- C. Rename the cloned schema and use an UNDROP SCHEMA statement.
- D. Contact Snowflake Support to retrieve the data from Fail-safe.

<details><summary>Show Answer</summary>
Correct Answer: C. Since the original schema was overwritten (not individually dropped tables), the new schema must be renamed out of the way, then UNDROP SCHEMA restores the original.
</details>

---

### Question 436
How does Snowflake recommend handling the bulk loading of data batches from files already available in cloud storage?
- A. Use Snowpipe
- B. Use the INSERT command
- C. Use an external table
- D. Use the COPY command

<details><summary>Show Answer</summary>
Correct Answer: D. COPY INTO is the recommended method for bulk-loading existing batches of files (Snowpipe is for continuous, event-driven loading).
</details>

---

### Question 437
What is Snowflake's general guideline for files used to load data?
- A. Files can be loaded directly into a table.
- B. Any delimiter is supported; the default is a semicolon.
- C. Electronic Data Interchange (EDI) is one of the supported formats.
- D. For delimited files, the default character set is UTF-8.

<details><summary>Show Answer</summary>
Correct Answer: D. UTF-8 is the default character set for delimited files (files must first be staged, not loaded directly; the default delimiter is a comma, not a semicolon; EDI is not a supported format).
</details>

---

### Question 438
How does a Snowflake user execute an anonymous block of code?
- A. The user must run the CALL command to execute the block.
- B. The statements that define the block must also execute the block.
- C. The SUBMIT command must run immediately after the block is defined.
- D. The block must be saved to a worksheet and executed using a connector.

<details><summary>Show Answer</summary>
Correct Answer: B. An anonymous block (BEGIN…END) is executed as soon as it is submitted — the defining statement is also the execution.
</details>

---

### Question 439
When unloading data from Snowflake, the user executes a COPY INTO [location] command into an internal stage. What additional command is required to load the file onto the local file system?
- A. GET
- B. LIST
- C. PUT
- D. REMOVE

<details><summary>Show Answer</summary>
Correct Answer: A. GET downloads files from an internal stage to the local file system (PUT does the reverse — uploading local files to a stage).
</details>

---

### Question 440
A Snowflake user has a query that is running for a long time. When viewing the query profiler, it indicates that a lot of data is spilling to disk. What is causing this to happen?
- A. The result cache is almost full and is unable to hold the results.
- B. The Cloud Storage staging area is not sufficient to hold the data.
- C. Clustering has not been applied to the table so the table is not optimized.
- D. The warehouse memory is not sufficient to hold the intermediate query results.

<details><summary>Show Answer</summary>
Correct Answer: D. Spilling occurs when a warehouse's available memory (and then local disk) is insufficient for intermediate results, indicating the warehouse may need to be resized larger.
</details>

---

### Question 441
What is the MOST efficient file format for loading data in Snowflake?
- A. CSV (Unzipped)
- B. Parquet
- C. CSV (Gzipped)
- D. ORC

<details><summary>Show Answer</summary>
Correct Answer: C. Compressed (gzipped) delimited files are generally the most efficient for the load process, since compression reduces transfer size while remaining splittable for parallel loading.
</details>

---

### Question 442
Which chart type does Snowsight support to visualize worksheet data?
- A. Box plot
- B. Bubble chart
- C. Pie chart
- D. Scatter plot

<details><summary>Show Answer</summary>
Correct Answer: D. Snowsight's chart builder supports scatter plots among its chart types (along with line, bar, and area charts).
</details>

---

### Question 443
Which result shows efficient pruning?
- A. Partitions scanned is greater than partitions total.
- B. Partitions scanned is less than partitions total.
- C. Partitions scanned is equal to the partitions total.
- D. Partitions scanned is greater than or equal to the partitions total.

<details><summary>Show Answer</summary>
Correct Answer: B. Efficient pruning means fewer partitions are scanned than exist in total, since irrelevant partitions are skipped.
</details>

---

### Question 444
Which clustering indicator will show if a large table in Snowflake will benefit from explicitly defining a clustering key?
- A. Percentage
- B. Depth
- C. Ratio
- D. Total partition count

<details><summary>Show Answer</summary>
Correct Answer: B. Average clustering depth is the key indicator — a higher depth suggests the table would benefit from a defined clustering key.
</details>

---

### Question 445
Which file format is MOST performant in Snowflake for data loading?
- A. Parquet
- B. CSV
- C. ORC
- D. JSON

<details><summary>Show Answer</summary>
Correct Answer: B. Delimited flat files (CSV) load fastest because Snowflake can split and parallelize them efficiently across compute resources; columnar formats like Parquet/ORC are better suited for query performance after loading.
</details>

---

### Question 446
What is to be expected when sharing worksheets in Snowsight?
- A. Worksheets can be shared with users that are internal or external to any organization.
- B. To run a shared worksheet a user must be granted the role used in the worksheet session context.
- C. Snowflake allows users to view and refresh results but not to edit shared worksheets.
- D. Snowsight offers different sharing permissions at the worksheet, folder, and dashboard level.

<details><summary>Show Answer</summary>
Correct Answer: B. A recipient needs the appropriate role granted to them in order to successfully run a worksheet shared with them (sharing is limited to users within the same account, and both view/edit access levels can be granted).
</details>

---

### Question 447
Which Snowflake objects track DML changes made to tables, like inserts, updates, and deletes?
- A. Pipes
- B. Streams
- C. Tasks
- D. Procedures

<details><summary>Show Answer</summary>
Correct Answer: B. Streams provide change data capture (CDC) by tracking DML changes on a source table.
</details>

---

### Question 448
Which table type is automatically deleted after a session is closed and has no Fail-safe or Time Travel cost?
- A. Temporary
- B. Transient
- C. Permanent
- D. External

<details><summary>Show Answer</summary>
Correct Answer: A. Temporary tables exist only for the session and have no Fail-safe period (Time Travel is limited to 0 or 1 day).
</details>

---

### Question 449
Which constraint type is enforced in Snowflake from the ANSI SQL standard?
- A. UNIQUE
- B. PRIMARY KEY
- C. FOREIGN KEY
- D. NOT NULL

<details><summary>Show Answer</summary>
Correct Answer: D. Snowflake supports UNIQUE, PRIMARY KEY, and FOREIGN KEY constraints, but they are informational only and not enforced. NOT NULL is the only constraint type that is actually enforced.
</details>

---

### Question 450
Which function or view is used to profile warehouse credit usage?
- A. WAREHOUSE_CREDIT_USAGE
- B. QUERY_HISTORY
- C. ACCOUNT_USAGE
- D. WAREHOUSE_METERING_HISTORY

<details><summary>Show Answer</summary>
Correct Answer: D. WAREHOUSE_METERING_HISTORY (in ACCOUNT_USAGE or via a table function) reports credit usage per warehouse over time.
</details>

---

### Question 451
What is a characteristic of the Snowflake query profiler?
- A. It can provide statistics on a maximum number of queries per week.
- B. It provides a graphic representation of the main components of the query processing.
- C. It provides detailed statistics about which queries are using the greatest number of compute resources.
- D. It can be used by third-party software using the query profiler API.

<details><summary>Show Answer</summary>
Correct Answer: B. Query Profile provides a visual, graphical breakdown of each query's execution plan and processing steps.
</details>

---

### Question 452
A Snowflake user wants to share transactional data with retail suppliers. However, some of the suppliers do not use Snowflake. According to best practice, what should the user do? (Choose two.)
- A. Provide each non-Snowflake supplier with their own reader account.
- B. Deploy a single account to be shared by all of the non-Snowflake suppliers.
- C. Create an ETL pipeline that uses select and inserts statements from the source to the target supplier accounts.
- D. Use a data share for suppliers in the same cloud region and a replicated proxy share for other cloud deployments.
- E. Unload the shared transactional data to an External Stage and use Cloud Storage utilities to reload the suppliers' systems.

<details><summary>Show Answer</summary>
Correct Answer: A, D. Non-Snowflake consumers should each get their own reader account, and shares should be used directly within the same region/cloud, replicating as needed for cross-region/cloud consumers.
</details>

---

### Question 453
Which statement about data sharing is true?
- A. Accounts can share with other accounts regardless of their Snowflake edition, without requiring help from Snowflake Support.
- B. Data sharing can cross regions, but not cloud providers.
- C. The Data Consumer can only see objects in the Data Provider's source database that have been explicitly added to the share.
- D. A Data Provider can only share with other Snowflake customers.

<details><summary>Show Answer</summary>
Correct Answer: C. Consumers can only access the specific objects a provider has explicitly granted into the share — nothing else in the source database is visible.
</details>

---

### Question 454
Which command is used to load files into an internal stage within Snowflake?
- A. PUT
- B. COPY INTO
- C. TRANSFER
- D. INSERT

<details><summary>Show Answer</summary>
Correct Answer: A. PUT uploads local files to an internal stage (COPY INTO then loads staged files into a table).
</details>

---

### Question 455
Which object type is granted permissions for reading a table?
- A. User
- B. Role
- C. Attribute
- D. Schema

<details><summary>Show Answer</summary>
Correct Answer: B. Snowflake uses RBAC — privileges are granted to roles, and roles are granted to users, not the other way around.
</details>

---

### Question 456
What is the default value in the Snowflake Web Interface (UI) for auto suspending a Virtual Warehouse?
- A. 1 minute
- B. 5 minutes
- C. 10 minutes
- D. 15 minutes

<details><summary>Show Answer</summary>
Correct Answer: C. The Snowsight default for AUTO_SUSPEND when creating a new warehouse is 10 minutes (600 seconds).
</details>

---

### Question 457
Several users are using the same virtual warehouse. The users report that the queries are running slowly, and that many queries are being queued. What is the recommended way to resolve this issue?
- A. Reduce the warehouse STATEMENT_TIMEOUT_IN_SECONDS parameter.
- B. Reduce the warehouse AUTO_SUSPEND parameter.
- C. Increase the warehouse MAX_CONCURRENCY_LEVEL parameter.
- D. Increase the warehouse MAX_CLUSTER_COUNT parameter.

<details><summary>Show Answer</summary>
Correct Answer: D. Enabling/increasing multi-cluster scaling (MAX_CLUSTER_COUNT) allows additional clusters to spin up automatically to absorb concurrent load and reduce queuing.
</details>

---

### Question 458
Which data types are valid in Snowflake? (Choose two.)
- A. BLOB
- B. Geography
- C. XML
- D. CLOB
- E. Variant

<details><summary>Show Answer</summary>
Correct Answer: B, E. GEOGRAPHY (and GEOMETRY) and VARIANT are valid Snowflake data types; BLOB, CLOB, and XML are not standalone Snowflake data types (XML-formatted data is stored as VARIANT).
</details>

---

### Question 459
What happens when the size of a virtual warehouse is changed?
- A. Queries that are running on the current warehouse are not impacted.
- B. Queries that are running on the current warehouse configuration are aborted and have to be resubmitted by the user.
- C. Queries that are running on the current warehouse configuration are aborted and are automatically resubmitted.
- D. Queries that are running on the current warehouse configuration are moved to the new configuration and finished.

<details><summary>Show Answer</summary>
Correct Answer: A. Resizing does not affect currently executing statements; new resources only apply to queued and future statements.
</details>

---

### Question 460
How often are encryption keys automatically rotated by Snowflake?
- A. 30 Days
- B. 60 Days
- C. 90 Days
- D. 365 Days

<details><summary>Show Answer</summary>
Correct Answer: A. Snowflake-managed keys (Account Master Keys, Table Master Keys, etc.) are automatically rotated every 30 days. Verified current as of July 2026.
</details>

---

### Question 461
As a best practice, all custom roles should be granted to which system-defined role?
- A. ACCOUNTADMIN
- B. ORGADMIN
- C. SECURITYADMIN
- D. SYSADMIN

<details><summary>Show Answer</summary>
Correct Answer: D. Best practice is to grant custom (functional) roles up to SYSADMIN, keeping object/warehouse administration separate from account-level administration.
</details>

---

### Question 462
Which Snowflake object can be accessed in the FROM clause of a query, returning a set of rows having one or more columns?
- A. A User-Defined Table Function
- B. A Scalar User-Defined Function (UDF)
- C. A Stored procedure
- D. A task

<details><summary>Show Answer</summary>
Correct Answer: A. A User-Defined Table Function (UDTF) returns a set of rows and can be referenced in a FROM clause (a scalar UDF returns only a single value).
</details>

---

### Question 463
How are micro-partitions typically generated in Snowflake?
- A. Automatically
- B. ORDER BY
- C. PARTITION BY
- D. GROUP BY

<details><summary>Show Answer</summary>
Correct Answer: A. Micro-partitioning is fully automatic based on the natural ingestion order of data — no manual partitioning syntax is required.
</details>

---

### Question 464
What does Snowflake recommend regarding database object ownership? (Choose two.)
- A. Create objects with ACCOUNTADMIN and do not reassign ownership.
- B. Create objects with SYSADMIN.
- C. Create with SECURITYADMIN to ease granting of privileges later.
- D. Create objects with a custom role and grant this role to SYSADMIN.
- E. Use only managed access schemas for objects owned by ACCOUNTADMIN.

<details><summary>Show Answer</summary>
Correct Answer: B, D. Snowflake recommends creating objects using SYSADMIN or a custom role that is itself granted to SYSADMIN — keeping ACCOUNTADMIN reserved for account-level administration only.
</details>

---

### Question 465
Other than ownership, what privileges does a user need to view and modify resource monitors in Snowflake? (Choose two.)
- A. ALTER
- B. MONITOR
- C. MODIFY
- D. CREATE
- E. DROP

<details><summary>Show Answer</summary>
Correct Answer: B, C. MONITOR allows viewing a resource monitor's details, while MODIFY allows changing its configuration.
</details>

---

### Question 466
What technique does Snowflake recommend for determining which virtual warehouse size to select?
- A. Always start with an X-Small and increase the size if the query does not complete in 2 minutes.
- B. Experiment by running the same queries against warehouses of different sizes.
- C. Use the default size Snowflake chooses.
- D. Use X-Large or above for tables larger than 1 GB.

<details><summary>Show Answer</summary>
Correct Answer: B. Snowflake recommends empirically testing the same query workload on different warehouse sizes to find the best fit.
</details>

---

### Question 467
Which command should be used when loading many flat files into a single table?
- A. PUT
- B. INSERT
- C. COPY INTO
- D. MERGE

<details><summary>Show Answer</summary>
Correct Answer: C. COPY INTO is the standard bulk-loading command for moving staged flat files into a table.
</details>

---

### Question 468
How can a Snowflake user share data with another user who does not have a Snowflake account?
- A. Share the data by implementing User-Defined Functions (UDFs).
- B. Create a reader account and create a share of the data.
- C. Grant the READER privilege to the database that is going to be shared.
- D. Move the Snowflake account to a region where data sharing is enabled.

<details><summary>Show Answer</summary>
Correct Answer: B. A reader account lets a provider extend Snowflake access to a consumer who does not have their own Snowflake account.
</details>

---

### Question 469
Which semi-structured data formats can be loaded into Snowflake with a COPY command? (Choose two.)
- A. CSV
- B. EDI
- C. HTML
- D. ORC
- E. XML

<details><summary>Show Answer</summary>
Correct Answer: D, E. Supported semi-structured formats for COPY INTO include JSON, Avro, ORC, Parquet, and XML (CSV is structured, not semi-structured; EDI and HTML are not supported).
</details>

---

### Question 470
Which statements reflect valid commands using secondary roles? (Choose two.)
- A. USE SECONDARY ROLES RESUME
- B. USE SECONDARY ROLES SUSPEND
- C. USE SECONDARY ROLES ALL
- D. USE SECONDARY ROLES ADD [Role Name]
- E. USE SECONDARY ROLES NONE

<details><summary>Show Answer</summary>
Correct Answer: C, E. USE SECONDARY ROLES only accepts ALL or NONE as valid arguments — you cannot add individual roles to the secondary roles list.
</details>

---

### Question 471
How long is a query visible in the Query History page in the Snowflake Web Interface (UI)?
- A. 60 minutes
- B. 24 hours
- C. 14 days
- D. 30 days

<details><summary>Show Answer</summary>
Correct Answer: C. The Snowsight Query History page displays queries executed over the last 14 days. Verified current as of July 2026.
</details>

---

### Question 472
Two users share a virtual warehouse. When one of the users loads data, the other one experiences performance issues while querying data. How does Snowflake recommend resolving this issue?
- A. Scale up the existing warehouse.
- B. Create separate warehouses for each user.
- C. Create separate warehouses for each workload.
- D. Stop loading and querying data at the same time.

<details><summary>Show Answer</summary>
Correct Answer: C. Snowflake recommends isolating different workload types (e.g., loading vs. querying) onto separate warehouses to avoid resource contention.
</details>

---

### Question 473
What is a feature of a stored procedure in Snowflake?
- A. They can be created as secure and hide the underlying metadata from all users.
- B. They can only access tables from a single database.
- C. They can contain a single statement.
- D. They can be created to run with a caller's rights or an owner's rights.

<details><summary>Show Answer</summary>
Correct Answer: D. Stored procedures can be defined with EXECUTE AS CALLER or EXECUTE AS OWNER, controlling whose privileges are used at runtime.
</details>

---

### Question 474
Which view will return users who have queried a table?
- A. SNOWFLAKE.ACCOUNT_USAGE.COLUMNS
- B. SNOWFLAKE.ACCOUNT_USAGE.VIEWS
- C. SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY
- D. SNOWFLAKE.ACCOUNT_USAGE.OBJECT_DEPENDENCIES

<details><summary>Show Answer</summary>
Correct Answer: C. ACCESS_HISTORY records which users read from or wrote to specific tables and columns.
</details>

---

### Question 475
Why do Snowflake's virtual warehouses have scaling policies?
- A. To help save storage costs
- B. To help increase the performance of serverless computing features
- C. To help control the credits consumed by a multi-cluster warehouse running in autoscale mode
- D. To help control the credits consumed by a multi-cluster warehouse running in maximized mode

<details><summary>Show Answer</summary>
Correct Answer: C. Scaling policies (Standard vs. Economy) govern how aggressively a multi-cluster warehouse in Auto-scale mode starts/stops additional clusters, balancing performance against credit consumption.
</details>

---

### Question 476
Where can a Snowflake user find the query history in Snowsight?
- A. Admin
- B. Activity
- C. Compute
- D. Data

<details><summary>Show Answer</summary>
Correct Answer: B. ⚠ Updated: In current Snowsight navigation, Query History is found under the **Monitoring** menu rather than "Activity" (the "Activity" label was used in an earlier version of Snowsight's navigation). Of the given options, "Activity" remains the closest legacy match, but be aware the current UI labels this section "Monitoring."
</details>

---

### Question 477
What is SnowSQL?
- A. Snowflake's new user interface where users can visualize data into charts and dashboards.
- B. Snowflake's proprietary extension of the ANSI SQL standard, including built-in keywords and system functions.
- C. Snowflake's command line client built on the Python Connector which is used to connect to Snowflake and execute SQL.
- D. Snowflake's library that provides a programming interface for processing data on Snowflake without moving it to the system where the application code resides.

<details><summary>Show Answer</summary>
Correct Answer: C. SnowSQL is the command-line client, built on the Python Connector, used to execute SQL and perform DDL/DML operations against Snowflake.
</details>

---

### Question 478
The following SQL statements have been executed:
`CREATE SEQUENCE seq_01;`
`SELECT seq_01.nextval;`
`SELECT seq_01.nextval;`
What will be the output of the last select statement?
- A. 0
- B. 1
- C. 2
- D. 3

<details><summary>Show Answer</summary>
Correct Answer: C. By default, a new sequence starts at 1 and increments by 1, so the first NEXTVAL call returns 1 and the second returns 2.
</details>

---

### Question 479
Which statement is true of Cloning?
- A. It increases storage costs as cloning a table requires storing its data twice.
- B. A cloned table includes the load history of the original.
- C. It is licensed as an additional Snowflake feature.
- D. All micro-partitions between the original and cloned tables are fully shared.

<details><summary>Show Answer</summary>
Correct Answer: D. Zero-copy cloning shares the same underlying micro-partitions initially; storage costs are only incurred once data diverges (no additional cost at clone time, and it is a native, unlicensed feature).
</details>

---

### Question 480
A Snowflake user has been granted the CREATE DATA EXCHANGE LISTING privilege with their role. Which tasks can this user now perform on the Data Exchange? (Choose two.)
- A. Rename listings
- B. Delete provider profiles
- C. Modify listing properties
- D. Modify incoming listing access requests
- E. Submit listings

<details><summary>Show Answer</summary>
Correct Answer: C, E. This privilege allows a user to create/submit new listings and modify listing properties.
</details>

---

### Question 481
Which parameter prevents streams on tables from becoming stale?
- A. MAX_DATA_EXTENSION_TIME_IN_DAYS
- B. DATA_RETENTION_TIME_IN_DAYS
- C. LOCK_TIMEOUT
- D. STALE_AFTER

<details><summary>Show Answer</summary>
Correct Answer: A. MAX_DATA_EXTENSION_TIME_IN_DAYS extends a table's data retention period as needed to keep dependent streams from becoming stale.
</details>

---

### Question 482
If a virtual warehouse runs for 30 seconds after it is provisioned, how many seconds will the customer be billed for?
- A. 30 seconds
- B. 60 seconds
- C. 121 seconds
- D. 1 hour

<details><summary>Show Answer</summary>
Correct Answer: B. Snowflake bills a 60-second minimum each time a warehouse starts, then per-second thereafter. Verified current as of July 2026.
</details>

---

### Question 483
When should a stored procedure be created with caller's rights?
- A. When the caller needs to be prevented from viewing the source code of the stored procedure
- B. When the caller needs to run a statement that could not execute outside of the stored procedure
- C. When the stored procedure needs to run with the privileges of the role that called the stored procedure
- D. When the stored procedure needs to operate on objects that the caller does not have privileges on

<details><summary>Show Answer</summary>
Correct Answer: C. Caller's rights procedures execute using the privileges of the role that called them, rather than the owner's privileges.
</details>

---

### Question 484
What JavaScript delimiters are available in Snowflake stored procedures? (Choose two.)
- A. Double quote ("")
- B. Single quote ('')
- C. Forward slash (//)
- D. Double backslash (\)
- E. Double dollar sign ($$)

<details><summary>Show Answer</summary>
Correct Answer: B, E. A JavaScript procedure body can be delimited with single quotes or double dollar signs.
</details>

---

### Question 485
What type of function can be used to estimate the approximate number of distinct values from a table that has trillions of rows?
- A. MD5
- B. Window
- C. External
- D. HyperLogLog (HLL)

<details><summary>Show Answer</summary>
Correct Answer: D. HyperLogLog functions (e.g., APPROX_COUNT_DISTINCT) provide fast, memory-efficient approximate distinct counts on very large datasets.
</details>

---

### Question 486
Which Data Definition Language (DDL) commands are supported by Snowflake to manage tags? (Choose two.)
- A. ALTER TAG
- B. DESCRIBE TAG
- C. CREATE TAG
- D. GRANT [privilege] TO TAG
- E. DROP TAG

<details><summary>Show Answer</summary>
Correct Answer: A, C. CREATE TAG and ALTER TAG are core DDL commands for managing tags (Snowflake also supports DROP TAG and SHOW TAGS, but of the listed options A and C are the intended pair).
</details>

---

### Question 487
What Snowflake objects can be added to a share? (Choose two.)
- A. Internal Stages
- B. Tables
- C. Stored procedures
- D. Users
- E. Secure Views

<details><summary>Show Answer</summary>
Correct Answer: B, E. Shares can include tables, secure views, and secure UDFs — internal stages, stored procedures, and users cannot be shared.
</details>

---

### Question 488
A Query Profile shows a UnionAll operator with an extra Aggregate operator on top. What does this signify?
- A. Exploding joins
- B. Inefficient pruning
- C. UNION without ALL
- D. Queries that are too large to fit in memory

<details><summary>Show Answer</summary>
Correct Answer: C. When a UNION (without ALL) is used, Snowflake performs a UnionAll followed by an Aggregate operator to deduplicate the results.
</details>

---

### Question 489
Which data governance control has Snowflake embedded in the application?
- A. Row access policies
- B. Credit computation
- C. Data storage
- D. Role-based access control

<details><summary>Show Answer</summary>
Correct Answer: A. Row access policies are a native Snowflake data governance feature controlling which rows a user can see.
</details>

---

### Question 490
What actions does the use of the PUT command do automatically? (Choose two.)
- A. It creates a file format object.
- B. It uses the last stage created.
- C. It compresses all files using GZIP.
- D. It encrypts the file data.
- E. It creates an empty target table.

<details><summary>Show Answer</summary>
Correct Answer: C, D. By default, PUT automatically compresses files with GZIP and encrypts them before uploading to a stage (both behaviors can be overridden with parameters).
</details>

---

### Question 491
Which command should a Snowflake user execute to load data into a table?
- A. COPY INTO mytable purge_mode = TRUE;
- B. COPY INTO mytable FROM @stage;
- C. COPY INTO mytable file_format = (format_name);
- D. COPY INTO my_table validation_mode = RETURN_ERRORS;

<details><summary>Show Answer</summary>
Correct Answer: B. COPY INTO mytable FROM @stage is valid, complete syntax for loading data (the other options reference invalid parameter names or are missing the required FROM clause).
</details>

---

### Question 492
Which function returns the URL of a stage using the stage name as the input?
- A. GET_STAGE_URL
- B. GENERATE_PRESIGNED_URL
- C. GET_STAGE_LOCATION
- D. GET_STAGE_FILE_URL

<details><summary>Show Answer</summary>
Correct Answer: C. GET_STAGE_LOCATION returns the cloud storage URL for a given stage.
</details>

---

### Question 493
Which is the MAXIMUM number of clusters that can be provisioned with a multi-cluster virtual warehouse?
- A. 1
- B. 5
- C. 10
- D. 100

<details><summary>Show Answer</summary>
Correct Answer: C. A multi-cluster warehouse supports a maximum of 10 clusters. Verified current as of July 2026.
</details>

---

### Question 494
Which Snowflake table supports unstructured data?
- A. Directory
- B. Transient
- C. Temporary
- D. Permanent

<details><summary>Show Answer</summary>
Correct Answer: A. ⚠ Updated: This question's framing is slightly outdated. Snowflake does not have a distinct "Directory table" *table type* — a directory table is metadata associated with an internal/external **stage** (enabled via `DIRECTORY = (ENABLE = TRUE)`) that catalogs unstructured files, not a standalone table type alongside Transient/Temporary/Permanent. Of the listed options, "Directory" is still the intended answer since it is the mechanism Snowflake uses to track and query unstructured files, but it is technically a stage property/table, not a table storage type.
</details>

---

### Question 495
When unloading data, which file format preserves the data values for floating-point number columns?
- A. CSV
- B. XML
- C. JSON
- D. Parquet

<details><summary>Show Answer</summary>
Correct Answer: D. Parquet preserves native floating-point precision, whereas CSV/JSON convert values to text representations that can lose precision.
</details>

---

### Question 496
Which virtual warehouse privilege is required to view a load-monitoring chart?
- A. MONITOR
- B. MODIFY
- C. OPERATE
- D. USAGE

<details><summary>Show Answer</summary>
Correct Answer: A. MONITOR privilege on a warehouse allows viewing its metrics, including load-monitoring charts.
</details>

---

### Question 497
Which use case will always cause an exploding join in Snowflake?
- A. A query that has more than 10 left outer joins.
- B. A query that is using a UNION without an ALL.
- C. A query that has not specified join criteria for tables.
- D. A query that has requested too many columns of data.

<details><summary>Show Answer</summary>
Correct Answer: C. Missing join criteria produces a cartesian product, causing row counts to explode.
</details>

---

### Question 498
How many resource monitors can be applied to a single virtual warehouse?
- A. Zero
- B. One
- C. Eight
- D. Unlimited

<details><summary>Show Answer</summary>
Correct Answer: B. A given warehouse can be assigned to only one resource monitor at a time (though one resource monitor can cover multiple warehouses).
</details>

---

### Question 499
What are the main differences between the account usage views and the information schema views? (Choose two.)
- A. No active warehouse is needed to query account usage views but one is needed to query information schema views.
- B. Account usage views do not contain data about tables but information schema views do.
- C. Account usage views contain dropped objects, information schema views do not.
- D. Data retention for account usage views is 1 year but is 7 days to 6 months for information schema views, depending on the view.
- E. Information schema views are read-only but account usage views are not.

<details><summary>Show Answer</summary>
Correct Answer: C, D. ACCOUNT_USAGE retains data for 365 days and includes dropped objects, while INFORMATION_SCHEMA has shorter, view-dependent retention (typically 7 days to 6 months) and excludes dropped objects.
</details>

---

### Question 500
Which file function generates a URL with access to a file on a stage without the need for authentication and authorization?
- A. BUILD_STAGE_FILE_URL
- B. GET_STAGE_LOCATION
- C. GET_PRESIGNED_URL
- D. BUILD_SCOPED_FILE_URL

<details><summary>Show Answer</summary>
Correct Answer: C. GET_PRESIGNED_URL generates a temporary URL that grants access to a staged file without requiring the requester to separately authenticate to Snowflake.
</details>

---

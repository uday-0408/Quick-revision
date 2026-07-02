# SnowPro Core – 100 Hard-Level Practice MCQs

> Click **"Show Answer"** under each question to reveal the correct option and explanation.

---

### Q1. A virtual warehouse is suspended mid-query due to a manual override. What happens to the running query?

A) The query is automatically requeued on the next warehouse resume
B) The query fails and must be resubmitted by the user
C) The warehouse ignores the suspend command until the query finishes
D) The query is migrated live to another running warehouse

<details><summary>Show Answer</summary>

**Answer: C) The warehouse ignores the suspend command until the query finishes**

A warehouse will not actually suspend while queries are still executing on it — the suspend action is deferred until all active queries complete. There is no live migration of a running query to a different warehouse, no automatic requeueing after resume, and the query does not simply fail because of a suspend request.
</details>

---

### Q2. Which statement about micro-partitions is correct?

A) Micro-partitions are typically between 50 MB and 500 MB of uncompressed data
B) Micro-partitions are typically between 50 MB and 500 MB of compressed data
C) A table row can span multiple micro-partitions if it is very wide
D) Micro-partitions must be manually created using the CREATE PARTITION command

<details><summary>Show Answer</summary>

**Answer: A) Micro-partitions are typically between 50 MB and 500 MB of uncompressed data**

Snowflake automatically divides table data into micro-partitions containing 50–500 MB of uncompressed data (stored compressed). Rows are never split across partitions, and there is no manual partition-creation command — partitioning is fully automatic.
</details>

---

### Q3. What metadata does Snowflake store for each micro-partition to enable pruning?

A) Row-level checksums for every column
B) Min/max values and distinct-value counts for each column in the partition
C) A full secondary index of every column value
D) A bloom filter for every row

<details><summary>Show Answer</summary>

**Answer: B) Min/max values and distinct-value counts for each column in the partition**

Snowflake maintains column-level metadata such as min/max values, null counts, and distinct value counts per micro-partition, which the query optimizer uses for pruning. It does not maintain row-level checksums, per-row bloom filters, or full secondary indexes.
</details>

---

### Q4. A table has natural clustering that has degraded over time. Which action directly improves clustering without rewriting the entire table history?

A) TRUNCATE and reload the table
B) Enable automatic clustering with a clustering key
C) Run VACUUM on the table
D) Increase the size of the virtual warehouse used for queries

<details><summary>Show Answer</summary>

**Answer: B) Enable automatic clustering with a clustering key**

Defining a clustering key and enabling automatic clustering lets Snowflake incrementally reorganize micro-partitions in the background. Snowflake has no VACUUM command, truncating and reloading is destructive and unnecessary, and warehouse size affects compute for queries, not clustering quality.
</details>

---

### Q5. What is the default Time Travel retention period for a table in the Enterprise edition on a permanent table?

A) 0 days
B) 1 day
C) 90 days
D) 1 day, but can be configured up to 90 days

<details><summary>Show Answer</summary>

**Answer: D) 1 day, but can be configured up to 90 days**

The default retention is 1 day across editions, but Enterprise and higher editions allow DATA_RETENTION_TIME_IN_DAYS to be configured up to 90 days for permanent tables. It is not fixed at 90 days by default, and it is not 0 days for permanent tables.
</details>

---

### Q6. Which object type does NOT support Time Travel?

A) Permanent tables
B) Transient tables
C) External tables
D) Temporary tables

<details><summary>Show Answer</summary>

**Answer: C) External tables**

External tables reference data stored outside Snowflake and do not support Time Travel. Permanent and transient tables support at least 1 day, and temporary tables support Time Travel only for the life of the session.
</details>

---

### Q7. A Fail-safe period begins immediately after which event?

A) The moment a table is created
B) The moment Time Travel retention expires for a table
C) The moment a warehouse is suspended
D) The moment a database is cloned

<details><summary>Show Answer</summary>

**Answer: B) The moment Time Travel retention expires for a table**

Fail-safe is a 7-day period (for permanent tables) that begins automatically once the Time Travel retention window ends, providing a final recovery mechanism accessible only by Snowflake support. It is not tied to warehouse state or cloning events.
</details>

---

### Q8. Which statement about zero-copy cloning is accurate?

A) Cloning immediately duplicates all physical micro-partitions
B) A clone shares underlying micro-partitions with the source until either object modifies the data
C) Clones cannot be created from a specific point in Time Travel history
D) Cloning a database also clones all Snowpipe pipe load history as new independent pipes

<details><summary>Show Answer</summary>

**Answer: B) A clone shares underlying micro-partitions with the source until either object modifies the data**

Zero-copy clones initially reference the same immutable micro-partitions as the source, with storage diverging only when new writes occur (copy-on-write). Clones can be created from historical points using Time Travel, and physical data is not duplicated at clone time.
</details>

---

### Q9. Which layer of Snowflake's architecture is responsible for query optimization, transaction management, and metadata?

A) Storage layer
B) Cloud services layer
C) Compute layer (virtual warehouses)
D) The cloud provider's native scheduler

<details><summary>Show Answer</summary>

**Answer: B) Cloud services layer**

The cloud services layer handles authentication, metadata management, query optimization, and transaction coordination. The storage layer holds compressed data, and the compute layer executes queries — neither performs optimization or metadata management themselves.
</details>

---

### Q10. What happens to data stored in the storage layer when all virtual warehouses in an account are suspended?

A) The data becomes temporarily inaccessible until a warehouse resumes
B) The data is automatically moved to Fail-safe
C) The data remains fully intact and accessible for metadata operations
D) The data is compressed further to save costs

<details><summary>Show Answer</summary>

**Answer: C) The data remains fully intact and accessible for metadata operations**

Storage is decoupled from compute, so suspending all warehouses does not affect stored data; only operations requiring compute (like querying) need a running warehouse. Data isn't moved to Fail-safe or recompressed simply because warehouses are suspended.
</details>

---

### Q11. A multi-cluster warehouse is set to "Auto-scale" mode with a minimum of 1 and maximum of 4 clusters. Which scaling policy minimizes credit usage at the cost of potential queuing?

A) Standard scaling policy
B) Economy scaling policy
C) Aggressive scaling policy
D) Balanced scaling policy

<details><summary>Show Answer</summary>

**Answer: B) Economy scaling policy**

The Economy scaling policy favors conserving credits by only starting additional clusters when it estimates there is enough queued workload to keep a new cluster busy for at least 6 minutes, which can lead to more queuing than the Standard policy. "Aggressive" and "Balanced" are not real Snowflake scaling policy names.
</details>

---

### Q12. Which SQL function would you use to view the query history including credits consumed, without enabling ACCOUNT_USAGE latency delays?

A) SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
B) SELECT * FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY())
C) SHOW QUERY_HISTORY
D) DESCRIBE QUERY_HISTORY

<details><summary>Show Answer</summary>

**Answer: B) SELECT * FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY())**

The INFORMATION_SCHEMA table function returns near real-time query history with low latency, whereas ACCOUNT_USAGE views can lag up to 45 minutes to 3 hours. SHOW QUERY_HISTORY and DESCRIBE QUERY_HISTORY are not valid Snowflake commands.
</details>

---

### Q13. What is the primary purpose of the RESULT_SCAN function?

A) To scan the results of a previously executed query without re-running it
B) To scan raw files in an external stage
C) To scan and validate table constraints
D) To estimate the cost of a future query before execution

<details><summary>Show Answer</summary>

**Answer: A) To scan the results of a previously executed query without re-running it**

RESULT_SCAN lets you query the result set of a prior statement (using a query ID or LAST_QUERY_ID()) directly from the result cache. It has nothing to do with scanning stage files, validating constraints, or cost estimation.
</details>

---

### Q14. Which caching layer persists even after a virtual warehouse is suspended and later resumed (assuming it wasn't dropped)?

A) The local disk (SSD) cache on compute nodes
B) The result cache in the cloud services layer
C) The metadata cache in the compute layer
D) The warehouse-local query plan cache

<details><summary>Show Answer</summary>

**Answer: B) The result cache in the cloud services layer**

The result cache is maintained independently of any warehouse for 24 hours (extendable up to 31 days on repeated use) and survives warehouse suspension. The local SSD cache is tied to warehouse compute nodes and is lost when a warehouse suspends.
</details>

---

### Q15. A user runs the same query twice in a row on a suspended-then-resumed warehouse, with no underlying data changes. Why might the second run NOT use the result cache?

A) Result caching is disabled by default for all accounts
B) The query includes a non-deterministic function such as CURRENT_TIMESTAMP()
C) The warehouse was resized between runs
D) Result caching only works within the same session

<details><summary>Show Answer</summary>

**Answer: B) The query includes a non-deterministic function such as CURRENT_TIMESTAMP()**

Queries containing non-deterministic functions bypass the result cache because the output could differ each run. Result caching is enabled by default, works across sessions/users, and is unaffected by warehouse resizing.
</details>

---

### Q16. Which command correctly creates a masking policy that redacts an SSN column for all roles except AUDITOR?

A) CREATE MASKING POLICY ssn_mask AS (val STRING) RETURNS STRING -> CASE WHEN CURRENT_ROLE() = 'AUDITOR' THEN val ELSE '***-**-****' END
B) CREATE POLICY MASKING ssn_mask ON COLUMN ssn REDACT EXCEPT AUDITOR
C) ALTER TABLE customers MASK COLUMN ssn WITH ROLE AUDITOR
D) CREATE ROW ACCESS POLICY ssn_mask AS (val STRING) RETURNS STRING -> val

<details><summary>Show Answer</summary>

**Answer: A) CREATE MASKING POLICY ssn_mask AS (val STRING) RETURNS STRING -> CASE WHEN CURRENT_ROLE() = 'AUDITOR' THEN val ELSE '***-**-****' END**

Column-level dynamic data masking uses CREATE MASKING POLICY with a CASE expression evaluated against session context like CURRENT_ROLE(). Options B and C use invalid syntax, and row access policies (option D) control row visibility, not column value redaction.
</details>

---

### Q17. A row access policy is applied to a table. Which statement is true about how it interacts with a masking policy on the same table?

A) Only one of the two policy types can be active on a table at a time
B) Row access policies and masking policies can both be applied simultaneously, filtering rows and masking values independently
C) Masking policies always override row access policies
D) Row access policies must be dropped before a masking policy can be created

<details><summary>Show Answer</summary>

**Answer: B) Row access policies and masking policies can both be applied simultaneously, filtering rows and masking values independently**

These two policy types operate independently and can coexist: row access policies determine which rows are visible, while masking policies determine how column values within visible rows are displayed. There's no restriction requiring one to be dropped for the other to exist.
</details>

---

### Q18. What is required for a role to be able to create a masking policy in a schema?

A) OWNERSHIP privilege on the database only
B) The APPLY MASKING POLICY privilege combined with USAGE on the schema
C) CREATE MASKING POLICY privilege on the schema
D) The ACCOUNTADMIN role exclusively

<details><summary>Show Answer</summary>

**Answer: C) CREATE MASKING POLICY privilege on the schema**

A role needs the CREATE MASKING POLICY privilege granted on the containing schema to define new masking policies there. APPLY MASKING POLICY is a separate privilege needed to attach a policy to a column, and ACCOUNTADMIN is not strictly required if the privilege has been granted elsewhere.
</details>

---

### Q19. Which statement about Secure Views is correct?

A) Secure Views expose their underlying query definition to any user with SELECT access
B) Secure Views prevent the query optimizer from bypassing the view's WHERE clause using injected predicates, at the cost of some optimization
C) Secure Views can only be created on external tables
D) Secure Views are always materialized on creation

<details><summary>Show Answer</summary>

**Answer: B) Secure Views prevent the query optimizer from bypassing the view's WHERE clause using injected predicates, at the cost of some optimization**

Secure views hide the underlying SQL definition and prevent certain optimizations (like predicate pushdown reordering) that could otherwise leak data through crafted queries, trading some performance for security. Regular views, not secure views, expose their definitions to viewers with sufficient privilege, and secure views are not materialized.
</details>

---

### Q20. In Snowflake's RBAC model, which system role should typically be used to manage users and roles, but NOT directly own data objects?

A) ACCOUNTADMIN
B) SYSADMIN
C) SECURITYADMIN
D) PUBLIC

<details><summary>Show Answer</summary>

**Answer: C) SECURITYADMIN**

SECURITYADMIN is intended to manage grants, users, and roles account-wide, while SYSADMIN is the recommended role for creating and owning warehouses, databases, and other data objects. ACCOUNTADMIN encompasses both but is reserved for top-level administration, and PUBLIC is the default role granted to every user.
</details>

---

### Q21. What is the effect of the GRANT ALL PRIVILEGES ON FUTURE TABLES IN SCHEMA command?

A) It grants privileges retroactively to all tables that already existed before the schema was created
B) It grants the specified privileges automatically to tables created in that schema after the grant is issued
C) It only applies to tables created by the ACCOUNTADMIN role
D) It has no effect unless combined with a stream

<details><summary>Show Answer</summary>

**Answer: B) It grants the specified privileges automatically to tables created in that schema after the grant is issued**

FUTURE grants apply to objects of the specified type created after the grant statement runs, not to pre-existing objects. It works regardless of which role creates the future table and has nothing to do with streams.
</details>

---

### Q22. A Snowpipe is configured for auto-ingest using cloud storage event notifications. Which statement about latency is accurate?

A) Files are guaranteed to load within 1 second of arrival
B) Snowpipe typically loads data within about 1 minute of file notification, but this is not a strict SLA
C) Snowpipe only checks for new files once every 24 hours
D) Auto-ingest requires manual triggering via REFRESH each time

<details><summary>Show Answer</summary>

**Answer: B) Snowpipe typically loads data within about 1 minute of file notification, but this is not a strict SLA**

Snowpipe is designed for near-real-time loading, typically completing within about a minute of receiving an event notification, though Snowflake does not guarantee an exact SLA. It is not a 1-second guarantee, doesn't check only daily, and auto-ingest does not require manual REFRESH.
</details>

---

### Q23. Which cost is incurred by Snowpipe usage that is separate from standard virtual warehouse credits?

A) Snowpipe uses a per-file flat fee regardless of file size or compute time
B) Snowpipe consumes serverless compute billed per second based on Snowflake-managed resources
C) Snowpipe has no associated compute cost, only storage cost
D) Snowpipe billing is identical to a Small warehouse running continuously

<details><summary>Show Answer</summary>

**Answer: B) Snowpipe consumes serverless compute billed per second based on Snowflake-managed resources**

Snowpipe uses Snowflake-managed serverless compute resources, billed based on actual per-second resource consumption for loading files, rather than a fixed per-file fee or a dedicated customer-managed warehouse.
</details>

---

### Q24. A Stream object is created on a table to enable change data capture. What happens to the stream's offset when a consuming DML transaction successfully commits against it?

A) The stream is dropped automatically
B) The stream's offset advances to reflect that the changes have been consumed
C) The stream resets to the table's original creation time
D) The stream must be manually refreshed using ALTER STREAM REFRESH

<details><summary>Show Answer</summary>

**Answer: B) The stream's offset advances to reflect that the changes have been consumed**

When a DML statement reads from a stream within a transaction that commits, the stream's offset advances past the consumed changes so they aren't returned again. Streams aren't dropped automatically, don't reset to creation time, and there is no ALTER STREAM REFRESH command.
</details>

---

### Q25. What does the METADATA$ACTION column in a stream indicate?

A) The user who performed the change
B) Whether the row change was an INSERT or DELETE
C) The warehouse used to make the change
D) The exact timestamp of the change

<details><summary>Show Answer</summary>

**Answer: B) Whether the row change was an INSERT or DELETE**

Streams include METADATA$ACTION (INSERT/DELETE), METADATA$ISUPDATE (whether it's part of an update), and METADATA$ROW_ID, but not the initiating user, warehouse, or an explicit timestamp column.
</details>

---

### Q26. A task is configured with `AFTER` to run following a predecessor task. What is the maximum number of tasks that can exist in a single task tree (as a general large practical limit prior to task graph enhancements)?

A) There is no limit; task trees can be infinitely large
B) 100 tasks per tree
C) 1,000 tasks per tree
D) 10 tasks per tree

<details><summary>Show Answer</summary>

**Answer: C) 1,000 tasks per tree**

A task graph (tree rooted at a scheduled task) supports up to 1,000 tasks, including the root task. There is a defined limit rather than an unbounded tree, and it is far larger than 10 or 100.
</details>

---

### Q27. Which condition must be true for a child task to execute after its predecessor completes?

A) The child task's SYSTEM$STREAM_HAS_DATA condition must always be true
B) The predecessor task must complete successfully, and if a WHEN condition is defined, it must evaluate to true
C) The child task must be manually resumed each time
D) Child tasks always run regardless of predecessor success or failure

<details><summary>Show Answer</summary>

**Answer: B) The predecessor task must complete successfully, and if a WHEN condition is defined, it must evaluate to true**

Task execution in a graph requires the predecessor to succeed; an optional WHEN clause (often using SYSTEM$STREAM_HAS_DATA) adds a further conditional gate. SYSTEM$STREAM_HAS_DATA is only relevant if explicitly used in a WHEN clause, not a universal requirement, and child tasks do not run after a failed predecessor by default.
</details>

---

### Q28. What is the effect of setting a resource monitor's action to "SUSPEND_IMMEDIATE" upon reaching 100% of quota?

A) Running queries finish, but no new queries can start until the next billing cycle
B) All currently running queries on affected warehouses are canceled immediately, and the warehouses are suspended
C) The warehouse is dropped permanently
D) Only new queries submitted after the threshold are throttled, with no impact on running queries

<details><summary>Show Answer</summary>

**Answer: B) All currently running queries on affected warehouses are canceled immediately, and the warehouses are suspended**

SUSPEND_IMMEDIATE cancels all currently executing statements right away and suspends the warehouse(s), unlike the plain SUSPEND action which lets running queries finish first. It does not drop the warehouse object itself.
</details>

---

### Q29. Which privilege must be granted to allow a role to receive notifications from a resource monitor without administrative access?

A) Resource monitors do not support user-level notification recipients other than account admins
B) Assigning notification recipients is configured via the Snowsight UI or by account admins, associated with users who have opted in to receive alerts
C) MONITOR USAGE privilege must be granted explicitly to each recipient
D) Recipients must be granted OWNERSHIP on the resource monitor

<details><summary>Show Answer</summary>

**Answer: B) Assigning notification recipients is configured via the Snowsight UI or by account admins, associated with users who have opted in to receive alerts**

Resource monitor notifications are configured by account administrators and delivered to users who have enabled notifications for their account, rather than through a dedicated grantable privilege per recipient. There isn't a MONITOR USAGE-style per-user grant or an ownership requirement for receiving alerts.
</details>

---

### Q30. A table is defined as TRANSIENT. Which statement correctly describes its Fail-safe behavior?

A) Transient tables have the standard 7-day Fail-safe period
B) Transient tables have no Fail-safe period
C) Transient tables have a configurable Fail-safe period up to 90 days
D) Transient tables have a fixed 1-day Fail-safe period

<details><summary>Show Answer</summary>

**Answer: B) Transient tables have no Fail-safe period**

Transient tables trade off Fail-safe protection (0 days) in exchange for lower storage costs, while still supporting Time Travel (up to 1 day). Only permanent tables carry the standard 7-day Fail-safe.
</details>

---

### Q31. What is the maximum Time Travel retention period configurable for a TRANSIENT table?

A) 0 days
B) 1 day
C) 7 days
D) 90 days

<details><summary>Show Answer</summary>

**Answer: B) 1 day**

Transient (and temporary) tables support a maximum Time Travel retention of 1 day, unlike permanent tables which can go up to 90 days on Enterprise edition and above.
</details>

---

### Q32. A user wants to share live data with a business partner who does not have a Snowflake account. Which feature enables this?

A) Standard Secure Data Sharing (requires the consumer to have a Snowflake account)
B) Reader Accounts, provisioned and managed by the data provider
C) SnowSQL export to CSV, emailed manually
D) Database replication only

<details><summary>Show Answer</summary>

**Answer: B) Reader Accounts, provisioned and managed by the data provider**

Reader accounts allow a provider to share data with consumers who don't have their own Snowflake account; the provider creates and manages the reader account and bears its compute costs. Standard secure sharing requires the consumer to already be a Snowflake customer, and manual CSV export isn't a live-sharing mechanism.
</details>

---

### Q33. Who is billed for the compute costs incurred by a Reader Account querying shared data?

A) The reader account's own billing entity
B) The data provider account that created the reader account
C) Snowflake absorbs this cost as part of the platform fee
D) The costs are split evenly between provider and consumer automatically

<details><summary>Show Answer</summary>

**Answer: B) The data provider account that created the reader account**

Since a reader account has no independent Snowflake contract, all compute and other charges it incurs are billed back to the provider account that created and manages it.
</details>

---

### Q34. Which object must be created before setting up a data share to expose specific tables to a consumer account?

A) A stage
B) A share object, using CREATE SHARE
C) A pipe
D) A stream

<details><summary>Show Answer</summary>

**Answer: B) A share object, using CREATE SHARE**

Data sharing is built around SHARE objects to which the provider grants database/schema/table access, and which are then made available to consumer accounts. Stages, pipes, and streams serve entirely different purposes (loading and change tracking, not cross-account sharing).
</details>

---

### Q35. In the context of Snowflake's Search Optimization Service, which query pattern benefits most?

A) Full table scans with no filter predicates
B) Highly selective point-lookup queries on columns not used as clustering keys
C) Queries that aggregate over the entire table
D) DDL statements like CREATE TABLE

<details><summary>Show Answer</summary>

**Answer: B) Highly selective point-lookup queries on columns not used as clustering keys**

Search Optimization builds a persisted maintenance structure that accelerates highly selective equality/substring lookups on columns that aren't already well-served by clustering, dramatically reducing partitions scanned. It offers little benefit for full scans, broad aggregations, or DDL.
</details>

---

### Q36. What type of cost does enabling Search Optimization Service primarily add, beyond standard warehouse compute for queries?

A) A one-time flat licensing fee
B) Ongoing storage cost for the search access path plus maintenance compute cost
C) No additional cost of any kind
D) A per-query flat surcharge billed to the consumer

<details><summary>Show Answer</summary>

**Answer: B) Ongoing storage cost for the search access path plus maintenance compute cost**

Search Optimization Service incurs additional storage for its internal search access data structure and background serverless compute to keep it maintained as data changes, on top of normal warehouse compute used to run queries.
</details>

---

### Q37. Which of the following is true about materialized views in Snowflake?

A) They can be defined with joins across multiple large tables freely without restriction
B) They support a limited subset of SQL — for example, they cannot contain joins, cannot use HAVING with certain constructs, and have other restrictions
C) They are refreshed only when manually triggered by the user
D) They eliminate the need for a virtual warehouse entirely to refresh

<details><summary>Show Answer</summary>

**Answer: B) They support a limited subset of SQL — for example, they cannot contain joins, cannot use HAVING with certain constructs, and have other restrictions**

Materialized views in Snowflake support only a single source table (no joins) among other restrictions, unlike full views. They are refreshed automatically in the background using serverless compute, not purely on manual trigger, and background refresh still consumes compute resources.
</details>

---

### Q38. Which scenario is materialized view maintenance billed under?

A) The customer's specified virtual warehouse, chosen at query time
B) Serverless compute managed by Snowflake, billed separately from customer warehouses
C) It is entirely free as part of storage costs
D) It uses the ACCOUNTADMIN's default warehouse only

<details><summary>Show Answer</summary>

**Answer: B) Serverless compute managed by Snowflake, billed separately from customer warehouses**

Background maintenance (refresh) of materialized views uses Snowflake-managed serverless compute, billed independently of any customer-defined virtual warehouse.
</details>

---

### Q39. What does the QUERY_ACCELERATION_SERVICE primarily help with?

A) Speeding up DDL execution
B) Offloading portions of eligible queries (e.g., large scans with filters) to serverless compute to reduce wall-clock time
C) Reducing storage costs for compressed data
D) Automatically rewriting inefficient SQL syntax

<details><summary>Show Answer</summary>

**Answer: B) Offloading portions of eligible queries (e.g., large scans with filters) to serverless compute to reduce wall-clock time**

Query Acceleration Service identifies portions of a query — typically large scan-and-filter operations — that can be parallelized out to additional serverless compute resources, reducing overall query duration for eligible workloads. It doesn't rewrite SQL, and it isn't about DDL or storage cost reduction.
</details>

---

### Q40. A developer wants to test whether a table would benefit from a clustering key before committing to one. Which function helps assess current clustering quality?

A) SYSTEM$CLUSTERING_INFORMATION
B) SYSTEM$ESTIMATE_QUERY_COST
C) SYSTEM$TABLE_SIZE
D) SYSTEM$PIPE_STATUS

<details><summary>Show Answer</summary>

**Answer: A) SYSTEM$CLUSTERING_INFORMATION**

SYSTEM$CLUSTERING_INFORMATION returns clustering depth and partition overlap statistics for a table, helping assess whether a clustering key would help. The other functions listed either don't exist as described or serve unrelated purposes (e.g., SYSTEM$PIPE_STATUS checks Snowpipe status).
</details>

---

### Q41. Which statement about automatic clustering costs is correct?

A) Automatic clustering uses the warehouse specified in the session running the query
B) Automatic clustering consumes serverless compute credits billed independently of customer warehouses
C) Automatic clustering is entirely free
D) Automatic clustering can only run during scheduled maintenance windows set by the customer

<details><summary>Show Answer</summary>

**Answer: B) Automatic clustering consumes serverless compute credits billed independently of customer warehouses**

Reclustering happens automatically in the background using Snowflake-managed serverless resources, billed to the account separately from any customer-controlled virtual warehouse, and is not free or bound to a customer-defined maintenance window.
</details>

---

### Q42. What happens when you execute UNDROP TABLE on a table that was dropped 2 days ago, given the table's retention period was 1 day?

A) The table is restored successfully regardless of retention period
B) The command fails because the table has passed its Time Travel retention window and moved to Fail-safe
C) The table is restored but with all data purged
D) UNDROP always works within a 7-day grace period regardless of table retention settings

<details><summary>Show Answer</summary>

**Answer: B) The command fails because the table has passed its Time Travel retention window and moved to Fail-safe**

UNDROP relies on the Time Travel retention window; once that window (1 day here) has elapsed, the object enters Fail-safe, which is recoverable only via Snowflake support, not through UNDROP.
</details>

---

### Q43. Which of the following correctly describes "AT" and "BEFORE" clauses in Time Travel queries?

A) AT includes a specified point in time; BEFORE selects data strictly prior to a specified point (e.g., before a specific statement)
B) AT and BEFORE are functionally identical and interchangeable
C) AT only works with timestamps, and BEFORE only works with offsets
D) BEFORE only applies to schema-level Time Travel, not tables

<details><summary>Show Answer</summary>

**Answer: A) AT includes a specified point in time; BEFORE selects data strictly prior to a specified point (e.g., before a specific statement)**

AT returns the state inclusive of the specified point (timestamp, offset, or statement), while BEFORE returns the state immediately preceding it — useful for undoing a specific mistaken statement. Both clauses accept timestamps, offsets, or statement IDs, not restricted to only one type each.
</details>

---

### Q44. A COPY INTO command is run with `ON_ERROR = 'SKIP_FILE'`. What happens if one row in a file fails to parse?

A) Only that row is skipped; the rest of the file loads normally
B) The entire file is skipped and none of its rows are loaded
C) The load aborts entirely for all files in the operation
D) The row is loaded with NULL values substituted

<details><summary>Show Answer</summary>

**Answer: B) The entire file is skipped and none of its rows are loaded**

With ON_ERROR = 'SKIP_FILE', any error in a file causes that whole file to be skipped, unlike 'CONTINUE' which skips only the problematic rows. It does not abort the entire multi-file operation, nor does it substitute NULLs for the bad row while keeping the rest of the file.
</details>

---

### Q45. Which staged file format option would you use to load JSON data where each file may contain multiple top-level JSON objects concatenated without an enclosing array?

A) TYPE = 'JSON', STRIP_OUTER_ARRAY = TRUE required always
B) TYPE = 'JSON' (Snowflake can parse concatenated JSON documents natively without needing an outer array)
C) TYPE = 'CSV' with FIELD_DELIMITER = 'NONE'
D) TYPE = 'AVRO' is required for any semi-structured data

<details><summary>Show Answer</summary>

**Answer: B) TYPE = 'JSON' (Snowflake can parse concatenated JSON documents natively without needing an outer array)**

Snowflake's JSON file format can parse a stream of concatenated JSON objects without requiring them to be wrapped in an array; STRIP_OUTER_ARRAY is only needed when the file has one outer array wrapping multiple objects that should be split. CSV and AVRO settings are unrelated to this scenario.
</details>

---

### Q46. When loading semi-structured data into a VARIANT column, which limitation applies to a single VARIANT value?

A) There is no size limit for a single VARIANT value
B) A single VARIANT value is limited to a maximum compressed size (16 MB by default)
C) VARIANT columns cannot store nested objects, only flat key-value pairs
D) VARIANT columns can only store arrays, not objects

<details><summary>Show Answer</summary>

**Answer: B) A single VARIANT value is limited to a maximum compressed size (16 MB by default)**

Individual VARIANT, OBJECT, or ARRAY values are subject to a maximum size limit (16 MB compressed). VARIANT fully supports nested structures including both objects and arrays, not just flat pairs.
</details>

---

### Q47. Which command allows you to view the definition of a view without needing SELECT privilege on the underlying base tables, assuming the view itself is properly granted?

A) SHOW VIEWS only, definitions are never visible
B) GET_DDL('VIEW', view_name), provided the caller has appropriate privileges on the view object
C) It's impossible to view a view's DDL without table-level access
D) DESCRIBE TABLE view_name

<details><summary>Show Answer</summary>

**Answer: B) GET_DDL('VIEW', view_name), provided the caller has appropriate privileges on the view object**

GET_DDL returns the SQL text used to create an object like a view, and access depends on privileges on the view itself, not on the underlying base tables (except for secure views, which restrict DDL visibility further). DESCRIBE TABLE returns column metadata, not the view's defining SQL.
</details>

---

### Q48. What is the primary difference between a Scoped URL and a Presigned URL when accessing files in an internal stage?

A) Scoped URLs never expire; Presigned URLs expire in 24 hours
B) A Scoped URL is only valid within the current Snowflake session/context, while a Presigned URL can be used outside Snowflake and has a configurable expiration
C) Presigned URLs require an active Snowflake session to work at all
D) There is no functional difference between them

<details><summary>Show Answer</summary>

**Answer: B) A Scoped URL is only valid within the current Snowflake session/context, while a Presigned URL can be used outside Snowflake and has a configurable expiration**

Scoped URLs are tied to the current user's session and encoded for temporary access from within Snowflake tools, whereas Presigned URLs can be shared and used independently outside Snowflake (e.g., in a browser) with an explicitly configured expiration time.
</details>

---

### Q49. A user needs to load a 500 GB file efficiently. What is the recommended file-sizing best practice for bulk loading via COPY INTO?

A) Use one massive single file for maximum throughput
B) Split files into roughly 100–250 MB compressed chunks to enable parallel loading across threads
C) File size has no impact on load performance
D) Files should always be exactly 16 MB to match the VARIANT limit

<details><summary>Show Answer</summary>

**Answer: B) Split files into roughly 100–250 MB compressed chunks to enable parallel loading across threads**

Snowflake recommends splitting large data sets into multiple compressed files in roughly the 100–250 MB range so that loading can be parallelized effectively across available compute threads, rather than using one giant file (which limits parallelism) or files sized to the unrelated VARIANT byte limit.
</details>

---

### Q50. Which of the following best describes how Snowflake handles concurrent DML transactions on the same table via row-level locking?

A) Snowflake uses table-level locks exclusively; no two transactions can touch the same table concurrently
B) Snowflake uses a form of optimistic concurrency at the partition/row level, allowing concurrent DML as long as they don't conflict on the same rows/partitions
C) Snowflake requires manual LOCK TABLE statements before any DML
D) Concurrent DML is not supported in Snowflake at all

<details><summary>Show Answer</summary>

**Answer: B) Snowflake uses a form of optimistic concurrency at the partition/row level, allowing concurrent DML as long as they don't conflict on the same rows/partitions**

Snowflake's transaction model allows concurrent DML statements to proceed against the same table as long as they don't modify overlapping rows/partitions, using a form of snapshot isolation with conflict detection, rather than blunt table-level locking or requiring explicit LOCK TABLE statements (which Snowflake does not use).
</details>

---

### Q51. What isolation level does Snowflake implement for its transactions?

A) Read Uncommitted
B) Read Committed (with some snapshot-isolation characteristics for statement consistency)
C) Serializable, strictly enforced for all statements
D) Repeatable Read only

<details><summary>Show Answer</summary>

**Answer: B) Read Committed (with some snapshot-isolation characteristics for statement consistency)**

Snowflake implements Read Committed isolation, where each statement sees a consistent snapshot of committed data as of the statement's start. It does not offer Read Uncommitted (dirty reads) or strict Serializable isolation as its default/only mode.
</details>

---

### Q52. Which of the following would cause a query to be denied purely due to network policy, even with correct credentials?

A) The user's IP address is not in the allowed IP list defined in the network policy attached to their user or account
B) The user's password has expired
C) The user's default role lacks USAGE on the warehouse
D) The virtual warehouse is suspended

<details><summary>Show Answer</summary>

**Answer: A) The user's IP address is not in the allowed IP list defined in the network policy attached to their user or account**

Network policies enforce IP allow/block lists at the account or user level; a mismatch blocks the connection at the network layer, before authentication or role/warehouse-related checks are even relevant to query execution. Password expiration, warehouse suspension, and role privileges are separate concerns from network policy filtering.
</details>

---

### Q53. Which authentication method allows federated single sign-on using an external identity provider?

A) Key pair authentication only
B) SAML 2.0-based SSO
C) OAuth exclusively, with no support for SAML
D) Basic username/password only, since Snowflake doesn't support SSO

<details><summary>Show Answer</summary>

**Answer: B) SAML 2.0-based SSO**

Snowflake supports federated authentication via SAML 2.0 with external identity providers (like Okta or ADFS) for single sign-on. Key pair auth and OAuth are valid but separate authentication mechanisms, and Snowflake does support SSO, contrary to option D.
</details>

---

### Q54. A user with the ACCOUNTADMIN role executes ALTER ACCOUNT SET NETWORK_POLICY. What is a risk of misconfiguring this at the account level?

A) It has no risk; account-level policies only affect new users
B) It could lock out all users, including administrators, if their IP addresses aren't included in the allow list
C) It only affects read-only queries
D) It automatically reverts after 24 hours if misconfigured

<details><summary>Show Answer</summary>

**Answer: B) It could lock out all users, including administrators, if their IP addresses aren't included in the allow list**

An account-level network policy applies broadly to all users (unless a more specific user-level policy overrides it), so an overly restrictive IP allow list can inadvertently lock out everyone, including admins, with no automatic reversion.
</details>

---

### Q55. Which statement about External Tables is accurate?

A) External tables physically store a full copy of the data inside Snowflake
B) External tables reference data files in external cloud storage and read metadata for query pruning without importing the data itself
C) External tables cannot be queried using standard SQL SELECT statements
D) External tables automatically convert all files to Snowflake's internal micro-partition format on creation

<details><summary>Show Answer</summary>

**Answer: B) External tables reference data files in external cloud storage and read metadata for query pruning without importing the data itself**

External tables provide a table-like SQL interface over data that physically remains in external storage (e.g., S3), using metadata for partition pruning, rather than duplicating the data into Snowflake's own micro-partition format.
</details>

---

### Q56. What refreshes the metadata of an external table when new files are added to the underlying cloud storage location?

A) It refreshes automatically at all times with zero configuration needed
B) Either auto-refresh via cloud storage event notifications, or a manual ALTER EXTERNAL TABLE ... REFRESH
C) External tables cannot be refreshed once created
D) Only dropping and recreating the external table updates metadata

<details><summary>Show Answer</summary>

**Answer: B) Either auto-refresh via cloud storage event notifications, or a manual ALTER EXTERNAL TABLE ... REFRESH**

External table metadata can be kept current either through configured auto-refresh (event notifications from cloud storage) or by manually issuing an ALTER EXTERNAL TABLE ... REFRESH command; it is not fully automatic without any configuration, nor does it require dropping and recreating the object.
</details>

---

### Q57. Which of the following correctly explains "pruning" in the context of Snowflake query execution?

A) Removing duplicate rows from result sets automatically
B) Skipping micro-partitions that cannot contain relevant data based on filter predicates and stored min/max metadata
C) Deleting old Time Travel versions of a table
D) Compressing columns more aggressively during storage

<details><summary>Show Answer</summary>

**Answer: B) Skipping micro-partitions that cannot contain relevant data based on filter predicates and stored min/max metadata**

Pruning uses the min/max metadata stored per micro-partition to skip reading partitions that can't possibly satisfy the query's filter conditions, improving performance. It is unrelated to deduplication, Time Travel version cleanup, or compression strategy.
</details>

---

### Q58. A table has a clustering key on column `order_date`. Queries filtering on `customer_id` show poor pruning. What is the most likely explanation?

A) Clustering keys automatically optimize pruning for all columns in the table
B) The clustering key only improves pruning for predicates aligned with the clustered column(s); customer_id isn't organized by the clustering key
C) Clustering keys are ignored by the query optimizer entirely
D) customer_id queries can never be pruned regardless of table design

<details><summary>Show Answer</summary>

**Answer: B) The clustering key only improves pruning for predicates aligned with the clustered column(s); customer_id isn't organized by the clustering key**

A clustering key co-locates rows by the clustered expression, benefiting queries that filter on that expression; it does not inherently improve pruning for unrelated columns like customer_id unless there happens to be a correlation, so unrelated filters see little benefit from that specific key.
</details>

---

### Q59. Which Snowflake edition is the minimum required to use Column-level Security (Dynamic Data Masking) and Row Access Policies?

A) Standard
B) Enterprise
C) Business Critical
D) Virtual Private Snowflake (VPS) only

<details><summary>Show Answer</summary>

**Answer: B) Enterprise**

Dynamic Data Masking and Row Access Policies are part of Snowflake's governance features available starting at the Enterprise edition and above, not included in the base Standard edition, and not exclusive to Business Critical or VPS tiers.
</details>

---

### Q60. Which Snowflake edition is required to guarantee Tri-Secret Secure (customer-managed keys combined with Snowflake-managed keys)?

A) Standard
B) Enterprise
C) Business Critical (or higher)
D) It's available on all editions equally

<details><summary>Show Answer</summary>

**Answer: C) Business Critical (or higher)**

Tri-Secret Secure, which combines a customer-managed key with Snowflake's own key for an additional layer of encryption control, is a Business Critical edition (and above) feature, not available on Standard or Enterprise.
</details>

---

### Q61. A pipe fails to auto-ingest new files even though the storage event notification is configured correctly. Which function helps diagnose recent pipe errors?

A) SYSTEM$PIPE_STATUS combined with COPY_HISTORY / PIPE_USAGE_HISTORY views
B) SHOW WAREHOUSES
C) SYSTEM$CLUSTERING_INFORMATION
D) VALIDATE_PIPE_LOAD() with no parameters

<details><summary>Show Answer</summary>

**Answer: A) SYSTEM$PIPE_STATUS combined with COPY_HISTORY / PIPE_USAGE_HISTORY views**

SYSTEM$PIPE_STATUS reports pipe execution state and pending file counts, while COPY_HISTORY (table function or ACCOUNT_USAGE view) reveals per-file load errors — together these are the standard diagnostic tools. SHOW WAREHOUSES and SYSTEM$CLUSTERING_INFORMATION are unrelated, and VALIDATE_PIPE_LOAD requires parameters like pipe name and time range.
</details>

---

### Q62. When defining a Snowpipe with `CREATE PIPE ... AS COPY INTO ...`, which property determines if the pipe listens for cloud storage event notifications automatically?

A) AUTO_INGEST = TRUE
B) MANUAL_TRIGGER = FALSE
C) EVENT_LISTENER = ENABLED
D) NOTIFICATION_INTEGRATION = REQUIRED (mandatory in all cases)

<details><summary>Show Answer</summary>

**Answer: A) AUTO_INGEST = TRUE**

Setting AUTO_INGEST = TRUE on pipe creation configures it to load files automatically based on cloud storage event notifications, rather than requiring manual REFRESH calls. The other listed property names/syntax are not valid pipe options.
</details>

---

### Q63. Which of the following statements correctly differentiates a Task's `SCHEDULE` parameter using CRON vs a fixed interval?

A) CRON expressions only work with Business Critical edition
B) A fixed interval (e.g., '5 MINUTE') runs periodically relative to the task's own creation/resume time, while CRON allows specifying exact calendar-based schedules (with timezone) such as "every weekday at 8am"
C) CRON and fixed interval scheduling behave identically in all respects
D) Tasks cannot use CRON expressions at all; only streams can

<details><summary>Show Answer</summary>

**Answer: B) A fixed interval (e.g., '5 MINUTE') runs periodically relative to the task's own creation/resume time, while CRON allows specifying exact calendar-based schedules (with timezone) such as "every weekday at 8am"</strong>**

Snowflake tasks support both a simple interval-based schedule and a CRON-style expression (with optional timezone) for more precise calendar scheduling, and CRON support isn't restricted to Business Critical edition, nor is CRON scheduling exclusive to streams (which don't have schedules at all).
</details>

---

### Q64. What is the significance of the SERVERLESS_TASK_HISTORY / task compute model when a task does not specify a WAREHOUSE?

A) The task fails immediately with an error requiring a warehouse
B) The task runs using Snowflake-managed serverless compute, automatically sized and billed accordingly
C) The task defaults to using the account's smallest available warehouse for free
D) Serverless tasks are not a supported feature

<details><summary>Show Answer</summary>

**Answer: B) The task runs using Snowflake-managed serverless compute, automatically sized and billed accordingly**

If a task is created without an explicit WAREHOUSE, Snowflake runs it on serverless compute that it automatically provisions and scales, billed based on actual usage — this doesn't cause an error, nor is it free, and it is a supported and common configuration.
</details>

---

### Q65. A user attempts `SELECT * FROM my_stream` outside of any transaction that would consume it. What happens to the stream's offset?

A) The offset always advances on any SELECT, consumed or not
B) The offset does NOT advance on a plain SELECT; it only advances when a DML statement consuming the stream is committed
C) The stream is dropped after a single SELECT
D) SELECT statements against streams are not permitted

<details><summary>Show Answer</summary>

**Answer: B) The offset does NOT advance on a plain SELECT; it only advances when a DML statement consuming the stream is committed**

Simply querying a stream with SELECT does not consume its changes or advance its offset — only a DML operation (such as INSERT/MERGE) that reads the stream and successfully commits will advance it, allowing repeatable reads until an actual consuming transaction occurs.
</details>

---

### Q66. Which type of stream should be used to also capture changes from an underlying external table?

A) Standard streams cannot be created on external tables at all
B) An external table stream, which supports INSERT-only change tracking on external table metadata refreshes
C) Only append-only streams work on any object type, including external tables, identically to standard tables
D) A stream can never be attached to an external table under any circumstances

<details><summary>Show Answer</summary>

**Answer: B) An external table stream, which supports INSERT-only change tracking on external table metadata refreshes</strong>**

Snowflake supports streams on external tables, but they behave as insert-only streams tracking new/refreshed file metadata, unlike full standard/append-only streams available on native tables that can capture inserts, updates, and deletes.
</details>

---

### Q67. Which storage cost optimization technique applies specifically to reducing Fail-safe costs for tables that don't need disaster recovery protection?

A) Increasing the Time Travel retention period
B) Using TRANSIENT or TEMPORARY table types instead of PERMANENT
C) Enabling automatic clustering
D) Enabling the Search Optimization Service

<details><summary>Show Answer</summary>

**Answer: B) Using TRANSIENT or TEMPORARY table types instead of PERMANENT</strong>**

Transient and temporary tables forgo the 7-day Fail-safe period entirely, reducing storage costs for data that doesn't require that level of disaster recovery, whereas increasing Time Travel retention would raise storage costs, and clustering/search optimization address performance, not Fail-safe cost.
</details>

---

### Q68. What is the effect of the `MIN_CLUSTER_COUNT` and `MAX_CLUSTER_COUNT` parameters both being set to the same value greater than 1 on a warehouse?

A) The warehouse behaves as a single-cluster warehouse
B) The warehouse runs in multi-cluster mode with a fixed (non-auto-scaling) number of clusters always active when running, for consistent concurrency handling
C) This configuration is invalid and will raise an error
D) The warehouse will randomly vary its cluster count regardless of the settings

<details><summary>Show Answer</summary>

**Answer: B) The warehouse runs in multi-cluster mode with a fixed (non-auto-scaling) number of clusters always active when running, for consistent concurrency handling</strong>**

Setting MIN and MAX cluster counts equal (and >1) creates a "maximized" multi-cluster warehouse that always runs that fixed number of clusters when active, useful for predictable high-concurrency workloads, rather than functioning as a single-cluster warehouse or being an invalid configuration.
</details>

---

### Q69. Which billing model applies to virtual warehouse compute in the standard (non-serverless) model?

A) Per-query flat fee regardless of duration
B) Per-second billing (after a minimum 60-second charge) based on warehouse size, only while running/active
C) A fixed monthly subscription per warehouse
D) Billing based solely on data volume scanned, not compute time

<details><summary>Show Answer</summary>

**Answer: B) Per-second billing (after a minimum 60-second charge) based on warehouse size, only while running/active</strong>**

Standard virtual warehouses bill per-second with a 60-second minimum each time they start, scaled by warehouse size (credits/hour), and only while the warehouse is actually running — not a flat per-query fee, fixed subscription, or based purely on data scanned.
</details>

---

### Q70. A warehouse is set with AUTO_SUSPEND = 60 and AUTO_RESUME = TRUE. A new query arrives 90 seconds after the warehouse suspended. What happens?

A) The query fails since the warehouse is suspended
B) The warehouse automatically resumes to process the new query, incurring the standard startup billing
C) The query queues indefinitely until manually resumed
D) AUTO_RESUME only works during business hours by default

<details><summary>Show Answer</summary>

**Answer: B) The warehouse automatically resumes to process the new query, incurring the standard startup billing</strong>**

With AUTO_RESUME enabled, a suspended warehouse automatically starts back up when a new query needs it, incurring normal startup billing (minimum charge applies), rather than failing the query or requiring manual intervention.
</details>

---

### Q71. What is the purpose of the `INITIALLY_SUSPENDED` parameter when creating a virtual warehouse?

A) It permanently disables the warehouse from ever starting
B) It creates the warehouse in a suspended state so it doesn't start consuming credits immediately upon creation
C) It forces the warehouse to run continuously without suspension
D) It only applies to multi-cluster warehouses

<details><summary>Show Answer</summary>

**Answer: B) It creates the warehouse in a suspended state so it doesn't start consuming credits immediately upon creation</strong>**

INITIALLY_SUSPENDED = TRUE creates the warehouse object without starting it, avoiding unnecessary credit consumption until it's actually needed, rather than permanently disabling it or forcing continuous operation.
</details>

---

### Q72. Which factor primarily determines the credit consumption rate (credits/hour) of a virtual warehouse?

A) The number of queries run per hour
B) The warehouse size (T-shirt size such as X-Small, Small, Medium, etc.), which doubles credit consumption per size increase
C) The number of databases the warehouse has access to
D) The total number of users assigned to the warehouse's role

<details><summary>Show Answer</summary>

**Answer: B) The warehouse size (T-shirt size such as X-Small, Small, Medium, etc.), which doubles credit consumption per size increase</strong>**

Each increase in warehouse size roughly doubles the credit consumption rate (e.g., X-Small=1 credit/hr, Small=2, Medium=4, and so on), independent of query count, number of accessible databases, or number of assigned users.
</details>

---

### Q73. In terms of query performance, when should you scale UP (increase warehouse size) rather than scale OUT (add more clusters)?

A) When facing high query concurrency with many simultaneous small/medium queries queuing
B) When individual queries are large/complex and slow due to insufficient compute per query, rather than a queuing problem
C) Scaling up and out solve identical problems and are interchangeable
D) Scaling up is never appropriate; only scaling out should be used

<details><summary>Show Answer</summary>

**Answer: B) When individual queries are large/complex and slow due to insufficient compute per query, rather than a queuing problem</strong>**

Scaling up (bigger warehouse) adds more compute resources to speed up individual complex/large queries, whereas scaling out (multi-cluster) is the right tool for handling high concurrency/queuing from many simultaneous smaller queries — the two solve different problems and aren't interchangeable.
</details>

---

### Q74. Which statement about the `USE SECONDARY ROLES` setting is accurate?

A) It has no effect on privilege evaluation
B) When set to ALL, it activates all roles granted to the user (in addition to the primary role) for privilege evaluation during the session
C) It only works for the ACCOUNTADMIN role
D) Secondary roles apply exclusively to DDL statements, never DML

<details><summary>Show Answer</summary>

**Answer: B) When set to ALL, it activates all roles granted to the user (in addition to the primary role) for privilege evaluation during the session</strong>**

USE SECONDARY ROLES ALL causes the session to consider privileges from every role granted to the user, not just the currently active primary role, which is useful for actions like DML across objects owned by different roles. It affects both DDL and DML contexts and isn't restricted to ACCOUNTADMIN.
</details>

---

### Q75. A role hierarchy has role A granted to role B, and role B granted to role C. If a user is granted only role C, which privileges can they exercise?

A) Only privileges directly granted to role C
B) Privileges from C, and inherited privileges from B and A due to the role hierarchy
C) Privileges from A only, since it's the top of the hierarchy
D) No privileges, since only directly-owned roles grant privileges

<details><summary>Show Answer</summary>

**Answer: B) Privileges from C, and inherited privileges from B and A due to the role hierarchy</strong>**

Snowflake roles inherit privileges downward through the hierarchy — if C is granted B, and B is granted A, then C effectively inherits everything granted to B and A as well, not just its own direct grants.
</details>

---

### Q76. What does the `CURRENT_AVAILABLE_ROLES()` function return in the context of a running session?

A) All roles that exist in the account, regardless of the current user
B) The set of roles available to the current session, considering the primary role and any activated secondary roles
C) Only the PUBLIC role
D) The roles owned by ACCOUNTADMIN exclusively

<details><summary>Show Answer</summary>

**Answer: B) The set of roles available to the current session, considering the primary role and any activated secondary roles</strong>**

CURRENT_AVAILABLE_ROLES() reflects which roles are usable in the current session context (primary plus any activated secondary roles), not every role that exists account-wide nor just PUBLIC or ACCOUNTADMIN's roles.
</details>

---

### Q77. Which of the following best explains "spillage" (local/remote disk spilling) in query execution, and its performance implication?

A) Spillage refers to data being copied between databases; it has no performance impact
B) Spillage occurs when a query's intermediate results exceed available memory and must be written to local or remote disk, which slows performance — often indicating a warehouse that's too small for the workload
C) Spillage is a normal part of every query and never indicates a problem
D) Spillage only affects DDL operations, not SELECT queries

<details><summary>Show Answer</summary>

**Answer: B) Spillage occurs when a query's intermediate results exceed available memory and must be written to local or remote disk, which slows performance — often indicating a warehouse that's too small for the workload</strong>**

When intermediate results don't fit in memory, Snowflake spills to local SSD and, if that's also insufficient, to remote storage — both progressively slower than memory — signaling that a larger warehouse (more memory/local disk) could improve performance; it's not unrelated to databases nor exclusive to DDL.
</details>

---

### Q78. Where in the Query Profile would you look to identify whether a query experienced significant remote disk spilling?

A) The "Bytes scanned" statistic only
B) The "Bytes spilled to remote storage" statistic in the query profile's execution details/statistics panel
C) The CREATE TABLE DDL history
D) The RESULT_SCAN output

<details><summary>Show Answer</summary>

**Answer: B) The "Bytes spilled to remote storage" statistic in the query profile's execution details/statistics panel</strong>**

The Query Profile explicitly surfaces spillage statistics (local and remote) as part of its execution details, distinct from bytes scanned (which measures data read, not memory overflow), and unrelated to DDL history or RESULT_SCAN.
</details>

---

### Q79. Which practice most directly reduces the risk of a "Cartesian product" style explosion in a join, causing excessive spillage?

A) Increasing the warehouse size only, without addressing the join logic
B) Ensuring proper join predicates/conditions are specified so rows aren't matched unintentionally many-to-many
C) Enabling Search Optimization Service
D) Enabling the result cache

<details><summary>Show Answer</summary>

**Answer: B) Ensuring proper join predicates/conditions are specified so rows aren't matched unintentionally many-to-many</strong>**

An unintended Cartesian product (or a poorly filtered many-to-many join) is a logic/design issue best fixed by correct join conditions; simply enlarging the warehouse addresses symptoms not the root cause, and neither Search Optimization Service nor the result cache is designed to fix join logic problems.
</details>

---

### Q80. Which statement about the ACCOUNT_USAGE schema versus the INFORMATION_SCHEMA is correct?

A) ACCOUNT_USAGE shows only currently active objects; INFORMATION_SCHEMA retains historical/dropped object data
B) ACCOUNT_USAGE retains history of dropped objects and has a longer retention window (up to 1 year) but has latency, while INFORMATION_SCHEMA is near real-time but typically limited to a shorter historical window and doesn't show dropped objects
C) They are functionally identical in every respect
D) INFORMATION_SCHEMA requires ACCOUNTADMIN; ACCOUNT_USAGE does not

<details><summary>Show Answer</summary>

**Answer: B) ACCOUNT_USAGE retains history of dropped objects and has a longer retention window (up to 1 year) but has latency, while INFORMATION_SCHEMA is near real-time but typically limited to a shorter historical window and doesn't show dropped objects</strong>**

ACCOUNT_USAGE views include dropped objects and provide up to a year of history, but with data latency of up to a few hours; INFORMATION_SCHEMA offers fresher, near real-time data but with more limited historical range and no visibility into dropped objects. Neither statement about identical behavior or the ACCOUNTADMIN requirement (option D, which is reversed/incorrect) is accurate.
</details>

---

### Q81. Which of these correctly ranks Snowflake warehouse sizes from smallest to largest (assuming standard T-shirt naming)?

A) Medium, Small, X-Small, Large
B) X-Small, Small, Medium, Large, X-Large
C) Small, X-Small, Large, Medium
D) Large, Medium, Small, X-Large

<details><summary>Show Answer</summary>

**Answer: B) X-Small, Small, Medium, Large, X-Large</strong>**

Snowflake warehouse sizes scale in order: X-Small, Small, Medium, Large, X-Large, 2X-Large, and so on up to 6X-Large, each roughly doubling compute (and credit consumption) over the previous size.
</details>

---

### Q82. A developer creates a temporary table with the same name as an existing permanent table in the same schema during their session. What happens when they query that table name?

A) An error is raised due to a naming conflict
B) The temporary table takes precedence for that session; the permanent table is unaffected and reappears once the session ends and the temp table is dropped
C) The permanent table is silently overwritten
D) Both tables are merged automatically

<details><summary>Show Answer</summary>

**Answer: B) The temporary table takes precedence for that session; the permanent table is unaffected and reappears once the session ends and the temp table is dropped</strong>**

Snowflake allows a session-scoped temporary table to shadow a permanent table of the same name for the duration of that session without error; the underlying permanent table remains completely untouched and becomes visible again once the temporary table is dropped or the session ends.
</details>

---

### Q83. Which of the following is true about a Temporary table's lifespan?

A) It persists indefinitely until manually dropped, just like a permanent table
B) It exists only for the duration of the session in which it was created and is automatically dropped when the session ends
C) It persists for exactly 24 hours regardless of session state
D) It requires an explicit DROP TABLE statement or it will remain forever

<details><summary>Show Answer</summary>

**Answer: B) It exists only for the duration of the session in which it was created and is automatically dropped when the session ends</strong>**

Temporary tables are inherently session-scoped: they are automatically purged when the creating session terminates, with no need for an explicit DROP, and they don't persist for a fixed 24-hour window or indefinitely like permanent tables.
</details>

---

### Q84. A cloned database includes a table with an active Stream. What happens to that stream in the clone?

A) The stream is cloned with its own independent offset matching the state at the point of cloning
B) Streams are never included when cloning a database
C) The cloned stream always starts with an empty/zero offset, ignoring source state
D) The clone shares the exact same stream object as the source, with changes to either affecting both

<details><summary>Show Answer</summary>

**Answer: A) The stream is cloned with its own independent offset matching the state at the point of cloning</strong>**

When a database/schema/table containing a stream is cloned, the stream object is also cloned along with its offset as of the cloning moment, becoming an independent object thereafter — not shared with the source and not reset to zero.
</details>

---

### Q85. Which statement correctly describes how Snowflake bills for cloud services layer compute?

A) Cloud services compute is always billed at full rate for every operation
B) Cloud services usage is billed only for the portion exceeding 10% of daily compute credit usage from warehouses (subject to change and account-level nuances), with typical light usage often not incurring extra charge
C) Cloud services compute is entirely and permanently free with no conditions
D) Cloud services usage is billed per query regardless of warehouse usage

<details><summary>Show Answer</summary>

**Answer: B) Cloud services usage is billed only for the portion exceeding 10% of daily compute credit usage from warehouses (subject to change and account-level nuances), with typical light usage often not incurring extra charge</strong>**

Snowflake has historically applied a policy where cloud services charges are only billed when they exceed 10% of that day's warehouse compute credits, meaning many accounts see effectively no separate cloud services charge under typical usage patterns — it is not a flat full-rate charge, unconditionally free, nor purely per-query.
</details>

---

### Q86. Which mechanism would you use to automatically pause and prevent runaway credit usage for a specific warehouse exceeding a budget?

A) A masking policy
B) A resource monitor assigned to that warehouse with an appropriate threshold and SUSPEND/SUSPEND_IMMEDIATE action
C) A row access policy
D) A network policy

<details><summary>Show Answer</summary>

**Answer: B) A resource monitor assigned to that warehouse with an appropriate threshold and SUSPEND/SUSPEND_IMMEDIATE action</strong>**

Resource monitors are purpose-built to track credit consumption against a defined quota and trigger actions like notification, suspend, or suspend-immediate when thresholds are crossed; masking policies, row access policies, and network policies address entirely different concerns (data visibility and network access, not billing).
</details>

---

### Q87. Which statement about "Data Sharing" and Time Travel interaction is correct?

A) Consumers of a share can use Time Travel on shared objects independently of the provider's retention settings
B) Time Travel is not available to consumers on shared database objects; they see only the current state as permitted by the share
C) Time Travel works identically for shares as for regular owned tables in every respect
D) Shares automatically extend Time Travel retention to 90 days for all consumers

<details><summary>Show Answer</summary>

**Answer: B) Time Travel is not available to consumers on shared database objects; they see only the current state as permitted by the share</strong>**

Data shares expose the current state of shared objects to consumers; Time Travel querying capability is not extended through a share, so consumers cannot independently time-travel on shared tables regardless of the provider's own retention configuration.
</details>

---

### Q88. What is the effect of the `COPY GRANTS` clause when creating or replacing a view/table?

A) It duplicates data along with the object
B) It preserves the existing privilege grants on the object being replaced, rather than resetting them to only the creator's default privileges
C) It copies grants from a completely different, unrelated object specified elsewhere
D) It has no effect and is deprecated syntax

<details><summary>Show Answer</summary>

**Answer: B) It preserves the existing privilege grants on the object being replaced, rather than resetting them to only the creator's default privileges</strong>**

COPY GRANTS ensures that when you CREATE OR REPLACE an object, the previously granted privileges on that object are retained on the new version, instead of being wiped and reset to just the default owner privileges — it has nothing to do with copying data or grants from an unrelated object.
</details>

---

### Q89. A user wants to test SQL logic against production-like data without risking changes to production. Which zero-copy cloning based practice is most efficient?

A) Manually re-exporting and re-importing all production data into a new database
B) Cloning the production database (or relevant schema/tables) into a dev/test database using CREATE DATABASE ... CLONE
C) Granting the developer direct write access to the production database
D) Using Time Travel exclusively without cloning, editing production directly and reverting later

<details><summary>Show Answer</summary>

**Answer: B) Cloning the production database (or relevant schema/tables) into a dev/test database using CREATE DATABASE ... CLONE</strong>**

Zero-copy cloning lets a full production-like environment be created almost instantly without duplicating storage upfront, providing a safe, isolated dev/test copy, unlike manually re-exporting/importing data (slow and costly), granting direct prod write access (risky), or editing production directly (extremely risky).
</details>

---

### Q90. Which statement about Object Tagging in Snowflake is correct?

A) Tags can only be applied to virtual warehouses, not tables or columns
B) Tags are key-value pair metadata objects that can be applied to various objects (tables, columns, warehouses, etc.) for governance, cost tracking, and classification purposes
C) Tags automatically enforce masking policies without any additional configuration
D) A tag can only ever have one possible value across the entire account

<details><summary>Show Answer</summary>

**Answer: B) Tags are key-value pair metadata objects that can be applied to various objects (tables, columns, warehouses, etc.) for governance, cost tracking, and classification purposes</strong>**

Object tagging supports attaching key-value metadata to a wide range of object types (not just warehouses) for purposes like cost attribution, data classification, and governance tracking; tags don't automatically enforce masking on their own (that requires separately linking a tag-based masking policy), and a tag can have many different allowed values.
</details>

---

### Q91. What is a Tag-Based Masking Policy, and how does it differ from applying a masking policy directly to a column?

A) There is no such thing as tag-based masking; masking policies must always be applied per-column manually
B) A tag-based masking policy is associated with a tag; any column with that tag automatically inherits the policy, simplifying governance at scale across many columns
C) Tag-based masking only works on entire tables, never individual columns
D) Tag-based masking policies cannot coexist with directly-applied column masking policies

<details><summary>Show Answer</summary>

**Answer: B) A tag-based masking policy is associated with a tag; any column with that tag automatically inherits the policy, simplifying governance at scale across many columns</strong>**

By associating a masking policy with a tag (e.g., "PII"), any column that has that tag applied automatically gets the masking behavior, avoiding the need to manually attach the policy to every individual column — a major efficiency gain for governance at scale; it does operate at the column level, not just whole tables.
</details>

---

### Q92. Which of the following correctly describes the relationship between a Database, Schema, and Table in Snowflake's object hierarchy?

A) Tables contain schemas, which contain databases
B) A database contains one or more schemas, and each schema contains objects like tables, views, and stages
C) Schemas and databases are the same object type with different names
D) A table can exist independently outside of any schema or database

<details><summary>Show Answer</summary>

**Answer: B) A database contains one or more schemas, and each schema contains objects like tables, views, and stages</strong>**

Snowflake's object hierarchy is Account > Database > Schema > Object (table, view, stage, etc.) — the reverse ordering in option A is incorrect, schemas and databases are distinct object types with different purposes, and every table must belong to a schema within a database.
</details>

---

### Q93. What happens to child objects (tables, views) when a schema is dropped, assuming they are all still within their Time Travel retention window?

A) They are permanently and immediately purged with no recovery possible
B) They can potentially be recovered via UNDROP SCHEMA, which restores the schema and its objects as they existed at drop time (within retention limits)
C) Child objects survive independently and remain queryable even after the parent schema is dropped
D) Dropping a schema has no effect on its child tables

<details><summary>Show Answer</summary>

**Answer: B) They can potentially be recovered via UNDROP SCHEMA, which restores the schema and its objects as they existed at drop time (within retention limits)</strong>**

Time Travel allows recovery of a dropped schema and its contained objects using UNDROP SCHEMA as long as the retention window hasn't expired; they are not immediately and irrecoverably purged, nor do child objects remain independently queryable after their parent schema is dropped.
</details>

---

### Q94. A developer needs to guarantee that a specific query always runs with a minimum amount of memory to avoid spilling, without permanently resizing the warehouse. Which session-level approach is most appropriate?

A) Resize the shared production warehouse permanently for all users
B) Temporarily switch to a larger warehouse for that specific session/query via USE WAREHOUSE, then switch back afterward
C) There is no way to control this; warehouse size is fixed once created
D) Use the RESULT_SCAN function to force more memory allocation

<details><summary>Show Answer</summary>

**Answer: B) Temporarily switch to a larger warehouse for that specific session/query via USE WAREHOUSE, then switch back afterward</strong>**

A common practical pattern is to switch the session's active warehouse to a larger one just for the demanding query, then switch back, avoiding a permanent resize that would affect all other users of the shared warehouse; RESULT_SCAN retrieves cached results and has no bearing on memory allocation.
</details>

---

### Q95. Which of the following is a valid use case specifically suited to Streams combined with Tasks, rather than a simple scheduled batch job alone?

A) Running a nightly full table reload regardless of whether data changed
B) Incrementally processing only new or changed rows detected since the last run, triggered on a schedule or via SYSTEM$STREAM_HAS_DATA condition
C) Creating a virtual warehouse
D) Rotating account credentials automatically

<details><summary>Show Answer</summary>

**Answer: B) Incrementally processing only new or changed rows detected since the last run, triggered on a schedule or via SYSTEM$STREAM_HAS_DATA condition</strong>**

The classic Streams + Tasks pattern enables efficient, incremental (change-data-capture-driven) processing pipelines, only doing work when relevant changes exist, unlike a blind full nightly reload that reprocesses everything regardless of change; warehouse creation and credential rotation are unrelated administrative actions.
</details>

---

### Q96. Which statement about the `VALIDATION_MODE` option in COPY INTO is correct?

A) It permanently loads data and validates it after the fact
B) It performs a dry-run validation of the load without actually inserting data, returning errors that would occur
C) It validates only file compression, not row-level content
D) It is required on every COPY INTO statement by default

<details><summary>Show Answer</summary>

**Answer: B) It performs a dry-run validation of the load without actually inserting data, returning errors that would occur</strong>**

VALIDATION_MODE lets you simulate a COPY INTO load (checking for errors like malformed rows up to a specified count or all rows) without committing any actual data to the table, useful for pre-flight checks; it is optional, not required by default, and it validates data content, not merely compression.
</details>

---

### Q97. Which statement correctly describes how Snowflake handles semi-structured data type inference when querying a VARIANT column with dot notation, e.g., `col:field::STRING`?

A) The `::STRING` cast is unnecessary since VARIANT always auto-converts to the correct type in output
B) The `::` operator explicitly casts the extracted VARIANT sub-element to a specific SQL type since VARIANT paths return VARIANT type by default
C) Dot notation is invalid syntax and only bracket notation is supported
D) VARIANT columns cannot use dot notation for nested field access

<details><summary>Show Answer</summary>

**Answer: B) The `::` operator explicitly casts the extracted VARIANT sub-element to a specific SQL type since VARIANT paths return VARIANT type by default</strong>**

Accessing a nested field via dot or bracket notation on a VARIANT column returns another VARIANT value by default; explicit casting with `::` (e.g., `::STRING`) is needed to convert it into a native SQL type for proper display/comparison — dot notation is valid syntax, and casting is not automatically inferred without the explicit cast.
</details>

---

### Q98. Which of the following best explains why Snowflake recommends avoiding excessively small (X-Small) warehouses for very large, complex transformation queries?

A) X-Small warehouses cannot connect to external stages
B) Limited compute/memory on smaller warehouses can lead to excessive spilling to disk and longer execution time for large, memory-intensive operations
C) X-Small warehouses have a hard row-count limit per query
D) Small warehouses cannot execute DML statements at all

<details><summary>Show Answer</summary>

**Answer: B) Limited compute/memory on smaller warehouses can lead to excessive spilling to disk and longer execution time for large, memory-intensive operations</strong>**

Smaller warehouses simply have less available memory and compute, so large/complex queries (big joins, sorts, aggregations) are more likely to spill to local or remote disk, increasing runtime; there's no special restriction preventing X-Small warehouses from connecting to stages, no hard row-count limit tied to size, and DML is fully supported on any warehouse size.
</details>

---

### Q99. A Snowflake account administrator wants a complete audit trail of all login attempts, including failures. Which view provides this?

A) SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
B) SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY
C) SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
D) SNOWFLAKE.ACCOUNT_USAGE.GRANTS_TO_ROLES

<details><summary>Show Answer</summary>

**Answer: B) SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY</strong>**

LOGIN_HISTORY records login attempts including successes and failures with details like client IP and error codes, while QUERY_HISTORY tracks executed queries, WAREHOUSE_METERING_HISTORY tracks compute credit usage, and GRANTS_TO_ROLES tracks privilege grants — none of which capture login attempt auditing.
</details>

---

### Q100. Which best summarizes the overall value proposition of Snowflake's separation of storage and compute architecture in relation to concurrency?

A) All queries must share a single compute resource, causing contention as usage grows
B) Independent virtual warehouses can be scaled and provisioned separately per workload, allowing concurrent workloads (e.g., ETL and BI reporting) to run without competing for the same compute resources against a shared storage layer
C) Storage and compute must always scale together in fixed proportions
D) Concurrency is only achievable by manually partitioning the underlying data files per team

<details><summary>Show Answer</summary>

**Answer: B) Independent virtual warehouses can be scaled and provisioned separately per workload, allowing concurrent workloads (e.g., ETL and BI reporting) to run without competing for the same compute resources against a shared storage layer</strong>**

Because storage and compute are decoupled, multiple independent virtual warehouses can be spun up for different teams/workloads, all reading and writing the same underlying storage without resource contention between them — unlike a shared single-compute model, and without needing storage/compute to scale in lockstep or manual data partitioning per team.
</details>

---

## Summary
100 hard-level SnowPro Core practice questions covering: warehouse behavior & scaling, micro-partitions & clustering, Time Travel & Fail-safe, zero-copy cloning, architecture layers, caching, security (masking, row access policies, RBAC, network policies), data sharing & reader accounts, Snowpipe & continuous loading, Streams & Tasks, performance tuning (Search Optimization, Query Acceleration, materialized views), cost optimization, and governance (tagging).

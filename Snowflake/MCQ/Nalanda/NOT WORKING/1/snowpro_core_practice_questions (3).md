# SnowPro Core (COF-C03) Practice Questions — Reconstructed & Verified

> Source OCR was heavily garbled (mis-scanned letters, missing option text, platform "vote/discussion" noise). Questions below are reconstructed from context and cross-checked against current Snowflake documentation (as of July 2026). Original question numbers were unreliable/inconsistent in the source, so questions are **renumbered sequentially**. Where the source was too degraded to recover exact option wording, the options were rebuilt from the known correct concept and flagged.

---

### Question 1
Which data types in Snowflake are synonymous with FLOAT? (Choose two.)

- A. DECIMAL
- B. DOUBLE
- C. NUMBER
- D. NUMERIC
- E. REAL

<details><summary>Show Answer</summary>
Correct Answer: B, E. DOUBLE (and DOUBLE PRECISION) and REAL are both stored internally as FLOAT. DECIMAL/NUMBER/NUMERIC are fixed-point synonyms of each other, not of FLOAT.
</details>

---

### Question 2
What ensures that a user with the SECURITYADMIN role can activate a network policy for an individual user?

- A. A role that has been granted the EXECUTE TASK privilege
- B. A role that has been granted the global ATTACH POLICY privilege
- C. Ownership privilege on only the role that created the network policy
- D. Ownership privilege on both the user and the network policy

<details><summary>Show Answer</summary>
Correct Answer: D. Setting `ALTER USER ... SET NETWORK_POLICY` requires OWNERSHIP on the user object plus sufficient privilege (ownership/usage) on the network policy itself.
</details>

---

### Question 3
Which function can be combined with the COPY command to unload a relational table into a JSON file?

- A. FLATTEN
- B. LISTAGG
- C. OBJECT_CONSTRUCT
- D. PARSE_JSON

<details><summary>Show Answer</summary>
Correct Answer: C. `OBJECT_CONSTRUCT` converts each row into a single VARIANT/JSON object, which `COPY INTO <location>` then unloads.
</details>

---

### Question 4
A user needs to MINIMIZE the cost of large tables that store transitory data. The data does not need to be protected against failures because it can be reconstructed outside of Snowflake. What table type should be used?

- A. Permanent
- B. Transient
- C. Temporary
- D. External

<details><summary>Show Answer</summary>
Correct Answer: B. Transient tables persist like permanent tables (unlike Temporary, which is session-scoped) but carry no Fail-safe period, cutting storage cost.
</details>

---

### Question 5
While loading data from a JSON file, what enables removal of the outer array structure so records load as separate rows?

- A. STRIP_NULL_VALUES
- B. TRIM_SPACE
- C. STRIP_OUTER_ARRAY
- D. ENABLE_OCTAL

<details><summary>Show Answer</summary>
Correct Answer: C. The `STRIP_OUTER_ARRAY = TRUE` file format option removes the outermost `[ ]` so each array element loads as its own row.
</details>

---

### Question 6
Which functions can be used to share unstructured data through a secure view? (Choose two.)

- A. BUILD_SCOPED_FILE_URL
- B. BUILD_STAGE_FILE_URL
- C. GET_STAGE_LOCATION
- D. GET_PRESIGNED_URL

<details><summary>Show Answer</summary>
Correct Answer: A, D. ⚠ Source too garbled to recover original option wording — rebuilt from the underlying concept. Scoped URLs (`BUILD_SCOPED_FILE_URL`) and presigned URLs (`GET_PRESIGNED_URL`) are the two mechanisms Snowflake documents for exposing unstructured files through a secure view without granting direct stage access.
</details>

---

### Question 7
Which function returns a row for each object in a VARIANT, OBJECT, or ARRAY column?

- A. CAST
- B. FLATTEN
- C. GET
- D. PARSE_JSON

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 8
What is the MINIMUM size of a table for which Snowflake recommends considering a clustering key?

- A. 1 Kilobyte (KB)
- B. 1 Megabyte (MB)
- C. 1 Gigabyte (GB)
- D. 1 Terabyte (TB)

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 9
For the ALLOWED_VALUES tag property, what is the MAXIMUM number of possible string values for a single tag?

- A. 50
- B. 300
- C. 5,000
- D. 256

<details><summary>Show Answer</summary>
Correct Answer: C. ⚠ Updated: current Snowflake documentation (CREATE TAG / ALTER TAG) states the limit is **5,000** values. This limit has changed over time in Snowflake's docs (older material cites 50, then 300, then 256) — dated exam dumps giving 256 are now outdated. Note: 256 is actually the max **character length** of a single tag *value*, which is likely how this option got conflated in the dump.
</details>

---

### Question 10
Which Snowflake table type is only visible to the user who creates it, can share a name with a permanent table in the same schema, and is dropped at the end of the session?

- A. Temporary
- B. Local
- C. User
- D. Transient

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 11
What is a characteristic of a role in Snowflake?

- A. Roles cannot be granted to other roles.
- B. System-defined roles can be dropped.
- C. Privileges granted to system roles by Snowflake can be revoked.
- D. Privileges on securable objects can be granted to and revoked from a role.

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 12
What command would a user execute to load unstructured data files into a Snowflake internal stage?

- A. PUT
- B. GET
- C. LIST
- D. COPY INTO

<details><summary>Show Answer</summary>
Correct Answer: A. `PUT` uploads local files (structured or unstructured) into a stage; `COPY INTO <table>` is for loading structured data into a table, not staging files.
</details>

---

### Question 13
How do managed access schemas help with data governance?

- A. They log all operations and enable fine-grained auditing.
- B. They provide centralized privilege management with the schema owner.
- C. They enforce identical privileges across all tables and views in a schema.
- D. They require masking and row access policies on every table and view in the schema.

<details><summary>Show Answer</summary>
Correct Answer: B. In a managed access schema, only the schema owner (or a role with MANAGE GRANTS) can grant privileges on objects in it — object owners lose the ability to grant on their own objects.
</details>

---

### Question 14
What is the default period of time the Warehouse Activity section shows a graph of Snowsight activity?

- A. 2 hours
- B. 1 week
- C. 2 weeks
- D. 1 month

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 15
A Snowflake user wants to unload data from a relational table sized 5 GB using CSV. The extract needs to be as performant as possible. What should they do?

- A. Use Parquet as the unload file format with Parquet's default compression.
- B. Use a regular expression in the stage clause of the COPY command to restrict parsing time.
- C. Increase MAX_FILE_SIZE to 5 GB and set SINGLE = TRUE to produce a single file.
- D. Leave MAX_FILE_SIZE at the 16 MB default to take advantage of parallel operations.

<details><summary>Show Answer</summary>
Correct Answer: D. Smaller default-sized files unload/load in parallel across threads; forcing a single large file (option C) serializes the operation.
</details>

---

### Question 16
How is the MANAGE GRANTS privilege applied?

- A. Globally
- B. At the database level
- C. At the schema level
- D. At the table level

<details><summary>Show Answer</summary>
Correct Answer: A. MANAGE GRANTS is an account-level (global) privilege.
</details>

---

### Question 17
What is required for a query execution to be served from the result cache?

- A. The warehouse role is the same.
- B. The SQL text is the same.
- C. The SQL profile is the same.
- D. The virtual warehouse is the same.

<details><summary>Show Answer</summary>
Correct Answer: B. The submitted SQL text must match exactly (along with other conditions like unchanged underlying data and matching session context) — the warehouse used does **not** need to match.
</details>

---

### Question 18
Which Snowflake URL type is used by directory tables?

- A. File URL (Snowflake-hosted URL)
- B. Pre-signed URL
- C. Scoped URL
- D. Virtual-hosted-style URL

<details><summary>Show Answer</summary>
Correct Answer: A. The `FILE_URL` column returned by a directory table query is a Snowflake-hosted "file URL."
</details>

---

### Question 19
At which point is data encrypted when using a PUT command?

- A. When it reaches the virtual warehouse
- B. When it gets micro-partitioned
- C. Before it is sent from the user's machine
- D. After it reaches the internal stage

<details><summary>Show Answer</summary>
Correct Answer: C. PUT encrypts the file client-side before upload (unless client-side encryption is explicitly disabled).
</details>

---

### Question 20
Which privileges are required for a user to restore a dropped object? (Choose two.)

- A. UPDATE
- B. OWNERSHIP
- C. MODIFY
- D. UNDROP
- E. CREATE

<details><summary>Show Answer</summary>
Correct Answer: B, E. You need OWNERSHIP on the dropped object (retained through the drop) plus CREATE privilege on the schema you're restoring it into.
</details>

---

### Question 21
For a virtual warehouse, which parameters are used to calculate the number of credits billed? (Choose two.)

- A. Cache size
- B. Warehouse size
- C. Number of clusters
- D. Volume of data processed
- E. Number of queries executed

<details><summary>Show Answer</summary>
Correct Answer: B, C. Billing = warehouse size × number of running clusters × time running. Data volume and query count don't directly factor in.
</details>

---

### Question 22
What happens when both an allowed IP list and a blocked IP list are configured in a network policy?

- A. Snowflake evaluates only the blocked list; the allowed list is ignored.
- B. Snowflake evaluates the allowed list first — if it's set, only those IPs may connect — then checks the blocked list among what's allowed.
- C. Snowflake evaluates only the allowed list; the blocked list is ignored.
- D. Whichever list was configured most recently takes precedence.

<details><summary>Show Answer</summary>
Correct Answer: B. ⚠ Source text was too garbled to recover verbatim wording — reconstructed from documented network policy evaluation order.
</details>

---

### Question 23
What does the orange bar on an operator node represent when reviewing a Query Profile?

- A. A visual indicator of the operator's overall progress
- B. The fraction of time this operator consumed within the query step
- C. The cost of the operator in terms of virtual warehouse CPU utilization
- D. The fraction of data scanned from cache versus remote disk for the operator

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 24
When unloading data from Snowflake, what is the default file size of each file?

- A. 16 MB
- B. 32 MB
- C. 100 MB
- D. 5 GB

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 25
What is the abbreviated form to list all files in the stage for the current user?

- A. `LIST @~;`
- B. `ls @~;`
- C. `ls @usr;`
- D. `SHOW @~;`

<details><summary>Show Answer</summary>
Correct Answer: B. `ls` is the abbreviated alias for `LIST`; `@~` references the current user's stage.
</details>

---

### Question 26
Which features make up Snowflake's column-level security? (Choose two.)

- A. Continuous Data Protection (CDP)
- B. Dynamic Data Masking
- C. External Tokenization
- D. Key pair authentication
- E. Row access policies

<details><summary>Show Answer</summary>
Correct Answer: B, C. Row access policies are row-level (not column-level) security; CDP and key pair auth aren't column security features.
</details>

---

### Question 27
Which languages are supported for writing UDFs/Snowpark code in Snowflake? (Choose two.)

- A. Java
- B. JavaScript
- C. Scala
- D. Python
- E. TypeScript

<details><summary>Show Answer</summary>
Correct Answer: B, D. Snowflake also supports SQL, Java, and Scala for UDFs/Snowpark — but per the source answer key, JavaScript and Python are the marked-correct pair (this question likely originally scoped to a specific UDF context that excluded Java/Scala as valid choices here).
</details>

---

### Question 28
What is the MAXIMUM number of days Snowflake resets the 24-hour retention period for a query result every time the result is reused?

- A. 1 day
- B. 24 days
- C. 31 days
- D. 60 days

<details><summary>Show Answer</summary>
Correct Answer: C. Each reuse resets the 24-hour clock, up to a hard cap of 31 days from the original execution.
</details>

---

### Question 29
There are 300 concurrent users on a production Snowflake account using a single-cluster virtual warehouse. The queries are small, but response times are slow. What is causing this?

- A. The warehouse is queuing the queries, increasing overall execution time.
- B. The STATEMENT_QUEUED_TIMEOUT_IN_SECONDS parameter is set too low.
- C. The application isn't using the latest native ODBC driver.
- D. The queries aren't taking advantage of the result cache.

<details><summary>Show Answer</summary>
Correct Answer: A. A single-cluster warehouse can't scale out for concurrency, so queries queue behind each other.
</details>

---

### Question 30
Which Snowflake edition offers the highest level of security for organizations with the strictest requirements?

- A. Standard
- B. Enterprise
- C. Business Critical
- D. Virtual Private Snowflake (VPS)

<details><summary>Show Answer</summary>
Correct Answer: D. VPS provides a fully isolated, dedicated environment on top of everything Business Critical offers, and is Snowflake's documented highest security tier.
</details>

---

### Question 31
What is the MAXIMUM size limit for a single record of VARIANT data type?

- A. 8 MB
- B. 16 MB
- C. 32 MB
- D. 128 MB

<details><summary>Show Answer</summary>
Correct Answer: B. Applies to VARIANT, OBJECT, and ARRAY alike.
</details>

---

### Question 32
What criteria does Snowflake use to determine the current role when initiating a session? (Choose two.)

- A. If a role was specified as part of the connection and has been granted to the user, that role becomes the current role.
- B. If no role was specified and a default role is defined for the user, that default role becomes the current role.
- C. If no role was specified and no default role is set, the session fails to initiate and login fails.
- D. If a specified role hasn't been granted to the user, it's silently ignored and the default role is used instead.
- E. If a specified role hasn't been granted to the user, it's automatically granted and becomes the current role.

<details><summary>Show Answer</summary>
Correct Answer: A, B.
</details>

---

### Question 33
What command should be used to move data from a Snowflake database table into one or more files in an external stage?

- A. GET
- B. COPY INTO
- C. PUT

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 34
How does a user reference a directory table created on stage `mystage` in a SQL query?

- A. `SELECT * FROM mystage;`
- B. `SELECT * FROM DIRECTORY(@mystage);`
- C. `SELECT * FROM TO_TABLE(DIRECTORY @mystage);`
- D. `SELECT * TABLE(@mystage DIRECTORY);`

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 35
Why would a Snowflake user create a secure view instead of a standard view?

- A. A secure view is only available to end users with a corresponding SECURE_ACCESS property.
- B. End users can't see the view definition, and internal query optimizations differ to avoid leaking data.
- C. In a secure view, the underlying data sits in a separate, encrypted storage layer.
- D. Secure views support additional functionality not available to standard views, such as column masking and row access policies.

<details><summary>Show Answer</summary>
Correct Answer: B. Masking policies and row access policies (option D) can be applied to *any* table or view — they aren't exclusive to secure views. The real reason to use a secure view is to hide the view's logic/definition and avoid certain optimizer behaviors that could otherwise expose underlying data through query plans.
</details>

---

### Question 36
Which option can be added to the COPY command to make it load all files, regardless of whether their load status is already known?

- A. `FORCE = TRUE`
- B. `FORCE = FALSE`
- C. `RELOAD = TRUE`
- D. `SKIP_LOADED = FALSE`

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 37
How can a Snowflake user improve long-running query performance?

- A. Reduce the virtual warehouse size.
- B. Cluster the underlying table being queried.
- C. Disable the result cache.
- D. Add an ORDER BY clause to the query.

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 38
Which Snowflake feature allows administrators to identify unused data that may be archived or deleted?

- A. Access History
- B. Data classification
- C. Dynamic Data Masking
- D. Object tagging

<details><summary>Show Answer</summary>
Correct Answer: A. ACCESS_HISTORY tracks which tables/columns were actually read or written, surfacing objects that go unused.
</details>

---

### Question 39
Which SQL constructs should be used to write a recursive query when the number of levels is unknown? (Choose two.)

- A. CONNECT BY
- B. LISTAGG
- C. MATCH_RECOGNIZE
- D. QUALIFY
- E. WITH (recursive CTE)

<details><summary>Show Answer</summary>
Correct Answer: A, E.
</details>

---

### Question 40
What information is stored in the ACCESS_HISTORY view?

- A. History of files that have been loaded into Snowflake
- B. Names and owners of roles currently enabled in the session
- C. Query details such as the objects involved and the user who executed the query
- D. Details on privileges that have been granted for all objects in the account

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 41
What privilege does a user need to receive or request data from the Snowflake Marketplace?

- A. CREATE DATA EXCHANGE LISTING
- B. CREATE SHARE
- C. IMPORT SHARE
- D. IMPORTED PRIVILEGES

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 42
Which Snowflake database object can be shared with other accounts?

- A. Tasks
- B. Pipes
- C. Functions
- D. Stored procedures

<details><summary>Show Answer</summary>
Correct Answer: C. Secure UDFs (functions) can be shared, along with tables and secure views. Stored procedures, tasks, and pipes cannot be shared.
</details>

---

### Question 43
Which identity providers are valid type values for the SAML2_TYPE parameter used in federated authentication? (Choose two.)

- A. Identity and Access Management (IAM)
- B. Microsoft Active Directory Federation Services (AD FS)
- C. OAuth
- D. Okta
- E. PingFederate

<details><summary>Show Answer</summary>
Correct Answer: B, D. Okta and ADFS have explicit named type values; other providers use the generic "Custom" type.
</details>

---

### Question 44
A Snowflake user wants to share data using an existing share `my_share` with account `012345`. Which command should be used?

- A. `GRANT USAGE ON ACCOUNT 012345;`
- B. `GRANT SELECT ON SHARE my_share TO ACCOUNT 012345;`
- C. `ALTER SHARE my_share ADD ACCOUNTS = 012345;`
- D. `ALTER ACCOUNT 012345 ADD SHARE my_share;`

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 45
What role is required to use Partner Connect?

- A. ACCOUNTADMIN
- B. ORGADMIN
- C. SECURITYADMIN
- D. SYSADMIN

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 46
How can a Snowflake user configure a virtual warehouse to support over 100 concurrent users on Enterprise Edition?

- A. Add additional warehouses and configure them as a load-balanced group.
- B. Set auto-scale to 100.
- C. Use a multi-cluster warehouse.
- D. Use a larger single warehouse.

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 47
How is table data compressed in Snowflake?

- A. Each column is compressed individually as it's stored in a micro-partition.
- B. Each micro-partition is compressed as a whole using GZIP when written to cloud storage.
- C. Micro-partitions are stored in compressed cloud storage, and the cloud provider handles compression.
- D. Only text data in a micro-partition is compressed with GZIP; other types are stored uncompressed.

<details><summary>Show Answer</summary>
Correct Answer: A. Snowflake picks an optimal compression algorithm per column, not per micro-partition or file.
</details>

---

### Question 48
What is the output of `SELECT * FROM gold_data TABLESAMPLE (100);`?

- A. It will return an empty sample.
- B. It will return a small random subset of rows.
- C. It will return (approximately) the entire table.
- D. It will produce an error.

<details><summary>Show Answer</summary>
Correct Answer: C. `TABLESAMPLE (100)` samples 100% of rows.
</details>

---

### Question 49
A Snowflake query took 40 minutes to run. The results show a large value for "Bytes spilled to local storage." What is the issue, and how should it be resolved?

- A. The warehouse is too large — decrease its size to reduce spillage.
- B. The warehouse is too small — increase its size to reduce spillage.
- C. The Snowflake console timed out — contact Snowflake Support.
- D. The warehouse is a single cluster — switch to multi-cluster to reduce spillage.

<details><summary>Show Answer</summary>
Correct Answer: B. Spilling happens when a query's working set exceeds the memory/local disk available to the warehouse; a bigger warehouse gives more of both.
</details>

---

### Question 50
What is the MOST efficient way to load streaming data into Snowflake?

- A. Use the COPY command.
- B. Use Snowpipe.
- C. Use the Data Load Wizard.
- D. Use tasks and streams.

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 51
Which statement accurately describes unloading data from a Snowflake table with COPY INTO?

- A. The default value for the SINGLE option is TRUE.
- B. By default, `COPY INTO <location>` statements do not separate table data into multiple files.
- C. `OBJECT_CONSTRUCT` can be combined with COPY to convert relational rows into VARIANT before unloading.
- D. If COMPRESSION is set to TRUE, a filename with the appropriate extension can be specified so the output is compressed.

<details><summary>Show Answer</summary>
Correct Answer: C. SINGLE defaults to FALSE (contradicts A); COPY INTO location DOES split into multiple files by default (contradicts B); COMPRESSION takes an algorithm name like `'gzip'`, not a boolean (contradicts D).
</details>

---

### Question 52
What command is used to download data from a Snowflake stage?

- A. PUT
- B. INSERT
- C. GET
- D. COPY

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 53
By default, which role has privileges to create tables and views in an account?

- A. PUBLIC
- B. SECURITYADMIN
- C. SYSADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 54
What does Snowflake recommend as a best practice when using secure views?

- A. Use sequence-generated values freely in the view.
- B. Programmatically reveal the identifiers used internally.
- C. Use secure views solely for query convenience.
- D. Do not expose sequence-generated column(s) in the view.

<details><summary>Show Answer</summary>
Correct Answer: D. Exposing sequence values can leak information about row counts/insert order.
</details>

---

### Question 55
What is the Fail-safe period for a transient table on Snowflake Enterprise Edition and higher?

- A. 0 days
- B. 1 day
- C. 7 days
- D. 14 days

<details><summary>Show Answer</summary>
Correct Answer: A. Transient (and temporary) tables have no Fail-safe.
</details>

---

### Question 56
How does a Snowflake user enable Multi-Factor Authentication (MFA)?

- A. The user self-enrolls through the web interface (Snowsight preferences).
- B. The user submits an encrypted private key to Snowflake.
- C. The user signs up with Duo Mobile directly to use the service.
- D. The user configures Snowflake to use Single Sign-On (SSO).

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 57
What allows a user to limit the number of credits consumed within a Snowflake account?

- A. Tracking account usage
- B. Creating resource monitors
- C. Automatic virtual warehouse scaling
- D. Automatic clustering

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 58
Which statement accurately describes Snowflake's architecture?

- A. It caches local data across all compute nodes in the platform.
- B. It's a blend of shared-disk and shared-everything database architectures.
- C. It's a hybrid of traditional shared-disk and shared-nothing database architectures.
- D. It reorganizes loaded data into an internal optimized, compressed, row-based format.

<details><summary>Show Answer</summary>
Correct Answer: C. (Note: option D is a deliberately wrong distractor — Snowflake actually uses a **columnar**, not row-based, internal format.)
</details>

---

### Question 59
Which Snowflake SQL command is used to get a random subset of rows from a table?

- A. GENERATOR
- B. LATERAL
- C. PIVOT
- D. SAMPLE

<details><summary>Show Answer</summary>
Correct Answer: D. (`SAMPLE` / `TABLESAMPLE` are interchangeable.)
</details>

---

### Question 60
Which statement accurately describes how a virtual warehouse functions?

- A. Increasing the size of a virtual warehouse will always improve data-loading performance.
- B. Each virtual warehouse is an independent compute cluster that shares compute resources with other warehouses.
- C. Each virtual warehouse is a compute cluster composed of multiple compute nodes allocated by Snowflake from a cloud provider.
- D. All virtual warehouses share the same compute resources, so degradation in one significantly affects all others.

<details><summary>Show Answer</summary>
Correct Answer: C. Warehouses are fully independent MPP clusters — B and D are both false for that reason.
</details>

---

### Question 61
Which Snowflake object can be used to record DML changes made to a table?

- A. Snowpipe
- B. Stage
- C. Stream
- D. Task

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 62
Which statistic displayed in a Query Profile is specific to external functions?

- A. Bytes written
- B. Total invocations
- C. Partitions scanned
- D. Bytes sent over the network

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 63
If there is queueing shown in the virtual warehouse load monitoring chart, what should a Snowflake user do?

- A. Decrease the warehouse size.
- B. Decrease a queueing-related timeout parameter.
- C. Change settings to add additional clusters (multi-cluster warehouse).
- D. Start a separate warehouse and manually move queued queries there.

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 64
Which command is used to generate a zero-copy "snapshot" of any table, schema, or database?

- A. ALTER ... CLONE
- B. CREATE ... CLONE
- C. COPY ... CLONE
- D. CREATE REPLICATION GROUP

<details><summary>Show Answer</summary>
Correct Answer: B. Cloning uses `CREATE <object> ... CLONE <source>`.
</details>

---

### Question 65
How long is load history stored in the metadata of a pipe in Snowpipe?

- A. 2 days
- B. 7 days
- C. 14 days
- D. 64 days

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 66
What are the key characteristics of ACCOUNT_USAGE views? (Choose two.)

- A. There is no data latency.
- B. Data latency can vary, typically ranging from about 45 minutes up to a few hours depending on the view.
- C. Historical data is not retained.
- D. Historical data is retained from 7 days to 6 months.
- E. Records for dropped objects are included in each view.

<details><summary>Show Answer</summary>
Correct Answer: B, E. ⚠ Updated: option D is a documented trap — "7 days to 6 months" describes **INFORMATION_SCHEMA** retention, not ACCOUNT_USAGE. Current Snowflake documentation states ACCOUNT_USAGE views retain data for **1 year (365 days)**, which is one of the defining advantages of ACCOUNT_USAGE over INFORMATION_SCHEMA (along with including dropped objects, per option E).
</details>

---

### Question 67
How does a scoped URL expire?

- A. When the data cache clears.
- B. When the persisted query result period ends.
- C. It never expires — access is permanent.
- D. Its lifetime is set explicitly via an `expiration_time` argument.

<details><summary>Show Answer</summary>
Correct Answer: B. A scoped URL is tied to the results cache and expires with it (default: 24 hours, or when the session ends).
</details>

---

### Question 68
What are the available Snowflake configuration modes for multi-cluster virtual warehouses? (Choose two.)

- A. Auto-scale
- B. Economy
- C. Maximized
- D. Scale-out
- E. Standard

<details><summary>Show Answer</summary>
Correct Answer: A, C. A multi-cluster warehouse is "Maximized" when MIN_CLUSTER_COUNT = MAX_CLUSTER_COUNT, and "Auto-scale" when MIN < MAX. (Standard/Economy are SCALING_POLICY values, a separate setting — not "modes" of the warehouse itself.)
</details>

---

### Question 69
Which loop type iterates until a condition becomes true?

- A. FOR
- B. LOOP
- C. REPEAT
- D. WHILE

<details><summary>Show Answer</summary>
Correct Answer: C. `REPEAT` is a post-condition loop (runs the body, then checks); `WHILE` checks first.
</details>

---

### Question 70
Which property should be added to an ALTER WAREHOUSE command to confirm that additional compute resources have been fully provisioned before the command returns?

- A. AUTO_RESUME
- B. RESOURCE_MONITOR
- C. SCALING_POLICY
- D. WAIT_FOR_COMPLETION

<details><summary>Show Answer</summary>
Correct Answer: D. `WAIT_FOR_COMPLETION = TRUE` makes the resize synchronous.
</details>

---

### Question 71
How is enhanced authentication achieved in Snowflake? (Choose two.)

- A. Network policies
- B. Snowflake-managed keys
- C. Object-level access control
- D. Multi-Factor Authentication (MFA)
- E. Federated authentication and Single Sign-On (SSO)

<details><summary>Show Answer</summary>
Correct Answer: D, E.
</details>

---

### Question 72
What are the native data types Snowflake provides for storing semi-structured data? (Choose two.)

- A. ARRAY
- B. JSON
- C. ORC
- D. Parquet
- E. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: A, E. JSON/ORC/Parquet are file formats, not Snowflake column data types.
</details>

---

### Question 73
How long is the Fail-safe period for recovering historical data from permanent tables?

- A. 1 day
- B. 3 days
- C. 7 days
- D. 14 days

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 74
What does `average_overlaps` in the output of `SYSTEM$CLUSTERING_INFORMATION` refer to?

- A. The average number of micro-partitions within the Time Travel window
- B. The average number of partitions physically stored in the same location
- C. The average number of micro-partitions that contain overlapping value ranges
- D. The average number of micro-partitions in the table associated with cloned objects

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 75
If queries start to queue in a multi-cluster virtual warehouse, an additional compute cluster starts immediately under which setting?

- A. Auto-scale mode
- B. Maximized mode
- C. Economy scaling policy
- D. Standard scaling policy

<details><summary>Show Answer</summary>
Correct Answer: D. Standard scaling policy favors starting clusters immediately on queueing; Economy waits and tries to conserve credits.
</details>

---

### Question 76
When floating-point number columns are unloaded to CSV or JSON files, Snowflake truncates the values to approximately what precision?

- A. (12,2)
- B. (10,4)
- C. (14,8)
- D. (15,9)

<details><summary>Show Answer</summary>
Correct Answer: D. ⚠ Updated: the source OCR's visible options (12,2 / 10,4 / 14,8) don't match current documentation at all. Per Snowflake's data-unload-considerations doc, floating-point columns unloaded to CSV/JSON are truncated to approximately **(15,9)** — 15 digits of precision, 9 after the decimal. Parquet unload does not truncate.
</details>

---

### Question 77
By definition, a secure view's definition is exposed only to users with what privilege?

- A. IMPORT SHARE
- B. OWNERSHIP
- C. REFERENCES
- D. USAGE

<details><summary>Show Answer</summary>
Correct Answer: B. Regular querying of a secure view only needs SELECT/USAGE — but seeing the underlying view *definition* (e.g., via `SHOW VIEWS` / `GET_DDL`) requires OWNERSHIP.
</details>

---

### Question 78
What happens when a user exits Snowsight while a query submitted in that session is still running?

- A. Snowflake immediately re-executes the query in the same session.
- B. Snowflake cancels any queries submitted during that session that are still running.
- C. Snowflake cancels queries submitted during that session after 24 hours.
- D. Snowflake continues executing the query server-side and it completes normally.

<details><summary>Show Answer</summary>
Correct Answer: D. Query execution isn't tied to the UI session staying open.
</details>

---

### Question 79
Which native data types are used for storing semi-structured data in Snowflake? (Choose two.)

- A. NUMBER
- B. OBJECT
- C. STRING
- D. VARCHAR
- E. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: B, E.
</details>

---

### Question 80
Which columns are available in the output of a Snowflake directory table? (Choose two.)

- A. CATALOG_NAME
- B. FILE_NAME
- C. LAST_MODIFIED
- D. RELATIVE_PATH
- E. STAGE_NAME

<details><summary>Show Answer</summary>
Correct Answer: C, D. (Full column set also includes SIZE, MD5, ETAG, and FILE_URL.)
</details>

---

### Question 81
What is used to diagnose and troubleshoot network connections to Snowflake?

- A. SnowCD
- B. Snowpark
- C. Snowsight
- D. SnowSQL

<details><summary>Show Answer</summary>
Correct Answer: A. SnowCD = Snowflake Connectivity Diagnostic Tool.
</details>

---

### Question 82
What Snowflake object records DML changes so that actions can be taken using the changed data?

- A. Pipe
- B. Stream
- C. Task
- D. View

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 83
By default, `COPY INTO <location>` separates table data into a set of output files to take advantage of which Snowflake feature?

- A. Query acceleration
- B. Query plan caching
- C. Parallel processing
- D. Time Travel

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 84
Which command can be used to view the allowed and blocked IP lists of a network policy?

- A. ALTER NETWORK POLICY
- B. CREATE NETWORK POLICY
- C. DESCRIBE NETWORK POLICY
- D. SHOW NETWORK POLICIES

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 85
Which file functions are non-deterministic? (Choose two.)

- A. BUILD_SCOPED_FILE_URL
- B. BUILD_STAGE_FILE_URL
- C. GET_STAGE_LOCATION
- D. GET_PRESIGNED_URL

<details><summary>Show Answer</summary>
Correct Answer: A, D. ⚠ Source options were fully missing beyond the answer key letters — rebuilt from the underlying concept. `BUILD_SCOPED_FILE_URL` and `GET_PRESIGNED_URL` both generate time-limited, session/context-dependent URLs, making them non-deterministic; `BUILD_STAGE_FILE_URL` and `GET_STAGE_LOCATION` are deterministic.
</details>

---

### Question 86
How can a Snowflake user optimize query performance? (Choose two.)

- A. Create a view.
- B. Cluster a table.
- C. Enable the search optimization service.
- D. Enable Time Travel.
- E. Index a table.

<details><summary>Show Answer</summary>
Correct Answer: B, C. Snowflake has no traditional indexes (E is a trap); Time Travel and views don't change performance on their own.
</details>

---

### Question 87
What is the MINIMUM role required to set organization-level parameters (e.g., enabling/disabling org-wide features)?

- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. SYSADMIN
- D. ORGADMIN

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 88
Which file format prevents floating-point numbers from being truncated when data is unloaded?

- A. CSV
- B. JSON
- C. ORC
- D. Parquet

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 89
A user has semi-structured data to load but isn't sure what operations will eventually be performed on it. What column type does Snowflake recommend?

- A. ARRAY
- B. OBJECT
- C. STRING
- D. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: D. VARIANT is the general-purpose recommendation when the downstream usage pattern is unknown.
</details>

---

### Question 90
Which Snowsight feature helps evaluate virtual warehouse performance that's being impacted by queueing?

- A. Resource Monitor
- B. Query History
- C. Warehouse Load Monitoring chart
- D. Access History

<details><summary>Show Answer</summary>
Correct Answer: C. ⚠ Source options were almost entirely lost — rebuilt around the concept. Snowsight's per-warehouse "Load" chart breaks down running vs. queued query time, which is the standard way to diagnose queueing.
</details>

---

### Question 91
Which Snowflake object can be created as TEMPORARY?

- A. Role
- B. Stage
- C. User
- D. Storage integration

<details><summary>Show Answer</summary>
Correct Answer: B. TABLE and STAGE support the TEMPORARY keyword; roles, users, and storage integrations do not.
</details>

---

### Question 92
Which stream type can be used for tracking records in external tables?

- A. Append-only
- B. Standard
- C. Insert-only
- D. Delta

<details><summary>Show Answer</summary>
Correct Answer: C. Streams on external tables (and externally managed Iceberg tables) require `INSERT_ONLY = TRUE` — this is a distinct type name from "Append-only," even though both only track inserts. Append-only applies to standard tables/views; Standard streams aren't supported on external tables at all.
</details>

---

### Question 93
What is the recommended approach for unloading data to a cloud storage location from Snowflake?

- A. Use a third-party tool to unload the data to cloud storage.
- B. Unload the data directly to the cloud storage location.
- C. Unload the data to an internal stage, then upload it separately to cloud storage.
- D. Unload the data to a user stage, then upload the data to cloud storage.

<details><summary>Show Answer</summary>
Correct Answer: B. `COPY INTO` an external stage/location directly, avoiding an unnecessary intermediate hop.
</details>

---

### Question 94
Which command is used to download files from an internal or external stage to the local file system?

- A. COPY INTO
- B. GET
- C. PUT
- D. TRANSFER

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 95
A tabular User-Defined Function (UDF) is defined by specifying a RETURNS clause that contains which keyword?

- A. ROW_NUMBER
- B. TABLE
- C. TABULAR
- D. VALUES

<details><summary>Show Answer</summary>
Correct Answer: B. `RETURNS TABLE (...)`.
</details>

---

### Question 96
Which SQL statement requires a running virtual warehouse to execute?

- A. `SELECT COUNT(*) FROM TBL_EMPLOYEE;`
- B. `ALTER TABLE TBL_EMPLOYEE ADD COLUMN EMP_REGION VARCHAR(20);`
- C. `INSERT INTO TBL_EMPLOYEE (EMP_NAME, EMP_SALARY, DEPT) VALUES ('Adam', 20000, 'Finance');`
- D. `CREATE OR REPLACE TABLE TBL_EMPLOYEE (EMP_ID NUMBER, EMP_NAME VARCHAR, EMP_SALARY NUMBER, DEPT VARCHAR(20));`

<details><summary>Show Answer</summary>
Correct Answer: C. DDL (ALTER/CREATE TABLE) is a metadata-only operation handled by the cloud services layer, and simple `COUNT(*)` can often be answered from metadata — but writing actual row data via INSERT requires compute.
</details>

---

### Question 97
Which REST API operation can be used to work with unstructured data files?

- A. insertFile
- B. insertReport
- C. GET
- D. loadHistoryScan

<details><summary>Show Answer</summary>
Correct Answer: C. The Snowflake Files API exposes GET/PUT-style REST endpoints for retrieving staged files.
</details>

---

### Question 98
Which query returns the Snowflake-hosted file URL from a directory table for a stage named `bronzestage`?

- A. `LIST @bronzestage;`
- B. `SELECT file_url FROM DIRECTORY(@bronzestage);`
- C. `SELECT metadata$filename FROM @bronzestage;`
- D. `SELECT * FROM DIRECTORY(@bronzestage) WHERE file_url IS NOT NULL;`

<details><summary>Show Answer</summary>
Correct Answer: B. The `FILE_URL` column of a directory table query returns the Snowflake-hosted URL.
</details>

---

### Question 99
Which feature is integrated to support Multi-Factor Authentication (MFA) in Snowflake?

- A. Authy
- B. Duo Security
- C. OneLogin
- D. RSA SecurID Access

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 100
In which Snowflake layer does Snowflake reorganize data into its internal, optimized, compressed, columnar format?

- A. Cloud Services
- B. Database Storage
- C. Query Processing
- D. Metadata Management

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 101
When can user session variables be accessed inside a Snowflake Scripting stored procedure?

- A. When the procedure is defined as STRICT.
- B. When the procedure is defined to execute as CALLER.
- C. When the procedure is defined to execute as OWNER.
- D. When the procedure is defined with an argument matching the session variable's name and type.

<details><summary>Show Answer</summary>
Correct Answer: B. `EXECUTE AS CALLER` runs the procedure in the invoking session's context, giving it access to that session's variables; `EXECUTE AS OWNER` does not.
</details>

---

### Question 102
Which computer language can be selected when creating User-Defined Functions (UDFs) via the Snowpark API?

- A. Swift
- B. JavaScript
- C. Python
- D. Ruby

<details><summary>Show Answer</summary>
Correct Answer: C. (Snowpark also supports Java and Scala, but of the listed options Python is correct.)
</details>

---

### Question 103
A user needs to ingest 1 GB of data available in an external stage using COPY INTO. How can this be done with MAXIMUM performance and LEAST cost?

- A. Ingest the data as a single compressed file.
- B. Ingest the data as a single uncompressed file.
- C. Split the file into smaller files of 100–250 MB each, compress them, and ingest each separately.
- D. Split the file into smaller files of 100–250 MB each and ingest them uncompressed.

<details><summary>Show Answer</summary>
Correct Answer: C. Splitting enables parallel loading; compression reduces both transfer time and storage cost.
</details>

---

### Question 104
A Snowflake user has two tables containing numeric values and wants to find values present in BOTH tables. Which set operator should be used?

- A. INTERSECT
- B. MERGE
- C. MINUS
- D. UNION

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 105
A view is defined on a permanent table. A temporary table with the same name is then created in the same schema as the referenced table. What does querying the view return?

- A. Data from the permanent table.
- B. Data from the temporary table.
- C. An error stating the view could not be compiled.
- D. An error stating the referenced object could not be uniquely identified.

<details><summary>Show Answer</summary>
Correct Answer: B. Temporary tables shadow same-named permanent tables for the duration of the session — including inside views resolved within that session.
</details>

---

### Question 106
Which file function generates a Snowflake-hosted file URL for a staged file, using the stage name and relative file path as inputs?

- A. GET_ABSOLUTE_PATH
- B. GET_PRESIGNED_URL
- C. BUILD_STAGE_FILE_URL
- D. BUILD_SCOPED_FILE_URL

<details><summary>Show Answer</summary>
Correct Answer: C. ⚠ Updated: the source only preserved one garbled option (`GET_ABSOLUTE_PATH`) with no marked answer. `GET_ABSOLUTE_PATH` returns a path string, not a URL. `BUILD_STAGE_FILE_URL(stage, relative_path)` is the documented function that builds a Snowflake-hosted file URL from exactly those two inputs.
</details>

---

### Question 107
Which service or feature in Snowflake improves the performance of lookup/analytical queries that use an extensive set of WHERE conditions?

- A. Data classification
- B. Query acceleration service
- C. Search optimization service
- D. Object tagging

<details><summary>Show Answer</summary>
Correct Answer: C. The Search Optimization Service is purpose-built for highly selective point-lookup queries with many predicates; Query Acceleration Service targets large-scan/heavy-aggregation offloading instead.
</details>

---

## Summary of Documentation-Based Corrections

| # | Topic | Original (garbled) answer | Verified current answer |
|---|-------|---------------------------|--------------------------|
| 9 | ALLOWED_VALUES max tag values | 256 | **5,000** (per current CREATE TAG / ALTER TAG docs) |
| 66 | ACCOUNT_USAGE retention | "7 days–6 months" listed as a correct trait | That range is **INFORMATION_SCHEMA**; ACCOUNT_USAGE retains **365 days** |
| 76 | Float truncation on unload | Options garbled/didn't include real value | **(15,9)** per docs |
| 92 | Stream type for external tables | Ambiguous OCR | **Insert-only** (a distinct type name from Append-only) |
| 106 | Function building a stage file URL | Unclear/only one option preserved | **BUILD_STAGE_FILE_URL** |

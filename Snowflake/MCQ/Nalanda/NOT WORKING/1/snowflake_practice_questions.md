# Snowflake Practice Questions — Cleaned & Verified

*Reconstructed from a heavily corrupted OCR scan. Questions have been renumbered sequentially (1–N) since the original numbering was inconsistent (largely erased or scrambled by the scan). Garbled options have been reconstructed to the most plausible, standard phrasing based on official Snowflake documentation. Answers were cross-checked against current Snowflake documentation as of July 2026; corrections are flagged with **⚠ Updated**. Click "Show Answer" to reveal each answer — try the question first!*

---

### Question 1
What technique does Snowflake recommend for determining which virtual warehouse size to select?

- A. Always start with an X-Small and increase the size if the query does not complete in 2 minutes
- B. Experiment by running the same queries against warehouses of different sizes
- C. Use the default size Snowflake chooses
- D. Use X-Large or above for tables larger than 1 GB

<details><summary>Show Answer</summary>
Correct Answer: B. Snowflake recommends experimenting with the same query/workload on different warehouse sizes to empirically find the best price/performance point, since warehouse performance doesn't scale in a simple linear way with size.
</details>

---

### Question 2
Which command should be used when loading many flat files into a single table?

- A. PUT
- B. INSERT
- C. COPY INTO
- D. MERGE

<details><summary>Show Answer</summary>
Correct Answer: C. COPY INTO is the bulk-loading command used to load staged files (one or many) into a table. PUT only stages files; it doesn't load them.
</details>

---

### Question 3
How can a Snowflake user share data with another user who does not have a Snowflake account?

- A. Share the data by implementing User-Defined Functions (UDFs)
- B. Create a reader account and create a share of the data
- C. Grant the READER privilege to the database that is going to be shared
- D. Move the Snowflake account to a region where data sharing is enabled

<details><summary>Show Answer</summary>
Correct Answer: B. A reader account lets a provider share data with a consumer who has no Snowflake account of their own; the provider account bears the compute/storage costs.
</details>

---

### Question 4
Which semi-structured data formats can be loaded into Snowflake with a COPY command? (Choose two.)

- A. CSV
- B. Avro
- C. HTML
- D. ORC
- E. XML

<details><summary>Show Answer</summary>
Correct Answer: D and E — ORC and XML. *(Note: this option list was badly garbled in the source scan and has been reconstructed. Snowflake's COPY INTO also supports JSON, Avro, and Parquet as semi-structured formats — CSV is structured, and HTML is not a supported load format.)*
</details>

---

### Question 5
Which statements reflect valid commands using secondary roles? (Choose two.)

- A. USE SECONDARY ROLES RESUME
- B. USE SECONDARY ROLES SUSPEND
- C. USE SECONDARY ROLES ALL
- D. USE SECONDARY ROLES ADD [role name]
- E. USE SECONDARY ROLES NONE

<details><summary>Show Answer</summary>
Correct Answer: C and E. The actual supported syntax is `USE SECONDARY ROLES ALL | NONE` — there is no RESUME, SUSPEND, or ADD variant.
</details>

---

### Question 6
How long is a query visible in the Query History page in the Snowflake Web Interface (UI)?

- A. 60 minutes
- B. 24 hours
- C. 14 days
- D. 30 days

<details><summary>Show Answer</summary>
Correct Answer: C. The Query History page in Snowsight shows queries executed in the last 14 days. (For longer retention, query the `SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY` view, which retains 365 days of history — confirmed current as of July 2026.)
</details>

---

### Question 7
Two users share a virtual warehouse. When one of the users loads data, the other experiences performance issues while querying data. How does Snowflake recommend resolving this issue?

- A. Scale up the existing warehouse
- B. Create separate warehouses for each user
- C. Create separate warehouses for each workload
- D. Stop loading and querying data at the same time

<details><summary>Show Answer</summary>
Correct Answer: C. Snowflake's best practice is to isolate different **workload types** (e.g., loading vs. querying) onto separate warehouses, not necessarily separate warehouses per user.
</details>

---

### Question 8
What is a feature of a stored procedure in Snowflake?

- A. They can be created as secure and hide the underlying metadata from all users.
- B. They can only access tables from a single database.
- C. They can contain only a single statement.
- D. They can be created to run with a caller's rights or an owner's rights.

<details><summary>Show Answer</summary>
Correct Answer: D. Stored procedures can be defined with `EXECUTE AS CALLER` or `EXECUTE AS OWNER`, controlling whose privileges are used at runtime.
</details>

---

### Question 9
Which view will return users who have queried a table?

- A. SNOWFLAKE.ACCOUNT_USAGE.COLUMNS
- B. SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY
- C. SNOWFLAKE.ACCOUNT_USAGE.TABLES
- D. SNOWFLAKE.ACCOUNT_USAGE.OBJECT_DEPENDENCIES

<details><summary>Show Answer</summary>
Correct Answer: B. *(Reconstructed — only two of the four options survived the scan.)* ACCESS_HISTORY records read and write operations, including which user accessed which table/columns, making it the correct view for this purpose.
</details>

---

### Question 10
Why do Snowflake's virtual warehouses have scaling policies?

- A. To help save storage costs
- B. To help increase the performance of serverless computing features
- C. To help control the credits consumed by a multi-cluster warehouse running in auto-scale mode
- D. To help control the credits consumed by a multi-cluster warehouse running in maximized mode

<details><summary>Show Answer</summary>
Correct Answer: C. Scaling policies (Standard vs. Economy) govern how aggressively a multi-cluster warehouse in **auto-scale** mode starts/stops clusters, balancing query queuing against credit consumption.
</details>

---

### Question 11
Where can a Snowflake user find the query history in Snowsight?

- A. Admin
- B. Activity
- C. Worksheets
- D. Data

<details><summary>Show Answer</summary>
Correct Answer: B. Query History lives under Monitoring/Activity in Snowsight's navigation menu.
</details>

---

### Question 12
What is SnowSQL?

- A. Snowflake's new user interface where users can visualize data into charts and dashboards.
- B. Snowflake's proprietary extension of the ANSI SQL standard, including built-in keywords and system functions.
- C. Snowflake's command-line client, built on the Python Connector, used to connect to Snowflake and execute SQL.
- D. Snowflake's library that provides a programming interface for processing data on Snowflake without moving it to the system where the application code runs.

<details><summary>Show Answer</summary>
Correct Answer: C. SnowSQL is the CLI client for Snowflake.
</details>

---

### Question 13
*This question (about the output of a `NEXTVAL` call on a sequence after several statements) was too badly corrupted in the source scan to reliably reconstruct — the actual SQL statements and answer options were lost. As general background: each call to `<sequence>.NEXTVAL` returns a new, generally increasing value and advances the sequence, regardless of transaction rollbacks, so repeated calls do not return the same number. If you have the original source for this question, it's worth re-extracting rather than trusting a guessed reconstruction here.*

---

### Question 14
Which statement is true of cloning?

- A. It increases storage costs, as cloning a table requires storing its data twice.
- B. A cloned table includes the load history of the original.
- C. It is licensed as an additional Snowflake feature.
- D. All micro-partitions between the original and cloned tables are fully shared.

<details><summary>Show Answer</summary>
Correct Answer: D. Zero-copy cloning means the clone initially shares all underlying micro-partitions with the source; storage cost only grows as the clone or source diverges.
</details>

---

### Question 15
A Snowflake user has been granted the CREATE DATA EXCHANGE LISTING privilege with their role. Which tasks can this user perform on the Data Exchange? (Choose two.)

- A. Rename listings
- B. Delete provider profiles
- C. Modify listing properties
- D. Modify incoming listing access requests
- E. Submit listings

<details><summary>Show Answer</summary>
Correct Answer: C and E. This privilege lets a user create, modify the properties of, and submit their own listings. Renaming listings, deleting provider profiles, and managing incoming access requests require higher-level administrative privileges.
</details>

---

### Question 16
Which parameter prevents streams on tables from becoming stale?

- A. MAX_DATA_EXTENSION_TIME_IN_DAYS
- B. MIN_DATA_RETENTION_TIME_IN_DAYS
- C. LOCK_TIMEOUT
- D. STALE_AFTER

<details><summary>Show Answer</summary>
Correct Answer: A. If a table's data retention period is shorter than the stream's unconsumed offset, Snowflake temporarily extends retention — up to the value set by MAX_DATA_EXTENSION_TIME_IN_DAYS (default max 14 days) — to keep the stream from going stale. (STALE_AFTER is just a computed/displayed timestamp, not a settable parameter.)
</details>

---

### Question 17
If a virtual warehouse runs for 30 seconds after it is provisioned, how many seconds will the customer be billed for?

- A. 30 seconds
- B. 60 seconds
- C. 120 seconds
- D. 1 hour

<details><summary>Show Answer</summary>
Correct Answer: B. Snowflake bills a 60-second minimum each time a warehouse starts, then per-second after that.
</details>

---

### Question 18
When should a stored procedure be created with caller's rights?

- A. When the caller needs to be prevented from viewing the source code of the stored procedure
- B. When the caller needs to run a statement that could not execute outside of the stored procedure
- C. When the stored procedure needs to run with the privileges of the role that called the stored procedure
- D. When the stored procedure needs to operate on objects that the caller does not have privileges on

<details><summary>Show Answer</summary>
Correct Answer: C. Caller's rights procedures execute using the privileges of whoever calls them, rather than the privileges of the procedure's owner (which is what owner's rights procedures do — and which is what enables D-style behavior instead).
</details>

---

### Question 19
What JavaScript delimiters are available in Snowflake stored procedures? (Choose two.)

- A. Double quote (")
- B. Single quote (')
- C. Forward slash (/)
- D. Double backslash (\\)
- E. Double dollar sign ($$)

<details><summary>Show Answer</summary>
Correct Answer: B and E. JavaScript procedure bodies can be delimited with single quotes or with `$$ ... $$`.
</details>

---

### Question 20
What type of function can be used to estimate the approximate number of distinct values from a table that has trillions of rows?

- A. MODE
- B. Window
- C. External
- D. HyperLogLog (HLL)

<details><summary>Show Answer</summary>
Correct Answer: D. HyperLogLog-based approximate functions (e.g., `APPROX_COUNT_DISTINCT`) estimate cardinality with a small, fixed memory footprint, making them practical at massive scale where an exact `COUNT(DISTINCT …)` would be expensive.
</details>

---

### Question 21
Which Data Definition Language (DDL) commands are supported by Snowflake to manage tags? (Choose two.)

- A. ALTER TAG
- B. DESCRIBE TAG
- C. CREATE TAG
- D. GRANT [privilege] TO TAG
- E. GRANT TAG

<details><summary>Show Answer</summary>
Correct Answer: A and C. Snowflake supports full tag DDL — CREATE TAG, ALTER TAG, DROP TAG, UNDROP TAG, SHOW TAGS — but there is no GRANT ... TO TAG or GRANT TAG statement (tags are applied via `ALTER <object> SET TAG`, not GRANT).
</details>

---

### Question 22
What Snowflake objects can be added to a share? (Choose two.)

- A. Tables
- B. Views
- C. Stored procedures
- D. Sequences
- E. Secure views

<details><summary>Show Answer</summary>
Correct Answer: A and E. Tables and secure views (as well as secure UDFs) can be added to a share; standard non-secure views cannot be shared by default, and stored procedures/sequences are not shareable objects. *(Option list reconstructed — the source scan duplicated "Views" and dropped one option.)*
</details>

---

### Question 23
A Query Profile shows a UnionAll operator with an extra Aggregate operator on top. What does this signify?

- A. Exploding joins
- B. Inefficient pruning
- C. UNION without ALL
- D. Queries that are too large to fit in memory

<details><summary>Show Answer</summary>
Correct Answer: C. Snowflake implements a plain `UNION` (which must de-duplicate) as a `UnionAll` followed by an `Aggregate` to remove duplicate rows — seeing that pattern in the profile is a signal that a `UNION ALL` (skipping the extra aggregate) may be usable instead if duplicates are acceptable/impossible.
</details>

---

### Question 24
Which data governance control has Snowflake embedded in the application?

- A. Row Access Policies
- B. Credit computation
- C. Data storage
- D. Attribute-based access control

<details><summary>Show Answer</summary>
Correct Answer: A. **⚠ Updated:** Some older question dumps mark "Attribute-based access control" (D) as correct, but this is inaccurate — Snowflake's own documentation does not describe its access model as ABAC (Snowflake uses DAC and RBAC). Snowflake's documentation explicitly lists Row Access Policies, along with Object Tagging, Dynamic Data Masking, and network policies, as governance controls embedded directly in the platform.
</details>

---

### Question 25
What actions does the use of the PUT command do automatically? (Choose two.)

- A. It creates a file format object.
- B. It uses the last stage created.
- C. It compresses files using GZIP.
- D. It encrypts the file data in transit.
- E. It creates an empty target table.

<details><summary>Show Answer</summary>
Correct Answer: C and D. By default, PUT compresses files with gzip (unless `AUTO_COMPRESS = FALSE`) and encrypts data automatically before upload. It does not create file formats or tables, and it requires you to specify the target stage explicitly.
</details>

---

### Question 26
Which command should a Snowflake user execute to load data into a table using a specific file format?

- A. `COPY INTO mytable PURGE_MODE = TRUE;`
- B. `COPY mytable;`
- C. `COPY INTO mytable FILE_FORMAT = (FORMAT_NAME = '<name>');`
- D. `COPY INTO mytable VALIDATION = TRUE;`

<details><summary>Show Answer</summary>
Correct Answer: C. This is the correct COPY INTO syntax for specifying a named file format; the others use invalid parameter names/syntax.
</details>

---

### Question 27
Which function returns a URL to a staged file using the stage name and file path as input, without requiring an active data retention/results-cache period?

- A. GET_PRESIGNED_URL
- B. BUILD_SCOPED_FILE_URL
- C. GET_RELATIVE_PATH
- D. BUILD_STAGE_FILE_URL

<details><summary>Show Answer</summary>
Correct Answer: D. BUILD_STAGE_FILE_URL generates a permanent, Snowflake-hosted file URL from a stage name and relative file path. *(This question's options were heavily garbled in the source; reconstructed based on the closest standard exam phrasing.)*
</details>

---

### Question 28
What is the MAXIMUM number of clusters that can be provisioned with a multi-cluster virtual warehouse?

- A. 2
- B. 5
- C. 10
- D. 100

<details><summary>Show Answer</summary>
Correct Answer: C. A multi-cluster warehouse supports up to 10 clusters.
</details>

---

### Question 29
Which Snowflake table type is specifically used to catalog and support access to unstructured data?

- A. Directory table
- B. Transient table
- C. Temporary table
- D. Permanent table

<details><summary>Show Answer</summary>
Correct Answer: A. A directory table stores a catalog of staged unstructured files and their URLs; it is not a standalone table type but a metadata layer associated with a stage.
</details>

---

### Question 30
When unloading data, which file format preserves the data values for floating-point number columns?

- A. CSV
- B. JSON
- C. ORC
- D. Parquet

<details><summary>Show Answer</summary>
Correct Answer: D. Parquet retains native floating-point precision on unload, whereas text formats like CSV can lose precision.
</details>

---

### Question 31
Which virtual warehouse privilege is required to view a load-monitoring chart?

- A. MONITOR
- B. MODIFY
- C. OPERATE
- D. USAGE

<details><summary>Show Answer</summary>
Correct Answer: A. The MONITOR privilege on a warehouse allows viewing its load/utilization charts.
</details>

---

### Question 32
Which use case will always cause an exploding join in Snowflake?

- A. A query that has more than 10 left outer joins
- B. A query that is using a UNION without an ALL
- C. A query that has not specified join criteria between tables
- D. A query that has requested too many columns of data

<details><summary>Show Answer</summary>
Correct Answer: C. A join with no join criteria (or overly permissive criteria) produces a Cartesian-style row explosion, dramatically inflating the result set size.
</details>

---

### Question 33
How many resource monitors can be assigned to a single virtual warehouse (below the account level)?

- A. Zero
- B. One
- C. Eight
- D. Unlimited

<details><summary>Show Answer</summary>
Correct Answer: B. A warehouse can be assigned to only a single resource monitor below the account level, though one resource monitor can be assigned to multiple warehouses. (Confirmed current in Snowflake documentation as of July 2026.)
</details>

---

### Question 34
What are the main differences between Account Usage views and Information Schema views? (Choose two.)

- A. No active warehouse is needed to query Account Usage views, but one is needed for Information Schema views.
- B. Account Usage views do not contain data about tables, but Information Schema views do.
- C. Account Usage views contain dropped-object information; Information Schema views do not.
- D. Data retention for Account Usage views is up to 1 year, but is 7 days to 6 months for Information Schema views, depending on the view.
- E. Information Schema views are read-only, but Account Usage views are not.

<details><summary>Show Answer</summary>
Correct Answer: C and D. Account Usage views include historical/dropped objects and retain data far longer (up to 1 year, with some latency), while Information Schema views are limited to a much shorter window and only show current, non-dropped objects.
</details>

---

### Question 35
Which file function builds a URL to a file on a stage without requiring authentication or authorization to access it?

- A. GET_PRESIGNED_URL
- B. BUILD_SCOPED_FILE_URL
- C. BUILD_STAGE_FILE_URL
- D. GET_RELATIVE_PATH

<details><summary>Show Answer</summary>
Correct Answer: A. GET_PRESIGNED_URL generates a time-limited URL that can be used to access a file directly (e.g., in a browser) without any further Snowflake authentication.
</details>

---

### Question 36
Which view can be used to determine if a table has frequent row updates or deletes?

- A. TABLES
- B. TABLE_STORAGE_METRICS
- C. LOAD_HISTORY
- D. STORAGE_USAGE

<details><summary>Show Answer</summary>
Correct Answer: B. TABLE_STORAGE_METRICS exposes storage broken out by active data, Time Travel, and Fail-safe bytes — high Time Travel/Fail-safe storage relative to active data is a signal of frequent updates/deletes. *(Options B and C were unreadable in the source scan and have been reconstructed.)*
</details>

---

### Question 37
How does the Snowflake Search Optimization Service improve query performance?

- A. It improves the performance of range searches.
- B. It defines different clustering keys on the same source table.
- C. It improves the performance of all queries running against a given table.
- D. It improves the performance of equality searches.

<details><summary>Show Answer</summary>
Correct Answer: D. Search Optimization Service is designed for selective point-lookup/equality queries on high-cardinality columns (later extended to some other predicate types, but equality search is the core, exam-tested case).
</details>

---

### Question 38
How is unstructured data retrieved from data storage?

- A. SQL functions like the GET command can be used to copy the unstructured data to a location on the client.
- B. SQL functions can be used to create different types of URLs pointing to the unstructured data; these URLs can be used to download the data to a client.
- C. SQL functions can retrieve the data from the query results cache, and when the query results are output to a client, the unstructured data is output as files.
- D. SQL functions can call different web extensions designed to display different file types as a web page; the extensions allow the files to be downloaded to the client.

<details><summary>Show Answer</summary>
Correct Answer: B. File functions (file URLs, scoped URLs, presigned URLs) are the mechanism for retrieving unstructured data from stages.
</details>

---

### Question 39
What is the recommended way to obtain a cloned table with the same grants as the source table?

- A. Clone the table with the COPY GRANTS command.
- B. Use an ALTER TABLE command to copy the grants.
- C. Clone the schema, then drop the unwanted tables.
- D. Create a script to extract grants and apply them to the cloned table.

<details><summary>Show Answer</summary>
Correct Answer: A. `CREATE TABLE ... CLONE ... COPY GRANTS` carries over the source object's access grants to the clone.
</details>

---

### Question 40
What common query issues can be identified using the Query Profile? (Choose two.)

- A. Data classification
- B. Exploding joins
- C. Unions
- D. Inefficient pruning
- E. Data masking

<details><summary>Show Answer</summary>
Correct Answer: B and D. Query Profile visually surfaces performance problems like exploding joins and poor partition pruning; it isn't a tool for data classification or masking configuration.
</details>

---

### Question 41
What is used to extract the content of PDF files stored in Snowflake stages?

- A. FLATTEN function
- B. Window function
- C. HyperLogLog (HLL) function
- D. Java User-Defined Function (UDF)

<details><summary>Show Answer</summary>
Correct Answer: D. A Java UDF (using a PDF-parsing library) is the classic exam answer for extracting text from PDF files on a stage. *(This question appeared twice, nearly identically, in the source scan — consolidated into one entry here.)*
</details>

---

### Question 42
What happens when a database is cloned?

- A. It does not retain privileges granted on the source.
- B. It replicates all granted privileges on the corresponding source objects.
- C. It replicates all granted privileges on the corresponding child objects.
- D. It replicates all granted privileges on the corresponding child schema objects.

<details><summary>Show Answer</summary>
Correct Answer: C. Cloning a database replicates privileges granted on the child objects within it (schemas, tables, etc.), not just at the database level itself.
</details>

---

### Question 43
What does a Query Profile provide in Snowflake?

- A. A multi-step query that displays each processing step in the same panel.
- B. A pre-computed data set derived from a query specification and stored for later use.
- C. A graphical representation of the main components of the processing plan for a query.
- D. A collapsible panel in the operator tree pane that lists nodes by execution time in descending order.

<details><summary>Show Answer</summary>
Correct Answer: C. Query Profile is a graphical execution-plan visualization.
</details>

---

### Question 44
When executing a COPY INTO command, performance can be negatively affected by using which optional parameter on a large number of files?

- A. FILE_FORMAT
- B. PATTERN
- C. VALIDATION_MODE
- D. FILES

<details><summary>Show Answer</summary>
Correct Answer: B. PATTERN applies a regular expression against every file in the location, which becomes expensive at scale; listing exact FILES is generally faster when the file list is large.
</details>

---

### Question 45
Which URL type should be used to get a permanent URL to a file in a stage?

- A. File URL
- B. Pre-signed URL
- C. Scoped URL
- D. Virtual-hosted-style URL

<details><summary>Show Answer</summary>
Correct Answer: A. A File URL does not expire, unlike Scoped URLs (tied to the results-cache period, ~24 hours) or Pre-signed URLs (configurable expiration, up to 7 days). Access still requires the caller's role to hold sufficient privileges on the stage.
</details>

---

### Question 46
Which operation will produce an error in Snowflake?

- A. Inserting duplicate values into a PRIMARY KEY column
- B. Inserting a NULL into a column with a NOT NULL constraint
- C. Inserting duplicate values into a column with a UNIQUE constraint
- D. Inserting a value into a FOREIGN KEY column that does not match a value in the referenced column

<details><summary>Show Answer</summary>
Correct Answer: B. Snowflake does not enforce PRIMARY KEY, UNIQUE, or FOREIGN KEY constraints by default (they're informational/metadata only), but it does enforce NOT NULL constraints.
</details>

---

### Question 47
How are URLs that access unstructured data in external stages retrieved?

- A. Through the navigation menu
- B. By querying a directory table
- C. By creating an external function
- D. By using the INFORMATION_SCHEMA schema

<details><summary>Show Answer</summary>
Correct Answer: B. Directory tables catalog staged files and expose their URLs via SQL query.
</details>

---

### Question 48
What is the Snowflake multi-cluster feature for virtual warehouses primarily used for?

- A. To improve the data-unloading process to the cloud
- B. To improve data loading from very large data sets
- C. To improve concurrency for users and queries
- D. To speed up slow or stalled individual queries

<details><summary>Show Answer</summary>
Correct Answer: C. Multi-cluster warehouses add or remove compute clusters to handle concurrency (many simultaneous users/queries), not to speed up any single query.
</details>

---

### Question 49
Which features could be used to improve the performance of queries that return a small subset of rows from a large table? (Choose two.)

- A. Search Optimization Service
- B. Automatic clustering
- C. Row access policies
- D. Multi-cluster warehouses
- E. Secure views

<details><summary>Show Answer</summary>
Correct Answer: A and B. Both improve pruning/lookup efficiency for selective queries on large tables; the other options address concurrency or security, not this kind of query performance.
</details>

---

### Question 50
Which command would return an empty sample?

- A. `SELECT * FROM testtable SAMPLE (0);`
- B. `SELECT * FROM testtable SAMPLE ROW (0);`
- C. `SELECT * FROM testtable SAMPLE (NULL);`
- D. `SELECT * FROM testtable SAMPLE (NONE);`

<details><summary>Show Answer</summary>
Correct Answer: A. Sampling 0% of rows returns an empty result set; `SAMPLE (NULL)` and `SAMPLE (NONE)` are not valid syntax and would raise an error rather than return an empty (but valid) sample.
</details>

---

### Question 51
What Snowflake function should be used to unload relational data to JSON?

- A. TO_JSON
- B. OBJECT_CONSTRUCT
- C. PARSE_JSON
- D. ARRAY_CONSTRUCT

<details><summary>Show Answer</summary>
Correct Answer: B. OBJECT_CONSTRUCT builds a JSON-like OBJECT (VARIANT) from column values, which can then be unloaded as JSON. PARSE_JSON goes the other direction (string → VARIANT).
</details>

---

### Question 52
Floating-point values are truncated (lose precision) when unloaded to which file format?

- A. ORC
- B. CSV
- C. Avro
- D. Parquet

<details><summary>Show Answer</summary>
Correct Answer: B. Text-based CSV output can truncate/round floating-point precision, unlike binary formats like Parquet.
</details>

---

### Question 53
Which levels can network policies apply to? (Choose two.)

- A. Account
- B. Database
- C. Role
- D. Schema
- E. User

<details><summary>Show Answer</summary>
Correct Answer: A and E. Network policies can be set at the account level (affecting all users) or applied to individual users; they cannot be attached to databases, schemas, or roles.
</details>

---

### Question 54
What causes objects in a data share to become unavailable to a consumer account?

- A. A parameter in the consumer account is set to 0.
- B. The consumer account runs `GRANT IMPORTED PRIVILEGES` on the data share every 24 hours.
- C. The objects in the data share are dropped/recreated and the grant is not re-applied to the share.
- D. The consumer account acquires the data share through a private data exchange.

<details><summary>Show Answer</summary>
Correct Answer: C. Recreating an object (even with the same name) is treated as a new object; it must be explicitly re-granted to the share, or consumers lose access.
</details>

---

### Question 55
Which resource can an administrator use to monitor updates (for example, SCIM API requests) sent to Snowflake by an identity provider?

- A. ACCESS_HISTORY
- B. LOGIN_HISTORY
- C. QUERY_HISTORY
- D. LOGIN_HISTORY / SCIM-related audit logs

<details><summary>Show Answer</summary>
Correct Answer: B (best available answer). **Note on confidence:** the options for this question were too corrupted in the source scan to fully reconstruct with certainty, and Snowflake does not have one single, universally-cited "SCIM history" view in the way it does for queries or logins. If this is a real exam item you need precisely, it's worth checking Snowflake's current SCIM/identity-provider monitoring documentation directly rather than relying on this reconstruction.
</details>

---

### Question 56
A Snowflake user is writing a User-Defined Function (UDF) with some unqualified object names. How will those object names be resolved during execution?

- A. Snowflake will resolve them according to the SEARCH_PATH parameter.
- B. Snowflake will only check the schema the UDF belongs to.
- C. Snowflake will first check the current schema, then the schema the previous query used.
- D. Snowflake will first check the current schema, then the PUBLIC schema of the current database.

<details><summary>Show Answer</summary>
Correct Answer: B. Unlike ordinary session SQL (which uses SEARCH_PATH), unqualified object references inside a UDF are resolved only against the schema in which the UDF itself was created — a common exam "gotcha."
</details>

---

### Question 57
Why should a user select a scaling policy for a multi-cluster warehouse?

- A. To prevent/minimize query queuing
- B. To increase performance of individual clusters
- C. To reduce concurrent user queries
- D. To conserve credits by keeping running clusters fully loaded

<details><summary>Show Answer</summary>
Correct Answer: D. This describes the Economy scaling policy's goal specifically. (Note: the Standard scaling policy instead optimizes for minimizing queuing, i.e., option A — the two policies represent a trade-off between these goals, so double-check which policy a real exam question is asking about.)
</details>

---

### Question 58
What is the MINIMUM privilege required on an external stage for any role to use the GET REST API to retrieve unstructured data files via a file URL?

- A. READ
- B. OWNERSHIP
- C. USAGE
- D. WRITE

<details><summary>Show Answer</summary>
Correct Answer: C. USAGE on the stage is the minimum privilege needed to retrieve files via their file URL.
</details>

---

### Question 59
Which view in SNOWFLAKE.ACCOUNT_USAGE shows the IP address from which a user connected to Snowflake?

- A. ACCESS_HISTORY
- B. LOGIN_HISTORY
- C. SESSIONS
- D. QUERY_HISTORY

<details><summary>Show Answer</summary>
Correct Answer: B. LOGIN_HISTORY records login attempts, including the client IP address.
</details>

---

### Question 60
Snowflake Partner Connect is limited to users with a verified email address and which role?

- A. SYSADMIN
- B. SECURITYADMIN
- C. ACCOUNTADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: C. Partner Connect requires the ACCOUNTADMIN role.
</details>

---

### Question 61
What unit of storage supports efficient query processing in Snowflake?

- A. Blobs
- B. JSON
- C. Block storage
- D. Micro-partitions

<details><summary>Show Answer</summary>
Correct Answer: D. Snowflake automatically divides table data into small, immutable, columnar micro-partitions, which underpin its pruning and performance model.
</details>

---

### Question 62
What is the difference between a stored procedure and a User-Defined Function (UDF)?

- A. Stored procedures can perform database operations (DDL/DML) while UDFs cannot.
- B. Returning a value is required in a stored procedure, while returning a value in a UDF is optional.
- C. Values returned by a stored procedure can be used directly in a SQL statement, while values returned by a UDF cannot.
- D. Multiple stored procedures can be called as part of a single executable statement, while a single SQL statement can only call one UDF.

<details><summary>Show Answer</summary>
Correct Answer: A. Stored procedures can execute DDL/DML and control flow logic; UDFs are meant for computing and returning a value and cannot perform side-effecting operations like CREATE or INSERT.
</details>

---

### Question 63
Which URL type does Snowflake recommend using when providing unstructured data to other accounts through a share?

- A. File URL
- B. Pre-signed URL
- C. Scoped URL
- D. Virtual-hosted-style URL

<details><summary>Show Answer</summary>
Correct Answer: C. Scoped URLs are recommended for sharing unstructured data across accounts because they provide time-limited, per-query access without exposing a permanent link or requiring the consumer to authenticate against the provider's stage directly.
</details>

---

### Question 64
What is the MAXIMUM Time Travel retention period for a transient table?

- A. 0 days
- B. 1 day
- C. 7 days
- D. 90 days

<details><summary>Show Answer</summary>
Correct Answer: B. Transient tables (and transient/temporary schemas and databases) are capped at a maximum of 1 day of Time Travel, regardless of Snowflake edition — unlike permanent tables, which can go up to 90 days on Enterprise Edition and above.
</details>

---

### Question 65
What is the advantage of using a reader account?

- A. It can be used by a client that does not have their own Snowflake account.
- B. It is read-only and prevents the shared data from being updated by the provider.
- C. It can be connected to a Snowflake account in a different region.
- D. It provides limited access to the data share, making it cheaper for the data provider.

<details><summary>Show Answer</summary>
Correct Answer: A. Reader accounts exist specifically to let a provider extend Snowflake access to consumers who don't have (and don't want to pay for) their own Snowflake account.
</details>

---

### Question 66
What command is used to export or unload data from Snowflake?

- A. `PUT @mystage`
- B. `GET @mystage`
- C. `COPY INTO @mystage`
- D. `INSERT @mystage`

<details><summary>Show Answer</summary>
Correct Answer: C. `COPY INTO <location>` unloads table data to a stage; PUT/GET move files between a stage and local disk, not table data.
</details>

---

### Question 67
A Snowflake user wants to share data with someone who does not have a Snowflake account. How can they do this?

- A. Use the Snowflake Marketplace.
- B. Create a reader account.
- C. Create a consumer account.
- D. Use a standard Snowflake share.

<details><summary>Show Answer</summary>
Correct Answer: B. As with Question 3, a reader account is the mechanism for sharing with someone outside the Snowflake ecosystem.
</details>

---

### Question 68
A user wants to add additional privileges to the system-defined roles for their virtual warehouse. How does Snowflake recommend they accomplish this?

- A. Grant the additional privileges to a custom role.
- B. Grant the additional privileges directly to the ACCOUNTADMIN role.
- C. Grant the additional privileges directly to the SYSADMIN role.
- D. Grant the additional privileges directly to the ORGADMIN role.

<details><summary>Show Answer</summary>
Correct Answer: A. Best practice is to avoid directly modifying system-defined roles; instead, create a custom role, grant it the needed privileges, and assign that role (or slot it into the role hierarchy) as appropriate.
</details>

---

### Question 69
How does Snowflake store a table's underlying data? (Choose two.)

- A. Columnar file format
- B. Micro-partitions
- C. Row-oriented flat files
- D. Uncompressed
- E. User-defined partitions

<details><summary>Show Answer</summary>
Correct Answer: A and B. Table data is stored in a compressed, columnar format, organized into automatically-managed micro-partitions — not user-defined partitions, and not uncompressed.
</details>

---

### Question 70
What is the MAXIMUM number of days a Snowflake-managed encryption key can be used before it is automatically rotated?

- A. 1 day
- B. 14 days
- C. 30 days
- D. 120 days

<details><summary>Show Answer</summary>
Correct Answer: C. Snowflake automatically rotates active encryption keys once they are 30 days old.
</details>

---

### Question 71
Which user object property requires contacting Snowflake Support in order to set a value for it?

- A. DISABLED
- B. MFA-related properties (e.g., DISABLE_MFA, MINS_TO_BYPASS_MFA)
- C. MINS_TO_BYPASS_NETWORK_POLICY
- D. DEFAULT_ROLE

<details><summary>Show Answer</summary>
Correct Answer: C. **⚠ Updated/clarified:** the property that specifically requires Snowflake Support to set is `MINS_TO_BYPASS_NETWORK_POLICY`, which temporarily bypasses an active network policy for a user. MFA-related properties like DISABLE_MFA and MINS_TO_BYPASS_MFA can actually be set directly by an ACCOUNTADMIN via `ALTER USER` — no Support ticket required. *(The original scan's options were largely unreadable; reconstructed and verified via current Snowflake documentation.)*
</details>

---

### Question 72
How does Snowflake handle the bulk unloading of data into single or multiple files?

- A. It assigns each unloaded data file a unique, system-generated name.
- B. It uses the PUT command to download the data by default.
- C. It uses COPY INTO for bulk unloading, with SINGLE as the default option.
- D. It uses `COPY INTO [location]` to copy data from a table into one or more files in an external stage.

<details><summary>Show Answer</summary>
Correct Answer: D. `COPY INTO <location>` is the unload command; by default it writes multiple files (SINGLE = FALSE), each with a generated unique name, unless configured otherwise.
</details>

---

### Question 73
What information is displayed in the Query Profile? (Choose two.)

- A. Index hints used in the query
- B. Credit usage details
- C. Clustering key details
- D. Details and statistics for the overall query
- E. A graphical representation of the query processing plan

<details><summary>Show Answer</summary>
Correct Answer: D and E. Query Profile shows overall query statistics and a graphical execution plan; Snowflake has no "index hints" concept, and it doesn't display raw credit usage or clustering key details directly in this view.
</details>

---

### Question 74
A Snowflake user wants to optimize performance for a query that returns only a small number of rows from a table, where each returned row requires significant processing, and the underlying data does not change frequently. What should the user do?

- A. Add a clustering key to the table.
- B. Add the Search Optimization Service to the table.
- C. Create a materialized view based on the query.
- D. Enable the Query Acceleration Service for the virtual warehouse.

<details><summary>Show Answer</summary>
Correct Answer: C. When results are expensive to compute but the source data is fairly static, a materialized view precomputes and stores the result, avoiding repeated heavy processing.
</details>

---

### Question 75
When using the ALLOW_CLIENT_MFA_CACHING parameter, how long is a cached Multi-Factor Authentication (MFA) token valid for?

- A. 1 hour
- B. 2 hours
- C. 4 hours
- D. 8 hours

<details><summary>Show Answer</summary>
Correct Answer: C. A cached MFA token is valid for up to four hours (confirmed current in Snowflake documentation as of July 2026).
</details>

---

### Question 76
When unloading data, which file formats are supported by the `COPY INTO [location]` command? (Choose two.)

- A. Avro
- B. JSON
- C. ORC
- D. Parquet
- E. XML

<details><summary>Show Answer</summary>
Correct Answer: B and D. Unloading via COPY INTO supports delimited (CSV/TSV), JSON, and Parquet. Avro, ORC, and XML are supported for *loading* but not for *unloading*.
</details>

---

### Question 77
A JSON object is loaded into a column named `data` using Snowflake's VARIANT data type. The root node of the object is `BIKE`, with a child attribute `BIKEID`. Which statement will allow the user to access `BIKEID`?

- A. `SELECT data.BIKEID`
- B. `SELECT data.BIKE.BIKEID`
- C. `SELECT data:BIKE.BIKEID`
- D. `SELECT data::BIKE:BIKEID`

<details><summary>Show Answer</summary>
Correct Answer: C. Snowflake uses a colon (`:`) to enter a VARIANT column, then dot notation for nested object attributes: `column:path.to.attribute`.
</details>

---

### Question 78
A custom role owns multiple tables. If this role is dropped from the system, who becomes the owner of these tables?

- A. ACCOUNTADMIN
- B. SYSADMIN
- C. The tables become standalone/orphaned.
- D. The role that dropped the custom role.

<details><summary>Show Answer</summary>
Correct Answer: D. When a role is dropped, ownership of objects it owned transfers to the role that executed the DROP ROLE statement.
</details>

---

### Question 79
Which function produces a lateral view of a VARIANT column?

- A. GET_PATH
- B. FLATTEN
- C. GET
- D. PARSE_JSON

<details><summary>Show Answer</summary>
Correct Answer: B. FLATTEN explodes a VARIANT/ARRAY/OBJECT column into multiple rows via a lateral join, commonly used to unnest semi-structured data.
</details>

---

### Question 80
Snowflake strongly recommends that all users with what type of role be required to use Multi-Factor Authentication (MFA)?

- A. USERADMIN
- B. ACCOUNTADMIN
- C. SECURITYADMIN
- D. SYSADMIN

<details><summary>Show Answer</summary>
Correct Answer: B. ACCOUNTADMIN is the highest-privilege role, so Snowflake strongly recommends MFA for anyone holding it. (Note: as of 2024–2026 policy changes, Snowflake has also moved toward requiring MFA for all human password users generally — but ACCOUNTADMIN remains the specific role called out in this classic exam question.)
</details>

---

### Question 81
What does it mean when the SAMPLE function uses the Bernoulli sampling method?

- A. Each row is considered independently for inclusion, with equal probability (like a per-row coin flip).
- B. Sampling is based on entire micro-partitions/blocks of the source data.
- C. Sampling is based on 1,000 rows of the source data.
- D. Sampling is deterministic and always returns the same rows.

<details><summary>Show Answer</summary>
Correct Answer: A. Bernoulli (row) sampling evaluates each row independently, unlike System (block) sampling, which is faster but samples at the micro-partition level and is less statistically uniform for small tables.
</details>

---

### Question 82
What are characteristics of Snowflake network policies? (Choose two.)

- A. They can be set for any Snowflake Edition.
- B. They can be applied to roles.
- C. They restrict or enable access from specific IP addresses.
- D. They are activated using an ALTER DATABASE SQL command.
- E. They can only be managed using the ORGADMIN role.

<details><summary>Show Answer</summary>
Correct Answer: A and C. Network policies are available on every Snowflake edition and work via IP allow/block lists; they attach to accounts or users (not roles or databases), and are managed via `ALTER ACCOUNT`/`ALTER USER`, not `ALTER DATABASE`.
</details>

---

### Question 83
Which function should be used to find the query ID of the second query executed in the current session?

- A. `SELECT LAST_QUERY_ID(-2);`
- B. `SELECT LAST_QUERY_ID(2);`
- C. `SELECT CURRENT_QUERY_ID(2);`
- D. `SELECT QUERY_HISTORY(2);`

<details><summary>Show Answer</summary>
Correct Answer: B. `LAST_QUERY_ID(<n>)` with a positive index counts forward from the start of the session, so `LAST_QUERY_ID(2)` returns the second query's ID; a negative index counts backward from the most recent query.
</details>

---

### Question 84
How is the hierarchy of database objects organized in Snowflake?

- A. A database consists of one or more schemas. A schema contains tables and views.
- B. A schema consists of one or more databases. A database contains tables and views.
- C. A schema consists of one or more databases. A database contains tables, views, and warehouses.
- D. A database consists of one or more schemas and warehouses. A schema contains tables and views.

<details><summary>Show Answer</summary>
Correct Answer: A. The hierarchy is Account → Database → Schema → Objects (tables, views, etc.). Virtual warehouses are account-level compute objects, not contained within a database.
</details>

---

### Question 85
Which role can successfully execute the `SHOW ORGANIZATION ACCOUNTS` command?

- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. ORGADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: C. Organization-level commands like `SHOW ORGANIZATION ACCOUNTS` require the ORGADMIN role.
</details>

---

## A note on this reconstruction

The source scan was extremely degraded in places — some option lists were duplicated, truncated, or unreadable, and roughly a dozen "Correct Answer" fields were blank or reduced to a single stray character. Where that happened, options and/or answers above were **reconstructed** to the most standard, plausible version of that exam question based on current Snowflake documentation, and flagged inline. Question 13 and Question 55 in particular could not be reliably reconstructed and are flagged as low-confidence — treat those two as "verify against a clean source" rather than "trust as-is."

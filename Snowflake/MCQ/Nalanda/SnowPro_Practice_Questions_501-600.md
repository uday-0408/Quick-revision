# SnowPro Core Practice Questions (501–600)

---

### Question 501
Which view can be used to determine if a table has frequent row updates or deletes?

- A. TABLES
- B. TABLE_STORAGE_METRICS
- C. DATABASE_USAGE
- D. STORAGE_USAGE

<details><summary>Show Answer</summary>
Correct Answer: B. TABLE_STORAGE_METRICS reports row counts, deleted rows, and other storage details that reveal update/delete activity.
</details>

---

### Question 502
How does the Snowflake search optimization service improve query performance?

- A. It improves the performance of range searches.
- B. It defines different clustering keys on the same source table.
- C. It improves the performance of all queries running against a given table.
- D. It improves the performance of equality point lookup searches.

<details><summary>Show Answer</summary>
Correct Answer: D
</details>

---

### Question 503
How is unstructured data retrieved from data storage?

- A. SQL functions like the GET command can be used to copy the unstructured data to a location on the client.
- B. SQL functions can be used to create different types of URLs pointing to the unstructured data. These URLs can be used to download the data to a client.
- C. SQL functions can be used to retrieve the data from the query results cache. When the query results are output to a client, the unstructured data will be output to the client as files.
- D. SQL functions can call on different web extensions designed to display different types of files as a web page. The web extensions will allow the files to be downloaded to the client.

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 504
What is the recommended way to obtain a cloned table with the same grants as the source table?

- A. Clone the table with the COPY GRANTS command.
- B. Use an ALTER TABLE command to copy the grants.
- C. Clone the schema then drop the unwanted tables.
- D. Create a script to extract grants and apply them to the cloned table.

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 505
What common query issues can be identified using the Query Profile? (Choose two.)

- A. Data Classification
- B. Exploding joins
- C. Unions
- D. Inefficient pruning
- E. Data masking

<details><summary>Show Answer</summary>
Correct Answer: B, D
</details>

---

### Question 506
What is used to extract the content of PDF files stored in Snowflake Stages?

- A. FLATTEN function
- B. Window function
- C. HyperLogLog (HLL) function
- D. Java User-Defined Function (UDF)

<details><summary>Show Answer</summary>
Correct Answer: D (a Java UDF using a library such as Apache PDFBox is the traditional approach).

**⚠ Updated:** Snowflake now offers a native, no-code option: the Cortex **AI_PARSE_DOCUMENT** function (formerly PARSE_DOCUMENT), which extracts text and layout from PDF, DOCX, and other document types directly from a stage using SQL — no custom UDF required. The Java UDF approach in option D is still valid and is likely the exam's intended answer, but current Snowflake documentation now recommends AI_PARSE_DOCUMENT for this task.
</details>

---

### Question 507
What is used to extract the content of PDF files stored in Snowflake stages?

- A. FLATTEN function
- B. Window function
- C. HyperLogLog (HLL) function
- D. Java User-Defined Function (UDF)

<details><summary>Show Answer</summary>
Correct Answer: D (duplicate of Question 506 — see note above about AI_PARSE_DOCUMENT).
</details>

---

### Question 508
What happens when a database is cloned?

- A. It does not retain privileges granted on the source.
- B. It replicates all granted privileges on the corresponding source objects.
- C. It replicates all granted privileges on the corresponding child objects.
- D. It replicates all granted privileges on the corresponding child schema objects.

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 509
What does a Query Profile provide in Snowflake?

- A. A multi-step query that displays each processing step in the same panel.
- B. A pre-computed data set derived from a query specification and stored for later use.
- C. A graphical representation of the main components of the processing plan for a query.
- D. A collapsible panel in the operator tree pane that lists nodes by execution time in descending order for a query.

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 510
When executing a COPY INTO command, performance can be negatively affected by using which optional parameter on a large number of files?

- A. FILE_FORMAT
- B. PATTERN
- C. VALIDATION_MODE
- D. FILES

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 511
Which URL type should be used to get a permanent URL to a file in a stage?

- A. File URL
- B. Pre-signed URL
- C. Saved URL
- D. Scoped URL

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 512
Which operation will produce an error in Snowflake?

- A. Inserting duplicate values into a PRIMARY KEY column
- B. Inserting a NULL into a column with a NOT NULL constraint
- C. Inserting duplicate values into a column with a UNIQUE constraint
- D. Inserting a value to a FOREIGN KEY column that does not match a value in the column referenced

<details><summary>Show Answer</summary>
Correct Answer: B. Snowflake does not enforce PRIMARY KEY, UNIQUE, or FOREIGN KEY constraints by default (they are informational only); NOT NULL is the only one enforced.
</details>

---

### Question 513
How are URLs that access unstructured data in external stages retrieved?

- A. Using the navigation menu
- B. By querying a directory table
- C. By creating an external function
- D. By using the INFORMATION_SCHEMA

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 514
What is the Snowflake multi-clustering feature for virtual warehouses used for?

- A. To improve the data unloading process to the cloud
- B. To improve data loading from very large data sets
- C. To improve concurrency for users and queries
- D. To speed up slow or stalled queries

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 515
Which features could be used to improve the performance of queries that return a small subset of rows from a large table? (Choose two.)

- A. Search optimization service
- B. Automatic clustering
- C. Row access policies
- D. Multi-cluster warehouses
- E. Secure views

<details><summary>Show Answer</summary>
Correct Answer: A, B
</details>

---

### Question 516
Which command would return an empty sample?

- A. select * from testtable sample
- B. select * from testtable sample (0);
- C. select * from testtable sample (null);
- D. select * from testtable sample (none);

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 517
Which Snowflake function should be used to unload relational data to JSON?

- A. TO_JSON()
- B. OBJECT_CONSTRUCT()
- C. PARSE_JSON()
- D. JSON_EXTRACT()

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 518
Floating point values are truncated when unloaded to which file format?

- A. ORC
- B. CSV
- C. Avro
- D. Parquet

<details><summary>Show Answer</summary>
Correct Answer: B. Floating-point columns unloaded to CSV (or JSON) are truncated to approximately (15,9) precision; Parquet is not affected.
</details>

---

### Question 519
Which levels can apply network policies? (Choose two.)

- A. Account
- B. Database
- C. Role
- D. Schema
- E. User

<details><summary>Show Answer</summary>
Correct Answer: A, E
</details>

---

### Question 520
What causes objects in a data Share to become unavailable to a consumer account?

- A. The parameter in the consumer account is set to 0.
- B. The consumer account runs the GRANT IMPORTED PRIVILEGES command on the data share every 24 hours.
- C. The objects in the data share are being deleted and the grant pattern is not re-applied.
- D. The consumer account acquires the data share through a private data exchange.

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 521
How can an administrator view updates (for example, SCIM API requests) sent to Snowflake by the identity provider?

- A. ACCESS_HISTORY
- B. LOAD_HISTORY
- C. QUERY_HISTORY
- D. REST_EVENT_HISTORY

<details><summary>Show Answer</summary>
Correct Answer: D. The REST_EVENT_HISTORY table function (in INFORMATION_SCHEMA) returns SCIM REST API requests made to Snowflake over a specified time interval.
</details>

---

### Question 522
A Snowflake user is writing a User-Defined Function (UDF) with some unqualified object names. How will those object names be resolved during execution?

- A. Snowflake will resolve them according to the SEARCH_PATH parameter.
- B. Snowflake will only check the schema the UDF belongs to.
- C. Snowflake will first check the current schema, and then the schema the previous query used.
- D. Snowflake will first check the current schema, and then the PUBLIC schema of the current database.

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 523
Why should a user select the economy scaling policy for a multi-cluster warehouse?

- A. To prevent/minimize query queuing.
- B. To increase performance of the clusters.
- C. To reduce queuing for concurrent user queries.
- D. To conserve credits by keeping running clusters fully loaded.

<details><summary>Show Answer</summary>
Correct Answer: D
</details>

---

### Question 524
What MINIMUM privilege is required on the external Stage for any role in the GET REST API to access unstructured data files using a file URL?

- A. READ
- B. OWNERSHIP
- C. USAGE
- D. WRITE

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 525
Which view in SNOWFLAKE.ACCOUNT_USAGE shows from which IP address a user connected to Snowflake?

- A. ACCESS_HISTORY
- B. LOGIN_HISTORY
- C. SESSIONS
- D. QUERY_HISTORY

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 526
Snowflake Partner Connect is limited to users with a verified email address and which role?

- A. SYSADMIN
- B. SECURITYADMIN
- C. ACCOUNTADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 527
What unit of storage supports efficient query processing in Snowflake?

- A. Blobs
- B. JSON
- C. Block Storage
- D. Micro-partitions

<details><summary>Show Answer</summary>
Correct Answer: D
</details>

---

### Question 528
What is the difference between a stored procedure and a User-Defined Function (UDF)?

- A. Stored procedures can perform database operations while UDFs cannot.
- B. Returning a value is required in a stored procedure while returning values in a UDF is optional.
- C. Values returned by a stored procedure can be used directly in a SQL statement while the values returned by a UDF cannot.
- D. Multiple stored procedures can be called as part of a single executable statement while a single SQL statement can only call one UDF.

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 529
Which URL type does Snowflake recommend to use when providing unstructured data to other accounts through a Share?

- A. File URL
- B. Pre-signed URL
- C. Scoped URL
- D. Direct URL

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 530
What is the MAXIMUM Time Travel retention period for a transient table?

- A. 0 days
- B. 1 day
- C. 7 days
- D. 90 days

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 531
What is the advantage of using a reader account?

- A. It can be used by a client that does not have a Snowflake account.
- B. It is read-only and prevents the shared data from being updated by the provider.
- C. It can be connected to a Snowflake account in a different region.
- D. It provides limited access to the data share and is therefore cheaper for the data provider.

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 532
What command is used to export or unload data from Snowflake?

- A. PUT @mystage
- B. GET @mystage
- C. COPY INTO @myStage
- D. INSERT INTO @mystage

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 533
A Snowflake user wants to share data with someone who does not have a Snowflake account. How can the Snowflake user share the data?

- A. Use the Snowflake Marketplace.
- B. Create a reader account.
- C. Create a consumer account.
- D. Use a Snowflake share.

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 534
A user wants to add additional privileges to the system-defined roles for their virtual warehouse. How does Snowflake recommend they accomplish this?

- A. Grant the additional privileges to a custom role, then grant the custom role to the system role.
- B. Grant the additional privileges directly to the ACCOUNTADMIN role.
- C. Grant the additional privileges directly to the SYSADMIN role.
- D. Grant the additional privileges directly to the ORGADMIN role.

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 535
How does Snowflake store a table's underlying data? (Choose two.)

- A. Columnar file format
- B. Micro-partitions
- C. Text file format
- D. Uncompressed
- E. User-defined partitions

<details><summary>Show Answer</summary>
Correct Answer: A, B
</details>

---

### Question 536
What is the MAXIMUM number of days a Snowflake-managed encryption key can be used before it gets automatically rotated?

- A. 1 day
- B. 14 days
- C. 30 days
- D. 120 days

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 537
Which user object property requires contacting Snowflake Support in order to set a value for it?

- A. DISABLED
- B. DEFAULT_ROLE
- C. MINS_TO_BYPASS_MFA
- D. PASSWORD

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 538
How does Snowflake handle the bulk unloading of data into single or multiple files?

- A. It assigns each unloaded data file a unique name.
- B. It uses the PUT command to download the data by default.
- C. It uses COPY INTO for bulk unloading where the default option is SINGLE = TRUE.
- D. It uses COPY INTO [location] to copy the data from a table into one or more files in an external stage.

<details><summary>Show Answer</summary>
Correct Answer: D
</details>

---

### Question 539
What information is included in the display in the Query Profile? (Choose two.)

- A. Index hints used in query
- B. Credit usage details
- C. Clustering keys details
- D. Details and statistics for the overall query
- E. Graphical representation of the query processing plan

<details><summary>Show Answer</summary>
Correct Answer: D, E
</details>

---

### Question 540
A Snowflake user wants to optimize performance for a query that queries only a small number of rows in a table. The rows require significant processing. The data in the table changes frequently. What should the user do?

- A. Add a clustering key to the table.
- B. Add the search optimization service to the table.
- C. Create a materialized view based on the query.
- D. Enable the query acceleration service for the virtual warehouse.

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 541
When using the ALLOW_CLIENT_MFA_CACHING parameter, how long is a cached Multi-Factor Authentication (MFA) token valid for?

- A. 1 hour
- B. 2 hours
- C. 4 hours
- D. 8 hours

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 542
When unloading data, which file formats are supported by the COPY INTO [location] command? (Choose two.)

- A. Avro
- B. JSON
- C. ORC
- D. Parquet
- E. XML

<details><summary>Show Answer</summary>
Correct Answer: B, D. Unloading supports CSV, JSON, and Parquet only — Avro, ORC, and XML are supported for loading but not unloading.
</details>

---

### Question 543
A JSON object is loaded into a column named `data` using a Snowflake variant datatype. The root node of the object is `BIKE`. The child attribute for this node is `BIKEID`. Which statement will allow the user to access BIKEID?

- A. select data_BIKE_ID
- B. select data.BIKE.BIKEID
- C. select data:BIKE.BIKEID
- D. select data::BIKEID

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 544
A custom role owns multiple tables. If this role is dropped from the system, who becomes the owner of these tables?

- A. ACCOUNTADMIN
- B. SYSADMIN
- C. Tables become standalone or orphaned
- D. The role that dropped the custom role.

<details><summary>Show Answer</summary>
Correct Answer: D
</details>

---

### Question 545
Which function produces a lateral view of a VARIANT column?

- A. GET_PATH
- B. FLATTEN
- C. GET
- D. PARSE_JSON

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 546
Snowflake strongly recommends that all users with what type of role be required to use Multi-Factor Authentication (MFA)?

- A. USERADMIN
- B. ACCOUNTADMIN
- C. SECURITYADMIN
- D. SYSADMIN

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 547
What does it mean when the SAMPLE function uses the Bernoulli sampling method?

- A. The data is based on sampling every row with a specific probability.
- B. The data is based on sampling of the entire source data as a block.
- C. The data is based on sampling blocks of the source data.
- D. The data is based on sampling exactly 1000 rows of the source data.

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 548
What are characteristics of Snowflake network policies? (Choose two.)

- A. They can be set for any Snowflake Edition.
- B. They can be applied directly to roles.
- C. They restrict or enable specific IP addresses.
- D. They are activated using ALTER DATABASE SQL commands.
- E. They can only be managed using the ORGADMIN role.

<details><summary>Show Answer</summary>
Correct Answer: A, C
</details>

---

### Question 549
Which function should be used to find the query ID of the second query executed in a current session?

- A. SELECT LAST_QUERY_ID()
- B. SELECT QUERY_ID
- C. SELECT LAST_QUERY_ID(-2)
- D. SELECT LAST_QUERY_ID(2)

<details><summary>Show Answer</summary>
Correct Answer: D
</details>

---

### Question 550
How is the hierarchy of database objects organized in Snowflake?

- A. A database consists of one or more schemas. A schema contains tables and views.
- B. A schema consists of one or more databases. A database contains tables and views.
- C. A schema consists of one or more databases. A database contains tables, views, and warehouses.
- D. A database consists of one or more schemas and warehouses. A schema contains tables and views.

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 551
Which role can execute the SHOW ORGANIZATION ACCOUNTS command successfully?

- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. ORGADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 552
Which data types in Snowflake are synonymous for FLOAT? (Choose two.)

- A. DECIMAL
- B. DOUBLE
- C. NUMBER
- D. NUMERIC
- E. REAL

<details><summary>Show Answer</summary>
Correct Answer: B, E. FLOAT, FLOAT4, FLOAT8, DOUBLE, DOUBLE PRECISION, and REAL are all synonymous and implemented as 64-bit double-precision floating point in Snowflake.
</details>

---

### Question 553
What ensures that a user with the role SECURITYADMIN can activate a network policy for an individual user?

- A. A role that has been granted the EXECUTE TASK privilege
- B. A role that has been granted the global ATTACH POLICY privilege
- C. Ownership privilege on only the role that created the network policy
- D. Ownership privilege on both the user and the network policy

<details><summary>Show Answer</summary>
Correct Answer: D
</details>

---

### Question 554
Which function can be combined with the COPY command to unload a relational table into a JSON file?

- A. FLATTEN
- B. LISTAGG
- C. OBJECT_CONSTRUCT
- D. PARSE_JSON

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 555
A user needs to MINIMIZE the cost of large tables that are used to store transitory data. The data does not need to be protected against failures because the data can be reconstructed outside of Snowflake. What table type should be used?

- A. Permanent
- B. Transient
- C. Temporary
- D. External

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 556
While loading data from a JSON file, what enables the removal of the outer array structure from the file and loads the records into separate table rows?

- A. FLATTEN
- B. ARRAY_CONSTRUCT
- C. STRIP_OUTER_ARRAY = TRUE
- D. PURGE_ARRAY = TRUE

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 557
Which functions can be used to share unstructured data through a secure view? (Choose two.)

- A. BUILD_SCOPED_FILE_URL
- B. GET_PRESIGNED_URL
- C. BUILD_STAGE_FILE_URL
- D. SYSTEM$AUTHORIZE_PRIVATELINK

<details><summary>Show Answer</summary>
Correct Answer: A, B
</details>

---

### Question 558
Which function will return a row for each object in a VARIANT, OBJECT, or ARRAY column?

- A. CAST
- B. FLATTEN
- C. GET
- D. PARSE_JSON

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 559
What is the MINIMUM size of a table for which Snowflake recommends considering adding a clustering key?

- A. 1 Kilobyte (KB)
- B. 1 Megabyte (MB)
- C. 1 Gigabyte (GB)
- D. 1 Terabyte (TB)

<details><summary>Show Answer</summary>
Correct Answer: D
</details>

---

### Question 560
Using the ALLOWED_VALUES tag property, what is the MAXIMUM number of possible string values for a single tag?

- A. 10
- B. 50
- C. 100
- D. 300

<details><summary>Show Answer</summary>
Correct Answer: D
</details>

---

### Question 561
Which Snowflake table type is only visible to the user who creates it, can have the same name as permanent tables in the same schema, and is dropped at the end of the session?

- A. Temporary
- B. Local
- C. User
- D. Transient

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 562
What is a characteristic of a role in Snowflake?

- A. Roles cannot be granted to other roles.
- B. System-defined roles can be dropped.
- C. Privileges granted to system roles by Snowflake can be revoked.
- D. Privileges on securable objects can be granted to a role.

<details><summary>Show Answer</summary>
Correct Answer: D
</details>

---

### Question 563
What command would a user execute to load unstructured data files into a Snowflake internal stage?

- A. PUT
- B. GET
- C. LIST
- D. COPY INTO

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 564
How do managed access schemas help with data governance?

- A. They log all operations and enable fine-grained auditing.
- B. They provide centralized privilege management with the schema owner.
- C. They enforce identical privileges across all tables and views in a schema.
- D. They require the use of masking and row access policies across every table and view in the schema.

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 565
What is the default period of time the Warehouse Activity section provides a graph of Snowsight activity?

- A. 2 hours
- B. 1 week
- C. 14 days (2 weeks)
- D. 1 month

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 566
A Snowflake user wants to unload data from a relational table sized 5 GB using CSV. The extract needs to be as performant as possible. What should they do?

- A. Use Parquet as the unload file format, using Parquet's default compression feature.
- B. Use a regular expression in the stage of the COPY command to restrict parsing time.
- C. Increase the default file size to 5 GB and set SINGLE = true to produce a single file.
- D. Leave the default max file size to 16 MB to take advantage of parallel operations.

<details><summary>Show Answer</summary>
Correct Answer: D
</details>

---

### Question 567
How is the MANAGE GRANTS privilege applied?

- A. Globally
- B. At the database level
- C. At the schema level
- D. At the table level

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 568
What is required for a query execution to be served from the result cache?

- A. The query logic is the same.
- B. The exact SQL text is the same.
- C. The SQL profile is the same.
- D. The virtual warehouse is the same.

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 569
Which Snowflake URL type is used by directory tables?

- A. File URL
- B. Pre-signed URL
- C. Scoped URL
- D. Virtual-hosted style

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 570
At which point is data encrypted when using a PUT command?

- A. When it reaches the virtual warehouse
- B. When it gets micro-partitioned
- C. Client-side before it is sent from the user's machine
- D. After it reaches the internal stage

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 571
Which privileges are required for a user to restore a dropped object using UNDROP? (Choose two.)

- A. UPDATE
- B. OWNERSHIP on the object
- C. MODIFY
- D. USAGE
- E. CREATE on the schema

<details><summary>Show Answer</summary>
Correct Answer: B, E
</details>

---

### Question 572
For a virtual warehouse, which parameters are used to calculate the number of credits billed? (Choose two.)

- A. Cache size
- B. Warehouse Size
- C. Number of clusters running
- D. Volume of data processed
- E. Number of queries executed

<details><summary>Show Answer</summary>
Correct Answer: B, C
</details>

---

### Question 573
What happens when the values for both an allowed list and a blocked list are used in a network policy?

- A. Snowflake ignores the first list.
- B. Snowflake applies the blocked list first.
- C. Snowflake applies the allowed list first.
- D. Snowflake throws an error.

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 574
What does the orange bar on an operator represent when reviewing the Query Profile?

- A. A percentage of progress of the operator's completion.
- B. The fraction of time that this operator consumed within the query step.
- C. The cost of the operator in terms of the virtual warehouse CPU utilization.
- D. The fraction of data scanned from cache versus remote disk for the operator.

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 575
When unloading data from Snowflake, what is the default max file size of each file?

- A. 16 MB
- B. 32 MB
- C. 5 GB
- D. Unlimited

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 576
What is the abbreviated form to get a list of all the files in the user stage?

- A. LIST
- B. LS @~;
- C. LS @usr;
- D. SHOW

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 577
Which features make up Snowflake's column-level security? (Choose two.)

- A. Continuous Data Protection (CDP)
- B. Dynamic Data Masking
- C. External Tokenization
- D. Key pair authentication
- E. Row access policies

<details><summary>Show Answer</summary>
Correct Answer: B, C
</details>

---

### Question 578
Which languages are supported for writing Snowflake UDFs? (Choose two.)

- A. JavaScript
- B. Python
- C. C++
- D. PHP
- E. TypeScript

<details><summary>Show Answer</summary>
Correct Answer: A, B. Snowflake also supports SQL, Java, and Scala for UDFs, but among these options only JavaScript and Python are valid.
</details>

---

### Question 579
What is the MAXIMUM number of days that Snowflake resets the 24-hour retention period for a query result every time the result is used?

- A. 1 day
- B. 14 days
- C. 31 days
- D. 60 days

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 580
There are 300 concurrent users on a production Snowflake account using a single cluster virtual warehouse. The queries are small, but the queuing is high. What is causing this to occur?

- A. The single cluster warehouse is queuing the queries because it cannot process 300 concurrent requests, increasing the overall query execution time.
- B. The warehouse parameter STATEMENT_QUEUED_TIMEOUT_IN_SECONDS is set too low.
- C. The application is not using the latest native ODBC driver.
- D. The queries are not taking advantage of the data cache.

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 581
Which Snowflake edition offers the highest level of security for organizations that have the strictest requirements?

- A. Standard
- B. Enterprise
- C. Business Critical
- D. Virtual Private Snowflake (VPS)

<details><summary>Show Answer</summary>
Correct Answer: D
</details>

---

### Question 582
What is the MAXIMUM size limit for a record of a VARIANT data type?

- A. 8 MB
- B. 16 MB
- C. 32 MB
- D. 128 MB

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 583
What criteria does Snowflake use to determine the current role when initiating a session? (Choose two.)

- A. If a role was specified as part of the connection and that role has been granted to the Snowflake user, the specified role becomes the current role.
- B. If no role was specified as part of the connection and a default role has been defined for the Snowflake user, that role becomes the current role.
- C. If no role was specified as part of the connection and a default role has not been set for the Snowflake user, the session will not be initiated and the login will fail.
- D. If a role was specified as part of the connection and that role has not been granted to the Snowflake user, it will be ignored and the default role will become the active role.
- E. If a role was specified as part of the connection and that role has not been granted to the Snowflake user, the role is automatically granted and it becomes the current role.

<details><summary>Show Answer</summary>
Correct Answer: A, B
</details>

---

### Question 584
What command should be used to move data from a Snowflake database table into one or more files in an external stage?

- A. GET
- B. COPY INTO
- C. PUT
- D. EXPORT

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 585
How does a Snowflake user reference a directory table created on stage `mystage` in a SQL query?

- A. SELECT * FROM DIRECTORY
- B. SELECT * FROM DIRECTORY(@mystage)
- C. SELECT * FROM TO_TABLE(DIRECTORY @mystage)
- D. SELECT * TABLE(@mystage DIRECTORY)

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 586
Why would a Snowflake user create a secure view instead of a standard view?

- A. The Secure View is only available to end users with the corresponding SECURE_ACCESS property.
- B. End users are unable to see the view definition, and internal optimizations differ with a secure view to protect underlying data.
- C. In a secure view, the underlying data is a separate storage layer with encryption.
- D. Secure views support additional functionality that is not supported for standard views, such as column masking and row level access.

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 587
Which command parameter can be added to the COPY command to make it load all files, whether or not the load status of the files is known?

- A. FORCE = TRUE
- B. FORCE = FALSE
- C. PURGE = TRUE
- D. PURGE = FALSE

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 588
How can a Snowflake user improve long-running query performance?

- A. Reduce the virtual warehouse size.
- B. Cluster the underlying table being queried.
- C. Disable the result cache.
- D. Add ORDER BY to the query.

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

### Question 589
Which Snowflake feature allows administrators to identify unused data that may be archived or deleted?

- A. Access History
- B. Data classification
- C. Dynamic Data Masking
- D. Object tagging

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 590
Which SQL commands should be used to write a recursive query if the number of levels is unknown? (Choose two.)

- A. CONNECT BY
- B. LISTAGG
- C. MATCH RECOGNIZE
- D. QUALIFY
- E. WITH RECURSIVE

<details><summary>Show Answer</summary>
Correct Answer: A, E
</details>

---

### Question 591
What information is stored in the ACCESS_HISTORY view?

- A. History of the files that have been loaded into Snowflake.
- B. Names and owners of the roles that are currently enabled in the session.
- C. Query details such as the objects read/modified and the user who executed the query.
- D. Details around the privileges that have been granted for all objects in an account.

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 592
What privilege does a user need in order to receive or request data from the Snowflake Marketplace?

- A. CREATE DATA EXCHANGE LISTING
- B. CREATE SHARE
- C. IMPORT SHARE
- D. IMPORTED PRIVILEGES

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 593
Which Snowflake database object can be shared with other accounts?

- A. Tasks
- B. Pipes
- C. Secure User-Defined Functions (UDFs)
- D. Stored Procedures

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 594
Which identity providers are valid type values for federated authentication on the security integration parameter? (Choose two.)

- A. Identity Access Management (IAM)
- B. Microsoft Active Directory Federation Services (AD FS)
- C. OAuth
- D. Okta
- E. PingFederate

<details><summary>Show Answer</summary>
Correct Answer: B, D
</details>

---

### Question 595
A Snowflake user wants to share data using my_share with account 12345. Which command should be used?

- A. grant usage on account 12345 to share my_share;
- B. grant select on share my_share to account 12345;
- C. alter share my_share add accounts = 12345;
- D. alter account 12345 add share my_share;

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 596
What role is required to use Partner Connect?

- A. ACCOUNTADMIN
- B. ORGADMIN
- C. SECURITYADMIN
- D. SYSADMIN

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 597
How can a Snowflake user configure a virtual warehouse to support over 100 users if their company has Enterprise Edition?

- A. Add additional warehouses and configure them as a pool.
- B. Set the auto-scale to 100.
- C. Use a multi-cluster warehouse.
- D. Use a larger warehouse.

<details><summary>Show Answer</summary>
Correct Answer: C
</details>

---

### Question 598
How is table data compressed in Snowflake?

- A. Each column is compressed as it is stored in a micro-partition.
- B. Each micro-partition is compressed as it is written into cloud storage using GZIP.
- C. The micro-partitions are stored in compressed Cloud Storage and the Cloud Storage handles it.
- D. The text data in a micro-partition is compressed with GZIP but other types are not compressed.

<details><summary>Show Answer</summary>
Correct Answer: A
</details>

---

### Question 599
What will be the output of the below query against the table name gold_data?

`select * from gold_data tablesample (100);`

- A. It will return an empty sample.
- B. It will return a 100 row sample.
- C. It will return the entire table.
- D. It will produce an error message.

<details><summary>Show Answer</summary>
Correct Answer: C. Without the ROWS keyword, the number is interpreted as a percentage probability (Bernoulli sampling); 100% probability returns (statistically) the entire table.
</details>

---

### Question 600
A Snowflake query took 40 minutes to run. The results indicate that 'Bytes spilled to local storage' was a large number. What is the issue and how can it be resolved?

- A. The warehouse is too large. Decrease the size of the warehouse to reduce the spillage.
- B. The warehouse is too small. Increase the size of the warehouse to reduce the spillage.
- C. The Snowflake console has timed-out. Contact Snowflake Support.
- D. The warehouse consists of a single cluster. Use a multi-cluster warehouse to reduce the spillage.

<details><summary>Show Answer</summary>
Correct Answer: B
</details>

---

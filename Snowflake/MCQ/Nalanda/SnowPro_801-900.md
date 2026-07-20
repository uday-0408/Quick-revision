# SnowPro Core Practice Questions — Batch 9 (Q801–Q900)

*Cleaned, reformatted, and cross-checked against current Snowflake documentation (as of July 2026). Answers are hidden in collapsible blocks — click "Show Answer" to reveal. Items with a ⚠ **Updated** note had an answer correction or an important clarification versus the original source.*

---

### Question 801
What does the Activity area of Snowsight allow users to do? (Choose two.)
- A. Schedule automated data backups.
- B. Monitor each step of an executed query.
- C. Monitor queries executed by users in an account.
- D. Create and manage user roles and permissions.
- E. Explore Snowflake Marketplace to find and integrate data.

<details><summary>Show Answer</summary>
Correct Answer: B, C. The Activity area (Query History / Copy History) lets you inspect the execution steps of a query and monitor queries run by others in the account.
</details>

---

### Question 802
In which Snowsight section can a user switch roles, modify their profile, and access documentation?
- A. The user menu
- B. The activity page
- C. The content pane
- D. The worksheets

<details><summary>Show Answer</summary>
Correct Answer: A. The user menu.
</details>

---

### Question 803
What is the recommended way to change the existing file format type in my_format from CSV to JSON?
- A. ALTER FILE FORMAT my_format SET TYPE=JSON;
- B. ALTER FILE FORMAT my_format SWAP TYPE WITH JSON;
- C. CREATE OR REPLACE FILE FORMAT my_format TYPE=JSON;
- D. REPLACE FILE FORMAT my_format TYPE=JSON;

<details><summary>Show Answer</summary>
Correct Answer: C. ALTER FILE FORMAT cannot change the TYPE property — the file format must be recreated.
</details>

---

### Question 804
Which features are included in Snowsight? (Choose two.)
- A. Worksheet sharing
- B. Referencing SnowSQL
- C. Exploring the Marketplace
- D. Changing the Snowflake account cloud provider
- E. Downloading query result data larger than 100 MB

<details><summary>Show Answer</summary>
Correct Answer: A, C.
</details>

---

### Question 805
How long can data that has a pre-signed URL access data files using Snowflake?
- A. Indefinitely
- B. Until the session ends
- C. Until the retention_time is met
- D. Until the expiration_time is exceeded

<details><summary>Show Answer</summary>
Correct Answer: D. Pre-signed URLs are generated with a specified expiration time.
</details>

---

### Question 806
What mechanisms can be used to inform Snowpipe that there are staged files available to load into a Snowflake table? (Choose two.)
- A. Cloud messaging
- B. Email integrations
- C. Error notifications
- D. REST endpoints
- E. Snowsight interactions

<details><summary>Show Answer</summary>
Correct Answer: A, D. Snowpipe can be triggered via cloud provider event notifications (auto-ingest) or by calling the REST API endpoints directly.
</details>

---

### Question 807
A Snowflake user needs to import a JSON file larger than 16 MB. What file format option could be used?
- A. trim_space = true
- B. compression = auto
- C. strip_outer_array = true
- D. ignore_utf8_errors = false

<details><summary>Show Answer</summary>
Correct Answer: C. Stripping the outer array lets Snowflake load each element as a separate row, avoiding the single-row VARIANT size limit.
</details>

---

### Question 808
What is a feature of column-level security in Snowflake?
- A. Row access policies
- B. Network policies
- C. Internal tokenization
- D. External tokenization

<details><summary>Show Answer</summary>
Correct Answer: D. Column-level security includes dynamic data masking and external tokenization; row access policies are row-level (not column-level) security.
</details>

---

### Question 809
Which common query problems can the Query Profile help a user identify and troubleshoot? (Choose two.)
- A. When Window functions are used incorrectly
- B. When there are exploding joins
- C. When there is a UNION without ALL
- D. When the SELECT DISTINCT command returns too many values
- E. When there are Common Table Expressions (CTEs) without a final SELECT statement

<details><summary>Show Answer</summary>
Correct Answer: B, C.
</details>

---

### Question 810
What is the Fail-safe retention period for permanent tables?
- A. 0 days
- B. 1 day
- C. 7 days
- D. 90 days

<details><summary>Show Answer</summary>
Correct Answer: C. Fail-safe is a fixed, non-configurable 7-day period following the Time Travel retention period for permanent tables.
</details>

---

### Question 811
Which features can be enabled by calling the SYSTEM$GLOBAL_ACCOUNT_SET_PARAMETER function by a user with the ORGADMIN role? (Choose two.)
- A. Clustering
- B. Client redirect
- C. Fail-safe
- D. Search optimization service
- E. Account and database replication

<details><summary>Show Answer</summary>
Correct Answer: B, E. This function toggles the ENABLE_ACCOUNT_DATABASE_REPLICATION and ENABLE_CLIENT_REDIRECT organization-level parameters.
</details>

---

### Question 812
What are characteristics of directory tables when used with unstructured data? (Choose two.)
- A. Only Cloud Storage Stages support directory tables.
- B. Each directory table has grantable privileges of its own.
- C. Directory tables store a catalog of staged files in cloud storage.
- D. A directory table can be added explicitly to a stage when the stage is created.
- E. A directory table is a separate database object that can be layered explicitly on a stage.

<details><summary>Show Answer</summary>
Correct Answer: C, D. A directory table is not a standalone object with its own privileges — it's an implicit catalog layered onto a stage via the `DIRECTORY = (ENABLE = TRUE)` parameter, either at creation or via ALTER STAGE.
</details>

---

### Question 813
Snowflake best practice recommends that which role be used to enforce a network policy on a Snowflake account?
- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. SYSADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 814
What is the default behavior of internal stages in Snowflake?
- A. Named internal stages are created by default.
- B. Users manually create their internal stages.
- C. Data files are automatically staged to a default location.
- D. Each user and table are automatically allocated an internal stage.

<details><summary>Show Answer</summary>
Correct Answer: D. Every user gets a user stage (`@~`) and every table gets a table stage automatically; named stages must be created manually.
</details>

---

### Question 815
The MAXIMUM size for a serverless task run is equivalent to what size virtual warehouse?
- A. Medium
- B. Large
- C. 2X-Large
- D. 4X-Large

<details><summary>Show Answer</summary>
Correct Answer: C. Confirmed current: Snowflake documentation states the maximum size for a serverless task run is equivalent to an XXLARGE (2X-Large) warehouse.
</details>

---

### Question 816
What storage cost is completely eliminated when a Snowflake table is defined as transient?
- A. Active
- B. Fail-safe
- C. Staged
- D. Time Travel

<details><summary>Show Answer</summary>
Correct Answer: B. Transient tables have no Fail-safe period (0 days); they still incur active storage and (limited) Time Travel costs.
</details>

---

### Question 817
How can a Snowflake user traverse semi-structured data?
- A. Insert a colon (:) between the VARIANT column name and any first-level element.
- B. Insert a colon (:) between the VARIANT column name and any second-level element.
- C. Insert a double colon (::) between the VARIANT column name and any first-level element.
- D. Insert a double colon (::) between the VARIANT column name and any second-level element.

<details><summary>Show Answer</summary>
Correct Answer: A. A single colon accesses the first-level element; dot or bracket notation continues traversal, and `::` is used for type casting, not traversal.
</details>

---

### Question 818
Based on Snowflake recommendations, when creating a hierarchy of custom roles, the top-most custom role should be assigned to which role?
- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. SYSADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 819
What happens to the privileges granted to Snowflake system-defined roles?
- A. The privileges cannot be revoked.
- B. The privileges can be revoked by an ACCOUNTADMIN.
- C. The privileges can be revoked by an ORGADMIN.
- D. The privileges can be revoked by any user-defined role with appropriate privileges.

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 820
By default, which role allows a user to manage a Snowflake Data Exchange share?
- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. SYSADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 821
How does the PARTITION BY option affect an expression for a COPY INTO [location] command?
- A. The unload operation partitions table data into separate files for the specified table.
- B. The unload operation partitions table rows into separate files unloaded to the specified stage.
- C. A single file will be loaded with a user-defined partition key and the user can use this partition key for clustering.
- D. A single file will be loaded with a Snowflake-defined partition key and Snowflake will use this key for pruning.

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 822
How does Snowflake improve the performance of queries that are designed to filter out a significant amount of data?
- A. The use of indexing
- B. The use of pruning
- C. The use of TableScan
- D. By increasing the number of partitions scanned

<details><summary>Show Answer</summary>
Correct Answer: B. Micro-partition metadata allows Snowflake to prune (skip) partitions that can't match the filter.
</details>

---

### Question 823
A JSON document is stored in the source_column of type VARIANT. The document has an array called elements. The array contains the name key that has a string value. How can a Snowflake user extract the name from the first element?
- A. select source_column.elements[0].name
- B. select source_column:elements.name[0]
- C. select source_column:elements[0].name
- D. select source_column.elements.name[0]

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 824
Which function should be used to insert JSON formatted string data into a VARIANT field?
- A. FLATTEN
- B. CHECK_JSON
- C. PARSE_JSON
- D. TO_VARIANT

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 825
Which permission on a Snowflake virtual warehouse allows the role to resize the warehouse?
- A. ALTER
- B. MODIFY
- C. MONITOR
- D. USAGE

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 826
What is it called when a customer managed key is combined with a Snowflake managed key to create a composite key for encryption?
- A. Hierarchical key model
- B. Client-side encryption
- C. Tri-secret Secure encryption
- D. Key pair authentication

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 827
What is the COPY INTO [location] command option default for unloading data into multiple files?
- A. SINGLE = TRUE
- B. SINGLE = NULL
- C. SINGLE = FALSE
- D. MULTIPLE = TRUE

<details><summary>Show Answer</summary>
Correct Answer: C. `SINGLE = FALSE` is the default, which splits unloaded data across multiple files.
</details>

---

### Question 828
A size 3X-Large multi-cluster warehouse runs one cluster for one full hour and then runs two clusters for the next full hour. What would be the total number of credits billed?
- A. 64
- B. 128
- C. 149
- D. 192

<details><summary>Show Answer</summary>
Correct Answer: D. A 3X-Large warehouse consumes 64 credits/hour per cluster. Hour 1: 1 cluster × 64 = 64 credits. Hour 2: 2 clusters × 64 = 128 credits. Total = 64 + 128 = 192 credits.
</details>

---

### Question 829
What is the impact of increasing the number of concurrent clusters on a Snowflake virtual warehouse?
- A. Improved performance for small, simple queries
- B. Improved performance for large, complex queries
- C. Decreased queuing for concurrent queries
- D. Decreased consumption of Snowflake credits

<details><summary>Show Answer</summary>
Correct Answer: C. Multi-cluster warehouses address concurrency/queuing, not single-query performance.
</details>

---

### Question 830
By default, how long is the standard retention period for Time Travel across all Snowflake accounts?
- A. 0 days
- B. 1 day
- C. 7 days
- D. 14 days

<details><summary>Show Answer</summary>
Correct Answer: B. The default is 1 day for all editions (Standard Edition's maximum is also 1 day; Enterprise Edition and above can extend up to 90 days for permanent tables, but the out-of-the-box default is 1 day).
</details>

---

### Question 831
What type of query will benefit from the query acceleration service?
- A. Queries without filters or aggregation
- B. Queries with large scans and selective filters
- C. Queries where the GROUP BY has high cardinality
- D. Queries on tables that have search optimization service enabled

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 832
How does the search optimization service help Snowflake users improve query performance?
- A. By clustering the tables
- B. It maintains a persistent data structure that keeps track of the values of the table's columns in each of its micro-partitions.
- C. It scans the disk cache to avoid scans on the tables used in the query.
- D. It keeps track of running queries and their results and saves those extra scans on the table.

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 833
What can be done to reduce queueing on a virtual warehouse?
- A. Increase the AUTO_SUSPEND setting for the warehouse.
- B. Change the warehouse to a multi-cluster warehouse.
- C. Increase the warehouse size.
- D. Lower the MAX_CONCURRENCY_LEVEL setting on the warehouse.

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 834
What are characteristics of Snowsight worksheets? (Choose two.)
- A. Worksheets can be grouped under folders, and a folder of folders.
- B. Each worksheet is a unique Snowflake session.
- C. Users are limited to running one query on a worksheet.
- D. The Snowflake session ends when a user switches worksheets.
- E. Users can import worksheets and share them with other users.

<details><summary>Show Answer</summary>
Correct Answer: A, B.
</details>

---

### Question 835
What are reasons for using the VALIDATE function after a COPY INTO command execution? (Choose two.)
- A. To validate the files that have been loaded earlier using the COPY INTO command
- B. To view changes that were made during the execution of the COPY command
- C. To return errors encountered during the execution of the COPY INTO command
- D. To identify potential issues in the COPY INTO command before it is executed
- E. To count the number of errors during execution of the COPY INTO command

<details><summary>Show Answer</summary>
Correct Answer: A, C. VALIDATE is a post-load, look-back function (it cannot preview a load before it happens).
</details>

---

### Question 836
Which types of URLs are provided by Snowflake to access unstructured data files? (Choose two.)
- A. Absolute URL
- B. Dynamic URL
- C. File URL
- D. Relative URL
- E. Scoped URL

<details><summary>Show Answer</summary>
Correct Answer: C, E. Snowflake provides File URLs, Scoped URLs, and Pre-signed URLs for unstructured data access.
</details>

---

### Question 837
Which query will return a sample of a table named testtable, in which each row has a 10% probability of being included in the sample?
- A. select * from testtable sample;
- B. select * from testtable sample (10);
- C. select * from testtable sample (10 percent);
- D. select * from testtable sample (10 rows);

<details><summary>Show Answer</summary>
Correct Answer: B. A bare numeric argument to SAMPLE/TABLESAMPLE is interpreted as a percentage (Bernoulli sampling) by default.
</details>

---

### Question 838
Which system can be used to manage access to the data in a share and display certain data only to paying customers?
- A. SYSTEM$ALLOWLIST
- B. SYSTEM$ALLOWLIST_PRIVATELINK
- C. SYSTEM$AUTHORIZE_PRIVATELINK
- D. Data Exchange / Data Marketplace listings

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 839
Which Snowflake object does not consume any storage costs?
- A. Secure View
- B. Materialized view
- C. Temporary table
- D. Transient table

<details><summary>Show Answer</summary>
Correct Answer: A. Views (including secure views) are query definitions only and store no data; materialized views do consume storage since they persist result data.
</details>

---

### Question 840
What does the LATERAL modifier for the FLATTEN function do?
- A. Casts the values of the flattened data
- B. Extracts the path of the flattened data
- C. Joins information outside the object with the flattened data
- D. Retrieves a single instance of a repeating element in the flattened data

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 841
How can a Snowflake user validate data that is loaded using the COPY INTO [location] command?
- A. Load the data into a CSV file.
- B. Load the data into a relational table.
- C. Use the VALIDATION_MODE = RETURN_ERRORS SQL statement.
- D. Use the VALIDATION_MODE = RETURN_ROWS statement.

<details><summary>Show Answer</summary>
Correct Answer: C. (Note: VALIDATION_MODE also supports `RETURN_<n>_ROWS` for testing before a real load, but `RETURN_ERRORS` is the standard validation option among the answers given.)
</details>

---

### Question 842
What role in Snowflake separates the management of users and roles from the management of all grants?
- A. ACCOUNTADMIN
- B. SYSADMIN
- C. SECURITYADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: D. USERADMIN is dedicated to user/role management; SECURITYADMIN (which inherits USERADMIN) manages grants globally.
</details>

---

### Question 843
Which command will unload data from a table into an external stage?
- A. PUT
- B. INSERT
- C. COPY INTO [location]
- D. GET

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 844
Why is a federated environment used for user authentication in Snowflake?
- A. To enhance data security and privacy
- B. To provide real-time monitoring of user activities
- C. To separate authentication from access
- D. To enable direct integration with external databases

<details><summary>Show Answer</summary>
Correct Answer: C. Federated authentication (SSO/SAML) separates the act of verifying identity (handled by the IdP) from Snowflake's authorization/access model.
</details>

---

### Question 845
What will happen if a Snowflake user increases the size of a suspended virtual warehouse?
- A. The provisioning of compute for the warehouse will begin immediately.
- B. The warehouse will remain suspended but new resources will be added to the query acceleration service.
- C. The provisioning of additional compute resources will be in effect when the warehouse is next resumed.
- D. The warehouse will resume immediately and start to share the compute load with other running virtual warehouses.

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 846
The VALIDATE table function has which parameter as an input argument for a Snowflake user?
- A. LAST_QUERY_ID
- B. CURRENT_STATEMENT
- C. UUID_STRING
- D. JOB_ID

<details><summary>Show Answer</summary>
Correct Answer: D. VALIDATE takes a JOB_ID (the query ID of the COPY INTO statement, or `'_last'`).
</details>

---

### Question 847
Which Snowflake edition supports Protected Health Information (PHI) data (in accordance with HIPAA and HITRUST CSF regulations), and has a dedicated metadata store and pool of compute resources?
- A. Standard
- B. Enterprise
- C. Business Critical
- D. Virtual Private Snowflake (VPS)

<details><summary>Show Answer</summary>
Correct Answer: D. Business Critical also supports HIPAA/HITRUST, but VPS is the edition specifically defined by a completely dedicated (non-multi-tenant) metadata store and compute resource pool.
</details>

---

### Question 848
Which Snowflake table types are used to manage costs for short-lived tables? (Choose two.)
- A. External tables
- B. Permanent tables
- C. Directory tables
- D. Temporary tables
- E. Transient tables

<details><summary>Show Answer</summary>
Correct Answer: D, E.
</details>

---

### Question 849
What are key characteristics of virtual warehouses in Snowflake? (Choose two.)
- A. Warehouses that are multi-cluster can have nodes of different sizes.
- B. Warehouses can be started and stopped at any time.
- C. Warehouses can be resized at any time, even while running.
- D. Warehouses are billed on a per-minute usage basis.
- E. Warehouses can only be used for querying and cannot be used for data loading.

<details><summary>Show Answer</summary>
Correct Answer: B, C. All clusters in a multi-cluster warehouse are the same size (A is false), and billing is per-second with a 60-second minimum, not strictly "per-minute" (D is imprecise and not selected).
</details>

---

### Question 850
What strategies can be used to optimize the performance of a virtual warehouse? (Choose two.)
- A. Reduce queuing.
- B. Allow memory spillage.
- C. Increase the STATEMENT_TIMEOUT_IN_SECONDS parameter.
- D. Increase the warehouse size.
- E. Suspend the warehouse frequently.

<details><summary>Show Answer</summary>
Correct Answer: A, D.
</details>

---

### Question 851
How are privileges inherited in a role hierarchy in Snowflake?
- A. Privileges are inherited by any higher roles in the hierarchy.
- B. Privileges are inherited by any roles at the same level in the hierarchy.
- C. Privileges are only inherited by the direct parent role in the hierarchy.
- D. Privileges are only inherited by the direct child role in the hierarchy.

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 852
At what level can the STATEMENT_TIMEOUT_IN_SECONDS parameter be set?
- A. Account
- B. Role
- C. Session, Warehouse, and Account
- D. Virtual warehouse

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 853
What entity is responsible for hosting and sharing data in Snowflake?
- A. Data provider
- B. Data consumer
- C. Reader account
- D. Managed account

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 854
Which function will provide the proxy information needed to protect Snowsight?
- A. SYSTEM$GET_TAG
- B. SYSTEM$GET_PRIVATELINK
- C. SYSTEM$ALLOWLIST_PRIVATELINK
- D. SYSTEM$AUTHORIZE_PRIVATELINK

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 855
The DATA_RETENTION_TIME_IN_DAYS property is set at which level?
- A. User
- B. Role
- C. Account, Database, Schema, Table
- D. Organization

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 856
When unloading the data for file format type specified (TYPE = 'CSV'), SQL NULL can be converted to string 'null' using which file format option?
- A. EMPTY_FIELD_AS_NULL
- B. FIELD_OPTIONALLY_ENCLOSED_BY
- C. NULL_IF
- D. ESCAPE_UNENCLOSED_FIELD

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 857
Which table function should be used to view details on a Directed Acyclic Graph (DAG) run that is presently scheduled or is executing?
- A. TASK_HISTORY
- B. TASK_DEPENDENTS
- C. CURRENT_TASK_GRAPHS
- D. DAG_HISTORY

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 858
What Snowflake database object is derived from a query specification, stored for later use, and can speed up expensive aggregation on large data sets?
- A. Table
- B. External table
- C. Secure view
- D. Materialized view

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 859
User1, who has the SYSADMIN role, executed a query on Snowsight. User2, who is in the same Snowflake account, wants to view the result set of the query executed by User1 using the Snowsight history. What will happen if User2 tries to access the query history?
- A. If User2 has the SYSADMIN role they will be able to see the results.
- B. If User2 has the SECURITYADMIN role they will be able to see the results.
- C. If User2 has the ACCOUNTADMIN role they will be able to see the results.
- D. User2 will be unable to view the result set of the query executed by User1.

<details><summary>Show Answer</summary>
Correct Answer: D. Query results are only visible to the user who ran the query (within the result cache window), regardless of role.
</details>

---

### Question 860
A permanent table and temporary table have the same name, TBL1, in a schema. What will happen if a user executes `SELECT * FROM TBL1;`?
- A. The temporary table will take precedence over the permanent table.
- B. The permanent table will take precedence over the temporary table.
- C. An error will say there cannot be two tables with the same name in a schema.
- D. The table that was created most recently will take precedence over the older table.

<details><summary>Show Answer</summary>
Correct Answer: A. Temporary tables exist only for the session and shadow any permanent/transient table of the same name within that session.
</details>

---

### Question 861
The effects of query pruning can be observed by evaluating which statistics? (Choose two.)
- A. Partitions scanned
- B. Partitions total
- C. Bytes scanned
- D. Bytes read from result
- E. Bytes written

<details><summary>Show Answer</summary>
Correct Answer: A, B. Comparing "partitions scanned" against "partitions total" in the Query Profile shows how effective pruning was.
</details>

---

### Question 862
Which data types optimally store semi-structured data? (Choose two.)
- A. ARRAY
- B. CHARACTER
- C. STRING
- D. VARCHAR
- E. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: A, E.
</details>

---

### Question 863
What compute resource is used when loading data using Snowpipe?
- A. Snowpipe uses virtual warehouses provided by the user.
- B. Snowpipe uses an Apache Kafka server for its compute resources.
- C. Snowpipe uses compute resources provided by Snowflake.
- D. Snowpipe uses cloud platform compute resources provided by the user.

<details><summary>Show Answer</summary>
Correct Answer: C. Snowpipe is a serverless feature — Snowflake manages and provisions the compute for it.
</details>

---

### Question 864
Which file function gives a user or application access to download unstructured data from a Snowflake stage?
- A. GET_STAGE_URL
- B. BUILD_STAGE_FILE_URL
- C. GET_PRESIGNED_URL
- D. BUILD_SCOPED_FILE_URL

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 865
By default, which role can create resource monitors?
- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. SYSADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 866
Which DDL/DML operation is allowed on an inbound data share?
- A. ALTER TABLE
- B. INSERT INTO
- C. MERGE
- D. SELECT

<details><summary>Show Answer</summary>
Correct Answer: D. Data shares are read-only for the consumer.
</details>

---

### Question 867
Which types of charts does Snowsight support? (Choose two.)
- A. Area charts
- B. Bar charts
- C. Column charts
- D. Radar charts
- E. Scorecards

<details><summary>Show Answer</summary>
Correct Answer: B, E.
</details>

---

### Question 868
Which role in Snowflake allows users to enable replication for multiple accounts?
- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. SYSADMIN
- D. ORGADMIN

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 869
Which Snowflake tool is recommended for data batch processing?
- A. SnowCD
- B. SnowSQL
- C. Snowsight
- D. The Snowflake API

<details><summary>Show Answer</summary>
Correct Answer: B. SnowSQL is the command-line client best suited for scripted/batch workloads; Snowsight is the browser-based UI oriented toward interactive use.
</details>

---

### Question 870
Which Snowflake mechanism is used to limit the number of micro-partitions scanned by a query?
- A. Caching
- B. Cluster depth
- C. Query pruning
- D. Retrieval optimization

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 871
While clustering a table, columns with which data types can be used as clustering keys? (Choose two.)
- A. BINARY
- B. GEOGRAPHY
- C. GEOMETRY
- D. OBJECT
- E. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: A, C. Confirmed current: Snowflake documentation states a clustering key can be any data type except GEOGRAPHY, VARIANT, OBJECT, or ARRAY — so BINARY and GEOMETRY are both valid.
</details>

---

### Question 872
Which use case does the search optimization service support?
- A. Disjuncts (OR) in join predicates
- B. Inequality join predicates
- C. Join predicates on VARIANT columns
- D. Conjunctions (AND) of multiple equality predicates

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 873
What should be used when creating a CSV file format where the columns are wrapped by single quotes or double quotes?
- A. BINARY_FORMAT
- B. ESCAPE_UNENCLOSED_FIELD
- C. FIELD_OPTIONALLY_ENCLOSED_BY
- D. ENCLOSED_BY

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 874
If a multi-cluster warehouse is using an economy scaling policy, how long will queries wait in the queue before another cluster is started?
- A. 1 minute
- B. 2 minutes
- C. 6 minutes
- D. 8 minutes

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 875
What does the TableScan operator represent in the Query Profile?
- A. The scan of a single table
- B. The access to data stored in stage objects
- C. The list of values provided with the VALUES clause
- D. The records generated using the construct

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 876
What information is found within the Statistic output in the Query Profile Overview?
- A. Operator tree
- B. Table pruning
- C. Most expensive nodes
- D. Nodes by execution time

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 877
Which roles can make grant decisions to objects within a managed access schema? (Choose two.)
- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. SYSADMIN
- D. ORGADMIN
- E. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: A, B. In a managed access schema, only the schema owner or a role with the MANAGE GRANTS privilege (ACCOUNTADMIN/SECURITYADMIN by default) can grant privileges on objects in that schema — object owners lose that ability.
</details>

---

### Question 878
How can a Snowflake user post-process the result of SHOW FILE FORMATS?
- A. Use the RESULT_SCAN function.
- B. Create a CURSOR for the command.
- C. Put it in the FROM clause in brackets.
- D. Assign the command to RESULTSET.

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 879
A Snowflake account administrator has set the resource monitors with actions defined for each resource monitor as "Notify & Suspend Immediately". What is the MAXIMUM limit of credits that Warehouse 2 can consume?
- A. 1000
- B. 1500
- C. 3500
- D. 5000

<details><summary>Show Answer</summary>
Correct Answer: D (as given in the original source).

**Note:** This question references a specific resource-monitor credit-quota table/scenario that was not included in the supplied source text, so the numeric answer cannot be independently re-derived here — it is carried over from the original answer key as-is. If you have the original exhibit/table for this question, double-check the quota assigned to "Warehouse 2" against it.
</details>

---

### Question 880
When initially creating an account in Snowflake, which settings can be specified? (Choose two.)
- A. Account name
- B. Organization name
- C. Account locator
- D. Region
- E. Snowflake edition

<details><summary>Show Answer</summary>
Correct Answer: D, E. Region and edition are chosen at creation; the account locator is system-generated.
</details>

---

### Question 881
What activities can a user with the ORGADMIN role perform? (Choose two.)
- A. Create an account for an organization.
- B. Edit the data for an organization.
- C. Delete the account data for an organization.
- D. View usage information for all accounts in an organization.
- E. Select all the data in tables across all databases in an organization.

<details><summary>Show Answer</summary>
Correct Answer: A, D.
</details>

---

### Question 882
What is one of the benefits of using a multi-cluster virtual warehouse?
- A. It will speed up loading.
- B. It will reduce the cost of running the warehouse.
- C. It will automatically increase the warehouse size as needed.
- D. It will automatically start and stop additional clusters as needed.

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 883
When should a multi-cluster virtual warehouse be used in Snowflake?
- A. When queuing is delaying query execution on the warehouse.
- B. When there is significant disk spilling shown on the Query Profile.
- C. When dynamic vertical scaling is being used in the warehouse.
- D. When there are no concurrent queries running on the warehouse.

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 884
What is used to denote a pre-computed data set derived from a SELECT query specification and stored for later use?
- A. View
- B. Secure view
- C. Materialized view
- D. External table

<details><summary>Show Answer</summary>
Correct Answer: C.
</details>

---

### Question 885
A Snowflake user wants to temporarily bypass a network policy by configuring the user object property MINS_TO_BYPASS_MFA. What should they do?
- A. Use the SECURITYADMIN role.
- B. Use SYSADMIN role.
- C. Use the ACCOUNTADMIN role.
- D. Contact Snowflake Support.

<details><summary>Show Answer</summary>
Correct Answer: D (as originally given), but see the ⚠ **Updated** note below.

⚠ **Updated:** This question mixes up two distinct, similarly-named user properties. `MINS_TO_BYPASS_MFA` temporarily suspends the MFA requirement for a user (e.g., if they lose their authenticator device) and is set directly with `ALTER USER ... SET MINS_TO_BYPASS_MFA = <minutes>` — no Snowflake Support ticket is required. That command can be run by any role with sufficient privilege over the user object (typically SECURITYADMIN or a role that inherits USERADMIN privileges, or ACCOUNTADMIN). Separately, there is a `MINS_TO_BYPASS_NETWORK_POLICY` property, which is what actually controls a temporary bypass of a **network policy**; it is likewise set via ALTER USER by an appropriately privileged role. Contacting Snowflake Support is generally only necessary for account-level lockouts (e.g., an administrator is locked out entirely and no one internally has the privilege to fix it) — not for routine, self-service bypasses of either MFA or a network policy. If the question is strictly interpreted as written (bypassing MFA via an admin-settable property), the best answer among the given options is actually **A. Use the SECURITYADMIN role**, not D.
</details>

---

### Question 886
What is the default access of a securable object until other access is granted?
- A. No access
- B. Read access
- C. Write access
- D. Full access

<details><summary>Show Answer</summary>
Correct Answer: A.
</details>

---

### Question 887
From what stage can a Snowflake user omit the FROM clause while loading data into a table?
- A. The user stage
- B. The table stage
- C. The internal named stage
- D. The external named stage

<details><summary>Show Answer</summary>
Correct Answer: B. Because a table stage is implicitly tied to its table, `COPY INTO <table>` can omit the FROM clause.
</details>

---

### Question 888
What is used during the FIRST execution of `SELECT COUNT(*) FROM ORDER`?
- A. Remote disk cache
- B. Virtual warehouse cache
- C. Result cache
- D. Metadata-based result

<details><summary>Show Answer</summary>
Correct Answer: D. Snowflake can answer simple aggregate queries like COUNT(*) directly from table metadata without scanning data or even requiring a running warehouse.
</details>

---

### Question 889
What is the purpose of a resource monitor in Snowflake?
- A. To monitor the query performance of virtual warehouses.
- B. To create and suspend virtual warehouses automatically.
- C. To manage cloud services needed for virtual warehouses.
- D. To control costs and credit usage by virtual warehouses.

<details><summary>Show Answer</summary>
Correct Answer: D.
</details>

---

### Question 890
Which data formats are supported by Snowflake when unloading semi-structured data? (Choose two.)
- A. Binary file in Avro
- B. Binary file in Parquet
- C. Comma-separated JSON
- D. Newline Delimited JSON
- E. Plain text file containing XML elements

<details><summary>Show Answer</summary>
Correct Answer: B, D.
</details>

---

### Question 891
In Snowflake, the use of federated authentication enables which Single Sign-On (SSO) activities? (Choose two.)
- A. Authorizing users
- B. Initiating user sessions
- C. Logging into Snowflake
- D. Logging out of Snowflake
- E. Performing role authentication

<details><summary>Show Answer</summary>
Correct Answer: C, D.
</details>

---

### Question 892
What does the worksheet and database explorer feature in Snowsight allow users to do?
- A. Add users from a worksheet.
- B. Move a worksheet to a folder or a dashboard.
- C. Combine multiple worksheets into a single worksheet.
- D. Tag frequently accessed worksheets for ease of access.

<details><summary>Show Answer</summary>
Correct Answer: B.
</details>

---

### Question 893
When unloading data from Snowflake to AWS, what permissions are required? (Choose two.)
- A. s3:DeleteObject
- B. s3:CopyObject
- C. s3:GetObject
- D. s3:PutObject
- E. s3:GetBucketLocation

<details><summary>Show Answer</summary>
Correct Answer: A, D. Snowflake's sample IAM policy for an external stage used for unloading includes `s3:PutObject` (to write files) and `s3:DeleteObject` (so Snowflake can overwrite/replace files as needed).
</details>

---

### Question 894
What step can resolve data spilling in Snowflake?
- A. Using a larger virtual warehouse
- B. Increasing the virtual warehouse maximum timeout limit
- C. Increasing the amount of remote storage for the virtual warehouse
- D. Using a Common Table Expression (CTE) instead of a temporary table

<details><summary>Show Answer</summary>
Correct Answer: A. Spilling happens when a query's intermediate data exceeds available memory/local disk; a larger warehouse provides more memory and local SSD to avoid spilling to remote storage.
</details>

---

### Question 895
Which user preferences can be set for a user profile in Snowsight? (Choose two.)
- A. Multi-Factor Authentication (MFA)
- B. Default database
- C. Default schema
- D. Notifications
- E. Username

<details><summary>Show Answer</summary>
Correct Answer: A, D.
</details>

---

### Question 896
What privilege is needed for a Snowflake user to see the definition of a secure view?
- A. OWNERSHIP
- B. MODIFY
- C. CREATE
- D. USAGE

<details><summary>Show Answer</summary>
Correct Answer: A. Secure views intentionally hide their definition (via SHOW VIEWS / GET_DDL / Information Schema) from everyone except the object owner.
</details>

---

### Question 897
What guideline does Snowflake recommend when setting the auto-suspension time limit?
- A. Set tasks for immediate suspension.
- B. Set tasks for suspension after 5 minutes.
- C. Set query warehouses for suspension after 15 minutes.
- D. Set query warehouses for suspension after 30 minutes.

<details><summary>Show Answer</summary>
Correct Answer: A.

⚠ **Updated:** The original source listed B as the answer. Current Snowflake documentation ("Optimizing the warehouse cache") explicitly states general guidelines: **for task warehouses, Snowflake recommends immediate suspension**; for DevOps/DataOps/Data Science warehouses, roughly 5 minutes; and for BI/query warehouses, at least 10 minutes (to preserve the warm cache). None of the given options describes "at least 10 minutes" for query warehouses, so among the choices provided, **A** is the one that matches current official guidance — the "5 minutes" figure in the original answer (B) belongs to a different workload category (DevOps/DataOps), not tasks.
</details>

---

### Question 898
When does Snowflake automatically encrypt data that is loaded into Snowflake? (Choose two.)
- A. After the data is staged.
- B. After loading the data into a table.
- C. After loading the data into an internal stage.
- D. After loading data into an external stage.
- E. Only when using an encrypted stage.

<details><summary>Show Answer</summary>
Correct Answer: B, C. Snowflake automatically encrypts data at rest once it reaches an internal stage or a table; encryption of data in a customer-managed external stage is the customer's/cloud provider's responsibility, not automatic Snowflake encryption.
</details>

---

### Question 899
When data is loaded into Snowflake, what formats does Snowflake use internally to store the data in cloud? (Choose two.)
- A. Key-value
- B. Columnar
- C. Graph
- D. Document
- E. Compressed

<details><summary>Show Answer</summary>
Correct Answer: B, E. Snowflake stores data in compressed, columnar micro-partitions.
</details>

---

### Question 900
What do temporary and transient tables have in common in Snowflake?
- A. Both tables have no Fail-safe period.
- B. Both tables have data retention period maximums of one day.
- C. Both tables are visible only to a single user session.
- D. For both tables, the retention period ends when the tables are dropped.
- E. For both tables, the retention period does not end when the session ends.

<details><summary>Show Answer</summary>
Correct Answer: A, B. (Note: only temporary tables are session-scoped — C is false for transient tables, which persist across sessions like permanent tables but lack Fail-safe.)
</details>

---

## Summary of Corrections & Notes

| Question | Original Answer | Status | Note |
|---|---|---|---|
| 815 | C | ✅ Confirmed | Serverless task max = 2X-Large (XXLARGE), per current docs |
| 871 | AC | ✅ Confirmed | Clustering keys exclude GEOGRAPHY/VARIANT/OBJECT/ARRAY only; BINARY & GEOMETRY are valid |
| 885 | D | ⚠ Updated | Question conflates MINS_TO_BYPASS_MFA with network-policy bypass; a self-service ALTER USER by an admin role is the realistic mechanism, not a Support ticket |
| 897 | B | ⚠ Updated | Current Snowflake guidance recommends **immediate** suspension for task warehouses (Answer A), not "5 minutes" |
| 879 | D | ℹ️ Unverifiable | Original question references a resource-monitor exhibit/table not present in the supplied source text |

All other answers were reviewed against current Snowflake product behavior and match standard, stable Snowflake functionality as of July 2026.

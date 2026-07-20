# SnowPro Core Practice Questions (701–800)

*Cleaned, reformatted, and cross-checked against Snowflake documentation (as of July 2026). Answers are hidden in collapsible blocks — click "Show Answer" to reveal.*

---

### Question 701
Which URL provides access to files in Snowflake without authorization?

- A. File URL
- B. Scoped URL
- C. Pre-signed URL
- D. Scoped file URL

<details><summary>Show Answer</summary>
Correct Answer: C. A pre-signed URL embeds a temporary access token, so the holder can retrieve the file without separately authenticating to Snowflake.
</details>

---

### Question 702
What type of NULL values are supported in semi-structured data? (Choose two.)

- A. JSON NULL
- B. XML NULL
- C. ORC NULL
- D. Parquet NULL
- E. SQL NULL

<details><summary>Show Answer</summary>
Correct Answer: A, E. Snowflake distinguishes between a JSON NULL (a stored value meaning "no value") and a SQL NULL (the absence of a value).
</details>

---

### Question 703
What are characteristics of transient tables in Snowflake? (Choose two.)

- A. Transient tables have a Fail-safe period of 7 days.
- B. Transient tables can be cloned to permanent tables.
- C. Transient tables persist until they are explicitly dropped.
- D. Transient tables can be altered to make them permanent tables.
- E. Transient tables have Time Travel retention periods of 0 or 1 day.

<details><summary>Show Answer</summary>
Correct Answer: C, E. Transient tables have no Fail-safe period (ruling out A) and persist like permanent tables until dropped, with a Time Travel window of 0 or 1 day.
</details>

---

### Question 704
The INFORMATION_SCHEMA included in each database contains which objects? (Choose two.)

- A. Views for all the objects contained in the database
- B. Views for all the objects contained in the Snowflake account
- C. Views for historical and usage data across the Snowflake account
- D. Table functions for historical and usage data in the Snowflake account
- E. Table functions for account-level objects, such as roles, virtual warehouses, and databases

<details><summary>Show Answer</summary>
Correct Answer: A, D. INFORMATION_SCHEMA scopes views to the current database's objects, plus table functions that expose historical/usage data at the account level.
</details>

---

### Question 705
The use of which technique or tool will improve Snowflake query performance on very large tables?

- A. Clustering keys
- B. Multi-clustering
- C. Materialized views
- D. Search optimization service

<details><summary>Show Answer</summary>
Correct Answer: A. Defining a clustering key co-locates related rows in micro-partitions, improving pruning on very large tables.
</details>

---

### Question 706
Which Snowflake layer is associated with virtual warehouses?

- A. Cloud services
- B. Query processing
- C. Elastic memory
- D. Database storage

<details><summary>Show Answer</summary>
Correct Answer: B. Virtual warehouses make up the query processing (compute) layer of Snowflake's architecture.
</details>

---

### Question 707
Which MINIMUM set of privileges is required to temporarily bypass an active network policy by configuring the user object property?

- A. Only while in the ACCOUNTADMIN role
- B. Only while in the SECURITYADMIN role
- C. Only the role with the OWNERSHIP privilege on the network policy
- D. Only Snowflake Support can set the value of this property

<details><summary>Show Answer</summary>
Correct Answer: D. The `MINS_TO_BYPASS_NETWORK_POLICY` user property can only be set by contacting Snowflake Support — no account-side role, including ACCOUNTADMIN, can set it directly. This remains accurate per current documentation.
</details>

---

### Question 708
What authentication method does the Kafka connector use within Snowflake?

- A. Key pair authentication
- B. Multi-Factor Authentication (MFA)
- C. OAuth
- D. Username and password

<details><summary>Show Answer</summary>
Correct Answer: A. The Kafka connector authenticates using key pair authentication.
</details>

---

### Question 709
What is the purpose of the Snowflake SPLIT_TO_TABLE function?

- A. To count the number of characters in a string
- B. To split a string into an array of sub-strings
- C. To split a string and flatten the results into rows
- D. To split a string and flatten the results into columns

<details><summary>Show Answer</summary>
Correct Answer: C. SPLIT_TO_TABLE splits an input string on a delimiter and returns each piece as a separate output row.
</details>

---

### Question 710
What feature of Snowflake Continuous Data Protection can be used for maintenance of historical data?

- A. Access control
- B. Fail-safe
- C. Network policies
- D. Time Travel

<details><summary>Show Answer</summary>
Correct Answer: D. Time Travel lets users query, clone, or restore historical versions of data within a configurable retention window.
</details>

---

### Question 711
What aspect of an executed query is represented by the remote disk I/O statistic of the Query Profile in Snowflake?

- A. Time spent scanning the table partitions for data based on the predicate
- B. Time spent caching the data to remote storage in order to buffer the data being extracted and exported
- C. Time spent reading and writing data from and to remote storage when the data being accessed does not fit into the executing virtual warehouse node memory
- D. Time spent reading and writing data from and to remote storage when the data being accessed does not fit into either the virtual memory or the local disk

<details><summary>Show Answer</summary>
Correct Answer: D.

⚠ **Updated:** The original source listed C as correct, but current Snowflake documentation on Query Profile statistics defines "Local Disk IO" as time blocked by local disk access, and "Remote Disk IO" as time blocked by remote disk access — which only happens once data has already spilled past both memory *and* local disk. Option C stops at "memory," which actually describes local disk spilling, not remote. Option D correctly captures the two-stage spill (memory → local disk → remote disk).
</details>

---

### Question 712
What action can a user take to address query concurrency issues?

- A. Enable the search optimization service.
- B. Enable the query acceleration service.
- C. Add additional clusters to the virtual warehouse.
- D. Resize the virtual warehouse to a larger instance size.

<details><summary>Show Answer</summary>
Correct Answer: C. Adding clusters to a multi-cluster warehouse increases the number of queries that can run concurrently without queuing.
</details>

---

### Question 713
What does the Client redirect feature in Snowflake enable?

- A. A redirect of client connections to Snowflake accounts in the same regions for business continuity.
- B. A redirect of client connections to Snowflake accounts in different regions for business continuity.
- C. A redirect of client connections to Snowflake accounts in different regions for data replication.
- D. A redirect of client connections to Snowflake accounts in the same regions for data replication.

<details><summary>Show Answer</summary>
Correct Answer: B. Client redirect allows connections to fail over to a secondary account in a different region for business continuity/disaster recovery.
</details>

---

### Question 714
Which Snowflake feature can be used to find sensitive data in a table or column?

- A. Masking
- B. Data classification
- C. Row policies
- D. External functions

<details><summary>Show Answer</summary>
Correct Answer: B. Data classification scans and tags columns to identify sensitive data categories.
</details>

---

### Question 715
Which Snowflake feature allows a user to track sensitive data for compliance, discovery, protection, and resource usage?

- A. Object tagging
- B. Comments
- C. Internal tokenization
- D. Row access policies

<details><summary>Show Answer</summary>
Correct Answer: A. Object tagging lets users attach metadata to objects to support governance, compliance, and cost-tracking use cases.
</details>

---

### Question 716
Snowflake's hierarchical key model includes which keys? (Choose two.)

- A. Account master keys
- B. Database master keys
- C. File keys
- D. Secure view keys
- E. Schema master keys

<details><summary>Show Answer</summary>
Correct Answer: A, C. Snowflake's key hierarchy runs from a root key down through account master keys, table master keys, to individual file keys. Of the listed options, account master keys and file keys are genuine tiers.
</details>

---

### Question 717
What can the Snowflake SCIM API be used to manage? (Choose two.)

- A. Integrations
- B. Network policies
- C. Session policies
- D. Roles
- E. Users

<details><summary>Show Answer</summary>
Correct Answer: D, E. The SCIM API synchronizes users and roles from an external identity provider into Snowflake.
</details>

---

### Question 718
Which privilege is required to use the search optimization service in Snowflake?

- A. GRANT SEARCH OPTIMIZATION ON SCHEMA [schema_name] TO ROLE [role]
- B. GRANT SEARCH OPTIMIZATION ON DATABASE TO ROLE
- C. GRANT ADD SEARCH OPTIMIZATION ON SCHEMA [schema_name] TO ROLE [role]
- D. GRANT ADD SEARCH OPTIMIZATION ON DATABASE [database_name] TO ROLE [role]

<details><summary>Show Answer</summary>
Correct Answer: C. The ADD SEARCH OPTIMIZATION privilege must be granted at the schema (or database) level using this exact syntax.
</details>

---

### Question 719
What is the FASTEST way to bulk load data files from a Stage?

- A. Specifying a list of specific files to load
- B. Loading by path (internal stages)
- C. Using the Snowpipe REST API
- D. Using pattern matching to identify specific files by pattern

<details><summary>Show Answer</summary>
Correct Answer: B. Loading by referencing a stage path avoids the extra processing overhead of explicit file lists or pattern matching.
</details>

---

### Question 720
How does a Snowflake user extract the URL of a directory table on an external stage for further transformation?

- A. Use the SHOW STAGES command.
- B. Use the DESCRIBE STAGE command.
- C. Use the GET_PRESIGNED_URL function.
- D. Use the BUILD_STAGE_FILE_URL function.

<details><summary>Show Answer</summary>
Correct Answer: D. BUILD_STAGE_FILE_URL generates a permanent, Snowflake-authenticated URL for a staged file that can be used in further transformation logic.
</details>

---

### Question 721
A Snowflake user needs to share unstructured data from an internal stage to a reporting tool that does not have Snowflake access. Which file function should be used?

- A. BUILD_SCOPED_FILE_URL
- B. BUILD_STAGE_FILE_URL
- C. GET_PRESIGNED_URL
- D. GET_STAGE_LOCATION

<details><summary>Show Answer</summary>
Correct Answer: C. GET_PRESIGNED_URL creates a temporary URL that grants access without requiring the recipient to authenticate to Snowflake — ideal for external tools.
</details>

---

### Question 722
The use of which Snowflake table type will reduce costs when working with ETL workflows?

- A. Permanent
- B. Temporary
- C. Transient
- D. External

<details><summary>Show Answer</summary>
Correct Answer: C. Transient tables skip Fail-safe storage costs, making them cost-effective for intermediate ETL staging tables.
</details>

---

### Question 723
What is one of the characteristics of data shares?

- A. Data shares support full DML operations.
- B. Data shares work by copying data to consumer accounts.
- C. Data shares utilize secure views for sharing view objects.
- D. Data shares are cloud agnostic and can cross regions by default.

<details><summary>Show Answer</summary>
Correct Answer: C. Only secure views (not standard views) can be added to a share.
</details>

---

### Question 724
What is the MINIMUM configurable idle timeout value for a session policy in Snowflake?

- A. 2 minutes
- B. 5 minutes
- C. 10 minutes
- D. 15 minutes

<details><summary>Show Answer</summary>
Correct Answer: B. The minimum value for `SESSION_IDLE_TIMEOUT_MINS` in a session policy is 5 minutes.
</details>

---

### Question 725
Which command is used to unload data from a Snowflake table to an external stage?

- A. COPY INTO
- B. COPY INTO followed by GET
- C. GET
- D. COPY INTO followed by PUT

<details><summary>Show Answer</summary>
Correct Answer: A. `COPY INTO <location>` unloads table data directly to an external (or internal) stage.
</details>

---

### Question 726
What is a characteristic of materialized views in Snowflake?

- A. Materialized views do not allow joins.
- B. Clones of materialized views can be created directly by the user.
- C. Multiple tables can be joined in the underlying query of a materialized view.
- D. Aggregate functions can be used as window functions in materialized views.

<details><summary>Show Answer</summary>
Correct Answer: A. A materialized view can reference only a single base table, so joins are not supported.
</details>

---

### Question 727
Which Snowflake URL type allows users or applications to download or access files directly from a Snowflake stage without authentication?

- A. Directory
- B. File
- C. Pre-signed
- D. Scoped

<details><summary>Show Answer</summary>
Correct Answer: C. A pre-signed URL includes a temporary access token, letting a holder download the file without authenticating to Snowflake.
</details>

---

### Question 728
Which SQL command will download all the data files from an internal table stage named TBL_EMPLOYEE to a local windows directory on a client machine, in a folder named "folder with space" within the C drive?

- A. `GET @%TBL_EMPLOYEE file://C:\folder with space;`
- B. `GET @%TBL_EMPLOYEE 'file://C:\folder with space';`
- C. `PUT @%TBL_EMPLOYEE 'file://C:\folder with space';`
- D. `COPY INTO 'file://C:\folder with space' FROM @%TBL_EMPLOYEE;`

<details><summary>Show Answer</summary>
Correct Answer: B. GET downloads files from a stage to a local machine, and a path containing spaces must be quoted.
</details>

---

### Question 729
How can the COPY command be used to unload data from a table to an internal stage?

- A. COPY INTO [location]
- B. COPY INTO @stage
- C. COPY INTO [location] with single=true
- D. COPY INTO s3://[bucket]

<details><summary>Show Answer</summary>
Correct Answer: A. `COPY INTO <location>` is the general unload syntax, where the location can be an internal stage.
</details>

---

### Question 730
How does a Snowflake stored procedure compare to a User-Defined Function (UDF)?

- A. A single executable statement can call only two stored procedures. In contrast, a single SQL statement can call multiple UDFs.
- B. A single executable statement can call only one stored procedure. In contrast, a single SQL statement can call multiple UDFs.
- C. A single executable statement can call multiple stored procedures. In contrast, multiple SQL statements can call the same UDFs.
- D. Multiple executable statements can call more than one stored procedure. In contrast, a single SQL statement can call multiple UDFs.

<details><summary>Show Answer</summary>
Correct Answer: B. A statement can invoke only one stored procedure, whereas a single SQL statement can reference multiple UDFs.
</details>

---

### Question 731
Which command should be used to unload all the rows from a table into one or more files in a named stage?

- A. COPY INTO
- B. GET
- C. INSERT INTO
- D. EXPORT

<details><summary>Show Answer</summary>
Correct Answer: A. COPY INTO unloads table rows into files in a stage.
</details>

---

### Question 732
Which command is used to unload data from a table or move a query result to a stage?

- A. COPY INTO
- B. GET
- C. MERGE
- D. PUT

<details><summary>Show Answer</summary>
Correct Answer: A. COPY INTO handles both table unloads and unloading query results to a stage.
</details>

---

### Question 733
What privileges are necessary for a consumer in the Data Exchange to make a request and receive data? (Choose two.)

- A. CREATE DATABASE
- B. IMPORT SHARE
- C. OWNERSHIP
- D. REFERENCE_USAGE
- E. USAGE

<details><summary>Show Answer</summary>
Correct Answer: A, B. The consumer role needs CREATE DATABASE (to materialize the shared data) and IMPORT SHARE (to accept the share).
</details>

---

### Question 734
What are benefits of using Snowpark? (Choose two.)

- A. Snowpark uses a Spark engine to generate optimized SQL query plans.
- B. Snowpark automatically sets up Spark within Snowflake virtual warehouses.
- C. Snowpark does not require that a separate cluster be running outside of Snowflake.
- D. Snowpark allows users to run existing Spark code on virtual warehouses without the need to reconfigure the code.
- E. Snowpark pushes as much work as possible to the Snowflake database for all operations including User-Defined Functions (UDFs).

<details><summary>Show Answer</summary>
Correct Answer: C, E. Snowpark executes entirely within Snowflake's own compute (no external Spark cluster needed) and pushes computation, including UDFs, down into Snowflake.
</details>

---

### Question 735
What are Snowflake best practices when assigning the ACCOUNTADMIN role to users? (Choose two.)

- A. The ACCOUNTADMIN role should be assigned to at least two users.
- B. The ACCOUNTADMIN role should be used to create Snowflake objects.
- C. The ACCOUNTADMIN role should be used for running automated scripts.
- D. The ACCOUNTADMIN role should be given to any user who needs a high level of authority.
- E. All users assigned the ACCOUNTADMIN role should use Multi-Factor Authentication (MFA).

<details><summary>Show Answer</summary>
Correct Answer: A, E. Best practice is to assign ACCOUNTADMIN to at least two users (for continuity) and to require MFA for all of them.
</details>

---

### Question 736
What is a recommended approach for optimizing query performance in Snowflake?

- A. Use subqueries whenever possible.
- B. Use a number of joins to combine data from multiple tables.
- C. Select all columns from tables, even if they are not needed in the query.
- D. Use a smaller number of larger tables rather than a larger number of smaller tables.

<details><summary>Show Answer</summary>
Correct Answer: D. Fewer, larger tables reduce join overhead and simplify query planning compared to many small tables.
</details>

---

### Question 737
When using SnowSQL, which configuration options are required when unloading data via a SQL query run on a local machine? (Choose two.)

- A. connection
- B. quiet
- C. output_file
- D. output_format
- E. header

<details><summary>Show Answer</summary>
Correct Answer: C, D. `output_file` and `output_format` are required to control where and how SnowSQL writes the unloaded data locally.
</details>

---

### Question 738
Which Snowflake view is used to support compliance auditing?

- A. ACCESS_HISTORY
- B. COPY_HISTORY
- C. QUERY_HISTORY
- D. LOGIN_HISTORY

<details><summary>Show Answer</summary>
Correct Answer: A. ACCESS_HISTORY records read/write activity against data, supporting compliance auditing.
</details>

---

### Question 739
How can a Snowflake user load duplicate files with a COPY INTO command?

- A. The COPY INTO options should be set to PURGE = FALSE
- B. The COPY INTO options should be set to FORCE = TRUE
- C. The COPY INTO options should be set to REPLACE = FALSE
- D. The COPY INTO options should be ON_ERROR = CONTINUE

<details><summary>Show Answer</summary>
Correct Answer: B. FORCE = TRUE tells COPY INTO to reload files even if they were already loaded and recorded in load metadata.
</details>

---

### Question 740
What is an advantage of using a multi-cluster virtual warehouse as compared to a single-cluster virtual warehouse in Snowflake?

- A. A user can auto-suspend a running warehouse due to inactivity.
- B. A user can specify a warehouse size while configuring it for use.
- C. A user can resize a warehouse at any time whether running or not.
- D. A user can specify the maximum and minimum number of clusters.

<details><summary>Show Answer</summary>
Correct Answer: D. Multi-cluster warehouses uniquely allow configuring a minimum and maximum cluster count for automatic scaling.
</details>

---

### Question 741
Which transformation techniques are supported for bulk loading data into Snowflake using the COPY INTO [table] command? (Choose two.)

- A. Column grouping
- B. Column omission
- C. Column reordering
- D. Column aggregation
- E. Selection of a limited number of rows

<details><summary>Show Answer</summary>
Correct Answer: B, C. COPY INTO [table] supports column reordering and column omission during a load; it does not support aggregation, grouping, or row limiting.
</details>

---

### Question 742
Which type of charts are supported by Snowsight? (Choose two.)

- A. Flowcharts
- B. Gantt charts
- C. Line charts
- D. Pie charts
- E. Scorecards

<details><summary>Show Answer</summary>
Correct Answer: C, E. Snowsight dashboards support chart types such as line charts and scorecards, among others.
</details>

---

### Question 743
A user wants to upload a file to an internal Snowflake stage using a PUT command. Which tools and/or connectors could be used to execute this command? (Choose two.)

- A. SnowCD
- B. SnowSQL
- C. SQL API
- D. Python connector
- E. Worksheets

<details><summary>Show Answer</summary>
Correct Answer: B, D. PUT is a client-side command supported by SnowSQL and the Snowflake drivers/connectors (e.g., Python), not by the SQL API or Snowsight worksheets.
</details>

---

### Question 744
Which Snowflake table is an implicit object layered on a stage, where the stage can be either internal or external?

- A. Directory table
- B. Temporary table
- C. Transient table
- D. A table with a materialized view

<details><summary>Show Answer</summary>
Correct Answer: A. A directory table is not a standalone object — it is implicitly layered on top of a stage to expose file-level metadata.
</details>

---

### Question 745
The Query Profile in the image is for a query executed in Snowsight. Four of the key nodes are highlighted in yellow. Which highlighted node will be the MOST expensive?

- A. Aggregate[1]
- B. Join[5]
- C. TableScan[2]
- D. TableScan[3]

<details><summary>Show Answer</summary>
Correct Answer: D. *(This question depends on a Query Profile screenshot that was not included in the source material, so the cost comparison cannot be independently re-verified here — the original answer is retained as-is.)*
</details>

---

### Question 746
What is a characteristic of the maintenance of a materialized view?

- A. Materialized views cannot be refreshed automatically.
- B. An additional set of scripts is needed to refresh data in materialized views.
- C. A materialized view is automatically refreshed by a Snowflake managed warehouse.
- D. A materialized view can be set up with the auto-refresh feature using the SQL SET command.

<details><summary>Show Answer</summary>
Correct Answer: C. Snowflake automatically maintains materialized views using background, Snowflake-managed compute, with no user scripts required.
</details>

---

### Question 747
Which command should be used to implement a masking policy that was already created in Snowflake?

- A. ALTER MASKING POLICY
- B. APPLY MASKING POLICY
- C. CREATE MASKING POLICY
- D. ALTER TABLE [table_name] MODIFY COLUMN [column_name] SET MASKING POLICY [policy_name]

<details><summary>Show Answer</summary>
Correct Answer: D. Attaching an already-created masking policy to a column requires an ALTER TABLE ... MODIFY COLUMN ... SET MASKING POLICY statement.
</details>

---

### Question 748
A Snowflake user runs a query for 36 seconds on a size 2XL virtual warehouse. What would be the credit consumption?

- A. Snowflake will charge for 36 seconds at the rate of 32 credits per hour.
- B. Snowflake will charge for 36 seconds at the rate of 64 credits per hour.
- C. Snowflake will charge for 60 seconds at the rate of 32 credits per hour.
- D. Snowflake will charge for 60 seconds at the rate of 64 credits per hour.

<details><summary>Show Answer</summary>
Correct Answer: C. A 2X-Large warehouse consumes 32 credits/hour, and Snowflake bills a 60-second minimum for each time the warehouse starts running.
</details>

---

### Question 749
Which statement accurately describes a characteristic of a materialized view?

- A. A materialized view can query only a single table.
- B. Data accessed through materialized views can be stale.
- C. Materialized view refreshes need to be maintained by the user.
- D. Querying a materialized view is slower than executing a query against the base table of the view.

<details><summary>Show Answer</summary>
Correct Answer: A. Materialized views are restricted to querying a single base table.
</details>

---

### Question 750
The use of which Snowflake table type will reduce costs when working with ETL workflows?

- A. Permanent
- B. Temporary
- C. Transient
- D. External

<details><summary>Show Answer</summary>
Correct Answer: C. Transient tables avoid Fail-safe storage costs, making them well-suited to transient ETL staging data.
</details>

---

### Question 751
A user wants to unload data from a relational table into a CSV file in an external stage. The table must be named exactly as specified by the user. Which file format option MUST be used to do this?

- A. encoding
- B. escape
- C. single = true
- D. file_extension

<details><summary>Show Answer</summary>
Correct Answer: C. Setting `single = true` produces one output file with a user-specified name, instead of Snowflake's default auto-generated, multi-part file names.
</details>

---

### Question 752
Which account usage view in Snowflake can be used to identify the most-frequently accessed tables?

- A. Access_History
- B. Object_Dependencies
- C. Tables
- D. Query_History

<details><summary>Show Answer</summary>
Correct Answer: A. ACCESS_HISTORY records which objects were read or written by each query, making it possible to identify frequently accessed tables.
</details>

---

### Question 753
What metadata does Snowflake store concerning all rows stored in a micro-partition? (Choose two.)

- A. A count of the number of total values in the micro-partition
- B. The range of values for each partition in the micro-partition
- C. The range of values for each of the rows in the micro-partition
- D. The range of values for each of the columns in the micro-partition
- E. The number of distinct values for each column in the micro-partition

<details><summary>Show Answer</summary>
Correct Answer: D, E. For each micro-partition, Snowflake stores the range of values and the count of distinct values, per column.
</details>

---

### Question 754
What role has the privileges to create and manage data shares by default?

- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. SYSADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: A. By default, only ACCOUNTADMIN (or a role explicitly granted the CREATE SHARE global privilege) can create and manage shares.
</details>

---

### Question 755
Which function determines the kind of value stored in a VARIANT column?

- A. CHECK_JSON
- B. IS_ARRAY
- C. IS_JSON
- D. TYPEOF

<details><summary>Show Answer</summary>
Correct Answer: D. TYPEOF returns the underlying data type of a value stored in a VARIANT column.
</details>

---

### Question 756
What operation can be performed using Time Travel?

- A. Restoring tables that have been dropped from a data share
- B. Extending a permanent table's retention duration from 90 to 100 days
- C. Creating a clone of an entire table at a point in the past from a permanent table
- D. Disabling Time Travel for a specific object by setting to NULL

<details><summary>Show Answer</summary>
Correct Answer: C. Time Travel supports cloning a table as it existed at a specific historical point in time.
</details>

---

### Question 757
What are characteristics of Snowflake directory tables? (Choose two.)

- A. Directory tables are separate database objects.
- B. Directory tables can only be used with an external stage.
- C. Directory tables contain data stored in binary format.
- D. Directory tables store file-level metadata about the data files in a stage.
- E. A directory table can be added explicitly to a stage when the stage is created, or later.

<details><summary>Show Answer</summary>
Correct Answer: D, E. Directory tables store file-level metadata for a stage and can be enabled either at stage creation or afterward via ALTER STAGE.
</details>

---

### Question 758
What does the VARIANT data type impose a 16 MB size limit on?

- A. All rows
- B. All columns
- C. Individual rows
- D. Individual columns

<details><summary>Show Answer</summary>
Correct Answer: C. The 16 MB limit applies to each individual VARIANT value (i.e., per row).
</details>

---

### Question 759
Which activities are included in the Cloud Services layer? (Choose two.)

- A. Data storage
- B. Dynamic data masking
- C. Partition scanning
- D. User authentication
- E. Infrastructure management

<details><summary>Show Answer</summary>
Correct Answer: D, E. Authentication and infrastructure management are handled by the Cloud Services layer, distinct from storage or compute.
</details>

---

### Question 760
What does the "scanned from cache" represent in the Query profile?

- A. The percentage of data scanned from the query cache
- B. The percentage of data scanned from the result cache
- C. The percentage of data scanned from the remote disk cache
- D. The percentage of data scanned from the local disk cache

<details><summary>Show Answer</summary>
Correct Answer: D. "Percentage scanned from cache" reflects the share of data read from the warehouse's local disk (SSD) cache instead of remote storage.
</details>

---

### Question 761
Which role has the ability to create a share from a shared database by default?

- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. SYSADMIN
- D. ORGADMIN

<details><summary>Show Answer</summary>
Correct Answer: A. By default, ACCOUNTADMIN holds the global CREATE SHARE privilege needed to create and manage shares. (Note: Snowflake does not allow directly re-sharing a database or objects that were themselves received via a share — this question is best read as asking which role manages sharing by default.)
</details>

---

### Question 762
Which object-level parameters can be set to help control query processing and concurrency? (Choose two.)

- A. MAX_CONCURRENCY_LEVEL
- B. STATEMENT_QUEUED_TIMEOUT_IN_SECONDS
- C. STATEMENT_TIMEOUT_IN_SECONDS
- D. MAX_STATEMENT_TIME
- E. WAREHOUSE_SIZE

<details><summary>Show Answer</summary>
Correct Answer: B, C. These two parameters control how long a statement can queue and how long it can run before timing out.
</details>

---

### Question 763
What metadata does Snowflake store for micro-partitions? (Choose two.)

- A. Range of values
- B. Distinct values
- C. Index values
- D. Sorted values
- E. Null values

<details><summary>Show Answer</summary>
Correct Answer: A, B. Snowflake tracks the range of values and count of distinct values per column for each micro-partition.
</details>

---

### Question 764
What are valid sub-clauses to the OVER clause for a window function? (Choose two.)

- A. GROUP BY
- B. LIMIT
- C. ORDER BY
- D. PARTITION BY
- E. UNION ALL

<details><summary>Show Answer</summary>
Correct Answer: C, D. Window functions use PARTITION BY and ORDER BY within the OVER clause.
</details>

---

### Question 765
Which kind of Snowflake table stores file-level metadata for each file in a stage?

- A. Directory
- B. External
- C. Temporary
- D. Transient

<details><summary>Show Answer</summary>
Correct Answer: A. A directory table stores file-level metadata for the files in its associated stage.
</details>

---

### Question 766
Which privileges apply to stored procedures? (Choose two.)

- A. MODIFY
- B. MONITOR
- C. OPERATE
- D. OWNERSHIP
- E. USAGE

<details><summary>Show Answer</summary>
Correct Answer: D, E. Stored procedures support OWNERSHIP and USAGE privileges.
</details>

---

### Question 767
What column type does a Kafka connector store formatted information in a single column?

- A. ARRAY
- B. OBJECT
- C. VARCHAR
- D. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: D. The Kafka connector loads each message's contents into a single VARIANT column.
</details>

---

### Question 768
If a size Small virtual warehouse costs two credits per hour, what is the credit cost per hour of a size Large virtual warehouse?

- A. 4
- B. 8
- C. 16
- D. 32

<details><summary>Show Answer</summary>
Correct Answer: B. Warehouse credit cost doubles with each size increase: XS=1, S=2, M=4, L=8 credits/hour.
</details>

---

### Question 769
Which SQL command will list the files in a named stage?

- A. `list @my_stage;`
- B. `get @my_stage;`
- C. `list my_stage;`
- D. `get my_stage;`

<details><summary>Show Answer</summary>
Correct Answer: A. `LIST @my_stage;` lists the files present in the named stage (the `@` prefix is required).
</details>

---

### Question 770
What is the effect of configuring a virtual warehouse auto-suspend value to 0 or NULL?

- A. The warehouse will never suspend automatically.
- B. The warehouse will suspend immediately upon work completion.
- C. The warehouse will not resume automatically.
- D. All clusters in the multi-cluster warehouse will resume immediately.

<details><summary>Show Answer</summary>
Correct Answer: A. Setting auto-suspend to 0 or NULL disables automatic suspension, so the warehouse keeps running (and billing) until manually suspended.
</details>

---

### Question 771
Which data types can be used in Snowflake to store semi-structured data? (Choose two.)

- A. ARRAY
- B. BLOB
- C. CLOB
- D. JSON
- E. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: A, E. ARRAY and VARIANT are Snowflake's semi-structured data types (alongside OBJECT).
</details>

---

### Question 772
While attempting to prevent data duplication, which COPY INTO [location] option should be used to load files with expired load metadata?

- A. PURGE
- B. FORCE
- C. VALIDATION_MODE
- D. LOAD_UNCERTAIN_FILES

<details><summary>Show Answer</summary>
Correct Answer: B. FORCE reloads files even when their load history/metadata has expired or already shows them as loaded.
</details>

---

### Question 773
What service is provided as an integrated Snowflake feature to enhance Multi-Factor Authentication (MFA) support?

- A. Duo Security
- B. OAuth
- C. Okta
- D. Single Sign-on (SSO)

<details><summary>Show Answer</summary>
Correct Answer: A. Snowflake's built-in MFA is powered by Duo Security.
</details>

---

### Question 774
What is the impact on queries that are being executed when a resource monitor set to the "Notify & Suspend" threshold level is exceeded?

- A. All statements being executed are queued.
- B. All statements being executed are restarted.
- C. All statements being executed are cancelled.
- D. All statements being executed are allowed to complete.

<details><summary>Show Answer</summary>
Correct Answer: D. "Notify & Suspend" lets in-flight queries finish, while blocking any new queries from starting.
</details>

---

### Question 775
What tasks can an account administrator perform in the Data Exchange? (Choose two.)

- A. Add and remove members.
- B. Delete data categories.
- C. Approve and deny listing approval requests.
- D. Transfer listing ownership.
- E. Transfer ownership of a provider profile.

<details><summary>Show Answer</summary>
Correct Answer: A, C. Data Exchange administrators manage membership and approve/deny listing requests.
</details>

---

### Question 776
Which types of subqueries does Snowflake support? (Choose two.)

- A. Scalar subqueries in WHERE clauses
- B. Uncorrelated scalar subqueries in any place that a value expression can be used
- C. EXISTS, ANY/ALL, and IN subqueries in WHERE clauses; these subqueries can be uncorrelated only
- D. EXISTS, ANY/ALL, and IN subqueries in WHERE clauses; these can be correlated only
- E. EXISTS, ANY/ALL, and IN subqueries in WHERE clauses; these subqueries can be correlated or uncorrelated

<details><summary>Show Answer</summary>
Correct Answer: B, E. Snowflake supports uncorrelated scalar subqueries anywhere a value expression is valid, and EXISTS/ANY/ALL/IN subqueries that may be correlated or uncorrelated.
</details>

---

### Question 777
How can network and private connectivity security be managed in Snowflake?

- A. By setting up network policies with IPv4 IP addresses
- B. By putting the URL on the allowed list for get method responses
- C. By manually setting up vulnerability patch management policies
- D. By manually setting up an Intrusion Prevention System (IPS) on each account

<details><summary>Show Answer</summary>
Correct Answer: A. Network policies built from allowed/blocked IPv4 addresses (and network rules) are Snowflake's mechanism for controlling network access.
</details>

---

### Question 778
What consideration should be made when loading data into Snowflake?

- A. Create small data files and stage them in cloud storage frequently.
- B. Create large data files to maximize the processing overhead for each file.
- C. The number of load operations that run in parallel can exceed the number of data files to be loaded.
- D. The number of data files that are processed in parallel is determined by the virtual warehouse.

<details><summary>Show Answer</summary>
Correct Answer: D. The virtual warehouse's size determines how many data files can be processed in parallel during a load.
</details>

---

### Question 779
How can a user improve the performance of a single large complex query in Snowflake?

- A. Scale up the virtual warehouse.
- B. Scale out the virtual warehouse.
- C. Enable standard warehouse scaling.
- D. Enable economy scaling.

<details><summary>Show Answer</summary>
Correct Answer: A. Scaling up (increasing warehouse size) adds compute/memory to a single query, whereas scaling out (multi-cluster) helps with concurrency, not single-query speed.
</details>

---

### Question 780
Who can access a referenced file through a scoped URL?

- A. Only the ACCOUNTADMIN
- B. Only the user who generates the URL
- C. Any role specified in GET REST API call with sufficient privileges
- D. Any user specified in the GET REST API call with sufficient privileges

<details><summary>Show Answer</summary>
Correct Answer: B. A scoped URL is tied to the session/user that generated it and cannot be used by anyone else.
</details>

---

### Question 781
Snowflake will return an error when a user attempts to share which object?

- A. Tables
- B. Secure views
- C. Standard views
- D. Secure materialized views

<details><summary>Show Answer</summary>
Correct Answer: C. Only secure views (and secure materialized views/UDFs) can be added to a share; attempting to share a standard view returns an error. This matches current Snowflake documentation.
</details>

---

### Question 782
What setting in Snowsight determines the databases, tables, and other objects that can be seen and the actions that can be performed on them?

- A. Active role
- B. Masking policy
- C. Column-level security
- D. Multi-Factor Authentication (MFA)

<details><summary>Show Answer</summary>
Correct Answer: A. The user's currently active role governs which objects are visible and which actions are permitted in Snowsight.
</details>

---

### Question 783
Why would a Snowflake user decide to use a materialized view instead of a regular view?

- A. The base tables do not change frequently.
- B. The results of the view change often.
- C. The query is not resource intensive.
- D. The query results are not used frequently.

<details><summary>Show Answer</summary>
Correct Answer: A. Materialized views are most beneficial when the underlying data changes infrequently but the (expensive) query is run often.
</details>

---

### Question 784
When a database is cloned, which objects in the clone inherit all granted privileges from the source object? (Choose two.)

- A. Account
- B. Database
- C. Schemas
- D. Tables
- E. Internal named stages

<details><summary>Show Answer</summary>
Correct Answer: C, D. Objects contained within a cloned database — such as schemas and tables — retain the privileges granted on their source objects; the cloned database itself does not inherit privileges granted on the source database.
</details>

---

### Question 785
How does the Access_History view enhance overall data governance pertaining to read and write operations? (Choose two.)

- A. Shows how accessed data was moved from the source to the target objects.
- B. Provides a unified picture of what data was accessed and when it was accessed.
- C. Protects sensitive data from unauthorized access while allowing authorized users to access it at query runtime.
- D. Identifies columns with personal information and tags them so masking policies can be applied to protect sensitive data.
- E. Determines whether a given row in a table can be accessed by the user by filtering the data based on a given policy.

<details><summary>Show Answer</summary>
Correct Answer: A, B. ACCESS_HISTORY tracks data lineage (source-to-target movement) and gives a unified record of what was accessed and when.
</details>

---

### Question 786
What does Snowflake recommend a user do if they need to connect to Snowflake with a tool or technology that is not listed in Snowflake's partner ecosystem?

- A. Use Snowflake's native API.
- B. Use a custom-built connector.
- C. Contact Snowflake Support for a new driver.
- D. Connect through Snowflake's JDBC or ODBC drivers.

<details><summary>Show Answer</summary>
Correct Answer: D. For tools outside the certified partner ecosystem, Snowflake recommends connecting via its standard JDBC or ODBC drivers.
</details>

---

### Question 787
What is the expiration period for a file URL used to access unstructured data in cloud storage?

- A. The remainder of the session
- B. An unlimited amount of time
- C. The length of time specified in the expiration_time
- D. The same length of time as the expiration period for the query results cache

<details><summary>Show Answer</summary>
Correct Answer: B. Unlike scoped or pre-signed URLs, a file URL does not expire.
</details>

---

### Question 788
Which applications can use key pair authentication? (Choose two.)

- A. Snowflake Marketplace
- B. SnowCD
- C. Snowsight
- D. SnowSQL
- E. Snowflake connector for Python

<details><summary>Show Answer</summary>
Correct Answer: D, E. Key pair authentication is supported by client tools such as SnowSQL and the Python connector.
</details>

---

### Question 789
Which commands can only be executed using SnowSQL? (Choose two.)

- A. COPY INTO
- B. GET
- C. LIST
- D. PUT
- E. REMOVE

<details><summary>Show Answer</summary>
Correct Answer: B, D. GET and PUT are client-side file transfer commands that require a client such as SnowSQL or a driver — they cannot run through Snowsight worksheets.
</details>

---

### Question 790
A user has enabled the STRIP_OUTER_ARRAY file format option for the COPY INTO {table} command. What else will this format option and command do?

- A. Load the records into separate table rows.
- B. Unload the records from separate table rows.
- C. Load data files in smaller chunks.
- D. Ensure each unique element stores values of a single native data type.

<details><summary>Show Answer</summary>
Correct Answer: A. STRIP_OUTER_ARRAY removes the outer array structure from JSON so each element loads as its own row.
</details>

---

### Question 791
Which objects will incur storage costs associated with Fail-safe?

- A. Temporary tables
- B. Permanent tables
- C. Data files available in internal stages
- D. Data files available in external stages

<details><summary>Show Answer</summary>
Correct Answer: B. Only permanent tables have a Fail-safe period, which incurs additional storage costs.
</details>

---

### Question 792
What technique does Snowflake use to limit the number of micro-partitions scanned by each query?

- A. B-tree
- B. Indexing
- C. Map reduce
- D. Pruning

<details><summary>Show Answer</summary>
Correct Answer: D. Pruning uses stored micro-partition metadata to skip partitions that can't match the query's filter predicates.
</details>

---

### Question 793
What activities can a user with the ORGADMIN role perform? (Choose two.)

- A. Create an INFORMATION_SCHEMA in a database.
- B. View usage information for all accounts in the organization.
- C. Enable database cloning for an account in the organization.
- D. Enable database replication for an account in the organization.
- E. View micro-partition information for all accounts in the organization.

<details><summary>Show Answer</summary>
Correct Answer: B, D. ORGADMIN can view organization-wide usage information and enable database replication across accounts in the organization.
</details>

---

### Question 794
In a managed access schema, who can grant privileges on objects in the schema to other roles? (Choose two.)

- A. The schema owner role
- B. The ORGADMIN system role
- C. The system role
- D. The role with the MANAGE GRANTS privilege
- E. The role that owns the object in the schema

<details><summary>Show Answer</summary>
Correct Answer: A, D. In a managed access schema, only the schema owner role or a role with the global MANAGE GRANTS privilege can grant privileges — individual object owners can no longer do so.
</details>

---

### Question 795
What are the recommended steps to optimize a SQL query due to data spilling? (Choose two.)

- A. Clone the base table.
- B. Fetch required attributes only.
- C. Use a larger virtual warehouse.
- D. Process the data in smaller batches.
- E. Add another cluster in the virtual warehouse.

<details><summary>Show Answer</summary>
Correct Answer: B, C. Reducing the columns/rows fetched and using a larger warehouse (more memory/local disk) both reduce or eliminate spilling.
</details>

---

### Question 796
A Snowflake user wants to share unstructured data through the use of secure views. Which URL types can be used? (Choose two.)

- A. Scoped URL
- B. HTTPS URL
- C. Cloud storage URL
- D. File URL
- E. Pre-signed URL

<details><summary>Show Answer</summary>
Correct Answer: A, E. Scoped URLs and pre-signed URLs are the two file-access URL types compatible with sharing unstructured data via secure views.
</details>

---

### Question 797
What are characteristics of reader accounts in Snowflake? (Choose two.)

- A. Reader account users cannot add new data to the account.
- B. Reader account users can share data to other reader accounts.
- C. A single reader account can consume data from multiple provider accounts.
- D. Data consumers are responsible for reader account setup and data usage costs.
- E. Reader accounts enable data consumers to access and query data shared by the provider.

<details><summary>Show Answer</summary>
Correct Answer: A, E. Reader account users can only query shared data (no new data can be added), and the whole point of a reader account is to let a consumer without their own Snowflake account query the provider's shared data. (Note: it is the *provider*, not the consumer, who is responsible for reader account setup and costs, which is why D is incorrect.)
</details>

---

### Question 798
Why should a Snowflake user configure a Secure view? (Choose two.)

- A. To encrypt the data in transit.
- B. To execute faster than a standard view.
- C. To protect hidden data from other users.
- D. To improve the performance of a query.
- E. To hide the view definition from unauthorized users.

<details><summary>Show Answer</summary>
Correct Answer: C, E. Secure views protect underlying data from being inferred by unauthorized users and hide the view's SQL definition from them. (Secure views typically run slower than standard views, not faster.)
</details>

---

### Question 799
Which activities are managed by Snowflake's Cloud Services layer?

- A. Authentication
- B. Access delegation
- C. Data pruning
- D. Data compression
- E. Query parsing and optimization

<details><summary>Show Answer</summary>
Correct Answer: A, E. Authentication and query parsing/optimization are functions of the Cloud Services layer.
</details>

---

### Question 800
The COPY INTO [location] command can unload data from a table directly into which locations? (Choose two.)

- A. A named internal stage
- B. A Snowpipe REST endpoint
- C. A network share on a client machine
- D. A local directory or folder on a client machine
- E. A named external stage that references an external cloud location

<details><summary>Show Answer</summary>
Correct Answer: A, E. COPY INTO [location] unloads directly to internal or external named stages — not to Snowpipe endpoints or local/network file systems (those require a separate GET after unloading to a stage).
</details>

---

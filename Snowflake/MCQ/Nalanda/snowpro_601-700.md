# SnowPro Core Practice Questions (601–700)

---

### Question 601
What is the MOST efficient way to load streaming data into Snowflake?

- A. Use the COPY command.
- B. Use Snowpipe.
- C. Use the Data Wizard.
- D. Use tasks and streams.

<details><summary>Show Answer</summary>
Correct Answer: B. Snowpipe enables continuous, micro-batch loading of streaming data via file-based ingestion.

⚠ **Updated:** As of current documentation, Snowflake also offers **Snowpipe Streaming**, which writes rows directly to Snowflake tables without staging files first, offering lower latency and lower cost than classic Snowpipe for row-level streaming use cases. For a file-based streaming scenario (the classic exam framing), Snowpipe (B) remains the best answer among the listed options, but be aware Snowpipe Streaming is the more modern/efficient choice when ingesting rows directly from a streaming source (e.g., Kafka).
</details>

---

### Question 602
Which COPY INTO statement accurately describes how to unload data from a Snowflake table?

- A. The default value for the SINGLE option is set to FALSE.
- B. By default, COPY INTO [location] statements do not separate table data into a set of files.
- C. The OBJECT_CONSTRUCT function can be combined with the COPY command to convert the rows in a relational table to a single VARIANT column.
- D. If the COMPRESSION option is set to TRUE, a file's name can be specified with the appropriate file extension so that the output file can be compressed.

<details><summary>Show Answer</summary>
Correct Answer: C. OBJECT_CONSTRUCT converts relational rows into a single VARIANT column for unloading (e.g., to JSON).
</details>

---

### Question 603
What command is used to download data from a Snowflake stage?

- A. PUT
- B. INSERT
- C. GET
- D. COPY

<details><summary>Show Answer</summary>
Correct Answer: C. GET downloads files from a stage to a local file system; PUT uploads files to a stage.
</details>

---

### Question 604
By default, which role has privileges to create tables and views in an account?

- A. PUBLIC
- B. SECURITYADMIN
- C. SYSADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: C. SYSADMIN has privileges to create warehouses, databases, and other objects in an account by default.
</details>

---

### Question 605
What does Snowflake recommend as a best practice for using secure views?

- A. Use sequence-generated values.
- B. Programmatically reveal the identifiers.
- C. Use Secure views solely for convenience.
- D. Do not expose the sequence-generated column(s).

<details><summary>Show Answer</summary>
Correct Answer: D. Exposing sequence-generated columns in a secure view can allow users to infer information about the underlying table (e.g., row counts, insert order).
</details>

---

### Question 606
What is the Fail-safe period for a transient table in the Snowflake Enterprise edition and higher?

- A. 0 days
- B. 1 day
- C. 7 days
- D. 14 days

<details><summary>Show Answer</summary>
Correct Answer: A. Transient tables (and temporary tables) have no Fail-safe period.
</details>

---

### Question 607
How does a Snowflake user enable Multi-Factor Authentication (MFA)?

- A. The user must enroll themselves through the web interface.
- B. The user must submit their encrypted private key to Snowflake.
- C. The user must sign up with Duo Mobile to use the service.
- D. The user must configure Snowflake to use Single Sign-On (SSO).

<details><summary>Show Answer</summary>
Correct Answer: A. Users self-enroll in MFA through Snowsight/the web interface; Snowflake's MFA is powered by Duo Security under the hood.
</details>

---

### Question 608
What allows a user to limit the number of credits consumed within a Snowflake account?

- A. Tracking account usage
- B. Creating resource monitors
- C. Automatic virtual warehouse scaling
- D. Automatic clustering

<details><summary>Show Answer</summary>
Correct Answer: B. Resource monitors track and can suspend warehouses/accounts once defined credit thresholds are reached.
</details>

---

### Question 609
Which statement accurately describes Snowflake's architecture?

- A. It utilizes local data for all compute nodes in the platform.
- B. It is a blend of shared-disk and shared-everything database architectures.
- C. It is a hybrid of traditional shared-disk and shared-nothing database architectures.
- D. It reorganizes loaded data into internal optimized, compressed, and row-based format.

<details><summary>Show Answer</summary>
Correct Answer: C. Snowflake combines a shared-disk model (centralized storage accessible from all nodes) with a shared-nothing model (MPP compute per virtual warehouse). Note: data is stored in **columnar**, not row-based, format — which makes D doubly wrong.
</details>

---

### Question 610
Which Snowflake SQL command is used to get a subset of rows randomly from a table?

- A. GENERATOR
- B. LATERAL
- C. PIVOT
- D. SAMPLE

<details><summary>Show Answer</summary>
Correct Answer: D. SAMPLE (or TABLESAMPLE) returns a random subset of rows.
</details>

---

### Question 611
Which statement accurately describes how a virtual warehouse functions?

- A. Increasing the size of a virtual warehouse will always improve data loading performance.
- B. Each virtual warehouse is an independent compute cluster that shares compute resources with other warehouses.
- C. Each virtual warehouse is a compute cluster composed of multiple compute nodes allocated by Snowflake from a cloud provider.
- D. All virtual warehouses share the same compute resources so performance degradation of one warehouse can significantly affect all the other warehouses.

<details><summary>Show Answer</summary>
Correct Answer: C. Warehouses are independent MPP compute clusters; they do not share compute resources with one another (contradicts B and D).
</details>

---

### Question 612
Which Snowflake object can be used to record DML changes made to a table?

- A. Snowpipe
- B. Stage
- C. Stream
- D. Task

<details><summary>Show Answer</summary>
Correct Answer: C. A Stream tracks change data capture (CDC) information — inserts, updates, deletes — for a table.
</details>

---

### Question 613
Which statistic displayed in a Query Profile is specific to external functions?

- A. Bytes written
- B. Total invocations
- C. Partitions scanned
- D. Bytes sent over the network

<details><summary>Show Answer</summary>
Correct Answer: B. "Total invocations" (or "External function calls") is specific to nodes involving external function execution.
</details>

---

### Question 614
If there is queueing in the virtual warehouse load monitoring chart, what should a Snowflake user do?

- A. Decrease the warehouse size.
- B. Decrease the maximum cluster count parameter.
- C. Change the settings to add additional clusters.
- D. Start a separate warehouse and move queued queries there.

<details><summary>Show Answer</summary>
Correct Answer: C. Increasing the maximum cluster count on a multi-cluster warehouse allows Snowflake to spin up additional clusters to absorb concurrent query load and reduce queueing.
</details>

---

### Question 615
Which command is used to generate a zero-copy 'snapshot' of any table, schema, or database?

- A. ALTER
- B. CREATE CLONE
- C. COPY
- D. CREATE REPLICATION GROUP

<details><summary>Show Answer</summary>
Correct Answer: B. Cloning (`CREATE ... CLONE`) creates a zero-copy snapshot of a table, schema, or database.
</details>

---

### Question 616
How long is the load history stored in the metadata of the pipe in Snowpipe?

- A. 2 days
- B. 7 days
- C. 14 days
- D. 64 days

<details><summary>Show Answer</summary>
Correct Answer: C. Snowpipe load history is maintained in pipe metadata for 14 days.
</details>

---

### Question 617
What are the key characteristics of ACCOUNT_USAGE views? (Choose two.)

- A. There is no data latency.
- B. The data latency can vary from 45 minutes to 3 hours.
- C. The historical data is not retained.
- D. The historical data can be retained from 7 days to 6 months.
- E. Records for dropped objects are included in each view.

<details><summary>Show Answer</summary>
Correct Answer: B, E. ACCOUNT_USAGE views have latency (varies by view) and include dropped objects, unlike INFORMATION_SCHEMA views.

⚠ **Updated:** Historical data in ACCOUNT_USAGE views is actually retained for **up to 365 days (1 year)**, not "7 days to 6 months" (D), so D remains incorrect for a different, more precise reason than originally stated. Latency for most views is documented as up to roughly 2 hours currently, though some views still cite ranges similar to 45 minutes–3 hours — this can vary by specific view, so treat exact latency numbers as approximate and check the specific view's documentation if precision matters.
</details>

---

### Question 618
How does a scoped URL expire?

- A. When the data cache clears.
- B. When the persisted query result period ends.
- C. The encoded URL access is permanent.
- D. The length of time is specified in the expiration_time argument.

<details><summary>Show Answer</summary>
Correct Answer: B. A scoped URL is valid only for the duration that the query results persist (typically 24 hours).
</details>

---

### Question 619
What are the available Snowflake scaling modes for configuring multi-cluster virtual warehouses? (Choose two.)

- A. Auto-Scale
- B. Economy
- C. Maximized
- D. Scale-Out
- E. Standard

<details><summary>Show Answer</summary>
Correct Answer: A, C. Multi-cluster warehouses run in either **Maximized** mode (min = max clusters, all running) or **Auto-scale** mode (min < max, clusters start/stop based on load). Note: "Standard" and "Economy" are **scaling policies** that govern *when* auto-scale adds/removes clusters — a related but distinct concept from scaling *mode*.
</details>

---

### Question 620
Which loop type iterates until a condition is true?

- A. FOR
- B. LOOP
- C. REPEAT
- D. WHILE

<details><summary>Show Answer</summary>
Correct Answer: C. REPEAT...UNTIL executes until a condition becomes true; WHILE executes while a condition remains true.
</details>

---

### Question 621
Which property needs to be added to the ALTER WAREHOUSE command to verify the additional compute resources for a virtual warehouse have been fully provisioned?

- A. AUTO_RESUME
- B. WAIT_FOR_COMPLETION
- C. RESOURCE_MONITOR
- D. SCALING_POLICY

<details><summary>Show Answer</summary>
Correct Answer: B. WAIT_FOR_COMPLETION makes the ALTER WAREHOUSE statement block until resizing completes.
</details>

---

### Question 622
How is enhanced authentication achieved in Snowflake? (Choose two.)

- A. Snowflake-managed keys
- B. Object level access control
- C. Password hashing
- D. Multi-Factor Authentication (MFA)
- E. Federated authentication and Single Sign-on (SSO)

<details><summary>Show Answer</summary>
Correct Answer: D, E. MFA and Federated authentication/SSO are Snowflake's enhanced authentication mechanisms beyond basic username/password.
</details>

---

### Question 623
What are the native data types that Snowflake provides to store semi-structured data? (Choose two.)

- A. ARRAY
- B. JSON
- C. ORC
- D. Parquet
- E. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: A, E. ARRAY, OBJECT, and VARIANT are Snowflake's native semi-structured data types. JSON, ORC, and Parquet are file formats, not column data types.
</details>

---

### Question 624
How long is the Fail-safe period for recovering historical data from permanent tables?

- A. 1 day
- B. 3 days
- C. 7 days
- D. 14 days

<details><summary>Show Answer</summary>
Correct Answer: C. Permanent tables have a standard 7-day Fail-safe period.
</details>

---

### Question 625
What does the average_overlaps in the output of SYSTEM$CLUSTERING_INFORMATION refer to?

- A. The average number of micro-partitions in Time Travel
- B. The average number of partitions physically stored in the same location
- C. The average number of micro-partitions which contain overlapping value ranges
- D. The average number of micro-partitions in the table associated with cloned objects

<details><summary>Show Answer</summary>
Correct Answer: C. average_overlaps measures how many micro-partitions overlap in value range with a given micro-partition — a key clustering health metric.
</details>

---

### Question 626
If queries start to queue in a multi-cluster virtual warehouse, an additional compute cluster starts immediately under what setting?

- A. Auto-scale mode
- B. Maximized mode
- C. Economy scaling policy
- D. Standard scaling policy

<details><summary>Show Answer</summary>
Correct Answer: D. The Standard scaling policy favors starting additional clusters quickly to minimize queuing (at the cost of credit efficiency), whereas Economy favors conserving credits and may allow some queuing.
</details>

---

### Question 627
When floating-point number columns are unloaded to CSV or JSON files, Snowflake truncates the values to approximately what?

- A. (12,2)
- B. (10,4)
- C. (14,8)
- D. (15,9)

<details><summary>Show Answer</summary>
Correct Answer: D. Floating-point values are truncated to approximately 15 digits of precision, 9 after the decimal point, when unloaded to CSV/JSON.
</details>

---

### Question 628
By definition, a secure view is exposed only to users with what privilege?

- A. IMPORT SHARE
- B. OWNERSHIP
- C. REFERENCES
- D. USAGE

<details><summary>Show Answer</summary>
Correct Answer: B. Only the OWNERSHIP role/privilege context can see the secure view's definition and details; other users cannot.
</details>

---

### Question 629
What happens when a user exits Snowsight during a session where a query is running?

- A. Snowflake executes the query during the same session immediately.
- B. Snowflake cancels any queries submitted during this session that are still running.
- C. Snowflake will cancel any queries submitted during this session after 24 hours.
- D. Snowflake will continue to execute and complete upon the next login.

<details><summary>Show Answer</summary>
Correct Answer: D. Closing the browser/UI does not cancel a running query — Snowflake continues executing it server-side.
</details>

---

### Question 630
Which native data types are used for storing semi-structured data in Snowflake? (Choose two.)

- A. NUMBER
- B. OBJECT
- C. STRING
- D. VARCHAR
- E. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: B, E. OBJECT and VARIANT are native semi-structured types (along with ARRAY).
</details>

---

### Question 631
Which columns are available in the output of a Snowflake directory table? (Choose two.)

- A. CATALOG_NAME
- B. FILE_NAME
- C. LAST_MODIFIED
- D. RELATIVE_PATH
- E. STAGE_NAME

<details><summary>Show Answer</summary>
Correct Answer: C, D. A directory table's output includes columns such as RELATIVE_PATH, SIZE, LAST_MODIFIED, MD5, ETAG, and FILE_URL.
</details>

---

### Question 632
What is used to diagnose and troubleshoot network connections to Snowflake?

- A. SnowCD
- B. Snowpark
- C. Snowsight
- D. SnowSQL

<details><summary>Show Answer</summary>
Correct Answer: A. SnowCD (Snowflake Connectivity Diagnostic Tool) tests and diagnoses network connectivity to Snowflake.
</details>

---

### Question 633
Which Snowflake object records Data Manipulation Language (DML) changes so that actions can be taken using the changed data?

- A. Pipe
- B. Stream
- C. Task
- D. View

<details><summary>Show Answer</summary>
Correct Answer: B. Streams record DML changes (CDC) for downstream processing, often paired with Tasks.
</details>

---

### Question 634
By default, the COPY INTO [location] statement will separate table data into a set of output files to take advantage of which Snowflake feature?

- A. Query acceleration
- B. Query plan caching
- C. Parallel processing
- D. Time Travel

<details><summary>Show Answer</summary>
Correct Answer: C. Splitting unloaded data into multiple files enables parallel operations across warehouse nodes.
</details>

---

### Question 635
Which command can be used to view the allowed and blocked IP list of a network policy?

- A. ALTER NETWORK POLICY
- B. CREATE NETWORK POLICY
- C. DESCRIBE NETWORK POLICY
- D. SHOW NETWORK POLICIES

<details><summary>Show Answer</summary>
Correct Answer: C. DESCRIBE NETWORK POLICY shows the allowed/blocked IP lists for a specific policy; SHOW NETWORK POLICIES only lists policy names/metadata.
</details>

---

### Question 636
Which file functions are nondeterministic? (Choose two.)

- A. BUILD_SCOPED_FILE_URL
- B. GET_STAGE_LOCATION
- C. GET_PRESIGNED_URL
- D. BUILD_STAGE_FILE_URL

<details><summary>Show Answer</summary>
Correct Answer: A, C. BUILD_SCOPED_FILE_URL and GET_PRESIGNED_URL generate URLs that vary between calls (e.g., due to expiration/token), making them nondeterministic.
</details>

---

### Question 637
How can a Snowflake user optimize query performance? (Choose two.)

- A. Create a view.
- B. Cluster a table.
- C. Enable the search optimization service.
- D. Enable Time Travel.
- E. Index a table.

<details><summary>Show Answer</summary>
Correct Answer: B, C. Clustering and the search optimization service are the two primary performance-tuning levers listed here. (Snowflake has no traditional indexes, so E is invalid.)
</details>

---

### Question 638
What is the MINIMUM role required to set the value for the DATA_RETENTION_TIME_IN_DAYS account parameter?

- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. SYSADMIN
- D. ORGADMIN

<details><summary>Show Answer</summary>
Correct Answer: A. Account-level parameters such as DATA_RETENTION_TIME_IN_DAYS require ACCOUNTADMIN (though it can be set at lower object levels, like a table, by users with appropriate object privileges).
</details>

---

### Question 639
Which file format will keep floating-point numbers from being truncated when data is unloaded?

- A. CSV
- B. JSON
- C. ORC
- D. Parquet

<details><summary>Show Answer</summary>
Correct Answer: D. Parquet preserves full floating-point precision; CSV/JSON truncate to ~(15,9).
</details>

---

### Question 640
A user has semi-structured data to load into Snowflake but is not sure what types of operations will need to be performed on the data. Based on this situation, what type of column does Snowflake recommend be used?

- A. ARRAY
- B. OBJECT
- C. STRING
- D. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: D. VARIANT is the flexible, general-purpose type recommended when future query patterns are unknown.
</details>

---

### Question 641
Which feature helps evaluate virtual warehouse performance impacted by queuing?

- A. Resource monitor
- B. Query history
- C. Load monitoring chart
- D. Task history

<details><summary>Show Answer</summary>
Correct Answer: C. The warehouse load monitoring chart visualizes queued vs. running queries over time.
</details>

---

### Question 642
Which Snowflake object can be created to be temporary?

- A. Role
- B. Stage
- C. User
- D. Storage integration

<details><summary>Show Answer</summary>
Correct Answer: B. Stages (along with tables) support the TEMPORARY object type; roles, users, and storage integrations do not.
</details>

---

### Question 643
Which stream type can be used for tracking the records in external tables?

- A. Append-only
- B. External
- C. Insert-only
- D. Standard

<details><summary>Show Answer</summary>
Correct Answer: C. Insert-only streams are used to track new records added to external tables (external tables don't support standard/append-only streams).
</details>

---

### Question 644
What is the recommended approach for unloading data to a cloud storage location from Snowflake?

- A. Use a third-party tool to unload the data to cloud storage.
- B. Unload the data directly to the cloud storage location.
- C. Unload the data to a local system, then upload it to cloud storage.
- D. Unload the data to a user stage, then upload the data to cloud storage.

<details><summary>Show Answer</summary>
Correct Answer: B. Unloading directly to an external stage/cloud location is the recommended, most efficient path.
</details>

---

### Question 645
Which command is used to unload files from an internal or external stage to a local file system?

- A. COPY INTO
- B. GET
- C. PUT
- D. TRANSFER

<details><summary>Show Answer</summary>
Correct Answer: B. GET downloads staged files to a local file system.
</details>

---

### Question 646
A tabular User Defined Function (UDF) is defined by specifying a return clause that contains which keyword?

- A. ROW_NUMBER
- B. TABLE
- C. TABULAR
- D. VALUES

<details><summary>Show Answer</summary>
Correct Answer: B. A tabular (table-valued) UDF uses `RETURNS TABLE (...)`.
</details>

---

### Question 647
Which SQL statement will require a virtual warehouse to run?

- A. SELECT COUNT(*) FROM TBL_EMPLOYEE;
- B. ALTER TABLE TBL_EMPLOYEE ADD COLUMN EMP_REGION VARCHAR(20);
- C. INSERT INTO TBL_EMPLOYEE (EMP_ID, EMP_NAME, EMP_SALARY, DEPT) VALUES(1, 'Adam', 20000, 'Finance');
- D. CREATE OR REPLACE TABLE TBL_EMPLOYEE (EMP_ID NUMBER);

<details><summary>Show Answer</summary>
Correct Answer: C. INSERT requires compute to write data. DDL (ALTER, CREATE) is metadata-only, and simple aggregate queries like COUNT(*) can sometimes be resolved from metadata without a running warehouse.
</details>

---

### Question 648
Which REST API endpoint can be used with unstructured data?

- A. insertReport
- B. PUT
- C. GET
- D. loadHistoryScan

<details><summary>Show Answer</summary>
Correct Answer: C. The GET endpoint is used to retrieve unstructured files via presigned/scoped URLs.
</details>

---

### Question 649
Which query contains a Snowflake hosted file URL in a directory table for a stage named bronzestage?

- A. list @bronzestage;
- B. select * from directory(@bronzestage);
- C. select metadata$filename from @bronzestage;
- D. select * from @bronzestage;

<details><summary>Show Answer</summary>
Correct Answer: B. Querying `directory(@stage_name)` returns the directory table, which includes the FILE_URL column.
</details>

---

### Question 650
Which feature is integrated to support Multi-Factor Authentication (MFA) at Snowflake?

- A. Authy
- B. Duo Security
- C. OneLogin
- D. RSA SecurID Access

<details><summary>Show Answer</summary>
Correct Answer: B. Snowflake's native MFA is powered by Duo Security.
</details>

---

### Question 651
In which Snowflake layer does Snowflake reorganize data into its internal optimized, compressed, columnar format?

- A. Cloud Services
- B. Database Storage
- C. Query Processing
- D. Metadata Management

<details><summary>Show Answer</summary>
Correct Answer: B. The Database Storage layer handles reorganizing and storing data in Snowflake's proprietary columnar format.
</details>

---

### Question 652
When can user session variables be accessed in a Snowflake scripting procedure?

- A. When the procedure is defined as STRICT.
- B. When the procedure is defined to execute as CALLER.
- C. When the procedure is defined to execute as OWNER.
- D. When the procedure is defined with an argument that has the same name and type as the session variable.

<details><summary>Show Answer</summary>
Correct Answer: B. Stored procedures executed with `EXECUTE AS CALLER` run in the caller's session context, so they can access the caller's session variables.
</details>

---

### Question 653
What computer languages can be selected when creating User-Defined Functions (UDFs) using the Snowpark API?

- A. Swift
- B. JavaScript
- C. Java, Scala, Python
- D. C++

<details><summary>Show Answer</summary>
Correct Answer: C. Snowpark supports Java, Scala, and Python for UDFs (JavaScript UDFs exist but are a separate, non-Snowpark mechanism).
</details>

---

### Question 654
A user needs to ingest 1 GB of data that is available in an external stage using a COPY INTO command. How can this be done with MAXIMUM performance and the LEAST cost?

- A. Ingest the data in a compressed format as a single file.
- B. Ingest the data in an uncompressed format as a single file.
- C. Split the file into smaller files of 100-250 MB each, compress and ingest each of the smaller files.
- D. Split the file into smaller files of 100-250 MB each and ingest each of the smaller files in an uncompressed format.

<details><summary>Show Answer</summary>
Correct Answer: C. Snowflake's file-sizing best practice is 100–250 MB compressed files to maximize parallel loading efficiency while minimizing storage/network cost.
</details>

---

### Question 655
A Snowflake user has two tables that contain numeric values and is trying to find which values are present in both tables. Which set operator should be used?

- A. INTERSECT
- B. MERGE
- C. MINUS
- D. UNION

<details><summary>Show Answer</summary>
Correct Answer: A. INTERSECT returns only rows/values present in both result sets.
</details>

---

### Question 656
A view is defined on a permanent table. A temporary table with the same name is created in the same schema as the referenced table. What will a query from the view return?

- A. The data from the permanent table.
- B. The data from the temporary table.
- C. An error stating that the view could not be compiled.
- D. An error stating that the referenced object could not be uniquely identified.

<details><summary>Show Answer</summary>
Correct Answer: A. A view's reference is bound at creation time to the permanent table; a same-named temporary table created later does not override that binding for the view.
</details>

---

### Question 657
Which file function generates a Snowflake-hosted file URL to a staged file using the stage name and relative file path as inputs?

- A. BUILD_STAGE_FILE_URL
- B. GET_ABSOLUTE_PATH
- C. GENERATE_PRESIGNED_URL
- D. BUILD_SCOPED_FILE_URL

<details><summary>Show Answer</summary>
Correct Answer: A. BUILD_STAGE_FILE_URL generates a permanent Snowflake-hosted URL from a stage name and relative path.
</details>

---

### Question 658
Which service or feature in Snowflake is used to improve the performance of certain types of lookup and analytical queries that use an extensive set of WHERE conditions?

- A. Data classification
- B. Query acceleration service
- C. Search optimization service
- D. Tagging

<details><summary>Show Answer</summary>
Correct Answer: C. The search optimization service speeds up selective point lookups and queries with many varied WHERE clause predicates.
</details>

---

### Question 659
What is the name of the SnowSQL file that can store connection information?

- A. history
- B. config
- C. snowsql.cnf
- D. credentials

<details><summary>Show Answer</summary>
Correct Answer: B. The `config` file stores SnowSQL connection parameters and named connections.
</details>

---

### Question 660
How do secure views compare to non-secure views in Snowflake?

- A. Secure views are slower compared to non-secure views.
- B. Non-secure views are preferred over secure views when sharing data.
- C. Secure views are similar to materialized views in that they are the most performant.
- D. End users are unable to see the view definition, and internal optimizations differ to protect underlying data.

<details><summary>Show Answer</summary>
Correct Answer: D. Secure views hide the definition from unauthorized users and forgo certain query-optimizer disclosures (which can slightly reduce performance) to protect underlying data.
</details>

---

### Question 661
Which type of join will list all rows in the specified table, even if those rows have no match in the other table?

- A. Cross join
- B. Inner join
- C. Natural join
- D. Outer join

<details><summary>Show Answer</summary>
Correct Answer: D. An outer join (left/right/full) preserves unmatched rows from one or both tables.
</details>

---

### Question 662
When unloading data to an external stage, what is the MAXIMUM file size supported per file?

- A. 1 GB
- B. 5 GB
- C. 16 MB
- D. 250 MB

<details><summary>Show Answer</summary>
Correct Answer: B. 5 GB is the documented maximum unloaded file size for external stages (S3, GCS, Azure). This is confirmed in current Snowflake documentation.
</details>

---

### Question 663
How long does Snowflake retain information in the ACCESS_HISTORY view?

- A. 14 days
- B. 28 days
- C. 90 days
- D. 365 days

<details><summary>Show Answer</summary>
Correct Answer: D. ACCESS_HISTORY (an ACCOUNT_USAGE view) retains up to 365 days of history, consistent with other ACCOUNT_USAGE views.
</details>

---

### Question 664
Which encryption type will enable client-side encryption for a directory table?

- A. AWS_CSE
- B. SNOWFLAKE_SSE
- C. SNOWFLAKE_FULL
- D. CLIENT_SIDE_ENCRYPTION

<details><summary>Show Answer</summary>
Correct Answer: A. AWS_CSE (client-side encryption) is required for directory tables on client-side-encrypted external stages; server-side encryption (SNOWFLAKE_SSE) is not supported for directory tables on external stages in the same way.
</details>

---

### Question 665
If file format options are specified in multiple locations, the load operation selects which option FIRST to apply in order of precedence?

- A. Table definition
- B. Stage definition
- C. Session level
- D. COPY INTO TABLE statement

<details><summary>Show Answer</summary>
Correct Answer: D. Options specified directly in the COPY INTO &lt;table&gt; statement take the highest precedence, overriding stage- and table-level defaults.
</details>

---

### Question 666
A complex SQL query involving eight tables with joins is taking a while to execute. The Query Profile shows that all partitions are being scanned. What is causing the query performance issue?

- A. Pruning is not being performed efficiently.
- B. A massive volume of data is being fetched, with many joins applied.
- C. Incorrect joins are being used, leading to scanning and pulling too many records.
- D. The columns in the micro-partitions need granular ordering based on the dataset.

<details><summary>Show Answer</summary>
Correct Answer: A. When all micro-partitions are scanned instead of a pruned subset, it indicates partition pruning isn't effective — usually due to poor clustering relative to the query's filter predicates.
</details>

---

### Question 667
What does the search optimization service support?

- A. External tables
- B. Materialized views
- C. Equality searches, casts on table columns (except for fixed-point numbers cast to strings), and IN predicates.
- D. Subqueries that return multiple rows

<details><summary>Show Answer</summary>
Correct Answer: C. Search optimization accelerates equality/IN lookups and most column casts (with documented exceptions like fixed-point-to-string casts).
</details>

---

### Question 668
Which table type is no longer available after the close of the session and therefore has no Fail-safe or Time Travel option?

- A. Permanent
- B. External
- C. Temporary
- D. Transient

<details><summary>Show Answer</summary>
Correct Answer: C. Temporary tables exist only for the session and are dropped automatically at session end, with no Fail-safe and Time Travel limited to that session's lifetime.
</details>

---

### Question 669
How many network policies can be assigned to an account or specific user at a time?

- A. One
- B. Two
- C. Five
- D. Unlimited

<details><summary>Show Answer</summary>
Correct Answer: A. Only one network policy can be active at a time for an account or a given user.
</details>

---

### Question 670
What is a characteristic of a tag associated with a masking policy?

- A. A tag can be dropped after a masking policy is assigned.
- B. A tag can have only one masking policy for each data type.
- C. A tag can have multiple masking policies for each data type.
- D. A tag can have multiple masking policies with varying data types.

<details><summary>Show Answer</summary>
Correct Answer: B. A single tag can be associated with only one masking policy per data type (though it can have different masking policies for different data types).
</details>

---

### Question 671
Which clients does Snowflake support Multi-Factor Authentication (MFA) token caching for? (Choose two.)

- A. Go driver
- B. Node.js driver
- C. ODBC driver
- D. Python connector
- E. Spark connector

<details><summary>Show Answer</summary>
Correct Answer: C, D. MFA token caching is documented as supported for the ODBC driver and Python connector (among other specific drivers), reducing repeated MFA prompts.
</details>

---

### Question 672
What is the Snowflake recommended Parquet file size when querying from external tables to optimize the number of parallel scanning operations?

- A. 1-16 MB
- B. 16-128 MB
- C. 100-250 MB
- D. 256-512 MB

<details><summary>Show Answer</summary>
Correct Answer: D. For external tables/Parquet, Snowflake recommends larger files (in the low hundreds of MB range) to balance parallelism and file overhead — larger than the general 100–250 MB rule of thumb used for standard staged loads.
</details>

---

### Question 673
Which data types can be used in a Snowflake table that holds semi-structured data? (Choose two.)

- A. ARRAY
- B. BINARY
- C. INTEGER
- D. VARIANT
- E. VARCHAR

<details><summary>Show Answer</summary>
Correct Answer: A, D. ARRAY and VARIANT are semi-structured data types.
</details>

---

### Question 674
Which constraint is actively enforced in Snowflake?

- A. FOREIGN KEY
- B. NOT NULL
- C. PRIMARY KEY
- D. UNIQUE KEY

<details><summary>Show Answer</summary>
Correct Answer: B. NOT NULL is the only constraint type actively enforced. PRIMARY KEY, UNIQUE, and FOREIGN KEY are supported for metadata/documentation and query optimization purposes but are not enforced.
</details>

---

### Question 675
Which pages are included in the Activity area of Snowsight? (Choose two.)

- A. Contacts
- B. Sharing settings
- C. Copy History
- D. Query History
- E. Automatic Clustering History

<details><summary>Show Answer</summary>
Correct Answer: C, D. The Activity area in Snowsight includes Query History and Copy History pages.
</details>

---

### Question 676
When should a user consider disabling auto-suspend for a virtual warehouse? (Choose two.)

- A. When users will be using compute at different times throughout a 24/7 period
- B. When managing a steady workload
- C. When the compute must be available with no delay or lag time
- D. When the user does not want to have to manually turn on the warehouse each time it is needed
- E. When the warehouse is shared between different teams

<details><summary>Show Answer</summary>
Correct Answer: B, C. Disabling auto-suspend makes sense for steady, continuous workloads or when zero warehouse resume latency is required — the credit cost tradeoff is acceptable in these cases.
</details>

---

### Question 677
What can a Snowflake user do in the Activity section in Snowsight?

- A. Create dashboards.
- B. Write and run SQL queries.
- C. Explore databases and objects.
- D. Monitor query performance and history.

<details><summary>Show Answer</summary>
Correct Answer: D. The Activity section is for monitoring query performance, query history, and copy history.
</details>

---

### Question 678
How does Snowflake reorganize data when it is loaded? (Choose two.)

- A. Binary
- B. Columnar format
- C. Compressed format
- D. Raw format
- E. Zipped format

<details><summary>Show Answer</summary>
Correct Answer: B, C. Loaded data is reorganized into a compressed, columnar internal format.
</details>

---

### Question 679
Which operations are handled in the Cloud Services layer of Snowflake? (Choose two.)

- A. Security and Authentication
- B. Data Storage
- C. Data visualization
- D. Query computation
- E. Metadata management

<details><summary>Show Answer</summary>
Correct Answer: A, E. The Cloud Services layer handles authentication, infrastructure management, metadata, access control, and query optimization/parsing (not the actual storage or compute execution).
</details>

---

### Question 680
At which point is data encrypted when using a PUT command?

- A. When it reaches the virtual warehouse
- B. When it gets micro-partitioned
- C. Client-side before it is sent from the user's machine
- D. After it reaches the internal stage

<details><summary>Show Answer</summary>
Correct Answer: C. PUT encrypts data client-side (128-bit or 256-bit AES) before uploading it to the stage.
</details>

---

### Question 681
What type of columns does Snowflake recommend to be used as clustering keys? (Choose two.)

- A. A VARIANT column
- B. A column with very low cardinality
- C. A column with very high cardinality
- D. A column that is most actively used in selective filters
- E. A column that is most actively used in join predicates

<details><summary>Show Answer</summary>
Correct Answer: D, E. Clustering keys should be columns frequently used in WHERE filters or JOIN predicates — and ideally with moderate (not extremely low or high) cardinality, though the best answer choices here focus on usage pattern rather than cardinality extremes.
</details>

---

### Question 682
Which objects together comprise a namespace in Snowflake? (Choose two.)

- A. Account
- B. Database
- C. Schema
- D. Table
- E. Virtual warehouse

<details><summary>Show Answer</summary>
Correct Answer: B, C. A namespace in Snowflake is composed of a database and schema (e.g., `db.schema.object`).
</details>

---

### Question 683
What statistical information in a Query Profile indicates that the query is too large to fit in memory? (Choose two.)

- A. Bytes spilled to local disk cache
- B. Bytes spilled to local storage
- C. Bytes spilled to remote cache
- D. Bytes spilled to remote storage
- E. Bytes spilled to remote metastore

<details><summary>Show Answer</summary>
Correct Answer: B, D. "Bytes spilled to local storage" and "Bytes spilled to remote storage" indicate the warehouse ran out of memory and had to spill intermediate results to disk (local) or cloud storage (remote, which is slower and more costly).
</details>

---

### Question 684
How do Snowflake data providers share data that resides in different databases?

- A. External tables
- B. Secure views
- C. Materialized views
- D. User-Defined Functions

<details><summary>Show Answer</summary>
Correct Answer: B. Secure views can reference objects across databases and are the standard mechanism for cross-database data sharing.
</details>

---

### Question 685
What operations can be performed while loading a simple CSV file into a Snowflake table using the COPY INTO command? (Choose two.)

- A. Performing aggregate calculations
- B. Reordering the columns
- C. Grouping by operations
- D. Converting the datatypes
- E. Selecting the first few rows

<details><summary>Show Answer</summary>
Correct Answer: B, D. During a COPY INTO load, you can reorder columns and convert data types; aggregate/group-by transformations are not supported during COPY INTO load transformations.
</details>

---

### Question 686
Which commands support a multiple-statement request to run and update Snowflake data? (Choose two.)

- A. CALL
- B. COMMIT
- C. GET
- D. ROLLBACK
- E. EXPLAIN

<details><summary>Show Answer</summary>
Correct Answer: B, D. COMMIT and ROLLBACK are transaction control commands used to finalize or undo multi-statement transactions.
</details>

---

### Question 687
Why should a Snowflake user implement a secure view? (Choose two.)

- A. To store unstructured data
- B. To increase query performance
- C. To limit access to sensitive data
- D. To optimize query concurrency and queuing
- E. To hide view definition and details from unauthorized users

<details><summary>Show Answer</summary>
Correct Answer: C, E. Secure views restrict access to sensitive underlying data and hide the view's definition from unauthorized users.
</details>

---

### Question 688
At what levels can a resource monitor be configured? (Choose two.)

- A. Account
- B. Database
- C. Organization
- D. Schema
- E. Virtual warehouse

<details><summary>Show Answer</summary>
Correct Answer: A, E. Resource monitors can be set at the account level or assigned to specific virtual warehouses.
</details>

---

### Question 689
What activities can be monitored by a user directly from Snowsight's Activity tab without using the Account_Usage views? (Choose two.)

- A. Login history
- B. Query history
- C. Copy history
- D. Event usage history
- E. Virtual warehouse metering history

<details><summary>Show Answer</summary>
Correct Answer: B, C. Query History and Copy History are directly available in the Snowsight Activity tab.
</details>

---

### Question 690
What can a Snowflake user do with the information included in the details section of a Query Profile?

- A. Determine the total duration of the query execution.
- B. Determine the role of the user who ran the query.
- C. Determine the source system that the queried table is from.
- D. Determine if the query was on structured or semi-structured data.

<details><summary>Show Answer</summary>
Correct Answer: A. The Query Profile details panel shows execution time/duration statistics, among other performance metrics.
</details>

---

### Question 691
How can a Snowflake user access a JSON object, given the following table? (Choose two.)

`{ "id": "1234", "customer": { "name": "user" } }`

- A. src:customer.name
- B. src:customer.name::string
- C. src['customer']['name']
- D. src.customer.name

<details><summary>Show Answer</summary>
Correct Answer: B, C. Colon notation (`src:customer.name`) requires an explicit cast (`::string`) to return a usable string type, and bracket notation (`src['customer']['name']`) is a valid alternative path syntax.
</details>

---

### Question 692
Which term is used to describe information about disk usage for operations where intermediate results cannot be accommodated in a Snowflake virtual warehouse memory?

- A. Pruning
- B. Spilling
- C. Join explosion
- D. Queueing

<details><summary>Show Answer</summary>
Correct Answer: B. "Spilling" describes when intermediate query results exceed available memory and must be written to local or remote disk.
</details>

---

### Question 693
There are two Snowflake accounts in the same cloud provider region: one is production and the other is non-production. How can data be easily transferred from the production account to the non-production account?

- A. Clone the data from the production account to the non-production account.
- B. Create a data share from the production to the non-production account.
- C. Create a subscription in the production account and have it publish to the non-production account.
- D. Create a reader account using the production account and link the reader account to the non-production account.

<details><summary>Show Answer</summary>
Correct Answer: B. Secure Data Sharing lets a provider account share data with a consumer account in the same region without copying/moving data. (Note: cross-account cloning is not directly possible — cloning only works within the same account, so A is invalid, reinforcing B as correct.)
</details>

---

### Question 694
A user is unloading data to a Stage using this command:

```sql
COPY INTO @message
FROM (SELECT object_construct('id', 1, 'first_name', 'Snowflake', 'last_name', 'User', 'city', 'Bozeman'))
file_format = (type = json)
```

What will the output file in the stage be?

- A. A single compressed JSON file with a single VARIANT column.
- B. Multiple compressed JSON files with a single VARIANT column.
- C. A single uncompressed JSON file with multiple VARIANT columns.
- D. Multiple uncompressed JSON files with multiple VARIANT columns.

<details><summary>Show Answer</summary>
Correct Answer: A. OBJECT_CONSTRUCT produces a single VARIANT column per row, and by default COPY INTO unloads data compressed (gzip) — for this small single-row query, this results in one compressed file.
</details>

---

### Question 695
A JSON file that contains lots of dates needs to be loaded into Snowflake. The user wants to ensure optimal performance while querying the data. How can this be achieved?

- A. Flatten the data and store it in structured data types in a flattened table. Query the table.
- B. Store the data in a table with a VARIANT data type. Query the table.
- C. Store the data in a table with a VARIANT data type and include indexing while loading the table. Query the table.
- D. Store the data in an external stage and create views on top of it. Query the views.

<details><summary>Show Answer</summary>
Correct Answer: A. Flattening semi-structured data into structured columns (rather than leaving it all in VARIANT) generally yields better query performance, since structured columns benefit more fully from pruning, clustering, and type-specific optimizations. (Snowflake has no user-managed indexing, so C is invalid.)
</details>

---

### Question 696
When referring to User-Defined Function (UDF) names in Snowflake, what does the term "overloading" mean?

- A. There are multiple SQL UDFs with the same names and the same number of arguments.
- B. There are multiple SQL UDFs with the same names and the same number of argument types.
- C. There are multiple SQL UDFs with the same names but with a different number of arguments or argument types.
- D. There are multiple SQL UDFs with different names but the same number of arguments or argument types.

<details><summary>Show Answer</summary>
Correct Answer: C. Overloading means multiple UDFs share a name but differ in argument count or argument types, allowing Snowflake to resolve which one to call based on the call signature.
</details>

---

### Question 697
Which key governance feature in Snowflake allows users to identify data objects that contain sensitive data and their related objects?

- A. Object tagging
- B. Data classification
- C. Row access policy
- D. Column-level security

<details><summary>Show Answer</summary>
Correct Answer: B. Data classification automatically analyzes and tags columns that may contain sensitive/PII data, helping identify related objects.
</details>

---

### Question 698
What can a Snowflake user do in the Admin area of Snowsight?

- A. Analyze query processing plans.
- B. Write queries and execute them.
- C. Provide an overview of the listings in the Snowflake Marketplace.
- D. Explore billing, usage, warehouses, resource monitors, users, and roles.

<details><summary>Show Answer</summary>
Correct Answer: D. The Admin area covers account administration: billing/usage, warehouses, resource monitors, users, and roles.
</details>

---

### Question 699
Which function generates a Snowflake hosted file URL to a staged file using the stage name and relative file path as inputs?

- A. GET_STAGE_URL
- B. BUILD_STAGE_FILE_URL
- C. GET_PRESIGNED_URL
- D. BUILD_SCOPED_FILE_URL

<details><summary>Show Answer</summary>
Correct Answer: B. BUILD_STAGE_FILE_URL builds a permanent Snowflake-hosted file URL from a stage name and relative path (duplicate concept to Question 657).
</details>

---

### Question 700
What is the purpose of using the OBJECT_CONSTRUCT function with the COPY INTO command?

- A. Reorder the columns in a relational table and then unload the data into a file.
- B. Convert the rows in a relational table to a single VARIANT column and then unload the rows into a file.
- C. Reorder the data columns according to a target table definition and then unload the rows into the table.
- D. Convert the rows in a source file to a single VARIANT column and then load the rows from the file to a variant table.

<details><summary>Show Answer</summary>
Correct Answer: B. OBJECT_CONSTRUCT is used during unload to collapse relational row data into a single VARIANT (JSON-like) column before writing it to a file.
</details>

---

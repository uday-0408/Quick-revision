# SnowPro Core Practice Questions (201–300)

*Reconstructed and cleaned from a garbled OCR source. Answers have been cross-checked against current Snowflake documentation (as of July 2026); corrections are flagged with ⚠ Updated. Questions are renumbered sequentially for clarity, since the original numbering in the source was inconsistent.*

---

### Question 201
What Snowflake features support virtual warehouses in handling high-concurrency workloads? (Choose two.)

- A. The ability to add warehouses
- B. The use of warehouse auto-scaling
- C. The ability to resize warehouses
- D. The use of multi-clustered warehouses
- E. The use of warehouse indexing

<details><summary>Show Answer</summary>
Correct Answer: B, D. Multi-cluster warehouses (with auto-scaling) automatically start and stop additional clusters to absorb concurrent query load. Resizing (C) helps individual query performance, not concurrency, and "warehouse indexing" (E) is not a real Snowflake feature.
</details>

---

### Question 202
Which `COPY INTO <location>` option outputs the unloaded data into a single file?

- A. `SINGLE = TRUE`
- B. `MAX_FILE_SIZE`
- C. `FILE_FORMAT`
- D. `MULTIPLE = FALSE`

<details><summary>Show Answer</summary>
Correct Answer: A. Setting `SINGLE = TRUE` in `COPY INTO <location>` unloads all query results into one file instead of Snowflake's default of splitting output into multiple files.
</details>

---

### Question 203
In which scenarios would an account pay Cloud Services costs? (Choose two.)

- A. Compute Credits = 50, Cloud Services Credits = 10
- B. Compute Credits = 80, Cloud Services Credits = 5
- C. Compute Credits = 100, Cloud Services Credits = 8
- D. Compute Credits = 120, Cloud Services Credits = 10
- E. Compute Credits = 200, Cloud Services Credits = 26

<details><summary>Show Answer</summary>
Correct Answer: A, E. Snowflake only charges for Cloud Services usage that exceeds 10% of daily compute credit consumption. A (10 > 5) and E (26 > 20) both exceed that 10% threshold; B, C, and D all fall at or under it.
</details>

---

### Question 204
A user created a new worksheet within the Snowsight UI and wants to share it with teammates. How can this worksheet be shared?

- A. Create a zero-copy clone of the worksheet and grant permissions to teammates.
- B. Create a private Data Exchange so that any teammate can use the worksheet.
- C. Share the worksheet with teammates directly within Snowsight.
- D. Create a database and grant all permissions to teammates.

<details><summary>Show Answer</summary>
Correct Answer: C. Snowsight worksheets have a native "Share" option that lets you grant view or edit access to specific users or roles.
</details>

---

### Question 205
How can a row access policy be applied to a table or a view? (Choose two.)

- A. Within the policy DDL at creation time
- B. Within the `CREATE TABLE` or `CREATE VIEW` statement
- C. Via a future grant that applies the policy to all objects in a schema
- D. Within a separate control table
- E. Using the command `ALTER <object> ADD ROW ACCESS POLICY <policy>`

<details><summary>Show Answer</summary>
Correct Answer: B, E. A row access policy can be attached when the table/view is first created, or later via `ALTER TABLE`/`ALTER VIEW ... ADD ROW ACCESS POLICY`.
</details>

---

### Question 206
Which command can be used to load local data files into a Snowflake stage?

- A. `JOIN`
- B. `COPY INTO`
- C. `PUT`
- D. `GET`

<details><summary>Show Answer</summary>
Correct Answer: C. `PUT` uploads (stages) local files into a Snowflake stage. `COPY INTO` then loads staged files into a table, and `GET` downloads files from a stage back to local storage.
</details>

---

### Question 207
What types of data listings are available in the Snowflake Marketplace? (Choose two.)

- A. Reader
- B. Consumer
- C. Vendor
- D. Standard
- E. Personalized

<details><summary>Show Answer</summary>
Correct Answer: D, E. Snowflake Marketplace listings are published as either Standard listings (available to any consumer) or Personalized listings (shared with a specific consumer account).
</details>

---

### Question 208
What is the maximum Time Travel retention period for a temporary Snowflake table?

- A. 90 days
- B. 1 day
- C. 7 days
- D. 45 days

<details><summary>Show Answer</summary>
Correct Answer: B. Temporary tables (like transient tables) support a maximum Time Travel retention of 1 day, and they exist only for the session in which they were created.
</details>

---

### Question 209
When should a multi-cluster warehouse be used in auto-scale mode?

- A. When it is unknown how much compute power is needed
- B. If the SELECT statement contains a large number of CTEs
- C. If the runtime of the executed query is very slow
- D. When a large number of concurrent queries are run against the same warehouse

<details><summary>Show Answer</summary>
Correct Answer: D. Auto-scaling multi-cluster warehouses exist to absorb concurrency (many simultaneous queries/users), not to speed up any single slow query.
</details>

---

### Question 210
What happens when a cloned table is replicated to a secondary database? (Choose two.)

- A. A read-only copy of the cloned table is stored.
- B. The replication will not be successful.
- C. The physical data is replicated.
- D. Additional storage costs are charged to the secondary account.
- E. Metadata pointers to the cloned table are replicated.

<details><summary>Show Answer</summary>
Correct Answer: C, D. A clone is normally "zero-copy" (it just points to the source object's micro-partitions). But replication crosses account boundaries, so the secondary account can't share the primary account's storage — the clone's physical data must actually be copied, and that consumes (and is billed as) additional storage in the secondary account.
</details>

---

### Question 211
Snowflake supports the use of external stages with which cloud platforms? (Choose three.)

- A. Amazon Web Services
- B. Docker
- C. IBM Cloud
- D. Microsoft Azure
- E. Google Cloud Platform
- F. Oracle Cloud

<details><summary>Show Answer</summary>
Correct Answer: A, D, E. External stages can point to buckets/containers in AWS S3, Azure Blob Storage, or Google Cloud Storage.
</details>

---

### Question 212
What is a limitation of a materialized view?

- A. A materialized view cannot support any aggregate functions.
- B. A materialized view can only reference up to two tables.
- C. A materialized view cannot be joined with other tables.
- D. A materialized view cannot be defined with a JOIN.

<details><summary>Show Answer</summary>
Correct Answer: D. A materialized view's defining query cannot include a JOIN (among other restrictions, such as no window functions, `HAVING`, `ORDER BY`, or references to other views).
</details>

---

### Question 213
In the Snowflake access control model, which entity owns an object by default?

- A. The user who created the object
- B. The SYSADMIN role
- C. Ownership depends on the type of object
- D. The role that was used to create the object

<details><summary>Show Answer</summary>
Correct Answer: D. Snowflake follows Discretionary Access Control (DAC): the role active in the session when an object is created becomes its owner, not the individual user.
</details>

---

### Question 214
What is the minimum Snowflake edition required to use Dynamic Data Masking?

- A. Standard
- B. Enterprise
- C. Business Critical
- D. Virtual Private Snowflake (VPS)

<details><summary>Show Answer</summary>
Correct Answer: B. Column-level security (masking policies / Dynamic Data Masking) requires Enterprise Edition or higher.
</details>

---

### Question 215
Which services does the Snowflake Cloud Services layer manage? (Choose two.)

- A. Compute resources
- B. Query execution
- C. Authentication
- D. Data storage
- E. Metadata

<details><summary>Show Answer</summary>
Correct Answer: C, E. The Cloud Services layer handles authentication, infrastructure management, metadata, query parsing/optimization, and access control — not the actual query execution (Compute layer) or physical data storage (Storage layer).
</details>

---

### Question 216
A company needs to allow some users to see Personally Identifiable Information (PII) while limiting other users from seeing its full value. Which Snowflake feature supports this?

- A. Row access policies
- B. Data masking policies
- C. Encryption
- D. Role-based access control

<details><summary>Show Answer</summary>
Correct Answer: B. Masking policies conditionally obscure column values (e.g., showing only the last 4 digits) based on the querying role, while leaving the underlying data intact for authorized roles.
</details>

---

### Question 217
A user unloaded data from a Snowflake table to an external stage. Which command can be used to verify the data was uploaded to the external stage named `my_stage`?

- A. `VIEW @my_stage`
- B. `LIST @my_stage`
- C. `SHOW @my_stage`
- D. `DISPLAY @my_stage`

<details><summary>Show Answer</summary>
Correct Answer: B. `LIST @my_stage` (or its alias `LS`) returns the files present in a stage.
</details>

---

### Question 218
Which tasks are performed by the Snowflake Cloud Services layer? (Choose two.)

- A. Management of metadata
- B. Computing/processing data
- C. Maintaining availability zones
- D. Infrastructure security
- E. Parsing and optimizing queries

<details><summary>Show Answer</summary>
Correct Answer: A, E. Metadata management and query parsing/optimization happen in Cloud Services. Actual data computation happens in the Compute layer; availability zones and infrastructure security are underlying cloud-provider concerns.
</details>

---

### Question 219
What is true about sharing data in Snowflake? (Choose two.)

- A. The provider pays for both data storage and the compute used to query shared data.
- B. Shared data is copied into the consumer account, so the consumer can modify it without impacting the provider's data.
- C. A Snowflake account can both provide and consume shared data.
- D. The provider is charged for compute resources used by the consumer to query the shared data.
- E. The consumer pays only for the compute resources used to query the shared data.

<details><summary>Show Answer</summary>
Correct Answer: C, E. Secure Data Sharing is read-only and live (no data copying) — the provider pays for storage, and each consumer pays only for the compute it uses to query the shared objects. Any Snowflake account can act as both a provider and a consumer.
</details>

---

### Question 220
The following JSON is stored in a VARIANT column called `src` in the `CAR_SALES` table:

```json
{
  "customer": [
    {
      "address": "San Francisco, CA",
      "name": "Jane Doe"
    }
  ],
  "date": "2022-01-28",
  "dealership": "Town Auto Sales"
}
```

How can a user extract the dealership information from the JSON?

- A. `select src:dealership from car_sales;`
- B. `select src.dealership from car_sales;`
- C. `select * from car_sales;`
- D. `select dealership from car_sales;`

<details><summary>Show Answer</summary>
Correct Answer: A. The colon (`:`) operator accesses a top-level element of a VARIANT column by key name.
</details>

---

### Question 221
Which of the following significantly improves the performance of selective point-lookup queries on a table?

- A. Clustering
- B. Materialized views
- C. Zero-copy cloning
- D. Search Optimization Service

<details><summary>Show Answer</summary>
Correct Answer: D. The Search Optimization Service builds a persisted search access path specifically to speed up highly selective point-lookup and substring-search queries.
</details>

---

### Question 222
Which of the following accurately describes shares?

- A. Tables, secure views, and secure UDFs can be shared.
- B. Shares themselves can be shared onward by the consumer.
- C. A new table can be cloned directly from a share.
- D. Access to a share cannot be revoked once granted.

<details><summary>Show Answer</summary>
Correct Answer: A. A provider can include tables, secure views, and secure UDFs in a share. Shares cannot be re-shared by consumers, cloning from a shared object isn't supported the way it is for objects you own, and providers can revoke access at any time.
</details>

---

### Question 223
What are best-practice recommendations for using the ACCOUNTADMIN system role? (Choose two.)

- A. Ensure all users with the ACCOUNTADMIN role use Multi-Factor Authentication (MFA).
- B. All users granted ACCOUNTADMIN must be owned by the ACCOUNTADMIN role.
- C. The ACCOUNTADMIN role must be granted to only one user.
- D. Assign the ACCOUNTADMIN role to at least two users, but as few as possible.
- E. All users granted ACCOUNTADMIN must also be granted SECURITYADMIN.

<details><summary>Show Answer</summary>
Correct Answer: A, D. Snowflake recommends enforcing MFA for ACCOUNTADMIN users and assigning the role to at least two people (for redundancy/business continuity) while keeping the group as small as possible.
</details>

---

### Question 224
In the Query Profile view for a query, which components represent areas that can help optimize query performance? (Choose two.)

- A. Bytes scanned
- B. Bytes sent over the network
- C. Number of partitions scanned
- D. Percentage scanned from cache
- E. External bytes scanned

<details><summary>Show Answer</summary>
Correct Answer: A, C. Bytes scanned and partitions scanned (especially relative to total partitions) are the key indicators of how much pruning is happening and where a query is spending its time.
</details>

---

### Question 225
What is the minimum Snowflake edition required for row-level security?

- A. Standard
- B. Enterprise
- C. Business Critical
- D. Virtual Private Snowflake

<details><summary>Show Answer</summary>
Correct Answer: B. Row access policies (row-level security) require Enterprise Edition or higher.
</details>

---

### Question 226
What is the minimum Fail-safe retention time period for transient tables?

- A. 1 day
- B. 7 days
- C. 12 hours
- D. 0 days

<details><summary>Show Answer</summary>
Correct Answer: D. Transient tables (and temporary tables) have no Fail-safe period at all — 0 days.
</details>

---

### Question 227
What is a machine learning and data science partner within the Snowflake Partner Ecosystem?

- A. Informatica
- B. Power BI
- C. Adobe
- D. DataRobot

<details><summary>Show Answer</summary>
Correct Answer: D. DataRobot is listed among Snowflake's machine learning / data science ecosystem partners; the others fall under data integration (Informatica) or BI (Power BI).
</details>

---

### Question 228
Which statements are correct concerning the use of third-party data from the Snowflake Marketplace? (Choose two.)

- A. Data is live, ready-to-query, and can be personalized.
- B. Data needs to be loaded into a cloud-provider account as a consumer.
- C. Data is available for copying/moving into an individual Snowflake account.
- D. Data is available without copying or moving it.
- E. Data transformations are always required when combining Marketplace datasets with existing data.

<details><summary>Show Answer</summary>
Correct Answer: A, D. Marketplace data listings provide live, ready-to-query access without any data movement or ETL required to make the data queryable.
</details>

---

### Question 229
What impacts the credit consumption of maintaining a materialized view? (Choose two.)

- A. Whether it is also a secure view
- B. How often the underlying base table is queried
- C. How often the base table changes
- D. Whether the materialized view has a clustering key defined
- E. How often the materialized view itself is queried

<details><summary>Show Answer</summary>
Correct Answer: C, D. Snowflake automatically refreshes a materialized view in the background whenever the base table changes — more frequent changes mean more background maintenance credits. If the materialized view also has a clustering key, ongoing automatic reclustering adds further maintenance cost. Querying the MV itself is billed as regular compute, not "maintenance."
</details>

---

### Question 230
What `COPY INTO <location>` setting should be used to unload data into multiple files?

- A. `SINGLE = TRUE`
- B. `MULTIPLE = TRUE`
- C. `MULTIPLE = FALSE`
- D. `SINGLE = FALSE`

<details><summary>Show Answer</summary>
⚠ Updated: The real `COPY INTO <location>` syntax only has a `SINGLE` parameter (there is no `MULTIPLE` parameter in Snowflake SQL). `SINGLE = FALSE` is the default and produces multiple output files. Correct Answer: D.
</details>

---

### Question 231
When cloning a database containing stored procedures and regular views that have fully qualified table references, what happens?

- A. The cloned views and stored procedures will reference the cloned tables in the new database.
- B. An error will occur, as views with qualified references cannot be cloned.
- C. An error will occur, as stored objects cannot be cloned.
- D. The stored procedures and views will continue to refer to tables in the original database.

<details><summary>Show Answer</summary>
Correct Answer: D. Views and stored procedures store fully qualified references at creation time; cloning the database does not rewrite those references, so they keep pointing back to the original (source) objects.
</details>

---

### Question 232
When loading data into Snowflake, how should the data be organized for best performance?

- A. Into files with roughly 100–250 MB of compressed data per file
- B. Into files with roughly 1–100 MB of compressed data per file
- C. Into files with a maximum size of 1 GB of compressed data per file
- D. Into files with a maximum size of 4 GB of data per file

<details><summary>Show Answer</summary>
Correct Answer: A. Snowflake recommends splitting data into compressed files of roughly 100–250 MB (or larger) to maximize parallelism during loading.
</details>

---

### Question 233
Which of the following objects can be directly restored using the `UNDROP` command? (Choose two.)

- A. Schema
- B. View
- C. Internal Stage
- D. Table
- E. User
- F. Role

<details><summary>Show Answer</summary>
⚠ Updated: Correct Answer (from the given options): A, D. `UNDROP` has always supported Table, Schema, and Database. Note that Snowflake has since expanded `UNDROP` support to many more object types (Dynamic Tables, Iceberg Tables, Notebooks, Streamlit apps, Tags, external volumes, and even accounts) — but among the six options listed here, only Schema and Table are (and were) valid. Views, internal stages, users, and roles are not restorable with `UNDROP`.
</details>

---

### Question 234
Which Snowflake SQL statement would be used to determine which users and roles have access to a role called `MY_ROLE`?

- A. `SHOW GRANTS OF ROLE MY_ROLE`
- B. `SHOW GRANTS TO ROLE MY_ROLE`
- C. `SHOW GRANTS FOR ROLE MY_ROLE`
- D. `SHOW GRANTS ON ROLE MY_ROLE`

<details><summary>Show Answer</summary>
Correct Answer: A. `SHOW GRANTS OF ROLE <name>` lists the users and roles to which the role has been granted. (`SHOW GRANTS TO ROLE <name>` instead lists the privileges the role itself holds.)
</details>

---

### Question 235
What is the minimum edition of Snowflake required to use a SCIM security integration?

- A. Business Critical Edition
- B. Standard Edition
- C. Virtual Private Snowflake (VPS)
- D. Enterprise Edition

<details><summary>Show Answer</summary>
Correct Answer: B. SCIM 2.0 provisioning is available across all Snowflake editions, including Standard.
</details>

---

### Question 236
A user created a transient table and made several changes to it over the course of several days. Three days after the table was created, the user wants to go back to the first version of the table. How can this be accomplished?

- A. Use Time Travel as long as `DATA_RETENTION_TIME_IN_DAYS` is set to at least 3 days.
- B. It cannot be done — transient tables have a maximum Time Travel retention of only 1 day and no Fail-safe period.
- C. Contact Snowflake Support to have the data retrieved from Fail-safe storage.
- D. Use the `FAILSAFE` parameter with Time Travel to retrieve the data from Fail-safe storage.

<details><summary>Show Answer</summary>
Correct Answer: B. Transient tables cap out at 1 day of Time Travel retention (it cannot be set to 3 days), and they have zero days of Fail-safe, so the original version is unrecoverable after that window closes.
</details>

---

### Question 237
When reviewing warehouse load, the load-monitoring chart shows a high volume of queries constantly queuing. According to best practice, what should be done to reduce the queue? (Choose two.)

- A. Use multi-cluster warehousing to scale out warehouse capacity.
- B. Scale up the warehouse size so queries execute faster.
- C. Stop and start the warehouse to clear the queued queries.
- D. Migrate some queries to a new warehouse to reduce load.
- E. Restrict users from accessing the warehouse so fewer queries run against it.

<details><summary>Show Answer</summary>
Correct Answer: A, D. Queuing is a concurrency problem, best solved by scaling out (multi-cluster) or spreading the workload across multiple warehouses — not by scaling up (which helps individual query speed, not queuing) or restarting the warehouse (which doesn't add capacity).
</details>

---

### Question 238
Which of the following features, associated with Continuous Data Protection (CDP), require additional Snowflake-provided data storage? (Choose two.)

- A. Tri-Secret Secure
- B. Time Travel
- C. Fail-safe
- D. Data encryption
- E. External stages

<details><summary>Show Answer</summary>
Correct Answer: B, C. Time Travel and Fail-safe both retain historical versions of data, which consumes additional storage that is billed to the account.
</details>

---

### Question 239
Where can a user find and review the failed logins of a specific user for the past 30 days?

- A. The `USERS` view in `ACCOUNT_USAGE`
- B. The `LOGIN_HISTORY` view in `ACCOUNT_USAGE`
- C. The `ACCESS_HISTORY` view in `ACCOUNT_USAGE`
- D. The `SESSIONS` view in `ACCOUNT_USAGE`

<details><summary>Show Answer</summary>
Correct Answer: B. `SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY` records login attempts, including failures, and (unlike `INFORMATION_SCHEMA.LOGIN_HISTORY`) retains up to 365 days of history.
</details>

---

### Question 240
What is the purpose of an External Function?

- A. To call code that executes outside of Snowflake
- B. To run a function in another Snowflake database
- C. To share data in Snowflake with external parties
- D. To ingest data from on-premises data sources

<details><summary>Show Answer</summary>
Correct Answer: A. External functions let SQL code call out to a remote service (e.g., an AWS Lambda or Azure Function) hosted outside of Snowflake.
</details>

---

### Question 241
Which of the following statements apply to Snowflake in terms of security? (Choose two.)

- A. Snowflake leverages a Role-Based Access Control (RBAC) model.
- B. Snowflake requires a user to configure an IAM user to connect to the database.
- C. All data in Snowflake is encrypted.
- D. Snowflake can run entirely within a user's own Virtual Private Cloud (VPC).
- E. All data in Snowflake is compressed.

<details><summary>Show Answer</summary>
Correct Answer: A, C. Snowflake's access model combines RBAC and DAC, and all data is automatically encrypted at rest and in transit by default, regardless of edition.
</details>

---

### Question 242
A single user of a virtual warehouse has set it to auto-resume and auto-suspend after 10 minutes. The warehouse is currently suspended, and the user performs the following:
1. Runs a query that takes 3 minutes to complete.
2. Leaves for 15 minutes.
3. Returns and runs a query that takes 10 seconds to complete.
4. Manually suspends the warehouse as soon as the last query completes.

How much billable compute time will have been consumed?

- A. 4 minutes
- B. 13 minutes
- C. 14 minutes
- D. 24 minutes

<details><summary>Show Answer</summary>
Correct Answer: C. 3 minutes (query 1) + 10 minutes idle before auto-suspend kicks in (the 15-minute absence exceeds the 10-minute timeout) + 1 minute minimum billing for the second query (Snowflake bills with a 60-second minimum) = 14 minutes.
</details>

---

### Question 243
What can be used to view warehouse usage time? (Choose two.)

- A. The `LOAD_HISTORY` view
- B. The Query History view
- C. The `SHOW WAREHOUSES` command
- D. The `WAREHOUSE_METERING_HISTORY` view in `ACCOUNT_USAGE`
- E. The Billing & Usage tab in the Snowflake web UI

<details><summary>Show Answer</summary>
Correct Answer: D, E. `ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY` and the Billing & Usage area in Snowsight both report warehouse credit/usage time.
</details>

---

### Question 244
What actions will prevent leveraging of the result set (query results) cache? (Choose two.)

- A. Removing a column from the query's SELECT list
- B. Stopping the virtual warehouse the query is running against
- C. Clustering the data used by the query
- D. Executing the `RESULT_SCAN` table function
- E. The underlying data used by the query has changed

<details><summary>Show Answer</summary>
Correct Answer: A, E. Any change to the query text (like removing a column) or to the underlying table data invalidates the cached result. Stopping the warehouse doesn't affect the cache (it's independent of any warehouse), and `RESULT_SCAN` reads the cache rather than breaking it.
</details>

---

### Question 245
Which statement is true about running tasks in Snowflake?

- A. A task can be called using a `CALL` statement to run a set of predefined SQL commands.
- B. A task allows a user to execute a single SQL statement or stored procedure call on a predefined schedule.
- C. A task allows a user to execute a set of SQL commands on a predefined schedule.
- D. A task can be executed using a `SELECT` statement to run a predefined SQL command.

<details><summary>Show Answer</summary>
Correct Answer: B. Each individual task executes one SQL statement (which can itself be a call to a stored procedure containing multiple statements) on a schedule; chaining multiple tasks together forms a task graph (DAG) for more complex workflows.
</details>

---

### Question 246
Which data types does Snowflake support when querying semi-structured data? (Choose two.)

- A. VARIANT
- B. VARCHAR
- C. XML
- D. ARRAY
- E. BLOB

<details><summary>Show Answer</summary>
Correct Answer: A, D. VARIANT stores arbitrary semi-structured data, and ARRAY/OBJECT are the structured sub-types used to navigate it. VARCHAR and BLOB are not semi-structured types, and XML is a file format, not a Snowflake column data type.
</details>

---

### Question 247
In an auto-scaling multi-cluster virtual warehouse with `SCALING_POLICY = ECONOMY`, when is an additional cluster started?

- A. When the system has enough load for 2 minutes
- B. When the system has enough load for 6 minutes
- C. When the system has enough load for 8 minutes
- D. When the system has enough load for 10 minutes

<details><summary>Show Answer</summary>
Correct Answer: B. The ECONOMY scaling policy favors keeping existing clusters fully loaded and only starts a new cluster if the queued load is expected to keep it busy for at least 6 minutes (this conserves credits compared to the STANDARD policy).
</details>

---

### Question 248
What is the following SQL command used for?

```sql
SELECT * FROM TABLE(VALIDATE(t1, JOB_ID => '_last'));
```

- A. To validate external table files against table t1 across all sessions
- B. To validate task SQL statements against table t1 over the last 14 days
- C. To validate a file for errors before it gets loaded via a `COPY` command
- D. To return errors from the last executed `COPY` command into table t1, within the current session

<details><summary>Show Answer</summary>
Correct Answer: D. `VALIDATE(table_name, JOB_ID => '_last')` returns the load errors from the most recent `COPY INTO` load for that table, scoped to the current session.
</details>

---

### Question 249
A table `FCT_SALES` has 100 million rows. The following query is executed:

```sql
SELECT COUNT(*) FROM FCT_SALES;
```

How did Snowflake fulfill this query?

- A. Query against the result set cache
- B. Query against a virtual warehouse's local disk cache
- C. Query against the most-recently created micro-partition
- D. Query against table metadata (no data scan required)

<details><summary>Show Answer</summary>
Correct Answer: D. Snowflake stores row counts as metadata for each micro-partition, so a simple `COUNT(*)` (with no filters) can be answered entirely from metadata, without scanning any actual data or even requiring a running warehouse.
</details>

---

### Question 250
What happens when a virtual warehouse is resized?

- A. When increasing the size of an active warehouse, all running and queued queries are affected immediately.
- B. When reducing the size of a warehouse, compute resources are removed only once they are no longer being used by any currently executing statement.
- C. The warehouse is suspended while the new compute resources are provisioned, then automatically resumes once provisioning completes.
- D. Users trying to use the warehouse will receive an error message until resizing completes.

<details><summary>Show Answer</summary>
Correct Answer: B. Scaling up adds resources for new queries without disrupting queries already running; scaling down removes resources gracefully, only after in-flight statements finish using them.
</details>

---

### Question 251
What tasks can be completed using the `COPY INTO <table>` command? (Choose two.)

- A. Columns can be renamed.
- B. Columns can be joined with an existing table.
- C. Columns can be reordered.
- D. Columns can be omitted.
- E. Data can be loaded without spinning up a virtual warehouse.

<details><summary>Show Answer</summary>
Correct Answer: C, D. Using a column list and `SELECT` transformation in the `COPY INTO <table>` statement, you can reorder or omit source columns during the load. A running warehouse (or Snowpipe's serverless compute) is still required to execute the load.
</details>

---

### Question 252
Which Snowflake layer can be directly configured by users?

- A. Database Storage
- B. Cloud Services
- C. Compute (Query Processing)
- D. Application Services

<details><summary>Show Answer</summary>
Correct Answer: C. Users create, resize, and manage virtual warehouses in the Compute layer. Storage and Cloud Services scale automatically and are not directly configured by customers.
</details>

---

### Question 253
Query compilation occurs in which layer of Snowflake's architecture?

- A. Compute layer
- B. Storage layer
- C. Cloud infrastructure layer
- D. Cloud Services layer

<details><summary>Show Answer</summary>
Correct Answer: D. Query parsing, compilation, and optimization all happen in the Cloud Services layer, before any work is dispatched to a virtual warehouse for execution.
</details>

---

### Question 254
If an X-Small virtual warehouse is made up of one server and a Small warehouse is made up of two servers, how many servers make up a Large warehouse?

- A. 4
- B. 8
- C. 16
- D. 32

<details><summary>Show Answer</summary>
Correct Answer: B. Each warehouse size doubles the compute of the previous size: X-Small=1, Small=2, Medium=4, Large=8, X-Large=16, and so on.
</details>

---

### Question 255
A clustering key was defined on a table but is no longer needed. How can the key be removed?

- A. `ALTER TABLE [table_name] PURGE CLUSTERING KEY`
- B. `ALTER TABLE [table_name] DELETE CLUSTERING`
- C. `ALTER TABLE [table_name] DROP CLUSTERING KEY`
- D. `ALTER TABLE [table_name] REMOVE CLUSTERING KEY`

<details><summary>Show Answer</summary>
Correct Answer: C. `ALTER TABLE ... DROP CLUSTERING KEY` is the correct syntax to remove a defined clustering key (the table itself is unaffected).
</details>

---

### Question 256
What is the purpose of clustering?

- A. To guarantee uniquely identifiable records in the database
- B. To increase scan efficiency in queries by improving partition pruning
- C. To improve performance by creating a separate file for point lookups
- D. To provide data redundancy by duplicating micro-partitions

<details><summary>Show Answer</summary>
Correct Answer: B. Clustering co-locates similar column values within micro-partitions so that queries filtering on the clustering key can prune (skip) irrelevant partitions.
</details>

---

### Question 257
Which statement is true about Multi-Factor Authentication (MFA) in Snowflake?

- A. MFA can be enforced for a given role.
- B. Snowflake users are automatically enrolled in MFA.
- C. Users enroll in MFA by submitting a request to Snowflake Support.
- D. MFA is a natively integrated Snowflake feature.

<details><summary>Show Answer</summary>
Correct Answer: D. Snowflake has native MFA (powered by Duo Security) built directly into the platform. Enrollment is self-service through the user's profile, not automatic and not something Support has to configure.
</details>

---

### Question 258
What data type should be used to store JSON data natively in Snowflake?

- A. JSON
- B. STRING
- C. OBJECT
- D. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: D. VARIANT natively stores semi-structured data such as JSON, Avro, ORC, Parquet, or XML, preserving its structure for querying.
</details>

---

### Question 259
What should be considered when deciding to use a Secure View? (Choose two.)

- A. Details of the query execution plan are hidden from non-owners in the Query Profile.
- B. Once created, there is no way to determine whether a view is secure or not.
- C. Secure views do not take advantage of the same internal optimizations as standard views.
- D. It is not possible to create a secure materialized view.
- E. The view definition of a secure view is always visible to all users via the Information Schema.

<details><summary>Show Answer</summary>
Correct Answer: A, C. Secure views intentionally hide internal query details (like the underlying SQL text and parts of the execution plan) from unauthorized users to protect the view logic, and as a trade-off they skip some query-optimization techniques (like predicate pushdown) that could otherwise leak information about the underlying data.
</details>

---

### Question 260
The Information Schema provides storage information for which of the following objects? (Choose two.)

- A. Users
- B. Databases
- C. Internal Stages
- D. Resource Monitors
- E. Pipes

<details><summary>Show Answer</summary>
Correct Answer: B, C (as sourced from the original question bank). Note: detailed storage metrics for tables/stages are more comprehensively available via `SNOWFLAKE.ACCOUNT_USAGE`, so treat this specific pairing with some caution if you encounter it on an actual exam.
</details>

---

### Question 261
What is a responsibility of Snowflake's virtual warehouses?

- A. Infrastructure management
- B. Metadata management
- C. Query execution
- D. Query parsing and optimization
- E. Management of storage

<details><summary>Show Answer</summary>
Correct Answer: C. Virtual warehouses (the Compute layer) are responsible for executing queries and DML operations. Parsing/optimization and metadata management happen in Cloud Services; storage is managed independently.
</details>

---

### Question 262
Which data type is supported by Snowflake's native data classification feature?

- A. FLOAT
- B. STRING
- C. GEOGRAPHY
- D. VARIANT

<details><summary>Show Answer</summary>
Correct Answer: B. Snowflake's automated data classification analyzes STRING-type column data to detect and tag categories like PII (names, emails, etc.).
</details>

---

### Question 263
When unloading data to an external stage, which compression format can be used for Parquet files with the `COPY INTO` command?

- A. BROTLI
- B. GZIP
- C. LZO
- D. ZSTD

<details><summary>Show Answer</summary>
Correct Answer: C. For `TYPE = PARQUET`, the supported `COMPRESSION` values are `AUTO`, `LZO`, `SNAPPY` (the default), or `NONE`. BROTLI, GZIP, and ZSTD are valid for other file types (CSV, JSON, Avro) but not for Parquet.
</details>

---

### Question 264
Which SQL command can be used to verify the privileges that are granted to a role?

- A. `SHOW GRANTS ON ROLE [role_name]`
- B. `SHOW ROLES [role_name]`
- C. `SHOW GRANTS TO ROLE [role_name]`
- D. `GRANTS FOR ROLE [role_name]`

<details><summary>Show Answer</summary>
Correct Answer: C. `SHOW GRANTS TO ROLE <name>` lists the privileges that have been granted to a role.
</details>

---

### Question 265
Which Query Profile result indicates that a warehouse is sized too small for a query?

- A. There are a lot of filter nodes.
- B. Bytes are spilling to local or remote storage.
- C. The percentage of partitions scanned is very high.
- D. The number of partitions scanned equals the total number of partitions.

<details><summary>Show Answer</summary>
Correct Answer: B. When a warehouse doesn't have enough memory for an operation (e.g., a large sort or join), Snowflake spills intermediate results to local disk and then to remote storage — a clear sign the warehouse should be resized larger.
</details>

---

### Question 266
What is the default Time Travel retention period?

- A. 1 day
- B. 7 days
- C. 45 days
- D. 90 days

<details><summary>Show Answer</summary>
Correct Answer: A. The account default for `DATA_RETENTION_TIME_IN_DAYS` is 1 day; Enterprise Edition (and higher) can extend this up to 90 days for permanent objects if explicitly configured.
</details>

---

### Question 267
Which of the following are best-practice recommendations to consider when loading data into Snowflake? (Choose two.)

- A. Load files that are approximately 25 MB or smaller.
- B. Remove all dates and timestamps.
- C. Load files that are approximately 100–250 MB (or larger) compressed.
- D. Avoid using embedded characters, such as commas, for numeric data types.
- E. Remove all semi-structured data types.

<details><summary>Show Answer</summary>
Correct Answer: C, D. Aim for 100–250 MB compressed files for load parallelism, and avoid embedding delimiters like commas inside numeric fields (which breaks parsing).
</details>

---

### Question 268
Which schema contains the `RESOURCE_MONITORS` view?

- A. `SNOWFLAKE.ACCOUNT_USAGE`
- B. `SNOWFLAKE.READER_ACCOUNT_USAGE`
- C. `INFORMATION_SCHEMA`
- D. `SNOWFLAKE.ORGANIZATION_USAGE`

<details><summary>Show Answer</summary>
Correct Answer: A. Resource monitors are account-level objects, and their metadata is surfaced through `SNOWFLAKE.ACCOUNT_USAGE.RESOURCE_MONITORS` — not a per-database `INFORMATION_SCHEMA`.
</details>

---

### Question 269
What is the purpose of enabling Federated Authentication on a Snowflake account?

- A. Disables the ability to use key-pair and basic username/password authentication when connecting.
- B. Allows dual Multi-Factor Authentication (MFA) when connecting to Snowflake.
- C. Forces users to connect through a secure network proxy.
- D. Allows users to connect using secure single sign-on (SSO) through an external identity provider.

<details><summary>Show Answer</summary>
Correct Answer: D. Federated authentication delegates login to an external SAML 2.0 identity provider (e.g., Okta, Azure AD), enabling SSO.
</details>

---

### Question 270
Which Snowflake Partner Ecosystem category is represented at the top of the (referenced) partner diagram?

- A. Business Intelligence
- B. Machine Learning and Data Science
- C. Security and Governance
- D. Data Integration

<details><summary>Show Answer</summary>
Correct Answer: D (as sourced from the original question bank). Note: the diagram referenced in the original source wasn't legible/available in the OCR text, so this answer could not be independently re-verified against an image — treat it with a bit more caution than the rest of this set.
</details>

---

### Question 271
Which object types are protected by Fail-safe? (Choose two.)

- A. Permanent tables
- B. Temporary tables
- C. External tables
- D. Materialized views
- E. Transient tables

<details><summary>Show Answer</summary>
Correct Answer: A, D. Fail-safe applies only to permanent objects (permanent tables and their materialized views). Temporary and transient tables have no Fail-safe, and external tables don't store data within Snowflake at all.
</details>

---

### Question 272
Snowflake's approach to the management of system access combines which of the following? (Choose two.)

- A. Security Assertion Markup Language (SAML)
- B. Role-Based Access Control (RBAC)
- C. Identity Access Management (IAM)
- D. Create, Read, Update, and Delete (CRUD)
- E. Discretionary Access Control (DAC)
- F. Mandatory Access Control (MAC)

<details><summary>Show Answer</summary>
Correct Answer: B, E. Snowflake's access control model combines Role-Based Access Control (privileges assigned to roles, which are assigned to users) with Discretionary Access Control (each object has an owning role that can grant access to it).
</details>

---

### Question 273
According to Snowflake best-practice recommendations, which role should be used to create databases?

- A. ACCOUNTADMIN
- B. SYSADMIN
- C. SECURITYADMIN
- D. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: B. SYSADMIN is the recommended role for creating and managing warehouses, databases, and other data objects, keeping ACCOUNTADMIN reserved for account-level administration.
</details>

---

### Question 274
To add or remove search optimization for a table, a user must have which of the following privileges? (Choose two.)

- A. The MODIFY privilege on the table
- B. The OWNERSHIP privilege on the table
- C. The SECURITYADMIN role
- D. The ADD SEARCH OPTIMIZATION privilege on the schema that contains the table
- E. The SELECT privilege on the table

<details><summary>Show Answer</summary>
Correct Answer: B, D. A role needs OWNERSHIP on the table itself, plus the ADD SEARCH OPTIMIZATION privilege (granted by default to the schema owner, or grantable to another role) on the containing schema.
</details>

---

### Question 275
While using a `COPY` command with a `VALIDATION_MODE` parameter, which of the following will return an error?

- A. Statements that insert a duplicate record during a load
- B. Statements that have a specific data type in the source
- C. Statements that have duplicate file names
- D. Statements that transform data during a load

<details><summary>Show Answer</summary>
Correct Answer: D. `VALIDATION_MODE` is not compatible with a `COPY INTO <table>` statement that also performs data transformations (i.e., uses a `SELECT` with column transformations) — this combination returns an error.
</details>

---

### Question 276
When is the result set (query results) cache no longer available? (Choose two.)

- A. When a different warehouse is used to execute the query
- B. When the user executes the `RESULT_SCAN` function
- C. When the underlying data used by the query has changed
- D. When the warehouse used to execute the query is suspended
- E. When it has been 24 hours since the query was last executed

<details><summary>Show Answer</summary>
Correct Answer: C, E. The results cache is invalidated once the underlying data changes, or after 24 hours have passed since the results were last used (each use resets that 24-hour clock, up to a maximum of 31 days from the original execution). It is independent of which warehouse runs the query or whether that warehouse is suspended.
</details>

---

### Question 277
What is the recommended file sizing for data loading using Snowpipe?

- A. A compressed file size greater than 100 MB, and up to 250 MB
- B. A compressed file size greater than 100 GB, and up to 250 GB
- C. A compressed file size greater than 10 MB, and up to 100 MB
- D. A compressed file size greater than 1 GB, and up to 2 GB

<details><summary>Show Answer</summary>
Correct Answer: A. As with bulk loading, Snowflake recommends compressed files of roughly 100–250 MB for Snowpipe to balance load latency and per-file overhead.
</details>

---

### Question 278
Which statements are true concerning Snowflake's underlying cloud infrastructure? (Choose three.)

- A. Snowflake data and services are deployed in a single availability zone within a cloud provider's region.
- B. Snowflake data and services are available only in a single cloud provider and region; use of multiple cloud providers is not supported.
- C. Snowflake can be deployed in a customer's private cloud, using the customer's own compute and storage resources.
- D. Snowflake uses the core compute and storage services of each cloud provider it runs on.
- E. All three layers (storage, compute, and cloud services) are deployed and managed entirely on the selected cloud platform.
- F. Snowflake data and services are deployed across at least three availability zones within a cloud provider's region.

<details><summary>Show Answer</summary>
Correct Answer: D, E, F. Snowflake runs entirely on top of the native compute/storage services of AWS, Azure, or GCP, spans at least three availability zones for resilience, and does not deploy into a customer's own private infrastructure.
</details>

---

### Question 279
A user unloaded a Snowflake table called `mytable` to an internal stage called `mystage`. Which command can be used to view the list of files uploaded to the stage?

- A. `LIST @mytable;`
- B. `LIST TABLE mystage;`
- C. `SHOW STAGE mystage;`
- D. `LIST @mystage;`

<details><summary>Show Answer</summary>
Correct Answer: D. `LIST @mystage;` lists the files present in the named internal stage `mystage`.
</details>

---

### Question 280
What is a best practice after creating a custom role?

- A. Create the custom role using the SYSADMIN role.
- B. Assign the custom role to the SYSADMIN role.
- C. Assign the custom role to the PUBLIC role.
- D. Add `_CUSTOM` to all custom role names.

<details><summary>Show Answer</summary>
Correct Answer: B. Best practice is to grant custom roles up to SYSADMIN in the role hierarchy so system administrators retain full visibility and control over all custom objects.
</details>

---

### Question 281
Which is the minimum required Snowflake edition to use AWS/Azure PrivateLink or Google Cloud Private Service Connect?

- A. Standard
- B. Premium
- C. Enterprise
- D. Business Critical

<details><summary>Show Answer</summary>
Correct Answer: D. Private connectivity (AWS PrivateLink, Azure Private Link, Google Cloud Private Service Connect) requires Business Critical Edition or higher.
</details>

---

### Question 282
Which of the following Query Profile indicators shows that a virtual warehouse is not sized correctly for the query being executed?

- A. Bytes sent over the network
- B. Synchronization time
- C. Initialization time
- D. Remote spillage (bytes spilled to remote storage)

<details><summary>Show Answer</summary>
Correct Answer: D. Spilling to remote storage means the warehouse ran out of local memory/disk for the operation — a strong signal the warehouse should be resized up.
</details>

---

### Question 283
Which of the following Snowflake capabilities are available in all Snowflake editions? (Choose two.)

- A. Encryption key management through Tri-Secret Secure
- B. Automatic encryption of all data
- C. Up to 90 days of data recovery through Time Travel
- D. Object-level access control
- E. Column-level security to apply masking policies to tables and views

<details><summary>Show Answer</summary>
Correct Answer: B, D. Always-on encryption of data at rest/in transit and RBAC-based object-level access control are baseline features of every edition, including Standard. Tri-Secret Secure requires Business Critical, extended Time Travel (beyond 1 day) requires Enterprise+, and masking policies require Enterprise+.
</details>

---

### Question 284
A `PUT` command can be used to stage local files from which Snowflake interface?

- A. SnowSQL (CLI)
- B. Snowflake Classic Console (UI)
- C. Snowsight
- D. .NET driver

<details><summary>Show Answer</summary>
Correct Answer: A. `PUT` requires access to the local file system, so it is executed from a CLI or driver context like SnowSQL — not from the browser-based Snowsight or Classic Console UI, which cannot access your local disk directly.
</details>

---

### Question 285
Which of the following indicate that it may be appropriate to define a clustering key for a table? (Choose two.)

- A. The table contains a column that has very low cardinality.
- B. DML statements being issued against the table are blocked.
- C. The table has a small number of micro-partitions.
- D. Queries on the table are running slower than expected.
- E. The clustering depth for the table is large.

<details><summary>Show Answer</summary>
Correct Answer: D, E. Slower-than-expected query performance combined with a large clustering depth (poor data organization relative to the clustering/filter columns) are the classic signals that clustering would help. Very low cardinality, few partitions, or DML blocking are not indicators for clustering.
</details>

---

### Question 286
Which cache type is used to cache the data output from SQL queries?

- A. Metadata cache
- B. Result (query results) cache
- C. Remote cache
- D. Local disk (warehouse) cache

<details><summary>Show Answer</summary>
Correct Answer: B. The result cache stores the actual output of a previously run query for near-instant retrieval on an identical repeat query, for up to 24 hours (extendable to 31 days with reuse).
</details>

---

### Question 287
Which of the following describes how clustering keys work in Snowflake?

- A. Clustering keys update micro-partitions in place with a full sort, and block DML operations.
- B. Clustering keys sort the designated columns over time, without blocking DML operations.
- C. Clustering keys create a distributed, parallel data structure of pointers to rows and columns.
- D. Clustering keys establish a hashed key on each node of a virtual warehouse to optimize joins at run-time.

<details><summary>Show Answer</summary>
Correct Answer: B. Automatic reclustering incrementally re-sorts micro-partitions in the background over time, and never blocks concurrent DML on the table.
</details>

---

### Question 288
Which of the following operations require the use of a running virtual warehouse? (Choose two.)

- A. Downloading data from an internal stage
- B. Listing files in a stage
- C. Executing a stored procedure
- D. Altering a table's metadata (DDL)
- E. Querying data from a materialized view

<details><summary>Show Answer</summary>
Correct Answer: C, E. Executing SQL inside a stored procedure and querying a materialized view both require compute. Listing/downloading stage files and most metadata-only DDL operations are handled by Cloud Services and don't need an active warehouse.
</details>

---

### Question 289
What is used to limit the credit usage of a virtual warehouse within a Snowflake account?

- A. Load monitor
- B. Resource monitor
- C. Query profile
- D. Warehouse policy

<details><summary>Show Answer</summary>
Correct Answer: B. Resource monitors track credit usage against defined thresholds and can trigger notifications or automatically suspend warehouses when limits are reached.
</details>

---

### Question 290
What are the benefits of the replication feature in Snowflake? (Choose two.)

- A. Disaster recovery
- B. Time Travel
- C. Fail-safe
- D. Database failover and failback
- E. Data security

<details><summary>Show Answer</summary>
Correct Answer: A, D. Cross-region/cross-cloud replication supports disaster recovery scenarios and enables account failover/failback to a secondary account (Business Critical Edition or higher for failover/failback specifically).
</details>

---

### Question 291
Which of the following roles are recommended to create and manage other users and roles? (Choose two.)

- A. SYSADMIN
- B. SECURITYADMIN
- C. PUBLIC
- D. ACCOUNTADMIN
- E. USERADMIN

<details><summary>Show Answer</summary>
Correct Answer: B, E. SECURITYADMIN manages grants globally and can create/manage users and roles; USERADMIN is specifically dedicated to creating and managing users and roles.
</details>

---

### Question 292
When can a newly configured virtual warehouse start running SQL queries?

- A. Immediately, while provisioning is still in progress
- B. Only during specific time slots defined by the ACCOUNTADMIN
- C. After warehouse provisioning has completed
- D. After warehouse replication has completed

<details><summary>Show Answer</summary>
Correct Answer: C. A virtual warehouse can accept and run queries as soon as its compute resources finish provisioning.
</details>

---

### Question 293
What action will prevent leveraging of the result set cache?

- A. Removing a column from the query's SELECT list
- B. Stopping the virtual warehouse that the query is running against
- C. The result not being reused within the last 12 hours
- D. Executing the `RESULT_SCAN` table function

<details><summary>Show Answer</summary>
Correct Answer: A. Changing the query text — including removing a column from the SELECT list — produces a different query signature, so it can't reuse a previous result. (Note: the real cache expiry window is 24 hours, not 12, so option C is a deliberately incorrect distractor; stopping the warehouse and using `RESULT_SCAN` don't invalidate the cache.)
</details>

---

### Question 294
Which of the following are benefits of micro-partitioning? (Choose two.)

- A. Micro-partitions cannot overlap in their range of values.
- B. Micro-partitions are immutable objects that support the use of Time Travel.
- C. Micro-partitions can reduce the amount of I/O from object storage to virtual warehouses.
- D. Rows are automatically stored in sorted order within every micro-partition.
- E. Micro-partitions can be defined on a schema-by-schema basis.

<details><summary>Show Answer</summary>
Correct Answer: B, C. Because micro-partitions are immutable, Snowflake can efficiently retain historical versions for Time Travel, and because each partition stores rich metadata, queries can prune irrelevant partitions and reduce I/O. (Partitions can overlap in value ranges, and micro-partitioning is automatic — not something configured per schema.)
</details>

---

### Question 295
Which data type can be used to store geospatial data in Snowflake?

- A. VARIANT
- B. OBJECT
- C. GEOMETRY
- D. GEOGRAPHY

<details><summary>Show Answer</summary>
⚠ Updated: Correct Answer: D (GEOGRAPHY) was the traditional/primary answer, and remains valid — GEOGRAPHY stores spherical (latitude/longitude, WGS 84) geospatial data. However, Snowflake has since also added a GEOMETRY data type for planar/Cartesian geospatial data, so C is now also a technically correct answer to "which data type can be used." If this is a single-select exam question, GEOGRAPHY (D) remains the expected answer.
</details>

---

### Question 296
If all virtual warehouse resources are maximized while processing a query workload, what happens to new queries submitted to the warehouse?

- A. All queries terminate once resources are maximized.
- B. The warehouse scales out automatically.
- C. The warehouse moves to a suspended state.
- D. New queries are queued and executed once capacity is available.

<details><summary>Show Answer</summary>
Correct Answer: D. New queries wait in a queue until sufficient compute capacity frees up (unless the warehouse is a multi-cluster warehouse configured to auto-scale, in which case additional clusters would start instead).
</details>

---

### Question 297
Masking policies can be applied to which of the following Snowflake objects? (Choose two.)

- A. A materialized view
- B. A stored procedure
- C. A table
- D. A stream
- E. A pipe
- F. A function

<details><summary>Show Answer</summary>
Correct Answer: A, C. Masking policies attach to columns on tables, views, and materialized views. They cannot be attached to procedural objects like stored procedures, streams, pipes, or functions.
</details>

---

### Question 298
What actions are supported by Snowflake resource monitors? (Choose two.)

- A. Alert (notify only)
- B. Notify
- C. Notify and suspend
- D. Abort
- E. Suspend immediately

<details><summary>Show Answer</summary>
Correct Answer: B, C. Snowflake resource monitors support three real trigger actions: **Notify**, **Notify & Suspend** (let running queries finish, block new ones), and **Notify & Suspend Immediately** (cancel all running queries too). "Abort" is not an actual resource monitor action.
</details>

---

### Question 299
A user executes the following SQL:

```sql
CREATE TABLE SALES_BKP LIKE SALES;
```

What are the cost implications of this statement?

- A. Processing costs will be generated based on how long the query takes.
- B. Storage costs will be generated based on the size of the data.
- C. No storage cost is incurred, since it relies on metadata only.
- D. The cost of running the virtual warehouse will be charged by the second.

<details><summary>Show Answer</summary>
Correct Answer: C. `CREATE TABLE ... LIKE` copies only the source table's structure (columns, not data), so no data is duplicated and effectively no additional storage cost is incurred.
</details>

---

### Question 300
What is the maximum Time Travel retention available in Snowflake Standard Edition?

- A. 1 day
- B. 7 days
- C. 30 days
- D. 90 days

<details><summary>Show Answer</summary>
Correct Answer: A. Standard Edition supports a maximum Time Travel retention of 1 day. Extending retention up to 90 days requires Enterprise Edition or higher.
</details>

---

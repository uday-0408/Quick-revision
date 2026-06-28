# SnowPro Core Test Prep Questions & Answers

Test your knowledge with these prep questions. Click **View Answer** to reveal the correct options.

---

### Question #1

Which statement best describes `clustering`?

- **A.** Clustering represents the way data is grouped together and stored within Snowflake's micro-partitions
- **B.** The database administrator must de ne the clustering methodology for each Snowflake table
- **C.** The clustering key must be included on the COPY command when loading data into Snowflake
- **D.** Clustering can be disabled within a Snowflake account

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #2

In which layer of its architecture does Snowflake store its metadata statistics?

- **A.** Storage Layer
- **B.** Compute Layer
- **C.** Database Layer
- **D.** Cloud Services Layer

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #3

Which type of table corresponds to a single Snowflake session?

- **A.** Temporary
- **B.** Transient
- **C.** Provisional
- **D.** Permanent

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #4

Select the three types of tables that exist within Snowflake (Choose three.)

- **A.** Temporary
- **B.** Transient
- **C.** Provisional
- **D.** Permanent

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, B, D

</details>

---

### Question #5

Which of the following statements are true of Snowflake data loading? (Choose three.)

- **A.** VARIANT null values are not the same as SQL NULL values
- **B.** It is recommended to do frequent, single row DMLs
- **C.** It is recommended to validate the data before loading into the Snowflake target table
- **D.** It is recommended to use staging tables to manage MERGE statements

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, C, D

</details>

---

### Question #6

Increasing the maximum number of clusters in a Multi-Cluster Warehouse is an example of:

- **A.** Scaling rhythmically
- **B.** Scaling max
- **C.** Scaling out
- **D.** Scaling up

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #7

What is the maximum compressed row size in Snowflake?

- **A.** 8KB
- **B.** 16MB
- **C.** 50MB
- **D.** 4000GB

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #8

Which of the following are common use cases for zero-copy cloning? (Choose three.)

- **A.** Quick provisioning of Dev and Test/QA environments
- **B.** Data backups
- **C.** Point in time snapshots
- **D.** Performance optimization

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, B, C

</details>

---

### Question #9

Query results are stored in the Result Cache for how long after they are last accessed, assuming no data changes have occurred?

- **A.** 1 Hour
- **B.** 3 Hours
- **C.** 12 hours
- **D.** 24 hours

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #10

Which of the following statements is true of zero-copy cloning?

- **A.** Zero-copy clones increase storage costs as cloning the table requires storing its data twice
- **B.** All zero-copy clone objects inherit the privileges of their original objects
- **C.** Zero-copy cloning is licensed as an additional Snowflake feature
- **D.** At the instance/instant a clone is created, all micro-partitions in the original table and the clone are fully shared

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #11

Which of the following terms best describes Snowflake's database architecture?

- **A.** Columnar shared nothing
- **B.** Shared disk
- **C.** Multi-cluster, shared data
- **D.** Cloud-native shared memory

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #12

Which of the following objects is not covered by Time Travel?

- **A.** Tables
- **B.** Schemas
- **C.** Databases
- **D.** Stages

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #13

When can a Virtual Warehouse start running queries?

- **A.** 12am-5am
- **B.** Only during administrator defined time slots
- **C.** When its provisioning is complete
- **D.** After replication

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #14

When scaling out by adding clusters to a multi-cluster warehouse, you are primarily scaling for improved:

- **A.** Concurrency
- **B.** Performance

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #15

What is the minimum Snowflake edition that provides multi-cluster warehouses and up to 90 days of Time Travel?

- **A.** Standard
- **B.** Premier
- **C.** Enterprise
- **D.** Business Critical Edition

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #16

What is the most granular object that the Time Travel retention period can be defined on?

- **A.** Account
- **B.** Database
- **C.** Schema
- **D.** Table

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #17

What is the minimum Snowflake edition that has column-level security enabled?

- **A.** Standard
- **B.** Enterprise
- **C.** Business Critical
- **D.** Virtual Private Snowflake

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #18

Which of the following are characteristics of Snowflake virtual warehouses? (Choose two.)

- **A.** Auto-resume applies only to the last warehouse that was started in a multi-cluster warehouse.
- **B.** The ability to auto-suspend a warehouse is only available in the Enterprise edition or above.
- **C.** SnowSQL supports both a configuration file and a command line option for specifying a default warehouse.
- **D.** A user cannot specify a default warehouse when using the ODBC driver.
- **E.** The default virtual warehouse size can be changed at any time.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, E

</details>

---

### Question #19

What versions of Snowflake should be used to manage compliance with Personal Identifiable Information (PII) requirements? (Choose two.)

- **A.** Custom Edition
- **B.** Virtual Private Snowflake
- **C.** Business Critical Edition
- **D.** Standard Edition
- **E.** Enterprise Edition

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, C - https://docs.snowflake.com/en/user-guide/intro-editions.html

</details>

---

### Question #20

What is the SNOWFLAKEFL.ACCOUNT_USAGE view that contains information about which objects were read by queries within the last 365 days
(1 year)?

- **A.** VIEWS_HISTORY
- **B.** OBJECT_HISTORY
- **C.** ACCESS_HISTORY
- **D.** LOGIN_HISTORY

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #21

How does Snowflake Fail-safe protect data in a permanent table?

- **A.** Fail-safe makes data available up to 1 day, recoverable by user operations.
- **B.** Fail-safe makes data available for 7 days, recoverable by user operations.
- **C.** Fail-safe makes data available for 7 days, recoverable only by Snowflake Support.
- **D.** Fail-safe makes data available up to 1 day, recoverable only by Snowflake Support.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #22

A virtual warehouse is created using the following command:
Create warehouse my_WH with
warehouse_size = MEDIUM
min_cluster_count = 1
max_cluster_count = 1
auto_suspend = 60
auto_resume = true;
The image below is a graphical representation of the warehouse utilization across two days.
What action should be taken to address this situation?

- **A.** Increase the warehouse size from Medium to 2XL.
- **B.** Increase the value for the parameter MAX_CONCURRENCY_LEVEL.
- **C.** Configure the warehouse to a multi-cluster warehouse.
- **D.** Lower the value of the parameter STATEMENT_QUEUED_TIMEOUT_IN_SECONDS.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #23

What is the minimum Snowflake edition required to create a materialized view?

- **A.** Standard Edition
- **B.** Enterprise Edition
- **C.** Business Critical Edition
- **D.** Virtual Private Snowflake Edition

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #24

Which methods can be used to delete staged files from a Snowflake stage? (Choose two.)

- **A.** Use the DROP command after the load completes.
- **B.** Specify the TEMPORARY option when creating the file format.
- **C.** Specify the PURGE copy option in the COPY INTO command.
- **D.** Use the REMOVE command after the load completes.
- **E.** Use the DELETE LOAD HISTORY command after the load completes.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, D

</details>

---

### Question #25

On which of the following cloud platforms can a Snowflake account be hosted? (Choose three.)

- **A.** Amazon Web Services
- **B.** Private Virtual Cloud
- **C.** Oracle Cloud
- **D.** Microsoft Azure Cloud
- **E.** Google Cloud Platform
- **F.** Alibaba Cloud

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, D, E

</details>

---

### Question #26

What Snowflake role must be granted for a user to create and manage accounts?

- **A.** ACCOUNTADMIN
- **B.** ORGADMIN
- **C.** SECURITYADMIN
- **D.** SYSADMIN

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #27

Assume there is a table consisting of five micro-partitions with values ranging from A to Z.
Which diagram indicates a well-clustered table?
A.
B.
C.
D.


<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #28

What feature can be used to reorganize a very large table on one or more columns?

- **A.** Micro-partitions
- **B.** Clustering keys
- **C.** Key partitions
- **D.** Clustered partitions

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #29

What is an advantage of using an explain plan instead of the query profiler to evaluate the performance of a query?

- **A.** The explain plan output is available graphically.
- **B.** An explain plan can be used to conduct performance analysis without executing a query.
- **C.** An explain plan will handle queries with temporary tables and the query profiler will not.
- **D.** An explain plan's output will display automatic data skew optimization information.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #30

Which data types are supported by Snowflake when using semi-structured data? (Choose two.)

- **A.** VARIANT
- **B.** VARRAY
- **C.** STRUCT
- **D.** ARRAY
- **E.** QUEUE

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, D

</details>

---

### Question #31

Why does Snowflake recommend file sizes of 100-250 MB compressed when loading data?

- **A.** Optimizes the virtual warehouse size and multi-cluster setting to economy mode
- **B.** Allows a user to import the files in a sequential order
- **C.** Increases the latency staging and accuracy when loading the data
- **D.** Allows optimization of parallel operations

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #32

Which of the following features are available with the Snowflake Enterprise edition? (Choose two.)

- **A.** Database replication and failover
- **B.** Automated index management
- **C.** Customer managed keys (Tri-secret secure)
- **D.** Extended time travel
- **E.** Native support for geospatial data

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D, E

</details>

---

### Question #33

What is the default file size when unloading data from Snowflake using the COPY command?

- **A.** 5 MB
- **B.** 8 GB
- **C.** 16 MB
- **D.** 32 MB

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #34

What features that are part of the Continuous Data Protection (CDP) feature set in Snowflake do not require additional configuration?
(Choose two.)

- **A.** Row level access policies
- **B.** Data masking policies
- **C.** Data encryption
- **D.** Time Travel
- **E.** External tokenization

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, D

</details>

---

### Question #35

Which Snowflake layer is always leveraged when accessing a query from the result cache?

- **A.** Metadata
- **B.** Data Storage
- **C.** Compute
- **D.** Cloud Services

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #36

Which connectors are available in the downloads section of the Snowflake web interface (UI)? (Choose two.)

- **A.** SnowSQL
- **B.** JDBC
- **C.** ODBC
- **D.** HIVE
- **E.** Scala

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, C old question, ignore.

</details>

---

### Question #37

A Snowflake Administrator needs to ensure that sensitive corporate data in Snowflake tables is not visible to end users, but is partially
visible to functional managers.
How can this requirement be met?

- **A.** Use data encryption.
- **B.** Use dynamic data masking.
- **C.** Use secure materialized views.
- **D.** Revoke all roles for functional managers and end users.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #38

Users are responsible for data storage costs until what occurs?

- **A.** Data expires from Time Travel
- **B.** Data expires from Fail-safe
- **C.** Data is deleted from a table
- **D.** Data is truncated from a table

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #39

A user has an application that writes a new file to a cloud storage location every 5 minutes.
What would be the MOST efficient way to get the files into Snowflake?

- **A.** Create a task that runs a COPY INTO operation from an external stage every 5 minutes.
- **B.** Create a task that PUTS the files in an internal stage and automate the data loading wizard.
- **C.** Create a task that runs a GET operation to intermittently check for new files.
- **D.** Set up cloud provider notifications on the file location and use Snowpipe with auto-ingest.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #40

What affects whether the query results cache can be used?

- **A.** If the query contains a deterministic function
- **B.** If the virtual warehouse has been suspended
- **C.** If the referenced data in the table has changed
- **D.** If multiple users are using the same virtual warehouse

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #41

Which of the following is an example of an operation that can be completed without requiring compute, assuming no queries have been
executed previously?

- **A.** SELECT SUM (ORDER_AMT) FROM SALES;
- **B.** SELECT AVG(ORDER_QTY) FROM SALES;
- **C.** SELECT MIN(ORDER_AMT) FROM SALES;
- **D.** SELECT ORDER_AMT * ORDER_QTY FROM SALES;

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #42

How many days is load history for Snowpipe retained?

- **A.** 1 day
- **B.** 7 days
- **C.** 14 days
- **D.** 64 days

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #43

What Snowflake features allow virtual warehouses to handle high concurrency workloads? (Choose two.)

- **A.** The ability to scale up warehouses
- **B.** The use of warehouse auto scaling
- **C.** The ability to resize warehouses
- **D.** Use of multi-clustered warehouses
- **E.** The use of warehouse indexing

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, D

</details>

---

### Question #44

Which COPY INTO command outputs the data into one file?

- **A.** SINGLE=TRUE
- **B.** MAX_FILE_NUMBER=1
- **C.** FILE_NUMBER=1
- **D.** MULTIPLE=FALSE

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #45

In which scenarios would a user have to pay Cloud Services costs? (Choose two.)

- **A.** Compute Credits = 50 Credits Cloud Services = 10
- **B.** Compute Credits = 80 Credits Cloud Services = 5
- **C.** Compute Credits = 100 Credits Cloud Services = 9
- **D.** Compute Credits = 120 Credits Cloud Services = 10
- **E.** Compute Credits = 200 Credits Cloud Services = 26

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, E

</details>

---

### Question #46

A user created a new worksheet within the Snowsight UI and wants to share this with teammates.
How can this worksheet be shared?

- **A.** Create a zero-copy clone of the worksheet and grant permissions to teammates.
- **B.** Create a private Data Exchange so that any teammate can use the worksheet.
- **C.** Share the worksheet with teammates within Snowsight.
- **D.** Create a database and grant all permissions to teammates.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #47

How can a row access policy be applied to a table or a view? (Choose two.)

- **A.** Within the policy DDL
- **B.** Within the create table or create view DDL
- **C.** By future APPLY for all objects in a schema
- **D.** Within a control table
- **E.** Using the command ALTER [object] ADD ROW ACCESS POLICY [policy];

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, E

</details>

---

### Question #48

Which command can be used to load data files into a Snowflake stage?

- **A.** JOIN
- **B.** COPY INTO
- **C.** PUT
- **D.** GET

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #49

What types of data listings are available in the Snowflake Data Marketplace? (Choose two.)

- **A.** Reader
- **B.** Consumer
- **C.** Vendor
- **D.** Standard
- **E.** Personalized

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D, E

</details>

---

### Question #50

What is the maximum Time Travel retention period for a temporary Snowflake table?

- **A.** 90 days
- **B.** 1 day
- **C.** 7 days
- **D.** 45 days

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #51

When should a multi-cluster warehouse be used in auto-scaling mode?

- **A.** When it is unknown how much compute power is needed
- **B.** If the select statement contains a large number of temporary tables or Common Table Expressions (CTEs)
- **C.** If the runtime of the executed query is very slow
- **D.** When a large number of concurrent queries are run on the same warehouse

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #52

What happens when a cloned table is replicated to a secondary database? (Choose two.)

- **A.** A read-only copy of the cloned tables is stored.
- **B.** The replication will not be successful.
- **C.** The physical data is replicated.
- **D.** Additional costs for storage are charged to a secondary account.
- **E.** Metadata pointers to cloned tables are replicated.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, D

</details>

---

### Question #53

Snowflake supports the use of external stages with which cloud platforms? (Choose three.)

- **A.** Amazon Web Services
- **B.** Docker
- **C.** IBM Cloud
- **D.** Microsoft Azure Cloud
- **E.** Google Cloud Platform
- **F.** Oracle Cloud

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, D, E

</details>

---

### Question #54

What is a limitation of a Materialized View?

- **A.** A Materialized View cannot support any aggregate functions
- **B.** A Materialized View can only reference up to two tables
- **C.** A Materialized View cannot be joined with other tables
- **D.** A Materialized View cannot be defined with a JOIN

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #55

In the Snowflake access control model, which entity owns an object by default?

- **A.** The user who created the object
- **B.** The SYSADMIN role
- **C.** Ownership depends on the type of object
- **D.** The role used to create the object

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #56

What is the minimum Snowflake edition required to use Dynamic Data Masking?

- **A.** Standard
- **B.** Enterprise
- **C.** Business Critical
- **D.** Virtual Private Snowflake

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #57

Which services does the Snowflake Cloud Services layer manage? (Choose two.)

- **A.** Compute resources
- **B.** Query execution
- **C.** Authentication
- **D.** Data storage
- **E.** Metadata

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, E

</details>

---

### Question #58

A company needs to allow some users to see Personally Identifiable Information (PII) while limiting other users from seeing the full
value of the PII.
Which Snowflake feature will support this?

- **A.** Row access policies
- **B.** Data masking policies
- **C.** Data encryption
- **D.** Role based access control

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #59

A user has unloaded data from a Snowflake table to an external stage.
Which command can be used to verify if data has been uploaded to the external stage named my_stage?

- **A.** view @my_stage
- **B.** list @my_stage
- **C.** show @my_stage
- **D.** display @my_stage

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #60

Which tasks are performed in the Snowflake Cloud Services layer? (Choose two.)

- **A.** Management of metadata
- **B.** Computing the data
- **C.** Maintaining Availability Zones
- **D.** Infrastructure security
- **E.** Parsing and optimizing queries

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, E

</details>

---

### Question #61

What is true about sharing data in Snowflake? (Choose two.)

- **A.** The Data Consumer pays for data storage as well as for data computing.
- **B.** The shared data is copied into the Data Consumer account, so the Consumer can modify it without impacting the base data of the Provider.
- **C.** A Snowflake account can both provide and consume shared data.
- **D.** The Provider is charged for compute resources used by the Data Consumer to query the shared data.
- **E.** The Data Consumer pays only for compute resources to query the shared data.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, E

</details>

---

### Question #62

The following JSON is stored in a VARIANT column called src of the CAR_SALES table:
A user needs to extract the dealership information from the JSON.
How can this be accomplished?

- **A.** select src:dealership from car_sales;
- **B.** select src.dealership from car_sales;
- **C.** select src:Dealership from car_sales;
- **D.** select dealership from car_sales;

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #63

Which of the following significantly improves the performance of selective point lookup queries on a table?

- **A.** Clustering
- **B.** Materialized Views
- **C.** Zero-copy Cloning
- **D.** Search Optimization Service

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #64

Which of the following accurately describes shares?

- **A.** Tables, secure views, and secure UDFs can be shared
- **B.** Shares can be shared
- **C.** Data consumers can clone a new table from a share
- **D.** Access to a share cannot be revoked once granted

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #65

What are best practice recommendations for using the ACCOUNTADMIN system-defined role in Snowflake ? (Choose two.)

- **A.** Ensure all ACCOUNTADMIN roles use Multi-factor Authentication (MFA).
- **B.** All users granted ACCOUNTADMIN role must be owned by the ACCOUNTADMIN role.
- **C.** The ACCOUNTADMIN role must be granted to only one user.
- **D.** Assign the ACCOUNTADMIN role to at least two users, but as few as possible.
- **E.** All users granted ACCOUNTADMIN role must also be granted SECURITYADMIN role.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, D

</details>

---

### Question #66

In the query profiler view for a query, which components represent areas that can be used to help optimize query performance? (Choose
two.)

- **A.** Bytes scanned
- **B.** Bytes sent over the network
- **C.** Number of partitions scanned
- **D.** Percentage scanned from cache
- **E.** External bytes scanned

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, C ? Open to debate.

</details>

---

### Question #67

What is the minimum Snowflake edition required for row level security?

- **A.** Standard
- **B.** Enterprise
- **C.** Business Critical
- **D.** Virtual Private Snowflake

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #68

What is the minimum Fail-safe retention time period for transient tables?

- **A.** 1 day
- **B.** 7 days
- **C.** 12 hours
- **D.** 0 days

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #69

What is a machine learning and data science partner within the Snowflake Partner Ecosystem?

- **A.** Informatica
- **B.** Power BI
- **C.** Adobe
- **D.** Data Robot

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #70

Which statements are correct concerning the leveraging of third-party data from the Snowflake Data Marketplace? (Choose two.)

- **A.** Data is live, ready-to-query, and can be personalized.
- **B.** Data needs to be loaded into a cloud provider as a consumer account.
- **C.** Data is not available for copying or moving to an individual Snowflake account.
- **D.** Data is available without copying or moving.
- **E.** Data transformations are required when combining Data Marketplace datasets with existing data in Snowflake.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, D

</details>

---

### Question #71

What impacts the credit consumption of maintaining a materialized view? (Choose two.)

- **A.** Whether or not it is also a secure view
- **B.** How often the underlying base table is queried
- **C.** How often the base table changes
- **D.** Whether the materialized view has a cluster key defined
- **E.** How often the materialized view is queried

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, D

</details>

---

### Question #72

What COPY INTO SQL command should be used to unload data into multiple files?

- **A.** SINGLE=TRUE
- **B.** MULTIPLE=TRUE
- **C.** MULTIPLE=FALSE
- **D.** SINGLE=FALSE

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #73

When cloning a database containing stored procedures and regular views, that have fully qualified table references, which of the
following will occur?

- **A.** The cloned views and the stored procedures will reference the cloned tables in the cloned database.
- **B.** An error will occur, as views with qualified references cannot be cloned.
- **C.** An error will occur, as stored objects cannot be cloned.
- **D.** The stored procedures and views will refer to tables in the source database.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #74

When loading data into Snowflake, how should the data be organized?

- **A.** Into single files with 100-250 MB of compressed data per file
- **B.** Into single files with 1-100 MB of compressed data per file
- **C.** Into files of maximum size of 1 GB of compressed data per file
- **D.** Into files of maximum size of 4 GB of compressed data per file

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #75

Which of the following objects can be directly restored using the UNDROP command? (Choose two.)

- **A.** Schema
- **B.** View
- **C.** Internal stage
- **D.** Table
- **E.** User
- **F.** Role

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, D

</details>

---

### Question #76

Which Snowflake SQL statement would be used to determine which users and roles have access to a role called MY_ROLE?

- **A.** SHOW GRANTS OF ROLE MY_ROLE
- **B.** SHOW GRANTS TO ROLE MY_ROLE
- **C.** SHOW GRANTS FOR ROLE MY_ROLE
- **D.** SHOW GRANTS ON ROLE MY_ROLE

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #77

What is the MINIMUM edition of Snowflake that is required to use a SCIM security integration?

- **A.** Business Critical Edition
- **B.** Standard Edition
- **C.** Virtual Private Snowflake (VPS)
- **D.** Enterprise Edition

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #78

A user created a transient table and made several changes to it over the course of several days. Three days after the table was created,
the user would like to go back to the first version of the table.
How can this be accomplished?

- **A.** Use Time Travel, as long as DATA_RETENTION_TIME_IN_DAYS was set to at least 3 days.
- **B.** The transient table version cannot be retrieved after 24 hours.
- **C.** Contact Snowflake Support to have the data retrieved from Fail-safe storage.
- **D.** Use the FAIL_SAFE parameter for Time Travel to retrieve the data from Fail-safe storage.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #79

When reviewing the load for a warehouse using the load monitoring chart, the chart indicates that a high volume of queries is always
queuing in the warehouse.
According to recommended best practice, what should be done to reduce the queue volume? (Choose two.)

- **A.** Use multi-clustered warehousing to scale out warehouse capacity.
- **B.** Scale up the warehouse size to allow queries to execute faster.
- **C.** Stop and start the warehouse to clear the queued queries.
- **D.** Migrate some queries to a new warehouse to reduce load.
- **E.** Limit user access to the warehouse so fewer queries are run against it.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, B - the keyword for me is "Always"....not sure?

</details>

---

### Question #80

Which of the following features, associated with Continuous Data Protection (CDP), require additional Snowflake -provided data
storage? (Choose two.)

- **A.** Tri-Secret Secure
- **B.** Time Travel
- **C.** Fail-safe
- **D.** Data encryption
- **E.** External stages

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, C

</details>

---

### Question #81

Where can a user find and review the failed logins of a specific user for the past 30 days?

- **A.** The USERS view in ACCOUNT_USAGE
- **B.** The LOGIN_HISTORY view in ACCOUNT_USAGE
- **C.** The ACCESS_HISTORY view in ACCOUNT_USAGE
- **D.** The SESSIONS view in ACCOUNT_USAGE

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #82

What is the purpose of an External Function?

- **A.** To call code that executes outside of Snowflake
- **B.** To run a function in another Snowflake database
- **C.** To share data in Snowflake with external parties
- **D.** To ingest data from on-premises data sources

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #83

Which of the following statements apply to Snowflake in terms of security? (Choose two.)

- **A.** Snowflake leverages a Role-Based Access Control (RBAC) model.
- **B.** Snowflake requires a user to configure an IAM user to connect to the database.
- **C.** All data in Snowflake is encrypted.
- **D.** Snowflake can run within a user's own Virtual Private Cloud (VPC).
- **E.** All data in Snowflake is compressed.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, C

</details>

---

### Question #84

A single user of a virtual warehouse has set the warehouse to auto-resume and auto-suspend after 10 minutes. The warehouse is
currently suspended and the user performs the following actions:
1. Runs a query that takes 3 minutes to complete
2. Leaves for 15 minutes
3. Returns and runs a query that takes 10 seconds to complete
4. Manually suspends the warehouse as soon as the last query was completed
When the user returns, how much billable compute time will have been consumed?

- **A.** 4 minutes
- **B.** 10 minutes
- **C.** 14 minutes
- **D.** 24 minutes

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #85

What can be used to view warehouse usage over time? (Choose two.)

- **A.** The LOAD HISTORY view
- **B.** The query history view
- **C.** The SHOW WAREHOUSES command
- **D.** The WAREHOUSE_METERING_HISTORY view
- **E.** The billing and usage tab in the Snowflake web UI

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D, E

</details>

---

### Question #86

What actions will prevent leveraging of the ResultSet cache? (Choose two.)

- **A.** Removing a column from the query SELECT list
- **B.** Stopping the virtual warehouse that the query is running against
- **C.** Clustering of the data used by the query
- **D.** Executing the RESULTS_SCAN() table function
- **E.** Changing a column that is not in the cached query

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, C

</details>

---

### Question #87

Which statement is true about running tasks in Snowflake?

- **A.** A task can be called using a CALL statement to run a set of predefined SQL commands.
- **B.** A task allows a user to execute a single SQL statement/command using a predefined schedule.
- **C.** A task allows a user to execute a set of SQL commands on a predefined schedule.
- **D.** A task can be executed using a SELECT statement to run a predefined SQL command.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #88

Which data types does Snowflake support when querying semi-structured data? (Choose two.)

- **A.** VARIANT
- **B.** VARCHAR
- **C.** XML
- **D.** ARRAY
- **E.** BLOB

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, D

</details>

---

### Question #89

In an auto-scaling multi-cluster virtual warehouse with the setting SCALING_POLICY = ECONOMY enabled, when is another cluster
started?

- **A.** When the system has enough load for 2 minutes
- **B.** When the system has enough load for 6 minutes
- **C.** When the system has enough load for 8 minutes
- **D.** When the system has enough load for 10 minutes

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #90

What is the following SQL command used for?
Select * from table(validate(t1, job_id => '_last'));

- **A.** To validate external table files in table t1 across all sessions
- **B.** To validate task SQL statements against table t1 in the last 14 days
- **C.** To validate a file for errors before it gets executed using a COPY command
- **D.** To return errors from the last executed COPY command into table t1 in the current session

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #91

A sales table FCT_SALES has 100 million records.
The following query was executed:
SELECT COUNT (1) FROM FCT_SALES;
How did Snowflake fulfill this query?

- **A.** Query against the result set cache
- **B.** Query against a virtual warehouse cache
- **C.** Query against the most-recently created micro-partition
- **D.** Query against the metadata cache

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #92

What happens when a virtual warehouse is resized?

- **A.** When increasing the size of an active warehouse the compute resource for all running and queued queries on the warehouse are affected.
- **B.** When reducing the size of a warehouse the compute resources are removed only when they are no longer being used to execute any current statements.
- **C.** The warehouse will be suspended while the new compute resource is provisioned and will resume automatically once provisioning is complete.
- **D.** Users who are trying to use the warehouse will receive an error message until the resizing is complete.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #93

What tasks can be completed using the COPY command? (Choose two.)

- **A.** Columns can be aggregated.
- **B.** Columns can be joined with an existing table.
- **C.** Columns can be reordered.
- **D.** Columns can be omitted.
- **E.** Data can be loaded without the need to spin up a virtual warehouse.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, D

</details>

---

### Question #94

Which Snowflake layer can be configured?

- **A.** Database Storage
- **B.** Cloud Services
- **C.** Query Processing
- **D.** Application Services

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #95

Query compilation occurs in which architecture layer of the Snowflake Cloud Data Platform?

- **A.** Compute layer
- **B.** Storage layer
- **C.** Cloud infrastructure layer
- **D.** Cloud services layer

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #96

If a size Small virtual warehouse is made up of two servers, how many servers make up a Large warehouse?

- **A.** 4
- **B.** 8
- **C.** 16
- **D.** 32

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #97

A clustering key was defined on a table, but it is no longer needed.
How can the key be removed?

- **A.** ALTER TABLE [TABLE NAME] PURGE CLUSTERING KEY
- **B.** ALTER TABLE [TABLE NAME] DELETE CLUSTERING KEY
- **C.** ALTER TABLE [TABLE NAME] DROP CLUSTERING KEY
- **D.** ALTER TABLE [TABLE NAME] REMOVE CLUSTERING KEY

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #98

What is a core benefit of clustering?

- **A.** To guarantee uniquely identifiable records in the database
- **B.** To increase scan efficiency in queries by improving pruning
- **C.** To improve performance by creating a separate file for point lookups
- **D.** To provide data redundancy by duplicating micro-partitions

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #99

Which statement is true about Multi-Factor Authentication (MFA) in Snowflake?

- **A.** MFA can be enforced or applied for a given role.
- **B.** Snowflake users are automatically enrolled in MFA.
- **C.** Users enroll in MFA by submitting a request to Snowflake Support.
- **D.** MFA is an integrated Snowflake feature.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #100

What data type should be used to store JSON data natively in Snowflake?

- **A.** JSON
- **B.** String
- **C.** Object
- **D.** VARIANT

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #101

What should be considered when deciding to use a Secure View? (Choose two.)

- **A.** No details of the query execution plan will be available in the query profiler.
- **B.** Once created there is no way to determine if a view is secure or not.
- **C.** Secure views do not take advantage of the same internal optimizations as standard views.
- **D.** It is not possible to create secure materialized views.
- **E.** The view definition of a secure view is still visible to users by way of the information schema.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, C

</details>

---

### Question #102

The information schema provides storage information for which of the following objects? (Choose two.)

- **A.** Users
- **B.** Databases
- **C.** Internal stages
- **D.** Resource monitors
- **E.** Pipes

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, C

</details>

---

### Question #103

What is a responsibility of SnowflakeÆs virtual warehouses?

- **A.** Infrastructure management
- **B.** Metadata management
- **C.** Query execution
- **D.** Query parsing and optimization
- **E.** Management of the storage layer

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #104

Which data type is supported by Snowflake data classification?

- **A.** Binary
- **B.** Float
- **C.** Geography
- **D.** Variant

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #105

When unloading data to an external stage, which compression format can be used for Parquet files with the COPY INTO command?

- **A.** BROTLI
- **B.** GZIP
- **C.** LZO
- **D.** ZSTD

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #106

Which SQL command can be used to verify the privileges that are granted to a role?

- **A.** SHOW GRANTS ON ROLE
- **B.** SHOW ROLES
- **C.** SHOW GRANTS TO ROLE
- **D.** SHOW GRANTS FOR ROLE

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #107

Which Query Profile result indicates that a warehouse is sized too small?

- **A.** There are a lot of filter nodes.
- **B.** Bytes are spilling to external storage.
- **C.** The number of processed rows is very high.
- **D.** The number of partitions scanned is the same as partitions total.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #108

What is the default Time Travel retention period?

- **A.** 1 day
- **B.** 7 days
- **C.** 45 days
- **D.** 90 days

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #109

Which Snowflake partner category is represented at the top of this diagram (labeled 1)?

- **A.** Business Intelligence
- **B.** Machine Learning and Data Science
- **C.** Security and Governance
- **D.** Data Integration

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #110

Which object types are protected by Fail-safe? (Choose two.)

- **A.** Permanent Tables
- **B.** Temporary Tables
- **C.** External Tables
- **D.** Materialized Views
- **E.** Transient Tables

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, D

</details>

---

### Question #111

Snowflake 's approach to the management of system access combines which of the following models? (Choose two.)

- **A.** Security Assertion Markup Language (SAML)
- **B.** Role-Based Access Control (RBAC)
- **C.** Identity Access Management (AM)
- **D.** Create, Read, Update, and Delete (CRUD)
- **E.** Discretionary Access Control (DAC)
- **F.** Mandatory Access Control (MAC)

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, E

</details>

---

### Question #112

According to Snowflake best practice recommendations, which role should be used to create databases?

- **A.** ACCOUNTADMIN
- **B.** SYSADMIN
- **C.** SECURITYADMIN
- **D.** USERADMIN

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #113

To add or remove search optimization for a table, a user must have which of the following privileges or roles? (Choose two.)

- **A.** The MODIFY privilege on the table
- **B.** The OWNERSHIP privilege on the table
- **C.** A SECURITYADMIN role
- **D.** The ADD SEARCH OPTIMIZATION privilege on the schema that contains the table
- **E.** The SELECT privilege on the table

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, D

</details>

---

### Question #114

While using a COPY command with a Validation_mode parameter, which of the following statements will return an error?

- **A.** Statements that insert a duplicate record during a load
- **B.** Statements that have a specific data type in the source
- **C.** Statements that have duplicate file names
- **D.** Statements that transform data during a load

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #115

When is the result set cache no longer available? (Choose two.)

- **A.** When another warehouse is used to execute the query
- **B.** When another user executes the query
- **C.** When the underlying data has changed
- **D.** When the warehouse used to execute the query is suspended
- **E.** When it has been 24 hours since the last query

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, E

</details>

---

### Question #116

What is the recommended file sizing for data loading using Snowpipe?

- **A.** A compressed file size greater than 100 MB, and up to 250 MB
- **B.** A compressed file size greater than 100 GB, and up to 250 GB
- **C.** A compressed file size greater than 10 MB, and up to 100 MB
- **D.** A compressed file size greater than 1 GB, and up to 2 GB

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #117

Which statements are true concerning SnowflakeÆs underlying cloud infrastructure? (Choose three.)

- **A.** Snowflake data and services are deployed in a single availability zone within a cloud providerÆs region.
- **B.** Snowflake data and services are available in a single cloud provider and a single region; the use of multiple cloud providers is not supported.
- **C.** Snowflake can be deployed in a customerÆs private cloud using the customerÆs own compute and storage resources for Snowflake compute and storage.
- **D.** Snowflake uses the core compute and storage services of each cloud provider for its own compute and storage.
- **E.** All three layers of SnowflakeÆs architecture (storage, compute, and cloud services) are deployed and managed entirely on a selected cloud platform.
- **F.** Snowflake data and services are deployed in at least three availability zones within a cloud providerÆs region.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D, E, F

</details>

---

### Question #118

A user unloaded a Snowflake table called mytable to an internal stage called mystage.
Which command can be used to view the list of files that has been uploaded to the stage?

- **A.** list @mytable;
- **B.** list @%mytable;
- **C.** list @%mystage;
- **D.** list @mystage;

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #119

What is a best practice after creating a custom role?

- **A.** Create the custom role using the SYSADMIN role.
- **B.** Assign the custom role to the SYSADMIN role.
- **C.** Assign the custom role to the PUBLIC role.
- **D.** Add _CUSTOM to all custom role names.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #120

Which is the MINIMUM required Snowflake edition that a user must have if they want to use AWS/Azure Privatelink or Google Cloud
Private Service Connect?

- **A.** Standard
- **B.** Premium
- **C.** Enterprise
- **D.** Business Critical

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #121

Which of the following query profiler variables will indicate that a virtual warehouse is not sized correctly for the query being executed?

- **A.** Bytes sent over the network
- **B.** Synchronization
- **C.** Initialization
- **D.** Remote spillage

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #122

Which of the following Snowflake capabilities are available in all Snowflake editions? (Choose two.)

- **A.** Customer-managed encryption keys through Tri-Secret Secure
- **B.** Automatic encryption of all data
- **C.** Up to 90 days of data recovery through Time Travel
- **D.** Object-level access control
- **E.** Column-level security to apply data masking policies to tables and views

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, D

</details>

---

### Question #123

A PUT command can be used to stage local files from which Snowflake interface?

- **A.** SnowSQL
- **B.** Snowflake classic web interface (UI)
- **C.** Snowsight
- **D.** .NET driver

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #124

Which of the following indicates that it may be appropriate to use a clustering key for a table? (Choose two.)

- **A.** The table contains a column that has very low cardinality.
- **B.** DML statements that are being issued against the table are blocked.
- **C.** The table has a small number of micro-partitions.
- **D.** Queries on the table are running slower than expected.
- **E.** The clustering depth for the table is large.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D, E - https://docs.snowflake.com/en/user-guide/tables-clustering-keys#label-considerations-
for-choosing-clustering

</details>

---

### Question #125

Which cache type is used to cache data output from SQL queries?

- **A.** Metadata cache
- **B.** Result cache
- **C.** Remote cache
- **D.** Local file cache

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #126

Which of the following describes how clustering keys work in Snowflake?

- **A.** Clustering keys update the micro-partitions in place with a full sort, and impact the DML operations.
- **B.** Clustering keys sort the designated columns over time, without blocking DML operations.
- **C.** Clustering keys create a distributed, parallel data structure of pointers to a table's rows and columns.
- **D.** Clustering keys establish a hashed key on each node of a virtual warehouse to optimize joins at run-time.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #127

Which of the following operations require the use of a running virtual warehouse? (Choose two.)

- **A.** Downloading data from an internal stage
- **B.** Listing files in a stage
- **C.** Executing a stored procedure
- **D.** Altering a table
- **E.** Querying data from a materialized view

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, E

</details>

---

### Question #128

What is used to limit the credit usage of a virtual warehouse within a Snowflake account?

- **A.** Load monitor
- **B.** Resource monitor
- **C.** Query Profile
- **D.** Stream

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #129

What are the benefits of the replication feature in Snowflake ? (Choose two.)

- **A.** Disaster recovery
- **B.** Time Travel
- **C.** Fail-safe
- **D.** Database failover and failback
- **E.** Data security

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, D

</details>

---

### Question #130

Which of the following roles are recommended to create and manage users and roles? (Choose two.)

- **A.** SYSADMIN
- **B.** SECURITYADMIN
- **C.** PUBLIC
- **D.** ACCOUNTADMIN
- **E.** USERADMIN

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, E

</details>

---

### Question #131

When can a newly configured virtual warehouse start running SQL queries?

- **A.** After 50% of the warehouse provisioning has completed
- **B.** During the time slots defined by the ACCOUNTADMIN
- **C.** When the warehouse provisioning is completed
- **D.** After the warehouse replication is completed

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #132

What actions will prevent leveraging of the ResultSet cache?

- **A.** Removing a column from the query SELECT list
- **B.** Stopping the virtual warehouse that the query is running against
- **C.** If the result has not been reused within the last 12 hours
- **D.** Executing the RESULTS_SCAN() table function

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #133

Which of the following are benefits of micro-partitioning? (Choose two.)

- **A.** Micro-partitions cannot overlap in their range of values.
- **B.** Micro-partitions are immutable objects that support the use of Time Travel.
- **C.** Micro-partitions can reduce the amount of I/O from object storage to virtual warehouses.
- **D.** Rows are automatically stored in sorted order within micro-partitions.
- **E.** Micro-partitions can be defined on a schema-by-schema basis.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, C

</details>

---

### Question #134

Which data type can be used to store geospatial data in Snowflake?

- **A.** Variant
- **B.** Object
- **C.** Geometry
- **D.** Geography

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C and D https://docs.snowflake.com/en/sql-reference/data-types-geospatial

</details>

---

### Question #135

If all virtual warehouse resources are maximized while processing a query workload, what will happen to new queries that are
submitted to the warehouse?

- **A.** All queries will terminate when the resources are maximized.
- **B.** The warehouse will scale out automatically
- **C.** The warehouse will move to a suspended state.
- **D.** New queries will be queued and executed when capacity is available.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #136

Masking policies can be applied to which of the following Snowflake objects? (Choose two.)

- **A.** A materialized view
- **B.** A stored procedure
- **C.** A table
- **D.** A stream
- **E.** A pipe
- **F.** A User-Defined Function (UDF)

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, C

</details>

---

### Question #137

What actions are supported by Snowflake resource monitors? (Choose two.)

- **A.** Alert
- **B.** Notify
- **C.** Notify and suspend
- **D.** Abort
- **E.** Suspend immediately

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, C https://docs.snowflake.com/en/user-guide/resource-monitors.html#actions

</details>

---

### Question #138

A user executes the following SQL query:
create table SALES_BKP like SALES;
What are the cost implications for processing this query?

- **A.** Processing costs will be generated based on how long the query takes.
- **B.** Storage costs will be generated based on the size of the data.
- **C.** No costs will be incurred as the query will use metadata.
- **D.** The cost for running the virtual warehouse will be charged by the second.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #139

What is the maximum length of time travel available in the Snowflake Standard Edition?

- **A.** 1 Day
- **B.** 7 Days
- **C.** 30 Days
- **D.** 90 Days

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #140

What happens when an external or an internal stage is dropped? (Choose two.)

- **A.** When dropping an external stage, the files are not removed and only the stage is dropped.
- **B.** When dropping an external stage, both the stage and the files within the stage are removed.
- **C.** When dropping an internal stage, the files are deleted with the stage and the files are recoverable.
- **D.** When dropping an internal stage, the files are deleted with the stage and the files are not recoverable.
- **E.** When dropping an internal stage, only selected files are deleted with the stage and are not recoverable.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, D

</details>

---

### Question #141

A user has 10 files in a stage containing new customer data. The ingest operation completes with no errors, using the following
command:
COPY INTO my_table FROM @my_stage;
The next day the user adds 10 files to the stage so that now the stage contains a mixture of new customer data and updates to the
previous data. The user did not remove the 10 original files.
If the user runs the same COPY INTO command what will happen?

- **A.** All data from all of the files on the stage will be appended to the table.
- **B.** Only data about new customers from the new files will be appended to the table.
- **C.** The operation will fail with the error UNCERTAIN FILES IN STAGE.
- **D.** All data from only the newly-added files will be appended to the table.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D - https://docs.snowflake.com/en/sql-reference/sql/copy-into-table#reloading-files

</details>

---

### Question #142

Which parameter can be used to instruct a COPY command to verify data files instead of loading them into a specified table?

- **A.** STRIP_NULL_VALUES
- **B.** SKIP_BYTE_ORDER_MARK
- **C.** REPLACE_INVALID_CHARACTERS
- **D.** VALIDATION_MODE

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #143

Which of the following SQL statements will list the version of the drivers currently being used?

- **A.** Execute SELECT CURRENT_ODBC_CLIENT(); from the Web UI
- **B.** Execute SELECT CURRENT_JDBC_VERSION(); from SnowSQL
- **C.** Execute SELECT CURRENT_CLIENT(); from an application
- **D.** Execute SELECT CURRENT_VERSION(); from the Python Connector

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #144

Which Snowflake technique can be used to improve the performance of a query?

- **A.** Clustering
- **B.** Indexing
- **C.** Fragmenting
- **D.** Using INDEX_HINTS

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #145

What happens to the shared objects for users in a consumer account from a share, once a database has been created in that account?

- **A.** The shared objects are transferred.
- **B.** The shared objects are copied.
- **C.** The shared objects become accessible.
- **D.** The shared objects can be re-shared.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #146

Using variables in Snowflake is denoted by using which SQL character?

- **A.** @
- **B.** &
- **C.** $
- **D.** #

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #147

Which commands should be used to grant the privilege allowing a role to select data from all current tables and any tables that will be
created later in a schema? (Choose two.)

- **A.** grant USAGE on all tables in schema DB1.SCHEMA to role MYROLE;
- **B.** grant USAGE on future tables in schema DB1.SCHEMA to role MYROLE;
- **C.** grant SELECT on all tables in schema DB1.SCHEMA to role MYROLE;
- **D.** grant SELECT on future tables in schema DB1.SCHEMA to role MYROLE;
- **E.** grant SELECT on all tables in database DB1 to role MYROLE;
- **F.** grant SELECT on future tables in database DB1 to role MYROLE;

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, D

</details>

---

### Question #148

How can a user change which columns are referenced in a view?

- **A.** Modify the columns in the underlying table
- **B.** Use the ALTER VIEW command to update the view
- **C.** Recreate the view with the required changes
- **D.** Materialize the view to perform the changes

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #149

Which statement describes pruning?

- **A.** The filtering or disregarding of micro-partitions that are not needed to return a query.
- **B.** The return of micro-partitions values that overlap with each other to reduce a query's runtime.
- **C.** A service that is handled by the Snowflake Cloud Services layer to optimize caching.
- **D.** The ability to allow the result of a query to be accessed as if it were a table.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #150

Which SQL command can be used to see the CREATE definition of a masking policy?

- **A.** SHOW MASKING POLICIES
- **B.** DESCRIBE MASKING POLICY
- **C.** GET_DDL
- **D.** LIST MASKING POLICIES

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #151

Which of the following is the Snowflake Account_Usage.Metering_History view used for?

- **A.** Gathering the hourly credit usage for an account
- **B.** Compiling an account's average cloud services cost over the previous month
- **C.** Summarizing the throughput of Snowpipe costs for an account
- **D.** Calculating the funds left on an account's contract

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #152

Query parsing and compilation occurs in which architecture layer of the Snowflake Cloud Data Platform?

- **A.** Cloud services layer
- **B.** Compute layer
- **C.** Storage layer
- **D.** Cloud agnostic layer

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #153

Which of the following Snowflake objects can be shared using a secure share? (Choose two.)

- **A.** Materialized views
- **B.** Sequences
- **C.** Procedures
- **D.** Tables
- **E.** Secure User Defined Functions (UDFs)

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D, E

</details>

---

### Question #154

What happens to the underlying table data when a CLUSTER BY clause is added to a Snowflake table?

- **A.** Data is hashed by the cluster key to facilitate fast searches for common data values
- **B.** Larger micro-partitions are created for common data values to reduce the number of partitions that must be scanned
- **C.** Smaller micro-partitions are created for common data values to allow for more parallelism
- **D.** Data may be co-located by the cluster key within the micro-partitions to improve pruning performance

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #155

Which of the following conditions must be met in order to return results from the results cache? (Choose two.)

- **A.** The user has the appropriate privileges on the objects associated with the query.
- **B.** Micro-partitions have been reclustered since the query was last run.
- **C.** The new query is run using the same virtual warehouse as the previous query.
- **D.** The query includes a User Defined Function (UDF).
- **E.** The query has been run within 24 hours of the previously run query.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, E

</details>

---

### Question #156

Which statement about billing applies to Snowflake credits?

- **A.** Credits are billed per-minute with a 60-minute minimum.
- **B.** Credits are used to pay for cloud data storage usage.
- **C.** Credits are consumed based on the number of credits billed for each hour that a warehouse runs.
- **D.** Credits are consumed based on the warehouse size and the time the warehouse is running.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #157

A user needs to create a materialized view in the schema MYDB.MYSCHEMA.
Which statements will provide this access?

- **A.** GRANT ROLE MYROLE TO USER USER1; CREATE MATERIALIZED VIEW ON SCHEMA MYDB.MYSCHEMA TO ROLE MYROLE;
- **B.** GRANT ROLE MYROLE TO USER USER1; CREATE MATERIALIZED VIEW ON SCHEMA MYDB.MYSCHEMA TO USER USER1;
- **C.** GRANT ROLE MYROLE TO USER USER1; CREATE MATERIALIZED VIEW ON SCHEMA MYDB.MYSCHEMA TO USER1;
- **D.** GRANT ROLE MYROLE TO USER USER1; CREATE MATERIALIZED VIEW ON SCHEMA MYDB.MYSCHEMA TO MYROLE;

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #158

What is the purpose of multi-cluster virtual warehouses?

- **A.** To create separate data warehouses to increase query optimization
- **B.** To allow users the ability to choose the type of compute nodes that make up a virtual warehouse cluster
- **C.** To eliminate or reduce queuing of concurrent queries
- **D.** To allow the warehouse to resize automatically

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #159

Which of the following is a valid source for an external stage when the Snowflake account is located on Microsoft Azure?

- **A.** An FTP server with TLS encryption
- **B.** An HTTPS server with WebDAV
- **C.** A Google Cloud storage bucket
- **D.** A Windows server file share on Azure

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #160

Which database objects can be shared with the Snowflake secure data sharing feature? (Choose two.)

- **A.** Files
- **B.** External tables
- **C.** Secure User-Defined Functions (UDFs)
- **D.** Sequences
- **E.** Streams

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, C

</details>

---

### Question #161

Which statements reflect key functionalities of a Snowflake Data Exchange? (Choose two.)

- **A.** If an account is enrolled with a Data Exchange, it will lose its access to the Snowflake Marketplace.
- **B.** A Data Exchange allows groups of accounts to share data privately among the accounts.
- **C.** A Data Exchange allows accounts to share data with third, non-Snowflake parties.
- **D.** Data Exchange functionality is available by default in accounts using the Enterprise edition or higher.
- **E.** The sharing of data in a Data Exchange is bidirectional. An account can be a provider for some datasets and a consumer for others.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, E

</details>

---

### Question #162

A Snowflake user executed a query and received the results. Another user executed the same query 4 hours later. The data had not
changed.
What will occur?

- **A.** No virtual warehouse will be used, data will be read from the result cache.
- **B.** No virtual warehouse will be used, data will be read from the local disk cache.
- **C.** The default virtual warehouse will be used to read all data.
- **D.** The virtual warehouse that is defined at the session level will be used to read all data.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #163

Which feature allows a user the ability to control the organization of data in a micro-partition?

- **A.** Range Partitioning
- **B.** Search Optimization Service
- **C.** Automatic Clustering
- **D.** Horizontal Partitioning

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C - https://docs.snowflake.com/en/user-guide/tables-auto-reclustering

</details>

---

### Question #164

Which privilege must be granted to a share to allow secure views the ability to reference data in multiple databases?

- **A.** CREATE_SHARE on the account
- **B.** SHARE on databases and schemas
- **C.** SELECT on tables used by the secure view
- **D.** REFERENCE_USAGE on databases

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #165

In which use case does Snowflake apply egress charges?

- **A.** Data sharing within a specific region
- **B.** Query result retrieval
- **C.** Database replication
- **D.** Loading data into Snowflake

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #166

Which of the following compute resources or features are managed by Snowflake? (Choose two.)

- **A.** Execute a COPY command
- **B.** Updating data
- **C.** Snowpipe
- **D.** AUTOMATIC_CLUSTERING
- **E.** Scaling up a warehouse

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, D

</details>

---

### Question #167

A materialized view should be created when which of the following occurs? (Choose two.)

- **A.** There is minimal cost associated with running the query.
- **B.** The query consumes many compute resources every time it runs.
- **C.** The base table gets updated frequently.
- **D.** The query is highly optimized and does not consume many compute resources.
- **E.** The results of the query do not change often and are used frequently.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, E

</details>

---

### Question #168

What privilege should a user be granted to change permissions for new objects in a managed access schema?

- **A.** Grant the OWNERSHIP privilege on the schema.
- **B.** Grant the OWNERSHIP privilege on the database.
- **C.** Grant the MANAGE GRANTS global privilege.
- **D.** Grant ALL privileges on the schema.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #169

What happens when a Data Provider revokes privileges to a share on an object in their source database?

- **A.** The object immediately becomes unavailable for all Data Consumers.
- **B.** Any additional data arriving after this point in time will not be visible to Data Consumers.
- **C.** The Data Consumers stop seeing data updates and become responsible for storage charges for the object.
- **D.** A static copy of the object at the time the privilege was revoked is created in the Data Consumers account.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #170

Which command can be used to load data into an internal stage?

- **A.** LOAD
- **B.** COPY
- **C.** GET
- **D.** PUT

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #171

What is the MINIMUM Snowflake edition required to use the periodic rekeying of micro-partitions?

- **A.** Enterprise
- **B.** Business Critical
- **C.** Standard
- **D.** Virtual Private Snowflake

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #172

Which stage type can be altered and dropped?

- **A.** Database stage
- **B.** External stage
- **C.** Table stage
- **D.** User stage

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #173

Which Snowflake object enables loading data from files as soon as they are available in a cloud storage location?

- **A.** Pipe
- **B.** External stage
- **C.** Task
- **D.** Stream

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #174

A user is loading JSON documents composed of a huge array containing multiple records into Snowflake. The user enables the
STRIP_OUTER_ARRAY file format option.
What does the STRIP_OUTER_ARRAY file format do?

- **A.** It removes the last element of the outer array.
- **B.** It removes the outer array structure and loads the records into separate table rows.
- **C.** It removes the trailing spaces in the last element of the outer array and loads the records into separate table columns.
- **D.** It removes the NULL elements from the JSON object eliminating invalid data and enables the ability to load the records.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #175

Which of the following describes how multiple Snowflake accounts in a single organization relate to various cloud providers?

- **A.** Each Snowflake account can be hosted in a different cloud vendor and region.
- **B.** Each Snowflake account must be hosted in a different cloud vendor and region.
- **C.** All Snowflake accounts must be hosted in the same cloud vendor and region.
- **D.** Each Snowflake account can be hosted in a different cloud vendor but must be in the same region.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #176

If a Snowflake user decides a table should be clustered, what should be used as the cluster key?

- **A.** The columns that are queried in the select clause.
- **B.** The columns with very high cardinality.
- **C.** The columns with many different values.
- **D.** The columns most actively used in the select filters.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #177

What are value types that a VARIANT column can store? (Choose two.)

- **A.** STRUCT
- **B.** OBJECT
- **C.** BINARY
- **D.** ARRAY
- **E.** CLOB

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, D

</details>

---

### Question #178

A company needs to read multiple terabytes of data for an initial load as part of a Snowflake migration. The company can control the
number and size of CSV extract files.
How does Snowflake recommend maximizing the load performance?

- **A.** Use auto-ingest Snowpipes to load large files in a serverless model.
- **B.** Produce the largest files possible, reducing the overall number of files to process.
- **C.** Produce a larger number of smaller files and process the ingestion with size Small virtual warehouses.
- **D.** Use an external tool to issue batched row-by-row inserts within BEGIN TRANSACTION and COMMIT commands.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #179

For non-materialized views, what column in Information Schema and Account Usage identifes whether a view is secure or not?

- **A.** CHECK_OPTION
- **B.** IS_SECURE
- **C.** IS_UPDATEABLE
- **D.** TABLE_NAME

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #180

The bulk data load history that is available upon completion of the COPY statement is stored where and for how long?

- **A.** In the metadata of the target table for 14 days
- **B.** In the metadata of the pipe for 14 days
- **C.** In the metadata of the target table for 64 days
- **D.** In the metadata of the pipe for 64 days

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C - https://docs.snowflake.com/en/user-guide/data-load-snowpipe-intro.html#load-history

</details>

---

### Question #181

User INQUISITIVE_PERSON has been granted the role DATA_SCIENCE. The role DATA_SCIENCE has privileges OWNERSHIP on the
schema MARKETING of the database ANALYTICS_DW.
Which command will show all privileges granted to that schema?

- **A.** SHOW GRANTS ON ROLE DATA_SCIENCE
- **B.** SHOW GRANTS ON SCHEMA ANALYTICS_DW.MARKETING
- **C.** SHOW GRANTS TO USER INQUISITIVE_PERSON
- **D.** SHOW GRANTS OF ROLE DATA_SCIENCE

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #182

Which of the following are characteristics of security in Snowflake?

- **A.** Account and user authentication is only available with the Snowflake Business Critical edition.
- **B.** Support for HIPAA and GDPR compliance is available for UI Snowflake editions.
- **C.** Periodic rekeying of encrypted data is available with the Snowflake Enterprise edition and higher
- **D.** Private communication to internal stages is allowed in the Snowflake Enterprise edition and higher.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #183

Which of the following objects can be shared through secure data sharing?

- **A.** Masking policy
- **B.** Stored procedure
- **C.** Task
- **D.** External table

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #184

Which formats does Snowflake store unstructured data in? (Choose two.)

- **A.** GeoJSON
- **B.** Array
- **C.** XML
- **D.** Object
- **E.** BLOB

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, D

</details>

---

### Question #185

A user is preparing to load data from an external stage.
Which practice will provide the MOST efficient loading performance?

- **A.** Organize files into logical paths
- **B.** Store the files on the external stage to ensure caching is maintained
- **C.** Use pattern matching for regular expression execution
- **D.** Load the data in one large file

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #186

What effect does WAIT_FOR_COMPLETION = TRUE have when running an ALTER WAREHOUSE command and changing the warehouse
size?

- **A.** The warehouse size does not change until all queries currently running in the warehouse have completed.
- **B.** The warehouse size does not change until all queries currently in the warehouse queue have completed.
- **C.** The warehouse size does not change until the warehouse is suspended and restarted.
- **D.** It does not return from the command until the warehouse has finished changing its size.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #187

Which of the following can be used when unloading data from Snowflake? (Choose two.)

- **A.** When unloading semi-structured data, it is recommended that the STRIP_OUTER_ARRAY option be used.
- **B.** Use the ENCODING file format option to change the encoding from the default UTF-8.
- **C.** The OBJECT_CONSTRUCT function can be used to convert relational data to semi-structured data.
- **D.** By using the SINGLE = TRUE parameter, a single file up to 5 GB in size can be exported to the storage layer.
- **E.** Use the PARSE_JSON function to ensure structured data will be unloaded into the VARIANT data type.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, D

</details>

---

### Question #188

What data is stored in the Snowflake storage layer? (Choose two.)

- **A.** Snowflake parameters
- **B.** Micro-partitions
- **C.** Query history
- **D.** Persisted query results
- **E.** Standard and secure view results

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, C

</details>

---

### Question #189

A data provider wants to share data with a consumer who does not have a Snowflake account. The provider creates a reader account
for the consumer following these steps:
1. Created a user called "CONSUMER"
2. Created a database to hold the share and an extra-small warehouse to query the data
3. Granted the role PUBLIC the following privileges: Usage on the warehouse, database, and schema, and SELECT on all the objects in
the share
Based on this configuration what is true of the reader account?

- **A.** The reader account will automatically use the Standard edition of Snowflake.
- **B.** The reader account compute will be billed to the provider account.
- **C.** The reader account can clone data the provider has shared, but cannot re-share it.
- **D.** The reader account can create a copy of the shared data using CREATE TABLE AS...

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #190

Which of the following activities consume virtual warehouse credits in the Snowflake environment? (Choose two.)

- **A.** Caching query results
- **B.** Running EXPLAIN and SHOW commands
- **C.** Cloning a database
- **D.** Running a custom query
- **E.** Running COPY commands

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D, E

</details>

---

### Question #191

When loading data into Snowflake, the COPY command supports which of the following?

- **A.** Joins
- **B.** Filters
- **C.** Column reordering
- **D.** Aggregates

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #192

What is cached during a query on a virtual warehouse?

- **A.** All columns in a micro-partition
- **B.** Any columns accessed during the query
- **C.** The columns in the result set of the query
- **D.** All rows accessed during the query

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** E, M, A, I, L me if u know for sure..

</details>

---

### Question #193

What is the default character set used when loading CSV files into Snowflake?

- **A.** UTF-8
- **B.** UTF-16
- **C.** ISO 8859-1
- **D.** ANSI_X3.4

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #194

Which of the following describes external functions in Snowflake?

- **A.** They are a type of User-defined Function (UDF).
- **B.** They contain their own SQL code.
- **C.** They call code that is stored inside of Snowflake.
- **D.** They can return multiple rows for each row received.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #195

Which of the following are valid methods for authenticating users for access into Snowflake? (Choose three.)

- **A.** SCIM
- **B.** Federated authentication
- **C.** TLS 1.2
- **D.** Key-pair authentication
- **E.** OAuth
- **F.** OCSP authentication

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, D, E

</details>

---

### Question #196

A user has a standard multi-cluster warehouse auto-scaling policy in place.
Which condition will trigger a cluster to shut-down?

- **A.** When after 2-3 consecutive checks the system determines that the load on the most-loaded cluster could be redistributed.
- **B.** When after 5-6 consecutive checks the system determines that the load on the most-loaded cluster could be redistributed.
- **C.** When after 5-6 consecutive checks the system determines that the load on the least-loaded cluster could be redistributed.
- **D.** When after 2-3 consecutive checks the system determines that the load on the least-loaded cluster could be redistributed.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #197

What is the minimum Snowflake edition needed for database failover and fail-back between Snowflake accounts for business continuity
and disaster recovery?

- **A.** Standard
- **B.** Enterprise
- **C.** Business Critical
- **D.** Virtual Private Snowflake

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #198

How would a user execute a series of SQL statements using a task?

- **A.** Include the SQL statements in the body of the task CREATE TASK mytask .. AS INSERT INTO target1 SELECT .. FROM stream_s1 WHERE .. INSERT INTO target2 SELECT .. FROM stream_s1 WHERE ..
- **B.** A stored procedure can have only one DML statement per stored procedure invocation and therefore the user should sequence stored procedure calls in the task definition CREATE TASK mytask .. AS call stored_proc1(); call stored_proc2();
- **C.** Use a stored procedure executing multiple SQL statements and invoke the stored procedure from the task. CREATE TASK mytask . ... AS call stored_proc_multiple_statements_inside();
- **D.** Create a task for each SQL statement (e.g. resulting in task1, task2, etc.) and string the series of SQL statements by having a control task calling task1, task2, etc. sequentially.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #199

How many resource monitors can be assigned at the account level?

- **A.** 1
- **B.** 2
- **C.** 3
- **D.** 4

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #200

Data storage for individual tables can be monitored using which commands and/or objects? (Choose two.)

- **A.** SHOW STORAGE BY TABLE;
- **B.** SHOW TABLES;
- **C.** Information Schema -> TABLE_HISTORY
- **D.** Information Schema -> TABLE_FUNCTION
- **E.** Information Schema -> TABLE_STORAGE_METRICS

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, E

</details>

---

### Question #201

How would a user run a multi-cluster warehouse in maximized mode?

- **A.** Configure the maximum clusters setting to "Maximum."
- **B.** Turn on the additional clusters manually after starting the warehouse.
- **C.** Set the minimum Clusters and maximum Clusters settings to the same value.
- **D.** Set the minimum clusters and maximum clusters settings to different values.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #202

What internal stages are available in Snowflake? (Choose three.)

- **A.** Schema stage
- **B.** Named stage
- **C.** User stage
- **D.** Stream stage
- **E.** Table stage
- **F.** Database stage

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, C, E

</details>

---

### Question #203

Which stages are used with the Snowflake PUT command to upload files from a local file system? (Choose three.)

- **A.** Schema Stage
- **B.** User Stage
- **C.** Database Stage
- **D.** Table Stage
- **E.** External Named Stage
- **F.** Internal Named Stage

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, D, F

</details>

---

### Question #204

Which data type can store more than one type of data structure?

- **A.** JSON
- **B.** BINARY
- **C.** VARCHAR
- **D.** VARIANT

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #205

User-level network policies can be created by which of the following roles? (Choose two.)

- **A.** ROLEADMIN
- **B.** ACCOUNTADMIN
- **C.** SYSADMIN
- **D.** SECURITYADNIN
- **E.** USERADMIN

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, D

</details>

---

### Question #206

What SQL command would be used to view all roles that were granted to USER1?

- **A.** show grants to user USER1;
- **B.** show grants user USER1;
- **C.** describe user USER1;
- **D.** show grants on user USER1;

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #207

Which ACCOUNT_USAGE views are used to evaluate the details of dynamic data masking? (Choose two.)

- **A.** ROLES
- **B.** POLICY_REFERENCES
- **C.** QUERY_HISTORY
- **D.** RESOURCE_MONITORS
- **E.** ACCESS_HISTORY
- **F.** MASKING_POLICIES

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, F

</details>

---

### Question #208

Which of the following are considerations when using a directory table when working with unstructured data? (Choose two.)

- **A.** A directory table is a separate database object.
- **B.** Directory tables store data file metadata.
- **C.** A directory table will be automatically added to a stage.
- **D.** Directory tables do not have their own grantable privileges.
- **E.** Directory table data cannot be refreshed manually.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, D

</details>

---

### Question #209

The first user assigned to a new account, ACCOUNTADMIN, should create at least one additional user with which administrative
privilege?

- **A.** USERADMIN
- **B.** PUBLIC
- **C.** ORGADMIN
- **D.** SYSADMIN

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #210

Which statement describes how Snowflake supports reader accounts?

- **A.** A reader account can consume data from the provider account that created it and combine it with its own data.
- **B.** A consumer needs to become a licensed Snowflake customer as data sharing is only supported between Snowflake accounts.
- **C.** The users in a reader account can query data that has been shared with the reader account and can perform DML tasks.
- **D.** The SHOW MANAGED ACCOUNTS command will view all the reader accounts that have been created for an account.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #211

How does Snowflake allow a data provider with an Azure account in central Canada to share data with a data consumer on AWS in
Australia?

- **A.** The data provider in Azure Central Canada can create a direct share to AWS Asia Pacific, if they are both in the same organization.
- **B.** The data consumer and data provider can form a Data Exchange within the same organization to create a share from Azure Central Canada to AWS Asia Pacific.
- **C.** The data provider uses the GET DATA work ow in the Snowflake Data Marketplace to create a share between Azure Central Canada and AWS Asia Pacific.
- **D.** The data provider must replicate the database to a secondary account in AWS Asia Pacific within the same organization then create a share to the data consumer's account

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #212

Which Snowflake objects can be shared with other Snowflake accounts? (Choose three.)

- **A.** Schemas
- **B.** Roles
- **C.** Secure Views
- **D.** Stored Procedures
- **E.** Tables
- **F.** Secure User-Defined Functions (UDFs)

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, E, F

</details>

---

### Question #213

Which Snowflake feature will allow small volumes of data to continuously load into Snowflake and will incrementally make the data
available for analysis?

- **A.** COPY INTO
- **B.** CREATE PIPE
- **C.** INSERT INTO
- **D.** TABLE STREAM

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #214

Which Snowflake partner specializes in data catalog solutions?

- **A.** Alation
- **B.** DataRobot
- **C.** dbt
- **D.** Tableau

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #215

Which of the following can be executed/called with Snowpipe?

- **A.** A User Defined Function (UDF)
- **B.** A stored procedure
- **C.** A single COPY_INTO statement
- **D.** A single INSERT_INTO statement

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #216

Which Snowflake objects will incur both storage and cloud compute charges? (Choose two.)

- **A.** Materialized view
- **B.** Sequence
- **C.** Secure view
- **D.** Transient table
- **E.** Clustered table

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A, E

</details>

---

### Question #217

What file formats does Snowflake support for loading semi-structured data? (Choose three.)

- **A.** TSV
- **B.** JSON
- **C.** PDF
- **D.** Avro
- **E.** Parquet
- **F.** JPEG

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, D, E

</details>

---

### Question #218

Which of the following statements about data sharing are true? (Choose two.)

- **A.** New objects created by a Data Provider are automatically shared with existing Data Consumers and Reader Accounts.
- **B.** All database objects can be included in a shared database.
- **C.** Reader Accounts are created by Data Providers.
- **D.** Shared databases are read-only.
- **E.** Reader Accounts are charged for warehouse usage.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, D

</details>

---

### Question #219

Credit charges for Snowflake virtual warehouses are calculated based on which of the following considerations? (Choose two.)

- **A.** The number of queries executed
- **B.** The number of active users assigned to the warehouse
- **C.** The size of the virtual warehouse
- **D.** The length of time the warehouse is running
- **E.** The duration of the queries that are executed

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, D

</details>

---

### Question #220

Which of the following are handled by the cloud services layer of the Snowflake architecture? (Choose two.)

- **A.** Query execution
- **B.** Data loading
- **C.** Time Travel data
- **D.** Security
- **E.** Authentication and access control

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D, E

</details>

---

### Question #221

What is a responsibility of SnowflakeÆs virtual warehouses?

- **A.** Infrastructure management
- **B.** Metadata management
- **C.** Query execution
- **D.** Query parsing and optimization
- **E.** Permanent storage of micro-partitions

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #222

What features does Snowflake Time Travel enable?

- **A.** Querying data-related objects that were created within the past 365 days
- **B.** Restoring data-related objects that have been deleted within the past 90 days
- **C.** Conducting point-in-time analysis for BI reporting
- **D.** Analyzing data usage/manipulation over all periods of time

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #223

Which of the following statements describes a schema in Snowflake?

- **A.** A logical grouping of objects that belongs to a single database
- **B.** A logical grouping of objects that belongs to multiple databases
- **C.** A named Snowflake object that includes all the information required to share a database
- **D.** A uniquely identified Snowflake account within a business entity

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #224

What is the recommended compressed file size range for continuous data loads using Snowpipe?

- **A.** 8-16 MB
- **B.** 16-24 MB
- **C.** 10-99 MB
- **D.** 100-250 MB

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #225

How long is Snowpipe data load history retained?

- **A.** As configured in the CREATE PIPE settings
- **B.** Until the pipe is dropped
- **C.** 64 days
- **D.** 14 days

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D

</details>

---

### Question #226

A company strongly encourages all Snowflake users to self-enroll in Snowflake's default Multi-Factor Authentication (MFA) service to
provide increased login security for users connecting to Snowflake.
Which application will the Snowflake users need to install on their devices in order to connect with MFA?

- **A.** Okta Verify
- **B.** Duo Mobile
- **C.** Microsoft Authenticator
- **D.** Google Authenticator

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #227

Which URL type allows users to access unstructured data without authenticating into Snowflake or passing an authorization token?

- **A.** Pre-signed URL
- **B.** Scoped URL
- **C.** Signed URL
- **D.** File URL

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #228

Where would a Snowflake user find information about query activity from 90 days ago?

- **A.** account_usage.query_history view
- **B.** account_usage.query_history_archive view
- **C.** information_schema.query_history view
- **D.** information_schema.query_history_by_session view

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #229

A marketing co-worker has requested the ability to change a warehouse size on their medium virtual warehouse called MKTG_WH.
Which of the following statements will accommodate this request?

- **A.** ALLOW RESIZE ON WAREHOUSE MKTG_WH TO USER MKTG_LEAD;
- **B.** GRANT MODIFY ON WAREHOUSE MKTG_WH TO ROLE MARKETING;
- **C.** GRANT MODIFY ON WAREHOUSE MKTG_WH TO USER MKTG_LEAD;
- **D.** GRANT OPERATE ON WAREHOUSE MKTG_WH TO ROLE MARKET;

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #230

Which of the following commands cannot be used within a reader account?

- **A.** CREATE SHARE
- **B.** ALTER WAREHOUSE
- **C.** DROP ROLE
- **D.** SHOW SCHEMAS
- **E.** DESCRIBE TABLE

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #231

Which TABLE function helps to convert semi-structured data to a relational representation?

- **A.** CHECK_JSON
- **B.** TO_JSON
- **C.** FLATTEN
- **D.** PARSE_JSON

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #232

Which query profile statistics help determine if efficient pruning is occurring? (Choose two.)

- **A.** Bytes sent over network
- **B.** Percentage scanned from cache
- **C.** Partitions total
- **D.** Bytes spilled to local storage
- **E.** Partitions scanned

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, E

</details>

---

### Question #233

What are the default Time Travel and Fail-safe retention periods for transient tables?

- **A.** Time Travel - 1 day, Fail-safe - 1 day
- **B.** Time Travel - 0 days, Fail-safe - 1 day
- **C.** Time Travel - 1 day, Failsafe - 0 days
- **D.** Transient tables are retained in neither Fail-safe nor Time Travel.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C

</details>

---

### Question #234

Which command is used to unload data from a Snowflake table into a file in a stage?

- **A.** COPY INTO
- **B.** GET
- **C.** WRITE
- **D.** EXTRACT INTO

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #235

What are advantages clones have over tables created with CREATE TABLE AS SELECT statement? (Choose two.)

- **A.** The clone always stays in sync with the original table.
- **B.** The clone has better query performance.
- **C.** The clone is created almost instantly.
- **D.** The clone will have time travel history from the original table.
- **E.** The clone saves space by not duplicating storage.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** C, E

</details>

---

### Question #236

How often are the Account and Table master keys automatically rotated by Snowflake?

- **A.** 30 Days
- **B.** 60 Days
- **C.** 90 Days
- **D.** 365 Days.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** A

</details>

---

### Question #237

Which privilege is required for a role to be able to resume a suspended warehouse if auto-resume is not enabled?

- **A.** USAGE
- **B.** OPERATE
- **C.** MONITOR
- **D.** MODIFY

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #238

Which statement MOST accurately describes clustering in Snowflake?

- **A.** The database ACCOUNTADMIN must define the clustering methodology for each Snowflake table.
- **B.** Clustering is the way data is grouped together and stored within Snowflake micro-partitions.
- **C.** The clustering key must be included in the COPY command when loading data into Snowflake.
- **D.** Clustering can be disabled within a Snowflake account.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B

</details>

---

### Question #239

Which of the following practices are recommended when creating a user in Snowflake? (Choose two.)

- **A.** Configure the user to be initially disabled.
- **B.** Force an immediate password change.
- **C.** Set a default role for the user.
- **D.** Set the number of minutes to unlock to 15 minutes.
- **E.** Set the user's access to expire within a specified timeframe.

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** B, C

</details>

---

### Question #240

Network policies can be applied to which of the following Snowflake objects? (Choose two.)

- **A.** Roles
- **B.** Databases
- **C.** Warehouses
- **D.** Users
- **E.** Accounts

<details>
<summary><strong>View Answer</strong></summary>
<br>

**Answer:** D, E

</details>

---


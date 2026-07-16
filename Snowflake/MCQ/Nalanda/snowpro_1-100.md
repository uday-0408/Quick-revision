# SnowPro Core Practice Questions --- Cleaned and Documentation-Checked

> Source text was OCR-damaged and contained several old SnowPro-era
> questions. I reconstructed the wording, normalized the options, and
> corrected answers that conflict with current Snowflake documentation
> as of **July 2026**.

> **Important:** These are study questions, not claimed to be current
> official exam questions. Questions tied to the retired Classic
> Console, old release behavior, old MFA guidance, or undocumented
> infrastructure/server counts have been modernized or flagged.

## Documentation cross-check used

-   Snowflake architecture and warehouses:
    https://docs.snowflake.com/en/user-guide/intro-key-concepts and
    https://docs.snowflake.com/en/user-guide/warehouses
-   Warehouse scaling policies:
    https://docs.snowflake.com/en/sql-reference/sql/create-warehouse
-   Micro-partitions and clustering:
    https://docs.snowflake.com/en/user-guide/tables-clustering-micropartitions
-   Persisted query results:
    https://docs.snowflake.com/en/user-guide/querying-persisted-results
-   Snowsight Query History:
    https://docs.snowflake.com/en/user-guide/ui-snowsight-activity
-   UDF languages:
    https://docs.snowflake.com/en/developer-guide/udf/udf-overview
-   Time Travel and Fail-safe:
    https://docs.snowflake.com/en/user-guide/data-availability
-   Snowflake editions:
    https://docs.snowflake.com/en/user-guide/intro-editions
-   PUT and internal-stage encryption:
    https://docs.snowflake.com/en/sql-reference/sql/put
-   Snowpipe:
    https://docs.snowflake.com/en/user-guide/data-load-snowpipe-intro
-   MFA guidance and rollout:
    https://docs.snowflake.com/en/user-guide/security-mfa and
    https://docs.snowflake.com/en/user-guide/security-mfa-rollout

------------------------------------------------------------------------

## Question 1

What Snowflake feature lets customers explicitly influence data
clustering beyond natural clustering?

-   A. Micro-partitions
-   B. Clustering keys
-   C. Key partitions
-   D. Clustered partitions

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 2

Which are valid multi-cluster warehouse scaling policies? (Choose two.)

-   A. Custom
-   B. Economy
-   C. Optimized
-   D. Standard

<details>
<summary>Show Answer</summary>

**Correct Answer: B, D**

</details>

------------------------------------------------------------------------

## Question 3

True or False: A single database can exist in more than one Snowflake
account.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 4

Which system role is recommended for creating and managing users and
roles?

-   A. SYSADMIN
-   B. SECURITYADMIN
-   C. PUBLIC
-   D. ACCOUNTADMIN

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 5

True or False: Bulk unloading from Snowflake supports using a SELECT
statement as the source.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 6

Which are the three types of internal stages? (Choose three.)

-   A. Named stage
-   B. User stage
-   C. Table stage
-   D. Schema stage

<details>
<summary>Show Answer</summary>

**Correct Answer: A, B, C**

</details>

------------------------------------------------------------------------

## Question 7

True or False: A customer using SnowSQL or native connectors cannot also
use the Snowflake web interface unless Snowflake Support explicitly
grants UI access.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 8

Where can account-level storage usage be monitored in Snowsight?

-   A. Data » Databases
-   B. Admin » Cost Management / usage views
-   C. INFORMATION_SCHEMA.ACCOUNT_USAGE_HISTORY
-   D. ACCOUNT_USAGE.ACCOUNT_USAGE_METRICS

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 9

Virtual warehouse credit consumption is primarily based on which two
factors? (Choose two.)

-   A. Number of users
-   B. Warehouse size
-   C. Amount of data processed
-   D. Number of active clusters

<details>
<summary>Show Answer</summary>

**Correct Answer: B, D**

</details>

------------------------------------------------------------------------

## Question 10

Which statement best describes clustering in Snowflake?

-   A. It describes how data is grouped within micro-partitions
-   B. An administrator must define clustering for every table
-   C. A clustering key must be specified in COPY
-   D. Clustering can be disabled account-wide

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 11

True or False: COPY must always specify a named file format object.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 12

Which command sets the virtual warehouse for the current session?

-   A. COPY WAREHOUSE FROM `<config_file>`{=html}
-   B. SET WAREHOUSE = `<warehouse_name>`{=html}
-   C. USE WAREHOUSE `<warehouse_name>`{=html}
-   D. USE VIRTUAL WAREHOUSE `<warehouse_name>`{=html}

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

</details>

------------------------------------------------------------------------

## Question 13

Which objects can be cloned? (Choose four from these options.)

-   A. Tables
-   B. Named file formats
-   C. Schemas
-   D. Shares
-   E. Databases

<details>
<summary>Show Answer</summary>

**Correct Answer: A, B, C, E**

</details>

------------------------------------------------------------------------

## Question 14

Which Snowflake object can enforce credit-consumption limits?

-   A. Account Tracking
-   B. Resource Monitor
-   C. Warehouse Limit Parameter
-   D. Credit Consumption Tracker

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 15

Snowflake is primarily designed for which workload characteristics?
(Choose two.)

-   A. OLAP / analytics workloads
-   B. OLTP workloads
-   C. Concurrent workloads
-   D. On-premises workloads

<details>
<summary>Show Answer</summary>

**Correct Answer: A, C**

</details>

------------------------------------------------------------------------

## Question 16

What are the three main layers of Snowflake architecture? (Choose
three.)

-   A. Compute
-   B. Tri-Secret Secure
-   C. Storage
-   D. Cloud Services

<details>
<summary>Show Answer</summary>

**Correct Answer: A, C, D**

</details>

------------------------------------------------------------------------

## Question 17

Why would a customer resize a warehouse from Small to Medium?

-   A. To add more clusters for concurrency
-   B. To add more users
-   C. To handle workload fluctuations automatically
-   D. To provide more compute for a more complex workload

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 18

True or False: Reader accounts create no compute cost for the data
provider.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

> **Documentation update:** Corrected: reader-account compute is paid by
> the provider account, so the provider can incur compute cost.

</details>

------------------------------------------------------------------------

## Question 19

Which clients can use MFA-based authentication when supported and
configured? (Choose all that apply.)

-   A. JDBC
-   B. SnowSQL
-   C. Snowsight
-   D. ODBC
-   E. Python connector

<details>
<summary>Show Answer</summary>

**Correct Answer: A, B, C, D, E**

</details>

------------------------------------------------------------------------

## Question 20

True or False: Snowflake charges a storage premium specifically because
data is semi-structured.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 21

Which statements describe benefits of separating compute and storage?
(Choose all that apply.)

-   A. Storage and compute must grow together
-   B. Storage can expand without adding compute
-   C. Compute can scale without adding storage
-   D. Multiple compute clusters can access shared stored data without
    storage contention

<details>
<summary>Show Answer</summary>

**Correct Answer: B, C, D**

</details>

------------------------------------------------------------------------

## Question 22

True or False: Snowflake can unload table/query data to JSON and Parquet
formats.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 23

In which architecture layer does Snowflake manage metadata and
optimization statistics?

-   A. Storage
-   B. Compute
-   C. Database
-   D. Cloud Services

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 24

True or False: Customers can directly delete data from Fail-safe.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 25

True or False: Snowflake was built from the ground up for the cloud
rather than on an existing database or Hadoop platform.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 26

Which statements about virtual warehouses are true? (Choose all that
apply.)

-   A. Warehouse size can be changed after creation
-   B. A running warehouse can be resized
-   C. A warehouse can auto-suspend after inactivity
-   D. A warehouse can auto-resume when a statement needs compute

<details>
<summary>Show Answer</summary>

**Correct Answer: A, B, C, D**

</details>

------------------------------------------------------------------------

## Question 27

Which statements about PUT are true? (Choose two.)

-   A. It automatically creates a file
-   B. It automatically uses the last-created stage
-   C. By default it compresses eligible files using gzip
-   D. Files on internal stages are encrypted

<details>
<summary>Show Answer</summary>

**Correct Answer: C, D**

</details>

------------------------------------------------------------------------

## Question 28

Which table type exists only for the current session?

-   A. Temporary
-   B. Transient
-   C. Provisional
-   D. Permanent

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 29

Which interfaces can create or manage virtual warehouses?

-   A. Snowsight
-   B. SQL commands
-   C. Tools that issue supported Snowflake SQL/API operations
-   D. All of the above

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 30

What happens when a pipe is recreated with CREATE OR REPLACE PIPE?

-   A. The new pipe has no retained load history for duplicate detection
-   B. REFRESH is automatically TRUE
-   C. Previously loaded files are always ignored
-   D. All of the above

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 31

What is the minimum Snowflake edition generally intended for customers
storing highly sensitive regulated data requiring enhanced security
capabilities?

-   A. Standard
-   B. Premier
-   C. Enterprise
-   D. Business Critical

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 32

Which three table types exist in Snowflake? (Choose three.)

-   A. Temporary
-   B. Transient
-   C. Provisional
-   D. Permanent

<details>
<summary>Show Answer</summary>

**Correct Answer: A, B, D**

</details>

------------------------------------------------------------------------

## Question 33

True or False: Snowpipe's REST API can reference only external stages.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 34

True or False: A third-party tool that supports standard JDBC/ODBC but
has no Snowflake-specific driver can connect without using a Snowflake
JDBC/ODBC driver.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 35

True or False: Data can be loaded without creating a named FILE FORMAT
object.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 36

True or False: A table can be queried only by the warehouse that loaded
it.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 37

Which are recommended data-loading practices? (Choose three.)

-   A. VARIANT JSON null is distinct from SQL NULL
-   B. Perform frequent single-row DML whenever possible
-   C. Validate data before loading into the target table
-   D. Use staging tables when useful for MERGE-based processing

<details>
<summary>Show Answer</summary>

**Correct Answer: A, C, D**

</details>

------------------------------------------------------------------------

## Question 38

Which statements about micro-partitions are true? (Choose two.)

-   A. They contain roughly 50--500 MB of uncompressed data
-   B. Compression occurs only when COMPRESS=TRUE is set on the table
-   C. They are immutable
-   D. They are encrypted only in Enterprise Edition and above

<details>
<summary>Show Answer</summary>

**Correct Answer: A, C**

> **Documentation update:** Corrected outdated size wording: current
> docs describe micro-partitions as 50--500 MB of uncompressed data;
> storage is compressed automatically.

</details>

------------------------------------------------------------------------

## Question 39

True or False: Query IDs are unique identifiers that can be provided to
Snowflake Support when troubleshooting queries.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 40

A deterministic query is executed and its result is persisted. Which
statements are true? (Choose two.)

-   A. Snowflake always reuses the result for 24 hours
-   B. The same query can reuse the result when reuse conditions are met
    and underlying data has not changed
-   C. The result is reused even if underlying data changed
-   D. Reusing a persisted result resets its 24-hour retention window,
    up to the documented maximum lifetime

<details>
<summary>Show Answer</summary>

**Correct Answer: B, D**

> **Documentation update:** Clarified persisted-result reuse: reuse is
> conditional; each successful reuse resets the 24-hour retention
> period, up to 31 days from initial execution.

</details>

------------------------------------------------------------------------

## Question 41

Increasing MAX_CLUSTER_COUNT for a multi-cluster warehouse is an example
of:

-   A. Rhythmic scaling
-   B. Scaling max
-   C. Scaling out
-   D. Scaling up

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

</details>

------------------------------------------------------------------------

## Question 42

Which statement best describes Snowflake tables?

-   A. They are logical representations of underlying Snowflake-managed
    storage
-   B. They are user-managed physical files
-   C. Every table requires a clustering key
-   D. Tables are owned directly by users rather than roles

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 43

Which migration item generally does not apply to Snowflake?

-   A. Migrate data
-   B. Migrate schemas
-   C. Migrate indexes
-   D. Build the data pipeline

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

</details>

------------------------------------------------------------------------

## Question 44

Which table types help reduce storage costs for short-lived/transitory
data? (Choose two.)

-   A. Temporary
-   B. Transient
-   C. Provisional
-   D. Permanent

<details>
<summary>Show Answer</summary>

**Correct Answer: A, B**

</details>

------------------------------------------------------------------------

## Question 45

Which statement correctly describes micro-partition size?

-   A. Exactly 8 GB compressed
-   B. Approximately 16 MB compressed
-   C. Approximately 50--500 MB uncompressed, with smaller compressed
    storage
-   D. Exactly 4 TB

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

> **Documentation update:** Corrected outdated '16 MB maximum compressed
> size' wording.

</details>

------------------------------------------------------------------------

## Question 46

Which are current top-level Snowsight navigation areas relevant to these
functions? (Choose three.)

-   A. Data
-   B. Tables
-   C. Compute
-   D. Projects / Worksheets

<details>
<summary>Show Answer</summary>

**Correct Answer: A, C, D**

> **Documentation update:** Modernized Classic Console navigation
> wording to current Snowsight areas.

</details>

------------------------------------------------------------------------

## Question 47

Which Snowflake data type is recommended for semi-structured data such
as JSON?

-   A. VARCHAR
-   B. RAW
-   C. LOB
-   D. VARIANT

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 48

Which statements best describe Snowflake releases? (Choose two.)

-   A. Snowflake continuously delivers releases/updates as a managed
    service
-   B. Customers must manually install monthly releases
-   C. Service upgrades are designed to be transparent to running
    customer workloads
-   D. Every customer receives a mandatory downtime window

<details>
<summary>Show Answer</summary>

**Correct Answer: A, C**

> **Documentation update:** Modernized release wording; the old
> question's release-cadence phrasing is no longer a good certification
> question.

</details>

------------------------------------------------------------------------

## Question 49

Which are common zero-copy cloning use cases? (Choose three.)

-   A. Rapid Dev/Test/QA provisioning
-   B. Logical snapshots or backup-like copies
-   C. Point-in-time clones using Time Travel
-   D. Direct query performance optimization

<details>
<summary>Show Answer</summary>

**Correct Answer: A, B, C**

</details>

------------------------------------------------------------------------

## Question 50

Warehouse compute resources roughly double with each size step. If a
Small warehouse is treated as 2 units, how many units does Medium have?

-   A. 4
-   B. 16
-   C. 32
-   D. 128

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

> **Documentation update:** Corrected answer from the OCR/dump: Medium
> is 4 units if Small is 2; warehouse sizes roughly double compute per
> size step.

</details>

------------------------------------------------------------------------

## Question 51

True or False: A consumer of a direct share can re-share that imported
database to other consumers.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 52

Which statements about network policies are true? (Choose two.)

-   A. Network policies are available across Snowflake editions
-   B. They are Business Critical-only
-   C. They can allow or block network identifiers such as IP ranges
-   D. They are activated with ALTER DATABASE

<details>
<summary>Show Answer</summary>

**Correct Answer: A, C**

</details>

------------------------------------------------------------------------

## Question 53

True or False: Snowflake charges a separate fee to a provider for each
share object created.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 54

How long is a persisted query result normally retained after it is
generated or successfully reused, assuming reuse remains valid?

-   A. 1 hour
-   B. 3 hours
-   C. 12 hours
-   D. 24 hours

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 55

A role owns two tables and is dropped. What happens to the owned tables?

-   A. They become orphaned
-   B. Ownership transfers to the user
-   C. Ownership always transfers to SYSADMIN
-   D. The DROP ROLE operation requires ownership handling; current
    Snowflake behavior should be evaluated via dependent
    privileges/OWNERSHIP rules

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

> **Documentation update:** Reworded because role dropping/OWNERSHIP
> behavior is more nuanced than the old dump's simplistic answer.

</details>

------------------------------------------------------------------------

## Question 56

Which client downloads were historically exposed in the classic
Snowflake web interface? (Choose two.)

-   A. SnowSQL
-   B. ODBC driver
-   C. Hive
-   D. None

<details>
<summary>Show Answer</summary>

**Correct Answer: A, B**

> **Documentation update:** Marked as historical Classic Console
> wording; not a current Snowsight navigation question.

</details>

------------------------------------------------------------------------

## Question 57

Which DML-style command is not a Snowflake SQL command?

-   A. UPSERT
-   B. MERGE
-   C. UPDATE
-   D. TRUNCATE TABLE

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 58

Which statement about zero-copy cloning is true?

-   A. A clone immediately duplicates all storage
-   B. Every cloned object inherits all source privileges
-   C. Cloning requires a separate feature license
-   D. At creation, clone and source initially share underlying
    micro-partitions

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 59

True or False: When a user creates a role, the user personally owns the
role until ownership is transferred.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 60

How much query history does the Snowsight Query History page display?

-   A. 60 minutes
-   B. 24 hours
-   C. 14 days
-   D. 90 days
-   E. 1 year

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

</details>

------------------------------------------------------------------------

## Question 61

How do you configure a multi-cluster warehouse for Auto-scale mode?

-   A. Set only MAX_CLUSTER_COUNT
-   B. Set warehouse TYPE
-   C. Set MIN_CLUSTER_COUNT and MAX_CLUSTER_COUNT to the same value
-   D. Set MIN_CLUSTER_COUNT and MAX_CLUSTER_COUNT to different values,
    with max greater than min

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 62

Which term best describes Snowflake's architecture?

-   A. Columnar shared-nothing
-   B. Shared disk
-   C. Multi-cluster, shared data
-   D. Cloud-native shared memory

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

</details>

------------------------------------------------------------------------

## Question 63

Which are warehouse configuration options? (Choose two.)

-   A. AUTO_DROP
-   B. AUTO_RESIZE
-   C. AUTO_RESUME
-   D. AUTO_SUSPEND

<details>
<summary>Show Answer</summary>

**Correct Answer: C, D**

</details>

------------------------------------------------------------------------

## Question 64

Warehouse AUTO_SUSPEND and AUTO_RESUME settings apply to:

-   A. Only the primary cluster
-   B. The warehouse
-   C. The database
-   D. Individual queries

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 65

Fail-safe is unavailable for which table types? (Choose two.)

-   A. Temporary
-   B. Transient
-   C. Provisional
-   D. Permanent

<details>
<summary>Show Answer</summary>

**Correct Answer: A, B**

</details>

------------------------------------------------------------------------

## Question 66

Which object is not protected by Time Travel?

-   A. Tables
-   B. Schemas
-   C. Databases
-   D. Stages

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 67

True or False: Micro-partition metadata allows some metadata-only
operations/queries to be answered without warehouse compute.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 68

Which operations are generally non-blocking with respect to concurrent
table reads because of Snowflake's concurrency architecture? (Choose
two.)

-   A. UPDATE
-   B. INSERT
-   C. MERGE
-   D. COPY INTO table

<details>
<summary>Show Answer</summary>

**Correct Answer: B, D**

</details>

------------------------------------------------------------------------

## Question 69

Which statements about Snowpipe are true? (Choose two.)

-   A. It can load only from internal stages
-   B. Every COPY INTO option is supported in a pipe definition
-   C. Snowflake manages the compute used by classic Snowpipe
-   D. Snowpipe tracks files already loaded

<details>
<summary>Show Answer</summary>

**Correct Answer: C, D**

> **Documentation update:** Updated MFA guidance: current Snowflake
> guidance/rollout applies MFA to human password users, not only
> ACCOUNTADMIN.

</details>

------------------------------------------------------------------------

## Question 70

Which users should use MFA under current Snowflake security guidance?

-   A. Only SECURITYADMIN and ACCOUNTADMIN
-   B. Only SYSADMIN
-   C. Only ACCOUNTADMIN
-   D. All human users; Snowflake is phasing out single-factor password
    sign-ins

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 71

When can a virtual warehouse begin running queries?

-   A. Only 12 AM--5 AM
-   B. Only in administrator-defined time slots
-   C. When provisioning/resume is complete enough to accept work
-   D. After replication

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

</details>

------------------------------------------------------------------------

## Question 72

True or False: Users can automatically see query result sets of other
users merely because they share the same role.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 73

True or False: A user must choose the specific cluster that runs a query
in a multi-cluster warehouse.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 74

True or False: Pipes can be paused/suspended and resumed.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

> **Documentation update:** Corrected outdated UDF language answer:
> current UDF handlers support Java, JavaScript, Python, Scala, and SQL.

</details>

------------------------------------------------------------------------

## Question 75

Which languages are currently supported for Snowflake UDF handlers?
(Choose all that apply.)

-   A. Java
-   B. JavaScript
-   C. SQL
-   D. Python
-   E. Scala

<details>
<summary>Show Answer</summary>

**Correct Answer: A, B, C, D, E**

</details>

------------------------------------------------------------------------

## Question 76

When might disabling AUTO_SUSPEND be reasonable? (Choose two.)

-   A. For sporadic workloads throughout a day
-   B. For a steady workload
-   C. When avoiding resume latency is important
-   D. Because AUTO_RESUME does not exist

<details>
<summary>Show Answer</summary>

**Correct Answer: B, C**

> **Documentation update:** Corrected: bulk COPY from an internal stage
> is also a valid loading method.

</details>

------------------------------------------------------------------------

## Question 77

Which are valid ways to load data into a Snowflake table? (Choose all
that apply.)

-   A. COPY INTO from a stage
-   B. Continuous loading with Snowpipe
-   C. Snowsight load-data workflow
-   D. Bulk COPY from an internal stage

<details>
<summary>Show Answer</summary>

**Correct Answer: A, B, C, D**

</details>

------------------------------------------------------------------------

## Question 78

When does AUTO_SUSPEND suspend a warehouse?

-   A. When all sessions terminate
-   B. Immediately after the last query
-   C. When there are no logins
-   D. After the configured period of warehouse inactivity

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 79

True or False: Snowflake MFA works only with SSO.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 80

Which factors influence how many queries a warehouse can process
concurrently? (Choose two.)

-   A. Query complexity/resource demand
-   B. A fixed account-wide concurrency number
-   C. Data volume/resource requirements of queries
-   D. The client tool name

<details>
<summary>Show Answer</summary>

**Correct Answer: A, C**

</details>

------------------------------------------------------------------------

## Question 81

Which statements about VALIDATION_MODE are true? (Choose two.)

-   A. It is a CREATE STAGE option
-   B. It is an option of COPY INTO
    ```{=html}
    <table>
    ```
-   C. It validates while completing the load
-   D. It validates input without loading data and returns validation
    results/errors

<details>
<summary>Show Answer</summary>

**Correct Answer: B, D**

</details>

------------------------------------------------------------------------

## Question 82

Which privileges are required to create a task?

-   A. Only global CREATE TASK
-   B. Only ACCOUNTADMIN can create tasks
-   C. No task-specific privileges are needed
-   D. CREATE TASK on the schema plus required object privileges; task
    execution also depends on EXECUTE TASK or serverless task privileges
    as applicable

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 83

Which three qualities are commonly sought in an enterprise data
warehouse? (Choose three.)

-   A. On-premises availability
-   B. Simplicity
-   C. Open-source base
-   D. Concurrency
-   E. Performance

<details>
<summary>Show Answer</summary>

**Correct Answer: B, D, E**

</details>

------------------------------------------------------------------------

## Question 84

True or False: Some metadata-only queries can be answered without an
active warehouse.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 85

Scaling out by adding clusters to a multi-cluster warehouse primarily
improves:

-   A. Concurrency / queued workload handling
-   B. Single-query performance
-   C. Storage capacity
-   D. Time Travel retention

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

> **Documentation update:** Corrected outdated answer: Secure Data
> Sharing is available in Standard Edition.

</details>

------------------------------------------------------------------------

## Question 86

What is the minimum Snowflake edition that supports Secure Data Sharing?

-   A. Standard
-   B. Premier
-   C. Enterprise
-   D. Business Critical

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 87

True or False: Snowsight worksheets can use different database, schema,
and warehouse contexts.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 88

True or False: Snowflake can query files in an external stage directly
without first loading them into a table.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

> **Documentation update:** Corrected FLATTEN: it is primarily for
> semi-structured values such as VARIANT/OBJECT/ARRAY.

</details>

------------------------------------------------------------------------

## Question 89

The FLATTEN table function is primarily used with which data?

-   A. Structured relational data only
-   B. Semi-structured data
-   C. Both equally as its primary purpose
-   D. None

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

> **Documentation update:** Clarified COPY INTO: warehouse requirements
> depend on the COPY form/service; classic Snowpipe is Snowflake-managed
> compute.

</details>

------------------------------------------------------------------------

## Question 90

True or False: A user-managed virtual warehouse is required for every
COPY INTO statement.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 91

True or False: Private connectivity such as AWS PrivateLink can provide
private network connectivity between customer networks and Snowflake
without traversing the public internet.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 92

True or False: Snowflake maintains metadata about columns in
micro-partitions for pruning and optimization.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 93

True or False: It is best practice to define a clustering key on every
table.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

</details>

------------------------------------------------------------------------

## Question 94

Which statement about Snowflake is true?

-   A. It was built specifically for the cloud
-   B. It was an on-premises database later ported to cloud
-   C. It is primarily a hybrid on-prem/cloud database
-   D. It was built on Hadoop
-   E. It is based on Oracle architecture

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 95

What is the minimum edition that provides multi-cluster warehouses and
extended Time Travel up to 90 days?

-   A. Standard
-   B. Premier
-   C. Enterprise
-   D. Business Critical

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

> **Documentation update:** Marked the old fixed share-limit question as
> outdated and unsuitable without a current limits reference.

</details>

------------------------------------------------------------------------

## Question 96

How many shares can a data consumer consume?

-   A. 10
-   B. 50
-   C. 100, hard limit
-   D. This old fixed-limit question is outdated; current sharing limits
    are governed by current Snowflake limits and feature behavior rather
    than this exam's 'Unlimited' claim

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 97

What is the lowest edition that supports Time Travel retention up to 90
days?

-   A. Standard
-   B. Premier
-   C. Enterprise
-   D. Business Critical

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

</details>

------------------------------------------------------------------------

## Question 98

Which statements about schemas are true? (Choose two.)

-   A. A schema contains databases
-   B. A database can contain schemas
-   C. A schema logically groups database objects
-   D. A schema is contained in a warehouse

<details>
<summary>Show Answer</summary>

**Correct Answer: B, C**

</details>

------------------------------------------------------------------------

## Question 99

True or False: A virtual warehouse can be resized while queries are
running.

-   A. True
-   B. False

<details>
<summary>Show Answer</summary>

**Correct Answer: A**

</details>

------------------------------------------------------------------------

## Question 100

What is the most granular object level at which
DATA_RETENTION_TIME_IN_DAYS can be set among these options?

-   A. Account
-   B. Database
-   C. Schema
-   D. Table

<details>
<summary>Show Answer</summary>

**Correct Answer: D**

</details>

------------------------------------------------------------------------

## Question 101

Which statement about Snowflake micro-partitioning is true?

-   A. It inherently introduces data skew
-   B. It requires a partitioning scheme up front
-   C. It is automatic and initially reflects the ordering of data as it
    is loaded/inserted
-   D. It can be disabled account-wide

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

</details>

------------------------------------------------------------------------

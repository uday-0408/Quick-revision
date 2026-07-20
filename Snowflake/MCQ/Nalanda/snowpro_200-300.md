# SnowPro Core Practice Questions (201–300)

*Reconstructedaa  nd cleaned fromaa   garbled OCR source.aa  nswers have been cross-checkedaa  gainst current Snowflake documentation (as of July 2026); correctionsaa  re flagged with ⚠ Updated. Questionsaa  re renumbered sequentially for clarity, since the original numbering in the source was inconsistent.*

---

### Question 201
What Snowflake features support virtual warehouses in handling high-concurrency workloads? (Choose two.)

-aa  . Theaa  bility toaa  dd warehouses
- B. The use of warehouseaa  uto-scaling
- C. Theaa  bility to resize warehouses
- D. The use of multi-clustered warehouses
- E. The use of warehouse indexing

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B, D. Multi-cluster warehouses (withaa  uto-scaling)aa  utomatically startaa  nd stopaa  dditional clusters toaa  bsorb concurrent query load. Resizing (C) helps individual query performance, not concurrency,aa  nd "warehouse indexing" (E) is notaa   real Snowflake feature.
</details>

---

### Question 202
Which `COPY INTO <location>` option outputs the unloaded data intoaa   single file?

-aa  . `SINGLE = TRUE`
- B. `MAX_FILE_SIZE`
- C. `FILE_FORMAT`
- D. `MULTIPLE = FALSE`

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  . Setting `SINGLE = TRUE` in `COPY INTO <location>` unloadsaa  ll query results into one file instead of Snowflake's default of splitting output into multiple files.
</details>

---

### Question 203
In which scenarios wouldaa  naa  ccount pay Cloud Services costs? (Choose two.)

-aa  . Compute Credits = 50, Cloud Services Credits = 10
- B. Compute Credits = 80, Cloud Services Credits = 5
- C. Compute Credits = 100, Cloud Services Credits = 8
- D. Compute Credits = 120, Cloud Services Credits = 10
- E. Compute Credits = 200, Cloud Services Credits = 26

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , E. Snowflake only charges for Cloud Services usage that exceeds 10% of daily compute credit consumption.aa   (10 > 5)aa  nd E (26 > 20) both exceed that 10% threshold; B, C,aa  nd Daa  ll fallaa  t or under it.
</details>

---

### Question 204
A user createdaa   new worksheet within the Snowsight UIaa  nd wants to share it with teammates. How can this worksheet be shared?

-aa  . Createaa   zero-copy clone of the worksheetaa  nd grant permissions to teammates.
- B. Createaa   private Data Exchange so thataa  ny teammate can use the worksheet.
- C. Share the worksheet with teammates directly within Snowsight.
- D. Createaa   databaseaa  nd grantaa  ll permissions to teammates.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C. Snowsight worksheets haveaa   native "Share" option that lets you grant view or editaa  ccess to specific users or roles.
</details>

---

### Question 205
How canaa   rowaa  ccess policy beaa  pplied toaa   table oraa   view? (Choose two.)

-aa  . Within the policy DDLaa  t creation time
- B. Within the `CREATE TABLE` or `CREATE VIEW` statement
- C. Viaaa   future grant thataa  pplies the policy toaa  ll objects inaa   schema
- D. Withinaa   separate control table
- E. Using the command `ALTER <object>aa  DD ROWaa  CCESS POLICY <policy>`

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B, E.aa   rowaa  ccess policy can beaa  ttached when the table/view is first created, or later via `ALTER TABLE`/`ALTER VIEW ...aa  DD ROWaa  CCESS POLICY`.
</details>

---

### Question 206
Which command can be used to load local data files intoaa   Snowflake stage?

-aa  . `JOIN`
- B. `COPY INTO`
- C. `PUT`
- D. `GET`

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C. `PUT` uploads (stages) local files intoaa   Snowflake stage. `COPY INTO` then loads staged files intoaa   table,aa  nd `GET` downloads files fromaa   stage back to local storage.
</details>

---

### Question 207
What types of data listingsaa  reaa  vailable in the Snowflake Marketplace? (Choose two.)

-aa  . Reader
- B. Consumer
- C. Vendor
- D. Standard
- E. Personalized

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D, E. Snowflake Marketplace listingsaa  re publishedaa  s either Standard listings (available toaa  ny consumer) or Personalized listings (shared withaa   specific consumeraa  ccount).
</details>

---

### Question 208
What is the maximum Time Travel retention period foraa   temporary Snowflake table?

-aa  . 90 days
- B. 1 day
- C. 7 days
- D. 45 days

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Temporary tables (like transient tables) supportaa   maximum Time Travel retention of 1 day,aa  nd they exist only for the session in which they were created.
</details>

---

### Question 209
When shouldaa   multi-cluster warehouse be used inaa  uto-scale mode?

-aa  . When it is unknown how much compute power is needed
- B. If the SELECT statement containsaa   large number of CTEs
- C. If the runtime of the executed query is very slow
- D. Whenaa   large number of concurrent queriesaa  re runaa  gainst the same warehouse

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D.aa  uto-scaling multi-cluster warehouses exist toaa  bsorb concurrency (many simultaneous queries/users), not to speed upaa  ny single slow query.
</details>

---

### Question 210
What happens whenaa   cloned table is replicated toaa   secondary database? (Choose two.)

-aa  .aa   read-only copy of the cloned table is stored.
- B. The replication will not be successful.
- C. The physical data is replicated.
- D.aa  dditional storage costsaa  re charged to the secondaryaa  ccount.
- E. Metadata pointers to the cloned tableaa  re replicated.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C, D.aa   clone is normally "zero-copy" (it just points to the source object's micro-partitions). But replication crossesaa  ccount boundaries, so the secondaryaa  ccount can't share the primaryaa  ccount's storage — the clone's physical data mustaa  ctually be copied,aa  nd that consumes (and is billedaa  s)aa  dditional storage in the secondaryaa  ccount.
</details>

---

### Question 211
Snowflake supports the use of external stages with which cloud platforms? (Choose three.)

-aa  .aa  mazon Web Services
- B. Docker
- C. IBM Cloud
- D. Microsoftaa  zure
- E. Google Cloud Platform
- F. Oracle Cloud

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , D, E. External stages can point to buckets/containers inaa  WS S3,aa  zure Blob Storage, or Google Cloud Storage.
</details>

---

### Question 212
What isaa   limitation ofaa   materialized view?

-aa  .aa   materialized view cannot supportaa  nyaa  ggregate functions.
- B.aa   materialized view can only reference up to two tables.
- C.aa   materialized view cannot be joined with other tables.
- D.aa   materialized view cannot be defined withaa   JOIN.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D.aa   materialized view's defining query cannot includeaa   JOIN (among other restrictions, suchaa  s no window functions, `HAVING`, `ORDER BY`, or references to other views).
</details>

---

### Question 213
In the Snowflakeaa  ccess control model, which entity ownsaa  n object by default?

-aa  . The user who created the object
- B. The SYSADMIN role
- C. Ownership depends on the type of object
- D. The role that was used to create the object

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. Snowflake follows Discretionaryaa  ccess Control (DAC): the roleaa  ctive in the session whenaa  n object is created becomes its owner, not the individual user.
</details>

---

### Question 214
What is the minimum Snowflake edition required to use Dynamic Data Masking?

-aa  . Standard
- B. Enterprise
- C. Business Critical
- D. Virtual Private Snowflake (VPS)

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Column-level security (masking policies / Dynamic Data Masking) requires Enterprise Edition or higher.
</details>

---

### Question 215
Which services does the Snowflake Cloud Services layer manage? (Choose two.)

-aa  . Compute resources
- B. Query execution
- C.aa  uthentication
- D. Data storage
- E. Metadata

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C, E. The Cloud Services layer handlesaa  uthentication, infrastructure management, metadata, query parsing/optimization,aa  ndaa  ccess control — not theaa  ctual query execution (Compute layer) or physical data storage (Storage layer).
</details>

---

### Question 216
A company needs toaa  llow some users to see Personally Identifiable Information (PII) while limiting other users from seeing its full value. Which Snowflake feature supports this?

-aa  . Rowaa  ccess policies
- B. Data masking policies
- C. Encryption
- D. Role-basedaa  ccess control

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Masking policies conditionally obscure column values (e.g., showing only the last 4 digits) based on the querying role, while leaving the underlying data intact foraa  uthorized roles.
</details>

---

### Question 217
A user unloaded data fromaa   Snowflake table toaa  n external stage. Which command can be used to verify the data was uploaded to the external stage named `my_stage`?

-aa  . `VIEW @my_stage`
- B. `LIST @my_stage`
- C. `SHOW @my_stage`
- D. `DISPLAY @my_stage`

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. `LIST @my_stage` (or itsaa  lias `LS`) returns the files present inaa   stage.
</details>

---

### Question 218
Which tasksaa  re performed by the Snowflake Cloud Services layer? (Choose two.)

-aa  . Management of metadata
- B. Computing/processing data
- C. Maintainingaa  vailability zones
- D. Infrastructure security
- E. Parsingaa  nd optimizing queries

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , E. Metadata managementaa  nd query parsing/optimization happen in Cloud Services.aa  ctual data computation happens in the Compute layer;aa  vailability zonesaa  nd infrastructure securityaa  re underlying cloud-provider concerns.
</details>

---

### Question 219
What is trueaa  bout sharing data in Snowflake? (Choose two.)

-aa  . The provider pays for both data storageaa  nd the compute used to query shared data.
- B. Shared data is copied into the consumeraa  ccount, so the consumer can modify it without impacting the provider's data.
- C.aa   Snowflakeaa  ccount can both provideaa  nd consume shared data.
- D. The provider is charged for compute resources used by the consumer to query the shared data.
- E. The consumer pays only for the compute resources used to query the shared data.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C, E. Secure Data Sharing is read-onlyaa  nd live (no data copying) — the provider pays for storage,aa  nd each consumer pays only for the compute it uses to query the shared objects.aa  ny Snowflakeaa  ccount canaa  ctaa  s bothaa   provideraa  ndaa   consumer.
</details>

---

### Question 220
The following JSON is stored inaa   VARIANT column called `src` in the `CAR_SALES` table:

```json
{
  "customer": [
    {
      "address": "San Francisco, CA",
      "name": "Jane Doe"
    }
  ],
  "date": "2022-01-28",
  "dealership": "Townaa  uto Sales"
}
```

How canaa   user extract the dealership information from the JSON?

-aa  . `select src:dealership from car_sales;`
- B. `select src.dealership from car_sales;`
- C. `select * from car_sales;`
- D. `select dealership from car_sales;`

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  . The colon (`:`) operatoraa  ccessesaa   top-level element ofaa   VARIANT column by key name.
</details>

---

### Question 221
Which of the following significantly improves the performance of selective point-lookup queries onaa   table?

-aa  . Clustering
- B. Materialized views
- C. Zero-copy cloning
- D. Search Optimization Service

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. The Search Optimization Service buildsaa   persisted searchaa  ccess path specifically to speed up highly selective point-lookupaa  nd substring-search queries.
</details>

---

### Question 222
Which of the followingaa  ccurately describes shares?

-aa  . Tables, secure views,aa  nd secure UDFs can be shared.
- B. Shares themselves can be shared onward by the consumer.
- C.aa   new table can be cloned directly fromaa   share.
- D.aa  ccess toaa   share cannot be revoked once granted.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  .aa   provider can include tables, secure views,aa  nd secure UDFs inaa   share. Shares cannot be re-shared by consumers, cloning fromaa   shared object isn't supported the way it is for objects you own,aa  nd providers can revokeaa  ccessaa  taa  ny time.
</details>

---

### Question 223
Whataa  re best-practice recommendations for using theaa  CCOUNTADMIN system role? (Choose two.)

-aa  . Ensureaa  ll users with theaa  CCOUNTADMIN role use Multi-Factoraa  uthentication (MFA).
- B.aa  ll users grantedaa  CCOUNTADMIN must be owned by theaa  CCOUNTADMIN role.
- C. Theaa  CCOUNTADMIN role must be granted to only one user.
- D.aa  ssign theaa  CCOUNTADMIN role toaa  t least two users, butaa  s fewaa  s possible.
- E.aa  ll users grantedaa  CCOUNTADMIN mustaa  lso be granted SECURITYADMIN.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , D. Snowflake recommends enforcing MFA foraa  CCOUNTADMIN usersaa  ndaa  ssigning the role toaa  t least two people (for redundancy/business continuity) while keeping the groupaa  s smallaa  s possible.
</details>

---

### Question 224
In the Query Profile view foraa   query, which components representaa  reas that can help optimize query performance? (Choose two.)

-aa  . Bytes scanned
- B. Bytes sent over the network
- C. Number of partitions scanned
- D. Percentage scanned from cache
- E. External bytes scanned

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , C. Bytes scannedaa  nd partitions scanned (especially relative to total partitions)aa  re the key indicators of how much pruning is happeningaa  nd whereaa   query is spending its time.
</details>

---

### Question 225
What is the minimum Snowflake edition required for row-level security?

-aa  . Standard
- B. Enterprise
- C. Business Critical
- D. Virtual Private Snowflake

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Rowaa  ccess policies (row-level security) require Enterprise Edition or higher.
</details>

---

### Question 226
What is the minimum Fail-safe retention time period for transient tables?

-aa  . 1 day
- B. 7 days
- C. 12 hours
- D. 0 days

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. Transient tables (and temporary tables) have no Fail-safe periodaa  taa  ll — 0 days.
</details>

---

### Question 227
What isaa   machine learningaa  nd data science partner within the Snowflake Partner Ecosystem?

-aa  . Informatica
- B. Power BI
- C.aa  dobe
- D. DataRobot

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. DataRobot is listedaa  mong Snowflake's machine learning / data science ecosystem partners; the others fall under data integration (Informatica) or BI (Power BI).
</details>

---

### Question 228
Which statementsaa  re correct concerning the use of third-party data from the Snowflake Marketplace? (Choose two.)

-aa  . Data is live, ready-to-query,aa  nd can be personalized.
- B. Data needs to be loaded intoaa   cloud-provideraa  ccountaa  saa   consumer.
- C. Data isaa  vailable for copying/moving intoaa  n individual Snowflakeaa  ccount.
- D. Data isaa  vailable without copying or moving it.
- E. Data transformationsaa  reaa  lways required when combining Marketplace datasets with existing data.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , D. Marketplace data listings provide live, ready-to-queryaa  ccess withoutaa  ny data movement or ETL required to make the data queryable.
</details>

---

### Question 229
What impacts the credit consumption of maintainingaa   materialized view? (Choose two.)

-aa  . Whether it isaa  lsoaa   secure view
- B. How often the underlying base table is queried
- C. How often the base table changes
- D. Whether the materialized view hasaa   clustering key defined
- E. How often the materialized view itself is queried

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C, D. Snowflakeaa  utomatically refreshesaa   materialized view in the background whenever the base table changes — more frequent changes mean more background maintenance credits. If the materialized viewaa  lso hasaa   clustering key, ongoingaa  utomatic reclusteringaa  dds further maintenance cost. Querying the MV itself is billedaa  s regular compute, not "maintenance."
</details>

---

### Question 230
What `COPY INTO <location>` setting should be used to unload data into multiple files?

-aa  . `SINGLE = TRUE`
- B. `MULTIPLE = TRUE`
- C. `MULTIPLE = FALSE`
- D. `SINGLE = FALSE`

<details><summary>Showaa  nswer</summary>
⚠ Updated: The real `COPY INTO <location>` syntax only hasaa   `SINGLE` parameter (there is no `MULTIPLE` parameter in Snowflake SQL). `SINGLE = FALSE` is the defaultaa  nd produces multiple output files. Correctaa  nswer: D.
</details>

---

### Question 231
When cloningaa   database containing stored proceduresaa  nd regular views that have fully qualified table references, what happens?

-aa  . The cloned viewsaa  nd stored procedures will reference the cloned tables in the new database.
- B.aa  n error will occur,aa  s views with qualified references cannot be cloned.
- C.aa  n error will occur,aa  s stored objects cannot be cloned.
- D. The stored proceduresaa  nd views will continue to refer to tables in the original database.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. Viewsaa  nd stored procedures store fully qualified referencesaa  t creation time; cloning the database does not rewrite those references, so they keep pointing back to the original (source) objects.
</details>

---

### Question 232
When loading data into Snowflake, how should the data be organized for best performance?

-aa  . Into files with roughly 100–250 MB of compressed data per file
- B. Into files with roughly 1–100 MB of compressed data per file
- C. Into files withaa   maximum size of 1 GB of compressed data per file
- D. Into files withaa   maximum size of 4 GB of data per file

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  . Snowflake recommends splitting data into compressed files of roughly 100–250 MB (or larger) to maximize parallelism during loading.
</details>

---

### Question 233
Which of the following objects can be directly restored using the `UNDROP` command? (Choose two.)

-aa  . Schema
- B. View
- C. Internal Stage
- D. Table
- E. User
- F. Role

<details><summary>Showaa  nswer</summary>
⚠ Updated: Correctaa  nswer (from the given options):aa  , D. `UNDROP` hasaa  lways supported Table, Schema,aa  nd Database. Note that Snowflake has since expanded `UNDROP` support to many more object types (Dynamic Tables, Iceberg Tables, Notebooks, Streamlitaa  pps, Tags, external volumes,aa  nd evenaa  ccounts) — butaa  mong the six options listed here, only Schemaaa  nd Tableaa  re (and were) valid. Views, internal stages, users,aa  nd rolesaa  re not restorable with `UNDROP`.
</details>

---

### Question 234
Which Snowflake SQL statement would be used to determine which usersaa  nd roles haveaa  ccess toaa   role called `MY_ROLE`?

-aa  . `SHOW GRANTS OF ROLE MY_ROLE`
- B. `SHOW GRANTS TO ROLE MY_ROLE`
- C. `SHOW GRANTS FOR ROLE MY_ROLE`
- D. `SHOW GRANTS ON ROLE MY_ROLE`

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  . `SHOW GRANTS OF ROLE <name>` lists the usersaa  nd roles to which the role has been granted. (`SHOW GRANTS TO ROLE <name>` instead lists the privileges the role itself holds.)
</details>

---

### Question 235
What is the minimum edition of Snowflake required to useaa   SCIM security integration?

-aa  . Business Critical Edition
- B. Standard Edition
- C. Virtual Private Snowflake (VPS)
- D. Enterprise Edition

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. SCIM 2.0 provisioning isaa  vailableaa  crossaa  ll Snowflake editions, including Standard.
</details>

---

### Question 236
A user createdaa   transient tableaa  nd made several changes to it over the course of several days. Three daysaa  fter the table was created, the user wants to go back to the first version of the table. How can this beaa  ccomplished?

-aa  . Use Time Travelaa  s longaa  s `DATA_RETENTION_TIME_IN_DAYS` is set toaa  t least 3 days.
- B. It cannot be done — transient tables haveaa   maximum Time Travel retention of only 1 dayaa  nd no Fail-safe period.
- C. Contact Snowflake Support to have the data retrieved from Fail-safe storage.
- D. Use the `FAILSAFE` parameter with Time Travel to retrieve the data from Fail-safe storage.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Transient tables cap outaa  t 1 day of Time Travel retention (it cannot be set to 3 days),aa  nd they have zero days of Fail-safe, so the original version is unrecoverableaa  fter that window closes.
</details>

---

### Question 237
When reviewing warehouse load, the load-monitoring chart showsaa   high volume of queries constantly queuing.aa  ccording to best practice, what should be done to reduce the queue? (Choose two.)

-aa  . Use multi-cluster warehousing to scale out warehouse capacity.
- B. Scale up the warehouse size so queries execute faster.
- C. Stopaa  nd start the warehouse to clear the queued queries.
- D. Migrate some queries toaa   new warehouse to reduce load.
- E. Restrict users fromaa  ccessing the warehouse so fewer queries runaa  gainst it.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , D. Queuing isaa   concurrency problem, best solved by scaling out (multi-cluster) or spreading the workloadaa  cross multiple warehouses — not by scaling up (which helps individual query speed, not queuing) or restarting the warehouse (which doesn'taa  dd capacity).
</details>

---

### Question 238
Which of the following features,aa  ssociated with Continuous Data Protection (CDP), requireaa  dditional Snowflake-provided data storage? (Choose two.)

-aa  . Tri-Secret Secure
- B. Time Travel
- C. Fail-safe
- D. Data encryption
- E. External stages

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B, C. Time Travelaa  nd Fail-safe both retain historical versions of data, which consumesaa  dditional storage that is billed to theaa  ccount.
</details>

---

### Question 239
Where canaa   user findaa  nd review the failed logins ofaa   specific user for the past 30 days?

-aa  . The `USERS` view in `ACCOUNT_USAGE`
- B. The `LOGIN_HISTORY` view in `ACCOUNT_USAGE`
- C. The `ACCESS_HISTORY` view in `ACCOUNT_USAGE`
- D. The `SESSIONS` view in `ACCOUNT_USAGE`

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. `SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY` records loginaa  ttempts, including failures,aa  nd (unlike `INFORMATION_SCHEMA.LOGIN_HISTORY`) retains up to 365 days of history.
</details>

---

### Question 240
What is the purpose ofaa  n External Function?

-aa  . To call code that executes outside of Snowflake
- B. To runaa   function inaa  nother Snowflake database
- C. To share data in Snowflake with external parties
- D. To ingest data from on-premises data sources

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  . External functions let SQL code call out toaa   remote service (e.g.,aa  naa  WS Lambda oraa  zure Function) hosted outside of Snowflake.
</details>

---

### Question 241
Which of the following statementsaa  pply to Snowflake in terms of security? (Choose two.)

-aa  . Snowflake leveragesaa   Role-Basedaa  ccess Control (RBAC) model.
- B. Snowflake requiresaa   user to configureaa  n IAM user to connect to the database.
- C.aa  ll data in Snowflake is encrypted.
- D. Snowflake can run entirely withinaa   user's own Virtual Private Cloud (VPC).
- E.aa  ll data in Snowflake is compressed.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , C. Snowflake'saa  ccess model combines RBACaa  nd DAC,aa  ndaa  ll data isaa  utomatically encryptedaa  t restaa  nd in transit by default, regardless of edition.
</details>

---

### Question 242
A single user ofaa   virtual warehouse has set it toaa  uto-resumeaa  ndaa  uto-suspendaa  fter 10 minutes. The warehouse is currently suspended,aa  nd the user performs the following:
1. Runsaa   query that takes 3 minutes to complete.
2. Leaves for 15 minutes.
3. Returnsaa  nd runsaa   query that takes 10 seconds to complete.
4. Manually suspends the warehouseaa  s soonaa  s the last query completes.

How much billable compute time will have been consumed?

-aa  . 4 minutes
- B. 13 minutes
- C. 14 minutes
- D. 24 minutes

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C. 3 minutes (query 1) + 10 minutes idle beforeaa  uto-suspend kicks in (the 15-minuteaa  bsence exceeds the 10-minute timeout) + 1 minute minimum billing for the second query (Snowflake bills withaa   60-second minimum) = 14 minutes.
</details>

---

### Question 243
What can be used to view warehouse usage time? (Choose two.)

-aa  . The `LOAD_HISTORY` view
- B. The Query History view
- C. The `SHOW WAREHOUSES` command
- D. The `WAREHOUSE_METERING_HISTORY` view in `ACCOUNT_USAGE`
- E. The Billing & Usage tab in the Snowflake web UI

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D, E. `ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY`aa  nd the Billing & Usageaa  rea in Snowsight both report warehouse credit/usage time.
</details>

---

### Question 244
Whataa  ctions will prevent leveraging of the result set (query results) cache? (Choose two.)

-aa  . Removingaa   column from the query's SELECT list
- B. Stopping the virtual warehouse the query is runningaa  gainst
- C. Clustering the data used by the query
- D. Executing the `RESULT_SCAN` table function
- E. The underlying data used by the query has changed

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , E.aa  ny change to the query text (like removingaa   column) or to the underlying table data invalidates the cached result. Stopping the warehouse doesn'taa  ffect the cache (it's independent ofaa  ny warehouse),aa  nd `RESULT_SCAN` reads the cache rather than breaking it.
</details>

---

### Question 245
Which statement is trueaa  bout running tasks in Snowflake?

-aa  .aa   task can be called usingaa   `CALL` statement to runaa   set of predefined SQL commands.
- B.aa   taskaa  llowsaa   user to executeaa   single SQL statement or stored procedure call onaa   predefined schedule.
- C.aa   taskaa  llowsaa   user to executeaa   set of SQL commands onaa   predefined schedule.
- D.aa   task can be executed usingaa   `SELECT` statement to runaa   predefined SQL command.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Each individual task executes one SQL statement (which can itself beaa   call toaa   stored procedure containing multiple statements) onaa   schedule; chaining multiple tasks together formsaa   task graph (DAG) for more complex workflows.
</details>

---

### Question 246
Which data types does Snowflake support when querying semi-structured data? (Choose two.)

-aa  . VARIANT
- B. VARCHAR
- C. XML
- D.aa  RRAY
- E. BLOB

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , D. VARIANT storesaa  rbitrary semi-structured data,aa  ndaa  RRAY/OBJECTaa  re the structured sub-types used to navigate it. VARCHARaa  nd BLOBaa  re not semi-structured types,aa  nd XML isaa   file format, notaa   Snowflake column data type.
</details>

---

### Question 247
Inaa  naa  uto-scaling multi-cluster virtual warehouse with `SCALING_POLICY = ECONOMY`, when isaa  naa  dditional cluster started?

-aa  . When the system has enough load for 2 minutes
- B. When the system has enough load for 6 minutes
- C. When the system has enough load for 8 minutes
- D. When the system has enough load for 10 minutes

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. The ECONOMY scaling policy favors keeping existing clusters fully loadedaa  nd only startsaa   new cluster if the queued load is expected to keep it busy foraa  t least 6 minutes (this conserves credits compared to the STANDARD policy).
</details>

---

### Question 248
What is the following SQL command used for?

```sql
SELECT * FROM TABLE(VALIDATE(t1, JOB_ID => '_last'));
```

-aa  . To validate external table filesaa  gainst table t1aa  crossaa  ll sessions
- B. To validate task SQL statementsaa  gainst table t1 over the last 14 days
- C. To validateaa   file for errors before it gets loaded viaaa   `COPY` command
- D. To return errors from the last executed `COPY` command into table t1, within the current session

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. `VALIDATE(table_name, JOB_ID => '_last')` returns the load errors from the most recent `COPY INTO` load for that table, scoped to the current session.
</details>

---

### Question 249
A table `FCT_SALES` has 100 million rows. The following query is executed:

```sql
SELECT COUNT(*) FROM FCT_SALES;
```

How did Snowflake fulfill this query?

-aa  . Queryaa  gainst the result set cache
- B. Queryaa  gainstaa   virtual warehouse's local disk cache
- C. Queryaa  gainst the most-recently created micro-partition
- D. Queryaa  gainst table metadata (no data scan required)

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. Snowflake stores row countsaa  s metadata for each micro-partition, soaa   simple `COUNT(*)` (with no filters) can beaa  nswered entirely from metadata, without scanningaa  nyaa  ctual data or even requiringaa   running warehouse.
</details>

---

### Question 250
What happens whenaa   virtual warehouse is resized?

-aa  . When increasing the size ofaa  naa  ctive warehouse,aa  ll runningaa  nd queued queriesaa  reaa  ffected immediately.
- B. When reducing the size ofaa   warehouse, compute resourcesaa  re removed only once theyaa  re no longer being used byaa  ny currently executing statement.
- C. The warehouse is suspended while the new compute resourcesaa  re provisioned, thenaa  utomatically resumes once provisioning completes.
- D. Users trying to use the warehouse will receiveaa  n error message until resizing completes.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Scaling upaa  dds resources for new queries without disrupting queriesaa  lready running; scaling down removes resources gracefully, onlyaa  fter in-flight statements finish using them.
</details>

---

### Question 251
What tasks can be completed using the `COPY INTO <table>` command? (Choose two.)

-aa  . Columns can be renamed.
- B. Columns can be joined withaa  n existing table.
- C. Columns can be reordered.
- D. Columns can be omitted.
- E. Data can be loaded without spinning upaa   virtual warehouse.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C, D. Usingaa   column listaa  nd `SELECT` transformation in the `COPY INTO <table>` statement, you can reorder or omit source columns during the load.aa   running warehouse (or Snowpipe's serverless compute) is still required to execute the load.
</details>

---

### Question 252
Which Snowflake layer can be directly configured by users?

-aa  . Database Storage
- B. Cloud Services
- C. Compute (Query Processing)
- D.aa  pplication Services

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C. Users create, resize,aa  nd manage virtual warehouses in the Compute layer. Storageaa  nd Cloud Services scaleaa  utomaticallyaa  ndaa  re not directly configured by customers.
</details>

---

### Question 253
Query compilation occurs in which layer of Snowflake'saa  rchitecture?

-aa  . Compute layer
- B. Storage layer
- C. Cloud infrastructure layer
- D. Cloud Services layer

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. Query parsing, compilation,aa  nd optimizationaa  ll happen in the Cloud Services layer, beforeaa  ny work is dispatched toaa   virtual warehouse for execution.
</details>

---

### Question 254
Ifaa  n X-Small virtual warehouse is made up of one serveraa  ndaa   Small warehouse is made up of two servers, how many servers make upaa   Large warehouse?

-aa  . 4
- B. 8
- C. 16
- D. 32

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Each warehouse size doubles the compute of the previous size: X-Small=1, Small=2, Medium=4, Large=8, X-Large=16,aa  nd so on.
</details>

---

### Question 255
A clustering key was defined onaa   table but is no longer needed. How can the key be removed?

-aa  . `ALTER TABLE [table_name] PURGE CLUSTERING KEY`
- B. `ALTER TABLE [table_name] DELETE CLUSTERING`
- C. `ALTER TABLE [table_name] DROP CLUSTERING KEY`
- D. `ALTER TABLE [table_name] REMOVE CLUSTERING KEY`

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C. `ALTER TABLE ... DROP CLUSTERING KEY` is the correct syntax to removeaa   defined clustering key (the table itself is unaffected).
</details>

---

### Question 256
What is the purpose of clustering?

-aa  . To guarantee uniquely identifiable records in the database
- B. To increase scan efficiency in queries by improving partition pruning
- C. To improve performance by creatingaa   separate file for point lookups
- D. To provide data redundancy by duplicating micro-partitions

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Clustering co-locates similar column values within micro-partitions so that queries filtering on the clustering key can prune (skip) irrelevant partitions.
</details>

---

### Question 257
Which statement is trueaa  bout Multi-Factoraa  uthentication (MFA) in Snowflake?

-aa  . MFA can be enforced foraa   given role.
- B. Snowflake usersaa  reaa  utomatically enrolled in MFA.
- C. Users enroll in MFA by submittingaa   request to Snowflake Support.
- D. MFA isaa   natively integrated Snowflake feature.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. Snowflake has native MFA (powered by Duo Security) built directly into the platform. Enrollment is self-service through the user's profile, notaa  utomaticaa  nd not something Support has to configure.
</details>

---

### Question 258
What data type should be used to store JSON data natively in Snowflake?

-aa  . JSON
- B. STRING
- C. OBJECT
- D. VARIANT

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. VARIANT natively stores semi-structured data suchaa  s JSON,aa  vro, ORC, Parquet, or XML, preserving its structure for querying.
</details>

---

### Question 259
What should be considered when deciding to useaa   Secure View? (Choose two.)

-aa  . Details of the query execution planaa  re hidden from non-owners in the Query Profile.
- B. Once created, there is no way to determine whetheraa   view is secure or not.
- C. Secure views do not takeaa  dvantage of the same internal optimizationsaa  s standard views.
- D. It is not possible to createaa   secure materialized view.
- E. The view definition ofaa   secure view isaa  lways visible toaa  ll users via the Information Schema.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , C. Secure views intentionally hide internal query details (like the underlying SQL textaa  nd parts of the execution plan) from unauthorized users to protect the view logic,aa  ndaa  saa   trade-off they skip some query-optimization techniques (like predicate pushdown) that could otherwise leak informationaa  bout the underlying data.
</details>

---

### Question 260
The Information Schema provides storage information for which of the following objects? (Choose two.)

-aa  . Users
- B. Databases
- C. Internal Stages
- D. Resource Monitors
- E. Pipes

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B, C (as sourced from the original question bank). Note: detailed storage metrics for tables/stagesaa  re more comprehensivelyaa  vailable via `SNOWFLAKE.ACCOUNT_USAGE`, so treat this specific pairing with some caution if you encounter it onaa  naa  ctual exam.
</details>

---

### Question 261
What isaa   responsibility of Snowflake's virtual warehouses?

-aa  . Infrastructure management
- B. Metadata management
- C. Query execution
- D. Query parsingaa  nd optimization
- E. Management of storage

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C. Virtual warehouses (the Compute layer)aa  re responsible for executing queriesaa  nd DML operations. Parsing/optimizationaa  nd metadata management happen in Cloud Services; storage is managed independently.
</details>

---

### Question 262
Which data type is supported by Snowflake's native data classification feature?

-aa  . FLOAT
- B. STRING
- C. GEOGRAPHY
- D. VARIANT

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Snowflake'saa  utomated data classificationaa  nalyzes STRING-type column data to detectaa  nd tag categories like PII (names, emails, etc.).
</details>

---

### Question 263
When unloading data toaa  n external stage, which compression format can be used for Parquet files with the `COPY INTO` command?

-aa  . BROTLI
- B. GZIP
- C. LZO
- D. ZSTD

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C. For `TYPE = PARQUET`, the supported `COMPRESSION` valuesaa  re `AUTO`, `LZO`, `SNAPPY` (the default), or `NONE`. BROTLI, GZIP,aa  nd ZSTDaa  re valid for other file types (CSV, JSON,aa  vro) but not for Parquet.
</details>

---

### Question 264
Which SQL command can be used to verify the privileges thataa  re granted toaa   role?

-aa  . `SHOW GRANTS ON ROLE [role_name]`
- B. `SHOW ROLES [role_name]`
- C. `SHOW GRANTS TO ROLE [role_name]`
- D. `GRANTS FOR ROLE [role_name]`

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C. `SHOW GRANTS TO ROLE <name>` lists the privileges that have been granted toaa   role.
</details>

---

### Question 265
Which Query Profile result indicates thataa   warehouse is sized too small foraa   query?

-aa  . Thereaa  reaa   lot of filter nodes.
- B. Bytesaa  re spilling to local or remote storage.
- C. The percentage of partitions scanned is very high.
- D. The number of partitions scanned equals the total number of partitions.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Whenaa   warehouse doesn't have enough memory foraa  n operation (e.g.,aa   large sort or join), Snowflake spills intermediate results to local diskaa  nd then to remote storage —aa   clear sign the warehouse should be resized larger.
</details>

---

### Question 266
What is the default Time Travel retention period?

-aa  . 1 day
- B. 7 days
- C. 45 days
- D. 90 days

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  . Theaa  ccount default for `DATA_RETENTION_TIME_IN_DAYS` is 1 day; Enterprise Edition (and higher) can extend this up to 90 days for permanent objects if explicitly configured.
</details>

---

### Question 267
Which of the followingaa  re best-practice recommendations to consider when loading data into Snowflake? (Choose two.)

-aa  . Load files thataa  reaa  pproximately 25 MB or smaller.
- B. Removeaa  ll datesaa  nd timestamps.
- C. Load files thataa  reaa  pproximately 100–250 MB (or larger) compressed.
- D.aa  void using embedded characters, suchaa  s commas, for numeric data types.
- E. Removeaa  ll semi-structured data types.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C, D.aa  im for 100–250 MB compressed files for load parallelism,aa  ndaa  void embedding delimiters like commas inside numeric fields (which breaks parsing).
</details>

---

### Question 268
Which schema contains the `RESOURCE_MONITORS` view?

-aa  . `SNOWFLAKE.ACCOUNT_USAGE`
- B. `SNOWFLAKE.READER_ACCOUNT_USAGE`
- C. `INFORMATION_SCHEMA`
- D. `SNOWFLAKE.ORGANIZATION_USAGE`

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  . Resource monitorsaa  reaa  ccount-level objects,aa  nd their metadata is surfaced through `SNOWFLAKE.ACCOUNT_USAGE.RESOURCE_MONITORS` — notaa   per-database `INFORMATION_SCHEMA`.
</details>

---

### Question 269
What is the purpose of enabling Federatedaa  uthentication onaa   Snowflakeaa  ccount?

-aa  . Disables theaa  bility to use key-pairaa  nd basic username/passwordaa  uthentication when connecting.
- B.aa  llows dual Multi-Factoraa  uthentication (MFA) when connecting to Snowflake.
- C. Forces users to connect throughaa   secure network proxy.
- D.aa  llows users to connect using secure single sign-on (SSO) throughaa  n external identity provider.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. Federatedaa  uthentication delegates login toaa  n external SAML 2.0 identity provider (e.g., Okta,aa  zureaa  D), enabling SSO.
</details>

---

### Question 270
Which Snowflake Partner Ecosystem category is representedaa  t the top of the (referenced) partner diagram?

-aa  . Business Intelligence
- B. Machine Learningaa  nd Data Science
- C. Securityaa  nd Governance
- D. Data Integration

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D (as sourced from the original question bank). Note: the diagram referenced in the original source wasn't legible/available in the OCR text, so thisaa  nswer could not be independently re-verifiedaa  gainstaa  n image — treat it withaa   bit more caution than the rest of this set.
</details>

---

### Question 271
Which object typesaa  re protected by Fail-safe? (Choose two.)

-aa  . Permanent tables
- B. Temporary tables
- C. External tables
- D. Materialized views
- E. Transient tables

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , D. Fail-safeaa  pplies only to permanent objects (permanent tablesaa  nd their materialized views). Temporaryaa  nd transient tables have no Fail-safe,aa  nd external tables don't store data within Snowflakeaa  taa  ll.
</details>

---

### Question 272
Snowflake'saa  pproach to the management of systemaa  ccess combines which of the following? (Choose two.)

-aa  . Securityaa  ssertion Markup Language (SAML)
- B. Role-Basedaa  ccess Control (RBAC)
- C. Identityaa  ccess Management (IAM)
- D. Create, Read, Update,aa  nd Delete (CRUD)
- E. Discretionaryaa  ccess Control (DAC)
- F. Mandatoryaa  ccess Control (MAC)

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B, E. Snowflake'saa  ccess control model combines Role-Basedaa  ccess Control (privilegesaa  ssigned to roles, whichaa  reaa  ssigned to users) with Discretionaryaa  ccess Control (each object hasaa  n owning role that can grantaa  ccess to it).
</details>

---

### Question 273
According to Snowflake best-practice recommendations, which role should be used to create databases?

-aa  .aa  CCOUNTADMIN
- B. SYSADMIN
- C. SECURITYADMIN
- D. USERADMIN

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. SYSADMIN is the recommended role for creatingaa  nd managing warehouses, databases,aa  nd other data objects, keepingaa  CCOUNTADMIN reserved foraa  ccount-levelaa  dministration.
</details>

---

### Question 274
Toaa  dd or remove search optimization foraa   table,aa   user must have which of the following privileges? (Choose two.)

-aa  . The MODIFY privilege on the table
- B. The OWNERSHIP privilege on the table
- C. The SECURITYADMIN role
- D. Theaa  DD SEARCH OPTIMIZATION privilege on the schema that contains the table
- E. The SELECT privilege on the table

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B, D.aa   role needs OWNERSHIP on the table itself, plus theaa  DD SEARCH OPTIMIZATION privilege (granted by default to the schema owner, or grantable toaa  nother role) on the containing schema.
</details>

---

### Question 275
While usingaa   `COPY` command withaa   `VALIDATION_MODE` parameter, which of the following will returnaa  n error?

-aa  . Statements that insertaa   duplicate record duringaa   load
- B. Statements that haveaa   specific data type in the source
- C. Statements that have duplicate file names
- D. Statements that transform data duringaa   load

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. `VALIDATION_MODE` is not compatible withaa   `COPY INTO <table>` statement thataa  lso performs data transformations (i.e., usesaa   `SELECT` with column transformations) — this combination returnsaa  n error.
</details>

---

### Question 276
When is the result set (query results) cache no longeraa  vailable? (Choose two.)

-aa  . Whenaa   different warehouse is used to execute the query
- B. When the user executes the `RESULT_SCAN` function
- C. When the underlying data used by the query has changed
- D. When the warehouse used to execute the query is suspended
- E. When it has been 24 hours since the query was last executed

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C, E. The results cache is invalidated once the underlying data changes, oraa  fter 24 hours have passed since the results were last used (each use resets that 24-hour clock, up toaa   maximum of 31 days from the original execution). It is independent of which warehouse runs the query or whether that warehouse is suspended.
</details>

---

### Question 277
What is the recommended file sizing for data loading using Snowpipe?

-aa  .aa   compressed file size greater than 100 MB,aa  nd up to 250 MB
- B.aa   compressed file size greater than 100 GB,aa  nd up to 250 GB
- C.aa   compressed file size greater than 10 MB,aa  nd up to 100 MB
- D.aa   compressed file size greater than 1 GB,aa  nd up to 2 GB

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  .aa  s with bulk loading, Snowflake recommends compressed files of roughly 100–250 MB for Snowpipe to balance load latencyaa  nd per-file overhead.
</details>

---

### Question 278
Which statementsaa  re true concerning Snowflake's underlying cloud infrastructure? (Choose three.)

-aa  . Snowflake dataaa  nd servicesaa  re deployed inaa   singleaa  vailability zone withinaa   cloud provider's region.
- B. Snowflake dataaa  nd servicesaa  reaa  vailable only inaa   single cloud provideraa  nd region; use of multiple cloud providers is not supported.
- C. Snowflake can be deployed inaa   customer's private cloud, using the customer's own computeaa  nd storage resources.
- D. Snowflake uses the core computeaa  nd storage services of each cloud provider it runs on.
- E.aa  ll three layers (storage, compute,aa  nd cloud services)aa  re deployedaa  nd managed entirely on the selected cloud platform.
- F. Snowflake dataaa  nd servicesaa  re deployedaa  crossaa  t least threeaa  vailability zones withinaa   cloud provider's region.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D, E, F. Snowflake runs entirely on top of the native compute/storage services ofaa  WS,aa  zure, or GCP, spansaa  t least threeaa  vailability zones for resilience,aa  nd does not deploy intoaa   customer's own private infrastructure.
</details>

---

### Question 279
A user unloadedaa   Snowflake table called `mytable` toaa  n internal stage called `mystage`. Which command can be used to view the list of files uploaded to the stage?

-aa  . `LIST @mytable;`
- B. `LIST TABLE mystage;`
- C. `SHOW STAGE mystage;`
- D. `LIST @mystage;`

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. `LIST @mystage;` lists the files present in the named internal stage `mystage`.
</details>

---

### Question 280
What isaa   best practiceaa  fter creatingaa   custom role?

-aa  . Create the custom role using the SYSADMIN role.
- B.aa  ssign the custom role to the SYSADMIN role.
- C.aa  ssign the custom role to the PUBLIC role.
- D.aa  dd `_CUSTOM` toaa  ll custom role names.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Best practice is to grant custom roles up to SYSADMIN in the role hierarchy so systemaa  dministrators retain full visibilityaa  nd control overaa  ll custom objects.
</details>

---

### Question 281
Which is the minimum required Snowflake edition to useaa  WS/Azure PrivateLink or Google Cloud Private Service Connect?

-aa  . Standard
- B. Premium
- C. Enterprise
- D. Business Critical

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. Private connectivity (AWS PrivateLink,aa  zure Private Link, Google Cloud Private Service Connect) requires Business Critical Edition or higher.
</details>

---

### Question 282
Which of the following Query Profile indicators shows thataa   virtual warehouse is not sized correctly for the query being executed?

-aa  . Bytes sent over the network
- B. Synchronization time
- C. Initialization time
- D. Remote spillage (bytes spilled to remote storage)

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. Spilling to remote storage means the warehouse ran out of local memory/disk for the operation —aa   strong signal the warehouse should be resized up.
</details>

---

### Question 283
Which of the following Snowflake capabilitiesaa  reaa  vailable inaa  ll Snowflake editions? (Choose two.)

-aa  . Encryption key management through Tri-Secret Secure
- B.aa  utomatic encryption ofaa  ll data
- C. Up to 90 days of data recovery through Time Travel
- D. Object-levelaa  ccess control
- E. Column-level security toaa  pply masking policies to tablesaa  nd views

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B, D.aa  lways-on encryption of dataaa  t rest/in transitaa  nd RBAC-based object-levelaa  ccess controlaa  re baseline features of every edition, including Standard. Tri-Secret Secure requires Business Critical, extended Time Travel (beyond 1 day) requires Enterprise+,aa  nd masking policies require Enterprise+.
</details>

---

### Question 284
A `PUT` command can be used to stage local files from which Snowflake interface?

-aa  . SnowSQL (CLI)
- B. Snowflake Classic Console (UI)
- C. Snowsight
- D. .NET driver

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  . `PUT` requiresaa  ccess to the local file system, so it is executed fromaa   CLI or driver context like SnowSQL — not from the browser-based Snowsight or Classic Console UI, which cannotaa  ccess your local disk directly.
</details>

---

### Question 285
Which of the following indicate that it may beaa  ppropriate to defineaa   clustering key foraa   table? (Choose two.)

-aa  . The table containsaa   column that has very low cardinality.
- B. DML statements being issuedaa  gainst the tableaa  re blocked.
- C. The table hasaa   small number of micro-partitions.
- D. Queries on the tableaa  re running slower than expected.
- E. The clustering depth for the table is large.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D, E. Slower-than-expected query performance combined withaa   large clustering depth (poor data organization relative to the clustering/filter columns)aa  re the classic signals that clustering would help. Very low cardinality, few partitions, or DML blockingaa  re not indicators for clustering.
</details>

---

### Question 286
Which cache type is used to cache the data output from SQL queries?

-aa  . Metadata cache
- B. Result (query results) cache
- C. Remote cache
- D. Local disk (warehouse) cache

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. The result cache stores theaa  ctual output ofaa   previously run query for near-instant retrieval onaa  n identical repeat query, for up to 24 hours (extendable to 31 days with reuse).
</details>

---

### Question 287
Which of the following describes how clustering keys work in Snowflake?

-aa  . Clustering keys update micro-partitions in place withaa   full sort,aa  nd block DML operations.
- B. Clustering keys sort the designated columns over time, without blocking DML operations.
- C. Clustering keys createaa   distributed, parallel data structure of pointers to rowsaa  nd columns.
- D. Clustering keys establishaa   hashed key on each node ofaa   virtual warehouse to optimize joinsaa  t run-time.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B.aa  utomatic reclustering incrementally re-sorts micro-partitions in the background over time,aa  nd never blocks concurrent DML on the table.
</details>

---

### Question 288
Which of the following operations require the use ofaa   running virtual warehouse? (Choose two.)

-aa  . Downloading data fromaa  n internal stage
- B. Listing files inaa   stage
- C. Executingaa   stored procedure
- D.aa  lteringaa   table's metadata (DDL)
- E. Querying data fromaa   materialized view

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C, E. Executing SQL insideaa   stored procedureaa  nd queryingaa   materialized view both require compute. Listing/downloading stage filesaa  nd most metadata-only DDL operationsaa  re handled by Cloud Servicesaa  nd don't needaa  naa  ctive warehouse.
</details>

---

### Question 289
What is used to limit the credit usage ofaa   virtual warehouse withinaa   Snowflakeaa  ccount?

-aa  . Load monitor
- B. Resource monitor
- C. Query profile
- D. Warehouse policy

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B. Resource monitors track credit usageaa  gainst defined thresholdsaa  nd can trigger notifications oraa  utomatically suspend warehouses when limitsaa  re reached.
</details>

---

### Question 290
Whataa  re the benefits of the replication feature in Snowflake? (Choose two.)

-aa  . Disaster recovery
- B. Time Travel
- C. Fail-safe
- D. Database failoveraa  nd failback
- E. Data security

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , D. Cross-region/cross-cloud replication supports disaster recovery scenariosaa  nd enablesaa  ccount failover/failback toaa   secondaryaa  ccount (Business Critical Edition or higher for failover/failback specifically).
</details>

---

### Question 291
Which of the following rolesaa  re recommended to createaa  nd manage other usersaa  nd roles? (Choose two.)

-aa  . SYSADMIN
- B. SECURITYADMIN
- C. PUBLIC
- D.aa  CCOUNTADMIN
- E. USERADMIN

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B, E. SECURITYADMIN manages grants globallyaa  nd can create/manage usersaa  nd roles; USERADMIN is specifically dedicated to creatingaa  nd managing usersaa  nd roles.
</details>

---

### Question 292
When canaa   newly configured virtual warehouse start running SQL queries?

-aa  . Immediately, while provisioning is still in progress
- B. Only during specific time slots defined by theaa  CCOUNTADMIN
- C.aa  fter warehouse provisioning has completed
- D.aa  fter warehouse replication has completed

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C.aa   virtual warehouse canaa  cceptaa  nd run queriesaa  s soonaa  s its compute resources finish provisioning.
</details>

---

### Question 293
Whataa  ction will prevent leveraging of the result set cache?

-aa  . Removingaa   column from the query's SELECT list
- B. Stopping the virtual warehouse that the query is runningaa  gainst
- C. The result not being reused within the last 12 hours
- D. Executing the `RESULT_SCAN` table function

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  . Changing the query text — including removingaa   column from the SELECT list — producesaa   different query signature, so it can't reuseaa   previous result. (Note: the real cache expiry window is 24 hours, not 12, so option C isaa   deliberately incorrect distractor; stopping the warehouseaa  nd using `RESULT_SCAN` don't invalidate the cache.)
</details>

---

### Question 294
Which of the followingaa  re benefits of micro-partitioning? (Choose two.)

-aa  . Micro-partitions cannot overlap in their range of values.
- B. Micro-partitionsaa  re immutable objects that support the use of Time Travel.
- C. Micro-partitions can reduce theaa  mount of I/O from object storage to virtual warehouses.
- D. Rowsaa  reaa  utomatically stored in sorted order within every micro-partition.
- E. Micro-partitions can be defined onaa   schema-by-schema basis.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B, C. Because micro-partitionsaa  re immutable, Snowflake can efficiently retain historical versions for Time Travel,aa  nd because each partition stores rich metadata, queries can prune irrelevant partitionsaa  nd reduce I/O. (Partitions can overlap in value ranges,aa  nd micro-partitioning isaa  utomatic — not something configured per schema.)
</details>

---

### Question 295
Which data type can be used to store geospatial data in Snowflake?

-aa  . VARIANT
- B. OBJECT
- C. GEOMETRY
- D. GEOGRAPHY

<details><summary>Showaa  nswer</summary>
⚠ Updated: Correctaa  nswer: D (GEOGRAPHY) was the traditional/primaryaa  nswer,aa  nd remains valid — GEOGRAPHY stores spherical (latitude/longitude, WGS 84) geospatial data. However, Snowflake has sinceaa  lsoaa  ddedaa   GEOMETRY data type for planar/Cartesian geospatial data, so C is nowaa  lsoaa   technically correctaa  nswer to "which data type can be used." If this isaa   single-select exam question, GEOGRAPHY (D) remains the expectedaa  nswer.
</details>

---

### Question 296
Ifaa  ll virtual warehouse resourcesaa  re maximized while processingaa   query workload, what happens to new queries submitted to the warehouse?

-aa  .aa  ll queries terminate once resourcesaa  re maximized.
- B. The warehouse scales outaa  utomatically.
- C. The warehouse moves toaa   suspended state.
- D. New queriesaa  re queuedaa  nd executed once capacity isaa  vailable.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: D. New queries wait inaa   queue until sufficient compute capacity frees up (unless the warehouse isaa   multi-cluster warehouse configured toaa  uto-scale, in which caseaa  dditional clusters would start instead).
</details>

---

### Question 297
Masking policies can beaa  pplied to which of the following Snowflake objects? (Choose two.)

-aa  .aa   materialized view
- B.aa   stored procedure
- C.aa   table
- D.aa   stream
- E.aa   pipe
- F.aa   function

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  , C. Masking policiesaa  ttach to columns on tables, views,aa  nd materialized views. They cannot beaa  ttached to procedural objects like stored procedures, streams, pipes, or functions.
</details>

---

### Question 298
Whataa  ctionsaa  re supported by Snowflake resource monitors? (Choose two.)

-aa  .aa  lert (notify only)
- B. Notify
- C. Notifyaa  nd suspend
- D.aa  bort
- E. Suspend immediately

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: B, C. Snowflake resource monitors support three real triggeraa  ctions: **Notify**, **Notify & Suspend** (let running queries finish, block new ones),aa  nd **Notify & Suspend Immediately** (cancelaa  ll running queries too). "Abort" is notaa  naa  ctual resource monitoraa  ction.
</details>

---

### Question 299
A user executes the following SQL:

```sql
CREATE TABLE SALES_BKP LIKE SALES;
```

Whataa  re the cost implications of this statement?

-aa  . Processing costs will be generated based on how long the query takes.
- B. Storage costs will be generated based on the size of the data.
- C. No storage cost is incurred, since it relies on metadata only.
- D. The cost of running the virtual warehouse will be charged by the second.

<details><summary>Showaa  nswer</summary>
Correctaa  nswer: C. `CREATE TABLE ... LIKE` copies only the source table's structure (columns, not data), so no data is duplicatedaa  nd effectively noaa  dditional storage cost is incurred.
</details>

---

### Question 300
What is the maximum Time Travel retentionaa  vailable in Snowflake Standard Edition?

-aa  . 1 day
- B. 7 days
- C. 30 days
- D. 90 days

<details><summary>Showaa  nswer</summary>
Correctaa  nswer:aa  . Standard Edition supportsaa   maximum Time Travel retention of 1 day. Extending retention up to 90 days requires Enterprise Edition or higher.
</details>

---

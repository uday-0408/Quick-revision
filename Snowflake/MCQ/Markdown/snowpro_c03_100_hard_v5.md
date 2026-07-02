# SnowPro Core (COF-C03) — 100 Hard Practice Questions (v5)

> **Exam Blueprint Coverage** | Domain 1: Architecture (25%) · Domain 2: Virtual Warehouses (20%) · Domain 3: Storage & Protection (15%) · Domain 4: Data Movement (15%) · Domain 5: Account & Security (15%) · Domain 6: Performance (10%)
>
> All questions are **new** — no duplicates from previous question sets. Difficulty tuned to match COF-C03 exam depth.

---

### Question 1
**Domain:** Domain 1 — Architecture

A query against a 50 TB table returns results in 2 seconds even though the warehouse has been suspended for the past hour and is auto-resumed only on demand. Which architectural component is primarily responsible for this fast response without warehouse compute?

- [ ] A. The metadata cache maintained by the Cloud Services layer, which reconstructs aggregate results from table statistics alone.
- [ ] B. The local SSD cache on the virtual warehouse nodes, which persists data even while the warehouse is suspended.
- [ ] C. The remote disk cache in cloud storage, which Snowflake scans directly without spinning up compute nodes.
- [ ] D. The result cache stored in the Cloud Services layer, which returns previously computed results for an identical query within 24 hours.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. The result cache stored in the Cloud Services layer, which returns previously computed results for an identical query within 24 hours.

**Explanation:**
Snowflake's **result cache** lives in the Cloud Services layer and stores the results of every query executed in the past 24 hours (subject to a few conditions). If an identical query is resubmitted and the underlying data hasn't changed, Snowflake returns the cached result instantly without using any virtual warehouse compute at all — which explains how a suspended warehouse can still serve a near-instant response.
</details>

---

### Question 2
**Domain:** Domain 1 — Architecture

Which statement correctly describes how Snowflake's multi-cluster shared data architecture separates storage and compute?

- [ ] A. Compute and storage are billed together as a single unit because virtual warehouses physically host the data they query.
- [ ] B. Storage is replicated to each virtual warehouse's local disk so that compute clusters never need to read from remote object storage.
- [ ] C. Storage and compute are decoupled, but both layers run on customer-managed infrastructure that Snowflake orchestrates remotely.
- [ ] D. Storage is centralized in cloud object storage while compute is provided by independently scalable virtual warehouses that all access the same data.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. Storage is centralized in cloud object storage while compute is provided by independently scalable virtual warehouses that all access the same data.

**Explanation:**
Snowflake's architecture separates **storage** (centralized, compressed, columnar data in cloud object storage) from **compute** (independently sized and scaled virtual warehouses). Multiple warehouses can read the same underlying data concurrently without contention, since each warehouse is an independent MPP compute cluster that pulls data from the shared storage layer rather than owning a private copy of it.
</details>

---

### Question 3
**Domain:** Domain 1 — Architecture

A table is defined as TRANSIENT instead of PERMANENT. What is the direct consequence for that table's data protection lifecycle?

- [ ] A. Transient tables have no Fail-safe period at all, though they can still have a configurable Time Travel retention of up to 1 day.
- [ ] B. Transient tables behave identically to permanent tables for both Time Travel and Fail-safe, differing only in clustering behavior.
- [ ] C. Transient tables disable Time Travel entirely but retain the full 7-day Fail-safe period like permanent tables.
- [ ] D. Transient tables get a Fail-safe period of exactly 1 day instead of the standard 7 days.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Transient tables have no Fail-safe period at all, though they can still have a configurable Time Travel retention of up to 1 day.

**Explanation:**
**Transient** tables support Time Travel (with a maximum retention of 1 day, versus up to 90 days for permanent tables on Enterprise+), but they have **no Fail-safe period whatsoever**. This makes transient tables cheaper to store but riskier for disaster recovery, since once Time Travel expires the data cannot be recovered by Snowflake support.
</details>

---

### Question 4
**Domain:** Domain 1 — Architecture

In Snowflake's micro-partition architecture, what metadata does the Cloud Services layer store for each micro-partition to enable pruning?

- [ ] A. Min/max values and distinct-value counts for each column within that micro-partition, along with the number of rows.
- [ ] B. A full secondary B-tree index built on every column to allow logarithmic-time lookups.
- [ ] C. The exact byte offsets of every row within the micro-partition, allowing row-level seeks during a scan.
- [ ] D. A bitmap index per column that flags which values are NULL across all micro-partitions in the table.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Min/max values and distinct-value counts for each column within that micro-partition, along with the number of rows.

**Explanation:**
For every micro-partition, Snowflake stores metadata including the **min and max values for each column**, the number of distinct values, and the row count. The query optimizer uses this metadata to prune (skip) micro-partitions that cannot possibly satisfy a filter predicate, dramatically reducing the amount of data scanned without needing traditional indexes.
</details>

---

### Question 5
**Domain:** Domain 1 — Architecture

A consumer account is granted access to a share from a provider account in a different Snowflake region on the same cloud platform. What must happen before the consumer can query the shared data?

- [ ] A. The provider must replicate the database into a secondary database in the consumer's region, and the share must reference that replicated copy.
- [ ] B. The consumer must enable Time Travel on their account so the replicated metadata can sync correctly.
- [ ] C. Cross-region sharing is not supported under any circumstances and the data must be exported and re-imported manually.
- [ ] D. Nothing extra is required; cross-region shares work automatically as long as both accounts are on the same cloud provider.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. The provider must replicate the database into a secondary database in the consumer's region, and the share must reference that replicated copy.

**Explanation:**
Standard secure data sharing requires the provider and consumer accounts to be in the **same region**. To share across regions (even on the same cloud platform), the provider must use **database replication** to create a replica of the database in the consumer's region, and then create the share against that replicated database.
</details>

---

### Question 6
**Domain:** Domain 1 — Architecture

Which Snowflake edition is the minimum required to use Tri-Secret Secure, where Snowflake combines a customer-managed key with a Snowflake-managed key to encrypt data?

- [ ] A. Enterprise Edition
- [ ] B. Virtual Private Snowflake Edition
- [ ] C. Business Critical Edition
- [ ] D. Standard Edition

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Business Critical Edition

**Explanation:**
**Tri-Secret Secure** is available starting with **Business Critical Edition** (and above). It composes a customer-managed key (held in a cloud KMS) with a Snowflake-maintained key to form a composite master key, giving the customer the ability to revoke access to their data by disabling their key.
</details>

---

### Question 7
**Domain:** Domain 1 — Architecture

A table has clustering applied on column `order_date`. After several months of continuous DML, the table's clustering depth metric rises significantly. What does this indicate?

- [ ] A. Snowflake has automatically re-clustered the table and depth simply reflects the new partition count after compaction.
- [ ] B. The overlap between micro-partitions with respect to the clustering key has increased, meaning queries filtering on order_date will likely scan more partitions than necessary.
- [ ] C. The table now has more total micro-partitions than rows, indicating excessive fragmentation.
- [ ] D. The column order_date has exceeded its maximum cardinality threshold and must be re-typed.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The overlap between micro-partitions with respect to the clustering key has increased, meaning queries filtering on order_date will likely scan more partitions than necessary.

**Explanation:**
**Clustering depth** measures the average overlap of micro-partitions for a given clustering key. As DML accumulates, partitions can become less well-organized with respect to the key, increasing overlap (depth). A higher depth means range-filtered queries on that key will need to scan more partitions, since the same value ranges are now spread across more, overlapping partitions.
</details>

---

### Question 8
**Domain:** Domain 1 — Architecture

What is the primary difference between a Snowflake Database and a Snowflake Schema in terms of the object hierarchy?

- [ ] A. A database is a logical container that exists inside a schema, and an account can contain only one schema.
- [ ] B. A schema spans multiple databases so that objects can be shared across databases without cross-database references.
- [ ] C. A schema is a logical container of tables and other objects that exists inside a database, which itself exists inside an account.
- [ ] D. Databases and schemas are interchangeable terms in Snowflake and can be used in any order in fully qualified object names.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. A schema is a logical container of tables and other objects that exists inside a database, which itself exists inside an account.

**Explanation:**
Snowflake's object hierarchy is **Account → Database → Schema → Object** (tables, views, etc.). A database is a top-level logical container, and each database holds one or more schemas, which in turn organize the actual data objects. Fully qualified names follow `database.schema.object`.
</details>

---

### Question 9
**Domain:** Domain 1 — Architecture

A Snowflake account uses Database Replication to maintain a secondary database in another region for disaster recovery. Which of the following is replicated automatically as part of that database replication, without any extra configuration?

- [ ] A. Virtual warehouses defined in the primary account.
- [ ] B. The structure and data of tables, views, and other schema-level objects within the replicated database.
- [ ] C. Network policies attached to the account.
- [ ] D. Resource monitors associated with the account.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The structure and data of tables, views, and other schema-level objects within the replicated database.

**Explanation:**
Database replication copies the **database-level objects** — tables, views, schemas, stored procedures, and their data — to the secondary database. Account-level objects like **virtual warehouses, network policies, and resource monitors** are not part of database replication; replicating those requires the broader **account replication / failover groups** feature instead.
</details>

---

### Question 10
**Domain:** Domain 1 — Architecture

A user runs `SELECT * FROM my_table AT (OFFSET => -3600)`. What does this query return?

- [ ] A. The state of my_table as it existed 3600 seconds (1 hour) before the current time, using Time Travel.
- [ ] B. An error, because OFFSET can only be used with negative values when combined with the BEFORE clause.
- [ ] C. The current state of my_table as it exists right now, ignoring the OFFSET clause for SELECT statements.
- [ ] D. The state of my_table as it existed exactly 3600 rows prior to the most recent INSERT operation.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. The state of my_table as it existed 3600 seconds (1 hour) before the current time, using Time Travel.

**Explanation:**
The `AT (OFFSET => seconds)` clause is part of Snowflake's **Time Travel** syntax. A negative offset represents seconds **in the past relative to the current time**, so `OFFSET => -3600` reconstructs the table as it looked **one hour ago**, provided that point in time falls within the table's Time Travel retention period.
</details>

---

### Question 11
**Domain:** Domain 1 — Architecture

Which statement about Snowflake's separation of the Cloud Services layer from the Compute layer is accurate?

- [ ] A. The Cloud Services layer is billed per second of warehouse uptime, identical to the Compute layer's billing model.
- [ ] B. The Cloud Services layer manages metadata, authentication, query optimization, and infrastructure coordination, while the Compute layer (virtual warehouses) executes the actual query workload.
- [ ] C. The Cloud Services layer is optional and can be disabled for accounts that only run batch ETL jobs.
- [ ] D. The Cloud Services layer handles query execution while the Compute layer only stores metadata about completed queries.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The Cloud Services layer manages metadata, authentication, query optimization, and infrastructure coordination, while the Compute layer (virtual warehouses) executes the actual query workload.

**Explanation:**
The **Cloud Services layer** is a collection of services that coordinate the platform: authentication, infrastructure management, metadata management, query parsing/optimization, and access control. The **Compute layer**, made up of virtual warehouses, is where the actual SQL execution against the data happens. These two layers scale and bill independently of each other.
</details>

---

### Question 12
**Domain:** Domain 1 — Architecture

An account administrator wants to guarantee that failover to a secondary account happens automatically without manual intervention if the primary region experiences an outage. Which Snowflake feature provides this?

- [ ] A. Resource monitors configured with a NOTIFY_AND_SUSPEND action on the primary account's warehouses.
- [ ] B. Business Continuity & Disaster Recovery using failover groups together with Client Redirect, which allows applications to reconnect to the secondary using a single redirect connection URL.
- [ ] C. Client Redirect combined with a connection URL that points only to the primary account, with manual DNS updates during failover.
- [ ] D. Standard database replication, which automatically redirects all client connections during an outage.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Business Continuity & Disaster Recovery using failover groups together with Client Redirect, which allows applications to reconnect to the secondary using a single redirect connection URL.

**Explanation:**
Snowflake's **Business Continuity and Disaster Recovery** offering combines **failover groups** (which replicate databases and account-level objects together) with **Client Redirect**, a feature that lets client applications connect through a single redirect URL that can be repointed to the secondary account during failover, minimizing manual reconfiguration of every client.
</details>

---

### Question 13
**Domain:** Domain 1 — Architecture

What happens to the data in a temporary table when the session that created it ends?

- [ ] A. The table is moved into Time Travel storage where it can be recovered for up to 7 days.
- [ ] B. The table is automatically converted into a transient table to preserve the data for Fail-safe purposes.
- [ ] C. The table and its data persist until explicitly dropped, just like a permanent table.
- [ ] D. The table and all its data are immediately and permanently dropped when the session terminates.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. The table and all its data are immediately and permanently dropped when the session terminates.

**Explanation:**
A **temporary table** exists only for the duration of the session that created it. As soon as that session ends (logout, disconnect, or expiry), Snowflake automatically and permanently drops the table along with all of its data — there is no Fail-safe and no recovery option after the session closes.
</details>

---

### Question 14
**Domain:** Domain 1 — Architecture

A provider wants to monetize a dataset by listing it on the Snowflake Marketplace so consumers can browse and request access without the provider manually granting each consumer. Which object type enables this discoverability?

- [ ] A. A materialized view exposed through a public schema with PUBLIC role access.
- [ ] B. An external table pointing to a publicly readable cloud storage bucket.
- [ ] C. A listing, which packages one or more shares with metadata, usage terms, and optional pricing for marketplace discovery.
- [ ] D. A direct share created with CREATE SHARE and manually granted per consumer account.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. A listing, which packages one or more shares with metadata, usage terms, and optional pricing for marketplace discovery.

**Explanation:**
A **listing** is the Marketplace-facing object that wraps a share (or shares) together with descriptive metadata, refresh schedule, sample queries, and optionally pricing/usage terms, so that the data product becomes discoverable and requestable through the **Snowflake Marketplace** rather than requiring the provider to manually grant access to each consumer ahead of time.
</details>

---

### Question 15
**Domain:** Domain 1 — Architecture

Which of the following best describes a 'reader account' in the context of Snowflake data sharing?

- [ ] A. A full-featured Snowflake account created by a consumer organization that already has its own Snowflake licensing agreement.
- [ ] B. An account created and managed by a data provider on behalf of a consumer who does not have a Snowflake account, allowing that consumer to query shared data while the provider bears the compute cost.
- [ ] C. An account type that can only execute SELECT statements against its own locally stored permanent tables.
- [ ] D. A read-only replica of the provider's account used exclusively for Business Continuity failover testing.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. An account created and managed by a data provider on behalf of a consumer who does not have a Snowflake account, allowing that consumer to query shared data while the provider bears the compute cost.

**Explanation:**
A **reader account** lets a data provider share data with a consumer organization that does **not** have its own Snowflake account. The provider creates and administers the reader account, and the provider's own virtual warehouses (or the reader account's own warehouses, billed back to the provider) are used to query the data, since the reader account has no independent Snowflake contract.
</details>

---

### Question 16
**Domain:** Domain 1 — Architecture

What is the maximum Time Travel retention period available for permanent tables, and which edition is required to access the upper end of that range?

- [ ] A. 7 days maximum regardless of edition, since Time Travel duration is not edition-dependent.
- [ ] B. 1 day maximum, available on Standard Edition only.
- [ ] C. 365 days maximum, available starting with Business Critical Edition.
- [ ] D. 90 days maximum, available starting with Enterprise Edition (Standard Edition caps at 1 day).

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. 90 days maximum, available starting with Enterprise Edition (Standard Edition caps at 1 day).

**Explanation:**
**Standard Edition** allows up to **1 day** of Time Travel retention. **Enterprise Edition and above** allow configuring retention up to **90 days** for permanent tables (the default is still 1 day unless changed). This extended retention is a key differentiator that justifies upgrading from Standard to Enterprise for compliance-sensitive workloads.
</details>

---

### Question 17
**Domain:** Domain 1 — Architecture

A schema is created with MANAGED ACCESS enabled (`CREATE SCHEMA my_schema WITH MANAGED ACCESS`). What changes about object privilege grants within that schema?

- [ ] A. All future objects created in the schema automatically inherit PUBLIC role access by default.
- [ ] B. Managed access schemas disable Time Travel for every object created within them.
- [ ] C. Object owners retain full grant authority, but only for SELECT privileges; all other privilege types must go through the schema owner.
- [ ] D. Only the schema owner can grant privileges on objects within the schema; object owners can no longer grant privileges on objects they create.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. Only the schema owner can grant privileges on objects within the schema; object owners can no longer grant privileges on objects they create.

**Explanation:**
In a **managed access schema**, the authority to grant privileges on objects within the schema is centralized with the **schema owner** (or roles with MANAGE GRANTS), rather than each individual object owner. This prevents object creators from independently granting access to others, which is useful for centralizing governance.
</details>

---

### Question 18
**Domain:** Domain 1 — Architecture

How does Snowflake's automatic micro-partitioning differ from traditional static partitioning schemes used in many on-premises databases?

- [ ] A. Snowflake automatically and transparently divides table data into small, immutable, contiguous units, with no manual partition key design required by the user.
- [ ] B. Snowflake creates exactly one partition per virtual warehouse, with partition count fixed at warehouse creation time.
- [ ] C. Snowflake requires the user to manually define partition boundaries based on a chosen column before any data is loaded.
- [ ] D. Snowflake partitions are mutable and updated in place row-by-row, similar to traditional heap tables.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Snowflake automatically and transparently divides table data into small, immutable, contiguous units, with no manual partition key design required by the user.

**Explanation:**
Unlike traditional databases where administrators must choose and maintain explicit partition keys, Snowflake **automatically** divides table data into **micro-partitions** (roughly 50–500 MB of uncompressed data each) as data is loaded, with no manual partitioning scheme required. These micro-partitions are immutable; any DML produces new micro-partitions rather than modifying existing ones in place.
</details>

---

### Question 19
**Domain:** Domain 1 — Architecture

What is the relationship between a Snowflake 'Account' and a Snowflake 'Organization'?

- [ ] A. An organization can only exist if all member accounts share the exact same region and cloud platform.
- [ ] B. An organization and an account are synonymous terms used interchangeably in the documentation.
- [ ] C. An organization is a top-level construct that groups multiple related Snowflake accounts together for centralized billing, usage monitoring, and account management.
- [ ] D. An organization is a single account that has been upgraded to Business Critical Edition.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. An organization is a top-level construct that groups multiple related Snowflake accounts together for centralized billing, usage monitoring, and account management.

**Explanation:**
A Snowflake **Organization** is a higher-level container that links together one or more **accounts** belonging to the same business entity, enabling centralized visibility into usage and billing across accounts, as well as features like account creation and management from a single ORGADMIN role, even when the underlying accounts span different regions or cloud platforms.
</details>

---

### Question 20
**Domain:** Domain 1 — Architecture

A query references a view that was created with the SECURE keyword. What practical effect does this have compared to a regular (non-secure) view?

- [ ] A. Secure views automatically encrypt the underlying table data with a customer-managed key.
- [ ] B. Secure views hide the view's internal SQL definition from non-owning roles and limit the optimizer's use of certain query rewrites that could otherwise leak data through query plans.
- [ ] C. Secure views can only be queried by the ACCOUNTADMIN role, regardless of other privilege grants.
- [ ] D. Secure views automatically convert to materialized views to improve performance for shared consumers.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Secure views hide the view's internal SQL definition from non-owning roles and limit the optimizer's use of certain query rewrites that could otherwise leak data through query plans.

**Explanation:**
A **secure view** restricts visibility of its definition (via `SHOW VIEWS` / `GET_DDL`, etc.) to authorized roles and limits the query optimizer from applying certain optimizations (such as pushing predicates in ways that could expose underlying data through timing or error-based side channels). Secure views are required when sharing views through Secure Data Sharing to prevent data leakage.
</details>

---

### Question 21
**Domain:** Domain 1 — Architecture

Which statement correctly distinguishes Snowflake's Fail-safe from Time Travel?

- [ ] A. Fail-safe is user-accessible via the AT/BEFORE clause, while Time Travel requires opening a support case.
- [ ] B. Fail-safe and Time Travel cover the exact same retention window but differ only in storage cost.
- [ ] C. Fail-safe applies only to transient and temporary tables, while Time Travel applies only to permanent tables.
- [ ] D. Fail-safe is a non-configurable 7-day period that begins after Time Travel ends, and recovery requires contacting Snowflake Support rather than self-service SQL.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. Fail-safe is a non-configurable 7-day period that begins after Time Travel ends, and recovery requires contacting Snowflake Support rather than self-service SQL.

**Explanation:**
**Fail-safe** is a fixed, non-configurable **7-day** period that begins immediately after a permanent table's Time Travel retention period ends. Unlike Time Travel, which users can query directly with `AT`/`BEFORE` clauses, data recovery during Fail-safe requires **contacting Snowflake Support**, and it exists purely as a last-resort disaster-recovery mechanism, not a self-service feature.
</details>

---

### Question 22
**Domain:** Domain 1 — Architecture

A consumer account receives a share and creates a local database from it using `CREATE DATABASE my_db FROM SHARE provider.share_name`. Can the consumer write new rows into the shared tables?

- [ ] A. Yes, as long as the consumer's role has been granted INSERT privilege by the provider.
- [ ] B. Yes, but only during the first 24 hours after the share is consumed, after which it reverts to read-only.
- [ ] C. Yes, but only into tables that were explicitly marked as WRITABLE when the share was created.
- [ ] D. No, shared data is always read-only to the consumer regardless of any privileges granted.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. No, shared data is always read-only to the consumer regardless of any privileges granted.

**Explanation:**
Data accessed through **Secure Data Sharing** is always **read-only** for the consumer. The provider retains full ownership of the underlying data, and consumers can query it but cannot insert, update, or delete rows in shared objects, regardless of any role or privilege configuration on the consumer side.
</details>

---

### Question 23
**Domain:** Domain 1 — Architecture

What triggers the creation of a new micro-partition version when a single row in an existing table is updated?

- [ ] A. Snowflake marks the entire micro-partition containing that row as stale and writes a new micro-partition containing all the rows from the original partition, with the updated row reflected.
- [ ] B. Snowflake defers the write until the next scheduled re-clustering job runs.
- [ ] C. Snowflake creates a brand-new table copy containing every row from every partition in the table.
- [ ] D. Snowflake updates only the specific row in place within its existing micro-partition, leaving the rest of the partition untouched.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Snowflake marks the entire micro-partition containing that row as stale and writes a new micro-partition containing all the rows from the original partition, with the updated row reflected.

**Explanation:**
Because micro-partitions are **immutable**, updating even a single row requires Snowflake to write an entirely new micro-partition that includes all rows from the original partition (with the changed row reflected), and mark the old micro-partition version as no longer current. The old version is retained for Time Travel/Fail-safe purposes until it ages out.
</details>

---

### Question 24
**Domain:** Domain 1 — Architecture

Which of the following is a distinguishing characteristic of Virtual Private Snowflake (VPS) compared to Business Critical Edition?

- [ ] A. VPS is the only edition that supports HIPAA-compliant workloads.
- [ ] B. VPS removes the requirement for network policies, since all traffic is automatically considered trusted.
- [ ] C. VPS includes Tri-Secret Secure while Business Critical does not support customer-managed keys at all.
- [ ] D. VPS provisions a completely separate, dedicated set of compute resources, including the Cloud Services layer, isolated from the multi-tenant environment used by all other editions.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. VPS provisions a completely separate, dedicated set of compute resources, including the Cloud Services layer, isolated from the multi-tenant environment used by all other editions.

**Explanation:**
**Virtual Private Snowflake (VPS)** is the highest edition tier, providing a completely separate Snowflake environment with **dedicated infrastructure** — including a dedicated Cloud Services layer — isolated from the metadata store shared by all other multi-tenant accounts. Business Critical Edition shares the multi-tenant Cloud Services layer but offers strong security controls like Tri-Secret Secure.
</details>

---

### Question 25
**Domain:** Domain 1 — Architecture

A developer wants to query the historical state of a table as it existed immediately before a specific DML statement, identified by its query ID. Which Time Travel clause is appropriate?

- [ ] A. BEFORE (STATEMENT => '<query_id>')
- [ ] B. AT (OFFSET => 0)
- [ ] C. AT (STATEMENT => '<query_id>')
- [ ] D. AT (TIMESTAMP => CURRENT_TIMESTAMP())

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. BEFORE (STATEMENT => '<query_id>')

**Explanation:**
The `BEFORE (STATEMENT => '<query_id>')` clause reconstructs the table's state **immediately prior to** the execution of the specified query, which is exactly what's needed to inspect data right before a particular DML operation ran. `AT (STATEMENT => ...)` would instead include the effects of that statement.
</details>

---

### Question 26
**Domain:** Domain 2 — Virtual Warehouses

A virtual warehouse is configured with AUTO_SUSPEND = 60 and AUTO_RESUME = TRUE. After a query finishes and the warehouse sits idle, what happens 60 seconds later, and what happens when a new query arrives afterward?

- [ ] A. The warehouse suspends after 60 seconds of inactivity and automatically resumes the next time a query is submitted to it.
- [ ] B. The warehouse reduces to its minimum cluster size after 60 seconds but never fully suspends.
- [ ] C. The warehouse is dropped after 60 seconds and must be manually recreated before the next query can run.
- [ ] D. The warehouse only suspends if AUTO_RESUME is also set to FALSE; otherwise it remains running indefinitely.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. The warehouse suspends after 60 seconds of inactivity and automatically resumes the next time a query is submitted to it.

**Explanation:**
With `AUTO_SUSPEND = 60`, the warehouse automatically **suspends** (stops billing) after being idle for 60 seconds. Because `AUTO_RESUME = TRUE`, the next time a query needs that warehouse, Snowflake automatically **resumes** it, typically within a few seconds, without requiring manual intervention.
</details>

---

### Question 27
**Domain:** Domain 2 — Virtual Warehouses

A multi-cluster warehouse is set to AUTO_SCALE mode with MIN_CLUSTER_COUNT = 1 and MAX_CLUSTER_COUNT = 4. What triggers Snowflake to spin up an additional cluster?

- [ ] A. The number of distinct users connected to the warehouse exceeding 10.
- [ ] B. The warehouse size (T-shirt size) being increased by an administrator.
- [ ] C. Queuing of queries because the currently running clusters cannot process the incoming concurrent query load fast enough.
- [ ] D. A single query that takes longer than the statement timeout threshold.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Queuing of queries because the currently running clusters cannot process the incoming concurrent query load fast enough.

**Explanation:**
In **multi-cluster auto-scale mode**, Snowflake monitors for query **queuing** due to concurrency pressure. When existing clusters can't keep up with simultaneous query load and queries start queuing, Snowflake automatically starts additional clusters (up to MAX_CLUSTER_COUNT) to absorb the extra concurrency, then scales back down as load decreases.
</details>

---

### Question 28
**Domain:** Domain 2 — Virtual Warehouses

Resizing a running virtual warehouse from Medium to Large while queries are actively executing on it has what effect on those already-running queries?

- [ ] A. The in-flight queries continue running on the original-size compute resources; only new queries submitted after the resize benefit from the larger size.
- [ ] B. Resizing a warehouse is not possible while any query is actively running against it.
- [ ] C. The in-flight queries are immediately killed and must be resubmitted to benefit from the larger size.
- [ ] D. The in-flight queries are automatically paused, migrated to the new compute resources mid-execution, and resumed.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. The in-flight queries continue running on the original-size compute resources; only new queries submitted after the resize benefit from the larger size.

**Explanation:**
Resizing a warehouse is **non-disruptive** to currently executing queries — those queries continue running against the original compute resources they started on. The resize takes effect for **new queries** submitted after the resize completes, allowing administrators to scale up or down without interrupting in-flight work.
</details>

---

### Question 29
**Domain:** Domain 2 — Virtual Warehouses

Which warehouse scaling policy, available for multi-cluster warehouses, prioritizes starting additional clusters quickly to minimize queuing, even if it means running with idle compute briefly?

- [ ] A. OPTIMIZED
- [ ] B. ECONOMY
- [ ] C. STANDARD
- [ ] D. MINIMAL

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. STANDARD

**Explanation:**
The **STANDARD** scaling policy favors starting additional clusters quickly in response to increased load, prioritizing query performance and reduced queuing over compute efficiency, even if that occasionally means a newly started cluster is briefly underutilized. **ECONOMY**, by contrast, waits longer and tries to fully utilize existing clusters before adding new ones, conserving credits at the cost of potential queuing.
</details>

---

### Question 30
**Domain:** Domain 2 — Virtual Warehouses

What unit does Snowflake use to bill virtual warehouse compute usage, and at what minimum increment?

- [ ] A. Storage GB-months, billed in 1 GB increments.
- [ ] B. Credits, billed with a 60-second minimum each time the warehouse starts, then billed per second thereafter.
- [ ] C. Flat monthly subscription fee independent of actual warehouse runtime.
- [ ] D. Credits, billed only in full-hour increments regardless of actual usage duration.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Credits, billed with a 60-second minimum each time the warehouse starts, then billed per second thereafter.

**Explanation:**
Virtual warehouse compute is billed in **credits**, consumed based on warehouse size and runtime. Each time a warehouse starts (or resumes from suspension), there is a **60-second minimum** billing charge, after which usage is billed **per-second** for as long as the warehouse remains running.
</details>

---

### Question 31
**Domain:** Domain 2 — Virtual Warehouses

A team wants queries from their ETL pipeline to never compete for compute resources with queries from their BI dashboard tool, even though both run constantly throughout the day. What is the most direct way to achieve this?

- [ ] A. Enable multi-cluster mode on a single warehouse, since each cluster automatically routes queries by source application.
- [ ] B. Create a single warehouse and rely on Snowflake's internal query prioritization to separate the workloads automatically.
- [ ] C. Increase the warehouse size to XXL so there is enough compute headroom for both workloads simultaneously.
- [ ] D. Provision two separate virtual warehouses — one dedicated to ETL and one dedicated to BI — so each workload has its own independent compute.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. Provision two separate virtual warehouses — one dedicated to ETL and one dedicated to BI — so each workload has its own independent compute.

**Explanation:**
Because virtual warehouses are **independent compute clusters**, the standard approach to isolating workloads is to use **separate warehouses** for each workload (e.g., one for ETL, one for BI). This guarantees that resource contention in one workload (like a heavy ETL load) cannot starve or slow down queries running in the other, since the warehouses don't share compute.
</details>

---

### Question 32
**Domain:** Domain 2 — Virtual Warehouses

A warehouse is sized XS (X-Small). According to Snowflake's credit consumption model, approximately how many credits per hour does an XS warehouse consume when running continuously, relative to a Small warehouse?

- [ ] A. An XS warehouse consumes 4 credits/hour, double the 2 credits/hour of a Small warehouse.
- [ ] B. An XS warehouse consumes 8 credits/hour because base warehouse sizing starts at a higher tier than Small.
- [ ] C. Both XS and Small warehouses consume the same 1 credit/hour, since sizing only affects storage I/O.
- [ ] D. An XS warehouse consumes 1 credit/hour, exactly half the 2 credits/hour of a Small warehouse, since each size doubles the credit rate of the previous one.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. An XS warehouse consumes 1 credit/hour, exactly half the 2 credits/hour of a Small warehouse, since each size doubles the credit rate of the previous one.

**Explanation:**
Snowflake warehouse sizes follow a doubling pattern in both compute power and credit consumption: **X-Small = 1 credit/hour**, Small = 2 credits/hour, Medium = 4 credits/hour, Large = 8 credits/hour, and so on. Each size up roughly doubles both the number of compute nodes and the credit consumption rate.
</details>

---

### Question 33
**Domain:** Domain 2 — Virtual Warehouses

A resource monitor is created with a credit quota and the action `SUSPEND_IMMEDIATE` set at 100% of quota. What is the effect of SUSPEND_IMMEDIATE compared to a plain SUSPEND action when the threshold is reached?

- [ ] A. SUSPEND_IMMEDIATE waits for currently running queries to complete before suspending the warehouse, while SUSPEND cancels them right away.
- [ ] B. SUSPEND_IMMEDIATE cancels all currently running queries and suspends the warehouse right away, while SUSPEND lets in-progress queries finish before suspending.
- [ ] C. There is no functional difference; both actions behave identically and only differ in the notification message sent.
- [ ] D. SUSPEND_IMMEDIATE only applies to multi-cluster warehouses, while SUSPEND applies only to single-cluster warehouses.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. SUSPEND_IMMEDIATE cancels all currently running queries and suspends the warehouse right away, while SUSPEND lets in-progress queries finish before suspending.

**Explanation:**
With a plain **SUSPEND** action, the warehouse stops accepting new queries but allows already-running queries to **finish** before suspending. **SUSPEND_IMMEDIATE** is more aggressive: it **cancels all currently running queries** and suspends the warehouse right away, which can be useful for hard credit-quota enforcement but risks interrupting in-flight work.
</details>

---

### Question 34
**Domain:** Domain 2 — Virtual Warehouses

A multi-cluster warehouse is configured in MAXIMIZED mode with MIN_CLUSTER_COUNT = 3 and MAX_CLUSTER_COUNT = 3. What does this configuration achieve?

- [ ] A. All 3 clusters run continuously at all times whenever the warehouse is started, providing constant maximum concurrency capacity regardless of current load.
- [ ] B. The configuration is invalid because MIN_CLUSTER_COUNT must always be less than MAX_CLUSTER_COUNT.
- [ ] C. The warehouse behaves exactly like auto-scale mode but is capped at 3 clusters maximum during peak load.
- [ ] D. Only 1 cluster runs by default, and the other 2 are provisioned but kept in a suspended standby state.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. All 3 clusters run continuously at all times whenever the warehouse is started, providing constant maximum concurrency capacity regardless of current load.

**Explanation:**
Setting `MIN_CLUSTER_COUNT` equal to `MAX_CLUSTER_COUNT` (with both greater than 1) effectively puts the warehouse in **maximized mode**: all clusters start together and run continuously whenever the warehouse is active, providing a fixed, constant level of concurrency capacity rather than scaling dynamically with load.
</details>

---

### Question 35
**Domain:** Domain 2 — Virtual Warehouses

Which workload characteristic is the strongest indicator that a query would benefit from a LARGER warehouse size rather than simply waiting longer on the current size?

- [ ] A. The query itself is scanning and processing a very large volume of data and its execution plan shows heavy local disk spilling.
- [ ] B. The query was submitted by a user with a role that has limited privileges on the target table.
- [ ] C. The query is queuing behind many other concurrent queries on a busy warehouse.
- [ ] D. The query references a transient table rather than a permanent table.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. The query itself is scanning and processing a very large volume of data and its execution plan shows heavy local disk spilling.

**Explanation:**
Increasing **warehouse size** adds more compute nodes and memory/local SSD per query, which helps when an individual query is **compute- or memory-bound** — for example, scanning huge data volumes or **spilling to local/remote disk** due to insufficient memory. Queuing due to many concurrent queries, by contrast, is better solved by adding **clusters** (multi-cluster/auto-scale), not by simply resizing up.
</details>

---

### Question 36
**Domain:** Domain 2 — Virtual Warehouses

A warehouse has STATEMENT_TIMEOUT_IN_SECONDS set to 3600 at the warehouse level, but a particular session sets STATEMENT_TIMEOUT_IN_SECONDS to 1800 at the session level. Which timeout applies to a query run in that session?

- [ ] A. The session-level timeout of 1800 seconds applies, since session-level parameter settings override warehouse-level defaults for that session.
- [ ] B. Both timeouts are summed, resulting in an effective timeout of 5400 seconds.
- [ ] C. Snowflake ignores both settings and uses the account-level default of 2 days.
- [ ] D. The warehouse-level timeout of 3600 seconds always takes precedence over any session-level setting.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. The session-level timeout of 1800 seconds applies, since session-level parameter settings override warehouse-level defaults for that session.

**Explanation:**
Snowflake parameters follow a hierarchy where more specific scopes override broader ones: **session-level** parameter settings take precedence over **warehouse-level** settings, which in turn override account-level defaults. Since the session explicitly set 1800 seconds, that value applies to queries in that session.
</details>

---

### Question 37
**Domain:** Domain 2 — Virtual Warehouses

What is the key difference between scaling a warehouse 'up' (resizing) versus scaling it 'out' (multi-cluster)?

- [ ] A. Scaling up increases the compute power of a single cluster to handle bigger/heavier individual queries; scaling out adds more clusters of the same size to handle more concurrent queries.
- [ ] B. Scaling up adds more virtual warehouses of the same size; scaling out increases the size of a single warehouse.
- [ ] C. Scaling up only applies to Standard Edition accounts, while scaling out is exclusive to Enterprise Edition and above.
- [ ] D. Scaling up and scaling out are two names for the exact same underlying mechanism in Snowflake.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Scaling up increases the compute power of a single cluster to handle bigger/heavier individual queries; scaling out adds more clusters of the same size to handle more concurrent queries.

**Explanation:**
**Scaling up** (resizing, e.g., Medium → Large) increases the size/power of a single warehouse cluster, which helps with large or complex individual queries. **Scaling out** (multi-cluster warehouses) adds additional clusters of the same size running in parallel, which helps absorb higher **concurrency** — many simultaneous queries — rather than speeding up any one query.
</details>

---

### Question 38
**Domain:** Domain 2 — Virtual Warehouses

A query running on a warehouse begins to spill intermediate results to local disk, and then further spills to remote cloud storage. What does this spilling pattern indicate about the query relative to the warehouse?

- [ ] A. The query's working set exceeds the memory available on the warehouse, and even local SSD capacity, forcing slower remote storage I/O.
- [ ] B. The query has encountered a syntax error that Snowflake recovers from by writing partial results to storage.
- [ ] C. The query is using clustering keys inefficiently, which always causes remote spilling regardless of warehouse size.
- [ ] D. Remote spilling is the expected, optimal behavior for any aggregation query and does not indicate a sizing problem.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. The query's working set exceeds the memory available on the warehouse, and even local SSD capacity, forcing slower remote storage I/O.

**Explanation:**
**Spilling** occurs when a query's intermediate working set doesn't fit in the warehouse's available memory. Snowflake first spills to **local SSD**; if that's also insufficient, it spills further to **remote cloud storage**, which is significantly slower. Remote spilling, in particular, is a strong signal the query would benefit from a **larger warehouse** with more memory/local storage.
</details>

---

### Question 39
**Domain:** Domain 2 — Virtual Warehouses

By default, when a virtual warehouse is created without specifying AUTO_SUSPEND, what value does Snowflake apply?

- [ ] A. AUTO_SUSPEND defaults to 600 seconds (10 minutes) of inactivity.
- [ ] B. AUTO_SUSPEND defaults to 0, meaning the warehouse never automatically suspends.
- [ ] C. AUTO_SUSPEND must always be explicitly specified; warehouse creation fails without it.
- [ ] D. AUTO_SUSPEND defaults to 86400 seconds (24 hours) of inactivity.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. AUTO_SUSPEND defaults to 600 seconds (10 minutes) of inactivity.

**Explanation:**
If not explicitly set, Snowflake applies a default `AUTO_SUSPEND` of **600 seconds (10 minutes)** of inactivity for newly created warehouses, after which the warehouse automatically suspends to stop incurring further credit usage. Administrators commonly tighten this further (e.g., to 60 seconds) to minimize idle-time billing.
</details>

---

### Question 40
**Domain:** Domain 2 — Virtual Warehouses

Which scenario is the best fit for using a multi-cluster warehouse in AUTO_SCALE mode rather than a single-cluster warehouse?

- [ ] A. A development warehouse used by a single engineer running ad-hoc exploratory queries.
- [ ] B. A customer-facing BI dashboard queried simultaneously by hundreds of analysts throughout business hours, with unpredictable concurrent load.
- [ ] C. A one-time historical data backfill that runs a single COPY INTO statement.
- [ ] D. A nightly batch job that runs one large, long-running query with no other concurrent activity.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. A customer-facing BI dashboard queried simultaneously by hundreds of analysts throughout business hours, with unpredictable concurrent load.

**Explanation:**
Multi-cluster auto-scale warehouses exist specifically to absorb **unpredictable, high-concurrency workloads** — such as many simultaneous dashboard users — by automatically starting additional clusters when query queuing is detected and scaling back down as concurrency drops. Single, long-running batch jobs or single-user workloads don't benefit from multi-cluster scaling since concurrency isn't the bottleneck.
</details>

---

### Question 41
**Domain:** Domain 2 — Virtual Warehouses

A virtual warehouse owner wants to ensure that, even during a sudden concurrency spike, costs never exceed what 5 running clusters would cost, while still allowing automatic scale-down during quiet periods. Which configuration achieves this?

- [ ] A. AUTO_SUSPEND = 0 with a single-cluster warehouse sized at 5X-Large
- [ ] B. MIN_CLUSTER_COUNT = 1, MAX_CLUSTER_COUNT = 5, SCALING_POLICY = STANDARD or ECONOMY
- [ ] C. MIN_CLUSTER_COUNT = 5, MAX_CLUSTER_COUNT = 5, SCALING_POLICY = STANDARD
- [ ] D. MIN_CLUSTER_COUNT = 1, MAX_CLUSTER_COUNT = 1, with a resource monitor capping credits at the 5-cluster equivalent

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. MIN_CLUSTER_COUNT = 1, MAX_CLUSTER_COUNT = 5, SCALING_POLICY = STANDARD or ECONOMY

**Explanation:**
Setting `MIN_CLUSTER_COUNT = 1` and `MAX_CLUSTER_COUNT = 5` puts the warehouse in **auto-scale mode**, allowing it to run as few as 1 cluster during quiet periods and scale up to a hard ceiling of 5 clusters during spikes — capping the maximum possible concurrent cost — while the chosen `SCALING_POLICY` (STANDARD or ECONOMY) tunes how aggressively new clusters are added.
</details>

---

### Question 42
**Domain:** Domain 2 — Virtual Warehouses

What happens to queries that are already queued on a warehouse when that warehouse is manually suspended via `ALTER WAREHOUSE ... SUSPEND`?

- [ ] A. All queued queries are silently dropped and must be resubmitted with no errors logged.
- [ ] B. Queued (not-yet-started) queries fail or remain queued indefinitely until the warehouse is resumed, while already-executing queries are allowed to finish before suspension completes.
- [ ] C. Queued queries are automatically rerouted to the next-available warehouse in the account.
- [ ] D. Manual suspension is blocked entirely by Snowflake if any query is queued or running.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Queued (not-yet-started) queries fail or remain queued indefinitely until the warehouse is resumed, while already-executing queries are allowed to finish before suspension completes.

**Explanation:**
A manual `SUSPEND` allows **currently executing** queries to complete, but newly queued queries that have not yet started will simply wait until the warehouse is resumed again (or eventually error depending on client-side timeout settings) — Snowflake does not automatically reroute them to another warehouse.
</details>

---

### Question 43
**Domain:** Domain 2 — Virtual Warehouses

An administrator notices that a Large warehouse used for ad-hoc analyst queries is rarely fully utilized and frequently sits with available compute capacity even while running. Which sizing adjustment is most appropriate to reduce cost without harming individual query performance, assuming concurrency is low?

- [ ] A. Disable AUTO_RESUME so the warehouse must be started manually each time, reducing idle billing.
- [ ] B. Decrease the warehouse size (e.g., to Medium or Small), since the workload doesn't need the extra per-query compute power that Large provides.
- [ ] C. Increase AUTO_SUSPEND so the warehouse stays running longer between queries.
- [ ] D. Switch to a multi-cluster warehouse with MAX_CLUSTER_COUNT increased, since the issue is concurrency-related.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Decrease the warehouse size (e.g., to Medium or Small), since the workload doesn't need the extra per-query compute power that Large provides.

**Explanation:**
When an individual warehouse consistently has unused compute capacity for the actual query workload (and concurrency/queuing isn't the issue), the appropriate fix is to **scale down** (reduce the T-shirt size), which lowers the credit consumption rate while typically still being sufficient for the lighter workload — rather than adjusting concurrency-oriented settings like multi-cluster count.
</details>

---

### Question 44
**Domain:** Domain 2 — Virtual Warehouses

Which of the following correctly describes how warehouse credit billing interacts with the local SSD cache when a warehouse is suspended and later resumed?

- [ ] A. Local SSD cache contents are billed separately as storage credits even while the warehouse is suspended.
- [ ] B. The local SSD cache persists across suspend/resume cycles as long as the warehouse is resumed on the exact same underlying compute nodes, but this is not guaranteed.
- [ ] C. Suspending a warehouse immediately and permanently destroys any cached data; resumed warehouses always start with a completely cold cache.
- [ ] D. The local SSD cache is permanently preserved in cloud storage during suspension and is always fully restored upon resume.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The local SSD cache persists across suspend/resume cycles as long as the warehouse is resumed on the exact same underlying compute nodes, but this is not guaranteed.

**Explanation:**
When a warehouse suspends, its compute nodes (and their local SSD caches) are released. Upon resume, Snowflake may provision the **same** underlying nodes (preserving some cache) or **different** nodes (resulting in a cold cache) — this is **not guaranteed** either way, so cache warmth after a resume is best-effort, not assured.
</details>

---

### Question 45
**Domain:** Domain 2 — Virtual Warehouses

A team wants to give read-only analysts the ability to start and stop a specific warehouse themselves, without granting them the ability to create or drop warehouses account-wide. Which privilege should be granted on that specific warehouse object?

- [ ] A. The OPERATE privilege on the warehouse, which allows starting, stopping, suspending, and resuming, without allowing structural changes like resizing.
- [ ] B. The MODIFY privilege on the warehouse, granted via the warehouse's role-based access control.
- [ ] C. The USAGE privilege alone, since USAGE inherently includes start/stop control over the warehouse.
- [ ] D. The OWNERSHIP privilege on the warehouse, which is the only privilege that allows suspend/resume actions.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. The OPERATE privilege on the warehouse, which allows starting, stopping, suspending, and resuming, without allowing structural changes like resizing.

**Explanation:**
The **OPERATE** privilege on a warehouse specifically allows a role to start, stop, suspend, and resume that warehouse, without granting the ability to alter its configuration (like size or scaling policy), which would require **MODIFY**, or full control, which would require **OWNERSHIP**. **USAGE** alone only allows the warehouse to be used for query execution, not operational control.
</details>

---

### Question 46
**Domain:** Domain 2 — Virtual Warehouses

A heavy, single complex join query is taking a long time to complete on a Small warehouse, and monitoring shows no spilling to disk and low queuing. The bottleneck is simply that one query needs more raw parallel compute. What is the most effective remediation?

- [ ] A. Lower AUTO_SUSPEND so the warehouse releases resources faster, freeing compute for the query.
- [ ] B. Add more clusters via multi-cluster auto-scale, since that increases total available compute for the query.
- [ ] C. Switch the warehouse to ECONOMY scaling policy to reduce credit consumption while the query runs.
- [ ] D. Increase the warehouse size (e.g., to Large or X-Large) so the single query can be parallelized across more compute nodes.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. Increase the warehouse size (e.g., to Large or X-Large) so the single query can be parallelized across more compute nodes.

**Explanation:**
Adding more **clusters** (multi-cluster) only helps when multiple queries are running concurrently and competing for resources — it does **not** speed up a single query, since each cluster processes separate queries independently. To give one individual query more parallel compute power, the correct lever is to **resize the warehouse up**, increasing the number of nodes available to that single query's execution.
</details>

---

### Question 47
**Domain:** Domain 3 — Storage & Protection

A file format object is created with `TYPE = CSV, FIELD_OPTIONALLY_ENCLOSED_BY = '"', SKIP_HEADER = 1`. During a COPY INTO load, a row contains a comma inside a quoted field. How does Snowflake handle this field during parsing?

- [ ] A. The quotes are stripped before delimiter parsing, causing the comma to always split the field regardless of FIELD_OPTIONALLY_ENCLOSED_BY.
- [ ] B. The entire row is rejected as malformed because commas are never allowed inside quoted fields.
- [ ] C. Snowflake treats the quoted field as a single value, correctly ignoring the delimiter character that appears inside the enclosing quotes.
- [ ] D. The comma inside quotes is treated as a field delimiter, splitting that value into two separate columns.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Snowflake treats the quoted field as a single value, correctly ignoring the delimiter character that appears inside the enclosing quotes.

**Explanation:**
With `FIELD_OPTIONALLY_ENCLOSED_BY` set, Snowflake recognizes that delimiters appearing **within the enclosing quote characters** are part of the field's value rather than a field separator. So a comma inside double quotes is correctly preserved as part of a single column value, not treated as splitting the row into extra columns.
</details>

---

### Question 48
**Domain:** Domain 3 — Storage & Protection

A Snowpipe is configured to auto-ingest files as they land in a cloud storage stage, using cloud provider event notifications. What happens if Snowpipe encounters a file that fails to parse due to a schema mismatch?

- [ ] A. The problematic file's rows are skipped or the file is recorded as an error (depending on ON_ERROR setting) while the pipe continues processing subsequent files.
- [ ] B. Snowpipe automatically alters the target table's schema to match the incoming file.
- [ ] C. The entire pipe is automatically and permanently disabled until manually re-enabled.
- [ ] D. All previously loaded files since pipe creation are automatically reprocessed to maintain consistency.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. The problematic file's rows are skipped or the file is recorded as an error (depending on ON_ERROR setting) while the pipe continues processing subsequent files.

**Explanation:**
Snowpipe continues operating continuously; when a file fails to load due to issues like schema mismatches, that **specific file's errors** are handled according to the pipe's `ON_ERROR` copy option (e.g., skipping bad rows or the whole file), and details are recorded in `COPY_HISTORY`/`PIPE_USAGE_HISTORY`, but the pipe itself **keeps running** for subsequent files rather than shutting down.
</details>

---

### Question 49
**Domain:** Domain 3 — Storage & Protection

What is the primary distinction between an internal stage and an external stage in Snowflake?

- [ ] A. Internal stages require a storage integration object, while external stages never need one.
- [ ] B. Internal stages can only hold CSV files, while external stages support any file format.
- [ ] C. Internal stages store files within Snowflake-managed cloud storage, while external stages reference files in a customer-managed cloud storage location (e.g., an S3 bucket) that Snowflake does not own.
- [ ] D. External stages are always temporary and disappear after 24 hours, while internal stages persist indefinitely.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Internal stages store files within Snowflake-managed cloud storage, while external stages reference files in a customer-managed cloud storage location (e.g., an S3 bucket) that Snowflake does not own.

**Explanation:**
An **internal stage** uses storage that Snowflake provisions and manages on the customer's behalf within Snowflake's own cloud storage. An **external stage** instead points to a storage location the customer already owns and manages directly (such as an S3 bucket, Azure container, or GCS bucket), with Snowflake reading/writing files there via a defined connection (often a storage integration).
</details>

---

### Question 50
**Domain:** Domain 3 — Storage & Protection

A storage integration object is created to allow Snowflake to securely access an external S3 bucket without embedding AWS credentials in the stage definition. What underlying AWS mechanism does this integration rely on?

- [ ] A. An IAM role with a trust policy that allows Snowflake's AWS account to assume that role, avoiding the need to store long-lived secret keys.
- [ ] B. A VPC peering connection between the customer's AWS account and Snowflake's AWS account.
- [ ] C. An AWS Lambda function that proxies all storage requests on Snowflake's behalf.
- [ ] D. A shared AWS root account access key stored encrypted within Snowflake's metadata.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. An IAM role with a trust policy that allows Snowflake's AWS account to assume that role, avoiding the need to store long-lived secret keys.

**Explanation:**
A **storage integration** configures an **IAM role** in the customer's AWS account with a trust relationship that permits Snowflake's AWS account to **assume that role** temporarily. This avoids embedding static AWS access keys/secret keys directly into stage or integration definitions, following AWS security best practices for cross-account access.
</details>

---

### Question 51
**Domain:** Domain 3 — Storage & Protection

A table column is defined with a MASKING POLICY that returns the full value for the ANALYST_FULL role but a redacted value for all other roles. A user with role ANALYST_FULL queries the table through a view owned by a different role that does not have ANALYST_FULL privileges. What value does the user see?

- [ ] A. The value depends entirely on whether the view is secure, with non-secure views always showing the full value.
- [ ] B. The redacted value, because masking policies always evaluate based on the view owner's role rather than the querying session's role.
- [ ] C. An error is thrown because masking policies cannot be applied to columns accessed through views.
- [ ] D. The full, unmasked value, because the masking policy evaluates based on the querying user's current active role regardless of the view owner.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The redacted value, because masking policies always evaluate based on the view owner's role rather than the querying session's role.

**Explanation:**
By default, when a masking policy is applied to a column and that column is accessed through a **view**, Snowflake evaluates the masking policy based on the **view owner's role** (the role that owns the view), not the querying user's role — unless the column-level security context is explicitly propagated. This is an important nuance that can lead to unexpected masking/unmasking behavior if not understood.
</details>

---

### Question 52
**Domain:** Domain 3 — Storage & Protection

What does the `VALIDATION_MODE` parameter do when used with a `COPY INTO <table>` statement?

- [ ] A. It performs a dry run that validates the data and reports errors without actually loading any rows into the target table.
- [ ] B. It enables row-level validation checks defined by a CHECK constraint on the table.
- [ ] C. It validates that the target table's schema matches the source file's schema before allowing any future loads.
- [ ] D. It permanently loads the data into the table only if zero errors are found across the entire file.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. It performs a dry run that validates the data and reports errors without actually loading any rows into the target table.

**Explanation:**
`VALIDATION_MODE` allows a `COPY INTO` statement to be run as a **dry run**: Snowflake parses and validates the specified number of rows (or the whole file) and reports any errors it would encounter, but it does **not** actually insert any rows into the target table. This is useful for catching format/schema issues before committing to a real load.
</details>

---

### Question 53
**Domain:** Domain 3 — Storage & Protection

A table is protected by a row access policy that restricts visibility based on a mapping table of role-to-region assignments. A user queries the table using `SELECT COUNT(*) FROM sales_table`. How does the row access policy affect this aggregate query?

- [ ] A. Row access policies only apply to SELECT * queries, so the COUNT(*) bypasses the policy and returns the true total.
- [ ] B. The row access policy is ignored unless the query also includes an explicit WHERE clause referencing the policy's mapping column.
- [ ] C. Aggregate functions automatically disable row access policies for performance reasons.
- [ ] D. The COUNT(*) reflects only the rows the user's role is permitted to see under the row access policy, just like any other query against that table.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. The COUNT(*) reflects only the rows the user's role is permitted to see under the row access policy, just like any other query against that table.

**Explanation:**
Row access policies apply to **all** queries against the protected table, including aggregates. `COUNT(*)` will only count rows that the row access policy permits the querying user's role to see — there's no special exemption for aggregate functions or for omitting explicit column references.
</details>

---

### Question 54
**Domain:** Domain 3 — Storage & Protection

A user wants to load semi-structured JSON data into a table column typed as VARIANT. Which COPY INTO file format setting is most relevant for correctly handling deeply nested JSON during the load?

- [ ] A. FIELD_DELIMITER, since JSON files are technically still delimited the same way as CSV.
- [ ] B. SKIP_HEADER, which determines how many JSON keys to ignore at the top of the document.
- [ ] C. ESCAPE_UNENCLOSED_FIELD, which only applies to fixed-width file formats.
- [ ] D. STRIP_OUTER_ARRAY, which controls whether an enclosing top-level array is removed so each element loads as its own row.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. STRIP_OUTER_ARRAY, which controls whether an enclosing top-level array is removed so each element loads as its own row.

**Explanation:**
For JSON loads, the `STRIP_OUTER_ARRAY` file format option determines whether Snowflake removes an outer `[ ]` array wrapper so that each element of the array becomes its **own row** in the VARIANT column, rather than loading the entire array as a single row's value. This is commonly needed when source JSON files represent a list of records as one big array.
</details>

---

### Question 55
**Domain:** Domain 3 — Storage & Protection

What is the function of Snowflake's automatic data encryption at rest, and at what point in the data lifecycle is it applied?

- [ ] A. Only data loaded via Snowpipe is automatically encrypted; data loaded via bulk COPY INTO is stored unencrypted by default.
- [ ] B. Data is encrypted only when explicitly requested via the ENCRYPT() SQL function on individual columns.
- [ ] C. Encryption at rest is an optional add-on that must be purchased separately from Snowflake regardless of edition.
- [ ] D. All customer data is automatically encrypted using AES-256 (or stronger) both at rest and in transit, with no action required by the customer to enable basic encryption.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. All customer data is automatically encrypted using AES-256 (or stronger) both at rest and in transit, with no action required by the customer to enable basic encryption.

**Explanation:**
Snowflake automatically encrypts **all** customer data **at rest** (using strong encryption such as AES-256) and **in transit**, by default, for every account regardless of edition, with no additional configuration required from the customer to get this baseline protection. Higher editions add further key-management options like Tri-Secret Secure.
</details>

---

### Question 56
**Domain:** Domain 3 — Storage & Protection

A pipe is created referencing a stage and a COPY INTO statement, but the underlying table's column structure is later altered to add a new NOT NULL column without a default. What happens to subsequent Snowpipe loads through that pipe?

- [ ] A. Snowpipe automatically detects the schema change and updates its internal copy statement to match.
- [ ] B. Snowpipe ignores NOT NULL constraints entirely, always inserting NULL regardless of the constraint.
- [ ] C. Loads through the pipe will begin failing for rows that don't supply a value for the new required column, since the pipe's COPY INTO logic isn't automatically rewritten.
- [ ] D. The pipe is automatically dropped and recreated by Snowflake whenever the target table's DDL changes.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Loads through the pipe will begin failing for rows that don't supply a value for the new required column, since the pipe's COPY INTO logic isn't automatically rewritten.

**Explanation:**
A pipe encapsulates a specific `COPY INTO` statement at creation time; it does not automatically adapt when the target table's schema changes. If a new **NOT NULL** column without a default is added and the pipe's copy logic doesn't supply a value for it, subsequent loads will start **failing** for those rows, since the implicit/explicit column mapping in the pipe is unaware of the new constraint.
</details>

---

### Question 57
**Domain:** Domain 3 — Storage & Protection

Which Snowflake feature allows compute-intensive search operations (point lookups and substring searches) on large tables to be dramatically accelerated by maintaining a specialized index structure, separate from clustering?

- [ ] A. Materialized views with an automatically maintained aggregate cache.
- [ ] B. Search Optimization Service, which builds and maintains a persistent search access path for equality/substring predicates.
- [ ] C. Result caching, which stores results of prior identical queries for 24 hours.
- [ ] D. Automatic clustering, which reorders micro-partitions based on a clustering key.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Search Optimization Service, which builds and maintains a persistent search access path for equality/substring predicates.

**Explanation:**
The **Search Optimization Service** builds and maintains a separate, persistent access structure optimized for fast point lookups, IN-list filters, and substring/LIKE searches against large tables — distinct from clustering keys, which optimize range-based pruning. It's especially useful for selective equality searches on high-cardinality columns in very large tables.
</details>

---

### Question 58
**Domain:** Domain 3 — Storage & Protection

A dynamic data masking policy applies to a SSN column. An unauthorized role queries `SELECT ssn FROM customers WHERE ssn = '123-45-6789'`. What does the masking policy typically prevent in this scenario, beyond simply masking the displayed output?

- [ ] A. The masking policy automatically rewrites the WHERE clause to compare against the masked value instead.
- [ ] B. The query is blocked entirely with a permissions error whenever a masked column appears anywhere in the query.
- [ ] C. Unauthorized roles cannot reference the masked column in a WHERE clause at all; only authorized roles can filter on it.
- [ ] D. Nothing additional — masking policies only affect SELECT output, so filtering on the raw unmasked value in a WHERE clause would still succeed for unauthorized roles, which is an important and frequently tested nuance.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. Nothing additional — masking policies only affect SELECT output, so filtering on the raw unmasked value in a WHERE clause would still succeed for unauthorized roles, which is an important and frequently tested nuance.

**Explanation:**
A common exam nuance: dynamic data masking affects what is **returned/displayed**, but it does not inherently prevent an unauthorized role from using the raw underlying value in a `WHERE` predicate to filter rows — meaning an unauthorized user could still potentially infer information by testing exact-match filters, unless combined with other controls like row access policies.
</details>

---

### Question 59
**Domain:** Domain 3 — Storage & Protection

What is the purpose of the `PURGE` copy option when running `COPY INTO <table> FROM @my_stage`?

- [ ] A. It clears the load metadata history so the same files can be reloaded without error.
- [ ] B. It validates the file checksum before allowing the load to proceed.
- [ ] C. It deletes the source files from the stage automatically after they have been successfully loaded.
- [ ] D. It removes duplicate rows from the target table after the load completes.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. It deletes the source files from the stage automatically after they have been successfully loaded.

**Explanation:**
Setting `PURGE = TRUE` in a `COPY INTO` statement instructs Snowflake to **automatically delete the source data files from the stage** once they have been successfully loaded into the target table, which helps keep stages clean and avoids accidental reprocessing of already-loaded files.
</details>

---

### Question 60
**Domain:** Domain 3 — Storage & Protection

A table has a tag applied via `ALTER TABLE customers SET TAG pii_classification = 'restricted'`. What is the primary governance benefit this provides on its own, without any additional policy attached?

- [ ] A. The tag automatically encrypts the table's data with a separate customer-managed key.
- [ ] B. The tag automatically enforces a masking policy on every column in the table.
- [ ] C. The tag prevents the table from being dropped by any role other than ACCOUNTADMIN.
- [ ] D. The tag enables centralized discovery and classification of objects (e.g., via ACCOUNT_USAGE views or Object Tagging features), supporting governance and auditing, even though it doesn't enforce access restrictions by itself.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. The tag enables centralized discovery and classification of objects (e.g., via ACCOUNT_USAGE views or Object Tagging features), supporting governance and auditing, even though it doesn't enforce access restrictions by itself.

**Explanation:**
**Object tagging** lets administrators attach metadata (like sensitivity classifications) to objects for discovery, reporting, and governance purposes — queryable through views like `TAG_REFERENCES`. On its own, a tag is purely **descriptive metadata**; it does not enforce masking or access restrictions unless paired with other mechanisms (e.g., tag-based masking policies).
</details>

---

### Question 61
**Domain:** Domain 3 — Storage & Protection

A file in an external stage uses Parquet format, and a user wants to load only specific top-level fields directly into typed columns rather than a single VARIANT column. What loading technique supports this?

- [ ] A. Using a COPY INTO statement with a SELECT list that references specific fields from the staged file using the file's columnar metadata, mapping them to individual typed target columns.
- [ ] B. Converting the Parquet file to CSV format manually before any load can specify individual columns.
- [ ] C. Creating a materialized view on the stage that automatically infers typed columns from Parquet schema metadata.
- [ ] D. This is not possible; Parquet files can only be loaded entirely into a single VARIANT column.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Using a COPY INTO statement with a SELECT list that references specific fields from the staged file using the file's columnar metadata, mapping them to individual typed target columns.

**Explanation:**
Snowflake supports a **COPY INTO `<table>` (col1, col2, ...) FROM (SELECT $1:field_name::TYPE, ... FROM @stage/file.parquet)** pattern, which allows selecting and casting specific fields out of a semi-structured (including Parquet) file directly into individual, typed target columns rather than loading the whole record into one VARIANT column.
</details>

---

### Question 62
**Domain:** Domain 4 — Data Movement

A pipeline uses `COPY INTO target_table FROM @stage` repeatedly on a schedule, pointing at the same stage location where new files are periodically added. Why does Snowflake avoid reloading files that were already successfully loaded in a previous run?

- [ ] A. Snowflake requires the FORCE = FALSE option to be manually specified every single time to prevent reloading.
- [ ] B. Snowflake renames files after loading them, so subsequent COPY INTO statements simply can't find the old file names anymore.
- [ ] C. Snowflake compares the row count of the target table before and after each run and rejects the load if it detects no new rows.
- [ ] D. Snowflake maintains load metadata (file name and checksum) for each table for a period of time and automatically skips files it recognizes as already loaded.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. Snowflake maintains load metadata (file name and checksum) for each table for a period of time and automatically skips files it recognizes as already loaded.

**Explanation:**
Snowflake tracks **load metadata** (file name, file size/checksum, load timestamp) per table for a period (historically 64 days). By default, `COPY INTO` automatically skips files whose metadata indicates they were already successfully loaded into that table, preventing duplicate loads even when the same file remains in the stage and the command is rerun.
</details>

---

### Question 63
**Domain:** Domain 4 — Data Movement

Which statement accurately describes the difference between Snowpipe (continuous/auto-ingest loading) and a scheduled Task running a COPY INTO statement on an interval?

- [ ] A. Snowpipe can only load CSV files, while Tasks can load any file format including JSON and Parquet.
- [ ] B. Snowpipe and Tasks are functionally identical; the only difference is which RBAC role can create them.
- [ ] C. Snowpipe uses serverless compute billed per-second based on actual usage, while a scheduled Task running COPY INTO uses a customer-managed virtual warehouse that the customer sizes and pays for by warehouse runtime.
- [ ] D. Tasks load data in near real-time as files arrive, while Snowpipe only runs on a fixed schedule defined by a cron expression.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Snowpipe uses serverless compute billed per-second based on actual usage, while a scheduled Task running COPY INTO uses a customer-managed virtual warehouse that the customer sizes and pays for by warehouse runtime.

**Explanation:**
**Snowpipe** uses Snowflake-managed, **serverless compute** that scales automatically and is billed based on actual per-second resource consumption for the load. A **Task** running `COPY INTO` on a schedule instead uses a **customer-managed virtual warehouse**, billed by warehouse uptime regardless of whether there's new data to load at each scheduled run, and runs on the defined interval rather than reacting instantly to new files.
</details>

---

### Question 64
**Domain:** Domain 4 — Data Movement

A Stream object is created on a table to track changes for downstream incremental processing. After a consumer queries the stream and consumes its contents within a transaction that also performs a DML operation against the stream's offset (e.g., via a task), what happens to the stream's offset?

- [ ] A. The offset never advances automatically; it must be manually reset using ALTER STREAM each time.
- [ ] B. The stream's offset advances to the current table version only when the DML transaction that consumed the stream successfully commits.
- [ ] C. Streams do not have an offset concept; they always return the full table history since creation.
- [ ] D. The offset advances immediately the moment the stream is queried with a plain SELECT, regardless of any transaction outcome.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The stream's offset advances to the current table version only when the DML transaction that consumed the stream successfully commits.

**Explanation:**
A stream's offset advances only when it is consumed within a **DML transaction that successfully commits** (e.g., an `INSERT ... SELECT FROM stream` or a task consuming it). A plain `SELECT` against the stream does **not** advance the offset — this distinction is important so that failed or rolled-back transactions don't lose track of unconsumed changes.
</details>

---

### Question 65
**Domain:** Domain 4 — Data Movement

A Task is defined with `AFTER` another task to form a dependency chain, and the root task in the chain has a schedule defined. What determines when the dependent (child) task actually executes?

- [ ] A. The child task executes simultaneously with the parent task, since AFTER only affects the order tasks appear in SHOW TASKS.
- [ ] B. The child task executes on its own independent schedule, ignoring the parent task's completion status entirely.
- [ ] C. The child task only executes if manually triggered via EXECUTE TASK after the parent completes.
- [ ] D. The child task executes automatically once its predecessor task in the DAG completes successfully, regardless of the predecessor's own schedule timing.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. The child task executes automatically once its predecessor task in the DAG completes successfully, regardless of the predecessor's own schedule timing.

**Explanation:**
When tasks are chained using `AFTER`, the dependent task runs automatically as part of the same **task graph (DAG)** once its predecessor completes **successfully**. The child task does not need (and typically doesn't have) its own independent schedule — its execution timing is driven entirely by the completion of the upstream task(s) it depends on.
</details>

---

### Question 66
**Domain:** Domain 4 — Data Movement

What is the main functional difference between an INSERT-only stream and a standard (default) stream created on a table?

- [ ] A. An insert-only stream has no offset and always returns the entire table, while a standard stream maintains an offset.
- [ ] B. An insert-only stream can only be created on views, while a standard stream can only be created on physical tables.
- [ ] C. An insert-only stream tracks only INSERT operations and ignores UPDATEs/DELETEs, making it suitable for append-only sources like external tables on cloud storage, while a standard stream tracks inserts, updates, and deletes.
- [ ] D. There is no functional difference; INSERT-only is purely a naming convention with identical underlying behavior.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. An insert-only stream tracks only INSERT operations and ignores UPDATEs/DELETEs, making it suitable for append-only sources like external tables on cloud storage, while a standard stream tracks inserts, updates, and deletes.

**Explanation:**
An **append-only (insert-only) stream** only records row **insertions**, ignoring updates and deletes — this is commonly used on **external tables** or append-only sources where update/delete tracking isn't meaningful or supported. A **standard stream** on a regular table captures the full set of row-level changes: inserts, updates (as paired delete+insert), and deletes.
</details>

---

### Question 67
**Domain:** Domain 4 — Data Movement

A user runs `COPY INTO @my_stage FROM my_table FILE_FORMAT = (TYPE = PARQUET)`. What kind of operation is this, and in which direction does data move?

- [ ] A. This is a data load operation, importing files already present in the stage into the table.
- [ ] B. This is invalid syntax because COPY INTO can never target a stage as the destination.
- [ ] C. This is a data unload operation, exporting query/table results from Snowflake out to files in the specified stage.
- [ ] D. This performs an in-place format conversion of the table's internal storage to Parquet.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. This is a data unload operation, exporting query/table results from Snowflake out to files in the specified stage.

**Explanation:**
When the **stage** appears as the target (right after `COPY INTO @stage`) and a table/query is the source, this is a **data unload** operation — Snowflake exports the table's (or query's) data out into files of the specified format (here, Parquet) written to the stage location, the reverse direction of a typical load.
</details>

---

### Question 68
**Domain:** Domain 4 — Data Movement

A task is scheduled using `SCHEDULE = 'USING CRON 0 9 * * MON-FRI America/New_York'`. What does this schedule represent?

- [ ] A. The task runs at midnight (0:00) on the 9th day of each month, only in months that start on a Monday.
- [ ] B. The task runs once at 9:00 AM Eastern time, on weekdays only (Monday through Friday).
- [ ] C. The task runs continuously throughout the 9 AM hour every day of the week.
- [ ] D. The task runs every 9 minutes, Monday through Friday, in Eastern time.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The task runs once at 9:00 AM Eastern time, on weekdays only (Monday through Friday).

**Explanation:**
The cron expression `0 9 * * MON-FRI` means: minute 0, hour 9, every day-of-month, every month, but only on **Monday through Friday** — i.e., the task fires once at **9:00 AM** on weekdays, evaluated in the specified `America/New_York` time zone.
</details>

---

### Question 69
**Domain:** Domain 4 — Data Movement

Which combination of objects is the canonical Snowflake pattern for building a continuous, incremental ELT pipeline that processes only newly changed rows as they arrive, without external orchestration tools?

- [ ] A. External tables combined with materialized views refreshed manually on a fixed schedule.
- [ ] B. Secure views combined with row access policies to filter only the newest rows on each query.
- [ ] C. Streams (to capture row-level changes) combined with Tasks (to schedule and execute the incremental processing logic).
- [ ] D. Resource monitors combined with network policies to gate when new data can be processed.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Streams (to capture row-level changes) combined with Tasks (to schedule and execute the incremental processing logic).

**Explanation:**
The standard native Snowflake pattern for incremental processing is **Streams + Tasks**: a stream captures the row-level change set (inserts/updates/deletes) since it was last consumed, and a scheduled or chained **task** executes the SQL logic that consumes those changes (e.g., merging into a downstream table), forming a lightweight, fully in-Snowflake ELT pipeline.
</details>

---

### Question 70
**Domain:** Domain 4 — Data Movement

A `MERGE INTO` statement is used to upsert rows from a staging table into a target table based on a matching key. If a row in the source matches an existing row in the target on the key, but no `WHEN MATCHED` clause is specified in the MERGE statement, what happens to that matched row?

- [ ] A. Nothing happens to that row in the target table — without a WHEN MATCHED clause, matched rows are simply left untouched.
- [ ] B. The matched row in the target table is automatically deleted, since MERGE defaults to a delete action for matches.
- [ ] C. Snowflake throws a syntax error, because WHEN MATCHED is a mandatory clause for every MERGE statement.
- [ ] D. The matched row is duplicated, inserting a second copy alongside the original.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Nothing happens to that row in the target table — without a WHEN MATCHED clause, matched rows are simply left untouched.

**Explanation:**
A `MERGE INTO` statement only takes the actions explicitly defined in its `WHEN MATCHED` / `WHEN NOT MATCHED` clauses. If `WHEN MATCHED` is omitted, rows that match the join condition are simply **left as-is** in the target table — `WHEN MATCHED` is optional, not mandatory, and you can have a MERGE with only a `WHEN NOT MATCHED` clause (insert-only) for example.
</details>

---

### Question 71
**Domain:** Domain 4 — Data Movement

An external table is defined over a set of Parquet files in cloud storage, with `AUTO_REFRESH = TRUE`. What does AUTO_REFRESH actually keep synchronized?

- [ ] A. It automatically reformats files from other formats into Parquet as they are added to the stage.
- [ ] B. It keeps the external table's metadata (the list of files and their partition information) in sync with the actual files present in the stage, using event notifications, without copying data into Snowflake.
- [ ] C. It refreshes only the query result cache associated with prior queries on the external table.
- [ ] D. It refreshes the external table's underlying file data, physically copying new file contents into Snowflake-managed storage every hour.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. It keeps the external table's metadata (the list of files and their partition information) in sync with the actual files present in the stage, using event notifications, without copying data into Snowflake.

**Explanation:**
`AUTO_REFRESH` on an external table uses cloud storage **event notifications** to keep the external table's **file-list metadata** current as files are added or removed in the underlying stage location. The actual file data is never copied into Snowflake-managed storage — external tables always read data directly from the customer's cloud storage at query time.
</details>

---

### Question 72
**Domain:** Domain 4 — Data Movement

A user wants a stored procedure to run automatically every day, but only if a particular condition (e.g., the existence of new rows in a staging table) is true, otherwise it should skip execution entirely without erroring. Which Task feature supports this?

- [ ] A. The optional WHEN clause on a task, which evaluates a boolean SQL condition before the task body executes, skipping the run if the condition is false.
- [ ] B. Task execution is always unconditional; conditional skipping must be implemented entirely inside the stored procedure body using RAISE.
- [ ] C. The task's WAREHOUSE parameter, which can be set to NULL to skip execution conditionally.
- [ ] D. The AFTER clause, which can reference a condition object instead of another task.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. The optional WHEN clause on a task, which evaluates a boolean SQL condition before the task body executes, skipping the run if the condition is false.

**Explanation:**
Tasks support an optional `WHEN <boolean_expression>` clause that Snowflake evaluates before running the task's SQL body. If the condition evaluates to false, the task's defined action is **skipped entirely** for that scheduled run (and this is reflected as a 'skipped' status in task history), without needing custom logic inside the procedure itself.
</details>

---

### Question 73
**Domain:** Domain 4 — Data Movement

A developer wants to load a large batch of files from an external stage but limit the load to process at most 1000 files in a single COPY INTO execution. Which option achieves this?

- [ ] A. There is no native COPY INTO option to cap the number of files processed per execution; this must be controlled by how files are organized/staged or via pattern matching scoped appropriately.
- [ ] B. MAX_FILES = 1000, a documented copy option specifically designed for this exact purpose.
- [ ] C. FILES = (file1.csv, file2.csv, ... up to 1000 explicit names)
- [ ] D. SIZE_LIMIT = 1000, which restricts the load to the first 1000 bytes of data.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. There is no native COPY INTO option to cap the number of files processed per execution; this must be controlled by how files are organized/staged or via pattern matching scoped appropriately.

**Explanation:**
Snowflake's `COPY INTO` does not provide a built-in option to cap the **number of files** processed in a single run (options like `FILES` require explicit naming, and `SIZE_LIMIT` caps bytes scanned, not file count). To control batch size by file count, pipelines typically organize files into prefixes/folders or use external orchestration rather than a COPY INTO parameter.
</details>

---

### Question 74
**Domain:** Domain 4 — Data Movement

A task graph (DAG) has a root task with a cron schedule, and three child tasks all defined with `AFTER root_task`. How do the three child tasks execute relative to each other once the root task completes?

- [ ] A. Only one of the three child tasks executes per run, chosen randomly by the scheduler.
- [ ] B. They execute strictly sequentially in the order they were created, one after another.
- [ ] C. They fail immediately because a task DAG only supports a single child per parent task.
- [ ] D. They can execute in parallel, since multiple tasks can depend on the same predecessor and are not required to run sequentially relative to each other.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. They can execute in parallel, since multiple tasks can depend on the same predecessor and are not required to run sequentially relative to each other.

**Explanation:**
Multiple tasks can share the same predecessor (`AFTER root_task`), forming a **fan-out** in the DAG. These sibling child tasks are not forced into a strict sequential order relative to each other — they can run **concurrently** once the shared predecessor completes successfully, as long as each has the resources (warehouse) available to do so.
</details>

---

### Question 75
**Domain:** Domain 4 — Data Movement

What does the `ON_ERROR = 'SKIP_FILE'` copy option do during a COPY INTO load when a particular file contains malformed rows?

- [ ] A. It immediately aborts the entire COPY INTO statement, skipping all remaining files in the batch.
- [ ] B. It skips only the malformed rows in that file and continues loading the valid rows from the same file.
- [ ] C. It skips loading that entire file (none of its rows are loaded) if any error is encountered, and continues to the next file in the batch.
- [ ] D. It silently converts malformed rows into NULLs across all columns and continues loading the file.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. It skips loading that entire file (none of its rows are loaded) if any error is encountered, and continues to the next file in the batch.

**Explanation:**
`ON_ERROR = 'SKIP_FILE'` causes Snowflake to abandon loading an **entire file** as soon as it encounters an error in that file (no partial rows from it are loaded), but processing **continues** with the next file in the batch rather than aborting the whole COPY INTO operation. This differs from `'CONTINUE'`, which skips only the bad rows and keeps loading valid rows from the same file.
</details>

---

### Question 76
**Domain:** Domain 4 — Data Movement

A Stream is created against a view rather than a base table. Which condition must be true for this to be supported?

- [ ] A. The view must be a materialized view exclusively, since standard views never support change tracking.
- [ ] B. Streams on views require the view to reference exactly one underlying table with no joins.
- [ ] C. The view must be a secure view, and streams on views require Enterprise Edition or higher along with change tracking enabled on the underlying objects.
- [ ] D. Streams cannot be created on views under any circumstances; only base tables are supported.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. The view must be a secure view, and streams on views require Enterprise Edition or higher along with change tracking enabled on the underlying objects.

**Explanation:**
Snowflake supports creating streams on views, but this requires the view to be defined with `CHANGE_TRACKING = TRUE` enabled (and underlying tables must also support change tracking), and this capability is available starting with **Enterprise Edition**. It allows tracking changes through more complex logical layers, not just directly on base tables.
</details>

---

### Question 77
**Domain:** Domain 5 — Account & Security

A custom role `analyst_role` is granted directly to a user, but is not part of the role hierarchy under SYSADMIN. What practical consequence does this have for account administration?

- [ ] A. Objects owned by analyst_role won't be visible/manageable by SYSADMIN through the standard role hierarchy, making it harder for administrators to centrally manage those objects unless analyst_role is granted to SYSADMIN.
- [ ] B. Users with analyst_role will be unable to log in until the role is placed under the hierarchy.
- [ ] C. The role will automatically inherit all privileges of SYSADMIN regardless of explicit grants.
- [ ] D. The role functions identically regardless of hierarchy placement, since RBAC hierarchy is purely cosmetic in Snowflake.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Objects owned by analyst_role won't be visible/manageable by SYSADMIN through the standard role hierarchy, making it harder for administrators to centrally manage those objects unless analyst_role is granted to SYSADMIN.

**Explanation:**
Snowflake's best practice is to build custom roles into the role hierarchy beneath **SYSADMIN**, so that SYSADMIN can manage (and see) objects created by those custom roles. If a custom role sits outside that hierarchy, **SYSADMIN won't automatically have visibility/control** over objects owned by it, which complicates centralized administration — a common reason exam questions test this hierarchy design principle.
</details>

---

### Question 78
**Domain:** Domain 5 — Account & Security

Which built-in system role has the privileges necessary to create, alter, and manage virtual warehouses, but does NOT automatically have privileges over user and role management?

- [ ] A. SECURITYADMIN
- [ ] B. SYSADMIN
- [ ] C. USERADMIN
- [ ] D. ACCOUNTADMIN

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. SYSADMIN

**Explanation:**
**SYSADMIN** is designed to manage the creation and configuration of **warehouses, databases, and other compute/data objects**, but it does not inherently manage **users and roles** — that responsibility belongs to **USERADMIN** (and **SECURITYADMIN**, which encompasses USERADMIN plus broader security object management). ACCOUNTADMIN sits above both and combines full administrative control.
</details>

---

### Question 79
**Domain:** Domain 5 — Account & Security

An account has a network policy applied at the account level that restricts allowed IP ranges. A specific service user also needs an exception to connect from an IP outside that range. What is the correct way to handle this without weakening the account-wide policy for everyone?

- [ ] A. Modify the account-level network policy's allowed IP list to include the new range for all users.
- [ ] B. Disable network policies account-wide temporarily whenever that service user needs to connect.
- [ ] C. Create a separate network policy and apply it directly to that specific user, which overrides the account-level policy for that user only.
- [ ] D. Grant the user the ACCOUNTADMIN role, which automatically bypasses all network policies.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Create a separate network policy and apply it directly to that specific user, which overrides the account-level policy for that user only.

**Explanation:**
Network policies can be applied at both the **account level** and the **individual user level**. A policy applied directly to a **user** takes precedence over the account-level policy for that specific user, allowing administrators to carve out exceptions for individual users/service accounts without loosening the broader account-wide restriction for everyone else.
</details>

---

### Question 80
**Domain:** Domain 5 — Account & Security

A role `r1` is granted to role `r2`, and `r2` is granted to user `alice`. Without explicitly granting `r1` directly to alice, can alice use the privileges granted to r1?

- [ ] A. Only if alice's default role is explicitly set to r1, regardless of the grant hierarchy.
- [ ] B. No, privileges never flow through nested role grants; alice would need r1 granted to her directly.
- [ ] C. Only temporarily, for 24 hours, after which the inherited grant automatically expires.
- [ ] D. Yes, because Snowflake's RBAC model allows roles to inherit privileges from other roles granted to them, so alice inherits r1's privileges through r2.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. Yes, because Snowflake's RBAC model allows roles to inherit privileges from other roles granted to them, so alice inherits r1's privileges through r2.

**Explanation:**
Snowflake RBAC supports **role hierarchies**: when role `r1` is granted to role `r2`, any user who has `r2` (directly or transitively) effectively inherits the privileges of `r1` as well. So `alice`, having `r2`, can exercise privileges granted to `r1` without needing `r1` granted to her individually.
</details>

---

### Question 81
**Domain:** Domain 5 — Account & Security

What is the function of `MULTI_FACTOR_AUTHENTICATION` enforcement combined with a network policy that also requires connections to originate from a specific corporate VPN IP range?

- [ ] A. Network policies automatically satisfy MFA requirements, making explicit MFA enforcement redundant once a network policy exists.
- [ ] B. These two controls are mutually exclusive; only one authentication/network control can be active on an account at a time.
- [ ] C. MFA enforcement overrides and disables any configured network policy for ACCOUNTADMIN-level users.
- [ ] D. They act as independent, layered controls — a user must both authenticate with MFA and connect from an allowed IP — providing defense-in-depth rather than either control alone.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. They act as independent, layered controls — a user must both authenticate with MFA and connect from an allowed IP — providing defense-in-depth rather than either control alone.

**Explanation:**
MFA and network policies are independent, complementary security controls. Enforcing both means a user must satisfy **both** conditions to connect — proving identity via a second factor **and** originating from an approved network/IP range — which is a standard **defense-in-depth** approach rather than either substituting for the other.
</details>

---

### Question 82
**Domain:** Domain 5 — Account & Security

A future grant is defined with `GRANT SELECT ON FUTURE TABLES IN SCHEMA my_schema TO ROLE analyst`. What does this accomplish?

- [ ] A. It grants SELECT on all tables that currently exist in my_schema at the time the statement runs.
- [ ] B. It schedules a one-time SELECT grant to be applied exactly 24 hours in the future on existing tables.
- [ ] C. It grants SELECT automatically on any new table created in my_schema after the future grant is defined, without requiring a separate GRANT statement for each new table.
- [ ] D. It is invalid syntax; future grants can only be applied at the database level, not the schema level.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. It grants SELECT automatically on any new table created in my_schema after the future grant is defined, without requiring a separate GRANT statement for each new table.

**Explanation:**
A **future grant** pre-authorizes privileges on objects that **don't exist yet**. `GRANT SELECT ON FUTURE TABLES IN SCHEMA my_schema TO ROLE analyst` ensures that any table subsequently created within that schema automatically has SELECT granted to the `analyst` role, eliminating the need to manually re-grant privileges every time a new table is added.
</details>

---

### Question 83
**Domain:** Domain 5 — Account & Security

Which authentication method allows a user to connect to Snowflake using a public/private key pair instead of a password, and is commonly used for service accounts and programmatic/API access?

- [ ] A. OAuth client credentials flow exclusively, since key pair authentication was deprecated in favor of OAuth.
- [ ] B. Federated SSO using SAML, which is the only supported method for non-interactive service accounts.
- [ ] C. Key pair authentication, where the user's public key is registered on their Snowflake user object and the private key is used client-side to sign the authentication request.
- [ ] D. SCIM provisioning, which authenticates users based on directory group membership alone.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Key pair authentication, where the user's public key is registered on their Snowflake user object and the private key is used client-side to sign the authentication request.

**Explanation:**
**Key pair authentication** lets a user register a **public key** on their Snowflake user object (`ALTER USER ... SET RSA_PUBLIC_KEY = ...`), while the corresponding **private key** is held securely by the client and used to sign authentication requests. This is widely used for service accounts, automation, and programmatic access (e.g., via drivers/connectors) where interactive password or SSO login isn't practical.
</details>

---

### Question 84
**Domain:** Domain 5 — Account & Security

An administrator wants to temporarily disable a specific user's ability to log in without dropping the user or revoking any of their role grants. Which approach achieves this?

- [ ] A. REVOKE ROLE PUBLIC FROM USER <username>, since PUBLIC is required for any login to succeed.
- [ ] B. ALTER USER <username> SET DEFAULT_ROLE = NULL, which automatically blocks all future logins.
- [ ] C. DROP USER followed by immediately recreating the user with the same name and grants.
- [ ] D. ALTER USER <username> SET DISABLED = TRUE, which prevents login while preserving the user object, its grants, and ownership of objects.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. ALTER USER <username> SET DISABLED = TRUE, which prevents login while preserving the user object, its grants, and ownership of objects.

**Explanation:**
Setting `DISABLED = TRUE` on a user object immediately prevents that user from logging in, while leaving the user record, its role grants, and any objects it owns completely intact. This is the clean way to temporarily suspend access (e.g., during an employee leave or investigation) without the destructive consequences of dropping and recreating the user.
</details>

---

### Question 85
**Domain:** Domain 5 — Account & Security

A column-level masking policy uses the CURRENT_ROLE() function within its conditional logic to decide whether to mask a value. What is a key limitation of relying solely on CURRENT_ROLE() in masking policy logic, compared to using IS_ROLE_IN_SESSION()?

- [ ] A. CURRENT_ROLE() evaluates at query compile time rather than at execution time, making it unsuitable for any dynamic policy.
- [ ] B. CURRENT_ROLE() always returns NULL when called from within a masking policy due to a security sandboxing restriction.
- [ ] C. CURRENT_ROLE() cannot be used inside any masking policy; only IS_ROLE_IN_SESSION() is permitted syntax.
- [ ] D. CURRENT_ROLE() only reflects the single currently active primary role in the session, so it can miss cases where the authorized role is available via a secondary role rather than being the active primary role.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. CURRENT_ROLE() only reflects the single currently active primary role in the session, so it can miss cases where the authorized role is available via a secondary role rather than being the active primary role.

**Explanation:**
`CURRENT_ROLE()` reflects only the session's single **active primary role**. If a user has the authorized role available as a **secondary role** (via `USE SECONDARY ROLES`) rather than as their active primary role, a check using only `CURRENT_ROLE()` would incorrectly mask the data. `IS_ROLE_IN_SESSION()` checks across both primary and secondary roles, making it the more robust choice for policies.
</details>

---

### Question 86
**Domain:** Domain 5 — Account & Security

Which privilege is required for a role to be able to view query history, login history, and other usage metadata for ALL users in the account via the ACCOUNT_USAGE schema, not just the querying user's own activity?

- [ ] A. The MONITOR privilege on every individual warehouse in the account, granted one at a time.
- [ ] B. The OWNERSHIP privilege on the ACCOUNT object itself.
- [ ] C. The IMPORTED PRIVILEGES privilege on the SNOWFLAKE database, typically granted to roles needing account-wide observability.
- [ ] D. Simply having any role granted to the user, since ACCOUNT_USAGE is globally readable by default.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. The IMPORTED PRIVILEGES privilege on the SNOWFLAKE database, typically granted to roles needing account-wide observability.

**Explanation:**
Access to the **SNOWFLAKE database**'s `ACCOUNT_USAGE` schema (which exposes account-wide metadata like query history and login history for all users) requires the **IMPORTED PRIVILEGES** privilege granted on the SNOWFLAKE database to a role. By default this is available to ACCOUNTADMIN, and can be extended to custom roles by granting IMPORTED PRIVILEGES to them.
</details>

---

### Question 87
**Domain:** Domain 5 — Account & Security

A SCIM integration is configured between Snowflake and an external identity provider (e.g., Okta or Azure AD). What does this integration primarily automate?

- [ ] A. Provisioning and deprovisioning of Snowflake users and roles based on group membership changes in the external identity provider, keeping them synchronized.
- [ ] B. Automatic warehouse auto-scaling based on the number of active directory users.
- [ ] C. Automatic rotation of customer-managed encryption keys stored in the identity provider's vault.
- [ ] D. Real-time replication of query results to the identity provider for centralized auditing dashboards.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Provisioning and deprovisioning of Snowflake users and roles based on group membership changes in the external identity provider, keeping them synchronized.

**Explanation:**
**SCIM (System for Cross-domain Identity Management)** integration automates the **lifecycle of users and group/role memberships** in Snowflake based on changes made in an external identity provider — e.g., automatically creating a Snowflake user when someone joins a group in Okta, or disabling them when they're removed, keeping identity state synchronized without manual administrative work.
</details>

---

### Question 88
**Domain:** Domain 5 — Account & Security

In Snowflake's RBAC model, what is the defining characteristic that differentiates a 'system-defined' role like SYSADMIN from a 'custom' role created by an administrator?

- [ ] A. System-defined roles come with a predefined, built-in set of privileges and intended purpose (e.g., SYSADMIN for object management), while custom roles are created by administrators with privileges assembled explicitly for a specific business need.
- [ ] B. System-defined roles can only be used by the ACCOUNTADMIN user, never assigned to other users.
- [ ] C. System-defined roles automatically expire after 90 days and must be recreated, while custom roles never expire.
- [ ] D. System-defined roles cannot be granted to other roles, while custom roles can be nested freely.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. System-defined roles come with a predefined, built-in set of privileges and intended purpose (e.g., SYSADMIN for object management), while custom roles are created by administrators with privileges assembled explicitly for a specific business need.

**Explanation:**
**System-defined roles** (ACCOUNTADMIN, SECURITYADMIN, SYSADMIN, USERADMIN, PUBLIC) ship with Snowflake and have a built-in, intended scope of responsibility and starting privilege set. **Custom roles** are created by administrators to model specific business functions (e.g., `finance_analyst`), with privileges explicitly assembled and typically nested under SYSADMIN in the hierarchy.
</details>

---

### Question 89
**Domain:** Domain 5 — Account & Security

A user's default role is set to `analyst_role`, but the user is also granted `auditor_role`. After logging in, the user runs `USE ROLE auditor_role;` to switch their active role mid-session. What happens to privileges associated with analyst_role during the remainder of that session?

- [ ] A. Switching roles mid-session is not permitted; a new session must be started to change the active role.
- [ ] B. Both roles remain simultaneously active as primary roles, with privileges from each combined for every statement.
- [ ] C. The user permanently loses analyst_role and must be re-granted it by an administrator to use it again.
- [ ] D. The user's active role switches to auditor_role for subsequent statements, but they retain the underlying grant of analyst_role and can switch back to it at any time with another USE ROLE statement.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. The user's active role switches to auditor_role for subsequent statements, but they retain the underlying grant of analyst_role and can switch back to it at any time with another USE ROLE statement.

**Explanation:**
`USE ROLE` changes which role is the session's **active primary role** for subsequent statements, but it does not revoke or remove the user's underlying grant of `analyst_role` — the user can freely switch back with another `USE ROLE analyst_role;` statement at any point, since both roles remain granted to the user throughout the session.
</details>

---

### Question 90
**Domain:** Domain 5 — Account & Security

What does Snowflake's `OBJECT_DEPENDENCIES` Account Usage view help an administrator identify?

- [ ] A. The set of network policies currently applied to each object in the account.
- [ ] B. The historical credit consumption attributable to each individual database object.
- [ ] C. The list of users who have ever queried a particular object, regardless of role.
- [ ] D. Dependency relationships between objects, such as which views or other objects rely on a given table, useful for assessing the blast radius before dropping or altering an object.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. Dependency relationships between objects, such as which views or other objects rely on a given table, useful for assessing the blast radius before dropping or altering an object.

**Explanation:**
The `OBJECT_DEPENDENCIES` view in `ACCOUNT_USAGE` surfaces relationships between objects — for example, which views, materialized views, or other downstream objects depend on a given table — helping administrators understand the potential impact (**blast radius**) of altering, renaming, or dropping an object before doing so.
</details>

---

### Question 91
**Domain:** Domain 5 — Account & Security

Which statement correctly describes the privilege required to create a new virtual warehouse in an account, assuming default out-of-the-box role configuration?

- [ ] A. CREATE WAREHOUSE requires both SYSADMIN and SECURITYADMIN roles to be active simultaneously via secondary roles.
- [ ] B. Only ACCOUNTADMIN can ever create warehouses; this privilege cannot be delegated to any other role.
- [ ] C. Any role with the PUBLIC role granted automatically has CREATE WAREHOUSE privilege, since PUBLIC is granted to every user.
- [ ] D. CREATE WAREHOUSE is an account-level privilege that SYSADMIN holds by default and can also be granted to custom roles as needed.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. CREATE WAREHOUSE is an account-level privilege that SYSADMIN holds by default and can also be granted to custom roles as needed.

**Explanation:**
`CREATE WAREHOUSE` is an account-level privilege. By default, **SYSADMIN** holds this privilege as part of its intended responsibility for managing compute and data objects, and administrators can also explicitly **grant CREATE WAREHOUSE** to other custom roles if delegated warehouse creation is desired, rather than this being exclusive to ACCOUNTADMIN.
</details>

---

### Question 92
**Domain:** Domain 6 — Performance

A query profile shows a large percentage of execution time attributed to a step labeled 'Bytes spilled to remote storage'. What is the most direct corrective action to reduce this specific bottleneck?

- [ ] A. Switch the warehouse to a multi-cluster configuration to distribute the spilled data across more clusters.
- [ ] B. Enable the search optimization service on the table to speed up the spill mechanism itself.
- [ ] C. Increase the size of the virtual warehouse running the query, giving it more memory and local SSD capacity to avoid spilling to slower remote storage.
- [ ] D. Add a clustering key on the table being queried to reduce micro-partition scanning.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Increase the size of the virtual warehouse running the query, giving it more memory and local SSD capacity to avoid spilling to slower remote storage.

**Explanation:**
Spilling to **remote storage** indicates the query's intermediate data exceeded both available memory and local SSD capacity. The direct fix is to provide more memory/local disk by **increasing the warehouse size**, which is generally the most effective remedy for memory-bound queries, as opposed to clustering keys (which help pruning, not memory) or multi-cluster (which helps concurrency, not single-query resources).
</details>

---

### Question 93
**Domain:** Domain 6 — Performance

Using the Query Profile, an analyst notices that a particular join step shows a very high 'percentage of time' along with a large 'bytes scanned' figure relative to a small number of rows actually produced. What does this combination most strongly suggest?

- [ ] A. The issue is unrelated to the table's physical layout and can only be solved by rewriting the join as a subquery.
- [ ] B. The join may benefit from better pruning — for example, via a clustering key on the join/filter columns — since a lot of data is being read and processed but relatively little survives the join.
- [ ] C. The join is efficient and well-optimized, since high scan volume typically indicates good parallelism.
- [ ] D. The warehouse is undersized and must be resized up immediately regardless of any other tuning options.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The join may benefit from better pruning — for example, via a clustering key on the join/filter columns — since a lot of data is being read and processed but relatively little survives the join.

**Explanation:**
When a step scans/reads a large volume of data but produces comparatively few output rows, it often signals that the engine is reading more micro-partitions than necessary because of poor **pruning** on the filter/join columns. Improving **clustering** on those columns (or adjusting the query/table design) can reduce the bytes scanned by allowing more partitions to be skipped.
</details>

---

### Question 94
**Domain:** Domain 6 — Performance

A materialized view is created on top of a base table to pre-aggregate daily sales totals. What ongoing cost does Snowflake incur to keep that materialized view up to date as the base table changes?

- [ ] A. No additional cost; materialized views are refreshed for free as part of standard storage billing.
- [ ] B. The materialized view only refreshes when manually triggered via ALTER MATERIALIZED VIEW REFRESH, incurring warehouse credits at that time only.
- [ ] C. Materialized views are refreshed using the same warehouse that loaded the base table, with no separate billing line.
- [ ] D. Snowflake automatically uses background serverless compute to incrementally maintain the materialized view whenever the base table changes, which is billed separately from regular warehouse usage.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. Snowflake automatically uses background serverless compute to incrementally maintain the materialized view whenever the base table changes, which is billed separately from regular warehouse usage.

**Explanation:**
Snowflake automatically and continuously maintains materialized views in the background using **serverless compute**, incrementally updating them as the underlying base table changes. This background maintenance work is billed separately (as **materialized view maintenance** credits), distinct from regular user-managed virtual warehouse billing.
</details>

---

### Question 95
**Domain:** Domain 6 — Performance

What is the primary performance benefit of defining a clustering key on a very large table that is frequently filtered on a specific column with high cardinality?

- [ ] A. It guarantees the table will never need a virtual warehouse larger than Medium to query efficiently.
- [ ] B. It eliminates the need for the result cache, since clustering precomputes all possible filter results.
- [ ] C. It automatically creates a secondary B-tree index that bypasses the need for any micro-partition scanning.
- [ ] D. It encourages Snowflake to physically co-locate rows with similar values for that column within the same micro-partitions, improving partition pruning for filters on that column.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. It encourages Snowflake to physically co-locate rows with similar values for that column within the same micro-partitions, improving partition pruning for filters on that column.

**Explanation:**
A **clustering key** influences how Snowflake organizes data into micro-partitions so that rows with similar values of the clustering key tend to be co-located. This improves the effectiveness of **partition pruning**: when a query filters on that column, Snowflake can skip a larger proportion of irrelevant micro-partitions, reducing the amount of data scanned and improving performance.
</details>

---

### Question 96
**Domain:** Domain 6 — Performance

A long-running query's profile shows significant time spent in a step labeled 'Partitions scanned: 9,800 / Partitions total: 10,000' despite a highly selective WHERE clause filtering for a narrow date range. What does this most likely indicate about the table?

- [ ] A. Partition pruning is fundamentally incompatible with range filters on TIMESTAMP columns.
- [ ] B. The table's clustering with respect to the filtered date column is poor, so the values for that narrow range are scattered across nearly all micro-partitions, defeating pruning.
- [ ] C. The query optimizer has a bug that ignores WHERE clauses on date-typed columns specifically.
- [ ] D. The warehouse is too small to apply pruning logic, and resizing up will fix the scan ratio.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The table's clustering with respect to the filtered date column is poor, so the values for that narrow range are scattered across nearly all micro-partitions, defeating pruning.

**Explanation:**
When nearly all partitions are scanned despite a highly selective filter, it usually means the data is **poorly clustered** with respect to the filtered column — rows matching the narrow date range are spread thinly across almost every micro-partition rather than being concentrated in a few, so pruning can't eliminate most partitions. Improving clustering on that column (or reloading data in a more sorted order) is the typical fix.
</details>

---

### Question 97
**Domain:** Domain 6 — Performance

Which of the following actions is most likely to IMPROVE the effectiveness of Snowflake's result cache for a recurring dashboard query?

- [ ] A. Increasing the warehouse size so the cache is computed faster.
- [ ] B. Switching the warehouse to multi-cluster mode so multiple result caches can be created in parallel.
- [ ] C. Keeping the query text and underlying data identical between runs, and avoiding constructs that make the query non-deterministic or that reference data that changes between executions.
- [ ] D. Using non-deterministic functions like CURRENT_TIMESTAMP() or RANDOM() directly within the query's filter logic.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Keeping the query text and underlying data identical between runs, and avoiding constructs that make the query non-deterministic or that reference data that changes between executions.

**Explanation:**
The result cache only serves a cached result when the **query text is identical** and the underlying data **hasn't changed** since the prior execution (among other conditions). Using non-deterministic functions (like `RANDOM()` or `CURRENT_TIMESTAMP()`) or referencing constantly-changing data defeats caching, since Snowflake can no longer guarantee the cached result is still valid/equivalent.
</details>

---

### Question 98
**Domain:** Domain 6 — Performance

A team observes that automatic clustering credit consumption has grown substantially for a table that receives very high-frequency, continuous micro-batch updates. What is the most appropriate tuning response?

- [ ] A. Reconsider whether a clustering key is appropriate for this table at all, since extremely high-frequency updates can cause continuous, costly re-clustering churn, and a coarser key or no clustering key may be more cost-effective.
- [ ] B. Disable Time Travel on the table, since active Time Travel retention is what drives clustering cost.
- [ ] C. Switch the table from permanent to transient, since transient tables are exempt from automatic clustering charges.
- [ ] D. Increase the clustering key's cardinality further, since higher cardinality always reduces reclustering cost.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Reconsider whether a clustering key is appropriate for this table at all, since extremely high-frequency updates can cause continuous, costly re-clustering churn, and a coarser key or no clustering key may be more cost-effective.

**Explanation:**
Automatic clustering is itself a background credit-consuming service, and on tables undergoing **very frequent, continuous DML**, maintaining tight clustering can become expensive because the engine keeps reorganizing micro-partitions as new data constantly disrupts the sort order. In such cases it's often more cost-effective to choose a **coarser clustering key**, cluster less frequently-changing columns, or reconsider whether clustering is justified at all for that table's access patterns.
</details>

---

### Question 99
**Domain:** Domain 6 — Performance

Two semantically equivalent queries return the same result set, but Query A explicitly lists only the 5 needed columns while Query B uses SELECT * against a 200-column wide table. Assuming both queries have an otherwise identical filter and equally good pruning, why does Query A typically perform better?

- [ ] A. There is no actual performance difference; column projection has no effect in Snowflake's storage engine.
- [ ] B. Query A reads less data overall, since Snowflake's columnar storage allows it to skip retrieving the unneeded 195 columns, reducing bytes scanned.
- [ ] C. Query A benefits from the result cache while Query B never can, regardless of any other factor.
- [ ] D. Query B is always rejected by the optimizer due to the wide column count exceeding a hard limit.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Query A reads less data overall, since Snowflake's columnar storage allows it to skip retrieving the unneeded 195 columns, reducing bytes scanned.

**Explanation:**
Because Snowflake stores data in a **columnar** format within micro-partitions, a query that explicitly selects only the needed columns allows Snowflake to **skip reading the data for unrequested columns** entirely. `SELECT *` against a very wide table forces all columns to be read, increasing bytes scanned and I/O even when row-level filtering/pruning is identical between the two queries.
</details>

---

### Question 100
**Domain:** Domain 6 — Performance

A table frequently joined on a UUID-type column shows poor join performance even after adding a clustering key on that column. Investigation reveals the UUID values are randomly generated and have no natural ordering relationship to insertion time or any other query pattern. Why might clustering provide limited benefit in this specific case?

- [ ] A. UUID columns are automatically excluded from micro-partition metadata, so pruning is structurally impossible on them.
- [ ] B. Because UUIDs are effectively random with no correlation to how data is naturally loaded or queried together, clustering on them provides little pruning benefit for typical range-style filters, and maintaining that clustering can be costly relative to the gain.
- [ ] C. Clustering always improves join performance regardless of the data distribution of the clustering key, so the observation must be due to an unrelated warehouse sizing issue.
- [ ] D. Clustering keys can only be defined on integer columns, so the UUID clustering key was silently ignored.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Because UUIDs are effectively random with no correlation to how data is naturally loaded or queried together, clustering on them provides little pruning benefit for typical range-style filters, and maintaining that clustering can be costly relative to the gain.

**Explanation:**
Clustering is most beneficial when there's a meaningful correlation between the clustering key's values and how data is loaded or filtered (e.g., a date column that naturally increases over time). **Random UUIDs** have no such correlation — values are scattered arbitrarily — so clustering on them yields limited pruning benefit for typical equality/range predicates while still incurring the ongoing cost of maintaining that clustering, making it often not worthwhile.
</details>

---

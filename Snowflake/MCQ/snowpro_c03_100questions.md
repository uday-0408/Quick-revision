# SnowPro Core Practice Questions

### Question 1
**Domain:** Domain 1 — Architecture

Which layer of Snowflake's architecture is responsible for query optimization, metadata management, and authentication, and runs on shared cloud infrastructure managed by Snowflake?

- [ ] A. Compute layer (virtual warehouses)
- [ ] B. Database Storage layer
- [ ] C. Cloud Services layer
- [ ] D. Metadata cache layer

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Cloud Services layer

**Explanation:**
The **Cloud Services layer** handles query parsing and optimization, metadata management, authentication, access control, and infrastructure management. It runs on shared compute managed by Snowflake itself, separate from customer-provisioned virtual warehouses.
</details>

---

### Question 2
**Domain:** Domain 1 — Architecture

A company needs to run Snowflake workloads on AWS in us-east-1 AND on Azure in eastus simultaneously and wants a single governance layer. Which Snowflake construct enables this?

- [ ] A. Multi-cluster warehouse across cloud regions
- [ ] B. Snowflake Organization with accounts in each cloud/region
- [ ] C. Cross-cloud database replication group
- [ ] D. Snowflake Data Sharing across provider accounts

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowflake Organization with accounts in each cloud/region

**Explanation:**
A **Snowflake Organization** is a top-level container that links multiple Snowflake accounts across clouds and regions under unified billing and governance. Individual accounts are created per cloud/region, and the Organization provides cross-account visibility, usage monitoring, and replication management.
</details>

---

### Question 3
**Domain:** Domain 1 — Architecture

What distinguishes a Transient table from a Temporary table in Snowflake?

- [ ] A. Transient tables persist beyond the session; Temporary tables are dropped when the session ends.
- [ ] B. Transient tables support Time Travel up to 90 days; Temporary tables support 0 days.
- [ ] C. Transient tables are visible to all roles; Temporary tables are visible only to their creating session.
- [ ] D. There is no difference; they are aliases for the same object type.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Transient tables persist beyond the session; Temporary tables are dropped when the session ends.

**Explanation:**
**Temporary tables** exist only within the creating session and are automatically dropped when the session ends. **Transient tables** persist across sessions like permanent tables but have no Fail-safe period and limited Time Travel (0 or 1 day). Both lack Fail-safe, which differentiates them from permanent tables.
</details>

---

### Question 4
**Domain:** Domain 1 — Architecture

A Snowflake account on Business Critical edition is required by a compliance team. Which feature is EXCLUSIVELY available on Business Critical and NOT on Enterprise edition?

- [ ] A. Multi-cluster warehouses
- [ ] B. Column-level security (Dynamic Data Masking)
- [ ] C. Tri-Secret Secure (customer-managed encryption keys via Snowflake + cloud KMS)
- [ ] D. Materialized views

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Tri-Secret Secure (customer-managed encryption keys via Snowflake + cloud KMS)

**Explanation:**
**Tri-Secret Secure** is a Business Critical-exclusive feature that combines Snowflake's encryption key with a customer-managed key in their cloud provider KMS. Both keys are required to decrypt data, ensuring Snowflake alone cannot access customer data. Multi-cluster warehouses and materialized views are available from Enterprise. Column-level security is available from Enterprise as well.
</details>

---

### Question 5
**Domain:** Domain 1 — Architecture

A developer queries INFORMATION_SCHEMA.TABLES and notices results appear instantly without warehouse consumption. Which Snowflake mechanism explains this?

- [ ] A. Result cache returning a cached prior query
- [ ] B. Metadata cache in the Cloud Services layer serving schema and statistics without compute
- [ ] C. Warehouse cache containing previously scanned table metadata
- [ ] D. Automatic query rewrite to a smaller micro-partition sample

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Metadata cache in the Cloud Services layer serving schema and statistics without compute

**Explanation:**
The **metadata cache** in the Cloud Services layer stores object definitions, statistics, and schema information. Queries against INFORMATION_SCHEMA that only read metadata (row counts, column names, table sizes) are served entirely from this cache — no virtual warehouse is needed or billed.
</details>

---

### Question 6
**Domain:** Domain 1 — Architecture

Which statement about Snowflake micro-partitions is TRUE?

- [ ] A. Each micro-partition is approximately 500MB–1GB of uncompressed data, stored in columnar format.
- [ ] B. Micro-partitions are 50–500MB of uncompressed data, stored as columnar compressed files with metadata including min/max values per column.
- [ ] C. Micro-partitions are row-oriented and rebalanced on every DML operation.
- [ ] D. Micro-partitions are user-defined; size is controlled with the MICRO_PARTITION_SIZE parameter.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Micro-partitions are 50–500MB of uncompressed data, stored as columnar compressed files with metadata including min/max values per column.

**Explanation:**
Snowflake **micro-partitions** are contiguous units of storage, 50–500MB of uncompressed data each, stored in columnar compressed format. Snowflake automatically maintains metadata per micro-partition including min/max values, distinct counts, and null counts per column. This metadata enables efficient partition pruning during queries.
</details>

---

### Question 7
**Domain:** Domain 1 — Architecture

A Secure View is created over a base table containing PII. Which statement correctly describes the security behavior of a Secure View?

- [ ] A. A Secure View encrypts query results before returning them to the user.
- [ ] B. A Secure View prevents the query optimizer from using the view's definition in EXPLAIN plans visible to unauthorized users, hiding the underlying SQL logic.
- [ ] C. A Secure View applies row-level security automatically based on the current user's role.
- [ ] D. A Secure View disables all caching for queries against it.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. A Secure View prevents the query optimizer from using the view's definition in EXPLAIN plans visible to unauthorized users, hiding the underlying SQL logic.

**Explanation:**
A **Secure View** hides the view's SQL definition from users who do not own it — the definition is not exposed via SHOW VIEWS, INFORMATION_SCHEMA, or query profiles/EXPLAIN for non-owners. It also prevents certain query optimizer shortcuts that could leak information about the underlying data structure. It does NOT automatically apply row-level security or disable caching entirely.
</details>

---

### Question 8
**Domain:** Domain 1 — Architecture

A developer needs to execute a Scala-based custom function inside a SQL pipeline that processes large DataFrame operations on Snowflake data. Which feature is the CORRECT choice?

- [ ] A. External Function via API Gateway
- [ ] B. Snowpark (Scala) running on a Snowpark-Optimized warehouse
- [ ] C. Snowflake Cortex Complete with a Scala prompt
- [ ] D. A JavaScript UDF calling an external Scala microservice

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowpark (Scala) running on a Snowpark-Optimized warehouse

**Explanation:**
**Snowpark** enables developers to write data pipelines in Python, Java, or Scala using DataFrames that execute directly within Snowflake — no data movement needed. For large DataFrame operations, a **Snowpark-Optimized warehouse** (16× more memory per node) is the recommended compute. External Functions add network latency and Cortex Complete is a generative AI feature.
</details>

---

### Question 9
**Domain:** Domain 1 — Architecture

Which Snowflake Cortex feature allows analysts to query structured Snowflake data using natural language, generating SQL automatically?

- [ ] A. Cortex Search
- [ ] B. Cortex Complete
- [ ] C. Cortex Analyst
- [ ] D. Cortex Guard

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Cortex Analyst

**Explanation:**
**Cortex Analyst** is the Snowflake feature that allows business users to ask questions in natural language and receive SQL-generated answers against structured Snowflake tables. **Cortex Search** powers semantic/vector search over unstructured text. **Cortex Complete** is a general LLM inference function.
</details>

---

### Question 10
**Domain:** Domain 1 — Architecture

A Standard (Gen 2) X-Large warehouse is auto-suspended after 10 minutes of inactivity. When the next query arrives, which statement about resume behavior is TRUE?

- [ ] A. The warehouse resumes in under 1 second because Snowflake pre-warms nodes.
- [ ] B. The warehouse resumes in a few seconds; the first query may experience slightly longer startup time compared to subsequent queries on a warm warehouse.
- [ ] C. The warehouse must be manually resumed; AUTO_RESUME only works for scheduled tasks.
- [ ] D. The warehouse resumes instantly because local disk cache is preserved across suspensions.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The warehouse resumes in a few seconds; the first query may experience slightly longer startup time compared to subsequent queries on a warm warehouse.

**Explanation:**
Snowflake warehouses typically **resume in a few seconds** (often 2–5s). The first query after resume may be slightly slower as the warehouse initializes. AUTO_RESUME = TRUE (the default) means the warehouse resumes automatically when a query arrives — no manual intervention needed. The local disk (warehouse) cache is **lost** on suspension, so cached data must be re-read from remote storage.
</details>

---

### Question 11
**Domain:** Domain 1 — Architecture

What is the purpose of a Sequence object in Snowflake?

- [ ] A. To define the load order of files in a Snowpipe ingestion pipeline
- [ ] B. To generate unique, monotonically increasing numeric values for use as surrogate keys
- [ ] C. To control the order in which micro-partitions are scanned during a query
- [ ] D. To define the priority sequence of warehouse scaling events

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. To generate unique, monotonically increasing numeric values for use as surrogate keys

**Explanation:**
A **Sequence** in Snowflake is a schema-level object that generates unique integer values (similar to AUTO_INCREMENT). It is commonly used to produce surrogate keys. Note: sequences are not guaranteed to produce gap-free consecutive values — gaps can occur due to caching and concurrency, which is acceptable for surrogate key use cases.
</details>

---

### Question 12
**Domain:** Domain 1 — Architecture

An Apache Iceberg table is created in Snowflake using an external catalog (AWS Glue). Which statement about this configuration is TRUE?

- [ ] A. Snowflake manages the Iceberg metadata and catalog; AWS Glue is only used for storage.
- [ ] B. Snowflake can read and write the Iceberg table; all metadata is stored in AWS Glue.
- [ ] C. With an external catalog, Snowflake can read Iceberg tables but write operations are not supported through Snowflake.
- [ ] D. Iceberg tables with external catalogs support Time Travel up to 90 days like permanent tables.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. With an external catalog, Snowflake can read Iceberg tables but write operations are not supported through Snowflake.

**Explanation:**
When using an **external catalog** (e.g., AWS Glue) for Iceberg tables in Snowflake, Snowflake operates in a **read-only** mode — it can query the data but cannot write to it. Writes must go through the catalog's native tooling. Snowflake-managed Iceberg tables (where Snowflake owns the catalog) support both read and write. Time Travel is not supported for Iceberg tables in the same way as native Snowflake tables.
</details>

---

### Question 13
**Domain:** Domain 1 — Architecture

A Dynamic Table is defined with a target lag of '1 hour'. What does this mean in practice?

- [ ] A. The table refreshes exactly every hour on the clock.
- [ ] B. Snowflake attempts to keep the Dynamic Table's data no more than 1 hour behind its base tables, refreshing as needed.
- [ ] C. The table caches query results for 1 hour before invalidating them.
- [ ] D. Users querying the Dynamic Table will wait up to 1 hour for a result before timeout.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowflake attempts to keep the Dynamic Table's data no more than 1 hour behind its base tables, refreshing as needed.

**Explanation:**
**Target lag** on a Dynamic Table defines the maximum acceptable staleness — Snowflake's scheduler aims to refresh the table often enough so the data is never more than the specified lag behind the source tables. It is a freshness SLA, not a fixed schedule. Actual refresh frequency depends on upstream changes and lag setting.
</details>

---

### Question 14
**Domain:** Domain 1 — Architecture

Which of the following is NOT a valid Snowflake warehouse scaling policy for multi-cluster warehouses?

- [ ] A. Economy
- [ ] B. Standard
- [ ] C. Performance
- [ ] D. Aggressive

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. Aggressive

**Explanation:**
Snowflake multi-cluster warehouses support two scaling policies: **Standard** (starts additional clusters as soon as there's a queued query) and **Economy** (waits until there's enough queued load to fully utilize an additional cluster before starting it). **Performance** and **Aggressive** are not valid Snowflake scaling policy options.
</details>

---

### Question 15
**Domain:** Domain 1 — Architecture

Snowflake Notebooks run on which compute type by default?

- [ ] A. Standard virtual warehouses only
- [ ] B. Snowpark-Optimized warehouses only
- [ ] C. Serverless compute (no warehouse needed)
- [ ] D. Container-based Snowpark Container Services compute

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Standard virtual warehouses only

**Explanation:**
**Snowflake Notebooks** use a **Standard virtual warehouse** as their default compute for SQL and Snowpark Python cells. They do not run on serverless compute by default, though Snowflake has been expanding compute options. Notebooks leverage the assigned warehouse for code execution.
</details>

---

### Question 16
**Domain:** Domain 2 — Governance

In Snowflake's RBAC model, which system-defined role has the ability to create and manage other roles, but does NOT have access to data by default?

- [ ] A. SYSADMIN
- [ ] B. SECURITYADMIN
- [ ] C. ACCOUNTADMIN
- [ ] D. USERADMIN

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. SECURITYADMIN

**Explanation:**
**SECURITYADMIN** can create and manage roles and grants (MANAGE GRANTS privilege), and manages users. It does NOT have access to data in databases by default — that access must be explicitly granted. ACCOUNTADMIN is the top-level role with all privileges. SYSADMIN manages warehouses and databases. USERADMIN manages users and roles but has narrower scope than SECURITYADMIN.
</details>

---

### Question 17
**Domain:** Domain 2 — Governance

A user needs to query tables in schema PROD.SALES but should not see the underlying table structures or grant further access. What is the MINIMUM set of privileges required?

- [ ] A. USAGE on database, USAGE on schema, SELECT on tables
- [ ] B. OWNERSHIP on database, USAGE on schema, SELECT on tables
- [ ] C. USAGE on database, CREATE TABLE on schema, SELECT on tables
- [ ] D. USAGE on database, MONITOR on schema, SELECT on tables

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. USAGE on database, USAGE on schema, SELECT on tables

**Explanation:**
To query tables, a user needs: **USAGE on the database** (to navigate to it), **USAGE on the schema** (to navigate to it), and **SELECT on the tables**. OWNERSHIP grants full control including the ability to re-grant — it's not needed for read access. MONITOR lets users see resource usage, not query data.
</details>

---

### Question 18
**Domain:** Domain 2 — Governance

A column masking policy uses CURRENT_ROLE() to conditionally mask values. A user has two active roles: ANALYST (primary) and AUDITOR (secondary). The policy unmasks for ANALYST. Which statement is TRUE?

- [ ] A. The user sees unmasked values because CURRENT_ROLE() returns the highest-privilege active role.
- [ ] B. The user sees unmasked values because CURRENT_ROLE() returns ANALYST, their primary role.
- [ ] C. The user sees masked values because CURRENT_ROLE() only checks the session's USE ROLE — secondary roles are not evaluated.
- [ ] D. The masking policy automatically checks all active roles and unmasks if any role qualifies.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The user sees unmasked values because CURRENT_ROLE() returns ANALYST, their primary role.

**Explanation:**
**CURRENT_ROLE()** returns the user's **primary (active) role** — the one set with USE ROLE. Secondary roles provide additional privileges for access control but are NOT considered by masking policy CASE expressions that call CURRENT_ROLE(). To check all active roles in a masking policy, you would use **IS_ROLE_IN_SESSION()**.
</details>

---

### Question 19
**Domain:** Domain 2 — Governance

What is the difference between Row Access Policies and Dynamic Data Masking in Snowflake?

- [ ] A. Row Access Policies filter entire rows from query results; Dynamic Data Masking obscures values in specific columns while the row remains visible.
- [ ] B. Row Access Policies apply to semi-structured data only; Dynamic Data Masking applies to structured tables.
- [ ] C. Row Access Policies are applied at query time; Dynamic Data Masking is applied at storage time during the COPY INTO load.
- [ ] D. Dynamic Data Masking can filter rows; Row Access Policies can only mask column values.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Row Access Policies filter entire rows from query results; Dynamic Data Masking obscures values in specific columns while the row remains visible.

**Explanation:**
**Row Access Policies** control which rows a user can see — unauthorized rows are completely excluded from results. **Dynamic Data Masking** controls what value appears in a column — the row is still returned but the sensitive column shows a masked value (e.g., hash, NULL, or partial value). Both are applied at query execution time, not at storage time.
</details>

---

### Question 20
**Domain:** Domain 2 — Governance

A network policy is applied at both the account level and to a specific user. The account policy allows IP range 10.0.0.0/8, and the user policy allows only 192.168.1.0/24. Which policy takes precedence?

- [ ] A. The account-level policy always takes precedence over user-level policies.
- [ ] B. The user-level policy takes precedence; the more specific policy wins.
- [ ] C. Both policies are evaluated with an AND condition; the user must be in both ranges.
- [ ] D. Network policies cannot be applied at both levels simultaneously — it causes an error.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The user-level policy takes precedence; the more specific policy wins.

**Explanation:**
When a network policy is applied to both the account and a specific user, the **user-level policy takes precedence** for that user. So a user with IP 192.168.1.5 would be allowed even if 10.0.0.0/8 is the account policy, because the user's specific policy (192.168.1.0/24) overrides the account policy for that user.
</details>

---

### Question 21
**Domain:** Domain 2 — Governance

An organization wants to implement column-level encryption where data is encrypted before being stored and queries decrypt on the fly using a customer-controlled key. Which Snowflake feature enables this?

- [ ] A. Dynamic Data Masking with a custom UDF
- [ ] B. Tri-Secret Secure with a customer-managed key in AWS KMS
- [ ] C. Snowflake-managed encryption (AES-256) which is always on by default
- [ ] D. External tokenization with an external tokenization provider via masking policy

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. External tokenization with an external tokenization provider via masking policy

**Explanation:**
**External tokenization** (available on Business Critical) uses a masking policy that calls an external tokenization service (via External Function) to replace sensitive values with tokens before storage. Queries call the detokenization service transparently. This gives full customer control over encryption keys outside Snowflake. Tri-Secret Secure encrypts the entire storage layer, not individual columns.
</details>

---

### Question 22
**Domain:** Domain 2 — Governance

Which view in SNOWFLAKE.ACCOUNT_USAGE provides a complete audit trail of all queries executed, including queries run by other users, with up to 365 days of history?

- [ ] A. INFORMATION_SCHEMA.QUERY_HISTORY
- [ ] B. ACCOUNT_USAGE.QUERY_HISTORY
- [ ] C. ACCOUNT_USAGE.LOGIN_HISTORY
- [ ] D. ACCOUNT_USAGE.ACCESS_HISTORY

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. ACCOUNT_USAGE.QUERY_HISTORY

**Explanation:**
**ACCOUNT_USAGE.QUERY_HISTORY** retains query history for up to **365 days** and is accessible to ACCOUNTADMIN (or roles with appropriate grants). It covers all users in the account. INFORMATION_SCHEMA.QUERY_HISTORY is limited to the last 7 days and the current user/session. ACCESS_HISTORY tracks which columns were accessed by which queries — a separate audit feature.
</details>

---

### Question 23
**Domain:** Domain 2 — Governance

A resource monitor is set with a CREDIT_QUOTA of 100 and a SUSPEND action at 90%. What happens when 90 credits are consumed?

- [ ] A. All running queries on monitored warehouses are immediately killed and the warehouses are suspended.
- [ ] B. Monitored warehouses are suspended after currently running queries complete; new queries are rejected.
- [ ] C. An alert notification is sent but no action is taken until 100% is reached.
- [ ] D. The warehouse is suspended and cannot be manually resumed until the quota resets.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Monitored warehouses are suspended after currently running queries complete; new queries are rejected.

**Explanation:**
The **SUSPEND** action on a resource monitor means: when the threshold is hit, monitored warehouses will not start new queries — they wait for currently running queries to finish, then suspend. Running queries are NOT killed mid-execution (that is the SUSPEND_IMMEDIATE action). The warehouse CAN be manually resumed by an ACCOUNTADMIN.
</details>

---

### Question 24
**Domain:** Domain 2 — Governance

Which authentication method in Snowflake uses a private/public key pair and is recommended for service accounts and programmatic access?

- [ ] A. Multi-Factor Authentication (MFA)
- [ ] B. OAuth 2.0 with authorization code flow
- [ ] C. Key-pair authentication
- [ ] D. SAML 2.0 federated authentication

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Key-pair authentication

**Explanation:**
**Key-pair authentication** uses a 2048-bit RSA private key on the client and a corresponding public key registered in Snowflake. It is the recommended method for service accounts, CI/CD pipelines, and programmatic access because it does not require interactive MFA and avoids password management. The client signs a JWT with the private key, and Snowflake verifies it with the public key.
</details>

---

### Question 25
**Domain:** Domain 2 — Governance

Data lineage in Snowflake is tracked through which feature, showing which columns were read and written by a query?

- [ ] A. ACCOUNT_USAGE.QUERY_HISTORY with QUERY_TEXT column
- [ ] B. ACCOUNT_USAGE.ACCESS_HISTORY with direct and base object columns
- [ ] C. Snowflake Trust Center compliance reports
- [ ] D. INFORMATION_SCHEMA.OBJECT_DEPENDENCIES view

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. ACCOUNT_USAGE.ACCESS_HISTORY with direct and base object columns

**Explanation:**
**ACCOUNT_USAGE.ACCESS_HISTORY** tracks fine-grained data lineage. Each record shows the **direct objects** accessed (e.g., the view queried) and **base objects** (the underlying tables/columns those views resolved to), including whether they were read or written. This enables column-level lineage tracing. OBJECT_DEPENDENCIES tracks DDL-level dependencies between schema objects.
</details>

---

### Question 26
**Domain:** Domain 2 — Governance

An ACCOUNTADMIN sets a resource monitor on a virtual warehouse with NOTIFY_USERS = ('user1','user2'). Where do notifications go when a threshold is crossed?

- [ ] A. To the email addresses associated with user1 and user2 in their Snowflake user profiles
- [ ] B. To a Snowflake-internal message center only visible in Snowsight
- [ ] C. To a webhook URL configured in the Snowflake account notification integration
- [ ] D. To user1 and user2's Snowflake session alerts only when they are logged in

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. To the email addresses associated with user1 and user2 in their Snowflake user profiles

**Explanation:**
Resource monitor notifications go to the **email addresses registered in the Snowflake user profiles** of the specified users. Snowflake must have a valid email on file for the user for the notification to be delivered. There is also account-level email notification support via Notification Integrations for alerts and tasks.
</details>

---

### Question 27
**Domain:** Domain 2 — Governance

What is the purpose of the Trust Center in Snowflake?

- [ ] A. It provides a customer-facing dashboard to monitor Snowflake's SLA compliance and uptime.
- [ ] B. It evaluates the Snowflake account against security best practices and compliance frameworks, surfacing risks and recommendations.
- [ ] C. It manages OAuth tokens and API keys for third-party integrations.
- [ ] D. It is the UI for managing Tri-Secret Secure key rotation.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. It evaluates the Snowflake account against security best practices and compliance frameworks, surfacing risks and recommendations.

**Explanation:**
The **Trust Center** is a Snowflake feature that continuously scans the account for security misconfigurations and compliance risks — such as overly permissive network policies, inactive users with ACCOUNTADMIN, or MFA not enforced — and provides prioritized recommendations. It maps findings to compliance frameworks like CIS, SOC 2, and HIPAA.
</details>

---

### Question 28
**Domain:** Domain 2 — Governance

A company wants to replicate a Snowflake database to a secondary region for disaster recovery, with the secondary being read-only until failover. Which object type is used?

- [ ] A. A cloned database with scheduled refresh via a Task
- [ ] B. A replication group or failover group with the secondary database
- [ ] C. An outbound share to the secondary account
- [ ] D. A Dynamic Table synced to a secondary account

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. A replication group or failover group with the secondary database

**Explanation:**
**Replication groups** and **failover groups** are the correct objects. A **failover group** extends replication groups with the ability to promote the secondary to primary (failover/failback). The secondary database is read-only until promoted. Shares share data between accounts but don't support failover. Clones exist within an account only.
</details>

---

### Question 29
**Domain:** Domain 2 — Governance

A secondary role allows a user to activate additional privileges beyond their primary role. Which function checks whether any active role (primary or secondary) in the current session has a specific privilege?

- [ ] A. CURRENT_ROLE()
- [ ] B. CURRENT_SECONDARY_ROLES()
- [ ] C. IS_ROLE_IN_SESSION()
- [ ] D. HAS_PRIVILEGE()

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. IS_ROLE_IN_SESSION()

**Explanation:**
**IS_ROLE_IN_SESSION('ROLE_NAME')** returns TRUE if the specified role is active in the current session — either as the primary role, an inherited role, or a secondary role. This is important for masking policies and row access policies where you want to check if a user has a role active anywhere in their session hierarchy, not just the primary role returned by CURRENT_ROLE().
</details>

---

### Question 30
**Domain:** Domain 2 — Governance

Object tagging in Snowflake enables which governance capability when combined with masking policies?

- [ ] A. Tag-based masking: a masking policy attached to a tag automatically applies to all columns tagged with that tag
- [ ] B. Tags encrypt the tagged column's data at the storage layer
- [ ] C. Tags trigger automatic data classification reports in the Trust Center
- [ ] D. Tags are required before a column can be included in an ACCESS_HISTORY record

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Tag-based masking: a masking policy attached to a tag automatically applies to all columns tagged with that tag

**Explanation:**
**Tag-based masking policies** allow a masking policy to be assigned to an object tag. When a column is tagged with that tag, the masking policy automatically applies — no need to manually assign the policy to each individual column. This scales governance across thousands of columns by simply tagging sensitive data (e.g., tag 'PII') and attaching the masking policy to the tag.
</details>

---

### Question 31
**Domain:** Domain 2 — Governance

Which credit consumption model applies to the Cloud Services layer?

- [ ] A. Cloud Services always consume credits at a fixed rate regardless of query volume.
- [ ] B. Cloud Services are billed at 10% of daily warehouse compute credits; usage above this threshold is charged.
- [ ] C. Cloud Services credit usage is fully included in warehouse credits and never billed separately.
- [ ] D. Cloud Services are billed only when queries access the ACCOUNT_USAGE schema.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Cloud Services are billed at 10% of daily warehouse compute credits; usage above this threshold is charged.

**Explanation:**
Snowflake provides a daily **free tier for Cloud Services equal to 10% of the daily warehouse compute credits** used. If Cloud Services credit consumption exceeds this threshold in a day, the excess is billed. For most workloads, Cloud Services stays under this 10% ceiling and incurs no additional charge.
</details>

---

### Question 32
**Domain:** Domain 3 — Data Loading

A data engineer stages 50 CSV files in an S3 bucket and runs COPY INTO. After the load, she adds 10 more files to the same S3 location. When she runs COPY INTO again, how many files are loaded?

- [ ] A. All 60 files, because COPY INTO always loads all files in the stage
- [ ] B. Only the 10 new files, because Snowflake tracks which files have already been loaded via load metadata
- [ ] C. The 10 new files only if FORCE = TRUE is set
- [ ] D. No files, because COPY INTO can only be run once per stage

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Only the 10 new files, because Snowflake tracks which files have already been loaded via load metadata

**Explanation:**
Snowflake maintains **load metadata** for each file processed by COPY INTO (stored for 64 days). When COPY INTO runs again, it skips files already in the load history. Only the **10 new files** (not previously loaded) will be ingested. FORCE = TRUE overrides this and reloads all files regardless of load history.
</details>

---

### Question 33
**Domain:** Domain 3 — Data Loading

Which internal stage type is automatically created for each Snowflake user and is private to that user?

- [ ] A. Table stage (@%TABLE_NAME)
- [ ] B. Named internal stage (CREATE STAGE …)
- [ ] C. User stage (@~)
- [ ] D. Schema stage (@SCHEMA_NAME)

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. User stage (@~)

**Explanation:**
The **user stage**, referenced as **@~**, is automatically created for every Snowflake user and is private — only that user can access it. The **table stage** (@%table_name) is auto-created per table and accessible to anyone with table privileges. Named stages are created explicitly and can be shared across users with appropriate grants.
</details>

---

### Question 34
**Domain:** Domain 3 — Data Loading

A COPY INTO command uses a file format with ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE and ON_ERROR = CONTINUE. A CSV row has 3 columns but the target table has 5. What happens?

- [ ] A. The row is skipped and counted as an error because column counts differ.
- [ ] B. The row is loaded; missing columns receive NULL values and extra columns are ignored.
- [ ] C. The entire file is rejected because of the schema mismatch.
- [ ] D. Snowflake automatically infers the schema and adds two new columns to the table.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The row is loaded; missing columns receive NULL values and extra columns are ignored.

**Explanation:**
With **ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE**, Snowflake does not error when the file's column count differs from the table's. Missing columns get **NULL** values and extra columns in the file are ignored. Combined with ON_ERROR = CONTINUE, any remaining column type errors are skipped and loading continues.
</details>

---

### Question 35
**Domain:** Domain 3 — Data Loading

Server-side encryption for an external stage on S3 requires which Snowflake object?

- [ ] A. A storage integration object that provides Snowflake an IAM role to access S3, with encryption handled by the S3 bucket policy
- [ ] B. An API integration to call AWS KMS directly during COPY INTO
- [ ] C. A key-pair authentication configuration on the external stage
- [ ] D. A named encryption policy applied to the stage object

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. A storage integration object that provides Snowflake an IAM role to access S3, with encryption handled by the S3 bucket policy

**Explanation:**
An **S3 external stage** uses a **storage integration** object, which creates an IAM role trusted by Snowflake's AWS account. This avoids storing AWS credentials in Snowflake. Server-side encryption (SSE-S3 or SSE-KMS) is configured on the S3 bucket or specified in the stage definition. Snowflake reads/writes encrypted data using the IAM role's permissions.
</details>

---

### Question 36
**Domain:** Domain 3 — Data Loading

What is the key architectural difference between Snowpipe (file-based) and Snowpipe Streaming?

- [ ] A. Snowpipe Streaming uses micro-batch files; classic Snowpipe uses a REST API for row-level inserts.
- [ ] B. Classic Snowpipe ingests staged files asynchronously via serverless compute; Snowpipe Streaming uses the Streaming Ingest SDK to write rows directly to Snowflake tables without staging files.
- [ ] C. Snowpipe Streaming requires a virtual warehouse; classic Snowpipe is serverless.
- [ ] D. Classic Snowpipe is for semi-structured data only; Snowpipe Streaming handles structured data.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Classic Snowpipe ingests staged files asynchronously via serverless compute; Snowpipe Streaming uses the Streaming Ingest SDK to write rows directly to Snowflake tables without staging files.

**Explanation:**
**Classic Snowpipe** works by loading staged files (S3, Azure Blob, GCS) asynchronously using serverless compute triggered by notifications (SQS, Event Grid) or REST API calls. **Snowpipe Streaming** uses the Streaming Ingest SDK (available via the Kafka Connector or directly) to write rows directly to Snowflake tables in memory without staging files, achieving much lower latency.
</details>

---

### Question 37
**Domain:** Domain 3 — Data Loading

A Snowflake Stream on a table returns change records. Which DML column is added to stream output that indicates whether a row was inserted, updated, or deleted?

- [ ] A. CHANGE_TYPE
- [ ] B. METADATA$ACTION
- [ ] C. STREAM_ACTION
- [ ] D. ROW_CHANGE_TYPE

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. METADATA$ACTION

**Explanation:**
Snowflake streams add three metadata columns: **METADATA$ACTION** (INSERT or DELETE — updates appear as a DELETE + INSERT pair), **METADATA$ISUPDATE** (TRUE if this record is part of an UPDATE operation), and **METADATA$ROW_ID** (unique ID for the row). METADATA$ACTION is the primary column indicating the change type.
</details>

---

### Question 38
**Domain:** Domain 3 — Data Loading

An APPEND_ONLY stream is created on a table. Which DML operations does this stream capture?

- [ ] A. INSERT, UPDATE, and DELETE operations
- [ ] B. INSERT operations only
- [ ] C. INSERT and UPDATE operations only
- [ ] D. DELETE and TRUNCATE operations only

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. INSERT operations only

**Explanation:**
An **APPEND_ONLY stream** captures only **INSERT** operations — it does not track UPDATEs or DELETEs. This is more efficient for use cases like event logs or CDC where only new rows matter. A standard (DEFAULT) stream captures inserts, updates, and deletes. APPEND_ONLY streams have lower overhead and work on append-only source tables.
</details>

---

### Question 39
**Domain:** Domain 3 — Data Loading

A task runs every 5 minutes but only if a stream on the source table has data. Which clause in the CREATE TASK statement implements this behavior?

- [ ] A. WHEN STREAM_HAS_DATA(stream_name) = TRUE in the SCHEDULE clause
- [ ] B. WHEN SYSTEM$STREAM_HAS_DATA('stream_name') in the task condition
- [ ] C. IF EXISTS (SELECT * FROM stream_name LIMIT 1) in the task body
- [ ] D. TRIGGER ON STREAM stream_name in the task definition

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. WHEN SYSTEM$STREAM_HAS_DATA('stream_name') in the task condition

**Explanation:**
The **WHEN SYSTEM$STREAM_HAS_DATA('stream_name')** clause in a CREATE TASK statement acts as a condition — the task checks if the stream has unconsumed records before executing. If the stream is empty, the task is skipped (no warehouse consumption). This is the standard pattern for stream-based task pipelines.
</details>

---

### Question 40
**Domain:** Domain 3 — Data Loading

Which Snowflake connector allows direct integration with dbt Core for transformation pipelines without requiring an external database connection string?

- [ ] A. Snowflake JDBC Driver
- [ ] B. Snowflake ODBC Driver
- [ ] C. Snowflake Connector for Python (snowflake-connector-python)
- [ ] D. dbt natively connects to Snowflake via the Snowflake dbt adapter using standard account credentials

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** D. dbt natively connects to Snowflake via the Snowflake dbt adapter using standard account credentials

**Explanation:**
**dbt** connects to Snowflake using the **dbt-snowflake adapter**, which internally uses the Snowflake Python connector. In dbt's profiles.yml, you specify the Snowflake account, user, role, warehouse, and database. There is no special Snowflake-side connector object needed — standard Snowflake account credentials and JDBC/Python connectivity handle it.
</details>

---

### Question 41
**Domain:** Domain 3 — Data Loading

A directory table on an external stage provides which capability NOT available from a plain external stage listing?

- [ ] A. It allows COPY INTO to load files from the external stage
- [ ] B. It maintains a catalog of staged files with metadata (file name, size, last modified) queryable via SQL and refreshable on demand
- [ ] C. It encrypts files in the external stage using Snowflake-managed keys
- [ ] D. It automatically triggers Snowpipe when new files are detected

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. It maintains a catalog of staged files with metadata (file name, size, last modified) queryable via SQL and refreshable on demand

**Explanation:**
A **directory table** attached to a stage provides a queryable catalog of the files in the stage — you can SELECT from it to get file paths, sizes, ETag, and last-modified timestamps. It must be refreshed with ALTER STAGE … REFRESH to pick up new files. It does not trigger Snowpipe, handle encryption, or affect COPY INTO behavior.
</details>

---

### Question 42
**Domain:** Domain 3 — Data Loading

A Git integration in Snowflake allows developers to do which of the following directly in Snowflake?

- [ ] A. Push committed Snowflake Notebook changes directly to a GitHub repository
- [ ] B. Reference files from a connected Git repository as a Snowflake stage, enabling SQL scripts and Python files to be executed from the repo
- [ ] C. Trigger Snowflake Tasks from GitHub Actions webhooks without an external integration
- [ ] D. Sync Snowflake table schemas with Prisma schema files in a Git repository

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Reference files from a connected Git repository as a Snowflake stage, enabling SQL scripts and Python files to be executed from the repo

**Explanation:**
The **Snowflake Git integration** creates a special repository stage that mirrors a connected Git repository. Developers can reference files (SQL scripts, Python UDFs, Snowpark code) from the repository stage and execute them in Snowflake. This enables source-control-driven workflows. It does not enable pushing from Snowflake to Git or triggering tasks from GitHub Actions directly.
</details>

---

### Question 43
**Domain:** Domain 3 — Data Loading

When unloading data from Snowflake to an external stage using COPY INTO (outbound), which statement is TRUE about file splitting?

- [ ] A. Snowflake always creates a single file per COPY INTO unload, regardless of data size.
- [ ] B. Snowflake splits output into multiple files based on the MAX_FILE_SIZE parameter (default 16MB) and warehouse parallelism.
- [ ] C. File splitting is not supported for COPY INTO unloading; use UNLOAD command instead.
- [ ] D. Each virtual warehouse node creates exactly one output file, so a 4-node XL warehouse produces 4 files.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowflake splits output into multiple files based on the MAX_FILE_SIZE parameter (default 16MB) and warehouse parallelism.

**Explanation:**
Snowflake COPY INTO (unload) splits output into **multiple parallel files** based on the **MAX_FILE_SIZE** parameter (default 16MB per file) and the number of threads/nodes in the warehouse. This parallelism speeds up large unloads. You can also use SINGLE = TRUE to force a single output file at the cost of performance.
</details>

---

### Question 44
**Domain:** Domain 4 — Performance

A query profile shows 'Bytes Spilled to Remote Storage: 50GB' for an aggregation query on a Medium warehouse. Which action is MOST effective at eliminating remote spill?

- [ ] A. Add a clustering key to the table being aggregated
- [ ] B. Upgrade the warehouse to a larger size to increase available memory and local SSD
- [ ] C. Enable the Query Acceleration Service on the warehouse
- [ ] D. Create a materialized view with the pre-aggregated results

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Upgrade the warehouse to a larger size to increase available memory and local SSD

**Explanation:**
**Remote storage spill** occurs when intermediate data exceeds both the warehouse's memory AND local SSD capacity — data overflows to S3/Azure/GCS, which is very slow. The most direct fix is **increasing warehouse size** to add more memory and local disk. Clustering helps with partition pruning but doesn't address memory pressure during aggregation. QAS and materialized views address different problems.
</details>

---

### Question 45
**Domain:** Domain 4 — Performance

Which Snowflake caching layer is shared across ALL virtual warehouses in an account and returns the exact same result set for identical repeated queries?

- [ ] A. Warehouse (local disk) cache
- [ ] B. Metadata cache
- [ ] C. Result cache (query result cache)
- [ ] D. Remote storage cache

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Result cache (query result cache)

**Explanation:**
The **result cache (query result cache)** is maintained in the Cloud Services layer and is shared across all virtual warehouses in the account. When an identical query is resubmitted (same SQL text, same underlying data, within 24 hours), Snowflake returns the cached result instantly — no warehouse is started or billed. The warehouse cache is per-warehouse; metadata cache is for schema information.
</details>

---

### Question 46
**Domain:** Domain 4 — Performance

A table has a compound clustering key on (COUNTRY, YEAR). A query filters on YEAR only (no COUNTRY filter). Which statement about partition pruning is TRUE?

- [ ] A. Snowflake cannot prune any partitions because the leading key (COUNTRY) is not in the WHERE clause.
- [ ] B. Snowflake can perform partial pruning on YEAR because micro-partition metadata stores min/max for all clustered columns.
- [ ] C. Snowflake prunes fully because YEAR is in the clustering key regardless of order.
- [ ] D. Snowflake rewrites the query to add COUNTRY = '*' to enable pruning.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowflake can perform partial pruning on YEAR because micro-partition metadata stores min/max for all clustered columns.

**Explanation:**
With a compound clustering key (COUNTRY, YEAR), data is sorted primarily by COUNTRY, then by YEAR within each COUNTRY group. A filter on only YEAR (without COUNTRY) means YEAR values are interleaved across many micro-partitions for different countries. Snowflake can do **partial pruning** using the per-column min/max metadata, but it's less effective than filtering on the leading key. A full table scan is likely if YEAR has wide distribution.
</details>

---

### Question 47
**Domain:** Domain 4 — Performance

The Search Optimization Service (SOS) is BEST suited for which workload?

- [ ] A. Range scans on date columns across billions of rows
- [ ] B. Selective equality and IN-list lookups on high-cardinality columns (e.g., order_id, email, UUID)
- [ ] C. GROUP BY aggregations on low-cardinality dimension columns
- [ ] D. JOIN operations between large fact and dimension tables

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Selective equality and IN-list lookups on high-cardinality columns (e.g., order_id, email, UUID)

**Explanation:**
The **Search Optimization Service** builds a persistent search access path (SAP) on tables, dramatically accelerating **highly selective point lookups** — equality predicates (=) and IN-list queries on high-cardinality columns like order IDs, UUIDs, or email addresses. It is NOT designed for range scans, aggregations, or JOINs. Those are better served by clustering, materialized views, or warehouse sizing.
</details>

---

### Question 48
**Domain:** Domain 4 — Performance

A Materialized View (MV) is defined over a base table. When the base table is updated via DML, how does Snowflake handle MV maintenance?

- [ ] A. The MV is immediately and synchronously updated as part of the DML transaction.
- [ ] B. The MV is marked stale and refreshed lazily when queried.
- [ ] C. Snowflake automatically and asynchronously maintains the MV in the background using serverless compute; the DML transaction does not block on MV refresh.
- [ ] D. The MV must be manually refreshed with ALTER MATERIALIZED VIEW … REFRESH.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Snowflake automatically and asynchronously maintains the MV in the background using serverless compute; the DML transaction does not block on MV refresh.

**Explanation:**
Snowflake **automatically maintains materialized views** in the background using serverless compute. When the base table changes, the MV is updated asynchronously — the DML transaction completes immediately without waiting for MV refresh. Queries against the MV always see consistent, up-to-date results because Snowflake either uses the refreshed MV or transparently falls back to the base table. No manual REFRESH is needed (unlike some other databases).
</details>

---

### Question 49
**Domain:** Domain 4 — Performance

The Query Acceleration Service (QAS) is enabled on a warehouse with MAX_SCALE_FACTOR = 5. What does this mean?

- [ ] A. The warehouse can scale out to 5 additional clusters under the multi-cluster policy.
- [ ] B. QAS can use up to 5× the warehouse's compute size worth of serverless compute for eligible query portions.
- [ ] C. The query result is cached for 5 times the normal 24-hour TTL.
- [ ] D. QAS splits each query into 5 parallel sub-queries automatically.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. QAS can use up to 5× the warehouse's compute size worth of serverless compute for eligible query portions.

**Explanation:**
**MAX_SCALE_FACTOR** for QAS limits how much serverless compute QAS can allocate relative to the warehouse size. A MAX_SCALE_FACTOR of 5 means QAS can use up to **5× the warehouse's credit-equivalent compute** to offload and accelerate eligible large scan portions of queries. Setting it higher allows more acceleration but at higher serverless cost.
</details>

---

### Question 50
**Domain:** Domain 4 — Performance

A developer is writing a SQL query that needs to calculate a running total of sales per customer, partitioned by customer_id and ordered by sale_date. Which SQL feature should be used?

- [ ] A. GROUP BY with a correlated subquery
- [ ] B. A window function with PARTITION BY customer_id ORDER BY sale_date using SUM() OVER(...)
- [ ] C. A recursive CTE accumulating totals by customer
- [ ] D. PIVOT on customer_id with SUM aggregation

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. A window function with PARTITION BY customer_id ORDER BY sale_date using SUM() OVER(...)

**Explanation:**
**Window functions** (analytic functions) are the correct SQL feature for running totals. SUM(sales_amount) OVER (PARTITION BY customer_id ORDER BY sale_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) computes the cumulative sum per customer in date order. Window functions operate on a partition of rows without collapsing them, unlike GROUP BY.
</details>

---

### Question 51
**Domain:** Domain 4 — Performance

An EXPLAIN plan in Snowflake shows 'Bytes Scanned: 10TB' for a simple query on a clustered table. The table has 500GB of actual data. What does this indicate?

- [ ] A. Snowflake is reading compressed data, which decompresses to 10TB.
- [ ] B. The clustering key does not align with the query's filter predicates, so Snowflake is scanning most micro-partitions instead of pruning effectively.
- [ ] C. The table was replicated 20 times, each copy adding 500GB to the scan.
- [ ] D. The result cache is disabled, forcing full re-reads.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The clustering key does not align with the query's filter predicates, so Snowflake is scanning most micro-partitions instead of pruning effectively.

**Explanation:**
If bytes scanned (10TB) far exceeds the table size (500GB), it indicates poor **partition pruning**. The query filter columns likely don't align with the table's clustering key, so Snowflake must read most or all micro-partitions. The fix is either to add/change the clustering key to match the query's filter columns, or to restructure the query.
</details>

---

### Question 52
**Domain:** Domain 4 — Performance

Which ACCOUNT_USAGE view is most useful for identifying queries that ran on a specific warehouse and consumed the most credits over the past 30 days?

- [ ] A. ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
- [ ] B. ACCOUNT_USAGE.QUERY_HISTORY joined with WAREHOUSE_METERING_HISTORY
- [ ] C. ACCOUNT_USAGE.QUERY_HISTORY filtering on WAREHOUSE_NAME and ordering by CREDITS_USED_CLOUD_SERVICES DESC
- [ ] D. ACCOUNT_USAGE.WAREHOUSE_LOAD_HISTORY

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. ACCOUNT_USAGE.QUERY_HISTORY joined with WAREHOUSE_METERING_HISTORY

**Explanation:**
**ACCOUNT_USAGE.QUERY_HISTORY** contains per-query details including WAREHOUSE_NAME, EXECUTION_TIME, BYTES_SCANNED, and CREDITS_USED_CLOUD_SERVICES. Joining it with **WAREHOUSE_METERING_HISTORY** (which tracks credit usage per warehouse per hour) gives the most complete picture of credit consumption by query. WAREHOUSE_LOAD_HISTORY shows concurrency and queuing, not per-query credit consumption.
</details>

---

### Question 53
**Domain:** Domain 4 — Performance

A semi-structured JSON column VARIANT stores nested arrays of events. Which Snowflake function is used to flatten these arrays into individual rows for SQL processing?

- [ ] A. JSON_EXTRACT()
- [ ] B. LATERAL FLATTEN()
- [ ] C. UNNEST()
- [ ] D. PARSE_JSON() with a CROSS JOIN

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. LATERAL FLATTEN()

**Explanation:**
**LATERAL FLATTEN()** is Snowflake's function for expanding arrays (and objects) within VARIANT columns into individual rows. It is used in the FROM clause with LATERAL JOIN: FROM table, LATERAL FLATTEN(INPUT => col:array_field) f. The output includes VALUE (element), INDEX (array position), KEY, and PATH columns.
</details>

---

### Question 54
**Domain:** Domain 4 — Performance

What is the purpose of CLUSTER_BY on a Snowflake table, and how does automatic clustering work?

- [ ] A. CLUSTER_BY physically sorts all rows in the table on INSERT; Snowflake automatically resorts on every DML.
- [ ] B. CLUSTER_BY defines a target clustering order; Snowflake's Automatic Clustering service periodically re-clusters micro-partitions in the background using serverless compute when they become poorly clustered.
- [ ] C. CLUSTER_BY creates a traditional B-tree index on the specified columns.
- [ ] D. CLUSTER_BY forces the warehouse to read micro-partitions in the specified column order.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. CLUSTER_BY defines a target clustering order; Snowflake's Automatic Clustering service periodically re-clusters micro-partitions in the background using serverless compute when they become poorly clustered.

**Explanation:**
**CLUSTER_BY** defines the desired clustering order for micro-partitions. Snowflake's **Automatic Clustering** service monitors the average overlap depth (a measure of clustering quality) and re-clusters micro-partitions in the background using **serverless compute** when the table becomes de-clustered due to DML. This is a continuous background process — users don't trigger it manually.
</details>

---

### Question 55
**Domain:** Domain 4 — Performance

A query joins a 10TB fact table with a 1MB dimension table. The query profile shows significant 'Build Hash Table' time. Which Snowflake optimization automatically addresses this scenario?

- [ ] A. Clustering key on the fact table join column
- [ ] B. Query Acceleration Service offloading the hash build
- [ ] C. Automatic broadcast join optimization, where the small dimension table is broadcast to each warehouse node
- [ ] D. Materialized view caching the join result

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Automatic broadcast join optimization, where the small dimension table is broadcast to each warehouse node

**Explanation:**
Snowflake's query optimizer automatically applies a **broadcast join** when one table in a join is small enough to fit in memory. The small dimension table (1MB) is broadcast to every compute node, and each node joins its portion of the large fact table locally — eliminating network shuffle. This is a built-in optimizer decision, not a user-configured parameter.
</details>

---

### Question 56
**Domain:** Domain 4 — Performance

Which statement about the SHOW TABLES command versus querying INFORMATION_SCHEMA.TABLES is TRUE from a performance perspective?

- [ ] A. SHOW TABLES requires a virtual warehouse; INFORMATION_SCHEMA queries are always serverless.
- [ ] B. Both SHOW TABLES and INFORMATION_SCHEMA queries use the metadata cache and do not require a virtual warehouse.
- [ ] C. INFORMATION_SCHEMA.TABLES queries require a virtual warehouse for tables with more than 100 columns.
- [ ] D. SHOW TABLES is deprecated; only INFORMATION_SCHEMA should be used.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Both SHOW TABLES and INFORMATION_SCHEMA queries use the metadata cache and do not require a virtual warehouse.

**Explanation:**
Both **SHOW TABLES** and queries against **INFORMATION_SCHEMA.TABLES** are served from the **Cloud Services metadata cache** and do **not require a virtual warehouse**. They are essentially metadata operations. This is why they execute almost instantly regardless of table data size.
</details>

---

### Question 57
**Domain:** Domain 5 — Collaboration

A provider shares a table with a consumer using Secure Data Sharing. The consumer queries the shared table. On whose account are the compute credits charged for the query?

- [ ] A. The provider's account, because the data resides in their storage
- [ ] B. The consumer's account, because the consumer's virtual warehouse executes the query
- [ ] C. Credits are split 50/50 between provider and consumer
- [ ] D. Snowflake absorbs the compute cost for all Data Sharing queries

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The consumer's account, because the consumer's virtual warehouse executes the query

**Explanation:**
In **Secure Data Sharing**, data is NOT copied — the consumer accesses the provider's data directly. However, the **consumer's virtual warehouse** executes the query, so **compute credits are charged to the consumer's account**. Storage costs remain with the provider. This is a key financial model difference from Reader Accounts, where the provider pays for all compute.
</details>

---

### Question 58
**Domain:** Domain 5 — Collaboration

What is the maximum Time Travel retention period for a permanent table on Enterprise edition?

- [ ] A. 7 days
- [ ] B. 30 days
- [ ] C. 90 days
- [ ] D. 365 days

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. 90 days

**Explanation:**
On **Enterprise edition and above**, permanent tables support up to **90 days of Time Travel**. The default is 1 day. Standard edition supports up to 1 day. Transient and temporary tables support 0 or 1 day regardless of edition. Increasing retention to 90 days increases storage costs because all changed data versions are retained.
</details>

---

### Question 59
**Domain:** Domain 5 — Collaboration

What is Fail-safe in Snowflake and how does it differ from Time Travel?

- [ ] A. Fail-safe is a 7-day period after Time Travel expires during which Snowflake can recover data internally; it cannot be accessed by customers via SQL.
- [ ] B. Fail-safe is an additional 24-hour recovery window after Time Travel that users can query with AT(TIMESTAMP =...) syntax.
- [ ] C. Fail-safe is enabled only on Business Critical edition and replaces Time Travel.
- [ ] D. Fail-safe permanently archives data in a separate Snowflake-managed S3 bucket accessible via support tickets.

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Fail-safe is a 7-day period after Time Travel expires during which Snowflake can recover data internally; it cannot be accessed by customers via SQL.

**Explanation:**
**Fail-safe** provides a **7-day** disaster recovery period for permanent tables that begins after the Time Travel retention period expires. During Fail-safe, **only Snowflake Support** can recover data — customers cannot query Fail-safe data via SQL or AT() syntax. It protects against catastrophic failures. Transient and temporary tables have no Fail-safe period, which is why they cost less storage.
</details>

---

### Question 60
**Domain:** Domain 5 — Collaboration

A CLONE of a 10TB table is created at 3:00 PM. By 4:00 PM, 1TB of data has been modified in the clone. How much additional storage does the clone consume at 4:00 PM?

- [ ] A. 10TB — a full copy of the original table
- [ ] B. 0TB — clones share all storage with the original and never incur additional cost
- [ ] C. Approximately 1TB — only the modified/new micro-partitions added after cloning are billed to the clone
- [ ] D. 5TB — clones always store 50% of the original table

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Approximately 1TB — only the modified/new micro-partitions added after cloning are billed to the clone

**Explanation:**
Snowflake **zero-copy cloning** creates a clone that initially shares all micro-partitions with the source (metadata copy only — no data copied). Storage is billed to the clone only for **micro-partitions that are added or modified after cloning**. At 4:00 PM, only the ~1TB of modified data is uniquely stored by the clone. The original 10TB is shared and billed only once.
</details>

---

### Question 61
**Domain:** Domain 5 — Collaboration

A data provider wants to allow a consumer to further share a received dataset with additional Snowflake accounts. Which feature enables this?

- [ ] A. This is not possible; Snowflake shares cannot be re-shared
- [ ] B. Re-sharing: the provider must grant the consumer the ability to reshare by including the GRANT OPTION on the share
- [ ] C. The consumer creates a new outbound share from the received data, which is natively supported
- [ ] D. The consumer must clone the shared data and create a new share from the clone

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. The consumer creates a new outbound share from the received data, which is natively supported

**Explanation:**
**Re-sharing** is natively supported in Snowflake. A consumer account that has received a share can create their own outbound share from the shared data to additional accounts, without needing to copy the data. The consumer needs privileges on the shared objects (granted from the inbound share) and creates a new share object. The provider doesn't need to explicitly grant a special re-share option.
</details>

---

### Question 62
**Domain:** Domain 5 — Collaboration

Which SQL syntax is used to query a Snowflake table AS OF a specific past timestamp using Time Travel?

- [ ] A. SELECT * FROM table WHERE TIMESTAMP = '2024-01-01 00:00:00'
- [ ] B. SELECT * FROM table AT(TIMESTAMP => '2024-01-01 00:00:00'::TIMESTAMP_NTZ)
- [ ] C. SELECT * FROM table VERSION AS OF '2024-01-01 00:00:00'
- [ ] D. SELECT * FROM table TRAVEL_TO TIMESTAMP '2024-01-01 00:00:00'

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. SELECT * FROM table AT(TIMESTAMP => '2024-01-01 00:00:00'::TIMESTAMP_NTZ)

**Explanation:**
Snowflake Time Travel uses the **AT or BEFORE clause**: SELECT * FROM table **AT(TIMESTAMP => '2024-01-01'::TIMESTAMP_NTZ)**. You can also use AT(OFFSET => -3600) for seconds ago, or AT(STATEMENT => 'query_id') to travel to the state before a specific query ran. The VERSION AS OF syntax is not Snowflake's syntax.
</details>

---

### Question 63
**Domain:** Domain 5 — Collaboration

A Snowflake Marketplace listing is published as a 'Private Listing'. What does this mean?

- [ ] A. The data is encrypted end-to-end and only the invited consumer can decrypt it
- [ ] B. The listing is only discoverable and accessible by specific Snowflake accounts the provider explicitly invites
- [ ] C. The listing is visible to all Marketplace users but requires a paid subscription
- [ ] D. The listing is visible only within the provider's Snowflake organization, not to external accounts

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The listing is only discoverable and accessible by specific Snowflake accounts the provider explicitly invites

**Explanation:**
A **Private Listing** on the Snowflake Marketplace is visible and accessible only to **specific Snowflake accounts explicitly invited** by the provider. It does not appear in the public Marketplace catalog. This is used for controlled data distribution to specific customers or partners. A Public Listing is discoverable by any Snowflake account browsing the Marketplace.
</details>

---

### Question 64
**Domain:** Domain 5 — Collaboration

A Native App built with the Snowflake Native App Framework is installed by a consumer. Where does the application's logic execute?

- [ ] A. On the provider's Snowflake account, using the provider's compute
- [ ] B. On the consumer's Snowflake account, using the consumer's compute and accessing only data the consumer grants to the app
- [ ] C. On Snowflake's shared serverless compute, billed equally to provider and consumer
- [ ] D. In Snowflake Container Services running in the provider's cloud region

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. On the consumer's Snowflake account, using the consumer's compute and accessing only data the consumer grants to the app

**Explanation:**
A **Native App** is deployed and runs entirely in the **consumer's Snowflake account**. The provider packages the application logic (stored procedures, UDFs, Streamlit UI) using the Native App Framework, and the consumer installs it. The app runs on the consumer's compute and accesses only data within the consumer's account (plus any data the consumer explicitly grants to the app). The provider's data is NOT directly accessible.
</details>

---

### Question 65
**Domain:** Domain 5 — Collaboration

A data clean room in Snowflake enables which privacy-preserving collaboration scenario?

- [ ] A. Two organizations share raw customer PII tables with each other for joint analysis
- [ ] B. Two organizations run overlapping customer analysis without either party seeing the other's raw customer data — only aggregate or anonymized results are produced
- [ ] C. A single organization archives sensitive tables to an external S3 bucket for compliance
- [ ] D. Snowflake encrypts shared data with both parties' keys before allowing any query

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Two organizations run overlapping customer analysis without either party seeing the other's raw customer data — only aggregate or anonymized results are produced

**Explanation:**
A **data clean room** allows two organizations to collaboratively analyze overlapping data (e.g., customer match rates, campaign attribution) without exposing raw records to each other. Only aggregate or privacy-preserved query results are produced. The raw data of each party stays within their own Snowflake account; only approved, limited queries against the shared schema are allowed.
</details>

---

### Question 66
**Domain:** Domain 1 — Architecture

A table has 10 billion rows. An analyst runs SELECT COUNT(*) FROM large_table. Which optimization makes this return in milliseconds without a warehouse?

- [ ] A. Result cache hit from a prior COUNT(*) run
- [ ] B. Metadata cache: Snowflake maintains row counts as metadata per table, serving COUNT(*) from the Cloud Services layer
- [ ] C. Automatic sampling: Snowflake returns an approximate count without scanning
- [ ] D. The query cannot complete without a warehouse for tables over 1 billion rows

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Metadata cache: Snowflake maintains row counts as metadata per table, serving COUNT(*) from the Cloud Services layer

**Explanation:**
Snowflake maintains precise row count statistics as **table-level metadata** in the Cloud Services layer. A simple COUNT(*) with no WHERE clause is resolved from this metadata cache — **no virtual warehouse is needed**, and results appear nearly instantly. This is a common exam trap: count queries against unfiltered tables are a known no-warehouse optimization.
</details>

---

### Question 67
**Domain:** Domain 1 — Architecture

Which editions support Snowflake's multi-cluster warehouse feature for handling high-concurrency workloads?

- [ ] A. Standard and above
- [ ] B. Enterprise and above
- [ ] C. Business Critical and above
- [ ] D. All editions including Virtual Private Snowflake (VPS)

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Enterprise and above

**Explanation:**
**Multi-cluster warehouses** are available on **Enterprise edition and above** (Enterprise, Business Critical, Virtual Private Snowflake). Standard edition supports single-cluster warehouses only. Multi-cluster enables automatic scaling out to additional clusters when concurrency demands exceed a single cluster's capacity.
</details>

---

### Question 68
**Domain:** Domain 1 — Architecture

Streamlit in Snowflake differs from hosting a Streamlit app on an external server in which key way?

- [ ] A. Streamlit in Snowflake does not support Python — only SQL widgets are available
- [ ] B. Streamlit in Snowflake runs the app within the Snowflake security perimeter, with direct access to Snowflake data via the Snowpark Session without external connectivity
- [ ] C. Streamlit in Snowflake requires a Snowpark-Optimized warehouse for all apps
- [ ] D. Streamlit in Snowflake is read-only; it cannot write data back to Snowflake tables

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Streamlit in Snowflake runs the app within the Snowflake security perimeter, with direct access to Snowflake data via the Snowpark Session without external connectivity

**Explanation:**
**Streamlit in Snowflake (SiS)** runs the Streamlit application entirely within Snowflake's environment. It uses a built-in **Snowpark Session** to query and write Snowflake data directly without external connectivity. The app inherits the caller's Snowflake role and permissions. Compared to external Streamlit, there's no need to manage credentials, network access, or external infrastructure.
</details>

---

### Question 69
**Domain:** Domain 1 — Architecture

A Snowflake account is in the AWS us-east-1 region. A partner account is in AWS eu-west-1. Direct Secure Data Sharing between these accounts is:

- [ ] A. Possible natively — Secure Data Sharing works across any two Snowflake accounts globally
- [ ] B. Not possible directly — both accounts must be in the same region and cloud for Secure Data Sharing; cross-region requires replication first
- [ ] C. Possible using a Data Exchange listing, which automatically copies data across regions
- [ ] D. Possible only if both accounts are in the same Snowflake Organization

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Not possible directly — both accounts must be in the same region and cloud for Secure Data Sharing; cross-region requires replication first

**Explanation:**
**Secure Data Sharing** only works between accounts in the **same cloud provider and region**. For cross-region or cross-cloud sharing, the provider must first replicate the database to a secondary account in the consumer's region/cloud, and then share from that secondary. Data Exchange listings have similar constraints.
</details>

---

### Question 70
**Domain:** Domain 1 — Architecture

Which Snowflake feature allows a Python developer to write a Pandas DataFrame transformation that runs at scale directly in Snowflake without moving data out?

- [ ] A. Snowflake Cortex Analyst
- [ ] B. Snowpark for Python with modin or Snowpark pandas
- [ ] C. External Function calling a Python Lambda
- [ ] D. Python UDF with IMPORT of a pandas wheel

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowpark for Python with modin or Snowpark pandas

**Explanation:**
**Snowpark for Python** includes **Snowpark pandas** (based on Modin), which provides a pandas-compatible API that runs natively in Snowflake on the warehouse compute. DataFrames are translated into Snowflake SQL plans executed in the warehouse — no data leaves Snowflake. A standard Python UDF runs Python per-row but doesn't offer full DataFrame semantics at scale.
</details>

---

### Question 71
**Domain:** Domain 2 — Governance

The ACCOUNTADMIN role grants a privilege on a table to the SYSADMIN role. Which principle of Snowflake's access control does this represent?

- [ ] A. Discretionary Access Control (DAC) — the owning role decides who gets access
- [ ] B. Mandatory Access Control (MAC) — Snowflake enforces labels
- [ ] C. Role-Based Access Control (RBAC) — access is based on job function
- [ ] D. Attribute-Based Access Control (ABAC) — access is based on object attributes

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. Discretionary Access Control (DAC) — the owning role decides who gets access

**Explanation:**
**Discretionary Access Control (DAC)** means the object owner (or a role with sufficient privilege) decides who else gets access. In Snowflake, any role that OWNS an object — or has been granted MANAGE GRANTS — can grant that object's privileges to others. This complements RBAC (which organizes privileges into roles). Snowflake uses both DAC and RBAC together.
</details>

---

### Question 72
**Domain:** Domain 2 — Governance

A privilege was granted WITH GRANT OPTION. What does this allow the grantee to do?

- [ ] A. The grantee can use the privilege but cannot see the grant in SHOW GRANTS
- [ ] B. The grantee can further grant the same privilege to other roles
- [ ] C. The grantee receives a temporary privilege that expires after 24 hours
- [ ] D. The grantee can revoke the privilege from the original grantor

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The grantee can further grant the same privilege to other roles

**Explanation:**
**WITH GRANT OPTION** allows the grantee role to **further grant the same privilege** to other roles or users. Without GRANT OPTION, the grantee can use the privilege but cannot propagate it. This is the Snowflake equivalent of SQL's GRANT WITH GRANT OPTION. ACCOUNTADMIN can always manage grants via MANAGE GRANTS privilege regardless.
</details>

---

### Question 73
**Domain:** Domain 2 — Governance

A Snowflake account uses federated authentication (SSO) with an Identity Provider (IdP). Which protocol does Snowflake use for this integration?

- [ ] A. OAuth 2.0 with PKCE flow
- [ ] B. SAML 2.0
- [ ] C. OpenID Connect (OIDC)
- [ ] D. Kerberos

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. SAML 2.0

**Explanation:**
Snowflake supports **SAML 2.0** for federated authentication (SSO). The Snowflake account is configured as a Service Provider (SP) and the corporate IdP (Okta, Azure AD, ADFS, etc.) is the Identity Provider (IdP). Snowflake also supports OAuth 2.0 for client application authorization, but SSO/federated authentication specifically uses SAML 2.0.
</details>

---

### Question 74
**Domain:** Domain 2 — Governance

Which ACCOUNT_USAGE view should be queried to find which roles were granted to which users over the past year?

- [ ] A. ACCOUNT_USAGE.GRANTS_TO_USERS
- [ ] B. ACCOUNT_USAGE.ROLE_GRANTS
- [ ] C. ACCOUNT_USAGE.ROLES
- [ ] D. ACCOUNT_USAGE.ACCESS_HISTORY

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. ACCOUNT_USAGE.GRANTS_TO_USERS

**Explanation:**
**ACCOUNT_USAGE.GRANTS_TO_USERS** records the history of role grants to users, including the grant date, grantor role, and whether the grant is still active. This is different from ROLE_GRANTS (which tracks privilege grants to roles). ACCOUNT_USAGE retains this data for up to 365 days.
</details>

---

### Question 75
**Domain:** Domain 2 — Governance

A privacy policy in Snowflake is applied to a table. What does a privacy policy control?

- [ ] A. It defines which roles can see the table in the INFORMATION_SCHEMA
- [ ] B. It controls the purpose-of-use and consent tracking for data in the table, part of Snowflake's data governance for privacy compliance
- [ ] C. It replaces column-level masking with differential privacy noise injection
- [ ] D. It restricts the table from being included in any outbound shares

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. It controls the purpose-of-use and consent tracking for data in the table, part of Snowflake's data governance for privacy compliance

**Explanation:**
**Privacy policies** in Snowflake are part of the Horizon data governance suite. They attach metadata about the purpose of use and data consent requirements to tables. They are used alongside data classification and object tagging for privacy compliance (GDPR, CCPA). They do NOT inject noise, replace masking policies, or block sharing — they are governance metadata.
</details>

---

### Question 76
**Domain:** Domain 3 — Data Loading

A VARIANT column contains: {'price': '19.99', 'qty': '5'}. A developer tries SELECT v:price * v:qty FROM t. The result is NULL. Why?

- [ ] A. VARIANT arithmetic is not supported in Snowflake
- [ ] B. The VARIANT values are stored as strings (quoted), so casting is needed: v:price::FLOAT * v:qty::INT
- [ ] C. The colon operator returns OBJECT type, not numeric
- [ ] D. VARIANT columns require PARSE_JSON() before field extraction

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The VARIANT values are stored as strings (quoted), so casting is needed: v:price::FLOAT * v:qty::INT

**Explanation:**
VARIANT field extraction preserves the JSON type. Since 'price' and 'qty' are **quoted strings** in the JSON (not numbers), the colon operator returns string VARIANTs. Multiplying two string VARIANTs returns NULL. The fix is explicit **casting**: v:price::FLOAT * v:qty::INT. If the JSON had unquoted numbers (e.g., {'price': 19.99}), the arithmetic would work directly.
</details>

---

### Question 77
**Domain:** Domain 3 — Data Loading

Which file format option in Snowflake allows loading of Parquet files directly into a VARIANT column without transformation?

- [ ] A. FILE_FORMAT = (TYPE = CSV PARQUET_CONVERT = TRUE)
- [ ] B. FILE_FORMAT = (TYPE = PARQUET) with COPY INTO targeting a VARIANT or OBJECT column
- [ ] C. Parquet files must be converted to JSON before loading into Snowflake
- [ ] D. FILE_FORMAT = (TYPE = AVRO) handles Parquet natively

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. FILE_FORMAT = (TYPE = PARQUET) with COPY INTO targeting a VARIANT or OBJECT column

**Explanation:**
Snowflake natively supports **Parquet as a file format** (TYPE = PARQUET). COPY INTO can load Parquet files directly into a table with a VARIANT column — each Parquet row becomes a VARIANT object. You can also load into individual typed columns by specifying the column mapping. No pre-conversion to JSON is needed.
</details>

---

### Question 78
**Domain:** Domain 3 — Data Loading

A task is defined with SCHEDULE = '5 MINUTE'. What is the MINIMUM interval allowed for task scheduling?

- [ ] A. 1 minute
- [ ] B. 5 minutes
- [ ] C. 15 minutes
- [ ] D. 1 hour

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. 1 minute

**Explanation:**
Snowflake Tasks support a minimum schedule of **1 minute** (SCHEDULE = '1 MINUTE'). For sub-minute latency requirements, Snowpipe Streaming or event-driven architectures are needed. Tasks also support CRON expressions for more complex scheduling (e.g., specific hours/days).
</details>

---

### Question 79
**Domain:** Domain 3 — Data Loading

A data engineer wants to load JSON files from Azure Blob Storage into Snowflake. Which combination of objects is needed?

- [ ] A. A named external stage pointing to Azure Blob, a storage integration with Azure credentials, and a file format of TYPE = JSON
- [ ] B. A named internal stage, a Snowpipe, and an Azure Service Bus trigger
- [ ] C. An API integration with Azure Functions and a webhook receiver
- [ ] D. A direct COPY INTO from 'azure://container/path' with no stage or integration needed

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. A named external stage pointing to Azure Blob, a storage integration with Azure credentials, and a file format of TYPE = JSON

**Explanation:**
To load from Azure Blob Storage: (1) create a **storage integration** (using the Azure service principal / managed identity to avoid storing credentials), (2) create a named **external stage** pointing to the Azure Blob container using the storage integration, (3) define a **FILE FORMAT of TYPE = JSON**, (4) run COPY INTO from the stage. All three objects are required.
</details>

---

### Question 80
**Domain:** Domain 4 — Performance

A developer notices a query uses GROUP BY on a column with only 3 distinct values across 500M rows. The query is slow. What is the MOST likely bottleneck?

- [ ] A. Exploding JOIN producing Cartesian product
- [ ] B. Low-cardinality GROUP BY causing large intermediate data shuffles across warehouse nodes for each group
- [ ] C. The result cache is not applicable to GROUP BY queries
- [ ] D. The metadata cache cannot serve GROUP BY operations

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Low-cardinality GROUP BY causing large intermediate data shuffles across warehouse nodes for each group

**Explanation:**
While low-cardinality GROUP BY columns are generally efficient (fewer groups to aggregate), in distributed systems a **data skew** problem can occur where one group (e.g., a popular value) accumulates a disproportionate amount of data on one node, causing a processing bottleneck. Also, if large volumes of data must be shuffled to co-locate identical key values across nodes, network overhead can be significant. Reviewing the query profile for uneven distribution is the diagnostic step.
</details>

---

### Question 81
**Domain:** Domain 4 — Performance

Which performance issue does 'Exploding Joins' describe in the Snowflake Query Profile?

- [ ] A. A join that runs out of memory and spills to remote storage
- [ ] B. A many-to-many join where matching rows on both sides produce a disproportionately large output (Cartesian-like explosion of row count)
- [ ] C. A JOIN where the build table exceeds the warehouse memory limit
- [ ] D. A JOIN that takes too long due to missing clustering keys on join columns

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. A many-to-many join where matching rows on both sides produce a disproportionately large output (Cartesian-like explosion of row count)

**Explanation:**
An **exploding join** occurs when a JOIN produces far more output rows than input rows — typically a **many-to-many relationship** where many rows on the left match many rows on the right (e.g., unintentional Cartesian products, missing join predicates, or genuinely high-fan-out data). The Query Profile shows this as a node where output rows >> input rows. Fix: add filters, correct join conditions, or deduplicate before joining.
</details>

---

### Question 82
**Domain:** Domain 4 — Performance

A developer wants to use a window function to assign a unique sequential rank to rows within each department, ordered by salary descending, with no gaps in ranking even when salaries are tied. Which function is correct?

- [ ] A. ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)
- [ ] B. RANK() OVER (PARTITION BY dept ORDER BY salary DESC)
- [ ] C. DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC)
- [ ] D. NTILE(1) OVER (PARTITION BY dept ORDER BY salary DESC)

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC)

**Explanation:**
**DENSE_RANK()** assigns ranks without gaps — if two rows tie for rank 2, the next rank is 3 (not 4). **RANK()** skips ranks after ties (2, 2, 4). **ROW_NUMBER()** gives unique sequential numbers regardless of ties (no concept of tied rank). The requirement 'no gaps in ranking for ties' specifically describes DENSE_RANK.
</details>

---

### Question 83
**Domain:** Domain 4 — Performance

What does 'Queuing' in the Snowflake Query Profile indicate?

- [ ] A. The query is waiting for a result cache lookup to complete
- [ ] B. The query waited in a queue because the warehouse had no available concurrency slots
- [ ] C. Snowflake is queuing micro-partitions for sequential scanning
- [ ] D. The auto-suspend timer is queuing the warehouse for shutdown

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The query waited in a queue because the warehouse had no available concurrency slots

**Explanation:**
**Queuing** in the Query Profile indicates the query spent time waiting for the virtual warehouse to have available **concurrency slots**. Each warehouse size has a limited number of concurrent query slots. When all slots are occupied, incoming queries queue. Solutions: increase warehouse size, enable multi-cluster warehouses, or separate workloads across warehouses.
</details>

---

### Question 84
**Domain:** Domain 4 — Performance

A developer uses TRY_CAST(column AS INTEGER) instead of CAST(column AS INTEGER). What is the behavioral difference?

- [ ] A. TRY_CAST is faster because it skips type validation
- [ ] B. TRY_CAST returns NULL instead of raising an error when the conversion fails; CAST raises an error
- [ ] C. TRY_CAST only works on VARCHAR to DATE conversions
- [ ] D. There is no difference; TRY_CAST is an alias for CAST in Snowflake

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. TRY_CAST returns NULL instead of raising an error when the conversion fails; CAST raises an error

**Explanation:**
**TRY_CAST** attempts the type conversion and returns **NULL** if it fails (e.g., casting 'abc' to INTEGER), rather than raising a runtime error. This is useful for data quality checks when loading dirty data. **CAST** (or ::**) raises an error on conversion failure, stopping the query.
</details>

---

### Question 85
**Domain:** Domain 4 — Performance

Which Snowflake view or function tells a developer the current AVERAGE_OVERLAPS depth for a clustered table, indicating how well-clustered it is?

- [ ] A. ACCOUNT_USAGE.TABLE_STORAGE_METRICS
- [ ] B. SYSTEM$CLUSTERING_INFORMATION(table_name, '(col1, col2)')
- [ ] C. INFORMATION_SCHEMA.TABLE_CLUSTERING_DEPTH
- [ ] D. SHOW TABLES LIKE '%table_name%' WITH CLUSTERING_INFO

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. SYSTEM$CLUSTERING_INFORMATION(table_name, '(col1, col2)')

**Explanation:**
**SYSTEM$CLUSTERING_INFORMATION('table_name', '(col1, col2)')** returns clustering metrics including AVERAGE_OVERLAPS (lower is better; 0 = perfectly clustered), AVERAGE_DEPTH, TOTAL_PARTITION_COUNT, and TOTAL_CONSTANT_PARTITION_COUNT. This is the function used to assess whether reclustering is needed or if the chosen clustering key is effective.
</details>

---

### Question 86
**Domain:** Domain 5 — Collaboration

A provider creates a share called MY_SHARE and adds a table to it. Which statement is TRUE about the shared object?

- [ ] A. The consumer can INSERT into the shared table
- [ ] B. The shared table is read-only for the consumer; only the provider can modify the data
- [ ] C. The consumer receives a full copy of the table data at the time of sharing
- [ ] D. The shared table is deleted from the provider's account once shared

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The shared table is read-only for the consumer; only the provider can modify the data

**Explanation:**
Secure Data Sharing is **read-only for consumers**. The consumer can SELECT from shared tables but cannot INSERT, UPDATE, DELETE, or CREATE objects in the share. Data is NOT copied — the consumer accesses the provider's live data. The provider's data is unaffected by the share creation.
</details>

---

### Question 87
**Domain:** Domain 5 — Collaboration

An organization needs to clone a production database to create a QA environment. Which statement about zero-copy cloning is TRUE?

- [ ] A. Cloning copies all physical data; the process takes proportionally longer for larger databases
- [ ] B. A cloned database is independent — DML on the clone does not affect the source, and the clone immediately incurs full storage cost
- [ ] C. Cloning creates a metadata pointer to the source micro-partitions; DML on the clone creates new micro-partitions only for changed data, with the original shared micro-partitions billed once
- [ ] D. Cloned databases cannot have their own Time Travel retention settings

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Cloning creates a metadata pointer to the source micro-partitions; DML on the clone creates new micro-partitions only for changed data, with the original shared micro-partitions billed once

**Explanation:**
**Zero-copy cloning** creates a clone by copying only the **metadata pointers** to existing micro-partitions — instantaneous and free. The clone is fully independent: DML on the clone creates new micro-partitions for changed data only. The original (shared) micro-partitions are billed once. Clones can have their own Time Travel settings (DATA_RETENTION_TIME_IN_DAYS) independent of the source.
</details>

---

### Question 88
**Domain:** Domain 5 — Collaboration

A consumer wants to query a dataset from the Snowflake Marketplace and immediately use it in their existing Snowflake account. Which steps are needed?

- [ ] A. Purchase/request the listing → Snowflake delivers a copy of the dataset to the consumer's account → query from their database
- [ ] B. Request/get the listing → a database is automatically created in the consumer's account referencing the provider's data → query directly with no data copied
- [ ] C. The consumer must create an External Stage pointing to the provider's S3 bucket after getting the listing
- [ ] D. The consumer must request an API key from the provider to access the Marketplace data via REST API

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Request/get the listing → a database is automatically created in the consumer's account referencing the provider's data → query directly with no data copied

**Explanation:**
When a consumer gets a Marketplace listing (free or paid), Snowflake creates a **database in the consumer's account** that references the provider's data via Secure Data Sharing. **No data is copied.** The consumer queries the database as if it were local. It appears in their database list in Snowsight immediately after accepting the listing.
</details>

---

### Question 89
**Domain:** Domain 5 — Collaboration

An organization uses Time Travel to restore a table that was accidentally dropped 3 days ago. The account is on Enterprise edition with DATA_RETENTION_TIME_IN_DAYS = 7. Which command restores it?

- [ ] A. UNDROP TABLE my_table
- [ ] B. CREATE TABLE my_table CLONE my_table AT(OFFSET => -259200)
- [ ] C. ALTER TABLE my_table RESTORE FROM TRASH
- [ ] D. COPY INTO my_table FROM TABLE_TRAVEL('my_table', -3)

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. UNDROP TABLE my_table

**Explanation:**
**UNDROP TABLE** is the command to restore a dropped table from Time Travel. Snowflake retains dropped tables in a 'dropped' state within the Time Travel retention window. As long as the table was dropped within the past 7 days (within the retention period), UNDROP TABLE my_table restores it with all its data, privileges, and child objects. Cloning AT() restores data without undropping.
</details>

---

### Question 90
**Domain:** Domain 5 — Collaboration

Which type of Snowflake Marketplace listing allows the provider to offer a free, public dataset that any Snowflake user can access without approval?

- [ ] A. Private Listing with no approval required
- [ ] B. Public Listing with instant access
- [ ] C. Personalized Listing with auto-approval
- [ ] D. Community Listing with open access

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Public Listing with instant access

**Explanation:**
A **Public Listing** on the Snowflake Marketplace can be configured for **instant access** — any Snowflake account can browse, request, and immediately receive the data without provider approval. Free datasets (like Starschema COVID data or financial reference data) are commonly published this way. Private Listings require the provider to invite specific accounts.
</details>

---

### Question 91
**Domain:** Domain 1 — Architecture

A Snowflake UDF is defined with CALLED ON NULL INPUT (default). What does this mean?

- [ ] A. The UDF raises an error if any argument is NULL
- [ ] B. The UDF is called even when one or more arguments are NULL; the function must handle NULL internally
- [ ] C. The UDF automatically returns NULL without executing when any argument is NULL
- [ ] D. The UDF converts NULL inputs to empty strings before execution

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The UDF is called even when one or more arguments are NULL; the function must handle NULL internally

**Explanation:**
The default behavior **CALLED ON NULL INPUT** means the UDF is **invoked even when arguments are NULL** — the function body must handle NULL values explicitly. The alternative is **RETURNS NULL ON NULL INPUT** (also called STRICT), which short-circuits and returns NULL without executing the function when any argument is NULL.
</details>

---

### Question 92
**Domain:** Domain 1 — Architecture

A stored procedure in Snowflake uses EXECUTE AS CALLER. What is the implication?

- [ ] A. The stored procedure runs with the owner's (creator's) privileges, not the caller's
- [ ] B. The stored procedure runs with the calling user's current role's privileges, not the owner's
- [ ] C. EXECUTE AS CALLER is required for all stored procedures to run
- [ ] D. The stored procedure inherits privileges from both the owner and caller

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The stored procedure runs with the calling user's current role's privileges, not the owner's

**Explanation:**
**EXECUTE AS CALLER** means the stored procedure executes using the **caller's current role and privileges** at the time of the call. The alternative is **EXECUTE AS OWNER** (default), where the procedure runs with the owner's privileges regardless of who calls it. EXECUTE AS CALLER is useful when you want the procedure to enforce the caller's access rights, not escalate them.
</details>

---

### Question 93
**Domain:** Domain 1 — Architecture

What is the Snowflake parameter hierarchy, from lowest to highest precedence?

- [ ] A. System default → Account → User → Session
- [ ] B. System default → Account → Session → User
- [ ] C. Account → System default → User → Session
- [ ] D. User → Account → Session → System default

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. System default → Account → User → Session

**Explanation:**
Snowflake parameters follow a hierarchy where more specific settings override broader ones: **System defaults → Account level → User level → Session level**. A session-level parameter (SET QUERY_TAG = 'x') overrides the user-level setting, which overrides the account-level setting, which overrides the system default.
</details>

---

### Question 94
**Domain:** Domain 1 — Architecture

Which object type allows an administrator to define a reusable collection of SQL transformation logic that returns a table result, usable in the FROM clause of a query?

- [ ] A. Stored Procedure
- [ ] B. Scalar UDF
- [ ] C. Table UDF (UDTF)
- [ ] D. Secure View

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Table UDF (UDTF)

**Explanation:**
A **Table UDF (User-Defined Table Function / UDTF)** is a function that returns a set of rows (a table result). It is invoked in the FROM clause using TABLE() syntax: SELECT * FROM TABLE(my_udtf(arg1)). Scalar UDFs return a single value per row. Stored procedures return results differently (via RESULTSET) and are not used in FROM clauses.
</details>

---

### Question 95
**Domain:** Domain 1 — Architecture

The Snowflake Cortex AI SQL function SNOWFLAKE.CORTEX.COMPLETE() is called with a model parameter of 'mistral-7b'. What does this function do?

- [ ] A. Fine-tunes the Mistral-7B model on the user's Snowflake data
- [ ] B. Calls the Mistral-7B LLM hosted in Snowflake Cortex to generate a text completion for the given prompt, running entirely within Snowflake without data leaving
- [ ] C. Deploys a Mistral-7B container in Snowflake Container Services for inference
- [ ] D. Generates SQL queries from natural language using Mistral-7B

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Calls the Mistral-7B LLM hosted in Snowflake Cortex to generate a text completion for the given prompt, running entirely within Snowflake without data leaving

**Explanation:**
**SNOWFLAKE.CORTEX.COMPLETE(model, prompt)** calls a hosted LLM (e.g., mistral-7b, llama3, claude) via Snowflake Cortex to **generate a text completion** for the given prompt. The inference runs entirely within Snowflake's secure environment — no data is sent to external APIs. It is a SQL function usable in SELECT statements for tasks like summarization, classification, or extraction.
</details>

---

### Question 96
**Domain:** Domain 2 — Governance

A Snowflake user is created with MUST_CHANGE_PASSWORD = TRUE. What happens at the user's first login?

- [ ] A. The user is immediately prompted to set a new password before accessing Snowflake
- [ ] B. The user has 24 hours to change their password before the account is locked
- [ ] C. The MUST_CHANGE_PASSWORD flag only applies to SSO users
- [ ] D. Snowflake sends an email with a one-time password reset link

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** A. The user is immediately prompted to set a new password before accessing Snowflake

**Explanation:**
When **MUST_CHANGE_PASSWORD = TRUE** is set on a user, Snowflake forces the user to **change their password at first login** before they can access anything else. This is commonly used when an admin creates a user account and sets a temporary initial password.
</details>

---

### Question 97
**Domain:** Domain 2 — Governance

Which role is required to create a new database in Snowflake using default role assignments?

- [ ] A. ACCOUNTADMIN
- [ ] B. SECURITYADMIN
- [ ] C. SYSADMIN
- [ ] D. USERADMIN

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. SYSADMIN

**Explanation:**
The **SYSADMIN** role (or any role with CREATE DATABASE privilege on the account) can create databases. By convention and Snowflake best practice, SYSADMIN is the role for creating and owning warehouses, databases, and schemas. ACCOUNTADMIN can also do this, but best practice is to use SYSADMIN for object management to keep ACCOUNTADMIN for account-level administration.
</details>

---

### Question 98
**Domain:** Domain 2 — Governance

An organization wants to view which IP addresses have been used to log into Snowflake over the past 90 days. Which view provides this?

- [ ] A. ACCOUNT_USAGE.QUERY_HISTORY with CLIENT_NET_ADDRESS column
- [ ] B. ACCOUNT_USAGE.LOGIN_HISTORY with CLIENT_IP column
- [ ] C. ACCOUNT_USAGE.SESSIONS with SOURCE_IP column
- [ ] D. ACCOUNT_USAGE.ACCESS_HISTORY with LOGIN_IP column

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. ACCOUNT_USAGE.LOGIN_HISTORY with CLIENT_IP column

**Explanation:**
**ACCOUNT_USAGE.LOGIN_HISTORY** records every login attempt including the **CLIENT_IP** address, user name, authentication method, success/failure status, and timestamp. It retains data for up to 365 days. QUERY_HISTORY does not include login events. ACCESS_HISTORY tracks data access, not logins.
</details>

---

### Question 99
**Domain:** Domain 3 — Data Loading

A COPY INTO command completes and returns status 'LOAD_FAILED'. Which ACCOUNT_USAGE or system view can an engineer query to get the specific error details per file?

- [ ] A. ACCOUNT_USAGE.QUERY_HISTORY with ERROR_MESSAGE column
- [ ] B. VALIDATE(table_name, job_id => '<query_id>') table function
- [ ] C. INFORMATION_SCHEMA.LOAD_HISTORY
- [ ] D. ACCOUNT_USAGE.COPY_HISTORY with STATUS = 'LOAD_FAILED'

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. VALIDATE(table_name, job_id => '<query_id>') table function

**Explanation:**
The **VALIDATE()** table function returns the error details from a previous COPY INTO execution identified by its query ID. It shows file name, error line/character, error message, and row number for each rejected row. COPY_HISTORY (INFORMATION_SCHEMA or ACCOUNT_USAGE) shows file-level load status. VALIDATE() gives row-level error details.
</details>

---

### Question 100
**Domain:** Domain 3 — Data Loading

A Snowpipe is created with AUTO_INGEST = TRUE for an S3 bucket. What AWS service must be configured to trigger Snowpipe when new files arrive?

- [ ] A. AWS Lambda monitoring S3 events and calling the Snowpipe REST API
- [ ] B. Amazon SQS: an SQS queue configured to receive S3 event notifications, with the SQS ARN referenced in the Snowpipe or stage definition
- [ ] C. AWS CloudWatch Events rule triggering on S3 PutObject events
- [ ] D. AWS Glue crawler detecting new partitions and notifying Snowpipe

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Amazon SQS: an SQS queue configured to receive S3 event notifications, with the SQS ARN referenced in the Snowpipe or stage definition

**Explanation:**
For **Snowpipe AUTO_INGEST** on S3, you configure the S3 bucket to send event notifications to an **Amazon SQS queue**, and provide the SQS ARN in the Snowflake stage or pipe definition. When a new file lands in S3, S3 sends a notification to SQS, and Snowflake's Snowpipe service polls SQS and automatically loads the file. No Lambda or CloudWatch is needed.
</details>

---

### Question 101
**Domain:** Domain 3 — Data Loading

A Dynamic Table is defined using a SELECT from multiple source tables with a JOIN. The target lag is '30 minutes'. Which statement about Dynamic Table refresh is TRUE?

- [ ] A. Dynamic Tables only support single-source queries; JOINs are not supported
- [ ] B. Snowflake uses incremental refresh for simple projections; complex JOINs trigger a full refresh
- [ ] C. Dynamic Tables always perform full refresh regardless of query complexity
- [ ] D. Dynamic Tables use the same stream mechanism as user-defined streams for change tracking

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowflake uses incremental refresh for simple projections; complex JOINs trigger a full refresh

**Explanation:**
Snowflake's Dynamic Table scheduler attempts **incremental refresh** (processing only changed rows) for simpler query patterns. For complex queries involving JOINs, aggregations, or certain SQL constructs that Snowflake cannot incrementally process, it falls back to a **full refresh**. The Snowflake documentation specifies which operations support incremental vs. full refresh.
</details>

---

### Question 102
**Domain:** Domain 4 — Performance

A developer sees 'Local Disk I/O' as a significant component in the Query Profile. What does this indicate?

- [ ] A. Data was read from the warehouse's local SSD cache (good — avoids remote storage reads)
- [ ] B. Data spilled from memory to the warehouse node's local SSD due to insufficient memory (bad — slower than in-memory processing)
- [ ] C. The query wrote results to a local file on the warehouse node
- [ ] D. The metadata cache was accessed via local disk storage in the Cloud Services layer

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Data spilled from memory to the warehouse node's local SSD due to insufficient memory (bad — slower than in-memory processing)

**Explanation:**
In the Query Profile, high **Local Disk I/O** indicates that intermediate data (e.g., sort buffers, hash join tables) exceeded available memory and **spilled to the warehouse node's local SSD**. This is faster than remote storage spill but still significantly slower than in-memory processing. The fix is typically increasing warehouse size or optimizing the query to reduce intermediate data volume.
</details>

---

### Question 103
**Domain:** Domain 4 — Performance

Which SQL function in Snowflake is used to parse a JSON string stored in a VARCHAR column into a queryable VARIANT?

- [ ] A. JSON_PARSE()
- [ ] B. PARSE_JSON()
- [ ] C. TO_VARIANT()
- [ ] D. CAST(col AS VARIANT)

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. PARSE_JSON()

**Explanation:**
**PARSE_JSON()** converts a JSON string (VARCHAR) into a Snowflake VARIANT object, enabling field extraction with the colon operator (v:field). TRY_PARSE_JSON() is the non-error variant (returns NULL on invalid JSON). TO_VARIANT() converts native SQL types to VARIANT, not JSON strings.
</details>

---

### Question 104
**Domain:** Domain 4 — Performance

A MERGE statement is run to upsert data from a staging table into a target table. The staging table has duplicate rows for the same key. What error occurs?

- [ ] A. The MERGE silently picks the last duplicate row for each key
- [ ] B. Snowflake raises a 'non-deterministic MERGE' error because multiple source rows match a single target row
- [ ] C. The MERGE loads all duplicate rows, creating multiple rows per key in the target
- [ ] D. The MERGE automatically deduplicates the source before matching

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowflake raises a 'non-deterministic MERGE' error because multiple source rows match a single target row

**Explanation:**
Snowflake's MERGE raises a **non-deterministic MERGE error** (SQL error 100132) when a single target row matches multiple source rows, because the result would be non-deterministic. The fix is to deduplicate the source table before the MERGE (e.g., using a CTE with ROW_NUMBER() to pick one row per key), or use a QUALIFY clause in the staging query.
</details>

---

### Question 105
**Domain:** Domain 4 — Performance

A query uses ILIKE to search a VARCHAR column for case-insensitive pattern matching. The column has 1 billion rows. Which optimization strategy is MOST effective?

- [ ] A. Add a clustering key on the VARCHAR column
- [ ] B. Enable the Search Optimization Service on the table for ILIKE/LIKE predicates
- [ ] C. Create a materialized view with LOWER(column) for case-normalized search
- [ ] D. Increase the warehouse size to XL to scan faster

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Enable the Search Optimization Service on the table for ILIKE/LIKE predicates

**Explanation:**
The **Search Optimization Service (SOS)** supports acceleration for LIKE and ILIKE predicates (wildcard pattern searches) on VARCHAR columns, in addition to equality lookups. SOS builds a persistent search access path that enables sub-second results for pattern searches that would otherwise require full table scans. This is the purpose-built optimization for this workload.
</details>

---

### Question 106
**Domain:** Domain 5 — Collaboration

A Snowflake database is cloned using 'CREATE DATABASE dev_db CLONE prod_db'. Which objects are NOT cloned?

- [ ] A. Tables and their data
- [ ] B. Views and stored procedures
- [ ] C. External stages and their referenced cloud storage credentials
- [ ] D. Named file formats and sequences

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. External stages and their referenced cloud storage credentials

**Explanation:**
When cloning a database, most schema objects (tables, views, stored procedures, file formats, sequences, streams, tasks) are cloned. However, **external stages** are cloned but the **cloud storage credentials or storage integration references** may need to be updated in the clone. Additionally, **shares** (inbound/outbound), **resource monitors**, and **users/roles** are account-level objects and are not cloned as part of a database clone.
</details>

---

### Question 107
**Domain:** Domain 5 — Collaboration

A Time Travel query uses AT(STATEMENT => '<query_id>'). What does this recover?

- [ ] A. The state of the table AFTER the specified query completed
- [ ] B. The state of the table BEFORE the specified query ran (as if the query never happened)
- [ ] C. The exact rows returned by the specified query, regardless of current table state
- [ ] D. The query plan used by the specified query ID for re-execution

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The state of the table BEFORE the specified query ran (as if the query never happened)

**Explanation:**
**AT(STATEMENT => 'query_id')** returns the table state as it existed **immediately before** the specified query ran. This is commonly used to recover from accidental DML: you identify the query ID of the bad UPDATE/DELETE, then use AT(STATEMENT => ...) to see the table state just before that operation, and recreate or insert the affected rows.
</details>

---

### Question 108
**Domain:** Domain 5 — Collaboration

A Reader Account created by a provider allows the consumer to run queries. Who manages the Reader Account users and warehouses?

- [ ] A. Snowflake manages all Reader Account users and warehouses automatically
- [ ] B. The consumer manages their own users and warehouses after being given ACCOUNTADMIN access to the Reader Account
- [ ] C. The provider creates and manages users and warehouses in the Reader Account; the consumer cannot modify account settings
- [ ] D. Reader Accounts share the provider's warehouses; no separate warehouses exist in the Reader Account

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. The provider creates and manages users and warehouses in the Reader Account; the consumer cannot modify account settings

**Explanation:**
The **provider creates and manages** all users, warehouses, and access controls in the Reader Account. The consumer accesses the Reader Account using credentials provisioned by the provider. The consumer does not have administrative access to the Reader Account — they can only run queries using the resources the provider sets up. This is different from a full consumer account where the consumer has ACCOUNTADMIN.
</details>

---

### Question 109
**Domain:** Domain 1 — Architecture

Which Snowflake edition introduced support for customer-managed Tri-Secret Secure encryption?

- [ ] A. Standard
- [ ] B. Enterprise
- [ ] C. Business Critical
- [ ] D. Virtual Private Snowflake (VPS)

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** C. Business Critical

**Explanation:**
**Tri-Secret Secure** is exclusive to **Business Critical edition**. It requires both Snowflake's managed key AND the customer's key (in AWS KMS, Azure Key Vault, or GCP KMS) to decrypt data. VPS also supports it as VPS is above Business Critical. Standard and Enterprise do not offer this feature.
</details>

---

### Question 110
**Domain:** Domain 2 — Governance

A column is tagged with the 'SENSITIVE_PII' object tag. A masking policy is attached to that tag. The column also has a separate masking policy directly assigned to it. Which policy takes effect?

- [ ] A. The tag-based masking policy always takes precedence
- [ ] B. The directly assigned column-level masking policy takes precedence over the tag-based policy
- [ ] C. Both policies are applied sequentially, with results chained
- [ ] D. Snowflake raises an error when both a direct and tag-based masking policy exist for the same column

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. The directly assigned column-level masking policy takes precedence over the tag-based policy

**Explanation:**
When a column has both a **directly assigned masking policy** and a **tag-based masking policy**, the **directly assigned column-level policy takes precedence**. Only one masking policy can be active on a column at a time. The tag-based policy is used as a fallback for columns that don't have a direct assignment.
</details>

---

### Question 111
**Domain:** Domain 3 — Data Loading

What is the purpose of the COPY_OPTIONS parameter PURGE = TRUE in a COPY INTO command loading from a named stage?

- [ ] A. Purges the load history metadata after loading, allowing the same files to be re-loaded
- [ ] B. Automatically deletes the staged source files from the stage after successful loading
- [ ] C. Removes failed-row error files from the stage after load completion
- [ ] D. Truncates the target table before loading new data

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Automatically deletes the staged source files from the stage after successful loading

**Explanation:**
**PURGE = TRUE** in COPY INTO automatically **deletes the source files from the stage** after they are successfully loaded into the target table. For internal stages this frees storage; for external stages it deletes the files from the cloud storage. Files that fail to load are NOT deleted. PURGE does not affect load history metadata.
</details>

---

### Question 112
**Domain:** Domain 4 — Performance

A developer wants to avoid recalculating an expensive subquery multiple times in a complex SQL statement. Which SQL feature in Snowflake allows the subquery to be defined once and referenced multiple times?

- [ ] A. Inline view (subquery in FROM clause)
- [ ] B. Common Table Expression (CTE) defined with WITH clause
- [ ] C. Temporary table created in the same session
- [ ] D. Materialized view with REFRESH ON COMMIT

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Common Table Expression (CTE) defined with WITH clause

**Explanation:**
A **Common Table Expression (CTE)** defined with the WITH clause allows a subquery to be named and referenced multiple times within the same SQL statement. CTEs improve readability and can avoid redundant computation, though Snowflake's optimizer may or may not materialize the CTE depending on the execution plan. For truly expensive reusable results across queries, a temporary table or materialized view is more appropriate.
</details>

---

### Question 113
**Domain:** Domain 5 — Collaboration

A Snowflake share can include which of the following object types? (Select the MOST comprehensive correct answer)

- [ ] A. Tables, views, and secure views only
- [ ] B. Tables, external tables, views, materialized views, secure views, and UDFs
- [ ] C. Tables only — views are not shareable via Secure Data Sharing
- [ ] D. Tables, views, and stored procedures

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Tables, external tables, views, materialized views, secure views, and UDFs

**Explanation:**
Snowflake Secure Data Sharing supports sharing of **tables, external tables, views, secure views, materialized views, and UDFs**. Secure views are the recommended way to share because they hide the view definition from the consumer. Stored procedures are not shareable via shares (they can be included in Native Apps instead).
</details>

---

### Question 114
**Domain:** Domain 1 — Architecture

A Snowflake account has AUTO_SUSPEND set to 60 seconds on a warehouse. The warehouse is currently running a query that takes 5 minutes. When does the warehouse suspend?

- [ ] A. After 60 seconds, regardless of running queries
- [ ] B. After the running query completes, then 60 seconds of inactivity triggers suspension
- [ ] C. The warehouse suspends immediately when the query finishes
- [ ] D. The warehouse never suspends while AUTO_SUSPEND is enabled during active queries

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. After the running query completes, then 60 seconds of inactivity triggers suspension

**Explanation:**
**AUTO_SUSPEND** counts inactivity — the timer starts only after **all queries finish**. If a query runs for 5 minutes, the warehouse runs the full 5 minutes. After the query completes, the 60-second inactivity timer begins. Only after 60 seconds of no new queries does the warehouse suspend. Queries in progress are never interrupted by AUTO_SUSPEND.
</details>

---

### Question 115
**Domain:** Domain 2 — Governance

Which Snowflake feature allows an account administrator to set a credit limit on a virtual warehouse that, when reached, automatically sends a notification and optionally suspends the warehouse?

- [ ] A. Budget alert on the ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY view
- [ ] B. Resource Monitor with a credit quota and SUSPEND or NOTIFY action
- [ ] C. Warehouse cost policy created with CREATE COST POLICY
- [ ] D. Account-level spending limit in the Snowflake billing console

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Resource Monitor with a credit quota and SUSPEND or NOTIFY action

**Explanation:**
A **Resource Monitor** is the Snowflake object for setting **credit quotas on warehouses**. You define a credit quota (e.g., 100 credits per month), attach the monitor to one or more warehouses, and configure actions at percentage thresholds: NOTIFY (email alert), SUSPEND (after current queries finish), or SUSPEND_IMMEDIATE (kill running queries). No other Snowflake feature natively handles this.
</details>

---

### Question 116
**Domain:** Domain 3 — Data Loading

When Snowpipe loads data, what type of compute does it use, and who is responsible for managing it?

- [ ] A. Customer-provisioned virtual warehouses — the customer manages sizing and suspension
- [ ] B. Snowflake-managed serverless compute — the customer does not provision or manage it; Snowflake bills per-file-ingested
- [ ] C. Shared community compute provided by Snowflake at no cost to the customer
- [ ] D. External compute in the customer's cloud account via a service integration

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowflake-managed serverless compute — the customer does not provision or manage it; Snowflake bills per-file-ingested

**Explanation:**
**Snowpipe** uses **Snowflake-managed serverless compute**. Customers do not provision, size, or manage warehouses for Snowpipe. Snowflake automatically allocates compute for each ingestion job. Billing is based on compute used per file loaded (credits consumed per ingested data volume), not on warehouse-hours. This distinguishes Snowpipe from COPY INTO, which uses customer-managed virtual warehouses.
</details>

---

### Question 117
**Domain:** Domain 4 — Performance

A developer runs the same SELECT query twice in quick succession. The second run returns results almost instantly. Which cache is responsible, and what conditions must be true for this to work?

- [ ] A. Warehouse cache — the warehouse node cached the raw column data from the first scan
- [ ] B. Result cache — the query text must be identical, underlying data must not have changed, and the result must be less than 24 hours old
- [ ] C. Metadata cache — Snowflake cached the query execution plan
- [ ] D. Remote storage cache — S3/Azure caches recent reads at the CDN layer

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Result cache — the query text must be identical, underlying data must not have changed, and the result must be less than 24 hours old

**Explanation:**
The **result cache (query result cache)** serves the second run instantly. The conditions are: (1) the **query text must be identical** (same SQL), (2) the **underlying table data must not have changed** since the first run, and (3) the cached result is **less than 24 hours old**. If all conditions are met, no warehouse is started — the result is returned directly from Cloud Services. The cache is invalidated by any DML on the queried tables.
</details>

---

### Question 118
**Domain:** Domain 1 — Architecture

A developer wants to monitor query execution in real time as it runs. Which tool or view provides a visual breakdown of query execution stages while the query is still running?

- [ ] A. ACCOUNT_USAGE.QUERY_HISTORY — real-time query monitoring view
- [ ] B. Snowsight Query Profile — accessible from the query history in Snowsight, showing execution stages and operator statistics
- [ ] C. INFORMATION_SCHEMA.QUERY_HISTORY with EXECUTION_STATUS = 'RUNNING'
- [ ] D. SYSTEM$QUERY_PLAN(query_id) function returning the execution plan

<details>
<summary><b>Click here to view Answer & Explanation</b></summary>

**Correct Answer:** B. Snowsight Query Profile — accessible from the query history in Snowsight, showing execution stages and operator statistics

**Explanation:**
The **Snowsight Query Profile** (also accessible via the Classic UI) provides a visual execution plan with nodes representing operators, showing bytes processed, rows, and time per node. It is accessible from the query history in Snowsight and updates while the query runs. ACCOUNT_USAGE.QUERY_HISTORY and INFORMATION_SCHEMA.QUERY_HISTORY show completed queries and metadata, not a real-time visual profile.
</details>

---


# Snowflake SnowPro Core — Complete Exam Reference
*Privileges & Roles · Edition Requirements · Masking/Row Access/Network Policies · ACCOUNT_USAGE vs INFORMATION_SCHEMA · Supporting Core Topics*

> Built for the exact question patterns you described: "minimum privilege/role to create X," "minimum edition for feature Y," "which command do I use (SET/APPLY/UNSET/DESCRIBE)," and "which view answers this scenario."

---

## PART 1 — Roles & Privileges

### 1.1 System-Defined Roles (top to bottom of default hierarchy)

| Role | Purpose | Key privileges | Notes |
|---|---|---|---|
| **ORGADMIN** (being phased out → **GLOBALORGADMIN**) | Manage the *organization* (multiple accounts) | Create accounts, view org-level usage | Not part of the account role hierarchy |
| **ACCOUNTADMIN** | Top-level account role | Encapsulates SYSADMIN + SECURITYADMIN | Not a superuser — only sees objects it (or a role below it) has privileges on. Should be tightly controlled, not used for daily work, not granted to other roles, should own no objects |
| **SECURITYADMIN** | Security/user/role management | Global **MANAGE GRANTS** (grant/revoke any privilege on any object); inherits USERADMIN | USERADMIN is a child of SECURITYADMIN by default |
| **USERADMIN** | Dedicated user & role management | **CREATE USER**, **CREATE ROLE** (global) | Can only manage users/roles it owns |
| **SYSADMIN** | Data/compute object management | Privileges to create warehouses, databases, and all database objects | Custom roles should be granted up to SYSADMIN so sysadmins can manage them |
| **PUBLIC** | Pseudo-role | Automatically granted to every user/role | Anything granted to PUBLIC is visible to *everyone* — avoid granting sensitive privileges here |

**Best-practice hierarchy:** custom roles → granted up to **SYSADMIN** (for object management) or **USERADMIN/SECURITYADMIN** (for role/user management). ACCOUNTADMIN sits at the very top and inherits everything.

### 1.2 Privilege Categories
- **Global / Account privileges** — e.g., `CREATE DATABASE`, `CREATE WAREHOUSE`, `CREATE ROLE`, `CREATE USER`, `CREATE SHARE`, `CREATE NETWORK POLICY`, `CREATE RESOURCE MONITOR`, `CREATE INTEGRATION`, `MANAGE GRANTS`, `APPLY MASKING POLICY`, `APPLY ROW ACCESS POLICY`, `APPLY TAG`, `IMPORT SHARE`, `EXECUTE TASK`, `EXECUTE MANAGED TASK`
- **Database privileges** — `USAGE`, `CREATE SCHEMA`, `MODIFY`, `MONITOR`, `IMPORTED PRIVILEGES` (for shared DBs)
- **Schema privileges** — `USAGE`, `CREATE TABLE/VIEW/STAGE/FILE FORMAT/SEQUENCE/PIPE/STREAM/TASK/FUNCTION/PROCEDURE/MASKING POLICY/ROW ACCESS POLICY/TAG`, `MODIFY`, `MONITOR`
- **Schema-object privileges** — `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, `REFERENCES`, `USAGE` (functions/UDFs/stages), `READ`/`WRITE` (stages), `OWNERSHIP`, `APPLY` (policies)
- **Warehouse privileges** — `USAGE` (run queries), `OPERATE` (start/stop/suspend/resume/abort), `MONITOR` (view usage), `MODIFY` (resize/config)

**Golden rule:** "Operating on an object in a schema requires at least one privilege on the parent database AND at least one privilege on the parent schema" — i.e., you always need `USAGE` on the DB and schema chain, in addition to the object-level privilege.

**To query data at minimum, a role needs:**
`USAGE` on database → `USAGE` on schema → `SELECT` on table/view → `USAGE` on a warehouse.

### 1.3 Minimum Privilege / Default-Owning-Role Cheat Sheet

| To create... | Minimum privilege required | Default system role that has it |
|---|---|---|
| Database | `CREATE DATABASE` on account | SYSADMIN |
| Schema | `CREATE SCHEMA` on database | SYSADMIN (or DB owner) |
| Warehouse | `CREATE WAREHOUSE` on account | SYSADMIN |
| Role | `CREATE ROLE` on account | USERADMIN |
| User | `CREATE USER` on account | USERADMIN (SECURITYADMIN also, since it inherits USERADMIN) |
| Share | `CREATE SHARE` on account | ACCOUNTADMIN (default) |
| Network Policy | `CREATE NETWORK POLICY` on account | SECURITYADMIN (default) |
| Resource Monitor | `CREATE RESOURCE MONITOR` on account | **ACCOUNTADMIN only**, by default (grantable) |
| Masking Policy | `CREATE MASKING POLICY` on schema | Schema owner role automatically has it |
| Row Access Policy | `CREATE ROW ACCESS POLICY` on schema | Schema owner role automatically has it |
| Tag | `CREATE TAG` on schema | Schema owner role |
| Table / View / Stage / File Format / Sequence / Pipe / Stream / Task / Function / Procedure | `CREATE <object>` on schema | Schema owner role |
| Integration (API/Storage/Notification/Security) | `CREATE INTEGRATION` on account | ACCOUNTADMIN (default) |
| Failover Group / Replication Group | `CREATE FAILOVER GROUP` / `CREATE REPLICATION GROUP` on account | ACCOUNTADMIN |
| Managed-access schema | `CREATE SCHEMA ... WITH MANAGED ACCESS` | same as schema, but object owners lose grant rights — only schema owner or `MANAGE GRANTS` role can grant on objects inside |

### 1.4 Special "APPLY" and "MANAGE" Global Privileges
These global (account-level) privileges are commonly tested because they're easy to confuse with `CREATE`:

| Privilege | Lets a role... |
|---|---|
| `APPLY MASKING POLICY` | SET/UNSET a masking policy on **any** column (account-wide) |
| `APPLY ROW ACCESS POLICY` | ADD/DROP a row access policy on **any** table/view (account-wide) |
| `APPLY TAG` | Assign a tag to **any** object (account-wide) |
| `APPLY SESSION POLICY` / `APPLY AUTHENTICATION POLICY` / `APPLY PASSWORD POLICY` | Attach those policy types account-wide |
| `MANAGE GRANTS` | Grant/revoke **any** privilege on **any** object — only SECURITYADMIN/ACCOUNTADMIN have it by default |
| `IMPORT SHARE` | View inbound shares; combined with `CREATE DATABASE`, create a DB from an inbound share |
| `EXECUTE TASK` / `EXECUTE MANAGED TASK` | Run tasks on a user warehouse / serverless compute |

> **Decentralized alternative:** instead of the account-wide `APPLY MASKING POLICY`/`APPLY ROW ACCESS POLICY` privilege, you can grant the narrower **`APPLY`** privilege on a *specific* policy object to a role that also owns (has `OWNERSHIP` on) the table/view — this lets object owners self-serve without giving them account-wide apply rights.

---

## PART 2 — Editions & Minimum-Edition Feature Requirements

### 2.1 The Four Editions (each builds on the previous)
1. **Standard** — full core feature set, 1-day Time Travel, entry-level security
2. **Enterprise** — Standard + performance/scale/governance features (see table below)
3. **Business Critical** (formerly "Enterprise for Sensitive Data") — Enterprise + top-tier data protection (HIPAA/PCI, Tri-Secret Secure, private connectivity, failover/failback)
4. **Virtual Private Snowflake (VPS)** — Business Critical + fully isolated, dedicated infrastructure (no shared hardware with other accounts)

Find your edition: `SELECT edition FROM SNOWFLAKE.ORGANIZATION_USAGE.ACCOUNTS WHERE account_name = CURRENT_ACCOUNT();`

### 2.2 Feature → Minimum Required Edition (exam-critical list)

| Feature | Minimum Edition |
|---|---|
| Time Travel (1 day, default) | Standard |
| Fail-safe (7 days, non-configurable) | Standard |
| Network policies, MFA, SSO/federated auth, OAuth | Standard |
| Object tags (basic) | Standard *(some tagging features need Enterprise+)* |
| Object-level access control | Standard |
| **Extended Time Travel (up to 90 days)** | **Enterprise** |
| **Multi-cluster warehouses** | **Enterprise** |
| **Materialized views** | **Enterprise** |
| **Search Optimization Service** | **Enterprise** |
| **Query Acceleration Service** | **Enterprise** |
| **Column-level Security (masking policies)** | **Enterprise** |
| **Row-level Security (row access policies)** | **Enterprise** |
| Aggregation policies | Enterprise |
| Projection policies | Enterprise |
| Differential privacy | Enterprise |
| Data classification | Enterprise |
| **Access History (`ACCOUNT_USAGE.ACCESS_HISTORY`)** | **Enterprise** |
| Periodic rekeying of encrypted data | Enterprise |
| Event tables associated with an object | Enterprise |
| Data Quality / data metric functions | Enterprise |
| Synthetic data generation | Enterprise |
| 24-hr early access to weekly releases | Enterprise |
| Create/manage your own Data Clean Rooms | Enterprise |
| **Tri-Secret Secure (customer-managed keys)** | **Business Critical** |
| **Private connectivity (AWS PrivateLink / Azure Private Link / GCP Private Service Connect)** | **Business Critical** |
| **Failover & Failback** | **Business Critical** |
| Redirecting client connections | Business Critical |
| PHI data / HIPAA & HITRUST CSF support | Business Critical |
| PCI DSS support | Business Critical |
| FedRAMP / ITAR (US Gov regions) | Business Critical |
| Dedicated metadata store & compute pool, full hardware isolation | VPS only |

> Things that are available on **every edition** and are common exam traps: Standard SQL, UDFs/stored procs, Snowpipe, Snowpipe Streaming, streams, tasks, external tables, hybrid tables, dynamic tables, clustering, Snowpark, Streamlit in Snowflake, Cortex AI functions, database/share replication (but **not** failover/failback — that's Business Critical+), Snowflake Marketplace (not available on VPS).

---

## PART 3 — Masking Policies (Column-level Security)

### 3.1 Core Commands
```sql
-- CREATE (needs CREATE MASKING POLICY on schema)
CREATE OR REPLACE MASKING POLICY email_mask AS (val STRING) RETURNS STRING ->
  CASE WHEN CURRENT_ROLE() IN ('ANALYST') THEN val ELSE '*********' END;

-- APPLY (SET) a masking policy to a column — needs APPLY MASKING POLICY (account) OR
-- (APPLY on that specific policy + OWNERSHIP on the table)
ALTER TABLE employee MODIFY COLUMN email SET MASKING POLICY email_mask;

-- UNSET (remove) the policy from a column
ALTER TABLE employee MODIFY COLUMN email UNSET MASKING POLICY;

-- ALTER the policy body
ALTER MASKING POLICY email_mask SET BODY -> CASE WHEN CURRENT_ROLE() IN ('ANALYST') THEN val ELSE SHA2(val, 512) END;

-- RENAME / COMMENT
ALTER MASKING POLICY email_mask RENAME TO email_mask_v2;
ALTER MASKING POLICY email_mask SET COMMENT = 'masks PII emails';

-- INSPECT
SHOW MASKING POLICIES;
DESCRIBE MASKING POLICY email_mask;
SELECT * FROM TABLE(INFORMATION_SCHEMA.POLICY_REFERENCES(POLICY_NAME => 'email_mask'));

-- DROP (must UNSET from every column/tag first)
DROP MASKING POLICY email_mask;
```

### 3.2 Key Facts (exam favorites)
- Masking policies are **schema-level objects** — DB + schema must exist first.
- **Minimum edition: Enterprise.**
- Object **owners do NOT automatically get to unset masking policies or see unmasked data** — this is deliberate segregation of duties (policy admin ≠ object owner).
- **Only one masking policy per column** at a time; a column can have both a masking policy *and* be referenced in a row access policy (row access policy evaluates first).
- **Tag-based masking policies**: attach a masking policy to a **tag** instead of a column directly — the policy then auto-applies anywhere that tag is used. Setting a tag-based policy needs both global `APPLY MASKING POLICY` and `APPLY TAG`, OR the tag owner already having a masking policy set on it.
- Cannot drop a masking policy while it's still attached anywhere (use `POLICY_REFERENCES` table function to check first).
- `CREATE OR REPLACE MASKING POLICY` **cannot change the argument/signature** once attached to a column — you must `DROP` and recreate instead.

### 3.3 Typical Setup (centralized governance model)
```sql
USE ROLE SECURITYADMIN;
GRANT CREATE MASKING POLICY ON SCHEMA mydb.myschema TO ROLE masking_admin;
GRANT APPLY MASKING POLICY ON ACCOUNT TO ROLE masking_admin;
GRANT ROLE masking_admin TO USER security_officer;
```

---

## PART 4 — Row Access Policies (Row-level Security)

### 4.1 Core Commands
```sql
-- CREATE (needs CREATE ROW ACCESS POLICY on schema)
CREATE OR REPLACE ROW ACCESS POLICY sales_policy AS (region VARCHAR) RETURNS BOOLEAN ->
  'sales_exec' = CURRENT_ROLE()
  OR EXISTS (SELECT 1 FROM sales_manager_regions WHERE sales_manager = CURRENT_ROLE() AND region = region);

-- ADD (apply) to a table/view — needs APPLY ROW ACCESS POLICY (account) OR
-- (APPLY on that specific policy + OWNERSHIP of table)
ALTER TABLE sales ADD ROW ACCESS POLICY sales_policy ON (region);

-- DROP from a table/view
ALTER TABLE sales DROP ROW ACCESS POLICY sales_policy;
ALTER TABLE sales DROP ALL ROW ACCESS POLICIES;

-- INSPECT
SHOW ROW ACCESS POLICIES;
DESCRIBE ROW ACCESS POLICY sales_policy;
SELECT * FROM TABLE(INFORMATION_SCHEMA.POLICY_REFERENCES(POLICY_NAME => 'sales_policy'));

-- DROP the policy object (must detach from all tables/views first)
DROP ROW ACCESS POLICY sales_policy;
```

### 4.2 Key Facts
- **Minimum edition: Enterprise.**
- Three privileges to know: **CREATE** (make a new policy), **APPLY** (add/drop the policy on tables — account-wide or per-policy), **OWNERSHIP** (full control, incl. replace/drop).
- Row access policy is **evaluated before** any masking policy on the same object.
- Cannot change the policy signature after attaching — drop & recreate.
- Cannot drop a row access policy while attached; a protected column also **cannot be dropped** from its table while the policy is attached.
- **No `UNDROP` support** for row access policy objects.
- Data providers/consumers **can** use row access policies in shared data; a provider **cannot** create a row access policy inside a reader account.

---

## PART 5 — Network Policies

### 5.1 Core Commands
```sql
-- CREATE (needs CREATE NETWORK POLICY on account — default: SECURITYADMIN)
CREATE NETWORK POLICY corp_policy
  ALLOWED_IP_LIST = ('192.168.1.0/24')
  BLOCKED_IP_LIST = ('192.168.1.99');

-- Newer / recommended approach — use network rules instead of raw IP lists:
CREATE NETWORK POLICY corp_policy
  ALLOWED_NETWORK_RULE_LIST = ('allow_rule')
  BLOCKED_NETWORK_RULE_LIST = ('block_rule');

-- ALTER
ALTER NETWORK POLICY corp_policy SET ALLOWED_IP_LIST = ('192.168.1.0/24','10.0.0.0/8');
ALTER NETWORK POLICY corp_policy SET BLOCKED_NETWORK_RULE_LIST = ('block_access_rule');

-- >>> VIEW THE ALLOWED/BLOCKED IP LIST <<<
DESCRIBE NETWORK POLICY corp_policy;      -- shows ALLOWED_IP_LIST / BLOCKED_IP_LIST (or rule lists) with actual values
SHOW NETWORK POLICIES;                    -- shows policy names + COUNTS (entries_in_allowed_ip_list, entries_in_blocked_ip_list) — not the values themselves

-- Which policy is currently active?
SHOW PARAMETERS LIKE 'network_policy' IN ACCOUNT;
SHOW PARAMETERS LIKE 'network_policy' IN USER my_user;

-- ACTIVATE (apply) a policy
ALTER ACCOUNT SET NETWORK_POLICY = corp_policy;      -- can be run by SECURITYADMIN (or higher) — the one exception to normal ALTER ACCOUNT rules
ALTER USER my_user SET NETWORK_POLICY = corp_policy;

-- REMOVE activation
ALTER ACCOUNT UNSET NETWORK_POLICY;

DROP NETWORK POLICY corp_policy;
```

### 5.2 Key Facts (exam favorites)
- **`DESCRIBE NETWORK POLICY <name>`** is the command that returns the actual **allowed/blocked IP addresses (or rule lists)** — `SHOW NETWORK POLICIES` only returns **counts**, not the values.
- **Precedence: BLOCKED list always wins** if an IP appears in both `ALLOWED_IP_LIST` and `BLOCKED_IP_LIST` (same for network-rule lists).
- `ALLOWED_IP_LIST`/`BLOCKED_IP_LIST` support **IPv4 only**; for IPv6 you must use **network rules** (`TYPE = IPV6`) added to `ALLOWED_NETWORK_RULE_LIST`/`BLOCKED_NETWORK_RULE_LIST`.
- Network policies can be applied at **account level**, **user level**, or on a **security integration**. Only **one account-level** network policy can be active at a time.
- Snowflake will **block you from removing your own IP** from the allowed list (prevents accidental lockout).
- **Network rules** are schema-level objects (newer method); Snowflake recommends network rules over the legacy IP-list parameters, and advises against mixing both methods in the same policy.
- If truly locked out: `MINS_TO_BYPASS_NETWORK_POLICY` user property (viewable via `DESCRIBE USER`) — but it can **only be set by Snowflake Support**, not by the customer.
- Minimum edition: **Standard** (network policies are available on all editions); **private connectivity** (PrivateLink etc.) requires **Business Critical**.
- To see blocked login attempts (e.g., by a network policy), query `SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY` (`IS_SUCCESS = 'NO'`).

---

## PART 6 — ACCOUNT_USAGE vs INFORMATION_SCHEMA

### 6.1 Head-to-Head Comparison

| Aspect | ACCOUNT_USAGE (`SNOWFLAKE.ACCOUNT_USAGE`) | INFORMATION_SCHEMA (per-database) |
|---|---|---|
| Scope | Whole **account** (all databases) | The **single database** it lives in (plus some account-wide views/table functions) |
| Latency | **45 min – 3 hrs** typical (some views up to 2 days under low-activity conditions) | **None — real-time** |
| Retention | **365 days (1 year)** for historical views | Short — roughly **7 days to 6 months** depending on the view/table function (commonly cited as 7–14 days for history table functions) |
| Dropped objects | **Included** (tracks history of deleted objects) | **Not included** — only current, active objects |
| Access | Default: **ACCOUNTADMIN only**; grant `IMPORTED PRIVILEGES` on the `SNOWFLAKE` DB (or a `SNOWFLAKE` database role) to extend to other roles | Automatically visible per your existing object privileges |
| Consistency | Point-in-time snapshot with latency | May be inconsistent with concurrent DDL during long-running queries |
| Best for | Long-term trend analysis, auditing, compliance, cost/credit analysis, tracking dropped objects | Real-time day-to-day object management, alerting, current-state metadata |

> **Rule of thumb for exam questions:** "need > 1 year / includes dropped objects / historical trend" → **ACCOUNT_USAGE**. "need it right now / no latency / current objects only" → **INFORMATION_SCHEMA**.

### 6.2 Scenario → View Cheat Sheet (mostly ACCOUNT_USAGE unless noted)

| Scenario | View / Function to use |
|---|---|
| Who logged in / failed logins / network-policy blocks | `ACCOUNT_USAGE.LOGIN_HISTORY` (or `INFORMATION_SCHEMA.LOGIN_HISTORY()` for real-time/short window) |
| All queries ever run, incl. old/dropped-object queries | `ACCOUNT_USAGE.QUERY_HISTORY` (vs `INFORMATION_SCHEMA.QUERY_HISTORY()` for recent/real-time) |
| Column-level read/write audit (who touched sensitive data) | `ACCOUNT_USAGE.ACCESS_HISTORY` *(Enterprise+ only)* |
| Credit usage per warehouse over time | `ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY` |
| Storage used, incl. Time Travel & Fail-safe bytes | `ACCOUNT_USAGE.TABLE_STORAGE_METRICS`, `ACCOUNT_USAGE.STORAGE_USAGE`, `ACCOUNT_USAGE.DATABASE_STORAGE_USAGE_HISTORY` |
| Data loaded via COPY INTO / Snowpipe | `ACCOUNT_USAGE.COPY_HISTORY` / `ACCOUNT_USAGE.LOAD_HISTORY` (or `INFORMATION_SCHEMA.COPY_HISTORY()` for recent) |
| Task execution history / failures | `ACCOUNT_USAGE.TASK_HISTORY` (or `INFORMATION_SCHEMA.TASK_HISTORY()`) |
| Automatic clustering credit usage | `ACCOUNT_USAGE.AUTOMATIC_CLUSTERING_HISTORY` |
| Materialized view refresh activity | `ACCOUNT_USAGE.MATERIALIZED_VIEW_REFRESH_HISTORY` |
| Replication credit/data usage | `ACCOUNT_USAGE.REPLICATION_USAGE_HISTORY` |
| Cross-region/cross-cloud data transfer | `ACCOUNT_USAGE.DATA_TRANSFER_HISTORY` |
| Which grants exist on a role/user | `ACCOUNT_USAGE.GRANTS_TO_ROLES`, `ACCOUNT_USAGE.GRANTS_TO_USERS` (or `INFORMATION_SCHEMA.APPLICABLE_ROLES` for a real-time role list) |
| Where a masking/row access policy is applied | `INFORMATION_SCHEMA.POLICY_REFERENCES()` table function (real-time; works either schema) |
| List all masking / row access policies in account | `ACCOUNT_USAGE.MASKING_POLICIES`, `ACCOUNT_USAGE.ROW_ACCESS_POLICIES` |
| Object dependency mapping (view → table, etc.) | `ACCOUNT_USAGE.OBJECT_DEPENDENCIES` |
| Tag assignments across the account | `ACCOUNT_USAGE.TAG_REFERENCES`, `TAG_REFERENCES_ALL_COLUMNS` |
| Active user sessions | `ACCOUNT_USAGE.SESSIONS` |
| List of all tables/views/columns account-wide, incl. dropped | `ACCOUNT_USAGE.TABLES`, `ACCOUNT_USAGE.COLUMNS`, `ACCOUNT_USAGE.VIEWS` |
| Real-time list of tables/columns in *this* database only | `INFORMATION_SCHEMA.TABLES`, `INFORMATION_SCHEMA.COLUMNS` |
| Org-wide spend / usage across multiple accounts | `ORGANIZATION_USAGE` schema (e.g., `USAGE_IN_CURRENCY_DAILY`, `WAREHOUSE_METERING_HISTORY`) — needs **ORGADMIN** |
| Reader-account usage | `READER_ACCOUNT_USAGE` schema |

> To grant a custom role access to `ACCOUNT_USAGE` views without exposing the whole SNOWFLAKE database (avoids over-privileging), grant a **SNOWFLAKE database role** (e.g., `GOVERNANCE_VIEWER`) instead of blanket `IMPORTED PRIVILEGES`.

---

## PART 7 — Warehouses, Credits & Resource Monitors

### 7.1 Warehouse Sizes & Credit Consumption (per full continuous hour, single cluster)

| Size | Credits/hr |
|---|---|
| X-Small | 1 |
| Small | 2 |
| Medium | 4 |
| Large | 8 |
| X-Large | 16 |
| 2X-Large | 32 |
| 3X-Large | 64 |
| 4X-Large | 128 |
| 5X-Large | 256 |
| 6X-Large | 512 |

- Billing is **per-second** with a **60-second minimum** each time a warehouse starts.
- Each size step **doubles** compute resources (and credit cost) vs. the previous size.
- **Multi-cluster warehouses require Enterprise Edition (or higher)** — used for concurrency scaling, not raw query speed.
- Auto-suspend / auto-resume are both **on by default**; auto-suspend wipes the warehouse's local disk cache.

### 7.2 Resource Monitors
- Track/limit **credit usage** at the **account** level or the **warehouse** level.
- **Only `ACCOUNTADMIN` can create a resource monitor by default** (privilege is grantable to custom roles via `CREATE RESOURCE MONITOR`).
- Actions per monitor: **one `SUSPEND`**, **one `SUSPEND_IMMEDIATE`**, **up to five `NOTIFY`** actions.
  - `SUSPEND` — lets currently running queries finish, then suspends (new queries blocked).
  - `SUSPEND_IMMEDIATE` — cancels running queries immediately and suspends.
  - `NOTIFY` — sends email only; doesn't stop anything.
- A single warehouse can be assigned to only **one** warehouse-level resource monitor (but an account-level monitor also applies on top).
```sql
CREATE OR REPLACE RESOURCE MONITOR limiter
  WITH CREDIT_QUOTA = 5000
  TRIGGERS ON 75 PERCENT DO NOTIFY
           ON 100 PERCENT DO SUSPEND
           ON 110 PERCENT DO SUSPEND_IMMEDIATE;
ALTER WAREHOUSE wh1 SET RESOURCE_MONITOR = limiter;
```

### 7.3 Caching Layers (classic exam topic)

| Cache | Layer | What it stores | Lifetime |
|---|---|---|---|
| **Query Result Cache** | Cloud Services | Full result set of a previous *identical* query | 24 hrs, extended on reuse, max 31 days; no warehouse needed |
| **Metadata Cache** | Cloud Services | Row counts, min/max per column, distinct counts, micro-partition stats | Always current; powers `COUNT(*)`/MIN/MAX without a warehouse; not usable for character-column min/max |
| **Warehouse (Local Disk/SSD) Cache** | Compute (per-warehouse) | Raw micro-partition data pulled from remote storage | Lost when warehouse suspends; not shared across clusters in a multi-cluster warehouse |

---

## PART 8 — Time Travel, Fail-safe & Cloning

| Parameter | Standard | Enterprise+ |
|---|---|---|
| Default Time Travel retention | 1 day | 1 day (default) |
| Max Time Travel (permanent tables) | **1 day (cannot go higher)** | **Up to 90 days** |
| Max Time Travel (transient/temporary tables, all editions) | 0 or 1 day | 0 or 1 day |
| Fail-safe (permanent tables only) | 7 days, non-configurable | 7 days, non-configurable |
| Fail-safe for transient/temporary tables | **None — not available** | **None — not available** |

- Set via object/account parameter: `DATA_RETENTION_TIME_IN_DAYS`. Settable by `ACCOUNTADMIN` (account level) or object owner (db/schema/table level).
- Time Travel **can be set to 0** (disables it for that object) but **cannot be disabled account-wide**.
- `MIN_DATA_RETENTION_TIME_IN_DAYS` — account parameter that enforces a minimum retention floor.
- Historical query syntax: `AT`/`BEFORE` clauses with `TIMESTAMP`, `OFFSET`, or `STATEMENT` (query ID); `UNDROP` restores dropped tables/schemas/databases/accounts/tags/external volumes.
- After Time Travel expires → data moves to Fail-safe (permanent tables only) → **Fail-safe recovery requires contacting Snowflake Support**, not user-accessible via SQL.
- External tables: **no Time Travel, no Fail-safe** (data lives outside Snowflake storage).
- Zero-copy **cloning** (`CREATE ... CLONE`) can target a point in time using `AT`/`BEFORE`; clone doesn't duplicate storage until data diverges.

---

## PART 9 — Data Loading, Stages, File Formats, Snowpipe

- **Stage types**: internal (user `@~`, table `@%table`, named `@stage`) vs. external (S3/Azure Blob/GCS).
- Privileges: `USAGE` (external stage), `READ`/`WRITE` (internal stage — `WRITE` requires `READ` be granted first).
- `COPY INTO <table>` — bulk/batch loading; tracks load history to avoid reloading the same file (`LOAD_HISTORY`/`COPY_HISTORY`).
- **Snowpipe** — continuous, serverless micro-batch loading (event-driven via cloud notifications or REST call); billed separately from warehouse credits.
- **Snowpipe Streaming** — low-latency row-level ingestion without staged files.
- File formats are schema-level objects (`CREATE FILE FORMAT`); can be inline in `COPY INTO` or referenced by name.
- `VALIDATION_MODE` and `ON_ERROR` control error handling during `COPY INTO`.

---

## PART 10 — Streams & Tasks

- **Streams** track table changes (insert/update/delete) via a "change table" derived from the source table's metadata/offset (not a physical copy).
  - **Standard stream** — full delta (inserts, updates, deletes); requires **change tracking enabled** on the source table.
  - **Append-only stream** — captures inserts only; faster for pure ingestion pipelines.
  - **Insert-only stream** — for **external tables** only.
  - Streams don't consume storage the way a table does; they become "stale" if unconsumed beyond the source table's Time Travel retention.
- **Tasks** schedule execution of a single SQL statement/procedure, often triggered `AFTER` a stream has data (checked via `SYSTEM$STREAM_HAS_DATA()`).
  - Can run on a **user-managed warehouse** or **serverless compute** (Snowflake-managed).
  - Tasks form **DAGs** (a root task plus child tasks); requires `EXECUTE TASK` (user-managed) or `EXECUTE MANAGED TASK` (serverless) privilege.
  - A **suspended** task must be resumed (`ALTER TASK ... RESUME`) — newly created tasks start suspended by default.

---

## PART 11 — Data Sharing

- **Provider side**: `CREATE SHARE` (account privilege, default ACCOUNTADMIN); grant DB/schema/object privileges to the share (or grant a **database role** to the share — recommended, more granular).
- **Consumer side**: global `IMPORT SHARE` privilege lets a role view inbound shares; combined with `CREATE DATABASE`, lets it materialize a share into a local database.
- **Reader accounts** — for consumers who don't have their own Snowflake account; created and managed by the provider.
- Sharing doesn't copy data — consumers query the provider's storage directly (no data movement/duplication, no separate storage cost to the consumer).
- **Replication & failover**: database/share replication is available on **all editions**; **failover/failback** (automatic promotion of a secondary to primary) requires **Business Critical**.

---

## PART 12 — Micro-Partitions & Clustering

- Snowflake automatically divides table data into immutable **micro-partitions** (roughly 50–500 MB uncompressed, columnar, compressed).
- Each micro-partition stores metadata: min/max per column, distinct counts — this metadata drives **pruning** (skipping irrelevant partitions before scanning).
- **Clustering keys** — optional; used on very large tables to co-locate similar values into the same micro-partitions, improving pruning. Automatic reclustering is a background (billed) service.
- **Search Optimization Service** *(Enterprise+)* — accelerates highly selective point-lookup queries (equality/`IN` predicates), different from clustering (which mainly benefits range predicates).

---

## PART 13 — Semi-Structured Data & Sharing Quick Notes

- Native types: `VARIANT`, `OBJECT`, `ARRAY`; native support for JSON, Avro, ORC, Parquet, XML.
- Dot notation and `:` for path traversal on VARIANT columns; `FLATTEN()` table function to explode arrays/objects into rows.
- `TRY_CAST`/`TRY_PARSE_JSON` for safe conversion without erroring on bad data.

---

## PART 14 — Quick Command Cheat Sheet (SET / APPLY / UNSET / SHOW / DESCRIBE)

| Task | Command pattern |
|---|---|
| Attach masking policy to a column | `ALTER TABLE t MODIFY COLUMN c SET MASKING POLICY p;` |
| Remove masking policy from a column | `ALTER TABLE t MODIFY COLUMN c UNSET MASKING POLICY;` |
| Attach row access policy to a table | `ALTER TABLE t ADD ROW ACCESS POLICY p ON (col);` |
| Remove row access policy from a table | `ALTER TABLE t DROP ROW ACCESS POLICY p;` |
| Grant account-wide right to attach masking policies | `GRANT APPLY MASKING POLICY ON ACCOUNT TO ROLE r;` |
| Grant account-wide right to attach row access policies | `GRANT APPLY ROW ACCESS POLICY ON ACCOUNT TO ROLE r;` |
| See a network policy's actual IP/rule values | `DESCRIBE NETWORK POLICY p;` |
| See which network policy is active | `SHOW PARAMETERS LIKE 'network_policy' IN ACCOUNT;` / `... IN USER u;` |
| Activate a network policy account-wide | `ALTER ACCOUNT SET NETWORK_POLICY = p;` |
| Activate a network policy for one user | `ALTER USER u SET NETWORK_POLICY = p;` |
| Find where a policy (masking/row access) is applied | `SELECT * FROM TABLE(INFORMATION_SCHEMA.POLICY_REFERENCES(POLICY_NAME => 'p'));` |
| Restore a dropped object | `UNDROP TABLE t;` (also works for SCHEMA, DATABASE, TAG, ACCOUNT, external volume) |
| Set custom Time Travel retention | `ALTER TABLE t SET DATA_RETENTION_TIME_IN_DAYS = 30;` |
| Suspend/resume a warehouse manually | `ALTER WAREHOUSE w SUSPEND;` / `ALTER WAREHOUSE w RESUME;` |
| Assign a resource monitor | `ALTER WAREHOUSE w SET RESOURCE_MONITOR = m;` |
| Resume a suspended task | `ALTER TASK t RESUME;` |
| Check if a stream has data before running a task | `SYSTEM$STREAM_HAS_DATA('stream_name')` |
| Disable result cache for a session | `ALTER SESSION SET USE_CACHED_RESULT = FALSE;` |

---

## PART 15 — Commonly Confused Pairs (exam traps)

| A | B | Difference |
|---|---|---|
| `CREATE MASKING POLICY` | `APPLY MASKING POLICY` | CREATE = author the policy object (schema privilege). APPLY = attach/detach it on a column (account privilege, or narrower per-policy APPLY + table OWNERSHIP) |
| `SUSPEND` | `SUSPEND_IMMEDIATE` (resource monitor) | SUSPEND waits for running queries to finish; SUSPEND_IMMEDIATE cancels them now |
| Time Travel | Fail-safe | Time Travel = user-queryable, configurable (0–90 days). Fail-safe = fixed 7 days, Snowflake-Support-only recovery, permanent tables only |
| `ACCOUNT_USAGE` | `INFORMATION_SCHEMA` | ACCOUNT_USAGE = account-wide, latent, 365-day retention, includes dropped objects. INFORMATION_SCHEMA = per-DB, real-time, short retention, active objects only |
| `SHOW NETWORK POLICIES` | `DESCRIBE NETWORK POLICY` | SHOW = list of policies + IP-count summary. DESCRIBE = actual IP/rule values for one policy |
| Masking policy | Row access policy | Masking = column-level, redacts/transforms a value. Row access = row-level, filters whether a row is returned at all. Row access evaluates first if both exist |
| `SECURITYADMIN` | `USERADMIN` | SECURITYADMIN has `MANAGE GRANTS` (any object, account-wide) + inherits USERADMIN. USERADMIN can only manage users/roles it owns |
| Standard stream | Append-only stream | Standard = tracks inserts+updates+deletes. Append-only = inserts only, lighter-weight |
| Multi-cluster warehouse | Warehouse resizing | Multi-cluster = scale **out** for concurrency (Enterprise+). Resizing = scale **up/down** compute per query (all editions) |
| `OWNERSHIP` privilege | `MANAGE GRANTS` privilege | OWNERSHIP = full control of one specific object. MANAGE GRANTS = ability to grant/revoke privileges on ANY object account-wide |

---

### Sources
Snowflake official documentation (docs.snowflake.com): editions & feature matrix, access control overview & privileges, CREATE/ALTER/DROP/DESCRIBE MASKING POLICY, CREATE/ALTER/DROP ROW ACCESS POLICY, network policies & network rules guide, CREATE/ALTER/SHOW/DESCRIBE NETWORK POLICY, ACCOUNT_USAGE reference, Time Travel guide, resource monitors guide, warehouses overview/considerations — current as of July 2026.


---

# PART 16 — Ultra High-Yield Exam Traps (Added & Updated)

## Minimum Roles / Privileges

| Task | Minimum privilege / role |
|------|---------------------------|
| Verify warehouse resize completed | `ALTER WAREHOUSE ... WAIT_FOR_COMPLETION = TRUE` |
| View privileges granted to a role | `SHOW GRANTS TO ROLE <role>;` |
| View network policy IP lists | `DESCRIBE NETWORK POLICY <policy>;` |
| Apply an existing masking policy | `ALTER TABLE ... MODIFY COLUMN ... SET MASKING POLICY ...` |
| Upload local file to internal stage | `PUT file://... @stage` |
| Download staged file | `GET @stage file://...` |
| Add Search Optimization | Requires **OWNERSHIP** on table **AND** `ADD SEARCH OPTIMIZATION` privilege on schema |
| Enable account replication | `ORGADMIN` (or `GLOBALORGADMIN` where applicable) sets `ENABLE_ACCOUNT_DATABASE_REPLICATION` |
| Create Resource Monitor | `CREATE RESOURCE MONITOR` account privilege (ACCOUNTADMIN has it by default) |

---

## COPY INTO Validation Modes

| Option | Returns |
|---------|----------|
| RETURN_n_ROWS | Rows that would be loaded |
| RETURN_ERRORS | Errors only for current COPY validation |
| **RETURN_ALL_ERRORS** | **Returns ALL errors including files partially loaded previously** ⭐ |
| RETURN_<n>_ROWS | Preview rows |

---

## COPY INTO Loading Facts

- COPY uses warehouse compute.
- Snowpipe is serverless.
- COPY can:
  - Reorder columns
  - Omit columns
  - Cast datatypes
  - Load subset of columns
- COPY cannot perform joins or aggregations.

---

## Streams

| Type | Supports |
|------|----------|
| Standard | Inserts + Updates + Deletes |
| Append-only | Inserts only |
| Insert-only | External Tables only |

Unconsumed streams temporarily extend table retention until stream offset.

---

## Result Cache Invalidators

Cache is NOT reused if:

- Query text changes.
- SELECT list changes.
- Underlying object changes.
- Non-deterministic functions are used.
- Micro-partitions change.
- Session settings affecting result change.
- Objects referenced change.

Cache IS reused even if:

- Warehouse changes.
- Warehouse size changes.
- Different warehouse executes identical query.

---

## ACCOUNT_USAGE vs INFORMATION_SCHEMA

Remember:

ACCOUNT_USAGE
- 45 min–3 hr latency
- Includes dropped objects
- 365-day retention
- Historical analysis

INFORMATION_SCHEMA
- Real-time
- Current metadata
- Table functions for recent history

---

## Warehouse Scaling

Scale Up
- Larger warehouse
- Faster single query

Scale Out
- Multi-cluster
- More concurrent queries

Scaling Policy

- Standard → performance first
- Economy → cost first

---

## Frequently Tested Commands

```sql
SHOW GRANTS TO ROLE role_name;

DESCRIBE NETWORK POLICY policy_name;

SHOW PARAMETERS LIKE 'NETWORK_POLICY' IN ACCOUNT;

ALTER TABLE t MODIFY COLUMN c SET MASKING POLICY p;

ALTER TABLE t MODIFY COLUMN c UNSET MASKING POLICY;

ALTER TABLE t ADD ROW ACCESS POLICY p ON(col);

ALTER TABLE t DROP ROW ACCESS POLICY p;

GET @stage file://...

PUT file://... @stage;

SELECT * FROM TABLE(VALIDATE(table_name, JOB_ID => '_last'));

SYSTEM$STREAM_HAS_DATA('stream');

ALTER WAREHOUSE wh SET WAIT_FOR_COMPLETION = TRUE;
```

---

## Very Common SnowPro MCQ Traps

- Fail-safe exists **only** for Permanent tables.
- Temporary & Transient tables have **no Fail-safe**.
- Search Optimization ≠ Clustering.
- Scale Up improves single-query performance.
- Scale Out improves concurrency.
- RESULT_SCAN reads cached results; it does **not** invalidate cache.
- BLOCKED IP always wins over ALLOWED IP.
- Secure Views hide both underlying SQL definition and protected data.
- ACCOUNT_USAGE contains dropped objects.
- INFORMATION_SCHEMA is real-time.
- PUT uploads.
- GET downloads.
- RETURN_ALL_ERRORS includes previously partially loaded files.
- APPLY privilege is different from CREATE privilege.
- SHOW NETWORK POLICIES does **not** display IP values.
- DESCRIBE NETWORK POLICY displays actual IP values.
- WAIT_FOR_COMPLETION waits until warehouse resize finishes.

---

## Memorize These Numbers

| Item | Value |
|------|-------|
| Query Result Cache | 24 hours |
| ACCOUNT_USAGE latency | 45 min–3 hours |
| Fail-safe | 7 days |
| Standard Time Travel | 1 day |
| Enterprise Time Travel | up to 90 days |
| Warehouse minimum billing | 60 seconds |
| Recommended bulk file size | 100–250 MB compressed |
| ACCOUNT_USAGE retention | 365 days |

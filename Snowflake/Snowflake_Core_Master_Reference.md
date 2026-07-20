# Snowflake SnowPro Core — Master Reference
*Built from your mock-test content, verified against current Snowflake documentation (July 2026). Organized by theme rather than by question number so you can use it as a standalone study sheet.*

> ⚠️ **A note on scope**: The md file an earlier Claude session generated for you wasn't actually attached to this conversation (only your exam export was), so this was rebuilt from scratch off that exam content — verified line by line — rather than edited in place. Two corrections worth flagging up front (details below in context): the "view network policy IP lists" answer, and the "UNDROP a stage" answer, both need a caveat against what's currently documented.

---

## 1. Roles, RBAC & Access Control

### System-defined role hierarchy
`ACCOUNTADMIN` → `SECURITYADMIN` / `SYSADMIN` → `USERADMIN` → `PUBLIC` (every role and user implicitly has `PUBLIC`).

| Role | Purpose |
|---|---|
| **ACCOUNTADMIN** | Top-level; combines SYSADMIN + SECURITYADMIN; billing, account-level settings |
| **SECURITYADMIN** | **Recommended role for creating and managing users and roles** (manages grants globally; inherits USERADMIN) |
| **SYSADMIN** | Recommended for creating warehouses, databases, and other objects; custom roles are typically created under SYSADMIN in the hierarchy |
| **USERADMIN** | Dedicated to user/role management only |
| **PUBLIC** | Automatically granted to every user/role; effectively a "default" role |
| **ORGADMIN** | Manages operations at the organization level (creating accounts, viewing all accounts, enabling replication) |

### Discretionary Access Control (DAC) vs Role-Based Access Control (RBAC)
- **DAC**: each object has an **owner**, and the owner can grant access to that object.
- **RBAC**: privileges are assigned to **roles**, and roles are assigned to users. Snowflake uses **both** models together — DAC governs object ownership/transfer, RBAC governs how privileges flow to users.

### Key privilege facts (minimum privilege / role questions)
| Action | Minimum privilege / role |
|---|---|
| Create/manage users & roles | **SECURITYADMIN** (or a role with equivalent grants) |
| Set `ENABLE_ACCOUNT_DATABASE_REPLICATION` parameter | **ORGADMIN** (uses `SYSTEM$GLOBAL_ACCOUNT_SET_PARAMETER`) — note: enabling/managing **database replication and failover itself** (after the org-level switch is on) requires **ACCOUNTADMIN** |
| Access a file URL from an external stage | **USAGE** privilege (on the stage) |
| Grant an object to a Share | `GRANT <privilege> ... TO SHARE <share_name>` (requires OWNERSHIP or WITH GRANT OPTION on the object) |
| Add/remove Search Optimization on a table | **OWNERSHIP** on the table **AND** the schema-level **ADD SEARCH OPTIMIZATION** privilege on the schema containing the table (both needed) |
| Set/replace a masking policy on a column | **APPLY MASKING POLICY** privilege (granted to a role, e.g. a custom `MASKING_ADMIN` role) |
| Create a masking policy | **CREATE MASKING POLICY** privilege on the schema |
| Create a network policy | **SECURITYADMIN** or higher, or a role granted the global **CREATE NETWORK POLICY** privilege |
| View/alter a network policy | Owner (OWNERSHIP privilege) or higher |
| Set a data retention **minimum** for the whole account | **ACCOUNTADMIN**, via the `MIN_DATA_RETENTION_TIME_IN_DAYS` **account-level** parameter |
| Set retention for an individual DB/schema/table | Any role with sufficient privilege on the object, via `DATA_RETENTION_TIME_IN_DAYS` |
| Alter a table (incl. adding clustering) | **OWNERSHIP** on the table; clustering also needs USAGE/OWNERSHIP on schema+database |
| Specify a cluster key | `CREATE TABLE ... CLUSTER BY` or `ALTER TABLE ... CLUSTER BY` (both valid — **not** `SET` or `SHOW`) |
| Remove a role from a user/another role | `REVOKE ROLE` |
| View grants on a role | `SHOW GRANTS ON ROLE <role>` (note: **not** "SHOW GRANTS TO ROLE" / "SHOW GRANTS FOR ROLE" — those aren't valid variants) |
| Replicate at the database/schema level | Requires the **REPLICATE** account-level privilege (grantable by ACCOUNTADMIN) plus USAGE on the DB/schema |

### Best practice for custom roles
Create custom roles **using the SYSADMIN role** (or a role below SYSADMIN in the hierarchy) so that SYSADMIN retains visibility/control over all custom objects — then grant the custom role to SYSADMIN or an appropriate role in the hierarchy so privileges roll up correctly. Avoid creating custom roles directly under ACCOUNTADMIN unless necessary.

---

## 2. Snowflake Editions — Minimum Edition Requirements

| Feature | Minimum Edition |
|---|---|
| Time Travel (up to 1 day) | **Standard** (all editions) |
| Time Travel beyond 1 day (up to 90 days) | **Enterprise** or higher |
| Multi-cluster warehouses | **Enterprise** or higher |
| Search Optimization Service | **Enterprise** or higher |
| Materialized Views | **Enterprise** or higher |
| **Column-level Security** (masking policies) | **Enterprise** or higher |
| Row Access Policies | **Enterprise** or higher |
| Object tagging | **Enterprise** or higher |
| PCI DSS / HIPAA-HITRUST (PHI) support, Tri-Secret Secure, account failover/failback | **Business Critical** or higher |
| **PII / highly regulated data compliance** — commonly tested as "choose two" | **Business Critical** *and* **Virtual Private Snowflake (VPS)** |
| **Dedicated metadata store + dedicated pool of compute resources** | **Virtual Private Snowflake (VPS) only** (this is the highest tier; Business Critical does *not* include this) |
| Complete environment isolation (no shared hardware with any other account) | **VPS only** |

> Quick way to remember the ladder: **Standard → Enterprise (perf/security-lite) → Business Critical (regulated data, PHI/HIPAA) → VPS (full isolation + dedicated metadata store)**. Each edition includes everything below it.

To check your account's edition in SQL: query the `EDITION` column of `SNOWFLAKE.ORGANIZATION_USAGE.ACCOUNTS` (requires access to that schema).

---

## 3. Virtual Warehouses

### Core architecture facts
- A virtual warehouse is an **independent MPP compute cluster**; each warehouse operates independently and does **not** share compute resources with other warehouses (this eliminates resource contention between workloads — one of Snowflake's key architecture selling points).
- Snowflake's architecture overall is described as a **multi-cluster, shared-data architecture** using a central data repository and **massively parallel processing (MPP)** — not "shared-nothing," not "single-cluster," not SMP.
- Resizing a warehouse **does not guarantee** better data-loading performance (loading is often bottlenecked by file count/size, not warehouse size).

### Scaling policies (multi-cluster warehouses, Enterprise+)
| Policy | Behavior |
|---|---|
| **Standard** (default) | Prioritizes preventing queuing — starts an additional cluster quickly (~20 seconds of sustained queuing) once the first cluster is fully loaded |
| **Economy** | Prioritizes credit conservation — waits (~6 minutes of sustained load) before starting another cluster, favoring keeping existing clusters fully loaded over spinning up new ones; may cause more queuing |
| ~~Legacy~~ | Removed; all warehouses using it were migrated to Standard |

- Valid scaling policies are only **Standard** and **Economy** — "Custom" and "Optimized" are **not** real Snowflake scaling policies.
- **Auto-scale mode**: MIN_CLUSTER_COUNT ≠ MAX_CLUSTER_COUNT. **Maximized mode**: MIN = MAX (all clusters start immediately, no policy applies).
- If **MIN_CLUSTER_COUNT is increased** and the new minimum is greater than the number of clusters currently running, Snowflake **immediately** starts additional clusters to satisfy the new minimum (this is the one scenario where clusters start right away regardless of scaling policy timing).

### Scale up vs. scale out
| Scenario | Solution |
|---|---|
| One large, complex query is slow | **Scale up** (resize the warehouse to a larger size) |
| Many concurrent queries are queuing (concurrency problem) | **Scale out** (multi-cluster warehouse / add clusters, or route to a second warehouse) — resizing does **not** fix concurrency/queuing by itself |

### Cost control
- **Resource monitors** let you cap credit consumption at the account or warehouse level (not "tracking usage" or "scaling," which don't cap anything by themselves).
- Automating **auto-suspend / auto-resume** is the standard way to lower ongoing credit consumption (not increasing cluster counts or resizing bigger).
- `INITIALLY_SUSPENDED = TRUE | FALSE` — parameter on `CREATE WAREHOUSE` controlling whether the warehouse starts immediately after creation.
- Billing is **per-second**, with a **60-second (1 minute) minimum** charge the first time a warehouse starts in a billing cycle.
- Warehouse cache: resizing a warehouse **can** lose the cached data if the resize is to a *smaller* size and the cache no longer fits — it is not guaranteed to survive, and it's also not automatically wiped just because compute changed.

### Verifying provisioning
`ALTER WAREHOUSE ... SET WAREHOUSE_SIZE = ... WAIT_FOR_COMPLETION = TRUE` — the `WAIT_FOR_COMPLETION` property makes the ALTER statement block until the additional compute resources are fully provisioned.

### Warehouses for tasks
Because a task's underlying SQL runs on its own schedule (not continuously), the recommended practice is to configure the **serverless (or dedicated) warehouse for the task with auto-suspend/auto-resume**, and size it based on the actual workload the task body performs — test the task's statements standalone to determine appropriate sizing.

### Use cases that require a running warehouse
Loading data, unloading data, and any query execution **require** an active warehouse. Metadata-only operations like `SHOW` and `LIST` commands **do not** require a running warehouse.

---

## 4. Data Loading & Unloading

### Loading
| Scenario | Best tool |
|---|---|
| Continuous, near real-time, serverless ingestion as files land in a stage | **Snowpipe** (event-driven, serverless — most efficient way to load *streaming* data) |
| Bulk historical/batch loads | `COPY INTO <table>` with a warehouse |
| Fastest way to bulk-load from a stage | Specify an **explicit list of files** (`FILES = (...)`) — this is generally faster than pattern matching (regex `PATTERN`) or listing by path/prefix, since Snowflake doesn't have to enumerate/filter |

**Recommended file size**: **100–250 MB compressed** per file. This size range allows Snowflake to **parallelize load operations effectively** across the threads of a warehouse (it is *not* about warehouse sizing/scaling mode, and not about sequential importing).
**Snowpipe file size** follows the same guidance: >100 MB up to 250 MB compressed is recommended (loading files that are too small increases per-file overhead).

**Best practices when loading**:
- Load files in the ~100–250 MB (or larger) compressed range.
- Avoid embedded characters (e.g., commas) inside numeric-type columns — they'll cause load errors or misparses.
- (You do *not* need to strip out dates/timestamps or remove semi-structured types before loading — those are both valid, supported data types.)

**COPY INTO <table> supports** (during a plain/standard load): reordering columns, omitting columns, and converting data types on load. It does **not** support aggregate functions or GROUP BY — those require a transformation query, and even then COPY's transformation support is limited (no joins with other tables either).

**Error handling / validation**
- `VALIDATION_MODE = RETURN_n_ROWS | RETURN_ERRORS | RETURN_ALL_ERRORS` — a COPY option that validates without loading.
  - `RETURN_n_ROWS` (e.g. `RETURN_10_ROWS`): validates and returns N rows if no errors.
  - `RETURN_ERRORS`: returns all errors across the specified files.
  - **`RETURN_ALL_ERRORS`**: returns all errors across the specified files **including files that were partially loaded in an earlier run** (where `ON_ERROR = CONTINUE`). This is the one to use when re-checking a batch that partially succeeded before.
- `VALIDATE(<table>, JOB_ID => '_last')` table function: returns all errors from the **most recently executed COPY INTO `<table>`** command **in the current session** (not "in the last N days," not "across all sessions").
- `ON_ERROR` options: `CONTINUE`, `SKIP_FILE`, `SKIP_FILE_<num>`, `SKIP_FILE_<num>%`, `ABORT_STATEMENT` (default).

**JSON / semi-structured performance**
For frequently-queried JSON with lots of nested arrays/dates, the **most performant** pattern is to **flatten the data into structured (relational) columns** in a proper table and query that — rather than leaving everything inside a single VARIANT column (VARIANT still works, but forces the engine to scan/traverse the whole semi-structured blob instead of scanning individual columnar data).

### Unloading (COPY INTO <location>)
- `OBJECT_CONSTRUCT` combined with `COPY INTO <location>` lets you convert relational table rows into a single VARIANT/JSON-like object per row before unloading — a common pattern for exporting JSON.
- By default, `COPY INTO <location>` **does** split output into multiple files (it does *not* produce one file per table unless `SINGLE = TRUE` is set); default for `SINGLE` is `FALSE`.
- To preserve an **exact target filename** on unload (a file format option, not a copy option): use the **`FILE_EXTENSION`** file format option to control the extension on the auto-generated unload filenames (SINGLE=TRUE plus a fully specified path is the mechanism for an exact single filename — but among *file format options* specifically, FILE_EXTENSION is the relevant knob).
- Recommended unload practice: **define an individual/explicit file format** for the unload rather than relying on defaults; avoid disabling `CAST`, and avoid assuming Parquet needs `OBJECT_CONSTRUCT(*)` (that's a JSON-export pattern, not required for Parquet).
- `COPY INTO @stage ...` is the actual unload/export statement. (`EXPORT TO`, `INSERT INTO @stage`, and `EXPORT_TO_STAGE(...)` are **not** real Snowflake syntax.)

---

## 5. Stages, PUT/GET & File URLs

| Command | Direction | Notes |
|---|---|---|
| **PUT** | Local machine → internal stage (upload) | Supported via **SnowSQL** and the **Python connector** (and other drivers); **not** supported via the SQL API |
| **GET** | Internal stage → local machine (download) | Same client support as PUT |
| **LIST** | Show files staged | No warehouse required |
| **COPY INTO** | Stage ↔ table | The actual load/unload data-movement command |

- `GET`/`PUT` file-path syntax should use **forward slashes**, even on Windows (e.g. `GET @%TBL_EMPLOYEE 'file://C:/folder with space/'`) — backslash-style Windows paths are not the correct syntax for these commands.
- A **Directory Table** stores **file-level metadata** (filename, size, last-modified, etc.) for the files staged in a given stage — separate from External or Temporary/Transient tables, which serve different purposes.

### URL types for accessing staged files
| URL type | Behavior |
|---|---|
| **File URL** | Requires authentication/authorization to access |
| **Scoped URL** | Temporary, session-scoped, requires the caller to be authenticated |
| **Pre-signed URL** | Grants **access without further Snowflake authorization** — the credential is embedded in the URL itself, with an expiration |
| **Scoped file URL** | Similar to scoped URL but tied to specific query context |

Minimum privilege to obtain/use a file URL from an external stage: **USAGE** on the stage (not SELECT/READ/MODIFY — those aren't the relevant privilege names here).

---

## 6. Storage, Time Travel, Fail-safe & CDP

### Time Travel
- Default retention: **1 day** for Standard Edition (can be set to 0).
- Retention beyond 1 day (up to 90 days) requires **Enterprise Edition** or higher.
- `DATA_RETENTION_TIME_IN_DAYS`: object/account parameter controlling Time Travel retention.
- **`MIN_DATA_RETENTION_TIME_IN_DAYS`**: an **account-level** parameter, settable only by **ACCOUNTADMIN**, that enforces a *floor* on retention across the account — the effective retention for any object is `MAX(object's DATA_RETENTION_TIME_IN_DAYS, MIN_DATA_RETENTION_TIME_IN_DAYS)`.
- **UNDROP** (part of Time Travel) currently supports: **tables, schemas, databases, accounts, external volumes, and tags**.
  > ⚠️ **Correction vs. a common exam answer**: your reference material's marked answer for "how to restore a dropped internal stage" was *"Execute the UNDROP command."* Per current Snowflake SQL reference, **UNDROP is not listed as supported for stages**, and the `DROP STAGE` documentation explicitly states that for an *internal* stage, all files are purged and **cannot be recovered** once the stage is dropped. In practice, the realistic answer today is to **recreate the stage** (an external stage recreation is trivial since the data lives outside Snowflake; an internal stage's files are genuinely gone). Treat "UNDROP a stage" with caution if it reappears on your real exam — it may reflect an older exam version.

### Fail-safe
| Table type | Fail-safe period |
|---|---|
| **Permanent tables** | **7 days**, recoverable **only by contacting Snowflake Support** (not self-service, not via SQL) |
| **Temporary tables** | **None** — no Fail-safe period |
| **Transient tables** | **None** — no Fail-safe period |

- Fail-safe storage costs apply to **permanent tables only** — not external tables, not files sitting in internal/external stages.
- Continuous Data Protection (CDP) for a **temporary table** maxes out at **24 hours total** (1 day Time Travel + 0 Fail-safe) — this is the smallest possible CDP window of any table type.

### Streams and retention interaction
If a stream on a table hasn't been consumed, and the table's normal retention period would otherwise expire, Snowflake **temporarily extends the retention period to the stream's offset** (up to a max of 14 days by default) so the stream doesn't go stale — the retention isn't *permanently* extended, and it isn't reduced to some fixed minimum.

---

## 7. Semi-Structured Data

- Native semi-structured data types: **VARIANT, ARRAY, OBJECT**. (**BLOB and CLOB are not Snowflake data types** — those are Oracle/other-RDBMS terms.)
- `FLATTEN` (used with `LATERAL`) converts a VARIANT/OBJECT/ARRAY into a relational row set. Its output columns are always: **SEQ, KEY, PATH, INDEX, VALUE, THIS.**
  - `SEQ`: sequence number of the input record.
  - `KEY`: for maps/objects, the key of the exploded value (NULL for plain arrays).
  - `PATH`: path to the flattened element.
  - `INDEX`: array position (NULL if not an array).
  - `VALUE`: the flattened value.
  - `THIS`: the element being flattened (useful for recursive flattening).
- For performance on data with lots of dates/arrays that will be queried often: **flatten into structured columns**, don't just leave it as VARIANT.

---

## 8. Streams & Tasks (Change Data Capture)

| Stream type | Tracks | Supported on |
|---|---|---|
| **Standard (delta)** | All DML: inserts, updates, deletes (incl. truncates); joins inserted+deleted rows to compute the net delta | Standard tables, directory tables, views, dynamic tables (delta only) |
| **Append-only** | **Inserts only** — much more performant since no delete-tracking overhead | Standard tables, directory tables, views |
| **Insert-only** | Inserts only, for **cloud storage files** behind external objects | **External tables**, Iceberg tables, Delta Direct tables — **not** standard tables |

- A stream contains **no data itself** — only an offset pointer into the source object's change history, plus metadata columns `METADATA$ACTION`, `METADATA$ISUPDATE`, `METADATA$ROW_ID`.
- `SYSTEM$STREAM_HAS_DATA('<stream_name>')` checks whether a stream currently has unconsumed change data.
- Streams/tasks are the standard mechanism for building CDC-driven ELT pipelines.

---

## 9. Query Performance, Caching & Search Optimization

### Caches
| Cache | Behavior |
|---|---|
| **Result cache** | Stores query *results*; persists ~24 hours from last use; a user can generally only reuse **their own** prior query's cached result (subject to same role/privileges on underlying data); evicted least-recently-used when full |
| **Metadata cache** | Used automatically — there's no `USE_METADATA_CACHE` setting a user needs to toggle |
| **Warehouse (local disk/data) cache** | Local to a running warehouse's compute; can be lost if the warehouse resizes down and the cache no longer fits, or when the warehouse fully suspends |

- `RESULT_SCAN(<query_id>)` table function can query/filter the contents of a **previous query's result set** (a common way to post-process `SHOW`/`COPY` command output too).
- Actions that **invalidate/bypass** the result cache: removing a column from the SELECT list, or any change to a column referenced by the query that isn't in the cached query's output (i.e., the underlying data changed in a way relevant to the query). Simply clustering data, or running `RESULT_SCAN()`, does **not** invalidate the cache.

### Query Profile — spotting a query that doesn't fit in memory
Look for: **"Bytes spilled to local storage"** and **"Bytes spilled to remote storage"** — remote spillage in particular indicates the query is memory-bound and needs a bigger warehouse (vertical scale-up), not more clusters.

Query Profile execution-time categories include things like: **Initialization**, **Local Disk I/O**, Remote Disk I/O, Network Communication, Synchronization, Processing (compute).

### Search Optimization Service (Enterprise+)
- Best suited for queries using **equality predicates or `IN` lists** on non-clustered columns of tables with many distinct values — it's built for point lookups on large tables, not for OR-only predicates, analytical window functions, or semi-structured filtering (those benefit less).
- Privileges to add/remove Search Optimization: **OWNERSHIP** on the table **and** **ADD SEARCH OPTIMIZATION** on the containing schema (both required, not either/or).
- Enable via: `ALTER TABLE <table> ADD SEARCH OPTIMIZATION [ON <columns>]`.

### Clustering
`CLUSTER BY` can be specified with either **`CREATE TABLE ... CLUSTER BY (...)`** or **`ALTER TABLE ... CLUSTER BY (...)`**. (`SET` and `SHOW` are not valid ways to specify a cluster key.)

---

## 10. Data Sharing & Replication

- Sharing an object with a consumer account: `GRANT <privilege> ... TO SHARE <share_name>` (there's no "automatic sharing" toggle, and you never need to drop/recreate objects to share them).
- Secure views/UDFs are the **recommended** objects to share (rather than raw tables) to avoid exposing sensitive underlying data.
- Replicating **accounts/databases** across regions:
  - Enabling the ability to replicate for a given account requires **ORGADMIN**, via `SYSTEM$GLOBAL_ACCOUNT_SET_PARAMETER(..., 'ENABLE_ACCOUNT_DATABASE_REPLICATION', 'true')`.
  - Actually configuring/managing database replication and failover day-to-day requires **ACCOUNTADMIN** (or a role granted the `REPLICATE` privilege).
  - Marketplace listings published into a **new remote region**: the **listing metadata** replicates automatically to selected regions, but the **underlying data does not** replicate automatically — for a standard listing, you can defer replicating the actual data until the first consumer in that region actually requests it.

---

## 11. Security & Governance

### Network Policies
- Defined as a set of rules that **control access to a Snowflake account by specifying allowed/blocked IP addresses (or CIDR ranges)**.
- Can be created/managed with `CREATE NETWORK POLICY`, `ALTER NETWORK POLICY`, and set at the **account level** or **individual user level** (user-level overrides account-level).
- If an IP address appears in **both** the allowed and blocked lists, Snowflake applies the **blocked list first** (the address is denied).
- Commands to inspect a network policy:
  - `SHOW NETWORK POLICIES` → returns a summary row per policy, including *counts* like `entries_in_allowed_ip_list` / `entries_in_blocked_ip_list` — **it does not display the actual IP values**.
  - **`DESCRIBE NETWORK POLICY <name>`** → this is the command that actually returns the **contents** of the allowed/blocked lists (or network rule lists).
  > ⚠️ **Correction vs. a common exam answer**: if your material marks `SHOW NETWORK POLICIES` as the way to "view the allowed and blocked IP list," that's imprecise per current docs — `SHOW` only gives you entry *counts*. To see the actual IP addresses/CIDR ranges, use **`DESCRIBE NETWORK POLICY`**.
- Newer Snowflake network policies use **network rules** (`CREATE NETWORK RULE`, schema-level objects) referenced via `ALLOWED_NETWORK_RULE_LIST` / `BLOCKED_NETWORK_RULE_LIST`, rather than the legacy `ALLOWED_IP_LIST` / `BLOCKED_IP_LIST` parameters directly on the policy. Both mechanisms still work, but Snowflake recommends network rules for new policies.

### Masking Policies (Column-level Security, Enterprise+)
| Task | Command |
|---|---|
| Create a masking policy | `CREATE MASKING POLICY <name> AS (val <type>) RETURNS <type> -> <expression>` |
| **Apply** (set) a masking policy on a column | `ALTER TABLE <t> MODIFY COLUMN <col> SET MASKING POLICY <policy>` (or `ALTER VIEW ... MODIFY COLUMN ... SET MASKING POLICY ...`) |
| Remove a masking policy from a column | `ALTER TABLE <t> MODIFY COLUMN <col> UNSET MASKING POLICY` |
| Update an existing policy's logic | `ALTER MASKING POLICY <name> SET BODY -> <new expression>` (column stays protected throughout the update — no need to unset first) |
| View a policy's current definition | `GET_DDL(...)` function, or `DESCRIBE MASKING POLICY <name>` |

Privileges: `CREATE MASKING POLICY` (on schema, to make one), `APPLY MASKING POLICY` (to attach one to a column), `OWNERSHIP` (full control).

### Secure Views
- Definition/logic is visible **only to users granted the role that owns the view** (i.e., holds **OWNERSHIP**) — everyone else cannot see the view's SQL text, even via `GET_DDL` or the Information Schema.
- Reasons to use a secure view: **(1) protect/hide sensitive underlying data from users who shouldn't see it, and (2) hide the view's definition/logic itself** — not to make it run faster (secure views typically run *slower* than regular views since they skip certain optimizer shortcuts) and not for "encryption in transit" (that's unrelated to view security).
- Created via `CREATE SECURE VIEW ...` / converted via `ALTER VIEW ... SET SECURE` (and `UNSET SECURE` to revert).
- `SHOW VIEWS` does **not** show secure view definitions — query `INFORMATION_SCHEMA.VIEWS` while using the owning role to see the definition.

### Multi-Factor Authentication (MFA)
- Snowflake's native MFA integration uses **Duo Security**.
- MFA is a built-in ("integrated") Snowflake feature — it is **not** enabled automatically for all users, and users are not auto-enrolled; each user enrolls themselves (it is *not* something requested from Snowflake Support).
- MFA can be **enforced at the role level** via authentication policies (in addition to being something a user opts into individually).

### Data Governance features
| Feature | What it does |
|---|---|
| **Data Classification** | Automatically identifies data objects (and related objects) likely to contain **sensitive data** (PII, etc.) and can tag them accordingly — this is the "identify sensitive data objects" answer, distinct from... |
| **Object Tagging** | A general labeling mechanism (tags can be assigned manually or via classification results) used for tracking/governance and can drive tag-based masking policies |
| **Row Access Policies** | Filter which *rows* a query returns based on the querying role/context |
| **Column-level Security (masking)** | Controls what *column values* a query sees |
| **`ACCESS_HISTORY`** (Account Usage view) | Tracks **what data was read and written, by whom, and when** — including which source columns/objects fed into which target objects (lineage); supports **compliance auditing** use cases. Available for **365 days** of history. |

---

## 12. Metadata Views: ACCOUNT_USAGE vs. INFORMATION_SCHEMA

| Dimension | `SNOWFLAKE.ACCOUNT_USAGE` (shared DB) | `INFORMATION_SCHEMA` (per-database) |
|---|---|---|
| Scope | Entire **account**, across all databases | Just the **current database** it lives in (read-only, auto-created in every DB) |
| Includes dropped objects? | **Yes** | No |
| Retention of historical/usage data | **365 days** (1 year) | Much shorter — roughly **7 days to 6 months** depending on the view/table function |
| Data latency | **Not real-time** — most views ~45 min–3 hrs (many commonly ~2 hrs) behind actual events | **No latency** — real-time |
| Access | Requires being granted access to the `SNOWFLAKE` shared database's roles (`OBJECT_VIEWER`, `USAGE_VIEWER`, `GOVERNANCE_VIEWER`, `SECURITY_VIEWER`) or `IMPORTED PRIVILEGES` | Automatically visible based on your role's privileges on the underlying objects |
| Best for | Long-term trend analysis, compliance history, auditing beyond a few days back | Near-real-time / alerting use cases where a couple hours of lag is unacceptable |

**INFORMATION_SCHEMA contains**: views describing objects in that specific database, **and** table functions that surface historical/usage-style data (e.g. `LOGIN_HISTORY()`, `QUERY_HISTORY()`) scoped more broadly across the account for a limited retention window — it does *not* contain views for objects across the whole account (that's ACCOUNT_USAGE's job).

**Storage monitoring**:
- **Account-level** storage usage: Snowsight UI under **Account → Billing & Usage**, or query `SNOWFLAKE.ACCOUNT_USAGE.STORAGE_USAGE`.
- **Per-table** storage: `SNOWFLAKE.ACCOUNT_USAGE.TABLE_STORAGE_METRICS` (account-wide, ~90 min latency, includes Fail-safe/Time-Travel/clone-retained bytes breakdown) and equivalent table-storage detail is also reachable via `INFORMATION_SCHEMA` for objects in the current database.

---

## 13. Architecture & Cloud Services Layer

Snowflake's three-layer architecture:
1. **Storage layer** — centralized, compressed, columnar storage (shared across all compute).
2. **Compute layer** — virtual warehouses; independent, isolated MPP clusters that eliminate resource contention between workloads.
3. **Cloud Services layer** — the "brain": handles **authentication & access control**, **metadata management**, **query parsing & optimization**, infrastructure management, and more. (It does **not** handle actual query *execution* or data storage — those belong to compute and storage layers respectively.)

Overall description: Snowflake is a **multi-cluster, shared-data architecture** built on a central repository with **massively parallel processing (MPP)**.

---

## 14. Serverless Features & Miscellaneous Billing

- **Serverless features** (e.g., Snowpipe, some serverless tasks, automatic clustering) are billed **per second**, multiplied by an **automatically determined sizing** for the specific job — you don't pick or control the compute size directly for these.
- **Continuous Data Protection (CDP)** charges (Time Travel + Fail-safe combined storage) are maximized at **24 hours for a temporary table** (no Fail-safe, default 1-day Time Travel — and transient tables behave similarly with 0 Fail-safe).

---

## 15. Quick Command Cheat-Sheet ("what command/keyword does X" questions)

| If the question asks about... | The answer is... |
|---|---|
| Downloading files from a stage to local machine | **`GET`** |
| Uploading files from local machine to a stage | **`PUT`** |
| Loading/unloading between a stage and a table | **`COPY INTO`** |
| Viewing files currently in a stage | **`LIST`** |
| Applying/setting a masking policy on a column | `ALTER TABLE/VIEW ... MODIFY COLUMN ... **SET MASKING POLICY**` |
| Removing a masking policy from a column | `... **UNSET MASKING POLICY**` |
| Updating an existing masking policy's logic | **`ALTER MASKING POLICY ... SET BODY`** |
| Viewing the *contents* of a network policy's allow/block lists | **`DESCRIBE NETWORK POLICY`** (not `SHOW NETWORK POLICIES`, which only gives counts) |
| Removing a role grant from a user or role | **`REVOKE ROLE`** |
| Restoring a dropped table/schema/database/tag/account/external volume | **`UNDROP <object>`** |
| Restoring a dropped **internal stage** | Not supported by UNDROP — files are purged on drop; **recreate the stage** |
| Validating a COPY load's errors (current session, most recent load) | **`VALIDATE(<table>, JOB_ID => '_last')`** |
| Validating files before loading (no data loaded) | **`VALIDATION_MODE = RETURN_ALL_ERRORS`** (or `RETURN_ERRORS` / `RETURN_n_ROWS`) |
| Finding what data a query/user accessed and when | **`SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY`** |
| Finding account-wide storage usage | **`SNOWFLAKE.ACCOUNT_USAGE.STORAGE_USAGE`** or Snowsight *Billing & Usage* |
| Finding per-table storage detail | **`TABLE_STORAGE_METRICS`** |
| Capping credit consumption | **Resource Monitors** |
| Handling sudden concurrency/queuing spikes | **Multi-cluster warehouse (scale out)** |
| Speeding up one big, complex query | **Resize the warehouse bigger (scale up)** |
| Verifying a warehouse resize has fully completed | `ALTER WAREHOUSE ... **WAIT_FOR_COMPLETION**` |
| Controlling whether a new warehouse starts immediately | **`INITIALLY_SUSPENDED`** on `CREATE WAREHOUSE` |
| Setting an account-wide floor on Time Travel retention | **`MIN_DATA_RETENTION_TIME_IN_DAYS`** (ACCOUNTADMIN only) |
| Enabling account-level replication in an organization | **`SYSTEM$GLOBAL_ACCOUNT_SET_PARAMETER(...,'ENABLE_ACCOUNT_DATABASE_REPLICATION',...)`** as **ORGADMIN** |
| Specifying/changing a table's cluster key | `CREATE TABLE ... CLUSTER BY` or `ALTER TABLE ... CLUSTER BY` |
| Adding Search Optimization to a table | `ALTER TABLE ... ADD SEARCH OPTIMIZATION` |
| Converting semi-structured data to rows | **`FLATTEN`** (used with `LATERAL`) |
| CDC on a table (inserts/updates/deletes) | **Standard stream** |
| CDC on a table (inserts only, more performant) | **Append-only stream** |
| CDC on files behind an external table | **Insert-only stream** |

---

## Sources
This reference was checked against Snowflake's official documentation (docs.snowflake.com) across: warehouses-multicluster, warehouses-considerations, account-usage, data-time-travel, security-column-ddm-use / security-column-intro, alter-masking-policy, network-policies / show-network-policies / alter-network-policy, account-replication-config, streams-intro / create-stream, intro-editions, views-secure, flatten, copy-into-table, drop-stage / undrop, and related pages, plus cross-checks against several independent SnowPro Core study summaries where documentation didn't spell out an exam-style phrasing directly. Snowflake's docs are updated frequently — for anything you plan to rely on operationally (not just for the exam), re-check the live docs page.

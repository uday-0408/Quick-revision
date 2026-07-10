# SnowPro Core — Parameters, Variables & Functions Cheat Sheet

> Defaults verified against current Snowflake docs (Jul 2026). Bold = common exam trap.

---

## 1. Warehouse Parameters (CREATE/ALTER WAREHOUSE)

| Parameter | Values / Range | Default | Notes |
|---|---|---|---|
| `WAREHOUSE_SIZE` | XS→6XL | XSMALL | Credit cost doubles per size step |
| `WAREHOUSE_TYPE` | STANDARD / SNOWPARK-OPTIMIZED / ADAPTIVE | STANDARD | Snowpark-Optimized = 1.5x credits, more memory/CPU per node |
| `AUTO_SUSPEND` | seconds, NULL=never | **600** (10 min) | Background suspend check runs ~every 30s — not precise |
| `AUTO_RESUME` | TRUE/FALSE | TRUE | Only triggers when **entire** warehouse (all clusters) suspended |
| `INITIALLY_SUSPENDED` | TRUE/FALSE | FALSE | Only valid at creation |
| `MIN_CLUSTER_COUNT` / `MAX_CLUSTER_COUNT` | 1–10 | 1 / 1 | Equal → **Maximized mode**; Min<Max → **Auto-scale mode** |
| `SCALING_POLICY` | STANDARD / ECONOMY | STANDARD | STANDARD favors starting new clusters fast; ECONOMY favors keeping existing ones full |
| `RESOURCE_MONITOR` | monitor name | none | |
| `ENABLE_QUERY_ACCELERATION` | TRUE/FALSE | FALSE | Requires **Enterprise Edition+** |
| `QUERY_ACCELERATION_MAX_SCALE_FACTOR` | 0–100 | **8** (explicit enable); **2** (auto-enabled Gen2/multi-cluster) | 0 = no upper bound |
| `MAX_CONCURRENCY_LEVEL` | integer | **8** | Object param — concurrency per cluster before queuing |
| `STATEMENT_QUEUED_TIMEOUT_IN_SECONDS` | seconds | 0 (no timeout) | Queued query cancelled after this |
| `STATEMENT_TIMEOUT_IN_SECONDS` | seconds | **172800 (2 days)** | Running query cancelled after this; can set at session/account too |
| `GENERATION` | '1' / '2' | account default | Gen2 = newer compute platform |

**Interactive Warehouses (newer object):** min `AUTO_SUSPEND` = **86400s (24h)**; created SUSPENDED by default; only query interactive tables.

**Multi-cluster gotcha:** auto-suspend for multi-cluster only fires when **minimum** cluster count is running and idle.

---

## 2. Session Parameters

| Parameter | Default | Notes |
|---|---|---|
| `TIMEZONE` | America/Los_Angeles | |
| `WEEK_START` | 0 (uses ISO/locale default) | 1=Monday...7=Sunday, 0=legacy Sunday |
| `WEEK_OF_YEAR_POLICY` | 0 | 0=ISO-like, 1=US style |
| `DATE_INPUT_FORMAT` / `DATE_OUTPUT_FORMAT` | AUTO / YYYY-MM-DD | |
| `TIMESTAMP_OUTPUT_FORMAT` | YYYY-MM-DD HH24:MI:SS.FF3 TZH:TZM | |
| `QUERY_TAG` | '' | Free text, shows in QUERY_HISTORY |
| `STATEMENT_TIMEOUT_IN_SECONDS` | 172800 (2 days) | Can override at session |
| `LOCK_TIMEOUT` | 43200 (12h) | 0 = fail immediately if locked |
| `AUTOCOMMIT` | TRUE | |
| `USE_CACHED_RESULT` | TRUE | Disables 24h result cache reuse if FALSE |
| `JSON_INDENT` | 2 | |
| `MULTI_STATEMENT_COUNT` | 1 | 0 = unlimited statements per batch |
| `ROWS_PER_RESULTSET` | 0 (no limit) | |
| `CLIENT_SESSION_KEEP_ALIVE` | FALSE | Keeps session alive w/o token refresh |

---

## 3. Database / Schema / Table Object Parameters

| Parameter | Default | Max (edition) | Notes |
|---|---|---|---|
| `DATA_RETENTION_TIME_IN_DAYS` | **1** | Standard: **0–1**; Enterprise+: **0–90** | Controls Time Travel window; 0 = disables TT for that object |
| `MIN_DATA_RETENTION_TIME_IN_DAYS` | 0 | account-level only, ACCOUNTADMIN | Effective retention = `MAX(DATA_RETENTION_TIME_IN_DAYS, MIN_DATA_RETENTION_TIME_IN_DAYS)` |
| `MAX_DATA_EXTENSION_TIME_IN_DAYS` | **14** | up to 90 | Extends retention automatically to avoid **stale streams** |
| Fail-safe period | **7 days fixed** | not configurable | Permanent tables only; **no direct query access**, Snowflake Support recovery only |
| Transient table retention | 0 or 1 only | max 1 regardless of account setting | No Fail-safe for transient/temporary tables |
| Temporary table retention | 0 or 1 | session-scoped, dropped at session end | No Fail-safe |
| `CHANGE_TRACKING` | FALSE | table param | Must be TRUE (or a stream exists) to enable CDC via streams |
| `DEFAULT_DDL_COLLATION` | '' | | |
| `SUSPEND_TASK_AFTER_NUM_FAILURES` | 10 | | |

**Trap:** Database created from a **share** always has `DATA_RETENTION_TIME_IN_DAYS = 0` (read-only, no Time Travel).

**Sequence: TT window ends → Fail-safe (7d, permanent tables only) → permanent purge.**

---

## 4. Account-Level Parameters (Security-relevant)

| Parameter | Purpose |
|---|---|
| `NETWORK_POLICY` | Attach IP allow/block list at account/user level |
| `ALLOW_ID_TOKEN` | SSO token caching for drivers |
| `PREVENT_UNLOAD_TO_INTERNAL_STAGES` | Block unload to internal named/user/table stages |
| `REQUIRE_STORAGE_INTEGRATION_FOR_STAGE_CREATION` | Force storage integration (no raw creds) on external stage creation |
| `REQUIRE_STORAGE_INTEGRATION_FOR_STAGE_OPERATION` | Same, enforced at operation time |
| `SAML_IDENTITY_PROVIDER` | Configure SAML SSO |
| `SSO_LOGIN_PAGE` | Enable/disable SSO login redirect |
| `MIN_DATA_RETENTION_TIME_IN_DAYS` | Account-wide floor for Time Travel |

---

## 5. Context Functions

| Function | Returns |
|---|---|
| `CURRENT_ACCOUNT()` | Account identifier |
| `CURRENT_ROLE()` | Active primary role in session |
| `CURRENT_AVAILABLE_ROLES()` | All roles grantable to current session |
| `CURRENT_SECONDARY_ROLES()` | Secondary roles active in session |
| `CURRENT_USER()` | Logged-in user |
| `CURRENT_SESSION()` | Session ID |
| `CURRENT_WAREHOUSE()` | Active warehouse |
| `CURRENT_DATABASE()` / `CURRENT_SCHEMA()` | Active DB/schema |
| `CURRENT_REGION()` | Cloud region of account |
| `CURRENT_TIMESTAMP()` / `CURRENT_DATE()` / `CURRENT_TIME()` | Clock values |
| `CURRENT_STATEMENT()` | SQL text of running statement |
| `CURRENT_CLIENT()` | Client driver/version |
| `CURRENT_VERSION()` | Snowflake release version |
| `CURRENT_IP_ADDRESS()` | Client IP |
| `LAST_QUERY_ID()` | Query ID of last executed statement in session |
| `INVOKER_ROLE()` | Role of caller of a stored proc/UDF (caller's rights context) |
| `INVOKER_SHARE()` | Share name if query run through a share |
| `IS_ROLE_IN_SESSION()` | TRUE/FALSE — checks role hierarchy incl. secondary roles |
| `IS_GRANTED_TO_INVOKER_ROLE()` | Checks privilege propagation for invoker's rights |

**Trap:** Secure UDFs/views use `INVOKER_ROLE()`/`INVOKER_SHARE()` correctly only because secure objects **hide the definition from unauthorized roles** — don't confuse with owner's rights procs (which ignore caller context).

---

## 6. System Functions (SYSTEM$...)

| Function | Purpose |
|---|---|
| `SYSTEM$STREAM_HAS_DATA('stream')` | TRUE if stream has unconsumed changes |
| `SYSTEM$PIPE_STATUS('pipe')` | Pipe execution state, pending file count |
| `SYSTEM$TASK_DEPENDENTS_ENABLE('task')` | Recursively resumes a task tree |
| `SYSTEM$CLUSTERING_INFORMATION('table')` | Clustering depth/ratio stats |
| `SYSTEM$CLUSTERING_DEPTH('table')` | Avg clustering depth number |
| `SYSTEM$ESTIMATE_QUERY_ACCELERATION('query_id')` | Predicts QAS benefit per scale factor (query must be ≤14 days old) |
| `SYSTEM$WAIT(seconds)` | Pause execution (testing) |
| `SYSTEM$CANCEL_QUERY('query_id')` | Cancel a running query |
| `SYSTEM$ABORT_SESSION(session_id)` | Kill a session |
| `SYSTEM$TYPEOF(expr)` | Returns data type |
| `SYSTEM$GET_LINEAGE(...)` | Programmatic access to object lineage graph |
| `SYSTEM$USER_TASK_CANCEL_ONGOING_EXECUTIONS('task')` | Cancels running task instance |
| `SYSTEM$SET_RETURN_VALUE(value)` | Used in stored proc to set task return value |
| `SYSTEM$GET_PREDECESSOR_RETURN_VALUE()` | Reads prior task's return value in a DAG |
| `SYSTEM$ALLOWLIST()` | Returns network allowlist info (for PrivateLink/firewall config) |

---

## 7. File Format Options (CREATE FILE FORMAT / COPY INTO)

| Option | Default | Notes |
|---|---|---|
| `FIELD_DELIMITER` | ',' | |
| `RECORD_DELIMITER` | '\n' | |
| `SKIP_HEADER` | 0 | Rows skipped |
| `FIELD_OPTIONALLY_ENCLOSED_BY` | NONE | e.g. `'"'` |
| `NULL_IF` | ('\\N') | List of strings → NULL |
| `EMPTY_FIELD_AS_NULL` | TRUE | |
| `COMPRESSION` | AUTO | Also: GZIP, BZ2, NONE, ZSTD, etc. |
| `TRIM_SPACE` | FALSE | |
| `ERROR_ON_COLUMN_COUNT_MISMATCH` | TRUE | |
| `STRIP_OUTER_ARRAY` | FALSE | JSON — strips outer `[ ]` so each element loads as a row |
| `STRIP_NULL_VALUES` | FALSE | JSON |
| `PARSE_HEADER` | FALSE | CSV — auto-derive column names (used with `INFER_SCHEMA`) |

---

## 8. COPY INTO Load Options

| Option | Values | Default | Notes |
|---|---|---|---|
| `ON_ERROR` | CONTINUE / SKIP_FILE / SKIP_FILE_\<n\> / SKIP_FILE_\<n\>% / ABORT_STATEMENT | **ABORT_STATEMENT** | Whole COPY fails on first error by default |
| `SIZE_LIMIT` | bytes | none | Caps bytes loaded per COPY run |
| `PURGE` | TRUE/FALSE | FALSE | Deletes staged files after successful load |
| `MATCH_BY_COLUMN_NAME` | CASE_SENSITIVE / CASE_INSENSITIVE / NONE | NONE | For semi-structured → column match by name |
| `VALIDATION_MODE` | RETURN_ERRORS / RETURN_\<n\>_ROWS / RETURN_ALL_ERRORS | none | Dry-run, no actual load |
| `FORCE` | TRUE/FALSE | FALSE | Reloads files even if already loaded (bypasses load metadata check, 64-day lookback) |
| `LOAD_UNCERTAIN_FILES` | TRUE/FALSE | FALSE | |
| `RETURN_FAILED_ONLY` | TRUE/FALSE | FALSE | |

**Trap:** Snowflake tracks loaded file history for **64 days** to prevent duplicate loads (same file name+path+ETag) — this is why re-running COPY normally is a no-op unless `FORCE=TRUE`.

---

## 9. Stage Parameters

| Parameter | Notes |
|---|---|
| `URL` | Cloud storage path (external stage) |
| `STORAGE_INTEGRATION` | Preferred over raw credentials |
| `CREDENTIALS` | Raw key/secret (discouraged; blocked if `REQUIRE_STORAGE_INTEGRATION_FOR_STAGE_CREATION=TRUE`) |
| `ENCRYPTION` | SSE-KMS, client-side, etc. |
| `DIRECTORY = ( ENABLE = TRUE )` | Enables directory table (file metadata queries) |
| Internal stage types | User stage (`@~`), table stage (`@%table`), named stage (`@stage`) |

---

## 10. Snowpipe Parameters

| Parameter | Notes |
|---|---|
| `AUTO_INGEST` | TRUE = event-driven via cloud notification (SNS/EventGrid/Pub-Sub) |
| `ERROR_INTEGRATION` | Routes load errors to external system |
| `AWS_SNS_TOPIC` | For AUTO_INGEST on S3 |
| Pipe billing | Serverless compute, billed **per-file/second**, not by warehouse |
| `SYSTEM$PIPE_STATUS` | Check `pendingFileCount`, `executionState` |

**Trap:** Snowpipe is **serverless** — doesn't use your virtual warehouse. REST-API-triggered Snowpipe uses `insertFiles` call, not `AUTO_INGEST`.

---

## 11. Stream Parameters & Behavior

| Parameter | Default | Notes |
|---|---|---|
| `APPEND_ONLY` | FALSE | TRUE = only tracks inserts (faster, ignores updates/deletes) |
| `SHOW_INITIAL_ROWS` | FALSE | TRUE = first read includes existing rows as inserts |
| Stream staleness | tied to `MAX_DATA_EXTENSION_TIME_IN_DAYS` (default 14d) | Stream becomes **stale** and unusable if offset falls outside retention window |
| Stream advances offset | only on **DML transaction that reads the stream and commits** | Reading in a rolled-back txn does not advance offset |
| Streams on views | Supported (incl. secure views) | Requires `CHANGE_TRACKING = TRUE` on underlying tables |
| Stream metadata columns | `METADATA$ACTION`, `METADATA$ISUPDATE`, `METADATA$ROW_ID` | Updates appear as paired DELETE+INSERT with `ISUPDATE=TRUE` |

---

## 12. Task Parameters

| Parameter | Default | Notes |
|---|---|---|
| `WAREHOUSE` | none | Omit → uses **serverless compute** |
| `USER_TASK_MANAGED_INITIAL_SIZE` | XSMALL | Initial size for serverless task |
| `SCHEDULE` | none | CRON or `MINUTE` interval |
| `ALLOW_OVERLAPPING_EXECUTION` | FALSE | |
| `USER_TASK_TIMEOUT_MS` | **3600000 (1hr)** | Per-run timeout |
| `SUSPEND_TASK_AFTER_NUM_FAILURES` | 10 | Auto-suspends task tree root after N consecutive failures |
| `CONFIG` | JSON string | Read via `SYSTEM$GET_TASK_GRAPH_CONFIG` |
| Root task | Must be resumed with `ALTER TASK ... RESUME` (children resumed automatically when root runs) | New/modified tasks default to **SUSPENDED** |
| Task DAG depth | max **100 tasks** per DAG (as of recent limits — verify current docs) | |

**Trap:** A **stream + task** combo is Snowflake's native "trigger only if new data" CDC pattern — task checks `SYSTEM$STREAM_HAS_DATA()` in a `WHEN` clause before running.

---

## 13. Alerts

| Component | Notes |
|---|---|
| `CREATE ALERT ... WAREHOUSE = ... SCHEDULE = ... IF (EXISTS(...)) THEN ...` | Condition + action, needs its own warehouse (or serverless) |
| Alert states | `STARTED`, `SUCCEEDED`, `FAILED`, `SUSPENDED` |
| New alerts | Created **SUSPENDED** by default — must `ALTER ALERT ... RESUME` |
| `SYSTEM$STREAM_HAS_DATA` | Common condition check inside alert body |
| Notification integration | Needed to send email/webhook from alert action |

---

## 14. Resource Monitors

| Parameter | Values | Notes |
|---|---|---|
| `CREDIT_QUOTA` | integer | Credits per frequency interval |
| `FREQUENCY` | DAILY / WEEKLY / MONTHLY / YEARLY / **NEVER** | NEVER = quota never resets |
| `START_TIMESTAMP` / `END_TIMESTAMP` | | |
| `TRIGGERS` | `ON <percent> PERCENT DO {NOTIFY \| SUSPEND \| SUSPEND_IMMEDIATE}` | Multiple triggers allowed |
| `SUSPEND` | Lets running queries finish, blocks new ones | |
| `SUSPEND_IMMEDIATE` | Cancels running queries too | |
| Scope | Can attach to **account** (only one at account level) or multiple **warehouses** | Only `ACCOUNTADMIN` (or delegated `MONITOR`/`MODIFY` privilege) manages |

---

## 15. Masking / Row Access Policy — Quick Reference

| Concept | Masking Policy | Row Access Policy |
|---|---|---|
| Applies to | Column | Row (via WHERE-like filter) |
| Object types | Tables, views, external tables | Tables, views |
| Language | SQL expression, single input col type | SQL expression, uses `CURRENT_ROLE()` etc. |
| Multiple policies same column | Not allowed (1 masking policy per column) | Multiple row access policies not stackable on same table (only 1 active) |
| Applies at | Query time (unmask if privileged) | Query time (filters rows) |
| Tag-based | Can attach masking policy to a **tag** → auto-applies to all tagged columns | N/A |

**Object tagging trap:** Tag **inheritance** (child object inherits tag from schema/db/table if not overridden) vs tag **propagation** through masking policy association — inheritance is about the tag *value*, propagation is about the *masking behavior* riding along with it.

---

## 16. Network Policy vs Authentication Policy

| | Network Policy | Authentication Policy |
|---|---|---|
| Controls | IP allow/block list | Auth methods allowed (password, MFA, SSO, etc.), client types |
| Attach level | Account, user, security integration | Account, user |
| Multiple active | One per user/account (most specific wins: user > account) | Same |

---

## 17. RBAC — Default System Roles (hierarchy, top→bottom)

```
ACCOUNTADMIN
   ├── SECURITYADMIN  → USERADMIN
   ├── SYSADMIN        (owns all warehouse/db/schema objects if best practice followed)
ORGADMIN (separate branch — manages orgs/accounts, not account objects)
PUBLIC (implicitly granted to every user/role — bottom of every hierarchy)
```
- `SECURITYADMIN`: manage grants + inherits `USERADMIN`
- `USERADMIN`: create/manage users & roles
- `SYSADMIN`: create warehouses/databases/schemas (best practice: custom roles granted to SYSADMIN)
- `GLOBALORGADMIN` / `ORGADMIN`: org-level, billing, account creation — **outside** normal account RBAC tree
- **Trap:** `ACCOUNTADMIN` is NOT automatically the owner of objects created by other roles — ownership follows the **creating role**, not a global admin.

## 18. Future Grants
```sql
GRANT SELECT ON FUTURE TABLES IN SCHEMA mydb.myschema TO ROLE analyst;
```
- Applies only to objects **created after** the grant — does not retroactively grant on existing objects.
- Can be set at database level (`IN DATABASE`) or schema level (`IN SCHEMA`).

## 19. Data Sharing — Terms

| Term | Meaning |
|---|---|
| Share | Named object bundling grants on DB objects, given to another account |
| Direct Share | Provider explicitly adds consumer account |
| Private Listing | Provider publishes to specific consumer(s) via Marketplace mechanics |
| Public Listing | Published to Snowflake Marketplace, discoverable by all |
| Reader Account | Provider-managed account for consumers **without** their own Snowflake account |
| Consumer database | Read-only, zero-copy — no storage cost to consumer, no Time Travel (retention=0) |

---

## 20. Semantic Views (Cortex Analyst / BI layer)
- `CREATE SEMANTIC VIEW` defines **tables, relationships, facts, dimensions, metrics** in one object.
- Used for natural-language / BI-tool consistent metric definitions across an org.
- Does **not** store data itself — it's metadata over existing tables.

---

## 21. Trust Center
- Components: **Scanners** (e.g., malware, MFA-not-enrolled users, unencrypted egress), **Findings**, **Notifications**, **Scanner Packages**.
- Scanner types: **schedule-based** (periodic) vs **event-driven** (real-time trigger, subset of scanners).
- Requires the `SNOWFLAKE.TRUST_CENTER` app; `ACCOUNTADMIN` manages by default.

---

## 22. Data Lineage / OpenLineage
- `SYSTEM$GET_LINEAGE()` + Snowsight lineage graph — shows object-to-object and column-level lineage.
- OpenLineage integration lets external orchestrators (Airflow etc.) emit lineage events Snowflake ingests.
- Lineage tracked automatically for: CTAS, INSERT INTO...SELECT, views, streams, tasks, COPY INTO, CLONE.

---

## 23. Cheat Numbers (fast recall table)

| Value | What |
|---|---|
| 600s (10 min) | Default `AUTO_SUSPEND` |
| 86400s (24h) | Min `AUTO_SUSPEND` for Interactive Warehouses |
| 8 | Default `MAX_CONCURRENCY_LEVEL`; default `QUERY_ACCELERATION_MAX_SCALE_FACTOR` (explicit) |
| 2 | Default QAS scale factor when auto-enabled on Gen2/multi-cluster |
| 172800s (2 days) | Default `STATEMENT_TIMEOUT_IN_SECONDS` |
| 1 day | Default `DATA_RETENTION_TIME_IN_DAYS` |
| 0–1 day | Time Travel range, Standard Edition |
| 0–90 days | Time Travel range, Enterprise Edition+ |
| 7 days | Fail-safe (fixed, not configurable, permanent tables only) |
| 14 days | Default `MAX_DATA_EXTENSION_TIME_IN_DAYS` (stream staleness window) |
| 64 days | COPY INTO load-history dedup lookback |
| 1 hour (3600000ms) | Default `USER_TASK_TIMEOUT_MS` |
| 10 | Default `SUSPEND_TASK_AFTER_NUM_FAILURES` |
| 24h | Default `LOCK_TIMEOUT` (43200s = 12h — check current docs, varies) |
| 256 chars | Max tag value length |

---

## 24. Common Exam Traps Summary
- `AUTO_SUSPEND=0` or `NULL` → warehouse **never** auto-suspends (careful, exam likes to test this as "always suspends immediately" — wrong).
- Fail-safe has **zero query access** — always a support ticket, always 7 days, never configurable, never applies to transient/temp tables.
- `ENABLE_QUERY_ACCELERATION` requires **Enterprise Edition or higher** — Standard Edition accounts cannot use QAS at all.
- Time Travel max days is an **edition** limit, not a warehouse limit.
- `FORCE=TRUE` in COPY INTO bypasses dedup — without it, re-running COPY on the same file is a silent no-op (not an error).
- Streams don't store data — they store a **change-tracking offset/pointer**; if the underlying table's retention window moves past that offset, the stream goes **stale** and must be recreated.
- Secondary roles are activated with `USE SECONDARY ROLES ALL` — privileges from secondary roles apply, but **object ownership** always resides with primary role context at creation time.
- Resource Monitor `SUSPEND` ≠ `SUSPEND_IMMEDIATE`: the former lets in-flight queries finish; only the latter cancels them.
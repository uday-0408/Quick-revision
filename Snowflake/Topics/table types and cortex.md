# Dynamic Tables, Table Types & Cortex AI — SnowPro Core Study Guide

Sourced only from official Snowflake docs (docs.snowflake.com). Built for exam gotchas, not general reading.

---

## 1. Dynamic Tables

### What it is
A dynamic table materializes a `SELECT` query's results and keeps them fresh automatically. You write the query + a freshness target (`TARGET_LAG`); Snowflake figures out scheduling, dependency order, and incremental vs. full refresh.

```sql
CREATE OR REPLACE DYNAMIC TABLE dt_orders
    TARGET_LAG = '10 minutes'
    WAREHOUSE = transform_wh
    REFRESH_MODE = INCREMENTAL
AS
    SELECT order_id, customer_id, order_date,
           TRIM(UPPER(product_name)) AS product_name,
           quantity, unit_price,
           quantity * unit_price AS line_total,
           order_status
    FROM raw_orders
    WHERE order_status != 'returned';
```

### Key concepts
| Concept | Detail |
|---|---|
| **TARGET_LAG** | Max acceptable staleness vs. base table. NOT a guarantee — actual lag can exceed it if refreshes take longer than the window. Minimum = **1 minute**. |
| **TARGET_LAG = DOWNSTREAM** | Table only refreshes when a downstream dependent needs it. **Gotcha:** if the *last* table in a chain is set to DOWNSTREAM, nothing ever triggers a refresh — data never updates. |
| **REFRESH_MODE** | `INCREMENTAL` (only changed rows), `FULL` (recompute everything), `AUTO` (Snowflake picks at creation time — Snowflake recommends AUTO only during dev, not prod, since behavior can change across releases), `ADAPTIVE` (incremental by default, auto-reinitializes on large upstream changes, preview), `CUSTOM_INCREMENTAL` (you supply `MERGE INTO SELF` / `INSERT INTO SELF` DML). |
| **Refresh atomicity** | New results apply atomically — readers never see a partial refresh. |
| **Pipelines** | Multiple dynamic tables reading from each other form a DAG. Snowflake infers dependencies automatically and picks a consistent snapshot timestamp — no manual orchestration. |

### Refresh mechanics — exam traps
- **Change tracking is mandatory** for incremental refresh. Snowflake tries to auto-enable it on base objects; if the creating role lacks `OWNERSHIP` on those base objects, it fails silently at the eligibility level — check the `change_tracking` column via `SHOW TABLES`/`SHOW VIEWS`.
- Change tracking requires **non-zero Time Travel retention** on all underlying objects.
- **AUTO mode resolves to FULL at creation** if the query isn't incrementally eligible — the `CREATE` statement itself never tells you which mode was picked. Confirm with:
```sql
SHOW DYNAMIC TABLES;  -- check refresh_mode and scheduling_state columns
```
- If incremental refresh later becomes impossible (e.g., a policy change on a base table), it **fails silently** unless you built your own alerting — no built-in notification.
- Query construct restrictions for incremental refresh: no stored procedures, no external functions, no UDTFs outside lateral joins, no volatile UDFs. Structured data types (structured OBJECT/ARRAY, MAP) are unsupported in *both* incremental and full refresh; geospatial types are full-refresh only.
- **UDFs default to VOLATILE** — must be explicitly `IMMUTABLE` to support incremental refresh, and Snowflake does NOT validate that the function is actually deterministic (garbage in, garbage out is on you).
- Cortex AI functions ARE supported in incremental-refresh dynamic tables, but **only in the SELECT clause** — not in `WHERE`, `GROUP BY`, `HAVING`, or `QUALIFY`.
- Masking/row access policies on the dynamic table itself don't affect its refresh mode. Policies on **base tables** do: `CURRENT_ROLE()`/`IS_ROLE_IN_SESSION()`-based policies still allow incremental; anything else (INFORMATION_SCHEMA views, table lookups) forces full refresh, and policy changes trigger reinitialization.

### Hard limitations (frequently tested)
- **Read-only.** No `INSERT`, `UPDATE`, `DELETE`, or `TRUNCATE`. It's fully derived from its query.
- **No MERGE support** — declarative model only (except `CUSTOM_INCREMENTAL`'s `REFRESH USING`).
- **Cannot create a temporary dynamic table.** Transient dynamic tables ARE allowed.
- Max **50,000** dynamic tables per account.
- Cannot use **secondary roles** — refreshes execute as the owner role.
- Cannot source from: **streams, materialized views, external tables, directory tables**, or a shared dynamic table/shared secure view referencing an upstream dynamic table.
- Cannot use dynamic SQL (session variables, unbound anonymous-block variables) in the definition.
- UDTF-sourced SELECT blocks must explicitly list columns — no `SELECT *` from a UDTF.
- Dynamic table queries are **not tracked in ACCESS_HISTORY**.
- Can't set `DATA_RETENTION_TIME_IN_DAYS = 0` if the base table is a shared table.
- No serverless option — always runs on a warehouse you provide.
- Schema is fixed at creation — you can't `ALTER` its columns after the fact.

### Table type interaction
- Dynamic tables can be **transient** (`CREATE TRANSIENT DYNAMIC TABLE` or clone to transient) — good for high-throughput tables where 7-day fail-safe storage cost isn't worth it.
- Default fail-safe retention for permanent dynamic tables = 7 days, same as regular permanent tables.
- Cloned dynamic tables are **suspended by default** (shows as `CLONED_AUTO_SUSPENDED` in `DYNAMIC_TABLE_GRAPH_HISTORY`); downstream tables show `UPSTREAM_CLONED_AUTO_SUSPENDED`.
- You cannot clone dynamic Iceberg tables; cloning a DB/schema containing one does not carry it over either.

### When NOT to use dynamic tables (decision-guide gotchas)
| Need | Use instead |
|---|---|
| Direct row modification (e.g., GDPR erasure) | Streams + Tasks with a standard table |
| Stored procedures / external function calls / conditional branching | Streams + Tasks |
| Freshness < 60 seconds guaranteed | Materialized views (always current) or Streams + Tasks |
| Query performance on a single base table, always-current | Materialized views |

### Cost model (3 buckets)
1. **Warehouse compute** — runs each refresh query.
2. **Cloud Services** — compiles refresh query, tracks dependencies, monitors changes, coordinates scheduling (shorter TARGET_LAG = more scheduling overhead).
3. **Storage** — micro-partitions added/replaced/removed each refresh + Time Travel retention.

---

## 2. Temporary, Transient & Permanent Tables

### The core comparison table (memorize this)

| Attribute | Permanent (default) | Transient | Temporary |
|---|---|---|---|
| Fail-safe | 7 days (fixed, non-configurable) | **None** | **None** |
| Time Travel max | 0–90 days (Enterprise+); 0–1 day on Standard | 0 or 1 day only | 0 or 1 day only |
| Persists across sessions? | Yes | Yes, until dropped | **No** — dropped when session ends |
| Visible to other users/sessions? | Yes | Yes (with privileges) | **No** — session-private |
| Can share name with existing table? | No (must be unique in schema) | No | **Yes** — temp table hides the permanent/transient table of the same name for that session |
| Storage cost | Highest (active + time travel + fail-safe) | Active + limited time travel, no fail-safe | Active + limited time travel, no fail-safe |
| Convertible after creation? | — | **No** — can't convert transient → other type post-creation | — |

### Definitions & syntax
```sql
CREATE [ OR REPLACE ] [ TEMPORARY | TRANSIENT ] TABLE <name> (...);
```
Default (no keyword) = **permanent**.

### Gotchas — this is where the exam bites
- **A Time Travel retention of 0 for a permanent table still enters Fail-safe immediately upon drop** — Fail-safe is not optional/configurable for permanent tables, only Time Travel is.
- Temporary tables *can* have Time Travel retention of 1 day, but the effective window is **whichever is shorter: 24 hours, or the remaining session length** — because the table is purged the instant the session ends regardless of the retention setting.
- A **long-running Time Travel query delays purge** of temporary/transient tables (and delays the fail-safe transition for permanent tables) until that query completes.
- Transient tables have **no Fail-safe**, so once the Time Travel retention window passes, data is genuinely gone — not recoverable by you or by Snowflake support. Use only for reproducible/replaceable data.
- You **cannot ALTER a table's type**. To move a permanent table to transient, the documented pattern is clone + swap + drop:
```sql
CREATE OR REPLACE TRANSIENT TABLE customer_transient
    CLONE customer COPY GRANTS;
ALTER TABLE customer SWAP WITH customer_transient;
DROP TABLE customer_transient;  -- now holds the old permanent table
```
- **Zero-copy clone**: cloning a permanent table to transient shares existing micro-partitions initially — no extra storage until rows are modified in the clone, which then creates partitions exclusive to the clone.
- Transient **databases/schemas** cascade: any table created inside a transient database or schema is transient by default. You cannot make a table permanent inside a transient database.
- **Hybrid tables cannot be temporary or transient**, and by extension can't live in a transient schema/database.
- Naming precedence: if you create a temporary table with the same name as an existing permanent table, the **temp table hides the permanent one** for that session; sessions without the temp table still see the permanent one. This can silently break scripts that assume they're hitting the "real" table.
- A **session ≠ a connection**. A Snowflake session persists until explicit termination or 4-hour inactivity timeout — disconnecting your client does not necessarily end the session, so a temp table can outlive a dropped connection if the session is still technically alive.
- `DATA_RETENTION_TIME_IN_DAYS` set to 0 at the account level via `ACCOUNTADMIN` cascades as the *default* for all new DBs/schemas/tables — can be overridden per-object at creation or later. There's also a `MIN_DATA_RETENTION_TIME_IN_DAYS` account parameter `ACCOUNTADMIN` can use to enforce a floor.
- Best practice per docs: fact/long-lived tables → **always permanent**; short-lived ETL/staging/work tables (< 1 day lifetime) → **transient**, to eliminate Fail-safe cost entirely.

---

## 3. Cortex AI Functions (formerly "LLM Functions")

### Access requirements (exam loves this)
To call **any** Cortex AI function, the role needs:
1. The **`USE AI FUNCTIONS`** account-level privilege, **and**
2. One of the database roles **`SNOWFLAKE.CORTEX_USER`** or **`SNOWFLAKE.CORTEX_USER`**'s narrower sibling **`SNOWFLAKE.AI_FUNCTIONS_USER`**.

`CORTEX_USER` is granted to `ACCOUNTADMIN` by default and must be explicitly propagated to other roles — it is **not granted to PUBLIC** automatically.

### Current function set (this replaced the old `SNOWFLAKE.CORTEX.*` naming)
| Function | Purpose |
|---|---|
| `AI_COMPLETE` | Generate a completion from text/image using a chosen LLM — the general-purpose workhorse |
| `AI_CLASSIFY` | Classify text/images into user-defined categories |
| `AI_FILTER` | Boolean True/False on text or image input — usable directly in `WHERE`/`JOIN...ON` |
| `AI_AGG` | Aggregate insights across many rows of a text column by prompt — **not** limited by context window |
| `AI_EMBED` | Generate embedding vectors (text or image) |
| `AI_EXTRACT` | Pull structured fields out of text/images/documents |
| `AI_SENTIMENT` | Sentiment extraction |
| `AI_SUMMARIZE_AGG` | Summarize across multiple rows — also not context-window-limited |
| `AI_SIMILARITY` | Embedding similarity between two inputs |
| `AI_TRANSCRIBE` | Audio/video → text, with timestamps + speaker info, from a stage |
| `AI_PARSE_DOCUMENT` | OCR mode or LAYOUT mode text extraction from staged documents; can pull embedded images |
| `AI_REDACT` | Redact PII from text |
| `AI_TRANSLATE` | Language translation |
| `SUMMARIZE` (`SNOWFLAKE.CORTEX.SUMMARIZE`) | Legacy-namespaced summary function, still current |

**Helper functions** (reduce failure modes, not generative themselves):
- `TO_FILE` — reference a staged file for use with `AI_COMPLETE` etc.
- `AI_COUNT_TOKENS` — token count for a given model/function before you call it, to avoid limit errors
- `PROMPT` — helps construct structured prompt objects

### Gotchas
- **`COMPLETE (SNOWFLAKE.CORTEX)` is a legacy function slated for deprecation by end of 2026.** Prefer `AI_COMPLETE` for new work — this old-vs-new naming split (`SNOWFLAKE.CORTEX.COMPLETE` vs `AI_COMPLETE`) is a classic exam distractor.
- `COMPLETE`/`AI_COMPLETE` are **stateless** — no memory between calls. A multi-turn "conversation" only works if you pass the entire prior history back in as the `prompt_or_history` array yourself each time. Token cost scales with the whole accumulated history on every round.
- Regional/model availability varies — always described as "select regions," not universal.
- Functions are optimized for **throughput/batch** (e.g., scanning a whole table). For low-latency interactive use, the docs explicitly steer you to the **REST API** (Complete API, Embed API, Agents API) instead of the SQL functions.
- Snowflake does **not** train customer-base LLMs on your Customer Data — a recurring "data governance" trivia point across all Cortex features.
- Cortex AI functions inside dynamic tables: allowed only in `SELECT`, never in filter/grouping clauses (see Dynamic Tables section above) — cross-topic exam trap.

---

## 4. Cortex Analyst

### What it is
A managed service that turns **natural language → SQL** against structured data, using a **semantic model** (or newer **Semantic View**) to bridge business vocabulary and physical schema (e.g., mapping "revenue" → a specific net-revenue expression, "CUST" → "customer").

### Semantic Model vs. Semantic View — the exam-relevant distinction
- **Semantic Model**: legacy approach — a YAML file stored in a **stage**. Still supported for backward compatibility.
- **Semantic View**: newer approach — a first-class **schema-level database object** (`CREATE SEMANTIC VIEW`), queryable directly in a `SELECT` statement, shareable via listings (private/public/organizational). **This is the recommended approach for new work.**
- Semantic views model **facts** (row-level, e.g., individual sale amount), **dimensions**, and **metrics** (aggregations like `SUM`/`AVG`/`COUNT` across rows). "Facts" were called "measures" in earlier Cortex Analyst releases — old term still backward compatible.
- Cortex Analyst reads the semantic view/model definition and generates SQL **against the physical tables directly** — it does not pre-materialize anything itself.

### Privileges — another commonly tested split
- **`SNOWFLAKE.CORTEX_USER`** → grants access to all "Covered AI" features broadly.
- **`SNOWFLAKE.CORTEX_ANALYST_USER`** → narrower, grants access to **Cortex Analyst only**. Use this to scope access down without handing out full Cortex.
- Also required: `SELECT` on underlying tables referenced by the model, `READ` (or `WRITE`) on the stage if using a legacy stage-based YAML model, and `USAGE` on any Cortex Search service the model references.
- Database roles (like `CORTEX_ANALYST_USER`) **cannot be granted directly to a user** — must go through a custom account role first (`GRANT DATABASE ROLE ... TO ROLE ...`, then `GRANT ROLE ... TO USER ...`).

### Data governance points (recurring theme across Cortex)
- Cortex Analyst does **not train on Customer Data**.
- For inference, only the semantic model's metadata (table/column names, types, descriptions) is used to generate SQL — the actual query then runs in **your own virtual warehouse**, inside your governance boundary.
- Fully integrates with existing RBAC — generated/executed SQL still respects your access controls; Cortex Analyst can't read data a role couldn't already see.
- Default backing models: Snowflake-hosted **Mistral and Meta (Llama)** models — data/prompts/metadata never leave Snowflake's boundary by default. You cannot pick a model per query; Snowflake auto-selects at runtime.

### Gotchas
- Cortex Analyst is optimized for **single-turn structured-data Q&A** — it's the "text-to-SQL" specialist. For unstructured document retrieval, that's Cortex Search's job (see below); for combining both in one conversational agent, that's Cortex Agents (different feature).
- Semantic models are treated as **metadata**, not data — important for classification/governance questions.
- A single legacy semantic model YAML is generally scoped to model one focused domain — Snowflake recommends starting with a simple star schema rather than dumping the entire warehouse schema into one file.

---

## 5. Cortex Search

### What it is
A fully managed **hybrid search** service (vector + keyword + semantic reranking) over text data — no manual embedding pipeline, vector DB, or index tuning required. Two primary use cases: **RAG** (retrieval for LLM chat apps) and **enterprise search** (search bars over internal docs).

Every query runs three passes automatically:
1. **Vector search** — semantically similar documents
2. **Keyword search** — lexically similar documents
3. **Semantic reranking** — combines/ranks both result sets

### Syntax
```sql
CREATE OR REPLACE CORTEX SEARCH SERVICE transcript_search_service
    ON transcript_text
    ATTRIBUTES region
    WAREHOUSE = cortex_search_wh
    TARGET_LAG = '1 day'
    EMBEDDING_MODEL = 'snowflake-arctic-embed-l-v2.0'
AS (
    SELECT transcript_text, region, agent_id
    FROM support_transcripts
);
```
- `ON <col>` — the single text column that gets embedded/searched.
- `ATTRIBUTES` — extra columns returned alongside results / usable for filtering. **Must be included in the source query** (explicit list or `*`) — a very common setup error.
- Multi-index variant: `TEXT INDEXES col1, col2 VECTOR INDEXES col3 (model='...')` — lets you search/filter across multiple text columns and one vector column in a single service.
- `TARGET_LAG` — same semantics as dynamic tables: how stale the index is allowed to get vs. the base table. **Gotcha:** must be shorter than the source table's `DATA_RETENTION_TIME_IN_DAYS` — if target lag exceeds retention, the service can lose the ability to detect changes and may need to be recreated from scratch.
- `PRIMARY KEY (col, ...)` — must be TEXT type; enables an optimized incremental refresh path, significantly cutting refresh cost/latency. Duplicate key rows are silently ignored in the index.

### Under the hood — internal dependency on dynamic tables
Cortex Search Services are implemented using **internal, automatically generated dynamic tables** to manage refresh. This is why:
- **Change tracking must be enabled on all underlying base objects**, same requirement as dynamic tables.
- You cannot create a Cortex Search service directly on a table from a **shared database** (e.g., Marketplace data) — change tracking can't be enabled on shared data. Workaround: copy to a local table first, enable change tracking there, then build the service on the local copy.
- Errors referencing "Insufficient privileges to operate on base table of Dynamic Table" when creating a search service are really about this hidden internal dynamic table, not something you created directly.

### Privileges
| Action | Privilege needed |
|---|---|
| Create the service | `CREATE CORTEX SEARCH SERVICE` (or `OWNERSHIP`) on the schema, `SELECT` on source table(s)/view(s), `USAGE` on the warehouse |
| Query the service | `USAGE` on the service **and** on its database and schema |
| Suspend/resume via `ALTER` | `OPERATE` privilege on the service |
| Create with SQL | Role needs a role granting Cortex embedding privileges — `CORTEX_USER` works; a narrower `CORTEX_EMBED_USER`-style role covers only embedding-scoped creation |

Services run with **owner's rights**, same security model as most other Snowflake objects — the querying user's own row-level access doesn't automatically restrict search results unless you've built filtering/RLS around it yourself.

### Gotchas
- **`FULL_INDEX_BUILD_INTERVAL_DAYS` is a s
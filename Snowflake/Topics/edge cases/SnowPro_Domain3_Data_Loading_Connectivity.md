# SnowPro Core — Domain 3.0: Data Loading, Unloading & Connectivity
*Compiled and cross-checked against Snowflake official documentation (docs.snowflake.com), July 2026*

---

## Table of Contents
- [3.1 Data Loading & Unloading](#31-data-loading--unloading)
  - [File Formats](#file-formats)
  - [Stages](#stages)
  - [Directory Tables](#directory-tables)
  - [COPY INTO](#copy-into)
  - [Error Handling](#error-handling-copy-into)
- [3.2 Automated Data Ingestion](#32-automated-data-ingestion)
  - [Snowpipe](#snowpipe)
  - [Snowpipe Streaming](#snowpipe-streaming)
  - [Streams](#streams)
  - [Tasks](#tasks)
  - [Dynamic Tables](#dynamic-tables)
  - [Openflow](#openflow-not-tested-until-ga)
- [3.3 Connectors & Integrations](#33-connectors--integrations)
  - [Drivers & Connectors](#drivers--connectors)
  - [Storage Integration](#storage-integration)
  - [API Integration](#api-integration)
  - [Git Integration](#git-integration)
- [Quick Comparison Tables](#quick-comparison-tables)
- [Rapid-Fire Exam Traps](#rapid-fire-exam-traps)

---

## 3.1 Data Loading & Unloading

### File Formats

Snowflake `FILE_FORMAT` objects define **how to parse** a file — they are independent, named, reusable database objects (schema-level).

**Supported types:** `CSV`, `JSON`, `AVRO`, `ORC`, `PARQUET`, `XML`, `CUSTOM`.

```sql
CREATE FILE FORMAT my_csv_format
  TYPE = CSV
  FIELD_DELIMITER = ','
  FIELD_OPTIONALLY_ENCLOSED_BY = '"'
  SKIP_HEADER = 1
  NULL_IF = ('', 'NULL', 'null')
  EMPTY_FIELD_AS_NULL = TRUE
  COMPRESSION = AUTO
  ERROR_ON_COLUMN_COUNT_MISMATCH = TRUE;
```

**Where a file format can be specified (precedence, most specific wins):**
1. Inline in the `COPY INTO` statement (`FILE_FORMAT = (...)`)
2. Attached to a **stage** definition
3. A **named file format** referenced from the stage or COPY statement

> ⚠️ **Gotcha:** A **User Stage** does **not support** setting `FILE_FORMAT` as a stage property — you must specify the file format directly in `COPY INTO`/`PUT` each time.

**Key CSV options often tested:**
| Option | Purpose |
|---|---|
| `SKIP_HEADER` | Skip N header rows |
| `FIELD_OPTIONALLY_ENCLOSED_BY` | Quote character |
| `ERROR_ON_COLUMN_COUNT_MISMATCH` | Fail if column count in file ≠ table |
| `PARSE_HEADER` | Auto-detect column names from header row (cannot be combined with `SKIP_HEADER`) |
| `COMPRESSION = AUTO` | Auto-detect compression from file extension |

**Parquet/Avro/ORC gotcha:** `MATCH_BY_COLUMN_NAME` (COPY option) maps semi-structured columns to table columns **by name** instead of position — critical for schema evolution (e.g., a new column added mid-file doesn't break the load). It **cannot** be combined with `VALIDATION_MODE`.

**Performance gotcha:** For very large uncompressed CSV files (>128 MB) following RFC4180, Snowflake can parallel-scan them **only** when `MULTI_LINE = FALSE`, `COMPRESSION = NONE`, and `ON_ERROR` is `ABORT_STATEMENT` or `CONTINUE`.

**Exam traps:**
- File format objects are **schema-level objects** — they need `USAGE` grants like other objects.
- A file format specified inline in `COPY INTO` **overrides** the stage's file format.
- Don't confuse `TYPE = CUSTOM` (arbitrary regex-based parsing) with standard types.

---

### Stages

A **stage** is a named (or implicit) location that holds data files for loading into tables or unloading from tables. Stages don't store table data — just files with metadata pointers.

#### Types of Stages

| Type | Category | Notes |
|---|---|---|
| **User Stage** | Internal | `@~` — one per user, auto-created, cannot set `FILE_FORMAT`, cannot be dropped/altered/shared, not suitable if multiple users need the same files or if the user lacks `INSERT` on the target table |
| **Table Stage** | Internal | `@%table_name` — one per table, auto-created, implicitly dropped with the table, **cannot** be referenced with a role other than the table owner's, does not support setting file format or copy options like a named stage, cannot grant privileges separately (tied to table privileges) |
| **Named Stage** | Internal or External | Explicit DB object created with `CREATE STAGE`; supports full grant model (`USAGE`, `READ`, `WRITE`), reusable across tables/users — the only stage type recommended for team/production use |
| **External Stage** | External | Points to S3 / Azure Blob / GCS; needs `URL` + credentials or a **Storage Integration** |

```sql
-- Named internal stage
CREATE OR REPLACE STAGE my_int_stage
  FILE_FORMAT = (TYPE = CSV SKIP_HEADER = 1)
  COMMENT = 'internal landing stage';

-- Named external stage (S3) using storage integration (recommended)
CREATE OR REPLACE STAGE my_ext_stage
  URL = 's3://my-bucket/path/'
  STORAGE_INTEGRATION = my_s3_int
  FILE_FORMAT = (TYPE = PARQUET);
```

#### Server-Side / Client-Side Encryption

Encryption is set via the `ENCRYPTION = (...)` clause on `CREATE STAGE` / `ALTER STAGE` / ad-hoc `COPY INTO`.

| Cloud | Encryption Types |
|---|---|
| **Internal stages** | `SNOWFLAKE_SSE` (default, Snowflake-managed AES-256) or `SNOWFLAKE_FULL` (required for **Tri-Secret Secure**; `SNOWFLAKE_SSE` does **not** support Tri-Secret Secure) |
| **AWS S3** | `AWS_CSE` (client-side, requires `MASTER_KEY`, symmetric key only), `AWS_SSE_S3` (server-side, no extra config), `AWS_SSE_KMS` (server-side, optional `KMS_KEY_ID`), `NONE` |
| **Azure Blob** | `AZURE_CSE` (client-side, requires `MASTER_KEY`), `NONE` |
| **GCS** | `GCS_SSE_KMS` (optional `KMS_KEY_ID`), `NONE` |

> ⚠️ **Exam trap:** For internal stages, **all data is always encrypted by Snowflake automatically** (SNOWFLAKE_SSE at minimum) — you cannot disable encryption on internal stages. The choice is only between `SNOWFLAKE_SSE` and `SNOWFLAKE_FULL`.
>
> ⚠️ **KMS gotcha:** If you set `AWS_SSE_KMS` with a customer-managed CMK, the **IAM role used by the storage integration** must be added as a **Key User** in AWS KMS, or loads fail with "access denied," even though bucket permissions look correct.

#### CREATE OR REPLACE STAGE — Hidden Consequences

Recreating a stage (`CREATE OR REPLACE STAGE`) has **non-obvious side effects** (classic exam trap):
- The stage's **directory table**, if any, is **dropped**. If recreated with a directory table, the new directory starts **empty**.
- The **association with any external table** referencing the stage **breaks** — because an external table links to a stage via a **hidden internal ID**, not by name. You must `CREATE OR REPLACE EXTERNAL TABLE` for each dependent external table to restore the link.
- Similarly, recreating a **storage integration** breaks its link to any stage referencing it (same hidden-ID mechanism) — you must `ALTER STAGE ... SET STORAGE_INTEGRATION = ...` again.

#### Temporary Stages

- Dropping a **temporary external stage** removes only the stage object — the underlying files in cloud storage are untouched.
- Dropping a **temporary internal stage** **purges all files** in it immediately, regardless of load status, and they **cannot be recovered**. (This prevents storage charges from orphaned temp stage files — but means you must keep external copies of anything staged temporarily.)

#### Access Control Cheat-Sheet

- Named stage: needs `CREATE STAGE` (to make it) and `USAGE` (internal) or `USAGE` (external) to reference it; `READ`/`WRITE` privileges control `GET`/`PUT`/`COPY INTO` direction.
- Loading via `COPY INTO <table>` from a stage also requires `INSERT` on the target table.

**Exam traps:**
- User stage and table stage **cannot be granted or shared** — only named stages support role-based grants, which is why Snowflake/the exam favors named stages for "multiple users" or "team" scenarios.
- A **stage does not store data twice** for internal stages loaded via `PUT` — the file lives in Snowflake-managed storage until removed (`REMOVE`) or purged (`COPY ... PURGE = TRUE`).
- `LIST @stage` returning 0 rows when you know files exist ⇒ almost always a **credentials/prefix/bucket-policy** problem, not a Snowflake bug.
- Storage Integration is *always* the preferred/recommended answer over embedding `AWS_KEY_ID`/`AWS_SECRET_KEY` directly in a stage — because it avoids storing long-lived credentials in Snowflake and delegates access via an IAM role (trust policy) instead.

---

### Directory Tables

A **directory table** is an **implicit metadata layer** on a stage (internal or external) — **not a separate grantable object** — that stores file-level metadata: file name/path, size, last-modified timestamp, and a Snowflake file URL (`METADATA$FILENAME`, etc. when queried).

```sql
CREATE STAGE my_stage
  URL = 's3://mybucket/files/'
  STORAGE_INTEGRATION = my_int
  DIRECTORY = (ENABLE = TRUE AUTO_REFRESH = TRUE);
```

- Used primarily for **unstructured data** (images, PDFs, audio) so you can `SELECT` a file listing, generate presigned/scoped URLs, or build downstream pipelines (e.g., streams on the directory table).
- **Refresh modes:**
  - **Manual:** `ALTER STAGE <name> REFRESH` — performs a `LIST` operation; can be **slow/expensive** for large or fast-growing stages. Use a selective `SUBPATH` to reduce cost.
  - **Automatic:** via cloud event notifications (S3 SQS/SNS, Azure Event Grid, GCS Pub/Sub) — billed similarly to Snowpipe (shows up as Snowpipe charges in `PIPE_USAGE_HISTORY`).
- **Manual refreshes block simultaneous automatic refreshes** on external stages; automated refresh resumes after the manual one finishes.
- If auto-refresh is never triggered even once after creation, **querying the directory table returns no results** until the first successful refresh event.
- **Internal stage auto-refresh** is currently **AWS-only** (preview); GCP/Azure-hosted accounts cannot auto-refresh directory tables on internal stages.
- Recreating the underlying stage (`CREATE OR REPLACE STAGE`) **drops the directory table**.
- Azure gotcha: only `BlobCreated`/`BlobDeleted` events trigger refresh; **renaming** a blob/directory does **not** trigger a refresh.
- GCS gotcha: only `OBJECT_FINALIZE` / `OBJECT_DELETE` events trigger refresh.

**Exam traps:**
- Directory tables have **no grantable privileges of their own** — access is controlled through privileges on the parent **stage**.
- Streams **can** be created on directory-table-enabled stages / standard tables / views (see Streams section) — a directory table is one of the supported stream source types.

---

### COPY INTO

`COPY INTO <table>` (loading) and `COPY INTO <location>` (unloading) are the workhorse bulk data-movement commands, executed by a **virtual warehouse** (never serverless, unlike Snowpipe/Tasks-serverless).

```sql
COPY INTO my_table
  FROM @my_stage
  FILE_FORMAT = (FORMAT_NAME = 'my_file_format')
  PATTERN = '.*contacts[1-5]\.csv'
  ON_ERROR = 'CONTINUE';
```

**Key copy options:**

| Option | Behavior / Gotcha |
|---|---|
| `FILES` | Explicit list of file names, max **1000** files; default `ON_ERROR=ABORT_STATEMENT` applies if a listed file is missing (unless overridden) |
| `PATTERN` | Regex match against staged file paths; slower than `FILES` for very large stages since it must evaluate the whole listing |
| `FORCE = TRUE` | Reloads files even if already loaded (bypasses **LOAD_HISTORY** dedupe check) — **can duplicate data**, use with care (e.g., replay after a bad transform) |
| `PURGE = TRUE` | Deletes files from the stage **after a successful load**; default `FALSE`; if the purge silently fails, **no error is raised** |
| `SIZE_LIMIT` | Max bytes for the *entire* COPY statement (not per file); COPY finishes the file that crosses the threshold before stopping |
| `MATCH_BY_COLUMN_NAME` | `CASE_SENSITIVE`/`CASE_INSENSITIVE`/`NONE` — maps semi-structured file columns to table columns by name; incompatible with `VALIDATION_MODE` |
| `VALIDATION_MODE` | Validates without loading (`RETURN_ERRORS`, `RETURN_ALL_ERRORS`, `RETURN_N_ROWS`); **cannot** be used with a COPY that also transforms data during load |
| `INCLUDE_METADATA` | Maps pseudo-columns (filename, row number, etc.) into target table columns |

**Load-history / deduplication mechanics (heavily tested):**
- Snowflake tracks loaded files (by name + file size/checksum) in table-level **LOAD_HISTORY** metadata for **64 days**.
- A file that matches a previously *successfully* loaded file (same name/size) is **skipped by default** — this is why re-running `COPY INTO` on the same stage doesn't duplicate rows, *unless* `FORCE = TRUE`.
- Snowpipe keeps its **own, separate** load-history metadata **tied to the pipe object, not the table** — retained **14 days**. Recreating a pipe **loses this history** (risk of duplicate loads if old files are still in the stage).

**Warehouse sizing gotcha:** COPY INTO parallelizes **by file** — an N-file load benefits from a warehouse with N worker threads, but throwing a bigger warehouse at a load of a few files (or one enormous single file processed serially) wastes credits. Start X-Small, scale only if `EXECUTION_TIME` proves the bottleneck.

**Region gotcha:** For best performance/cost, the external stage's cloud storage should be in the **same region** as the Snowflake account/warehouse to minimize cross-region transfer latency/cost.

**Unloading data (`COPY INTO <location>`):**
- Supports the same file format types (mostly used with CSV/JSON/Parquet).
- `SINGLE = TRUE` unloads to one file (no file extension is auto-appended — you must supply it in the path).
- `MAX_FILE_SIZE` controls output chunking (default ~16 MB uncompressed data per unloaded file, can be tuned).
- `HEADER = TRUE` includes a header row in CSV unload output (default `FALSE`).
- Unloaded files applying `bucket-owner-full-control` ACL require `STORAGE_AWS_OBJECT_ACL` set on the storage integration.

**Exam traps:**
- `VALIDATE()` function / `VALIDATION_MODE` — used to inspect a **previous** COPY execution's errors via its Query ID, or pre-validate before loading. `RETURN_ALL_ERRORS` returns errors even from files partially loaded in an earlier `ON_ERROR = CONTINUE` run.
- `INFORMATION_SCHEMA.COPY_HISTORY` (table-scoped) vs `ACCOUNT_USAGE.COPY_HISTORY` (account-wide, longer retention, latency) vs `ACCOUNT_USAGE.LOAD_HISTORY` — know which to query for troubleshooting.
- COPY INTO from stage requires `INSERT` on the target table **and** `USAGE`/`READ` on the stage.

---

### Error Handling (COPY INTO)

`ON_ERROR` controls what happens when a row/file fails to parse or violates constraints.

| Value | Behavior |
|---|---|
| `CONTINUE` *(default in many code samples, but NOT the true default)* | Skips only the erroring **rows**, loads the rest of the file; error message reports max **1 error per file** in output; `ROWS_PARSED` − `ROWS_LOADED` = error row count |
| `SKIP_FILE` | Skips the **entire file** if **any** row errors — buffers the whole file first (whether errors exist or not), so it's **slower** than `CONTINUE`/`ABORT_STATEMENT` |
| `SKIP_FILE_<num>` | Skip file only if error-row count ≥ `<num>` |
| `'SKIP_FILE_<num>%'` | Skip file only if error-row **percentage** ≥ `<num>%` |
| `ABORT_STATEMENT` | **This is the true default** if `ON_ERROR` is omitted — stops the **entire load** on the first error found in any file |

> ⚠️ **Exam trap — the actual default:** If you omit `ON_ERROR`, Snowflake uses **`ABORT_STATEMENT`**, not `CONTINUE`. Many blog posts/diagrams gloss over this; the official docs are explicit that ABORT_STATEMENT is the default and "isn't always the best option."

**Decision guidance (scenario-question fodder):**
- High-volume, loosely-structured logs/clickstream where partial data loss is tolerable → `CONTINUE`.
- Files with no logical row grouping (e.g., auto-generated at intervals) and you want max throughput → `CONTINUE` over `SKIP_FILE` (SKIP_FILE's full-file buffering wastes time/credits on large files with only a few bad rows).
- Financial/compliance data where every row matters → `ABORT_STATEMENT`.
- Backfilling messy historical data with a known noise tolerance → `SKIP_FILE_<n>%`.

**Common production pitfall:** `ON_ERROR = CONTINUE` on a very large file **can exceed warehouse timeout** before completing, leaving the table in a **partially-loaded state** — mitigate by splitting files or right-sizing the warehouse.

**Diagnosing failures:**
```sql
SELECT * FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
  TABLE_NAME=>'my_table', START_TIME=> DATEADD(hours,-1,CURRENT_TIMESTAMP())));

SELECT * FROM TABLE(VALIDATE(my_table, JOB_ID => '<query_id>'));
```

**Exam traps:**
- `ON_ERROR` and `VALIDATION_MODE` **can** be combined (`RETURN_ALL_ERRORS`) *except* when the COPY statement also applies **transformations** during load — that combination is unsupported.
- Skipped files due to prior successful load (name/size match) are **not "errors"** — they simply don't reprocess. Confirm via `COPY_HISTORY` (`status = 'LOAD_SKIPPED'`).

---

## 3.2 Automated Data Ingestion

### Snowpipe

**Snowpipe** is Snowflake's serverless, continuous **file-based** ingestion service. Internally it just runs the `COPY INTO` you define inside a **PIPE** object, triggered by new files landing in a stage.

```sql
CREATE OR REPLACE PIPE my_pipe
  AUTO_INGEST = TRUE
  AS
  COPY INTO my_table
  FROM @my_ext_stage/landing/
  FILE_FORMAT = (FORMAT_NAME = my_csv_format);
```

**Two trigger mechanisms:**
1. **Auto-ingest (`AUTO_INGEST = TRUE`):** Cloud event notifications (S3 SQS/SNS, Azure Event Grid, GCS Pub/Sub) tell Snowpipe a new file landed; Snowpipe polls its queue and loads. **Most common** approach.
2. **REST API (`AUTO_INGEST = FALSE`):** Client app calls `insertFiles` REST endpoint (or SDK) naming files to load; requires **key-pair authentication with JWT**.

**Serverless & billing:**
- No warehouse needed — Snowflake-managed compute.
- Cost = **compute time** (~1.25 credits/compute-hour equivalent) **+ a per-file overhead fee** (~0.06 credits / 1000 files processed). This is why **file sizing matters enormously**: many tiny files rack up overhead fees even if total bytes are small. Snowflake recommends **100–250 MB compressed** files.
- Latency: typically **60 seconds to a few minutes** (not true real-time); depends on file detection + queue depth + file size/complexity.

**Pipe lifecycle & staleness (heavily tested):**
- Pipes can be **paused**: `ALTER PIPE my_pipe SET PIPE_EXECUTION_PAUSED = TRUE;`
- While paused, incoming event notifications are **retained for 14 days** (default retention).
- If a pipe stays paused **longer than 14 days**, it becomes **stale** — resuming requires `SYSTEM$PIPE_FORCE_RESUME('pipe_name')` with an explicit staleness override acknowledgment.
- On resume after a long pause, Snowflake **skips notifications older than the 14-day window** — those files are silently NOT loaded via the queue (you'd need `ALTER PIPE ... REFRESH` or manual `COPY INTO` for the backlog).
- `ALTER PIPE ... REFRESH` — manually queues files staged within the **previous 7 days** (default and max) that were missed by notifications; good for closing gaps, not for old historical backfills (use bulk `COPY INTO` for anything older).
- File-loading metadata (dedupe history) is tied to the **pipe object**, retained **14 days** — **recreating the pipe drops this history**, risking duplicate loads of files still sitting in the stage.
- Snowpipe **ignores modified files re-staged with the same name** — to reload a changed file you must recreate the pipe (`CREATE OR REPLACE PIPE`).
- Modifying the referenced stage's `URL` can silently **break** an `AUTO_INGEST=TRUE` pipe that depends on that stage's cloud messaging config — requires pipe recreation.
- Files are **not guaranteed to load in the order staged** — multiple queue-consumer processes pull in parallel (though generally FIFO-ish/oldest-first).

**Monitoring:**
- `INFORMATION_SCHEMA.PIPE_USAGE_HISTORY` — 14-day window.
- `ACCOUNT_USAGE.PIPE_USAGE_HISTORY` — 365-day window, latency for freshness.
- `SYSTEM$PIPE_STATUS('pipe_name')` — JSON status (executionState, lastReceivedMessageTimestamp, etc.) — use to debug why a pipe isn't loading (e.g., `executionState: PAUSED`).

**Exam traps:**
- Snowpipe does **not** load files that existed in the stage **before** the pipe was created (unless `ALTER PIPE ... REFRESH`, bounded to 7 days, or manual `COPY INTO`).
- Never mix bulk `COPY INTO` and Snowpipe against the **same set of files/target** — their load histories are tracked **separately**, risking duplicate loads.
- Snowpipe is a **serverless** feature — you can't choose a warehouse for it (unlike a user-managed COPY task).

---

### Snowpipe Streaming

**Snowpipe Streaming** ingests **rows directly** (no staged files, no COPY INTO under the hood for classic/simple ingestion) via an SDK — designed for row-based sources like Kafka topics, IoT devices, CDC streams, application events.

**Two architectures (know both — commonly conflated in questions):**

| | Classic Architecture | High-Performance Architecture |
|---|---|---|
| Ingest target | Directly into the **table** | Into the table via a **PIPE object** |
| SDK language | Java only | Java, Python, and REST (GA) |
| In-flight transformation | No | **Yes** — filter, reorder, cast, apply expressions using COPY-like syntax inside the PIPE, plus pre-clustering at ingest |
| Pricing | Compute + open client connections | **Flat per-GB rate**: 0.0037 credits/uncompressed GB (identical to Snowpipe file-ingestion pricing) |
| Status | **Planned for deprecation** (no immediate action required; existing workloads still supported) | Current recommended path |
| Schema evolution | Manual (unless Kafka + schema registry) | Automatic |
| Iceberg table support | No | Yes (v2 & v3) |

**Key features:**
- **Latency:** as low as 5–10 seconds end-to-end (some marketing claims "5 seconds"); much faster than Snowpipe's ~60s–minutes.
- **Throughput:** designed for up to 10 GB/s per table.
- **Exactly-once semantics:** via **offset tokens** — the client app tracks the last committed offset per **channel** and resumes/replays from there on recovery, preventing duplicates or gaps.
- **Channels map to partitions** (e.g., a Kafka partition ↔ a channel) — ordered ingestion **within** a channel, not necessarily across channels.
- **No native UPSERT/DELETE** — it's **insert-only**; deduplication/merges must happen downstream via Streams+Tasks, Dynamic Tables, or scheduled `MERGE`.
- Soft limits: Classic architecture supports up to **~100 channels per table** (varies with clustering config); Snowpipe classic (file-based) can queue up to **10,000 files** per pipe.

**Exam traps:**
- Snowpipe Streaming is meant to **complement**, not replace, Snowpipe — use it when data arrives as **rows** (Kafka/IoT/app events), keep classic Snowpipe for **file-based** batch drops.
- A bare row insert via Snowpipe Streaming does **not** guarantee immediate consistency with concurrent DML on the same table from another session; check documentation nuances on transactional isolation before assuming otherwise.
- **The producer is responsible for retries/error-handling** — Snowflake doesn't auto-recover a crashed producer's unflushed buffer.

---

### Streams

A **stream** (a.k.a. **table stream**) is a first-class Snowflake object providing **Change Data Capture (CDC)**: it records **DML changes** (INSERT/UPDATE/DELETE, including TRUNCATE) to a source object between two transactional offsets. **A stream stores no table data itself — only an offset pointer + metadata**, so creating many streams on one table is cheap.

```sql
CREATE OR REPLACE STREAM my_stream ON TABLE my_table;
-- optionally: APPEND_ONLY = TRUE | INSERT_ONLY = TRUE, SHOW_INITIAL_ROWS = TRUE, AT/BEFORE (time travel)
```

**Metadata columns exposed when querying a stream:**
| Column | Meaning |
|---|---|
| `METADATA$ACTION` | `INSERT` or `DELETE` |
| `METADATA$ISUPDATE` | `TRUE` if part of an UPDATE (represented internally as paired DELETE+INSERT) |
| `METADATA$ROW_ID` | Immutable row identifier, used to track a row's changes over time |

#### Three Stream Types

| Type | Tracks | Supported on | Notes |
|---|---|---|---|
| **Standard (delta)** | INSERT, UPDATE, DELETE, TRUNCATE | Standard tables, directory tables, views | Most complete but most "expensive" to consume if many deletes/updates occur between reads |
| **Append-only** | INSERT only | Standard tables, directory tables, views (**NOT** dynamic tables) | Ignores UPDATE/DELETE entirely — e.g., 10 inserted rows then 5 deleted before consumption still shows **10** inserted rows; more performant since deletions add no processing overhead |
| **Insert-only** | INSERT (file-arrival) only | **External tables** and externally-managed Iceberg tables only | Tracks new files appearing in cloud storage; does **not** record the diff when a file is overwritten — old file's removal is invisible, new file's rows appear as fresh inserts |

> ⚠️ **Exam trap:** "Insert-only" ≠ "Append-only." Insert-only is specifically for **external-table/file-based** sources; Append-only is for **native tables/views**. A question that says "only supported on external tables" is describing **insert-only**, not append-only.

**Offset & staleness mechanics (very commonly tested):**
- A stream's offset advances **only** when the stream is consumed inside a **successful DML transaction** (`INSERT ... SELECT FROM stream`, `MERGE ... USING stream`, etc.) — a bare `SELECT` does **not** advance the offset.
- A stream becomes **stale** when its offset falls **outside the data retention period** of the source table — at that point, **historical/unconsumed data is lost**, and the stream must be **dropped and recreated**.
- If the source table's `DATA_RETENTION_TIME_IN_DAYS` is less than 14 days and the stream hasn't been consumed, Snowflake **temporarily extends** retention (up to `MAX_DATA_EXTENSION_TIME_IN_DAYS`, default cap 14 days) to avoid staleness — this **costs extra storage**.
- Check staleness via `SHOW STREAMS` / `DESCRIBE STREAM` → `STALE_AFTER` column (predicted staleness timestamp) and the `stale` boolean.
- **Best practice:** create **one stream per consumer** — don't point two different downstream consumers at the same stream object, since consuming it in one pipeline advances the offset for everyone, causing the other consumer to silently miss data.

**Object-lifecycle gotchas:**
- `CREATE OR REPLACE TABLE` on the source table **drops its history**, making any dependent stream **stale immediately**.
- **Renaming** a source table does **not** break its streams or cause staleness.
- If a table is **dropped and a new table created with the same name**, the **old stream does not attach** to the new table.
- You **cannot ALTER** a stream's type (standard ↔ append-only ↔ insert-only) — `APPEND_ONLY`/`INSERT_ONLY` can only be set at **creation time**; to change, drop & recreate (optionally with `AT(STREAM => 'old_stream')` to preserve the current offset, or `CREATE OR REPLACE STREAM ... CLONE`).
- Streams on **views**: supported since March 2022; underlying tables must be **native Snowflake tables**; the view's SQL is limited to projections, filters, `UNION ALL`, and inner/cross joins (no complex aggregations).
- Streams do **not** track changes from **materialized view** refreshes, nor from operations like **reclustering** that don't produce logical change data.
- Streams on **shared tables**: the data **provider** must explicitly enable `CHANGE_TRACKING = TRUE` on the shared table/view before a **consumer** can create a stream on it.
- Streams **cannot** be created directly `ON TABLE` for a **dynamic table** — use `ON DYNAMIC TABLE` syntax instead; `APPEND_ONLY` is unsupported on dynamic-table streams (filter `METADATA$ACTION = 'INSERT'` downstream instead, or use a custom-incremental dynamic table).
- Streams on **standard/append-only** types are **not supported** on Apache Iceberg tables using an **external catalog**.

**Exam traps:**
- "Does a stream contain table data?" → **No.** It only stores the **offset** + relies on the source table's change-tracking metadata/versioning history.
- "Do streams guarantee exactly-once semantics automatically?" → They give you the **mechanism** (offset advancing on successful DML) to build exactly-once pipelines, but it's the **consuming transaction's atomicity** that provides the guarantee — a failed transaction leaves the offset unmoved so a retry is safe (idempotent by design), but poorly written downstream logic can still double-process.

---

### Tasks

A **Task** executes a **single SQL statement** (or a stored procedure call) on a **schedule** or in response to an **event (trigger)**. Tasks are Snowflake's built-in orchestration primitive, frequently paired with Streams for ELT.

```sql
CREATE OR REPLACE TASK my_task
  WAREHOUSE = my_wh          -- omit this + set warehouse-related params for SERVERLESS
  SCHEDULE = '5 MINUTE'
  WHEN SYSTEM$STREAM_HAS_DATA('my_stream')
  AS
  INSERT INTO target_table
  SELECT * FROM my_stream WHERE METADATA$ACTION = 'INSERT';
```

**Compute models:**
| Model | Behavior |
|---|---|
| **User-managed** | Runs on a specified virtual warehouse you size/manage |
| **Serverless** (omit `WAREHOUSE`) | Snowflake auto-predicts and scales resources; billed **1.5x** the equivalent user-managed warehouse compute rate; max size ≈ **XXLARGE** warehouse-equivalent — if you need more, switch to user-managed |

**Scheduling:**
- **CRON** or simple interval (`'5 MINUTE'`) syntax.
- New tasks are created **SUSPENDED** by default — must explicitly `ALTER TASK ... RESUME`.
- Snowflake guarantees **only one instance** of a scheduled task runs at a time — if a run is still executing when the next scheduled time arrives, **that scheduled run is skipped** (not queued).
- **Triggered tasks:** use `WHEN SYSTEM$STREAM_HAS_DATA('stream')` — when first resumed, checks whether the stream has changes since the last run; if none, **skips the run without consuming compute**. Minimum re-trigger interval is **30 seconds** by default, tunable down to **10 seconds** via `USER_TASK_MINIMUM_TRIGGER_INTERVAL_IN_SECONDS`.
- If a triggered task hasn't run in **12 hours**, Snowflake schedules a **health check** to prevent the underlying stream from going stale (timing not guaranteed).

**Task Graphs (DAGs):**
- Composed of one **root task** + dependent **child tasks** via `AFTER <parent_task_name>`.
- **Directed, acyclic** — no loops permitted.
- Limits: max **1000 tasks per graph**; each task can have max **100 parent** and **100 child** tasks.
- An optional **finalizer task** runs after all other tasks in the graph complete (success or failure) — for cleanup.
- `CREATE TASK ... AFTER taskA, taskB` lets a child depend on **multiple** predecessors (requires them **all** to succeed first).
- Timeout precedence: if `STATEMENT_TIMEOUT_IN_SECONDS` and `USER_TASK_TIMEOUT_MS` are both set, the **lower non-zero value** wins. A timeout set on the **root task** applies to the **entire graph**; a timeout on a **child/finalizer task** applies only to that task; if both root and child set one, the **child's timeout takes precedence for that child**.
- `SUSPEND_TASK_AFTER_NUM_FAILURES` — root task auto-suspends after N consecutive failures of any single task in the graph (0 = unlimited retries, no auto-suspend).

**DML/transaction gotchas:**
- Tasks executing DML require `AUTOCOMMIT = TRUE`; if the account-level parameter is `FALSE`, you must explicitly set `AUTOCOMMIT = TRUE` on the individual task, or its DML statements **fail**.
- **Only one task should consume a given stream** — consuming it in multiple parallel tasks causes race conditions on the offset.

**Recreate/clone gotchas:**
- `CREATE OR REPLACE TASK` → the recreated task is **suspended by default**; if it was a root/standalone task, its **next scheduled run is cancelled**.
- Cloning a task graph requires cloning **each** dependent task individually and re-establishing `AFTER` relationships with `ALTER TASK ... ADD AFTER`.

**Cost:** No separate "task" fee — you only pay for the compute (warehouse credits or serverless credits) consumed when the task actually runs. Serverless is ~1.5x cost per compute-second vs. equivalent user-managed warehouse, but often cheaper overall for short/sporadic jobs (no warehouse spin-up/idle cost) — rough breakeven is around tasks that run **<40 seconds**.

**Exam traps:**
- Tasks are **not inherently event-based** — they're fundamentally **schedule-based**; "event-driven" behavior (`WHEN SYSTEM$STREAM_HAS_DATA`) is layered on top of a schedule/trigger mechanism, not truly push-based like a webhook.
- A **task cannot call another task directly** — dependency is expressed via `AFTER`, and Snowflake's scheduler handles execution order; there's no imperative "call" semantics.
- `SYSTEM$STREAM_HAS_DATA` returning `FALSE` just means **no new change rows since the last consumed offset** — not that the stream is broken.

---

### Dynamic Tables

**Dynamic Tables** declaratively materialize the result of a query and **automatically** keep it fresh according to a **Target Lag**, without you writing Streams/Tasks/MERGE logic yourself.

```sql
CREATE OR REPLACE DYNAMIC TABLE dt_orders
  TARGET_LAG = '10 minutes'
  WAREHOUSE = transform_wh
  REFRESH_MODE = INCREMENTAL   -- or FULL, AUTO, ADAPTIVE
  AS
  SELECT order_id, customer_id, order_date, quantity * unit_price AS line_total
  FROM raw_orders
  WHERE order_status != 'returned';
```

#### Target Lag
- Defines the **maximum acceptable staleness** vs. base tables — a *goal*, **not a guarantee**. Actual lag can exceed target due to warehouse size, data volume, query complexity, or pipeline depth.
- **`TARGET_LAG = DOWNSTREAM`**: the table has **no independent schedule** — it only refreshes when a downstream dynamic table (that reads from it) needs it to. Snowflake uses the **shortest** target lag among all downstream consumers.
  - ⚠️ **Gotcha:** If a `DOWNSTREAM` table has **no downstream consumers at all**, it **never auto-refreshes**, and Snowflake gives **no warning**.
  - Soft dependencies created via `DYNAMIC_TABLE_REFRESH_BOUNDARY()` do **not** count as downstream consumers for scheduling purposes.

#### Refresh Modes
| Mode | Behavior |
|---|---|
| `INCREMENTAL` | Processes only rows changed since last refresh (via lightweight internal streams) — efficient for filters/joins/most aggregations |
| `FULL` | Recomputes the entire result from scratch every refresh — required when the query uses constructs incompatible with incremental (e.g., `EXCEPT`, non-deterministic functions in some contexts, `SAMPLE`/`TABLESAMPLE`, sequences) |
| `AUTO` (default if omitted) | Snowflake decides INCREMENTAL vs FULL **once, at creation time** — resolved mode won't silently change later, but behavior of the *heuristic itself* can change across Snowflake releases, causing inconsistent behavior across recreations. **Best practice: set REFRESH_MODE explicitly in production.** |
| `ADAPTIVE` | Behaves like INCREMENTAL but auto-reinitializes (full recompute) when it detects large upstream changes; can use a separate `INITIALIZATION_WAREHOUSE` for the occasional full reinit so routine refreshes use a smaller warehouse |
| `CUSTOM_INCREMENTAL` | Advanced: user-defined refresh logic via MERGE/INSERT DML |

**Constraint (frequently tested):** A dynamic table using `INCREMENTAL` (or `ADAPTIVE`) refresh **cannot be downstream of** a `FULL`-refresh dynamic table — a full refresh replaces the *entire* output each cycle, giving no delta for the incremental table to consume (unless the upstream has a system-derived unique key or a defined **frozen region**).

#### Initialization
- `INITIALIZE = ON_CREATE` (default): populates data **immediately** at creation — the `CREATE` statement blocks until done.
- `INITIALIZE = ON_SCHEDULE`: `CREATE` returns immediately; the table stays **empty** and shows a *"Dynamic table is not initialized"* error if queried before the first scheduled refresh completes within the target-lag window.

#### Pipelines & Consistency
- Multiple chained dynamic tables form a **pipeline**; Snowflake infers the dependency graph, picks a **consistent snapshot timestamp**, and refreshes upstream-first so every downstream table sees a coherent point-in-time view (**snapshot isolation** across the whole DAG).
- If an upstream table's data hasn't changed, Snowflake can **skip** refreshing dependents that read from it.

#### Limitations (classic gotcha list)
- Can't use **session variables** / dynamic SQL in the definition.
- Can't use **`SAMPLE`/`TABLESAMPLE`**.
- Can't use **sequences** (`my_seq.NEXTVAL`).
- Requires **fresher than 60 seconds** target lag? Not supported — 60 seconds is the practical minimum granularity.
- Stored procedures / external functions generally unsupported in the definition (with narrow exceptions for certain UDTF/lateral-join patterns).
- `SELECT`-ing directly from the **base table** does **not** get query-rewritten to use the dynamic table (unlike materialized views, which support automatic query rewrite).
- **Dynamic table SQL cannot be altered in place** — any change to the defining query requires `CREATE OR REPLACE DYNAMIC TABLE` (this also **reinitializes** the table and any downstream incremental dynamic tables; downstream FULL-refresh tables are unaffected).
- Cloned **incremental** dynamic tables may perform a **full refresh** the first time after cloning.
- Operations on dynamic tables are **not** captured in `ACCESS_HISTORY`.

#### Dynamic Tables vs. Materialized Views vs. Streams+Tasks (classic comparison question)
| | Materialized View | Dynamic Table | Streams + Tasks |
|---|---|---|---|
| Query complexity | Single table only | Full SQL: joins, aggregations, window functions, unions | Full SQL (imperative, you write the logic) |
| Freshness | Always fresh (auto) | Configurable via Target Lag | You control via schedule |
| Query rewrite | Yes (queries on base table can transparently use it) | No | N/A |
| Control | Declarative, no orchestration code | Declarative, no orchestration code | Imperative, you write MERGE/INSERT logic |
| Best for | Simple aggregation/projection of one table | Multi-stage, multi-table declarative pipelines | Fine-grained control, custom incremental logic dynamic tables can't express |

**Exam traps:**
- "Dynamic tables always guarantee data within the target lag" → **False.** It's a best-effort target, not an SLA.
- "You can ALTER the SELECT statement of a dynamic table" → **False**, must recreate.
- Streams **can** be layered on top of dynamic tables (`CREATE STREAM ... ON DYNAMIC TABLE`) for hybrid pipelines (dynamic table → stream → triggered task) — but only on **incremental**-refresh dynamic tables (FULL refresh dynamic tables replace all output each cycle, so there's no incremental change history to expose to a stream).

---

### Openflow *(not tested until GA)*

Per the current exam guide, **Openflow** content is **excluded from testing until the feature reaches General Availability**. Candidates preparing today do not need deep operational knowledge of Openflow for the exam — just be aware it exists as Snowflake's data-integration/ingestion offering and watch for exam-guide updates once GA is announced.

---

## 3.3 Connectors & Integrations

### Drivers & Connectors

**Drivers** (language/protocol-level libraries) vs. **Connectors** (integration tools for specific platforms/ecosystems) — the exam guide separates these, and both terms appear in Snowflake's own docs somewhat interchangeably, so know the *practical* distinction:

**Drivers (`docs.snowflake.com/developer-guide/drivers`):**
| Driver | Language/Use |
|---|---|
| **JDBC** | Java and any JDBC-compatible tool (also underlies the Spark connector) |
| **ODBC** | C-based / BI tools (Tableau, Power BI, SQL clients); OS-specific builds |
| **Go Snowflake Driver** | Go apps — note: **does not support PUT/GET** |
| **.NET Driver** | C#/.NET apps — supports a **limited** operation set vs. Python/JDBC |
| **Node.js Driver** | Native async JavaScript interface |
| **PHP Driver** (PDO_Snowflake) | PHP apps |

**Connectors:**
| Connector | Purpose / Gotcha |
|---|---|
| **Python Connector** | Native Python package implementing the **Python DB API 2.0 spec**; does **not** depend on ODBC/JDBC (pure Python-native implementation) — distinct from **Snowpark for Python**, which pushes DataFrame operations down into Snowflake compute rather than just fetching rows |
| **Spark Connector** | Bi-directional; supports **predicate pushdown** and query pushdown (translates Spark logical plan operations into SQL executed in Snowflake); version-locked to specific Spark releases (e.g., v2.11.0 dropped Spark 3.0 support) — **must match Spark version** or jobs fail |
| **Kafka Connector** | Reads from Kafka topics, loads into Snowflake tables; runs inside a **Kafka Connect cluster**; supports both **Confluent** and **OSS** Kafka packages; can use either **Snowpipe** (file-buffered) or **Snowpipe Streaming** ingestion mode under the hood — configurable per connector instance |
| **SnowSQL** | Command-line client (not a "driver" per se) for interactive/scripted SQL execution |
| **SnowCD** (Snowflake Connectivity Diagnostic Tool) | Standalone tool to **troubleshoot network connectivity** issues (proxies, firewalls, allow-listing) before/instead of blaming the driver |

**Exam traps:**
- The **Go driver** notably **lacks PUT/GET support** — a scenario asking "how do I stage files from a Go app" should point you toward SnowSQL, JDBC, or Python instead.
- Spark connector + JDBC driver versions must be **compatible** — mismatches are a real documented gotcha (e.g., certain JDBC driver version ranges don't work with the Spark connector on GCP accounts).
- The Python Connector is **not** the same as **Snowpark Python** — Python Connector = classic client-side row fetching; Snowpark = server-side DataFrame/UDF execution pushed into Snowflake's engine.
- Kafka connector replication/failover: Snowflake does **not** support replication/failover for Snowpipe-mode Kafka connector ingestion, but **does** support it for **Snowpipe Streaming**-mode Kafka connector ingestion.

---

### Storage Integration

A **Storage Integration** is a Snowflake object that stores a **generated cloud identity** (an IAM role ARN for AWS, a service principal for Azure, a service account for GCS) that Snowflake uses to access external cloud storage — **without requiring long-lived secret keys to be stored in Snowflake**.

```sql
CREATE STORAGE INTEGRATION my_s3_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::123456789:role/snowflake-role'
  STORAGE_ALLOWED_LOCATIONS = ('s3://my-bucket/path/');
```

- **`TYPE` values:** `EXTERNAL_STAGE` (the common case — interface to external cloud storage), `POSTGRES_EXTERNAL_STORAGE` / `POSTGRES_INTERNAL_STORAGE` (Snowflake Postgres–specific, AWS-only currently).
- **One storage integration can back multiple stages** — a many-to-one relationship; you don't need a separate integration per stage.
- `STORAGE_ALLOWED_LOCATIONS` (allow-list) and `STORAGE_BLOCKED_LOCATIONS` (deny-list, useful with a `*` wildcard allow-list to carve out exceptions) scope exactly what buckets/paths the integration can touch — a key **least-privilege** control.
- Setup flow (AWS): create the integration → `DESC INTEGRATION` to get the auto-generated **IAM user ARN** and **External ID** → configure the **trust policy** on your AWS IAM role to allow that Snowflake IAM user to assume it (scoped by the external ID to prevent the "confused deputy" problem) → then create external stages referencing the integration.
- **Recreating** a storage integration (`CREATE OR REPLACE STORAGE INTEGRATION`) **breaks the link** to any stage using it — same hidden-ID mechanism as stages — requires `ALTER STAGE ... SET STORAGE_INTEGRATION = ...` on each affected stage afterward.
- Requires `CREATE INTEGRATION` privilege (account-level, typically ACCOUNTADMIN or a delegated role).

**Exam traps:**
- The **"most secure" answer** on a scenario question about avoiding long-lived credentials in Snowflake is almost always: **Storage Integration with IAM role delegation**, not embedding `AWS_KEY_ID`/`AWS_SECRET_KEY` in the stage.
- A storage integration is a schema-**less**, **account-level** object (unlike stages, which are schema-level) — it must be explicitly `GRANT USAGE`'d to roles that create stages referencing it.

---

### API Integration

An **API Integration** stores connection info + credentials for an **HTTPS API** — most commonly used for two purposes on the exam:
1. **Git integration** (connecting to GitHub/GitLab/Bitbucket/Azure DevOps repos)
2. **External functions** (calling out to an API Gateway — AWS API Gateway, Azure API Management, GCP API Gateway)
3. (Newer) External MCP servers used by MCP Connectors for Cortex Agents

```sql
CREATE OR REPLACE API INTEGRATION git_api_integration
  API_PROVIDER = git_https_api
  API_ALLOWED_PREFIXES = ('https://github.com/my-org')
  ALLOWED_AUTHENTICATION_SECRETS = (my_secret)
  ENABLED = TRUE;
```

- `API_PROVIDER = git_https_api` is the **only valid value** for Git-repository integrations.
- `API_ALLOWED_PREFIXES` — a **comma-separated, prefix-matched** allow-list of HTTPS endpoints; this is how you scope which repos/APIs the integration can reach (supports custom/internal Git server URLs too, not just public providers).
- `ALLOWED_AUTHENTICATION_SECRETS` — restricts which Snowflake **Secret** objects (holding a PAT/username-password) can be used with this integration; `= all` allows any secret, `= none` disallows authenticated access (public repos only).
- Creating an API Integration requires the **`CREATE API INTEGRATION`** privilege — typically restricted to **ACCOUNTADMIN** or a delegated admin role in most accounts (a common least-privilege exam scenario: "who must create the API integration for Git?").
- Public repositories **don't require authentication/secrets** — but you **cannot push/commit changes** to a public repo through Snowflake workspaces in that mode (read-only).

**Exam traps:**
- **API Integration ≠ Storage Integration.** Storage Integration → cloud object storage (S3/Azure/GCS). API Integration → HTTPS APIs (Git, external functions, MCP). Don't conflate them on a scenario question.
- The API integration doesn't itself hold Git credentials — that's the job of a separate **SECRET** object, referenced by the API integration's allow-list.

---

### Git Integration

**Git Integration** (GA since ~2024) lets Snowflake **natively sync** a remote Git repository (GitHub, GitLab, Bitbucket, Azure DevOps, CodeCommit, or a self-hosted server) into a special stage type called a **repository stage** (Git repository object).

**Setup sequence (exam loves ordering questions here):**
1. Create a **Secret** (personal access token or username/password) — *skip if public repo*.
2. Create an **API Integration** with `API_PROVIDER = git_https_api`, scoped via `API_ALLOWED_PREFIXES` to the Git host/org, and (if private) `ALLOWED_AUTHENTICATION_SECRETS` referencing the secret.
3. Create a **Git Repository** object referencing the API integration + (optionally) the secret + the repo `ORIGIN` URL.

```sql
CREATE OR REPLACE GIT REPOSITORY my_repo
  API_INTEGRATION = git_api_integration
  GIT_CREDENTIALS = my_secret          -- omit for public repos
  ORIGIN = 'https://github.com/my-org/my-repo.git';
```

**Usage patterns:**
- Files are addressable by branch/tag/commit using a special path syntax:
  `@my_repo/branches/main/...`, `@my_repo/tags/v1.0/...`, `@my_repo/commits/<hash>/...`
- `LIST @my_repo/branches/main` works just like listing any stage.
- `ALTER GIT REPOSITORY my_repo FETCH;` pulls the latest commits from the remote — **you must fetch explicitly**; Snowflake does **not** auto-poll the remote repo for changes.
- `EXECUTE IMMEDIATE FROM @my_repo/branches/main/setup.sql;` runs a SQL script straight from the synced repo.
- Can be used as the source for **Snowpark stored procedure/UDF handler code**, **Streamlit app** root locations (`CREATE STREAMLIT ... ROOT_LOCATION='@my_repo/branches/main/streamlit'`), and **Workspaces** (Snowsight's dev environment) which support full **two-way sync** — commit, push, branch, and resolve merge conflicts directly in Snowsight.
- **Empty repositories are not supported** — a Git repo must contain at least one branch before it can be linked.

**Read vs. write:**
- Basic `GIT REPOSITORY` objects (used for stored procedures/Streamlit/EXECUTE IMMEDIATE) are effectively **read-only** — you fetch, you don't push, from that mechanism.
- **Workspaces** git integration (a Snowsight-specific feature) supports **full read/write**: commit, push, pull, branch, conflict resolution — this is the two-way version.
- Public repositories can be read without a secret, but **cannot** be pushed to.

**Exam traps:**
- Order matters: **Secret → API Integration → Git Repository** — you cannot create the Git Repository object without a working, enabled API Integration referencing it, and the API Integration references the secret (not the other way around).
- `ACCOUNTADMIN`-level privilege (`CREATE API INTEGRATION`) is typically required upfront by an admin, even though day-to-day repo usage can be delegated to developer roles afterward.
- Fetching does **not** happen automatically on a schedule — a common pattern is to wrap `ALTER GIT REPOSITORY ... FETCH` inside a scheduled **Task** for pseudo-CI/CD auto-sync.

---

## Quick Comparison Tables

### Snowpipe vs. Snowpipe Streaming vs. COPY INTO (bulk)

| | Bulk COPY INTO | Snowpipe | Snowpipe Streaming |
|---|---|---|---|
| Trigger | Manual / scheduled Task | Cloud event notification or REST call | Application/SDK push |
| Unit | File(s) | File(s) | Rows |
| Compute | User-managed warehouse | Serverless | Serverless (billed per GB) |
| Latency | Depends on schedule | ~60s–minutes | ~5–10 seconds |
| Load history | Table metadata, 64 days | Pipe metadata, 14 days | Offset tokens (app-managed) |
| Best for | Large scheduled batch loads | Continuous file-drop pipelines | Kafka/CDC/IoT/event streams |

### Streams: Standard vs. Append-only vs. Insert-only

| | Standard | Append-only | Insert-only |
|---|---|---|---|
| Tracks | Insert/Update/Delete/Truncate | Insert only | Insert (file-arrival) only |
| Source objects | Tables, directory tables, views | Tables, directory tables, views | External tables, Iceberg (external catalog) |
| Performance | Baseline | Faster (skips delete/update overhead) | Faster, file-metadata based |

### Dynamic Table Refresh Modes

| Mode | When chosen | Reinit behavior |
|---|---|---|
| INCREMENTAL | Explicit or AUTO-resolved when query supports it | Processes deltas only |
| FULL | Explicit or AUTO-resolved when incremental unsupported (e.g., `EXCEPT`) | Recomputes everything every cycle |
| AUTO | Default; resolved once at creation | Behavior may shift across Snowflake releases |
| ADAPTIVE | Explicit | Incremental normally, auto full-reinit on big upstream shifts |

### Storage Integration vs. API Integration vs. Git Integration

| | Storage Integration | API Integration | Git Repository (Git Integration) |
|---|---|---|---|
| Connects to | Cloud object storage (S3/Azure/GCS) | Any HTTPS API (Git host, API Gateway, MCP server) | A specific Git remote (uses an API Integration under the hood) |
| Object level | Account-level | Account-level | Schema-level (repository stage) |
| Typical use | External stages | Git sync, external functions | Sourcing code/SQL/Streamlit from version control |
| Credential storage | IAM role/service account (no static keys) | Secrets (referenced, not stored inline) | Uses the API Integration's secret |

---

## Rapid-Fire Exam Traps

1. Default `ON_ERROR` for `COPY INTO` is **`ABORT_STATEMENT`**, not `CONTINUE`.
2. `SKIP_FILE` is **slower** than `CONTINUE`/`ABORT_STATEMENT` because it buffers the whole file regardless of error count.
3. `FORCE = TRUE` **can duplicate data** — it bypasses the load-history dedupe check.
4. `PURGE = TRUE` failing silently returns **no error**.
5. Table/user stages **cannot be shared or granted** — only named stages support role-based access control.
6. `CREATE OR REPLACE STAGE` **drops the directory table** and **breaks external table linkage** (hidden-ID mechanism).
7. Recreating a **storage integration** breaks stage linkage the same way.
8. Internal stages are **always** encrypted (`SNOWFLAKE_SSE` minimum) — you can't turn encryption off; `SNOWFLAKE_FULL` is required for **Tri-Secret Secure**.
9. Dropping a **temporary internal stage** purges all files immediately and irrecoverably; dropping a temporary **external** stage leaves cloud files untouched.
10. Snowpipe pipe becomes **stale after 14 days** of pausing; force-resume with `SYSTEM$PIPE_FORCE_RESUME`.
11. `ALTER PIPE ... REFRESH` only reaches back **7 days** — older backlogs need bulk `COPY INTO`.
12. Streams only advance their offset on a **successful DML consumption**, not a bare `SELECT`.
13. **Append-only** ≠ **Insert-only**: append-only = native tables/views (ignores updates/deletes); insert-only = external tables/Iceberg (file-arrival tracking).
14. A stream becomes **stale** if unconsumed past the source table's (possibly auto-extended) data retention window — must be **recreated**, not repaired.
15. You **cannot ALTER** a stream's type (standard/append/insert) after creation.
16. Tasks are **suspended by default** on creation — must explicitly `RESUME`.
17. Only **one instance** of a scheduled task runs at a time; overlapping scheduled runs are **skipped**, not queued.
18. Task graphs cap at **1000 tasks**, **100 parents/100 children** per task.
19. Dynamic table `TARGET_LAG` is a **goal, not a guarantee**.
20. `TARGET_LAG = DOWNSTREAM` with **no downstream consumers** → table **never refreshes**, with **no warning**.
21. Dynamic table SQL **cannot be altered** — must `CREATE OR REPLACE`, which **reinitializes** it (and incremental downstream dependents).
22. **Storage Integration** (not embedded keys) is always the "most secure"/"best practice" answer for external stage credentials.
23. **API Integration ≠ Storage Integration** — APIs (incl. Git) vs. cloud object storage, respectively.
24. Git integration setup order: **Secret → API Integration → Git Repository**.
25. Git Repository objects don't auto-fetch — `ALTER GIT REPOSITORY ... FETCH` must be run explicitly (often via a scheduled Task).
26. The **Go driver** doesn't support `PUT`/`GET`.
27. **Python Connector** ≠ **Snowpark for Python** — client-side fetch vs. server-side pushed execution.
28. Kafka connector supports **both** Snowpipe and Snowpipe Streaming as its underlying ingestion mode — but only Streaming mode supports replication/failover.

---

*End of Domain 3.0 study notes. Recommend pairing this with hands-on practice: create a named external stage with a storage integration, load/unload with varied `ON_ERROR` settings, build a Stream+Task pipeline, and convert it to a Dynamic Table to feel the tradeoffs firsthand.*

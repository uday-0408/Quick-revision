# Snowflake File Formats, Stages & COPY INTO — Complete SnowPro Core Reference

> Everything below is built only from Snowflake's official documentation (docs.snowflake.com — CREATE FILE FORMAT, CREATE STAGE, COPY INTO `<table>`, Directory Tables, Introduction to Unstructured Data, etc.), rewritten in plain language for quick studying. No third-party blogs were used as sources — only paraphrased for clarity, not invented.

## Table of Contents

1. [File Formats](#1-file-formats)
2. [Stages](#2-stages)
   - [2.1 Internal Stages](#21-internal-stages)
   - [2.2 External Stages](#22-external-stages)
   - [2.3 Server-Side Encryption](#23-server-side-encryption)
   - [2.4 Directory Tables](#24-directory-tables)
3. [COPY INTO Command](#3-copy-into-table)
4. [Error Handling Options](#4-error-handling-options)
5. [Quick Revision Cheat-Sheet](#5-quick-revision-cheat-sheet)
6. [Rapid-Fire Exam Traps](#6-rapid-fire-exam-traps)

---

## 1. File Formats

### 1.1 What it actually is, and why it exists

A **file format** is a named, schema-level database object that tells Snowflake how to *interpret the bytes* in a staged file — what separates one field from the next, what separates one record from the next, whether the file is compressed and with what algorithm, how nulls are represented, whether there's a header row, and so on. It is metadata *about* a file, not the file itself.

Why Snowflake built it as a standalone object instead of forcing you to type 15 options into every `COPY INTO` statement: real-world files are messy and inconsistent (pipe-delimited vs comma-delimited, gzip vs none, `NULL` vs empty string for missing values...). Packaging that parsing logic once, as a named, reusable, **grantable** object, means a team can standardize how "the CSVs from vendor X" get parsed without repeating themselves in every load statement.

File formats are used in three places: loading data (`COPY INTO <table>`), unloading data (`COPY INTO <location>`), and defining external tables over staged files.

### 1.2 Syntax

```sql
CREATE [ OR REPLACE ] [ { TEMP | TEMPORARY | VOLATILE } ] FILE FORMAT [ IF NOT EXISTS ] <name>
  [ TYPE = { CSV | JSON | AVRO | ORC | PARQUET | XML } [ formatTypeOptions ] ]
  [ COMMENT = '<string_literal>' ]
```

- Default `TYPE` if omitted: `CSV`.
- `TEMP`/`TEMPORARY`/`VOLATILE` file formats live only for the session and are dropped automatically at session end.
- A **preview** variant, `CREATE OR ALTER FILE FORMAT`, creates the format if it's missing or alters an existing one in place, following the same rules as `ALTER FILE FORMAT`. You can change any format option or the comment this way, but you **cannot** change `TYPE` — that always requires a full replace.

### 1.3 Supported types — load vs unload matrix

| Type | Load | Unload | Notes |
|---|---|---|---|
| CSV | ✅ | ✅ | despite the name, any single- or multi-byte character can be the delimiter, not only comma |
| JSON | ✅ | ✅ (NDJSON only) | loading accepts either NDJSON or a comma-separated array of documents; **unloading always produces NDJSON** |
| AVRO | ✅ | ❌ | binary format, load-only |
| ORC | ✅ | ❌ | binary format, load-only |
| PARQUET | ✅ | ✅ | binary format |
| XML | ✅ | ❌ | plain text, load-only |

**Exam trap:** AVRO, ORC, and XML can never be unloaded — only CSV, JSON, and PARQUET support both directions.

### 1.4 CSV format options worth memorizing

| Option | Default | What it actually controls |
|---|---|---|
| `COMPRESSION` | `AUTO` | codec of the file; `AUTO` can't auto-detect Brotli |
| `RECORD_DELIMITER` | newline (logical — `\r\n` counts as one newline on Windows files) | what separates rows |
| `FIELD_DELIMITER` | `,` | what separates columns; max 20 chars, accepts hex/octal escapes |
| `SKIP_HEADER` | `0` | number of raw lines to blindly skip at the top — **not aware of RECORD_DELIMITER**, just skips N literal CRLF-delimited lines |
| `PARSE_HEADER` | `FALSE` | use row 1 as column names for `INFER_SCHEMA` / `MATCH_BY_COLUMN_NAME`; **cannot be combined with `SKIP_HEADER`** |
| `FIELD_OPTIONALLY_ENCLOSED_BY` | `NONE` | quote character wrapping string fields |
| `ERROR_ON_COLUMN_COUNT_MISMATCH` | `TRUE` | hard error when a row's field count ≠ table's column count; **ignored entirely for COPY transformations** (SELECT-based loads) |
| `EMPTY_FIELD_AS_NULL` | `TRUE` | two delimiters in a row (`,,`) → SQL NULL instead of an empty string |
| `NULL_IF` | `\N` | string(s) treated as SQL NULL on load |
| `TRIM_SPACE` | `FALSE` | strip stray whitespace outside/inside quoted fields |
| `SKIP_BLANK_LINES` | `FALSE` | otherwise a blank line is an end-of-record error |
| `MULTI_LINE` | `TRUE` | allow a record delimiter to appear inside a quoted field without erroring |
| `ESCAPE_UNENCLOSED_FIELD` | backslash | if a data row physically ends in `\`, that backslash escapes the following newline and **merges two rows into one** — a classic silent-corruption gotcha; set to `NONE` to avoid it |
| `ENCODING` | `UTF8` | source character set; Snowflake always stores data internally as UTF-8 |

**Performance gotcha worth knowing for the exam:** Snowflake can parallel-scan large uncompressed CSV files (>128 MB, RFC4180-compliant) *only* when `MULTI_LINE = FALSE`, `COMPRESSION = NONE`, and `ON_ERROR` is `ABORT_STATEMENT` or `CONTINUE`.

### 1.5 JSON format options

| Option | Default | What it does |
|---|---|---|
| `STRIP_OUTER_ARRAY` | `FALSE` | removes the enclosing `[ ]` so each element loads as its own row |
| `STRIP_NULL_VALUES` | `FALSE` | drops object fields / array elements whose value is `null` |
| `ALLOW_DUPLICATE` | `FALSE` | allow duplicate object keys (last one wins) instead of erroring |
| `ENABLE_OCTAL` | `FALSE` | permit octal number literals |
| `IGNORE_UTF8_ERRORS` | `FALSE` | alternate spelling of `REPLACE_INVALID_CHARACTERS` — same effect |
| `MULTI_LINE` | `TRUE` | allow a JSON record to span multiple physical lines |

### 1.6 AVRO / ORC / PARQUET / XML — the highlights

- **PARQUET**: `USE_LOGICAL_TYPE` (interpret Parquet logical types correctly — recommended `TRUE`); `USE_VECTORIZED_SCANNER` (faster, columnar-aware reader — default is currently `FALSE` but Snowflake has said a future behavior-change bulletin will flip the default to `TRUE`; forces `BINARY_AS_TEXT = FALSE` and `USE_LOGICAL_TYPE = TRUE` whenever it's on); `BINARY_AS_TEXT` defaults `TRUE` but Snowflake recommends setting it `FALSE` to avoid silent type-conversion surprises.
- **XML**: `STRIP_OUTER_ELEMENT` (expose second-level elements as separate documents), `PRESERVE_SPACE`, `DISABLE_AUTO_CONVERT` (stop auto-casting numeric/boolean-looking text).
- **AVRO/ORC**: mostly just `COMPRESSION`, `TRIM_SPACE`, `REPLACE_INVALID_CHARACTERS`, `NULL_IF` — much thinner option set than CSV/JSON because the binary formats are already strongly typed.

### 1.7 Where format options can live, and who wins a conflict

You can set file-format options in three separate places:

1. Inline inside a `COPY INTO <table>` statement
2. Attached to a named stage (`CREATE/ALTER STAGE ... FILE_FORMAT = ...`)
3. Attached to a table definition

If the same option is set in more than one place, precedence is:

**COPY INTO statement > stage definition > table definition** (COPY wins).

Snowflake's own recommendation: don't set *copy options* (as opposed to file-format options) via `CREATE STAGE`, `ALTER STAGE`, `CREATE TABLE`, or `ALTER TABLE` at all — always set copy options explicitly inside the `COPY INTO <table>` statement itself.

**Exam trap:** `CREATE OR REPLACE FILE FORMAT` silently breaks the link to any external table that references it, because an external table links to a file format by a hidden internal ID, not by name — replacing the format drops the old object and creates a brand-new one under the hood. You must recreate the dependent external table(s) afterward (`GET_DDL` can hand you the DDL to redo it).

### 1.8 Access control

- `CREATE FILE FORMAT` privilege on the **schema** is required to create a new permanent file format.
- `OWNERSHIP` on the file format itself is required to run `CREATE OR ALTER FILE FORMAT` against an *existing* format.

---

## 2. Stages

A **stage** is simply a pointer to where data files sit — inside Snowflake's own storage, or out in your cloud storage — so that `COPY INTO <table>` (loading) and `COPY INTO <location>` (unloading) know where to read from / write to. Every load and unload in Snowflake passes through a stage of some kind.

Stages come in two families: **internal** (Snowflake-managed storage) and **external** (your own cloud storage bucket/container).

### 2.1 Internal Stages

Snowflake supports three flavors of internal stage: **User**, **Table**, and **Named**. Every user and every table automatically gets one (User and Table); Named stages are the only kind you have to explicitly create.

#### User Stage

- Referenced with `@~` (e.g., `LIST @~;`).
- Exists automatically for every user — one per user, always there, nothing to create.
- **Cannot** be altered or dropped.
- Does **not** support attaching file-format options to the stage itself — you must specify `FILE_FORMAT`/copy options directly in the `COPY INTO <table>` command instead.
- Good fit: a single user needs to load the same set of files into *several different* tables.
- Bad fit: multiple users need access to the files, or the current user lacks `INSERT` on the target table.

#### Table Stage

- Referenced with `@%table_name` (e.g., a table `mytable` has stage `@%mytable`).
- Exists automatically for every table — same implicit-object idea as the user stage.
- **Cannot** be altered or dropped.
- Not a real, separate database object — it's an implicit stage tied to the table, so it has **no grantable privileges of its own**. To list, query, or drop files on it, you must be the table owner (hold the `OWNERSHIP` privilege on the table).
- Good fit: files only ever need to land in *this one* table, but several users need to reach them.
- Bad fit: you need the same files loaded into multiple different tables. Table stages also don't support COPY *transformations* (a `SELECT`-based load).
- **Iceberg tables in Snowflake do not get a table stage at all** — a nice, sharp exam trap.

#### Named (Internal) Stage

- An explicit database/schema object you create with `CREATE STAGE`.
- The most flexible option: it's a real securable object, so ordinary GRANT/REVOKE rules apply, and ownership can be transferred to another role.
- Recommended whenever loads are regular and could span multiple users and/or multiple tables.

```sql
-- Minimal named internal stage
CREATE STAGE my_int_stage
  ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE');

-- Temporary internal stage (dropped, and its files purged, at session end)
CREATE TEMPORARY STAGE my_temp_int_stage;

-- Attach a reusable file format
CREATE STAGE my_int_stage
  FILE_FORMAT = my_csv_format;
```

Loading data has two required steps for any internal stage: `PUT` the file onto the stage first (client → stage), then `COPY INTO <table>` (stage → table).

**Internal stage comparison**

| | User Stage | Table Stage | Named Stage |
|---|---|---|---|
| Reference | `@~` | `@%table_name` | `@stage_name` |
| Auto-created? | Yes | Yes | No — must `CREATE STAGE` |
| ALTER/DROP? | No | No | Yes |
| File format attachable? | No | Yes (but not the common recommendation) | Yes |
| Grantable privileges | N/A | None (owner-only) | Full RBAC |
| Best for | 1 user → many tables | 1 table → many users | many users → many tables |

### 2.2 External Stages

An **external stage** points at a location in your own cloud storage instead of Snowflake-managed storage. Supported providers:

- **Amazon S3** buckets
- **Google Cloud Storage** buckets
- **Microsoft Azure** containers (Blob storage, Data Lake Storage Gen2, general-purpose v1/v2)
- **Amazon S3-compatible** storage (via a custom endpoint)
- **Microsoft Fabric OneLake**

The storage location can be public or private/protected. **You cannot reach archival storage tiers that require a restore step first** — e.g., Amazon S3 Glacier Flexible Retrieval / Glacier Deep Archive, or Azure Archive Storage — the files must be restored to a retrievable tier before Snowflake can read them.

```sql
-- Amazon S3, using a storage integration (recommended)
CREATE STAGE my_ext_stage
  URL = 's3://load/files/'
  STORAGE_INTEGRATION = myint;

-- Amazon S3, using direct IAM credentials
CREATE STAGE my_ext_stage1
  URL = 's3://load/files/'
  CREDENTIALS = (AWS_KEY_ID = '1a2b3c' AWS_SECRET_KEY = '4x5y6z');

-- Google Cloud Storage
CREATE STAGE my_ext_stage
  URL = 'gcs://load/files/'
  STORAGE_INTEGRATION = myint;

-- Microsoft Azure
CREATE STAGE my_ext_stage
  URL = 'azure://myaccount.blob.core.windows.net/load/files/'
  STORAGE_INTEGRATION = myint;
```

**Authentication: `STORAGE_INTEGRATION` vs `CREDENTIALS`**

- `STORAGE_INTEGRATION` (strongly recommended): a separate Snowflake object that delegates auth to a cloud IAM entity, so no cloud secrets are ever typed into a stage definition. Requires `USAGE` on the integration.
- `CREDENTIALS`: direct secrets — `AWS_KEY_ID`/`AWS_SECRET_KEY`(/`AWS_TOKEN`) or `AWS_ROLE` for S3, `AZURE_SAS_TOKEN` for Azure. GCS has no direct-credentials option — it's storage-integration only.
- **Government-region / China-region gotcha:** a storage integration only works when the Snowflake account *and* the cloud storage are hosted in the *same* government or China region. Cross-region access in those cases requires `CREDENTIALS` instead.
- Permanent ("long-term") credentials are technically accepted by `COPY INTO` but Snowflake explicitly discourages them for security reasons — prefer temporary/scoped credentials, or better, a storage integration.

`CREATE STAGE` does **not** validate the URL or credentials at creation time — a typo only surfaces the first time you actually try to use the stage.

### 2.3 Server-Side Encryption

Encryption on a stage answers "how are the bytes protected at rest, and can Snowflake itself decrypt them to hand you a usable URL?" It's configured differently for internal vs external stages, and **you cannot change the encryption type after the stage is created**.

#### Internal stages

```sql
ENCRYPTION = ( TYPE = 'SNOWFLAKE_FULL' | TYPE = 'SNOWFLAKE_SSE' )
```

| Type | Meaning | Default? |
|---|---|---|
| `SNOWFLAKE_FULL` | Client-side **and** server-side. `PUT` encrypts the file on the client before upload (128-bit key by default, configurable to 256-bit via the `CLIENT_ENCRYPTION_KEY_SIZE` parameter), and Snowflake *additionally* applies AES-256 server-side. | **Yes, this is the default** |
| `SNOWFLAKE_SSE` | Server-side only — encrypted by the cloud service hosting your Snowflake account when the file arrives. | No |

**Why this matters beyond the exam:** if you plan to generate pre-signed, file, or scoped URLs to let something *outside* a Snowflake session read a staged file (the whole point of "unstructured data" support), you need `SNOWFLAKE_SSE`. Under the default `SNOWFLAKE_FULL`, the file stays client-side encrypted, and anything downloading it through a generated URL gets back unreadable, still-encrypted bytes — because the decrypting key lives with the client, not the Snowflake service.

**Compliance gotcha:** if you need **Tri-Secret Secure**, you must use `SNOWFLAKE_FULL` — `SNOWFLAKE_SSE` does **not** support Tri-Secret Secure. This is a very common trick question because it inverts the "SSE is more advanced" intuition.

Client versions matter too: creating an internal stage with server-side encryption currently requires JDBC Driver v3.12.11 or higher.

#### External stages (varies per cloud)

| Cloud | Types | Notes |
|---|---|---|
| **Amazon S3** | `AWS_CSE` (client-side, needs `MASTER_KEY`), `AWS_SSE_S3` (server-side, no extra config), `AWS_SSE_KMS` (server-side, optional `KMS_KEY_ID`), `NONE` | Default `NONE` |
| **Google Cloud Storage** | `GCS_SSE_KMS` (server-side, optional `KMS_KEY_ID`), `NONE` | Default `NONE` |
| **Microsoft Azure** | `AZURE_CSE` (client-side, needs `MASTER_KEY`), `NONE` | Default `NONE` |

`ENCRYPTION` on an external stage is only required if the storage location/files are already encrypted — public, unencrypted buckets don't need it. `KMS_KEY_ID` is ignored for *loading* (it only matters when *unloading*, to pick which KMS key encrypts the new files).

```sql
-- Internal stage, server-side encryption, with a directory table
CREATE STAGE my_int_stage
  ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE')
  DIRECTORY = (ENABLE = true);
```

### 2.4 Directory Tables

A **directory table** is *not* a separate database object — it's an **implicit metadata catalog layered on top of a stage** (internal or external). Conceptually it's similar to an external table, except instead of exposing row data it exposes **file-level metadata**: which files exist on the stage, their size, when they last changed, and a URL to reach them. It has no grantable privileges of its own — access is controlled through the underlying stage's privileges.

You enable one either at stage-creation time or later:

```sql
-- At creation
CREATE STAGE mystage
  DIRECTORY = (ENABLE = TRUE)
  FILE_FORMAT = myformat;

-- Later
ALTER STAGE mystage SET DIRECTORY = (ENABLE = TRUE);
```

**What it's used for**

- Listing every file on a stage along with its metadata (size, last-modified, URL) via a simple `SELECT`.
- Joining that metadata against a real Snowflake table to build a "rich view" that pairs unstructured files with structured data about them.
- Building event-driven file-processing pipelines: a **stream** on the directory table detects new/changed files, and a **task** fires a UDF/stored procedure to process them.

**Refresh options** (keeping the metadata catalog in sync with what's actually in cloud storage)

| Parameter | Applies to | Meaning | Default |
|---|---|---|---|
| `AUTO_REFRESH` | internal + external | `TRUE` = Snowflake automatically keeps the metadata current as files change (via event notifications). `FALSE` = you must run `ALTER STAGE ... REFRESH` yourself. | `FALSE` (internal); `FALSE` (external, but see below) |
| `REFRESH_ON_CREATE` | external only | One-time automatic refresh immediately after the stage is created, to register files that already exist at that path. | `TRUE` |

**Gotcha:** if the storage location already holds close to a million files or more, Snowflake recommends setting `REFRESH_ON_CREATE = FALSE` and instead running several scoped `ALTER STAGE ... REFRESH` calls against sub-paths, to avoid one enormous refresh operation.

**Querying a directory table**

```sql
SELECT * FROM DIRECTORY(@mystage);
SELECT FILE_URL FROM DIRECTORY(@mystage) WHERE SIZE > 100000;
SELECT FILE_URL FROM DIRECTORY(@mystage) WHERE RELATIVE_PATH LIKE '%.csv';
```

Output columns:

| Column | Type | Meaning |
|---|---|---|
| `RELATIVE_PATH` | TEXT | path to the file, used to build a file URL |
| `SIZE` | NUMBER | file size in bytes |
| `LAST_MODIFIED` | TIMESTAMP_TZ | last update timestamp in the stage |
| `MD5` | HEX | MD5 checksum |
| `ETAG` | HEX | ETag header |
| `FILE_URL` | TEXT | permanent Snowflake file URL, format `https://<account_identifier>/api/files/<db>.<schema>.<stage>/<relative_path>` |

**Rich view example** — joining directory metadata with a real table of business context:

```sql
CREATE VIEW reports_information AS
  SELECT
    file_url as report_link,
    author,
    publish_date,
    approved_date,
    geography,
    num_of_pages
  FROM directory(@my_pdf_stage) s
  JOIN report_metadata m
  ON s.file_url = m.file_url;
```

**Billing:** the event-notification overhead needed for automatic directory-table refreshes shows up on your bill as Snowpipe charges (it scales with how many new files land in cloud storage). A manual `ALTER STAGE ... REFRESH` incurs a small, separate cloud-services charge instead.

**Access control for directory tables** (privileges live on the *stage*, not the directory table itself)

| Action | Internal stage needs | External stage needs |
|---|---|---|
| `SELECT * FROM DIRECTORY(@stage)` | `READ` | `READ` or `USAGE` |
| `PUT` files | `WRITE` | n/a (internal only) |
| `REMOVE` files | `WRITE` | `WRITE` or `USAGE` |
| `ALTER STAGE ... REFRESH` | `WRITE` | `WRITE` or `USAGE` |

**Exam trap:** `CREATE OR REPLACE STAGE` drops the existing directory table entirely. If the stage is recreated with a directory table again, it starts **empty** and needs a fresh refresh.

**Troubleshooting connection to encryption:** if files downloaded from an internal stage come back corrupted, the standard first check is whether `ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE')` was actually set on the stage — this ties directly back to section 2.3.

---

## 3. COPY INTO `<table>`

### 3.1 What it does

`COPY INTO <table>` bulk-loads data from files that are already staged (internal or external) — or sitting directly at an ad hoc external cloud location — into an *existing* Snowflake table. It is the workhorse command behind virtually every batch load pattern, and shares almost all the same options with Snowpipe's continuous loading.

### 3.2 Syntax — standard load

```sql
COPY INTO [<namespace>.]<table_name>
     FROM { internalStage | externalStage | externalLocation }
[ FILES = ( '<file_name>' [ , ... ] ) ]
[ PATTERN = '<regex_pattern>' ]
[ FILE_FORMAT = ( { FORMAT_NAME = '<file_format_name>' |
                    TYPE = { CSV | JSON | AVRO | ORC | PARQUET | XML } [ formatTypeOptions ] } ) ]
[ copyOptions ]
[ VALIDATION_MODE = RETURN_<n>_ROWS | RETURN_ERRORS | RETURN_ALL_ERRORS ]
```

### 3.3 Syntax — load with transformation

```sql
COPY INTO [<namespace>.]<table_name> [ (<col_name> [ , ... ]) ]
     FROM ( SELECT [<alias>.]$<file_col_num>[.<element>] [ , ... ]
            FROM { internalStage | externalStage } )
[ FILES = (...) ]
[ PATTERN = '...' ]
[ FILE_FORMAT = (...) ]
[ copyOptions ]
```

A `SELECT` sourced from `$1`, `$2`, etc. (positional file columns) lets you reorder, cast, or extract nested elements *while* loading — but note the file's own column order/count no longer has to match the table. `VALIDATION_MODE` and `MATCH_BY_COLUMN_NAME` **cannot** be combined with this transformation style.

### 3.4 Where files can come from

| `FROM` value | Meaning |
|---|---|
| `@int_stage_name[/path]` | named internal stage |
| `@%table_name[/path]` | that table's own stage |
| `@~[/path]` | the current user's stage |
| `@ext_stage_name[/path]` | named external stage |
| `'s3://bucket[/path]'`, `'gcs://bucket[/path]'`, `'azure://account.blob.core.windows.net/container[/path]'` | ad hoc external location, no named stage involved |

`path` is treated as a **prefix**, not a folder in the traditional sense — Snowflake performs a prefix match against file names. Relative modifiers like `/./` or `/../` are interpreted *literally* as part of the prefix string, not resolved like a filesystem path.

### 3.5 Key copy options (beyond error handling — see Section 4)

| Option | Default | What it does |
|---|---|---|
| `FORCE` | `FALSE` | reload files even if their checksum shows they were already loaded — will duplicate rows |
| `PURGE` | `FALSE` | delete source files from the stage automatically after a successful load (silently no-ops on failure to purge — periodically `LIST` and clean up manually as a safety net) |
| `MATCH_BY_COLUMN_NAME` | `NONE` | `CASE_SENSITIVE` / `CASE_INSENSITIVE` match semi-structured data into table columns *by name* instead of position; column order in the file becomes irrelevant. Unmatched file columns are dropped; unmatched (nullable) table columns get NULL. **Cannot** combine with a COPY transformation |
| `LOAD_UNCERTAIN_FILES` | `FALSE` | by default, files are skipped if their load status is *unknown* (file staged >64 days ago **and** the table's first load was >64 days ago **and**, if it was already loaded once, that happened >64 days ago) — turn this on (or just use `FORCE`) to load them anyway |
| `SIZE_LIMIT` | none | stop the COPY once this many bytes have been loaded (at least one file always loads regardless) |
| `RETURN_FAILED_ONLY` | `FALSE` | only return failed files in the result set |
| `TRUNCATECOLUMNS` / `ENFORCE_LENGTH` | `FALSE` / `TRUE` | two names for opposite polarities of the same knob — auto-truncate over-length strings, or error on them; provided for compatibility with tools migrating from other databases; only set one |
| `INCLUDE_METADATA` | none | maps `METADATA$` fields (`FILENAME`, `FILE_ROW_NUMBER`, `FILE_CONTENT_KEY`, `FILE_LAST_MODIFIED`, `START_SCAN_TIME`) into real target columns that must already exist; requires `MATCH_BY_COLUMN_NAME` |
| `CLUSTER_AT_INGEST_TIME` | `FALSE` | pre-clusters rows by the target table's clustering key at ingest time — only meaningful with Snowpipe Streaming's high-performance architecture |
| `LOAD_MODE` | `FULL_INGEST` | Iceberg-table only: `FULL_INGEST` rewrites Parquet data under the table's base location; `ADD_FILES_COPY` just registers already Iceberg-compatible Parquet files in place (no rewrite, needs case-sensitive `MATCH_BY_COLUMN_NAME`, works best paired with `PURGE = TRUE`) |

### 3.6 Output columns

| Column | Type | Meaning |
|---|---|---|
| `FILE` | TEXT | source file name + relative path |
| `STATUS` | TEXT | loaded / load failed / partially loaded |
| `ROWS_PARSED` | NUMBER | rows parsed from the file |
| `ROWS_LOADED` | NUMBER | rows actually loaded |
| `ERROR_LIMIT` | NUMBER | error count that would trigger an abort |
| `ERRORS_SEEN` | NUMBER | error rows found in the file |
| `FIRST_ERROR` | TEXT | text of the first error |
| `FIRST_ERROR_LINE` | NUMBER | line number of the first error |
| `FIRST_ERROR_CHARACTER` | NUMBER | character position of the first error |
| `FIRST_ERROR_COLUMN_NAME` | TEXT | column implicated in the first error |

### 3.7 Working examples

```sql
-- From a named internal stage
COPY INTO mytable FROM @my_int_stage;

-- From a table's own stage (FROM can be omitted — Snowflake checks the table stage automatically)
COPY INTO mytable FILE_FORMAT = (TYPE = CSV);

-- From a named external stage, single file
COPY INTO mycsvtable FROM @my_ext_stage/tutorials/dataloading/contacts1.csv;

-- Column-name matching for semi-structured data
COPY INTO mytable
  FROM @my_ext_stage/tutorials/dataloading/sales.json.gz
  FILE_FORMAT = (TYPE = 'JSON')
  MATCH_BY_COLUMN_NAME = 'CASE_INSENSITIVE';

-- Pattern matching (compressed CSVs only, any subfolder depth)
COPY INTO mytable
  FILE_FORMAT = (TYPE = 'CSV')
  PATTERN = '.*/.*/.*[.]csv[.]gz';

-- Force a reload of unchanged files (creates duplicates on purpose)
COPY INTO load1 FROM @%load1/data1/
  FILES = ('test1.csv', 'test2.csv')
  FORCE = TRUE;

-- Purge successfully loaded files from the stage afterward
COPY INTO mytable PURGE = TRUE;
```

---

## 4. Error Handling Options

Error handling in Snowflake data loading really lives in **three layers**, and mixing them up is a very common source of confusion (and exam traps):

1. **`ON_ERROR`** — a `COPY INTO <table>` *copy option* that decides what happens *during* a load when bad rows are hit.
2. **`VALIDATION_MODE`** — a *dry-run* switch that checks files for problems **without loading any data at all**.
3. A handful of **file-format options** (`ERROR_ON_COLUMN_COUNT_MISMATCH`, `REPLACE_INVALID_CHARACTERS`/`IGNORE_UTF8_ERRORS`, `SKIP_BYTE_ORDER_MARK`) that decide whether specific *parsing* conditions count as errors at all.

### 4.1 `ON_ERROR` — the core copy option

```sql
ON_ERROR = { CONTINUE | SKIP_FILE | SKIP_FILE_<num> | 'SKIP_FILE_<num>%' | ABORT_STATEMENT }
```

| Value | Behavior |
|---|---|
| `CONTINUE` | Keep loading past errors. The statement reports **at most one error per file** in its result. `ROWS_PARSED − ROWS_LOADED` tells you how many rows had at least one problem — but a single bad row can carry multiple distinct errors, so don't treat that delta as a literal error count. |
| `SKIP_FILE` | Discard the *entire* file the moment any error is found in it. Because Snowflake has to buffer/read the whole file before it can decide to skip it, this is **slower** than `CONTINUE` or `ABORT_STATEMENT`. Skipping large files over a handful of bad rows wastes time and credits — `CONTINUE` is usually better for files with no clean logical row boundary (e.g., machine-generated at rough intervals). |
| `SKIP_FILE_<num>` (e.g. `SKIP_FILE_10`) | Skip the file only once its error-row count reaches or exceeds `<num>`. |
| `'SKIP_FILE_<num>%'` (e.g. `'SKIP_FILE_10%'`) | Skip the file once its error-row *percentage* exceeds `<num>`. |
| `ABORT_STATEMENT` | Stop the whole load on the first error found. **Nuance:** this truly halts the statement for a *missing* file only when that file was explicitly named via the `FILES = (...)` option — if a file simply can't be found some other way (wrong pattern, etc.), the load quietly continues without it. |

**Default value differs depending on the loading mechanism — a favorite exam trap:**

- Bulk loading (`COPY INTO <table>` run directly) → default is **`ABORT_STATEMENT`**
- Snowpipe (continuous loading) → default is **`SKIP_FILE`**

**A load that aborts on a missing file never appears in `COPY_HISTORY`** (nothing was actually ingested) — check `QUERY_HISTORY` instead to investigate what happened.

Some scenarios where `ON_ERROR` doesn't behave cleanly and deserve extra caution:

- `SELECT DISTINCT` inside a COPY transformation
- Loading into clustered tables
- A stream sitting on the target table while loading CSV data
- Partitioned Iceberg tables — a bad partition-transform value fails the job even with `ON_ERROR = CONTINUE`

### 4.2 `VALIDATION_MODE` — pre-flight checking without loading

```sql
VALIDATION_MODE = RETURN_<n>_ROWS | RETURN_ERRORS | RETURN_ALL_ERRORS
```

| Value | Behavior |
|---|---|
| `RETURN_<n>_ROWS` (e.g. `RETURN_10_ROWS`) | Validates just that many rows; stops at the **first** error encountered within them if any exist. |
| `RETURN_ERRORS` | Returns every error (parsing, conversion, etc.) across every file named in the statement. |
| `RETURN_ALL_ERRORS` | Same as above, but **also** surfaces errors from files that were *partially* loaded in an earlier run where `ON_ERROR = CONTINUE` was used. |

Restrictions: cannot be combined with a COPY transformation (a `SELECT`-based load), cannot be combined with `MATCH_BY_COLUMN_NAME`, and is **not supported for Iceberg tables** at all.

```sql
-- See every problem across all target files, without loading anything
COPY INTO mytable VALIDATION_MODE = 'RETURN_ERRORS';

-- Spot-check the first 10 rows before committing to a real load
COPY INTO mytable VALIDATION_MODE = 'RETURN_10_ROWS';
```

### 4.3 Investigating errors after the fact

- **`VALIDATE(table_name, JOB_ID => query_id | '_last')`** — a table function that returns *every* error from a past `COPY INTO <table>` execution (more complete than the single `FIRST_ERROR` you get from the COPY output or from `COPY_HISTORY`). It returns **no results** for a load that used the default `ABORT_STATEMENT` (nothing landed, so there's nothing to validate), and it doesn't support COPY statements that used a transformation.
- **`COPY_HISTORY(table_name, start_time, [end_time], [pipe_name])`** — an Information Schema table function covering the **last 14 days** of load activity for both bulk `COPY INTO` and Snowpipe. `STATUS` shows loaded / partially loaded / failed; `FIRST_ERROR_MESSAGE` only ever shows the *first* reason if a file had several issues. Dropping/recreating the target table wipes its history.
- **Account Usage `COPY_HISTORY` view** — same idea, longer retention window, higher latency.
- **Recovering the actual bad rows to a file** — a genuinely useful pattern that chains three features together:

```sql
COPY INTO mytable FROM @mystage/myfile.csv.gz
  VALIDATION_MODE = RETURN_ALL_ERRORS;

SET qid = LAST_QUERY_ID();

COPY INTO @mystage/errors/load_errors.txt
  FROM (SELECT rejected_record FROM TABLE(RESULT_SCAN($qid)));
```

### 4.4 Error-adjacent knobs that actually live in the file format

A few "does this count as an error" decisions aren't `ON_ERROR` at all — they're file-format options that change whether a condition is even flagged:

| Option | Default | Effect |
|---|---|---|
| `ERROR_ON_COLUMN_COUNT_MISMATCH` (CSV) | `TRUE` | mismatched field count vs table columns is a hard parsing error unless turned off — and it's silently **ignored entirely** during COPY transformations |
| `REPLACE_INVALID_CHARACTERS` / `IGNORE_UTF8_ERRORS` (same effect, different format types use different names) | `FALSE` | when `TRUE`, bad UTF-8 bytes are swapped for the Unicode replacement character (`�`) instead of erroring the load |
| `SKIP_BYTE_ORDER_MARK` | `TRUE` | silently drops a leading BOM; if set `FALSE`, a BOM can either throw an error or get merged into the first column's value |

### 4.5 Other load-error gotchas worth knowing

- **Google Cloud Storage "directory blobs":** GCS can list zero-byte, slash-terminated entries (e.g. `my_gcs_stage/load/`) as if they were files when they were created via the GCS console. A `COPY` that references the stage can fail on these — the fix is a `PATTERN` clause to filter them out, or setting `ON_ERROR = SKIP_FILE`.
- The COPY command does **not** validate Parquet data-type conversions — bad type mappings there won't necessarily surface as a load error the way CSV/JSON ones do.

---

## 5. Quick Revision Cheat-Sheet

**Stage types**

| Stage | Auto-created | ALTER/DROP | Grantable |
|---|---|---|---|
| User (`@~`) | Yes | No | No |
| Table (`@%t`) | Yes | No | Owner-only |
| Named internal | No | Yes | Yes |
| Named external | No | Yes | Yes |

**Encryption defaults**

| Context | Default | Notes |
|---|---|---|
| Internal stage | `SNOWFLAKE_FULL` | client + server side; use `SNOWFLAKE_SSE` for URL-based unstructured access; use `SNOWFLAKE_FULL` for Tri-Secret Secure |
| External S3 | `NONE` | `AWS_SSE_S3` / `AWS_SSE_KMS` / `AWS_CSE` available |
| External GCS | `NONE` | `GCS_SSE_KMS` available |
| External Azure | `NONE` | `AZURE_CSE` available |

**Directory table refresh**

| Param | Applies to | Default |
|---|---|---|
| `AUTO_REFRESH` | internal + external | `FALSE` |
| `REFRESH_ON_CREATE` | external only | `TRUE` |

**`ON_ERROR` defaults**

| Mechanism | Default |
|---|---|
| Bulk `COPY INTO` | `ABORT_STATEMENT` |
| Snowpipe | `SKIP_FILE` |

**Key COPY copy-option defaults**

| Option | Default |
|---|---|
| `FORCE` | `FALSE` |
| `PURGE` | `FALSE` |
| `MATCH_BY_COLUMN_NAME` | `NONE` |
| `LOAD_UNCERTAIN_FILES` | `FALSE` |
| `RETURN_FAILED_ONLY` | `FALSE` |
| `ENFORCE_LENGTH` | `TRUE` |
| `TRUNCATECOLUMNS` | `FALSE` |

---

## 6. Rapid-Fire Exam Traps

- Unloadable formats: only **CSV, JSON, PARQUET**. AVRO/ORC/XML are load-only.
- `PARSE_HEADER = TRUE` and `SKIP_HEADER` **cannot** be used together.
- `ERROR_ON_COLUMN_COUNT_MISMATCH` is ignored during COPY transformations.
- File-format precedence: **COPY statement > stage > table**.
- `CREATE OR REPLACE FILE FORMAT` breaks the hidden-ID link to dependent external tables.
- Iceberg tables get **no table stage**.
- Table stages don't support COPY transformations.
- `SNOWFLAKE_SSE` does **not** support Tri-Secret Secure — `SNOWFLAKE_FULL` does.
- Encryption type on a stage **cannot be changed after creation**.
- Client-side-encrypted (`SNOWFLAKE_FULL`) files come back unreadable through generated URLs — you need `SNOWFLAKE_SSE` for that.
- `CREATE OR REPLACE STAGE` **drops the existing directory table**.
- `REFRESH_ON_CREATE` only exists for **external** stages, not internal ones.
- `ON_ERROR` default flips between bulk load (`ABORT_STATEMENT`) and Snowpipe (`SKIP_FILE`).
- `SKIP_FILE` is slower than `CONTINUE`/`ABORT_STATEMENT` because it must buffer the whole file first.
- `VALIDATION_MODE` cannot combine with a COPY transformation, `MATCH_BY_COLUMN_NAME`, or Iceberg tables.
- `VALIDATE()` returns nothing for a load that used the default `ABORT_STATEMENT`.
- `COPY_HISTORY` (function) only covers the **last 14 days**.
- `MATCH_BY_COLUMN_NAME` and a COPY transformation (`SELECT`-based load) **cannot** be combined.
- `INCLUDE_METADATA` requires `MATCH_BY_COLUMN_NAME` and the target columns must **already exist** in the table.
- Government/China-region external stages: storage integrations only work same-region; use `CREDENTIALS` for cross-region.
- Archival cloud storage tiers (S3 Glacier, Azure Archive) must be restored before Snowflake can read them at all.
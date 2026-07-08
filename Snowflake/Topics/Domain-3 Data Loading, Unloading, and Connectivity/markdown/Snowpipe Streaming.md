This page is just a landing/index page — pointing to the 4 real sub-pages. Let me pull those.# 📘 SNOWPIPE STREAMING — Complete Notes (SnowPro Core)

---

## 1. What Is Snowpipe Streaming? (The Big Picture)

Normal Snowpipe works on **files** — something writes a file to a stage, Snowpipe notices it, loads it. But a lot of real-world data never naturally exists as a file: Kafka topics, IoT sensor readings, app click events. Forcing these into files first just adds delay.

**Snowpipe Streaming's job**: let an application write **rows directly into a table** — no file, no staging area, no waiting for a file to "land."

**Simple analogy**: Snowpipe = bucket brigade (fill a bucket/file, carry it, pour it in). Snowpipe Streaming = an actual pipe — water (rows) flows in continuously.

### Key numbers (memorize — exam loves numbers)

| Metric | Value |
|---|---|
| Max throughput per table | **10 GB/s** |
| Latency (ingest → queryable) | **as low as 5 seconds** |
| Delivery guarantee | **Exactly-once** |
| Ordering | **Within a channel only** — not across the whole table |

⚠️ **Exam trap**: If a question describes multiple channels writing to the same table, ordering is guaranteed *inside each channel*, but there's **no guarantee how different channels interleave**.

---

## 2. Core Streaming Principles (the 5 pillars)

- **Continuous ingestion** — data flows in as produced, over long-lived connections, not batched.
- **Exactly-once delivery** — achieved through offset token tracking (explained below).
- **Ordered ingestion** — preserved *within a channel*.
- **Low latency** — queryable in as low as 5 seconds.
- **Serverless** — Snowflake manages all the compute for ingestion; you provision nothing.

---

## 3. How Data Actually Flows

```
Client app (Kafka / IoT / custom app)
        │
        ▼
SDK (Java/Python/Node) or REST API  →  appendRows() / Append Rows endpoint
        │
        ▼
     CHANNEL   (ordered stream, tracks offset token)
        │
        ▼
     PIPE object   (validates schema → in-flight transforms →
                     pre-clustering → commits)
        │
        ▼
   Target Table  (queryable within ~5 sec)
```

**Connection options**: Java SDK, Python SDK, Node.js SDK, REST API, or the Snowflake Connector for Kafka. Snowflake now **recommends the SDKs over raw REST**, because all three SDKs share a common high-performance core under the hood. REST API is mainly for lightweight/edge devices that can't embed an SDK.

---

## 4. Channels — The Ordering & Identity Unit

A **channel** = a logical, named streaming connection into a table. Think of it as a dedicated lane on the pipe.

Two guarantees a channel gives you:
- **Ordered ingestion** within that channel.
- **Exactly-once delivery** via offset tokens.

**Rules to remember:**
- Channels are opened **against a pipe**.
- A single client can open channels to **multiple pipes**, but **can't open channels across accounts**.
- Channels are meant to be **long-lived** — reuse them across app restarts, don't recreate them every time, because the offset token history lives on the channel.
- You can **drop a channel** two ways: at closing (data auto-flushed first — safe) or "blindly" (not recommended — discards pending data).
- `SHOW CHANNELS` lists channels you have privileges on.

⚠️ **Critical gotcha — 30-day auto-cleanup**: Inactive channels (and their offset tokens) are **automatically deleted after 30 days of inactivity**. This is a favorite exam trap — if a channel that hasn't ingested anything in over a month suddenly needs to resume, its offset history is gone. Best practice: maintain your own separate copy of the offset outside Snowflake if there's a chance of a long gap.

---

## 5. Offset Tokens — How "Exactly-Once" Actually Works

This is the mechanism that makes exactly-once delivery possible, and it's a favorite scenario-question topic.

**Simple explanation**: An offset token is just a bookmark string *you* define (e.g., a Kafka partition offset, or `filename:line_number`). Every time you send rows, you attach the offset token that represents "how far I've gotten." Snowflake stores that token once the rows are committed.

**Key behavior:**
- Token starts as **NULL** when a channel is created.
- It updates only when rows tagged with that token are **actually committed**.
- Call `getLatestCommittedOffsetToken` any time to check progress.
- When you **re-open** a channel, Snowflake hands you back the last committed token — and **any uncommitted buffered data is discarded** (important! don't assume in-flight data survives a reopen).

**Crash recovery pattern (this is THE pattern to know for scenario questions):**
1. App crashes.
2. App restarts, reopens the *same-named* channel.
3. App calls `getLatestCommittedOffsetToken` → gets, say, `20`.
4. App resumes reading its source from position `21` onward.
5. No duplicates, no data loss.

This exact pattern applies whether the source is Kafka (`partition:offset`) or a log file (`filename:line_number`) — the docs give both examples and they demonstrate that offset tokens are a **general-purpose bookmark**, not something tied specifically to Kafka.

### `offsetToken` vs `continuationToken` — a classic "compare the two" exam question

| | `continuationToken` | `offsetToken` |
|---|---|---|
| Who manages it | **Snowflake** | **You (the client/user)** |
| Used by direct REST API only? | Yes | No — used by SDK and REST |
| Purpose | Keeps a single streaming session's internal request sequence correct (detects gaps/duplicates *within* the Append Rows call sequence) | Lets *you* track your own ingestion progress against your *external* source (Kafka, log file, etc.) |
| Does Snowflake use it internally? | Yes | **No** — Snowflake just stores it; it's your responsibility to use it |

⚠️ **Exam trap**: A question might say "which token does Snowflake use internally to detect duplicate requests during retries?" → answer is `continuationToken`, NOT `offsetToken`. The offsetToken is purely for *your* bookkeeping against your source system.

---

## 6. The PIPE Object — The Server-Side Brain

Every stream of rows flows through a PIPE object. It's not just a queue manager (like in classic Snowpipe) — here it actively does work:

- **In-flight transformations**: filter rows, reorder columns, cast types, apply expressions — using standard COPY command transformation syntax. No separate ETL step needed.
- **Pre-clustering**: sorts data during ingestion based on the table's clustering keys, for better query performance later.
- **Schema validation**: checks incoming data against the pipe's defined schema before committing.
- **Table feature support**: works with clustering keys, DEFAULT value columns, and AUTOINCREMENT/IDENTITY columns.

### Default Pipe — know this cold, it's a big exam topic

Snowflake auto-creates a **default pipe** for every table so you can start streaming with **zero DDL**.

- **Created on-demand**: it doesn't exist until the *first successful* pipe-info call or open-channel call is made against that table. You can't `SHOW PIPES` and see it before that first call happens.
- **Naming convention**: `<TABLE_NAME>-STREAMING` (e.g., `MY_TABLE-STREAMING`).
- **Fully Snowflake-managed**: you **cannot** run `CREATE`, `ALTER`, or `DROP` on it.
- Visible via `SHOW PIPES`, `DESCRIBE PIPE`, `SHOW CHANNELS`, and appears in `ACCOUNT_USAGE.PIPES`, `ACCOUNT_USAGE.METERING_HISTORY`, `ORGANIZATION_USAGE.PIPES`.
- **Supports**: clustering-key pre-sorting, AUTOINCREMENT/IDENTITY generation, DEFAULT value filling.
- **Does NOT support**: in-flight transformations. It uses `MATCH_BY_COLUMN_NAME` internally, which is a straight column-name match — no reshaping.

⚠️ **Exam trap**: "I need to transform/filter data during ingestion" → you **must create a custom named pipe**. The default pipe can't do it. This is one of the cleanest "which pipe do I need" scenario questions.

### Pre-clustering — a subtle but testable detail

- Requires the target table to already have **clustering keys defined**.
- Enabled via `CLUSTER_AT_INGEST_TIME = TRUE` in the `COPY INTO` statement of a custom pipe.
- ⚠️ **Important warning straight from docs**: Don't disable auto-clustering on the destination table just because you're pre-clustering at ingest — doing so causes **degraded query performance over time**. Pre-clustering at ingest and background auto-clustering are meant to work together, not replace each other.

### Binary column encoding — a sneaky detail

Snowflake supports `BASE64`, `HEX`, `UTF-8` for binary columns.
- **Default pipe** → always `BASE64`, fixed, no matter what account/session parameters say.
- **Custom pipe** → follows the account-level `BINARY_INPUT_FORMAT` parameter.

⚠️ Gotcha: if you're using a custom pipe and someone changes `BINARY_INPUT_FORMAT` mid-session, the SDK's cached client becomes invalid — you must **close and reopen** it. Also, the SDK does **not** validate/infer string encoding for binary columns — mismatches are your responsibility to avoid.

---

## 7. Table Support & Schema

### Apache Iceberg support
Snowpipe Streaming (high-performance architecture) supports both **Iceberg v2 and v3** tables. This is a direct contrast point:

⚠️ **Exam trap — Classic vs. High-performance architecture on Iceberg**: 
- **Classic architecture** → Iceberg **v2 only**.
- **High-performance architecture** → Iceberg **v2 and v3**.
If a question mentions needing v3 support, the answer must involve the high-performance architecture.

### Schema evolution
Snowpipe Streaming supports **automatic schema evolution** — Snowflake can auto-add new columns detected in the incoming stream, and can drop NOT NULL constraints to accommodate new data shapes.

**Limitations (memorize these):**
- Only works on **standard Snowflake tables** — NOT external tables, NOT Iceberg tables.
- Can't automatically **increase precision/scale/length** of existing columns.
- Not supported for **structured data types** — but new columns containing structured data get inferred as **VARIANT** instead.

### Insert-only!
This is a big one: **the API only supports INSERT.** No update, no delete, no merge directly through Snowpipe Streaming.

**Workaround pattern** (classic CDC-style architecture, and a likely scenario question): write raw records into staging table(s) via streaming, then use a **continuous pipeline** (Streams + Tasks, or Dynamic Tables) to merge/transform/join into the final reporting table.

⚠️ **Exam trap**: "How do you UPDATE existing rows using Snowpipe Streaming?" → Trick question — **you can't directly**. You insert raw data, then process it downstream with Streams & Tasks.

### Supported Java data types (high level — don't need to memorize every row, but know the shape)
Common sense mapping: strings/numerics/booleans map flexibly; VARIANT/ARRAY/OBJECT accept JSON strings or native Java collection types; GEOGRAPHY/GEOMETRY are supported directly.

---

## 8. Operations, Monitoring & Privileges

### Monitoring
- **`SNOWPIPE_STREAMING_CHANNEL_HISTORY`** view (Account Usage) — channel state, offset progress, ingestion health.
- **`GET_CHANNEL_STATUS`** API — programmatic equivalent.

⚠️ Don't confuse this with classic Snowpipe's `COPY_HISTORY` — that's file-based load history and is a completely different view.

### Required privileges — a very testable table

| Object | Privilege Needed |
|---|---|
| Table | **OWNERSHIP**, or minimum **INSERT** + **EVOLVE SCHEMA** (EVOLVE SCHEMA only needed if using schema evolution with the Kafka connector) |
| Database | USAGE |
| Schema | USAGE |
| Pipe | **OPERATE** |

⚠️ **Exam trap**: People assume you need `CREATE PIPE` privilege to stream data. Not true if you're using the **default pipe** — you never issue CREATE PIPE for that. You just need OPERATE on the pipe (which gets auto-created) and INSERT on the table.

---

## 9. Client Configuration (SDK) — Conceptual Points Worth Knowing

Two categories of configuration:
1. **Process-wide environment variables** — control logging/metrics for the *whole app process*, must be set **before** the client initializes (e.g., `SS_ENABLE_METRICS`, `SS_LOG_LEVEL`).
2. **Client-side properties** — define connection + target (url, user, account, private_key, pipe name), scoped to one client object.

**Important structural fact**: a single application process can run **multiple client objects**, each with its own connection properties, but they all **share the same process-wide env-var settings** for logging/metrics. This is a subtle "scope" distinction the exam might probe.

**Authentication methods** (`authorization_type`):
- `JWT` (key-pair auth) — the **default**.
- `OAUTH` — needs client id/secret + refresh token.
- `PAT` (personal access token) — notably, **PAT auth doesn't require a `user` property** — that's a specific documented quirk worth remembering.

**Best practice**: externalize secrets (private key, OAuth credentials) into a proper key management service (e.g., AWS KMS) rather than hardcoding them.

---

## 10. Iceberg Tables Deep Dive

Beyond the v2/v3 distinction above:

- Snowpipe Streaming writes data as **Iceberg-compatible Parquet files** with Iceberg metadata, registered with **Snowflake acting as the Iceberg catalog**.
- Storage can be **Snowflake-managed** (no external volume needed) or **your own external volume** (you manage the cloud storage location).
- ⚠️ **Only Snowflake as catalog is supported** — externally-catalogued Iceberg tables (AWS Glue, Hive Metastore) are **not supported** for Snowpipe Streaming ingestion. (You *can* sync to Snowflake Open Catalog afterward, but that's a separate feature.)

**Iceberg-specific limitations (exam-relevant list):**
- **Partitioned Iceberg tables are NOT supported.**
- **Schema evolution is NOT supported** for Iceberg tables (contrast with standard tables, where it IS supported).
- **Length-constrained VARCHAR** (e.g. `VARCHAR(100)`) is **not supported** — must use unconstrained `STRING`/`VARCHAR`.

---

## 🔑 Master Comparison Table: Snowpipe Streaming vs. Classic Snowpipe

| Feature | Snowpipe Streaming | Snowpipe (Classic) |
|---|---|---|
| Data form | Rows | Files |
| Ordering guarantee | Within a channel | None |
| History view | `SNOWPIPE_STREAMING_CHANNEL_HISTORY` | `COPY_HISTORY` |
| Pipe object role | Active processor (validate/transform/cluster) | Queue manager for staged files |
| Iceberg support | v2 + v3 (high-perf architecture) | — |
| Operation type | Insert-only | Insert (via COPY) |

---

## 🧠 Summary — The 10 Things To Lock In

1. Rows flow directly into tables — **no staging, no file**.
2. Throughput up to **10 GB/s**, latency **as low as 5 sec**.
3. **Exactly-once** via **offset tokens** — user-managed bookmark, NOT the same as Snowflake's internal `continuationToken`.
4. Ordering guaranteed **within a channel only**.
5. Channels auto-delete after **30 days inactivity** — offset history can be lost.
6. **Default pipe** = zero-DDL, auto-named `<TABLE>-STREAMING`, can't transform data (`MATCH_BY_COLUMN_NAME` only).
7. Need transformations or pre-clustering? → **custom pipe** required.
8. **Insert-only API** — updates/deletes require downstream Streams & Tasks / merge pipeline.
9. Schema evolution works on standard tables only — not external, not Iceberg.
10. Iceberg: high-performance architecture supports v2 **and** v3; classic supports v2 only; partitioned Iceberg tables unsupported.

---

## ❓ Practice Questions

**Q1.** A team has 3 channels streaming into the same table. What ordering guarantee applies?
A) Global ordering across all 3 channels B) No ordering guarantee at all C) Ordering within each channel individually D) Ordering only for the first channel opened

*Answer: C*

**Q2.** Which token does Snowflake use internally to detect duplicate or missing data during an SDK retry on the Append Rows REST endpoint?
A) offsetToken B) continuationToken C) sessionToken D) channelToken

*Answer: B — offsetToken is user-managed and not used by Snowflake internally for this purpose.*

**Q3. (True/False)** The default pipe supports in-flight data transformations.

*Answer: False — it uses MATCH_BY_COLUMN_NAME and doesn't support transformations. You'd need a custom pipe.*

**Q4.** A channel has been inactive for 45 days. What happens to its offset token?
A) It's preserved indefinitely B) It's automatically deleted after 30 days of inactivity C) It resets to 0 D) It's archived to ACCOUNT_USAGE

*Answer: B*

**Q5.** Which Iceberg table configuration is NOT supported by Snowpipe Streaming (either architecture)?
A) Snowflake-managed storage B) External volume storage C) Partitioned Iceberg tables D) Iceberg v2

*Answer: C — partitioned Iceberg tables aren't supported under either architecture.*

**Q6.** You need to UPDATE existing rows in a table that's fed by Snowpipe Streaming. What's the correct approach?
A) Use appendRows with an UPDATE flag B) Stream raw inserts to a staging table, then merge via Streams & Tasks C) Not possible in Snowflake D) Use DROP and recreate the channel

*Answer: B — the API is insert-only; updates require a downstream pipeline.*

**Q7.** Which privilege on the PIPE object is required to call the Snowpipe Streaming API?
A) OWNERSHIP B) OPERATE C) USAGE D) MODIFY

*Answer: B*

---

That covers everything from your list under Snowpipe Streaming (I skipped the two pure-code walkthrough tutorial pages — getting-started and REST cURL tutorial — since they're step-by-step command instructions with no new concepts beyond what's captured above; happy to pull specific code from them if you want to actually implement something).

Want me to move on to **Virtual Warehouses** next, same style?
# 🌊 Snowflake Streams — SnowPro Core Study Notes

**Topic 1 of your Streams & Tasks series** · Sources: *Introduction to Streams* + *Manage Streams* (official Snowflake docs)

### How to read this document
- **⚠️ Exam Trap** = a specific way SnowPro loves to trick you on this point
- **🔑 Gotcha** = a real-world "this will bite you in production" behavior
- **💡 Real-world** = a concrete scenario so the concept sticks
- **🧩 Mix-up** = a misconception almost everyone has at first

---

## Why This Topic Matters

Snowflake has **no traditional triggers**. So how do you know *what changed* in a table since yesterday, without scanning the whole thing again? That's the exact problem Streams solve — they give you built-in Change Data Capture (CDC). This is one of the most heavily tested SnowPro topics because it's genuinely tricky (the offset behavior trips up even experienced engineers), and it's the foundation for the next topic, **Tasks** — most real pipelines are literally "a Task that runs when a Stream has data."

---

## Chapter 1 — What Problem Are We Even Solving?

Imagine a `raw_orders` table that gets thousands of inserts, updates, and deletes every day. A downstream job needs to update a `reporting_orders` table — but only for rows that actually changed. Without CDC, you have two bad options: reprocess the *entire* table every time (slow, wasteful), or bolt on manual `last_updated` timestamp columns and application-level logic (fragile, and it can't cleanly capture deletes).

A **Stream** is Snowflake's native answer: a database object that quietly watches a table (or view) and lets you ask, at any moment, "give me every row that changed since I last checked."

**Objects you're allowed to put a stream on:** standard tables (including tables shared to you from another account), views (including secure views), directory tables, dynamic tables, Apache Iceberg™ tables (with some limits), event tables, and external tables.

> **🧩 Mix-up:** People assume *materialized views* are fair game too since they're "just a special view." They are **not** — Snowflake explicitly does not support tracking changes on materialized views. If a question asks "which of these can a stream NOT be created on," materialized view is the answer to watch for.

---

## Chapter 2 — What a Stream Actually *Is* (and Isn't)

This is the single most important mental model in this whole topic, so let's get it right.

**A stream stores zero table data.** It only stores one thing: an **offset** — basically a bookmark marking a point in time in the table's history. Think of the table as a very long, constantly-growing book. The stream doesn't photocopy any pages; it just remembers which page you last read. Whenever you "read the stream," Snowflake flips to your bookmark, reads forward to the current page, and hands you everything that happened in between.

Here's what makes that possible: the moment you create the **first** stream on a table, Snowflake quietly adds a few **hidden change-tracking columns** to that source table. These columns record the metadata needed to reconstruct history. They cost a small amount of storage — but only once per table, not once per stream.

```
Table  ──►  gets hidden change-tracking columns (added once, on first stream)
Stream ──►  just stores an offset (a timestamp/version pointer)

Querying a stream = offset (from the stream) + change-tracking metadata (from the table)
```

> **🔑 Gotcha:** Because a stream is so lightweight, creating 5 or 10 streams on the same table is *cheap*. This directly matters for Chapter 8 (Multiple Consumers) — the fix for that problem is "just make more streams," and this is *why* that fix is affordable.

> **⚠️ Exam Trap:** "Streams may read deleted data to reconstruct history, so the stream replays rows using the table's *current* schema." If you later change the table's structure — say you add a `NOT NULL` constraint on a column, or drop a column — old historical rows might not satisfy that new schema anymore, and **querying the stream can outright fail**. This is a real documented limitation, not a hypothetical.

---

## Chapter 3 — Offsets, Table Versions, and the Golden Rule

Every time a transaction with a DML statement commits against a table, Snowflake stamps out a new **table version**. A stream's offset always sits *between* two of these versions.

```
Table timeline:   v1 → v2 → v3 → v4 → v5 → ... → v9 → v10 (latest)
                            ▲                            ▲
                     stream's offset                    "now"
                     (sitting between v3 and v4)

Querying the stream right now → returns everything from v4 through v10
```

Now for **the golden rule**, the fact that SnowPro questions this the most:

> **A `SELECT` on a stream, by itself, never moves the offset — even inside an explicit transaction.** The offset only advances when the stream is read *as part of a DML statement that successfully commits.*

"DML" here is broader than people expect — it includes `INSERT`, `MERGE`, `UPDATE`, `DELETE`, a `CREATE TABLE ... AS SELECT` (CTAS), and even a `COPY INTO <location>` (unloading to a stage). All of these consume the stream if they commit.

```
DML statement reads FROM a stream
        │
        ▼
Does the transaction commit successfully?
        │
   ┌────┴─────┐
  YES          NO
   │            │
   ▼            ▼
Offset        Offset stays
advances      exactly where
              it was
```

> **💡 Real-world:** An analyst runs `SELECT * FROM order_stream;` fifty times a day just to "peek" at what's pending. No matter how many times they do this, nothing is consumed — the actual ETL job's `MERGE` statement is still the only thing that will ever move the bookmark forward.

**Want to advance the offset without doing real work with the data?** (Useful for "flushing" a stream you don't actually need right now.) Two documented tricks:
1. `CREATE OR REPLACE STREAM` — just recreate it at the current position.
2. Run a throwaway DML statement that consumes but discards everything:

```sql
CREATE TEMPORARY TABLE stream_flush_dummy AS
SELECT * FROM my_stream WHERE 1 = 0;
```

The `WHERE 1 = 0` guarantees zero rows actually get processed, but because it's a CTAS (a DML transaction), the offset still jumps forward. Keep this snippet in your back pocket — it shows up again in Chapter 7 as the fix for a very specific staleness problem.

> **⚠️ Exam Trap:** "If I query a stream inside an explicit transaction, which timestamp does it use — when the transaction *began*, or when the `SELECT` actually ran?" Answer: **transaction start time**, not statement-execution time. This detail is the whole basis of Chapter 4.

---

## Chapter 4 — Repeatable Read: Why a Transaction "Freezes" the Stream

Normal Snowflake tables give you **read committed** isolation — inside one transaction, a later statement *can* see changes committed by an earlier statement in that same transaction. Streams behave differently: they give you **repeatable read**.

In plain English: once your transaction begins and you query a stream, every subsequent query to that *same* stream *within that same transaction* returns the identical set of rows — frozen at the moment the transaction started — no matter how many times you ask, and no matter what anyone else commits to the table in the meantime.

```
Session A: BEGIN transaction  ─────────────────────────────► COMMIT
              │                                                  │
              ├─ query stream: sees rows X, Y                    │
              ├─ (someone else commits new changes elsewhere)    │
              ├─ query stream AGAIN: still sees only X, Y        │
              └──────────────────────────────────────────────────┘
                            offset only NOW advances, to
                            the transaction's start time
```

Using the stream inside a DML statement effectively **locks** it — other sessions' changes to the source table keep getting tracked by the underlying change-tracking system, but they won't become part of "the stream's official state" until your transaction finishes committing.

> **⚠️ Exam Trap:** Two transactions, A and B. Transaction A starts, then B starts (before A commits), then A commits. Question: does B's view of the stream include A's changes? **No** — B's stream view is frozen at B's own start time, which was *before* A committed. B will not see A's changes even though A finished while B was still open. If B had started *after* A committed, then yes, B would see them.

---

## Chapter 5 — The Metadata Columns

When you query a stream, you get the source object's normal columns *plus* three extra ones:

| Column | What it tells you |
|---|---|
| `METADATA$ACTION` | `INSERT` or `DELETE` — the operation that happened |
| `METADATA$ISUPDATE` | `TRUE`/`FALSE` — was this part of an `UPDATE`? (An update is represented internally as a paired DELETE + INSERT) |
| `METADATA$ROW_ID` | A stable, unique ID for that row, used to track it across changes over time |

> **⚠️ Exam Trap (very commonly tested):** A row is **inserted**, then **updated**, both before the stream is consumed. How does the stream show it? Not as an insert-then-update pair — it nets out to a **single `INSERT` row with `METADATA$ISUPDATE = FALSE`**. The stream only shows the net *delta* between two offsets, not a literal replay of every statement that happened.

**Three guarantees about `METADATA$ROW_ID`** worth memorizing:
1. Two different streams on the *same table* produce the same row IDs for the same rows.
2. A stream on a table and a stream on a **clone** of that table produce matching row IDs for rows that existed at clone time.
3. A stream on a table and a stream on its **replica** produce matching row IDs for replicated rows.

> **🧩 Mix-up:** People assume a stream on `view1` (defined as a plain `SELECT * FROM table1`) will produce the *same* row IDs as a stream directly on `table1`, since the view "is basically the same data." **Not guaranteed.** Row-ID matching is only promised between two streams on the exact same underlying object, table clones, or replicas — not automatically across a view boundary.

> **🔑 Gotcha:** If `CHANGE_TRACKING` is disabled and then re-enabled later on the source object, `METADATA$ROW_ID` values can change going forward — the "immutable" guarantee has a hidden asterisk.

---

## Chapter 6 — Three Kinds of Streams

This is a big exam area. There are exactly three types, and they're distinguished by *what* they choose to track.

| | **Standard** (delta) | **Append-only** | **Insert-only** |
|---|---|---|---|
| Tracks | INSERT, UPDATE, DELETE, TRUNCATE | INSERT only | New "insert-like" file arrivals only |
| Can sit on | Standard tables, dynamic tables, Snowflake-managed Iceberg tables, externally-managed V3 Iceberg tables, directory tables, views | Standard tables, dynamic tables, Snowflake-managed Iceberg tables, views | External tables, externally-managed Iceberg tables, Delta Direct tables *without* partition columns |
| Nets out insert+delete of the same row? | Yes — if inserted then deleted before consumption, it vanishes from the delta entirely | N/A (deletes are invisible to it anyway) | N/A |
| Performance | Normal | Faster — cheaper to compute since it never has to reconcile deletes | N/A (file-level, not row-level) |
| Special restriction | Cannot retrieve change data for **geospatial** columns | Cannot be created on a secondary (replicated) object in a target account | Doesn't compute a real diff between old/new file versions |

> **💡 Real-world:** Use **standard** for a `customers` table where you need full fidelity (someone edited an address, someone else was deleted for GDPR). Use **append-only** for a high-volume `clickstream_events` staging table that only ever receives inserts and gets truncated periodically — the stream doesn't care about the truncate, and it's measurably faster. Use **insert-only** for an external table pointed at a cloud storage bucket collecting sensor-log file drops.

> **⚠️ Exam Trap:** A table stores geospatial (`GEOGRAPHY`/`GEOMETRY`) data and needs CDC. Which stream type? **Append-only** — standard streams explicitly cannot retrieve change data for geospatial columns.

> **🔑 Gotcha (insert-only specifics):** If `File1` is deleted from cloud storage and `File2` is added in its place, the insert-only stream shows *only* `File2`'s rows as inserts — the removal of `File1` is a complete no-op, never recorded as a delete. Overwriting a file is handled the same way: old version quietly disappears, new version's rows appear as fresh inserts. **No diff is computed between the old and new file** — so if you're not careful about how your pipeline consumes this, you can double-count rows that existed in both versions.

> **⚠️ Exam Trap:** Streams are **not supported at all on partitioned external tables** (insert-only or otherwise). And you can't use standard/append-only streams on an Iceberg table that's on an *external* catalog — only insert-only works there.

---

## Chapter 7 — Staleness: The #1 Tested Gotcha

A stream goes **stale** when its offset falls outside the source table's data retention window. Once that happens, the historical versioning data needed to compute the delta is simply gone — the stream is permanently broken and must be recreated with `CREATE STREAM`, losing whatever changes hadn't been consumed yet.

**Snowflake tries to protect you from this automatically.** If a table's `DATA_RETENTION_TIME_IN_DAYS` is set lower than 14, Snowflake temporarily *extends* that table's retention — up to whatever `MAX_DATA_EXTENSION_TIME_IN_DAYS` allows — specifically so an unconsumed stream doesn't die from a short retention setting. The moment the stream finally gets consumed, retention reverts to the table's normal setting.

**The real rule to memorize:**

```
Effective "safe to wait" window = MAX(DATA_RETENTION_TIME_IN_DAYS, MAX_DATA_EXTENSION_TIME_IN_DAYS)
```

- Retention = 5 days, max extension = 14 days → you effectively get **14 days** (extension wins)
- Retention = 20 days, max extension = 7 days → you effectively get **20 days** (base retention wins, since it already beats the extension cap)

> **⚠️ Exam Trap:** "A table has `DATA_RETENTION_TIME_IN_DAYS = 0` (e.g., Standard Edition, no Time Travel). Does that mean any stream on it is basically useless?" **No!** If `MAX_DATA_EXTENSION_TIME_IN_DAYS` is set to something like 90, the stream can still survive up to ~90 days unconsumed. Retention = 0 does not automatically mean the stream dies instantly.

**Checking staleness:** run `SHOW STREAMS` or `DESCRIBE STREAM`. Two columns matter:
- **`STALE_AFTER`** — a predicted timestamp, calculated as *last consumption time* + the effective window above. If it's in the past, the stream is probably already stale.
- **`STALE`** — whether the stream is *expected* to be stale. 

> **🔑 Gotcha:** These are both best-effort signals, not guarantees. Reads can keep succeeding for a while *after* `STALE_AFTER` has passed — but the stream **can go stale at any moment** once you're past that point. Don't build logic that trusts a stream's results once it's past `STALE_AFTER`, and don't fully trust `SYSTEM$STREAM_HAS_DATA()` results at that stage either.

> **🔑 Gotcha:** "Zero rows returned" doesn't always mean "nothing happened." An append-only stream ignores updates/deletes, so it can show 0 rows even after a table had heavy update activity. Also, some operations — like automatic reclustering — never generate change records at all. And here's the sneaky part: **consuming a stream always advances the offset to the present, whether or not there was anything to consume.** A "just in case" dummy consume still moves your bookmark forward, even past changes you never actually looked at.

**What breaks a stream vs. what doesn't:**

| Action | Effect on the stream |
|---|---|
| `CREATE OR REPLACE TABLE` on the source | **Breaks it** — stream goes stale immediately (table history was reset) |
| Recreating/dropping any underlying table of a source *view* | **Breaks** any stream on that view |
| `RENAME` the source object | **Safe** — renaming does not break or stale a stream |
| `DROP` the object, then create a *new* object with the *same name* | Old streams stay linked to the (now-gone) original object — they do **not** silently reattach to the new one |
| Cloning a schema/DB containing a stream + its source table together | Any **unconsumed** records in the cloned stream become inaccessible — consistent with how Time Travel resets on a fresh clone |

> **⚠️ Exam Trap:** This rename-vs-recreate distinction is a favorite. "Renaming = safe. `CREATE OR REPLACE` = stale." Also: same table name ≠ same table object — streams bind to the underlying object identity, not the name string.

**One more carve-out:** streams sitting on tables/views that were **shared to you** from another Snowflake account do **not** cause the extension of that provider's retention period. This exists so that a lazy consumer on your side can't quietly inflate someone else's storage bill.

---

## Chapter 8 — The Multiple-Consumers Trap

This one bites real production pipelines constantly. A stream's offset is a **single shared bookmark**. If Consumer A's DML commits against the stream, the offset moves — permanently, for everyone.

> **💡 Real-world:** A single `orders_stream` is wired up to *both* a real-time dashboard-refresh job *and* a nightly warehouse-load job. Whichever one's DML commits first "eats" the change data. The second job queries the same stream afterward and finds... nothing. Its changes are gone, already consumed by the other job.

**The fix, straight from Snowflake's own guidance:** create a **separate stream per consumer**. Since a stream is nearly free (Chapter 2 — no real data stored, hidden columns already exist after the first stream), there's no meaningful cost penalty to having several streams pointed at the same table.

```
                     ┌── Stream_A ──► Task A (dashboard refresh)
   Source Table ─────┤
                      └── Stream_B ──► Task B (nightly load)

Each stream tracks its OWN independent offset — A consuming
doesn't affect B's ability to see the same changes.
```

> **⚠️ Exam Trap:** "Is it expensive to create multiple streams on one table for different downstream jobs?" — **No.** This is explicitly called out as a non-issue, precisely because streams store offsets, not data.

---

## Chapter 9 — Streams on Views (Extra Rules Apply)

You *can* put a stream on a view — but only if it clears a checklist first.

**Underlying tables must:**
- Be native Snowflake tables (not external tables underneath, etc.)

**The view itself can only use:**
- Projections, filters, inner/cross joins, `UNION ALL`

**Not allowed in the view:**
- `GROUP BY`, `QUALIFY`, subqueries outside the `FROM` clause, correlated subqueries, `LIMIT`, `DISTINCT`
- Select-list functions must be plain system-defined scalar functions (no UDFs, no aggregates)

**Before creating the stream:** change tracking must be explicitly enabled on the view *and* on its underlying tables — unless the same owning role controls both, in which case creating the stream via that role enables tracking everywhere in one shot. Only the **object owner** (the role holding `OWNERSHIP`) can flip on change tracking — having `ALTER` alone isn't enough.

> **🔑 Gotcha:** Turning on `CHANGE_TRACKING` via `ALTER` **locks the object** for the duration of the change, which can cause latency for any concurrent DML/DDL. Plan this for a quiet maintenance window on busy tables.

### The join-delta gotcha

Say a view joins `students` with `grades` on `student_id`, and each currently has one matching row. If you insert one *new* matching row into `grades`, the stream shows one new combined row: the new grades row joined with the *existing* students row (Δgrades × students).

But if you'd instead inserted a new matching row into **both** tables in that same window, the delta wouldn't be just one row — it'd be **three**: new-student × old-grades, old-student × new-grades, **and** new-student × new-grades (all the cross-combinations get surfaced). This is because a stream on a joined view has to account for changes hitting *either side* of the join, plus both sides changing together.

> **⚠️ Exam Trap:** This "explodes into multiple delta rows" behavior is very easy to get wrong intuitively — people expect "1 new row inserted = 1 new stream row," but a join can multiply that.

> **⚠️ Exam Trap, and a preview of the next topic:** If a **Task** is triggered by a stream on a view, *any* change to *any* table the view's query touches will fire that task — regardless of whether the view's own filters or joins would have actually surfaced a visible row. The trigger doesn't pre-check the view's logic; it fires eagerly. Keep this in your back pocket for the Tasks topic.

---

## Chapter 10 — The `CHANGES` Clause: Streams' Quieter Cousin

Sometimes you don't want a standing, offset-tracking object at all — you just want a one-off answer to "what changed between timestamp A and timestamp B?" That's what the `CHANGES` clause on a `SELECT` statement is for.

**Key difference from a stream:** the `CHANGES` clause is purely **read-only** — it never advances or consumes anything. Multiple people can query the exact same time window as many times as they want with zero side effects (the opposite of Chapter 8's problem!). You specify the start point with `AT | BEFORE`, and optionally an `END` point.

**Requirement:** change tracking metadata must already exist for the period you're asking about — either by having `CHANGE_TRACKING = TRUE` on the table, or by having ever created a stream on it (which enables the same underlying tracking).

| | **Stream** | **`CHANGES` clause |
|---|---|---|
| Stateful? | Yes — remembers an offset | No — you supply both endpoints each time |
| Consumes on DML? | Yes | Never |
| Best for | Repeated, automated pipelines (paired with Tasks) | Ad hoc analysis, arbitrary custom time windows, auditing |

> **⚠️ Exam Trap:** "Which option should you use if 5 different analysts need to independently check the same change window without any of them affecting the others?" → `CHANGES` clause, not a stream (streams have the single-shared-offset problem from Chapter 8).

---

## Chapter 11 — Who's Allowed to Query a Stream?

| Object | Privilege needed |
|---|---|
| Database | USAGE |
| Schema | USAGE |
| **Stream** | **SELECT** |
| Table (streams on tables) | SELECT |
| View (streams on views) | SELECT |
| External stage (streams on directory tables, external stage) | USAGE |
| Internal stage (streams on directory tables, internal stage) | READ |

> **⚠️ Exam Trap:** People assume "if I can `SELECT` the underlying table, I can automatically query any stream on it." **False** — the stream object needs its *own* explicit `SELECT` grant, separate from the grant on the table or view it watches.

*(A "directory table" here just means the file-metadata layer Snowflake maintains for files sitting in a stage — that's why stage-level privileges show up in this table at all.)*

---

## Chapter 12 — What Do Streams Actually Cost?

- **The stream object itself:** essentially free — no dedicated stream storage fee, no separate "stream pricing."
- **Storage cost, indirectly:** if you don't consume a stream, Snowflake may have to extend the source table's Time Travel retention (Chapter 7) to keep it from going stale — and *that* extended retention means more historical micro-partitions are kept around, which does show up in your storage bill.
- **Compute cost:** ordinary warehouse credits for whatever query/DML you run to read or consume the stream — no special stream-specific compute charge.

> **⚠️ Exam Trap:** "Do streams have their own billing model?" → No. Cost only shows up indirectly, through (a) possible storage-retention extension, and (b) normal compute time.

---

## Chapter 13 — Grab-Bag of Official Limitations

- No standard or append-only streams on Iceberg tables using an **external** catalog (insert-only is the only option there).
- Can't consume append-only streams on Snowflake-managed Iceberg V2/V3 tables if the changes came from an **external engine**.
- On those same externally-modified V2 Iceberg tables, a standard (delta) stream treats the changes as brand-new delete+insert pairs, not a smart merge.
- Can't track a view containing `GROUP BY`.
- Adding or altering a column to `NOT NULL` can break stream queries if historical delta rows contain `NULL`s that now violate the new constraint (this connects back to the schema-drift gotcha from Chapter 2).
- Streams-on-views + triggered Tasks over-fire, as covered in Chapter 9.
- Not supported on **partitioned** external tables, period.

---

## Chapter 14 — Creating & Administering Streams Day-to-Day

**Enabling change tracking** happens one of two ways:
1. Just create the stream using the view-owner's role — if that role also owns the underlying tables, tracking switches on everywhere automatically.
2. Explicitly turn it on ahead of time:

```sql
-- On a view (at creation)
CREATE SECURE VIEW my_view CHANGE_TRACKING = TRUE AS SELECT col1, col2 FROM my_table;

-- On a view (after the fact)
ALTER VIEW my_view SET CHANGE_TRACKING = TRUE;

-- On a table (at creation)
CREATE TABLE my_table (col1 STRING, col2 NUMBER) CHANGE_TRACKING = TRUE;

-- On a table (after the fact)
ALTER TABLE my_table SET CHANGE_TRACKING = TRUE;
```

> **🔑 Gotcha:** Only the role with **OWNERSHIP** can do this — and running the `ALTER` locks the object for the duration, which can stall concurrent DML/DDL. Don't run this against a busy production table at peak hours.

**Avoiding staleness in practice — the `SYSTEM$STREAM_HAS_DATA()` trap:**

Calling `SYSTEM$STREAM_HAS_DATA()` on an *empty* stream prevents it from going stale, but only while it keeps returning `FALSE`.

> **⚠️ Exam Trap (this is emphasized twice in the official docs, so it's almost certainly tested):** If `SYSTEM$STREAM_HAS_DATA()` returns `TRUE`, you're expected to go ahead and consume the stream via DML — **even if it turns out to be a false positive with nothing meaningful inside.** If you skip that consumption because "eh, probably nothing," any Task whose `WHEN` clause checks this function will just **keep firing forever**, burning warehouse credits every single run for no benefit, since the function keeps saying `TRUE`.

The fix is the same dummy-consume trick from Chapter 3:

```sql
CREATE TEMPORARY TABLE stream_flush_dummy AS
SELECT * FROM my_stream WHERE 1 = 0;
```

**Snowsight UI (brief):** *Catalog → Database Explorer → (your database/schema) → Streams → pick your stream.* From there you can see which table it watches, its type, whether it's stale, the exact SQL used to create it, and manage its privileges.

---

## 🎯 Final Revision Sheet — Read This the Night Before the Exam

1. A stream stores **only an offset**, never actual row data.
2. `SELECT` alone **never** consumes a stream — only DML that commits does (`INSERT`, `MERGE`, `UPDATE`, `DELETE`, CTAS, `COPY INTO <location>`).
3. Inside an explicit transaction, the stream is frozen at the **transaction's start time**, repeatable-read style — not read-committed like normal tables.
4. Insert-then-update in the same window nets out to **one INSERT row**, `METADATA$ISUPDATE = FALSE`.
5. Three types: **Standard** (all DML, nets insert+delete pairs), **Append-only** (inserts only, faster), **Insert-only** (external tables/Iceberg-external/Delta Direct, file-level, no true diffing).
6. Standard streams **cannot** handle geospatial data — use append-only instead.
7. Staleness = offset falls outside the retention window. Effective safe window = `MAX(DATA_RETENTION_TIME_IN_DAYS, MAX_DATA_EXTENSION_TIME_IN_DAYS)`. A 0-day retention table can still protect its stream if the extension parameter is high.
8. `CREATE OR REPLACE TABLE` **breaks** streams. **Renaming does not.**
9. One stream = one shared offset. Multiple independent consumers need **their own separate streams**, and that's cheap to do.
10. Materialized views are **not** supported for streams. Regular views need change tracking explicitly enabled first.
11. A stream on a joined view can multiply one row change into **multiple delta rows** (cross-combinations).
12. `CHANGES` clause = the stateless, non-consuming alternative to a stream — great for ad hoc, repeatable checks.
13. Querying the stream object itself needs its own `SELECT` grant — separate from the table/view grant.
14. Streams have no dedicated fee; cost is indirect (extended retention storage + normal compute).
15. `SYSTEM$STREAM_HAS_DATA() = TRUE` still means "go consume it," even on a false positive — skipping this causes runaway Task executions.

---

## 📊 Quick Comparison: Stream vs. `CHANGES` Clause

| | Stream | `CHANGES` clause |
|---|---|---|
| Stores an offset? | Yes | No |
| Consumes on DML? | Yes | Never |
| Good for | Repeated automated pipelines (Tasks) | One-off / ad hoc / auditing |
| Multiple independent readers? | Problematic (Ch. 8) unless you make separate streams | No problem — fully read-only |

---

## 🧪 Practice Questions

*Cover the answer and try each one yourself first.*

**Q1.** You open an explicit transaction, run only `SELECT * FROM my_stream;`, then `COMMIT` — no other statements touch the stream. What happens to the stream's offset?

A) It advances to the transaction's start time, because the transaction was committed.
B) It stays exactly where it was, because a plain SELECT never consumes stream data.
C) It advances only if `SYSTEM$STREAM_HAS_DATA` had already returned TRUE for that stream.
D) It rolls back further, since no rows in the source table actually changed.

**Answer: B.** A `SELECT` never consumes a stream, no matter how it's wrapped. Only DML that commits moves the offset.

---

**Q2.** A row is inserted into a table and then updated, both before a standard stream on that table is consumed. How does this row appear when you query the stream?

A) As a single INSERT record, with `METADATA$ISUPDATE` set to FALSE.
B) As a DELETE and an INSERT record, both marked as updates.
C) As two separate INSERT records, one per statement that ran.
D) As nothing, since the insert and the update cancel out.

**Answer: A.** Streams show the *net* delta between offsets, not a literal replay — insert-then-update collapses into one INSERT with `ISUPDATE = FALSE`.

---

**Q3.** A table has `DATA_RETENTION_TIME_IN_DAYS` set to 0, but `MAX_DATA_EXTENSION_TIME_IN_DAYS` is set to 90. Roughly how long can an unconsumed stream on it survive before going stale?

A) It goes stale almost immediately, since base retention is zero.
B) About 7 days, Snowflake's default grace period for any stream.
C) Exactly 1 day, matching the platform's minimum offset guarantee.
D) Up to about 90 days, based on the extension setting.

**Answer: D.** The effective window is the *larger* of the two parameters — here, the 90-day extension dominates the 0-day retention.

---

**Q4.** Two different downstream jobs both need the full set of changes from the same source table, independently of each other. What's the recommended setup?

A) Point both jobs at one shared stream, since reads don't move the offset.
B) Use one stream, but always run the slower job before the faster one.
C) Create a separate stream on that table for each consuming job.
D) Wrap both jobs in a single transaction so they share a view.

**Answer: C.** One stream = one shared offset. Whichever job's DML commits first consumes the data for good — separate streams avoid the conflict entirely, and streams are cheap enough to duplicate freely.

---

**Q5.** You run `CREATE OR REPLACE TABLE` on a table that has an active stream attached to it. What happens to that stream?

A) It becomes stale immediately, because the table's history was reset.
B) It pauses until the next scheduled task run refreshes it.
C) It keeps working normally, since the table name didn't change.
D) It's silently dropped and removed from the schema entirely.

**Answer: A.** Recreating the table wipes its history, and any stream depending on that history instantly goes stale — even though the name is unchanged.

---

**Q6.** You want to create a stream directly on a view that joins two native tables. What must already be true for this to work?

A) The view must include a QUALIFY clause to dedupe join keys.
B) The two underlying tables must first be merged into one.
C) The view must be owned by the ACCOUNTADMIN role specifically.
D) Change tracking must be enabled on the view and its underlying tables.

**Answer: D.** QUALIFY clauses are actually disallowed in a stream-eligible view, not required — change tracking on the view *and* its base tables is the real prerequisite.

---

**Q7.** An append-only stream sits on a staging table. After the stream is consumed, every row is deleted and 5 new rows are inserted. What does querying the stream show next?

A) All the deleted rows, followed by the 5 new inserted rows.
B) Only the 5 newly inserted rows, since deletes aren't tracked.
C) Nothing at all, because deletes clear an append-only stream's state.
D) The 5 rows, but flagged as updates instead of inserts.

**Answer: B.** Append-only streams are blind to deletes and truncates entirely — they only ever surface inserted rows.

---

**Q8.** `SYSTEM$STREAM_HAS_DATA()` returns TRUE for a stream, but querying it happens to return no meaningful rows this time. What should you do?

A) Ignore it, since a TRUE value with no rows is always a bug.
B) Lower `MAX_DATA_EXTENSION_TIME_IN_DAYS` so this false positive stops recurring.
C) Consume the stream through a DML statement anyway, to keep the offset current.
D) Wait for the next scheduled run before taking any further action.

**Answer: C.** Skipping consumption on a "false positive" leaves the function returning TRUE forever, causing a dependent Task to keep re-firing needlessly.

---

**Q9.** On an external table, a file is removed from cloud storage and replaced by a new file between two stream offsets. How does an insert-only stream represent this?

A) As one delete record for the old file and one insert for the new.
B) As a row-level diff of exactly what changed between the two files.
C) As nothing, since insert-only streams ignore all file-level changes entirely.
D) As insert records for the new file's rows only, with no delete shown.

**Answer: D.** Insert-only streams never record removals — the old file's disappearance is a silent no-op, and only the new file's rows show up, as inserts.

---

**Q10.** A team wants to check what changed between two arbitrary timestamps on a table, without creating a lasting object or touching any existing stream's offset. Which fits best?

A) Querying the table directly with the `CHANGES` clause on a SELECT.
B) A temporary stream that gets dropped right after being used.
C) The `SYSTEM$STREAM_HAS_DATA` function called directly on the table.
D) A standard stream with `MAX_DATA_EXTENSION_TIME_IN_DAYS` explicitly set to 0.

**Answer: A.** The `CHANGES` clause is purpose-built for exactly this — stateless, repeatable, and it never consumes anything.

---

## What's Next

That's the whole Streams picture — mechanics, all 3 types, staleness, the multiple-consumer trap, views, and the admin side. Tasks is next, and it's a much bigger unit (you've linked 11 separate doc pages for it), so we'll likely want to split it into a couple of passes rather than one giant chunk.

Whenever you're ready: say the word to move on to **Tasks**, or stick around here if you want me to go deeper on any one chapter, drill you with more scenario questions, or clear up anything that didn't fully click.
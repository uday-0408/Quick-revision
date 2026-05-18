# 🚀 MODULE 7: CONCURRENCY & LOCKING — COMPLETE REVISION NOTES

> **Oracle Locking · Deadlocks · MVCC · SELECT FOR UPDATE · Isolation Levels · Blocking Sessions**

---

# 📌 MODULE GOAL

This module teaches how Oracle handles:

* multiple users
* simultaneous updates
* safe transactions
* locking
* consistency
* deadlock prevention

This is one of the **most important real-world Oracle DBA + backend development topics**.

---

# 🧠 WHY CONCURRENCY MATTERS

In real systems:

* thousands of users access DB simultaneously
* many transactions happen together

Without concurrency control:

* data corruption occurs
* updates overwrite each other
* inconsistent reads appear

---

# 🚨 COMMON CONCURRENCY PROBLEMS

| Problem             | Meaning                            |
| ------------------- | ---------------------------------- |
| Lost Update         | One transaction overwrites another |
| Dirty Read          | Reading uncommitted data           |
| Non-repeatable Read | Same query gives different results |
| Phantom Read        | New rows appear between queries    |

---

# 🧠 ORACLE'S CONCURRENCY PHILOSOPHY

---

# 📌 Oracle Rules

| Rule                            | Meaning                              |
| ------------------------------- | ------------------------------------ |
| Readers never block writers     | SELECT doesn't stop UPDATE           |
| Writers never block readers     | UPDATE doesn't stop SELECT           |
| Writers block writers           | Only same-row modifications conflict |
| Locks held till COMMIT/ROLLBACK | Predictable behavior                 |

---

# ⚡ KEY ORACLE FEATURE

Oracle uses:

```text
MVCC (Multi-Version Concurrency Control)
```

This is the heart of Oracle concurrency.

---

# 🧠 MVCC (MULTI-VERSION CONCURRENCY CONTROL)

---

# 📌 Main Idea

Oracle keeps:

* current data
* old versions in UNDO

This allows:

```text
consistent reads without blocking
```

---

# ⚡ Example Flow

---

## Time T1

Session A:

```sql
SELECT salary FROM employees;
```

Sees:

```text
50000
```

---

## Time T2

Session B:

```sql
UPDATE employees
SET salary = 55000;
```

Old value stored in:

```text
UNDO SEGMENT
```

---

## Time T3

Session A still sees:

```text
50000
```

Because Oracle reconstructs old version using UNDO.

---

# 🚨 Important Interview Point

Oracle achieves:

```text
high concurrency WITHOUT read locks
```

using MVCC.

---

# 🔒 LOCKS vs LATCHES

---

# 📌 LOCKS

Protect:

```text
user data
```

Examples:

* rows
* tables

---

# 📌 LATCHES

Protect:

```text
internal memory structures
```

Examples:

* shared pool
* buffer cache

---

# ⚡ Comparison

| Locks                | Latches               |
| -------------------- | --------------------- |
| User data            | Oracle memory         |
| Transaction duration | Microseconds          |
| Can deadlock         | No deadlock detection |
| User-visible         | Internal              |

---

# 📌 LOCK TYPES

---

# 🧠 TX LOCK

Transaction lock.

Used for:

```text
row-level locking
```

Acquired during:

* INSERT
* UPDATE
* DELETE

---

# 🧠 TM LOCK

Table-level lock.

Automatically acquired during DML.

---

# ⚡ IMPORTANT

Oracle primarily uses:

```text
ROW-LEVEL LOCKING
```

This minimizes contention.

---

# 🧠 ROW-LEVEL LOCKING

---

# 📌 Example

Session 1:

```sql
UPDATE employees
SET salary = 50000
WHERE employee_id = 100;
```

Row 100 locked.

---

Session 2:

```sql
UPDATE employees
SET salary = 60000
WHERE employee_id = 101;
```

✅ succeeds

Different row.

---

Session 2:

```sql
UPDATE employees
SET salary = 60000
WHERE employee_id = 100;
```

❌ waits

Same row locked.

---

# 🧠 TABLE LOCK MODES

| Mode | Meaning             |
| ---- | ------------------- |
| RS   | Row Share           |
| RX   | Row Exclusive       |
| S    | Share               |
| SRX  | Share Row Exclusive |
| X    | Exclusive           |

---

# 📌 Most Common

DML operations usually use:

```text
ROW EXCLUSIVE (RX)
```

---

# 📌 EXCLUSIVE LOCK

Blocks:

* reads needing locks
* writes
* DDL

---

# 📌 Explicit Table Lock

```sql
LOCK TABLE employees
IN EXCLUSIVE MODE;
```

---

# ⚡ NOWAIT

```sql
LOCK TABLE employees
IN EXCLUSIVE MODE NOWAIT;
```

Fails immediately if locked.

---

# 🧠 DEADLOCKS

---

# 📌 What is Deadlock?

Circular waiting condition.

---

# ⚡ Example

---

## Session 1

Locks:

```text
Row A
```

Needs:

```text
Row B
```

---

## Session 2

Locks:

```text
Row B
```

Needs:

```text
Row A
```

---

# 🚨 Result

```text
DEADLOCK
```

---

# 📌 Oracle Behavior

Oracle automatically:

```text
kills one transaction
```

Error:

```text
ORA-00060
```

---

# 🧠 DEADLOCK PREVENTION

---

# ✅ Strategy 1 — Consistent Lock Order

Always lock rows:

```text
in same order
```

Example:

```text
lowest account_id first
```

---

# ✅ Strategy 2 — Lock Everything Early

Acquire all locks at transaction start.

---

# ✅ Strategy 3 — Retry Logic

Use:

* NOWAIT
* retries
* backoff

---

# 📌 Exponential Backoff

```sql
DBMS_LOCK.SLEEP(POWER(2, retry));
```

---

# ✅ Strategy 4 — Short Transactions

Never hold locks during:

* API calls
* emails
* long processing

---

# 🚨 BAD

```sql
UPDATE accounts ...

external_api_call();

COMMIT;
```

Locks held too long.

---

# ✅ GOOD

```sql
UPDATE accounts ...
COMMIT;

external_api_call();
```

---

# 🧠 SELECT FOR UPDATE

---

# 📌 Purpose

Locks selected rows for future modification.

---

# 📌 Syntax

```sql
SELECT *
FROM employees
FOR UPDATE;
```

---

# 📌 Behavior

Other sessions:

```text
cannot modify locked rows
```

until:

* COMMIT
* ROLLBACK

---

# 📌 FOR UPDATE OF

```sql
FOR UPDATE OF salary
```

Locks only relevant table/columns.

---

# 🧠 FOR UPDATE OPTIONS

---

# 📌 NOWAIT

```sql
FOR UPDATE NOWAIT
```

Fail immediately.

Error:

```text
ORA-00054
```

---

# 📌 WAIT n

```sql
FOR UPDATE WAIT 5
```

Wait max 5 seconds.

---

# 📌 SKIP LOCKED

```sql
FOR UPDATE SKIP LOCKED
```

Skip locked rows.

---

# 🚀 MOST IMPORTANT REAL-WORLD USE

```text
JOB QUEUES
```

---

# ⚡ Queue Worker Example

Worker 1:

```text
Gets jobs 1–10
```

Worker 2:

```text
Skips locked rows
Gets jobs 11–20
```

Parallel processing without conflicts.

---

# 🧠 WHERE CURRENT OF

Used with:

```text
FOR UPDATE cursors
```

---

# 📌 Example

```sql
UPDATE employees
SET salary = salary * 1.1
WHERE CURRENT OF c_emp;
```

Updates current cursor row directly.

---

# 🧠 ISOLATION LEVELS

---

# 📌 READ COMMITTED (DEFAULT)

Each query sees:

```text
latest committed data
```

---

# ⚡ Important

Two queries inside same transaction:

```text
may return different values
```

---

# 📌 SERIALIZABLE

Transaction sees:

```text
snapshot from transaction start
```

---

# ⚡ Benefit

Consistent reporting.

---

# 🚨 Risk

Error:

```text
ORA-08177
```

Serialization failure.

Usually solved by:

```text
retry transaction
```

---

# 🧠 READ CONSISTENCY

---

# 📌 READ COMMITTED

Consistency per:

```text
statement
```

---

# 📌 SERIALIZABLE

Consistency per:

```text
entire transaction
```

---

# 🚨 ORA-01555 — SNAPSHOT TOO OLD

---

# 📌 Cause

Long-running query needs UNDO data that got overwritten.

---

# ✅ Solutions

| Solution                | Meaning                        |
| ----------------------- | ------------------------------ |
| Increase UNDO retention | Keep older versions longer     |
| Shorter queries         | Reduce undo dependency         |
| Flashback query         | Read older consistent snapshot |

---

# 📌 Flashback Example

```sql
SELECT *
FROM employees
AS OF TIMESTAMP
(SYSTIMESTAMP - INTERVAL '1' HOUR);
```

---

# 🧠 BLOCKING SESSIONS

---

# 📌 What is Blocking?

One session waits because another holds required lock.

---

# ⚡ Blocking Chain

```text
Session A blocks B
Session B blocks C
```

---

# 📌 Root Blocker

Original blocking session.

---

# 🧠 FINDING BLOCKERS

---

# 📌 Important View

```sql
v$session
```

---

# 📌 Find Blocked Sessions

```sql
SELECT *
FROM v$session
WHERE blocking_session IS NOT NULL;
```

---

# 📌 Find Current Locks

```sql
SELECT *
FROM v$lock;
```

---

# 📌 Locked Objects

```sql
SELECT *
FROM v$locked_object;
```

---

# 🧠 RESOLVING BLOCKING

---

# ✅ Option 1

Wait for blocker.

---

# ✅ Option 2

Contact user.

---

# ✅ Option 3

Kill session.

```sql
ALTER SYSTEM KILL SESSION 'sid,serial#';
```

---

# 🚨 Immediate Kill

```sql
ALTER SYSTEM KILL SESSION 'sid,serial#' IMMEDIATE;
```

---

# 🧠 SKIP LOCKED REAL-WORLD ARCHITECTURE

---

# 📌 Used In

| System               | Purpose                 |
| -------------------- | ----------------------- |
| Job Queues           | Parallel workers        |
| Order Processing     | Distributed systems     |
| Inventory Allocation | Concurrent reservations |
| Background Workers   | Non-overlapping tasks   |

---

# 🧠 OPTIMISTIC LOCKING

---

# 📌 Idea

Do NOT lock immediately.

Instead:

* use version number
* update only if version unchanged

---

# 📌 Example

```sql
UPDATE products
SET quantity = quantity - 1,
    version = version + 1
WHERE version = old_version;
```

---

# ⚡ Benefit

Excellent for:

```text
high-concurrency systems
```

---

# 🧠 MONITORING & DIAGNOSTICS

---

# 📌 Important Views

| View            | Purpose        |
| --------------- | -------------- |
| v$lock          | Current locks  |
| v$session       | Session info   |
| v$transaction   | Transactions   |
| v$locked_object | Locked objects |

---

# 📌 Wait Events

```sql
SELECT *
FROM v$system_event
WHERE event LIKE '%lock%';
```

---

# 🧠 MOST IMPORTANT ERRORS

| Error     | Meaning               |
| --------- | --------------------- |
| ORA-00060 | Deadlock              |
| ORA-00054 | Resource busy         |
| ORA-30006 | WAIT timeout          |
| ORA-01555 | Snapshot too old      |
| ORA-08177 | Serialization failure |

---

# 🧠 BEST PRACTICES

| Practice                 | Why                   |
| ------------------------ | --------------------- |
| Lock in consistent order | Prevent deadlocks     |
| Keep transactions short  | Reduce blocking       |
| Commit frequently        | Release locks         |
| Use SKIP LOCKED          | Parallel queues       |
| Use NOWAIT/WAIT          | Avoid hanging forever |
| Monitor blocking         | Detect issues early   |

---

# 🚀 MOST IMPORTANT INTERVIEW TOPICS

| Topic              | Importance |
| ------------------ | ---------- |
| MVCC               | ⭐⭐⭐⭐⭐      |
| SELECT FOR UPDATE  | ⭐⭐⭐⭐⭐      |
| Deadlocks          | ⭐⭐⭐⭐⭐      |
| SKIP LOCKED        | ⭐⭐⭐⭐⭐      |
| Isolation Levels   | ⭐⭐⭐⭐⭐      |
| Blocking Sessions  | ⭐⭐⭐⭐⭐      |
| ORA-01555          | ⭐⭐⭐⭐⭐      |
| Optimistic Locking | ⭐⭐⭐⭐⭐      |

---

# 🏁 FINAL TAKEAWAYS

✅ Oracle uses MVCC for high concurrency
✅ Readers and writers generally do not block each other
✅ Row-level locking minimizes contention
✅ Deadlocks occur from circular waiting
✅ Consistent lock ordering prevents deadlocks
✅ SELECT FOR UPDATE enables pessimistic locking
✅ SKIP LOCKED is ideal for parallel workers and queues
✅ READ COMMITTED is Oracle’s default isolation level
✅ SERIALIZABLE provides transaction-level consistency
✅ Blocking sessions can be diagnosed using V$ views
✅ Optimistic locking improves scalability in high-concurrency systems


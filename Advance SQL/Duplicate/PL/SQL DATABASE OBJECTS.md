# Advanced SQL Concurrency & Locking Deep Dive Guide

> 🔥 A comprehensive, interview-ready, exam-focused deep dive into Database Concurrency, Transaction Isolation, Locking mechanisms, and tricky edge cases. Built for backend engineers and DBAs who want to truly understand how databases handle thousands of simultaneous users without corrupting data.

---

## 📌 Table of Contents

1. [🏗️ Concurrency Architecture (MVCC)](https://www.google.com/search?q=%23%EF%B8%8F-concurrency-architecture-mvcc)
2. [🔒 Locking Mechanisms (Implicit vs. Explicit)](https://www.google.com/search?q=%23-locking-mechanisms-implicit-vs-explicit)
3. [⏱️ Explicit Locking (FOR UPDATE)](https://www.google.com/search?q=%23%EF%B8%8F-explicit-locking-for-update)
4. [⏭️ Queue Processing (SKIP LOCKED)](https://www.google.com/search?q=%23%EF%B8%8F-queue-processing-skip-locked)
5. [💀 Deadlocks & Prevention](https://www.google.com/search?q=%23-deadlocks--prevention)
6. [🛡️ Transaction Isolation Levels](https://www.google.com/search?q=%23%EF%B8%8F-transaction-isolation-levels)
7. [🧪 Advanced Practice MCQs](https://www.google.com/search?q=%23-advanced-practice-mcqs)
8. [🔥 Interview Cheat Sheet](https://www.google.com/search?q=%23-interview-cheat-sheet)
9. [⚡ Performance Optimization Tips](https://www.google.com/search?q=%23-performance-optimization-tips)
10. [🚨 Common Pitfalls](https://www.google.com/search?q=%23-common-pitfalls)

---

## 🏗️ Concurrency Architecture (MVCC)

### Definition

**Multi-Version Concurrency Control (MVCC)** is the engine design used by Oracle, PostgreSQL, and others to provide high concurrency. The golden rule of MVCC is: **Readers do not block writers, and writers do not block readers.**

### Internal Working

```text
DATABASE SHARED MEMORY (SGA)
┌──────────────────────────────────────────────────────────────┐
│  DATA BLOCKS (Current Version)                               │
│  Row 100: [Alice, Salary: $55,000] ← UNCOMMITTED UPDATE      │
│                                      (Locked by Session 1)   │
├──────────────────────────────────────────────────────────────┤
│  UNDO SEGMENT (Previous Versions)                            │
│  Row 100: [Alice, Salary: $50,000] ← Snapshot preserved      │
└──────────────────────────────────────────────────────────────┘

SESSION 1 (Writer)                       SESSION 2 (Reader)
──────────────────                       ──────────────────
UPDATE emp SET salary = 55000            
WHERE id = 100;                          
[Acquires Row Lock]                      

                                         SELECT salary FROM emp
                                         WHERE id = 100;
                                         [Reads from UNDO Segment!]
                                         → Returns $50,000 instantly.
                                         (No waiting, no blocking)

```

### Key Components

| Component | Role |
| --- | --- |
| **Data Blocks** | Store the most current (even if uncommitted) version of the data. |
| **Undo Segments** | Store the *pre-modification* images of data. Used for read consistency and rollbacks. |
| **SCN (System Change Number)** | A logical internal timestamp. Queries only see data committed *before* their SCN. |

### ⚠️ Edge Cases

* **Snapshot Too Old (ORA-01555):** If Session 2 runs a massive 4-hour `SELECT` query, and Session 1 commits its update and does 50 more updates, the Undo segment might overwrite the old $50,000 value. Session 2 will crash because it can no longer reconstruct the past.
* **Writers BLOCK Writers:** MVCC only separates reading and writing. If Session 2 tries to `UPDATE` row 100 while Session 1 is still updating it, Session 2 *will* hang and wait.

### ❌ Common Mistakes

* Assuming a `SELECT` statement locks a row in Oracle/PostgreSQL. It does not.
* Building custom application-level lock tables instead of trusting the database's native MVCC engine.

### 🎯 Interview Traps

> **"Does a reader ever have to wait for a writer to finish?"**
> **NO.** Under MVCC, a standard `SELECT` never waits for an `UPDATE` or `DELETE`. It simply reconstructs the older version of the data from the Undo logs.

---

## 🔒 Locking Mechanisms (Implicit vs. Explicit)

### Definition

Locks are mechanisms used to protect data integrity during concurrent transactions.

* **Implicit Locks:** Automatically acquired by the database engine during DML operations (`INSERT`, `UPDATE`, `DELETE`).
* **Explicit Locks:** Manually requested by the developer (e.g., `SELECT ... FOR UPDATE`).

### Types of Locks

| Lock Level | Oracle Name | Description |
| --- | --- | --- |
| **Row-Level** | TX (Transaction Lock) | Protects individual rows. Highest concurrency. |
| **Table-Level** | TM (DML Enqueue) | Protects table structure. Prevents someone from running `DROP TABLE` or `ALTER TABLE` while rows are being updated. |

### ⚠️ Edge Cases

* **Bitmap Indexes:** Updating a single row in a table with a Bitmap index can lock massive chunks of the table (hundreds of rows), destroying concurrency. Never use Bitmap indexes in OLTP systems.
* **Unindexed Foreign Keys:** In older database versions, deleting a parent row without an index on the child table's foreign key could cause a full table lock on the child table.

### ❌ Common Mistakes

```sql
-- ❌ Doing heavy logic in the middle of an active transaction
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- ... Application goes to sleep or calls external slow API ...
-- The row remains locked, blocking all other users!
COMMIT;

```

---

## ⏱️ Explicit Locking (FOR UPDATE)

### Definition

Sometimes implicit locking isn't enough. If your application needs to read a value, perform complex calculations in code, and then update that value, you must lock the row *before* you read it to prevent race conditions (lost updates).

### Syntax Variations

```sql
-- 1. Standard FOR UPDATE (Waits indefinitely if locked)
SELECT balance FROM accounts WHERE id = 100 FOR UPDATE;

-- 2. NOWAIT (Fails instantly if locked - raises ORA-00054)
SELECT balance FROM accounts WHERE id = 100 FOR UPDATE NOWAIT;

-- 3. WAIT N (Waits N seconds, then fails if still locked)
SELECT balance FROM accounts WHERE id = 100 FOR UPDATE WAIT 5;

-- 4. Specific Columns (Locks only rows in the table that owns this column)
SELECT e.name, d.department_name 
FROM employees e JOIN departments d ON e.dept_id = d.id
FOR UPDATE OF e.salary; -- Only locks the employees table rows!

```

### ⚠️ Edge Cases

* Explicit locks remain active until the transaction ends via `COMMIT` or `ROLLBACK`. There is no `UNLOCK` command.
* If a client application crashes after issuing a `FOR UPDATE` but before committing, the lock becomes an "orphaned lock" until the database detects the dead connection and cleans it up.

### 🎯 Interview Traps

> **"What happens if two sessions run `SELECT ... FOR UPDATE` on the same row?"**
> The first session acquires the lock. The second session **hangs and waits** (unless `NOWAIT` or `WAIT` is specified). It does not error out immediately by default.

---

## ⏭️ Queue Processing (SKIP LOCKED)

### Definition

`SKIP LOCKED` allows a query to lock rows for processing while instantly bypassing any rows that are already locked by other sessions. It is the gold standard for building highly concurrent, lock-free **Message Queues** directly inside a relational database.

### Internal Working

```sql
-- Multiple background workers (consumers) run this exact same code continuously:
DECLARE
    CURSOR c_pending IS 
        SELECT job_id, job_data 
        FROM job_queue 
        WHERE status = 'PENDING' 
        ORDER BY priority 
        FOR UPDATE SKIP LOCKED; 
    v_job c_pending%ROWTYPE;
BEGIN
    OPEN c_pending;
    FETCH c_pending INTO v_job;
    
    IF c_pending%FOUND THEN
        -- Mark as processing so others don't pick it up even after we commit
        UPDATE job_queue SET status = 'PROCESSING' WHERE CURRENT OF c_pending;
        COMMIT; 
        
        -- ... Do actual heavy application work here ...
        
        -- Mark completed
        UPDATE job_queue SET status = 'COMPLETED' WHERE job_id = v_job.job_id;
        COMMIT;
    END IF;
    CLOSE c_pending;
END;

```

### Why it's brilliant:

* **Worker A** locks row 1.
* **Worker B** executes the query a millisecond later. Instead of waiting for row 1, it instantly skips it and locks row 2.
* **Worker C** skips rows 1 and 2, locking row 3.
* **Zero blocking, maximum throughput.**

### ❌ Common Mistakes

* Building database queues using standard `FOR UPDATE`. All workers will bottleneck trying to lock the exact same "top" row, reducing your parallel workers to a single-file line.

---

## 💀 Deadlocks & Prevention

### Definition

A **Deadlock** occurs when two or more transactions hold locks that the other transactions need, creating an infinite waiting loop.

### The Deadlock Scenario

```text
SESSION 1                          SESSION 2
─────────                          ─────────
UPDATE accounts                    
SET bal = bal - 100                
WHERE id = 1;                      
[Locks Account 1]                  
                                   UPDATE accounts
                                   SET bal = bal - 50
                                   WHERE id = 2;
                                   [Locks Account 2]

UPDATE accounts                    
SET bal = bal + 100                
WHERE id = 2;                      
[WAITS for Session 2]              
                                   UPDATE accounts
                                   SET bal = bal + 50
                                   WHERE id = 1;
                                   [WAITS for Session 1]
                                   
                     💥 DEADLOCK 💥

```

### Database Resolution

Databases automatically detect this circular dependency within seconds. The database will **kill one of the statements** (raising an exception like `ORA-00060: deadlock detected while waiting for resource`) and roll back that specific statement, freeing the lock so the other session can proceed.

### Prevention Strategy (Lock Ordering)

Always lock resources in a **consistent, hierarchical order**.

```sql
-- ✅ CORRECT WAY: Always lock the lower ID first, regardless of transfer direction.
IF p_from_account < p_to_account THEN
    v_first_acct := p_from_account;
    v_second_acct := p_to_account;
ELSE
    v_first_acct := p_to_account;
    v_second_acct := p_from_account;
END IF;

-- Now lock them in order. Deadlocks are mathematically impossible.
SELECT balance INTO v_dummy FROM accounts WHERE account_id = v_first_acct FOR UPDATE;
SELECT balance INTO v_dummy FROM accounts WHERE account_id = v_second_acct FOR UPDATE;

```

### 🎯 Interview Traps

> **"How do you fix a deadlock in production?"**
> You cannot "fix" a deadlock via database configuration. Deadlocks are **application logic bugs**. They must be fixed in the code by enforcing strict lock ordering or using `NOWAIT` retry logic.

---

## 🛡️ Transaction Isolation Levels

### Definition

Isolation levels determine how strictly a transaction is isolated from the modifications made by other concurrent transactions.

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Oracle Support? |
| --- | --- | --- | --- | --- |
| **Read Uncommitted** | Possible | Possible | Possible | ❌ No |
| **Read Committed** | Prevented | Possible | Possible | ✅ Yes (Default) |
| **Repeatable Read** | Prevented | Prevented | Possible | ❌ No |
| **Serializable** | Prevented | Prevented | Prevented | ✅ Yes |

### 1. READ COMMITTED (The Default)

A transaction only sees data that was committed *before the specific statement began*.

* If you run `SELECT SUM(amount)` at 10:00, it sees data as it was at 10:00.
* If you run the exact same `SELECT SUM(amount)` at 10:05 in the same transaction, and someone else committed a change at 10:02, you *will* see the new data. (This is a "Non-Repeatable Read").

### 2. SERIALIZABLE

A transaction sees data exactly as it was *at the moment the transaction began*.

* If you start a transaction at 10:00, every query you run for the next hour will see the database exactly as it was at 10:00, ignoring all subsequent commits by others.

```sql
-- Setting isolation level
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

```

### ⚠️ Edge Cases

* **Serialization Failure (ORA-08177):** If Session 1 is in `SERIALIZABLE` mode and tries to update a row that was changed by Session 2 *after* Session 1 started, the database throws an error. Your application *must* catch this exception and retry the transaction from scratch.

### ❌ Common Mistakes

* Using `SERIALIZABLE` to prevent all concurrency issues without writing the application logic to catch and retry ORA-08177 errors.

---

## 🧪 Advanced Practice MCQs

**Q1.** You execute an `UPDATE` on row 5, but do not commit. Another session executes a `SELECT` on row 5. What happens?

* A) The `SELECT` waits for the `UPDATE` to commit.
* B) The `SELECT` reads the newly updated, uncommitted value (Dirty Read).
* C) The `SELECT` reads the original value from the Undo Segment. ✅
* D) The database throws a lock error.

*Explanation:* Under MVCC, readers don't wait for writers. The database reconstructs the old version for the reader.

---

**Q2.** Which explicit locking clause allows multiple background workers to pull tasks from a table without blocking each other?

* A) `FOR UPDATE NOWAIT`
* B) `FOR UPDATE WAIT 10`
* C) `FOR UPDATE SKIP LOCKED` ✅
* D) `LOCK TABLE IN EXCLUSIVE MODE`

*Explanation:* `SKIP LOCKED` skips over rows already locked by other workers, allowing true parallel processing of queues.

---

**Q3.** A deadlock is detected by the database. What action does the database take?

* A) Kills the database connection for both sessions.
* B) Rolls back the statement of one session and returns an error to it. ✅
* C) Rolls back the entire transaction of both sessions.
* D) Pauses both sessions until a DBA intervenes.

*Explanation:* The database sacrifices one session (usually the one that initiated the deadlock loop) by rolling back its current statement and returning an ORA-00060 error, allowing the other to proceed.

---

**Q4.** In Oracle's default isolation level (Read Committed), which anomaly is possible?

* A) Dirty Read
* B) Non-Repeatable Read ✅
* C) Lost Update
* D) None of the above

*Explanation:* In Read Committed, if you read a row twice in the same transaction, and another session commits an update in between, your second read will see the new data (Non-Repeatable Read).

---

## 🔥 Interview Cheat Sheet

### The 10 Most Important Facts

```text
1. MVCC Rule: Readers don't block writers; writers don't block readers.
2. Writers DO block writers.
3. Undo Segments provide read consistency (snapshots of old data).
4. Implicit locks (TX) are acquired automatically during DML.
5. SELECT FOR UPDATE acquires a TX lock manually before updating.
6. NOWAIT prevents your session from hanging indefinitely on a locked row.
7. SKIP LOCKED is the only correct way to build a DB-based message queue.
8. Deadlocks are application bugs (circular locking logic), not database flaws.
9. Prevent deadlocks by always locking tables/rows in a consistent sorting order.
10. SERIALIZABLE isolation guarantees consistency but requires app-level retry logic.

```

---

## ⚡ Performance Optimization Tips

### 1. Keep Transactions Short

The longer a transaction stays open, the longer locks are held, and the larger the Undo segments grow. Commit as soon as logical business units of work are complete.

### 2. Lock Late, Commit Early

Do not run `SELECT FOR UPDATE`, then make a slow API call to Stripe/AWS, and *then* update the database. Make the API call first, then acquire the lock, update, and commit instantly.

### 3. Avoid Table Locks at All Costs

Never use `LOCK TABLE table_name IN EXCLUSIVE MODE`. Rely on row-level (`FOR UPDATE`) locking to maximize concurrent throughput.

---

## 🚨 Common Pitfalls

### 1. The Lost Update Problem (Without Locks)

```sql
-- ❌ Session 1 and Session 2 both read balance (e.g., 100)
SELECT balance FROM accounts WHERE id = 1; 

-- Session 1 adds 50 (writes 150)
UPDATE accounts SET balance = 150 WHERE id = 1;

-- Session 2 subtracts 20 (writes 80). 
-- Session 1's deposit is completely overwritten and lost!
UPDATE accounts SET balance = 80 WHERE id = 1; 

```

**Fix:** Use `SELECT FOR UPDATE` on the read, or use relative updates (`UPDATE accounts SET balance = balance + 50`).

### 2. The Mutating Table Trigger Trap

```sql
-- ❌ Trying to SELECT from the same table you are currently UPDATING via a Row-Level trigger.
-- Oracle throws ORA-04091 because the table is inconsistent mid-transaction.
CREATE TRIGGER trg_salary BEFORE UPDATE ON emp FOR EACH ROW
BEGIN
    SELECT AVG(salary) INTO v_avg FROM emp; -- CRASH!
END;

```

**Fix:** Use Compound Triggers to defer the `SELECT` until the `AFTER STATEMENT` timing point.

### 3. Ignoring ORA-00054 (Resource Busy)

```sql
-- ❌ Using NOWAIT but not catching the exception
SELECT * FROM table FOR UPDATE NOWAIT;
-- If locked, application crashes with unhandled exception.

```

**Fix:** Always wrap `NOWAIT` in an exception block and build exponential backoff/retry logic, or return a friendly "Resource is busy, please try again" message to the user.
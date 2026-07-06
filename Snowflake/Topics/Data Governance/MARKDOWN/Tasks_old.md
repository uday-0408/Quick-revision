# 📘 Snowflake Tasks — Part 1: Introduction & Compute Models

Before we jump in — quick connection: You already know **Streams** track changes (INSERT/UPDATE/DELETE) in a table. A **Task** is the thing that actually *acts* on that change. Streams = "what changed", Tasks = "what to do about it". Together they build automated pipelines (this is why Snowflake documents them as "Streams and Tasks").

Let's go section by section.

---

## 1️⃣ What is a Task? (Concept)

**Simple English:** A Task is just a scheduled or event-triggered job in Snowflake that runs a SQL statement or a stored procedure (Python/Java/Scala/JS/Snowflake Scripting). Think of it like a **cron job**, but living inside Snowflake, with Snowflake managing compute for you (optionally).

**Why this feature exists:**
- Companies need automation: "every hour, move data from staging to final tables" or "whenever new rows arrive, transform them."
- Without Tasks, you'd need an external scheduler (Airflow, cron server) just to run SQL — extra infrastructure, extra cost, extra failure points.
- Tasks let the *whole* pipeline (ingest → transform → load) live inside Snowflake.

**Real-world example:** An e-commerce company has an `ORDERS_RAW` table getting new rows every minute. A Task runs every 5 minutes to clean and insert them into `ORDERS_CLEAN`.

⭐ **Exam importance: ⭐⭐⭐⭐⭐** (Tasks + Streams is one of the most tested areas in SnowPro Core)

---

## 2️⃣ Task Creation Workflow (the 6-step lifecycle)

This is important — SnowPro loves asking "what state is a task in after X action?"

1. **Create a task admin role** (best practice, not mandatory)
2. **CREATE TASK** — define compute, schedule/trigger, failure handling, session params
3. **EXECUTE TASK** — manually test it once
4. **ALTER TASK … RESUME** — this is the step that actually "activates" the task
5. **Monitor costs**
6. **ALTER TASK** — modify later as needed

### 🚨 Biggest Gotcha #1 (exam favorite):
> **A task is ALWAYS created in a SUSPENDED state.** It will NOT run automatically just because you used `CREATE TASK`. You must explicitly run `ALTER TASK <name> RESUME;`

Students often assume "I created it with a schedule, so it should run" — **wrong**. This is a very common trick question.

---

## 3️⃣ Two Compute Models for Tasks

This is a **major conceptual pillar**. Every task needs compute to run its SQL. You choose one of two models:

| | **Serverless Tasks** | **User-Managed Tasks** |
|---|---|---|
| Who manages compute | Snowflake (auto-predicts size) | You (attach a virtual warehouse) |
| How you create | `CREATE TASK` **without** `WAREHOUSE` param | `CREATE TASK` **with** `WAREHOUSE = 'wh_name'` |
| Billing | Per actual compute-second used (serverless compute-hours) | Warehouse credits, 60-second minimum per resume |
| Max size | Capped at **XXLARGE** equivalent | No cap — you can use any size, even multi-cluster |
| Required privilege | `EXECUTE MANAGED TASK` (account level) | Just `USAGE` on the warehouse |
| Best for | Under-utilized / bursty / unpredictable-but-small workloads | Fully utilized shared warehouses, heavy workloads, or workloads needing >XXLARGE |

### 🧠 Internal working (Serverless):
```
Task Scheduled to Run
        │
        ▼
Snowflake checks recent run history of THIS task
        │
        ▼
Predicts required compute size (between MIN and MAX statement size)
        │
        ▼
Auto-provisions serverless compute
        │
        ▼
Runs task → measures actual performance
        │
        ▼
Adjusts prediction for NEXT run
```

This is key: **serverless sizing is dynamic and self-learning** — it looks at *your task's own history*, not a generic model.

### 🚨 Gotcha #2:
> Serverless task compute is capped at **XXLARGE**. If your workload genuinely needs bigger than XXLARGE (e.g., 4XL, 5XL, 6XL warehouse), you **must** use a user-managed task with an explicitly sized warehouse. This is a classic "which one would you choose" scenario question.

### 🚨 Gotcha #3 — SERVERLESS_TASK_MIN/MAX_STATEMENT_SIZE:
- `SERVERLESS_TASK_MIN_STATEMENT_SIZE` → default **XSMALL**
- `SERVERLESS_TASK_MAX_STATEMENT_SIZE` → default **XXLARGE**
- These are just *guardrails*. Setting MIN too high wastes money on light workloads; setting MAX too low can force timeouts on heavy days.

### 🚨 Gotcha #4 — Target Completion Interval:
- Lets you say "I want this daily task to finish by 2 AM even though it starts at midnight."
- **Important nuance:** If the task is *already* at max warehouse size (its ceiling) and still running long, Snowflake **ignores** the target completion interval — it will NOT exceed the max size you configured, and will simply keep running (until timeout via `USER_TASK_TIMEOUT_MS`).
- This is a subtle exam trap: people assume "target completion interval" is a hard guarantee — it's **not**, it's a best-effort scaling target, bounded by your max size setting.

---

## 4️⃣ Common Misconceptions (Section 1)

❌ *"CREATE TASK with a schedule means it starts running immediately."*
✅ Reality: Task is suspended by default. Needs `ALTER TASK … RESUME`.

❌ *"Serverless tasks can scale infinitely to hit my target time."*
✅ Reality: Capped at XXLARGE (or whatever MAX_STATEMENT_SIZE you set, whichever is lower).

❌ *"User-managed tasks are always cheaper."*
✅ Reality: User-managed warehouses bill with a 60-second minimum per resume — for tiny/infrequent tasks, serverless can actually be cheaper since billing is closer to actual usage.

---

## 5️⃣ Quick Revision Notes (Part 1)

- Task = automated SQL/stored-proc runner (schedule OR event-based).
- New tasks = **always suspended** at creation.
- Two compute models: **Serverless** (no WAREHOUSE param, needs `EXECUTE MANAGED TASK` privilege) vs **User-Managed** (WAREHOUSE param, needs warehouse USAGE).
- Serverless max size = **XXLARGE**.
- `SERVERLESS_TASK_MIN/MAX_STATEMENT_SIZE` control cost guardrails.
- `TARGET_COMPLETION_INTERVAL` = best-effort scaling goal, not a hard promise; ignored once at max size.
- Choose serverless → light/unpredictable/bursty loads. Choose user-managed → heavy, predictable, shared, or >XXLARGE workloads.

---

## 📝 Mini Mock Test (Part 1)

**1.** After running `CREATE TASK my_task SCHEDULE='5 MINUTE' AS SELECT 1;`, what is the task's state?
A) RUNNING B) SUSPENDED ✅ C) STARTED D) PENDING
*Why: All new tasks start suspended, regardless of schedule.*

**2.** Which privilege is REQUIRED specifically for serverless tasks (in addition to normal task privileges)?
A) USAGE on warehouse B) EXECUTE MANAGED TASK ✅ C) OWNERSHIP D) CREATE WAREHOUSE
*Why: Serverless has no warehouse, so warehouse USAGE isn't needed — instead the account-level EXECUTE MANAGED TASK privilege is required.*

**3.** What is the maximum compute size available for a serverless task?
A) LARGE B) XXLARGE ✅ C) 4XLARGE D) Unlimited

**4 (Scenario):** Your serverless task's TARGET_COMPLETION_INTERVAL is 30 minutes, and MAX_STATEMENT_SIZE is LARGE. The task is already running at LARGE and taking 45 minutes. What happens?
A) Snowflake scales beyond LARGE to meet target B) Task is killed instantly C) Snowflake ignores the target interval since it's at max size, and the task keeps running until timeout/completion ✅ D) Task auto-converts to user-managed

**5 (T/F):** A user-managed task can use a multi-cluster warehouse with no compute size cap.
✅ True

---

Should I continue to **Part 2: Schedules & Triggers (CRON, stream-triggered tasks) + Failure Handling (SUSPEND_TASK_AFTER_NUM_FAILURES, auto-retry)**? Or do you want to attempt the mock test answers first before I move ahead?
# Snowflake TASKS — Complete SnowPro Core Notes
*(Covers: tasks-intro, tasks-triggered, tasks-graphs, tasks-monitor, tasks-errors, tasks-errors-integrate, tasks-success-integrate, tasks-events, ui-snowsight-tasks, tasks-python-jvm, tasks-ts)*

---

## 0. The Big Picture — Where Tasks Fit

Think of Snowflake automation like this:

```
STREAM  →  "Something changed in my data" (the detector)
TASK    →  "Do something about it" (the doer)
```

A **Stream** just watches a table/view for changes (insert/update/delete). It never *does* anything by itself.
A **Task** is the thing that actually *runs code* — SQL, a stored procedure, Python, Java, Scala, or Snowflake Scripting — either on a schedule or when triggered.

Put them together and you get an automated pipeline: "whenever new rows land in this stream, run this task to process them."

**Exam importance: ⭐⭐⭐⭐⭐** — Tasks are one of the most tested SnowPro Core topics, especially compute models, scheduling, task graphs, and security.

---

## 1. What Is a Task? (Introduction)

### Concept
A task is a scheduled or event-driven container for running one SQL statement, or a call to a stored procedure. It's Snowflake's built-in "cron job + orchestrator" — you don't need an external scheduler like Airflow just to run a query every hour.

### Why this exists
Before tasks, if you wanted a query to run every night, you needed an external tool (cron on a server, Airflow, dbt Cloud, etc.) that connects to Snowflake and fires the SQL. Tasks move that scheduling logic *inside* Snowflake, so there's one less moving part, one less external system to secure and maintain.

### The Task Creation Workflow (memorize this order — it's the "how" of tasks)
1. Create a **task admin role**.
2. `CREATE TASK` — define compute, schedule/trigger, failure handling, session params.
3. Test it manually with `EXECUTE TASK`.
4. `ALTER TASK … RESUME` — let it run continuously.
5. Monitor cost.
6. Refine using `ALTER TASK`.

### 🚨 Critical Gotcha #1 — Tasks are born SUSPENDED
**Every task starts in the SUSPENDED state when created.** It will NOT run — not on schedule, not on trigger — until you explicitly run:
```sql
ALTER TASK my_task RESUME;
```
This is probably the single most common "why didn't my task run" exam scenario and real-world mistake. If someone says "I created a task with a schedule but it never fires," the answer is almost always: *they forgot to resume it.*

---

## 2. Compute Models — Serverless vs User-Managed

This is a **heavily tested comparison**.

| | **Serverless Tasks** | **User-Managed Tasks** |
|---|---|---|
| Who manages compute | Snowflake auto-predicts & assigns | You assign a virtual warehouse |
| SQL syntax | No `WAREHOUSE` parameter | Must include `WAREHOUSE = 'wh_name'` |
| Max size | Equivalent to **XXLARGE** | Any size you choose |
| Billing | Per actual compute-resource usage (compute-hours) | Per warehouse-second, 60-second minimum per resume |
| Required privilege | `EXECUTE MANAGED TASK` (account-level) | Just `USAGE` on the warehouse |
| Best for | Few/short/unpredictable-but-stable tasks, underutilized warehouses | Fully-utilized warehouses, many concurrent tasks, predictable heavy loads |
| Resource tuning | `SERVERLESS_TASK_MIN_STATEMENT_SIZE` / `_MAX_STATEMENT_SIZE` (default XSMALL–XXLARGE) | You size the warehouse manually |

### Key mechanism: how serverless sizing actually works
Snowflake looks at the **most recent runs of the same task** and dynamically predicts how much compute it needs. After each run, it reviews performance and adjusts sizing for *future* runs (within your min/max bounds). So a serverless task literally "learns" over time.

### Target Completion Interval
- Lets you tell Snowflake "try to finish by X," and it scales compute to hit that target.
- **Required** if you want a *serverless triggered task* (see Section 4).
- 🚨 **Gotcha:** If the task is already at max warehouse size and still running long, the target is simply *ignored* — it won't force a completion, it just does its best.

### 🎯 Exam Trap
Q: "You want a serverless task to always complete within 15 minutes no matter what." 
❌ Wrong answer: "Set `TARGET_COMPLETION_INTERVAL` and it's guaranteed."
✅ Correct: Snowflake will *try* to scale up to hit the target, but if it's already maxed at XXLARGE, the target is silently ignored — no guarantee.

### Max compute size
The absolute ceiling for **serverless** is XXLARGE. If a workload genuinely needs more (e.g., a 4XL warehouse), you **must** switch to a user-managed task — this is a classic distractor pairing on the exam.

---

## 3. Scheduling Tasks

### Two scheduling styles
```sql
-- Fixed interval
CREATE TASK t1 SCHEDULE = '60 MINUTES' AS SELECT 1;

-- CRON (specific day/time, with time zone)
CREATE TASK t2 SCHEDULE = 'USING CRON 0 3 * * SUN America/Los_Angeles' AS SELECT 1;
```

### 🚨 Gotcha #2 — Only ONE instance of a scheduled task runs at a time
If a task is still running when its next scheduled time arrives, **that scheduled run is skipped entirely** — it does NOT queue up and run twice back-to-back. This matters a lot for tasks with short intervals (e.g., every 10 seconds) doing longer-than-10-second work.

### Real-world example
A task scheduled every 5 minutes that sometimes takes 7 minutes to run will occasionally "miss" a cycle — this is expected, documented behavior, not a bug.

---

## 4. Triggered Tasks (Event-Driven, via Streams)

### Concept
Instead of polling on a timer, a triggered task fires **only when a stream has new data.**

```sql
CREATE TASK my_triggered_task
  TARGET_COMPLETION_INTERVAL='15 MINUTES'
  WHEN SYSTEM$STREAM_HAS_DATA('my_order_stream')
  AS
    INSERT INTO customer_activity
    SELECT customer_id, order_total, order_date, 'order'
    FROM my_order_stream;
```

### Why this exists
Polling wastes compute — checking a stream every minute even when nothing changed still costs something with a scheduled task. Triggered tasks **don't consume compute until the event actually fires.** This is huge for unpredictable, bursty ELT workloads.

### Rules for creating a triggered task
- Use `WHEN`, **not** `SCHEDULE`.
- User-managed → include `WAREHOUSE`.
- Serverless → include `TARGET_COMPLETION_INTERVAL`, do **not** include `WAREHOUSE`.

### What's supported vs not (memorize — classic MCQ list)
✅ Supported: Tables, Views, Dynamic tables, Apache Iceberg tables (managed & unmanaged), Data shares, Directory tables (must be refreshed first).
❌ NOT supported: **Hybrid tables**, **Streams on external tables**.

### 🚨 Gotcha #3 — Directory tables need a manual/auto refresh
A stream on a directory table won't "see" new files automatically — the directory table has to be refreshed first, either via auto-refresh or `ALTER STAGE … REFRESH`. Then the triggered task fires.

### 🚨 Gotcha #4 — The 30-second (and 12-hour) rules — VERY exam-relevant
- Triggered tasks check the stream **at most every 30 seconds by default.** You can reduce this down to **10 seconds** using `USER_TASK_MINIMUM_TRIGGER_INTERVAL_IN_SECONDS`.
- If a task is triggered again while it's still running, the next check starts **30 seconds after the previous scheduled check** — not immediately.
- If a triggered task **hasn't run in 12 hours**, Snowflake automatically runs a "health check" to make sure the stream doesn't go stale. Timing of this check is **not guaranteed**. If no changes exist, it's skipped with zero compute cost.
- **Stream staleness is still your responsibility:** if your task doesn't consume the stream data before the retention period expires, the stream goes stale (loses ability to return correct changes) regardless of triggers.

### 🚨 Gotcha #5 — Streams on Views trigger on ANY underlying table change
If a triggered task watches a Stream on a View, **any change to any table referenced by that view's query triggers the task** — even if a JOIN, filter, or aggregation in the view means that specific change wouldn't show up in the stream's output. This is a subtle, frequently-missed point.

### Monitoring triggered tasks
- `SHOW TASKS` / `DESC TASK` → `SCHEDULE` column shows **NULL** for triggered tasks.
- `TASK_HISTORY` → `SCHEDULED_FROM` column shows **`TRIGGER`**.

### Migrating scheduled → triggered (and warehouse → serverless triggered)
Always the same 3-step dance:
1. **Suspend**
2. Modify with `ALTER TASK` (unset SCHEDULE/WAREHOUSE, add WHEN/TARGET_COMPLETION_INTERVAL)
3. **Resume**

### Combine both: scheduled AND conditional
```sql
CREATE TASK t
  SCHEDULE = '1 HOUR'
  WHEN SYSTEM$STREAM_HAS_DATA('orders_stream')
  AS SELECT 1;
```
This checks the stream every hour, but only actually runs the SQL if there's new data — best of both worlds for cost control.

### 🎯 Exam Trap
Two-stream `AND` vs `OR` in the `WHEN` clause:
- `WHEN … OR …` → runs if **either** stream has data.
- `WHEN … AND …` → runs **only if both** streams have data; if just one has changes, the task is **skipped entirely** (not run partially).

---

## 5. Failure Handling

### Auto-suspend after failures
```sql
SUSPEND_TASK_AFTER_NUM_FAILURES = 3
```
Suspends the task after N **consecutive** failures/timeouts. Set at task, schema, database, or account level — **lower-level settings override higher-level ones.**

### Auto-retry
```sql
TASK_AUTO_RETRY_ATTEMPTS = 2
```
- **Disabled by default** (must explicitly enable, value > 0).
- Each retry attempt that still fails sends its own error notification (if configured).
- For **task graphs**: if a child task fails, the retry re-runs the **whole graph**, not just the failed task — up to the configured number of attempts.

### 🚨 Gotcha #6 — Default failure behavior differs: standalone task vs task graph
| | Standalone task | Task graph (root task) |
|---|---|---|
| Default retry | None (disabled) | None (disabled) |
| Default auto-suspend threshold | Must set manually, no fixed default noted | **10 consecutive failures** (documented default!) |

The "task graph suspends after 10 failures by default" is a specific, quotable exam fact.

---

## 6. Task Security & Privileges

This is dense but **very testable** — SnowPro loves privilege tables.

### Creating a task
| Object | Privilege | Notes |
|---|---|---|
| Account | `EXECUTE MANAGED TASK` | Only for serverless |
| Database | `USAGE` | |
| Schema | `USAGE`, `CREATE TASK` | |
| Warehouse | `USAGE` | Only for user-managed |

### Running a task (task owner needs)
| Object | Privilege | Notes |
|---|---|---|
| Account | `EXECUTE TASK` | Revoking this on a role stops ALL task runs owned by that role |
| Account | `EXECUTE MANAGED TASK` | Only serverless |
| Database/Schema/Task | `USAGE` | |
| Warehouse | `USAGE` | Only user-managed |

### 🚨 Gotcha #7 — Suspend/Resume needs LESS than you'd think
A role only needs the **`OPERATE`** privilege on the task (plus `USAGE` on db/schema) to suspend or resume it — it does **not** need ownership. This trips people up: OPERATE ≠ OWNERSHIP, but OPERATE is enough for start/stop control.

### Viewing task history — needs ONE of:
- ACCOUNTADMIN role
- OWNERSHIP on the task
- Global `MONITOR EXECUTION` privilege
(Snowsight also accepts `MONITOR` or `OPERATE` privilege per the UI page — slightly broader there.)

### Who actually "runs" the task? — The System Service
**By default, tasks run as a system service**, using the task owner role's privileges — **not** as any real user. Why does this matter?
- If the user who created the task gets locked out, has roles revoked, or is deleted, **the task keeps running fine** — because it was never tied to a person.
- Query history shows the task run as `SYSTEM` — no individual can "be" this identity.

### `EXECUTE AS USER` — running tasks with a real user's privileges
Use this when you need:
1. **Multi-role privilege combination** — a real user's primary + secondary roles combined (system service only uses the single owner role).
2. **User-based masking/row-access policies** that depend on *which user* is querying.
3. **Accountability** — logs attribute the run to a specific named user instead of `SYSTEM`.

Access control requirement: task owner role must have `IMPERSONATE` on the target user, AND that user must be granted the task's owner role.

### 🚨 Gotcha #8 — Best practice is a dedicated SERVICE USER, not a real person
Documentation explicitly warns against impersonating an actual employee's account for production tasks:
- If you impersonate a real person, the task inherits **all** of that person's privileges — including ones granted *after* the task was set up (privilege creep, security risk).
- If that person leaves the company, the task can suddenly break.
- **Correct pattern:** create a dedicated service user (e.g., `task_user`), a dedicated role (e.g., `task_role`), grant IMPERSONATE on the service user to that role, and run tasks as that service user. This is a realistic scenario-based exam question.

### Dropping a task owner role
If you `DROP ROLE` on a task's owner, ownership **transfers to the role that ran the DROP command**, and the task is **automatically paused** — it must be manually resumed by the new owner. If the task was mid-run when the role was dropped, that run finishes under the old (now-dropped) role.

---

## 7. Task Versioning

### Concept
When you first resume/manually run a **standalone task**, Snowflake locks in an "initial version" — essentially a snapshot of its SQL/config. If you suspend + modify + resume, a **new version** is set at resume time.

### 🚨 Gotcha #9 — In-flight runs use the OLD version, even after you modify
If a task is currently running when you suspend and edit it, the **currently running instance keeps using the version that was active when it started** — your edits don't retroactively change a run in progress. Only the *next* resume/execute picks up the new version.

Same logic applies to **task graphs**: the whole graph gets versioned together when the root task resumes/executes. Also — editing a stored procedure's *body* while a task graph is running can cause the NEW procedure code to run mid-graph (since the proc itself isn't versioned the same way the task definition is) — a subtle inconsistency worth knowing.

Query `TASK_VERSIONS` (Account Usage view) to see version history.

---

## 8. Task Duration, Timeouts, and Costs

### Task Duration = Queuing Time + Execution Time
- **Queuing time:** waiting for compute to free up (compare `SCHEDULED_TIME` vs `QUERY_START_TIME` in TASK_HISTORY).
- **Execution time:** actual run (compare `QUERY_START_TIME` vs `COMPLETED_TIME`).

### Timeout precedence rules (frequently tested!)
- If both `STATEMENT_TIMEOUT_IN_SECONDS` and `USER_TASK_TIMEOUT_MS` are set → the **lower non-zero value wins**.
- If both `STATEMENT_QUEUED_TIMEOUT_IN_SECONDS` and `USER_TASK_TIMEOUT_MS` are set → **`USER_TASK_TIMEOUT_MS` takes precedence** (not "lowest wins" here — different rule from the pair above!).
- **Default timeout for a task with no `USER_TASK_TIMEOUT_MS` set = 60 minutes (3,600,000 ms)** — this exact number shows up in troubleshooting scenarios.

### 🎯 Exam Trap
Two different timeout-combination rules exist depending on *which pair* of parameters is set. Don't assume "lowest always wins" — memorize both rules separately.

### Costs
| Compute model | Billed on |
|---|---|
| User-managed warehouse | Standard warehouse credit usage; 60-second minimum per resume |
| Serverless | Actual compute-hours used, scales with size + runtime |

Cost best practices (an easy "select all that apply" style question):
- Schedule less frequently where possible.
- Use auto-suspend/auto-retry to avoid paying for endlessly-failing tasks.
- Use **triggered tasks** instead of frequent polling schedules.
- Set up **budgets/alerts** on serverless spend.

Query `SERVERLESS_TASK_HISTORY` (function or Account Usage view) for credit consumption; filter `METERING_HISTORY` / `METERING_DAILY_HISTORY` on `service_type = SERVERLESS_TASK` or `SERVERLESS_TASK_FLEX`.

---

## 9. Task Graphs (DAGs)

### Concept
A **task graph** = a root task + dependent child tasks, forming a Directed Acyclic Graph (DAG) — no loops allowed, always flows start-to-finish.

```sql
CREATE TASK task_root SCHEDULE = '1 MINUTE' AS SELECT 1;
CREATE TASK task_a AFTER task_root AS SELECT 1;
CREATE TASK task_b AFTER task_root AS SELECT 1;   -- runs in parallel with task_a
CREATE TASK task_c AFTER task_a, task_b AS SELECT 1;  -- waits for BOTH
```

### Hard limits (memorize exact numbers — classic MCQ)
- Max **1000 tasks** per task graph.
- Max **100 parent tasks** and **100 child tasks** per single task.

### Parallel vs sequential logic
- Same parent, multiple children → they run **in parallel**.
- Multiple parents, one child → child waits until **all** parents succeed (or are skipped — see below) before starting.

### 🚨 Gotcha #10 — A child with a skipped/suspended parent can still run
A child task with multiple predecessors will run as long as **at least one predecessor is resumed and all resumed predecessors succeed** — even if some parents were suspended (and therefore "skipped"), the child doesn't automatically fail. This is non-intuitive and worth a slow re-read.

### Finalizer Task
An optional task that runs **after everything else completes or fails** — for cleanup, notifications, or error correction.

```sql
CREATE TASK task_finalizer FINALIZE = task_root AS SELECT 1;
```

Rules:
- Exactly **one finalizer per root task**, and a finalizer can belong to only **one** root task.
- If the root task itself is **skipped** (e.g. due to overlapping runs), the finalizer **won't run at all**.
- A finalizer **cannot have child tasks** — it's always the true end of the line.
- It's scheduled only once nothing else in that graph run is still active or queued.

### 🎯 Exam Trap
"Can a finalizer task have children?" → **No.** This is a common trick option.

### Task Graph Ownership
- **All tasks in one graph must share the same owner role** and live in the same database/schema.
- Transfer ownership via `DROP ROLE` (ownership passes to the dropping role) or `GRANT OWNERSHIP` on all tasks.
- 🚨 Transferring ownership of a **single task** (not the whole graph) **severs its dependency links** — it becomes standalone/root. This is a deliberate design choice to prevent partial-graph ownership confusion.
- Database **replication doesn't work for task graphs** if the graph's owner role differs from the replication-performing role.

### Running/scheduling a graph
- Only the **root task** defines the schedule or trigger — children just say `AFTER <parent>`.
- To manually run once: `RESUME` all desired child tasks first, then `EXECUTE TASK` on the **root**.
- To fully enable a graph at once: `SYSTEM$TASK_DEPENDENTS_ENABLE(<root_task_name>)` — resumes the whole tree in one call (instead of resuming each task one by one).

### Modify/Suspend/Retry
- To modify **any** task in a graph → suspend the **root task** first (not each child). Children retain their individual state.
- **Skip a child**: suspend just that child — the graph proceeds as if it succeeded.
- **Retry the latest failed run**: `EXECUTE TASK … RETRY LAST`.
- **Retry an older run**: `EXECUTE TASK … RETRY GRAPH RUN GROUP` with a `GRAPH_RUN_GROUP_ID`.
- Manual retries in Snowsight only work within **14 days** of the run, and only if the graph hasn't been altered since.

### Unlinking parent/child relationships
- `ALTER TASK … REMOVE AFTER` / `UNSET FINALIZE` → removes specific links.
- `DROP TASK` or `GRANT OWNERSHIP` (single task) → severs **all** links for that task (both to its parent and its children).
- 🚨 If you `GRANT OWNERSHIP` of a task to its **current** owner (i.e., no actual change), links might **not** be severed — a documented edge case.

### Overlap Policy — VERY exam-relevant
| Setting | Behavior |
|---|---|
| `NO_OVERLAP` (default) | Fully serial. Next root run only scheduled after ALL tasks finish. If total runtime > schedule interval, a run gets **skipped**. |
| `ALLOW_CHILD_OVERLAP` | Children can overlap across runs, but the **root task itself never overlaps**. |
| `ALLOW_ALL_OVERLAP` | Full parallelism — even multiple root task instances can run concurrently. |

### 🎯 Exam Trap
"If a task graph consistently takes longer than its schedule interval, what happens by default?" → With `NO_OVERLAP` (the default), **at least one scheduled run gets skipped entirely** — it doesn't queue up.

### Versioning + Timeouts for graphs
- Whole-graph version set at root resume/execute — same in-flight-run caveat as standalone tasks.
- `USER_TASK_TIMEOUT_MS` on the **root** → applies to the **entire graph run**.
- `USER_TASK_TIMEOUT_MS` on a **child** → applies to just that task, and **overrides** the root's timeout for that specific child.

### Task graph logic (dynamic behavior)
- `SYSTEM$SET_RETURN_VALUE()` / `SYSTEM$GET_PREDECESSOR_RETURN_VALUE()` — pass data between tasks.
- `CONFIG = '{"key":"value"}'` on the root + `SYSTEM$GET_TASK_GRAPH_CONFIG()` — pass static config to all tasks.
- `EXECUTE TASK … USING CONFIG` — override config for a single ad-hoc run without changing the task definition.
- `SYSTEM$TASK_RUNTIME_INFO()` — introspect the current run (e.g., `CURRENT_ROOT_TASK_NAME`, `CURRENT_TASK_GRAPH_ORIGINAL_SCHEDULED_TIMESTAMP`).

### 🚨 Gotcha #11 — Case sensitivity trap with `SYSTEM$GET_PREDECESSOR_RETURN_VALUE`
Some of these system functions are **case-sensitive**, but unquoted `CREATE TASK task_c` is stored in **UPPERCASE** internally. So `SYSTEM$GET_PREDECESSOR_RETURN_VALUE('task_c')` (lowercase) can silently fail to match — you must call it as `'TASK_C'`, or always use quoted identifiers consistently. This is a sneaky, very SnowPro-style gotcha about identifier case rules colliding with function behavior.

---

## 10. Monitoring Task Runs

### Task owners & who ran what
- `SHOW TASKS` / `DESC TASK` → `OWNER` column (role), `EXECUTE_AS_USER` column (NULL by default; shows a user name if impersonation is configured).
- `QUERY_HISTORY` → `QUERY EXECUTED BY TASK` column shows `"SYSTEM"` for the default system-service runs, or the impersonated user's name otherwise.

### Views/functions for history
| Function/View | Purpose |
|---|---|
| `TASK_HISTORY` (function, Info Schema) | Run history for **one task** |
| `TASK_HISTORY` (Account Usage view) | Account-wide task history |
| `CURRENT_TASK_GRAPHS` | Currently scheduled/running graph runs |
| `COMPLETE_TASK_GRAPHS` (function + view) | Graph runs completed/failed/cancelled in the **last 60 minutes** (function) |
| `TASK_VERSIONS` | Version history |
| `TASK_DEPENDENTS` | All child tasks under a root |

---

## 11. Error & Success Notifications (Cloud Messaging)

### Concept
Snowflake can push a message to **your own cloud messaging service** when a task errors out, or when an entire graph finishes successfully.

### 🚨 Gotcha #12 — Supported services are limited, and NOT cross-cloud
Only these three are supported, and you must use the one matching **your Snowflake account's own cloud platform** (no cross-cloud notifications):
- AWS **SNS**
- Azure **Event Grid**
- Google **Pub/Sub**

🚨 **Email and webhook notification integrations are explicitly NOT supported** for this specific task-notification feature (even though webhooks/email work for other Snowflake notification use cases, like alerts). This is a very "gotcha-y" distinction the exam loves.

### At-least-once delivery
Snowflake **guarantees at least one delivery attempt succeeds** — meaning **duplicate messages are possible**, and your downstream consumer needs to handle idempotency/dedup itself.

### Error notifications
- Config: `ERROR_INTEGRATION = <integration_name>` in `CREATE TASK` / `ALTER TASK`.
- 🚨 **Only set on the root task of a graph** — a failing child task's error still gets routed through the root's integration (children don't need/can't have their own).
- If `TASK_AUTO_RETRY_ATTEMPTS > 0`, **each failed retry attempt sends its own notification** (so you could get multiple error messages for one logical failure).
- Requires: `USAGE` on the notification integration + (`CREATE TASK` on schema OR `OWNERSHIP` on the task).

### Success notifications
- Config: `SUCCESS_INTEGRATION = <integration_name>`.
- 🚨 **Only fires for a fully successful ENTIRE task graph** — standalone tasks running successfully **never** send a success notification, even if configured. This asymmetry (error = any task in a graph; success = graph-level only, no standalone support at all) is a classic exam trap.
- Shown via `SHOW TASKS` / `DESC TASK` in the `success_integration` column — **NULL for all child tasks**, populated only on the root.

### 🎯 Exam Trap — Compare Error vs Success Notification
| | Error Notification | Success Notification |
|---|---|---|
| Set on | Root task only | Root task only |
| Standalone tasks supported? | Yes (a standalone task failing sends error) | **No — graph only** |
| Triggers on | Any task run that fails/times out | Only when the **whole graph** completes successfully |
| Retry behavior | Each failed retry sends its own message | N/A |

---

## 12. Monitoring Events for Task Executions (Event Table)

### Concept
Instead of cloud-messaging notifications, you can log task run outcomes as **events** into an event table — then build alerts or run SQL queries against it.

### 🚨 Gotcha #13 — Nothing is captured unless you explicitly set the severity level
By default, **no task events are logged at all.** You must set `LOG_EVENT_LEVEL` at account, database/schema, or individual task level:
- `ERROR` → only failed runs.
- `INFO` → both successful and failed runs.

```sql
ALTER ACCOUNT SET LOG_EVENT_LEVEL = ERROR;
ALTER TASK my_task SET LOG_EVENT_LEVEL = ERROR;
```

### Limitation
**Task events aren't supported for Snowflake Native Apps.**

### Cost note
Logging task events **incurs telemetry costs** — this is a real cost consideration, not just a technical footnote.

### How to build an alert
1. Set `LOG_EVENT_LEVEL`.
2. Create an "alert on new data" against the event table, filtering `resource_attributes:"snow.executable.type" = 'TASK'` and `value:state = 'FAILED'`.
3. Wire the alert to a notification (e.g., Slack webhook via `SYSTEM$SEND_SNOWFLAKE_NOTIFICATION`).

### Key fields to remember
- `resource_attributes` → identifies the task (`snow.executable.name`, `snow.executable.type = 'TASK'`, `snow.database.name`, `snow.warehouse.name`, `snow.owner.name`, `snow.query.id`, etc.)
- `record.severity_text` → `INFO` (success) or `ERROR` (failure).
- `value.state` → `SUCCEEDED` or `FAILED`; `value.message` → the error text (only present on failure).

### 🎯 Exam Trap: Event Table vs Cloud Notification Integration — two DIFFERENT monitoring mechanisms
Don't confuse these:
- **Notification Integration (Section 11)** = push to AWS SNS / Azure Event Grid / GCP Pub/Sub. External, real-time push.
- **Event Table (Section 12)** = internal Snowflake logging you must query yourself (or pair with an Alert). Requires you to explicitly set `LOG_EVENT_LEVEL`, or you get *nothing*.

---

## 13. Viewing Tasks & Task Graphs in Snowsight

Exam relevance is lower here (mostly navigation), but a few facts are testable:

- Path: **Catalog » Database Explorer » [db/schema] » Tasks**, or **Transformation » Tasks** for account-wide history.
- 🚨 **Editing a task in Snowsight automatically suspends it, then resumes it when you finish** — this is the UI doing the suspend/modify/resume dance for you behind the scenes.
- 🚨 **Task history data (in Snowsight) is only available if the task ran in the last 7 days.**
- Retry mechanics in Snowsight:
  - **Auto-retry**: governed by `TASK_AUTO_RETRY_ATTEMPTS` at the root.
  - **Manual retry**: available within **14 days** of the latest graph run, only if the graph hasn't been modified since, and only re-runs the failed/cancelled tasks (already-succeeded tasks are not re-run).
  - Requires **OPERATE** privilege (not necessarily OWNERSHIP) to retry.
  - Account-level task run history has up to a **45-minute latency** for showing retry attempts.

### 🎯 Exam Trap
"Do you need OWNERSHIP to retry a failed task in Snowsight?" → **No — OPERATE privilege is enough**, same pattern as suspend/resume (Gotcha #7).

---

## 14. Python and Java Support for Serverless Tasks

### Concept
A task's `AS` clause can call:
- A **UDF** (for custom computation logic returned as a value).
- A **stored procedure** (for administrative, multi-statement, or table-returning operations).
- Or embed Python/Java code **directly inline** in the AS clause.

### When to use which (this maps to the general UDF-vs-stored-procedure distinction, which IS separately tested)
| | UDF | Stored Procedure |
|---|---|---|
| Purpose | Compute/transform a value | Perform admin/orchestration logic, run SQL |
| Called from | Inside a SQL expression | `CALL procedure_name(...)` |
| Typical task use | `SELECT add_one(SYSTEM$GET_PREDECESSOR_RETURN_VALUE())` | `CALL filter_by_role(table, role)` |

### Example pattern — passing a predecessor's return value into a UDF/proc
```sql
CREATE TASK my_task2
  AFTER my_task1
  AS
    SELECT add_one(SYSTEM$GET_PREDECESSOR_RETURN_VALUE());
```
This combines two concepts from Section 9 (task graph return values) with actual UDF logic — a good "connect the dots" exam-style question.

---

## 15. Troubleshooting Tasks — The Diagnostic Checklist

This page is basically the **exam's favorite scenario-question source** — "a task isn't running, what do you check first?" Memorize this order.

### "My task did not run" — checklist
1. **Verify it actually didn't run** — query `TASK_HISTORY`. (It might have *run* but the SQL inside it failed — check `SCHEDULED_TIME`, `COMPLETED_TIME`, error code/message. If it's a child task, check whether its **predecessor** succeeded.)
2. **Verify it was RESUMED** — remember, tasks are born SUSPENDED (Gotcha #1). Check via `DESCRIBE TASK` / `SHOW TASKS`. Also double check the CRON expression is correct and that at least one scheduled occurrence has actually passed.
3. **Verify privileges on the task owner role** — `EXECUTE TASK` (account), `USAGE` (db/schema), `OWNERSHIP` (task), `USAGE` (warehouse). Use `SHOW GRANTS TO ROLE`.
4. **Verify the WHEN condition** — for triggered tasks, check whether the stream actually had CDC data at the scheduled time. You can query historical stream state with an `AT | BEFORE` clause.
5. **Check predecessor tasks** — in a graph, if a parent failed, all its children are **skipped**, not run.

### "My task timed out / exceeded its schedule window"
- **Default single-run limit: 60 minutes (3,600,000 ms)** — a safeguard against runaway/non-terminating tasks.
- Query `TASK_HISTORY` to see if it was cancelled or exceeded its window — usually caused by an **undersized warehouse**.
- Fix options: increase warehouse size, OR increase `USER_TASK_TIMEOUT_MS`.
- Check current timeout: `SHOW PARAMETERS LIKE 'USER_TASK_TIMEOUT_MS' IN TASK <name>;` — no result = still on the 60-minute default.
- 🚨 **Gotcha:** neither bumping the warehouse size NOR the timeout will help if the real problem is a **query parallelization bottleneck** in the SQL itself — sometimes the fix is rewriting the query, not throwing more compute at it.

---

## 16. Common Misconceptions (Read This Twice)

1. ❌ "A task will run as soon as I create it with a schedule." → **False.** All tasks start SUSPENDED; you must RESUME.
2. ❌ "Serverless tasks can scale to any warehouse size if needed." → **False.** Capped at XXLARGE; bigger needs = user-managed.
3. ❌ "If a scheduled task is still running, the next scheduled run just queues up and runs right after." → **False.** It's **skipped**, not queued.
4. ❌ "Success notifications work for any successfully completed task." → **False.** Only for a fully successful **entire task graph** via the root — standalone tasks never send success notifications.
5. ❌ "Triggered tasks constantly poll and burn compute waiting for stream data." → **False.** They use **zero compute** until the event actually fires.
6. ❌ "A task's owner needs OWNERSHIP privilege to suspend/resume it." → **False.** `OPERATE` is enough.
7. ❌ "If I modify a running task's SQL, the in-flight run picks up my change immediately." → **False.** The current run finishes on its original version; only the next resume/execute uses the new version.
8. ❌ "Task events are automatically logged once you have an event table set up." → **False.** You must explicitly set `LOG_EVENT_LEVEL` — otherwise, nothing is captured, no matter what event table exists.
9. ❌ "A child task with a suspended parent always gets skipped." → **False.** It runs as long as at least one resumed predecessor succeeds.

---

## 17. Master Comparison Table — Quick Revision

| Feature | Standalone Task | Task Graph (Root) |
|---|---|---|
| Default failure suspend threshold | Not fixed by default (must set) | **10 consecutive failures** |
| Retry granularity | Retries the task itself | Retries the **entire graph** |
| Success notification | ❌ Never sent | ✅ Sent when whole graph succeeds |
| Error notification | ✅ Sent on failure | ✅ Sent (config lives on root only) |
| Versioning | Task-level version | Whole-graph version, set together |
| Timeout scope | Applies to itself | Root timeout = whole graph; child timeout overrides for that child only |

| Feature | Serverless | User-Managed |
|---|---|---|
| `WAREHOUSE` param | Not used | Required |
| Max size | XXLARGE | Unlimited (any warehouse size) |
| Privilege | `EXECUTE MANAGED TASK` | `USAGE` on warehouse |
| Billing | Compute-hours (actual usage) | Warehouse-second, 60s minimum |
| For Triggered Tasks | Requires `TARGET_COMPLETION_INTERVAL` | Requires `WAREHOUSE` |

---

## 18. One-Page Cheat Sheet (Memorize These Numbers)

- Tasks are created **SUSPENDED** — always.
- Default task timeout = **60 minutes**.
- Default task graph auto-suspend = **10 consecutive failures**.
- Max task graph size = **1000 tasks**; max **100 parents / 100 children** per task.
- Serverless max compute = **XXLARGE**.
- Triggered task default check interval = **30 seconds**; can lower to **10 seconds** via parameter.
- Health check runs if a triggered task is idle for **12 hours** (timing not guaranteed).
- Task history in Snowsight only shows **last 7 days**.
- Manual retry of a failed graph run only works within **14 days**.
- Notification services supported: **AWS SNS, Azure Event Grid, Google Pub/Sub only** — no email/webhook, no cross-cloud.
- `COMPLETE_TASK_GRAPHS` function shows graphs completed in the **last 60 minutes**.
- Suspend/Resume/Retry needs **OPERATE**, not OWNERSHIP.
- Success notifications = **graph-level only**, never standalone.

---

## 19. Mini Mock Test

**Conceptual (5)**

1. What state is a newly created task in by default?
   *Answer: SUSPENDED. It must be explicitly resumed with `ALTER TASK … RESUME`.*

2. What is the maximum compute size available for a serverless task?
   *Answer: XXLARGE. Anything larger requires a user-managed task with a bigger warehouse.*

3. What privilege is sufficient to suspend or resume a task without owning it?
   *Answer: OPERATE privilege on the task (plus USAGE on the containing db/schema).*

4. Which three cloud messaging services support Snowflake task notifications?
   *Answer: AWS SNS, Microsoft Azure Event Grid, Google Pub/Sub. (Email and webhook are NOT supported for this feature.)*

5. By default, how long can a single task run before timing out?
   *Answer: 60 minutes (3,600,000 ms), via the default USER_TASK_TIMEOUT_MS.*

**Scenario (3)**

6. A root task has `SCHEDULE = '5 MINUTE'` and `OVERLAP_POLICY = NO_OVERLAP` (default). Its full task graph consistently takes 7 minutes to finish. What happens?
   *Answer: At least one scheduled run gets skipped — no overlap, no queuing. To fix: increase the interval, use bigger/serverless compute, or explicitly allow overlap.*

7. A triggered task watches `SYSTEM$STREAM_HAS_DATA('stream_a') AND SYSTEM$STREAM_HAS_DATA('stream_b')`. Only stream_a has new data. What happens?
   *Answer: The task is skipped entirely — AND requires both conditions true.*

8. A child task has two parent tasks; one parent is suspended, the other succeeds. Does the child run?
   *Answer: Yes — it runs as long as at least one resumed predecessor completes successfully; a suspended parent doesn't block it.*

**True/False (2)**

9. "A finalizer task can have its own child tasks." → **False.** Finalizers are always terminal — no children allowed.

10. "Task events are automatically logged to the event table as soon as an event table is associated with the database." → **False.** You must explicitly set `LOG_EVENT_LEVEL` (ERROR or INFO) — otherwise nothing is captured.

**Trick Question (1)**

11. "You set `TARGET_COMPLETION_INTERVAL = '10 MINUTES'` on a serverless task. Will Snowflake guarantee the task finishes in 10 minutes?"
    *Answer: No. Snowflake scales resources to try to hit that target, but if the task is already at max warehouse size (XXLARGE) and still running long, the target is simply ignored — there's no hard guarantee, only a best-effort scaling behavior.*

---

*Notes compiled from Snowflake's official documentation pages on Tasks (as of July 2026). Next up whenever you're ready: **Streams** (streams-intro, streams-manage) to complete the Streams + Tasks pairing.*
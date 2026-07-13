I've got everything needed. Now let me put together comprehensive exam-focused notes on Virtual Warehouses.

---

# 📘 Virtual Warehouses — SnowPro Core Notes

## 1. What is a Virtual Warehouse?

A **compute engine** — the thing that actually runs your queries and loads data. It's completely separate from **storage** (that's why Snowflake's architecture is called "multi-cluster shared data" — storage is one layer, compute is another).

Key mental model: **No warehouse = no queries.** You can't even run a `SELECT` without one (except a few metadata-only commands like `SHOW TABLES`, which don't need compute at all — free of charge).

---

## 2. Warehouse Sizing

| Size | Credits/hour (Gen1) |
|---|---|
| X-Small | 1 |
| Small | 2 |
| Medium | 4 |
| Large | 8 |
| X-Large | 16 |
| 2X-Large | 32 |
| 3X-Large | 64 |
| 4X-Large | 128 |
| 5X-Large | 256 |
| 6X-Large | 512 |

**Pattern to memorize:** each size doubles the credits of the one below it.

⚠️ **Gotcha:** Default size differs by interface:
- `CREATE WAREHOUSE` in SQL → defaults to **X-Small**
- Creating via **Snowsight UI** → defaults to **X-Large**

⚠️ 5X-Large and 6X-Large are GA on AWS/Azure only; preview in US Gov regions.

---

## 3. Billing — the exact rules (exam loves this)

- **Per-second billing**, but with a **60-second minimum** charge every time compute starts.
- No benefit to stopping before 60s — you're already billed for the full minute.
- After the first 60 seconds, billing is per-second.

**Memorize this exact scenario (it's a classic exam trap):**
> Warehouse runs 61 seconds → shuts down → restarts → runs <60 seconds again
> → Billed: 60 + 1 + 60 = **121 seconds**, not 61+60=121... wait, actually it IS 121, but the point is **each restart re-triggers the 60s minimum**, even for a tiny second run.

- **Resizing** provisions more compute immediately → billed from that moment.
- Resizing between 5XL/6XL ↔ 4XL-or-smaller → brief period billed for **both** old and new warehouse (during quiesce).

---

## 4. Auto-Suspend / Auto-Resume

- **Both enabled by default.**
- Auto-suspend: warehouse shuts down after inactivity period you set.
- Auto-resume: warehouse wakes up automatically when a query needs it.

⚠️ **Gotcha for multi-cluster warehouses:**
- Auto-suspend only kicks in when the **minimum** number of clusters is running (not necessarily 0 clusters) AND there's no activity.
- Auto-resume only applies when the **entire warehouse** is suspended (all clusters down).

**Recommendation from docs:** keep auto-suspend low (5-10 min) because of per-second billing — but not *so* low that normal query gaps cause constant suspend/resume cycles (each resume re-triggers the 60s minimum charge).

To fully disable auto-suspend: must explicitly set **`0`** or **`NULL`** in SQL, or select "Never" in UI.

---

## 5. Caching Gotcha (high-yield exam topic)

- Each **running** warehouse maintains a **data cache** (results of table data it has scanned).
- **Cache size scales with warehouse size** — bigger warehouse = bigger cache.
- **Suspending a warehouse drops its cache.** Next resume = cold cache = slower first queries until cache rebuilds.
- **Resizing down also drops cache** tied to removed compute resources — same performance hit as suspend.

➡️ Classic exam question: "Why did query performance degrade after resizing a warehouse down / after resuming a suspended warehouse?" → **Cache was dropped.**

---

## 6. Multi-Cluster Warehouses (Enterprise Edition+ only)

**Purpose: solve CONCURRENCY problems, not slow single-query performance.** (Resizing solves slow queries; multi-cluster solves queuing from too many simultaneous users/queries.)

### Setup
- Defined by **MIN_CLUSTER_COUNT** and **MAX_CLUSTER_COUNT**.
- Max cluster count depends on warehouse size (bigger size = lower max clusters allowed):

| Size | Max clusters |
|---|---|
| X-Small/Small/Medium | 300 |
| Large | 160 |
| X-Large | 80 |
| 2X-Large | 40 |
| 3X-Large | 20 |
| 4X-Large, 5X-Large, 6X-Large | 10 |

⚠️ Snowsight UI caps you at 10 clusters max — to go higher, you **must use SQL**.

### Two modes
- **Maximized**: `min = max` (and >1). All clusters start immediately when warehouse starts. Static.
- **Auto-scale**: `min < max`. Snowflake starts/stops clusters dynamically based on load.

### Scaling Policies (Auto-scale mode only — irrelevant in Maximized mode)
| Policy | Behavior |
|---|---|
| **Standard** (default) | Favors starting clusters fast to avoid queuing, even if it costs more credits |
| **Economy** | Favors conserving credits; only starts a new cluster if it estimates ≥6 min of work to justify it; shuts down clusters with <6 min of work left |

⚠️ **Gotcha:** "Legacy" scaling policy was **removed** — any warehouse using it is now silently on Standard.

⚠️ **Gotcha:** Interactive warehouses only support **Standard** scaling policy (not Economy), and scale more proactively/aggressively.

### Credit calc example (exam loves these):
Medium warehouse (4 credits/hr) with 3 clusters, Maximized mode, running 2 hours straight:
= 4 credits × 3 clusters × 2 hours = **24 credits total**

For Auto-scale, you must track each cluster's individual runtime and sum.

### Best practice from docs
- If on Enterprise+, **all warehouses should be multi-cluster** (even if min=max=1, gives you headroom).
- Keep MIN at 1 unless you need HA guarantees.
- Set MAX as high as you're comfortable paying for.

---

## 7. Query Acceleration Service (QAS) — Enterprise Edition+

**What it does:** offloads parts of expensive/outlier queries to shared serverless compute so one heavy query doesn't hog the warehouse for everyone else.

### Eligible query patterns
1. Large scans + selective filter/aggregation
2. Large INSERT/COPY/UPDATE/DELETE operations

### Ineligible reasons (exam-testable)
- Not enough partitions to scan (overhead > benefit)
- Filters not selective enough / GROUP BY cardinality too high
- Nondeterministic functions (`RANDOM()`, `SEQ`)
- (Used to exclude LIMIT-without-ORDER-BY, but Snowflake now auto-detects eligibility for these)

### Enabling
```sql
CREATE WAREHOUSE my_wh WITH ENABLE_QUERY_ACCELERATION = true;
```
⚠️ **Gotcha — default scale factor differs by context:**
- Explicitly enabled via `ENABLE_QUERY_ACCELERATION = TRUE` → default scale factor = **8**
- Auto-enabled on **Gen2** or **multi-cluster** warehouse creation → default scale factor = **2**

⚠️ **Gotcha:** Creating a **new** multi-cluster warehouse auto-enables QAS. But **converting** an existing single-cluster warehouse to multi-cluster does **NOT** auto-enable QAS — it stays whatever it was.

- Billed **separately** from warehouse compute, per-second, serverless.
- Tools to check eligibility: `SYSTEM$ESTIMATE_QUERY_ACCELERATION()` function, and `QUERY_ACCELERATION_ELIGIBLE` account_usage view.

---

## 8. Snowpark-Optimized Warehouses

- Purpose: memory-heavy workloads (ML training via stored procs, UDFs/UDTFs with big memory needs).
- Default: **16x memory per node** vs standard warehouse.
- Configurable via `RESOURCE_CONSTRAINT`:

| Memory | RESOURCE_CONSTRAINT | Min warehouse size |
|---|---|---|
| 16GB | MEMORY_1X / MEMORY_1X_x86 | XSMALL |
| 256GB | MEMORY_16X / MEMORY_16X_x86 | Medium |
| 1TB | MEMORY_64X / MEMORY_64X_x86 (preview, AWS only) | Large |

⚠️ Doesn't apply to Gen2 (Gen2 clause is for standard warehouses only).
⚠️ Creation/resumption can take **longer** than standard warehouses.

---

## 9. Gen2 Standard Warehouses

- "Next-gen" hardware, faster DML (update/delete/merge) and table scans.
- Set via `GENERATION = '2'` (recommended) or `RESOURCE_CONSTRAINT = STANDARD_GEN_2`.
- ⚠️ **NOT available in Snowsight UI** — SQL only.
- ⚠️ **Not available for X5Large/X6Large** sizes.
- Converting Gen1→Gen2 while warehouse is **running**: old queries finish on Gen1 resources, new queries go to Gen2 — you're **billed for both simultaneously** until old queries drain. Convert while **suspended** to avoid double billing.

---

## 10. Interactive Warehouses (newer, GA in select regions) — worth knowing edge cases

- Purpose: low-latency, high-concurrency (dashboards, APIs).
- Works only with **Interactive Tables** (created via `CREATE INTERACTIVE TABLE ... CLUSTER BY (...)` — CLUSTER BY is **mandatory**).

Key gotchas:
- **Statement timeout fixed at 5 seconds** for SELECT — you can lower it, **cannot** raise it.
- **Minimum auto-suspend = 24 hours (86400 sec)** — wildly different from standard warehouses.
- Minimum **billable period = 1 hour** (not 60 seconds like standard warehouses!).
- Can only query interactive tables — **cannot** query standard/hybrid tables from an interactive warehouse (must `USE WAREHOUSE` to switch).
- Max **10 interactive tables** per interactive warehouse (temporary limit).
- No stored procedure `CALL`, no `->>` pipe operator (uses stored procs internally).
- Interactive tables **don't support** DML except `INSERT OVERWRITE`; no Fail-safe (but Time Travel still works); can't be a Dynamic Table source.
- Supports a **fallback warehouse** — if a query times out at 5s, it auto-retries on a standard fallback warehouse.

---

## 11. Adaptive Warehouses (very new — GA select AWS regions only)

- "No more picking sizes" — Snowflake auto-scales/auto-routes per query.
- Controlled by two knobs instead of size:
  - `MAX_QUERY_PERFORMANCE_LEVEL` (t-shirt size, default XLARGE) — upper bound of performance per query
  - `QUERY_THROUGHPUT_MULTIPLIER` (integer, default 2, `0` = unlimited) — controls concurrency/throughput budget
- **Query-based billing**, not warehouse-size-based.
- Can convert standard ↔ Adaptive **online**, no downtime (same double-billing-during-transition pattern as Gen1→Gen2).
- ⚠️ Cannot convert to/from X5Large/X6Large, Snowpark-optimized, or Interactive warehouses.
- Enable/disable via `ALTER WAREHOUSE ... ENABLE/DISABLE` — disabled = rejects new jobs but lets running ones finish.

*(Likely too new for current SnowPro Core question bank — but good to know exists.)*

---

## 12. Scaling Up vs Scaling Out (classic conceptual exam question)

| | Scale UP (resize) | Scale OUT (multi-cluster) |
|---|---|---|
| Fixes | Slow individual queries | Queuing / concurrency from many users |
| Requires | Any edition | Enterprise Edition+ |
| Effect on running queries | Doesn't help queries already running; only helps queued/new queries | New clusters absorb new/queued queries |

---

## 13. Warehouse Usage in Sessions (defaults precedence)

Order of precedence for which warehouse a session uses (each overrides the previous):
1. Default warehouse for the **user** (`CREATE/ALTER USER`)
2. Default in **client config file** (SnowSQL, JDBC, etc.)
3. Default passed via **command line / driver connection parameter**

Can always override mid-session with `USE WAREHOUSE`.

---

## 14. Monitoring Warehouse Load

- **Query Load** = execution time of all queries in interval ÷ total interval time. A load of 1.0 = warehouse fully utilized.
- Requires **MONITOR** privilege to view the load chart.
- Statuses tracked: Running, Queued (Provisioning), Blocked, Queued (overload).

**Decision rules (exam-testable):**
- High running load + queuing → add a warehouse or add multi-cluster capacity.
- Low running load + slow queries → **resize up** (more resources per query).
- Warehouse running but load consistently < 1 → **wasting credits**, downsize or lower MIN_CLUSTER_COUNT.

---

## Quick-Fire Gotcha Summary (for rapid revision)

| Trap | Answer |
|---|---|
| Default warehouse size via SQL vs Snowsight | X-Small (SQL) vs X-Large (Snowsight) |
| Multi-cluster requires | Enterprise Edition+ |
| QAS requires | Enterprise Edition+ |
| Multi-cluster solves | Concurrency, NOT slow queries |
| Resizing solves | Slow queries, NOT concurrency |
| Suspending/resizing-down drops | The warehouse's data cache |
| Auto-suspend/resume applies to | Whole multi-cluster warehouse, not individual clusters |
| Legacy scaling policy | Removed, silently became Standard |
| QAS default scale factor | 8 (explicit) vs 2 (Gen2/multi-cluster auto-enabled) |
| Converting single→multi-cluster | Does NOT auto-enable QAS |
| Interactive warehouse min auto-suspend | 24 hours (86400s), NOT low like standard |
| Interactive warehouse min billing period | 1 hour, NOT 60 sec |
| Interactive warehouse SELECT timeout | Fixed 5s, can lower not raise |
| Gen2 available in Snowsight? | No — SQL only |
| Gen2 available for X5L/X6L? | No |

---

Want me to move to **Snowpipe** next (the classic `data-load-snowpipe-*` series), or generate a batch of MCQs on virtual warehouses first using your practice-question workflow?
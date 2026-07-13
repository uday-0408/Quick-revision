# Resource Monitors — Chapter 1

Before we dive in, here's where this topic sits in the bigger picture (this will help all your other topics click into place later):

```
Snowflake Cost Management has 3 pillars:

1. UNDERSTAND cost  → cost-understanding-compute, cost-understanding-data-transfer
2. CONTROL cost      → Resource Monitors ⬅ we are here, Budgets, warehouse-level controls
3. OPTIMIZE cost     → cost-optimize

...and ACCOUNT_USAGE is the SQL "black box recorder" that feeds data into all three.
```

Resource Monitors live in the **Control** pillar. Let's build this from zero.

---

## 1. What is a Resource Monitor? (in plain English)

Think of it like the **fuel gauge + auto-kill-switch** on a rental car you've given to a teenager. Snowflake warehouses burn **credits** (money) every second they run. A Resource Monitor:

- Watches how many credits get burned by your warehouses
- Can **tap you on the shoulder** (send a notification) when spending gets close to a limit
- Can **grab the wheel and stop the car** (suspend the warehouse) if spending hits the limit

It's an object in Snowflake — you create it, name it, and attach it to something.

**Only the `ACCOUNTADMIN` role can create one.** Other roles can be *given* permission to view or tweak an existing one, but they can't spin up a new one from scratch. Keep this in your back pocket — it's tested constantly.

---

## 2. The #1 Gotcha — Memorize This Before Anything Else

> **Resource Monitors only watch warehouses.** Nothing else.

That means credits burned by **Snowpipe, Automatic Clustering, Materialized View maintenance, Search Optimization, AI Services, replication** — none of that is seen by a Resource Monitor. Also: **storage costs** and **data transfer costs** are completely invisible to it too.

**Why does this matter?** Because SnowPro loves to write a question like: *"A company wants to cap total spending including Snowpipe costs — what should they configure?"* If you answer "Resource Monitor," you're wrong. The answer is a **Budget** (we'll compare them properly in section 9).

**Why did Snowflake design it this way?** Warehouses are *user-controlled* — someone can accidentally leave one running, or run a runaway query. Serverless features are more automatically managed by Snowflake itself, so they got a separate, broader tool (Budgets) later.

---

## 3. The Anatomy of a Resource Monitor (4 properties)

Every Resource Monitor is built from exactly 4 building blocks:

### 3.1 Credit Quota

The number of credits allowed in one interval (e.g., 1000 credits/month). Snowflake keeps a running counter of "used credits" that resets to 0 at each interval.

**Gotcha (important, testable):** The quota counts credits from **both** the warehouse itself **and** the cloud services layer that supports it. Example: warehouse burns 700, cloud services burns 300 → that's 1000 total against your quota, even though "cloud services" feels like a separate thing.

**The sneaky gotcha inside that gotcha:** Snowflake normally gives you a **free daily 10% cloud-services adjustment** — if your cloud services usage is under 10% of your warehouse usage for the day, you don't get billed for it. **Resource Monitors completely ignore this discount.** They count 100% of raw cloud services consumption toward your quota, even the portion that would never appear on your actual bill.

*Real exam trap:* "My bill shows $0 extra for cloud services, so why did my Resource Monitor fire a Notify action?" → Because the RM math doesn't apply the 10% waiver. Your bill and your RM's internal counter are NOT the same number.

### 3.2 Monitor Type: Account vs Warehouse

| | Account Monitor | Warehouse Monitor |
|---|---|---|
| Covers | **All** warehouses in the account | Only the warehouse(s) assigned to it |
| How many allowed | **Exactly 1** per account | Many allowed |
| Warehouse limit | N/A | Up to 500 warehouses per monitor |

**Gotcha:** If you create a Resource Monitor but never set its type (never assign it to the account or to a warehouse), it just sits there **doing absolutely nothing** — it's "dormant." Creating it isn't enough; you have to attach it.

**Gotcha:** A single warehouse can only belong to **one** warehouse-level monitor at a time — but it is *still separately* covered by the account monitor if one exists (more on this in section 4, this is a big one).

### 3.3 Schedule (Frequency / Start / End)

- **Frequency** — how often the used-credit counter resets: `Daily`, `Weekly`, `Monthly` (the default), `Yearly`, or `Never` (counter never resets — once you hit quota, that's it forever until you manually raise it).
- **Start** — `Immediately` or a future timestamp.
- **End** — optional. A future timestamp where Snowflake **force-suspends every assigned warehouse regardless of whether the quota was ever reached.**

**Gotcha #1:** Resets always happen at **12:00 AM UTC**, no matter what local time you specified as your start time.

**Gotcha #2 (classic exam trick):** If you set your start date to the **last day of a month** (like Jan 31) with Monthly frequency, Snowflake doesn't try to reset on "the 31st" of every month. It resets on the **last day** of each following month: Feb 28 (or 29), Mar 31, Apr 30... It's smart about it, so don't assume it breaks in February.

**Gotcha #3:** If you customize the frequency, you're *required* to also set a start timestamp — you can't set one without the other.

**Gotcha #4:** Once you customize a schedule away from the default, **you cannot switch it back**. You'd have to drop the monitor and create a brand-new one.

**Gotcha #5 (very sneaky):** The **End** date doesn't just "stop tracking" — it actively fires a **Suspend Immediate** on every assigned warehouse, and the notification it sends will awkwardly phrase this as "reached X% of quota," even if the real reason is just "time ran out," not "you overspent."

### 3.4 Actions (a.k.a. Triggers)

An action = a threshold (% of quota) + what to do when it's crossed.

| Action | What happens |
|---|---|
| **Notify** | Just sends an alert. Nothing else. |
| **Notify & Suspend** | Sends alert + lets currently running queries **finish**, then suspends the warehouse. **Blocks new queries** from starting once triggered. |
| **Notify & Suspend Immediately** | Sends alert + **cancels** whatever is currently running + suspends right away. |

**Exact limits (memorize this cold):**
- Maximum **1** Suspend action
- Maximum **1** Suspend Immediate action
- Maximum **5** Notify actions
- Thresholds can go **above 100%** (e.g., a Suspend Immediate set at 110%, to allow a small buffer before the hard stop)

**Gotcha:** A monitor with a quota but **zero actions defined** will silently track usage forever and never actually *do* anything when the quota is blown. No notification, no suspend. Nothing.

**Gotcha (huge one):** With a plain **Suspend**, queries that were already running when the threshold hit are allowed to complete — meaning your **actual spend can exceed the quota**. This is documented and expected behavior, not a bug.

---

## 4. How Account-Level and Warehouse-Level Monitors Work Together

This relationship gets tested a lot, so let's build the mental model with real numbers.

```
Resource Monitor A (Account-level): 5000 credit quota, Suspend @ 100%
   covers ALL 5 warehouses in the account
        │
        ├── Warehouse 1 (no dedicated monitor — only Monitor A applies)
        ├── Warehouse 2 (no dedicated monitor — only Monitor A applies)
        ├── Warehouse 3 → also has its OWN Monitor B: 1000 credit quota
        ├── Warehouse 4 → also has its OWN Monitor C: 2500 credit quota (shared with WH5)
        └── Warehouse 5 → shares Monitor C with WH4
```

**The rule: it's "OR" logic, not "override" logic.**

- If Monitor A (account) hits 5000 credits → **all 5 warehouses** get suspended, even if Warehouse 3 has only used 200 of its own 1000 credit allowance.
- If Warehouse 3's own Monitor B hits 1000 credits → **only Warehouse 3** gets suspended, even if the account is nowhere near 5000.

**Whichever limit is hit first "wins" and fires first.** The account monitor does **not** override or replace the individual warehouse monitors — they run in parallel, watching the same warehouse from two different angles.

**Gotcha:** If Warehouse 4 and Warehouse 5 **share** the same monitor (Monitor C), they share the **same pool** of 2500 credits. A greedy Warehouse 4 can eat the whole 2500 and get an innocent, barely-used Warehouse 5 suspended too. **Snowflake's own recommendation:** if you want strict, isolated control per warehouse, assign **one warehouse per monitor**, not several.

---

## 5. Suspension and Coming Back to Life

Once a Suspend/Suspend Immediate fires, the warehouse **cannot resume** until *one* of these happens:

1. The next interval starts (e.g., the month rolls over)
2. Someone raises the credit quota
3. Someone raises the threshold % for that action
4. The warehouse is removed from the monitor
5. The monitor itself is dropped

There's no "just wait and it'll be fine" option unless the interval genuinely resets.

**Adaptive Warehouses are different terminology, same idea:** for [Adaptive Warehouses](https://docs.snowflake.com/user-guide/warehouses-adaptive), Suspend/Suspend Immediate = **Disable**. You'll see `STATE = DISABLED` with `DISABLED_REASONS` pointing at the monitor. When conditions clear, it auto-**enables** again (same effect as running `ALTER WAREHOUSE ... ENABLE`).

---

## 6. What a Resource Monitor Can NEVER Do (Boundaries)

- ❌ Cannot directly suspend cloud services usage on its own — the only lever it has is suspending the *warehouse*.
- ❌ Even after suspension, a query someone submits against that warehouse can still generate a **small residual cloud services charge** (e.g., the cost of Snowflake telling you "no, this warehouse is suspended"). "Suspended" doesn't mean "zero cost forever," it just means "no compute runs."
- ❌ Not built for split-second precision. Snowflake explicitly says: don't rely on it to cap spend down to the exact credit. Even Suspend Immediate has a bit of lag, so you can still slightly overshoot.
  - **Snowflake's own fix for this:** set your action threshold at ~90% instead of 100% to build in a safety buffer, and use one-warehouse-per-monitor for tighter control.
- ❌ An account monitor still doesn't touch serverless compute (Snowpipe, Auto-clustering, etc.) — repeating this because it's tested from multiple angles.

---

## 7. Notifications — Who Actually Gets Told

This is a detail people gloss over, and it's very testable.

- Notifications are **OFF by default**. You must explicitly opt in, and your email must be verified first.
- **For Account Monitors:** Only (a) the admin who owns it (if they've opted in) and (b) any other ACCOUNTADMIN who separately opted in via Snowsight. **A non-admin can never be added to an account monitor's notify list — period.**
- **For Warehouse Monitors:** All opted-in admins get notified, **plus** you can explicitly add up to **5 non-admin users** by name (`NOTIFY_USERS` parameter). But those non-admins only get **email**, never the in-app Snowsight alert — and if even one of those 5 users doesn't have a verified email, the whole command to add them **fails**.

---

## 8. Who's Allowed to Touch a Resource Monitor (Privileges)

| Action | Required |
|---|---|
| **Create** a new monitor | `ACCOUNTADMIN` only |
| **Set/change** account-level assignment | `ACCOUNTADMIN` only |
| **Change monitor type** (warehouse ↔ account) | `ACCOUNTADMIN` only |
| **Add/remove warehouses** from a warehouse monitor | `ACCOUNTADMIN` only |
| Change quota, schedule, or actions | Role with `MODIFY` privilege |
| View only | Role with `MONITOR` privilege |

**Gotcha:** Changing the quota, schedule, or actions is **never retroactive** — it only affects credit usage counted *after* the change is saved. Past usage-to-date stays exactly as it was.

**Gotcha (edition trap):** Replicating a resource monitor to another account (via replication/failover groups) requires **Business Critical edition or higher**.

---

## 9. Resource Monitor vs Budget — The Comparison SnowPro Loves to Ask

This is one of the single most common "which tool do I use" trap questions.

| | **Resource Monitor** | **Budget** |
|---|---|---|
| What it watches | Warehouses only (+ their cloud services) | Warehouses **and** serverless/AI features (Snowpipe, AI Services, Auto-Clustering, Materialized Views, Search Optimization, Replication, etc.) |
| Can it stop spending automatically? | **Yes**, natively — Suspend / Suspend Immediate, no code needed | **No**, not natively — it only *notifies*. To actually enforce a stop, you must write your **own** stored procedure and hook it in via "custom actions" |
| Alert trigger | Fires when **actual usage** crosses a real threshold | Fires when spend is **forecasted/projected** to exceed the limit (predictive, time-series based), sent daily |
| Interval | Flexible: daily/weekly/monthly/yearly/never, custom start/end | Fixed: always one calendar month |
| Who can create it | `ACCOUNTADMIN` only | A role with the right budget privileges (not necessarily `ACCOUNTADMIN`) |
| Replication | Possible (Business Critical+) | **Not possible**, ever |

**The one-line way to remember it:** Resource Monitor = a **hard brake pedal**, but only for warehouses. Budget = a **wide-angle smoke detector**, covering almost everything, but it only screams — it doesn't stop the car unless you personally wire up that logic yourself.

---

## 10. Real-World Example

Picture a bank's data platform team with 5 warehouses: `ETL_WH`, `BI_WH`, `ADHOC_WH`, `ML_WH`, `DEV_WH`.

Finance says: *"The whole account cannot exceed 5,000 credits/month."* Data engineering separately says: *"`DEV_WH` is used by junior engineers running experimental queries — cap it hard at 200 credits so one bad query can't hurt us."*

```sql
-- Account-level guardrail (needs ACCOUNTADMIN)
USE ROLE ACCOUNTADMIN;

CREATE OR REPLACE RESOURCE MONITOR RM_ACCOUNT_CAP
  WITH CREDIT_QUOTA = 5000
  FREQUENCY = MONTHLY
  START_TIMESTAMP = IMMEDIATELY
  TRIGGERS ON 75 PERCENT DO NOTIFY
           ON 100 PERCENT DO SUSPEND;

ALTER ACCOUNT SET RESOURCE_MONITOR = RM_ACCOUNT_CAP;

-- Dedicated tight leash on the risky dev warehouse
CREATE OR REPLACE RESOURCE MONITOR RM_DEV_STRICT
  WITH CREDIT_QUOTA = 200
  TRIGGERS ON 80 PERCENT DO NOTIFY
           ON 100 PERCENT DO SUSPEND_IMMEDIATE;

ALTER WAREHOUSE DEV_WH SET RESOURCE_MONITOR = RM_DEV_STRICT;
```

Result: If a junior engineer's bad query burns through 200 credits on `DEV_WH` alone, **only `DEV_WH`** gets killed immediately — the other 4 warehouses keep running untouched, because they're only watched by the 5,000-credit account monitor, which hasn't been threatened at all.

---

## 11. What Happens Internally (Behind the Scenes)

```
Query submitted to a warehouse
        │
        ▼
Warehouse consumes credits while running
        │
        ▼
Snowflake's metering service updates "used credits"
for EVERY resource monitor watching that warehouse
(its own warehouse monitor AND the account monitor, if any)
        │
        ▼
Each monitor independently checks: used ≥ any defined threshold?
        │
        ▼
   ┌────┴─────┐
   │  No       │  Yes → check which action is attached to that threshold
   └───────────┘         │
                          ▼
              Notify → send alert only
              Suspend → let running queries finish, then block new ones
              Suspend Immediate → cancel running queries + block new ones now
```

---

## 12. Common Misconceptions

- **"Resource Monitors control all my Snowflake costs."** → No. Warehouse compute only. Storage, data transfer, and serverless features are invisible to it.
- **"Suspend Immediate means zero cost from that second on."** → Not quite — tiny residual cloud services costs and some lag are still possible.
- **"If I have an account monitor, I don't need warehouse monitors."** → They're independent and both apply; one doesn't replace the other.
- **"The 10% free cloud services adjustment protects me from hitting my Resource Monitor threshold."** → It doesn't. RM math ignores that adjustment entirely.
- **"I can always switch a custom schedule back to default."** → You can't — you have to drop and recreate.
- **"Any admin automatically gets alerted."** → Notifications are opt-in and require a verified email, always.

---

## 13. SnowPro Exam Traps to Watch For

- A scenario shows a warehouse still burning credits *after* it crossed 100% quota with a plain **Suspend** action → this is **correct, expected behavior** (in-flight queries finish), not a malfunction.
- A question asks who can **create** a resource monitor → the answer is always `ACCOUNTADMIN`, never "any role with MODIFY" (MODIFY only lets you edit an *existing* one's quota/schedule/actions).
- A question tests exact action limits → 1 Suspend + 1 Suspend Immediate + up to 5 Notify. Any other combination is wrong.
- A question implies storage or data transfer is covered → it is **not**, that's a Budget's job (or other tools entirely).
- A question about whether raising the quota affects past usage → it does **not**, only future usage.
- A question about custom schedule start date landing on a month-end (like Jan 31) → remember it rolls to the **actual last day** of each following month, not a non-existent date.

---

## 14. Quick Revision Cheat-Sheet

- Created by: **ACCOUNTADMIN only**
- Watches: **Warehouses + their cloud services** — nothing else
- Properties: **Credit Quota, Monitor Type, Schedule, Actions**
- Monitor types: **1 Account monitor max**, unlimited **Warehouse monitors** (max 500 warehouses each, 1 monitor per warehouse below account level)
- Actions: **1 Suspend, 1 Suspend Immediate, up to 5 Notify** — thresholds can exceed 100%
- Default frequency: **Monthly**; other options: Daily/Weekly/Yearly/Never
- Resets always at: **12:00 AM UTC**
- Cloud services 10% billing adjustment: **ignored** by RM math
- Account + warehouse monitors: run **in parallel**, first one to hit its threshold wins
- Recovery from suspension: next interval starts / quota raised / threshold raised / warehouse unassigned / monitor dropped
- Can't precisely enforce to-the-credit limits → use ~90% buffer thresholds
- Not for serverless/storage/data transfer costs → that's what **Budgets** are for
- Replication of resource monitors → **Business Critical edition or higher**

---

## 15. Practice Questions

**Q1.** Which of the following costs is **NOT** something a Resource Monitor can ever control or suspend?

A. A standard virtual warehouse that reached its credit threshold
B. **Compute used by Snowpipe to continuously load streaming data** ✅
C. An Adaptive Warehouse that exceeded its quota
D. Cloud services credits consumed to support a running warehouse

*Answer: B. Resource Monitors only see warehouse-tied consumption. Snowpipe is a serverless feature and is completely outside their visibility — that's Budget territory.*

---

**Q2.** An account has an account-level monitor with a 5,000 credit quota and a Suspend action at 100%. Warehouse `ETL_WH` also has its own dedicated monitor with a 500 credit quota and a Suspend Immediate action at 100%. This month, `ETL_WH` has used 500 credits while total account usage is only 2,000 credits. What happens?

A. **`ETL_WH` is suspended immediately, because its own monitor independently reached its threshold** ✅
B. `ETL_WH` keeps running, because the account monitor hasn't reached its quota yet
C. `ETL_WH` is suspended only after its current queries finish, since Suspend always takes priority over Suspend Immediate
D. The account monitor's 5,000 credit quota is automatically reduced to 500 once a warehouse-level monitor is attached

*Answer: A. The two monitors act independently — whichever hits its own threshold first fires its own action. The account monitor being far from its limit is irrelevant to `ETL_WH`'s own monitor.*

---

**Q3. True or False:** If a Resource Monitor's threshold is reached and the action is **Suspend** (not Suspend Immediately), a query that was already running before the threshold was crossed will still be allowed to finish — even if that pushes total usage past the stated quota.

*Answer: **True.** This is documented, expected behavior. Suspend never cancels in-flight work; it only blocks new queries and waits for current ones to complete.*

---

**Q4.** A warehouse consumes 950 credits in a month, and its cloud services layer consumes 60 credits — an amount that falls under Snowflake's standard daily 10% cloud-services billing adjustment and would normally not appear on the invoice. The resource monitor watching this warehouse has a 1,000 credit quota with a Notify action set at 100%. Does the Notify action fire?

A. No, because the 60 credits fall under the free adjustment and are excluded from resource monitor math
B. No, because resource monitors never count cloud services credits at all
C. **Yes, because resource monitors count gross cloud services consumption regardless of the billing adjustment (950 + 60 = 1,010, over the 1,000 quota)** ✅
D. Yes, but only a partial notification is sent reflecting the post-adjustment billed amount

*Answer: C. The 10% adjustment is a billing-only concept. Resource monitor threshold math always uses raw, unadjusted cloud services consumption.*

---

**Q5.** A resource monitor uses a custom schedule with a Monthly frequency and a start date of March 31. On what date will Snowflake reset the used credits in April?

A. **April 30** ✅
B. April 31
C. May 31 of the following year
D. April 1 (the default monthly reset day)

*Answer: A. Snowflake resets on the actual last day of each following month when the start date falls on a month's last day. April only has 30 days, so April 31 doesn't exist — that option is a trap.*

---

**Q6.** Which combination of actions is a **valid** configuration for a single resource monitor?

A. Two Suspend actions and three Notify actions
B. **Two Notify actions, one Suspend action, and one Suspend Immediate action** ✅
C. One Notify action, one Suspend action, and two Suspend Immediate actions
D. Six Notify actions and one Suspend action

*Answer: B. The hard limits are exactly 1 Suspend, 1 Suspend Immediate, and up to 5 Notify. Every other option breaks one of those three limits.*

---

**Q7. True or False:** A single warehouse can be assigned to two different warehouse-level resource monitors at the same time, in addition to being covered by the account-level monitor.

*Answer: **False.** A warehouse can only belong to one resource monitor below the account level. It can, however, simultaneously be covered by the one account-level monitor if one exists — that part is fine.*

---

**Q8.** A user has been granted only the `MODIFY` privilege on an existing warehouse-level resource monitor (no `ACCOUNTADMIN` role). What is this user allowed to do?

A. **Raise the credit quota and add a new Notify action to the monitor** ✅
B. Convert the monitor from a warehouse monitor into an account monitor
C. Assign an additional warehouse to the monitor's warehouse list
D. Create a brand-new resource monitor for a different set of warehouses

*Answer: A. `MODIFY` covers quota, schedule, and actions only. Changing the monitor type, editing the warehouse list, and creating new monitors all require `ACCOUNTADMIN`.*

---

That's the full picture on Resource Monitors. Take your time with this one — the account-vs-warehouse layering (§4) and the Resource Monitor vs Budget comparison (§9) are genuinely the two areas I'd bet real exam points on.

Got questions on anything above, or ready to move to **Cost Optimize** next?
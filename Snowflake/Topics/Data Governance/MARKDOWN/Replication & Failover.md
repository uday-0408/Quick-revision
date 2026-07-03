# Snowflake Replication & Failover — SnowPro Core Notes

**Topic 1 of 6** in this series (Replication/Failover → Masking Policies → Row Access Policies → Object Tagging → Trust Center → Data Lineage). Object tags and policies show up again later — they're literally on the "what gets replicated" list below, so this chapter quietly sets up that one too.

Sources: Snowflake official docs — Introduction to replication & failover, Replicating databases and account objects, Failing over account objects, Replication of security integrations & network policies, Stage/pipe/load history replication, Multi-Location Resilience for Data Pipelines, Git repository replication, Understanding replication cost.

---

## 1. The big picture — why this feature exists

**Concept, in plain English:** Your Snowflake account normally lives in one region on one cloud. If that region has an outage, your account is unreachable — not because your data is gone, but because the *building* lost power. Replication makes a copy of your account's objects in a second Snowflake account (a different region, or even a different cloud entirely). Failover is the ability to flip that copy from "read-only mirror" into "the real thing" in minutes.

**Why Snowflake built it this way:** regulated industries (banking, healthcare, insurance) are often contractually or legally required to prove they can survive a full region loss. Rather than making you build your own copy pipeline, Snowflake manages the copying, the consistency checks, and the promotion mechanics for you.

**Two terms worth separating:**
- **Business continuity** — your operations keep running through a disruption.
- **Disaster recovery** — how you get back to normal after one.

⭐ Exam importance: ⭐⭐⭐⭐⭐ | 💡 Real-world importance: ⭐⭐⭐⭐⭐ — this is a real architecture pattern, not exam trivia.

---

## 2. Core vocabulary (learn these before anything else)

| Term | Meaning |
|---|---|
| Source / target account | Where the primary currently is / where the copy lands |
| Primary object | The current read-write original |
| Secondary object | The read-only copy |
| Refresh | Copying the latest changes, primary → secondary (one-way, asynchronous) |
| Replication group | A named bundle of objects replicated as a unit — **can never be promoted** |
| Failover group | Same idea, but **can** be promoted to primary |
| Promote / failover | Flipping a secondary to become the new primary |
| Failback | Promoting the *original* primary back once it recovers |

**⚠️ Gotcha:** a replication group and a failover group are different object types. You cannot convert one into the other — `ALTER ... SET ENABLE FAILOVER` doesn't exist. You drop and recreate.

---

## 3. How it works end to end

1. An **organization administrator** enables replication for each account (one-time, via `SYSTEM$GLOBAL_ACCOUNT_SET_PARAMETER(..., 'ENABLE_ACCOUNT_DATABASE_REPLICATION', 'true')`).
2. Source account: `CREATE FAILOVER GROUP` — list what's included and which target accounts are allowed.
3. Target account: `CREATE FAILOVER GROUP ... AS REPLICA OF <source>` — this is the secondary.
4. Snowflake runs an automatic **initial refresh**.
5. Ongoing refreshes happen on a **schedule** (or manually) — always one-way, always asynchronous. This is not synchronous replication; there is lag.
6. Disaster: on the target, `ALTER FAILOVER GROUP ... PRIMARY`. Roles flip instantly.
7. Once the old primary recovers: refresh it first (to pull in anything that happened while it was secondary), *then* promote it back — that's failback.

*(See the diagram above — same shapes, same idea, this is just the text version.)*

---

## 4. Editions: who can use what (the single most-tested table on this topic)

| Capability | Standard | Enterprise | Business Critical+ |
|---|:---:|:---:|:---:|
| Replicate a database | ✅ | ✅ | ✅ |
| Replicate a share | ✅ | ✅ | ✅ |
| Replication Group (databases/shares only) | ✅ | ✅ | ✅ |
| Replicate other account objects (users, roles, warehouses, integrations, network policies, parameters, resource monitors...) | ❌ | ❌ | ✅ |
| **Failover Group (any contents, even one small DB)** | ❌ | ❌ | ✅ |
| Tri-Secret Secure on replicated data | ❌ | ❌ | ✅ |
| Dataset replication / Cortex Search Service replication | ❌ | ❌ | ✅ |

**🎯 THE #1 exam trap on this whole topic:** edition requirement attaches to the **group type**, not the contents. A Failover Group containing nothing but one tiny database *still* requires Business Critical Edition or higher. A Replication Group with the same single database works fine on Standard. The word "Failover" is the entire tell — if you see it, the answer involves Business Critical.

**⚠️ Other edition gotchas:**
- Upgrading an account to Business Critical can take **up to 12 hours** before failover capability actually activates — it's not instant.
- Business Critical accounts replicating a DB/share-only group to a *lower*-edition account get **blocked by default** (protects sensitive data from landing somewhere less secure) — override with `IGNORE EDITION CHECK` if you really mean it. This override does **not** apply to Failover Groups — those can only ever go to Business Critical+ targets, full stop.
- Region support: all AWS/GCP/Azure regions support replication, and you can replicate freely within a **region group**. Crossing region groups (e.g., commercial → government) needs a Snowflake Support request.

---

## 5. What actually gets replicated (and what quietly doesn't)

### Account-level objects
Everything below **requires Business Critical+** except Databases and Shares (all editions):

Databases, external volumes, integrations (security/API/notification/storage/external-access), listings, network policies, account-level parameters, profiles, resource monitors, roles, shares, users, warehouses, workspaces, datasets (ML), Cortex Search Services.

**⚠️ Roles/Users are read-only on the secondary** — you cannot create or modify a replicated user/role in the target account. They must be created in the source and flow down through replication.

### Inside a replicated database — what's IN vs OUT

| ✅ Replicated | ❌ NOT replicated |
|---|---|
| Permanent & transient tables, error tables | **Temporary tables** |
| Dynamic tables | **External tables** |
| Apache Iceberg tables (Snowflake-managed only — needs external volume replication too) | **Hybrid tables** |
| Views, materialized views, secure views, semantic views | **Event tables** |
| Sequences, UDTs, file formats, UDFs, stored procedures | **Temporary stages** |
| Streams, tasks, DMFs | **Online feature tables** |
| Stages & pipes — **but only via replication/failover groups, never via legacy single-database replication** | Files sitting on external stages (only the stage *pointer*/credentials replicate) |
| Masking policies, row access policies, tag-based masking, all other policy types, tags, alerts, secrets, network rules | |
| Git repository clones (≤5GB), Snowflake Notebooks, dbt projects, Cortex Knowledge Extensions | |

**⚠️ Gotcha — cross-database references:** if a view, semantic view, or foreign key references objects in *another* database, **both** databases must be replicated or the reference breaks (foreign keys referencing another DB's primary/unique key are excluded from replication entirely).

**⚠️ Gotcha — the REPLICATE/FAILOVER privileges are never replicated.** Even with `ROLES` fully included in your group, you must manually `GRANT REPLICATE` and `GRANT FAILOVER` on the group in **both** the source and target accounts, separately, every time.

**Current documented limitations:**
- Databases created **from a share** cannot be replicated.
- A refresh fails if the primary DB has a stream on an unsupported source object, or if that source object was dropped.
- Append-only streams aren't supported on replicated source objects.
- Task graphs don't replicate correctly if the graph is owned by a role different from the role performing replication.
- **An object (database, share, or account-object type) can only belong to ONE replication/failover group at a time.**

---

## 6. Roles and grants — the nuance-heavy part

- Including `ROLES` replicates **all roles** (no selective picking) plus role hierarchies.
- A grant on an object only replicates to the target if **all three** are true: (1) the privilege was granted by the object's owner, or by a role granted it `WITH GRANT OPTION` by the owner; (2) both the grantor and grantee role exist in the target account; (3) the object type itself is included in the group.
- **Future grants** replicate if `ROLES` is included — even for object types not yet supported for replication (e.g., future grants on external tables replicate, so when you eventually create one in the target, permissions are already correct).
- If `ROLES` is **not** replicated and new objects appear in the target during a refresh, ownership defaults to `GLOBALORGADMIN`. If `ROLES` **is** replicated, ownership matches the source's role.
- The user who executes `ALTER FAILOVER GROUP ... REFRESH` must exist in the **source** account and hold a role with the `REPLICATE` privilege — otherwise the refresh fails outright. Snowflake checks this as a safety measure.

---

## 7. Setting it up — the SQL shape to recognize

```sql
-- one-time, by an ORGADMIN, for each account
SELECT SYSTEM$GLOBAL_ACCOUNT_SET_PARAMETER(
  '<org>.<account>', 'ENABLE_ACCOUNT_DATABASE_REPLICATION', 'true');

-- SOURCE account
CREATE FAILOVER GROUP myfg
  OBJECT_TYPES = USERS, ROLES, WAREHOUSES, RESOURCE MONITORS, DATABASES
  ALLOWED_DATABASES = db1, db2
  ALLOWED_ACCOUNTS = myorg.myaccount2, myorg.myaccount3
  REPLICATION_SCHEDULE = '10 MINUTE';

-- TARGET account
CREATE FAILOVER GROUP myfg AS REPLICA OF myorg.myaccount1.myfg;

-- grant REPLICATE / FAILOVER separately, in BOTH accounts
GRANT REPLICATE ON FAILOVER GROUP myfg TO ROLE my_replication_role;
GRANT FAILOVER  ON FAILOVER GROUP myfg TO ROLE my_failover_role;

-- manual refresh (run from target)
ALTER FAILOVER GROUP myfg REFRESH;

-- promote target to primary (run from target, during a real failover)
ALTER FAILOVER GROUP myfg PRIMARY;
```

**Migrating from legacy single-database replication:** if a database was set up with the older `ALTER DATABASE ... ENABLE REPLICATION` approach, you must run `SYSTEM$DISABLE_DATABASE_REPLICATION('mydb')` before it can join a group. Good news: Snowflake does **not** re-copy already-replicated data — only the delta since the last refresh moves.

**Schema-level control (`REPLICABLE_WITH_FAILOVER_GROUPS`):** lets you exclude specific schemas from an otherwise-replicated database, or include specific schemas even when the parent database is excluded. Default is `YES` (inherited from parent unless overridden). Needs the account-level `REPLICATE` privilege plus `USAGE` on the db/schema.

```sql
ALTER DATABASE db1 SET REPLICABLE_WITH_FAILOVER_GROUPS = 'NO';
ALTER SCHEMA  db1.sch1 SET REPLICABLE_WITH_FAILOVER_GROUPS = 'YES'; -- carve-out
```

---

## 8. Refresh mechanics — the heartbeat

- Best practice: set `REPLICATION_SCHEDULE` (interval like `'10 MINUTE'` or a cron expression) rather than relying on manual refreshes.
- Timing rule: the next refresh is scheduled relative to when the **previous one started**, not when it finished — *unless* it's still running, in which case the next one waits until the current one completes. **Only one refresh runs at a time, ever.**
- Scheduled refreshes execute using the role with `OWNERSHIP` on the group.
- Snowflake auto-verifies every refresh (file hash, row count, byte count comparisons). You can additionally hand-verify with `HASH_AGG()` compared against an `AT`/`BEFORE` Time Travel snapshot on the primary.
- You **cannot promote a group while it's mid-refresh** — `ALTER FAILOVER GROUP ... PRIMARY` errors out. Fix: `SUSPEND` (waits for the current refresh to finish) or `SUSPEND IMMEDIATE` (cancels now — riskier, see below).

**⚠️ Gotcha:** canceling a refresh that's in the `SECONDARY_DOWNLOADING_METADATA` or `SECONDARY_DOWNLOADING_DATA` phase can leave the target in an inconsistent state. Check the phase first via the `REPLICATION_GROUP_REFRESH_PROGRESS` table function before force-cancelling.

### Optimized Refresh vs. Replication Classic (newer, Preview, Business Critical+)

| | Replication Classic | Optimized Refresh |
|---|---|---|
| Refresh cost/time driven by | Total object count in the account | Rate of change since last refresh |
| Opt-in | Default | Set `OPTIMIZED_REFRESH = TRUE` on the **primary** group only |
| Requirement | none specific | `REPLICATION_SCHEDULE` ≤ 6 hours (older gap → falls back to a full refresh) |
| First refresh after enabling | n/a | Slow "bootstrapping" refresh, billed like Classic |
| Pricing | Data transfer + compute (delta calc) | 5 credits/TB replicated + 0.2 credit per 10,000 changed objects (25M/month free) |
| Reversible? | — | Yes — `ALTER FAILOVER GROUP ... UNSET OPTIMIZED_REFRESH` |
| Mixed use | You can have some groups on Classic and others Optimized in the same account | |

Check current mode: `SHOW FAILOVER GROUPS` → `is_optimized_refresh_enabled` column.

---

## 9. Failover & failback mechanics

- **Bulk failover** (Snowsight): promote every relevant failover group *and* connection together — Snowflake's recommended, most-consistent approach. You choose whether to wait for in-flight refreshes to finish (default, safest) or force through immediately (riskier).
- **Single-group failover**: promote just one group — used to retry a group that failed during a bulk failover.
- SQL: `ALTER FAILOVER GROUP <name> PRIMARY`, run on the target account, by a role holding the `FAILOVER` privilege.
- **⚠️ Big gotcha:** the instant a failover completes, **scheduled refresh auto-suspends on every secondary group** involved. Nothing resumes automatically — you must run `ALTER FAILOVER GROUP ... RESUME` on each target account that now holds a secondary.
- **Snowpipe Streaming:** tables fed by Snowpipe Streaming *do* replicate, but after failover you must manually reopen the ingest channels (`openChannel`, fetch `getLatestCommittedOffsetToken`, re-insert any missing rows). The Kafka connector needs a different fix: point the connector at the new source account, pull channel offsets via `SHOW CHANNELS`, reset the Kafka topic offsets per partition, restart the connector. Note: classic Snowpipe + Kafka connector is **not** supported for replication at all — only Snowpipe *Streaming* + Kafka is.

---

## 10. Client redirect (Connections) — hiding the flip from your apps

**Why it exists:** without this, every client application, BI tool, and script would need its connection string manually repointed after every failover. A `CONNECTION` object gives everyone one stable URL that always resolves to whichever account is currently primary.

```sql
-- target account
CREATE CONNECTION global AS REPLICA OF myorg.myaccount1.global;
-- promote on failover
ALTER CONNECTION global PRIMARY;
-- source account, enabling specific targets
ALTER CONNECTION global ENABLE FAILOVER TO ACCOUNTS myorg.myaccount2;
```

**⚠️ SAML2 gotcha:** the connection-URL approach to SAML SSO only works on the **current primary** connection — SSO via the shared URL does not work on the secondary until it's promoted. If you need SSO live on *both* accounts simultaneously, you must configure fully independent SAML integrations on each — not the shared-connection-URL method.

**SCIM:** only one account (the primary) receives SCIM push updates from the identity provider at a time; you can redesignate later.

**OAuth:** Snowflake OAuth users don't need to re-authenticate after failover. External OAuth users *might* need to — only if the authorization server isn't configured to issue refresh tokens, so make sure it is.

**Network policies:** replicating a policy also requires replicating whatever it depends on — network rules live inside a database (so include `DATABASES`), a policy assigned at account level needs `ACCOUNT PARAMETERS`, one assigned to a user needs `USERS`, one attached to a security integration needs `INTEGRATIONS` with security integrations allowed.

---

## 11. Object-by-object edge cases (exam-favorite territory)

### Stages
| Type | What replicates |
|---|---|
| Table stage | Empty shell only — files never replicate |
| User stage | Empty shell only (needs Business Critical, tied to user replication) — files never replicate |
| Named internal stage | Replicates with the DB; **files only replicate if a directory table is enabled** on the stage |
| Named external stage | The stage object/credentials replicate; **files on external storage never replicate** — both accounts point at the same cloud URL |

**⚠️ Gotchas:**
- Stage/pipe replication **only works through replication/failover groups** — never through legacy single-database `ALTER DATABASE` replication.
- A directory table containing any file **over 5GB** makes the refresh fail. Fix: disable the directory table, move the large file elsewhere, re-enable — and do this *before* adding the database to a group. You can't disable a directory table once it's been replicated.
- Storage-integration-based external stages need `STORAGE INTEGRATIONS` explicitly added to `ALLOWED_INTEGRATION_TYPES`, **plus** a manual post-replication step to trust the replica's new IAM identity in your cloud account.

### Pipes
- Secondary pipes sit in `READ_ONLY` state → flip to `FAILING_OVER` during promotion → land on `RUNNING`, then catch up on anything queued since the last refresh.
- **Auto-ingest pipes** need their cloud notification wiring (SNS/SQS, Pub/Sub, Event Grid) manually configured on the target account **before** failover — this does not come along automatically. This is why "notification integration replication is not supported" shows up in the pipes docs: it refers specifically to this inbound auto-ingest wiring, not the general EMAIL/WEBHOOK/outbound-queue notification integrations, which *do* replicate fine.
- **REST-endpoint pipes** (Snowpipe REST API) replicate their load history automatically with zero extra config — just call the REST API against the newly-promoted account.
- Pipe copy history only replicates if the pipe is in the **same** group as its target table.

### Warehouses & Resource Monitors
- Warehouses always replicate in a **suspended** state — the primary's running/suspended status is never carried over.
- Resource monitor **quota reset schedules** stay in sync with the primary. But email notification settings only partly replicate: non-admin user notifications replicate if `USERS` and `RESOURCE MONITORS` are both included; **administrator notification settings never replicate** — an admin must manually enable email notifications separately in each account.

### Users & Roles — the scary one
If you replicate `USERS`/`ROLES` for the **first time** and the target already has users with matching names, the refresh **fails** and gives you two choices: `REFRESH FORCE` (drops the conflicting target users and replaces them) or `SYSTEM$LINK_ACCOUNT_OBJECTS_BY_NAME` (links same-named users so they survive, drops the rest).

**🎯 If there are NO matching users at all in the target, the first refresh drops every existing user in the target account.** Depending on what's included, that can silently destroy worksheets, query history, privilege grants to users, and even privilege grants to shares. Always link or pre-create matching users/roles *before* the first replication, not after.

### Git repositories
- No separate enable step — replicates automatically when the containing database/schema is in the group, secrets included.
- Secondary is read-only: you can read code, but cannot commit/fetch/push to `origin` until it's promoted.
- **Size cap: 5GB per repo clone** — larger repos are not currently supported.

---

## 12. Multi-Location Resilience for Data Pipelines (MLSI / MQNI)

📎 Flagging this as **lower Core-exam priority but genuinely useful real-world knowledge** — it's a newer, more specialized Business Critical+ feature layered on top of everything above, specifically for Snowpipe/`COPY INTO` pipelines surviving a cross-cloud or cross-region outage.

- **MLSI** (Multi-Location Storage Integration): one storage integration object holding *multiple* cloud storage locations (can span providers), with one marked `ACTIVE`.
- **MQNI** (Multi-Queue Notification Integration): same idea for cloud message queues, for Snowpipe auto-ingest specifically.
- Only supports Snowpipe (auto-ingest) and `COPY INTO` — **not** Openflow, **not** Snowpipe Streaming.
- Shared responsibility: Snowflake replicates the target table + load history together in one snapshot (so no duplicates); **you** are responsible for routing files to the secondary storage location.
- **Dual-write (recommended):** your app writes to both storage locations simultaneously — failover and failback both reconcile automatically, no manual work.
- **Single-write:** you only redirect writes to secondary storage *after* an outage starts — any files "in flight" at the exact moment of the outage are temporarily missing and need manual reconciliation against `COPY_HISTORY` afterward.
- **⚠️ Critical failback warning:** refreshing to fail back overwrites the primary database with the secondary's data. If you used single-write routing and never reconciled orphaned files first, they're gone permanently.

---

## 13. Cost model

- Charges land on the **target** account (both data transfer and compute).
- **Data transfer:** cross-region/cross-cloud egress, priced by the **source** account's region.
- **Compute:** billed under service type `REPLICATION` — covers delta calculation plus the actual data copy.
- Target also pays normal storage for the secondary database, plus serverless costs for background maintenance (materialized views, search optimization) on the secondary.
- **You're billed even if the refresh fails** — but any data already copied is reusable by a follow-up refresh within 14 days, so it's not fully wasted.
- Cost scales with (a) how much table data changes and (b) how often you refresh — your two levers for cost control.
- Monitor via `REPLICATION_GROUP_USAGE_HISTORY` (table function = 14 days of history; Account Usage view = 365 days).
- **Optimized Refresh pricing is a separate model:** 5 credits/TB of replicated data + 0.2 credits per 10,000 changed objects (first 25,000,000/month free) — storage and egress costs are unaffected either way.

---

## 14. Head-to-head comparison tables

**Replication Group vs. Failover Group**

| | Replication Group | Failover Group |
|---|---|---|
| Can be promoted? | ❌ Never | ✅ Yes |
| Secondary access | Always read-only | Read-only until promoted |
| Minimum edition | Standard (DB/share only) | **Always** Business Critical+ |
| Convert between types? | ❌ Drop & recreate only | ❌ Drop & recreate only |

**Failover vs. Failback**

| | Failover | Failback |
|---|---|---|
| Direction | Secondary → new primary | Original primary promoted again |
| Pre-condition | No refresh in progress | Refresh first to catch up, *then* promote |
| Risk if skipped | Promotion blocked / inconsistent data | Data loss (especially single-write MLSI) |

**Legacy Database Replication vs. Group-Based Replication**

| | `ALTER DATABASE ... ENABLE REPLICATION` | Replication/Failover Groups |
|---|---|---|
| Scope | One database | Databases + shares + account objects together |
| Stages/pipes replicate? | ❌ No | ✅ Yes |
| Can fail over? | ❌ No such capability | ✅ Yes, if it's a Failover Group |

---

## 15. Common misconceptions

- **"A Replication Group can eventually be failed over if I need it to."** No — it structurally cannot. You'd have to drop it and build a Failover Group instead.
- **"My database replicates, so my stage files come along too."** Only true for internal stages with a directory table enabled, and only via group-based replication — external stage files never replicate at all.
- **"REPLICATE/FAILOVER privileges follow role replication automatically."** They don't — grant them manually, in both accounts, every time.
- **"A secondary database is basically live/real-time."** It's only as fresh as your last refresh — asynchronous, not synchronous. Your RPO equals your refresh interval.
- **"After failover, replication just keeps going on its own."** No — every secondary's schedule auto-suspends on failover; you must resume it manually.
- **"I can put one database in two failover groups for extra safety."** Not allowed — one group membership per object, period.

---

## 16. One-page revision sheet

- Database & share replication: **all editions**. Everything else: **Business Critical+**.
- Failover Group = **always** Business Critical+, no matter how small.
- Replication is **one-way and asynchronous** — never assume zero lag.
- `REPLICATE` / `FAILOVER` privileges are **never** auto-replicated.
- Warehouses always land **suspended** on the secondary.
- Internal stage files replicate **only** with a directory table enabled; external stage files **never** replicate.
- Auto-ingest pipe cloud notifications must be **manually** configured on the secondary before failover.
- First-time USERS/ROLES replication can **drop existing target users** if names don't match — link or pre-create first.
- Can't promote mid-refresh — `SUSPEND` first.
- Failover auto-suspends every secondary's schedule — **manual `RESUME`** required.
- One object = one group, always.
- Cost is billed to the **target** account, driven by data change volume × refresh frequency.

---

## 17. Mini mock test

**Conceptual (5)**

1. What's the core difference between a replication group and a failover group?
   A) Replication groups include databases; failover groups can't
   B) Failover groups can be promoted to primary; replication groups can't
   C) Replication groups need Business Critical; failover groups don't
   D) They're interchangeable terms
   **Answer: B**

2. Minimum edition to create a Failover Group containing just one small database?
   A) Standard  B) Enterprise  C) Business Critical  D) Depends on database size
   **Answer: C** — the requirement is on the group type, not the contents.

3. Which privileges are *not* automatically replicated even when ROLES is included?
   A) SELECT  B) OWNERSHIP  C) REPLICATE and FAILOVER  D) USAGE
   **Answer: C**

4. What state are pipes in on a secondary database before failover?
   A) RUNNING  B) PAUSED  C) READ_ONLY  D) SUSPENDED
   **Answer: C** (→ `FAILING_OVER` during promotion → `RUNNING` after)

5. What state do replicated warehouses land in on the target account?
   A) Same as the primary's current state  B) Always suspended  C) Always running  D) Resized to XSMALL
   **Answer: B**

**Scenario-based (3)**

1. A team builds a Replication Group with two databases on Standard Edition, planning to fail over during an outage. It fails. Why?
   **Answer:** A Replication Group has no promote capability at all — they needed a Failover Group, which requires Business Critical Edition regardless of contents.

2. First-time replication of USERS/ROLES to a target that already has five same-named users. What happens on that first refresh by default?
   **Answer:** It fails, requiring either `REFRESH FORCE` (drops the existing target users) or `SYSTEM$LINK_ACCOUNT_OBJECTS_BY_NAME` (links them by name so they survive).

3. An auto-ingest Snowpipe pipe is replicated to a DR account, but the target region's SNS/SQS/Pub-Sub wiring was never configured. Failover happens for real. What breaks?
   **Answer:** The secondary pipe never receives "new file" notifications because that cloud-native notification wiring must be manually set up in the target *before* failover — it isn't carried over by replication.

**True/False (2)**

1. A database can belong to two different failover groups at once for redundancy. **False** — one group membership per object.
2. After failover, scheduled refreshes keep running automatically on the newly-demoted secondary. **False** — all secondary schedules auto-suspend on failover; `RESUME` is manual.

**Trick question (1)**

Your `REPLICATION_SCHEDULE` is `'10 MINUTE'`, but one refresh takes 25 minutes because of a large data change. When does the *next* refresh run?

**Answer:** Immediately after that 25-minute refresh finishes — **not** stacked, and not skipped to catch the next 10-minute mark. Snowflake only ever runs one refresh at a time per group; if the next scheduled time arrives while one is still running, it waits.

---

*End of Chapter 1. Ready for Chapter 2 (Masking Policies) whenever you are — or tell me which section here you want to go deeper on first.*
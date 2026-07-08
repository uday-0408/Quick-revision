# Chapter 1 — Row Access Policies (Row-Level Security)

**Series:** Snowflake Governance & Security — SnowPro Core Notes
**Source docs:**
- Understanding row access policies — `docs.snowflake.com/en/user-guide/security-row-intro`
- Use row access policies — `docs.snowflake.com/en/user-guide/security-row-using`
**Exam relevance:** ⭐⭐⭐⭐⭐ (Data Governance domain — consistently one of the most tested features on SnowPro Core)

> **Where this fits in the bigger picture:** Row access policies (RAP) are one half of Snowflake's fine-grained access control system. RAP controls *which rows* a query returns. Masking policies (your next chapter) control *how a column's value looks*. Object tagging (later chapter) can attach classification metadata that governance teams use to decide where policies go. Keep this chapter fresh — almost every exam question that mentions "row access policy" is really testing whether you can tell it apart from a masking policy or from ordinary RBAC.

## Table of Contents
1. [Core Concept](#1-core-concept)
2. [Query Runtime Mechanics](#2-query-runtime-mechanics)
3. [Nested Policies (Table → View → View)](#3-nested-policies-table--view--view)
4. [Two Ways to Write a Policy](#4-two-ways-to-write-a-policy)
5. [Context Functions Deep Dive](#5-context-functions-deep-dive-the-1-gotcha-zone)
6. [Performance](#6-performance)
7. [Benefits](#7-benefits)
8. [Limitations (Hard Rules)](#8-limitations-hard-rules)
9. [Considerations (Soft Gotchas)](#9-considerations-soft-gotchas)
10. [Interaction Matrix with Other Features](#10-interaction-matrix-with-other-snowflake-features)
11. [Privileges & Management Models](#11-privileges--management-models)
12. [DDL Command Reference](#12-ddl-command-reference)
13. [Monitoring & Auditing](#13-monitoring--auditing)
14. [Troubleshooting Reference](#14-troubleshooting-reference)
15. [End-to-End Walkthrough](#15-end-to-end-walkthrough-centralized-model)
16. [Memoizable Functions in Practice](#16-memoizable-functions-in-practice)
17. [Advanced Pattern: Protecting the Mapping Table](#17-advanced-pattern-protecting-the-mapping-table-itself)
18. [RAP vs Masking Policy](#18-compare-row-access-policy-vs-masking-policy)
19. [Common Misconceptions](#19-common-misconceptions)
20. [Exam Traps](#20-snowpro-core-exam-traps)
21. [One-Page Revision Sheet](#21-one-page-revision-sheet)
22. [Mini Mock Test](#22-mini-mock-test)

---

## 1. Core Concept

### What it is
A **row access policy** is a **schema-level object** that decides, row by row, whether a given row in a table or view is allowed to appear in a query result. It doesn't touch the *columns* — every column is still visible for a row that passes — it only decides whether the *row itself* gets returned at all.

### Why this feature exists
Before row access policies, "show sales managers only their own region" meant one of two bad options: build a separate view per region (an operational nightmare that multiplies with every new region/role), or bake `WHERE region = ...` filters into every downstream query and BI tool (easy to forget, easy to bypass). A row access policy centralizes that logic **once**, on the object, so it's enforced no matter what tool, role, or query shape hits the table.

### The two facts to memorize cold
| Fact | Detail |
|---|---|
| **Edition requirement** | Enterprise Edition or higher. Standard Edition cannot use this feature at all. |
| **Object type** | Schema-level object — it lives inside a schema, needs a database/schema context (or a fully qualified name) to create, and is governed by schema-level privileges like `CREATE ROW ACCESS POLICY ON SCHEMA`. |

### What statements RAP affects — and the one it doesn't
A row access policy filters rows for:
- `SELECT`
- The rows *targeted* by `UPDATE`, `DELETE`, and `MERGE`

It does **not** filter or block:
- `INSERT` — RAP never stops a new row from being written.
- Updates/deletes of a row the policy already lets you see — once a row passes the policy, it can be fully updated or deleted like any other row. RAP is a **visibility filter**, not a data-modification guard.

> 🎯 **Real-world translation:** RAP is not a substitute for column-level `UPDATE`/`DELETE` privileges or triggers-style enforcement. If you need to stop inserts of out-of-region data, that's a different control (e.g., a check constraint pattern, a stream/task validation, or restricting the `INSERT` privilege itself).

### The counter-intuitive fact almost everyone misses
The policy-driven model **overrides the normal "owner sees everything" assumption**. Normally, the role holding `OWNERSHIP` on a table has unrestricted access to its data. A row access policy still applies to that owner. Ownership of the *table* gives you control over the table object — it does not exempt you from a row access policy attached to it. The only way to see all rows is for the **policy's own logic** to grant that (e.g., an explicit `'admin_role' = CURRENT_ROLE()` branch).

### Internal workflow — the big picture
```
                 CREATE / ALTER TABLE|VIEW
                 ... ADD ROW ACCESS POLICY rap ON (col)
                              │
                              ▼
              Policy is now bound to the object + column(s)
              (this binding is metadata — no data is rewritten)
                              │
                              ▼
                 Every future SELECT / UPDATE / DELETE / MERGE
                 against this object triggers policy evaluation
                              │
                              ▼
        Snowflake evaluates the BOOLEAN policy expression using the
        PRIVILEGES OF THE POLICY OWNER — not the querying user
                              │
                              ▼
              Row-by-row: TRUE → row is visible, FALSE → row is hidden
```

### Why "policy owner's rights" instead of "querying user's rights"?
This is the single most important architectural fact in the whole topic, and it's the reason mapping-table-based RAP even works. If the policy ran with the *querying user's* privileges, every single user would need direct `SELECT` on the mapping table, defeating the whole purpose of centralizing governance. Instead:
- The **policy owner role** needs `SELECT` on the mapping table.
- The **querying user's role** only needs `SELECT` on the protected table itself and does **not** need any privilege on the mapping table.

This is what lets a governance team hide a sensitive entitlement/mapping table from end users while still using it to drive row filtering.

### Real-world example
A hospital's `claims` table has a row access policy so that `regional_case_manager` roles only see claims from their assigned hospital network, defined via a mapping table of `case_manager → network_id` that only the governance team can query directly. Analysts querying `claims` never need — and never get — access to that mapping table.

---

## 2. Query Runtime Mechanics

### The 4-step evaluation process (documented, exam-favorite)
1. Snowflake checks whether the target object has a row access policy attached. If yes, **every row** in that object is subject to the policy — there's no partial application.
2. Snowflake builds a **dynamic secure view** — Snowflake's internal term for a secure inline view generated on the fly — wrapping the object.
3. The column(s) named in the `ADD ROW ACCESS POLICY ... ON (...)` clause are bound to the policy's signature parameter(s), and the boolean expression is evaluated.
4. The query output contains only the rows where the expression evaluated to `TRUE`.

### Why "dynamic secure view" matters for the exam
Because RAP always compiles to a secure-view-style execution plan, you lose the usual query-planner shortcuts that come from stored table statistics — more on this in the Performance section. This single design fact explains three unrelated-looking exam topics at once: why `EXPLAIN` shows `DynamicSecureView`, why `COUNT(*)` gets slow, and why the `CHANGES` clause isn't supported on a RAP-protected view (all three are downstream consequences of the same mechanism).

### Reading it in `EXPLAIN`
```
EXPLAIN SELECT * FROM claims;

step | operation          | objects
-----+--------------------+-------------------------------
1    | DynamicSecureView  | "CLAIMS (+ RowAccessPolicy)"
```
- If a query causes the policy to be invoked twice (e.g., the table is referenced twice, such as in a correlated subquery comparing a row to an aggregate of the same table), you'll see **two** `DynamicSecureView` rows.
- `EXPLAIN` and **Query History never reveal the policy's name, signature, or expression**, nor which objects the policy body itself touches (e.g., the mapping table). This is intentional — it stops a curious analyst from reverse-engineering sensitive entitlement logic just by reading a query plan.

---

## 3. Nested Policies (Table → View → View)

Snowflake supports **stacking** row access policies: a policy on the base table *and* a separate policy on a view built over that table *and* another policy on a view built over that view.

### Evaluation order — memorize this
```
Table  (RAP evaluated 1st)
   │
   ▼
View 1 (RAP evaluated 2nd)
   │
   ▼
View 2 (RAP evaluated 3rd)
   │
   ▼
... continues left → right for View n
   │
   ▼
Final result set
```
- The table-level policy **always runs first**.
- View-level policies then run in the order the views are layered, left to right.
- A row must pass **every** applicable policy in the chain to appear in the final result — the policies effectively `AND` together (a row hidden at any layer stays hidden downstream).

### Real-world example
A `finance` team builds `gl_transactions` (RAP: business-unit filter) → `gl_transactions_regional_v` (view, RAP: region filter for regional controllers) → `gl_transactions_regional_quarterly_v` (view, RAP: fiscal-quarter access window for auditors). An auditor querying the outermost view is filtered by all three policies in sequence.

---

## 4. Two Ways to Write a Policy

### Pattern A — Simple role check (cheapest, fastest)
```sql
CREATE OR REPLACE ROW ACCESS POLICY governance.policies.rap_hr_only
AS (employee_id VARCHAR) RETURNS BOOLEAN ->
  CURRENT_ROLE() = 'HR_ADMIN'
;
```
No table lookups, no joins — just a comparison against a context function. Documented as having **negligible performance cost**. You can extend this with `CASE` for multiple branches without materially hurting performance, because `CASE` is still just expression logic, not a table scan.

### Pattern B — Mapping-table lookup (flexible, costs more)
```sql
CREATE OR REPLACE ROW ACCESS POLICY governance.policies.rap_region
AS (sales_region VARCHAR) RETURNS BOOLEAN ->
  'SALES_EXEC' = CURRENT_ROLE()
  OR EXISTS (
      SELECT 1 FROM governance.tables.manager_region_map
      WHERE manager_role = CURRENT_ROLE()
      AND region = sales_region
  )
;
```
This is the pattern used for real entitlement systems: an `EXISTS` subquery against a centralized mapping table (manager → region, tenant → account, department → cost-center, etc.). It's more flexible — entitlements can change by editing table rows instead of rewriting the policy — but it costs more at query time because Snowflake has to evaluate the subquery per row/partition.

> **Documented best practice:** if the policy references a mapping table, store that mapping table **in the same database** as the protected object. This matters especially when the policy calls `IS_DATABASE_ROLE_IN_SESSION()`, since database roles are database-scoped.

---

## 5. Context Functions Deep Dive (the #1 gotcha zone)

There are two *completely different* "role" concepts in play, and the exam loves conflating them to test if you actually understand the architecture:

| Concept | What it controls | Fixed or author's choice? |
|---|---|---|
| **Execution privileges** for the policy body (e.g., who can actually read the mapping table during evaluation) | Whether the `EXISTS` subquery against the mapping table can even run | **Always fixed** to the policy **owner's** role — the "owner's rights" model, not configurable per query |
| **Context function used inside the expression** (`CURRENT_ROLE()`, `CURRENT_USER()`, `IS_ROLE_IN_SESSION()`, `IS_DATABASE_ROLE_IN_SESSION()`, `INVOKER_ROLE()`) | What gets compared to decide TRUE/FALSE for a given row | **The policy author's design choice** — you pick which function to call |

### The primary-vs-secondary role trap
`CURRENT_ROLE()` returns only the **one active primary role** of the session. If a user has `SALES_MANAGER` granted and active as a **secondary** role (common when someone has multiple roles activated at once), a policy written as `CURRENT_ROLE() = 'SALES_MANAGER'` will **evaluate FALSE even though the user genuinely has that role active**. This is exactly the kind of "user swears they have access but gets zero rows" ticket that shows up in real production incidents.

**Fix:** Snowflake's documented recommendation is to use:
- `IS_ROLE_IN_SESSION('SALES_MANAGER')` for **account roles** — checks the full role hierarchy and both primary/secondary activation, not just the exact primary role.
- `IS_DATABASE_ROLE_IN_SESSION('SALES_MANAGER')` for **database roles**.

### `INVOKER_ROLE()` vs `CURRENT_ROLE()` on views
When a policy sits on a view, the expression can reference either:
- `INVOKER_ROLE()` — effectively the view owner's context in certain secure-view invocation patterns, or
- `CURRENT_ROLE()` — the actual querying session's active role.

Which one is "correct" depends entirely on whether you want the filtering decision to be based on *who's really asking* vs. *the view's own identity context*. There's no universal right answer — this is a design decision the policy author makes based on the use case.

---

## 6. Performance

Performance guidance is high-yield for scenario questions ("a RAP-protected table is slow — what's the most likely fix?").

| Guideline | Why |
|---|---|
| **Limit the number of policy arguments (columns)** | Snowflake must scan every column bound in the policy signature, even if that column isn't in the `SELECT` list of the user's query. More bound columns = more forced scanning. |
| **Prefer simple expressions (`CASE`, direct comparisons) over mapping-table lookups** | Table lookups mean subqueries/joins; simple boolean logic doesn't. |
| **Replace a mapping-table subquery with a memoizable function** | A memoizable UDF is evaluated once and its result reused for repeated calls with the same input during the query, instead of re-running the lookup per row. See Section 16 for a full example. |
| **Cluster the table by the column(s) used in the policy** | For large tables, clustering on the filtered attribute reduces the partitions that must be scanned. |
| **Search Optimization Service** | Can be enabled to improve performance on tables that carry a masking policy or row access policy. |
| **Don't benchmark with `COUNT(*)`** | Without RAP, `SELECT COUNT(*) FROM t` is answered from stored metadata in milliseconds. With RAP attached, Snowflake must actually evaluate the policy per row to know the *permitted* count — there is no metadata shortcut. This is a large, well-documented performance difference, but the doc explicitly warns it is **not representative of typical real-world queries** — don't panic-optimize based on a `COUNT(*)` benchmark alone. |

> 🎯 **Exam trap:** "Why did `SELECT COUNT(*) FROM sales_table` suddenly get much slower after a row access policy was added?" — correct answer is about lost metadata-based optimization, **not** "the table got bigger" or "warehouse too small."

---

## 7. Benefits

- **Ease of use** — write the policy once, attach it to many tables/views across databases and schemas (many-to-one relationship: one policy, many protected objects).
- **Change management** — update the policy body (or just the mapping table's rows) without re-attaching it to every object.
- **Segregation of Duties (SoD)** — a **central governance role decides what's protected**, independent of who owns the underlying table. Ownership and protection-decision authority are deliberately split.
- **Contextual authorization** — filtering can be role-based, custom-entitlement-based (via mapping tables), or a hybrid of both.

---

## 8. Limitations (Hard Rules)

These are documented as things Snowflake **does not support**, full stop (workarounds noted where they exist):

1. The `CHANGES` clause cannot be used on a view protected by a row access policy.
2. An **external table cannot be used as the mapping table** referenced inside a policy body.
3. A row access policy **cannot be attached directly to a stream object**. (It's still enforced indirectly — see Section 10 → Streams.)
4. **Future grants** of privileges on row access policies aren't supported (i.e., you can't say "automatically grant APPLY on any row access policy created in this schema going forward"). Workaround: grant the **account-level** `APPLY ROW ACCESS POLICY` privilege to a custom role instead, which lets that role apply *any* row access policy on the account.

---

## 9. Considerations (Soft Gotchas)

These won't necessarily block you, but they will surprise you if you don't know about them:

- **Stacking policies can error.** Attaching a row access policy to a table/view that already carries another row access policy or a masking policy can cause errors depending on the specific combination — test before assuming it "just works."
- **Subqueries inside the policy body can cause errors.** Keep the number of subqueries and joins low, and keep `WHERE` logic simple.
- **Lost query optimizations.** As covered in Performance — `COUNT`, `MAX`, and similar metadata-driven shortcuts don't apply once RAP is attached, and any statistics Snowflake *does* return reflect only the **permitted** subset of rows, not the table's true statistics.
- **Native Apps + versioned schemas** — take extra care with setup scripts when a row access policy lives inside a versioned schema of a Snowflake Native App.
- **`CURRENT_DATABASE()` / `CURRENT_SCHEMA()` inside a policy body do NOT mean what you think.** ⭐ This is one of the highest-value gotchas in the entire topic. Inside a row access (or masking) policy expression, these two functions return the **database/schema that contains the protected object**, not the database/schema currently in use in the querying session. If you wrote a policy assuming it reflects the caller's session context, you wrote it wrong.

---

## 10. Interaction Matrix with Other Snowflake Features

### Masking policies
- If an object has **both** a row access policy and one or more masking policies, **the row access policy is evaluated first** (rows get filtered before any masking transformation is considered for the remaining rows).
- A single column **cannot** be referenced in both a row access policy signature and a masking policy signature at the same time — pick one per column.

### `CREATE TABLE` variants — a favorite exam cluster
| Variant | What happens to the row access policy |
|---|---|
| `CREATE TABLE ... CLONE` | Cloning a single policy object on its own isn't supported, but **cloning a schema clones every policy inside it**. A cloned table keeps the same policy attachments as its source. If the source policy lived in the *same* schema/database being cloned, the clone points to the **newly cloned copy** of the policy. If the source policy lived in a *different* schema/database (a foreign reference), the clone **keeps pointing to that original, un-cloned policy**. |
| `CREATE TABLE ... LIKE` | The new table does **not** get the row access policy — and it's empty (structure only, no data, no policy). |
| `CREATE TABLE ... AS SELECT` (CTAS) | The new table contains only the **already-filtered** rows (whatever the policy allowed at the time of the CTAS) — but the new table itself has **no row access policy** attached going forward. |

### External tables
- You can attach a RAP to an external table's `VALUE` column (at creation or via `ALTER TABLE`).
- You **cannot** attach a RAP directly to a *virtual* column of an external table — instead, build a view on top of the external table and attach the policy to the view's column.
- An external table can never be used as the **source mapping table** inside a policy (this repeats the Limitations point because it's genuinely easy to forget).
- **Cloning gotcha:** cloning a database clones the row access policy but does **not** clone the external table's underlying data/stage reference the same way — so a cloned policy can end up with a dangling reference to an external table that isn't really present/valid in the cloned database context. Workaround: move the external table's data into a dedicated schema inside the same database before cloning, and reference it in the policy by fully qualified name.

### Streams
- You cannot attach a RAP to the stream object itself.
- But when a stream reads change data from a table that **does** have a RAP, that policy is still enforced on the data the stream returns.

### Views
- RAP can be set on a base table **and** on views built on top of it (see Section 3, nested evaluation).
- The policy expression can key off `INVOKER_ROLE()` (view-owner-oriented context) or `CURRENT_ROLE()` (querying-session context) — an authoring decision, not a fixed rule.

### Materialized views
- You **can** add a RAP to a materialized view — **but only if** the underlying base table/view does **not already** have a row access policy.
- You **cannot** create a materialized view from a table that already has a RAP.
- You **cannot** add a RAP to a table that already has a materialized view built from it.
- **Workaround if you want both behaviors:** use a **dynamic table** instead — dynamic tables support having a row access policy, a masking policy, and a tag all at once.

### Dynamic tables
- Fully support a row access policy, a masking policy, and object tags simultaneously — this is explicitly the flexible alternative when materialized views won't work.

### Time Travel
- Time Travel is supported on RAP-protected tables/views.
- **Gotcha:** the row access policy's mapping table is evaluated using **current** data at query time — Time Travel does **not** roll back the mapping table. Only the underlying data you're time-traveling into is historical; the entitlement logic is always "as of now."

### Replication
- Row access policies and their table/view assignments **can** replicate via database replication and replication groups.
- Replication **fails** when either:
  1. The primary database is Enterprise Edition+ and has a policy, but one or more approved target accounts are on a **lower edition**, or
  2. A table/view in the primary database has a **dangling reference** to a row access policy that lives in a *different, not-replicated* database.
- **Fix for the dangling-reference case:** replicate the related databases together inside a single **replication group** instead of replicating just one database in isolation.
- Failover/failback actions specifically require **Business Critical Edition or higher** (a separate, higher bar than the Enterprise minimum for RAP itself).

### Data Sharing — heavily tested, know this cold
- If a provider's policy calls `CURRENT_ROLE()`, `CURRENT_USER()`, or a **secure UDF**, and that policy is on a **shared** object, those functions return **NULL** in the consumer's account. The provider has no visibility into the consumer's roles/users, so there's nothing meaningful to evaluate.
  - **Workaround 1:** use `CURRENT_ACCOUNT()` in the policy instead, since the provider *can* know which consumer account is querying.
  - **Workaround 2 (documented, preferred pattern):** write the policy against `IS_DATABASE_ROLE_IN_SESSION()` and **share the database role** itself; the consumer then grants that shared database role to one of their own account roles.
- **Provider cannot create a policy inside a reader account.**
- **Consumers cannot attach a policy to a shared table/view.** Workaround: import the shared database, then build a **local view** on top of the shared object (policies can be applied to that local view instead).
- **Consumers cannot query a shared table/view whose policy references a mapping table coming from a *different* share/provider.** The base table's own provider *can* still query it successfully — it's specifically the consumer, cross-share case that fails.
- **External functions inside a policy are blocked for shared objects** — you'll get an error either updating a policy on a shared object to call an external function, or trying to assign an external-function-calling policy to a shared object in the first place.

### Snowflake Native App Framework
- Has its own additional restrictions around sharing policy-protected data content and using "proxy views," plus a list of context functions that are blocked inside Native App setup scripts. Treat this as "know it exists, check the dedicated Native App docs for the specifics" for Core-level exam purposes.

### Streamlit in Snowflake
- Context functions used inside a row access policy body have documented limitations when the query is coming from a Streamlit-in-Snowflake app specifically.

### Apache Iceberg via Apache Spark™
- Snowflake Horizon Catalog can enforce row access policies set on Apache Iceberg tables even when those tables are queried externally through Apache Spark — governance follows the table outside of Snowflake's own query engine.

---

## 11. Privileges & Management Models

### The two privileges
| Privilege | What it lets you do |
|---|---|
| **APPLY** | Add/drop the row access policy on a table or view. Note: the **account-level** grant (`APPLY ROW ACCESS POLICY` on `ACCOUNT`) additionally enables running `DESCRIBE` on protected tables/views — a nuance worth remembering. |
| **OWNERSHIP** | Full control of the policy object itself; required to alter most of its properties. Only **one role** can hold `OWNERSHIP` on a given policy at any time (standard Snowflake ownership rule, not unique to RAP). |

### Three management philosophies
| Action | Centralized | Hybrid | Decentralized |
|---|---|---|---|
| Create policies | Governance officer | Governance officer | Individual teams |
| Apply policies to objects | Governance officer | Individual teams | Individual teams |

- **Centralized** — one governance role does everything. Simplest to audit, least scalable.
- **Hybrid** — governance creates (for consistency/performance review), individual teams apply (for agility). This is the pattern shown in the official worked example.
- **Decentralized** — teams do both. Most scalable, hardest to govern centrally.

### Example: hybrid grants
```sql
USE ROLE SECURITYADMIN;
GRANT CREATE ROW ACCESS POLICY ON SCHEMA governance.policies TO ROLE rap_admin;
GRANT APPLY ON ROW ACCESS POLICY governance.policies.rap_region TO ROLE finance_role;
```
`rap_admin` can author new policies account-wide; `finance_role` can only attach/detach the one specific policy it was granted `APPLY` on.

---

## 12. DDL Command Reference

| Command | Purpose | Privilege needed |
|---|---|---|
| `CREATE ROW ACCESS POLICY` | Define a new policy | `CREATE ROW ACCESS POLICY` on the schema |
| `ALTER ROW ACCESS POLICY` | Modify the policy's own definition (e.g., its expression) without detaching it from any object | `OWNERSHIP` on the policy |
| `DROP ROW ACCESS POLICY` | Remove the policy object entirely | `OWNERSHIP` on the policy, or `OWNERSHIP` on the schema |
| `SHOW ROW ACCESS POLICIES` | List policies | `APPLY ROW ACCESS POLICY`, or `OWNERSHIP`/`APPLY` on the specific policy |
| `DESCRIBE ROW ACCESS POLICY` | View a policy's signature/body | Same as `SHOW` above |
| `ALTER TABLE / ALTER EXTERNAL TABLE / ALTER VIEW ... ADD/DROP ROW ACCESS POLICY` | Attach/detach a policy on a specific object | `APPLY ROW ACCESS POLICY` on account, **or** `OWNERSHIP` on the object + `APPLY` on the policy |

> ⚠️ **Documented quirk to know:** `UNDROP` is **not supported** for row access policy objects. If one is dropped, you must recreate it from scratch with `CREATE ROW ACCESS POLICY` and re-attach it with `ALTER TABLE`/`ALTER VIEW`.
>
> ⚠️ **Another quirk:** don't assume every `ALTER ROW ACCESS POLICY ... SET <property>` form is supported — Snowflake's own troubleshooting docs show a `SET COMMENT` attempt on a policy failing with an "unsupported token" error. The safe, always-documented way to change a policy's definition is `CREATE OR REPLACE ROW ACCESS POLICY`. Attaching/detaching is always done via `ALTER TABLE`/`ALTER VIEW`, never by altering the policy object to reference a table.

---

## 13. Monitoring & Auditing

### SQL-based discovery
| Tool | Scope | Use for |
|---|---|---|
| `SNOWFLAKE.ACCOUNT_USAGE.ROW_ACCESS_POLICIES` | Account-wide | Catalog of every RAP that exists in the account |
| `SNOWFLAKE.ACCOUNT_USAGE.POLICY_REFERENCES` | Account-wide | Which objects/columns have a policy assigned |
| `INFORMATION_SCHEMA.POLICY_REFERENCES(...)` table function | Per-database | Same idea, scoped to one database; can query by policy name or by target object |

### Snowsight (Governance & security → Tags & policies)
- Two panes: **Dashboard** (coverage % and prevalence of policies/tags) and **Tagged Objects** (searchable, filterable list).
- Both **require a running warehouse**.
- **Dashboard refreshes every 12 hours** — not real time.
- **Tagged Objects** has up to **2 hours of latency** and returns a maximum of **1,000 objects**.
- Access requires Enterprise Edition+ and either the `ACCOUNTADMIN` role, or an account role **directly** granted both the `SNOWFLAKE.GOVERNANCE_VIEWER` and `SNOWFLAKE.OBJECT_VIEWER` database roles. Note the word *directly* — Snowsight does **not** walk role hierarchies for this check, so granting these database roles to a parent role that your analyst inherits from is **not** sufficient; the grant must land on the exact role being used.

### Auditing patterns
- `SHOW ROW ACCESS POLICIES` lists everything not yet dropped.
- A RAP owner can use **Time Travel or streams on the mapping table** to reconstruct historical entitlement state.
- To answer "what could user X see," the RAP owner assumes that user's role and queries directly — **Snowflake has no built-in "query as user X" audit function**; any such capability has to be custom-built into the policy's own logic.
- Errors from RAP evaluation are captured in `QUERY_HISTORY` — Snowflake records the **first** error message encountered during evaluation of a given query.

---

## 14. Troubleshooting Reference

High-yield error scenarios (condensed — know the *pattern*, not the exact error text):

| Symptom | Root cause | Fix |
|---|---|---|
| "Return type is not BOOLEAN" | Forgot `RETURNS BOOLEAN` in `CREATE ROW ACCESS POLICY` | Every policy body must resolve to boolean — no exceptions |
| "No current database" | Tried to create the policy without a database/schema context | `USE DATABASE`/`USE SCHEMA` first, or use a fully qualified name (expected, since RAP is schema-level) |
| "Row access policy cannot be attached to a Materialized view" | Underlying table/view already has a RAP | See Section 10 → Materialized views |
| Insufficient privileges on schema | Missing `CREATE ROW ACCESS POLICY` or `APPLY` grants | Re-check Section 11's privilege table |
| `UNDROP` unsupported error | Tried to `UNDROP` a dropped policy | Recreate with `CREATE ROW ACCESS POLICY` + re-attach; there is no undo |
| "Unsupported token" on an `ALTER ROW ACCESS POLICY ... SET ...` | Not every ALTER sub-clause is supported on the policy object | Use `CREATE OR REPLACE ROW ACCESS POLICY` to change the definition |

---

## 15. End-to-End Walkthrough (Hybrid/Centralized Model)

This mirrors the documented worked example, with the roles-and-reasons made explicit — this sequencing is a realistic exam scenario question in disguise.

```sql
-- 1) The protected table
CREATE TABLE sales (
  customer   VARCHAR,
  product    VARCHAR,
  spend      DECIMAL(20,2),
  sale_date  DATE,
  region     VARCHAR
);

-- 2) A dedicated mapping table, in a governance schema end users never touch
CREATE TABLE security.manager_region_map (
  sales_manager VARCHAR,
  region        VARCHAR
);

-- 3) SECURITYADMIN creates a role whose ONLY job is to be able to read the mapping table
USE ROLE SECURITYADMIN;
CREATE ROLE mapping_role;
GRANT SELECT ON TABLE security.manager_region_map TO ROLE mapping_role;

-- 4) The schema owner authors the policy itself
USE ROLE schema_owner_role;
CREATE OR REPLACE ROW ACCESS POLICY security.sales_policy
AS (sales_region VARCHAR) RETURNS BOOLEAN ->
  'SALES_EXECUTIVE_ROLE' = CURRENT_ROLE()
  OR EXISTS (
      SELECT 1 FROM security.manager_region_map
      WHERE sales_manager = CURRENT_ROLE()
      AND region = sales_region
  )
;

-- 5) SECURITYADMIN reassigns OWNERSHIP of the *policy* to mapping_role,
--    and lets the analyst role attach/detach it
USE ROLE SECURITYADMIN;
GRANT OWNERSHIP ON ROW ACCESS POLICY security.sales_policy TO ROLE mapping_role;
GRANT APPLY ON ROW ACCESS POLICY security.sales_policy TO ROLE sales_analyst_role;

-- 6) Bind the policy to the table
ALTER TABLE sales ADD ROW ACCESS POLICY security.sales_policy ON (region);

-- 7) Let the actual sales managers query the table
GRANT SELECT ON TABLE sales TO ROLE sales_manager_role;
```

### Why step 5 is the whole point of the exercise
Once `OWNERSHIP` of the policy moves to `mapping_role`, the policy's `EXISTS` subquery against `manager_region_map` runs using **`mapping_role`'s** privileges at query time — not `SECURITYADMIN`'s, and not the querying `sales_manager_role`'s. This is "owner's rights" in action, and it's exactly why `sales_manager_role` in step 7 only needed `SELECT` on `sales` — never on the mapping table. This is the **Principle of Least Privilege**, implemented structurally rather than by hoping nobody queries the mapping table directly.

---

## 16. Memoizable Functions in Practice

The problem: a policy with three separate mapping-table lookups pays the cost of three subqueries per evaluation:
```sql
CREATE OR REPLACE ROW ACCESS POLICY rap_no_memo
AS (region_id NUMBER, customer_id NUMBER, product_id NUMBER)
RETURNS BOOLEAN ->
  EXISTS(SELECT 1 FROM regions   WHERE id = region_id)   OR
  EXISTS(SELECT 1 FROM customers WHERE id = customer_id) OR
  EXISTS(SELECT 1 FROM products  WHERE id = product_id)
;
```
The fix: convert each lookup into a **memoizable** scalar UDF that returns the whole allowed-set as an array, then check membership with `ARRAY_CONTAINS` instead of re-querying per row:
```sql
CREATE OR REPLACE FUNCTION allowed_regions()
  RETURNS ARRAY
  MEMOIZABLE
  AS 'SELECT ARRAY_AGG(id) FROM regions';

CREATE OR REPLACE ROW ACCESS POLICY rap_with_memo
AS (region_id NUMBER, customer_id NUMBER, product_id NUMBER)
RETURNS BOOLEAN ->
  ARRAY_CONTAINS(region_id,   allowed_regions())   OR
  ARRAY_CONTAINS(customer_id, allowed_customers())  OR
  ARRAY_CONTAINS(product_id,  allowed_products())
;
```
A `MEMOIZABLE` function's result is cached for repeated calls with the same input during query execution instead of being recomputed — this is exactly the optimization the Performance section (Section 6) points to when it says "replace the mapping table reference with a memoizable function."

---

## 17. Advanced Pattern: Protecting the Mapping Table Itself

This is the most conceptually advanced scenario in the topic, and a great source of hard exam questions: **a mapping table can itself carry its own row access policy.**

```sql
-- The mapping table
CREATE OR REPLACE TABLE sales.tables.regional_managers (
  allowed_region VARCHAR,
  allowed_role   VARCHAR
);

-- Policy #1 — protects the MAPPING table so only the relevant role
-- can even see its own row (role-hierarchy aware)
CREATE OR REPLACE ROW ACCESS POLICY governance.policies.rap_map_guard
AS (allowed_role VARCHAR) RETURNS BOOLEAN ->
  IS_ROLE_IN_SESSION(allowed_role);

ALTER TABLE sales.tables.regional_managers
  ADD ROW ACCESS POLICY governance.policies.rap_map_guard ON (allowed_role);

-- Policy #2 — protects the actual DATA table, using a lookup
-- against the (now separately protected) mapping table
CREATE OR REPLACE ROW ACCESS POLICY governance.policies.rap_data_guard
AS (region VARCHAR) RETURNS BOOLEAN ->
  EXISTS (
    SELECT 1 FROM sales.tables.regional_managers
    WHERE allowed_region = region
  );

ALTER TABLE sales.tables.data
  ADD ROW ACCESS POLICY governance.policies.rap_data_guard ON (region);
```

### Why this is tricky
When `rap_data_guard` runs its `EXISTS` subquery against `regional_managers`, that subquery is itself subject to `rap_map_guard` — evaluated using **`rap_map_guard`'s own owner's rights**, which may be a completely different role than the owner of `rap_data_guard`. You end up with two independent "owner's rights" contexts layered on top of each other. Get the ownership/grants wrong on either policy and the outer policy silently returns fewer rows than intended (because its subquery, itself filtered, "sees" less of the mapping table than the policy author assumed).

---

## 18. Compare: Row Access Policy vs Masking Policy

| Aspect | Row Access Policy | Masking Policy |
|---|---|---|
| What it protects | Whole **rows** (visible or not) | **Column values** (shown, masked, or transformed) |
| Object level | Schema-level object | Schema-level object |
| Applies to `INSERT`? | No — never blocks inserts | N/A — masking applies to *reads*, not write-time blocking either |
| Order when both exist on one object | **Evaluated first** | Evaluated **after** the row access policy has already filtered rows |
| Can share a column between the two policy types? | No — a column can be in a RAP signature **or** a masking policy signature, never both | Same rule, other direction |
| Applies to the object owner? | Yes — ownership does not exempt you | (confirm in the Masking Policy chapter — expect the same governance-overrides-ownership philosophy, but we'll verify against the masking docs directly rather than assume) |
| Typical use case | "Which rows can this role see at all" (regional data, multi-tenant isolation, department-level access) | "This role can see the row, but sensitive columns (SSN, salary) should be hidden/redacted" |

We'll build out the full masking-policy side of this table with confirmed detail in the next chapter — flagging it now so you see the comparison forming early.

---

## 19. Common Misconceptions

1. **"The table owner always sees everything."** False for RAP-protected tables — ownership controls the object, not policy-driven visibility.
2. **"RAP stops bad data from being inserted."** False — RAP is read/target-of-DML filtering only, never an insert guard.
3. **"`CURRENT_ROLE()` in a policy sees all of a user's active roles."** False — it only reflects the single active primary role; secondary roles need `IS_ROLE_IN_SESSION()`.
4. **"`CURRENT_DATABASE()`/`CURRENT_SCHEMA()` inside a policy reflect the querying session."** False — they reflect the protected object's own database/schema.
5. **"Cloning a table always keeps the policy pointing at the same original policy object."** Only true for **foreign** (cross-schema/database) references. Same-schema/database references get remapped to the clone's own copy of the policy.
6. **"A masking policy and a row access policy can both reference the same column, since they do different things."** False — a column is exclusively bound to one type of policy signature at a time.
7. **"Row access policies work fine as a mapping-table source using an external table for flexibility."** False — external tables are explicitly unsupported as the mapping table source.

---

## 20. SnowPro Core Exam Traps

- Questions that swap **RAP** and **masking policy** behavior in the answer choices (e.g., attributing "evaluated first" or "protects columns" to the wrong one).
- Questions describing a user with a role granted as **secondary**, expecting you to know `CURRENT_ROLE()` won't catch it.
- Questions about **Standard Edition** trying to use RAP — remember, it's Enterprise+ only, same trap pattern shows up for masking policies and object tagging.
- Questions where a **data share consumer** gets unexpected `NULL`/no-rows behavior — test whether you know it's because `CURRENT_USER()`/`CURRENT_ROLE()` don't resolve across account boundaries.
- Questions asking what happens on `CREATE TABLE ... LIKE` vs `... AS SELECT` vs `... CLONE` with respect to the policy — these three are commonly used as distractors against each other because their behavior is genuinely different (no policy at all / filtered-but-unprotected data / policy retained).
- Questions testing whether you know `COUNT(*)` performance changes are due to **lost metadata optimization**, not "the policy makes the table bigger" (it doesn't — no data is duplicated).
- A "trick" option that says an `ALTER ROW ACCESS POLICY` statement can rename or fully redefine attachment to a table — attachment is **always** via `ALTER TABLE`/`ALTER VIEW`, never by altering the policy to "point at" a table.

---

## 21. One-Page Revision Sheet

```
ROW ACCESS POLICY — QUICK RECALL
─────────────────────────────────
Edition:            Enterprise+  (same tier as masking, tagging)
Object level:       Schema-level object
Filters:            SELECT, target rows of UPDATE/DELETE/MERGE
Never filters:      INSERT
Owner exempt?       NO — even table OWNERSHIP doesn't bypass RAP
Execution rights:   POLICY OWNER's privileges (NOT the querying user's)
Order vs masking:   RAP evaluated FIRST, masking after
Shared column?      A column = RAP signature OR masking signature, never both
Nested views:       Table policy → then view policies, left to right, all must pass
Role gotcha:        CURRENT_ROLE() = primary role ONLY; use IS_ROLE_IN_SESSION()
                     for secondary/hierarchical roles
CURRENT_DATABASE()/
CURRENT_SCHEMA()
inside policy:      Returns the PROTECTED OBJECT's db/schema, not session's
Mapping table:      Store in same DB as protected table; external tables
                     can NEVER be a mapping table source
Perf fix:            fewer args > simple CASE > memoizable function >
                     mapping-table subquery (roughly cheapest → priciest)
COUNT(*) gotcha:     Loses metadata shortcut once RAP attached — expected slow
CLONE:               same-schema policy ref → remapped to cloned policy
                     cross-schema (foreign) policy ref → stays on original
LIKE:                no policy, empty table
CTAS:                filtered data copied, but NO policy on new table
Time Travel:         supported; mapping table always evaluated as CURRENT, not historical
Data share + CURRENT_USER/CURRENT_ROLE/secure UDF: returns NULL for consumer
                     → fix: CURRENT_ACCOUNT() or shared database role +
                       IS_DATABASE_ROLE_IN_SESSION()
UNDROP:              NOT supported for policy objects — recreate from scratch
Change the policy:   CREATE OR REPLACE ROW ACCESS POLICY (safe, documented)
Attach/detach:       ALTER TABLE / ALTER VIEW ... ADD/DROP ROW ACCESS POLICY
Privileges:          APPLY (attach/detach; account-level APPLY also enables
                     DESCRIBE) | OWNERSHIP (full control of policy object)
```

---

## 22. Mini Mock Test

### Conceptual (5)

**1.** Which Snowflake edition is the minimum required to use row access policies?
A) Standard B) Enterprise C) Business Critical D) VPS
**Answer: B.** RAP is an Enterprise Edition+ feature; Standard Edition cannot use it at all.

**2.** When a row access policy's body includes a subquery against a mapping table, whose privileges are used to execute that subquery?
A) The querying user's role B) The protected table's owner C) The row access policy's owner D) ACCOUNTADMIN, always
**Answer: C.** This is the "owner's rights" model — it's what lets end users query the protected table without ever needing direct access to the mapping table.

**3.** Which statement type is **not** filtered by a row access policy?
A) SELECT B) The rows targeted by MERGE C) INSERT D) The rows targeted by DELETE
**Answer: C.** RAP never blocks inserts — it's purely a visibility filter for reads and for the rows targeted by UPDATE/DELETE/MERGE.

**4.** What type of Snowflake object is a row access policy?
A) Account-level object B) Database-level object C) Schema-level object D) Warehouse-level object
**Answer: C.**

**5.** Which privilege grants full control over a row access policy, including the ability to alter most of its properties?
A) APPLY B) USAGE C) OWNERSHIP D) MODIFY
**Answer: C.** `APPLY` only lets you attach/detach the policy on objects; it doesn't let you redefine the policy itself.

### Scenario (3)

**6.** A provider shares a table protected by a row access policy whose expression calls `CURRENT_USER()`. A consumer queries the shared table. What's the most likely outcome?
A) The query fails outright B) `CURRENT_USER()` returns NULL in the consumer's context, so that branch of the policy effectively can't grant access C) The consumer sees all rows automatically D) Snowflake automatically substitutes the provider's admin user
**Answer: B.** The provider account has no visibility into the consumer's users; the documented workaround is `CURRENT_ACCOUNT()` or sharing a database role and using `IS_DATABASE_ROLE_IN_SESSION()`.

**7.** An architect wants to build a materialized view on top of a table that already has a row access policy attached. What happens?
A) It works exactly like any other materialized view B) It fails — a materialized view cannot be created from a table that already has a row access policy C) It works, but the policy is silently dropped D) It works only if run as ACCOUNTADMIN
**Answer: B.** You can add a RAP to an MV, or build an MV from an already-protected table — not both; the base object must be unprotected first if an MV is the goal (or use a dynamic table instead).

**8.** A user has `SALES_MANAGER` granted and currently active as a **secondary** role (their primary role is something else). A row access policy checks `CURRENT_ROLE() = 'SALES_MANAGER'`. What happens when this user queries the protected table?
A) They see the rows normally, since the role is active B) They see zero qualifying rows from that branch, because `CURRENT_ROLE()` only reflects the primary role C) Snowflake throws a compilation error D) The policy is skipped entirely
**Answer: B.** This is exactly why Snowflake recommends `IS_ROLE_IN_SESSION()` when role hierarchy/secondary roles matter.

### True / False (2)

**9.** "A row access policy can prevent a role from inserting a row with an out-of-scope value into a protected table." **False** — RAP never governs INSERT; it only filters what's returned by reads and DML targeting.

**10.** "If a table references a row access policy that lives in a different database, and that table is cloned, the cloned table keeps pointing at the exact same original policy (not a clone of it)." **True** — this is the documented "foreign reference is retained" behavior, which only differs from the same-schema case where the clone gets remapped to the cloned policy copy.

### Trick Question (1)

**11.** A row access policy on `orders` includes `CURRENT_SCHEMA() = 'SESSION_WORK'` as one of its conditions, expecting to match whatever schema the querying user currently has active via `USE SCHEMA`. The user runs `USE SCHEMA session_work;` and then queries `orders`, which physically lives in a schema called `PROD_SALES`. Does the condition evaluate `TRUE`?
A) Yes, because the user's session is in `SESSION_WORK` B) No — inside a row access policy body, `CURRENT_SCHEMA()` returns the schema of the **protected object** (`PROD_SALES`), not the querying session's active schema C) It depends on the warehouse D) It throws an error
**Answer: B.** This is the exact documented gotcha from the Considerations section — `CURRENT_DATABASE()`/`CURRENT_SCHEMA()` inside a policy body resolve to the protected object's own location, never the caller's session context. A policy author trying to use these functions to detect "session context" has built a policy that can never behave the way they expect.

---

**Next chapter:** Masking Policies — where we'll revisit the RAP-vs-masking comparison table with full detail, and cover column-level security, external tokenization, and tag-based masking.
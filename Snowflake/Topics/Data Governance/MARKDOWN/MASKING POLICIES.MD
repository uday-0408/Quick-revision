# SnowPro Core — Data Governance & Security Mastery Notes

*Compiled instructor-style notes for SnowPro Core exam prep. Built for real architectural understanding first, exam recall second — because on scenario questions, understanding beats memorization every time.*

**How these notes are marked:**
- 🏢 **Real-World** — how this actually gets used on a live enterprise account
- 🎯 **Exam Angle** — why SnowPro Core cares about this specific point
- ⚠️ **Gotcha** — the thing that trips people up, in the docs or in the exam
- 🧠 **Misconception** — what people *assume* that is actually wrong

## 🗂️ Coverage Tracker

- [x] **1. Masking Policies** (Column-Level Security: Dynamic Data Masking + External Tokenization)
- [ ] 2. Row Access Policies
- [ ] 3. Object Tagging
- [ ] 4. Trust Center
- [ ] 5. Replication & Failover
- [ ] 6. Data Lineage

---

# Topic 1: Masking Policies (Column-Level Security)

## 1.1 Where This Fits in the Bigger Picture

Snowflake's governance model answers four questions about any piece of data: *who can see which rows*, *who can see which column values*, *how is this object labeled/classified*, and *who touched it and when*. Masking Policies answer the second question — they control **values within a column**, not entire rows. That's the key distinction from Row Access Policies (Topic 2), which control **whether a row is returned at all**. Object Tags (Topic 3) will turn out to be a *delivery mechanism* for masking policies at scale, so this chapter is genuinely the foundation the rest of the governance stack builds on.

## 1.2 What Is Column-Level Security? (The Umbrella Term)

> 🎯 **Exam Angle:** Column-level Security requires **Enterprise Edition or higher**. Edition-gating is one of the most reliably-tested facts on SnowPro Core — memorize which features need which edition.

Column-Level Security is the umbrella term for **two** distinct features:

| Feature | What it does | Data at rest | Requires |
|---|---|---|---|
| **Dynamic Data Masking (DDM)** | Masks plain-text data *at query time* | Stored **unmasked** in Snowflake | Enterprise Edition+ |
| **External Tokenization (ET)** | De-tokenizes data *at query time* that was already tokenized before it ever reached Snowflake | Stored **tokenized** — Snowflake never even briefly holds plaintext | Enterprise Edition+ (to integrate a provider; the underlying external-function mechanism itself only needs Standard Edition) |

Both features share the *exact same object type* — a **masking policy** — and the *exact same DDL commands* (`CREATE/ALTER/DROP/SHOW/DESCRIBE MASKING POLICY`). The only structural difference: an External Tokenization policy body calls an [external function](https://docs.snowflake.com/sql-reference/external-functions) to make a REST call out to a third-party tokenization provider; a DDM policy body is pure SQL logic.

> ⚠️ **Gotcha (Edition nuance):** Writing external functions in general only requires **Standard Edition**. It's specifically *using* them to integrate a tokenization provider for External Tokenization that pushes you into Enterprise Edition territory. Don't conflate "external functions need Enterprise" (false) with "External Tokenization needs Enterprise" (true).

## 1.3 What *Is* a Masking Policy? (Anatomy)

**Why this feature exists:** Before masking policies, the only way to restrict column visibility was **secure views** — but that meant an explosion of views (one per audience) and no separation of duties, because whoever managed the view effectively controlled access. Masking policies solve both problems: one policy definition can protect **thousands of columns** across every database and schema in the account, and — critically — the policy is a **schema-level object completely decoupled from the table**.

A masking policy always has exactly this shape:

```sql
CREATE MASKING POLICY employee_ssn_mask AS (val string) RETURNS string ->
  CASE
    WHEN CURRENT_ROLE() IN ('PAYROLL') THEN val
    ELSE '******'
  END;
```

| Component | Meaning |
|---|---|
| **One data type** | Input and output type of the value being evaluated |
| **One or more conditions** | Built using [conditional expression functions](https://docs.snowflake.com/sql-reference/expressions-conditional), [context functions](https://docs.snowflake.com/sql-reference/functions-context) (`CURRENT_ROLE`, `INVOKER_ROLE`, `IS_ROLE_IN_SESSION`…), or a lookup against a custom entitlement table |
| **One or more masking functions** | Built-in functions (`REGEXP_REPLACE`, `SHA2`), a UDF, or an external function (for detokenization) |

### 🧠 The Single Most Important Misconception to Kill Right Now

> **"The table owner can always see the full data — ownership means full access."** **False.** Masking policies exist specifically to break that assumption. By default:
> - Object owners (the role holding `OWNERSHIP`) **cannot unset** a masking policy from a column.
> - Object owners **cannot view** column data that a masking policy protects.
> - The policy evaluation has **no hardcoded bypass** for powerful default roles either — if a policy's `CASE` logic doesn't explicitly allow `ACCOUNTADMIN` or `SECURITYADMIN`, **those roles see masked data too**, exactly like any custom role.

> 🎯 **Exam Angle:** This is a *favorite* SnowPro trap. Any question implying "the account admin can obviously bypass a masking policy" — the answer is **no**, unless the policy body explicitly grants that role visibility.

## 1.4 How a Masking Policy Works — Internal Workflow

```text
User submits a query referencing a masked column
        │
        ▼
Snowflake's compiler scans EVERY location that column appears —
SELECT list, JOIN predicates, WHERE predicates, GROUP BY, ORDER BY
        │
        ▼
Query is REWRITTEN: every reference to the column is wrapped
in the masking policy's CASE / SQL expression
        │
        ▼
At each reference, the policy condition is evaluated using
session context (CURRENT_ROLE / INVOKER_ROLE / entitlement lookup…)
        │
        ▼
   Condition TRUE                    Condition FALSE
        │                                  │
        ▼                                  ▼
Real / detokenized value returned    Masked / tokenized / fixed value returned
```

**The key insight:** masking happens **at the query layer**, not the storage layer. The physical micro-partition never changes. Two different roles querying the *same row at the same instant* get two different logical results from identical bytes on disk. (The one big exception — CTAS — is covered in §1.11.4, and it's a great exam trap.)

## 1.5 Applying a Policy to a Column

Two paths, depending on whether the object already exists:

```sql
-- NEW table/view: bake it in at creation
CREATE OR REPLACE TABLE user_info (ssn string MASKING POLICY ssn_mask);
CREATE OR REPLACE VIEW  user_info_v (ssn MASKING POLICY ssn_mask_v) AS SELECT * FROM user_info;

-- EXISTING table/view: attach after the fact
ALTER TABLE IF EXISTS user_info MODIFY COLUMN ssn SET MASKING POLICY ssn_mask;
ALTER VIEW  user_info_v MODIFY COLUMN ssn SET MASKING POLICY ssn_mask_v;
```

### Replacing a Policy on a Column — Two Paths, Two Different Risks

| Approach | Syntax | The catch |
|---|---|---|
| **Unset then set** (2 statements) | `ALTER TABLE t1 MODIFY COLUMN c1 UNSET MASKING POLICY;` then `ALTER TABLE t1 MODIFY COLUMN c1 SET MASKING POLICY p2;` | ⚠️ **Data-leak window.** Between the two statements, the column is completely **unprotected**. Anyone querying in that gap sees raw data. |
| **FORCE** (1 statement) | `ALTER TABLE t1 MODIFY COLUMN c1 SET MASKING POLICY p2 FORCE;` | No leak window, **but**: `p2`'s data type must exactly match the currently-attached policy's data type, and if `p2`'s conditions are *stricter* than `p1`'s, users who previously had access can suddenly lose it with zero downtime buffer. `FORCE` also composes with a conditional-masking `USING` clause: `SET MASKING POLICY policy1 USING (c1, c3, c4) FORCE;` |

> 🎯 **Exam Angle:** Classic scenario question — "which method avoids exposing data during a policy swap?" → **FORCE**. Classic follow-up trap — "does FORCE avoid *all* risk?" → No, it trades a *leak* risk for a *sudden access-loss* risk.

Separately: **`ALTER MASKING POLICY`** (changing the policy's internal logic/body) does **not** require unsetting the policy from any column first — the column stays protected throughout the update. This is a deliberately safer operation than replacing which policy is attached.

**Dropping** a policy (`DROP MASKING POLICY`) is the opposite — you **must** unset it from every column first, or the drop fails.

## 1.6 Conditional Masking Policies

A regular masking policy takes one argument (the column to mask). A **conditional** masking policy takes the masked column *plus one or more additional columns* whose values decide whether masking applies:

```sql
CREATE MASKING POLICY email_visibility AS
(email varchar, visibility string) RETURNS varchar ->
  CASE
    WHEN CURRENT_ROLE() = 'ADMIN' THEN email
    WHEN VISIBILITY = 'PUBLIC' THEN email
    ELSE '***MASKED***'
  END;
```

- The **first** argument is always the column being masked; every argument after it is a conditional column.
- Applying it requires a `USING (...)` clause: `ALTER TABLE user_info MODIFY COLUMN email SET MASKING POLICY email_visibility USING (email, visibility);`

**Considerations (🎯 exam-relevant):**
- All columns named in the policy must live in the **same table or view**.
- **Minimize the number of conditional columns** — Snowflake evaluates every one of them at query runtime, so more columns = slower queries.
- Track which columns use conditional masking via the `POLICY_REFERENCES` table function.

> ⚠️ **Gotcha:** Conditional masking policies inherit **every normal masking-policy limitation**, *plus* extra friction specifically around **Row Access Policies** and **External Tables** (see §1.9 and §1.11.3). Don't assume "conditional" is a strict superset with zero extra caveats.

## 1.7 Query-Runtime Rewrite Mechanics — THE Exam Trap Zone

This is, without question, the highest-yield section for scenario-based exam questions.

**Rule:** a masking policy applies to its column **everywhere that column is referenced** — not just in the `SELECT` list. That includes JOIN predicates, WHERE predicates, GROUP BY, and ORDER BY.

**Why:** if masking only applied to the `SELECT` list, a curious analyst could de-anonymize data by cleverly filtering on it (e.g., binary-searching a masked salary column via `WHERE salary > X`). Applying the mask *everywhere* closes that loophole.

**Normal case:**
```sql
-- User writes:
SELECT email, city FROM customers WHERE city = 'San Mateo';
-- Snowflake actually runs:
SELECT MASK_FN(email), city FROM customers WHERE city = 'San Mateo';
```

**Anti-pattern #1 — masked column in a WHERE comparison:**
```sql
-- User writes:
SELECT email FROM customers WHERE email = 'user@example.com';
-- Snowflake actually runs:
SELECT MASK_FN(email) FROM customers WHERE MASK_FN(email) = 'user@example.com';
```
⚠️ **The masked side almost never equals the literal string**, so this silently returns **zero rows** — even though the row genuinely exists. This is a real production debugging nightmare disguised as "the data must be missing."

**Anti-pattern #2 — masked column in a JOIN predicate:**
```sql
-- User writes:
SELECT b.email, d.city FROM emp_basic b JOIN emp_details d ON b.email = d.email;
-- Snowflake actually runs:
SELECT MASK_FN(b.email), d.city FROM emp_basic b JOIN emp_details d ON MASK_FN(b.email) = MASK_FN(d.email);
```
If both sides mask to the **same fixed value** (e.g., `'******'`), you can get **spurious matches** across unrelated rows — the opposite failure mode, silently *wrong* data instead of *missing* data.

> 🎯 **Exam Angle:** Any question with the shape "a query that should return rows returns nothing" or "a join is returning matches that don't make sense" and a masked column is anywhere in the WHERE/JOIN clause → the answer is this rewrite behavior, not a bug, not a permissions error.

## 1.8 Nested Policies: Table → View → View → … Evaluation Order

Snowflake supports masking policies stacked on **both** a table *and* a view built on that table simultaneously. The evaluation order is fixed:

1. The policy on the **table** always executes **first**.
2. The policy on the **view** executes **after** the table's policy.
3. For a chain (`table` → `view_1` → `view_2` → … → `view_n`), policies apply **left to right**, in that sequence.

> 🏢 **Real-World:** This is why the "expose an aggregate without exposing raw values" pattern in §1.11.5 works at all — the table-level policy has already run by the time the view's own logic executes, so there's no way to bypass it by querying through the view.

## 1.9 The One Rule You Cannot Work Around: Masking Policy vs. Row Access Policy

**A single column cannot be referenced in both a masking policy signature *and* a row access policy signature at the same time.** This applies to conditional columns too — if a column is used as a conditional argument in a masking policy, it *also* can't simultaneously be part of a row access policy signature.

> 🎯 **Exam Angle:** This is a guaranteed comparison-style question once you reach Topic 2. Column-level protection and row-level protection are architecturally independent tools that cannot double up on the *exact same* column reference.

## 1.10 Simulating a Policy Before You Break Something

The `POLICY_CONTEXT` function lets you **dry-run** what a query would return under a *different* role/context — without actually switching roles:

```sql
EXECUTE USING POLICY_CONTEXT(
  SNOWFLAKE$SESSION_ACTIVATED_ROLES => ('ANALYST', 'PUBLIC')
) AS SELECT * FROM empl_info;
```

🏢 **Real-World:** this is exactly how a policy administrator QA-tests a new masking or row access policy against multiple roles before rolling it out — no need for a shared test account per role.

## 1.11 Interactions With Other Snowflake Features (The Gotcha Zone)

This is where most exam "trick" questions live — masking policies quietly change behavior in a dozen adjacent features.

### 1.11.1 Materialized Views — The Biggest Limitation

```text
Do you need masking AND a materialized view on overlapping data?
        │
        ▼
Does an MV already exist, built from this base table?
   │                                    │
  YES                                   NO
   │                                    │
   ▼                                    ▼
❌ You CANNOT add a masking policy    Can the masking policy live
   to the base table column until      directly on the MV's OWN
   the MV is dropped                   column instead of the base table?
                                              │
                                    ┌─────────┴──────────┐
                                   YES                   NO — must be on the base table
                                    │                     │
                                    ▼                     ▼
                           ✅ Works — apply policy    Consider a DYNAMIC TABLE
                              to the MV column          instead of a plain MV;
                              via CREATE/ALTER MV        dynamic tables support
                                                          masking + row access +
                                                          tags directly.
```

Two concrete, memorize-these limitations:

1. **You cannot add a masking policy to a base table column once a materialized view already exists on that table.** Snowflake errors out: *"One or more materialized views exist on the table."*
2. **If a base table column already has a masking policy, that column is excluded when you try to build a materialized view from it** — you'll hit an `Unsupported feature 'CREATE ON MASKING POLICY COLUMN'` error if you try to include it.

Net effect: **masked columns and materialized views pulling from those same columns are fundamentally incompatible.** You must choose one layer to own the masking — the base table or the MV — and if you need both masking *and* incremental materialization on the exact same logical column, reach for a **Dynamic Table** instead.

### 1.11.2 Dynamic Tables

Unlike plain materialized views, a **dynamic table** *can* carry a row access policy, a masking policy, **and** a tag simultaneously — making it the more flexible option when you need governance controls plus materialized/incremental behavior together.

### 1.11.3 External Tables

- You **cannot** assign a masking policy to the auto-created `VALUE` column at `CREATE EXTERNAL TABLE` time — it must be done afterward via `ALTER TABLE ... ALTER COLUMN`.
- Any policy protecting the `VALUE` column **must** use data type `VARIANT`.
- **Virtual columns** (derived from `VALUE`) can carry their *own*, potentially less-strict policy — but only if the `VALUE` column's policy explicitly sets `EXEMPT_OTHER_POLICIES = TRUE`. Without that property, the virtual column just silently inherits the `VALUE` column's policy.
- A virtual column **can** be used as a *conditional* column argument, but it **cannot** be the *primary* masked/tokenized column.
- ⚠️ **Gotcha:** an external table can never be used as a lookup/entitlement table inside a masking policy's subquery logic.
- ⚠️ **Bigger gotcha — cloning:** cloning a database clones the masking policy but **not** the external table's underlying stage data. The cloned policy ends up **dangling**, referencing a table that doesn't exist in the clone. Workaround: move the external table into its own dedicated schema *inside* the database before cloning, and reference it with a fully-qualified name in the policy.

### 1.11.4 CREATE TABLE … AS SELECT (CTAS) — The Storage Exception

> 🧠 **Misconception:** "Masking never touches data at rest, only query output." **Mostly true — except CTAS.**

Running a CTAS statement applies any masking policies on the source columns **before the new table is populated** — meaning the **masked value gets physically baked into the new table's storage**. Why: a CTAS target may have a different column layout than the source, so Snowflake has no reliable way to implicitly re-attach the original masking policy to the new table's columns.

**Workaround** to get real (unmasked) data into the new table: run the CTAS using a role authorized to see the unmasked values, then transfer ownership of the new table and have the policy administrator (re-)apply masking policies to its columns.

> 🎯 **Exam Angle:** "Does CTAS output contain masked or plaintext data?" is a great trick question because the *default* DDM behavior (query-time only, storage untouched) is the opposite of what happens specifically during CTAS.

### 1.11.5 Aggregate Functions on Masked Columns

Aggregates (`COUNT`, `COUNT(DISTINCT …)`, etc.) run without error against masked columns. But if an unauthorized role directly selects the raw column, every row shows the **same fixed masked value** — so an aggregate computed on that raw masked output is essentially meaningless for that role.

**The documented access pattern:**
1. Table owner creates a view wrapping the aggregate: `CREATE VIEW ssn_count AS SELECT DISTINCT(ssn) FROM table1;`
2. Grant the limited role (e.g., `ANALYST`) privileges on the **view only** — never on the base table.
3. The masking policy lives on the **table** column. Because table-level policies always evaluate before view-level logic (§1.8), the data stays protected end-to-end even though the role never touches the table directly.

This is primarily an **access-surface pattern** (limit exposure via view + grant), not a way to magically unmask data. If the actual goal is *"let an unauthorized role compute accurate counts/groupings without ever seeing real values,"* that's precisely where **tokenization-style masking (a deterministic transform like `SHA2`, or true External Tokenization) beats a fixed-string mask** — a deterministic transform preserves distinctness (two identical inputs always mask to the same output), so `COUNT(DISTINCT …)` over the masked values still reflects real cardinality. A fixed string like `'***MASKED***'` collapses *every* row into one indistinguishable value, destroying that analytical value entirely. (This is exactly the "Preserve analytical value after de-identification" row in the DDM vs. ET table, §1.12.)

### 1.11.6 UDFs Inside a Masking Policy

The data type of the **column**, the **UDF**, and the **masking policy's declared return type** must all match exactly. A mismatch (e.g., column and UDF are `VARIANT` but the policy declares `VARCHAR`) surfaces as an error when the protected column is queried.

### 1.11.7 Tag-Based Masking + The Precedence Rule

Masking policies can also be attached to a **tag** instead of a column directly (full mechanics in Topic 3). The one rule to bank now:

> **A masking policy directly assigned to a column always takes precedence over a tag-based masking policy on that same column.**

🏢 **Real-World:** tag-based masking is how you get *scale* (attach one policy to a tag, and every tagged column across the account inherits it automatically). Direct column assignment is how you carve out a manual **exception** to that broad rule for one specific column.

### 1.11.8 Streams

Masking policies on a table's columns **carry over to any stream built on that table**. Unauthorized roles querying the stream see masked data, same as querying the table directly. For evaluation purposes, a stream uses the **latest table version available at query time** for any tables the policy itself references.

### 1.11.9 Cloning

- You **cannot** clone an individual policy object by itself.
- Cloning a **schema** clones every policy inside it.
- A cloned table maps to policies based on where the *original* policy lived:
  - If the source policy lived in the **same** database/schema as the source table → the clone points to the **newly-cloned copy** of that policy.
  - If the source table referenced a policy in a **different** (foreign) schema/database → the clone **keeps pointing to the original, non-cloned policy** — it does *not* get its own copy.

> ⚠️ **Gotcha:** that second bullet is exactly the kind of "which reference wins" detail SnowPro loves to test.

### 1.11.10 Data Sharing

| Situation | Behavior |
|---|---|
| Policy conditions call `CURRENT_ROLE()` / `CURRENT_USER()` on a shared object | Returns **NULL** in the *consumer's* account — the provider doesn't control the consumer's roles/users |
| Workaround #1 | Use `CURRENT_ACCOUNT()` in the policy condition instead |
| Workaround #2 (recommended) | Write the condition around `IS_DATABASE_ROLE_IN_SESSION`, share a **database role**, and have the consumer grant that shared database role to one of their own account roles |
| Provider creating a policy in a **reader account** | ❌ Not supported |
| Consumer applying a *new* policy to a shared table/view | ❌ Not supported — workaround: import the share, build a **local view** on top of it |
| Consumer querying a shared object whose policy references a mapping/entitlement table from a **different provider's share** | ❌ Fails for the consumer (the original provider can still query it successfully) |
| Policy calls an **external function** on a shared object | ❌ Always errors — you cannot update a shared table's policy to call an external function, or attach an external-function-based policy to an already-shared object |

> 🎯 **Exam Angle — memorize this one-liner:** **External Tokenization is fundamentally incompatible with Secure Data Sharing**, because ET policies require external functions, and external functions cannot be invoked in the context of a share. If a question describes wanting *both* tokenization-based masking *and* sharing the table externally, DDM is the only viable answer.

### 1.11.11 Replication

Masking policies and their column assignments **do** replicate as part of database replication / replication groups — but replication **fails** if either:
- The primary database is Enterprise Edition+ (has a policy), but one or more target accounts approved for replication are on a **lower edition**, or
- A table/view in the primary database has a **dangling reference** to a policy that lives in a *different, non-replicated* database.

The dangling-reference failure is avoided by replicating the related databases together as a single **replication group** rather than individually. (Full mechanics in Topic 5 — and remember: failover/failback specifically requires **Business Critical Edition+**, a stricter requirement than base replication.)

### 1.11.12 Query Profile / EXPLAIN

`EXPLAIN` on a query touching a masked column shows the **masked output value**, never the masking policy's internal logic. This is a deliberate security design choice — running `EXPLAIN` should never leak *how* a policy decides to mask something.

### 1.11.13 Unloading Data

`COPY INTO <location>` respects masking — an unauthorized role exporting a masked column to a file/stage gets the **masked** data in the exported file. Data cannot "escape" a masking policy through an unload.

## 1.12 Choosing Between Dynamic Data Masking and External Tokenization

**Benefits comparison (condensed):**

| Capability | DDM | External Tokenization |
|---|---|---|
| Preserves analytical grouping value after de-identification | | ✔ |
| Data pre-loaded already tokenized (plaintext never touches Snowflake) | | ✔ |
| Data pre-loaded unmasked, no third party needed | ✔ | |
| Compatible with Secure Data Sharing | ✔ | |
| Write once, apply to thousands of columns | ✔ | ✔ |
| Supports centralized/decentralized/hybrid admin (SoD) | ✔ | ✔ |
| Can restrict even ACCOUNTADMIN/SECURITYADMIN | ✔ | ✔ |
| Replicable via database/account replication | ✔ | ✔ |

**Limitations comparison (condensed) — both features share these:**

| Limitation | Notes |
|---|---|
| Materialized views | See §1.11.1 |
| `DROP MASKING POLICY` | Must unset from all columns first |
| `DROP DATABASE` / `DROP SCHEMA` | Only works cleanly if the policy and every column it's used on are **fully self-contained** within that database/schema — no cross-boundary references |
| External tables | Can't be used as a lookup/subquery table (§1.11.3) |
| Input/output data type mismatch | The policy's input and output type **must be identical** |
| Future grants | Not supported on masking policies — workaround: grant the **global** `APPLY MASKING POLICY ON ACCOUNT` privilege to a custom role instead of relying on per-policy future grants |

## 1.13 Privileges & Management Models

| Privilege | What it actually lets you do |
|---|---|
| **APPLY** | Set/unset an *existing* policy on a column. Does **not** let you change the policy's logic. The *global* form (`APPLY MASKING POLICY ON ACCOUNT`) has a side effect: it also enables running `DESCRIBE` on tables/views. |
| **OWNERSHIP** | Full control over the policy's own definition. Only **one role** can hold this on a given policy at a time. |

You also always need **USAGE** on the parent database and schema to do anything with a policy inside it.

**Three management philosophies (🎯 exam loves comparing these):**

| Model | Who creates policies | Who applies them to columns | Trade-off |
|---|---|---|---|
| **Centralized** | Security officer role only | Security officer role only | Most consistent, but a bottleneck |
| **Hybrid** | Security officer role | Individual teams (via granted `APPLY`) | Consistent policy creation, distributed application |
| **Decentralized** | Individual teams | Individual teams | Fastest, but highest risk of inconsistent/missing masking |

`CREATE MASKING POLICY` requires the privilege on the **schema**; `ALTER`/`DROP` require **OWNERSHIP** on the policy itself.

## 1.14 Monitoring & Discovery

| Tool | What it shows |
|---|---|
| `SNOWFLAKE.ACCOUNT_USAGE.MASKING_POLICIES` | Catalog of **every policy that exists** in the account |
| `SNOWFLAKE.ACCOUNT_USAGE.POLICY_REFERENCES` | Account-wide catalog of **assignments** (which columns use which policy) |
| `INFORMATION_SCHEMA.POLICY_REFERENCES(...)` table function | Same idea, scoped to a **single database** |
| Snowsight → **Governance & Security → Tags & Policies** | Visual **Dashboard** (coverage %, most-used policies; refreshes every **12 hours**) and **Tagged Objects** explorer (up to **2-hour** latency, caps at **1000 objects**) |

> ⚠️ **Gotcha:** Snowsight's governance dashboard requires Enterprise Edition+, and requires either the `ACCOUNTADMIN` role **or** an account role with a **direct** grant of both the `GOVERNANCE_VIEWER` and `OBJECT_VIEWER` database roles. Role **hierarchy does not count** here — Snowsight does not evaluate inherited grants for this feature, the grant must be direct.

## 1.15 Troubleshooting Patterns (Scenario-Question Goldmine)

| Scenario | Root cause | Fix |
|---|---|---|
| Trying to apply a policy to a feature that doesn't support it (e.g., a virtual column, a materialized view column mid-rewrite) | Unsupported target for `SET MASKING POLICY` | Apply the policy to the underlying source column instead |
| A role gets an "insufficient privileges" error trying to `CREATE OR REPLACE` a policy | Role lacks `CREATE MASKING POLICY` on the schema | Grant it, verify with `SHOW GRANTS TO ROLE`, retry |
| A role can't attach a policy to a table | Role lacks `APPLY MASKING POLICY` | Grant the privilege at account or policy level |
| Restoring a dropped table produces a policy error | The table pointed to a policy that no longer exists | Unset the (broken) policy reference, then reapply a valid policy |
| A column already has a policy and a second one is attempted | A column can only carry **one** masking policy at a time | Decide which policy should win, unset the old one first (or use `FORCE`) |
| An `ALTER ... IF EXISTS` on a policy "succeeds" but nothing changes | `IF EXISTS` silently no-ops if the policy name doesn't match | Drop `IF EXISTS`, rerun, and let the real error surface |
| `CREATE OR REPLACE` fails with a type-mismatch complaint | The `CASE` expression's return type doesn't match the declared `RETURNS` type | Align the types and reissue |
| A cloned table's owner can't unset its inherited policy | The cloned owner role lacks `APPLY` on that policy | Grant `APPLY` to the cloned table's owning role |

## 1.16 Quick Comparison Recap

| | Secure Views | Masking Policy |
|---|---|---|
| Scales to 1000s of columns from one definition | ❌ (one view per audience) | ✔ |
| True separation of duties (object owner ≠ policy controller) | ❌ | ✔ |
| Data at rest | Unaffected | Unaffected (except CTAS, §1.11.4) |

| | Dynamic Data Masking | External Tokenization |
|---|---|---|
| Data ever exists as plaintext inside Snowflake | Yes (until masked at query time) | **Never** |
| Works with Secure Data Sharing | ✔ | ❌ |
| Needs a third-party provider | ❌ | ✔ |

## 1.17 Common Misconceptions — Rapid Fire

- 🧠 *"OWNERSHIP always means full data visibility."* → False by design (§1.3).
- 🧠 *"Masking modifies the underlying stored data."* → False, except CTAS (§1.11.4).
- 🧠 *"If I avoid SELECTing the masked column, I dodge masking."* → False — masking applies wherever the column is *referenced*, including WHERE/JOIN/GROUP BY/ORDER BY (§1.7).
- 🧠 *"A masking policy can protect a column of any data type as long as the logic makes sense."* → False — input type, output type, and column type must all match exactly.
- 🧠 *"Unsetting then resetting a policy is effectively instantaneous/atomic."* → False unless you use `FORCE` — the two-statement approach has a real, exploitable data-exposure window.
- 🧠 *"External Tokenization is just a fancier Dynamic Data Masking — pick whichever."* → False — they're architecturally different (third-party dependency, sharing-incompatible, "never plaintext" guarantee) with real trade-offs.
- 🧠 *"A masking policy and row access policy can both reference the same column for layered security."* → False — that specific column can only appear in one or the other (§1.9).

## 1.18 Revision Cheat-Sheet (Must-Know One-Liners)

- Column-Level Security = **Enterprise Edition+**; two features: **DDM** and **External Tokenization**.
- A masking policy = **1 data type + conditions + masking function(s)**, applied at **query time**, stored as a **schema-level object**.
- Masking applies **everywhere** the column is referenced — SELECT, JOIN, WHERE, GROUP BY, ORDER BY.
- Table policy always evaluates **before** view policy; view chains apply **left to right**.
- A column **cannot** be governed by both a masking policy and a row access policy at once.
- **`FORCE`** swaps a policy atomically but risks sudden access loss; **unset-then-set** avoids that but opens a leak window.
- `ALTER MASKING POLICY` (editing the body) never requires unsetting — the column stays protected throughout.
- **Materialized views + masking = pick one layer.** Can't add masking to a base table once an MV exists on it; can't include already-masked columns when building an MV. Use a **Dynamic Table** if you need both.
- **CTAS bakes the masked value into the new table's storage** — the one real exception to "masking never touches data at rest."
- **External Tokenization cannot be used with Secure Data Sharing** — DDM can.
- Column-level tag-based masking loses to a **direct column-level masking policy** if both exist.
- Cloned tables re-point to cloned policies **only** for same-schema references; foreign-schema policy references stay pointed at the **original**.
- `APPLY` = attach/detach only. `OWNERSHIP` = control the policy's logic. Only one role holds `OWNERSHIP` at a time.
- Snowsight's governance dashboard needs **direct** grants of `GOVERNANCE_VIEWER` + `OBJECT_VIEWER` — hierarchy doesn't count.

## 1.19 Practice Questions

**Q1.** A role holds the `OWNERSHIP` privilege on a table with a masking policy on one column. The policy's logic does not mention this role at all. What does this role see when it queries that column?

A. The role sees masked data, because masking policy visibility is controlled by policy conditions, not by object ownership
B. The role sees the real value because `OWNERSHIP` always grants full data visibility, overriding any policy
C. The role sees the real value only if it is also the masking policy owner
D. The role sees a `NULL` value instead of either the real or masked value

<details><summary>Answer</summary>

**A.** Ownership of the object and visibility under a masking policy are completely separate concerns — this separation of duties is the entire point of masking policies (§1.3).
</details>

---

**Q2.** A user runs `SELECT email FROM customers WHERE email = 'jane@example.com'`, and this user's role sees a fixed masked string for the `email` column. What is the most likely result?

A. The query returns Jane's row normally, because WHERE clause comparisons are evaluated before masking is applied
B. The query returns zero rows, since Snowflake rewrites both sides of the comparison to the masked value
C. Snowflake blocks the query with a compilation error, since masked columns cannot appear in a WHERE clause
D. The query returns every row in the table, because the masking policy disables filtering on that column entirely

<details><summary>Answer</summary>

**B.** Snowflake rewrites the query so *both* the projected column and the WHERE-clause reference get wrapped in the masking expression. The masked output essentially never matches the literal comparison string, so the query silently returns zero rows (§1.7).
</details>

---

**Q3. True or False:** When a masking policy is attached to a column via `CREATE TABLE`, the underlying data stored in Snowflake's micro-partitions is physically rewritten into its masked form.

<details><summary>Answer</summary>

**False.** Masking is applied at query time only; the physical storage keeps the original plaintext value. (The one documented exception is CTAS output, §1.11.4 — the *new* table's storage does get the masked value baked in during that specific operation.)
</details>

---

**Q4.** Which capability is unique to External Tokenization and **not** available with Dynamic Data Masking?

A. Applying a single policy definition to thousands of columns across many schemas
B. Supporting centralized or decentralized administration of who can create versus apply policies
C. Being usable in combination with Secure Data Sharing to protect a consumer's view of the data
D. Ensuring sensitive values are never present in plaintext form at any point after Snowflake ingestion

<details><summary>Answer</summary>

**D.** This is External Tokenization's defining trait — data is tokenized *before* it ever lands in Snowflake. (C is actually reversed: DDM works with Secure Data Sharing; External Tokenization explicitly does **not**, §1.11.10.)
</details>

---

**Q5.** A materialized view already exists, built from a base table. A data engineer now tries to add a brand-new masking policy to a column on that same base table for the first time. What happens?

A. The masking policy is applied successfully and instantly propagates to the materialized view as well
B. Snowflake applies the policy to the base table but silently drops the corresponding column from the materialized view
C. Snowflake rejects the operation — a masking policy cannot be added to a base table column once a materialized view already exists on that table
D. Snowflake applies the policy only during the materialized view's next scheduled refresh cycle

<details><summary>Answer</summary>

**C.** Once an MV exists on a table, that table's columns are locked against new masking policies until the MV is dropped (§1.11.1).
</details>

---

**Q6.** A governance admin must swap masking policy `p1` for `p2` on column `c1` of table `t1`, with **zero tolerance** for even a brief moment where `c1` is completely unprotected. Which approach satisfies this?

A. Run `ALTER TABLE t1 MODIFY COLUMN c1 SET MASKING POLICY p2 FORCE` in a single statement
B. Run `UNSET MASKING POLICY`, then immediately run a second statement to `SET p2`
C. Drop `p1` entirely and recreate it under the name `p2` using `CREATE OR REPLACE MASKING POLICY`
D. Grant the `APPLY` privilege on `p2` to every role that previously had `APPLY` on `p1`

<details><summary>Answer</summary>

**A.** `FORCE` performs the swap in one atomic statement, eliminating the unprotected window that the two-statement unset/set approach creates (§1.5).
</details>

---

**Q7.** A custom role can successfully attach and detach an existing masking policy on columns it manages, but cannot modify the logic inside that policy's `CASE` expression. Which privilege does this role most likely hold?

A. `OWNERSHIP`
B. `APPLY`
C. `USAGE`
D. `CREATE MASKING POLICY`

<details><summary>Answer</summary>

**B.** `APPLY` grants the ability to set/unset a policy on a column but not to alter the policy's own definition — that requires `OWNERSHIP` (§1.13).
</details>

---

**Q8.** A team wants a single column, `home_address`, governed by **both** a row access policy and a masking policy directly on that same column at the same time. What does Snowflake require?

A. Nothing special — a column can be referenced in both signatures simultaneously
B. The two policies must be combined into a single `CREATE MASKING POLICY` statement using a `USING` clause
C. This isn't possible — the same column cannot be specified in both a row access policy signature and a masking policy signature at the same time
D. It's possible only if the row access policy and masking policy are owned by the same role

<details><summary>Answer</summary>

**C.** This is a hard architectural rule, not a permissions issue — the same column reference can't appear in both policy types at once (§1.9).
</details>

---

## ⏭️ Coming Next: Topic 2 — Row Access Policies

Masking Policies control **which values** in a column a role sees. Row Access Policies control **which entire rows** a role sees in the first place — and, as you now know from §1.9, they're mutually exclusive on any single column. Once we cover Row Access Policies, we'll build the full side-by-side comparison table (evaluation order when both exist on a table, performance considerations, and when to use one vs. the other vs. both together on different columns).

Say the word when you're ready to continue.
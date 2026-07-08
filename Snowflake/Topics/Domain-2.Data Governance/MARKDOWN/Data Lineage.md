# Data Lineage — SnowPro Core Notes

> Part of the Snowflake Horizon **Data Governance** family, alongside Object Tagging, Masking Policies, and Row Access Policies. Where tagging/masking *protect* data, lineage helps you *see the flow of* data — and the two work together (tags and masking status show up directly inside the lineage graph).

---

## Chapter 1: What Is Data Lineage, and Why Does It Exist?

**The idea in plain English:**
Data lineage is Snowflake's "family tree" for your data. For any table, view, or other object, it answers two questions:
- Where did this data *come from*? (upstream)
- Where does this data *go to*? (downstream)

**Two distinct kinds of relationship — this distinction is exam-critical:**

| Relationship type | What happens | Example |
|---|---|---|
| **Data movement** | Data is physically copied or materialized into a new object | `CTAS`, `INSERT ... SELECT`, `MERGE` |
| **Object dependency** | An object *references* another but does not copy/store its data | A view selecting from a table |

Both count as "lineage," but they are conceptually different — one moves bytes, the other just points at bytes.

**Why Snowflake built this (the business reasoning):**
- **Impact analysis** — "If I drop/alter this table, what breaks downstream?"
- **Troubleshooting** — trace bad data back to its source.
- **Compliance** — prove where sensitive data travelled (a regulator's favorite question).
- **Governance workflow** — surfaces where tags/masking policies are missing.
- **Trust** — analysts can verify a number actually traces back to a legitimate source.
- **Delegated administration** — you can let specific roles view lineage without giving them full data access.

**Edge Case (Edition requirement):**
Data Lineage requires **Enterprise Edition or higher**. This is a recurring SnowPro pattern — most Horizon governance features (tagging, masking, row access policies, lineage, Trust Center) are Enterprise+ features, not available on Standard.

**Exam Trap:** A question describing "a view built on a table, no data copied" — is that lineage? **Yes** — it's an *object dependency* relationship, still valid lineage, just not *data movement*. Don't assume lineage = only copy operations.

---

## Chapter 2: Upstream and Downstream

**Concept:** The **source** object is *upstream*. The **target** object is *downstream*.

```
CREATE TABLE table2 AS SELECT col1 FROM table1;
```
- `table1` = upstream (source)
- `table2` = downstream (target)
- Even the individual column `col1` has its own downstream lineage into `table2` — lineage tracks at the **column level**, not just the object level.

**Internal behavior / UI behavior:** In Snowsight, viewing `table1` shows an arrow pointing *toward* `table2` (downstream). Viewing `table2` shows an arrow pointing *back toward* `table1` (upstream).

**Gotcha:** Snowsight does **not** dump the entire lineage graph at once. It reveals the graph **incrementally, one hop at a time**, in whichever direction (upstream/downstream) you expand. This is a deliberate UX/performance decision — good to know if a question describes "the graph only shows part of the picture at first."

---

## Chapter 3: Lineage Node Grouping

**Concept:** As graphs grow large, Snowsight auto-organizes nodes into collapsible groups:
- **Snowflake objects** → grouped by database, then schema.
- **External objects** → grouped by vendor (e.g., all dbt-sourced nodes together).
- **Objects you lack access to** → placed in their own group, **collapsed by default**.

**Gotcha:** Large groups get **paginated**. If a node is hidden inside a collapsed/paginated group, the edges (arrows) still connect — just to the *group header*, not the individual node. Don't assume a missing individual node means broken lineage.

---

## Chapter 4: Column Lineage

**Concept:** You can trace lineage at the *column* level, not just the object level — e.g., "everywhere `customer.email` ends up, across every downstream table/view."

**Key mechanic — Distance:**
- Distance = 1 → the column lives in an object created *directly* from your starting object.
- Distance = 2 → the column lives two hops away (an object built from an object built from your starting point).

**Edge Case (must memorize):** Column lineage is **NOT supported for Semantic Views**. Object-level lineage works for semantic views, but you cannot drill into column-level detail for them.

---

## Chapter 5: Tags Inside the Lineage Graph

**Why this exists:** Previously (Object Tagging topic), we learned tags can be applied to columns to classify sensitive data (PII, PCI, etc.), and tags can **inherit** down an object hierarchy. Lineage closes the loop: it shows you *where tags are missing* along the actual data flow, not just the hierarchy.

**Concept:**
- The side panel for any object/column shows tags directly — no need to jump to a separate screen.
- Snowflake compares tags across upstream/downstream columns that share lineage and flags mismatches:
  - **Dashed border** on a tag = the column is **missing** that tag.
  - **Yellow border** on a tag = the tag exists but the **value doesn't match** upstream/downstream.
- You can then **Review and Apply** to propagate the correct tag/value in one action.

**Real-world example:** `raw.customers.ssn` is tagged `PII = HIGH`. A downstream view `analytics.customer_summary.ssn_masked` was created later and never got tagged. Lineage will flag this gap so the same classification (and by extension, masking policy) can be applied downstream — closing a compliance hole that's easy to miss manually.

**Exam Trap:** Whether you can *see or apply* tags in the Lineage tab depends on your role's **tagging privileges** — lineage viewing access (`VIEW LINEAGE`) alone is not enough to manage tags.

---

## Chapter 6: Masking Policies Inside the Lineage Graph

**Concept:** A symbol next to a column in the side panel indicates it's protected by a masking policy. Hovering shows the policy name/details.

**Edge Case:** If a column has a **conflict** — e.g., multiple masking policies somehow assigned to the same column — the UI shows **"Policy Error"** instead of the mask icon. This is a direct pointer toward the "Tag and policy discovery / troubleshoot tag-based masking" workflows from the Masking Policy topic. Lineage is essentially a *diagnostic dashboard* for masking policy conflicts.

---

## Chapter 7: Lineage Created by Stored Procedures or Tasks

**Concept:** If a stored procedure or task caused a downstream object to be created/populated, clicking the connecting arrow reveals:
- **Direct** — the stored procedure that directly caused it.
- **Root** — if that procedure was itself called by another (nested procedures), this shows the top-level procedure in the chain.

**Edge Cases (frequently tested):**
1. If the stored procedure was called **anonymously** (`CALL ... WITH` anonymous block), **no procedure details appear** in lineage — the data movement is tracked, but the "who caused it" metadata is lost.
2. Stored procedure/task details are **not backfilled**. If the lineage relationship existed before Snowflake added support for tracking procedures/tasks, you'll see the object lineage but **not** the procedure/task attribution.
3. You need **privileges on the stored procedure/task itself** to view these details — having lineage access to the objects isn't sufficient.

---

## Chapter 8: Retrieving Lineage Programmatically

**Concept:** `GET_LINEAGE (SNOWFLAKE.CORE)` is a SQL function that returns lineage data — but it's a **subset** of what the Snowsight UI shows.

**Edge Case:** You **cannot** use `GET_LINEAGE` to retrieve lineage information tied to a **stored procedure**. (Tested again later under Limitations — Snowflake documentation repeats this because it's commonly missed.)

---

## Chapter 9: Which Operations Create Lineage?

These SQL operations **create** an upstream→downstream relationship:

| Operation | Creates lineage? |
|---|---|
| `COPY INTO` | ✅ |
| `CREATE TABLE ... AS SELECT` (CTAS) | ✅ |
| `CREATE TABLE ... CLONE` | ✅ |
| `CREATE VIEW` | ✅ |
| `CREATE MATERIALIZED VIEW` | ✅ |
| `CREATE SEMANTIC VIEW` | ✅ |
| `INSERT ... SELECT ...` | ✅ |
| `MERGE` | ✅ |
| `UPDATE ... FROM another_table` | ✅ |

**Exam Trap (high-value):** A plain `INSERT INTO table VALUES (...)` with literal values does **not** create lineage — there's no source *object* for data to have moved from. Same logic for a plain `UPDATE table SET col = 'literal'` — no source table involved, so no lineage edge. The operation must reference another table-like object as the data's origin.

---

## Chapter 10: Supported Objects

**Table-like objects** (all fully supported for both object-level and column-level lineage, except where noted):
- Tables
- Dynamic tables
- External tables
- Iceberg tables
- Views
- Materialized views
- Semantic views *(object-level only — no column lineage, see Ch. 4)*

**Also participate in lineage:**
- Stages
- ML objects: Datasets, Feature Views (technically dynamic tables/views under the hood), Models

**Note:** ML Lineage is technically a distinct, specialized feature — it focuses on *how data is transformed through ML workflows* (dataset → feature view → model) rather than general movement/dependency tracking, but it uses the same underlying object types.

---

## Chapter 11: External Lineage — Bringing Outside Tools Into the Picture

**Connect to previous chapter:** Everything so far tracked lineage *inside* Snowflake. But real pipelines usually involve dbt, Airflow, or other ETL tools transforming data *outside* Snowflake before/after it touches Snowflake. Without External Lineage, your lineage graph would have invisible gaps wherever an external tool touched the data.

**Concept:** External Lineage extends the native lineage graph to include external sources/destinations, using **OpenLineage** — an open, vendor-neutral standard for sharing lineage metadata. Snowflake exposes a REST endpoint that accepts OpenLineage-formatted events; tools like dbt and Airflow can push events to it.

**Status:** Preview Feature (Open), available to Enterprise Edition+ accounts.

**Endpoint:**
```
POST https://<account_identifier>.snowflakecomputing.com/api/v2/lineage/external-lineage
```

**Internal Workflow:**
```
Data tool (dbt / Airflow)
        │  (runs a job, e.g., dbt run)
        ▼
OpenLineage event generated (COMPLETE event)
        │
        ▼
Sent via REST POST to Snowflake's external-lineage endpoint
        │
        ▼
Snowflake authenticates the request + checks INGEST LINEAGE privilege
        │
        ▼
Event merged into native lineage graph
        │
        ▼
External object appears in Snowsight, labeled as an "external node"
```

**Setup workflow (4 steps):**
1. Grant the **`INGEST LINEAGE`** privilege on the account to a role, then grant that role to the service user (e.g., the dbt/Airflow service account).
2. Configure the data tool's OpenLineage transport settings (base URL + endpoint + auth token).
3. Choose an authentication method supported by Snowflake REST APIs (e.g., key-pair JWT), generate a token tied to a specific user.
4. Run the tool normally — events flow automatically (for dbt, you swap `dbt run` → `dbt-ol run`).

**Real-world example:** A dbt job transforms data in a Postgres staging database, then loads the result into a Snowflake table. Without external lineage, Snowsight only shows the Snowflake table appearing "from nowhere." With external lineage configured, the graph shows the Postgres object as an upstream external node feeding into the Snowflake table — full pipeline visibility.

### Payload Rules (heavily testable edge cases)
- Must conform to the OpenLineage spec, and **only `COMPLETE` events are accepted** — any other `eventType` is silently ignored.
- Required fields: `inputs`, `outputs`, `eventType`, `eventTime`, `job`. (`run` is optional, but useful for identifying a specific job execution.)
- **Critical rule:** `inputs`/`outputs` must be a **mix** of one Snowflake object and one external object. You **cannot** use this endpoint to link two external objects together, and you **cannot** use it to link two Snowflake objects together (that's what native lineage already does). If both sides are the same type → the request fails with **HTTP 404**.
- The `facets` field lets you label the object type (e.g., `VIEW`). If you omit it, the object defaults to type **"External Node."**
- If a payload lists **multiple inputs** for one output, the lineage graph shows the output as downstream of **all** of those inputs (fan-in relationship).

### Removing Lineage
- `DELETE` requests to the same endpoint remove established links.
- You can scope the delete three ways: (1) a specific source↔target pair, (2) a source and *all* its downstream links, or (3) a target and *all* its upstream links.
- Requires the **`DELETE LINEAGE`** privilege on the account.

### Limitations (exam-favorite list)
- A Snowflake object must be **either** the input **or** the output of every event — external-to-external lineage isn't supported at all.
- **OpenLineage version 2 is NOT supported** — only version 1 events work.
- Retention: **1 year**.
- Only `COMPLETE` events are recognized; everything else is dropped.
- External lineage results **do not** appear in `GET_LINEAGE` function output — UI only.
- **No column-level lineage** for external objects — object level only.
- Fully qualified dataset name limited to **1000 characters**.
- Hard cap of **10,000 events per account** — once hit, you must delete old events before adding new ones.

---

## Chapter 12: Access Control for (Native) Lineage

**Concept:** Viewing lineage requires a combination of privileges — it's not automatically tied to data-access privileges alone.

To view an object's lineage, a role needs **all** of:
1. **`VIEW LINEAGE`** privilege on the account.
2. Some privilege on the specific object — e.g., `SELECT`. If you want someone to see *that lineage exists* without letting them read the actual data, grant **`REFERENCES`** instead of `SELECT`.
3. **`USAGE`** on the database and schema containing the object.

**Big gotcha:** `VIEW LINEAGE` is granted to **`PUBLIC` by default** — meaning **everyone can view lineage out of the box**. If you want to restrict this, you must explicitly **revoke** it from `PUBLIC` and grant it to specific custom roles instead.

**Shortcut privilege:** `RESOLVE ALL` on the account lets a role see the **full** lineage graph of *every* object — even ones it has zero access to otherwise. But note: the role **still** needs `VIEW LINEAGE` too — `RESOLVE ALL` alone isn't sufficient.

**What happens with insufficient privileges:** The object is shown **greyed out** with an "insufficient privileges" message. Important nuance: **this does NOT mean it's a dead end / terminal node in the graph** — it just means *you* can't see any further in that direction. There might be more lineage beyond it that a different role could see. Same greyed-out behavior applies to objects/columns restricted by other access policies (e.g., row access policies).

**One more permission wrinkle:** To see the actual **SQL statement** behind a lineage edge (the "how was this created" detail), the viewing user also needs access to the **`QUERY_HISTORY`** account-usage view.

---

## Chapter 13: History and Retention

- Data Lineage launched **November 2024**.
- **Object dependency** lineage (e.g., "this view reads from this table") from *before* that date **IS still available** — because dependency can be derived from existing object metadata retroactively.
- **Data movement** lineage (e.g., "this table was created via CTAS from that table") from *before* that date is **NOT available** — because Snowflake didn't capture the movement event itself at the time.
- Both **column lineage** and **object lineage** are retained for **1 year**.

**Exam Trap:** This is an *asymmetric backfill* — dependency lineage got backfilled, movement lineage did not. A question might ask "why can I see a view's source table from 2023 but not a CTAS table's source from 2023?" — this is exactly why.

---

## Chapter 14: Limitations and Considerations (Native Lineage)

This is the single richest source of exam gotchas in the whole topic — go through each carefully.

**No lineage available for:**
- Objects inside a **shared database** (i.e., data shared to you via Secure Data Sharing).
- Objects inside the **`SNOWFLAKE`** shared system database.
- Objects inside any database's **`INFORMATION_SCHEMA`**.
- **Semantic views created before early February 2026.**

**Dynamic tables:** They **appear** in the lineage graph of *other* objects (as an upstream/downstream node), but a dynamic table does **not have its own Lineage tab**. You can see it referenced, but can't "start" a lineage exploration from it directly.

**Object rename vs. delete:**
- **Deleted** tables disappear from the lineage graph entirely.
- **Renamed** tables continue to appear — Snowflake tracks the underlying object identity, not just the name.

**Temporary tables:** Never shown in the lineage graph, period — regardless of what operations touch them.

**"Touched but not moved" rule — filter/join tables don't count:**
```sql
CREATE TABLE target_table AS
  SELECT t1.c1, t1.c2
  FROM t1, t2
  WHERE t1.c3 = t2.c3;
```
Here, `t2` is used only to **filter/join**, but no column *from* `t2` ends up *in* `target_table`. So `t2` is **NOT** considered part of the lineage of `target_table` — only `t1` is. This is a subtle but very testable rule: lineage tracks where the *selected data itself* came from, not everything referenced in the query.

**Disjointed/multi-statement movement is NOT tracked:**
```sql
SET read_output1 = (SELECT c1 FROM sourceTable1);
INSERT INTO target_table(c1) VALUES ($read_output1);
```
Even though data logically flowed from `sourceTable1` to `target_table`, Snowflake **cannot** connect these two separate, independent statements into one lineage edge. This limitation also applies when a **stored procedure** internally does this kind of separated read-then-write — the lineage engine only tracks movement within a single traceable statement/operation, not arbitrary application logic.

**`GET_LINEAGE` limitation (repeated for emphasis):** Cannot be used to obtain lineage info tied to a stored procedure.

---

## Quick Comparison: Native Lineage vs. External Lineage

| Aspect | Native Lineage | External Lineage |
|---|---|---|
| Scope | Objects entirely inside Snowflake | Bridges Snowflake objects ↔ external tools/systems |
| Standard used | Snowflake's own internal tracking | OpenLineage (open standard) |
| Trigger | Automatic — Snowflake watches DML/DDL | Requires explicit event push from external tool via REST API |
| Column-level detail | Supported (except semantic views) | **Not supported** — object level only |
| Retrieval via function | `GET_LINEAGE` (subset of UI data) | **Not available** via `GET_LINEAGE` — UI only |
| Privilege to contribute | N/A (automatic) | `INGEST LINEAGE` on account |
| Privilege to remove | N/A | `DELETE LINEAGE` on account |
| Retention | 1 year | 1 year |
| Edition | Enterprise+ | Enterprise+ (Preview) |

---

## Summary — The 10 Things to Remember

1. Lineage = tracks **data movement** (copy) and **object dependency** (reference) — two different relationship types, both valid.
2. Requires **Enterprise Edition or higher**.
3. Source = **upstream**, target = **downstream**; Snowsight reveals the graph **one hop at a time**.
4. Column lineage works for all table-like objects **except semantic views**.
5. Lineage view integrates **tags** (flags missing/mismatched tags) and **masking policies** (flags policy conflicts as "Policy Error").
6. Viewing lineage needs **`VIEW LINEAGE`** (default granted to `PUBLIC`!) **+** some object privilege (or just `REFERENCES`) **+** `USAGE` on db/schema. `RESOLVE ALL` bypasses per-object privilege checks but still needs `VIEW LINEAGE`.
7. Only **movement-type operations** create lineage — plain `INSERT ... VALUES` or literal `UPDATE`s do **not**.
8. Filter/join-only tables and **disjointed multi-statement** data flows are **not** tracked as lineage.
9. History: object *dependency* lineage was backfilled before Nov 2024 launch; object *movement* lineage was **not**.
10. External Lineage uses **OpenLineage**, only accepts **`COMPLETE`** events, requires a **mix** of one Snowflake + one external object per event, and caps at **10,000 events/account**.

---

## Exam Traps Cheat-Sheet

| Trap | Correct Answer |
|---|---|
| "Does a view count as lineage even though no data is copied?" | Yes — it's an object *dependency* relationship. |
| "Can PUBLIC view lineage by default?" | Yes — `VIEW LINEAGE` is granted to PUBLIC by default; must be revoked to restrict. |
| "Does `INSERT INTO t VALUES (1,2,3)` create lineage?" | No — no source object referenced. |
| "Is column lineage available for semantic views?" | No — object-level only. |
| "Can `GET_LINEAGE` show stored procedure lineage?" | No — explicitly unsupported. |
| "Does a greyed-out node mean lineage ends there?" | No — it just means *you* lack privilege to see further; more may exist. |
| "Can external lineage link two external systems together?" | No — must always include at least one Snowflake object. |
| "Does OpenLineage v2 work with Snowflake external lineage?" | No — only v1 is supported. |
| "Is a table used only in a WHERE join clause part of lineage?" | No — only tables whose data actually lands in the target count. |
| "Is data movement across two separate/unrelated SQL statements tracked?" | No — must happen within one traceable statement/operation. |

---

## Practice Questions

**Q1 (Conceptual).** Which of the following creates a data lineage relationship?
A) `INSERT INTO orders VALUES (101, 'shipped')`
B) `CREATE VIEW v1 AS SELECT * FROM orders`
C) `SELECT * FROM orders WHERE id = 101`
D) `SHOW TABLES`

*Answer: B.* A CREATE VIEW establishes an object-dependency relationship. A is a literal insert (no source object). C and D don't create or modify objects at all.

**Q2 (Scenario).** A role has `SELECT` on `sales.orders` and `VIEW LINEAGE` on the account, but no privileges on `sales.customers`, which is two hops upstream of `orders`. What does the role see when viewing `orders`' lineage and expanding upstream toward `customers`?

*Answer:* The role can see `orders` and one hop upstream normally, but `customers` (or whatever intermediate object it lacks privilege on) will appear **greyed out** with an insufficient-privileges message — not omitted entirely, and not implying lineage stops there.

**Q3 (True/False).** External Lineage events of type `RUNNING` or `START` will be reflected in the Snowsight lineage graph.

*Answer: False.* Only `COMPLETE` events are accepted; all others are ignored.

**Q4 (Trick wording).** "A dynamic table shows up when you view another table's lineage, so you can also open the dynamic table's own Lineage tab directly." True or False?

*Answer: False.* Dynamic tables appear as nodes in *other* objects' lineage graphs, but they don't have their own Lineage tab.

**Q5 (Scenario).** A payload is sent to the external lineage endpoint with `inputs` = a Postgres table and `outputs` = a Snowflake table, `eventType` = `COMPLETE`, and no `facets` field on the output. What object type will Snowflake assign to the Postgres input node in the graph, and would this request succeed?

*Answer:* The request succeeds (one Snowflake + one external object = valid mix). Since no `facets` field is provided, the object type defaults to **"External Node."**

---

*Next topic in this series: (your choice) — Object Tagging & Inheritance, Masking Policies, Row Access Policies, Trust Center, or Account Replication/Failover. Let me know which one to cover next.*
# Object Tagging — SnowPro Core Notes

*Chapter 1 of your governance series (next up: Masking Policies → Row Access Policies → Trust Center → Replication/Failover → Lineage)*

---

## 1. Where this fits in the bigger picture

Snowflake's governance toolkit answers three different questions:

- **"What is this data?"** → Object Tagging (this chapter) — metadata
- **"What can this role SEE in a column?"** → Masking Policies — value-level
- **"What can this role SEE as a row?"** → Row Access Policies — row-level

Tagging is the *foundation* the other two sit on top of, because a tag can literally **trigger** a masking policy automatically. So this is the right place to start.

---

## 2. What is a tag? (Simple English)

A **tag** is basically a labeled sticky note that Snowflake lets you stick onto almost anything — a table, a column, a warehouse, a user, even a policy. The sticky note has a name and a value, written as `key = value`.

Example: you stick a note that says `cost_center = sales` onto a warehouse. Now Snowflake "knows" that warehouse belongs to Sales, and you can ask Snowflake "show me everything tagged `sales`" at any time.

Technically:
- A tag is a **schema-level object** — it lives inside a database/schema, just like a table does.
- The tag **name** must be unique within its schema.
- The tag **value** is always a string, up to **256 characters**.
- One tag can be stuck on many different objects, and those objects don't even have to be the same type — the same `cost_center` tag can sit on a warehouse and a table at the same time.
- The value doesn't have to be unique either — ten tables can all carry `cost_center = sales`, or each can carry a different value. Snowflake doesn't enforce uniqueness on the value.

### Why this feature exists
Before tags, "which columns hold PII?" or "which warehouse belongs to which team?" lived in a spreadsheet or a wiki page somewhere — completely disconnected from the actual data, and guaranteed to go stale the moment someone adds a new table. Tags move that metadata **into Snowflake itself**, attached directly to the object, queryable with SQL, and — critically — usable as a **trigger** for automated governance (a masking policy can watch for a tag and act on it) instead of being just a passive comment.

---

## 3. How Snowflake handles a tag internally

```
You run:  ALTER TABLE orders MODIFY COLUMN ssn SET TAG pii_type = 'ssn'
                │
                ▼
   Snowflake checks your role's privileges:
   • Global APPLY TAG on the ACCOUNT?           ─┐
        OR                                        ├─ need ONE of these
   • APPLY on this specific tag                   │
     AND OWNERSHIP on the table            ───────┘
                │
                ▼ (privilege check passes)
   Snowflake stores a reference row:
   tag_name = pii_type, value = 'ssn',
   object = orders.ssn, apply_method = 'MANUAL'
                │
                ▼
   Reference becomes queryable via
   TAG_REFERENCES / ACCOUNT_USAGE.TAG_REFERENCES
                │
                ▼
   If a masking policy is attached to
   pii_type = 'ssn' → column is now
   auto-protected the next time it's queried
```

---

## 4. The 5 ways a tag can land on an object ⚠️ (exam favorite)

Most people assume "manually set" is the only way a tag appears on an object. It's actually **one of five**:

1. **Manual** — someone runs `CREATE <object>` or `ALTER <object> ... SET TAG`
2. **Inheritance** — the object's parent in the hierarchy has the tag (see §5)
3. **Automatic propagation** — Enterprise Edition feature, tag hops across a *dependency* (e.g., a view built on a tagged table) or *data movement* (e.g., a CTAS)
4. **Sensitive data classification** — Snowflake's auto-classifier detects PII-looking data and tags the column for you
5. **Cloning** — `CREATE TABLE ... LIKE` or `CREATE TABLE ... CLONE` carries the source table's tags over to the new table

💡 **How do you tell which one happened?** Every tag reference has an `apply_method` column. Query it with:

```sql
SELECT tag_name, tag_value, apply_method, level, domain
FROM TABLE(my_db.INFORMATION_SCHEMA.TAG_REFERENCES('my_table', 'TABLE'));
```

Or at account scale: `ACCOUNT_USAGE.TAG_REFERENCES`, `ACCOUNT_USAGE.TAG_REFERENCES_WITH_LINEAGE`, `INFORMATION_SCHEMA.TAG_REFERENCES_ALL_COLUMNS`.

---

## 5. Inheritance vs. Propagation — the #1 trap on this topic

These two get confused constantly. They are **not the same mechanism**.

```
INHERITANCE  →  automatic, ALL editions, based on CONTAINMENT (the object literally lives inside another)

  ACCOUNT ──▶ DATABASE ──▶ SCHEMA ──▶ TABLE ──▶ COLUMN
   tag set here flows all the way down automatically,
   unless a descendant sets its OWN value (override)


PROPAGATION  →  opt-in only, ENTERPRISE EDITION, based on DEPENDENCY / DATA LINEAGE

  TABLE (tagged) ──▶ VIEW ──▶ MATERIALIZED VIEW
   NOT automatic — only happens if the tag was created
   or altered with PROPAGATE = ON_DEPENDENCY / ON_DATA_MOVEMENT / BOTH
```

**The exact trap Snowflake's own docs warn about:** if you have `table_1 → view_1 → materialized_view_1`, tagging `table_1` does **not** automatically tag `materialized_view_1`. A materialized view is not a "child" of the table in the containment sense — it's a *dependent* object. Inheritance only walks the containment tree (account/db/schema/table/column). To make tags hop across dependencies like this, you need propagation, explicitly configured on that tag, and Enterprise Edition.

### Overriding inheritance
You can always override an inherited value by manually setting the tag on the lower object directly. Example: a column inherits `cost_center = sales` from its table, but you set the column's own tag to `cost_center = sales_na` — the column now shows `sales_na`. More specific wins.

Two more overwrite rules straight from the docs:
- An **inherited** tag's value gets overwritten if **automatic propagation** later applies the tag to that same object.
- An **inherited** tag's value gets overwritten by **sensitive data classification**.

---

## 6. What can you actually tag? (breadth check)

Almost everything. A partial list, grouped by where it sits in the hierarchy:

| Level | Examples |
|---|---|
| Organization/Account | Account, User, Role, Warehouse, Database, Share, Network Policy, Integration, Compute Pool, Failover Group, Replication Group, Application/Application Package |
| Database | Schema, Database Role |
| Schema | Table, View, External Table, Materialized View, Dynamic Table, Iceberg Table, Stage, Pipe, Stream, Task, Procedure, Function/UDF, Alert, Git Repository, Notebook, Streamlit, Cortex Search Service, **and even other policies** (masking, row access, password, session, aggregation, join, projection) |
| Table/View | Column (incl. event table columns) |
| Semantic View | Logical table, dimension, fact, metric (very granular) |

⚠️ **Gotcha:** Tags set on a **Share** by the provider are **not visible to the consumer**. Don't assume tagging metadata travels with a share.

⚠️ **Gotcha:** You can tag a *policy itself* (e.g., tag a masking policy with `owner_team = finance`). Governance objects are themselves governable.

---

## 7. Working with tags — SQL you should know cold

```sql
-- 1. Create a basic tag (any edition)
CREATE TAG cost_center;

-- 2. Create a tag with a controlled vocabulary (max 5,000 allowed values, 256 chars each)
CREATE TAG cost_center
  ALLOWED_VALUES 'finance', 'engineering';

-- Add / remove allowed values later
ALTER TAG cost_center ADD ALLOWED_VALUES 'marketing';
ALTER TAG cost_center DROP ALLOWED_VALUES 'engineering';

-- Check what's allowed
SELECT SYSTEM$GET_TAG_ALLOWED_VALUES('governance.tags.cost_center');

-- 3. Create a self-propagating tag (Enterprise Edition only)
CREATE TAG data_sensitivity PROPAGATE = ON_DEPENDENCY;
ALTER TAG data_sensitivity SET PROPAGATE = ON_DEPENDENCY_AND_DATA_MOVEMENT;
ALTER TAG data_sensitivity UNSET PROPAGATE;

-- 4. Apply a tag while creating an object
CREATE WAREHOUSE mywarehouse WITH TAG (cost_center = 'sales');

-- 5. Apply a tag to something that already exists
ALTER WAREHOUSE wh1 SET TAG cost_center = 'sales';

-- 6. Apply a tag to a specific column
ALTER TABLE hr.tables.empl_info
  MODIFY COLUMN job_title
  SET TAG cost_center = 'marketing';

-- 7. Remove a tag from an object
ALTER TABLE t1 UNSET TAG cost_center;

-- 8. Delete a tag entirely (24-hour grace period!)
DROP TAG cost_center;
UNDROP TAG cost_center;   -- restores the tag AND every reference it had
```

💡 You can also tag table + columns in one shot at creation:
```sql
CREATE TABLE t1 (
  COL1 INT WITH TAG (tag1='col1', tag2='col1'),
  COL2 INT WITH TAG (tag1='col2')
)
WITH TAG (tag3='t1');
```

---

## 8. Privileges — the part people get wrong

Three privileges govern tags:

| Privilege | What it lets you do |
|---|---|
| **CREATE** (on a schema) | Create a new tag inside that schema |
| **APPLY** (on a tag) | Set/unset *that specific tag* on objects |
| **OWNERSHIP** (on a tag) | Full control — required to alter most tag properties (allowed values, propagation, comment), and (with schema USAGE) to drop/undrop it |

There's also a special **global** privilege: **`APPLY TAG` granted `ON ACCOUNT`**. This is different from `APPLY` on one tag — it lets a role set/unset **any** tag on **any** object account-wide.

### ⚠️ The trap: owning the object is NOT enough
To set/unset a tag on an object, a role needs **either**:
- Global `APPLY TAG` on the account, **or**
- `APPLY` on that specific tag **+ OWNERSHIP of the target object**

This means: **owning a table does not let you tag it**, unless you also have rights on the tag itself. This surprises a lot of people and is a very testable fact.

### Quick reference

| Operation | Privilege required |
|---|---|
| Create a tag | CREATE TAG on the schema |
| Create a tag that can propagate | `APPLY TAG` on account **+** OWNERSHIP on the tag (stricter!) |
| Alter a tag's properties | OWNERSHIP on the tag |
| Drop / Undrop a tag | OWNERSHIP on the tag + USAGE on its db & schema |
| Set/unset tag on an object | Global `APPLY TAG` on account, **or** `APPLY` on the tag + OWNERSHIP on the object |
| Set/unset tag on a column | Same as above, but ownership needed is on the table/view |
| `SHOW TAGS` | USAGE on the schema where tags live, **or** global `APPLY TAG` on account |

⚠️ **Gotcha:** **Future grants are not supported for tag privileges.** You cannot say "automatically grant APPLY on every future tag in this schema." Workaround: just grant the account-wide `APPLY TAG` privilege instead.

💡 To manage tags through the **Snowsight UI**, you additionally need Enterprise Edition and either the `ACCOUNTADMIN` role or a role granted both the `GOVERNANCE_VIEWER` and `OBJECT_VIEWER` database roles.

---

## 9. Tag quotas & limits (memorize these numbers)

| Limit | Value |
|---|---|
| Max tags on a single object (table, view, etc.) | **50** |
| Max *distinct* tags across all columns of one table (separate quota) | **50** |
| Max unique tag-entity associations in a single CREATE/ALTER statement | **100** |
| Max allowed values per tag (`ALLOWED_VALUES`) | **5,000** |
| Max length of a tag value | **256 characters** |
| Grace period after `DROP TAG` before permanent deletion | **24 hours** |

⚠️ **Worked gotcha (straight from Snowflake's own example):** a table has 1,000 columns and you want the same tag on every column. One statement can only create 100 unique tag-entity associations — so you need **at least 10 separate ALTER statements**, not one.

⚠️ **Worked gotcha #2:** the 50-tag column limit counts **distinct tag names**, not tag instances. If `tag1` is applied to both `COL1` and `COL2`, that's still just 1 toward the 50 — not 2.

---

## 10. Which edition do you actually need?

- **Creating and setting basic tags** → works on **every edition**, including Standard.
- **Automatic tag propagation** → requires **Enterprise Edition or higher**.
- **Tag-based masking policies** → requires **Enterprise Edition or higher**.

⚠️ **Trap:** Don't let an exam question trick you into thinking "tagging needs Enterprise Edition" as a blanket statement. Only the *advanced* capabilities (propagation, tag-based masking) do.

---

## 11. Real-world examples

**Enterprise example (compliance/PII):** A bank has 500+ tables across dozens of schemas. Instead of maintaining a spreadsheet of "which columns are sensitive" that goes stale immediately, the security team tags every PII column with `data_classification = 'PII'`. A masking policy is bound to that same tag. Result: the moment anyone tags a new column `PII`, it's automatically masked for non-privileged roles — no one has to remember to attach a policy manually, and auditors get a live, accurate inventory just by querying `TAG_REFERENCES`.

**Cost-attribution example:** A company runs 20 shared virtual warehouses. Finance tags each one with `cost_center = <team>`. A nightly job joins warehouse metering history with the tag value to produce a per-department chargeback report — without touching billing exports by hand.

---

## 12. Common misconceptions

- ❌ *"If I own the table, I can tag it."* → No — you also need APPLY rights on the tag (or the account-wide privilege).
- ❌ *"Tagging a table tags every view built from it."* → No — that's propagation (Enterprise, opt-in), not inheritance.
- ❌ *"Tags are only for tables and columns."* → No — nearly every securable object is taggable, including users, warehouses, shares, and even other policies.
- ❌ *"I have to untag every object before I can drop a tag."* → No — `DROP TAG` removes the tag and all its references in one go (with a 24-hour undo window).
- ❌ *"Basic tagging needs Enterprise Edition."* → No — only propagation and tag-based masking do.
- ❌ *"A column permanently inherits its table's tag value forever."* → No — you can override it with a more specific value at any time.

---

## 13. SnowPro exam traps to watch for

- A scenario describing `table → view → materialized_view` and asking whether the materialized view "automatically" gets the tag — answer is **no**, unless propagation was explicitly configured.
- A math question: "tag 1,000 columns with one tag, how many ALTER statements minimum?" → **10** (100-per-statement cap).
- A scenario where a table **owner** tries to tag their own object and it fails — testing whether you know ownership alone isn't sufficient.
- A "which edition" question that tries to make you say Enterprise for *all* tagging — only true for propagation/tag-based masking.
- "Can a data share consumer see tags the provider applied?" → **No.**
- True/False: "Tag values must be unique across objects." → **False.**
- Questions mixing up `APPLY` (per-tag privilege) with `APPLY TAG ... ON ACCOUNT` (global privilege) — read carefully whether "ON ACCOUNT" is present.

---

## 14. How Object Tagging compares to what's coming next

| Tool | What it controls | Granularity |
|---|---|---|
| **Object Tag** (this chapter) | Attaches searchable metadata; can *trigger* a masking policy | Any object, down to a column |
| **Masking Policy** (next chapter) | Transforms/hides column *values* at query time based on role | Column |
| **Row Access Policy** (after that) | Filters which *rows* are visible based on role/context | Row |

---

## 15. Quick self-check

1. **T/F** — Tagging a database means every table inside it shows that tag unless overridden.
   **True** — that's inheritance, flows down the containment hierarchy.

2. **T/F** — Tagging a table means every view built on it will carry the same tag.
   **False** — that's propagation, which is opt-in and Enterprise-only.

3. You own `customers` but have no tag privileges. Can you run `ALTER TABLE customers SET TAG dept='sales'`?
   **No** — ownership of the object alone doesn't grant tag rights.

4. You need to tag 500 columns on one table with the same tag. One statement?
   **No** — max 100 unique tag-entity associations per statement, so at least 5 statements.

5. Which edition do you need just to create and apply a tag? For tag-based masking?
   **Any edition** for basic create/apply; **Enterprise+** for tag-based masking (and for propagation).

6. You accidentally drop a tag. Gone forever?
   **No** — 24-hour grace period, `UNDROP TAG` restores it and every reference.

7. A provider tags a shared table `confidential`. Does the consumer see it?
   **No.**

8. **T/F** — A tag's value must be unique wherever it's applied.
   **False** — only the tag *name* must be unique within its schema; the value can repeat freely.

---

## Sources
- [Introduction to object tagging](https://docs.snowflake.com/en/user-guide/object-tagging/introduction)
- [Tag inheritance](https://docs.snowflake.com/en/user-guide/object-tagging/inheritance)
- [Work with object tags](https://docs.snowflake.com/en/user-guide/object-tagging/work)
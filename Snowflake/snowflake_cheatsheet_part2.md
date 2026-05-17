# 6. MANAGING USER ROLES AND PERMISSIONS

## 6.1 Role-Based Access Control (RBAC) Overview

Snowflake uses **RBAC** (Role-Based Access Control) as the primary access model, with **DAC** (Discretionary Access Control) layered on top (object owners can grant others access to their objects).

```text
ACCOUNTADMIN (top)
    ↓ inherits
SYSADMIN ←──────────── Custom Roles (ANALYST, DEVELOPER, etc.)
    ↓ inherits
USERADMIN
    ↓ inherits
SECURITYADMIN
    ↓ inherits
PUBLIC (bottom — all users have this by default)
```

---

## 6.2 System-Defined Roles

| Role | Capabilities | Who Should Use |
|------|-------------|----------------|
| `ACCOUNTADMIN` | Everything + billing + account config | 2-3 people max |
| `SYSADMIN` | Create/manage databases, warehouses, objects | Senior DBA/admin |
| `SECURITYADMIN` | Manage users, roles, network policies, grants | Security team |
| `USERADMIN` | Create/modify users and roles only | HR/provisioning |
| `PUBLIC` | Auto-granted to all users; minimal perms | Everyone |
| `ORGADMIN` | Manage organization-level settings | Org admins only |

> **Critical Best Practice:** Never use `ACCOUNTADMIN` for daily work. Create custom roles. Grant ACCOUNTADMIN to max 2-3 emergency accounts only.

---

## 6.3 Custom Role Hierarchy (Real-World Pattern)

```text
SYSADMIN
    └── PROD_ADMIN
            ├── ANALYST_ROLE       (SELECT on prod tables)
            ├── DEVELOPER_ROLE     (SELECT, INSERT, UPDATE on dev)
            └── ETL_ROLE           (USAGE on warehouse, INSERT on target tables)

SECURITYADMIN
    └── Creates all roles, grants role to roles
    └── Grants roles to users
```

```sql
-- Create custom role
CREATE ROLE ANALYST_ROLE COMMENT = 'Read-only analyst access';
CREATE ROLE DEVELOPER_ROLE;
CREATE ROLE ETL_ROLE;

-- Build role hierarchy
GRANT ROLE ANALYST_ROLE TO ROLE SYSADMIN;     -- so SYSADMIN can manage it
GRANT ROLE DEVELOPER_ROLE TO ROLE SYSADMIN;
GRANT ROLE ETL_ROLE TO ROLE SYSADMIN;

-- Grant role to user
GRANT ROLE ANALYST_ROLE TO USER alice;
GRANT ROLE DEVELOPER_ROLE TO USER bob;

-- Set user default role
ALTER USER alice SET DEFAULT_ROLE = ANALYST_ROLE;
```

---

## 6.4 Object Privileges — Complete Reference

### Privilege Hierarchy: Must grant in order

```text
To access a TABLE, user needs:
  USAGE on DATABASE
  USAGE on SCHEMA
  SELECT (or other) on TABLE
```

```sql
-- Grant database access
GRANT USAGE ON DATABASE PROD_DB TO ROLE ANALYST_ROLE;

-- Grant schema access
GRANT USAGE ON SCHEMA PROD_DB.SALES TO ROLE ANALYST_ROLE;

-- Grant table privileges
GRANT SELECT ON TABLE PROD_DB.SALES.ORDERS TO ROLE ANALYST_ROLE;
GRANT SELECT ON ALL TABLES IN SCHEMA PROD_DB.SALES TO ROLE ANALYST_ROLE;

-- Future grants — auto-grant to new objects
GRANT SELECT ON FUTURE TABLES IN SCHEMA PROD_DB.SALES TO ROLE ANALYST_ROLE;
GRANT SELECT ON FUTURE VIEWS IN SCHEMA PROD_DB.SALES TO ROLE ANALYST_ROLE;
GRANT ALL ON FUTURE TABLES IN DATABASE PROD_DB TO ROLE DEVELOPER_ROLE;

-- Warehouse access
GRANT USAGE ON WAREHOUSE COMPUTE_WH TO ROLE ANALYST_ROLE;
GRANT OPERATE ON WAREHOUSE COMPUTE_WH TO ROLE ANALYST_ROLE; -- start/stop
GRANT MONITOR ON WAREHOUSE COMPUTE_WH TO ROLE ANALYST_ROLE; -- see usage

-- Stage access
GRANT READ ON STAGE PROD_DB.PUBLIC.my_stage TO ROLE ETL_ROLE;
GRANT WRITE ON STAGE PROD_DB.PUBLIC.my_stage TO ROLE ETL_ROLE;
```

### All Privilege Types by Object

| Object | Privileges |
|--------|-----------|
| Account | MONITOR USAGE, OVERRIDE SHARE RESTRICTIONS |
| Database | USAGE, CREATE SCHEMA, IMPORT SHARE, MODIFY, MONITOR |
| Schema | USAGE, CREATE TABLE, CREATE VIEW, CREATE STAGE, MODIFY, MONITOR |
| Table | SELECT, INSERT, UPDATE, DELETE, TRUNCATE, REFERENCES |
| View | SELECT, REFERENCES |
| Warehouse | USAGE, OPERATE, MODIFY, MONITOR |
| Stage | READ, WRITE |
| File Format | USAGE |
| Function/Procedure | USAGE |
| Task | MONITOR, OPERATE |
| Stream | SELECT |
| Sequence | USAGE |
| Integration | USAGE |

---

## 6.5 Show, Revoke, and Manage Grants

```sql
-- Show grants TO a role (what a role can do)
SHOW GRANTS TO ROLE ANALYST_ROLE;

-- Show grants ON an object (who has access to this)
SHOW GRANTS ON TABLE PROD_DB.SALES.ORDERS;

-- Show grants TO a user (which roles)
SHOW GRANTS TO USER alice;

-- Show all role hierarchy
SHOW ROLES;

-- Revoke
REVOKE SELECT ON TABLE PROD_DB.SALES.ORDERS FROM ROLE ANALYST_ROLE;
REVOKE ROLE ANALYST_ROLE FROM USER alice;

-- WITH GRANT OPTION — allows the grantee to further grant this privilege
GRANT SELECT ON TABLE orders TO ROLE ANALYST_ROLE WITH GRANT OPTION;
```

---

## 6.6 Ownership and Object Ownership Transfer

```sql
-- Check current owner
SHOW TABLES LIKE 'ORDERS';   -- look at the OWNER column

-- Transfer ownership (must be done by current owner or SECURITYADMIN)
GRANT OWNERSHIP ON TABLE PROD_DB.SALES.ORDERS 
  TO ROLE DEVELOPER_ROLE 
  REVOKE CURRENT GRANTS;   -- revokes existing grants from old owner
  
-- Or COPY current grants
GRANT OWNERSHIP ON TABLE PROD_DB.SALES.ORDERS 
  TO ROLE DEVELOPER_ROLE 
  COPY CURRENT GRANTS;
```

> **Exam:** Only the **object owner** or a role with **MANAGE GRANTS** privilege can transfer ownership. `ACCOUNTADMIN` can always transfer.

---

## 6.7 Role Switching

```sql
-- Switch role in session
USE ROLE ANALYST_ROLE;

-- Check all roles available to current user
SHOW ROLES;

-- In Python connector
conn = connect(user='alice', ..., role='ANALYST_ROLE')
```

---

## 6.8 Common RBAC Exam Scenarios

**Scenario 1:** Alice needs to query `PROD_DB.SALES.ORDERS` but gets "object does not exist or access denied."

**Debug Checklist:**
```sql
-- Check 1: Does alice have USAGE on database?
SHOW GRANTS TO ROLE ANALYST_ROLE;  -- look for PROD_DB USAGE

-- Check 2: USAGE on schema?
-- Check 3: SELECT on the table?
-- Check 4: Active role correct?
SELECT CURRENT_ROLE();

-- Fix
GRANT USAGE ON DATABASE PROD_DB TO ROLE ANALYST_ROLE;
GRANT USAGE ON SCHEMA PROD_DB.SALES TO ROLE ANALYST_ROLE;
GRANT SELECT ON TABLE PROD_DB.SALES.ORDERS TO ROLE ANALYST_ROLE;
```

**Scenario 2:** New tables are added to SALES schema monthly. Analyst keeps losing access.

**Fix:**
```sql
-- Use FUTURE grants
GRANT SELECT ON FUTURE TABLES IN SCHEMA PROD_DB.SALES TO ROLE ANALYST_ROLE;
```

---

# 7. DATA GOVERNANCE AND PROTECTION

## 7.1 Data Governance Stack

```text
┌─────────────────────────────────────────┐
│         Snowflake Data Governance        │
├─────────────────────────────────────────┤
│  Tag-Based Masking Policies             │  ← Classify then auto-protect
│  Column-Level Security (Masking)        │  ← Mask sensitive columns
│  Row Access Policies                    │  ← Filter rows per user/role
│  Object Tagging                         │  ← Classify data
│  Access History (ACCOUNT_USAGE)         │  ← Audit who saw what
│  Data Classification (Horizon)          │  ← Auto-detect PII
│  Governance Policies (Horizon)          │  ← Policy management
└─────────────────────────────────────────┘
```

---

## 7.2 Dynamic Data Masking

Masking policies hide sensitive column values based on the querying role — **without changing the underlying data**.

```sql
-- Step 1: Create masking policy
CREATE MASKING POLICY mask_ssn
  AS (val STRING) RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() IN ('HR_ADMIN', 'SYSADMIN') THEN val         -- show full
    WHEN CURRENT_ROLE() = 'HR_ANALYST' THEN 'XXX-XX-' || RIGHT(val,4) -- partial mask
    ELSE '***-**-****'                                                  -- full mask
  END;

-- Step 2: Apply to column
ALTER TABLE employees 
  MODIFY COLUMN ssn 
  SET MASKING POLICY mask_ssn;

-- Step 3: Query — different roles see different values
-- SYSADMIN sees: 123-45-6789
-- HR_ANALYST sees: XXX-XX-6789
-- ANALYST sees: ***-**-****

-- View applied policies
SHOW MASKING POLICIES;
DESC MASKING POLICY mask_ssn;

-- Remove masking policy
ALTER TABLE employees 
  MODIFY COLUMN ssn 
  UNSET MASKING POLICY;
```

### Masking Policy on Nested JSON

```sql
-- Mask a field inside VARIANT
CREATE MASKING POLICY mask_email_variant
  AS (val VARIANT) RETURNS VARIANT ->
  CASE
    WHEN CURRENT_ROLE() = 'DATA_ADMIN' THEN val
    ELSE TO_VARIANT(REGEXP_REPLACE(val::STRING, '([^@]+)@', '***@'))
  END;

ALTER TABLE user_events
  MODIFY COLUMN payload
  SET MASKING POLICY mask_email_variant;
```

---

## 7.3 Row Access Policies

Row access policies filter which rows a user can see at query time — a dynamic WHERE clause.

```sql
-- Step 1: Create a mapping table (who sees which region)
CREATE TABLE region_access_map (
  role_name VARCHAR,
  region    VARCHAR
);

INSERT INTO region_access_map VALUES
  ('ANALYST_WEST', 'WEST'),
  ('ANALYST_EAST', 'EAST'),
  ('SALES_ADMIN', 'WEST'),
  ('SALES_ADMIN', 'EAST'),
  ('SALES_ADMIN', 'CENTRAL');

-- Step 2: Create row access policy
CREATE ROW ACCESS POLICY region_policy
  AS (region VARCHAR) RETURNS BOOLEAN ->
  CURRENT_ROLE() IN ('SYSADMIN', 'ACCOUNTADMIN')       -- admins see all
  OR EXISTS (
    SELECT 1 FROM region_access_map
    WHERE role_name = CURRENT_ROLE()
    AND region_access_map.region = region              -- match row's region
  );

-- Step 3: Apply to table
ALTER TABLE sales 
  ADD ROW ACCESS POLICY region_policy ON (region);

-- ANALYST_WEST queries sales → only sees WEST rows
-- SALES_ADMIN queries sales → sees all rows
-- ANALYST_EAST → sees only EAST rows

-- Remove row access policy
ALTER TABLE sales DROP ROW ACCESS POLICY region_policy;

-- View policies on a table
SHOW ROW ACCESS POLICIES;
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.POLICY_REFERENCES 
WHERE REF_ENTITY_NAME = 'SALES';
```

---

## 7.4 Object Tagging

Tags allow you to classify Snowflake objects with key-value metadata for governance, cost allocation, and compliance.

```sql
-- Create tag (in a dedicated schema)
CREATE TAG sensitivity ALLOWED_VALUES 'PUBLIC', 'INTERNAL', 'CONFIDENTIAL', 'RESTRICTED';
CREATE TAG cost_center COMMENT = 'Department cost center code';
CREATE TAG pii_type   ALLOWED_VALUES 'EMAIL', 'SSN', 'PHONE', 'NONE';

-- Apply tag to column
ALTER TABLE employees 
  MODIFY COLUMN ssn 
  SET TAG pii_type = 'SSN', sensitivity = 'RESTRICTED';

-- Apply tag to table
ALTER TABLE employees SET TAG sensitivity = 'CONFIDENTIAL', cost_center = 'HR-001';

-- Apply tag to database
ALTER DATABASE PROD_DB SET TAG cost_center = 'ENGINEERING';

-- Query tag references
SELECT * FROM TABLE(SNOWFLAKE.ACCOUNT_USAGE.TAG_REFERENCES_ALL_COLUMNS('EMPLOYEES', 'TABLE'));

-- Find all columns tagged as PII
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.TAG_REFERENCES
WHERE TAG_NAME = 'PII_TYPE'
AND TAG_VALUE != 'NONE';
```

---

## 7.5 Tag-Based Masking Policies

Combine tags + masking to automatically protect all columns with a specific tag.

```sql
-- 1. Create masking policy
CREATE MASKING POLICY pii_mask
  AS (val STRING) RETURNS STRING ->
  CASE WHEN CURRENT_ROLE() IN ('DATA_ADMIN') THEN val ELSE '***MASKED***' END;

-- 2. Associate masking policy with tag
ALTER TAG pii_type SET MASKING POLICY pii_mask;

-- Now: any column tagged pii_type='SSN' auto-applies the masking policy!
-- No need to apply per column — tag drives it.
```

---

## 7.6 Access History — Who Saw What?

```sql
-- Which users read which columns (last 7 days)
SELECT 
  QUERY_START_TIME,
  USER_NAME,
  ROLES_USED,
  DIRECT_OBJECTS_ACCESSED
FROM SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY
WHERE QUERY_START_TIME >= DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY QUERY_START_TIME DESC;

-- Find who accessed a sensitive table
SELECT USER_NAME, QUERY_START_TIME, QUERY_TEXT
FROM SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY
WHERE ARRAY_CONTAINS('PROD_DB.SALES.EMPLOYEES'::VARIANT, 
                      DIRECT_OBJECTS_ACCESSED::VARIANT)
ORDER BY QUERY_START_TIME DESC;
```

---

## 7.7 Projection Policies (Column-level access)

Projection policies prevent columns from appearing in SELECT * results.

```sql
CREATE PROJECTION POLICY no_project_ssn
  AS () RETURNS PROJECTION_CONSTRAINT ->
  CASE
    WHEN CURRENT_ROLE() = 'HR_ADMIN' THEN PROJECTION_CONSTRAINT(ALLOW=>TRUE)
    ELSE PROJECTION_CONSTRAINT(ALLOW=>FALSE)
  END;

ALTER TABLE employees
  MODIFY COLUMN ssn
  SET PROJECTION POLICY no_project_ssn;
-- Now: SELECT * FROM employees won't show ssn for non-HR_ADMIN roles
-- Must explicitly SELECT ssn to trigger masking policy
```

---

## 7.8 Governance Exam Summary

| Feature | What It Controls | Applied To | Dynamic? |
|---------|-----------------|------------|----------|
| Masking Policy | Column value display | Column | ✅ Per role |
| Row Access Policy | Row visibility | Table | ✅ Per user/role |
| Object Tag | Classification metadata | Any object | N/A |
| Tag-Based Masking | Auto-mask tagged columns | Column via tag | ✅ |
| Projection Policy | Column projection (SELECT *) | Column | ✅ |
| Network Policy | IP-based connection | Account/User | ❌ Static |
| RBAC | Object access | All objects | ❌ |

---

# 8. QUERY PERFORMANCE AND EXECUTION

## 8.1 Query Execution Lifecycle

```text
SQL Query submitted
       ↓
Cloud Services Layer:
  1. Parse SQL → AST
  2. Optimize → Execution Plan
  3. Check Result Cache
       ↓ (cache miss)
Virtual Warehouse receives plan
  4. Check Local Disk Cache (SSD)
       ↓ (disk cache miss)
  5. Fetch micro-partitions from S3/blob
  6. Decompress + decode columnar data
  7. Apply filters (pruning already done)
  8. Join / Aggregate / Sort
  9. Return results to Cloud Services
       ↓
Cloud Services:
  10. Store in Result Cache (24h)
  11. Return to client
```

---

## 8.2 Three Cache Layers

| Cache | Location | What It Stores | TTL | Hit = Cost? |
|-------|----------|----------------|-----|------------|
| **Result Cache** | Cloud Services | Exact query results | 24 hours | 0 credits |
| **Metadata Cache** | Cloud Services | Partition stats, counts | Persistent | 0 credits |
| **Local Disk Cache** | Virtual Warehouse SSD | Decompressed micro-partitions | Warehouse running | Credits already paid |

```sql
-- Force result cache bypass (for benchmarking)
ALTER SESSION SET USE_CACHED_RESULT = FALSE;

-- Check if query used result cache
SELECT QUERY_ID, QUERY_TEXT, EXECUTION_STATUS, 
       IS_CLIENT_GENERATED_CHILD_QUERY
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE QUERY_ID = 'your-query-id';
-- Look for execution_time = 0ms → result cache hit
```

> **Exam:** Result cache is invalidated when: (1) underlying data changes, (2) 24h expires, (3) session USE_CACHED_RESULT=FALSE, (4) query modified.

---

## 8.3 Query Profile — Reading the Execution Plan

```text
In Snowsight: Query History → Select Query → Query Profile tab

Key nodes to look for:
┌──────────────────────────────────────────────┐
│ TableScan     → reading micro-partitions     │
│ Filter        → WHERE clause execution       │
│ Join          → join type (hash/merge/cross) │
│ Aggregate     → GROUP BY                     │
│ Sort          → ORDER BY                     │
│ WindowFunction → analytic functions          │
│ Explode       → FLATTEN on VARIANT           │
│ Result        → final output                 │
└──────────────────────────────────────────────┘

Key Metrics in Profile:
- Partitions Scanned vs Total Partitions → pruning efficiency
- Bytes Spilled to Local Storage → memory pressure (bad!)
- Bytes Spilled to Remote Storage → very bad, resize warehouse
- Rows Produced → join explosion check
```

---

## 8.4 Partition Pruning Optimization

```sql
-- BAD — function on column prevents pruning
SELECT * FROM sales WHERE YEAR(sale_date) = 2024;
-- Snowflake can't prune because metadata stores raw DATE values

-- GOOD — range filter enables pruning
SELECT * FROM sales 
WHERE sale_date BETWEEN '2024-01-01' AND '2024-12-31';

-- BAD — leading wildcard prevents pruning on clustering key
SELECT * FROM customers WHERE name LIKE '%smith%';

-- GOOD — use specific filter on clustered column
SELECT * FROM customers WHERE region = 'WEST' AND name LIKE 'Smith%';

-- Check pruning in query history
SELECT 
  QUERY_ID,
  QUERY_TEXT,
  PARTITIONS_SCANNED,
  PARTITIONS_TOTAL,
  ROUND(PARTITIONS_SCANNED/PARTITIONS_TOTAL * 100, 2) AS pct_scanned
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE PARTITIONS_TOTAL > 0
ORDER BY pct_scanned DESC
LIMIT 20;
```

---

## 8.5 Join Optimization

```sql
-- Snowflake uses hash join by default for large tables
-- Broadcast join: for small table vs large table (Snowflake auto-detects)

-- Check join type in Query Profile → Join node shows "HASH JOIN" or "BROADCAST"

-- Help optimizer with join order (large table → small table is generally better)
-- Usually Snowflake handles this, but if not:

-- Use subquery to pre-filter before join
SELECT s.*, c.customer_name
FROM (SELECT * FROM sales WHERE sale_date >= '2024-01-01') s
JOIN customers c ON s.customer_id = c.customer_id;

-- Avoid cartesian product (CROSS JOIN without condition = disaster)
SELECT * FROM orders, customers;  -- BAD: cartesian product
SELECT * FROM orders o JOIN customers c ON o.customer_id = c.customer_id;  -- GOOD
```

---

## 8.6 Spilling — The Performance Killer

```text
Query execution order for large operations (sorts, joins, aggregations):

1. Use Virtual Warehouse RAM (fastest)
   ↓ if not enough RAM
2. Spill to Local SSD (slower — within warehouse)
   ↓ if SSD also full
3. Spill to Remote Storage (S3/blob) — very slow, expensive
```

**Fix for spilling:**
```sql
-- Option 1: Scale up warehouse (more RAM per node)
ALTER WAREHOUSE my_wh SET WAREHOUSE_SIZE = 'X-LARGE';

-- Option 2: Reduce data before expensive operations
-- Filter early, aggregate early, don't SELECT *

-- Option 3: Break query into steps using temp tables
CREATE TEMPORARY TABLE pre_agg AS
SELECT customer_id, SUM(amount) AS total
FROM sales
WHERE sale_date >= '2024-01-01'
GROUP BY customer_id;

SELECT * FROM pre_agg
JOIN customers c ON pre_agg.customer_id = c.customer_id
ORDER BY total DESC;
```

---

## 8.7 Search Optimization Service

For point lookups and selective queries on non-clustered columns.

```sql
-- Enable search optimization on a table
ALTER TABLE customers ADD SEARCH OPTIMIZATION;

-- Selective (specific columns)
ALTER TABLE customers ADD SEARCH OPTIMIZATION ON EQUALITY(email, phone);

-- Check search optimization status
SHOW TABLES LIKE 'CUSTOMERS';
-- Look at SEARCH_OPTIMIZATION and SEARCH_OPTIMIZATION_PROGRESS columns

-- Remove
ALTER TABLE customers DROP SEARCH OPTIMIZATION;
```

**When to use Search Optimization:**

| Scenario | Search Opt | Clustering Key |
|----------|-----------|----------------|
| Point lookups (WHERE id = 123) | ✅ | Overkill |
| Range scans (date ranges) | ❌ | ✅ |
| High cardinality exact match | ✅ | ❌ |
| Frequent ad-hoc filters on any column | ✅ | ❌ |
| Sequential scans of large ranges | ❌ | ✅ |

---

## 8.8 Query Acceleration Service

For warehouse-level acceleration of long-running queries without resizing.

```sql
-- Enable on warehouse
ALTER WAREHOUSE my_wh SET ENABLE_QUERY_ACCELERATION = TRUE;

-- Set max scale factor (how many extra compute nodes)
ALTER WAREHOUSE my_wh SET QUERY_ACCELERATION_MAX_SCALE_FACTOR = 8;
-- Snowflake can burst up to 8x the warehouse size for eligible queries

-- Check which queries were accelerated
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_ACCELERATION_ELIGIBLE
ORDER BY ELIGIBLE_QUERY_ACCELERATION_TIME DESC;
```

---

## 8.9 Performance Diagnostics Cheat-Sheet

| Symptom | Root Cause | Fix |
|---------|-----------|-----|
| Query slow, 100% partitions scanned | No pruning | Add clustering key or fix WHERE clause |
| "Bytes Spilled to Remote" in profile | Warehouse too small | Scale up warehouse size |
| Query slow, no disk spill | Compute bottleneck | Scale up warehouse |
| Multiple users slow simultaneously | Resource contention | Multi-cluster warehouse |
| Same query slow repeatedly (no cache) | Result cache disabled | `ALTER SESSION SET USE_CACHED_RESULT=TRUE` |
| JOIN producing millions of extra rows | Missing join condition / cartesian | Fix JOIN ON clause |
| Analytic (window) function slow | Large window frame | Partition by more columns |

---

## 8.10 Explain Plan

```sql
-- View execution plan without running (no compute cost)
EXPLAIN SELECT * FROM sales s
JOIN customers c ON s.customer_id = c.customer_id
WHERE s.sale_date = '2024-06-01';

-- EXPLAIN USING TEXT (default)
EXPLAIN USING TEXT SELECT COUNT(*) FROM large_table;

-- EXPLAIN USING JSON — for programmatic parsing
EXPLAIN USING JSON SELECT * FROM sales;

-- EXPLAIN USING TABULAR — best for humans
EXPLAIN USING TABULAR 
SELECT region, SUM(amount) 
FROM sales 
GROUP BY region;
```

---

# 9. VIRTUAL WAREHOUSE SIZING AND SCALING

## 9.1 Warehouse Sizes and Credit Consumption

| Size | Credits/Hour | vCPUs (approx) | RAM (approx) | Nodes |
|------|-------------|-----------------|--------------|-------|
| X-Small (XS) | 1 | 2 | 4 GB | 1 |
| Small (S) | 2 | 4 | 16 GB | 1 |
| Medium (M) | 4 | 8 | 32 GB | 2 |
| Large (L) | 8 | 16 | 64 GB | 4 |
| X-Large (XL) | 16 | 32 | 128 GB | 8 |
| 2X-Large | 32 | 64 | 256 GB | 16 |
| 3X-Large | 64 | 128 | 512 GB | 32 |
| 4X-Large | 128 | 256 | 1 TB | 64 |
| 5X-Large | 256 | — | — | 128 |
| 6X-Large | 512 | — | — | 256 |

> **Exam:** Each size up = **2× credits/hour**. A Medium costs **4×** an XS. Minimum billing = **60 seconds** per start.

```text
Credit pricing examples (at $2/credit):
XS: 1 credit/hr = $2/hr = $0.033/min (60-sec minimum = $0.033 minimum charge)
XL: 16 credits/hr = $32/hr
4XL: 128 credits/hr = $256/hr
```

---

## 9.2 Warehouse Configuration

```sql
-- Create warehouse with all options
CREATE WAREHOUSE analytics_wh
  WAREHOUSE_SIZE = MEDIUM
  AUTO_SUSPEND = 300                    -- suspend after 5 min idle (seconds)
  AUTO_RESUME = TRUE                   -- auto-start on query
  MIN_CLUSTER_COUNT = 1                -- multi-cluster: min clusters
  MAX_CLUSTER_COUNT = 3                -- multi-cluster: max clusters (Enterprise+)
  SCALING_POLICY = STANDARD            -- STANDARD or ECONOMY
  INITIALLY_SUSPENDED = TRUE
  MAX_CONCURRENCY_LEVEL = 8            -- concurrent queries before queueing
  STATEMENT_QUEUED_TIMEOUT_IN_SECONDS = 0    -- 0 = queue forever
  STATEMENT_TIMEOUT_IN_SECONDS = 172800      -- kill queries after 48h
  COMMENT = 'Analytics team warehouse';

-- Resize (takes effect immediately for new queries)
ALTER WAREHOUSE analytics_wh SET WAREHOUSE_SIZE = LARGE;

-- Manually suspend / resume
ALTER WAREHOUSE analytics_wh SUSPEND;
ALTER WAREHOUSE analytics_wh RESUME;

-- Monitor warehouse
SHOW WAREHOUSES LIKE 'ANALYTICS_WH';
```

---

## 9.3 Multi-Cluster Warehouses (MCW) — Enterprise Feature

```text
Multi-Cluster Warehouse:
  Single warehouse name, but Snowflake spins up additional clusters
  when concurrency demand increases.

          ┌────────────────────────────────────────┐
          │         analytics_wh (MCW)              │
          │  MIN=1 cluster, MAX=5 clusters          │
          │                                          │
          │  Cluster 1 (always on)                  │
          │  ┌──────────────────────────────┐       │
          │  │ Thread 1, 2, 3, ..., N       │       │
          │  └──────────────────────────────┘       │
          │                                          │
          │  Cluster 2 (spins up when busy)         │
          │  ┌──────────────────────────────┐       │
          │  │ Thread 1, 2, 3, ..., N       │       │
          │  └──────────────────────────────┘       │
          │                                          │
          │  Clusters 3-5 (on demand)               │
          └────────────────────────────────────────┘
```

### Scaling Policies

| Policy | Behavior | Use Case |
|--------|----------|----------|
| **STANDARD** | Spin up extra cluster immediately when queries queue | Latency-sensitive |
| **ECONOMY** | Wait until cluster busy for 6 min before adding | Cost-sensitive |

```sql
-- MCW example
CREATE WAREHOUSE bi_reporting_wh
  WAREHOUSE_SIZE = LARGE
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 5
  SCALING_POLICY = STANDARD
  AUTO_SUSPEND = 120;

-- View cluster usage
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
WHERE WAREHOUSE_NAME = 'BI_REPORTING_WH'
AND START_TIME >= DATEADD('day', -7, CURRENT_TIMESTAMP());
```

---

## 9.4 Warehouse Sizing Decision Guide

```text
Scale UP (bigger warehouse) when:
  ✅ Query spills to local/remote storage
  ✅ Single long-running query is slow
  ✅ Complex transforms or large sorts
  ✅ ETL jobs with massive aggregations

Scale OUT (more clusters, MCW) when:
  ✅ Many concurrent users hitting same warehouse
  ✅ Queries are queuing (wait time > 0)
  ✅ BI dashboard users experiencing slowness
  ✅ Peak/off-peak concurrency pattern

Keep same size when:
  ✅ Queries run fast, no spilling
  ✅ Adding size doesn't reduce time proportionally
  ✅ Cost is primary concern
```

---

## 9.5 AUTO_SUSPEND Best Practices

| Environment | AUTO_SUSPEND Setting | Rationale |
|-------------|---------------------|-----------|
| Production ETL | 300-600s (5-10 min) | Frequent use, don't want cold starts |
| BI Reporting | 120-300s | Balance cost and user experience |
| Dev/Test | 60s | Minimize idle costs |
| Batch (run once) | 60s | Always suspend after batch |
| Scheduled tasks | Serverless or 60s | Use serverless tasks when possible |

> **Exam:** AUTO_SUSPEND minimum = **60 seconds**. Below that, it's still at least 60s. Setting to 0 means **never suspend** (bad for cost!).

---

## 9.6 Warehouse Monitoring Queries

```sql
-- Warehouse credit usage last 30 days
SELECT 
  WAREHOUSE_NAME,
  SUM(CREDITS_USED) AS total_credits,
  SUM(CREDITS_USED) * 2.5 AS estimated_cost_usd  -- $2.5/credit
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
WHERE START_TIME >= DATEADD('day', -30, CURRENT_TIMESTAMP())
GROUP BY WAREHOUSE_NAME
ORDER BY total_credits DESC;

-- Warehouse queue and load analysis
SELECT 
  WAREHOUSE_NAME,
  AVG(AVG_RUNNING) AS avg_concurrent_queries,
  AVG(AVG_QUEUED_LOAD) AS avg_queue_depth,
  MAX(AVG_QUEUED_LOAD) AS max_queue
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_LOAD_HISTORY
WHERE START_TIME >= DATEADD('hour', -24, CURRENT_TIMESTAMP())
GROUP BY WAREHOUSE_NAME;

-- Idle warehouse detection (suspended but had no usage)
SELECT 
  WAREHOUSE_NAME,
  DATEDIFF('hour', LAST_STARTED_TIME, CURRENT_TIMESTAMP()) AS hours_since_use
FROM (
  SELECT WAREHOUSE_NAME, MAX(START_TIME) AS LAST_STARTED_TIME
  FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
  GROUP BY WAREHOUSE_NAME
)
WHERE hours_since_use > 168;  -- not used in 7 days
```

---

# 10. COST MANAGEMENT AND GOVERNANCE

## 10.1 Snowflake Cost Components

```text
Total Snowflake Cost
├── Compute Cost (Virtual Warehouses)
│     Credits/hour × hours running × $/credit
│
├── Storage Cost
│     avg daily bytes × $/TB/month
│     Includes: active data + Time Travel + Fail-safe + stages
│
├── Cloud Services Cost
│     10% of daily compute credits (free tier)
│     Excess over 10% charged separately
│
├── Data Transfer Cost
│     Moving data OUT of cloud region or between clouds
│     (Same region = free; cross-region = charged)
│
└── Snowflake Marketplace / Listings
      Per-listing pricing (if applicable)
```

---

## 10.2 Resource Monitors

Resource monitors set credit limits and trigger alerts or suspension.

```sql
-- Create resource monitor (ACCOUNTADMIN required)
CREATE RESOURCE MONITOR analytics_monitor
  CREDIT_QUOTA = 500                      -- 500 credits per month
  FREQUENCY = MONTHLY
  START_TIMESTAMP = '2024-01-01 00:00 UTC'
  TRIGGERS 
    ON 75 PERCENT DO NOTIFY               -- email alert at 75%
    ON 90 PERCENT DO NOTIFY               -- email alert at 90%
    ON 100 PERCENT DO SUSPEND             -- suspend warehouse at 100%
    ON 110 PERCENT DO SUSPEND_IMMEDIATE;  -- kill running queries at 110%

-- Apply to a warehouse
ALTER WAREHOUSE analytics_wh SET RESOURCE_MONITOR = analytics_monitor;

-- Apply to entire account
ALTER ACCOUNT SET RESOURCE_MONITOR = analytics_monitor;

-- View monitors
SHOW RESOURCE MONITORS;

-- Modify quota mid-cycle
ALTER RESOURCE MONITOR analytics_monitor SET CREDIT_QUOTA = 750;
```

### Trigger Actions

| Action | Effect |
|--------|--------|
| `NOTIFY` | Send email to ACCOUNTADMIN(s) |
| `SUSPEND` | Suspend warehouse after current queries finish |
| `SUSPEND_IMMEDIATE` | Kill all running queries, suspend warehouse |

> **Exam:** `SUSPEND_IMMEDIATE` kills in-flight queries. `SUSPEND` waits for current queries to complete. Resource monitors reset at FREQUENCY cycle start.

---

## 10.3 Cost Attribution and Chargeback

```sql
-- Cost per warehouse (for department chargeback)
SELECT 
  WAREHOUSE_NAME,
  DATE_TRUNC('month', START_TIME) AS month,
  SUM(CREDITS_USED) AS credits,
  SUM(CREDITS_USED) * 3.0 AS cost_usd  -- adjust per contract rate
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
GROUP BY 1, 2
ORDER BY 2 DESC, 3 DESC;

-- Cloud services cost (should stay < 10% of compute)
SELECT 
  DATE_TRUNC('day', USAGE_DATE) AS day,
  CREDITS_USED_COMPUTE,
  CREDITS_USED_CLOUD_SERVICES,
  CREDITS_USED_CLOUD_SERVICES / NULLIF(CREDITS_USED_COMPUTE, 0) AS cloud_svc_ratio
FROM SNOWFLAKE.ACCOUNT_USAGE.METERING_HISTORY
WHERE USAGE_DATE >= DATEADD('day', -30, CURRENT_DATE())
ORDER BY day DESC;

-- Tag-based cost attribution
-- Apply cost_center tag to warehouses
ALTER WAREHOUSE analytics_wh SET TAG cost_center = 'ANALYTICS-001';
ALTER WAREHOUSE etl_wh SET TAG cost_center = 'ENGINEERING-002';

-- Query cost by tag (use TAG_REFERENCES view)
SELECT 
  r.TAG_VALUE AS cost_center,
  SUM(m.CREDITS_USED) AS total_credits
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY m
JOIN SNOWFLAKE.ACCOUNT_USAGE.TAG_REFERENCES r 
  ON r.OBJECT_NAME = m.WAREHOUSE_NAME 
  AND r.TAG_NAME = 'COST_CENTER'
  AND r.OBJECT_DOMAIN = 'WAREHOUSE'
WHERE m.START_TIME >= DATEADD('month', -1, CURRENT_TIMESTAMP())
GROUP BY r.TAG_VALUE;
```

---

## 10.4 Identifying and Reducing Waste

```sql
-- Long-running queries (potential optimization targets)
SELECT 
  USER_NAME, WAREHOUSE_NAME,
  QUERY_TEXT,
  TOTAL_ELAPSED_TIME/1000 AS elapsed_seconds,
  CREDITS_USED_CLOUD_SERVICES,
  BYTES_SCANNED/1e9 AS gb_scanned,
  PARTITIONS_SCANNED, PARTITIONS_TOTAL
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE TOTAL_ELAPSED_TIME > 120000  -- > 2 minutes
AND START_TIME >= DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY TOTAL_ELAPSED_TIME DESC
LIMIT 50;

-- Queries with full table scans (no pruning)
SELECT QUERY_ID, QUERY_TEXT, PARTITIONS_SCANNED, PARTITIONS_TOTAL
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE PARTITIONS_TOTAL > 0
  AND PARTITIONS_SCANNED = PARTITIONS_TOTAL  -- 100% scan
  AND START_TIME >= DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY PARTITIONS_TOTAL DESC
LIMIT 20;

-- Warehouses running but idle (no queries)
SELECT 
  w.WAREHOUSE_NAME,
  COUNT(q.QUERY_ID) AS query_count,
  w.CREDITS_USED
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY w
LEFT JOIN SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY q
  ON q.WAREHOUSE_NAME = w.WAREHOUSE_NAME
  AND q.START_TIME BETWEEN w.START_TIME AND w.END_TIME
WHERE w.START_TIME >= DATEADD('day', -7, CURRENT_TIMESTAMP())
GROUP BY 1, 3
HAVING query_count = 0;
```

---

## 10.5 Cost Optimization Checklist

| Area | Optimization | Expected Saving |
|------|-------------|-----------------|
| **Compute** | Set AUTO_SUSPEND = 60s for dev warehouses | 40-70% |
| **Compute** | Right-size warehouses (don't over-provision) | 20-50% |
| **Compute** | Use MCW instead of giant single warehouse | 15-30% |
| **Compute** | Schedule ETL in off-peak (same cost, less contention) | Quality improvement |
| **Storage** | Use TRANSIENT tables for staging | Eliminate Fail-safe cost |
| **Storage** | Reduce Time Travel on large tables with frequent changes | 30-60% storage |
| **Storage** | DROP unused clones, stages | Variable |
| **Query** | Cache results — don't repeat identical queries | Direct credit saving |
| **Query** | Clustering on high-filter columns | Less scanning |
| **Cloud Svcs** | Avoid excessive metadata operations in loops | <10% threshold |

---

## 10.6 Snowflake Budgets (Newer Feature)

```sql
-- Create a budget (newer governance feature)
CREATE SNOWFLAKE.CORE.BUDGET my_budget
  CREDIT_QUOTA = 1000
  FREQUENCY = MONTHLY;

-- Add warehouse to budget scope
CALL my_budget!ADD_RESOURCE(
  SYSTEM$REFERENCE('WAREHOUSE', 'ANALYTICS_WH', 'SESSION', 'MONITOR')
);

-- Check budget status
CALL my_budget!GET_SPENDING_HISTORY();
CALL my_budget!GET_SERVICE_TYPE_USAGE(
  SERVICE_TYPE=>'WAREHOUSE_METERING',
  TIME_LOWER_BOUND=>DATEADD('day', -30, CURRENT_TIMESTAMP()),
  TIME_UPPER_BOUND=>CURRENT_TIMESTAMP()
);
```

---

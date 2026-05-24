# Advanced SQL — Complete Syntax Revision Guide

> **How to use this file:** Every section shows the exact syntax you need, with short explanations only where the behaviour isn't obvious. Think of it as your personal cheat-sheet, not a textbook.

---

## Table of Contents
1. [SQL Performance Tuning](#1-sql-performance-tuning)
2. [Security Management](#2-security-management)
3. [Advanced SQL Features (Window Functions, Hierarchical Queries, Subqueries, MERGE, PIVOT)](#3-advanced-sql-features)
4. [PL/SQL Fundamentals](#4-plsql-fundamentals)
5. [PL/SQL Database Objects (Procedures, Functions, Packages, Triggers)](#5-plsql-database-objects)
6. [Concurrency & Locking](#6-concurrency--locking)
7. [SQL Testing & Validation](#7-sql-testing--validation)

---

## 1. SQL Performance Tuning

### 1.1 Viewing Execution Plans

#### Oracle
```sql
-- EXPLAIN PLAN
EXPLAIN PLAN FOR
SELECT e.employee_name, d.department_name
FROM employees e JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 50000;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);

-- AUTOTRACE (SQL*Plus)
SET AUTOTRACE ON EXPLAIN STATISTICS

-- Real-time monitoring
SELECT DBMS_SQLTUNE.REPORT_SQL_MONITOR(sql_id => 'abc123', type => 'TEXT') FROM dual;
```

#### Snowflake
```sql
-- Get query ID
SELECT LAST_QUERY_ID();

-- Query history
SELECT * FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY_BY_USER())
WHERE QUERY_ID = '<query_id>'
ORDER BY START_TIME DESC;
```

#### Databricks / Spark SQL
```sql
EXPLAIN SELECT * FROM orders WHERE order_date = '2024-01-15';
EXPLAIN EXTENDED SELECT ...;
EXPLAIN COST SELECT ...;
EXPLAIN FORMATTED SELECT ...;   -- easiest to read
```

---

### 1.2 Oracle Indexes

```sql
-- B-Tree (default)
CREATE INDEX emp_lastname_idx ON employees(last_name);

-- Composite (leading column drives index usage)
CREATE INDEX emp_dept_job_idx ON employees(department_id, job_id);

-- Unique
CREATE UNIQUE INDEX emp_email_uk ON employees(email);
ALTER TABLE employees ADD CONSTRAINT emp_pk PRIMARY KEY (employee_id);

-- Function-Based (needed when you filter on a function)
CREATE INDEX emp_upper_name_idx ON employees(UPPER(last_name));
CREATE INDEX emp_year_idx ON employees(EXTRACT(YEAR FROM hire_date));

-- Bitmap (DW/read-only only — causes severe locking in OLTP!)
CREATE BITMAP INDEX emp_gender_bidx ON employees(gender);

-- Reverse Key (distributes sequential inserts; can't use for range scans)
CREATE INDEX order_id_ridx ON orders(order_id) REVERSE;
```

---

### 1.3 Oracle Statistics

```sql
-- Gather table stats
EXEC DBMS_STATS.GATHER_TABLE_STATS(
    ownname          => 'HR',
    tabname          => 'EMPLOYEES',
    estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE,
    method_opt       => 'FOR ALL COLUMNS SIZE AUTO',
    cascade          => TRUE
);

-- Gather schema stats
EXEC DBMS_STATS.GATHER_SCHEMA_STATS('HR');

-- Histogram for skewed column
EXEC DBMS_STATS.GATHER_TABLE_STATS(
    ownname => 'HR', tabname => 'EMPLOYEES',
    method_opt => 'FOR COLUMNS SIZE 254 department_id'
);

-- Check freshness
SELECT table_name, num_rows, last_analyzed,
       ROUND((SYSDATE - last_analyzed), 1) days_old
FROM user_tables ORDER BY last_analyzed NULLS FIRST;
```

---

### 1.4 Anti-Patterns to Avoid

| Pattern | Bad | Good |
|---------|-----|------|
| Function on indexed column | `TRUNC(order_date) = '2024-01-15'` | `order_date >= '2024-01-15' AND order_date < '2024-01-16'` |
| Implicit type conversion | `WHERE phone_number = 5551234` | `WHERE phone_number = '5551234'` |
| Leading wildcard | `LIKE '%SMITH%'` | `LIKE 'SMITH%'` or Oracle Text |
| SELECT * (Snowflake/Databricks) | `SELECT * FROM wide_table` | Select only needed columns |
| ORDER BY without LIMIT (Snowflake) | `ORDER BY date DESC` | `ORDER BY date DESC LIMIT 1000` |

---

### 1.5 Snowflake Clustering Keys

```sql
-- Check clustering depth (lower = better; 1-2 is ideal)
SELECT SYSTEM$CLUSTERING_DEPTH('orders');
SELECT SYSTEM$CLUSTERING_INFORMATION('orders', '(o_orderdate)');

-- Add clustering key
ALTER TABLE orders CLUSTER BY (o_orderdate);
ALTER TABLE sales  CLUSTER BY (sale_date, region);

-- Expression-based
ALTER TABLE events CLUSTER BY (TO_DATE(event_timestamp));
```

#### Snowflake Caching
```sql
-- Bypass result cache (force fresh run)
ALTER SESSION SET USE_CACHED_RESULT = FALSE;
ALTER SESSION SET USE_CACHED_RESULT = TRUE;

-- Check cache hit
SELECT QUERY_ID, PERCENTAGE_SCANNED_FROM_CACHE
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY()) ORDER BY START_TIME DESC LIMIT 10;
```

#### Snowflake Warehouse Management
```sql
CREATE WAREHOUSE analytics_wh
    WAREHOUSE_SIZE      = 'LARGE'
    AUTO_SUSPEND        = 300
    AUTO_RESUME         = TRUE
    MIN_CLUSTER_COUNT   = 1
    MAX_CLUSTER_COUNT   = 4
    SCALING_POLICY      = 'STANDARD';

ALTER WAREHOUSE analytics_wh SET WAREHOUSE_SIZE = 'XLARGE';
```

#### Snowflake Materialized Views
```sql
CREATE MATERIALIZED VIEW mv_daily_sales AS
SELECT sale_date, region, COUNT(*) AS num_sales, SUM(amount) AS total
FROM sales GROUP BY sale_date, region;

SHOW MATERIALIZED VIEWS;
ALTER MATERIALIZED VIEW mv_daily_sales REFRESH;
```

#### Snowflake Search Optimization
```sql
ALTER TABLE customers ADD SEARCH OPTIMIZATION;
SELECT SYSTEM$ESTIMATE_SEARCH_OPTIMIZATION_COSTS('customers');
```

---

### 1.6 Databricks / Delta Lake

```sql
-- Partition pruning-friendly query
SELECT * FROM sales WHERE sale_date = '2024-01-15';

-- Create partitioned table
CREATE TABLE sales (...) USING DELTA PARTITIONED BY (sale_date);

-- Z-Ordering (multi-dimensional clustering within partitions)
OPTIMIZE sales ZORDER BY (customer_id, product_id);
OPTIMIZE events WHERE event_date >= '2024-01-01' ZORDER BY (user_id);

-- Compact small files
OPTIMIZE orders;

-- Remove old files (keep 7 days of history)
VACUUM orders RETAIN 168 HOURS;

-- Inspect table
DESCRIBE DETAIL orders;
DESCRIBE HISTORY orders;

-- Auto-optimize
ALTER TABLE orders SET TBLPROPERTIES (
    'delta.autoOptimize.optimizeWrite' = 'true',
    'delta.autoOptimize.autoCompact'   = 'true'
);

-- Session-level auto-optimize
SET spark.databricks.delta.optimizeWrite.enabled = true;
SET spark.databricks.delta.autoCompact.enabled   = true;
```

#### Databricks Join Hints
```sql
-- Broadcast join (for small table)
SELECT /*+ BROADCAST(d) */ e.name, d.dept_name
FROM employees e JOIN departments d ON e.dept_id = d.dept_id;

-- Increase broadcast threshold
SET spark.sql.autoBroadcastJoinThreshold = 100m;
```

#### Databricks AQE (Adaptive Query Execution)
```sql
SET spark.sql.adaptive.enabled                          = true;
SET spark.sql.adaptive.coalescePartitions.enabled       = true;
SET spark.sql.adaptive.skewJoin.enabled                 = true;
SET spark.sql.adaptive.localShuffleReader.enabled       = true;
```

#### Databricks Bloom Filters
```sql
CREATE BLOOMFILTER INDEX ON TABLE orders FOR COLUMNS (order_id);
SET spark.databricks.io.skipping.bloomFilter.enabled = true;
```

#### Databricks Caching
```sql
CACHE TABLE customers;
CACHE TABLE orders OPTIONS ('storageLevel' 'MEMORY_AND_DISK');
UNCACHE TABLE customers;
CLEAR CACHE;
```

#### Databricks Statistics
```sql
ANALYZE TABLE employees COMPUTE STATISTICS;
ANALYZE TABLE employees COMPUTE STATISTICS FOR COLUMNS department_id, salary;
```

---

## 2. Security Management

### 2.1 User and Profile Management

```sql
-- Create user
CREATE USER ecom_app IDENTIFIED BY "Str0ng_P@ss!2024"
    DEFAULT TABLESPACE users
    TEMPORARY TABLESPACE temp
    QUOTA 500M ON users
    ACCOUNT UNLOCK;

-- Modify user
ALTER USER ecom_user IDENTIFIED BY "N3w$ecureP@ss2024";
ALTER USER former_employee ACCOUNT LOCK;
ALTER USER ecom_user ACCOUNT UNLOCK;
ALTER USER contractor_user PASSWORD EXPIRE;
ALTER USER ecom_user DEFAULT TABLESPACE new_tablespace;
ALTER USER ecom_user QUOTA 2G ON ecom_data;
ALTER USER ecom_user PROFILE prod_security_profile;

-- Drop user (CASCADE removes all owned objects)
DROP USER test_user CASCADE;

-- Check user info
SELECT username, account_status, default_tablespace, created
FROM dba_users ORDER BY created DESC;
```

---

### 2.2 Profiles (Password & Resource Policies)

```sql
CREATE PROFILE prod_security_profile LIMIT
    PASSWORD_LIFE_TIME       90
    PASSWORD_REUSE_TIME      365
    PASSWORD_REUSE_MAX       12
    PASSWORD_VERIFY_FUNCTION ora12c_verify_function
    PASSWORD_LOCK_TIME       1
    PASSWORD_GRACE_TIME      7
    FAILED_LOGIN_ATTEMPTS    5
    SESSIONS_PER_USER        3
    CPU_PER_CALL             60000
    CONNECT_TIME             480
    IDLE_TIME                30;

CREATE PROFILE service_account_profile LIMIT
    PASSWORD_LIFE_TIME UNLIMITED
    FAILED_LOGIN_ATTEMPTS 3;
```

---

### 2.3 Privileges

```sql
-- System privileges
GRANT CREATE SESSION   TO dev_user;
GRANT CREATE TABLE     TO dev_user;
GRANT CREATE VIEW      TO dev_user;
GRANT CREATE PROCEDURE TO dev_user;
GRANT CREATE SEQUENCE  TO dev_user;

-- WITH ADMIN OPTION (lets grantee pass system privs to others; revoke does NOT cascade)
GRANT CREATE SESSION TO team_lead WITH ADMIN OPTION;

-- Object privileges
GRANT SELECT ON hr.employees TO reporting_user;
GRANT SELECT, INSERT, UPDATE ON sales.orders TO sales_app;

-- WITH GRANT OPTION (lets grantee pass object privs; revoke DOES cascade)
GRANT SELECT ON hr.employees TO manager WITH GRANT OPTION;

-- Column-level
GRANT SELECT (employee_id, first_name, last_name) ON hr.employees TO public_portal;
GRANT UPDATE (phone_number, email) ON hr.employees TO self_service_app;

-- Stored procedure
GRANT EXECUTE ON hr.calculate_bonus TO payroll_app;

-- Revoke
REVOKE CREATE TABLE FROM former_developer;
REVOKE SELECT ON hr.employees FROM manager;
```

---

### 2.4 Roles

```sql
-- Create role
CREATE ROLE sales_role;
CREATE ROLE sensitive_data_role IDENTIFIED BY "R0l3P@ss#2024";

-- Build a role
GRANT CREATE SESSION           TO sales_role;
GRANT SELECT ON sales.orders   TO sales_role;
GRANT INSERT, UPDATE ON sales.orders TO sales_role;
GRANT EXECUTE ON sales.calculate_discount TO sales_role;

-- Role hierarchy (inherit a role inside another role)
GRANT employee_base_role TO sales_role;

-- Assign roles to users
GRANT sales_role TO john_smith;
GRANT sales_role, reporting_role TO jane_doe WITH ADMIN OPTION;

-- Control default roles
ALTER USER john_smith DEFAULT ROLE sales_role;
ALTER USER secure_user DEFAULT ROLE NONE;

-- Enable/disable roles in a session
SET ROLE admin_role IDENTIFIED BY "Adm1n#S3cur3";
SET ROLE ALL;
SET ROLE NONE;
SET ROLE sales_role, reporting_role;

-- Drop role
DROP ROLE old_role;
```

---

### 2.5 Security Dictionary Views

```sql
-- Users and accounts
SELECT username, account_status, lock_date, expiry_date FROM dba_users;
SELECT username, sid, serial#, status, logon_time FROM v$session WHERE type = 'USER';

-- Privileges granted to a user
SELECT grantee, privilege, admin_option FROM dba_sys_privs WHERE grantee = 'SALES_USER';
SELECT grantee, owner, table_name, privilege, grantable FROM dba_tab_privs WHERE grantee = 'SALES_USER';
SELECT grantee, owner, table_name, column_name, privilege FROM dba_col_privs WHERE grantee = 'SALES_USER';

-- Roles
SELECT role, password_required FROM dba_roles;
SELECT grantee, granted_role, admin_option, default_role FROM dba_role_privs WHERE grantee = 'SALES_USER';
SELECT role, privilege FROM role_sys_privs WHERE role = 'SALES_ROLE';
```

---

### 2.6 Views as Security Layer

```sql
-- Column-level security: expose only safe columns
CREATE OR REPLACE VIEW hr.employee_directory AS
SELECT employee_id, first_name, last_name, email, phone, job_id, department_id
FROM hr.employees;

GRANT SELECT ON hr.employee_directory TO public_portal_role;

-- Row-level security: filter by context value
CREATE OR REPLACE VIEW saas.my_customers AS
SELECT customer_id, customer_name, email
FROM saas.customers
WHERE tenant_id = SYS_CONTEXT('tenant_ctx', 'tenant_id');
```

---

### 2.7 VPD (Virtual Private Database)

```sql
-- Policy function that returns a WHERE predicate string
CREATE OR REPLACE FUNCTION sales.customer_policy(p_schema VARCHAR2, p_object VARCHAR2)
RETURN VARCHAR2 AS
    v_role VARCHAR2(50) := SYS_CONTEXT('sales_ctx', 'role');
BEGIN
    IF v_role = 'SALES_REP' THEN
        RETURN 'assigned_rep_id = SYS_CONTEXT(''sales_ctx'', ''employee_id'')';
    ELSIF v_role = 'VP_SALES' THEN
        RETURN '1=1';
    ELSE
        RETURN '1=0';
    END IF;
END;
/

-- Attach the policy
BEGIN
    DBMS_RLS.ADD_POLICY(
        object_schema   => 'SALES',
        object_name     => 'CUSTOMERS',
        policy_name     => 'CUSTOMER_ACCESS_POLICY',
        function_schema => 'SALES',
        policy_function => 'CUSTOMER_POLICY',
        statement_types => 'SELECT, INSERT, UPDATE, DELETE',
        update_check    => TRUE,
        enable          => TRUE
    );
END;
/
```

---

### 2.8 Data Redaction

```sql
-- Partial mask: show only last 4 digits of SSN
BEGIN
    DBMS_REDACT.ADD_POLICY(
        object_schema       => 'HR',
        object_name         => 'EMPLOYEES',
        column_name         => 'SSN',
        policy_name         => 'SSN_REDACT_POLICY',
        function_type       => DBMS_REDACT.PARTIAL,
        function_parameters => 'VVVFVVFVVVV,VVV-VV-VVVV,X,1,5',
        expression          => 'SYS_CONTEXT(''USERENV'', ''SESSION_USER'') != ''HR_ADMIN'''
    );
END;
/
-- Non-admins see: XXX-XX-6789
```

---

### 2.9 Auditing

```sql
-- Unified audit policy
CREATE AUDIT POLICY sensitive_data_access_policy
    ACTIONS SELECT ON finance.transactions, SELECT ON hr.employee_salaries;
AUDIT POLICY sensitive_data_access_policy;

CREATE AUDIT POLICY ddl_changes_policy
    ACTIONS CREATE TABLE, ALTER TABLE, DROP TABLE, CREATE INDEX, DROP INDEX;
AUDIT POLICY ddl_changes_policy;

CREATE AUDIT POLICY login_policy ACTIONS LOGON, LOGOFF;
AUDIT POLICY login_policy;

-- Query audit trail
SELECT event_timestamp, dbusername, action_name, object_schema, object_name, sql_text
FROM unified_audit_trail
WHERE event_timestamp > SYSDATE - 7
ORDER BY event_timestamp DESC;

-- Fine-Grained Auditing (FGA) — fires only on specific condition
BEGIN
    DBMS_FGA.ADD_POLICY(
        object_schema   => 'FINANCE',
        object_name     => 'TRANSACTIONS',
        policy_name     => 'HIGH_VALUE_TRANSACTION_AUDIT',
        audit_condition => 'amount > 100000',
        audit_column    => 'AMOUNT',
        statement_types => 'SELECT, INSERT, UPDATE'
    );
END;
/
```

---

## 3. Advanced SQL Features

### 3.1 Window Functions — Core Syntax

```sql
function_name(arguments) OVER (
    [PARTITION BY partition_expression]
    [ORDER BY sort_expression [ASC|DESC]]
    [ROWS | RANGE BETWEEN frame_start AND frame_end]
)
```

Frame boundaries: `UNBOUNDED PRECEDING`, `n PRECEDING`, `CURRENT ROW`, `n FOLLOWING`, `UNBOUNDED FOLLOWING`

---

### 3.2 Ranking Functions

```sql
-- ROW_NUMBER: unique number per row (no ties)
ROW_NUMBER() OVER (PARTITION BY region ORDER BY amount DESC)

-- RANK: gaps after ties (1,1,3)
RANK() OVER (ORDER BY amount DESC)

-- DENSE_RANK: no gaps (1,1,2)
DENSE_RANK() OVER (ORDER BY amount DESC)

-- PERCENT_RANK: percentile 0-1
PERCENT_RANK() OVER (PARTITION BY department_id ORDER BY salary)

-- NTILE: divide rows into n buckets
NTILE(5) OVER (ORDER BY salary)        -- for RFM scoring, deciles, etc.
```

**Practical — top N per group:**
```sql
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY region ORDER BY amount DESC) AS rn
    FROM sales_data
) WHERE rn <= 3;
```

---

### 3.3 Aggregate Window Functions

```sql
-- Keep all rows but add aggregated columns
SELECT
    amount,
    SUM(amount)   OVER ()                          AS grand_total,
    SUM(amount)   OVER (PARTITION BY region)       AS region_total,
    AVG(amount)   OVER (PARTITION BY salesperson)  AS person_avg,
    COUNT(*)      OVER (PARTITION BY region)       AS region_count,
    ROUND(amount / SUM(amount) OVER (PARTITION BY region) * 100, 1) AS pct_of_region
FROM sales_data;
```

---

### 3.4 Running Totals

```sql
SUM(amount) OVER (
    ORDER BY sale_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS running_total
```

---

### 3.5 Moving Averages

```sql
-- 3-day moving average
AVG(daily_total) OVER (
    ORDER BY sale_date
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
) AS moving_avg_3day

-- 7-day
AVG(daily_total) OVER (
    ORDER BY sale_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
)

-- Centered 5-day (2 before + current + 2 after)
AVG(daily_total) OVER (
    ORDER BY sale_date
    ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING
)
```

---

### 3.6 LAG and LEAD

```sql
-- Access previous row value (offset=1, default_if_no_row=0)
LAG(amount, 1, 0)  OVER (ORDER BY sale_date)             AS prev_sale
LEAD(amount, 1, 0) OVER (ORDER BY sale_date)             AS next_sale

-- Same column, partition by person
LAG(amount) OVER (PARTITION BY salesperson ORDER BY sale_date) AS prev_sale_by_person

-- Month-over-month growth
ROUND(
    (revenue - LAG(revenue) OVER (ORDER BY month))
    / NULLIF(LAG(revenue) OVER (ORDER BY month), 0) * 100,
    1
) AS mom_growth_pct

-- Year-over-year (lag by 12 months)
LAG(revenue, 12) OVER (ORDER BY month) AS same_month_last_year
```

---

### 3.7 FIRST_VALUE, LAST_VALUE, NTH_VALUE

> **Important:** `LAST_VALUE` and `NTH_VALUE` require an explicit full-partition frame or you get the wrong result.

```sql
FIRST_VALUE(amount) OVER (
    PARTITION BY region ORDER BY amount DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
) AS highest_in_region

LAST_VALUE(amount) OVER (
    PARTITION BY region ORDER BY amount DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
) AS lowest_in_region

NTH_VALUE(amount, 2) OVER (
    PARTITION BY region ORDER BY amount DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
) AS second_highest
```

---

### 3.8 Hierarchical Queries (Oracle CONNECT BY)

```sql
-- Top-down org chart
SELECT
    LEVEL,
    LPAD(' ', (LEVEL - 1) * 4) || employee_name AS org_chart,
    job_title, salary
FROM employees_hier
START WITH  manager_id IS NULL
CONNECT BY PRIOR employee_id = manager_id
ORDER SIBLINGS BY employee_name;
```

**Key pseudocolumns / functions:**

| Element | Purpose |
|---------|---------|
| `LEVEL` | Depth (1 = root) |
| `ORDER SIBLINGS BY col` | Order within the same level |
| `CONNECT_BY_ISLEAF` | 1 if row has no children |
| `CONNECT_BY_ISCYCLE` | 1 if cycle detected |
| `SYS_CONNECT_BY_PATH(col, '/')` | Full path from root |
| `CONNECT_BY_ROOT col` | Root value for any row |

```sql
-- Full path example
SYS_CONNECT_BY_PATH(employee_name, ' → ') AS reporting_path

-- Root blocker
CONNECT_BY_ROOT employee_name AS ultimate_boss

-- Bottom-up (start from leaf, go up)
START WITH employee_name = 'Diana Lorentz'
CONNECT BY employee_id = PRIOR manager_id;   -- reversed PRIOR

-- Limit depth
WHERE LEVEL <= 3

-- Leaf nodes only
WHERE CONNECT_BY_ISLEAF = 1
```

---

### 3.9 Recursive CTEs

```sql
WITH org_hierarchy (employee_id, employee_name, manager_id, lvl, path) AS (
    -- Anchor (root)
    SELECT employee_id, employee_name, manager_id, 1, employee_name
    FROM employees_hier WHERE manager_id IS NULL

    UNION ALL

    -- Recursive member
    SELECT e.employee_id, e.employee_name, e.manager_id,
           h.lvl + 1,
           h.path || ' → ' || e.employee_name
    FROM employees_hier e
    JOIN org_hierarchy h ON e.manager_id = h.employee_id
)
SELECT lvl, LPAD(' ', (lvl-1)*4) || employee_name AS chart, path
FROM org_hierarchy ORDER BY path;
```

---

### 3.10 Subqueries

#### Correlated Subquery (executes once per outer row)
```sql
SELECT e.employee_name, e.salary,
       (SELECT AVG(salary) FROM employees_hier WHERE department = e.department) AS dept_avg
FROM employees_hier e
WHERE e.salary > (SELECT AVG(salary) FROM employees_hier WHERE department = e.department);
```

#### Inline View
```sql
SELECT e.employee_name, dept_stats.avg_salary
FROM employees_hier e
JOIN (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees_hier GROUP BY department
) dept_stats ON e.department = dept_stats.department;
```

#### Multi-Column Subquery
```sql
-- Rows where (department, salary) match the department's max salary
WHERE (department, salary) IN (
    SELECT department, MAX(salary) FROM employees_hier GROUP BY department
);
```

#### EXISTS vs IN
```sql
-- EXISTS: stops at first match — use for large subquery results
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id)

-- NOT EXISTS: safe with NULLs
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id)

-- NOT IN is DANGEROUS if subquery can contain NULLs (returns 0 rows)
-- Fix: WHERE customer_id NOT IN (SELECT customer_id FROM orders WHERE customer_id IS NOT NULL)
```

---

### 3.11 CTEs (WITH Clause)

```sql
-- Multiple CTEs (later ones can reference earlier ones)
WITH
dept_stats AS (
    SELECT department, AVG(salary) AS avg_salary, COUNT(*) AS emp_count
    FROM employees_hier GROUP BY department
),
company_stats AS (
    SELECT AVG(salary) AS company_avg FROM employees_hier
)
SELECT d.department, d.emp_count, d.avg_salary,
       CASE WHEN d.avg_salary > c.company_avg THEN 'Above Average' ELSE 'Below Average' END
FROM dept_stats d CROSS JOIN company_stats c;
```

---

### 3.12 MERGE (UPSERT)

```sql
MERGE INTO warehouse_inventory tgt
USING supplier_inventory src
ON (tgt.product_code = src.product_code)

WHEN MATCHED THEN
    UPDATE SET
        tgt.product_name  = src.product_name,
        tgt.quantity      = src.quantity,
        tgt.last_updated  = SYSDATE
    DELETE WHERE src.quantity = 0          -- optional delete clause

WHEN NOT MATCHED THEN
    INSERT (product_code, product_name, quantity, created_date)
    VALUES (src.product_code, src.product_name, src.quantity, SYSDATE)
    WHERE src.quantity > 0;                -- optional filter on insert
```

---

### 3.13 Multi-Table INSERT

```sql
-- INSERT ALL: every row goes to ALL matching targets
INSERT ALL
    WHEN type = 'SALE' THEN
        INTO sales_log (txn_id, txn_date, amount) VALUES (id, txn_date, amount)
    WHEN amount > 10000 THEN
        INTO high_value_transactions VALUES (id, txn_date, amount, 'REVIEW')
    WHEN 1=1 THEN                          -- always true = every row
        INTO audit_log VALUES (id, txn_date, amount, SYSDATE)
SELECT id, txn_date, amount, type FROM transactions;

-- INSERT FIRST: only first matching condition fires per row
INSERT FIRST
    WHEN amount > 10000 THEN INTO high_value_transactions VALUES (...)
    WHEN type = 'REFUND' THEN INTO refund_log VALUES (...)
    WHEN type = 'SALE'   THEN INTO sales_log VALUES (...)
    ELSE                      INTO other_transactions VALUES (...)
SELECT ... FROM transactions;
```

---

### 3.14 RETURNING Clause

```sql
-- Capture generated ID after INSERT
DECLARE v_new_id NUMBER;
BEGIN
    INSERT INTO customers (customer_id, customer_name, email, created_date)
    VALUES (customer_seq.NEXTVAL, 'New Customer', 'new@email.com', SYSDATE)
    RETURNING customer_id INTO v_new_id;
END;
/

-- Capture new value after UPDATE
UPDATE employees SET salary = salary * 1.1 WHERE employee_id = 104
RETURNING salary INTO v_new_salary;

-- Bulk delete with RETURNING
DELETE FROM temp_customers WHERE status = 'INACTIVE'
RETURNING customer_id, customer_name BULK COLLECT INTO v_ids, v_names;
```

---

### 3.15 PIVOT

```sql
SELECT *
FROM (
    SELECT region, TO_CHAR(sale_date, 'Mon') AS month, amount
    FROM sales_data
)
PIVOT (
    SUM(amount)
    FOR month IN ('Jan' AS Jan, 'Feb' AS Feb, 'Mar' AS Mar,
                  'Apr' AS Apr, 'May' AS May, 'Jun' AS Jun,
                  'Jul' AS Jul, 'Aug' AS Aug, 'Sep' AS Sep,
                  'Oct' AS Oct, 'Nov' AS Nov, 'Dec' AS Dec)
)
ORDER BY region;

-- Multiple aggregates
PIVOT (
    COUNT(*) AS cnt, SUM(amount) AS total
    FOR quarter IN ('1' AS Q1, '2' AS Q2, '3' AS Q3, '4' AS Q4)
)
```

---

### 3.16 UNPIVOT

```sql
-- Turn columns back into rows
SELECT *
FROM quarterly_results
UNPIVOT (
    sales_amount                                    -- new value column name
    FOR quarter                                     -- new category column name
    IN (q1_sales AS 'Q1', q2_sales AS 'Q2',
        q3_sales AS 'Q3', q4_sales AS 'Q4')
)
ORDER BY region, quarter;

-- Include NULLs (by default they are excluded)
UNPIVOT INCLUDE NULLS (sales_amount FOR quarter IN (...))
```

---

### 3.17 ROLLUP, CUBE, GROUPING SETS

```sql
-- ROLLUP: hierarchical subtotals (region → grand total)
GROUP BY ROLLUP(region, salesperson)

-- CUBE: every combination of subtotals
GROUP BY CUBE(region, product)

-- GROUPING SETS: only specific combinations
GROUP BY GROUPING SETS (
    (region, product),
    (region),
    ()                     -- grand total
)

-- GROUPING(): returns 1 when that column is part of the aggregation level
-- GROUPING_ID(): bitmap of which columns are aggregated
SELECT region, salesperson,
       SUM(amount),
       GROUPING(region)     AS is_region_subtotal,
       GROUPING_ID(region, salesperson) AS level
FROM sales_data
GROUP BY ROLLUP(region, salesperson);
```

---

## 4. PL/SQL Fundamentals

### 4.1 Block Structure

```sql
[<<label>>]
DECLARE
    -- variables, constants, cursors, types
BEGIN
    -- executable code
EXCEPTION
    WHEN exception_name THEN
        -- handler
END [label];
/
```

---

### 4.2 Variables & Types

```sql
DECLARE
    -- Basic declarations
    v_name          VARCHAR2(100);
    v_salary        NUMBER(10,2) := 0;
    v_is_active     BOOLEAN      := TRUE;
    v_date          DATE         := SYSDATE;

    -- Constants
    c_max_salary    CONSTANT NUMBER    := 999999;
    c_currency      CONSTANT VARCHAR2(3) := 'USD';

    -- NOT NULL (must initialize)
    v_required      VARCHAR2(50) NOT NULL := 'Required';

    -- Anchored to column type (stays in sync with schema changes)
    v_emp_id        employees.employee_id%TYPE;
    v_sal           employees.salary%TYPE;

    -- Anchored to whole row
    v_emp_row       employees%ROWTYPE;
BEGIN
    SELECT * INTO v_emp_row FROM employees WHERE employee_id = 100;
    DBMS_OUTPUT.PUT_LINE(v_emp_row.first_name || ': $' || v_emp_row.salary);
END;
/
```

---

### 4.3 Control Structures

```sql
-- IF / ELSIF / ELSE
IF v_salary >= 100000 THEN
    v_bonus := 0.20;
ELSIF v_salary >= 75000 THEN
    v_bonus := 0.15;
ELSE
    v_bonus := 0.05;
END IF;

-- CASE expression (returns a value)
v_gpa := CASE v_grade WHEN 'A' THEN 4.0 WHEN 'B' THEN 3.0 ELSE 0.0 END;

-- CASE statement (controls flow)
CASE v_status
    WHEN 'PENDING'   THEN DBMS_OUTPUT.PUT_LINE('Processing');
    WHEN 'SHIPPED'   THEN DBMS_OUTPUT.PUT_LINE('On its way');
    ELSE                  DBMS_OUTPUT.PUT_LINE('Unknown');
END CASE;

-- LOOP with EXIT WHEN
LOOP
    v_counter := v_counter + 1;
    EXIT WHEN v_counter >= 5;
END LOOP;

-- WHILE
WHILE v_counter <= 5 LOOP
    v_counter := v_counter + 1;
END LOOP;

-- FOR (numeric)
FOR i IN 1..5 LOOP ...  END LOOP;
FOR i IN REVERSE 1..5 LOOP ... END LOOP;

-- CONTINUE (skip even numbers)
FOR i IN 1..10 LOOP
    CONTINUE WHEN MOD(i, 2) = 0;
    DBMS_OUTPUT.PUT_LINE('Odd: ' || i);
END LOOP;

-- Nested loops with labels (EXIT outer loop)
<<outer_loop>>
FOR i IN 1..10 LOOP
    <<inner_loop>>
    FOR j IN 1..10 LOOP
        EXIT outer_loop WHEN i * j = 42;
    END LOOP inner_loop;
END LOOP outer_loop;
```

---

### 4.4 Cursors

#### Implicit Cursor Attributes
```sql
UPDATE employees SET salary = salary * 1.05 WHERE department_id = 60;
IF SQL%FOUND    THEN DBMS_OUTPUT.PUT_LINE('Rows updated: ' || SQL%ROWCOUNT); END IF;
IF SQL%NOTFOUND THEN DBMS_OUTPUT.PUT_LINE('No rows'); END IF;
```

#### Explicit Cursor (full lifecycle)
```sql
DECLARE
    CURSOR c_employees IS
        SELECT employee_id, first_name, salary FROM employees WHERE department_id = 60;
    v_emp c_employees%ROWTYPE;
BEGIN
    OPEN c_employees;
    LOOP
        FETCH c_employees INTO v_emp;
        EXIT WHEN c_employees%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_emp.first_name || ': $' || v_emp.salary);
    END LOOP;
    CLOSE c_employees;
END;
/
```

#### Cursor FOR Loop (simplest — open/fetch/close automatic)
```sql
-- Using declared cursor
FOR dept IN c_depts LOOP
    DBMS_OUTPUT.PUT_LINE(dept.department_name);
END LOOP;

-- Inline (no declaration needed)
FOR emp IN (SELECT first_name, salary FROM employees WHERE department_id = 60) LOOP
    DBMS_OUTPUT.PUT_LINE(emp.first_name);
END LOOP;
```

#### Parameterized Cursor
```sql
CURSOR c_dept_emps(p_dept_id NUMBER, p_min_salary NUMBER DEFAULT 0) IS
    SELECT first_name FROM employees
    WHERE department_id = p_dept_id AND salary >= p_min_salary;

FOR emp IN c_dept_emps(60)       LOOP ... END LOOP;
FOR emp IN c_dept_emps(60, 8000) LOOP ... END LOOP;
```

#### REF CURSOR (dynamic)
```sql
DECLARE
    TYPE emp_cursor IS REF CURSOR;
    c_emps emp_cursor;
    v_name VARCHAR2(100);
    v_sal  NUMBER;
BEGIN
    OPEN c_emps FOR SELECT first_name||' '||last_name, salary FROM employees;
    LOOP
        FETCH c_emps INTO v_name, v_sal;
        EXIT WHEN c_emps%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_name || ': $' || v_sal);
    END LOOP;
    CLOSE c_emps;
END;
/
```

#### BULK COLLECT (fetch all rows at once)
```sql
DECLARE
    TYPE emp_tab IS TABLE OF employees%ROWTYPE;
    v_employees emp_tab;
BEGIN
    SELECT * BULK COLLECT INTO v_employees FROM employees WHERE department_id = 60;
    FOR i IN 1..v_employees.COUNT LOOP
        DBMS_OUTPUT.PUT_LINE(v_employees(i).first_name);
    END LOOP;
END;
/

-- With LIMIT to process in batches
FETCH c_all BULK COLLECT INTO v_batch LIMIT 100;
```

---

### 4.5 Exception Handling

```sql
DECLARE
    e_invalid_salary EXCEPTION;              -- user-defined exception
    e_fk_violation   EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_fk_violation, -2292);  -- map to ORA error code
BEGIN
    IF v_salary < 0 THEN RAISE e_invalid_salary; END IF;

    RAISE_APPLICATION_ERROR(-20001, 'Custom error message');  -- -20000 to -20999

EXCEPTION
    WHEN NO_DATA_FOUND        THEN DBMS_OUTPUT.PUT_LINE('No row found');
    WHEN TOO_MANY_ROWS        THEN DBMS_OUTPUT.PUT_LINE('Multiple rows');
    WHEN DUP_VAL_ON_INDEX     THEN DBMS_OUTPUT.PUT_LINE('Duplicate key');
    WHEN e_invalid_salary     THEN DBMS_OUTPUT.PUT_LINE('Bad salary');
    WHEN e_fk_violation       THEN DBMS_OUTPUT.PUT_LINE('FK violated');
    WHEN OTHERS               THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
        DBMS_OUTPUT.PUT_LINE('Code: '  || SQLCODE);
        DBMS_OUTPUT.PUT_LINE(DBMS_UTILITY.FORMAT_ERROR_BACKTRACE);
END;
/
```

**Common predefined exceptions:**

| Exception | ORA Code | Trigger |
|-----------|----------|---------|
| `NO_DATA_FOUND` | -1403 | SELECT INTO returned no rows |
| `TOO_MANY_ROWS` | -1422 | SELECT INTO returned > 1 row |
| `ZERO_DIVIDE` | -1476 | Division by zero |
| `VALUE_ERROR` | -6502 | Type conversion failure |
| `DUP_VAL_ON_INDEX` | -0001 | Unique constraint violated |

---

### 4.6 Collections

#### Associative Array (INDEX BY)
```sql
TYPE t_salary_tab IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
TYPE t_name_tab   IS TABLE OF VARCHAR2(100) INDEX BY VARCHAR2(20);

v_salaries t_salary_tab;
v_salaries(100) := 50000;

-- Iterate string-indexed array
v_key := v_names.FIRST;
WHILE v_key IS NOT NULL LOOP
    DBMS_OUTPUT.PUT_LINE(v_key || ': ' || v_names(v_key));
    v_key := v_names.NEXT(v_key);
END LOOP;
```

#### Nested Table
```sql
TYPE t_list IS TABLE OF NUMBER;
v_nums t_list := t_list(10, 20, 30);

v_nums.EXTEND;          -- add one element slot
v_nums(4) := 40;
v_nums.DELETE(2);       -- delete element at index 2

FOR i IN 1..v_nums.LAST LOOP
    IF v_nums.EXISTS(i) THEN
        DBMS_OUTPUT.PUT_LINE(v_nums(i));
    END IF;
END LOOP;
```

#### VARRAY (fixed max size)
```sql
TYPE t_phones IS VARRAY(5) OF VARCHAR2(20);
v_phones t_phones := t_phones('555-1111', '555-2222');
v_phones.EXTEND;
v_phones(3) := '555-3333';
DBMS_OUTPUT.PUT_LINE('Limit: ' || v_phones.LIMIT);
```

**Collection methods:**
`COUNT`, `FIRST`, `LAST`, `NEXT(n)`, `PRIOR(n)`, `EXISTS(n)`, `EXTEND`, `TRIM`, `DELETE`

---

### 4.7 BULK Operations

```sql
-- FORALL: single DML over collection (much faster than row-by-row)
DECLARE
    TYPE t_id_tab  IS TABLE OF NUMBER;
    TYPE t_sal_tab IS TABLE OF NUMBER;
    v_ids      t_id_tab;
    v_salaries t_sal_tab;
BEGIN
    SELECT employee_id, salary * 1.05
    BULK COLLECT INTO v_ids, v_salaries
    FROM employees WHERE department_id = 60;

    FORALL i IN 1..v_ids.COUNT
        UPDATE employees SET salary = v_salaries(i) WHERE employee_id = v_ids(i);

    COMMIT;
END;
/

-- FORALL with SAVE EXCEPTIONS (continue despite errors)
DECLARE
    e_bulk_errors EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_bulk_errors, -24381);
BEGIN
    FORALL i IN 1..v_ids.COUNT SAVE EXCEPTIONS
        DELETE FROM employees WHERE employee_id = v_ids(i);
EXCEPTION
    WHEN e_bulk_errors THEN
        FOR j IN 1..SQL%BULK_EXCEPTIONS.COUNT LOOP
            DBMS_OUTPUT.PUT_LINE('Error at ' || SQL%BULK_EXCEPTIONS(j).ERROR_INDEX ||
                                 ': ' || SQLERRM(-SQL%BULK_EXCEPTIONS(j).ERROR_CODE));
        END LOOP;
END;
/

-- FORALL with INDICES OF (handles sparse arrays)
FORALL i IN INDICES OF v_emp_ids
    UPDATE employees SET commission_pct = 0.15 WHERE employee_id = v_emp_ids(i);
```

---

## 5. PL/SQL Database Objects

### 5.1 Stored Procedures

```sql
CREATE OR REPLACE PROCEDURE hire_employee(
    p_first_name    IN  VARCHAR2,
    p_last_name     IN  VARCHAR2,
    p_salary        IN  NUMBER,
    p_department_id IN  NUMBER,
    p_employee_id   OUT NUMBER        -- output parameter
) IS
    v_min_salary NUMBER;
    v_max_salary NUMBER;
BEGIN
    SELECT min_salary, max_salary INTO v_min_salary, v_max_salary
    FROM jobs WHERE job_id = 'IT_PROG';

    IF p_salary NOT BETWEEN v_min_salary AND v_max_salary THEN
        RAISE_APPLICATION_ERROR(-20001, 'Salary out of range');
    END IF;

    SELECT employees_seq.NEXTVAL INTO p_employee_id FROM dual;

    INSERT INTO employees (employee_id, first_name, last_name, salary, department_id, hire_date)
    VALUES (p_employee_id, p_first_name, p_last_name, p_salary, p_department_id, SYSDATE);

    COMMIT;
EXCEPTION
    WHEN DUP_VAL_ON_INDEX THEN RAISE_APPLICATION_ERROR(-20003, 'Duplicate record');
    WHEN OTHERS THEN ROLLBACK; RAISE;
END hire_employee;
/

-- Call: positional notation
DECLARE v_id NUMBER;
BEGIN hire_employee('John', 'Smith', 6000, 60, v_id); END;
/

-- Call: named notation (order doesn't matter)
BEGIN hire_employee(p_first_name => 'Jane', p_last_name => 'Doe',
                    p_salary => 7000, p_department_id => 80, p_employee_id => v_id); END;
/
```

**Parameter modes:**

| Mode | Read | Write | Notes |
|------|------|-------|-------|
| `IN` (default) | ✓ | ✗ | Can have default value |
| `OUT` | ✗ | ✓ | Must be written before procedure ends |
| `IN OUT` | ✓ | ✓ | Pass in, modify, return |

**NOCOPY hint** (pass by reference, not copy — faster for large collections):
```sql
PROCEDURE process(p_data IN OUT NOCOPY large_collection_type) IS ...
```

---

### 5.2 Stored Functions

```sql
CREATE OR REPLACE FUNCTION get_annual_salary(p_employee_id IN NUMBER)
RETURN NUMBER IS
    v_salary NUMBER;
BEGIN
    SELECT salary * 12 INTO v_salary FROM employees WHERE employee_id = p_employee_id;
    RETURN v_salary;
EXCEPTION
    WHEN NO_DATA_FOUND THEN RETURN NULL;
END get_annual_salary;
/

-- DETERMINISTIC: same inputs always yield same output; Oracle can cache results
CREATE OR REPLACE FUNCTION get_tax_rate(p_income NUMBER) RETURN NUMBER
DETERMINISTIC IS
BEGIN
    RETURN CASE WHEN p_income > 500000 THEN 0.35 ELSE 0.15 END;
END;
/

-- RESULT_CACHE: Oracle caches across sessions
CREATE OR REPLACE FUNCTION get_config(p_key VARCHAR2) RETURN VARCHAR2
RESULT_CACHE RELIES_ON (configuration) IS
    v_value VARCHAR2(4000);
BEGIN
    SELECT config_value INTO v_value FROM configuration WHERE config_key = p_key;
    RETURN v_value;
END;
/
```

#### Pipelined Table Function (returns rows one-at-a-time)
```sql
CREATE OR REPLACE TYPE emp_row AS OBJECT (employee_id NUMBER, name VARCHAR2(100), salary NUMBER);
/
CREATE OR REPLACE TYPE emp_table AS TABLE OF emp_row;
/

CREATE OR REPLACE FUNCTION get_high_earners(p_min NUMBER DEFAULT 10000)
RETURN emp_table PIPELINED IS
BEGIN
    FOR r IN (SELECT employee_id, first_name||' '||last_name, salary
              FROM employees WHERE salary >= p_min)
    LOOP
        PIPE ROW(emp_row(r.employee_id, r.name, r.salary));
    END LOOP;
    RETURN;
END;
/

-- Use like a table
SELECT * FROM TABLE(get_high_earners(15000));
```

---

### 5.3 Packages

```sql
-- SPECIFICATION (public interface)
CREATE OR REPLACE PACKAGE emp_mgmt AS
    TYPE t_emp_rec IS RECORD (employee_id NUMBER, name VARCHAR2(100), salary NUMBER);
    TYPE t_emp_tab IS TABLE OF t_emp_rec INDEX BY PLS_INTEGER;

    c_max_salary      CONSTANT NUMBER := 100000;
    e_salary_exceeded EXCEPTION;

    PROCEDURE hire_employee(p_name VARCHAR2, p_salary NUMBER, p_emp_id OUT NUMBER);
    PROCEDURE terminate_employee(p_emp_id NUMBER);
    FUNCTION  get_employee_count(p_dept_id NUMBER) RETURN NUMBER;
END emp_mgmt;
/

-- BODY (implementation)
CREATE OR REPLACE PACKAGE BODY emp_mgmt AS
    g_last_emp_id NUMBER;    -- private package-level variable (persists for session)

    PROCEDURE log_activity(p_emp_id NUMBER, p_action VARCHAR2) IS  -- private
        PRAGMA AUTONOMOUS_TRANSACTION;
    BEGIN
        INSERT INTO activity_log VALUES (p_emp_id, p_action, SYSDATE, USER);
        COMMIT;
    END;

    PROCEDURE hire_employee(p_name VARCHAR2, p_salary NUMBER, p_emp_id OUT NUMBER) IS
    BEGIN
        IF p_salary > c_max_salary THEN RAISE e_salary_exceeded; END IF;
        SELECT emp_seq.NEXTVAL INTO p_emp_id FROM dual;
        INSERT INTO employees (employee_id, employee_name, salary)
        VALUES (p_emp_id, p_name, p_salary);
        g_last_emp_id := p_emp_id;
        log_activity(p_emp_id, 'HIRE');
        COMMIT;
    END;

    FUNCTION get_employee_count(p_dept_id NUMBER) RETURN NUMBER IS
        v_count NUMBER;
    BEGIN
        SELECT COUNT(*) INTO v_count FROM employees WHERE department_id = p_dept_id;
        RETURN v_count;
    END;
-- Initialization block (runs once per session when package first accessed)
BEGIN
    g_last_emp_id := 0;
END emp_mgmt;
/

-- Usage
DECLARE v_id NUMBER;
BEGIN
    emp_mgmt.hire_employee('John Smith', 50000, v_id);
    DBMS_OUTPUT.PUT_LINE('Count: ' || emp_mgmt.get_employee_count(60));
EXCEPTION
    WHEN emp_mgmt.e_salary_exceeded THEN DBMS_OUTPUT.PUT_LINE('Salary too high');
END;
/
```

**Overloading** (same name, different parameter signatures):
```sql
CREATE OR REPLACE PACKAGE util_pkg AS
    FUNCTION format_name(p_first VARCHAR2, p_last VARCHAR2) RETURN VARCHAR2;
    FUNCTION format_name(p_employee_id NUMBER)              RETURN VARCHAR2;
END;
/
```

---

### 5.4 Triggers

#### BEFORE trigger (validation + auto-fill)
```sql
CREATE OR REPLACE TRIGGER trg_employee_validation
BEFORE INSERT OR UPDATE ON employees
FOR EACH ROW
DECLARE
    v_min NUMBER; v_max NUMBER;
BEGIN
    -- Auto-generate PK on INSERT
    IF INSERTING AND :NEW.employee_id IS NULL THEN
        :NEW.employee_id := employee_seq.NEXTVAL;
    END IF;

    -- Timestamps
    IF INSERTING THEN
        :NEW.created_date := SYSDATE;
        :NEW.created_by   := USER;
    END IF;
    :NEW.modified_date := SYSDATE;
    :NEW.modified_by   := USER;

    -- Salary validation
    SELECT min_salary, max_salary INTO v_min, v_max FROM jobs WHERE job_id = :NEW.job_id;
    IF :NEW.salary NOT BETWEEN v_min AND v_max THEN
        RAISE_APPLICATION_ERROR(-20001, 'Salary out of range');
    END IF;

    -- Standardize email
    :NEW.email := LOWER(TRIM(:NEW.email));
END;
/
```

#### AFTER trigger (auditing)
```sql
CREATE OR REPLACE TRIGGER trg_salary_audit
AFTER UPDATE OF salary ON employees
FOR EACH ROW
BEGIN
    INSERT INTO salary_audit (employee_id, old_salary, new_salary, changed_by, changed_date)
    VALUES (:NEW.employee_id, :OLD.salary, :NEW.salary, USER, SYSDATE);
END;
/
```

**`:OLD` and `:NEW` availability:**

| Operation | `:OLD` | `:NEW` |
|-----------|--------|--------|
| INSERT | NULL | New values |
| UPDATE | Original | New values |
| DELETE | Original | NULL |

**INSERTING / UPDATING / DELETING predicates:**
```sql
v_op := CASE WHEN INSERTING THEN 'INSERT' WHEN UPDATING THEN 'UPDATE' ELSE 'DELETE' END;
```

#### INSTEAD OF trigger (make views updatable)
```sql
CREATE OR REPLACE TRIGGER trg_emp_dept_insert
INSTEAD OF INSERT ON emp_dept_view
FOR EACH ROW
DECLARE v_dept_id NUMBER;
BEGIN
    SELECT department_id INTO v_dept_id FROM departments
    WHERE department_name = :NEW.department_name;
    INSERT INTO employees (employee_id, employee_name, salary, department_id)
    VALUES (employee_seq.NEXTVAL, :NEW.employee_name, :NEW.salary, v_dept_id);
END;
/
```

#### Compound Trigger (avoids mutating table error; efficient bulk auditing)
```sql
CREATE OR REPLACE TRIGGER trg_salary_compound
FOR UPDATE OF salary ON employees
COMPOUND TRIGGER
    TYPE t_audit IS TABLE OF salary_audit%ROWTYPE INDEX BY PLS_INTEGER;
    g_audits t_audit;
    g_idx    PLS_INTEGER := 0;

    AFTER EACH ROW IS
    BEGIN
        g_idx := g_idx + 1;
        g_audits(g_idx).employee_id := :NEW.employee_id;
        g_audits(g_idx).old_salary  := :OLD.salary;
        g_audits(g_idx).new_salary  := :NEW.salary;
    END AFTER EACH ROW;

    AFTER STATEMENT IS
    BEGIN
        FORALL i IN 1..g_audits.COUNT
            INSERT INTO salary_audit VALUES g_audits(i);
    END AFTER STATEMENT;
END;
/
```

#### DDL and System Triggers
```sql
-- Audit DDL
CREATE OR REPLACE TRIGGER trg_ddl_audit AFTER DDL ON SCHEMA
BEGIN
    INSERT INTO ddl_audit_log VALUES (ddl_seq.NEXTVAL, ORA_SYSEVENT, ORA_DICT_OBJ_TYPE,
                                      ORA_DICT_OBJ_NAME, USER, SYSTIMESTAMP);
END;
/

-- Prevent drops
CREATE OR REPLACE TRIGGER trg_prevent_drop BEFORE DROP ON SCHEMA
BEGIN
    IF ORA_DICT_OBJ_TYPE = 'TABLE' AND ORA_DICT_OBJ_NAME IN ('EMPLOYEES') THEN
        RAISE_APPLICATION_ERROR(-20001, 'Cannot drop critical table');
    END IF;
END;
/

-- LOGON trigger
CREATE OR REPLACE TRIGGER trg_after_logon AFTER LOGON ON DATABASE
BEGIN
    INSERT INTO user_login_log VALUES (login_seq.NEXTVAL, USER, SYSTIMESTAMP,
                                       SYS_CONTEXT('USERENV', 'IP_ADDRESS'));
    COMMIT;
EXCEPTION WHEN OTHERS THEN NULL;
END;
/
```

#### Trigger Management
```sql
ALTER TRIGGER trg_salary_audit DISABLE;
ALTER TRIGGER trg_salary_audit ENABLE;
ALTER TABLE employees DISABLE ALL TRIGGERS;
ALTER TABLE employees ENABLE ALL TRIGGERS;
DROP TRIGGER trg_salary_audit;
```

---

### 5.5 Transaction Management

```sql
-- Savepoints
BEGIN
    INSERT INTO orders VALUES (...);
    SAVEPOINT after_order;

    INSERT INTO order_items VALUES (...);

    ROLLBACK TO after_order;   -- undo order_items, keep orders insert
    COMMIT;                    -- make permanent
END;
/

-- Autonomous Transaction (commits independently of the caller)
PROCEDURE log_error(p_msg VARCHAR2) IS
    PRAGMA AUTONOMOUS_TRANSACTION;
BEGIN
    INSERT INTO error_log VALUES (SYSDATE, p_msg, USER);
    COMMIT;   -- only commits this insert, even if caller rolls back
END;
/
```

---

## 6. Concurrency & Locking

### 6.1 SELECT FOR UPDATE

```sql
-- Basic (waits indefinitely)
SELECT employee_id, salary FROM employees WHERE department_id = 60 FOR UPDATE;

-- Specific column
SELECT * FROM employees WHERE employee_id = 100 FOR UPDATE OF salary;

-- NOWAIT: fail immediately if locked (ORA-00054)
SELECT * FROM employees WHERE employee_id = 100 FOR UPDATE NOWAIT;

-- WAIT n: wait up to n seconds (ORA-30006 on timeout)
SELECT * FROM employees WHERE employee_id = 100 FOR UPDATE WAIT 5;

-- SKIP LOCKED: skip rows locked by others (queue pattern)
SELECT job_id, payload FROM job_queue WHERE status = 'PENDING'
ORDER BY priority FOR UPDATE SKIP LOCKED FETCH FIRST 10 ROWS ONLY;
```

#### WHERE CURRENT OF (update/delete cursor row)
```sql
DECLARE CURSOR c IS SELECT employee_id, salary FROM employees FOR UPDATE OF salary;
BEGIN
    FOR emp IN c LOOP
        UPDATE employees SET salary = emp.salary * 1.10 WHERE CURRENT OF c;
    END LOOP;
    COMMIT;
END;
/
```

---

### 6.2 Explicit Table Locking

```sql
LOCK TABLE employees IN SHARE MODE;
LOCK TABLE employees IN EXCLUSIVE MODE;
LOCK TABLE employees IN EXCLUSIVE MODE NOWAIT;
LOCK TABLE employees IN EXCLUSIVE MODE WAIT 10;
```

---

### 6.3 Deadlock Prevention

```sql
-- Strategy 1: Always lock in consistent order (e.g., lower account_id first)
IF p_from_account < p_to_account THEN
    first  := p_from_account; second := p_to_account;
ELSE
    first  := p_to_account;   second := p_from_account;
END IF;
SELECT balance INTO v FROM accounts WHERE account_id = first  FOR UPDATE;
SELECT balance INTO v FROM accounts WHERE account_id = second FOR UPDATE;
```

```sql
-- Strategy 2: NOWAIT with exponential backoff retry
DECLARE
    e_resource_busy EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_resource_busy, -54);
    e_deadlock EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_deadlock, -60);
    v_retries NUMBER := 0;
BEGIN
    WHILE v_retries < 3 LOOP
        BEGIN
            UPDATE employees SET salary = ... WHERE employee_id = ...;
            COMMIT;
            EXIT;
        EXCEPTION
            WHEN e_resource_busy OR e_deadlock THEN
                ROLLBACK;
                v_retries := v_retries + 1;
                DBMS_LOCK.SLEEP(POWER(2, v_retries) * 0.1);
        END;
    END LOOP;
END;
/
```

---

### 6.4 Isolation Levels

```sql
-- Set for a transaction
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Set for a session
ALTER SESSION SET ISOLATION_LEVEL = SERIALIZABLE;

-- Handle serialization failure
DECLARE
    e_serial EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_serial, -8177);
BEGIN
    SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    -- ... DML ...
    COMMIT;
EXCEPTION
    WHEN e_serial THEN ROLLBACK; -- retry
END;
/
```

---

### 6.5 Finding and Killing Blockers

```sql
-- Blocked sessions
SELECT s1.sid AS blocked, s1.username, s2.sid AS blocker, s2.username,
       s1.seconds_in_wait, s1.event
FROM v$session s1
JOIN v$session s2 ON s1.blocking_session = s2.sid;

-- Kill session
ALTER SYSTEM KILL SESSION '142,5678';
ALTER SYSTEM KILL SESSION '142,5678' IMMEDIATE;

-- Cancel just the SQL (12c+)
ALTER SYSTEM CANCEL SQL '142,5678';
```

---

### 6.6 Dynamic SQL (EXECUTE IMMEDIATE)

```sql
-- Query with bind variable (ALWAYS use binds — prevents SQL injection)
EXECUTE IMMEDIATE 'SELECT COUNT(*) FROM employees WHERE department_id = :1'
    INTO v_count USING p_dept_id;

-- DML
EXECUTE IMMEDIATE 'UPDATE ' || v_table || ' SET status = :1 WHERE id = :2'
    USING 'ACTIVE', v_id;

-- DDL (no bind variables possible)
EXECUTE IMMEDIATE 'CREATE TABLE ' || v_table_name || ' AS SELECT * FROM source WHERE 1=0';

-- Dynamic cursor
OPEN p_results FOR 'SELECT * FROM employees WHERE last_name = :name' USING p_name;
```

---

## 7. SQL Testing & Validation

### 7.1 Data Reconciliation

```sql
-- Aggregate comparison
SELECT 'SOURCE' AS sys, COUNT(*) AS records, SUM(order_total) AS total FROM source_orders
UNION ALL
SELECT 'TARGET',        COUNT(*),             SUM(order_total)             FROM target_orders;

-- Records in source but NOT target
SELECT 'SOURCE_ONLY' AS status, s.*
FROM source_orders s
WHERE NOT EXISTS (SELECT 1 FROM target_orders t WHERE t.order_id = s.order_id);

-- FULL OUTER JOIN to see all discrepancies
SELECT COALESCE(s.order_id, t.order_id) AS order_id,
       CASE WHEN s.order_id IS NULL THEN 'TARGET_ONLY'
            WHEN t.order_id IS NULL THEN 'SOURCE_ONLY'
            WHEN s.order_total != t.order_total THEN 'AMOUNT_MISMATCH'
            ELSE 'MATCH' END AS status
FROM source_orders s FULL OUTER JOIN target_orders t ON s.order_id = t.order_id
WHERE s.order_id IS NULL OR t.order_id IS NULL OR s.order_total != t.order_total;
```

---

### 7.2 Checksums

```sql
-- Row-level hash
SELECT order_id,
       ORA_HASH(order_id || '|' || customer_id || '|' || order_total || '|' || status) AS row_hash
FROM orders;

-- Compare hashes
SELECT COALESCE(s.order_id, t.order_id) AS order_id,
       CASE WHEN s.row_hash IS NULL THEN 'MISSING_IN_SOURCE'
            WHEN t.row_hash IS NULL THEN 'MISSING_IN_TARGET'
            WHEN s.row_hash != t.row_hash THEN 'DATA_DIFFERS'
            ELSE 'MATCH' END AS status
FROM (SELECT order_id, ORA_HASH(order_id||customer_id||order_total) AS row_hash FROM source_orders) s
FULL JOIN (SELECT order_id, ORA_HASH(order_id||customer_id||order_total) AS row_hash FROM target_orders) t
ON s.order_id = t.order_id
WHERE s.row_hash IS NULL OR t.row_hash IS NULL OR s.row_hash != t.row_hash;
```

---

### 7.3 Null Analysis

```sql
-- Count nulls per column
SELECT 'email' AS column_name,
       COUNT(*) AS total,
       COUNT(*) - COUNT(email) AS null_count,
       ROUND((COUNT(*) - COUNT(email)) * 100.0 / COUNT(*), 2) AS null_pct
FROM customers;

-- Three-valued logic reminder:
-- NULL = NULL → UNKNOWN  (use IS NULL, not = NULL)
-- COUNT(*) counts all rows; COUNT(col) skips NULLs
```

---

### 7.4 Duplicate Detection

```sql
-- Exact duplicates on a key
SELECT email, COUNT(*) AS occurrences,
       LISTAGG(customer_id, ', ') WITHIN GROUP (ORDER BY customer_id) AS ids
FROM customers WHERE email IS NOT NULL
GROUP BY email HAVING COUNT(*) > 1;

-- Using ROW_NUMBER (returns every row that's a duplicate except the first)
SELECT * FROM (
    SELECT c.*, ROW_NUMBER() OVER (PARTITION BY email ORDER BY customer_id) AS rn
    FROM customers c
) WHERE rn > 1;

-- Fuzzy match with SOUNDEX
SELECT c1.customer_id, c1.last_name, c2.customer_id AS match_id, c2.last_name AS match
FROM customers c1
JOIN customers c2 ON SOUNDEX(c1.last_name) = SOUNDEX(c2.last_name)
WHERE c1.customer_id < c2.customer_id AND c1.last_name != c2.last_name;

-- Similarity score with UTL_MATCH (> 85 = likely same person)
WHERE UTL_MATCH.JARO_WINKLER_SIMILARITY(c1.last_name, c2.last_name) > 85
```

---

### 7.5 Referential Integrity Checks

```sql
-- Orphan orders (no matching customer)
SELECT 'Orders without customers' AS issue, COUNT(*) AS count
FROM orders o WHERE NOT EXISTS (SELECT 1 FROM customers c WHERE c.customer_id = o.customer_id);

-- Multi-table orphan check
WITH checks AS (
    SELECT 'orders → customers' AS rel, o.order_id AS id
    FROM orders o WHERE NOT EXISTS (SELECT 1 FROM customers c WHERE c.customer_id = o.customer_id)
    UNION ALL
    SELECT 'order_items → orders', oi.item_id
    FROM order_items oi WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.order_id = oi.order_id)
)
SELECT rel, COUNT(*) AS orphan_count FROM checks GROUP BY rel;
```

---

### 7.6 Business Rule Validation

```sql
-- Intra-record date sequence
SELECT order_id, order_date, ship_date, delivery_date,
       CASE WHEN ship_date < order_date     THEN 'Ship before order'
            WHEN delivery_date < ship_date  THEN 'Delivery before ship'
            ELSE 'VALID' END AS check
FROM orders WHERE ship_date < order_date OR delivery_date < ship_date;

-- Inter-record: order total = sum of line items
SELECT o.order_id, o.order_total,
       SUM(oi.quantity * oi.unit_price)             AS calculated,
       ABS(o.order_total - SUM(oi.quantity * oi.unit_price)) AS diff
FROM orders o JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id, o.order_total
HAVING ABS(o.order_total - SUM(oi.quantity * oi.unit_price)) > 0.01;
```

---

### 7.7 Regression Testing Pattern

```sql
-- Capture baseline snapshot
CREATE TABLE snapshot_orders AS
SELECT order_id, customer_id, order_total, status,
       ORA_HASH(order_id || customer_id || order_total || status) AS row_hash,
       SYSDATE AS snapshot_date
FROM orders;

-- Post-release comparison
SELECT COALESCE(s.order_id, o.order_id) AS order_id,
       CASE WHEN s.order_id IS NULL THEN 'NEW'
            WHEN o.order_id IS NULL THEN 'DELETED'
            WHEN s.row_hash != ORA_HASH(o.order_id||o.customer_id||o.order_total||o.status) THEN 'MODIFIED'
            ELSE 'UNCHANGED' END AS change_type
FROM snapshot_orders s FULL JOIN orders o ON s.order_id = o.order_id
WHERE s.order_id IS NULL OR o.order_id IS NULL
   OR s.row_hash != ORA_HASH(o.order_id||o.customer_id||o.order_total||o.status);

-- Aggregate baseline comparison (flag >1% variance)
SELECT b.table_name, b.metric, b.value AS baseline, c.current_value,
       ROUND(ABS(c.current_value - b.value) / NULLIF(b.value, 0) * 100, 2) AS variance_pct
FROM aggregate_baseline b
JOIN (
    SELECT 'ORDERS' AS table_name, 'COUNT' AS metric, COUNT(*) AS current_value FROM orders
    UNION ALL SELECT 'ORDERS', 'SUM', SUM(order_total) FROM orders
) c ON b.table_name = c.table_name AND b.metric = c.metric
WHERE ABS(c.current_value - b.value) / NULLIF(b.value, 0) > 0.01;
```

---

## Quick-Reference Summary

### Platform Decision Matrix

| Need | Use |
|------|-----|
| Speed up Oracle query on filtered column | Create B-Tree index on that column |
| Function on filtered column | Function-based index |
| Low-cardinality column in DW | Bitmap index (never OLTP!) |
| Snowflake large table + date filter | `CLUSTER BY (date_col)` |
| Databricks point lookup on high-cardinality column | `CREATE BLOOMFILTER INDEX` |
| Databricks many small files | `OPTIMIZE table` |
| Databricks mixed filters (date + user_id) | Partition by date, then `ZORDER BY user_id` |

### Window Function Cheat-Sheet

| Goal | Syntax |
|------|--------|
| Unique row number | `ROW_NUMBER() OVER (PARTITION BY g ORDER BY s)` |
| Rank with gaps | `RANK() OVER (ORDER BY s DESC)` |
| Rank without gaps | `DENSE_RANK() OVER (ORDER BY s DESC)` |
| Running total | `SUM(c) OVER (ORDER BY d ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` |
| 3-row moving average | `AVG(c) OVER (ORDER BY d ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)` |
| Previous row value | `LAG(c, 1, 0) OVER (ORDER BY d)` |
| Next row value | `LEAD(c, 1, 0) OVER (ORDER BY d)` |
| YoY comparison | `LAG(rev, 12) OVER (ORDER BY month)` |

### PL/SQL Quick-Reference

```sql
-- Block             DECLARE ... BEGIN ... EXCEPTION ... END;
-- Cursor FOR loop   FOR rec IN (SELECT ...) LOOP ... END LOOP;
-- Bulk fetch        SELECT col BULK COLLECT INTO v_tab FROM t;
-- Bulk DML          FORALL i IN 1..n UPDATE t SET col = v_tab(i) WHERE id = v_ids(i);
-- Procedure         CREATE PROCEDURE name(p IN type, p_out OUT type) IS BEGIN ... END;
-- Function          CREATE FUNCTION name(p type) RETURN type IS BEGIN RETURN val; END;
-- Trigger           CREATE TRIGGER name BEFORE|AFTER event ON table [FOR EACH ROW] BEGIN ... END;
-- Dynamic SQL       EXECUTE IMMEDIATE 'sql' [INTO vars] [USING binds];
-- Savepoint         SAVEPOINT sp_name;  ROLLBACK TO sp_name;
-- Autonomous txn    PRAGMA AUTONOMOUS_TRANSACTION; inside procedure/function
```


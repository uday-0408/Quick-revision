# ❄️ SNOWFLAKE COMPLETE ENGINEERING CHEAT-SHEET
### Senior Data Engineer · Certification · Interview · Production Reference

---

> **How to use this document:** Every topic follows the same structure — What It Is → Internal Working → SQL Syntax → Exam Traps → Interview Answers → Optimization → Troubleshooting. Use `Ctrl+F` to jump to any topic.

---

# TABLE OF CONTENTS

| # | Topic |
|---|-------|
| 1 | AI Data Cloud Features and Architecture |
| 2 | Tools and User Interface |
| 3 | Catalog and Objects |
| 4 | Storage Concepts |
| 5 | Authentication Framework |
| 6 | Managing User Roles and Permissions |
| 7 | Data Governance and Protection |
| 8 | Query Performance and Execution |
| 9 | Virtual Warehouse Sizing and Scaling |
| 10 | Cost Management and Governance |
| 11 | Storage Optimization |
| 12 | Data Loading Performance |
| 13 | Data Loading Foundations |
| 14 | Data Unloading Best Practices |
| 15 | Advanced SQL Transformations |
| 16 | Processing Semi-Structured Data |
| 17 | Extending Snowflake |
| 18 | Data Protection and Point Recovery |
| 19 | Secure Data Sharing and Collaboration |

---

# 1. AI DATA CLOUD FEATURES AND ARCHITECTURE

## 1.1 What It Is

Snowflake is a **cloud-native, multi-cluster shared data architecture** — NOT a lift-and-shift of an on-prem database. It was built from scratch to separate storage, compute, and cloud services into three independent, elastically scalable layers.

**The "AI Data Cloud"** is Snowflake's brand for its unified platform that includes:
- Data warehousing (structured/semi-structured)
- Data lake capabilities
- Data sharing across organizations
- ML/AI workloads (Snowpark, Cortex AI)
- Application development (Snowflake Native Apps)
- Collaboration (Snowflake Marketplace)

**Why companies use it:**
- Zero copy cloning → dev/test without storage cost
- Time Travel → instant point-in-time recovery
- Separation of compute from storage → pay only for what you use
- Multi-cloud (AWS, Azure, GCP) in one platform

---

## 1.2 Internal Architecture — Three-Layer Model

```text
┌─────────────────────────────────────────────────────┐
│              CLOUD SERVICES LAYER                    │
│  Authentication · Query Optimizer · Metadata        │
│  Security · Transaction Manager · Result Cache      │
└────────────────────────┬────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────┐
│              QUERY PROCESSING LAYER                  │
│         Virtual Warehouses (Compute Clusters)        │
│   XS → S → M → L → XL → 2XL → 3XL → 4XL           │
│   Each = independent MPP cluster of EC2/VMs         │
└────────────────────────┬────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────┐
│              DATABASE STORAGE LAYER                  │
│   Micro-Partitions (columnar compressed files)      │
│   Stored in cloud blob storage (S3/Azure/GCS)       │
│   Owned and managed by Snowflake                    │
└─────────────────────────────────────────────────────┘
```

### Layer Deep Dive

| Layer | What It Does | Who Controls It | Cost Driver |
|-------|-------------|-----------------|-------------|
| Cloud Services | Auth, compile, optimize, metadata | Snowflake (always on) | Serverless — small % of compute |
| Virtual Warehouse | CPU, RAM, execution | You (start/stop/resize) | Credits/hour |
| Storage | Store compressed micro-partitions | Snowflake (transparent) | $/TB/month |

---

## 1.3 Snowflake Editions

| Edition | Key Features | Use Case |
|---------|-------------|----------|
| **Standard** | Core DW, Time Travel 1 day | Development, SMB |
| **Enterprise** | Multi-cluster WH, Time Travel 90 days, column masking | Production enterprise |
| **Business Critical** | HIPAA, PCI, Private Link, Tri-Secret Secure | Healthcare, finance |
| **Virtual Private** | Dedicated metadata, isolated VPC | Government, max security |

> **Exam Trap:** Time Travel max = **1 day (Standard)**, **90 days (Enterprise+)**. Fail-safe is always **7 days** (non-configurable, only Snowflake can recover from it).

---

## 1.4 Deployment Models

```text
Single Region Deployment (default):
  Customer Account → One Cloud Region → One Storage + Compute Region

Multi-Region / Business Continuity:
  Primary Account → Replication → Secondary Account (different region)
  Failover on DR events

Cross-Cloud:
  Data Sharing across AWS, Azure, GCP — no data movement
```

---

## 1.5 Snowflake Cortex AI Features

| Feature | What It Does |
|---------|-------------|
| `SNOWFLAKE.CORTEX.COMPLETE()` | LLM inference (Mistral, LLaMA, etc.) |
| `SNOWFLAKE.CORTEX.SENTIMENT()` | Sentiment scoring of text |
| `SNOWFLAKE.CORTEX.SUMMARIZE()` | Text summarization |
| `SNOWFLAKE.CORTEX.TRANSLATE()` | Language translation |
| `SNOWFLAKE.CORTEX.EMBED_TEXT_768()` | Text embeddings for vector search |
| Arctic | Snowflake's own open-source LLM |

```sql
-- AI in SQL — Cortex Example
SELECT 
  review_text,
  SNOWFLAKE.CORTEX.SENTIMENT(review_text) AS sentiment_score,
  SNOWFLAKE.CORTEX.SUMMARIZE(review_text) AS summary
FROM customer_reviews;
```

---

## 1.6 Snowpark

Snowpark allows you to write Python/Java/Scala code that **runs inside Snowflake compute** — not on your laptop.

```python
# Snowpark Python Example
from snowflake.snowpark import Session
from snowflake.snowpark.functions import col, avg

session = Session.builder.configs(connection_params).create()

df = session.table("SALES")
result = df.filter(col("REGION") == "WEST").groupBy("PRODUCT").agg(avg("REVENUE"))
result.show()
```

**Key Point:** Snowpark pushes the DataFrame logic to the warehouse — zero data leaves Snowflake.

---

## 1.7 Architecture Flow — Query Lifecycle

```text
User/Client submits SQL
        ↓
Cloud Services Layer receives query
        ↓
Authentication & Authorization check
        ↓
Query Parser → Abstract Syntax Tree
        ↓
Query Optimizer → Execution Plan
  (uses metadata: partition stats, column ranges, clustering keys)
        ↓
Check Result Cache (Cloud Services Layer)
  → HIT? Return instantly (0 credit cost)
  → MISS? Route to Virtual Warehouse
        ↓
Virtual Warehouse checks Local Disk Cache
  → HIT? Serve from SSD cache
  → MISS? Fetch micro-partitions from S3/blob
        ↓
MPP Execution across worker nodes
        ↓
Results aggregated → returned to client
        ↓
Results stored in Result Cache (24 hours)
```

---

## 1.8 Exam Quick-Fire: Architecture

| Question | Answer |
|----------|--------|
| Is storage shared across warehouses? | YES — all warehouses read from same S3/blob |
| Can two warehouses run same query simultaneously? | YES — no resource contention |
| What layer handles authentication? | Cloud Services Layer |
| What is the result cache TTL? | 24 hours (invalidated if data changes) |
| Does resizing a warehouse affect running queries? | NO — only new queries use new size |
| Which edition supports multi-cluster warehouses? | Enterprise and above |

---

# 2. TOOLS AND USER INTERFACE

## 2.1 Snowsight (Web UI)

Snowsight is the primary browser-based interface for Snowflake.

| Feature | Location in Snowsight |
|---------|----------------------|
| Run SQL queries | Worksheets |
| View query history | Activity → Query History |
| Monitor warehouse usage | Admin → Warehouses |
| Manage users/roles | Admin → Users & Roles |
| View databases/tables | Data → Databases |
| Marketplace access | Data Products → Marketplace |
| Cost monitoring | Admin → Cost Management |

### Worksheet Shortcuts
| Action | Shortcut |
|--------|----------|
| Run selected query | `Ctrl + Enter` |
| Run all | `Ctrl + Shift + Enter` |
| Format SQL | `Ctrl + Shift + F` |
| Comment line | `Ctrl + /` |

---

## 2.2 SnowSQL (CLI)

```bash
# Install and connect
snowsql -a <account_identifier> -u <username>

# Account identifier format
# <org_name>-<account_name>  (e.g., myorg-myaccount)
# OR legacy: <account>.<region>.<cloud>

# Run file
snowsql -a myorg-myaccount -u myuser -f my_script.sql

# Execute single query
snowsql -a myorg-myaccount -u myuser -q "SELECT CURRENT_VERSION()"

# Output to CSV
snowsql -a myorg-myaccount -u myuser -q "SELECT * FROM sales" -o output_format=csv > out.csv
```

### SnowSQL Config (~/.snowsql/config)
```ini
[connections.myconn]
accountname = myorg-myaccount
username = myuser
password = mypassword
dbname = PROD_DB
schemaname = PUBLIC
warehousename = COMPUTE_WH
```

---

## 2.3 Snowflake Connectors and Drivers

| Tool | Language | Use Case |
|------|----------|----------|
| Python Connector | Python | ETL, scripting |
| Snowpark Python | Python | In-Snowflake compute |
| JDBC Driver | Java | BI tools, JVM apps |
| ODBC Driver | Any | Excel, Tableau, Power BI |
| Node.js Driver | JavaScript | Web apps |
| Go Driver | Go | Microservices |
| .NET Connector | C# | Enterprise apps |
| Kafka Connector | Streaming | Real-time ingestion |
| Spark Connector | Scala/Python | Big data pipelines |

```python
# Python Connector Example
import snowflake.connector

conn = snowflake.connector.connect(
    user='myuser',
    password='mypassword',
    account='myorg-myaccount',
    warehouse='COMPUTE_WH',
    database='PROD_DB',
    schema='PUBLIC'
)

cur = conn.cursor()
cur.execute("SELECT COUNT(*) FROM orders")
result = cur.fetchall()
print(result)
cur.close()
conn.close()
```

---

## 2.4 Information Schema vs Account Usage

| Aspect | INFORMATION_SCHEMA | ACCOUNT_USAGE |
|--------|-------------------|---------------|
| Location | Each database | Shared `SNOWFLAKE` database |
| Latency | Real-time (no lag) | 45-minute lag |
| History retention | 7 days | 1 year |
| Dropped objects visible? | NO | YES |
| Good for | Current state queries | Auditing, cost analysis |

```sql
-- Information Schema — current state
SELECT * FROM MYDB.INFORMATION_SCHEMA.TABLES;

-- Account Usage — historical audit (1 year)
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE START_TIME >= DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY TOTAL_ELAPSED_TIME DESC
LIMIT 100;

-- Find most expensive queries
SELECT 
  QUERY_ID, QUERY_TEXT,
  TOTAL_ELAPSED_TIME/1000 AS elapsed_sec,
  CREDITS_USED_CLOUD_SERVICES,
  WAREHOUSE_NAME
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE TOTAL_ELAPSED_TIME > 60000  -- > 60 seconds
ORDER BY TOTAL_ELAPSED_TIME DESC;
```

---

## 2.5 Key System Functions Reference

```sql
-- Session context
SELECT CURRENT_USER();           -- logged-in user
SELECT CURRENT_ROLE();           -- active role
SELECT CURRENT_DATABASE();       -- active database
SELECT CURRENT_SCHEMA();         -- active schema
SELECT CURRENT_WAREHOUSE();      -- active warehouse
SELECT CURRENT_TIMESTAMP();      -- current time
SELECT CURRENT_VERSION();        -- Snowflake version
SELECT CURRENT_ACCOUNT();        -- account name
SELECT CURRENT_REGION();         -- cloud region

-- Set context
USE ROLE SYSADMIN;
USE WAREHOUSE COMPUTE_WH;
USE DATABASE PROD_DB;
USE SCHEMA SALES;
```

---

# 3. CATALOG AND OBJECTS

## 3.1 Object Hierarchy

```text
Organization (Org)
  └── Account (e.g., myorg-myaccount)
        └── Database
              └── Schema
                    ├── Table (permanent/transient/temporary/external)
                    ├── View (standard/materialized/secure)
                    ├── Stage (internal/external)
                    ├── File Format
                    ├── Sequence
                    ├── Procedure
                    ├── Function (UDF/UDTF)
                    ├── Task
                    ├── Stream
                    ├── Pipe
                    └── Alert
        └── Warehouse
        └── Role
        └── User
        └── Resource Monitor
        └── Integration (storage/notification/security/API)
        └── Network Policy
```

---

## 3.2 Database and Schema

```sql
-- Database
CREATE DATABASE PROD_DB
  DATA_RETENTION_TIME_IN_DAYS = 90   -- Enterprise+
  COMMENT = 'Production database';

CREATE DATABASE DEV_DB CLONE PROD_DB;  -- Zero-copy clone!

ALTER DATABASE PROD_DB RENAME TO PROD_DB_V2;
DROP DATABASE PROD_DB_V2;

-- Schema
CREATE SCHEMA PROD_DB.SALES
  DATA_RETENTION_TIME_IN_DAYS = 30;

CREATE TRANSIENT SCHEMA STAGING;  -- No Time Travel, no Fail-safe
```

---

## 3.3 Table Types — THE CRITICAL COMPARISON

| Property | Permanent | Transient | Temporary | External |
|----------|-----------|-----------|-----------|----------|
| Time Travel | ✅ 0-90 days | ✅ 0-1 day | ✅ 0-1 day (session only) | ❌ |
| Fail-safe | ✅ 7 days | ❌ | ❌ | ❌ |
| Storage cost | Higher | Lower | Lower (auto-drop) | Only metadata |
| Visible to others | ✅ | ✅ | ❌ (session-scoped) | ✅ |
| Use case | Production data | Staging/ETL | Intermediate results | External lake |
| Persists after session | ✅ | ✅ | ❌ | ✅ |

```sql
-- Permanent (default)
CREATE TABLE sales (
  sale_id     NUMBER       NOT NULL,
  sale_date   DATE,
  amount      DECIMAL(18,2),
  customer_id NUMBER,
  region      VARCHAR(50)
);

-- Transient — no Fail-safe, 1-day Time Travel max
CREATE TRANSIENT TABLE staging_load (
  raw_data VARIANT
);

-- Temporary — session-scoped, auto-drops
CREATE TEMPORARY TABLE session_calc AS
SELECT * FROM sales WHERE region = 'WEST';

-- External table — reads from external stage
CREATE EXTERNAL TABLE ext_sales (
  sale_id     NUMBER     AS (VALUE:sale_id::NUMBER),
  amount      FLOAT      AS (VALUE:amount::FLOAT)
)
LOCATION = @my_s3_stage/sales/
FILE_FORMAT = (TYPE = PARQUET)
AUTO_REFRESH = TRUE;

-- CTAS — Create Table As Select
CREATE TABLE sales_2024 AS
SELECT * FROM sales
WHERE YEAR(sale_date) = 2024;

-- Clone — zero copy!
CREATE TABLE sales_backup CLONE sales;
```

---

## 3.4 Column Data Types Reference

| Category | Types |
|----------|-------|
| Numeric | NUMBER, INT, BIGINT, FLOAT, DOUBLE, DECIMAL(p,s) |
| String | VARCHAR(n), CHAR(n), STRING, TEXT |
| Date/Time | DATE, TIME, TIMESTAMP_NTZ, TIMESTAMP_LTZ, TIMESTAMP_TZ |
| Boolean | BOOLEAN |
| Semi-structured | VARIANT, OBJECT, ARRAY |
| Geospatial | GEOGRAPHY, GEOMETRY |
| Binary | BINARY, VARBINARY |

> **Exam:** `TIMESTAMP_NTZ` = no timezone (stored as-is). `TIMESTAMP_LTZ` = local timezone (stored UTC, displayed in session timezone). `TIMESTAMP_TZ` = includes timezone offset.

```sql
-- VARIANT — stores ANY JSON/XML/Avro
CREATE TABLE events (
  event_id  NUMBER,
  payload   VARIANT,    -- can hold nested JSON
  ts        TIMESTAMP_LTZ
);

INSERT INTO events VALUES (
  1,
  PARSE_JSON('{"user":"alice","action":"login","meta":{"ip":"1.2.3.4"}}'),
  CURRENT_TIMESTAMP()
);
```

---

## 3.5 Views

```sql
-- Standard view — runs query every time
CREATE VIEW v_active_customers AS
SELECT customer_id, name, email
FROM customers
WHERE status = 'ACTIVE';

-- Secure view — hides definition from non-owners (for data sharing)
CREATE SECURE VIEW v_restricted_sales AS
SELECT region, SUM(amount) AS total
FROM sales
GROUP BY region;

-- Materialized view — pre-computed, auto-maintained
CREATE MATERIALIZED VIEW mv_daily_sales AS
SELECT DATE(sale_date) AS day, region, SUM(amount) AS revenue
FROM sales
GROUP BY 1, 2;
-- Snowflake auto-refreshes this when base table changes
-- Query mv_daily_sales = instant results (no full scan)
```

### View Comparison

| Property | Standard View | Secure View | Materialized View |
|----------|--------------|-------------|-------------------|
| Definition visible? | YES | NO (to non-owners) | YES |
| Pre-computed? | NO | NO | YES |
| Auto-refreshes? | N/A | N/A | YES (background) |
| Storage cost? | No | No | YES (stores result) |
| Use for sharing? | Risk | RECOMMENDED | Sometimes |
| Supports clustering? | NO | NO | YES |

---

## 3.6 Sequences

```sql
CREATE SEQUENCE seq_order_id
  START = 1000
  INCREMENT = 1
  COMMENT = 'Order ID sequence';

-- Use in INSERT
INSERT INTO orders (order_id, customer_id, total)
VALUES (seq_order_id.NEXTVAL, 101, 500.00);

-- View current value
SELECT seq_order_id.CURRVAL;
```

---

## 3.7 Streams and Tasks (Brief — detailed in Topic 17)

```sql
-- Stream — CDC (change data capture) on a table
CREATE STREAM orders_stream ON TABLE orders;

-- Task — scheduled SQL execution
CREATE TASK refresh_mv
  WAREHOUSE = COMPUTE_WH
  SCHEDULE = 'USING CRON 0 * * * * UTC'  -- every hour
AS
  INSERT INTO daily_agg SELECT ...;

ALTER TASK refresh_mv RESUME;  -- Tasks start SUSPENDED
```

---

## 3.8 Naming Conventions and Identifiers

| Rule | Detail |
|------|--------|
| Unquoted | Uppercase internally, case-insensitive |
| Quoted ("name") | Case-sensitive, preserved exactly |
| Max length | 255 characters |
| Valid chars (unquoted) | A-Z, 0-9, _, $ |
| Reserved words | Must quote if used as name |

```sql
-- Unquoted — stored as SALES_TABLE
CREATE TABLE sales_table (...);

-- Quoted — stored exactly as "Sales Table" (with space, case-sensitive)
CREATE TABLE "Sales Table" (...);

-- Must use quotes to reference it later
SELECT * FROM "Sales Table";
```

> **Exam Trap:** Snowflake stores unquoted identifiers as UPPERCASE. So `myTable` and `MYTABLE` and `mytable` are the same object. `"myTable"` is different.

---

# 4. STORAGE CONCEPTS

## 4.1 Micro-Partitions — The Core of Snowflake Storage

Micro-partitions are Snowflake's fundamental storage unit. Every table is automatically divided into micro-partitions.

```text
Table: SALES (billions of rows)
         ↓  Automatic partitioning (no user intervention)
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ Partition P1 │ │ Partition P2 │ │ Partition P3 │ │ Partition P4 │
│ Rows 1-15M  │ │ Rows 15M-30M │ │ Rows 30M-45M │ │ Rows 45M-60M │
│ Jan data     │ │ Feb data     │ │ Mar data     │ │ Apr data     │
│              │ │              │ │              │ │              │
│ Columnar     │ │ Columnar     │ │ Columnar     │ │ Columnar     │
│ Compressed   │ │ Compressed   │ │ Compressed   │ │ Compressed   │
│              │ │              │ │              │ │              │
│ Metadata:    │ │ Metadata:    │ │ Metadata:    │ │ Metadata:    │
│ MIN/MAX val  │ │ MIN/MAX val  │ │ MIN/MAX val  │ │ MIN/MAX val  │
│ Row count    │ │ Row count    │ │ Row count    │ │ Row count    │
│ NULL count   │ │ NULL count   │ │ NULL count   │ │ NULL count   │
└──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘
```

### Micro-Partition Properties

| Property | Value |
|----------|-------|
| Size (uncompressed) | 50–500 MB |
| Typical compressed size | ~16 MB |
| Storage format | Columnar (like Parquet) |
| Created by | Snowflake automatically (you can't control) |
| Compression | Automatic (Snappy, Zstd, etc.) |
| Metadata stored | MIN, MAX, COUNT, NULL_COUNT per column per partition |
| Immutable? | YES — DML creates new partitions |

---

## 4.2 Partition Pruning — How Queries Get Fast

```text
Query: SELECT * FROM sales WHERE sale_date = '2024-03-15'

Cloud Services Layer reads metadata:
  P1: sale_date MIN=2024-01-01 MAX=2024-01-31  → SKIP ✗
  P2: sale_date MIN=2024-02-01 MAX=2024-02-28  → SKIP ✗
  P3: sale_date MIN=2024-03-01 MAX=2024-03-31  → SCAN ✓
  P4: sale_date MIN=2024-04-01 MAX=2024-04-30  → SKIP ✗

Only P3 is read from S3 → 75% of I/O eliminated
```

> **Key Insight:** Partition pruning happens in the **Cloud Services Layer** before any data moves. This is why Snowflake is fast even without indexes.

---

## 4.3 Clustering Keys

Clustering keys tell Snowflake to co-locate related data in the same micro-partitions.

```sql
-- Add clustering key on a large table
ALTER TABLE sales CLUSTER BY (sale_date, region);

-- Clustering key on expression
ALTER TABLE sales CLUSTER BY (DATE_TRUNC('month', sale_date));

-- Check clustering information
SELECT SYSTEM$CLUSTERING_INFORMATION('sales', '(sale_date, region)');

-- Check clustering depth (lower = better clustered)
SELECT SYSTEM$CLUSTERING_DEPTH('sales');

-- Suspend/resume automatic clustering (costs credits)
ALTER TABLE sales SUSPEND RECLUSTER;
ALTER TABLE sales RESUME RECLUSTER;
```

### Clustering Depth Guide

| Depth | Meaning | Action |
|-------|---------|--------|
| 1 | Perfect — each partition has unique values | None needed |
| 1-3 | Good clustering | Monitor |
| 3-6 | Moderate — consider reclustering | Investigate |
| 6+ | Poor — high overlap | Add/change clustering key |

```sql
-- View clustering health from ACCOUNT_USAGE
SELECT 
  table_name,
  clustering_key,
  average_overlaps,
  average_depth
FROM SNOWFLAKE.ACCOUNT_USAGE.TABLE_STORAGE_METRICS
WHERE average_depth > 4;
```

---

## 4.4 Data Compression

Snowflake automatically compresses data. You cannot manually choose compression per column.

| Data Type | Typical Compression Ratio |
|-----------|--------------------------|
| Integer columns | 10:1 or better |
| String columns | 3:1 to 5:1 |
| High cardinality strings | 1.5:1 to 2:1 |
| Timestamps | 5:1 to 8:1 |
| VARIANT/JSON | 2:1 to 4:1 |

> **Exam:** Compression is **automatic** in Snowflake. Users cannot specify compression algorithms.

---

## 4.5 Storage Billing Model

```text
Storage charged = AVERAGE daily storage used × $/TB/month

Includes:
  - Active table data
  - Time Travel data (historical versions)
  - Fail-safe data (7 days, Snowflake-managed)
  - Clone metadata (only diverged data)
  - Stage files (internal stages)

Does NOT include:
  - External stage files (you pay cloud provider)
  - Query result cache
```

```sql
-- Check storage usage
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.STORAGE_USAGE
ORDER BY USAGE_DATE DESC LIMIT 30;

-- Table-level storage
SELECT 
  TABLE_SCHEMA,
  TABLE_NAME,
  ACTIVE_BYTES / 1e9 AS active_gb,
  TIME_TRAVEL_BYTES / 1e9 AS time_travel_gb,
  FAILSAFE_BYTES / 1e9 AS failsafe_gb
FROM SNOWFLAKE.ACCOUNT_USAGE.TABLE_STORAGE_METRICS
ORDER BY ACTIVE_BYTES DESC;
```

---

## 4.6 Stages — Temporary Data Landing Zones

### Stage Types

```text
Stages
├── Internal Stages (managed by Snowflake)
│   ├── User Stage    — @~          (per user, auto-created)
│   ├── Table Stage   — @%tablename (per table, auto-created)
│   └── Named Stage   — @my_stage   (you create and manage)
│
└── External Stages (point to cloud storage)
    ├── S3 Stage      (AWS)
    ├── Azure Stage   (Azure Blob Storage)
    └── GCS Stage     (Google Cloud Storage)
```

```sql
-- Named internal stage
CREATE STAGE my_internal_stage
  FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER = ',' SKIP_HEADER = 1);

-- External S3 stage (with storage integration — recommended)
CREATE STAGE my_s3_stage
  URL = 's3://mybucket/data/'
  STORAGE_INTEGRATION = s3_int
  FILE_FORMAT = (TYPE = PARQUET);

-- External stage with credentials (not recommended — use integration)
CREATE STAGE my_s3_stage_creds
  URL = 's3://mybucket/data/'
  CREDENTIALS = (AWS_KEY_ID='xxx' AWS_SECRET_KEY='yyy');

-- List stage contents
LIST @my_internal_stage;
LIST @my_s3_stage;

-- Remove file from internal stage
REMOVE @my_internal_stage/myfile.csv;

-- Upload file (SnowSQL only)
PUT file:///local/path/data.csv @my_internal_stage;
```

---

## 4.7 File Formats

```sql
-- CSV file format
CREATE FILE FORMAT my_csv_format
  TYPE = CSV
  FIELD_DELIMITER = ','
  RECORD_DELIMITER = '\n'
  SKIP_HEADER = 1
  NULL_IF = ('NULL', 'null', '')
  EMPTY_FIELD_AS_NULL = TRUE
  FIELD_OPTIONALLY_ENCLOSED_BY = '"'
  DATE_FORMAT = 'YYYY-MM-DD'
  TIMESTAMP_FORMAT = 'YYYY-MM-DD HH24:MI:SS';

-- JSON file format
CREATE FILE FORMAT my_json_format
  TYPE = JSON
  STRIP_OUTER_ARRAY = TRUE     -- removes [ ] wrapper
  STRIP_NULL_VALUES = FALSE;

-- Parquet file format
CREATE FILE FORMAT my_parquet_format
  TYPE = PARQUET
  SNAPPY_COMPRESSION = TRUE;

-- ORC file format
CREATE FILE FORMAT my_orc_format TYPE = ORC;

-- Avro
CREATE FILE FORMAT my_avro_format TYPE = AVRO;
```

### File Format Support

| Format | Load | Unload | Semi-structured? |
|--------|------|--------|-----------------|
| CSV | ✅ | ✅ | ❌ |
| JSON | ✅ | ✅ | ✅ |
| Parquet | ✅ | ✅ | ✅ |
| Avro | ✅ | ❌ | ✅ |
| ORC | ✅ | ❌ | ✅ |
| XML | ✅ | ❌ | ✅ |

---

# 5. AUTHENTICATION FRAMEWORK

## 5.1 Authentication Methods Overview

```text
User tries to connect to Snowflake
              ↓
  ┌───────────────────────────────┐
  │   Authentication Options      │
  │                               │
  │  1. Username + Password       │
  │  2. MFA (TOTP via app)        │
  │  3. Key-pair (RSA)            │
  │  4. SSO (OAuth/SAML)          │
  │  5. External OAuth            │
  │  6. Snowflake OAuth           │
  │  7. SCIM provisioning         │
  └───────────────────────────────┘
              ↓
  Cloud Services Layer validates
              ↓
  Session established with Role + WH
```

---

## 5.2 Username + Password

```sql
-- Create user with password
CREATE USER alice
  PASSWORD = 'SecurePass123!'
  DEFAULT_ROLE = ANALYST
  DEFAULT_WAREHOUSE = COMPUTE_WH
  DEFAULT_NAMESPACE = PROD_DB.PUBLIC
  EMAIL = 'alice@company.com'
  MUST_CHANGE_PASSWORD = TRUE;

-- Alter password
ALTER USER alice SET PASSWORD = 'NewPass456!';

-- Lock user
ALTER USER alice SET DISABLED = TRUE;

-- View user properties
DESC USER alice;
SHOW USERS LIKE 'alice';
```

---

## 5.3 Multi-Factor Authentication (MFA)

```sql
-- Enforce MFA for a user (they must enroll via Duo app)
ALTER USER alice SET MINS_TO_BYPASS_MFA = 0;

-- Allow temporary bypass (e.g., during testing)
ALTER USER alice SET MINS_TO_BYPASS_MFA = 60;

-- Network policy to enforce MFA
CREATE NETWORK POLICY require_mfa_policy
  ALLOWED_IP_LIST = ('192.168.0.0/16')
  BLOCKED_IP_LIST = ('0.0.0.0/0');
```

---

## 5.4 Key-Pair Authentication (Recommended for Service Accounts)

```bash
# 1. Generate RSA key pair (terminal)
openssl genrsa 2048 | openssl pkcs8 -topk8 -inform PEM -out rsa_key.p8 -nocrypt
openssl rsa -in rsa_key.p8 -pubout -out rsa_key.pub

# 2. Get public key value (strip header/footer)
cat rsa_key.pub
```

```sql
-- 3. Assign public key to user in Snowflake
ALTER USER alice SET RSA_PUBLIC_KEY='MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ...';

-- Assign secondary key (for rotation without downtime)
ALTER USER alice SET RSA_PUBLIC_KEY_2='MIIBIjANBgkq...';
```

```python
# 4. Connect with private key
from snowflake.connector import connect
from cryptography.hazmat.primitives import serialization

with open("rsa_key.p8", "rb") as key_file:
    private_key = serialization.load_pem_private_key(key_file.read(), password=None)

conn = connect(
    user='alice',
    account='myorg-myaccount',
    private_key=private_key,
    warehouse='COMPUTE_WH',
    database='PROD_DB'
)
```

---

## 5.5 OAuth Authentication

### Snowflake OAuth (Internal)
```sql
-- Create OAuth integration for a client app
CREATE SECURITY INTEGRATION my_oauth_integration
  TYPE = OAUTH
  OAUTH_CLIENT = CUSTOM
  OAUTH_CLIENT_TYPE = CONFIDENTIAL
  OAUTH_REDIRECT_URI = 'https://myapp.com/oauth/callback'
  OAUTH_ISSUE_REFRESH_TOKENS = TRUE
  OAUTH_REFRESH_TOKEN_VALIDITY = 7776000;  -- 90 days

-- View integration
DESC INTEGRATION my_oauth_integration;
```

### External OAuth (Okta, Azure AD, etc.)
```sql
CREATE SECURITY INTEGRATION okta_oauth
  TYPE = EXTERNAL_OAUTH
  EXTERNAL_OAUTH_TYPE = OKTA
  EXTERNAL_OAUTH_ISSUER = 'https://mycompany.okta.com/oauth2/default'
  EXTERNAL_OAUTH_JWS_KEYS_URL = 'https://mycompany.okta.com/oauth2/default/v1/keys'
  EXTERNAL_OAUTH_AUDIENCE_LIST = ('https://myorg-myaccount.snowflakecomputing.com')
  EXTERNAL_OAUTH_TOKEN_USER_MAPPING_CLAIM = 'sub'
  EXTERNAL_OAUTH_SNOWFLAKE_USER_MAPPING_ATTRIBUTE = 'email_address'
  ENABLED = TRUE;
```

---

## 5.6 SAML / SSO

```sql
-- SAML2 Security Integration (for IdP like Okta, ADFS, Azure AD)
CREATE SECURITY INTEGRATION my_saml_integration
  TYPE = SAML2
  ENABLED = TRUE
  SAML2_ISSUER = 'https://mycompany.okta.com'
  SAML2_SSO_URL = 'https://mycompany.okta.com/app/snowflake/sso/saml'
  SAML2_PROVIDER = 'OKTA'
  SAML2_X509_CERT = 'MIIDpDCCAoygAwIBAgIGAX...'
  SAML2_SP_INITIATED_LOGIN_PAGE_LABEL = 'Okta SSO'
  SAML2_ENABLE_SP_INITIATED = TRUE;

SHOW INTEGRATIONS;
```

---

## 5.7 Network Policies

Network policies restrict which IP addresses can connect to Snowflake.

```sql
-- Create network policy
CREATE NETWORK POLICY office_only
  ALLOWED_IP_LIST = ('203.0.113.0/24', '198.51.100.5')
  BLOCKED_IP_LIST = ()
  COMMENT = 'Allow only office IPs';

-- Apply to entire account
ALTER ACCOUNT SET NETWORK_POLICY = office_only;

-- Apply to specific user only
ALTER USER alice SET NETWORK_POLICY = office_only;

-- Remove policy from user
ALTER USER alice UNSET NETWORK_POLICY;

-- View network policies
SHOW NETWORK POLICIES;
```

> **Exam:** User-level network policy **overrides** account-level policy. User policy takes precedence.

---

## 5.8 SCIM (System for Cross-domain Identity Management)

SCIM allows automatic provisioning/deprovisioning of Snowflake users from an IdP (Okta, Azure AD).

```sql
-- Create SCIM security integration
CREATE SECURITY INTEGRATION my_scim_integration
  TYPE = SCIM
  SCIM_CLIENT = 'OKTA'
  RUN_AS_ROLE = 'OKTA_PROVISIONER';

-- Generate SCIM access token
SELECT SYSTEM$GENERATE_SCIM_ACCESS_TOKEN('my_scim_integration');
```

**SCIM Flow:**
```text
Okta/Azure AD (IdP)
  → User provisioned/deprovisioned in IdP
  → SCIM API call to Snowflake
  → User created/disabled in Snowflake automatically
  → Role assignments synced
```

---

## 5.9 Authentication Exam Summary Table

| Method | Best For | Supports MFA? | Service Accounts? |
|--------|----------|---------------|-------------------|
| Username + Password | Humans (dev/test) | Optional (Duo) | Not recommended |
| Key-pair (RSA) | Service accounts, CI/CD | No | ✅ BEST |
| MFA | High-security human users | ✅ YES | No |
| OAuth (Snowflake) | BI tools, web apps | No | Sometimes |
| External OAuth | Enterprise SSO | Via IdP | Sometimes |
| SAML/SSO | Enterprise single sign-on | Via IdP | No |
| SCIM | Auto user provisioning | N/A | N/A |

---

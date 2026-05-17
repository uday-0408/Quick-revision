# ❄️ Snowflake Compute Pool — Complete Cheat Sheet

> **Compute Pools** are sets of virtual machine (VM) nodes used exclusively by **Snowpark Container Services (SPCS)** to run containerized workloads. They are separate from Virtual Warehouses (which handle SQL/query workloads).

---

## 📋 Table of Contents

1. [What Is a Compute Pool?](#what-is-a-compute-pool)
2. [Compute Pool vs Virtual Warehouse](#compute-pool-vs-virtual-warehouse)
3. [Instance Families & Node Types](#instance-families--node-types)
4. [Lifecycle & States](#lifecycle--states)
5. [Creating a Compute Pool](#creating-a-compute-pool)
6. [Altering a Compute Pool](#altering-a-compute-pool)
7. [Dropping a Compute Pool](#dropping-a-compute-pool)
8. [Suspending & Resuming](#suspending--resuming)
9. [Auto-Suspend & Auto-Resume](#auto-suspend--auto-resume)
10. [Viewing Compute Pools](#viewing-compute-pools)
11. [Privileges & Access Control](#privileges--access-control)
12. [Associating with Services & Jobs](#associating-with-services--jobs)
13. [Resource Monitoring & Cost Control](#resource-monitoring--cost-control)
14. [Networking Considerations](#networking-considerations)
15. [Limits & Quotas](#limits--quotas)
16. [Best Practices](#best-practices)
17. [Troubleshooting](#troubleshooting)
18. [Quick-Reference Command Table](#quick-reference-command-table)

---

## 1. What Is a Compute Pool?

A **Compute Pool** is a named cluster of one or more virtual machine nodes provisioned inside your Snowflake account to execute **Snowpark Container Services** workloads:

- Runs **long-running services** (web apps, APIs, model-serving endpoints).
- Runs **batch jobs** (one-shot containerized tasks).
- Each node is a full VM with CPU/memory/GPU (depending on instance family).
- Billed per **node-hour** (nodes are the unit, not credits like warehouses).
- Managed fully by Snowflake — you don't interact with the underlying infrastructure.

```
┌─────────────────────────────────────────┐
│            SNOWFLAKE ACCOUNT            │
│                                         │
│  ┌───────────────┐  ┌────────────────┐  │
│  │  Compute Pool │  │    SPCS Image  │  │
│  │  (VM Nodes)   │◄─┤    Registry    │  │
│  │               │  └────────────────┘  │
│  │  ┌─────────┐  │                      │
│  │  │  Node 1 │  │  ┌────────────────┐  │
│  │  │  Node 2 │  │◄─┤ Service / Job  │  │
│  │  │  Node N │  │  └────────────────┘  │
│  └───────────────┘                      │
└─────────────────────────────────────────┘
```

---

## 2. Compute Pool vs Virtual Warehouse

| Feature | Compute Pool | Virtual Warehouse |
|---|---|---|
| **Purpose** | Container workloads (SPCS) | SQL queries, DML, data loading |
| **Unit** | VM nodes | T-shirt sizes (XS–6XL) |
| **Billing** | Node-hours | Snowflake credits |
| **Scaling** | Min/Max nodes (manual or auto) | Scale-out clusters |
| **GPU Support** | ✅ Yes (GPU instance families) | ❌ No |
| **Persistent** | Can run 24/7 | Auto-suspend idle |
| **Use Case** | ML inference, APIs, batch jobs | Analytics, ETL |

---

## 3. Instance Families & Node Types

Instance families control the CPU, memory, and GPU available per node.

### CPU Instance Families

| Instance Family | vCPU | Memory | Use Case |
|---|---|---|---|
| `CPU_X64_XS` | 2 vCPU | 8 GB | Dev/test, lightweight workloads |
| `CPU_X64_S` | 4 vCPU | 16 GB | Small services, APIs |
| `CPU_X64_M` | 8 vCPU | 32 GB | Medium workloads, inference |
| `CPU_X64_L` | 16 vCPU | 64 GB | Large compute jobs |
| `CPU_X64_XL` | 32 vCPU | 128 GB | Heavy batch processing |
| `CPU_X64_2XL` | 64 vCPU | 256 GB | Very large workloads |
| `CPU_X64_4XL` | 96 vCPU | 384 GB | Extreme CPU workloads |

### GPU Instance Families

| Instance Family | vCPU | Memory | GPU | GPU Memory | Use Case |
|---|---|---|---|---|---|
| `GPU_NV_S` | 6 vCPU | 27 GB | 1× NVIDIA A10G | 24 GB | Small GPU inference |
| `GPU_NV_M` | 44 vCPU | 178 GB | 4× NVIDIA A10G | 96 GB | Multi-GPU training/inference |
| `GPU_NV_L` | 92 vCPU | 388 GB | 8× NVIDIA A10G | 192 GB | Large-scale GPU workloads |

> **Note:** GPU families are available in supported regions only. Check availability before designing architecture.

---

## 4. Lifecycle & States

```
                  ┌──────────┐
                  │  CREATED │  (not yet started)
                  └────┬─────┘
                       │ START / AUTO-RESUME
                  ┌────▼─────┐
         ┌────────┤  STARTING │  (provisioning nodes)
         │        └────┬─────┘
         │             │ nodes ready
         │        ┌────▼─────┐
    ERROR│        │  ACTIVE   │◄──────────────────┐
         │        └────┬─────┘                   │
         │             │ idle > AUTO_SUSPEND secs │RESUME
         │        ┌────▼──────┐                  │
         │        │ SUSPENDING│                  │
         │        └────┬──────┘                  │
         │             │                         │
         │        ┌────▼──────┐──────────────────┘
         └────────► SUSPENDED │
                  └────┬──────┘
                       │ DROP
                  ┌────▼──────┐
                  │  DROPPED  │
                  └───────────┘
```

| State | Description |
|---|---|
| `STARTING` | Nodes are being provisioned. Services/jobs queued. |
| `ACTIVE` | Nodes running and available. Services can execute. |
| `SUSPENDING` | Draining connections, shutting down nodes. |
| `SUSPENDED` | No nodes running. No compute charges. |
| `STOPPING` | Transitional stop state. |
| `IDLE` | Pool active but no workloads running (charges still apply). |

---

## 5. Creating a Compute Pool

### Basic Syntax

```sql
CREATE COMPUTE POOL <name>
  MIN_NODES = <integer>
  MAX_NODES = <integer>
  INSTANCE_FAMILY = <instance_family>
  [ AUTO_RESUME = { TRUE | FALSE } ]
  [ AUTO_SUSPEND_SECS = <integer> ]
  [ COMMENT = '<string>' ]
  [ INITIALLY_SUSPENDED = { TRUE | FALSE } ];
```

### Parameters

| Parameter | Required | Default | Description |
|---|---|---|---|
| `MIN_NODES` | ✅ | — | Minimum number of VM nodes (≥ 1) |
| `MAX_NODES` | ✅ | — | Maximum number of VM nodes (≥ MIN_NODES) |
| `INSTANCE_FAMILY` | ✅ | — | VM type (e.g., `CPU_X64_S`, `GPU_NV_S`) |
| `AUTO_RESUME` | ❌ | `TRUE` | Auto-start pool when a service/job is submitted |
| `AUTO_SUSPEND_SECS` | ❌ | `3600` | Seconds of inactivity before auto-suspend (0 = never) |
| `COMMENT` | ❌ | `''` | Human-readable description |
| `INITIALLY_SUSPENDED` | ❌ | `FALSE` | Create pool in suspended state |

### Examples

```sql
-- Minimal CPU pool for development
CREATE COMPUTE POOL dev_pool
  MIN_NODES = 1
  MAX_NODES = 2
  INSTANCE_FAMILY = CPU_X64_XS;

-- Production GPU pool for ML inference
CREATE COMPUTE POOL ml_inference_pool
  MIN_NODES = 2
  MAX_NODES = 10
  INSTANCE_FAMILY = GPU_NV_S
  AUTO_RESUME = TRUE
  AUTO_SUSPEND_SECS = 1800
  COMMENT = 'Production pool for model serving'
  INITIALLY_SUSPENDED = FALSE;

-- Cost-efficient pool that starts suspended
CREATE COMPUTE POOL batch_pool
  MIN_NODES = 1
  MAX_NODES = 5
  INSTANCE_FAMILY = CPU_X64_M
  AUTO_RESUME = TRUE
  AUTO_SUSPEND_SECS = 600
  INITIALLY_SUSPENDED = TRUE;
```

---

## 6. Altering a Compute Pool

```sql
ALTER COMPUTE POOL <name> SET
  [ MIN_NODES = <integer> ]
  [ MAX_NODES = <integer> ]
  [ AUTO_RESUME = { TRUE | FALSE } ]
  [ AUTO_SUSPEND_SECS = <integer> ]
  [ COMMENT = '<string>' ];
```

> ⚠️ **You CANNOT change `INSTANCE_FAMILY` after creation.** Drop and recreate if you need a different instance type.

### Examples

```sql
-- Scale up max nodes
ALTER COMPUTE POOL ml_inference_pool SET MAX_NODES = 20;

-- Disable auto-suspend (pool runs indefinitely)
ALTER COMPUTE POOL dev_pool SET AUTO_SUSPEND_SECS = 0;

-- Enable auto-resume and set 10 min suspend
ALTER COMPUTE POOL batch_pool SET
  AUTO_RESUME = TRUE
  AUTO_SUSPEND_SECS = 600;

-- Update comment
ALTER COMPUTE POOL dev_pool SET COMMENT = 'Dev environment - do not use in prod';
```

---

## 7. Dropping a Compute Pool

```sql
DROP COMPUTE POOL <name>;
```

> ⚠️ **Prerequisites before dropping:**
> - All **services** using the pool must be **stopped/dropped**.
> - All **jobs** must be **completed or stopped**.
> - The pool must be in `SUSPENDED` or `IDLE` state (Snowflake will auto-suspend before dropping).

```sql
-- Safe drop pattern
ALTER COMPUTE POOL my_pool SUSPEND;
-- wait for state = SUSPENDED, then:
DROP COMPUTE POOL my_pool;
```

---

## 8. Suspending & Resuming

### Suspend

```sql
ALTER COMPUTE POOL <name> SUSPEND;
```

- Gracefully stops all running containers.
- Terminates VM nodes (billing stops).
- Services transition to a waiting state.
- Can take several minutes depending on running workloads.

### Resume

```sql
ALTER COMPUTE POOL <name> RESUME;
```

- Provisions new VM nodes.
- Services automatically restart on the pool.
- Can take 2–10 minutes for nodes to become ready.

### Stop All Services First (recommended)

```sql
-- Stop a service before suspending pool
ALTER SERVICE my_service SUSPEND;
ALTER COMPUTE POOL my_pool SUSPEND;
```

---

## 9. Auto-Suspend & Auto-Resume

### AUTO_SUSPEND_SECS

Controls how many seconds of **inactivity** (no services or jobs running) before the pool automatically suspends.

```sql
-- Suspend after 5 minutes of inactivity
ALTER COMPUTE POOL my_pool SET AUTO_SUSPEND_SECS = 300;

-- Never auto-suspend
ALTER COMPUTE POOL my_pool SET AUTO_SUSPEND_SECS = 0;

-- Default: suspend after 1 hour
ALTER COMPUTE POOL my_pool SET AUTO_SUSPEND_SECS = 3600;
```

| Value | Behavior |
|---|---|
| `0` | Never auto-suspend (runs 24/7 — watch your costs!) |
| `60`–`3600` | Common range; 300–600 good for dev pools |
| `> 3600` | Long-lived pools; suitable for 24/7 services |

### AUTO_RESUME

When `TRUE`, submitting a service or job to a **suspended** pool automatically triggers a resume.

```sql
ALTER COMPUTE POOL my_pool SET AUTO_RESUME = TRUE;
```

| AUTO_RESUME | Behavior when pool is suspended |
|---|---|
| `TRUE` | Pool auto-starts when a service/job is submitted |
| `FALSE` | You must manually `RESUME` the pool; submissions queue or fail |

---

## 10. Viewing Compute Pools

### List All Compute Pools

```sql
SHOW COMPUTE POOLS;
```

Returns: name, state, min/max nodes, current nodes, instance family, auto_resume, auto_suspend_secs, owner, comment, created_on.

### Describe a Specific Pool

```sql
DESCRIBE COMPUTE POOL <name>;
-- or
DESC COMPUTE POOL <name>;
```

### Query via Information Schema / Account Usage

```sql
-- Current pool status
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.COMPUTE_POOLS
WHERE NAME = 'MY_POOL';

-- All pools with their states
SELECT
  NAME,
  STATE,
  MIN_NODES,
  MAX_NODES,
  INSTANCE_FAMILY,
  AUTO_RESUME,
  AUTO_SUSPEND_SECS,
  COMMENT,
  CREATED
FROM SNOWFLAKE.ACCOUNT_USAGE.COMPUTE_POOLS
ORDER BY CREATED DESC;
```

### Useful Filtering Examples

```sql
-- Find all active GPU pools
SHOW COMPUTE POOLS LIKE '%gpu%';

-- Find suspended pools (potential cost optimization)
SELECT NAME, STATE, MIN_NODES, MAX_NODES
FROM SNOWFLAKE.ACCOUNT_USAGE.COMPUTE_POOLS
WHERE STATE = 'SUSPENDED';
```

---

## 11. Privileges & Access Control

### Required Privileges

| Action | Privilege | Object |
|---|---|---|
| Create a compute pool | `CREATE COMPUTE POOL` | Account |
| Use a compute pool | `USAGE` | Compute Pool |
| Monitor a compute pool | `MONITOR` | Compute Pool |
| Modify a compute pool | `OPERATE` | Compute Pool |
| Drop a compute pool | Ownership | Compute Pool |

### Granting Privileges

```sql
-- Allow a role to use a pool
GRANT USAGE ON COMPUTE POOL my_pool TO ROLE data_scientist_role;

-- Allow monitoring (view stats, describe)
GRANT MONITOR ON COMPUTE POOL my_pool TO ROLE monitoring_role;

-- Allow operating (suspend/resume)
GRANT OPERATE ON COMPUTE POOL my_pool TO ROLE ops_role;

-- Grant all privileges
GRANT ALL ON COMPUTE POOL my_pool TO ROLE admin_role;
```

### Revoking Privileges

```sql
REVOKE USAGE ON COMPUTE POOL my_pool FROM ROLE data_scientist_role;
```

### Ownership Transfer

```sql
GRANT OWNERSHIP ON COMPUTE POOL my_pool TO ROLE new_owner_role
  REVOKE CURRENT GRANTS;
```

---

## 12. Associating with Services & Jobs

### Creating a Service on a Compute Pool

```sql
CREATE SERVICE my_service
  IN COMPUTE POOL my_pool
  FROM SPECIFICATION $$
    spec:
      containers:
        - name: app
          image: /my_db/my_schema/my_repo/my_image:latest
          resources:
            requests:
              memory: "2Gi"
              cpu: "1"
            limits:
              memory: "4Gi"
              cpu: "2"
      endpoints:
        - name: http
          port: 8080
          public: true
  $$
  MIN_INSTANCES = 1
  MAX_INSTANCES = 3;
```

### Creating a Job (one-shot container) on a Compute Pool

```sql
EXECUTE JOB SERVICE
  IN COMPUTE POOL my_pool
  NAME = my_batch_job
  FROM SPECIFICATION $$
    spec:
      containers:
        - name: worker
          image: /my_db/my_schema/my_repo/batch_image:latest
          env:
            INPUT_PATH: "@my_stage/input/"
            OUTPUT_PATH: "@my_stage/output/"
  $$;
```

### Service-to-Pool Relationship

- A **service** is bound to exactly **one compute pool**.
- A compute pool can host **multiple services**.
- Services compete for resources within a pool; Kubernetes-style scheduling applies.
- You can move a service to a different pool by **dropping and recreating** it.

---

## 13. Resource Monitoring & Cost Control

### Viewing Node Usage

```sql
-- Current node counts per pool
SELECT
  NAME,
  STATE,
  MIN_NODES,
  MAX_NODES,
  -- ACTIVE_NODES not directly in schema; use SHOW:
  INSTANCE_FAMILY
FROM SNOWFLAKE.ACCOUNT_USAGE.COMPUTE_POOLS;

-- Show detailed pool info including active node count
SHOW COMPUTE POOLS;
```

### Cost Estimation

Compute pools are billed based on:
- **Instance family** (larger = more expensive)
- **Number of active nodes**
- **Hours nodes are running** (partial hours billed as full hours in some regions)

```
Monthly Cost ≈ (Active Nodes) × (Node Price/hr) × (Hours Running)
```

> Check the [Snowflake Pricing Page](https://www.snowflake.com/pricing/) for current rates per instance family and region.

### Cost Control Strategies

```sql
-- 1. Use INITIALLY_SUSPENDED for dev pools
CREATE COMPUTE POOL dev_pool
  MIN_NODES = 1
  MAX_NODES = 1
  INSTANCE_FAMILY = CPU_X64_XS
  INITIALLY_SUSPENDED = TRUE
  AUTO_SUSPEND_SECS = 300;  -- suspend after 5 min idle

-- 2. Set aggressive auto-suspend for non-critical pools
ALTER COMPUTE POOL etl_pool SET AUTO_SUSPEND_SECS = 300;

-- 3. Use Resource Monitors at account level to cap spend
CREATE RESOURCE MONITOR spcs_monitor
  WITH CREDIT_QUOTA = 500
  TRIGGERS
    ON 80 PERCENT DO NOTIFY
    ON 100 PERCENT DO SUSPEND;
```

### Monitoring with Account Usage

```sql
-- Approximate node uptime per pool (last 30 days)
SELECT
  POOL_NAME,
  SUM(CREDITS_USED) AS total_credits,
  COUNT(DISTINCT DATE_TRUNC('day', START_TIME)) AS active_days
FROM SNOWFLAKE.ACCOUNT_USAGE.COMPUTE_POOL_EVENTS  -- check view name in your version
GROUP BY POOL_NAME
ORDER BY total_credits DESC;
```

---

## 14. Networking Considerations

### Egress & Ingress

- Compute pool nodes can make **outbound network calls** (to external APIs, registries) if the network policy allows it.
- Services can expose **public endpoints** (HTTPS) or remain internal.
- Use **Snowflake Private Connectivity** to keep traffic inside your VPC/VNet.

### Image Registry

Images for SPCS must be stored in the **Snowflake Image Registry** (not Docker Hub directly):

```sql
-- Create image repository
CREATE IMAGE REPOSITORY my_repo;

-- Get repository URL
SHOW IMAGE REPOSITORIES;

-- Tag and push from local
docker tag my_image:latest <account>.registry.snowflakecomputing.com/my_db/my_schema/my_repo/my_image:latest
docker push <account>.registry.snowflakecomputing.com/my_db/my_schema/my_repo/my_image:latest
```

### DNS & Service Endpoints

```sql
-- Get the public endpoint of a service
SHOW ENDPOINTS IN SERVICE my_service;

-- Result: ingress_url like https://abc123.snowflakecomputing.app
```

---

## 15. Limits & Quotas

| Limit | Default | Notes |
|---|---|---|
| Max compute pools per account | 10 | Can request increase via Support |
| Max nodes per compute pool | 100 | Soft limit; contact Snowflake to raise |
| Min nodes | 1 | — |
| Max services per compute pool | No hard limit | Resource-bound |
| Container image size | No hard limit | Larger images = slower cold starts |
| Auto-suspend minimum | 60 seconds | Cannot set below 60 (except 0 = disabled) |
| GPU pools per account | Varies by region | Subject to GPU availability |

> 📌 Limits vary by **Snowflake edition** (Standard, Enterprise, Business Critical) and **region**. Always verify via `SHOW PARAMETERS` or Snowflake documentation.

---

## 16. Best Practices

### Pool Design

```
✅ DO:
  - Separate pools by workload type (dev vs prod, CPU vs GPU)
  - Use MIN_NODES = 1 for most pools (scale from there)
  - Set AUTO_SUSPEND_SECS = 300–600 for dev/staging pools
  - Tag pools with COMMENT for documentation
  - Use INITIALLY_SUSPENDED = TRUE for pools not needed immediately

❌ AVOID:
  - Setting AUTO_SUSPEND_SECS = 0 unless you need 24/7 availability
  - Putting all services on a single large pool (reduces isolation)
  - Over-provisioning MAX_NODES — start small and scale up
  - Sharing GPU pools between unrelated teams (resource contention)
```

### Naming Convention

```sql
-- Pattern: <env>_<team>_<workload>_pool
CREATE COMPUTE POOL prod_mlteam_inference_pool ...;
CREATE COMPUTE POOL dev_dataeng_etl_pool ...;
CREATE COMPUTE POOL staging_api_serving_pool ...;
```

### Right-sizing

| Workload | Recommended Family | MIN | MAX |
|---|---|---|---|
| Dev/test | `CPU_X64_XS` | 1 | 2 |
| REST API (light) | `CPU_X64_S` | 1 | 5 |
| Data processing | `CPU_X64_M` | 2 | 10 |
| ML inference (CPU) | `CPU_X64_L` | 1 | 8 |
| ML inference (GPU) | `GPU_NV_S` | 1 | 4 |
| LLM serving | `GPU_NV_M` or `GPU_NV_L` | 1 | 3 |

---

## 17. Troubleshooting

### Pool Stuck in STARTING

```sql
-- Check pool state
SHOW COMPUTE POOLS LIKE 'my_pool';

-- Check for quota limits
SELECT SYSTEM$GET_COMPUTE_POOL_STATUS('my_pool');

-- Try suspending and resuming
ALTER COMPUTE POOL my_pool SUSPEND;
ALTER COMPUTE POOL my_pool RESUME;
```

**Common causes:**
- Regional GPU capacity unavailable → switch to a different region or family.
- Account-level node quota exceeded → request a quota increase.
- Network policy blocking node provisioning → check with account admin.

### Service Won't Start

```sql
-- Get service status and events
SHOW SERVICES LIKE 'my_service';
SELECT SYSTEM$GET_SERVICE_STATUS('my_service');
SELECT SYSTEM$GET_SERVICE_LOGS('my_service', '0', 'app', 100);
```

**Common causes:**
- Pool is `SUSPENDED` and `AUTO_RESUME = FALSE`.
- Container image pull fails (wrong path, auth issue).
- Insufficient resources on pool nodes (OOM, CPU).
- Port conflicts in service spec.

### High Costs / Pool Not Suspending

```sql
-- Check if services are still running (preventing suspend)
SHOW SERVICES IN COMPUTE POOL my_pool;

-- Suspend all services first, then pool
ALTER SERVICE service_1 SUSPEND;
ALTER SERVICE service_2 SUSPEND;
ALTER COMPUTE POOL my_pool SUSPEND;
```

### Checking Pool Events & Logs

```sql
-- System function for pool status details
SELECT SYSTEM$GET_COMPUTE_POOL_STATUS('my_pool');

-- Service-level logs for debugging container issues
CALL SYSTEM$GET_SERVICE_LOGS('my_service', '0', 'container_name', 200);
```

---

## 18. Quick-Reference Command Table

| Action | Command |
|---|---|
| **Create** pool | `CREATE COMPUTE POOL <n> MIN_NODES=1 MAX_NODES=3 INSTANCE_FAMILY=CPU_X64_S;` |
| **List** all pools | `SHOW COMPUTE POOLS;` |
| **Describe** pool | `DESC COMPUTE POOL <n>;` |
| **Suspend** pool | `ALTER COMPUTE POOL <n> SUSPEND;` |
| **Resume** pool | `ALTER COMPUTE POOL <n> RESUME;` |
| **Modify** settings | `ALTER COMPUTE POOL <n> SET MAX_NODES=10;` |
| **Drop** pool | `DROP COMPUTE POOL <n>;` |
| **Grant USAGE** | `GRANT USAGE ON COMPUTE POOL <n> TO ROLE <r>;` |
| **Grant OPERATE** | `GRANT OPERATE ON COMPUTE POOL <n> TO ROLE <r>;` |
| **Grant MONITOR** | `GRANT MONITOR ON COMPUTE POOL <n> TO ROLE <r>;` |
| **Revoke** privilege | `REVOKE USAGE ON COMPUTE POOL <n> FROM ROLE <r>;` |
| **Get pool status** | `SELECT SYSTEM$GET_COMPUTE_POOL_STATUS('<n>');` |
| **Create service** on pool | `CREATE SERVICE <s> IN COMPUTE POOL <n> FROM SPECIFICATION $$...$$;` |
| **Show services** on pool | `SHOW SERVICES IN COMPUTE POOL <n>;` |
| **Auto-suspend OFF** | `ALTER COMPUTE POOL <n> SET AUTO_SUSPEND_SECS=0;` |
| **Auto-resume ON** | `ALTER COMPUTE POOL <n> SET AUTO_RESUME=TRUE;` |

---

## 📎 Related Concepts

| Concept | Relation to Compute Pool |
|---|---|
| **Snowpark Container Services (SPCS)** | The platform that uses compute pools to run containers |
| **Services** | Long-running containers deployed on a compute pool |
| **Jobs** | One-shot containers executed on a compute pool |
| **Image Repository** | Stores Docker images used by services/jobs on the pool |
| **Virtual Warehouse** | Separate compute for SQL — does NOT use compute pools |
| **Resource Monitor** | Can monitor/limit account-level spend including SPCS |
| **Network Policy** | Controls inbound/outbound traffic for pool nodes |
| **Stages** | Object storage used to pass data to/from containers |

---

*Last updated: May 2026 | Snowflake documentation: [docs.snowflake.com](https://docs.snowflake.com/en/developer-guide/snowpark-container-services/overview-compute-pool)*

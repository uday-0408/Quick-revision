# dbt `config()` Cheatsheet

Quick reference for the most useful parameters you can set inside `{{ config(...) }}` in models, seeds, snapshots, and tests.

## Core Behavior

| Parameter | Purpose | Example |
|---|---|---|
| `materialized` | How the model is built: `view`, `table`, `incremental`, `ephemeral`, `materialized_view` | `{{ config(materialized='table') }}` |
| `schema` | Target schema override | `{{ config(sche# dbt `config()` Cheatsheet (Complete Reference)

Quick reference for parameters you can set inside `{{ config(...) }}` in models, seeds, snapshots, sources, and tests — including adapter-specific and edge-case configs.

## Core Behavior

| Parameter | Purpose | Example |
|---|---|---|
| `materialized` | How the model is built: `view`, `table`, `incremental`, `ephemeral`, `materialized_view`, `dynamic_table` (Snowflake), `snapshot` | `{{ config(materialized='table') }}` |
| `schema` | Target schema override (appends to default unless `generate_schema_name` macro overridden) | `{{ config(schema='MARTS') }}` |
| `database` | Target database override | `{{ config(database='ANALYTICS') }}` |
| `alias` | Rename the output table/view | `{{ config(alias='sales') }}` |
| `tags` | Label models for selective `--select tag:` runs | `{{ config(tags=['finance','daily']) }}` |
| `enabled` | Turn a model on/off | `{{ config(enabled=false) }}` |
| `meta` | Arbitrary key-value metadata, surfaced in docs/manifest | `{{ config(meta={"owner": "data-eng"}) }}` |
| `group` | Assign model to a dbt group (controls cross-group reference access) | `{{ config(group='finance') }}` |
| `access` | Model visibility in a project: `private`, `protected`, `public` | `{{ config(access='public') }}` |
| `vars` | Pass model-scoped variable overrides | `{{ config(vars={"my_var": 1}) }}` |

## Incremental Models

| Parameter | Purpose | Example |
|---|---|---|
| `unique_key` | Row identifier for incremental/snapshot merges (string or list) | `{{ config(unique_key='customer_id') }}` |
| `incremental_strategy` | `merge`, `append`, `delete+insert`, `insert_overwrite`, `microbatch` | `{{ config(incremental_strategy='merge') }}` |
| `merge_update_columns` | Only update these columns on merge | `{{ config(merge_update_columns=['price','quantity']) }}` |
| `merge_exclude_columns` | Update all columns except these on merge | `{{ config(merge_exclude_columns=['created_at']) }}` |
| `on_schema_change` | `ignore`, `append_new_columns`, `sync_all_columns`, `fail` | `{{ config(on_schema_change='sync_all_columns') }}` |
| `full_refresh` | Force or block full refresh, overriding the CLI flag | `{{ config(full_refresh=true) }}` |
| `incremental_predicates` | Extra SQL predicates added to the merge condition (useful for partition pruning) | `{{ config(incremental_predicates=["DBT_INTERNAL_DEST.order_date >= '2024-01-01'"]) }}` |

### Microbatch incremental (dbt 1.9+)

| Parameter | Purpose | Example |
|---|---|---|
| `event_time` | Column dbt uses to define batch windows | `{{ config(event_time='order_date') }}` |
| `batch_size` | Granularity of each batch: `hour`, `day`, `month`, `year` | `{{ config(batch_size='day') }}` |
| `lookback` | Number of batches to reprocess on each run (handles late-arriving data) | `{{ config(lookback=2) }}` |
| `begin` | Start date for the first batch on initial build | `{{ config(begin='2023-01-01') }}` |
| `concurrent_batches` | Allow batches to run in parallel where supported | `{{ config(concurrent_batches=true) }}` |

## Performance / Storage

| Parameter | Purpose | Example |
|---|---|---|
| `cluster_by` | Cluster table data (Snowflake/BigQuery) | `{{ config(cluster_by=['order_date']) }}` |
| `partition_by` | Partition table (BigQuery, Databricks); dict supports `field`, `data_type`, `granularity`, `range` | `{{ config(partition_by={"field":"order_date","data_type":"date"}) }}` |
| `transient` | Snowflake transient table (cheaper, no fail-safe/time-travel) | `{{ config(transient=true) }}` |
| `secure` | Snowflake secure view (hides definition/underlying data) | `{{ config(secure=true) }}` |
| `automatic_clustering` | Enable Snowflake auto-clustering on a clustered table | `{{ config(automatic_clustering=true) }}` |
| `target_lag` | Snowflake dynamic table refresh lag (e.g. `'1 hour'`, `'downstream'`) | `{{ config(target_lag='1 hour') }}` |
| `snowflake_warehouse` | Warehouse used to build/refresh a dynamic table or model | `{{ config(snowflake_warehouse='TRANSFORM_WH') }}` |
| `refresh_mode` | Dynamic table refresh mode: `AUTO`, `FULL`, `INCREMENTAL` | `{{ config(refresh_mode='INCREMENTAL') }}` |
| `initialize` | Dynamic table initialize behavior: `ON_CREATE`, `ON_SCHEDULE` | `{{ config(initialize='ON_CREATE') }}` |
| `auto_refresh` | BigQuery materialized view auto-refresh toggle | `{{ config(auto_refresh=true) }}` |
| `dist` | Redshift distribution style/key | `{{ config(dist='customer_id') }}` |
| `sort` | Redshift sort key (list or single column) | `{{ config(sort=['order_date']) }}` |
| `sort_type` | Redshift sort type: `compound` or `interleaved` | `{{ config(sort_type='compound') }}` |
| `bind` | Redshift: bind view's data to underlying table at creation | `{{ config(bind=false) }}` |
| `file_format` | Spark/Databricks file format: `delta`, `parquet`, `csv`, etc. | `{{ config(file_format='delta') }}` |
| `location_root` | Spark/Databricks: base path for table storage | `{{ config(location_root='/mnt/data/tables') }}` |
| `partitions` | Athena: partition columns for table | `{{ config(partitions=['dt']) }}` |

## Hooks & Permissions

| Parameter | Purpose | Example |
|---|---|---|
| `pre_hook` | SQL run before the model builds (string or list) | `{{ config(pre_hook="DELETE FROM audit_log WHERE model='sales'") }}` |
| `post_hook` | SQL run after the model builds (string or list) | `{{ config(post_hook="GRANT SELECT ON {{ this }} TO ROLE ANALYST") }}` |
| `grants` | Auto-apply grants; supports `+` prefix to add without revoking existing | `{{ config(grants={"select":["ANALYST_ROLE"]}) }}` |
| `copy_grants` | Preserve grants on table rebuild (Snowflake) | `{{ config(copy_grants=true) }}` |

## Docs & Governance

| Parameter | Purpose | Example |
|---|---|---|
| `persist_docs` | Store descriptions in the database (`relation`, `columns`) | `{{ config(persist_docs={"relation":true,"columns":true}) }}` |
| `contract` | Enforce model matches declared YAML schema (column names/types) | `{{ config(contract={"enforced":true}) }}` |
| `docs` | Show/hide model in generated docs, or set `node_color` | `{{ config(docs={"show":true,"node_color":"#FF5733"}) }}` |

## Seeds

| Parameter | Purpose | Example |
|---|---|---|
| `column_types` | Explicit column data types for a seed | `{{ config(column_types={"id":"varchar(20)"}) }}` |
| `delimiter` | CSV delimiter character | `{{ config(delimiter=';') }}` |
| `quote_columns` | Whether to quote column names in the seed table DDL | `{{ config(quote_columns=true) }}` |

## Snapshots

| Parameter | Purpose | Example |
|---|---|---|
| `strategy` | `timestamp` or `check` change-detection strategy | `{{ config(strategy='timestamp') }}` |
| `updated_at` | Column used by the `timestamp` strategy | `{{ config(updated_at='updated_at') }}` |
| `check_cols` | Columns checked by the `check` strategy, or `'all'` | `{{ config(check_cols=['status','price']) }}` |
| `target_schema` | Schema where snapshot results are stored | `{{ config(target_schema='snapshots') }}` |
| `target_database` | Database where snapshot results are stored | `{{ config(target_database='ANALYTICS') }}` |
| `invalidate_hard_deletes` | Mark deleted source rows as expired in the snapshot | `{{ config(invalidate_hard_deletes=true) }}` |
| `hard_deletes` | (newer syntax) `ignore`, `invalidate`, or `new_record` for handling deletes | `{{ config(hard_deletes='new_record') }}` |

## Tests

| Parameter | Purpose | Example |
|---|---|---|
| `severity` | `error` or `warn` on failure | `{{ config(severity='warn') }}` |
| `error_if` | Custom threshold expression to trigger an error | `{{ config(error_if='>10') }}` |
| `warn_if` | Custom threshold expression to trigger a warning | `{{ config(warn_if='>0') }}` |
| `fail_calc` | Custom SQL expression used to calculate the failure count | `{{ config(fail_calc='count(*)') }}` |
| `limit` | Cap the number of failing rows returned | `{{ config(limit=100) }}` |
| `where` | Filter applied before the test query runs | `{{ config(where="created_at > '2024-01-01'") }}` |
| `store_failures` | Persist failing rows to a table instead of just counting | `{{ config(store_failures=true) }}` |
| `store_failures_as` | Storage format for failures: `table`, `view` | `{{ config(store_failures_as='table') }}` |

## Python Models

| Parameter | Purpose | Example |
|---|---|---|
| `submission_method` | How the python model is submitted (e.g. `all_purpose_cluster`, `job_cluster`, `serverless_cluster`) | `{{ config(submission_method='job_cluster') }}` |
| `packages` | Extra Python packages to install for the run | `{{ config(packages=['pandas','scikit-learn']) }}` |
| `cluster_id` / `create_notebook` | Databricks-specific execution controls | `{{ config(cluster_id='1234-567-abc') }}` |

## Sources & Freshness (in `.yml`, not inline `config()`, but commonly paired)

| Parameter | Purpose | Example |
|---|---|---|
| `loaded_at_field` | Column used to compute source freshness | `loaded_at_field: _etl_loaded_at` |
| `freshness.warn_after` / `freshness.error_after` | Freshness SLA thresholds | `error_after: {count: 24, period: hour}` |

## Snowflake-Specific Extras

| Parameter | Purpose | Example |
|---|---|---|
| `query_tag` | Tag queries for monitoring/debugging | `{{ config(query_tag='daily_sales_pipeline') }}` |
| `sql_header` | Prepend session-level SQL | `{{ config(sql_header="ALTER SESSION SET TIMEZONE='UTC';") }}` |

## Edge Cases & Gotchas

- **Config precedence**: inline `config()` in the SQL file overrides `dbt_project.yml`, which overrides package defaults. The most specific (file-level) setting always wins.
- **`+` prefix in `dbt_project.yml`**: when setting these same configs at the project/folder level (not inline), keys need a `+` prefix, e.g. `+materialized: table`. Inline `config()` does not use the `+`.
- **Lists vs strings**: `pre_hook`, `post_hook`, `cluster_by`, `unique_key`, and `check_cols` all accept either a single string or a list — be consistent within a project.
- **`full_refresh=false`** is a common safety pin for expensive incremental models you never want accidentally fully rebuilt, even with `dbt run --full-refresh`.
- **`contract.enforced=true`** requires every column's `data_type` to be declared in the YAML schema file, and disables some flexibility (e.g. you can't `select *`).
- **`incremental_predicates`** only takes effect with `merge`/`insert_overwrite` strategies on adapters that support it (Snowflake, BigQuery, Databricks).
- **`grants`** are *additive by default* on re-run unless you use `+grant_name` syntax or set them explicitly each run; omitted grants are revoked unless using incremental grant config.
- **`materialized='ephemeral'`** models can't have most storage/perf configs (`partition_by`, `cluster_by`, etc.) since they're never built as objects — they're inlined as CTEs.
- **Microbatch (`incremental_strategy='microbatch'`)** requires `event_time` to be set on both the model and any upstream models/sources it filters by; `--full-refresh` reprocesses all batches from `begin`.

## Quick Picks by Use Case

- **Staging models** → `materialized='view'` or `transient=true`
- **Incremental fact tables** → `materialized='incremental'`, `unique_key`, `incremental_strategy='merge'`
- **Huge append-only event tables** → `incremental_strategy='microbatch'`, `event_time`, `batch_size`, `lookback`
- **Production marts** → `schema`, `alias`, `grants`, `copy_grants`, `cluster_by`, `group`, `access`
- **Selective runs** → `tags`
- **Schema evolution control** → `on_schema_change`
- **Strict data contracts** → `contract={"enforced": true}`
- **Near-real-time Snowflake tables** → `materialized='dynamic_table'`, `target_lag`, `snowflake_warehouse`
- **Custom test tolerances** → `severity`, `error_if`, `warn_if`, `store_failures`

### Combining configs

```sql
{{ config(
    materialized='incremental',
    unique_key='customer_id',
    incremental_strategy='merge',
    schema='MARTS',
    tags=['finance','daily'],
    cluster_by=['order_date'],
    transient=true,
    copy_grants=true,
    grants={"select": ["ANALYST_ROLE"]},
    on_schema_change='sync_all_columns',
    full_refresh=false,
    meta={"owner": "finance-team"}
) }}
```

### Microbatch example

```sql
{{ config(
    materialized='incremental',
    incremental_strategy='microbatch',
    event_time='order_date',
    batch_size='day',
    lookback=2,
    begin='2024-01-01'
) }}
```ma='MARTS') }}` |
| `database` | Target database override | `{{ config(database='ANALYTICS') }}` |
| `alias` | Rename the output table/view | `{{ config(alias='sales') }}` |
| `tags` | Label models for selective `--select tag:` runs | `{{ config(tags=['finance','daily']) }}` |
| `enabled` | Turn a model on/off | `{{ config(enabled=false) }}` |

## Incremental Models

| Parameter | Purpose | Example |
|---|---|---|
| `unique_key` | Row identifier for incremental/snapshot merges | `{{ config(unique_key='customer_id') }}` |
| `incremental_strategy` | `merge`, `append`, `delete+insert`, `insert_overwrite` | `{{ config(incremental_strategy='merge') }}` |
| `merge_update_columns` | Only update these columns on merge | `{{ config(merge_update_columns=['price','quantity']) }}` |
| `on_schema_change` | `ignore`, `append_new_columns`, `sync_all_columns`, `fail` | `{{ config(on_schema_change='sync_all_columns') }}` |
| `full_refresh` | Force or block full refresh, overriding the CLI flag | `{{ config(full_refresh=true) }}` |

## Performance / Storage

| Parameter | Purpose | Example |
|---|---|---|
| `cluster_by` | Cluster table data (Snowflake/BigQuery) | `{{ config(cluster_by=['order_date']) }}` |
| `partition_by` | Partition table (mainly BigQuery) | `{{ config(partition_by={"field":"order_date","data_type":"date"}) }}` |
| `transient` | Snowflake transient table (cheaper, no fail-safe) | `{{ config(transient=true) }}` |

## Hooks & Permissions

| Parameter | Purpose | Example |
|---|---|---|
| `pre_hook` | SQL run before the model builds | `{{ config(pre_hook="DELETE FROM audit_log WHERE model='sales'") }}` |
| `post_hook` | SQL run after the model builds | `{{ config(post_hook="GRANT SELECT ON {{ this }} TO ROLE ANALYST") }}` |
| `grants` | Auto-apply grants | `{{ config(grants={"select":["ANALYST_ROLE"]}) }}` |
| `copy_grants` | Preserve grants on table rebuild (Snowflake) | `{{ config(copy_grants=true) }}` |

## Docs & Governance

| Parameter | Purpose | Example |
|---|---|---|
| `persist_docs` | Store descriptions in the database | `{{ config(persist_docs={"relation":true,"columns":true}) }}` |
| `contract` | Enforce model matches declared YAML schema | `{{ config(contract={"enforced":true}) }}` |
| `docs` | Show/hide model in generated docs | `{{ config(docs={"show":true}) }}` |

## Snowflake-Specific Extras

| Parameter | Purpose | Example |
|---|---|---|
| `query_tag` | Tag queries for monitoring/debugging | `{{ config(query_tag='daily_sales_pipeline') }}` |
| `sql_header` | Prepend session-level SQL | `{{ config(sql_header="ALTER SESSION SET TIMEZONE='UTC';") }}` |

## Quick Picks by Use Case

- **Staging models** → `materialized='view'` or `transient=true`
- **Incremental fact tables** → `materialized='incremental'`, `unique_key`, `incremental_strategy='merge'`
- **Production marts** → `schema`, `alias`, `grants`, `copy_grants`, `cluster_by`
- **Selective runs** → `tags`
- **Schema evolution control** → `on_schema_change`
- **Strict data contracts** → `contract={"enforced": true}`

### Combining configs

```sql
{{ config(
    materialized='incremental',
    unique_key='customer_id',
    incremental_strategy='merge',
    schema='MARTS',
    tags=['finance','daily'],
    cluster_by=['order_date'],
    transient=true,
    copy_grants=true,
    grants={"select": ["ANALYST_ROLE"]}
) }}
```
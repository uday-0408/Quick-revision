# dbt `config()` Cheatsheet

Quick reference for the most useful parameters you can set inside `{{ config(...) }}` in models, seeds, snapshots, and tests.

## Core Behavior

| Parameter | Purpose | Example |
|---|---|---|
| `materialized` | How the model is built: `view`, `table`, `incremental`, `ephemeral`, `materialized_view` | `{{ config(materialized='table') }}` |
| `schema` | Target schema override | `{{ config(schema='MARTS') }}` |
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
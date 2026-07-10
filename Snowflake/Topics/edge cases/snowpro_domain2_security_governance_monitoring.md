# SnowPro Core — Domain 2: Access Control, Data Governance, Monitoring & Cost Management

Sourced and cross-checked against official Snowflake documentation (docs.snowflake.com). Focus: exact defaults, edge cases, exam traps.

---

# 2.1 Access Control & Authentication

## RBAC, DAC, and the securable object hierarchy

- Snowflake combines **RBAC** (privileges → roles → users) with **DAC** (the creator/owner of an object controls grants on it by default).
- **Securable object**: anything access can be granted on. Access is **denied by default** unless an explicit grant exists.
- **Hierarchy**: Organization → Account → Database → Schema → Object (table, view, stage, function, sequence, file format, etc.). USAGE on the container (database + schema) is required to reach any object inside it — an object-level grant alone is not enough.
- Every securable object has exactly **one owner role** (normally the role that created it), which holds full privileges including the ability to grant/revoke to other roles. Ownership is **transferable**.
- **Exam trap**: Granting `SELECT` on a table without granting `USAGE` on its database and schema still fails. "At least one privilege on parent DB + at least one privilege on parent schema" is the rule (usually `USAGE`).

## System-defined roles (cannot be dropped, cannot have their base privilege set changed)

| Role | Purpose | Key facts |
|---|---|---|
| **ACCOUNTADMIN** | Top-level role; encapsulates SYSADMIN + SECURITYADMIN | Assign to a very limited number of users; not recommended for scripts/automation; first user on account creation gets this role |
| **SECURITYADMIN** | Manage grants globally, create/manage users & roles | Has global `MANAGE GRANTS` privilege; inherits USERADMIN via role hierarchy; MANAGE GRANTS lets it modify/revoke *any* grant, but does **not** itself allow creating objects |
| **USERADMIN** | Dedicated to user & role management | Has `CREATE USER` and `CREATE ROLE`; child of SECURITYADMIN by default |
| **SYSADMIN** | Create warehouses, databases, and all DB objects | Custom roles should roll up to SYSADMIN so admins can manage all account objects; if a custom role is NOT granted to SYSADMIN, sysadmins cannot manage its objects |
| **PUBLIC** | Pseudo-role auto-granted to every user/role | Can own objects like any role — those objects become available to everyone; never grant sensitive privileges to PUBLIC |

**Default hierarchy**: `USERADMIN → SECURITYADMIN → ACCOUNTADMIN`, and `SYSADMIN → ACCOUNTADMIN`. Custom roles are typically chained up to SYSADMIN.

## Functional / custom roles

- **Account roles**: standard custom roles, can hold privileges on any securable object across the account. Created via `CREATE ROLE` by USERADMIN (or any role with `CREATE ROLE`).
- **Database roles**: scoped to a **single database**; cannot be activated directly in a session (`USE ROLE` doesn't work on them) — must be granted to an account role (or another database role in the same DB) to be usable. Privileges on a database role can only reference objects **inside that database** (cross-database grants to a DB role are not possible). Created by the database owner or a role with `CREATE DATABASE ROLE`. When created, USAGE on the containing database is auto-granted to the DB role.
- Only an **account role** can hold OWNERSHIP on the database object itself (a database role cannot own the database).
- **Instance roles**: rarest type; tied to a Snowflake Class instance; granted to account roles to let users call class methods. Currently only Snowflake can create Classes.
- **Exam trap**: New role is not assigned to any user or role by default, and doesn't inherit anything — you must explicitly build the hierarchy.
- **Managed access schema**: object owners lose the ability to grant on their own objects; only the schema owner or a role with `MANAGE GRANTS` can grant (including future grants) — centralizes privilege management.

## Secondary roles

- `USE SECONDARY ROLES ALL | NONE | <role_list>` activates additional roles for a session in addition to the primary role.
- **ALL** — dynamically re-evaluated on every SQL statement; newly granted roles become active immediately without re-running the command; revoked roles drop out immediately.
- **NONE** — disables secondary roles; only the primary role's privileges apply.
- **Primary role only** performs object creation (CREATE) — ownership always goes to the primary role, never a secondary role. This is a common trap: if you create an object while secondary roles are active but haven't switched your primary role, the object is still owned by whatever your *primary* role is.
- `DEFAULT_SECONDARY_ROLES` user property. **Behavior-change bundle 2024_08 (BCR-1692)**: default value changed so that if unset/NULL, it now defaults to **('ALL')** (previously secondary roles were off unless explicitly enabled). To preserve old behavior, explicitly set `DEFAULT_SECONDARY_ROLES = ()`.
- Database roles **can** be activated as secondary roles.
- Use case: UBAC (user-based access control) — privileges granted directly to a *user* only take effect when secondary roles are active in the session.

## Network policies

- Restrict access by source IP using `ALLOWED_IP_LIST` / `BLOCKED_IP_LIST` (or, with network rules, `ALLOWED_NETWORK_RULE_LIST` / `BLOCKED_NETWORK_RULE_LIST`).
- **Blocked list is evaluated first.** Never put `0.0.0.0/0` in `BLOCKED_IP_LIST` — it would lock out everyone including you. Snowflake blocks all self-lockout attempts is NOT guaranteed by IP-list based rules the same way for network *rules* — be careful.
- Can be applied at **three levels**: account, user, and security integration (SCIM / Snowflake OAuth).
- **Precedence order (most → least specific)**: **security integration > user > account**. A more specific policy fully overrides — it does not merge with — a more general one.
- Exception: for Snowflake OAuth, the **user-level** network policy is NOT considered for client-to-Snowflake token requests — only integration-level and account-level policies apply there.
- Only **one** network policy can be active per account and per user at a time (not additive).
- Only IPv4 supported for `ALLOWED_IP_LIST`/`BLOCKED_IP_LIST`; IPv6 is supported only via **network rules** (`TYPE = IPV6`), and IPv6 rules protect the Snowflake service only — **not** internal stages (use IPv4 rules for stage protection).
- Requires `SECURITYADMIN` (or higher) or global `ATTACH POLICY` privilege to activate; `CREATE NETWORK POLICY` privilege (or SECURITYADMIN+) to create.
- `MINS_TO_BYPASS_NETWORK_POLICY` user property can temporarily bypass a lockout — but it can **only be set by Snowflake Support**, not by the customer.
- Network policies support replication and failover/failback.
- **Authentication policies vs. network policies**: network policies are evaluated first; if an IP is blocked, evaluation stops there and the authentication policy is never checked.

## Authentication methods

### Multi-Factor Authentication (MFA)
- Powered by **Cisco Duo**; no separate Duo signup needed, just install Duo Mobile.
- MFA methods: **passkey** (recommended), **authenticator app (TOTP)**, **Duo**. Duo is **not replicated**; passkeys/TOTP are.
- Historically **not enabled by default** — opt-in per user. As of BCR-1784 (2024-08), **new accounts** get a built-in authentication policy enforcing MFA enrollment for password-based human users automatically (trial and reader accounts are exempt). Service-type ACCOUNTADMIN can be exempted via `ADMIN_RSA_PUBLIC_KEY` at account creation (key-pair, no MFA needed) or a temporary `LEGACY_SERVICE` type.
- `MFA_ENROLLMENT` parameter (in an **authentication policy**): `OPTIONAL` vs `REQUIRED`. Latest BCR changed `OPTIONAL` semantics toward `REQUIRED_SNOWFLAKE_UI_PASSWORD_ONLY` as part of deprecating single-factor password logins.
- SSO users are **not required** to use Snowflake MFA by default — Snowflake trusts the IdP to enforce MFA; you can harden this with an authentication policy.
- Disabling MFA for a user requires `ACCOUNTADMIN`: `ALTER USER x SET DISABLE_MFA = TRUE`.
- Default second-factor mechanism for Duo is **Duo Push**; a Duo passcode can be used instead via `--mfa-passcode` (CLI) or `passcodeInPassword=on` (JDBC).

### Federated Authentication / SSO
- Uses an external IdP compatible with **SAML 2.0**.
- Enables single sign-on; Snowflake becomes the Service Provider (SP), IdP handles credential verification.
- Works alongside SCIM (System for Cross-domain Identity Management) for automated user/role provisioning from the IdP.

### OAuth
- Two flavors: **Snowflake OAuth** (built-in, no external IdP needed) and **External OAuth** (integrates with an external OAuth authorization server, e.g. Okta, Azure AD, Ping).
- Used heavily by partner/BI tool integrations.
- Network policy on Snowflake OAuth security integration: user-level policy is skipped for token requests (see Network Policies above).

### Key-pair authentication
- Alternative to username/password, common for **service accounts / non-interactive connections**.
- Minimum **2048-bit RSA** key pair generated via **OpenSSL only** (keys from PuTTYgen/OpenSSH on Windows are NOT supported).
- Public key assigned via `ALTER USER ... SET RSA_PUBLIC_KEY = '...'`.
- Snowflake supports **two key slots per user**: `RSA_PUBLIC_KEY` and `RSA_PUBLIC_KEY_2` — this enables **zero-downtime key rotation**: register the new key in slot 2, migrate all clients over, then remove/replace slot 1.
- **Exam trap**: Key-pair auth is designed for programmatic/service use, not paired with interactive MFA prompts. Recommended pattern: SSO+MFA for humans, key-pair for service accounts.
- JWT for key-pair auth: max lifetime **1 hour**; subject/issuer built from `<ACCOUNT>.<USERNAME>.<PUBLIC_KEY_FINGERPRINT>` (SHA-256, prefixed `SHA256:`).

## Account identifiers

- **Preferred format**: `organization_name-account_name` (e.g., `myorg-account123`). This is what should be used going forward, including in replication/failover commands, Okta SSO, SCIM, private connectivity URLs.
- **Legacy format**: `account_locator` (+ optional cloud region / cloud provider segments depending on region). Discouraged but still supported.
- Underscore in account name breaks some integrations (Okta SSO, SCIM don't support `_` in URLs) — a hyphen-substituted version must be used in those URL contexts.
- Constraints on the identifier string: must start with a letter, no spaces/special chars except `_`, shouldn't end in `_`, combined org+account+hyphens string ≤ **63 characters**.
- `CURRENT_ACCOUNT()` returns the **account locator**; `CURRENT_ACCOUNT_NAME()` returns the **account name** (part of the preferred identifier). Common exam trap — know which function returns which.
- VPS (Virtual Private Snowflake) editions use a different locator naming convention.

## Logging and tracing (event tables)

- **No event table exists by default** in older accounts — behavior-change bundle 2024_06 introduced a **default event table**: `SNOWFLAKE.TELEMETRY.EVENTS` (in the `SNOWFLAKE` database, `TELEMETRY` schema), auto-activated if no other event table is active.
- Telemetry types: **log messages**, **trace events** (structured, grouped into spans), **metrics** (CPU/memory for procs & UDFs).
- Levels:
  - `LOG_LEVEL` (log messages via logging APIs) / `LOG_EVENT_LEVEL` (system-generated EVENT records e.g. Snowpipe, tasks, dynamic tables) — severity thresholds; only that level and *more severe* are captured.
  - `METRIC_LEVEL` — binary: **ALL** or **NONE**.
  - `TRACE_LEVEL` — `ALWAYS` (overrides all), `ON_EVENT` (only capture trace data explicitly emitted by your code), `OFF`.
- **Precedence**: session-level and object-level settings override account defaults; more specific (object) beats less specific (account/database). An **event table associated with a database takes precedence over one associated with the account** for objects in that database.
- SQL text capture during tracing is governed separately by `SQL_TRACE_QUERY_TEXT` (ACCOUNTADMIN-only to set); captures up to 1024 characters.
- Event table is a special table type: predefined immutable columns, cannot be `UPDATE`d, but supports `SELECT`, `TRUNCATE`, `DELETE`. Only **one event table can be active per account** at a time. Max span payload: 128 trace events / 128 span attributes per span; log/trace payload capped at 1 MB.
- Logging/tracing/metrics collection is a **serverless** feature — billed, no warehouse required. Query `EVENT_USAGE_HISTORY` for cost.

---

# 2.2 Data Governance

## Column-level Security — Dynamic Data Masking

- **Masking policy** = schema-level object; applied to a column via `ALTER TABLE ... MODIFY COLUMN ... SET MASKING POLICY`.
- **One masking policy per column at a time** (cannot stack two directly) — combine logic in a single `CASE` expression if multiple rules are needed.
- Evaluated **at query runtime**, on every occurrence of the column (including in views built on it) — underlying data is never physically altered.
- `CURRENT_ROLE()` inside a policy evaluates the **session's active role** regardless of SQL execution context (e.g. views owned by another role); `INVOKER_ROLE()` evaluates the **execution context** (e.g. who called the view) — a very common exam trap distinguishing the two.
- **Conditional masking**: policy can reference additional (non-masked) columns to decide masking logic — those extra columns cannot themselves already be protected by a row access policy or reside in an external table's VALUE-protecting policy without extra config.
- Tag-based masking policies: attach a masking policy to a **tag**, then apply the tag to a column (or to a table/schema/database, cascading to newly added matching-type columns) — huge win for managing policy assignment at scale without touching every column.
- Collision risk: hash/crypto masking functions are NOT guaranteed 1:1; collisions become likely once input cardinality approaches the square root of the output space — never use hashing in a masking policy where uniqueness must be guaranteed.
- Alternative: **External Tokenization** — calls an external function to detokenize; raw value never has to live in Snowflake at rest. Contrast: Dynamic Data Masking works fine with **Secure Data Sharing**; external tokenization does **not** (external functions can't run in a share's execution context).

## Row-level Security — Row Access Policies

- Schema-level object; boolean expression; attached via `ALTER TABLE/VIEW ... ADD ROW ACCESS POLICY ... ON (...)`.
- **When both a row access policy and masking policy(s) exist on the same object, the row access policy is evaluated FIRST**, then masking is applied to the surviving rows.
- A given column **cannot** be referenced in both a row access policy signature and a masking policy signature at the same time.
- Row access policies **disable common statistics-based query shortcuts** (e.g., fast `COUNT(*)`, `MAX()` using metadata) since Snowflake must evaluate row visibility per query — plan for potential performance impact.
- Subqueries/JOINs inside the policy body are supported but should be minimized (can cause errors/perf issues); recursive CTEs are disallowed on aggregation-constrained tables (see below), and row access policies attached to tables already protected by other row/masking policies can also error.
- Cloning: cloning a schema clones all policies within it; a cloned table maps to the cloned policy (not the original) if source and policy live in the same schema/DB. If an external table is involved, cloning a database clones the row access policy definition but **not** the external table itself — the policy can reference a now-missing table.

## Aggregation Policies (Privacy)

- Force queries against the protected table/view to **aggregate** (SUM, COUNT, AVG, GROUP BY, etc.) rather than return individual rows — minimum group size specified by the policy.
- Two modes:
  - **Row-level privacy** (no entity key): minimum group size = minimum *rows* per group.
  - **Entity-level privacy** (entity key specified, e.g. `email` or `user_id`): minimum group size = minimum *distinct entities* per group, even across multiple rows for the same entity, even if the key column itself doesn't appear in the query.
- **Remainder groups**: if a group would be smaller than the minimum, Snowflake merges it into a NULL-keyed remainder group rather than exposing it.
- Restrictions on protected tables: no `ROLLUP`/`CUBE`/`GROUPING SETS`, no window functions, no recursive CTEs, most set operators disallowed (UNION ALL is the notable exception, each source must independently satisfy min group size).
- Interaction with other policies: **masking policies are enforced BEFORE aggregation policies** (aggregation operates on already-masked data); **projection policies are enforced AFTER aggregation policies**. A masking/row-access/projection policy body **cannot reference** an aggregation-constrained table.
- Not bulletproof: with enough creative queries a determined analyst could still infer values — Snowflake explicitly states this is intended for trusted partners, not a hard privacy guarantee.
- Applying to a **view** does not make the underlying table aggregation-constrained.

## Projection Policies (Privacy)

- Controls whether a column can appear in the **final projected output** of a query — not whether it can be used internally (WHERE clauses, inner subqueries, joins can still leak information indirectly — this is an explicit documented limitation, not a bug).
- One projection policy per column max.
- Projection-constrained column shows as `NULL` in the outer query result; functions on it also return NULL; but a nested/inner query is NOT subject to projection-policy suppression (only the outermost projected result is).
- Cannot be applied to a virtual column / VALUE column in an external table directly (workaround: build a view over the external table and apply the policy there); also disallowed as the `value_column` in a `PIVOT`.
- A projection policy body cannot reference a column already protected by a masking policy, nor a table protected by a row access policy.
- **Exam trap**: projection policy ≠ full row-level privacy — it hides a *column*, not the fact that a specific individual's row exists in a result.

## Object Tagging

- Schema-level object; key–value pair applied to almost any Snowflake object (tables, views, columns, warehouses, users, the account itself, etc.).
- **Current (2026) quotas** — verify against docs, these have grown over time:
  - Max **50 unique tags** on a single object (e.g., a table).
  - Max **50 unique tags** across all columns of a single table (separate pool from the table-level limit).
  - Max **300 allowed values** per tag definition (via `ALLOWED_VALUES`).
  - Tag string value: max **256 characters**.
  - Max **10,000 tags** per account (dropped tags count toward this for 24 hours, i.e. until purge/undrop window closes).
- **Tag lineage** (inheritance): a tag set on a table is inherited by its columns by default; a same-named tag set directly on the column overrides the inherited value. This is distinct from **tag propagation** (below).
- **Tag propagation** (`PROPAGATE` parameter, Enterprise+): automatically copies a tag from a source object to downstream objects.
  - `ON_DEPENDENCY` — propagate when a target object depends on the source (e.g., a view built on a table).
  - `ON_DATA_MOVEMENT` — propagate when data physically moves (CTAS, INSERT INTO, etc.).
  - Conflict resolution (`ON_CONFLICT`): default is the literal string `'CONFLICT'`; can set a custom string; or `ALLOWED_VALUES_SEQUENCE` (first-matching value in the `ALLOWED_VALUES` list wins).
  - Propagation is capped at **10,000 downstream objects per triggering transaction**.
  - Inherited tags are **not** further propagated; tags do **not** propagate from a share to local objects.
- Requires `CREATE TAG` (schema-level) to define, `APPLY TAG` (global or object-scoped) to assign.
- `SYSTEM$GET_TAG_ALLOWED_VALUES()` — returns allowed values (NULL = unrestricted).
- Future grants on tags are **not** supported (workaround: grant `APPLY TAG` to a role).
- A tag **can be dropped even while still assigned to objects** — except tags referenced by an active masking policy (must UNSET first). Best practice: `UNSET` from all objects before `DROP` to avoid the 24-hour undrop-restoration surprise.

## Privacy Policies (umbrella term) & the "Privacy in Snowflake" feature set
- Umbrella term covering **aggregation policies**, **projection policies**, and **differential privacy** — all designed to let a data *provider* keep control over shared data even after a *consumer* queries it (crucial for Secure Data Sharing / Marketplace scenarios).
- **Exam trap**: don't confuse "privacy policies" (aggregation/projection/differential privacy) with "masking policy" / "row access policy" (classic column/row-level security) — the exam outline lists them as a distinct governance feature.

## Trust Center

- Framework to **evaluate, monitor, and reduce security risk** in a Snowflake account, based on metadata-driven **scanners**, grouped into **scanner packages**.
- Built-in scanner packages: **Security Essentials** (enabled **by default**, e.g. checks MFA enforcement + network policy use), **CIS Benchmarks** (checks against the CIS Snowflake Foundations Benchmark, off by default, evaluates dozens of numbered checks like "1.1", "1.4"), **Threat Intelligence**, **AI Security**. Only Security Essentials is on out of the box — all others must be explicitly enabled.
- After enabling a scanner package, it **runs immediately**, then follows its schedule (default once daily; configurable, except Security Essentials' schedule cannot be changed).
- Roles: `SNOWFLAKE.TRUST_CENTER_VIEWER` (read findings) vs `SNOWFLAKE.TRUST_CENTER_ADMIN` (manage scanners) — application roles granted by ACCOUNTADMIN.
- **Important exam nuance**: Trust Center checks whether a security measure exists (metadata-based), NOT whether it's implemented effectively — absence of a violation ≠ guaranteed good security posture. Section 2 of the CIS benchmark ("activities are monitored") isn't fully automatable and surfaces only via a raw query against `snowflake.trust_center.findings`.
- Trust Center scans incur **serverless compute cost** — trackable via `ACCOUNT_USAGE.METERING_HISTORY` filtered on `service_type = 'TRUST_CENTER'`.
- Extensible via **Trust Center Extensions** — Native App Framework packages that add custom scanner packages (from partners or self-built), shareable within an org or via Marketplace.
- Not available for Snowflake **reader accounts**.

## Encryption Key Management

- **All customer data encrypted by default**, AES-256, hierarchical key model rooted in a cloud-provider HSM. Zero customer configuration required for baseline encryption.
- **Hierarchical key model** (parent encrypts child): Root Key (HSM) → Account Master Key (AMK) → Table Master Key (TMK) → File Key (encrypts the actual micro-partition data file).
- **Key rotation**: Account Master Key and Table Master Keys are **automatically rotated every 30 days**. A key transitions Active (originator/encrypt usage) → Retired (recipient/decrypt-only usage). Rotation limits how long any single key is actively used to encrypt new data — a NIST-recommended practice.
- **Rekeying**: independent, orthogonal concept from rotation. Re-encrypts *existing* data files with a brand-new key so the old (retired) key can finally be destroyed. **Retired keys older than 1 year get automatically re-encrypted (rekeyed)** if **Periodic Rekeying** is enabled (`ALTER ACCOUNT SET PERIODIC_DATA_REKEYING = TRUE`). Requires **Enterprise Edition or higher**, ACCOUNTADMIN to enable.
  - Mnemonic: **Rotation = "new data gets a fresh key." Rekeying = "old data gets a fresh key" (and the old key can then be destroyed).**
- **Tri-Secret Secure (TSS)**: combines a Snowflake-maintained key + a **customer-managed key (CMK)** hosted in the customer's own cloud KMS → composite master key. Requires **Business Critical Edition or higher**. If the customer revokes/deletes the CMK, Snowflake **can no longer decrypt** that account's data (customer holds the "kill switch").
  - Composite master key is never used to encrypt raw data directly — it wraps the table master keys, which derive file keys.
  - Self-service CMK rotation via `SYSTEM$REGISTER_CMK_INFO`, `SYSTEM$ACTIVATE_CMK_INFO`, `SYSTEM$GET_CMK_INFO`, `SYSTEM$DEACTIVATE_CMK_INFO`. Rekeying after CMK swap can take under an hour, up to 24 hours; account stays available during the process. **Never delete/revoke an old CMK version before confirmation that rekeying finished** — doing so causes data loss.
  - Only **one CMK** can be registered at a time.
- **Exam trap**: don't confuse "Tri-Secret Secure" (Business Critical+, customer-managed key layer) with plain automatic Snowflake-managed rotation/rekeying (available on lower editions, periodic rekeying needs Enterprise+).

## Alerts

- Schema-level object. Structure: **condition** (IF EXISTS SQL) → **action** (THEN SQL, e.g. `SYSTEM$SEND_EMAIL` or `SYSTEM$SEND_SNOWFLAKE_NOTIFICATION`) → **schedule** (interval in minutes, or `USING CRON`).
- Compute model: either a **user-specified virtual warehouse** or **serverless** (omit the `WAREHOUSE` parameter). For infrequent conditions, serverless avoids the ≥1-minute minimum warehouse charge that even a trivial email-send action would otherwise incur.
- Serverless alert compute caps out at the equivalent of an **XXLARGE warehouse**; billed per-second, rounded up to the nearest second.
- **New alerts are created SUSPENDED by default** — must explicitly `ALTER ALERT ... RESUME` to activate. Classic exam gotcha.
- Max scheduling interval: **11520 minutes (8 days)** — beyond that the alert never evaluates.
- If a scheduled evaluation is still running when the next scheduled time arrives, that run is **skipped** (no overlap/backlog).
- `SCHEDULED_TIME()` / `LAST_SUCCESSFUL_SCHEDULED_TIME()` — special functions usable only inside an alert's condition/action, useful for "only process new rows since last successful run" patterns.
- CREATE/ALTER ALERT do **not** validate identifier resolution, data types, or function signatures at definition time — a broken condition/action only errors at execution time.
- Privileges: `EXECUTE ALERT` (warehouse-based) or `EXECUTE MANAGED ALERT` (serverless) — global, grantable only by ACCOUNTADMIN.
- **Alerts are schedule/poll-based, not event-driven** — they are not real-time triggers; they check periodically.

## Notifications

- Distinct from Alerts, though often paired with them. Notification Integrations let Snowflake push messages to external destinations:
  - **Email** (via `SYSTEM$SEND_EMAIL`) — requires a verified email address on the recipient's Snowflake user; the mail relay historically runs on AWS in specific regions regardless of your account's region, so avoid putting PII/sensitive content in the email body.
  - **Cloud provider queue**: Amazon SNS, Azure Event Grid, Google Cloud Pub/Sub — commonly used for **Snowpipe error notifications** and **task failure notifications**.
  - **Webhook** notifications — JSON payloads to arbitrary endpoints (e.g., PagerDuty, Slack via webhook relay).
- Resource Monitor notifications are a separate, narrower mechanism: **email-only**, sent to account admins with notifications enabled plus up to **5 named non-administrator users** (`NOTIFY_USERS`) — every named user must have a **verified email** or the `CREATE/ALTER RESOURCE MONITOR` statement **fails**.

## Data Replication and Failover

- **Database replication and share replication**: available on **all editions**.
- **Replication of all other account objects (users, roles, warehouses, network policies, security integrations, etc.) + Failover/Failback + Client Redirect**: require **Business Critical Edition or higher**.
- **Replication group** — defines *what* to replicate (databases, shares, specific account object types), *where*, and *how often* (read/write remains on the primary; replicas are read-only).
- **Failover group** — same as a replication group but adds the ability to **promote** a secondary to become the new primary (failover) and later demote it back (failback). Business Critical+ only.
- A database or share object can only belong to **one** replication OR failover group (not both simultaneously); an account can only have **one** replication/failover group containing non-database/non-share objects (i.e., one "account objects" group per account).
- Inbound shares (received from a provider) **cannot** be replicated — only **outbound** shares you own.
- Upgrading to Business Critical Edition can take **up to 12 hours** before failover capability becomes usable.
- Recommended `REPLICATION_SCHEDULE` for optimized refresh: **10 minutes or less**, for small incremental syncs.
- Streams, tasks: replicated streams only track changes for tables/views within the *same* replicated database; task graphs owned by a different role than the replicating role are not replicated correctly via database replication.
- Client Redirect: a stable connection URL that can redirect Snowflake clients to a different (promoted) account during failover — used for seamless application-level continuity.

## Data Lineage

- **Native, automatic** — captured whenever data moves or an object depends on another (CREATE TABLE AS SELECT, INSERT INTO, MERGE, view definitions, etc.) — no manual instrumentation required for in-Snowflake objects.
- **Object-level (table) lineage**: available via `OBJECT_DEPENDENCIES` (ACCOUNT_USAGE view) on **all editions** — lightweight, no column detail.
- **Column-level lineage**: derived from `ACCESS_HISTORY` (`OBJECTS_MODIFIED` JSON column mapping source→target columns per write). Requires **Enterprise Edition or higher**.
- **Snowsight Lineage tab** (visual graph, tags/masking-policy overlay, upstream/downstream neighborhood exploration) and the `GET_LINEAGE` (SNOWFLAKE.CORE) programmatic function also require **Enterprise Edition or higher**. `GET_LINEAGE` has a max traversal depth of **5 levels**.
- **ML Lineage**: tracks Feature Views, Datasets, Models (Snowflake ML / Feature Store / Model Registry). Predictions tables do **not** currently retain a lineage link back to the model that produced them (documented gap). Lineage is not replicated across accounts.
- **External lineage** (dbt, Airflow, Spark, etc.): not captured automatically — requires explicit ingestion via the **OpenLineage** standard or the Snowflake REST API, and is currently **table-level only** (no column-level detail for externally-sourced lineage) — a known gap he's already studied.
- Lineage metadata retention: **365 days (1 year)** in ACCOUNT_USAGE, same overall latency characteristics as other ACCOUNT_USAGE views (45 min–3 hrs).
- Deleted/dropped objects remain visible in the lineage graph for **14 days**.
- **Exam trap**: `OBJECT_DEPENDENCIES` (table-level, all editions) vs `ACCESS_HISTORY` (column-level, Enterprise+) vs Snowsight Lineage UI (visual, Enterprise+) — know which one satisfies which requirement.

---

# 2.3 Monitoring and Cost Management

## Resource Monitors

- Purpose: **track and cap credit consumption** at the **account level** (one account-level monitor max) or **warehouse level** (multiple warehouse-level monitors; each warehouse can be assigned to only **one** resource monitor at a time — reassigning moves it and resets used-credit tracking to zero under the new monitor).
- Created/managed only by **ACCOUNTADMIN** (or a role explicitly granted the privilege) — no lower system role can create them by default.
- **`CREDIT_QUOTA`** — total credits allowed per interval (default: no quota → never triggers). Includes warehouse compute + serverless + cloud services usage attributed to the monitored warehouse(s)/account — **does not** include the daily 10% cloud-services adjustment/waiver.
- **`FREQUENCY`** — `MONTHLY` (default reset behavior if unspecified — legacy default), `DAILY`, `WEEKLY`, `YEARLY`, `NEVER` (credits never reset).
- **`TRIGGERS ... ON <percent> PERCENT DO <action>`** — up to **5** actions; percentages can exceed 100 (e.g., 110%) to allow controlled overage before hard suspension.
- Actions:
  - **NOTIFY** — email alert to admins with notifications enabled + `NOTIFY_USERS` list (max 5 non-admin users); no suspension.
  - **SUSPEND** — waits for **currently executing queries to finish**, then blocks new queries/prevents warehouse resume; graceful.
  - **SUSPEND_IMMEDIATE** — **cancels all running queries immediately** and suspends; hard stop.
- **Exam trap**: warehouses may take some time to actually suspend even on `SUSPEND_IMMEDIATE`, so credits can still accrue briefly past the threshold — best practice is to set trigger thresholds with a buffer (e.g., 90% instead of 100%) rather than relying on exact enforcement.
- If a resource monitor's threshold blocks a warehouse from resuming, an admin must **raise the credit quota or the trigger threshold** via `ALTER RESOURCE MONITOR` to unblock it — `TRIGGERS` is **not additive** on ALTER; it replaces the full trigger set, so you must re-specify existing triggers plus new ones.
- `START_TIMESTAMP` / `END_TIMESTAMP` — optional schedule bounds; if an `END_TIMESTAMP` is reached, **all assigned warehouses are suspended regardless of quota usage**, and a synthetic "reached X% quota, triggered SUSPEND_IMMEDIATE" notification is generated even if the real percentage doesn't match any configured trigger.
- Notification requires each named non-admin user to have a **verified email** — otherwise the CREATE/ALTER statement **fails outright** (not a silent failure at creation; but if a verified email later becomes unverified, notification then **fails silently**).

## Virtual Warehouse Credit Usage

- **Standard (Gen1) credits/hour by size**, doubling each tier: **XS=1, S=2, M=4, L=8, XL=16, 2XL=32, 3XL=64, 4XL=128, 5XL=256, 6XL=512**. (5XL/6XL not GA in every region.)
- Compute nodes double with each size step: XS = 1 node, S = 2 nodes, M = 4 nodes, etc. — each node has a fixed core/RAM footprint; doubling warehouse size doubles cores, RAM, and local disk cache.
- **Gen2 warehouses**: faster hardware/software; cost **more credits/hour** than Gen1 for the same size (roughly 1.25×–1.35× depending on cloud), max size 4XL (no 5XL/6XL on Gen2).
- **Snowpark-optimized warehouses**: higher memory-per-node ratio for ML/memory-intensive workloads; different (higher) credit table, uses `MEMORY_1X` / `MEMORY_16X` resource constraints.
- **Billing model**: per-second billing with a **60-second (1-minute) minimum** when a warehouse starts/resumes/upsizes. Suspended warehouses incur **zero** compute cost. Resizing **up** only bills for the newly added resources. Resizing **down** from 5XL/6XL to 4XL-or-smaller briefly double-bills (old + new) during the quiesce period.
- Formula: `credits = (warehouse size credit rate) × (actual running seconds / 3600)`, then `$ cost = credits × price-per-credit` (varies by edition/region/contract).
- **Cloud Services layer**: charged only when daily cloud-services consumption **exceeds 10% of that day's total warehouse (compute) credit consumption** — the "10% daily adjustment." This is the free-tier buffer resource monitors do **not** additionally waive.
- Multi-cluster warehouses: designed for **concurrency scaling**, not raw per-query speed — requires Enterprise Edition or higher.
- Query performance heuristic for choosing warehouse size: aim for roughly **250 micro-partitions per thread** (each node ≈ 8 threads/cores) — oversizing a warehouse for a small scan wastes credits with idle threads; undersizing causes spill/queueing.

## ACCOUNT_USAGE schema vs INFORMATION_SCHEMA

| Aspect | INFORMATION_SCHEMA | ACCOUNT_USAGE |
|---|---|---|
| Scope | Per-database (plus some account-level views/table functions) | Account-wide, all databases, part of shared `SNOWFLAKE` database |
| Latency | **None — real-time** | **Latency exists**: most views ~2 hours (120 min); range is 45 min–3 hrs depending on view; some views (e.g. tag-related) can lag up to **2 days** under specific low-activity conditions |
| Retention | Short: **7 days up to 6 months** depending on the view | Long: **1 year (365 days)** for historical usage views |
| Dropped objects | **Not shown** (current-state only) | **Included** — dropped object rows remain, with NULL name columns once fully purged (parent ID columns also go NULL) |
| Default access | Governed by the querying role's own privileges | By default only `ACCOUNTADMIN`; must explicitly grant `IMPORTED PRIVILEGES` on the `SNOWFLAKE` database, or (preferred, more granular) grant specific **SNOWFLAKE database roles** like `OBJECT_VIEWER`, `USAGE_VIEWER`, `GOVERNANCE_VIEWER`, `SECURITY_VIEWER` |
| Consistency | No guaranteed consistency under concurrent DDL during a long-running query | N/A (historical/derived) |

- Frequently-tested views: `QUERY_HISTORY`, `LOGIN_HISTORY`, `WAREHOUSE_METERING_HISTORY`, `STORAGE_USAGE`, `LOAD_HISTORY`/`COPY_HISTORY`, `ACCESS_HISTORY`, `METERING_HISTORY`, `RESOURCE_MONITORS` (a `READER_ACCOUNT_USAGE`-only view for reader-account resource monitors).
- `ORGANIZATION_USAGE` — a further-zoomed-out schema, aggregating across **all accounts in the organization** (still under the shared `SNOWFLAKE` database).
- **Exam trap**: "I need real-time data" → `INFORMATION_SCHEMA`. "I need to see a dropped table's history" or "I need >7 days of history" → `ACCOUNT_USAGE`.

---

## Quick Cross-Topic Comparison Table (common exam confusions)

| A | B | Key distinguishing fact |
|---|---|---|
| Masking policy | Row access policy | Masking hides column values; row access filters whole rows. If both exist, **row access evaluated first**. |
| Masking policy | Projection policy | Masking substitutes a value; projection hides the column from output entirely (returns NULL), but doesn't stop it being used in WHERE/inner subqueries. |
| Aggregation policy | Row access policy | Aggregation forces GROUP BY/aggregate functions with a minimum group size; row access filters rows outright, no aggregation requirement. |
| CURRENT_ROLE() | INVOKER_ROLE() | CURRENT_ROLE = session's active role; INVOKER_ROLE = execution context (who's really running the query, e.g. through a view). |
| Account role | Database role | Account role: account-wide scope, directly usable via USE ROLE. Database role: single-DB scope, must be granted to an account role to be activated. |
| Tag lineage | Tag propagation | Lineage = automatic inheritance down the object hierarchy (table→column). Propagation = explicit cross-object copying on dependency/data movement (Enterprise+, opt-in via PROPAGATE). |
| Key rotation | Rekeying | Rotation = new key issued every 30 days for new data. Rekeying = old (retired >1yr) data re-encrypted with a new key so the old key can be destroyed (needs Periodic Rekeying, Enterprise+). |
| Replication group | Failover group | Both replicate; only a failover group supports promotion (failover/failback) — Business Critical+ required for both non-database objects and for any failover capability. |
| OBJECT_DEPENDENCIES | ACCESS_HISTORY | Table-level lineage, all editions vs. column-level lineage, Enterprise+. |
| INFORMATION_SCHEMA | ACCOUNT_USAGE | Real-time/short retention/no dropped objects vs. latent/1-year retention/includes dropped objects. |
| SUSPEND | SUSPEND_IMMEDIATE | Graceful (finish running queries) vs. hard cancel of in-flight queries. |
| Network policy | Authentication policy | Network policy evaluated first (IP-based); if it blocks, auth policy is never reached. |

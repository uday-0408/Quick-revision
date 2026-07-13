# 3.3 Snowflake Connectors and Integrations
*SnowPro-style study notes — sourced from official Snowflake Documentation (docs.snowflake.com)*

---

## 🗺️ The Big Picture (mental map first)

Before the details, lock in this map — it is the #1 source of exam confusion:

| Snowflake calls it... | What it actually is | Examples |
|---|---|---|
| **Drivers** | Language-level libraries you install in *your* application to talk to Snowflake (client-side) | JDBC, ODBC, .NET, Node.js, Go, PHP PDO, **Python Connector** |
| **Connectors** | Purpose-built bridges between Snowflake and a *specific external platform/ecosystem* | Kafka Connector, Spark Connector |
| **Storage Integration** | A Snowflake object that lets Snowflake securely access **your cloud storage** (S3/GCS/Azure) without you handing over secret keys | Used by external stages, `COPY INTO`, external tables |
| **API Integration** | A Snowflake object that lets Snowflake securely call an **external HTTPS proxy service** (API Gateway) | External functions, Git integration, external MCP servers |
| **Git Integration** | Uses an API Integration (+ optionally a Secret) to sync a **remote Git repo** into a special stage inside Snowflake | GitHub, GitLab, BitBucket, Azure DevOps, AWS CodeCommit |

> ⚠️ **Exam trap #1:** The Snowflake Python Connector is documented under **Drivers**, not under **Connectors**. "Connectors" (in official docs) refers specifically to the **Kafka** and **Spark** connectors.

---

## 1. Snowflake Drivers

### 1.1 What they are
Drivers let you write applications in languages like **Go, C#, JavaScript, Java, PHP, and Python** that connect to Snowflake and perform standard operations (query execution, loading, etc.).

### 1.2 The official driver list

| Driver | Purpose |
|---|---|
| **Go Snowflake Driver** | Interface for developing applications in Go |
| **JDBC Driver** | Connects from any client tool/application that supports JDBC |
| **.NET Driver** | Interface to the Microsoft .NET framework |
| **Node.js Driver** | Native **asynchronous** Node.js interface |
| **ODBC Driver** | Connects from ODBC-based client applications |
| **PHP PDO Driver** | Interface for PHP applications |
| **Snowflake Connector for Python** | Pure Python package for Python applications |

### 1.3 TLS support (frequently tested table)

All Snowflake drivers support **TLS 1.2**. Most also support **TLS 1.3**, with one notable exception:

| Driver | TLS 1.2 | TLS 1.3 | Note |
|---|---|---|---|
| Go Driver | ✔ | ✔ | — |
| JDBC Driver | ✔ | ✔ | — |
| **.NET Driver** | ✔ | **Partial** | macOS does **not** currently support TLS 1.3 for .NET (pending .NET 10); Windows supports TLS 1.3 for .NET 3.0+/.NET Framework 4.8+ |
| Node.js Driver | ✔ | ✔ | — |
| ODBC Driver | ✔ | ✔ | — |
| PHP PDO Driver | ✔ | ✔ | — |
| Python Connector | ✔ | ✔ | — |

> ⚠️ **Exam gotcha:** If a question asks "which driver has an exception to TLS 1.3 support," the answer is **.NET**, and specifically on **macOS**.

### 1.4 JDBC Driver — key facts
- Snowflake provides a **JDBC type 4 driver** supporting core JDBC functionality.
- Must be installed in a **64-bit environment**.
- Requires **Java LTS 1.8 or higher**.
- `sfsql` (deprecated CLI client) was an example of a JDBC-based application.
- Supports **OAuth 2.0 Client Credentials flow** for machine-to-machine (M2M) authentication.
- Supports **workload identity federation** — drivers can auto-obtain short-lived credentials from a platform's identity provider (`authenticator = WORKLOAD_IDENTITY`, with `workloadIdentityProvider` = AWS/AZURE/GCP/OIDC).
- OCSP (Online Certificate Status Protocol) checks the revocation status of the TLS certificate during connection handshake. This is configurable (`ocspFailOpen`), and OCSP-related failures are a classic real-world connectivity issue.

### 1.5 ODBC Driver — key facts
- Prerequisites **differ by platform/OS** — always check the platform-specific install guide.
- Different **versions** of the ODBC driver support `GET`/`PUT` commands depending on which **cloud platform** hosts the Snowflake account.

> ⚠️ **Exam gotcha:** ODBC driver capability (e.g., GET/PUT support) can depend on **both the driver version and the underlying cloud provider (AWS/Azure/GCP)** — don't assume uniform behavior across clouds.

### 1.6 Snowflake Connector for Python — key facts
- It is a **pure Python package**, distributed via **PyPI**, released under **Apache License 2.0**.
- It does **not** depend on JDBC or ODBC — it talks to Snowflake natively.
- Supports all standard operations: DDL/DML, bulk loading, querying, retrieving query IDs, cancelling queries by query ID, binding data, and retrieving column metadata.
- **Snowflake SQLAlchemy** toolkit is built on top of the Python Connector for ORM-style access.
- For advanced Python workloads that need in-Snowflake compute, **Snowpark for Python** is recommended (DataFrame ops, UDFs, stored procedures run *inside* Snowflake — no data movement needed).

### 1.7 Connectivity troubleshooting: SnowCD
- **SnowCD** (Snowflake Connectivity Diagnostic Tool) is used to **evaluate and troubleshoot network connectivity** to Snowflake.
- Can be used both during **initial setup** and **on-demand** at any time afterward.

### 1.8 Account identifier gotcha (applies to all drivers/connectors)
- For the **Snowflake Python APIs**, **underscores are not supported** in the account identifier setting — replace underscores with dashes.
- Account identifier format is generally `orgname-account_name` (preferred) or the legacy `account_locator` format — the exact parameter name/format differs slightly between JDBC, ODBC, Python, Node.js, and .NET connection strings.

> ⚠️ **Exam gotcha:** A question may show an account identifier with an underscore used in a Python connection and ask "why does this fail?" → underscores aren't valid there; must use a dash.

---

## 2. Snowflake Connectors (Kafka & Spark)

Per official docs, this section covers "connectors described in this section [that let you] connect Snowflake with systems external to it" — specifically **Kafka** and **Spark**.

### 2.1 Kafka Connector

#### Core concept
- Apache Kafka uses a **publish/subscribe** model. Producers publish to a **topic**; consumers subscribe to topics. Topics can be split into **partitions** for scalability.
- **Kafka Connect** is the framework that connects Kafka to external systems (a separate cluster from the Kafka cluster itself).
- The **Snowflake Connector for Kafka** runs inside a Kafka Connect cluster, reads from Kafka topics, and writes rows into Snowflake tables.
- Two distribution packages exist:
  - **Confluent package version** (also available as a hosted connector inside **Confluent Cloud**)
  - **Open source software (OSS) Apache Kafka package**

> ⚠️ **Exam gotcha (2026 update):** The **classic Kafka connector (v3 and earlier)** is being phased out. It is still fully supported today, but Snowflake plans a formal deprecation announcement (mid-2026) followed by an 18-month migration window. **New implementations should use the Snowflake Connector for Kafka (v4).**

#### The connector is limited to **loading data into Snowflake only** (one direction). It supports two load mechanisms:
- **Snowpipe**
- **Snowpipe Streaming**

#### Objects created automatically per Kafka topic
For **each topic**, the connector creates:
1. **One internal stage** — temporarily stores data files for that topic
2. **One pipe** — ingests the data files for each topic partition
3. **One table** — if it doesn't already exist, the connector creates it; if it does exist, the connector **adds** `RECORD_CONTENT`/`RECORD_METADATA` columns and verifies other columns are **nullable** (any extra columns must allow NULL, or the connector errors)

#### Table naming rules (classic exam trap)
The connector converts a Kafka topic name into a valid Snowflake table name:
- Lowercase topic names → converted to **UPPERCASE**
- If first character isn't a letter or underscore → an underscore is **prepended**
- Any illegal character inside the name → replaced with an underscore

> ⚠️ **Exam gotcha:** Topics `numbers+x` and `numbers-x` would *both* sanitize to `NUMBERS_X`, causing a **collision**. To prevent silently merging data from two different topics into one table, the connector **appends a generated hash suffix** to disambiguate. Best practice: choose topic names that already follow Snowflake identifier rules.

#### Default table schema
By default (Snowpipe or Snowpipe Streaming), every Kafka-loaded table has exactly **two VARIANT columns**:

| Column | Contents |
|---|---|
| `RECORD_CONTENT` | The actual Kafka message (JSON or Avro), unparsed |
| `RECORD_METADATA` | Metadata about the message |

`RECORD_METADATA` fields include: `topic`, `partition` (Kafka partition, **not** a Snowflake micro-partition — don't confuse these two!), `offset`, `CreateTime`/`LogAppendTime`, `SnowflakeConnectorPushTime` (Snowpipe Streaming only), `key` (only captured if `key.converter` = `StringConverter`), `schema_id` (Avro + schema registry only), and `headers`.

> ⚠️ **Exam gotcha:** "Partition" in `RECORD_METADATA` refers to the **Kafka topic partition**, not a Snowflake micro-partition. These are unrelated concepts that share a name.

> 💡 With **schema detection and evolution** enabled, or when ingesting into an **Iceberg table**, the schema differs from this two-VARIANT-column default (Iceberg uses structured-type columns instead of VARIANT).

#### Ingest workflow (know this flow cold)
1. Apps publish JSON/Avro records → Kafka topic partitions.
2. Connector **buffers** messages until a threshold (time, memory, or message count) is hit, then writes to a temp file in the **internal stage** and triggers **Snowpipe**.
3. A Snowflake-provided **virtual warehouse** (Snowpipe's serverless compute) loads the staged file into the target table via the pipe.
4. Connector **monitors** Snowpipe and deletes the staged file once confirmed loaded; on failure, the file moves to the **table stage** with an error.
5. Repeat.

> ⚠️ **Exam gotcha:** Snowflake polls the ingest status API for **one hour**. If a file's load status isn't confirmed within that hour, it's moved to the table stage — meaning **failed-file visibility can be delayed up to ~1 hour**.

#### Fault tolerance & limitations (heavily tested)
- Data **deduplication** logic in the Snowpipe pipeline removes duplicate copies **except in rare cases**.
- Malformed records (bad JSON/Avro) are **not loaded** — moved to a table stage instead.
- Kafka topics have **retention limits** (default **7 days**). If the connector is offline longer than retention, or storage limits are exceeded, **expired records are lost** and will never load.
- If messages in Kafka are **deleted or updated**, this is **not** reflected in the already-loaded Snowflake table.
- **Instances of the connector do not communicate with each other.** Running multiple connector instances against the **same topic/partition is not recommended** — it can insert **duplicate rows**.
- **No guarantee of insertion order** — rows may not land in the same order they were originally published.
- Kafka connector with **Snowpipe Streaming** supports **dead-letter queues (DLQ)** for error handling.

> ⚠️ **Exam gotcha:** Two of the most-tested "gotcha" facts about the Kafka connector:
> 1. **Row order is not guaranteed.**
> 2. **Running multiple connector instances on the same topic can produce duplicate rows** (Snowflake explicitly recommends one instance per topic).

#### Single Message Transformations (SMT) limitation
- If `key.converter` or `value.converter` is set to one of Snowflake's own converters (`SnowflakeJsonConverter`, `SnowflakeAvroConverter`, `SnowflakeAvroConverterWithoutSchemaRegistry`), **SMTs are NOT supported** on that key/value.
- If neither converter is explicitly set, **most SMTs work**, except `regex.router`.
- Kafka connector 1.4.3+ supports many **community-based converters** (e.g., `io.confluent.connect.avro.AvroConverter`) that do allow SMTs.

#### Billing
- **No direct charge** for the connector itself.
- Indirect costs: **Snowpipe processing time** (charged) + **data storage** (charged).

#### Protobuf
- Kafka connector **1.5.0+** supports Protocol Buffers via a protobuf converter.

---

### 2.2 Spark Connector

#### Core concept
- The **Snowflake Connector for Spark** is not *strictly* required (3rd-party JDBC drivers can work), but Snowflake **recommends** it because — combined with the JDBC driver — it's optimized for large data transfers and supports **query pushdown**.
- **Query pushdown**: translates Spark SQL logical plans (fully or partially) into Snowflake SQL, so the **actual processing happens inside Snowflake** rather than pulling all data into Spark first. Supported since **connector v2.1.0+**.
- Pushdown is **not possible for everything** — e.g., **Spark UDFs cannot be pushed down**.

> ⚠️ **Exam gotcha:** If you need pushdown to cover **all** operations including UDFs, the documentation recommends using the **Snowpark API** instead of the Spark Connector, since Snowpark also supports pushdown of Snowflake UDFs.

#### Transfer modes
| Mode | Description |
|---|---|
| **Internal transfer** | Uses a **temporary location created/managed by Snowflake** internally |
| **External transfer** | Uses a storage location **created and managed by the user** (usually temporary) |

> ⚠️ **Exam gotcha:** If you're on **Spark Connector v2.1.x or lower** (no internal transfer support) and your transfer takes **36+ hours**, it will fail — because **internal transfer uses temporary credentials that expire after 36 hours**. This is a specific, documented edge case.

- **Column mapping** (mapping mismatched Spark ↔ Snowflake column names via the `columnmapping` parameter) is supported **only for internal transfer**, not external.

#### Ecosystem integrations
- **Databricks** has integrated the Spark Connector natively into its Unified Analytics Platform.
- **Qubole** integrated it into the Qubole Data Service (QDS) ecosystem.

> ⚠️ **Exam gotcha (Iceberg-specific):** Snowpark supports pushdown for all operations including Snowflake UDFs, **but** if you need to **enforce row and column policies on Iceberg tables**, you must use the **Snowflake Spark Connector**, not plain Snowpark.

---

## 3. Storage Integration

### 3.1 What it is and why it exists
A **storage integration** is a **named, first-class Snowflake object** that stores a generated **cloud identity and access management (IAM) entity** for your external cloud storage (Amazon S3, Google Cloud Storage, or Microsoft Azure), along with an optional allow/block list of storage locations.

**The core problem it solves:** without it, every external stage would need explicit AWS/GCP/Azure secret keys or tokens embedded in its definition. A storage integration lets you **avoid supplying credentials** entirely when creating stages or loading/unloading data.

- **A single storage integration can support multiple external stages.**
- The stage's URL must fall within the integration's `STORAGE_ALLOWED_LOCATIONS`.

### 3.2 Creating one (AWS example — the most tested)

```sql
CREATE STORAGE INTEGRATION s3_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::001234567890:role/myrole'
  ENABLED = TRUE
  STORAGE_ALLOWED_LOCATIONS = ('s3://mybucket1/path1/', 's3://mybucket2/path2/');
```

Key parameters:
- `STORAGE_AWS_ROLE_ARN` — the ARN of the AWS IAM role granting bucket privileges.
- `STORAGE_AWS_EXTERNAL_ID` — optional; Snowflake **auto-generates** one if you don't specify it. Used to establish trust between Snowflake and AWS (prevents the "confused deputy" problem).
- `STORAGE_ALLOWED_LOCATIONS` / `STORAGE_BLOCKED_LOCATIONS` — restrict/exclude specific bucket paths.
- `STORAGE_AWS_OBJECT_ACL = 'bucket-owner-full-control'` — needed when unloading data to a bucket owned by a **different** AWS account, so that account retains full control of the files.

### 3.3 The trust-relationship workflow (must know cold)
1. **In AWS**: create an IAM policy granting access to the S3 bucket, attach it to an IAM role, with a **placeholder** trusted entity/external ID for now.
2. **In Snowflake**: run `CREATE STORAGE INTEGRATION` referencing that role's ARN.
3. **In Snowflake**: run `DESCRIBE INTEGRATION <name>` to retrieve:
   - `STORAGE_AWS_IAM_USER_ARN` — the actual Snowflake-generated IAM **user** that needs access
   - `STORAGE_AWS_EXTERNAL_ID` — the real external ID to use
4. **Back in AWS**: edit the IAM role's **trust policy**, replacing the placeholders with the real `STORAGE_AWS_IAM_USER_ARN` and `STORAGE_AWS_EXTERNAL_ID`.
5. **In Snowflake**: create an external stage referencing `storage_integration = s3_int`.

> ⚠️ **Exam gotcha — extremely important:** Snowflake creates **ONE single IAM user per Snowflake account**, and **that same IAM user is referenced by ALL S3 storage integrations in that account.** It's the external stage's storage integration + trust policy that scopes access — not a unique IAM user per integration.

> ⚠️ **Exam gotcha:** If you **recreate** a storage integration (`CREATE OR REPLACE STORAGE INTEGRATION`) **without specifying your own external ID**, the new integration gets a **different, auto-generated external ID**. The old AWS trust policy will now be **broken** until you update it with the new external ID. (You can avoid this churn by deliberately setting your own `STORAGE_AWS_EXTERNAL_ID` up front, reusable across multiple integrations.)

> ⚠️ **Exam gotcha:** Snowflake **caches temporary credentials** obtained from the cloud provider for **up to 60 minutes**. This means: if you **revoke** Snowflake's access in AWS, users might *still* be able to list files / load data from that location for **up to 60 minutes** until the cache expires. Access revocation is **not instantaneous**.

### 3.4 Privileges required
- **Only `ACCOUNTADMIN`** or a role with the **global `CREATE INTEGRATION`** privilege can run `CREATE STORAGE INTEGRATION`.
- To create a **stage** that uses the integration, a role needs `CREATE STAGE` on the schema **and** `USAGE` on the storage integration.

### 3.5 Validation
- Use `SYSTEM$VALIDATE_STORAGE_INTEGRATION` to test whether the configuration actually works before/after setup.

### 3.6 Storage Integration vs External Volume (common confusion — different exam objective, but frequently mixed up)
- **Storage Integration** → used for **external stages**, `COPY INTO`, and **external tables**.
- **External Volume** → a *different*, account-level object used specifically for **Apache Iceberg tables**.

> ⚠️ **Exam gotcha:** **You cannot access the cloud storage location referenced by an external volume using a storage integration.** Each external volume requires its **own separate trust relationship** configuration — the two object types are not interchangeable, even though they solve a conceptually similar problem (secure, credential-less cloud storage access).

### 3.7 GCS / Azure variants (same pattern, different parameters)
```sql
CREATE STORAGE INTEGRATION gcs_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'GCS'
  ENABLED = TRUE
  STORAGE_ALLOWED_LOCATIONS = ('gcs://mybucket1/path1/', 'gcs://mybucket2/path2/');
```
- If your cloud storage is on a **different cloud platform** than your Snowflake account, the storage location must be in a **public cloud**, not a virtual private environment.

---

## 4. API Integration

### 4.1 What it is
An **API integration** is a Snowflake database object that stores security/connection information needed to interact with a **proxy service** (an HTTPS gateway like Amazon API Gateway or Azure API Management), primarily used for:
- **External Functions** (call code running outside Snowflake — Lambda, Azure Functions, any HTTPS server)
- **Git Integration** (special `API_PROVIDER = git_https_api`)
- Newer use cases like external **MCP servers**

### 4.2 The request flow — must know this architecture
**Snowflake never calls the remote service directly.** The flow is:

```
Client SQL statement
        │
        ▼
  Snowflake reads external function definition + API integration
        │
        ▼
  Snowflake composes an HTTPS POST → Proxy Service (e.g., API Gateway)
        │
        ▼
  Proxy Service relays the request → Remote Service (e.g., AWS Lambda)
        │
        ▼
  Response relayed back through the Proxy → Snowflake → Client
```

> ⚠️ **Exam gotcha:** The **proxy service** (API Gateway / Azure API Management) sits *between* Snowflake and the actual remote service. The proxy can add authentication, subscription/billing enforcement, and relays responses back. Snowflake talks to the **proxy**, not the Lambda/Function directly.

### 4.3 Creating one (AWS example)
```sql
CREATE OR REPLACE API INTEGRATION demonstration_external_api_integration_01
  API_PROVIDER = aws_api_gateway
  API_AWS_ROLE_ARN = 'arn:aws:iam::123456789012:role/my_cloud_account_role'
  API_ALLOWED_PREFIXES = ('https://xyz.execute-api.us-west-2.amazonaws.com/production')
  ENABLED = TRUE;
```
`API_PROVIDER` accepts: `aws_api_gateway`, `aws_private_api_gateway`, `aws_gov_api_gateway`, `aws_gov_private_api_gateway`, `azure_api_management`, or (for Git) `git_https_api`. (These values are **not** quoted in the statement.)

### 4.4 Key parameters
- **`API_ALLOWED_PREFIXES`** — a comma-separated list of URL **prefixes**. Any external function using this integration can only call URLs that start with one of these prefixes. **Best practice: restrict as narrowly as practical** (least privilege).
- **`API_KEY`** — optional "subscription key" some gateways require. It's **opaque to Snowflake** — Snowflake doesn't validate or interpret it, just passes it along. **API keys are never displayed** in query history, `DESCRIBE INTEGRATION`, or `DESCRIBE API INTEGRATION` output — treated as sensitive.
- **`ENABLED`** — if `FALSE`, any dependent external function **immediately stops working**.

> ⚠️ **Exam gotcha:** An API integration is tied to **one specific cloud platform account/role**, but **NOT** tied to one specific proxy URL. This means:
> - **One API integration** can be reused to authenticate to **multiple different proxy service instances** in that same cloud account.
> - **Multiple external functions** can share the **same** API integration object.

### 4.5 Privileges
- Creating an API integration requires **`ACCOUNTADMIN`** or a role with **`CREATE INTEGRATION`**.
- Only roles with **`OWNERSHIP`** or **`USAGE`** on the API integration can use it (e.g., in a `CREATE EXTERNAL FUNCTION` statement).

### 4.6 Recreating it (same trap as storage integration)
- If you **re-create** an API integration, you must **re-verify/update** the IAM role trust relationship — `API_AWS_IAM_USER_ARN` typically stays the same (tied to your Snowflake account), but the `API_AWS_EXTERNAL_ID` can require re-syncing depending on configuration.

### 4.7 Using it in an external function
```sql
CREATE OR REPLACE EXTERNAL FUNCTION local_echo(string_col VARCHAR)
  RETURNS VARIANT
  API_INTEGRATION = demonstration_external_api_integration_01
  AS 'https://xyz.execute-api.us-west-2.amazonaws.com/production/remote_echo';
```
- An external function is a **type of UDF** — it behaves like any other UDF in SQL (can appear in `SELECT`, `WHERE`, view definitions, expressions), but its code runs **outside** Snowflake.

---

## 5. Git Integration

### 5.1 What it is
Git integration lets you connect a **remote Git repository** (GitHub, GitLab, BitBucket, Azure DevOps, AWS CodeCommit — including **custom URLs** on these platforms) so its files sync into a **Git repository clone** inside Snowflake: a special type of **stage** that mirrors branches, tags, and commits.

> 💡 The clone is a **separate client** of your repo — your local git workflow (VS Code, CLI, etc.) is unaffected. Snowflake just becomes "another clone."

### 5.2 The three building blocks
| Object | Purpose |
|---|---|
| **Secret** (`CREATE SECRET`) | Stores credentials (e.g., username + Personal Access Token) for authenticating to the remote repo |
| **API Integration** (`API_PROVIDER = git_https_api`) | Defines which HTTPS endpoints Snowflake may reach, and which secret(s) are allowed |
| **Git Repository** (`CREATE GIT REPOSITORY`) | The actual clone/stage object inside a Snowflake schema |

### 5.3 Setting it up
```sql
-- 1. Store credentials
CREATE OR REPLACE SECRET my_git_secret
  TYPE = password
  USERNAME = 'my_username'
  PASSWORD = 'my_personal_access_token';

-- 2. Create the API integration (git-specific provider)
CREATE OR REPLACE API INTEGRATION my_git_api_integration
  API_PROVIDER = git_https_api
  API_ALLOWED_PREFIXES = ('https://github.com/my-account')
  ALLOWED_AUTHENTICATION_SECRETS = (my_git_secret)
  ENABLED = TRUE;

-- 3. Create the repository clone
CREATE OR REPLACE GIT REPOSITORY my_git_repo
  API_INTEGRATION = my_git_api_integration
  GIT_CREDENTIALS = my_git_secret
  ORIGIN = 'https://github.com/my-account/my-repo.git';
```

- **`ORIGIN` must use HTTPS.** Snowflake supports any HTTPS Git URL, including corporate/self-hosted Git servers at custom domains.
- `GIT_CREDENTIALS` is **optional** — omit it for public repos or when using a default secret already tied to the API integration.
- As a **best practice**, use a **Personal Access Token (PAT)** as the password value rather than an actual account password.

### 5.4 Authentication method choices
Per official guidance, pick based on the use case:
| Use case | Recommended auth |
|---|---|
| Interactive development (pulling/pushing/creating files in **Workspaces**) | **OAuth2** authentication |
| Automated pipelines / ML projects | **Token-based** (PAT via Secret) — no manual sign-in needed |
| Quick start with a **public** repo | **No authentication** at all |

### 5.5 Working with the repository
- `ALTER GIT REPOSITORY <repo> FETCH` — pulls the latest branches/tags/commits from the remote. **This is a manual, on-demand action — Snowflake does NOT auto-sync in the background.**
- Files are addressed with a special path structure:
  - `@repo_name/branches/<branch_name>/...`
  - `@repo_name/tags/<tag_name>/...`
  - `@repo_name/commits/<commit_hash>/...`
- `EXECUTE IMMEDIATE FROM @repo_name/branches/main/scripts/setup.sql` — runs a `.sql` file **directly from the repo clone**, without manually staging/uploading it first.
- Handler code (for procedures/UDFs) can be **imported directly** from a repository file path.

> ⚠️ **Exam gotcha:** Refreshing/fetching a Git repository clone is **not automatic** — you (or a scheduled Task/pipeline) must explicitly run `ALTER GIT REPOSITORY ... FETCH` to pick up new commits from the remote. Forgetting this is a very common real-world (and exam) trap: *"why isn't my updated code running?"* → because the clone was never fetched.

### 5.6 Read vs. write access
- Historically (initial preview/GA), interaction with a Git repository from Snowflake was **read-only** — you could pull files in, but not commit/push changes back.
- Snowflake has since added the ability to **commit and push** changes back to the remote repository — but **only** from specific surfaces: **Workspaces**, **Streamlit apps**, and **Snowflake Notebooks**.

> ⚠️ **Exam gotcha:** Don't assume "read-only" or "read-write" universally — it depends on **which Snowflake feature** is accessing the repo. Generic access via stored procedures/UDFs importing handler code is still fundamentally about **reading** files; push/commit capability is scoped to the interactive surfaces (Workspaces/Streamlit/Notebooks).

### 5.7 Network & connectivity
- By default, Git access happens over the **public internet**.
- **Private connectivity (PrivateLink)** to a Git server is available for accounts on **Business Critical** edition (or higher / VPS) — lets Snowflake reach a Git server behind a firewall without exposing it publicly.
- Snowflake also supports a pre-configured **Snowflake GitHub App** (OAuth2-based) to simplify GitHub authentication specifically.

### 5.8 Known limitations to remember
- A Git repository used for a **Workspace** must contain **at least one branch** — empty repositories are not supported for that flow.
- **Sharing** repository stages via **data sharing** or **Native Apps** is **not supported**.
- Creating repository stages **inside application packages**, or inside Native Apps on the **consumer** side, is **not supported**.
- All Git operations (fetch, list, execute) consume **virtual warehouse compute** — cost scales with how long each git operation takes.

---

## 🧠 Cross-Topic Comparison (high-yield for exam)

| Feature | Storage Integration | API Integration | Git Integration |
|---|---|---|---|
| What it connects to | Cloud storage (S3/GCS/Azure) | An HTTPS proxy service (API Gateway) | A remote Git repo (via `git_https_api` API integration) |
| Object created | `STORAGE INTEGRATION` | `API INTEGRATION` | `GIT REPOSITORY` (built on top of an API integration + optional secret) |
| Needs a Secret object? | No | Optional (`API_KEY`) | Often yes (`GIT_CREDENTIALS`) |
| Credential model | Snowflake-generated cloud IAM identity | Cloud IAM role / OAuth / API key | Username+PAT secret, OAuth2, or none |
| Who can create | `ACCOUNTADMIN` or role with `CREATE INTEGRATION` | Same | Same (for the underlying API integration) |
| Common gotcha | Recreating without custom external ID breaks trust; 60-min credential cache on revoke | Recreating can break trust; API key hidden from all describe/history output | Must manually `FETCH`; write access only via Workspaces/Streamlit/Notebooks |

---

## ✅ Rapid-Fire Exam Revision Notes

1. **Drivers vs. Connectors**: Python/JDBC/ODBC/.NET/Node.js/Go/PHP-PDO = **Drivers**. Kafka & Spark = **Connectors**.
2. **.NET driver** is the outlier for **TLS 1.3** (macOS gap).
3. Kafka connector table names: illegal chars → underscore; collisions get a **hash suffix**.
4. Kafka: **no row-order guarantee**; **don't run multiple instances on one topic** (duplicates).
5. Kafka default schema = `RECORD_CONTENT` + `RECORD_METADATA`, both **VARIANT**.
6. Kafka `partition` in metadata = **Kafka partition**, not Snowflake micro-partition.
7. Spark Connector v2.1.x and below **lacks internal transfer** → 36-hour temp credential expiry risk on long transfers.
8. Spark UDFs **cannot** be pushed down; use **Snowpark** if you need full UDF pushdown.
9. Storage integration: **one shared IAM user per Snowflake account** for all S3 integrations.
10. Storage integration: recreating without a custom external ID **breaks** the AWS trust policy.
11. Storage integration: revoked access can still work for **up to 60 minutes** (credential caching).
12. Storage Integration ≠ External Volume — **Iceberg tables use External Volume**, and you **cannot** substitute a storage integration for it.
13. API Integration: Snowflake always talks to a **proxy service**, never directly to the remote service.
14. API Integration: one integration can serve **multiple proxies** and **multiple external functions**.
15. API keys are **never** shown in `DESCRIBE INTEGRATION` or query history.
16. Git Integration uses `API_PROVIDER = git_https_api` specifically.
17. Git repository clones require a manual **`ALTER GIT REPOSITORY ... FETCH`** — no auto-sync.
18. Git write-back (commit/push) is only available from **Workspaces, Streamlit, and Notebooks** — not generic stored-procedure access.
19. Only `ACCOUNTADMIN` or a role granted **`CREATE INTEGRATION`** can create any of these three integration types.

---

## 📚 Sources (Snowflake Official Documentation)
- Drivers overview — docs.snowflake.com/en/developer-guide/drivers
- JDBC Driver — docs.snowflake.com/en/developer-guide/jdbc/jdbc
- ODBC Driver — docs.snowflake.com/en/developer-guide/odbc/odbc
- Overview of the Kafka connector — docs.snowflake.com/en/user-guide/kafka-connector/classic/overview
- Overview of the Spark Connector — docs.snowflake.com/en/user-guide/spark-connector-overview
- CREATE STORAGE INTEGRATION — docs.snowflake.com/en/sql-reference/sql/create-storage-integration
- Configuring a Snowflake storage integration for Amazon S3 — docs.snowflake.com/en/user-guide/data-load-s3-config-storage-integration
- CREATE API INTEGRATION — docs.snowflake.com/en/sql-reference/sql/create-api-integration
- Introduction to external functions — docs.snowflake.com/en/sql-reference/external-functions-introduction
- Securing an external function — docs.snowflake.com/en/sql-reference/external-functions-security
- Using a Git repository in Snowflake — docs.snowflake.com/en/developer-guide/git/git-overview
- Setting up Snowflake to use Git — docs.snowflake.com/en/developer-guide/git/git-setting-up
- Connect to a Git repository over a public/private network — docs.snowflake.com/en/developer-guide/git/git-setting-up-public / git-setting-up-private
- CREATE GIT REPOSITORY — docs.snowflake.com/en/sql-reference/sql/create-git-repository
- Storage for Apache Iceberg tables / external volumes — docs.snowflake.com/en/user-guide/tables-iceberg-storage, tables-iceberg-configure-external-volume
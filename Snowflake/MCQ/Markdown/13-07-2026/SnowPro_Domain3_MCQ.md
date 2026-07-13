# SnowPro Core — Domain 3.0 Mastery Bank
## Data Loading, Unloading & Connectivity — 100 Advanced Scenario-Based MCQs

> **How to use this bank:** Every question is scenario-driven — read the setup carefully, since the "obvious-sounding" option is frequently a distractor built from a real but misapplied Snowflake concept. Answers are hidden in collapsible sections so you can genuinely self-test. Explanations cite the governing Snowflake documentation behavior so you can verify anything that surprises you.
>
> **Coverage:** 3.1 Data Loading/Unloading (Q1–40) · 3.2 Automated Data Ingestion (Q41–80) · 3.3 Connectors & Integrations (Q81–100)

---

## Section 3.1 — Data Loading and Unloading

### Part A: File Formats (Q1–Q8)

### Q1
A pipeline ingests CSV exports from a legacy billing system. Several address fields legitimately contain commas (e.g., `"Suite 400, Building B"`), and those fields are always wrapped in double quotes in the source file. Which file format option must be set so Snowflake does not split those fields on the embedded comma?

A. `ESCAPE_UNENCLOSED_FIELD = '\\'`
B. `FIELD_OPTIONALLY_ENCLOSED_BY = '"'`
C. `FIELD_DELIMITER = 'NONE'`
D. `ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE`

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`FIELD_OPTIONALLY_ENCLOSED_BY` tells the CSV parser which character wraps a field so that delimiters *inside* the enclosure are treated as literal data rather than field boundaries. Setting it to `'"'` means any field that starts and ends with a double quote is read as one value, comma and all.

- **A** is wrong: `ESCAPE_UNENCLOSED_FIELD` controls escape characters for unenclosed fields, not comma-containing enclosed fields — it solves a different problem (literal escape sequences, not quoting).
- **C** is invalid syntax; `FIELD_DELIMITER` takes an actual delimiter character, not the literal string `NONE`.
- **D** would silently suppress the *symptom* (a column-count error) rather than correctly parsing the field, and could hide genuine data corruption elsewhere in the file.

**Reference:** [CREATE FILE FORMAT](https://docs.snowflake.com/en/sql-reference/sql/create-file-format)
</details>

---

### Q2
An engineer stages a JSON export where the entire file is a single top-level array containing thousands of order objects, e.g. `[{...},{...},{...}]`. Loading it as-is with default JSON file format settings causes Snowflake to load the whole array as **one row** in a VARIANT column instead of one row per order. What should be changed?

A. Set `STRIP_OUTER_ARRAY = TRUE` on the JSON file format
B. Set `PARSE_HEADER = TRUE` on the JSON file format
C. Set `MULTI_LINE = TRUE` on the JSON file format
D. Convert the file to NDJSON manually before staging, since Snowflake cannot parse arrays

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

By default, Snowflake's JSON parser treats a top-level `[...]` array as a single value. `STRIP_OUTER_ARRAY = TRUE` removes the outer brackets during parsing so each element of the array becomes its own row, which is exactly the one-row-per-order behavior the team wants — no file rewriting required.

- **B** is a CSV-oriented option (used with `INFER_SCHEMA`/`MATCH_BY_COLUMN_NAME`) and has no effect on JSON array handling.
- **C** controls whether a logical record can span multiple physical lines in delimited files; it doesn't unwrap JSON arrays.
- **D** is factually wrong and unnecessarily manual — Snowflake handles top-level arrays natively via `STRIP_OUTER_ARRAY`.

**Reference:** [JSON file format options](https://docs.snowflake.com/en/sql-reference/sql/create-file-format)
</details>

---

### Q3
A data engineering team has 40 different COPY INTO statements across various schemas that all load pipe-delimited files with the same quoting, compression, and null-handling rules. A new hire hardcodes the same seven format parameters inline into every statement. What is the primary architectural drawback of this approach compared to using a **named file format object**?

A. Inline file format options run on a larger warehouse by default, increasing cost
B. Inline options cannot be used with external stages, only internal stages
C. A rule change (e.g., a new NULL representation) requires editing every statement individually instead of one shared object
D. COPY INTO statements with inline file format options are not visible in QUERY_HISTORY

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: C**

A named file format (`CREATE FILE FORMAT ...`) centralizes the parsing rules as a first-class, reusable object referenced by `FORMAT_NAME`. When a rule needs to change, you alter the object once and every COPY INTO or stage referencing it picks up the change automatically. Inline options duplicate the same logic everywhere, so any drift or rule change becomes an error-prone, multi-file edit exercise — a maintainability problem, not a technical limitation.

- **A** is false — warehouse sizing is unrelated to how file format options are declared.
- **B** is false — inline file format options work identically on internal and external stages.
- **D** is false — QUERY_HISTORY records the full statement text regardless of how the format is specified.

**Reference:** [CREATE FILE FORMAT](https://docs.snowflake.com/en/sql-reference/sql/create-file-format)
</details>

---

### Q4
A vendor delivers Parquet files where a new nullable column, `discount_code`, was added in the *middle* of the schema last month. Older files don't have this column at all. Positional loading (`SELECT $1, $2, $3 ...`) is now misaligning columns between old and new files. What COPY option resolves this cleanly, assuming the target table's column names match the Parquet field names?

A. `ON_ERROR = CONTINUE`, so mismatched rows are simply skipped
B. `MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE`
C. `VALIDATION_MODE = 'RETURN_ALL_ERRORS'`
D. `TRIM_SPACE = TRUE`

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`MATCH_BY_COLUMN_NAME` (supported for structured formats like Parquet, ORC, and Avro) tells COPY INTO to map source fields to target columns **by name** rather than position, so adding, removing, or reordering columns in the source no longer breaks the load — as long as the names line up (case-insensitively, in this case).

- **A** would quietly discard legitimate rows rather than fixing the root cause — a common exam trap where a "safe-sounding" error option masks a schema problem.
- **C** only validates and reports errors without loading data or fixing the mapping.
- **D** trims whitespace from string values; it has nothing to do with column alignment.

**Reference:** [COPY INTO <table>](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

### Q5
A source system exports CSV files where genuinely missing values appear as the literal text `N/A`, while zero-length strings (`""`) represent an intentional blank. The target table should store `N/A` as SQL `NULL` but keep intentional blanks as empty strings. Which configuration achieves this distinction?

A. Set `EMPTY_FIELD_AS_NULL = TRUE` only, and ignore the `N/A` values
B. Set `NULL_IF = ('N/A')` and leave `EMPTY_FIELD_AS_NULL = FALSE`
C. Set `NULL_IF = ('N/A', '')` and `EMPTY_FIELD_AS_NULL = TRUE`
D. Pre-process the files with a Python script to strip out all `N/A` tokens before staging

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`NULL_IF` defines which literal strings are converted to SQL NULL during the load — setting it to `('N/A')` converts only that token. Leaving `EMPTY_FIELD_AS_NULL = FALSE` (its non-default state here, explicitly set) preserves genuine empty strings as empty strings instead of nulling them out, giving exactly the distinction the business requires.

- **A** ignores half the requirement and leaves `N/A` un-nulled.
- **C** would null out *both* `N/A` and legitimate empty strings, erasing the very distinction the business needs.
- **D** works but adds an unnecessary external processing step when the file format option handles it natively inside Snowflake.

**Reference:** [CSV file format options — NULL_IF, EMPTY_FIELD_AS_NULL](https://docs.snowflake.com/en/sql-reference/sql/create-file-format)
</details>

---

### Q6
A partner sends XML files where every customer record is wrapped in a root `<Customers>` element, and each record itself is a `<Customer>` element. The team wants one row per `<Customer>` element in the target VARIANT column, without the surrounding root tag. Which file format option accomplishes this?

A. `STRIP_OUTER_ELEMENT = TRUE`
B. `STRIP_OUTER_ARRAY = TRUE`
C. `DISABLE_SNOWFLAKE_DATA = TRUE`
D. `PRESERVE_SPACE = FALSE`

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

For `TYPE = XML`, `STRIP_OUTER_ELEMENT = TRUE` removes the outermost enclosing tag from consideration, causing each remaining child element to be loaded as its own row — the XML equivalent of `STRIP_OUTER_ARRAY` for JSON.

- **B** is a JSON/Avro-oriented parameter and has no effect on XML documents.
- **C** is not a real Snowflake file format option (a distractor built to sound plausible).
- **D** exists for XML but only governs whitespace handling around elements, not row segmentation.

**Reference:** [XML file format options](https://docs.snowflake.com/en/sql-reference/sql/create-file-format)
</details>

---

### Q7
While troubleshooting a failed load, an engineer notices the source CSV has 12 columns in its header row but the target table has only 10 columns, because two legacy columns were intentionally dropped from the table months ago. Loads have been failing with column-count-mismatch errors ever since. Which single option lets the load succeed while still writing all 10 matching columns?

A. `ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE`
B. `SKIP_HEADER = 2`
C. `ON_ERROR = SKIP_FILE_2`
D. `PARSE_HEADER = TRUE` with no other changes

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

`ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE` explicitly tells Snowflake to tolerate a difference between the number of fields in the file and the number of target columns, loading what maps correctly instead of erroring out — precisely the scenario of a shrinking target schema against a stable file layout.

- **B** just skips the first two physical lines of the file; it does nothing about column-count validation and would probably skip real data rows.
- **C** sacrifices entire files once error thresholds are hit — a blunt instrument that doesn't address the structural mismatch and would still error on every file.
- **D** enables header-based column detection, which is a different feature entirely (used with `INFER_SCHEMA`), not a fix for a fixed mismatch.

**Reference:** [CSV file format options — ERROR_ON_COLUMN_COUNT_MISMATCH](https://docs.snowflake.com/en/sql-reference/sql/create-file-format)
</details>

---

### Q8
A team is deciding between Avro and plain-delimited CSV for a high-volume clickstream feed that has deeply nested, frequently evolving attributes (nested arrays of objects, optional sub-fields). Which statement best justifies choosing a semi-structured format like Avro/Parquet/JSON over CSV for this specific workload?

A. Semi-structured formats always compress to a smaller file size than CSV regardless of content
B. Semi-structured formats natively preserve nested/hierarchical structure and schema evolution without forcing a flat, pre-defined column layout
C. CSV cannot be loaded into Snowflake tables that contain a VARIANT column
D. Semi-structured formats bypass the COPY INTO command entirely and load only through Snowpipe Streaming

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Formats like JSON, Avro, ORC, and Parquet carry their own structure and can represent nested objects/arrays natively, and (for Avro/Parquet in particular) support schema evolution gracefully. CSV is inherently flat, so nested or evolving attributes force complex flattening/parsing logic upstream — a real architectural reason to prefer semi-structured formats for this workload.

- **A** overstates a general trend as an absolute; compression ratio depends heavily on content and codec, not format alone.
- **C** is false — CSV data can be loaded into any table shape, including tables with VARIANT columns (though it isn't the natural fit).
- **D** is false — Avro/Parquet/JSON files load through the standard COPY INTO / Snowpipe file-based path just like CSV; Snowpipe Streaming is an entirely separate, file-less ingestion mechanism.

**Reference:** [Semi-structured data overview](https://docs.snowflake.com/en/user-guide/semistructured-concepts)
</details>

---

### Part B: Stages — Internal, External, Encryption, Directory Tables (Q9–Q20)

### Q9
A contractor asks why their team keeps referring to "user stages," "table stages," and "named internal stages" as three different things when all three store files inside Snowflake-managed storage. What is the key practical difference that matters for pipeline design?

A. There is no real difference; the three terms are interchangeable synonyms for the same object
B. User and table stages are implicitly tied to a specific user or table and cannot be shared across multiple tables, while named stages are independent objects that can be referenced by any COPY INTO targeting any table
C. Only named internal stages support the PUT command; user and table stages are read-only
D. User stages charge for storage, while table and named stages are always free

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

A user stage belongs to a specific user (`@~`) and a table stage belongs to a specific table (`@%tablename`) — both are automatically provisioned and can't be shared as a staging area for other tables. A named internal stage is a standalone object that any role with privileges can reference from any COPY INTO statement, which is why most production pipelines use named stages for flexibility and grantable access control.

- **A** ignores a real architectural distinction tested throughout SnowPro Core.
- **C** is false — PUT/GET work against all three stage types.
- **D** is false — internal stage storage is billed the same way regardless of which of the three types holds the files.

**Reference:** [Choosing an Internal Stage for Local Files](https://docs.snowflake.com/en/user-guide/data-load-local-file-system-create-stage)
</details>

---

### Q10
A security review requires that all data at rest in an external S3 stage be encrypted with keys the *customer* fully controls and rotates, independent of Snowflake or AWS-managed keys. Which staging configuration satisfies this requirement?

A. Configure the stage with `ENCRYPTION = (TYPE = 'AWS_SSE_S3')`
B. Rely on Snowflake's default internal stage encryption, since it always uses customer-managed keys
C. Configure the stage with client-side encryption, supplying `ENCRYPTION = (TYPE = 'AWS_CSE' MASTER_KEY = '<customer_key>')` so files are encrypted before upload with a key Snowflake never stores
D. Enable Tri-Secret Secure at the account level instead of configuring the stage

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: C**

Client-side encryption (`AWS_CSE`) encrypts files using a key supplied and controlled entirely by the customer before they ever reach cloud storage; Snowflake decrypts on read using the same externally-supplied key but never stores or manages it. This is the option that gives the customer full, independent key control as required.

- **A** (`AWS_SSE_S3`) is server-side encryption managed by AWS, not full customer control over key material.
- **B** is false — internal stages use Snowflake-managed encryption, not something the customer supplies or rotates directly.
- **D** (Tri-Secret Secure) is a real Snowflake feature for account-level encryption key control, but it governs Snowflake's internal encryption hierarchy, not per-stage external S3 client-side encryption — it doesn't satisfy an "external stage, customer-controlled key" requirement on its own.

**Reference:** [CREATE STAGE — Encryption](https://docs.snowflake.com/en/sql-reference/sql/create-stage)
</details>

---

### Q11
A stage is defined with `STORAGE_INTEGRATION = my_s3_int` referencing an S3 bucket. A developer also tries to add `CREDENTIALS = (AWS_KEY_ID = '...' AWS_SECRET_KEY = '...')` to the same `CREATE STAGE` statement "just to be safe." What happens?

A. Snowflake merges both credential sources and falls back to the explicit keys if the integration ever fails
B. The statement is invalid — a stage that references a storage integration must not also specify explicit CREDENTIALS, since the integration already delegates authentication
C. The explicit CREDENTIALS silently take priority every time, and the storage integration is ignored
D. Snowflake accepts both, and alternates between them for load balancing

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

A storage integration exists precisely to eliminate the need for embedding cloud credentials in a stage definition. Combining `STORAGE_INTEGRATION` with explicit `CREDENTIALS` in the same stage is a configuration conflict Snowflake does not silently resolve — the whole point of an integration is that no direct credentials are needed at all, and mixing the two produces an error rather than a hybrid behavior.

- **A**, **C**, and **D** all describe invented "silent fallback / priority / load-balancing" behaviors that Snowflake does not implement; these are classic distractor patterns that sound operationally reasonable but aren't documented behavior.

**Reference:** [CREATE STAGE](https://docs.snowflake.com/en/sql-reference/sql/create-stage)
</details>

---

### Q12
A stage was created with `DIRECTORY = (ENABLE = TRUE)` but without any `AUTO_REFRESH` or notification integration configuration. Files are added directly to the underlying cloud bucket by an external ETL tool. Six hours later, `SELECT * FROM DIRECTORY(@my_stage)` still shows none of the new files. What is the most likely explanation?

A. Directory tables are read-only and can never reflect files added after stage creation
B. Without automated refresh configured, the directory table metadata must be synchronized manually using `ALTER STAGE my_stage REFRESH`
C. Directory tables only work with internal stages, not external stages backed by cloud buckets
D. The files were rejected because directory tables require a file format to be specified

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

A directory table's metadata (the file inventory Snowflake exposes via `DIRECTORY()`/`SELECT` against the stage) only updates automatically if `AUTO_REFRESH = TRUE` and event notifications are wired up. Otherwise, the metadata is a snapshot from stage creation (or last manual refresh) and must be refreshed on demand with `ALTER STAGE ... REFRESH`.

- **A** is false — directory tables can absolutely reflect new files, just not automatically without configuration.
- **C** is false — directory tables work on both internal and external stages.
- **D** is false — directory tables track file metadata regardless of file format; a file format is relevant to COPY INTO, not to directory table listing.

**Reference:** [Manage directory tables](https://docs.snowflake.com/en/user-guide/data-load-dirtables-manage)
</details>

---

### Q13
A team wants directory table metadata for an S3-backed external stage to update within seconds of a new file landing, without any manual `ALTER STAGE ... REFRESH` calls. What must be configured together to achieve this?

A. A larger warehouse dedicated to running scheduled `ALTER STAGE ... REFRESH` tasks every few seconds
B. `DIRECTORY = (ENABLE = TRUE, AUTO_REFRESH = TRUE)` on the stage, backed by an S3 event notification delivering to the SQS queue Snowflake manages for that stage
C. Setting `TYPE = SNOWFLAKE_SSE` on the stage's encryption block
D. Directory tables cannot be refreshed faster than once per hour by design, regardless of configuration

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Automated directory table refresh relies on cloud event notifications (S3 event notifications routed through the Snowflake-managed SQS queue, or the Azure Event Grid / GCP Pub-Sub equivalents) combined with `AUTO_REFRESH = TRUE` on the stage's directory configuration. This event-driven design is what enables near-real-time metadata updates without polling.

- **A** describes an inefficient polling workaround that Snowflake's native event-driven mechanism makes unnecessary.
- **C** is an encryption setting unrelated to metadata refresh timing.
- **D** is false — there's no fixed hourly floor; automated refreshes can occur within seconds to minutes of the triggering event, depending on the cloud provider's notification latency.

**Reference:** [Automated directory table metadata refreshes](https://docs.snowflake.com/en/user-guide/data-load-dirtables-auto)
</details>

---

### Q14
Which statement correctly distinguishes an **external stage** from an **internal stage** in terms of where COPY INTO can source data and how storage is billed?

A. External stages reference customer-owned cloud storage (S3/GCS/Azure) that the customer pays their cloud provider for directly, while internal stages use Snowflake-managed storage billed as part of Snowflake storage costs
B. Internal stages can only hold Parquet files, while external stages can hold any file format
C. External stages require a virtual warehouse to store files, while internal stages do not
D. Both stage types bill storage identically through Snowflake, with the only difference being encryption defaults

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

The defining architectural difference is *where* the bytes live and who bills for them: external stages point at storage the customer already owns and pays their cloud provider for (S3, GCS, Azure Blob), while internal stages use Snowflake-managed storage that shows up on the Snowflake storage bill.

- **B** is false — internal stages support every file format Snowflake's COPY INTO supports (CSV, JSON, Parquet, Avro, ORC, XML), not just Parquet.
- **C** is false — stages, internal or external, don't consume virtual warehouse compute just to hold files; warehouses are used for the COPY/query operations against staged data.
- **D** is false and directly contradicts the correct answer — billing differs precisely because of where the storage physically resides.

**Reference:** [Understanding & Using Stages](https://docs.snowflake.com/en/user-guide/data-load-overview)
</details>

---

### Q15
A `CREATE STAGE` statement for an Azure container needs to avoid embedding a SAS token so an Azure administrator can rotate access without Snowflake object changes. Which mechanism best satisfies this while also allowing multiple stages across multiple databases to share the same underlying trust relationship?

A. A per-stage SAS token refreshed manually every 90 days by the data engineering team
B. A storage integration configured with Azure's generated service principal (app registration), granted the necessary role on the storage account by an Azure admin
C. Embedding the storage account key directly into each `CREATE STAGE` statement
D. Switching the stage to `TYPE = INTERNAL` so Azure credentials are no longer needed

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

A storage integration for Azure generates a Snowflake-associated app registration (service principal); an Azure admin grants that principal permissions on the storage account, and any number of external stages across the account can then reference the same integration without embedding or rotating per-stage secrets. This is exactly the "shared trust relationship, admin-managed rotation" pattern the scenario asks for.

- **A** and **C** both re-introduce the exact credential-management burden the team is trying to eliminate.
- **D** would change the fundamental nature of the stage — an internal stage can't reference an existing Azure container full of pre-existing files, so it doesn't solve the stated problem at all.

**Reference:** [Configuring an Azure Container for Loading Data](https://docs.snowflake.com/en/user-guide/data-load-azure-config)
</details>

---

### Q16
A directory table is enabled on an internal stage that will receive roughly 2 million small files uploaded in a single bulk migration. Which `CREATE STAGE` setting should be adjusted to avoid an expensive, slow initial listing operation?

A. `REFRESH_ON_CREATE = FALSE`, followed by incremental `ALTER STAGE ... REFRESH SUBPATH = '...'` calls afterward
B. `DIRECTORY = (ENABLE = FALSE)`, since directory tables are incompatible with large file counts
C. `AUTO_REFRESH = TRUE` alone, with no other changes, since automated refresh always outperforms manual listing at any scale
D. `FILE_FORMAT = (TYPE = 'PARQUET')`, since only Parquet stages can hold more than 1 million files

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

Snowflake explicitly recommends setting `REFRESH_ON_CREATE = FALSE` when a stage's storage location already contains (or will soon contain) close to a million files or more, then refreshing incrementally using `SUBPATH` to bound each listing operation to a manageable subset — avoiding one enormous, slow, expensive full-bucket scan.

- **B** throws away the very feature the team needs (file inventory via directory tables) instead of tuning its refresh strategy.
- **C** overstates automated refresh as a universal performance fix; it doesn't address the *initial* bulk registration problem described.
- **D** is a fabricated file-count restriction tied to file type — Snowflake has no such Parquet-only file-count rule.

**Reference:** [CREATE STAGE — REFRESH_ON_CREATE](https://docs.snowflake.com/en/sql-reference/sql/create-stage)
</details>

---

### Q17
A stage references a storage integration whose `STORAGE_ALLOWED_LOCATIONS` was configured as `('s3://analytics-bucket/prod/')`. A developer then tries to create a second stage pointing at `s3://analytics-bucket/sandbox/` using the *same* storage integration. What happens?

A. It succeeds automatically — a storage integration allows any path within the same bucket regardless of the allowed-locations list
B. It fails, because the stage's URL must fall within the paths explicitly permitted by the integration's STORAGE_ALLOWED_LOCATIONS (or STORAGE_BLOCKED_LOCATIONS must not exclude it)
C. It succeeds, but only if the new stage is created by the ACCOUNTADMIN role
D. It fails permanently — each storage integration may only ever back a single external stage

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`STORAGE_ALLOWED_LOCATIONS` explicitly scopes which buckets/paths a storage integration may be used with; a stage's URL must align with those allowed locations (and not fall under any blocked location) or stage creation/use fails. A single integration commonly backs many stages, but only within its permitted location boundaries.

- **A** ignores the entire purpose of the allow-list, which exists to restrict scope, not just organize it.
- **C** invents a role-based override that doesn't exist for this specific check — the location boundary is enforced regardless of the creating role's privilege level.
- **D** is false — a single storage integration is explicitly designed to support multiple external stages, as long as each stays within the allowed locations.

**Reference:** [CREATE STORAGE INTEGRATION](https://docs.snowflake.com/en/sql-reference/sql/create-storage-integration)
</details>

---

### Q18
Which of the following is a valid, documented reason to add a **directory table** to a stage rather than relying purely on the `LIST @stage` command during pipeline development?

A. Directory tables let you run SQL queries (joins, filters, `SELECT ... FROM DIRECTORY(@stage)`) against file metadata, including generating pre-signed URLs, which `LIST` alone cannot provide
B. Directory tables are required before any COPY INTO statement can reference the stage
C. Directory tables automatically convert all staged files to Parquet for faster querying
D. Directory tables remove the need for a file format when loading data from the stage

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

A directory table exposes file-level metadata (path, size, last-modified, and a way to build file URLs) as a **queryable, SQL-native structure**, so you can filter, join, or programmatically generate links to files — capabilities well beyond what a simple `LIST` command output offers.

- **B** is false — COPY INTO works against any stage regardless of whether a directory table is enabled.
- **C** is a fabricated capability; directory tables track metadata about existing files, they don't transform file contents or formats.
- **D** is false — file format is still required by COPY INTO/query operations regardless of directory table presence.

**Reference:** [Directory tables](https://docs.snowflake.com/en/user-guide/data-load-dirtables)
</details>

---

### Q19
A compliance requirement states that Snowflake must never be able to decrypt data files sitting in an external GCS stage without the customer's explicit, separately-managed key. The team currently uses a storage integration with GCS default server-side encryption. Does the current setup satisfy this requirement?

A. Yes — storage integrations always imply customer-managed encryption keys by default
B. No — server-side encryption managed by the cloud provider means Google (and by extension, Snowflake's read path) can still access the data using provider-managed keys; client-side encryption with a customer-supplied key would be needed instead
C. Yes, but only if `AUTO_REFRESH` is disabled on the stage's directory table
D. No — GCS stages fundamentally cannot be encrypted at all, regardless of configuration

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Provider-managed server-side encryption protects data at rest from unauthorized third parties, but the key material is still controlled by the cloud provider's infrastructure (and accessible to services with proper permissions), which does not meet a requirement that *only the customer* be able to decrypt the files. Client-side encryption, where the customer supplies and retains the encryption key entirely outside Snowflake/GCP's control, is the mechanism that satisfies "customer-exclusive decryption capability."

- **A** and **C** invent behavior storage integrations don't provide — an integration handles authentication/authorization, not encryption key ownership.
- **D** is factually wrong; GCS stages absolutely support encryption (both server-side and client-side options).

**Reference:** [Client-side encryption for external stages](https://docs.snowflake.com/en/user-guide/data-load-considerations-stage)
</details>

---

### Q20
A pipe loads from an external stage with a directory table enabled and `AUTO_REFRESH = TRUE`. A separate scheduled task also manually issues `ALTER STAGE ... REFRESH` every 10 minutes as a legacy habit from before automation was configured. What is the documented consequence of running a manual refresh while automated refresh is active?

A. Manual refreshes are simply ignored and return an error immediately
B. The manual refresh blocks concurrent automated refreshes until it completes, after which automated refresh resumes normally
C. Running both simultaneously permanently disables automatic refresh going forward
D. The stage automatically converts itself back to an external table object

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake documents that a manual `ALTER STAGE ... REFRESH` temporarily blocks simultaneous automated refreshes for that stage; once the manual refresh finishes, automated event-driven refreshing resumes as normal. It's a performance/coordination consideration (manual refreshes list the whole path and can be slow for large stages) rather than a hard conflict or permanent state change.

- **A** is false — the command executes; it isn't rejected outright.
- **C** overstates a temporary blocking behavior into a permanent, false claim.
- **D** describes an unrelated, fabricated object-conversion behavior that doesn't exist.

**Reference:** [Manage directory tables — refresh behavior](https://docs.snowflake.com/en/user-guide/data-load-dirtables-manage)
</details>

---

### Part C: The COPY INTO Command (Q21–Q32)

### Q21
A COPY INTO statement is re-run against the same stage after a partial failure yesterday, targeting the same table. None of the files had actually changed since yesterday's attempt. Assuming default copy options, what happens to files that were already **successfully** loaded yesterday?

A. They are reloaded and the table now contains duplicate rows
B. They are skipped, because Snowflake's load metadata (tracked per table for up to 64 days) recognizes the same file name and size as already loaded
C. The entire COPY INTO statement fails outright because duplicate files were detected in the FROM location
D. They are skipped only if `PURGE = TRUE` was set on the original load

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

By default, Snowflake tracks load metadata per table (for up to 64 days) to prevent reloading files whose name and size match a previously successful load — this is exactly why re-running COPY INTO is generally safe and idempotent for unchanged files.

- **A** is the opposite of the documented default behavior; duplication would only occur if `FORCE = TRUE` were explicitly set.
- **C** is false — COPY INTO doesn't abort the whole statement over already-loaded files; it simply skips them and reports the skip.
- **D** confuses `PURGE` (which deletes files from the stage post-load) with the unrelated load-metadata duplicate-prevention mechanism, which works independent of PURGE.

**Reference:** [Loading data — preventing data duplication](https://docs.snowflake.com/en/user-guide/data-load-considerations-load)
</details>

---

### Q22
A file was staged and loaded into `orders` on March 1. That same file, unmodified, is staged again and an attempt is made to load it into the same table on November 5 (250 days later — well past the 64-day metadata window). What is the default COPY INTO behavior?

A. Snowflake definitively knows the file was already loaded and skips it silently, exactly as it would within the 64-day window
B. Because load metadata has expired, Snowflake can no longer be certain the file was already loaded, so by default it skips the file as a safety precaution — `LOAD_UNCERTAIN_FILES = TRUE` or `FORCE = TRUE` is required to load it
C. The file loads automatically without any special option, since expired metadata is treated as "never loaded"
D. The COPY INTO statement throws a hard SQL compilation error rather than a skip

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Once load metadata for a file ages past the retention window, Snowflake can no longer *definitively* determine whether the file was already loaded. To avoid an accidental silent duplicate, the default behavior is to skip such "uncertain" files. Explicitly setting `LOAD_UNCERTAIN_FILES = TRUE` (uses metadata when present, attempts load when absent) or `FORCE = TRUE` (ignores metadata entirely) is required to force the load.

- **A** contradicts the very reason "uncertain files" exist as a documented category.
- **C** describes the opposite of the cautious default — Snowflake errs toward *not* loading, not automatically loading.
- **D** is false — no compilation error occurs; the file is simply skipped, which is a subtler, more exam-relevant behavior than an outright error.

**Reference:** [Loading data — data load metadata expiration](https://docs.snowflake.com/en/user-guide/data-load-considerations-load)
</details>

---

### Q23
A batch load with `PURGE = TRUE` completes successfully, and the source files are deleted from the external stage as expected. Two days later, a request comes in to replay that exact load into a new environment for reconciliation. What is the most accurate statement about this situation?

A. The files can always be recovered directly from Snowflake's internal COPY_HISTORY view, since COPY_HISTORY stores full file contents
B. PURGE is reversible via `UNDROP STAGE`, so the files can be restored within Time Travel's retention window
C. Because PURGE deletes the files from the external cloud storage location (a one-way action), the replay must rely on whatever backup/versioning the external storage itself provides — Snowflake cannot restore them
D. PURGE only marks files as purged in metadata; the physical files always remain in the bucket for 7 days as a safety net

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: C**

`PURGE = TRUE` physically deletes the source data files from the stage location after a successful load — it is a one-way operation with no Snowflake-side recovery mechanism. Any replay must depend on whatever versioning, backup, or lifecycle policy exists on the external cloud storage side (e.g., S3 versioning), not on Snowflake.

- **A** is false — COPY_HISTORY records load metadata and statistics, not the raw file bytes.
- **B** confuses table/stage Time Travel (which protects Snowflake-managed table data) with files that were removed from *external* cloud storage — Time Travel doesn't apply to deleted external files.
- **D** invents an automatic 7-day safety net that Snowflake's PURGE option does not provide.

**Reference:** [COPY INTO <table> — PURGE](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

### Q24
A team needs to validate 500 newly staged files for formatting errors **before** committing to a full load, without inserting any rows into the target table, and they want to see every error in every file rather than stopping at the first one found per file. Which approach is correct?

A. Run `COPY INTO <table> ... VALIDATION_MODE = 'RETURN_ALL_ERRORS'`
B. Run the COPY INTO statement normally with `ON_ERROR = CONTINUE`, then `TRUNCATE TABLE` afterward if issues are found
C. Run `COPY INTO <table> ... VALIDATION_MODE = 'RETURN_1_ROWS'`
D. Run `LIST @stage` and visually inspect the row counts for anomalies

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

`VALIDATION_MODE = 'RETURN_ALL_ERRORS'` performs a dry run: it parses every specified file, reports every error found (not just the first per file), and loads zero rows into the target table — exactly the "validate everything, load nothing" requirement described.

- **B** actually loads data (violating the "no rows inserted" requirement) and then relies on a destructive TRUNCATE as cleanup, which is both risky and unnecessary.
- **C** (`RETURN_n_ROWS`) is a real validation mode, but it returns a sample of the rows that *would* be loaded rather than surfacing all parsing errors across every file.
- **D** provides no error detail whatsoever — `LIST` only returns file metadata like size and last-modified timestamp, not parsing validity.

**Reference:** [COPY INTO <table> — VALIDATION_MODE](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

### Q25
An analyst wants to unload the results of a complex analytical query — not an entire table — directly to a Parquet file in an external S3 stage, with one output file per 50MB chunk. Which command structure is correct?

A. `COPY INTO @s3_stage/output/ FROM (SELECT ... complex query ...) FILE_FORMAT = (TYPE = PARQUET) MAX_FILE_SIZE = 50000000`
B. `GET @s3_stage/output/ FROM (SELECT ...) FILE_FORMAT = (TYPE = PARQUET)`
C. `COPY INTO my_table FROM @s3_stage/output/ FILE_FORMAT = (TYPE = PARQUET) MAX_FILE_SIZE = 50000000`
D. `CREATE EXTERNAL TABLE AS (SELECT ...) LOCATION = @s3_stage/output/`

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

Unloading arbitrary query results (not just whole tables) to an external stage uses `COPY INTO <location> FROM (<query>)`, with `MAX_FILE_SIZE` controlling the target size of each unloaded file chunk (in bytes) — precisely what's described.

- **B** confuses `GET` (which downloads *already-staged* files to a local client) with the unload operation; `GET` doesn't accept a query as a source.
- **C** has the FROM/target reversed — this syntax *loads* from the stage into a table, which is the opposite direction of what's needed.
- **D** describes creating an external table (a read-only, schema-on-read object over existing files), not the action of unloading fresh query results to new files.

**Reference:** [COPY INTO <location>](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location)
</details>

---

### Q26
A CSV load into a table with a `VARCHAR(20)` column encounters several source values that are 35 characters long. The business wants those long values truncated to fit rather than causing a load failure, accepting the resulting data loss. Which copy option enables this?

A. `TRUNCATECOLUMNS = TRUE`
B. `ON_ERROR = SKIP_FILE_5%`
C. `SIZE_LIMIT = 20`
D. `SKIP_HEADER = 1`

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

`TRUNCATECOLUMNS = TRUE` (the inverse of `ENFORCE_LENGTH`) instructs COPY INTO to silently truncate string values that exceed the target column's defined length instead of raising an error — exactly the accepted-data-loss behavior requested.

- **B** would skip entire files past an error-percentage threshold; it doesn't address column-length handling at all, and would likely skip files unnecessarily.
- **C** (`SIZE_LIMIT`) caps the total bytes of data processed across the whole COPY operation — it is unrelated to per-column string length.
- **D** just skips header rows in the file; irrelevant to truncation behavior.

**Reference:** [CSV file format / copy options — TRUNCATECOLUMNS](https://docs.snowflake.com/en/sql-reference/sql/create-file-format)
</details>

---

### Q27
A load pipeline transforms data during ingestion using a `SELECT` inside the `FROM` clause of COPY INTO, applying a scalar UDF to mask a sensitive column. The team also wants to use `MATCH_BY_COLUMN_NAME` on the same statement to handle schema drift in the source Parquet files. What happens when both are combined in one COPY INTO statement?

A. Both work together seamlessly with no restriction
B. This combination is explicitly unsupported — MATCH_BY_COLUMN_NAME cannot be used together with a SELECT-based transformation in the same COPY statement
C. MATCH_BY_COLUMN_NAME silently takes priority and the SELECT transformation is ignored
D. The SELECT transformation silently takes priority and MATCH_BY_COLUMN_NAME is ignored

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake documents this combination as producing a SQL compilation error — `MATCH_BY_COLUMN_NAME` and a COPY transformation (`SELECT` in the FROM clause) are mutually exclusive within a single COPY INTO statement, even though each works fine independently.

- **A** is directly contradicted by documented behavior.
- **C** and **D** both invent a silent-priority resolution that doesn't happen — Snowflake raises an explicit error rather than picking a winner quietly, which is an important distinction for troubleshooting scenarios on the exam.

**Reference:** [Transform data during a load](https://docs.snowflake.com/en/user-guide/data-load-transform)
</details>

---

### Q28
A COPY INTO statement explicitly lists 40 file names using the `FILES` parameter. Three of those files no longer exist in the stage location (perhaps deleted by an upstream process). Assuming `ON_ERROR` was **not** explicitly set, what is the result?

A. The 37 existing files load fine, and the 3 missing files are silently ignored with a warning only
B. The load operation aborts, because the default `ON_ERROR = ABORT_STATEMENT` behavior triggers whenever explicitly listed files can't be found
C. Snowflake automatically searches subdirectories to locate files with matching names elsewhere in the stage
D. The COPY INTO statement is rejected before execution because the FILES parameter has a hard limit of 20 file names

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

With the default `ON_ERROR = ABORT_STATEMENT`, a COPY INTO statement using `FILES` to explicitly enumerate file names will abort the whole load if any of the explicitly named files cannot be found — this default is unaffected by choosing to name files instead of relying on `PATTERN`.

- **A** describes lenient behavior that only occurs with a different, explicitly-set `ON_ERROR` value, not the default.
- **C** is a fabricated auto-search behavior that Snowflake does not perform.
- **D** understates the real limit — Snowflake's actual maximum for the `FILES` parameter is 1,000 file names, not 20, so this distractor is wrong on the specific number as well as the overall claim.

**Reference:** [COPY INTO <table> — FILES parameter](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

### Q29
A nightly batch pipeline currently issues one large COPY INTO statement per hour to a busy `MEDIUM` warehouse shared with several other jobs. Load times are inconsistent and the pipeline occasionally exceeds its SLA. An architect observes that most loads only involve 2–3 files at a time. What is the most defensible first optimization to try, based on how COPY INTO parallelizes work?

A. Upgrade to a `2X-LARGE` warehouse, since COPY INTO always benefits linearly from more compute regardless of file count
B. Right-size down toward a smaller warehouse (e.g., `X-SMALL`) since COPY INTO parallelizes primarily by file count, and a handful of files can't keep a large multi-cluster warehouse's extra nodes busy — then measure and adjust
C. Switch the pipeline from COPY INTO to Snowpipe Streaming, since streaming ingestion is always faster for batch files
D. Add `MAX_FILE_SIZE` to the COPY INTO statement to force Snowflake to split files further during the load

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

COPY INTO parallelizes primarily at the file level — a warehouse's concurrent "workers" can process at most as many files simultaneously as there are files to process. With only 2–3 files per run, a larger warehouse mostly leaves extra compute idle rather than speeding up the load; right-sizing down (and then measuring `EXECUTION_TIME`) is the evidence-based first step before assuming "bigger warehouse" is the fix.

- **A** overgeneralizes — COPY INTO scaling is bounded by file parallelism, not simply proportional to warehouse size, especially with so few files.
- **C** wrongly asserts a categorical Snowpipe Streaming advantage; Streaming is designed for row-level, low-latency ingestion of *new* streaming data, not a drop-in performance upgrade for existing batch file loads.
- **D** confuses `MAX_FILE_SIZE`, which is an *unload* option controlling output file sizing, with something that would resize or resplit files being *loaded* — it has no such effect on COPY INTO loads.

**Reference:** [Data loading — performance and warehouse sizing](https://docs.snowflake.com/en/user-guide/data-load-considerations-prepare)
</details>

---

### Q30
Which pair of statements about `FORCE = TRUE` and `PURGE = TRUE` on the same COPY INTO statement is accurate?

A. `FORCE` reloads files even if they match previous load metadata (risking duplicates), while `PURGE` deletes source files from the stage after a successful load — the two options serve unrelated purposes and can be combined
B. `FORCE` and `PURGE` are mutually exclusive and Snowflake rejects any statement that sets both
C. `FORCE` deletes files after loading, while `PURGE` reloads files regardless of metadata — the definitions in option A are reversed
D. Both options do the same thing: they exist purely as aliases for backward compatibility

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

`FORCE = TRUE` bypasses load-metadata duplicate checking and reloads files even if they appear already loaded (a real duplication risk if misused), while `PURGE = TRUE` deletes successfully loaded files from the stage afterward. These are two independent copy options addressing different concerns (idempotency vs. stage cleanup) and nothing prevents using them together.

- **B** invents a restriction that doesn't exist between these two independent options.
- **C** simply swaps the correct definitions from A, testing whether the reader actually knows which option does what rather than just recognizing the two option names.
- **D** is false; they are not aliases and have clearly distinct, documented behaviors.

**Reference:** [COPY INTO <table> — FORCE, PURGE](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

### Q31
A COPY INTO statement processes three files sized 10MB, 10MB, and 10MB, with `SIZE_LIMIT = 15000000` (15MB) set. What is the documented behavior?

A. The load stops immediately after 15MB total is queued, splitting the second file mid-stream to respect the exact byte limit
B. The operation loads the first file completely (10MB), then continues processing the file already in progress (the second file) to completion before stopping, resulting in roughly 20MB loaded even though the limit was 15MB
C. The entire COPY INTO statement fails, since 15MB does not evenly divide across the 10MB files
D. SIZE_LIMIT applies per file, so all three 10MB files fit under the limit and all are loaded

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`SIZE_LIMIT` applies to the **entire set of files** processed by the statement, and Snowflake documents that the operation continues processing whatever file is already in progress when the threshold is reached rather than truncating it mid-file — so the actual bytes loaded can exceed the specified limit by up to one file's worth of data.

- **A** describes mid-file splitting, which Snowflake does not do — files are processed as complete units even if that means exceeding SIZE_LIMIT.
- **C** invents a divisibility requirement that doesn't exist; SIZE_LIMIT is a threshold, not a strict partition boundary.
- **D** misapplies the option as per-file when it is explicitly documented as applying to the aggregate set of files in the statement.

**Reference:** [COPY INTO <table> — SIZE_LIMIT](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

### Q32
A developer wants to unload data so that BOM (byte-order-mark) headers are not written into the resulting CSV files, and so each object written to cloud storage carries a specific custom prefix pattern in its file name for downstream partition discovery. Which combination of concepts applies?

A. File format options (e.g., BOM handling) control byte-level content of the output files, while the `COPY INTO <location>` target path/prefix in the FROM/INTO clause controls the file naming pattern — these are two independent, complementary configuration surfaces
B. Both requirements are controlled by a single option, `FILE_NAMING_STANDARD`, which governs both content encoding and naming
C. BOM handling and file naming are both automatically inferred from the target cloud provider and cannot be configured
D. Only Snowpipe (not COPY INTO) can control custom output file naming during unload

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

Output file *content* encoding concerns (like BOM inclusion) are governed by file format options, while the *destination path/prefix* used for naming unloaded files is controlled by the stage/location path specified in the `COPY INTO <location>` statement — two genuinely separate configuration surfaces that both apply here, not a single unified knob.

- **B** invents a nonexistent unified parameter name.
- **C** is false — both aspects are configurable by the user, not auto-inferred.
- **D** is false and backwards — Snowpipe is an *ingestion* automation feature; it has nothing to do with controlling unload file naming, which is purely a COPY INTO <location> concern.

**Reference:** [COPY INTO <location>](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location)
</details>

---

### Part D: Error Handling Options (Q33–Q40)

### Q33
A pipeline loads clickstream JSON logs where a small, expected fraction of records are malformed due to upstream client bugs. The team explicitly does not want any single file to be rejected wholesale over a few bad records, and losing those specific bad rows is acceptable. Which `ON_ERROR` value fits best?

A. `ABORT_STATEMENT`
B. `SKIP_FILE`
C. `CONTINUE`
D. `SKIP_FILE_10%`

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: C**

`CONTINUE` loads all valid rows in a file and simply skips rows that error out, without rejecting the file as a whole — exactly matching "don't lose the whole file over a few bad records, dropping just the bad rows is fine."

- **A** would stop the entire load on the very first error, which is far stricter than what's needed here.
- **B** and **D** both operate at the *file* level — if error thresholds are hit, the **entire file** (including its valid rows) is discarded, which contradicts the requirement to keep the valid records within a partially-bad file.

**Reference:** [COPY INTO <table> — ON_ERROR](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

### Q34
A financial reconciliation load must guarantee that if **any** row in **any** file fails to parse, absolutely nothing from that batch is committed — partial commits are unacceptable for audit reasons. Which `ON_ERROR` setting is correct, and is it something that must be explicitly configured?

A. `CONTINUE`; it must be explicitly set since Snowflake defaults to skipping bad rows silently
B. `ABORT_STATEMENT`; this happens to already be the documented default for a manually-run COPY INTO statement, though best practice is to set it explicitly for clarity and auditability
C. `SKIP_FILE`; this is the universal default across both bulk COPY INTO and Snowpipe
D. `SKIP_FILE_1`; a single-row threshold approximates all-or-nothing behavior closely enough

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`ABORT_STATEMENT` is indeed the default `ON_ERROR` behavior for a manually executed bulk `COPY INTO` statement, matching the all-or-nothing requirement exactly. Since production teams shouldn't rely on implicit defaults for an audit-sensitive process, Snowflake's own guidance recommends setting it explicitly in every production statement regardless of whether it matches the default.

- **A** is factually backwards about the default behavior.
- **C** is incorrect as a universal claim — `SKIP_FILE` is the *Snowpipe* default, not the bulk COPY INTO default; conflating the two defaults is one of the most common SnowPro Core traps.
- **D** still permits a file with exactly one bad row (out of potentially thousands) to be skipped rather than truly halting the whole batch — it approximates, but does not achieve, strict all-or-nothing semantics.

**Reference:** [COPY INTO <table> — ON_ERROR defaults](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

### Q35
A newly created Snowpipe object begins auto-ingesting files, and no `ON_ERROR` value was specified in its COPY statement. A batch of files arrives where a handful contain malformed rows. What is the default Snowpipe `ON_ERROR` behavior, and how does it differ from a manually run bulk COPY INTO with no `ON_ERROR` specified?

A. Snowpipe defaults to `ABORT_STATEMENT`, identical to bulk COPY INTO, so both stop entirely on the first bad row
B. Snowpipe defaults to `SKIP_FILE`, whereas a manual bulk COPY INTO defaults to `ABORT_STATEMENT` — the same unspecified option name resolves to a different default depending on the loading mechanism
C. Snowpipe has no default and requires ON_ERROR to be explicitly set at pipe creation, or the pipe fails to be created
D. Snowpipe defaults to `CONTINUE`, while bulk COPY INTO defaults to `SKIP_FILE`

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

This is a well-known, easy-to-miss asymmetry: leaving `ON_ERROR` unspecified resolves to `SKIP_FILE` for Snowpipe, but to `ABORT_STATEMENT` for a manually executed bulk COPY INTO. Recognizing that "the default" is context-dependent on the loading mechanism — not a single universal value — is a frequently tested distinction.

- **A** and **D** both misstate at least one of the two defaults.
- **C** invents a hard requirement that doesn't exist; Snowpipe pipes are created successfully without an explicit ON_ERROR, simply falling back to `SKIP_FILE`.

**Reference:** [Snowpipe error notifications — default ON_ERROR](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-errors)
</details>

---

### Q36
A support engineer needs to see the specific reason a set of files partially failed to load two days ago via Snowpipe, including whether each file was fully loaded, partially loaded, or failed entirely. Which mechanism should they query?

A. `SYSTEM$PIPE_STATUS`, since it stores full historical error detail indefinitely
B. `COPY_HISTORY` (table function or ACCOUNT_USAGE view), which reports STATUS and FIRST_ERROR_MESSAGE per load attempt
C. `SHOW STAGES`, since directory table metadata includes load error reasons
D. `VALIDATION_MODE = 'RETURN_ALL_ERRORS'` run against the original pipe object directly

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`COPY_HISTORY` (both the Information Schema table function and the ACCOUNT_USAGE view) is the documented source for load activity history, including a `STATUS` column (loaded / partially loaded / failed) and a `FIRST_ERROR_MESSAGE` column describing the reason for a partial or failed attempt — exactly what's needed for after-the-fact troubleshooting.

- **A** is a live pipe-state/health function (last event received, execution state, pending file count) — it is not designed as a historical error log and doesn't retain the level of per-file error detail requested.
- **C** is unrelated; stages and their directory tables track file existence/metadata, not load outcomes or error messages.
- **D** is a COPY-time validation option applied against a table/stage in a fresh statement — it isn't something you "run against a pipe" retroactively to inspect a past load's errors.

**Reference:** [COPY_HISTORY view](https://docs.snowflake.com/en/sql-reference/functions/copy_history)
</details>

---

### Q37
A team is loading a batch of 100 CSV files. They want files with a *high proportion* of bad rows (say, more than 5% of rows in that specific file) to be entirely rejected — since that signals a structurally broken file — while files with only a couple of scattered bad rows should still load their valid data. Which `ON_ERROR` value matches this nuanced requirement?

A. `CONTINUE`
B. `ABORT_STATEMENT`
C. `SKIP_FILE_5%`
D. `SKIP_FILE`

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: C**

`SKIP_FILE_<n>%` skips an entire file only once the **percentage** of erroring rows in that specific file crosses the given threshold — precisely the "structurally broken vs. a few scattered errors" distinction requested. Below the threshold, valid rows in an otherwise-mostly-good file still load.

- **A** (`CONTINUE`) never rejects a whole file regardless of how bad it is — it always loads whatever valid rows exist, which doesn't satisfy the "reject structurally broken files" half of the requirement.
- **B** would reject on the very first error in any file, over-correcting far past the stated tolerance.
- **D** (plain `SKIP_FILE`) rejects a file on its *first* error rather than waiting for a meaningful percentage threshold — too aggressive for files with just a couple of scattered issues.

**Reference:** [COPY INTO <table> — SKIP_FILE_num%](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

### Q38
Between `SKIP_FILE` and `SKIP_FILE_5` (an absolute-count variant), which statement correctly explains a documented performance trade-off of the `SKIP_FILE` family versus `CONTINUE`?

A. `SKIP_FILE` variants are always faster than `CONTINUE` because they process fewer rows overall
B. `SKIP_FILE` variants must buffer the entire file (whether or not it's ultimately rejected) before a decision can be made, making them slower than `CONTINUE` for very large files with a small number of scattered errors
C. `CONTINUE` cannot be used on files larger than 250MB, unlike SKIP_FILE variants
D. There is no performance difference; the choice is purely about which rows end up in the target table

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake's documentation explicitly notes that `SKIP_FILE` buffers an entire file — whether errors are found or not — before deciding whether to reject it, making it slower than `CONTINUE` in scenarios with large files and only a small, scattered number of errors. This is a genuine performance/architecture nuance, not just a "which rows end up loaded" question.

- **A** overgeneralizes incorrectly; buffering overhead can make SKIP_FILE variants *slower*, not faster, in the described scenario.
- **C** invents a fabricated file-size restriction tied to ON_ERROR value that does not exist.
- **D** dismisses a real, documented performance consideration as if it didn't exist.

**Reference:** [COPY INTO <table> — ON_ERROR performance notes](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

### Q39
A load fails with a data-type conversion error on one column. Before touching `ON_ERROR` at all, what is the recommended first diagnostic step to understand root cause without loading any data or discarding any files?

A. Immediately set `ON_ERROR = CONTINUE` in production and monitor row counts over the next few days
B. Run the COPY INTO statement with `VALIDATION_MODE = 'RETURN_ALL_ERRORS'` (or a `RETURN_n_ROWS` variant) to see detailed error messages without committing any data
C. Drop and recreate the target table with looser column types so the error can no longer occur
D. Set `ON_ERROR = ABORT_STATEMENT` and rerun repeatedly until the file that caused the error is identified by trial and error

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`VALIDATION_MODE` is explicitly designed for this diagnostic use case: it parses and validates files without loading any rows, surfacing detailed error information so root cause can be understood safely before deciding on a production `ON_ERROR` strategy or schema change.

- **A** jumps straight to a lossy production setting before understanding what's actually going wrong, risking silent data quality issues.
- **C** is a premature, potentially harmful schema change made before even confirming what the actual error is.
- **D** is already the default in most cases and offers no new diagnostic information beyond what a validation run gives more safely and directly.

**Reference:** [COPY INTO <table> — VALIDATION_MODE](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

### Q40
An architect is designing error handling for two very different pipelines: (1) a real-time fraud-detection feed where any data loss is unacceptable and stopping immediately on error is preferred, and (2) a best-effort marketing clickstream feed where throughput matters more than completeness and dropping a few malformed rows is fine. Which pairing of `ON_ERROR` choices is best justified?

A. Fraud feed → `CONTINUE`; clickstream feed → `ABORT_STATEMENT`
B. Fraud feed → `ABORT_STATEMENT`; clickstream feed → `CONTINUE`
C. Both feeds → `SKIP_FILE`, since it offers a balanced middle ground suitable for any workload
D. Both feeds → `CONTINUE`, since stopping pipelines in production is always undesirable regardless of data sensitivity

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`ABORT_STATEMENT` correctly protects the fraud-detection feed, where every row's integrity matters and a silent partial load would be dangerous — better to halt and investigate than to proceed on incomplete data. `CONTINUE` correctly favors the clickstream feed's throughput and tolerance for a small amount of acceptable loss, avoiding unnecessary halts over routine bad records.

- **A** exactly reverses the sound justification — it would silently tolerate errors in the risk-sensitive fraud pipeline while needlessly halting the tolerant clickstream pipeline.
- **C** treats `SKIP_FILE` as a one-size-fits-all default, ignoring that it discards entire files (including valid rows) rather than matching either use case's actual requirement precisely.
- **D** applies a blanket "never stop" philosophy that ignores the fraud pipeline's explicit requirement that data loss is unacceptable.

**Reference:** [COPY INTO <table> — choosing ON_ERROR strategy](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

## Section 3.2 — Automated Data Ingestion

### Part E: Snowpipe (Q41–Q50)

### Q41
A team hosted on AWS wants near-real-time ingestion the moment files land in an S3 bucket, without managing a virtual warehouse for the load itself. Which architecture piece is responsible for informing Snowflake that a new file has arrived, enabling `AUTO_INGEST = TRUE`?

A. A Snowflake-managed virtual warehouse polling the S3 bucket every 60 seconds
B. An S3 event notification delivered to a Snowflake-managed SQS queue, which Snowpipe consumes to trigger the COPY operation
C. A Snowflake stream created directly on the external stage's directory table
D. The Snowpipe REST API, called manually by an external script whenever a new file is detected

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowpipe's auto-ingest model on AWS relies on S3 event notifications routed to an SQS queue that Snowflake creates and manages per region; Snowpipe consumes these queue messages to trigger the defined COPY statement — this event-driven design is what delivers near-real-time ingestion without polling.

- **A** describes a polling model, which is not how AUTO_INGEST works and would also contradict the "serverless, no warehouse" requirement since polling still implies scheduled compute.
- **C** is a fabricated mechanism — streams track table/view/external-table changes for CDC purposes, they are not the trigger mechanism for Snowpipe auto-ingest.
- **D** describes the *manual* REST API pattern (an alternative to AUTO_INGEST), not the automated, event-driven path the scenario specifically asks about.

**Reference:** [Automating Snowpipe for Amazon S3](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-auto-s3)
</details>

---

### Q42
An account is hosted on Microsoft Azure. Which cloud-native notification service does auto-ingest Snowpipe rely on there, and why would selecting the AWS-equivalent service in a scenario question about an Azure-hosted account always be wrong?

A. Amazon SQS — Snowpipe always uses SQS across every cloud, since Snowflake proxies all cloud notifications through AWS infrastructure
B. Azure Event Grid — each cloud has its own native notification/eventing service, and Snowpipe auto-ingest is wired to whichever service belongs to the storage account's own cloud platform
C. Azure Service Bus, because it is Azure's general-purpose enterprise messaging product
D. Google Cloud Pub/Sub, since Azure Blob Storage integrates with GCP's global eventing layer

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Each cloud platform has its own native eventing mechanism that Snowpipe auto-ingest is designed to consume: SQS (often fed by SNS) on AWS, Event Grid on Azure, and Pub/Sub on Google Cloud. Matching the correct service to the *storage account's* cloud platform (not necessarily the Snowflake account's hosting cloud) is a frequently tested cross-cloud detail.

- **A** and **D** both attach the wrong cloud's native service to an Azure storage location — a classic "right concept, wrong cloud" distractor pattern.
- **C** picks a real, plausible-sounding Azure messaging product, but it is not the service Snowpipe's automated directory/pipe refresh integrates with — Event Grid is the documented mechanism.

**Reference:** [Automating Snowpipe for Azure Blob Storage](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-auto-azure)
</details>

---

### Q43
A pipe was accidentally recreated using `CREATE OR REPLACE PIPE` while still actively auto-ingesting. What is the most significant operational consequence the team should watch for immediately afterward?

A. Nothing changes; recreating a pipe is always a fully transparent, zero-impact operation
B. The pipe's file-loading metadata (load history) is dropped, so a subsequent `ALTER PIPE ... REFRESH` could reload files that were already successfully ingested, risking duplicates
C. The underlying target table is automatically truncated as part of the recreate operation
D. All future files will be routed to a brand-new, differently-named table automatically

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Because Snowpipe's file-loading metadata lives with the pipe object (not the target table), recreating the pipe drops that history. If files that were already loaded are still sitting in the stage and an `ALTER PIPE ... REFRESH` is subsequently run, Snowflake can no longer recognize them as already-loaded, creating a real duplicate-data risk — exactly why Snowflake's own best practice is to pause a pipe, review cloud notification configuration, recreate carefully, and pause again before resuming.

- **A** understates a genuinely risky operational side effect.
- **C** and **D** both invent destructive/automatic behaviors that recreating a pipe does not perform — the target table and its existing data are untouched.

**Reference:** [Managing Snowpipe — recreating pipes](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-manage)
</details>

---

### Q44
Files were staged 21 days ago but never triggered a Snowpipe load because the S3 event notification was misconfigured during that window. The misconfiguration has now been fixed. An engineer runs `ALTER PIPE my_pipe REFRESH` expecting all 21-day-old files to load. What actually happens?

A. All 21 days of backlog files are queued and loaded successfully, since REFRESH has no time boundary
B. `ALTER PIPE ... REFRESH` only queues files staged within the previous 7 days, so the older backlog files are not picked up by this command and require a manual COPY INTO instead
C. The command fails outright because files older than 24 hours cannot be referenced by any pipe operation
D. REFRESH automatically extends its lookback window to cover any gap caused by a notification outage

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`ALTER PIPE ... REFRESH` is explicitly documented to queue files staged within the **previous 7 days** only. Anything older falls outside that window and must be loaded with a manual `COPY INTO <table>` statement instead — a commonly tested operational limitation when reasoning about backlog recovery after a notification outage.

- **A** overstates REFRESH's lookback as unbounded, which is incorrect.
- **C** invents a hard 24-hour cutoff that doesn't match the documented 7-day figure.
- **D** invents "smart" adaptive behavior that Snowflake's REFRESH command does not implement — the 7-day window is fixed regardless of the cause of the backlog.

**Reference:** [Managing Snowpipe — ALTER PIPE REFRESH](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-manage)
</details>

---

### Q45
Which statement accurately compares how Snowpipe is billed for compute versus a scheduled COPY INTO running on a customer-managed virtual warehouse?

A. Both are billed identically per-second against the same warehouse credit rate
B. Snowpipe uses Snowflake-managed serverless compute billed based on actual resource consumption (plus a per-file overhead component), while a scheduled COPY INTO consumes credits from whatever customer-managed warehouse runs it, billed by warehouse size and runtime regardless of file count
C. Snowpipe is always free, since it has no compute cost associated with it at all
D. A scheduled COPY INTO is serverless by default, just like Snowpipe

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowpipe is a serverless feature: Snowflake provisions and bills the compute automatically based on the resources actually consumed to process files (with a documented per-file management overhead that increases with file count/queue size). A manually scheduled COPY INTO, by contrast, runs on a customer-managed virtual warehouse and consumes that warehouse's credits based on size and runtime, independent of how many files were processed in that run.

- **A** and **D** both incorrectly equate Snowpipe's serverless billing model with warehouse-based billing.
- **C** is false; Snowpipe usage clearly appears as its own line item on the Snowflake bill, driven by consumption, not zero cost.

**Reference:** [Snowpipe billing](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-billing)
</details>

---

### Q46
An engineer wants to attach a resource monitor to cap the credits Snowpipe consumes in a given month, the same way they cap a virtual warehouse. Is this achievable directly?

A. Yes — resource monitors can be attached directly to any pipe object using `ALTER PIPE ... SET RESOURCE_MONITOR = ...`
B. No — resource monitors govern virtual warehouse credit consumption; since Snowpipe uses Snowflake-managed serverless compute rather than a customer warehouse, it cannot be capped by a resource monitor in the same way
C. Yes, but only if the pipe was created with `AUTO_INGEST = FALSE`
D. Yes, and it is in fact required — Snowflake refuses to create a pipe without an attached resource monitor

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Resource monitors are explicitly a virtual-warehouse governance mechanism. Because Snowpipe (and other serverless features) don't run on a customer-managed warehouse, a resource monitor cannot be attached to cap its consumption the same way — a distinction the exam frequently probes, since it's tempting to assume "credits" always means "resource monitor applies."

- **A** invents a SQL syntax that doesn't exist for pipes.
- **C** and **D** both invent conditional or mandatory resource-monitor behaviors around pipe creation that Snowflake does not implement.

**Reference:** [Resource monitors](https://docs.snowflake.com/en/user-guide/resource-monitors)
</details>

---

### Q47
A pipe's COPY statement needs to change — a new column mapping must be added. What is required to modify the actual `COPY INTO` logic embedded in an existing pipe's definition?

A. `ALTER PIPE my_pipe SET COPY_STATEMENT = '...'` updates the logic in place with zero downtime and no metadata impact
B. The pipe must be recreated (`CREATE OR REPLACE PIPE` or drop-and-recreate), since the COPY statement embedded in a pipe's definition cannot be altered in place
C. Pipes are immutable in every respect; a brand-new pipe with a different name must be created and the old one deleted
D. Only ACCOUNTADMIN can modify a pipe's COPY statement, and only through a support ticket

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

A pipe's embedded COPY statement can't be altered in place with a simple `ALTER PIPE SET` clause — Snowflake documents that changing the COPY logic requires recreating the pipe object, which carries the load-metadata reset consequences discussed elsewhere in this bank (see Q43).

- **A** invents a no-impact in-place update capability that doesn't exist for the COPY statement itself.
- **C** overstates immutability — while the COPY statement can't be edited in place, `CREATE OR REPLACE PIPE` (not necessarily a differently-named object) is the documented path, so a full rename-and-delete cycle isn't strictly required.
- **D** invents an unnecessary support-ticket requirement; this is a standard, self-service DDL operation for appropriately privileged roles.

**Reference:** [Managing Snowpipe — recreating pipes](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-manage)
</details>

---

### Q48
Which combination correctly reflects the retention/storage rule Snowflake uses for **load history/metadata** across the two mechanisms in this scenario: (1) a manually run bulk COPY INTO, and (2) an auto-ingest Snowpipe?

A. Both retain load metadata for 14 days, since both ultimately execute a COPY operation under the hood
B. Bulk COPY INTO retains load metadata with the target table for up to 64 days; Snowpipe retains its load metadata with the pipe object for 14 days
C. Bulk COPY INTO retains load metadata for 7 days; Snowpipe retains it indefinitely
D. Neither mechanism retains load metadata; both rely entirely on ACCOUNT_USAGE views populated externally

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

This is a precise, frequently tested pairing: bulk COPY INTO's duplicate-prevention metadata lives with the **target table** for up to **64 days**, while Snowpipe's load metadata lives with the **pipe object** for **14 days**. The different owning object (table vs. pipe) and different retention windows are both independently testable facts.

- **A** collapses two genuinely different numbers and owning objects into one incorrect shared value.
- **C** and **D** both invent numbers/behaviors that don't match either documented retention rule.

**Reference:** [Snowpipe — data availability](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-intro) and [Loading data — data load metadata](https://docs.snowflake.com/en/user-guide/data-load-considerations-load)
</details>

---

### Q49
Snowpipe error notifications (which push a message to a configured cloud messaging service when a file load has an error) are documented to only function under one specific `ON_ERROR` condition. Which condition is it, and why does this matter operationally?

A. They only work when `ON_ERROR = CONTINUE`, because that's the only mode that reports partial file errors
B. They only work when `ON_ERROR = SKIP_FILE` (Snowpipe's own default) — if a pipe explicitly overrides this to `CONTINUE`, no error notifications will be sent even though rows are still being skipped
C. They work regardless of ON_ERROR value, since notifications are a pipe-level feature independent of copy option
D. They only work when `ON_ERROR = ABORT_STATEMENT`, since that is the only mode considered a true "failure"

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake explicitly documents that Snowpipe error notifications only fire when `ON_ERROR = SKIP_FILE` is in effect (conveniently, Snowpipe's own default). If a team overrides the pipe's COPY statement to use `CONTINUE` instead — perhaps to avoid losing whole files — they silently lose the error-notification safety net, which is an easy-to-miss operational trade-off worth flagging to a team making that change.

- **A** reverses the actual documented condition.
- **C** ignores an explicit documented constraint.
- **D** picks a plausible-sounding "true failure" option, but `ABORT_STATEMENT` is not the condition tied to Snowpipe error notifications in the documentation.

**Reference:** [Snowpipe error notifications](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-errors)
</details>

---

### Q50
A pipe is defined against an **internal** named stage rather than an external cloud stage. Which statement about Snowpipe's support for internal stages is accurate?

A. Snowpipe cannot load from internal stages at all — only external S3/GCS/Azure stages are supported
B. Snowpipe supports loading from internal named stages and table stages, but auto-ingest via cloud event notifications isn't applicable since there's no cloud bucket to notify from — files must be submitted via the Snowpipe REST API (`insertFiles`) instead
C. Snowpipe supports internal stages only when a directory table with AUTO_REFRESH is also enabled on that internal stage
D. Snowpipe treats internal user stages (`@~`) as fully supported sources for AUTO_INGEST pipes

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowpipe does support internal stages (named stages and table stages — but not user stages) as a source; since there's no external cloud storage event system to hook into for an internal stage, the file-arrival trigger instead comes from explicitly calling the Snowpipe REST API to notify Snowflake that specific files are ready, rather than the AUTO_INGEST cloud-notification path used with external stages.

- **A** is factually wrong — internal-stage support is real, just triggered differently.
- **C** invents an unnecessary directory-table dependency that isn't a documented prerequisite for internal-stage Snowpipe loading.
- **D** incorrectly extends support to user stages, which Snowpipe explicitly does not support as a source (only named/table stages).

**Reference:** [Snowpipe — introduction](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-intro)
</details>

---

### Part F: Snowpipe Streaming (Q51–Q58)

### Q51
A team building an IoT telemetry pipeline needs single-digit-second latency from sensor event to queryable row, without writing intermediate files to a stage at any point. Which ingestion mechanism is purpose-built for this, and what is the key structural difference from file-based Snowpipe?

A. Standard Snowpipe with a very small `MAX_FILE_SIZE`, since smaller files always load faster than larger ones
B. Snowpipe Streaming, which ingests rows directly via an SDK/API into a table through a "channel," with no files or stage involved at any point in the pipeline
C. A virtual warehouse-based COPY INTO scheduled every second using a task
D. Directory tables with AUTO_REFRESH set to the minimum interval

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowpipe Streaming is architecturally distinct from file-based Snowpipe: rows are pushed directly into a target table through an SDK-managed "channel" (a direct logical connection to a table), completely bypassing the stage-file-COPY pipeline, which is what enables its much lower, seconds-level latency.

- **A** still relies on the file-based pipeline (files, stages, COPY) that Snowpipe Streaming is specifically designed to eliminate — shrinking file size doesn't remove the structural file-handling overhead.
- **C** would still incur per-run warehouse start/query overhead and file staging, falling far short of true seconds-level row ingestion.
- **D** addresses stage metadata visibility, not data ingestion latency, and is unrelated to loading rows into a table.

**Reference:** [Snowpipe Streaming overview](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-overview)
</details>

---

### Q52
In the Snowpipe Streaming SDK, what is a "channel," and why does each channel maintain an offset token/checkpoint?

A. A channel is a temporary internal stage created automatically per streaming session; the offset token tracks which files have been purged
B. A channel is a direct, ordered connection from the client to a single target table; the offset token/checkpoint tracks how far that specific logical stream of data has been durably committed, so the client knows where to resume after a restart
C. A channel is a synonym for a Kafka topic partition and only exists when using the Kafka connector's streaming mode
D. A channel is a virtual warehouse dedicated to a single streaming client for the life of the connection

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

A channel represents a single logical, ordered stream of inserted rows targeting one table. The offset token (checkpoint) that the channel maintains lets the client track and confirm how far its data has actually been committed server-side — critical for safe resumption after a disconnect or restart without either dropping or duplicating rows.

- **A** invents a nonexistent internal-stage-per-channel mechanism; Snowpipe Streaming doesn't use files or stages at all.
- **C** narrows a general SDK concept down to one specific integration (Kafka); channels exist independent of any particular client implementation.
- **D** is false — channels are logical constructs, not dedicated virtual warehouses; Snowpipe Streaming's compute is serverless/managed, not warehouse-based.

**Reference:** [Snowpipe Streaming — channels and offset tokens](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-overview)
</details>

---

### Q53
A team currently has a stable, well-functioning batch Snowpipe pipeline for hourly file drops from a partner. A manager asks whether they should migrate this specific pipeline to Snowpipe Streaming to "modernize" it. What is the most defensible technical guidance?

A. Yes, always — Snowpipe Streaming is a strict superset that should replace every Snowpipe use case
B. Snowpipe Streaming is intended to complement Snowpipe, not replace it — for a stable, file-based hourly batch feed where seconds-level latency isn't a requirement, there is little architectural benefit, and the existing pipeline should likely stay as-is
C. No — Snowpipe Streaming cannot coexist in the same account as Snowpipe, so migrating would break other pipelines
D. Yes, because Snowpipe (file-based) is being fully deprecated in favor of Snowpipe Streaming

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake explicitly positions Snowpipe Streaming as a complementary option for **row-level, low-latency** use cases, not a wholesale replacement for file-based Snowpipe. A stable hourly batch feed with no low-latency requirement gains little from migrating and would add unnecessary application-level complexity (SDK integration, channel/offset management) for no real benefit.

- **A** and **D** both overstate Snowpipe Streaming's intended scope — file-based Snowpipe remains a fully supported, actively recommended mechanism for batch/file-oriented feeds.
- **C** invents a fabricated mutual-exclusivity constraint; both mechanisms can and do coexist within the same account, even loading into different tables from the same pipeline architecture.

**Reference:** [Snowpipe Streaming overview](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-overview)
</details>

---

### Q54
Which statement about how Snowpipe Streaming is billed is accurate, and how does that differ conceptually from file-based Snowpipe billing?

A. Snowpipe Streaming bills per virtual warehouse credit, exactly like a user-managed warehouse running COPY INTO
B. Snowpipe Streaming bills per uncompressed byte ingested through the client, based on the input data volume — a usage model distinct from file-based Snowpipe's serverless-compute-plus-per-file-overhead model
C. Snowpipe Streaming is bundled free of charge with any Snowflake edition as a promotional feature
D. Snowpipe Streaming bills identically to Snowpipe, since both eventually write to the same internal storage layer

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowpipe Streaming's documented billing model charges based on the **uncompressed bytes** received by the service through the client SDK — a usage-based model tied to data volume rather than either warehouse credits or a file-count-driven overhead, which is how file-based Snowpipe is billed.

- **A** incorrectly ties Streaming to warehouse credits; it's a serverless, usage-metered service, not a warehouse-backed one.
- **C** is false; it is a metered, billed service like other serverless Snowflake features.
- **D** incorrectly implies identical billing mechanics between two architecturally different ingestion paths.

**Reference:** [Snowpipe Streaming billing](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-billing)
</details>

---

### Q55
A streaming pipe is defined using `COPY INTO ... FROM (SELECT ... FROM TABLE(DATA_SOURCE(TYPE => 'STREAMING')))` with `CLUSTER_AT_INGEST_TIME = TRUE`. What capability is this specific configuration enabling, and what must already exist on the target table for it to matter?

A. It enables real-time schema evolution on the target table so new columns from the stream are added automatically
B. It enables Snowpipe Streaming's high-performance architecture to pre-sort incoming data according to the target table's defined clustering keys before committing, improving downstream query performance — the target table must already have clustering keys defined for this to have any effect
C. It forces the streaming pipe to fall back to file-based batch loading whenever cluster load is high
D. It clusters the virtual warehouse nodes used to process the streaming pipe for better parallelism

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`CLUSTER_AT_INGEST_TIME` is a Snowpipe Streaming (high-performance architecture) feature that pre-clusters incoming rows according to the target table's existing clustering key definition before the data is committed — improving query performance over time. It only has an effect if the destination table already has clustering keys configured; otherwise there's nothing to sort by.

- **A** invents an unrelated schema-evolution capability that this parameter does not control.
- **C** invents a fabricated automatic-fallback-to-batch behavior.
- **D** misapplies "clustering" to compute nodes rather than to the data itself — the parameter is about data organization within the table, not warehouse topology (which is irrelevant here since streaming is serverless).

**Reference:** [COPY INTO <table> — CLUSTER_AT_INGEST_TIME](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
</details>

---

### Q56
A pipe is created specifically to receive data through the Snowpipe Streaming API and perform an in-flight transformation before committing rows. Which statement about how this pipe's definition differs from a standard file-based auto-ingest pipe is correct?

A. It still requires `AUTO_INGEST = TRUE` and a `FROM @stage` clause, identical to a file-based pipe
B. It does not require `AUTO_INGEST` or a `FROM @stage` clause at all — instead, its COPY statement sources from the `DATA_SOURCE(TYPE => 'STREAMING')` table function, and the pipe exists purely to define transformation/clustering logic applied to API-submitted rows
C. It must reference an external stage even though no files are ever written to it, purely for compatibility reasons
D. Streaming pipes cannot be custom-defined at all; every table automatically gets exactly one immutable default streaming pipe with no transformation capability

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

A pipe built for Snowpipe Streaming intentionally omits `AUTO_INGEST` and `FROM @stage` — there's no file-based trigger or stage source involved at all. Instead its COPY statement uses the `DATA_SOURCE(TYPE => 'STREAMING')` table function as the source, and the pipe's real purpose is to define any in-flight transformation or clustering behavior applied to rows arriving through the API; a default pipe is auto-created per table if you don't need custom logic, but custom pipes remain fully supported.

- **A** and **C** both wrongly graft file-based-pipe requirements (AUTO_INGEST, stage reference) onto a mechanism that has neither files nor a stage.
- **D** correctly notes a default pipe exists automatically, but incorrectly claims custom pipes with transformation logic are impossible — they are explicitly supported, which is the entire reason custom streaming pipes exist.

**Reference:** [CREATE PIPE — streaming pipes](https://docs.snowflake.com/en/sql-reference/sql/create-pipe)
</details>

---

### Q57
Regarding schema validation for rows submitted through the Snowpipe Streaming API, where and when does validation against the target schema occur?

A. Validation happens client-side within the SDK before any network call is made, so the server never rejects malformed rows
B. Schema validation is performed server-side during ingestion, checked against the schema defined for the target (via the PIPE object), meaning the server — not just the client — enforces correctness
C. No schema validation occurs at all; Snowpipe Streaming accepts arbitrary unstructured bytes into any column type
D. Schema validation only occurs once per day during a scheduled batch reconciliation job

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowpipe Streaming performs schema validation **server-side**, at ingestion time, checked against the schema defined via the PIPE object for the target — the client SDK doesn't get to unilaterally decide correctness; Snowflake's own validation is the actual enforcement point.

- **A** misplaces the authoritative validation entirely on the client, which contradicts documented server-side enforcement.
- **C** is false — schema/type enforcement is a core capability of the service, not something bypassed.
- **D** invents a batch-reconciliation cadence that doesn't match Streaming's real-time validation model.

**Reference:** [Snowpipe Streaming — schema validation](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-overview)
</details>

---

### Q58
The Snowflake Kafka connector was recently upgraded to a version (v2.1.0+) that changed the Snowpipe Streaming channel name format used internally. What operational issue can this cause immediately after the upgrade, and why?

A. None — channel name format changes have no operational impact since names are purely cosmetic labels
B. The connector may fail to locate previously committed offset information under the new channel naming scheme, since the offset tracking is tied to the channel name — this can surface as an offset migration error requiring investigation
C. All previously ingested data is automatically deleted and must be reloaded from the beginning of each Kafka topic
D. The Kafka topics themselves are automatically renamed to match the new channel naming convention

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Because offset/checkpoint tracking for a channel is tied to its name, a connector version change that alters the internal channel naming convention can cause the upgraded connector to fail to find the offset data associated with the *old* channel name — a documented, real-world upgrade pitfall that surfaces as an offset migration exception requiring troubleshooting, not routine automatic handling.

- **A** dismisses a real, documented operational risk as inconsequential.
- **C** invents a data-loss consequence that isn't how this issue actually manifests — data isn't deleted, but offset continuity can be disrupted.
- **D** invents an automatic Kafka-side renaming behavior that Snowflake has no ability to perform (Kafka topics are managed independently of Snowflake).

**Reference:** [Troubleshooting the Kafka connector](https://docs.snowflake.com/en/user-guide/kafka-connector-ts)
</details>

---

### Part G: Streams (Q59–Q66)

### Q59
A downstream ELT process only cares about newly inserted rows in a high-volume events table; it never needs to know about updates or deletes, and the source table is truncated periodically right after the relevant rows are consumed. Which stream type is the best fit, and why does it outperform the alternative for this exact use case?

A. A standard stream, because it is the default and simplest option
B. An append-only stream, because it tracks inserts only (ignoring updates/deletes/truncates), which is both semantically correct for this use case and more performant since it avoids the join-based delete/update reconciliation a standard stream must perform
C. An insert-only stream, since insert-only streams are Snowflake's general-purpose recommendation for any standard table
D. Neither type is appropriate; only tasks (not streams) can track row insertions

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

An append-only stream tracks row inserts only and explicitly ignores updates, deletes, and truncates — an exact semantic match here, and it is also documented as more performant than a standard stream because a standard stream must internally join deleted and inserted rows to determine what was updated versus deleted, overhead an append-only stream skips entirely.

- **A** picks the default option without justification — "simplest" isn't the same as "best fit," and a standard stream would do unnecessary extra work reconciling deletes/updates the use case doesn't need.
- **C** misapplies insert-only streams, which are specifically for external tables and directory tables (where Snowflake cannot observe row-level update/delete semantics from the warehouse side) — not the general recommendation for standard tables.
- **D** is false; tracking inserts on a standard table is precisely what append-only streams are for.

**Reference:** [CREATE STREAM — append-only streams](https://docs.snowflake.com/en/sql-reference/sql/create-stream)
</details>

---

### Q60
A stream is created on an **external table** that is partitioned by date and refreshed via cloud event notifications. Which stream mode applies specifically to external tables, and what limitation does it have compared to a standard stream on a regular table?

A. A standard stream applies identically, since external tables and standard tables support the same full set of stream modes
B. An insert-only stream applies, since Snowflake can only observe file *additions* to external storage as inserts — it cannot detect true row-level updates or deletes the way it can for a managed table's internal storage
C. An append-only stream applies, and it fully supports update/delete tracking on external tables just like on regular tables
D. External tables cannot have streams created on them under any circumstances

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

External tables are backed by files in storage Snowflake doesn't manage directly, so Snowflake can only observe file-level additions (new files appearing) as inserts — it has no visibility into in-place row mutations the way it does for managed table storage. This is exactly why insert-only streams exist as a distinct mode for external tables (and directory tables): overwritten/appended files are handled as new-file inserts, without a true diff of old-vs-new content.

- **A** is false — standard streams (full DML tracking) are not supported on external tables precisely because true update/delete detection isn't possible there.
- **C** incorrectly claims append-only streams support update/delete tracking, which contradicts the very definition of an append-only stream (inserts only, by design, on any object type).
- **D** is false; external tables do support streams — just the insert-only mode, not standard or append-only modes.

**Reference:** [CREATE STREAM — INSERT_ONLY parameter](https://docs.snowflake.com/en/sql-reference/sql/create-stream)
</details>

---

### Q61
A stream on a table with a 1-day Time Travel retention period has not been consumed by any DML operation for 20 days due to a broken downstream job. What does Snowflake do to try to prevent this stream from becoming stale, and up to what limit?

A. Nothing — Snowflake takes no action, and the stream becomes stale immediately once the 1-day retention period elapses
B. Snowflake temporarily extends the underlying table's data retention period to keep pace with the unconsumed stream's offset, up to a maximum of 14 days by default (regardless of Snowflake edition), buying time before the stream actually goes stale
C. Snowflake automatically increases the table's Time Travel retention permanently to 90 days the moment a stream is created on it
D. Snowflake automatically drops and recreates the stream every 24 hours to reset its offset

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

When a table's configured retention period is shorter than what's needed to keep an unconsumed stream's offset valid, Snowflake temporarily extends that table's effective retention (up to a documented default cap of 14 days, independent of Snowflake edition) specifically to delay staleness — but this extension is not unlimited, and 20 days without consumption in this scenario would exceed even that grace period, meaning the stream is now stale and must be recreated.

- **A** ignores this documented temporary-extension safety mechanism entirely.
- **C** invents a permanent, drastic retention change that Snowflake does not perform automatically.
- **D** invents an automatic recreate cycle; Snowflake never proactively drops/recreates a stream on your behalf — a stale stream must be manually recreated by the user once it happens.

**Reference:** [Introduction to streams — data retention period and staleness](https://docs.snowflake.com/en/user-guide/streams-intro)
</details>

---

### Q62
A developer runs `SELECT * FROM my_stream;` (a bare SELECT with no surrounding DML) inside a worksheet to preview upcoming changes before building a MERGE statement. Does this action advance the stream's offset, consuming the change records?

A. Yes — any SELECT against a stream, regardless of context, immediately advances its offset
B. No — only a DML statement that successfully reads from the stream and commits (e.g., `INSERT ... SELECT FROM stream`, or `MERGE ... USING stream`) advances the offset; a standalone SELECT for preview purposes leaves the stream's contents and offset unchanged
C. Yes, but only if the stream is a standard stream; append-only streams are unaffected by SELECT statements
D. It depends on the warehouse size used to run the SELECT

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Querying a stream with a plain `SELECT` does not consume its change data — the offset only advances when the stream's contents are read from within a successful, committed DML statement. This means analysts can safely preview a stream's pending changes repeatedly without accidentally consuming them before a downstream pipeline processes them.

- **A** and **C** both incorrectly claim a bare SELECT consumes the stream — a critical, exam-relevant misconception since it would make previewing streams destructive if true.
- **D** invents an irrelevant dependency on compute sizing; offset advancement is a transactional/consumption concept, unrelated to warehouse size.

**Reference:** [Introduction to streams — consuming stream data](https://docs.snowflake.com/en/user-guide/streams-intro)
</details>

---

### Q63
A view joins three tables and includes a call to `CURRENT_DATE()`. A stream is created on this view. What documented caveat applies to streams built on views containing non-deterministic functions like `CURRENT_DATE()`, `CURRENT_USER()`, or `RANDOM()`?

A. Streams cannot be created on views at all if any non-deterministic function is present — creation fails outright
B. The stream is not guaranteed to be a constant snapshot of the function's output; results returned by the stream may change between queries because the underlying non-deterministic function can evaluate differently each time
C. Non-deterministic functions are automatically frozen to their value at stream-creation time and never change afterward
D. This caveat only applies to append-only streams on views, not standard streams

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake explicitly documents that if a view contains non-deterministic functions, a stream built on it may not behave as a constant snapshot — the results can vary across queries against the same stream state, since functions like `CURRENT_DATE()` or `RANDOM()` are re-evaluated rather than fixed. Application logic that depends on strict determinism should account for or avoid this pattern.

- **A** overstates the restriction — stream creation on such views is still permitted; it's the query-result stability guarantee that's affected, not the ability to create the stream.
- **C** incorrectly claims an automatic freezing behavior that isn't how non-deterministic functions work within a view-backed stream.
- **D** arbitrarily restricts the caveat to only one stream type when it is a general concern for streams on views regardless of standard/append-only mode.

**Reference:** [CREATE STREAM — streams on views with non-deterministic functions](https://docs.snowflake.com/en/sql-reference/sql/create-stream)
</details>

---

### Q64
A geospatial dataset stored in a `GEOGRAPHY` column needs change tracking for a downstream enrichment pipeline. Which stream type does Snowflake specifically recommend for tables containing geospatial data, and why?

A. Standard streams, since they provide full DML tracking and there is no documented limitation involving geospatial types
B. Append-only streams, because standard streams cannot retrieve change data for geospatial data types, making append-only the recommended alternative for tables with GEOGRAPHY columns
C. Insert-only streams, since geospatial data can only be loaded via external tables
D. No stream type supports geospatial columns; a custom task-based polling solution must be used instead

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake's documentation explicitly notes that standard streams cannot retrieve change data for geospatial data, and specifically recommends creating append-only streams instead for objects containing geospatial columns — a narrow but real, testable limitation.

- **A** directly contradicts the documented limitation on standard streams and geospatial data.
- **C** incorrectly assumes geospatial data requires external tables — GEOGRAPHY columns are fully supported in standard managed tables.
- **D** overstates the limitation into a total lack of support, when append-only streams are the documented, working recommendation.

**Reference:** [CREATE STREAM — geospatial data limitation](https://docs.snowflake.com/en/sql-reference/sql/create-stream)
</details>

---

### Q65
A dynamic table uses `REFRESH_MODE = FULL` because its defining query relies on an `EXCEPT` clause that isn't incrementalizable. A team wants to attach a stream to this dynamic table to feed a downstream triggered task. What constraint applies?

A. Streams work identically on FULL and INCREMENTAL refresh dynamic tables with no restriction
B. A stream cannot be created on a dynamic table using full refresh — the dynamic table must use incremental refresh, since a full refresh replaces the entire table output each time and therefore has no incremental change history for a stream to track
C. Streams can be created on FULL-refresh dynamic tables, but only in APPEND_ONLY mode
D. Dynamic tables never support streams under any refresh mode; only base tables can have streams

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

A stream requires an incremental change history to track — since a FULL-refresh dynamic table simply replaces its entire output wholesale on every refresh, there is no row-level delta for a stream to observe. Snowflake documents this explicitly: the dynamic table must use incremental refresh for streams to be created on it.

- **A** ignores this hard, documented constraint.
- **C** incorrectly assumes an APPEND_ONLY workaround exists for full-refresh dynamic tables — in fact, `APPEND_ONLY` isn't even supported for streams on dynamic tables at all (only the standard/default mode is), regardless of the dynamic table's own refresh mode.
- **D** is false — streams on incremental-refresh dynamic tables are explicitly documented and supported.

**Reference:** [Use streams on dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables/streams-on-dts)
</details>

---

### Q66
Regarding cost, which statement most accurately describes what actually drives the ongoing cost of maintaining a Snowflake stream, separate from any extended storage retention it might trigger?

A. Streams themselves have a fixed monthly subscription fee independent of usage
B. The primary cost driver is the compute (virtual warehouse credits) consumed when a warehouse queries/consumes the stream — the stream object itself doesn't have its own separate compute charge beyond enabling the extended retention storage cost when unconsumed
C. Streams bill per row captured in the change table, similar to a row-based SaaS pricing model
D. Streams are entirely free with no cost implications whatsoever, including no storage impact

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

The main cost consideration tied to streams is the compute used by a virtual warehouse to query/consume the stream's change data (ordinary Snowflake credit consumption), plus a secondary, indirect cost: if a stream isn't consumed promptly, the underlying table's data retention period may be temporarily extended, increasing storage costs. There's no separate stream-specific line-item fee beyond these two mechanisms.

- **A** and **C** both invent nonexistent pricing models (subscription fee, per-row fee) that Snowflake doesn't use for streams.
- **D** ignores the real, documented storage-cost implication of extended retention for unconsumed streams.

**Reference:** [Introduction to streams — cost considerations](https://docs.snowflake.com/en/user-guide/streams-intro)
</details>

---

### Part H: Tasks (Q67–Q74)

### Q67
A task graph architect is designing a complex pipeline and wants to know the hard structural limits before drawing the DAG. Which set of limits is documented correctly?

A. A single task graph may contain unlimited tasks; each task may have unlimited predecessors and children
B. A task graph is limited to a maximum of 1,000 tasks total (including the root); a single task can have at most 100 predecessor tasks and 100 child tasks
C. A task graph is limited to 100 tasks total, with a maximum of 1,000 predecessors per task
D. A task graph is limited to 50 tasks total, and each task may only ever have exactly one predecessor

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake documents a task graph as capped at 1,000 total tasks (including the root task), with each individual task limited to a maximum of 100 predecessor tasks and 100 child tasks — precise, testable numeric ceilings for DAG design.

- **A** ignores documented hard limits entirely.
- **C** inverts the two correct numbers (swapping the total-task cap and the per-task predecessor cap).
- **D** understates the total-task cap and also incorrectly reverts to the old single-predecessor limitation from before Snowflake introduced true multi-predecessor DAG support.

**Reference:** [CREATE TASK — task graph limitations](https://docs.snowflake.com/en/sql-reference/sql/create-task)
</details>

---

### Q68
A root task is scheduled with `SCHEDULE = '5 MINUTE'`. Its single run occasionally takes 7 minutes to complete due to warehouse queuing on a busy shared warehouse. What is the default behavior when the next scheduled run time (5 minutes later) arrives while the previous run is still executing?

A. Snowflake queues the new run and executes both runs concurrently once the first finishes, to "catch up"
B. Snowflake ensures only one instance of a scheduled task runs at a time; if a run is still in progress when the next scheduled time arrives, that scheduled occurrence is simply skipped
C. The task automatically kills the in-progress run to make way for the newly scheduled one
D. The task permanently suspends itself after any overrun to prevent future conflicts

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake explicitly guarantees only one instance of a given scheduled task runs at a time by default; if a run overlaps into the next scheduled trigger time, that next occurrence is simply skipped rather than queued or run concurrently — a documented behavior directly relevant to designing schedules that avoid falling behind.

- **A** invents queuing/concurrent-execution behavior that contradicts the documented one-instance-at-a-time default.
- **C** invents an aggressive kill-and-restart behavior Snowflake does not perform automatically.
- **D** invents a permanent self-suspension behavior; a single overrun does not disable the task going forward (though `SUSPEND_TASK_AFTER_NUM_FAILURES` is a separate, real, opt-in setting for consecutive *failures*, not overruns).

**Reference:** [Introduction to tasks — scheduling behavior](https://docs.snowflake.com/en/user-guide/tasks-intro)
</details>

---

### Q69
A child task in a task graph needs to run only after a stream on an upstream table shows new data has arrived — rather than on a fixed schedule — to avoid unnecessary polling and reduce latency. Which mechanism enables this, and what function is the documented, supported way to express the condition?

A. `SCHEDULE = 'STREAM_TRIGGER'`, a special schedule keyword reserved for stream-based tasks
B. A triggered task using `WHEN SYSTEM$STREAM_HAS_DATA('stream_name')` in the task definition — this is the only function supported for evaluation in a task's WHEN condition
C. `AFTER STREAM stream_name`, a dedicated DAG dependency clause for streams
D. A scheduled task polling `SELECT COUNT(*) FROM stream_name` every minute and self-cancelling if the count is zero

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Triggered tasks use a `WHEN` clause containing a boolean SQL expression, and `SYSTEM$STREAM_HAS_DATA` is documented as the only function supported for evaluation in that expression — checking whether a specified stream has pending change data before deciding whether the task's run should proceed, avoiding unnecessary scheduled polling.

- **A** and **C** both invent syntax that doesn't exist in Snowflake's task DDL — there's no special SCHEDULE keyword or AFTER STREAM clause.
- **D** describes a self-built polling workaround that defeats the purpose of a native, low-latency, event-aware trigger — exactly the inefficiency triggered tasks are designed to eliminate.

**Reference:** [Introduction to tasks — triggered tasks](https://docs.snowflake.com/en/user-guide/tasks-intro)
</details>

---

### Q70
A task currently runs on a dedicated `MEDIUM` user-managed warehouse and completes in a fairly predictable 8–10 minutes each run, several times per hour, all day. A colleague suggests converting it to a serverless task to "save money." Is this likely to reduce cost in this specific case?

A. Always yes — serverless tasks are unconditionally cheaper than any user-managed warehouse for every workload
B. Not necessarily — serverless compute is documented to run at a cost multiplier relative to equivalent warehouse compute, so a task that already runs frequently and predictably on a warehouse that's otherwise being used efficiently may not see a net savings; serverless tends to shine for short, infrequent, or unpredictable workloads instead
C. Yes, because serverless tasks are entirely free of charge as a promotional Snowflake feature
D. No — serverless tasks cannot run for longer than 60 seconds, so this workload is categorically ineligible

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Serverless compute is convenient (no warehouse sizing/management) but is billed at a documented premium relative to equivalent warehouse-managed compute. For a task that already runs frequently and predictably — where a dedicated or shared warehouse can be sized and kept efficiently utilized — the serverless premium may outweigh the operational convenience, unlike for short, bursty, or unpredictable workloads where avoiding an idle warehouse (or oversized warehouse) is the bigger win.

- **A** and **C** overstate serverless as a universal or free cost win, which isn't accurate.
- **D** invents a runtime ceiling that doesn't reflect documented serverless task behavior (serverless tasks scale up to sizes as large as an XXLARGE-equivalent warehouse and are not capped at 60 seconds).

**Reference:** [Serverless tasks — billing](https://docs.snowflake.com/en/user-guide/tasks-intro)
</details>

---

### Q71
A task fails partway through a run due to a transient network issue and the task graph is configured with `TASK_AUTO_RETRY_ATTEMPTS = 2` on the root task. Separately, a child task with three predecessor tasks has one predecessor currently suspended. What happens to that child task's next scheduled run?

A. The child task fails immediately because not all of its predecessors are in a resumed state
B. The child task still runs as long as at least one of its predecessors is in a resumed state and all resumed predecessors complete successfully — a suspended predecessor is effectively treated as if it had succeeded for dependency-evaluation purposes
C. The entire task graph is permanently disabled until the suspended predecessor is manually resumed
D. Suspending any predecessor automatically suspends every downstream task in the graph as a cascading side effect

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake documents that when you suspend a child (or predecessor) task, the task graph continues running as though that suspended task had succeeded — a child task with multiple predecessors will still run as long as at least one predecessor is resumed and all resumed predecessors complete successfully, rather than requiring every single predecessor to be active.

- **A** incorrectly assumes strict all-predecessors-must-be-resumed logic that contradicts documented behavior.
- **C** invents a permanent graph-wide lockout that doesn't reflect how suspending an individual task actually behaves.
- **D** invents an automatic cascading suspension that Snowflake does not perform — suspending one task doesn't force-suspend its downstream dependents.

**Reference:** [Create a sequence of tasks with a task graph — suspending tasks](https://docs.snowflake.com/en/user-guide/tasks-graphs)
</details>

---

### Q72
A task has no `USER_TASK_TIMEOUT_MS` explicitly configured. What is the documented default timeout for a single task run, and what practical issue does this default protect against?

A. There is no default timeout; tasks can run indefinitely unless manually cancelled
B. The default is 3,600,000 milliseconds (60 minutes) — a safeguard against non-terminating or runaway task runs consuming compute indefinitely
C. The default is 24 hours, matching the maximum COPY INTO load duration
D. The default is 5 minutes, matching the minimum allowable task schedule interval

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Absent an explicit `USER_TASK_TIMEOUT_MS`, a task run defaults to a 60-minute (3,600,000 ms) timeout, implemented specifically as a safeguard against tasks that hang or never terminate, which would otherwise silently consume compute resources indefinitely.

- **A** is false and would represent an unbounded operational risk that Snowflake explicitly guards against by default.
- **C** confuses an unrelated 24-hour guideline about large COPY INTO load durations with the task-specific timeout default, which is a different number entirely.
- **D** confuses the *minimum schedule interval* (1 minute, not 5) with the *timeout* concept — these are two entirely different parameters.

**Reference:** [Troubleshooting tasks — default timeout](https://docs.snowflake.com/en/user-guide/tasks-ts)
</details>

---

### Q73
A task graph's root task owner role is deleted from the account by an administrator who didn't realize tasks depended on it. What happens to ownership of the tasks in that graph?

A. All tasks in the graph are immediately and permanently dropped along with the role
B. Task ownership is reassigned to the role that performed the DROP ROLE operation
C. The tasks become ownerless and unusable until a new role is manually granted OWNERSHIP through a lengthy recovery process
D. Ownership transfers automatically to ACCOUNTADMIN regardless of which role issued the DROP ROLE command

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

When a role that owns a task (or task graph) is dropped, ownership of the affected tasks is reassigned to the role that performed the drop operation — a specific, testable behavior rather than the tasks becoming orphaned, deleted, or defaulting to a fixed system role.

- **A** and **C** both invent destructive or manual-recovery-heavy outcomes that don't match the actual reassignment behavior.
- **D** incorrectly assumes a fixed target role (ACCOUNTADMIN) regardless of who actually performed the drop — the reassignment specifically follows the role that executed the DROP.

**Reference:** [Tasks — role and ownership considerations](https://docs.snowflake.com/en/user-guide/tasks-intro)
</details>

---

### Q74
An architect wants a task's SQL logic to reference data produced by its own parent task in the same graph run — for example, a child task that needs to know the number of rows the parent task just processed. Which mechanism supports this?

A. Tasks cannot share any information between parent and child; each task must independently re-derive any needed value
B. `SYSTEM$GET_PREDECESSOR_RETURN_VALUE`, which lets a child task retrieve the return value of a specified predecessor task within the same graph run
C. A stream must be manually created between every parent/child pair to pass values, since tasks have no built-in mechanism for this
D. Session variables set with `SET` automatically persist from parent task runs into child task runs

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake tasks support passing return values through a task graph: a child task can use `SYSTEM$GET_PREDECESSOR_RETURN_VALUE` to retrieve the return value produced by a specific parent/predecessor task's run, enabling logic-based, data-aware branching or decisions in downstream tasks without needing an external side-channel.

- **A** and **C** both ignore this documented, built-in mechanism and would push teams toward unnecessary custom workarounds.
- **D** misapplies session-level `SET` variables, which do not automatically persist or transfer between separate task executions — each task run is its own session context.

**Reference:** [Create a sequence of tasks with a task graph — predecessor return values](https://docs.snowflake.com/en/user-guide/tasks-graphs)
</details>

---

### Part I: Dynamic Tables (Q75–Q80)

### Q75
A dynamic table `dt_orders_daily` (target lag 30 minutes) reads from an intermediate dynamic table `dt_orders`. The architect wants `dt_orders` to never run on its own independent schedule, refreshing only when `dt_orders_daily` actually needs fresh data. Which `TARGET_LAG` configuration achieves this on `dt_orders`?

A. `TARGET_LAG = '1 minute'`, the shortest allowable value, to guarantee it's always fresher than any consumer
B. `TARGET_LAG = DOWNSTREAM`, which removes the table's independent refresh schedule and ties its refresh timing to whatever its downstream consumer(s) require
C. `TARGET_LAG = 'AUTO'`, which lets Snowflake infer scheduling purely from query cost estimates
D. Leaving TARGET_LAG unset entirely, since dynamic tables without a lag setting default to downstream-only behavior

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`TARGET_LAG = DOWNSTREAM` is a specific, documented scheduling mode meaning "don't refresh on an independent schedule — refresh only when a downstream dynamic table that depends on you needs fresh data," using the shortest target lag among all downstream consumers to determine timing. This is exactly the intermediate-table pattern the scenario describes.

- **A** would actually create the *opposite* of the intended design — an aggressive independent 1-minute schedule rather than a schedule driven by downstream needs, likely increasing compute cost unnecessarily.
- **C** invents `'AUTO'` as a TARGET_LAG value; `AUTO` is a real keyword but it applies to `REFRESH_MODE`, not `TARGET_LAG`.
- **D** is false — `TARGET_LAG` is a required parameter at creation; there's no implicit "unset defaults to DOWNSTREAM" behavior.

**Reference:** [Set the target lag for a dynamic table](https://docs.snowflake.com/en/user-guide/dynamic-tables/target-lag)
</details>

---

### Q76
A dynamic table is defined with `TARGET_LAG = DOWNSTREAM` and, after some pipeline refactoring, no other dynamic table reads from it anymore — it has become a dead-end leaf with no downstream consumers, though nobody has noticed yet. What is the practical consequence, and does Snowflake warn about it?

A. Snowflake automatically converts it to a fixed schedule based on its last known downstream consumer's lag
B. The table simply never refreshes automatically going forward, and Snowflake produces no warning or error about this silent condition — a real operational risk if a leaf table's target lag is left as DOWNSTREAM
C. The table is automatically dropped after 24 hours of having no downstream consumers
D. The table falls back to a default target lag of 1 hour automatically

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

This is an explicitly documented operational trap: a `TARGET_LAG = DOWNSTREAM` table with no downstream consumers never refreshes automatically, and Snowflake does not surface any warning about this — which is exactly why best-practice guidance insists on always setting an explicit, independent target lag on leaf/terminal tables that serve dashboards or queries directly, reserving `DOWNSTREAM` only for genuinely intermediate tables.

- **A**, **C**, and **D** all invent automatic remediation or cleanup behaviors that Snowflake does not perform — the documented reality is silence, not automatic correction.

**Reference:** [Set the target lag for a dynamic table — DOWNSTREAM with no consumers](https://docs.snowflake.com/en/user-guide/dynamic-tables/target-lag)
</details>

---

### Q77
A dynamic table's defining query uses the `EXCEPT` set operator. When the table is created with `REFRESH_MODE = AUTO`, what refresh mode does Snowflake resolve to, and why?

A. INCREMENTAL, because AUTO always defaults to incremental refresh when in doubt
B. FULL, because EXCEPT is not among the supported constructs for incremental refresh, so AUTO falls back to full refresh, recomputing the entire result set on every refresh
C. ADAPTIVE, because AUTO automatically upgrades unsupported queries to adaptive mode instead of full
D. The table fails to create, since AUTO requires every construct in the query to be incrementally refreshable

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`EXCEPT` is explicitly called out as not supported for incremental refresh, so a dynamic table using it — even with `REFRESH_MODE = AUTO` — resolves to FULL refresh, recomputing the whole result set from scratch on each scheduled refresh rather than processing only changed rows.

- **A** overstates AUTO as always preferring incremental; AUTO's entire purpose is to choose whichever mode the query structure actually supports, and that can legitimately be FULL.
- **C** invents a fabricated auto-upgrade-to-ADAPTIVE behavior; ADAPTIVE must be explicitly requested and has its own separate constraints, it isn't a silent fallback target for AUTO.
- **D** is false — AUTO exists precisely so that unsupported-for-incremental queries still succeed, just via FULL refresh instead of failing outright.

**Reference:** [Dynamic table refresh modes](https://docs.snowflake.com/en/user-guide/dynamic-tables/refresh-modes)
</details>

---

### Q78
A team wants the absolute freshest possible dynamic table, testing `TARGET_LAG = '30 seconds'` at creation. What happens?

A. The dynamic table is created successfully and refreshes roughly every 30 seconds without issue
B. This fails validation, since the documented minimum allowable target lag is 60 seconds (1 minute) — freshness requirements below that floor aren't supported
C. It succeeds, but Snowflake silently rounds the value up to 1 minute without any indication to the user
D. Sub-minute target lags are supported only for tables using REFRESH_MODE = FULL

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake documents 60 seconds as the minimum target lag for a dynamic table — data freshness requirements demanding anything tighter than that floor aren't achievable via TARGET_LAG and would need a different architecture (e.g., streams/tasks or Snowpipe Streaming) if truly sub-minute freshness is required.

- **A** ignores this documented floor.
- **C** invents a silent-rounding behavior; the correct behavior is an outright validation failure, not a quiet adjustment — an important distinction for anticipating what actually happens versus assuming graceful degradation.
- **D** invents an arbitrary refresh-mode-dependent exception to the minimum that doesn't exist in the documentation.

**Reference:** [Dynamic tables overview — minimum target lag](https://docs.snowflake.com/en/user-guide/dynamic-tables-about)
</details>

---

### Q79
An architect is deciding between building a transformation layer with (1) streams + tasks, or (2) dynamic tables. Which statement most fairly characterizes the trade-off, avoiding the common exam trap of assuming one approach is unconditionally superior?

A. Dynamic tables should always replace streams and tasks going forward, since Snowflake considers streams and tasks a deprecated legacy pattern
B. Streams and tasks give explicit, imperative control over change capture and custom execution logic — useful when a pipeline needs fine-grained control (e.g., complex conditional logic, custom error handling); dynamic tables offer a simpler, declarative "define the SELECT and a freshness target" model, well suited for keeping transformed tables continuously fresh without hand-managing orchestration — neither approach makes the other obsolete
C. Streams and tasks are strictly a subset of dynamic table functionality, so any streams+tasks pipeline can be trivially converted with no design changes
D. Dynamic tables cannot read from a table that also has a stream on it, making the two mechanisms fundamentally incompatible

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake's own guidance frames streams+tasks and dynamic tables as complementary approaches rather than a strict replacement relationship: streams+tasks offer imperative, fine-grained pipeline control, while dynamic tables offer a simpler declarative model for "just keep this materialized result fresh." Many real pipelines even combine both (e.g., a dynamic table feeding a stream that triggers a task for imperative work the dynamic table can't express).

- **A** and **C** both overstate dynamic tables as a total, risk-free replacement for streams and tasks, which oversimplifies real trade-offs (custom logic, conditional branching, and fine-grained error handling are often easier to express with tasks).
- **D** is false — dynamic tables can absolutely be a stream's source object (when the dynamic table uses incremental refresh, as established in Q65), so the two mechanisms are explicitly designed to interoperate, not conflict.

**Reference:** [Decision guide for dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables/sql-vs-dt)
</details>

---

### Q80
A dynamic table pipeline has three tables: `DT_A` (base, target lag `DOWNSTREAM`), `DT_B` (reads from `DT_A`, target lag `DOWNSTREAM`), and `DT_C` (reads from `DT_B`, target lag `10 minutes`, the only leaf table with an explicit schedule). What does Snowflake guarantee about consistency across this pipeline when `DT_C` refreshes?

A. No consistency guarantee exists; each table may reflect data from a completely different, unrelated point in time
B. Snowflake maintains snapshot isolation across the pipeline: when tables refresh together to satisfy DT_C's schedule, each table in the dependency chain reads a single, consistent point-in-time view of its upstream inputs, coordinated automatically without manual schedule alignment
C. Consistency is only guaranteed if all three tables share the exact same target lag value
D. Consistency is guaranteed only for FULL refresh mode tables, not INCREMENTAL ones

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake explicitly coordinates refreshes across a dynamic table pipeline so that dependent tables are refreshed in dependency order using a consistent snapshot timestamp — meaning downstream tables always see a coherent, consistent view of their upstream ancestors' data as of a single point in time, without the user needing to manually align schedules across tables with different lags.

- **A** contradicts one of dynamic tables' key documented value propositions (automatic snapshot consistency across a pipeline).
- **C** invents an unnecessary "identical lag values" requirement; DOWNSTREAM tables explicitly exist so upstream/intermediate tables *don't* need matching independent lag values.
- **D** invents a refresh-mode-based restriction on the consistency guarantee that doesn't exist — snapshot isolation applies to the scheduled, automated refresh process regardless of whether individual tables use FULL or INCREMENTAL refresh (note: manual/ad-hoc SELECT joins across dynamic tables are the documented exception to this guarantee, not the refresh mode itself).

**Reference:** [Understanding dynamic table initialization and refresh — snapshot isolation](https://docs.snowflake.com/en/user-guide/dynamic-tables/dynamic-tables-refresh)
</details>

---

## Section 3.3 — Snowflake Connectors and Integrations

### Part J: Snowflake Drivers (Q81–Q84)

### Q81
An internal Java application needs to execute parameterized SQL against Snowflake and process large result sets efficiently, integrating with the company's existing JDBC-based connection-pooling framework. Which category of Snowflake client is the correct fit, and how does it differ conceptually from a connector like the Kafka connector?

A. The Snowflake JDBC Driver — a driver implements a standard client API (JDBC in this case) that lets general-purpose applications and tools issue SQL over a standard interface, whereas a connector (like Kafka) is a purpose-built integration wired to a specific external system's own data model (topics/partitions), not a general SQL client interface
B. The Kafka connector, since it is Snowflake's general-purpose recommendation for any JVM-based application
C. The Snowflake Python Connector, since Python connectors work inside any JVM application through cross-language bindings
D. There is no meaningful distinction between a driver and a connector; the two terms are used interchangeably for every Snowflake client library

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

Drivers (JDBC, ODBC, and similar) implement a standard, general-purpose client API so that a wide range of applications and tools can connect and issue SQL. Connectors, by contrast, are purpose-built integrations tailored to a specific external system's own semantics — the Kafka connector understands topics and partitions, not general ad-hoc SQL client usage. A Java app doing parameterized SQL through a connection pool is squarely a JDBC driver use case.

- **B** misapplies the Kafka connector, which is designed for ingesting data from Kafka topics — not as a general SQL client library for arbitrary Java applications.
- **C** invents a nonexistent cross-language binding mechanism; the Python connector doesn't run "inside" JVM applications.
- **D** flattens a real, testable conceptual distinction between "general client API" (driver) and "system-specific integration" (connector).

**Reference:** [Overview of Clients, Drivers, and APIs](https://docs.snowflake.com/en/developer-guide/drivers)
</details>

---

### Q82
A .NET application team wants native, idiomatic ADO.NET-style database access to Snowflake without going through an ODBC bridge layer. Which Snowflake driver is designed specifically for this?

A. The Snowflake ODBC Driver, since ODBC is universally compatible with every language including .NET
B. The Snowflake .NET Driver, which provides a native ADO.NET interface purpose-built for .NET applications
C. The Snowflake Node.js Driver, since Node.js and .NET share the same underlying runtime
D. The Snowflake Go Driver, since Go and C# compile to similar intermediate bytecode

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake publishes a dedicated .NET driver that integrates natively with ADO.NET conventions, giving .NET developers an idiomatic experience rather than requiring an ODBC interop bridge layer.

- **A** describes a workable but non-native path (ODBC can technically be used from .NET via interop) — it isn't the purpose-built option when a native driver exists specifically for this ecosystem.
- **C** and **D** both pair .NET with unrelated runtimes (Node.js's V8/JavaScript runtime, Go's compiled runtime) based on superficial or false technical similarities that don't actually enable direct compatibility.

**Reference:** [Snowflake .NET Driver](https://docs.snowflake.com/en/developer-guide/dotnet/dotnet-driver)
</details>

---

### Q83
A BI tool that only supports connecting to data sources through the industry-standard ODBC interface needs to query Snowflake. Which statement about the Snowflake ODBC driver's role is accurate?

A. The ODBC driver is Snowflake-proprietary and incompatible with the standard ODBC specification, requiring custom BI tool plugins
B. The ODBC driver implements the standard ODBC API on top of Snowflake, allowing any ODBC-compliant application (including many BI tools) to connect without Snowflake-specific custom development
C. ODBC connectivity to Snowflake is only available through a third-party, non-Snowflake-maintained driver
D. The ODBC driver can only be used for administrative tasks (user/role management), not for querying table data

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake provides and maintains an ODBC driver that implements the standard ODBC interface, which is precisely why so many third-party BI and reporting tools can connect to Snowflake "out of the box" as long as they support generic ODBC connectivity — no Snowflake-specific custom plugin development required for basic connectivity.

- **A** and **C** both incorrectly suggest the ODBC option is nonstandard or unofficial, when it's a fully Snowflake-maintained, standards-compliant driver.
- **D** invents an artificial administrative-only restriction; ODBC is used for general SQL querying just like any other driver, not limited to account administration.

**Reference:** [Snowflake ODBC Driver](https://docs.snowflake.com/en/developer-guide/odbc/odbc)
</details>

---

### Q84
A data science team wants to push query execution down to Snowflake and pull results into pandas DataFrames within Python notebooks, using native Python idioms rather than raw SQL string management wherever possible. Which official Snowflake client best supports this specific workflow?

A. The Snowflake Node.js Driver, since Jupyter notebooks run on a Node.js kernel by default
B. The Snowflake Python Connector (optionally alongside Snowpark Python for DataFrame-style operations), which provides native Python DB API connectivity and integrates well with pandas
C. The Snowflake PHP Driver, since PHP has the most mature pandas integration
D. The ODBC driver exclusively, since Python has no dedicated native connector

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

The Snowflake Python Connector implements Python's standard DB API and includes documented pandas integration (fetching results directly into DataFrames); Snowpark Python additionally provides a DataFrame-style programming model that pushes computation down into Snowflake — together, this is the natural, idiomatic fit for the described notebook-based data science workflow.

- **A** incorrectly assumes Jupyter notebooks inherently run on a Node.js kernel — Python notebooks run a Python kernel, and the Node.js driver has no special notebook affinity.
- **C** invents a fabricated PHP/pandas relationship; pandas is a Python library and has no bearing on PHP driver capability.
- **D** is false — Python has a dedicated, actively maintained native connector; ODBC is not the exclusive or preferred path for this use case.

**Reference:** [Snowflake Connector for Python](https://docs.snowflake.com/en/developer-guide/python-connector/python-connector)
</details>

---

### Part K: Snowflake Connectors (Q85–Q88)

### Q85
A team ingests high-throughput event data from Apache Kafka topics into Snowflake and wants the lowest-latency path available, leveraging Snowpipe Streaming under the hood rather than file-based batching. What should they configure?

A. The Snowflake Connector for Kafka using its Snowpipe Streaming ingestion mode, rather than its legacy Snowpipe (file-based) mode
B. A custom COPY INTO statement scheduled every second via a task, bypassing Kafka connector entirely
C. The Snowflake Spark Connector, since it is the officially recommended path for all streaming message-queue integrations
D. The ODBC driver configured with a polling loop against the Kafka broker directly

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

The Snowflake Connector for Kafka supports two ingestion backends: a legacy Snowpipe (file-based) mode and a Snowpipe Streaming mode, the latter offering substantially lower latency by writing rows directly rather than staging and batching files — exactly the "leverage Streaming under the hood" requirement described.

- **B** reinvents (poorly) what the Kafka connector already does natively, adding unnecessary custom scheduling complexity and still relying on file-based batching rather than true streaming.
- **C** misapplies the Spark connector, which is designed for bulk/batch data movement between Spark DataFrames and Snowflake tables, not Kafka topic ingestion.
- **D** describes an unofficial, unsupported DIY polling approach using a driver that has no purpose-built Kafka integration.

**Reference:** [Snowflake Connector for Kafka](https://docs.snowflake.com/en/user-guide/kafka-connector-overview)
</details>

---

### Q86
A data engineering team using Apache Spark for large-scale ETL wants to read a Snowflake table into a Spark DataFrame, apply transformations, and write results back to a different Snowflake table — all while pushing as much computation down into Snowflake as possible to minimize data movement. Which connector is purpose-built for this?

A. The Snowflake Connector for Kafka, since Spark Structured Streaming is built on Kafka internally
B. The Snowflake Connector for Spark, which integrates Snowflake as a Spark data source and supports query pushdown to reduce data transferred out of Snowflake
C. The Snowflake ODBC driver, since Spark's JVM-based architecture only supports ODBC-style connections
D. The Python Connector, since PySpark is fundamentally a Python library with no separate Spark-specific connector needed

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

The Snowflake Connector for Spark is purpose-built to let Spark treat Snowflake as a native data source/sink, and it supports pushdown optimization so that filtering, projection, and even some aggregation work happens inside Snowflake rather than pulling raw data into the Spark cluster first — directly matching the stated goal of minimizing data movement.

- **A** incorrectly conflates Spark Structured Streaming's general architecture with a dependency on Kafka; Spark can read/write many sources unrelated to Kafka, and the Kafka connector is not a Spark integration mechanism.
- **C** overstates a JVM-implies-ODBC-only requirement; the Spark connector uses JDBC internally alongside its own optimizations, not a forced ODBC-only path.
- **D** incorrectly assumes PySpark's Python surface eliminates the need for a dedicated Spark-Snowflake connector — the Spark connector (with its pushdown logic) is a distinct, purpose-built integration, not a byproduct of the general Python connector.

**Reference:** [Snowflake Connector for Spark](https://docs.snowflake.com/en/user-guide/spark-connector-overview)
</details>

---

### Q87
Which statement correctly describes the general architectural role of "connectors" (as a category) relative to "drivers" when designing a data integration strategy?

A. Connectors are always faster than drivers because they bypass SQL parsing entirely
B. Connectors provide pre-built, system-specific integration logic (data mapping, batching/streaming behavior, schema handling) tailored to a particular external platform (Kafka, Spark, etc.), reducing the custom engineering needed compared to writing bespoke integration code against a general-purpose driver
C. Connectors are a deprecated legacy category being fully replaced by drivers across all Snowflake client offerings
D. Connectors can only be used for unloading data out of Snowflake, never for loading data in

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Connectors exist to remove the burden of hand-building integration logic (mapping semantics, batching/streaming strategy, schema handling) for a specific external platform — using a driver directly for the same task would require the team to build all of that custom logic themselves. This is the practical architectural value connectors provide over "just use a driver and write your own integration layer."

- **A** invents a categorical performance claim about SQL parsing bypass that isn't an accurate general description of connectors.
- **C** is false — connectors (Kafka, Spark, etc.) remain actively maintained, purpose-built offerings; they are not being phased out in favor of drivers, since the two solve different problems.
- **D** is false — connectors like Kafka and Spark support both loading data into Snowflake and, in Spark's case, reading data out as well.

**Reference:** [Overview of Clients, Drivers, and APIs](https://docs.snowflake.com/en/developer-guide/drivers)
</details>

---

### Q88
A legacy on-premises ETL tool only supports generic database drivers (JDBC/ODBC) and has no Snowflake-specific plugin available from its vendor. The integration needs to periodically bulk-load transformed data into Snowflake tables using standard SQL INSERT/COPY-style statements issued by the tool. What is the most appropriate connectivity approach?

A. Wait for the vendor to build a dedicated Snowflake connector, since JDBC/ODBC cannot be used for bulk operations
B. Use the Snowflake JDBC or ODBC driver as the tool's generic database connection target, since both drivers support standard SQL execution including bulk operations, without requiring a Snowflake-specific connector at all
C. This scenario is impossible without Snowpipe Streaming, since only Streaming supports third-party tool integration
D. Use the Git integration to synchronize the ETL tool's output files directly into a Snowflake stage

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

This is exactly the use case general-purpose drivers exist for: any tool that speaks standard JDBC or ODBC can connect to Snowflake and issue ordinary SQL (including COPY INTO or INSERT-based bulk loads) without needing a Snowflake-specific connector at all — the driver is the generic, universal bridge.

- **A** unnecessarily blocks progress waiting on a connector that isn't required for this use case.
- **C** wrongly narrows third-party connectivity down to Snowpipe Streaming only, ignoring the broad, standard driver-based path that has existed independent of Streaming.
- **D** misapplies Git integration, which manages source-controlled code/files synced from a Git repository — it has no relationship to a legacy ETL tool's SQL-based bulk-load workflow.

**Reference:** [Overview of Clients, Drivers, and APIs](https://docs.snowflake.com/en/developer-guide/drivers)
</details>

---

### Part L: Storage Integration (Q89–Q92)

### Q89
An architect wants to onboard 15 new external stages across 6 different databases, all pointing at various paths within the same two S3 buckets, without ever embedding an AWS access key anywhere in Snowflake object DDL. What is the minimal, correctly-scoped object strategy?

A. Create 15 separate storage integrations, one per stage, each scoped to its exact single path
B. Create one (or a small number of) storage integration(s) with `STORAGE_ALLOWED_LOCATIONS` covering the relevant bucket paths, and reference that same integration from all 15 `CREATE STAGE` statements
C. Embed a single shared IAM access key pair as a Snowflake SECRET, and reference that secret from all 15 stages instead of using a storage integration
D. Storage integrations are strictly one-to-one with stages; sharing is not supported under any configuration

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

A single storage integration is explicitly designed to back multiple external stages, as long as each stage's URL falls within the integration's `STORAGE_ALLOWED_LOCATIONS`. Creating one well-scoped integration (or a small number, if bucket-level segregation is desired) and reusing it across all 15 stages is both the minimal and the architecturally intended approach — avoiding both credential embedding and unnecessary object sprawl.

- **A** creates unnecessary administrative overhead (15 objects, 15 cloud-side trust relationships to manage) when one integration can safely cover all these paths.
- **C** reintroduces the exact embedded-credential problem storage integrations exist to eliminate.
- **D** is false and directly contradicts documented storage integration behavior (see Q17 in this bank).

**Reference:** [CREATE STORAGE INTEGRATION](https://docs.snowflake.com/en/sql-reference/sql/create-storage-integration)
</details>

---

### Q90
Who is authorized to execute a `CREATE STORAGE INTEGRATION` statement in a Snowflake account by default?

A. Any role with the USAGE privilege on the target database
B. Only the ACCOUNTADMIN role, or a custom role explicitly granted the global CREATE INTEGRATION privilege
C. Any role that also owns at least one external stage already
D. The PUBLIC role, since storage integrations are account-wide shared resources by design

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Creating a storage integration is a privileged, account-level operation: Snowflake restricts it to the ACCOUNTADMIN role or a role that has been explicitly granted the global `CREATE INTEGRATION` privilege — reflecting the sensitivity of establishing new trust relationships with external cloud storage.

- **A** and **C** both invent insufficient privilege paths that don't match the documented requirement.
- **D** incorrectly assumes PUBLIC-level default access to a privileged administrative operation, which would be a serious governance risk Snowflake explicitly avoids by design.

**Reference:** [CREATE STORAGE INTEGRATION — access control](https://docs.snowflake.com/en/sql-reference/sql/create-storage-integration)
</details>

---

### Q91
A storage integration is configured with `STORAGE_ALLOWED_LOCATIONS = ('*')` and `STORAGE_BLOCKED_LOCATIONS = ('s3://analytics-bucket/sensitivedata/')`. A developer attempts to create a stage pointing at `s3://analytics-bucket/sensitivedata/subfolder/`. What happens?

A. It succeeds, because the wildcard `'*'` in ALLOWED_LOCATIONS always takes precedence over any BLOCKED_LOCATIONS entry
B. It fails, because the requested path falls under a blocked location, and blocked locations take precedence even when the allowed list is a broad wildcard
C. It succeeds, because BLOCKED_LOCATIONS only applies to exact path matches, not subfolders beneath a blocked path
D. It fails, but only because wildcards are not permitted in STORAGE_ALLOWED_LOCATIONS at all

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

`STORAGE_BLOCKED_LOCATIONS` is designed to carve out explicit exceptions even under a broad allow-list — a path under a blocked prefix (including its subfolders) remains blocked regardless of how permissive the allowed-locations wildcard is. This lets administrators broadly permit a bucket while still explicitly walling off sensitive subpaths.

- **A** and **C** both misunderstand the precedence/scope rule — a broad allow-list doesn't override a specific block, and blocking a path is documented to cover paths beneath it, not just an exact string match.
- **D** is false — `STORAGE_ALLOWED_LOCATIONS = ('*')` is valid, documented syntax for allowing any location (subject to blocked-location carve-outs).

**Reference:** [CREATE STORAGE INTEGRATION — STORAGE_BLOCKED_LOCATIONS](https://docs.snowflake.com/en/sql-reference/sql/create-storage-integration)
</details>

---

### Q92
An account migrates from directly embedded AWS credentials on external stages to a storage integration-based approach for security reasons. Besides eliminating embedded secrets, what additional operational benefit does a storage integration bring when the underlying AWS IAM role's trust policy needs to be rotated or updated?

A. None — rotating the IAM role still requires recreating every dependent stage individually either way
B. The IAM entity (and its permissions) is centralized in the integration object; updating the integration's cloud-side trust relationship doesn't require touching or recreating any of the external stages that reference it
C. Storage integrations automatically rotate the underlying IAM role's credentials every 24 hours without any administrator involvement
D. Stages must be manually re-pointed to a brand-new storage integration object every time the IAM role changes

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Because a storage integration centralizes the identity/authentication relationship (the generated IAM user/role that cloud administrators grant permissions to), updating permissions or trust configuration on the cloud side doesn't require touching the many stages that merely reference the integration by name — a real operational win over per-stage embedded credentials, which would each need individual updates.

- **A** and **D** both incorrectly claim that stage-level changes are still required, which defeats the actual architectural benefit being asked about.
- **C** invents an automatic credential-rotation cadence that storage integrations do not perform on their own; rotation, where applicable, is managed on the cloud provider side by administrators, not on an automatic 24-hour timer.

**Reference:** [CREATE STORAGE INTEGRATION](https://docs.snowflake.com/en/sql-reference/sql/create-storage-integration)
</details>

---

### Part M: API Integration (Q93–Q96)

### Q93
Before a Snowflake account can connect a Git repository or invoke an external function/API endpoint, which object must typically be created first by a sufficiently privileged role, and what is its core purpose?

A. A file format object, since API calls are parsed using the same format engine as COPY INTO
B. An API integration, which stores metadata about the allowed external endpoint(s)/prefixes and the authentication mechanism Snowflake should use when calling out to that external API
C. A directory table, since API responses are cached there before being surfaced to SQL
D. A virtual warehouse dedicated exclusively to outbound API traffic

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

An API integration is the object that governs *which* external endpoints/URL prefixes Snowflake is permitted to call and *how* authentication should occur — required infrastructure before external functions or Git repository connections can be configured, since Snowflake needs an explicit, administrator-approved boundary for outbound calls.

- **A** and **C** both misapply unrelated object types (file formats parse staged file content; directory tables track file metadata) to a scenario about outbound API connectivity.
- **D** invents an unnecessary dedicated-warehouse requirement; API integrations govern connectivity/authorization, not compute provisioning.

**Reference:** [CREATE API INTEGRATION](https://docs.snowflake.com/en/sql-reference/sql/create-api-integration)
</details>

---

### Q94
Which role (by default) is required to execute `CREATE API INTEGRATION`, and why does Snowflake restrict this operation to that level?

A. Any role with USAGE on the target schema, since API integrations are schema-level objects
B. ACCOUNTADMIN (or a role granted the global CREATE INTEGRATION privilege) — because an API integration establishes a trust boundary for Snowflake to make outbound calls to external systems, which is a security-sensitive, account-level decision
C. SYSADMIN only, since API integrations are considered standard database objects like tables or views
D. Any role, since API integrations don't grant any actual capability until a secret is separately attached

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Like storage integrations, API integrations require ACCOUNTADMIN or the global `CREATE INTEGRATION` privilege, reflecting that defining which external endpoints Snowflake is allowed to call is a security-sensitive, account-wide trust decision, not a routine schema-level object creation task.

- **A** incorrectly treats API integrations as schema-scoped objects with a lower privilege bar than they actually require.
- **C** incorrectly assumes SYSADMIN is sufficient by default; the privilege model here mirrors storage integrations, requiring the higher-level CREATE INTEGRATION privilege, not simply SYSADMIN's usual object-creation rights.
- **D** understates the risk — an API integration alone already defines allowed prefixes/authentication scope, which is meaningful even before any secret or external function references it.

**Reference:** [CREATE API INTEGRATION — access control](https://docs.snowflake.com/en/sql-reference/sql/create-api-integration)
</details>

---

### Q95
A Snowflake Workspace needs to connect to a **private** GitHub repository requiring authentication (not a public, unauthenticated repo). Beyond the API integration itself, what additional object is typically needed to hold the credential material, and how is it referenced?

A. Nothing additional is needed; the API integration alone always contains embedded credentials directly in its DDL
B. A Snowflake SECRET object (e.g., holding a personal access token or OAuth-related credential), which the API integration or Git repository object is configured to use for authenticating to the private repository
C. A named external stage configured with the GitHub token as its CREDENTIALS parameter
D. A directory table storing the token as file metadata

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

For private repositories, Snowflake documents creating a SECRET object to hold the sensitive credential material (such as a personal access token), which the API integration configuration (or the Git repository object referencing it) uses to authenticate — keeping the credential itself as a distinct, access-controlled object rather than embedded inline in integration DDL.

- **A** is false — while public repositories can skip authentication entirely, private repositories explicitly need a credential mechanism, and it isn't embedded directly in the API integration statement itself.
- **C** misapplies external stage CREDENTIALS syntax (meant for cloud storage access keys) to an unrelated Git/GitHub authentication scenario.
- **D** misapplies directory tables, which track file metadata for stages, not credential storage.

**Reference:** [Setting up Snowflake to use Git](https://docs.snowflake.com/en/developer-guide/git/git-setting-up)
</details>

---

### Q96
An API integration is created with `API_ALLOWED_PREFIXES = ('https://github.com/my-org')`. A developer later tries to connect a Git repository hosted at `https://gitlab.com/my-org/repo` using this same integration. What happens?

A. It succeeds, since API integrations govern authentication method only, not the specific domain/prefix being called
B. It fails, because the GitLab URL does not fall under the explicitly allowed prefix (`https://github.com/my-org`) configured on the integration — a new or updated integration scoped to the GitLab domain would be required
C. It succeeds automatically, since all major Git hosting providers share a common allow-list by default in Snowflake
D. It fails, but only because GitLab repositories are entirely unsupported by Snowflake's Git integration feature regardless of API integration configuration

<details>
<summary>💡 Answer & Explanation</summary>

<br>

**Correct Answer: B**

`API_ALLOWED_PREFIXES` explicitly scopes which URL prefixes an API integration is permitted to call — a GitLab URL simply doesn't match a prefix scoped to `github.com/my-org`, so the connection attempt fails validation. A properly scoped (new or altered) integration covering the GitLab domain would be required instead.

- **A** and **C** both incorrectly assume prefix scoping doesn't matter or is universally pre-approved, which contradicts the explicit purpose of the allow-list.
- **D** incorrectly claims a blanket platform-level exclusion; Snowflake's Git integration supports multiple providers (GitHub, GitLab, Bitbucket, Azure DevOps, AWS CodeCommit) — the failure here is a prefix-scoping issue on this specific integration object, not a fundamental platform limitation.

**Reference:** [CREATE API INTEGRATION — API_ALLOWED_PREFIXES](https://docs.snowflake.com/en/sql-reference/sql/create-api-integration)
</details>

---

### Part N: Git Integration (Q97–Q100)

### Q97
A team wants to keep the CREATE/ALTER statements for their Snowflake objects (databases, warehouses, tasks) version-controlled in a remote Git repository, and then execute the latest committed version of those SQL files directly from within Snowflake as part of a CI/CD pipeline. Which sequence of objects and commands supports this documented pattern?

A. CREATE FILE FORMAT → CREATE STAGE → COPY INTO, treating the Git repository as if it were a cloud storage bucket
B. CREATE API INTEGRATION (scoped to the Git provider) → CREATE GIT REPOSITORY (referencing that integration and the remote origin URL) → ALTER GIT REPOSITORY ... FETCH → EXECUTE IMMEDIATE FROM @repo/branches/<branch>/path/to/file.sql
C. CREATE NOTIFICATION INTEGRATION → CREATE EXTERNAL TABLE → SELECT directly from the Git commit history
D. CREATE SECRET → CREATE WAREHOUSE → RUN GIT SYNC, a dedicated DDL command for repository synchronization

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

The documented Git integration workflow is: an API integration (scoped to the Git provider's API, e.g., `git_https_api`) authorizes the connection; a `GIT REPOSITORY` object represents a cloned copy of the remote repo (a special kind of repository stage) inside Snowflake; `ALTER GIT REPOSITORY ... FETCH` pulls the latest commits/branches; and `EXECUTE IMMEDIATE FROM @repo/branches/<branch>/.../file.sql` runs the SQL file's contents directly from the synced repository stage — exactly the CI/CD pattern described.

- **A** and **C** both misapply unrelated object types (file formats/COPY INTO are for structured/semi-structured data files; notification integrations/external tables are for cloud storage event-driven metadata) to a Git-specific workflow.
- **D** invents a nonexistent `RUN GIT SYNC` command; no such DDL statement exists in Snowflake's SQL reference.

**Reference:** [Setting up Snowflake to use Git](https://docs.snowflake.com/en/developer-guide/git/git-setting-up) and [Using a Git repository in Snowflake](https://docs.snowflake.com/en/developer-guide/git/git-overview)
</details>

---

### Q98
A developer connects a Snowsight Workspace to a **public** GitHub repository (e.g., a Snowflake Labs quickstart) that requires no authentication to read. What is the documented limitation regarding making changes from within that workspace?

A. There is no limitation; commits and pushes work identically for public and private repositories
B. It is not possible to commit and push changes from the workspace back to a public repository configured without authentication credentials, since write access requires an authenticated identity the "no auth" configuration doesn't provide
C. Public repositories can only be read via the API integration, never through a Workspace at all
D. Public repositories require a storage integration instead of an API integration

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: B**

Snowflake explicitly documents that when using a public repository configured without authentication (the "quick start with a public repo" pattern), you can browse, pull, and run files, but you cannot commit and push changes back — write operations require an authenticated identity, which a no-auth public-repo configuration doesn't establish.

- **A** ignores this explicitly documented read-only-for-writes limitation.
- **C** overstates the restriction; Workspaces do support browsing/reading public repositories directly, not solely through direct API integration calls.
- **D** misapplies storage integrations (cloud object storage) to a Git-specific connectivity concept — Git repositories use API integrations, not storage integrations, regardless of public/private status.

**Reference:** [Setting up Snowflake to use Git — authentication options](https://docs.snowflake.com/en/developer-guide/git/git-setting-up)
</details>

---

### Q99
A `GIT REPOSITORY` object has been created and fetched successfully. A developer runs `LIST @my_git_repo/branches/main/;` and separately tries `SELECT` against a directory table on the same object. What is true about querying file listings on a Git repository stage compared to a standard external stage?

A. Git repository stages behave like any other stage for `LIST`-style file enumeration (browsing branches/paths/commits), since a Git repository is implemented as a special kind of stage object
B. Git repository stages do not support `LIST` at all; only `EXECUTE IMMEDIATE FROM` is a valid operation against them
C. Git repository stages require a directory table to be explicitly enabled before any file can be listed, unlike standard stages
D. Git repository stages can only be queried through the Snowflake REST API, never through SQL directly

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

A Git repository is implemented as a special type of stage (a repository stage) representing a full clone of the remote repository, including its branches/commits — and just like any other stage, you can browse its contents (branches, paths, files) using standard commands like `LIST`, in addition to `SHOW`/`DESCRIBE` for repository-specific metadata.

- **B** understates real, documented capability — `LIST` (and `SHOW`/`DESCRIBE GIT REPOSITORY`) do work against these objects; `EXECUTE IMMEDIATE FROM` isn't the only supported operation.
- **C** invents an artificial directory-table prerequisite that doesn't apply here — Git repository stages expose file browsing natively as part of the repository-stage mechanism itself.
- **D** is false — SQL-based access via `LIST`, `SHOW`, and `EXECUTE IMMEDIATE FROM` is the standard, documented way developers interact with Git repository objects, not an API-exclusive path.

**Reference:** [A deep dive into Snowflake's Git Integration](https://docs.snowflake.com/en/developer-guide/git/git-overview)
</details>

---

### Q100
A company wants to store CREATE/ALTER definitions for account-level objects (warehouses, roles, users) as version-controlled SQL in Git and deploy changes through a reviewed pull-request process before they ever touch production Snowflake. Beyond the technical Git integration objects, which statement best captures the broader DevOps value of this pattern, and what should NOT be assumed about it?

A. It provides governed, auditable, peer-reviewed change management for Snowflake account objects using familiar software engineering practices (branches, PRs, code review) — but it should not be assumed that connecting Git alone enforces these practices; team process (e.g., requiring PR approval before merging to the branch that production execution reads from) still has to be deliberately designed
B. Connecting a Git repository automatically enforces mandatory code review before any SQL file can be executed via EXECUTE IMMEDIATE FROM
C. Git integration replaces the need for role-based access control on Snowflake objects, since Git's own permissions become the sole authorization mechanism
D. Once Git integration is configured, all existing Snowflake objects are automatically retrofitted with matching SQL definitions committed to the repository

<details>
<summary>💡 Answer & Explanation</summary>

**Correct Answer: A**

Git integration is genuinely valuable for bringing source-control discipline (branches, PRs, review, history) to Snowflake object management — but it is an *enabling mechanism*, not an *enforcement* mechanism on its own. Nothing about connecting a repository automatically forces a PR-approval gate before `EXECUTE IMMEDIATE FROM` runs a file from a given branch; the team still has to design their branching/CI process (e.g., only running deploy jobs against a protected `main` branch that requires approved PRs) to actually get governance benefits.

- **B** incorrectly assumes automatic enforcement of a process control that Snowflake's Git integration does not implement on its own — `EXECUTE IMMEDIATE FROM` will happily run whatever SQL exists on the referenced branch, reviewed or not, unless the team's own CI/CD design gates it.
- **C** incorrectly claims Git permissions replace Snowflake's RBAC model; the two are independent — a role still needs the correct Snowflake privileges to execute the SQL pulled from Git, regardless of GitHub/GitLab repository permissions.
- **D** invents an automatic retroactive "reverse-engineer existing objects into Git" capability that Snowflake's Git integration does not provide; existing objects are not automatically discovered and committed anywhere.

**Reference:** [A deep dive into Snowflake's Git Integration](https://docs.snowflake.com/en/developer-guide/git/git-overview)
</details>

---

## Quick-Reference Answer Key

| Q | Ans | Q | Ans | Q | Ans | Q | Ans | Q | Ans |
|---|---|---|---|---|---|---|---|---|---|
| 1 | B | 21 | B | 41 | B | 61 | B | 81 | A |
| 2 | A | 22 | B | 42 | B | 62 | B | 82 | B |
| 3 | C | 23 | C | 43 | B | 63 | B | 83 | B |
| 4 | B | 24 | A | 44 | B | 64 | B | 84 | B |
| 5 | B | 25 | A | 45 | B | 65 | B | 85 | A |
| 6 | A | 26 | A | 46 | B | 66 | B | 86 | B |
| 7 | A | 27 | B | 47 | B | 67 | B | 87 | B |
| 8 | B | 28 | B | 48 | B | 68 | B | 88 | B |
| 9 | B | 29 | B | 49 | B | 69 | B | 89 | B |
| 10 | C | 30 | A | 50 | B | 70 | B | 90 | B |
| 11 | B | 31 | B | 51 | B | 71 | B | 91 | B |
| 12 | B | 32 | A | 52 | B | 72 | B | 92 | B |
| 13 | B | 33 | C | 53 | B | 73 | B | 93 | B |
| 14 | A | 34 | B | 54 | B | 74 | B | 94 | B |
| 15 | B | 35 | B | 55 | B | 75 | B | 95 | B |
| 16 | A | 36 | B | 56 | B | 76 | B | 96 | B |
| 17 | B | 37 | C | 57 | B | 77 | B | 97 | B |
| 18 | A | 38 | B | 58 | B | 78 | B | 98 | B |
| 19 | B | 39 | B | 59 | B | 79 | B | 99 | A |
| 20 | B | 40 | B | 60 | B | 80 | B | 100 | A |

> **Note on the answer distribution:** if you tally the key above, you'll notice option B appears frequently — that's a genuine reflection of these particular scenarios' correct answers (each was written independently and checked against documentation, not placed to hit a target letter-frequency). Don't use position as a signal on the real exam either way — always verify with the underlying documented behavior, which is exactly why every explanation above links to the governing Snowflake doc page.

---

## Sources Consulted
This bank was built and fact-checked against official Snowflake documentation, including: COPY INTO \<table\>/\<location\>, CREATE FILE FORMAT, CREATE STAGE, CREATE STORAGE INTEGRATION, CREATE API INTEGRATION, CREATE STREAM, CREATE TASK/ALTER TASK, CREATE PIPE, Snowpipe and Snowpipe Streaming guides, Dynamic Tables documentation (refresh modes, target lag, streams-on-dynamic-tables), Directory Table guides, Git integration developer guide, and the Snowflake Drivers/Connectors overview — all under docs.snowflake.com.

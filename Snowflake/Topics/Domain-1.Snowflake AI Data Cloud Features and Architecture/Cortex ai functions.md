# Snowflake Cortex, AI SQL Functions, Cortex Search & Cortex Analyst
### A SnowPro-focused study guide (sourced from official Snowflake documentation)

> **How to use this guide:** Each feature is explained in plain language first, then followed by a **⚠️ Gotchas / Exam Traps** box. Numbers, limits, privilege names, and parameter names are taken directly from Snowflake's documentation (`docs.snowflake.com`) as of **July 2026** — these are exactly the kind of concrete facts SnowPro loves to test. Snowflake ships new AI features constantly, so always sanity-check hard numbers (token limits, row limits, QPS limits) against the live docs before an exam or production deployment, since these values change often.

---

## Table of Contents
1. [Part 1 — Snowflake Cortex: The Big Picture](#part-1)
2. [Part 2 — Cortex AI Functions (AI SQL)](#part-2)
3. [Part 3 — Cortex Search](#part-3)
4. [Part 4 — Cortex Analyst](#part-4)
5. [Part 5 — Cortex Agents](#part-5)
6. [Part 6 — Quick-Reference: Every AI Function & Agent Object](#part-6)
7. [Part 7 — Master Gotchas / Exam Traps Cheat Sheet](#part-7)
8. [Part 8 — Practice Questions](#part-8)

---

<a name="part-1"></a>
## Part 1 — Snowflake Cortex: The Big Picture

**Snowflake Cortex** is Snowflake's umbrella term for the AI/ML capabilities that run *inside* the Snowflake perimeter — no data ever has to leave Snowflake to reach an LLM. It is made up of four broad pillars that SnowPro exams treat as **distinct, individually-nameable features**:

| Pillar | What it does | Interface |
|---|---|---|
| **Cortex AI Functions** (a.k.a. AI SQL / AISQL) | SQL functions (`AI_COMPLETE`, `AI_CLASSIFY`, `AI_SENTIMENT`, etc.) that call LLMs on text, images, audio, and documents, row by row or in aggregate. | SQL, Python (Snowpark/Snowflake ML), REST |
| **Cortex Search** | A managed hybrid (vector + keyword) search/retrieval engine over your text data — the "R" in RAG. | SQL preview function, Python SDK, REST API |
| **Cortex Analyst** | Turns natural-language business questions into SQL against a governed **semantic model / semantic view**, so business users get direct answers without writing SQL. | REST API (embed in Streamlit, Slack, Teams, etc.) |
| **Cortex Agents** | A fully managed **agentic** orchestration layer that plans a task, calls tools (Analyst, Search, code execution, custom tools, MCP, web search), reflects on results, and produces a final answer — combining structured + unstructured reasoning in one governed loop. | Snowsight, REST API, SQL, Snowflake CoWork, Cortex Code |

All Cortex AI Functions are **deployed within the Snowflake Service perimeter** — LLMs from OpenAI, Anthropic, Meta, Mistral AI, DeepSeek, and Google are hosted and served *by Snowflake*, not called out to a third-party API endpoint from your account.

### ⚠️ Gotchas / Exam Traps
- **Cortex ≠ Snowflake ML.** Cortex AI Functions are *generative*/LLM-based (unstructured analytics: text, image, audio, documents). **Snowflake ML** (`SNOWFLAKE.ML.FORECAST`, `SNOWFLAKE.ML.ANOMALY_DETECTION`, `SNOWFLAKE.ML.CLASSIFICATION`, the Model Registry, Feature Store) is *classical/traditional ML* for structured, tabular data. Access to Snowflake ML objects is **not** gated by the `SNOWFLAKE.CORTEX_USER` role — it's controlled by ordinary object-creation privileges (e.g., `CREATE SNOWFLAKE.ML.FORECAST` on a schema). This distinction is a classic SnowPro trap.
- **Cortex Analyst is opt-in; most other Cortex features are opt-out.** Most Cortex features are available to all users by default via the `PUBLIC` role. Cortex Analyst specifically requires explicit access (it is not accessible to users by default in the same automatic way, and can also be globally disabled with `ALTER ACCOUNT SET ENABLE_CORTEX_ANALYST = FALSE`).
- Snowflake **Copilot** (legacy) is a *separate* feature gated by its own `SNOWFLAKE.COPILOT_USER` database role — don't confuse it with Cortex Analyst or Cortex Code.
- **Snowflake Intelligence** is not a separate product — it's essentially the combination of **Cortex Agents + Cortex Analyst + Cortex Search + Cortex AI Functions** working together in a governed Snowflake environment, surfaced as a conversational experience in Snowsight.

---

<a name="part-2"></a>
## Part 2 — Cortex AI Functions (AI SQL)

### 2.1 Access control — the privilege model (high-yield exam topic)

To call **any** Cortex AI Function, a role needs **both** of the following (an AND relationship, two independent locks on the same door):

1. **An account-level privilege**: either the blanket `USE AI FUNCTIONS` privilege, **or** a narrower per-function `USE AI FUNCTION <name>` privilege (e.g., `USE AI FUNCTION AI_COMPLETE`). If a role has the blanket privilege, it can call every AI function regardless of any per-function grants/revokes — the two privilege types have an **OR relationship** with each other.
2. **A database role**: either `SNOWFLAKE.CORTEX_USER` (broad — grants access to all "Covered AI Features," including Cortex Search, Cortex Analyst orchestration pieces, etc.) or the narrower `SNOWFLAKE.AI_FUNCTIONS_USER` (scoped to just the scalar AI functions — **no** Cortex Agent, Analyst, or Search access).

By default, `USE AI FUNCTIONS` **and** `CORTEX_USER` are both granted to the `PUBLIC` role — meaning every user in a Snowflake account can call Cortex AI Functions out of the box unless an admin revokes this.

**Special carve-out:** If a role has `USE AI FUNCTIONS` but **lacks** `CORTEX_USER`, it can *still* call `AI_AGG` and `AI_SUMMARIZE_AGG` — an explicitly documented exception worth remembering.

Other important database roles:
| Role | Grants access to |
|---|---|
| `SNOWFLAKE.CORTEX_USER` | All Covered AI Features (broadest) |
| `SNOWFLAKE.AI_FUNCTIONS_USER` | Scalar AI functions only (not Agent/Analyst/Search) |
| `SNOWFLAKE.CORTEX_EMBED_USER` | Just the embedding functions (`AI_EMBED`, `EMBED_TEXT_768/1024`) — useful for isolating embedding-only workloads |
| `SNOWFLAKE.CORTEX_ANALYST_USER` | Cortex Analyst only (not the rest of Cortex) |
| `SNOWFLAKE.CORTEX_AGENT_USER` | Cortex Agents only |
| `SNOWFLAKE.CORTEX_REST_API_USER` | The generic Cortex REST API (Messages/Chat Completions) only — not AI Functions, Search, Analyst, Agents, or Fine-tuning |
| `SNOWFLAKE.COPILOT_USER` | Snowflake Copilot (legacy) / Cortex Code |

None of these SNOWFLAKE-database roles can be granted **directly to a user** — they must be granted to a custom account role first, and that role is then granted to the user (`GRANT DATABASE ROLE SNOWFLAKE.CORTEX_USER TO ROLE my_role; GRANT ROLE my_role TO USER my_user;`).

### ⚠️ Gotchas / Exam Traps — Privileges
- **Two-lock model:** missing *either* the account privilege or the database role → the call fails. Exam questions often give you a role with only one of the two and ask "will this succeed?" — the answer is no.
- **Native Apps loophole:** `USE AI FUNCTIONS` currently does **not** apply to AI Function calls running *inside a Snowflake Native App* — such a query succeeds regardless of the calling role's privilege.
- **Revoke order matters:** if a role already has `CORTEX_USER`, you must explicitly `REVOKE DATABASE ROLE SNOWFLAKE.CORTEX_USER` before a narrower role like `CORTEX_ANALYST_USER` will actually take effect as a restriction (the broader grant otherwise still allows access).
- Even `ACCOUNTADMIN` can be blocked from a feature by an **account parameter** (e.g., `ENABLE_CORTEX_ANALYST = FALSE`) — RBAC isn't the only gate; some features are gated by session/account parameters that any ACCOUNTADMIN can flip back.
- Model-level RBAC (restricting *which LLMs* a role may use) is applied **globally** — you cannot restrict a model to "only Cortex Analyst" or "only AI_COMPLETE"; the restriction applies to every Cortex feature that could use that model.

### 2.2 The two function categories

Cortex AI Functions are grouped into:
- **Task-specific AI functions** — purpose-built, managed functions for a specific job (classification, translation, summarization, extraction, etc.) that need no custom prompt engineering.
- **Helper functions** — small utility functions (`TO_FILE`, `AI_COUNT_TOKENS`, `PROMPT`) that reduce failures in the task-specific functions, e.g., by pre-checking token counts or building well-formed prompt objects.

### 2.3 Legacy vs. current function names (important for exam questions using older names)

Snowflake rebranded most Cortex LLM functions from `SNOWFLAKE.CORTEX.<name>` to a flat `AI_<NAME>` naming convention through 2025. **Both names may appear on the exam.** The legacy functions still work (for backward compatibility) but are slated for eventual deprecation — Snowflake has stated legacy functions like `COMPLETE` and `CLASSIFY_TEXT` **will be deprecated by the end of 2026**.

| Legacy name (`SNOWFLAKE.CORTEX.*`) | Current canonical name |
|---|---|
| `COMPLETE` | `AI_COMPLETE` |
| `CLASSIFY_TEXT` | `AI_CLASSIFY` |
| `EXTRACT_ANSWER` | `AI_EXTRACT` |
| `SENTIMENT` / `ENTITY_SENTIMENT` | `AI_SENTIMENT` |
| `TRANSLATE` | `AI_TRANSLATE` |
| `PARSE_DOCUMENT` | `AI_PARSE_DOCUMENT` |
| `EMBED_TEXT_768` / `EMBED_TEXT_1024` | `AI_EMBED` |
| `SUMMARIZE` | *(still current — no `AI_SUMMARIZE` scalar equivalent; see `AI_AGG`/`AI_SUMMARIZE_AGG` for the aggregate versions)* |

> **Exam trap:** `SUMMARIZE (SNOWFLAKE.CORTEX)` is a *row-level* scalar function (one text in → one summary out) and is **still GA/current**, not deprecated. It is capped at **32,000 input tokens and 4,096 output tokens**. Don't confuse it with `AI_SUMMARIZE_AGG`, which is an *aggregate* function across many rows/a `GROUP BY` and explicitly is **not** subject to context-window limitations because it processes data in chunks internally.

### 2.4 Complete function inventory

| Function | Purpose | Notes |
|---|---|---|
| `AI_COMPLETE` | General-purpose LLM completion from a text prompt and/or image/document input; the workhorse for anything not covered by a task-specific function. | Supports single string, single image, or a multi-file "prompt object" built with `PROMPT()`. Supports `response_format` for structured JSON output. |
| `AI_CLASSIFY` | Classifies text, image, or documents into user-supplied categories (single- or multi-label). | 2–100 categories; case-sensitive; supports `task_description` and few-shot `examples`. |
| `AI_FILTER` | Returns `TRUE`/`FALSE` for a text/image input against a natural-language predicate — designed to be dropped straight into `WHERE`, `SELECT`, or `JOIN … ON`. | Has a built-in query-optimization path (2–10× faster, up to 60% fewer tokens) triggered automatically on qualifying query shapes. |
| `AI_AGG` | Aggregate function: reduces a whole **column** of text (optionally `GROUP BY`) into one result per group, driven by a free-form natural-language instruction. | Not limited by a single model's context window — handles arbitrarily large datasets. |
| `AI_SUMMARIZE_AGG` | Aggregate function: general-purpose summary of a text column across many rows. | Also not limited by context window. Use `AI_AGG` when you need a *specific* shape of output; use this for a generic summary. |
| `AI_EMBED` | Generates a vector embedding (`VECTOR` type) from text or an image, for use in similarity search/clustering/classification. | Successor to `EMBED_TEXT_768`/`EMBED_TEXT_1024`. |
| `AI_SIMILARITY` | Computes cosine similarity (range **−1 to 1**) directly between two raw text or image inputs, without you having to pre-compute embeddings yourself. | Billed under the `AI_EMBED` line item in usage views. Cannot compare text to image directly (must be same modality). |
| `AI_EXTRACT` | Extracts structured information (answers to questions, or a JSON schema) from text, images, or documents; supports multiple languages. | Successor to `EXTRACT_ANSWER`. Optional `scores` parameter returns a 0–1 confidence score per extracted field. |
| `AI_SENTIMENT` | Overall **and** aspect/entity-based sentiment analysis. | Successor to `SENTIMENT`/`ENTITY_SENTIMENT`. Trained/optimized for ~2,048 input tokens (~1,600 words); supports English, French, German, Hindi, Italian, Spanish, Portuguese. |
| `AI_TRANSLATE` | Translates text between supported languages. | Successor to `TRANSLATE`. |
| `AI_TRANSCRIBE` | Speech-to-text for audio/video files on a stage; optional word- or speaker-level timestamps (diarization). | Max file size 700 MB; max duration 120 min (60 min if timestamps requested); billed at 50 tokens/sec of audio with a 1-minute (3,000-token) minimum. |
| `AI_PARSE_DOCUMENT` | Converts documents (scanned or digital) into text or structured Markdown, optionally extracting embedded images. | Two modes: `OCR` (fast plain text) and `LAYOUT` (preserves headers/tables/reading order; required for image extraction). Incompatible with custom network policies. |
| `AI_REDACT` | Detects and redacts PII from text (`redact` mode, default) or just reports PII locations (`detect` mode) for a custom redaction workflow. | US PII plus some UK/Canadian categories only. Input+output capped at **4,096 tokens total**, output capped at **1,024 tokens**. |
| `SUMMARIZE (SNOWFLAKE.CORTEX)` | Row-level summary of a single text value. | Still GA/current (not an "AI_" function). 32,000 input / 4,096 output token cap. |

**Helper functions:**

| Function | Purpose |
|---|---|
| `TO_FILE` | Builds a `FILE` reference to a staged file for use in `AI_COMPLETE` and other file-accepting functions. |
| `AI_COUNT_TOKENS` | Returns the token count of an input string for a given model/function, to preempt context-window failures. |
| `PROMPT` | Builds a templated prompt object with numbered placeholders (`{0}`, `{1}`, …) that can mix strings and `FILE` objects — used heavily with `AI_FILTER`/`AI_COMPLETE` for multi-column or multi-file prompts. |

### ⚠️ Gotchas / Exam Traps — Function inventory
- **`AI_AGG` vs `AI_SUMMARIZE_AGG`**: both are aggregate, both bypass context-window limits — the difference is *control*. `AI_SUMMARIZE_AGG` gives a generic summary with zero configuration; `AI_AGG` takes a natural-language instruction so you can shape the output (e.g., "list only the negative themes as bullet points").
- **Row-level vs. aggregate summarizers are easy to mix up on the exam**: `SUMMARIZE`/`AI_COMPLETE` operate **per row** and *are* bound by the model's context window; `AI_AGG`/`AI_SUMMARIZE_AGG` operate **across rows** and are explicitly *not* bound by a single context window because Snowflake internally chunks/map-reduces the data.
- **`AI_FILTER` and `AI_CLASSIFY` return `NULL` on failure by default** (not an error) — a multi-row query with a few bad rows still completes; only rows that error return `NULL`. Passing `return_error_details => TRUE` instead returns an object with a `value`/`error` field so you can distinguish "the model said no" from "the call failed."
- **`AI_REDACT`'s default error behavior is the opposite**: by default, if it can't process the input, the **entire query fails** (not just that row) — unless you set the session parameter `AI_SQL_ERROR_HANDLING_USE_FAIL_ON_ERROR = FALSE`, at which point per-row `NULL`/error-object behavior kicks in like the other functions. This asymmetry between `AI_REDACT` and functions like `AI_FILTER`/`AI_CLASSIFY` is a subtle, testable detail.
- **`AI_SIMILARITY` cannot compare a text input against an image input** — both sides must be the same modality.
- **`AI_PARSE_DOCUMENT`'s two modes are not interchangeable**: `OCR` mode is faster/cheaper but throws away layout; `LAYOUT` mode is required if you want embedded images extracted, and is generally the recommended default for anything with tables or multi-column layout.
- **Snowflake AI functions cannot operate on `FILE`s from certain stage encryption types**: internal stages with `ENCRYPTION = TYPE = 'SNOWFLAKE_FULL'`, and *any* customer-managed-key-encrypted external stage, are **not** supported inputs to AI functions that accept files.
- **Categories in `AI_CLASSIFY` are capped in practice, not in theory**: the array must have at least 2 and at most 100 unique values technically, but Snowflake's own guidance warns that classification accuracy tends to *degrade* past roughly 20 categories even though the hard limit is much higher.
- **`AI_EXTRACT` has two distinct code paths**: the modern one (text/image/document + natural-language `responseFormat`) and a **legacy Document AI model path** (`AI_EXTRACT(model => 'my_db.my_schema.my_model', file => ...)`) that runs a previously-trained Arctic-TILT Document AI model instead of a general LLM. Confidence scores are **not** supported on the legacy Document AI path.
- **Document AI (the older, GUI-driven, train-your-own-model feature) is being decommissioned** — the `model_build_name!PREDICT` method is already decommissioned. `AI_EXTRACT` is the direct replacement and needs no training step.

### 2.5 Cost & performance considerations

- Cortex AI Functions are **optimized for throughput / batch processing** — Snowflake explicitly recommends them for processing large numbers of rows in SQL tables. For **low-latency, interactive** use cases, Snowflake instead recommends the **REST API** (Complete API for inference, Embed API for embeddings, Agents API for agentic apps).
- Cost is driven by **tokens**, both input and output, and varies by model size (small models are cheaper per token than large ones).
- Every extra label, description, or few-shot example you add to `AI_CLASSIFY`/`CLASSIFY_TEXT` increases the input token count — and therefore the cost — of every single call.
- `AI_TRANSCRIBE` bills strictly by audio duration (50 tokens/second, ~180,000 tokens/hour) — language and timestamp granularity do **not** change the token rate — with a 1-minute minimum bill per file, which is why batching many tiny audio clips into one file is a documented cost optimization.
- Monitor usage via `SNOWFLAKE.ACCOUNT_USAGE.CORTEX_FUNCTIONS_USAGE_HISTORY` / `CORTEX_FUNCTIONS_QUERY_USAGE_HISTORY`, and Cortex Search's dedicated `CORTEX_SEARCH_DAILY_USAGE_HISTORY` view.

### 2.6 Regional availability & cross-region inference

Cortex AI Functions (and Search/Analyst/Agents) are only natively available in **select Snowflake regions**. If a model or feature isn't available in your account's home region, you can opt in to **cross-region inference** so requests are transparently routed to a supported region:

```sql
ALTER ACCOUNT SET CORTEX_ENABLED_CROSS_REGION = 'ANY_REGION';
```

### ⚠️ Gotchas / Exam Traps — Cost & Regions
- If you see a `model not found`–style error, the *first* thing to check is whether the model is available in the account's region — not whether the privilege/role setup is wrong.
- Cross-region inference is **not supported in U.S. SnowGov regions** — a specific, testable exception.
- During cross-region inference, if both the source and destination regions are on the **same cloud provider** (e.g., both AWS), traffic stays inside that provider's global backbone network and is automatically encrypted at the physical layer — Snowflake explicitly does not send it over the public internet.
- Snowflake states that user inputs, prompts, and outputs are **not stored or cached** during cross-region inference.

---

<a name="part-3"></a>
## Part 3 — Cortex Search

### 3.1 What it is

**Cortex Search** is Snowflake's managed **hybrid search** engine: low-latency "fuzzy" search over text data that blends:
1. **Vector search** — semantic/meaning-based retrieval (embeddings).
2. **Keyword search** — exact/lexical matching.
3. **Semantic reranking** — a final reranking pass across the combined candidate set.

This hybrid approach means Cortex Search doesn't require you to manage embeddings, infrastructure, index refreshes, or search-quality tuning yourself — "search quality out of the box." The two primary use cases are:
- **RAG (Retrieval-Augmented Generation)** engine that feeds relevant, up-to-date document snippets into an LLM prompt.
- **Enterprise search** — powering a plain search bar in an application.
- It is also the standard **retrieval tool used by Cortex Agents** for unstructured data.

### 3.2 Creating a service

```sql
CREATE OR REPLACE CORTEX SEARCH SERVICE transcript_search_service
  ON transcript_text
  ATTRIBUTES region
  WAREHOUSE = cortex_search_wh
  TARGET_LAG = '1 day'
  EMBEDDING_MODEL = 'snowflake-arctic-embed-l-v2.0'
  AS (
    SELECT transcript_text, region, agent_id
    FROM support_transcripts
);
```

Key clauses:
- `ON <column>` — the single text column to search (single-index service).
- `TEXT INDEXES` / `VECTOR INDEXES` — for **multi-index** services (see §3.6).
- `ATTRIBUTES` — extra columns returned/filterable alongside search results; must be present in the source query (explicitly or via `*`).
- `WAREHOUSE` — used to run the source query and (re)build/refresh the index. Snowflake recommends a **dedicated warehouse no larger than MEDIUM** per service.
- `TARGET_LAG` — how stale the service is allowed to get vs. the base table before it's refreshed.
- `PRIMARY KEY (col, ...)` — optional, TEXT-typed column(s) uniquely identifying each row; enables an **optimized incremental refresh path**.
- `EMBEDDING_MODEL` — **cannot be changed after creation** — you must `CREATE OR REPLACE` to switch models.

### 3.3 Embedding models

| Model | Dimensions | Context window | Language support |
|---|---|---|---|
| `snowflake-arctic-embed-m-v1.5` **(default)** | 768 | 512 tokens | English only |
| `snowflake-arctic-embed-l-v2.0` | 1024 | 512 tokens | Multilingual |
| `snowflake-arctic-embed-l-v2.0-8k` | 1024 | 8,192 tokens | Multilingual |
| `voyage-multilingual-2` | 1024 | 32,000 tokens | Multilingual |

- The **default embedding model is English-only** — a common trap if the source question mentions non-English data and asks "will search quality be good with the default configuration?"
- Not every model is available in every region — check regional availability before assuming a model choice will work.
- During both indexing **and** query time, any text beyond the model's context window is **truncated for the vector/semantic pass** — but the **full text is still used for keyword search**. This means a document longer than 512 tokens can still be found by exact keyword, even though its "meaning" past token 512 is invisible to the semantic matcher.
- Snowflake's own research recommends **chunking source text into ≤512-token pieces** for the best retrieval quality — bigger isn't automatically better, even with an 8k/32k-context model available.

### 3.4 Refreshes & primary keys

- Cortex Search services **behave like Dynamic Tables** for refresh purposes — the source query must be a candidate for **dynamic-table incremental refresh** (this restriction exists specifically to prevent runaway embedding costs from unbounded re-computation).
- Without a primary key, every refresh **may need to re-evaluate the full result set**. With a `PRIMARY KEY`, refreshes become **incremental** — only changed/added rows are re-embedded — dramatically cutting cost and latency. Primary key columns must be **TEXT** type, and duplicate key values are silently **ignored** in the resulting index.
- Even incremental (primary-keyed) services **periodically run a full index rebuild** to compact fragmented segments; you can influence (not strictly guarantee) the cadence with `FULL_INDEX_BUILD_INTERVAL_DAYS` — Snowflake explicitly documents this as a **soft target**, not a hard schedule. A full rebuild does **not** re-embed unchanged data.

### 3.5 Suspension behavior

A service has **two independently controllable states**: **indexing** and **serving**.
- **Indexing auto-suspends** after **5 consecutive refresh failures** on the source query (mirrors Dynamic Table behavior) — check `INDEXING_STATE`/`INDEXING_ERROR` via `DESCRIBE CORTEX SEARCH SERVICE` or the `CORTEX_SEARCH_SERVICES` view, fix the root cause, then `ALTER ... RESUME INDEXING`.
- **Serving can auto-suspend on inactivity** (Preview) — configurable via `AUTO_SUSPEND` (minimum **1,800 seconds / 30 minutes**). The service auto-resumes on the next query, but resuming may take a couple of minutes, and any *concurrent* requests that arrive during that resume window get an HTTP **429** — client code needs retry/backoff logic to handle this gracefully.

### 3.6 Multi-index search & user-provided embeddings

Multi-index Cortex Search lets you mix:
- **Text indexes** — best for exact/fuzzy keyword fields (SKUs, names, categories).
- **Vector indexes** — best for long, semantically rich fields (descriptions, reviews).
- **User-provided vector embeddings** — precomputed vectors from *any* model (open-source, commercial, or custom) loaded straight into a `VECTOR`-typed column. These are indexed as-is with **no additional embedding cost**.

Two query-time modes for user-provided vectors:
| Mode | Index time | Query time |
|---|---|---|
| Fully user-managed | Vectors supplied in a `VECTOR` column | You must also supply a query **vector** |
| User-managed + managed query embeddings | Vectors supplied in a `VECTOR` column | You can supply **query text**, and Cortex Search embeds it with a specified model |

> You **cannot** `INSERT` a raw array directly as a vector — you must explicitly `CAST` the array to the `VECTOR` type.

### 3.7 Known limitations (frequently tested)

| Limit | Value |
|---|---|
| Max rows in the materialized source query | **100 million** (creation fails above this without contacting Snowflake) |
| Max sustained query throughput per service | **20 QPS**; across all services in an account, **140 QPS** (contact Snowflake to raise) |
| Response truncation for `SEARCH_PREVIEW` | Truncates results over **300 KB**; the REST API allows up to **10 MB** |
| Cloning | **Not supported** for Cortex Search Services |
| `DATA_RETENTION_TIME_IN_DAYS` | Cannot be set to **0** on base tables/schema/database backing a service |
| Table mutability while running | Base tables must not be altered/dropped while the service is running — **stop the service first** |
| Source query constructs | Must satisfy the same restrictions as **Dynamic Tables** (no non-deterministic functions, etc.) |

### 3.8 Cost components

Cortex Search bills across **five** categories: virtual-warehouse compute (only if there's new data to refresh — zero-change refreshes cost nothing), `EMBED_TEXT` token compute for new/changed rows, multi-index-specific embedding costs, **serving compute** billed per **GB/month of indexed (uncompressed) data** (charged even while idle, as long as the service is available to serve), storage, and cloud-services compute.

### 3.9 Privileges

To **create** a service: `SNOWFLAKE.CORTEX_USER` or `SNOWFLAKE.CORTEX_EMBED_USER`, plus `CREATE CORTEX SEARCH SERVICE` (or `OWNERSHIP`) on the schema, `SELECT` on the underlying tables, and `USAGE` on the refresh warehouse. **Change tracking must be enabled** on all underlying objects.

To **query** a service, the role needs `USAGE` on the service **and** its database/schema — Cortex Search services execute with **owner's rights** (like a stored procedure), following the same security model.

### ⚠️ Gotchas / Exam Traps — Cortex Search
- **100M row cap** and **20 QPS single-service throughput** are the two numeric limits most likely to show up verbatim in a question.
- **Owner's rights execution** is a subtlety: the querying role only needs `USAGE` on the service itself — it does **not** need `SELECT` on the underlying base table to get results back, because the service runs with the *creator's* privileges, not the querier's.
- The **embedding model cannot be swapped after creation** — this is a one-way door; changing models requires `CREATE OR REPLACE`.
- **512-token semantic truncation but full-text keyword matching** — a favorite "trick" fact: a long unchunked document *can* still surface via keyword search even though its later content is invisible to vector search.
- Services **cannot be cloned** — if a question asks about database cloning strategies that include a Cortex Search Service, the correct answer flags this as unsupported.
- Auto-suspend's **minimum threshold is 30 minutes**, and a manually-resumed service needs another **full** `AUTO_SUSPEND` idle period before it can auto-suspend again — it doesn't just resume the countdown from where it left off.

---

<a name="part-4"></a>
## Part 4 — Cortex Analyst

### 4.1 What it is

**Cortex Analyst** is a fully-managed, LLM-powered **text-to-SQL** service for **structured** data. A business user asks a plain-English question; Cortex Analyst generates and can execute a SQL query against your governed Snowflake tables and returns a direct answer — no RAG pipeline, model experimentation, or GPU capacity planning required on your part. It's exposed as a **REST API**, meant to be embedded into chat UIs, Streamlit apps, Slack, Teams, etc.

The core insight behind Cortex Analyst: handing an LLM a raw database schema alone produces poor text-to-SQL results, because schemas lack business context (what "revenue" means, how "last month" should be calculated, which abbreviation maps to which real-world concept). Cortex Analyst closes this gap with a **semantic model**.

### 4.2 Semantic Views (current) vs. legacy semantic model YAML

- **Semantic Views** are the **recommended, current approach**: schema-level Snowflake objects (`CREATE SEMANTIC VIEW`) that define **logical tables**, **dimensions**, **facts** (row-level quantities — formerly called "measures"), **metrics** (aggregated business KPIs), and **relationships** (join paths). They get full native RBAC, can be shared via Snowflake's normal sharing mechanisms (private/organizational/Marketplace listings), and can even be queried directly in a plain `SELECT`.
- **Legacy semantic model YAML files** (uploaded to a stage, referenced by path) are still supported for **backward compatibility only** — new implementations should use Semantic Views.
- Facts were formerly called **measures** — the terms are backward-compatible synonyms, a classic renaming trap.

### 4.3 Access control

To call Cortex Analyst, a role needs **either**:
- `SNOWFLAKE.CORTEX_USER` (broad — all Covered AI Features), **or**
- `SNOWFLAKE.CORTEX_ANALYST_USER` (narrow — Cortex Analyst *only*).

Plus, for the semantic model/view itself: `READ`/`WRITE` on the stage (if using legacy YAML-on-stage), `USAGE` on any Cortex Search services referenced inside the semantic model, and `SELECT` on the underlying tables.

> **Important stage-sharing gotcha:** if a semantic model YAML is stored on a stage, **anyone with access to that stage can use the model** — even if they personally lack `SELECT` on the underlying tables the model is built from. Snowflake explicitly warns: ensure every role with stage access also has `SELECT` on every table referenced by every model on that stage, or you can create an unintended data-access path.

### 4.4 Multi-turn conversations

Cortex Analyst supports follow-up questions by passing the full conversation history (`messages` array of `role: "user"` / `role: "analyst"` turns) with every request.

**Documented limitations of multi-turn conversations:**
- Cortex Analyst has **no access to the results of previous SQL queries** — e.g., after "What are my products?", asking "What's the revenue of the *second* product?" fails because Analyst never saw the *rows* returned, only the SQL and text summary.
- Cortex Analyst can only answer questions **resolvable via SQL** — it will not generate freeform "what trends do you see?" business insight.
- Very long or intent-shifting conversations degrade follow-up interpretation — the documented remedy is simply to **reset the conversation**.
- The LLM itself is **stateless between requests** — the *entire* conversation history is reprocessed (and re-billed) on every new turn, so cost grows with conversation length.

### 4.5 Model selection

You **cannot pick a specific model** for Cortex Analyst to use directly — Cortex Analyst automatically assigns each request to a model (or model combination) based on: models available in your region, your cross-region inference configuration, and any model-level RBAC restrictions. Documented preference order (most-preferred first, falls back down the list based on access):
1. Anthropic Claude Sonnet 4.6
2. Anthropic Claude Sonnet 4.5
3. OpenAI GPT 4.1
4. Arctic Text2SQL R1.5 (with thinking enabled)
5. A combination of Mistral Large 2 and Llama 3.1 70B

If your role has access to **none** of the models in this chain, the request simply **fails** — disabling models therefore directly increases risk of query failure, it doesn't just "downgrade" quality silently forever.

### 4.6 Cost

Billing is based on the **number of messages processed** (only successful/HTTP 200 responses count) — **not** on token count, *unless* Cortex Analyst is invoked **through Cortex Agents**, in which case token count does affect cost. Separately, the **warehouse cost of actually executing** the generated SQL is billed as normal warehouse compute, on top of the AI cost.

### ⚠️ Gotchas / Exam Traps — Cortex Analyst
- **Facts = measures** (legacy term) — don't be fooled by a question using the old vocabulary.
- **Cortex Analyst never trains on customer data**, and by default runs on Snowflake-hosted models (Mistral/Meta lineage in the fallback chain) so metadata/prompts never leave Snowflake's governance boundary.
- Cortex Analyst is **disabled** entirely with `ALTER ACCOUNT SET ENABLE_CORTEX_ANALYST = FALSE` — a global kill switch independent of RBAC.
- Remember the **stage-access-implies-model-access** subtlety above — it's a realistic "identify the security risk" scenario.
- Billing is **per successful message**, not per token, **except** when accessed via an Agent — a nuance easy to mix up with Cortex Search/Cortex Agents billing (which *are* token/GB-based).

---

<a name="part-5"></a>
## Part 5 — Cortex Agents

### 5.1 What it is

**Cortex Agents** is Snowflake's **fully managed agentic platform**. Where Cortex Analyst answers one structured-data question and Cortex Search retrieves from one unstructured corpus, an **Agent** can autonomously combine both — plus code execution, custom tools, and external systems — into a single multi-step, reasoning workflow, without you building your own orchestration loop, runtime, or sandbox.

### 5.2 Key concepts

| Concept | Description |
|---|---|
| **Agent** | A reusable, schema-level object bundling model choice, tools, orchestration settings, and natural-language instructions. |
| **Tools** | The capabilities available to the agent (see §5.4). |
| **Orchestration** | The LLM-driven plan → use tools → reflect loop; shaped by natural-language planning/response instructions you provide. |
| **Thread** | Persisted conversation context across turns, so the client app doesn't need to manage state itself. |
| **Run** | A single call to `agent:run`; the agent streams events exposing its reasoning, tool calls, and reflections along the way. |

### 5.3 The reasoning loop

Every agent request follows a **Plan → Use tools → Reflect and respond** loop, repeated as many times as needed within a single request:
1. **Plan** — parse the request, disambiguate vague questions, split complex requests into subtasks, and decide which tool handles each subtask.
2. **Use tools** — actually invoke Cortex Analyst, Cortex Search, code execution, or other configured tools.
3. **Reflect and respond** — evaluate tool outputs and decide: ask a clarifying question, call another tool, or produce the final answer.

### 5.4 Tools available to an agent

| Tool | What it does |
|---|---|
| **Cortex Analyst** | Text-to-SQL over structured data via a semantic view. |
| **Cortex Search** | Retrieval over unstructured data; the agent can dynamically adjust filters, columns, result counts, and per-index/time-decay settings based on the query. |
| **Code execution** | Runs Python in a secure, isolated sandbox for calculations/data processing. |
| **Data to Chart** | Auto-generates a Vega-Lite visualization from tool output. |
| **Custom tools** | Stored procedures / UDFs implementing your own business logic or backend calls. |
| **Agent skills** | Packaged, reusable, task-specific bundles of instructions + scripts. |
| **MCP connectors** | Tools hosted on remote **Model Context Protocol** servers (e.g., Atlassian Jira, Salesforce, or your own MCP server). |
| **Web search** | Real-time public internet information — must be explicitly enabled at the **account level** before an agent can use it. |

### 5.5 Models

Cortex Agents supports models from the **Claude (Anthropic), GPT (OpenAI), Grok (SpaceX/xAI), and Gemini (Google)** families. Snowflake's recommendation is to select **`auto`** for the orchestration model so Cortex always uses the current highest-quality option as new models roll out, rather than hand-picking one that may fall behind over time.

### 5.6 The five-step build lifecycle

1. **Create an agent** (Snowsight, SQL, or REST API) — name, model, instructions.
2. **Add tools** — configure each tool's required resources (semantic views, search services, warehouses).
3. **Test the agent** in the Snowsight playground, iterating on tools/instructions.
4. **Integrate** via the `agent:run` REST API using **threads** for conversation context; end users can also reach agents through **Snowflake CoWork** and **Cortex Code**.
5. **Monitor, evaluate, and iterate** — review threads/logs/traces, collect end-user ratings/feedback, and run formal evaluations over the agent's lifecycle.

You don't strictly need a persisted Agent object to experiment — `agent:run` can also be called directly with the full agent configuration inlined in each request.

### 5.7 Access control & known limitations

- Calling an agent requires `SNOWFLAKE.CORTEX_USER` or `SNOWFLAKE.CORTEX_AGENT_USER`, plus privileges on the agent object itself **and** on every object each configured tool touches. Session permissions are derived from the **querying user's default role**.
- **Cortex Agents APIs are not supported from a Streamlit-in-Snowflake (SiS) app running on the warehouse runtime** — you must use a **container runtime** instead to call Agents APIs from SiS.
- Snowflake explicitly cautions that **accuracy of LLM responses and citations is not guaranteed** — always review agent output before serving it to end users; this is a documented, testable disclaimer, not just marketing boilerplate.

### 5.8 Cost

Agents incur charges on **multiple fronts simultaneously**:
- **Orchestration** — billed by tokens used for the plan/reflect loop.
- **Cortex Analyst tool calls** — billed per token (note: this differs from calling Cortex Analyst *standalone*, which is billed per message).
- **Cortex Search tool calls** — billed by index size and how long it has persisted.
- **Custom tools** — ordinary warehouse compute cost, based on warehouse size and runtime.

### ⚠️ Gotchas / Exam Traps — Cortex Agents
- **Agent ≠ Analyst ≠ Search** — a common exam distractor is describing a multi-step, multi-tool scenario ("combine sales figures with contract PDF language") and asking which single feature handles it: the answer is **Cortex Agents**, because Analyst alone only does structured SQL and Search alone only does unstructured retrieval — neither *orchestrates between* the two by itself.
- **Cortex Analyst's billing model changes when it's called through an Agent** (token-based) vs. standalone (message-based) — this cost-model shift is a subtle, testable detail.
- **SiS + warehouse runtime cannot call Agents APIs** — remember "container runtime" as the fix.
- Web search as a tool is **off by default** and must be turned on at the **account** level, not just added to an individual agent's tool list.
- MCP connectors let an agent call **external** systems (Jira, Salesforce, etc.) — this is the mechanism, if a question asks how an agent could integrate with a third-party ticketing or CRM system.

---

<a name="part-6"></a>
## Part 6 — Quick-Reference: Every AI Function & "Agent" Object Worth Knowing for SnowPro

This is the condensed list — if you only memorize one table before the exam, make it this one.

### Generative / task-specific AI SQL functions
| Function | One-line job |
|---|---|
| `AI_COMPLETE` | Free-form LLM completion (text/image/document in, text/structured JSON out) |
| `AI_CLASSIFY` | Single- or multi-label classification against your categories |
| `AI_FILTER` | Natural-language boolean predicate for `WHERE`/`JOIN` |
| `AI_AGG` | Custom-instruction aggregation across rows (no context-window limit) |
| `AI_SUMMARIZE_AGG` | Generic summary aggregation across rows (no context-window limit) |
| `SUMMARIZE` | Generic summary of a single row's text (context-window limited: 32K in / 4K out) |
| `AI_SENTIMENT` | Overall + aspect-based sentiment |
| `AI_TRANSLATE` | Language translation |
| `AI_EXTRACT` | Structured field/table/list extraction from text, image, or document (+ optional confidence scores) |
| `AI_REDACT` | PII detection/redaction |
| `AI_PARSE_DOCUMENT` | Document → text/Markdown (OCR or LAYOUT mode); can pull embedded images |
| `AI_TRANSCRIBE` | Audio/video → text, with optional word/speaker timestamps |
| `AI_EMBED` | Text/image → vector embedding |
| `AI_SIMILARITY` | Cosine similarity between two raw inputs (no manual embedding step) |
| `AI_COUNT_TOKENS` | Token-count helper |
| `PROMPT` | Templated prompt-object builder |
| `TO_FILE` | Stage-file → `FILE` reference builder |

### Legacy names you may still see on the exam
`COMPLETE`, `CLASSIFY_TEXT`, `EXTRACT_ANSWER`, `SENTIMENT`, `ENTITY_SENTIMENT`, `TRANSLATE`, `PARSE_DOCUMENT`, `EMBED_TEXT_768`, `EMBED_TEXT_1024`, `SEARCH_PREVIEW` (still current — used to *test* a Cortex Search Service from SQL, not a production query path).

### The "agent-shaped" objects and platforms
| Object / platform | Category | Role |
|---|---|---|
| **Cortex Search Service** | Retrieval object | Hybrid vector+keyword search over unstructured text; the retrieval half of RAG |
| **Semantic View / semantic model YAML** | Metadata object | Business-terminology layer that Cortex Analyst uses to generate accurate SQL |
| **Cortex Analyst** | Single-purpose service | Text-to-SQL over one semantic view/model per request |
| **Cortex Agent (object)** | Orchestrator | Multi-tool, multi-step reasoning loop combining Analyst + Search + code execution + custom/MCP tools |
| **Cortex Code** | Agentic coding assistant in Snowsight | SQL/dbt development help, diff views, requires `COPILOT_USER` + (`CORTEX_USER` or `CORTEX_AGENT_USER`) |
| **Snowflake CoWork** | Agentic knowledge-work surface | Lets end users interact with Agents directly, outside custom app-building |
| **Snowflake Intelligence** | Composite experience | = Cortex Agents + Cortex Analyst + Cortex Search + Cortex AI Functions, unified in Snowsight |
| **Snowflake Copilot (legacy)** | Predecessor assistant | Distinct `COPILOT_USER` role; being superseded by Cortex Code |

### Related-but-different: Snowflake ML (classical ML, *not* Cortex)
Frequently confused with Cortex on exams because both live under "Snowflake AI & ML":
| Function/Object | Job |
|---|---|
| `SNOWFLAKE.ML.FORECAST` | Time-series forecasting |
| `SNOWFLAKE.ML.ANOMALY_DETECTION` | Anomaly detection on structured data |
| `SNOWFLAKE.ML.CLASSIFICATION` | Classical (non-LLM) classification model |
| Model Registry / Feature Store | MLOps infrastructure for custom Python models |

> **Rule of thumb:** if the question involves an **LLM, text, images, audio, or natural language** → it's **Cortex**. If it involves **classical statistical/ML models on structured numeric data** → it's **Snowflake ML**, and the `CORTEX_USER` role is irrelevant to it.

---

<a name="part-7"></a>
## Part 7 — Master Gotchas / Exam Traps Cheat Sheet

**Access & privileges**
- Two independent locks: account-level privilege (`USE AI FUNCTIONS` or per-function) **AND** database role (`CORTEX_USER` or `AI_FUNCTIONS_USER`).
- `AI_AGG`/`AI_SUMMARIZE_AGG` work with `USE AI FUNCTIONS` alone, even without `CORTEX_USER` — a documented exception.
- SNOWFLAKE database roles can never be granted directly to a user — always via an intermediate account role.
- Native Apps bypass the `USE AI FUNCTIONS` check entirely.
- `ENABLE_CORTEX_ANALYST` account parameter can override even `ACCOUNTADMIN`'s access.
- Model-level RBAC restrictions apply account-wide across *every* Cortex feature — can't scope a restriction to just one feature.

**Function semantics**
- Row-level functions (`AI_COMPLETE`, `SUMMARIZE`, `AI_SENTIMENT`) are bound by the model's context window; aggregate functions (`AI_AGG`, `AI_SUMMARIZE_AGG`) are not.
- `AI_FILTER`/`AI_CLASSIFY` fail "softly" (`NULL` per row) by default; `AI_REDACT` fails "hard" (whole query fails) by default — opposite defaults, controlled by `AI_SQL_ERROR_HANDLING_USE_FAIL_ON_ERROR`.
- `AI_SIMILARITY` can't mix modalities (text vs. image).
- `AI_REDACT` token cap: 4,096 combined in+out, 1,024 output max.
- `AI_SENTIMENT` trained/optimized around a ~2,048-token input window.
- `SUMMARIZE`: 32,000 input / 4,096 output token cap; still current, not deprecated.
- `AI_TRANSCRIBE`: 700 MB / 120 min max (60 min if timestamps requested); 50 tokens/sec billing, 1-minute minimum bill.
- `AI_PARSE_DOCUMENT`: LAYOUT mode required for image extraction; incompatible with custom network policies.
- Legacy functions (`COMPLETE`, `CLASSIFY_TEXT`, etc.) still work but are slated for deprecation by end of 2026.

**Cortex Search**
- 100M row cap on the materialized source query; 20 QPS per service / 140 QPS per account (raise via Snowflake support).
- Embedding model is locked in at creation — no in-place change.
- Default embedding model is English-only.
- 512-token semantic truncation, but full-text is still searchable via keyword.
- Refresh behavior mirrors Dynamic Tables — same incremental-refresh restrictions apply to the source query.
- Services execute with **owner's rights** — querying role needs only `USAGE` on the service, not `SELECT` on base tables.
- No cloning support.
- Auto-suspend minimum is 30 minutes; manual resume resets the full countdown.

**Cortex Analyst**
- "Facts" = legacy "measures" — same concept, renamed.
- Semantic Views (current) vs. semantic model YAML on stage (legacy, still supported).
- Anyone with stage access can use a YAML-based semantic model, even without `SELECT` on its underlying tables — ensure stage grants and table grants stay in sync.
- No access to prior query *results* in multi-turn conversations — only prior questions/SQL/text.
- Billed per successful message normally; per token when invoked through an Agent.
- Model choice is automatic, from a documented fallback preference order; not directly selectable.

**Cortex Agents**
- The orchestration layer that spans structured + unstructured + code execution + custom/MCP tools — this is the differentiator vs. Analyst/Search alone.
- Not usable from SiS on the warehouse runtime — needs container runtime.
- Web search tool must be enabled at the account level.
- Snowflake explicitly disclaims guaranteed accuracy of agent responses/citations — always review before serving to users.

---

<a name="part-8"></a>
## Part 8 — Practice Questions

**Q1.** A role has been granted `USE AI FUNCTIONS` at the account level but has **not** been granted `SNOWFLAKE.CORTEX_USER` or `SNOWFLAKE.AI_FUNCTIONS_USER`. Which of the following function calls will still succeed?
A) `AI_COMPLETE`
B) `AI_CLASSIFY`
C) `AI_SUMMARIZE_AGG`
D) `AI_TRANSLATE`

<details><summary>Answer</summary>
<b>C.</b> Snowflake documents a specific exception: a role with the account-level <code>USE AI FUNCTIONS</code> privilege but no <code>CORTEX_USER</code>/<code>AI_FUNCTIONS_USER</code> database role can still call <code>AI_AGG</code> and <code>AI_SUMMARIZE_AGG</code>. All other AI functions require both the privilege AND the database role.
</details>

**Q2.** You need to summarize 50,000 customer support transcripts, grouped by product line, into one paragraph per product. Which function should you use, and why?
A) `SUMMARIZE`, because it's the simplest
B) `AI_COMPLETE` with a long prompt containing all transcripts
C) `AI_SUMMARIZE_AGG` or `AI_AGG`, because they aggregate across rows without being limited by a single model's context window
D) `AI_EXTRACT`, because it returns structured JSON

<details><summary>Answer</summary>
<b>C.</b> <code>SUMMARIZE</code> and <code>AI_COMPLETE</code> both operate on a single input and are bound by the model's context window — 50,000 transcripts would overflow it. <code>AI_SUMMARIZE_AGG</code> (generic summary) or <code>AI_AGG</code> (custom-instruction summary) are aggregate functions explicitly documented as not subject to context-window limits, and support <code>GROUP BY</code>.
</details>

**Q3.** A Cortex Search Service was created without a `PRIMARY KEY`. What is the main operational consequence?
A) The service cannot be queried via the REST API
B) Refreshes cannot use the optimized incremental path, so more data may be re-embedded on each refresh
C) The service is limited to 1 million rows instead of 100 million
D) The embedding model cannot be changed

<details><summary>Answer</summary>
<b>B.</b> A TEXT-typed <code>PRIMARY KEY</code> enables an optimized incremental refresh path that only re-embeds changed/added rows, reducing refresh cost and latency. Without one, refreshes are less efficient. It does not change the row cap, REST API availability, or the embedding-model-lock-in rule (which applies regardless of primary key).
</details>

**Q4.** True or False: Cortex Analyst can be invoked without any per-token cost as long as fewer than 100 messages are sent per day.
<details><summary>Answer</summary>
<b>False.</b> Cortex Analyst billing (when called standalone, not via an Agent) is based on the number of successful messages processed — there is no free daily allotment implied by the documentation, and the pricing model is per-message, not a threshold-based free tier.
</details>

**Q5.** Which statement correctly distinguishes Cortex Agents from Cortex Analyst?
A) Cortex Agents only works with unstructured data; Cortex Analyst only works with structured data — they can never be combined
B) Cortex Analyst performs single-turn/multi-turn text-to-SQL against one semantic view; Cortex Agents can orchestrate multiple tools (including Analyst and Search) across several reasoning steps to answer a broader request
C) Cortex Agents is simply a rebranded version of Cortex Analyst with no functional differences
D) Cortex Analyst requires a Cortex Agent object to function

<details><summary>Answer</summary>
<b>B.</b> Cortex Analyst is a single-purpose text-to-SQL service scoped to a semantic view. Cortex Agents is the higher-level orchestrator that can call Cortex Analyst <i>and</i> Cortex Search <i>and</i> code execution/custom tools within one multi-step Plan → Use Tools → Reflect loop. Cortex Analyst can be used completely independently of any Agent.
</details>

**Q6.** An administrator wants to guarantee that a support engineer role can call `AI_COMPLETE` for troubleshooting write-ups but must never be able to use Cortex Search or Cortex Analyst, even indirectly. Which database role should be granted?
A) `SNOWFLAKE.CORTEX_USER`
B) `SNOWFLAKE.AI_FUNCTIONS_USER`
C) `SNOWFLAKE.CORTEX_ANALYST_USER`
D) `SNOWFLAKE.COPILOT_USER`

<details><summary>Answer</summary>
<b>B.</b> <code>AI_FUNCTIONS_USER</code> is scoped specifically to the scalar AI functions and explicitly excludes Cortex Agent, Analyst, and Search access — unlike the broader <code>CORTEX_USER</code>, which grants all Covered AI Features.
</details>

---

## Sources

All facts, limits, parameter names, and privilege names in this guide are drawn from official Snowflake documentation, primarily:
- `docs.snowflake.com/en/user-guide/snowflake-cortex/aisql` (Cortex AI Functions overview)
- `docs.snowflake.com/en/user-guide/snowflake-cortex/aisql-privileges-and-access`
- `docs.snowflake.com/en/sql-reference/functions/ai_*` (individual function reference pages)
- `docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-search/cortex-search-overview`
- `docs.snowflake.com/en/sql-reference/sql/create-cortex-search`
- `docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst`
- `docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents`
- `docs.snowflake.com/en/user-guide/snowflake-cortex/redact-pii`, `.../ai-audio`, `.../ai-documents`, `.../ai-sentiment`, `.../vector-embeddings`
- `docs.snowflake.com/en/user-guide/snowflake-cortex/opting-out`

Snowflake ships new Cortex features and revises limits frequently — before an exam attempt or production build, re-check current numbers (token limits, row/QPS caps, regional availability, and pricing) directly against the live documentation.
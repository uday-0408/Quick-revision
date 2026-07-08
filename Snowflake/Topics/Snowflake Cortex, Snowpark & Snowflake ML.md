# Snowflake Cortex, Snowpark & Snowflake ML — Complete Guide

*A simple, detailed explanation of Snowpark, Snowflake Cortex, AI SQL Functions, Cortex Search, Cortex Analyst, and Snowflake ML — based entirely on official Snowflake documentation (docs.snowflake.com).*

---

## Table of Contents

1. [The Big Picture](#1-the-big-picture)
2. [Snowpark](#2-snowpark)
3. [Snowflake Cortex (the umbrella)](#3-snowflake-cortex-the-umbrella)
4. [AI SQL Functions (Cortex AI Functions)](#4-ai-sql-functions-cortex-ai-functions)
5. [Cortex Search](#5-cortex-search)
6. [Cortex Analyst](#6-cortex-analyst)
7. [Snowflake ML](#7-snowflake-ml)
8. [Quick Comparison Table](#8-quick-comparison-table)
9. [Sources](#9-sources)

---

## 1. The Big Picture

Snowflake documentation groups its intelligent features into **two broad categories**:

| Category | What it is | What's inside |
|---|---|---|
| **Snowflake Cortex** | A suite of AI features that use large language models (LLMs) to understand unstructured data, answer freeform questions, and give intelligent assistance | Cortex Agents, AI SQL Functions (including LLM functions), Cortex Analyst, Cortex Fine-tuning, Cortex Search, Snowflake CoWork, Cortex Code, Cortex AI Guardrails |
| **Snowflake ML** | Lets you build, train, and manage your **own** custom ML models | ML Functions (no-code), Snowpark ML (developer-facing), Feature Store, Model Registry, Container Runtime, ML Jobs, Experiments, ML Observability, ML Lineage |

Snowpark sits alongside these as the general-purpose **developer framework** — it's how you run Python, Java, or Scala code directly next to your data in Snowflake, and it's also the toolkit that powers custom Snowflake ML development.

All of Snowflake's AI features are built around three core principles (straight from the documentation):

- **Full security** — except as you elect, all AI models run inside Snowflake's security and governance perimeter. Your data is never available to other customers or model developers.
- **Data privacy** — Snowflake never uses your Customer Data to train the models made available to its customer base.
- **Control** — you control your team's use of these features through familiar role-based access control (RBAC).

Snowflake also documents an **AI/ML model lifecycle**: models move through Private Preview → Public Preview → General Availability (GA) → Legacy → End of Life (EOL). For GA models, Snowflake gives at least 60 days' notice before deprecating them; preview models can change with shorter notice.

---

## 2. Snowpark

### What it is

> "The Snowpark API provides an intuitive library for querying and processing data at scale in Snowflake. Using a library for any of three languages, you can build applications that process data in Snowflake without moving data to the system where your application code runs, and process at scale as part of the elastic and serverless Snowflake engine."

In simple words: **Snowpark lets you write Python, Java, or Scala code that runs right next to your data inside Snowflake**, instead of pulling the data out to your own servers. Snowflake describes it as "the set of libraries and code execution environments that run Python and other programming languages next to your data in Snowflake." You can use it to build data pipelines, ML models, apps, and other data processing tasks.

### Supported languages

Snowflake currently provides Snowpark libraries for **three languages**:

- Java
- Python
- Scala

### Why it exists (key benefits vs. the Spark Connector)

Compared with using the Snowflake Connector for Spark, Snowpark gives you:

- Support for querying/processing data using libraries and patterns built specifically for each language, without giving up performance or functionality.
- Support for authoring code in local tools like **Jupyter, VS Code, or IntelliJ**.
- **Pushdown for all operations**, including UDFs — meaning Snowpark pushes all the data transformation and heavy lifting down into Snowflake itself, so you can work with data of any size efficiently.
- **No separate cluster** is needed outside Snowflake. All computation happens inside Snowflake, and scale/compute management is handled by Snowflake.

### Core building blocks

**1. Native SQL construction**
Instead of writing raw SQL strings, Snowpark gives you programming-language constructs — for example a `.select()` method — so you get intelligent code completion and type checking.

```python
from snowflake.snowpark.functions import col

df = session.table("sample_product_data").select(
    col("id"), col("name"), col("serial_number")
)
df.show()
```

**2. The DataFrame — lazy evaluation**
The core abstraction in Snowpark is the **DataFrame**. Building a DataFrame does *not* run the query. Snowpark operations are executed **lazily** on the server — meaning the library delays actual execution until as late as possible, batching operations together. This reduces the data transferred between your client and Snowflake and improves performance.

```python
# This does NOT execute the query yet:
df = session.table("sample_product_data").select(col("id"), col("name"))

# This sends the query to Snowflake and returns the results:
results = df.collect()
```

**3. Inline User-Defined Functions (UDFs)**
You can create UDFs inline, in the same language as your client code (lambda functions in Python, anonymous functions in Scala, etc.). Snowpark automatically pushes this custom code to the Snowflake engine, so it runs on the server where the data already lives — you don't need to move data to your client to execute it.

```python
from snowflake.snowpark.types import IntegerType

add_one = udf(
    lambda x: x + 1,
    return_type=IntegerType(),
    input_types=[IntegerType()],
    name="my_udf",
    replace=True
)
```

### What you can build with Snowpark (Python specifically)

According to the Snowpark Developer Guide for Python, you can:

- Query and process data with a **DataFrame**.
- Run **pandas code directly** on data in Snowflake (pandas on Snowflake).
- Create **User-Defined Functions (UDFs)** and **User-Defined Table Functions (UDTFs)**.
- Write **stored procedures** to automate tasks and build data pipelines (including scheduling them as tasks).
- **Train, score, and tune ML models** using Snowpark Python stored procedures, and deploy trained models using UDFs.
- Write directly in **Python worksheets in Snowsight** (no local setup required) or in a local development environment (Jupyter, VS Code, IntelliJ).
- **Troubleshoot** with logging statements, view underlying SQL, and record log/trace messages in an event table.

### Snowpark Container Services (SPCS)

For workloads that need more flexibility than Python/Java/Scala functions can offer, Snowflake also provides **Snowpark Container Services** — a fully managed container orchestration platform *within* Snowflake. You package your app and dependencies into an OCI (Open Container Initiative) image (any language, framework, or library) and Snowflake handles the underlying infrastructure, security, and configuration.

Common workloads for Snowpark Container Services:

- **Batch Data Processing Jobs** — flexible jobs (similar to stored procedures) that pull data, process it, and produce results; can use GPUs for AI/ML workloads.
- **Service Functions** — a service that your SQL queries can call directly, sending it batches of data to process.

Your application inside SPCS can connect to Snowflake, run SQL in a virtual warehouse, access stage files, and use your existing Snowflake configuration — all while being deployable across AWS, Azure, or GCP without you worrying about the underlying cloud platform.

### Getting started

You can download the Snowpark library for any of the three languages from the Snowpark Client Download page on the Snowflake Developer Center, or use one of Snowflake's Quickstarts (Machine Learning with Snowpark Python, Data Engineering Pipelines with Snowpark Python, Getting Started with Snowpark for Python and Streamlit, and others).

---

## 3. Snowflake Cortex (the umbrella)

### What it is

> "Snowflake Cortex is a suite of AI features that use large language models (LLMs) to understand unstructured data, answer freeform questions, and provide intelligent assistance."

Think of **Snowflake Cortex** as the umbrella name for *all* of Snowflake's generative-AI-powered features. According to the documentation, this suite comprises:

- **Cortex Agents** — orchestrate across structured and unstructured data sources to answer complex questions.
- **Snowflake Cortex AI Functions** (including LLM functions) — SQL/Python functions for text and image tasks (see Section 4).
- **Cortex Analyst** — natural-language-to-SQL over your structured data (see Section 6).
- **Cortex Fine-tuning** — lets you fine-tune LLMs on your own data.
- **Cortex Search** — hybrid (vector + keyword) search engine for unstructured data (see Section 5).
- **Snowflake CoWork** — a Cortex-powered work assistant.
- **Cortex Code** — includes Cortex Code in Snowsight, the Cortex Code CLI, an Agent SDK, and MCP/ACP protocol support and plugins.
- **Cortex AI Guardrails** — built-in safety controls to reduce harmful content.

All Cortex models run **within the Snowflake Service perimeter**, and features are provided as **SQL functions**, also available **in Python**.

### Important legal/usage notes (straight from documentation)

- Your use of Snowflake AI Features is subject to Snowflake's **Acceptable Use Policy**.
- **Outputs may be inaccurate, inappropriate, inefficient, or biased.** Decisions based on these outputs — including ones built into automated pipelines — should have human oversight and review to make sure they're safe, accurate, and suitable.
- If a feature is powered by a third-party or open-source model, your use of it is also subject to that model's license/acceptable-use terms.

### Model updates & behavior changes

Snowflake continuously updates the models powering Cortex to improve quality, performance, and availability. A change is considered a **"behavior change"** if it:

- Changes required syntax (e.g., how you specify a model/version),
- Changes the structure of model outputs, or
- Deprecates a model.

These are communicated via **Behavior Change Releases (BCRs)** (changes that may need customer action) or the **What's New** log (improvements that don't materially change how you interact with models). Deprecations are always communicated separately and clearly, with the 60-day/GA and shorter/preview notice periods mentioned earlier.

---

## 4. AI SQL Functions (Cortex AI Functions)

### What they are

> "Use Cortex AI Functions in Snowflake to run unstructured analytics on text and images with industry-leading LLMs from OpenAI, Anthropic, Meta, Mistral AI, and DeepSeek."

These are the functions formerly generally known as "Cortex LLM functions" — Snowflake's official current documentation calls the full page **"Snowflake Cortex AI Functions (including LLM functions)."** They let you run generative-AI tasks directly in **SQL** (also available in Python) without setting up any infrastructure. All the underlying LLMs are deployed inside the Snowflake Service perimeter.

**Typical use cases**, per the documentation:

- Extracting entities to enrich metadata and streamline validation
- Aggregating insights across customer tickets
- Filtering and classifying content by natural language
- Sentiment and aspect-based analysis for service improvement
- Translating and localizing multilingual content
- Parsing documents for analytics and RAG pipelines

### The functions themselves

**Cortex AI functions** (task-specific, purpose-built, no customization needed):

| Function | What it does |
|---|---|
| `AI_COMPLETE` | Generates a completion for a text string or image using a chosen LLM. Use this for most generative AI tasks. |
| `AI_CLASSIFY` | Classifies text or images into categories you define. |
| `AI_FILTER` | Returns `True`/`False` for a text or image input — useful for filtering in `SELECT`, `WHERE`, or `JOIN … ON` clauses. |
| `AI_AGG` | Aggregates a text column and returns insights across multiple rows based on a prompt you write. Not limited by context-window size. |
| `AI_EMBED` | Generates an embedding vector for text or an image, useful for similarity search, clustering, and classification. |
| `AI_EXTRACT` | Extracts information from a string or file (text, images, documents). Supports multiple languages. |
| `AI_SENTIMENT` | Extracts sentiment from text. |
| `AI_SUMMARIZE_AGG` | Aggregates a text column and returns a summary across multiple rows. Not limited by context-window size. |
| `AI_SIMILARITY` | Calculates embedding similarity between two inputs. |
| `AI_TRANSCRIBE` | Transcribes audio/video files stored in a stage, extracting text, timestamps, and speaker information. |
| `AI_PARSE_DOCUMENT` | Extracts text (OCR mode) or text-with-layout (LAYOUT mode) from documents in a stage; can also pull out images found in the document. |
| `AI_REDACT` | Redacts personally identifiable information (PII) from text. |
| `AI_TRANSLATE` | Translates text between supported languages. |
| `SUMMARIZE` (`SNOWFLAKE.CORTEX`) | Returns a summary of specified text. *(Note: `COMPLETE (SNOWFLAKE.CORTEX)` is the legacy version of the completion function and is scheduled to be deprecated by the end of 2026 — use `AI_COMPLETE` instead.)* |

**Helper functions** (reduce failure cases when calling the functions above):

| Function | What it does |
|---|---|
| `TO_FILE` | Creates a reference to a file in a stage for use with `AI_COMPLETE` and similar functions. |
| `AI_COUNT_TOKENS` | Returns the token count of an input, based on the model or function specified. |
| `PROMPT` | Helps you build prompt objects for use with `AI_COMPLETE` and other functions. |

### Example query

```sql
SELECT AI_COMPLETE(
    'llama3.1-70b',
    'Summarize this customer feedback in one sentence: ' || feedback_text
) AS summary
FROM customer_feedback;
```

### Privileges needed

To call any of these functions, your role needs:

- The account-level privilege **`USE AI FUNCTIONS`**, **and**
- One of the database roles **`CORTEX_USER`** or **`AI_FUNCTIONS_USER`**.

### Performance: batch vs. interactive

- Cortex AI Functions are **optimized for throughput** — best for processing lots of inputs at once (e.g., text across large SQL tables). Batch processing is a great fit.
- For **interactive** use cases where latency matters, use the **REST API** instead — available for simple inference (Complete API), embeddings (Embed API), and agentic applications (Agents API).

### Data classification & legal notice

Input is classified as "Usage Data" and output as "Customer Data." Generally available functions are "Covered AI Features"; preview functions are "Preview AI Features" (defined terms in Snowflake's AI Terms and Acceptable Use Policy).

---

## 5. Cortex Search

### What it is

> "Cortex Search enables low-latency, high-quality 'fuzzy' search over your Snowflake data... Cortex Search gets you up and running with a hybrid (vector and keyword) search engine on your text data in minutes, without having to worry about embedding, infrastructure maintenance, search quality parameter tuning, or ongoing index refreshes."

In simple terms: **Cortex Search is Snowflake's built-in, managed search engine** for your text data. You point it at a table/view (or files in a stage), and it automatically handles embeddings, indexing, and keeping the index fresh.

### When to use it

- **RAG engine for LLM chatbots** — use it for Retrieval Augmented Generation (RAG), where you retrieve relevant data from a knowledge base to ground an LLM's response in your own up-to-date data.
- **Enterprise search** — as the backend for a high-quality search bar embedded in your application.
- **Powering agents** — as the retrieval layer for Cortex Agents, combined with analytical search for large document sets.

### How search quality works — "hybrid" retrieval

Every Cortex Search query combines three techniques:

1. **Vector search** — retrieves semantically similar documents.
2. **Keyword search** — retrieves lexically similar documents.
3. **Semantic reranking** — reranks the most relevant documents in the result set.

You can also customize scoring — numeric boosts, time decays, adjusting component weights, or disabling reranking.

### Embedding models available

| Model | Output dimensions | Context window | Language support |
|---|---|---|---|
| `snowflake-arctic-embed-m-v1.5` *(default)* | 768 | 512 tokens | English-only |
| `snowflake-arctic-embed-l-v2.0` | 1024 | 512 tokens | Multilingual |
| `snowflake-arctic-embed-l-v2.0-8k` | 1024 | 8,192 tokens | Multilingual |
| `voyage-multilingual-2` | 1024 | 32,000 tokens | Multilingual |

For best retrieval quality, Snowflake recommends splitting search-column text into chunks of **no more than 512 tokens** (~385 English words) — even though longer-context models exist, research shows smaller chunks typically give better retrieval and downstream LLM response quality. Snowflake provides a built-in `SPLIT_TEXT_RECURSIVE_CHARACTER` function to help split text into chunks.

### Creating a service (SQL example)

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

- `TARGET_LAG` controls how often the service checks the base table for updates.
- The named `WAREHOUSE` materializes the results initially and on every refresh.
- Columns in `ATTRIBUTES` become filterable alongside the main search column.

Querying it (Python API):

```python
from snowflake.core import Root
from snowflake.snowpark import Session

session = Session.builder.configs(CONNECTION_PARAMETERS).create()
root = Root(session)

service = (root.databases["cortex_search_db"]
               .schemas["services"]
               .cortex_search_services["transcript_search_service"])

resp = service.search(
    query="internet issues",
    columns=["transcript_text", "region"],
    filter={"@eq": {"region": "North America"}},
    limit=1
)
```

### Primary keys & multi-index search

- You can define a **primary key** (must be `TEXT` type columns) so refreshes only re-process changed rows instead of the whole dataset — much cheaper and faster.
- **Multi-index Cortex Search** lets you index multiple columns, mix text indexes (great for exact/fuzzy keyword fields like product codes) with vector indexes (great for longer text like descriptions or reviews), and even bring your own pre-computed embeddings.

### Required privileges

- `SNOWFLAKE.CORTEX_USER` or `SNOWFLAKE.CORTEX_EMBED_USER` database role to create a service.
- `CREATE CORTEX SEARCH SERVICE` (or `OWNERSHIP`) privilege on the schema.
- `SELECT` on the underlying table(s)/view(s).
- `USAGE` on the refreshing warehouse.
- To *query* a service, the querying role needs `USAGE` on the service, database, and schema.
- **Change tracking must be enabled** on all underlying source objects.

### Cost considerations

Cortex Search bills across five categories:

| Category | What drives the cost |
|---|---|
| Virtual warehouse compute | Refreshing/materializing the service and running the initial index build |
| Embedding token compute | Embedding each row of text (only for new/changed rows) |
| Multi-index Cortex Search | Extra cost scales with vector size and number of indexed columns |
| Serving compute | Billed per GB/month of indexed data, even if idle (unless auto-suspended) |
| Storage | Flat rate per TB for the materialized table + search data structures |

You can reduce idle serving cost using `AUTO_SUSPEND` (minimum 1800 seconds / 30 minutes of inactivity before the service suspends).

### Known limitations

- Base table (materialized query result) must be **under 100 million rows** by default.
- Default throughput limits: **20 QPS** per service, **140 QPS** across all services in an account (can be raised by contacting your Snowflake account team).
- Source queries must follow the same restrictions as **Dynamic Tables** (they must support incremental refresh).
- **Cloning is not currently supported.**
- Underlying tables must not be modified/dropped while the service is running — stop the service first if you need to change them.

---

## 6. Cortex Analyst

### What it is

> "Cortex Analyst is a fully-managed, LLM-powered Snowflake Cortex feature that helps you create applications capable of reliably answering business questions based on your structured data in Snowflake. With Cortex Analyst, business users can ask questions in natural language and receive direct answers without writing SQL."

In short: **Cortex Analyst turns natural-language questions into accurate SQL** against your structured data, and it's available as a REST API you can plug into any application (Streamlit, Slack, Teams, custom chat UIs, and more).

### Key features

- **Self-serve analytics via natural language** — business users ask questions in plain English and get answers instantly.
- **Convenient REST API** — API-first design, so you control the end-user experience completely.
- **Powered by state-of-the-art LLMs** — by default, Cortex Analyst runs on Snowflake-hosted models from Mistral and Meta, but at runtime it automatically selects the best combination of available models for accuracy and performance.
- **Semantic models for precision** — rather than relying on raw database schemas (which lack business context), Cortex Analyst uses a **semantic model** to bridge business language and database structure.
- **Security & governance** — no data (including metadata or prompts) leaves Snowflake's governance boundary by default; fully integrated with Snowflake RBAC.

### Semantic Views — the recommended approach

Cortex Analyst uses **Semantic Views** (schema-level Snowflake objects) to understand your data. A Semantic View defines:

- **Logical tables** — business entities (customers, orders, products)
- **Dimensions** — categorical context (customer name, product category, order date)
- **Facts** — row-level quantitative data (sale amounts, quantities)
- **Metrics** — aggregated business KPIs (total revenue, average order value)
- **Relationships** — how tables join together

Why Semantic Views help:

- **Rich metadata** — descriptions and synonyms help the LLM understand your data.
- **Correct business logic** — metrics capture the right aggregation/calculation rules.
- **Predefined join paths** — ensures correct multi-table queries.
- **Verified examples** — sample questions with their SQL answers guide generation.

Benefits over legacy semantic model YAML files (still supported, but not recommended for new work): native RBAC/governance, easy sharing, derived metrics across tables, and public/private access modifiers on facts and metrics.

### Access control

- Use a role with **`SNOWFLAKE.CORTEX_USER`** (access to all Covered AI features) or **`SNOWFLAKE.CORTEX_ANALYST_USER`** (Cortex Analyst only) database role.
- `CORTEX_USER` is granted to `PUBLIC` by default — meaning every user has it unless you revoke it.
- To use a semantic model on a stage, you also need `READ`/`WRITE` on that stage, `USAGE` on any Cortex Search services it references, and `SELECT` on referenced tables.

### Multi-turn conversations

Cortex Analyst supports **follow-up questions** that build on previous ones. Example from the documentation:

> User: *"What is the month-over-month revenue growth for 2021 in Asia?"*
> Follow-up: *"What about North America?"*

Cortex Analyst recognizes the context and rephrases the follow-up into a complete question before generating SQL. You pass conversation history via the `messages` field, alternating `"role": "user"` and `"role": "analyst"` entries.

**Known limitations of multi-turn conversations:**

- No access to the *results* of previous SQL queries (it can't refer back to a specific returned value).
- Limited to questions resolvable with SQL — it can't generate open-ended business insights like "what trends do you observe?"
- Very long or intent-shifting conversations may confuse follow-up interpretation — in that case, reset the conversation.

### Model selection

You can't pick a model directly — Cortex Analyst automatically assigns your request to the best available option, in this order of preference:

1. Anthropic Claude Sonnet 4.6
2. Anthropic Claude Sonnet 4.5
3. OpenAI GPT 4.1
4. Arctic Text2SQL R1.5 (with thinking enabled)
5. A combination of Mistral Large 2 and Llama 3.1 70b

(Model-level RBAC can restrict which of these your role can use, but Snowflake recommends against this unless you have specific regulatory requirements.)

### Cost

Billing is based on the **number of messages processed** (only successful/HTTP 200 responses count). Token count only affects cost when Cortex Analyst is invoked through Cortex Agents. Note: this covers the *text-to-SQL* AI cost only — running the generated SQL still incurs normal warehouse compute cost.

### Getting started (documented workflow)

1. **Create a semantic model** — start from your list of intended questions, then build it with the Semantic View Autopilot or by hand via YAML.
2. **Upload the semantic model** to a stage (or pass its YAML as a string in the API request).
3. **Build a Streamlit app** (in Snowflake or standalone) that takes a natural-language question and calls the Cortex Analyst API.
4. **Interact** with the app by asking questions in plain language.

### Disabling it

```sql
USE ROLE ACCOUNTADMIN;
ALTER ACCOUNT SET ENABLE_CORTEX_ANALYST = FALSE;
```

---

## 7. Snowflake ML

### What it is

> "Snowflake ML is an integrated set of capabilities for end-to-end machine learning in a single platform on top of your governed data... optimized for large-scale distributed feature engineering, model training and inference on CPU and GPU compute without manual tuning or configuration."

While Cortex is about *using* pre-built LLMs, **Snowflake ML is about building your own models** — traditional ML or custom pipelines — without moving your data out of Snowflake.

Snowflake ML lets you automate, from simple prompts via **Cortex Code** (agentic ML) or manually:

- Prepare data
- Create and use features with the **Feature Store**
- Train models with CPUs or GPUs, using any open-source package, from **Snowflake Notebooks on Container Runtime**
- Create **Experiments** to evaluate trained models against metrics
- Operationalize pipelines using **Snowflake ML Jobs**
- Deploy models for inference at scale with the **Model Registry**
- Monitor production models with **ML Observability and Explainability**
- Track lineage from source data → features → datasets → models with **ML Lineage**

Models built in Snowflake can also be deployed outside Snowflake, and externally-trained models can be brought in for inference.

### Agentic ML

**Cortex Code** powers *agentic ML* — it can autonomously plan, execute, and iterate on ML workflows for you. From one natural-language prompt, it can explore data, engineer features, train and evaluate models, debug issues, and prepare models for deployment — either for a single task or a broader multi-step goal.

### Model training — Container Runtime

ML training runs inside **Container Runtime**, a pre-built ML environment optimized for efficient data loading, distributed training, and hyperparameter tuning on large datasets. It comes with popular packages preinstalled (PyTorch, XGBoost, scikit-learn), and you can install anything else from HuggingFace or PyPI. You access it directly from **Snowflake Notebooks** — a Jupyter-like environment with no infrastructure to manage.

### Snowflake Feature Store

> "The Snowflake Feature Store lets data scientists and ML engineers create, maintain, and use ML features in data science and ML workloads, all within Snowflake."

**Features** are the data inputs to an ML model (e.g., a "day-of-week" column derived from a timestamp). **Feature engineering** is deciding what features you need and how to derive them. A **feature store** standardizes these transformations in a central, reusable repository — reducing duplicated effort and keeping features fresh and consistent.

Key benefits documented:

- Data stays fully inside Snowflake's governance.
- Snowsight UI makes features searchable/discoverable.
- Fine-grained role-based access control.
- Supports **both batch and streaming** data with automatic incremental updates.
- Supports **point-in-time correct** features via `ASOF JOIN` (important for avoiding data leakage when building training sets).
- Feature transformations can be written in **Python or SQL**.
- Works with external tools like **dbt** for user-managed pipelines.
- Fully integrated with the **Model Registry** and **ML Lineage** — so inference can automatically retrieve the correct feature values without you supplying every input manually.

**How it's structured internally:**

| Feature Store concept | Underlying Snowflake object |
|---|---|
| Feature store | Schema |
| Feature view | Dynamic table or view |
| Entity | Tag |
| Feature | Column in a dynamic table or view |

A **feature view** encapsulates a Python or SQL pipeline that transforms raw data into features, all refreshed together. Feature views come in two flavors: **Snowflake-managed** (auto-refreshed on your schedule) or **External** (maintained by an outside process like dbt). Feature views are grouped by **entities** — higher-level subjects like "users" or "movies" that features relate to.

### Snowflake Model Registry

> "After training your model, operationalizing the model and running inference in Snowflake starts with logging the model in the Snowflake Model Registry."

The Model Registry:

- Stores and manages **model versions, metrics, and metadata**.
- Serves models and runs **distributed inference** via Python, SQL, or REST API endpoints.
- Manages the model **life cycle** across dev → prod.
- Monitors performance/drift via **ML Observability**.
- Manages access securely with **RBAC**.

Models are stored as **first-class schema-level objects**. Built-in support covers scikit-learn, XGBoost, LightGBM, Prophet, CatBoost, PyTorch, TensorFlow, Keras, Hugging Face pipelines, Sentence Transformers, and MLflow pyfunc models — plus your own custom code.

Basic Python workflow:

```python
from snowflake.ml.registry import Registry

reg = Registry(session=session, database_name="ML", schema_name="REGISTRY")

mv = reg.log_model(
    clf,
    model_name="my_model",
    version_name="v1",
    comment="My awesome ML model",
    metrics={"score": 96}
)
```

Some useful facts from the docs:

- Each model can have **unlimited versions** (max 1000 per model); each version supports up to 10 methods.
- **`USAGE`** privilege lets a role run warehouse inference without seeing internals; **`READ`** lets a role also use SPCS (Snowpark Container Services) inference and see metadata.
- You can compute **Shapley values** to explain which features drive a model's predictions (Model Explainability).
- Models can be exported, loaded back into Python, shared, or deployed as a containerized inference service for GPU-based serving.
- Note: models trained with the no-code **ML Functions** (like `FORECAST`) do *not* appear in the Model Registry; Cortex Fine-Tuned LLMs appear in the Snowsight UI but aren't managed through the registry API.

### ML Jobs, Experiments, Observability & Lineage

- **Snowflake ML Jobs** — develop and automate ML pipelines; also let teams working from external IDEs (VS Code, PyCharm, SageMaker Notebooks) dispatch functions/files down to Snowflake's Container Runtime.
- **Experiments** — record model-training results in an organized way so you can compare and pick the best model, whether trained in Snowflake or logged from elsewhere.
- **ML Observability** — monitor performance and drift metrics in production, with alerting on thresholds.
- **ML Lineage** — traces the full path from source data → features → datasets → models, supporting reproducibility, compliance, and debugging.

### Snowpark ML Modeling (the developer library)

For teams that prefer to write standard open-source-style ML code, Snowflake documentation describes the **`snowflake.ml.modeling`** package:

- Provides **distributed preprocessing** functions (`snowflake.ml.modeling.preprocessing`) for scalable data transformations (scalers, encoders, etc., modeled closely on scikit-learn).
- Includes a large collection of **model development classes** based on **scikit-learn, XGBoost, and LightGBM**.
- Provides **Framework Connectors** for optimized, secure data delivery to PyTorch and TensorFlow in their native loader formats.
- Recommends using your **existing open-source code** directly — no need to rewrite your pipeline just to run it in Snowflake.
- Distributed APIs let you scale feature engineering and training **across multiple nodes**, including via **Ray** (an open-source framework for parallelizing Python).

### ML Functions — the no-code option

Distinct from the developer-facing Snowflake ML platform above, Snowflake also documents a simpler, **no-programming-required** set of SQL-based **ML Functions**, aimed at analysts:

> "These powerful analysis functions give you automated predictions and insights into your data using machine learning. Snowflake provides an appropriate type of model for each feature, so you don't have to be a machine learning expert to take advantage of them. All you need is your data."

**Time-Series Functions** (need time-series data):

- **Forecasting** — predicts future metric values from past trends.
- **Anomaly Detection** — flags metric values that differ from typical expectations; supports both unsupervised and supervised (labeled) modes, and lets you tune sensitivity via a `prediction_interval`.

**Other Analysis Functions** (no time-series data needed):

- **Classification** — sorts rows into two or more classes using a gradient boosting machine; supports binary and multi-class classification. Common uses: churn prediction, fraud detection, spam detection.
- **Top Insights (Contribution Explorer)** — helps you find which dimensions and values affect a metric in surprising ways (root-cause analysis).

Example — creating and using an anomaly detection model:

```sql
CREATE OR REPLACE SNOWFLAKE.ML.ANOMALY_DETECTION my_detector(
    INPUT_DATA => TABLE(training_data_view),
    TIMESTAMP_COLNAME => 'date',
    TARGET_COLNAME => 'sales'
);

CALL my_detector!DETECT_ANOMALIES(
    INPUT_DATA => TABLE(new_data_view),
    TIMESTAMP_COLNAME => 'date',
    TARGET_COLNAME => 'sales'
);
```

Notes on ML Functions:

- Each trained model is stored as a **schema-level object**.
- You must have **`AUTOCOMMIT`** enabled in your session (it's on by default).
- Costs come from **storage** (the model instances created during training) and **compute** (training and prediction).
- Classification models support two scoped roles: **`mladmin`** (full access, including exploratory/evaluation APIs) and **`mlconsumer`** (prediction only).
- These models **cannot be cloned**, and are skipped when cloning/replicating a database.

---

## 8. Quick Comparison Table

| Feature | Best for | Interface | Needs your own model? |
|---|---|---|---|
| **Snowpark** | Running Python/Java/Scala code next to your data; building pipelines, apps, and custom ML | DataFrame API, UDFs, stored procs (Java/Python/Scala) | You write the logic |
| **Snowflake Cortex** (umbrella) | The whole suite of ready-made generative AI capabilities | SQL, Python, REST APIs | No — uses hosted LLMs |
| **AI SQL Functions** | Quick text/image tasks: summarize, translate, classify, extract, embed, sentiment, transcribe | SQL functions / Python | No — pick a hosted model |
| **Cortex Search** | Fast "fuzzy" hybrid search over your text data; RAG and enterprise search | SQL (`CREATE CORTEX SEARCH SERVICE`) / REST / Python | No — managed embeddings & index |
| **Cortex Analyst** | Turning natural-language questions into accurate SQL over structured data | REST API + Semantic Views | No — uses hosted LLMs, guided by your semantic model |
| **Snowflake ML** | Building, training, and operationalizing your **own custom** ML models at scale | Python (`snowflake-ml-python`), Snowsight, SQL, Notebooks | Yes — you develop/train it (or use no-code ML Functions) |

---

## 9. Sources

All information above comes directly from official Snowflake documentation:

- [Snowpark API](https://docs.snowflake.com/en/developer-guide/snowpark/index)
- [Snowpark Developer Guide for Python](https://docs.snowflake.com/en/developer-guide/snowpark/python/index)
- [Snowpark Container Services](https://docs.snowflake.com/en/developer-guide/snowpark-container-services/overview)
- [Snowflake AI and ML (overview)](https://docs.snowflake.com/en/guides-overview-ai-features)
- [Snowflake Cortex AI Functions (including LLM functions)](https://docs.snowflake.com/en/user-guide/snowflake-cortex/aisql)
- [Cortex Search](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-search/cortex-search-overview)
- [Cortex Analyst](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst)
- [Snowflake ML: End-to-End Agentic ML (overview)](https://docs.snowflake.com/en/developer-guide/snowflake-ml/overview)
- [Snowflake Feature Store](https://docs.snowflake.com/en/developer-guide/snowflake-ml/feature-store/overview)
- [Snowflake Model Registry](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/overview)
- [ML Functions (overview)](https://docs.snowflake.com/en/guides-overview-ml-functions)
- [Anomaly Detection (Snowflake ML Functions)](https://docs.snowflake.com/en/user-guide/ml-functions/anomaly-detection)
- [Classification (Snowflake ML Functions)](https://docs.snowflake.com/en/user-guide/ml-functions/classification)
- [Snowflake ML Model Development / Snowpark ML Modeling](https://docs.snowflake.com/developer-guide/snowflake-ml/modeling)

*Guide compiled July 2026. Snowflake updates its documentation frequently — especially model lists, regional availability, and pricing — so always check the linked pages above for the latest details before relying on specifics like model names, quotas, or costs in production.*
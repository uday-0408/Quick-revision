# Trust Center — SnowPro Core Notes
*Topic 1 of your Snowflake security notes series*

---

## Table of Contents
1. [What is Trust Center? (Plain English)](#1-what-is-trust-center-plain-english)
2. [How It Works — The Core Flow](#2-how-it-works--the-core-flow)
3. [Violations vs Detections](#3-violations-vs-detections)
4. [Schedule-Based vs Event-Driven Scanners](#4-schedule-based-vs-event-driven-scanners)
5. [The Four Scanner Packages](#5-the-four-scanner-packages)
6. [Roles & Access Control](#6-roles--access-control)
7. [The Trust Center Interface](#7-the-trust-center-interface)
8. [Managing Violations (Lifecycle)](#8-managing-violations-lifecycle)
9. [Managing Detections](#9-managing-detections)
10. [Notifications: Email vs Notification Integrations](#10-notifications-email-vs-notification-integrations)
11. [Monitoring Trust Center Cost](#11-monitoring-trust-center-cost)
12. [Trust Center Extensions](#12-trust-center-extensions)
13. [Consolidated Comparison Tables](#13-consolidated-comparison-tables)
14. [Common Misconceptions](#14-common-misconceptions)
15. [Exam Trap Cheat Sheet](#15-exam-trap-cheat-sheet)
16. [Key Numbers to Memorize](#16-key-numbers-to-memorize)
17. [Self-Check Questions](#17-self-check-questions)
18. [Sources](#18-sources)

---

## 1. What is Trust Center? (Plain English)

**One-line definition:** Trust Center is a built-in Snowflake feature that constantly checks your account for security problems and tells you what to fix.

**Analogy:** Think of it like a building inspector who walks through your house on a schedule, checks a list of safety rules (locked windows, working smoke alarms), and leaves a sticky note wherever something's wrong. Some notes are about things that are *still* wrong right now (an unlocked window — a **Violation**). Other notes are about something that *happened once* (someone rang the doorbell at 3 AM last night — a **Detection**). You can't "un-ring" that doorbell, but you can look into whether it matters.

**Why Snowflake built it:** Before Trust Center, security teams had to write their own SQL against `ACCOUNT_USAGE` views, build their own dashboards, and remember to check regularly for things like "did someone forget to turn on MFA?" or "is some old employee's account still active?" Trust Center automates that detective work directly inside Snowflake, using serverless compute Snowflake manages for you, and gives you ready-made fixes instead of just raw data.

**Important:** Not every scan produces a finding. If your account passes a check, nothing shows up — silence is a good sign here.

---

## 2. How It Works — The Core Flow

```
Your Snowflake account
 (configuration + activity)
          │
          ▼
   A SCANNER runs
 (on a schedule, OR
  triggered by an event)
          │
          ▼
 Checks against a rule
 e.g. "Is MFA enforced
 for all human users?"
          │
   ┌──────┴───────┐
   ▼               ▼
 Rule passes     Rule broken /
 → no finding    risky event found
                    │
                    ▼
             FINDING created
          (a Violation or a Detection)
                    │
                    ▼
         Shown in Snowsight
     (Violations / Detections tab)
                    │
                    ▼
        Notification sent — IF the
        finding's severity meets your
        configured threshold
     (Email, and/or Webhook/Queue)
```

**Core vocabulary:**

| Term | Plain-English meaning |
|---|---|
| **Scanner** | A background process that checks one specific thing (e.g., "is there an account-level network policy?"). |
| **Scanner Package** | A bundle of related scanners, grouped by theme (e.g., all the CIS benchmark checks). You enable/disable at the package level first. |
| **Finding** | The output when a scanner spots a problem. Two kinds: **Violation** and **Detection** (see Section 3). |
| **Severity** | How bad the finding is: `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`. This drives notification thresholds. |

---

## 3. Violations vs Detections

This is probably **the single most important distinction** in the whole topic — expect it to be tested directly.

| | **Violation** | **Detection** |
|---|---|---|
| What it represents | An ongoing *configuration* problem | A one-time *event* that already happened |
| Based on | Current state of your account right now | Something that occurred at a specific point in time |
| Persists? | Yes — keeps getting reported until you fix the config | No — it's a snapshot of the past |
| Can you "remediate" it? | Yes, by changing the configuration | No direct remediation — the event already happened; you can only *investigate* and prevent recurrence |
| Lifecycle management (mute/reopen)? | **Yes** — Open / Muted status, with Mute and Unmute actions | **No** — you currently cannot manage a detection's lifecycle at all |
| Visible at Organization (org-account) level? | **Yes** | **No** — detections are *never* aggregated to the org level, only violations are |
| Example | "Some users haven't set up MFA" (true today, true tomorrow, until fixed) | "A login came from an unrecognized IP address last Tuesday at 2 AM" |

**Why this design makes sense:** A violation is like an unlocked window — it stays unlocked until someone locks it, so it makes sense to track its state over time (open → muted → gone). A detection is like a doorbell ring at 3 AM — the event is over the instant it happens, so there's nothing to "close." All you can do is decide whether it was meaningful.

**⚠️ Gotcha:** Even if the *same kind* of event happens again, the scanner reports it as a brand-new, separate detection — it doesn't "continue" the old one. And the docs are explicit that a scanner "might or might not" report a similar detection again; there's no guarantee of one detection per unique issue.

**⚠️ Gotcha:** Because a violation is judged only on *current* configuration, if you fix the misconfiguration, the violation doesn't vanish immediately. It stays visible until:
- The scanner package's *next scheduled run*, **or**
- You manually run the scanner package on demand.

So "I fixed it but Trust Center still shows the violation" is expected behavior, not a bug — the scanner just hasn't looked again yet.

---

## 4. Schedule-Based vs Event-Driven Scanners

| | **Schedule-based** | **Event-driven** |
|---|---|---|
| Runs when | At set times, on a configurable schedule | Immediately when a relevant event happens |
| Can you change its schedule? | Yes (usually) | **No** — you can only enable or disable it |
| Reporting delay | Depends on schedule frequency | Reported within about **1 hour** of the event |
| Generates | Can generate either violations or detections | Typically generates detections |
| Appears in `METERING_HISTORY`? | As a single item per run | **May appear as multiple items** for one logical "run" |

**Why both types exist — the classic exam scenario:**

Imagine a scanner checks a `TRUE`/`FALSE` parameter every 10 minutes (schedule-based). If someone flips it to `FALSE` and back to `TRUE` within 9 minutes, the schedule-based scanner never catches it — by the time it looks again, everything's back to normal. It "toggled" underneath the schedule's radar.

An **event-driven** scanner doesn't wait for a clock. It reacts the instant the value changes, so it *would* catch both the `TRUE→FALSE` change and the `FALSE→TRUE` change, even if both happened inside that 10-minute gap.

**⚠️ Exam trap:** If a question describes a security-relevant setting that was changed and changed back quickly, and asks which scanner type would catch it — the answer is **event-driven**, not schedule-based.

---

## 5. The Four Scanner Packages

**⚠️ Gotcha (defaults):** All four packages are **disabled by default**, *except* **Security Essentials**, which is **enabled by default** and cannot be turned off.

**⚠️ Gotcha (enabling a package):** The moment you enable any scanner package, it runs **immediately** — it does not wait for its configured schedule to come around.

### 5.1 Security Essentials

The baseline package. It checks four things:
1. There's an authentication policy that forces human users to enroll in MFA if they log in with a password.
2. Human users are actually enrolled in MFA (not just required to be).
3. There's an account-level network policy restricting access to trusted IPs.
4. If your account has Native App event sharing enabled, an event table is set up to receive those logs.

Key facts:
- **Only scans human users** — user objects with `TYPE` of `PERSON` or `NULL`. Service users are out of scope for this package.
- **Cannot be disabled**, and its **schedule cannot be changed**.
- Runs on its fixed schedule (once a **month**, by default) at **no serverless compute cost**. Running it *any other way* (e.g., on demand) *does* incur incremental serverless compute charges.

**⚠️ Gotcha:** "Free" only applies to the automatic scheduled run. Manually triggering Security Essentials on demand costs credits.

### 5.2 CIS Benchmarks

Checks your account against the community-built **CIS Snowflake Benchmarks** (published externally by the Center for Internet Security). Findings are labeled with the benchmark's section number (e.g., `1.1`).

- Opt-in (disabled by default).
- Runs **once a day** by default — and unlike Security Essentials, **this schedule can be changed**.

**Special case — Section 2 (all of it):** These scanners run complex queries whose violations **don't show up in the Snowsight console at all**. To see them, you must query the findings view directly:

```sql
SELECT start_timestamp, end_timestamp, scanner_id, scanner_short_description,
       impact, severity, total_at_risk_count, AT_RISK_ENTITIES
FROM snowflake.trust_center.findings
WHERE scanner_type = 'Threat' AND completion_status = 'SUCCEEDED'
ORDER BY event_id DESC;
```
**⚠️ Gotcha:** Even though these are CIS *section 2* scanners, the `scanner_type` value to filter on is `'Threat'`, not something CIS-related. Easy detail to get tripped up on.

**The "checks existence, not effectiveness" pattern — a big exam theme:**

Several CIS checks only confirm that *a control exists somewhere*, not that it's actually protecting anything. Think of it like a fire inspector who checks "do you own a fire extinguisher?" without checking if it's charged, unexpired, or even reachable.

| Benchmark | What it actually checks | What it does **NOT** check |
|---|---|---|
| **3.1** | Does an account-level network policy exist? | Whether the *right* IPs are allowed/blocked |
| **4.3** | Is `DATA_RETENTION_TIME_IN_DAYS` = 90 for the account or **at least one** object? | Whether the 90-day setting is applied to data that's actually critical |
| **4.10** | Does **at least one** masking policy exist anywhere in the account? | Whether that masking policy is actually **assigned** to any table or view |
| **4.11** | Does **at least one** row access policy exist anywhere in the account? | Whether that policy is actually **assigned** to any table or view |

**⚠️ Huge gotcha:** You could create a masking policy or row access policy object, never attach it to anything, and 4.10/4.11 would show **no violation** — because the check is "does this object type exist in the account," period.

### 5.3 Threat Intelligence

Opt-in, runs **once a day** by default (schedule changeable). Looks at user types, authentication methods, login activity, and abnormal failure rates.

| Scanner | Type | What it flags |
|---|---|---|
| Migrate human users off password-only sign-in | Schedule | Human users without MFA who signed in with just a password recently, or haven't signed in in 90 days |
| Migrate legacy service users off password-only sign-in | Schedule | Same idea, for legacy service users |
| High volume of authentication failures | Schedule | Users with unusually many auth failures/job errors — could mean attempted takeover, misconfig, or quota/permission issues |
| Authentication policy changes | **Event-driven** | Any change to an authentication policy (account- or user-level) |
| Dormant user sign-ins | **Event-driven** | A sign-in from a user who hadn't signed in for 90+ days |
| Entities with long-running queries | Schedule | Queries running 2 standard deviations beyond the 7-day average duration |
| Login protection | **Event-driven** | Sign-ins from unusual IP addresses (fed by the Malicious IP Protection service — flagged as needing immediate attention) |
| Sensitive parameter protection | **Event-driven** | Only reports when `PREVENT_UNLOAD_TO_INLINE_URL`, `REQUIRE_STORAGE_INTEGRATION_FOR_STAGE_CREATION`, or `REQUIRE_STORAGE_INTEGRATION_FOR_STAGE_OPERATION` gets flipped from `TRUE` to `FALSE` (these are `TRUE` by default for good reason) |
| Users with administrator privileges | Schedule | New users whose default role is admin, or existing users freshly granted an admin role |
| Unusual applications used in sessions | Schedule | Users connecting with unusual client applications |

**⚠️ Gotcha:** "Entities with long-running queries" builds a 30-day cache the first time it runs, which can make that first run cost more — and it reports a detection the very first time it runs (not just once a real anomaly appears).

**⚠️ Gotcha:** "Sensitive parameter protection" only fires on the `TRUE→FALSE` transition — it won't say anything about the steady `TRUE` state or a `FALSE→TRUE` change.

### 5.4 AI Security *(Preview feature)*

For securing AI/agentic workloads — recommended if you use Cortex Agents, Snowflake Intelligence, or Cortex Code. Opt-in, runs **once a day** by default.

| Scanner | Finding type | Severity | What it checks |
|---|---|---|---|
| Cortex Search Service Privileged Roles | Violation | High | Flags Cortex Search Services owned by `ACCOUNTADMIN`, `SECURITYADMIN`, or `GLOBALORGADMIN` — because the service inherits *all* of the owning role's privileges (owner's-rights execution), violating least-privilege |
| Cortex Code CLI PAT usage without role restriction/network policy | Violation | High | Programmatic Access Tokens used with the Cortex Code CLI that lack an `ALLOWED_ROLES` restriction or a compliant network policy |
| Sensitive Data Accessed by Agent | Detection | High | An AI agent accessed a column tagged with a sensitive classification (`SNOWFLAKE.CORE.PRIVACY_CATEGORY` / `SEMANTIC_CATEGORY`) with **no masking or row access policy** applied |
| Cortex AI Guardrails (Advanced Prompt Injection) not enabled | Violation | High | Checks the `advanced_prompt_injection` guardrail in the `AI_SETTINGS` account parameter. **Requires Enterprise Edition or higher** |

**⚠️ Gotcha:** The **AI Security tab** in the UI is separate from the scanner package — the tab exists regardless, but its violations/detections charts stay empty **until you actually enable the AI Security scanner package**.

**Connects forward:** That "Sensitive Data Accessed by Agent" scanner is exactly why masking policies, row access policies, and object tagging (your next topics) matter — this scanner literally checks whether those protections are missing.

---

## 6. Roles & Access Control

**Core fact:** Trust Center access is controlled by **application roles**, not ordinary account roles — `SNOWFLAKE.TRUST_CENTER_VIEWER` and `SNOWFLAKE.TRUST_CENTER_ADMIN`. A user with `ACCOUNTADMIN` must grant one of these application roles to your role before you can use Trust Center.

**⚠️ Gotcha:** In an **organization account**, you grant these using `GLOBALORGADMIN`, not `ACCOUNTADMIN`.

| Task | Tab | Minimum role needed |
|---|---|---|
| View detections | Detections | `TRUST_CENTER_VIEWER` (or `ADMIN`) |
| View violations | Violations | `TRUST_CENTER_VIEWER` (or `ADMIN`) |
| Manage violation lifecycle (mute/unmute) | Violations | `TRUST_CENTER_ADMIN` |
| Manage scanner packages / scanners | Manage scanners | `TRUST_CENTER_ADMIN` |
| View AI Security tab | AI Security | `TRUST_CENTER_VIEWER` (or `ADMIN`) — charts need the AI Security package enabled |
| View org-level violations | Organization | `ORGANIZATION_SECURITY_VIEWER` **+** `TRUST_CENTER_ADMIN`, and **only visible inside an Organization account** |

Example of setting up a split viewer/admin role structure:

```sql
USE ROLE ACCOUNTADMIN;

CREATE ROLE trust_center_admin_role;
GRANT APPLICATION ROLE SNOWFLAKE.TRUST_CENTER_ADMIN TO ROLE trust_center_admin_role;

CREATE ROLE trust_center_viewer_role;
GRANT APPLICATION ROLE SNOWFLAKE.TRUST_CENTER_VIEWER TO ROLE trust_center_viewer_role;

GRANT ROLE trust_center_admin_role TO USER example_admin_user;
GRANT ROLE trust_center_viewer_role TO USER example_nonadmin_user;
```

**⚠️ Gotcha:** From the **Organization** tab you can only *view* org-wide violation counts and drill in — you **cannot** resolve or reopen a violation there. To act on it, you have to sign in to the *specific account* that owns the violation and use its Violations tab.

**Other access notes:**
- Snowflake **reader accounts are not supported** by Trust Center at all.
- Trust Center supports **private connectivity**.

---

## 7. The Trust Center Interface

Quick tour of the Snowsight tabs (found under **Governance & security » Trust Center**):

| Tab | Purpose |
|---|---|
| **Overview** | High-level security posture summary — findings summary, MFA readiness, data security |
| **Violations** | View/filter/triage/mute ongoing configuration problems |
| **Detections** | View one-time flagged events |
| **Manage scanners** | Two sub-tabs: **Scanner packages** (enable/schedule/run) and **Extensions** (install third-party scanner packages) |
| **Data Security** | Set up and monitor sensitive data classification |
| **AI Security** | Dedicated dashboard for AI agent security posture (needs AI Security package enabled to populate) |
| **Organization** *(org accounts only)* | Org-wide violation rollup across all member accounts |

**⚠️ Gotcha — scanner enable/disable hierarchy:** If you disable a *scanner package*, **every scanner inside it gets disabled too — including ones you'd individually enabled**. Package-level state always wins over individual scanner state.

**⚠️ Gotcha — schedule hierarchy (the opposite direction):** If you set a *custom schedule* on an individual scanner, that custom schedule **overrides** the package's schedule from then on — and it **stays overridden** even if you later change the package's schedule. You have to explicitly select "Reset to scanner package schedule" to resync it.

---

## 8. Managing Violations (Lifecycle)

A new violation always starts with status **Open**.

**Muting** a violation:
- Is a *triage* action — you're saying "I've seen this, hide the noise for now," **not** "I fixed it."
- **Stops the periodic email notifications** for that specific violation.
- Does **not** stop the scanner — it keeps running on schedule and will keep internally detecting the same issue as long as the misconfiguration exists. Muting just suppresses what you *see and get notified about*.
- Is reversible any time via **Unmute**.
- Is entirely optional — you never *have* to mute anything for Trust Center to work correctly.

**Note on terminology:** Snowflake's own docs use two different words for related ideas here. The Violations-tab UI uses **Open / Muted** with **Mute / Unmute** buttons. Elsewhere, Snowflake's overview material describes changing a violation's status to **"Resolved"** to suppress notifications. If you see "Resolved" on the exam in this context, treat it as describing the same notification-suppression behavior as "Muted."

**Actually fixing (remediating)** a violation is a *different* action from muting:
- You fix the underlying misconfiguration (e.g., turn on MFA for the flagged users).
- Trust Center **automatically removes** the violation from the tab — no manual "close" step needed.
- **But** this only happens once the scanner *re-runs and confirms* the fix — either on its next scheduled run, or when you run it on demand.

**Cortex Code remediation** *(Preview)*: Selecting **Begin Remediation** on a violation opens an AI chat pre-loaded with that violation's context (type, severity, affected entities). It can explain the issue, suggest fixes, generate SQL for you to review, and even run approved SQL. Requirements: `TRUST_CENTER_ADMIN` application role + `SNOWFLAKE.CORTEX_USER` database role.

**⚠️ Gotcha (precise number, good exam bait):** Even after you remediate and the scanner re-runs, a fixed violation **may still show as Open in Snowsight for up to 3 hours** afterward.

**⚠️ Gotcha:** Some violations need action *outside* Snowflake entirely (e.g., coordinating an org-wide MFA rollout, or investigating whether a login was really suspicious). Cortex Code can explain what to do but **cannot execute those steps for you**.

**⚠️ Gotcha:** Cortex Code remediation is for **violations only**. For detections, it can still help you *investigate*, but there's no "remediation" step because the event is already in the past.

---

## 9. Managing Detections

- You currently **cannot** mute, reopen, or otherwise manage a detection's lifecycle — no status to change.
- Detections are **never rolled up to the Organization tab** — only violations get that org-wide view.
- The Detections tab supports filtering by: **Detection Type** (e.g., Abnormal Account Activities, Insecure Login, Privilege Escalation), **Severity**, **Entity Type** (`QUERY`, `ROLE`, `USER`, etc.), **Reported By** (which scanner package), and **Time Range**.
- Each detection's detail pane has a **Remediation** tab where you can open a worksheet with follow-up investigative queries — "remediation" here really means "investigation," since you can't undo a past event.

---

## 10. Notifications: Email vs Notification Integrations

Trust Center findings can reach you two ways: **native email** (mature, generally available) and **notification integrations** (webhooks/queues — currently a **Preview** feature). Notification integrations sit *on top of* email; they don't replace it.

### 10.1 Email notifications (native)

- Delivered through Snowflake's AWS deployment via **Amazon SES**. Message content may be retained **up to 30 days** for delivery purposes, then deleted.
- **⚠️ Gotcha: Trial accounts cannot use this feature at all.**
- Only goes to users with a **verified email address** in their Snowsight profile.
- **Recipients**, two modes:
  - **Admin users** — Trust Center tries, in order: (1) the org-level security notification contact, (2) if none, the account-level security notification contact, (3) if still none, `ACCOUNTADMIN` users with verified emails.
  - **Custom** — a hand-picked list.
- **⚠️ Gotcha (hard limit): maximum of 50 users** can receive email notifications for a given scanner/package.
- **⚠️ Gotcha (defaults you must memorize):** Out of the box, only **Security Essentials** sends email — to **Admin users**, for findings of **Critical** severity only, and only as often as it runs (**once a month** by default). Every other package/scanner sends **no email at all** until you configure it.
- **⚠️ Gotcha (hierarchy):** Turning off notifications at the *package* level means you can't turn them on for individual scanners inside it — package-level "off" blocks scanner-level "on," same pattern as the enable/disable hierarchy in Section 7. A scanner *can* either inherit the package's trigger/recipients or override them with its own.
- **⚠️ Edge case:** If your organization account and a member account sit in **different deployments**, org-level "Security Updates" email settings configured at the org level won't be visible from that member account.

### 10.2 Notification integrations *(Preview)*

Lets Trust Center push findings to **webhooks** (PagerDuty, Slack, Microsoft Teams) or **queues** (Amazon SNS, Azure Event Grid, Google Pub/Sub) — independent of Snowsight and email.

**⚠️ Gotcha: only OUTBOUND notification integrations are supported.**

**Setup flow:**

1. **Create the integration** with `CREATE NOTIFICATION INTEGRATION`. Example (AWS SNS):
   ```sql
   CREATE NOTIFICATION INTEGRATION test_aws_int
     ENABLED = TRUE
     DIRECTION = OUTBOUND
     TYPE = QUEUE
     NOTIFICATION_PROVIDER = AWS_SNS
     AWS_SNS_TOPIC_ARN = 'arn:aws:sns:us-east-2:1234567890:sns-topic-name'
     AWS_SNS_ROLE_ARN = 'arn:aws:iam::1234567890:role/sns-access-role';
   ```
   For a webhook (e.g., PagerDuty), Trust Center fills a placeholder called `SNOWFLAKE_WEBHOOK_MESSAGE` with the finding's JSON payload — your `WEBHOOK_BODY_TEMPLATE` just needs to reference it.

2. **Grant access — and here's the gotcha:** you grant usage **to the `SNOWFLAKE` application itself**, not to a role:
   ```sql
   GRANT USAGE ON INTEGRATION test_pagerduty_int TO APPLICATION snowflake;
   ```
   If the integration uses a `SECRET` (like a webhook key), the `SNOWFLAKE` application also needs:
   ```sql
   GRANT READ ON SECRET test_db.test_schema.integration_key TO APPLICATION snowflake;
   GRANT USAGE ON DATABASE test_db TO APPLICATION snowflake;
   GRANT USAGE ON SCHEMA test_db.test_schema TO APPLICATION snowflake;
   ```

3. **Configure which findings trigger it**, per scanner or per package, using a stored procedure:
   ```sql
   CALL SNOWFLAKE.TRUST_CENTER.SET_CONFIGURATION(
     'NOTIFICATION_INTEGRATION',
     ARRAY_CONSTRUCT(
       OBJECT_CONSTRUCT(
         'SEVERITY_THRESHOLD', 'HIGH',
         'INTEGRATION_NAME', 'TEST_PAGERDUTY_INT',
         'INCLUDE_AT_RISK_ENTITIES_AND_FINDING_METADATA', 'TRUE'
       )
     )::VARCHAR,
     'CIS_BENCHMARKS'   -- optionally add a specific scanner name as a 4th argument
   );
   ```
   - `SEVERITY_THRESHOLD`: `LOW`, `MEDIUM`, `HIGH`, or `CRITICAL` — notified only at or above this level.
   - `INCLUDE_AT_RISK_ENTITIES_AND_FINDING_METADATA`: **defaults to `FALSE`.** This is a privacy-conscious default — user names, IPs, and other entity details are *left out* of the outbound payload unless you explicitly opt in.
   - You can stack **multiple integrations on the same scanner/package**, each with its own severity threshold — e.g., send everything to SNS, only High+ to PagerDuty, only Critical to Event Grid.
   - Remove a configuration with `SNOWFLAKE.TRUST_CENTER.UNSET_CONFIGURATION(...)`.

4. **Run the scanner** on demand via Snowsight, or with:
   ```sql
   CALL SNOWFLAKE.TRUST_CENTER.EXECUTE_SCANNER('CIS_BENCHMARKS', 'CIS_BENCHMARKS_CIS3_1');
   ```

**⚠️ Gotcha:** An event-driven scanner sends **no** notification if it found nothing. And regardless of scanner type, a notification only fires when the finding's severity **meets or exceeds** your configured threshold.

**Troubleshooting:**
```sql
-- General notification platform delivery log
SELECT * FROM TABLE(SNOWFLAKE.INFORMATION_SCHEMA.NOTIFICATION_HISTORY());

-- Trust-Center-specific notification history
SELECT * FROM SNOWFLAKE.TRUST_CENTER.NOTIFICATION_HISTORY ORDER BY SENT_ON DESC;
```

The payload includes a `finding_identifier` field explicitly meant for **deduplication** on your receiving end.

---

## 11. Monitoring Trust Center Cost

Trust Center scans use **serverless compute**, which shows up in usage views under `service_type = 'TRUST_CENTER'`.

| View | Schema | Role needed |
|---|---|---|
| `METERING_HISTORY` | `ACCOUNT_USAGE` | `ACCOUNTADMIN` or `USAGE_VIEWER` database role |
| `METERING_DAILY_HISTORY` | `ACCOUNT_USAGE` | `ACCOUNTADMIN` or `USAGE_VIEWER` database role |
| `METERING_DAILY_HISTORY` | `ORGANIZATION_USAGE` | `ORGADMIN` or `ORGANIZATION_USAGE_VIEWER` database role |
| `USAGE_IN_CURRENCY_DAILY` | `ORGANIZATION_USAGE` | `ORGADMIN` or `ORGANIZATION_BILLING_VIEWER` database role |

Example:
```sql
SELECT SUM(credits_used) AS total_credits
FROM snowflake.account_usage.metering_history
WHERE service_type = 'TRUST_CENTER'
  AND start_time >= '2024-12-01' AND end_time <= '2024-12-31';
```

**⚠️ Gotcha (ties back to Section 5.1):** Security Essentials on its normal schedule is free. Everything else — CIS Benchmarks, Threat Intelligence, AI Security, any on-demand run, any custom schedule you set — **does** incur credits.

---

## 12. Trust Center Extensions

Extensions let security partners (or your own org) build **custom scanner packages** using the **Snowflake Native App Framework**, then share them privately or list them on the Snowflake Marketplace.

### 12.1 Access control to build/manage extensions
`ACCOUNTADMIN` must grant your role:
- `SNOWFLAKE.TRUST_CENTER_ADMIN` (application role)
- `CREATE APPLICATION PACKAGE`
- `CREATE APPLICATION`

### 12.2 The manifest contract

**⚠️ Gotcha:** The manifest file must be named exactly `tc_extension_manifest.yml` and must live at the **root** of the stage, alongside the Native App's own `manifest.yml`.

```yaml
manifest_version: '2.0'
scanner_packages:
  - id: 'se_extension'
    name: 'Security Extension'
    short_description: '...'
    description: '...'
    scanners:
      - id: 'se_mfa'
        name: 'MFA Required for Users'
        short_description: '...'
        description: '...'
        type: 'VULNERABILITY'
        callback:
          schema: 'security_essentials_mfa_required_for_users_check'
          name: 'scan'
          version: '1.0'
```

Character limits (favorite trivia for exam writers):

| Field | Max length |
|---|---|
| `scanner_packages.id` | 25 (ASCII alphanumeric + underscore only) |
| `scanner_packages.name` | 30 |
| `scanner_packages.short_description` | 150 |
| `scanner_packages.description` | 700 |
| `scanners.id` | 25 (ASCII alphanumeric + underscore only) |
| `scanners.name` | 30 |
| `scanners.short_description` | 150 |
| `scanners.description` | 1,500 |

**⚠️ Gotcha:** `manifest_version` currently only supports `'2.0'`. The *callback's* `version` field is a completely separate thing and currently only supports `'1.0'`. Don't mix these two up.

**⚠️ Gotcha:** `scanners.type` currently only supports `'VULNERABILITY'` — no other scanner types exist yet as options.

**⚠️ Gotcha:** The scanner's stored procedure name in the callback must currently be exactly `scan` — that's a fixed, required name, not just a convention.

### 12.3 The scanner's stored procedure contract

Each scanner is a stored procedure (named `scan`, inside a **versioned schema**) that must return a table with these columns:

| Column | Type | Notes |
|---|---|---|
| `risk_id` | VARCHAR | |
| `risk_name` | VARCHAR | |
| `total_at_risk_count` | NUMBER | `0` if nothing's at risk |
| `scanner_type` | VARCHAR | Only `VULNERABILITY` currently |
| `risk_description` | VARCHAR | |
| `suggested_action` | VARCHAR | |
| `impact` | VARCHAR | |
| `severity` | VARCHAR | `LOW` / `MEDIUM` / `HIGH` / `CRITICAL` |
| `at_risk_entities` | ARRAY of OBJECT | Each object needs `entity_name` and `entity_object_type` (required); `entity_id` is optional; `entity_detail` holds custom data |

**⚠️ Gotcha (exact limits):** Maximum **1,000** at-risk entities, and the **combined size of the array is capped at 128 MB**. Also, the procedure must return **exactly one row per severity + risk_id combination** — that's a strict data contract, not a suggestion.

### 12.4 Wiring it into Trust Center

1. In `setup_script.sql`, create an **application role** (e.g., `trust_center_integration_role`) and grant it `USAGE` on the scanner's schema and procedure.
2. In `manifest.yml`, declare the privilege `IMPORTED PRIVILEGES ON SNOWFLAKE DB` — needed so the extension can read `SNOWFLAKE.ACCOUNT_USAGE` views to actually do its scanning.
3. Create the application package → register a version → create the application (standard Native App flow).
4. **Grant the application role to the platform itself — not to a user role:**
   ```sql
   GRANT APPLICATION ROLE tc_extension.trust_center_integration_role TO APPLICATION snowflake;
   ```
   This mirrors the same pattern from notification integrations (Section 10.2) — Trust Center operates *as* the `SNOWFLAKE` application, so grants for it to actually use your objects go to `APPLICATION snowflake`, not to a role.
5. Register with `SNOWFLAKE.TRUST_CENTER.REGISTER_EXTENSION('APPLICATION PACKAGE', pkg_name, ext_name)` (needs `TRUST_CENTER_ADMIN`). Deregister with `DEREGISTER_EXTENSION`. See what's registered via the `EXTENSIONS` view.
6. Test by querying `SNOWFLAKE.TRUST_CENTER.FINDINGS` — if `COMPLETION_STATUS = 'FAILED'`, check `ERROR_CODE`/`ERROR_MESSAGE`. The most common cause of failure is missing privileges.

### 12.5 Installing someone else's extension (as a consumer)

| | **Public extension (Marketplace)** | **Private extension (private listing)** |
|---|---|---|
| Discovery | Marketplace tab inside Manage Scanners → Extensions | Shared directly with you |
| Registration with Trust Center | **Automatic** | **Manual** — you must call `REGISTER_EXTENSION` yourself after installing, and grant `TRUST_CENTER_INTEGRATION_ROLE` to the `SNOWFLAKE` application |
| Notification on install | Snowsight notification **and** email sent | **No notification or email at all** |

**⚠️ Gotcha:** Either way, newly installed scanner packages are **disabled by default** — you still have to open the package, grant its requested privileges, and enable it.

To see findings from just one extension:
```sql
SELECT * FROM snowflake.trust_center.findings WHERE extension_id = 4486988721;
```
(Find the IDs via the `EXTENSIONS` view.)

---

## 13. Consolidated Comparison Tables

**Violation vs Detection** — see Section 3 (the big one).

**Schedule-based vs Event-driven** — see Section 4.

**TRUST_CENTER_VIEWER vs TRUST_CENTER_ADMIN**

| | VIEWER | ADMIN |
|---|---|---|
| View violations/detections | ✅ | ✅ |
| Mute/unmute violations | ❌ | ✅ |
| Enable/disable/schedule scanners | ❌ | ✅ |
| Manage/register extensions | ❌ | ✅ |

**Email notifications vs Notification integrations**

| | Email | Notification integrations |
|---|---|---|
| Maturity | Generally available | Preview |
| Destinations | Verified user email addresses | Webhooks (PagerDuty, Slack, Teams), Queues (SNS, Event Grid, Pub/Sub) |
| Max recipients | 50 users | No such user cap — goes to a system endpoint |
| Configured via | Snowsight UI (Settings tab) | SQL (`CREATE NOTIFICATION INTEGRATION` + `SET_CONFIGURATION`) |
| Sensitive detail inclusion | N/A | Off by default (`INCLUDE_AT_RISK_ENTITIES_AND_FINDING_METADATA = FALSE`) |
| Default for Security Essentials | On (Critical severity, Admin users) | Off — must be configured manually |

**Public vs Private extensions** — see Section 12.5.

---

## 14. Common Misconceptions

- **"If I fix a violation, it disappears immediately."** No — it disappears only after the scanner *re-runs* and confirms the fix. That could mean waiting for the schedule, or you run it on demand yourself.
- **"Muting a violation means it's resolved."** No — muting only hides notifications. The scanner keeps running and keeps internally flagging the same issue if the config hasn't actually changed.
- **"A CIS violation means my data isn't protected."** Not necessarily — several CIS checks (3.1, 4.3, 4.10, 4.11) only confirm a control's *existence*, not that it's actually applied or effective. A "no violation" result can be misleadingly reassuring.
- **"All scanner packages are free to run."** No — only Security Essentials on its default schedule is free; everything else (and any non-default/on-demand run) costs serverless compute credits.
- **"I can just grant `ACCOUNTADMIN` to access Trust Center."** `ACCOUNTADMIN` is what *grants* the Trust Center application roles, but the actual permission to use Trust Center comes from `SNOWFLAKE.TRUST_CENTER_VIEWER`/`ADMIN` — separate, dedicated application roles.
- **"Detections show up at the org level too, just like violations."** No — only violations roll up to the Organization tab; detections never do.
- **"Disabling one scanner in a package only affects that scanner."** True in one direction, but be careful the other way: disabling the whole *package* disables every scanner inside it, even ones you'd individually turned on.

---

## 15. Exam Trap Cheat Sheet

- Violation = ongoing config problem, has a lifecycle (Open/Muted). Detection = one-time event, **no** lifecycle management, **never** rolls up to org level.
- Event-driven scanners: reported within ~1 hour, schedule **cannot** be changed (enable/disable only), catch fast toggles schedule-based scanners would miss.
- Only **Security Essentials** is on-by-default, non-disable-able, and has a non-changeable schedule.
- Security Essentials scans **human users only** (`TYPE` = `PERSON` or `NULL`).
- CIS 3.1 / 4.3 / 4.10 / 4.11 check *existence*, not *effectiveness or assignment*.
- CIS Section 2 violations don't appear in Snowsight — must query `snowflake.trust_center.findings` with `scanner_type = 'Threat'`.
- AI Security's Guardrails scanner needs **Enterprise Edition or higher**; the other AI Security scanners don't carry that specific restriction.
- Default email notifications: **only** Security Essentials, **only** Critical severity, **only** Admin users, **only** on its monthly schedule. Everything else is opt-in.
- Email notification hard cap: **50 users**. Trial accounts: **can't use email notifications** at all.
- Notification integrations: **outbound only**; grants go **`TO APPLICATION snowflake`**, not to a role; `INCLUDE_AT_RISK_ENTITIES_AND_FINDING_METADATA` defaults to `FALSE`.
- Remediated violation via Cortex Code can still show as Open for **up to 3 hours** post-fix.
- Extension manifest file name and location are fixed: `tc_extension_manifest.yml` at stage root. Scanner type is only `VULNERABILITY`. Stored proc must be named `scan`.
- At-risk entities limit: **1,000** entities, **128 MB** combined array size.
- Public extension install → auto-registered + notified. Private extension install → manual registration, **no** notification at all.
- Org-level **Organization** tab is view/drill-down only — you can't resolve/reopen a violation from there; must sign in to the owning account.
- In an organization account, grant Trust Center app roles using **`GLOBALORGADMIN`**, not `ACCOUNTADMIN`.
- Snowflake **reader accounts** are not supported by Trust Center.

---

## 16. Key Numbers to Memorize

| Number | What it means |
|---|---|
| **50** | Max users who can receive Trust Center email notifications |
| **30 days** | How long AWS SES may retain email content for delivery purposes |
| **90 days** | Standard "inactive user" threshold used across several scanners |
| **~1 hour** | Delay before an event-driven scanner's detection appears |
| **Up to 3 hours** | A remediated violation may still show as Open after Cortex Code fix + re-run |
| **25 chars** | Max length of a scanner/scanner-package `id` in an extension manifest |
| **30 / 150 / 700 / 1,500 chars** | Max lengths for `name` / `short_description` / package `description` / scanner `description` |
| **1,000** | Max at-risk entities returned by a custom scanner |
| **128 MB** | Max combined size of the `at_risk_entities` array |
| **'2.0'** | Only supported `manifest_version` |
| **'1.0'** | Only supported callback `version` |
| **4** | Severity levels: LOW, MEDIUM, HIGH, CRITICAL — and also the number of built-in scanner packages |
| **Monthly** | Security Essentials' default schedule |
| **Daily** | Default schedule for CIS Benchmarks, Threat Intelligence, and AI Security |

---

## 17. Self-Check Questions

**Q1.** A scanner detects that `REQUIRE_STORAGE_INTEGRATION_FOR_STAGE_CREATION` was switched from `TRUE` to `FALSE` at 2:14 PM. What kind of finding is this, and how soon should it appear?
> **Answer:** A **Detection**, from an **event-driven** scanner (Sensitive parameter protection). It should appear within about **1 hour** of the change.

**Q2.** Your account has one row access policy defined, but it isn't attached to any table. Will CIS benchmark 4.11 show a violation?
> **Answer:** **No.** 4.11 only checks whether at least one row access policy *exists* in the account — it doesn't check whether it's actually assigned anywhere.

**Q3.** You mute a violation about missing MFA. A week later, has the underlying scanner stopped checking for that issue?
> **Answer:** **No.** The scanner keeps running on schedule and keeps detecting the issue internally. Muting only suppresses the notifications and hides it from the default view — it doesn't pause the scanner or fix anything.

**Q4.** Which application role must be granted to run `SNOWFLAKE.TRUST_CENTER.SET_CONFIGURATION()` or register an extension?
> **Answer:** `SNOWFLAKE.TRUST_CENTER_ADMIN`. Viewing findings only requires `TRUST_CENTER_VIEWER`, but any configuration/management action requires `ADMIN`.

**Q5.** True or False: Disabling the Threat Intelligence scanner package still lets individually-enabled scanners inside it keep running.
> **Answer: False.** Disabling a package disables every scanner in it, regardless of individual scanner settings.

**Q6.** You've configured a notification integration with `SEVERITY_THRESHOLD = 'HIGH'`. A Low-severity finding occurs. What happens?
> **Answer:** Nothing is sent — notifications only fire when severity **meets or exceeds** the configured threshold.

**Q7.** A finding's payload doesn't include the affected entity's username or IP address, even though `INCLUDE_AT_RISK_ENTITIES_AND_FINDING_METADATA` was never explicitly set. Why?
> **Answer:** It defaults to **`FALSE`** — you must explicitly opt in to include entity-level details in an outbound notification payload.

**Q8.** Can you view org-wide detection totals from the Organization tab in an org account?
> **Answer: No.** Detections are never aggregated at the organization level — only violation totals are shown there.

---

## 18. Sources

- [Trust Center — Overview](https://docs.snowflake.com/en/user-guide/trust-center/overview)
- [Using the Trust Center](https://docs.snowflake.com/en/user-guide/trust-center/using-the-trust-center)
- [Programmatic notifications for Trust Center findings](https://docs.snowflake.com/en/user-guide/trust-center/notification-integrations)
- [Email notifications for Trust Center findings](https://docs.snowflake.com/en/user-guide/trust-center/notifications-trust-center)
- [Using Trust Center extensions](https://docs.snowflake.com/en/user-guide/trust-center/trust-center-extensions)
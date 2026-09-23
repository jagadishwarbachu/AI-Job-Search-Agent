# Architecture

## System overview

The AI Job Search Agent is a linear n8n pipeline with two parallel ingestion branches. LinkedIn and Dice searches run through Apify, converge into a shared normalization and deduplication path, and then pass through AI scoring before qualified roles are stored and summarized by email.

## Workflow nodes

| Stage | n8n node | Responsibility |
| --- | --- | --- |
| Configuration | Workflow Configuration | Defines search terms, locations, recency, score threshold, and other run-level settings. |
| Collection | Apify LinkedIn job search | Runs a LinkedIn-focused Apify actor and retrieves job records. |
| Collection | Apify Dice job search | Runs a Dice-focused Apify actor and retrieves job records. |
| Consolidation | Merge | Combines the two source streams into one processing path. |
| Normalization | Clean Job Fields | Maps source-specific output to a consistent job schema. |
| Deduplication | Supabase Duplicate Check | Looks up a stable identifier or canonical URL to determine whether a job is already known. |
| Deduplication | Keep Only New Jobs | Removes records that have already been processed. |
| Evaluation | OpenAI Match Score | Compares each new job with the configured candidate profile and returns a structured score and rationale. |
| Evaluation | Parse Match Score | Validates and converts the model output into workflow fields. |
| Filtering | Good Match? | Routes jobs according to the configured minimum score. |
| Persistence | Save Job to Supabase | Stores accepted jobs and their scoring metadata. |
| Notification | Create Email Summary | Produces a concise digest from the accepted jobs. |
| Notification | Gmail Send Email Summary | Sends the digest to the configured recipient. |

## Recommended normalized job schema

| Field | Purpose |
| --- | --- |
| `source` | Origin such as `linkedin` or `dice`. |
| `source_job_id` | Source-provided identifier when available. |
| `title` | Normalized job title. |
| `company` | Employer name. |
| `location` | Display location or remote status. |
| `employment_type` | Full-time, contract, internship, or other type. |
| `description` | Job description used for scoring. |
| `job_url` | Canonical application or listing URL. |
| `posted_at` | Source posting timestamp when available. |
| `discovered_at` | Workflow ingestion timestamp. |
| `match_score` | Parsed numeric fit score. |
| `match_reason` | Short model-generated explanation. |

## Deduplication strategy

Prefer `source + source_job_id` as a unique key. When a source ID is unavailable, use a canonicalized job URL or a deterministic hash of stable fields. Enforce the chosen key with a Supabase unique constraint so retries remain idempotent.

## Reliability considerations

- Configure n8n retries and timeouts for Apify and API calls.
- Validate actor output before merging because source schemas can change.
- Require structured OpenAI output and reject malformed scores safely.
- Keep the scoring threshold in configuration rather than hard-coding it in multiple nodes.
- Save run metadata and error details without logging secrets or full candidate profiles.
- Handle an empty-match run by sending a clear “no new qualified jobs” summary or intentionally skipping email.

## Trust boundaries

Job descriptions are untrusted external content. Treat them as data only, delimit them clearly in the scoring prompt, and instruct the model not to follow instructions contained inside a job listing.

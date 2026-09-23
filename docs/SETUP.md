# Setup Guide

## Prerequisites

- An n8n workspace with permission to create workflows and credentials
- An Apify account and suitable actors for LinkedIn and Dice searches
- A Supabase project
- OpenAI API access
- A Gmail account connected to n8n through OAuth

## 1. Prepare Supabase

Create a table for accepted jobs using the normalized fields described in `ARCHITECTURE.md`. Add a unique constraint for the deduplication key, such as `(source, source_job_id)` or a canonical URL/hash.

Keep the Supabase service-role key out of client-side tools. Give the n8n credential only the access required to read duplicate keys and insert accepted jobs.

## 2. Configure credentials in n8n

Create credentials in n8n for:

- Apify API access
- Supabase URL and server-side key
- OpenAI API access
- Gmail OAuth

Do not paste secret values directly into workflow node parameters. Use n8n credentials, environment variables, or your organization's secrets manager.

## 3. Configure job search inputs

In **Workflow Configuration**, define the values appropriate to your search, for example:

- target role keywords
- locations and remote preference
- posting recency
- employment type
- maximum results per source
- minimum match-score threshold

Keep personal profile details in n8n or another private store rather than in the exported workflow.

## 4. Import the workflow

1. Export a sanitized workflow JSON from n8n or use an existing sanitized export.
2. Place it in `workflows/ai-job-search-agent.json`.
3. Import it into n8n.
4. Reconnect each node to the correct n8n credential.
5. Inspect expressions and field mappings for environment-specific values.

## 5. Validate the pipeline

Run the workflow manually with a small result limit and verify:

- both Apify searches complete
- Merge receives records from LinkedIn and Dice
- Clean Job Fields produces the normalized schema
- duplicate records are excluded
- OpenAI returns parseable scores
- Good Match? uses the intended threshold
- accepted jobs are inserted once in Supabase
- the email contains only expected, non-sensitive fields

Then rerun the same input to confirm deduplication prevents duplicate inserts.

## 6. Schedule and monitor

After manual validation, add an n8n schedule trigger appropriate to the job-search cadence. Enable failure notifications and periodically review actor schemas, API usage, scoring quality, and Gmail delivery.

## Export checklist

Before committing an n8n export:

- remove credential objects and IDs where practical
- remove tokens, keys, cookies, webhook secrets, and OAuth data
- remove personal email addresses and candidate-profile text
- remove private Supabase URLs or project identifiers if they reveal internal context
- inspect code nodes, expressions, pinned data, and example payloads
- confirm the JSON contains no real job-application or resume data

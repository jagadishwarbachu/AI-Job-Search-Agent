# Workflow exports

Place the sanitized n8n workflow export here as:

```text
workflows/ai-job-search-agent.json
```

The production export is intentionally not committed yet. Before adding it, review the JSON carefully and remove:

- API keys, tokens, cookies, OAuth material, and webhook secrets
- credential objects and sensitive credential identifiers
- personal email addresses, resume text, and candidate-profile details
- private Supabase project identifiers or internal URLs
- pinned execution data and real job records

After importing the sanitized file, reconnect credentials inside n8n and test with a small result set before enabling a schedule.

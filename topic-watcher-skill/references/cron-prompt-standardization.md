# Cron Prompt Standardization

When monthly watch report jobs were created at different times, they may have divergent prompt formats (old "Justification" format vs new "Signal & Fracture" format). To align all jobs to the same template:

## Detection

1. List all cron jobs: `cronjob(action='list')`
2. Identify monthly watch report jobs (name contains "Monthly Watch Report" or "monthly-report")
3. Check the prompt structure — look for the Step 4 structure section in each job's prompt. Detect which format it specifies:
   - **Old (Justification):** "Step 4: Write/update the watch report... **Justification**: Political, Economic, Military, Technological analysis" — no Metadata, Signal & Fracture, Watch Indicators, or Probability Triggers
   - **New (Signal & Fracture):** "Step 4: Write/update the watch report... with the Signal & Fracture format: ## Metadata, ## Signal & Fracture, ## Prediction, ## Analysis, ## Watch Indicators, ## Probability Triggers"
   - **Mixed:** Has some new sections but lacks others (e.g., has `---` separators but no S&F)

## Batch Update Procedure

Rather than updating each job individually via `cronjob(action='update')`, batch-edit `~/.hermes/cron/jobs.json` directly:

1. **Build the standardized prompt template** with placeholders for `{topic_name}`, `{slug}`, `{geography}`. Include:
   - Cron delivery header
   - Topic-specific info (topic name, slug, repo)
   - Step 1-3: Read wiki, identify changes, decide new vs update
   - Step 4: Write in Signal & Fracture format (Metadata, S&F, Prediction, What's New, Analysis, Watch Indicators, Probability Triggers, Key Sources, Disclaimer, Notes — all with `---` separators)
   - Step 4b: Review prompt (mandatory)
   - Step 5: Update index.md
   - Step 5b-5c: Update per-topic + global summaries
   - Step 6: Update log.md
   - Step 7: Generate PDF
   - Step 8: Commit and push (with scoped `git add {slug}/`)
   - Step 9: Report back
   - IMPORTANT RULES section

2. **Map each job ID to its parameters** (topic_name, slug, geography) — get from `_config.yaml`

3. **Write a Python script** that reads `jobs.json`, matches job IDs, replaces each prompt, and writes back:

```python
import json

with open('/root/.hermes/cron/jobs.json') as f:
    data = json.load(f)

template = """[standardized prompt with {placeholders}]"""

job_configs = {
    "90f1ad1706cb": ("Sri Lankan Financial Relationship with China", "sri-lanka-china", "Sri Lanka, Indian Ocean"),
    # ... etc
}

for job in data['jobs']:
    jid = job['id']
    if jid in job_configs:
        topic_name, slug, geography = job_configs[jid]
        job['prompt'] = template.format(topic_name=topic_name, slug=slug, geography=geography)

with open('/root/.hermes/cron/jobs.json', 'w') as f:
    json.dump(data, f, indent=2)
```

4. **Verify** by listing jobs again — each should have the standardized structure.

## After Standardization

- Existing watch reports from previous runs are NOT retroactively changed — only future cron runs use the new prompt.
- If a previous report already exists in the old format (e.g., China-Southern Africa 02-07-2026), manually rewrite it to the new format if a fresh PDF is needed.
- The master template lives at `references/monthly-watch-report-prompt.md` — keep this as the canonical source. If you update the master template, re-run the batch standardization to propagate changes.

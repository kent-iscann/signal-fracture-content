# Cron Failure Recovery

When a wiki source monitor cron job fails, use this procedure for manual recovery.

## Diagnosis

1. List cron output directories: `ls -lt ~/.hermes/cron/output/<job_id>/`
2. Read the latest output file (by date in filename)
3. Check for these failure types:
   - **HTTP 429** — rate limit. Re-run with delays between searches.
   - **RuntimeError / connection error** — transient. Re-run as-is.
   - **Partial output** — some sources were added before the crash. Check git log to see if a partial commit landed.

## Recovery Steps

1. Read existing `sources.md` for dedup context (don't re-add what's already there)
2. Re-run searches with 3-5s delays between each
3. Extract promising new sources via `tavily_extract`
4. Dedup against existing entries (same core facts = skip, even if different URL)
5. Update: `sources.md` → `timeline/` → `concepts/` or `entities/` → `index.md` → `log.md`
6. Commit and push: `cd /root/wiki && git add -A && git commit -m "..." && git push`
7. Report: number of new sources, any skipped (dupes/errors)

## Key Insight

Cron job status (`ok`) only means the process didn't crash — it does NOT mean the job succeeded. Always read the actual output file to confirm.

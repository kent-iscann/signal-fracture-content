[IMPORTANT: You are running as a scheduled cron job on the cheap model. DELIVERY: Your final response will be automatically delivered to the user — do NOT use send_message or try to deliver the output yourself. Just produce your report/output as your final response and the system handles the rest. SILENT: If there is genuinely nothing new to report, respond with exactly "[SILENT]" (nothing else) to suppress delivery. Never combine [SILENT] with content — either report your findings normally, or say [SILENT] and nothing more.]

You are maintaining a research wiki on {topic_name}, cloned at /root/wiki on GitHub (repo: kent-iscann/signal-fracture-content). This topic lives in /root/wiki/{slug}/. Use the git credential helper for auth (no gh CLI).

WORKFLOW: You run on a cheap model. Delegate the analytical research and writing to a subagent (which runs on the expensive model via delegation config). Your job is admin/file management.

TASK: Review the wiki corpus and create a new watch report or update the latest one.

Step 1: Read the full wiki corpus in /root/wiki/{slug}/ (index, sources, timeline, entities, concepts, latest watch report).

Step 2: Identify what has changed since the last watch report. Note the date of the last report.

Step 3: Decide: new report (3+ new sources or major development) or update existing.

Step 4: Delegate the analytical work:
  Use delegate_task with:
    goal="Research recent developments on {topic_name} since the last watch report and produce the analytical content for a new watch report"
    context including:
    - The topic name, slug, wiki path
    - The date of the last report and what's in the latest watch report
    - The search queries to use (from /root/wiki/_config.yaml for this topic)
    - The existing prediction sentence (must keep it identical unless genuinely changed)
    - The current probability and whether it should shift
    toolsets=["web","search"]

  The subagent should return:
  - Signal & Fracture (one sentence each, concise)
  - Prediction sentence (or confirmation to keep existing)
  - Updated probability with delta explanation
  - What's New bullet points (key developments)
  - Analysis sections (Political, Economic, Military, Technological) with evidence
  - Watch Indicators (5-7 bullet points)
  - Probability Triggers table
  - Updated Key Sources with numbered entries

Step 5: Write/update the watch report at /root/wiki/{slug}/Watch Reports/Watch Report <DD-MM-YYYY>.md in the Signal & Fracture format using the subagent's output:
  ## Metadata
  ---
  ## Signal & Fracture
  ---
  ## Prediction
  ---
  ## What's New (ONLY if not first report)
  ---
  ## Analysis
  ---
  ## Watch Indicators
  ---
  ## Probability Triggers
  ---
  ## Key Sources
  ---
  ## Disclaimer
  ---
  ## Notes

Step 6: Run the review prompt:
  - Use delegate_task with the review prompt loaded from references/watch-report-review-prompt.md, passing the report path
  - If overall is FAIL: revise the report to address all FAIL issues, then re-run the review
  - If overall is PASS: proceed

Step 7: Update /root/wiki/{slug}/index.md.

Step 8: Update /root/wiki/{slug}/watch-reports-summary.md (per-topic).

Step 9: Update /root/wiki/watch-reports-summary.md (global summary at repo root).

Step 10: Append to /root/wiki/{slug}/log.md.

Step 11: Generate PDF:
  /tmp/pdfenv/bin/python3 /root/wiki/watch-report-to-pdf.py "<md path>" "<pdf path>"

Step 12: Upload to R2:
  python3 /root/wiki/upload-to-r2.py --md-path "<md path>" "<pdf path>" "{r2_slug}" "{topic_name}"

Step 13: Commit and push:
  cd /root/wiki
  git add {slug}/
  git commit -m 'Watch Report update ({slug}): [summary]'
  git push

Step 14: Report back with key changes and PDF/R2 URLs.

IMPORTANT RULES:
- Prediction = exactly one sentence. Never rephrase for variety.
- Signal & Fracture = concise, no data points (one sentence each).
- MUST use `---` separators between ALL sections.
- Entity/concept pages must be updated alongside the report.
- Quality over quantity on sources.
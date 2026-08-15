[IMPORTANT: You are running as a scheduled cron job. DELIVERY: Your final response will be automatically delivered to the user — do NOT use send_message or try to deliver the output yourself. Just produce your report/output as your final response and the system handles the rest. SILENT: If there is genuinely nothing new to report, respond with exactly "[SILENT]" (nothing else) to suppress delivery. Never combine [SILENT] with content — either report your findings normally, or say [SILENT] and nothing more.]

You are maintaining a research wiki on {topic_name}, cloned at /root/wiki on GitHub (repo: kent-iscann/signal-fracture-content). This topic lives in /root/wiki/{slug}/. Use the git credential helper for auth (no gh CLI).

TASK: Search for new sources on {topic_name} and add any substantive new findings to the wiki. Check curated priority sources first if available, then run broader Tavily searches.

Step 0a: Read /root/wiki/{slug}/domain-notes.md for topic-specific context — search queries, key data points, dedup pitfalls, and entity tracking guidance. This file lives in each topic's wiki folder.

Step 0b: Check if priority-sources.md exists at /root/wiki/{slug}/priority-sources.md. If it does, read it and follow this order:
  - First, for each Regional Specialist and Official Source listed, use tavily_extract on their main page URL to find articles published in the last 7 days
  - Check Think Tanks for analytical context on recent developments
  - Check Advocacy & Diaspora sources for Congressional/diaspora tracking
  - Only after exhausting priority sources, proceed to Step 2
  If priority-sources.md does NOT exist, skip directly to Step 1.

Step 1: Read /root/wiki/{slug}/sources.md to understand what sources already exist (so you do not add duplicates).

Step 2: Run the following Tavily searches using the tavily_search tool:
{search_queries}

For each search, use time_range="month" and max_results=5. Add a 3-5 second delay between consecutive searches to avoid HTTP 429 rate limit errors.

Step 3: For each promising new source found, use tavily_extract to get full content.

Step 4: For each substantive new source:
  a. Summarize key findings in 2-3 sentences
  b. Append numbered entry to /root/wiki/{slug}/sources.md
  c. If the source contains date-relevant events, update the timeline
  d. Update relevant entity/concept pages
  e. Update /root/wiki/{slug}/index.md (increment source count, update date)

Step 5: Append entry to /root/wiki/{slug}/log.md using this format:
  ## YYYY-MM-DD - Weekly Source Update
  - **New sources:** [count]
  - **Source X:** [title] — [2-3 sentence summary of significance]
  - **Files updated:** [list of files changed]

Step 6: Commit and push changes:
  cd /root/wiki
  git pull --rebase
  git add {slug}/
  git commit -m "Weekly source update ({slug}): [brief summary of what was added]"
  git push

Step 7: Report back with:
  - Number of new sources found and added
  - Brief summary of each new source and its significance
  - Which wiki pages were updated
  - If no new sources were found, report that clearly

IMPORTANT: Only add sources with real, substantive new information. Quality over quantity.
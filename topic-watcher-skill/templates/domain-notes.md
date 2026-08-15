# {Topic Name} — Domain Notes

## Wiki Location
- Path: `/root/wiki/{slug}/`
- Repo: `kent-iscann/signal-fracture-content`
- Git auth: credential helper (no gh CLI needed)

## Folder Structure
```
/root/wiki/{slug}/
├── index.md              (source count + last-updated date + page catalog)
├── sources.md            (numbered entries or table)
├── domain-notes.md       (this file — topic context, search queries, dedup tips)
├── priority-sources.md   (curated high-value outlets)
├── log.md                (append-only session entries)
├── timeline/
│   └── {slug}-timeline.md
├── entities/
├── concepts/
├── Watch Reports/
└── watch-reports-summary.md
```

## Search Queries (canonical)
1. "{query 1}"
2. "{query 2}"
3. "{query 3}"
4. "{query 4}"
5. "{query 5}"

## Key Entities Tracked
| Entity | Role | Key Actors |
|--------|------|------------|

## Key Data Points
- 

## Common Dedup Pitfalls
- 

## High-Signal Source Types
- 

## Prediction History
| Date | Probability | Δ | Key Driver |
|------|-------------|---|------------|

## File Update Checklist (per source added)
1. ✅ `sources.md` — append numbered entry
2. ✅ `timeline/{slug}-timeline.md` — add event if date-relevant
3. ✅ `entities/<entity>.md` — update if source mentions specific actor
4. ✅ `concepts/<concept>.md` — update if source relates to a tracked concept
5. ✅ `index.md` — increment source count, update date
6. ✅ `log.md` — append session entry
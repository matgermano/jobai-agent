# /hunt-jobs

## Goal
Search for remote Product Manager job listings. Extract data from search result snippets only. Do NOT fetch individual job pages. Maximum 2 fetches total per run.

## Token budget rules (STRICT)
- Maximum 4 web searches per run — stop when done, do not explore beyond what is needed
- Maximum 2 page fetches per run — only listing pages, never individual job URLs
- Extract all job data from search snippets and listing pages only
- Do NOT follow links to individual job postings
- Do NOT fetch ziprecruiter, greenhouse, lever, or any ATS pages

## Steps

### 1. Load context (do not re-read if already loaded)
- Read config.json
- Read data/cv.md

### 2. Run these 4 searches — do not add more
Stop after all 4 are complete. Do not run bonus searches.
```
Search 1: site:remoteok.com "product manager" remote worldwide -"US only" -"United States only"
Search 2: site:remotive.com "product manager" worldwide remote -"US only"
Search 3: site:himalayas.app "product manager" remote worldwide Brazil
Search 4: site:wellfound.com "product manager" remote worldwide -"US only"
```

### 3. Extract from search snippets only
For each result in the search snippets, extract WITHOUT fetching the page:
- title (from snippet title)
- company (from snippet)
- url (from search result)
- source (which site)
- location_type (from snippet text)
- salary_range (from snippet if shown, otherwise "not listed")
- 3–5 key requirements (from snippet text only)

### 4. Apply location filter (hard rules)
Reject if snippet contains ANY of:
- "US only", "US citizens", "United States only"
- "EU only", "Europe only"
- "requires relocation", "on-site", "hybrid"

Accept if snippet contains:
- "worldwide", "global", "anywhere", "LATAM", "Americas"
- No location restriction mentioned

### 5. Calculate fit score (0–10) from snippet data only
- Role match: +0–3
- Salary in $70k–$100k range: +2 (unknown = +1)
- Location confirmed worldwide: +2
- Keyword overlap with cv.md: +0–2
- Seniority match: +0–1

Only keep fit_score >= `search.min_fit_score` from config.json (default 5).

### 6. Save output

**Daily output:**
Create `outputs/jobs_YYYY-MM-DD.json` with this schema:
```json
[{
  "title": "",
  "company": "",
  "url": "",
  "source": "",
  "location_type": "",
  "salary_range": "",
  "fit_score": 0,
  "key_requirements": [],
  "found_at": "YYYY-MM-DDTHH:MM:SS"
}]
```

**Main knowledge base (with deduplication + TTL):**
Read `data/knowledge_base.json` (or create as [] if not exists).
Apply TTL: remove any entry where `found_at` is older than 90 days from today.
Then deduplicate: filter out any entries whose `url` already exists in the (already-pruned) file.
Append only new (non-duplicate) jobs, then write back.

**Weekly knowledge base slice:**
Determine the current ISO week: YYYY-WNN (e.g. 2026-W21).
Read `data/kb_YYYY-WNN.json` (or create as [] if not exists).
Same URL-based dedup before appending (no TTL on weekly slices — they are archival).
Write back to `data/kb_YYYY-WNN.json`.

### 7. Email alert (requires Gmail MCP)
Check if any jobs in today's filtered list have `fit_score >= 9`.

**If none:** skip this step entirely.

**If any exist:** use the Gmail MCP to send one email to the address in `config.json` (`profile.email`):
- **To:** `profile.email` from config.json
- **Subject:** `jobAI Alert — X job(s) scored 9+ today (YYYY-MM-DD)`
- **Body:**
```
Hi,

The job hunter found X job(s) scoring 9 or 10 today.

[For each job with fit_score >= 9:]
──────────────────────────────
Title: [title]
Company: [company]
Score: [fit_score]/10
URL: [url]
Salary: [salary_range]
Key requirements: [key_requirements joined by ", "]
──────────────────────────────

Full results: outputs/jobs_YYYY-MM-DD.json

jobAI · running automatically for you
```

If Gmail MCP is not connected, skip this step and add a note to the print summary.

### 8. Git commit
```
git add outputs/ data/
git commit -m "feat(hunt): YYYY-MM-DD · X jobs found · job-hunter"
git pull --rebase origin main
git push origin main
```

### 9. Print summary — then STOP
```
✅ Job Hunt — YYYY-MM-DD
Searches run: X | Pages fetched: X
Jobs found: X | Passed filter: X | New (non-duplicate): X
Score 9+: X (email sent) OR Score 9+: 0 (no alert)
Top matches:
  1. [Title] @ [Company] — score X/10 — [URL]
  2. ...
Saved: outputs/jobs_YYYY-MM-DD.json
Weekly KB: data/kb_YYYY-WNN.json
```

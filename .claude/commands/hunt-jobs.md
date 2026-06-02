# /hunt-jobs

## Goal
Search for remote job listings. Score each one. For every job scoring 9 or 10, automatically generate a full application package. Send one email with everything ready.

## Token budget rules (STRICT)

**Phase 1 — Search (hard ceiling, do not exceed):**
- Maximum 4 web searches
- Maximum 2 page fetches (listing pages only, never individual job URLs)
- Extract all job data from snippets only

**Phase 2 — Apply-prep for 9+ jobs (per job, only if any scored 9+):**
- 1 fetch for the job URL (required)
- Maximum 2 web searches for company research
- Maximum 1 additional fetch for company site (only if needed)
- Maximum 3 jobs processed per run to keep total calls bounded

## Steps

### 1. Load context (do not re-read if already loaded)
- Read `config.json`
- Read `data/cv.md`
- Read `application_profile.json` (interests alignment scoring + apply-prep inputs)
- Read `application_narratives.md` (needed for apply-prep step)

### 2. Run these 4 searches — do not add more
Stop after all 4 are complete. Do not run bonus searches.
```
Search 1: site:remoteok.com "product manager" remote worldwide -"US only" -"United States only"
Search 2: site:remotive.com "product manager" worldwide remote -"US only"
Search 3: site:himalayas.app "product manager" remote worldwide Brazil
Search 4: site:wellfound.com "product manager" remote worldwide -"US only"
```

### 3. Extract from search snippets only
For each result extract WITHOUT fetching the page:
- title, company, url, source, location_type
- salary_range (from snippet if shown, otherwise "not listed")
- 3–5 key_requirements (from snippet text only)

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
- Interest alignment: +1 if role/requirements match `interests.want_to_do` keywords (AI, automation, agentic, product strategy, greenfield); -1 if matches `interests.want_to_avoid`

Only keep fit_score >= `search.min_fit_score` from config.json (default 5).

### 6. Save daily output and update knowledge base

**Daily output file:**
Save `outputs/jobs_YYYY-MM-DD.json`:
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
  "found_at": "YYYY-MM-DDTHH:MM:SSZ"
}]
```

**Main knowledge base:**
Read `data/knowledge_base.json` (or create as `[]`).
Apply TTL: remove entries where `found_at` is older than 90 days.
Deduplicate: remove entries whose `url` already exists.
Append new jobs. Write back.

**Weekly KB slice:**
Read `data/kb_YYYY-WNN.json` (or create as `[]`). Same URL dedup, no TTL. Write back.

### 7. Generate apply-prep packages for 9+ scoring jobs

Check today's filtered jobs for fit_score >= 9. If none: skip to step 8.

**For each job with fit_score >= 9 (max 3 jobs):**

Create company slug: lowercase company name, hyphens, no special characters.

**7a. Fetch the job (1 fetch)**
Fetch the job URL. Extract:
- Full job title and company name
- Key requirements (5–10 items)
- Nice-to-haves (3–5 items)
- Team / product area

**7b. Research the company (max 2 searches + 1 optional fetch)**
Run up to 2 searches: `"[Company] product what they do culture"` and `"[Company] recent news funding 2026"`.
One additional fetch of the company website if a critical fact is still missing.

**7c. Generate tailored CV**
Compare job requirements vs `data/cv.md`:
- Strong match: requirement clearly present with evidence → verify it's visible
- Weak match: experience exists but not keyword-aligned → rewrite the bullet using the job's language
- Gap: not present → note it, never add it

Rewrite the Professional Summary and any weak-match bullets. Preserve ALL metrics, dates, company names, titles. Add no new bullet points.
Save: `outputs/apply-prep/[company-slug]/cv_[company-slug].md`

**7d. Company briefing**
Save: `outputs/apply-prep/[company-slug]/company_briefing.md`
```
# Company Briefing — [Company] — YYYY-MM-DD
## What they do
## Product / platform (what you'd be working on)
## Culture signals
## Recent news
## Why this role fits your background (specific, not generic — cite real metrics)
## Questions to ask in the interview (3–5, grounded in research)
```

**7e. Open question drafts**
Save: `outputs/apply-prep/[company-slug]/open_questions.md`
Draft answers to these 5 questions using only `application_narratives.md`, tailored to this role:
1. "Why are you interested in this role / company?"
2. "Tell me about yourself" (30-second version)
3. "What's your biggest strength relevant to this role?"
4. "Describe a time you led a cross-functional initiative"
5. "Where do you see yourself in 3 years?"
Each answer: 3–4 sentences, candidate's voice, real experience only.

**7f. STAR story matching**
Save: `outputs/apply-prep/[company-slug]/star_stories.md`
From `application_narratives.md`, pick the 2–3 most relevant stories for this role's key requirements.
For each: story title, why it fits this role, key metric to lead with, how to open.

### 8. Email alert (requires Gmail MCP)

Check if any 9+ jobs exist. If none: skip.

Send one email to `config.json profile.email`:
- **Subject:** `jobAI Alert — X job(s) scored 9+ today (YYYY-MM-DD)`
- **Body:**
```
Hi,

The job hunter found X job(s) scoring 9 or 10 today.
Application packages are ready — review them before applying.

[For each job with fit_score >= 9:]
──────────────────────────────
Title: [title]
Company: [company]
Score: [fit_score]/10
URL: [url]
Salary: [salary_range]

Application package:
  CV:        outputs/apply-prep/[slug]/cv_[slug].md
  Briefing:  outputs/apply-prep/[slug]/company_briefing.md
  Questions: outputs/apply-prep/[slug]/open_questions.md
  STAR:      outputs/apply-prep/[slug]/star_stories.md

Next steps:
  1. Review the package
  2. Run /apply-ats [url] to fill the ATS form (Playwright auto-fills)
  3. Review the form in the browser and click Submit
  4. Run /log-application [url] to track it
──────────────────────────────

jobAI · running automatically for you
```

If Gmail MCP is not connected, skip and note it in the summary.

### 9. Git commit
```
git add outputs/ data/
git commit -m "feat(hunt): YYYY-MM-DD · X jobs found · Y packages generated · job-hunter"
git pull --rebase origin main
git push origin main
```

### 10. Print summary — then STOP
```
✅ Job Hunt — YYYY-MM-DD
Searches run: X | Pages fetched: X
Jobs found: X | Passed filter: X | New (non-duplicate): X
Score 9+: X → packages generated: [list of company slugs]
Email: sent / skipped (Gmail not connected)
```
STOP after printing.

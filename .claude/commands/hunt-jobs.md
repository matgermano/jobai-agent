# /hunt-jobs

## Goal
Search for remote job listings across 9 sources. Score each one. For every job scoring 9 or 10, automatically generate a full application package. Send one email with everything ready.

## Token budget rules (STRICT)

**Phase 1 — Search:**
- Maximum 12 web searches (covering all sources and role types)
- Maximum 4 page fetches (listing pages only — never individual job URLs, never ATS pages)
- Extract data from snippets first; only fetch a listing page if snippets are insufficient for that source

**Phase 2 — Apply-prep for 9+ jobs (only if any scored 9+):**
- 1 fetch for the job URL (required)
- Maximum 2 web searches for company research
- Maximum 1 additional fetch for company site
- Maximum 3 jobs processed per run

## Steps

### 1. Load context
- Read `config.json` — profile, target roles, scoring rules
- Read `data/cv.md` — source of truth for scoring and apply-prep
- Read `application_profile.json` — interests alignment + apply-prep inputs
- Read `application_narratives.md` — needed for apply-prep STAR stories

### 2. Run all 12 searches

**Block A — Product Manager roles (5 searches):**
```
Search A1: site:remoteok.com "product manager" remote worldwide -"US only" -"United States only"
Search A2: site:remotive.com "product manager" worldwide remote -"US only"
Search A3: site:himalayas.app "product manager" remote worldwide
Search A4: site:wellfound.com "product manager" remote worldwide -"US only"
Search A5: site:weworkremotely.com "product manager" remote -"US only"
```

**Block B — LinkedIn and aggregators (3 searches):**
```
Search B1: site:linkedin.com/jobs "product manager" remote worldwide -"US only" -"United States only"
Search B2: site:remoterocketship.com "product manager" OR "head of product" remote -"US only"
Search B3: site:jobnagringa.com.br "product manager" remote
```

**Block C — AI, Automation, and adjacent roles (4 searches):**
```
Search C1: ("AI builder" OR "AI engineer" OR "AI product manager") remote worldwide -"US only"
Search C2: ("automation specialist" OR "process transformation" OR "solutions engineer") remote worldwide -"US only"
Search C3: site:turing.com "product manager" OR "AI engineer" remote
Search C4: site:himalayas.app ("AI engineer" OR "automation" OR "solutions engineer") remote worldwide
```

Run all 12. Do not skip any. Do not add more.

### 3. Extract from search snippets
For each result, extract WITHOUT fetching the page:
- title, company, url, source, location_type
- salary_range (from snippet if shown, otherwise "not listed")
- 3–5 key_requirements (from snippet text only)

For sources where snippets are thin (LinkedIn, Job na Gringa, Turing), you may use 1 page fetch to get a listing page — counts against the 4-fetch budget.

### 4. Apply location filter (hard rules — zero exceptions)
**Reject immediately if** snippet contains ANY of:
- "US only", "US citizens only", "United States only", "must be based in the US"
- "EU only", "Europe only", "must be in Europe"
- "requires relocation", "on-site", "hybrid", "in-office"
- "visa sponsorship required" (means they need you to have specific visa)

**Accept if** snippet contains:
- "worldwide", "global", "anywhere", "LATAM", "Americas", "UTC-3"
- No location restriction mentioned
- "remote" with no country restriction

### 5. Calculate fit score (0–10)
- Role match with `config.json target_roles`: +0–3
  - Primary roles (PM, Senior PM, etc.): up to +3
  - Secondary roles (AI Builder, Automation Specialist): up to +2
  - Monitor-only roles (Head of Product, Product Lead): score but flag as monitor-only
- Salary in $70k–$100k range: +2 (unknown or unlisted = +1)
- Location confirmed worldwide/LATAM: +2
- Keyword overlap with `data/cv.md` skills section: +0–2
- Seniority match: +0–1
- Interest alignment from `application_profile.json interests`:
  - Matches `want_to_do` keywords (AI, automation, agentic, product strategy, greenfield): +1
  - Matches `want_to_avoid` keywords (coordination only, compliance only, governance only): -1

**Thresholds:**
- fit_score < `search.min_fit_score` (default 5): discard
- fit_score 5–8: save to KB, include in weekly report
- fit_score 9–10: save to KB + generate apply-prep package + send email alert

### 6. Save daily output and update knowledge base

**Daily output:** `outputs/jobs_YYYY-MM-DD.json`
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

**Main knowledge base:** `data/knowledge_base.json`
- Apply TTL: remove entries older than 90 days
- Deduplicate by URL
- Append new jobs. Write back.

**Weekly slice:** `data/kb_YYYY-WNN.json`
- URL dedup only (no TTL on slices)
- Append new jobs. Write back.

### 7. Generate apply-prep packages for 9+ scoring jobs

If no jobs scored 9+: skip to step 8.

**For each 9+ job (max 3 per run):**

Compute company slug: lowercase, hyphens, no special characters.

**7a. Fetch job description (1 fetch)**
Extract: title, company, full requirements (hard + nice-to-have), team context.

**7b. Research company (max 2 searches + 1 optional fetch)**
```
Search: [Company] product what they do culture remote
Search: [Company] recent news funding 2026
```
Optional: fetch company website if a critical fact is still missing after searches.

**7c. Tailored CV** → `outputs/apply-prep/[slug]/cv_[slug].md`
Compare requirements vs `data/cv.md`. Classify each requirement: strong match / weak match / gap.
- Strong match: verify it's clearly visible
- Weak match: rewrite the bullet using the job's exact vocabulary — same experience, better alignment
- Gap: note it, never add it
Rewrite the Professional Summary. Preserve ALL metrics, dates, company names, titles. No new bullets.

**7d. Company briefing** → `outputs/apply-prep/[slug]/company_briefing.md`
```
# Company Briefing — [Company] — YYYY-MM-DD
## What they do
## Product / platform (what you'd be working on)
## Culture signals
## Recent news
## Why this role fits your background (specific — cite real metrics from cv.md)
## Questions to ask in the interview (3–5, grounded in actual research)
```

**7e. Open question drafts** → `outputs/apply-prep/[slug]/open_questions.md`
5 questions, each 3–4 sentences, grounded in `application_narratives.md`, tailored to this role:
1. "Why are you interested in this role / company?"
2. "Tell me about yourself" (30-second version)
3. "What's your biggest strength relevant to this role?"
4. "Describe a time you led a cross-functional initiative"
5. "Where do you see yourself in 3 years?"

**7f. STAR stories** → `outputs/apply-prep/[slug]/star_stories.md`
From `application_narratives.md`, pick 2–3 stories most relevant to this role's key requirements.
For each: story title / why it fits / key metric to lead with / how to open.

### 8. Email alert (requires Gmail MCP)

If no 9+ jobs: skip.

Send one email to `config.json profile.email`:
- **Subject:** `jobAI Alert — X job(s) scored 9+ today (YYYY-MM-DD)`
- **Body:**
```
Hi,

The job hunter found X job(s) scoring 9 or 10 today.
Full application packages are ready — review before applying.

[For each 9+ job:]
──────────────────────────────
Title:   [title]
Company: [company]
Score:   [fit_score]/10
Source:  [source]
URL:     [url]
Salary:  [salary_range]

Package ready at:
  outputs/apply-prep/[slug]/cv_[slug].md
  outputs/apply-prep/[slug]/company_briefing.md
  outputs/apply-prep/[slug]/open_questions.md
  outputs/apply-prep/[slug]/star_stories.md

Next steps:
  1. Review the package (git pull to get latest)
  2. Run /apply-ats [url]  ← Playwright fills the ATS form
  3. Review in browser → click Submit yourself
  4. Run /log-application [url]  ← tracks it in KB + Notion
──────────────────────────────

Total jobs found today: X | Passed filter: X | Score 5–8: X
Full results: outputs/jobs_YYYY-MM-DD.json

jobAI · running automatically for you
```

If Gmail MCP is not connected: skip, note in summary.

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
Sources searched: 9 (A1–A5, B1–B3, C1–C4)
Searches run: 12 | Page fetches: X/4
Jobs found: X | Passed location filter: X | New to KB: X
Score 9+: X → packages: [list of slugs]
Score 5–8: X (in KB, in weekly report)
Email: sent / skipped
```

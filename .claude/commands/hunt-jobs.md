# /hunt-jobs

## Goal
Search for remote job listings across 17 sources using 3 parallel search agents. Score every result. For every job scoring 9 or 10, automatically generate a full application package. Send one email with everything ready.

## Architecture — multi-agent
Three search subagents run in parallel, each owning one block of sources. The orchestrator (this command) consolidates their results, scores, and handles all downstream steps.

```
hunt-jobs (orchestrator)
├── hunt-search-a  → Block A: 7 PM board searches    ┐
├── hunt-search-b  → Block B: 5 LATAM/discovery       ├── parallel
└── hunt-search-c  → Block C: 6 AI/automation         ┘
        ↓ merge + dedup + score
        ↓ apply-prep for 9+ (sequential, max 3)
        ↓ Notion Top Jobs + email + git
```

## Token budget

**Phase 1 — Search (per subagent, independent budgets):**
- Agent A: 7 searches, max 2 fetches
- Agent B: 5 searches, max 2 fetches
- Agent C: 6 searches, max 1 fetch

**Phase 2 — Apply-prep (orchestrator, per 9+ job):**
- 1 fetch for job URL
- Max 2 web searches for company research
- Max 1 optional fetch for company site
- Max 3 jobs processed per run

---

## Steps

### 1. Load context
- Read `config.json` — profile, target roles, scoring rules
- Read `data/cv.md` — source of truth for scoring and apply-prep
- Read `application_profile.json` — interests + apply-prep inputs
- Read `application_narratives.md` — needed for STAR stories in apply-prep

### 2. Spawn 3 search agents in parallel
Launch all 3 simultaneously using the Agent tool with `run_in_background: true`.

**Agent A prompt:**
> "Read `.claude/commands/hunt-search-a.md` for your full instructions. Execute every step exactly as written. Run all 7 searches. Return only a JSON array of candidates that passed the pre-filter — no other text."

**Agent B prompt:**
> "Read `.claude/commands/hunt-search-b.md` for your full instructions. Execute every step exactly as written. Run all 5 searches. Return only a JSON array of candidates that passed the pre-filter — no other text."

**Agent C prompt:**
> "Read `.claude/commands/hunt-search-c.md` for your full instructions. Execute every step exactly as written. Run all 6 searches. Return only a JSON array of candidates that passed the pre-filter — no other text."

Wait for all 3 agents to complete. Each returns a JSON array of raw candidates.

### 3. Merge and deduplicate
Concatenate the 3 arrays into one list.
Remove duplicates by URL — keep the first occurrence.
This is the raw candidate pool.

### 4. Apply location filter (hard rules — zero exceptions)
**Reject immediately if** snippet or page contains ANY of:
- "US only", "US citizens only", "United States only", "must be based in the US"
- "EU only", "Europe only", "must be in Europe"
- "requires relocation", "on-site", "hybrid", "in-office"
- "must be authorized to work in" (unless "worldwide" or "Brazil" is specified)

**Accept if:**
- "worldwide", "global", "anywhere", "LATAM", "Americas", "UTC-3"
- "remote" with no geographic restriction
- No location mentioned at all

### 5. Calculate fit score (0–10)
- Role match with `config.json target_roles`: +0–3
  - Primary roles (Product Manager, Senior PM, etc.): up to +3
  - Secondary roles (AI Builder, AI Engineer, Automation Specialist, Solutions Engineer): up to +2
  - Monitor-only roles (Head of Product, Product Lead): score but flag
- Salary in $70k–$100k range: +2 (unlisted = +1)
- Location confirmed worldwide/LATAM/global: +2
- Keyword overlap with `data/cv.md` skills: +0–2
- Seniority match: +0–1
- Interest alignment (`application_profile.json interests`):
  - Matches `want_to_do` (AI, automation, agentic, product strategy, greenfield): +1
  - Matches `want_to_avoid` (coordination only, compliance only, governance only): -1

**Thresholds:**
- < `search.min_fit_score` (default 5): discard
- 5–8: save to KB, in weekly report
- 9–10: save to KB + auto generate apply-prep package + email alert

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

**Main KB:** `data/knowledge_base.json`
- Apply TTL: remove entries older than 90 days
- Deduplicate by URL
- Append new jobs. Write back.

**Weekly slice:** `data/kb_YYYY-WNN.json`
- URL dedup only (no TTL). Append new jobs. Write back.

### 7. Generate apply-prep packages for 9+ scoring jobs

If no jobs scored 9+: skip to step 8.

**For each 9+ job (max 3 per run):**

Compute company slug: lowercase, hyphens, no special characters.

**7a. Fetch job description (1 fetch)**
Extract: full title, company, key requirements (hard + nice-to-have), team/product context.

**7b. Research company (max 2 searches + 1 optional fetch)**
```
Search: [Company] product what they do culture remote
Search: [Company] recent news funding 2026
```
Optional: 1 fetch of company website if critical facts are missing.

**7c. Tailored CV** → `outputs/apply-prep/[slug]/cv_[slug].md`
Classify each requirement: strong match / weak match / gap.
- Strong: verify it's visible in the CV
- Weak: rewrite bullet using the job's exact vocabulary — same experience, better alignment
- Gap: note it, never add it
Rewrite the Professional Summary. Preserve ALL metrics, dates, company names, titles.

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
5 questions grounded in `application_narratives.md`, tailored to this role (3–4 sentences each):
1. "Why are you interested in this role / company?"
2. "Tell me about yourself" (30-second version)
3. "What's your biggest strength relevant to this role?"
4. "Describe a time you led a cross-functional initiative"
5. "Where do you see yourself in 3 years?"

**7f. STAR stories** → `outputs/apply-prep/[slug]/star_stories.md`
From `application_narratives.md`, select 2–3 most relevant stories.
For each: story title / why it fits this role / key metric to lead with / how to open.

### 8. Email alert (requires Gmail MCP)

If no 9+ jobs: skip.

Send one email to `config.json profile.email`:
- **Subject:** `jobAI Alert — X job(s) scored 9+ today (YYYY-MM-DD)`
- **Body:**
```
Hi,

The job hunter searched 17 sources and found X job(s) scoring 9 or 10 today.
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
  1. git pull → review the package
  2. /apply-ats [url]  → Playwright fills the ATS form
  3. Review in browser → click Submit yourself
  4. /log-application [url]  → tracks it in KB + Notion
──────────────────────────────

Total found today: X | Passed filter: X | Score 5–8: X
Full results: outputs/jobs_YYYY-MM-DD.json

jobAI · 17 sources · running automatically
```

If Gmail MCP not connected: skip, note in summary.

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
Architecture: 3 parallel search agents (A: 7 · B: 5 · C: 6 searches)
Agent A results: X raw | Agent B results: X raw | Agent C results: X raw
After merge + dedup: X candidates | Passed location filter: X | New to KB: X
Score 9+: X → packages: [list of company slugs]
Score 5–8: X
Email: sent / skipped
```

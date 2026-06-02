# jobAI — System Architecture

## What it does

jobAI is an autonomous job search agent that runs 24/7 in Anthropic's cloud without requiring the user's computer to be on. Three agents work on a fixed schedule:

- **Daily Job Hunter** — finds remote jobs every morning and alerts on top matches
- **Weekly CV Optimizer** — analyzes skill gaps and rewrites the CV to match the market
- **Weekly Report** — delivers a Monday briefing with stats, trends, and a study list

**Core principle:** The agent surfaces and reframes what already exists in the CV. It never invents experience, skills, or metrics.

---

## System diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Anthropic Cloud                           │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────┐  │
│  │ Daily Job Hunter│  │ Weekly CV       │  │ Weekly     │  │
│  │ Every day 7AM   │  │ Optimizer       │  │ Report     │  │
│  │ BRT             │  │ Sunday 8PM BRT  │  │ Mon 6AM    │  │
│  └────────┬────────┘  └────────┬────────┘  └─────┬──────┘  │
│           │                    │                  │          │
└───────────┼────────────────────┼──────────────────┼──────────┘
            │   clone → work → commit → push        │
            ▼                                       ▼
┌───────────────────────────────────────────────────────────────┐
│                    GitHub (Private Repo)                       │
│                                                               │
│  config.json               ← search config & profile rules   │
│  CLAUDE.md                 ← agent memory & behavioral rules  │
│  data/cv.md                ← source-of-truth CV               │
│  data/knowledge_base.json  ← all jobs found (90-day TTL)     │
│  data/kb_YYYY-WNN.json     ← weekly slices for trend calcs   │
│  outputs/gap_report.md     ← latest skill gap analysis       │
│  outputs/cv_v[N].md        ← versioned CV outputs            │
│  outputs/weekly_report.md  ← Monday morning briefing         │
│  .claude/commands/         ← agent prompt files (the logic)  │
└───────────────────────────┬───────────────────────────────────┘
                            │
                            │  auto-sync (architecture only)
                            ▼
┌───────────────────────────────────────────────────────────────┐
│                    GitHub (Public Repo)                        │
│           Portfolio showcase — no personal data               │
└───────────────────────────────────────────────────────────────┘

External services (MCP):
  Notion ← Kanban board for roadmap tracking (/updatekanban)
  Gmail  ← Email alerts for jobs scoring ≥ 9/10
```

---

## The three agents

### 1. Daily Job Hunter
**Schedule:** Every day at 7:00 AM BRT (10:00 UTC)

1. Reads `config.json` and `data/cv.md` to load the candidate profile
2. Searches 4 job boards: RemoteOK, Remotive, Himalayas, Wellfound
3. Applies hard location filters — rejects US-only, EU-only, on-site, hybrid
4. Scores each job 1–10 based on role match, skills, location, and salary
5. Saves results to `outputs/jobs_YYYY-MM-DD.json`
6. Appends new jobs to `data/knowledge_base.json` (URL-deduplicated)
7. Creates or updates `data/kb_YYYY-WNN.json` (weekly slice)
8. Sends a Gmail alert for any job with `fit_score >= 9`
9. Commits and pushes everything to the repo

**Token budget:** Max 4 web searches, max 2 page fetches. Data is extracted from search snippets to keep costs predictable.

---

### 2. Weekly CV Optimizer
**Schedule:** Every Sunday at 8:00 PM BRT (23:00 UTC) — runs as two sequential steps

**Step 1 — Analyze Gaps:**
1. Loads this week's and last week's KB slices for comparison
2. Identifies keywords from job descriptions and counts frequency
3. Classifies each keyword:
   - ⚠️ **Surface gap** — skill exists in the CV but is not visible enough (rewordable)
   - ❌ **Genuine gap** — skill is truly missing from the candidate's background (study list only, never added to CV)
4. Calculates real trend numbers: `↑ 4→7 jobs` instead of vague arrows
5. Saves `outputs/gap_report.md`

**Step 2 — Optimize CV:**
1. Reads `gap_report.md`
2. For each ⚠️ surface gap: rewrites the relevant CV bullet using market language
3. Hard constraints: never change metrics, dates, company names, titles; never address ❌ genuine gaps
4. Saves `outputs/cv_v[N].md` — version number increments on every run
5. Updates `outputs/cv_changelog.md` — what changed, why, which gap it addresses

---

### 3. Weekly Report
**Schedule:** Every Monday at 6:00 AM BRT (09:00 UTC)

1. Loads the two most recent weekly KB slices
2. Computes stats: jobs found, fit score distribution, top companies, top sources
3. Calculates keyword trends with actual numbers vs prior week
4. Builds a study list from genuine gaps only, ranked by frequency
5. Saves `outputs/weekly_report.md` and pushes to the repo

---

## Data flow across a full week

```
Mon–Sat   Job Hunter runs daily:
          outputs/jobs_YYYY-MM-DD.json  (new each day)
          data/knowledge_base.json      (accumulates, deduped)
          data/kb_2026-W23.json         (same week slice, grows daily)

Sunday    CV Optimizer runs:
          reads kb_2026-W23.json + kb_2026-W22.json (prior week)
          → outputs/gap_report.md
          → outputs/cv_v[N].md + cv_changelog.md

Monday    Weekly Report runs:
          reads kb_2026-W23.json + kb_2026-W22.json
          reads gap_report.md + cv_v[latest].md
          → outputs/weekly_report.md
```

---

## File structure

```
jobAI/
├── CLAUDE.md                        ← agent memory: rules, profile, schedule
├── config.json                      ← search parameters (source of truth)
├── application_profile.json         ← structured profile for ATS form-filling
├── application_narratives.md        ← STAR stories and interview prep
├── docs/                            ← learning documentation (this folder)
├── data/
│   ├── cv.md                        ← source-of-truth CV (read-only for agents)
│   ├── knowledge_base.json          ← accumulated jobs (90-day TTL, URL dedup)
│   └── kb_YYYY-WNN.json             ← weekly snapshots for trend analysis
└── outputs/
    ├── jobs_YYYY-MM-DD.json         ← daily job hunt results
    ├── gap_report.md                ← latest skill gap analysis
    ├── cv_v[N].md                   ← versioned CV outputs
    ├── cv_changelog.md              ← audit trail of every CV change and why
    └── weekly_report.md             ← Monday morning briefing
```

---

## Technology stack

| Component | Technology | Why |
|---|---|---|
| Agent runtime | Claude Sonnet 4.6 | Reads/writes files, runs tools, calls MCPs |
| Scheduling | Claude Routines (Anthropic cloud) | Runs 24/7 without local machine; uses Pro plan |
| Data storage | GitHub (private repo) | Agents clone each run; git history = full audit trail |
| Web search | WebSearch + WebFetch tools | Searches job boards and extracts structured data |
| Notifications | Gmail MCP | Immediate email for top-scoring jobs |
| Roadmap tracking | Notion MCP | Kanban board updated via `/updatekanban` command |
| Public showcase | GitHub (public repo) | Auto-synced from private repo on every push |

---

## Key design decisions

**Why CLAUDE.md as agent memory?**
Claude reads `CLAUDE.md` automatically at the start of every session as a system prompt. All three agents share the same behavioral rules without repeating them. Change the rules once in `CLAUDE.md` — all agents inherit the update on their next run.

**Why separate `config.json` from `CLAUDE.md`?**
`CLAUDE.md` holds behavioral rules (stable, rarely changed). `config.json` holds parameters (target roles, salary range, sources — frequently tuned). Separating them means you can expand the job search scope without touching the agent's behavioral constraints.

**Why the surface gap / genuine gap distinction?**
This is the core ethical constraint of the system. Adding skills to a CV that you don't actually have is misrepresentation. The agent can only reword bullets to make existing skills more visible. Genuine gaps go to a study list — the human decides whether to learn them.

**Why versioned CV outputs?**
`cv_v1.md`, `cv_v2.md`, etc. mean you can always see how the CV evolved over weeks and roll back to a previous version. The changelog explains every change, making the optimization process auditable and trustworthy.

**Why 90-day TTL on the knowledge base?**
Job market data older than 3 months is stale. Skill demand shifts. Pruning keeps the gap analysis reflecting the current market rather than mixing in outdated snapshots.

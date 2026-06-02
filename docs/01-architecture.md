# jobAI — System Architecture

## What it is

jobAI is an autonomous job search agent. It runs 24/7 in the cloud without requiring the user's computer to be on. Three agents — Job Hunter, CV Optimizer, Weekly Reporter — work together on a schedule to find the right jobs, evolve the CV to match the market, and deliver a Monday briefing with actionable priorities.

**Core principle:** The agent surfaces and reframes what already exists in the CV. It never invents experience, skills, or metrics.

---

## Components

```
┌─────────────────────────────────────────────────────────┐
│                  Anthropic Cloud                         │
│                                                          │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│   │ Daily Job    │  │ Weekly CV    │  │ Weekly       │ │
│   │ Hunter       │  │ Optimizer    │  │ Report       │ │
│   │ 7AM BRT      │  │ Sun 8PM BRT  │  │ Mon 6AM BRT  │ │
│   └──────┬───────┘  └──────┬───────┘  └──────┬───────┘ │
│          │                 │                  │          │
└──────────┼─────────────────┼──────────────────┼──────────┘
           │  clone + push   │                  │
           ▼                 ▼                  ▼
┌──────────────────────────────────────────────────────────┐
│              GitHub (Private Repo)                        │
│                                                           │
│  config.json          ← agent configuration               │
│  data/cv.md           ← source of truth CV                │
│  data/knowledge_base.json  ← accumulated jobs (90d TTL)  │
│  data/kb_YYYY-WNN.json     ← weekly slices for trends    │
│  outputs/gap_report.md     ← weekly skill gap analysis   │
│  outputs/cv_v[N].md        ← versioned CV outputs        │
│  outputs/weekly_report.md  ← Monday morning briefing     │
│  .claude/commands/         ← agent prompts (the logic)   │
└──────────────────────────────────────────────────────────┘
           │
           │  sync (architecture only, no personal data)
           ▼
┌──────────────────────────────────────────────────────────┐
│              GitHub (Public Repo)                         │
│  Portfolio showcase — code, architecture, examples        │
└──────────────────────────────────────────────────────────┘
```

**External services connected via MCP:**
- **Notion** — Kanban board for roadmap tracking, `/updatekanban` command
- **Gmail** — Email alerts when a job scores ≥ 9 (fit score)

---

## Three Agents

### 1. Daily Job Hunter
**When:** Every day at 7:00 AM BRT  
**What it does:**
1. Reads `config.json` and `data/cv.md` to understand the profile
2. Searches 4 job boards: RemoteOK, Remotive, Himalayas, Wellfound
3. Applies hard location filters (rejects US-only, EU-only, on-site, hybrid)
4. Scores each job 1–10 based on role match, skills, location, salary
5. Saves results to `outputs/jobs_YYYY-MM-DD.json`
6. Appends new jobs (deduplicated by URL) to `data/knowledge_base.json`
7. Creates/updates `data/kb_YYYY-WNN.json` (weekly slice)
8. Sends email via Gmail MCP for any job with fit_score ≥ 9
9. Commits and pushes everything to the repo

**Token budget:** Max 4 searches, max 2 page fetches. Extracts data from snippets to keep costs low.

### 2. Weekly CV Optimizer
**When:** Every Sunday at 8:00 PM BRT (two jobs run in sequence)

**Job 1 — Analyze Gaps:**
1. Loads current week's jobs and prior week's jobs from KB slices
2. Identifies skill keywords appearing in job descriptions
3. Classifies each gap:
   - ⚠️ **Surface gap** — skill exists in the CV but isn't visible enough (fixable with rewording)
   - ❌ **Genuine gap** — skill truly missing from the candidate's background (study list only, never added to CV)
4. Calculates real trend numbers: "↑ 4→7 jobs" not just "↑↑↑"
5. Saves `outputs/gap_report.md`

**Job 2 — Optimize CV:**
1. Reads the gap report
2. For each ⚠️ surface gap: rewrites the relevant CV bullet to surface the skill using market language
3. Hard rules: never change metrics, dates, company names, job titles; never address ❌ genuine gaps
4. Saves `outputs/cv_v[N].md` (version number increments each run)
5. Updates `outputs/cv_changelog.md` (what changed, why, which gap it addresses)

### 3. Weekly Report
**When:** Every Monday at 6:00 AM BRT  
**What it does:**
1. Reads the two most recent weekly KB slices
2. Computes stats: jobs found, fit score distribution, top companies, top sources
3. Extracts keyword trends with actual numbers
4. Builds study list from genuine gaps, ranked by frequency
5. Saves `outputs/weekly_report.md`
6. Commits and pushes

---

## Data Flow

```
Day 1   Job Hunter → jobs_2026-06-02.json
                   → knowledge_base.json (append, dedup)
                   → kb_2026-W23.json (create/update)

Day 2-6  Same as Day 1, accumulating in the same week's KB slice

Sunday  CV Optimizer → reads kb_2026-W23.json + kb_2026-W22.json
                     → gap_report.md
                     → cv_v[N].md + cv_changelog.md

Monday  Weekly Report → reads kb_2026-W23.json + kb_2026-W22.json
                      → reads gap_report.md + cv_v[latest].md
                      → weekly_report.md
```

---

## File Structure

```
jobAI/
├── CLAUDE.md                   ← agent memory (behavioral rules, profile, schedule)
├── config.json                 ← user parameters (source of truth for search config)
├── application_profile.json    ← structured profile for ATS form-filling (future)
├── application_narratives.md   ← STAR stories and interview prep (future)
├── docs/                       ← learning documentation (this folder)
├── data/
│   ├── cv.md                   ← source of truth CV (read-only for agents)
│   ├── knowledge_base.json     ← all jobs ever found (90-day TTL, URL dedup)
│   └── kb_YYYY-WNN.json        ← weekly snapshots for trend analysis
├── outputs/
│   ├── jobs_YYYY-MM-DD.json    ← daily job hunt results
│   ├── gap_report.md           ← latest skill gap analysis
│   ├── cv_v[N].md              ← versioned CV outputs (current: latest N)
│   ├── cv_changelog.md         ← audit trail of every CV change and why
│   └── weekly_report.md        ← Monday morning briefing
└── .claude/
    ├── commands/
    │   ├── hunt-jobs.md        ← job hunter agent prompt
    │   ├── analyze-gaps.md     ← gap analysis agent prompt
    │   ├── optimize-cv.md      ← CV optimizer agent prompt
    │   ├── weekly-report.md    ← weekly report agent prompt
    │   └── updatekanban.md     ← Notion kanban update command
    └── settings.local.json     ← local tool permission allowlist
```

---

## Technology Stack

| Component | Technology | Why |
|---|---|---|
| Agent runtime | Claude Code (claude-sonnet-4-6) | Reads/writes files, runs commands, calls tools |
| Scheduling | Claude Routines (Anthropic cloud) | Runs 24/7 without local machine; uses Pro plan, no API cost |
| Code + data storage | GitHub (private repo) | Agents clone on each run; git history = audit trail |
| Job search | WebSearch + WebFetch tools | Searches job boards and extracts structured data |
| Notifications | Gmail MCP | Email alerts for top-scoring jobs |
| Roadmap tracking | Notion MCP | Kanban board synced from code via /updatekanban |
| Public showcase | GitHub (public repo) | Portfolio — auto-synced from private repo |

---

## Key Design Decisions

**Why CLAUDE.md as agent memory?**  
Claude reads `CLAUDE.md` at the start of every session as a system prompt. It defines who the agent is, what rules it must follow, and what files to use. Centralizing this means all three agents share the same behavioral constraints without repeating them in every command file.

**Why separate config.json from CLAUDE.md?**  
`CLAUDE.md` is behavioral (rules, constraints, process). `config.json` is parametric (which roles, which salary range, which sources). Separating them means you can change target roles or salary range without touching the behavioral rules — and vice versa.

**Why the surface gap / genuine gap distinction?**  
This is the core ethical constraint of the system. Adding skills to a CV that you don't have is fraud. The agent can only reword bullets to make existing skills more visible — never add skills that aren't there. Genuine gaps go to a study list so the human can decide whether to learn them.

**Why versioned CV outputs?**  
`cv_v1.md`, `cv_v2.md`, etc. mean you can always see how the CV evolved and roll back to a previous version. The changelog explains every change, so you always know why something was rewritten.

**Why 90-day TTL on the knowledge base?**  
Job market data older than 3 months is stale — companies change, roles change, keywords evolve. Pruning keeps the gap analysis reflecting the current market, not a snapshot from months ago.

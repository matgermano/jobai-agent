# jobAI

> Autonomous job search agent built with Claude Code & Claude Routines. Configure once, runs forever — no server, no machine required.

**The problem:** Job hunting is a part-time job — 3 to 5 hours a week of manual searching, copy-pasting, and reactive CV updates with no real market data behind the decisions.

**The solution:** A multi-agent system that hunts jobs daily, analyzes market gaps weekly, keeps your CV always aligned with what the market is actually asking for, and delivers a Monday morning briefing with exactly what to study next.

> 📐 **Full process map (FigJam):** [AS IS & TO BE — Job Hunter](https://www.figma.com/board/UEWrfBJHElNMT4MdvWgo1S/AsIS-e-ToBe---JobHunter?node-id=0-1&t=OthMs6R4TE8Unrkw-1)

---

## Before vs After

| | Before (manual) | After (jobAI) |
|---|---|---|
| Job search | 2–3h/week, 2–3 sites | Automated daily, 4 sources |
| CV updates | Occasional, reactive | Every Sunday, data-driven |
| Market awareness | Gut feeling | Top keywords ranked by real frequency |
| Study direction | Generic advice | Specific gaps from real listings |
| Top job alerts | Check manually | Instant email when score ≥ 9 |
| Weekly effort | 3–5 hours | **15 minutes of focused review** |

**~92% reduction in manual job search effort.**

---

## What's live today

| Feature | Status |
|---|---|
| Daily job search across 4 remote boards | ✅ Live |
| Hard location filter — rejects US/EU-only automatically | ✅ Live |
| Fit scoring per listing (0–10) | ✅ Live |
| Instant email alert for score 9–10 jobs (Gmail MCP) | ✅ Live |
| Weekly CV gap analysis vs real market keywords | ✅ Live |
| Surface gap vs genuine gap distinction (never fabricates) | ✅ Live |
| Weekly CV rewrite guided by recurring gaps | ✅ Live |
| Versioned CV outputs with full changelog | ✅ Live |
| Data-driven trend tracking (↑ 4→8 jobs, not just arrows) | ✅ Live |
| Monday morning digest report | ✅ Live |
| Notion Kanban — roadmap tracked via MCP | ✅ Live |
| Full git history of every run as audit trail | ✅ Live |

---

## How it works

Three agents run on a fixed schedule in Anthropic's cloud — no machine needs to be on:

```
Every day    07:00 BRT  →  Daily Job Hunter
Every Sunday 20:00 BRT  →  CV Gap Analyzer + CV Optimizer
Every Monday 06:00 BRT  →  Weekly Report
```

### Agent 1 — Daily Job Hunter
Searches 4 remote job boards, applies hard location filters (rejects US/EU-only, on-site, hybrid), scores each listing 0–10 for fit, deduplicates by URL, saves structured JSON. Snippet-only extraction — max 4 searches, max 2 page fetches per run for token efficiency. Sends Gmail alert immediately for any score ≥ 9.

**Active sources:** Remote OK · Remotive · Himalayas · Wellfound  
**Roadmap sources:** We Work Remotely · Remote Rocketship · Job na Gringa · Turing · Contra · LinkedIn (Chrome MCP)

### Agent 2 — CV Gap Analyzer + CV Optimizer
Runs as two sequential steps every Sunday. The analyzer compares this week's jobs vs last week with real numbers (not arrows alone) and classifies each skill gap:
- ⚠️ **Surface gap** — skill exists in the CV but isn't visible enough → optimizer rewrites the bullet
- ❌ **Genuine gap** — skill doesn't exist in experience → goes to study list, never added to CV

The optimizer only addresses surface gaps. It never invents experience, never changes metrics, dates, company names, or job titles. Every change is logged in `cv_changelog.md`.

### Agent 3 — Weekly Report
Delivered Monday at 6AM: jobs found, fit score distribution, top companies, keyword trends with real numbers, and a prioritized study list built from genuine gaps — not generic advice.

---

## Architecture

```
Anthropic Cloud (Claude Routines)
├── Daily Job Hunter     07:00 BRT
├── Weekly CV Optimizer  Sun 20:00 BRT
└── Weekly Report        Mon 06:00 BRT
         ↓  clone → work → commit → push
GitHub (Private repo)
├── config.json + CLAUDE.md + data/cv.md    ← agent config & CV
├── data/knowledge_base.json                ← 90-day TTL, URL dedup
├── data/kb_YYYY-WNN.json                   ← weekly slices for trends
├── outputs/gap_report.md                   ← weekly skill gap analysis
├── outputs/cv_v[N].md + cv_changelog.md    ← versioned CV outputs
└── outputs/weekly_report.md                ← Monday briefing

External services (MCP)
├── Notion  ← Kanban board for roadmap tracking
└── Gmail   ← Instant alerts for score 9+ jobs
```

All outputs are plain markdown and JSON — no database, no server, no infrastructure.

---

## Tech stack

| Layer | Tool |
|---|---|
| Agent runtime | Claude Sonnet 4.6 |
| Scheduling | Claude Routines (Anthropic cloud) |
| Storage | Git repo — plain JSON + Markdown |
| Job search | WebSearch + WebFetch tools |
| Notifications | Gmail MCP |
| Roadmap tracking | Notion MCP |
| Auth | Claude Code OAuth (Pro plan) |

---

## Roadmap

| Phase | Focus | Status |
|---|---|---|
| Phase 0 | Foundation — Routines, profile, Obsidian, two-repo strategy, security | ✅ Done |
| Phase 1 | Smart discovery — role expansion, dedup+TTL, trend tracking, email alerts | ✅ Done |
| Phase 2 | Positioning — CV per job, LinkedIn analysis, interest alignment | ⬜ Next |
| Phase 3 | Assisted application — Chrome MCP, ATS form-filling, application tracker | ⬜ Backlog |
| Phase 4 | Specialization — custom Skills, Cowork, interview prep | ⬜ Backlog |
| Phase 5 | Mastery — multi-agent orchestration, publishable case study | ⬜ Backlog |

---

## Quick start

### Prerequisites
- Claude Pro or Max plan
- GitHub account

### Setup (4 steps)

**1. Use this repo as a template**

Copy the structure: `config.example.json` → `config.json`, `data/cv.example.md` → `data/cv.md`, `application_profile.example.json` → `application_profile.json`. Fill in your real data.

**2. Create a private GitHub repo**

Push everything to a **private** repo. Your CV, salary target, and contact info must never be in a public repo. See `docs/06-two-repo-strategy.md` for the full pattern.

**3. Connect MCP services**

At `claude.ai/customize/connectors`:
- Connect **Notion** (for roadmap tracking via `/updatekanban`)
- Connect **Gmail** (for instant alerts on score 9+ jobs)

**4. Create the three Routines**

In Claude Code, run `/schedule` and create:
- Daily Job Hunter: `0 10 * * *` UTC (7AM BRT), reads `hunt-jobs.md`
- Weekly CV Optimizer: `0 23 * * 0` UTC (8PM BRT Sunday), reads `analyze-gaps.md` + `optimize-cv.md`
- Weekly Report: `0 9 * * 1` UTC (6AM BRT Monday), reads `weekly-report.md`

See `docs/02-claude-routines.md` for full configuration details including exact prompts.

---

## Output files

| File | Updated | Description |
|---|---|---|
| `outputs/jobs_YYYY-MM-DD.json` | Daily | Scored job listings |
| `data/knowledge_base.json` | Daily | Accumulated jobs (URL dedup, 90-day TTL) |
| `data/kb_YYYY-WNN.json` | Daily | Weekly slices for trend calculation |
| `outputs/gap_report.md` | Weekly | Market gap analysis with real trend numbers |
| `outputs/cv_v[N].md` | Weekly | Optimized CV version |
| `outputs/cv_changelog.md` | Weekly | What changed, why, which gap it addressed |
| `outputs/weekly_report.md` | Weekly | Monday briefing + prioritized study list |

---

## CV optimization rules

The optimizer follows strict rules to ensure integrity:

- ✅ Rewrites existing bullets to surface skills already in the CV
- ✅ Injects market keywords into real experience descriptions
- ✅ Preserves all metrics exactly as written
- ❌ Never invents experience, projects, skills, or certifications
- ❌ Never changes company names, job titles, or employment dates
- ❌ Never addresses genuine gaps — those go to the study list only

---

## Why this exists

Built as a practical demonstration of agentic AI applied to a real personal problem. It maps directly to what AI transformation and product roles require: process discovery (AS IS/TO BE), automation orchestration, responsible AI design (the optimizer never fabricates), and shipping something that actually runs in production.

The `docs/` folder contains detailed notes on every architectural decision — Claude Routines vs GitHub Actions, MCP integrations, data hygiene, prompt engineering patterns, and the two-repo strategy for public portfolios with private data.

---

## License

MIT — clone it, adapt it, make it yours.

---

*Built by [Alex Rivera](https://linkedin.com/in/alexrivera-pm) · Powered by Claude Code & Claude Routines*

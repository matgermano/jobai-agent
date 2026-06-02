# jobAI 🤖

> Open-source job search automation built with Claude Code & GitHub Actions. Configure once, run forever.

**The problem:** Job hunting is a part-time job in itself — 3 to 5 hours a week of manual searching, copy-pasting, and reactive CV updates with no real data behind the decisions.

**The solution:** A multi-agent system that hunts jobs daily, analyzes market gaps weekly, keeps your CV always aligned, and delivers a Monday morning report telling you exactly what to study next.

---

## AS IS vs TO BE

> 📐 **Full process map (FigJam):** [AS IS & TO BE — Job Hunter](https://www.figma.com/board/UEWrfBJHElNMT4MdvWgo1S/AsIS-e-ToBe---JobHunter?node-id=0-1&t=OthMs6R4TE8Unrkw-1)

| | Before (Manual) | After (jobAI) |
|---|---|---|
| Job search | 2–3h/week, 2–3 sites | Automated daily, 4 sources (10 on roadmap) |
| CV updates | Occasional, reactive | Every Sunday, data-driven |
| Market awareness | Gut feeling | Top keywords ranked by frequency |
| Study direction | Generic advice | Specific gaps from real listings |
| Weekly effort | 3–5 hours | **15 minutes of focused review** |

**Time saved: ~92% reduction in manual effort.**

---

## What it does today

| Feature | Status |
|---|---|
| Daily job search across 4 remote boards (6 more on roadmap) | ✅ Live |
| Hard location filter (rejects US/EU-only) | ✅ Live |
| Fit scoring per listing (0–10) | ✅ Live |
| Weekly CV gap analysis vs market | ✅ Live |
| CV rewrite guided by recurring gaps | ✅ Live |
| Monday morning digest report | ✅ Live |
| Full git history of every run | ✅ Live |

---

## Backlog

| # | Feature | What it does |
|---|---------|--------------|
| 1 | **Email alert on high-fit jobs** | Sends an instant notification when a score 9–10 job is found — no waiting until Monday |
| 2 | **Apply prep command** | Given a job URL, generates a tailored briefing: which CV bullets to lead with, company context, and likely interview questions |
| 3 | **Learning recommendations** | Maps each genuine CV gap to a specific YouTube video, course, or article — concrete next step, not generic advice |
| 4 | **Notion integration** | Pushes weekly reports and job listings directly into a Notion workspace for easier reading and tracking |
| 5 | **Multi-source expansion** | Adds Remote Rocketship, Job na Gringa, Turing, Contra, and We Work Remotely to the daily search |
| 6 | **LinkedIn job search** | Navigates LinkedIn Jobs via Chrome MCP browser automation — the largest source of PM listings |
| 7 | **LinkedIn profile sync** | Reads the LinkedIn profile, compares it to market keywords, and suggests copy updates |
| 8 | **Auto-apply to jobs** | Submits applications automatically on supported job boards based on fit score threshold |
| 9 | **Outreach automation** | Drafts and sends personalized connection messages to people at target companies |

---

## Known limitations

| Limitation | Status | Notes |
|---|---|---|
| LinkedIn search | Not implemented | Requires Chrome MCP browser automation — in backlog |
| Active sources | 4 of 10 planned | Remaining 6 require Chrome MCP or token budget validation |
| Auto-apply | Not implemented | Each job board has a different application flow — high complexity |
| Trend data | Baseline only | Requires 2+ weeks of data for real comparisons |

---

## How it works

jobAI runs autonomous agents on a fixed schedule using Claude Code and GitHub Actions:

```
Every day 7AM BRT      → Job Hunter Agent
Every Sunday 8PM BRT   → CV Gap Analyzer + CV Optimizer
Every Monday 6AM BRT   → Weekly Report
```

### Module 1 — Job Hunter
Searches 4 remote job boards daily, applies hard location filters (rejects US/EU-only remote), scores each listing 0–10 for fit, and saves structured JSON output. No individual page fetches — snippet-only extraction for token efficiency.

**Active sources:** Remote OK · Remotive · Himalayas · Wellfound

**Roadmap sources:** We Work Remotely · Remote Rocketship · Job na Gringa · Turing · Contra · LinkedIn (Chrome MCP)

### Module 2 — CV Gap Analyzer
Reads the full week of accumulated job data, groups requirements by frequency, and identifies what the market is asking for that your CV isn't surfacing clearly.

### Module 3 — CV Optimizer
Rewrites your CV bullets to surface skills you already have — guided by real market gaps. **Never invents experience. Never removes real metrics.** Saves versioned output with a full changelog.

### Module 4 — Weekly Report
Delivered Monday at 6AM: jobs found, top keywords, fit score trends, and a concrete study list based on genuine gaps — not generic advice.

---

## Tech stack

| Layer | Tool |
|---|---|
| Agent runtime | Claude Code (Anthropic) |
| Scheduling | GitHub Actions (cron) |
| Storage | Local files + Git (no database) |
| Version control | GitHub |
| Auth | Claude Code OAuth token |

---

## Architecture

```
GitHub Actions (cron trigger)
        ↓
Claude Code agent (headless mode, Haiku model)
        ↓
Anthropic API → Web Search (4 sources) → Filter → Score → jobs_YYYY-MM-DD.json
                                                              ↓
                                                    knowledge_base.json (deduped, TTL 90d)
                                                    kb_YYYY-WNN.json (weekly slices)
                                                              ↓
                                          Gap Analysis → gap_report.md (with trend data)
                                                              ↓
                                          CV Optimizer → cv_v[N].md + changelog
                                                              ↓
                                          Weekly Report → weekly_report.md
                                                              ↓
                                          Auto git commit → pushed to main
```

All outputs are plain markdown and JSON files — no database, no server, no infrastructure to maintain.

---

## Quick start

### Prerequisites
- Claude Pro or Max plan
- GitHub account
- Node.js 18+

### Setup (5 steps)

**1. Clone the repo**
```bash
git clone https://github.com/yourusername/jobAI.git
cd jobAI
```

**2. Edit `config.json` with your profile**
```json
{
  "profile": {
    "name": "Your Name",
    "location": "Your City, Country"
  },
  "search": {
    "target_roles": ["Product Manager", "Senior PM"],
    "salary_min_usd": 80000,
    "salary_max_usd": 110000
  }
}
```

**3. Paste your CV into `data/cv.md`**

Plain text or markdown — the agent reads this as the source of truth for all optimization.

**4. Add your OAuth token to GitHub Secrets**

`Settings → Secrets → Actions → New secret`
- Name: `CLAUDE_CODE_OAUTH_TOKEN`
- Value: your Claude Code OAuth token (run `claude auth status` in your terminal to retrieve it)

**5. Push and let it run**

The workflows trigger automatically on schedule. To test manually: `Actions → Daily Job Hunter → Run workflow`.

---

## Output files

| File | Updated | Description |
|---|---|---|
| `outputs/jobs_YYYY-MM-DD.json` | Daily | Scored job listings |
| `data/knowledge_base.json` | Daily | Accumulated jobs (deduped, 90-day TTL) |
| `data/kb_YYYY-WNN.json` | Daily | Weekly knowledge base slices |
| `outputs/gap_report.md` | Weekly | Market gap analysis with trend data |
| `outputs/cv_v[N].md` | Weekly | Optimized CV version |
| `outputs/cv_changelog.md` | Weekly | What changed and why |
| `outputs/weekly_report.md` | Weekly | Summary + study list |

---

## CV optimization rules

The optimizer follows strict rules to ensure integrity:

- ✅ Rewrites existing bullets to surface relevant skills
- ✅ Injects recurring market keywords into real experience descriptions
- ✅ Preserves all metrics exactly as written
- ❌ Never invents experience, projects, or skills
- ❌ Never changes company names, titles, or dates

---

## Why this exists

This project was built as a practical demonstration of agentic AI applied to a real personal problem: the inefficiency of manual job searching.

It maps directly to what modern AI transformation roles require — process discovery (AS IS), future-state design (TO BE), automation orchestration, metrics tracking, and responsible AI principles (the optimizer never fabricates data).

Built in a weekend. Runs forever.

---

## License

MIT — clone it, adapt it, make it yours.

---

*Built by [Alex Rivera](https://linkedin.com/in/alexrivera-pm) · Powered by Claude Code*

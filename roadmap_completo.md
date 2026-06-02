# jobAI — Complete Roadmap
# Product + Learning + Documentation
# Pace: 1h/day + 2h weekend = ~9h/week

---

## How to use this roadmap

Each card has:
- What to do (concrete action)
- What to learn (the concept)
- What to document in Obsidian (note to create)
- Estimated time
- Done criteria (how to know you're finished)

Golden rule: no card is Done without all three — built, working, and documented.

Import into Notion as a database with these columns:
Status | Phase | Type | Time | Week

---

## PHASE 0 — Organize the house
### Goal: stabilize what exists before building more

---

### Card 0.1 — Configure Obsidian
- **Type:** Setup
- **Time:** 1h
- **Week:** 1

**What to do:**
Create the "Second Brain" vault with this folder structure:
```
00_Inbox        ← quick capture
01_Projects     ← jobAI and future projects
02_Knowledge    ← Claude Code, MCP, Routines, etc.
03_Career       ← jobs, CV, networking
04_Daily        ← daily notes
```

**What to learn:**
Why a single vault (not one per project). The value comes from connections between things — when you link what you learned about MCP to where you applied it in jobAI, it becomes navigable knowledge.

**Document in Obsidian:**
Create note `02_Knowledge/Obsidian — How to use this vault.md` explaining the structure and the linking rule.

**Done criteria:** Vault created, 5 folders, first note in each.

---

### Card 0.2 — Add profile files to the project
- **Type:** Feature
- **Time:** 30min
- **Week:** 1

**What to do:**
Copy both files to the jobAI project root:
- `application_profile.json`
- `application_narratives.md`

Commit: `chore(profile): add structured application profile and narratives`

**What to learn:**
Why to separate structured data (JSON) from narratives (markdown). The agent uses JSON to fill in fields and markdown to answer open-ended questions.

**Document in Obsidian:**
`01_Projects/jobAI/Application profile.md` — what each file is and how the agent uses it.

**Done criteria:** Both files in the repo, committed, CV dates verified.

---

### Card 0.3 — Migrate from GitHub Actions to Routines
- **Type:** Infra
- **Time:** 2h
- **Week:** 1 (weekend)

**What to do:**
1. Open Claude Code in the project folder
2. Create the 3 Routines with `/schedule`:
   - Daily Job Hunter: every day 7AM BRT
   - Weekly CV Optimizer: Sunday 8PM BRT
   - Weekly Report: Monday 6AM BRT
3. Disable the GitHub Actions workflows
4. Test by running one Routine manually

**What to learn:**
Routines run on Anthropic's cloud using your Pro plan — no API cost, no need for your machine to be on. GitHub Actions requires an API key and charges per token.

**Document in Obsidian:**
`02_Knowledge/Claude Routines.md` — what they are, how to create, difference vs GitHub Actions, Pro plan run limits.

**Done criteria:** 3 Routines active, GitHub Actions disabled, one successful manual run.

---

### Card 0.4 — Build the Kanban in Notion
- **Type:** Setup
- **Time:** 1h
- **Week:** 2

**What to do:**
1. Go to `claude.ai/customize/connectors` and add Notion
2. Create a "jobAI" page in Notion
3. Add Claude connection to that page (menu "..." → Add connections)
4. Open Claude Code locally and run:

```
Read jobAI_roadmap_notion.md and create a Notion database
inside the page called "jobAI" with these properties:
- Card (title)
- Phase (select): Phase 0, Phase 1, Phase 2, Phase 3, Phase 4, Phase 5
- Type (select): Feature, Learning, Docs, Infra, Setup
- Time (text)
- Status (select): Backlog, To Do, Doing, Done

Create one entry per card in the file. Set Phase 0 cards to
"To Do" and everything else to "Backlog". Do not invent cards.
```

5. In Notion, switch the view to Board grouped by Status

**What to learn:**
MCP (Model Context Protocol) — the protocol that lets Claude operate external systems like Notion. When you connect via claude.ai, the connectors become available in Claude Code locally automatically (only works with OAuth Pro, not with API key).

**Document in Obsidian:**
`02_Knowledge/MCP — What it is and how it works.md` — definition, difference from traditional API, how to connect, permission model.

**Done criteria:** Kanban in Notion with all cards, Board view working.

---

## PHASE 1 — Smarter discovery
### Goal: the right jobs arriving without effort

---

### Card 1.1 — Expand role scope
- **Type:** Feature
- **Time:** 30min
- **Week:** 3

**What to do:**
Open `config.json` and add to target_roles:
- AI Builder
- AI Engineer
- Automation Specialist
- Process Transformation Specialist
- Solutions Engineer

**What to learn:**
How `config.json` shapes the behavior of all agents. Changing one central configuration file is more powerful than changing each agent individually.

**Document in Obsidian:**
`01_Projects/jobAI/How config shapes the agents.md`

**Done criteria:** Next job hunter run brings jobs from the new categories.

---

### Card 1.2 — Deduplication and TTL
- **Type:** Feature
- **Time:** 1h
- **Week:** 3

**What to do:**
Open Claude Code and ask:
```
Read .claude/commands/hunt-jobs.md and data/knowledge_base.json.
Add URL-based deduplication before appending new jobs.
Add TTL: remove entries older than 90 days on each weekly run.
Commit the changes.
```

**What to learn:**
Data hygiene — why dirty data distorts analysis. A job saved 5 times looks 5x more in demand than it actually is.

**Document in Obsidian:**
`02_Knowledge/Data hygiene in agents.md`

**Done criteria:** knowledge_base does not duplicate, does not grow unbounded.

---

### Card 1.3 — Real trend tracking
- **Type:** Feature
- **Time:** 1h
- **Week:** 4

**What to do:**
Ask Claude Code to update `analyze-gaps.md` to compare keywords from this week vs last week with real numbers — not estimated arrows.

**What to learn:**
Temporal data analysis. Real trends vs isolated snapshots.

**Document in Obsidian:**
`01_Projects/jobAI/How trend tracking works.md`

**Done criteria:** Weekly report shows "↑ 4→7 jobs" instead of "↑↑↑".

---

### Card 1.4 — Email alert for top jobs
- **Type:** Feature
- **Time:** 1h 30min
- **Week:** 4

**What to do:**
1. Connect Gmail at `claude.ai/customize/connectors`
2. Ask Claude Code to update hunt-jobs to send an email when score >= 9

**What to learn:**
Conditional trigger — the agent acts differently depending on the result. Second MCP connected (Gmail in addition to Notion).

**Document in Obsidian:**
`02_Knowledge/MCP — Gmail.md` — how to connect, use cases.

**Done criteria:** Receives email when a 9–10 job appears, without waiting until Monday.

---

## PHASE 2 — Positioning
### Goal: CV and LinkedIn always aligned with the market

---

### Card 2.1 — CV customized per job
- **Type:** Feature
- **Time:** 2h
- **Week:** 5 (weekend)

**What to do:**
Create new command `.claude/commands/cv-by-job.md`:
Given a job URL, the agent reads the job, compares it with the profile, and generates a `cv_[company].md` customized for that specific opportunity.

**What to learn:**
Context-conditional generation. Why a generic CV converts less than an aligned CV.

**Document in Obsidian:**
`01_Projects/jobAI/CV per job — how it works.md`

**Done criteria:** Generates cv_[company].md that differs from the generic CV, with realigned bullets.

---

### Card 2.2 — LinkedIn analysis
- **Type:** Feature
- **Time:** 2h
- **Week:** 6

**What to do:**
Create a command that compares your LinkedIn headline, about, and experience sections with the most in-demand keywords (from gap_report), and suggests copy improvements.

**What to learn:**
LinkedIn as a product. Headline is SEO, about is pitch, experience is proof.

**Document in Obsidian:**
`03_Career/LinkedIn — positioning strategy.md`

**Done criteria:** Report with specific improvement suggestions for the profile.

---

### Card 2.3 — Interest alignment
- **Type:** Feature
- **Time:** 1h
- **Week:** 6

**What to do:**
Add an `interests` section to `application_profile.json` with what you want to do, enjoy doing, and want to avoid. The agent will start weighing this in its recommendations.

**What to learn:**
Multi-objective optimization — balancing what the market demands with what you want.

**Document in Obsidian:**
`03_Career/What I want — direction clarity.md`

**Done criteria:** Agent recommendations reflect your interests, not just market demand.

---

## PHASE 3 — Assisted application
### Goal: eliminate the mechanical work of filling forms

---

### Card 3.1 — Learn Chrome MCP
- **Type:** Learning
- **Time:** 2h
- **Week:** 7 (weekend)

**What to do:**
1. Install Claude in Chrome (beta extension)
2. Test the agent filling a simple form (not a job application yet)
3. Understand the permission model and the risks

**What to learn:**
Browser automation — the agent operates the browser like a human. More powerful and more risky than API-based MCP.

**Document in Obsidian:**
`02_Knowledge/Chrome MCP — browser automation.md` — what it is, how to install, risks, when to use.

**Done criteria:** Can make the agent open a page and fill in a simple field.

---

### Card 3.2 — ATS form-filling engine
- **Type:** Feature
- **Time:** 3h
- **Week:** 8

**What to do:**
Create command `.claude/commands/apply-ats.md`:
Given a job URL on Greenhouse, Lever, or Ashby, the agent opens the form, reads `application_profile.json`, fills all fields, and stops before submitting for your review.

**What to learn:**
Field mapping — how the agent relates form fields to structured data. Assisted automation vs full automation.

**Document in Obsidian:**
`01_Projects/jobAI/Apply ATS — how it works.md`

**Done criteria:** Fills a real ATS application — you only review and click submit.

---

### Card 3.3 — Complete application package
- **Type:** Feature
- **Time:** 2h
- **Week:** 9

**What to do:**
Create command `/apply-prep`:
Given a job URL, the agent generates in a single package:
- CV customized for the job
- Drafted answers for open-ended questions
- Company briefing (context, product, culture)
- Your most relevant STAR stories for that specific job

**What to learn:**
RAG in practice — the agent combines external data (job, company) with your internal data (profile, narratives) to generate something personalized.

**Document in Obsidian:**
`01_Projects/jobAI/Apply prep — application package.md`

**Done criteria:** Complete package in under 5 minutes per job.

---

### Card 3.4 — Application tracker in Notion
- **Type:** Feature
- **Time:** 1h 30min
- **Week:** 9

**What to do:**
Add `applied` and `status` fields to knowledge_base. Create a second Notion database "Applications" with: company, job, date, status, next step.

**What to learn:**
Application pipeline management. Why tracking is as important as applying.

**Document in Obsidian:**
`03_Career/Application pipeline.md`

**Done criteria:** Notion dashboard showing all applications and their status.

---

## PHASE 4 — Specialization
### Goal: you as an AI professional, not just an AI user

---

### Card 4.1 — Build your own Skill
- **Type:** Learning
- **Time:** 2h
- **Week:** 10 (weekend)

**What to do:**
Package "CV optimization for ATS" as a reusable Claude Skill — one that anyone can use in their own project.

**What to learn:**
Skills — reusable knowledge packages. The difference between using Claude Code and extending Claude Code.

**Document in Obsidian:**
`02_Knowledge/Claude Skills — how to create.md`

**Done criteria:** Skill works and could be used in another project.

---

### Card 4.2 — Explore Cowork
- **Type:** Learning
- **Time:** 1h
- **Week:** 11

**What to do:**
Test Cowork for a non-technical knowledge task — for example, organizing your Obsidian notes automatically.

**What to learn:**
Difference between Claude Code (dev, code, files) and Cowork (knowledge work, non-technical). When to use each.

**Document in Obsidian:**
`02_Knowledge/Cowork vs Claude Code — when to use each.md`

**Done criteria:** Can explain the difference and cite a real use case for each.

---

### Card 4.3 — Interview preparation per job
- **Type:** Feature
- **Time:** 2h
- **Week:** 11

**What to do:**
Create command `/interview-prep`:
Given a company name, the agent researches the product, culture, recent news, and generates a personalized briefing with your most relevant stories for that interview.

**What to learn:**
Real-time contextual research + your internal data. How to prepare with AI without sounding artificial.

**Document in Obsidian:**
`03_Career/How to prepare for interviews with AI.md`

**Done criteria:** Briefing generated in under 3 minutes covering company, likely questions, and your stories.

---

### Card 4.4 — Publishable case study
- **Type:** Docs
- **Time:** 3h
- **Week:** 12 (weekend)

**What to do:**
Transform the entire jobAI journey into a documented case study:
- AS IS (before)
- Technical decisions and why
- What broke and how you debugged it
- Real results (jobs found, time saved)
- What you learned

**What to learn:**
Technical storytelling. How to transform a project into proof of competence.

**Document in Obsidian:**
`01_Projects/jobAI/Complete case study.md`

**Done criteria:** Publishable text you can show in interviews and on LinkedIn.

---

## Executive summary

| Phase | Focus | Weeks | Hours |
|---|---|---|---|
| Phase 0 | Organize the house | 1–2 | ~5h |
| Phase 1 | Smart discovery | 3–4 | ~6h |
| Phase 2 | Positioning | 5–6 | ~7h |
| Phase 3 | Assisted application | 7–9 | ~10h |
| Phase 4 | Specialization | 10–12 | ~10h |
| **Total** | | **12 weeks** | **~38h** |

---

## What you learn in each phase (knowledge track)

- **Phase 0:** Obsidian, Claude Routines, basic MCP (Notion)
- **Phase 1:** Agent configuration, data hygiene, temporal analysis, Gmail MCP
- **Phase 2:** Conditional generation, LinkedIn as a product, multi-objective optimization
- **Phase 3:** Chrome MCP, browser automation, RAG in practice, application pipeline
- **Phase 4:** Skills, Cowork, contextual research, technical storytelling

---

## Obsidian vault structure at the end of 12 weeks

```
00_Inbox/
01_Projects/
  └── jobAI/
        ├── Application profile
        ├── How config shapes the agents
        ├── CV per job
        ├── Apply ATS
        ├── Apply prep
        ├── Application pipeline
        └── Complete case study
02_Knowledge/
  ├── Obsidian — How to use this vault
  ├── Claude Routines
  ├── MCP — What it is and how it works
  ├── MCP — Gmail
  ├── Chrome MCP — browser automation
  ├── Data hygiene in agents
  ├── Claude Skills — how to create
  └── Cowork vs Claude Code
03_Career/
  ├── LinkedIn — positioning strategy
  ├── What I want — direction clarity
  ├── Application pipeline
  └── How to prepare for interviews with AI
04_Daily/
```

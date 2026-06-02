# Claude Routines — How They Work

## What are Routines

Claude Routines are remote agents that run on a schedule in Anthropic's cloud infrastructure. Each execution is a fully isolated session that:

1. Clones the configured GitHub repository
2. Receives the prompt defined at creation time
3. Executes with the permitted tools and MCP connections
4. Commits and pushes results back to the repo
5. Terminates — no state persists between runs

**Key insight:** The agent starts from scratch on every run. It reconstructs its context by reading `CLAUDE.md` and the command files from the cloned repo. This is why those files must be complete and self-contained — the agent has no memory of previous sessions.

---

## Routines vs GitHub Actions

jobAI originally ran on GitHub Actions and was migrated to Routines. Understanding the difference explains why the migration was worth it.

| Aspect | Claude Routines | GitHub Actions |
|---|---|---|
| **Where it runs** | Anthropic cloud | GitHub cloud |
| **Cost** | Included in Claude Pro plan | Charges per API token |
| **Agent model** | Configurable (Sonnet, Opus, Haiku) | Must call API — you pay per token |
| **Trigger** | Cron (1-hour minimum) | Cron, push, PR, webhook |
| **File access** | Clones repo via git | Clones repo via git |
| **MCP integrations** | Native OAuth support | Requires custom setup |
| **Monitoring** | claude.ai/code/routines | GitHub Actions logs |
| **Setup complexity** | Low — just a prompt | High — YAML workflow + secrets |

**Why the migration happened:** GitHub Actions charged per API token on every run. Routines use the Pro plan, which is already paid. For LLM-based automation, Routines are the right choice when you have Claude Pro.

**The old GitHub Actions workflows still exist in the repo** but are disabled (`disabled_manually`). They serve as documentation and a fallback option.

---

## How a Routine is structured

```json
{
  "name": "Daily Job Hunter",
  "cron_expression": "0 10 * * *",
  "enabled": true,
  "job_config": {
    "ccr": {
      "environment_id": "env_013rG4kcq6FpiQnqJ2Ypy78k",
      "session_context": {
        "model": "claude-sonnet-4-6",
        "sources": [
          {"git_repository": {"url": "https://github.com/matgermano/jobAI"}}
        ],
        "allowed_tools": [
          "Bash", "Read", "Write", "Edit", "Glob", "Grep",
          "WebSearch", "WebFetch"
        ]
      },
      "events": [
        {
          "data": {
            "type": "user",
            "message": {
              "role": "user",
              "content": "The agent prompt goes here — must be self-contained."
            }
          }
        }
      ]
    }
  },
  "mcp_connections": [
    {
      "connector_uuid": "cf2d86aa-dc6e-46ab-95cf-1de1ab3020c9",
      "name": "Notion",
      "url": "https://mcp.notion.com/mcp"
    }
  ]
}
```

### Field reference

**`cron_expression`** — 5-field cron in UTC. Minimum interval: 1 hour.
- `0 10 * * *` → every day at 10:00 UTC = 7:00 AM BRT
- `0 23 * * 0` → every Sunday at 23:00 UTC = 8:00 PM BRT
- `0 9 * * 1` → every Monday at 09:00 UTC = 6:00 AM BRT

**`sources`** — The GitHub repo the agent clones at the start of each run. The agent has full read/write access to all files in the repo.

**`allowed_tools`** — What the agent is permitted to do. Apply the principle of least privilege:
- Job Hunter needs `WebSearch` and `WebFetch` to search job boards
- CV Optimizer only needs `Read`, `Write`, `Edit` — no web access needed
- Always include `Bash` for `git commit` and `git push`

**`mcp_connections`** — External service connectors. Uses the `connector_uuid` from connectors configured at `claude.ai/customize/connectors`. The `name` field cannot contain spaces or dots.

**`model`** — Which Claude model to use. jobAI uses `claude-sonnet-4-6` for all routines. Use `claude-haiku-4-5-20251001` for simpler tasks with lighter plan usage.

---

## The three routines in this project

### Routine 1 — Daily Job Hunter
```
Trigger ID:       trig_01M7cV6VLteb92jJZMMuvEsQ
Schedule:         0 10 * * *  (every day 10:00 UTC = 7:00 AM BRT)
Model:            claude-sonnet-4-6
Allowed tools:    Bash, Read, Write, Edit, Glob, Grep, WebSearch, WebFetch
MCP connections:  Notion
Next run:         daily
```

**Prompt:**
```
You are the jobAI automated job hunter. Read CLAUDE.md for full context
on the candidate profile and rules, then read .claude/commands/hunt-jobs.md
and execute every step exactly as written. Strict token budget: max 4 searches,
max 2 page fetches, extract from snippets only. Apply location filters, score
each job, save outputs, deduplicate the knowledge base, commit, and push to
origin main. Print the summary and stop.
```

**What it reads:** `CLAUDE.md`, `config.json`, `data/cv.md`, `data/knowledge_base.json`
**What it writes:** `outputs/jobs_YYYY-MM-DD.json`, `data/knowledge_base.json`, `data/kb_YYYY-WNN.json`
**Side effect:** Sends Gmail alert if any job scores ≥ 9

---

### Routine 2 — Weekly CV Optimizer
```
Trigger ID:       trig_01QN7yGs2Ts68K2fZkV6TTxX
Schedule:         0 23 * * 0  (every Sunday 23:00 UTC = 8:00 PM BRT)
Model:            claude-sonnet-4-6
Allowed tools:    Bash, Read, Write, Edit, Glob, Grep
MCP connections:  Notion
```

**Prompt:**
```
You are the jobAI CV optimizer. Read CLAUDE.md for full context on rules
and constraints. Then execute these two steps in sequence:

1. Read .claude/commands/analyze-gaps.md and execute every step exactly as
written. Compare current-week vs prior-week jobs from the knowledge base,
classify surface vs genuine gaps, save outputs/gap_report.md, commit and push.

2. After gap analysis is committed, read .claude/commands/optimize-cv.md and
execute every step exactly as written. Only address ⚠️ surface gaps — never
fabricate anything. Increment the CV version number, save outputs/cv_v[N].md,
update outputs/cv_changelog.md, commit and push.

Complete both steps before stopping. Never invent skills, metrics, or
experience not already in data/cv.md.
```

**What it reads:** `CLAUDE.md`, `config.json`, `data/cv.md`, `data/knowledge_base.json`, KB slices
**What it writes:** `outputs/gap_report.md`, `outputs/cv_v[N].md`, `outputs/cv_changelog.md`

---

### Routine 3 — Weekly Report
```
Trigger ID:       trig_01VmonY9AJUTapxC4zbt6rWD
Schedule:         0 9 * * 1  (every Monday 09:00 UTC = 6:00 AM BRT)
Model:            claude-sonnet-4-6
Allowed tools:    Bash, Read, Write, Edit, Glob, Grep
MCP connections:  Notion
```

**Prompt:**
```
You are the jobAI weekly reporter. Read CLAUDE.md for full context on the
candidate profile and goals, then read .claude/commands/weekly-report.md and
execute every step exactly as written. Load the two most recent weekly KB
slices and gap_report.md, compute data-driven keyword trends with real numbers
(not arrows alone), build the study list from genuine gaps only, save
outputs/weekly_report.md, commit, and push to origin main. Print the summary
and stop.
```

**What it reads:** `CLAUDE.md`, `config.json`, `data/kb_YYYY-WNN.json` (2 most recent), `outputs/gap_report.md`, `outputs/cv_v[latest].md`
**What it writes:** `outputs/weekly_report.md`

---

## How to verify a routine ran

Every routine commits and pushes results to the repo. The git log is the execution log:

```bash
git log --oneline
# feat(hunt): 2026-06-02 · 12 jobs found · job-hunter    ← ran today
# feat(hunt): 2026-06-01 · 8 jobs found · job-hunter     ← ran yesterday
```

If a day is missing from the log, the routine failed silently. Check:
- `claude.ai/code/routines` → click the routine → see run history
- Common causes: GitHub push conflict (another run pushed first), tool permission error

---

## Managing routines

**List all routines:** `/schedule list` in Claude Code, or visit `claude.ai/code/routines`

**Run a routine immediately:** In Claude Code: `/schedule` → select the routine → "run now". Or via the web UI button.

**Update a routine:** `/schedule` → describe what to change. The skill handles the API call.

**Disable a routine:** Update via `/schedule` with `enabled: false`, or toggle in the web UI.

**Delete a routine:** Cannot be done via Claude Code. Go to `claude.ai/code/routines` and delete from there.

---

## Writing effective routine prompts

The most critical part of a Routine is the prompt. Because the agent starts fresh every run with no memory, the prompt must be completely self-contained.

**Pattern used in jobAI:**
```
You are [agent role]. Read CLAUDE.md for full context, then read
.claude/commands/[command].md and execute every step exactly as written.
[Key constraint inline]. [What to save]. [Commit and push].
Print the summary and stop.
```

**Why this pattern works:**
- `Read CLAUDE.md` → agent gets the full behavioral context without you repeating it in the prompt
- `execute every step exactly as written` → prevents the agent from improvising or skipping steps
- Inline constraint (e.g., "Never invent skills") → the most critical rule is stated in the prompt itself, not just in the command file
- `Print the summary and stop` → explicit termination signal prevents the agent from continuing unnecessarily

---

## Limitations

- **Minimum 1-hour interval** — cannot run more frequently than once per hour
- **No persistent state between sessions** — each run starts from scratch by cloning the repo
- **No local file access** — only files in the GitHub repo are available
- **No custom environment variables** — secrets must live in the repo or be passed differently
- **Manual runs require the web UI or Claude Code** — no `workflow_dispatch`-style HTTP trigger from external systems (use the `RemoteTrigger` API instead if needed)

# Prompt Engineering — Patterns Used in jobAI

## What prompt engineering means here

In jobAI, prompt engineering is not about crafting clever phrases for a chatbot. It is about architecting instructions that an autonomous agent follows reliably, repeatably, and safely — without human supervision on each run.

The difference between an agent that works well and one that hallucinates or wastes tokens comes almost entirely from the quality of the prompts and how context is delivered. Seven concrete patterns are used throughout this project.

---

## Pattern 1 — CLAUDE.md as persistent agent memory

**What it is:**
Instead of repeating the candidate profile, location rules, and ethical constraints in every command file, all of that lives in `CLAUDE.md`. Claude reads this file automatically at the start of every session — it acts as the agent's system prompt.

**Why it works:**
Every agent that runs in this project — whether triggered locally or via a Routine — starts with the same behavioral context. You change the rules once in `CLAUDE.md` and all three agents inherit the update on their next run. No duplication, no drift between agents.

```markdown
# CLAUDE.md (excerpt)

## Prime directive
I only work with what exists. I never invent experience, skills,
projects, or achievements that are not already in data/cv.md.
I reframe and surface — I never fabricate.
```

This rule appears once. The CV Optimizer, Gap Analyzer, and Job Hunter all follow it — because all three read `CLAUDE.md`.

**When NOT to use CLAUDE.md for something:**
Frequently-tuned parameters (target roles, salary range, job sources) live in `config.json`. `CLAUDE.md` is for stable behavioral rules. `config.json` is for configurable parameters. Keeping them separate means you can expand the job search scope without touching the agent's behavioral constraints.

---

## Pattern 2 — Explicit token budgets

**The problem without this pattern:**
An agent without limits might decide to run 20 web searches, open 15 full pages, and consume 100k tokens in a single execution. Beyond cost, this creates non-deterministic behavior — results vary based on how much content the agent happened to fetch that day.

**Implementation in `hunt-jobs.md`:**
```markdown
## Token Budget (STRICT — do not exceed)
- Max 4 web searches total across all sources
- Max 2 full page fetches (only if search snippet is insufficient)
- Extract all data from snippets — do not fetch pages unnecessarily
- If budget is reached, stop searching and proceed to scoring
```

**The result:**
Every Job Hunter run consumes between 1,500 and 2,500 tokens. Predictable cost, predictable behavior.

**General rule:**
For any agent that accesses the web or reads many files, define explicit limits in the prompt: "max X searches", "max Y fetches", "read only files matching pattern Z". The agent will respect them.

---

## Pattern 3 — Hard rules stated explicitly and repeated

**The problem:**
LLMs are trained to be helpful and complete patterns. If you ask a model to optimize a CV without explicitly stating what it cannot do, it will "help" by adding skills that seem relevant but that the candidate doesn't have. On a CV, that is misrepresentation.

**Implementation in `optimize-cv.md`:**
```markdown
## Hard Rules — NEVER violate
- NEVER add skills, tools, experiences, or qualifications not present in data/cv.md
- NEVER change company names, job titles, or dates
- NEVER address ❌ genuine gaps — those belong in the study list only
- Rewrite existing bullets only — do not add new bullet points
- Preserve ALL metrics exactly as written (never round up, never estimate)
```

These rules appear multiple times across the command file — in the introduction, in the step-by-step instructions, and in the verification checklist. Deliberate repetition: the model needs to encounter the constraint at multiple points to consistently respect it.

**The surface vs. genuine gap distinction:**
This is the core ethical constraint of the entire system.

```
⚠️ Surface gap:
  The skill EXISTS in the candidate's experience but is not visible in the CV.
  → The agent MAY rewrite the bullet to surface it using market language.
  → Example: candidate used data to make decisions but bullet says "led product"
             Agent rewrites: "led data-driven product decisions, reducing time-to-market by 35%"

❌ Genuine gap:
  The skill DOES NOT exist in the candidate's experience.
  → The agent NEVER adds it to the CV.
  → Goes to the study list. The human decides whether to learn it.
  → Example: candidate has no ML pipeline experience
             Goes to study list as "Machine Learning pipelines — appeared in 5 jobs this week"
```

---

## Pattern 4 — config.json as the single source of truth for parameters

**What it is:**
Any parameter that might need to change in the future lives in `config.json`, not hardcoded in prompts. Command files reference the file rather than embedding values.

```markdown
# In hunt-jobs.md:
Read config.json and use:
- search.target_roles for role matching and scoring
- search.location_rules.accept and .reject for hard filtering
- search.salary_min_usd and salary_max_usd for salary scoring
- search.sources for which job boards to search
```

**Why it matters:**
When a new target role is added to `config.json` (e.g., "AI Builder" — added in card 1.1), all three agents pick it up on their next run. Zero edits to command files. This is the same separation-of-concerns principle from software engineering applied to agent design: separate configuration from logic.

---

## Pattern 5 — Explicit success criteria and termination signal

**What it is:**
Every agent prompt ends with a clear definition of done and an explicit stop instruction.

```markdown
# At the end of hunt-jobs.md:
When complete:
- outputs/jobs_YYYY-MM-DD.json saved with all scored jobs
- data/knowledge_base.json updated (new jobs appended, deduplicated)
- data/kb_YYYY-WNN.json updated (weekly slice current)
- Git commit and push completed
- Print: "Job Hunt complete: X new jobs found, Y duplicates skipped, Z alerts sent"
- Stop.
```

**Why "Stop." explicitly?**
Without it, the agent might continue — refining scores, checking one more source, adding analysis. "Stop." is the termination signal. In Routines, a clean stop means the session ends and resources are released.

---

## Pattern 6 — Versioned, auditable outputs

**CV versioning:**
Every CV Optimizer output is saved as `cv_v[N].md` where N increments on each run. The previous version is never overwritten.

**Why:**
- **Rollback:** If an optimization made the CV worse, you can compare versions and revert
- **Auditability:** The changelog explains every change: what was rewritten, why, which gap it addressed
- **Portfolio evidence:** The version history proves the system works — you can show that specific skills became more visible across multiple optimization cycles

**Changelog format:**
```markdown
## cv_v3.md — 2026-06-08
### Changes
- Meridian Software Group, bullet 3: added "AI and automation integration"
  Why: "AI integration" was a surface gap — present in experience, not visible in CV
  Gap addressed: ⚠️ AI integration (appeared in 7 jobs this week, ↑ from 4)
```

The changelog is machine-generated by the agent — every change is documented without requiring manual notes.

---

## Pattern 7 — Self-contained prompts for stateless agents

**The problem with prompts that assume context:**
In Routines, each execution starts with zero memory. The agent doesn't know what it did yesterday. A prompt that says "continue from where you left off" will fail because the agent has no idea where that was.

**How jobAI prompts are structured:**
```
You are the jobAI automated job hunter.
Read CLAUDE.md for full context on the candidate profile and rules,
then read .claude/commands/hunt-jobs.md and execute every step exactly as written.
```

This instructs the agent to reconstruct all necessary context from the repository files on every run. It is deterministic — the same behavior every execution, regardless of what happened in previous runs.

**The test for a good Routine prompt:**
Imagine handing the prompt to someone who just joined the project and has only access to the repository. Can they execute the task correctly with no other information? If yes, the prompt is self-contained. If no, it has implicit dependencies that will cause failures.

---

## What not to do

**Vague task definition:**
```
❌ "Search for relevant jobs"
✅ "Search these 4 sources for these 12 roles, apply hard location filters,
    score 1–10 using this criteria, save to this file format"
```

**No resource limits:**
```
❌ "Search as many sources as needed"
✅ "Max 4 searches, max 2 page fetches. Stop when budget is reached."
```

**No termination signal:**
```
❌ "Analyze gaps and save the report"
✅ "Save outputs/gap_report.md with these exact sections. Commit and push.
    Print summary. Stop."
```

**Implicit ethical constraints:**
```
❌ (assume the agent knows not to fabricate skills)
✅ "NEVER add skills not present in data/cv.md" — stated explicitly, repeated 3 times
```

**Mixing configuration with logic:**
```
❌ "Search for Product Manager, Senior PM, AI Builder, Automation Specialist..."
   (hardcoded in the prompt — requires editing the prompt to add a new role)
✅ "Read config.json and use search.target_roles"
   (roles are configured separately — add a role in config.json, done)
```

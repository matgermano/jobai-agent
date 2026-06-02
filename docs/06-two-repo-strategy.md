# Two-Repo Strategy — Private Data + Public Portfolio

## The problem

jobAI contains two types of content with opposing requirements:

| Type | Examples | Requirement |
|---|---|---|
| **Architecture** | Commands, workflows, CLAUDE.md, docs | Public — portfolio and showcase |
| **Personal data** | cv.md, config.json, application_profile.json | Private — never expose |

There is also a hard technical constraint: **Routines clone the GitHub repository on every execution.** The agents need `data/cv.md`, `config.json`, and `application_profile.json` to exist in the repo in order to run. You cannot gitignore them without breaking the automation.

This means a single public repo cannot work — it would either expose personal data or break the automation.

---

## The solution — two repos with automatic sync

```
github.com/matgermano/jobAI  (PRIVATE)
├── All personal data lives here
├── All architecture files also live here
├── Routines clone from here — automation works correctly
└── Never made public

        ↓  GitHub Action fires on every push
        ↓  Copies only safe files

github.com/matgermano/jobai-agent  (PUBLIC)
├── Architecture files only (commands, workflows, docs, CLAUDE.md)
├── Anonymized example files (config.example.json, cv.example.md)
└── Portfolio showcase — anyone can see, clone, and adapt it
```

The sync is **fully automatic**. When you push any change to a command file, workflow, or documentation note in the private repo, the public repo updates itself within seconds. You never need to manually copy or sync anything.

---

## What lives where

### Private repo only (never synced)
```
config.json                    ← real salary target, real preferences
data/cv.md                     ← real CV with real employer history
application_profile.json       ← real name, phone, email, LinkedIn
application_narratives.md      ← real STAR stories
data/knowledge_base.json       ← accumulated job search history
data/kb_YYYY-WNN.json          ← weekly job data slices
outputs/                       ← all generated reports and CV versions
.github/workflows/sync-to-public.yml  ← the sync mechanism itself
```

### Public repo (auto-synced from private)
```
CLAUDE.md                      ← agent architecture (uses "Alex Rivera")
README.md                      ← project documentation
roadmap_completo.md            ← full project roadmap
LICENSE + .gitignore           ← standard repo files
config.example.json            ← config structure with fictional data
data/cv.example.md             ← CV structure with fictional data
application_profile.example.json ← profile structure with fictional data
.claude/commands/*.md          ← all agent logic — the main showcase
.github/workflows/*.yml        ← all automation workflows (except sync)
docs/*.md                      ← this documentation folder
```

---

## How the sync workflow works

The file `.github/workflows/sync-to-public.yml` in the private repo:

```yaml
on:
  workflow_dispatch:          ← can be triggered manually at any time
  push:
    branches: [main]
    paths:                    ← only fires when relevant files change
      - 'CLAUDE.md'
      - '.claude/commands/**'
      - '.github/workflows/**'
      - 'docs/**'
      - '*.example.*'
      # ... (other architecture files)

jobs:
  sync:
    steps:
      - Checkout private repo
      - Clone public repo to /tmp/public-repo
      - Copy architecture files (explicitly listed — not a wildcard)
      - Skip sync-to-public.yml itself (private-only workflow)
      - git add -A in public repo
      - If changes exist: commit + push to public repo
      - If no changes: exit silently
```

**Why explicit file lists instead of wildcards?**
Wildcards could accidentally include new personal data files added in the future. Explicit lists mean only the files you've consciously decided are safe will ever be synced. Adding a new file to the public repo requires a deliberate decision to add it to the sync list.

---

## Setup (one-time, already done)

For reference and for anyone adapting this pattern:

### 1. Create the public repo
GitHub → New repository → name: `jobai-agent` → Public → Create (empty)

### 2. Create a Personal Access Token (PAT)
- GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
- Name: `jobai-public-repo-sync`
- Expiration: 1 year
- Scope: `repo`
- Copy the generated token (shown only once, starts with `ghp_...`)

### 3. Add the PAT as a secret in the private repo
- Private repo → Settings → Secrets and variables → Actions
- New repository secret → Name: `PUBLIC_REPO_PAT` → Value: the token
- Save

### 4. First sync
Push any change to a tracked file in the private repo, or trigger the workflow manually via GitHub → Actions → Sync to Public Showcase → Run workflow.

---

## Verifying the sync

After any push to the private repo that touches architecture files:

```bash
# Check the sync workflow ran successfully
gh run list --workflow=sync-to-public.yml --limit=3

# Expected output:
# completed  success  [commit message]  Sync to Public Showcase  main  push
```

You can also visit `github.com/matgermano/jobai-agent` and verify the files updated.

If the sync fails, the most common causes are:
- `PUBLIC_REPO_PAT` secret expired (PATs expire annually by default — regenerate and update the secret)
- The public repo's default branch name differs (should be `main`)
- A merge conflict in the public repo (shouldn't happen with this pattern since the sync always overwrites)

---

## What the public repo shows to an interviewer

Someone reviewing `github.com/matgermano/jobai-agent` sees:

1. **Agent architecture** — `CLAUDE.md` as a system prompt, `config.json` pattern, file structure design decisions
2. **Prompt engineering** — the command files show real token budgets, hard ethical rules, self-contained prompts, explicit success criteria
3. **CI/CD for agents** — GitHub Actions workflows with job dependencies, artifact passing, error handling, output verification steps
4. **Data design** — TTL strategy, URL deduplication, weekly KB slices for trend analysis, versioned CV outputs
5. **MCP integrations** — Notion and Gmail connected as native agent tools
6. **Product thinking** — the roadmap shows phased delivery, learning goals per card, and criteria for done

All of this is visible without any personal data being exposed. The git history in the private repo (which remains private) is the full audit trail of the system in production.

---

## Adapting this pattern for other projects

The two-repo strategy with auto-sync is applicable to any project where:
- The automation needs secrets or personal data in the repo
- You want a clean public showcase of the architecture
- You want the public version to stay current without manual effort

The key elements to replicate:
1. A GitHub Actions workflow in the private repo that copies specific files to the public repo using a PAT
2. Explicit allowlist of files to sync (not wildcards)
3. Anonymized example files for anything personal (config, data, profile)
4. The sync workflow excluded from its own sync list

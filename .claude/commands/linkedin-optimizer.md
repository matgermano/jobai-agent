# /linkedin-optimizer

## Goal
Generate LinkedIn-optimized copy (headline, about, experience bullets) derived from the latest CV version and market keywords. Runs automatically after /optimize-cv every Sunday — no manual input required.

## Token budget
- No web fetches — local files only
- Stop after saving and committing

## Hard rules
- ❌ NEVER suggest adding experience, skills, or achievements not grounded in data/cv.md
- ✅ All suggestions must be traceable to real experience in the CV
- ✅ Suggestions are for human review — never auto-apply to LinkedIn
- ✅ Every suggestion cites which gap or keyword it addresses

## Steps

### 1. Load context
- Find the latest CV version: `ls outputs/cv_v*.md | sort -V | tail -1`
- Read that file (e.g. `outputs/cv_v3.md`) — this is the post-optimization source of truth
- Read `outputs/gap_report.md` — top market keywords and trends
- Read `application_profile.json` — interests and positioning
- Read `config.json` — target roles

### 2. Generate headline (max 220 chars)
The headline must:
- Lead with the primary target role from `config.json`
- Include 1–2 of the top market keywords from `gap_report.md` that appear in the CV
- End with a differentiator (what makes this profile distinct)

Draft 2 variants for the user to choose from.

### 3. Generate about section
Structure: Hook → What you do → Proof → Call to action

- Hook: one sentence that captures the value you deliver
- What you do: 2–3 sentences covering the primary role and key skills (use market keywords)
- Proof: 2–3 quantified achievements pulled directly from the CV
- Call to action: what you're looking for

Max 2,600 characters (LinkedIn limit).

### 4. Generate experience bullets (top 3 roles)
For each of the top 3 roles in `cv_v[latest].md`:
- Take the rewritten CV bullets
- Adapt them for LinkedIn format: slightly more narrative, still metric-led
- Surface any ⚠️ gaps from `gap_report.md` that exist in that role's experience
- Max 3–5 bullets per role

LinkedIn bullets differ from CV bullets: CV is scanned by ATS, LinkedIn is read by humans. Slightly more context is appropriate.

### 5. Save suggestions
Save as `outputs/linkedin_suggestions.md`:

```
# LinkedIn Suggestions — YYYY-MM-DD
_Based on cv_v[N].md and gap_report.md from this week_

---

## Headline
**Option A:** [headline variant 1 — max 220 chars]
**Option B:** [headline variant 2 — max 220 chars]
**Keywords added:** [list]

---

## About Section
[Full suggested text — ready to paste]

**What changed vs current:**
- Added keywords: [list from gap_report]
- Added metrics: [list]

---

## Experience Updates

### [Company — Role]
[Bullet 1]
[Bullet 2]
[Bullet 3]
_Keywords surfaced: [list]_

### [Company — Role]
...

---

## Keyword coverage summary
| Keyword | Market demand | In this suggestion |
|---|---|---|
| [keyword] | X jobs/week | ✅ |
```

### 6. Git commit
```
git add outputs/linkedin_suggestions.md
git commit -m "feat(linkedin): YYYY-MM-DD · linkedin suggestions generated from cv_v[N]"
git pull --rebase origin main
git push origin main
```

### 7. Print summary — then STOP
```
✅ LinkedIn suggestions generated — YYYY-MM-DD
Based on: cv_v[N].md
Saved: outputs/linkedin_suggestions.md
Headline variants: 2
Keywords surfaced: [list]
Review and apply to LinkedIn when ready.
```

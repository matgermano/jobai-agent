# /optimize-cv

## Goal
Rewrite the CV based on gap analysis results. Surface skills the user already has but hasn't highlighted. Align language with market keywords. Never invent anything.

## Token budget rules (STRICT)
- Read only `outputs/gap_report.md` and the sections of `data/cv.md` directly relevant to the top 3 surface gaps — do not read the entire CV on every run
- Do not search the web — all inputs come from local files
- Stop immediately after saving outputs and committing

## Hard rules — read before doing anything
- ✅ Rewrite bullet points to surface existing skills more clearly
- ✅ Inject recurring market keywords naturally into existing experience descriptions
- ✅ Keep executive tone: [action verb] + [context] + [measurable result]
- ✅ Preserve ALL real metrics exactly as written (R$5M, 18M leads, 50% TTM, R$1M+, 40%, 30%, 70%, 25%, 350%, 45%, 20%)
- ❌ NEVER add skills, tools, certifications, or experiences not in data/cv.md
- ❌ NEVER change company names, job titles, or employment dates
- ❌ NEVER remove or soften real quantified achievements
- ❌ NEVER fabricate projects, clients, or outcomes

## Steps

### 1. Load inputs
- Read `data/cv.md` — source of truth
- Read `outputs/gap_report.md` — which items to surface (⚠️ only)
- Read `config.json` — CV rules

### 2. Determine current version number
Check outputs/ for existing cv_v[N].md files.
Next version = highest N + 1. If none exist, start at cv_v1.md.

### 3. Process each ⚠️ item from gap report
For each "surface" item:
- Find the relevant experience in cv.md where this skill was used
- Rewrite the bullet to make that skill explicit and keyword-aligned
- Do not change the core meaning — only the framing and vocabulary

### 4. Rewrite Professional Summary (if needed)
If the summary lacks alignment with top 3 market keywords, rewrite it.
Keep the same achievements and positioning — update vocabulary only.

### 5. Do NOT touch ❌ gap items
Genuine gaps go to the weekly report study list.
They never appear in the CV.

### 6. Save outputs

**Optimized CV:**
`outputs/cv_v[N].md`

Same structure as data/cv.md but with rewritten sections.

**Changelog:**
Append to `outputs/cv_changelog.md`:
```
## cv_v[N] — YYYY-MM-DD

### Changes made
- [Section / Company] — [what changed] — [gap addressed]
- ...

### What was NOT changed
- All metrics preserved: [list]
- All dates, titles, companies: unchanged

### Projected impact
- Fit score improvement: +X.X pts avg (from gap_report)
```

### 7. Git commit
```
git add outputs/
git commit -m "feat(cv): YYYY-MM-DD · cv_v[N] generated · cv-optimizer"
git pull --rebase origin main
git push origin main
```

### 8. Print summary
```
✅ CV optimization complete — YYYY-MM-DD
Version: cv_v[N].md
Changes made: X bullet points rewritten
Keywords added: [list]
Metrics preserved: all
Genuine gaps (not touched): [list]
Projected fit improvement: +X.X pts
Saved to: outputs/cv_v[N].md
Changelog: outputs/cv_changelog.md
```

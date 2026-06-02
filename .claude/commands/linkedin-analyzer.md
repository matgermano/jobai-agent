# /linkedin-analyzer

## Goal
Analyze the current LinkedIn profile against real market keyword data from this week's gap analysis. Generate specific, actionable copy improvements for headline, about section, and experience bullets — grounded entirely in real experience.

## Important: manual input required
This command reads from `data/linkedin_profile.md`.  
**Before running:**
1. Open your LinkedIn profile
2. Copy your current headline, about section, and top 3 experience entries (bullets only)
3. Paste into `data/linkedin_profile.md` following the template structure in that file
4. Save the file, then run `/linkedin-analyzer`

Without this file populated, the command cannot run.

## Token budget
- No web fetches — local files only
- Stop after saving and committing

## Hard rules
- ❌ NEVER suggest adding skills or experience not grounded in `data/cv.md` or `application_profile.json`
- ✅ All suggested copy must be based on real background and real experience
- ✅ Suggestions are recommendations only — never modify `data/cv.md` or `application_profile.json`
- ✅ Every suggestion must cite which gap or keyword it addresses

## Steps

### 1. Load context
- Read `data/linkedin_profile.md` — current LinkedIn content
- Read `outputs/gap_report.md` — market keywords by frequency and trend
- Read `data/cv.md` — source of truth for experience and metrics
- Read `application_profile.json` — structured profile, interests
- Read `config.json` — target roles and location preferences

If `data/linkedin_profile.md` is empty or not filled in, print:
```
⚠️ data/linkedin_profile.md is empty.
Open LinkedIn, copy your headline, about, and top 3 experience sections, and paste into that file.
Then run /linkedin-analyzer again.
```
And stop.

### 2. Analyze headline
The LinkedIn headline is the most important SEO field — it determines who finds you in search.

Evaluate:
- Does it contain the primary target role from `config.json`?
- Does it include any of the top 5 market keywords from `gap_report.md`?
- Is it specific (role + differentiator) or generic ("Professional | Open to opportunities")?
- Character count — LinkedIn shows ~220 characters in search results

### 3. Analyze about section
The About section is a pitch, not a summary.

Evaluate:
- Does it open with a hook that states the value you deliver?
- Does it include quantified achievements?
- Does it use any of the top market keywords?
- Does it reflect the interests from `application_profile.json interests.want_to_do`?

### 4. Analyze experience bullets (top 3 roles)
For each of the top 3 roles in `data/linkedin_profile.md`:

- Which bullets already contain market keywords from `gap_report.md`?
- Which bullets have strong metrics but could surface a gap keyword with a rewrite?
- Are there ⚠️ surface gaps from `gap_report.md` that could be surfaced in these bullets?

### 5. Generate LinkedIn Report
Save as `outputs/linkedin_report.md`:

```
# LinkedIn Profile Analysis — YYYY-MM-DD

## Headline
**Current:** [current headline]
**Issues:** [what's missing]
**Suggested:** [specific rewrite — within 220 chars]
**Why:** [which roles/keywords it improves, which gap it surfaces]

---

## About Section
**Current:**
[current about]

**Issues:** [evaluation — missing hook, no metrics, missing keywords]

**Suggested:**
[full rewrite]

**Why:** [what changed and why — keywords added, positioning improved]

---

## Experience Improvements

### [Company — Role]
| Current bullet | Suggested rewrite | Gap addressed |
|---|---|---|
| [bullet text] | [rewrite] | [keyword from gap_report] |
...

### [Company — Role]
...

---

## Keyword coverage
| Keyword | Market demand | In current profile | In suggested profile |
|---|---|---|---|
| [keyword] | X jobs/week | ❌ / ⚠️ partial / ✅ | ✅ |

---

## Summary
- Keywords added or improved: X
- Headline improvements: [yes/no + what changed]
- About section: [yes/no + what changed]
- Experience bullets improved: X across Y roles
```

### 6. Git commit
```
git add outputs/linkedin_report.md
git commit -m "feat(linkedin): YYYY-MM-DD · linkedin analysis complete · X improvements identified"
git pull --rebase origin main
git push origin main
```

### 7. Print summary — then STOP
```
✅ LinkedIn Analysis — YYYY-MM-DD
Saved: outputs/linkedin_report.md
Headline: [improved / already strong]
About: [improved / already strong]
Experience bullets improved: X
Top keywords to add: [list]
```

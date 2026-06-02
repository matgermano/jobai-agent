# /weekly-report

## Goal
Generate a concise Monday morning report. Give Alex a clear picture of what happened this week: jobs found, market trends, CV status, and exactly what to study to become more competitive.

## Steps

### 1. Load inputs
- **Primary:** If `data/kb_YYYY-WNN.json` files exist, load the two most recent weekly slices.
  - Current week = most recent file
  - Prior week = second most recent file (for trend comparison)
- **Fallback:** Read `data/knowledge_base.json` and filter last 7 days (current) and 8–14 days (prior).
- Read `outputs/gap_report.md` — latest gap analysis
- Read `outputs/cv_changelog.md` — latest CV changes
- Read `outputs/cv_v[latest].md` — current CV version (find highest N in outputs/)

### 2. Calculate stats
- Total jobs found this week
- Jobs that passed location filter
- Jobs with fit_score >= 7 (strong matches)
- Jobs with fit_score 5–6 (potential matches)
- Top 5 companies hiring this week
- Sources that produced the most results

### 3. Extract top keywords with data-driven trends
**Prefer reuse:** If `outputs/gap_report.md` exists and was generated this week, extract its trend table directly — do not recompute.
Otherwise, compute from raw KB data:
- `freq_now` = count in current week
- `freq_prev` = count in prior week (0 if new)
- Trend: ↑ (freq_now > freq_prev), ↓ (freq_now < freq_prev), → (equal), NEW (freq_prev = 0)
- Always write actual counts, never symbols alone — e.g. "↑ 4→7 jobs", never "↑↑↑"
Show top 10 keywords.

### 4. Build study list
Based ONLY on ❌ genuine gaps from gap_report.md:
- List specific skills/tools to learn
- Be concrete: "Learn Amplitude for product analytics" not "improve analytics"
- For each item, suggest ONE free resource (documentation, course, or article)
- Prioritize by frequency in job listings (most demanded = study first)

### 5. Save report
Save as `outputs/weekly_report.md`:

```
# jobAI Weekly Report — Week of YYYY-MM-DD
_Generated Monday, YYYY-MM-DD at 06:00 BRT_

---

## 📊 This week in numbers
| Metric | Value |
|--------|-------|
| Jobs found | X |
| Passed location filter | X |
| Strong matches (score 7+) | X |
| Potential matches (score 5–6) | X |
| Sources with most results | X, Y, Z |

## 🏢 Top companies hiring
1. Company — Role — Score — URL
2. ...

---

## 🔑 Top 10 keywords this week
| Rank | Keyword | This week | Last week | Trend |
|------|---------|-----------|-----------|-------|
| 1 | ... | X jobs | Y jobs | ↑ X→Y |
...

---

## 📄 CV status
- Current version: cv_v[N].md
- Changes this week: [summary of what was rewritten]
- Projected fit improvement: +X.X pts avg

---

## 📚 Study list — what to learn this week
Ranked by market demand:

### 1. [Skill/Tool name]
**Why:** Asked in X% of jobs this week
**What to do:** [Concrete action]
**Resource:** [Link or name of free resource]

### 2. [Skill/Tool name]
...

---

## 💡 One insight this week
[One observation about the market based on data — a trend, a pattern, a shift in what companies are asking for]

---
_Next run: Tuesday 07:00 BRT · CV update: next Sunday 20:00 BRT_
```

### 6. Git commit
```
git add outputs/weekly_report.md
git commit -m "feat(report): YYYY-MM-DD · weekly-report · X jobs · [top keyword]"
git push origin main
```

### 7. Print summary
```
✅ Weekly report generated — YYYY-MM-DD
Jobs this week: X | Strong matches: X | Study items: X
Saved to: outputs/weekly_report.md
```

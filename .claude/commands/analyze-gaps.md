# /analyze-gaps

## Goal
Analyze CV gaps based on jobs found this week. Identify what the market is asking for that isn't visible in the current CV. Compute data-driven keyword trends by comparing this week vs. the prior week.

## Steps

### 1. Load inputs
- Read `data/cv.md` — source of truth
- Read `data/knowledge_base.json` — all jobs found; filter by date
- Read `config.json` — profile rules

### 2. Partition jobs by week
From `knowledge_base.json`:
- **Current week:** `found_at` within the last 7 days
- **Prior week:** `found_at` between 8 and 14 days ago

If weekly KB slices exist (`data/kb_YYYY-WNN.json`), prefer those for cleaner partitioning:
- Current week = most recent weekly file
- Prior week = second most recent weekly file (if it exists)

### 3. For each job in the current week, identify gaps
Two types:
- ⚠️ Surface gap: skill exists in CV but isn't highlighted or keyword-aligned
- ❌ Genuine gap: skill/domain genuinely not present in CV (do not touch in CV)

### 4. Aggregate across all current-week jobs
Count how many jobs require each gap item.
Sort by frequency (most demanded = rank 1).

### 5. Compute trends vs. prior week
For each keyword/gap item:
- Count frequency in current-week jobs → `freq_now`
- Count frequency in prior-week jobs → `freq_prev`
- Trend: ↑ if freq_now > freq_prev, ↓ if freq_now < freq_prev, → if equal, NEW if freq_prev = 0
- Write actual numbers, not symbols alone — e.g. "↑ 5→8 jobs"

### 6. Save report
Save as `outputs/gap_report.md`:

```
# Gap Analysis Report — YYYY-MM-DD

## ⚠️ Surface gaps (can be fixed in CV now)
| Rank | Gap | This week | Last week | Trend | Where in CV | Action |
|------|-----|-----------|-----------|-------|-------------|--------|
| 1 | ... | X jobs | Y jobs | ↑ X→Y | [section] | Reframe bullet at [company] |

## ❌ Genuine gaps (study list — never touch CV)
| Rank | Gap | This week | Last week | Trend | Notes |
|------|-----|-----------|-----------|-------|-------|
| 1 | ... | X jobs | Y jobs | NEW | ... |

## Fit score summary
- Average current fit score: X.X / 10
- Projected fit score after surface fixes: X.X / 10
- Jobs analyzed (current week): X
- Jobs in prior week (for trend): Y
```

### 7. Git commit
```
git add outputs/gap_report.md
git commit -m "feat(analysis): YYYY-MM-DD · X surface + Y genuine gaps found · cv-analyzer"
git push origin main
```

### 8. Print summary
```
✅ Gap analysis complete — YYYY-MM-DD
Jobs analyzed (this week): X | Prior week: Y
Surface gaps (fixable): X
Genuine gaps (study): X
Avg fit score: X.X → X.X projected
Top trending keywords: [keyword1] ↑ X→Y · [keyword2] ↑ X→Y · [keyword3] NEW
Saved: outputs/gap_report.md
```

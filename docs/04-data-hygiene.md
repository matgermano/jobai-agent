# Data Hygiene in AI Agents

## The problem

Agents that accumulate data without cleaning it get progressively worse over time. The analysis degrades because it's built on dirty data, and dirty data produces incorrect insights.

In jobAI, two specific problems arise without hygiene:

**Problem 1 — Duplicate jobs:**
The same job posting can appear on RemoteOK and Remotive in the same week. Without deduplication, the knowledge base stores it twice. The gap analysis then counts that job as two separate instances of demand for its required skills. A skill that appears in 3 real jobs looks like it appears in 6. Study priorities get distorted — you'd optimize for skills that seem high-demand but aren't.

**Problem 2 — Stale data:**
The job market evolves. A skill heavily demanded in October may be saturated by March. Without expiration, the knowledge base mixes market data from a year ago with current data. The weekly report reflects an average of past and present demand — which is neither past nor present. Recommendations become generic and lag behind the actual market.

---

## Solution 1 — URL-based deduplication

Before saving any job to `knowledge_base.json`, the agent checks if the URL already exists. If it does, the job is discarded. Same URL = same posting.

**Why URL, not title?**
Job titles vary slightly between sources. "Senior Product Manager" on Himalayas might appear as "Sr. Product Manager" on Remotive for the same job. The URL is unique per posting regardless of how the title is displayed.

```json
// Before deduplication:
[
  {"url": "https://remoteok.io/jobs/123", "title": "PM at Acme"},
  {"url": "https://remotive.com/job/456", "title": "PM at Acme"},
  {"url": "https://remoteok.io/jobs/123", "title": "PM at Acme"}  ← duplicate
]

// After deduplication (unique_by .url):
[
  {"url": "https://remoteok.io/jobs/123", "title": "PM at Acme"},
  {"url": "https://remotive.com/job/456", "title": "PM at Acme"}
]
```

**Where it happens:**
- The Job Hunter agent deduplicates before appending to `knowledge_base.json`
- The Weekly CV Optimizer workflow also runs a `jq unique_by(.url)` pass as a safety net before the gap analysis step

---

## Solution 2 — 90-day TTL (Time To Live)

Every job entry has a `found_at` timestamp. On each run of the Weekly CV Optimizer, entries older than 90 days are removed:

```bash
CUTOFF=$(date -d '90 days ago' +%Y-%m-%dT%H:%M:%S)
jq --arg cutoff "$CUTOFF" \
  '[.[] | select(.found_at >= $cutoff)] | unique_by(.url)' \
  data/knowledge_base.json > /tmp/kb_clean.json
mv /tmp/kb_clean.json data/knowledge_base.json
```

**Why 90 days specifically?**
- 30 days is too short — misses monthly hiring cycles and creates noise from week-to-week variation
- 180 days is too long — market signals from 6 months ago can actively mislead current strategy
- 90 days = one quarter = enough history to identify real trends without mixing in outdated market conditions

---

## Weekly KB slices

In addition to the full `knowledge_base.json` (rolling 90 days), the system maintains weekly snapshot files: `data/kb_YYYY-WNN.json`.

**Naming convention:**
- `YYYY` = 4-digit year
- `W` = literal "W"
- `NN` = ISO week number with leading zero (01–53)
- Example: `kb_2026-W23.json`, `kb_2026-W22.json`

**Why they exist:**
The gap analysis needs to answer: "Did demand for this skill increase compared to last week?" To answer that with real numbers, you need two isolated weekly datasets. You cannot reliably reconstruct weekly buckets from the full knowledge base after the fact, because a job found Monday gets added to the same KB as a job found Friday.

The slices provide clean, pre-partitioned weekly views:

```
Last week:  "Python" appeared in 4 jobs  (from kb_2026-W22.json)
This week:  "Python" appeared in 7 jobs  (from kb_2026-W23.json)
Report:     "↑ Python 4→7 jobs (+75%)"
```

Without slices, the best you could do is: "↑ Python" — directional but not quantified.

**How they're used:**
The Weekly Report always loads the **2 most recent** KB slices:
```
1. glob data/kb_*.json
2. sort descending (newest first)
3. take first 2 files
4. compare keyword frequency between them
```

---

## Job entry structure

Every entry in `knowledge_base.json` follows this schema:

```json
{
  "title": "Senior Product Manager",
  "company": "Acme Corp",
  "url": "https://himalayas.app/jobs/123456",
  "source": "himalayas",
  "location_type": "remote_worldwide",
  "salary_range": "$90k–$120k",
  "fit_score": 8,
  "key_requirements": [
    "5+ years product management",
    "B2B SaaS experience",
    "data-driven decision making",
    "API product experience"
  ],
  "found_at": "2026-06-02T10:15:33Z"
}
```

The `fit_score` (1–10) is calculated by the Job Hunter based on:
- **Role match** — how closely the title aligns with `target_roles` in `config.json`
- **Skills overlap** — how many `key_requirements` match experience in `data/cv.md`
- **Location compliance** — whether it passes the hard filter rules
- **Salary alignment** — whether the range overlaps with the target in `config.json`

---

## Impact on gap analysis quality

Data hygiene is what makes the gap analysis trustworthy rather than directionally correct:

```
Gap report without hygiene:
  ❌ "Python" — appears in 12 jobs  ← actually 6 jobs, each counted twice

Gap report with dedup + TTL:
  ❌ "Python" — appears in 6 jobs   ← accurate, current, actionable
```

Study list priority is entirely driven by frequency. If frequency numbers are wrong, you optimize for the wrong skills. Clean data = correct priorities = better outcomes.

---

## When to scale beyond JSON files

The current storage approach (flat JSON files in the repo) works well for this volume. Consider migrating to a database when:

- `knowledge_base.json` exceeds ~10 MB (roughly 5,000+ job entries)
- Gap analysis runs start taking more than 30 seconds
- You want to query across multiple dimensions simultaneously (e.g., salary range by role by source)

At that point, SQLite (still local, no infra) or a hosted Postgres would be the right move. The agent code in the command files would need minimal changes — swap `jq` operations for SQL queries.

For the current scale of 10–30 new jobs per day, JSON files with git as storage are simple, portable, and fully auditable.

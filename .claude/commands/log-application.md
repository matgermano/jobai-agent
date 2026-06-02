# /log-application

## Goal
Log a job application in two places: update its entry in `data/knowledge_base.json`, and create or update the matching row in the **Applications** Notion database via Notion MCP.

## Usage
```
/log-application [job URL] [status]
```

**Status options:** `applied`, `interview_scheduled`, `interviewed`, `offer`, `rejected`, `withdrawn`

If no status is given, default to `applied`. If the status is not one of the valid options, stop and list the valid options.

## Hard rules
- ❌ NEVER invent company, role, or source data. If the URL isn't in the KB, create a minimal entry with only what's known (URL + status + date).
- ✅ Match KB entries by **exact URL**.
- ✅ Today's date is the current date in `YYYY-MM-DD` format (timezone America/Sao_Paulo).

## Steps

### 1. Read the knowledge base
Read `data/knowledge_base.json` (it is a JSON array). If it doesn't exist, treat it as `[]`.

### 2. Find the entry by URL (exact match)
Search for an entry whose `url` exactly matches the provided job URL.

### 3. Update or create the entry
**If found:** add/update these fields on that entry, preserving all existing fields:
```json
{
  "applied": true,
  "applied_date": "YYYY-MM-DD",
  "status": "<status from command>",
  "next_step": ""
}
```
If the entry was already logged before, keep the original `applied_date` and only update `status` (and `next_step` if relevant) — e.g. when moving `applied` → `interview_scheduled`.

**If not found:** append a new minimal entry:
```json
{
  "url": "<job URL>",
  "title": "",
  "company": "",
  "source": "",
  "applied": true,
  "applied_date": "YYYY-MM-DD",
  "status": "<status from command>",
  "next_step": "",
  "found_at": "YYYY-MM-DDTHH:MM:SS"
}
```

### 4. Write back
Write the full updated array back to `data/knowledge_base.json`. Do not drop or reorder unrelated entries.

### 5. Sync to the Applications Notion database (Notion MCP)
The Applications database is at: `https://www.notion.so/9c5731d782a544c09eafea0b1787a045`
Data source ID: `collection://5d5e12d4-45e0-469f-a74b-508ed2427760`

1. Use `notion-search` to find an existing page in the Applications database matching this job URL. Search query: the company name or job title.
2. **If a page exists:** use `notion-update-page` to update: `Status`, `Next Step`, `date:Applied Date:start`.
3. **If no page exists:** use `notion-create-pages` with `data_source_id: "5d5e12d4-45e0-469f-a74b-508ed2427760"` and these exact property names:
   - `Name` → `[Job Title] @ [Company]`
   - `Company` → company name from KB entry
   - `Role` → job title from KB entry
   - `userDefined:URL` → the job URL
   - `date:Applied Date:start` → today's date (YYYY-MM-DD)
   - `date:Applied Date:is_datetime` → 0
   - `Status` → map the command status to Notion option: `applied`→`Applied`, `interview_scheduled`→`Interview Scheduled`, `interviewed`→`Interviewed`, `offer`→`Offer`, `rejected`→`Rejected`, `withdrawn`→`Withdrawn`
   - `Source` → source from KB entry (one of: himalayas, remoteok, remotive, wellfound, manual)
   - `Fit Score` → fit_score from KB entry (number, or omit if not in KB)
   If the Notion operation fails, skip silently, note it in the summary, and continue.

### 6. Git commit
```
git add data/knowledge_base.json
git commit -m "feat(log): YYYY-MM-DD · logged application · [Company] — [status]"
git pull --rebase origin main
git push origin main
```

### 7. Print summary — then STOP
```
✅ Application logged
Company: [name] | Role: [title]
Status: [status]
KB updated: data/knowledge_base.json
Notion: [link to Applications database page, or "skipped — database not found"]

When you receive an interview invite:
  /interview-prep [url]  → generates prep + writes it directly into this Notion row
```
STOP after printing the summary.

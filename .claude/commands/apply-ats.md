# /apply-ats

## Goal
Given a job application URL on Greenhouse, Lever, or Ashby, use **Chrome MCP** to navigate to the form, read `application_profile.json`, fill every mappable field, and **STOP before submitting** so the user can review.

## Usage
```
/apply-ats [job application URL]
```

## Prerequisite — Chrome MCP must be connected
This command **requires Chrome MCP**. Before doing anything else, confirm a Chrome MCP browser tool is available in this session.

If Chrome MCP is **not** available, print exactly this and STOP — do nothing else:
```
⚠️ Chrome MCP is not connected.
To use /apply-ats:
1. Install "Claude in Chrome" extension from the Chrome Web Store
2. Connect it at claude.ai/customize/connectors
3. Add the connection to the jobAI page in Notion
4. Re-run this command
```

## Hard rules — read before doing anything
- ❌ **NEVER click Submit or Apply** — always STOP before the final submit button. This is the single most important rule.
- ❌ NEVER fill fields that ask for references unless the user explicitly instructed it
- ❌ NEVER fill any field with information not present in `application_profile.json` or `data/cv.md`
- ❌ NEVER invent answers — if a required field has no mapping, leave it blank and note it
- ✅ Cover letter may be generated inline (2 paragraphs, role-specific), grounded only in real experience

## Steps

### 1. Confirm Chrome MCP
Verify the Chrome MCP tool is available. If not, print the error block above and STOP.

### 2. Detect the ATS platform from the URL
- contains `greenhouse.io` → **Greenhouse**
- contains `lever.co` → **Lever**
- contains `ashby.io` or `jobs.ashby.com` → **Ashby**
- anything else → attempt **generic** form filling, with extra caution (verify each field label before typing)

### 3. Load the profile
- Read `application_profile.json` — the source of truth for all field values
- Read `data/cv.md` — used for the cover letter and any role-context fields
- Note the company name and job title from the page once loaded (for the cover letter and summary)

### 4. Navigate
Use Chrome MCP to open the application URL. If the page has an "Apply" entry button that only opens the form (does not submit anything), you may click it to reveal the form fields. Wait for the form to render before reading fields.

### 5. Read the form, then map fields
Read the actual field labels on the page and map each to the profile. Use this mapping:

| Form field | Source |
|---|---|
| First name | `personal.preferred_name` (first word) |
| Last name | `personal.full_name` (last word) |
| Email | `personal.email` |
| Phone | `personal.phone` |
| LinkedIn URL | `personal.linkedin` |
| Portfolio / Website | `personal.portfolio` |
| GitHub | `personal.github` |
| Location / City | `personal.location` |
| Resume / CV upload | `outputs/cv_[company-slug].md` if it exists, else the latest `outputs/cv_v[N].md` |
| Cover letter | generate inline (2 paragraphs, role-specific) |
| Salary expectation | `preferences.salary_target_usd_annual` |
| Work authorization / sponsorship | `common_form_answers.visa_sponsorship_needed` |
| Pronouns | `personal.pronouns` |
| Years of experience | `common_form_answers.years_of_experience` |
| "Why this role / why us?" | adapt `application_profile.json` material to THIS company |
| "Tell me about yourself" | `personal.preferred_name` + latest role + one differentiator |

Notes on specific fields:
- **Resume upload:** compute the company slug (lowercase, hyphens, no special chars). Check in this order:
  1. `outputs/apply-prep/[company-slug]/cv_[company-slug].md` (from /apply-prep — most tailored)
  2. `outputs/cv_[company-slug].md` (from /cv-by-job — job-specific)
  3. Highest-numbered `outputs/cv_v[N].md` (weekly optimized generic CV)
  Use the first one that exists. If none exist, skip the upload and note it.
- **Cover letter:** 2 short paragraphs. Paragraph 1: who you are + why this company/role (use real positioning). Paragraph 2: one or two relevant real achievements tied to the job's needs. No invented facts.
- **Dropdowns / radios (authorization, sponsorship, pronouns):** select the option matching the profile. If no option matches cleanly, leave it untouched and note it.
- **References:** do not fill unless the user explicitly asked.
- **Any field with no source mapping:** leave blank, add it to the skipped list.

### 6. Fill the form
Fill each mapped field via Chrome MCP. Do NOT touch the submit/apply button. Count fields filled vs. total fields, and track which were skipped (and why).

### 7. STOP and print the review summary — DO NOT SUBMIT
```
⚠️  PAUSED — Review before submitting

Company: [name]
Platform: [Greenhouse/Lever/Ashby/Generic]
Fields filled: X / Y total
Fields skipped (no data): [list, with reason]
Cover letter: [yes/no — generated inline]

Review the form, make any changes, then click Submit yourself.
DO NOT submit automatically — always review first.
```
After printing this, STOP. Do not click any submit, apply, or continue-to-submit button under any circumstances.

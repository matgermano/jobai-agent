# /cv-by-job

## Goal
Generate a CV version tailored to a specific job listing. Given a job URL, fetch the full description, analyze the requirements against the CV, and rewrite relevant bullets to maximize alignment — without inventing anything.

## Token budget
- 1 page fetch for the job URL (required — full description needed)
- Do not fetch additional pages
- Stop after saving and committing

## Hard rules — identical to optimize-cv
- ✅ Rewrite existing bullets to surface relevant skills using the job's language
- ✅ Adjust the Professional Summary to match the role's keywords and level
- ✅ Preserve ALL metrics exactly as written
- ❌ NEVER add skills, tools, certifications, or experiences not in data/cv.md
- ❌ NEVER change company names, job titles, or employment dates
- ❌ NEVER add new bullet points — only rewrite existing ones

## Steps

### 1. Load context
- Read `config.json` — profile rules and target roles
- Read `data/cv.md` — source of truth
- Read `application_profile.json` — structured experience and interests

### 2. Fetch the job
Fetch the job URL provided by the user.
Extract:
- Job title and company name
- Location and remote policy
- Salary range (if shown)
- Key requirements (hard requirements — 5–10 items)
- Nice-to-haves (3–5 items)
- Brief company description (2–3 sentences)

### 3. Analyze fit against CV
Compare job requirements against `data/cv.md`:

**Strong match** — requirement clearly present with strong evidence in CV  
**Weak match** — underlying experience exists but not keyword-aligned (candidate for rewrite)  
**Gap** — requirement not present in CV (note, do not add)

### 4. Generate the tailored CV
Create a version of `data/cv.md` that:
- Rewrites the **Professional Summary** to lead with the skills and experience most relevant to this role, using the job's exact language where the underlying experience matches
- For each **weak match**: rewrite the relevant bullet point to make the skill explicit using the job's keyword vocabulary — same meaning, better alignment
- For each **strong match**: verify the bullet already surfaces the skill clearly; if not, tighten the phrasing
- Leaves **gaps** completely untouched — do not add, do not hint at

Create a company slug from the company name: lowercase, hyphens, no special characters.  
Example: "Acme Corp" → `acme-corp`, "TechCo Ltd" → `techco-ltd`

Save as: `outputs/cv_[company-slug].md`

### 5. Log the changes
Append to `outputs/cv_changelog.md`:
```
## cv_[company-slug].md — YYYY-MM-DD
**Job:** [Job Title] @ [Company]
**URL:** [job URL]

**Strong matches (surfaced clearly):**
- [requirement] → already visible in [section/company]

**Rewrote (weak matches surfaced):**
- [section/company, bullet]: changed "[old phrasing]" → "[new phrasing]" — surfaces [requirement]

**Gaps noted (not added to CV):**
- [requirement] — not present in experience

**Fit estimate:** [X]/10
**Summary:** [1-sentence note on fit]
```

### 6. Git commit
```
git add outputs/
git commit -m "feat(cv): YYYY-MM-DD · cv_[company-slug] tailored for [Job Title] @ [Company]"
git pull --rebase origin main
git push origin main
```

### 7. Print summary — then STOP
```
✅ CV tailored — [Job Title] @ [Company]
Saved: outputs/cv_[company-slug].md
Strong matches: X | Rewrote: X bullets | Gaps noted: X
Fit estimate: X/10
```

# /apply-prep

## Goal
Given a single job URL, generate a complete application package in under 5 minutes: a tailored CV, a company briefing, drafted answers to common open questions, and a matched set of STAR stories. Everything is grounded in real material — nothing is invented.

## Usage
```
/apply-prep [job URL]
```

## Token budget rules (STRICT)
- **1 page fetch** for the job URL (required — full description needed)
- **Max 2 web searches** for company research (e.g. `"[Company] product culture"`, `"[Company] recent news funding"`)
- **Max 1 additional fetch** for the company website or about page — only if the searches leave a real gap
- Hard ceiling: 1 fetch (job) + 2 searches + 1 optional fetch (company). Do not exceed it. Stop researching once you can write each section.

## Hard rules — read before doing anything
- ❌ NEVER add experience, skills, or certifications not in `data/cv.md`
- ❌ NEVER invent metrics, company names, job titles, or employment dates
- ❌ NEVER change real metrics — preserve them exactly as written
- ✅ Tailored CV follows the exact same rules as `/cv-by-job`: rewrite existing bullets to surface relevant skills using the job's language; never add bullets
- ✅ Open question drafts are grounded ONLY in `application_narratives.md` — reframe, never fabricate
- ✅ STAR stories come ONLY from `application_narratives.md` — do not invent new stories

## Steps

### 1. Load context (do not re-read if already loaded)
- Read `config.json` — profile rules and target roles
- Read `data/cv.md` — source of truth for the CV
- Read `application_profile.json` — structured experience, skills, interests
- Read `application_narratives.md` — STAR stories and positioning narratives

### 2. Fetch the job (1 fetch)
Fetch the job URL provided by the user. Extract:
- Job title and company name
- Location and remote policy
- Salary range (if shown)
- Key requirements (hard requirements — 5–10 items)
- Nice-to-haves (3–5 items)
- Brief company description (2–3 sentences)
- Team / product area (what you'd actually be working on, if stated)

Create a **company slug**: lowercase company name, hyphens, no special characters.
Example: "Acme Corp" → `acme-corp`, "TechCo Ltd." → `techco-ltd`.

### 3. Research the company (max 2 searches + 1 optional fetch)
Run up to two web searches to learn:
- What the product / platform is and who it serves
- Culture signals (remote-first? fast-moving startup? enterprise?)
- Recent news — funding rounds, product launches, key hires
- Company size / stage

Suggested searches (adapt to what the job description already told you — skip a search if you already have the answer):
```
Search 1: [Company] product what they do culture
Search 2: [Company] recent news funding 2026 OR launch OR hiring
```
If a critical fact is still missing after the searches, you may do **one** fetch of the company's website or about page. Otherwise do not fetch.

### 4. Tailored CV (replicate /cv-by-job logic inline — do NOT call the command)
Compare the job requirements against `data/cv.md`:
- **Strong match** — requirement clearly present with strong evidence in the CV
- **Weak match** — underlying experience exists but isn't keyword-aligned (candidate for rewrite)
- **Gap** — requirement not present in the CV (note it, never add it)

Produce a tailored CV that:
- Rewrites the **Professional Summary** to lead with the skills and experience most relevant to this role, using the job's exact language where the underlying experience genuinely matches
- For each **weak match**: rewrites the relevant existing bullet to make the skill explicit in the job's vocabulary — same meaning, better alignment
- For each **strong match**: verifies the bullet already surfaces the skill clearly; tightens phrasing only if needed
- Leaves **gaps** completely untouched — do not add, do not hint
- Preserves ALL metrics, company names, titles, and dates exactly as in `data/cv.md`
- Adds no new bullet points

Save as: `outputs/apply-prep/[company-slug]/cv_[company-slug].md`

### 5. Company briefing
Write `outputs/apply-prep/[company-slug]/company_briefing.md` using exactly this structure:
```
# Company Briefing — [Company] — YYYY-MM-DD

## What they do (2–3 sentences)

## Product / platform (what you'd be working on)

## Culture signals (remote-first? fast-moving? enterprise?)

## Recent news (funding, product launches, key hires)

## Why this role fits your background (1 paragraph — specific, not generic)

## Questions to ask in the interview (3–5 smart ones based on research)
```
The "Why this role fits" paragraph must reference specific experience from `data/cv.md` / `application_profile.json` (real companies, real metrics) tied to specific requirements in this job. No generic filler. The interview questions must be grounded in what you actually found in research — not the boilerplate ones.

### 6. Open question drafts
Write `outputs/apply-prep/[company-slug]/open_questions.md`. Draft answers to these five standard questions, each grounded in `application_narratives.md`, each tailored to THIS specific role and company:
1. "Why are you interested in this role / company?"
2. "Tell me about yourself" (30-second version)
3. "What's your biggest strength relevant to this role?"
4. "Describe a time you led a cross-functional initiative"
5. "Where do you see yourself in 3 years?"

Each answer: **3–4 sentences**, in the candidate's voice, using only real experience from the narratives. For Q1, fill the company-specific hook using what you learned in research. For Q4, lead with the STAR story whose requirement best matches this job.

Format:
```
# Open Question Drafts — [Job Title] @ [Company] — YYYY-MM-DD

## 1. Why are you interested in this role / company?
[3–4 sentences]

## 2. Tell me about yourself (30-second version)
[3–4 sentences]

## 3. What's your biggest strength relevant to this role?
[3–4 sentences]

## 4. Describe a time you led a cross-functional initiative
[3–4 sentences — anchored in a real STAR story]

## 5. Where do you see yourself in 3 years?
[3–4 sentences]
```

### 7. STAR story matching
Write `outputs/apply-prep/[company-slug]/star_stories.md`. From `application_narratives.md`, select the **2–3 STAR stories most relevant** to this job's key requirements. For each:
- Story name / title (as it appears in the narratives)
- Why it's relevant to THIS job — name the specific requirement it addresses
- The key metric to lead with (real number from the story)
- One sentence on how to frame the opening

Format:
```
# Matched STAR Stories — [Job Title] @ [Company] — YYYY-MM-DD

## [Story title]
- **Why it fits this role:** [specific requirement it addresses]
- **Lead with this metric:** [real number]
- **How to open:** [one sentence]

## [Story title]
...
```

### 8. Git commit
```
git add outputs/apply-prep/
git commit -m "feat(apply-prep): YYYY-MM-DD · [Job Title] @ [Company]"
git pull --rebase origin main
git push origin main
```

### 9. Print summary — then STOP
```
✅ Application package ready — [Job Title] @ [Company]
Fit estimate: X/10

Files:
  outputs/apply-prep/[company-slug]/cv_[company-slug].md
  outputs/apply-prep/[company-slug]/company_briefing.md
  outputs/apply-prep/[company-slug]/open_questions.md
  outputs/apply-prep/[company-slug]/star_stories.md

Strong matches: X | Rewrote: X bullets | Gaps noted: X
Searches used: X/2 | Fetches used: X/2
```
STOP after printing the summary.

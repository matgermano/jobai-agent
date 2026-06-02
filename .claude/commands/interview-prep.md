# /interview-prep [job-url]

## Goal
Generate a focused interview prep document for a specific job. Combines real-time company research with your CV and narratives to produce prep that sounds genuine — specific stories, specific numbers, specific questions — not rehearsed AI output.

## Usage
```
/interview-prep [job URL]
```

## Token budget (STRICT)
- **If `outputs/apply-prep/[slug]/` already exists:** 0 fetches, 0 searches — read from existing package only
- **If no package exists:** same budget as `/apply-prep` — 1 job fetch + max 2 searches + 1 optional company fetch
- Do not fetch/search beyond this. The prep document is built from local data whenever possible.

## Steps

### 1. Load context
- Read `config.json` — profile, target roles
- Read `data/cv.md` — source of truth
- Read `application_narratives.md` — STAR stories
- Read `application_profile.json` — interests, wants, avoids

### 2. Check for existing apply-prep package
Compute the company slug from the URL: lowercase, hyphens, no special characters.
Examples: "Acme Corp" → `acme-corp`, "PostHog" → `posthog`.

Check if `outputs/apply-prep/[slug]/` exists:
- **YES** → read `company_briefing.md`, `open_questions.md`, `star_stories.md`, `cv_[slug].md` from the package. Skip steps 3 and 4.
- **NO** → run steps 3 and 4 (same research logic as `/apply-prep`).

### 3. Fetch job description (1 fetch — only if no package)
Extract: title, company, full requirements (hard + nice-to-have), team/product context, compensation if shown.

### 4. Research company (max 2 searches + 1 optional fetch — only if no package)
```
Search 1: [Company] product what they do culture remote
Search 2: [Company] recent news funding 2026
```
Optional: 1 fetch of company site if a critical fact is still missing.

### 5. Build interview prep document

**Section A — Quick brief (read this 10 minutes before the call)**
Three lines max. Memorise these.
```
Company: [what they do in one sentence — plain English, no jargon]
Role: [what this person will actually do — the core problem they'll solve]
Your angle: [why you specifically, in one sentence — lead with your strongest match]
```

**Section B — 5 likely questions + your answer**
For each question: what they're really probing + your answer grounded in `application_narratives.md` + the metric to lead with.

Cover these five:
1. "Tell me about yourself" (30-second PM version — open with your strongest metric for this role)
2. "Why this company / this role?" (company-specific — use research from section A)
3. A product execution question tied to the #1 hard requirement in the JD
4. A stakeholder / cross-functional challenge question
5. "Where do you see yourself in 3 years?" (tie to what this role enables)

Format per question:
```
### Q: [Question]
**What they're probing:** [the real concern behind the question]
**Your answer:** [3–4 sentences in your voice, grounded in a specific story from narratives]
**Lead metric:** [the number to open with — exact, from cv.md or narratives]
**Story used:** [name of the narrative story this draws from]
```

**Section C — The hard question**
Identify the 1 requirement in the JD where your CV is weakest.
Write a bridge answer: acknowledge the gap honestly, then redirect to adjacent real experience.
Never fabricate. Weak-but-honest beats polished-but-fake.

Format:
```
### Hard question this role might surface
**The gap:** [requirement + why your CV is weak here]
**The bridge:** [2–3 sentences — what adjacent experience you do have + what you'd do on day 1]
```

**Section D — Salary signals**
- What the market pays for this role/seniority in the remote market (use KB data + general knowledge)
- Is the advertised range (if any) above, within, or below your $70k–$100k target?
- How to handle "what are your salary expectations?": one specific script (range + reasoning)

**Section E — 5 questions to ask them**
Grounded in company research — not generic questions. Each one should signal real homework.
```
1. [About product strategy or where the roadmap is heading — based on news/research]
2. [About how PM interacts with eng/design — reveal the actual working style]
3. [About what success looks like for this role in the first 90 days]
4. [About a specific challenge or tension visible in the JD or company context]
5. [About growth — what does a strong person in this role do next?]
```

**Section F — Watch out for**
1–3 things to clarify or probe based on the JD and research:
- Vague JD language that could signal scope creep or role confusion
- Missing info (no salary listed, no team size, reporting line unclear)
- Red flags: recent leadership churn, layoffs, pivot language, "wear many hats" without structure

### 6. Save output
```
outputs/interview-prep/[slug]/interview_prep_YYYY-MM-DD.md
```

### 7. Git commit
```
git add outputs/interview-prep/
git commit -m "feat(interview-prep): YYYY-MM-DD · [Job Title] @ [Company]"
git pull --rebase origin main
git push origin main
```

### 8. Print summary — then STOP
```
✅ Interview Prep — [Job Title] @ [Company] — YYYY-MM-DD
Source: apply-prep package / fresh research
Slug: [slug]

Quick brief:
  [Company in one sentence]
  [Role in one sentence]
  [Your angle in one sentence]

Hard question flagged: [the requirement to watch out for]
Salary: [within / above / below target — one line]
Saved: outputs/interview-prep/[slug]/interview_prep_YYYY-MM-DD.md
```
STOP after printing the summary.

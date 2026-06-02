# /apply-ats

## Goal
Given a job application URL on Greenhouse, Lever, or Ashby, fill every form field from `application_profile.json` and **STOP before submitting** for user review.

Supports two execution paths:
- **Chrome MCP** (macOS) — if the Chrome MCP connector is available in this session
- **Playwright** (Windows / all platforms) — generates and runs a Node.js script that opens a headed (visible) browser

## Usage
```
/apply-ats [job application URL]
```

## Hard rules — read before doing anything
- ❌ **NEVER click Submit or Apply** — always STOP before the final submit button. This is the single most important rule.
- ❌ NEVER fill fields that ask for references unless explicitly instructed
- ❌ NEVER fill any field with information not in `application_profile.json` or `data/cv.md`
- ❌ NEVER invent answers — if a required field has no mapping, leave it blank and note it
- ✅ Cover letter may be generated inline (2 paragraphs), grounded only in real experience

---

## Steps

### 1. Load context
- Read `application_profile.json` — all field values
- Read `data/cv.md` — for cover letter and role context
- Detect the company slug from the URL (lowercase, hyphens, no special chars)

### 2. Detect execution path
**Check Chrome MCP:** Is a Chrome/browser MCP tool available in this session?

- **Yes → follow Path A (Chrome MCP)**
- **No → follow Path B (Playwright)**

---

## PATH A — Chrome MCP (macOS)

### A1. Detect ATS platform from URL
- `greenhouse.io` → Greenhouse
- `lever.co` → Lever
- `ashby.io` or `jobs.ashby.com` → Ashby
- Other → Generic (proceed with caution, verify each label)

### A2. Navigate and fill
Use Chrome MCP to open the URL. Read the actual field labels on the page and map each to `application_profile.json` using the field mapping table below.

### A3. Stop and print review summary (see end of file)

---

## PATH B — Playwright (Windows / all platforms)

### B1. Check Playwright is installed
Run:
```bash
node -e "require('playwright')" 2>/dev/null && echo "OK" || echo "NOT_INSTALLED"
```

If `NOT_INSTALLED`, print exactly this and STOP:
```
⚠️ Playwright is not installed. Run these commands once to set it up:

  npm install playwright
  npx playwright install chromium

Then re-run /apply-ats.
```

### B2. Detect ATS platform from URL
- `greenhouse.io` → Greenhouse
- `lever.co` → Lever
- `ashby.io` or `jobs.ashby.com` → Ashby
- Other → Generic (fetch the page to read field selectors)

### B3. Fetch the application page
Fetch the job URL to read the HTML and confirm field names/IDs for this specific form. For known platforms, use the standard selectors below — only fetch if you need to verify an unusual field.

### B4. Generate the Playwright script
Create `outputs/apply-ats/[company-slug]-apply.js` using this template, filled with real values from `application_profile.json`:

```javascript
const { chromium } = require('playwright');

(async () => {
  const browser = await chromium.launch({ headless: false, slowMo: 100 });
  const context = await browser.newContext();
  const page = await context.newPage();

  console.log('Opening application form...');
  await page.goto('[JOB_URL]', { waitUntil: 'networkidle' });

  // ── GREENHOUSE SELECTORS ──────────────────────────────────────
  // Adapt selectors based on platform detected in B2.
  // For Greenhouse:
  try { await page.fill('input#first_name', '[FIRST_NAME]'); } catch {}
  try { await page.fill('input#last_name', '[LAST_NAME]'); } catch {}
  try { await page.fill('input#email', '[EMAIL]'); } catch {}
  try { await page.fill('input#phone', '[PHONE]'); } catch {}
  try { await page.fill('input[id*="linkedin"]', '[LINKEDIN_URL]'); } catch {}
  try { await page.fill('input[id*="website"]', '[PORTFOLIO_URL]'); } catch {}
  try { await page.fill('input[id*="location"]', '[LOCATION]'); } catch {}

  // Cover letter (if present)
  const coverLetter = `[COVER_LETTER_PARAGRAPH_1]

[COVER_LETTER_PARAGRAPH_2]`;
  try { await page.fill('textarea[id*="cover"]', coverLetter); } catch {}

  // ── END OF FIELD FILLING ──────────────────────────────────────

  console.log('');
  console.log('✅ Form filled. Review the browser window before submitting.');
  console.log('   Make any corrections needed, then click Submit yourself.');
  console.log('   The browser will stay open for 10 minutes.');
  console.log('   Press Ctrl+C in this terminal when you are done.');
  console.log('');

  // Keep browser open for review — do NOT submit
  await page.waitForTimeout(10 * 60 * 1000);
  await browser.close();
})();
```

**Platform-specific selectors to use:**

**Greenhouse:**
```
input#first_name, input#last_name, input#email, input#phone
input[id*="linkedin_profile"], input[id*="website"]
input[id*="location"], textarea[id*="cover_letter"]
```

**Lever:**
```
input[name="name"]  (full name — combine first + last)
input[name="email"], input[name="phone"]
input[name="urls[LinkedIn]"], input[name="urls[Other]"]  (portfolio)
textarea[name="comments"]  (cover letter)
```

**Ashby:**
```
input[autocomplete="given-name"], input[autocomplete="family-name"]
input[autocomplete="email"], input[autocomplete="tel"]
input[placeholder*="LinkedIn"], input[placeholder*="portfolio"]
textarea[placeholder*="cover"], textarea[placeholder*="message"]
```

**Cover letter** (2 paragraphs, write inline):
- Paragraph 1: who you are + why this specific company/role (use real positioning from application_profile.json)
- Paragraph 2: 1–2 real achievements from cv.md tied to the job's key requirements

### B5. Run the script
```bash
node outputs/apply-ats/[company-slug]-apply.js
```

The browser opens headed. The script fills every mapped field with `slowMo: 100` so you can watch it happen. When done, it prints the review message and waits.

---

## Field mapping (both paths)

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
| Resume / CV | Check: `outputs/apply-prep/[slug]/cv_[slug].md` → `outputs/cv_[slug].md` → latest `outputs/cv_v[N].md` |
| Cover letter | Generated inline (2 paragraphs) |
| Salary expectation | `preferences.salary_target_usd_annual` |
| Work authorization | `common_form_answers.visa_sponsorship_needed` |
| Pronouns | `personal.pronouns` |
| Years of experience | `common_form_answers.years_of_experience` |
| "Why this role?" | Adapt `application_profile.json` to this company |
| References | **SKIP** — do not fill |

---

## Final summary (both paths)

After the form is filled:
```
⚠️  PAUSED — Review before submitting

Company: [name]
Platform: [Greenhouse/Lever/Ashby/Generic]
Execution: [Chrome MCP / Playwright]
Fields filled: X
Fields skipped (no data): [list with reason]
Cover letter: generated inline

Review the form, make any corrections, then click Submit yourself.
The script will keep the browser open for 10 minutes (Playwright path).
```

STOP after printing. Do not submit under any circumstances.

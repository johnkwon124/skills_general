---
name: deal-onepager
description: >
  Generates a structured deal introduction One Pager as a DOCX for pipeline meeting packages.
  Based on web research + optional meeting transcript, produces a Pain Points / Solution /
  How it Works / Funding / Founders / Key Milestones structured document. Wilson verifies
  dates, unconfirmed person info, and fabricated figures before delivering the final file.

  ALWAYS trigger when ANY of these apply:
  - "make a one pager", "deal intro doc", "pipeline meeting package", "company intro"
  - A new deal is being added to a weekly pipeline meeting ("add X to this week's pipeline meeting")
  - User attaches a transcript or meeting notes and asks to "turn it into a one pager"
  - "deal summary for CEO", "company intro for LPs", "internal review one pager"

  Difference from Meeting Notes skill:
  - Meeting Notes → post-meeting internal record, faithful to transcript
  - One Pager → pre-meeting / pipeline registration, structured summary suitable for sharing
---

# Deal One Pager

## Overview

A structured company introduction document prepared whenever a new deal enters the pipeline.
Section structure and review criteria are fixed; the primary goal is to prevent date and
person information errors from entering the document.

**Common principles**:
- Length: Letter 1–1.5 pages (sufficient whitespace, not dense)
- Language: English. Natural, concise prose + bullet combination
- Figures require source citation or "(Company-stated)" tag. No fabricated figures
- Date expressions ("this summer", "this year", "next year") → always convert to absolute year based on meeting date
- Person information from sources older than 2 years → remove; keep only co-founders
- **DOCX format must follow the "Output Format Spec" section below**

---

## Step 0 — Input Collection

After skill trigger, confirm the following:

```
Required:
  · Company name
  · Meeting date (required for date verification — must be requested if missing)

Optional (use if available):
  · Transcript or meeting notes
  · Sector hint (e.g., "SST", "BECCS", "AI infra")
```

If meeting date is missing, do not proceed. Request as follows:

```
"Meeting date is needed — please provide the date and I'll proceed immediately."
```

---

## Step 1 — Research

**Source selection**: Follow the sector-based source mapping table from the Meeting Notes skill.
One Pager-specific additional sources:

```
[Common additions]
  · Founder LinkedIn (recent experience only — verify current position)
  · Most recent funding round announcement (within 6 months preferred)
  · Official customer/partnership announcements

[If transcript available]
  · Extract figures/date statements from transcript → create Flag list
  · Flag all relative date expressions: "this year", "this summer", "next year"
  · Flag names/titles mentioned → verify current position on LinkedIn
```

**Collection rules**:
- Figures → cite source
- Company's own statements → tag "(Company-stated)"
- Unverified person info (source 2+ years old) → tag [REMOVE]
- Relative date expressions → tag [DATE: needs conversion based on meeting date]

---

## Step 2 — Wilson Content Pre-Approval

Before DOCX writing, Soto submits the following outline to Wilson:

```
[Wilson pre-approval request format]
"Section structure confirmation:
 1. Market Pain Points — [number of items]
 2. Solution — [core value proposition in one line]
 3. How It Works — [number of technical steps]
 4. Funding — [number of rounds, total amount]
 5. Founders — [persons included, source year]
 6. Key Milestones — [year range]

 Date flags: [list of items to convert]
 Person flags: [remove/keep decision list]
 Approve to begin DOCX generation."
```

---

## Step 3 — DOCX Generation

### Fixed Section Structure

```
[Title: Company Name]
[Subtitle: one-line tagline — italic]
[Meta row: HQ | Founded | Stage | Website]
─────────────────────────────────────────
1. Market Pain Points
2. Solution
3. How It Works
4. Funding
5. Founders & Key Team
6. Key Milestones to Watch
─────────────────────────────────────────
[Footer: "Prepared by [Your Firm Name] | Sources: ..."]
```

### Section-by-Section Writing Guide

**1. Market Pain Points**
- Bullet Type C (plain): structural market problems, customer pain points
- No generalities ("X is a growing market") — specific figures/examples or omit
- 3–5 bullets max

**2. Solution**
- Lead prose 1–2 sentences: core value proposition summary (company name **Bold**)
- Bullet Type A ("**Bold Label**: description"): 3–5 differentiation points
- Separate items for dual revenue / customer segments / deployment method if applicable

**3. How It Works**
- Bullet Type A: technical steps or operating principles in sequence
- Maintain order where steps are clear (input → process → output)
- Technical terms: add brief parenthetical explanation

**4. Funding**
- Bullet Type A: amount/date/key investors per round
- Total cumulative amount stated on first line
- Include offtake/revenue contracts as separate item if applicable

**5. Founders & Key Team**
- **Co-founders only by default** (CEO/CTO)
- Other team members only if directly confirmed from transcript
- Source older than 2 years → remove. No exceptions
- Format: "**Name — Title**: Previous role + one-line background"

**6. Key Milestones to Watch**
- Bullet Type A: "**Year/Quarter**: Milestone"
- No relative date expressions — absolute year based on meeting date only
- Company-stated milestones: append "(Company-stated)"

### DOCX Code — Defensive Code Required

```javascript
// 1. Package check
const { Document, Packer, Paragraph, TextRun,
        AlignmentType, LevelFormat, BorderStyle } = require('docx');

// 2. Output path
const outputDir = "/mnt/user-data/outputs";
if (!fs.existsSync(outputDir)) fs.mkdirSync(outputDir, { recursive: true });

// 3. Full try/catch wrapper
try {
  // ... DOCX generation ...
  Packer.toBuffer(doc).then(buffer => {
    fs.writeFileSync(`${outputDir}/${filename}`, buffer);
    console.log(`Saved: ${filename}`);
  });
} catch(e) { console.error(e); process.exit(1); }
```

Consult the `docx` skill's SKILL.md before generating.

---

## Step 4 — Wilson Review

DOCX must pass all checklist items before delivery to the user.
Items that fail: provide feedback to Soto for rework (maximum 2 loops).

### Wilson Review Checklist

```
Date review:
[ ] No relative date expressions ("this summer", "this year", "next year")
[ ] All dates expressed as absolute year based on meeting date
[ ] Future milestones tagged with "(Company-stated)"

Person information review:
[ ] Only co-founders (CEO/CTO) included unless transcript-confirmed
[ ] Person info from sources 2+ years old fully removed
[ ] Titles/affiliations expressed as current (former positions marked "former")

Figures/data review:
[ ] No fabricated/estimated figures
[ ] Company's own figures tagged "(Company-stated)"
[ ] Figures without clear source removed

Structure review:
[ ] All 6 sections present (Pain Points / Solution / How it Works / Funding / Founders / Milestones)
[ ] Meta row (HQ / Founded / Stage / Website) below title
[ ] Footer included ("Prepared by [Your Firm Name] | Sources: ...")
[ ] Length within Letter 1–1.5 pages

Format review:
[ ] Title: Calibri Bold 18pt+
[ ] Section headers: Calibri Bold 11pt, bottom border
[ ] Bullets: Calibri Regular 10pt (size: 20)
[ ] Page: Letter, margins 0.75–1 inch
```

---

## Output Format Spec

### Document Title
- Text: Company Name (standalone)
- Font: Calibri Bold, 18pt (size: 36)
- Alignment: left

### Subtitle (tagline)
- Font: Calibri Italic, 11pt (size: 22)
- One-line core value proposition summary

### Meta Row
- Font: Calibri Regular, 9.5pt (size: 19)
- Bold labels + regular text: "**HQ**: El Segundo, CA  |  **Founded**: 2022  | ..."

### Section Headers
- Font: Calibri Bold, 11pt (size: 22)
- Bottom border: BorderStyle.SINGLE, size: 6, color: "000000"
- Spacing: 12pt above, 4pt below

### Bullet Points

**Type A — Spec Bullets** (Solution, How it Works, Funding, Milestones):
```
• **Bold Label**: description text
```

**Type C — Plain Bullets** (Market Pain Points):
```
• plain text description
```

**Common bullet specs**:
- Symbol: round bullet (LevelFormat.BULLET, text: "•")
- Indent: left: 320, hanging: 320
- Font: Calibri 10pt (size: 20)
- Bullet spacing: 2–3pt (after: 30)

### Footer
- Top border: BorderStyle.SINGLE, size: 4, color: "CCCCCC"
- Font: Calibri Italic, 8pt (size: 16), color: "888888"
- Text: "Prepared by [Your Firm Name]  |  Sources: [research source list]"

### Page Setup
- Paper: Letter (width: 12240, height: 15840 DXA)
- Margins: top/bottom 1080, left/right 1260 (0.75 inch)

---

## File Naming Convention

```
[Company Name] — One Pager.docx
Example: Arbor Energy — One Pager.docx
```

---

## Wilson Final Delivery Format

```
On pass:
"[Company Name] One Pager complete. Key flag resolutions:
 · Date conversions: [conversion list]
 · Persons removed: [removal list or 'none']
 · Figures tagged: [tag list or 'none']"

On fail:
"Review failed items: [list]
 Sending back to Soto for rework."
```

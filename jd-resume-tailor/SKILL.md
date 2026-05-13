---
name: jd-resume-tailor
description: >
  JD-based tailored resume generator. Uses your master resume as the source and
  reconfigures it to be a perfect fit for the presented JD — selecting relevant experience,
  rewriting bullet language, and outputting DOCX + PDF.

  Trigger when ANY of the following applies:
  - "tailor my resume to this JD", "resume for this position", "resume tailoring"
  - LinkedIn URL or JD text pasted with a resume request
  - "rework my resume", "adjust this for the JD"
  - Specific company name + "apply" + resume mentioned
---

# JD Resume Tailor

Uses your master resume to produce a JD-tailored DOCX + PDF.
**Always follow this sequence. Do not skip steps.**

---

## File Structure

```
jd-resume-tailor/
├── SKILL.md                        ← this file
└── references/
    ├── master-resume.md            ← your experience data + tags (always load)
    ├── intro-variants.md           ← intro variant versions (load only if finance-heavy/bd-heavy)
    └── docx-template.js            ← DOCX format engine (replace CONTENT block only)
```

---

## PHASE 0 — Input Collection

### JD Collection
- **Pasted text**: use as-is
- **URL provided**: extract via `web_fetch` → convert to text

### Reference Load
Read `references/master-resume.md`. (Once only.)

---

## PHASE 1 — JD Analysis (Lightweight)

Do not keep the full JD text in context.
Compress to the **JD Summary** below (7 fields only), then discard the original.

```
[JD Summary]
COMPANY:        {company name}
ROLE:           {title}
CORE_FUNCTION:  {core role in 1–2 sentences}
MUST_HAVES:     {required competency tags, max 8, comma-separated}
NICE_TO_HAVES:  {preferred competency tags, max 5}
SECTOR_CONTEXT: {industry/team context in 1 sentence}
RED_FLAGS:      {requirements that could be weaknesses for this candidate (or "None")}
```

MUST_HAVES/NICE_TO_HAVES should use Tags from the `master-resume.md` Tags column where possible.

---

## PHASE 2 — Gap Analysis & Mapping

Match JD Summary × `master-resume.md` tags to make the following decisions.

### JD Type Determination
- **finance-heavy**: MUST_HAVES contains 2+ of `financial-modeling`, `corporate-finance`, `accounting`, `treasury` → load `intro-variants.md`
- **bd-heavy**: MUST_HAVES contains 2+ of `deal-sourcing`, `bd`, `partnership` → load `intro-variants.md`
- **other**: use default intro from `master-resume.md` (do not load `intro-variants.md`)

### Section Selection Rules

**Summary**
- Write fresh in Phase 3, based on COMPANY, ROLE, and SECTOR_CONTEXT
- Do not recycle existing Summary text

**Experience bullets**
- MUST_HAVES tag matches: always include, place at top
- NICE_TO_HAVES tag matches: include if space permits
- No-match bullets: remove
- Bullet count per role: calibrate based on seniority and relevance (most recent/relevant firm gets most bullets; earlier roles get 1–3)

**Selected Deals**
- Choose 4–6 deals closest to SECTOR_CONTEXT
- Do not include deals from companies you did not actually invest in

**Core Competencies**
- Left column: MUST_HAVES items take priority
- Right column: SECTOR_CONTEXT-relevant domain items + Tools (JD-mentioned tools first)
- Intentional asymmetry in left/right item counts (reduces AI-detection signal)

---

## PHASE 3 — Content Rewriting

### Tagline
3 keywords combined from ROLE and SECTOR_CONTEXT.
Example: `"Strategic Finance  ·  AI Infrastructure  ·  Deal Execution"`

### Summary (3 sentences or fewer)
- Sentence 1: Years of experience + career trajectory (toward JD sector)
- Sentence 2: Key performance figures (minimum 2: drawn from master-resume.md verified numbers only)
- Sentence 3: Connection to this specific position + relevant credential (MBA, CFA, etc.)

### Human Writing Rules (AI Detection Score target: ~20%)
1. Vary bullet start points: mix verb-start / number-start / context-start
2. Intentional bullet length variation: ~30% short / ~40% medium / ~30% long
3. Prohibited phrases: "Proven track record", "Combines X with Y", "Collaborated with cross-functional stakeholders"
4. Use only figures present in `master-resume.md`. No fabricated numbers.
5. Naturally absorb 1–2 phrases from the JD (do not copy-paste)

---

## PHASE 4 — DOCX Generation

### Procedure

```bash
# 1. Locate skill directory
SKILL_DIR=$(find /mnt/skills -type d -name "jd-resume-tailor" 2>/dev/null | head -1)

# 2. Copy template
cp "$SKILL_DIR/references/docx-template.js" /home/claude/resume_tailor_run.js

# 3. Inject Phase 2–3 results into CONTENT block (str_replace)
#    - summary: Phase 3 written text
#    - contact.tagline: Phase 3 Tagline
#    - jobs[]: selected bullets array
#    - deals[]: selected deals array
#    - competencies: left/right items

# 4. Run
node /home/claude/resume_tailor_run.js
```

### File Naming Convention
`{Company}_{RoleAbbr}_Resume_{MMDDYYYY}.docx`
Example: `Anthropic_CorpDev_Resume_05122026.docx`

---

## PHASE 4.5 — PDF Conversion (run immediately after DOCX)

```bash
libreoffice --headless --convert-to pdf \
  --outdir {output directory} \
  {DOCX path}
```

- Filename: same base name as DOCX, extension `.pdf`
- If conversion fails: report the error and proceed with DOCX only
- Copy both DOCX + PDF to workspace

---

## PHASE 4.6 — Automated Verification (run immediately after generation)

```bash
DOCX_SKILL=$(find /mnt/skills -path "*/docx/scripts/office/validate.py" 2>/dev/null | head -1)

# Structure validation
python3 "$DOCX_SKILL" /home/claude/resume_output.docx

# Figure verification (count = 0 means figure missing — recheck immediately)
pandoc /home/claude/resume_output.docx -o /tmp/resume_check.txt 2>/dev/null
echo "=== Figure verification ==="
for term in $(python3 -c "
import json
with open('references/master-resume.md') as f:
    content = f.read()
# Extract configured check terms from master-resume.md
# Add key figures here that must appear in the final output
"); do
  count=$(grep -c "$term" /tmp/resume_check.txt 2>/dev/null || echo 0)
  echo "$term: $count"
done
```

Configure the verification terms based on key figures in your `master-resume.md`
(e.g., fund AUM, deal multiples, portfolio company count, graduation year).
Count = 0 → section missing. Fix via `str_replace` on that section, then re-run.

---

## PHASE 5 — Final Output and Feedback Loop

Save DOCX + PDF to workspace and provide links.

```bash
cp /home/claude/resume_output.docx "/mnt/user-data/outputs/{filename}.docx"
cp /home/claude/resume_output.pdf  "/mnt/user-data/outputs/{filename}.pdf"
```

Accept one round of user revision instructions after DOCX output.

### Revision Principles
- Section-level `str_replace` surgical edits only
- No full regeneration
- Edit → validate → re-output DOCX → re-convert PDF

### On Completion
- Add `_final` to the final filename
- Do not overwrite previous version

---

## Token Efficiency Principles

| Principle | Method |
|-----------|--------|
| Discard JD text early | Keep only 7-field JD Summary after Phase 1; discard original |
| Master resume loaded once | Read once in Phase 2, select, then discard |
| Conditional load of intro-variants.md | Load only if finance-heavy / bd-heavy |
| Reuse DOCX engine | Replace CONTENT block only; no format engine rewriting |
| str_replace first | For feedback integration, edit section only — no full regeneration |
| Minimize interim reporting | Only surface: Phase 1 JD Summary + Phase 4.6 verification + final file |

---

## Error Handling

| Situation | Response |
|-----------|----------|
| URL fetch fails | Ask user to paste JD text directly |
| validate fails | Read error message → `str_replace` fix on that section |
| Figure verification = 0 | Recheck that section content and fix |
| No JD tag matches | Include capital-deployment + executive-communication bullets as defaults |
| Figure uncertain | Never use figures not present in `master-resume.md` |
| PDF conversion fails | Provide DOCX only and report reason |

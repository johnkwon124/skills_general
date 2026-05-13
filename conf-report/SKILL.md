---
name: conf-report
description: >
  Generates a professional conference participation report as a DOCX. Use whenever
  a user attended a conference and wants a structured internal document: sector theme
  diagnosis, session deep dives with analytical insight, session briefs for sessions
  not attended, VC/startup meeting notes, and a conclusion with investment implications.
  Works for any conference (CERAWeek, OCP Summit, Davos, CES, Climate Week). Wilson
  delegates conference report tasks to this skill.

  ALWAYS trigger when ANY of these apply:
  - "write conference report", "conference recap", "participation report", "recap report"
  - User returned from a conference and has session notes or transcripts to work from
  - Combining session notes + startup meetings + VC meetings into one document
  - "summarize session notes into a report", "conference notes to report"
  - Wilson receives a delegated conference report task
  - Post-conference internal report or CEO participation brief

  Do NOT use for: session summaries only, emails, pitch decks, blog posts, or market maps.
---

# Conference Participation Report Skill (v2)

## Overview

This skill generates an **analysis-driven DOCX report** from conference participation records.
The goal goes beyond simple relay — each session includes critical evaluation and analytical insight.

**Input processing principle**: No bulk file loading. In Phase 0, files are classified by type,
and only necessary files are loaded sequentially after user confirmation.

**Web search hard cap**: Section-level limits must be strictly followed. No exceptions.

---

## Report Structure (Fixed)

| Part | Title | Description | Skip Condition |
|------|-------|-------------|----------------|
| I    | Sector Theme Diagnosis | Overall sector direction from a VC investment perspective | Required (no skip) |
| II   | Session Deep Dive | Sessions attended: recap + analytical insight + web supplement | Skip if no sessions attended |
| III  | Sessions Not Attended | Key message + recent trend summary | Skip if none |
| IV   | Meeting Notes | VC + startup meeting cards | Skip if no meetings |
| V    | Conclusion | Investment implications + follow-up actions | Required (no skip) |

**Page target**: No fixed value. Determined naturally: sessions attended × 1.5p + Part I (2p) + Part IV (meetings × 0.5p) + Part V (1p).

---

## Web Search Hard Cap (strictly enforced)

| Section | Purpose | Limit |
|---------|---------|-------|
| Part II (attended sessions) | Verify uncertain figures/claims, fill background gaps | Max 2 per session |
| Part III (missed sessions) | Recent 1-year trends on the topic | Max 1 per session |
| Part IV (meetings) | Startup funding status, VC portfolio verification | Max 1 per item |
| Part I, V | Synthesis based on collected information | No additional search |
| **Total cap** | | **Sessions × 2 + Meetings × 1 + 5 (missed sessions)** |

---

## Phase 0: Input Assessment and File Classification

### 0-A. Metadata Confirmation

Do not re-ask for items already clear from conversation context. Ask only for missing items.

**Required:**
- Conference name, dates, location
- Your firm name
- Report language (default: English)
- Workspace folder path

### 0-B. File List and Type Assignment

```bash
python3 -c "
import os
folder = 'WORKSPACE_PATH'
files = sorted([f for f in os.listdir(folder) if f.endswith(('.docx','.txt','.pdf')) and not f.startswith('~')])
for i, f in enumerate(files):
    print(f'[{i+1}] {f}')
" 2>/dev/null
```

Present the file list to the user and confirm type assignments:

```
[File List]

Please confirm the type for each file:
[A] Transcript — session recording → Part II
[B] Notes/Memos — missed session notes → Part III
[C] Meeting Memo — VC/startup meeting → Part IV
[D] Agenda — reference (used in Parts I, III)
[E] Exclude

Example: enter as "1-A, 2-A, 3-C, 4-D"
```

Proceed with file reading only after user confirms assignments.

### 0-C. Additional Confirmation (ask if not provided)

- Startup meeting company list (extract from files if not available)
- VC/investor meeting counterparty list
- Portfolio company list (to be marked with ★)

---

## Phase 1: File Reading (Sequential by Type)

Read files by type. Do not bulk-load all at once.

### Transcript / Notes DOCX Reading

```bash
pip install python-docx --break-system-packages -q 2>/dev/null
python3 -c "
from docx import Document
doc = Document('FILE_PATH')
for p in doc.paragraphs:
    if p.text.strip(): print(p.text)
" 2>/dev/null
```

### PDF Agenda Reading

```bash
pip install pymupdf --break-system-packages -q 2>/dev/null
python3 -c "
import fitz
doc = fitz.open('FILE_PATH')
for page in doc:
    print(page.get_text())
" 2>/dev/null | head -400
```

**Reading principle**: Read each file once, then proceed. No re-reading the same file.

---

## Phase 2: Section Content Generation

### Part I — Sector Theme Diagnosis

Synthesize all session information (transcripts + notes + agenda). **No additional web search.** Judgment based only on inputs already collected.

Content to cover:
- Directional signals for this sector from the conference (1–2p)
- 3–5 recurring keywords/themes with supporting evidence
- Shifts in discussion direction vs. prior year (if agenda available)
- Investment implications: which technology/business areas are attracting focus

---

### Part II — Session Deep Dive (Attended Sessions)

Apply the following 3-block structure to each session.

#### [Session Recap]
Summarize the session's key arguments based on the transcript.
- Speaker's main claims
- Data/examples presented
- Key Q&A content (if applicable)

Write as a narrative following the session's logical flow, not a simple list.

#### [Gap Fill] *(only if applicable)*
Use web search to supplement areas where the transcript is uncertain or lacks supporting evidence. **Maximum 2 searches per session.**
- Supplement with source citation (e.g., `IEA World Energy Outlook 2024`)
- If search results conflict with transcript claims, present both sides

#### [Analytical Insight]
Critically evaluate the session from a VC investment perspective.
- Logical strengths and weaknesses of the speaker's argument
- Gaps vs. market reality (if any)
- Key points to watch or risks from an investment perspective
- How this session connects to the broader sector direction

**Writing standard**: Maintain objective perspective. Avoid positive/negative biased adjectives.
Base judgments on transcript content and web research; flag when reasoning is inferential.

---

### Part III — Sessions Not Attended

Based on agenda or user notes, summarize sessions worth noting that were not attended.

For each session:
- Session title + speaker (from agenda)
- Summary of key trends from the past 12 months (**1 web search**)
- Why this session is worth monitoring from an investment perspective (1–2 sentences)

**Writing principle**: Since the session was not directly observed, focus on fact-based summaries from search. No speculative content.

---

### Part IV — Meeting Notes

#### Startup Card

```javascript
startupCard('CompanyName', 'Sector / Technology', [
  ['CEO',             'Name  |  Background'],
  ['Core Technology', 'Description'],
  ['Differentiation', 'vs competitors'],
  ['Funding',         'Series X $YM (Year)  |  Lead Investor'],
  ['Investors',       'Investor 1, Investor 2'],
  ['Discussion',      'Key topics from the meeting'],
  ['Next Step',       'Action item'],
  // Portfolio company: add "  ★ Portfolio Company" to sector parameter
])
```

#### VC Card

```javascript
vcCard('Firm Name', 'Contact Name (Title)', [
  'AUM: $Xbn  |  Stage: Growth  |  Focus: AI Infra, Energy',
  'Recent portfolio: Company A, Company B',
  'Discussion: ...',
  'Next step: ...',
])
```

**Web search**: 1 search per item, limited to verifying funding status or VC portfolio. Remove fields where no public information exists. No placeholders.

---

### Part V — Conclusion

Synthesize all collected information. **No additional web search.**

- Investment implications from this conference (by sector)
- Short-term follow-up actions (meeting follow-ups, items for further review)
- Medium-to-long-term monitoring points

---

## Phase 3: Structure Confirmation Gate (MANDATORY)

Before content generation, report the proposed structure to the user and obtain approval.

```
[Report Structure Proposal]

Conference: [Name]
Included parts:
  - Part I  Sector Theme Diagnosis (~2p)
  - Part II Session Deep Dive — [N] sessions (~[N×1.5]p)
  - Part III Sessions Not Attended — [N] sessions (~[N×0.5]p)
  - Part IV Meeting Notes — [N] startups / [N] VCs (~[N×0.5]p)
  - Part V  Conclusion (~1p)
Omitted parts: [None / specify]
Estimated total: ~[N]p
Report language: [English / other]

Proceeding will begin DOCX generation.
```

Proceed to Phase 4 after approval. If revision requested, update and re-report.

---

## Phase 4: DOCX Generation

### Scaffold Loading

```javascript
'use strict';
const {
  h1, h2, h3, h4,
  body, bodyRuns, run, boldRun, blueRun,
  bullet, subBullet,
  space, pageBreak, callout,
  sectionDivider, startupCard, vcCard,
  buildDocument,
} = require('SKILL_DIR/scripts/base_scaffold');
```

### Section Pattern

```javascript
// ── PART II: SESSION DEEP DIVE ────────────────────────────────────────
const sessionBlock = (title, recap, gapFill, insight) => [
  h3(title),
  h4('Session Recap'),
  body(recap),
  ...(gapFill ? [h4('Gap Fill'), body(gapFill)] : []),
  h4('Analytical Insight'),
  body(insight),
  space(80),
];
```

### Metadata Configuration

```javascript
const meta = {
  firmName:   'YOUR_FIRM_NAME',         // replace with your firm name
  confName:   'CONF_NAME',
  confTheme:  'CONF_THEME',
  dates:      'CONF_DATES',
  location:   'CONF_LOCATION',
  attendee:   'ATTENDEE_NAME, TITLE, FIRM_NAME',
  reportDate: 'REPORT_DATE',
  outputPath: 'OUTPUT_PATH',
};
buildDocument(allChildren, meta);
```

**Path principle**: Insert actual paths confirmed in Phase 0. No `/path/to/` placeholders in final script.

### Execution

```bash
# npm install (only if node_modules absent)
cd WORKING_DIR && npm install docx

# Syntax check then run
node --check report_script.js && node report_script.js
```

### Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `TypeError: Cannot read properties of undefined (reading 'cells')` | Undefined element in array (double comma `,,`) | Remove double commas |
| `SyntaxError: Unexpected identifier` | Apostrophe inside single-quote string | Use double quotes: `"O'Brien"` |
| `ShadingType.SOLID` rendering broken | Package version issue | Use `ShadingType.CLEAR` |
| `WidthType.PERCENTAGE` unstable | Table nesting calculation error | Use `WidthType.DXA` |

---

## Phase 5: Verification (Page Count + Factcheck)

### 5-A. Page Count Estimate

```bash
python3 - <<'EOF'
from docx import Document
output_path = 'OUTPUT_PATH'
doc = Document(output_path)
tc = sum(len(p.text) for p in doc.paragraphs)
tt = sum(len(c.text) for t in doc.tables for r in t.rows for c in r.cells)
est = (tc + tt) / 2500
print(f'Estimated pages : {est:.1f}')
print(f'Tables          : {len(doc.tables)}')
print(f'Non-empty paras : {sum(1 for p in doc.paragraphs if p.text.strip())}')
EOF
```

### 5-B. Factcheck Pass (MANDATORY)

After DOCX generation, check the following 4 items:

1. **Person information** — Do speaker names, titles, and affiliations match external sources (LinkedIn, company IR, conference site)? Check for speech-to-text errors that distort names.
2. **Numerical verification** — For all quantitative data: (a) primary source verifiable / (b) attributed to speaker / (c) neither → delete. Especially verify citations from IEA, McKinsey, Goldman.
3. **Banned phrase scan** — Confirm none of the banned phrases from "Output Tone Rules" remain:
   ```bash
   for term in "transcript" "the speaker said" "as an AI" "this report" "groundbreaking" "revolutionary"; do
     python3 -c "from docx import Document; d=Document('OUTPUT_PATH'); print('$term:', sum(1 for p in d.paragraphs if '$term' in p.text.lower()))"
   done
   ```
4. **Source conflicts** — If transcript claims conflict with web search results, verify both sides are presented (do not silently adopt one).

Issues found in Factcheck require body edits followed by re-running 5-A.

---

## Content Writing Principles

1. **No placeholders** — Remove fields where information cannot be verified.
2. **Statistics require sources** — Only publicly verifiable information. Source citation required.
3. **Unbiased tone** — Avoid positive/negative biased adjectives; maintain analytical perspective.
4. **Flag inferences** — In Analytical Insight, distinguish facts from judgments ("appears to suggest," "may indicate," "raises the question of").
5. **Report language** — Follow language confirmed in Phase 0. Default is English.
6. **Section independence** — Each part should be understandable without reading the others.
7. **Portfolio companies** — Add `"  ★ Portfolio Company"` badge to `startupCard` sector parameter.
8. **No simple relay; insight required** — Recap reconstructs the session's logical flow. End each session with 1–2 critical comments (market reality gap, argument weaknesses, investment implications).

---

## Output Tone and Expression Rules (MANDATORY)

### AI Detection Target
- **Target AI Detection Score: 20% or below** (natural, human-written tone)
- Use plain, accessible English. Minimize unnecessary flowery adjectives/adverbs.
- Avoid repeating sentence structures (e.g., don't start every paragraph with "First...", "Second...", "Finally...").
- Mix short sentences with longer analytical ones. Uniform length is an AI signal.

### Banned Phrases (remove all from output)

| Category | Examples to ban |
|---------|----------------|
| Transcript reference | "according to the transcript", "based on the recording", "from the audio" |
| AI generation signals | "this report was generated", "as an AI", "based on the input provided" |
| Meta-narration | "in this section we will discuss", "this report covers", "the following sections present" |
| Abstract clichés | "in today's rapidly evolving landscape", "in an era of unprecedented change", "navigating complexity" |
| Excessive emphasis | "extremely groundbreaking", "revolutionary", "game-changing" |

### Natural Tone Guide
- When relaying speaker remarks, use speaker name + appropriate verb directly (e.g., "Heitbrink argued that...", "Schoiswohl framed AI as..."). Avoid "the speaker said."
- Present figures with context (e.g., "415 TWh — roughly Japan's annual demand").
- Insight sentences: observational over declarative ("the more relevant question is...", "worth flagging because...").

---

## Quick Recap Mode (Optional)

If the user explicitly requests "quick recap," "brief summary," or "2–3 pages," use this lightweight mode instead of the full 5-phase workflow.

### Triggers
- "make it a quick recap"
- "short version, 2–3 pages"
- "summary / brief version"

### Differences

| Item | Full Mode | Quick Recap Mode |
|------|-----------|-----------------|
| Phase 3 Gate | MANDATORY (structure approval) | SKIP (user already specified scope/tone) |
| Structure | Full Parts I–V | Single flow: Sector Takeaway → Session Briefs → Implications |
| Per session | Recap + gap fill + insight (3 blocks, ~1.5p) | Single paragraph ~80–120 words, closing critical line |
| Web search cap | Sessions × 2 | Sessions × 1 (uncertain items only) |
| Length | Naturally determined | 2–3p fixed (user instruction takes priority) |

Quick Recap Mode applies the same "Output Tone and Expression Rules" and "Phase 5 Factcheck."

---

## Wilson Integration

When Wilson receives a conference report request:

1. **Wilson pre-collects**: Extracts or asks the user for conference metadata, file paths, meeting list, and report language.
2. **conf-report Phases 0–2**: File classification → type-based reading → section content generation.
3. **⛳ Gate**: Report the Phase 3 structure proposal to the user and obtain approval. DOCX generation is not permitted without approval.
4. **conf-report Phases 4–5**: Generate and verify DOCX after approval.
5. **Wilson final report**: Deliver results with file link.

---

## Checklist

- [ ] Phase 0-B: File list output → user type assignment confirmed
- [ ] Phase 0-C: Meeting list, portfolio companies confirmed
- [ ] Phase 1: Type-based sequential file reading complete (no bulk loading)
- [ ] Phase 2: Web search hard cap enforced (no section limit exceeded)
- [ ] Phase 3: ⛳ Gate — structure proposal approved (required before DOCX generation; skip in Quick Recap Mode)
- [ ] Phase 4: Actual paths inserted (no placeholders) → `node --check` passed → executed
- [ ] Phase 5-A: Page count estimate verified
- [ ] Phase 5-B: Factcheck (persons / figures / banned phrases / source conflicts) complete
- [ ] Tone check: AI Detection 20% target, banned phrases = 0
- [ ] Output saved to workspace folder and link provided

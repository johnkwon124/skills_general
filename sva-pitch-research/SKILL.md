---
name: sva-pitch-research
description: >
  Generates a deep tech investment research report in your firm's Pitch format using the
  Cubs multi-agent method (Aaron coordinator + Ohtani researcher + Soto writer,
  with bidirectional Q&A). Output is a 10-page+ narrative prose DOCX file.

  Trigger when ANY of the following applies:
  - "write a pitch report", "investment research report", "deep tech report"
  - "Cubs method", "multi-agent report", "Aaron/Ohtani/Soto"
  - Specific deep tech sector report requested (e.g., Space Data Center, Grid Orchestration, AI Infra)
  - Strengthening or revising an existing report ("update the report", "add more insight")
  - Recurring report requests ("two reports per week", "regular deep tech reports")
---

# Deep Tech Pitch Research — Cubs Multi-Agent Skill (v2)

## Overview

This skill automates a senior VC analyst's deep tech investment research workflow.
Three independent agents (Aaron, Ohtani, Soto) collaborate through bidirectional
communication to produce a 7-chapter narrative prose DOCX report grounded in verified data.

**Why three agents?** Separating research (Ohtani) from writing (Soto) allows Soto to
directly question Ohtani about gaps encountered during drafting. This back-and-forth Q&A
loop materially improves data accuracy and analytical depth versus a single-context approach.

**Output language**: English by default. If your firm's internal reports use a different
language, configure in `references/report-config.md`.

---

## Session Path Setup (run once at skill start)

```
SESSION_PATH: /sessions/[current-session-slug]
```

Aaron confirms the current session slug at skill start via `pwd` or `ls /sessions/`,
then substitutes `{SESSION_PATH}` with the real path in all file references below.

**File path conventions:**
| Purpose | Path |
|---------|------|
| Phase 1–3 combined output | `{SESSION_PATH}/phase1-3_[topic_slug].md` |
| Final DOCX | `{SESSION_PATH}/mnt/[output_folder]/Pitch_[TopicCamelCase]_Cubs_[YYYY].docx` |
| DOCX validation script | `{SESSION_PATH}/mnt/.skills/skills/docx/scripts/office/validate.py` |
| Node.js docx package | `{SESSION_PATH}/node_modules/docx` |

Configure `[output_folder]` to match your workspace folder structure.

---

## Agent Role Definitions

### Aaron (Coordinator — the Claude running this conversation)
- Communicates with the user and manages Gate checkpoints
- Runs Ohtani and Soto as independent Agent tool calls and relays results
- Reviews final output quality and reports to the user

### Ohtani (Deep Tech Researcher)
- Focuses on web search and gathering current sources
- Verifies figures, statistics, funding rounds, and technical specs, collecting them in
  **structured table format**
- Provides specific answers to Soto's questions (Phase 3)

### Soto (Pitch Writer)
- Writes in the 7-chapter Pitch structure in narrative prose
- After reviewing Ohtani's research, generates **5 clarification questions** for Ohtani (Phase 2)
- Integrates Q&A answers to complete the final DOCX (Phase 4)

---

## Workflow: 5-Stage Cubs Process

```
Gate 1 → Phase 1 → Gate 2 → Phase 2 → Phase 3 → [new chat] → Phase 4 → Phase 5
```

---

### ⛳ Gate 1 — Topic Selection and Briefing Approval (user confirmation required)

If no topic is specified, Aaron proposes 3–5 candidates based on:
- Sectors not already covered in existing research
- Areas where investable startups exist (preferably US-based)
- Recent trends from relevant sector conferences and forums

After topic is confirmed, Aaron writes a **research briefing** (1–2 paragraphs: why this topic,
3 core questions) and receives user approval.

**⛔ Do not start Phase 1 before clearing Gate 1.**

---

### Phase 1 — Ohtani Initial Research

Run an independent agent to collect the following.

**Research scope:**
- Technology status and TRL (Technology Readiness Level)
- Key startups (especially US-based, top-tier VC-backed)
- Funding rounds and investor profiles
- Market size figures (source citation required for all)
- Competitive landscape (domestic and international players)
- Technical constraints and open debates

**Quality standards (strictly enforced):**
- All figures recorded with source (company filing, press report, market research report)
- Estimates clearly marked as such
- Industry participant's own claims tagged as "(Company-stated)"

**Phase 1 output format (structured tables — required):**

```markdown
# Phase 1 Research: [Topic]
Date: YYYY-MM-DD

## Sector Overview
| Item | Content | Source | Tag |
|------|---------|--------|-----|
| Market size TAM | $X B (YYYY) | [Institution] | Verified / Estimated |
| Market size SAM | $X B | [Institution] | Verified / Estimated |
| Core technology TRL | TRL N | [Source] | Verified |
| Key growth drivers | ... | ... | ... |

## Company Profiles

### [Company A]
| Item | Value | Source | Tag |
|------|-------|--------|-----|
| Founded | YYYY | Crunchbase | Verified |
| HQ | City, State | Official website | Verified |
| Latest funding round | $XM Series N | [Publication] YYYY.MM | Verified |
| Total funding | $XM | Crunchbase | Verified / Estimated |
| Key investors | [VC name] | [Source] | Verified |
| Core technology | ... | [Source] | Verified / Company-stated |
| TRL level | TRL N | [Source] | Verified / Estimated |
| Customers / Partners | ... | [Source] | Verified / Company-stated |
| Differentiation claimed | ... | Company IR | Company-stated |
| Differentiation verified | ... | [Third-party source] | Verified / Unverifiable |

### [Company B]
...

## Competitive Landscape
| Company | Country | Approach | Funding | Source |
|---------|---------|----------|---------|--------|
| ... | ... | ... | ... | ... |

## Technical Debates and Constraints
| Issue | Skeptic view | Counterargument | Source |
|-------|-------------|-----------------|--------|
| ... | ... | ... | ... |
```

---

### ⛳ Gate 2 — Outline Approval (user confirmation required) ← run immediately after Phase 1

**Immediately after Phase 1 completes**, before Soto begins writing, Aaron presents the user
with a **7-chapter outline draft**:
- Core argument for each chapter (1–2 sentences)
- List of key data points to use (selected from Phase 1 tables)
- 3 anticipated differentiation points

**⛔ Do not start Phase 2 before Gate 2 is approved.**

> **Why this order matters**: If the outline direction needs adjustment, running Gate 2 here
> prevents having to redo the entire Q&A loop (Phases 2–3).

---

### Phase 2 — Soto Draft Questions

Having read Ohtani's research, Soto writes **5 additional questions** it needs answered
to write the 7-chapter structure.

**Question selection criteria (strict):**
Only ask about data gaps that directly affect the investment thesis.

Good questions:
- "Company X's actual GPU compute capacity may differ from marketing materials — is there
  an independently verified figure?"
- "Can you confirm whether the defense revenue share figure comes from an industry report
  or company announcement?"

Bad questions (do not generate):
- Questions already answerable from Phase 1 tables
- Vague questions like "Can you tell me more about X?"
- Questions about information that is definitionally non-public

---

### Phase 3 — Ohtani Q&A Answers

Ohtani answers Soto's 5 questions via web search with specific responses.
Questions that cannot be answered are clearly marked: "No public information — [reason]."

**After Phase 3 — Cross-Phase Save Protocol:**

Aaron combines the Phase 1 structured tables + Phase 2's 5 questions + Phase 3's 5 answers
into a single file:

```
Save path: {SESSION_PATH}/phase1-3_[topic_slug].md
```

After saving, Aaron gives the user the following instructions:

```
✅ Phases 1–3 complete. phase1-3_[topic].md saved.

📋 Next step (Phase 4 — DOCX writing):
1. Upload the file above to Project Knowledge
2. Open a new chat
3. Enter: "Pitch Research Phase 4 start: [topic]. Reference phase1-3_[topic].md."

Phase 4 DOCX writing begins in the new chat, free of context noise.
```

**⛔ Phase 4 must start in a new chat.**

---

### Phase 4 — Soto Final DOCX Writing (new chat)

Reference `phase1-3_[topic].md` from Project Knowledge to generate the final DOCX.

Before generating, read these reference files for DOCX format spec and 7-chapter structure:
- `references/chapter-structure.md` — 7-chapter Pitch structure details
- `references/docx-generation.md` — Node.js docx package usage and format rules

Save path: `{SESSION_PATH}/mnt/[output_folder]/Pitch_[TopicCamelCase]_Cubs_[YYYY].docx`

---

### Phase 5 — Verification and Report

```bash
python3 {SESSION_PATH}/mnt/.skills/skills/docx/scripts/office/validate.py \
  [DOCX path]
```

After validation passes, Aaron reports:
- File size and paragraph count
- Key insight summary for each of the 7 chapters (2–3 lines each)
- User feedback incorporated at Gates

---

## Analysis Quality Standards (Soto must follow)

These principles reflect the core writing standards for this research format.

### Absolute prohibitions
- Presenting company's own figures as fact without verification
- Biased adjectives: "innovative", "groundbreaking", "game-changer"
- Market size figures without source
- Using another deal's outcome as evidence for a different business model

### Required inclusions
- Three-layer structure for company claims:
  > "Company X claims Y. However, given [source limitation / alternative interpretation]..."
- "(Company-stated)" and "(unverified)" tags on self-reported figures
- TRL level stated explicitly (TRL 1–9 scale)
- Due diligence checkpoints from an investor perspective

### Style rules
- Full narrative prose (no bullet points; minimize tables)
- Minimum 800 characters per chapter
- Arabic numerals; units clearly stated
- Natural, human-quality writing; AI detection target: 10% or below

---

## Agent Execution Pattern (for Aaron)

### Running Ohtani
```
Use Agent tool:
- description: "Ohtani: [topic] research"
- prompt: [see ohtani-prompt template in references/agent-prompts.md]
```

### Running Soto
```
Use Agent tool:
- description: "Soto: [topic] writing"
- prompt: [see soto-prompt template in references/agent-prompts.md]
```

See `references/agent-prompts.md` for full prompt templates.

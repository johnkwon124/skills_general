---
name: meeting-notes
description: >
  Automatically generates meeting notes with analytical insight for all meeting types.
  Four modes: Startup First Meeting (MODE A), Startup Follow-up Meeting (MODE B),
  VC/LP Meeting (MODE C), and Conference Session Summary (MODE D).
  Auto-detects the appropriate mode and generates an optimized DOCX meeting note.

  ALWAYS trigger when ANY of these apply:
  - "write meeting notes for X" / "summarize meeting notes" / "create meeting notes"
  - User attaches a transcript or meeting memo and asks for a summary
  - Startup, VC, or conference session summary/record requested
  - "follow-up meeting" or "second meeting" context where a prior meeting exists
  - Conference presentation content summary requested

  MODE A (Startup First Meeting) recommended input format:
  "Company Name / Sector or Core Business"
  Example: "Arbor Energy / Grid Battery Storage / AI-optimized dispatch"
---

# Meeting Notes

## Overview

A meeting note generation workflow covering all meeting types for VC professionals.
The output is a letter-sized 2–3 page DOCX with analysis and insight, not a simple information relay.

**Common principles (all modes)**:
- Length: 2–3 letter pages (approx. 600–900 words)
- Language: English by default. Plain, natural sentences. Target AI Detection Score ≤ 20%
- Estimated/speculative data → delete immediately. Figures must include source citation
- Generalizations ("X is a growing market") → delete. Replace with specific figures/examples or omit
- Each section should end with a VC-perspective analytical insight, not just factual relay
- No investment opinion or Next Step recommendation (meeting notes are records + analysis documents)
- **DOCX format must follow the "Output Format Spec — DOCX" section below**

---

## Step 0 — Mode Auto-Detection

Upon receiving a meeting notes request, auto-classify using the criteria below.

```
Transcript/memo attached?
        │
        NO → Company name only → MODE A (Startup First Meeting)
        │
       YES → Determine counterparty type
              │
              ├─ Startup + no prior meeting  → MODE A (first meeting, transcript-based)
              ├─ Startup + prior meeting exists → MODE B (Follow-up Meeting)
              ├─ VC / LP / Co-investor      → MODE C (VC Meeting)
              └─ Conference / panel / session → MODE D (Conference Session)
```

**After detection, confirm with the user in this format. Proceed only after explicit approval ("yes", "correct", "go ahead"):**

```
"Proceeding as MODE [X] — [Company/Counterparty] [Meeting Type].
 Confirm to proceed."
```

**After mode is confirmed: reference only that MODE section. Ignore instructions in all other MODE sections.**

---

## MODE A — Startup First Meeting

**Trigger**: Startup meeting notes requested + no transcript / first meeting
**Agents**:
- Ohtani (Research): sector-mapped 8–12 target source research
- Soto (Writing): DOCX generation
- Wilson: review and final sign-off

### Ohtani Research Guidelines

**Input format**: User provides `"Company Name / Sector or Core Business"`.
Example: `"Arbor Energy / Grid Battery Storage"`, `"Nexus Labs / Electrochemical 3D Printing / thermal management"`

**Source selection**: Based on sector tag, select 8–12 sources from the mapping table below.
Quality and category fit matter more than quantity. Do not explore low-signal sources.

**Sector Source Mapping Table**:

```
[Common — required for all sectors]
  ① Company official site (product specs, About, Press)
  ② Crunchbase (funding stage, investors, headcount)
  ③ LinkedIn (founder/CTO background, team size)
  ④ Recent press release / funding announcement (TechCrunch, PR Newswire, etc.)

[Power Electronics / SST / Power Infrastructure]
  ⑤ IEEE Spectrum or Power Electronics News
  ⑥ Wood Mackenzie or BloombergNEF (market data)
  ⑦ Google Patents (1–2 key patents)
  ⑧ Competitor official sites (2–3 leading competitors in space)

[AI Infra / Data Center / Cooling]
  ⑤ SemiAnalysis or The Register
  ⑥ Uptime Institute or Data Center Frontier
  ⑦ Competitor official sites (2–3)
  ⑧ Key customer/partnership announcements (hyperscaler blogs, etc.)

[Climate / Energy Storage / Renewable]
  ⑤ BloombergNEF or Wood Mackenzie
  ⑥ DOE Grant DB (ARPA-E, Loan Programs Office)
  ⑦ IEA or RMI related reports
  ⑧ Competitor official sites (2–3)

[Biotech / HealthTech / MedDevice]
  ⑤ PubMed (1–2 key papers)
  ⑥ FDA 510(k) or ClinicalTrials.gov (if applicable)
  ⑦ FierceBiotech or MedTechDive
  ⑧ Competitor official sites (2)

[Manufacturing / Advanced Materials / Hardware]
  ⑤ Google Patents (key patents)
  ⑥ Industry trade publications (SME, Manufacturing Engineering, etc.)
  ⑦ Supply chain partner announcements
  ⑧ Competitor official sites (2–3)

[Sector unspecified or other]
  → Common 4 sources + 2–3 competitors + Google Patents (if applicable)
```

**Section-by-section collection items**:

```
Company Overview     → founding year, HQ, headcount, recent filings, funding stage
Key Technology       → core product spec figures, technical differentiators, patents, tech announcements
Market Focus         → target customers, applications, geographic strategy
Competitive Position → 3–5 direct competitors + each company's public tech/funding specs
Supply Chain & Mfg   → cost structure, manufacturing method/facility, production timeline
Firm Meeting         → record only if prior meeting with your firm exists — facts only, no speculation
Key Challenges       → key technical/business problems the company has explicitly addressed
Commercial Timeline  → pilot, production, funding milestones (by year)
```

**Collection rules**:
- Figures/statistics → cite source URL or report name
- Unverifiable items → tag [UNVERIFIED] and pass to Soto (delete decision is Soto/Wilson's)
- If a sector-mapped source yields no useful information, skip it and substitute another. Do not add low-quality sources just to reach the 8–12 count.

### Soto Writing Guidelines — 7-Section Structure

Follow the structure below exactly. Section order, bullet types (A/B/C), and prose paragraph placement are all defined here.

```
[Title: Company Name Meeting Summary]
[No horizontal rule — MODE A starts Company Overview paragraph directly below title]

1. Company Overview  [prose paragraph — no bullets]
   - Bold company name on first mention (e.g., "Arbor Energy is a grid battery storage...")
   - Content: company description + headcount + HQ + most recent funding stage + next round timing (if public)
   - 2–4 sentences, prose format. No bullets.
   [VC insight] Risk/opportunity from current growth stage and investment timing (evidence-based judgment)

2. Key Technology Highlights  [two subsections]

   Product Specifications  [subsection header — Bold]
   → Bullet Type A: "• **Bold Label**: description"
   Example:
       • **Power Rating**: 10MW in 10-foot ISO container
       • **Efficiency**: 98.5%+
       • **Voltage Range**: 6.6–34.5kV input, 800V–1500V DC output
   3–5 key technical specs.

   Technical Differentiators  [subsection header — Bold]
   → Bullet Type B: "• **Bold keyword** description [no colon]"
   Example:
       • **Resonant topology** enabling high-frequency operation (70kHz vs competitors' 20–30kHz)
       • **Liquid cooling system** with redundancy (no external auxiliary power needed)
   3–5 differentiation points vs. competitors.
   [VC insight] Technology maturity (can reference TRL scale) and remaining commercialization hurdles

3. Market Focus
   → Bullet Type A: "• **Bold Label**: content"
   Must include: Primary target / Application / Geographic strategy
   [VC insight] Market timing and technology adoption stage analysis

4. Competitive Positioning  [lead sentence + plain bullets]
   - First line: competitive position summary in one sentence
     Example: "Leading the upper tier alongside two well-funded competitors, with key advantages in:"
   - Following bullets: Type C (plain). 2–4 key competitive advantages.
   [VC insight] Assessment of actual competitive position within the landscape

5. Supply Chain & Manufacturing  [hardware/deep tech companies only]
   → Bullet Type A
   Must include: Cost drivers / Manufacturing / Timeline
   [VC insight] Manufacturing readiness and scale risk

6. Key Technical Challenges Solved  (or Key Challenges Addressed)
   → Bullet Type A: "• **Bold challenge area**: resolution approach"
   Include only what the company has explicitly claimed to have solved.

7. Commercial Timeline
   → Bullet Type A: "• **Year**: milestone"
   Example:
       • **2026**: Customer pilots and product refinement
       • **2027**: Production scaling and Series B fundraising

[Firm Meeting — include only if applicable]
   - Include only if a prior meeting with your firm (or a BU partner) is on record.
   - State factual meeting details only. No speculative collaboration or partnership assertions.
   - Example: "Company met with [Firm] BU leadership on [Date]."
```

### Wilson Content Pre-Approval (before DOCX generation)

Before Soto writes any DOCX code, submit the following outline to Wilson.
Wilson approves before DOCX generation begins. This step catches section omissions, factual errors, and format issues in advance.

```
[Wilson pre-approval request format]
"Section structure confirmation:
 1. Company Overview — [one-line summary]
 2. Key Technology — [number of spec items, number of differentiators]
 3. Market Focus — [primary target in one line]
 4. Competitive Positioning — [number of competitors]
 5. Supply Chain & Mfg — [included / not applicable]
 6. Key Challenges — [number of items]
 7. Commercial Timeline — [year range]
 Firm Meeting included: [yes / no]
 Approve to begin DOCX generation."
```

### Soto DOCX Code — Defensive Code Required

```python
# 1. Package check and install
import subprocess, sys
try:
    from docx import Document
except ImportError:
    subprocess.check_call([sys.executable, "-m", "pip", "install",
                           "python-docx", "--break-system-packages", "-q"])
    from docx import Document

# 2. Output path check
import os
output_dir = "/mnt/user-data/outputs"
os.makedirs(output_dir, exist_ok=True)

# 3. Font fallback (Calibri → Arial → DejaVu Sans)
def safe_font(run, name="Calibri"):
    for font_name in [name, "Arial", "DejaVu Sans"]:
        try:
            run.font.name = font_name
            break
        except Exception:
            continue

# 4. Full execution try/except wrapper
try:
    # ... DOCX generation code ...
    doc.save(os.path.join(output_dir, filename))
    print(f"DOCX saved: {filename}")
except Exception as e:
    print(f"DOCX generation failed: {e}")
    raise
```

- Sentences should be concise and direct. Subject + verb focus, minimal modifiers.
- VC insight paragraphs: written in the same style as body text (Calibri Regular 11pt, no indent), 1–2 sentences, prose format. Non-declarative tone ("suggests", "indicates", "raises the question of"). No labels.
- **AI Detection ≤ 20% — specific rules**:
  - Vary sentence openings. Avoid repeating "The company...", "This technology..."
  - Mix short sentences (8–12 words) with longer ones (20–30 words). Avoid monotonous rhythm.
  - Ban AI marker words: "notably", "importantly", "significantly", "furthermore", "moreover"
  - Avoid passive voice overuse. Allow where natural.
  - Minimize excessive modifiers: "highly innovative", "cutting-edge", "revolutionary"
  - Avoid consecutive -ing participial phrases: "enabling..., allowing..., providing..."
  - Avoid excessive parallel structure: no more than 3 consecutive "X, Y, and Z" bullet patterns

### Wilson Review Criteria (MODE A)

- Remove all [UNVERIFIED] items or replace with "(unconfirmed)"
- Verify VC insight paragraphs are evidence-based, not unsupported speculation
- If length exceeds 3 pages, instruct Soto to compress each VC insight to 1 sentence
- Verify bullet types follow section rules (Product Spec → Type A / Tech Diff → Type B / rest → Type A or C)
- Verify Company Overview is a prose paragraph (no bullets)
- If Firm Meeting included, verify facts only (no collaboration speculation)

---

## MODE B — Startup Follow-up Meeting

**Trigger**: Startup transcript attached + prior meeting on record

**Principle**: Transcript content > web research. Web search only to verify figures/claims from transcript.

### Ohtani Research Guidelines

```
Step 1 — Full transcript read
  Extract key statements (CEO/CTO direct quotes prioritized)
  Flag list:
    - Figures unclear (e.g., "approximately $X" → exact figure unknown)
    - Claims requiring verification (e.g., "only player in market" → check for competitors)
    - Missing context (e.g., customer name mentioned but deal size/structure unclear)
    - Urgent/critical items (M&A, LOI, deadlines, runway pressure) → candidate for red text

Step 2 — Web search limited to flagged items
  Verified → merge into body + tag source
  Unverified → mark "(unconfirmed per transcript)"
```

### Soto Writing Guidelines — Topic-Based Structure

Title format: `[Company Name] — Meeting Update`
Horizontal rule below title. No Company Overview section — begin directly with topic sections.
Section structure is flexible based on transcript topics (below is a typical example).

```
[Title: Company Name — Meeting Update]
[Horizontal Rule — directly below title]

Sections (based on transcript topics — typical structure):

1. Product Development / Product Update  [if applicable]
   → Bullet Type C (plain)
   - Development progress, prototype status, next steps

2. Product Roadmap  [if applicable]
   → Bullet Type C

3. Software / Platform  [if applicable — may include product name]
   → Bullet Type C

4. Customer Pipeline / Go-to-Market  [if applicable]
   → Bullet Type C

5. Commercialization Timeline  [if applicable]
   → Bullet Type C. Year-based.
   Example: • 2026: get 2–3 beta units into real pilot deployments

6. Funding  [if applicable]
   → Bullet Type C
   - Burn rate, runway, funding structure, next round plans
   - M&A LOI, acquisition negotiations, or imminent deadlines → red text

[No closing paragraph]
```

**Red text criteria**:
- M&A LOI or active acquisition negotiations
- Investment round deadline explicitly stated
- Imminent runway concern affecting company viability
- Items the counterparty specifically emphasized as urgent
- Use sparingly: maximum 1–3 bullets per document

**Apply the same defensive code pattern from MODE A "Soto DOCX Code" section.**

### Wilson Review Criteria (MODE B)

- Horizontal rule present below title
- No Company Overview section — topics begin directly
- No closing paragraph
- Red text applied only to qualifying items (max 3 per document)
- No unauthorized additions beyond transcript content

---

## MODE C — VC / LP Meeting

**Trigger**: Transcript attached + counterparty is VC, LP, Co-investor, or Family Office

**Principle**: Relationship context, collaboration signals, and deal flow information take priority over company introductions.

### Ohtani Research Guidelines

```
Transcript parsing first. Web search only to supplement:
  - Target fund AUM, vintage, public portfolio info
  - Contact's LinkedIn public background
  - Recent investment history (past 12 months, public deals only)
```

### Soto Writing Guidelines — 4-Section Structure

```
[Title: Fund/Person Name — Meeting Notes]
[Horizontal Rule]

1. Meeting Context
   - Counterparty: fund name, contact name + title, AUM (if public), primary sectors
   - Meeting occasion or intro source

2. Their Investment Focus
   - Current sector focus and stage (based on transcript statements)
   - Recent themes of interest or technology trends they're watching
   [VC insight] Overlap or complementary relationship between counterparty thesis and your portfolio

3. Collaboration Signals
   - Co-investment interests or specific deals mentioned
   - Deal sourcing channel sharing interest, portfolio introduction potential
   [VC insight] Current relationship temperature: information exchange level, or developing into real collaboration?

4. Follow-up Items
   - Agreed next actions (material sharing, re-meeting, introductions)
   - Items requested by counterparty or items your firm needs to follow up on
```

---

## MODE D — Conference Session

**Trigger**: Conference session script or listening notes attached

**Principle**: Speaker's key claims + Q&A insights only. Generic/promotional statements automatically excluded. Shortest format.

### Ohtani Research Guidelines

```
Transcript/notes parsing:
  - Extract only statements that include figures or specific examples (skip generalities)
  - From Q&A: extract only items where speaker gave a concrete answer

Web search (limited):
  - Verify speaker identity and affiliation
  - Verify cited reports/figures (TechCrunch, BloombergNEF, IEA, etc.)
```

### Soto Writing Guidelines — 3-Section Structure (1–2 letter pages)

```
[Title: Conference Name — Session Title]
[Horizontal Rule]

1. Session Overview
   - Conference name, session title, speaker name + affiliation, date

2. Key Points
   - Figure/example-based key claims (max 5–7 bullets)
   - Each bullet must include specific data or a concrete example
   [VC insight] 1–2 most noteworthy signals from a VC perspective

3. Notable Q&A
   - Audience questions + speaker answers with genuine insight (1–3 items only)
   - Also record questions the speaker avoided or answered vaguely — these have signal value
```

---

## Output Format Spec — DOCX

**This format spec is derived directly from real meeting note templates.
Must be followed when generating DOCX. The old navy/blue color header approach is discontinued.**

### Document Title
- Text: "[Company Name] Meeting Summary" (MODE A) or "[Company Name] — Meeting Update" (MODE B/C/D)
- Font: **Calibri Bold, 13pt, black (#000000)**
- Alignment: left
- Spacing: 6pt below title

### Horizontal Rule
- **MODE A**: No rule. Begin Company Overview paragraph directly below title.
- **MODE B / C / D**: Insert thin solid line directly below title.
  - Style: 0.5pt, black, full width

### Section Headers
- Font: **Calibri Bold, 11pt, black**
- Color: **none** (no navy/blue/color)
- Background: none
- Bottom border: none
- Spacing: 12pt above, 4pt below

### Subsection Headers (e.g., Product Specifications / Technical Differentiators in MODE A)
- Font: **Calibri Bold, 11pt, black** (same as section headers)
- Spacing: 6pt above, 2pt below

### Body Text / Prose Paragraphs
- Font: Calibri Regular, 11pt, black
- Line spacing: 1.15
- Spacing after paragraph: 6pt

### Bullet Points — 3 Types

**Type A — Spec Bullets** (Product Specifications, Market Focus, Supply Chain, Partnership, Timeline sections):
```
• **Bold Label**: description text
Example:
    • **Power Rating**: 10MW in 10-foot ISO container (industry-leading density)
    • **Primary target**: Hyperscalers (Google, Amazon, Microsoft, Meta)
    • **2026**: Customer pilots and product refinement
```

**Type B — Differentiator Bullets** (Technical Differentiators section only):
```
• **Bold keyword** description text  [no colon]
Example:
    • **Resonant topology** enabling high-frequency operation (70kHz vs competitors' 20–30kHz)
    • **Liquid cooling system** with redundancy (no external auxiliary power needed)
    • **Battery backup unit (BBU)** integration for 2–5 minute UPS replacement
```

**Type C — Plain Bullets** (all of MODE B, Competitive Positioning sub-items, general narrative):
```
• plain text description
Example:
    • Bench-level prototype is complete and currently under testing
    • No plans to build their own factory — scaling will happen through partnerships
    • Team of 100 — reportedly the largest dedicated team in this space globally
```

**Common bullet specs**:
- Symbol: round bullet (•)
- Indent: 0.25 inch (left indent)
- Font: Calibri 11pt
- Line spacing within bullet: single
- Spacing between bullets: 2pt (space after)

### Emphasis Text — Red (#FF0000)
- **MODE B only**. Do not use in other modes.
- Applies to:
  - M&A LOI or active acquisition negotiations
  - Investment round deadline explicitly stated
  - Runway pressure directly affecting company viability
- Apply to entire bullet text
- Use sparingly: max 1–3 items per document

### Company Name Inline Bold (MODE A only)
- Bold the company name on first appearance in Company Overview prose paragraph
- Example: "**Arbor Energy** is a grid battery storage startup targeting AI data centers..."

### VC Insight Paragraph
- Positioned as last item within its section
- Font: Calibri Regular, 11pt, black — same style as body text
- No indent. No "[Analysis]" or similar label. No bullet.
- 1–2 sentences, prose. Judgmental but non-declarative.

### Page Setup
- Paper: Letter (8.5 × 11 inches)
- Margins: 1 inch on all sides
- Default font: Calibri 11pt

### DOCX Generation Reference
Consult the `docx` skill's SKILL.md before generating.
When using python-docx, implement the above specs directly in code.

---

## Wilson Fact-Check Pass (web search based)

Immediately after DOCX generation, Wilson cross-verifies key factual items via web search.
This step is especially important when relying heavily on Otter.ai transcripts — to prevent
speech-to-text errors from carrying over into the DOCX.

### Why It's Needed

Otter.ai exhibits the following systematic error patterns:
- **VC fund names**: "Hulu Ventures" → actual "Ulu Ventures" (seed VC, Palo Alto)
- **Award names**: "Judge's Choice" → misheard as something different
- **Time expressions**: "thirty-second" → transcribed as "32nd" or "32-second"
- **Person titles**: partial omissions (e.g., co-affiliation dropped)
- **Conference names**: acronyms/proper nouns misrecognized

### Fact-Check Priority Order

Focus on uncertain items (not necessary to search all):

1. **Person titles** — cross-check via LinkedIn, institutional pages, school/lab profiles
2. **VC fund names** — verify via Crunchbase, fund official website (watch for similar-sounding names)
3. **Conference names / awards** — verify via official event pages, organizer announcements
4. **Technical figures** — compare with published papers, press releases, company website specs
5. **Team member names** — cross-check via paper co-authors, LinkedIn, company About page

### Error Handling

| Situation | Action |
|-----------|--------|
| Correct information verified via search | Update DOCX with accurate information |
| Cannot verify (no information available) | Keep [UNVERIFIED] tag, report to user |
| Search result conflicts with transcript | Apply web source, report correction |

### Otter.ai Numeric Error Note

When ordinal numbers appear in time/duration contexts, treat with suspicion.
- "32-second", "32nd" → may actually be "thirty-second (30 seconds)" in speech
- Use context (medical device, test duration, reaction speed) to judge if the figure is plausible
- If in doubt, verify against actual product/technology specs via web search

### Wilson Review Report Format

After Fact-Check Pass, report to the user in this format:

```
[Fact-Check Results]
Corrections: X items
- (item): "(original)" → "(corrected)" | Source: [URL]
Unverified: Y items (marked [UNVERIFIED])
No issues: Z items
```

---

## Wilson Final Review Checklist

```
All modes (common):
[ ] No estimated/fabricated data (figures cite source or are removed)
[ ] [UNVERIFIED] items resolved
[ ] VC insight paragraphs are evidence-based judgments (no speculation, no labels, body style)
[ ] No generic bullets ("X is growing" type removed)
[ ] No investment opinion / Next Step
[ ] Length within 2–3 letter pages (MODE D: 1–2 pages)
[ ] AI Detection target met (all modes ≤ 20%)
[ ] Title: Calibri Bold 13pt, black
[ ] Section headers: Calibri Bold 11pt, black (no navy/color)
[ ] Bullet font: Calibri Regular 11pt
[ ] Fact-Check Pass complete and results reported to user

MODE A specific:
[ ] Wilson content pre-approval step complete before DOCX generation
[ ] No horizontal rule below title
[ ] Company Overview → prose paragraph (no bullets), company name Bold
[ ] Key Tech → Product Spec (Type A) + Technical Diff (Type B) distinguished
[ ] Firm Meeting (if included): facts only, no collaboration speculation
[ ] No closing paragraph
[ ] No red text

MODE B specific:
[ ] Horizontal rule below title
[ ] No Company Overview — topic sections begin directly
[ ] Bullet Type C (plain) as default
[ ] Red text: applied only to qualifying items (max 3 per document)
[ ] No closing paragraph

MODE C / D:
[ ] Horizontal rule below title
```

Review passed: deliver DOCX to user.
Items not passed: feedback to Soto for rework (maximum 2 loops).

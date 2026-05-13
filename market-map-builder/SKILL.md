---
name: market-map-builder
description: >
  Dedicated VC sector research skill. Takes a target investment sector (e.g., Agentic AI, Data Center, Energy, AI Infra, Climate) and performs web research → company discovery → 3-axis evaluation (investors/founders/partnerships) → Excel market map + DOCX sector report.

  Use this skill when the user says:
  - "sector research", "build a market map", "sector overview", "deal sourcing"
  - "find startups in X space", "sector investment report", "sourcing"
  - requesting both Excel + DOCX report outputs simultaneously
  - asking "what companies are in X space?" or "what startups should I look at in X?"
---

# Market Map Builder Skill

Performs deal sourcing research for an investment sector and generates two outputs:
1. **Excel Market Map** — full company list + 3-axis scores + competitive landscape by layer + A-grade shortlist
2. **DOCX Sector Report** — 8-section analyst report for investors

---

## Workflow Overview

```
Step 0: Define sector and research scope
Step 1: Web research — company discovery (target: 6–12 companies)
Step 2: Apply 3-axis evaluation framework → derive grades
Step 3: Generate Excel market map (openpyxl)
Step 4: Generate DOCX sector report (Node.js docx)
Step 5: Review outputs and deliver to user
```

---

## Step 0: Sector Definition

Before starting research, confirm the following:
- **Sector name**: e.g., "Solid State Transformer", "Agentic AI Infra", "Grid Orchestration"
- **Sub-theme**: which layer/position within the sector to focus on
- **Investment stage**: stage restriction if any (default: stage-agnostic)
- **Geography**: regional focus if any (default: global)

If the user provides only a sector name, proceed directly to research. If additional inputs would meaningfully improve output quality, ask as a follow-up after delivering results.

---

## Step 1: Web Research

### Objective
Using only public information, discover 6–12 startups in the sector. Data integrity is the top priority — do not include unverified figures.

### Research Sequence
1. Use `web_search` to collect recent news and funding announcements in the sector
2. Search individually for each company's investors, founder background, and partnerships
3. Include exited companies (acquired/IPO) but mark them separately

### Data Points to Collect (per company)
| Field | Description |
|-------|-------------|
| Company name, founded year, HQ | Basic info |
| Stage / Total raised | Based on most recent round |
| Core product / differentiator | Technical approach |
| Key investors (fund name + tier) | Based on most recent round |
| Founder background | Previous employer, education, domain experience |
| Official partnerships | Include only announced contracts/collaborations |
| Value chain position | Which layer of the sector this company occupies |

---

## Step 2: 3-Axis Evaluation Framework

### Evaluation Principles
- Base assessments only on publicly confirmed facts. Mark estimates in Notes.
- Apply the same rubric regardless of stage.
- If information is insufficient, assign a lower score and note the reason.

### Axis 1 — Investor Profile (1–5)

Top 30 Tier Reference:
- **Tier 1**: Sequoia, a16z, Benchmark, Accel, Kleiner Perkins, Greylock, Lightspeed, General Catalyst, NEA, Bessemer
- **Tier 2**: Founders Fund, Khosla, Coatue, Tiger Global, Insight, IVP, Battery, Index, Redpoint, Spark
- **Tier 3**: Lux Capital, 8VC, USV, First Round, True Ventures, Menlo, Radical Ventures, Initialized, Obvious, Lowercarbon

| Score | Criteria |
|-------|----------|
| 5 | 2+ Tier 1 funds, or Tier 1 + major strategic investor (e.g., NVIDIA NVentures) |
| 4 | Tier 1 ×1 + Tier 2 or higher, or 2+ from Top 30 |
| 3 | Tier 2 focused, or well-known Climate/Deep Tech specialist VC outside Top 30 |
| 2 | Recognized institution but outside Top 30, or PE/CVC-heavy |
| 1 | Unverified / Angel / small seed fund only |

### Axis 2 — Founder Background (1–5)

| Score | Criteria |
|-------|----------|
| 5 | FAANG/OpenAI/leading AI lab + serial founder, or 15+ years domain expertise |
| 4 | Top-tier big tech background or relevant domain PhD + startup experience |
| 3 | Solid industry track record, domain experience present, first-time founder |
| 2 | Technical background but indirect connection to the sector |
| 1 | Insufficient founder information / unverifiable domain experience |

### Axis 3 — Partnerships (1–5)

| Score | Criteria |
|-------|----------|
| 5 | Official contract with Microsoft/Google/NVIDIA/AWS/major utility (publicly announced) |
| 4 | Official collaboration with Fortune 500 or major industry platform |
| 3 | Collaboration with recognized startup/institution (verifiable form) |
| 2 | Partnership exists but limited scale/credibility, or pilot stage |
| 1 | No publicly disclosed partnerships |

### Grade Thresholds

| Total | Grade | Action |
|-------|-------|--------|
| 13–15 | **A** | Prioritize for outreach/meeting |
| 9–12 | **B** | Add to monitoring queue, re-evaluate at next round/milestone |
| 5–8 | **C** | Hold, maintain sector tracking only |
| ≤4 | **Pass** | Exclude |
| — | **EXITED** | Acquired/IPO complete (include for benchmarking) |

---

## Step 3: Excel Market Map Generation

### Tech Stack
- **Language**: Python + `openpyxl`
- **Install**: `pip install openpyxl --break-system-packages`

### File Structure (3 sheets)

**Sheet 1 — Long List (Full Market Map)**
- Title block (sector name, date, data source disclosure)
- Headers: Company / Founded / HQ / Stage / Total Funding / Value Chain Position / Customer Segment / Business Model / Core Product+Differentiator / Key Investors / Investor Score / Founder Background / Founder Score / Key Partnerships / Partnership Score / Total Score / Rating / Evaluation Notes
- Row colors by grade: A=green(DCFCE7), B=yellow(FEF3C7), C=gray(F3F4F6), Pass=red(FEE2E2), EXITED=mid-gray

**Sheet 2 — Competitive Landscape**
- Classify sector by value chain layer
- Map players per layer (company name, core approach, key customers, competitive dynamics, total funding, key investors)
- "Key Structural Observations" block at bottom (5–6 investor-perspective items)

**Sheet 3 — A-Grade Shortlist**
- Only companies graded A (≥13 points)
- Columns: Company / Score / Stage+Funding / Why A-Grade / Key Risks / Recommended Next Step
- B-grade companies noted in a bottom row as monitoring queue

### Style Guide
```python
DARK_NAVY   = "1F2D3D"  # title background
MID_BLUE    = "2E4A6B"  # subtitle background
ACCENT_BLUE = "3B82F6"  # column header background
LIGHT_GRAY  = "F1F5F9"  # alternating row background
```
- Font: Arial throughout
- Hide grid lines (`showGridLines = False`)
- Freeze panes at row 4
- Set column widths individually to fit content
- Apply thin_border to all cells

---

## Step 4: DOCX Sector Report Generation

### Tech Stack
- **Language**: Node.js + `docx` npm package
- **Install**: `npm install docx`

### Report Structure (8 sections)

```
[Cover Page]
  Sector name (large text) + "AI Infrastructure × Energy — Investor Overview"
  Date | Internal Research | Confidential
  List of companies covered

1. Why This Sector — Structural Problem
   - Evidence for supply-demand imbalance (based on IEA/NERC/public data)
   - Key metrics table (metric / value)
   - Source citation note block

2. Core Technologies and Approaches
   - Technology layer classification within the sector (typically 3–4 layers)
   - Table per layer: tech layer / representative approach / technical core

3. Key Investment Differentiators
   - 4–5 criteria that distinguish strong companies in this sector
   - Bullet + explanation per criterion

4. Why Now — Reasons for Market Interest
   - Structural timing factors (technology maturity, regulation, large player movements)

5. Key Investor Dynamics
   - 8–10 VCs/CVCs/PEs active in this sector
   - Table: Investor / Investment style / Representative portfolio / Investment thesis

6. Competitive Landscape
   - Positioning table by layer: Layer / Representative companies / Approach / Key customers / Competitive dynamics / Total funding / Key investors
   - Mark public companies with "(NASDAQ/NYSE)"

7. Structural Risks
   - 5 investment risks for the sector overall
   - Table: Risk / Specifics / Impact level

8. Investment Perspective Summary
   - Where value is captured in the stack, key validation points, exit pathways
   - Investment priority perspective within the sector
```

### Style Guide
```javascript
const NAVY  = "1F2D3D";   // H1, cover, table headers
const BLUE  = "2E4A6B";   // H2, sub-headers
const LBLUE = "DBEAFE";   // highlighted row background
const LGRAY = "F1F5F9";   // alternating rows, note background
```
- Font: Arial throughout
- H1: size 30, bold, NAVY
- H2: size 24, bold, BLUE + bottom border (3B82F6)
- Body: size 20
- Page margins: top/right/bottom/left = 1080 DXA
- Table full width: 10080 DXA

### Output Language
- Default: **English**
- Company names, product names, and investor names remain in their original form
- All figures and data must cite their source

---

## Step 5: Deliver Outputs

Save both files to the workspace folder and present them to the user.

File naming convention:
```
{SectorName}_MarketMap.xlsx
{SectorName}_Sector_Overview.docx
```

Include a brief summary when delivering:
- Number of companies discovered and grade distribution (A/B/C/Pass)
- Names of A-grade companies
- 1–2 key observations from the research

---

## Data Integrity Principles

1. **Cite public sources only** — IEA, NERC, PitchBook, official press releases, SEC filings, etc.
2. **No estimates** — unverifiable figures should be marked "N/A" or "No public information"
3. **Prioritize recency** — funding data based on announcements within the past 12 months
4. **Minimize bias** — avoid positive/negative adjectives; use fact-based descriptions

---

## Reference Files

| File | Description |
|------|-------------|
| `references/excel_example.py` | Full openpyxl code for a sector Excel market map |
| `references/docx_example.js` | DOCX sector report v1 code (docx npm) |
| `references/docx_example_v2.js` | Updated version with 6-column competitive landscape table |

When starting a new sector, read these files for structural reference and adapt the content to the new sector. Do not copy code directly — restructure the layer breakdown, table fields, and analysis content to fit the sector's specific characteristics.

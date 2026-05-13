---
name: job-search-curator
description: >
  Automatically curates VC/CVC/Corp Dev/Strategy positions daily from Gmail and LinkedIn
  alerts, evaluates each with a 5-axis Attractiveness Score (Career Trajectory / Sector /
  Function / Comp / Strategic Optionality), and delivers results via HTML email with a
  one-line highlight per position. Configurable salary floor, location filter, and role
  preferences. Excludes previously recommended positions. Runs automatically at a
  configured daily time.
---

# Job Search Curator

Curates VC/CVC/Big Tech Corp Dev/Strategy positions daily based on your investment background,
evaluates them using a structured scoring system, and delivers results as a formatted HTML email.

## Overview

### Purpose
- **Daily automated collection**: Parse position data from Gmail (LinkedIn job alert emails)
  and direct ATS scans
- **Intelligent filtering**: Apply salary floor, location, and role-type filters (configurable)
- **Background-based curation**: Match positions against your configured career profile
- **Deduplication**: Exclude previously recommended positions using Job ID tracking
- **Proactive suggestions**: Surface additional roles that fit your profile beyond LinkedIn alerts

### Configuration (edit in `config.json`)
- `min_base_salary`: Minimum base salary (default: $230K)
- `preferred_locations`: Geographic preferences (default: SF Bay Area; Remote/Hybrid also considered)
- `target_roles`: VC Investor, CVC Manager, Corp Dev, Strategy
- `excluded_industries`: Industries to exclude from results
- `email_recipient`: Your email address for daily digest delivery
- `email_subject_prefix`: Subject line prefix (default: "Job Digest")

---

## Workflow

### Step 1A: Gmail Collection (LinkedIn Alerts)
- Search query: `from:jobalerts-noreply@linkedin.com newer_than:2d`
- Fetch up to 20 results, then individually retrieve related threads via `get_thread`
- LinkedIn Job Alert parsing pattern: extract URL from `View job:` line → read lines above
  URL in reverse order to skip badge lines (Fast growing, Easy Apply, Early applicant, etc.)
  → extract (title, company, location) as 3 fields

### Step 1B: Greenhouse Direct Scan
Parallel scan using slug list in `references/greenhouse-boards.md`:
```
GET https://boards-api.greenhouse.io/v1/boards/{slug}/jobs
```
- Include keywords (title): `invest|corp.dev|m&a|strategy|partnerships|venture|deal|principal|managing.director|growth`
- Location filter: US/Remote first; exclude India/Mexico/EMEA-only
- Individually fetch JD URL for positions that pass the filter

### Step 1C: Eightfold AI ATS Direct Scan (Netflix, etc.)
Some companies use Eightfold AI ATS instead of Greenhouse (URL pattern: `explore.jobs.{company}.net`). API endpoint:
```
GET https://explore.jobs.{company}.net/api/apply/v2/jobs?domain={company}.com&query={keyword}&start=0&num=50&sort_by=relevance
```

**Target company slugs (extend as needed)**:
| Company | API base |
|---------|----------|
| Netflix | `explore.jobs.netflix.net` (domain=netflix.com) |
| (add more using the same pattern) | |

**Scan pipeline**:
1. Run 5 keyword queries in parallel per company: `corporate development`, `strategy`, `m&a`, `investments`, `partnerships`
2. Parse `positions[]` from response:
   - `name` → title
   - `location` or `locations[]` → location
   - `department` → team
   - `display_job_id` → job ID
   - `canonicalPositionUrl` → JD URL
3. Location filter: US/Remote first; exclude APAC/EMEA/LATAM-only
4. Seniority filter: title must include `Director|Principal|Senior|Head|VP|Lead|Manager`
5. Department filter: prioritize Finance, Strategy, Corp Dev, Partnerships
6. **Dedup**: Multiple keywords may return the same position — dedup by `id` or `display_job_id`
7. **JD body**: `job_description` in list response is empty string; individually fetch `canonicalPositionUrl` (HTML) if needed
8. Feed into Step 4 scoring

### Step 2: Deduplication (seen_jobs)
1. Read `seen_jobs.json` (read-only)
2. Query `list_drafts` for subject containing configured prefix from last 30 days → extract Job ID patterns to supplement seen list
3. Exclude all positions in the combined seen list

### Step 3: Position Attractiveness Scoring

**Career Profile Configuration (edit in `config.json` or inline below)**

| Field | Description |
|-------|-------------|
| Current level | Managing Director / VP / Principal (configure to match your level) |
| Experience | Total years and firm trajectory |
| Target roles | CVC MD+, Corp Dev Director/VP, Strategy Sr. Director+ |
| Target sectors | Tier 1 (configure to match your focus sectors) |
| Comp floor | Minimum base (default: $230K) |
| Location | Preferred metro; Remote/Hybrid acceptable |
| Strengths | Full-cycle CVC, sector validation, bilingual, structured finance, etc. |
| Career direction | Upgrade = larger fund / broader authority / more independence; Lateral = equivalent comp/brand; Downgrade = IC only / no investment authority / sector retreat |

**Sector Tier Classification (configure to match your focus)**
- Tier 1 (full score): e.g., AI Infra, Data Center, Energy Storage/Grid, Power Electronics, Climate Tech
- Tier 2 (partial): e.g., Enterprise SaaS (infra angle), Industrial IoT, Semiconductor
- Tier 3 (low score): Consumer Tech, Healthcare, Fintech, Crypto

---

#### 3-A. Hard Filters (AND logic — fail any one → move to excluded table)

| Condition | Criteria | Exclusion reason |
|-----------|----------|-----------------|
| Seniority | MD / Director / VP / Principal or above | "Below seniority threshold" |
| Location | US (preferred metro) / Remote / Hybrid | "Location mismatch" |
| Sector | Tier 3 only (Consumer, Crypto, etc.) | "Sector mismatch" |
| Duplicate | Already in seen_jobs.json | "Previously recommended" |

Positions with no salary listed are not excluded — marked "Not disclosed" and passed through.

---

#### 3-B. 5-Axis Attractiveness Score (total: 10 points)

Each axis scores 0.0–2.0 points. Sum is the final score. **Positions below 4.0 move to the excluded table.**

---

**Axis 1 — Career Trajectory Fit (0–2 pts)**
Assess the career move character relative to your current level and direction.

| Assessment | Score | Criteria |
|------------|-------|----------|
| Upgrade | 1.8–2.0 | Larger fund / broader investment authority / more independent decision-making / title advancement |
| Strong Lateral | 1.4–1.7 | Equivalent CVC/Corp Dev, improved brand or comp |
| Lateral | 1.0–1.3 | Same function and level, no improvement factors |
| Downgrade | 0–0.9 | IC only, no investment authority, sector retreat, title decline |

---

**Axis 2 — Sector Depth Fit (0–2 pts)**
Match JD's stated investment sector or business domain against your configured Tier classification.

| Assessment | Score | Criteria |
|------------|-------|----------|
| Tier 1 primary | 1.6–2.0 | Your Tier 1 sectors are the main focus |
| Tier 1+2 mix | 1.2–1.5 | Tier 1 represents 40%+ of sectors mentioned |
| Tier 2 primary | 0.8–1.1 | Semiconductor, Industrial IoT, Enterprise SaaS (infra angle) |
| Tier 3 mix | 0.4–0.7 | Low Tier 1 share, Consumer/Fintech present |
| Tier 3 only | 0–0.3 | (Mostly caught by hard filter, handled as exception) |

---

**Axis 3 — Function Fit (0–2 pts)**
How closely the position's actual work overlaps with your strengths.

| Assessment | Score | Criteria |
|------------|-------|----------|
| CVC Full Cycle | 1.8–2.0 | Sourcing + DD + Board + Portfolio Management |
| Corp Dev M&A | 1.4–1.7 | M&A execution, strategic investment, partnership deals |
| Strategy/BD | 1.0–1.3 | Partnerships/market analysis/BD without investment authority |
| Research/Advisory | 0.5–0.9 | Analysis or advisory without execution authority |
| Other | 0–0.4 | Operations, PM, financial management, etc. |

---

**Axis 4 — Comp Attractiveness (0–2 pts)**
Where stated, use base salary. Where not, estimate from company size, level, and sector.

| Assessment | Score | Criteria |
|------------|-------|----------|
| Base ≥ $300K | 2.0 | |
| Base $270–299K | 1.6 | |
| Base $250–269K | 1.2 | |
| Base $230–249K | 0.8 | |
| Base < $230K | 0.2 | |
| Not disclosed (large tech/CVC) | 1.2 est. | Must mark as estimated |
| Not disclosed (small/startup) | 0.8 est. | Must mark as estimated |

---

**Axis 5 — Strategic Optionality (0–2 pts)**
Does this position expand future career options (brand leverage, network, exit path)?

| Assessment | Score | Criteria |
|------------|-------|----------|
| High Optionality | 1.6–2.0 | Platform for independent VC launch / Hyperscaler brand / large PE entry / global network |
| Moderate | 1.0–1.5 | Industry brand present, network expansion possible |
| Neutral | 0.6–0.9 | Similar to current level |
| Limiting | 0–0.5 | Niche sector lock-in, small scale, weak brand |

---

#### 3-C. Attractiveness Highlight

For each position, write a **one-line summary in English (under 15 words)** capturing the core attraction.
Base it on the highest-scoring axis.

Examples:
- `"Full-cycle CVC with broader mandate and larger fund size"` → Trajectory + Sector strong
- `"Hyperscaler brand with strong global network leverage"` → Optionality strong
- `"Base $320K, highly competitive TC"` → Comp strong

If a position is not attractive: `"Limited improvement factors relative to current role"` — be direct.

---

#### 3-D. Scoring Record Format (internal — before email generation)

For each position, organize internally with this structure before passing to Step 4:

```
Company: [name]
Title: [position]
Score: [X.X / 10]
  - Trajectory: X.X  → [Upgrade/Lateral/Downgrade + 1-line rationale]
  - Sector:     X.X  → [Tier classification + rationale]
  - Function:   X.X  → [Full Cycle/Corp Dev/etc. + rationale]
  - Comp:       X.X  → [$XXXk stated or estimated]
  - Optionality:X.X  → [High/Moderate/Neutral/Limiting + rationale]
Highlight: [one-line attractiveness summary]
Action: INCLUDE / EXCLUDE (reason)
```

> ⚠️ NO HALLUCINATION: Only evaluate positions that were actually collected. Comp not stated in the JD must be marked "estimated". If zero positions are collected: write "No LinkedIn alerts received today — will check again tomorrow." as the draft body, then stop.

### Step 4: Table Generation & Email Send (HTML body required)

**Core principle**: Gmail strips markdown and plain text of formatting. The `create_draft` call
must use the `htmlBody` parameter with the full HTML below. The `body` (plain text) field
should contain only a 3-line text fallback.

**Gmail HTML constraints**:
- `<style>` blocks, external CSS, `<script>` tags → Gmail strips all of these. Forbidden.
- All styles must use **inline `style="..."`** only
- Use `<table>` layout (div float/flexbox breaks in Gmail)

---

**Full HTML template** (use this structure exactly; fill with real data):

```html
<div style="font-family:-apple-system,'Helvetica Neue',Arial,sans-serif;color:#1a1a1a;max-width:900px;margin:0 auto;padding:16px;">

  <!-- ① Header banner -->
  <table cellpadding="0" cellspacing="0" border="0" style="width:100%;border-collapse:collapse;margin-bottom:4px;">
    <tr>
      <td style="background:#0b3d91;padding:18px 20px;border-radius:6px 6px 0 0;">
        <div style="color:#ffffff;font-size:20px;font-weight:700;margin:0;">Job Digest — [Day of Week], [Month] [D], [YYYY]</div>
        <div style="color:#b8cef0;font-size:12px;margin-top:4px;">LinkedIn alerts received: XX · New recommendations: XX (score 4.0+)</div>
      </td>
    </tr>
  </table>

  <!-- ② KPI strip (4 cells) -->
  <table cellpadding="0" cellspacing="0" border="0" style="width:100%;border-collapse:collapse;margin-bottom:20px;">
    <tr>
      <td style="background:#f0f4fa;border:1px solid #d6e0ee;border-top:none;padding:12px 16px;width:25%;text-align:center;">
        <div style="font-size:11px;color:#666;margin-bottom:4px;letter-spacing:0.5px;text-transform:uppercase;">Alerts received (48h)</div>
        <div style="font-size:26px;font-weight:700;color:#0b3d91;line-height:1;">XX</div>
      </td>
      <td style="background:#f0faf4;border:1px solid #c3e6cb;border-top:none;border-left:none;padding:12px 16px;width:25%;text-align:center;">
        <div style="font-size:11px;color:#666;margin-bottom:4px;letter-spacing:0.5px;text-transform:uppercase;">New recommendations</div>
        <div style="font-size:26px;font-weight:700;color:#1a6b3a;line-height:1;">XX</div>
      </td>
      <td style="background:#fffbf0;border:1px solid #f5d78e;border-top:none;border-left:none;padding:12px 16px;width:25%;text-align:center;">
        <div style="font-size:11px;color:#666;margin-bottom:4px;letter-spacing:0.5px;text-transform:uppercase;">Top match score</div>
        <div style="font-size:26px;font-weight:700;color:#a06800;line-height:1;">X.X</div>
      </td>
      <td style="background:#fff5f5;border:1px solid #f5c6cb;border-top:none;border-left:none;padding:12px 16px;width:25%;text-align:center;">
        <div style="font-size:11px;color:#666;margin-bottom:4px;letter-spacing:0.5px;text-transform:uppercase;">Duplicates excluded</div>
        <div style="font-size:26px;font-weight:700;color:#b02a37;line-height:1;">XX</div>
      </td>
    </tr>
  </table>

  <!-- ③ Top 3 cards -->
  <div style="font-size:15px;font-weight:700;color:#0b3d91;border-left:4px solid #0b3d91;padding-left:10px;margin-bottom:10px;">Top 3 by Attractiveness Score</div>
  <table cellpadding="0" cellspacing="0" border="0" style="width:100%;border-collapse:collapse;margin-bottom:24px;">
    <tr>
      <!-- Rank 1 card -->
      <td style="background:#fffbe5;border:1px solid #e6d278;border-left:5px solid #d4a017;padding:14px;width:33%;vertical-align:top;">
        <div style="font-size:10px;color:#a06800;font-weight:700;letter-spacing:1px;margin-bottom:6px;">🥇 #1</div>
        <div style="font-size:24px;font-weight:800;color:#a06800;margin-bottom:2px;">X.X<span style="font-size:13px;font-weight:500;color:#888;"> / 10</span></div>
        <div style="font-size:13px;font-weight:700;color:#1a1a1a;margin-bottom:2px;">[Company]</div>
        <div style="font-size:12px;color:#333;margin-bottom:6px;">[Position title]</div>
        <!-- 5-axis mini scores -->
        <table cellpadding="0" cellspacing="0" border="0" style="width:100%;font-size:10px;color:#666;margin-bottom:8px;">
          <tr><td style="padding:1px 0;">Trajectory</td><td style="text-align:right;color:#a06800;font-weight:700;">X.X</td></tr>
          <tr><td style="padding:1px 0;">Sector</td><td style="text-align:right;color:#a06800;font-weight:700;">X.X</td></tr>
          <tr><td style="padding:1px 0;">Function</td><td style="text-align:right;color:#a06800;font-weight:700;">X.X</td></tr>
          <tr><td style="padding:1px 0;">Comp</td><td style="text-align:right;color:#a06800;font-weight:700;">X.X</td></tr>
          <tr><td style="padding:1px 0;">Optionality</td><td style="text-align:right;color:#a06800;font-weight:700;">X.X</td></tr>
        </table>
        <!-- Highlight -->
        <div style="background:#fff8dc;border:1px solid #e6d278;border-radius:3px;padding:6px 8px;font-size:11px;color:#7a5500;font-style:italic;">[Attractiveness highlight]</div>
        <div style="font-size:11px;color:#888;margin-top:6px;">[Location] · [Salary or estimated/not disclosed]</div>
      </td>
      <!-- Rank 2 card -->
      <td style="background:#f7f7f7;border:1px solid #d0d0d0;border-left:5px solid #8a9bb0;padding:14px;width:33%;vertical-align:top;">
        <div style="font-size:10px;color:#5a6a7a;font-weight:700;letter-spacing:1px;margin-bottom:6px;">🥈 #2</div>
        <div style="font-size:24px;font-weight:800;color:#3a5a85;margin-bottom:2px;">X.X<span style="font-size:13px;font-weight:500;color:#888;"> / 10</span></div>
        <div style="font-size:13px;font-weight:700;color:#1a1a1a;margin-bottom:2px;">[Company]</div>
        <div style="font-size:12px;color:#333;margin-bottom:6px;">[Position title]</div>
        <table cellpadding="0" cellspacing="0" border="0" style="width:100%;font-size:10px;color:#666;margin-bottom:8px;">
          <tr><td style="padding:1px 0;">Trajectory</td><td style="text-align:right;color:#3a5a85;font-weight:700;">X.X</td></tr>
          <tr><td style="padding:1px 0;">Sector</td><td style="text-align:right;color:#3a5a85;font-weight:700;">X.X</td></tr>
          <tr><td style="padding:1px 0;">Function</td><td style="text-align:right;color:#3a5a85;font-weight:700;">X.X</td></tr>
          <tr><td style="padding:1px 0;">Comp</td><td style="text-align:right;color:#3a5a85;font-weight:700;">X.X</td></tr>
          <tr><td style="padding:1px 0;">Optionality</td><td style="text-align:right;color:#3a5a85;font-weight:700;">X.X</td></tr>
        </table>
        <div style="background:#eef3fb;border:1px solid #c0d0e8;border-radius:3px;padding:6px 8px;font-size:11px;color:#2a4a75;font-style:italic;">[Attractiveness highlight]</div>
        <div style="font-size:11px;color:#888;margin-top:6px;">[Location] · [Salary or estimated/not disclosed]</div>
      </td>
      <!-- Rank 3 card -->
      <td style="background:#fdf5ee;border:1px solid #ddc8a8;border-left:5px solid #b87333;padding:14px;width:33%;vertical-align:top;">
        <div style="font-size:10px;color:#7a4a20;font-weight:700;letter-spacing:1px;margin-bottom:6px;">🥉 #3</div>
        <div style="font-size:24px;font-weight:800;color:#7a4a20;margin-bottom:2px;">X.X<span style="font-size:13px;font-weight:500;color:#888;"> / 10</span></div>
        <div style="font-size:13px;font-weight:700;color:#1a1a1a;margin-bottom:2px;">[Company]</div>
        <div style="font-size:12px;color:#333;margin-bottom:6px;">[Position title]</div>
        <table cellpadding="0" cellspacing="0" border="0" style="width:100%;font-size:10px;color:#666;margin-bottom:8px;">
          <tr><td style="padding:1px 0;">Trajectory</td><td style="text-align:right;color:#7a4a20;font-weight:700;">X.X</td></tr>
          <tr><td style="padding:1px 0;">Sector</td><td style="text-align:right;color:#7a4a20;font-weight:700;">X.X</td></tr>
          <tr><td style="padding:1px 0;">Function</td><td style="text-align:right;color:#7a4a20;font-weight:700;">X.X</td></tr>
          <tr><td style="padding:1px 0;">Comp</td><td style="text-align:right;color:#7a4a20;font-weight:700;">X.X</td></tr>
          <tr><td style="padding:1px 0;">Optionality</td><td style="text-align:right;color:#7a4a20;font-weight:700;">X.X</td></tr>
        </table>
        <div style="background:#fdf0e0;border:1px solid #ddc8a8;border-radius:3px;padding:6px 8px;font-size:11px;color:#7a4a20;font-style:italic;">[Attractiveness highlight]</div>
        <div style="font-size:11px;color:#888;margin-top:6px;">[Location] · [Salary or estimated/not disclosed]</div>
      </td>
    </tr>
  </table>

  <!-- ④ Full position table -->
  <div style="font-size:15px;font-weight:700;color:#0b3d91;border-left:4px solid #0b3d91;padding-left:10px;margin-bottom:10px;">All Recommended Positions</div>
  <table cellpadding="0" cellspacing="0" border="0" style="width:100%;border-collapse:collapse;font-size:12px;margin-bottom:24px;">
    <thead>
      <tr style="background:#0b3d91;color:#ffffff;">
        <th style="padding:9px 10px;text-align:left;font-weight:600;border:1px solid #0b3d91;white-space:nowrap;">Job ID</th>
        <th style="padding:9px 10px;text-align:left;font-weight:600;border:1px solid #0b3d91;">Company</th>
        <th style="padding:9px 10px;text-align:left;font-weight:600;border:1px solid #0b3d91;">Position</th>
        <th style="padding:9px 10px;text-align:center;font-weight:600;border:1px solid #0b3d91;white-space:nowrap;">Base</th>
        <th style="padding:9px 10px;text-align:center;font-weight:600;border:1px solid #0b3d91;">Location</th>
        <th style="padding:9px 10px;text-align:center;font-weight:600;border:1px solid #0b3d91;white-space:nowrap;">Score</th>
        <th style="padding:9px 10px;text-align:left;font-weight:600;border:1px solid #0b3d91;">Highlight</th>
        <th style="padding:9px 10px;text-align:center;font-weight:600;border:1px solid #0b3d91;">JD</th>
      </tr>
    </thead>
    <tbody>
      <!-- Rows: odd rows background #ffffff, even rows #f8f9fb / rank 1 row: background #fffbe5 -->
      <!-- Score colors: 8.0+ → color:#a06800;font-weight:700 / 6.0–7.9 → color:#3a5a85;font-weight:700 / 4.0–5.9 → color:#555;font-weight:600 -->
      <tr style="background:#fffbe5;">
        <td style="padding:8px 10px;border:1px solid #e8e8e8;font-family:monospace;font-size:11px;color:#555;">[JOB-ID]</td>
        <td style="padding:8px 10px;border:1px solid #e8e8e8;font-weight:700;color:#0b3d91;">[Company]</td>
        <td style="padding:8px 10px;border:1px solid #e8e8e8;">[Position title]</td>
        <td style="padding:8px 10px;border:1px solid #e8e8e8;text-align:center;white-space:nowrap;">$XXXk</td>
        <td style="padding:8px 10px;border:1px solid #e8e8e8;text-align:center;">[Location]</td>
        <td style="padding:8px 10px;border:1px solid #e8e8e8;text-align:center;color:#a06800;font-weight:700;">X.X</td>
        <td style="padding:8px 10px;border:1px solid #e8e8e8;font-size:11px;color:#555;font-style:italic;">[Highlight]</td>
        <td style="padding:8px 10px;border:1px solid #e8e8e8;text-align:center;"><a href="[URL]" style="color:#0b3d91;text-decoration:none;font-weight:600;">▸ View</a></td>
      </tr>
    </tbody>
  </table>

  <!-- ⑤ Analysis comment -->
  <div style="font-size:15px;font-weight:700;color:#0b3d91;border-left:4px solid #0b3d91;padding-left:10px;margin-bottom:8px;">Analysis Comment</div>
  <div style="background:#f7f9fc;border:1px solid #dde6f0;border-radius:4px;padding:14px 16px;font-size:13px;line-height:1.7;color:#333;margin-bottom:20px;">
    [2–4 sentences of analysis on today's curation. Notable companies, connections between JD requirements and your background, VC-angle insights.]
  </div>

  <!-- ⑥ Excluded positions (only if present) -->
  <div style="font-size:15px;font-weight:700;color:#0b3d91;border-left:4px solid #0b3d91;padding-left:10px;margin-bottom:8px;">Excluded Positions</div>
  <table cellpadding="0" cellspacing="0" border="0" style="width:100%;border-collapse:collapse;font-size:12px;margin-bottom:20px;">
    <thead>
      <tr style="background:#6c757d;color:#fff;">
        <th style="padding:7px 10px;text-align:left;border:1px solid #6c757d;">Company</th>
        <th style="padding:7px 10px;text-align:left;border:1px solid #6c757d;">Position</th>
        <th style="padding:7px 10px;text-align:left;border:1px solid #6c757d;">Reason</th>
      </tr>
    </thead>
    <tbody>
      <tr style="background:#f8f9fb;">
        <td style="padding:7px 10px;border:1px solid #e0e0e0;color:#555;">[Company]</td>
        <td style="padding:7px 10px;border:1px solid #e0e0e0;color:#555;">[Position]</td>
        <td style="padding:7px 10px;border:1px solid #e0e0e0;color:#888;font-size:11px;">[Previously recommended / Below salary floor / Location mismatch / etc.]</td>
      </tr>
    </tbody>
  </table>

  <!-- ⑦ Proactive suggestions -->
  <div style="font-size:15px;font-weight:700;color:#0b3d91;border-left:4px solid #0b3d91;padding-left:10px;margin-bottom:8px;">Proactive Suggestions (beyond LinkedIn alerts)</div>
  <div style="background:#eef3fb;border:1px solid #c9d9f0;border-radius:4px;padding:14px 16px;font-size:13px;line-height:1.8;color:#333;margin-bottom:20px;">
    [1–3 position suggestions connected to your background. Company + position + one-line rationale each.]
  </div>

  <!-- ⑧ Footer -->
  <div style="border-top:1px solid #e0e0e0;padding-top:10px;font-size:11px;color:#999;text-align:center;">
    Job Search Curator · Auto-generated · [Your Firm] — [Your Name]
  </div>

</div>
```

**create_draft call pattern**:
```javascript
create_draft({
  to: ["[your-email@yourfirm.com]"],
  subject: "[Job Digest] (YYYY-MM-DD)",
  body: "New recommendations today: X (Top: [Company] [Position] X.X pts). See HTML body for full table.",
  htmlBody: <full HTML above>
})
```

**Data fill checklist**:
- [ ] Header: date (with day of week), alerts received count, new recommendations count
- [ ] KPI strip: all 4 figures with real values
- [ ] Top 3 cards: real score, company, position, location, salary data
- [ ] Full table: all recommended positions (score descending, zebra stripe, score colors applied)
- [ ] Excluded table: add rows if exclusions exist; omit entire section if none
- [ ] Analysis comment: real insight in 2–4 sentences

### Step 5: Job ID Storage
Add all today's recommended Job IDs to `seen_jobs.json` (prevents next-cycle duplicates).

---

## Setup & Requirements

### Gmail Access
- Account: configured in `config.json`
- Permissions: read + `create_draft`

### LinkedIn Alerts
- Must be configured to send daily email alerts to your Gmail account

### Storage Structure
```
job-search-curator/
├── SKILL.md (this file)
├── seen_jobs.json (recommendation history)
├── config.json (settings: email, salary floor, location prefs, etc.)
├── blocklist.json (excluded keywords/locations)
├── references/
│   └── greenhouse-boards.md
└── logs/
    └── YYYY-MM-DD.log
```

### Expected Runtime
- Under 30 seconds total (HTML body generation adds ~5 seconds)

---

## Notes
- **HTML body required**: Do not use markdown tables — Gmail won't render them
- **Gmail limitation**: Email body access only; LinkedIn dashboard scraping not available
- **Time zone**: Run at your configured local time, accounting for daylight saving
- **Job URL**: If no link is available in the email, mark as "request required"

---

## Customization

Edit `config.json` to adjust:
- `min_base_salary`: current default $230K
- `preferred_locations`: current default Bay Area
- `excluded_industries`
- `matching_weights`
- `email_recipient`
- `run_time`

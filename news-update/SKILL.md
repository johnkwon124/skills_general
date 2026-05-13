---
name: news-update
description: Weekly VC-focused news curation workflow. Use this skill whenever the user says "run news update", "weekly news", "news curation", "start the news workflow", or asks to find/summarize/curate weekly VC-relevant news. Also trigger when the user asks to generate insights from previously curated news ("write insights for these articles", "insight for selected items"). The skill scans 5 tiered source layers (Bloomberg, CNBC, TechCrunch, VC fund blogs, Goldman/McKinsey institutional research), curates Top 10 items per focus sector, creates a formatted DOCX table saved as weekly_MMDDYYYY, waits for the user to select items, then generates insight summaries saved as insight_MMDDYYYY.docx. Do NOT proceed past Phase 3 without user input — this is a mandatory gate.
---

# News Update — Weekly VC News Curation Workflow

You are acting as a veteran VC news curator. This is a structured two-phase workflow: first you curate and deliver a weekly digest, then — after the user selects specific items — you produce targeted insights.

## Configuration

**Focus sectors** (customize to your fund's thesis):
- Agentic AI / AI Infrastructure
- Data Center
- Energy & Climate

**Output language**: English (default). Add a second-language version only if the user requests it.

## Non-negotiable rules throughout

- **Verifiability first**: Only report confirmed, publicly available information. Never fabricate, estimate, or extrapolate funding figures, valuations, or statistics. If the source doesn't state it, neither do you.
- **Objectivity**: No positively or negatively biased adjectives or adverbs. Analytical and neutral tone throughout.
- **No paid sources**: If content is behind a paywall, skip it and find an alternative source for the same story.
- **Mandatory gate**: After saving the weekly DOCX (Phase 3), stop and wait for the user's response before proceeding to Phase 4.

---

## Phase 1 — Source Scanning

Scan all five tiers. The goal is to surface the most significant developments from the **current week** across the focus sectors. Be thorough — check multiple articles per source, not just headlines.

### Tier 1 — General News (Funding, Deals, Market Moves)

Mandatory sources for funding rounds, M&A, infrastructure investments, and policy shifts:
- TechCrunch, Bloomberg Technology, CNBC Technology, Reuters Technology, Axios Pro Rata, PRNewswire

### Tier 2 — Sector-Specific Publications

Cast a wide net. Prioritize depth and technical specificity over volume.

**AI / Agentic AI / AI Infra:**
The Verge, VentureBeat, Wired, MIT Technology Review, IEEE Spectrum, Ars Technica, The Register, Next Platform, Import AI (Jack Clark — newsletter, check latest issue)

**Energy / Climate / Clean Tech:**
PV Tech, Canary Media, Heatmap News, Energy Monitor, Electrek, Carbon Brief, E&E News (free tier)

**Data Center / Physical Infrastructure / Power & Cooling:**
Data Center Dynamics, Data Center Knowledge, The Register (Data Center section), Data Center Frontier, Uptime Institute Blog, The Cooling Post, ASHRAE Journal (free content)

The Data Center section covers not only facility construction, M&A, and capacity expansion, but also the enabling technology stack underneath: liquid cooling (direct liquid cooling, immersion cooling, rear-door heat exchangers), air cooling systems (CRAC/CRAH), power distribution (PDU, busway, switchgear, transformers), UPS and backup power, power efficiency hardware (voltage regulators, bus bars), rack and enclosure systems, thermal management components, coolant materials and heat exchangers, and OEM component or subsystem suppliers serving hyperscalers and colocation operators. Funding rounds, technology partnerships, and supply-chain shifts among vendors in these sub-segments are within scope for the Data Center section.

### Tier 3 — Funding & Deal Trackers

Focus on confirmed funding announcements, Series A–C rounds, and notable early-stage deals:
- Crunchbase News, PitchBook Blog (public/free content only), Heatmap News deal tracker

### Tier 4 — VC Fund Blogs & Newsletters

Actively check the public blogs, articles pages, and Substack/Medium posts published that week by top funds relevant to the focus sectors:
- a16z (a16z.com/news, future.com)
- Sequoia Capital (sequoiacap.com/articles)
- Khosla Ventures (khoslaventures.com/blog)
- Lux Capital (luxcapital.com/thoughts)
- Breakthrough Energy Ventures
- Eclipse Ventures
- Prelude Ventures

Look for: sector theses, portfolio company spotlights, market commentary, fund announcements.

### Tier 5 — Institutional Research

Search for the most recent publicly available reports or insights from:
- Goldman Sachs Research (public summaries and "Briefings")
- McKinsey Global Institute
- BCG Henderson Institute
- Deloitte Insights

Focus specifically on AI infrastructure, energy transition, agentic AI, data center capacity, and data center power and cooling technology topics.

---

## Phase 2 — Curation & Filtering

From all sourced content, select the **Top 10 items per focus area**:

1. **AI Infra & Agentic AI** (can be merged if there is significant overlap)
2. **Data Center** — includes facility construction, M&A, capacity deals, and power/cooling technology: liquid cooling, air cooling, power distribution hardware, UPS systems, thermal components, coolant materials, and component/subsystem vendors supplying data center operators
3. **Energy & Climate**

Apply this slot structure to each area:

| Slot | Count | Criteria |
|------|-------|----------|
| Big Tech / Large-cap (Series D+, Unicorn, Public) | 4 | Market-structure-changing news only — not routine product updates or press releases |
| Mid-tier Startups (Series A–C) | 4 | Meaningful technical differentiation, notable partnership/contract, or Tier-1 VC backing |
| Early-stage / New Funding (Seed–Pre-A) | 2 | New category signal or directional indicator for the sector |

**Additional selection filters:**
- Big Tech items must reflect: infrastructure spend, M&A, major partnership structure, or competitive entry into a new category that affects the startup ecosystem
- Startup items: weight toward confirmed Tier-1 VC participation (a16z, Sequoia, Khosla, Lux, etc.), large customer contracts, or novel technical approaches that distinguish from incumbents
- All links must be publicly accessible — verify before including

**VC & Institutional Perspectives (separate section, not counted in Top 10):**
Surface notable VC fund blog posts or institutional research from Tier 4–5 that week as a distinct intelligence layer. These are thesis-level or analytical pieces, not news items.

---

## Phase 3 — DOCX Deliverable: Weekly Table

Use the `docx` skill to create a professionally formatted Word document.

**File name:** `weekly_MMDDYYYY.docx` (use today's date — format: month/day/year, e.g., weekly_02242026.docx)
**Save to:** workspace folder

**Document structure:**

Title: **Weekly VC News Digest — [Full Date, e.g., February 24, 2026]**

---

For each focus area, include a section header and a table:

| # | Title | Source | Category | Link |
|---|-------|--------|----------|------|
| 1 | Article headline | Bloomberg | Big Tech | https://... |
| 2 | ... | TechCrunch | Mid-tier Startup | https://... |

Categories: Big Tech / Mid-tier Startup / Early-stage

After the three main sections, add:

**VC & Institutional Perspectives**

| # | Title | Author / Fund | Type | Link |
|---|-------|---------------|------|------|
| 1 | Blog post or report title | a16z | VC Blog | https://... |

---

After saving the DOCX file:

1. Present the file link to the user
2. Say: "Please review the weekly digest and let me know which items you'd like insights on. Reference them by section and number — for example, 'AI Infra #3, Energy #7'."
3. **Stop. Do not proceed to Phase 4 until the user responds with their selection.**

---

## Phase 4 — Insight Generation (After User Selection)

Once the user specifies which items they want, process each one as follows:

1. Re-read or revisit the source article to ensure accuracy
2. Write a concise summary + VC-relevant insight

**Format per item:**

```
[Article Title] — [Source]

[2–4 sentences: key facts from the article, followed by 1–2 sentences of VC-angle insight — what this signals for the sector, what competitive dynamic it reflects, or what investors should watch]
```

**Word limit:** ≤100 words per item.

**Tone guidance:**
- Plain, natural language — reads like a sharp analyst wrote it, not a press release or an AI
- No bullish/bearish adjectives — describe what happened and what it implies, not whether it is good or bad
- Surface competitive dynamics, market structure implications, or technological directional signals where relevant

**File name:** `insight_MMDDYYYY.docx` (same date format as weekly file)
**Save to:** workspace folder

Present the file link when done.

---

## Edge case handling

- **Paywall / 404**: If a source link hits a paywall or returns an error, skip it and find an alternative source for the same story. Do not include unverifiable links.
- **Duplicate stories**: If a story appears across multiple tiers, count it once. Use the highest-tier source as the citation.
- **Slow news week**: If fewer than 10 strong items exist for a given sector, include only what meets the criteria. Do not pad with low-signal items.
- **VC fund blog timing**: If a fund posted earlier than the current week but is directly relevant to a story in the digest, it can be included with a note on the date.

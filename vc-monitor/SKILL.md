---
name: vc-monitor
description: >
  Automated VC deal flow and news monitoring for your firm. Trigger on "run monitoring",
  "check deal flow", "portfolio update", "weekly monitoring", "news monitoring", "run monitor",
  or requests to review collected news. Also triggers when asking about your configured portfolio
  companies or target startups, or when setting up the Apps Script infrastructure.
  Three modes: (A) Infrastructure setup — deploy vc_news_monitor_v2.gs and Google Alerts;
  (B) Manual weekly review — read Google Sheet, produce DOCX digest, wait for selection,
  generate bilingual insights (Phase 3 gate is mandatory); (C) Scheduled weekly delivery —
  curate 7-day window via web search, ship bilingual EN+KR digests to your inbox.
  In Mode C, attempt direct send first; fall back to draft only if no send-capable Gmail
  tool is connected.
---

# VC Monitor — Automated Deal Flow & News Monitoring

This skill operates in two modes depending on context.

**Mode A — Infrastructure Setup**: User wants to deploy or configure the monitoring system.
**Mode B — Weekly Review**: The Apps Script has already run; user wants Claude to process
and curate the week's collected data.

Detect which mode applies from the conversation. If ambiguous, ask before proceeding.

---

## Non-negotiable rules

- **Verifiability only**: Never fabricate, estimate, or extrapolate funding figures, valuations,
  or statistics. Report only what is explicitly stated in the source.
- **No paywalled content**: If a link requires login or subscription, skip it.
- **Objectivity**: No positively or negatively biased language. Analytical tone throughout.
- **Bilingual outputs**: Phase 4 insight DOCX in English and Korean. Phase 3 digest in English only.
- **Mandatory gate at Phase 3**: After delivering the weekly DOCX, stop and wait for user's
  item selection before proceeding to Phase 4.
- **Phase 4 fetch limit**: Accept a maximum of 5 items per session. If the user selects more,
  ask them to prioritize to 5 and note that remaining items can be processed in a new session.

---

## Mode A — Infrastructure Setup (Compressed)

For full setup instructions, refer to the Apps Script file (`vc_news_monitor_v2.gs`) comments
and Google Alerts configuration guide. Key steps: (1) deploy the `.gs` file to Google Apps Script,
(2) configure entity-based Google Alerts per `references/entities.md`, (3) set a Monday 07:00
time-based trigger. If the user needs detailed step-by-step guidance, ask them to start a
dedicated setup session.

---

## Mode B — Weekly Review Workflow

This is the primary recurring workflow. It runs after Apps Script has populated the Google Sheet.

### Phase 1 — Data Ingestion

The Apps Script (`vc_news_monitor_v2.gs`) automatically collects from 60+ RSS feeds and Gmail
Google Alerts every Monday at 07:00. It writes results to the "VC Monitor" Google Sheet with tabs:

| Tab | Content |
|-----|---------|
| All News | Full collection, sorted by score descending |
| AI Foundation | Leading foundation model labs & releases |
| AI Data Center | Infrastructure, hardware, cooling, power |
| Energy | Solar, geothermal, ESS, clean power |
| Portfolio | Configured portfolio company mentions |
| Key Startups | Target startup mentions |
| VC Blogs | Posts from a16z, Sequoia, Khosla, Lux, etc. |
| Research | McKinsey, Goldman, BCG, IEA public reports |

Configure your portfolio companies and key startups in `references/entities.md`.

**If user provides the Sheet link**: Use Google Drive fetch to read the Sheet data.
**If user pastes data directly**: Process what's provided.
**If no data is available**: Prompt the user to run the Apps Script first, or offer to switch
to the manual `news-update` skill which uses web search directly.

---

### Phase 2 — Curation & Filtering

Load `references/entities.md` to apply entity-specific flag criteria and alias matching.

From the Sheet data, apply the following selection logic per category.

**Portfolio & Key Startups (priority review — always first)**

These are entity-based. For flag criteria and alias lists, refer to `references/entities.md`.
Any mention is potentially significant. Apply the flag criteria defined in that file.

**Thematic categories: AI Foundation / AI Data Center / Energy**

Apply this slot structure (Top 10 per category):

| Slot | Count | Criteria |
|------|-------|----------|
| Big Tech / Large-cap | 4 | Market-structure-changing only. Not routine product updates. |
| Mid-tier Startups (Series A–C) | 4 | Tier-1 VC backing, notable contract, or technical differentiation |
| Early-stage / New Funding | 2 | New category signal or directional indicator |

**VC Blogs & Research (separate section)**

Surface top 3–5 items from the VC Blogs and Research tabs as a distinct intelligence layer.
These are thesis-level pieces, not news items. Include fund name and publication date.

**Scoring override rules:**
- Any item from a VC Blog or institutional research source: +5 to displayed priority
- Any confirmed funding round with Tier-1 VC participation: +4
- Portfolio or Key Startup mentions: automatically surface regardless of score

---

### Phase 3 — DOCX Deliverable: Weekly Digest

Use the `docx` skill to produce a formatted Word document.

**File name:** `weekly_MMDDYYYY.docx` (today's date, e.g., `weekly_04062026.docx`)
**Save to:** Google Drive / VC Monitor folder
**Language:** English only (bilingual output reserved for Phase 4 insights)

**Document structure:**

```
Title: VC Monitor — Weekly Digest [Full Date]
Subtitle: AI Foundation · AI Data Center · Energy · Portfolio · Key Startups

─────────────────────────────────────────────────
SECTION 1: PORTFOLIO COMPANIES
─────────────────────────────────────────────────
Table: # | Title | Source | Category | Score | Link

─────────────────────────────────────────────────
SECTION 2: KEY STARTUPS TO WATCH
─────────────────────────────────────────────────
Table: # | Title | Source | Category | Score | Link

─────────────────────────────────────────────────
SECTION 3: AI FOUNDATION MODELS
─────────────────────────────────────────────────
Table: # | Title | Source | Category | Link
(Category column: Big Tech / Mid-tier / Early-stage)

─────────────────────────────────────────────────
SECTION 4: AI DATA CENTER & INFRASTRUCTURE
─────────────────────────────────────────────────
Table: [same structure]

─────────────────────────────────────────────────
SECTION 5: ENERGY SOLUTIONS
─────────────────────────────────────────────────
Table: [same structure]

─────────────────────────────────────────────────
SECTION 6: VC & INSTITUTIONAL PERSPECTIVES
─────────────────────────────────────────────────
Table: # | Title | Author / Fund | Type | Link
(Type: VC Blog / Institutional Research)

─────────────────────────────────────────────────
Footer: Generated [timestamp] | Total items collected: [N]
```

After saving and presenting the file:

1. Provide the Google Drive link to the file
2. Say in both English and Korean:
   "Please review the digest and select up to 5 items you'd like insights on.
   Reference by section and number — e.g., 'AI Data Center #3, Energy #7, Portfolio #1'.
   다이제스트를 검토하신 후 인사이트가 필요한 항목을 최대 5건 선택해 주세요.
   섹션과 번호로 지정해 주세요 — 예: 'AI Data Center #3, Energy #7, Portfolio #1'."
3. **Stop. Do not proceed to Phase 4 until the user responds.**

---

### Phase 4 — Insight Generation (After User Selection)

**Maximum 5 items per session.** If the user selects more, ask them to prioritize to 5 before
proceeding. Remaining items can be covered in a new session.

Once the user confirms their selection (≤ 5 items), for each selected item:

1. If the article URL is accessible (not paywalled), fetch it to verify key facts.
   If the URL is paywalled or unavailable, work from the headline and snippet in the Sheet only —
   note this limitation in the insight.
2. Write insight using this structure:

```
[Section] #[N] — [Article Title]
Source: [Publication] | [Date]

English:
[2–3 sentences: verified key facts] [1–2 sentences: VC-angle insight —
what this signals for the sector, competitive dynamic, or what investors should watch]

한국어:
[동일 구조 — 확인된 핵심 사실 2–3문장, VC 관점 인사이트 1–2문장]
```

**Constraints:**
- ≤ 100 words per language version per item
- No adjectives implying sentiment (exciting, significant, impressive, concerning)
- Surface: competitive dynamics, market structure shift, technology direction, or fund flow signal

**File name:** `insight_MMDDYYYY.docx` (same date as weekly file)
**Save to:** Google Drive / VC Monitor folder

Present the file link when complete.

---

## Mode C — Scheduled Weekly Delivery (Inbox-First)

Mode C is the autonomous flow used by the `vc-monitor-weekly` scheduled task. It bypasses
the Sheet/Drive pipeline entirely and curates from the open web. The user is not present;
output goes straight to the configured inbox.

### C-1 — Strict 7-day window
Curate only items dated within the last 7 calendar days. No padding with older items except as
explicitly flagged "pre-window" in early-stage / VC-blog slots when no in-window equivalent exists.

### C-2 — Coverage
- **Portfolio & Key Startups (highest priority)**: Load entity list from `references/entities.md`.
  Configure your portfolio companies and target startups there.
- **5 Sector Themes (5 items each, slot structure: Big Tech 2 / Mid-tier 2 / Early-stage 1)**:
  Configure your fund's focus sectors in `references/entities.md`
  (default: Agentic AI · AI Data Center · Energy & Grid · AI Infrastructure · Climate Tech).
- **VC Blog Perspectives (3 items)**: whitelist-only. Use the inline whitelist defined in the
  scheduled task prompt (Tier A: top VC funds by AUM; Tier B: sector-specialist VCs aligned
  with your fund's focus).

### C-3 — Item structure
Each item: exactly 2 bullets — (a) factual summary, (b) VC perspective / investment thesis
implication in your fund's focus areas. Objective tone, no superlatives, no fabricated numbers.

### C-4 — Delivery (CRITICAL — DIRECT SEND FIRST)

Generate two HTML emails:
- **English**: Subject `VC Monitor Weekly — MM/DD/YYYY` to `[your-email@yourfirm.com]`
- **Korean**: Subject `VC 모니터 위클리 — MM/DD/YYYY` to `[your-email@yourfirm.com]`
  (Korean is a natural professional rewrite — not literal translation. Mirror structure,
  preserve VC tone.)

Configure the recipient email in `references/entities.md`.

**Send-tool routing** (priority order — stop at first available, do not chain):

1. If a Gmail MCP exposing a `send_message` / `send_draft` / `send` tool is connected,
   use it to send both emails directly. This is the preferred path.
2. Else, if Chrome MCP is connected AND exactly one browser is connected (no selection prompt
   required), open `https://mail.google.com/mail/u/0/#drafts`, locate each draft by subject,
   open it, and click Send. Do NOT prompt the user to pick a browser inside an autonomous run.
   If multiple browsers are connected, skip this step (treat as unavailable) — do not block
   on user input.
3. Else (current default state as of skill v1.1), fall back to `create_draft` for both versions.
   In the completion report, surface the limitation explicitly:
   "Direct send unavailable — connect a Gmail MCP exposing a send function to enable end-to-end
   delivery. Drafts are saved at https://mail.google.com/mail/u/0/#drafts."

### C-5 — Completion report
Items per category, sources that returned empty, draft IDs (if fallback used),
and any whitelist firms that produced no qualifying within-window content.

### C-6 — Failure handling
If any single search step fails twice, stop and report status. Do not fabricate content to fill
gaps. If draft creation or direct send fails twice, do not retry indefinitely — surface the
error in the completion report and stop.

---

## Reference Files

| File | When to Load | Contents |
|------|-------------|----------|
| `references/entities.md` | Phase 2 (always) | Portfolio companies, key startups, VC blogs, institutional research sources |
| `references/sources.md` | Mode A setup or if tab mapping clarification needed | RSS feed structure, tab-to-source mapping, scoring logic |

---

## Edge Case Handling

- **Sheet not yet populated**: Prompt user to run `runWeeklyMonitoring()` in Apps Script first,
  or offer to switch to the manual `news-update` skill.
- **Duplicate items across categories**: Keep in the highest-priority category only.
  Priority order: Portfolio > Key Startups > AI Foundation > AI Data Center > Energy.
- **Broken links in Sheet**: Note as [link unavailable], skip from Phase 4 fetch. Work from
  headline and snippet only.
- **Slow week (< 5 items in a category)**: Include only items meeting the criteria.
  Do not pad with low-signal content.
- **Portfolio item with no article body**: Note the source and headline only. Do not
  fabricate context.
- **User selects > 5 items in Phase 4**: Ask user to prioritize to 5. Note that additional
  items can be processed in a separate session to manage token usage.

<!-- skill-version: 1.1 (generalized from firm-specific version; configure entities.md for your portfolio) -->

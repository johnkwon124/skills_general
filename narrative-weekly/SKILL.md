---
name: narrative-weekly
description: >
  Weekly CEO narrative report generator for VC professionals. Use this skill whenever the user says
  "write weekly narrative", "weekly narrative", "weekly update", "CEO report", "weekly report",
  "write weekly update", or pastes raw meeting notes asking for a report.
  Also trigger when the user mentions generating a Friday update, CEO brief, or weekly summary
  covering VC meetings, deal updates, corporate partner interactions, or portfolio/market news.
  This skill takes unstructured raw notes (from Google Keep, email threads, Slack, or memory)
  and produces a concise narrative-style report. Often used together with the `weekly-deal-context`
  skill (compact 40-word blocks); this skill is the prose layer.
---

# Weekly Narrative Report Generator

## Purpose

Converts raw weekly notes — in any format — into a polished, narrative-style CEO briefing. The output is prose-driven, implication-aware, and structured by deal/company/theme.

The user's job is to capture facts. Your job is to write the story. Even messy or abbreviated inputs yield a clean, professional output.

## Relationship to weekly-deal-context

This skill produces the **prose narrative layer**: 60–70 words per item, written in flowing sentences with implication framing.

The companion `weekly-deal-context` skill produces the **compact context layer**: ≤40 words per item, keyword-driven, used as a quick-reference brief.

When the user asks for both ("narrative + deal context"), generate two **independent markdown files** so each can stand alone. Do not merge them into one document.

## Coverage Context

Refer to **userMemories** for current portfolio companies, pipeline targets, and corporate partner details.

Primary verticals: configure to match your fund's focus (e.g., AI Infrastructure, Energy, Climate Tech, Deep Tech).

---

## Workflow

### Phase 1 — Collect Input

If the user has already pasted notes, proceed directly to Phase 2.

Otherwise, prompt:
> "Please paste your weekly notes — meeting updates, deal progress, corporate partner interactions, market signals. Any format works."

Accept any format: bullet points, shorthand, abbreviated names, mixed languages, fragmented notes.

### Phase 2 — Parse and Categorize (internal — do not output)

Mentally organize into:
1. **VC Meetings / Deal Updates** — new/ongoing deals, funding, DD progress, competitive signals
2. **Corporate Partner Meetings** — interactions with strategic/corporate partners
3. **Internal Process** — committee briefings, internal alignment
4. **Market News / Signals** — sector moves, M&A, developments worth flagging

Place ambiguous items where they add the most strategic value.

### Phase 3 — Fact-Check (mandatory, silent)

For any new deal, company, or external reference: web-search to verify funding rounds, investors, deployment numbers, founder info. Do not fabricate stats. If a figure cannot be verified, omit or flag it.

**Do not surface the search step in the output.** Never write "based on web search," "according to PitchBook/Crunchbase," "the data shows," or any phrase that reveals the sourcing process. The author would not write that. Facts appear in the prose as if they're already known.

### Phase 4 — Write the Narrative Draft

Write each item as a single block of 60–70 words. Order sections by strategic priority (most time-sensitive first).

Word count is checked **silently** (internal audit). Do not display word counts in the final output.

### Phase 5 — Review Pass (mandatory)

Before presenting the draft, run a self-review against the banned-phrasing list (see Style Rules). The review is silent — do not narrate it to the user. Strip or rewrite anything that fails:

- AI-generated tone markers (generic openers, hedge stacking, summary closers)
- Third-party / report-style framing ("the analysis suggests," "this report finds")
- Source attribution language ("based on web search," "per PitchBook")
- Evaluation adjectives that don't add information ("substantial," "compelling," "strong")

### Phase 6 — Confirm and Generate Output

Present draft in-conversation:
> "Here's the draft. Let me know if you'd like any edits — otherwise I'll save it as [markdown / DOCX]."

Default output is **markdown** (lighter, easier to chain with other skills). DOCX only when user explicitly requests it or when the output is going to a CEO/external recipient.

**Markdown filename**: `Weekly_Narrative_MMDDYYYY.md`
**DOCX filename**: `Weekly_Narrative_MMDDYYYY.docx` (Friday date)
**Save to**: workspace folder
**DOCX settings** (if used): Margins 1in, font Calibri 11pt, section headings bold 12pt, no bullet points

---

## Output Format

**Header**:
```
[Fund/Coverage Area] — Weekly Narrative Update
Week of [Month DD, YYYY] | [Author Name]
```

**Body**: One section per deal, company, or theme. **60–70 words each**, written as a single flowing paragraph (or two short paragraphs for the highest-priority item if necessary). No bullet points. **No word count display.**

**Target total length**: 4–8 items, 300–500 words total.

---

## Style Rules

### Pattern (implicit — do not label)
Every section: **[What happened]** → **[Why it matters]** → **[What's next]**

### Do
- Use "we" consistently — internal memo, not third-party report
- Name people when relevant ("Partner Chris," "CFO," "CEO Brian")
- Flag risks and time pressure explicitly ("worth resolving quickly")
- Include exact numbers when verified (revenue figures, check sizes, timelines)
- Connect new signals to existing thesis ("one more reason to push on X's timeline")
- Distinguish primary targets from market references ("not a direct target, but...")
- Stay within 60–70 words. If the item is genuinely larger, split into two items.

### Avoid (banned phrasing)

**Source attribution / search artifacts** — never appear in output:
- "Based on web search," "Per the search results," "According to PitchBook/Crunchbase/[source]"
- "The data shows," "Reports indicate," "Public filings suggest"
- Any phrase that reveals the writing was assisted by retrieval

**Third-party / report tone** — this is a memo, not an analyst report:
- "This analysis finds," "It is observed that," "The findings suggest"
- "In conclusion," "To summarize," "Overall"
- Passive constructions ("A meeting was held" → "We met")

**AI-generated tone markers**:
- Generic openers ("This week saw...," "There were several developments...")
- Bullet-style summary closers ("Key takeaways: ...")
- Hedge stacking ("It may be possible that potentially...")

**Evaluation adjectives without information content**:
- "substantial," "significant," "compelling," "promising," "strong," "interesting," "exciting"
- Replace with concrete numbers or remove entirely

**Other**:
- "investment thesis" not "투자 테제"
- "Worth noting" / "worth flagging" — usable but sparingly; not in every paragraph

### Calibrated phrases (effective VC memo voice — fine to use)
| Phrase | When to use |
|--------|-------------|
| "worth flagging" | New signal the reader should know, doesn't change direction |
| "the broader read here is" | Paragraph-level implication synthesis |
| "maps well onto" | Strategic fit between company and a partner |
| "one more reason to" | New signal reinforcing existing action direction |
| "will need internal alignment" | Internal process bottleneck, no blame |
| "we should move with some urgency" | Time-sensitive — use sparingly |
| "consistent with our [X] pacing" | Cross-referencing existing positions |
| "the angle for us is whether" | Framing the diligence question |

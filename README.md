# Claude Cowork Skills — VC Edition

A collection of Claude Cowork skills built for VC/CVC investment workflows. Each skill is a plug-in module that extends Claude's capabilities for a specific task — from deal sourcing and research to weekly reporting and job curation.

All skills are **generalized and firm-agnostic**. Configure them with your own portfolio companies, email, and career profile to make them yours.

---

## Skills

### Research & Sourcing

| Skill | What it does |
|-------|-------------|
| [market-map-builder](./market-map-builder/SKILL.md) | Sector research → company discovery → Excel market map + DOCX sector report |
| [sva-pitch-research](./sva-pitch-research/SKILL.md) | Deep tech investment research report via Cubs multi-agent workflow (Aaron + Ohtani + Soto) |
| [news-update](./news-update/SKILL.md) | Weekly VC-focused news curation across 5 sectors → DOCX digest + bilingual insights |
| [vc-news-digest](./vc-news-digest/SKILL.md) | Automated daily/evening news digest from Google Alerts → bilingual email delivery |
| [vc-monitor](./vc-monitor/SKILL.md) | Automated deal flow & portfolio news monitoring via Google Sheets + Apps Script |

### Deal Workflow

| Skill | What it does |
|-------|-------------|
| [deal-onepager](./deal-onepager/SKILL.md) | Structured company introduction one-pager (DOCX) for pipeline meetings |
| [meeting-notes](./meeting-notes/SKILL.md) | Meeting notes generator — startup first/follow-up, VC/LP, conference session (DOCX) |
| [conf-report](./conf-report/SKILL.md) | Post-conference participation report with session deep dives + meeting notes (DOCX) |

### Writing & Communication

| Skill | What it does |
|-------|-------------|
| [narrative-weekly](./narrative-weekly/SKILL.md) | Weekly CEO narrative briefing from raw meeting notes (prose, 60–70 words/item) |
| [comm-drafting](./comm-drafting/SKILL.md) | Professional email and message drafting — cold outreach, replies, introductions |
| [linkedin-cold-dm](./linkedin-cold-dm/SKILL.md) | LinkedIn cold DM drafting for deal sourcing or job search outreach |

### Career Tools

| Skill | What it does |
|-------|-------------|
| [jd-resume-tailor](./jd-resume-tailor/SKILL.md) | JD-based resume tailoring → DOCX + PDF output |
| [job-search-curator](./job-search-curator/SKILL.md) | Daily VC/CVC/Corp Dev position curation with 5-axis scoring → HTML email digest |

### Orchestration

| Skill | What it does |
|-------|-------------|
| [wilson](./wilson/SKILL.md) | Strategic PM agent — decomposes tasks, runs specialist agents, reviews outputs |
| [token-auditors](./token-auditors/SKILL.md) | Token consumption audit across skills and workflows |

---

## How to Use

These skills are designed for [Claude Cowork](https://claude.ai). To use them:

1. Install Claude Cowork (desktop app)
2. Add skills via the plugin/skill interface
3. Each skill's `SKILL.md` contains full setup instructions and configuration options

Most skills require minor configuration (email address, portfolio company list, career profile) — see each skill's `SKILL.md` for details.

---

## Structure

```
skills_general/
├── [skill-name]/
│   └── SKILL.md        ← skill definition and instructions
│   └── references/     ← supporting config files (where applicable)
```

---

## Notes

- All skills follow a **no hallucination** principle — figures require sources, estimates are labeled as such
- AI detection target: 10–20% (natural, human-quality output)
- Built and maintained by [@johnkwon124](https://github.com/johnkwon124)

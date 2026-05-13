---
name: vc-news-digest
description: |
  Automated VC-focused news curation and delivery. Collects Google Alerts from multiple sectors (VC/Funding, AI/ML, Energy/Cleantech, Data Center/Infrastructure, Semiconductors, Digital Health), curates by relevance, and sends bilingual (Korean + English) email digests. Use this skill when you need to: (1) set up or run daily morning/evening news digests, (2) curate VC deal flow and tech news, (3) generate email reports for portfolio monitoring, or (4) maintain automated news monitoring workflows. Trigger on phrases like "set up news digest", "run news digest", "VC news setup", "automate news curation", or when asked to automate news delivery workflows.
---

# VC News Digest Skill

Automated, bilingual news curation for VC professionals. This skill sets up, configures, and runs daily news digest workflows that collect Google Alerts, curate by sector relevance, and deliver dual-language emails (English + Korean) to your inbox.

## Quick Start

### What This Skill Does

1. **Collects** Google Alert emails from configured sectors
2. **Curates** by relevance (HIGH/MEDIUM flags) for VC decision-making
3. **Organizes** across 6 sectors (VC Funding, AI/ML, Energy, Data Center, Semiconductors, Digital Health)
4. **Sends** two separate emails (English and Korean versions) with identical content
5. **Runs automatically** on a schedule (morning 7 AM PT, evening 6 PM PT)

### Key Features

- **Bilingual delivery**: English and Korean emails sent separately
- **Relevance scoring**: HIGH (direct deal signal) / MEDIUM (adjacent sectors)
- **Sector organization**: 6 VC-relevant industries tracked
- **Gmail integration**: Uses gog CLI for OAuth email access
- **Flexible scheduling**: Configure for any time via cron expressions
- **Sonnet 4.6 powered**: Uses Claude Sonnet for quality curation

## Setup & Configuration

### Prerequisites

1. **Google Account Setup**
   - Ensure `gog` CLI is configured with your Gmail account
   - Run: `gog gmail messages list --account your-email@gmail.com --max 1 'from:anyone'`
   - This will prompt for OAuth authentication if needed

2. **Google Alerts**
   - Set up Google Alerts for these search terms:
     - "VC funding Series A OR Series B OR Series C"
     - "AI startups funding"
     - "energy startup" OR "cleantech" OR "ESS" OR "hydrogen"
     - "data center cooling" OR "GPU" OR "AI infrastructure"
     - "semiconductor funding" OR "chip design"
     - "digital health" OR "healthcare AI"

### Configuration Options

When setting up the skill, you can customize:

- **Email account**: The Gmail account that receives digests (`your-email@gmail.com`)
- **Schedule**: Morning and/or evening times (default: 7 AM and 6 PM PT)
- **Model**: Use `sonnet-4-6` for quality curation (default) or `haiku-4-5` for cost
- **Sectors**: Enable/disable specific sectors (default: all 6)

## Workflow: Manual Execution

If you need to run a digest on-demand (not on schedule):

```
Run the cron job manually:
cron action=run jobId=<morning-or-evening-job-id>
```

The system will:
1. Search Gmail for Google Alerts from the past 24 hours
2. Extract article titles, sources, URLs, summaries
3. Curate by sector and relevance
4. Generate English version and send
5. Generate Korean version (한글) and send

Both emails arrive within 2-3 minutes.

## Workflow: Automatic Scheduling

The skill creates two cron jobs when fully configured:

1. **Morning Digest** (7:00 AM PT)
   - Collects overnight alerts
   - Curates fresh morning reading
   - Delivers English + Korean emails

2. **Evening Digest** (6:00 PM PT)
   - Afternoon/late-day updates
   - Focuses on news not covered in morning
   - Delivers English + Korean emails

## Email Output Format

### Structure

Each email (English and Korean) includes:

```
Header: Date/Time (e.g., "Monday, March 30, 2026 — 7:00 AM PT")

By Sector (in order):
  💰 VC / Startup Funding
  🤖 AI / ML
  ⚡ Energy / Cleantech
  🏗️ Data Center / Infrastructure
  💻 Semiconductors
  🏥 Digital Health

Per Article:
  🔴 HIGH or 🟡 MEDIUM | Title
  Source: [Name]
  Summary: [1-2 sentences]
  URL: [Link]

Footer: "Curated for [Your Firm Name]"
```

### Example English Email Subject

`🌅 VC News Digest — March 30, 2026 Morning`

### Example Korean Email Subject

`🌅 VC 뉴스 다이제스트 — 3월 30, 2026 아침`

## Troubleshooting

### Emails Not Arriving

1. **Check Gmail account**: Verify `gog status` shows correct account
2. **Check cron status**: Run `cron list` to see if jobs are enabled
3. **Manual test**: Run `gog gmail send --account your-email@gmail.com --to your-email@gmail.com --subject "Test" --body "test"`
4. **Check Gmail logs**: Look for delivery issues in your Gmail spam/trash

### Missing Articles

1. **Google Alerts not active**: Verify Google Alerts are set up for your sectors
2. **Time zone**: Ensure alert times overlap with collection window (digests run at fixed PT times)
3. **Alert frequency**: Some low-volume alerts may not trigger daily

### Wrong Language

- Both versions are always sent as separate emails with language-specific subjects
- English email arrives first, Korean second (within 1-2 minutes)

## Reference Files

For detailed technical setup and advanced configuration, see:
- **references/setup.md** - Gmail & gog configuration guide
- **references/cron-jobs.md** - Cron job payload structures
- **references/sectors.md** - Sector definitions and relevance criteria

## Iterating on the Skill

Common customizations:

- **Change email recipients**: Modify cron job `to` field
- **Add/remove sectors**: Edit sector list in cron payload
- **Adjust timing**: Update cron expression (e.g., `0 9 * * *` for 9 AM)
- **Change model**: Switch between `sonnet-4-6` and `haiku-4-5` in payload

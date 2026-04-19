---
name: linkedin-cold-dm
description: >
  LinkedIn cold outreach DM drafter and sender. Use this skill whenever the user wants to draft
  and send a LinkedIn cold message, quick intro, or reach-out DM — for deal sourcing, job search,
  partnership, or any first-contact outreach. Handles the full workflow: gather context, draft two
  versions, user review, then send via LinkedIn using Chrome. Trigger on phrases like "LinkedIn DM",
  "cold DM", "cold outreach", "reach out to [name]", "LinkedIn message", "DM draft", "quick intro",
  or any request to initiate contact with someone on LinkedIn for the first time.
  ALWAYS use this skill when the user wants to send a first-touch LinkedIn message, even if phrased
  as "just drop them a note" or "can you message [name]".
---

# LinkedIn Cold DM — Draft · Review · Send

Handles the full lifecycle of a LinkedIn cold outreach message: drafting, user review, and sending via Chrome.

---

## Modes

**MODE A — Deal Sourcing / Business Development**
Goal: Initiate a first conversation with a founder, operator, or executive for investment, partnership, or business development purposes.

**MODE B — Job Search / Career Networking**
Goal: Reach out to a hiring manager, recruiter, or professional at a target firm to explore career opportunities or build a relationship.

**MODE C — General Outreach**
Goal: Any other first-contact purpose (research interview, expert network, speaker invitation, etc.). Apply the same principles as A and B.

---

## Step 1: Gather Context

If the request is clear enough, skip this step and draft immediately. Only ask if information is genuinely missing.

**Minimum required:**
- Target person's name and company/role
- LinkedIn URL or profile handle (needed for send step)
- Mode: A, B, or C
- Sender's background: name, title, organization, and the one-sentence reason for reaching out

**Helpful but optional:**
- Any specific hook: recent news, shared connection, mutual event, relevant work they've done
- Tone preference: more formal vs. more conversational

If the user has not provided their background, ask once in a single question: "Quick context: what's your name, role, and org — and what's the main reason for reaching out?"

---

## Step 2: Draft

Generate **2 versions** of the message. Each version must:

- Be **≤ 5 sentences** total
- Open with a **specific hook** — something about the recipient, their company, a recent development, or a shared context. Never open with "My name is..." or "I came across your profile."
- Introduce the sender's name, title, and organization **once**, naturally embedded mid-message
- State the **reason for reaching out** in one sentence
- Close with a **soft ask**: a 20–30 minute call "over the next few weeks" — phrased casually, not urgently
- Sound like a person wrote it — no bullet points, no headers, no "I hope this message finds you well"
- Stay within **1,200 characters** (LinkedIn DM practical limit)

Label each version clearly:

```
[Version 1 — Recommended]
Intent: ___ | Tone: ___ | Trade-off: ___

[message body]

---

[Version 2 — Alternative]
Intent: ___ | Tone: ___ | Trade-off: ___

[message body]
```

### Mode-specific framing

**MODE A — Deal Sourcing / BD:**
Lead with what's interesting about the recipient's company or work. Establish the sender's credibility through their firm and relevant focus area. Frame the ask as wanting to learn more — not pitching, not transacting. Founder-respecting tone.

Example structure:
> [Specific hook about their company/recent news/sector angle]. I'm [Name], [Title] at [Org] — we focus on [relevant domain]. [One sentence on why this company is relevant]. Would you be open to a quick call over the next few weeks?

**MODE B — Job Search / Networking:**
Lead with something specific about the recipient's firm, role, or background — not generic flattery. Establish the sender's current context briefly. Frame it as exploring and genuinely curious — not desperate or transactional. The ask is a conversation, not a referral.

Example structure:
> [Specific hook about their firm or background]. I'm [Name] — [current role and org, one line]. I'm starting to think about what's next, and [their firm/company] is somewhere I'm genuinely interested in. Would love a 20–30 minute conversation if you're open over the next few weeks.

**MODE C — General:**
Lead with the specific reason this person was worth reaching out to. State who you are and what you're working on. Make the ask proportionate to the nature of the outreach (a call, a quick question, an intro, etc.).

### Writing principles

Vary sentence length — mix short and medium sentences. Avoid: "looking forward to connecting," "synergy," "ecosystem," "passionate about." Avoid modal stacking ("I would love to potentially explore"). One ask only — no multiple options or alternatives. If the user provided a specific hook, use it in Version 1.

AI detection risk increases when sentences follow the same length pattern, transitions are overly smooth, and vocabulary is uniformly elevated. Counteract by varying rhythm and letting one sentence be notably shorter than the others.

---

## Step 3: User Review

After presenting the two versions, prompt:

> "Which version works — or any edits before I send?"

Apply requested edits and confirm once before proceeding. Do not send without explicit approval.

---

## Step 4: Send via LinkedIn

Use the Chrome MCP tools (`mcp__Claude_in_Chrome__*`) to send the approved message. Never modify the approved message text during send.

### 4a. Navigate to Profile

- If the user provided a LinkedIn URL → navigate directly
- If only name/company provided → ask: "I'll need their LinkedIn URL to send. Can you drop the link?"

```
Navigate to: https://www.linkedin.com/in/[profile-handle]/
```

### 4b. Click Message

Use `find` to locate the "Message" button on the profile page, then click it.

If the message button is not visible (e.g., only "Connect" is shown):
- Check connection degree
- If not 1st degree, inform the user: "They're not a 1st-degree connection yet — LinkedIn may block direct DM. Want me to send a connection request with a short note instead?"

### 4c. Type and Send

Use `form_input` or `javascript_tool` to enter the approved message text into the compose box, then send.

### 4d. Confirm

Take a screenshot after sending to confirm delivery. Report:
> "Sent to [Name]. Screenshot confirms delivery."

---

## Error Handling

- Chrome not accessible or LinkedIn not open → ask the user to open LinkedIn in Chrome first
- Send fails after 2 attempts → stop, report the issue, ask how to proceed. Do not retry endlessly.
- Profile URL returns 404 or access denied → report immediately
- Login wall encountered → ask user to log in and retry

---

## Notes

Keep the overall message lean. The goal is a message that reads like it was written by a thoughtful professional in 3 minutes — not drafted, polished, and redrafted. Senior people recognize over-crafted cold messages instantly. Specificity beats polish every time.

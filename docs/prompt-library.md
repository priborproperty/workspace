# Prompt library — testing the PRD in Claude before building it

**Purpose:** work your actual day through Claude + connectors for two weeks, and find out empirically which parts of the [PRD](prd-client-workspace.md) Claude already handles and which genuinely need an application with a shared UI.

**Where to run these:** Cowork, with these connectors enabled — **Atlassian (Jira)**, **Slack**, **Gmail**, **Google Calendar**, **Google Drive**.

**Real values already filled in** so these are copy-paste:

| Thing | Value |
|---|---|
| Jira site | `dbhq.atlassian.net` |
| Tempo Account field | `customfield_10030` (Account → customer + category) |
| ETB | Account `ETB` → customer **ETB** → Billable. Projects `ETB`, `ETBPCF` |
| Novuna | Account `PBF` → customer **Novuna** → Billable. ~60 `PBF*` projects |
| Other accounts | `LCFC Support` (Write off), `DB` (Admin), `DB Improvements`, `Sports` (R&D), `Trade` (Billable) |
| Slack | `digital-b.slack.com` |

**How to read the verdict column:**
🟢 expect this to work well · 🟡 expect it to half-work · 🔴 **expect this to fail — that failure is the point**

---

## A. Daily catch-up — replaces "the feed"

*PRD §1, §4 — the three-bucket hierarchy and the client page.*

**A1 — Morning triage across all three buckets** 🟢
```
Give me a morning brief organised into exactly three buckets: CLIENT WORK,
DB WORK, PERSONAL.

For each, pull from the last 18 hours:
- Slack messages in channels I'm in that mention me or need a reply
- Emails needing a response
- Jira issues assigned to me that changed status

Under CLIENT WORK, group by client (ETB, Novuna, LCFC, Trade). Put anything
you can't confidently bucket under UNASSIGNED — never guess.
Flag the three things that will cause a problem if I ignore them today.
```

**A2 — What did I miss** 🟢
```
I've been off since Friday. What happened across Slack, email and Jira that
I need to know about? Order by consequence-if-ignored, not chronologically.
Separate "needs me specifically" from "just be aware".
```

**A3 — End of day** 🟡
```
Based on my Slack activity, emails sent, Jira issues I touched, and calendar
events today — reconstruct where my time went, split by client and by
DB-internal work. Show it as rough hours and tell me your confidence in each.
```
*Tests whether reconstructed time is good enough to skip a timer. Watch the confidence — this is the crux of the whole time-tracking question.*

---

## B. The client page — replaces Slice 1

*PRD §4 — the ETB page.*

**B1 — The ETB board** 🟢
```
Show me the current state of the ETB Jira board on dbhq.atlassian.net.
Group issues by status column, newest activity first. For each: key, summary,
assignee, days since last update. Flag anything stalled more than 5 days
and anything in "01-Pending Client".
```

**B2 — One issue, in full** 🟢
```
Give me the full picture of ETB-1103: description, current status, all
comments in order with who said what, any linked documents, time logged,
and what the actual next action is and whose it is.
```

**B3 — Everything ETB, both systems** 🟢
```
Everything happening on ETB right now, pulling from both Jira and Slack:
open issues by status, Slack discussion in the last week from any ETB
channel, and any email threads with ETB people. Tell me the three things
most likely to need me this week.
```
*This is Slice 1's actual value proposition. If it feels good, the read-only client page may not be worth building.*

**B4 — Novuna across the sprawl** 🟢
```
Novuna's work spans roughly 60 Jira projects prefixed PBF*, all rolling up
to Tempo Account "PBF". Give me one consolidated view: which PBF projects
have had activity in the last 14 days, what's in flight, and who's on it.
```
*Tests whether the account-not-project insight holds up in practice.*

---

## C. Routing and classification — replaces the mapping page

*PRD §5, §6, §7.3.*

**C1 — Discover the channel map** 🟡
```
List every public Slack channel in digital-b.slack.com. For each, tell me
which bucket it belongs to — a specific client, DB internal, or personal —
and how confident you are. Where you can't tell from the name, look at
recent messages. Give me the ones you're unsure about separately.
```
*The output here is a first draft of the mapping table. Keep it.*

**C2 — Who are these people** 🟡
```
From my Slack and email over the last month, list every external person I've
interacted with. For each: their email domain, which client or partner they
belong to, and whether they're a client contact or a partner (Corfinity and
Akuvu are partners who work across several projects, not clients).
```

**C3 — The messy DM** 🔴→🟢
```
Take my DM thread with [person]. It covers more than one client. Split it
into segments by which client each part is about, with the message ranges
and dates. Where a segment is ambiguous, say so rather than guessing.
```
*This is the problem the PRD deferred as too hard to build. If Claude does it well, it should never be built — it should be a Claude call from the app.*

---

## D. Time and Tempo — where it should break

*PRD §7.5, §8.*

**D1 — Where did the time go** 🟢
```
Using Tempo accounts on dbhq.atlassian.net (the Account field is
customfield_10030, which carries a customer and a category), show me all
time logged in the last 30 days grouped by account category — Billable,
Write off, Admin, DB Improvements, R&D — and within Billable, by customer.
```

**D2 — Write-off exposure** 🟢
```
Show me all time logged to accounts in the "Write off" category in the last
quarter, by customer and by person. Which client is absorbing the most
unproductive time?
```

**D3 — The timer** 🔴
```
Start tracking my time against ETB now. Tell me when I've been on it an hour.
```
*Expected to fail outright. There is no persistent state, no clock, no ambient session. **This is the single clearest thing the app must own.***

**D4 — Team-wide** 🔴
```
For everyone in the team, show me this week's split between client work,
DB work, and unlogged time. Flag anyone whose logged hours look
significantly out of line with their working pattern.
```
*Expected to fail or be badly incomplete — it depends on data Jira doesn't hold and on a shared definition of "unlogged". Second clear thing the app must own.*

---

## E. Cross-system questions — the ones a UI can't anticipate

**E1 — Commercial context** 🟢
```
For ETB: current Jira workload, recent Slack sentiment from their channel,
and any outstanding invoices in Xero. Is this account healthy?
```

**E2 — Before a call** 🟢
```
I have a call with Tony from ETB in an hour. Brief me: what's open on their
Jira, what we last promised them and when, anything overdue, and anything
in Slack or email in the last fortnight I should know before I talk to him.
```

**E3 — Prep from documents** 🟡
```
Find the most recent ETB PRD or development plan in Google Drive, summarise
what it commits us to, and tell me which parts are now done based on Jira.
```

---

## F. The stress tests — designed to fail

Run these deliberately. **Their failure is the requirements document.**

| # | Prompt | What its failure proves |
|---|---|---|
| **F1** | `What's my current project? I'll tell you when I switch.` | No ambient state between conversations |
| **F2** | `Remember: #etb-support maps to ETB. Use that from now on.` — then start a new conversation and ask what `#etb-support` maps to | Rules don't persist or share; they need to be rows in a database |
| **F3** | `Log 45 minutes to ETB-1103 for the work I just did.` | Writing worklogs needs deliberate, auditable action — not inference |
| **F4** | Ask a colleague to run **A1** and compare their output to yours | Per-person results diverge; there's no shared truth |
| **F5** | `Show me the ETB board` twice, an hour apart | Prose isn't glanceable; a board wants to be a board |

---

## What to record

Keep a running note. For each prompt, one line:

```
A1  2026-07-28  worked / partial / failed
    - what was good:
    - what was missing:
    - would a UI have been better than prose here? yes/no
```

After two weeks, the pattern in that last question **is** the app's scope.

## Prediction

Written down now so it can be checked honestly later:

- **A, B, E will work well.** Reading, summarising and cross-system questions are Claude's strength. If so, the read-only client page in Slice 1 is largely redundant, and the PRD should shrink.
- **C will half-work** and produce a useful first draft of the mapping tables — but the rules will need to live in the app to persist and be shared.
- **D3, D4 and all of F will fail.** Persistent state, shared rules, deterministic worklogs, multi-user truth.

**If that prediction holds, the app becomes: state, rules, timers, worklogs, identity and audit — with Claude as the intelligence layer over it.** Not a Jira board reimplementation.

If it doesn't hold, that's more valuable still — and the PRD gets rewritten against evidence rather than assumption.

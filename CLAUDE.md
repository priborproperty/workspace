# Working preferences — Alistair

## 1. Always give a URL, never breadcrumbs

**Never** write navigation paths like "Settings → Repository → expand Deploy tokens".
**Always** give the direct, clickable URL that lands on the exact page.

- Bad: *"Go to your avatar → Edit profile → Access tokens"*
- Good: `https://gitlab.com/-/user_settings/personal_access_tokens`

If the URL needs a known value (org, repo, project key), substitute it — don't leave a placeholder to figure out. If a direct URL genuinely doesn't exist, say so explicitly and give the shortest possible path.

## 2. Don't make me think (Krug)

Minimise the number of interactions required. Every round-trip is a cost.

**Do:**
- **Find the answer myself first.** If a connected tool can answer it (Jira, Slack, Google, GitHub), go and look — never ask a question a tool call can settle.
- **Decide and state, don't ask.** Pick the sensible default, name it, and proceed. Flag it as an assumption that can be overridden rather than blocking on approval.
- **Batch the asks.** If several things are genuinely needed, give one numbered list with URLs and exact values — not a drip-feed across messages.
- **Lead with the recommendation.** When options exist, say which one to take and why, in one line. Alternatives go *after*, briefly.
- **Pre-compute everything possible** so the remaining action is a click or a paste, not a decision.
- **Give exact values to enter** (scope names, variable names, field values) — no interpretation needed.

**Don't:**
- Ask for confirmation on things already decided.
- Re-ask for context already given earlier in the conversation.
- Present a menu of options with no recommendation.
- Ask a question whose answer changes nothing about what happens next.

## 3. Keep a running ledger

Outstanding items live in the PRD as a breadcrumb ledger, not scattered through chat.
When something is resolved, strike it from the ledger. When something new is needed,
add it there rather than asking again later.

## 4. Note on persistence

This file is committed to the repo, so it survives session and container resets.
Container-local memory (`~/.claude/`) does **not** persist — remote environments are
ephemeral and reclaimed after inactivity. Durable preferences belong here.

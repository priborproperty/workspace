# PRD — DBWorks Client Workspace (working draft, v4)

**Status:** Draft for review · **Owner:** Alistair · **Date:** 2026-07-27
**One-liner:** A workspace at `workspace.digitalboutique.co.uk` where each **client is a Project**. Opening a client (e.g. **ETB**) shows that client's **Jira board + issues** next to its **Slack channels**. Everything that happens rolls up to a single top-down question: **is this client work, DB work, or personal?** — which is also what makes time triage possible later.

> **v4 changes:** Slice 1 trimmed to the smaller bite — **Jira read-only + Slack channels**. DMs deferred. Adds the **three-bucket hierarchy**, the **comms convention** (channels/threads over DMs), **partner** handling, and the **time-triage** end goal with its trust guardrail.
>
> **Repo caveat:** the assistant cannot read the GitLab repo (`digitalboutique/internalprojects/people`) in this session. Items tagged _[verify]_ need checking against code.

---

## 1. The organising principle — one top-down question

Every channel, thread, contact, and (later) hour resolves to exactly one bucket:

```
1. CLIENT WORK   → ETB, Navuna, …            (billable, per-client)
2. DB WORK       → Digital Boutique internal (incl. private/"special ops")
3. PERSONAL      → chit-chat, non-work
```

This hierarchy is the product. The client Project page is the first *view* of it; time triage is the eventual *payoff*. Anything that can't be bucketed automatically stays visible in an **Unassigned** lane — never silently dropped, never guessed.

---

## 2. Problem & goal

A client's reality is split across Jira and Slack, and there's no single "ETB" surface. Separately, there's no way to answer *"where is everyone's time actually going?"* — client vs internal vs personal.

**Goal (now):** a per-client page that reproduces the client's Jira board and shows the client's Slack channels.
**Goal (later):** the same routing spine answers the time question for the whole team.

**Non-goals (now):** People/HR modules; a full email client; replacing Jira as system-of-record; DM ingestion; mobile.

---

## 3. Decisions locked

| Decision | Choice |
|---|---|
| Foundation | Extend existing Laravel app; reuse Google SSO _[verify]_ |
| **Slice 1** | **Jira read-only (board + issue detail) + Slack channels for one client (ETB)** |
| Jira depth v1 | **Read-only.** Comment / transition / assign is Slice 2 |
| Slack v1 | **Channels only.** DMs deferred to a later slice |
| Routing pass 1 | Channel → client (explicit map); external contact → client (by **email domain**) |
| Partners | Corfinity, Akuvu etc. are **partners**, not clients — routed by **channel**, never by domain |
| Comms convention | Channels/threads first; DMs reserved for genuinely personal |
| Parked | People / Who's Off / Drive segregation / RBAC → §9 |

---

## 4. Slice 1 — the ETB page (BUILD FIRST)

Deliberately the smaller bite. Ship it, get traction, then iterate.

### 4.1 Scope
- **Project switcher:** ETB, Navuna, Digital Boutique, Personal.
- **Jira panel (read-only):** the ETB board rendered as status columns with issue cards; click a card → issue detail (summary, description, status, assignee, comments). _[verify: Cloud vs Server/DC; which board = ETB]_
- **Slack panel (channels only):** ETB's mapped channels, recent threads readable inline.
- **Channel→client mapping UI** + a **dry-run tester** (paste a channel name → shows which bucket/client it lands in).

### 4.2 Acceptance criteria
- Selecting ETB shows the live ETB board by status; opening an issue shows its real detail and comments.
- ETB's mapped channels appear with readable threads; no other client's traffic bleeds in.
- An unmapped channel appears in **Unassigned**, not mis-filed.
- Deploys to a Laravel Cloud preview environment.

### 4.3 Explicitly out
Jira writes, Slack DMs, email, timers, Jira worklogs, Claude co-work.

---

## 5. Comms convention — designing the mess away

Rather than build clever DM-untangling, **change where conversation lives.** The product encourages structure instead of parsing chaos.

- **Client talk belongs in the client channel** — including one-to-one talk, as a **thread inside `#etb-*`** (e.g. a thread with Jitish). Benefit beyond routing: *project comms are all in one place, and nobody has to read every thread to stay current.*
- **DMs are for personal comms.** If it matters to a project, it has a channel.
- **Private channels** are legitimate (e.g. DB-internal matters between Alistair and Gary) — these are **DB WORK**, bucket 2, with restricted membership. The hierarchy still applies; privacy changes *who can see it*, not *which bucket it is*.
- **Nudge, don't police:** when client talk is detected in a DM, offer "move this to the ETB channel?" — DM volume shrinks over time.

**Deferred:** the naming pattern `ETB-Alistair-Jitish` for group channels, and AI segmentation of cross-client DM threads. Both stay in the backlog until channels-first is proven.

---

## 6. Identifying people

| Type | How we bucket them | Confidence |
|---|---|---|
| **External client contact** (e.g. Tony @ ETB) | **Email domain → client.** Client domains are a clean, maintainable map | High |
| **Partner** (Corfinity, Akuvu) | Works across several projects → domain is *not* the signal. Flagged as partner; the **channel** they're in defines the project | Medium — needs channel discipline |
| **Internal** (DB staff) | Not client-specific; bucket comes from the **channel**, never the person | n/a |

Consequence: person-based routing only auto-fires for **external client contacts**. Everyone else is routed by *where they're talking*. This is why the channel convention (§5) matters more than clever inference.

---

## 7. The end goal — time triage (direction, not Slice 1)

Once comms are bucketed, time can be too: **how much time went to client work vs DB work vs personal.** Purpose is *reconciliation and fairness*, not surveillance.

**Guardrail (Alistair's principle, recorded deliberately):**
> If someone's **output isn't in question, their time isn't scrutinised.** Time is only examined when output raises a question. **Value is easy** — visible results end the conversation.

Design implications:
- Time data is for **triage and review context**, not routine monitoring or dashboards ranking people.
- Team-wide tracking must be **transparent to those tracked** — people know what's collected and can see their own data.
- Individual time is **not** the default lens; aggregate/project-level is.
- _[Open]_ Team-wide tracking has employment/GDPR implications (purpose limitation, transparency notice). Worth a short policy note before it ships beyond Alistair.

Sequencing: Alistair-only timer → Jira worklogs first. Team-wide only after the model is trusted.

---

## 8. Open questions

1. **Jira** — Cloud or Server/DC? Which board/project is ETB? Does one issue-key prefix map cleanly to one client?
2. **Slack** — one workspace or several? Bot token sufficient for channel reading in Slice 1? (DM/user-token questions deferred with DMs.)
3. **Client + partner domain map** — who maintains it? Does it live in the app, or come from HubSpot/Apollo?
4. **Private channels** — should DB-private channels surface in the workspace at all in v1, or stay Slack-only until membership rules are modelled?
5. **Stack/SSO** _[verify]_ — Laravel version, Livewire/Inertia, existing auth wiring.
6. **Repo home** — PRD and app code should live in a DB workspace repo, not `barra-woodview` (see §11).

---

## 9. Backlog (parked)
- Jira writes (comment / transition / assign) — **Slice 2 candidate**
- Slack DMs + `ETB-Alistair-Jitish` naming + AI thread segmentation
- Email (Gmail) ingestion into the client feed
- Timer → Jira worklogs → team-wide time triage
- Claude co-work chat
- People / Who's Off, segmented Google Drive, RBAC + dev-sandbox data segregation

---

## 10. Risks
- **Channel discipline is a people problem, not a code problem** — the design leans on it; the nudge and Unassigned lane are the mitigations.
- **Partner routing** is the weakest link (no domain signal) — depends entirely on channel mapping.
- **Time tracking can read as surveillance** — the §7 guardrail must be stated to the team, not just implied.
- **Board fidelity** — v1 renders status columns, not full Jira board parity (swimlanes, filters).

---

## 11. Repo / tooling note
- **This repo (`priborproperty/workspace`) is the home** for DBWorks specs and, later, the workspace application code. (Earlier drafts lived in `priborproperty/barra-woodview` — the Barra cottage site — where every push triggered an unrelated Cloudflare Pages rebuild. That PR should be closed unmerged.)
- **Do not connect this repo to the cottage site's Cloudflare Pages project.** If preview deploys are wanted later, wire them deliberately to a DBWorks target.
- The People/Laravel app lives in **GitLab** (`digitalboutique/internalprojects/people`). There is no GitLab connector available to Claude in this environment; options are mirroring to GitHub, or running Claude Code locally against a GitLab clone.
- **Compartmentalisation:** specs and workspace code here; the People/HR app stays separate in GitLab with its own sandbox data rules. Credentials (Jira, Slack, Google) never enter either repo.

---

## 12. Success signal (Slice 1)
Alistair opens ETB in the workspace instead of switching between Jira and Slack: the live board is there, issues open with their comments, and the ETB channels sit beside them — small enough to have shipped, useful enough to keep using.

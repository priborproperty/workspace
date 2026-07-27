# PRD — DBWorks Client Workspace (working draft, v5)

**Status:** Draft for review · **Owner:** Alistair · **Date:** 2026-07-27 · **Rev:** v5
**One-liner:** A workspace at `workspace.digitalboutique.co.uk` where each **client is a Project**. Opening a client (e.g. **ETB**) shows that client's **Jira board + issues** next to its **Slack channels**. Everything that happens rolls up to a single top-down question: **is this client work, DB work, or personal?** — which is also what makes time triage possible later.

> **v5 changes:** Confirms Jira Cloud, Google Workspace as the staff source of truth, and the private spec repo. Adds **directory sync** (§7) — pull staff from Google Workspace, users + public channels from Slack — and a **mapping page** where channels and external domains get bucketed. Notes that Jira maps **client → account** (ETB single, Navuna multi-space). Replaces open questions with an **outstanding-items ledger** (§9).
>
> **Repo caveat:** Claude cannot yet read the GitLab repo (`digitalboutique/internalprojects/people`). Items tagged _[verify]_ need checking against code once the push-mirror in §12 is live.

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
| **Jira** | **Jira Cloud** (confirmed) |
| Identity source of truth | **Google Workspace** — a Google Workspace account *is* what makes you staff |
| Spec/app repo | `priborproperty/workspace` — **private, Alistair-only** (confirmed) |
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

## 7. Directory sync & the mapping page

Rather than hand-maintaining lists, the workspace **pulls its directories** from the systems that already hold the truth, then gives one screen to map them.

### 7.1 Staff sync — from Google Workspace
- Pull **all staff from Google Workspace** (Directory API) on a schedule.
- **A Google Workspace account is what makes you staff.** Everyone here has one — no Google account would be highly unusual, and that path can be handled later rather than designed for now.
- This gives the internal-person list for free, and dovetails with the existing Google SSO.

### 7.2 Slack sync
- Pull **users** and **public channels** from Slack on a schedule.
- **Public channels are fine** to enumerate, and public channels can be added to the workspace freely — low-friction, no per-channel approval dance.
- Private channels are a later decision (§9 backlog): membership rules need modelling first.

### 7.3 The mapping page
One screen where the pulled directories get bucketed:
- List **all Slack channels** → map each to a **client / DB work / personal** bucket.
- List **external domains** seen in traffic → map to client, or flag as **partner**.
- Unmapped items sit in **Unassigned** — visible, never guessed.
- Includes the **dry-run tester** (paste a channel or address → shows where it lands).

### 7.4 Jira accounts & categories
Jira Cloud has **accounts** and **account categories**, and that's where time ultimately logs:
- **ETB** = one account, all its time logs there.
- **Navuna** = an account that spans **multiple spaces/projects**.
- So the client → Jira mapping is **client → account**, not simply client → project. A client may have several Jira projects/spaces feeding one account.
- **Action:** experiment with the real account/category structure when we reach the time-logging slice, rather than guessing the shape now.

---

## 8. The end goal — time triage (direction, not Slice 1)

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

## 9. Outstanding items — breadcrumbs

Running list of what's still needed. **Not questions to answer now** — the ledger so nothing gets lost.

### Owed by Alistair (blocking, in priority order)
| # | Item | Blocks |
|---|---|---|
| 1 | **GitLab access to the Laravel app** — via push-mirror to a private GitHub repo (see §12) | Everything code-related; resolves all `[verify]` tags |
| 2 | **ETB Jira board link** + **Jira Cloud API token** | Slice 1 Jira panel |
| 3 | **Slack app** with narrow scopes (`channels:read`, `channels:history`, `users:read`) | Slice 1 Slack panel |
| 4 | Google Workspace **Directory API** access (service account or admin consent) | Staff sync (§7.1) |

### To resolve during build (not blocking now)
- **Jira account/category structure** — confirm real shape for ETB (single) vs Navuna (multi-space); experiment when we reach time logging.
- **Client + partner domain map** — where it's maintained: in-app table, or sourced from HubSpot/Apollo.
- **Private channels** — surface in the workspace, or stay Slack-only until membership rules are modelled.
- **Slack workspace count** — one or several.
- **Laravel stack specifics** _[verify]_ — version, Livewire/Inertia, existing auth wiring.
- **Non-Google staff edge case** — deliberately deferred; handle later if it ever occurs.
- **Client-data sandbox rule** — recommended as a Slice 1 constraint (see §11 risks).

---

## 10. Backlog (parked)
- Jira writes (comment / transition / assign) — **Slice 2 candidate**
- Slack DMs + `ETB-Alistair-Jitish` naming + AI thread segmentation
- Email (Gmail) ingestion into the client feed
- Timer → Jira worklogs → team-wide time triage
- Claude co-work chat
- People / Who's Off, segmented Google Drive, RBAC + dev-sandbox data segregation

---

## 11. Risks
- **Client comms in the database are the same problem as HR data.** Once ETB's Slack and Jira content lands in a table, any developer with DB access can read client conversations. **Recommendation: apply the sandbox rule from day one** — devs build against synthetic/seeded client data; production credentials stay with Alistair. Far cheaper now than retrofitted later.
- **Channel discipline is a people problem, not a code problem** — the design leans on it; the nudge and Unassigned lane are the mitigations.
- **Partner routing** is the weakest link (no domain signal) — depends entirely on channel mapping.
- **Time tracking can read as surveillance** — the §8 guardrail must be stated to the team, not just implied.
- **Board fidelity** — v1 renders status columns, not full Jira board parity (swimlanes, filters).

---

## 12. Repo / tooling

### Repos (kept deliberately separate)
| Repo | Contents | Access |
|---|---|---|
| `priborproperty/workspace` (GitHub) | DBWorks specs + workspace app code | Private, Alistair-only |
| `digitalboutique/internalprojects/people` (GitLab) | Laravel People/HR app | DB devs, sandboxed |
| `priborproperty/barra-woodview` (GitHub) | Unrelated personal site | — |

- **Do not connect `workspace` to the cottage site's Cloudflare Pages project.** Wire preview deploys deliberately to a DBWorks target if wanted later.
- Credentials (Jira, Slack, Google) never enter any repo — environment config only.

### Getting Claude access to the GitLab app
**There is no GitLab connector in the Claude connector directory**, so there's no OAuth "authorize" flow. The chosen route is a **GitLab push mirror to a private GitHub repo**, which is one-time setup and then automatic:

1. Create a **private** GitHub repo (e.g. `priborproperty/people`).
2. Create a GitHub Personal Access Token (classic, `repo` scope) — entered **into GitLab**, never into a chat transcript.
3. In GitLab: **Settings → Repository → Mirroring repositories** → direction **Push** → URL `https://github.com/priborproperty/people.git` → username + token → **Mirror repository**.
4. Every GitLab push then syncs to GitHub, where Claude can read it via `add_repo`.

*Alternatives considered:* running Claude Code locally against a GitLab clone (works, but no remote-session access); pasting a GitLab read token into chat (rejected — puts a credential in the transcript).

---

## 13. Success signal (Slice 1)
Alistair opens ETB in the workspace instead of switching between Jira and Slack: the live board is there, issues open with their comments, and the ETB channels sit beside them — small enough to have shipped, useful enough to keep using.

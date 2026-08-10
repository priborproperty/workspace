# PRD — DBWorks Client Workspace (working draft, v12)

**Status:** Draft for review · **Owner:** Alistair · **Date:** 2026-07-27 · **Rev:** v12
**One-liner:** A workspace at `workspace.digitalboutique.co.uk` where each **client is a Project**. Opening a client (e.g. **ETB**) shows that client's **Jira board + issues** next to its **Slack channels**. Everything that happens rolls up to a single top-down question: **is this client work, DB work, or personal?** — which is also what makes time triage possible later.

> **v12 changes:** **Slice 0 replaces the read-only board as the first build** (§5): account tabs, context switching, a timer, and a plain issue list — built and run locally on Sail before deploying. The Jira board is explicitly dropped from the first slice.
>
> **v11 changes:** Adds **§3 — product owner requirements**, which reframes the product. The portal is not an aggregator but a **workflow layer with guardrails**: single front door, enforced quality gates on Jira issues, a deliberate context switcher that starts the timer and compartmentalises the view, a structured blocked/escalation signal, and calendar time-boxing that reconciles to Tempo. Slices need revisiting against this.
>
> **v10 changes:** Source control stays on **GitLab** (no mirror). Adds §14.5 — the org split protects credentials but not the deploy trigger; production auto-deploy must be off and the production branch protected.
>
> **v9 changes:** Adds the **environment & access model** (§14): two Laravel Cloud organizations — the existing `Digital Boutique` as the dev/staging boundary, and a new `DB Workspace` holding production with restricted membership. Clarifies repo locations (the Laravel app is **GitLab-only**; the GitHub repo holds specs).
>
> **v8 changes:** **The Laravel repo has been read** (§13). Foundation is Laravel 12 + **Filament 5** on Sail. Large parts of this PRD already exist: Google SSO, **Google Workspace staff sync**, a **JiraClient** contract, **Filament Shield RBAC**, and a complete **Who's Off / leave module**. Slice 1 restated against the real codebase and its conventions.
>
> **v7 changes:** **Tempo Accounts found and verified** (§8.5) — `customfield_10030` exposes Account → {customer, category}. PBF → customer **Novuna**; account `Category` (Billable / Write off / Admin / R&D) already *is* the three-bucket hierarchy. Live counter-examples prove **project ≠ client**, so the client switcher must be driven by Account/Customer, not Jira project.
>
> **v6 changes:** Jira verified directly against `dbhq.atlassian.net` via the now-live Atlassian connector — 137 projects, clients span many projects each, existing `Write Off` category, DB-internal projects already present (§8.4). Client name corrected to **Novuna**. Jira access removed from the blocking list.
>
> **v5 changes:** Confirms Jira Cloud, Google Workspace as the staff source of truth, and the private spec repo. Adds **directory sync** (§8) — pull staff from Google Workspace, users + public channels from Slack — and a **mapping page** where channels and external domains get bucketed. Notes that Jira maps **client → account** (ETB single, Navuna multi-space). Replaces open questions with an **outstanding-items ledger** (§10).
>
> **Repo access:** resolved — the GitLab repo has been cloned and read. Findings in §13.

---

## 1. The organising principle — one top-down question

Every channel, thread, contact, and (later) hour resolves to exactly one bucket:

```
1. CLIENT WORK   → ETB, Novuna, …            (billable, per-client)
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

## 3. Product owner requirements (Alistair, with Gary)

**This section reframes the product.** Earlier revisions treated the workspace as an *aggregator* — bring Jira and Slack into one place to read. That is the part Claude already does well (see the prompt library). What follows is different: the portal as a **workflow layer that enforces good practice**. Enforcement is precisely what a conversational assistant cannot do, and it is the strongest argument for building.

### 3.1 One front door

Staff sign in to the portal; the subsystems (Jira, Slack, email, Drive) are linked in behind it. The value is not convenience — it is that a single entry point is the only place where **guardrails, standards and defaults can be applied consistently**.

*Honest constraint:* "only let staff log in to the portal" is aspirational as stated — people will still have Gmail on their phones and Slack on their desktops, and short of revoking those accounts the portal cannot be the literal only door. **What is genuinely enforceable is issue creation** (see §3.2). Treat the front door as *the easiest and best-supported path*, with hard enforcement applied at the specific points that matter.

### 3.2 Quality gates — the flagship capability

**Requirement:** a Jira ticket can only be submitted through the portal, and only when it meets an agreed standard. Gary's well-formed issue becomes the template rather than the exception.

This is the clearest thing the portal can do that nothing else can: **make the good ticket the only ticket that can exist**. It gives Alistair and Gary certainty about ways of working rather than hoping standards are followed.

Mechanism sketch (draft):
- Portal owns a guided issue-creation form per issue type, with required fields, acceptance criteria, and reproduction steps enforced before submit.
- **Jira permissions restricted** so issues cannot be created directly in Jira by staff — creation flows through the portal's service account. This is what makes the gate real rather than advisory.
- Claude can assist inside the gate — draft the description, sanity-check completeness, suggest acceptance criteria — but the *gate itself* is deterministic rules, not model judgement.
- The correct **Tempo Account** (§8.5) is set at creation, so time is attributable from the start.

### 3.3 The context switcher

**Requirement:** a **dropdown**, deliberately not tabs. Options are the buckets and clients: `DB`, `Personal`, `ETB`, `Novuna`, `LCFC`, `Goodfellow`, …

The friction is the feature: switching context should be a **conscious physical act**, not an accidental drift between browser tabs.

Selecting a context does three things at once:
1. **Starts the timer** for that context (see §3.4).
2. **Filters the whole workspace** — you see only that client's email, channels and spaces.
3. Sets the default target for anything created while in that context.

The compartmentalisation is for **focus**, not security — it is a view filter. Where restriction rather than focus is required, that is RBAC and must be stated separately.

*Design risk:* the timer's accuracy depends on remembering to switch. The deliberate friction is also the failure mode. Mitigations to design: idle detection, a periodic "still on ETB?" nudge, and end-of-day reconciliation against actual Jira/Slack/email activity (which the prompt library's **A3** is testing).

### 3.4 Timer and time-boxing

**Requirement:** context selection *is* the timer. Some of the team already use **Toggl**, so a timer is an accepted behaviour to absorb rather than a new imposition.

**Calendar as the time-box.** "I'm taking this task", "working on PM", "reviewing the board" should fill out the calendar as the day proceeds — so the day is deliberately blocked out rather than reactive.

**Then Tempo.** The calendar becomes the intermediate, human-readable record that reconciles into Tempo worklogs against the right account.

Chain: `context switch → timer → calendar event → reviewed → Tempo worklog`

*Open:* retrospective correction. Estimates are wrong — an hour becomes three. The calendar must be editable after the fact, and the Tempo sync must follow the corrected version, never the optimistic original.

### 3.5 The blocked signal — removing the meerkat

**The problem in Alistair's words:** he round-robins between systems "like a hypervigilant meerkat" checking nobody is blocked. That vigilance is a tax on his attention and it does not scale.

**Requirement:** a structured way to say **"I'm blocked, I need an answer within 10 minutes"** — distinct from an ordinary message, with urgency as a first-class property rather than tone of voice.

Why it matters: it inverts the model. Instead of Alistair scanning for blockage, **blockage announces itself**. That is what lets him stop scanning and trust the system.

Draft requirements:
- Blocked state attaches to a person and, where relevant, an issue — not just a chat message.
- Severity/response window is explicit (e.g. 10 minutes, today, this week).
- It must **reach the responder where they are** — push/mobile — or it is just another inbox.
- Visible aggregate: who is blocked right now, on what, for how long.
- Resolution is recorded, so blockage becomes measurable rather than anecdotal.

### 3.6 What this means for the build

The centre of gravity moves from *reading* to *doing*:

| | Owner |
|---|---|
| Quality gates, context state, timers, calendar, worklogs, blocked signal, identity, audit | **The portal** |
| Summarising, catching up, drafting inside the gate, classifying ambiguous threads | **Claude** |

**The slices in §5 and §10 predate this section and need revisiting.** A read-only Jira board is no longer an obvious first slice — the context switcher plus timer, or the quality-gated issue form, may deliver more of what is actually wanted. That re-slicing is deliberately not done here; it should follow the prompt-library experiment.

---

## 4. Decisions locked

| Decision | Choice |
|---|---|
| Foundation | **Laravel 12 · PHP 8.4 · Filament 5 · Sail/Docker · MySQL 8.4 · Redis · Horizon** (verified, §13) |
| **Jira** | **Jira Cloud** (confirmed) |
| Identity source of truth | **Google Workspace** — a Google Workspace account *is* what makes you staff |
| Spec/app repo | `priborproperty/workspace` — **private, Alistair-only** (confirmed) |
| **First build** | **Slice 0 (§5): account tabs + context switch + timer + plain issue list, local on Sail.** The read-only board slice is superseded |
| Jira depth v1 | **Read-only.** No board in Slice 0 — a plain issue list only |
| Slack v1 | **Channels only.** DMs deferred to a later slice |
| Routing pass 1 | Jira: **issue → Tempo Account → Customer** (verified). Slack: channel → client (explicit map). Email: external contact → client (by **email domain**) |
| Partners | Corfinity, Akuvu etc. are **partners**, not clients — routed by **channel**, never by domain |
| Comms convention | Channels/threads first; DMs reserved for genuinely personal |
| Parked | People / Who's Off / Drive segregation / RBAC → §10 |

---

## 5. Slice 0 — account tabs and the timer (BUILD FIRST)

**Supersedes the earlier "read-only ETB page" slice**, which was written before §3. That slice was largely the part Claude already does well; this one is the part it structurally cannot do.

**Built and run locally on Sail first.** Laravel Cloud remains the target, and nothing here diverges from it — same app, same migrations, same code path. Push to `dev` regularly so staging does not go stale.

### 5.1 What it is

A single page with a **tab bar of Tempo accounts**. Selecting a tab switches context and starts a timer against that account.

Tabs are the real accounts (§8.5): `ETB` · `PBF (Novuna)` · `LCFC` · `Trade` · `Sports` · `DB` · `DB Improvements`.

**Tabs rather than the dropdown of §3.3.** The concern in §3.3 was accidental drift between *browser* tabs; in-app tabs are explicit. For a running timer, **always-visible current context beats deliberate friction** — a dropdown hides which account you are on until you open it. Recorded as a conscious reversal, not drift.

### 5.2 Scope

1. **Accounts** — a table seeded with the known accounts. **No sync in this slice**; the list is short and known.
2. **Tab bar** — one tab per account, current one clearly active.
3. **Context switch starts a timer** — a `TimeEntry` (account, started_at, ended_at). Switching closes the open entry and opens a new one.
4. **Per-tab issue list** — that account's open Jira issues as a **plain list, not a board**, via JQL on `customfield_10030`.
5. **Today's tally** — hours per account so far today.

### 5.3 Explicitly out

The **Jira board** (biggest build, least differentiated, Jira does it better) · Slack · email · calendar · Tempo write-back · quality gates · blocked signal · multi-user. All additive once the loop exists.

### 5.4 Why this scope

It proves the whole proposition end-to-end — **switch context → time accrues → time is attributable to a real account** — and it is precisely what failed as prompt **D3** in the [prompt library](prompt-library.md). One week of real use answers the question the PRD cannot: does deliberate context switching feel natural or like a chore? If it is a chore, that is learned cheaply, before anything larger is built on top.

### 5.5 Implementation notes

Follow the existing architecture (§13.3) and standards (§13.4) — `declare(strict_types=1)`, final classes, Pint `psr12`, PHPStan level 6, tests against in-memory SQLite.

- **Widen `JiraClient`** with one method: issues for an account. Extend the contract, implement in `HttpJiraClient`, no-op in `NullJiraClient`, so it degrades cleanly when unconfigured — the existing idiom.
- `Queries/AccountIssuesQuery` — read side.
- `Actions/Workspace/SwitchContext` — closes the open `TimeEntry`, opens the next.
- `TimeEntry` model + migration; `Account` model + seeder.
- One custom Filament **Page** for the workspace surface.

### 5.6 Acceptance criteria

- Switching tabs closes the previous time entry and opens a new one, with no gaps or overlaps.
- Each tab lists that account's open Jira issues; an unconfigured Jira degrades to an empty state rather than an error.
- Today's tally reconciles against the raw `time_entries` rows.
- `composer quality` passes; the loop is covered by feature tests.

## 6. Comms convention — designing the mess away

Rather than build clever DM-untangling, **change where conversation lives.** The product encourages structure instead of parsing chaos.

- **Client talk belongs in the client channel** — including one-to-one talk, as a **thread inside `#etb-*`** (e.g. a thread with Jitish). Benefit beyond routing: *project comms are all in one place, and nobody has to read every thread to stay current.*
- **DMs are for personal comms.** If it matters to a project, it has a channel.
- **Private channels** are legitimate (e.g. DB-internal matters between Alistair and Gary) — these are **DB WORK**, bucket 2, with restricted membership. The hierarchy still applies; privacy changes *who can see it*, not *which bucket it is*.
- **Nudge, don't police:** when client talk is detected in a DM, offer "move this to the ETB channel?" — DM volume shrinks over time.

**Deferred:** the naming pattern `ETB-Alistair-Jitish` for group channels, and AI segmentation of cross-client DM threads. Both stay in the backlog until channels-first is proven.

---

## 7. Identifying people

| Type | How we bucket them | Confidence |
|---|---|---|
| **External client contact** (e.g. Tony @ ETB) | **Email domain → client.** Client domains are a clean, maintainable map | High |
| **Partner** (Corfinity, Akuvu) | Works across several projects → domain is *not* the signal. Flagged as partner; the **channel** they're in defines the project | Medium — needs channel discipline |
| **Internal** (DB staff) | Not client-specific; bucket comes from the **channel**, never the person | n/a |

Consequence: person-based routing only auto-fires for **external client contacts**. Everyone else is routed by *where they're talking*. This is why the channel convention (§6) matters more than clever inference.

---

## 8. Directory sync & the mapping page

Rather than hand-maintaining lists, the workspace **pulls its directories** from the systems that already hold the truth, then gives one screen to map them.

### 8.1 Staff sync — from Google Workspace
- Pull **all staff from Google Workspace** (Directory API) on a schedule.
- **A Google Workspace account is what makes you staff.** Everyone here has one — no Google account would be highly unusual, and that path can be handled later rather than designed for now.
- This gives the internal-person list for free, and dovetails with the existing Google SSO.

### 8.2 Slack sync
- Pull **users** and **public channels** from Slack on a schedule.
- **Public channels are fine** to enumerate, and public channels can be added to the workspace freely — low-friction, no per-channel approval dance.
- Private channels are a later decision (§10 backlog): membership rules need modelling first.

### 8.3 The mapping page
One screen where the pulled directories get bucketed:
- List **all Slack channels** → map each to a **client / DB work / personal** bucket.
- List **external domains** seen in traffic → map to client, or flag as **partner**.
- Unmapped items sit in **Unassigned** — visible, never guessed.
- Includes the **dry-run tester** (paste a channel or address → shows where it lands).

### 8.4 Jira — verified against the live site

**Confirmed by direct inspection of `dbhq.atlassian.net` (cloudId `3fd3e2ba-24f6-47b3-924d-e7a71e121373`), 2026-07-27. Claude has read + write Jira scopes via the Atlassian connector — no API token needed.**

**Scale: 137 projects.** This is the single most important finding for the design.

**Clients span many projects each** — the client → project relationship is emphatically one-to-many:

| Client | Jira projects (sample) |
|---|---|
| **ETB** | `ETB`, `ETBPCF` (ETB - PCF), `ETBBTO` *(archived)* |
| **Novuna** | `RAID` (Novuna Raid Log) — *note the spelling: **Novuna**, not "Navuna"* |
| **PBF** | ~60 projects (`PBFAX`, `PBFCC`, `PBFDMT`, `PBFST`, …) — by far the largest |
| **Slide7** | `SPORTS`, `S7`, `S7BAS`, `S7GFBAS`, `S7RM`, `SLD7`, `SSUB`, `LCFC*` |
| **Stoneridge** | `SR`, `SRM2` |
| **Trade Radiators** | `TR`, `TRD`, `TRDES` |
| **Goodfellow** | `GDF`, `GFS` |

### 8.5 ⭐ Tempo Accounts — the real mapping layer (VERIFIED)

**This supersedes any project-based mapping.** Tempo is installed (worklogs are authored by *"Timesheets by Tempo – Jira Time Tracking"*), and the Tempo **Account** field is exposed on every issue as **`customfield_10030`** ("Account"). Its structure, read directly from live issues:

```
Issue → Account → { customer, category }
```

Example, from `ETB-1103`:
```json
"customfield_10030": {
  "value": "ETB",
  "optionProperties": {
    "key": "ETB", "id": 6, "status": "OPEN",
    "customer": { "name": "ETB", "key": "ETB", "id": 3 },
    "category": { "name": "Billable", "id": 2 }
  }
}
```

**Accounts observed (sampled, not exhaustive):**

| Account | Customer | Category |
|---|---|---|
| ETB | **ETB** | Billable |
| PBF | **Novuna** | Billable |
| LCFC | — | Billable |
| LCFC Support | — | **Write off** |
| Trade | — | Billable |
| Sports | — | R&D |
| DB | — | Admin |
| DB Improvements | — | DB Improvements |

**Three findings that change the design:**

1. **`Customer` is the client identity — not the Jira project.** `PBFIF` and `PBFST` both carry Account **PBF** → Customer **Novuna**. So the ~60 `PBF*` projects roll up to Novuna via one account, exactly as expected. **The client switcher should be driven by Tempo Customer/Account, not by Jira project.**

2. **`Category` already *is* the three-bucket hierarchy.** The org has been classifying time this way all along:
   - `Billable`, `Write off` → **CLIENT WORK** (bucket 1)
   - `Admin`, `DB Improvements`, `R&D` → **DB WORK** (bucket 2)

   This is a large de-risk: we adopt an existing, trusted vocabulary instead of inventing one, and finance already reconciles against it.

3. **Project ≠ client, empirically.** Counter-examples found in live data:
   - The `DBIMPROVE` project contains an issue booked to the **ETB** account (billable client work inside an internal project).
   - The `SPORTS` project spans **Sports** (R&D) *and* **LCFC Support** (Write off).
   - `LCFCSUB` spans **LCFC** (Billable) and **LCFC Support** (Write off).

   Any project→client mapping would have mis-filed all of these. **Account is the unit; project is not.**

**Also observed:** many issues have **no account set** (e.g. 22 of 49 sampled `LCFCSUB` issues). The **Unassigned** lane (§1) is therefore not hypothetical — it needs to exist on day one.

**Archived ≠ irrelevant:** `zarchive `-prefixed projects still carry historic time against real accounts. Exclude them from *active boards*, but never from *account roll-ups*.

**Still to verify:** the complete account list (sampling only revealed the accounts on recently-updated issues — a full sweep needs the Tempo Accounts API at `api.tempo.io`, which requires its own token), and which board within `ETB` is the active one for Slice 1.

### 8.6 Jira projects — supporting detail
- **137 projects.** Prefix families: `PBF*` (~60), `S7*`/`SLD7`/`SPORTS`, `ETB*`, `TR*`, `SR*`, `DB*`.
- **DB internal projects** — `DBADMIN`, `DBBUSINESS`, `DBHD`, `DBIMPROVE`, `DDSSO`, `CTT`, `PO`.
- `DBOOO` (DB Out of Office) and `DBSTAFF` exist — relevant when the parked People/Who's Off work resumes.
- Worklogs are readable through the **standard Jira worklog API** (confirmed on `ETB-1103`), so the future timer slice may not need the Tempo API to *write* time — to be validated.

---

## 9. The end goal — time triage (direction, not Slice 1)

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

## 10. Outstanding items — breadcrumbs

Running list of what's still needed. **Not questions to answer now** — the ledger so nothing gets lost.

### Owed by Alistair (blocking, in priority order)
| # | Item | Blocks |
|---|---|---|
| 1 | **Slack app** with narrow scopes (`channels:read`, `channels:history`, `users:read`) | Slice 1 Slack panel |
| 2 | **Jira API token** for the app itself (`JIRA_BASE_URL`, `JIRA_USER_EMAIL`, `JIRA_API_TOKEN`) — Claude has connector access, but the *app* has none | Slice 1 Jira panel going live |
| 3 | Google **service account** with domain-wide delegation | Activates the already-built staff sync |

**Resolved:** ~~GitLab repo access~~ — cloned and read (§13). ~~Jira discovery~~ — the **Atlassian connector is live** with read + write scopes on `dbhq.atlassian.net`. No API token needed. Jira structure verified directly (§8.4).

### To resolve during build (not blocking now)
- **Jira account/category structure** — confirm real shape for ETB (single) vs Navuna (multi-space); experiment when we reach time logging.
- **Client + partner domain map** — where it's maintained: in-app table, or sourced from HubSpot/Apollo.
- **Private channels** — surface in the workspace, or stay Slack-only until membership rules are modelled.
- **Slack workspace count** — one or several.
- **Laravel stack specifics** _[verify]_ — version, Livewire/Inertia, existing auth wiring.
- **Non-Google staff edge case** — deliberately deferred; handle later if it ever occurs.
- **Client-data sandbox rule** — recommended as a Slice 1 constraint (see §12 risks).

---

## 11. Backlog (parked)
- Jira writes (comment / transition / assign) — **Slice 2 candidate**
- Slack DMs + `ETB-Alistair-Jitish` naming + AI thread segmentation
- Email (Gmail) ingestion into the client feed
- Timer → Jira worklogs → team-wide time triage
- Claude co-work chat
- People / Who's Off, segmented Google Drive, RBAC + dev-sandbox data segregation

---

## 12. Risks
- **Client comms in the database are the same problem as HR data.** Once ETB's Slack and Jira content lands in a table, any developer with DB access can read client conversations. **Recommendation: apply the sandbox rule from day one** — devs build against synthetic/seeded client data; production credentials stay with Alistair. Far cheaper now than retrofitted later.
- **Channel discipline is a people problem, not a code problem** — the design leans on it; the nudge and Unassigned lane are the mitigations.
- **Partner routing** is the weakest link (no domain signal) — depends entirely on channel mapping.
- **Time tracking can read as surveillance** — the §9 guardrail must be stated to the team, not just implied.
- **Board fidelity** — v1 renders status columns, not full Jira board parity (swimlanes, filters).

---

## 13. ⭐ The existing codebase (VERIFIED — read 2026-07-27)

Cloned from `gitlab.com/digitalboutique/internalprojects/people`. **Default branch is `dev`** (not `main`); feature branches follow `DBIMPROVE-###` and merge into `dev` — the same key as the `DBIMPROVE` Jira project.

### 13.1 Stack
PHP 8.4 · Laravel 12 · **Filament 5** (panel at `/admin`) · Laravel Sail (Docker) · MySQL 8.4 · Redis · Horizon (`/horizon`) · Filament Spotlight (`Cmd/Ctrl+K`) · Socialite · **Filament Shield** (RBAC).

### 13.2 Already built — do not rebuild

| PRD item | Status in the codebase |
|---|---|
| **Google SSO** | **Built and live.** Socialite + `GoogleAuthController` + `UpsertGoogleUser`. Latest commit: *"Restrict admin panel login to Google SSO only"*. Domain-restricted via `AllowedDomain` allowlist. |
| **Staff sync from Google Workspace** (§8.1) | **Built, dormant.** `StaffDirectory` contract, `GoogleWorkspaceDirectory`, `SyncStaffFromDirectory` action, "Sync from Google" button. Needs only a domain-wide-delegated service account. |
| **RBAC** | **Built.** Filament Shield + Spatie roles (`super_admin`, `panel_user`), one `Policy` per model. |
| **Who's Off / leave** *(the "parked" People slice)* | **Built.** `WhosOffQuery`, `LeaveRequest`, `LeaveAllowance`, `LeaveDayCalculator`, `RequestLeave`/`ApproveLeave` actions, `DayPortion`/`LeaveStatus` enums, bank-holiday sync, half-days, per-person working patterns. |
| **Jira client** | **Built but narrow.** `JiraClient` contract + `HttpJiraClient`/`NullJiraClient`. Only does `createLearningSubtask()` and `issueUrl()`. Config keys already exist: `JIRA_BASE_URL`, `JIRA_USER_EMAIL`, `JIRA_API_TOKEN`. |
| **Settings layer** | **Built.** `PortalSettings` reads the `settings` table with `config/people.php` fallback; admins edit live in *Configuration → Settings*. Ideal home for the client/channel/domain maps. |
| **Slack config stub** | Partial. `config/services.php` already has `slack.notifications.bot_user_oauth_token`. |

**Correction to earlier revisions:** Who's Off was parked as future work — it is in fact **already built**. Any People work is *extension*, not greenfield.

### 13.3 Architecture — where new code goes

Business logic lives outside Filament, in single-purpose classes:

| Layer | Purpose | Where the workspace lands |
|---|---|---|
| `Actions/` | One class per write op | `Actions/Workspace/MapChannelToClient`, `…/StartTimer` |
| `Queries/` | Read-side/reporting | `Queries/ClientBoardQuery`, `ClientFeedQuery` |
| `Data/` | DTOs between layers | `Data/ClientFeedItem`, `Data/BoardColumn` |
| `Contracts/` + `Services/` | Integrations, each with a real **and** a `Null*` implementation bound in `AppServiceProvider` | `Contracts/SlackClient` + `Services/Slack/{Http,Null}SlackClient` |
| `Support/` | Framework-agnostic helpers | routing/bucketing rules |
| `Filament/` | Resources, Pages, Widgets — **thin**, delegates to Actions/Queries | the client workspace UI |
| `Policies/` | One per model, drives Shield authorization | client-scoped access |

**The null-implementation pattern is the key idiom:** bindings are chosen at boot from whether config is filled in; when credentials are absent the null implementation binds and the UI hides itself. **Slack must follow this exactly** — it means Slice 1 can merge before the Slack app exists.

### 13.4 Conventions to follow
- `declare(strict_types=1);` in every PHP file; classes `final` by default (enforced by Pint).
- Pint `psr12` + project rules; **PHPStan/Larastan level 6** over `app/`.
- Commit subjects imperative, **under 72 chars**, no trailing full stop.
- Tests: PHPUnit against **in-memory SQLite**, array cache, sync queue — the Feature suite needs no containers. `composer quality` = lint + analyse.

### 13.5 Open questions raised *by* the code

1. **Filament vs custom UI — the biggest architectural decision.** Filament 5 is an admin-CRUD framework; it excels at tables/forms and gives RBAC, auth and navigation free. But a **Jira board and a Slack feed are not CRUD**. Options: (a) custom Filament **Pages** with Livewire/Blade — stays in one app, inherits auth/RBAC, but fights the grain; (b) a separate front-end against a Laravel API — free UI hand, duplicates auth/RBAC. **Recommendation: (a)**, because Slice 1 is read-only panels and the auth/RBAC reuse is worth more than UI freedom at this stage.
2. **`JiraClient` needs widening** — from one sub-task method to boards, issue search, issue detail, and (later) worklogs + the Tempo Account field `customfield_10030` (§8.5).
3. **Deploy target is contradictory.** `docs/roadmap.md` says *"No deploy target is chosen… current preference is Render"*, but Laravel Cloud is in use (there's a *"script to import a database dump from Laravel Cloud"* commit) and was stated as the platform. **The roadmap doc is stale — confirm Laravel Cloud and correct it.**
4. **Client-data sandbox rule (§12)** — the app has no client comms today. Decide before Slack lands, not after.

---

## 14. Environments & access model (DECIDED)

Per-environment RBAC in Laravel Cloud requires Advanced RBAC (Business/Enterprise). Rather than pay for it, **organization membership is the access boundary** — non-membership is the only true "no access", since even the `Viewer` role can read environment variables.

### 14.1 Two Laravel Cloud organizations

| Org | Purpose | Members |
|---|---|---|
| **Digital Boutique** *(existing)* | **Sandbox / staging.** Devs work here against a copy. | Devs, incl. Drew (Admin today) |
| **DB Workspace** *(new)* | **Production.** | Alistair only at first; **Gary added as second Admin** once the system is mission-critical, for disaster recovery |

**Working model:** devs build and test against staging in `Digital Boutique`; Alistair pulls, commits, and promotes to production in `DB Workspace`. Devs have no path to production credentials because they are not members of that org.

This is the practical resolution of the developer-access concern that has run through this PRD: it is enforced by org boundaries rather than by roles, and costs nothing.

**Note on Gary as second Admin:** Admin can read every secret in the org — there is no break-glass-only role. This is an accepted, deliberate trade for disaster recovery, not an oversight.

### 14.2 ⚠️ The dependency this model rests on

**Staging must not contain real production data.** The whole separation collapses if a production database dump is loaded into the sandbox org — devs would then read real client communications and real HR data from staging, exactly the outcome the split exists to prevent.

A sanitised/seeded dataset is therefore **not optional** under this model; it is what makes it work. The codebase already has the raw materials (`DemoSeeder`, `people:reset`), so this is a matter of discipline and a documented rule rather than new engineering.

### 14.3 First-admin bootstrap — accepted risk

`PEOPLE_ADMIN_EMAILS` is deliberately **not** being set. The first user to sign in becomes `super_admin`; only Alistair will know the deployed URL initially and will sign in first. Risk accepted knowingly.

### 14.5 Deploy control — the gap the org split does not close

**Decision: source control stays on GitLab.** The code lives there, devs already work there, and no mirror is introduced. Both Laravel Cloud organizations connect to that same repository.

**The gap:** organization membership controls *credentials*, not the *deploy trigger*. The production environment in `DB Workspace` watches a branch in the shared repo, so a developer with push access to that branch can cause a production deploy without ever being a member of the production org or seeing its secrets.

**Two controls, both required:**

1. **Disable auto-deploy on production.** Deploys become a manual action in the Laravel Cloud dashboard, and only members of `DB Workspace` can trigger them. This is the stronger control and sits entirely with Alistair.
2. **Protect the production branch in GitLab** (*Settings → Repository → Protected branches*): `main` restricted to Maintainers for push and merge. Devs work on `dev` and `DBIMPROVE-###` branches; promotion to `main` is Alistair's.

With both in place: devs own `dev` → staging; Alistair owns `main` → production.

**Residual, accepted:** a migration merged to `main` runs against production data, with Alistair as the only reviewer in front of it. Acceptable at current team size; revisit as the team grows.

### 14.4 Open — one app or two?

**Preference is a single unified app.** The tension is that the People/HR data and the client-workspace data have different audiences and different sensitivity, which pulls toward separate permissions and possibly separate databases. It is possible People/HR later moves to its own deployment.

**Not being decided now** — the two-org split above buys time, because production is restricted regardless of how the app is eventually partitioned.

---

## 15. Repo / tooling

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

## 16. Success signal (Slice 1)
Alistair opens ETB in the workspace instead of switching between Jira and Slack: the live board is there, issues open with their comments, and the ETB channels sit beside them — small enough to have shipped, useful enough to keep using.

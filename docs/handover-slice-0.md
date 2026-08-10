# Handover pack — Slice 0

What a developer needs to pick up Slice 0 and deliver it without coming back with questions.

---

## 1. What to hand over

| # | Item | Status |
|---|---|---|
| 1 | **[Slice 0 build brief](slice-0-build-brief.md)** — scope, architecture, acceptance criteria | ✅ Written. **This is the actual ticket.** |
| 2 | **Repo access** — GitLab `digitalboutique/internalprojects/people`, branch off `dev` | Confirm they have it |
| 3 | **Local setup** — `docs/setup.md` in the People repo, plus `composer setup` and `sail artisan migrate --seed` | ✅ Already in repo |
| 4 | **Product context** — *why* this exists, trimmed (see §3 below) | Extract, don't hand the whole PRD |
| 5 | **Jira ticket** — this work should be an issue in `DBIMPROVE`, matching the branch convention | To create |
| 6 | **Deploy target + data rule** (§4) | Decide and state |
| 7 | **Definition of done + reviewer** (§5) | State explicitly |

**Recommendation: copy items 1 and 4 into the People repo's `docs/`.** The developer lives in that repo; do not make them get access to a private personal GitHub repo to read their own spec.

---

## 2. What they do *not* need

- The prompt library — that is Alistair's evaluation exercise.
- Laravel Cloud org strategy, GitLab access history, the tabs-vs-dropdown reasoning.
- PRD revision history.

---

## 3. ⚠️ What to withhold, and why

**Do not hand over the full PRD unfiltered.**

- **§9 (time triage)** describes tracking the whole team's time and using it in performance reviews — including the line about scrutinising someone's time when their output is questioned.
- **§3.5 (blocked signal)** describes monitoring who is blocked and for how long.

Both are legitimate management intentions and both are recorded honestly. But a developer reading that they are building the instrument of their own time scrutiny — discovered in a spec rather than told directly — is a trust problem, not a technical one.

**Two options, in order of preference:**

1. **Tell the team directly, first.** The PRD already argues time tracking needs a transparency note before it goes beyond one user (§9). Doing that *before* the build removes the problem entirely and is the better outcome anyway.
2. **Hand over a trimmed product-context extract** covering only what Slice 0 needs: the three-bucket hierarchy (§1), the account-not-project finding (§8.5), and the portal-as-guardrails framing (§3.1–3.3).

Slice 0 is single-user and has no team dimension, so option 2 loses the developer nothing.

---

## 4. Decisions to make before handing over

| Decision | Options | Note |
|---|---|---|
| **Jira access for the developer** | (a) No Jira credentials — build against `NullJiraClient`, which returns empty results by design; (b) a scoped Jira API token | **(a) is viable for most of the slice** and matches the compartmentalisation goal. Only the `HttpJiraClient` JQL needs real Jira, and that can be verified by Alistair. |
| **Deploy target** | Staging in the `Digital Boutique` org | **Not production.** Production lives in the `DB Workspace` org, which they are not a member of. |
| **Data rule** | Seeded/synthetic only | **No production database dump in the sandbox** — this is the rule the whole two-org split depends on (PRD §14.2). State it explicitly; do not assume it. |
| **Branch + MR target** | Branch off `dev`, MR back into `dev` | `main` is protected for production. |

---

## 5. Definition of done

- [ ] All acceptance criteria in the build brief met
- [ ] `composer quality` passes — Pint + PHPStan level 6
- [ ] Feature tests cover the timer state machine: switch, re-switch to the same account, switch with no prior entry, tally arithmetic
- [ ] Unconfigured Jira degrades to an empty state, not an error — verified by unsetting `JIRA_BASE_URL`
- [ ] `createLearningSubtask()` still works — the training integration must not regress
- [ ] MR opened against `dev`, reviewed by **Alistair**
- [ ] Commit subjects imperative, under 72 characters, no trailing full stop

---

## 6. The brief in one paragraph

> Add a workspace page to the People portal: a tab bar of Tempo accounts (ETB, PBF/Novuna, LCFC, Trade, Sports, DB, DB Improvements). Selecting a tab switches the user's active context and starts a timer against that account; switching again closes the previous entry and opens a new one, with no gaps or overlaps and never more than one open entry per user. Each tab lists that account's open Jira issues as a plain list — **not** a board — fetched via JQL on the Tempo account field `customfield_10030`. A footer shows today's hours per account. Everything else is out of scope.

---

## 7. The most likely ways this goes wrong

Worth saying out loud at handover:

1. **The timer state machine looks trivial and isn't.** Transactional close-and-open, no zero-length churn when re-selecting the current account, no double-open entries. Every hour of time data afterwards depends on this. Tests first, UI later.
2. **Scope creep into the Jira board.** It is the obvious next thing and it is explicitly excluded. Jira already does it better.
3. **Mapping client → Jira project.** Verified wrong: Novuna spans ~60 `PBF*` projects under one account. Account is the unit.
4. **Breaking the training integration** while widening `JiraClient`.

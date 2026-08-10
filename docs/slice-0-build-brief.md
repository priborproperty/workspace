# Slice 0 — build brief

**For:** a Claude Code session running locally in the People repo (`digitalboutique/internalprojects/people`), via Remote Control.
**Spec:** [`prd-client-workspace.md`](prd-client-workspace.md) §5. This brief is the implementation detail; the PRD is the why.

---

## Goal

A single page with a **tab bar of Tempo accounts**. Selecting a tab switches context and starts a timer against that account. Each tab lists that account's open Jira issues.

Run it locally on Sail. Laravel Cloud is the eventual target; nothing here diverges from it.

---

## Context you need

**Stack:** PHP 8.4, Laravel 12, Filament 5 (panel at `/admin`), Sail/Docker, MySQL 8.4, Redis, Horizon, Filament Shield.

**Branch:** default is `dev`. Feature branches are `DBIMPROVE-###`.

**Architecture — business logic lives outside Filament:**

| Layer | Purpose |
|---|---|
| `app/Actions/` | One class per write operation |
| `app/Queries/` | Read-side / reporting queries |
| `app/Data/` | DTOs between layers |
| `app/Contracts/` + `app/Services/` | Integrations — each has a real **and** a `Null*` implementation, bound in `AppServiceProvider` based on whether config is present |
| `app/Support/` | Framework-agnostic helpers |
| `app/Filament/` | Resources, Pages, Widgets — **thin**, delegates to Actions/Queries |
| `app/Policies/` | One per model, drives Shield authorization |

**Standards (enforced):**
- `declare(strict_types=1);` in every PHP file
- Classes `final` by default (Pint enforces)
- Pint `psr12` + project rules
- PHPStan/Larastan **level 6** over `app/`
- Commit subjects imperative, **under 72 chars**, no trailing full stop
- Tests: PHPUnit against in-memory SQLite (no containers needed for the Feature suite)

**Commands:**
```bash
sail up -d
sail artisan migrate
composer quality        # Pint + PHPStan — must pass before commit
composer test
```

---

## Jira facts (already verified — do not re-derive)

- Site: `dbhq.atlassian.net`
- Config keys already exist in `config/services.php`: `JIRA_BASE_URL`, `JIRA_USER_EMAIL`, `JIRA_API_TOKEN`
- **Tempo Account is `customfield_10030`.** Its shape on an issue:

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

- **Account is the client unit, not project.** A client spans many Jira projects — Novuna's ~60 `PBF*` projects all roll up to account `PBF`. Never map client→project.

**Accounts to seed:**

| Account | Customer | Category |
|---|---|---|
| ETB | ETB | Billable |
| PBF | Novuna | Billable |
| LCFC | — | Billable |
| LCFC Support | — | Write off |
| Trade | — | Billable |
| Sports | — | R&D |
| DB | — | Admin |
| DB Improvements | — | DB Improvements |

---

## What to build

### 1. `Account` model + migration + seeder
Fields: `key`, `name`, `customer`, `category`, `is_active`, `sort_order`.
Seed with the table above. **No Jira sync in this slice** — the list is short and known.

### 2. `TimeEntry` model + migration
Fields: `user_id`, `account_id`, `started_at`, `ended_at` (nullable), `source` (default `switcher`).

Index on `(user_id, ended_at)` — the open-entry lookup is the hot path.

### 3. `Actions/Workspace/SwitchContext`
The core of the slice. Given a user and an account:

- Close the user's currently open `TimeEntry` (set `ended_at = now()`)
- Open a new one for the target account
- **Do both in a transaction**
- **No gaps, no overlaps** — the new entry's `started_at` equals the closed entry's `ended_at`
- Switching to the account already active is a **no-op**, not a close-and-reopen
- A user may have **at most one** open entry at any time — enforce it, don't assume it

### 4. Widen `JiraClient`
Add **one** method to the contract:

```php
/**
 * Open issues for a Tempo account, newest activity first.
 * Returns an empty array when Jira is not configured.
 *
 * @return list<IssueSummary>
 */
public function issuesForAccount(string $accountKey, int $limit = 50): array;
```

- Implement in `HttpJiraClient` — JQL along the lines of
  `cf[10030] = "<key>" AND statusCategory != Done ORDER BY updated DESC`
- Return `[]` from `NullJiraClient`
- **Do not touch `createLearningSubtask()`** — the training integration must keep working
- Add a `Data/IssueSummary` DTO: key, summary, status, assignee name, updated

### 5. `Queries/AccountIssuesQuery`
Read side. Wraps `JiraClient::issuesForAccount()`. Cache briefly (60s) — the page will be reloaded often and Jira rate limits.

### 6. `Queries/TodaysTimeQuery`
Hours per account for the current user today, derived from `time_entries`. Count the open entry up to `now()`.

### 7. One custom Filament Page
- Tab bar of active accounts; current one clearly active
- Clicking a tab calls `SwitchContext`
- Body: that account's open issues as a **plain list — not a board**
- Footer or sidebar: today's tally per account
- When Jira is unconfigured, show an empty state, **not an error**

---

## Explicitly out of scope

Jira board · Slack · email · calendar · Tempo write-back · quality gates · blocked signal · multi-user views · account sync from Jira.

All additive once the loop works. Resist them.

---

## Acceptance criteria

- [ ] Switching tabs closes the previous entry and opens a new one — **no gaps, no overlaps**
- [ ] A user never has more than one open `TimeEntry`
- [ ] Switching to the already-active account changes nothing
- [ ] Each tab lists that account's open Jira issues
- [ ] Unconfigured Jira → empty state, not an error (verify by unsetting `JIRA_BASE_URL`)
- [ ] Today's tally reconciles against raw `time_entries` rows
- [ ] `composer quality` passes
- [ ] Feature tests cover: switch, re-switch to same, switch with no prior entry, tally arithmetic

---

## Suggested order

1. Migrations + models + seeder — confirm `sail artisan migrate --seed` works
2. `SwitchContext` + its tests — **get the state machine right before any UI**
3. Widen `JiraClient` + `NullJiraClient` + DTO
4. Queries
5. Filament Page
6. `composer quality`, then commit

Start on a branch off `dev`: `git checkout -b DBIMPROVE-xxx-workspace-slice-0`

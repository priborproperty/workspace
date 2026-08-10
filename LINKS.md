# Canonical links — Digital Boutique

**Purpose:** the single place to get a correct, direct URL when handing something to someone. Never give menu breadcrumbs — give the link from here.

If a link changes, change it here first.

---

## Naming

| Thing | Name |
|---|---|
| The product | **DB Staff Portal** |
| Git repository | `staff-portal` — **rename pending**, currently `people` |
| Jira epic | DB Staff Portal - Slice 0 |

⚠️ **Renaming the GitLab repo has consequences:** every clone URL changes, and the Laravel Cloud source connection for both environments must be re-pointed. GitLab keeps a redirect for the old path, but CI, deploy hooks and local remotes should be updated deliberately rather than left to the redirect.

---

## DB Staff Portal

| What | URL |
|---|---|
| Repository (GitLab) | https://gitlab.com/digitalboutique/internalprojects/people |
| **Staging / dev** | https://people-dev-svl0ms.laravel.cloud/admin |
| Production | *Not yet deployed — `DB Workspace` org, Alistair only* |
| Protected branches | https://gitlab.com/digitalboutique/internalprojects/people/-/settings/repository |
| Project access tokens | https://gitlab.com/digitalboutique/internalprojects/people/-/settings/access_tokens |

**Branches:** default is `dev`. Feature branches `DBIMPROVE-###`. `main` is production and protected.

**Laravel Cloud organisations:**

| Org | Purpose | Who |
|---|---|---|
| `Digital Boutique` | Dev + staging | Developers |
| `DB Workspace` | Production | Alistair (Gary later, for DR) |

---

## Jira — `dbhq.atlassian.net`

| What | URL |
|---|---|
| Site | https://dbhq.atlassian.net |
| DB Improvements project | https://dbhq.atlassian.net/browse/DBIMPROVE |
| **Epic — DB Staff Portal Slice 0** | https://dbhq.atlassian.net/browse/DBIMPROVE-490 |
| Workspace tabs + timer (Jan) | https://dbhq.atlassian.net/browse/DBIMPROVE-502 |

**Tempo:** the Account field is `customfield_10030`, carrying `customer` and `category`. **Account is the client unit, not Jira project.**

---

## Claude

| What | URL |
|---|---|
| **Connectors** (add / reconnect) | https://claude.ai/new#settings/customize-connectors |
| Claude Code sessions | https://claude.ai/code |
| Account settings / trusted devices | https://claude.ai/settings/account |
| Claude Code admin (org-wide) | https://claude.ai/admin-settings/claude-code |

**Remote Control** — run Claude Code locally, drive it from the web or phone. From a project directory:
```bash
claude remote-control
```
Docs: https://code.claude.com/docs/en/remote-control

---

## Google

| What | URL |
|---|---|
| OAuth credentials (client ID / secret) | https://console.cloud.google.com/apis/credentials |

The portal needs `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, and `GOOGLE_REDIRECT_URI` — the redirect must be the **absolute** URL `https://<domain>/auth/google/callback`, registered in the OAuth client.

---

## GitLab account-level

| What | URL |
|---|---|
| SSH keys *(preferred for cloning)* | https://gitlab.com/-/user_settings/ssh_keys |
| Personal access tokens | https://gitlab.com/-/user_settings/personal_access_tokens |

---

## Specs and planning

| What | URL |
|---|---|
| Workspace specs repo (GitHub, private) | https://github.com/priborproperty/workspace |
| Open PR — PRD and briefs | https://github.com/priborproperty/workspace/pull/1 |

---

## Onboarding a developer — hand over exactly this

1. GitLab repo — https://gitlab.com/digitalboutique/internalprojects/people *(Developer role)*
2. Their Jira ticket — e.g. https://dbhq.atlassian.net/browse/DBIMPROVE-502
3. Staging — https://people-dev-svl0ms.laravel.cloud/admin
4. Laravel Cloud — **`Digital Boutique` org only**, never `DB Workspace`
5. Local setup — `docs/setup.md` in the repo, then `composer setup` and `sail artisan migrate --seed`

**State explicitly, do not assume:** no production database dump in the development environment. Seeded and synthetic data only.

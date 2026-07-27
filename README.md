# DBWorks Workspace

Product specs and (later) application code for the Digital Boutique workspace —
an internal platform at `workspace.digitalboutique.co.uk` that organises work
around **clients as Projects**, bringing a client's Jira board and Slack
conversations onto one page.

## What's here

| Path | Purpose |
|---|---|
| `docs/` | Product requirement documents and specs |

## Organising principle

Everything resolves to one of three buckets:

1. **Client work** — ETB, Navuna, …
2. **DB work** — Digital Boutique internal
3. **Personal**

The client Project page is the first view of this; time triage is the eventual payoff.

## Current state

Spec stage. See [`docs/prd-client-workspace.md`](docs/prd-client-workspace.md)
for the active PRD and the sequenced slices.

## Conventions

- **No secrets in the repo.** Jira/Slack/Google credentials go in environment
  config, never committed.
- Specs are versioned in-place; each revision notes what changed and why.

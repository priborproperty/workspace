# Changes destined for the People repo

The People/HR Laravel app lives in GitLab at
`digitalboutique/internalprojects/people` — **not** in this repository.
Claude can read that repo but cannot push to it (GitLab access is read-only,
and there is no GitLab connector).

Files here are drafted changes for that repo, kept in version control so they
survive session and container resets. Apply them there via a normal branch and
merge request.

| File | Destination | Purpose |
|---|---|---|
| `roadmap.md` | `docs/roadmap.md` | Updated roadmap (full replacement) |
| `roadmap.patch` | — | The same change as a diff, for `git apply` |

## Applying

```bash
# in a clone of the People repo, from the default branch (dev)
git checkout -b DBIMPROVE-xxx-update-roadmap
git apply /path/to/roadmap.patch     # or copy roadmap.md over docs/roadmap.md
git commit -am "Update roadmap for Laravel Cloud and client workspace"
git push -u origin DBIMPROVE-xxx-update-roadmap
```

Note the People repo's conventions: default branch is `dev`, feature branches
are named `DBIMPROVE-###`, and commit subjects are imperative and under 72
characters with no trailing full stop.

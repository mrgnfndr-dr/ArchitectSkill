# CONTRACT — <task name>

> Committed to the base branch BEFORE the sessions start. Sessions cannot see each other;
> this file is the only shared memory. Do not change it without telling the architect.

- Base branch: `<branch>` · Date: <YYYY-MM-DD>

## Interfaces between pieces
| From piece | To piece | Interface (name / format / schema) | Notes |
|---|---|---|---|
| <piece A> | <piece B> | `<function / endpoint / file format>` | <constraints> |

Shared names, types and formats (copy exact definitions here):
```
<schema, signatures, constants>
```

## File ownership
| Piece | Owns (may edit) | Must not edit |
|---|---|---|
| <piece A> | `<paths>` | `<paths owned by others>` |
| <piece B> | `<paths>` | `<paths owned by others>` |

A file needed by two pieces is a contract change: ask the architect, do not edit it.

## Merge order
1. <piece A> — <why first>
2. <piece B> — <depends on A's interface>
3. Integrator session (spawned together with the pieces) waits for every pushed report, merges into one branch `<integration-branch>`, then the Sonnet tester runs the full gates.

MAIN: <pre-approved | ask>
<!-- pre-approved = after green gates the integrator merges and pushes to main itself,
     verifies the deploy and cleans up. Never covers money, data deletion or production
     data by hand; red gates or a contract violation turn it back into "ask". -->

## Rules for every session
- Own branch/worktree; commit to it; never merge to the main branch; never deploy.
- End with `<reports folder>/<task>/<piece>-REPORT.md` (`templates/REPORT.md`), then PUSH the branch yourself — the pushed report is the "done" signal for the integrator.

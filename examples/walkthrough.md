# Walkthrough — one fictional run

> FICTIONAL example: "add CSV export to a small web app" (`shop-web`). All names and numbers
> are made up. The architect runs on Fable 5.1 / `medium`.

## 1. Task
User: `/architect add CSV export of the orders table, with a filter by date range`

## 2. Audit summary (Step 0, first run, no marker)
A Sonnet 5 subagent measured the project and returned 1 page of numbers. The architect showed:

| file | size now | Cyrillic % | problem | proposed action | estimate after |
|---|---|---|---|---|---|
| `CLAUDE.md` | 31 KB | 55% | decision history inside rules | slim + translate | ~11 KB |
| `docs/GUIDE.md` | 40 KB | 3% | none | none | 40 KB |

Estimates are approximate. The user replied "not now". The architect recorded a project note
"token audit declined on <date>, sizes: 31 KB / 40 KB" and went on with the task.

## 3. Size the task (Step 1)
One contour (backend endpoint + one UI button), about an hour: this session plus subagents.
No parallel sessions.

## 4. Reconnaissance (Step 2)
An Explore agent was asked where orders are queried and how existing exports work. Answer in
half a page: query in `src/orders/repo.ts:41`, no existing export, UI table in
`web/OrdersPage.tsx:88`.

## 5. Plan shown to the user (Step 3)

| Piece | Files | Executor (model / effort) | Verification | Depends on |
|---|---|---|---|---|
| A. CSV endpoint `GET /orders.csv?from&to` | `src/orders/*` | Sonnet 5 / high | build + `orders.test.ts` | — |
| B. "Export CSV" button | `web/OrdersPage.tsx` | Sonnet 5 / high | build + own component test | A (URL and params) |
| C. Full gates before merge | none (read-only) | Sonnet 5 tester / high | `npm test`, base-branch comparison for failures | A, B |

Total: 2 executors in sequence (contract is one line, so no contract file), 1 tester,
architect integrates. The user confirmed: "go".

## 6. Briefs (Step 4)
Each brief followed `templates/BRIEF.md`. Piece A's brief copied in the contract:
`GET /orders.csv?from=YYYY-MM-DD&to=YYYY-MM-DD` -> `text/csv`, header row
`id,created_at,total,status`; NOT to touch `web/`; own branch `csv-export`; no deploy.
Piece B's brief copied the same URL and params.

## 7. Review (Step 6)
The architect read `git diff` for A. Defect: dates were parsed in local time, not UTC, and
the brief said UTC. Sent back to the same agent with `SendMessage`; fixed. B: clean.

## 8. Tester (Step 7)
A Sonnet 5 tester ran `npm test` (brief contained the command and the expected duration).
Report in `templates/REPORT.md` shape: 1 failing test, `pricing.test.ts`, 8 log lines; it also
fails on the base branch, so pre-existing, not caused by the change. Nothing edited by the tester.

## 9. Confirmation and merge
Architect to the user, 3 lines: what is done (endpoint + button), what was verified and by
whom (executors' self-checks, the tester's full gate, one pre-existing failure), what is left
(the failing `pricing.test.ts` is unrelated). The user said "merge". One merge of `csv-export`
into the main branch, per the project's own rules. Session closed.

# BRIEF — <piece name>

> Written in English. The executor starts cold: everything it needs is in this file.
> Planned model: <Sonnet 5 | Opus 5> · effort: <high> · branch/worktree: <name>

## 1. Goal and definition of done
- Goal: <one sentence>
- Done when:
  - <observable criterion 1>
  - <observable criterion 2>

## 2. Files
- Touch: `<path>`, `<path>`
- Do NOT touch: `<path>`, `<area>` (owned by another piece: <piece name>)

## 3. Contracts (copied in, not referenced)
<names, formats, schema, function signatures, API shapes>

## 4. Project rules that apply
- <e.g. work in your own branch/worktree; never commit to the main branch>
- <e.g. no deploy; no paid calls; no data deletion>
- <other rule copied from the project's CLAUDE.md that is relevant to this piece>

## 5. Self-check (short, not the full gate)
- Run: `<build command>` and `<own test file command>`
- Do NOT run the full gate; a separate tester does that before merge.

## 6. Report
Write `<reports folder>/<task>/<piece>-REPORT.md` following `templates/REPORT.md`
(English, ≤ 1 page): what changed with `file:line`, what was run and its result, what is
unverified, open questions. No code dumps. Commit to your branch. Do NOT merge and do NOT deploy.

---
name: architect
description: >-
  Token-saving mode for the Claude Code desktop app (Code tab; Windows and macOS) ONLY: the most
  capable model (Fable 5.1, medium effort) only plans, writes briefs, reviews diffs and integrates,
  while cheaper models (Sonnet 5, Opus 5, high effort) execute as subagents or separate sessions,
  and a dedicated Sonnet 5 tester always runs the gates before merge. On the first run in a project
  it audits the instruction files (CLAUDE.md, memory index, guides) and offers to slim and translate
  them to English. Use ONLY when the user explicitly calls it: "/architect", "architect",
  "architect mode", «архитектор», «режим архитектора», «распиши план и исполнителей». It never
  self-activates. Not for the terminal CLI, IDE extensions or claude.ai chat.
---

# architect — plan expensive, execute cheap

> **DESKTOP APP ONLY.** This skill is for the **Claude Code desktop app (Windows and macOS), the
> Code tab, only.** It is NOT for the terminal `claude` CLI, VS Code / JetBrains extensions,
> claude.ai chat in the browser, ChatGPT or any other model or product. Step 5 (parallel
> sessions) uses tools that exist only in the desktop app: `spawn_task`, `list_sessions`,
> `set_session_model`, `set_session_effort`, `get_session`, `send_message` / `SendMessage`.
> Designed and tested on Windows; macOS should behave identically, only install paths differ.
> If these tools are missing in the user's app version, fall back to subagents (`Agent` tool)
> only — no parallel sessions.

Body is in English on purpose: it is loaded into context, and English costs fewer tokens.
**Reply in the language the user converses in. Keep replies short** — output length is a cost too.
Templates live in `templates/` next to this file; read one only when you need it.

## Roles

- **Architect (this session, Fable 5.1):** understand the task, plan, write briefs, review
  diffs, run gates, integrate. Does NOT explore code or implement by itself.
- **Executors:** subagents (`Agent` tool, `model` param) or separate sessions (`spawn_task`).

Project rules (CLAUDE.md: git branches, deploy, money, data deletion) stay in force and
override this skill. Every brief must repeat the ones relevant to its piece.

## First-run intro (shown once per project, before Step 0)

If the project has no marker (see Step 0), show the user this intro FIRST, translated into the
language the user converses in. Keep it as short as below; do not add to it.

> **Architect: fewer tokens for the same work**
>
> **The problem.** When one strong model does everything (reads code, runs tests, fixes small
> things), tokens and subscription limits go to routine work that does not need that strength.
> And a big task in one long session bloats a single context.
>
> **What the skill does — three things.**
>
> **1. Splits the roles between models.** The strongest of your models only thinks: it
> analyzes the task, writes the plan, sets the assignments and checks the result by the diff.
> Cheaper models do the routine work, and a separate executor always runs the tests before the
> merge. For example: "Fable 5.1 Medium" plans, "Opus 5 High" takes the hard parts,
> "Sonnet 5 High" does the rest.
> Your set of models may differ; change it in `SKILL.md`.
>
> **2. Splits big tasks into sessions.** It sizes the task and proposes in the plan: keep it
> all in one session, move independent parts into parallel sessions, or run a chain of
> sessions when the pieces go one after another. A big job does not bloat one context.
> Nothing starts without your confirmation.
>
> **3. Audits your project and proposes how to spend less.** It looks at the instructions
> (`CLAUDE.md`, `MEMORY.md`, big guides) and shows what burns tokens in every session. It
> proposes to compress the text, translate the instructions into English, and split huge guides
> into blocks so the agent reads only what it needs. Nothing changes without your consent.
>
> **How to call it.** `/architect` followed by the task in the same message, or the words
> "architect" / «архитектор» / «режим архитектора».
>
> **Right now.** This is the first run in this project, so the audit comes first, then the plan.
>
> *Not needed for small one-sentence tasks. Works only in the Claude Code desktop app.*

## Step 0 — first-run token audit + instruction-file hygiene

Always-loaded files (`CLAUDE.md` in the project root and subfolders, the project's
`MEMORY.md` index, agent/skill files the project adds) are paid by every session and every
subagent. Cyrillic in UTF-8 costs roughly 40–50% more tokens than the same English; history
and rationale inside rules files cost even more.

**Trigger.** Look for the marker "architect token audit done <date>": a project memory note
(if the project has no memory, a small file `Research/architect/AUDIT.md` with the date and
sizes — default location; use your project's reports folder if it has one).

- **No marker → show the intro above, then run the full audit BEFORE the plan.** The marker is
  written once the audit is done or declined, so the intro is never shown twice.
- **Marker present → only the cheap size check** (bytes of the instruction files vs the sizes
  recorded in the marker). Full re-audit only if the instruction files grew > 25% since the
  recorded sizes, or the user says "token audit" / «аудит токенов».

**Delegation.** A **Sonnet 5 subagent** does the measuring. It does NOT read the files in full;
it uses any shell method suitable for the OS (PowerShell on Windows, bash/python on macOS —
do not hardcode one shell). It returns ≤ 1 page of numbers. The architect does not read the
files.

**Measure:**
- (a) every `CLAUDE.md` in the root and subfolders: bytes, line count, Cyrillic share,
  presence of history / decision dates / incident stories inside rules;
- (b) the project's `MEMORY.md` index (auto-memory): size, one line per entry or not;
- (c) guides/docs that agents must read before every task: > 100 KB or lacking an index;
- (d) advisory, without measurement: skills and MCP servers the project never uses — their
  descriptions are loaded into every session and every subagent. Only the user can disable
  them; never do it for them.

**Output to the user** (in the user's language), following `templates/AUDIT-REPORT.md`: a
table `file | size now | Cyrillic % | problem | proposed action | estimate after` plus 3–4
lines of summary. Token estimates are approximate; say so.

**Proposed actions:** (1) slim — rules stay, history moves to `docs/rules-history.md`, read on
demand; (2) translate the instruction files to English; (3) split huge guides into an index +
per-topic files so agents load only the relevant block.

**It is an OFFER, not a gate.** The user declines → continue with the task, record a project
memory "token audit declined on <date>, sizes: …" and do not raise it again unless the files
grew > 25%. Nothing is changed without the user's confirmation.

Bloat thresholds (an offer is made when any is hit): `CLAUDE.md` > 25 KB, or Cyrillic share
> 30%, or any single guide that agents must read before every task > 100 KB.

**Files absent (new project):** create them in ENGLISH by default, no question asked.
Rules only: imperative bullets and tables, concrete paths/commands/keys; no history, no
dates of decisions, no incident stories (those go to `docs/rules-history.md`, read on
demand). Budget: `CLAUDE.md` ≤ 15 KB for a new project. End the file with:
`**Language.** This file is in English to save tokens; the user speaks <their language> — reply in the user's language.`
User trigger phrases and other strings that are DATA stay in the user's language inside «».
Same defaults for any doc the task creates that agents will read routinely (guides, specs,
briefs, handoffs). Docs the USER reads (reports, research deliverables) stay in the user's
language.

**If the user accepts the slimming**, it is its own piece in the plan, done the proven way:
Opus 5 executor builds a numbered INVENTORY of every normative statement → slims →
independent cold Opus 5 auditor re-derives the rules from the original and lists LOST /
WEAKENED / DISTORTED → defects go back to the same executor → translation is LINE-FOR-LINE
(same line count, so inventory line refs stay valid) → second audit. Modality never softens;
code, paths, numbers and data strings stay verbatim. Files outside git get a backup copy
first. (First run on a real production project: 71 KB → 36 KB; the audits caught 32 defects
after slimming and 8 after translation — never skip the auditor.)

## Step 1 — size the task, pick a rung

| Task | How |
|---|---|
| Trivial, clear in one sentence, one recipe | Tell the user: open a Sonnet session directly; architect is overhead. If the user insists — one Sonnet subagent. |
| Medium, one contour, up to ~2 hours | This session + 1–4 subagents. |
| Large, has 2–3 INDEPENDENT contours (disjoint files, no waiting on each other) | Plan + contract file + 2–3 parallel sessions + integrator session at the end. |
| Large but sequential | Chain of sessions, one after another, with a handoff file. No parallelism. |

Criterion for splitting is independence of pieces, not duration.

## Step 2 — reconnaissance is delegated

Do not read code "to understand". Send an `Explore` / Sonnet agent with a precise question;
ask for conclusions in ≤ 1 page, with `file:line` references, no file dumps.

## Step 3 — the plan shown to the user (in the user's language, compact)

For each piece: what is done · files/contour · executor model + effort · how it is verified ·
dependencies. Then total: how many agents/sessions, what runs in parallel, who integrates
(with separate sessions the integrator session is always a line of the plan, plus the
main-merge question from Step 5.5).
**Wait for the user's confirmation before launching anything.**

## Model choice

Recommended setup (the author's defaults): architect **Fable 5.1 on `medium`**; executors
**Opus 5 and Sonnet 5 on `high`**.

- **Sonnet 5 (default):** running gates/tests before merge (always, see Step 7), mechanical work, recipe from the project guide, tests, docs,
  translations, search, well-specified changes.
- **Opus 5:** unknown root cause (debugging, incidents); changes touching invariants
  (dedup, DB writer, bridges, money paths, schema); pieces that cannot be specified tightly.
- **Fable executes a piece itself only** when it is at the limit of difficulty. Never by default.
- Integration / final review: Fable or Opus — small context, high cost of error.

**Customize.** Model ids and effort levels above are the author's defaults; change them to
your taste. If you have no Fable access, use Opus 5 as the architect.

## Step 4 — briefs (written in English)

Executors start cold. A brief is self-contained; fill in `templates/BRIEF.md`:

1. Goal and the definition of done.
2. Exact files to touch; files and areas NOT to touch.
3. Contracts: names, formats, schema — copied in, not referenced.
4. Project rules that apply (branch/worktree, no deploy, no paid calls, no data deletion…).
5. Self-check: the SHORT verification (build + own test file), not the full gate.
6. Report format: English, ≤ 1 page (`templates/REPORT.md`) — what changed (`file:line`),
   what was run and its result, what is unverified, open questions. No code dumps.

Few large pieces beat many small ones: every executor re-reads CLAUDE.md and docs
(tens of thousands of tokens before the first edit). Parallel subagents in one worktree
must have disjoint files; otherwise `isolation: "worktree"`.

## Step 5 — parallel sessions (large tasks)

1. BEFORE the split, fix the shared contract in a file committed to the base branch
   (`Research/architect/<task>/CONTRACT.md` — default; use your project's reports folder if
   it has one; shape: `templates/CONTRACT.md`): interfaces between pieces, who owns which
   files, merge order. Sessions cannot see each other — the file is the only shared memory.
2. Sessions are created in TWO PHASES, because a chip-spawned session inherits the app's
   model (Fable) and writing "use Sonnet" in a prompt changes nothing — the badge under the
   chat is what runs and what is billed:
   a. `spawn_task` per piece with a STUB prompt only: `[architect stub — planned model: X,
      effort: Y] Staging turn. Do NOT read files or run tools. Reply exactly "ready" and
      stop; the architect will send the brief next.` Title = the piece's name. Put the real
      brief in `Research/architect/<task>/<piece>-BRIEF.md` beforehand.
   b. The user clicks the chips and says so. Then, per session: `list_sessions` → find by
      title, check `isRunning: false` → `set_session_model` (ids: `claude-sonnet-5`,
      `claude-opus-5`) → `set_session_effort` → `get_session` to confirm `model`/`effort` took → send the real brief (`SendMessage` /
      `send_message`). Model and effort apply from the NEXT turn, so the brief must never
      travel in the chip prompt.
   If a `start_session` tool is available, prefer it over chips and set the model at once.
   Each session works in its own branch/worktree.
   **Effort is set explicitly for every spawned session.** Recommended: `high` for both
   Sonnet 5 and Opus 5, with the architect on `medium`. A spawned session inherits the
   architect's effort, so `set_session_effort` is mandatory for every session, never
   skipped, and `get_session` must show the intended effort before the brief is sent. Show
   model + effort per piece in the plan (e.g. "Sonnet 5 / high").
3. Each session ends by committing to its branch, writing
   `Research/architect/<task>/<piece>-REPORT.md` (`templates/REPORT.md`) and **PUSHING its
   branch to the remote itself** — the pushed report is the "done" signal; the user never
   pushes by hand. Its last chat line to the user: "<piece> done, branch pushed, the
   integrator will pick it up". It does NOT merge to main and does NOT deploy.
4. **Whenever separate sessions are spawned, ALWAYS spawn an integrator session in the SAME
   batch** (Opus 5 / high, fresh context) — never leave it for the user to open later.
   Its brief lists every piece's branch + report path, and its first action is ONE
   background wait command that polls the remote every 2–3 min until all reports exist
   (no tokens spent while waiting). Before waiting it tells the user in one line what it
   waits for and how often it checks; after each poll that changes state it says which
   pieces are in (`2/4: K1, K3`) — a silent integrator looks hung. A piece late by > 2× its
   planned duration → name it to the user instead of waiting forever.
   Then, with no further prompting: review each diff against the contract → merge pieces
   into one integration branch in the contract's order → Sonnet tester runs the full gates
   (Step 7) → push the integration branch.
5. **Main.** The architect asks ONCE, in the plan (Step 3): "after green gates, may the
   integrator merge into main itself — yes/no?" and writes the answer into the contract as
   `MAIN: pre-approved` or `MAIN: ask`. `pre-approved` → the integrator merges and pushes
   to main itself after green gates, verifies the deploy (project receipt/health checks)
   and cleans up branches and worktrees — no manual step is left to the user.
   `ask` → it stops with a one-screen summary and one question. Pre-approval NEVER covers
   money, data deletion, production data by hand, or anything the project's CLAUDE.md gates
   separately; red gates or a contract violation cancel it — the integrator asks.
   ONE merge, one deploy.

Sequential chain: same, but one session at a time; each ends with `HANDOFF.md`
(`templates/HANDOFF.md`: state, decisions made, next step, traps found), the next one starts
from it.

## Step 6 — review, never trust the report

After each executor: read the diff (`git diff`) and check the brief's definition of done —
tests do not catch "did something other than asked". Defect → send the same agent back
(`SendMessage`, context intact), do not fix by hand and do not spawn a new one.

## Step 7 — testing before merge: ALWAYS a Sonnet 5 agent

Gates are never run by the architect or by the model that wrote the code — test logs in an
expensive context are pure waste. Spawn a dedicated **Sonnet 5 tester** (background; the
project's gate commands and durations copied into its brief). It only runs and reports, in
the shape of `templates/REPORT.md`: pass/fail per gate, names of failing tests, the 5–10
relevant log lines per failure, whether the failure also reproduces on the base branch
(pre-existing vs caused by the change). It fixes NOTHING and edits no files.

- Failure caused by the change → back to the executor that wrote it (`SendMessage`).
- Failure with unclear cause → Opus 5 agent to diagnose.
- After a fix, the same tester re-runs (`SendMessage`), not a new one.

Show the tester as a separate line in the plan (Step 3). Report to the user: what is
done, what was verified and by what, what is left.

## Keeping the architect cheap

- One architect session = one task. Task done → session closed.
- Long-running pieces go to background agents; the desktop app must stay open (background
  agents die with it).
- Do not paste executor reports back to the user verbatim — two or three lines.

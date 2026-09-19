# ArchitectSkill

**architect** — a Claude Code skill: plan expensive, execute cheap.

English · [Русский](README.ru.md)

> ## DESKTOP APP ONLY
> This skill is for the **Claude Code desktop app (Windows and macOS), the Code tab, ONLY.**
> It is **NOT** for the terminal `claude` CLI, VS Code / JetBrains extensions, claude.ai chat
> in the browser, ChatGPT or any other model or product.
>
> Why: the parallel-sessions phase (Step 5) uses tools that exist only in the desktop app:
> `spawn_task`, `list_sessions`, `set_session_model`, `set_session_effort`, `get_session`,
> `send_message` / `SendMessage`.
>
> Designed and tested on Windows. macOS should behave identically; only the install paths
> differ.

## What it does and why

**The problem.** When one strong model does everything (reads code, runs tests, fixes small
things), tokens and subscription limits go to routine work that does not need that strength.
And a big task in one long session bloats a single context.

**Three things the skill does:**

1. **Splits the roles between models.** The strongest of your models only thinks: it analyzes
   the task, writes the plan, sets the assignments and checks the result by the diff. Cheaper
   models do the routine work; a separate executor always runs the tests before the merge.
2. **Splits big tasks into sessions.** It sizes the task and proposes in the plan: keep it all
   in one session, move independent parts into parallel sessions, or run a chain of sessions
   when the pieces go one after another. Nothing starts without your confirmation.
3. **Audits your project and proposes how to spend less.** It looks at your instructions
   (`CLAUDE.md`, `MEMORY.md`, big guides), shows what burns tokens in every session, and
   proposes to compress the text, translate the instructions into English, and split huge
   guides into blocks so the agent reads only what it needs. Nothing changes without your
   consent.

The models below are the author's example setup; use whatever models you have.

| Role | Model / effort | Job |
|---|---|---|
| Architect | Fable 5.1, `medium` | Plans, writes briefs, reviews diffs, integrates. Does not explore code or implement by itself. |
| Executor (hard pieces) | Opus 5, `high` | Unknown root causes, invariants, pieces that cannot be specified tightly. |
| Executor (default) | Sonnet 5, `high` | Mechanical work, recipes, tests, docs, translations, well-specified changes. |
| Tester | Sonnet 5, `high` | Always runs the gates before merge. Only runs and reports; fixes nothing. |

The skill is invoked **only when you call it explicitly**. It never self-activates.

## Requirements

- Claude Code **desktop app**, Windows or macOS, the **Code tab**.
- Access to the models in the table (see "Customize" below if you lack one).
- `git`, or just a ZIP download.

## Install

The repository root is the skill folder. Clone it into your skills directory as `architect`.

Windows (PowerShell):

```powershell
git clone https://github.com/mrgnfndr-dr/ArchitectSkill "$env:USERPROFILE\.claude\skills\architect"
```

macOS:

```bash
git clone https://github.com/mrgnfndr-dr/ArchitectSkill ~/.claude/skills/architect
```

Or download the ZIP from GitHub and copy the folder to `<you>\.claude\skills\architect`
(Windows) or `~/.claude/skills/architect` (macOS), so that `SKILL.md` sits directly inside it.

Restart or reload the app if the skill is not listed.

## How to call it

In the Code tab:

```
/architect
```

Also recognized: "architect", "architect mode", «архитектор», «режим архитектора»,
«распиши план и исполнителей». The skill replies in the language you write in.

## What happens on the first run in a project

Before planning, `architect` looks for a marker "architect token audit done <date>" (a project
memory note, or `Research/architect/AUDIT.md` if the project has no memory).

- **No marker:** a Sonnet 5 subagent measures your always-loaded instruction files without
  reading them in full: every `CLAUDE.md`, the `MEMORY.md` index, and the guides agents must
  read before each task. It also flags skills and MCP servers the project never uses (advisory;
  only you can disable them).
- You get one table: `file | size now | Cyrillic % | problem | proposed action | estimate after`,
  plus 3-4 lines of summary (see `templates/AUDIT-REPORT.md`). Token estimates are approximate;
  Cyrillic in UTF-8 costs roughly 40-50% more tokens than the same English.
- Proposed actions: slim (rules stay, history moves to `docs/rules-history.md`), translate the
  instruction files to English, split huge guides into an index plus per-topic files.
- **It is an offer, not a gate.** Decline and the task continues; the decline is recorded and
  not raised again unless the files grew more than 25%. Nothing is changed without your
  confirmation. If you accept, slimming is done with an independent audit after each stage.
- Marker present: only a cheap size check; a full re-audit if the files grew more than 25%, or
  you ask for a "token audit".

## Models: customize

The recommended setup is architect Fable 5.1 on `medium`, executors Opus 5 and Sonnet 5 on
`high`. Model ids and effort levels are the author's defaults: change them in `SKILL.md` to your
taste. If you have no Fable access, use Opus 5 as the architect.

## When NOT to use it

Trivial tasks that fit in one sentence and one recipe: open a Sonnet session directly. The
architect is overhead there, and the skill itself will tell you so.

## The steps

0. **First-run token audit** and instruction-file hygiene (above).
1. **Size the task** and pick a rung: one subagent, one contour with a few subagents, parallel
   sessions for independent contours, or a sequential chain with a handoff file.
2. **Reconnaissance is delegated** to an Explore/Sonnet agent; conclusions in one page.
3. **The plan** for your confirmation: piece, files, executor model + effort, verification,
   dependencies. Nothing launches before you confirm.
4. **Briefs** (`templates/BRIEF.md`): self-contained, six points, English.
5. **Parallel sessions** for large tasks: a contract file first (`templates/CONTRACT.md`), then
   two-phase session creation so the model and effort really apply, then an integrator. If the
   session tools are missing in your app version, the skill falls back to subagents only.
   Sequential chains use `templates/HANDOFF.md`.
6. **Review**: read the diff, never trust the report; defects go back to the same executor.
7. **Testing before merge**: always a dedicated Sonnet 5 tester (`templates/REPORT.md` shape);
   then one merge, and only with your say-so.

A full fictional run is in `examples/walkthrough.md`.

## Project rules always win

Your project's `CLAUDE.md` (git branches, deploy, money, data deletion) stays in force and
overrides this skill. Every brief repeats the rules relevant to its piece.

## Layout

```
SKILL.md          the skill itself
templates/        BRIEF, CONTRACT, HANDOFF, REPORT, AUDIT-REPORT
examples/         walkthrough.md
```

## License

MIT, see [LICENSE](LICENSE).

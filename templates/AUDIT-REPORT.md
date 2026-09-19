# AUDIT-REPORT — token audit overview (template)

> Shown to the user in the user's language. **The numbers below are FICTIONAL** (a made-up
> "shop-backend" project) — replace them with the measurements returned by the Sonnet 5
> measuring subagent. Token estimates are approximate; Cyrillic in UTF-8 costs roughly
> 40–50% more tokens than the same English.

Project: `shop-backend` (FICTIONAL) · Date: <YYYY-MM-DD>

| file | size now | Cyrillic % | problem | proposed action | estimate after |
|---|---|---|---|---|---|
| `CLAUDE.md` (root) | 58 KB, 640 lines | 62% | history and incident stories inside rules | slim + translate to English | ~18 KB (-69%) |
| `api/CLAUDE.md` | 9 KB, 110 lines | 0% | none | none | 9 KB |
| `MEMORY.md` (auto-memory index) | 14 KB, 96 lines | 41% | several entries span 3+ lines | one line per entry, translate | ~7 KB |
| `docs/DEV-GUIDE.md` | 140 KB | 5% | > 100 KB, no index | split into index + per-topic files | ~6 KB index; agents load ~10 KB per task |

**Unused skills / MCP servers (advisory, not measured):** the project never uses <skill A>,
<MCP server B>; their descriptions are loaded into every session and every subagent. Only
you can disable them.

**Summary**
- Every session and subagent pays for ~80 KB of always-loaded text; about 45% of it is Cyrillic.
- Slimming plus translation would cut this to roughly 34 KB (approximate).
- Proposed order: slim `CLAUDE.md` -> translate -> split the guide. Independent audits follow each step.
- This is an offer: decline and the task continues unchanged; nothing is edited without your confirmation.

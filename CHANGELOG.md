# Changelog

## 3.2.3 - 2026-11-26

- Corrected the `lean-worker` description in `README.md`, `SETUP.md`, `examples/CLAUDE.md.snippet` and the token-lean `SKILL.md`. They said it had read-only tools. It has Read, Glob, Grep and Bash, and Bash can change files; it has no Edit or Write tool.
- `SKILL.md` and `plugin.json` carry 3.2.3; rebuilt `dist/token-lean.zip`.

## 3.2.2 - 2026-11-24

- Removed `docs/LIMITATIONS.md` and every link to it. `memory-map` 1.1.1 now carries its secrets-search command inline instead of pointing to that page.
- Rebuilt `dist/token-lean.zip` and `dist/memory-map.zip`.

## 3.2.1 - 2026-11-24

- New `SETUP.md`: a first-time setup guide with one copy-paste prompt for Claude Code (install the plugin, add the core rules to `CLAUDE.md`, set `effortLevel` to `low`, copy the `lean-worker` agent, check the start model), and a table of why each step is a setting or file rather than an always-loaded rule.
- Default effort is documented as a setting, not a rule: in a 30-run test, low effort matched high effort on accuracy and cost about 3 percent less per run. Thinking-heavy tasks and a comparison with the model default were not tested.
- README links the guide; `SKILL.md` and `plugin.json` carry 3.2.1; rebuilt `dist/token-lean.zip`.

## 3.2.0 - 2026-11-24

- New agent `lean-worker` (`trimtab:lean-worker` in the plugin; file at `plugins/trimtab/agents/lean-worker.md`): haiku with Read, Glob, Grep and Bash only. The rules now send delegated mechanical steps there and never to a general-purpose agent, which carries every tool. In one end-to-end test the worker cost about $0.017 against about $0.57 for the default agent on the same step.
- Cost rules: batch reads (over 5 files, one Grep or Bash call or parallel Reads, never one Read per turn); keep a haiku worker under 100K tokens because Haiku 5.5 costs 5x more above that; `/clear` is no longer called "free" (the next prompt rebuilds the cache).
- Plan-aware model rules for Free, Pro and Max: the start model and any planner above it are named by tier (mid, strongest, cheapest), not by model. If a tier is not on the plan, Claude says so once and uses the next one down.
- Docs-search rule capped at one search or page read, skipped when a dated answer under 30 days old is already in the session or memory.
- Advisor rules shortened: tags once per group of claims with the same confidence, none on what is plainly in the user's message or tool output; the pre-send check is removed.
- Price note corrected: Haiku 5.5 is $0.10/$0.50 per million tokens only under 100K prompt tokens; "the cheapest tier is the one large saving" is replaced with the measured figure (about 25 percent per step).
- Rebuilt `dist/token-lean.zip`.

## 3.1.0 - 2026-11-23

- Fixed a conflict between two rules: "never open with agreement; lead with a challenge" versus reply styles that say "lead with the result". Under a concise or brief style the result now comes first, and a challenge follows only if it would change what the user does. The core rules in `SKILL.md` and `examples/CLAUDE.md.snippet` both carry the change.
- Fixed the routing text: a step that fails the mechanical test's "no taste or judgement" check goes to no worker, whatever its size. Before, the large-step rule skipped that check, which contradicted "judgement work is never given to a weaker model".
- Made the large-step rule explicit: a step that passes all three tests but reads or moves more than 30K tokens goes to the middle tier (`sonnet`) rather than the cheapest one, which matches "`sonnet` for large ones" in the CLAUDE.md snippet. In a test, the 3.0.0 wording sent such a step to `haiku`.
- `memory-map` 1.1.0: before the first write it now asks the user where to keep the memory folder (default `~/.claude/memory/`) and waits for the answer, instead of choosing a folder itself.
- Rebuilt `dist/token-lean.zip` and `dist/memory-map.zip` from the current skills (the token-lean zip was one edit behind 3.0.0).

## 3.0.0 - 2026-11-19

- Renamed the project to **trimtab**. One plugin, `trimtab`, now holds both skills: `token-lean` (`/trimtab:token-lean`) and `memory-map` (`/trimtab:memory-map`). The marketplace is `tegaseries`.
- Breaking for plugin users: uninstall `token-lean@token-lean` and `token-lean-memory@token-lean`, then run `/plugin marketplace add tegaseries/trimtab` and `/plugin install trimtab@tegaseries`. Folder copies and claude.ai uploads keep the name `/token-lean`.
- Added a plugin icon and a plugin README.
- README covers every place the skills run. On claude.ai, the desktop app and Cowork, paid plans add this repo as a plugin marketplace (one install that also syncs to Claude Code); free plans upload the skill ZIP. Added "Check it works" and "Update" steps.
- Upload files are now `dist/*.zip` (the help center asks for a ZIP); the old `dist/*.skill` files are removed.
- The README header is now a wordmark (Archivo ExtraBold Italic, with light and dark versions in `assets/`) in place of the icon.
- Confidence tags are now required on every claim the user may act on, with a definition for each tag and a check before each reply. In the 2.0.1 test the rules reached the model but it used no tags.
- Model self-check: Claude checks which model it is running as and says once if that model is clearly wrong for the task. Workers are named by alias (`haiku`, `sonnet`, `opus`) so they always get the newest model, and a tier that is out of usage drops one level with a one-line note. A skill cannot see which models an account has, so it does not try.
- Search first: questions about things that change (Claude's own features, libraries, APIs, prices) now get a docs search before the answer, not a disclaimer after it. In a test, Claude answered a "how do skills work" question from memory and missed uploading skills to claude.ai.

## 2.0.1 - 2026-11-19

- The core rules now sit at the top of `SKILL.md`; `core.md` is removed. In a test, Opus loaded the skill but did not follow its "read `core.md` first" line, so the rules never applied.

## 2.0.0 - 2026-11-19

- New `core.md` beside the skill: the always-on rules. The skill reads it first when the rules are not already in CLAUDE.md, so typing `/token-lean` at session start is enough.
- Quality first: a check before "done", a write-less ladder, and a spec with plan approval for big or vague tasks.
- Routing reworked: the strongest model plans and judges; only steps that pass the mechanical test go to cheap workers, each with exact steps, a check and low effort; one retry, then the main model.
- `/clear` preferred over `/compact` between tasks; never switch models mid-session.
- `examples/CLAUDE.md.snippet` now matches `core.md` (about 650 tokens).

## 1.0.0 - 2026-11-19

First public release.

- `token-lean` skill: advisor rules, confidence tags, a tone dial, three-tier model routing with a mechanical test and a delegation threshold, context hygiene, session handoff, lean replies.
- `memory-map` skill (plugin `token-lean-memory`): a root index, topic hubs, notes and work logs in plain markdown, with starter templates.
- `examples/CLAUDE.md.snippet`: optional always-on rules.
- `dist/*.skill`: ready-made files for upload to claude.ai.
- Markdown only: no scripts, hooks or settings changes, so it runs the same on Windows, macOS and Linux.

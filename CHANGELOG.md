# Changelog

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

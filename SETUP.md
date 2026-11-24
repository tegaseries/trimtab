# First-time setup

One prompt sets trimtab up in Claude Code, so the rules do not have to live in always-loaded text. Settings and agent files are applied once. Rules in `CLAUDE.md` are paid for in every session.

## Copy this prompt (Claude Code)

Paste it into a new Claude Code session. Claude tells you before each write and backs up every file it changes.

```
Set up trimtab for me on this computer. Do the steps in order. Tell me before each file write, and copy any file you change to <name>.bak-<date> first.

1. Plugin. You cannot run slash commands, so ask me to run `/plugin install trimtab --marketplace tegaseries/trimtab` (on older Claude Code: `/plugin marketplace add tegaseries/trimtab`, then `/plugin install trimtab@tegaseries`). Wait until I say it is done.
2. Always-on rules. Read https://raw.githubusercontent.com/tegaseries/trimtab/main/examples/CLAUDE.md.snippet. Put the block from <!-- token-lean:begin --> to <!-- token-lean:end --> into ~/.claude/CLAUDE.md. If those markers are already there, replace only what is between them. Keep everything else in the file.
3. Default effort. In ~/.claude/settings.json set the top-level key "effortLevel" to "low". Keep every other key.
4. Worker agent. If the plugin is installed, skip this: it already provides `trimtab:lean-worker`. Otherwise copy https://raw.githubusercontent.com/tegaseries/trimtab/main/plugins/trimtab/agents/lean-worker.md to ~/.claude/agents/lean-worker.md.
5. Start model. Tell me which model this session runs as. Then tell me which model my plan offers for routine work (mid tier) and for planning and debugging (strongest tier). Do not change the model: I choose it with /model.
6. Check. List each file you changed and the lines you added. Say what you did not check.
```

On Windows, `~` is your user folder (for example `C:\Users\<you>`).

## What each step does

| Step | What it does | Why it is a step, not a rule |
|---|---|---|
| Plugin | Installs the `token-lean` and `memory-map` skills and the `lean-worker` agent | The skill text loads only when needed |
| Always-on rules | Adds about 1,000 tokens of core rules to every session | This is the one part that must stay in `CLAUDE.md` |
| Default effort `low` | Starts every session at low effort | A setting costs no tokens; a rule would |
| Worker agent | A haiku worker with read-only tools for mechanical steps | An agent file is only read when it is used |
| Start model | You pick the model for the session | A session cannot change its own model |

## About default effort

- In a test (Sonnet 5.5, 30 headless runs, five small tasks), low effort matched high effort on accuracy (12 of 12 checked tasks passed in both) and cost about 3 percent less per run, with about 19 percent fewer output tokens. The saving is small because the fixed setup of every run dominates a small task. Directional only; see [docs/LIMITATIONS.md](docs/LIMITATIONS.md).
- Not tested: thinking-heavy tasks, where low effort may do worse, and comparison with the model's own default (`medium` for Haiku, Sonnet and Opus 5.5 according to the Claude Code docs).
- Raise it when the work needs judgement: `/effort high` for planning, debugging or design, and back to `low` afterwards. `/effort` overrides the setting for the session, and the `CLAUDE_CODE_EFFORT_LEVEL` environment variable overrides both. The settings key accepts `low`, `medium`, `high` and `xhigh`.
- Remove the `"effortLevel"` line from `~/.claude/settings.json` to go back to the model's default.

## Other places

- **claude.ai, desktop app, Cowork:** there is no `settings.json`. Use the steps in the [README](README.md#quick-start) (add the marketplace, or upload the ZIP on a free plan) and pick the model and effort in the model menu if your plan shows them.
- **Copied folder, not the plugin:** copy `plugins/trimtab/agents/lean-worker.md` to `~/.claude/agents/` (step 4 does this).

## Check it worked

Start a new session and ask something you would act on, such as "Should I use SQLite or Postgres for a small web app?". A working setup leads with the answer and tags its claims `[Certain]`, `[Likely]` or `[Guessing]`. Run `/effort` to see the level.

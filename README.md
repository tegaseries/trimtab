<h1 align="center">
  <br>
  <a href="https://github.com/tegaseries/trimtab">
    <img src="plugins/trimtab/.claude-plugin/icon.png" alt="trimtab" width="160">
  </a>
  <br>
  trimtab
  <br>
</h1>

<h4 align="center">Wasted tokens come from redone work, re-read context and padded replies. trimtab steers Claude away from all three with a few small rules.</h4>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-2323FF.svg" alt="License: MIT"></a>
  <a href="CHANGELOG.md"><img src="https://img.shields.io/badge/version-3.0.0-2323FF.svg" alt="Version 3.0.0"></a>
  <a href="#claude-code"><img src="https://img.shields.io/badge/Claude%20Code-plugin-2323FF.svg" alt="Claude Code plugin"></a>
  <a href="#claudeai-desktop-app-and-cowork"><img src="https://img.shields.io/badge/claude.ai%20%C2%B7%20desktop%20%C2%B7%20Cowork-skill-2323FF.svg" alt="claude.ai, desktop and Cowork skill"></a>
  <a href="#how-it-works"><img src="https://img.shields.io/badge/markdown-only-2323FF.svg" alt="Markdown only"></a>
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> •
  <a href="#what-you-get">What You Get</a> •
  <a href="#how-it-works">How It Works</a> •
  <a href="#configuration">Configuration</a> •
  <a href="#known-limits">Known Limits</a> •
  <a href="#license">License</a>
</p>

<p align="center">
  A trim tab is the small flap on a rudder that turns a big ship. trimtab is two skills that work in Claude Code, claude.ai, the Claude desktop app and Cowork: <b>token-lean</b> makes Claude lead with the point that matters, say how sure it is, check its work before calling it done, hand mechanical steps to cheaper models, and write a handoff before a session gets long. <b>memory-map</b> (optional) gives Claude a small linked memory it reads before asking you the same thing twice.
</p>

---

> **Status: v3.0.0.** In a one-task test on Opus (one run per setup), the skill's rules reached the model and output quality matched a plain session, but no token saving showed up. Earlier, a first small benchmark ([bench/RESULTS.md](bench/RESULTS.md)) found that sending a bulk extraction to the cheapest model kept accuracy at far lower cost, that the memory map cost slightly more than a plain index at 40 notes, and that the lean-reply skill did not improve answers on its test. Savings over a real session are **not measured**. See [docs/LIMITATIONS.md](docs/LIMITATIONS.md).

## Quick Start

Pick the place you use Claude. The rules are the same everywhere.

### Claude Code

In Claude Code 2.1.275 or later:

```
/plugin install trimtab --marketplace tegaseries/trimtab
```

On older versions, use two steps instead (add the `tegaseries` marketplace from this repo, then install the `trimtab` plugin from it):

```
/plugin marketplace add tegaseries/trimtab
/plugin install trimtab@tegaseries
```

Then start a session with `/trimtab:token-lean`. The core rules apply for the rest of the session. Installing the plugin does not edit your `CLAUDE.md` or `settings.json`.

### claude.ai, desktop app and Cowork

1. Download [token-lean.skill](https://github.com/tegaseries/trimtab/raw/main/dist/token-lean.skill) (and [memory-map.skill](https://github.com/tegaseries/trimtab/raw/main/dist/memory-map.skill) if you want the memory skill). Each file is a ZIP of the skill folder.
2. In claude.ai or the desktop app, open **Customize**, then **Skills**. Click **+**, then **Upload a skill**, and choose the file.
3. Start a chat or a Cowork task with `/token-lean`.

Skills need code execution turned on, and they are available on Free, Pro, Max, Team and Enterprise plans ([Anthropic help center](https://support.claude.com/en/articles/12512180-use-skills-in-claude)). A skill you upload to your account also loads in Claude Code when you sign in with the same account, so on one machine pick either the plugin or the upload, not both.

<details>
<summary><b>Copy the folder instead (Claude Code, any OS)</b></summary>

**Copy the folder.** Download this repository as a ZIP (green **Code** button, then **Download ZIP**), unzip it, and copy the skill folder into your skills directory.

macOS or Linux:

```bash
cp -r trimtab-main/plugins/trimtab/skills/token-lean ~/.claude/skills/
```

Windows PowerShell:

```powershell
Copy-Item -Recurse trimtab-main\plugins\trimtab\skills\token-lean $env:USERPROFILE\.claude\skills\
```

To install for one project only, copy it into that project's `.claude/skills/` folder instead. For `memory-map`, use the folder `plugins/trimtab/skills/memory-map` the same way. A copied skill is typed without the plugin prefix: `/token-lean`.

</details>

**Other places.** Developers can load the same skill folder through the Claude API's code execution tool (beta). Uploading custom skills from the mobile app is not confirmed; upload on the web or desktop app first. claude.ai accepts only these frontmatter keys: `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`. The skills here use only those.

## What You Get

| Skill | Command | What it does |
|---|---|---|
| `token-lean` | `/trimtab:token-lean` (plugin) or `/token-lean` (upload or folder copy) | Core rules at the top of `SKILL.md`: lead with the point, confidence tags (`[Certain]`, `[Likely]`, `[Guessing]`), checks before "done", write less, read less, lean replies. Then the detail: task sizing, a "mechanical test" for handing steps to cheaper models, context hygiene, session handoff. |
| `memory-map` | `/trimtab:memory-map` (plugin) or `/memory-map` (upload or folder copy) | Optional. A memory made of plain markdown files: root index, then one topic hub, then one note. Rules for looking up, saving and logging work. Starter templates included. |

Both are plain markdown. No scripts, no hooks, no network access, and nothing edits your settings. They behave the same on Windows, macOS and Linux.

## How It Works

A skill is a folder with a `SKILL.md` file. Claude keeps only each skill's one-line description in context until the skill is needed; typing the command loads the full file. token-lean puts its core rules at the very top, so they take effect as soon as it loads and stay for the rest of the session.

- After `/clear` or a compaction the rules can drop out: type the command again.
- For rules in **every** session without typing anything, paste the block from [examples/CLAUDE.md.snippet](examples/CLAUDE.md.snippet) into your `CLAUDE.md`. It costs roughly 900 tokens per session.
- For memory, copy the files from `plugins/trimtab/skills/memory-map/templates/` into a folder (default `~/.claude/memory/`) and ask Claude to use `memory-map`.

New to skills? [docs/HOW-IT-WORKS.md](docs/HOW-IT-WORKS.md) is a plain-language walk-through.

## Configuration

The skill ends with a table of the numbers you can change: the session cap (about 30 messages), when to delegate (batches of 3+ mechanical pieces), the pilot pass mark, confidence tags and tone. To change the always-on rules, edit the "Core rules" section of `SKILL.md` and your pasted snippet. Prefer a gentler tone? Say "softer": Claude keeps every concern and opens with one short acknowledgement.

These are starting points. Compare your own usage with and without them before you trust them.

## Uninstall

- Plugin: `/plugin uninstall trimtab@tegaseries`.
- Copied folder: delete `~/.claude/skills/token-lean`.
- claude.ai, desktop app or Cowork: open **Customize**, then **Skills**, and turn the skill off or delete it.
- Snippet: delete the block between the `token-lean:begin` and `token-lean:end` markers in your `CLAUDE.md`.

## Known Limits

The savings are mostly unmeasured, the skills add a small fixed token cost, subagents only pay off on large jobs, the memory has no automation, and everything stored in it is plain text. Details and the numbers behind them are in [docs/LIMITATIONS.md](docs/LIMITATIONS.md).

## Contributing

Issues and pull requests are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md). Release notes are in [CHANGELOG.md](CHANGELOG.md).

## License

[MIT](LICENSE). Made by [tegaseries](https://www.tegaseries.com).

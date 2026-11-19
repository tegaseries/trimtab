# trimtab plugin

A trim tab is the small flap on a rudder that turns a big ship. This plugin is a few small rules that steer Claude toward better work for fewer tokens.

## Skills

| Skill | Command | What it does |
|---|---|---|
| `token-lean` | `/trimtab:token-lean` | Core rules at the top (advisor-style replies, checks before "done", write less, read less, lean replies), then detail: task sizing, a mechanical test for handing steps to cheaper models, context hygiene and session handoff. Type it at the start of a session. |
| `memory-map` | `/trimtab:memory-map` | Optional. A small linked memory in plain markdown files: a root index, one hub per topic, one note per fact, and work logs. Starter templates are in `skills/memory-map/templates/`. |

Markdown only: no scripts, no hooks, no network access, and nothing edits your settings.

Install in Claude Code (2.1.275 or later):

```
/plugin install trimtab --marketplace tegaseries/trimtab
```

On claude.ai, the desktop app or Cowork, upload `dist/token-lean.skill` under **Customize**, then **Skills**, and type `/token-lean`. Steps, other routes and limits: see the [main README](../../README.md).

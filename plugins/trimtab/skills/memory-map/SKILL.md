---
name: memory-map
description: A small linked memory for Claude made of plain markdown files: a root index, one hub per topic, one note per fact, and work-log notes. Use to look up what the user has told you, or what you did together, before asking them again; to save durable facts; and to log finished work. Works on any surface that can read the folder.
license: MIT
compatibility: Plain markdown files in one folder. Claude Code reads and writes it directly; claude.ai, Cowork and mobile need a connector (such as a file-sync or drive connector) that can read the same folder. No scripts, no network access.
metadata:
  version: "1.1.1"
---

# memory-map

Claude forgets between sessions. This skill keeps a small memory as files and gives Claude a fixed route to find things: **root index, then one hub, then one note**. Facts are traced, not searched.

## Layout

Pick one folder and use it everywhere. Before the first write, ask the user where to keep it and wait for the answer; propose `~/.claude/memory/` in Claude Code as the default, and never create or fill a folder they have not agreed to. To let other surfaces read it, put the folder inside a synced folder (Google Drive, OneDrive, Dropbox, iCloud) that they can reach through a connector.

```
memory/
  MEMORY.md              root index: at most 30 lines, always read first
  hubs/hub-<topic>.md    one row per note in that topic
  <name>.md              one note: one fact, or one project history
  <topic>-history.md     work log for a project
```

Starter files are in `templates/`. Copy them into the folder to begin.

### Root, `MEMORY.md`

Four sections and one date line: `Updated: YYYY-MM-DD`; **Pins** (standing rules, a few lines); **Hubs** (one line per hub: topic, file, count, open items); **Active plans** (paths); **Inbox** (notes waiting to be filed).

### Hub, `hubs/hub-<topic>.md`

A table with one row per note: `name | state | where the output lives | links | open`. `state` is one short line taken from the note's description. `open` is what is unfinished, or `-`.

### Note, `<name>.md`

Frontmatter, then the fact:

```
---
name: short-kebab-name
description: one line saying what this note holds
metadata:
  type: user | feedback | project | reference
---
The fact, in a few lines. Link related notes with [[other-name]].
```

One fact per note. Project notes add `Why:` and `How to apply:` lines.

## What to do

**1. Look up before asking.** Before asking the user for a link, a file, a profile detail or a project fact they may have given before: read the root, open the one hub that matches, read the note, follow its `[[links]]`. Ask only when root, hub and note all come up empty.

**2. Save durable facts.** Save things that will still matter later: preferences, stable project facts, links and files the user will want again, corrections to how you work. Do not save one-off details, anything in the code or history already, or guesses. After saving, say what you saved in one line. If the user asks you to forget something, delete the note and its hub row.

**3. Log the work, not just the facts.** When a piece of work finishes or pauses (built, drafted, edited, researched, decided), create or extend `<topic>-history.md`: what you did, what the user did, the date, tools used, where the output lives (full paths or URLs), decisions and why, and open items on a line starting `Open:`. Add to the history; never overwrite it.

**4. Keep the map true.** On every change, update the hub row, the root counts and the `Updated:` date in the same step. Split a hub when it passes about 40 rows. Mark a note `state: stale` instead of deleting it silently. If a hub row and a note disagree, trust the note and fix the row.

**5. Say when the map is old.** If the root's `Updated:` date is more than 30 days old and the user mentions recent work, say once that the memory may be out of date.

**6. Read-only surfaces.** If this surface can read the folder but not write it, do not claim to have saved anything. End the reply with a short list under "Save in memory:" that the user can paste into a session that can write.

## Never store

Passwords, API keys, tokens, recovery codes, card or bank numbers, government ID numbers. If the user pastes one, do not copy it into memory; say that you left it out and why.

Memory files are plain text. Anyone who can open the folder can read every note, so do not put the folder in a public repository or a shared drive. Before syncing or sharing the folder, search it for secrets, for example with `grep -rniE "(password|passwd|secret|api[_-]?key|token)[[:space:]]*[:=]|sk-[A-Za-z0-9]{20,}|AKIA[0-9A-Z]{16}" memory/` (macOS, Linux, Git Bash) or the same pattern in `Select-String` (PowerShell). A pattern search is a rough check: it misses secrets that do not match and flags harmless text.

## Limits

- No scripts and no hooks: Claude keeps the hubs and counts up to date by hand, so they can drift. Rule 4 limits the drift; rule 5 flags it.
- Quality depends on the model following the routine. If a lookup fails, fix the note or the hub row and say so.

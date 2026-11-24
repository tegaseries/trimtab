# Limitations

Read this before relying on the skills.

## 1. The savings are barely benchmarked

- **Estimated (the memory lookup):** following the map (root index, one hub, one note) reads a few kilobytes however many notes exist: roughly 3 KB for the root, 1 to 2 KB for a hub and a few KB for the note, about 2K tokens in all. Reading every note grows with the count: 40 notes of 4 KB each is about 160 KB, roughly 40K tokens. That is a saving of around 95 percent, but nobody reads every note, so treat it as an upper bound. Against a plain auto-loaded index with notes opened on demand, the saving is much smaller. Token counts here are estimated as bytes divided by four.
- **First small benchmark, 2026-11-19** ([bench/RESULTS.md](../bench/RESULTS.md); two to ten runs per arm, synthetic notes, directional only):
  - At 40 notes the memory map cost about 12 percent more per question than a plain index, with the same accuracy. The map is built for larger folders; the crossover point was not measured.
  - Moving a 40K-token bulk extraction to the cheapest model gave the same accuracy as the middle model; in a later inline test the cheapest model was about 25 percent cheaper per step, not a multiple.
  - The lean-reply skill made replies about a third longer with no quality gain on the test rubric.
- **Second benchmark, 2026-11-24** (Sonnet 5.5, headless Claude Code, four small tasks: bulk extraction from 45 files, a code fix, an advice question, a constrained post; 8 runs per arm in the first round, 3 per task per arm in the second; directional only):
  - Always-on rules did not make small tasks cheaper. The 3.1.0 core cost about 11 percent more per run than no rules and a core half its size about 12 percent more (about $0.004 per run), with the same turns and the same correctness (9 of 9 checked tasks passed in both arms). The extra cost looks fixed per run, not tied to the size of the rules.
  - Delegating a mechanical step to the restricted `lean-worker` cost about $0.017 for the worker against about $0.57 for the default general-purpose agent doing the same step. One end-to-end run each, so treat it as a single data point.
  - Grep or Bash over a folder was 30 to 90 times cheaper than one Read per file on bulk extraction.
  - Not measured: whether the rules prevent rework, and large multi-turn tasks, which is where the savings are expected. Expect these rules to be roughly cost-neutral on small tasks.
- **Still not measured:** the session cap, savings over a real multi-turn session, and any percentage for model routing or lean replies. They are reasonable habits, not proven savings.
- To test it yourself, run the same task with and without the skill and compare the usage numbers your plan reports.

## 2. They add a small fixed cost

- The skill descriptions are loaded in every session.
- The full `token-lean` skill (about 10 KB, roughly 2.5K tokens) is loaded when it triggers.
- The optional snippet in [examples/CLAUDE.md.snippet](../examples/CLAUDE.md.snippet) costs roughly 1,000 tokens in every session.
- In a short session that never touches these topics, the skill can cost more than it saves.

## 3. Subagents can cost more than they save

A subagent starts with no context and has to re-read what it needs. The skill sets a threshold (about 30K tokens of reading or 15 tool calls) so that small jobs stay inline. The numbers are starting points, not findings. Tune them against your own usage.

## 4. The memory has no automation

- There are no scripts or hooks, so Claude keeps the hubs, counts and dates up to date by hand. They can drift. The skill tells Claude to fix a row when it disagrees with its note, and to say when the root's `Updated:` date is more than 30 days old.
- Other surfaces (claude.ai, Cowork, mobile) can read the folder only through a connector and usually cannot write to it. The skill tells Claude to list what to save instead of claiming it saved it.
- Cross-device reading depends on a synced folder and a connector. This repository ships neither.

## 5. Privacy and secrets

- Memory files are plain text. Anyone who can open the folder can read every note. Do not put the folder in a public repository or a shared drive.
- The skill tells Claude never to store passwords, keys, tokens, recovery codes, card or bank numbers, or government ID numbers. It is an instruction, not a filter, so check the folder yourself before syncing or sharing it. Pattern searches are a rough check: they miss secrets that do not match, and they flag harmless text.

  macOS or Linux (and Git Bash):

  ```bash
  grep -rniE "(password|passwd|secret|api[_-]?key|token)[[:space:]]*[:=]|sk-[A-Za-z0-9]{20,}|AKIA[0-9A-Z]{16}" memory/
  ```

  Windows PowerShell:

  ```powershell
  Get-ChildItem memory -Recurse -Filter *.md | Select-String -Pattern '(password|passwd|secret|api[_-]?key|token)\s*[:=]|sk-[A-Za-z0-9]{20,}|AKIA[0-9A-Z]{16}'
  ```

## 6. Style is a preference

"Lead with substance", "skip filler" and "hold your position" suit some people and read as curt to others. Say "softer" for a gentler voice, or delete the rules you do not want from your copy of the skill.

## 7. Platform notes

- The skills are markdown only, so they run on Windows, macOS and Linux.
- Subagent routing and `/compact` exist only in Claude Code. On claude.ai and Cowork the advisor rules, knowledge-gap rule, lean replies and session cap still apply.
- Model names and prices change, so the skill names tiers (planner, main, worker) instead of models. Map them to the models your plan offers.
- A skill is a suggestion Claude follows, not a guarantee. Behaviour varies by model and by how long the session is.

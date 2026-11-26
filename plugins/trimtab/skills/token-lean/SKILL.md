---
name: token-lean
description: Advisor-style replies plus quality-first token saving - checks before "done", write-less ladder, strongest model plans while mechanical steps go to cheap workers, lean context, session handoff. Use at the start of a session when the token-lean core rules are not already in CLAUDE.md (type /token-lean, or /trimtab:token-lean from the trimtab plugin), before planning multi-step work, delegating to subagents or writing a plan file, or when the user mentions tokens, cost, usage limits or "lean".
license: MIT
compatibility: Works in Claude Code, Cowork and claude.ai. Subagents, /clear and /compact apply only where those features exist. No scripts, no network access, no settings changes.
metadata:
  version: "3.2.2"
---

# token-lean

## Core rules: follow these for the rest of the session
Same text as the optional CLAUDE.md block (`examples/CLAUDE.md.snippet`); if that block is already loaded, these repeat it.

Goal: the best output for the fewest tokens; rework is the biggest waste.

**Replies**
- Lead with the most useful point (the answer, a concern, a gap or a question the user may not have thought of); never open with agreement or praise. Under a concise or brief reply style the result comes first, and a challenge follows only if it would change what the user does.
- Tag claims the user may act on: `[Certain]` hard evidence (a file, a test, a source) · `[Likely]` strong inference · `[Guessing]` filling gaps. One tag per group of claims with the same confidence; no tag on what is plainly in the user's message or the tool output. If mostly guessing, say so first. Tags apply under any reply style.
- Disagree plainly: "I disagree because [reason]. Instead: [alternative]. Risk: [downside]." Hold the position unless given new information.
- Short plain sentences, no filler, no recap, no narration between tool calls. Never cut negations, numbers, exact errors, commands, security warnings or confirmations before irreversible actions.

**Work**
- Check before "done": a test or command for code and data; for writing and design, a 5–8 item pass/fail list made from the request before starting. Say what was not checked.
- Write less: skip what is not needed, reuse what exists, use built-in features, then the minimum that works. Never drop a requirement, validation, error handling, security or a test.
- Big or unclear task (several files or pieces, over an hour, or a vague ask): ask 3–5 multiple-choice questions, write a one-page spec with its checks, show the plan before building. Small tasks: just do them.
- Read less: search before reading, read line ranges (the whole file before editing it), cap command output, batch calls (over 5 files: one Grep or Bash call, or all Reads in one parallel turn, never one Read per turn), never re-read known files. Say when a read was partial.

**Models**
- Start model (the user picks it; a session cannot change its own): the mid tier of the user's plan for routine work with a clear ask, the strongest tier for what the user steers or that needs judgement, the cheapest for purely mechanical sessions. A skill cannot see which models a plan includes: if a tier is not available, say so once and use the next one down. Check which model you are (the system prompt names it); if it is clearly wrong for the task, say so once in one line.
- Delegate only mechanical steps (the skill's test) to the `lean-worker` agent (haiku with Read, Glob, Grep and Bash; in the plugin it is `trimtab:lean-worker`), never to a general-purpose agent, which carries every tool and cost 10-20x more in a test. Keep a haiku worker's whole context under 100K tokens (above that Haiku costs 5x); send bigger reads to `sonnet` or split them. Name workers by alias (`haiku`, `sonnet`, `opus`), never a pinned model ID. A failed check gets one retry with the error, then the session's model does it. No agent teams.
- Use a planner above the session's model (if the plan includes one) only when the user asks for deep analysis or a bug survived one fix; architecture, security and data-loss calls stay with the strongest model the session has. A tier is out only on "out of usage credits" or a 429 that repeats after one retry 60 s later: drop one tier, say so in one line, and never let a weaker model plan silently.
- Facts that change (Claude's own apps and models, libraries, APIs, prices, news): when the answer would change what the user does, make one search or read one official page first, unless this session or memory already holds a dated answer under 30 days old. Answer from memory only when no search tool is available, and say so in one line.

**Sessions**
- One task per session. When a task ends, suggest `/clear` before unrelated work; suggest `/compact` only when the next work needs this context (it is itself a large request). Never switch models mid-session.
- At about 30 user messages: finish the step, write a handoff (25 lines or fewer), offer a new session.

Everything below is the detail behind these rules. These are working heuristics, not measured guarantees; the last section lists the knobs.

## Where tokens go
1. **Re-read context.** Every turn re-sends the whole conversation, tool results included.
2. **Rework.** A wrong first attempt pays twice. Checks and a clear spec prevent it.
3. **Output and thinking.** Thinking is billed as output.
4. **Model price.** Check your plan's pricing page. When the strongest model costs only about 2x the middle one, one redo erases the saving, so start on the strongest model for anything that needs judgement. Haiku 5.5 is $0.10/$0.50 per million tokens only while the whole prompt (cache included) stays under 100K tokens, and $0.50/$2.50 above it (pricing page, read 2026-11-23). Cache writes cost 2x input (1 h) and cache reads 0.05x on Sonnet and Opus 5.5 (0.1x on Haiku). In our tests the cheapest tier on mechanical work saved about 25% per step, not a multiple.
5. **Loaded extras.** CLAUDE.md, skills, tool schemas, subagent cold starts.

## Size the task first
| Size | Signs | Do |
|---|---|---|
| Small | one file or answer, clear ask | do it → check → done |
| Big | several files or pieces, over ~1 hour, or a vague ask | questions → spec → plan → user approval → build in pieces → check each → review |

Spec (big only): 3–5 multiple-choice questions, then one page: goal, must-haves, out of scope, the checks. In background or unattended runs, stop after the plan for approval.

## Checks
Write the check before the work.
- Code and data: a test, command, count or expected output. Run it.
- Writing and design: 5–8 pass/fail items from the request (word limit, no placeholders, required names and links present, matches the brief, nothing invented). Report failures, not passes.
- State what was not checked. Never call an unchecked result done.
- Big work: one reviewer subagent reads only the spec and the changed files, reports broken requirements and security or data risks, and is told "skip style". Fix real findings only.

## Write less
Stop at the first rung that works: not needed → skip · exists in the project → reuse · standard library or built-in feature → use · installed dependency → use · one line · the minimum that works. Never cut a requirement, validation, error handling, security, accessibility or a test. Minimum code, not minimum look: design still meets the brief.

## Routing
Applies where subagents exist (for example Claude Code). Three tiers: **planner** (strongest), **main** (balanced), **worker** (cheapest).

**Mechanical test**, per step; any doubt = no.
1. Can the exact result, or a check that proves it, be stated before starting?
2. Does it need no taste, no call on the user's preferences, no reconciling of 3+ sources?
3. Is a wrong result cheap to spot and undo?

All yes → worker. All yes but the step reads or moves >30K tokens → main tier with the exact spec and check. Anything else stays with the session's own model; a step that fails test 2 goes to no worker, whatever its size.
- Delegate only when it saves: a batch of 3+ mechanical pieces, or bulk reading that only extracts facts. A single small step is cheaper inline. One agent per batch, not per file.
- **Worker type:** always the restricted `lean-worker` agent (haiku; Read, Glob, Grep, Bash only; `trimtab:lean-worker` in the plugin, or copy `plugins/trimtab/agents/lean-worker.md` to `~/.claude/agents/`), never a general-purpose agent: it carries every tool and cost about 30x more in our test ($0.57 against $0.017 for the same step). Keep a haiku worker's whole context under 100K tokens; bigger reads go to `sonnet` or are split.
- Worker brief: exact steps, files, the check, low effort (over 5 files: one Grep or Bash call, or parallel Reads, never one Read per turn), report ≤10 lines ending `LOG: done=… | outputs=… | decisions=… | open=…`. The worker runs its own check and reports pass or fail with the evidence.
- Fail → one retry, same worker, with the error. Second fail → the session's model does that step. Judge workers by their check, never by re-reading passed work.
- Bulk extraction by fixed rules: pilot 5 on the worker; keep it if 4 of 5 pass, else main tier.
- Work that needs judgement is never judged by a weaker model, nor only by its author.
- Continue a live worker with a message; replace a dead one with "resume <plan path> from step N".
- No agent teams (several times the tokens); use subagents.
- Briefs carry the user's words verbatim. Output over ~40 lines goes to a file; relay the path plus ≤10 lines.

**Plan file** (work that delegates or may span sessions): `plans/<YYYY-MM-DD>-<slug>.md`. Line 1: `# <title> - status: active|paused|done - created <YYYY-MM-DD>`. Then Request (verbatim), Pointers, Decisions (dated, append-only), Steps (`[tier] step — check: … — status: todo|done|failed`), Log (append each LOG line). The file, not the agent, is the durable state.

**Tier failure.** A tier is out only when the error says usage or credits are exhausted, or a rate-limit error repeats after one retry at least 60 s later; report any other error as an error. When a tier is out: say so in one line, re-send the same brief to the next tier, skip that tier this session. Never let the cheapest tier plan non-mechanical work silently: ask the user first and state the risk.

## Context hygiene
- Search first (file names only, capped), then read only the needed lines. Read the whole file before editing it. Say when a read was partial.
- Never re-read a file already in context or just edited. Cap output (`| head -n 40`, quiet flags); quote the one decisive line.
- Load skills and tool schemas only when the task needs them. Prefer a CLI over an MCP tool when both work.
- Lower effort only for mechanical worker steps. Planning, review and anything the user steers stay at the session's effort.
- Keep always-loaded files (CLAUDE.md, memory index) short; detail belongs in on-demand skills.

## Session end and handoff
- Task ends → suggest `/clear` before unrelated work. `/compact` only when the next work needs this context; it re-reads everything to summarise it.
- After `/clear` or a compaction, these rules may drop out: type `/token-lean` (plugin: `/trimtab:token-lean`) again (or paste `examples/CLAUDE.md.snippet` into CLAUDE.md, see the README).
- At ~30 user messages (automated notices don't count; after compaction treat the cap as reached): finish the step, write the handoff, ask once to continue in a new session.
- Handoff ≤25 lines: goal, decisions and why, done, current state (files, paths, commands), open, exact next step. Working folder → `HANDOFF.md`; no folder → one fenced block.
- Work needing more than 30 messages or several sessions: suggest a Claude Project early and offer to draft its instructions (≤25 lines).

## Other surfaces
- **Claude Code:** everything applies.
- **Cowork and claude.ai chat:** advisor rules, checks, write-less, lean replies and the session cap apply. Subagents, `/clear` and `/compact` do not (and plan model lists differ, so use the model menu you have); the user picks the model in the model menu.
- **Durable memory** across sessions is a separate, optional skill: `memory-map`, in the same trimtab plugin. This skill works without it.

## Tune it
| Knob | Default | Change it when |
|---|---|---|
| Session cap | about 30 user messages | Your sessions run cheaper or dearer than the default |
| Delegation | batches of 3+ mechanical pieces, or >30K tokens for the main tier | Your usage numbers show delegation costing more or less |
| Pilot pass mark | 4 of 5 | A bulk task needs stricter accuracy |
| Confidence tags | on for non-trivial claims | The tags clutter short answers |
| Tone | direct | The user asks for softer: keep every concern and number, open with one sentence on what is reasonable in their view |

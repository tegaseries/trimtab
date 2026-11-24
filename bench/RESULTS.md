# Benchmark results (2026-11-19)

Small, cheap, controlled runs. They show direction, not proof. Read the confounds before quoting any number.

## Method

- Every run was a fresh subagent with a `RUN-ID` tag in its prompt. Token usage was read from its transcript (input, cache write, cache read, output) and priced with the public per-million-token rates for the model it ran on.
- Rates used (per million tokens): Haiku 5.5 input 0.10, output 0.50; Sonnet 5.5 input 2.00, output 10.00, cache read 0.10. Cache write was priced at 1.25 times input, and Haiku cache read at 0.01 (assumed, not looked up).
- Haiku prompts above 100K tokens are billed at a higher rate (0.50 / 2.50). The Haiku delegation runs probably crossed that line on their last turn, so their cost is a lower bound. The upper bound, with every token at the long rate, is about 0.11 USD per run.
- Fixture: 40 synthetic notes of about 4 KB across 8 topics (160 KB, about 40K tokens), generated from a fixed seed. Two layouts of the same notes: `fixture/flat/` (notes plus one short index) and `fixture/map/` (root, 8 hubs, notes). No personal data.
- Answer keys were kept outside the repository during the runs so the agents could not read them.

## Test 1: memory lookup (10 questions, Haiku, one fresh agent per question)

| | flat index | memory map |
|---|---|---|
| Correct answers | 10 / 10 | 10 / 10 |
| Cost per question | 0.00686 USD | 0.00771 USD (+12%) |
| Turns | 3.5 | 4.2 |
| Tool calls | 4.7 | 6.1 |
| Output tokens | 1,306 | 1,684 |

Flat was cheaper on 9 of 10 questions. At 40 notes the map's extra hop (root, then hub, then note) and the cost of reading the skill outweigh what its index saves.

Not tested: more notes (the crossover point), stale hubs, or the skill being read once per session instead of once per question.

## Test 2: lean replies (8 prompts, Sonnet, with and without the token-lean skill)

| | without skill | with skill |
|---|---|---|
| Quality (24-point rubric) | 24 / 24 | 24 / 24 |
| Output tokens per reply | 1,518 | 2,025 (+33%) |
| Turns | 1 | 2 (the skill is loaded first) |
| Cost per run | 0.137 USD | 0.192 USD (+40%) |

No quality gain on this rubric, and the rubric was already at its ceiling. The baseline already carried the author's own advisor rules in `CLAUDE.md`, so this measures the skill on top of those rules, not against a clean install. The extra turn is a one-time load in a real session but is counted on every run here, so the cost figure overstates a long session. The output increase is not a load effect.

## Test 3: delegation (bulk extraction, 40 notes, about 40K tokens, 2 runs per model)

Task: list name, first fact and latest date for all 40 notes in a fixed format.

| | Haiku 5.5 | Sonnet 5.5 |
|---|---|---|
| Accuracy (name, fact, date) | 40/40, 40/40 (both runs) | 40/40, 40/40 (both runs) |
| Cost per run | 0.022 USD (up to about 0.11) | 0.139 and 0.168 USD |
| Turns | 3 | 5 and 6 |
| Output tokens | 12.9K and 13.4K | 2.0K and 3.8K |
| How it read | 40 to 41 single-file reads | 2 to 3 shell commands |

Same accuracy, 1.3 to 7.6 times the cost on Sonnet depending on which bound you take for Haiku. Haiku cost more output tokens by reading file by file, which Sonnet avoided with a shell loop.

Arithmetic for the main loop doing the same work inline (not measured, assumes a main loop at 4.00 / 20.00 with cache read 0.20):

- Read 40K tokens: about 0.20 USD to write into the cache, about 0.03 for the answer, 0.008 for each later turn that re-reads it. With 10 later turns: about 0.31 USD.
- Delegate to Haiku: about 0.03 to 0.12 USD including the brief and the report.
- Delegate to Sonnet: about 0.15 to 0.18 USD. Against a Sonnet main loop doing it inline (about 0.16 USD) that is break-even, so delegating to the same tier saves nothing.

## What this supports

- Dropping a bulk, fully specified extraction to the cheapest tier saved money at no accuracy cost, in this test.
- A memory map is not cheaper than a plain index at 40 notes.
- The lean-reply behaviour did not improve answers on this rubric, and it made replies longer.

## What it does not support

- Any percentage saving for model routing, lean replies or the session cap over a real session.
- Anything about large memories, stale hubs, or other models.
- The 30K-token delegation threshold. The test sat at 40K. A smaller job was not run.

## Confounds

- Two to ten runs per arm, one model per test, synthetic notes.
- Subagents carry a base context of about 60K tokens that is re-read from cache on every turn, so extra turns cost more than extra content.
- Subagents inherited the author's `CLAUDE.md`, so there was no clean baseline.
- Prices came from a public table; cache rates for Haiku were assumed.
- Skill and note reads are counted per run, where a real session pays them once.

# Multi-Agent Orchestration Playbook

This file is loaded into every Claude Code session in this project (referenced from `CLAUDE.md`). It tells the **top-level Claude session** how to drive a team of specialist sub-agents.

## Why orchestration lives at the top level

Claude Code sub-agents **cannot spawn other sub-agents** — this is a hard architectural limit ([docs](https://code.claude.com/docs/en/sub-agents), [issue #4182](https://github.com/anthropics/claude-code/issues/4182)). The YAML `tools:` field is descriptive, not enforced. So the orchestrator role belongs to the top-level session — that's you, reading this. There is no `supervisor-orchestrator` sub-agent; if you see one referenced, treat it as deprecated.

## When to orchestrate vs. just do it

**Orchestrate** when the request involves any of:
- A new feature with UX + architecture + implementation
- A multi-bug fix
- A refactor that crosses several files
- The user explicitly says "use the team", "orchestrate this", "send the team", or names multiple agents

**Just do it** (no orchestration) when:
- Single-file edit, simple bug fix, one-shot question
- Exploratory question — answer directly or use `Explore`
- The user wants quick iteration and asked for solo work

When in doubt, ask the user before kicking off a multi-agent flow — it costs more tokens and time.

## The team

| Agent | Role | Produces |
|---|---|---|
| `project-manager` | Owns ticket + Definition of Done. Tracks scope. Flags decisions for the user. | `.tickets/YYYY-MM-DD-{slug}.md` |
| `team-leader-architect` | Diagnoses root cause; defines fix/build approach. Does NOT implement. | `.architecture/{slug}.md` |
| `product-designer` | Specifies UX/UI decisions. Does NOT implement. | `.design/{slug}.md` |
| `senior-engineer` | Implements per ticket + architecture + design specs. Runs build/tests. | Code changes; passing build |
| `qa-automation` | Verifies acceptance criteria; adds regression tests. | New tests; SHIP / DO NOT SHIP verdict |
| `code-reviewer` | Quality pass on landed code. | APPROVE / WITH NITS / REQUEST CHANGES |

## Default 3-wave pattern

**Wave 1 — Specification (parallel)**
- `project-manager` → ticket + DoD
- `product-designer` → UX spec (skip if no UX work)
- `team-leader-architect` → root cause / architecture (skip if no design decisions)

Run them in parallel: a single message with multiple `Agent` tool calls. They each consume the same brief independently.

**Wave 2 — Implementation (sequential)**
- `senior-engineer` → reads Wave 1 outputs, implements, runs `compile` / `test` / `build`. Reports back with file list, test count, and any spec deviations.

**Wave 3 — Verification (parallel)**
- `qa-automation` → re-runs tests, adds regression fixtures, walks the DoD, gives SHIP verdict
- `code-reviewer` → reads diffs, gives APPROVE verdict, may surface non-blocking nits

If Wave 3 surfaces blockers, loop Engineer + verifiers until clean.

## Right-sizing

For small cleanups (few nits, no UX/architecture change):
> PM → Engineer → (QA + Reviewer in parallel)

For brand-new projects:
> Architect first (stack/structure), then PM, then implementation flow

For UX-only changes (no logic):
> Designer → Engineer → (QA + Reviewer)

The 3-wave pattern is a default, not a contract. Cut agents that have nothing to do.

## Hard rules

- **Always run independent agents in parallel.** A single message with multiple `Agent` calls. Sequential calls when they could be parallel wastes wall-clock time and the user's patience.
- **Each agent must produce an artifact** — a file at `.tickets/`, `.design/`, `.architecture/`, or actual code/test changes. Not just narration.
- **Per-agent attribution in the final report.** Show a table or list naming each agent and what they produced. If your final report reads like one person did everything, you skipped delegation.
- **Surface blocking decisions to the user before Wave 2.** Don't ask the user about decisions a sub-agent can make autonomously (designer picks layout patterns, engineer picks variable names). Do ask about things that affect scope or external commitments.
- **Use `TodoWrite`** for waves so the user sees the orchestration unfold.
- **The supervisor role is YOU.** Do not invoke a `supervisor-orchestrator` sub-agent (it doesn't exist; see top of this file).

## Anti-patterns

- ❌ Spawning a "supervisor" sub-agent and expecting it to dispatch — sub-agents can't spawn other sub-agents. You drive.
- ❌ Doing implementation work yourself with Write/Bash when `senior-engineer` could. Exception: trivial single-file edits.
- ❌ Sequencing Wave 1 agents one at a time — waste.
- ❌ Skipping QA or Code Reviewer "to save time". They catch what the engineer misses.
- ❌ Producing a single end-of-task narrative summary that hides which agent did what — that's the failure mode the orchestration pattern is supposed to prevent.

## Briefing each agent

When you call an agent, the prompt is the only context it has. Include:
- The user's actual request (or the ticket reference if PM has written one)
- Paths to artifacts from earlier waves it should read first
- Concrete acceptance criteria — what "done" looks like
- Constraints (no scope creep, must keep tests passing, etc.)
- The format of the report you want back (e.g., "under 250 words; list of files changed; exit codes")

A vague prompt produces vague output. Treat each agent like a smart colleague who just walked into the room with no prior context.

## Triggering phrases

Watch for these in user messages — they mean "use this playbook":
- "use the team" / "send the team"
- "orchestrate this" / "coordinate this"
- "@supervisor-orchestrator" or "use the supervisor"
- "use the @agent:..." with multiple agents named
- A multi-step feature or bug request that obviously needs >1 specialist

When triggered, follow the playbook. Otherwise just do the work directly.

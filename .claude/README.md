# `.claude/` — Multi-Agent Orchestration Pattern

A drop-in pack that gives any Claude Code project a 6-agent specialist team plus an orchestration playbook the top-level Claude session follows.

## Install in a new project

1. Copy this directory into the new project root:
   ```sh
   cp -r /path/to/source/.claude /path/to/new-project/.claude
   ```
2. Add this to the project's `CLAUDE.md` (create one if it doesn't exist):
   ```md
   ## Multi-agent orchestration

   For non-trivial multi-step tasks (features, multi-bug fixes, refactors) follow the playbook in `.claude/ORCHESTRATION.md`. The orchestrator role lives in the top-level Claude session — there is no `supervisor-orchestrator` sub-agent.
   ```
3. Restart your Claude Code session. The new session will read CLAUDE.md, follow the pointer to ORCHESTRATION.md, and discover the 6 agents in `agents/`.

That's it. Ask Claude something like *"use the team to add X feature"* and watch the waves run.

## What's in this pack

| File | Purpose |
|---|---|
| `ORCHESTRATION.md` | The playbook — when to orchestrate, the 3-wave pattern, hard rules, anti-patterns |
| `agents/project-manager.md` | Writes tickets and Definition of Done |
| `agents/team-leader-architect.md` | Diagnoses root causes; defines architecture |
| `agents/product-designer.md` | Specifies UX/UI |
| `agents/senior-engineer.md` | Implements features and fixes |
| `agents/qa-automation.md` | Verifies acceptance criteria; adds regression tests |
| `agents/code-reviewer.md` | Quality pass on landed code |

## Important architectural note

Claude Code sub-agents **cannot spawn other sub-agents** ([docs](https://code.claude.com/docs/en/sub-agents)). The YAML `tools:` allowlist is descriptive, not enforced. That's why there's no `supervisor-orchestrator` agent file — the orchestrator role lives in the top-level session, which reads `ORCHESTRATION.md`.

If a future Claude version supports nested sub-agents (or if you opt into experimental Agent Teams via `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`), the playbook can move into a sub-agent. Until then, top-level drives.

## Customizing per project

The 6 agent definitions are intentionally generic. Tighten them for a specific project by adding constraints to each agent's system prompt — e.g., a Rails project might add "always use ActiveRecord, never raw SQL" to `senior-engineer.md`. Keep `ORCHESTRATION.md` mostly stable; it's the meta layer.

## Convention: artifact directories

Agents write artifacts to:
- `.tickets/YYYY-MM-DD-{slug}.md` — PM tickets
- `.architecture/{slug}.md` — architect diagnoses and design notes
- `.design/{slug}.md` — designer specs

Add these to `.gitignore` if you don't want them committed, or commit them as decision records — your call.

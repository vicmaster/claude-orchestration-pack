# claude-orchestration-pack

A drop-in `.claude/` directory that gives any [Claude Code](https://code.claude.com) project a 6-agent specialist team plus an orchestration playbook the top-level Claude session follows.

Ask Claude *"use the team to add feature X"* — and watch six specialists hand off in parallel waves: PM writes the ticket, Designer writes the UX spec, Architect writes the diagnosis, Engineer implements, QA verifies, Code Reviewer signs off.

## Install

```sh
# In any Claude Code project root:
git clone https://github.com/vicmaster/claude-orchestration-pack /tmp/cop \
  && cp -r /tmp/cop/.claude . \
  && rm -rf /tmp/cop
```

Then add this to your project's `CLAUDE.md` (create one if needed):

```md
## Multi-agent orchestration

For non-trivial multi-step tasks, follow the playbook in `.claude/ORCHESTRATION.md`. The orchestrator role lives in the top-level Claude session — there is no `supervisor-orchestrator` sub-agent.
```

Restart Claude Code in that project. Done.

## What's in the pack

```
.claude/
├── ORCHESTRATION.md      The playbook the top-level Claude session follows
├── README.md             Pack-level install notes
└── agents/
    ├── project-manager.md         Ticket + Definition of Done
    ├── team-leader-architect.md   Root cause / architecture
    ├── product-designer.md        UX/UI specs
    ├── senior-engineer.md         Implementation
    ├── qa-automation.md           Verification + regression tests
    └── code-reviewer.md           Quality pass
```

## How it works

When you ask Claude something that triggers orchestration ("use the team", "send the team", "orchestrate this", or any obviously multi-step request), the top-level session follows a 3-wave pattern:

| Wave | Agents | Mode |
|---|---|---|
| **1 — Specification** | `project-manager` + `product-designer` + `team-leader-architect` | parallel |
| **2 — Implementation** | `senior-engineer` | single |
| **3 — Verification** | `qa-automation` + `code-reviewer` | parallel |

Each agent produces a real artifact (`.tickets/`, `.design/`, `.architecture/`, or code/test diffs) — not just narration. The final report shows per-agent attribution so you can see who did what.

## Why no supervisor sub-agent?

Claude Code sub-agents [cannot spawn other sub-agents](https://code.claude.com/docs/en/sub-agents) — it's a hard architectural limit to prevent runaway nesting. The YAML `tools:` field is descriptive, not enforced; the spawner tool is intentionally withheld from sub-agents.

So the orchestrator role lives in the **top-level session** — your Claude Code conversation. `ORCHESTRATION.md` is the playbook it reads. There's no `supervisor-orchestrator.md` agent file because such an agent would be structurally useless.

If you want true nested delegation, you can opt into [experimental Agent Teams](https://docs.anthropic.com) (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) or use the [`nested-subagent`](https://github.com/gruckion/nested-subagent) plugin. Neither is required for this pack.

## Customizing per project

The 6 agent definitions are intentionally generic. Tighten them for your stack by adding constraints to the system prompts — e.g., "always use Drizzle, never raw SQL" in `senior-engineer.md`. Keep `ORCHESTRATION.md` mostly stable; it's the meta layer.

## Triggering phrases

Watch for these in user messages — they should trigger the playbook:
- "use the team" / "send the team"
- "orchestrate this" / "coordinate this"
- "use the supervisor" or "@supervisor-orchestrator"
- A multi-step feature or bug request needing more than one specialist

For simple one-off edits or quick questions, Claude should just do the work directly.

## Anti-patterns the playbook prevents

- ❌ Running independent agents sequentially when they could run in parallel
- ❌ Agents producing only narration, no artifacts
- ❌ Final reports that hide which agent did what
- ❌ Skipping QA / Code Reviewer "to save time"
- ❌ Spawning a "supervisor" sub-agent and expecting it to dispatch (it can't)

## License

MIT — see [LICENSE](LICENSE).

## Contributing

Issues and PRs welcome. The pack is intentionally minimal; if you have stack-specific tweaks (Rails, Next.js, etc.), open an issue with the use case before sending a PR.

## Who built this

Built by [Victor Velazquez](https://github.com/vicmaster) — COO at [MagmaLabs](https://www.magmalabs.io), working on AI agent platforms. Follow me on X [@zazvick](https://x.com/zazvick) and say hi! 👋

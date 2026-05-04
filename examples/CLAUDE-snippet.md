# Snippet to add to your project's CLAUDE.md

Copy this section into the `CLAUDE.md` at the root of your project after installing the pack:

```md
## Multi-agent orchestration

For non-trivial multi-step tasks (new features, multi-bug fixes, refactors), follow the playbook in `.claude/ORCHESTRATION.md`. The orchestrator role lives in the **top-level Claude session** — there is no `supervisor-orchestrator` sub-agent (Claude Code sub-agents cannot spawn other sub-agents).

Available specialist sub-agents: `project-manager`, `team-leader-architect`, `product-designer`, `senior-engineer`, `qa-automation`, `code-reviewer`.

Trigger phrases like "use the team", "send the team", "orchestrate this" mean the user wants the multi-agent flow. For simple single-file edits or quick questions, Claude should just do the work directly.
```

If you don't have a `CLAUDE.md` yet, run `/init` in Claude Code to scaffold one, then add this section.

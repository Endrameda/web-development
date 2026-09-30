---
tags:
  - ai/architecture
  - ai/agents
status: seed
---

# CC - Subagents

> A subagent is a specialized instance of Claude Code that runs a task in its own isolated context window — its own system prompt, tool access, and permissions — and hands back only the result, so the main conversation's context never fills up with the intermediate search results, logs, or file contents that produced it.

## When to reach for it

Reach for a subagent whenever a task would flood your main context with things you won't need again: searching a large codebase for a symbol, running a test suite and wanting only the failures, or doing a code review where you only care about the findings. The subagent absorbs the exploration cost in a context you'll never see; you only pay for the summary that comes back.

Don't reach for one when the task genuinely needs everything already in the conversation — the history, the files already read, your steering along the way. That's what a **fork** is for (see Gotchas): it inherits the full session instead of starting cold.

## How

**Definition.** A subagent is a markdown file with YAML frontmatter — the frontmatter is metadata, the markdown body becomes the subagent's system prompt.

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.
```

Only `name` and `description` are required. Other fields worth knowing: `tools` (allowlist — inherits everything if omitted), `disallowedTools` (denylist against the inherited set), `model`, `permissionMode`, `maxTurns`, `skills` (preloaded into context at startup), and `isolation: worktree` (runs in its own git worktree).

**Where they live**, broadest to narrowest:

| Location | Scope |
| --- | --- |
| Managed settings | Whole org |
| `.claude/agents/` | This project — commit it, shared with the team |
| `~/.claude/agents/` | All your projects |
| Plugin's `agents/` directory | Wherever the plugin is enabled |

**Invocation** escalates from one-off to session-wide:

1. Natural language — "use the code-reviewer subagent" — Claude decides, based on the `description` field matching the task.
2. `@`-mention — `@agent-code-reviewer look at the auth changes` — guarantees it runs.
3. Session-wide — `claude --agent code-reviewer`, or `"agent": "code-reviewer"` in settings.

**What a fresh subagent starts with:** its own system prompt, the delegation message from the main conversation, the full CLAUDE.md hierarchy, a git status snapshot, and any preloaded skills. It does **not** see the conversation history, files already read, or output style preferences — it starts cold except for what's listed above.

## Gotchas

- **A fork is not a subagent.** A fork inherits the entire conversation — same system prompt, full message history, same model and permissions. Everything else starts fresh. Use a fork when the side task needs too much background to explain from scratch; use a regular subagent when the isolation is the point.
- **Tool restrictions don't apply to forks.** `tools` and `disallowedTools` in frontmatter are ignored for forks — they always inherit the main conversation's exact tool set.
- **`description` is a shared budget, not per-file.** All subagent descriptions combined have a 15,000-token limit. Keep the description short — it's just the trigger; the real instructions belong in the markdown body, which only loads when that subagent actually runs.
- **Permission mode can be overridden upward.** If the main conversation is running in `bypassPermissions`, `acceptEdits`, or `auto`, a subagent inherits that mode regardless of what its own frontmatter says.
- **Subagents can spawn subagents**, up to a depth limit (3 by default) and a concurrency limit (20 at once by default) — worth knowing before assuming a chain of delegation just keeps working.
- **They don't see each other.** Two subagents running in parallel have no visibility into each other's work; only the main conversation sees everyone's results.

## Sources

- [Claude Docs — Subagents](https://code.claude.com/docs/en/sub-agents)

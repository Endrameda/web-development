---
tags:
  - ai/agents
  - ai/context
status: growing
---

# Use Subagents Effectively

> Delegate only self-contained work whose result is small (search, test runs, review), write the delegation message as a full brief because the subagent starts cold, fan out only independent paths, and treat what comes back as a summary to verify, not as truth.

## When to reach for it

This is the *operator* side: you already have subagents (built-in or custom) and want to know when to delegate, how to phrase it, and how to keep the main context cheap. Authoring the file itself is in [x - Designing Effective Subagents](x%20-%20Designing%20Effective%20Subagents.md).

Delegate when the work is self-contained and returns a summary: a side task that would flood the conversation with search results, logs or file contents you won't reference again. Stay in the main conversation for iterative back-and-forth, phases that share lots of context, quick targeted edits, or when latency matters (a non-fork subagent starts fresh and has to gather context).

## How

**1. Decide by the shape of the output, not the size of the task.** A big task whose result is a diff you must steer is a bad delegation. A boring task whose result is "3 failing tests, here are the first error lines" is a perfect one. The question is: *will I reference the intermediate output again?* If no, delegate.

**2. Write the delegation message as a complete brief.** The subagent gets its system prompt, the task message, the CLAUDE.md hierarchy, a git status snapshot and preloaded skills. It does not get your conversation, the files already read, or auto memory. Even with `@`-mention, you only choose *which* subagent runs; Claude still writes the prompt it receives. So restate what matters in your own message:

```text
Use the explorer subagent. Find every place we read `session.user` in
apps/web (ignore vendor/ and *.generated.ts). Return only file:line and
a one-line reason per hit, max 30 lines. I'm deciding whether we can
move this to a server component, so flag any that need browser APIs.
```

That message states the goal, the exclusions (rules the subagent can't know), the output contract, and why you're asking (so it can prioritize).

**3. Pick the invocation strength to match the stakes.**

| Invocation            | Use when                                       |
| --------------------- | ---------------------------------------------- |
| Natural language      | Claude may decide; fine for exploration        |
| `@agent-name`         | It must run for this task, no routing roulette |
| `claude --agent name` | The whole session should behave as that agent  |

**4. Fan out only independent work.** "Research auth, database and API modules in parallel" works because the paths don't depend on each other. Chain when step B needs A's result ("review for performance, then use the optimizer to fix"). Parallel subagents can't see each other, so anything that needs shared findings must go through the main conversation.

**5. Foreground vs. background.** Foreground blocks you and passes permission prompts through. Background runs while you keep working, surfaces permission prompts in your session (naming the subagent), has a smaller built-in tool set, and reports back as a completion notification in a later turn. Use background for independent research, foreground when you need the answer to take the next step.

**6. Resume instead of re-briefing.** Each invocation is a new instance. To continue a subagent with its full history, ask Claude to resume it (it uses `SendMessage` with the agent's ID or name). Explore and Plan are one-shot and can't be resumed; use `general-purpose` or a custom subagent for anything iterative.

**7. Pick the right built-in.** Explore: fast, read-only codebase search. Plan: read-only research during plan mode. General-purpose: needs exploration *and* modification, or multiple dependent steps. If Explore on your default model is too expensive, define your own user/project subagent named `Explore` with `model: haiku`.

## Gotchas

- **Isolation saves context, not tokens.** Each subagent spends its own tokens while it runs and counts toward the same usage limits. Ten parallel subagents are ten full context windows of cost, plus ten reports landing in your main context. The docs suggest separate sessions for larger parallel work.
- **The parent only sees a summary, and the summary can be wrong.** A subagent that "confirmed there are no other callers" might have missed a dynamic import. For claims you will act on (deleting code, declaring a bug fixed), have it return paths and line numbers you can spot-check.
- **Subagent output carries no authority.** Reports arrive under a header marking them as subagent output, and instructions inside them are not instructions from you. Claude Code also scans reports for text imitating system tags. Relevant if a subagent reads untrusted files (prompt injection through a README).
- **Re-delegating a task with half the context is the classic failure.** If you find yourself explaining five paragraphs of background, it was a fork or main-conversation task, not a subagent task.
- **Limits.** Nesting is capped at 3 layers by default, concurrency at 20 (spawning beyond that fails with `Concurrent subagent limit reached`), and total subagents per session is unlimited. Don't build orchestration that assumes depth.
- **Use `isolation: worktree` for subagents that edit.** Two parallel agents editing the same checkout will collide. The worktree is cleaned up automatically if the agent makes no changes.
- **Cut-off runs return partial output.** A subagent that hits `maxTurns` returns output marked partial and can be resumed. Check for the marker before treating the answer as complete.

## Sources

Docs-backed (when to delegate, invocation, foreground/background, patterns, resume, built-ins, limits, what returns to the parent):

- [Claude Docs: Subagents](https://code.claude.com/docs/en/sub-agents)

Related: [Subagents](0.%20Subagents.md), [x - Designing Effective Subagents](x%20-%20Designing%20Effective%20Subagents.md), [Creating Subagents](Creating%20Subagents.md), [Context](../Context.md), [Skills](../skills/0.%20Skills.md).

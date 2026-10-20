---
tags:
  - ai/agents
  - ai/architecture
status: growing
---

# Designing Effective Subagents

> A good subagent is one narrow job, a `description` written as a routing trigger, the fewest tools that job needs, and a system prompt that carries the whole brief and fixes the shape of the answer — because it starts cold and the parent only ever sees its final summary.

## When to reach for it

Reach for this when you're about to write a subagent file, or when an existing one never gets picked, does too much, or returns a wall of text that defeats the point of isolating it.

The docs give two triggers for a subagent at all: a side task that would flood the main context with output you won't reuse, or a worker you keep re-spawning with the same instructions. Stay in the main conversation for iterative back-and-forth, phases that share lots of context, quick targeted edits, or when latency matters (a non-fork subagent starts fresh and has to gather context).

## How

Five decisions, in this order.

**1. One job, one output.** Design around a single task and a single result shape ("failing tests with file and first error line"), not a role ("QA engineer"). The docs' own advice is "design focused subagents: each should excel at one specific task."

**2. `description` is the router, not the manual.** Claude reads every subagent's description to decide whether to delegate, so write it as *when to use me* plus a one-clause *what I return*. "Use proactively" nudges Claude to delegate without being asked. All non-built-in descriptions share one budget (15,000 tokens before a startup warning), so detail belongs in the body, which only loads when the agent actually runs. Descriptions also have to differ from each other, or routing is a coin flip.

**3. `tools` is the capability boundary.** Omit it and the subagent inherits every tool available to subagents, MCP tools included. Allowlist with `tools`; use `disallowedTools` to subtract from the inherited set (applied first). Both are coarse: `disallowedTools: Bash(git push *)` still removes all of Bash. For "Bash, but only read-only SQL" use a `PreToolUse` hook that exits with code 2, see [Hooks](../hooks/0.%20Hooks.md).

**4. `model` is a cost/quality dial.** Resolution order: the per-invocation `model` Claude passes, then frontmatter (`inherit` = parent's model), then `CLAUDE_CODE_SUBAGENT_MODEL`, then the parent's model. The docs call out Haiku for cheap, fast exploration; `inherit` (or a stronger model) for work where reasoning quality is the bottleneck.

**5. The body is a complete brief.** The subagent gets only that system prompt plus environment details, not Claude Code's own system prompt and not your conversation. Write the workflow (numbered steps), the criteria, and an explicit output contract: what to include, how long, in what order. The summary is the only thing that crosses back into the parent context.

Example, a subagent that exists purely to keep test output out of the main context:

```markdown
---
name: test-runner
description: Runs the test suite and returns only failures (file, test name, first error line). Use proactively after code changes, before declaring a task done.
tools: Bash, Read, Grep, Glob
model: haiku
---

You run the project's tests and report failures. Do not fix anything.

1. Find the test command (package.json scripts, CI config).
2. Run it once; if it exceeds 5 minutes, report that instead of retrying.
3. Return ONLY: pass/fail counts, then one line per failure:
   `path/to/file.test.ts > test name: first line of the error`
4. If everything passes, say so in one line. Never paste full logs.
```

Why each line is there: `tools` has no `Edit`/`Write`, so "do not fix anything" is enforced rather than hoped for; `haiku` is my call (running a command and filtering output is not a reasoning-heavy job); step 3 is the output contract that protects the parent's context.

#### Tooling
Not every subagent need access to every tool. Think about what a subagent actually needs to do, and only give it the tools required for that job. This does two things: it prevents unintended side effects, and it makes each subagent's role clearer when you have several of them.

Here's how to think about tool access for common subagent types:

- **Research / read-only subagent** -- Only needs `Glob`, `Grep`, and `Read`. Cannot accidentally modify files.
- **Code reviewer** -- Needs `Bash` access to run `git diff` and see what changed, but still doesn't need `Edit` or `Write`.
- **Styling / code modification agent** -- This is where you give `Edit` and `Write` access, because the subagent's job is to actually change your code.

## Gotchas

- **A typo'd field silently widens access.** Unrecognized frontmatter fields are ignored (docs), so `tool: Read` instead of `tools: Read` means the agent inherits everything. Validate with `claude plugin validate .claude/agents` and read the file back. The consequence is my inference from that docs rule.
- **Cold start means restating rules.** The subagent gets CLAUDE.md files (except Explore, Plan, and agents with `omitClaudeMd`), but not your conversation, the files already read, or auto memory. Put non-negotiable constraints in the body and in the delegation message (docs advise restating critical rules).
- **Many subagents with verbose results bloat the parent anyway.** Isolation only pays if the return value is small. Parallel fan-out multiplies the summaries landing in the main context. Cap output length in the body.
- **Vague or overlapping descriptions cause under-delegation or the wrong pick.** A description that reads like a job title ("senior reviewer") gives Claude nothing to match against a task. Name the trigger ("after modifying code"). That framing is my heuristic; the docs only say to be specific and to use "use proactively".
- **Name collisions resolve by priority, not by recency.** Managed settings > `--agents` flag > `.claude/agents/` > `~/.claude/agents/` > plugins. A project agent shadows your personal one of the same name.
- **Plugin subagents ignore `hooks`, `mcpServers` and `permissionMode`.** If your design relies on a hook for tool gating, it works as a project or user agent, not shipped in a plugin.
- **A subagent can spawn subagents** (up to 3 layers by default). If the agent shouldn't, omit `Agent` from `tools`.
- **Resuming is per instance.** Each invocation is a new instance. Explore and Plan are one-shot; a custom subagent can be resumed with its full history, so anything you expect to continue should be custom.

## Sources

Docs-backed (when to use, description, tools, model order, context isolation, nesting, name priority, plugin limits, `omitClaudeMd`):

- [Claude Docs: Subagents](https://code.claude.com/docs/en/sub-agents)

My own judgment, not in the docs: the "one job, one output" framing, the model pick in the example, the typo consequence, and the description-as-trigger heuristic.

Related: [Subagents](0.%20Subagents.md), [Creating Subagents](Creating%20Subagents.md), [Context](../Context.md), [Claude MD](../Claude%20MD.md).

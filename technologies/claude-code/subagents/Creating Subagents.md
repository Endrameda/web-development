---
tags:
  - ai/architecture
  - ai/agents
status: seed
---

# Creating Subagents

> Describe the subagent you want to Claude Code directly — in plain language, including where to save it — and Claude writes the frontmatter and system prompt for you; you review and adjust from there.

## When to reach for it

Reach for this the moment you catch yourself wanting the same kind of delegated task twice — a read-only code reviewer, a test runner that only reports failures, a researcher that explores without editing. Don't hand-write the frontmatter from memory; describing the subagent to Claude and having it draft the file is the documented, faster path, and it gets the field names right on the first try.

## How

**Step 1 — ask Claude to create it.** Describe what the subagent should do and where it should live:

```text
Create a personal code-improver subagent in ~/.claude/agents/ that scans
files and suggests improvements for readability, performance, and best
practices. It should explain each issue, show the current code, and
provide an improved version. Make it read-only and have it use Sonnet.
```

Claude writes the file with a `name`, a `description`, a `tools` list, a `model`, and a system prompt — you don't write the YAML by hand.

**Step 2 — review the file.** The result looks like this:

```markdown
---
name: code-improver
description: Scans files and suggests improvements for readability, performance, and best practices. Use after writing or modifying code.
tools: Read, Grep, Glob
model: sonnet
---

You are a code improvement specialist. For each issue you find, explain
the problem, show the current code, and provide an improved version.
```

Saved at `~/.claude/agents/code-improver.md`. Because it's in `~/.claude/agents/` (not a project's `.claude/agents/`), it's available in *every* project on your machine — move it into a project's `.claude/agents/` to scope it to just that project.

**Step 3 — try it out.** Ask Claude to delegate to it by name:

```text
Use the code-improver agent to suggest improvements in this project
```

The delegation shows up in the transcript as a tool-call row like `code-improver(Suggest code improvements)`.

## Gotchas

- **A brand-new `~/.claude/agents/` directory needs a restart.** If that folder didn't exist when the session started, Claude won't see a subagent you just saved there until you restart Claude Code — a running session doesn't watch for a newly created `agents` directory.
- **`/agents` isn't a builder anymore in current versions.** It just reminds you to ask Claude or edit the files directly. Only Claude Code v2.1.197 and earlier opens the old interactive wizard (Running / Library tabs).
- **Location decides scope, not a flag.** There's no "make this project-only" setting — where you save the file *is* the scope. `~/.claude/agents/` = every project; `.claude/agents/` inside a repo = that project only.

## Sources

- [Claude Docs — Quickstart: Create your first subagent](https://code.claude.com/docs/en/sub-agents#quickstart-create-your-first-subagent)

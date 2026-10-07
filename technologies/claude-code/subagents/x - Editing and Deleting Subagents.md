---
tags:
  - ai/architecture
  - ai/agents
status: seed
---

# Editing and Deleting Subagents

> Claude Code watches your agent directories and picks up edits within seconds, no restart needed in the common case — but there's no delete command: removing a subagent just means deleting its file, and built-in subagents (Explore, Plan, General-purpose) can't be deleted at all, only disabled.

## When to reach for it

Reach for this when you're iterating on a subagent's prompt or tool list and aren't sure whether your change actually took effect, or when you want to retire one and aren't sure if "just delete the file" is really the whole story (it mostly is).

## How

**Editing.** Claude Code watches `~/.claude/agents/` and `.claude/agents/`. Edit the file yourself, or ask Claude to edit it — either way, the next delegation uses the updated definition within a few seconds, no restart.

**Three cases still need a restart**, because the watcher only covers what existed when the session started:

- Creating the *first* agent file in a brand-new `agents` directory — the directory itself didn't exist at session start, so the watcher never attached to it.
- Editing agents inside a directory added mid-session via `--add-dir` / `/add-dir` — those aren't watched at all.
- Any session started with `--disable-slash-commands` — it doesn't watch these directories, period.

**Validating frontmatter.** Run `claude plugin validate <path>` (e.g. `claude plugin validate .claude/agents`) to find files whose frontmatter fails to parse. Needs Claude Code v2.1.233+. It won't catch a file that parses fine but is missing `name` — that's a different failure mode.

**Deleting.** There's no delete command in current versions — a subagent is just a markdown file, so removing it is removing the file: `~/.claude/agents/<name>.md` for user-level, `.claude/agents/<name>.md` for project-level. On Claude Code v2.1.197 and earlier, `/agents` opened an interactive wizard with a Library tab that could create, edit, and delete; in current versions `/agents` just reminds you to ask Claude or edit the files directly.

**Built-in subagents** (Explore, Plan, General-purpose) can't be deleted this way — they're not files you own. Instead:

- Block one specific built-in type by adding it to `permissions.deny`.
- Block delegation entirely by denying the `Agent` tool itself.
- Disable Explore and Plan specifically with `CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS=1`.

## Gotchas

- **Deleting one that's still referenced isn't destructive to the session, just degraded.** If you resume a session that named a subagent you've since deleted, it continues with the default tools and shows a warning that the agent is no longer available — it doesn't error out.
- **The three restart cases are easy to forget mid-session** — if a newly created or just-added subagent doesn't show up, that's the first thing to check before assuming something's broken.
- **No undo.** Deleting is just `rm`-ing a markdown file — there's no trash, no confirmation, nothing Claude Code–specific to rely on.

## Sources

- [Claude Docs — Subagents: Editing and Deleting Subagents](https://code.claude.com/docs/en/sub-agents)

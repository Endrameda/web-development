---
tags:
  - ai/architecture
status: seed
---

# CC - Claude MD

> A markdown file Claude Code reads at the start of every session to get persistent instructions — coding standards, project layout, conventions — so you stop re-explaining the same things every conversation.

## When to reach for it

Add to it the moment you'd otherwise repeat yourself: Claude makes the same mistake twice, a code review catches something Claude should already know about this codebase, or you type the same correction into chat two sessions in a row. Keep it to facts that should be true in *every* session — build commands, "always do X" rules, project layout. A multi-step procedure that only matters for one part of the codebase belongs in a **skill** or a **path-scoped rule**, not CLAUDE.md.

CLAUDE.md is not the only memory mechanism. **Auto memory** is the other half: Claude writes its own notes about your corrections and preferences without you asking, stored per-project in `~/.claude/projects/<project>/memory/`. The split is simple — you write CLAUDE.md, Claude writes auto memory. Use CLAUDE.md when you want to actively steer behavior; let auto memory pick up the slack for things you'd forget to write down.

## How

**Where files live**, in load order from broadest to most specific (a project instruction is read *after* a user instruction, so it has the "last word" in context):

| Scope | Location | Shared with |
| --- | --- | --- |
| Managed policy | `/etc/claude-code/CLAUDE.md` (Linux), platform-equivalent elsewhere | Whole org, can't be excluded |
| User | `~/.claude/CLAUDE.md` | Just you, every project |
| Project | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team, via git |
| Local | `./CLAUDE.local.md` | Just you, this project — gitignore it |

Files in the directory tree above your working directory load at launch; files in subdirectories load on demand, only when Claude reads a file there. Run `/context` in a session to see which memory files actually loaded — if one's missing from that list, Claude can't see it.

**Bootstrapping:** run `/init` and Claude scans the codebase and writes a starting CLAUDE.md (build commands, conventions it can detect). It won't overwrite an existing one — it proposes improvements instead.

**Imports:** pull other files in with `@path/to/file` anywhere in the note — `@README`, `@package.json`. Relative paths resolve against the file doing the importing, not your cwd, and imports can nest up to 4 levels deep. Wrap a path in backticks (`` `@README` ``) to mention it without importing it.

**Size:** keep each file under ~200 lines. Longer files cost more context per session and Claude follows them less reliably — split with imports or path-scoped rules (`.claude/rules/*.md`, scoped via `paths:` frontmatter) instead of one giant file.

**Editing it live:** run `/memory` to list and open every memory file (CLAUDE.md, CLAUDE.local.md, auto memory) from inside a session. Or just ask — "add this to CLAUDE.md" works.

## Gotchas

- **It's context, not config.** CLAUDE.md is delivered as a message, not baked into the system prompt — Claude tries to follow it, but nothing enforces it. For a rule that must *always* hold (block a command, run something before every commit), use a hook instead, not CLAUDE.md.
- **`AGENTS.md` can silently win.** If your repo has an `AGENTS.md` and no `CLAUDE.md`/`CLAUDE.local.md` anywhere above your working directory, Claude reads `AGENTS.md` by default instead. The moment either kind of CLAUDE.md file exists on the path, it wins and `AGENTS.md` is ignored — unless you set **Project instructions** to read both.
- **Vague instructions get followed inconsistently.** "Format code properly" is noise; "use 2-space indentation" is a rule. Same for conflicting instructions across files — Claude may pick either one arbitrarily, so periodically prune stale or contradictory lines (`/doctor prompt-audit` does this for you).
- **Imports still cost context.** Splitting into `@path` imports helps organization, not token budget — every imported file still loads in full at launch alongside the CLAUDE.md that references it.
- **Root CLAUDE.md survives `/compact`**, since Claude re-reads it from disk after compaction. Nested CLAUDE.md files and path-scoped rules don't reload automatically — only when Claude next touches a matching file. An instruction given only in chat is gone after compaction unless it's written down.

## Sources

- [Claude Docs — How Claude remembers your project](https://code.claude.com/docs/en/memory)

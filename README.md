# web-development-vault

A working cheat sheet for web engineering, kept as an Obsidian vault and readable straight from
GitHub. Open it, get the answer, and follow the source link when you need the full story.

Written by me, for me, on the way from senior frontend engineer to principal. Nothing in here is
copy-pasted — if a note could be replaced by a link to MDN, it gets deleted and replaced by that
link.

## Start here

| Area                                                                           | What it covers                                                                            |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| [Web Platform](web-platform/0.%20Web%20Platform.md)                            | Browser internals: DOM, event loop, rendering pipeline, HTTP, caching, storage, workers.  |
| [Frontend Architecture](frontend-architecture/0.%20Frontend%20Architecture.md) | Patterns above the framework: state, data fetching, real-time, rendering, design systems. |
| [Technologies](technologies/0.%20Technologies.md) | Framework- and tool-specific knowledge: what React (and whatever comes next) does with the concepts. |
| [System Design](system-design/0.%20System%20Design.md)                         | Design across the client/server boundary: APIs, BFFs, caching layers, queues.             |
| [Languages & Runtimes](languages/0.%20Languages%20&%20Runtimes.md)             | JS/TS semantics, type-system depth, Node runtime behaviour.                               |
| [Engineering Practice](engineering-practice/0.%20Engineering%20Practice.md)    | Testing, performance, accessibility, security, CI/CD, observability.                      |
| [CS Fundamentals](cs-fundamentals/0.%20CS%20Fundamentals.md)                   | Data structures, algorithms, complexity, and the maths underneath.                        |
| [AI](ai/0.%20AI.md)                                                            | LLM tooling and how it changes the workflow.                                              |
| [Reading](reading/0.%20Reading.md)                                             | What I'm reading, and the one line I took from it.                                        |

Each folder's `0. …` note is its index: a table of what's written, what each note answers, and an
explicit list of the gaps still to fill. That's the entry point — not this file, which stays short
on purpose so it doesn't go stale.

[`inbox/`](inbox/0.%20Inbox.md) holds raw capture and the weekly review checklist.
`templates/` holds the Obsidian note template.

## How a note is shaped

A cheat sheet is only useful if the answer is above the fold. Every note follows the same shape:

```markdown
---
tags: [frontend/rendering]
status: growing
---

# Virtual DOM

> One sentence. The answer you came here for, before any explanation.

## When to reach for it
## How
## Gotchas
## Sources
```

The blockquote directly under the title is the load-bearing part. Obsidian's hover preview shows
it without opening the note, so most lookups end there.

## Conventions

**Frontmatter — exactly two fields.** `tags` and `status`, nothing more.

```bash
git log --diff-filter=A --format=%as -- "path/to/note.md"   # created
git log -1 --format=%as -- "path/to/note.md"                # last touched
```

**Status.** `seed` (captured, not understood — opening it will disappoint you) → `growing`
(rewritten in my own words, has a concrete example) → `evergreen` (I could teach it, states the
trade-off, links to at least two other notes). Search `[status:seed]` in Obsidian for the backlog
of things collected but not actually learned. A seed note says so in its own blockquote, so the
cheat sheet never pretends to have an answer it doesn't.

**Tags.** Hierarchical, about concepts rather than folders: `#frontend/state`,
`#web-platform/networking`. Folders answer "where does this live"; tags answer "what else is like
this". Use Obsidian's autocomplete rather than typing from memory — that's the only thing stopping
`#frontend/state` and `#frontend/state-management` from both existing.

**Links.** Standard markdown, relative paths — `[Server-Sent Events](../web-platform/Server-Sent%20Events.md)`.
Obsidian is configured to emit these (`useMarkdownLinks: true`), so they resolve in the app *and*
render as real links on github.com. Wikilinks would break the second half of that.

**File names.** Title Case with spaces, so the sidebar and graph read like English. Never use
`[ ] # ^ |` — Obsidian reserves them. Index notes are prefixed `0. ` to pin them to the top of
their folder in both Obsidian and GitHub's file browser; the prefix never appears in link text.

**Folders.** Lowercase kebab-case, two levels maximum. A subfolder only appears once its parent
passes roughly 20 notes — before that, tags do the job better without forcing a note into one
bucket.

## Working rules

- Capture into `inbox/` during the week. No rules, just get it down.
- Weekly: drain the inbox, promote what's worth keeping, add the summary line, add the source.
- A note isn't finished until it links somewhere.
- Anything still `seed` after a month gets written properly or deleted. A cheat sheet full of
  empty headings is worse than a small one that's true.

## Focus

Depth in the areas that actually decide architecture:

1. Browser and network fundamentals, deep enough to reason about performance from first
   principles rather than from a profiler screenshot.
2. Frontend architecture patterns **with their failure modes**, not just their happy paths.
3. Enough backend and distributed-systems literacy to design across the boundary, not up to it.
4. CS fundamentals rebuilt properly — algorithms, complexity, and the maths under them.
5. Written technical influence: real RFCs and ADRs. There is deliberately no folder for this
   yet — it gets created the day the first real one exists, not as a placeholder before then.

Points 1–4 make the work correct. Point 5 makes it visible.

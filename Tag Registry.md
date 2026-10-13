---
tags:
  - moc
status: growing
---

# Tag Registry

> Every tag in use and what it means. Check here before adding one, so `frontend/state` and `frontend/state-management` never both exist.

- Form is `area/topic`, lowercase kebab-case. `moc` is the one flat exception, for `0. …` index notes only.
- No synonym or singular/plural variants of a registered tag.
- A new tag gets its row here in the same commit that first uses it.

| Tag                        | Use for                                                                                                      |
| -------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `moc`                      | Index notes (`0. …`) only.                                                                                   |
| `ai/agents`                | Agent concepts: subagents, delegation, orchestration.                                                        |
| `ai/architecture`          | How LLM systems are wired: loops, retrieval, memory files. Broad; split it if it keeps absorbing everything. |
| `ai/context`               | What goes into the context window and how it is managed.                                                     |
| `ai/prompting`             | Prompt structure and technique.                                                                              |
| `distributed-systems/crdt` | CRDTs and conflict-free replication.                                                                         |
| `frontend/data-fetching`   | Fetching, pagination, caching of server data on the client.                                                  |
| `frontend/real-time`       | Polling, SSE, WebSockets and live-update patterns.                                                           |
| `frontend/rendering`       | Rendering strategies and the render pipeline in the framework layer.                                         |
| `frontend/state`           | Where client and server state live and who owns them.                                                        |
| `frontend/ux`              | UX behaviour that has an architectural cost (e.g. optimistic updates).                                       |
| `react/hooks`              | React hooks.                                                                                                 |
| `react/patterns`           | React component patterns.                                                                                    |
| `web-platform/networking`  | Browser-level networking and HTTP transport.                                                                 |

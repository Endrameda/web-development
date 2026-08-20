---
tags: [frontend/real-time]
status: seed
---

# Real-Time Update Patterns

> Three ways a client learns that server state changed. Same trade-off axis every time: freshness vs connection cost vs infrastructure complexity.

The three ways a client finds out that server state changed. Trade-off is always the same axis:
freshness vs. connection cost vs. infrastructure complexity.

## Polling

Short polling.

## Server-Sent Events

One-way, server → client, over plain HTTP. See [Server-Sent Events](../web-platform/Server-Sent%20Events.md).

## WebSockets

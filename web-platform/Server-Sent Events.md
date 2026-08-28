---
tags: [web-platform/networking, frontend/real-time]
status: growing
---

# Server-Sent Events

> One-way push, server → client, over a plain HTTP connection. You own the connection lifecycle and the heartbeat.

Here we have an example how we can setup base SSE with js.

We have to keep in mind that we should handle the connections, and usually need set up heartbeat for it. 

On FE it will look like this, in example of react useEffect()
Can use [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)

```TSX
useEffect(() => {
    if (typeof window === "undefined") {
      return;
    }

    const es = new EventSource(`/api/board/${boardId}/events`);

    const parseEventData = (raw: string) => {
      try {
        return JSON.parse(raw) as Record<string, unknown>;
      } catch (err) {
        console.error("Failed to parse SSE payload", err);
        return null;
      }
    };

    es.addEventListener("card-assigned", (event) => {
      const data = parseEventData(event.data);
      if (!data) return;

      const cardId = data.id as string;
      const user = data.assignee as { id: number } | undefined;
      if (user) {
        upsertUser(user);
        updateCard(cardId, { assigneeId: user.id });
      } else {
        updateCard(cardId, { assigneeId: undefined });
      }
    });

    es.addEventListener("card-updated", (event) => {
      const data = parseEventData(event.data);
      if (!data) return;

      const cardId = data.id as string;
      const title = data.title as string | undefined;
      const description = data.description as string | undefined;
      const user = data.assignee as { id: number } | undefined;

      if (user) {
        upsertUser(user);
      }

      updateCard(cardId, {
        title,
        description,
        assigneeId: user?.id,
      });
    });

    es.onerror = () => {
      console.error("SSE connection error");
    };

    return () => {
      es.close();
    };
  }, [boardId, upsertUser, updateCard]);
```

And on the BE side we should listen this endpoint, and send request 

```TS
// GET /api/board/:id/events  (SSE)
app.get("/api/board/:id/events", async (req, res) => {
  const boardId = req.params.id;

  // SSE headers
  res.writeHead(200, {
    "Content-Type": "text/event-stream",
    "Cache-Control": "no-cache",
    Connection: "keep-alive",
  });

  // helper to send one event
  const sendEvent = (eventType: string, data: any) => {
    res.write(`event: ${eventType}\n`);
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  };

  // initial connection event
  sendEvent("connected", { boardId });

  // listeners (same event name as your emitter usage)
  const handleCardAssigned = (data: any) => {
    sendEvent("card-assigned", data);
  };

  const handleCardUpdated = (data: any) => {
    sendEvent("card-updated", data);
  };

  mockEventEmitter.on("card-assigned", handleCardAssigned);
  mockEventEmitter.on("card-updated", handleCardUpdated);

  // heartbeats (keeps proxies happy)
  const heartbeat = setInterval(() => res.write(":keep-alive\n\n"), 15_000);

  // cleanup on disconnect
  req.on("close", () => {
    clearInterval(heartbeat);
    mockEventEmitter.off("card-assigned", handleCardAssigned);
    mockEventEmitter.off("card-updated", handleCardUpdated);
    res.end();
  });
});
```

## Sources

- [MDN — EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)

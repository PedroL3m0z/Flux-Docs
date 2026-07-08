---
title: Events
description: The event types and how to consume them — live in the dashboard or as an SSE/webhook stream.
---

Every instance emits **normalized events** when something happens — a new message,
an edit, a read receipt, a status change. You can consume them two ways: a
realtime **SSE stream**, or durable [**webhooks**](/Flux-Docs/webhooks/).

## Event types

| Type | Fires when | Payload (summary) |
| --- | --- | --- |
| `session.status` | An instance connects, disconnects, errors or is revoked | `{ status, username?, phone? }` |
| `message.new` | A message is received or sent | `MessageView` |
| `message.edited` | A message is edited | `MessageView` |
| `message.deleted` | One or more messages are deleted | `{ chat?, tgMessageIds[] }` |
| `message.read` | A read receipt ("seen") | `{ chat, maxId, direction }` |
| `message.reaction` | A reaction is added or removed | `{ chat, tgMessageId, reactions[] }` |

:::note[Read direction]
In `message.read`, `direction: "outbound"` means the recipient read **your**
message (the classic "seen"); `"inbound"` means you read theirs.
:::

Each event is wrapped in a `DomainEvent` envelope (`instanceId`, `type`, `at`,
`payload`). Exact payload shapes are in
[Types & contracts](/Flux-Docs/types/#event-payloads).

## Consume events

### In the dashboard

Open an instance's chats — new messages, edits, reactions and read receipts appear
**live** as they arrive, powered by the same event stream. No setup needed.

### Via the API (SSE)

Open a Server-Sent Events connection and process each `data` payload as it
arrives.

| Stream | Route |
| --- | --- |
| New messages for one instance | `GET /telegram/instances/:id/messages/stream` |
| Status transitions for all instances | `GET /telegram/instances/status/stream` |

Both require JWT + API key. From a browser, `EventSource` can't set headers — pass
the key in the query string instead:

```js
const es = new EventSource(
  'http://localhost:3000/telegram/instances/ID/messages/stream?apiKey=API_KEY',
);
es.onmessage = (e) => console.log(JSON.parse(e.data));
```

## SSE vs. webhooks

| | SSE | Webhooks |
| --- | --- | --- |
| Direction | Server → connected client | Server → your URL |
| Best for | Live dashboards, in-app realtime | Backend integrations, automation |
| Delivery | While connected (not durable) | Durable, retried, signed |
| Event scope | Messages + status streams | Any subset of types you subscribe |

Use SSE for a UI that's open; use [webhooks](/Flux-Docs/webhooks/) when you need
guaranteed, replayable delivery to a server.

---
title: Webhooks
description: Receive Telegram events on your own endpoint — signed and retried. Manage from the dashboard or the API.
---

A **webhook** delivers events to a URL you own. You subscribe it to a subset of
[event types](/flux-docs/events/) and link it to one or more
[instances](/flux-docs/instances/); whenever a matching event happens, Flux POSTs
it to your endpoint with retries and an HMAC signature.

![Webhooks list in the dashboard, annotated](../../assets/screenshots/webhooks-annotated.png)

## Create a webhook

### In the dashboard

![Add webhook modal, annotated step by step](../../assets/screenshots/webhook-modal-annotated.png)

1. **Webhooks → Add webhook**.
2. Set a **name** and your **URL**, tick the **event types**, link the
   **instances** to listen on.
3. **Create** — the signing **secret** is shown **once**; copy it now.

### Via the API

`POST /webhooks` — JWT + API key.

| Field | Type | Required | Rules | Description |
| --- | --- | :---: | --- | --- |
| `name` | string | yes | 1–80 chars | Label for the webhook |
| `url` | string | yes | valid URL | Where deliveries are POSTed |
| `events` | string[] | yes | non-empty; each a valid [event type](/flux-docs/events/) | Which events to send |
| `instanceIds` | string[] | no | instance ids | Instances to listen on (link later if omitted) |

```json
// POST /webhooks
{
  "name": "My integration",
  "url": "https://example.com/hooks/flux",
  "events": ["message.new", "message.read"],
  "instanceIds": ["<instanceId>"]
}
```

The response is a `WebhookWithSecret` — the `secret` (prefix `whsec_`) appears
**only here**. List the subscribable types with `GET /webhooks/event-types`.

## Receiving a delivery

Flux sends a `POST` with this body:

```json
{
  "event": "message.new",
  "instanceId": "<instanceId>",
  "at": "2026-06-19T12:00:00.000Z",
  "data": { "...": "the event payload, e.g. a MessageView" }
}
```

And these headers:

| Header | Content |
| --- | --- |
| `Content-Type` | `application/json` |
| `User-Agent` | `Flux-Webhooks/1.0` |
| `X-Flux-Event` | event type (e.g. `message.new`) |
| `X-Flux-Delivery` | delivery id — use it for idempotency |
| `X-Flux-Instance` | source instance id (when applicable) |
| `X-Flux-Signature` | `sha256=<hmac-hex>` of the raw body, using the webhook `secret` |

## Verify the signature

Always verify before trusting a delivery. Sign the **raw** body with your secret
and compare in constant time:

```ts
import { createHmac, timingSafeEqual } from 'node:crypto';

function verify(rawBody: string, header: string, secret: string): boolean {
  const expected = `sha256=${createHmac('sha256', secret).update(rawBody).digest('hex')}`;
  const a = Buffer.from(expected);
  const b = Buffer.from(header);
  return a.length === b.length && timingSafeEqual(a, b);
}
```

## Delivery guarantees

- **Durable** — every attempt is stored, surviving restarts.
- **Retried with backoff** — `10s → 1m → 5m → 30m → 2h`; after **6 attempts** a
  delivery is marked `dead`.
- **Auditable** — inspect the log and re-send manually.

## Manage

### In the dashboard

![Webhook row actions, annotated](../../assets/screenshots/webhooks-row-annotated.png)

Each row: toggle **active**, view **deliveries**, **rotate secret**, **edit**,
**delete**. The deliveries log shows status, HTTP code and attempts, with a manual
resend:

![Webhook deliveries modal, annotated](../../assets/screenshots/webhook-deliveries-annotated.png)

### Via the API

| Action | Route | Method |
| --- | --- | --- |
| List your webhooks | `/webhooks` | GET |
| Get one | `/webhooks/:id` | GET |
| Update | `/webhooks/:id` | PATCH |
| Delete (and its deliveries) | `/webhooks/:id` | DELETE |
| Rotate the secret (returned once) | `/webhooks/:id/regenerate-secret` | POST |
| Link an instance | `/webhooks/:id/instances/:instanceId` | POST |
| Unlink an instance | `/webhooks/:id/instances/:instanceId` | DELETE |
| Delivery log (`?limit=`, default 50) | `/webhooks/:id/deliveries` | GET |
| Re-queue a delivery now | `/webhooks/deliveries/:deliveryId/resend` | POST |

**Update** (`PATCH /webhooks/:id`) — all fields optional:

| Field | Type | Rules | Description |
| --- | --- | --- | --- |
| `name` | string | 1–80 chars | Rename |
| `url` | string | valid URL | Change the endpoint |
| `active` | boolean | — | Enable/disable delivery |
| `events` | string[] | valid event types | Replace the subscription |

:::caution[Public URLs only]
URLs pointing at localhost, private/reserved IP ranges or cloud metadata are
rejected on create/update, and re-checked (with DNS resolution) before each
delivery. Use a publicly reachable HTTPS URL.
:::

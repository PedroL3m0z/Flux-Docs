---
title: Telegram credentials
description: Set the api_id / api_hash that power the Telegram layer — from the dashboard or the API.
---

Flux connects to Telegram over MTProto, which needs an **`api_id`** and an
**`api_hash`**. Without them, the Telegram endpoints return `503 Service
Unavailable`.

## Get your api_id / api_hash

1. Sign in at [my.telegram.org](https://my.telegram.org).
2. Open **API development tools**.
3. Create an app — you get an `api_id` (a number) and an `api_hash` (a string).

## Set them globally

These become the default for every instance.

### In the dashboard

![The Settings page with the Telegram credentials form, annotated](../../assets/screenshots/settings-annotated.png)

1. Go to **Settings**.
2. Fill in **api_id** and **api_hash**, then **Save**.
3. The api_hash shows as set but is never displayed back; leave it blank to keep
   the current one.

### Via the API

`PUT /telegram/settings` — requires JWT + API key.

| Field | Type | Required | Rules | Description |
| --- | --- | :---: | --- | --- |
| `apiId` | string | no | digits only | GramJS `api_id` |
| `apiHash` | string | no | — | GramJS `api_hash` (encrypted at rest) |

```json
// PUT /telegram/settings
{ "apiId": "123456", "apiHash": "0123456789abcdef0123456789abcdef" }
```

`GET /telegram/settings` reads them back — the **hash is never returned**, only
whether one is set:

```json
{ "apiId": 123456, "hasApiHash": true }
```

You can also set these at startup with the `TELEGRAM_API_ID` / `TELEGRAM_API_HASH`
environment variables.

## Override per instance

When creating an [instance](/Flux-Docs/instances/) you may pass `apiId` / `apiHash`
to override the global values for that account only. The effective credentials are
the **global settings merged with the instance's overrides**.

:::caution[Stored encrypted]
The `api_hash` is encrypted at rest (AES-256-GCM) and never leaves the server.
Treat your `api_id` / `api_hash` like passwords.
:::

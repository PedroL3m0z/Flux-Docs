---
title: Sessions
description: Authorize an account via QR or phone + 2FA, from the dashboard or the API, and keep it connected.
---

A **session** is the saved credential that keeps an instance logged in to
Telegram. You create one by completing a login flow once; Flux then persists it
(encrypted) and reconnects automatically on restart.

There are two ways to log in — **QR code** or **phone number** — both ending in
`authorized`.

## Connect an account

### In the dashboard

![Add instance modal with the QR / phone choice, annotated](../../assets/screenshots/instance-modal-annotated.png)

1. When creating (or connecting) an instance, pick **QR code** or **Phone
   number**.
2. **QR**: a code appears — open Telegram on your phone → **Settings → Devices →
   Link Desktop Device** and scan it.
3. **Phone**: enter your number, then the code Telegram sends you.
4. If the account has 2FA, enter the **password** when asked. Status becomes
   `authorized`.

### Via the API — QR

`GET /telegram/instances/:id/login/qr` is a **Server-Sent Events** stream. Render
each `qr` url as a QR code for the user to scan. Events, in order:

| Event | Meaning |
| --- | --- |
| `{ "type": "qr", "url": "tg://login?token=…", "expires": 30 }` | Show as a QR; refreshes periodically |
| `{ "type": "password_required" }` | The account has 2FA — submit the password |
| `{ "type": "authorized", "me": { … } }` | Done — the instance is connected |
| `{ "type": "error", "message": "…" }` | Login failed |

### Via the API — phone

Two steps, then optional 2FA:

**1. Start** — `POST /telegram/instances/:id/login/phone`. Telegram sends a code.

| Field | Type | Required | Rules | Description |
| --- | --- | :---: | --- | --- |
| `phone` | string | yes | international format `^\+[1-9]\d{6,14}$` | e.g. `+5511999999999` |

**2. Submit the code** — `POST /telegram/instances/:id/login/code`.

| Field | Type | Required | Rules | Description |
| --- | --- | :---: | --- | --- |
| `code` | string | yes | 5–6 digits | The OTP from Telegram |

Returns `{ status: "authorized", me }` **or** `{ status: "password_required" }`.

**3. 2FA (if required)** — `POST /telegram/instances/:id/login/password`. Same
endpoint for QR and phone.

| Field | Type | Required | Description |
| --- | --- | :---: | --- |
| `password` | string | yes | Your Telegram 2FA password |

## Persistence & reconnection

- The session string is stored in Redis, **encrypted at rest** (AES-256-GCM).
- On startup, every authorized instance is rehydrated and reconnected.
- A periodic health check detects sessions **revoked remotely** (e.g. you ended
  the session from the Telegram app). The instance then moves to `error` and
  clears its session — just log in again.

## Stop vs. log out

- `POST /telegram/instances/:id/stop` disconnects but **keeps** the session
  (resume later with `start`).
- `DELETE /telegram/instances/:id` removes the instance **and** its session.

See [Instances](/Flux-Docs/instances/) for start/stop/delete and status values.

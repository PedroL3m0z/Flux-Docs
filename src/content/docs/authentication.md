---
title: Authentication
description: Sign in to the dashboard or get a JWT for the API, and send the API key on every request.
---

Two layers protect Flux. Most actions require **both**:

1. **JWT** — identifies the dashboard user (from signing in).
2. **API key** — the gateway's static key, sent as the `x-api-key` header.

## Sign in

### In the dashboard

![The Flux sign-in screen, annotated](../../assets/screenshots/login-annotated.png)

1. Open `http://localhost:3000/dashboard`.
2. If prompted to **unlock**, paste the `x-api-key` — the dashboard stores it and
   sends it with every request for you.
3. Enter your **username or email** and **password**, then **Sign in**.

The browser keeps the JWT in an `httpOnly` cookie, so you stay logged in.

### Via the API

`POST /auth/login` — **public** (no headers needed).

| Field | Type | Required | Rules | Description |
| --- | --- | :---: | --- | --- |
| `username` | string | yes | username **or** email | Who you are |
| `password` | string | yes | — | Account password |

Returns `{ "accessToken": "<JWT>" }`. The token is also set as an `httpOnly`
cookie. Send it as `Authorization: Bearer <JWT>` on later calls.

```json
// POST /auth/login
{ "username": "admin", "password": "••••••••" }
```

## Sending authenticated requests

Every protected route needs these two headers:

| Header | Value | Where from |
| --- | --- | --- |
| `Authorization` | `Bearer <JWT>` | the `accessToken` from `/auth/login` |
| `x-api-key` | `<API_KEY>` | printed on first boot, or your `API_KEY` env |

:::note[Streams & media in the browser]
`EventSource` (SSE) and `<img>` tags can't send headers. For those, pass the key
as a query parameter instead: `?apiKey=<API_KEY>`.
:::

## Helper endpoints

| Route | Method | Auth | Purpose |
| --- | --- | --- | --- |
| `/auth/login` | POST | public | Sign in; returns the JWT and sets the cookie |
| `/auth/logout` | POST | public | Clear the auth cookie |
| `/auth/me` | GET | JWT only (no API key) | The current user |
| `/auth/api-key-check` | GET | `x-api-key` | `200` if the key is valid |

## What's exempt

The `auth` routes and `/health` don't need the API key. `/auth/me` needs the JWT
but not the key. Everything else needs both.

Under the hood: passwords are hashed with **Argon2id**, the API key is compared in
constant time, and requests are rate-limited per IP. To manage **who** can sign in
and what they can do, see [Accounts](/flux-docs/accounts/).

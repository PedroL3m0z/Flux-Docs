---
title: Accounts
description: Manage dashboard users and what they're allowed to do — from the dashboard or the API.
---

**Accounts** are the dashboard users who sign in to Flux — distinct from the
Telegram [instances](/Flux-Docs/instances/) they operate. Each account has one
**role** that decides what it can do. Managing accounts requires the **admin**
role.

![Users list in the dashboard, annotated](../../assets/screenshots/users-annotated.png)

## Roles & permissions

| Capability | viewer | operator | admin |
| --- | :---: | :---: | :---: |
| Read instances, chats, messages, webhooks | ✅ | ✅ | ✅ |
| Manage/delete instances, send messages & media, manage webhooks | | ✅ | ✅ |
| Manage users (create, edit, delete, set roles) | | | ✅ |

The first user, seeded on first boot, is an **admin**.

## Create a user

### In the dashboard

![Add user modal, annotated step by step](../../assets/screenshots/user-modal-annotated.png)

1. **Users → Add user**.
2. Enter **email**, **username** and **password**, then **Create**.
3. New users start as `viewer`; change the role from the edit modal (below).

### Via the API

`POST /auth/register` — admin only (JWT + API key).

| Field | Type | Required | Rules | Description |
| --- | --- | :---: | --- | --- |
| `email` | string | yes | valid email | Login email |
| `username` | string | yes | 3–32 chars | Login username |
| `password` | string | yes | 8–128 chars | Initial password |

```json
// POST /auth/register
{ "email": "jane@example.com", "username": "jane", "password": "S3cureP@ss" }
```

The new account defaults to the `viewer` role.

## Edit, change role, delete

### In the dashboard

![Edit user modal, annotated step by step](../../assets/screenshots/user-edit-modal-annotated.png)

The row pencil opens **Edit user** — change email, username, password or role in
one place. Leaving the password blank keeps the current one. You **can't** change
your own role here.

### Via the API

| Action | Route | Method |
| --- | --- | --- |
| List users | `/users` | GET |
| Change role | `/users/:id/role` | PATCH |
| Edit user | `/users/:id` | PATCH |
| Delete user | `/users/:id` | DELETE |

**Change role** (`PATCH /users/:id/role`):

| Field | Type | Required | Rules | Description |
| --- | --- | :---: | --- | --- |
| `role` | string | yes | `admin` \| `operator` \| `viewer` | New global role |

**Edit user** (`PATCH /users/:id`) — all fields optional; only sent fields change:

| Field | Type | Rules | Description |
| --- | --- | --- | --- |
| `email` | string | valid email | New email |
| `username` | string | — | New username |
| `password` | string | — | New password (omit to keep current) |
| `role` | string | `admin` \| `operator` \| `viewer` | New role |

**Delete** (`DELETE /users/:id`) cascades the user's instances and webhooks.

:::caution[Self-protection]
You **cannot** change your own role or delete your own account through these
routes — this prevents an admin from accidentally locking everyone out. Ask
another admin, or edit a different field.
:::

User records never expose the password hash. See
[Authentication](/Flux-Docs/authentication/) for how these accounts sign in.

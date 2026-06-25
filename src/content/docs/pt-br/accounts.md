---
title: Contas
description: Gerencie os usuários do dashboard e o que podem fazer — pelo dashboard ou pela API.
---

**Contas** são os usuários do dashboard que fazem login na Flux — distintos das
[instâncias](/flux-docs/pt-br/instances/) do Telegram que operam. Cada conta tem um
**papel** (role) que decide o que pode fazer. Gerenciar contas exige o papel
**admin**.

![Lista de usuários no dashboard, anotada](../../../assets/screenshots/users-annotated.png)

## Papéis e permissões

| Capacidade | viewer | operator | admin |
| --- | :---: | :---: | :---: |
| Ler instâncias, chats, mensagens, webhooks | ✅ | ✅ | ✅ |
| Gerenciar/excluir instâncias, enviar mensagens e mídia, gerenciar webhooks | | ✅ | ✅ |
| Gerenciar usuários (criar, editar, excluir, definir papéis) | | | ✅ |

O primeiro usuário, semeado no primeiro boot, é um **admin**.

## Crie um usuário

### No dashboard

![Modal de adicionar usuário, anotado passo a passo](../../../assets/screenshots/user-modal-annotated.png)

1. **Users → Add user**.
2. Informe **e-mail**, **usuário** e **senha**, então **Create**.
3. Novos usuários começam como `viewer`; mude o papel no modal de edição (abaixo).

### Pela API

`POST /auth/register` — só admin (JWT + API key).

| Campo | Tipo | Obrigatório | Regras | Descrição |
| --- | --- | :---: | --- | --- |
| `email` | string | sim | e-mail válido | E-mail de login |
| `username` | string | sim | 3–32 caracteres | Usuário de login |
| `password` | string | sim | 8–128 caracteres | Senha inicial |

```json
// POST /auth/register
{ "email": "jane@example.com", "username": "jane", "password": "S3cureP@ss" }
```

A nova conta tem como padrão o papel `viewer`.

## Editar, mudar papel, excluir

### No dashboard

![Modal de editar usuário, anotado passo a passo](../../../assets/screenshots/user-edit-modal-annotated.png)

O lápis da linha abre **Edit user** — mude e-mail, usuário, senha ou papel num só
lugar. Deixar a senha em branco mantém a atual. Você **não pode** mudar seu próprio
papel aqui.

### Pela API

| Ação | Rota | Método |
| --- | --- | --- |
| Listar usuários | `/users` | GET |
| Mudar papel | `/users/:id/role` | PATCH |
| Editar usuário | `/users/:id` | PATCH |
| Excluir usuário | `/users/:id` | DELETE |

**Mudar papel** (`PATCH /users/:id/role`):

| Campo | Tipo | Obrigatório | Regras | Descrição |
| --- | --- | :---: | --- | --- |
| `role` | string | sim | `admin` \| `operator` \| `viewer` | Novo papel global |

**Editar usuário** (`PATCH /users/:id`) — todos os campos opcionais; só os enviados mudam:

| Campo | Tipo | Regras | Descrição |
| --- | --- | --- | --- |
| `email` | string | e-mail válido | Novo e-mail |
| `username` | string | — | Novo usuário |
| `password` | string | — | Nova senha (omita para manter a atual) |
| `role` | string | `admin` \| `operator` \| `viewer` | Novo papel |

**Excluir** (`DELETE /users/:id`) cascateia as instâncias e webhooks do usuário.

:::caution[Autoproteção]
Você **não pode** mudar seu próprio papel ou excluir sua própria conta por essas
rotas — isso evita que um admin tranque todo mundo para fora por acidente. Peça a
outro admin, ou edite um campo diferente.
:::

Registros de usuário nunca expõem o hash da senha. Veja
[Autenticação](/flux-docs/pt-br/authentication/) para como essas contas fazem login.

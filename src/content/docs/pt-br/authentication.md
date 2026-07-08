---
title: Autenticação
description: Faça login no dashboard ou obtenha um JWT para a API, e envie a API key em cada requisição.
---

Duas camadas protegem a Flux. A maioria das ações exige **ambas**:

1. **JWT** — identifica o usuário do dashboard (a partir do login).
2. **API key** — a chave estática do gateway, enviada no header `x-api-key`.

## Faça login

### No dashboard

![A tela de login da Flux, anotada](../../../assets/screenshots/login-annotated.png)

1. Abra `http://localhost:3000/dashboard`.
2. Se pedirem para **desbloquear**, cole a `x-api-key` — o dashboard a guarda e a
   envia em cada requisição por você.
3. Informe seu **usuário ou e-mail** e **senha**, então **Entrar**.

O navegador mantém o JWT num cookie `httpOnly`, então você continua logado.

### Pela API

`POST /auth/login` — **público** (sem headers necessários).

| Campo | Tipo | Obrigatório | Regras | Descrição |
| --- | --- | :---: | --- | --- |
| `username` | string | sim | usuário **ou** e-mail | Quem você é |
| `password` | string | sim | — | Senha da conta |

Retorna `{ "accessToken": "<JWT>" }`. O token também é definido como cookie
`httpOnly`. Envie-o como `Authorization: Bearer <JWT>` nas chamadas seguintes.

```json
// POST /auth/login
{ "username": "admin", "password": "••••••••" }
```

## Enviando requisições autenticadas

Toda rota protegida precisa destes dois headers:

| Header | Valor | De onde vem |
| --- | --- | --- |
| `Authorization` | `Bearer <JWT>` | o `accessToken` do `/auth/login` |
| `x-api-key` | `<API_KEY>` | impressa no primeiro boot, ou sua env `API_KEY` |

:::note[Streams e mídia no navegador]
`EventSource` (SSE) e tags `<img>` não conseguem enviar headers. Para esses, passe
a chave como query parameter: `?apiKey=<API_KEY>`.
:::

## Endpoints auxiliares

| Rota | Método | Auth | Propósito |
| --- | --- | --- | --- |
| `/auth/login` | POST | público | Login; retorna o JWT e define o cookie |
| `/auth/logout` | POST | público | Limpa o cookie de auth |
| `/auth/me` | GET | só JWT (sem API key) | O usuário atual |
| `/auth/api-key-check` | GET | `x-api-key` | `200` se a chave for válida |

## O que é isento

As rotas `auth` e `/health` não precisam da API key. `/auth/me` precisa do JWT
mas não da chave. Todo o resto precisa de ambos.

Por baixo dos panos: senhas são hasheadas com **Argon2id**, a API key é comparada
em tempo constante, e as requisições têm rate limit por IP. Para gerenciar **quem**
pode logar e o que pode fazer, veja [Contas](/Flux-Docs/pt-br/accounts/).

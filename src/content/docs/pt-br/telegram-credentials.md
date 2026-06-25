---
title: Credenciais do Telegram
description: Defina o api_id / api_hash que alimentam a camada do Telegram — pelo dashboard ou pela API.
---

A Flux se conecta ao Telegram via MTProto, o que exige um **`api_id`** e um
**`api_hash`**. Sem eles, os endpoints do Telegram retornam `503 Service
Unavailable`.

## Obtenha seu api_id / api_hash

1. Faça login em [my.telegram.org](https://my.telegram.org).
2. Abra **API development tools**.
3. Crie um app — você recebe um `api_id` (um número) e um `api_hash` (uma string).

## Defina-os globalmente

Estes viram o padrão para toda instância.

### No dashboard

![A página Settings com o formulário de credenciais do Telegram, anotada](../../../assets/screenshots/settings-annotated.png)

1. Vá em **Settings**.
2. Preencha **api_id** e **api_hash**, então **Save**.
3. O api_hash aparece como definido mas nunca é exibido de volta; deixe em branco
   para manter o atual.

### Pela API

`PUT /telegram/settings` — exige JWT + API key.

| Campo | Tipo | Obrigatório | Regras | Descrição |
| --- | --- | :---: | --- | --- |
| `apiId` | string | não | só dígitos | `api_id` do GramJS |
| `apiHash` | string | não | — | `api_hash` do GramJS (criptografado em repouso) |

```json
// PUT /telegram/settings
{ "apiId": "123456", "apiHash": "0123456789abcdef0123456789abcdef" }
```

`GET /telegram/settings` os lê de volta — o **hash nunca é retornado**, apenas se
um está definido:

```json
{ "apiId": 123456, "hasApiHash": true }
```

Você também pode defini-los na inicialização com as variáveis de ambiente
`TELEGRAM_API_ID` / `TELEGRAM_API_HASH`.

## Sobrescrever por instância

Ao criar uma [instância](/flux-docs/pt-br/instances/) você pode passar `apiId` / `apiHash`
para sobrescrever os valores globais só para aquela conta. As credenciais efetivas são
as **configurações globais mescladas com os overrides da instância**.

:::caution[Armazenado criptografado]
O `api_hash` é criptografado em repouso (AES-256-GCM) e nunca sai do servidor.
Trate seu `api_id` / `api_hash` como senhas.
:::

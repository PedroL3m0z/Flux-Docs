---
title: Engines
description: Os backends do Telegram em que uma instância pode rodar.
---

Um **engine** é o backend que de fato fala com o Telegram por uma instância.
A Flux é agnóstica a engine: cada instância escolhe um pelo campo `engine`, e o
resto da API (sessões, mensagens, eventos, webhooks) se comporta igual
independentemente do engine usado.

## Engines disponíveis

| Engine | `key` | Status | O que suporta | Login |
| --- | --- | --- | --- | --- |
| **GramJS** | `gramjs` | ✅ disponível | mensageria completa (ler/enviar/receber) | QR + 2FA, telefone |
| Telegraf | `telegraf` | 🔜 reservado | bot-token (planejado) | Bot token |

`gramjs` é o padrão e roda **contas de usuário** sobre MTProto — é o que você
quer para conectar um número pessoal/comercial.

## Escolhendo um engine

Passe `engine` ao criar uma instância (o padrão é `gramjs`):

```bash
curl -X POST http://localhost:3000/telegram/instances \
  -H 'Authorization: Bearer <JWT>' -H 'x-api-key: <API_KEY>' \
  -H 'Content-Type: application/json' \
  -d '{"label":"Conta principal","engine":"gramjs"}'
```

Solicitar um engine que não está disponível retorna `400 Bad Request`.

## Capacidades

Cada engine declara o que sabe fazer — por exemplo login por QR e mensageria. A API
só expõe ações que um engine de fato suporta, então um fluxo como login por QR fica
disponível quando o engine anuncia essa capacidade. O GramJS anuncia login por QR e
mensageria completa; o engine reservado Telegraf mira a Bot API.

:::note[Para integradores]
Novos engines são adicionados implementando o contrato de engine e registrando-o —
sem mudança em instâncias, sessões, eventos ou webhooks. Da perspectiva do usuário,
um novo engine simplesmente aparece como mais uma `key` selecionável.
:::

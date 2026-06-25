---
title: Eventos
description: Os tipos de evento e como consumi-los — ao vivo no dashboard ou como stream SSE/webhook.
---

Toda instância emite **eventos normalizados** quando algo acontece — uma nova
mensagem, uma edição, um recibo de leitura, uma mudança de status. Você pode
consumi-los de duas formas: um **stream SSE** em tempo real, ou
[**webhooks**](/flux-docs/pt-br/webhooks/) duráveis.

## Tipos de evento

| Tipo | Dispara quando | Payload (resumo) |
| --- | --- | --- |
| `session.status` | Uma instância conecta, desconecta, dá erro ou é revogada | `{ status, username?, phone? }` |
| `message.new` | Uma mensagem é recebida ou enviada | `MessageView` |
| `message.edited` | Uma mensagem é editada | `MessageView` |
| `message.deleted` | Uma ou mais mensagens são excluídas | `{ chat?, tgMessageIds[] }` |
| `message.read` | Um recibo de leitura ("visto") | `{ chat, maxId, direction }` |
| `message.reaction` | Uma reação é adicionada ou removida | `{ chat, tgMessageId, reactions[] }` |

:::note[Direção da leitura]
Em `message.read`, `direction: "outbound"` significa que o destinatário leu **sua**
mensagem (o clássico "visto"); `"inbound"` significa que você leu a dele.
:::

Cada evento é envolvido num envelope `DomainEvent` (`instanceId`, `type`, `at`,
`payload`). Os formatos exatos de payload estão em
[Tipos e contratos](/flux-docs/pt-br/types/#payloads-de-evento).

## Consuma eventos

### No dashboard

Abra os chats de uma instância — novas mensagens, edições, reações e recibos de
leitura aparecem **ao vivo** conforme chegam, movidos pelo mesmo stream de eventos.
Sem configuração.

### Pela API (SSE)

Abra uma conexão de Server-Sent Events e processe cada payload `data` conforme
chega.

| Stream | Rota |
| --- | --- |
| Novas mensagens de uma instância | `GET /telegram/instances/:id/messages/stream` |
| Transições de status de todas as instâncias | `GET /telegram/instances/status/stream` |

Ambos exigem JWT + API key. Do navegador, `EventSource` não consegue definir
headers — passe a chave na query string:

```js
const es = new EventSource(
  'http://localhost:3000/telegram/instances/ID/messages/stream?apiKey=API_KEY',
);
es.onmessage = (e) => console.log(JSON.parse(e.data));
```

## SSE vs. webhooks

| | SSE | Webhooks |
| --- | --- | --- |
| Direção | Servidor → cliente conectado | Servidor → sua URL |
| Melhor para | Dashboards ao vivo, tempo real no app | Integrações de backend, automação |
| Entrega | Enquanto conectado (não durável) | Durável, com retentativa, assinado |
| Escopo de evento | Streams de mensagens + status | Qualquer subconjunto de tipos assinado |

Use SSE para uma UI aberta; use [webhooks](/flux-docs/pt-br/webhooks/) quando precisar
de entrega garantida e reproduzível para um servidor.

---
title: Sessões
description: Autorize uma conta via QR ou telefone + 2FA, pelo dashboard ou pela API, e a mantenha conectada.
---

Uma **sessão** é a credencial salva que mantém uma instância logada no
Telegram. Você cria uma completando um fluxo de login uma vez; a Flux então a
persiste (criptografada) e reconecta automaticamente ao reiniciar.

Há duas formas de logar — **QR code** ou **número de telefone** — ambas terminando
em `authorized`.

## Conecte uma conta

### No dashboard

![Modal de adicionar instância com a escolha QR / telefone, anotado](../../../assets/screenshots/instance-modal-annotated.png)

1. Ao criar (ou conectar) uma instância, escolha **QR code** ou **Número de
   telefone**.
2. **QR**: aparece um código — abra o Telegram no celular → **Configurações →
   Dispositivos → Vincular Dispositivo Desktop** e escaneie.
3. **Telefone**: informe seu número, depois o código que o Telegram envia.
4. Se a conta tiver 2FA, informe a **senha** quando pedido. O status vira
   `authorized`.

### Pela API — QR

`GET /telegram/instances/:id/login/qr` é um stream de **Server-Sent Events**.
Renderize cada url `qr` como um QR code para o usuário escanear. Eventos, em ordem:

| Evento | Significado |
| --- | --- |
| `{ "type": "qr", "url": "tg://login?token=…", "expires": 30 }` | Mostre como QR; atualiza periodicamente |
| `{ "type": "password_required" }` | A conta tem 2FA — envie a senha |
| `{ "type": "authorized", "me": { … } }` | Pronto — a instância está conectada |
| `{ "type": "error", "message": "…" }` | Login falhou |

### Pela API — telefone

Dois passos, depois 2FA opcional:

**1. Iniciar** — `POST /telegram/instances/:id/login/phone`. O Telegram envia um código.

| Campo | Tipo | Obrigatório | Regras | Descrição |
| --- | --- | :---: | --- | --- |
| `phone` | string | sim | formato internacional `^\+[1-9]\d{6,14}$` | ex.: `+5511999999999` |

**2. Enviar o código** — `POST /telegram/instances/:id/login/code`.

| Campo | Tipo | Obrigatório | Regras | Descrição |
| --- | --- | :---: | --- | --- |
| `code` | string | sim | 5–6 dígitos | O OTP do Telegram |

Retorna `{ status: "authorized", me }` **ou** `{ status: "password_required" }`.

**3. 2FA (se necessário)** — `POST /telegram/instances/:id/login/password`. Mesmo
endpoint para QR e telefone.

| Campo | Tipo | Obrigatório | Descrição |
| --- | --- | :---: | --- |
| `password` | string | sim | Sua senha 2FA do Telegram |

## Persistência e reconexão

- A string de sessão é guardada no Redis, **criptografada em repouso** (AES-256-GCM).
- Na inicialização, toda instância autorizada é reidratada e reconectada.
- Um health check periódico detecta sessões **revogadas remotamente** (ex.: você
  encerrou a sessão pelo app do Telegram). A instância então vai para `error` e
  limpa sua sessão — basta logar de novo.

## Parar vs. deslogar

- `POST /telegram/instances/:id/stop` desconecta mas **mantém** a sessão
  (retome depois com `start`).
- `DELETE /telegram/instances/:id` remove a instância **e** sua sessão.

Veja [Instâncias](/flux-docs/pt-br/instances/) para start/stop/delete e os valores de status.

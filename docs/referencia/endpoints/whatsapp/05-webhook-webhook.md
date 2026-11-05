---
id: whatsapp.05
titulo: "Webhook (`/webhook`)"
tipo: referencia
area: api
sistema: central
tags: [api, whatsapp, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 495
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## Webhook (`/webhook`)

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `POST` | `/webhook/openwa` | **gateway** (HMAC; **sem JWT**) | Corpo cru do envelope do OpenWA (limite **1 MiB**); headers `X-OpenWA-Signature` (`sha256=<hex>`), `X-OpenWA-Idempotency-Key`, `X-OpenWA-Event`, `X-OpenWA-Retry-Count`, `X-OpenWA-Delivery-Id` | `200 { aceito: true, duplicado: false }` na primeira entrega; `200 { aceito: true, duplicado: true }` em reentrega (o índice único é o que impede o reprocessamento); `401` assinatura ausente/inválida; `400` payload ilegível; `413` acima de 1 MiB; `503` módulo desabilitado ou sem segredo configurado |

A assinatura é HMAC-SHA256 do **corpo cru**, comparada em **tempo constante**. O corpo é lido em buffer **antes** de desserializar e é entregue a uma **fila durável no SQL Server**; o processamento acontece depois, no worker. A resposta é rápida de propósito: o gateway dá timeout em 10 s e repete a entrega.

**Filtro de avisos de sistema no `message.received`.** Antes de registrar a entrada, o processador descarta o payload com `isStatusBroadcast = true` e os tipos `unknown`, `notification`, `protocolmessage`, `protocol_message`, `system`, `e2e_notification`, `stub`, `stubtype` e `revoked`; o texto de aviso de serviço seguro da Meta (ex.: "Esta empresa agora usa um serviço seguro da Meta...") também é descartado, mesmo com `type = text`. Esses avisos chegam no mesmo `message.received` do contato (`fromMe = false`, com corpo vazio ou texto de aviso) e **não** criam contato, conversa, atendimento nem mensagem — sem o filtro, abriam um card na fila e contavam como não lida.


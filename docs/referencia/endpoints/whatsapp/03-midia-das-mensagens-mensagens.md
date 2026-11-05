---
id: whatsapp.03
titulo: "Mídia das mensagens (`/mensagens`)"
tipo: referencia
area: api
sistema: central
tags: [api, whatsapp, endpoints]
status: ativo
revisado_em: "2026-10-30"
fonte:
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 320
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## Mídia das mensagens (`/mensagens`)

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/mensagens/{id}/midia` | atendente | Path `id` (mensagem) | `200 Blob` (bytes da mídia, `Content-Type` validado pela whitelist `WhatsAppMidiaSeguranca`, `Content-Disposition: attachment` com nome original); `404` "Midia indisponivel (inexistente ou removida pela retencao)." |

O controller aplica whitelist de MIME: tipos fora da lista viram `application/octet-stream`. Header `X-Content-Type-Options: nosniff`. Áudio/vídeo habilita `enableRangeProcessing` para seek. Mídia inexistente **ou** já removida pela retenção respondem o mesmo `404` — não há como distinguir sem vazar detalhe.

O **envio** de mídia (imagem sem SVG, áudio, vídeo `mp4`/`webm` e GIF ≤ 10 MB — Fases 6/7 do plano 2026-10) e o envio de **cartão de contato** (sem mídia persistida, `tipo: "contact"`) são rotas do atendimento, detalhadas em [Atendimentos (`/atendimentos`)](./01-atendimentos-atendimentos.md).


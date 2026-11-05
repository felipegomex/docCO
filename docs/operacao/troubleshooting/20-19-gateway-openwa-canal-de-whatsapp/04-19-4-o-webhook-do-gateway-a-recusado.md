---
id: 20-19-gateway-openwa-canal-de-whatsapp.04
titulo: "19.4 O webhook do gateway Ã© recusado"
tipo: operacao
area: diagnostico
sistema: central
tags: [whatsapp, openwa, gateway, erro]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - scripts
testes:
  - docs/
tokens_estimados: 615
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.4 O webhook do gateway Ã© recusado

| Sintoma | Causa | SoluÃ§Ã£o |
|---|---|---|
| O registro do webhook Ã© recusado no gateway, ou a entrega nÃ£o chega | A URL do webhook aponta para **endereÃ§o privado** (`192.168.2.130`/`192.168.2.254`) e o host **nÃ£o** estÃ¡ na lista de exceÃ§Ãµes do guard de SSRF â€” por padrÃ£o o OpenWA recusa entregar (e registrar) webhook para faixa privada | Liste os dois servidores em `SSRF_ALLOWED_HOSTS=192.168.2.130,192.168.2.254` no `.env` do gateway (o `docker-compose.yml` repassa a variÃ¡vel) e reinicie o gateway |
| `401` "Assinatura invÃ¡lida" | A assinatura HMAC do **corpo cru** nÃ£o confere: segredo diferente entre o gateway e o backend, ou corpo alterado no caminho | O segredo registrado no gateway Ã© o **mesmo** valor de `WhatsApp__WebhookSecret`. O receptor recomputa HMAC-SHA256 sobre o corpo **antes** de desserializar e compara em tempo constante |
| `503` "MÃ³dulo de WhatsApp indisponÃ­vel" | MÃ³dulo desabilitado (`WhatsApp:Habilitado = false`) **ou** `WhatsApp:WebhookSecret` vazio | Ligue o mÃ³dulo no servidor (variÃ¡vel de ambiente `WhatsApp__Habilitado=true`) e defina o segredo. O `503` Ã© proposital: o gateway repete a entrega, e assim nada se perde enquanto o mÃ³dulo estÃ¡ fora |
| `413` | Payload acima do limite de 1 MiB | O gateway nÃ£o deve enviar mÃ­dia inline acima de ~768 KiB; nesse caso ele manda `{ omitted: true, sizeBytes }` |
| A entrega chega e Ã© processada **duas vezes** | Entrega *at-least-once* sem a chave de dedupe | A defesa Ã© o `X-OpenWA-Idempotency-Key` (fallback: `idempotencyKey`, `deliveryId`, hash do corpo) gravado **antes** de processar, com Ã­ndice Ãºnico em `tbwhatsapp_webhook_fila`. Reentrega responde `200 { duplicado: true }` e nÃ£o reprocessa |
| **Os dois ambientes** (HML e PROD) criam atendimentos a partir do mesmo nÃºmero | A sessÃ£o tem **mais de um** webhook ativo: o OpenWA aceita vÃ¡rios por sessÃ£o e o backend registra um a cada criaÃ§Ã£o de nÃºmero | Um gateway, **um** webhook ativo por sessÃ£o. Na virada de ambiente, desative o webhook antigo antes de ativar o novo e confira `GET /api/sessions/{id}/webhooks` |


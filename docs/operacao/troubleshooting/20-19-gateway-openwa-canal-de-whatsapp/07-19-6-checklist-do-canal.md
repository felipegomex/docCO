---
id: 20-19-gateway-openwa-canal-de-whatsapp.07
titulo: "19.6 Checklist do canal"
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
tokens_estimados: 347
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.6 Checklist do canal

1. O gateway responde em `127.0.0.1:2785` (`docker compose ps` em `~/openwa`) **e** em `192.168.2.253:2785` a partir dos dois servidores da Central.
2. `GET http://192.168.2.253:2785/api/health` com `X-API-Key` responde `200`.
3. SessÃ£o em `ready` (ou `qr_ready` aguardando o QR).
4. Um envio real **passa** (no caminho Docker, se falhar com `the engine returned no message`, confirme a versÃ£o da imagem e o log do container antes de partir para os *patchers* â€” ver Â§19.2).
5. Uma mensagem recebida cria **um** atendimento na fila (prova de que o webhook foi assinado, deduplicado e processado).
6. O backend responde `2xx` no `POST /api/v1/whatsapp/webhook/openwa` e a fila nÃ£o acumula.
7. A sessÃ£o tem **uma** URL de webhook ativa, a do ambiente em uso.
8. `GET /api/v1/whatsapp/saude` responde `200` **com a sessÃ£o ativa** â€” Ã© o que expÃµe o defeito do `lastActive` (Â§19.10.3).
9. `tbwhatsapp_webhook_fila` nÃ£o tem linha parada em `Processando` (Â§19.10.2) e `tbwhatsapp_numero.WAT_Numero` estÃ¡ preenchido (Â§19.10.1).


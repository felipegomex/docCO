---
id: 20-19-gateway-openwa-canal-de-whatsapp.06
titulo: "19.5 A fila de webhook cresce e nada Ã© processado"
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
tokens_estimados: 496
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.5 A fila de webhook cresce e nada Ã© processado

| Sintoma | Causa | SoluÃ§Ã£o |
|---|---|---|
| `tbwhatsapp_webhook_fila` acumula linhas em `Pendente`/`Erro` | O worker (`WhatsAppWebhookProcessor`) nÃ£o estÃ¡ rodando: mÃ³dulo desabilitado, serviÃ§o do backend reiniciado com o mÃ³dulo desligado, ou as linhas passaram das **6 tentativas** | Confirme `WhatsApp:Habilitado = true` no backend. O worker sÃ³ processa o que estÃ¡ habilitado. O backoff Ã© exponencial a partir de **5 s**, limitado a **10 min**; depois de 6 tentativas a linha fica em `Erro` e precisa de intervenÃ§Ã£o |
| Uma linha fica sempre em `Erro` com a mesma mensagem | Payload incompatÃ­vel com o envelope esperado | O erro fica em `WAT_Erro` (500 caracteres) e a tentativa Ã© registrada no log do backend |
| Uma linha fica **presa em `Processando`** (`WAT_Estado = 1`) e nunca mais Ã© processada | O worker pegou a linha e o **processo morreu no meio** (redeploy, recycle do pool do IIS). O evento se perde para sempre | O worker agora **se auto-recupera** (Â§19.10.2): na largada ele devolve **todas** as linhas presas e, a cada 30 s, as que estÃ£o presas hÃ¡ mais de 2 min. Para reenfileirar na mÃ£o: `UPDATE tbwhatsapp_webhook_fila SET WAT_Estado = 0, WAT_Tentativas = 0, WAT_ProximoProcessamentoEm = NULL, WAT_Erro = NULL WHERE WAT_Estado = 1;` |
| A linha fica presa em `Processando` **e** o telefone do nÃºmero nunca Ã© gravado | **NÃ£o Ã© a fila**: o handler de status gravava o **nome do evento** (`session.authenticated`, 21 caracteres) em `WAT_Status varchar(20)` e o SQL Server recusava o INSERT/UPDATE com o **erro 8152** | Â§19.10.1 â€” o nome do evento nÃ£o Ã© status |


---
id: 20-19-gateway-openwa-canal-de-whatsapp.05
titulo: "19.4.1 Aviso de sistema da Meta vira atendimento"
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
tokens_estimados: 374
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.4.1 Aviso de sistema da Meta vira atendimento

| Sintoma | Causa | Solução |
|---|---|---|
| Logo após conectar um número aparecem atendimentos com mensagem de corpo vazio (tipo `unknown`) ou com o texto "Esta empresa agora usa um serviço seguro da Meta...", sem o cliente ter escrito nada | O gateway entrega avisos de sistema/protocolo da Meta no mesmo `message.received` das mensagens do contato (`fromMe = false`); o backend tratava o evento como fala do cliente | Corrigido em `WhatsAppWebhookService.ProcessarMensagemRecebidaAsync`: o payload com `isStatusBroadcast = true` e os tipos `unknown`, `notification`, `protocolmessage`, `protocol_message`, `system`, `e2e_notification`, `stub`, `stubtype` e `revoked` são ignorados antes de registrar a entrada, assim como o texto de aviso de serviço seguro da Meta mesmo com `type = text`. O log registra "Mensagem de sistema do WhatsApp ignorada..." e não há contato, conversa, atendimento nem mensagem. Coberto por [`WhatsAppMensagemSistemaTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppMensagemSistemaTests.cs). Atendimentos abertos por esses avisos **antes** da correção continuam no histórico |


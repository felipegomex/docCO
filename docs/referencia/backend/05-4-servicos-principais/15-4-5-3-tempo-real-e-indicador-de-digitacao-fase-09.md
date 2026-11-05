---
id: 05-4-servicos-principais.15
titulo: "4.5.3 Tempo real e indicador de digitação (Fase 09)"
tipo: referencia
area: backend
sistema: central
tags: [backend, services, di, efcore]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Configuration
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1255
ficha: docs/modulos/README.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

#### 4.5.3 Tempo real e indicador de digitação (Fase 09)

**SignalR é notificação de saída, não fonte da verdade.** O hub `/hubs/whatsapp` (`Hubs/WhatsAppHub.cs`) apenas avisa que algo mudou; o cliente relê o dado pela API autenticada. Por isso nenhum payload carrega texto de mensagem, observação ou nota interna — só IDs, status e metadados (decisão D03).

| Item | Implementação confirmada |
|---|---|
| Autenticação | O hub é `[Authorize(Policy = "perm:aba.suporte.whatsapp:Ler")]`. Como o navegador não envia o header `Authorization` no handshake do WebSocket, o JWT viaja em `access_token` na **query string** — e o `JwtBearerEvents.OnMessageReceived` só aceita esse transporte quando o path começa em `/hubs/whatsapp`; o resto da API continua **header-only** (`Program.cs`) |
| Endpoint | `app.MapHub<WhatsAppHub>("/hubs/whatsapp")`, fora do versionamento `/api/v1` e sem relação com os controllers REST |
| Grupos | `atendimento-{id}` (quem abriu o atendimento), `fila-{numeroId}` (Kanban/Minhas Conversas) e `operador-{operadorId}` (eventos do próprio operador). `EntrarNoOperador` usa o id do claim, nunca um id do cliente |
| Entrada em grupo | `EntrarNoAtendimento` é validado por `WhatsAppHubAuthorization`: policy do módulo + operador ativo + atendimento existente. `EntrarNaFila` e `EntrarNoOperador` não expõem dado — a leitura continua autenticada pela API |
| Métodos do hub | `EntrarNoAtendimento(atendimentoId)`, `SairDoAtendimento(atendimentoId)`, `EntrarNaFila(numeroId)`, `SairDaFila(numeroId)`, `EntrarNoOperador()` e `Digitando(atendimentoId, modo = "mensagem")` |
| Publicação | Sempre **depois do commit**: a entrada de webhook publica `mensagem_recebida` + `atendimento_atualizado` + `fila_alterada` após `SaveChanges`; `message.ack`/`message.sent`/`message.failed` publicam `mensagem_entregue` depois de gravar o status; e **toda mutação do atendimento** (`PublicarAlteracaoAsync`) publica `atendimento_atualizado` + `fila_alterada` — envio de texto/mídia pela Central, assumir, transferir, devolver, resolver, reabrir, aguardar cliente, retomar, marcar visto, nota interna, etiquetas, investigação, observação, prioridade/assunto, finalizar, arquivar, solicitar privado e iniciar conversa. Antes, só o fluxo de entrada publicava, então o envio de um operador só aparecia para ele até o polling de 30 s. A falha de publicação é registrada em log (`Warning`) e **não desfaz** a operação de negócio (D02) |

**Eventos publicados** (todos com payload mínimo):

| Evento | Grupos | Payload |
|---|---|---|
| `atendimento_atualizado` | `atendimento-{id}`, `fila-{numeroId}` | `atendimentoId`, `status`, `responsavelId`, `numeroId` |
| `mensagem_recebida` | `atendimento-{id}`, `fila-{numeroId}` | `atendimentoId`, `mensagemId`, `chatId` |
| `mensagem_entregue` | `atendimento-{id}` | `atendimentoId`, `mensagemId`, `status` |
| `evento_registrado` | `atendimento-{id}` | `atendimentoId`, `eventoId`, `tipo` |
| `fila_alterada` | `fila-{numeroId}` | `numeroId` |
| `digitando` | `atendimento-{id}` | `atendimentoId`, `operadorId` |

**Indicador de digitação.**

- **Interno (sempre):** `WhatsAppTypingHandler` publica `digitando` no grupo do atendimento, com apenas `atendimentoId` e `operadorId`. Outros operadores veem "alguém está digitando"; o próprio operador é ignorado pela tela.
- **Externo (somente `modo = "mensagem"`):** `WhatsAppTypingExternoService` resolve `ChatId` da conversa e `SessaoId` do número e chama `OpenWaClient.EnviarEstadoChatAsync(sessaoId, chatId, "typing")`. A capacidade foi **confirmada no OpenWA 0.25**: `POST /api/sessions/{sessionId}/chats/typing` com `{ "chatId": "...", "state": "typing" }`; o cliente aceita `typing`, `recording` e `paused` e recusa qualquer outro valor antes de sair da Central.
- **Em `modo = "nota"`:** o gateway **nunca** é chamado — a nota interna não pode gerar presença externa. O evento interno continua.
- **Debounce:** o frontend limita a **1 evento a cada 3 s**, e o handler reforça a mesma janela por `atendimentoId:operadorId` (`IntervaloMinimo = 3 s`). Um evento suprimido devolve `false` sem publicar nada.
- **Limites do gateway:** o OpenWA 0.25 aplica **10 req/1 s, 100 req/60 s e 1000 req/1 h por IP**. O debounce de 3 s mantém a presença dentro da cota.
- **Falha de presença:** `OpenWaException` é logada em `Debug` e a presença é simplesmente descartada; o operador não recebe erro e o polling continua reconciliando a tela.

**Sem schema nesta fase.** A Fase 09 não cria tabela, coluna, índice, migration EF ou script SQL: o hub e o typing vivem em memória/`IHubContext` e usam apenas `tbwhatsapp_atendimento`, `tbwhatsapp_conversa` e `tbwhatsapp_numero` já existentes.


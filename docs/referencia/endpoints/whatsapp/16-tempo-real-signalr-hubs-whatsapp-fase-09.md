---
id: whatsapp.16
titulo: "Tempo real (SignalR) — `/hubs/whatsapp` — Fase 09"
tipo: referencia
area: api
sistema: central
tags: [api, whatsapp, endpoints]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 1117
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## Tempo real (SignalR) — `/hubs/whatsapp` — Fase 09

**Não é API REST.** O hub (`backend/Central_BackEnd/Hubs/WhatsAppHub.cs`) existe só para **notificar a saída**: quem recebe um evento relê o dado pela API autenticada. Nenhum payload carrega texto de mensagem, observação ou nota interna — apenas IDs, status e metadados.

| Item | Contrato |
|---|---|
| URL | `/hubs/whatsapp` (fora de `/api/v1`; mapeado por `MapHub<WhatsAppHub>`) |
| Autenticação | `[Authorize(Policy = "perm:aba.suporte.whatsapp:Ler")]`. O JWT vai em `access_token` na **query string** porque o navegador não envia o header no handshake do WebSocket; o `JwtBearerEvents.OnMessageReceived` só aceita esse transporte para paths que começam em `/hubs/whatsapp`. Toda a API REST continua **header-only** |
| Grupos | `atendimento-{id}` (atendimento aberto), `fila-{numeroId}` (Kanban/Minhas Conversas) e `operador-{operadorId}` (eventos do próprio operador) |
| `EntrarNoAtendimento(atendimentoId)` | Valida a leitura por `WhatsAppHubAuthorization`: policy do módulo + operador ativo + atendimento existente; sem permissão → `HubException`. Entra no grupo `atendimento-{id}` |
| `EntrarNaFila(numeroId)` | Entra no grupo `fila-{numeroId}` (Kanban/Minhas Conversas). Não devolve dado — a leitura continua na API |
| `EntrarNoOperador()` | Entra no grupo do **próprio** operador; o id vem do claim, nunca do cliente. Sessão sem operador → `HubException` |
| `Digitando(atendimentoId, modo = "mensagem")` | Modo só aceita `mensagem` ou `nota`. Valida escrita no atendimento e aplica debounce de 3 s por atendimento+operador. Publica `digitando` (interno) e, **somente em `mensagem`**, chama o gateway; em `nota` o gateway **nunca** é chamado |
| Métodos de saída | `SairDoAtendimento(atendimentoId)` e `SairDaFila(numeroId)` removem a conexão do grupo |

**Eventos enviados pelo servidor** (todos com payload mínimo):

| Evento | Grupos | Payload |
|---|---|---|
| `atendimento_atualizado` | atendimento + fila | `atendimentoId`, `status`, `responsavelId`, `numeroId` |
| `mensagem_recebida` | atendimento + fila | `atendimentoId`, `mensagemId`, `chatId` |
| `mensagem_entregue` | atendimento | `atendimentoId`, `mensagemId`, `status` |
| `evento_registrado` | atendimento | `atendimentoId`, `eventoId`, `tipo` |
| `fila_alterada` | fila | `numeroId` |
| `digitando` | atendimento | `atendimentoId`, `operadorId` |

**Cliente (frontend) — resolução da URL e reconexão.** O `WhatsAppRealtimeService.hubUrl()` só aceita o `whatsappHubUrl` do runtime-config quando a origem é a **mesma** da `apiBaseUrl` e não é `localhost`/`127.0.0.1` num host remoto; caso contrário deriva `{origem da apiBaseUrl}/hubs/whatsapp`. Assim um `runtime-config` preservado de outro ambiente (HML em PROD, ou o `localhost` do arquivo versionado) **não** arrasta o hub para o servidor errado. `conectar()` **aguarda o auth inicializar** e tenta de novo quando ainda não há token (antes desistia para sempre e só o polling atualizava). O Kanban assina `mensagem_recebida` além de `atendimento_atualizado`/`fila_alterada`; o atendimento assina também `reacao_atualizada`. Todos os eventos continuam sendo aviso: a tela relê pela API.

**Publicação e falhas.** O evento sai **depois do commit**: a entrada de webhook publica `mensagem_recebida` + `atendimento_atualizado` + `fila_alterada` após `SaveChanges`; `message.ack`, `message.sent` e `message.failed` publicam `mensagem_entregue` depois de gravar o status; e as **mutações do `AtendimentoService`** publicam `atendimento_atualizado` + `fila_alterada` — envio de texto/mídia pela Central, assumir, transferir, devolver, resolver, reabrir, aguardar cliente, retomar, marcar visto, nota interna, etiquetas, investigação, observação, prioridade/assunto, finalizar, arquivar, solicitar privado e iniciar conversa. Antes de 2026-10-28 (commit `e02098e`) só o fluxo de entrada publicava: o envio de um operador só aparecia para ele até o polling. Falha de publicação é logada e **não desfaz** a operação (não vira 500). O polling de 30 s das telas continua como **fallback** quando o hub está desligado ou a conexão cai.

**Ack não regride.** O status da mensagem só **avança** (`pendente` → `enviada` → `entregue` → `lida`): ack fora de ordem (reentrega do gateway) não rebaixa o que já foi confirmado. Quando o `status` textual falta ou é desconhecido, o servidor usa o campo numérico `ack` (0 a 4) e, se ainda assim não entender, **não mexe** no status. `failed` só substitui `pendente`/`enviada`, e uma entrega/lida posterior **substitui** a falha.

**Indicador externo (cliente vê "digitando").** Capacidade confirmada no OpenWA 0.25: `POST /api/sessions/{sessionId}/chats/typing` com `{ "chatId": "...", "state": "typing" }`; os estados aceitos são `typing`, `recording` e `paused`. O frontend limita a 1 evento a cada 3 s e o handler reforça o mesmo intervalo no servidor. O gateway documenta **10 req/1 s, 100 req/60 s e 1000 req/1 h por IP**.


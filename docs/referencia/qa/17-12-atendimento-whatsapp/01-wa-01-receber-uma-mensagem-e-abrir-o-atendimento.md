---
id: 17-12-atendimento-whatsapp.01
titulo: "WA-01 — Receber uma mensagem e abrir o atendimento (+9)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Controllers/WhatsApp
  - backend/Central_BackEnd/Services/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - frontend/src/app/features/whatsapp/pages/kanban/whatsapp-kanban.component.spec.ts
tokens_estimados: 3462
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-01 — Receber uma mensagem e abrir o atendimento

- **Objetivo:** a mensagem do cliente entrar no histórico e gerar o card na fila de atendimento.
- **Pré-condições:** gateway OpenWA conectado (sessão `ready`), número cadastrado na Central, módulo habilitado no backend e webhook registrado no gateway.
- **Passos:** 1. O cliente envia uma mensagem no WhatsApp. 2. Abrir `/whatsapp` (ou o item **Atendimento WhatsApp** na seção *SUPORTE*).
- **Resultado esperado:** o gateway entrega o evento por `POST /api/v1/whatsapp/webhook/openwa` (assinatura HMAC do corpo cru). O receptor responde `2xx` rápido e enfileira; o worker processa. A mensagem aparece no histórico, o contato é criado (se for a primeira vez), a conversa é criada e **um** atendimento nasce em **Não atendido**, com evento `abertura`. A mensagem enviada pelo **celular da equipe** (`fromMe` sem operador) entra no histórico **sem** assumir o atendimento.
- **Estado atual/bloqueio:** a entrega do gateway é *at-least-once*: uma reentrega **não** pode virar mensagem duplicada (índice único `(WAT_ChatId, WAT_ExternalId)`). A fila de webhook é **durável** (SQL Server), porque o pool do IIS recicla. Com o hub conectado, o card aparece assim que a entrada é publicada (`mensagem_recebida`/`atendimento_atualizado`/`fila_alterada`); sem o hub, o polling de **30 s** continua sendo o fallback.
- **Rotas/endpoints/tabelas:** `/whatsapp`; `POST /api/v1/whatsapp/webhook/openwa` (sem JWT, HMAC), `GET /api/v1/whatsapp/atendimentos`; `tbwhatsapp_webhook_fila`, `tbwhatsapp_contato`, `tbwhatsapp_conversa`, `tbwhatsapp_atendimento`, `tbwhatsapp_mensagem`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; não há teste automatizado do fluxo.
### WA-01b — Mensagem de sistema da Meta não abre atendimento

- **Objetivo:** garantir que aviso de sistema/protocolo entregue pelo gateway no `message.received` não vire mensagem do cliente, card na fila nem não lida.
- **Pré-condições:** gateway OpenWA conectado (sessão `ready`), número cadastrado na Central, módulo habilitado no backend e webhook registrado no gateway.
- **Passos:** 1. Conectar o número e aguardar os avisos automáticos do WhatsApp (ex.: "Esta empresa agora usa um serviço seguro da Meta..."). 2. Conferir `/whatsapp` e o detalhe do atendimento. 3. Em seguida, enviar uma mensagem real do cliente.
- **Resultado esperado:** o aviso é descartado antes de registrar a entrada. Payload com `isStatusBroadcast = true` e tipos `unknown`, `notification`, `protocolmessage`, `protocol_message`, `system`, `e2e_notification`, `stub`, `stubtype` e `revoked` são ignorados; o texto de aviso de serviço seguro da Meta também, mesmo com `type = text`. O log registra a mensagem ignorada e **não** há contato, conversa, atendimento nem mensagem — a fila e o contador de não lidas continuam como estavam. A mensagem real do passo 3 continua abrindo **um** atendimento em **Não atendido**, com evento `abertura`.
- **Estado atual/bloqueio:** o filtro é do backend, no processamento do webhook; o gateway continua entregando o evento. Atendimentos abertos por esses avisos **antes** da correção permanecem no histórico.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/webhook/openwa`; `tbwhatsapp_webhook_fila`, `tbwhatsapp_contato`, `tbwhatsapp_conversa`, `tbwhatsapp_atendimento`, `tbwhatsapp_mensagem`.
- **Status/automação:** coberto por teste unitário — `WhatsAppMensagemSistemaTests` (tipo `unknown`, aviso da Meta como texto, status broadcast e mensagem normal).
### WA-02 — Assumir o atendimento pela primeira resposta

- **Objetivo:** o atendimento passar a ter responsável, e o responsável ser o operador que respondeu.
- **Pré-condições:** atendimento em **Não atendido**, sem responsável.
- **Passos:** 1. Abrir `/whatsapp/atendimento/:id`. 2. Escrever a resposta e enviar. 3. Conferir a coluna e o card.
- **Resultado esperado:** a **primeira resposta enviada pela Central atribui o atendimento** ao operador autenticado: o card vai para **Em atendimento**, com o responsável e o evento `atribuicao`. A atribuição é feita com a linha do atendimento travada e o estado revalidado dentro da transação — **duas respostas concorrentes não geram dois responsáveis**: a segunda leva `409` e a tela recarrega. O botão **Assumir** do card faz o mesmo por ação explícita.
- **Estado atual/bloqueio:** se o card já tiver responsável de outro operador, **Assumir** responde `409` ("use transferir") e a ação **Transferir** é que serve. O `OperadorId` do corpo **nunca** define o autor: ele vem do claim do token.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/mensagens`, `POST /api/v1/whatsapp/atendimentos/{id}/assumir`; `tbwhatsapp_atendimento`, `tbwhatsapp_mensagem`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-03 — Transferir o atendimento com histórico

- **Objetivo:** passar o atendimento para outro suporte, mantendo a trilha de quem tinha e quem passou a ter.
- **Pré-condições:** atendimento **Em atendimento**, com responsável.
- **Passos:** 1. No card, usar **Transferir**. 2. Escolher o operador de destino. 3. Conferir o histórico.
- **Resultado esperado:** o `POST /atendimentos/{id}/transferir` recebe `{ "operadorId": "..." }` — o corpo é o **destino**, nunca o autor. O responsável muda, `WAT_ResponsavelDesde` é renovado e o evento `transferencia` grava de/para. O destino precisa existir em `tboperador`; transferir para o mesmo operador, ou para o próprio card, é recusado.
- **Estado atual/bloqueio:** o destino é **obrigatório** — é o que impede um "arrastar para o lado" genérico; por isso o Kanban do módulo **não tem drag-and-drop**. Transferir **não** tira o card da coluna "Aguardando cliente".
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/transferir`; `tbwhatsapp_atendimento`, `tbwhatsapp_evento`, `tboperador`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-04 — Devolver o atendimento à fila

- **Objetivo:** soltar o atendimento para que outro suporte o assuma.
- **Pré-condições:** atendimento com responsável e **não** resolvido.
- **Passos:** 1. No card, usar **Devolver para a fila**. 2. Conferir a coluna e o histórico.
- **Resultado esperado:** o responsável e o `WAT_ResponsavelDesde` são limpos, o status volta a **Não atendido** e o evento `devolucao` guarda quem devolveu. O card aparece na primeira coluna da fila.
- **Estado atual/bloqueio:** devolver um atendimento já resolvido é recusado (`409`); devolver um card sem responsável também (ele já está na fila).
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/devolver`; `tbwhatsapp_atendimento`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-05 — Resolver com mensagem de encerramento

- **Objetivo:** encerrar o atendimento e, quando configurado, enviar a mensagem de encerramento ao cliente.
- **Pré-condições:** atendimento **não** resolvido; o número pode ter `WAT_MensagemEncerramento` configurado.
- **Passos:** 1. No card, usar **Resolver**. 2. Marcar o checkbox **"Enviar mensagem de encerramento ao cliente"** (desmarcado por padrão). 3. Informar (ou deixar em branco) o texto de encerramento. 4. Confirmar.
- **Resultado esperado:** o status vai para **Resolvido**, `WAT_ResolvidoEm` é gravado, `MensagensNaoLidas` é zerado na mesma transação e o evento `resolucao` é registrado. O texto enviado é, **nesta ordem**: o configurado no número, o texto do corpo, ou nada — e leva o prefixo `*NOME:* ` do operador (maiúsculas, sem duplicar). A mensagem aparece no histórico como enviada e o chat é marcado como lido no WhatsApp.
- **Estado atual/bloqueio:** a resolução é confirmada **antes** do envio: **falha do gateway não desfaz a resolução** — o card fica resolvido, a mensagem fica como **falhou**, o evento `encerramento_falhou` é gravado e a tela mostra a falha (o `502` do gateway é traduzido). Resolver não apaga o histórico; o atendimento continua no filtro **Resolvido** do Kanban.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/resolver` (`enviarEncerramento: true`); `tbwhatsapp_atendimento`, `tbwhatsapp_mensagem`, `tbwhatsapp_evento`, `tbwhatsapp_numero`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-05b — Resolver sem enviar mensagem

- **Objetivo:** encerrar o atendimento sem falar com o cliente, limpando o badge de não lidas e a leitura pendente no WhatsApp.
- **Pré-condições:** atendimento **não** resolvido e com mensagens não lidas; o número pode ter `WAT_MensagemEncerramento` configurado.
- **Passos:** 1. Abrir o card no Kanban (`/whatsapp`) ou no atendimento (`/whatsapp/atendimento/:id`). 2. Usar **Resolver** e deixar o checkbox **"Enviar mensagem de encerramento ao cliente"** desmarcado (padrão). 3. Conferir que o textarea de encerramento está desabilitado. 4. Confirmar.
- **Resultado esperado:** `POST /api/v1/whatsapp/atendimentos/{id}/resolver` recebe `enviarEncerramento: false` e `mensagem: null`; o card vai para **Resolvido** com `WAT_ResolvidoEm`, `MensagensNaoLidas = 0` na mesma transação e os eventos `resolucao` e `mensagem_sistema` ("Atendimento resolvido sem mensagem de encerramento."). **Nenhuma** mensagem é enviada — nem a configurada no número. Depois do commit, o backend chama `POST /api/sessions/{sessaoId}/chats/read` com `{ chatId }` e o badge some também no WhatsApp; se o gateway falhar, a resolução continua valendo e a falha fica no log.
- **Estado atual/bloqueio:** a marcação de leitura no gateway é **best-effort** e acontece depois do commit — não desfaz a resolução. Não há schema nem rota novos.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/resolver`; gateway `POST /api/sessions/{sessaoId}/chats/read`; `tbwhatsapp_atendimento`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-05c — Finalizar é terminal e não envia mensagem

- **Objetivo:** confirmar que **Finalizar** encerra o card sem falar com o cliente e sem formulário, e que o encerramento pertence ao **Resolver**.
- **Pré-condições:** atendimento em **Resolvido**, com responsável (o próprio operador) ou perfil **Administrador**; no Kanban, o card na coluna **Resolvido**.
- **Passos:** 1. No card, usar **Finalizar** (ação direta, sem modal/textarea). 2. Conferir a coluna **Finalizado** e a timeline. 3. Conferir no WhatsApp do cliente que **nenhuma** mensagem chegou. 4. Pela API, enviar `POST /atendimentos/{id}/finalizar` sem body e, em outro card, com body vazio.
- **Resultado esperado:** `POST /api/v1/whatsapp/atendimentos/{id}/finalizar` aceita corpo vazio (`FinalizarAtendimentoRequest` não tem campos), responde `200 AtendimentoDetalhe` e grava **`Finalizado` (4)** + evento **`finalizacao`**. **Nenhuma** mensagem é persistida nem enviada ao gateway; o atendimento **não** é arquivado nessa ação. `400` se o card não estiver **Resolvido** e `403` se não for o responsável nem admin.
- **Estado atual/bloqueio:** depois de finalizado, nova mensagem do cliente abre um **novo** atendimento (o card finalizado não é reaberto); a automação pode levar o card adiante para **Arquivado** (WA-86).
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/finalizar`; `tbwhatsapp_atendimento`, `tbwhatsapp_evento`.
- **Status/automação:** coberto por `WhatsAppAtendimentoTests.cs` (Finalizar não envia mensagem).
### WA-06 — Reabrir o atendimento

- **Objetivo:** reativar um atendimento encerrado, preservando o histórico dos ciclos anteriores.
- **Pré-condições:** atendimento **Resolvido**.
- **Passos:** 1. Usar **Reabrir** no card — ou, sem ação, **fazer o cliente enviar uma nova mensagem**. 2. Conferir coluna, contador e histórico.
- **Resultado esperado:** manual ou automático, a reabertura devolve o **mesmo atendimento** (não cria outro): status **Não atendido**, responsável nulo, `WAT_ContadorReaberturas` incrementado, `WAT_AbertoEm` = agora (novo ciclo), `WAT_PrimeiraRespostaEm`/`WAT_ResolvidoEm` zerados e `WAT_ReabertoEm` gravado. O evento `reabertura` fica no histórico, junto com os ciclos anteriores.
- **Estado atual/bloqueio:** o ciclo **reinicia** (a espera do cliente zera com ele) e o atendimento não volta para a coluna de onde saiu — ele volta para a fila. Reabrir um card que não está resolvido é recusado (`409`).
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/reabrir` e o webhook `message.received`; `tbwhatsapp_atendimento`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-07 — Aguardar o cliente e retomar

- **Objetivo:** marcar que a bola está com o cliente, sem transferir a responsabilidade.
- **Pré-condições:** atendimento **Em atendimento** (status 1).
- **Passos:** 1. No card, usar **Aguardar cliente**. 2. Confirmar que o card mudou de coluna. 3. Clicar em **Retomar** — ou responder ao cliente, ou esperar a resposta dele.
- **Resultado esperado:** o status vai a **Aguardando cliente** (3) **mantendo o responsável** — a espera é do cliente, não do suporte. **Retomar** devolve para **Em atendimento** (se houver responsável) ou para a fila (se não houver). A resposta enviada pelo suporte **também** traz o card de volta para **Em atendimento** (com evento `retomada`), e a **resposta do cliente** faz o mesmo, sem reiniciar o ciclo.
- **Estado atual/bloqueio:** só se entra em espera a partir do status 1 — da fila (0) não há quem espere por ninguém (`409`). Transferir **não** tira o card da espera. O tempo em espera é **descontado** do tempo de primeira resposta e do tempo de resolução nas métricas: não é atraso do suporte.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/aguardar-cliente`, `POST /api/v1/whatsapp/atendimentos/{id}/retomar`; `tbwhatsapp_atendimento`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-08 — Registrar nota interna

- **Objetivo:** deixar um recado no atendimento que **só a equipe** vê.
- **Pré-condições:** atendimento aberto (qualquer status não resolvido para responder; a nota grava no histórico do card).
- **Passos:** 1. No painel, alternar o compositor para **nota interna**. 2. Escrever o texto e enviar.
- **Resultado esperado:** a nota entra no histórico com direção **2** (nota interna) e o evento `nota`. Ela **nunca** é enviada ao WhatsApp — é o único caminho de escrita que não passa pelo gateway.
- **Estado atual/bloqueio:** o texto é limitado a **4000 caracteres** e não pode ser vazio (`400`).
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/notas`; `tbwhatsapp_mensagem`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.

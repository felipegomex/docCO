---
id: 17-12-atendimento-whatsapp.16
titulo: "WA-105 — Reações da Fase 8: flag, evidência e modo limitado (+3)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias, reacao]
status: ativo
revisado_em: "2026-10-30"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Services/WhatsApp
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - backend/Central_BackEnd.Tests/WhatsAppWebhookTests.cs
  - frontend/src/app/features/whatsapp/components/whatsapp-conversation/whatsapp-message.component.spec.ts
tokens_estimados: 1500
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-105 — Menu de reações oculto com a flag desligada ou sem `ExternalId` (Fase 8)

- **Objetivo:** a Fase 8 estar implementada **não** pode expor reação onde o gateway não é chamável.
- **Pré-condições:** flag **ligada por configuração desde 2026-10-30** (`appsettings.json` com `ReacoesHabilitadas=true`; o modelo de ambiente novo, `appsettings.sample.json`, nasce `false`); atendimento com mensagem recebida (`externalId` preenchido) e outra sem `ExternalId`. Para exercitar o ramo "desligada", defina `WhatsApp__ReacoesHabilitadas=false` explicitamente.
- **Passos:** 1. Abrir o atendimento com a configuração atual (flag ligada). 2. Passar o mouse/clicar no balão de uma mensagem com `externalId`. 3. Desligar a flag (`WhatsApp__ReacoesHabilitadas=false`) e reabrir. 4. Repetir sobre a mensagem sem `externalId`.
- **Resultado esperado:** com a flag desligada o `AtendimentoDetalhe` traz `permissoes.reacoesHabilitadas=false` e o menu **não existe** em nenhum balão. Com a flag ligada, o menu aparece só nas mensagens com `externalId`; a mensagem sem `ExternalId` continua sem menu.
- **Estado atual/bloqueio:** o menu é decidido por `reacoesHabilitadas && !!externalId && !ehNota()`. Nenhuma chamada ao gateway acontece ao abrir o menu. O cenário é **executável** desde a aplicação do SQL nos dois bancos de homologação (2026-10-30) e a ligação da flag; a ressalva é o **payload** do `message.reaction` (C3).
- **Rotas/endpoints/tabelas:** `GET /api/v1/whatsapp/atendimentos/{id}`; nenhuma tabela.
- **Status/automação:** **cenário derivado do código**; coberto por `whatsapp-message.component.spec.ts` (menu escondido com flag/sem `externalId`).

### WA-106 — Nota interna nunca oferece reação (Fase 8)

- **Objetivo:** conteúdo interno não vira reação no WhatsApp.
- **Pré-condições:** flag ligada; atendimento com uma **nota interna** (`direcao = 2`) e `externalId` presente.
- **Passos:** 1. Abrir o atendimento. 2. Passar o mouse sobre o balão da nota interna. 3. Comparar com um balão de mensagem real.
- **Resultado esperado:** o balão da nota interna **não** exibe o botão/menu de reação, mesmo com a flag ligada e `externalId` preenchido; o balão da mensagem real exibe. O chip agregado também não aparece na nota.
- **Estado atual/bloqueio:** a regra `!ehNota()` está no componente; nota interna é criada sem passagem pelo gateway. O endpoint exige `ExternalId`, mas a garantia de "nota nunca reage" é **da tela**.
- **Rotas/endpoints/tabelas:** nenhum endpoint chamado; `tbwhatsapp_mensagem`.
- **Status/automação:** **cenário derivado do código**; coberto pelo spec ("nota interna nunca tem menu de reação").

### WA-107 — Chip agregado com contagem e autores (Fase 8)

- **Objetivo:** mostrar o agregado vindo do servidor, sem calcular no cliente.
- **Pré-condições:** flag ligada; mensagem com `reacoes: [{ emoji: "👍", quantidade: 3, autores: "Ana, Bruno" }]`.
- **Passos:** 1. Abrir o atendimento. 2. Conferir o chip abaixo do balão. 3. Passar o mouse para ver o tooltip. 4. Receber uma reação de outro participante (evento `reacao_atualizada`).
- **Resultado esperado:** o chip mostra o emoji e a **quantidade**; o tooltip lista os **autores**. O componente apenas renderiza o agregado do `AtendimentoDetalhe` — não soma nem deduplica no cliente. O evento `reacao_atualizada` traz **só** `AtendimentoId` e `MensagemId` (D03) e a tela **relê** o atendimento pela API em vez de aplicar estado local.
- **Estado atual/bloqueio:** o toggle local do próprio operador só acontece **após o 2xx**; não há estado otimista.
- **Rotas/endpoints/tabelas:** `GET /api/v1/whatsapp/atendimentos/{id}`; hub `/hubs/whatsapp` (evento `reacao_atualizada`); `tbwhatsapp_mensagem_reacao`.
- **Status/automação:** **cenário derivado do código**; coberto pelo spec do balão (chip com contagem/autores). A chegada real do evento tem só **evidência informal** (relato do usuário de 2026-10-30, sem payload) — medição formal **adiada** (bloco A).

### WA-108 — Falha do gateway (502) e mensagem sem `ExternalId` (400) sem mudar estado (Fase 8)

- **Objetivo:** reação não fica "meio aplicada" quando o gateway falha ou o alvo não é reagível.
- **Pré-condições:** flag ligada; mensagem com `externalId`; simular gateway indisponível; segunda rodada com mensagem sem `ExternalId`.
- **Passos:** 1. Reagir com o gateway fora do ar. 2. Conferir o balão e o banco. 3. Reagir a uma mensagem sem `ExternalId`. 4. Reagir com a flag desligada explicitamente (`WhatsApp__ReacoesHabilitadas=false`).
- **Resultado esperado:** gateway falho → **`502`** e **nenhuma** reação gravada (a Central só grava depois do 2xx; sem toggle otimista na tela). Mensagem sem `ExternalId` → **`400`** ("Esta mensagem não pode receber reação."). Flag desligada → **`400`**. `404` quando o atendimento/mensagem está fora do escopo; `409` para atendimento resolvido ou de outro operador.
- **Estado atual/bloqueio:** a ordem é deliberada — guardas → gateway → gravação local (`AtendimentoService.ReagirAsync`). Emoji vazio = remoção, marcada em `RemovidaEm` (remoção **lógica**).
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/{id}/mensagens/{mensagemId}/reacao`; `tbwhatsapp_mensagem_reacao`.
- **Status/automação:** **cenário derivado do código**; coberto por `WhatsAppAtendimentoTests.cs` (502, flag desligada, sem `ExternalId`, remoção).

> **Evidência e pendência da Fase 8 (2026-10-30):** o SQL
> [`WhatsApp_Fase9_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Fase9_Idempotente.sql)
> foi **aplicado** em `dbCENTRAL_OPERACAO` e `dbBUSINESS_HML` (`192.168.2.154`)
> e a flag está **ligada por configuração** — os cenários ficam executáveis. O
> preflight de deploy não bloqueia mais nesses bancos. **A evidência do gateway
> é apenas informal:** relato do usuário de 2026-10-30 (remoção com emoji vazio
> funcionou e o evento `message.reaction` chegou), **sem status HTTP nem payload
> registrados**; a medição formal (bloco A) foi **adiada** e `messages/react`,
> `GET .../reactions` e `message.reaction` seguem **NÃO TESTADOS** (§ 3.6.3 de
> `material-de-projeto/atendimento-whatsapp/PLANO-EVOLUCAO-WHATSAPP-2026-10.md`).
> **Modo limitado aceito (C3):** payload de `message.reaction` fora do formato
> esperado (`messageId`, `chatId`, `reaction`, `senderId`, `contact.name` em
> `data`) faz a reação sair sem agregar/persistir — mantenha a ressalva ao
> registrar o resultado do teste.

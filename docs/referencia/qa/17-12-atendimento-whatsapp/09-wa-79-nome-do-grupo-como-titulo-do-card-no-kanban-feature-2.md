---
id: 17-12-atendimento-whatsapp.09
titulo: "WA-79 — Nome do grupo como título do card no Kanban (Feature 2) (+6)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Controllers/WhatsApp
  - backend/Central_BackEnd/Services/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - frontend/src/app/features/whatsapp/pages/kanban/whatsapp-kanban.component.spec.ts
tokens_estimados: 3231
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-79 — Nome do grupo como título do card no Kanban (Feature 2)

- **Objetivo:** mostrar o nome real do grupo no card, separando o grupo do participante que abriu o atendimento.
- **Pré-condições:** número conectado a um grupo; atendimento de grupo no Kanban; gateway OpenWA 0.25 acessível para a sincronização do título.
- **Passos:** 1. Receber mensagem de um participante no grupo. 2. Abrir `/whatsapp`. 3. Localizar o card. 4. Conferir o título e a linha do solicitante. 5. Abrir Minhas Conversas (se o atendimento estiver sob responsabilidade do operador). 6. Comparar com um atendimento privado.
- **Resultado esperado:** o título principal do card é o **nome do grupo** com ícone de pessoas (fallback “Grupo sem nome”) e abaixo aparece **“Solicitante: X”**, com o participante que abriu o ciclo. O contato do chat não ocupa o lugar do grupo. Não há bloco duplicado “Grupo: … · Solicitante: …”. Atendimento privado continua exibindo contato/telefone. Em Minhas Conversas, `nomeDo` usa `grupoNome` quando há `grupoId`.
- **Estado atual/bloqueio:** o nome é enriquecimento: se o gateway falhar, o card mantém o nome já gravado (ou “Grupo sem nome”) e a mensagem continua sendo processada. A sincronização tem cache em memória de **5 min por número**. Sem schema novo.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/minhas-conversas`; `GET /api/v1/whatsapp/atendimentos` (`grupoId`/`grupoNome`/`solicitanteNome`); rota interna do gateway `GET /api/sessions/{sessaoId}/groups` (**não** é endpoint público da Central); `tbwhatsapp_grupo`.
- **Status/automação:** **cenário derivado do código**; coberto por `WhatsAppGrupoSyncTests` e pelo caso de lista em `WhatsAppGrupoAtendimentoTests` (backend) e por 2 specs do card (frontend).
### WA-80 — Integrantes do grupo com contagem e atualização (Feature 5)

- **Objetivo:** ver **todos** os participantes do grupo no modal, sem depender de quem já falou, e poder atualizar a lista a partir do gateway.
- **Pré-condições:** número conectado a um grupo; sessão do gateway em `ready`; atendimento de grupo aberto; o grupo com mais de um participante.
- **Passos:** 1. Abrir `/whatsapp/atendimento/{id}` de um atendimento de grupo. 2. Abrir o modal **Integrantes** (painel do cliente/conversa). 3. Conferir a contagem no título e a lista (nome, telefone/`chatId`, badge ativo). 4. Clicar no botão **atualizar** (`bi-arrow-clockwise`). 5. Remover/adicionar um participante no WhatsApp e atualizar de novo. 6. Simular gateway fora do ar e reabrir o modal.
- **Resultado esperado:** o modal chama `GET /api/v1/whatsapp/grupos/{id}/participantes`, que **sincroniza antes de ler** com `GET /api/sessions/{sessaoId}/groups/{chatId}` (única rota do OpenWA 0.25 que devolve `participants`). O upsert é por `WAT_ChatId`: nome/telefone são atualizados, quem voltou é **reativado** e quem saiu é **desativado, nunca apagado** (preserva o histórico de solicitante). Lista vazia do gateway **não desativa** ninguém; falha do gateway é logada e a resposta é o cadastro local. O botão fica desabilitado durante a carga e a contagem reflete a lista atual quando há integrantes.
- **Estado atual/bloqueio:** o webhook cria apenas o participante que falou; sem a sincronização o modal vinha vazio ou só com o solicitante. A sincronização depende do `SessaoId` do número e é ignorada se ele estiver vazio. Sem schema novo.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `GET /api/v1/whatsapp/grupos/{id}/participantes`; rota interna do gateway `GET /api/sessions/{sessaoId}/groups/{chatId}`; `tbwhatsapp_grupo`, `tbwhatsapp_participante`.
- **Status/automação:** **cenário derivado do código**; coberto por `WhatsAppGrupoSyncTests` (4 casos de participantes: sincronização, desativação de quem saiu, falha do gateway e lista vazia).
### WA-81 — SLA visível na fila e no atendimento (Feature 4)

- **Objetivo:** o SLA calculado pelo backend aparecer no card e no painel, em vez de ficar ausente por falha silenciosa da consulta.
- **Pré-condições:** backend rodando **contra SQL Server** (`Database:UseSqlServer = true`) — o InMemory não reproduz a tradução; banco em `compatibility_level 100`; metas de SLA configuradas.
- **Passos:** 1. Receber uma mensagem e abrir `/whatsapp`. 2. Conferir o SLA no card. 3. Abrir o atendimento e conferir o indicador no painel/aba Resumo. 4. Filtrar por situação de SLA (`no_prazo`/`critico`/`atrasado`/`estourado`). 5. Conferir o log do backend em busca de `SqlException`/`OPENJSON`.
- **Resultado esperado:** `Program.cs` registra o SQL Server com `UseCompatibilityLevel(100)` (`NivelCompatibilidadeSqlServer`) nos **dois** ramos (Development com SQL e ambiente publicado), então o EF 8 traduz `Contains` de lista como `IN (...)` e não emite `OPENJSON(... '$')`. A consulta de `CalcularSlaAsync` deixou de falhar e o SLA aparece no card e no detalhe, com as duas dimensões (primeira resposta e resolução). O `try/catch` do serviço continua existindo: por isso, SLA ausente **sem erro na tela** deve ser investigado como falha de query, não como dado faltando.
- **Estado atual/bloqueio:** o nível 100 é o real de `dbCENTRAL_OPERACAO`, `dbBUSINESS_HML` e `dbBUSINESS`; não há migration nem mudança de schema nesta correção. Se o SLA voltar a sumir, confirme o provider no log/`Program.cs` antes de suspeitar do cálculo.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/atendimento/:id`; `GET /api/v1/whatsapp/atendimentos` (filtros de SLA) e o detalhe; `tbwhatsapp_sla_*`, `tbwhatsapp_atendimento`.
- **Status/automação:** **cenário derivado do código**; sem teste automatizado — exige SQL Server real e a declaração do nível no provider.
### WA-82 — Balões e avatar no layout do WhatsApp Web (Feature 3)

- **Objetivo:** a conversa ler como WhatsApp Web — hora e ticks no lugar certo, autor identificado no grupo e identidade no cabeçalho.
- **Pré-condições:** atendimento com mensagens recebidas e enviadas; um atendimento de grupo com mais de um participante; temas claro e escuro.
- **Passos:** 1. Abrir `/whatsapp/atendimento/{id}`. 2. Conferir a hora no **canto inferior direito** de cada balão. 3. Enviar uma mensagem e acompanhar os ticks (`bi-clock` → `bi-check` → `bi-check-all`, azul quando lida). 4. Abrir um atendimento de grupo e conferir o nome do autor colorido nos balões recebidos. 5. Conferir o avatar do cabeçalho (iniciais para conversa, `bi-people` para grupo). 6. Alternar para o tema escuro e conferir o fundo `#0b141a` (claro: `#efeae2`).
- **Resultado esperado:** balões com cauda, hora no canto inferior direito e ticks **inline** na mensagem enviada (lida em azul; falha com `bi-exclamation-circle`). Em grupo, o balão recebido mostra `autorNome` (pushName gravado em `WAT_AutorNome`); sem autor, cai no nome do cliente. O avatar do cabeçalho usa a foto via proxy quando disponível e cai em `iniciaisDoNome`/`corDoNome` (cor estável por hash); grupo usa `bi-people`. Nota interna mantém visual próprio e não recebe ticks.
- **Estado atual/bloqueio:** em mensagens anteriores à coluna `WAT_AutorNome`, o autor pode vir nulo e o balão usa o nome do cliente/solicitante. O script de preenchimento é aproximado de propósito e não sobrescreve pushName já gravado.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `GET /api/v1/whatsapp/atendimentos/{id}` e histórico do cliente; `tbwhatsapp_mensagem` (`WAT_AutorNome`); `WhatsApp_AutorMensagem_Idempotente.sql`.
- **Status/automação:** **cenário derivado do código**; sem spec dedicada — `npx ng build` verde nesta revisão.
### WA-83 — Recorte de status em "Minhas Conversas" (Fase 5)

- **Objetivo:** abrir a lista já no fluxo operacional e alternar entre **Abertas**, **Todas** e um status específico, sem que conversas resolvidas empurrem as abertas para fora do teto de 100 itens.
- **Pré-condições:** operador autenticado; atendimentos com `responsavelId` do operador logado em mais de um status.
- **Passos:** 1. Abrir `/whatsapp/minhas-conversas`. 2. Conferir o filtro **Status** (padrão **Abertas**). 3. Trocar para **Todas** e depois para um status específico (0–6). 4. Selecionar **Arquivado** e conferir que ele aparece. 5. Trocar o recorte com a lista vazia e conferir o texto do estado vazio.
- **Resultado esperado:** o filtro é **server-side** e cada troca recarrega a lista. **Abertas** envia `somenteAbertos=true`; o backend aplica `status` 0, 1 e 2 em `OR`. **Todas** não manda recorte de status. Um status específico envia `status=<n>`. **Arquivado** envia `status=5` **e** `incluirArquivados=true` (sem isso, arquivados são excluídos por padrão). A lista mantém `somenteMeus=true` e `tamanho=100`; busca local e ordenação client-side valem sobre o recorte carregado. O estado vazio muda conforme o recorte: em **Abertas** orienta assumir um atendimento na fila; nos demais, informa que não há conversa para o status.
- **Estado atual/bloqueio:** o recorte é server-side, mas a ordenação e a busca textual continuam locais sobre até 100 itens.
- **Rotas/endpoints/tabelas:** `/whatsapp/minhas-conversas`; `GET /api/v1/whatsapp/atendimentos?somenteMeus=true&tamanho=100&somenteAbertos=true`; `tbwhatsapp_atendimento`.
- **Status/automação:** **cenário derivado do código**; sem teste automatizado específico.
### WA-84 — Protocolo visível e busca exata no Kanban (Fases 0/1)

- **Objetivo:** todo atendimento ter um identificador legível e estável, exibido nas telas e pesquisável no servidor.
- **Pré-condições:** `WhatsApp_Fase8_Idempotente.sql` aplicado no banco do ambiente; atendimento existente; operador autenticado.
- **Passos:** 1. Abrir `/whatsapp`. 2. Conferir o protocolo no card. 3. Digitar o protocolo completo no filtro de protocolo e aplicar. 4. Repetir com minúsculas/espaços. 5. Abrir `/whatsapp/minhas-conversas`, o atendimento e o cabeçalho e conferir o mesmo protocolo. 6. Testar um id antigo sem protocolo.
- **Resultado esperado:** o protocolo segue `WA-AAAAMMDD-000123`, derivado de `WAT_Id` + `WAT_DataInclusao` e **imutável**; o filtro é **igualdade exata** (`trim` + maiúsculas) na coluna computada indexada, e a busca livre também o encontra (junto com id, empresa e telefone). Card, lista, cabeçalho e dados do atendimento mostram o mesmo valor; sem o campo na resposta, `protocoloDe` cai em `#<id>` — nunca vazio.
- **Estado atual/bloqueio:** o protocolo **não** é editável nem sequencial; é computado pelo banco. Sem o script, a leitura falha com `Invalid column name 'WAT_Protocolo'`.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/minhas-conversas`, `/whatsapp/atendimento/:id`; `GET /api/v1/whatsapp/atendimentos?protocolo=...`; `tbwhatsapp_atendimento.WAT_Protocolo` + `UX_tbwhatsapp_atendimento_Protocolo`.
- **Status/automação:** **cenário derivado do código**; backend coberto por `WhatsAppProtocoloTests` (3 casos: projeção, busca por protocolo/id e filtro exato) e a busca local por `whatsapp.utils.spec.ts` (1 caso, **não executado**).
### WA-85 — Telefone do contato (Fases 0/1)

- **Objetivo:** o card/cabeçalho mostrar um telefone legível (inclusive para `@lid`) e a busca local encontrar por telefone, sem confundir display com identidade.
- **Pré-condições:** `WhatsApp_Fase8_Idempotente.sql` aplicado; contato com `WAT_TelefoneNormalizado`; contato `@lid` com `WAT_TelefoneResolvido` preenchido; atendimento aberto.
- **Passos:** 1. Abrir `/whatsapp` e `/whatsapp/atendimento/:id`. 2. Conferir o telefone no resumo/cabeçalho. 3. Abrir um atendimento `@lid`. 4. Em Minhas Conversas, buscar pelos dígitos do telefone. 5. Abrir a ficha e a gestão de contatos e conferir `telefoneExibicao`.
- **Resultado esperado:** `contatoTelefone` vem formatado pelo backend (`TelefoneResolvido ?? TelefoneNormalizado ?? Numero`) no detalhe; na lista, o card usa a chave normalizada. No `@lid`, o resolvido é **apenas display**. A busca local de Minhas Conversas cobre telefone e protocolo. `telefoneExibicao` vem preenchido nas projeções; a URL do gateway nunca vai ao navegador. A foto tem roteiro próprio em WA-87.
- **Estado atual/bloqueio:** a chave de identidade continua sendo `chatId`/telefone normalizado, não o telefone resolvido. Na entrada, `WAT_Numero` é gravado sem JID; `SolicitarPrivadoAsync` monta o `ChatId` pelo normalizador.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/minhas-conversas`, `/whatsapp/atendimento/:id`, `/contatos` e `/admin/contatos`; `tbwhatsapp_contato` (`WAT_TelefoneResolvido`), `tbwhatsapp_parametro`.
- **Status/automação:** **cenário derivado do código**; telefone formatado coberto por `WhatsAppProtocoloTests` (3 casos) e a busca por telefone por `whatsapp.utils.spec.ts` (**não executado**).

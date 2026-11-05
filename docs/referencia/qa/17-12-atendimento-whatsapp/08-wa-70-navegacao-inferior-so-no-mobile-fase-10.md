---
id: 17-12-atendimento-whatsapp.08
titulo: "WA-70 — Navegação inferior só no mobile (Fase 10) (+8)"
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
tokens_estimados: 3427
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-70 — Navegação inferior só no mobile (Fase 10)

- **Objetivo:** no mobile, a prioridade **conversa > cliente > operação** ser explícita e a navegação inferior não aparecer onde não deve.
- **Pré-condições:** atendimento carregado; modo avulso (`/whatsapp/atendimento/{id}`); uma segunda sessão em Minhas Conversas.
- **Passos:** 1. Em largura ≥ 768 px, conferir que a navegação inferior **não** aparece. 2. Reduzir para < 768 px e conferir as três abas. 3. Trocar para **Cliente** e **Operação** e conferir o conteúdo e o estado ativo. 4. Abrir o mesmo atendimento em `/whatsapp/minhas-conversas/{id}` (modo embutido) no mobile.
- **Resultado esperado:** a barra `.wa-mobile-nav` só existe no mobile e **não** no modo embutido; tem **três** botões (**Conversa**, **Cliente**, **Operação**), com `aria-current="page"` no ativo. A área inicial é **Conversa**; trocar de aba mostra a área correspondente e a prioridade é respeitada. No modo embutido, a alternância existente entre lista/conversa/painel é preservada.
- **Estado atual/bloqueio:** comportamento coberto por `atendimento-estados.component.spec.ts` (navegação só no mobile, sem barra no embutido e troca de área); falta a conferência visual em HML.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`, `/whatsapp/minhas-conversas/:id`; `GET /api/v1/whatsapp/atendimentos/{id}` e histórico do cliente; sem schema.
- **Status/automação:** **cenário a executar em HML — não é aceite**.
### WA-71 — Kanban estreito com rolagem horizontal e arquivados (Fase 10)

- **Objetivo:** o quadro continuar legível abaixo de 1100 px, inclusive com as colunas extras de arquivados.
- **Pré-condições:** fila com cards nas quatro colunas operacionais; pelo menos um card finalizado/arquivado; filtro "incluir arquivados" disponível.
- **Passos:** 1. Abrir `/whatsapp` em largura > 1100 px. 2. Reduzir para < 1100 px e conferir a rolagem **horizontal** do quadro. 3. Ativar "incluir arquivados" e rolar até as colunas extras. 4. Conferir o skeleton do quadro no carregamento.
- **Resultado esperado:** abaixo de **1100 px** o quadro inteiro rola na horizontal, com largura mínima legível por coluna; os cards mantêm a hierarquia visual e **não** há quebra de layout nas colunas extras de arquivados. Durante a carga aparece o skeleton do quadro (`.wa-kanban__esqueleto`), e não uma tela em branco.
- **Estado atual/bloqueio:** o comportamento está no SCSS do Kanban (`@media (max-width: 1100px)`); o spec cobre o skeleton, mas **não** a matriz visual. Falta executar em HML.
- **Rotas/endpoints/tabelas:** `/whatsapp`; `GET /api/v1/whatsapp/atendimentos`; `tbwhatsapp_atendimento`, `tbwhatsapp_atendimento_etiqueta`; sem schema.
- **Status/automação:** **cenário a executar em HML — não é aceite**.
### WA-72 — Gaveta de contatos como modal no mobile e Esc (Fase 10)

- **Objetivo:** editar um contato no mobile sem a gaveta lateral ocupar a tela inteira e com saída por teclado.
- **Pré-condições:** administrador autenticado; contato listado em `/whatsapp/admin/contatos`; navegador com largura < 768 px.
- **Passos:** 1. Abrir a edição de um contato no mobile. 2. Conferir o modal centralizado. 3. Pressionar **Esc**. 4. Reabrir, conferir o foco inicial e navegar por Tab. 5. Repetir a edição em ≥ 768 px.
- **Resultado esperado:** abaixo de **768 px** a gaveta de edição vira **modal centralizado**, com `role="dialog"` e `aria-modal="true"`; **Esc** fecha; o foco vai para a gaveta ao abrir e a navegação por teclado funciona. Em ≥ 768 px a gaveta lateral permanece. A tabela continua em contêiner rolável com largura mínima, sem espremer as colunas.
- **Estado atual/bloqueio:** o comportamento está no SCSS/HTML (`@media (max-width: 767px)`, `#wa-drawer-edicao`) e no `@HostListener('document:keydown.escape')`; o spec cobre `role="dialog"`, `aria-modal` e Esc. Falta a conferência visual em HML.
- **Rotas/endpoints/tabelas:** `/whatsapp/admin/contatos`; `GET /api/v1/whatsapp/admin/contatos`, `GET /api/v1/whatsapp/contatos/{id}`, `PATCH /api/v1/whatsapp/contatos/{id}`; `tbwhatsapp_contato`, `tbcliente`; sem schema novo.
- **Status/automação:** **cenário a executar em HML — não é aceite**.
### WA-73 — Movimento reduzido e tema escuro (Fase 10)

- **Objetivo:** respeitar `prefers-reduced-motion` e manter contraste legível nos temas claro e escuro.
- **Pré-condições:** navegador com emulação de `prefers-reduced-motion: reduce` e alternância de tema; telas de Kanban, atendimento e contatos admin.
- **Passos:** 1. Ativar movimento reduzido no sistema/navegador. 2. Abrir as três telas e conferir os skeletons. 3. Alternar para o tema escuro. 4. Repetir a leitura de textos, badges, botões e foco visível.
- **Resultado esperado:** com movimento reduzido, as animações de skeleton são desativadas; o tema escuro mantém contraste mínimo nos textos, badges, botões e no foco visível (`--ring`). As telas usam os tokens existentes do design system — não há token novo do módulo. Nenhuma informação essencial depende de hover.
- **Estado atual/bloqueio:** `prefers-reduced-motion` existe no `whatsapp.shared.scss` e nos componentes ajustados; a conferência de contraste e a matriz de temas **não foram executadas** nesta sessão. Os cenários são para HML, não um aceite.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/atendimento/:id`, `/whatsapp/admin/contatos`; sem endpoint ou schema novo.
- **Status/automação:** **cenário a executar em HML — não é aceite**.
### WA-74 — Marcar como visto e reabrir (F6)

- **Objetivo:** retirar da fila do suporte um assunto que não é do setor, **sem** assumir nem resolver, e voltar ao card quando o cliente responder.
- **Pré-condições:** atendimento em **Não atendido**, **Em atendimento** ou **Aguardando cliente**; operador com acesso ao canal.
- **Passos:** 1. Abrir `/whatsapp`. 2. Usar **Marcar como visto** no card (menu do card ou header/painel do atendimento). 3. Conferir a coluna **Visto** com “incluir arquivados” ligado. 4. Tentar assumir/resolver o card visto. 5. Enviar nova mensagem do cliente. 6. Reabrir manualmente outro card visto.
- **Resultado esperado:** `POST /api/v1/whatsapp/atendimentos/{id}/visto` devolve o detalhe com **status 6**, `mensagensNaoLidas = 0`, responsável/`responsavelDesde` nulos e evento **`visto`**. O chat também é marcado como lido no WhatsApp pelo gateway (`POST /api/sessions/{sessaoId}/chats/read`), em best-effort: falha do gateway não desfaz o Visto e `mensagensNaoLidas` continua 0 na Central. O card sai das colunas operacionais; **Visto** só aparece com “incluir arquivados”. Em Visto, `acoesDoCard` habilita apenas **Reabrir** (assumir/resolver explicam o motivo no `title`). Nova mensagem do cliente **reabre o mesmo card** (`0`, evento `reabertura`); `ReabrirAsync` manual aceita **Resolvido ou Visto**. Métricas continuam contando `emAndamento` como 1/2 e a fila como 0; Visto **não** é resolvido nem atendimento ativo no número.
- **Estado atual/bloqueio:** só é possível a partir de **0/1/2**; estado terminal devolve `409`. A coluna e o filtro de status dependem de “incluir arquivados”. Não há coluna nova: o valor `6` depende da `CHECK` 0..6.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/visto`; `tbwhatsapp_atendimento` (`WAT_Status` 6), `tbwhatsapp_evento`; `WhatsApp_Visto_Idempotente.sql`.
- **Status/automação:** **cenário derivado do código**; coberto por `WhatsAppVistoTests` (5 casos), por `WhatsAppAtendimentoTests` (Visto também marca o chat como lido e falha do gateway não desfaz o Visto) e por `whatsapp.utils.spec.ts` (matriz de ações com Visto).
### WA-75 — Histórico com eventos de finalização e arquivamento (F4)

- **Objetivo:** ver na timeline a ação real de **finalizar** e **arquivar**, com rótulo e ícone próprios, em vez de reaproveitar “Resolução”.
- **Pré-condições:** atendimento resolvido; um administrador para finalizar e arquivar.
- **Passos:** 1. Abrir o atendimento e ir à aba **Histórico**. 2. Finalizar o card. 3. Arquivar o card (manual). 4. Deixar o job de arquivamento agir em outro card, se configurado. 5. Conferir a timeline.
- **Resultado esperado:** `FinalizarAsync` grava o evento **`finalizacao`** (antes usava `resolucao`); `ArquivarAsync` e o `WhatsAppArquivamentoJob` gravam **`arquivamento`**. A timeline mostra **“Finalização”** (`bi-check2-all`) e **“Arquivamento”** (`bi-archive`) com data/hora e operador; resolver continua gravando `resolucao`. Os seis eventos que não tinham rótulo (`contato_unificado`, `etiqueta_adicionada`, `etiqueta_removida`, `investigacao_iniciada`, `investigacao_finalizada`, `observacao_atualizada`) também passam a ter rótulo/ícone próprios.
- **Estado atual/bloqueio:** a timeline já exibia qualquer evento; a mudança é de **rótulo/ícone** e de **tipo gravado**. O arquivamento automático depende da configuração do job.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /atendimentos/{id}/finalizar`, `POST /atendimentos/{id}/arquivar`; `tbwhatsapp_evento` (`WAT_Tipo` `finalizacao`/`arquivamento`).
- **Status/automação:** **cenário derivado do código**; coberto por `WhatsAppFinalizacaoTests` (2 casos, incluindo o job).
### WA-76 — Grupo por solicitante e um atendimento ativo por grupo (F5)

- **Objetivo:** exibir o grupo e a pessoa que falou, mantendo **um** atendimento ativo por grupo e trocando o solicitante no ciclo seguinte.
- **Pré-condições:** número conectado a um grupo; dois participantes enviando mensagens; atendimento de grupo aberto.
- **Passos:** 1. Receber mensagem do participante A no grupo. 2. Abrir o atendimento e conferir título/header. 3. Receber mensagem do participante B **com o atendimento ativo**. 4. Finalizar. 5. Receber nova mensagem do participante B. 6. Testar **Solicitar atendimento privado**.
- **Resultado esperado:** a conversa recebe `GrupoId` e o **primeiro** `SolicitanteId` do ciclo; o atendimento recebe o participante que abriu o ciclo. Enquanto houver atendimento ativo, as mensagens de B ficam no **mesmo** card. Depois de finalizar, a nova mensagem cria **outro** atendimento com **B** como solicitante. A listagem/detalhe preferem `Atendimento.SolicitanteId` (fallback `Conversa.SolicitanteId`); `tituloContato()` e o header mostram `grupoNome`/`solicitanteNome`. `ObterOuCriarSolicitanteGrupoAsync` atualiza nome do grupo/participante e telefone normalizado. O **privado** usa o `Contato` (não o `Participante`), o `NumeroId` do grupo e preenche `SolicitanteId` no novo atendimento.
- **Estado atual/bloqueio:** sem mudança de schema (`WAT_GrupoId`/`WAT_SolicitanteId` já existiam). Grupo sem `chatId` de solicitante fica sem solicitante.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /atendimentos/{id}/solicitar-privado`; `tbwhatsapp_grupo`, `tbwhatsapp_participante`, `tbwhatsapp_conversa`, `tbwhatsapp_atendimento`.
- **Status/automação:** **cenário derivado do código**; coberto por `WhatsAppGrupoAtendimentoTests` (5 casos).
### WA-77 — Abas Resumo/IA/Ações/Histórico estilizadas e acessíveis (F1)

- **Objetivo:** dar às quatro abas do painel de operação o padrão visual do projeto, inclusive dentro do componente filho.
- **Pré-condições:** atendimento aberto em `/whatsapp/atendimento/:id`; temas claro e escuro.
- **Passos:** 1. Abrir o atendimento. 2. Alternar **Resumo → IA → Ações → Histórico**. 3. Navegar por teclado (Tab/Enter) e conferir o foco. 4. Repetir em 1440, 1024 e 390 px.
- **Resultado esperado:** as abas usam `.wa-lado__abas`/`.wa-lado__aba`/`.wa-lado__aba--ativa`, agora em **`whatsapp.shared.scss`** (antes o SCSS da página não alcançava o componente filho por encapsulamento). Cada aba tem `id`/`aria-controls`, `role="tab"` e `aria-selected`; o painel tem `role="tabpanel"` e `aria-labelledby`. O bloco de ações (`.wa-acoes`) tem layout mínimo e o foco visível usa os tokens do design system.
- **Estado atual/bloqueio:** a matriz visual manual (claro/escuro e larguras) **não foi executada** nesta revisão; o critério é visual, ver WA-68 a WA-73 para responsividade/tema.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; sem endpoint ou schema novo.
- **Status/automação:** **cenário derivado do código**; sem spec dedicada — `npx ng build` verde nesta revisão.
### WA-78 — Áudio compacto "AUDIO" com play e velocidades (F7)

- **Objetivo:** simplificar o áudio recebido para um chip compacto, sem modais e sem os controles nativos pesados.
- **Pré-condições:** atendimento com mensagem de áudio (`audio/*`) recebida.
- **Passos:** 1. Abrir a conversa. 2. Conferir o chip **AUDIO**. 3. Tocar/pausar. 4. Baixar o arquivo. 5. Abrir **Velocidade** e trocar para `0.75x`/`2x`.
- **Resultado esperado:** `MidiaMensagemComponent` mostra chip **“AUDIO”** (com a duração quando os metadados chegam), play/pause customizado e download; o `<audio>` nativo fica **sem `controls`** e oculto (é o elemento que toca); as velocidades (`0.75`, `1`, `1.25`, `1.5`, `2`) ficam dentro de `<details>`, colapsadas. **Nenhum modal** é aberto.
- **Estado atual/bloqueio:** o blob continua sendo carregado sob demanda e autenticado; sem metadados, o rótulo fica só “AUDIO”.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `GET /api/v1/whatsapp/mensagens/{id}/midia`; sem schema novo.
- **Status/automação:** **cenário derivado do código**; sem spec dedicada — `npx ng build` verde nesta revisão.

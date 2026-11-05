---
id: 17-12-atendimento-whatsapp.19
titulo: "WA-118 — Contatos, iniciar conversa e vínculo pela conversa (rodada 2026-11-03) (+10)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, contatos, iniciar-conversa, kanban]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Services/WhatsApp
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppIniciarConversaTests.cs
  - frontend/src/app/features/whatsapp/components/whatsapp-contatos-painel/whatsapp-contatos-painel.component.spec.ts
  - frontend/src/app/features/whatsapp/components/iniciar-conversa-modal/iniciar-conversa-modal.component.spec.ts
  - frontend/src/app/features/whatsapp/pages/atendimento/atendimento-modal-responsabilidade.component.spec.ts
tokens_estimados: 2150
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-118 — Contato salvo inicia conversa pelo painel (rodada 2026-11-03)

- **Objetivo:** abrir o atendimento de um contato já salvo a partir do painel Contatos do Kanban.
- **Pré-condições:** número conectado e ativo; contato salvo com `ChatId` privado `@c.us` e sem atendimento ativo.
- **Passos:** 1. Abrir `/whatsapp` e clicar em **Contatos**. 2. Buscar o contato e clicar em **Conversar**. 3. Confirmar no `iniciar-conversa-modal`. 4. Conferir a navegação e `tbwhatsapp_atendimento`/`tbwhatsapp_conversa`.
- **Resultado esperado:** `POST /atendimentos/iniciar` recebe `{ numeroId, chatId }` e responde `200 { atendimentoId, conversaId, status, criado }`; a tela navega para `/whatsapp/atendimento/:atendimentoId`. A conversa é reaproveitada por (`NumeroId`, `ChatId`) com o `ContatoId` do contato salvo; o atendimento nasce `EmAtendimento` com responsável = operador e evento de atribuição. **Nenhuma mensagem é enviada ao gateway** e nenhum contato é criado. Auditoria `iniciar_conversa` com número, chat e operador.
- **Estado atual/bloqueio:** `AtendimentoService.IniciarConversaAsync`; `whatsapp-contatos-painel` (`conversar`) e `iniciar-conversa-modal`.
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/iniciar`; `/whatsapp/atendimento/:id`; `tbwhatsapp_atendimento`, `tbwhatsapp_conversa`, `tbwhatsapp_evento`, `tbwhatsapp_auditoria`.
- **Status/automação:** **cenário derivado do código**; `WhatsAppIniciarConversaTests.cs` e `whatsapp-contatos-painel.component.spec.ts`.

### WA-119 — Número novo inicia por telefone sem criar contato (rodada 2026-11-03)

- **Objetivo:** iniciar uma conversa com número que não tem contato salvo, sem cadastrá-lo.
- **Pré-condições:** número conectado e ativo; telefone com DDD válido e sem contato por `ChatId` ou telefone normalizado.
- **Passos:** 1. Abrir **Contatos** no Kanban. 2. No bloco **"Nova conversa com número"**, digitar o telefone e confirmar. 3. Conferir `tbwhatsapp_contato` e a conversa gravada.
- **Resultado esperado:** o telefone é normalizado por `NormalizarParaCartao` (DDI 55, com o 9) e vira `chatId = "<cartão>@c.us"`; a resposta é `200` com `criado = true`. A conversa nasce com `ContatoId` **nulo** e o atendimento `EmAtendimento`; `tbwhatsapp_contato` **não** ganha linha. O gateway não recebe mensagem.
- **Estado atual/bloqueio:** `MontarChatIdDeTelefone` e `IniciarConversaAsync`; validação/máscara no painel.
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/iniciar`; `tbwhatsapp_conversa`, `tbwhatsapp_atendimento`, `tbwhatsapp_contato`.
- **Status/automação:** **cenário derivado do código**; `WhatsAppIniciarConversaTests.cs`.

### WA-120 — Atendimento ativo é assumido ou reaproveitado (rodada 2026-11-03)

- **Objetivo:** não duplicar atendimento quando a conversa já tem card ativo.
- **Pré-condições:** conversa com atendimento em `NaoAtendido`, `EmAtendimento`, `AguardandoCliente` ou `Visto`; um caso sem responsável e outro já do próprio operador.
- **Passos:** 1. Iniciar a conversa pelo painel com o card sem responsável. 2. Repetir com o card já atribuído ao mesmo operador. 3. Conferir a lista de atendimentos da conversa.
- **Resultado esperado:** sem responsável, o service reutiliza `AssumirAsync` e devolve `criado = false` com o atendimento assumido; se já for do operador, reaproveita o mesmo card. Não há novo atendimento nem novo evento de abertura duplicado.
- **Estado atual/bloqueio:** `IniciarConversaAsync` (status ativos e `AssumirAsync`).
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/iniciar`; `tbwhatsapp_atendimento`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário derivado do código**; `WhatsAppIniciarConversaTests.cs`.

### WA-121 — Atendimento de outro operador devolve 409 e abre o modal (rodada 2026-11-03)

- **Objetivo:** impedir que um operador invada o atendimento ativo de outro pelo painel.
- **Pré-condições:** conversa com atendimento ativo e responsável diferente do operador logado.
- **Passos:** 1. Abrir **Contatos** e clicar em **Conversar** no contato. 2. Confirmar no modal. 3. Conferir o corpo do `409` e o modal exibido. 4. Assumir pelo modal e repetir.
- **Resultado esperado:** `409` com `motivo: "responsavel_diferente"`, `responsavelNome` e `atendimentoId`; a tela fecha o `iniciar-conversa-modal` e abre o `atendimento-modal-responsabilidade`. Assumindo, a conversa segue para o atendimento. Nenhum atendimento novo é criado antes da assunção.
- **Estado atual/bloqueio:** `ConflictException` em `IniciarConversaAsync`; `whatsapp-contatos-painel` (`aoConfirmarInicio`) e `atendimento-modal-responsabilidade`.
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/iniciar`; `tbwhatsapp_atendimento`.
- **Status/automação:** **cenário derivado do código**; `WhatsAppIniciarConversaTests.cs` e specs do painel/modal.

### WA-122 — Payload inválido do iniciar devolve 400 (rodada 2026-11-03)

- **Objetivo:** validar o contrato de entrada do iniciar conversa.
- **Pré-condições:** número conectado; operador autenticado.
- **Passos:** 1. Enviar `numeroId` com `chatId` **e** `telefone`. 2. Repetir sem nenhum dos dois. 3. Enviar `chatId` de grupo (`@g.us`) e de `@lid`. 4. Enviar telefone sem DDD, com DDD < 11 e com tamanho fora de 12–13 dígitos.
- **Resultado esperado:** todos os casos respondem `400 { mensagem }`: os dois/nenhum ("Informe o chat ou o telefone, não os dois."), grupo ("Grupo não é conversa privada."), `@lid` ("Use o telefone: @lid não é número."), JID inválido, telefone inválido ou DDD inválido. Nada é gravado e a tela mostra o motivo no campo de telefone quando aplicável.
- **Estado atual/bloqueio:** `IniciarConversaAsync` (`ArgumentException`) e `ValidarChatIdPrivado`/`MontarChatIdDeTelefone`.
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/iniciar`; `tbwhatsapp_atendimento`, `tbwhatsapp_conversa`.
- **Status/automação:** **cenário derivado do código**; `WhatsAppIniciarConversaTests.cs`.

### WA-123 — Número ou sessão indisponível devolve 404 (rodada 2026-11-03)

- **Objetivo:** tratar número inexistente ou desconectado sem abrir conversa.
- **Pré-condições:** um `numeroId` inexistente e um número cadastrado com sessão parada/desconectada.
- **Passos:** 1. Enviar `POST /atendimentos/iniciar` com `numeroId` inexistente. 2. Repetir com número inativo/desconectado. 3. Conferir a tela.
- **Resultado esperado:** `404 { mensagem: "Número indisponível. Verifique a conexão." }` nos dois casos; nenhuma conversa/atendimento é criado e a tela exibe a mesma mensagem no modal.
- **Estado atual/bloqueio:** `IniciarConversaAsync` devolve `null`; `SessaoDesconectada` cobre `disconnected`/close/closed/stopped/created/qr_ready.
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/iniciar`; `tbwhatsapp_numero`.
- **Status/automação:** **cenário derivado do código**; `WhatsAppIniciarConversaTests.cs`.

### WA-124 — Atendimento terminal abre novo card na mesma conversa (rodada 2026-11-03)

- **Objetivo:** iniciar uma nova conversa quando o atendimento anterior já terminou.
- **Pré-condições:** conversa com atendimento em `Resolvido`, `Finalizado` ou `Arquivado`; número conectado.
- **Passos:** 1. Iniciar a conversa pelo painel. 2. Conferir status, responsável e a lista de atendimentos da conversa.
- **Resultado esperado:** é criado um **novo** atendimento na mesma conversa, com `EmAtendimento`, responsável = operador e evento de atribuição; o atendimento terminal permanece intacto. A resposta traz `criado = true` e o novo `atendimentoId`.
- **Estado atual/bloqueio:** `IniciarConversaAsync` (status terminais e transação de criação).
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/iniciar`; `tbwhatsapp_atendimento`, `tbwhatsapp_conversa`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário derivado do código**; `WhatsAppIniciarConversaTests.cs`.

### WA-125 — Painel Contatos no Kanban (rodada 2026-11-03)

- **Objetivo:** consultar contatos e iniciar conversa sem sair do Kanban.
- **Pré-condições:** número conectado; contatos salvos e chats sem contato; pelo menos dois números conectados em um dos casos.
- **Passos:** 1. Abrir `/whatsapp` e clicar em **Contatos** no cabeçalho. 2. Buscar, paginar e conferir avatar/nome/empresa/telefone. 3. Abrir **"Nova conversa com número"** e o `iniciar-conversa-modal`. 4. Alternar entre números conectados. 5. Fechar o painel.
- **Resultado esperado:** o painel lateral abre **sem trocar de rota**; `listarContatos(busca, pagina, tamanho)` consome `GET /contatos` paginado e o vazio/erro tem mensagem em PT-BR. O modal mostra o destinatário e o seletor de número **só** com mais de um conectado, sem campo de mensagem inicial; "Cancelar" fecha; "Iniciar e abrir" chama `iniciarConversa`. Contatos `@lid` sem telefone utilizável não habilitam "Conversar".
- **Estado atual/bloqueio:** `whatsapp-kanban.component.*` (botão Contatos), `whatsapp-contatos-painel`, `iniciar-conversa-modal`; `whatsapp.service.ts` (`iniciarConversa`, `listarContatos`) e `whatsapp.model.ts`.
- **Rotas/endpoints/tabelas:** `/whatsapp` (sem rota nova no painel); `GET /api/v1/whatsapp/contatos`, `POST /api/v1/whatsapp/atendimentos/iniciar`.
- **Status/automação:** **cenário derivado do código**; specs do painel, do modal, da responsabilidade e do Kanban.

### WA-126 — Vincular empresa pela conversa (rodada 2026-11-03)

- **Objetivo:** cadastrar o contato direto do painel da conversa, sem passar pelo Admin e sem gravar nada antes da confirmação do operador.
- **Pré-condições:** chat privado sem contato (`contexto.salvo === false`); mensagem recebida com pushName; operador com `perm:aba.suporte.whatsapp:Ler`; uma empresa ativa no ERP.
- **Passos:** 1. Abrir `/whatsapp/atendimento/:id` de um chat sem contato. 2. Conferir o bloco **"Contato ainda não vinculado"** com Nome/Telefone e que `tbwhatsapp_contato` continua sem linha. 3. Clicar em **Vincular empresa**, deixar o **Nome** em branco e confirmar — deve bloquear. 4. Preencher o Nome, escolher a empresa e confirmar. 5. Repetir em um chat já identificado e em um `chatId` sem conversa.
- **Resultado esperado:** o `vincular-empresa-modal` recebe `chatId`, `nomeSugerido` (pushName) e `telefoneSugerido` e exige **Nome**; `POST /contatos/vincular` responde `200 ContatoFichaDto` e cria o contato, o bloco vira **"Cliente identificado"** e o atendimento recarrega em silêncio. Chat já identificado → `409 CONFLITO_CONTATO_IDENTIFICADO`; `chatId` sem conversa existente → `400` "Chat não encontrado: selecione uma conversa existente."; sem acesso ao canal → `403`. O overlay `contato-whatsapp` permanece **somente leitura**.
- **Estado atual/bloqueio:** `cliente-context-panel` (`abrirVinculo`/`aoVincular`), `vincular-empresa-modal` e `aoIdentificacaoAtualizada`; `ContatoVinculoService.CriarEVincularAsync` e `WhatsAppContatosController.VincularNovo`.
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/contatos/vincular`; `/whatsapp/atendimento/:id`; `tbwhatsapp_contato`, `tbwhatsapp_conversa`, `tbwhatsapp_auditoria`.
- **Status/automação:** **cenário derivado do código**; suíte `WhatsApp*`, specs do painel/modal e QA manual.

### WA-127 — Conversar a partir dos integrantes do grupo (rodada 2026-11-03)

- **Objetivo:** abrir o atendimento privado de um integrante a partir do modal **"Integrantes"** do atendimento de grupo, enviando pelo número do próprio grupo.
- **Pré-condições:** atendimento de grupo aberto com `resumo.numeroId` de número conectado; integrantes com `@c.us`, `@lid` com telefone resolvido e `@lid` sem telefone.
- **Passos:** 1. Abrir `/whatsapp/atendimento/:id` de um grupo e clicar em **Integrantes**. 2. Conferir o botão **Conversar** em cada linha. 3. Clicar em **Conversar** no integrante `@c.us`. 4. Repetir no `@lid` com telefone resolvido. 5. Conferir o `@lid` sem telefone. 6. Repetir com o atendimento privado já sob outro operador.
- **Resultado esperado:** o modal recebe `[numeroId]` = `resumo.numeroId` do `cliente-context-panel` e envia `POST /atendimentos/iniciar` com `{ numeroId, chatId }` para `@c.us` e `{ numeroId, telefone }` (telefone resolvido) para `@lid`; o `@lid` sem telefone utilizável **não** habilita o botão. `200` fecha o modal e navega para `/whatsapp/atendimento/{atendimentoId}`; `409 responsavel_diferente` mostra "O atendimento está com {responsável}. Assuma para continuar." **sem** abrir o modal de responsabilidade. Nenhum contato é criado e o atendimento privado é aberto ou reaproveitado.
- **Estado atual/bloqueio:** `grupo-integrantes-modal.component.ts` (`podeConversar`/`conversar`/`rotularErroInicio`) e `cliente-context-panel.component.html` (`[numeroId]`); `AtendimentoService.IniciarConversaAsync`.
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/iniciar`; `/whatsapp/atendimento/:id`; `tbwhatsapp_atendimento`, `tbwhatsapp_conversa`.
- **Status/automação:** **cenário derivado do código**; `grupo-integrantes-modal.component.spec.ts` e `WhatsAppIniciarConversaTests.cs`.

### WA-128 — Filtrar contatos pela empresa do vínculo (rodada 2026-11-03)

- **Objetivo:** restringir o painel Contatos do Kanban à empresa do vínculo com o ERP, sozinho ou junto da busca textual.
- **Pré-condições:** número conectado; contatos vinculados a pelo menos duas empresas distintas e um contato sem vínculo; empresas ativas na `tbcliente`.
- **Passos:** 1. Abrir `/whatsapp` e clicar em **Contatos**. 2. Digitar ao menos **2 caracteres** no typeahead de empresa e selecionar uma. 3. Conferir o chip e a lista. 4. Paginar, depois trocar a empresa e observar a página. 5. Combinar o filtro com a **busca** textual. 6. Remover o chip e conferir a página.
- **Resultado esperado:** `GET /contatos` recebe `empresaId` (além de `busca`, `pagina`, `tamanho`) e filtra por `WAT_EmpresaId`; o filtro combina em **AND** com a busca textual. O typeahead (`GET /contatos/empresas`) só consulta com **2+ caracteres**, selecionar mostra o **chip** e limpar o remove; qualquer seleção/limpeza volta para a **página 1**. Só aparecem contatos da empresa escolhida — contatos de outras empresas (e sem vínculo) ficam fora. Disponível a qualquer operador autenticado (não exige admin).
- **Estado atual/bloqueio:** `whatsapp-contatos-painel` (`empresaSelecionada`, `selecionarEmpresa`, `limparEmpresa`, `aoDigitarEmpresa`); `WhatsAppContatosController.Listar` e `ContatoUnificacaoService.ListarAsync`.
- **Rotas/endpoints/tabelas:** `GET /api/v1/whatsapp/contatos` (`empresaId`), `GET /api/v1/whatsapp/contatos/empresas`; `/whatsapp`; `tbwhatsapp_contato` (`WAT_EmpresaId`), `tbcliente`.
- **Status/automação:** **cenário derivado do código**; `whatsapp-contatos-painel.component.spec.ts` e testes de `ContatoUnificacaoService`.

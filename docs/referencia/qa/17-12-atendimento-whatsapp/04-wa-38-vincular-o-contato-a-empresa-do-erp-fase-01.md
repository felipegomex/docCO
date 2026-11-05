---
id: 17-12-atendimento-whatsapp.04
titulo: "WA-38 — Vincular o contato à empresa do ERP (Fase 01) (+7)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias]
status: ativo
revisado_em: "2026-10-30"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Controllers/WhatsApp
  - backend/Central_BackEnd/Services/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - frontend/src/app/features/whatsapp/pages/kanban/whatsapp-kanban.component.spec.ts
tokens_estimados: 3750
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-38 — Vincular o contato à empresa do ERP (Fase 01)

- **Objetivo:** o contato **pendente** sair do atendimento com empresa, tipo e observação gravados, sem que o WhatsApp cadastre identidade própria.
- **Pré-condições:** atendimento aberto com contato **sem vínculo** (pendente); operador autenticado com acesso ao canal.
- **Passos:** 1. Abrir `/whatsapp/atendimento/:id`. 2. No painel de contexto (coluna esquerda), usar **Identificar** — o modal busca a empresa (mínimo 2 caracteres; fantasia, razão social ou CNPJ com/sem máscara). 3. Selecionar a empresa, classificar o tipo (não classificado/TI/cobrança/gestor/planejamento/**supervisor de cobrança**) e salvar a observação. 4. Confirmar. 5. Conferir no topo do modal o número do contato e, quando houver, a linha **“Número normalizado”**.
- **Resultado esperado:** `GET /api/v1/whatsapp/contatos/empresas` devolve só empresas **ativas** da `tbcliente`; o `PATCH /api/v1/whatsapp/contatos/{id}` envia `empresaId`, `tipoContato`, `observacao` e a **`versao`** da ficha (lock otimista). O `tipoContato` aceita **0–5** (`Enum.IsDefined` no `ContatoVinculoService`) — **5 = Supervisor de Cobrança** (`SupervisorCobranca`); fora da faixa responde `400`. A resposta é a ficha recém-lida: fantasia/razão social e grupo vêm **do ERP em tempo real** (nada é copiado). A ação é auditada (`acao "vincular"`). No modal, o número exibido é o `telefoneExibicao` (prioridade `resolvido ?? normalizado ?? numero`), com fallback `chatId`; quando `telefoneNormalizado` difere dele, aparece a linha **“Número normalizado”**.
- **Estado atual/bloqueio:** o WhatsApp **nunca** cadastra empresa; não existe sugestão por DDD — o vínculo nasce de busca e seleção explícita. Campos nulos no PATCH **mantêm** o valor atual. O estado pendente/identificado é **derivado** de `empresaId` ser nulo ou não.
- **Evidência de execução (2026-10-30 — pré-reset):** backfill de telefone dos contatos `@lid` em `dbCENTRAL_OPERACAO` pelo resolvedor do gateway (`GET /api/sessions/{sessao}/contacts/{chatId}/phone`): **43 atualizados de 43, 0 falhas**, com `WAT_TelefoneResolvido`/`WAT_Numero` normalizados e `WAT_UsuarioAlteracao='openwa'`; os **10** contatos `@g.us` seguiam sem telefone. **Esta evidência é pré-reset: os dados foram apagados na limpeza de 2026-10-30** (`scripts/whatsapp-reset.ps1 -Ambiente AMBOS -Confirmar`, 13 tabelas operacionais zeradas em `dbCENTRAL_OPERACAO` e `dbBUSINESS_HML`, incluindo `tbwhatsapp_contato`) — **não há telefone resolvido gravado hoje**; o módulo está zerado, com o gateway sem sessão, aguardando a reconexão de um número para refazer o backfill. Execução **manual**, sem teste automatizado para o backfill.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `GET /api/v1/whatsapp/contatos/{id}`, `GET /api/v1/whatsapp/contatos/empresas`, `PATCH /api/v1/whatsapp/contatos/{id}`; `tbwhatsapp_contato` (`WAT_EmpresaId`, `WAT_TipoContato`, `WAT_Observacao`, `WAT_Versao`), `tbcliente`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-39 — Concorrência e administração do vínculo (409 e 403 — Fase 01)

- **Objetivo:** dois operadores mexendo na mesma ficha não sobrescreverem um ao outro, e a troca/remoção de vínculo de contato **já identificado** ficar restrita ao administrador.
- **Pré-condições:** mesma ficha aberta por dois operadores; e, no segundo caso, um contato **já identificado** e um operador sem papel administrativo.
- **Passos:** 1. Operador A abre a ficha e fica parado. 2. Operador B vincula uma empresa à mesma ficha. 3. Operador A tenta salvar com a versão antiga. 4. Operador não-admin tenta **trocar** (ou remover) o vínculo de um contato já identificado.
- **Resultado esperado:** o caso 3 responde **`409`** com código `CONFLICT_CONTATO_VERSAO` e o corpo traz `{ mensagem, codigo, contato }` com a **ficha atual** — a tela recarrega sem fechar a conversa (o lock é duplo: checagem prévia e `UPDATE` com `WHERE WAT_Versao = <enviada>`, com `DbUpdateConcurrencyException` traduzida). O caso 4 responde **`403`** ("exige perfil administrativo") — **o controller checa e o service revalida** (a tela não concede).
- **Estado atual/bloqueio:** vincular empresa a contato **pendente** continua sendo de qualquer atendente; só a troca/remoção de vínculo de **identificado** (D06) é administrativa. Em InMemory (testes) a rowversion não existe; o service garante valor para o próximo PATCH continuar com lock.
- **Rotas/endpoints/tabelas:** `PATCH /api/v1/whatsapp/contatos/{id}`; `tbwhatsapp_contato.WAT_Versao` (rowversion).
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-40 — Layout Service Desk: cabeçalho fixo e 3 colunas recolhíveis (Fase 02)

- **Objetivo:** o atendente enxergar "quem é / em que estado está / qual SLA" e as quatro ações principais sem rolar a página, com foco configurável na conversa.
- **Pré-condições:** atendimento aberto em `/whatsapp/atendimento/:id`; desktop largo (≥ 1440px).
- **Passos:** 1. Abrir o atendimento. 2. Conferir o cabeçalho fixo (empresa/grupo, contato, status, SLA, não lidas, responsável e ações Assumir/Transferir/Aguardar/Resolver). 3. Recolher a coluna esquerda e a direita pelos toggles do cabeçalho. 4. Recarregar a página. 5. Repetir em `/whatsapp/minhas-conversas/:id`.
- **Resultado esperado:** o cabeçalho é **sticky** (a conversa rola, ele fica) e as ações aparecem desabilitadas com `title` explicativo quando o estado não permite. Os toggles recolhem as colunas **sem desmontar a conversa nem perder o rascunho**; a preferência é persistida por operador/estação em `localStorage` (`wa.atendimento.colunas`) e sobrevive ao reload. No desktop largo (≥ 1440px) as 3 colunas ficam a vista: contexto do cliente · conversa · operação. No **modo embutido** não há coluna esquerda (o toggle do cliente some) e o painel consultivo clássico mantém a lista + conversa + abas.
- **Estado atual/bloqueio:** a preferência de colunas é **local** — nenhum dado operacional vai ao banco; outro computador começa do padrão.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`, `/whatsapp/minhas-conversas/:id`; nenhum endpoint novo; componente `WhatsappAtendimentoHeaderComponent`.
- **Status/automação:** **cenário documentado**; há spec de componente (`whatsapp-atendimento-header.component.spec.ts`), sem suíte executada no pipeline.
### WA-41 — Contexto do cliente na coluna esquerda (Fase 03)

- **Objetivo:** abrir o atendimento e já saber quem é o cliente, a observação permanente e o histórico — sem chamada extra.
- **Pré-condições:** atendimento aberto; contato com (ou sem) vínculo de empresa.
- **Passos:** 1. Abrir `/whatsapp/atendimento/:id`. 2. Ler a coluna esquerda: identificação, observação, grupo/solicitante e os 5 últimos atendimentos. 3. Abrir um atendimento vindo de **grupo de WhatsApp**, sem vínculo de empresa no contato.
- **Resultado esperado:** o contexto vem **na mesma leitura do atendimento** (`AtendimentoDetalhe.Contato` = `ContatoContextoDto` — sem N+1): identificação pendente/identificada, empresa e grupo do ERP, tipo, e a **observação em card destacado sempre visível** (sem card vazio quando não há observação). O grupo do WhatsApp e o solicitante aparecem **separados da empresa** — a empresa **não é presumida** para o grupo inteiro. A lista mostra os **5 atendimentos mais recentes** com o atual destacado; o "Ver todos" continua **desabilitado ("em breve")** — a gestão de contatos da Fase 08 é uma tela própria (`/whatsapp/admin/contatos`, ver WA-59 a WA-61) e o componente nunca aponta para rota inexistente. Erro de histórico não oculta a identificação.
- **Estado atual/bloqueio:** a coluna é de **leitura** — alterações de vínculo passam pelo modal (WA-38).
- **Rotas/endpoints/tabelas:** `GET /api/v1/whatsapp/atendimentos/{id}` (campo `contato`); `tbwhatsapp_contato`, `tbcliente`, `tbcliente_grupo`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-42 — Conversa, eventos e compositor com seletor de destino (Fase 04)

- **Objetivo:** a conversa mostrar **somente as mensagens** do atendimento atual (com os eventos operacionais consultáveis no **Histórico**), e o compositor deixar claro **para onde** cada envio vai — sem nota interna escapar para o WhatsApp.
- **Pré-condições:** atendimento aberto com mensagens e eventos (atribuição, transferência etc.) no histórico.
- **Passos:** 1. Conferir a conversa: **apenas mensagens** do atendimento atual — nenhum evento como balão ou linha no chat. 2. Abrir a aba/seção **Histórico** e conferir os eventos operacionais lá. 3. Escrever no compositor com anexo **ou** contato pendente e **trocar o destino para "Nota interna"**. 4. Anexar imagem/áudio/vídeo/GIF acima de 10 MB ou em SVG/documento. 5. Usar `Ctrl+Enter`, respostas rápidas e o botão de IA. 6. Clicar duas vezes no envio. 7. Reproduzir um áudio recebido.
- **Resultado esperado:** a conversa mostra **somente mensagens**; os eventos operacionais aparecem **apenas na aba/seção Histórico** (`TimelineEventoComponent` / `AtendimentoHistoryPanelComponent`) e **nunca** como balão no chat. O compositor tem **seletor explícito de destino** ("Mensagem WhatsApp" × "Nota interna") e o botão diz para onde vai; **trocar de modo mantém o texto e descarta o anexo/contato com aviso** ("nota interna não envia mídia/contato"). Anexo é validado no cliente e revalidado no backend: **10 MB**, MIME da lista fechada — imagem **sem SVG**, áudio, `video/mp4`, `video/webm` e `image/gif` (Fase 6 do plano 2026-10; antes eram só imagem/áudio). O mini-form de **contato** (Fase 7) é mutuamente exclusivo com o anexo e também é bloqueado aqui. `Ctrl+Enter` envia; respostas rápidas substituem `{nome}`/`{protocolo}`/`{numero}` (variável sem valor continua visível); o botão de IA **só abre o assistente**; o envio é bloqueado após o primeiro clique (nada de mensagem duplicada). O player de áudio tem **velocidade 0,75x a 2x** aplicada sem recarregar o blob.
- **Estado atual/bloqueio:** a nota interna continua **nunca** indo ao gateway (ver WA-08); o seletor é o que impede o engano no dia a dia.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/mensagens`, `POST .../notas`, `POST .../mensagens/midia`, `POST .../mensagens/contato`, `GET /api/v1/whatsapp/mensagens/{id}/midia`; `tbwhatsapp_mensagem`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; há specs de compositor e cabeçalho, sem suíte executada no pipeline.
### WA-43 — SLA crítico antes do vencimento com percentual configurável (Fase 05)

- **Objetivo:** a fila piscar **antes** de o SLA vencer, com o ponto de aviso configurável e as duas dimensões (primeira resposta e resolução) avaliadas de forma independente.
- **Pré-condições:** administrador para editar o percentual; metas e horário útil configurados; atendimento em andamento.
- **Passos:** 1. Em `/whatsapp/sla`, ajustar o **percentual de aviso** (ex.: 80) e salvar. 2. Acompanhar um atendimento até 80% da meta. 3. Abrir o atendimento e conferir o painel de Resumo. 4. Tentar salvar o percentual fora de 50–99.
- **Resultado esperado:** o `PUT /sla/configuracao` grava `percentualAviso` (padrão 80) e a mudança **vale na hora, sem deploy**; valor fora de **50–99** responde `400` (validação do service e `CHECK` no banco). A partir do percentual da meta o card sai de `no_prazo` para **`critico`** (rótulo "Crítico", badge laranja) — antes do vencimento; a 100% vira `atrasado`; resolvido depois da meta fica `estourado`. O painel mostra a barra **decorrido/meta (limitada a 100%)** e as situações **independentes** de primeira resposta e resolução, com a situação geral do card = **pior caso** (`SlaCalculo.PiorSituacao`); o `AtendimentoResumo.SlaSituacao` alimenta Kanban e cabeçalho sem recálculo na tela.
- **Estado atual/bloqueio:** sem meta ou sem horário útil a tela mostra o tempo **sem** veredito (`null`); config antiga com percentual fora da faixa é ignorada pelo cálculo (só no_prazo/atrasado).
- **Rotas/endpoints/tabelas:** `/whatsapp/sla`, `/whatsapp/atendimento/:id`; `GET /api/v1/whatsapp/sla`, `PUT /api/v1/whatsapp/sla/configuracao`, `GET /api/v1/whatsapp/atendimentos/{id}` (campo `slaOperacao`); `tbwhatsapp_sla_config.WAT_PercentualAviso` (CHECK 50..99), `tbwhatsapp_sla_historico`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-44 — Aplicar e remover etiquetas do atendimento (Fase 06)

- **Objetivo:** classificar o atendimento com etiquetas operacionais, incluindo remover todas.
- **Pré-condições:** catálogo com etiquetas ativas; atendimento aberto; operador atendente autenticado.
- **Passos:** 1. Abrir `/whatsapp/atendimento/:id` e a aba **Resumo**. 2. No seletor, buscar e adicionar uma etiqueta. 3. Conferir o chip e a conversa. 4. Remover a etiqueta; em outro atendimento, salvar o conjunto vazio.
- **Resultado esperado:** `PUT /atendimentos/{id}/etiquetas` substitui o conjunto (`etiquetaIds[]`; vazio é válido e remove todas). A resposta é o `AtendimentoDetalhe` relido do banco; a tela troca o estado **sem F5**. Cada diferença real grava `etiqueta_adicionada`/`etiqueta_removida`; a definição da etiqueta **nunca** é apagada. Aplicar etiqueta **não** altera responsável, status nem SLA.
- **Estado atual/bloqueio:** IDs inexistentes ou inativos → `400`; o conjunto é deduplicado no backend.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id` (aba Resumo); `GET /api/v1/whatsapp/etiquetas`, `PUT /api/v1/whatsapp/atendimentos/{id}/etiquetas`; `tbwhatsapp_etiqueta`, `tbwhatsapp_atendimento_etiqueta`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; `WhatsAppEtiquetaTests` cobre aplicação/remoção no backend — incluindo a **substituição do conjunto na mesma chamada** (adiciona uma e remove outra, com os dois eventos e os vínculos finais) — e `WhatsAppQueryTraducaoTests` prova que a consulta do catálogo por ids gera `IN (...)` literal, **sem `OPENJSON`** (banco legado); o spec de `EtiquetaSelectorComponent` cobre adicionar/remover no frontend.
### WA-45 — Conjunto igual não gera evento (Fase 06)

- **Objetivo:** salvar o mesmo conjunto de etiquetas não poluir o histórico.
- **Pré-condições:** atendimento já com as etiquetas A e B.
- **Passos:** 1. Abrir a aba **Resumo**. 2. Salvar novamente o conjunto A + B (mesma ordem ou não). 3. Conferir o histórico de eventos e os vínculos.
- **Resultado esperado:** o diff do `AtendimentoService` não encontra adição nem remoção: nenhuma linha nova em `tbwhatsapp_atendimento_etiqueta` e **nenhum** evento `etiqueta_adicionada`/`etiqueta_removida`. A resposta continua `200` com o atendimento relido.
- **Estado atual/bloqueio:** a deduplicação dos ids e o diff contra o conjunto atual são feitos no backend; a tela só envia o conjunto desejado.
- **Rotas/endpoints/tabelas:** `PUT /api/v1/whatsapp/atendimentos/{id}/etiquetas`; `tbwhatsapp_atendimento_etiqueta`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; coberto por `WhatsAppEtiquetaTests` (evento apenas quando o conjunto muda).

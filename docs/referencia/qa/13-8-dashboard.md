---
id: sec.13-8-dashboard
titulo: "8. Dashboard"
tipo: referencia
area: qa
sistema: central
tags: [qa, historias, testes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 3536
ficha: docs/modulos/dashboard.md
---
> Ficha: [Dashboard Integrado](../../modulos/dashboard.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 8. Dashboard

### DBN-01 — Consultar a home do Dashboard

- **Objetivo:** Exibir em uma tela os KPIs comercial e administrativo, com atalhos para os três módulos.
- **Pré-condições:** operador autenticado; API `/api/v1/dashboard/*` acessível.
- **Passos:** 1. Abrir `/dashboard` pela Sidebar (grupo `Dashboard`, abaixo de Início/Agenda). 2. Conferir os cartões de KPI de Comercial e de Administrativo. 3. Navegar pelos atalhos para os submódulos.
- **Resultado esperado:** a home carrega **apenas** `GET /comercial/kpi` e `GET /administrativo/resumo` (com `catchError` → `null`, então falha de API vira aviso sem quebrar a página) e oferece acesso a Comercial, Administrativo e Financeiro (este último só para `Administrador`). **Não há bloco "Financeiro" nem cards de Faturamento mensal / Reajustes 90d**: o `forkJoin` não chama `/financeiro/kpis`.
- **Estado atual/bloqueio:** pendências conhecidas: integração do contexto JOTA não implementada e a automação de contrato a partir de `Negócio Fechado` está adiada (só grava `UltimoContato`). Os indicadores financeiros foram retirados da home porque **todo valor de faturamento é uma projeção de `tbcontrato` ainda não liberada** (`FlagsDashboard.FaturamentoLiberado = false`); o card Financeiro permanece como atalho, com a descrição “Custos mensais, orçamentos e evolução”.
- **Rotas/endpoints/tabelas:** `/dashboard`; `GET /api/v1/dashboard/comercial/kpi`, `GET /api/v1/dashboard/administrativo/resumo`; `tblead`, `tboportunidade`, `tbcliente`, `tbcontrato`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.

### DBN-02 — Operar o kanban de Leads

- **Objetivo:** Mover leads pelo funil comercial (Prospecção → Apresentação → Proposta → Negócio Fechado) e mantê-los atualizados.
- **Pré-condições:** operador autenticado; leads existentes ou cadastro novo; migrations `20260928182047_LeadContatoVisualizado` e `20261008090000_ComercialFunilProdutos` aplicadas no banco.
- **Passos:** 1. Abrir `/dashboard/comercial`. 2. Buscar um lead pelo campo de busca. 3. Arrastar um card para outra coluna (CDK Drag&Drop). 4. Criar um lead pelo botão de novo (modal `LeadModalComponent`). 5. Clicar no toggle "Viu/Não viu" do card para alternar o flag de visualização do contato. 6. Arquivar um lead em `Negócio Fechado` (ícone de arquivo, informa o motivo) e conferir que ele sai da visão. 7. Clicar em "Ver arquivados", conferir o lead arquivado e desarquivar. 8. Editar e excluir com confirmação.
- **Resultado esperado:** a movimentação é otimista e usa rollback se a API falhar; criar/editar validam título e cliente; o toggle de visualização é otimista com rollback e reverte com toast em caso de erro; arquivar remove o card da visão ativa (histórico preservado) e desarquivar o traz de volta; excluir leva junto as oportunidades do lead; o aviso do módulo aparece via `DashboardToastService`.
- **Estado atual/bloqueio:** limitação conhecida: mover para `Negócio Fechado` apenas grava `UltimoContato` — a criação automática de contrato está adiada por decisão. O módulo tem apenas 2 abas (Kanban de Leads e Propostas); a rota antiga `/dashboard/comercial/oportunidades` redireciona para o Kanban.
- **Rotas/endpoints/tabelas:** `/dashboard/comercial`; `GET/POST/PUT/DELETE /api/v1/dashboard/comercial/leads` (query `incluirArquivados`/`somenteArquivados`), `PATCH /leads/{id}/etapa`, `PATCH /leads/{id}/contato-visualizado`, `PATCH /leads/{id}/arquivar`, `PATCH /leads/{id}/desarquivar`; `tblead`, `tboportunidade`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.

### DBN-03 — Criar oportunidade a partir do lead e operar as Propostas

- **Objetivo:** ofertar produtos do catálogo (`tbproduto`) a um lead em Proposta e acompanhar o ciclo das propostas enviadas.
- **Pré-condições:** operador autenticado; lead existente **na etapa `Proposta`** (fora dela a API recusa com `400`); colunas de `20261008090000_ComercialFunilProdutos` aplicadas.
- **Passos:** 1. Mover o lead para `Proposta`. 2. Abrir o card e, no bloco "Oportunidades / Produtos" → "Nova", marcar um ou mais produtos do checklist. 3. Informar valor de implementação, valor por PA/licença, estimativa de PAs e observação; clicar em "Ofertar produtos marcados". 4. Conferir a lista (produto, composição de valores, observação) e o total do bloco. 5. Abrir `/dashboard/comercial/propostas`. 6. Marcar proposta enviada/visualizada e registrar a resposta (SIM/NÃO); ao marcar NÃO, informar o motivo da perda. 7. Arquivar o lead de proposta não convertida. 8. Limpar a resposta quando necessário. 9. Com a lista vazia, usar o link do empty state para voltar ao Kanban.
- **Resultado esperado:** o bloco de produtos só existe na edição e na etapa `Proposta`; cada produto marcado cria 1 linha em `tboportunidade` com `ValorEstimado = implementação + porLicença × qtde`; enviar a proposta grava `dataPropostaEnviada`; responder NÃO grava `motivoPerda`; limpar zera resposta e motivo; o empty state orienta para o Kanban.
- **Estado atual/bloqueio:** a tela de Oportunidades foi **removida** (redundante com a prospecção); `oportunidades.component.*` e `oportunidade-modal.component.*` não existem mais, e a rota `/dashboard/comercial/oportunidades` redireciona para o Kanban. Os endpoints de `/oportunidades` continuam ativos porque esta aba, o KPI comercial e o valor no card dependem de `tboportunidade`. A exclusão de lead apaga as oportunidades em cascata. `tbproduto` é somente leitura (produtos nascem no legado).
- **Rotas/endpoints/tabelas:** `/dashboard/comercial`, `/dashboard/comercial/propostas`; `GET /api/v1/dashboard/comercial/produtos`, `GET/POST/PUT/DELETE /api/v1/dashboard/comercial/oportunidades`, `PATCH /oportunidades/{id}/proposta` (+ `motivoPerda`); `tboportunidade`, `tbproduto`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.

### DBN-04 — Consultar a base de clientes e os grupos econômicos

- **Objetivo:** conferir a carteira de clientes com total de licenças e grupo econômico, tudo em leitura somente.
- **Pré-condições:** operador autenticado (sem exigência de papel `Administrador`); legadas `tbcliente`, `tbcliente_grupo` e `tbcontrato` acessíveis.
- **Passos:** 1. Abrir `/dashboard/administrativo`. 2. Conferir os **três** KPIs do topo (clientes ativos, contratos ativos, licenças em contratos ativos). 3. Na barra de filtros, escolher "Filtrar por" (Todos/Nome do cliente/CNPJ/Grupo econômico), digitar o termo e pressionar Enter. 4. Aplicar os selects de Situação, Grupo e Ordenar por. 5. Observar o contador "N registro(s) · X licença(s)" no cabeçalho da tabela. 6. Clicar em "Limpar" para restaurar a lista completa. 7. Conferir que um cliente sem contrato aparece na lista exibindo `0` licença.
- **Resultado esperado:** tudo em modo leitura (as legadas nunca são alteradas por esta tela); a busca livre cobre fantasia, razão social, CNPJ e nome do grupo, e o seletor restringe o campo; a ordenação por licenças usa o total somado; a tabela é única (`# | Cliente | CNPJ | Situação | Licenças | Grupo`).
- **Estado atual/bloqueio:** a **tabela de contratos foi removida** desta tela e, junto com ela, o alerta "N reajuste(s) em até 30 dias" — o componente deixou de chamar `GET /administrativo/contratos` (saíram o signal `contratos`, o computed `reajustesCriticos` e `classeReajuste`); o endpoint continua existindo. Também foram removidos os cards "Faturamento mensal" e "Reajustes em 90 dias". Valores do legado aparecem sem tratamento: `ATIVO` em `tbcliente` é `S`/`N` (ausente é tratado como ativo) e `SITUACAO` em `tbcontrato` só assume `NULL`, `Entregue` ou `Pendente`. Quando o alerta voltar, o filtro **deve** testar `diasParaReajuste !== null` — ver [`09-TROUBLESHOOTING.md`](../../operacao/troubleshooting.md).
- **Rotas/endpoints/tabelas:** `/dashboard/administrativo`; `GET /api/v1/dashboard/administrativo/clientes` (filtros `busca`, `campo`, `situacao`, `grupoId`, `ordenacao`), `GET /administrativo/grupos`, `GET /administrativo/resumo`; `tbcliente`, `tbcliente_grupo`, `tbcontrato`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.

### DBN-05 — Operar o módulo Financeiro

- **Objetivo:** Acompanhar custos, orçamentos e a evolução mensal.
- **Pré-condições:** perfil `Administrador` (`adminGuard` no frontend e role no backend).
- **Passos:** 1. Abrir `/dashboard/financeiro` (perfil não administrador é redirecionado para `/`). 2. Conferir os **dois** KPIs do topo (orçamentos aprovados pendentes, reajustes em 90 dias) e o gráfico Chart.js de custos × orçamentos aprovados. 3. Cadastrar um custo (tipo, competência, valor, pagamento). 4. Cadastrar e transicionar um orçamento (Rascunho → Enviado → Aprovado → Faturado).
- **Resultado esperado:** tipo de custo e competência `AAAA-MM` são validados no backend; aprovar grava `DataAprovacao`, sair de `Aprovado` limpa e `Faturado` grava a data atual; exclusões pedem confirmação; avisos usam o toast do módulo.
- **Estado atual/bloqueio:** limitação conhecida: o gráfico é renderizado no cliente com retry agendado; a rota não é prerenderizada. **Faturamento e atraso foram desligados por decisão do usuário** (`FlagsDashboard.FaturamentoLiberado = false`): saíram os KPIs "Faturamento previsto (mês)", "Faturamento atrasado", "Custo do mês" e "Resultado do mês", o card combinado virou só "Reajustes em 90 dias" e a tabela "Faturamentos em atraso" deixou de existir (o signal `atrasos` e a chamada `listarAtrasos()` saíram do componente; `financeiro.service.ts` manteve o método, sem consumidor). `GET /financeiro/atrasos` continua existindo e volta a responder ao religar a constante. Custos, Orçamentos e o gráfico **permanecem** porque vêm de `tbcusto`/`tborcamento`, tabelas do próprio sistema (ambas vazias hoje). A coluna "Faturamento" dos Orçamentos permanece porque é `tborcamento.dataFaturamento` (dado nosso), e não a projeção do legado.
- **Rotas/endpoints/tabelas:** `/dashboard/financeiro`; `GET /api/v1/dashboard/financeiro/kpis`, CRUD em `/custos` e `/orcamentos`, todos com `Administrador`; `tbcusto`, `tborcamento`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.

### DBN-06 — Registrar e consultar o histórico de contatos do lead (prospecção)

- **Objetivo:** Manter o rastro de prospecção por lead com tipos fixos (Envio de mensagem, Mensagem respondida, Sem resposta, Ligação realizada, E-mail enviado, Reunião agendada, Outro) e exibir badge visual "aguardando resposta" quando o último contato foi um envio sem resposta posterior.
- **Pré-condições:** operador autenticado; lead existente em qualquer etapa do funil.
- **Passos:** 1. Abrir `/dashboard/comercial`. 2. No card do lead, clicar no ícone "Registrar contato" (balão de conversa). 3. No modal, informar o **nome do contato** (obrigatório), escolher o tipo, ajustar a data/hora e escrever a observação (opcional). 4. Salvar. 5. Observar o badge "aguardando resposta" aparecer no card quando o tipo for "Envio de mensagem". 6. Abrir o modal de edição do lead e conferir a timeline de contatos (nome, tipo colorido, data, observação, botão excluir). 7. Registrar "Mensagem respondida" e ver o badge sumir. 8. Excluir um contato e ver a timeline atualizar.
- **Resultado esperado:** o registro atualiza `LEAD_UltimoContato` do lead; o badge âmbar "aguardando resposta" aparece exclusivamente quando o último tipo for "Envio de mensagem"; a timeline no modal do lead lista os contatos do mais recente para o mais antigo; excluir remove da timeline e recompõe `UltimoContato` com a maior data restante; o histórico **não move** o lead entre etapas (a progressão continua pelo drag & drop).
- **Estado atual/bloqueio:** limitação conhecida: não há automação de follow-up baseada em tempo decorrido (ex.: alerta após X dias sem resposta). A exclusão do último contato mantém o `UltimoContato` atual do lead (não zera). O badge usa a cor âmbar (`#f59e0b`) no card e no modal.
- **Rotas/endpoints/tabelas:** `/dashboard/comercial`; `GET/POST /api/v1/dashboard/comercial/leads/{id}/contatos`, `DELETE /api/v1/dashboard/comercial/contatos/{id}`; `tbleadcontato`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.

### DBN-07 — Marcar manualmente que o cliente viu o contato

- **Objetivo:** registrar, pelo responsável, que o cliente visualizou o contato do lead, sem depender de evento automático.
- **Pré-condições:** operador autenticado; lead existente; coluna `tblead.LEAD_ContatoVisualizado` presente no banco (migration `20260928182047_LeadContatoVisualizado`).
- **Passos:** 1. Abrir `/dashboard/comercial`. 2. No card do lead, clicar no botão "Não viu" (ícone de olho). 3. Conferir a troca imediata para "Viu" (verde). 4. Abrir o modal de edição do lead e alternar o checkbox "Cliente visualizou o contato", salvar e reabrir o modal. 5. Recarregar a página e confirmar que o estado persistiu. 6. Clicar novamente para desmarcar e confirmar a volta para "Não viu" (cinza).
- **Resultado esperado:** o toggle aplica o novo valor imediatamente (update otimista) e reverte com toast se a API falhar; o clique não abre o modal do card; o estado sobrevive ao recarregamento porque é persistido em `tblead`.
- **Estado atual/bloqueio:** o flag é **exclusivamente manual** — nenhum código o alterna automaticamente. Ele é **independente** do badge "aguardando resposta" (derivado do último tipo de contato), e os dois podem mostrar estados contraditórios sem que nada os sincronize.
- **Rotas/endpoints/tabelas:** `/dashboard/comercial`; `PATCH /api/v1/dashboard/comercial/leads/{id}/contato-visualizado` (toggle do card) e `PUT /leads/{id}` com `contatoVisualizado` (checkbox do modal); `tblead.LEAD_ContatoVisualizado`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.


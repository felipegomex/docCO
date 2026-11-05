---
id: 05-3-implantacao.01
titulo: "IM-01 — Consultar o Dashboard de Implantação (+4)"
tipo: referencia
area: qa
sistema: central
tags: [qa, implantacao, historias]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app/features/implantacao
  - backend/Central_BackEnd/Controllers/Implantacao
testes:
  - docs/
tokens_estimados: 2134
ficha: docs/modulos/kanban.md
---
> Parte de [3. Implantação](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

### IM-01 — Consultar o Dashboard de Implantação

- **Objetivo:** Visualizar indicadores, prazos, alertas e produtividade da operação de implantação.
- **Pré-condições:** operador autenticado; projetos e tarefas disponíveis para o filtro selecionado.
- **Passos:** 1. Abrir `/implantacao/dashboard`. 2. Selecionar `Todos os projetos` ou um projeto. 3. Ler KPIs e gráficos. 4. Alterar o período de Produtividade e abrir um prazo/tarefa.
- **Resultado esperado:** o filtro recarrega o dashboard; KPIs, gráficos, próximos prazos e alertas refletem os dados do backend; a seção de Produtividade calcula taxa de conclusão, tempo médio e SLA no cliente a partir das tarefas, e a lista de **atrasadas** usa a decisão do backend (`t.atrasado` — o resumo da tarefa), com o número de dias calculado na tela. Os cards do Dashboard devem levar às listas correspondentes já filtradas. A tela não escreve dados.
- **Estado atual/bloqueio:** limitação conhecida: os cards usam `queryParams` como `status` e `tipo`, mas `ProjetosComponent` e `TarefasComponent` não leem esses parâmetros; os atalhos podem abrir a lista sem o filtro anunciado.
- **Rotas/endpoints/tabelas:** `/implantacao/dashboard`; `GET /api/v1/implantacao/dashboard?projetoId=N`, `GET /api/v1/implantacao/projetos` e `GET /api/v1/implantacao/tarefas?projetoId=N`, todos `[Authorize]`; `tbprojeto`, `tbtarefa`, `tbtarefaapontamento` e `tbtarefareponsavel`.
- **Status/automação:** **cenário documentado**; não há teste do dashboard.
### IM-02 — Listar, filtrar e agrupar Projetos

- **Objetivo:** Localizar projetos e entender seu progresso, responsável, prazo e etapa atual.
- **Pré-condições:** operador autenticado; a listagem com etapas pode ser carregada.
- **Passos:** 1. Abrir `/implantacao/projetos`. 2. Aplicar busca, status, responsável e período. 3. Escolher ordenação e abas Todos/Ativos/Concluídos/Atrasados. 4. Recolher filtros/resumo e abrir um card.
- **Resultado esperado:** o layout é compacto em três colunas; a lista é vertical, agrupada e cada card apresenta as nove etapas. Busca/status/responsável são enviados ao endpoint; período e ordenação são aplicados no cliente. A aba "Atrasados" e a ordenação "Atrasados Primeiro" usam a **decisão do backend** (`projeto.atrasado` — prazo do projeto vencido e não concluído/cancelado), a mesma do KPI "projetos atrasados" do Dashboard. Estados vazios por grupo são honestos.
- **Estado atual/bloqueio:** limitação conhecida: o backend aceita `clienteId`, mas a tela de Projetos não oferece nem consome esse filtro; a regra `ClienteObrigatorio` do tipo de Projeto também não é aplicada uniformemente ao formulário e ao serviço de criação.
- **Rotas/endpoints/tabelas:** `/implantacao/projetos`; `GET /api/v1/implantacao/projetos/com-etapas` e `GET /api/v1/agenda/operadores`, autenticados; `tbprojeto`, `tbprojetoetapa`, `tbtipoprojeto` e `tboperador`.
- **Status/automação:** **cenário documentado**; não há teste automatizado da listagem.
### IM-03 — Criar ou editar Projeto

- **Objetivo:** Cadastrar ou alterar um Projeto com listas de apoio reais e identificar a etapa inicial.
- **Pré-condições:** operador autenticado; tipos, clientes, operadores, etapas e próximo código disponíveis.
- **Passos:** 1. Abrir `/implantacao/projetos/novo` ou `/implantacao/projetos/:id/editar`. 2. Informar Nome, Tipo e Responsável; preencher os campos opcionais. 3. Escolher a etapa inicial na criação quando aplicável. 4. Salvar ou cancelar.
- **Resultado esperado:** o formulário impede salvar sem os três campos obrigatórios; o código é sugerido na criação e preservado na edição; a criação inicializa as nove etapas e a edição retorna ao detalhe. O formulário não expõe um controle de status.
- **Estado atual/bloqueio:** limitação conhecida: `TipoProjeto.ClienteObrigatorio` é persistido, mas o formulário considera obrigatórios apenas Nome, Tipo e Responsável, e `ProjetoService.CriarAsync` aceita `ClienteId` nulo; a obrigatoriedade de cliente precisa ser corrigida e validada no servidor.
- **Rotas/endpoints/tabelas:** `/implantacao/projetos/novo`, `/implantacao/projetos/:id/editar`; consultas de apoio: `GET /api/v1/implantacao/tipos-projeto?apenasAtivos=true`, `GET /api/v1/implantacao/projetos/clientes`, `GET /api/v1/agenda/operadores`, `GET /api/v1/implantacao/projetos/etapas-padrao` e `GET /api/v1/implantacao/projetos/proximo-codigo`; `POST /api/v1/implantacao/projetos` e `PUT /api/v1/implantacao/projetos/{id}`; `tbprojeto`, `tbprojetoetapa`, `tbtipoprojeto`, `tbcliente` e `tboperador`.
- **Status/automação:** **cenário documentado**; não há teste automatizado dos formulários.
### IM-04 — Acompanhar o detalhe e as etapas do Projeto

- **Objetivo:** Consultar os dados do Projeto, as nove etapas e as tarefas vinculadas.
- **Pré-condições:** operador autenticado; Projeto existente.
- **Passos:** 1. Abrir `/implantacao/projetos/:id`. 2. Ler a visão geral e as nove etapas. 3. Abrir uma etapa e navegar no checklist/histórico/comentários. 4. Alternar para Tarefas e usar Nova tarefa/Editar. 5. Usar Editar ou Excluir no hero.
- **Resultado esperado:** hero, progresso e KPIs são carregados; as etapas têm estado, percentual, tarefas/checklist e atraso; reabertura de tarefa reabre a etapa afetada e coloca as posteriores concluídas em Bloqueada (cascata, com histórico), e a conclusão automática só ocorre com a etapa anterior concluída. A aba Histórico do Projeto continua sendo um placeholder “Em breve”. O modal deve permitir checklist, comentários e documentos; retorno de etapa deve ser confirmado; a exclusão pede confirmação e retorna à listagem.
- **Estado atual/bloqueio:** limitação conhecida: o modal trata upload como `URL.createObjectURL` e persiste esse URL temporário, sem storage real; a exclusão de documento está em TODO. O retorno de etapa e parte das operações do modal capturam/ignoram erro, portanto podem falhar sem uma indicação confiável.
- **Rotas/endpoints/tabelas:** `/implantacao/projetos/:id`; `GET /api/v1/implantacao/projetos/{id}`, `GET /api/v1/implantacao/projetos/{id}/etapas`, `GET /api/v1/implantacao/tarefas?projetoId={id}` e `DELETE /api/v1/implantacao/projetos/{id}`; no modal, `GET /api/v1/implantacao/projetos/{id}/etapas/{ordem}`, `PUT /api/v1/implantacao/projetos/{id}/etapas/{ordem}`, `POST /api/v1/implantacao/projetos/{id}/etapas/{ordem}/checklist`, `POST /api/v1/implantacao/projetos/{id}/etapas/{ordem}/documentos` e `POST /api/v1/implantacao/projetos/{id}/etapas/{ordem}/comentarios`; `tbprojeto`, `tbprojetoetapa`, `tbprojetoetapachecklist`, `tbprojetoetapacomentario` e `tbtarefa`. Todos os endpoints do controller exigem `[Authorize]`.
- **Status/automação:** **cenário documentado**; não há teste automatizado do detalhe ou do modal de etapa.
### IM-05 — Filtrar e administrar Tarefas

- **Objetivo:** Localizar tarefas por atalhos, projeto e texto, e executar ações de cartão.
- **Pré-condições:** operador autenticado; lista de projetos e tarefas disponível.
- **Passos:** 1. Abrir `/implantacao/tarefas`. 2. Selecionar Todas, Atrasadas, Em andamento, Concluídas, Bugs ou Features. 3. Pesquisar, filtrar por projeto e marcar `Mostrar arquivadas`. 4. Editar, arquivar/desarquivar ou excluir uma tarefa.
- **Resultado esperado:** a lista usa `GET` com os filtros correspondentes; apenas tarefas concluídas podem ser arquivadas; exclusão pede confirmação e remove o item da lista. A opção “Sem projeto” deve retornar somente tarefas com `projetoId` nulo.
- **Estado atual/bloqueio:** limitação conhecida: ao escolher “Sem projeto”, o frontend converte o valor para `undefined` e não envia `projetoId`; o backend só filtra quando o parâmetro tem valor, portanto a opção não restringe a lista. Os atalhos internos são aplicados localmente, mas não são todos inicializados por query string.
- **Rotas/endpoints/tabelas:** `/implantacao/tarefas`; `GET /api/v1/implantacao/tarefas`, `GET /api/v1/implantacao/projetos`, `PATCH /api/v1/implantacao/tarefas/{id}/arquivar`, `PATCH /api/v1/implantacao/tarefas/{id}/desarquivar` e `DELETE /api/v1/implantacao/tarefas/{id}`; `tbtarefa`, `tbprojeto`, `tbcolunakanban` e `tbauditoriaimplantacao` para auditoria.
- **Status/automação:** **cenário documentado**; não há teste automatizado da lista.

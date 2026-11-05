---
id: sec.10-5-empresa-agenda
titulo: "5. Empresa/Agenda"
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
tokens_estimados: 2092
ficha: docs/modulos/agenda.md
---
> Ficha: [Agenda](../../modulos/agenda.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 5. Empresa/Agenda

### EA-01 — Abrir Empresa e o Onboarding

- **Objetivo:** Levar o colaborador da área Empresa ao início do Onboarding Corporativo.
- **Pré-condições:** operador autenticado.
- **Passos:** 1. Abrir `/empresa`. 2. Confirmar o redirecionamento para `/empresa/onboarding`. 3. Ler a apresentação e os quinze capítulos. 4. Clicar em `Iniciar Onboarding` ou em um capítulo.
- **Resultado esperado:** `/empresa` não tem tela própria; a Home de Onboarding mostra os quinze cards e leva ao primeiro capítulo. O conteúdo é local e não há persistência de progresso.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; o redirecionamento e a navegação são locais.
- **Rotas/endpoints/tabelas:** `/empresa` → `/empresa/onboarding`; nenhum endpoint/tabela; `onboarding-data.ts`.
- **Status/automação:** **cenário documentado**; não há teste automatizado do Onboarding.

### EA-02 — Navegar por um capítulo de Onboarding

- **Objetivo:** Ler os blocos de um capítulo e avançar ou voltar na jornada.
- **Pré-condições:** operador autenticado; ID numérico presente no conteúdo estático.
- **Passos:** 1. Abrir `/empresa/onboarding/capitulo/:id`. 2. Expandir os blocos e acionar os acordeões. 3. Clicar em `Voltar` ou `Próximo`. 4. Abrir um ID inválido.
- **Resultado esperado:** o capítulo, número e progresso são exibidos; a navegação sequencial funciona e o último capítulo retorna à Home. ID inválido redireciona para `/empresa/onboarding`.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; o progresso é calculado localmente e não representa registro de leitura no backend.
- **Rotas/endpoints/tabelas:** `/empresa/onboarding/capitulo/:id`; nenhum endpoint/tabela; `CAPITULOS` em `onboarding-data.ts`.
- **Status/automação:** **cenário documentado**; não há teste automatizado do capítulo.

### EA-03 — Navegar e filtrar a Agenda

- **Objetivo:** Visualizar a agenda em Dia, Semana, Mês ou Janela Operacional de 48 horas e restringir eventos por operador/função.
- **Pré-condições:** operador autenticado; listas de tipos, operadores e funções carregadas.
- **Passos:** 1. Abrir `/agenda`. 2. Alternar as quatro visões. 3. Usar anterior, `Hoje`, próximo e o seletor de data. 4. Alternar `Meus`/`Geral`, escolher Responsável/Função e limpar os filtros.
- **Resultado esperado:** a grade renderiza mesmo vazia; o período sem eventos mostra a faixa na grade; `Geral` limpa filtros de operador/função antes de recarregar; a visão inicial é Semana/Meus. O filtro de função usa a tabela legada de funcionários, não o campo de função de `tboperador`. A prioridade é visual nos cards: **crítica** com borda 6px + ícone de perigo (CSS `var(--color-error)`), **alta** com borda 5px âmbar + alerta, **normal** com borda 2px translúcida neutra e **baixa** sem borda — em todas as visões (Dia, Semana, Mês e miniatura), sem a cor do tipo ser lavada pela da prioridade.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; a listagem depende da fonte de funcionários legada e dos dados de Agenda retornados pelo backend.
- **Rotas/endpoints/tabelas:** `/agenda`; `GET /api/v1/agenda/eventos?inicio=X&fim=Y&responsavelId=Z&funcaoId=N`, `GET /api/v1/agenda/tipos`, `GET /api/v1/agenda/operadores`, `GET /api/v1/agenda/funcoes`, todos `[Authorize]`; `tbagenda`, `tbtipoevento`, `tbagendaparticipante`, `tboperador`, `tbfuncao`, `tbfuncionario` e `tbprojeto`.
- **Status/automação:** **cenário documentado**; não há teste automatizado da Agenda.

### EA-04 — Criar, editar e repetir um evento

- **Objetivo:** Registrar um evento com tipo, responsável, prioridade e, quando permitido, recorrência.
- **Pré-condições:** operador autenticado; tipo e responsável ativo disponíveis.
- **Passos:** 1. Clicar em `Novo evento` ou em uma célula. 2. Preencher título, descrição/local, datas/horas, tipo, responsável e prioridade. 3. Selecionar `Repetir até` e padrão quando o tipo permitir. 4. Salvar ou cancelar. 5. Editar e excluir um evento próprio. 6. Ao editar um evento recorrente, conferir que o modal **pré-preenche** `Repetir até` com o fim da série e o padrão; mudar o fim e salvar reorganiza as datas do lote (estreita, alarga e preenche buracos na âncora do primeiro evento, sem alterar conteúdo dos irmãos e sem remover a instância editada); deixar `Repetir até` vazio e salvar não mexe na estrutura.
- **Resultado esperado:** título, data inicial, tipo e responsável são obrigatórios; fim deve ser posterior ao início; Férias exige data de retorno e dia inteiro; Treinamento/Daily aceitam lote; Reunião/Atendimento são de um dia. O contrato de edição deve preservar participantes e SLA enviados, e a exclusão usa confirmação. No `PUT`, `dataRepeticaoFim`/`padraoRecorrencia` (presentes só quando o usuário informa fim) disparam a reconciliação da série; `dataRepeticaoFim` anterior ao início retorna `400`; o `409` de conflito é validado apenas nas datas a serem criadas e **antes** de gravar. A recorrência na edição foi corrigida em 2026-10-05 (commit `0ba68be`): antes, `dataRepeticaoFim` era ignorado pelo backend e nada mudava no lote.
- **Estado atual/bloqueio:** limitação conhecida: o modal não expõe participantes nem SLA e envia `participantesIds: undefined`; o `PUT` limpa a coleção de participantes e pode gravar SLA nulo. Projeto também não é controlado pelo modal atual, apesar de existir no contrato.
- **Rotas/endpoints/tabelas:** `/agenda` + modal; `GET /api/v1/agenda/eventos/{id}`, `POST /api/v1/agenda/eventos`, `POST /api/v1/agenda/eventos/lote`, `PUT /api/v1/agenda/eventos/{id}` (com `dataRepeticaoFim?`/`padraoRecorrencia?` novos), `DELETE /api/v1/agenda/eventos/{id}`; `tbagenda` (coluna nova `AGD_SerieId`), `tbagendaparticipante`, `tbtipoevento`, `tbprojeto` e `tboperador`.
- **Status/automação:** **cenário documentado**; não há teste automatizado do modal.

### EA-05 — Respeitar dono, administrador e conflitos

- **Objetivo:** Impedir alteração de evento de terceiro e informar sobreposição de horários.
- **Pré-condições:** eventos de teste com responsáveis distintos; usuário proprietário, usuário sem propriedade e Administrador; agenda com conflito.
- **Passos:** 1. Visualizar um evento de terceiro em cada visão. 2. Tentar abrir/editar pelo card. 3. Chamar `PUT`, `DELETE` ou `PATCH` direto para evento de terceiro. 4. Criar/atualizar/mover um evento que sobreponha outro. 5. Conferir a tabela afetada.
- **Resultado esperado:** cards de terceiros ficam somente leitura; a API de alteração/movimentação retorna 403 para quem não é proprietário; a criação não exige que o usuário atual seja o responsável e pode informar outro responsável ativo; a sobreposição retorna 409 com `code: CONFLICT_HORARIOS` e a lista de conflitos — **desde 2026-10-05 o `409` dispara de fato para eventos de qualquer duração**: `FimEfetivo` soma a hora ao início em horário de Brasília e só então compara em UTC (antes, a soma em UTC deixava ~3h de janela sem cobertura e o conflito era silenciosamente ignorado). A verificação cobre apenas o responsável — não os participantes enviados (limitação conhecida). A listagem lê `tbagenda`; a criação/escrita grava `tbagenda` e participantes quando enviados; `tbtipoevento`, `tboperador` e `tbfuncao` não são escritas pela Agenda. Não há arrastar-e-soltar de evento na interface atual; `PATCH /mover` é um contrato de API.
- **Estado atual/bloqueio:** limitações conhecidas: a query de conflito filtra apenas `Agenda.OperadorId`, não participantes; a edição atual pode apagar participantes e SLA, portanto a cobertura de conflitos não é uniforme. Também não há arrastar-e-soltar de evento na interface atual.
- **Rotas/endpoints/tabelas:** `/agenda`; `PUT/DELETE /api/v1/agenda/eventos/{id}` e `PATCH /api/v1/agenda/eventos/{id}/mover`, autenticados; `tbagenda`, `tbagendaparticipante`, `tbprojeto`, `tbtipoevento`, `tboperador` e `tbfuncao`.
- **Status/automação:** **cenário documentado**; não há teste de autorização/conflito.


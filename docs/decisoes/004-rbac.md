---
titulo: "ADR 004 - RBAC: permissao por recurso e por funcao, com heranca de grupo e excecao individual"
status: "aceito"
data: "2026-10-21"
---

# ADR 004 - RBAC: permissao por recurso e por funcao, com heranca de grupo e excecao individual

## Contexto

A autorizacao do sistema hoje e **implicita e espalhada**: nao existe policy, nem tabela de
permissao, nem matriz central fora do codigo. `Program.cs:136` registra `AddAuthorization()` vazio.
A decisao esta em **23 pontos** — 8 controllers repetindo a mesma pergunta "este e admin?" (tres
claims cada, em `TarefasController.cs:27`, `IdeiasController.cs:64`, `GestoresController.cs:49`,
`AuditoriaController.cs:39`, `IdeiasSeedController.cs:47`, `AgendaController.cs:302`,
`ConhecimentoController.cs:198` e os de WhatsApp) — mais `FuncaoAuthorizationService`,
`GestorFuncaoService` e a matriz de Ideias (`IdeiasPermissaoService.cs:133-204`).

Seis eixos de decisao coexistem: perfil global (`PERFIL_ID` A/S/F/P + `SE_ADMIN`), funcao do
operador (`tboperador.FUNCAO_ID`), funcao do recurso (`TRF_FuncaoId` / `PRJ_FuncaoId`), gestor
(`tbgestor`), papel sobre a solicitacao (criador/gestor de Ideias) e prova de senha (ticket do
modulo Acessos). O cliente espelha parte disso: **9 copias** de "e admin?" no frontend, cada uma
com criterio diferente — `kanban.component.ts:1288` aceita `perfilId`, `sidebar.component.ts:165` nao
aceita.

Consequencias observaveis do estado atual (verificadas em 2026-10-21):

- Das **30 rotas** da sidebar, **2** sao restritas por papel (`header-nav.config.ts`: "Financeiro" e
  "Números do WhatsApp", ambos `roles: ['Administrador']`). O filtro de menu reconhece 2 dos 9 tokens do
  tipo `NavRole` (`sidebar.component.ts:108-112`), de modo que `roles: ['A']` nao filtra nada.
- **15 escritas** nao checam dono, funcao ou papel — entre elas resolver, reabrir e transferir
  atendimento alheio no WhatsApp (`AtendimentoService.cs:317`, `:362`, `:215`) e a flag
  `Permissoes` hardcoded em `true` (`AtendimentoService.cs:937`).
- O **Database Explorer** (metadados, query builder e **geracao de script DDL**) responde so
  `[Authorize]`: qualquer autenticado navega no schema do banco de producao
  (`Controllers/Database/DatabaseController.cs:14`, `:234-403`). Os dashboards administrativo e
  comercial (`AdministrativoController.cs:17`, `ComercialController.cs:17`) tambem, enquanto o
  financeiro e admin-only (`FinanceiroController.cs:18`).
- A **leitura** e aberta: qualquer autenticado ve o funil comercial, a carteira de clientes e o
  diretorio de operadores. O escopo "proprio / da equipe / tudo" **nao existe** como conceito.
- O **acesso negado nao e auditado** em nenhum lugar: `KanbanBoardService.cs:126` faz so
  `LogWarning`; a trilha `tbauditoriaimplantacao` nao tem IP, funcao nem perfil. Sao 12 lacunas.
- "Gestor geral" tem **dois sentidos opostos**: `IdeiasPermissaoService.cs:289` (todas as
  funcoes) e `GestorFuncaoService.cs:83` (so a propria).

## Decisao

Adotar **RBAC** para decidir **o que** o operador pode fazer, mantendo os eixos atuais para
decidir **onde** (a linha). As duas portas precisam passar.

1. **Recurso** — chave estavel com 4 tipos: `aba`, `api`, `relatorio`, `tabela`. O catalogo e
   **semeado pelo extrator que ja existe** (`scripts/extract-screens.ts` -> `scripts/screens-data.json`,
   que hoje extrai 61 rotas, 190 endpoints, 81 componentes e 38 tabelas): nada de catalogo escrito a
   mao, e nenhuma rota nova passa sem recurso.
2. **Grupo** — heranca. Comeca como **projecao do estado atual**: quatro grupos semeados de
   `PERFIL_ID` (A -> Administrador, S -> Suporte, F -> Financeiro, P -> Produtor) mais um
   "Visualizador", com `tbgrupo_operador` populado do perfil de cada operador. A evolucao para um
   catalogo de grupos proprio e negocio, e fica para uma fase posterior.
3. **Acao** — `Ler`, `Criar`, `Editar`, `Excluir`, `Aprovar`/`Publicar`, em bitset.
4. **Escopo** — `proprio` / `funcao` / `tudo`, gravado na permissao e **valendo na leitura** (e o
   que hoje nao existe).
5. **Excecao individual** — por operador, sobre (recurso, funcao): pode conceder **e** revogar; a
   revogacao vence o grupo.
6. **Unico ponto de decisao** — um `PermissaoService` no backend, exposto como policy do ASP.NET
   (`[Authorize(Policy = "perm:...")]`), e um `GET /api/v1/permissoes/minhas` que alimenta o menu.
   As decisoes de linha (funcao do recurso, gestor, dono) **continuam** onde estao.
7. **Menor privilegio** — quem nao tem grupo nao tem nada; o grupo do perfil e o **piso**, nao o
   teto. Ampliar e excecao individual.
8. **Auditoria** — tabela nova de acesso (permitido **e** negado: recurso, funcao, acao, usuario,
   IP) e tabela nova de alteracao de permissao (quem concedeu/revogou o que, quando).
9. **Aplicacao incremental** — primeiro os recursos que hoje estao abertos a qualquer autenticado; a
   migracao dos 190 endpoints vem depois, com o motor ja maduro. As decisoes em codigo viram dados
   no fim (a matriz de Ideias inclusive).
10. **O motor absorve** a Fase 2 do plano `qualidade-logica` (o "PolicyService" de permissao por
    funcao): nao havera dois motores de autorizacao no sistema.

## Consequencias

- O **primeiro passo nao muda comportamento**: modelo, carga e motor vivem ao lado das decisoes
  atuais, e um **teste de paridade** compara, para cada (grupo, recurso, funcao), a decisao antiga e
  a nova. Se divergirem, o teste falha. Carga errada deixa de ser um risco silencioso.
- A **Fase 0 exige migration em producao sem staging** (ver [ADR 003](./003-sem-staging.md)):
  backup obrigatorio, `RESTORE VERIFYONLY` e script idempotente aplicado **antes** do publish.
- **Uma divergencia atual passa a ser visivel**: Suporte e Financeiro sao superusuarios do Kanban
  (editam qualquer funcao) mas nao veem `/dashboard/financeiro`, que e admin-only; e o "gestor" (4
  gestores, um por funcao) gerencia tarefas e agenda, mas nao projetos, etapas nem bloqueios. A carga
  inicial **registra** o comportamento atual; mudar essas duas coisas e decisao de negocio, a ser
  tomada na tela de gestao, com o antes/depois visivel.
- O **catalogo de recursos passa a ser parte do contrato**: rota nova sem recurso correspondente
  precisa ser tratada explicitamente (o extrator gera o recurso; o grupo sem permissao o ve como
  negado — que e o menor privilegio, e o comportamento desejado).
- O **modulo Acessos permanece com prova de senha e sem papel** (`AcessosController.cs:29-31`):
  "Autorizacao por papel continua FORA de proposito".
- A `tboperador.SENHA` (material equivalente a senha em claro) **nao e tratada por este ADR**: e um
  projeto proprio, registrado em `docs/arquitetura/componentes.md`.

## Fonte

- Levantamento do backend e do frontend de 2026-10-21 (decisoes Q1-Q7 registradas pelo dono do
  projeto em 2026-10-21).
- `backend/Central_BackEnd/Program.cs:136` — `AddAuthorization()` vazio.
- `backend/Central_BackEnd/Services/Implantacao/FuncaoAuthorizationService.cs:44-49` —
  `PodeEditarFuncao` (decisao de linha).
- `backend/Central_BackEnd/Services/Implantacao/GestorFuncaoService.cs:76-93` — regra do gestor.
- `backend/Central_BackEnd/Services/Ideias/IdeiasPermissaoService.cs:133-204` — a unica matriz
  declarativa, hoje em codigo.
- `backend/Central_BackEnd/Controllers/Database/DatabaseController.cs:14`, `:234-403` — leitura e
  geracao de DDL liberadas a qualquer autenticado.
- `backend/Central_BackEnd/Services/WhatsApp/AtendimentoService.cs` — escritas sem checagem de
  responsavel: `AssumirAsync`, `TransferirAsync`, `ResolverAsync` e o `AtendimentoPermissoes`
  hardcoded em `true` dentro de `MontarDetalheAsync`. (As referencias por linha usadas na versao
  original desta ADR ficaram defasadas: o arquivo cresceu; os metodos acima sao a referencia estavel.)
- `backend/Central_BackEnd/Services/Implantacao/KanbanBoardService.cs:126` — acesso negado sem
  trilha.
- `frontend/src/app/shared/config/header-nav.config.ts` e
  `frontend/src/app/layout/sidebar/sidebar.component.ts:108-112` — 30 rotas, 2 restritas; filtro
  reconhece 2 dos 9 tokens.
- `scripts/extract-screens.ts` / `scripts/screens-data.json` — a fonte do catalogo de recursos.
- Plano de implantação: `docs/planos/rbac-fechamento.md`.

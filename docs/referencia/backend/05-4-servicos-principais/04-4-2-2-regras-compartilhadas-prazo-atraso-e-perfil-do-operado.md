---
id: 05-4-servicos-principais.04
titulo: "4.2.2 Regras compartilhadas — prazo/atraso e perfil do operador"
tipo: referencia
area: backend
sistema: central
tags: [backend, services, di, efcore]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Configuration
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1035
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../../modulos/auth-rbac-sistema.md).

### 4.2.2 Regras compartilhadas — prazo/atraso e perfil do operador

Duas classes **estáticas** em [`Services/Shared`](../../../../backend/Central_BackEnd/Services/Shared) concentram decisões que antes eram tomadas em cada serviço (e, no frontend, em cada tela). Não entram no DI: são chamadas diretas.

| Classe | Métodos | Decide |
|---|---|---|
| `RegrasPrazo` | `PrazoTarefa`, `AtrasadaTarefa` (entidade e valores soltos), `VenceHojeTarefa`, `DiasAtrasoTarefa`, `PrazoProjeto`, `AtrasadaProjeto` (entidade e valores soltos), `VenceHojeProjeto`, `DiasAtrasoProjeto`, `DiasAtrasoEtapa` | se um item está **atrasado** e há quantos dias |
| `PerfilOperador` | `Normalizar`, `EhAdmin`, `Rotulo` | como o `PERFIL_ID` é lido e como o perfil é rotulado |

**Regra de prazo (dias corridos — a comparação é sempre por `.Date`):**

| Objeto | Prazo | Atrasado quando |
|---|---|---|
| Tarefa | `DataEntrega ?? DataPrevisao` | o prazo passou, **exceto** tarefa concluída dentro do prazo; concluída depois do prazo conta; **cancelada nunca conta** |
| Projeto | `DataPrevisao` | o prazo passou e o projeto **não** está concluído nem cancelado (projeto concluído deixa de contar) |
| Etapa | `DataFimPrevista` | atraso em dias = (referência − `DataFimPrevista`), com referência = `DataFimReal` ou hoje; etapa sem prazo previsto não tem atraso |

A **semântica escolhida para projeto** é *prazo vencido* — a mesma que o KPI "projetos atrasados" do Dashboard já usava. Antes, a aba "Atrasados" da tela de Projetos contava **etapa** atrasada, e os dois números discordavam.

**Quem consome a regra:**

- `TarefaService` — filtro "vence hoje" e `TarefaResumo.atrasado`;
- `DashboardService` — KPI de projetos atrasados e de tarefas atrasadas (`OndeAtrasadas`);
- `ProjetoEtapaService` — cálculo do `AtrasoDias` (na edição manual **e** na sincronização por tarefas);
- `ProjetoService` — `ProjetoResumo.atrasado`;
- `KanbanBoardService` — `CardKanban.atrasado` e `CardKanban.dataConclusao`.

**Filtro no banco:** listar atrasadas é uma consulta, então a regra também existe como **expressão** em [`TarefaAtrasoExtensions.OndeAtrasadas`](../../../../backend/Central_BackEnd/Services/Implantacao/TarefaAtrasoExtensions.cs) — a mesma regra escrita para o EF traduzir. O teste de contrato [`RegrasPrazoTests`](../../../../backend/Central_BackEnd.Tests/RegrasPrazoTests.cs) amarra as duas ponta a ponta, para a lista do banco não voltar a divergir do KPI.

**Decisão exposta no contrato:** a decisão de "atrasado" é do **backend** — `ProjetoResumo.atrasado`, `TarefaResumo.atrasado` e `CardKanban.atrasado` são a decisão pronta, e o **cliente consome** o campo (só o helper `ehAtrasada` recalcula, e apenas como **reserva**, quando o endpoint não traz `atrasado`). `CardKanban` passou a trazer também `dataConclusao`, sem a qual o cliente não consegue avaliar a exceção "concluída dentro do prazo".

**`PerfilOperador`:** `PERFIL_ID` é uma letra (`A`/`S`/`F`/`P`) em `tboperador`, coluna `varchar(1)` com **cadastro manual** — nada impede `"a"` minúsculo ou `" A "` com espaço. A normalização é `Trim().ToUpperInvariant()`; `EhAdmin` = `SE_ADMIN` ou `PERFIL_ID` normalizado igual a `"A"`; `Rotulo` devolve `Administrador`/`Suporte`/`Financeiro`/`Produtor`/`Usuario` (o `F` é o Financeiro — `FUNCAO_ID` 4 — e o rótulo **não move nenhum direito**). Consumidores: `AuthService` (claim `perfil_id` do token e rótulo do `UsuarioResponse` — o campo `UsuarioResponse.PerfilId` continua sendo o valor **cru** do banco; o normalizado é a claim do token), `FuncaoAuthorizationService` (ponte de admin do Kanban Geral) e `TarefaService.PodeAtribuirTerceiroAsync` (atribuição a terceiros por Master/gestor da função). Antes, o Kanban normalizava e os apontamentos comparavam a letra crua: o mesmo operador era admin num lugar e não-admin no outro.


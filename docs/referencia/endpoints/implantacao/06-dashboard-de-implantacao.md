---
id: implantacao.06
titulo: "Dashboard de Implantação"
tipo: referencia
area: api
sistema: central
tags: [api, implantacao, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers/Implantacao
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 274
ficha: docs/modulos/kanban.md
---
> Parte de [Endpoints — Controllers/Implantacao](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

## Dashboard de Implantação

A classe abaixo é `DashboardController`, declarada no mesmo arquivo de tarefas.

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `GET` | `/api/v1/implantacao/dashboard` | JWT + `PodeVerFuncao` | — | Query `equipe?`, `projetoId?`, `funcaoId?` | `200 DashboardGeral` com KPIs, agrupamentos e prazos **filtrados pelo setor** (`funcaoId`; sem ele, usa a função do operador ou a primeira função gerida). `403` quando o setor não é a função do operador nem uma função que ele gerencia, ou quando o operador não tem setor e não é Master. `equipe` é aceito, mas não altera o cálculo. |

Fonte: [TarefasController.cs](../../../../backend/Central_BackEnd/Controllers/Implantacao/TarefasController.cs).


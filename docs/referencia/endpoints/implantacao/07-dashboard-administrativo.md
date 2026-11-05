---
id: implantacao.07
titulo: "Dashboard administrativo"
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
tokens_estimados: 200
ficha: docs/modulos/kanban.md
---
> Parte de [Endpoints — Controllers/Implantacao](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

## Dashboard administrativo

Fonte: [AdminDashboardController.cs](../../../../backend/Central_BackEnd/Controllers/Implantacao/AdminDashboardController.cs).

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `GET` | `/api/v1/admin/dashboard` | JWT + Administrador | — | Query `funcaoId?`, `diasRetro=7` | `200 AdminDashboardDto` com totais, funções, responsáveis e alertas; `diasRetro` é aceito, mas não participa dos cálculos. |


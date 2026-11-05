---
id: raiz.14
titulo: "CargosController (`/cargos`)"
tipo: referencia
area: api
sistema: central
tags: [api, endpoints, rotas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 184
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../../modulos/auth-rbac-sistema.md).

## CargosController (`/cargos`)

Fonte: [CargosController.cs](../../../../backend/Central_BackEnd/Controllers/CargosController.cs). **1 operação** versionada, `[Authorize]`:

| Método | Rota | Entrada | Resposta resumida |
|---|---|---|---|
| `GET` | `/api/v1/cargos` | Sem corpo | `200 List<CargoResumo>` (`id`, `codigo`, `descricao`, `escopoPadrao`, `ativo`) com os cargos **ativos**, ordenados por `CARGO_ID`. |


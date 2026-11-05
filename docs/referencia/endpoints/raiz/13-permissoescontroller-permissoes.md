---
id: raiz.13
titulo: "PermissoesController (`/permissoes`)"
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
tokens_estimados: 566
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../../modulos/auth-rbac-sistema.md).

## PermissoesController (`/permissoes`)

Fonte: [PermissoesController.cs](../../../../backend/Central_BackEnd/Controllers/PermissoesController.cs) e [PermissaoService.cs](../../../../backend/Central_BackEnd/Services/Permissoes/PermissaoService.cs). **5 operações** versionadas, `[Authorize]` na classe; não há policy `perm:*` aqui — a matriz usa `PodeGerenciarCargoAsync`.

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/api/v1/permissoes/minhas` | JWT | Sem corpo | `200 PermissoesMinhasDto`: `operadorId`, `cargoId`, `cargoCodigo`, `funcaoId` e `permissoes[]` com `recurso`, `rota`, `tipo`, `acoes` (bitset), `escopo` e `funcaoId`. É a fonte do menu e do guard do frontend; o backend revalida. |
| `GET` | `/api/v1/permissoes/recursos` | JWT + **`GESTOR_MASTER`** | Sem corpo | `200` com a lista de recursos e `recurso`, `tipo`, `rota`, `acoes`; **`403`** para quem não é Master. |
| `GET` | `/api/v1/permissoes/cargos-gerenciaveis` | JWT | Sem corpo | `200 List<CargoResumo>` (`id`, `codigo`, `descricao`, `escopoPadrao`, `ativo`) com os cargos **ativos** cuja matriz o operador pode abrir/editar: Master vê todos, exceto `GESTOR_MASTER`; Tech Lead vê só os do seu setor e de nível inferior. Sem cargo elegível, devolve lista vazia. |
| `GET` | `/api/v1/permissoes/matriz/{cargoId}` | JWT + `PodeGerenciarCargoAsync` | Path | `200 MatrizCargoDto` (`cargoId`, `codigo`, `descricao`, `permissoes[]` com `recurso`, `rota`, `tipo`, `acoes`, `escopo`); `403` quando o operador não gerencia o cargo e `404` para cargo inexistente. |
| `PUT` | `/api/v1/permissoes/matriz/{cargoId}` | JWT + `PodeGerenciarCargoAsync` | `MatrizSalvarRequest` (`permissoes[]`) | `204` sem corpo; `400` para ação fora do bitset (0–127), escopo fora de 1–3 ou recurso inexistente; `403` fora do escopo. Substitui **somente** as linhas gerais (`PEC_FuncaoId IS NULL`) do cargo — as linhas por função são preservadas. |


---
id: implantacao.01
titulo: "Tipos de projeto"
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
tokens_estimados: 380
ficha: docs/modulos/kanban.md
---
> Parte de [Endpoints — Controllers/Implantacao](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

## Tipos de projeto

Fonte: [TiposProjetoController.cs](../../../../backend/Central_BackEnd/Controllers/Implantacao/TiposProjetoController.cs).

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `GET` | `/api/v1/implantacao/tipos-projeto` | JWT | — | Query `apenasAtivos=true` | `200 List<TipoProjetoResumo>`. |
| `GET` | `/api/v1/implantacao/tipos-projeto/{id}` | JWT | — | Path `id`; sem body | `200 TipoProjetoDetalhe` efetivo ou `404`. A assinatura do controller declara `ActionResult<TipoProjetoResumo>`, mas o serviço retorna detalhe. |
| `POST` | `/api/v1/implantacao/tipos-projeto` | JWT + Administrador | — | Body `TipoProjetoCriarRequest` | `201` com o detalhe retornado pelo serviço; `400` para argumento inválido. |
| `PUT` | `/api/v1/implantacao/tipos-projeto/{id}` | JWT + Administrador | — | Body `TipoProjetoAtualizarRequest` | `200` com o detalhe retornado pelo serviço ou `404`. |
| `DELETE` | `/api/v1/implantacao/tipos-projeto/{id}` | JWT + Administrador | — | Path `id`; sem body | `204` ou `404`; se houver projeto vinculado, o `InvalidOperationException` do serviço não é tratado pelo controller e pode resultar em `500`. |


---
id: raiz.12
titulo: "OperadoresController (`/operadores`)"
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
tokens_estimados: 687
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../../modulos/auth-rbac-sistema.md).

## OperadoresController (`/operadores`)

Fonte: [OperadoresController.cs](../../../../backend/Central_BackEnd/Controllers/OperadoresController.cs) e [OperadorAdminService.cs](../../../../backend/Central_BackEnd/Services/Shared/OperadorAdminService.cs). **6 operações** em `api/v{version:apiVersion}/operadores`, `[Authorize]` na classe. As escritas usam a policy `perm:aba.sistema.operadores:<acao>`; as leituras exigem apenas JWT.

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/api/v1/operadores` | JWT | Query `busca?`, `funcaoId?`, `apenasAtivos` (default `true`), `page` (default 1), `pageSize` (default 50) | `200 PaginadoOperadorResumo` (`items`, `totalCount`, `page`, `pageSize`, `totalPages`); cada item traz `cargoId`/`cargoCodigo`/`cargoDescricao`. |
| `GET` | `/api/v1/operadores/{operadorId}` | JWT | Path | `200 OperadorDetalhe` (inclui `cargoId`, `cargoCodigo`, `cargoDescricao`) ou `404`. |
| `POST` | `/api/v1/operadores` | `perm:aba.sistema.operadores:Criar` | `OperadorCriarRequest`: `operadorId`, `nome?`, `senha`, `email?`, **`cargoId`**, `funcaoId?` | `201 OperadorDetalhe`; `400` para ID/senha/e-mail/cargo/setor inválidos, `409` para `OPERADOR_DUPLICADO`/`EMAIL_DUPLICADO` e `403` quando o chamador não é Master/Tech Lead ou tenta criar acima do próprio nível. Criar Master liga `SE_ADMIN`/`PERFIL_ID = 'A'`. |
| `PUT` | `/api/v1/operadores/{operadorId}` | `perm:aba.sistema.operadores:Editar` | `OperadorAtualizarRequest`: `nome?`, `email?`, `cargoId`, `funcaoId?` | `200 OperadorDetalhe`; `400`/`409`/`403`. Recusa alterar o próprio cargo (`403`), recusa Tech Lead alterando Master/Tech Lead e, ao promover a Master, liga `SE_ADMIN`/`PERFIL_ID`; rebaixar não limpa. |
| `PATCH` | `/api/v1/operadores/{operadorId}/status` | `perm:aba.sistema.operadores:Editar` | `OperadorStatusRequest`: `seAtivo` (`S`/`N`) | `200 OperadorDetalhe`; `400` se não for `S`/`N`; `403` para autoinativação ou escopo inválido. **Não há `DELETE`**: a saída é inativar. |
| `PATCH` | `/api/v1/operadores/{operadorId}/senha` | `perm:aba.sistema.operadores:Editar` | `OperadorSenhaRequest`: `senha` | `200 OperadorDetalhe`; `400` se vazia ou com **mais de 10 caracteres**; `403` fora do escopo. A auditoria `SENHA` não registra o valor. |

Regras completas de precedência, Master/Tech Lead, níveis, e-mail e senha em [`referencia/regras-negocio.md`](../../regras-negocio/INDEX.md).


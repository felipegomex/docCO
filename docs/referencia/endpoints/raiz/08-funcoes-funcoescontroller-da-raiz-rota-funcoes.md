---
id: raiz.08
titulo: "Funções (`FuncoesController` da raiz, rota `/funcoes`)"
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
tokens_estimados: 420
ficha: docs/modulos/README.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

## Funções (`FuncoesController` da raiz, rota `/funcoes`)

Fonte: [FuncoesController.cs](../../../../backend/Central_BackEnd/Controllers/FuncoesController.cs). Duas rotas vivas com a mesma palavra existem hoje, com controllers e serviços diferentes: esta (`/api/v1/funcoes`, pasta raiz) e a [`/api/v1/implantacao/funcoes` da pasta Implantacao](../implantacao/INDEX.md). O fato está descrito como está; não há registro de rota antiga ou de substituição.

| Método | Rota | Autorização | Entrada | Resposta |
|---|---|---|---|---|
| `GET` | `/api/v1/funcoes` | JWT (`[Authorize]` na classe) | Query `apenasAtivos=true` | `200 List<FuncaoResumo>`, via `IFuncoesService.ListarAsync`. |
| `GET` | `/api/v1/funcoes/{id}` | JWT | Path `id` | `200 FuncaoDetalhe` (inclui etapas) ou `404`. |
| `GET` | `/api/v1/funcoes/{id}/etapas` | JWT | Path `id` | `200 List<EtapaFuncaoResumo>`. |

Consumidor confirmado na varredura de 2026-10-18: o serviço de frontend [`shared/domain/services/funcoes.service.ts`](../../../../frontend/src/app/shared/domain/services/funcoes.service.ts) monta a base `/funcoes` e é consumido por [`gestores.component.ts`](../../../../frontend/src/app/features/gestores/pages/gestores/gestores.component.ts) (`funcoesSvc.listar(true)`, tela de gestores). As duas operações `GET /{id}` e `GET /{id}/etapas` não apareceram com consumidor nessa varredura.


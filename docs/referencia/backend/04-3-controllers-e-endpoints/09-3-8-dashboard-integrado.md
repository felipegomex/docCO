---
id: 04-3-controllers-e-endpoints.09
titulo: "3.8 Dashboard Integrado"
tipo: referencia
area: backend
sistema: central
tags: [backend, controllers, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 669
ficha: docs/modulos/dashboard.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Dashboard Integrado](../../../modulos/dashboard.md).

### 3.8 Dashboard Integrado

Três controllers com `[ApiVersion("1.0")]` e rota `api/v{version:apiVersion}/dashboard/...`. Os contratos estão em [`DashboardDtos.cs`](../../../../backend/Central_BackEnd/Dtos/Dashboard/DashboardDtos.cs). O catálogo detalhado de rotas fica em [05-ENDPOINTS.md](../../endpoints.md).

| Controller | Arquivo canônico | Rotas | Autorização | Escopo |
|---|---|---|---|---|
| `ComercialController` | `Controllers/ComercialController.cs` | `/api/v1/dashboard/comercial/leads*` (+ `/arquivar`, `/desarquivar`), `/produtos`, `/contatos*`, `/oportunidades*`, `/kpi` (20 operações) | JWT | Funil comercial (`tblead`, `tboportunidade`, `tbproduto` leitura) |
| `AdministrativoController` | `Controllers/Dashboard/AdministrativoController.cs` | `/api/v1/dashboard/administrativo/{clientes,grupos,contratos,resumo}` (4 operações) | JWT | Leitura das legadas `tbcliente`/`tbcliente_grupo`/`tbcontrato` + KPIs |
| `FinanceiroController` | `Controllers/Dashboard/FinanceiroController.cs` | `/api/v1/dashboard/financeiro/{kpis,atrasos,custos*,orcamentos*}` (10 operações) | JWT + `Administrador` | Custos e orçamentos (`tbcusto`, `tborcamento`) |

> **Colisão de rota resolvida.** Existiam **dois** controllers com a mesma rota `api/v{version:apiVersion}/dashboard/comercial`: `Controllers/ComercialController.cs` e `Controllers/Dashboard/ComercialController.cs`. Toda rota declarada nos dois resolução como ambígua no `ActionSelector` e devolvia HTTP 500 (`AmbiguousMatchException`) — `GET /leads`, `GET /leads/{id}`, `POST/PUT/PATCH/DELETE /leads`, todas as `/oportunidades` e `GET /kpi`. O duplicado foi **apagado**; o controller canônico é `Controllers/ComercialController.cs`, que trata erro com `IHostEnvironment` (devolve `detalhe` da exceção em Development) e aplica `EnableRateLimiting` **por endpoint**: `"leitura"` nos `GET` e `"validacao"` nas escritas. Antes de criar um controller sob `Controllers/Dashboard`, confira se a rota já não existe em `Controllers/`.

Regras aplicadas nos serviços: `ArgumentException` vira `400 { mensagem }`; mover lead para `Negócio Fechado` apenas grava `UltimoContato` (a automação de contrato está adiada); excluir lead apaga as oportunidades em cascata; enviar proposta move a oportunidade para a etapa `Proposta`; transições de status do orçamento regem `DataAprovacao`/`DataFaturamento`.


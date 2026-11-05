---
id: 05-4-servicos-principais.03
titulo: "4.2.1 Kanban Geral (controllers e DI)"
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
tokens_estimados: 460
ficha: docs/modulos/kanban.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

### 4.2.1 Kanban Geral (controllers e DI)

Registros em [`Program.cs`](../../../../backend/Central_BackEnd/Program.cs), todos `Scoped`, com dependência cruzada explícita:

```csharp
builder.Services.AddScoped<IFuncaoAuthorizationService, FuncaoAuthorizationService>();
builder.Services.AddScoped<IBloqueioService, BloqueioService>();
builder.Services.AddScoped<IFuncaoService, FuncaoService>();
builder.Services.AddScoped<IEtapaFuncaoService, EtapaFuncaoService>();
builder.Services.AddScoped<IKanbanBoardService, KanbanBoardService>();
```

| Controller | Rota-base | Operações | Autorização |
|---|---|---:|---|
| `FuncoesController` | `api/v{version}/implantacao/funcoes` | 1 | `[Authorize]`; **somente leitura**, sem escrita |
| `EtapasFuncaoController` | `api/v{version}/implantacao/etapas-funcao` | 6 | `[Authorize]`; escrita com `Roles = "Administrador"` |
| `BloqueiosController` | `api/v{version}/implantacao/bloqueios` | 3 | `[Authorize]` + regra de função no controller |
| `GargalosController` (mesmo arquivo) | `api/v{version}/implantacao/gargalos` | 1 | `[Authorize]`, leitura aberta |
| `KanbanController` | `api/v{version}/implantacao/kanban` | 2 | `[Authorize]`; `403` na regra de função |

`ProjetosController` ganhou `PATCH /projetos/{id}/coluna`, que chama o **mesmo** `IKanbanBoardService.MoverCardAsync("projeto", …)` — a permissão fica resolvida em um único lugar. Detalhamento de rotas em [`05-ENDPOINTS.md`](../../endpoints.md) e regras em [`03-REGRAS-NEGOCIO.md`](../../regras-negocio/INDEX.md).


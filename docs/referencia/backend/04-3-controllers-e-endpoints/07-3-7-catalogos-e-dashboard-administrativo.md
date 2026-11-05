---
id: 04-3-controllers-e-endpoints.07
titulo: "3.7 Catálogos e dashboard administrativo"
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
tokens_estimados: 517
ficha: docs/modulos/dashboard.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Dashboard Integrado](../../../modulos/dashboard.md).

### 3.7 Catálogos e dashboard administrativo

| Método e caminho | Corpo/parâmetros | Autorização |
|---|---|---|
| `GET /api/v1/implantacao/tipos-projeto` | `apenasAtivos?` | JWT |
| `GET /api/v1/implantacao/tipos-projeto/{id}` | Sem corpo | JWT |
| `POST /api/v1/implantacao/tipos-projeto` | `TipoProjetoCriarRequest` | Admin |
| `PUT /api/v1/implantacao/tipos-projeto/{id}` | `TipoProjetoAtualizarRequest` | Admin |
| `DELETE /api/v1/implantacao/tipos-projeto/{id}` | Sem corpo | Admin |
| `GET /api/v1/implantacao/colunas-kanban` | `apenasAtivas?`, `funcaoId?`, `todas?` | JWT |
| `GET /api/v1/implantacao/colunas-kanban/visao/{funcaoId}` | Sem corpo | JWT + **gestor da função ou administrador** (403 caso contrário) |
| `PUT /api/v1/implantacao/colunas-kanban/visao/{funcaoId}` | `ColunaKanbanVisaoSalvarRequest` | JWT + **gestor da função ou administrador** |
| `POST /api/v1/implantacao/colunas-kanban` | `ColunaKanbanCriarRequest` | Admin |
| `PUT /api/v1/implantacao/colunas-kanban/{id}` | `ColunaKanbanAtualizarRequest` | Admin |
| `POST /api/v1/implantacao/colunas-kanban/reordenar` | `ColunaKanbanReordenarRequest` | Admin |
| `GET /api/v1/admin/dashboard` | `funcaoId?`, `diasRetro?` | Admin |

Os contratos de catálogo estão em [`TipoProjetoEtapaColunaDtos.cs`](../../../../backend/Central_BackEnd/Dtos/Implantacao/TipoProjetoEtapaColunaDtos.cs).

> ⚠️ **`DELETE /api/v1/implantacao/colunas-kanban/{id}` foi removido.** Não existe mais exclusão de coluna: a coluna é do **catálogo compartilhado** e o caminho de saída é **desativar** (`CLK_Ativa`), que é **global** — a coluna sai do catálogo inteiro, não só de um setor. O `PUT` recusa desativar coluna que tenha cards (`"Não é possível desativar: existem cards nesta coluna."`).


---
id: 06-5-modelos-dbcontext-e-relacionamentos.05
titulo: "5.4 Modelos sem DbSet"
tipo: referencia
area: backend
sistema: central
tags: [backend, modelos, dbcontext, efcore]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 190
ficha: docs/modulos/database-explorer.md
---
> Parte de [5. Modelos, DbContext e relacionamentos](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### 5.4 Modelos sem DbSet

Requests/responses de Chat e Conhecimento são contratos HTTP, não entidades persistidas. As cinco entidades locais do módulo — `ChatWorkspaceFuncao`, `ChatAuditoria`, `ConhecimentoRevisorFuncao`, `ConhecimentoCredencialGitea` e `ConhecimentoAuditoria` — têm `DbSet` e correspondem às tabelas criadas por `20261021003310_ChatConhecimento`. `AcessoBase` é abstrata, mas **é** entidade: as 5 derivadas têm `DbSet`.


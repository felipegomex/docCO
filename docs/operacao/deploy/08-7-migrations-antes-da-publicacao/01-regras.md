---
id: 08-7-migrations-antes-da-publicacao.01
titulo: "Regras"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, migration, iis, schema]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts/deploy/deploy.ps1
  - scripts/preflight-schema.ps1
  - backend/Central_BackEnd/Migrations
testes:
  - docs/
tokens_estimados: 180
ficha: docs/modulos/database-explorer.md
---
> Parte de [7. Migrations antes da publicação](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### Regras

- `deploy.ps1` não aplica migrations.
- O backend não executa `Database.Migrate()` no startup.
- InMemory não valida migrations SQL.
- A design-time factory aponta para LocalDB; um `dotnet ef database update` sem conexão explícita pode atingir o banco errado.
- O agregado `Migrations/Sql/AllMigrations.sql` era obsoleto e foi **apagado**; gere e revise SQL a partir do commit atual.


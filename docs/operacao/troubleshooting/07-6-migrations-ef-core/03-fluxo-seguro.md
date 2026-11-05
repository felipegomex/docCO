---
id: 07-6-migrations-ef-core.03
titulo: "Fluxo seguro"
tipo: operacao
area: diagnostico
sistema: central
tags: [migration, efcore, sqlserver, erro]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Migrations
  - scripts
testes:
  - docs/
tokens_estimados: 241
ficha: docs/modulos/database-explorer.md
---
> Parte de [6. Migrations EF Core](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### Fluxo seguro

```powershell
Set-Location .\backend\Central_BackEnd
dotnet ef migrations script --idempotent --output "$env:TEMP\central-operacao-migrations.sql"
Pop-Location
```

Antes de executar o arquivo:

1. FaÃ§a backup do banco.
2. Leia o SQL gerado; confirme inÃ­cio e fim, histÃ³rico, tabelas e rollback operacional.
3. Execute com a ferramenta e a autenticaÃ§Ã£o aprovadas para o banco alvo.
4. Confirme o resultado:

```sql
SELECT MigrationId
FROM __EFMigrationsHistory
ORDER BY MigrationId;
```

5. Teste as telas e APIs afetadas.
6. Guarde o script, o hash do commit e a evidÃªncia de execuÃ§Ã£o conforme a polÃ­tica do DBA.


---
id: 10-migrations-e-tabelas-historicas.02
titulo: "Introdução — parte 2"
tipo: referencia
area: dados
sistema: central
tags: [schema, migrations, historico]
status: ativo
revisado_em: "2026-11-05"
fonte:
  - backend/Central_BackEnd/Migrations
testes:
  - docs/
tokens_estimados: 436
ficha: docs/modulos/database-explorer.md
---
> Parte de [Migrations e tabelas históricas](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### Introdução — parte 2

| Migration | Mudança estrutural confirmada |
|---|---|
| `20261021003310_ChatConhecimento` | Cria `tbchat_workspace_funcao`, `tbchat_auditoria`, `tbconhecimento_revisor_funcao`, `tbconhecimento_credencial_gitea` e `tbconhecimento_auditoria`, com índices e FKs declaradas no modelo. Script idempotente equivalente em [`Sql/ChatConhecimento_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/ChatConhecimento_Idempotente.sql); migration e script aplicados em HML e PROD em 2026-10-30. |
| `20261029213303_ChatProvedorConfig` | **Só cria/derruba `tbchat_provedor`** (provedor de IA configurável, chave cifrada) e atualiza o snapshot; nenhuma tabela existente muda. Script idempotente equivalente em [`Sql/ChatProvedor_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/ChatProvedor_Idempotente.sql), que cria a tabela e registra a migration em `__EFMigrationsHistory`. **Aplicada em HML e PROD em 2026-10-30** (implementada em código/testes). `scripts/aplicar-chat-sql.ps1` aplica os 4 SQLs e exige as 4 migrations de chat. |

⚠️ **O RBAC não tem classe de migração.** As quatro tabelas (`tbcargo`, `tbpermissao_recurso`, `tbperm_cargo`, `tbperm_operador`) e `tboperador.CARGO_ID` foram criadas por [`Sql/Permissoes_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/Permissoes_Idempotente.sql), com carga, mapa, seed de operadores e matriz nos demais scripts `Permissoes_*.sql`. O `AppDbContextModelSnapshot.cs` **não** contém as entidades do RBAC — a contagem de 36 classes de migração não muda, e um `dotnet ef migrations add` continuaria produzindo um `Up()` alheio ao schema real (ver a seção do snapshot). `deploy.ps1` **não** aplica migration: o preflight de schema é quem bloqueia o publish quando `CARGO_ID`, as quatro tabelas ou as colunas `tbcargo.CARGO_Nivel`/`CARGO_FuncaoId` faltam (ele confere existência, **não** o tipo da coluna).
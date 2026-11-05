---
id: 08-7-migracoes-e-persistencia.03
titulo: "7.2 Execução e seeds"
tipo: referencia
area: backend
sistema: central
tags: [backend, migrations, persistencia]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Migrations
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 343
ficha: docs/modulos/database-explorer.md
---
> Parte de [7. Migrações e persistência](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### 7.2 Execução e seeds

- Não há aplicação automática de migrações nem `EnsureCreated()` em `Program.cs`; o source não cria o schema InMemory durante o startup.
- Em Development, o seed estrutural cria operadores apenas se `Operadores` estiver vazio, funções (incluindo a 5/CIAA), o mapeamento `tbchat_workspace_funcao` (funções 1, 2, 3 e 5), sete colunas, quatro tipos de Projeto e sete tipos de evento; valores de credenciais não são reproduzidos aqui.
- Seeds de Projetos/Tarefas e cinco eventos de Agenda estão dentro de `#if false`.
- Não há seed de Etapas Globais; os nove cards são por Projeto.
- [`Migrations/Sql`](../../../../backend/Central_BackEnd/Migrations/Sql) contém **34** scripts idempotentes auxiliares (o agregado `AllMigrations.sql` era obsoleto e foi **apagado** — gere SQL a partir do commit atual); eles são artefatos adicionais ao histórico EF. Os três do RBAC (`Permissoes_Idempotente.sql`, `Permissoes_CargaInicial_Idempotente.sql`, `Permissoes_SeedOperadores_Idempotente.sql`) não têm classe de migração correspondente.


---
id: 08-7-migrations-antes-da-publicacao.06
titulo: "`20260929034944_KanbanGeral_SetoresEtapasBloqueios` em detalhe"
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
tokens_estimados: 384
ficha: docs/modulos/database-explorer.md
---
> Parte de [7. Migrations antes da publicação](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### `20260929034944_KanbanGeral_SetoresEtapasBloqueios` em detalhe

O detalhamento por etapa do script (1 a 12) está em [`04-ESTRUTURA-DADOS.md`](../../../arquitetura/banco-dados.md). O que importa no deploy:

- **antes**: rode o backup do banco; **depois**: confira `__EFMigrationsHistory`, as 3 tabelas, o seed de 5 setores, `CLK_Ativa = 0` na coluna "BLOQUEADO" e a ausência de `TRF_Bloqueada`.
- Se aparecer `Invalid column name 'TRF_SetorId'`, o script **não** foi aplicado (o `deploy.ps1` não aplica nada): rode-o manualmente.
- ⚠️ **Não use `dotnet ef database update` nem o `Down` desta migration para o Kanban Geral.** O `Up` da classe é vazio de propósito e o `Down` apenas derruba as tabelas novas (perde os bloqueios, sem recriar `TRF_Bloqueada`/`TRF_MotivoBloqueio`). Além disso, o snapshot do EF declara a tabela de colunas como `tbcolunakanban` (o modelo e o banco usam `tbcolunakanban`) — detalhes em [`04-ESTRUTURA-DADOS.md`](../../../arquitetura/banco-dados.md). A reversão é manual e por script, com backup.
- O que é **só cadastro** e não tem migration: `tbfuncao.FUNCAO_ID = 4` ("Financeiro"), em `Migrations/Sql/FuncaoFinanceiro_Idempotente.sql`. Não crie migration para isso; rode o script.


---
id: sec.13-migrations-carimbadas-manualmente-e-armadilhas-de-historico
titulo: "Migrations carimbadas manualmente e armadilhas de histórico"
tipo: referencia
area: dados
sistema: central
tags: [schema, banco, tabelas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 607
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Migrations carimbadas manualmente e armadilhas de histórico

Rodar `dotnet ef database update` num ambiente já tratado por scripts manuais pode falhar ou tentar reexecutar trabalho. Registrar o que foi carimbado à mão:

| Migration | Situação |
|---|---|
| `20260914144751_AgendaConflitoHorarios` | Já aplicada fisicamente via script manual; inserida manualmente em `__EFMigrationsHistory`. |
| `20260917202045_TarefaProjetoOpcional` | Idem. |
| `20260920185734_TarefaProjetoEtapaId` | Idem. |
| `20260922154202_RemoveEtapaAntiga` | Aplicada normalmente (sem carimbo manual). |

**Armadilha `AgendaGeral`:** o banco tem o registro `20260910183240_AgendaGeral` em `__EFMigrationsHistory`, mas **não existe** o arquivo `.cs` correspondente no repositório (roll-back da Agenda). O EF ignora linhas desconhecidas do histórico, então não há erro — mas comparar ambientes pela contagem/ordem do histórico pode confundir. O snapshot atual é a fonte autoritativa do modelo.

Os scripts auxiliares usados nesse processo ficam em `scripts/db/` e em `Migrations/Sql/`. Hoje existe **um único** script de produção: [renomear-tabelas-tb-idempotente.sql](../../../scripts/db/renomear-tabelas-tb-idempotente.sql), gerado a partir das duas migrations já corrigidas. Ele é idempotente e cobre o conjunto inteiro — tabelas, PKs, índices e a unique constraint — equivalendo a `PadraoTabelasTb` + `RenomeiaIndicesPkTb` (necessário porque `scripts/deploy/deploy.ps1` **não** aplica migrations). Os 33 `sp_rename` de índice usam o formato correto (`sp_rename N'<tabela>.IX_…', N'IX_…', N'INDEX'`), sem a forma defeituosa `sp_rename N'dbo.IX_…'`, e o script carimba 11 migrations em `__EFMigrationsHistory`. O antigo `scripts/db/migracao-tarefa-projeto-etapa-id.sql` era **obsoleto — não executar** (substituído pelo script único) e foi **apagado** do repositório. Para auditar, [conferencia-padrao-tb.sql](../../../scripts/db/conferencia-padrao-tb.sql) aceita nomes antigos e novos: rode antes e depois e compare as seções 1 e 2 (contagens não podem mudar).


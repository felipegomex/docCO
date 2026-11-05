---
id: 10-migrations-e-tabelas-historicas.06
titulo: "`20260929034944_KanbanGeral_SetoresEtapasBloqueios` — o que o script faz"
tipo: referencia
area: dados
sistema: central
tags: [schema, migrations, historico]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Migrations
testes:
  - docs/
tokens_estimados: 1157
ficha: docs/modulos/database-explorer.md
---
> Parte de [Migrations e tabelas históricas](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### `20260929034944_KanbanGeral_SetoresEtapasBloqueios` — o que o script faz

| Etapa do script | Efeito |
|---|---|
| 1–2 | `CREATE TABLE tbsetor` e `tbetapsetor` com nomes únicos **filtrados por ativo** e FK `Cascade` de etapa para setor |
| 3 | `CREATE TABLE tbbloqueio` com `UX_tbbloqueio_Origem_Aberto` (índice único filtrado em `BLQ_ResolvidoEm IS NULL`) |
| 4 | Colunas de setor em `tboperador`/`tbprojeto`/`tbtarefa` + `tbtarefa.TRF_EtapaTemplateId`, todas com FK `SET NULL` |
| 5 | Colunas de quadro: `CLK_SetorId`, `CLK_StatusCard`, `CLK_SincronizaAgenda` (`-1` = nunca sincronizar) e `CLK_Ordem` → `decimal(5,2)` (`CLK_Bloqueada` foi removida em 2026-10-14 e nunca é recriada) |
| 6–7 | Seed dos **5 setores** (1 `Suporte`, 2 `Implantacao`, 3 `Financeiro`, 4 `RH`, 5 `Comercial`) e das etapas padrão (ciclo de implantação para o setor 2; fluxo genérico de 4 etapas para os demais) |
| 8–9 | Mapeia `CLK_StatusCard` das colunas padrão por nome (só `CLK_Padrao = 1`, nunca coluna customizada de setor) e faz o backfill `TRF_SetorId`/`PRJ_SetorId` = setor Implantação |
| 10 | Migra `TRF_Bloqueada = 1` para linhas de `tbbloqueio`, devolve os cards bloqueados à coluna do estágio e desativa a coluna "BLOQUEADO" |
| 11 | **Drop** de `TRF_Bloqueada` e `TRF_MotivoBloqueio` (por SQL dinâmico, executado por último) |
| 12 | Carimba `20260929034944_KanbanGeral_SetoresEtapasBloqueios` em `__EFMigrationsHistory` |

⚠️ **O script tem `DROP` e não há staging:** `dbBUSINESS_HML` (`192.168.2.154`) é o banco de produção. Faça backup antes e rode por partes se houver dúvida. Como o SQL Server compila o batch antes de executar, as seções 10 e 11 usam `sp_executesql` para não quebrar quando a coluna ainda não existe. Detalhes de execução em [`10-DEPLOY.md`](../../../operacao/deploy.md).

#### Armadilha "tabela das colunas do Kanban" — nome grafado com as letras trocadas

Fatos verificados nos arquivos (sem inferir intenção):

- O modelo (`Models/Implantacao/ColunaKanban.cs`) declara `[Table("tbcolunakanban")]` em `[Table(...)]`, e é esse o nome contra o qual a aplicação consulta em produção. Vale lembrar que o nome é `tbcoluna` + `kanban` **colado**, sem sublinhado entre as duas palavras. Nomes de índice e de FK já gravados no banco usam um prefixo com as letras trocadas, e isso é anterior a este trabalho.
- **Até `20260929221556_SetorParaFuncao`, o `AppDbContextModelSnapshot.cs` e os `Designer` gravavam o nome com as letras trocadas**, inclusive o da migration do Kanban Geral — ou seja, modelo e snapshot não descreviam a mesma tabela. O snapshot foi **regenerado** por esta mudança e hoje **bate com o modelo**.
- O `Up` de `20260928144824_DashboardTabelas.cs` — migration cujo nome é do Dashboard Integrado e que já está carimbada como aplicada no banco de produção — carrega o DDL do Kanban Geral: `DROP` de `TRF_Bloqueada`/`TRF_MotivoBloqueio`, um `RenameTable` entre as duas grafias, `CREATE TABLE` de `tbbloqueio`/`tbsetor`/`tbetapsetor` e as colunas de setor. A `20260929034944_KanbanGeral_SetoresEtapasBloqueios` tem `Up` **vazio de propósito** (o DDL real está no script revisado) e um `Down` que **derruba** `tbbloqueio`,/`tbetapsetor` e `tbsetor`.

Consequências práticas:

- Como `20260928144824` já está carimbada em `dbBUSINESS_HML`, um `dotnet ef database update` **pula** esse DDL — o banco de produção não é afetado por esse caminho. ✅
- Em um **banco novo** (ou sem o carimbo), `dotnet ef database update` criaria a tabela com a grafia do snapshot, que antes estava trocada. Com o snapshot regenerado, esse caminho voltou a ser coerente. ✅
- O `Down` do Kanban Geral não é o inverso do `Up` do script: reverter por EF **perde os dados de bloqueio** e não recria `TRF_Bloqueada`/`TRF_MotivoBloqueio`.
- A armadilha se repetiu em `20260929221556_SetorParaFuncao`: o scaffold resolveria a divergência com um `RenameTable` indevido, e por isso o `Up` ficou **vazio de propósito**, com o DDL nos dois scripts de `Migrations/Sql/`.
**Orientação:** não rode `dotnet ef database update` para o Kanban Geral nem para a conversão Setor → Função; use os scripts de `Migrations/Sql`, como já orientado em [`10-DEPLOY.md`](../../../operacao/deploy.md). A correção do snapshot/`Up` é decisão de desenvolvimento com o DBA — este documento registra o desvio, não uma correção aplicada.


---
id: 08-7-migrations-antes-da-publicacao.05
titulo: "Migration aplicada: `20260929034944_KanbanGeral_SetoresEtapasBloqueios` (Kanban Geral)"
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
tokens_estimados: 726
ficha: docs/modulos/database-explorer.md
---
> Parte de [7. Migrations antes da publicação](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### Migration aplicada: `20260929034944_KanbanGeral_SetoresEtapasBloqueios` (Kanban Geral)

⚠️ **Já aplicada** em `dbBUSINESS_HML`. Foi a primeira vez que um `DROP` de coluna do módulo de Implantação foi a produção (`TRF_Bloqueada`/`TRF_MotivoBloqueio`). **Supersedida em parte** por `20260929221556_SetorParaFuncao`, que descarta `tbsetor`/`tbetapsetor` e as colunas `*_SetorId` — ver a seção seguinte.

| Item | Valor |
|---|---|
| Alteração | 3 `CREATE TABLE` (`tbsetor`, `tbetapsetor`, `tbbloqueio`), 4 colunas de setor (`tboperador.SetorId`, `tbprojeto.PRJ_SetorId`, `tbtarefa.TRF_SetorId`, `tbcolunakanban.CLK_SetorId`) + `tbtarefa.TRF_EtapaTemplateId`, 2 colunas de quadro (`CLK_StatusCard`, `CLK_SincronizaAgenda` default `-1` — `CLK_Bloqueada` foi removida em 2026-10-14 pela `20261014000000_RemoveClkBloqueada` e o script atual a **remove** onde existir) e `CLK_Ordem` `int` → `decimal(5,2)` |
| Dados | Seed de **5 setores** (1 `Suporte`, 2 `Implantacao`, 3 `Financeiro`, 4 `RH`, 5 `Comercial`) e das etapas padrão; backfill de `TRF_SetorId`/`PRJ_SetorId` = setor 2; migração dos bloqueios abertos de `TRF_Bloqueada` para `tbbloqueio` e devolução dos cards à coluna do estágio; desativação da coluna "BLOQUEADO" |
| `DROP` | `tbtarefa.TRF_Bloqueada`/`TRF_MotivoBloqueio` — **o `Down` da classe EF não os recria** (ele só derruba as tabelas novas), portanto reverter por EF **perde os dados de bloqueio** |
| Tabelas legadas afetadas | **nenhuma** (`tbcliente`, `tbcontrato`, `tbcliente_grupo`, `tbchamado`, `tbfuncionario` não são tocadas) |
| Risco | Médio: aditivo, exceto pelos dois `DROP`; as seções de backfill e de `DROP` usam `sp_executesql` porque o SQL Server compila o batch antes de executar |
| Idempotência | Guarda por `OBJECT_ID`, `COL_LENGTH`, `sys.columns`, `NOT EXISTS` e `__EFMigrationsHistory`; rodar de novo é no-op |
| `ProductVersion` | `8.0.31`, igual às demais migrations |

Confirmação pós-aplicação:

```sql
SELECT MigrationId, ProductVersion FROM __EFMigrationsHistory
WHERE MigrationId = N'20260929034944_KanbanGeral_SetoresEtapasBloqueios';

SELECT name FROM sys.tables WHERE name IN (N'tbsetor', N'tbetapsetor', N'tbbloqueio');

SELECT SET_SetorId, SET_Nome, SET_Ordem, SET_Ativo FROM tbsetor ORDER BY SET_Ordem;

SELECT COUNT(*) AS BloqueiosAbertos FROM tbbloqueio WHERE BLQ_ResolvidoEm IS NULL;

SELECT COL_LENGTH('tbtarefa', 'TRF_Bloqueada') AS DeveSerNULL,
       COL_LENGTH('tbtarefa', 'TRF_SetorId')    AS DeveExistir;

SELECT CLK_Id, CLK_Nome, CLK_Ativa, CLK_StatusCard
FROM tbcolunakanban WHERE UPPER(CLK_Nome) LIKE '%BLOQUE%';
```


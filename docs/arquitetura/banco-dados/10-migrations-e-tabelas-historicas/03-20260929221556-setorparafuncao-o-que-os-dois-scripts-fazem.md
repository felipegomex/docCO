---
id: 10-migrations-e-tabelas-historicas.03
titulo: "`20260929221556_SetorParaFuncao` — o que os dois scripts fazem"
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
tokens_estimados: 455
ficha: docs/modulos/database-explorer.md
---
> Parte de [Migrations e tabelas históricas](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### `20260929221556_SetorParaFuncao` — o que os dois scripts fazem

| Fase | Script | Efeito |
|---:|---|---|
| 0 | `SetorParaFuncao_Idempotente.sql` | Cria `tbfuncaoetapa` (`FUN_*`, FK `Cascade` para `tbfuncao`, nome único filtrado por ativo) e as colunas `tbtarefa.TRF_FuncaoId`, `tbprojeto.PRJ_FuncaoId` e `tbcolunakanban.CLK_FuncaoId`; **backfill pelo mapa de conversão**; índices e FKs `SetNull`; semeia as etapas genéricas nas funções 3 e 4. **Não** carimba `__EFMigrationsHistory` e **não** remove nada. |
| 3 | `SetorParaFuncao_Consolidar_Idempotente.sql` | Aborta (`THROW`) se a Fase 0 não tiver rodado; carimba `20260929221556_SetorParaFuncao` em `__EFMigrationsHistory`; remove as 4 FKs e os 4 índices `*_SetorId`; derruba as colunas `tboperador.SetorId`, `TRF_SetorId`, `PRJ_SetorId` e `CLK_SetorId` (por SQL dinâmico); descarta `tbetapsetor` e depois `tbsetor`. |

**Mapa de conversão usado no backfill:** setor 1 `Suporte` → função 2; setor 2 `Implantacao` → função 1; setor 3 `Financeiro` → função 4; setor 4 `RH` → `NULL`; setor 5 `Comercial` → `NULL`. ⚠️ **RH e Comercial não existem como função**: os cards que estavam nesses dois setores ficam com `FUNCAO_ID NULL` e **não aparecem em nenhum recorte por função** — e, com a pill "Todas" removida do quadro em 2026-10-01, **não aparecem em lugar nenhum** até receberem função. Confira a consulta da seção 7 do script da Fase 0 antes de aplicar e valide com o negócio se há conteúdo desses dois setores que precise de outra função.


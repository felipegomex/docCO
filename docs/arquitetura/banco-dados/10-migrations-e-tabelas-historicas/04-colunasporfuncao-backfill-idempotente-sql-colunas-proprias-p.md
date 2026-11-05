---
id: 10-migrations-e-tabelas-historicas.04
titulo: "`ColunasPorFuncao_Backfill_Idempotente.sql` — colunas próprias por função (spec Seção 3)"
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
tokens_estimados: 740
ficha: docs/modulos/database-explorer.md
---
> Parte de [Migrations e tabelas históricas](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### `ColunasPorFuncao_Backfill_Idempotente.sql` — colunas próprias por função (spec Seção 3)

> ⚠️ **Modelo superado (2026-10-20).** Este script clonou as 7 colunas para cada função e o setor podia **excluir** a sua cópia — o que quebrava a comparação entre setores. Hoje a coluna é do **catálogo compartilhado** e o setor controla apenas a **visão** (`tbcolunafuncao`). O script abaixo **não** deve ser aplicado em banco novo: quem faz isso é [`ColunasCompartilhadas_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/ColunasCompartilhadas_Idempotente.sql), com o procedimento na seção seguinte. Ele fica registrado aqui como histórico do que foi aplicado em `dbBUSINESS_HML` em 2026-10-02.

Script avulso (sem migration EF correspondente) em [Sql/ColunasPorFuncao_Backfill_Idempotente.sql](../../../../backend/Central_BackEnd/Migrations/Sql/ColunasPorFuncao_Backfill_Idempotente.sql), **já aplicado** em `dbBUSINESS_HML` em 2026-10-02:

| Passo | Efeito |
|---|---|
| Pré-condições (`THROW` 50001–50005) | Aborta se `CLK_FuncaoId`, `TRF_FuncaoId`, `PRJ_FuncaoId`, `CLK_StatusCard` ou `CLK_SincronizaAgenda` não existirem (exige a Fase 0 do `SetorParaFuncao` antes) |
| Clone set-based | Copia as colunas globais para **todas as funções ativas** (`CROSS JOIN tbfuncao ATIVO = 1` + `NOT EXISTS (funcaoId, nome)` — idempotente, segunda execução é no-op) |
| Remapeamento de cards | `tbtarefa`/`tbprojeto` com `FuncaoId` passam a apontar para a coluna **da própria função**, casando por nome via `CROSS APPLY` (nunca deixa card órfão) |
| Verificações | 4 conferências: esperado **0** funções ativas sem coluna própria e **0** cards fora do quadro da própria função; grava `UsuarioInclusao = 'backfill-colunasfuncao'` |

**Nunca carimba `__EFMigrationsHistory`.** Ordem obrigatória em qualquer ambiente: **script ANTES do deploy** (código antigo com colunas duplicadas é tolerável por instantes; código novo sem backfill mostra quadros vazios). Aplicação, backup pré e checklist em [`10-DEPLOY.md`](../../../operacao/deploy.md); sintomas em [`09-TROUBLESHOOTING.md`](../../../operacao/troubleshooting.md).

⚠️ **O `Up` vazio não é engano.** O snapshot antigo do EF descrevia a tabela de colunas com um nome divergente do banco (`tbcolunakanban` no snapshot × `tbcolunakanban` no modelo e no banco), então o scaffold desta migration gerou um `RenameTable` indevido: executá-lo renomearia uma tabela já correta para um nome que só existe no snapshot. Pelo mesmo motivo da `20260929034944`, o DDL real fica em scripts revisados à mão. Sintomas e diagnóstico em [`09-TROUBLESHOOTING.md`](../../../operacao/troubleshooting.md).


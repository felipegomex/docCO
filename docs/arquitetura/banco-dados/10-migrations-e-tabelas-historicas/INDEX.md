---
id: 10-migrations-e-tabelas-historicas.indice
titulo: "Migrations e tabelas históricas"
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
tokens_estimados: 300
ficha: docs/modulos/database-explorer.md
---

# Migrations e tabelas históricas

> Índice do documento dividido. Documento original: [10-migrations-e-tabelas-historicas.md](../10-migrations-e-tabelas-historicas.md).
> Ficha: [Database Explorer](../../../modulos/database-explorer.md).

## Partes

- [Introdução](./01-introducao.md)
- [Introdução — parte 2](./02-introducao-parte-2.md)
- [`20260929221556_SetorParaFuncao` — o que os dois scripts fazem](./03-20260929221556-setorparafuncao-o-que-os-dois-scripts-fazem.md)
- [`ColunasPorFuncao_Backfill_Idempotente.sql` — colunas próprias por função (spec Seção 3)](./04-colunasporfuncao-backfill-idempotente-sql-colunas-proprias-p.md)
- [`ColunasCompartilhadas_Idempotente.sql` — catálogo compartilhado e visão por setor](./05-colunascompartilhadas-idempotente-sql-catalogo-compartilhado.md)
- [`20260929034944_KanbanGeral_SetoresEtapasBloqueios` — o que o script faz](./06-20260929034944-kanbangeral-setoresetapasbloqueios-o-que-o-sc.md)
- [`20260928212856_AcessosEmpresas` em detalhe](./07-20260928212856-acessosempresas-em-detalhe.md)
- [`20260926010406_PadraoTabelasTb` em detalhe](./08-20260926010406-padraotabelastb-em-detalhe.md)
- [`20260926022906_RenomeiaIndicesPkTb` em detalhe](./09-20260926022906-renomeiaindicespktb-em-detalhe.md)

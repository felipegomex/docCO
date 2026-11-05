---
id: 10-migrations-e-tabelas-historicas.09
titulo: "`20260926022906_RenomeiaIndicesPkTb` em detalhe"
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
tokens_estimados: 628
ficha: docs/modulos/database-explorer.md
---
> Parte de [Migrations e tabelas históricas](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### `20260926022906_RenomeiaIndicesPkTb` em detalhe

Complemento da anterior: alinha os **nomes** de índice, PK e unique constraint ao padrão `tb*`. Escrita à mão, com dois helpers (`RenameIndex` e `RenameConstraint`) que emitem `sp_rename` guardado por `IF EXISTS`/`IF NOT EXISTS`, portanto idempotentes.

| Grupo | Quantidade | Resultado |
|---|---:|---|
| Índices | 33 | Todos para a convenção do EF `IX_<tabela>_<colunas>`, entre eles `IX_IMPL_Agenda_Operador_DataInicio_DataFim`→`IX_tbagenda_AGD_OperadorId_AGD_DataInicio_AGD_DataFim`, `IX_IMPL_Projeto_PRJ_Codigo`→`IX_tbprojeto_PRJ_Codigo`, `IX_TBOPERADOR_FUNCAO_ID`→`IX_tboperador_FUNCAO_ID` e `IX_RefreshTokens_TokenHash`→`IX_tbrefreshtoken_TokenHash`. |
| PKs | 5 | As que o EF deixou com nome automático (`PK__IMPL_Age__…` e afins): `PK_tbagenda`, `PK_tbagendaparticipante`, `PK_tbauditoriaacesso`, `PK_tbrefreshtoken`, `PK_tbtipoevento`. |
| Unique constraint | 1 | `UQ__CC_TipoE__7D8FE3B22A85213B`→`AK_tbtipoevento_Nome`, pela convenção do EF para chave alternativa (`AK_<tabela>_<colunas>`). |

**Duas armadilhas do `sp_rename` que causaram falha silenciosa** (o motivo de a `PadraoTabelasTb` ter saído com 33 índices e 5 PKs pendentes, apesar de ter sido aplicada sem erro):

1. **Índice exige o itemtype `'INDEX'`.** Sem esse argumento, `OBJECT_ID('dbo.<índice>')` devolve `NULL`, porque índice não vive no namespace de schema. O guard `IF NOT EXISTS` então passava e o `sp_rename` era **pulado sem erro** — falha silenciosa. Constraint (PK/UQ), ao contrário, é objeto de schema (tipo `PK`/`UQ`): o guard por `OBJECT_ID` funciona e o itemtype default basta.
2. **O nome do índice precisa ser qualificado com a tabela**, no formato `"tabela.índice"`. Tanto `"dbo.índice"` quanto `"índice"` falham com o erro **15248** (*@objname é ambíguo ou o @objtype INDEX reivindicado está errado*).

⚠️ **Lacuna no `Down`:** o `Down` desta migration repete as mesmas chamadas de renomeação do `Up` (não reverte para os nomes antigos). Reverter exige um ajuste manual antes de qualquer uso.

Conferência após a aplicação no banco alvo: 0 tabelas, 0 índices e 0 constraints de PK/AK fora do padrão, com as 24 tabelas do snapshot presentes.


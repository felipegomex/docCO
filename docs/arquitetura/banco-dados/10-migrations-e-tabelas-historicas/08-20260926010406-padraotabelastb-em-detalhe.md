---
id: 10-migrations-e-tabelas-historicas.08
titulo: "`20260926010406_PadraoTabelasTb` em detalhe"
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
tokens_estimados: 643
ficha: docs/modulos/database-explorer.md
---
> Parte de [Migrations e tabelas históricas](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### `20260926010406_PadraoTabelasTb` em detalhe

Migration **escrita à mão** (o scaffold do EF não serviria) com três motivos: `tbfuncao` já existe como legada, então o EF geraria `RenameTable` e falharia; as tabelas órfãs da Agenda não pertencem ao modelo, então o EF nunca as removeria; e o scaffold emitiria `Drop`/`Add` de todas as PKs, FKs e índices, o que é desnecessário porque `sp_rename` só mexe em metadados.

| Grupo | O que faz |
|---|---|
| Renomeações (14) | `IMPL_Projeto`→`tbprojeto`, `IMPL_Tarefa`→`tbtarefa`, `IMPL_ComentarioTarefa`→`tbcomentariotarefa`, `IMPL_TarefaResponsavel`→`tbtarefareponsavel`, `IMPL_TarefaChamado`→`tbtarefachamado`, `IMPL_TarefaApontamento`→`tbtarefaapontamento`, `IMPL_TipoProjeto`→`tbtipoprojeto`, `IMPL_Agenda`→`tbagenda`, `IMPL_Auditoria`→`tbauditoriaimplantacao`, `CC_TipoEvento`→`tbtipoevento`, `CC_AgendaParticipante`→`tbagendaparticipante`, `RefreshTokens`→`tbrefreshtoken`, `AuditoriaAcessos`→`tbauditoriaacesso` e `IMPL_ColunaKanban`→`tbcolunakanban`. |
| Normalização de caixa (6) | `tbprojetoEtapa`→`tbprojetoetapa`, `tbprojetoEtapaChecklist`→`tbprojetoetapachecklist`, `tbprojetoEtapaDocumento`→`tbprojetoetapadocumento`, `tbprojetoEtapaHistorico`→`tbprojetoetahistorico`, `tbprojetoEtapaComentario`→`tbprojetoetapacomentario`, `TBOPERADOR`→`tboperador`. |
| Constraints | 20 PKs renomeadas por `sp_rename` (`PK_tbprojeto`, `PK_tboperador`, `PK_tbprojetoetapa*`, …), mais a `PK_tbfuncao` recriada em `ALTER TABLE` durante a fusão. **Nenhum índice foi renomeado aqui** e 5 PKs ficaram com o nome automático do EF: ambos ficaram para a `RenomeiaIndicesPkTb`. **As FKs não foram renomeadas** (cosmético; ver a seção de nomenclatura). |
| Fusão da função | `CC_Funcao` descartada; `tbfuncao` legada enriquecida (ver seção própria abaixo). |
| Remoções | `IMPL_Cliente`, `CC_Funcao`, `AgendaEvento` e `AgendaEventoParticipante`. |
| Preservadas de propósito | `chamado_bkp`, `chamado_itens_bkp`, `funcionarios` e `tb_testes` — não pertencem ao projeto. |

O `Down` é best-effort: devolve os nomes antigos e recria `CC_Funcao` a partir da `tbfuncao` enriquecida, mas **não** recria `IMPL_Cliente`, `AgendaEvento` nem `AgendaEventoParticipante` (devolver schema, não os dados de um cadastro morto).


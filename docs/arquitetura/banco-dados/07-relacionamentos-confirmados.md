---
id: sec.07-relacionamentos-confirmados
titulo: "Relacionamentos confirmados"
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
tokens_estimados: 1737
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Relacionamentos confirmados

| Origem | Destino | Chave / ação no banco | Observação |
|---|---|---|---|
| `Operador` | `Funcao` | `FUNCAO_ID`, `SetNull` | Relação opcional (setor). |
| `Operador` | `Cargo` | `CARGO_ID`, `SetNull` | Relação opcional (responsabilidade RBAC); perder o cargo **não** apaga o operador. |
| `PermissaoCargo` | `Cargo`/`PermissaoRecurso`/`Funcao` | `PEC_CargoId`/`PEC_RecursoId`/`PEC_FuncaoId`, `Restrict` | A matriz não é apagada em cascata; `PEC_FuncaoId NULL` = todas as funções. |
| `PermissaoOperador` | `Operador`/`PermissaoRecurso`/`Funcao` | `PEO_OperadorId`/`PEO_RecursoId`/`PEO_FuncaoId`, `Restrict` | Exceção individual; a negativa vence o cargo. |
| `RefreshToken` | `Operador` | `OperadorId`, `Cascade` | Exclusão do operador remove refresh tokens. |
| `Projeto` | `TipoProjeto` | `TipoProjetoId`, `Restrict` | O banco impede a exclusão de tipo com projeto vinculado; o serviço lança `InvalidOperationException` e o controller atual não a trata, podendo resultar em `500` (limitação). |
| `Projeto` | `ClienteLegado` | `ClienteId`, `SetNull` | A FK aponta para a legada `tbcliente`; não existe mais tabela `IMPL_Cliente`. |
| `Projeto` | `ColunaKanban` | `ColunaKanbanId`, `SetNull` | Coluna pode ser removida do projeto. |
| `Projeto` | `ProjetoEtapa` | `ProjetoId`, `Cascade` | Etapas e dependentes são removidos em cascata. |
| `ProjetoEtapa` | checklist/documento/histórico/comentário | `ProjetoEtapaId`, `Cascade` | Quatro coleções filhas. |
| `Tarefa` | `Projeto` | `ProjetoId`, `Cascade` | Tarefa pode existir sem projeto porque a coluna é anulável. |
| `Tarefa` | `ProjetoEtapa` | `ProjetoEtapaId`, `Restrict` | Vínculo de etapa do mesmo projeto é validado no serviço. |
| `Tarefa` | `ColunaKanban` | `ColunaKanbanId`, `SetNull` | Coluna é opcional. |
| `Tarefa` | comentários/responsáveis/chamados/apontamentos | `TarefaId`, `Cascade` | Quatro tabelas filhas. |
| `AgendaItem` | `Projeto` | `ProjetoId`, `SetNull` | Evento continua sem projeto se o projeto for removido. |
| `AgendaItem` | `TipoEvento` | `TipoId`, `SetNull` | Tipo é opcional. |
| `AgendaItem` | `AgendaParticipante` | `AgendaId`, `Cascade` | Participantes são filhos do evento. |
| `AgendaParticipante` | `Operador` | `ParticipanteId`, `Restrict` | Participante precisa existir. |
| `TbLead` | `ClienteLegado` | `ClienteId`, `SetNull` | Apagar o cliente legado não derruba o lead. |
| `TbLead` | `Operador` | `OperadorId`, `SetNull` | Operador é opcional. |
| `TbOportunidade` | `TbLead` | `LeadId`, `Cascade` | Excluir o lead apaga as oportunidades. |
| `TbOportunidade` | `ProdutoLegado` | `ProdutoId`, `SetNull` (lógico, sem FK física) | Produto vem da legada `tbproduto` (leitura); sem produto vinculado a linha continua válida. |
| `TbOrcamento` | `ClienteLegado` | `ClienteId`, `SetNull` | Cliente é opcional no orçamento. |
| `TbLeadContato` | `TbLead` | `LeadId`, `Cascade` | Apagar o lead remove o histórico de contatos. |
| `ClienteLegado` | `ClienteGrupoLegado` | `GrupoId`, `Restrict` | `tbcliente.GRUPO_ID` → `tbcliente_grupo.GRUPO_ID`. O legado não declara FK, então o `Restrict` é explícito e serve só à leitura. |
| `TbContratoLegado` | `ClienteLegado` | `ClienteId`, `Restrict` | Base do somatório de `QTDE_LICENCA` na tela Administrativa; a consulta usa LEFT JOIN para cliente sem contrato continuar na lista. |
| `ClienteAcesso` | `ClienteGrupoLegado` | `CAO_GrupoId`, `Restrict` | Grupo vem da legada `tbcliente_grupo` (`GRUPO_ID smallint`). É **anulável**; grupo inativo continua vinculado, mas a criação/edição de cliente **recusa** grupo inexistente ou inativo (`400`). A tabela `tbgrupoacesso` e a entidade `GrupoAcesso` foram **removidas**. |
| `ClienteAcesso` | `ClienteLegado` | `CAO_ClienteId`, `Restrict` | Vínculo com a empresa do ERP (`tbcliente.CLIENTE_ID`), exigido pelo código: toda unidade nasce de `clienteId` no `POST /clientes` (o cadastro manual foi removido). Índice único filtrado garante no máximo uma unidade por empresa; a coluna continua anulável no esquema só por herança histórica. |
| `AcessoRdp`/`Vpn`/`AnyDesk`/`Banco`/`ActyonWeb` | `ClienteAcesso` | `*_ClienteAcessoId`, `Restrict` | Cinco FKs idênticas em estrutura. `ClienteAcessoId` é **obrigatório**, e criar acesso para cliente inativo é recusado (`400`). Nada é removido em cascata: quem desativa é `Ativo`. |
| `Tarefa`/`Projeto`/`ColunaKanban` | `Funcao` | `TRF_FuncaoId`/`PRJ_FuncaoId`/`CLK_FuncaoId`, `SetNull` | Eixo do Kanban Geral. Perder a função **não** apaga tarefa, projeto nem coluna: a função é o classificador do quadro, não o dono do dado. No modelo vigente `CLK_FuncaoId` é sempre `NULL` — a coluna é do catálogo. |
| `ColunaFuncao` | `ColunaKanban` | `CLF_ColunaKanbanId`, `Cascade` | A visão aponta para a coluna do catálogo; se a coluna sair do banco, a visão cai junto. |
| `ColunaFuncao` | `Funcao` | `CLF_FuncaoId`, `Cascade` | A visão pertence ao setor; apagar a função apaga as visões dela. |
| `EtapaFuncao` | `Funcao` | `FUNCAO_ID`, `Cascade` | Etapas da função são filhas da função dona. |
| `Tarefa` | `EtapaFuncao` | `EtapaTemplateId` | Etapa padrão da função, gravada junto da etapa real do projeto. O `Down` da `20260929221556` a declara `NoAction`. |

### IDs lógicos sem FK declarada

O modelo não cria FK para vários identificadores usados como referência de negócio. Isso não deve ser tratado como integridade referencial garantida pelo banco:

- `Projeto.ClienteLegadoId` e `Tarefa.ChamadoLegadoId` foram adicionados como colunas indexadas sem FK, para não bloquear inserções quando o legado mudar. O serviço valida `ChamadoLegadoId` em vários fluxos, mas a validação de `Projeto.ClienteLegadoId` não é equivalente à de `ClienteId`.
- `TarefaChamado.ChamadoId`, `TarefaResponsavel.OperadorId`, `TarefaApontamento.OperadorId`, `AgendaItem.OperadorId`, `Projeto.ResponsavelId` e `Tarefa.ResponsavelId` são referências lógicas; algumas são verificadas no serviço, outras não.
- `FuncionarioLegado.FuncaoId` não possui FK para `tbfuncao`; a consulta da agenda usa a correspondência por `OperadorId`, função e ativo.
- `ComentarioTarefa.AutorId` e campos de auditoria também não têm FK para `tboperador`.

`ClienteLegado` é a única entidade de cliente do modelo: a entidade `Cliente` e a tabela `IMPL_Cliente` foram removidas, e a migração `ClienteApontaTbCliente` aponta `Projeto.ClienteId` para `tbcliente`. O código do serviço usa `ClientesLegado` para listar clientes. O grupo econômico é um espelho separado (`ClienteGrupoLegado`/`tbcliente_grupo`), também somente leitura e fora das migrations.


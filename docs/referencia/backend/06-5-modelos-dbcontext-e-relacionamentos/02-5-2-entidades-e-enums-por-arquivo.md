---
id: 06-5-modelos-dbcontext-e-relacionamentos.02
titulo: "5.2 Entidades e enums por arquivo"
tipo: referencia
area: backend
sistema: central
tags: [backend, modelos, dbcontext, efcore]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 2891
ficha: docs/modulos/database-explorer.md
---
> Parte de [5. Modelos, DbContext e relacionamentos](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### 5.2 Entidades e enums por arquivo

| Arquivo | Conteúdo persistente ou de domínio |
|---|---|
| [`Operador.cs`](../../../../backend/Central_BackEnd/Models/Operador.cs) | Operador, flags, perfil, vínculo com função (`FUNCAO_ID` = setor) e **`CARGO_ID`** (cargo RBAC) |
| [`Permissoes/Cargo.cs`](../../../../backend/Central_BackEnd/Models/Permissoes/Cargo.cs) | Cargo RBAC (`tbcargo`): código único, descrição, escopo padrão, `Nivel` (**`byte`** ↔ `CARGO_Nivel tinyint`) e `FuncaoId` (setor do cargo) |
| [`Permissoes/PermissaoRecurso.cs`](../../../../backend/Central_BackEnd/Models/Permissoes/PermissaoRecurso.cs) | Catálogo de recursos (`tbpermissao_recurso`): chave, tipo, rota/controller |
| [`Permissoes/PermissaoCargo.cs`](../../../../backend/Central_BackEnd/Models/Permissoes/PermissaoCargo.cs) | Matriz cargo × recurso × função-alvo (`tbperm_cargo`) |
| [`Permissoes/PermissaoOperador.cs`](../../../../backend/Central_BackEnd/Models/Permissoes/PermissaoOperador.cs) | Exceção individual (`tbperm_operador`), com `PEO_Negativa` |
| [`Permissoes/AcaoPermissao.cs`](../../../../backend/Central_BackEnd/Models/Permissoes/AcaoPermissao.cs) | Enums `AcaoPermissao` (bitset), `EscopoPermissao` e `TipoRecurso` |
| [`Funcao.cs`](../../../../backend/Central_BackEnd/Models/Funcao.cs) | Função e classificação |
| [`RefreshToken.cs`](../../../../backend/Central_BackEnd/Models/RefreshToken.cs) | Hash, expiração, revogação e substituição |
| [`Projeto.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/Projeto.cs) | Projeto e enums de status/prioridade |
| [`ProjetoEtapa.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/ProjetoEtapa.cs) | Card e quatro coleções 1:N |
| [`Tarefa.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/Tarefa.cs) | Tarefa, comentário e enums de status/prioridade/tipo |
| [`TarefaResponsavel.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/TarefaResponsavel.cs) | N:N Tarefa–Operador |
| [`TarefaChamado.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/TarefaChamado.cs) | N:N Tarefa–Chamado legado |
| [`TarefaApontamento.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/TarefaApontamento.cs) | Apontamento decimal de horas |
| [`AgendaItem.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/AgendaItem.cs) | Agenda, prioridade e recorrência |
| [`AgendaParticipante.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/AgendaParticipante.cs) | N:N Agenda–Participante |
| [`TipoEvento.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/TipoEvento.cs) | Tipo de evento ativo |
| [`TipoProjeto.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/TipoProjeto.cs) | Tipo de Projeto |
| [`ColunaKanban.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/ColunaKanban.cs) | Coluna do **catálogo compartilhado**: ordem `decimal(5,2)`, WIP, flags, `StatusCard` e `FuncaoId` (sempre `NULL` no modelo vigente) — tabela `tbcolunakanban` |
| [`ColunaFuncao.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/ColunaFuncao.cs) | **Visão do setor** sobre o catálogo: `Visivel` (oculta/ativa a coluna no quadro do setor) e `NomeAlternativo` (máscara de nome) — tabela `tbcolunafuncao` |
| [`Funcao.cs`](../../../../backend/Central_BackEnd/Models/Funcao.cs) | **Eixo do Kanban Geral** (`tbfuncao`): descrição, classificação e ativo. Sem cor e sem ordem — a cor vem da `CLASSIFICACAO` no cliente |
| [`EtapaFuncao.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/EtapaFuncao.cs) | Etapa-template fixa da função (`tbfuncaoetapa`); não confundir com `ProjetoEtapa` |
| [`Bloqueio.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/Bloqueio.cs) | `TipoOrigemBloqueio` (Tarefa/Projeto/ProjetoEtapa) e a linha de bloqueio como atributo |
| [`AuditoriaImplantacao.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/AuditoriaImplantacao.cs) | Entidade, ação, antes/depois, usuário e observação |
| [`ClienteLegado.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/ClienteLegado.cs) | Espelho somente leitura de `tbcliente`, com `GrupoId`, navegação `Grupo` e coleção `Contratos` |
| [`ClienteGrupoLegado.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/ClienteGrupoLegado.cs) | Espelho somente leitura de `tbcliente_grupo` (grupo econômico), fora das migrations |
| [`FuncionarioLegado.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/FuncionarioLegado.cs) | Espelho somente leitura de `tbfuncionario` |
| [`ChamadoLegado.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/ChamadoLegado.cs) | Espelho somente leitura de `tbchamado` |
| [`TbLead.cs`](../../../../backend/Central_BackEnd/Models/Dashboard/TbLead.cs), [`TbOportunidade.cs`](../../../../backend/Central_BackEnd/Models/Dashboard/TbOportunidade.cs), [`ProdutoLegado.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/ProdutoLegado.cs) | Funil comercial do Dashboard Integrado (lead com arquivamento, oportunidade por produto, catálogo `tbproduto`) |
| [`TbOrcamento.cs`](../../../../backend/Central_BackEnd/Models/Dashboard/TbOrcamento.cs), [`TbCusto.cs`](../../../../backend/Central_BackEnd/Models/Dashboard/TbCusto.cs) | Orçamentos e custos do Dashboard Integrado |
| [`TbContratoLegado.cs`](../../../../backend/Central_BackEnd/Models/Dashboard/TbContratoLegado.cs) | Espelho somente leitura de `tbcontrato` |
| [`ConstantesDashboard.cs`](../../../../backend/Central_BackEnd/Models/Dashboard/ConstantesDashboard.cs) | Etapas comerciais, status de orçamento, tipos de custo e respostas de proposta (domínio, sem persistência) |
| [`Ideia.cs`](../../../../backend/Central_BackEnd/Models/Ideias/Ideia.cs) | Solicitação do módulo de Ideias e os enums `TipoIdeia` (0/1/2), `StatusIdeia` (0..6) e `PrioridadeIdeia` (0..2) |
| [`MensagemIdeia.cs`](../../../../backend/Central_BackEnd/Models/Ideias/MensagemIdeia.cs) | Mensagem append-only e o enum `TipoMensagemIdeia` (0 Comentário, 1 Resposta a esclarecimento) |
| [`HistoricoStatusIdeia.cs`](../../../../backend/Central_BackEnd/Models/Ideias/HistoricoStatusIdeia.cs) | Transição de status **e** decisão administrativa, append-only |
| [`AnaliseIdeia.cs`](../../../../backend/Central_BackEnd/Models/Ideias/AnaliseIdeia.cs) | Execução da análise por IA e o enum `EstadoAnaliseIdeia` (0..6, um estado por modo de falha) |
| [`EspecificacaoIdeia.cs`](../../../../backend/Central_BackEnd/Models/Ideias/EspecificacaoIdeia.cs) | Versão da especificação técnica e o enum `EstadoEspecificacaoIdeia` (0/1/2) |
| [`Gestor.cs`](../../../../backend/Central_BackEnd/Models/Gestores/Gestor.cs) | Gestor do módulo de Ideias, por função ou geral |
| [`TbDoc.cs`](../../../../backend/Central_BackEnd/Models/TbDoc.cs) | Espelho de um documento de `docs/` em `tbdoc_doc` (`Caminho` único, `Titulo`, `Secao`, corpo markdown, `Hash` de 8 hex, `SincronizadoEm` UTC); só leitura pelo backend |
| [`WhatsAppEnums.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/WhatsAppEnums.cs) | Enums do módulo com **base `byte` obrigatória** (as colunas são `tinyint`; com `int` o EF pede `int` e o SQL Server devolve `byte`, morrendo com `InvalidCastException`): `StatusAtendimentoWhatsApp` (**0–6**: 0 não atendido · 1 em atendimento · 2 aguardando cliente · 3 resolvido · 4 finalizado · 5 arquivado · **6 visto**), `DirecaoMensagemWhatsApp` (0–2), `StatusMensagemWhatsApp` (0–4), `PrioridadeAtendimento` (0–2), **`TipoContatoWhatsApp` (0–4 — Fase 01 do novo ciclo: 0 não classificado · 1 TI · 2 operador de cobrança · 3 gestor · 4 planejamento — com a extensão `.Rotulo()`, que devolve "Não classificado" para valor desconhecido)** e a classe de tipos de evento `TiposEventoWhatsApp` (que ganhou, na **Fase 06**, `etiqueta_adicionada`, `etiqueta_removida`, `investigacao_iniciada`, `investigacao_finalizada` e `observacao_atualizada`; na **F4**, `finalizacao` e `arquivamento` — antes a finalização reutilizava `resolucao`; e, na **F6**, `visto`). **Não** usam `JsonStringEnumConverter`: a numeração é o contrato do módulo |
| [`Numero.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/Numero.cs), [`Contato.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/Contato.cs), [`Conversa.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/Conversa.cs) | Entidades do canal: número (sessão do gateway), contato (chat) e conversa (canal por número) |
| [`Atendimento.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/Atendimento.cs) | O card do Kanban: status, responsável, prioridade, marcas de tempo do ciclo e contador de reaberturas; **Fase 06**: `EmInvestigacao` (`WAT_EmInvestigacao`, atributo booleano — não é status) e `ObservacaoInterna` (`WAT_ObservacaoInterna`, `nvarchar(500)` do atendimento atual); **Fase 07**: navegação de leitura `Etiquetas` (`ICollection<AtendimentoEtiqueta>`) usada pelo filtro `EXISTS` e pela consulta agrupada — sem coluna nova |
| [`Etiqueta.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/Etiqueta.cs) | **Fase 06**: `Etiqueta` (`tbwhatsapp_etiqueta` — nome, cor fechada, ativo e auditoria) e `AtendimentoEtiqueta` (`tbwhatsapp_atendimento_etiqueta` — N:N com PK composta e auditoria de inclusão); taxonomia **nova**, sem relação com a Fase D removida |
| [`Mensagem.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/Mensagem.cs) | Mensagem da conversa (recebida, enviada ou **nota interna**), com estado de entrega, mídia opcional e **`AutorNome`** (`WAT_AutorNome`, `nvarchar(120)` nulo) — o pushName de quem enviou, usado no balão de grupo; nulo na enviada pela Central e em dados anteriores à coluna |
| [`Evento.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/Evento.cs) | Histórico **append-only** das transições de atendimento |
| [`WebhookFila.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/WebhookFila.cs) | Fila **durável** dos webhooks recebidos (chave de dedupe, payload, estado, tentativas e backoff) |
| [`Parametro.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/Parametro.cs) | Parâmetro chave/valor do módulo (`tbwhatsapp_parametro`), com índice único pela chave |
| [`RespostaRapida.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/RespostaRapida.cs) | Resposta rápida (`WAT_Atalho` único, categoria, título, corpo, ativo) |
| [`SlaConfig.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/SlaConfig.cs), [`SlaConfig.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/SlaConfig.cs) (SlaHorario/SlaHistorico consolidados) | Metas de primeira resposta/resolução, horário útil por dia ISO (1 = segunda … 7 = domingo) e histórico de alteração de meta |
| [`RetencaoConfig.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/RetencaoConfig.cs), [`RetencaoConfig.cs`](../../../../backend/Central_BackEnd/Models/WhatsApp/RetencaoConfig.cs) (RetencaoExecucao consolidado) | Regra de retenção (`WAT_DiasRetencao`, padrão 1) e o registro de cada execução (origem, contagens, bytes liberados, erro) |

⚠️ **`StatusIdeia` não conflita com `StatusTarefa` nem `StatusCard`.** Os três têm valores 0..N e descrevem coisas diferentes: `StatusIdeia` é o ciclo de vida do **pedido**, `StatusTarefa` e `StatusCard` são o estado do **cartão** no Kanban. O `CHECK` no banco limita `IDE_Status` a 0..6; os enums são gravados como `tinyint`.


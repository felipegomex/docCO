---
id: 06-inventario-de-entidades-e-tabelas.01
titulo: "Introdução"
tipo: referencia
area: dados
sistema: central
tags: [schema, tabelas, entidades, sqlserver]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
  - backend/Central_BackEnd/Migrations
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 3464
ficha: docs/modulos/database-explorer.md
---
> Parte de [Inventário de entidades e tabelas](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### Introdução

> Parte de [docs/arquitetura/banco-dados.md](../../banco-dados.md).

`Gerida` significa que a entidade está incluída no modelo de migrações. `Legada` significa `ExcludeFromMigrations()` e uso somente leitura no código de negócio.

| `DbSet` / entidade | Tabela | Chave | Gestão | Função / campos de controle |
|---|---|---|---|---|
| `Operadores` / `Operador` | `tboperador` | `OperadorId` | Gerida | Identidade, estado, perfil, função, referência de auditoria, `FUNCAO_ID` (setor — eixo do Kanban Geral) e **`CARGO_ID`** (responsabilidade RBAC; FK anulável para `tbcargo`, `SetNull`, índice `IX_tboperador_CARGO_ID`). A coluna `SetorId` foi **removida**. |
| `Funcoes` / `Funcao` | `tbfuncao` | `Id` (`FUNCAO_ID`) | Gerida | **Eixo do Kanban Geral.** `DESCRICAO`, `CLASSIFICACAO` e `ATIVO` — sem cor e sem ordem. Cinco linhas na base real (2026-10-21): 1 `Analista de Sistemas`/`Implantador`, 2 `Suporte`/`Atendimento`, 3 `Programador`/`Desenvolvimento`, 4 `Executivo(a) Comercial` e 5 `CIAA`. ⚠️ `FUNCAO_ID` **não é IDENTITY** (próximo livre é manual) e `DESCRICAO` é `varchar(20)` na base, embora o modelo declare 100. |
| `RefreshTokens` / `RefreshToken` | `tbrefreshtoken` | `Id` | Gerida | Hash do refresh, expiração, revogação, substituição e operador. |
| `AuditoriaAcessos` / `AuditoriaAcesso` | `tbauditoriaacesso` | `Id` | Gerida | Revelação de credencial: operador, empresa, tipo, data/hora, IP, user-agent e o vínculo `AAC_ClienteAcessoId`/`AAC_AcessoTipo`/`AAC_AcessoId`. Nunca guarda a senha. |
| `TiposProjeto` / `TipoProjeto` | `tbtipoprojeto` | `Id` | Gerida | Código único, nome, obrigatoriedade de cliente, ordem e ativo. |
| `ColunasKanban` / `ColunaKanban` | `tbcolunakanban` | `Id` | Gerida | **Catálogo compartilhado de colunas** do Kanban Geral: as mesmas colunas para todos os setores (a análise comparativa entre funções exige o mesmo conjunto). Ordem (`decimal(5,2)`, permite inserir coluna entre duas), cor, padrão, ativo, limite WIP, `CLK_StatusCard` e `CLK_SincronizaAgenda`. `CLK_FuncaoId` é **sempre `NULL`** no modelo vigente — a coluna não pertence a uma função; quem a rege é a visão em `tbcolunafuncao` (o campo sobrou do eixo Função e não é mais escrito). A coluna "BLOQUEADO" existe, mas está **inativa**. Ver a seção `ColunasCompartilhadas_Idempotente.sql`. |
| `ColunasFuncao` / `ColunaFuncao` | `tbcolunafuncao` | `Id` (`CLF_Id`) | Gerida | **Visão do setor sobre o catálogo**: `CLF_ColunaKanbanId` (FK `Cascade` → `tbcolunatecanban.CLK_Id`), `CLF_FuncaoId` (FK `Cascade` → `tbfuncao.FUNCAO_ID`), `CLF_Visivel` (oculta/ativa a coluna no quadro do setor) e `CLF_NomeAlternativo` (máscara de nome, `varchar(50)` anulável — vazio = usa o nome do catálogo), mais auditoria. Índice **único** `(CLF_ColunaKanbanId, CLF_FuncaoId)` e índice em `CLF_FuncaoId`. Sem linha para o setor = visão padrão (coluna visível, sem máscara). |
| `Projetos` / `Projeto` | `tbprojeto` | `Id` | Gerida | Código global, cliente, tipo, responsável, datas, progresso, horas, go-live e `PRJ_FuncaoId` (Kanban Geral). |
| `ProjetoEtapas` / `ProjetoEtapa` | `tbprojetoetapa` | `Id` | Gerida | Jornada de nove etapas por projeto; `ProjetoId + Ordem` é único. |
| `ProjetoEtapaChecklists` / `ProjetoEtapaChecklist` | `tbprojetoetapachecklist` | `Id` | Gerida | Itens de checklist e conclusão. |
| `ProjetoEtapaDocumentos` / `ProjetoEtapaDocumento` | `tbprojetoetapadocumento` | `Id` | Gerida | Nome, URL, descrição e autoria do documento. |
| `ProjetoEtapaHistoricos` / `ProjetoEtapaHistorico` | `tbprojetoetahistorico` | `Id` | Gerida | Histórico textual das alterações da etapa. |
| `ProjetoEtapaComentarios` / `ProjetoEtapaComentario` | `tbprojetoetapacomentario` | `Id` | Gerida | Comentário, usuário e data. |
| `Tarefas` / `Tarefa` | `tbtarefa` | `Id` | Gerida | Projeto/etapa opcionais, status, prazo, entrega, responsável, horas, arquivamento e, no Kanban Geral, `TRF_FuncaoId` e `TRF_EtapaTemplateId`. **Não** tem mais `TRF_Bloqueada`/`TRF_MotivoBloqueio` (removidas); o bloqueio vive em `tbbloqueio`. |
| `ComentariosTarefa` / `ComentarioTarefa` | `tbcomentariotarefa` | `Id` | Gerida | Comentário de tarefa e autor lógico. |
| `TarefaResponsaveis` / `TarefaResponsavel` | `tbtarefareponsavel` | `(TarefaId, OperadorId)` | Gerida | Responsáveis adicionais em N:N. |
| `TarefaChamados` / `TarefaChamado` | `tbtarefachamado` | `(TarefaId, ChamadoId)` | Gerida | Ligação N:N entre tarefa e chamado legado. |
| `TarefaApontamentos` / `TarefaApontamento` | `tbtarefaapontamento` | `Id` | Gerida | Horas, operador lógico, data e observação. |
| `EtapasFuncao` / `EtapaFuncao` | `tbfuncaoetapa` | `Id` (`FUN_EtapaFuncaoId`) | Gerida | Catálogo de etapas **fixas** por função, com `FUNCAO_ID` (FK `Cascade`), `FUN_Nome` (único entre ativas da mesma função), `FUN_Ordem`, `FUN_Cor`, `FUN_Descricao`, `FUN_Ativo` e auditoria. Substitui `tbetapsetor`. O projeto clona os nomes para `tbprojetoetapa`. |
| `Bloqueios` / `Bloqueio` | `tbbloqueio` | `Id` (`BLQ_Id`) | Gerida | Bloqueio como **atributo** do card: `BLQ_OrigemTipo` (0 Tarefa, 1 Projeto, 2 Etapa de projeto), `BLQ_OrigemId`, `BLQ_Motivo`, `BLQ_ResolvidoEm`/`BLQ_ResolvidoPor`. Índice único filtrado garante 1 bloqueio aberto por origem. |
| `ChamadosLegado` / `ChamadoLegado` | `tbchamado` | `Id` | Legada | Espelho somente leitura; colunas ignoradas pelo modelo. |
| `Agenda` / `AgendaItem` | `tbagenda` | `Id` | Gerida | Responsável, intervalo, tipo, prioridade, SLA, projeto, recorrência (`Recorrente`, `PadraoRecorrencia`), **`AGD_SerieId`** (int anulável, **sem FK** — agrupa as ocorrências de um lote para a reconciliação na edição) e participantes. |
| `TiposEvento` / `TipoEvento` | `tbtipoevento` | `Id` | Gerida | Nome único, cor e ativo. |
| `AgendaParticipantes` / `AgendaParticipante` | `tbagendaparticipante` | `Id` | Gerida | Participante do evento; par evento/participante é único. |
| `AuditoriaImplantacao` / `AuditoriaImplantacao` | `tbauditoriaimplantacao` | `Id` | Gerida | Entidade, ação, usuário, data, observação e JSON antes/depois. |
| `FuncionariosLegado` / `FuncionarioLegado` | `tbfuncionario` | `FuncionarioId` | Legada | Fallback de função quando o operador está sem função ([`FuncionarioLegado.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/FuncionarioLegado.cs)). `FuncaoId` é `short?` (`FUNCAO_ID smallint` — corrigido no commit `0d72b9e`); desde o commit `e718cee` a fonte padrão é `tboperador` e a Agenda não usa mais este espelho. |
| `ClientesLegado` / `ClienteLegado` | `tbcliente` | `Id` | Legada | CNPJ, razão social, fantasia, ativo e `GRUPO_ID`; fonte de clientes do módulo e base da tela Administrativa. |
| `ClientesGrupo` / `ClienteGrupoLegado` | `tbcliente_grupo` | `Id` (`short`) | Legada | Grupo econômico: `GRUPO_ID` (PK `smallint`), `NOME` e auditoria (`DATA_ALTERACAO`, `USUARIO_ALTERACAO`). Fora das migrations. |
| `ProdutosLegado` / `ProdutoLegado` | `tbproduto` | `Id` (`PRODUTO_ID`) | Legada | Catálogo de produtos da Proposta: `DESCRICAO`, `OBSE` (não `OBS`), `PRODUTO_ID` (`smallint`), auditoria de inclusão/alteração. Fora das migrations, somente leitura. |
| `Leads` / `TbLead` | `tblead` | `Id` | Gerida | Funil comercial: título, cliente/operador legados, etapa, contatos, anotações, flag manual `LEAD_ContatoVisualizado` e arquivamento (`LEAD_Arquivado`, `LEAD_DataArquivamento`, `LEAD_MotivoArquivamento`). |
| `Oportunidades` / `TbOportunidade` | `tboportunidade` | `Id` | Gerida | Uma linha por produto ofertado: `OPP_ProdutoId` (lógico → `tbproduto`), `OPP_ValorImplementacao`, `OPP_ValorPorLicenca` (PA = licença), `OPP_QtdeLicencas`, `OPP_MotivoPerda`, valor estimado calculado, etapa, ciclo de proposta e observação. |
| `Orcamentos` / `TbOrcamento` | `tborcamento` | `Id` | Gerida | Orçamentos financeiros: cliente legado, valor, status e datas de aprovação/faturamento. |
| `Custos` / `TbCusto` | `tbcusto` | `Id` | Gerida | Custos mensais por tipo (Salário, Imposto, Estrutura, Outro) e competência `AAAA-MM`. |
| `ContatosLead` / `TbLeadContato` | `tbleadcontato` | `Id` | Gerida | Histórico de prospecção: `LeadId` (FK cascade), `LDC_NomeContato` obrigatório, `Tipo` fixo (7 valores), `DataContato`, `Observacao`, auditoria. Índices em `LeadId` e `Tipo`. |
| `ContratosLegado` / `TbContratoLegado` | `tbcontrato` | `Id` | Legada | Espelho somente leitura do contrato legado. Colunas mapeadas conforme o dicionário real; as demais são ignoradas. Detalhes na seção própria abaixo. |
| `ClientesAcesso` / `ClienteAcesso` | `tbclienteacesso` | `Id` (`CAO_Id`) | Gerida | **Nível 2**: cliente/unidade/filial, dono de todos os acessos. `CAO_GrupoId` (FK `Restrict`, anulável → `tbcliente_grupo.GRUPO_ID`), **`CAO_ClienteId`** (FK `Restrict`, anulável → `tbcliente.CLIENTE_ID`, índice **único filtrado** — vincula a unidade à empresa do ERP e é a **chave de unicidade**), soft delete por `CAO_Ativo` e as 5 flags **derivadas** `CAO_TemRdp/TemVpn/TemAnyDesk/TemBanco/TemActyonWeb`. **Não guarda identidade**: `CAO_Fantasia`/`CAO_Cnpj` foram dropadas (`20261002161552`) — fantasia, CNPJ e grupo vêm da `tbcliente` por JOIN. |
| `AcessosRdp` / `AcessoRdp` | `tbacessordp` | `Id` (`RDP_Id`) | Gerida | **Nível 3A**: `RDP_Servidor`, `RDP_Ip`, `RDP_Porta` (default 3389), `RDP_Usuario`, `RDP_SenhaCifrada`. |
| `AcessosVpn` / `AcessoVpn` | `tbacessovpn` | `Id` (`VPN_Id`) | Gerida | **Nível 3B**: `VPN_TipoVpn`, `VPN_Nome`, `VPN_Usuario`, `VPN_SenhaCifrada`, `VPN_Gateway` (formato `host:porta`), `VPN_ConfigArquivo`. |
| `AcessosAnyDesk` / `AcessoAnyDesk` | `tbacessoanydesk` | `Id` (`ADK_Id`) | Gerida | **Nível 3C**: `ADK_NumeroAnyDesk`, `ADK_Usuario`, `ADK_SenhaCifrada`. Nenhum campo obrigatório. |
| `AcessosBanco` / `AcessoBanco` | `tbacessobanco` | `Id` (`BCO_Id`) | Gerida | **Nível 3D**: `BCO_NomeBanco`, `BCO_Servidor`, `BCO_Porta` (default 1433), `BCO_Usuario`, `BCO_SenhaCifrada`, `BCO_VersaoSql`, `BCO_VersaoCob`, `BCO_ArquivoConfig`. |
| `AcessosActyonWeb` / `AcessoActyonWeb` | `tbacessoactyonweb` | `Id` (`AWB_Id`) | Gerida | **Nível 3E**: `AWB_Url`, `AWB_Porta` (default 1010), `AWB_Stack`, `AWB_Ambiente` e `Usuario` (herdada da base, **sem prefixo**). **Sem senha** — o acesso é por URL/sessão. Tabela com **11 colunas**. |
| `Ideias` / `Ideia` | `tbideia` | `Id` (`IDE_Id`) | Gerida (script próprio) | Módulo de Ideias. `IDE_Tipo` (0 Ideia, 1 Melhoria, 2 Relato de erro), `IDE_Status` (0..6), `IDE_Titulo`, campos por tipo, `IDE_FuncaoRelacionadaId`, `IDE_PrioridadeSugerida` e `IDE_PrioridadeOficial` (**anulável** — só o gestor define), `IDE_CriadorId`, `IDE_TarefaVinculadaId` (**fonte de verdade do vínculo com o Kanban**), arquivamento lógico, `IDE_Origem`/`IDE_DemoChave` (seed de demonstração) e auditoria. |
| `MensagensIdeia` / `MensagemIdeia` | `tbmensagemideia` | `Id` (`MSG_Id`) | Gerida (script próprio) | Conversa sobre a solicitação. **Append-only**: sem edição e sem remoção. `MSG_Tipo` (0 Comentário, 1 Resposta a esclarecimento). |
| `HistoricoStatusIdeia` / `HistoricoStatusIdeia` | `tbhistoricostatusideia` | `Id` (`HIS_Id`) | Gerida (script próprio) | **Append-only**. Concentra as mudanças de status **e** as decisões administrativas numa tabela só, para não duplicar o mesmo evento. `HIS_StatusAnterior` anulável (linha de criação), `HIS_Justificativa`, `HIS_Decisao` (rótulo para a timeline). Sobrevive a reabertura e ao arquivamento. |
| `AnalisesIdeia` / `AnaliseIdeia` | `tbanaliseideia` | `Id` (`AID_Id`) | Gerida (script próprio) | Execuções da análise por IA. `AID_Estado` (0 Concluída … 6 Falha do provedor) distingue sucesso de cada modo de falha; `AID_Resultado` é **anulável** — análise que falhou não tem texto para exibir. |
| `EspecificacoesIdeia` / `EspecificacaoIdeia` | `tbespecificacaoideia` | `Id` (`ESP_Id`) | Gerida (script próprio) | Versões da especificação técnica. `UNIQUE (ESP_IdeiaId, ESP_Versao)`: revisão cria a versão seguinte e preserva a anterior. `ESP_Estado` (0 Rascunho, 1 Aprovada, 2 Obsoleta), `ESP_ExigeRevisao`, `ESP_ContextoSnapshot`/`Hash`. **Aprovar a especificação não aprova a solicitação.** |
| `Gestores` / `Gestor` | `tbgestor` | `Id` (`GES_Id`) | Gerida (script próprio) | Quem administra o módulo de Ideias. `GES_FuncaoId` **anulável** = gestor geral (administra todas as funções). Índice único filtrado garante no máximo um gestor geral ativo. Substitui a ideia de `Ideias:GestorId` em configuração. |
| `Cargos` / `Cargo` | `tbcargo` | `Id` (`CARGO_ID`) | Gerida (script próprio) | Cargo RBAC: `CARGO_Codigo` `varchar(40)` **único** (`UQ_tbcargo_Codigo`), `CARGO_Descricao` `varchar(100)`, `CARGO_EscopoPadrao` `tinyint` (default 1 = `Proprio`), **`CARGO_Nivel` `tinyint`** (default 1; Master 4 > Tech Lead 3 > comum 2 > Visualizador 1; CLR **`byte`** em `Cargo.Nivel` — não `int`, porque o `SqlDataReader.GetInt32` não converte `tinyint`), **`CARGO_FuncaoId` `int NULL`** (setor do cargo; FK para `tbfuncao`; `NULL` nos globais), `CARGO_Ativo` e auditoria. Eixo **diferente** de `tboperador.FUNCAO_ID` (setor). Os **11** códigos semeados são os nove da especificação + `VISUALIZADOR` + `TECH_LEAD_ADM`; só `GESTOR_MASTER` tem comportamento próprio no motor (bypass). |
| `PermissoesRecurso` / `PermissaoRecurso` | `tbpermissao_recurso` | `Id` (`PER_Id`) | Gerida (script próprio) | Catálogo de recursos protegidos: `PER_Chave` `varchar(150)` **única** (`UQ_tbpermissao_recurso_Chave`) no formato `aba.<área>.<tela>` ou `api.<controller>.<acao>`, `PER_Tipo` `tinyint` (`Aba`=1, `Api`=2, `Relatorio`=3, `Tabela`=4), `PER_Rota`, `PER_Controller`, `PER_Descricao` e `PER_Ativo`. |

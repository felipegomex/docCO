---
id: 06-5-modelos-dbcontext-e-relacionamentos.01
titulo: "5.1 DbSets atuais"
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
tokens_estimados: 1371
ficha: docs/modulos/database-explorer.md
---
> Parte de [5. Modelos, DbContext e relacionamentos](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### 5.1 DbSets atuais

| Domínio | DbSets/tabelas |
|---|---|
| Identidade | `Operadores`/`tboperador` (com `CARGO_ID`), `Funcoes`/`tbfuncao`, `RefreshTokens`/`tbrefreshtoken`, `AuditoriaAcessos`/`tbauditoriaacesso` |
| RBAC | `Cargos`/`tbcargo`, `PermissoesRecurso`/`tbpermissao_recurso`, `PermissoesCargo`/`tbperm_cargo`, `PermissoesOperador`/`tbperm_operador` |
| Cadastros | `TiposProjeto`/`tbtipoprojeto`, `ColunasKanban`/`tbcolunakanban` |
| Projetos | `Projetos`/`tbprojeto`, `ProjetoEtapas`/`tbprojetoetapa`, `ProjetoEtapaChecklists`/`tbprojetoetapachecklist`, `ProjetoEtapaDocumentos`/`tbprojetoetapadocumento`, `ProjetoEtapaHistoricos`/`tbprojetoetahistorico`, `ProjetoEtapaComentarios`/`tbprojetoetapacomentario` |
| Tarefas | `Tarefas`/`tbtarefa`, `ComentariosTarefa`/`tbcomentariotarefa`, `TarefaResponsaveis`/`tbtarefareponsavel`, `TarefaChamados`/`tbtarefachamado`, `TarefaApontamentos`/`tbtarefaapontamento` |
| Kanban Geral | `EtapasFuncao`/`tbfuncaoetapa`, `Bloqueios`/`tbbloqueio` (o eixo é `Funcoes`/`tbfuncao`, em Identidade) |
| Agenda | `Agenda`/`tbagenda`, `TiposEvento`/`tbtipoevento`, `AgendaParticipantes`/`tbagendaparticipante` |
| Auditoria | `AuditoriaImplantacao`/`tbauditoriaimplantacao` |
| Acessos Empresas | `ClientesAcesso`/`tbclienteacesso`, `AcessosRdp`/`tbacessordp`, `AcessosVpn`/`tbacessovpn`, `AcessosAnyDesk`/`tbacessoanydesk`, `AcessosBanco`/`tbacessobanco`, `AcessosActyonWeb`/`tbacessoactyonweb` |
| Dashboard Integrado | `Leads`/`tblead`, `LeadsContato`/`tbleadcontato`, `Oportunidades`/`tboportunidade`, `Orcamentos`/`tborcamento`, `Custos`/`tbcusto`, `ProdutosLegado`/`tbproduto` (leitura, fora das migrations) |
| Ideias (implementado) | `Ideias`/`tbideia`, `MensagensIdeia`/`tbmensagemideia`, `HistoricoStatusIdeia`/`tbhistoricostatusideia`, `AnalisesIdeia`/`tbanaliseideia`, `EspecificacoesIdeia`/`tbespecificacaoideia`, `Gestores`/`tbgestor` |
| Documentação (docs-api) | `Documentos`/`tbdoc_doc` |
| **Atendimento WhatsApp** | `Numeros`/`tbwhatsapp_numero`, `Contatos`/`tbwhatsapp_contato`, `Conversas`/`tbwhatsapp_conversa`, `Atendimentos`/`tbwhatsapp_atendimento`, `Mensagens`/`tbwhatsapp_mensagem`, `Eventos`/`tbwhatsapp_evento`, `WhatsAppWebhookFila`/`tbwhatsapp_webhook_fila`, `Parametros`/`tbwhatsapp_parametro`, `RespostasRapidas`/`tbwhatsapp_resposta_rapida`, `SlaConfigs`/`tbwhatsapp_sla_config`, `SlaHorarios`/`tbwhatsapp_sla_horario`, `SlaHistoricos`/`tbwhatsapp_sla_historico`, `RetencaoConfigs`/`tbwhatsapp_retencao_config`, `RetencaoExecucoes`/`tbwhatsapp_retencao_execucao`, `Grupos`/`tbwhatsapp_grupo`, `Participantes`/`tbwhatsapp_participante`, `Auditorias`/`tbwhatsapp_auditoria`, `IaConfigs`/`tbwhatsapp_ia_config`, `Etiquetas`/`tbwhatsapp_etiqueta`, `AtendimentoEtiquetas`/`tbwhatsapp_atendimento_etiqueta` |
| Legadas | `ChamadosLegado`/`tbchamado`, `FuncionariosLegado`/`tbfuncionario`, `ClientesLegado`/`tbcliente`, `ClientesGrupo`/`tbcliente_grupo`, `ContratosLegado`/`tbcontrato` |

São **78 `DbSet`**. O módulo de WhatsApp tem **20 conjuntos** (geridos no modelo, mas **sem classe de migração** — entram pelos scripts `WhatsApp_Idempotente.sql`, `WhatsApp_Fase0/1/3/4/5/6/7_Idempotente.sql` e `WhatsApp_SlaAviso_Idempotente.sql`), o RBAC tem **4** (também **sem classe de migração** — entram por `Permissoes_Idempotente.sql`) e há 6 legados somente leitura (`tbproduto` incluso); os demais conjuntos são gerenciados pelas migrações e pelas entidades mapeadas. Nomes de tabela seguem o padrão `tb*` (regra, colunas e regras de PK/FK em [04-ESTRUTURA-DADOS.md](../../../arquitetura/banco-dados.md)). **A Fase 09 não alterou esse número**: tempo real e digitação não criam `DbSet`, tabela, coluna, migration ou script. As 6 tabelas do módulo de Ideias são geridas no modelo, mas **não têm classe de migração** — em produção entram pelo script `Ideias_Idempotente.sql`; `tbdoc_doc` é gerida pela migration `20261019203821_TbDocDocumentos` (escrita à mão, sem `.Designer.cs`).

⚠️ **`ClienteLegado`/`tbcliente` e `ClienteAcesso`/`tbclienteacesso` são coisas diferentes.** A primeira é o espelho somente leitura da base legada, usada por Implantação e pelo Dashboard Administrativo. A segunda é a entidade nova e gravável do módulo Acessos Empresas, com as próprias flags e os próprios acessos. Elas não compartilham FK, id nem ciclo de vida.

O `AcessoBase` abstrato ([`AcessoBase.cs`](../../../../backend/Central_BackEnd/Models/Acessos/AcessoBase.cs)) não tem atributos `[Column]`: as colunas reais ficam nas 5 derivadas, cada uma com o prefixo da sua tabela (`RDP_`, `VPN_`, `ADK_`, `BCO_`, `AWB_`). **Exceção:** `Usuario` tem `[MaxLength(100)]` sem `[Column]`; as 4 derivadas com senha a redeclaram com prefixo, e `AcessoActyonWeb` não — herdando-a, o EF mapeia para a coluna `Usuario` **sem prefixo**, que é legítima. Ver a armadilha de `SenhaCifrada`/`TemSenha` em [`04-ESTRUTURA-DADOS.md`](../../../arquitetura/banco-dados.md).

A entidade `Cliente` e a tabela `IMPL_Cliente` foram removidas: `ProjetoService.ListarClientesAsync` usa `ClientesLegado`, e a fonte efetiva do dropdown é `tbcliente`. A função passou a mapear a legada `tbfuncao` (antes `CC_Funcao`, hoje descartada).


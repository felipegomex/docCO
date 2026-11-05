---
id: regras-negocio.01
titulo: "Índice para busca"
tipo: referencia
area: negocio
sistema: central
tags: [regras, negocio, permissoes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 2153
ficha: docs/modulos/README.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## Índice para busca

| Termo | Fonte primária |
|---|---|
| autenticação, JWT, refresh | [AuthController.cs](../../../backend/Central_BackEnd/Controllers/AuthController.cs), [AuthService.cs](../../../backend/Central_BackEnd/Services/AuthService.cs), [token-storage.service.ts](../../../frontend/src/app/core/services/token-storage.service.ts) |
| prazo e atraso (tarefa, projeto, etapa) | [RegrasPrazo.cs](../../../backend/Central_BackEnd/Services/Shared/RegrasPrazo.cs), [TarefaAtrasoExtensions.cs](../../../backend/Central_BackEnd/Services/Implantacao/TarefaAtrasoExtensions.cs), [ProjetoEtapaService.cs](../../../backend/Central_BackEnd/Services/Implantacao/ProjetoEtapaService.cs) |
| perfil do operador (`PERFIL_ID`) | [PerfilOperador.cs](../../../backend/Central_BackEnd/Services/Shared/PerfilOperador.cs), [AuthService.cs](../../../backend/Central_BackEnd/Services/AuthService.cs), [FuncaoAuthorizationService.cs](../../../backend/Central_BackEnd/Services/Implantacao/FuncaoAuthorizationService.cs) |
| bloqueio, WIP, arquivamento | [Tarefa.cs](../../../backend/Central_BackEnd/Models/Implantacao/Tarefa.cs), [TarefaService.cs](../../../backend/Central_BackEnd/Services/Implantacao/TarefaService.cs) |
| apontamentos e horas | [TarefaApontamento.cs](../../../backend/Central_BackEnd/Models/Implantacao/TarefaApontamento.cs), [TarefaService.cs](../../../backend/Central_BackEnd/Services/Implantacao/TarefaService.cs) |
| data de conclusão real da tarefa (do servidor) | [TarefaDtos.cs](../../../backend/Central_BackEnd/Dtos/Implantacao/TarefaDtos.cs), [TarefaService.cs](../../../backend/Central_BackEnd/Services/Implantacao/TarefaService.cs) |
| sincronização tarefa–agenda | [TarefaService.cs](../../../backend/Central_BackEnd/Services/Implantacao/TarefaService.cs), [AgendaService.cs](../../../backend/Central_BackEnd/Services/Implantacao/AgendaService.cs) |
| etapas e jornada | [ProjetoEtapaService.cs](../../../backend/Central_BackEnd/Services/Implantacao/ProjetoEtapaService.cs), [ProjetoEtapa.cs](../../../backend/Central_BackEnd/Models/Implantacao/ProjetoEtapa.cs), [TarefaService.cs](../../../backend/Central_BackEnd/Services/Implantacao/TarefaService.cs) |
| propriedade, perfis e permissões | [AuthService.cs](../../../backend/Central_BackEnd/Services/AuthService.cs), [TarefaService.cs](../../../backend/Central_BackEnd/Services/Implantacao/TarefaService.cs), [AgendaController.cs](../../../backend/Central_BackEnd/Controllers/AgendaController.cs), [auth.service.ts](../../../frontend/src/app/core/services/auth.service.ts) |
| conflitos de agenda | [AgendaService.cs](../../../backend/Central_BackEnd/Services/Implantacao/AgendaService.cs), [ConflictException.cs](../../../backend/Central_BackEnd/Exceptions/ConflictException.cs) |
| Dashboard Integrado (funil, custos, orçamentos) | [ConstantesDashboard.cs](../../../backend/Central_BackEnd/Models/Dashboard/ConstantesDashboard.cs), [Services/Dashboard](../../../backend/Central_BackEnd/Services/Dashboard) |
| regra de contrato vigente do legado (`ATIVO` / `SITUACAO`) | [TbContratoLegado.cs](../../../backend/Central_BackEnd/Models/Dashboard/TbContratoLegado.cs) (`EhAtivo`, `ValorMensal`) |
| faturamento não liberado (interruptor de exibição) | [ConstantesDashboard.cs](../../../backend/Central_BackEnd/Models/Dashboard/ConstantesDashboard.cs) (`FlagsDashboard.FaturamentoLiberado`) |
| "cliente viu o contato" (flag manual) | [TbLead.cs](../../../backend/Central_BackEnd/Models/Dashboard/TbLead.cs), [ComercialService.cs](../../../backend/Central_BackEnd/Services/Dashboard/ComercialService.cs) (`DefinirContatoVisualizadoAsync`) |
| restrição permanente SQL 2008 (sem `OPENJSON`) | [DashboardService.cs](../../../backend/Central_BackEnd/Services/Implantacao/DashboardService.cs), [TarefaService.cs](../../../backend/Central_BackEnd/Services/Implantacao/TarefaService.cs), [ComercialService.cs](../../../backend/Central_BackEnd/Services/Dashboard/ComercialService.cs) |
| código global PRJ | [ProjetoService.cs](../../../backend/Central_BackEnd/Services/Implantacao/ProjetoService.cs), [Projeto.cs](../../../backend/Central_BackEnd/Models/Implantacao/Projeto.cs) |
| dados sensíveis e fontes externas | [AcessosController.cs](../../../backend/Central_BackEnd/Controllers/AcessosController.cs), [AcessosService.cs](../../../backend/Central_BackEnd/Services/Acessos/AcessosService.cs), [Cifrador.cs](../../../backend/Central_BackEnd/Services/Seguros/Cifrador.cs), [DatabaseConnectionService.cs](../../../backend/Central_BackEnd/Services/Database/DatabaseConnectionService.cs) |
| módulo Acessos Empresas (3 níveis, 5 tipos, soft delete, flags derivadas) | [AcessosService.cs](../../../backend/Central_BackEnd/Services/Acessos/AcessosService.cs), [ValidadoresAcesso.cs](../../../backend/Central_BackEnd/Services/Acessos/ValidadoresAcesso.cs), [AcessoBase.cs](../../../backend/Central_BackEnd/Models/Acessos/AcessoBase.cs) |
| módulo Ideias (gestor por setor via `tbgestor`, leitor do CIAA, exclusão administrativa) | [IdeiasPermissaoService.cs](../../../backend/Central_BackEnd/Services/Ideias/IdeiasPermissaoService.cs), [GestorService.cs](../../../backend/Central_BackEnd/Services/Gestores/GestorService.cs), [IdeiaController.cs](../../../backend/Central_BackEnd/Controllers/IdeiasController.cs) |
| Kanban Geral (função, matriz de permissão por função, bloqueio como atributo, etapas fixas) | [FuncaoAuthorizationService.cs](../../../backend/Central_BackEnd/Services/Implantacao/FuncaoAuthorizationService.cs), [FuncaoService.cs](../../../backend/Central_BackEnd/Services/Implantacao/FuncaoService.cs), [EtapaFuncaoService.cs](../../../backend/Central_BackEnd/Services/Implantacao/EtapaFuncaoService.cs), [BloqueioService.cs](../../../backend/Central_BackEnd/Services/Implantacao/BloqueioService.cs), [KanbanBoardService.cs](../../../backend/Central_BackEnd/Services/Implantacao/KanbanBoardService.cs), [Bloqueio.cs](../../../backend/Central_BackEnd/Models/Implantacao/Bloqueio.cs) |
| cifra das credenciais e sua liberação | [Cifrador.cs](../../../backend/Central_BackEnd/Services/Seguros/Cifrador.cs), [Program.cs](../../../backend/Central_BackEnd/Program.cs) |
| trilha de auditoria do módulo | [AuditoriaFila.cs](../../../backend/Central_BackEnd/Services/Auditoria/AuditoriaFila.cs), [ProcessadorAuditoria.cs](../../../backend/Central_BackEnd/Services/Auditoria/ProcessadorAuditoria.cs), [ConstantesAcessos.cs](../../../backend/Central_BackEnd/Models/Acessos/ConstantesAcessos.cs) |
| módulo de atendimento WhatsApp (atribuição pela primeira resposta, transferência, devolução, resolução, reabertura, finalização automática, "Aguardando cliente", nota interna, métricas) | [AtendimentoService.cs](../../../backend/Central_BackEnd/Services/WhatsApp/AtendimentoService.cs), [WhatsAppFinalizacaoAutomaticaService.cs](../../../backend/Central_BackEnd/Services/WhatsApp/WhatsAppFinalizacaoAutomaticaService.cs), [WhatsAppFinalizacaoAutomaticaJob.cs](../../../backend/Central_BackEnd/Services/WhatsApp/WhatsAppFinalizacaoAutomaticaJob.cs), [WhatsAppEnums.cs](../../../backend/Central_BackEnd/Models/WhatsApp/WhatsAppEnums.cs), [WhatsAppWebhookService.cs](../../../backend/Central_BackEnd/Services/WhatsApp/WhatsAppWebhookService.cs), [WhatsAppMetricasService.cs](../../../backend/Central_BackEnd/Services/WhatsApp/WhatsAppMetricasService.cs) |
| RBAC (cargo, recurso, ação, escopo, precedência, Master, policies, hierarquia de cargos e edição da matriz) | [PermissaoService.cs](../../../backend/Central_BackEnd/Services/Permissoes/PermissaoService.cs), [IPermissaoService.cs](../../../backend/Central_BackEnd/Services/Permissoes/IPermissaoService.cs), [AcaoPermissao.cs](../../../backend/Central_BackEnd/Models/Permissoes/AcaoPermissao.cs), [PermissaoPolicyProvider.cs](../../../backend/Central_BackEnd/Authorization/PermissaoPolicyProvider.cs), [PermissoesController.cs](../../../backend/Central_BackEnd/Controllers/PermissoesController.cs) |
| gestão de operadores (Master/Tech Lead, sem DELETE, senha, e-mail, ponte `SE_ADMIN`) | [OperadorAdminService.cs](../../../backend/Central_BackEnd/Services/Shared/OperadorAdminService.cs), [OperadoresController.cs](../../../backend/Central_BackEnd/Controllers/OperadoresController.cs), [OperadorDtos.cs](../../../backend/Central_BackEnd/Dtos/Shared/OperadorDtos.cs) |


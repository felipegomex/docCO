---
id: sec.01-indice-para-busca
titulo: "Índice para busca"
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
tokens_estimados: 1675
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Índice para busca

| Tema | Fonte |
|---|---|
| entidades e DbSets | [AppDbContext.cs](../../../backend/Central_BackEnd/Data/AppDbContext.cs) |
| esquema final | [AppDbContextModelSnapshot.cs](../../../backend/Central_BackEnd/Migrations/AppDbContextModelSnapshot.cs) |
| histórico do esquema | [AppDbContextModelSnapshot.cs](../../../backend/Central_BackEnd/Migrations/AppDbContextModelSnapshot.cs) e os 33 arquivos de migração, o mais recente [20261021003310_ChatConhecimento.cs](../../../backend/Central_BackEnd/Migrations/20261021003310_ChatConhecimento.cs) |
| autenticação e refresh | [Operador.cs](../../../backend/Central_BackEnd/Models/Operador.cs), [RefreshToken.cs](../../../backend/Central_BackEnd/Models/RefreshToken.cs) |
| projetos, tarefas e etapas | [Projeto.cs](../../../backend/Central_BackEnd/Models/Implantacao/Projeto.cs), [Tarefa.cs](../../../backend/Central_BackEnd/Models/Implantacao/Tarefa.cs), [ProjetoEtapa.cs](../../../backend/Central_BackEnd/Models/Implantacao/ProjetoEtapa.cs) |
| agenda | [AgendaItem.cs](../../../backend/Central_BackEnd/Models/Implantacao/AgendaItem.cs), [AgendaParticipante.cs](../../../backend/Central_BackEnd/Models/Implantacao/AgendaParticipante.cs) |
| legadas | [ClienteLegado.cs](../../../backend/Central_BackEnd/Models/Implantacao/ClienteLegado.cs), [ClienteGrupoLegado.cs](../../../backend/Central_BackEnd/Models/Implantacao/ClienteGrupoLegado.cs), [ChamadoLegado.cs](../../../backend/Central_BackEnd/Models/Implantacao/ChamadoLegado.cs), [FuncionarioLegado.cs](../../../backend/Central_BackEnd/Models/Implantacao/FuncionarioLegado.cs), [TbContratoLegado.cs](../../../backend/Central_BackEnd/Models/Dashboard/TbContratoLegado.cs) |
| Dashboard Integrado | [Models/Dashboard](../../../backend/Central_BackEnd/Models/Dashboard) e as migrations [20260928144824_DashboardTabelas.cs](../../../backend/Central_BackEnd/Migrations/20260928144824_DashboardTabelas.cs) e [20260928182047_LeadContatoVisualizado.cs](../../../backend/Central_BackEnd/Migrations/20260928182047_LeadContatoVisualizado.cs) (scripts de produção em [Sql](../../../backend/Central_BackEnd/Migrations/Sql)) |
| Acessos Empresas | [Models/Acessos](../../../backend/Central_BackEnd/Models/Acessos), [AppDbContext.cs](../../../backend/Central_BackEnd/Data/AppDbContext.cs), [AcessosEmpresas.cs](../../../backend/Central_BackEnd/Migrations/20260928212856_AcessosEmpresas.cs), [AcessosEmpresas_Idempotente.sql](../../../backend/Central_BackEnd/Migrations/Sql/AcessosEmpresas_Idempotente.sql), [Cifrador.cs](../../../backend/Central_BackEnd/Services/Seguros/Cifrador.cs) |
| Kanban Geral | [Funcao.cs](../../../backend/Central_BackEnd/Models/Funcao.cs), [ColunaKanban.cs](../../../backend/Central_BackEnd/Models/Implantacao/ColunaKanban.cs), [ColunaFuncao.cs](../../../backend/Central_BackEnd/Models/Implantacao/ColunaFuncao.cs), [EtapaFuncao.cs](../../../backend/Central_BackEnd/Models/Implantacao/EtapaFuncao.cs), [Bloqueio.cs](../../../backend/Central_BackEnd/Models/Implantacao/Bloqueio.cs), [20260929034944_KanbanGeral_SetoresEtapasBloqueios.cs](../../../backend/Central_BackEnd/Migrations/20260929034944_KanbanGeral_SetoresEtapasBloqueios.cs), [KanbanGeral_Idempotente.sql](../../../backend/Central_BackEnd/Migrations/Sql/KanbanGeral_Idempotente.sql), [20260929221556_SetorParaFuncao.cs](../../../backend/Central_BackEnd/Migrations/20260929221556_SetorParaFuncao.cs), [SetorParaFuncao_Idempotente.sql](../../../backend/Central_BackEnd/Migrations/Sql/SetorParaFuncao_Idempotente.sql), [SetorParaFuncao_Consolidar_Idempotente.sql](../../../backend/Central_BackEnd/Migrations/Sql/SetorParaFuncao_Consolidar_Idempotente.sql), [20261020205521_ColunaFuncao.cs](../../../backend/Central_BackEnd/Migrations/20261020205521_ColunaFuncao.cs), [ColunasCompartilhadas_Idempotente.sql](../../../backend/Central_BackEnd/Migrations/Sql/ColunasCompartilhadas_Idempotente.sql), [FuncaoFinanceiro_Idempotente.sql](../../../backend/Central_BackEnd/Migrations/Sql/FuncaoFinanceiro_Idempotente.sql) |
| RBAC (cargo, catálogo de recursos e exceções) | [Models/Permissoes](../../../backend/Central_BackEnd/Models/Permissoes), [Operador.cs](../../../backend/Central_BackEnd/Models/Operador.cs) (`CARGO_ID`), [AppDbContext.cs](../../../backend/Central_BackEnd/Data/AppDbContext.cs) (`Cargos`, `PermissoesRecurso`, `PermissoesCargo`, `PermissoesOperador`), [Permissoes_Idempotente.sql](../../../backend/Central_BackEnd/Migrations/Sql/Permissoes_Idempotente.sql), [Permissoes_CargaInicial_Idempotente.sql](../../../backend/Central_BackEnd/Migrations/Sql/Permissoes_CargaInicial_Idempotente.sql), [Permissoes_SeedOperadores_Idempotente.sql](../../../backend/Central_BackEnd/Migrations/Sql/Permissoes_SeedOperadores_Idempotente.sql) |
| Database Explorer | [DatabaseConnectionService.cs](../../../backend/Central_BackEnd/Services/Database/DatabaseConnectionService.cs), [DatabaseMetadataService.cs](../../../backend/Central_BackEnd/Services/Database/DatabaseMetadataService.cs) |
| padrão de nomes `tb*` | [20260926010406_PadraoTabelasTb.cs](../../../backend/Central_BackEnd/Migrations/20260926010406_PadraoTabelasTb.cs) e [20260926022906_RenomeiaIndicesPkTb.cs](../../../backend/Central_BackEnd/Migrations/20260926022906_RenomeiaIndicesPkTb.cs) |
| conferência antes/depois | [conferencia-padrao-tb.sql](../../../scripts/db/conferencia-padrao-tb.sql) |
| aplicação em produção | [renomear-tabelas-tb-idempotente.sql](../../../scripts/db/renomear-tabelas-tb-idempotente.sql) (tabelas, PKs, índices e AK) |
| **Atendimento WhatsApp** | [`Models/WhatsApp`](../../../backend/Central_BackEnd/Models/WhatsApp), [`AppDbContext.cs`](../../../backend/Central_BackEnd/Data/AppDbContext.cs), [`Migrations/Sql/WhatsApp_Idempotente.sql`](../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Idempotente.sql) e os scripts de fase `WhatsApp_Fase0/1/3/4/5_Idempotente.sql`, mais `WhatsApp_Fase6_Idempotente.sql` (vínculo do contato com o ERP, novo ciclo), `WhatsApp_SlaAviso_Idempotente.sql` (percentual de aviso do SLA), `WhatsApp_Fase7_Idempotente.sql` (etiquetas, investigação e observação — Fase 06), `WhatsApp_Visto_Idempotente.sql` (F6: CHECK do status 0..6) e `WhatsApp_Fase8_Idempotente.sql` (Fases 0/1 do plano: protocolo computado, foto/telefone do contato e parâmetros) (**sem migration EF** — o DDL vive nos scripts) |


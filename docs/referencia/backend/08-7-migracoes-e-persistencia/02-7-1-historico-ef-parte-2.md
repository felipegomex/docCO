---
id: 08-7-migracoes-e-persistencia.02
titulo: "7.1 Histórico EF — parte 2"
tipo: referencia
area: backend
sistema: central
tags: [backend, migrations, persistencia]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Migrations
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 638
ficha: docs/modulos/database-explorer.md
---
> Parte de [7. Migrações e persistência](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### 7.1 Histórico EF — parte 2

⚠️ **Antes de rodar `dotnet ef database update`:** `20260914144751_AgendaConflitoHorarios`, `20260917202045_TarefaProjetoOpcional` e `20260920185734_TarefaProjetoEtapaId` já estavam fisicamente aplicadas via scripts manuais e foram carimbadas à mão em `__EFMigrationsHistory`. O banco também tem o registro órfão `20260910183240_AgendaGeral`, cujo arquivo `.cs` não existe (o EF ignora). Em produção, `scripts/deploy/deploy.ps1` não aplica migrations: usar o script único `scripts/db/renomear-tabelas-tb-idempotente.sql` (tabelas, PKs, índices e unique constraint, equivalente a `PadraoTabelasTb` + `RenomeiaIndicesPkTb`), `scripts/db/conferencia-padrao-tb.sql` para auditar, `Migrations/Sql/DashboardTabelas_Idempotente.sql` para as tabelas do Dashboard Integrado, `Migrations/Sql/LeadContatoVisualizado_Idempotente.sql` para a coluna do flag de visualização do contato, `Migrations/Sql/ComercialFunilProdutos_Idempotente.sql` para arquivamento de leads, nome do contato e oportunidades por produto (antes do publish que levar o funil comercial), `Migrations/Sql/AcessosEmpresas_Idempotente.sql` para o módulo Acessos, `Migrations/Sql/AcessosGrupoClienteErp_Idempotente.sql` para a troca de grupos/ERP do Acessos (imediatamente antes do publish), `Migrations/Sql/KanbanGeral_Idempotente.sql` para setores/etapas/bloqueios, `Migrations/Sql/SetorParaFuncao_Idempotente.sql` + `Migrations/Sql/SetorParaFuncao_Consolidar_Idempotente.sql` para a conversão Setor → Função, `Migrations/Sql/FuncaoFinanceiro_Idempotente.sql` para o cadastro da função 4 (na base de 2026-10-21, `Executivo(a) Comercial`), `Migrations/Sql/ColunasCompartilhadas_Idempotente.sql` para o catálogo compartilhado de colunas e a visão por setor (**antes do publish** do código novo) e `Migrations/Sql/TbDocDoc_Idempotente.sql` para `tbdoc_doc` (espelho da documentação — antes do publish do backend com `/api/v1/docs`) e `Migrations/Sql/WhatsApp_Idempotente.sql` para as 7 tabelas `tbwhatsapp_*` do módulo de atendimento WhatsApp (antes do publish do backend com `/api/v1/whatsapp`; ainda **não aplicado**). Detalhes em [04-ESTRUTURA-DADOS.md](../../../arquitetura/banco-dados.md) e [10-DEPLOY.md](../../../operacao/deploy.md).


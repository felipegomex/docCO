---
id: sec.12-11-incertezas-remanescentes
titulo: "11. Incertezas remanescentes"
tipo: referencia
area: backend
sistema: central
tags: [backend, aspnet, controllers, services]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 873
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/backend.md](../backend.md).

# 11. Incertezas remanescentes

- Não foi executado build, suíte backend, smoke ou aplicação das migrações; o documento representa o código atual, não o estado de um banco publicado.
- Migration e SQL do chat **aplicados em HML e PROD em 2026-10-30**; chat publicado (2026-10-28); VM, OAuth e segredos configurados nos dois ambientes. A existência dos fluxos no código não substitui a verificação fim-a-fim em ambiente.
- `equipe` em Tarefas/Dashboard, `diasRetro` no dashboard administrativo e campos legados de Cliente não são aplicados integralmente pelos serviços atuais.
- Não foi possível inferir se a divergência de `APP_VERSION` do frontend tem efeito no versionamento do backend; ela está registrada em [`06-COMPONENTES-FRONTEND.md`](../frontend.md).
- A existência dos scripts SQL auxiliares não comprova que foram aplicados em algum ambiente. A migration `20260928212856_AcessosEmpresas` **é** a exceção declarada: foi aplicada em `dbBUSINESS_HML`, com backup prévio.
- A divergência de schema em `tbacessoactyonweb` **foi encerrada**: `SenhaCifrada` foi removida pela migration `20260929013916_AcessosEmpresas_DropColunasOrfasActyonWeb` (aplicada em `dbBUSINESS_HML`) e o snapshot foi regenerado, estando em sincronia com o banco. `Usuario` **não** era órfã: é mapeada por herança de `AcessoBase` (que a declara com `[MaxLength(100)]` mas sem `[Column]`) e foi preservada. Ver [`04-ESTRUTURA-DADOS.md`](../../arquitetura/banco-dados.md).
- Não foi executado build, `dotnet test`, suíte frontend, smoke ou aplicação das migrações nesta revisão; o documento representa o código atual, não o estado de um banco publicado.
- **RBAC:** o schema e a carga inicial foram aplicados em `dbBUSINESS_HML` em 2026-10-21 (4 tabelas, `CARGO_ID`, 10 cargos, 8 `GESTOR_MASTER`, 1 `TECH_LEAD_IMPLANTACAO`, 32 sem cargo), mas as entidades **não estão no snapshot do EF** e não há classe de migração. Em 2026-10-27 entraram `CARGO_Nivel`/`CARGO_FuncaoId` e `TECH_LEAD_ADM` (11 cargos) e a matriz ganhou a hierarquia. Em outro banco, os cinco scripts `Migrations/Sql/Permissoes_*.sql` precisam ser aplicados antes do publish (`Permissoes_Idempotente.sql` antes de `Permissoes_Matriz_Idempotente.sql`); o preflight cobre a ausência.
- **Módulo de atendimento WhatsApp:** o código existe e está completo, e o canal foi **publicado na homologação em 2026-10-21** — schema aplicado em `dbBUSINESS_HML`, gateway OpenWA v0.25.0 **no ar** na VM Ubuntu `192.168.2.253` e webhook validado ponta a ponta. O módulo é ligado por **variável de ambiente** no servidor, e o `deploy.ps1` **preserva** o `web.config` que contém chaves `WhatsApp__` (não é mais preciso reaplicar o bloco após cada deploy). **Piloto em operação**: número pareado (sessão `central-wa-7`, `ready`, telefone `558592644863`) e Kanban recebendo mensagens reais — a primeira resposta assumiu o atendimento, houve resolução e reabertura. **A produção (`.254`) foi publicada em 2026-10-26**: lá, a existência das rotas `/api/v1/whatsapp` ainda **não** significa canal com mensagens circulando.


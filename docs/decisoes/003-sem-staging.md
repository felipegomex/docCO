---
titulo: "ADR 003 — Produção sem staging"
status: "aceito"
data: "2026-10-18"
---

> Data de registro. A data da decisão não está documentada nas fontes deste ADR.

# ADR 003 — Produção sem staging

## Contexto

O projeto não tem ambiente de staging: os bancos usados são `dbBUSINESS_HML` (SQL Server `192.168.2.154`) para HML e `dbBUSINESS` para PROD (do IIS PROD `192.168.2.254`; desde 2026-10-14 o Central usa banco próprio `dbCENTRAL_OPERACAO`, restaurado por `BACKUP COPY_ONLY` de `dbBUSINESS`, com zero escrita no legado). `dbBUSINESS_HML` tem `compatibility_level 100` (SQL Server 2008) e **é o banco de produção** — mudanças de schema nele são mudanças de produção de primeira vez. Além disso, `scripts/deploy/deploy.ps1` **não aplica migrations** e o backend não executa `Database.Migrate()` no startup: schema e binário podem divergir silenciosamente.

## Decisão

Publicar diretamente em produção **sem ambiente intermediário de staging**, adotando em compensação um protocolo de segurança por migração:

1. **Backup prévio obrigatório** do banco antes de qualquer script com `DROP` (e backup IIS com `-Backup 1`, obrigatório em PROD, onde o script também exige digitar `PROD` para confirmar).
2. **Migrations preferencialmente aditivas**, aplicadas manualmente via script idempotente revisado à mão de `backend/Central_BackEnd/Migrations/Sql` — **antes** do publish do binário que depende deles.
3. **Revisão do SQL com o responsável pelo banco** antes de aplicar; confirmação pós-aplicação consultando `__EFMigrationsHistory` e o schema real (`sys.tables`, `COL_LENGTH`).
4. Janelas de incompatibilidade declaradas por migração (ex.: `AcessosGrupoClienteErp` e `RemoveIdentidadeLocalAcessos` exigem script e publish em minutos; `SetorParaFuncao` tem ordem Fase 0 → deploy → Fase 3).
5. `dotnet ef migrations add` **não é rodado** neste repositório enquanto o snapshot do EF diverge do banco (migration de sincronia pendente) — qualquer migration gerada precisa de revisão linha a linha.

## Consequências

- Toda migration é mudança de schema em **produção de primeira vez** — backup e revisão são passos obrigatórios, não opcionais.
- Rollback de schema não é automático: migrações com `DROP` (ex.: `AcessosEmpresas_DropColunasOrfasActyonWeb`, Kanban Geral, Fase 3 de `SetorParaFuncao`) não restauram dados por `Down`; reverter é restore de backup do banco, com plano do DBA.
- O protocolo já foi seguido em produção: `AcessosEmpresas` (backup `CentralConhecimento_pre_AcessosEmpresas.bak`, 21.856 MB), `SetorParaFuncao` (2026-10-01, com backups `bkp_20261001_pre_setorfuncao_*`), `AcessosGrupoClienteErp` (2026-09-30), `ColunasPorFuncao` (2026-10-02) e o módulo de Ideias (2026-10-11, 100% aditivo, sem backup por decisão do dono do projeto).

## Fonte

- `docs/operacao/deploy.md` — seção 4 (SQL Homolog: nota "não existe ambiente de staging"), seção 7 (migrations antes da publicação), seção 11 (backup/rollback) e item 23 de limites conhecidos
- `docs/arquitetura/banco-dados.md` — nota nos scripts do KanbanGeral ("O script tem `DROP` e não há staging") e seção do módulo Acessos ("Só aditiva, e o banco de homologação é o de produção")
- `docs/operacao/troubleshooting.md` — seção 10.1 ("O banco de homologação É o de produção") e seção 6 (migrations)
- `scripts/deploy/deploy.ps1` — trava de PROD (`-Backup 1` + digitar `PROD`), `-SenhaRemota` explícito

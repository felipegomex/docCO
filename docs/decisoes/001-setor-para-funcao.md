---
titulo: "ADR 001 — Eixo do Kanban Geral: Setor → Função"
status: "aceito"
data: "2026-10-01"
---

# ADR 001 — Eixo do Kanban Geral: Setor → Função

## Contexto

O Kanban Geral (migration `20260929034944_KanbanGeral_SetoresEtapasBloqueios`) nasceu com o **eixo de Setor**: tabelas `tbsetor` (5 setores: 1 `Suporte`, 2 `Implantacao`, 3 `Financeiro`, 4 `RH`, 5 `Comercial`) e `tbetapsetor`, mais colunas `tboperador.SetorId`, `tbprojeto.PRJ_SetorId`, `tbtarefa.TRF_SetorId` e `tbcolunakanban.CLK_SetorId`. O recorte do quadro, o eixo de etapas e a fonte da identidade do card ficavam no setor.

## Decisão

Substituir o eixo Setor pela **`tbfuncao`** (função do operador), na migration `20260929221556_SetorParaFuncao`:

- cria `tbfuncaoetapa` (`FUN_*`, catálogo de etapas fixas por função) e as colunas `tbtarefa.TRF_FuncaoId`, `tbprojeto.PRJ_FuncaoId`, `tbcolunakanban.CLK_FuncaoId`;
- descarta `tbsetor`, `tbetapsetor` e as quatro colunas `*_SetorId` (incluindo `tbtarefa.TRF_EtapaTemplateId` apontando para etapa de função);
- o backfill usa o mapa de conversão: setor 1 `Suporte` → função 2; setor 2 `Implantacao` → função 1; setor 3 `Financeiro` → função 4; setor 4 `RH` → `NULL`; setor 5 `Comercial` → `NULL`.

Aplicação em `dbBUSINESS_HML` (banco de produção) feita em **duas fases**, na ordem obrigatória e não negociável: Fase 0 (`SetorParaFuncao_Idempotente.sql`, aditivo, roda com o **código antigo no ar**, não carimba `__EFMigrationsHistory`) → publicar o código novo → Fase 3 (`SetorParaFuncao_Consolidar_Idempotente.sql`, descarta as tabelas/colunas antigas e carimba a migration; aborta com `THROW` se a Fase 0 não rodou). Concluída em `dbBUSINESS_HML` em 2026-10-01 (Fase 0 + Fase 3, com backups `bkp_20261001_pre_setorfuncao_*`). Quem fica no eixo: card, projeto e coluna carregam `FuncaoId`; perdê-la não apaga o dado (`SetNull`). A função também **não** é fonte de super-usuário — a permissão de editar tudo segue vindo de `PERFIL_ID` (`FuncaoAuthorizationService`).

## Consequências

- Cards dos setores **RH** e **Comercial** (mapa → `NULL`) ficaram sem função e, com a retirada da pill "Todas" do quadro em 2026-10-01, **não aparecem em recorte algum** até receberem função — destino a confirmar com o negócio.
- A função **4 (`Financeiro`)** foi cadastrada só como dado de cadastro (`Migrations/Sql/FuncaoFinanceiro_Idempotente.sql`), sem migration EF.
- Após a Fase 3, **não existe rollback de schema por script**: reverter exige restaurar o backup do banco.
- O quadro sempre abre sob uma função (sem pill "Todas"; o endpoint ainda aceita `funcaoId` nulo via URL/API).

## Fonte

- `docs/arquitetura/banco-dados.md` — migration `20260929221556_SetorParaFuncao`, seção "o que os dois scripts fazem" e mapa de conversão
- `docs/operacao/deploy.md` — seção "Migration do eixo Setor → Função" e item 24 de limites conhecidos
- `backend/Central_BackEnd/Migrations/20260929221556_SetorParaFuncao.cs`
- `backend/Central_BackEnd/Migrations/Sql/SetorParaFuncao_Idempotente.sql` e `SetorParaFuncao_Consolidar_Idempotente.sql`
- `docs/operacao/troubleshooting.md` — seção 6 (fase/inversões de ordem e pills por função)

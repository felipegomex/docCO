---
id: 10-migrations-e-tabelas-historicas.05
titulo: "`ColunasCompartilhadas_Idempotente.sql` — catálogo compartilhado e visão por setor"
tipo: referencia
area: dados
sistema: central
tags: [schema, migrations, historico]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Migrations
testes:
  - docs/
tokens_estimados: 795
ficha: docs/modulos/database-explorer.md
---
> Parte de [Migrations e tabelas históricas](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### `ColunasCompartilhadas_Idempotente.sql` — catálogo compartilhado e visão por setor

Script de produção em [Sql/ColunasCompartilhadas_Idempotente.sql](../../../../backend/Central_BackEnd/Migrations/Sql/ColunasCompartilhadas_Idempotente.sql), com carimbo da migration `20261020205521_ColunaFuncao`. **Já aplicado em `dbBUSINESS_HML` em 2026-10-20** (backup conferido com `RESTORE VERIFYONLY`).

| Passo | Efeito |
|---|---|
| Pré-condições (`THROW` 50001–50005) | Aborta se `CLK_FuncaoId`, `CLK_StatusCard`, `TRF_ColunaKanbanId` ou `PRJ_ColunaKanbanId` não existirem, ou se não houver nenhuma coluna global (sem ela não há catálogo para montar) |
| Cria `dbo.tbcolunafuncao` | Idempotente: só cria se `OBJECT_ID('dbo.tbcolunafuncao','U')` for `NULL`; FKs `Cascade` para `tbcolunatecanban` e `tbfuncao`, índice único `(CLF_ColunaKanbanId, CLF_FuncaoId)` e índice em `CLF_FuncaoId` |
| Grava a visão de cada setor | Para cada coluna **de função** existente, cria a visão (coluna do catálogo de mesma `CLK_Ordem`, função) com `CLF_Visivel = 1` e **máscara** = o nome da coluna do setor quando ela difere do catálogo (é o caso do Financeiro: "A FAZER HOJE", "EM DESENVOLVIMENTOO" — o *O* duplo é typo do dado antigo, **preservado**; corrigir é um clique na tela de visão do setor) |
| Remapeia os cards | `tbtarefa`/`tbprojeto` que apontavam para uma coluna de função passam a apontar para a coluna do **catálogo** de mesma `CLK_Ordem` (11 tarefas na homolog). O `CROSS APPLY` com `THROW 50006` impede remapear card para coluna errada: sem par, o script para |
| Remove as cópias por função | `DELETE` das linhas com `CLK_FuncaoId IS NOT NULL` — o catálogo (as 7 colunas) permanece |
| Carimba a migration | `20261020205521_ColunaFuncao` em `__EFMigrationsHistory` (`ProductVersion` `8.0.31`) |
| Conferência | 6 contagens: colunas do catálogo, cópias restantes, visões, visões com máscara, visões ocultas e cards ainda apontando para cópia |

⚠️ **Ordem obrigatória: script ANTES do publish** (backup prévio). O código novo só serve coluna do catálogo; publicar antes deixa o **quadro das funções vazio** e os cards somem do quadro. Na janela entre script e publish o quadro antigo mostra as colunas de função por poucos minutos — aceitável.

⚠️ **O script usa o schema explícito** (`dbo.tbcolunatecanban`, `dbo.tbcolunafuncao`, …) porque o login de aplicação pode ter schema padrão diferente de `dbo`. Não edite o script para remover o prefixo.

⚠️ **`CLK_FuncaoId` continua existindo** em `tbcolunatecanban` (é `NULL` no catálogo), mas **não é mais escrito**: o serviço grava `FuncaoId = null` ao criar coluna. O eixo Função da coluna ficou sem uso — quem rege a coluna agora é a visão em `tbcolunafuncao`.

**Idempotente**: rodar de novo não faz nada (as cópias por função já foram removidas e a visão já existe).

**Não há dados de senha neste arquivo.**


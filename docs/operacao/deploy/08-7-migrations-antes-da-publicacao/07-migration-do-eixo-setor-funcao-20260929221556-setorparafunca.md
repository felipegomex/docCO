---
id: 08-7-migrations-antes-da-publicacao.07
titulo: "Migration do eixo Setor → Função: `20260929221556_SetorParaFuncao`"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, migration, iis, schema]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts/deploy/deploy.ps1
  - scripts/preflight-schema.ps1
  - backend/Central_BackEnd/Migrations
testes:
  - docs/
tokens_estimados: 1449
ficha: docs/modulos/database-explorer.md
---
> Parte de [7. Migrations antes da publicação](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### Migration do eixo Setor → Função: `20260929221556_SetorParaFuncao`

⚠️ **Não existe staging: `dbBUSINESS_HML` (`192.168.2.154`) é o banco de produção.** Esta é a primeira vez que o projeto **descarta** um eixo inteiro (tabelas e colunas) em produção. A ordem abaixo **não é negociável**.

| Item | Valor |
|---|---|
| **Status** | ✅ **Concluída em `dbBUSINESS_HML` em 2026-10-01**: Fase 0 aplicada anteriormente; Fase 3 executada em 2026-10-01 com backups `bkp_20261001_pre_setorfuncao_{tbtarefa,tbprojeto,tboperador,tbcolunakanban,tbsetor,tbetapsetor}`; conferido zero coluna `*SetorId` em `tb%` (fora backups), `tbsetor`/`tbetapsetor` descartadas e migration carimbada. A ordem abaixo continua valendo para **outros ambientes**. |
| Alteração | Cria `tbfuncaoetapa`; adiciona `tbtarefa.TRF_FuncaoId`, `tbprojeto.PRJ_FuncaoId` e `tbcolunakanban.CLK_FuncaoId`; backfill pelo mapa de conversão; índices e FKs `SetNull`; semeia as etapas genéricas das funções 3 e 4 |
| Dados | Copia `tbetapsetor` → `tbfuncaoetapa` preservando nome, ordem, cor, descrição, ativo e auditoria; **descarta** as linhas dos setores 4 (RH) e 5 (Comercial) |
| `DROP` | Só na **Fase 3**: `tbsetor`, `tbetapsetor` e as colunas `tboperador.SetorId`, `tbtarefa.TRF_SetorId`, `tbprojeto.PRJ_SetorId`, `tbcolunakanban.CLK_SetorId` |
| Tabelas legadas afetadas | **nenhuma** (`tbcliente`, `tbcontrato`, `tbcliente_grupo`, `tbchamado`, `tbfuncionario` não são tocadas) |
| Risco | **Alto na Fase 3** (perde o eixo antigo), baixo na Fase 0 (aditiva) |
| `ProductVersion` | `8.0.31`, carimbado pelo script da Fase 3 |
| `Up` da classe | **Vazio de propósito** — não é engano; ver a nota sobre o `RenameTable` indevido abaixo |

#### Ordem obrigatória

```text
Fase 0   Migrations/Sql/SetorParaFuncao_Idempotente.sql          (aditivo, com o código VELHO no ar)
   ↓
        publicar o código NOVO (deploy limpo: validar → package → publish → smoke)
   ↓
Fase 3   Migrations/Sql/SetorParaFuncao_Consolidar_Idempotente.sql  (descarta o eixo antigo; carimba a migration)
```

⚠️ **A Fase 3 não pode ser aplicada antes do deploy.** O app antigo ainda lê `tboperador.SetorId` e `TRF_SetorId`; o `DROP` derruba a tela inteira do Kanban Geral. O próprio script se protege: ele aborta com `THROW` (`50001`–`50004`) se a Fase 0 não tiver criado `TRF_FuncaoId`, `PRJ_FuncaoId`, `CLK_FuncaoId` e `tbfuncaoetapa`.

#### Mapa de conversão do backfill

| Setor antigo (`SET_SetorId`) | Função (`FUNCAO_ID`) | Efeito |
|---:|---:|---|
| 1 `Suporte` | 2 `Suporte` | mantém o recorte |
| 2 `Implantacao` | 1 `Analista de Sistemas` (`CLASSIFICACAO = 'Implantador'`) | mantém o recorte |
| 3 `Financeiro` | 4 `Financeiro` | mantém o recorte |
| 4 `RH` | `NULL` | ⚠️ **card sem função: não aparece em nenhum recorte por função** — e, com a pill "Todas" removida do quadro em 2026-10-01, **não aparece em lugar nenhum** até ganhar função |
| 5 `Comercial` | `NULL` | ⚠️ idem |

⚠️ **Antes de aplicar a Fase 0**, rode a consulta da seção 7 do script (`--- 3. CONFERIR COM O NEGOCIO ---`) e valide com o negócio se havia card/projeto/coluna de RH ou Comercial que precise de outra função em vez de ficar sem dono. Depois, confira as etapas genéricas semeadas nas funções 3 e 4 (as duas que mais usam o quadro e estavam sem conjunto próprio).

#### Confirmação pós-Fase 0 (antes do deploy)

```sql
SELECT COL_LENGTH('tbtarefa', 'TRF_FuncaoId')   AS Tarefa,
       COL_LENGTH('tbprojeto', 'PRJ_FuncaoId')  AS Projeto,
       COL_LENGTH('tbcolunakanban', 'CLK_FuncaoId') AS Coluna,
       OBJECT_ID('tbfuncaoetapa')               AS Etapas;  -- todos devem existir

SELECT TRF_FuncaoId, COUNT(*) AS tarefas FROM tbtarefa GROUP BY TRF_FuncaoId ORDER BY TRF_FuncaoId;
SELECT PRJ_FuncaoId, COUNT(*) AS projetos FROM tbprojeto GROUP BY PRJ_FuncaoId ORDER BY PRJ_FuncaoId;
SELECT e.FUNCAO_ID, f.DESCRICAO, e.FUN_Nome, e.FUN_Ordem
FROM tbfuncaoetapa e LEFT JOIN tbfuncao f ON f.FUNCAO_ID = e.FUNCAO_ID
ORDER BY e.FUNCAO_ID, e.FUN_Ordem;
```

#### Confirmação pós-Fase 3

```sql
SELECT MigrationId, ProductVersion FROM __EFMigrationsHistory
WHERE MigrationId = N'20260929221556_SetorParaFuncao';

SELECT t.name AS tabela, c.name AS coluna
FROM sys.tables t JOIN sys.columns c ON c.object_id = t.object_id
WHERE c.name LIKE '%SetorId%' OR c.name LIKE 'SET[_]%' OR c.name LIKE 'SEP[_]%';
-- deve retornar ZERO linhas

SELECT name FROM sys.tables WHERE name IN (N'tbfuncaoetapa');
```

⚠️ **O `Up` vazio é deliberado.** O snapshot antigo do EF descrevia a tabela de colunas do Kanban com o nome `tbcolunakanban`, enquanto o modelo (`ColunaKanban.cs`) e o banco usam `tbcolunakanban`. O scaffold resolveu essa divergência gerando um `RenameTable` que renomearia uma tabela **já correta** para um nome que só existe no snapshot. Pelo mesmo motivo da `20260929034944`, o DDL real está em dois scripts revisados à mão. **Não** "complete" o `Up`, não rode `dotnet ef database update` e não use o `Down` automático: o `Down` recria `tbsetor`/`tbetapsetor` e as colunas antigas, mas **não** devolve `TRF_Bloqueada`/`TRF_MotivoBloqueio` e não restaura nenhum dado de `tbsetor`. A reversão é manual, por script, com backup.

#### Rollback

Se o quadro novo quebrar depois da Fase 0 e **antes** da Fase 3, o rollback é o binário antigo: ele ainda lê `tbsetor` e as colunas `*_SetorId`, porque a Fase 0 não removeu nada. Depois da Fase 3, o rollback exige restaurar o backup do banco — **não** existe rollback de schema por script para o eixo Função.


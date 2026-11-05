---
id: 08-7-migrations-antes-da-publicacao.08
titulo: "Migration `20260930132443_AcessosGrupoClienteErp` (grupos do Acessos via ERP) — aplicada em 2026-09-30"
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
tokens_estimados: 1103
ficha: docs/modulos/acessos.md
---
> Parte de [7. Migrations antes da publicação](./INDEX.md). Ficha: [Acessos Empresas](../../../modulos/acessos.md).

### Migration `20260930132443_AcessosGrupoClienteErp` (grupos do Acessos via ERP) — aplicada em 2026-09-30

✅ **Aplicada em `dbBUSINESS_HML` em 2026-09-30**, imediatamente antes do publish do commit `d646b19` (janela cumprida na ordem correta), com backup prévio das tabelas afetadas (`bkp_tbgrupoacesso_pre_AcessosGrupoClienteErp`, `bkp_tbclienteacesso_grupo_pre_AcessosGrupoClienteErp`). Dropa `tbgrupoacesso` e repointa os grupos do módulo para a legada `tbcliente_grupo`. Pré-requisito: `20260928212856_AcessosEmpresas` aplicada. Outros ambientes ainda precisam do script.

| Item | Valor |
|---|---|
| Alteração | `DropForeignKey` de `CAO_GrupoId` → `tbgrupoacesso`; **`UPDATE tbclienteacesso SET CAO_GrupoId = NULL`**; `DropTable tbgrupoacesso`; `AlterColumn CAO_GrupoId int → smallint`; `AddColumn CAO_ClienteId int NULL`; índice **único filtrado** em `CAO_ClienteId` (`WHERE CAO_ClienteId IS NOT NULL`); 2 FKs `Restrict`/`NO CHECK` (`CAO_GrupoId` → `tbcliente_grupo.GRUPO_ID`, `CAO_ClienteId` → `tbcliente.CLIENTE_ID`); drop/recria de `IX_tbclienteacesso_CAO_GrupoId` antes do `ALTER COLUMN` |
| Dados | `CAO_GrupoId` é zerado (`NULL`) no ramo do drop: os vínculos antigos de `tbgrupoacesso` não se traduzem para `tbcliente_grupo` (não há mapa). `CAO_ClienteId` nasce `NULL` — o vínculo com a empresa do ERP é preenchido ao criar unidade pelo modal |
| `DROP` | Tabela **`tbgrupoacesso` inteira** (e a entidade `GrupoAcesso` saiu do código) |
| Tabelas legadas afetadas | **nenhuma escrita**: `tbcliente_grupo` e `tbcliente` só recebem FKs, sem `UPDATE` |
| Risco | **Médio–alto**: é o primeiro `DROP TABLE` do módulo em produção e há **janela de incompatibilidade** (abaixo). `CAO_GrupoId` é convertido de `int` para `smallint` — seguro porque a nova fonte (`GRUPO_ID`) é `smallint` |
| Idempotência | Guarda por `__EFMigrationsHistory`, `OBJECT_ID`, `sys.foreign_keys`, `sys.columns` e `sys.indexes`; ramos com `IF`/`ALTER COLUMN` só executam quando necessário; rodar de novo é no-op |
| `ProductVersion` | `8.0.31`, igual às demais migrations |
| Script | [`Sql/AcessosGrupoClienteErp_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/AcessosGrupoClienteErp_Idempotente.sql) |

⚠️ **Janela de incompatibilidade — ordem obrigatória:**

```text
        código VELHO no ar (grava ids de tbgrupoacesso em CAO_GrupoId)
   ↓
AcessosGrupoClienteErp_Idempotente.sql   (repoint da FK; CAO_GrupoId vira smallint ligado a tbcliente_grupo)
   ↓
publicar o código NOVO (deploy limpo: validar → package → publish → smoke)
```

Rodar o script e **esperar** separa script e publish: nesse meio-tempo o código antigo continua gravando ids de `tbgrupoacesso` num campo que agora aponta para `tbcliente_grupo` (FK em `NO CHECK`, então o SQL Server não bloqueia — o erro aparece depois, como grupo "fantasma" na tela). Aplicar o script **imediatamente antes** do publish da API nova.

Confirmação pós-aplicação:

```sql
SELECT MigrationId, ProductVersion FROM __EFMigrationsHistory
WHERE MigrationId = N'20260930132443_AcessosGrupoClienteErp';

SELECT OBJECT_ID(N'tbgrupoacesso') AS DeveSerNULL;          -- tabela dropada
SELECT COL_LENGTH(N'tbclienteacesso', N'CAO_GrupoId')       AS GrupoSmallint,  -- 2
       COL_LENGTH(N'tbclienteacesso', N'CAO_ClienteId')     AS ClienteNovo;    -- 4

SELECT name FROM sys.indexes
WHERE object_id = OBJECT_ID(N'tbclienteacesso')
  AND name IN (N'IX_tbclienteacesso_CAO_GrupoId', N'IX_tbclienteacesso_CAO_ClienteId');

SELECT COUNT(*) AS ClientesComGrupoInvalido                 -- deve ser 0
FROM tbclienteacesso c LEFT JOIN tbcliente_grupo g ON g.GRUPO_ID = c.CAO_GrupoId
WHERE c.CAO_GrupoId IS NOT NULL AND g.GRUPO_ID IS NULL;
```

#### Rollback

O `Down` da classe recria `tbgrupoacesso` **vazia**, devolve a FK antiga (`int`) e **remove** `CAO_ClienteId` (o vínculo com a empresa do ERP some junto com o índice). Os vínculos de grupo apagados pelo `UPDATE ... SET CAO_GrupoId = NULL` **não são restaurados**. Na prática, revertido o schema, as unidades ficam todas sem grupo e sem empresa. Rollback real = restore do backup do banco.


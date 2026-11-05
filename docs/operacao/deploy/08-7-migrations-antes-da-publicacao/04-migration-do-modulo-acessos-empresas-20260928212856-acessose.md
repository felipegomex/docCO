---
id: 08-7-migrations-antes-da-publicacao.04
titulo: "Migration do módulo Acessos Empresas: `20260928212856_AcessosEmpresas`"
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
tokens_estimados: 1065
ficha: docs/modulos/acessos.md
---
> Parte de [7. Migrations antes da publicação](./INDEX.md). Ficha: [Acessos Empresas](../../../modulos/acessos.md).

### Migration do módulo Acessos Empresas: `20260928212856_AcessosEmpresas`

⚠️ **Aplicada com sucesso em `dbBUSINESS_HML` (`192.168.2.154`), que é o banco de produção.** Não existe staging: esta foi a primeira mudança de schema do módulo em ambiente real.

| Item | Valor |
|---|---|
| Alteração | 7 `CREATE TABLE` (`tbgrupoacesso`, `tbclienteacesso`, `tbacessordp`, `tbacessovpn`, `tbacessoanydesk`, `tbacessobanco`, `tbacessoactyonweb`), 3 `ADD COLUMN` **nuláveis** em `tbauditoriaacesso` (`AAC_ClienteAcessoId`, `AAC_AcessoTipo`, `AAC_AcessoId`) e 10 `CREATE INDEX` |
| Tabelas legadas afetadas | **nenhuma** (`tbcliente`, `tbcontrato`, `tbcliente_grupo`, `tbchamado`, `tbfuncionario` não são tocadas) |
| Natureza | **Somente aditiva.** Nada de `DROP`, `RENAME` ou coluna `NOT NULL` nova. As 58 linhas existentes de `tbauditoriaacesso` foram preservadas |
| Risco | Baixo–médio: 7 tabelas novas são inócuas; o ponto de atenção é o **índice único filtrado** de `CAO_Cnpj`, que exige os `SET` de sessão do padrão (o script já os emite) |
| FKs | Todas `ON DELETE NO ACTION` (= `Restrict`). O SQL Server recusa múltiplos caminhos em cascata, e quem desativa é a coluna `Ativo` |
| `Down` | Derruba as 7 tabelas e as 3 colunas — **perde todos os cadastros e a trilha de auditoria do módulo**; prefira correção só para a frente |
| Idempotência | O script guarda por `__EFMigrationsHistory` **e** por `OBJECT_ID`/`COL_LENGTH`/`sys.indexes`/`sys.foreign_keys`; rodar duas vezes é seguro |
| `ProductVersion` | `8.0.31`, igual às demais migrations |
| Backup prévio | `CentralConhecimento_pre_AcessosEmpresas.bak` (21.856 MB), no diretório padrão de backup do servidor |

Confirmação pós-aplicação:

```sql
SELECT MigrationId, ProductVersion
FROM __EFMigrationsHistory
WHERE MigrationId = N'20260928212856_AcessosEmpresas';

SELECT name FROM sys.tables
WHERE name IN (N'tbgrupoacesso', N'tbclienteacesso', N'tbacessordp', N'tbacessovpn',
               N'tbacessoanydesk', N'tbacessobanco', N'tbacessoactyonweb');

SELECT COL_LENGTH(N'tbauditoriaacesso', N'AAC_ClienteAcessoId') AS Cliente,
       COL_LENGTH(N'tbauditoriaacesso', N'AAC_AcessoTipo')    AS Tipo,
       COL_LENGTH(N'tbauditoriaacesso', N'AAC_AcessoId')      AS Acesso;

SELECT COUNT(*) AS EventosPerdidos FROM tbauditoriaacesso;  -- deve continuar 58
```

⚠️ **O aviso de divergência de DDL que ficava aqui não se aplicava mais e foi removido.** A suposta divergência entre snapshot e script em `tbacessoactyonweb` **não existia**: `Usuario` é coluna legítima (herança de `AcessoBase` sem `[Column]`), `SenhaCifrada` foi removida pela `20260929013916` (aplicada) e o snapshot foi regenerado em sincronia com o banco. Quanto a `CAO_Cnpj`: existia em `nvarchar(14)` e **foi dropada** pela `20261002161552_RemoveIdentidadeLocalAcessos` (aplicada em 2026-10-02) — hoje a `tbclienteacesso` não guarda identidade.

**Os riscos reais que restam, no lugar do anterior:**

| Risco | Por quê |
|---|---|
| **`20260929013916` faz `DROP COLUMN` e o `Down` não devolve dado** | O `Up` derruba `SenhaCifrada`; o `Down` recria a coluna **vazia** (`nvarchar(400) NULL`). Reverter a migration restaura o schema, **não** o conteúdo. Requer backup prévio |
| **Não existe staging** | `dbBUSINESS_HML` (`192.168.2.154`) é o banco de produção. É a única migration do módulo com `DROP`, e é a primeira vez que um `DROP` do módulo vai a produção |
| **O pipeline não aplica migration** | `scripts/deploy/deploy.ps1` não invoca `dotnet ef` nem toca em `__EFMigrationsHistory`. Schema e binário podem divergir silenciosamente: a migration tem de ser aplicada à mão, **antes** de publicar o binário correspondente |
| **Reexecutar o script idempotente é no-op** | `AcessosEmpresas_Idempotente.sql` guarda por `__EFMigrationsHistory`, `OBJECT_ID`, `COL_LENGTH`, `sys.indexes` e `sys.foreign_keys`; rodá-lo de novo mantém 11 colunas em `tbacessoactyonweb`, 2 migrations, 7 tabelas, 18 índices e 5 FKs |


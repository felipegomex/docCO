---
id: 08-7-migrations-antes-da-publicacao.09
titulo: "Migration pendente no banco de produção: `20260928182047_LeadContatoVisualizado`"
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
tokens_estimados: 529
ficha: docs/modulos/dashboard.md
---
> Parte de [7. Migrations antes da publicação](./INDEX.md). Ficha: [Dashboard Integrado](../../../modulos/dashboard.md).

### Migration pendente no banco de produção: `20260928182047_LeadContatoVisualizado`

Esta **não** é mais a última migration do modelo: depois dela vieram `20260929013916_AcessosEmpresas_DropColunasOrfasActyonWeb` (Acessos, já aplicada), `20260929034944_KanbanGeral_SetoresEtapasBloqueios` (Kanban Geral, já aplicada), `20260929221556_SetorParaFuncao` (Setor → Função, **já aplicada — Fase 3 concluída em 2026-10-01**) e `20260930132443_AcessosGrupoClienteErp` (Acessos, já aplicada em 2026-09-30). Ainda assim, se o binário a publicar corresponder a um modelo que a inclui, ela precisa estar aplicada no banco de produção **antes** de publicar.

| Item | Valor |
|---|---|
| Alteração | `tblead` ganha `LEAD_ContatoVisualizado bit NOT NULL` com `DEFAULT 0` e constraint `DF_tblead_LEAD_ContatoVisualizado` |
| Tabelas legadas afetadas | **nenhuma** (`tbcliente`, `tbcontrato`, `tbcliente_grupo` não são tocadas) |
| Risco | Baixo: coluna nova `NOT NULL` com default; linhas existentes são gravadas com `0` (cliente ainda não visualizou o contato) |
| `Down` | Remove a coluna; o dado é um flag manual, portanto o rollback perde marcações |
| Idempotência | O script guarda por `__EFMigrationsHistory` **e** por `COL_LENGTH`, então rodar duas vezes é seguro |
| `ProductVersion` | `8.0.31`, igual ao das demais migrations |

Confirmação pós-aplicação:

```sql
SELECT MigrationId, ProductVersion
FROM __EFMigrationsHistory
WHERE MigrationId = N'20260928182047_LeadContatoVisualizado';

SELECT COL_LENGTH(N'tblead', N'LEAD_ContatoVisualizado') AS ColunaExiste;
```

O mesmo vale para homologação: confirme o histórico antes de culpar o código. `tbcliente_grupo` **não tem migration** (é legada, `ExcludeFromMigrations()`); se faltar no banco, o problema é do ambiente legado e não deste projeto.


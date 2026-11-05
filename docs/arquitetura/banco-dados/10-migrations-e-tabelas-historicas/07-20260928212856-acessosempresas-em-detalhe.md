---
id: 10-migrations-e-tabelas-historicas.07
titulo: "`20260928212856_AcessosEmpresas` em detalhe"
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
tokens_estimados: 648
ficha: docs/modulos/acessos.md
---
> Parte de [Migrations e tabelas históricas](./INDEX.md). Ficha: [Acessos Empresas](../../../modulos/acessos.md).

### `20260928212856_AcessosEmpresas` em detalhe

| Grupo | Quantidade | Conteúdo |
|---|---:|---|
| `CREATE TABLE` | 7 | Os 3 níveis do módulo, com `IDENTITY` na PK, `DEFAULT` nas portas/flags e FKs `Restrict` |
| `ADD COLUMN` | 3 | Só em `tbauditoriaacesso`, todas `NULL`: preserva as 58 linhas antigas, que registravam consulta por nome de empresa sem grupo |
| `CREATE INDEX` | 10 | 1 em grupo, 3 em cliente (incluindo o **único filtrado** de CNPJ), 5 compostos `(cliente, ativo)`, 1 em `tbauditoriaacesso` |

⚠️ **Só aditiva, e o banco de homologação é o de produção.** Não há `DROP`, `RENAME` nem coluna `NOT NULL` nova. A execução ocorreu em `192.168.2.154 / dbBUSINESS_HML` com backup prévio (`CentralConhecimento_pre_AcessosEmpresas.bak`, 21.856 MB, no diretório padrão de backup do servidor). Procedimento e riscos em [`09-TROUBLESHOOTING.md`](../../../operacao/troubleshooting.md) e [`10-DEPLOY.md`](../../../operacao/deploy.md).

O script idempotente emite os `SET` de sessão `ANSI_NULLS`/`QUOTED_IDENTIFIER`/`ANSI_PADDING`/`ANSI_WARNINGS`/`ARITHABORT`/`CONCAT_NULL_YIELDS_NULL` **antes** do índice filtrado: o SQL Server recusa `CREATE INDEX` filtrado se qualquer uma dessas opções divergir do padrão da sessão. É a causa mais provável de falha se alguém rodar o script num SSMS com opções diferentes.

**Divergência resolvida.** Este texto antes alertava que a migration, o `Designer`/snapshot e o script divergiam em duas colunas sem prefixo (`SenhaCifrada` e `Usuario`) e em `CAO_Cnpj`. Não há mais divergência: `SenhaCifrada` foi removida pela `20260929013916` (aplicada) e o snapshot foi regenerado, **`Usuario` é legítima** (mapeada por herança de `AcessoBase`, ver a seção do módulo acima), e `CAO_Cnpj` — que existia em `nvarchar(14)` — **foi dropada** junto com `CAO_Fantasia` pela `20261002161552` (aplicada em 2026-10-02). O snapshot voltou a ser fonte confiável de DDL para `tbacessoactyonweb`.

`IMPL_Equipe`, `IMPL_MembroEquipe` e `IMPL_Etapa` são históricos, não entidades do `AppDbContext` atual. `tbcliente_grupo` **não** possui migration: é legada e está marcada com `ExcludeFromMigrations()`. A existência de uma migração não permite afirmar que o banco de destino foi atualizado; é necessário consultar o histórico aplicado no ambiente.


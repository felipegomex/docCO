---
id: sec.21-20-login-500-apos-deploy-do-rbac-schema-e-tipo-de-cargo-nivel
titulo: "20. Login 500 após deploy do RBAC (schema e tipo de `CARGO_Nivel`)"
tipo: operacao
area: diagnostico
sistema: central
tags: [erro, diagnostico]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts
  - backend/Central_BackEnd
testes:
  - docs/
tokens_estimados: 1196
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 20. Login 500 após deploy do RBAC (schema e tipo de `CARGO_Nivel`)

**Sintomas:** dois casos distintos, com causas diferentes.

- **Schema faltando:** `POST /api/v1/auth/login` responde **500** para qualquer usuário, inclusive um
  inexistente. O `smoke.ps1` antigo não pegava porque só checava o Swagger.
- **Tipo incompatível (2026-10-28):** **500 só para usuário válido**; usuário inexistente continua
  **401**. O Event Viewer/log do backend registra `System.InvalidCastException: Unable to cast object
  of type 'System.Byte' to type 'System.Int32'`, com stack em
  `Microsoft.Data.SqlClient.SqlBuffer.get_Int32`.

**Causa (schema):** o backend foi publicado com o modelo que mapeia `tboperador.CARGO_ID` antes de o script
`Permissoes_Idempotente.sql` ser aplicado. O EF seleciona a coluna e o SQL Server devolve
`Invalid column name 'CARGO_ID'`.

A mesma falha reapareceu em **PROD, 2026-10-27**, com o publish da **árvore de trabalho não commitada** sobre `b52d7bc`: `CARGO_ID` já existia, mas faltavam **`CARGO_Nivel`/`CARGO_FuncaoId`** — o EF devolveu `Invalid column name 'CARGO_Nivel'`. Na época o `preflight-schema.ps1` não verificava essas duas colunas (só `CARGO_ID`, as 4 tabelas, 10 cargos ativos e a função 5), então o publish passou; hoje ele verifica.

**Causa (tipo):** `dbo.tbcargo.CARGO_Nivel` é **`tinyint`**, mas
[`Models/Permissoes/Cargo.cs`](../../../backend/Central_BackEnd/Models/Permissoes/Cargo.cs) declarava
`public int Nivel`. Como o `AuthService` carrega o operador com `Include(o => o.Cargo)`, o EF lê a
coluna com `SqlDataReader.GetInt32`, que **não** converte `byte` para `int`, e lança a exceção. O
erro só aparece quando uma linha de `tbcargo` é materializada — por isso o login fictício (operador
inexistente, nenhuma linha de `Cargo` lida) passava em **401** e o login real quebrava em **500**.
Confirmado no Event Viewer de PROD em **2026-10-28**.

**Diagnóstico:**

```sql
SELECT COL_LENGTH('dbo.tboperador','CARGO_ID');     -- NULL = schema faltando
SELECT OBJECT_ID('dbo.tbcargo');                     -- NULL = schema faltando
SELECT COL_LENGTH('dbo.tbcargo','CARGO_Nivel');      -- NULL = schema faltando
SELECT COL_LENGTH('dbo.tbcargo','CARGO_FuncaoId');   -- NULL = schema faltando
-- tipo da coluna (deve ser tinyint)
SELECT DATA_TYPE FROM INFORMATION_SCHEMA.COLUMNS
 WHERE TABLE_NAME = 'tbcargo' AND COLUMN_NAME = 'CARGO_Nivel';
```

Para a variante de tipo, confira o Event Viewer do servidor ou o log do backend: a assinatura é o
`InvalidCastException` `Byte -> Int32` no `get_Int32` do `SqlClient`, e não um `Invalid column name`.
Confira também `deploy/BUILD_INFO.txt`: se o `GitCommit` é o commit do RBAC e a data é anterior à
aplicação do script, é o caso de schema.

**Correção (schema):** aplicar `backend/Central_BackEnd/Migrations/Sql/Permissoes_Idempotente.sql` no banco
do ambiente. O script é idempotente e cria as 4 tabelas, as colunas `CARGO_ID`/`CARGO_Nivel`/
`CARGO_FuncaoId`, a FK, os índices e os **11 cargos** (o preflight exige 10 ativos). Em seguida,
aplique `Permissoes_Matriz_Idempotente.sql` — a matriz usa a hierarquia e o `TECH_LEAD_ADM`. No
incidente de **PROD em 2026-10-27** os dois scripts foram aplicados após backup
`dbCENTRAL_OPERACAO_pre_permissoes_20261027-233801.bak`. Conferir com
`SELECT COUNT(*) FROM dbo.tbcargo` (11, com 10 ativos) e repetir o login fictício: **401** é o
esperado.

**Correção (tipo):** alinhar o CLR ao SQL — `Cargo.Nivel` declarado como `public byte Nivel`
(`byte` ↔ `tinyint`). **Não é preciso alterar o banco**: a coluna continua `tinyint` e os scripts
`Permissoes_*.sql` não mudam. Os consumidores em `PermissaoService` seguem compilando, porque `byte`
converte implicitamente para `int` (ex.: `(int?)o.Cargo.Nivel`). Corrigido no commit `059183d` e
publicado em HML/PROD no `a7d3447`; login real retorna **200**.

**Prevenção:** `deploy.ps1` agora chama `scripts/preflight-schema.ps1` antes do `app_offline.htm` e
aborta se `CARGO_ID`/`tbcargo` não existirem; o preflight também exige as colunas
`CARGO_Nivel`/`CARGO_FuncaoId`. `smoke.ps1` exige 401 no login fictício. A lacuna restante é o
**tipo** da coluna: preflight e `validate` checam existência e compilação, não o mapeamento
CLR ↔ SQL. Publique somente de commit limpo, aplique o SQL do commit (RBAC incluído) e mantenha o
modelo espelhando o tipo real. Não use `-PularPreflightSchema` sem necessidade consciente.


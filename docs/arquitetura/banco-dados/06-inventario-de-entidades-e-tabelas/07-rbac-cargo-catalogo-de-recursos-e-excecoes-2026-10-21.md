---
id: 06-inventario-de-entidades-e-tabelas.07
titulo: "RBAC — cargo, catálogo de recursos e exceções (2026-10-21)"
tipo: referencia
area: dados
sistema: central
tags: [schema, tabelas, entidades, sqlserver]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
  - backend/Central_BackEnd/Migrations
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1553
ficha: docs/modulos/database-explorer.md
---
> Parte de [Inventário de entidades e tabelas](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### RBAC — cargo, catálogo de recursos e exceções (2026-10-21)

**Sem classe de migração EF.** As quatro tabelas e `tboperador.CARGO_ID` vivem em [`Permissoes_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/Permissoes_Idempotente.sql); o `AppDbContextModelSnapshot.cs` **não** descreve as entidades do RBAC. O DDL é aditivo e idempotente, em lotes separados por `GO`, e o script exige `ANSI_NULLS`/`QUOTED_IDENTIFIER` por causa dos índices filtrados.

| Objeto | Efeito confirmado no script |
|---|---|
| `tbcargo` | Cria a tabela, a unique `UQ_tbcargo_Codigo` e semeia os **11** cargos: `GESTOR_MASTER`, `SUPORTE_N2`, `SUPORTE_N1`, `TECH_LEAD_IMPLANTACAO`, `IMPLANTACAO`, `DESENVOLVEDOR`, `TECH_LEAD_DESENVOLVIMENTO`, `CIAA`, `TECH_LEAD_CIAA`, `VISUALIZADOR` e `TECH_LEAD_ADM`. `GESTOR_MASTER` nasce com escopo padrão `3` (`Tudo`); `VISUALIZADOR`, `1` (`Proprio`); os demais, `2` (`Funcao`). O lote de hierarquia preenche `CARGO_Nivel`/`CARGO_FuncaoId`: Master `4`/`NULL`; Tech Lead `3` na função do setor (1, 3, 4 ou 5); `IMPLANTACAO`/`DESENVOLVEDOR`/`CIAA`/`SUPORTE_N1`/`SUPORTE_N2` `2`; `VISUALIZADOR` `1`/`NULL`. As colunas entram por `ALTER TABLE` idempotente, com FK `FK_tbcargo_tbfuncao_FuncaoId`. |
| `tbpermissao_recurso` | Catálogo de abas/APIs/relatórios/tabelas. A chave é estável e gerada pelo extrator de telas; recurso novo não cadastrado é **negado**, não liberado. |
| `tbperm_cargo` | Matriz cargo × recurso × função-alvo. `PEC_FuncaoId = NULL` = todas as funções; escopo no par `(Função, Escopo)`: `(NULL, Funcao)` = a função do próprio operador, `(2, Tudo)` = só a função 2 para qualquer operador, `(NULL, Tudo)` = superusuário. A tela de matriz edita **somente** as linhas gerais (`PEC_FuncaoId = NULL`); as linhas por função (ex.: `PEC_FuncaoId = 5` em `aba.ciaa.ideias`) são concessões de setor e sobrevivem ao **salvar**. |
| `tbperm_operador` | Exceção individual por operador (e função-alvo opcional). `PEO_Negativa = 1` **revoga** as ações listadas e vence o cargo. |
| `tboperador.CARGO_ID` | Coluna `int NULL`, FK `FK_tboperador_tbcargo_CARGO_ID` para `tbcargo` com `ON DELETE SET NULL` e índice `IX_tboperador_CARGO_ID`. `NULL` = Visualizador até o mapa ser aplicado (o motor nega o que não vier de exceção individual). |

**Enums gravados como bitset/`tinyint`** ([`AcaoPermissao.cs`](../../../../backend/Central_BackEnd/Models/Permissoes/AcaoPermissao.cs)): `AcaoPermissao` é `[Flags]` (`Ler`=1, `Criar`=2, `Editar`=4, `Excluir`=8, `Aprovar`=16, `Gerir`=32, `Auditar`=64), gravado em `PEC_Acoes`/`PEO_Acoes` — **não** existe uma linha por ação. `EscopoPermissao` é `Proprio`=1, `Funcao`=2, `Tudo`=3. `TipoRecurso` é `Aba`=1, `Api`=2, `Relatorio`=3, `Tabela`=4.

**Scripts idempotentes e ordem:**

| Script | Papel | Estado em `dbBUSINESS_HML` (2026-10-21) |
|---|---|---|
| [`Permissoes_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/Permissoes_Idempotente.sql) | Cria as 4 tabelas, a coluna `CARGO_ID`, FK/índice, as colunas `CARGO_Nivel`/`CARGO_FuncaoId` (FK para `tbfuncao`) e os **11** cargos, com a hierarquia de níveis carregada. Não altera o acesso de ninguém: as tabelas de permissão nascem vazias e `CARGO_ID` nasce `NULL`. | **Aplicado**: 4 tabelas + `CARGO_ID`; 10 cargos na carga original. **Reaplicar** para criar `TECH_LEAD_ADM` e as colunas de hierarquia. |
| [`Permissoes_CargaInicial_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/Permissoes_CargaInicial_Idempotente.sql) | Projeta o estado atual em `CARGO_ID`: `SE_ADMIN = 1` ou `PERFIL_ID = 'A'` → `GESTOR_MASTER`; gestor ativo de função 1/3/5 → `TECH_LEAD_IMPLANTACAO`/`TECH_LEAD_DESENVOLVIMENTO`/`TECH_LEAD_CIAA`. Só preenche `CARGO_ID NULL` e **não** altera `SE_ADMIN`/`PERFIL_ID`. | **Aplicado**: **8** `GESTOR_MASTER`, **1** `TECH_LEAD_IMPLANTACAO` e **32** operadores sem cargo (Visualizador). |
| [`Permissoes_SeedOperadores_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/Permissoes_SeedOperadores_Idempotente.sql) | Semeia os recursos `aba.sistema.operadores` (`/sistema/operadores`, controller `OperadoresController`) e `aba.sistema.funcoes-administrativas` (`/sistema/funcoes-administrativas`) e concede `Ler|Criar|Editar` (7) com escopo `Funcao` (2) aos três cargos de Tech Lead. O `GESTOR_MASTER` não precisa de linha: o motor tem bypass pelo código do cargo. | **Aplicado**. |
| [`Permissoes_MapaCargos_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/Permissoes_MapaCargos_Idempotente.sql) | Aplica o mapa nominal de cargos e de `tbgestor` (inclui `TECH_LEAD_ADM`) aos operadores ativos, sem alterar `PERFIL_ID` dos não-Masters. | **Aplicado**. |
| [`Permissoes_Matriz_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/Permissoes_Matriz_Idempotente.sql) | Semeia o catálogo de recursos `aba.*` e a matriz padrão por cargo (Master sem linha; Tech Lead do setor `63`/escopo Função; comum `7`/Próprio; compartilhados e Ideias para todos). Em 2026-10-22 acrescentou `aba.ciaa.projetos`, `aba.ciaa.ideias`, `aba.implantacao.ideias`, `aba.suporte.whatsapp.respostas-rapidas`, `aba.ferramentas.mock-api` e `aba.ferramentas.mock-request`, e atualizou `aba.ferramentas.base-conhecimento` para `/base-conhecimento`. Em 2026-10-27 passou a conceder `aba.sistema.permissoes` (`Ler`, escopo `Funcao`) aos quatro Tech Leads e a criar `aba.ciaa.ideias` com `PEC_FuncaoId = 5` (`Ler`, escopo `Funcao`) para todo cargo não-Master. É idempotente e roda **depois** de `Permissoes_MapaCargos_Idempotente.sql` e de `Permissoes_Idempotente.sql`. | **Aplicado** (recursos novos entram na próxima execução em cada ambiente). |

`tbfuncao` na mesma base tem **5** linhas: `FUNCAO_ID` 4 = `Executivo(a) Comercial` e 5 = `CIAA` (a função 5 é exigida pelo preflight de schema). As funções continuam sendo o **setor** (`tboperador.FUNCAO_ID`) e não se confundem com `tbcargo`.


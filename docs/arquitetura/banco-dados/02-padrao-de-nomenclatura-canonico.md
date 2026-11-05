---
id: sec.02-padrao-de-nomenclatura-canonico
titulo: "Padrão de nomenclatura (canônico)"
tipo: referencia
area: dados
sistema: central
tags: [schema, banco, tabelas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1273
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Padrão de nomenclatura (canônico)

### Tabela

Nome de tabela = `tb` + nome da entidade em minúsculas, **colado, sem underscore**. Exemplos: `tbprojeto`, `tbtarefa`, `tbagenda`, `tboperador`, `tbrefreshtoken`, `tbprojetoetapa`.

- **Exceção — sub-entidades:** `tb<entidade>_<sub>`, com um único underscore separando o agregado pai da sub-entidade. É o padrão já presente no legado (`tbcliente_contato`, `tbchamado_itens`) e vale também para os espelhos legados mapeados, como `tbcliente_grupo`.
- A tabela de colunas do Kanban **não** segue a regra: `ColunaKanban.cs` e o script de produção declaram **`tbcolunakanban`** (14 caracteres), que é o nome real no banco. ⚠️ **O snapshot do EF e todos os `Designer` de `20260928182047` em diante declaram `tbcolunakanban`** (14 caracteres) — ver a armadilha "tabela das colunas do Kanban" na seção de migrations.
- Nomes de tabela não levam prefixo de módulo: o prefixo `tb` é único e substituiu `IMPL_`/`CC_`.

### Colunas (padrão anterior, inalterado)

A padronização afeta **apenas nomes de tabela**. As colunas seguem como antes:

| Elemento | Padrão |
|---|---|
| Chave primária | `Id` na coluna; nas legadas o nome é o próprio identificador (`OPERADOR_ID`, `CLIENTE_ID`, `FUNCAO_ID`). |
| Prefixo de coluna | Prefixo curto da entidade + `_`: `PRJ_` (projeto), `TRF_` (tarefa), `AGD_` (agenda), `TPP_`, `CLK_`, `CMT_`, `APT_`, `PEP_`, `PEC_`, `PED_`, `PEH_`, `AUD_`. O prefixo `CLI_` saiu do modelo junto com a tabela `IMPL_Cliente`. |
| Caixa | Nas tabelas novas é PascalCase (`PRJ_Id`); nas legadas é UPPER (`OPERADOR_ID`, `CHAMADO_ID`). |
| Chaves compostas N:N | Usam o nome da entidade singular + `Id`, sem prefixo (`TRF_Id`, `CHAMADO_ID`, `OPERADOR_ID`). |

### Chaves primárias

- `Id` simples por padrão.
- Chave **composta** apenas nas tabelas N:N: `tbtarefareponsavel` (`TarefaId`, `OperadorId`) e `tbtarefachamado` (`TarefaId`, `ChamadoId`).
- Nomes de PK acompanham o nome novo da tabela (`PK_tbprojeto`, `PK_tbfuncao`, …).

### Chaves estrangeiras

| Ação | Quando |
|---|---|
| `Cascade` | Tabela filha cujo ciclo de vida acompanha o pai (comentários, responsáveis, chamados, apontamentos, participantes, as quatro coleções de etapa). |
| `SetNull` | Vínculo opcional (`Projeto.ClienteId`, `Projeto.ColunaKanbanId`, `Tarefa.ColunaKanbanId`, `AgendaItem.ProjetoId`, `AgendaItem.TipoId`, `Operador.FuncaoId`). |
| `Restrict` | Quando o SQL Server rejeitaria cascata por múltiplos caminhos (`Projeto.TipoProjetoId`, `Tarefa.ProjetoEtapaId`, todas as 6 FKs do módulo Acessos Empresas) ou quando a referência precisa existir (`AgendaParticipante.ParticipanteId`). No Acessos, o soft delete pela coluna `Ativo` é quem "remove": nada é apagado em cascata. |

### Nomes de PK, FK e índice

- **PKs e o índice composto de conflito da agenda foram renomeados** para acompanhar o prefixo `tb`.
- **As FKs NÃO foram renomeadas.** Constraints como `FK_IMPL_Projeto_IMPL_ColunaKanban_PRJ_ColunaKanbanId` continuam com o prefixo antigo mesmo apontando para `tbcolunakanban`. **Isso é cosmético, não é bug**: `sp_rename` de tabela já reescreve internamente o objeto referenciado, e renomear a constraint exigiria derrubá-la e recriá-la. Nunca concluir que uma FK "quebrou" pelo nome antigo.
- Exceção criada na fusão da função: `FK_tboperador_tbfuncao_FUNCAO_ID` já nasce com o nome novo, porque a constraint antiga caiu junto com a tabela `CC_Funcao`.
- **Índices** seguem a convenção do EF `IX_<tabela>_<colunas>`; **chave alternativa** (unique constraint) segue `AK_<tabela>_<colunas>`. Os 33 índices com prefixo antigo e a unique constraint `AK_tbtipoevento_Nome` foram renomeados pela `RenomeiaIndicesPkTb`.
- **Índice de conflito da agenda (divergência encerrada):** o nome final é `IX_tbagenda_AGD_OperadorId_AGD_DataInicio_AGD_DataFim`, derivado por convenção. O `HasDatabaseName("IX_IMPL_Agenda_Operador_DataInicio_DataFim")` foi **removido** do `AppDbContext` (bloco `Entity<AgendaItem>`); o `AppDbContext` não fixa mais nome de índice, e modelo, snapshot e banco concordam.

### Fluxo de criação de tabela

1. Modelo/entidade em `backend/Central_BackEnd/Models/`.
2. `DbSet` e bloco de mapeamento em [AppDbContext.cs](../../../backend/Central_BackEnd/Data/AppDbContext.cs) (`HasKey`, `HasIndex`, `HasOne`/`OnDelete`). Tabela legada somente leitura recebe `ToTable(... , t => t.ExcludeFromMigrations())`.
3. `dotnet ef migrations add <Nome> -o Migrations`.
4. **Aplicação manual** no ambiente alvo — o backend não executa `Migrate()` no startup e `scripts/deploy/deploy.ps1` não aplica migrations. Para renomeação em massa, o caminho de produção é [renomear-tabelas-tb-idempotente.sql](../../../scripts/db/renomear-tabelas-tb-idempotente.sql).


---
id: 06-inventario-de-entidades-e-tabelas.11
titulo: "`tbcontrato` — colunas mapeadas e regras do espelho"
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
tokens_estimados: 597
ficha: docs/modulos/dashboard.md
---
> Parte de [Inventário de entidades e tabelas](./INDEX.md). Ficha: [Dashboard Integrado](../../../modulos/dashboard.md).

### `tbcontrato` — colunas mapeadas e regras do espelho

A tabela legada **não** tem coluna `DATA_FATURAMENTO`. Mapear essa coluna derrubava **todos** os endpoints que leem `tbcontrato` com `Invalid column name`. O espelho segue o schema real:

| Propriedade | Coluna | Tipo CLR | Observação |
|---|---|---|---|
| `Id` | `CONTRATO_ID` | `int` | PK. |
| `ClienteId` | `CLIENTE_ID` | `int?` | FK lógica; navegação `Cliente`. |
| `QtdeLicenca` | `QTDE_LICENCA` | `short?` | **`smallint` no banco.** Mapear como `decimal?` quebra a leitura com `InvalidCastException` de `Int16` para `Decimal` no SqlClient; por isso o `AppDbContext` **não** aplica `HasPrecision` nesta coluna. |
| `ValorUnitario` | `VALOR_UNITARIO` | `decimal?` | `decimal(10,2)`. |
| `Ativo` | `ATIVO` | `string?` (`char(1)`) | Flag `S`/`N`. |
| `DiaFaturamento` | `DIA_FATURAMENTO` | `short?` | **Dia do mês** do faturamento (1–31). Não existe data de faturamento no legado; o ciclo é mensal e recorrente. |
| `DataProximoReajuste` | `DATA_PROXIMO_REAJUSTE` | `DateTime?` | Única data real de ciclo. |
| `Situacao` | `SITUACAO` | `string?` | Valores reais: `NULL`, `Entregue`, `Pendente`. **`"Ativo"` nunca ocorre.** |
| `Cliente` | — | `ClienteLegado?` | Navegação de leitura, carregada só quando solicitada. |
| `EhAtivo` | — | `bool` (`[NotMapped]`) | `ATIVO == "S"` **ou** `SITUACAO == "Entregue"`. Centraliza a regra antes duplicada em `AdministrativoService` e `FinanceiroService`. |
| `ValorMensal` | — | `decimal` (`[NotMapped]`) | `(QtdeLicenca ?? 1) * (ValorUnitario ?? 0)`; fallback de 1 licença quando o legado não informa a quantidade. É uma **projeção**, não receita confirmada, e por isso só é exposta quando `FlagsDashboard.FaturamentoLiberado` está `true`. |

O contrato legado é **somente leitura**: a entity usa `ExcludeFromMigrations()` e nenhum serviço do projeto escreve nesta tabela.

O antigo `DbSet<Cliente>` / `IMPL_Cliente` **não existe mais**: a entidade `Cliente` foi apagada junto com o `DbSet` e o bloco de mapeamento. A fonte única de clientes é a legada `tbcliente`.


---
id: sec.09-colunas-geridas-funil-comercial-com-produtos-2026-10-08
titulo: "Colunas geridas: funil comercial com produtos (2026-10-08)"
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
tokens_estimados: 495
ficha: docs/modulos/dashboard.md
---
> Ficha: [Dashboard Integrado](../../modulos/dashboard.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Colunas geridas: funil comercial com produtos (2026-10-08)

Migration `20261008090000_ComercialFunilProdutos`; script idempotente de produção em [Sql/ComercialFunilProdutos_Idempotente.sql](../../../backend/Central_BackEnd/Migrations/Sql/ComercialFunilProdutos_Idempotente.sql) — **aplicar em `dbBUSINESS_HML` antes do publish**, pois o Kanban e as Propostas quebram com `Invalid column name` sem elas. Nenhuma tabela legada (`tbcliente`, `tbcontrato`, `tbproduto`) é criada ou alterada.

| Coluna | Tipo | Nulidade | Default | Uso |
|---|---|---|---|---|
| `tblead.LEAD_Arquivado` | `bit` | `NOT NULL` | `0` | Soft-delete: negócio fechado ou proposta não convertida sai da visão principal (índice próprio). |
| `tblead.LEAD_DataArquivamento` | `datetime2` | `NULL` | — | Quando arquivou. |
| `tblead.LEAD_MotivoArquivamento` | `varchar(500)` | `NULL` | — | Motivo informado no arquivamento. |
| `tbleadcontato.LDC_NomeContato` | `varchar(150)` | `NOT NULL` | `''` (backfill) | Nome da pessoa contatada (obrigatório no `POST`). |
| `tboportunidade.OPP_ProdutoId` | `int` | `NULL` | — | Produto ofertado (`tbproduto.PRODUTO_ID`, vínculo lógico, índice próprio). |
| `tboportunidade.OPP_ValorImplementacao` | `decimal(10,2)` | `NULL` | — | Valor de implementação. |
| `tboportunidade.OPP_ValorPorLicenca` | `decimal(10,2)` | `NULL` | — | Valor por PA/licença (mesmo conceito). |
| `tboportunidade.OPP_QtdeLicencas` | `int` | `NULL` | — | Estimativa de PAs/licenças. |
| `tboportunidade.OPP_MotivoPerda` | `varchar(1000)` | `NULL` | — | Motivo da proposta não convertida (`Resposta = NÃO`). |


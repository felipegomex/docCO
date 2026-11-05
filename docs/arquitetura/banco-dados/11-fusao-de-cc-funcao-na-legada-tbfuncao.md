---
id: sec.11-fusao-de-cc-funcao-na-legada-tbfuncao
titulo: "Fusão de `CC_Funcao` na legada `tbfuncao`"
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
tokens_estimados: 366
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Fusão de `CC_Funcao` na legada `tbfuncao`

A entidade `Funcao` passou a mapear a tabela legada `tbfuncao` (antes apontava `CC_Funcao`, criada pela migration `AddFuncao`).

| Coluna | Antes | Agora |
|---|---|---|
| `FUNCAO_ID` | `smallint` (legada) | `int NOT NULL` — **widening necessário** porque `tboperador.FUNCAO_ID` é `int`. Exige derrubar e recriar a PK. |
| `DESCRICAO` | legada | inalterada (limite 100) |
| `CLASSIFICACAO` | inexistente na legada | `varchar(50) NULL` |
| `ATIVO` | inexistente na legada | `bit NOT NULL DEFAULT 1` |

Conteúdo preservado: `FUNCAO_ID` 1, 2 e 3, com `DESCRICAO` da base legada (Analista de Sistemas, Suporte, Programador) e `CLASSIFICACAO` preenchida pela migration como `Implantador`, `Atendimento` e `Desenvolvimento`; todas com `ATIVO = 1`. Nenhuma FK de outro sistema referenciava `tbfuncao` antes da alteração.

A FK `tboperador.FUNCAO_ID` foi recriada com `ON DELETE SET NULL` e nome novo `FK_tboperador_tbfuncao_FUNCAO_ID` — a única FK que acompanha o padrão, porque a constraint antiga caiu junto com a tabela `CC_Funcao`.


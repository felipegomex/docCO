---
id: sec.05-provider-de-persistencia
titulo: "Provider de persistência"
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
tokens_estimados: 204
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Provider de persistência

- Em `Development`, [Program.cs](../../../backend/Central_BackEnd/Program.cs) usa SQL Server quando `Database:UseSqlServer=true`; caso contrário, usa InMemory `CentralDev`, cujos dados não sobrevivem ao fechamento do processo.
- Fora de `Development`, o provider selecionado é SQL Server. A configuração de conexão e as flags do provider são externas a este inventário e não são reproduzidas aqui.


---
id: sec.15-database-explorer
titulo: "Database Explorer"
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
tokens_estimados: 611
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Database Explorer

- [DatabaseConnectionService.cs](../../../backend/Central_BackEnd/Services/Database/DatabaseConnectionService.cs) lê `DB_EXPLORER_*` com precedência sobre `DatabaseExplorer:*`. A senha nunca deve entrar em documentação ou resposta de configuração.
- A classe `DatabaseConnectionConfig` declara `Encrypt=false` e `TrustServerCertificate=true`; `FromConfiguration` recalcula as duas opções e, sem valor booleano configurável, obtém `false`. A configuração efetiva do ambiente precisa ser verificada separadamente.
- O serviço consulta tabelas, colunas, índices, FKs, PKs (`ListarPkAsync`), procedimentos, funções, gatilhos, dependências e relacionamentos no SQL Server configurado.
- Relacionamentos confirmados vêm de FKs reais; relacionamentos possíveis são inferidos por nomes/tipos/índices e carregam pontuação. A inferência não é uma FK.
- `DatabaseQueryBuilderService` valida até cinco tabelas, monta SELECT/CTE/joins/filtros/ordenações e devolve SQL; não foi localizado endpoint que execute esse SQL.
- `DatabaseConnectionConfigDto` mascara a senha no controller. Entretanto, todos os endpoints do controller exigem apenas `Authorize`, e os DTOs de procedimento/gatilho podem conter o corpo completo do objeto. A exposição deve ser considerada em função do perfil do usuário.
- `SqlScriptGeneratorService` consome o resultado da comparação (com `SchemaArquivo`/`SchemaJca`) e devolve `SqlScriptResultDto` (`SqlScriptDto` com `tabela`/`severidadeOrigem`, `SqlScriptResumoDto`), somente com criações vindas do arquivo; os contratos de geração/validação ficam em `DatabaseDtos.cs` e não são persistidos em nenhuma tabela (sem EF/migração). A comparação de procedures usa `ProceduresComparisonResultDto` também em `DatabaseDtos.cs`, sem persistência.
- A comparação de schemas carrega a chave primária em `SchemaInfoDto.Pk` (`SchemaPkInfoDto`: `nome` + `colunas`) e classifica as diferenças em `SchemaDifferenceDto.Categoria` (`Coluna`, `Tipo`, `Nullable`, `Indice`, `Fk`, `Pk` e `Tabela`); tudo é contrato de API, sem tabela própria.


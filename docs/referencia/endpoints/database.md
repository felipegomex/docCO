---
title: "Catálogo de endpoints — pasta Database"
description: "Controller de backend/Central_BackEnd/Controllers/Database: o Database Explorer, com leitura de metadados, comparação de schemas e geração de scripts de correção."
date: "2026-10-18"
language: "pt-BR"
status: "canônico"
---

# Endpoints — Controllers/Database

Pasta `backend/Central_BackEnd/Controllers/Database`: **1 classe** e **24 operações** `[Http*]`, conferidas em 2026-10-18 pela contagem recursiva dos atributos. Índice do catálogo em [`endpoints.md`](../endpoints.md); seções por pasta: [`raiz.md`](raiz/INDEX.md), [`implantacao.md`](implantacao/INDEX.md) e [`dashboard.md`](dashboard.md).

Suma por controller (operações `[Http*]`):

| Controller | Operações | Rota base |
|---|---:|---|
| `DatabaseController` / Database Explorer | 24 | `/api/v1/database` |
| **Total** | **24** | — |

## Database Explorer

Fonte: [DatabaseController.cs](../../../backend/Central_BackEnd/Controllers/Database/DatabaseController.cs). Todas as operações exigem JWT; as comparações (schemas, banco inteiro e procedures) e os endpoints de scripts de correção usam `validacao`.

| Método | Rota | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/api/v1/database/status` | — | Sem body | `200 DatabaseStatusDto` com teste de conexão, servidor, banco, mensagem e duração. |
| `GET` | `/api/v1/database/info` | — | Sem body | `200 DatabaseInfoDto` com contagens de objetos SQL Server; pode retornar `Conectado=false` e mensagem. |
| `GET` | `/api/v1/database/tables` | — | Query `schema?`, `filtro?` | `200 List<TableDto>`. |
| `GET` | `/api/v1/database/tables/{schema}/{nome}` | — | Path `schema`, `nome` | `200 TableDto` ou `404`. |
| `GET` | `/api/v1/database/tables/{schema}/{nome}/columns` | — | Path `schema`, `nome` | `200 List<ColumnDto>`. |
| `GET` | `/api/v1/database/tables/{schema}/{nome}/indexes` | — | Path `schema`, `nome` | `200 List<IndexDto>`. |
| `GET` | `/api/v1/database/relationships` | — | Query `schema?`, `tabela?`, `incluirPossiveis=false`, `take=500` | `200 List<RelationshipDto>`, com confirmadas e inferidas. |
| `GET` | `/api/v1/database/graph` | — | Query `tabela`, `profundidade=2`, `incluirPossiveis=false` | `200 List<RelationshipDto>`; `400` se `tabela` ausente. |
| `GET` | `/api/v1/database/column-usage` | — | Query `coluna` | `200 List<RelationshipDto>`; `400` se `coluna` ausente. |
| `GET` | `/api/v1/database/search` | — | Query `termo`, `take=200` | `200 List<DatabaseSearchResult>`; termo vazio devolve lista vazia. |
| `GET` | `/api/v1/database/procedures` | — | Query `schema?`, `busca?`, `take=5000` | `200 List<ProcedureResumoDto>`. O `take` é limitado por `Math.Clamp` no serviço, mas não aplicado à consulta. |
| `GET` | `/api/v1/database/procedures/{schema}/{nome}` | — | Path `schema`, `nome` | `200 ProcedureDetalheDto` ou `404`. |
| `GET` | `/api/v1/database/triggers` | — | Query `schema?`, `tabela?` | `200 List<TriggerDto>`. |
| `GET` | `/api/v1/database/triggers/{schema}/{nome}` | — | Path `schema`, `nome` | `200 TriggerDto` ou `404`. |
| `GET` | `/api/v1/database/tables/{schema}/{nome}/dependencies` | — | Path `schema`, `nome` | `200 List<DependencyDto>`. |
| `GET` | `/api/v1/database/procedures/{schema}/{nome}/analysis` | — | Path `schema`, `nome` | `200 ProcedureAnalysisDto` ou `404`. |
| `POST` | `/api/v1/database/query-builder-advanced` | — | Body `QueryBuilderAdvancedRequest` | `200 DatabaseQueryBuilderResult`; gera SQL para até cinco tabelas, não o executa. `400` se nenhuma tabela for enviada. |
| `GET` | `/api/v1/database/config` | — | Sem body | `200 DatabaseConnectionConfigDto`; senha preenchida é substituída por `***`. |
| `POST` | `/api/v1/database/compare-schemas` | `validacao` | Form `schema`, `tabela`, arquivo JSON | `200 SchemaComparisonResultDto` (schemas com `SchemaInfoDto.Pk` e diferenças de categoria incluindo `Pk`); limite de 5 MB e erros de arquivo/JSON retornam `400`; caracteres de controle crus em string JSON são reparseados com escape `\u00XX` (nome preservado) e **não** contam como JSON inválido — o arquivo gerado pelo script do sistema pode ser enviado sem edição manual. |
| `POST` | `/api/v1/database/compare-schemas-bulk` | `validacao` | Form com arquivo JSON | `200 BulkSchemaComparisonResultDto` (mesmo contrato, com PK por tabela); limite de 20 MB; erros de arquivo/JSON retornam `400` — a mensagem do parse bulk inclui o número da linha e um trecho quando uma linha JSON Lines/CSV-Lines não pode ser lida (nunca há perda silenciosa de linha); caracteres de controle crus (0x00-0x1F, exceto `\t\n\r`) em string JSON são reparseados com escape `\u00XX` e o caractere é preservado no nome (divergência visível), sem gerar `400`. |
| `POST` | `/api/v1/database/compare-procedures` | `validacao` | Form com arquivo JSON | `200 ProceduresComparisonResultDto` (itens com `status` `Compativel`/`Divergente`/`SomenteBanco`/`SomenteArquivo` e corpos); limite de 20 MB; somente leitura, nenhum script é gerado; caracteres de controle crus em string JSON são reparseados com escape `\u00XX` (valores preservados), sem gerar `400`. |
| `POST` | `/api/v1/database/generate-correction-scripts` | `validacao` | Body `GerarScriptsRequest` (resultado da comparação; `opcoes?` aceito mas sem efeito) | `200 SqlScriptResultDto` só com **criações** vindas do arquivo (`CREATE_TABLE`, `ADD_COLUMN`, `CREATE_INDEX`, `ALTER_FK`), cada script com `tabela` e `severidadeOrigem` (a PK do `CREATE TABLE` vem de `arquivo.Pk`, com fallback para índice `PK*`); limite de 25 MB; nunca executa SQL. |
| `POST` | `/api/v1/database/generate-correction-scripts-bulk` | `validacao` | Body `GerarScriptsBulkRequest` (resultado bulk; `opcoes?` aceito mas sem efeito) | `200 SqlScriptResultDto`; mesmo contrato do modo tabela única, ordenado por tabela; limite de 25 MB; nunca executa SQL. |
| `POST` | `/api/v1/database/validate-script` | `validacao` | Body `ValidarScriptRequest` (`sql`) | `200 ValidarScriptResultDto` com `valido`, `erros` e `avisos` (validação estática, sem execução); limite de 2 MB. |

O `DatabaseConnectionService` prioriza `DB_EXPLORER_*` sobre `DatabaseExplorer:*`; nenhuma senha ou string de conexão deve ser incluída em exemplo. A consulta avançada é um construtor de SQL: não foi localizada uma rota que execute o resultado. Os endpoints de scripts recebem o JSON da comparação pronto (sem novo upload) e apenas geram/validam texto SQL.

## Erros confirmados (Database)

| Situação | Retorno explicitamente tratado |
|---|---|
| Comparação de schema sem arquivo/JSON inválido | `400` com mensagem específica do Database Explorer. Caracteres de controle crus em string JSON **não** contam como inválido: o parse reparseia com `\u00XX` e preserva o caractere no nome (divergência visível). |

Não se deve transformar uma mensagem de erro textual em novo contrato; o código pode mudar.

## Limitações confirmadas de consulta (Database)

- Em `GET /api/v1/database/procedures`, `take` é limitado por `Math.Clamp` no serviço, mas o valor não é aplicado à consulta SQL; portanto o parâmetro não corta a lista retornada.

## Limitações de contrato a corrigir (Database)

- [Limitação] O Database Explorer não possui um controller de execução de SQL; não documentar o SQL gerado como se tivesse sido executado.

## Limitações conhecidas (ex-arquivo da pasta Database)

Divergências já confirmadas no código (documentos arquivados que as continham foram removidos em 09/10):

| Divergência | Estado vigente |
|---|---|
| Total 81 endpoints / `DatabaseController` com 19 operações | a contagem vigente é a da tabela acima (24 operações no Database Explorer) |
| `POST /api/v1/database/procedures/{schema}/{name}` | o controller atual só expõe `GET` nessa rota |

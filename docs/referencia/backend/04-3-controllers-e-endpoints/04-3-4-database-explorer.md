---
id: 04-3-controllers-e-endpoints.04
titulo: "3.4 Database Explorer"
tipo: referencia
area: backend
sistema: central
tags: [backend, controllers, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1503
ficha: docs/modulos/database-explorer.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### 3.4 Database Explorer

Todos os endpoints usam JWT. Os três de comparação (schemas, banco inteiro e procedures) e os três de scripts de correção também usam `validacao` e limite de requisição.

| Método e caminho | Corpo/parâmetros | Serviço |
|---|---|---|
| `GET /api/v1/database/status` | Sem corpo | Testa `SELECT 1` |
| `GET /api/v1/database/info` | Sem corpo | Versão e contadores de objetos |
| `GET /api/v1/database/tables` | `schema?`, `filtro?` | Lista tabelas e contagem de registros |
| `GET /api/v1/database/tables/{schema}/{nome}` | Sem corpo | Detalhe da tabela |
| `GET /api/v1/database/tables/{schema}/{nome}/columns` | Sem corpo | Colunas e valores padrão |
| `GET /api/v1/database/tables/{schema}/{nome}/indexes` | Sem corpo | Índices |
| `GET /api/v1/database/relationships` | `schema?`, `tabela?`, `incluirPossiveis?`, `take?` | FKs confirmadas e inferidas |
| `GET /api/v1/database/graph` | `tabela`, `profundidade?`, `incluirPossiveis?` | Grafo por percurso BFS |
| `GET /api/v1/database/column-usage` | `coluna` | Ocorrências exatas do nome da coluna |
| `GET /api/v1/database/search` | `termo`, `take?` | Busca em tabelas, colunas, views, procedures, functions e triggers |
| `GET /api/v1/database/procedures` | `schema?`, `busca?`, `take?` | Procedures resumidas |
| `GET /api/v1/database/procedures/{schema}/{nome}` | Sem corpo | Procedure, parâmetros e corpo da definição |
| `GET /api/v1/database/triggers` | `schema?`, `tabela?` | Triggers e eventos inferidos do corpo |
| `GET /api/v1/database/triggers/{schema}/{nome}` | Sem corpo | Trigger específica e corpo da definição |
| `GET /api/v1/database/tables/{schema}/{nome}/dependencies` | Sem corpo | Dependências por busca textual e metadados |
| `GET /api/v1/database/procedures/{schema}/{nome}/analysis` | Sem corpo | Análise textual da procedure |
| `POST /api/v1/database/query-builder-advanced` | `QueryBuilderAdvancedRequest` | Gera SQL; não executa a consulta |
| `GET /api/v1/database/config` | Sem corpo | Retorna servidor, porta, banco, usuário e flags; o controller substitui a senha configurada por `***` |
| `POST /api/v1/database/compare-schemas` | Multipart: `schema`, `tabela`, `arquivo` | Compara uma tabela; upload vazio rejeitado |
| `POST /api/v1/database/compare-schemas-bulk` | Multipart: `arquivo` | Compara várias tabelas; upload vazio rejeitado |
| `POST /api/v1/database/compare-procedures` | Multipart: `arquivo` | `DatabaseSchemaComparisonService.CompararProceduresAsync`; compara corpos de procedures (20 MB), não gera script |
| `POST /api/v1/database/generate-correction-scripts` | `GerarScriptsRequest` | `SqlScriptGeneratorService.GerarScripts`; gera somente criações, não executa |
| `POST /api/v1/database/generate-correction-scripts-bulk` | `GerarScriptsBulkRequest` | `SqlScriptGeneratorService.GerarScriptsBulk`; gera scripts, não executa |
| `POST /api/v1/database/validate-script` | `ValidarScriptRequest` | `SqlScriptGeneratorService.ValidarScriptAsync`; validação estática, não executa |

Os DTOs do módulo estão em [`DatabaseDtos.cs`](../../../../backend/Central_BackEnd/Dtos/Database/DatabaseDtos.cs). A comparação de schemas devolve `SchemaArquivo` e `SchemaJca` (quando disponíveis), que a geração de scripts usa para classificar a compatibilidade dos tipos; `SchemaInfoDto` carrega a chave primária em `Pk` (`SchemaPkInfoDto` com `nome` e `colunas`) e `SchemaDifferenceDto.Categoria` aceita `Coluna`, `Tipo`, `Nullable`, `Indice`, `Fk`, `Pk` (novo) e `Tabela`; a comparação de procedures devolve `ProceduresComparisonResultDto` (`status` `Compativel`/`Divergente`/`SomenteBanco`/`SomenteArquivo`, com os dois corpos quando existem) e não passa pela geração de scripts.

#### SqlScriptGeneratorService

[`SqlScriptGeneratorService.cs`](../../../../backend/Central_BackEnd/Services/Database/SqlScriptGeneratorService.cs) (scoped, registrado em `Program.cs` junto aos demais serviços do módulo) transforma o resultado da comparação — enviado como JSON pronto, sem novo upload — em `SqlScriptResultDto`:

- **Tipos de script:** somente criações — `CREATE_TABLE`, `ADD_COLUMN`, `CREATE_INDEX` e `ALTER_FK`. No modo banco inteiro a saída é ordenada por tabela (ordem de processamento da comparação) e, dentro da tabela, por tipo (criação → coluna → índice/FK).
- **Direção:** o banco JCA é a fonte da verdade. Divergência, tabela/coluna só no JCA ou ausente no arquivo não geram script (seguem o padrão do banco); só o que existe no arquivo e falta no JCA vira script, montado a partir do schema do arquivo.
- **PK do `CREATE_TABLE`:** a constraint usa a PK declarada no arquivo (`arquivo.Pk`), com fallback para o índice cujo nome começa com `PK`; o índice de PK não é gerado de novo.
- **Metadados por script:** cada `SqlScriptDto` carrega `tabela` e `severidadeOrigem` (critico para criação de tabela/coluna; severidade da diferença de origem para índice e FK), usados pelo filtro interno do painel de scripts no frontend (hoje inativo: a tela envia `filtroSeveridade: null`). `ScriptsGenOpcoesDto` (`gerarBackup`/`modoEstrito`) continua no contrato por compatibilidade, mas não tem efeito.
- **`resumo`:** totais por categoria, impacto estimado (`Low` sem criar tabela, `Medium` com `CREATE_TABLE`), contagem de avisos e `revisaoManual` (apenas tabelas sem definição no arquivo, que impedem a montagem).
- **`ValidarScriptAsync`:** validação **estática** (nunca executa no SQL Server): limpa comentários/strings, aplica allowlist de verbos e objetos, emite avisos para `DROP TABLE/COLUMN`, `TRUNCATE` e `DELETE`/`UPDATE` sem `WHERE`, e confere a existência de tabelas por consulta leve em `INFORMATION_SCHEMA.TABLES` apenas quando a conexão está disponível.


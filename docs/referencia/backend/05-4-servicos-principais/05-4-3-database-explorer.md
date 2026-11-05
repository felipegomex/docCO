---
id: 05-4-servicos-principais.05
titulo: "4.3 Database Explorer"
tipo: referencia
area: backend
sistema: central
tags: [backend, services, di, efcore]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Configuration
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1035
ficha: docs/modulos/database-explorer.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### 4.3 Database Explorer

| Serviço | Responsabilidade | Dependências | Validações/efeitos |
|---|---|---|---|
| `DatabaseConnectionService` | Configuração e abertura de conexão SQL independente | `IConfiguration`, logger; singleton | Variáveis `DB_EXPLORER_*` têm precedência; exige servidor, banco e usuário; `Encrypt` e `TrustServerCertificate` são lidos da configuração, com padrões `false` e `true`; a configuração não é gravada em runtime |
| `DatabaseMetadataService` | Tabelas, colunas, índices, FKs, PKs, procedures, triggers, dependências e extração de schemas | `IDatabaseConnectionService`, logger | Consultas de metadados/SELECT; devolve corpos de procedures/triggers e a listagem de corpos usada pela comparação de procedures (`ListarCorposProceduresAsync`, só objetos do usuário); `ListarPkAsync` extrai a PK (`sys.key_constraints` type `PK`) usada por `ExtrairSchemasAsync`/comparação; contagem sem permissão retorna `-1`; nenhuma escrita de dados |
| `DatabaseSearchService` | Busca unificada de objetos | Conexão | Termo vazio retorna vazio; `take` limitado a 1–500; somente leitura |
| `DatabaseRelationshipInferenceService` | FKs confirmadas, candidatas, uso de coluna e grafo | Conexão, logger | Candidatas usam nome, tipo, PK e índice; grafo limita profundidade a 1–5; somente leitura |
| `DatabaseQueryBuilderService` | Monta SQL a partir de metadados e relacionamentos | Conexão, metadata, inferência, logger | Limita a cinco tabelas, valida tabelas e `HAVING`; gera texto SQL, não executa a consulta |
| `DatabaseSchemaComparisonService` | Compara tabelas com JSON (colunas, tipos, nullable, índices, FKs e PK) e compara corpos de procedures | `IDatabaseMetadataService`, `IDatabaseConnectionService`, logger | Exige arquivo não vazio, extensão `.json`, 5 MB por tabela, 20 MB no bulk ou nas procedures; ordem das colunas **não** é considerada diferença; a PK é lida de `pk`/`primaryKey`/`primary_key`/`PK` no JSON (propriedade ausente = JSON antigo, comparação de PK pulada; `null` explícito = sem PK), com severidade Critico para divergência/PK só no JCA/PK ausente no JCA e Aviso para PK só no arquivo; nas procedures a chave é `schema.nome` (sem distinção de caixa), o corpo é normalizado só em quebras de linha/espaço à direita e a comparação é somente leitura (sem scripts); **o parse de bulk nunca perde linha em silêncio**: o unwrap Excel/SSMS (aspas externas + `""`→`"`) só roda quando o conteúdo é um único valor lógico — em JSON Lines/CSV-Lines cada linha tem unwrap próprio; linha que parece JSON mas não parseia vira `ArgumentException` com o número da linha e um trecho (mapeado a 400); linha sem cara de JSON (cabeçalho, “N linhas afetadas”) é ignorada sem erro; o mesmo vale para procedures; **caracteres de controle crus (0x00-0x1F, exceto `\t\n\r`) dentro de string JSON não invalidam o arquivo**: o `TentarParseJson` (Regex `ControlesInvalidosJson`) reparseia com escape `\u00XX` e o caractere é **preservado** no nome (roundtrip JSON → caractere real), aparecendo como divergência visível na comparação — o arquivo gerado pelo script do sistema pode ser enviado cru, sem edição manual; **todas as frases de existência** (tabela, coluna, índice, FK e PK) nomeiam o banco conectado (`DatabaseConnectionService.GetConfig().Banco`), repassado ao método `Comparar` pelo parâmetro `nomeBanco` (`NomeBancoConectado`), no padrão `"X existe no banco {banco} mas nao existe no banco importado"` / `"X existe no banco importado mas nao existe no banco {banco}"`, e tratam o upload como banco importado |
| `SqlScriptGeneratorService` | Gera e valida estaticamente scripts de correção a partir do resultado da comparação | `IDatabaseConnectionService`, logger | Scoped; não executa SQL nem persiste nada; gera somente criações vindas do arquivo (o banco JCA é a fonte da verdade) |


---
id: mod.database-explorer
titulo: "Database Explorer"
tipo: modulo
area: dados
sistema: central
tags: [database, sqlserver, schema, tabela, coluna, indice, procedure, trigger, comparacao, script]
dono: dados
status: ativo
revisado_em: "2026-10-29"
fonte: [backend/Central_BackEnd/Controllers/Database/DatabaseController.cs, backend/Central_BackEnd/Services/Database, frontend/src/app/features/database]
testes: [backend/Central_BackEnd.Tests/DatabaseSchemaComparisonParseTests.cs]
tokens_estimados: 1700
---

# Database Explorer

## 1. Quando usar este card

Explorar tabelas/colunas/índices, relacionamentos e grafo, busca no schema,
procedures/triggers, dependências, comparação de schema com arquivo, geração e
validação de scripts, configuração de conexão.

## 2. Propósito e escopo

Ferramenta de leitura do SQL Server: metadados, relacionamentos inferidos, busca
e comparação de schema. **Não executa SQL de escrita**: gera script e valida
sintaxe; a execução é fora da Central.

## 3. Rotas do frontend

| URL | Componente | Fonte |
|---|---|---|
| `/database` | redirect para `/database/visao-geral` | `frontend/src/app/features/database/database.routes.ts` |
| `/database/visao-geral` | `DbVisaoGeralComponent` | idem |
| `/database/explorador` | `DbExploradorComponent` | idem |
| `/database/relacionamentos` | `DbRelacionamentosComponent` | idem |
| `/database/consultas` | `DbConsultasComponent` | idem |
| `/database/diferencas` | `DbDiferencasComponent` | idem |
| `/database/configuracao` | `DbConfiguracaoComponent` | idem |
| `/database/tabela/:schema/:tabela` | `TableDetailComponent` | idem |

A rota exige `aba.ferramentas.banco-dados` (`frontend/src/app/features.routes.ts`).

## 4. Endpoints do backend

`DatabaseController` em `/api/v{version}/database`, `[Authorize]`.

| Rotas | Observação | Fonte |
|---|---|---|
| `GET /status`, `/info`, `/config` | Estado e configuração | `backend/Central_BackEnd/Controllers/Database/DatabaseController.cs` |
| `GET /tables`, `/tables/{schema}/{nome}`, `/columns`, `/indexes` | Metadados | idem |
| `GET /relationships`, `/graph`, `/column-usage` | Relacionamentos | idem |
| `GET /search` | Busca no schema | idem |
| `GET /procedures`, `/procedures/{schema}/{nome}`, `/analysis` | Procedures | idem |
| `GET /triggers`, `/triggers/{schema}/{nome}`, `/dependencies` | Triggers e dependências | idem |
| `POST /query-builder-advanced` | Monta consulta | idem |
| `POST /compare-schemas`, `/compare-schemas-bulk`, `/compare-procedures` | Comparação | idem |
| `POST /generate-correction-scripts`, `/generate-correction-scripts-bulk` | Gera script | idem |
| `POST /validate-script` | Valida sintaxe | idem |

## 5. Tabelas e entidades

| Classe → tabela | Papel | Fonte |
|---|---|---|
| — → `sys.tables`, `sys.columns`, `sys.indexes`, `sys.foreign_keys`, `sys.procedures`, `sys.triggers` | Metadados lidos | `backend/Central_BackEnd/Services/Database/DatabaseMetadataService.cs` |
| — → `INFORMATION_SCHEMA` | Catálogo | idem |

Não persiste dados; a configuração vem de `DatabaseConnectionService`.

## 6. Serviços e lógica

| Camada | Serviço/peça | Responsabilidade | Fonte |
|---|---|---|---|
| backend | `DatabaseConnectionService` | Conexão e configuração sanitizada | `backend/Central_BackEnd/Services/Database/DatabaseConnectionService.cs` |
| backend | `DatabaseMetadataService` | Tabelas, colunas, índices, procedures, triggers | `backend/Central_BackEnd/Services/Database/DatabaseMetadataService.cs` |
| backend | `DatabaseRelationshipInferenceService` | FKs, grafo e uso de coluna | `backend/Central_BackEnd/Services/Database/DatabaseRelationshipInferenceService.cs` |
| backend | `DatabaseSearchService` | Busca no schema | `backend/Central_BackEnd/Services/Database/DatabaseSearchService.cs` |
| backend | `DatabaseSchemaComparisonService` | Comparação de schema | `backend/Central_BackEnd/Services/Database/DatabaseSchemaComparisonService.cs` |
| backend | `SqlScriptGeneratorService` | Gera e valida scripts | `backend/Central_BackEnd/Services/Database/SqlScriptGeneratorService.cs` |
| frontend | `DatabaseService` | HTTP do módulo | `frontend/src/app/features/database/services/database.service.ts` |
| frontend | `DatabaseShellComponent` | Shell/navegação | `frontend/src/app/features/database/database-shell.component.ts` |

## 7. Regras de negócio

- Conexão somente leitura para metadados; não há endpoint de DML.
- Configuração é sanitizada antes de sair (`DatabaseConnectionConfig.Sanitized`).
- Comparação é contra arquivo `.sql` enviado; o resultado é um plano.
- Script gerado precisa passar por `ValidateScriptAsync` antes de uso.
- Relacionamentos “possíveis” são inferência, não FK confirmada.
- Grafo respeita `profundidade` e `take`.

## 8. Autorização

- Policy `perm:aba.ferramentas.banco-dados:Ler` na classe; concedida aos cargos
  não-Master no SQL da matriz.
- Frontend exige `aba.ferramentas.banco-dados`; o backend revalida a permissão.
- Credencial nunca volta em claro; `Sanitized()` mascara.

## 9. Onde alterar

| Intenção | Backend (arquivo#símbolo) | Frontend (arquivo#símbolo) | Teste |
|---|---|---|---|
| “listar tabelas/colunas” | `backend/Central_BackEnd/Services/Database/DatabaseMetadataService.cs#ListarTabelasAsync` | `frontend/src/app/features/database/pages/db-explorador.component.ts#DbExploradorComponent` | — (sem teste dedicado) |
| “detalhe da tabela” | `backend/Central_BackEnd/Services/Database/DatabaseMetadataService.cs#ObterTabelaAsync` | `frontend/src/app/features/database/pages/table-detail.component.ts#TableDetailComponent` | — (sem teste dedicado) |
| “relacionamentos/grafo” | `backend/Central_BackEnd/Services/Database/DatabaseRelationshipInferenceService.cs#ListarGrafoAsync` | `frontend/src/app/features/database/pages/db-relacionamentos.component.ts#DbRelacionamentosComponent` | — (sem teste dedicado) |
| “busca no schema” | `backend/Central_BackEnd/Services/Database/DatabaseSearchService.cs#BuscarAsync` | `frontend/src/app/features/database/pages/db-consultas.component.ts#DbConsultasComponent` | — (sem teste dedicado) |
| “comparar schema” | `backend/Central_BackEnd/Services/Database/DatabaseSchemaComparisonService.cs#CompararComArquivoAsync` | `frontend/src/app/features/database/pages/db-diferencas.component.ts#DbDiferencasComponent` | `backend/Central_BackEnd.Tests/DatabaseSchemaComparisonParseTests.cs` |
| “gerar script” | `backend/Central_BackEnd/Services/Database/SqlScriptGeneratorService.cs#GerarScripts` | `frontend/src/app/features/database/pages/db-diferencas.component.ts#DbDiferencasComponent` | `backend/Central_BackEnd.Tests/DatabaseSchemaComparisonParseTests.cs` |
| “configuração” | `backend/Central_BackEnd/Services/Database/DatabaseConnectionService.cs#BuildConnectionString` | `frontend/src/app/features/database/pages/db-configuracao.component.ts#DbConfiguracaoComponent` | — (sem teste dedicado) |

## 10. Contratos que não podem quebrar

- Nenhum endpoint executa DML/DDL; só lê metadados e gera/valida texto.
- `/config` devolve credencial mascarada; não devolver senha.
- Comparação/geração usam o arquivo enviado e não alteram o banco; schema/nome
  vêm da rota e precisam de validação.

## 11. Armadilhas conhecidas

- Compat 100: prefixos e maiúsculas nos nomes do legado.
- Inferência pode sugerir FK inexistente; não promover a “confirmada”.
- Consulta dinâmica exige validação; `ValidateScript` valida sintaxe, não executa.

## 12. Testes

- Backend: `DatabaseSchemaComparisonParseTests` e `PermissoesContratosTests`.
- Frontend: sem spec das páginas.
- QA: `docs/referencia/qa/09-4-database.md`; troubleshooting em
  `docs/operacao/troubleshooting/10-9-database-explorer.md`.

## 13. Orçamento de leitura

- Abra primeiro esta ficha; depois o arquivo da seção 9 e o teste.
- Não abra `docs/arquitetura/banco-dados/INDEX.md` inteiro.
- Limite: 1 ficha, 4 trechos de código e ≤ 12 tool calls.

## 14. Pendências

- Metadados, busca e grafo sem teste dedicado; a policy é coberta por
  `PermissoesContratosTests`.

---
id: sec.04-fontes-de-verdade
titulo: "Fontes de verdade"
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
tokens_estimados: 730
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Fontes de verdade

| Fonte | Papel confirmado |
|---|---|
| `AppDbContext`, modelos e snapshot | Fonte do estado ORM atual; o snapshot representa o modelo final gerado pelas migrações. |
| `tboperador` | Fonte de operadores, perfil, estado, função e dados usados na autenticação. O campo de senha é sensível. |
| `tbcargo` + `tbperm_cargo` + `tbperm_operador` | **Fonte do RBAC.** `tboperador.CARGO_ID` aponta para `tbcargo`; a permissão efetiva vem de `tbperm_cargo` (cargo) e `tbperm_operador` (exceção individual, com a negativa vencendo). `GESTOR_MASTER` tem bypass. O catálogo de recursos fica em `tbpermissao_recurso`. |
| `tbfuncao` | **Eixo do Kanban Geral.** Fonte das funções usadas por perfil, por filtro e pela permissão de edição. A relação de `tboperador.FUNCAO_ID` pode ser nula, e nesse caso o operador só vê/edita o que o cargo `GESTOR_MASTER` ou `tbgestor` permitir. |
| `tbcliente` | Fonte de clientes do módulo de Implantação e base da tela Administrativa; `ProjetoService` consulta `ClienteLegado` e a migração `ClienteApontaTbCliente` aponta `PRJ_ClienteId` para essa tabela. Somente leitura no código de negócio, com a exceção do `INSERT` bruto no fechamento de lead. |
| `tbcliente_grupo` | Fonte dos grupos econômicos exibidos e filtrados na tela Administrativa **e fonte dos grupos do módulo Acessos Empresas** (substituiu a dropada `tbgrupoacesso`). Legada, somente leitura, fora das migrations. |
| `tbcontrato` | Fonte de licenças, valores, dia de faturamento e reajuste. Legada, somente leitura, fora das migrations; o schema real não tem `DATA_FATURAMENTO` e `QTDE_LICENCA` é `smallint`. |
| `tbchamado` | Fonte de chamados legados para busca e vínculo com tarefas; a tabela é espelhada sem escrita. |
| `tbfuncionario` | Fallback da função do operador quando `tboperador.FUNCAO_ID` está nulo (decisão do dono: divergências seguem `tboperador`, pois são ex-funcionários). `FUNCAO_ID` é `smallint` (enquanto `tboperador.FUNCAO_ID` é `int`). |
| Acessos Empresas | Fonte de unidades e credenciais do módulo (tabelas `tb*` geridas pelo `AppDbContext`); os **grupos** vêm da legada `tbcliente_grupo` e as **empresas** da legada `tbcliente` (typeahead/criação via ERP). |
| Database Explorer | Fonte externa de metadados do SQL Server configurado em `DatabaseExplorer`; não é o `AppDbContext` da aplicação. |
| Chat/Conhecimento | AnythingLLM e Claude são provedores externos; Gitea é o repositório versionado de documentos. A integração local persiste mapeamentos e auditorias, não conteúdo do chat. |


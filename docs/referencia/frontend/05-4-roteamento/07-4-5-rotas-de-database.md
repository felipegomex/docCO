---
id: 05-4-roteamento.07
titulo: "4.5 Rotas de Database"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, rotas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 224
ficha: docs/modulos/database-explorer.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### 4.5 Rotas de Database

| URL | Componente/redirect |
|---|---|
| `/database/visao-geral` | `DbVisaoGeralComponent` |
| `/database/explorador` | `DbExploradorComponent` |
| `/database/relacionamentos` | `DbRelacionamentosComponent` |
| `/database/consultas` | `DbConsultasComponent` |
| `/database/diferencas` | `DbDiferencasComponent` |
| `/database/configuracao` | `DbConfiguracaoComponent` |
| `/database/tabela/:schema/:tabela` | `TableDetailComponent` |

As seis primeiras telas são hijas de `DatabaseShellComponent`. O detalhe de tabela é lazy e fica fora do shell.


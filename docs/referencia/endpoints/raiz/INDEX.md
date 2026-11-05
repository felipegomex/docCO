---
id: raiz.indice
titulo: "Endpoints — Controllers na raiz"
tipo: referencia
area: api
sistema: central
tags: [api, endpoints, rotas]
status: ativo
revisado_em: "2026-10-30"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 300
ficha: docs/modulos/README.md
---

# Endpoints — Controllers na raiz

> Índice do documento dividido. Documento original: [raiz.md](../raiz.md).
> Ficha: [Fichas de módulo](../../../modulos/README.md).

Pasta `backend/Central_BackEnd/Controllers` (fora de subpastas): **16 classes** e **127 operações** `[Http*]`, contadas a partir dos atributos dos controllers atuais. As rotas são versionadas por `/api/v1`, exceto Chat (`/api/chat`) e Conhecimento (`/api/conhecimento`). Índice do catálogo em [`endpoints.md`](../../endpoints.md); seções por pasta: [`implantacao.md`](../implantacao/INDEX.md), [`dashboard.md`](../dashboard.md) e [`database.md`](../database.md).

Fatos de estrutura observados no código (2026-10-18; RBAC conferido em 2026-10-21):

- `ComercialController` tem **20 operações** (a mesma quantidade das linhas de tabela deste documento; a contagem "17" que constava no resumo antigo do catálogo ficou defasada).
- `ContextoProjetoController` (rota base `/ia`) não constava do índice anterior do catálogo.
- `FuncoesController` (raiz, rota `/funcoes`) também não constava do índice anterior: os dois atributos existem no código desde antes da conferência e o consumidor está mapeado na seção de Funções abaixo.
- As operações `GET /gestores/{id}` e `GET /gestores/resumo` existem no código e não tinham linha no catálogo anterior.
- O RBAC acrescentou `PermissoesController` (hoje 5), `CargosController` (1) e quatro operações de escrita em `OperadoresController` (que passou de 2 para 6); as policies são `perm:<recurso>:<acao>` e o recurso das telas de Sistema é `aba.sistema.operadores`/`aba.sistema.funcoes-administrativas`.

Suma por controller (operações `[Http*]`):

| Controller | Operações | Rota base |
|---|---:|---|
| `AuthController` / autenticação | 4 | `/api/v1/auth` |
| `AcessosController` / Acessos Empresas | 31 | `/api/v1/acessos` |
| `AuditoriaController` / auditoria | 2 | `/api/v1/auditoria` |
| `AgendaController` / agenda | 10 | `/api/v1/agenda` |
| `ComercialController` / Dashboard Comercial | 20 | `/api/v1/dashboard/comercial` |
| `ContextoProjetoController` / contexto de projeto por IA | 1 | `/api/v1/ia` |
| `DocsController` / documentação sincronizada (docs-api) | 2 | `/api/v1/docs` |
| `FuncoesController` / Funções (cadastro de cargo) | 3 | `/api/v1/funcoes` |
| `GestoresController` / gestores de função | 6 | `/api/v1/gestores` |
| `IdeiasController` / Ideias | 16 | `/api/v1/ideias` |
| `IdeiasSeedController` / seed de Ideias | 1 | `/api/v1/ideias/seed` |
| `OperadoresController` / operadores | 6 | `/api/v1/operadores` |
| `PermissoesController` / permissões do operador | 5 | `/api/v1/permissoes` |
| `CargosController` / catálogo de cargos | 1 | `/api/v1/cargos` |
| `ChatSsoController` / Chat de conhecimento (SSO) | 1 | `/api/chat` |
| `ConhecimentoController` | 18 | `/api/conhecimento` |
| **Total** | **127** | — |

## Partes

- [Autenticação](./01-autenticacao.md)
- [Acessos Empresas](./02-acessos-empresas.md)
- [Auditoria](./03-auditoria.md)
- [Agenda](./04-agenda.md)
- [Dashboard Comercial (`ComercialController`)](./05-dashboard-comercial-comercialcontroller.md)
- [ContextoProjetoController (`/ia`)](./06-contextoprojetocontroller-ia.md)
- [DocsController (`/docs`) — leitura da documentação sincronizada](./07-docscontroller-docs-leitura-da-documentacao-sincronizada.md)
- [Funções (`FuncoesController` da raiz, rota `/funcoes`)](./08-funcoes-funcoescontroller-da-raiz-rota-funcoes.md)
- [GestoresController (`/gestores`)](./09-gestorescontroller-gestores.md)
- [Ideias (`IdeiasController`, `/ideias`)](./10-ideias-ideiascontroller-ideias.md)
- [IdeiasSeedController (`/ideias/seed`)](./11-ideiasseedcontroller-ideias-seed.md)
- [OperadoresController (`/operadores`)](./12-operadorescontroller-operadores.md)
- [PermissoesController (`/permissoes`)](./13-permissoescontroller-permissoes.md)
- [CargosController (`/cargos`)](./14-cargoscontroller-cargos.md)
- [Chat de Conhecimento](./15-chat-de-conhecimento.md)
- [Conhecimento e revisão documental](./16-conhecimento-e-revisao-documental.md)
- [Exemplos mínimos confirmados (raiz)](./17-exemplos-minimos-confirmados-raiz.md)
- [Erros confirmados (raiz)](./18-erros-confirmados-raiz.md)
- [Limitações conhecidas (histórico da pasta raiz)](./19-limitacoes-conhecidas-historico-da-pasta-raiz.md)

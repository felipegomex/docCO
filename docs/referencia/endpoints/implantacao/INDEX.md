---
id: implantacao.indice
titulo: "Endpoints — Controllers/Implantacao"
tipo: referencia
area: api
sistema: central
tags: [api, implantacao, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers/Implantacao
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 300
ficha: docs/modulos/kanban.md
---

# Endpoints — Controllers/Implantacao

> Índice do documento dividido. Documento original: [implantacao.md](../implantacao.md).
> Ficha: [Kanban Geral](../../../modulos/kanban.md).

Pasta `backend/Central_BackEnd/Controllers/Implantacao`: **11 classes** e **59 operações** `[Http*]`, conferidas em 2026-10-20 pela contagem recursiva dos atributos. Duas classes são declaradas em arquivos com outro nome: `GargalosController` é a segunda classe de `BloqueiosController.cs`, e `DashboardController` é a segunda classe de `TarefasController.cs`. A contagem é de métodos de ação com atributo HTTP, não de rotas Swagger ou de endpoints gerados pelo framework. Índice do catálogo em [`endpoints.md`](../../endpoints.md); seções por pasta: [`raiz.md`](../raiz/INDEX.md), [`dashboard.md`](../dashboard.md) e [`database.md`](../database.md).

Suma por controller (operações `[Http*]`):

| Controller | Operações | Rota base |
|---|---:|---|
| `TiposProjetoController` / Implantação | 5 | `/api/v1/implantacao/tipos-projeto` |
| `ColunasKanbanController` / Implantação | 6 | `/api/v1/implantacao/colunas-kanban` |
| `FuncoesController` / Kanban Geral | 1 | `/api/v1/implantacao/funcoes` |
| `EtapasFuncaoController` / Kanban Geral | 6 | `/api/v1/implantacao/etapas-funcao` |
| `BloqueiosController` / Kanban Geral | 3 | `/api/v1/implantacao/bloqueios` |
| `GargalosController` (segunda classe de `BloqueiosController.cs`) / Kanban Geral | 1 | `/api/v1/implantacao/gargalos` |
| `KanbanController` / Kanban Geral | 2 | `/api/v1/implantacao/kanban` |
| `ProjetosController` / Implantação | 17 | `/api/v1/implantacao/projetos` |
| `TarefasController` / Implantação | 16 | `/api/v1/implantacao/tarefas` |
| `DashboardController` (segunda classe de `TarefasController.cs`) / Implantação | 1 | `/api/v1/implantacao/dashboard` |
| `AdminDashboardController` / administração | 1 | `/api/v1/admin/dashboard` |
| **Total** | **59** | — |

## Partes

- [Tipos de projeto](./01-tipos-de-projeto.md)
- [Colunas Kanban](./02-colunas-kanban.md)
- [Kanban Geral — quadro, funções, etapas da função, bloqueios e gargalos](./03-kanban-geral-quadro-funcoes-etapas-da-funcao-bloqueios-e-gar.md)
- [Projetos](./04-projetos.md)
- [Tarefas](./05-tarefas.md)
- [Dashboard de Implantação](./06-dashboard-de-implantacao.md)
- [Dashboard administrativo](./07-dashboard-administrativo.md)
- [Exemplos mínimos confirmados (Implantação)](./08-exemplos-minimos-confirmados-implantacao.md)
- [Erros confirmados (Implantação)](./09-erros-confirmados-implantacao.md)
- [Limitações confirmadas de consulta e cálculo (Implantação)](./10-limitacoes-confirmadas-de-consulta-e-calculo-implantacao.md)
- [Limitações de contrato a corrigir (Implantação)](./11-limitacoes-de-contrato-a-corrigir-implantacao.md)

---
id: implantacao.09
titulo: "Erros confirmados (Implantação)"
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
tokens_estimados: 164
ficha: docs/modulos/kanban.md
---
> Parte de [Endpoints — Controllers/Implantacao](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

## Erros confirmados (Implantação)

| Situação | Retorno explicitamente tratado |
|---|---|
| Tarefa com projeto sem etapa válida | `400 {"mensagem":"Etapa do projeto é obrigatória para tarefas de projeto"}` ou mensagem de etapa inválida. |

Não se deve transformar uma mensagem de erro textual em novo contrato; o código pode mudar.


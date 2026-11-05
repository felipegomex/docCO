---
id: implantacao.10
titulo: "Limitações confirmadas de consulta e cálculo (Implantação)"
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
tokens_estimados: 386
ficha: docs/modulos/kanban.md
---
> Parte de [Endpoints — Controllers/Implantacao](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

## Limitações confirmadas de consulta e cálculo (Implantação)

- `ProjetoService.ListarAsync` aplica corte `.Take(500)`; `GET /implantacao/projetos/com-etapas` parte dessa mesma listagem e herda o corte.
- `TarefaService.ListarAsync` aplica corte `.Take(500)`.
- `equipe` é aceito no dashboard e na listagem de tarefas, mas não participa dos filtros ou cálculos; `funcaoClassificacao` é aceito na listagem de tarefas e também não altera a consulta. `funcaoId` é filtro efetivo na listagem de tarefas, no dashboard de Implantação e na listagem de projetos; o dashboard e o detalhe/etapas de projeto aplicam `PodeVerFuncao`/`PodeLerAsync` e devolvem `403` para setor não permitido.
- O campo `atrasado` (resumo de projeto, resumo de tarefa e card do quadro) segue **uma regra só**, calculada no backend: tarefa = prazo (`dataEntrega ?? dataPrevisao`) vencido, **exceto** concluída dentro do prazo (cancelada nunca conta); projeto = **prazo vencido** e não concluído/cancelado; etapa = atraso em dias com referência `DataFimReal` ou hoje. O resumo de tarefa expõe **só a decisão** (não `diasAtraso`) e `ProjetoDetalhe` não tem `atrasado`.
- `diasRetro` é aceito no dashboard administrativo, mas não participa dos cálculos.


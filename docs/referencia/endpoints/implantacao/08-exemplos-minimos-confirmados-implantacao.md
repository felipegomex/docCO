---
id: implantacao.08
titulo: "Exemplos mínimos confirmados (Implantação)"
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
tokens_estimados: 331
ficha: docs/modulos/kanban.md
---
> Parte de [Endpoints — Controllers/Implantacao](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

## Exemplos mínimos confirmados (Implantação)

Os placeholders abaixo não são segredos nem valores de configuração.

### Tarefa sem projeto

Uma tarefa pode ser criada sem `projetoId`; nesse caso a validação não exige `projetoEtapaId`. Com projeto, o mesmo `projetoEtapaId` precisa pertencer ao projeto. A criação atual atribui o valor recebido mesmo sem projeto; a atualização zera a etapa quando não há projeto.

```json
{
  "projetoId": null,
  "projetoEtapaId": null,
  "colunaKanbanId": null,
  "chamadoLegadoId": null,
  "titulo": "Ajustar importação de dados",
  "descricao": "Tarefa de exemplo",
  "responsavelId": "operador.exemplo",
  "responsavelIds": ["operador.exemplo"],
  "criadorId": "operador.exemplo",
  "prioridade": 1,
  "tipo": 0,
  "ordem": 0,
  "dataPrevisao": "2026-09-30",
  "dataEntrega": null,
  "horasEstimadas": 4
}
```

### Apontamento

```json
{
  "operadorId": "operador.exemplo",
  "data": "2026-09-24T13:00:00",
  "horas": 1.5,
  "observacao": "Análise inicial"
}
```


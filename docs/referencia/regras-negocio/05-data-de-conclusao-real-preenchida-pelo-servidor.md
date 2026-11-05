---
id: regras-negocio.05
titulo: "Data de conclusão real — preenchida pelo servidor"
tipo: referencia
area: negocio
sistema: central
tags: [regras, negocio, permissoes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 417
ficha: docs/modulos/README.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## Data de conclusão real — preenchida pelo servidor

**Situação: implementada; o cliente não informa a data.**

- `TarefaCriarRequest` e `TarefaAtualizarRequest` **não têm** `DataConclusao`: a data de conclusão real é **derivada do status/coluna** e gravada pelo `TarefaService`.
- A tarefa **nasce** com a data nula e a registra se já nascer `Concluida`; uma **edição comum preserva** a data já registrada; e a data **some** quando a tarefa sai de `Concluida` — pela derivação do status/coluna e pela invariante final do serviço. O movimento no Kanban (`PATCH .../coluna`) segue a mesma regra.
- Consequência: a data **não "força" mais a conclusão**. A regressão travada por [`TarefaDataConclusaoTests`](../../../backend/Central_BackEnd.Tests/TarefaDataConclusaoTests.cs) é o contrato inverso — o DTO expunha `DataConclusao`, o formulário permitia digitá-la e o `PUT` apagava a data de uma tarefa concluída quando o campo não vinha.
- O campo **continua nas respostas** (`TarefaDetalhe`, `TarefaResumo`, `CardKanban`), mas é **somente leitura**: o formulário da tarefa mostra o valor desabilitado ("Definida pelo sistema quando a tarefa é concluída") e o Kanban não o envia.
- É a entrada da exceção "concluída dentro do prazo" da seção anterior: tarefa concluída **sem** data conta como atrasada, porque não dá para provar que ficou no prazo.


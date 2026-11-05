---
id: implantacao.04
titulo: "Projetos"
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
tokens_estimados: 1135
ficha: docs/modulos/kanban.md
---
> Parte de [Endpoints — Controllers/Implantacao](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

## Projetos

Fonte: [ProjetosController.cs](../../../../backend/Central_BackEnd/Controllers/Implantacao/ProjetosController.cs). O controller tem `[Authorize]` e `validacao` no nível de classe; a ação de clientes usa `leitura`.

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `GET` | `/api/v1/implantacao/projetos` | JWT | `validacao` (controller) | Query `tipo`, `status`, `clienteId?`, `responsavelId?`, `buscar?`, `perfilId?`, `funcaoId?` | `200 List<ProjetoResumo>`; corte de 500 registros. Cada resumo traz **`atrasado`** — decisão do backend (prazo do projeto vencido e não concluído/cancelado), para o cliente consumir a decisão. Quando o operador não é `GESTOR_MASTER`, o serviço cruza a função pedida com `FuncoesDeLeitura` (a própria função + as que ele gerencia) e devolve lista vazia para setor não permitido. |
| `GET` | `/api/v1/implantacao/projetos/com-etapas` | JWT | `validacao` (controller) | Mesmos filtros da listagem | `200 List<ProjetoComEtapasResumo>`, com resumo (com o mesmo `atrasado`) e etapas; corte de 500 projetos. |
| `GET` | `/api/v1/implantacao/projetos/{id}` | JWT + `PodeLerAsync` | `validacao` (controller) | Path `id`; sem body | `200 ProjetoDetalhe`, `403` quando o projeto é de setor fora das funções permitidas ou `404`. |
| `GET` | `/api/v1/implantacao/projetos/etapas-padrao` | JWT | `validacao` (controller) | Sem body | `200 List<object>` com `ordem` e `nome` das nove etapas. |
| `GET` | `/api/v1/implantacao/projetos/{id}/etapas` | JWT + `PodeLerAsync` | `validacao` (controller) | Path `id`; sem body | `200 List<ProjetoEtapaResumo>` ou `403` para setor não permitido. |
| `GET` | `/api/v1/implantacao/projetos/{id}/etapas/{ordem}` | JWT + `PodeLerAsync` | `validacao` (controller) | Path `id`, `ordem`; sem body | `200 ProjetoEtapaDetalhe`, `403` para setor não permitido ou `404`. |
| `GET` | `/api/v1/implantacao/projetos/proximo-codigo` | JWT | `validacao` (controller) | Sem body | `200 { codigo }` com a prévia `PRJ-####`. |
| `GET` | `/api/v1/implantacao/projetos/clientes` | JWT | `leitura` (ação) | Sem body | `200 List<ClienteResumo>` de clientes legados ativos. |
| `POST` | `/api/v1/implantacao/projetos` | JWT + `PodeCriarFuncao` | `validacao` (controller) | Body `ProjetoCriarRequest` (inclui `FuncaoId?`) | `201 ProjetoDetalhe`; o controller inicializa as etapas depois de criar. `FuncaoId` é opcional: quando enviado, precisa existir/estar ativo e passar em `PodeCriarFuncao`; sem ele, o projeto herda a função do criador. `400 "Coluna inválida para a função do projeto."` se `colunaKanbanId` **não estiver visível no quadro** da função do projeto. |
| `PUT` | `/api/v1/implantacao/projetos/{id}` | JWT | `validacao` (controller) | Body `ProjetoAtualizarRequest` | `200 ProjetoDetalhe` ou `404`; mesma validação de coluna do `POST` (`400 "Coluna inválida para a função do projeto."` quando a coluna não está visível no quadro do setor). |
| `PUT` | `/api/v1/implantacao/projetos/{id}/etapas/{ordem}` | JWT | `validacao` (controller) | Body `ProjetoEtapaAtualizarRequest` | `200 ProjetoEtapaDetalhe`. |
| `POST` | `/api/v1/implantacao/projetos/{id}/etapas/retornar` | JWT | `validacao` (controller) | Body `ProjetoEtapaRetornoRequest` | `200 ProjetoEtapaDetalhe`. |
| `POST` | `/api/v1/implantacao/projetos/{id}/etapas/{ordem}/checklist` | JWT | `validacao` (controller) | Body `ProjetoEtapaChecklistItemRequest` | `200 ProjetoEtapaDetalhe`. |
| `POST` | `/api/v1/implantacao/projetos/{id}/etapas/{ordem}/documentos` | JWT | `validacao` (controller) | Body `ProjetoEtapaDocumentoRequest` | `200 ProjetoEtapaDetalhe`. |
| `POST` | `/api/v1/implantacao/projetos/{id}/etapas/{ordem}/comentarios` | JWT | `validacao` (controller) | Body `ProjetoEtapaComentarioRequest` | `200 ProjetoEtapaDetalhe`. |
| `PATCH` | `/api/v1/implantacao/projetos/{id}/coluna` | JWT | `validacao` (controller) | Body `CardMoverColunaRequest` (`colunaKanbanId`) | `204` ou `404`; `403` quando o projeto é de outra função. Delega ao mesmo `KanbanBoardService.MoverCardAsync("projeto", …)` usado pelo quadro. |
| `DELETE` | `/api/v1/implantacao/projetos/{id}` | JWT | `validacao` (controller) | Path `id`; sem body | `204` ou `404`. |


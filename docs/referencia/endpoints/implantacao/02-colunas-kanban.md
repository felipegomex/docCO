---
id: implantacao.02
titulo: "Colunas Kanban"
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
tokens_estimados: 916
ficha: docs/modulos/kanban.md
---
> Parte de [Endpoints — Controllers/Implantacao](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

## Colunas Kanban

Fonte: [ColunasKanbanController.cs](../../../../backend/Central_BackEnd/Controllers/Implantacao/ColunasKanbanController.cs).

O **catálogo** de colunas (`tbcoluna` + `kanban`; colunas com `CLK_FuncaoId` nulo) é **compartilhado**: as mesmas colunas para todos os setores, porque a análise comparativa entre funções exige o mesmo conjunto. O que o setor controla é a sua **visão** (`tbcolunafuncao`): `CLF_Visivel` oculta/ativa a coluna no quadro do setor e `CLF_NomeAlternativo` é a **máscara de nome** (como a coluna aparece naquele setor, sem renomear a do catálogo).

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `GET` | `/api/v1/implantacao/colunas-kanban` | JWT | — | Query `apenasAtivas=true`, `funcaoId?`, `todas?` | `todas=true` → `200 List<ColunaKanbanResumo>` com o **catálogo inteiro** (tela de administração); `funcaoId=N` → `200 List<ColunaKanbanQuadro>` com a **visão do setor** (catálogo ativo filtrado pela visibilidade, com a máscara aplicada no campo `nome` e o nome do catálogo em `nomeCatalogo`); sem parâmetros → `200 List<ColunaKanbanResumo>` do catálogo ativo. |
| `GET` | `/api/v1/implantacao/colunas-kanban/visao/{funcaoId}` | JWT + **gestor da função ou administrador** | — | Path `funcaoId`; sem body | `200 ColunaKanbanVisaoResposta` — o catálogo **completo** (inclusive as colunas inativas) com o que o setor definiu (`visivel`, `nomeAlternativo`); `403` para quem não é gestor da função nem administrador. |
| `PUT` | `/api/v1/implantacao/colunas-kanban/visao/{funcaoId}` | JWT + **gestor da função ou administrador** | — | Body `ColunaKanbanVisaoSalvarRequest` (`funcaoId`, `colunas[]` com `colunaKanbanId`, `visivel`, `nomeAlternativo?`, `usuarioAlteracao`) | `200 ColunaKanbanVisaoResposta` (a visão já gravada); `400 { mensagem }` para coluna fora do catálogo (`Coluna fora do catálogo: …`), função inexistente/inativa (`Função informada não existe.`) ou `funcaoId` do corpo divergente da rota (`FuncaoId do corpo diverge da rota.`); `403` sem permissão. |
| `POST` | `/api/v1/implantacao/colunas-kanban` | JWT + Administrador (`Roles = "Administrador"`) | — | Body `ColunaKanbanCriarRequest` (`nome`, `ordem`, `cor?`, `limiteWip?`, `statusCard?`, `sincronizaAgenda?`, `usuarioInclusao`) | `200 ColunaKanbanDetalhe` efetivo — a coluna nasce **do catálogo** (`FuncaoId = null`); `400 { mensagem }` para `InvalidOperationException`/`ArgumentException`. |
| `PUT` | `/api/v1/implantacao/colunas-kanban/{id}` | JWT + Administrador | — | Body `ColunaKanbanAtualizarRequest` | `200 ColunaKanbanDetalhe` efetivo ou `404`; `400 { mensagem }` — entre outros, **desativar coluna com cards** ("Não é possível desativar: existem cards nesta coluna."). |
| `POST` | `/api/v1/implantacao/colunas-kanban/reordenar` | JWT + Administrador | — | Body `ColunaKanbanReordenarRequest` (`idsEmOrdem` na nova ordem) | `204` após renumerar o **catálogo inteiro** (uma ordem só, para todos os setores); `400 { mensagem }` para sequência vazia, id repetido ou id inexistente. |

⚠️ **`DELETE /api/v1/implantacao/colunas-kanban/{id}` foi removido.** Não existe mais exclusão de coluna: o caminho de saída é **desativar** (`CLK_Ativa`), e ele é **global** — a coluna sai do catálogo inteiro, não só de um setor. O backend recusa desativar coluna que tenha cards.


---
id: implantacao.03
titulo: "Kanban Geral — quadro, funções, etapas da função, bloqueios e gargalos"
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
tokens_estimados: 1789
ficha: docs/modulos/kanban.md
---
> Parte de [Endpoints — Controllers/Implantacao](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

## Kanban Geral — quadro, funções, etapas da função, bloqueios e gargalos

Fontes: [KanbanController.cs](../../../../backend/Central_BackEnd/Controllers/Implantacao/KanbanController.cs), [FuncoesController.cs](../../../../backend/Central_BackEnd/Controllers/Implantacao/FuncoesController.cs), [EtapasFuncaoController.cs](../../../../backend/Central_BackEnd/Controllers/Implantacao/EtapasFuncaoController.cs), [BloqueiosController.cs](../../../../backend/Central_BackEnd/Controllers/Implantacao/BloqueiosController.cs) (que também declara `GargalosController`) e os DTOs de [KanbanGeralDtos.cs](../../../../backend/Central_BackEnd/Dtos/Implantacao/KanbanGeralDtos.cs).

Todos usam `[ApiVersion("1.0")]` e `[Authorize]`. O quadro valida `PodeVerFuncao` quando recebe o operador; a **escrita** passa por `FuncaoAuthorizationService` (Master, a própria função ou uma função gerida; origem sem função só para o Master), exceto o CRUD de etapas da função, restrito a `[Authorize(Roles = "Administrador")]`.

O eixo do quadro é **`tbfuncao`** (`GET /implantacao/funcoes?apenasAtivos=`). O antigo eixo Setor foi removido: não existe mais `/implantacao/setores` nem `?setorId=` em nenhuma rota.

### Funções (1 operação, somente leitura)

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `GET` | `/api/v1/implantacao/funcoes` | JWT | — | Query `apenasAtivos=true` | `200 List<FuncaoKanbanResumo>` (`id`, `descricao`, `classificacao`, `ativo`, `podeEditar`, `totalOperadores` de operadores **ativos**), **na ordem de `FUNCAO_ID`** — que é a ordem dos pills. |

Não há `POST`/`PUT`/`DELETE` de função **por decisão**: `tbfuncao` é a tabela legada unificada de cargo e quem a mantém é o cadastro de operadores. As antigas rotas `GET/PUT /implantacao/setores/operadores` (atribuir setor) e o CRUD de setor foram removidos — nunca eram chamados pelo frontend.

Existe ainda um `FuncoesController` na raiz da pasta `Controllers`, com a rota `/api/v1/funcoes`: outra rota viva, com controller e serviço diferentes — ver [raiz.md](../raiz/INDEX.md).

### Etapas da função (fixas por decisão de negócio)

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `GET` | `/api/v1/implantacao/etapas-funcao` | JWT | — | Query `funcaoId?` | `200 List<EtapaFuncaoResumo>`. |
| `GET` | `/api/v1/implantacao/etapas-funcao/{id}` | JWT | — | Path `id` | `200 EtapaFuncaoResumo` ou `404`. |
| `POST` | `/api/v1/implantacao/etapas-funcao` | JWT + Administrador | — | Body `EtapaFuncaoCriarRequest` | `200 EtapaFuncaoResumo`; `400` para argumento inválido ou nome duplicado na função. |
| `PUT` | `/api/v1/implantacao/etapas-funcao/{id}` | JWT + Administrador | — | Body `EtapaFuncaoAtualizarRequest` | `200 EtapaFuncaoResumo` ou `404`. |
| `DELETE` | `/api/v1/implantacao/etapas-funcao/{id}` | JWT + Administrador | — | Path `id`; sem body | `204` ou `404`. |
| `POST` | `/api/v1/implantacao/etapas-funcao/reordenar` | JWT + Administrador | — | Body `EtapaFuncaoReordenarRequest` (`funcaoId`, `idsEmOrdem[]`) | `204`. |

⚠️ **O CRUD existe, mas o frontend não o expõe**: a tela de Administração do quadro apenas lista as etapas e informa que são fixas. Considere essas rotas contrato de backend, não funcionalidade de tela.

### Bloqueios (atributo do card)

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `GET` | `/api/v1/implantacao/bloqueios/{tipo}/{origemId}` | JWT | — | `tipo` = 0 Tarefa, 1 Projeto, 2 Etapa de projeto | `200 BloqueioResumo` (com `diasParado` calculado) ou **`204`** quando não há bloqueio aberto; `400` para tipo inválido. |
| `POST` | `/api/v1/implantacao/bloqueios/abrir` | JWT + regra de função | — | Body `BloqueioAbrirCardRequest` (`origemTipo`, `origemId`, `motivo`, `usuarioInclusao`) | `200 BloqueioResumo`; `400` para tipo inválido ou motivo vazio (`Motivo do bloqueio é obrigatório.`); `403` quando o item é de outra função. Se já houver bloqueio aberto, **atualiza o motivo** da linha aberta em vez de criar outra. |
| `POST` | `/api/v1/implantacao/bloqueios/{tipo}/{origemId}/fechar` | JWT + regra de função | — | Sem body; fecha a linha aberta | `200 BloqueioResumo` (com `diasParado` fechado) ou `204` se não havia bloqueio; `403` para outra função. |

### Gargalos

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `GET` | `/api/v1/implantacao/gargalos` | JWT | — | Query `funcaoId?` | `200 List<GargaloItem>` com bloqueios abertos de **tarefa, projeto e etapa**, ordenados do **mais parado** para o mais recente. Leitura aberta: o gestor precisa enxergar a operação toda. |

### Quadro e movimento de card

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `GET` | `/api/v1/implantacao/kanban/board` | JWT | — | Query `funcaoId?` (nulo = Todas no contrato; o Kanban Geral do front **não** envia nulo desde 2026-10-01 — a pill "Todas" foi removida), `tipo?` (`tarefas` \| `projetos` \| ausente = ambos), `limite?` (padrão **1000**) | `200 BoardKanban` com `funcaoId`/`funcaoNome`, `colunas` (do **catálogo compartilhado na visão do setor**: cada uma com `nome` efetivo — com a máscara do setor — e `nomeCatalogo`; `funcaoId`/`funcaoNome` da coluna vêm **nulos**), `cards` (tarefa **e** projeto no mesmo carrinho, com `funcaoClassificacao` para a cor do badge), `totalCards`, `totalBloqueados`, `truncado` e `limiteAplicado`; `400` para tipo inválido. Cada `CardKanban` traz **`atrasado`** (decisão do backend pela regra única de prazo) e **`dataConclusao`** (data de conclusão da tarefa — sem ela o cliente não consegue avaliar a exceção "concluída dentro do prazo"). A leitura **não** é filtrada por perfil. `modoTodos` foi **removido** do contrato. |
| `POST` | `/api/v1/implantacao/kanban/cards/{tipo}/{id}/coluna` | JWT + regra de função | — | Body `CardMoverColunaRequest` (`colunaKanbanId`, `novaOrdem?` — posição do card na coluna destino; desde 2026-10-07 o quadro envia `event.currentIndex` para que a reordenação na mesma coluna persista —); `tipo` = `tarefa` \| `projeto` | `204` ou `404`; `403` para item de outra função (`Sem permissão para alterar itens de outra função.`); `400` para tipo inválido, coluna inexistente, coluna inativa, coluna **não visível no quadro do setor** (`A coluna de destino não está visível neste quadro.`) ou WIP excedido. Tarefa **delega** ao `TarefaService` (com `novaOrdem`, ausente = `0`); projeto grava coluna e status. O mesmo movimento de **projeto** também está exposto em `PATCH /api/v1/implantacao/projetos/{id}/coluna`, que chama o mesmo serviço. |

O bloqueio **não** é COLUMN do quadro: a coluna "BLOQUEADO" está inativa, `tbtarefa.TRF_Bloqueada`/`TRF_MotivoBloqueio` não existem mais e mover um card nunca abre ou fecha bloqueio.


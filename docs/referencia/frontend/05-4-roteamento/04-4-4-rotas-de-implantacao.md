---
id: 05-4-roteamento.04
titulo: "4.4 Rotas de Implantação"
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
tokens_estimados: 1239
ficha: docs/modulos/kanban.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

### 4.4 Rotas de Implantação

| URL | Componente/redirect |
|---|---|
| `/implantacao/dashboard` | `DashboardComponent` |
| `/implantacao/kanban` | **redirect para `/kanban`** (o quadro de tarefa e projeto passou a ser o Kanban Geral) |
| `/implantacao/projetos` | `ProjetosComponent` |
| `/implantacao/projetos/novo` | `ProjetoFormComponent` |
| `/implantacao/projetos/:id/editar` | `ProjetoFormComponent` |
| `/implantacao/projetos/:id` | `ProjetoDetalheComponent` |
| `/implantacao/tarefas` | `TarefasComponent` |
| `/implantacao/tarefas/novo` | `TarefaFormComponent`; aceita `?projetoId=` e `?tarefaId=` |
| `/implantacao/tarefas/:id/editar` | `TarefaFormComponent`; o Projeto não é alterável |

A ordem de `projetos/:id/editar` antes de `projetos/:id` faz a rota de edição ser reconhecida antes do detalhe paramétrico.

`TarefaFormComponent` resolve o modo pelo snapshot da rota: `:id/editar` entra em edição, `/novo` permanece em create e, ainda em create, `?tarefaId=` carrega o detalhe da tarefa como formulário de duplicação (o `id` não é adotado, portanto o envio continua sendo `POST`). `?projetoId=` apenas pré-seleciona o projeto. A duplicação é disparada por `KanbanComponent.duplicarTarefa`.

**Select de coluna do catálogo (2026-10-20):** o form carrega `todasColunas` com `listarCatalogo(false)` (`GET /implantacao/colunas-kanban?todas=true&apenasAtivas=false`) e o dropdown lista o **catálogo** de colunas. O `computed` `colunasKanban` continua filtrando pela função efetiva (edição = função da tarefa; criação com projeto = função do projeto; criação sem projeto = função do operador), **mas o filtro deixou de ter efeito**: a coluna não pertence mais a uma função (`CLK_FuncaoId` é `NULL` no catálogo), então o fallback devolve o catálogo inteiro. A validação que vale é a do backend: a coluna precisa estar **visível no quadro do setor** (`ColunaVisaoExtensions`), e a tela não sabe se a coluna está oculta — quem oculta é o gestor, na tela de visão do setor.

**Regra de prazo — a decisão é do backend (2026-10-21):** "atrasado" tem **uma** regra, calculada no backend ([`RegrasPrazo`](../../../../backend/Central_BackEnd/Services/Shared/RegrasPrazo.cs)) e exposta no contrato. O frontend **não recalcula a decisão** — consome o `atrasado` que o backend devolve; as duas exceções são o helper `ehAtrasada`, que só recalcula como **reserva** quando o endpoint não traz o campo, e o **número de dias** do Dashboard, que o resumo não traz:

| Tela | O que consome |
|---|---|
| Projetos — aba "Atrasados" e ordenação "Atrasados Primeiro" | `projeto.atrasado` (`ProjetoResumo`); antes recalculava aqui ("qualquer etapa com `atrasoDias > 0`"), que é outra regra |
| Projetos — card (`ProjetoCardComponent`) | `projeto.atrasado` para o estado visual; a **criticidade** (pulso) continua vindo do maior atraso de **etapa** (`atrasoDias`), que é um sinal mais fino |
| Dashboard — "d de atraso" | `t.atrasado` decide se é atraso; o **número de dias** é calculado na tela porque o resumo não traz `diasAtraso` (as tarefas vêm de `GET /implantacao/tarefas`, que já traz o campo) |
| Kanban — métrica "atrasadas" | filtra por `t.atrasado` — no modo geral (o único alcançável) o card **recebe a decisão do backend**: `CardKanban.atrasado` → `paraCardQuadro` → card local (`TarefaQuadro`), que é o `t` do template do card |
| Kanban — rótulo de SLA do card ("Vencida") | `ehAtrasada` devolve o `atrasado` do backend quando o card o traz; o espelho local só entra no `TarefaDetalhe` do drawer, cujo DTO não expõe `atrasado` |

O helper [`ehAtrasada`](../../../../frontend/src/app/features/implantacao/models/tarefa.model.ts) ficou como **reserva**: quando a resposta traz `atrasado`, ele devolve o valor do backend; só quando o endpoint **não** devolve o campo ele aplica o espelho local (prazo = `dataEntrega ?? dataPrevisao` vencido, exceto concluída dentro do prazo; cancelada nunca conta). O caso que sobra é o `TarefaDetalhe` do drawer, cujo DTO não expõe `atrasado` — lá o espelho local é a única fonte.

**Estado do mapeamento (2026-10-21):** o modelo `CardKanban` do frontend ([`funcao.model.ts`](../../../../frontend/src/app/features/implantacao/models/funcao.model.ts)) já declara `dataConclusao` e `atrasado`, e a conversão `paraCardQuadro` do `KanbanComponent` repassa os dois para o card local (`TarefaQuadro`). Ou seja: no quadro geral a métrica "atrasadas" e o rótulo "Vencida" do card consomem a **decisão do backend**. O que **resta** é o `TarefaDetalhe` do drawer: o DTO não expõe `atrasado`/`diasAtraso`, então o rótulo "Vencida" **dentro do drawer** usa o espelho local do helper — o card do quadro, não.


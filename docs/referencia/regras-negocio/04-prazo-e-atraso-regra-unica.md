---
id: regras-negocio.04
titulo: "Prazo e atraso — regra única"
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
tokens_estimados: 1131
ficha: docs/modulos/README.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## Prazo e atraso — regra única

**Situação: implementado; a decisão é do backend, o cliente consome, e o helper `ehAtrasada` fica como reserva — só recalcula quando o endpoint não traz o campo.**

Até 2026-10-21, "atrasado" tinha **3 semânticas no backend** e **4 no frontend**, e as telas mostravam números discordantes: a aba "Atrasados" da tela de Projetos contava **etapa** atrasada, o Dashboard contava **prazo do projeto** e o Kanban usava a regra de tarefa. A regra passou a morar em um lugar só: [`RegrasPrazo`](../../../backend/Central_BackEnd/Services/Shared/RegrasPrazo.cs) (classe estática, sem DI), com comparação sempre por `.Date` (dias corridos). Desde a unificação, o frontend não tem mais regra própria: todas as telas consomem o `atrasado` do backend, e o único ponto que ainda usa o espelho local é o `TarefaDetalhe` do drawer, cujo DTO não expõe `atrasado` (ver frontend.md § 4.4).

| Objeto | Prazo | Atrasado quando |
|---|---|---|
| **Tarefa** | `DataEntrega ?? DataPrevisao` | o prazo já passou, **exceto** tarefa concluída **dentro** do prazo (concluída depois do prazo conta como atraso; concluída sem data de conclusão conta, porque não dá para provar que ficou no prazo); **cancelada nunca conta** |
| **Projeto** | `DataPrevisao` | o prazo já passou e o projeto **não** está concluído nem cancelado — concluído deixa de contar, e o projeto **não** tem a exceção de "concluído dentro do prazo" |
| **Etapa** | `DataFimPrevista` | atraso em dias = (referência − `DataFimPrevista`), com referência = `DataFimReal` ou hoje; etapa sem prazo previsto **não** tem atraso |

**Semântica escolhida para projeto: prazo vencido.** É a regra que o KPI "projetos atrasados" do Dashboard já usava; a tela de Projetos passou a contar pelo mesmo critério. A **criticidade** do card (pulso) continua vindo do maior atraso de **etapa** — é um sinal mais fino que a régua de prazo do projeto, e não é o indicador de "projeto atrasado".

Consequências confirmadas (tarefa):

| Situação | Resultado |
|---|---|
| `DataEntrega` e `DataPrevisao` ausentes | não atrasada |
| prazo hoje | não atrasada, pois a comparação é estritamente `<` |
| tarefa aberta com prazo anterior | atrasada |
| concluída até o prazo | não atrasada |
| concluída depois do prazo | atrasada |
| concluída sem data de conclusão | atrasada |
| cancelada | nunca conta como atrasada |
| `DataEntrega` e `DataPrevisao` preenchidas | `DataEntrega` manda |

**Quem aplica a regra:** `TarefaService` (filtro "vence hoje" e o `atrasado` do resumo), `DashboardService` (KPI de projetos atrasados e de tarefas atrasadas), `ProjetoEtapaService` (cálculo do `AtrasoDias`), `ProjetoService` (o `atrasado` do resumo) e `KanbanBoardService` (o `atrasado` e a `dataConclusao` do card).

**Filtro no banco:** listar atrasadas é uma consulta, então a regra também existe como **expressão** em [`TarefaAtrasoExtensions.OndeAtrasadas`](../../../backend/Central_BackEnd/Services/Implantacao/TarefaAtrasoExtensions.cs) — a mesma regra escrita para o EF traduzir. O teste de contrato [`RegrasPrazoTests`](../../../backend/Central_BackEnd.Tests/RegrasPrazoTests.cs) fixa as duas lado a lado, para a lista do banco não voltar a divergir do KPI.

**A decisão sobe no contrato:** `ProjetoResumo.atrasado`, `TarefaResumo.atrasado` e `CardKanban.atrasado` são a decisão pronta; `CardKanban` também traz `dataConclusao` (sem ela o cliente não consegue avaliar a exceção "concluída dentro do prazo"). O frontend deixou de recalcular; o helper `ehAtrasada` ficou como **reserva**, para endpoints que ainda não devolvem o campo. No quadro, o mapeamento no cliente **está feito** (2026-10-21): o modelo `CardKanban` do frontend e a conversão `paraCardQuadro` repassam `atrasado`/`dataConclusao` para o card local, então a métrica "atrasadas" e o rótulo "Vencida" do card consomem a decisão do backend; resta apenas o `TarefaDetalhe` do drawer, cujo DTO não expõe `atrasado` (frontend.md § 4.4).

**Atraso da etapa:** o `AtrasoDias` é recalculado **na sincronização por tarefas** (`SincronizarEtapasPorTarefasAsync`), com a mesma regra da edição manual. Antes só a edição manual recalculava, e o número que a tela lia ficava defasado por construção.


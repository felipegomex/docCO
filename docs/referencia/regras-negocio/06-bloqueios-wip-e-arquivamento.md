---
id: regras-negocio.06
titulo: "Bloqueios, WIP e arquivamento"
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
tokens_estimados: 462
ficha: docs/modulos/kanban.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Kanban Geral](../../modulos/kanban.md).

## Bloqueios, WIP e arquivamento

**Situação: parcial.** O bloqueio deixou de ser coluna e virou **atributo** do card; WIP e arquivamento seguem como antes.

- O bloqueio **não é mais coluna do quadro**: a coluna 22 "BLOQUEADO" foi desativada e `tbtarefa.TRF_Bloqueada`/`TRF_MotivoBloqueio` foram removidas do banco pelo script do Kanban Geral. O card permanece na coluna da **etapa** em que estava, e o estado vem da linha aberta de `tbbloqueio` (ver a seção "Kanban Geral" abaixo).
- O contrato HTTP **continua igual**: `TarefaCriarRequest`/`TarefaAtualizarRequest` seguem com `bloqueada` e `motivoBloqueio`, e `TarefaService` traduz o par para `IBloqueioService.AbrirAsync`/`FecharAsync`. Criar ou atualizar com `bloqueada = true` sem motivo é recusado (`Motivo do bloqueio é obrigatório para tarefa bloqueada.`).
- Abrir ou fechar o bloqueio **não move mais o card** entre colunas. Ao mover uma tarefa, a coluna enviada continua prevalecendo sobre o status, e a coluna BLOQUEADO deixou de derivar status.
- A movimentação para uma coluna com `LimiteWip` positivo é recusada quando a quantidade de tarefas na coluna já atingiu o limite. A contagem não filtra tarefas arquivadas.
- Somente tarefas com status `Concluida` podem ser arquivadas. Desarquivar força status `Concluida` e escolhe a coluna cujo nome contém `CONCLUID`, se existir.
- O alerta administrativo chamado `Bloqueadas` não conta o bloqueio: ele conta `Cancelada`, `Backlog` e `A Fazer`. Essa é uma divergência de regra que deve ser corrigida ou renomeada.


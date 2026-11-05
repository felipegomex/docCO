---
id: regras-negocio.09
titulo: "Etapas do projeto"
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
tokens_estimados: 669
ficha: docs/modulos/kanban.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Kanban Geral](../../modulos/kanban.md).

## Etapas do projeto

**Situação: implementado com fórmulas de progresso concorrentes.**

- A jornada padrão tem nove etapas, nesta ordem: `KICKOFF`, `LEVANTAMENTO`, `DESENVOLVIMENTO`, `HOMOLOGAÇÃO`, `TREINAMENTO`, `GO LIVE`, `PÓS-IMPLANTAÇÃO`, `PASSAR PARA O SUPORTE` e `CONCLUÍDO`.
- A criação do projeto inicializa as nove etapas, com checklists padrão. A ordem de início é 1–9: etapas anteriores nascem concluídas, a etapa inicial nasce em andamento e as seguintes pendentes.
- `ProjetoId + Ordem` é único. Checklist, documento, histórico e comentário dependem da etapa e são removidos em cascata.
- Tarefa com `ProjetoId` exige `ProjetoEtapaId` **do mesmo projeto**. `TarefaService.ValidarEtapaFixaAsync` exige a etapa (`Etapa do projeto é obrigatória para tarefas de projeto`) e rejeita etapa de outro projeto (`Etapa do projeto inválida para esta tarefa`); a validação é de aplicação, no serviço, e vale para criação e atualização.
- Tarefa sem projeto **e** sem etapa continua válida. Na criação, informar etapa sem projeto é rejeitado com mensagem explícita (`Não é possível informar uma etapa do projeto sem informar o projeto.`).
- Na atualização, o projeto não é trocado: `TarefaAtualizarRequest` não tem `ProjetoId` e `AtualizarAsync` não escreve `t.ProjetoId`. A etapa enviada é validada contra o projeto já persistido e, em tarefa sem projeto, é gravada como `null`.
- A sincronização por tarefas ignora tarefas arquivadas e calcula percentual por tarefas concluídas. Uma tarefa reaberta reabre a etapa afetada e, em cascata, as etapas posteriores `Concluida` passam a `Bloqueada` (histórico `Reabertura automática em cascata`, `DataFimReal` limpo). A conclusão automática (N/N) só ocorre quando a etapa anterior está `Concluida` e libera a próxima etapa `Pendente` ou `Bloqueada`, que volta a `EmAndamento` preservando `DataInicio`.
- O retorno manual aceita ordem alvo 1–8, reseta as etapas posteriores e mantém o checklist da etapa alvo. Não foi localizada regra que restrinja esse retorno por responsável.
- Existem duas rotinas de progresso: `ProjetoJornadaService` calcula média dos percentuais das etapas, enquanto `ProjetoEtapaService` calcula `(etapasConcluídas * 100 + percentualAtual) / 9`. A atualização direta de projeto ainda aceita `Progresso` e `Status` na requisição. Não há uma fórmula única garantida para todo fluxo.


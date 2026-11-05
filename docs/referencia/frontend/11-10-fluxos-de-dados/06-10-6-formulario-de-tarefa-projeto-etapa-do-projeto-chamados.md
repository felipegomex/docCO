---
id: 11-10-fluxos-de-dados.06
titulo: "10.6 Formulário de Tarefa — projeto, etapa do projeto, chamados e data de conclusão"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, fluxos]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 1136
ficha: docs/modulos/kanban.md
---
> Parte de [10. Fluxos de dados](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

### 10.6 Formulário de Tarefa — projeto, etapa do projeto, chamados e data de conclusão

`TarefaFormComponent` mantém `projetoId` e `projetoEtapaId` em signals e resolve o contrato de etapa sem sair do componente:

- O campo **Etapa do projeto (card)** é renderizado nos dois modos. Sem projeto ele fica desabilitado, com placeholder "Selecione um projeto primeiro" (ou "Tarefa sem projeto" na edição) e a dica de que é preciso escolher um projeto; com projeto ele mostra o placeholder de carregamento, o de erro ou a lista das nove etapas. O `required` é condicional (`[required]="temProjeto()"`).
- `carregarEtapasFixas` zera lista e etapa antes de cada chamada, marca `etapasLoading` e usa um contador de requisição: a resposta só é aplicada quando a sequência e o `projetoId` ainda conferem, portanto trocar ou remover o projeto descarta respostas antigas, inclusive quando a resposta obsoleta chega por `error`.
- Estados separados: `etapasError` (falha de comunicação, mensagem `Não foi possível carregar as etapas do projeto.`) e `etapasVazia` (projeto sem nenhum card cadastrado, `Nenhuma etapa cadastrada para este projeto.`). Projeto sem etapas não vira erro de comunicação e, nos dois casos, `podeSalvar` fica falso porque a etapa obrigatória não existe.
- Recuperação: com projeto, erro e carga parada, o campo `podeTentarEtapas` exibe o botão **Tentar novamente**, que chama `tentarNovamenteEtapas` e repete o `GET` do mesmo projeto. Ele existe também na edição, onde o seletor de Projeto está desabilitado — é o único caminho de recuperação quando a carga inicial falha ali.
- Default com projeto, isolado em `escolherEtapaPadrao`: etapa preservada se pertencer à lista, senão `EmAndamento` → primeira não concluída → primeira disponível.
- Reuso: ao preencher o formulário (carga inicial, duplicação e retorno do `PUT`), `preencherFormulario` só chama `carregarEtapasFixas` quando o projeto mudou ou a lista ainda não foi carregada; com etapas em memória — inclusive o estado vazio já conhecido — reaproveita a lista e reaplica o default **sem novo `GET`**.
- `podeSalvar` exige etapa válida (presente na lista, sem erro e sem carregamento) sempre que há projeto; sem projeto, a etapa não é exigida.
- Payload: o create envia `projetoId` e `projetoEtapaId`; o update envia `projetoEtapaId` e **não** envia `projetoId`. O select de Projeto é desabilitado na edição, com a dica de que o projeto não pode ser alterado, e a Etapa permanece habilitada. Tarefa sem projeto não envia etapa alguma no create.
- **Chamados (dropdown "atrelar chamado"):** a busca é **server-side** — `ChamadoDropdownComponent` chama `GET /api/v1/implantacao/tarefas/chamados/busca` (`take=50`, `clienteId` do projeto selecionado) com debounce de 300 ms e **não refiltra a resposta no cliente**; a lista da tela é exatamente o que a API devolveu. O `@Input() rotulos` (mapa id → título montado em `preencherFormulario` a partir de `TarefaDetalhe.Chamados`) faz o chip de um chamado já vinculado **continuar mostrando o título** quando a busca o esconde — é o caso do `STATUS "F"`, que o backend esconde por padrão.
- **Data de Conclusão Real — somente leitura:** o campo aparece **só quando o servidor devolveu data** (`*ngIf="dataConclusaoServidor()"`), o input é `readonly` + `disabled` com a dica "Definida pelo sistema quando a tarefa é concluída; não pode ser editada", e o formulário **não envia** o campo — `TarefaCriarRequest`/`TarefaAtualizarRequest` não o têm. O Kanban também não o envia. A data é formatada em TypeScript (`dd/MM/yyyy`), de propósito: o pipe `date` trata "yyyy-MM-dd" como UTC e viraria dia anterior no BRT.
- Acessibilidade: o select recebe `aria-busy` durante o carregamento e `aria-describedby="etapaFixa-mensagem"` sempre que há mensagem; a mensagem tem `role="alert"` no erro e `role="status"` nos demais casos (carregando, vazio, sem projeto).
- Ciclo de vida: a assinatura de etapas usa `takeUntilDestroyed(this.destroyRef)`; a classe implementa `OnDestroy` para limpar o temporizador do toast.
- O comportamento é coberto por [`tarefa-form.component.spec.ts`](../../../../frontend/src/app/features/implantacao/pages/tarefas/tarefa-form.component.spec.ts); a suíte não foi executada nesta revisão.


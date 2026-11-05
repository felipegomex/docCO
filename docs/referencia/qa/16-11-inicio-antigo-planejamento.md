---
id: sec.16-11-inicio-antigo-planejamento
titulo: "11. Início (antigo Planejamento)"
tipo: referencia
area: qa
sistema: central
tags: [qa, historias, testes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1388
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 11. Início (antigo Planejamento)

### PL-01 — Ver a semana de tarefas e agenda do operador logado

- **Objetivo:** Oferecer uma visão de planejamento semanal das tarefas do operador logado (admin incluído) e da agenda da semana exibida, com atalho para a edição da tarefa.
- **Pré-condições:** operador autenticado; tarefas filtradas por `responsavelId` = operador logado (responsável ou co-responsável, admin vê somente as próprias); eventos de agenda filtrados por `responsavelId` (somente eventos em que o operador é responsável, não participante); `dataEntrega`/`dataPrevisao` preenchidas nas tarefas que devem cair na semana.
- **Passos:** 1. Abrir `/` pelo menu Início. 2. Conferir os seis atalhos rápidos do topo (grade de 6 colunas numa linha; 3 colunas abaixo de 1400px e 1 coluna abaixo de 1100px) e a **faixa de resumo do dia** logo abaixo (pendentes, atrasadas, urgentes, compromissos de hoje). 3. Navegar a semana na seção **Minha semana** (‹ anterior, Hoje, › próxima, atualizar; chip "esta semana" quando na semana corrente, botão Hoje desabilitado nela e destacado fora dela). 4. Ler os dois cards do topo (mesma altura, ditada pelas 6 tarefas): **Lista de tarefas** (prévia de até 6 **pendências** do operador logado — sem bloqueadas, desempate por prioridade e prazo — com contagem total e link "Ver Todas >>") e **Agenda** (eventos de segunda a sexta da semana exibida em que o operador é responsável, com contador de compromissos, eventos ampliados e rolagem interna por coluna, link "Ver agenda"). 5. Conferir o painel **Não posso esquecer!**. 6. Clicar num item de tarefa (lista ou painel) para abri-lo em `/implantacao/tarefas/:id/editar`.
- **Resultado esperado:** a página `/` (Início, `HomeComponent` — a antiga `/planejamento` foi fundida aqui e a rota removida): hero com logo JCA, saudação por nome, busca global (`origem="home"`) e grade de **Acessos rápidos** (6 atalhos numa linha). Logo abaixo, a **faixa de resumo do dia** mostra quatro estatísticas (pendentes, atrasadas, urgentes, compromissos de hoje) derivadas dos mesmos signals do componente — atrasadas/urgentes em vermelho quando > 0 e "hoje" com acento `--signal`. A seção **Minha semana** traz a navegação de semana (rótulo `dd/mm a dd/mm`) e, em `grid-topo`, dois cards: **Lista de tarefas** (somente as **pendências** do operador logado — responsável ou co-responsável, admin incluído —, **excluindo concluídas, canceladas e arquivadas**; o status é comparado **sem acento**, porque o backend serializa `Concluida` e `Cancelada`; a prévia exibe até 6 itens via `tarefasVisiveis()` — sem bloqueadas, ordenadas por prioridade (Alta→Baixa) → prazo mais próximo, sem prazo por último; o contador do subtítulo e o estado vazio usam a **mesma lista** (`pendentesLista()`), que **exclui as tarefas Urgentes**, porque elas têm o card próprio do painel "Não posso esquecer!"; cada item é âncora para `/implantacao/tarefas/:id/editar` com hover/foco e seta; "Ver Todas >>" abre a lista completa) e **Agenda** (somente eventos em que o operador é responsável, em colunas segunda–sexta da semana exibida, dia corrente com rótulo "hoje", eventos ampliados com fundo tingido pela cor do evento, título em até 2 linhas e prioridade em cor de borda, dia sem eventos marcado como "livre"; altura do card acompanha as 6 tarefas, com rolagem interna por coluna; sem filtro de recorrência — é o que está na agenda para cada dia). Por último, o painel **Não posso esquecer!** lista **somente as tarefas Urgente em aberto** (não concluídas, não canceladas e não arquivadas) do próprio operador. Sem usuário logado a página não faz chamada HTTP (lista vazia até `currentUser$` emitir); o carregamento exibe **skeleton** (faixa + dois painéis com pulso), não spinner. **A seção "Quadro semanal" foi removida** (redundante com o card Agenda); não há `grid-3` nem cards "Pendentes"/"Próxima semana". Textos usam plural correto (helper `plural()`), a navegação tem `aria-label`, há `focus-visible` nos interativos e `prefers-reduced-motion` desliga animações. Falha de carga exibe estado de erro com **Tentar novamente**, sem dados fictícios. A página não edita inline — o clique abre o formulário da tarefa.
- **Estado atual/bloqueio:** sem limitação específica identificada; a edição inline não existe nesta versão. A prévia da lista exibe até 6 tarefas (a carga usa `limite=2000`; skeleton com 6 linhas); dias lotados da Agenda rolam por coluna sem estourar a altura.
- **Rotas/endpoints/tabelas:** `/` (a rota `/planejamento` foi removida; o fallback global `** → ''` leva a `/`); `GET /api/v1/implantacao/tarefas?responsavelId=&limite=2000` (responsável/co-responsável = operador logado, admin incluído; sem `responsavelId` a página não carrega) e `GET /api/v1/agenda/eventos?inicio=X&fim=Y&responsavelId=` com janela ampla de ±semanas em torno da semana exibida (‑28/+35 dias, para navegar sem nova chamada); `TarefasService.listar`, `AgendaService.listarEventos`, modelo `TarefaResumo`, helper `ehAtrasada` de `tarefa.model.ts`. Sem novo endpoint no backend — os dois já aceitam `responsavelId`.
- **Status/automação:** **cenário documentado**; não há teste automatizado desta tela.


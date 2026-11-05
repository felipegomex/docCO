---
id: sec.02-1-escopo-e-fontes
titulo: "1. Escopo e fontes"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, rotas, componentes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 1269
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/frontend.md](../frontend.md).

# 1. Escopo e fontes

| Fonte | Papel factual |
|---|---|
| [`frontend/package.json`](../../../frontend/package.json) | Angular 18, RxJS, Bootstrap, ng-bootstrap e Chart.js |
| [`frontend/angular.json`](../../../frontend/angular.json) | Builder application, browser, assets e estilos globais |
| [`frontend/tsconfig.json`](../../../frontend/tsconfig.json) | Aliases `@core`, `@layout`, `@shared`, `@features` e `@env` |
| [`frontend/src/main.ts`](../../../frontend/src/main.ts) | Bootstrap do `AppComponent` no navegador |
| [`frontend/src/main.ts`](../../../frontend/src/main.ts) | Bootstrap do browser (único bootstrap; SSR desativado) |
| [`frontend/src/app/app.routes.ts`](../../../frontend/src/app/app.routes.ts) | Rotas raiz e guards |
| [`frontend/src/app/features.routes.ts`](../../../frontend/src/app/features.routes.ts) | Rotas da área autenticada principal |
| [`frontend/src/app/features/admin/admin.routes.ts`](../../../frontend/src/app/features/admin/admin.routes.ts) | Rotas administrativas |
| [`frontend/src/app/features/implantacao/implantacao.routes.ts`](../../../frontend/src/app/features/implantacao/implantacao.routes.ts) | Rotas lazy de Implantação |
| [`frontend/src/app/features/database/database.routes.ts`](../../../frontend/src/app/features/database/database.routes.ts) | Rotas lazy do Database Explorer |
| [`frontend/src/app/features/historias-telas/`](../../../frontend/src/app/features/historias-telas/) | **Só o arquivo de dados** `historias-telas.data.ts` — gerado por `scripts/extract-historias-telas.ts` a partir de `docs/referencia/qa-historias.md` (fonte repontada na fase 5 do plano docs-api). **Não é editado à mão.** O componente e o template foram removidos em 2026-10-20 (Fase 0): `/historias-telas` virou redirect para `/faq-sistema`, que é a tela canônica do mesmo conteúdo |
| [`frontend/src/app/features/faq-sistema/`](../../../frontend/src/app/features/faq-sistema/) | Página `/faq-sistema` (plano docs-api, fase 4): catálogo em cards por tela do sistema — cada card responde "o que a tela faz", "o que usa" (endpoints/tabelas), "história passo a passo" e "estado atual", com badge de status; busca (tela/título/rota/grupo/objetivo/endpoints) + chips de filtro por grupo e por status + expandir/recolher. Fonte v1: a mesma `historiasTelas` de `historias-telas.data.ts`; fonte v2 prevista: `GET /api/v1/docs` sobre `tbdoc_doc` (trocando a fonte no service, não no template). Desde 2026-10-20 (Fase 0) é a **rota canônica** do catálogo: o item de menu é **"Guia do Sistema"** e `/historias-telas` redireciona para cá; o bloco "Contexto JCA / Actyon" migrou da tela antiga para esta, com estilos próprios `.fqs__contexto` |
| [`frontend/src/app/features/home/`](../../../frontend/src/app/features/home/) | Página `/` (Início — antiga `/planejamento`, rota removida): Planejamento Semanal com hero + busca global + grade de Acessos rápidos de 6 atalhos numa linha, seguido da **faixa de resumo do dia** (4 stats derivados dos signals: pendentes, `qtdAtrasadas`, urgentes e `eventosHoje`); seção **Minha semana** com navegação ‹ / Hoje / › (chip "esta semana" na semana corrente, Hoje desabilitado nela e destacado fora, `aria-labels`) e dois cards em `grid-topo` com altura máxima compartilhada (**Lista de tarefas**: prévia de até 6 itens via `tarefasVisiveis()` — sem bloqueadas, ordenadas por prioridade e prazo; a lista vem de `pendentesLista()` (só **pendências**: sem concluídas, canceladas ou arquivadas, e **sem as Urgentes**, que têm o card do painel "Não posso esquecer!"), e o **contador do subtítulo e o estado vazio usam a mesma lista**; link "Ver Todas >>" → `/implantacao/tarefas`; itens clicáveis → `/implantacao/tarefas/:id/editar`, **Agenda** de segunda a sexta da semana exibida com rolagem interna por coluna e eventos ampliados — título em até 2 linhas — eventos tingidos pela cor do evento, dia vazio "livre" — só eventos em que ele é responsável) e painel **"Não posso esquecer!"** (só prioridade Urgente **em aberto**, do próprio operador, itens clicáveis); carregamento com **skeleton** (shimmer) e falha com erro + retry; plurais via helper `plural()`; `focus-visible` e `prefers-reduced-motion`; escopo por `responsavelId` em tarefas e agenda (admin incluído, sem filtro de recorrência; sem usuário logado não há chamada HTTP); o **quadro semanal foi removido** (redundante com a Agenda); usa `TarefasService.listar`, `AgendaService.listarEventos` e o helper `ehAtrasada` de `tarefa.model.ts` |
| [`frontend/src/styles.scss`](../../../frontend/src/styles.scss) | Bootstrap parcial, tokens, temas claro/escuro e escala global |

O código não usa `backend/src/...`; esta documentação descreve somente o frontend do monorepo.


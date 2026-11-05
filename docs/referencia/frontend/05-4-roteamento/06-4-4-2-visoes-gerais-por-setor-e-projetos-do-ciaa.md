---
id: 05-4-roteamento.06
titulo: "4.4.2 Visões gerais por setor e Projetos do CIAA"
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
tokens_estimados: 565
ficha: docs/modulos/kanban.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

### 4.4.2 Visões gerais por setor e Projetos do CIAA

A visão geral de cada setor reaproveita o **mesmo** `DashboardComponent` de Implantação; o setor vem de `route.data.funcaoId` e o contexto dos links, de `route.data.contexto`. O componente envia `funcaoId` em `GET /implantacao/dashboard` e em `GET /implantacao/projetos`, e o backend aplica `PodeVerFuncao` (403 quando o setor não é a função do operador nem uma função que ele gerencia).

| URL | `data` da rota | Comportamento |
|---|---|---|
| `/suporte/visao-geral` | `{ funcaoId: 2, contexto: 'suporte' }` | KPIs e listas do setor Suporte; como o setor não tem aba Projetos, os atalhos de projeto levam ao Kanban |
| `/desenvolvimento/visao-geral` | `{ funcaoId: 3, contexto: 'desenvolvimento' }` | KPIs e listas do setor Desenvolvimento; atalhos de projeto levam ao Kanban |
| `/ciaa/visao-geral` | `{ funcaoId: 5, contexto: 'ciaa' }` | KPIs e listas do CIAA; atalhos de projeto levam a `/ciaa/projetos` |
| `/ciaa/projetos` | `{ funcaoId: 5, contexto: 'ciaa', basePath: '/ciaa/projetos' }` | `ProjetosComponent` reutilizado, filtrando `funcaoId=5` |
| `/ciaa/projetos/novo` | idem | `ProjetoFormComponent`; o `POST` envia `FuncaoId = 5` |
| `/ciaa/projetos/:id` | idem | `ProjetoDetalheComponent`; o detalhe lê `GET /implantacao/projetos/{id}` (403 se o projeto for de outro setor) |
| `/ciaa/projetos/:id/editar` | idem | `ProjetoFormComponent`; ao salvar/cancelar, volta para `basePath` |

As rotas `/suporte` e `/desenvolvimento` e `/ciaa` redirecionam `''` para `visao-geral` (`pathMatch: full`). A ordem `projetos/:id/editar` antes de `projetos/:id` vale também aqui. O `TarefasComponent` compartilhado lê `?funcaoId=` da URL e envia o filtro para `GET /implantacao/tarefas`; a sidebar usa esse parâmetro para abrir a lista já no setor correspondente (`2`, `1`, `3` ou `5`). No Kanban, as pills de função são limitadas à função do operador e às funções que ele gerencia.


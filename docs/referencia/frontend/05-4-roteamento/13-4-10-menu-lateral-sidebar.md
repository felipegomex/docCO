---
id: 05-4-roteamento.13
titulo: "4.10 Menu lateral (sidebar)"
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
tokens_estimados: 1625
ficha: docs/modulos/README.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### 4.10 Menu lateral (sidebar)

O menu é declarado em [`header-nav.config.ts`](../../../../frontend/src/app/shared/config/header-nav.config.ts) (`SIDEBAR_SECTIONS`) e consumido por [`sidebar.component.ts`](../../../../frontend/src/app/layout/sidebar/sidebar.component.ts). Desde **2026-10-22** a árvore é organizada por **setor**, com **Ferramentas** como seção única compartilhada e **Sistema** ao final.

| Seção | Itens-folha | Rotas |
|---|---:|---|
| **Início** | 3 | `/`, `/agenda`, `/chat` |
| **SUPORTE** | 13 (2 níveis) | `/suporte/visao-geral`, `/fraseologia`, `/modelo-chamados`, `/kanban?funcaoId=2`, `/implantacao/tarefas?funcaoId=2`; grupo **WhatsApp**: `/whatsapp`, **`/whatsapp/minhas-conversas`**, `/whatsapp/metricas`, `/whatsapp/respostas-rapidas`, `/whatsapp/numeros`, **`/whatsapp/admin`** (só `Administrador`, Fase 08) e `/whatsapp/retencao` (só `Administrador`) |
| **Implantação** | 5 | `/implantacao/dashboard`, `/implantacao/projetos`, `/kanban?funcaoId=1`, `/implantacao/tarefas?funcaoId=1`, `/ideias/lista` |
| **Desenvolvimento** | 3 | `/desenvolvimento/visao-geral`, `/kanban?funcaoId=3`, `/implantacao/tarefas?funcaoId=3` |
| **CIAA** | 5 | `/ciaa/visao-geral`, `/ciaa/projetos`, `/kanban?funcaoId=5`, `/implantacao/tarefas?funcaoId=5`, `/ideias/lista` |
| **ADM** | 5 | `/dashboard`, `/dashboard/comercial`, `/dashboard/financeiro`, `/dashboard/administrativo`, `/visao-adm` |
| **Ferramentas** | 7 (2 desabilitados) | `/database`, `/acessos/clientes`, **Mock API** (`disabled`, "Em breve"), **Mock Request** (`disabled`, "Em breve"), `/ferramentas`, `/base-conhecimento`, `/cursos` |
| **Sistema** | 4 | `/sistema/funcoes-administrativas`, `/faq-sistema`, `/auditoria`, `/ideias/lista` |
| **Total** | **42** | **32 URLs distintas** (40 itens com rota; 2 itens "em breve" sem link) |

Decisões confirmadas no código e no plano [`rbac-fechamento`](../../../planos/rbac-fechamento.md):

- **Cada setor tem a sua Visão Geral.** Implantação usa `/implantacao/dashboard`; Suporte, Desenvolvimento e CIAA reutilizam o mesmo componente com `data.funcaoId` `2`, `3` e `5` (§4.4.2). O dashboard ADM continua em `/dashboard`.
- **Kanban e Tarefas são abas do setor**, não itens de Início. Os links carregam `?funcaoId=` (`1` Implantação, `2` Suporte, `3` Desenvolvimento, `5` CIAA), e o Kanban limita as pills à função do operador e às funções que ele gerencia.
- **ADM perdeu Kanban, Tarefas e Onboarding da sidebar.** As rotas continuam existindo (`/kanban?funcaoId=4`, `/implantacao/tarefas?funcaoId=4`, `/empresa/onboarding`), mas não há item de menu.
- **Ideias e Melhorias é item de primeiro nível** na Implantação, no CIAA e no Sistema; os três apontam para o mesmo `/ideias/lista` e o guard aceita qualquer um dos recursos (`aba.implantacao.ideias`, `aba.ciaa.ideias`, `aba.sistema.ideias`).
- **Ferramentas é uma única seção compartilhada** — os mesmos itens para todos. **Cursos** saiu de Conhecimento e entrou aqui; **Mock API** e **Mock Request** estão previstos com `disabled: true` e `dica: 'Em breve'`, sem rota.
- **A seção Conhecimento foi removida.** Stack, Glossário e Conhecimento Rápido foram consolidados em `/base-conhecimento` (§4.2); `Guia do Sistema` foi para **Sistema**.
- **WhatsApp continua como grupo** com as telas existentes. A implementação mantém **Retenção de mídia** visível só para `Administrador` (`roles: ['Administrador']`) e sem recurso RBAC próprio; SLA e Configuração existem como rota, mas não aparecem na sidebar. **Minhas Conversas** entrou como segundo item do grupo (rota `/whatsapp/minhas-conversas`, recurso `aba.suporte.whatsapp`). **Admin** entrou na Fase 08 (`/whatsapp/admin`, `roles: ['Administrador']`), como porta de entrada do hub; a tela de contatos fica um nível abaixo (`/whatsapp/admin/contatos`), sem item próprio.
- **Filtro por `recurso`.** O item declara `recurso: 'aba.*'` e a sidebar aplica `PermissoesService.pode(recurso, Ler)`; `roles` permanece como compatibilidade. Um grupo sem filhos visíveis não é renderizado.
- **Estado dos grupos** é um `signal` persistido em `localStorage` (chave `cc.nav.grupos`); se o armazenamento estiver indisponível, o estado segue em memória.

Rótulos e seções alterados em 2026-10-22:

- **"Painel da empresa"** virou **"Visão Geral"** em ADM; **"Indicadores de implantação"** virou **"Visão Geral"** em Implantação; **"Chat com o JOTA"** virou **"Agentes"** em Início.
- **"Glossário"**, **"Stack"** e as quatro trilhas deixaram de ser itens de menu: agora são abas de **Base de Conhecimento** em Ferramentas.
- **"Ideias, Melhorias e Erros"** (Sistema) e **"Ideias e Melhorias"** (CIAA/Implantação) apontam para o mesmo `/ideias/lista`.
- **"Kanban Geral"** e **"Ideias"** saíram de Início; o Kanban passou a ser item de cada setor e Ideias passou aos setores e a Sistema.
- **"Kanban ADM"** já havia saído em 2026-10-20 (duplicava `/kanban`); o recorte `?perfil=` continua aceito como deep link.

A interface `NavItem` perdeu os campos `exact` e `grupo`: eram declarados e **nunca lidos** — o template fixa `routerLinkActiveOptions` em `{ exact: true }` nos itens de topo e `{ exact: false }` nos itens de grupo, e o subtítulo de cada seção vem de uma tabela fixa por título (`getSubtitle`). `queryParams`, `recurso`, `dica`, `badge` e `disabled` permanecem em uso (template).

Comportamento do componente (corrigido na Fase 0):

| Item | Regra confirmada no código |
|---|---|
| Filtro por papel | `visivelPara()` aplica `roles` **tanto nos itens de topo quanto nos itens dentro de grupo**. Antes o filtro cobria só o primeiro nível, e um item de grupo com `roles` apareceria para todo mundo. Um grupo que fique sem itens visíveis **deixa de aparecer** (`.filter(g => g.itens.length > 0)`) |
| Estado dos grupos | `gruposExpandidos` é um `signal<ReadonlySet<string>>` **persistido em `localStorage`** (chave `cc.nav.grupos`) e lido na inicialização. Antes o estado vivia só em memória e **começava fechado**, o que escondia os itens da seção *SUPORTE* em toda sessão. Se o `localStorage` estiver indisponível (modo privado, cota), o estado segue em memória sem quebrar a tela |


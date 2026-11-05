---
id: visao-geral.04
titulo: "3. Módulos e rotas ativas"
tipo: arquitetura
area: produto
sistema: central
tags: [modulos, rotas, ambientes, roadmap]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app/features.routes.ts
  - backend/Central_BackEnd/Program.cs
testes:
  - docs/
tokens_estimados: 2781
ficha: docs/modulos/README.md
---
> Parte de [01 — Visão geral](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 3. Módulos e rotas ativas

As rotas abaixo foram conferidas em `app.routes.ts`, `features.routes.ts`, `acessos.routes.ts`, `ideias.routes.ts`, `auditoria.routes.ts`, `dashboard.routes.ts`, `comercial.routes.ts`, `implantacao.routes.ts`, `whatsapp.routes.ts`, `sistema.routes.ts`, `database.routes.ts` e `admin.routes.ts`.

| Área | Rotas ativas | Acesso |
|---|---|---|
| Login | `/login` | Público no frontend |
| Início | `/` (Minha semana — tarefas e agenda do operador logado) | Autenticado |
| Agenda | `/agenda` | Autenticado |
| Dashboard | `/dashboard`, `/dashboard/comercial` (Kanban de Leads), `/dashboard/comercial/propostas`, `/dashboard/administrativo` (base de clientes), `/dashboard/financeiro` | Autenticado; `/dashboard/financeiro` exige `Administrador` (`adminGuard` + role no backend); `/dashboard/comercial/*` autenticado; histórico de contatos em `/dashboard/comercial/leads/{id}/contatos` |
| Conhecimento estático | `/fraseologia`, `/modelo-chamados`, `/politica`, `/visao-adm`, `/visao-adm/detalhe/:id`; `/stack`, `/ferramentas/faq` e `/trilhas/*` **redirecionam** para `/base-conhecimento` | Autenticado |
| Chat (JOTA, com agentes por setor) | `/chat` e widget JOTA global | Autenticado; pergunta (e agente, na página) encaminhadas ao backend |
| Revisão de Conhecimento | `/conhecimento`, `/conhecimento/nova`, `/conhecimento/configuracao`, `/conhecimento/editor`, `/conhecimento/diff`, `/conhecimento/historico` | Autenticado; o módulo consulta a permissão do revisor no backend; configurações administrativas são revalidadas pela API |
| Cursos | `/cursos`, `/cursos/detalhe/:id` | Autenticado; item da seção **Ferramentas** |
| Ferramentas | `/ferramentas`, `/ferramentas/detalhe/:id`, `/ferramentas/workspace?painel=:id&comparar=:id` (área de iframes integrados: único, tela cheia interna e comparação lado a lado), `/base-conhecimento` (abas Stack, Glossário e Conhecimento Rápido) | Autenticado; `/base-conhecimento` usa `permissaoGuard('aba.ferramentas.base-conhecimento')`; Mock API e Mock Request aparecem "em breve", sem rota |
| Acessos das Empresas | `/acessos` redireciona para `/acessos/clientes`; também existem `/acessos/clientes/novo` (assistente de 3 etapas) e `/acessos/clientes/:id` (detalhe da unidade), além do redirect de compatibilidade `/ferramentas/acessos` → `/acessos/clientes` | Autenticado; o detalhe exige gate de senha do operador |
| Empresa | `/empresa` redireciona para `/empresa/onboarding`; `/empresa/onboarding/capitulo/:id` | Autenticado; o onboarding **saiu da sidebar** em 2026-10-22 (a rota continua) |
| Ideias, Melhorias e Erros | `/ideias` redireciona para `/ideias/lista`; `/ideias/nova`, `/ideias/nova/:tipo` e `/ideias/:id` (detalhe). Gestão de gestores em `/gestores-ideias` (tela **GESTORES**) | Autenticado; o `permissaoGuard` aceita **any-of** `aba.sistema.ideias`, `aba.ciaa.ideias` ou `aba.implantacao.ideias`. **Escopo derivado de `tbgestor` no backend**: o criador vê só o que criou, o gestor vê a sua função, e quem não tem acesso recebe **404** (não 403) para não revelar existência. A tela GESTORES é alcançada pelo dropdown do header (**Perfil → Administração → Gestores**), visível só para `Administrador` no front; a rota fica sob `authGuard` e o backend revalida por claim — não-admin que digita a URL recebe **403** e vê o aviso na tela. **Em construção** — ainda não publicado |
| Kanban Geral | `/kanban` (quadro por função, `data.modo = 'geral'` — **única rota que carrega o quadro**), `/kanban/gargalos`, `/kanban/configurar` (visão do setor sobre o catálogo de colunas) e `/kanban/admin/colunas`; aceita `?funcaoId=`, `?tipo=` e `?perfil=`. O quadro usa o **catálogo de colunas compartilhado** (o mesmo conjunto para todos os setores) na **visão do setor** (`tbcolunafuncao`: o que o setor oculta e a máscara de nome) | Autenticado; a leitura valida `PodeVerFuncao` quando o operador é informado e as pills são limitadas à própria função + funções geridas; a escrita depende da função do operador, a **visão do setor** exige gestor da função ou Master e a administração do quadro exige `Administrador` |
| Suporte, Desenvolvimento e CIAA | `/suporte/visao-geral`, `/desenvolvimento/visao-geral`, `/ciaa/visao-geral` (reutilizam o dashboard com `data.funcaoId` 2/3/5); `/ciaa/projetos`, `/ciaa/projetos/novo`, `/ciaa/projetos/:id`, `/ciaa/projetos/:id/editar` (reutilizam os componentes de Projetos com `data.funcaoId = 5` e `data.basePath = '/ciaa/projetos'`) | Autenticado + `permissaoGuard` por recurso (`aba.suporte.visao-geral`, `aba.desenvolvimento.visao-geral`, `aba.ciaa.visao-geral`, `aba.ciaa.projetos`); o backend aplica `PodeVerFuncao`/`PodeLerAsync` e responde **403** para outro setor |
| Implantação | `/implantacao` redireciona para `/implantacao/dashboard`; `/implantacao/kanban` redireciona para `/kanban`; também existem `/implantacao/projetos`, `/implantacao/projetos/novo`, `/implantacao/projetos/:id`, `/implantacao/projetos/:id/editar`, `/implantacao/tarefas` (aceita `?funcaoId=`), `/implantacao/tarefas/novo` e `/implantacao/tarefas/:id/editar` | Autenticado; dashboard e projetos filtram por `funcaoId` (padrão 1); a seção inclui Tarefas com o setor no query |
| Banco de Dados | `/database` redireciona para `/database/visao-geral`; também existem `/database/explorador`, `/database/relacionamentos`, `/database/consultas`, `/database/diferencas`, `/database/configuracao` e `/database/tabela/:schema/:tabela` | Autenticado |
| Administração | `/admin` redireciona para `/admin/cadastros/tipos-projeto`; também existem `/admin/cadastros/tipos-projeto/novo` e `/admin/cadastros/tipos-projeto/editar/:id` | Autenticado e `Administrador` no frontend |
| Auditoria | `/auditoria` (duas abas somente leitura: revelações de credencial e eventos da implantação, com filtros e paginação) | Autenticado; a leitura é restrita no backend a **administrador ou gestor de função** — sem papel o backend devolve **403** e a tela avisa. Entrada pelo botão **Auditoria** no menu do usuário do header (visível a todo autenticado) |
| **Sistema (RBAC)** | `/sistema` redireciona para `/sistema/funcoes-administrativas` (cards com atalhos para Operadores, Gestores, Cadastros, Colunas do Kanban e Auditoria); `/sistema/operadores` (cadastro de operadores com cargo e setor); `/sistema/permissoes` (tela de controle da matriz) | Autenticado + **permissão por recurso**: `permissaoGuard('aba.sistema.funcoes-administrativas')`, `aba.sistema.operadores` e `aba.sistema.permissoes` no frontend; o backend revalida com policies `perm:<recurso>:<acao>`. As escritas de operador exigem `perm:aba.sistema.operadores:Criar`/`Editar`; `GET /permissoes/recursos` é exclusivo do `GESTOR_MASTER` |
| **Atendimento WhatsApp** | `/whatsapp` (Kanban da fila de atendimento — desde a Fase 07 com **card do Service Desk** e filtros server-side de prioridade/empresa/etiqueta/investigação/SLA; só a ordem é local e no mobile o quadro rola na horizontal), **`/whatsapp/minhas-conversas`** (lista dos atendimentos do operador com conversa ao lado e filtro de status **server-side**, padrão **Abertas** — status 0–2), `/whatsapp/atendimento/:id` (painel **Service Desk**: cabeçalho fixo com empresa/grupo, contato, status, SLA e ações + **3 colunas recolhíveis** — contexto do cliente (identificação, observação e histórico), conversa e operação, com **etiquetas**, **marca de investigação** e **observação interna** na aba Resumo; **mídia sob demanda**), `/whatsapp/metricas` (indicadores com abas **Geral**, **Qualidade**, **Clientes**), `/whatsapp/respostas-rapidas` (textos prontos), **`/whatsapp/auditoria`** (log genérico filtrável, admin), **`/whatsapp/ia-configuracao`** (configuração IA Gemini, admin), `/whatsapp/numeros` (administração dos números conectados e do QR), `/whatsapp/configuracao`, `/whatsapp/sla` (metas, **percentual de aviso** e horário útil, admin), `/whatsapp/retencao` (admin), **`/whatsapp/admin`** (hub administrativo com indicadores, atalhos e configuração avançada — Fase 08), **`/whatsapp/admin/contatos`** (filtros, tabela paginada, edição e histórico do contato) e **`/whatsapp/etiquetas`** (catálogo operacional). **Fase 09:** Kanban, Minhas Conversas e atendimento recebem eventos do hub `/hubs/whatsapp` (atualização, mensagem, ack, evento, fila e digitação) e recarregam sem esperar o polling de 30 s, que segue como fallback | Autenticado; a rota-mãe usa `permissaoGuard('aba.suporte.whatsapp')`. A fila, **Minhas Conversas**, o painel e as métricas são de **qualquer atendente**; `/whatsapp/numeros`, `/whatsapp/auditoria`, `/whatsapp/ia-configuracao`, `/whatsapp/configuracao`, `/whatsapp/sla`, `/whatsapp/retencao`, `/whatsapp/admin` e `/whatsapp/admin/contatos` são **só administrador**; o catálogo `/whatsapp/etiquetas` é **legível por qualquer atendente**, com escrita **só administrador** (itens de menu com `roles: ['Administrador']` e backend responde **403**). O módulo nasce **desabilitado** no `appsettings.json` versionado (`WhatsApp:Habilitado = false`) e é **ligado por variável de ambiente** no servidor (`WhatsApp__Habilitado=true`); o gateway OpenWA v0.25.0 está **no ar** na VM Ubuntu `192.168.2.253` e o **piloto está em operação na homologação** (`.130`), com número pareado e mensagens reais no Kanban — a **produção (`.254`) foi publicada em 2026-10-26** |
| Chat | `/chat` (iframe do AnythingLLM) | Autenticado; página `/chat` (`ChatIframeComponent`) carrega o AnythingLLM com SSO via `GET /api/chat/sso`; sem API REST de agentes e sem widget global |
| Guia do Sistema | `/faq-sistema` (canônico); `/historias-telas` redireciona para ele | Autenticado + `permissaoGuard('aba.sistema.guia')` |

Redirecionamentos de compatibilidade ainda presentes:

- `/ferramentas/acessos` → `/acessos/clientes` (endereço da tela antiga, que lia do Google Sheets);
- `/ferramentas/contra-senha` → `/ferramentas`;
- `/ferramentas/modelos` → `/modelo-chamados`;
- `/ferramentas/faq` → `/base-conhecimento?aba=glossario`;
- `/stack` → `/base-conhecimento?aba=stack`;
- `/trilhas/resolver` → `/base-conhecimento?aba=conhecimento&trilha=resolver`; `/trilhas/sql`, `/trilhas/rede` e `/trilhas/infra` seguem o mesmo padrão;
- `/suporte` → `/suporte/visao-geral`; `/desenvolvimento` → `/desenvolvimento/visao-geral`; `/ciaa` → `/ciaa/visao-geral`;
- `/whatsapp-flow` → `/fraseologia`;
- `/implantacao/relatorio` → `/implantacao/dashboard`;
- `/implantacao/agenda` → `/agenda`;
- `/implantacao/kanban` → `/kanban` (o quadro de tarefa e projeto passou a ser o Kanban Geral; o caminho antigo continua como atalho para links antigos);
- `/historias-telas` → `/faq-sistema` (o catálogo de telas ficou canônico em `/faq-sistema`; o caminho antigo continua como atalho);
- `/database/query-builder` → `/database/consultas`;
- rota não encontrada → `/`.

Observações de navegação:


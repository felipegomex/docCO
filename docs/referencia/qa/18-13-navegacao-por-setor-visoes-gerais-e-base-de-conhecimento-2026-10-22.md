---
id: sec.18-13-navegacao-por-setor-visoes-gerais-e-base-de-conhecimento-2026-10-22
titulo: "13. Navegação por setor, visões gerais e Base de Conhecimento (2026-10-22)"
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
tokens_estimados: 1513
ficha: docs/modulos/chat-conhecimento.md
---
> Ficha: [Chat e Conhecimento](../../modulos/chat-conhecimento.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 13. Navegação por setor, visões gerais e Base de Conhecimento (2026-10-22)

### NAV-01 — Sidebar por setor com Ferramentas compartilhadas

- **Objetivo:** Cada operador vê apenas as seções do seu setor, as Ferramentas iguais para todos e a seção Sistema filtrada por recurso.
- **Pré-condições:** operador autenticado; `GET /api/v1/permissoes/minhas` disponível; recursos da reorganização aplicados no banco.
- **Passos:** 1. Entrar com um operador de Suporte. 2. Conferir as seções **Início**, **SUPORTE**, **Ferramentas** e **Sistema**. 3. Repetir com operadores de Implantação, Desenvolvimento, CIAA e ADM. 4. Abrir **Ferramentas** e conferir os itens iguais.
- **Resultado esperado:** a sidebar renderiza **8 seções** e **42 itens-folha** (40 com rota; Mock API e Mock Request desabilitados com "Em breve"). Seções sem item visível não aparecem; itens com `recurso` usam `PermissoesService.pode(recurso, Ler)`. **ADM** não mostra Kanban, Tarefas nem Onboarding. **Ideias e Melhorias** aparece na Implantação, no CIAA e no Sistema, todas apontando para `/ideias/lista`. **Cursos** fica em Ferramentas; a seção **Conhecimento** não existe mais.
- **Estado atual/bloqueio:** o código ainda mantém **Retenção de mídia** como quinto filho do grupo WhatsApp, visível só para `Administrador`; o plano fechado lista quatro itens. Divergência registrada para alinhamento posterior.
- **Rotas/endpoints/tabelas:** `header-nav.config.ts`, `sidebar.component.ts`; `GET /api/v1/permissoes/minhas`; `tbpermissao_recurso`, `tbperm_cargo`, `tbperm_operador`.
- **Status/automação:** **cenário documentado**; não há teste de frontend da sidebar.

### NAV-02 — Visão Geral por setor

- **Objetivo:** Abrir o dashboard já recortado pelo setor do operador.
- **Pré-condições:** operador com função 2 (Suporte), 3 (Desenvolvimento) ou 5 (CIAA); permissão `aba.<setor>.visao-geral`.
- **Passos:** 1. Abrir `/suporte/visao-geral`, `/desenvolvimento/visao-geral` ou `/ciaa/visao-geral`. 2. Conferir KPIs, tarefas e próximos prazos. 3. Tentar abrir a visão de outro setor por URL direta.
- **Resultado esperado:** o componente envia `GET /api/v1/implantacao/dashboard?funcaoId=2|3|5`; projetos e tarefas são filtrados pelo setor; atalhos de projeto levam ao Kanban nos setores sem aba Projetos e a `/ciaa/projetos` no CIAA. A URL de outro setor devolve **403** e a tela mostra estado de sem acesso.
- **Rotas/endpoints/tabelas:** `DashboardComponent` com `data.funcaoId`; `GET /api/v1/implantacao/dashboard`; `ProjetosService`/`ProjetoService`; `tbprojeto`, `tbtarefa`.
- **Status/automação:** **cenário documentado**; `dotnet test` cobre a matriz de função, não a tela.

### NAV-03 — Projetos do CIAA

- **Objetivo:** Listar, criar, editar e abrir projetos do CIAA no contexto da função 5.
- **Pré-condições:** permissão `aba.ciaa.projetos`; operador do CIAA ou gestor/Master.
- **Passos:** 1. Abrir `/ciaa/projetos` e conferir que a lista só traz projetos do CIAA. 2. Criar um projeto em `/ciaa/projetos/novo`. 3. Abrir o detalhe e editar. 4. Tentar abrir por URL um projeto de outro setor.
- **Resultado esperado:** a lista envia `GET /api/v1/implantacao/projetos?funcaoId=5`; o `POST` envia `FuncaoId = 5` e o backend valida `PodeCriarFuncao`; o projeto nasce com `PRJ_FuncaoId = 5`; `Obter`, `ObterEtapas` e `ObterEtapaDetalhe` aplicam `PodeLerAsync` e devolvem **403** para projeto de outro setor. Formulário e detalhe voltam para `basePath = /ciaa/projetos`.
- **Rotas/endpoints/tabelas:** rotas `/ciaa/projetos*`; `GET/POST /api/v1/implantacao/projetos`, `GET /{id}`, `GET /{id}/etapas`, `GET /{id}/etapas/{ordem}`; `tbprojeto`.
- **Status/automação:** **cenário documentado**.

### NAV-04 — Base de Conhecimento única

- **Objetivo:** Consultar Stack, Glossário e Conhecimento Rápido em uma única tela, sem conteúdo duplicado.
- **Pré-condições:** permissão `aba.ferramentas.base-conhecimento`.
- **Passos:** 1. Abrir `/base-conhecimento`. 2. Alternar as abas **Stack**, **Glossário** e **Conhecimento Rápido**. 3. Dentro de Conhecimento Rápido, alternar Resolver, SQL, Rede e Infra. 4. Abrir `/stack`, `/ferramentas/faq` e `/trilhas/resolver`.
- **Resultado esperado:** a tela abre na aba `stack` por padrão; `?aba=glossario` e `?aba=conhecimento&trilha=sql` selecionam a aba/sub-aba; os caminhos antigos redirecionam para `/base-conhecimento` com o parâmetro correspondente. O item **Base de Conhecimento** aparece em Ferramentas e o recurso aponta `PER_Rota = '/base-conhecimento'`.
- **Rotas/endpoints/tabelas:** `/base-conhecimento`, `/stack`, `/ferramentas/faq`, `/trilhas/*`; `tbpermissao_recurso`.
- **Status/automação:** **cenário documentado**.

### NAV-05 — Ideias e Melhorias reutilizado por três seções

- **Objetivo:** Abrir a mesma listagem de Ideias a partir do Sistema, do CIAA e da Implantação sem duplicar componente.
- **Pré-condições:** operador com pelo menos um de `aba.sistema.ideias`, `aba.ciaa.ideias` ou `aba.implantacao.ideias`.
- **Passos:** 1. Abrir **Ideias e Melhorias** pela sidebar do CIAA. 2. Repetir pela Implantação e por **Sistema → Ideias, Melhorias e Erros**. 3. Entrar com um operador sem nenhum dos três recursos.
- **Resultado esperado:** as três entradas levam a `/ideias/lista` e renderizam a mesma tela; o `permissaoGuard` aceita **any-of** (qualquer um dos três recursos). Sem nenhum deles, a navegação é barrada; o backend continua devolvendo **404** para solicitação fora do escopo, e a tela traduz em "Solicitação não encontrada".
- **Rotas/endpoints/tabelas:** `/ideias/lista`; `features.routes.ts`; `GET /api/v1/ideias`; `tbideia`, `tbgestor`.
- **Status/automação:** **cenário documentado** (módulo ainda não publicado).


---
id: visao-geral.05
titulo: "3. Módulos e rotas ativas — parte 2"
tipo: arquitetura
area: produto
sistema: central
tags: [modulos, rotas, ambientes, roadmap]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - frontend/src/app/features.routes.ts
  - backend/Central_BackEnd/Program.cs
testes:
  - docs/
tokens_estimados: 930
ficha: docs/modulos/README.md
---
> Parte de [01 — Visão geral](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

### 3. Módulos e rotas ativas — parte 2

- A sidebar segue a árvore por setor: **Início** (Home, Agenda, Agentes), **SUPORTE** (Visão Geral, Fraseologia, Modelo de Chamados, Kanban, Tarefas e grupo WhatsApp com Atendimento, Minhas Conversas, Contatos (painel), Indicadores, **Admin** (só Administrador), Auditoria, Configuração IA, Números e Retenção de mídia), **Implantação** (Visão Geral, Projetos, Kanban, Tarefas, Ideias e Melhorias), **Desenvolvimento** (Visão Geral, Kanban, Tarefas), **CIAA** (Visão Geral, Projetos, Kanban, Tarefas, Ideias e Melhorias), **ADM** (Visão Geral, Comercial, Financeiro, Administrativo, Procedimentos), **Ferramentas** (Banco de Dados, Acesso das Empresas, Mock API e Mock Request "em breve", Central de Utilidades, Base de Conhecimento, Cursos) e **Sistema** (Funções Administrativas, Guia do Sistema, Auditoria de Logs, Ideias/Melhorias/Erros). São **8 seções, 44 itens-folha e 36 URLs distintas** (42 itens com rota; 2 "em breve" sem link). O detalhe por item está em [`referencia/frontend.md`](../../referencia/frontend.md) §4.10.
- **Kanban e Tarefas pertencem ao setor**: cada link carrega `?funcaoId=` (`1` Implantação, `2` Suporte, `3` Desenvolvimento, `5` CIAA) e o Kanban limita as pills à própria função e às funções geridas. **ADM perdeu Kanban, Tarefas e Onboarding da sidebar**, mas as rotas continuam existindo.
- **Ideias e Melhorias** é item de primeiro nível na Implantação, no CIAA e no Sistema; os três apontam para `/ideias/lista` e o guard aceita qualquer um dos três recursos.
- **Ferramentas é uma seção única e compartilhada**; **Cursos** passou para lá e **Mock API/Mock Request** estão previstos sem rota. A antiga seção **Conhecimento** deixou de existir: Stack, Glossário e Conhecimento Rápido viraram abas de `/base-conhecimento`, e o **Guia do Sistema** foi para **Sistema**.
- A seção fantasma **Administração** (grupo `administracao`, sem filhos) foi **removida da sidebar**. A configuração de gestores é alcançada pelo dropdown do usuário no header: **Perfil → Administração → Gestores** (rota `/gestores-ideias`). A seção **ADM** permanece na sidebar — são coisas distintas.
- Os itens **Empresa** (`/empresa`, que só redirecionava ao onboarding) e **Políticas** saíram da sidebar — `/politica` continua acessível por URL.
- O módulo Comercial tem **duas abas**: *Kanban de Leads* (padrão) e *Propostas*. A aba *Oportunidades* e a rota `/dashboard/comercial/oportunidades` foram **removidas**; qualquer subpath desconhecido em `/dashboard/comercial` redireciona para o Kanban. A oportunidade continua existindo e é criada a partir do lead, dentro do modal dele.
- `/conhecimento` é o módulo de revisão documental Gitea, separado de `/base-conhecimento`; sua guarda consulta `GET /api/conhecimento/permissao`. As rotas `/admin/kanban`, `/admin/tarefas` e `/admin/projetos` saíram por reaproveitarem componentes já existentes.
- Chat e Conhecimento são APIs sem versão: o chat é iframe do AnythingLLM autenticado por `GET /api/chat/sso`; o Conhecimento está em `/api/conhecimento/*`. O contrato antigo `/api/rag-proxy` (assim como `/api/chat/agentes`, `/api/chat/perguntar` e `/api/chat/provedor`) foi removido no `cb36ad3` (2026-11-03).
- Não há rota frontend `/admin/dashboard` em `admin.routes.ts`, embora exista endpoint administrativo no backend.
- `scripts/screens-data.json` ainda pode listar rotas antigas e não inventaria as telas do Dashboard Integrado nem as do módulo Acessos Empresas; ele não altera essa lista.


---
id: visao-geral.06
titulo: "4. Stack real"
tipo: arquitetura
area: produto
sistema: central
tags: [modulos, rotas, ambientes, roadmap]
status: ativo
revisado_em: "2026-11-05"
fonte:
  - frontend/src/app/features.routes.ts
  - backend/Central_BackEnd/Program.cs
testes:
  - docs/
tokens_estimados: 740
ficha: docs/modulos/README.md
---
> Parte de [01 — Visão geral](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 4. Stack real

### Frontend

- Angular 18.2 em uma única aplicação, com componentes standalone.
- TypeScript 5.5, RxJS 7.8 e Angular Router.
- Build de produção gera apenas o artefato de browser (`browser/`), publicado no IIS; aplicação somente CSR.
- Bootstrap 5.3 importado parcialmente, Bootstrap Icons 1.13 e ng-bootstrap 17.
- Angular CDK 18 para interações de Kanban.
- Chart.js 4.4 para visualizações.

### Backend

- ASP.NET Core 8 em `backend/Central_BackEnd/Central_BackEnd.csproj`.
- C# com nullable e implicit usings habilitados.
- Controllers MVC, injeção de dependência e serviços separados por domínio.
- Entity Framework Core 8 com provider SQL Server ou InMemory conforme ambiente e flag.
- API Versioning com versão pública `v1`.
- JWT Bearer e Swagger/Swashbuckle.
- Limitação de taxa e proteção de tentativas de login.
- Cliente HTTP tipado (`IHttpClientFactory`) para o gateway OpenWA, fila durável de webhooks no SQL Server e worker de background — base do módulo de atendimento WhatsApp.
- **SignalR nativo do ASP.NET Core 8** no hub `/hubs/whatsapp`: **notificação de saída** do módulo WhatsApp (Fase 09). O hub autentica o JWT por `access_token` na query **apenas** nesse path, publica eventos **depois do commit** e nunca carrega texto/observação/nota — o cliente relê pela API. A fonte da verdade continua sendo a API versionada; o polling de 30 s permanece como fallback.

### Dados e integrações

- SQL Server para dados operacionais, implantação, agenda, autenticação, auditoria, o módulo Acessos Empresas e o **atendimento WhatsApp** (**22 tabelas** `tbwhatsapp_*`; a Fase 09 não criou schema e a 22ª tabela da Fase 8, `tbwhatsapp_mensagem_reacao`, nasceu do script `WhatsApp_Fase9_Idempotente.sql`, **aplicado em 2026-10-30** em `dbCENTRAL_OPERACAO` e `dbBUSINESS_HML` (`192.168.2.154`)).
- SQL Server separado, configurado por seção ou variáveis de ambiente, para o Database Explorer.
- **OpenWA** (gateway do WhatsApp): serviço **externo ao IIS**, que roda na **VM Ubuntu `192.168.2.253`** (hospedada no Hyper-V do servidor de produção) e é alcançado **pela rede interna** (`192.168.2.253:2785`), com a porta liberada **apenas** para os dois servidores da Central (`192.168.2.130` e `192.168.2.254`). O navegador nunca fala com ele.
- Chat: AnythingLLM responde dentro do iframe do AnythingLLM autenticado pelo SSO da Central (`GET /api/chat/sso`); a Central não formula e não grava pergunta/resposta.
- Revisão: Gitea armazena documentos, branches e pull requests; a VM executa sincronização de Markdown aprovado para workspaces AnythingLLM.

### Convenções visuais confirmadas

- Escala global de 80% por `html.scaled` em todas as telas, exceto `/login`.
- Unidades estruturais em `rem`, com bordas e linhas finas em `px`.
- A lista de projetos usa layout e cards compactos; o detalhe está em [`06-COMPONENTES-FRONTEND.md`](../../referencia/frontend.md).


---
id: componentes.04
titulo: "3. Frontend Angular"
tipo: arquitetura
area: arquitetura
sistema: central
tags: [arquitetura, componentes, jwt, cors]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - docs/
tokens_estimados: 686
ficha: docs/modulos/README.md
---
> Parte de [02 — Arquitetura vigente](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 3. Frontend Angular

### Bootstrap e renderização

- `src/main.ts` usa `bootstrapApplication` para iniciar o `AppComponent` standalone.
- `app.config.ts` registra Router, hydration, animações, `HttpClient` com `fetch` e o interceptor de autenticação.
- `angular.json` define o builder `application` apenas para o browser.

O build de produção gera `browser/`, que é o único artefato publicado no IIS. A aplicação é somente CSR.

### Rotas e composição

- `app.routes.ts` cria os shells público, principal e administrativo.
- `features.routes.ts` contém as rotas do shell principal.
- `implantacao.routes.ts`, `database.routes.ts` e `dashboard.routes.ts` (com `comercial.routes.ts` interno) usam `loadChildren`.
- A maioria das rotas dos módulos usa `loadComponent`; `admin.routes.ts` ainda importa diretamente os componentes de Kanban, tarefas e projetos de Implantação.
- `/login` é público; o shell principal usa `authGuard`; `/admin` usa `authGuard` e `adminGuard`.
- A rota curinga redireciona para `/`.
- `authGuard` e `adminGuard` são principalmente proteções de navegação no cliente. A proteção real depende do backend.
- O JOTA está disponível como página `/chat` e como widget global no `MainLayout`; ambos usam o endpoint de pergunta do Chat de Conhecimento.
- `/conhecimento` carrega o shell/lista e as rotas filhas de revisão; seu guard Angular consulta a permissão do backend, que continua sendo a fronteira efetiva.

### Sessão no cliente

- `TokenStorageService` mantém access token e usuário somente em memória.
- `authInterceptor` envia cookies, anexa `Authorization: Bearer` e executa refresh single-flight após `401`.
- O frontend tenta restaurar a sessão no startup por `/auth/refresh`.
- Enquanto há sessão ativa, o frontend verifica o access token a cada cinco minutos e rotaciona o refresh token.
- Logout local e no servidor limpam o estado em memória; o cookie é apagado pelo backend.

### Regras de apresentação

- `html.scaled` reduz a raiz tipográfica para 80% em todas as telas, exceto `/login`.
- A lista de projetos usa cards e espaçamento compactos.
- Componentes devem continuar standalone, lazy e compatíveis com o browser.
- O mascote do chat (`app-jota-3d`, usado em `/chat` e no widget do `MainLayout`) é **presentacional**: a tela alimenta o `estado` (pensando/erro/sucesso/idle) e o componente só o reflete, com fallback 2D; nenhum estado de animação decide resultado de operação.


---
id: sec.03-2-runtime-e-organizacao
titulo: "2. Runtime e organização"
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
tokens_estimados: 675
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/frontend.md](../frontend.md).

# 2. Runtime e organização

### 2.1 Stack e bootstrap

| Item | Estado no código |
|---|---|
| Framework | Angular 18 com componentes standalone e router |
| Renderização | Browser (CSR; SSR desativado) |
| Reactive state | RxJS, `BehaviorSubject` e signals Angular |
| HTTP | `HttpClient` com `withFetch()` e interceptor funcional |
| Tempo real | `@microsoft/signalr` — cliente do hub `/hubs/whatsapp` (Fase 09), com reconexão automática e fallback para o polling de 30 s |
| UI base | Import parcial de Bootstrap SCSS |
| Componentes interativos | `NgbModule` e modais/serviços do ng-bootstrap |
| Tipografia | Fontes carregadas no [`frontend/src/index.html`](../../../frontend/src/index.html) |
| Ícones | Bootstrap Icons carregado por CDN no `index.html` |
| Charts | Chart.js 4 é dependência declarada; o uso fica nos componentes de gráficos |

A aplicação roda somente no browser, com `bootstrapApplication` em [`main.ts`](../../../frontend/src/main.ts). Não há `app.config.server.ts` nem `provideServerRendering()` no código atual.

### 2.2 Árvore lógica de `frontend/src/app`

| Área | Responsabilidade | Exemplos |
|---|---|---|
| raiz | Bootstrap, outlet, rotas e configuração | `app.component.ts`, `app.routes.ts`, `app.config.ts` |
| `core` | Estado singleton, autenticação, guards, interceptor e modelos compartilhados | `core/services`, `core/guards`, `core/interceptors` |
| `layout` | Shell da aplicação e componentes persistentes | `main-layout`, `header`, `sidebar`, `breadcrumb`, `footer` |
| `shared` | Componentes, diretiva, configuração, utilitários e metadados reutilizáveis | `PageHeaderComponent`, `GlobalSearchComponent`, `Jota3dComponent` (`app-jota-3d`, em `shared/components/jota-3d/`), `header-nav.config.ts` |
| `features` | Módulos funcionais e sub-rotas | `home`, `agenda`, `implantacao`, `database`, `admin` |
| `features/services` | Serviços de acervos locais consumidos por features | `cursos.service.ts`, `ferramentas.service.ts` |

### 2.3 Aliases TypeScript

| Alias | Destino |
|---|---|
| `@core/*` | `src/app/core/*` |
| `@layout/*` | `src/app/layout/*` |
| `@shared/*` | `src/app/shared/*` |
| `@features/*` | `src/app/features/*` |
| `@env/*` | `src/environments/*` |

O TypeScript usa `strict`, `strictTemplates`, `noImplicitReturns` e `noPropertyAccessFromIndexSignature`.


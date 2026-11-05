---
id: sec.04-3-bootstrap-e-provedores
titulo: "3. Bootstrap e provedores"
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
tokens_estimados: 340
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/frontend.md](../frontend.md).

# 3. Bootstrap e provedores

[`app.config.ts`](../../../frontend/src/app/app.config.ts) registra:

| Provedor | Efeito confirmado |
|---|---|
| `provideZoneChangeDetection` | Coalescência de eventos de change detection |
| `provideRouter(appRoutes)` | Router com as rotas raiz |
| `provideClientHydration()` | Provider presente em `app.config.ts`, sem efeito em CSR |
| `provideAnimations()` | Animações Angular |
| `provideHttpClient(withFetch(), withInterceptors(...))` | Fetch HTTP e `authInterceptor` |
| `importProvidersFrom(NgbModule)` | Provedores do ng-bootstrap |

[`AppComponent`](../../../frontend/src/app/app.component.ts) contém apenas um `router-outlet`. No navegador, ele observa `NavigationEnd` e alterna a classe `scaled` no elemento `<html>`:

- todas as URLs navigadas recebem escala de 80%;
- `/login`, sem string de consulta, fragmento ou barra final, permanece em 100%;
- a remoção da escala ocorre no browser, pois o código só chama `document` no browser.


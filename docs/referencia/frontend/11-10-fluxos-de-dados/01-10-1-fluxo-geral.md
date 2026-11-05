---
id: 11-10-fluxos-de-dados.01
titulo: "10.1 Fluxo geral"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, fluxos]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 131
ficha: docs/modulos/README.md
---
> Parte de [10. Fluxos de dados](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### 10.1 Fluxo geral

```text
bootstrapApplication
  -> appRoutes
  -> authGuard
  -> MainLayout/AdminLayout
  -> componente standalone
  -> serviço root
  -> HttpClient + authInterceptor
  -> API
```


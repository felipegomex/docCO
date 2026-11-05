---
id: 06-5-features-e-status-de-uso.03
titulo: "Navegação registrada para o módulo"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, componentes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 252
ficha: docs/modulos/README.md
---
> Parte de [5. Features e status de uso](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### Navegação registrada para o módulo

Atualizados junto com a implementação: `shared/config/header-nav.config.ts` (link **"Acessos das Empresas"** — `/acessos/clientes`, com badge dinâmico — como item de **Ferramentas**; a seção independente "Acessos Empresas" e o item *Nova unidade* foram removidos da sidebar), o novo `core/services/acessos-badge.service.ts` (lê `totalCount` de `GET /clientes?pageSize=1`, cache de 60 s, signal `badge` consumido por `layout/sidebar/sidebar.component.ts`), `core/services/busca-index.service.ts`, `core/services/recentes.service.ts`, `layout/breadcrumb/breadcrumb.component.ts` e `features/home/home.component.ts` (atalho "Acessos das Empresas").


---
id: 05-4-roteamento.10
titulo: "4.8 Rotas do Dashboard Integrado"
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
tokens_estimados: 548
ficha: docs/modulos/dashboard.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Dashboard Integrado](../../../modulos/dashboard.md).

### 4.8 Rotas do Dashboard Integrado

Todas herdam `authGuard` do shell em `/`; a rota de Financeiro tem `adminGuard` próprio em [`dashboard.routes.ts`](../../../../frontend/src/app/features/dashboard/dashboard.routes.ts).

| URL | Componente/redirect |
|---|---|
| `/dashboard` | `DashboardHomeComponent` (KPIs comercial + administrativo e atalhos; **sem indicadores financeiros**) |
| `/dashboard/comercial` | `ComercialComponent` (shell com 2 abas) → `LeadsKanbanComponent` |
| `/dashboard/comercial/propostas` | `PropostasComponent` |
| `/dashboard/comercial/*` desconhecido | redirect para `''` (Kanban de Leads) |
| `/dashboard/administrativo` | `AdministrativoComponent` |
| `/dashboard/financeiro` | `FinanceiroComponent` (`adminGuard`; exige papel `Administrador`) |
| `/dashboard/*` desconhecido | redirect para `/dashboard` |

O módulo inteiro é lazy (`loadChildren` em [`features.routes.ts`](../../../../frontend/src/app/features.routes.ts)) e nenhuma rota de `/dashboard` está listada em `frontend/prerender-routes.txt`.

#### Abas do módulo Comercial

O módulo tem **duas abas** em [`comercial.component.ts`](../../../../frontend/src/app/features/dashboard/comercial/comercial.component.ts): *Kanban de Leads* (rota padrão) e *Propostas*. A aba **Oportunidades foi removida** por ser redundante com a prospecção: `oportunidades.component.{ts,html,scss}` e `oportunidade-modal.component.ts` foram apagados, e com eles as referências em `comercial.routes.ts` (substituída por `{ path: '**', redirectTo: '' }`), a pill do shell, o índice de busca (`busca-index.service.ts`), o breadcrumb, `header-nav.config.ts` e a home do Dashboard. **A entidade `tboportunidade` e seus endpoints continuam ativos** porque a aba Propostas, o KPI comercial (`ValorPipeline`/`ValorFechado`) e o `valorOportunidades` do card dependem deles; apenas a *tela* deixou de existir.


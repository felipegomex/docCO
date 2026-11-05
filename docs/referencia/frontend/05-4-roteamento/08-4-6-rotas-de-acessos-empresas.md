---
id: 05-4-roteamento.08
titulo: "4.6 Rotas de Acessos Empresas"
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
tokens_estimados: 571
ficha: docs/modulos/acessos.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Acessos Empresas](../../../modulos/acessos.md).

### 4.6 Rotas de Acessos Empresas

`acessos.routes.ts` é lazy e filho de `path: "acessos"` em `features.routes.ts`. Herdam `authGuard` do shell em `/`.

| URL | Componente/redirect |
|---|---|
| `/acessos` | redirect para `clientes` (`pathMatch: full`) |
| `/acessos/clientes` | `ListaClientesCardsComponent` (grade de cards) |
| `/acessos/clientes/novo` | `FormClienteWizardComponent` (assistente de 3 etapas, **declarada antes de** `:id` para não ser engolida pelo param) |
| `/acessos/clientes/:id` | `DetalheClienteComponent` (também aberto como modal a partir da lista) |
| `/acessos/**` | redirect para `clientes` |
| `/ferramentas/acessos` | redirect de compatibilidade para `/acessos/clientes`, declarado em `features.routes.ts` |

⚠️ A rota `/acessos` é **top-level**, Deliberadamente fora de `/ferramentas`: o módulo deixou de ser uma planilha de consulta e passou a ter navegação, estado e rotas próprias. O redirect antigo permanece só para links e favoritos.

`ListaClientesCardsComponent` (`/acessos/clientes`) confirma no código: breadcrumb manual `<nav class="acesso-breadcrumb">` (Início → Acessos das Empresas), **linha de topo única** `.acesso-total-linha` (card com o total de empresas + botão `+ Novo Acesso`, `routerLink="/acessos/clientes/novo"` — o **único** ponto de entrada do assistente; não existe mais no header nem no estado vazio), cards `.acesso-cardempresa` com `min-height: 14rem`, badges `.acesso-badge` (**4 recursos**: RDP, VPN, AnyDesk, Actyon Web — BANCO saiu da grade), código do ERP exibido só como `#N`, **badge de status** `.acesso-status` (● Ativo/● Inativo) e paginação `.acesso-paginacao`. Cada card tem um **menu ⋮** sem caret (`ngbDropdown`, `.acesso-menu`, `::after` anulado) com: Visualizar acessos, Gerenciar acessos, Editar unidade (abre o `FormClienteComponent` como modal) e Inativar unidade (`DELETE` com gate `EDITAR`). Estilos `.acesso-status`/`.acesso-menu`/`.acesso-total-linha` no SCSS do componente.


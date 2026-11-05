---
id: 05-4-roteamento.03
titulo: "4.3 Rotas administrativas"
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
tokens_estimados: 382
ficha: docs/modulos/README.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### 4.3 Rotas administrativas

Todas as linhas abaixo herdam `authGuard` e `adminGuard` do shell em `/admin`.

| URL | Componente/redirect |
|---|---|
| `/admin/cadastros/tipos-projeto` | `TiposProjetoComponent` |
| `/admin/cadastros/tipos-projeto/novo` | `TipoProjetoFormComponent` |
| `/admin/cadastros/tipos-projeto/editar/:id` | `TipoProjetoFormComponent` |

O layout administrativo tem um outlet próprio e não reutiliza `MainLayoutComponent`.

As rotas `/admin/kanban`, `/admin/tarefas` e `/admin/projetos` foram **removidas** em 2026-10-20 (Fase 0 do plano `qualidade-logica`): reaproveitavam os mesmos componentes de `/kanban` e `/implantacao/*` e nenhum link do app apontava para elas. As telas seguem acessíveis nos caminhos canônicos.

> `/admin/ideias/gestores` **não existe**: a tela de gestores é a rota de topo `/gestores-ideias`, na área principal (`authGuard`, sem `adminGuard`) — ver §4.2. Ela fica fora do shell administrativo porque **quem cadastra gestor é administrador**, não porque o módulo de Ideias seja administrativo. `adminGuard` já é o filtro correto e o backend revalida por claim (`eh_admin` ou `perfil_id = 'A'`).


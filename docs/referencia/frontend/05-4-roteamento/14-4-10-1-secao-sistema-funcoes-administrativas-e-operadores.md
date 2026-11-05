---
id: 05-4-roteamento.14
titulo: "4.10.1 Seção Sistema — funções administrativas e operadores"
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
tokens_estimados: 643
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../../modulos/auth-rbac-sistema.md).

### 4.10.1 Seção Sistema — funções administrativas e operadores

O item **Funções Administrativas** (`/sistema/funcoes-administrativas`) é a entrada da seção **Sistema** e tem `recurso: 'aba.sistema.funcoes-administrativas'` no `header-nav.config.ts`. A rota é protegida por `permissaoGuard('aba.sistema.funcoes-administrativas')`; o guard chama `PermissoesService.carregar()` (uma vez, se ainda não houver cache) e, quando a ação `Ler` não está no conjunto efetivo, redireciona para **`/sem-acesso?recurso=<recurso>`** — não mais para `/` em silêncio. O `MainLayoutComponent` também chama `carregar()` no início, para a sidebar reagir ao signal. O backend revalida — o guard é só navegação.

A página renderiza cards de atalho: **Operadores** (`/sistema/operadores`), **Gestores** (`/gestores-ideias`), **Permissões** (`/sistema/permissoes` — tela de controle da matriz, exibida por **RBAC** com `recurso: 'aba.sistema.permissoes'`, não por `hasRole('Administrador')`), **Cadastros** (`/admin/cadastros/tipos-projeto`, só `Administrador`), **Colunas do Kanban** (`/kanban/admin/colunas`, só `Administrador`) e **Auditoria de Logs** (`/auditoria`). `/sistema/operadores` tem o próprio guard `permissaoGuard('aba.sistema.operadores')` e a tela de cadastro usa `SistemaService` para `GET/POST/PUT/PATCH` de operadores e `GET /cargos`. A tela de permissões usa `GET /permissoes/cargos-gerenciaveis` e `GET/PUT /permissoes/matriz/{cargoId}`; a lista de ações é estática, `grupos` é um `computed`, o template usa `trackBy` e a troca de item é imutável no signal (necessário para o `OnPush`).

`PermissoesService` ([`permissoes.service.ts`](../../../../frontend/src/app/core/services/permissoes.service.ts)) mantém o espelho de `GET /api/v1/permissoes/minhas` em signals: `pode(recurso, acao)` testa o bitset de ações e `podeRota(rota)` testa `Ler` pela rota. O `permissaoGuard` ([`permissao.guard.ts`](../../../../frontend/src/app/core/guards/permissao.guard.ts)) usa `pode`; a sidebar aplica `pode(item.recurso, Ler)` quando o item declara `recurso` (o método `podeRota` existe para a checagem por rota). Os bits de `AcaoPermissao` no cliente (`Ler`=1, `Criar`=2, `Editar`=4, `Excluir`=8, `Aprovar`=16, `Gerir`=32, `Auditar`=64) são os mesmos do backend.


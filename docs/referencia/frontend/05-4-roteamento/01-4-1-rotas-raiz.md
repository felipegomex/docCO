---
id: 05-4-roteamento.01
titulo: "4.1 Rotas raiz"
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
tokens_estimados: 172
ficha: docs/modulos/README.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### 4.1 Rotas raiz

| URL | Configuração | Proteção |
|---|---|---|
| `/login` | `LoginComponent` | Pública |
| `/` | `MainLayoutComponent`, com `featuresRoutes` como filhos | `authGuard` |
| `/admin` | `AdminLayoutComponent`, com `adminRoutes` como filhos | `authGuard` + `adminGuard` |
| qualquer URL não mapeada | redirect para `/` | Não há tela de 404 dedicada |


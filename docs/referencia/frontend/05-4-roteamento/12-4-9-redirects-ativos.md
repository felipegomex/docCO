---
id: 05-4-roteamento.12
titulo: "4.9 Redirects ativos"
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
tokens_estimados: 668
ficha: docs/modulos/README.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### 4.9 Redirects ativos

| Origem | Destino | Regra no router |
|---|---|---|
| `/ferramentas/acessos` | `/acessos/clientes` | `pathMatch: full` (compatibilidade com a tela antiga do Google Sheets) |
| `/ferramentas/contra-senha` | `/ferramentas` | `pathMatch: full` |
| `/ferramentas/modelos` | `/modelo-chamados` | `pathMatch: full` |
| `/ferramentas/faq` | `/base-conhecimento?aba=glossario` | `pathMatch: full` (o Glossário virou aba) |
| `/stack` | `/base-conhecimento?aba=stack` | `pathMatch: full` |
| `/trilhas/resolver` | `/base-conhecimento?aba=conhecimento&trilha=resolver` | `pathMatch: full` |
| `/trilhas/sql` | `/base-conhecimento?aba=conhecimento&trilha=sql` | `pathMatch: full` |
| `/trilhas/rede` | `/base-conhecimento?aba=conhecimento&trilha=rede` | `pathMatch: full` |
| `/trilhas/infra` | `/base-conhecimento?aba=conhecimento&trilha=infra` | `pathMatch: full` |
| `/suporte` | `/suporte/visao-geral` | redirect do módulo (`pathMatch: full`) |
| `/desenvolvimento` | `/desenvolvimento/visao-geral` | redirect do módulo (`pathMatch: full`) |
| `/ciaa` | `/ciaa/visao-geral` | redirect do módulo (`pathMatch: full`) |
| `/historias-telas` | `/faq-sistema` | `pathMatch: full` (a tela antiga renderizava o mesmo array; o catálogo de `/faq-sistema` ficou canônico em 2026-10-20) |
| `/whatsapp-flow` | `/fraseologia` | `pathMatch: full` |
| `/empresa` | `/empresa/onboarding` | `pathMatch: full` |
| `/implantacao` | `/implantacao/dashboard` | `pathMatch: full` |
| `/implantacao/kanban` | `/kanban` | `pathMatch: full` (compatibilidade: link antigo, e-mail ou favorito cai no Kanban Geral) |
| `/implantacao/relatorio` | `/implantacao/dashboard` | `pathMatch: full` |
| `/implantacao/agenda` | `/agenda` | `pathMatch: full` |
| `/database` | `/database/visao-geral` | `pathMatch: full` |
| `/database/query-builder` | `/database/consultas` | redirecionamento de um segmento |
| `/admin` | `/admin/cadastros` | `pathMatch: full` |
| `/admin/cadastros` | `/admin/cadastros/tipos-projeto` | `pathMatch: full` |
| `/**` | `/` | redirect global |

A rota comentada `/administrativo` em [`features.routes.ts`](../../../../frontend/src/app/features.routes.ts) não está registrada. Desde 2026-10-20 o `KanbanComponent` é carregado **apenas** por `/kanban` (modo geral): `/admin/kanban` foi removida e `/implantacao/kanban` virou redirect.


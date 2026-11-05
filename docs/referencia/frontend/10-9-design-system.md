---
id: sec.10-9-design-system
titulo: "9. Design system"
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
tokens_estimados: 986
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/frontend.md](../frontend.md).

# 9. Design system

### 9.1 Fundamentos

[`styles.scss`](../../../frontend/src/styles.scss) importa apenas módulos Bootstrap usados: reboot, grid, containers, botões, alerts, nav, dropdown, badge, forms, modal e utilities. Ele define:

- variáveis de marca, superfícies, texto, bordas, erros, overlays, spinners e shadows;
- tokens `Space Grotesk`, `Inter` e `IBM Plex Mono`;
- raios `sm`, `md` e `lg`, ring de foco e tokens do “Console de operações”;
- tema claro no `:root` e tema escuro em `[data-theme='dark']`;
- utilitários compartilhados como `.eyebrow`, `.section-rail`, `.chip`, `.meta-mono` e a família `.adm-*`.

As telas do Dashboard Integrado reutilizam a família `.adm-*` (`.adm-stats`, `.adm-search`, `.adm-pills`, `.adm-btn`, `.adm-empty`, `.adm-table`, `.adm-card`) e o partial compartilhado [`dash-kpis.scss`](../../../frontend/src/app/features/dashboard/shared/dash-kpis.scss), importado via `@use` no `styleUrls` de cada componente (o encapsulamento impede herança automática). Famílias locais distinguem os módulos: `.dash-kanban*` no Kanban de Leads, `.dash-adm*` na tela Administrativa e `.dash-vinculo*` (inclusive `.dash-vinculo-check`) no modal do lead.

### 9.2 Shell e responsividade

| Camada | Comportamento |
|---|---|
| Header | Barra fixa no fluxo superior, busca central, menu de usuário e alternância de tema |
| Sidebar | 17,5 rem expandida, 4,5 rem recolhida, grupos configuráveis (estado dos grupos persistido em `localStorage`, chave `cc.nav.grupos`) e indicador de papel |
| Conteúdo | Coluna flexível com scroll próprio, breadcrumb e outlet |
| Footer | Ano corrente, navegação e versão exibida |
| Página | `PageHeaderComponent` alinha título, descrição e ações projetadas |

O header usa skeleton durante a inicialização da autenticação. O shell principal e o administrativo são visualmente separados; o administrativo não usa sidebar.

### 9.3 Layout compacto de Projetos

[`projetos.component.scss`](../../../frontend/src/app/features/implantacao/pages/projetos/projetos.component.scss) implementa o layout vigente de Projetos:

| Elemento | Implementação |
|---|---|
| Grade | Filtros 15 rem, conteúdo flexível e resumo 16,5 rem; classes `sem-filtros`/`sem-resumo` removem colunas |
| Laterais | Começam abertas por signals; podem ser ocultadas e viram drawers fixos nos breakpoints de 1.400 px e 1.024 px |
| Lista | Projetos ativos no topo e concluídos abaixo; cartões em coluna vertical, de largura total |
| Cartão | [`projeto-card.component.scss`](../../../frontend/src/app/features/implantacao/components/projeto-card/projeto-card.component.scss) e [`projeto-etapa-card.component.scss`](../../../frontend/src/app/features/implantacao/components/projeto-etapa-card/projeto-etapa-card.component.scss) usam gaps, paddings e metadados compactos; a escala global ainda transforma `rem` em 80% fora do login |
| Responsivo | Em até 768 px, a página e os cabeçalhos de grupo reduzem padding e tipografia |

A fonte de dados da lista é `ProjetosService.listarComEtapas()`, portanto cada item carrega Projeto e seus nove cards em uma resposta composta.

### 9.4 Tema e escala

- `HeaderComponent` alterna `data-theme` em `document.documentElement` e persiste apenas `localStorage.theme`.
- `AppComponent` controla a classe `html.scaled`; a escala não depende do tema.
- Valores em `rem` acompanham a escala; bordas e sombras declaradas em pixels permanecem unidades absolutas, conforme o comentário e a implementação do stylesheet.
- A folha global também contém regras de impressão que ocultam header, sidebar e footer.

